# Author: LT'syreal'Jones
# Category: Forensics
# Medium

___

After downloading the ZIP archive, we extract its contents.

<img width="772" height="45" alt="image" src="https://github.com/user-attachments/assets/e32e237c-06de-43ca-888e-f94d148c73bc" />
<img width="772" height="45" alt="image" src="https://github.com/user-attachments/assets/9cf17c15-8613-4195-bf42-b3e44d4f90c5" />


We use the  `mmls` command to inspect the partition table of a disk image on Ubuntu.

<img width="585" height="45" alt="image" src="https://github.com/user-attachments/assets/2609dc42-7116-434d-84b3-2bd442e60222" />

If the command returns empty results, it means this image is not a full-disk image, but rather a single-partition image (Partition Image).

The challenge provides a hint about creating a Sleuth Kit MAC timeline, so we can use `The Sleuth Kit (TSK)` utilities to perform a deep-dive forensic investigation and generate the timeline.

We use the `img_stat` command to examine the overall details and metadata of the disk image.

<img width="666" height="141" alt="image" src="https://github.com/user-attachments/assets/44c9e973-c6ab-43d7-8003-1fc1244d1575" />

Based on the output shown in the image, this command provides the core technical specifications of the disk image:

- Image Type: The format of the image file (e.g., standard raw image).

- Size in bytes: The exact total size of the image file in bytes.

- Sector size: The sector size of the disk (typically 512 bytes).

Next, we proceed to generate the timeline based on the given hint.

Before generating the timeline, we need one more step: scanning the file system and extracting all metadata into an intermediate file.

<img width="895" height="282" alt="image" src="https://github.com/user-attachments/assets/8357465d-1881-4548-93ca-3c4993b7a903" />

After this step, we proceed to generate the timeline using the `mactime` command with the intermediate file as input.

<img width="745" height="85" alt="image" src="https://github.com/user-attachments/assets/a8aff039-f97a-469b-a996-7354f12bcd7c" />

We proceed to analyze the timeline.txt file. We can use the less command to view it or open the text file directly. For this challenge, a striking anomaly immediately stands out during timeline analysis: while all other timeline entries fall within the 2024 to 2025 timeframe, there is one anomalous entry dated back to 1985.

<img width="1142" height="56" alt="image" src="https://github.com/user-attachments/assets/edb29096-2b48-4bb7-ab69-24e228529233" />

We use the `icat` command to extract the contents of that file based on its inode number: `4945`

Then, we use the `strings` command to inspect the printable character strings contained within the file.

<img width="821" height="95" alt="image" src="https://github.com/user-attachments/assets/c6bce458-765f-4ee6-9e3b-fb0ceb17f4ee" />

We spot the Base64-encoded string, feed it into CyberChef to decode it, and successfully retrieve the flag.

<img width="1130" height="576" alt="image" src="https://github.com/user-attachments/assets/3a1f2fdc-f87f-477b-b549-e3958ca5045b" />


___
