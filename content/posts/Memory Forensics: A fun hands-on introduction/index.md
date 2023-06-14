---

title: "Memory Forensics: A Fun Hands on Introduction"
date: 2022-03-20T20:30:00+01:00
draft: false
toc: false
images:
tags:
  - Digital Forensics
  - Memory Forensics
  - CyberTrace
  - Workshop

---

Last weekend, **CyberTrace** held its first large-scaled event, the Cyber Summit. It held a multitude of conferences and workshops alongside an on-site CTF at night. And I had the honor to be one of the trainers that animated one of the aforementioned workshops. So, as many attendees asked me to, I'm writing this walkthrough to collect and preserve the knowledge shared through the workshop. Plus, I'm hoping this becomes something of use for people wanting to dive in the forensics field as I didn't find as many well-maintained ressources as i wished while
preparing it.

![CyberTrace](https://cdn-images-1.medium.com/max/800/1*n3sEc6aukj0e0Z93t_stGQ.png)

The workshop's topic was memory forensics, and the concerned audience was total beginners that had no clue what does forensics even mean. Considering the short allocated time, which is 2 hours, I chose to challenge myself preparing some quitely dense content and trying to deliver it efficiently. My goal was to introduce the audience to the field without being too superficial. Plus, I was really tired of the classiscal method that the majority of forensics introductions would approach, that is just loosely defining the field and then bombing the audience with volatility's plugins and what they do. So, considering all these factors, I decided that my workshop should be nothing like that and follow a kind of exotic approach.

### Workshop's scenario and tools:
I made the workshop as to be a real life simulation that goes like this:  
A non-IT friend of yours notices some weird behaviour on his computer. Random files would appear and disappear, sudden screen blackouts, indicator leds going on and off. So, he reachs out to you, the Cybersecurity saviour, to diagnose his problem. We instruct him on how to take a memory dump of his computer and send it to us, and the rest of the procedure is what we're gonna do during the workshop.

Before we start, let's talk a little about the tools we're gonna need. In fact, it's only one tool called Volatility. It could be considered as the de-facto framework for memory analysis. It's a modular open source tool that contains a multitude of plugins, each one analyzes certainaspects of a dump and provides an output that helps the analyst create an outlook on what is happening. Volatility has two major versions, 2 and 3. Volatility 2 is the well developed older release written in python2. Volatility 3 is the new player in the scene, a complete rewrite of its predecessor written in python3. Although a little underdeveloped as it is somewhat new, it's faster than it's big brother, and we're gonna rely on it throughout the workshop.

![Volatility's logo](https://cdn-images-1.medium.com/max/800/1*KfO3uxglMpLOYcyFJ4iDiA.png)

### Enough talking, let's get our hands dirty: 
First thing we must do is to try and get some initial info on the machine, this could be done using *windows.info* (We already know our friend uses windows). This plugin contain some intuitive but necessary info such as the OS version, architecture type and the system time when the dump was made.

{{< image src="info.png" alt="windows.info" position="center" style="border-radius: 8px;" >}}

We could see that our friend has windows 10. Starting with volatility 3 was a good decision, because if we were to run volatility 2's *imageinfo* on a windows 10 dump, it would have taken an eternity.

Having a black box dump, a nice starting point would be checking what processes were runnig on the computer, this could be done using *windows.pslist* plugin (For now, as sometimes a more thorough scan would be needed using *windows.psscan*).

What i truly advise when using volatility is to save the output of each plugin in a file as it's sometimes very long and verbose which could be hard to manipulate on the terminal.  
We're gonna need the output of *windows.pslist* throughout the whole workshop, but for now, let's analyze it and check for interesting
processes.

``` 
python3 vol.py -f ./workshopMemoryDump.raw windows.pslist | tee ./Workshop/pslist
```

{{< image src="pslistChrome.png" alt="chrome.exe in windows.pslist's output" position="center" style="border-radius: 8px;" >}}

We could notice **chrome.exe** running, which is pretty interesting knowing that we could extract its history searching for some leads. The problem is that, as we stated above, volatility 3 is still in its early days and lacking many plugins, namely the volatility 2's community plugin *chromehistory*. We could of course extract history the hard way (By dumping chrome's database), but, for the sake of simplicity, I'm going to fallback to volatility 2.

{{< image src="chromehistory.png" alt="Volatility2's chromehistory output" position="center" style="border-radius: 8px;" >}}

Okay, pretty interesting, from what we've seen so long, we could try to form an initial image of what our friend was doing. I think he wanted to crack his GTA 5 installation, so he went through torrent sites to find a valid crack. This could be reinforced by remarking the
**qbittorrent.exe** process running.  

I think it's time to have a look at what we could recover from our friend's files. This could be done using *windows.filescan* plugin. An important note worth remembering here is that *windows.filescan* is not gonna show us a list of all the files that exist on the system, rather it's going to list files recently or currently loaded in memory.

``` 
python3 vol.py -f ./workshopMemoryDump.raw windows.filescan| tee ./Workshop/filescan
```

Since the output of this plugin is really really really long (as it's going to list all of the operating systems files and dlls alongside other entries). We need to look for certain patterns. What I prefer doing is grepping common user directories such as *Desktop*, *Documents*
and *Downloads*.

{{< image src="grepDownloads.png" alt="Grepping User directories in windows.filescan's output" position="center" style="border-radius: 8px;" >}}

One remarkable entry in the output is the **GTA V Crack** directory, containing **patch.exe** and **README.txt**. As we all know, READMEs usually contain instructions as to how to use a software, so let's take a look at **README.txt** by dumping it. All we need to do is to execute the *windows.dumpfiles* plugin, which is used, as its name states, to dump files from memory. An important fact about dealing with files in relation to memory is that finding them in *windows.filescan* doesn't necessarily mean it's always possible to dump them. This is a consequence to the volatility of the memory, knowing that a file has existed at a certain time in the memory doesn't mean is still does exist. In other words, not all files that shows up in the output of *windows.filescan* could be dumped.

{{< image src="dumpfiles.png" alt="Trying to dump README.txt through windows.dumpfiles" position="center" style="border-radius: 8px;" >}}

We could see that **README.txt** couldn't be dumped, and this is something I struggled with when preparing the lab, due to the very small size of the text file. But, we musn't get desperate, because there's a solution. The NTFS filesystem, used in windows since 1993 uses a data structure known as the Master File Table to index all its files. Each record in the MFT includes some metadata about the file and a pointer to its location on the disk, *usually*. This is not the case for very small files, which NTFS stores alongside their metadata in their MFT records, such files are called *resident files*. Here, Volatility3 prematurity arises again, as it only contains *winows.mftscan*, which is aged a mere 2 months and is incapable of extracting info from the MFT, so we're going to use volatility 2's *mftparser*.

``` 
python2 vol.py -f ./workshopMemoryDump.raw mftparser | tee ./Workshop/mftparser
```

{{< image src="readmemft.png" alt="README.txt MFT record" position="center" style="border-radius: 8px;" >}}

And as we predicted, **README.txt**'s data is there in its MFT record, and it is instructing the user to run **patch.exe**. Going back to *windows.pslist*'s output, we could see that **patch.exe** is in fact running.

{{< image src="patch.png" alt="patch.exe in windows.pslist" position="center" style="border-radius: 8px;" >}}

So, let's take a little pause to recapitulate our findings. What we found out to this point is this: Our friend, whose computer had weird behaviour, was searching for a crack for GTA 5 on torrent sites. He apparently downloaded a certain torrent and ran the shady executable it contains, as he was instructed by the README. I think that the culprit is becoming crystal clear, **patch.exe** probably contains a malware. But, let's not rush into a conclusion this fast. In fact, as of this point, many analysis possibilities show up. But, I'm going to choose an easy path, for the sake of simplicity. Knowing that the weird behaviour on our friend's computer doesn't have a clear pattern, we could safely assume that if a malware exists, it's controlled by an attacker. That means that it needs to connect to a remote address. So, let's check remote connections made by processes using *windows.netstat *.

``` 
python3 vol.py -f ./workshopMemoryDump.raw windows.netstat| tee ./Workshop/netstat
```

{{< image src="patchNetstat.png" alt="patch.exe connecting to the remote port 4444" position="center" style="border-radius: 8px;" >}}

As we predicted, **patch.exe** is connecting to a remote address. The remarkable thing about this connection is the remote port which is **4444**. For those who don't know, **4444** is the default listening port for *Metasploit's Meterpreter*. So it's somewhat safe to assume by a high probability that the computer is controlled by a *meterpreter*. But a remote port alone isn't enough to strictly confirm. That's why we're continuing our analysis by executing *windows.malfind*, which tries to locate malicious code instructions and dlls inside the memory. For the sake of simplicity, here's what I'm going to do: I'm going to run *windows.malfind* with the --- dump flag which dumps the suspicious memory areas and then pass it through *clamav* to test it.

``` 
python3 vol.py -f ./workshopMemoryDump.raw windows.malfind | tee ./Workshop/malfind
```

{{< image src="clamscan.png" alt="clamscan result" position="center" style="border-radius: 8px;" >}}

As we could see, our suspicion turned out to be true, a *meterpreter* was detected by *clamav*.

## Conclusion: 

To conclude our analysis , let's have a little summary of our findings. Our friend, who have a weirdly-behaving computer, searched for a GTA 5 crack and installed a sketchy executable through some torrent then ran it. This executable contained a *meterepreter* which infected the computer and allowed the attacker to control it and execute whatever malicious command he wished to.

**Coming to this conclusion, I mark the end of this workshop. I hope you find it useful, and I hope that it's clear enough for you to understand as I tried to engage in some pretty advanced concepts without losing the beginner-friendly aspect. I'd be so happy hearing your feedback, remarks or criticism. So, don't hesitate to contact me.**
____

### Remarks: 

- For reference, here's how a volatility command usually looks like:  

```
python vol.py -f < Memory dump's path > < plugin name > < plugin arguments>
```

- Why a Windows 10 dump ? Because I've had enough of Windows 7 tutorials. Yeah it's more convenient to use but it's way too outdated. People don't use it anymore. So, they can't relate to it. Why Volatility3 ? for the same reason, enough of volatility2. Volatility3 is faster, more compatible with Windows 10, and is starting to catch on Volatility2. Plus, if anyone wants to learn Volatility2 and Windows7 , the internet is full of such tutorials, they won't need to attend a workshop and waste their time.
- You could notice that the workshop's flow was kind of linear. That's was made on purpose to keep the attendee's attention. But beware, real life isn't even nearly like this. That's why you should always be cautious not to fall in a rabbit hole.
- I advise you to reproduce the steps stated above on your own to better grasp the discussed concepts.
- Bonus tips: you could dump a process's memory space using windows.memmap, or its binary using windows.dumpfiles.
____
### Ressources:
- [The memory dump used.](https://mega.nz/file/AvYHHbqa#tfPm4ZhkS7IUqat7ab__9SKVRtw96BB6Q2R6JqCwm2Q)
- [Volatility 3's repo.](https://github.com/volatilityfoundation/volatility3)
- [Volatility 2's repo.](https://github.com/volatilityfoundation/volatility)
