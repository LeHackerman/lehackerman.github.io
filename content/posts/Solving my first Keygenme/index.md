---
title: "Solving My First Keygenme"
date: 2022-06-12T13:31:43+01:00
draft: false
toc: false
images:
tags:
  - Reverse engineering
  - Keygen
  - C++
  - Ghidra

---

I've been wanting to dive in the reverse engineering field for a long time. But, everytime I start learning it I get frustrated and stop. I think this is a feeling that every rookie has faced at some point. So, this time, I decided that I should make a slight change in my learning process. I decided that I'm gonna add some fun and incentive to it, that's why I choose to pursue one of my childhod dreams: cracking softwares. And I decided that I should start solving keygenmes. 

To be honest this wasn't the first one to try out, I solved some easy ones before (too easy to even consider) and I struggled with harder ones. But, this one is the first that presented a challenge for me which ended by solving it and writing a keygen.

## Initial Reconnaissance

Let's start first by running file command on the executable to collect some initial info. We can see that it's a 32-bit PE executable (which is predictable considering the exe extension)
{{< image src="file.png" alt="file being executed on the executable" position="center" style="border-radius: 8px;" >}}

Importing it to PEStudio, we notice it's written in either C or C++ since its PE signature is Microsoft Visual C++. So, our next step would be to decompile it with Ghidra.

{{< image src="pestudio.png" alt="PEStudion analysing the executable" position="center" style="border-radius: 8px;" >}}

## Static analysis

After importing the executable and opening Ghidra's Code Browser, I take a look at the recovered functions. We could see a bunch of weirdly named functions, among which stands out the entry function, which represents Ghidra's attempt at decompiling the executable's entrypoint.  Double-clicking on it, We get a disassembly of the function along with this seemingly scary source code. That's ghidra's attempt at predicting the executable's source code that resulted in the associated machine code. 

{{< image src="entry.png" alt="Entrypoint of our executable" position="center" style="border-radius: 8px;" >}}

An important point to consider is that the entrypoint of a C or C++ binary isn't the main function, but a sequence of initialization routines called before the execution of the user's code. Of course, our first goal is to recover the main function, which could be done in various ways. 
### Finding the main function
#### An intuitive approach
To find the main function, what we could do is run the executable and take a note of the printed strings:

{{< image src="execution.png" alt="Execution of the program" position="center" style="border-radius: 8px;" >}}

Then using Ghidra's defined strings window, we could search for these strings:

{{< image src="definedStrings.png" alt="Executable's Defined String" position="center" style="border-radius: 8px;" >}}

And looking at the references to those strings, this could lead us to the main function, which in our case is FUN\_00401630 :

{{< image src="mainFunction.png" alt="Program's main function" position="center" style="border-radius: 8px;" >}}

#### A methodological approach
Another way to find the main function is to look for the \_\_p\_\_\_argc() and \_\_p\_\_\_argv() function calls, whose purpose is to return the argc and argv before passing them to the main function when called.

{{< image src="mainFunctionCalledDecompiled.png" alt="Decompilation of the section where the main function is called" position="center" style="border-radius: 8px;" >}}

We can see the aforementionned functions' return values pushed the stack. So, the next function call is going to be the main function.

{{< image src="mainFunctionCalledAssembly.png" alt="Disassembly of the section where the main function is called" position="center" style="border-radius: 8px;" >}}

### main function's analysis

At the beginning, we notice this bunch of somewhat look-alike function calls. Seeing they have cin\_exref, cout\_xref and the printed string literals as arguments. It would be safe to guess that they are just printing the prompts and scanning our input. This could be better affirmed seeing that ghidra identified the use of operator>>.

{{< image src="firstPart.png" alt="First part of the main function" position="center" style="border-radius: 8px;" >}}

And then comes this big mess.

{{< image src="mess.png" alt="Inlined function" position="center" style="border-radius: 8px;" >}}

 To be honest, I first I doubted that it's just an inlined standard library's function. But, I decided that I should analyze it anyways to make sure I'm not missing anything. Two days and a lot of frustration later, I just concluded that it is indeed an inlined function that isn't useful in our context. And this was confirmed later by dynamically analysing the program.   
So, I moved on to the rest of the code, And things started making sense. We could see the "win" conditional branch, an if condition that prints correct to the console if its condition is fulfilled.

{{< image src="winBranch.png" alt="Win branch" position="center" style="border-radius: 8px;" >}}

What we have to do now is to find when is that condition fulfilled. First, let's simplify it to `` _DAT_0040543c == DAT_00405440 * 0xe5b0``. We could remember that \_DAT\_0040543c was used before in the call to operator>>. So, we could assume it is one of our inputs. By dynamically analysing the program, I noticed that it holds the Serial value. Now, we have to check where DAT\_00405440 comes from.  
Looking at the instruction that is right before, we could see that DAT\_00405440 is being assigned the result of a certain function. The function is being passed \_dst as a parameter, which is being assigned a value in that messy bunch of code we discussed earlier. To make it simple, \_dst holds our name input, which is confirmed as usual by dynamic analysis.  
Having identified all of our variables, let's have a look at FUN\_00401230. Knowing that it is returning the value compared later, let's first have a look at its return instruction.

{{< image src="processingFunctionReturn.png" alt="The return value of the processing function" position="center" style="border-radius: 8px;" >}}

So, our function is returning uVar4 with some mathematical operations applied. We're gonna jump straight to the block where uVar4 is being assigned a value. We're ignoring all of the preceding code as it's just there to distarct us.

{{< image src="processingFunctionCode.png" alt="The relevant section of the processing function" position="center" style="border-radius: 8px;" >}}

So, let's start understanding what this do while loop does. Through dynamic analysis, I concluded that \_in\_stack\_00000014 is our name input's length and that the ``if(0xf< \_in\_stack\_00000018)`` branch is irrelevant to us. With that out of the way, let's look at the rest of the instructions. We know that param\_1 got passed \_dst, which is our name input. So, ppuVar3 is a pointer to our name input. uVar6, being initialized to 0 before the loop, is incremented by 1 at each iteration. So, it's simply a counter. pcVar1 contains the uVar6'th character of our name input. uVar5 is nothing but an auxiliary buffer holding last iteration's uVar4 value. Having understood all of this, we could finally conclude what's uVa4 holding. But, let's jut clear a little confusing bit before. In C, the char type is simply a 1-byte numerical type. So, when we apply mathematical operations on a char, it is treated as a number. Back at uVar4, at each iteration, it's being assigned the sum of uVar5 and pcVar1 values. That means that at the end of the loop, uVar4 will contain the sum of the ASCII representation of each character in our name's input.  

We're nearly finished. At this level, we know that the value stored in \_DAT\_0040543c should be `` ((uVar4 ^ 6) * 0x498 ^ 7) * 0xe5b0)``. But, there's one little detail that we should pay attention to. To be honest, I've remarked it during the dynamic analysis and I don't know how to identify it from the disassembly : \_DAT\_0040543c is a signed int. That means that our input (which is positive) couldn't be bigger than 0x7fffffff (Since the MSB is a sign bit). So, we need a way to bypass this restriction as some names result in a serial bigger than that value. The solution is easy. In C++, negative values are stored using the two's complement. So here's what are we gonna do : If our serial is less than 0x7fffff, we're good. If not, we'll insert the negative of the two's complement of our serial. That way, the stored value will be our serial. This works obviously because - -x = x.  

We're done, now we know how the serial is calculated, all we have to do is write a keygen. Since this is a simple algorithm, I've opted to writing the keygen in python :

``` Python
#!/usr/bin/env python3

name = input('name: ')
serial = 0
for char in name:
        serial += ord(char)
serial = (((serial^6) * 0x498 ^ 7) * 0xe5b0) & 0xffffffff
if serial > 0x7fffffff:
    serial = - ((~serial+1) & 0xffffffff)
print("Your serial is:  ", serial)

```
If you've got any question, don't hesitate to contact me.
______
#### Remarks:  

- You can find the keygenme here : https://crackmes.one/crackme/614b55d433c5d458fcb36575
- For the sake of simplicity, I didn't discuss the dynamic analysis process.

  
