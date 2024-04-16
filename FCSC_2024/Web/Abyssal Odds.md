We must find seeds to open all of these boxes:

```typescript
function _openBox(box: LootBox, key: number) {
  const ts = Math.floor(Date.now() / 1000);
  switch (true) {
    case key === box.seed:
      return 1;
    case Math.abs(key - ts) < 60:
      return 2;
    case box.seed % key === 0:
      return 3;
    case Math.cos(key) * 0 !== 0:
      return 4;
    case key && (box.seed * key) % 1337 === 0:
      return 5;
    case key && (box.seed | key) === (box.seed | 0):
      return 6;
    case !(key < 0) && box.seed / key < 0:
      return 7;
    default:
      return 0;
  }
}
```

# Box 1
This one seems hard...
```typescript
    case key === box.seed:
```
I found this in the code:
```typescript
// Patch the app, someone managed to get all the cards from the deck
// I don't know how they did it, but I'm sure it's related to the random number generator
// With this patch, it should be impossible to predict the next number
// I hope this is enough to fix the issue
export default defineEventHandler((event) => {
  const rounds = Math.floor(Math.random() * 100);

  for (let i = 0; i < rounds; i++) {
    Math.random();
  }
});
```
https://blog.securityevaluators.com/hacking-the-javascript-lottery-80cc437e3b7f
Haven't found that one.
# Box 2
```typescript
    case Math.abs(key - ts) < 60:
```
We need to simply get the timestamp, and send a key equal to it +60s.
# Box 3
```typescript
    case box.seed % key === 0:
```
If we use 1 as key value, any positive number will become 0.
# Box 4
```typescript
    case Math.cos(key) * 0 !== 0:
```
If we input Infinity as key, Math.cos will return NaN, which will make NaN * 0 !== 0.
# Box 5
```typescript
    case key && (box.seed * key) % 1337 === 0:
```
Here we input 1337 as key, so we know box.seed * key is a multiple of 1337.
# Box 6
```typescript
    case key && (box.seed | key) === (box.seed | 0):
```
Here we needed to input a value that, if "or"-ed with the seed, give the seed "or"-ed with zero.
So i inputted: 4294967296
Which is `0x100000000` (seed is only 4 bytes long)
That way, this passes the check.
# Box 7
```typescript
    case !(key < 0) && box.seed / key < 0:
```
Inputting -0 passes this check, thanks JavaScript.

For posterity, he's the best JS meme of history:

![](_attachments/Pasted%20image%2020240412173514.png)