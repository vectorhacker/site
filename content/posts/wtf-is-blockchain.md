---
title: "WTF is Blockchain?"
date: 2018-01-27T19:44:17-04:00
draft: false
---

This post was inspired by a talk given by Jared Haight from Microsoft at [BSidesPR](http://bsidespr.org/2017/). Jared talked about giving back to the InfoSec community at a large, and he mentioned things to write about, including Blockchain. Specifically, he asked "WTF is Blockchain?" and said it that if you actually title it like that, he'd read it. So, this is my attempt at writing that description.

#### TLDR: Blockchain is an unchangeable series of blocks that each reference the previous' block's contents via a hash of that content. Blocks are a collection of transactions that have been grouped together by miners.

# Basic Computer Science
To really understand blockchain techonlogy you must first understand a few computer science concepts. Don't worry, I wont bore you with the details.

## Linked Lists and Merkle Trees
At the heart of blockchain is the concept of Linked Lists and Merkle Trees. A linked list consists of many simple datastructure elements that contain of a value and a pointer. The pointer "points" to another element in the linked list, usually the next one in the list. Linked List elements can point to the next or the previous element or both.

![Linked List](https://steemitimages.com/DQmQRFECQN5yQNGRmwh2yEBHUGiHgVPNKmr8kbJRCYTT5jY/single-list.png)

Simple enough, right? All you really need to know is that Linked Lists are a collection of elements that each point to the next element or the previous.

A Merkle Tree is similar in construction to a Linked List except that each element has two children instead of a previous and next element. This datastructure is used to construct hashes out of smaller hashes. A hash is a cryptographic number that is used to identify some arbitrary data. A Merkle Tree is useful because you can send someone the complete hash and the sub hashes and then verify yourself that the complete hash can be built out of the subhashes. This adds security without trust.

![Merkle Tree](https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Hash_Tree.svg/1200px-Hash_Tree.svg.png)

Each subhash is built out of distinct data blocks. In many blockchain implementations the hashes are usually SHA 256 or above. 

# Blockchain: A chain of blocks and proof of work
![Decentralized Ledger](https://s3.amazonaws.com/cbi-research-portal-uploads/2017/11/20155651/112017-Blockchain-4-V2.png)
The basic building block of blockchain (no pun intended) is a Block. A Block is a collection of transactions that need to stored. Along with the transactions the Block usually contains a few more things that make it immutable (i.e. unchangable). These are the hash of the previous block, something called a nonce number, and the hash of the current block. But how does that hash get generated and what makes it immuatable? Well that hash has to include the contents of the block, the nonce number and the hash of the previous block. The hash also has to match a specific pattern which is very hard to find. In order to find that pattern you must keep incrementing the nonce number in order for the hash to become a certain pattern. Once that pattern is found it is said to be the proof of work for that block. A proof of work is one of the ways many blockchains make sure that blocks have not been tampered with, because generating a cryptographic hash matching a certain pattern that uses the contents of a block is very hard to find. Because it is very hard to find it is a great way to even the playing field when it comes to finding that magic number because everyone has a chance to find it. One added security to this is making the pattern harder to find depending on the number of members in the network.

# Proof of Work:  Incentives!
We've discussed why proof of work makes sure that no block has been tampered with and even why it makes it so that everyone finding the number has about as much chance as everyone else to find it. In public blockchains you can give incentive to people in the network finding number by giving them a reward for confirming blocks. In cryptocurrency blockchains Miners get new cryptocurrency units (Bitcoins, Etherium, Doge Coins...) whenever they mine new blocks. Wait Minners? Mining? Yep, that's what finding that magic number is usually called in the cryptocurrency world. They use minners to create new coins and add to the supply. Minners spend those credits and move it into circulation.

# Summary
I hope this somehow helped you understand a blockchain a little bit more. This is by no means an in depth explination. It's really a TL;DR kind of thing. Nevertheless, I hope you enjoyed it. 