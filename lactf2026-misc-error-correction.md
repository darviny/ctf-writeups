---
description: by pstorm
---

# \[lactf2026] misc/error-correction

> #### **Description:**&#x20;
>
> Looks like their error correction's no match for my error creation!
>
> #### **Files:**
>
> * [chall.png](https://github.com/uclaacm/lactf-archive/blob/main/2026/misc/error-correction/chall.png)
> * [chall.py](https://github.com/uclaacm/lactf-archive/blob/main/2026/misc/error-correction/chall.py)
>
> [<sub>https://github.com/uclaacm/lactf-archive/tree/main/2026/misc/error-correction</sub>](https://github.com/uclaacm/lactf-archive/tree/main/2026/misc/error-correction)

## Introduction

This is the second challenge I did for LACTF 2026. The reason I chose it is because it was the only unsolved misc challenge with a good number of solves on the second day.

We are provided a obstrucfued QR code and a python script. Seems simple enough, or so it seems.

## Recon

First thing I did was throwing the obstrucfued QR code and the python script into LLM to get a quick overview. The LLM quickly pinpointed these lines,

```python
qr = segno.make(flag, mode='byte', error='L', boost_error=False, version=7) 
qr.save("temp.txt", border=0) 
with open("temp.txt", 'r') as file: 
    code = file.read()
code = [255-(int(l)255) for l in code if l in ("0","1")] 
chunks = [c for chunk in [[[code[405y+45ysub+9x:405y+45ysub+9*(x+1)] for ysub in range(9)] for x in range(5)] for y in range(5)] for c in chunk] 
random.shuffle(chunks)
```

The script generates a version 7 QR code from a flag, then the QR code is divided into 5x5 chunks, with each chunk containing 45 bits, and finally the chunks are shuffled and put back together.

So our job is to unshuffle the chunks and reconstruct the QR code like a jigsaw puzzle.

<figure><img src=".gitbook/assets/chall_1 (1).png" alt=""><figcaption><p>Finders and Alignment patterns</p></figcaption></figure>

## Solve (Phase 1)

<figure><img src=".gitbook/assets/output-onlinepngtools.png" alt=""><figcaption></figcaption></figure>

1. Slice the QR code into 25 chunks like it was sliced in chall.py.
2. Find the most obvious chunks – **Finders**, **Alignment** and **Timing**.

* **Finders** are the chunks with the big squares. (3 chunks)\
  &#xNAN;_&#x50;osition: top left, top right and bottom left._
* **Alignment** are the chunks with smaller squares. (6 chunks)\
  &#xNAN;_&#x50;osition: center, top, left, right, bottom and bottom right._
* **Timing** are the chunks with dotting lines. (4 chunks)\
  &#xNAN;_&#x54;wo vertical and two horizontal chunks._ The right horizontal and bottom vertical chunks have an **Version Information** pattern.

<figure><img src=".gitbook/assets/chall_topright_chunk-mh.png" alt=""><figcaption><p>Notice the Timing pattern below the Version Information pattern.</p></figcaption></figure>

And we have 13 out of 25 chunks solved!

## Solve (Phase 2)

To continue, we must find out what masking is the QR code using. I looked at the script, hoping I would find that information. Unfortunately, the masking mode parameter is not specificied. Instead, we rely on `segno` to automatically choose the most optimal masking.

`qr = segno.make(flag, mode='byte', error='L', boost_error=False, version=7)`

### What is Masking?

> To avoid big blobs of white/black area, we run the raw QR code through one of the eight masking pattern to make sure we have a machine readable QR code.

To find out which version of masking we are using, we can use the **Format information** bits in the QR code. You can find it at the top left Finder chunk. ( The Top Right and Bottom Left Finders each contain half of the Format information as a form of redundancy, in cae the QR code is damaged ).&#x20;

The Format information itself is masked. We can unmask it by XORing it with `101010000010010`. (Unlike the data, the Format information always uses this mask.)

<figure><img src=".gitbook/assets/block_24-mh.png" alt=""><figcaption><p>Format bits are highlighted here.</p></figcaption></figure>

```
Format bits:  111 0111 1001 0001
XOR mask:     101 0100 0001 0010
Reuslt:       010 0011 1000 0011
```

The first two bits are the **Error Correction** bits, followed by three **Mask Pattern** bits, and ten **BCH Error Correciton** bits.

```
Error Correction (2 bits) : 01
Mask Pattern (3 bits): 000
BCH Error-Correction (10 bits) : 11 1000 0011
```

`000` tells us that  we are using Mask 0 which is

> &#x20;$$(i+j)(mod2)==0$$&#x20;

which means if the sum of x and y coordinates is divisible by 2 then XOR it with 1.

With the help of LLM, I wrote a script that can unmask the QR code.&#x20;

<figure><img src=".gitbook/assets/unmasked_block_19.png" alt=""><figcaption><p>The bottom right chunk now unmasked.</p></figcaption></figure>

### Reading the Data

To read the data on a QR code, we start reading it from the bottom right bit. Then the next bit is its left, then diagonally top right, then left, and repeat.

<figure><img src=".gitbook/assets/unmasked_block_19 big-mh-2.png" alt=""><figcaption></figcaption></figure>

* **Mode Bits**: The first four bits are. `0100` is for **byte mode** which matches `mode='byte'` in the python script.
* **Length Bits:** The eight bits after the Mode Bits. It is the length of our data in binaries. `0100 0111` means the length of our data is 71 bits.

Following the **Mode Bits** and **Length Bits** are the data bits in our QR code. In our case, it should be a string starting with `lactf{`, so the next eight bits should be 'l' in binaries or `01101100`. However, our next four bits are `0101` instead, which means something is wrong!

After some discussions on Discord, I realized the bits in our **QR code are not sequential but actually interleveled**. Meaning when you read the bytes or codewords as they are called, they alternative into two sets.

<figure><img src=".gitbook/assets/unmasked_block_19 big 2-mh.png" alt=""><figcaption><p>Codewords alternate into two sets.</p></figcaption></figure>

In our case, it means after first codeword `0100 0100`, it is not followed by the second codeword `0111 0101`, but rather the third codeword, which we only know start with `11`. This means the length bits are `0100 11xx`. Which means we are uncertain what our length is anymore, however, one thing that is certain is that the data bits start with 'l' or `0110 1100`. That also means the chunk above the starting chunk must match the pattern `xx0110 xxxxxxxx 1100`.

<figure><img src=".gitbook/assets/unmasked_darvin_chall copy-mh.png" alt=""><figcaption><p>The codewords in Set 1 (Yellow) give us the correct length and character 'l'.</p></figcaption></figure>

Now that we  know the correct length is 79. We can use the QR generation script, using a dummy flag with the same length, to generate variants of the QR code (without shuffling the chunks of course).

And notice that in every chunk, there are parts that consistently look similar.



&#x20;to our sliced up chunks. That is because because our data is only 79 chars, there are many padding bits that would look the same independent of the actual value of the data.

With this pattern method, we are left with only 7 unidentified pieces, which we brute forced easily.
