---
description: by MisterPine
---

# \[GPNCTF 2026] misc/organized

For GPNCTF 2026, teams can self designate as **Organic**. These are teams that commit to AI abstinence. Our team decided to try this, with more experienced players replacing AI to give guidance and hints to junior members.

Knowing that I wouldn't be able to rely on AI, I picked the category I am the most familiar with – stenography. We are given a binary file with the following description.

> Isn't this just a file of random data? Well, maybe you just don't appreciate the organization in your life.&#x20;
>
> [Download ](https://gpn24.ctf.kitctf.de/api/challenges/handout/organized)

Besides being judged on how I live my life, the text offers almost no useful details. So we went straight to the binary file.

First thing we did was `binwalk -a` . It is quick way to check if there were any hidden files inside the file.

<figure><img src=".gitbook/assets/Screenshot 2026-06-21 at 5.08.59 PM.png" alt=""><figcaption></figcaption></figure>

The result was interesting. It seems binwalk detected multiple GPG signed and Zlib files in the binary file. However the file sizes seem unusually small.

To further investigate, we installed a hex editor called **imHex**, as recommended by one of the experienced players.

imHex is a powerful tool that allows you to write your own patterns and filters. It was my first time using it, so it took me a while to get used to its Rust and C-style hybrid scripting language.

After a quick tutorial and help from teammates, I started inspecting the memory addresses where the supposed GPG and zlib files were located.

That was disappointing. Aside from the magic numbers, these byte sequences did not look anything like actual GPG or zlib files.

<figure><img src=".gitbook/assets/Screenshot 2026-06-21 at 6.53.02 PM.png" alt=""><figcaption></figcaption></figure>

This was where I learned that binwalk is not always reliable. It mainly scans the binaries for known magic numbers, so random data can sometimes resemble a file signautre purely by chance. In this case, because zlib files often begin with `78 01`, `78 5E`, `78 9C`, `78 DA` , binwalk reported a false positive. In order words, just because binwalk reports a possible embedded file does not mean there is actually a valid file there.

Focusing back on the binary file itself, we noticed two things.

1. The file is exactly **7,650,000 bytes**.
2. There are many `0x00` spread throughout the file.&#x20;

At first, we thought it might be an ELF file. However after inspecting the beginning, middle and end of the file, we could not find anything that looks like a valid ELF file structure. What we found instead was that there are chunks with many zeroes and that there are chunks where there are mostly random nonzeroes. So we tried splitting the file into chunks to see whether any structure would emerge. Since the file size was exactly 7,650,000 bytes, we tried using chunk sizes with from its factors. After some trials and errors, the size that divided the file most cleanly is 12,500 bytes. Then we wrote a script to count the zeroes in each chunk.

```python
with open("data", "rb") as f:
    num_chunks = 612 # 7650000 / 12500 = 612
    offset = 0
    for i in range(num_chunks):
        chunk = f.read(7650000//num_chunks)
        if not chunk:
            break
        zeroes = chunk.count(0x00)
        total = len(chunk)
        non_zeroes = total - zeroes
        offset += total
        print(f"Chunk {i}: {zeroes} zeroes, {non_zeroes} non-zeroes, total {total}, offset {offset}")
```

And here are some of the results.

```
Chunk 12: 5337 zeroes, 7163 non-zeroes, total 12500, offset 162500
Chunk 13: 5447 zeroes, 7053 non-zeroes, total 12500, offset 175000
Chunk 14: 53 zeroes, 12447 non-zeroes, total 12500, offset 187500
Chunk 15: 5386 zeroes, 7114 non-zeroes, total 12500, offset 200000
Chunk 16: 52 zeroes, 12448 non-zeroes, total 12500, offset 212500
Chunk 17: 5377 zeroes, 7123 non-zeroes, total 12500, offset 225000
Chunk 18: 5308 zeroes, 7192 non-zeroes, total 12500, offset 237500
```

The pattern is now obvious that the chunks could be divided into two types and parsed as binary code.

1. Chunks with \~50 zeroes. **(0 in binaries)**
2. Chunks with \~5500 zeroes. **(1 in binaries)**

And here is the result.

```
1110001110001101011100111100011101011111110101011110001101011100
1111010111110101010111100111010111001000010111110100010111111011
0101110111110101111000110101110010110101110000010101110110000101
1100001001011101010101011100000101011111010101011100001001011100
0001010111010110010111100010010111110100010111101100010111000010
0101111111000101110110100101110111100101110000010101111001100101
1100001101011110110101011100000101011100001101011110110001011110
0100110111011110010111100011010111011100110111101001010111011010
0101111000100101111001001101110000010101110001110101111111010101
111000110101110111101101110100000101
```

We tried looking for the flag prefix `gpnctf` in the code, both as ASCII binary (`01000111 01010000 01001110 01000011 01010100 01010101`) and as Base64 (`Z3BuY3Rm`), but we could not find any matching string.&#x20;

In addition, we noted that the length of decoded binaries is not divisable by 8. So we conluded that the flag is probably not directly encoded in the data. (Big mistake!)

## The Rabbit Hole

<figure><img src=".gitbook/assets/Screenshot_2026-06-05_at_10.28.33_PM.png" alt=""><figcaption><p>There are definitely some sorts of pattern in the code.</p></figcaption></figure>

After abandoning the earlier appraoch, we started experimenting with different approaches:

1. Different chunk sizes.
2. Count the zeroes on bit level instead of byte level. (Which makes no difference)
3. Using the ratio of zeroes in each chunk as a pixel in a greyscale bitmap.

<figure><img src=".gitbook/assets/output.png" alt="" width="300"><figcaption><p>We though maybe it is a QR code taken at a weird angle.</p></figcaption></figure>

But nothing worked.&#x20;

At this point, we are exhausted.

And the final blow that finally broke my brain was this vertical image.&#x20;

<figure><img src=".gitbook/assets/output3 (1).png" alt=""><figcaption><p>Cropped version of the vertical image.</p></figcaption></figure>

The image is generated with the following script.

```python
from PIL import Image

with open("data", "rb") as f:
    num_chunks = 1530
    width, height = 6, 255 

    pixels = []
    for i in range(num_chunks):
        chunk = f.read(7650000 // num_chunks)
        if not chunk:
            break
        non_zeroes = int.from_bytes(chunk, "big").bit_count()
        total = len(chunk) * 8
        zeroes = total - non_zeroes
        pixels.append(int(zeroes / total * 255))

    img = Image.new("L", (width, height))
    img.putdata(pixels)
    img.save("output.png")
```

At that point, we have probably spent more than ten hour on this chall. To my flag deprived brain, this looks like a string written in a funky font, yet I just couldn't decipher what the actual characters are written. It was getting really late. And we called it a day.



