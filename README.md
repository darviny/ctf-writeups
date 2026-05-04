---
cover: .gitbook/assets/landscape_8343abe8fb00bab9b7e2c00774ffbae2.jpg
coverY: 90.76923076923077
coverHeight: 408
layout:
  width: default
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
---

# \[1337ctf2026] misc/maple-signals

## Challenge Details:

<details>

<summary>Author: Yana</summary>

Date: 12/12/2026

Solves: 10

Tools Used: 123123123123

</details>

A group of ppl are in a circle chatting when I enter the room, with a few others sitting at the sides working solo. I don't recognize any faces, I find an empty seat by the corner and pull up the Maple Bacon 1337 CTF page.

It is my first CTF and I know almost nothing about cybersecurity, so I pick the OSINT challenge because it looks the simplest. It is just a photo, can't be that hard right? We are supposed to find the location of where the picture is taken, and the nearby classrooms. I figure the coordinates would be in the metadata, I would pop them into Google Maps, and that would be it.

Using **Exiftool** (as Gemini recommended), I find nothing out of the ordinary in the metadata. So are they really expecting us to walk around and find it? I even begged Aditya, who is right behind me, for a hint, and he just says "Start walking!" I prefer not to sneak around the campus in the evening, so that is the end of my first attempt to solve a challenge.

Pushing forward, I choose the second seem-to-be-the-simplest challenge, Forensics. It is just a wav file. It even comes with a nice short story in the description. Maybe this time, the metadata would prove to be helpful.

And I am wrong again.

Asking Gemini for help. "Download **Sonic Visualiser**, that's what the pros use," it answers. I am not a pro, but I will use it. Opening the wav file, I am greeted with a solid white bar. Oh, I am zoomed out too far. Zooming in turned the solid bar into a waveform. Cool, but I still have no idea what I am looking at. Playing around with the menus, it seems there are layers of different spectrograms. I cycled through them, zooming in and out. Still no dice. I guess I am not a pro after all.

<figure><img src=".gitbook/assets/Screenshot 2026-03-04 at 6.25.42 PM.png" alt=""><figcaption><p>Sonic Visualiser</p></figcaption></figure>



The next suggestion Gemini gives me is [Slow Scan Television (SSTV)](https://en.wikipedia.org/wiki/Slow-scan_television). Maybe the flag is an image. Except when I compare the sound of our wav file to an SSTV sample on YouTube, they sound nothing alike. Another dead end.

Perhaps sensing the hopelessness, Yana comes over to check on me. When I tell her I am stuck, she suggests trying another challenge first, it might be easier, but I told her I want to give this one more hour. She said "It is all about perspective" and leaves.

Then Zhou comes over. "Wanna work together?", and we start showing each other what we have gotten so far. "That looks like a bar code," Zhou says, when I show him one of the spectrograms. We try scanning it with a bar code scanner. Unfortunately, that doesn't work. It would be really cool if it did.

<figure><img src=".gitbook/assets/Screenshot 2026-03-04 at 6.19.24 PM.png" alt=""><figcaption><p>The "Bar Code"</p></figcaption></figure>

However, we go back to the challenge description and notice a weirdly specific detail: **100 BPM**. That seems weirdly specific in a block of text that is mostly for flavor. "Does that mean 100ms?" I ask Zhou. "10ms," he replies. (Which turns out, we were both wrong lol. 100 BPM is 1 beat every 600ms.) Then Zhou has to go. Before he leaves, he shows me a Python library called `librosa`. He has been using to it to interact with the wav file.

After Zhou leaves, I keep messing with different "perspectives" until I hit the Waveform layer and adjusted the timescale to 10ms. Suddenly, I see it. The waveform is not continuous. There are distinct spikes, some upward, some downward. These patterns seem to have an interval of 10ms, sort of like Morse code.

<figure><img src=".gitbook/assets/Screenshot 2026-03-04 at 6.19.06 PM.png" alt=""><figcaption><p>Example of an upward spike that would transcribe to 1 in binaries.</p></figcaption></figure>

I am about to grab a pen and paper to write them manually, but then I remember the library Zhou told me about. With a bit of help from Gemini, I wrote a script that reads the wav file in 10ms chunks and prints 0 or 1 based on the spike direction.

```python
y, sr = librosa.load("maple-signals.wav", sr=None)

samples_per_bit = int(sr * 0.01)

bits = []

for i in range(0, len(y), samples_per_bit):
    chunk = y[i : i + samples_per_bit]
    if len(chunk) == 0: continue

    peak = chunk[np.argmax(np.abs(chunk))]

    bits.append("1" if peak > 0 else "0")

bin = "".join(bits)
```

With the binaries fresh out of oven, and a random binary-to-ASCII decoder I found on Google. (I recognize it is ASCII because we were just talking about it in CPSC 121) And voila, a string that is not complete gibberish shows up. At this point, I was not even sure what a flag looks like. I message Zhou. He explains to me the flag should be in maple{...} format. And that's how I got my first flag.

> maple{M4ple\_bY\_\_fr<3nd}

p.s. Also thank you to Lyndon who showed me how to actually submit it to maple-chan.
