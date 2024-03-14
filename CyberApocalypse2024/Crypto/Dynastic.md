We're given a source code and an output txt file.
The source code is pretty straight forward and just encrypt a message using a simple algorithm.

The algorithm is basically doing a caesar cipher, but increasing the offset by 1 on every character.

Here is my decrypt function, and here we are:
```python
def to_identity_map(a):
    return ord(a) - 0x41

def from_identity_map(a):
    return chr(a % 26 + 0x41)

def decrypt(ciphertext):
    decrypted_text = ''
    for i in range(len(ciphertext)):
        ch = ciphertext[i]
        if not ch.isalpha():
            dch = ch
        else:
            cci = to_identity_map(ch)
            dch = from_identity_map(cci - i)
        decrypted_text += dch
    return decrypted_text

def main():
    try:
        with open('output.txt', 'r') as file:
            ciphertext = file.read().strip()

        decrypted_text = decrypt(ciphertext)
        print("Decrypted Text:")
        print(decrypted_text)

    except FileNotFoundError:
        print("output.txt file not found.")

if __name__ == "__main__":
    main()

```

DID_YOU_KNOW_ABOUT_THE_TRITHEMIUS_CIPHER?!_IT_IS_SIMILAR_TO_CAESAR_CIPHER

So the flag is HTB{DID_YOU_KNOW_ABOUT_THE_TRITHEMIUS_CIPHER?!_IT_IS_SIMILAR_TO_CAESAR_CIPHER}

I didn't know the name of Trithemius tho ! :)