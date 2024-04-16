I wrote this script to talk with the server:
```python
from Crypto.Util.number import inverse
from pwn import *
from random import randrange
REMOTE_SERVER = "challenges.france-cybersecurity-challenge.fr"
REMOTE_PORT = 2151


def craft_valid_signature(p, g, y, m):
    while True:
        # Craft arbitrary values for r and s
        r = randrange(1, p - 1)
        s = randrange(1, p - 1)
        # Calculate g^m mod p
        gm_mod_p = pow(g, m, p)
        # Calculate (y^r * r^s) mod p
        y_pow_r = pow(y, r, p)
        r_pow_s = pow(r, s, p)
        y_pow_r_times_r_pow_s_mod_p = (y_pow_r * r_pow_s) % p
        # If the crafted values pass the verification, return them
        if y_pow_r_times_r_pow_s_mod_p == gm_mod_p:
            return r, s

conn = remote(REMOTE_SERVER, REMOTE_PORT)

vals = conn.recvuntil(b">>> ")
vals = vals.decode("utf-8")
p = int(vals.split('\n')[1].split(" = ")[1])
g = int(vals.split('\n')[2].split(" = ")[1])
y = int(vals.split('\n')[3].split(" = ")[1])

print("p =", p)
print("g =", g)
print("y =", y)

message = b"123456789"
# Send the payload
conn.sendline(message)

res = conn.recvuntil(b">>> ")

m = 123456789  # Chosen message

# Forge signature
r, s = craft_valid_signature(p, g, y, m)

# Ensure r and s are within the correct range
r = r % p
s = s % (p - 1)

# Send the forged signature
print("r =",str(r))
conn.sendline(str(r))
conn.recvuntil(b">>> ")

print("s =",str(s))
conn.sendline(str(s))

# Receive the result
res = conn.recvall(timeout=2)
print(res.decode())

```
