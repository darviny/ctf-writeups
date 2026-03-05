---
description: by yana
---

# \[1337ctf 2026] misc/maple-signals

A group of ppl are in a circle chatting when I enter the room, with a few others sitting at the sides working solo. I don't recognize any faces, I find an empty seat by the corner and pull up the 1337 CTF page.

It is my first CTF and I know almost nothing about cybersecurity, so I pick the OSINT challenge because it looks the simplest. It is just a photo, can't be that hard right? We are supposed to find the location of where the picture is taken, and the nearby classrooms. I figure the coordinates would be in the metadata, I would pop them into Google Maps, and that would be it.

Using Exiftool (as Gemini recommended), I find noting out of the ordinary in the metadata. So are they really expecting us to walk around and find it? I even begged Aditya, who is right behind me, for a hint, and he just says "Start walking!" I prefer not to sneak around the campus in the evening, so that is the end of my first attempt to solve a challenge.

Pushing forward, I choose the second seem-to-be-the-simplest challenge, Forensics. It is just a wav file. It even comes with a nice short story in the description. Maybe this time, the metadata would prove to be helpful.

And I am wrong again.

Asking Gemini for help. "Download Sonic Visualiser, that's what the pros use," it answers. I am not a pro, but I will use it. Opening the wav file, I am greeted with a solid white bar. Oh, I am zoomed out too far. Zooming in turned the solid bar into a waveform. Cool, but I still have no idea what I am looking at. Playing around with the menus, it seems there are layers of different spectrograms. I cycled through them, zooming in and out. Still no dice. I guess I am not a pro after all.

The next suggestion Gemini gives me is Slow Scan Television (SSTV). Maybe the flag is an image. Except when I compare the sound of our wav file to an SSTV sample on YouTube, they sound nothing alike. Another dead end.

Perhaps sensing the hopelessness, Yana comes over to check on me. When I tell her I am stuck, she suggests trying another challenge first, it might be easier, but I told her I want to give this one more hour. She said "It is all about perspective" and leaves.

Then Zhou comes over. "Wanna work together?", and we start showing each other what we have gotten so far. "That looks like a bar code," Zhou says, when I show him one of the spectrograms. We try scanning it with a bar code scanner. Unfortunately, that doesn't work. It would be really cool if it did.

However, we go back to the challenge description and notice a weirdly specific detail: 100 BPM. That seems weirdly specific in a block of text that is mostly for flavor. "Does that mean 100ms?" I ask Zhou. "10ms," he replies. (Which turns out, we were both wrong lol. 100 BPM is 1 beat every 600ms.) Then Zhou has to go. Before he leaves, he shows me a Python library called librosa. He has been using to it to interact with the wav file.

After Zhou leaves, I keep messing with different "perspectives" until I hit the Waveform layer and adjusted the timescale to 10ms. Suddenly, I see it. The waveform is not continuous. There are distinct spikes, some upward, some downward. These patterns seem to have an interval of 10ms, sort of like Morse code.

I am about to grab a pen and paper to write them manually, but then I remember the library Zhou told me about. With a bit of help from Gemini, I wrote a script that reads the wav file in 10ms chunks and prints 0 or 1 based on the spike direction.

With the binaries fresh out of the oven, and a random binary-to-ASCII decoder I found on Google. (I recognize it is ASCII because we were just talking about it in CPSC 121) And voila, a string that is not complete gibberish shows up. At this point, I was not even sure what a flag looks like. I message Zhou. He explains to me the flag should be in maple{...} format. And that's how I got my first flag.

p.s. Also thank you to Lyndon who showed me how to actually submit it to maple-chan.
