+++
published = true
categories = ["writeup"]
title = "CSAW CTF Quals 2018 Writeup"
date = 2018-09-18T00:00:00.000Z
author = "batterycharger"
tags = ["csaw", "quals", "ctf", "python", "forensics", "web", "misc"]
+++
## CSAW CTF Quals 2018

The Quals for the CSAW CTF happened from 14th-16th September 2018, I participated with my team `eavesdroppers` from the India region. After the 48 hour toil, we ended up 8th Regionally and 87th Globally. 

## Challenges

### `bin_t` - Misc

On connecting to the given server and port using `nc misc.chal.csaw.io 9001`, we receive a list of integers and and instructions to follow, i.e, insert the numbers in an AVL Tree and return the pre order traversal list of the balanced AVL Tree. 

```
Add these numbers to a AVL Binary Tree, then send them back in the preorder traversal!
94,5,58,87,38,32,96,47,2,63,11,4,61,83,7,57,5,11,30,100,73,22,68,7,99,8,24,53,3,58,11,86,97,95,42,19,88,23,31,64,19,15,88,58,87,51,75,22,29,5,74,11,14,51,45,35,14,23,50,59,58,75,36,13,1,73,2,38,56,33,26,6,92,92,50,30,83,28,20,86,80,82,56,32,2,91,13,98,65,59,68,23,81,37,25,3,38,69,13,99
Send the preorder traversal in a comma sperated list.
```

Upon scouring the internet for some code to create an AVL Tree, I found an `AVLTree` Class in Python and glued it together with the server's response, upon sending the correct preorder traversal list (after going through multiple AVL Tree classes!), the flag was received.

```
38,27,8,3,1,0,6,4,14,12,9,10,13,22,17,15,19,26,24,32,30,29,31,34,33,37,35,73,57,49,45,41,40,43,48,47,55,52,66,62,61,64,63,65,70,69,71,90,85,79,78,82,83,87,86,89,97,92,99,98,100
you got it!
flag{HOW_WAS_IT_NAVIGATING_THAT_FOREST?}
```
Flag: **flag{HOW_WAS_IT_NAVIGATING_THAT_FOREST?}**

Code: [Gist](https://gist.github.com/arush15june/d44efc6af73d81cc4aa5a3a560c8fa82)

### `sso` - Web
  **TODO**
  
### `Algebra` - Misc
  **TODO**
  
### `Ldab` - Web
  **TODO**
  
### `Take an L` - Misc
  **TODO**
  
### `Rewind` - Forensics
  **TODO**

### `simple_recovery` - Forensics
  **TODO**
