---
description: Learning how to read QR codes by eyes
---

# \[lactf2026] misc/error-correction

Flag Objective: Reconstruct a shuffled QR code.

> Looks like their error correction's no match for my error creation!
>
> * [chall.png](https://github.com/uclaacm/lactf-archive/blob/main/2026/misc/error-correction/chall.png)
> * [chall.py](https://github.com/uclaacm/lactf-archive/blob/main/2026/misc/error-correction/chall.py)
>
> [https://github.com/uclaacm/lactf-archive/tree/main/2026/misc/error-correction](https://github.com/uclaacm/lactf-archive/tree/main/2026/misc/error-correction)

First thing I did was throwing the shuffled QR code and the python [script](https://github.com/uclaacm/lactf-archive/blob/main/2026/misc/error-correction/chall.py) into LLM to get freebies without spiking my cortisol. From the script, the LLM  identified this is a gen 7 QR code that has been divided into 5x5 pieces and scrambled. And to solve the challenge, we just need to put the 25 pieces back into order.

<figure><img src=".gitbook/assets/output-onlinepngtools.png" alt=""><figcaption></figcaption></figure>

To do that, I (with the help of LLM) wrote a script to slice 25 chunks with equal size. And reverse engineer the QR skeleton such its Finders, Alignment and Timing pieces. Finders are the chunks with the big squares, Alignment are the chunks with smaller squares, Timing are the chunks with the dotting lines. The Finder and Alignment pieces are relatively easy to find as each chunk has a distinctive square position. (e.g. The middle alignment  has a small square in the middle of the chunk, the bottom alignment has a small square at the top of the chunk, etc). With the help of  Version Information bits, the Timing pieces are also identified. These are 13 out of 25 pieces solved.

To continue, we must find out what masking is the QR code using. I looked at the script, hoping I would find that information. Unfortunately, the masking mode parameter is not specificied. Instead, we rely on `segno` to automatically choose the best masking.

`qr = segno.make(flag, mode='byte', error='L', boost_error=False, version=7)`

#### What is Masking?

> To avoid big blobs of white/black area, we run the raw QR code through one of the eight masking pattern to make sure we have a machine readable QR code.

To find out which version of masking we are using, we can use the **Format information** in the QR code. You can find it at the top left Finder chunk. ( The Top Right and Bottom Left Finders each contain half of the Format information as a form of redundancy, in cae the QR code is damaged ).&#x20;

<figure><img src=".gitbook/assets/block_24.png" alt=""><figcaption></figcaption></figure>
