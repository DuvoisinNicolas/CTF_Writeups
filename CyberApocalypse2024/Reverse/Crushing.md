After many time reading the code on Ghidra, it seems like the program is rearanging the input file to a map, containing characters, and counting the number of each occurence, and storing their position.

After that, it writes its "map" on a file.

I decided to do some tests to understand it better.

Here are some outputs i've got:
```bash
$echo -n 'aaa' | ./crush | hexdump 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
0000300 0000 0000 0000 0000 0003 0000 0000 0000
0000310 0000 0000 0000 0000 0001 0000 0000 0000
0000320 0002 0000 0000 0000 0000 0000 0000 0000
0000330 0000 0000 0000 0000 0000 0000 0000 0000
*
0000810
```

Our 'a' character got stored in the file.
First, on `0000308`, we can see `03`, meaning that we stored our letter A 3 times.
On `0000310`, we can see ` 0000 [...] 0001 [...]`
Same goes for the next line, with `0002 [...]`.

Let's try to add some B letters after.
```bash
$echo -n 'aaabb' | ./crush | hexdump 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
0000300 0000 0000 0000 0000 0003 0000 0000 0000
0000310 0000 0000 0000 0000 0001 0000 0000 0000
0000320 0002 0000 0000 0000 0002 0000 0000 0000
0000330 0003 0000 0000 0000 0004 0000 0000 0000
0000340 0000 0000 0000 0000 0000 0000 0000 0000
*
0000820
```

Same than before, except that we also see `0002` on `0000328`; then followed by `0003` and `0004`.

It looks like that contains the position on the value in the message.

Let's now mix letters up, to see how it works.

```bash
echo -n 'bbaaabb' | ./crush | hexdump 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
0000300 0000 0000 0000 0000 0003 0000 0000 0000
0000310 0002 0000 0000 0000 0003 0000 0000 0000
0000320 0004 0000 0000 0000 0004 0000 0000 0000
0000330 0000 0000 0000 0000 0001 0000 0000 0000
0000340 0005 0000 0000 0000 0006 0000 0000 0000
0000350 0000 0000 0000 0000 0000 0000 0000 0000
*
0000830
```
Okay now, we see that the letters already starts with `0003`, because there are 3 letters A.
After that, we see that there is not `0000` anymore, but `0002`.
This is because A isn't at pos 0 anymore, but now it's pos 2.

If we go down, at the start of the B listing, we can see length is 0004.
After that, we can see 0000, 0001, 0005, and 0006.
That means that B is on pos 0,1,5, and 6 ! 
Great :)

I think i got this.
Except for one part : when i use `xxd` on the original file...
```bash
$ hexdump message.txt.cz                                       
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
0000050 000c 0000 0000 0000 0049 0000 0000 0000
0000060 004a 0000 0000 0000 008e 0000 0000 0000
0000070 008f 0000 0000 0000 0119 0000 0000 0000
0000080 011a 0000 0000 0000 01b3 0000 0000 0000
0000090 01b4 0000 0000 0000 022f 0000 0000 0000
```

It does start at address `0000050` instead of `0000300`.
That might mean that there's a special character involved.
I'm going to try to put numbers, since they might be in the flag.

```bash
$ echo -n '0123456789bbaaabb' | ./crush | hexdump 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
0000180 0001 0000 0000 0000 0000 0000 0000 0000
0000190 0001 0000 0000 0000 0001 0000 0000 0000
00001a0 0001 0000 0000 0000 0002 0000 0000 0000
00001b0 0001 0000 0000 0000 0003 0000 0000 0000
00001c0 0001 0000 0000 0000 0004 0000 0000 0000
00001d0 0001 0000 0000 0000 0005 0000 0000 0000
00001e0 0001 0000 0000 0000 0006 0000 0000 0000
00001f0 0001 0000 0000 0000 0007 0000 0000 0000
0000200 0001 0000 0000 0000 0008 0000 0000 0000
0000210 0001 0000 0000 0000 0009 0000 0000 0000
0000220 0000 0000 0000 0000 0000 0000 0000 0000
*
0000350 0000 0000 0000 0000 0003 0000 0000 0000
0000360 000c 0000 0000 0000 000d 0000 0000 0000
0000370 000e 0000 0000 0000 0004 0000 0000 0000
0000380 000a 0000 0000 0000 000b 0000 0000 0000
0000390 000f 0000 0000 0000 0010 0000 0000 0000
00003a0 0000 0000 0000 0000 0000 0000 0000 0000
*
0000880
```
It starts at 0000180. 0 is 30 in hex, and 30 times 8 is 180 (in hex). 

So the first character being 50, it's probably 50/8 in hex, which is 10.
10 in ascii table is a carriage return, that's why it is so low.

Let's try if it works...
Say the flag is in the format `HTB`, let's see if we find these characters follows each other.

We could try to decode it now manually but that would be too long. Let's try to do it in python !

...

After many time, i only managed to retreive a list of positions of each character.

```python
import json

# Open the file
with open("rev_crushing/hexdump.txt", "r") as file:
    # Read the content of the file and remove newline characters
    content = file.read().replace('\n', '')

# Split the content into pairs of 4 letters (2 bytes)
pairs = [content[i:i+4] for i in range(0, len(content), 4)]

# Convert pairs to little-endian format
little_endian_pairs = [pair[2:] + pair[:2] for pair in pairs]

# Create a dictionary with hexadecimal positions as keys and pairs as values
hex_dict = {int(i * 8): int(pair, 16) for i, pair in enumerate(little_endian_pairs[::4])}

# Sort the dictionary keys by hexadecimal position
sorted_keys = sorted(hex_dict.keys())

# Pretty print the first 1000 elements ordered by hexadecimal position
print("Dictionary with hexadecimal positions as keys (every 4th element, ordered by hex order):")
ordered_hex_dict = {key: hex_dict[key] for key in sorted_keys[:1000]}
print(json.dumps(ordered_hex_dict, indent=4))

# Initialize the pos_values dictionary
pos_values = {}

# Variable to keep track of the current position
current_pos = None




# Get the sorted keys
sorted_keys = sorted(hex_dict.keys())

# Iterate over the sorted keys
i = 0
while i < len(sorted_keys):
    key = sorted_keys[i]
    value = hex_dict[key]

    # If the value is non-zero
    if value != 0:
        # Set the current position to the key
        current_pos = key
        # Create a new key in pos_values if it doesn't exist
        if current_pos not in pos_values:
            pos_values[current_pos] = []

        # Skip the next 'value' elements and store them in pos_values
        for j in range(value):
            i += 1
            pos_values[current_pos].append(hex_dict[sorted_keys[i]])

        # Move the current position to after the skipped elements
        current_pos = sorted_keys[i]
        i += 1
        
    else:
        # If the value is zero, move to the next iteration
        i += 1

# Convert index values of pos_values to character representation
char_pos_values = {}
for index, values in pos_values.items():
    char_pos_values[chr(int(index/8))] = values
print(char_pos_values)

string_val = "x" * 850
list_val = list(string_val)
for key, value in char_pos_values.items():
    for value2 in value:
        list_val[value2] = key
print("".join(list_val))

print("Dictionary 'pos_values':", char_pos_values)

```

Afterwards, i didn't manage to find how to identify each character.
So i've print all the elements inside an array, and tried some smart manual bruteforce.

For example, i've used the frequency of letters in english: for the character that we've seen the most, i've put 'e'.

I ended up with text like this one:

```
xrganixer xx xeyx did you finalixe the password for the nextxxx you knowx

xrganixer xx xeahx x didx xtxs xxxxxxxxxryxbxdxcomprxssxonxschxmxx

xrganixer xx xxxxxxxxxryxbxdxcomprxssxonxschxmxxx got itx xounds ominous enough to keep things interestingx xhere do we spread the wordx

xrganixer xx xetxs stick to the usual channelsx encrypted messages to the leaders and discreetly slip it into the training manuals for the participantsx

xrganixer xx xerfectx xnd letxs make sure itxs not leaked this timex xast thing we need is an early bird getting the wormx

xrganixer xx xgreedx xe canxt afford any slipxupsx especially with the stakes so highx xhe anticipation leading up to it should be palpablex

xrganixer xx xbsolutelyx xhe thrill of the unknown is what keeps them coming back for morex xxxxxxxxxryxbxdxcomprxssxonxschxmxx it is thenxxxxxxxx
```

And now i just have to replace the missing letters (X was my placeholder), and get the final text !

Here is my final code, with guessed characters !

```python
char_pos_values = {
 '\n': [73, 74, 142, 143, 281, 282, 435, 436, 559, 560, 701, 702],
 ' ': [9, 12, 17, 21, 25, 34, 38, 47, 51, 55, 63, 67, 84, 87, 93, 95, 100, 105, 153, 156, 194, 198, 202, 209, 217, 224, 227, 232, 239, 252, 258, 261, 264, 271, 275, 292, 295, 301, 307, 310, 314, 320, 330, 340, 349, 352, 356, 364, 368, 379, 384, 387, 392, 396, 405, 413, 417, 421, 446, 449, 458, 462, 468, 473, 478, 483, 487, 494, 499, 505, 510, 516, 519, 524, 527, 530, 536, 541, 549, 553, 570, 573, 581, 584, 590, 597, 601, 611, 622, 627, 631, 638, 641, 647, 651, 664, 672, 675, 678, 681, 688, 691, 712, 715, 727, 731, 738, 741, 745, 753, 756, 761, 767, 772, 779, 784, 788, 794, 831, 834, 837],
 '': [106, 141, 157, 193, 795, 830], 
 '\'': [103, 299, 466, 481, 588], 
 '!': [16, 92, 192, 610], 
 '-': [606], 
 '.': [60, 61, 62, 99, 201, 251, 434, 457, 504, 558, 580, 646, 700, 726, 793, 842],
 '1': [10, 130, 154, 181, 447, 713, 819], 
 '2': [85, 293, 571], 
 '3': [114, 127, 137, 139, 165, 178, 188, 190, 803, 816, 826, 828], 
 '4': [111, 119, 162, 170, 800, 808], 
 ':': [11, 86, 155, 294, 329, 448, 572, 714], 
 '?': [72, 280], 
 'A': [459, 574, 716], 
 'B': [109, 160, 798], 
 'H': [13, 107, 158, 796], 
 'I': [94, 101], 
 'L': [296, 506], 
 'O': [0, 75, 144, 283, 437, 561, 703], 
 'P': [450], 
 'S': [203], 
 'T': [108, 159, 648, 728, 797], 
 'W': [253, 582], 
 'Y': [88], 
 '_': [112, 117, 121, 133, 163, 168, 172, 184, 801, 806, 810, 822], 
 'a': [3, 29, 40, 78, 90, 147, 269, 286, 318, 323, 345, 359, 365, 399, 407, 410, 423, 430, 440, 470, 490, 507, 528, 532, 564, 586, 591, 598, 618, 634, 652, 659, 667, 693, 696, 706, 759, 781], 
 'b': [118, 169, 537, 689, 697, 717, 780, 807], 
 'c': [122, 135, 173, 186, 305, 321, 333, 372, 427, 455, 585, 616, 656, 773, 782, 811, 824], 
 'd': [18, 20, 46, 96, 98, 120, 171, 207, 259, 270, 279, 339, 360, 367, 369, 461, 493, 523, 540, 579, 596, 668, 687, 809], 
 'e': [7, 14, 33, 37, 54, 57, 82, 89, 151, 218, 229, 230, 243, 245, 255, 257, 263, 268, 274, 290, 297, 313, 326, 331, 338, 342, 347, 355, 358, 361, 374, 375, 395, 420, 444, 451, 454, 464, 472, 477, 489, 492, 503, 518, 521, 522, 531, 543, 552, 568, 577, 578, 583, 612, 615, 630, 636, 650, 666, 690, 699, 710, 723, 730, 744, 763, 764, 770, 792, 840], 
 'f': [26, 48, 414, 453, 592, 593, 740, 785], 
 'g': [2, 77, 146, 195, 222, 237, 250, 285, 346, 404, 439, 515, 542, 548, 563, 575, 644, 671, 705, 778], 
 'h': [36, 53, 91, 136, 187, 223, 234, 254, 273, 312, 322, 354, 394, 419, 496, 512, 551, 626, 629, 642, 645, 649, 683, 729, 733, 743, 758, 769, 825, 839], 
 'i': [5, 19, 27, 31, 80, 97, 149, 199, 212, 235, 240, 248, 288, 304, 370, 382, 385, 388, 400, 402, 426, 428, 442, 479, 497, 501, 513, 525, 538, 546, 566, 604, 617, 624, 643, 655, 657, 661, 669, 679, 708, 735, 754, 776, 832, 835], 
 'k': [68, 228, 306, 471, 491, 635, 748, 762, 783], 
 'l': [30, 319, 327, 357, 377, 381, 411, 463, 488, 534, 603, 619, 620, 665, 686, 694, 698, 720, 724, 736, 737], 
 'm': [124, 138, 175, 189, 211, 341, 406, 469, 502, 557, 771, 775, 789, 813, 827], 
 'n': [4, 28, 56, 69, 79, 132, 148, 183, 206, 213, 219, 236, 241, 249, 287, 324, 325, 332, 366, 389, 401, 403, 408, 431, 441, 460, 484, 514, 520, 529, 547, 565, 587, 599, 653, 663, 670, 707, 747, 749, 752, 777, 821, 841], 
 'o': [23, 44, 49, 65, 70, 123, 131, 174, 182, 196, 204, 210, 214, 220, 226, 260, 277, 309, 351, 391, 415, 485, 555, 594, 640, 662, 677, 684, 719, 739, 750, 774, 786, 790, 812, 820], 
 'p': [39, 125, 176, 231, 266, 336, 383, 422, 429, 605, 608, 614, 658, 674, 692, 695, 765, 814], 
 'r': [1, 8, 45, 50, 76, 83, 115, 126, 145, 152, 166, 177, 244, 256, 267, 278, 284, 291, 334, 362, 373, 398, 416, 424, 438, 445, 452, 476, 533, 539, 556, 562, 569, 576, 595, 704, 711, 734, 787, 791, 804, 815], 
 's': [41, 42, 104, 128, 129, 134, 179, 180, 185, 208, 216, 238, 246, 265, 300, 302, 316, 328, 343, 344, 348, 363, 371, 380, 412, 433, 467, 474, 482, 498, 508, 526, 602, 609, 613, 632, 637, 639, 682, 718, 755, 766, 817, 818, 823, 836], 
 't': [35, 52, 59, 102, 197, 200, 225, 233, 242, 247, 272, 298, 303, 308, 311, 337, 350, 353, 376, 386, 390, 393, 397, 418, 425, 432, 456, 465, 480, 486, 495, 500, 509, 511, 544, 545, 550, 589, 625, 628, 633, 654, 660, 676, 680, 722, 732, 742, 760, 768, 833, 838], 
 'u': [24, 66, 205, 215, 221, 315, 317, 409, 475, 607, 673, 685, 721, 746], 
 'v': [113, 164, 802], 
 'w': [43, 71, 262, 276, 517, 554, 623, 751, 757], 
 'x': [58], 
 'y': [15, 22, 64, 116, 167, 335, 378, 535, 600, 621, 725, 805], 
 'z': [6, 32, 81, 150, 289, 443, 567, 709], 
 '{': [110, 161, 799], 
 '}': [140, 191, 829]}

string_val = "x" * 850
list_val = list(string_val)
for key, value in char_pos_values.items():
    for value2 in value:
        list_val[value2] = key
print("".join(list_val))

#print("Dictionary 'pos_values':", char_pos_values)
```

Not the best approach... Can't wait to see a writeup to see where i was wrong !

Flag is : `HTB{4_v3ry_b4d_compr3ss1on_sch3m3}`

