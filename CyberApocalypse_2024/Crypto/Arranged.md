This looks like the same challenge than Blunt, but it's using Diffie Hellman using Elliptic Curves.
I know literally nothing about these, but time to break it down ! :)

Here is the code:

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Util.number import long_to_bytes
from hashlib import sha256

from secret import FLAG, p, b, priv_a, priv_b

F = GF(p)
E = EllipticCurve(F, [726, b])
G = E(926644437000604217447316655857202297402572559368538978912888106419470011487878351667380679323664062362524967242819810112524880301882054682462685841995367, 4856802955780604241403155772782614224057462426619061437325274365157616489963087648882578621484232159439344263863246191729458550632500259702851115715803253)

A = G * priv_a
B = G * priv_b

print(A)
print(B)

C = priv_a * B

assert C == priv_b * A

# now use it as shared secret
secret = C[0]

hash = sha256()
hash.update(long_to_bytes(secret))

key = hash.digest()[16:32]
iv = b'u\x8fo\x9aK\xc5\x17\xa7>[\x18\xa3\xc5\x11\x9en'
cipher = AES.new(key, AES.MODE_CBC, iv)

encrypted = cipher.encrypt(pad(FLAG, 16))
print(encrypted)
```

This is a Sage file, which seems to be something used for Math, but that mostly looks like Python.

I also have the output.txt containing the file, and some weird elliptic curves shenanigans.

Let's try to understand lines one by one, to make sure i fully understand !

First, we can see that we don't know some variables that are privates:
```python
from secret import FLAG, p, b, priv_a, priv_b
```
- Flag, obviously our flag
- p, no idea what this is yet
- b, no idea either
- priv_a and priv_b, which are probably some kind of private keys ?


Now this line:
```python
F = GF(p)
```
The GF function defines a finite field named F. The value p is a prime number. GF mean "Gallois Field".

Let's dig into other lines, so we might understand better.
```python
E = EllipticCurve(F, [726, b])
```
So, this creates an Elliptic Curve, over the field F.
This curve will be used to create the private/public key pairs, so it's very important.
I've read the equation of an elliptic curve is `y^2 = x^3 + ax + b`, not sure that this will help me tho.
In this equation, 726 is a, and b... is b.
So we already know one of the value of the equation, maybe we could find a way to bruteforce b to get something ? Maybe.

Let's carry on:
```python
G = E(926644437000604217447316655857202297402572559368538978912888106419470011487878351667380679323664062362524967242819810112524880301882054682462685841995367, 4856802955780604241403155772782614224057462426619061437325274365157616489963087648882578621484232159439344263863246191729458550632500259702851115715803253)
```
This defines a point on the elliptic curve E.
I'm not a math expert, but if we have x, y, and a; couldn't we find b from the above equation ?
Maybe.

A and B seems to be simply the private key times the G coordinates.
```python
A = G * priv_a
B = G * priv_b
```
That's a scalar multiplication (from Wikipedia), probably because these are coordinates. Gosh i really need to learn maths.
A and B are the public keys of Alice and Bob.

Their shared secret C (-another point on the curve-) is obtained either by doing 
```python
C = priv_a * B
```
or
```python
C = priv_B * a
```


Okay so, we need to find that shared secret C.

I'm going to follow my instinct, and try to find `a`.
We know one point of a curve, so that's our `x` and `y`.
We also know the elliptic curve equation: `y^2 = x^3 + ax + b`
And finally, we know `b`, as it is given in the code.

I'm going to write a python script to find a.
After some time, i understood that since our curve is inside a Gallois field, we have to work with modulus...

