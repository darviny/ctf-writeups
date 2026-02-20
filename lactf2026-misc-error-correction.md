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

## Recon

First thing I did was throwing the shuffled QR code and the python script into LLM (Gemini 3 Pro) to get a high level overview. Looking at these lines in the script,

```python
qr = segno.make(flag, mode='byte', error='L', boost_error=False, version=7) 
qr.save("temp.txt", border=0) 
with open("temp.txt", 'r') as file: 
    code = file.read()
code = [255-(int(l)255) for l in code if l in ("0","1")] 
chunks = [c for chunk in [[[code[405y+45ysub+9x:405y+45ysub+9*(x+1)] for ysub in range(9)] for x in range(5)] for y in range(5)] for c in chunk] 
random.shuffle(chunks)
```

It seems a version 7 QR code is generated from a flag, and it is divided into 5x5 chunks, with each piece containing 45 bits, and the chunks are shuffled.&#x20;

<figure><img src=".gitbook/assets/chall_1 (1).png" alt=""><figcaption><p>Finders and Alignment patterns</p></figcaption></figure>

Looking at the chall.png, we can see the 3 **Finder** and **Alignment** patterns. So now we just need to unshuffle and reconstruct the QR code.

## Phase 1

<figure><img src=".gitbook/assets/output-onlinepngtools.png" alt=""><figcaption></figcaption></figure>

First thing to do is to slice the shuffled QR code into 25 chunks with equal size. And like putting a jigsaw puzzle back together, we start from the most obvious chunks – **Finders**, **Alignment** and **Timing**.&#x20;

**Finders** are the chunks with the big squares, there are three of them –  at top left, top right and bottom left.

**Alignment** are the chunks with smaller squares, there are six of them – at center, top, left, right, bottom and bottom right.

**Timing** are the chunks with dotting lines, there are four of them. Two veritical and two horizontal. The right horizontal and bottom vertical chunks have an identifiable **Version Information** pattern.

<figure><img src=".gitbook/assets/chall_topright_chunk-mh.png" alt=""><figcaption><p>Notice the Timing pattern below the Version Information pattern.</p></figcaption></figure>

With relatively ease, we have the skeleton of the QR code (13 out of 25 chunks) solved!

## Phase 2

To continue, we must find out what masking is the QR code using. I looked at the script, hoping I would find that information. Unfortunately, the masking mode parameter is not specificied. Instead, we rely on `segno` to automatically choose the best masking.

`qr = segno.make(flag, mode='byte', error='L', boost_error=False, version=7)`

#### What is Masking?

> To avoid big blobs of white/black area, we run the raw QR code through one of the eight masking pattern to make sure we have a machine readable QR code.

To find out which version of masking we are using, we can use the **Format information** in the QR code. You can find it at the top left Finder chunk. ( The Top Right and Bottom Left Finders each contain half of the Format information as a form of redundancy, in cae the QR code is damaged ).&#x20;

<figure><img src=".gitbook/assets/block_24.png" alt=""><figcaption></figcaption></figure>

The Format information itself is masked. To unmask it,we must unmask it by XORing it with `101010000010010` . (Unlike the data, the Format information always uses the same mask.)

```
Masked bits:  111 0111 1001 0001
XOR mask:     101 0100 0001 0010
Reuslt:       010 0011 1000 0011
```

To read the result,

```
Error Correction (2 bits) : 01
Mask Pattern (3 bits): 000
BCH Error-Correction (10 bits) : 11 1000 0011
```

So now we know we are using Mask 0 which is

> &#x20;$$(i+j)(mod2)==0$$&#x20;

which means if the sum of x and y coordinates is divisible by 2 then XOR with 1.

Using LLM, I wrote a script that can unmask a chunk of the QR code, provided we know we know exactly the coordinates of its bits. Using that script, I unmasked the chunk at the bottom right ( where QR code begins).

<figure><img src=".gitbook/assets/unmasked_block_19.png" alt=""><figcaption></figcaption></figure>

In the unmasked chunk, we start from the bottom right bit, then its left, then diagonally top right, then left, and repeat. In our case, the first byte is `0100 0100` , followed by `0111 0101`. The first four bits is our mode bits. (`0100` is byte mode which matches our script, feel free to consult the internet for other mode codes). And the next eight bits are the length bits which is `0100 0111`  or 71 in decimal. So now we know the flag has a length of 71. Then we try to read the first char of our flag which we know is 'l' which is `01101100` (or `01001100` for uppercase). However, our next four bits are `0101` , which means something is wrong!

After consulting with teammates, LLM and the Internet. I realized the raw bytes in QR code are not sequential but actually interleveled. Meaning when you read the bytes, they alternative into two sets.&#x20;

In our case, it means it means after first byte `0100 0100`, it is not followed by the second byte `0111 0101` but rather the third byte, we only know partialy, `11xx xxxx`. That also means the chunk above the bottom right must have `xx01 10xx xxxx xx11 0001 10xx` you read from the botton right zig zag up.&#x20;

Once we found the matching chunk. We now know the correct length of our flag is `0100 1111`  which is 79.

We use the QR generation script, using a dummy flag with the same length, commenting out the shuffle line, we generate variants of the QR code. And there notice that in every chunk, there are parts that look similar to our sliced up chunks. That is because because our data is only 79 chars, there are many padding bits that would look the same independent of the actual value of the data.

With this pattern method, we are left with only 7 unidentified pieces, which we brute forced easily.
