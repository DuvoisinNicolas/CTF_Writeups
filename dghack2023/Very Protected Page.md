We can see in the page code that it uses AES with CTR mode, and we can also see 2 cyphertexts:

A very long one, starting with:

`34aff6de8f8c01b25c56c52261e49cbddQsBGjy+uKhZ7z3+zPhswKWQHMYJpz7wffAe4Es/bwrJmMo99Kv7XJ8P63TbN/8X`

And a short one, which is :

`34aff6de8f8c01b25c56c52261e49cbdC19FW3jqqqxd6G/z0fcpnOSIBsUSvD+jZ7E9/VkscwDMrdk9i9efIvJw1Fj6Fs0R`

We can see that the 32 first chars are the same !

We decode both strings since they are in base 64, and we get the following ciphertexts in hex:

`df869f7fa75ef1ff1cd356f6e5ce7a739db6eb57b8f5c6dd750b011a3cbeb8a859ef3dfeccf86cc0a5901cc609a73ef07df01ee04b3f6f0ac998ca3df4abfb5c9f0feb74db37ff17`
`df869f7fa75ef1ff1cd356f6e5ce7a739db6eb57b8f5c6dd0b5f455b78eaaaac5de86ff3d1f7299ce48806c512bc3fa367b13dfd592c7300ccadd93d8bd79f22f270d458fa16cd11`

We xor them together, because IV and key were the same, that will give us the XOR from both plaintext.

`0000000000000000000000000000000000000000000000007E544441445412040407520D1D0F455C41181A031B1B01531A41231D12131C0A053513007F7C647E6D7F3F2C21213206`

Without the zeros and the IVs:

Ciphers:

Long one : `750b011a3cbeb8a859ef3dfeccf86cc0a5901cc609a73ef07df01ee04b3f6f0ac998ca3df4abfb5c9f0feb74db37ff17`

Short one: `0b5f455b78eaaaac5de86ff3d1f7299ce48806c512bc3fa367b13dfd592c7300ccadd93d8bd79f22f270d458fa16cd11`

Xor of both plaintexts:

`7E544441445412040407520D1D0F455C41181A031B1B01531A41231D12131C0A053513007F7C647E6D7F3F2C21213206`


This is the XOR of both plaintexts. Now we need to identify one of the plaintexts, and that will give us the other one.

The very long one seems to be a file that's decrypted, maybe HTML page ? Let's XOR the HTML page with our item. That didn't work.

But at the beginning of the javscript code, we can see the string `Build with love, kitties and flowers`.

Tried to XOR with it...

And we get:

`<!-- temporary password : My2uperPas`

Time to bruteforce !!!

I used this program (thank you chatGPT for turning my disgusting code to this one):
```
{
const baseStr = "My2uperPas";
// const letters = [
//   "sS$5",
//   "wW",
//   "oO0@",
//   "rR5",
//   "dD5",
//   "!#$%-:;?()"
// ];

const letters = [
    "sS$5",
    "pP",
    "hH#",
    "rR5",
    "aA@4",
    "sS$5",
    "eE3"
  ];

function generatePasswordsRecursive(currentPassword, currentIndex) {
  if (currentIndex === letters.length) {
    console.log(currentPassword);
    document.querySelector("#crypt-password").value = currentPassword;
    document.querySelector(".crypt-decrypt-button").click();
    return;
  }

  const currentLetterSet = letters[currentIndex];

  for (let i = 0; i < currentLetterSet.length; i++) {
    const newPassword = currentPassword + currentLetterSet[i];
    generatePasswordsRecursive(newPassword, currentIndex + 1);
  }
}

function generatePasswords() {
  generatePasswordsRecursive(baseStr, 0);
}

generatePasswords();
}
```

I ran it in my browser console, and got the flag !

Password was `My2uperPassphras3`, and flag was `DGHACK{w3ak_pa22word2_ar3n-t_n3at` !

My first crypto chall, kinda happy on that one :)
