# Author: Makuta A   
# Category: Forensics
# Easy

___

## Challenge Overview
First, download the image to your machine and open it up.

<img width="665" height="1140" alt="Eisoptrophobiactf" src="https://github.com/user-attachments/assets/33181a9f-3b1e-4d24-b66a-aaef088a0ae2" />

Examine the image carefully and read the provided challenge description to catch any obvious visual clues.

The description mentions: "investigate the image, and uncover what lies beneath the surface." This clearly indicates that the challenge belongs to the Steganography category.
## Technical Walkthrough
We will now dig deeper into the image. First, let's inspect the image file's metadata using ExifTool:

<img width="812" height="422" alt="Ảnh chụp màn hình 2026-11-11 141933" src="https://github.com/user-attachments/assets/fa175cfa-9ef6-4641-a3f8-3649bd225cfe" />

The output shows that this is a PNG file with a resolution of 1365 × 2048, RGB color mode, and no noticeable anomalies in the metadata.
Since this is a steganography challenge, we proceed by using zsteg to see if there is any suspicious hidden data embedded deeper inside the image layers:

<img width="937" height="261" alt="Ảnh chụp màn hình 2026-11-11 142103" src="https://github.com/user-attachments/assets/b2b09624-552d-447a-bb92-952e58fcd644" />

The scan results reveal a suspicious Base64-encoded string extracted from the b1,rgb,lsb,xy color channel:
c3Bvbmt5e3RoZV9yZWZsZWN0aW9uX25ldmVyX2xpZXN9

Next, decode this extracted string using CyberChef to check the output:
<img width="956" height="762" alt="Ảnh chụp màn hình 2026-11-11 142313" src="https://github.com/user-attachments/assets/4d948351-2b3d-435f-a9ed-e806fd4c8d42" />

We successfully obtain the flag:
FLAG: spooky{the_reflection_never_lies}

___

This is a basic-level Steganography challenge. Mastering a few fundamental file analysis and steganography commands is more than enough to solve it.
However, the reason I wanted to document this writeup isn't because of its technical difficulty. Eisoptrophobia was the very first challenge I successfully solved upon entering my first official CTF competition. Although it didn't require complex techniques, this lab holds a special meaning for me — serving as a milestone that marks the beginning of my cybersecurity journey. Looking back in the future, this will stand as a reminder that I once started from humble, foundational steps like these.
