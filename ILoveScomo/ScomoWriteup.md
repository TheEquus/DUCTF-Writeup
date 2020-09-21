# Scomo Challenge
How I ended up solving a challenge at 4am

## The Challenge
![][ChallDesc]  
Here, we are given a challenge by someone who absolutely adores Scomo. We're told upfront that it's a stego challenge, so clearly this'll be fun. **Hidden space** is in bold, so that might be important in the future...

But for now, let's take a look at the image provided:  
![][ILoveScomo]

## The Thought Process
I can't have been the only one who looked at this image and thought that this image was missing a bit of "owo uwu". It was also somewhere around 12am by the time I got around to doing this, so that's always a great start. Anyway, given that we're told that this is steganography, and we're given a jpg image, it's always best to go through some simple stego checks.  

Let's go through my rough checklist:  
### 1. Exif data
I actually use two tools to check exif data, just in case one of them misses some crucial info, though most people just stick with one. I'd say better safe than sorry.  
![][ExifOnline]  
So using one of my favourite sites for viewing image metadata, [Jeffrey's Image Metadata Viewer](http://exif.regex.info/exif.cgi), it returned nothing.  

![][Exiv2]  
My next favourite metadata viewer, exiv2, also returned nothing, shame. Now onto check number 2  

### 2. Image Manipulation
Sometimes, all you need to see hidden data is to change up how you see the image. Altering brightness/contrast, or focusing on specific colour planes can help! Let's start off with [Photopea](https://www.photopea.com/), a very nice online photoshop.  

![][photopea]
With levels, I usually just drag the dots so that it "focuses" on the peaks of the graph. Unfortunately, this gave us nothing helpful (except for deepfried scomo, quite a sight for the midnight brain). Now, let's try an old stego favourite, stegsolve.

![][Stegsolve]  
Some interesting views of scomo, but still, nothing that stands out...  

### 3. Pixel/Bits Analysis
This usually involves looking at the individual pixels of the image, and analysing their values.  
A simple example: For a single pixel with RGB values (in binary) `01100001 01100010 01100011`, if we were to convert them to ASCII values, we'd get `abc`.
Least significant bit (LSB) extends on this, by looking at only one bit of every byte. For the above example, it'd look at the least significant bit (usually the rightmost bit). So for the example, the LSB of the three bytes are `1,0,1`.  

How useful was this knowledge for solving the challenge? As it turns out, not at all! But let's quickly go through some of the failures:
1. Writing a python script
    - Python Image Library (PIL) is a godsend. Just using some get_pixel() functions we get the binary values of each pixel, but they turned out useless for this anyway.
2. [Steganography Online](http://stylesuxx.github.io/steganography/)
    - An online LSB encoder/decoder
3. JSteg
    - Given that the image provided was a jpg, time to start directing some jpg dedicated tools at it.
    ![][JSteg]
    - Well that's a shame...
---
At this point, the clock struck 2am. Did it really take 2 hours to go through the checklist? No, but it did take me that long to repeat the entire process again and again, only to realise there were a couple things that I had missed...   

---
### Some Extra Miscellaneous Stuff
To sort of guarantee that the data needed wasn't hidden visually, I decided to try and find the original image (without the hearts and blatant love), and something that could highlight any differences. This check wasn't too important, since usually Stegsolve would cover this.

Now, we go to the magical *Steghide*. I did not try using Steghide in the initial checklist since most of the time, a known password is needed to reveal anything. Though apparently, sometimes not having a password works, but not in this case. While you could use the command line version of steghide, I ended up using an [online version](http://futureboy.us/stegano/decinput.html), but they both work the same.
![][Steghide]  
Evidently, I had tried quite a number of possible passwords based on the challenge description and the image itself (the actual correct one I actual hadn't tried at the time, I ended up putting it later on when I actually got the password, hence why it appears in this list).

---
Now the clock read 03:00. I had exhausted my relatively short stego list of things to check, so what now? This is where the fun really begins.

---

### Go Mad \<insert search engine\>-ing
Looking for anything, that could give me a glimmer of hope, I spent a good chunk of time searching online, trying to find some useful tools, maybe even try and read a couple of research papers. Turns out, the answer lies in a very useful github repo.

### Stegcracker
Now, I'm sure many people who've done thousands upon thousands of stego challenges have come across this tool, but for me, this was brand new, and absolutely beautiful. It's simple, it bruteforces Steghide passwords based on a wordlist you provide. It's so simple, there's no way it'd work on a challenge like this, one that at this point, I'd spent 3 hours staring at...  
![][Stegcracker]  
Oh you're joking...  
"Tried 5 passwords" really rubs salt into the wound. Mocking for something it found so simple.  

Now, we have an interesting output, let's take a look at what it could be.  
![][ScomoOutput]  
That looks to me like the Australian Anthem, but repeated a good number of times.

## Stage 2
No flag in sight yet, so onto the surprise stage 2 we go!  
Given that the bold **hidden space** hint hadn't been used yet, I figured maybe it was referencing Stegsnow, by hinting at some sort of whitespace steganography. Issue is, Stegsnow  uses spaces and tabs and encodes data using binary that way, but upon closer inspection of each character used in this, there were only spaces (and new lines). So there was no way it was Stegsnow. Well what else could it be?

I happened to come across this next bit accidentally. Whilst trying to figure out how many times the anthem was repeated, I noticed something peculiar...  
![][PeculiarHighights]  
Not all the first verses were highlighted, which was quite unexpected.  

Upon insanely close inspection, I noticed that the end of some lines had an extra space, whilst others didn't! Just to double check that this was something interesting, I decided to manually check the first 8 lines, assuming it was binary, and trying an extra space as 1, 0 otherwise:  
`01000100`  
Converting this into ASCII we get the letter `D`, a possible beginning to the flag!

Now, I didn't have all day to decode this (03:50 is not a good time to manually decode long binary strings), I quickly wrote up a simple script to get the job done for me.

```py
data = open('scomo.txt').readlines()

binary = ''

for line in data:
    if (len(line) >= 2):
        if ord(line[-2]) == 32:
            binary += '1'
        else:
            binary += "0"
    else:
        binary += '0'
print(binary)
```
Not the cleanest, but it got the job done. I had initially ignored the single new line lines, thinking they weren't important, but turns out they were very important for getting the flag.

Now we are at the final step.  
`010001000101010101000011010101000100011001111011011010010101111101010010001100110110110001001100011010010101111101101100001100000011000000110000001100000100111100110000011011110110111100110000011101100011001101011111001101010110001100110000011011010011000001111101000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000`  
Converting this binary mess into (what is hopefully) the flag, and celebrating. Thankfully, this was trivial, and after hours of fooling around at the start, we at long last, get our flag:   
`DUCTF{i_R3lLi_l0000O0oo0v3_5c0m0}`
![][Solve]    
And at 03:54, I was somewhat at peace at last. The birds had even started to chirp to celebrate this moment with me. Since when did birds wake up at 4am?

## Conclusion

I can't say that I can love Scomo after this chall, after all, I had been staring at his face for way too many hours...  
![][Aftermath]  
05:00 and I was still awake, after being tortured by that chall for about 4 hours. Despite all that, I still had quite a blast solving the challenge.  


<!-- image references -->
[ChallDesc]: images/ChallDesc.png
[ILoveScomo]: images/ILoveScomo.jpg
[ExifOnline]: images/ExifOnline.png
[Exiv2]: images/Exiv2.png
[Photopea]: images/Photopea.gif
[Stegsolve]: images/Stegsolve.gif
[JSteg]: images/JSteg.png
[Steghide]: images/Steghide.gif
[Stegcracker]: images/Stegcracker.png
[ScomoOutput]: images/ScomoOutput.png
[PeculiarHighights]: images/PeculiarHighlights.png
[Solve]: images/Solve.png
[Aftermath]: images/Aftermath.png
