# Reversing WASM in a nutshell

For this prechall, we are given a Dino game just like in Chrome.
I saw this code on the page:
```javascript
<script>
  // Prechall Game
  fetch('files/teasing/fcsc2024-dino.wasm')
    .then(response => response.arrayBuffer())
    .then(bytes => WebAssembly.instantiate(bytes, {Math}))
    .then(source => {
      let instance = source.instance;
      let canvasData = new Uint8Array(instance.exports.mem.buffer, 0x3000, 90000);
      let canvas = document.querySelector('canvas');
      let context = canvas.getContext('2d');
      let imageData = context.createImageData(300, 75);
```

I downloaded this file `files/teasing/fcsc2024-dino.wasm`, and decompiled it by using internet.
I got an ugly WAT file, mostly unreadable.

After that, i played with the program a little on Chrome debugger, and i saw that there was a part of the code that the program never reached.

```asm
    i32.lt_u
    if
      i32.const 5
      i32.const 5
      i32.const 33
      i32.const 33
      i32.const -1409286144
      i32.const 1260
      call $g
      drop
    end
  )
```
The program was never entering this "if" on each iteration.
It was then calling the function $g, which wasn't called neither.
Since that might be the function printing the flag, i decided to modify it.

I didn't find directly how to edit the code from the debugger, so i captured the request using Burpsuite:
![](_attachments/Pasted%20image%2020240402211801.png)
Once i got this request, i asked for burp to catch the response right after.
![](_attachments/Pasted%20image%2020240402211830.png)

Then i got the file in burp.
I turned on the "hex" view.

And now, i looked what were the opcodes for the code around where i was:
1. if: 0x04
2. less than (i32.lt_u): 0x49 
3. greater than (i32.gt_u): 0x4b 
4. i32.const: 0x41 (modifié)

So i looked in burp, and i found this line:
00000430: 4180 cab5 ee01 2301 **4904** 4041 0541 0541 A.....#.I.@A.A.A

49 04, which are "if" and "less than", followed by multiple "data" + "0x41", which corresponds to the multiples i32.const.

So finally, i edited the 49 value (less than), to turn it into 4b (greater than) to reverse the program behaviour.

And i got a pretty QR code:
![](_attachments/Pasted%20image%2020240402212307.png)
Or... did i ?

# Part 2: The code...

Gosh, there are multiple parts !
I followed the QRCode:
[https://france-cybersecurity-challenge.fr/bded79a9edb3e51a1d752c5ea6dfae1a](https://france-cybersecurity-challenge.fr/bded79a9edb3e51a1d752c5ea6dfae1a "https://france-cybersecurity-challenge.fr/bded79a9edb3e51a1d752c5ea6dfae1a")

And i ended up on a page, asking me for the input for this code:

```python
from hashlib import sha256

CHARSET = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz!?€$*"
L1 = "$ABFIZht!HPUYhirKOXdhjxy?DGJSWehCENahklo*cghmpu€MQRVfhvzLTbhnqsw"
L2 = "!FOWamwzFRSTUdlp*EFVXbei?FMNPgjsCDFHcnvxFJQYkqy€FGKLforu"
L3 = "!FOWamwz$RcejkrwDIKNVYpwGHMXltw€*APSovwy?BCQdiuwEJUZfgwx"

def check(A, B):
    res = True
    for a in range(7):
        for b in range(7):
            S = set(A[a])
            for x in range(7):
                y = (a * x + b) % 7
                S = S.intersection(set(B[y][x]))
            res &= (len(S) == 1)
    for x in range(7):
        S = set(A[7])
        for y in range(7):
            S = S.intersection(set(B[y][x]))
        res &= (len(S) == 1)
    return res

s = input(">>> ")

assert len(s) == 8 * (8 * 7 + 1)
assert all(x in CHARSET for x in s)

s1, s2 = s[:8 * 8], s[8 * 8:]
A = [ s1[j:j + 8] for j in range(0, 8 * 8, 8) ]
B = [ [s2[i:i + 8 * 7][j:j + 8] for j in range(0, 8 * 7, 8) ] for i in range(0, len(s2), 8 * 7)]

assert L1 == "".join(A)
assert L2 == "".join([ B[0][i] for i in range(7) ])
assert L3 == "".join([ B[i][0] for i in range(7) ])
assert all([ sorted(B[i][j]) == list(B[i][j]) for i in range(7) for j in range(7) ])
assert check(A, B)

h = sha256(s.encode()).hexdigest()
print(f"Congrats! You can now go here: https://france-cybersecurity-challenge.fr/{h}")
```

To be honest, i don't see how i can find the input "easily" in that code.
The right approach is probably to use maths and solvers, and i don't know anything about maths..

I tried for an entire day multiple solvers (z3, sympy, compiling to C and then using angr...) but no way to make it work.
So i decided to try to solve the program by myself !

So i've done this code:
```python
from hashlib import sha256


CHARSET = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz!?€$*"
L1 = "$ABFIZht!HPUYhirKOXdhjxy?DGJSWehCENahklo*cghmpu€MQRVfhvzLTbhnqsw"
L2 = "!FOWamwzFRSTUdlp*EFVXbei?FMNPgjsCDFHcnvxFJQYkqy€FGKLforu"
L3 = "!FOWamwz$RcejkrwDIKNVYpwGHMXltw€*APSovwy?BCQdiuwEJUZfgwx"

# Initialize basic string
s = "$ABFIZht!HPUYhirKOXdhjxy?DGJSWehCENahklo*cghmpu€MQRVfhvzLTbhnqsw!FOWamwzFRSTUdlp*EFVXbei?FMNPgjsCDFHcnvxFJQYkqy€FGKLforu"
L3_s= [L3[x:x+8] for x in range(0,len(L3),8)]

for elem in L3_s[1:]:
    # Appending header
    s += elem
    # Appending 6 empty 8 chars bytes (@ not in charset for debugging)
    s += "@"*8*6


s1, s2 = s[:8 * 8], s[8 * 8:]
A = [ s1[j:j + 8] for j in range(0, 8 * 8, 8) ]
B = [ [s2[i:i + 8 * 7][j:j + 8] for j in range(0, 8 * 7, 8) ] for i in range(0, len(s2), 8 * 7)]

print(len(s))
assert len(s) == 8 * (8 * 7 + 1)
assert L1 == "".join(A)
assert L2 == "".join([ B[0][i] for i in range(7) ])
print("".join([ B[i][0] for i in range(7) ]))

assert L3 == "".join([ B[i][0] for i in range(7) ])

for elem in A:
    print(elem, sep="", end="\t")

for row in B:
    print("\n", sep="", end="")
    for block_of_8 in row:
        print(block_of_8, sep="", end="\t")

```
The goal is to remove the logic behind L1, L2 and L3.
All assert works, meaning this part is correct.

This is what the program outputs:
```
$ABFIZht        !HPUYhir        KOXdhjxy        ?DGJSWeh        CENahklo        *cghmpu€        MQRVfhvz        LTbhnqsw
!FOWamwz        FRSTUdlp        *EFVXbei        ?FMNPgjs        CDFHcnvx        FJQYkqy€        FGKLforu
$Rcejkrw        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@
DIKNVYpw        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@
GHMXltw€        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@
*APSovwy        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@
?BCQdiuw        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@
EJUZfgwx        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@        @@@@@@@@
```

So now, we have 2 remaining things to do:
- Sort all elements
- Apply the "check" logic.
Since most of the fields are placeholders, I'll apply the check logic first.

In order to understand what "check" does, i will replace the placeholders with numbers.

Here are the sets that seems to be compared:
```
$ABFIZht        !HPUYhir        KOXdhjxy        ?DGJSWeh        CENahklo        *cghmpu€        MQRVfhvz        LTbhnqsw
00000000        66666666        55555555        44444444        33333333        22222222        11111111
11111111        00000000        66666666        55555555        44444444        33333333        22222222
22222222        11111111        00000000        66666666        55555555        44444444        33333333
33333333        22222222        11111111        00000000        66666666        55555555        44444444
44444444        33333333        22222222        11111111        00000000        66666666        55555555
55555555        44444444        33333333        22222222        11111111        00000000        66666666
66666666        55555555        44444444        33333333        22222222        11111111        00000000
```
So we have to fill 7 values of 8 characters, so that's 56.
Also, our charset is 59 char long, so we have enough chars.
But, we have to use only 1 letter from the str inside "A".
Since they are 8 bytes long, that's impossible.
Ahem.
Wait, no.
That's wrong.

The goal is to have len(Intersect("$ABFIZht", "00000000" "00000000" "00000000" "00000000" "00000000" "00000000" "00000000")) == 1.

That means that we just have to use different characters in every slot. Except for one.
So for example, that should work:
"$ABFIZht", "ACCCCCC" "ACCCCCC" "ACCCCCC" "ACCCCCC" "ACCCCCC" "ACCCCCC" "ACCCCCC"

But that's only 1 loop. We have to do this for every value in A.
Let's try this logic:
We add the a-th A[a] value to our string.

I got something like this
```
$ABFIZht        !HPUYhir        KOXdhjxy        ?DGJSWeh        CENahklo        *cghmpu€        MQRVfhvz        LTbhnqsw
$!K?C*M0        $ijSagQ1        $hdDlmR2        $YOWNuV3        $UxGkcf4        $PheEhh5        $HXJhpv6
AHODEcQ1        A!xWhhR2        AihGCpV3        AhXea*f4        AYKJlgh5        AUj?Nmv6        APdSkuM0
BPXGNgR2        BHKekmV3        B!jJEuf4        Bid?hch5        BhOSChv6        BYxDapM0        BUhWl*Q1
FUdJahV3        FPO?lpf4        FHxSN*h5        F!hDkgv6        FiXWEmM0        FhKGhuQ1        FYjeCcR2
IYhShmf4        IUXDCuh5        IPKWacv6        IHjGlhM0        I!deNpQ1        IiOJk*R2        Ihx?EgV3
ZhjWkph5        ZYdGE*v6        ZUOehgM0        ZPxJCmQ1        ZHh?auR2        Z!XSlcV3        ZiKDNhf4
hixeluv6        hhhJNcM0        hYX?khQ1        hUKSEpR2        hPjDh*V3        hHdWCgf4        h!OGamh5
```

That passes the first part of "check".
But i overwrite the first column and the second row, that needs to stay the same to pass the first asserts.

So i used a different approach:
- First, check all "fixed" values during the loop (first B row, and first B column)
- Find their intersection with "A[a]"
- Append this intersection to the loop's fields
- Sort all fields
- Get the flag !

And that worked, after a day of programming, printing, debugging...
Here is my final code (sorry, it's not quite beautiful...):
```python
from hashlib import sha256

def check(A, B):
    res = True
    for a in range(7):
        for b in range(7):
            S = set(A[a])
            for x in range(7):
                y = (a * x + b) % 7
                S = S.intersection(set(B[y][x]))
            res &= (len(S) == 1)
    for x in range(7):
        S = set(A[7])
        for y in range(7):
            S = S.intersection(set(B[y][x]))
        res &= (len(S) == 1)
    return res



CHARSET = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz!?€$*"
L1 = "$ABFIZht!HPUYhirKOXdhjxy?DGJSWehCENahklo*cghmpu€MQRVfhvzLTbhnqsw"
L2 = "!FOWamwzFRSTUdlp*EFVXbei?FMNPgjsCDFHcnvxFJQYkqy€FGKLforu"
L3 = "!FOWamwz$RcejkrwDIKNVYpwGHMXltw€*APSovwy?BCQdiuwEJUZfgwx"

# Initialize basic string
s = "$ABFIZht!HPUYhirKOXdhjxy?DGJSWehCENahklo*cghmpu€MQRVfhvzLTbhnqsw!FOWamwzFRSTUdlp*EFVXbei?FMNPgjsCDFHcnvxFJQYkqy€FGKLforu"
L3_s= [L3[x:x+8] for x in range(0,len(L3),8)]

for elem in L3_s[1:]:
    # Appending header
    s += elem
    # Appending 6 empty 8 chars bytes (@ not in charset for debugging)
    s += "@"*8*6


s1, s2 = s[:8 * 8], s[8 * 8:]
A = [ s1[j:j + 8] for j in range(0, 8 * 8, 8) ]
B = [ [s2[i:i + 8 * 7][j:j + 8] for j in range(0, 8 * 7, 8) ] for i in range(0, len(s2), 8 * 7)]

def beautiful_print():
    for elem in A:
        print(elem, sep="", end="\t")

    for row in B:
        print("\n", sep="", end="")
        for block_of_8 in row:
            print(block_of_8, sep="", end="\t")
    
    print("\n", sep="", end="")



print(len(s))
assert len(s) == 8 * (8 * 7 + 1)
assert L1 == "".join(A)
assert L2 == "".join([ B[0][i] for i in range(7) ])
assert L3 == "".join([ B[i][0] for i in range(7) ])


# Try to fill the placeholders with good values
# empty placeholders
for a in range (7):
    for b in range (7):
        if B[a][b] == "@@@@@@@@":
            B[a][b] = ""

# Fill all values except last:
for a in range(7):
    for b in range (7):
        S = set(A[a])
        intersect = ""
        for x in range(7):
            y = (a * x + b) % 7
            if len(B[y][x]) == 8 and (x==0 or y==0):
                #print("The fixed value for", A[a], "is",B[y][x])
                # Find the character we have to add in this loop
                intersect = S.intersection(B[y][x])
                #print("intersect:", intersect)
        # Add intersect in every empty field
        for x in range (7):
            y = (a * x + b) % 7
            if len(B[y][x]) < 8:
                B[y][x] += "".join(intersect)
                #print("appending", intersect)

print("")
print("pre-filling done:")
print("")
beautiful_print()

# Fill the last value:
for x in range(7):
    S = set(A[7])
    intersections = set()
    for y in range (7):
        intersect = S.intersection(B[y][x])
        #print(intersect)
        if len(intersect) != 0:
            intersections.add("".join(intersect))
    #print("final intersect:",intersections)
    # Now that we found intersect, add the same value after each element so intersect doesn't change
    for y in range (7):
        if len(B[y][x]) != 8:
            B[y][x] += "".join(intersections)

print("")
print("filling done:")
print("")
beautiful_print()

# Fill last valyes
assert check(A, B)

# now we just have to sort and rebuilding s to get the hash
s = ""
for elem in A:
    s += "".join(sorted(elem))

for elem in B:
    for elem2 in elem:
        s += "".join(sorted(elem2))
print(s)

h = sha256(s.encode()).hexdigest()
print(f"Congrats! You can now go here: https://france-cybersecurity-challenge.fr/{h}")
```

Program output:
```
Congrats! You can now go here: https://france-cybersecurity-challenge.fr/b476ca7cd889ee0d2de7db528f2cc094329be7da417d10a24a808997825fff90
```

Here is my flag !!
And:
![](_attachments/Pasted%20image%2020240404115714.png)

That was probably the hardest programing challenge i've ever done.
The dino part was also new to me, but that wasn't too hard hopefully.
Can't wait of the real challenges !