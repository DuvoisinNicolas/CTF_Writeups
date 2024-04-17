We're given an encrypted archive, and the challenge says that the packets on the target aren't updated.

After some researches, i found this article:
https://www.acceis.fr/cracking-encrypted-archives-pkzip-zip-zipcrypto-winzip-zip-aes-7-zip-rar/

It says that some files zipped with the wrong crypto method can lead to cracking.

![](_attachments/Pasted%20image%2020240407143056.png)

Running it on my zipped file:
```bash
debian@PC:~$ 7z l -slt archive.encrypted | grep Method
Method = ZipCrypto Deflate
Method = ZipCrypto Deflate
Method = ZipCrypto Deflate
```

The articles specifies that `It is also possible if the archive use ZipCrypto Deflate but it is harder since files are compressed before encryption.`

So i might look for that kind of exploit online first.

Also:
```
### Recover internal keys

[](https://github.com/kimci86/bkcrack#recover-internal-keys)

The attack requires at least 12 bytes of known plaintext. At least 8 of them must be contiguous. The larger the contiguous known plaintext, the faster the attack.
```

I need to know some part of the plaintext.
I can guess FCSC{...}, but that's only 6 bytes.

I found this tool, i'll clone it to try how it works:
https://github.com/kimci86/bkcrack#recover-internal-keys

So, i need to know at least 12 bytes of the plaintext.
That could technically be the header of tgz file ?
I tried using a tgz of mine:
```bash
debian@PC:~$ xxd test.tgz
00000000: 1f8b 0800 0000 0000 0003 edce b10a 8340  ...............@
00000010: 1084 e1ad f314 f704 b2eb dde9 f328 5e91  .............(^.
00000020: 2645 5cc1 c78f 4202 3622 1622 81ff 6ba6  &E\...B.6"."..k.
00000030: 9829 c6cb ec95 cf2e 17d2 4593 d29a d666  .)........E....f
00000040: dde6 5716 4b8d a66c 758c ada8 d51a 4d82  ..W.K..lu.....M.
00000050: 5e79 ea67 1abd 7b87 2043 e99f dd6b 7f77  ^y.g..{. C...k.w
00000060: d4ff 292f a33f ee3e 0100 0000 0000 0000  ..)/.?.>........
00000070: 0000 0000 0000 38ed 0325 2782 0800 2800  ......8..%'...(.
00000080: 00
```
The header seems to be fixed, if i do another content with another file:
```bash
debian@PC:~$ xxd test2.tgz
00000000: 1f8b 0800 0000 0000 0003 edce b10a 8340  ...............@
00000010: 1084 e1ad f314 f704 b2eb dde9 f328 5e91  .............(^.
00000020: 2645 5cc1 c78f 4202 3622 1622 81ff 6ba6  &E\...B.6"."..k.
00000030: 9829 c6cb ec95 cf2e 17d2 4593 d29a d666  .)........E....f
00000040: dde6 5716 4b8d a66c 758c ada8 d51a 4d82  ..W.K..lu.....M.
00000050: 5e79 ea67 1abd 7b87 2043 e99f dd6b 7f77  ^y.g..{. C...k.w
00000060: d4ff 292f a33f ee3e 0100 0000 0000 0000  ..)/.?.>........
00000070: 0000 0000 0000 38ed 0325 2782 0800 2800  ......8..%'...(.
00000080: 00
```

Let's try to use the tool now, saying that we have the same header than the tgz file in the archive.

I created a similiar zip with the same encryption method:
```bash
debian@PC:~$ ./bkcrack -L test.zip
bkcrack 1.6.1 - 2024-01-22
Archive: test.zip
Index Encryption Compression CRC32    Uncompressed  Packed size Name
----- ---------- ----------- -------- ------------ ------------ ----------------
    0 Other      Deflate     cff00887          129          150 test.tgz
```

```bash
debian@PC:~$ ./bkcrack -C archive.encrypted -c tmp/temp-scanner-archive-20240315-065846.tgz -P test.zip -p test.tgz
bkcrack 1.6.1 - 2024-01-22
Zip error: entry "test.tgz" is encrypted.
```

Uh, i get an error because test.tgz is encrypted...

I saw an example online that was giving their file:
```bash
debian@PC:~$ ./bkcrack -L /mnt/c/Users/duvoi/Downloads/plain.zip
bkcrack 1.6.1 - 2024-01-22
Archive: /mnt/c/Users/duvoi/Downloads/plain.zip
Index Encryption Compression CRC32    Uncompressed  Packed size Name
----- ---------- ----------- -------- ------------ ------------ ----------------
    0 None       Store       d509ee84           38           38 plain.txt
```
The difference here is that encryption is set to "none", where mine is "others".

I retried specifying that i don't want any encryption:
```bash
debian@PC:~$ ./bkcrack -L test.zip
bkcrack 1.6.1 - 2024-01-22
Archive: test.zip
Index Encryption Compression CRC32    Uncompressed  Packed size Name
----- ---------- ----------- -------- ------------ ------------ ----------------
    0 None       Deflate     cff00887          129          122 test.tgz
```

I used that python script to zip my file:
```python
import sys
import zipfile

def create_zip(input_file, output_zip):
    with zipfile.ZipFile(output_zip, 'w', zipfile.ZIP_DEFLATED, allowZip64=True) as zipf:
        zipf.write(input_file)

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print("Usage: python script.py <input_file> <output_zip>")
        sys.exit(1)

    input_file = sys.argv[1]  # Input file
    output_zip = sys.argv[2]  # Output ZIP file name

    create_zip(input_file, output_zip)

```

But..
```bash
debian@PC-Nico:~$ ./bkcrack -C archive.encrypted -c tmp/temp-scanner-archive-20240315-065846.tgz -P test.zip -p test.tgz
bkcrack 1.6.1 - 2024-01-22
[15:01:27] Z reduction using 115 bytes of known plaintext
100.0 % (115 / 115)
[15:01:27] Attack on 74349 Z values at index 6
100.0 % (74349 / 74349)
[15:02:00] Could not find the keys.
```

That didn't work :(

I found this online:
"`gzip`" is often also used to refer to the `gzip` file format, which is:
- a 10-byte header, containing a magic number (1f 8b), compression ID (08 for DEFLATE), file flags, a 32-bit timestamp, compression flags and operating system ID.
- optional extra headers denoted by file flags, such as the original filename
- a body, containing a DEFLATE-compressed payload
- an 8-byte footer, containing a CRC-32 checksum and the length of the original uncompressed data, modulo 2^{32}

So gzip has "fixed" 10 bytes data.
We could technically bruteforce the 2 last ones, but that would take years (65000~ times of ~2min runs).

After a while, i read the article again, and i understood better the last part: the CRC.

"We can obtain a free additional byte from CRC, as explained in the [ZIP file format specification](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT), a 12-byte encryption header in prepended to the data in the archive. The last byte of the encryption header is the most significant byte of the file’s CRC."

"We can get the CRC of the file using various tools:"

```plaintext
$ 7z l -slt archive.zip  logo_acceis.svg | grep CRC
CRC = 1916B617
$ unzip -Z -v archive.zip logo_acceis.svg | grep CRC
  32-bit CRC value (hex):                         1916b617
```

"So the byte just before the plaintext (at offset -1) is 0x19.
Then we can start the attack using [bkcrack](https://github.com/kimci86/bkcrack):"

```plaintext
$ printf '<?xml version="1.0"' > plain.svg
$ bkcrack -C archive.zip -c logo_acceis.svg -p plain.svg -x -1 19
```


So instead of bruteforcing 2 bytes, we can bruteforce 1, which is more reasonable.

So first, let's extract the CRC last byte for our tar archive.

```
debian@PC:~$ unzip -Z -v archive.encrypted  tmp/temp-scanner-archive-20240315-065846.tgz | grep CRC
  32-bit CRC value (hex):                         126407b2
```

So our CRC value in hex is `0xb2`.

Now we can write a kinda bruteforce script !
```
debian@PC-:~$ xxd b.tgz | head
00000000: 1f8b 0800 0000 0000 0003 ec9b 0b70 5cd5  .............p\.
00000010: 79c7 efae 5e2b 5bb2 5660 8178 85c5 c844  y...^+[.V`.x...D
00000020: 60cb ecea 2db0 4146 5afb 2e96 8c90 6423  `...-.AFZ.....d#
00000030: 1ef6 d5ea ee95 74f1 bebc 7b57 5a89 9782  ......t...{WZ...
00000040: f18c b6ae a6a2 09a9 9916 a264 c28c 9b96  ...........d....
00000050: c6a1 819a 26b4 2260 ec49 0b95 490c ee4c  ....&."`.I..I..L
00000060: 49dc 14c8 12d7 d83c 0232 60dc 73ee 39f7  I......<.2`.s.9.
00000070: ecb7 873d c638 9399 74c6 cbc8 f7fe 7fe7  ...=.8..t.......
00000080: 9cef 7cdf 775e 771f 0c6c 5163 7e75 8bf4  ..|.w^w..lQc~u..
00000090: c77c b9d1 aba9 a101 5f3d 4d0d 6e78 75bb  .|......_=M.nxu.

debian@PC-:~$ xxd a.tgz | head
00000000: 1f8b 0800 0000 0000 0003 edce 3d0a c240  ............=..@
00000010: 1086 e195 3436 7a03 713b ad64 d7ec 668f  ....46z.q;.d..f.
00000020: e111 24fe 2016 a648 368d 95bd 37b0 ca55  ..$. ..H6...7..U
00000030: f420 dec0 dad6 9580 2828 5641 84f7 81e1  . ......((VA....
00000040: 1b66 a618 9f97 d97c ea57 db51 28d1 0c15  .f.....|.W.Q(...
00000050: 24c6 dc53 3bab 9eb3 66ad d026 51c6 8eb5  $..S;...f..&Q...
00000060: d661 ae63 6562 2155 43ff bc28 0b9f e652  .a.ceb!UC..(...R
00000070: 8ac5 72b6 4eb3 cf77 dff6 7faa bf6f d74d  ..r.N..w.....o.M
00000080: 7439 b63a 214f d575 b2e9 baea 71b1 73d1  t9.:!O.u....q.s.
00000090: e1dc 1b88 e14f fe03 0000 0000 0000 0000  .....O..........
```
The headers of my 2 files are starting with "1f8b 0800 0000 0000 0003 e".
I'm adding the "e", because that might help for the bruteforce attempts ?

So i wrote a script:
```python
import os

# Create a file named 'c.tgz' and write the updated initial hex values
with open('c.tgz', 'wb') as f:
    hex_values = bytearray.fromhex('1f8b080000000000000300')
    f.write(hex_values)

# Loop over all possible bytes and replace the last byte in the file
for i in range(224, 240):

    with open('c.tgz', 'r+b') as f:
        f.seek(-1, os.SEEK_END)  # Move the cursor to the position of the last byte
        f.write(bytes([i]))  # Replace the last byte with the new byte

    # Run the command ./bkcrack -C archive.encrypted -c tmp/temp-scanner-archive-20240315-065846.tgz -P test.zip -p c.tgz -x -1 b2
    bkcrack_command = './bkcrack -C archive.encrypted -c tmp/temp-scanner-archive-20240315-065846.tgz -p c.tgz -x -1 b2'
    os.system(bkcrack_command)

```


And i got the keys!!!!
```
debian@PC-:~$ python3 hacks.py
bkcrack 1.6.1 - 2024-01-22
[17:38:05] Z reduction using 4 bytes of known plaintext
100.0 % (4 / 4)
[17:38:05] Attack on 1321248 Z values at index 6
Keys: 89155ba3 b51db097 b766c61e
0.5 % (6058 / 1321248)
Found a solution. Stopping.
You may resume the attack with the option: --continue-attack 6058
[17:38:07] Keys
89155ba3 b51db097 b766c61e
```

So now that i got keys... let's try to guess the password :)

But they didn't work.
In fact, multiple didn't work.

Since i'm brute forcing, there might be cases where a key is possible, but since my plaintext is wrong (since i'm guessing 1 byte), there might be errors.

Also, once a key has been found, the program don't continue.
Instead it prints "You may resume the attack with the option: --continue-attack 1173701"

Maybe our key was inside these, but since it didn't continue...
I will retry for those that succeeded, adding the header by hand.



Here are the keys i've got, and that didn't work:
```
e0: 
- 89155ba3 b51db097 b766c61e
- 51d642c7 3183ef7f c109cfb0
- cdd93550 f16a2876 6d03768a
- 8471e0a2 7300a04c 90c5b7d8
e1: 
- 90b676c3 7ce61c1f c0ccb340
e2: Fail
e3: Fail
e4: 
- d98b8062 81686a16 aee18df6
- 17e9c82a ff5bd7c0 dbd8f296
e5: 
- b40f157c 106d75b1 749dbca0
- 0394bbbc fdeb9241 62ec815b
- 3294fe98 8b076410 294ef991
e6: 
- 37c11287 6eb64212 65772b72
e7: 
- 8ec09e4a 4ceafbeb 21886cfc
- 8c8e962e 42ec3d8f 477cc37e
e8: 
- Fail
e9: 
- b3e80223 360b05e0 c27c8c7f
ea: Fail
eb: Fail
ec: 
- a631f388 b83e2269 8fec7599
ed: 
- 44cfeac6 3592e264 ebf21594
ee: 
- fb252f93 9839a437 610513e8
ef: 
- ce40e286 9776e880 8ec285e7
```
I'm starting to lose faith...

Tonight i'll run the command on every single byte possible using this script:
```python
import os
import subprocess
import re

# Create or open the file to save the keys
keys_file_path = 'extracted_keys.txt'
with open(keys_file_path, 'w') as keys_file:
    keys_file.write('')  # Clear the file contents if already exists

# Create a file named 'c.tgz' and write the updated initial hex values
with open('c.tgz', 'wb') as f:
    hex_values = bytearray.fromhex('1f8b080000000000000300')
    f.write(hex_values)

# Loop over all possible bytes and replace the last byte in the file
for i in range(256):

    with open('c.tgz', 'r+b') as f:
        f.seek(-1, os.SEEK_END)  # Move the cursor to the position of the last byte
        f.write(bytes([i]))  # Replace the last byte with the new byte

    # Run the bkcrack command and capture its output
    bkcrack_command = ['./bkcrack', '-C', 'archive.encrypted', '-c', 'tmp/temp-scanner-archive-20240315-065846.tgz', '-p', 'c.tgz', '-x', '-1', 'b2'] #'-e']
    result = subprocess.run(bkcrack_command, capture_output=True, text=True)

    # Search for the line containing the keys in the command output
    match = re.search(r'Keys: ([a-f0-9]+) ([a-f0-9]+) ([a-f0-9]+)', result.stdout)
    if match:
        # If keys are found, append them to the file
        with open(keys_file_path, 'a') as keys_file:
            keys_line = f'{match.group(1)} {match.group(2)} {match.group(3)}\n'
            keys_file.write(keys_line)

```
And then read flags using this one:
```python
import subprocess

def decrypt_and_show_flag(key_line):
    # Split the line into individual keys
    keys = key_line.strip().split()
    if len(keys) == 3:
        # Construct the bkcrack command with the keys
        command = ['./bkcrack', '-C', 'archive.encrypted', '-c', 'data/flag.txt', '-d', 'flag.txt', '-k'] + keys
        # Run the bkcrack command with suppressed output
        subprocess.run(command, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

        # Use strings to filter the readable content from flag.txt
        try:
            result = subprocess.run(['strings', 'flag.txt'], capture_output=True, text=True)
            printable_content = result.stdout
            print(f"Readable content of flag.txt with keys {key_line.strip()}:\n{printable_content}\n")
        except FileNotFoundError:
            print("flag.txt not found. Decryption may have failed or the file does not exist.")

# Read the keys from extracted_keys.txt
with open('extracted_keys.txt', 'r') as keys_file:
    for key_line in keys_file:
        decrypt_and_show_flag(key_line)

```

And pray...

That didn't work... T_T


Also, i realized:
Since the zip is using the method Deflate, i have to know the entire file i'm attacking as plaintext. Because zips are stored encrypted in the archive, therefore if i don't use the exact same file, the encryption will be different and i won't be able to process the attack.


After some days, i though i could maybe find the "home/VERSION" file online, as we know the service that was used.

I used Google dorking to do my research:
`"/home/VERSION" ivanti`

I finally ended up on this page:
https://www.synacktiv.com/sites/default/files/2024-01/synacktiv-pulseconnectsecure-multiple-vulnerabilities.pdf

![](_attachments/Pasted%20image%2020240409130935.png)

I tried to echo the content of the VERSION file and count its size:
```bash
duvni@duvni:~/Documents/RnD/FCSC$ echo 'export DSREL_MAJOR=22
export DSREL_MINOR=4
export DSREL_MAINT=2
export DSREL_DATAVER=5000
export DSREL_PRODUCT=ssl-vpn
export DSREL_DEPS=ive
export DSREL_BUILDNUM=1531
export DSREL_COMMENT="R2"' | wc
      8      16     194
```

194 ? Exactly like our file (uncompressed) in the archive ! :
```bash
duvni@duvni:~/Documents/RnD/FCSC$ bkcrack-1.6.1-Linux/bkcrack -L archive.encrypted 
bkcrack 1.6.1 - 2024-01-22
Archive: archive.encrypted
Index Encryption Compression CRC32    Uncompressed  Packed size Name
----- ---------- ----------- -------- ------------ ------------ ----------------
    0 ZipCrypto  Deflate     126407b2        64697        64714 tmp/temp-scanner-archive-20240315-065846.tgz
    1 ZipCrypto  Deflate     6c3a35f8          194          120 home/VERSION
    2 ZipCrypto  Deflate     07ff9365           33           44 data/flag.txt
```


I searched for other versions of this file using this research:
`"DSREL_DATAVER="`

And i found this page:
https://www.assetnote.io/resources/research/high-signal-detection-and-exploitation-of-ivantis-pulse-connect-secure-auth-bypass-rce

```bash
duvni@duvni:~/Documents/RnD/FCSC$ echo 'export DSREL_MAJOR=22
export DSREL_MINOR=3
export DSREL_MAINT=1
export DSREL_DATAVER=4802
export DSREL_PRODUCT=ssl-vpn
export DSREL_DEPS=ive
export DSREL_BUILDNUM=1647
export DSREL_COMMENT="R1"' > version.txt 
```
That matches exactly our version.

I made a similar version of the zip (same compression method, no password):
```bash
duvni@duvni:~/Documents/RnD/FCSC$ bkcrack-1.6.1-Linux/bkcrack -L version.zip 
bkcrack 1.6.1 - 2024-01-22
Archive: version.zip
Index Encryption Compression CRC32    Uncompressed  Packed size Name
----- ---------- ----------- -------- ------------ ------------ ----------------
    0 None       Deflate     6c3a35f8          194          108 version.txt
```

I ran Bcrack:
```bash
duvni@duvni:~/Documents/RnD/FCSC$ ./bkcrack-1.6.1-Linux/bkcrack -C archive.encrypted -c home/VERSION -P version.zip -p version.txt
bkcrack 1.6.1 - 2024-01-22
[13:35:33] Z reduction using 101 bytes of known plaintext
100.0 % (101 / 101)
[13:35:33] Attack on 83134 Z values at index 6
Keys: 6ed5a98a a1bb2e0e c9172a2f
67.4 % (56035 / 83134)
Found a solution. Stopping.
You may resume the attack with the option: --continue-attack 56035
[13:36:20] Keys
6ed5a98a a1bb2e0e c9172a2f
```
I found the keys, which i used to change the archive password:
```bash
duvni@duvni:~/Documents/RnD/FCSC$ bkcrack-1.6.1-Linux/bkcrack -C archive.encrypted -k 6ed5a98a a1bb2e0e c9172a2f -U unlocked.zip 123
bkcrack 1.6.1 - 2024-01-22
[13:38:14] Writing unlocked archive unlocked.zip with password "123"
100.0 % (3 / 3)
Wrote unlocked archive.
```

And got the flag after unzipping it !

```bash
duvni@duvni:~/Documents/RnD/FCSC$ unzip unlocked.zip 
Archive:  unlocked.zip
[unlocked.zip] tmp/temp-scanner-archive-20240315-065846.tgz password: 
  inflating: tmp/temp-scanner-archive-20240315-065846.tgz  
  inflating: home/VERSION            
  inflating: data/flag.txt           
duvni@duvni:~/Documents/RnD/FCSC$ cat data/flag.txt 
50c53be3eece1dd551bebffe0dd5535c
```

