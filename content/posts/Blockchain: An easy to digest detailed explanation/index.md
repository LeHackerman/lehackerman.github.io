---
title: "Blockchain: An Easy to Digest Detailed Explanation"
date: 2021-06-20T17:00:00+01:00
draft: false
toc: false
images:
tags:
  - Blockchain
---

Who didn't hear about Bitcoin and the fascinating surge that occured to its value ? And who didn't encounter a post or an article talking about the magic of cryptocurrencies that could make you rich suddenly ? I imagine that everybody could relate to these questions. Unfortunately, this trend and hype is usually used by medias and blogs in order to gain views and reach, but rarely to discuss the technologies behind it. So today, I decided to talk to you about Blockchain (the underlying architecture of Bitcoin), its fundementals and how it revolutionized the currency market.

First of all, I think it's necessary to clarify some misconceptions. Blockchain isn't Bitcoin, and Bitcoin isn't Blockchain. In fact, it's true that Blockchain was created by *Satoshi Nakamoto* for the sole prurpose of inventing the Bitcoin. But, it could be used in way more applications other than cryptocurrencies. So, to make it clear since the beginning, I'm going to talk about the Blockchain as a data structure and a network, in its broad definition, and not about its usage in cryptocurrencies. And I might be a little biased to the original Bitcoin Blockchain . Although, I am going to cover other variations in another post.

**So, to begin with, let's talk about what blockchain is ? and what are its main components ?**  

Blockchain is a *decentralised, distributed* and usually public data structure and network. Its main goal is to be a *trustworthy ledger*, or generally speaking, a *trustworthy archive of information*, without needing a central authority to ensure this trust.  

A Blockchain is formed of blocks. Each block is no more than a set of data that is structured according to strict rules, and is built on top of a previous block, starting from the initial *Genesis Block*. A block's header contains necessarily a *Merkle root*, the previous block's hash (except for the initial one of course) and a *nonce* in some cases (that we'll discuss later). A blocks's body contains the *Merkle Tree* associated with the forementioned *Merkle Root*. Before passing to the next question, let's first discuss what a *Merkle Tree* is.  

To put it in simple terms, a *Merkle tree* is a full binary tree of hashes. *Merkle tree*'s leaves are transactions/data included in the block. Each parent in the tree is a hash of his two siblings. So, if a single transaction is changed, even slightly, the *Merkle Root* will change drastically. This is useful when checking for the integrity of a block.

**So, now we know what a blockchain is, and how it is formed. But, who is responsible for creating and verifying the blockchain ?**  
In fact, the answer to this question is as famous as Bitcoin. The creation and verification of a Blockchain is the miners' job. To better illustrate a miner's work, let's talk about the general process of block creation. First, a transaction/data is issued and broadcasted to the network. It will be then included in a transaction/data pool. Then, a miner will grab a bunch of transactions/data according to some criteria to form a new block. He creates the *Merkle Tree* out of those transactions/pieces of data and includes it in the block. He calculates he nonce if there is any. And when all of this is ready, he adds the block to the blockchain by appending the previous block's hash in the present hash's header.

**But, what if two miners submit their blocks at the same time ?**  

Well, don't worry, blockchain is built ready for this problem. Both blocks will be accepted, but one of them will be later dismissed. In fact, a strict rule in blockchain is that the longest chain must be the main chain. In other words, when both blocks are added, future blocks will be built on either of these conflicting ones. The block on which a longer chain is built will be kept, while the other block and the chain that followed him will be deprecated and orphaned, their transactions/data will be sent back to a transactions/data pool.  

**Here, a serious question arises, how are we sure that every miner will submit an honest trustworthy block ?**  

This is where consensus kicks in. What's consensus ? it is any form of rules, procedures or protocols that leads to a major undebatable agreement. One of many ways to reach consensus in blockchains is the *Proof-Of-Work*. *Proof-Of-Work* is a cryptographic challenge that proves undoubtedly that a certain party has exerced some amount of work. As usual, to make things simple and clear, let's talk about how *Proof-Of-Work* is applied in the case of blockchain. Let's go back to block creation. When a block is created for a blockchain that requires *PoW*, the miner has to find a number called a *nonce* (mentioned above) that, when added to the block's header, results in a block hash that starts with a given number of zeros. But how does this ensures the honesty of the blockchain ? Okay, let's suppose a malicious miner has decided to append a dishonest block (where he tries to spend a coin he already spent for example). Since the rest of the miners will refuse to build on top of this block, an honest block will be appended on the same height, creating a conflict. To make sure his block will be accepted, the dishonest miner must race the rest of miners and create the longer chain on top of his dishonest block. That means his hashrate must surpass the hashrate of all the other miners combined. Put in simple terms, for a dishonest miner to append a dishonest block, he must own over half of the processing power of the blockchain. This is one way of reaching consensus and building an honest blckchain, but it isn't the only one. We'll discuss other methods in another post.

**The image is starting to get clear now. But, we must ask a very important question. Why would a miner bother sacrifice all this power and CPU time with no apparent benefice ?**  

Well, because there is a benefice. The benefice is called an *incentive*. It is some kind of a reward that a miner gets when his block becomes a part of the blockchain. It could take many forms. In Bitcoin there is two forms. One way of getting rewarded is that a miner would append a special transaction on his block that states that he received a certain fixed amount of coins. In this way, if his block is appended, he would possess that amount of coins. The other way is by obtaining a tip from users on every transaction performed, called *transaction fees*. Like this, we would make sure that miners are motivated and satisfied.

This was a little guide on how blockchains work. I hope that you enjoyed your read and that you didn't get bored. Expect another post from me that talks about other variations that are slightly different from the one discussed above. And if you have the slightest comment to add, don't hesitate to contact me.
___
*References:*
- [Bitcoin's original white paper.](https://bitcoin.org/bitcoin.pdf)
- [What is a Merkle Tree and How Does it Affect Blockchain Technology?](https://selfkey.org/what-is-a-merkle-tree-and-how-does-it-affect-blockchain-technology/)
- [But how does bitcoin actually work?](https://www.youtube.com/watch?v=bBC-nXj3Ng4)
