---
description: 'chall author: yana'
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

# \[1337ctf2026] forensics/maple-signals

A group of ppl are in a circle chatting when I enter the room, with a few others sitting at the sides working solo. I don't recognize any faces, so I find an empty seat by the corner and pull up the Maple Bacon 1337 CTF page.

## First Try

It is my first CTF. I know nothing about cybersecurity. I scroll through the challenges and pick the OSINT one. It is just a photo in .JPG file, can't be that hard right? We are supposed to find the location of where the picture was taken, and the nearby classrooms. I figure the coordinates would be in the metadata, then I would pop them into Google Maps, and that would be it.

Using **Exiftool** (as Gemini recommended), I find nothing out of the ordinary in the metadata. So are they really expecting us to walk around and physically find it? I beg Aditya, who is right behind me, for a hint. He smirks at me and says "Start walking!". I prefer not to sneak around the campus in the evening, so that is the end of my first attempt to solve a challenge.

## Second Try

Pushing forward, I pick the second seem-to-be-the-simplest challenge – Forensics. It is just a .WAV file. It even comes with a nice short story in the description. Maybe this time, the metadata would prove to be helpful.

And I am wrong again.

Not knowing what to do, I ask Gemini for help again. "Download **Sonic Visualiser**, that's what the pros use," it answers. I am not a pro, but I will use it. Opening the .WAV file, I am greeted with a solid blue bar. Oh, I am zoomed out too far. And as I zoom in, the solid bar starts to turn into a waveform. Cool, but I still have no idea what I am looking at. Playing around with the menus, it seems there are layers of different [spectrograms](https://en.wikipedia.org/wiki/Spectrogram). After cycling through all of them, zooming in and out. I still have no idea what I am looking at. I guess I am not a pro after all.

<figure><img src=".gitbook/assets/Screenshot 2026-03-04 at 6.25.42 PM.png" alt=""><figcaption><p>Sonic Visualiser</p></figcaption></figure>

The next suggestion Gemini gives me is [Slow Scan Television (SSTV)](https://en.wikipedia.org/wiki/Slow-scan_television). Maybe the flag is an image hidden in the spectrogram. Except when I compare the sound of our .WAV file to a SSTV sample on YouTube, they sound nothing alike. Another dead end.

Perhaps sensing the hopelessness, Yana comes over to check on me. After seeing that I am stuck, she suggests trying another challenge. It might be easier. I tell her I want to give this one more hour. She looks at me and says "It is all about perspective." and leaves.

Then Zhou, who sits at the other corner of the room, rolls his chair over and asks "Wanna work together?". We show each other what we have gotten so far. When I show him one of the spectrograms, "That looks like a bar code," Zhou says. We try scanning it with a bar code scanner. Unfortunately, it doesn't work. It would be really cool if it does.

<figure><img src=".gitbook/assets/Screenshot 2026-03-04 at 6.19.24 PM.png" alt=""><figcaption><p>The "Bar Code"</p></figcaption></figure>

Part not willing to admit defeat, part trying to impress each other. We find ourselves staring at the challenge description. A weirdly specific detail started to catch our attention : **100 BPM**.&#x20;

It seems weirdly specific to include that in a block of text that is mostly for flavor. "Does that mean 100ms?" I ask Zhou. "10ms," he replies. (Which turns out, we were both wrong lol. 100 BPM is 1 beat every 600ms.) Then Zhou has to go. Before he leaves, he shows me a Python library called `librosa`. He has been using to it to read the .WAV file in his Python script.

Going back to Sonic Visualiser, I keep playing with the different "perspectives". Then suddenly, I see it. In the Waveform layer, with the timescale sets to 10ms, the waveform is not continuous. There are distinct spikes, some upward, some downward. These patterns seem to have a constantinterval, sort of like Morse code.

<figure><img src=".gitbook/assets/Screenshot 2026-03-04 at 6.19.06 PM.png" alt=""><figcaption><p>Example of an upward spike that would transcribe to 1 in binaries.</p></figcaption></figure>

I am about to grab a pen to write the code manually, then I remember the Python library Zhou just told me. With a bit of help from Gemini to understand the API, I now have a Python script that reads the .WAV file in 10ms chunks and prints 0 or 1 depending on the spike direction.

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

With the binaries fresh out of oven, and a random binary-to-ASCII decoder I found on Google. (I recognize it is ASCII because we were just talking about it in CPSC 121). A string that is not complete gibberish shows up. At this point, I am not even sure what a flag looks like. I message Zhou. He explains to me the flag should be in `maple{...}` format. And that's how I got my first flag.

p.s. Also thank you to Lyndon who showed me how to actually submit it to maple-chan.

## Flag

> ### **`maple{M4ple_bY__fr<3nd}`**



{% tabs %}
{% tab title="Details" %}
chall author: [**yana**](https://maplebacon.org/authors/yana/)\
files: [**maple\_signals.wav**](https://maplebacon.org/assets/1337-2026/forensics/maple-signals.wav)\
\
<sup>LLM used:</sup> <sup></sup><sup>**Gemini 3.0 Pro**</sup>\
<sup>time to solve:</sup> <sup></sup><sup>**\~3 hr**</sup>\
<sup>date:</sup> <sup></sup><sup>**1/16/2026**</sup>
{% endtab %}
{% endtabs %}

