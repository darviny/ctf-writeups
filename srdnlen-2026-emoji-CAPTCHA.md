# \[srdnlen2026] misc/emoji-CAPTCHA

This challenge looks interesting. I have spent some time dealing with OCR and of course emojis (IYKYK) before and I didn’t hate them.



Without looking at the actual Captcha outputs from the server, I was thinking about building a hash table using the emojis as hash. That idea quickly went away when I got my first sample from the server.



The Captcha is a fixed size image with two rows of four emojis. The emojis are in fixed locations, same size, non distorted, and the background is always white. What broke the hashtable idea is that they are randomly rotated. This creates antialiasing and if we use loosy matching, then we won’t be able to hit the accuracy we need.



To get the flag, we must pass 100 rounds of solving the captcha without any mistakes. That’s 800 emojis. Even with 99% accuracy, it will fail.



After bouncing some ideas with Gemini and consulting Google (actually had to venture into stackoverflow lol), I decided to try building a model with features extracted using Hu Moments and color histograms. These features are the most effective when dealing with rotated images.



To build the training materials, we need to render the emojis in various angles. The author posted a git hosting the font files. I thought about doing a deep dive into the files (maybe the flag is hidden in there?) but who got time for that.



With some small hiccups dealing with multi code point (solved with replacing Pillow + RAQG with Cairo), I started iterating on my models with local benchmark. What I found is that, the color histogram is doing most of the work, which in hindsight makes sense because emojis, unlike characters, are lot more richer in terms of color data. Hu Moments is useful for more broad strokes. So the final tuning for the weights is 0.05 Hu Moments and 0.95 Color Histogram.



At this stage, I was getting 95% accuracy in my local benchmark. And as I said , we need more than 99%. Thankfully, there are enough gaps in the confidence score that I have a pretty good idea which are the risky emojis, about 300 of them out of 3800. So I built a second model using only those emojis but with higher resolution features. And with a cascading solving system. (If the first model has low confidence then use second model) This pushed the accuracy to 97%. Time to test the system live. Then things got weird.



I was failing at Round One and Two constantly. Something is not right. I looked through the CAPCHA the server is sending, then I noticed some emojis are misrendered. (The layered glyphs don’t have proper offsets) Something sus is going on with the font files.



I started doing a deep dive on them. I started reading about SBIX and how it stores the rendering data. Are they tempered? That would be one hell of a challenge if they were. I thought about collecting the rendered emoji from the server, that would require a really long time and possibly manual data labeling.



At this point, I was just ranting to Gemini how stupid emojis are and of course it was invented by Apple for iPhone. Then Gemini started taking about morx table. Apple proprietary version of emoji rendering technology. As soon as I saw the word proprietary, then I knew this is a cross platform problem.



Our font is the Apple Emoji font, and while it renders flawlessly on my MacBook, there are certain glitches when you run it on a Linux server. To test my hypothesis, I spin up my docker and run the rendering script again. And viola, now we can train the model with matching data with the server.



With my updated model, I am able to push to Round Ten consistently. (One really lucky run I got to Round 44) That’s still a farcry from Round 100. Then I have this really stupid idea. I am going to manually solve the CAPTCHA. I just want to get the flag and go to sleep.



So I built a frontend for the solving script. When the confidence level is low, it would show a pop up window with five possible best guesses, and the human (me in my sleep deprived status) would pick the correct one.



With this human + machine hybrid approach, I was doing better but there is still a gap before we can get to Round 100. While I was doing the CAPCHA manually, I started to notice patterns in the low confidence results. It is not good with samey colors and symmetrical shape. I started building specialized models (faces, hands, clocks, blue squares, purple squares ) and chain them in the cascading solving system.



About an hour before the deadline, with some luck (there is one specific emoji my system is blind to) and hand eye coordination. I finally got the flag. GGWP
