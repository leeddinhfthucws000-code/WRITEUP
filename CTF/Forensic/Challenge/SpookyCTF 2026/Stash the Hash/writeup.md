# Author: Jtjnjit
# Category: Forensics
# Medium

___

First, read the provided scenario to check for clues. We see the line "If you can find out what their password is you may be able to obtain this information for yourself", which indicates that we need to find a password either left exposed or hidden somewhere inside the attached zip folder in order to retrieve the information (likely the flag), as the rest of the text contains no other useful details. Next, we download the archive to our machine and extract it.

<img width="882" height="497" alt="Ảnh chụp màn hình 2026-11-12 221350" src="https://github.com/user-attachments/assets/f619725d-fe07-473b-a542-5dc0bef44ddb" />

<img width="812" height="160" alt="Ảnh chụp màn hình 2026-11-12 221427" src="https://github.com/user-attachments/assets/39e89830-ff61-4a59-ac17-c0ae855368e5" />

We obtain a .7z archive and a .txt file. Let's open the .txt file to inspect its contents.

<img width="665" height="352" alt="Ảnh chụp màn hình 2026-11-12 221524" src="https://github.com/user-attachments/assets/ea1018ee-e497-4948-9b38-54bc5fa58abd" />

We notice something unusual in the password field: ad7bcd7ab3cc113f496e1840bc582bbd is a standard 32-character hexadecimal string representing an MD5 hash. We take this hash and run it through specialized tools such as CyberChef or CrackStation to see if we can recover the plaintext password.

<img width="1917" height="767" alt="Ảnh chụp màn hình 2026-11-12 222150" src="https://github.com/user-attachments/assets/69f9c89b-52fb-404d-a4b3-549d6a366ee3" />

We successfully crack ad7bcd7ab3cc113f496e1840bc582bbd into newjerseydevils. Combining this with the clue gathered at the beginning of the challenge regarding the password, we use this newly obtained string as the password to extract the remaining .7z file.

<img width="1917" height="1022" alt="Ảnh chụp màn hình 2026-11-12 222344" src="https://github.com/user-attachments/assets/2f62137a-d0c3-4526-8abf-c92c8cb4d515" />

<img width="632" height="292" alt="Ảnh chụp màn hình 2026-11-12 222400" src="https://github.com/user-attachments/assets/61a5a2c0-21e2-400e-9e8c-b80ff5d6e390" />

After extracting the archive, we get an image depicting a mythical/eldritch creature and no other useful files. There is a very high chance that the flag is hidden inside this image, so we will proceed to analyze it. We use basic Linux steganography and metadata commands such as exiftool and zsteg to inspect its basic properties and check if anything unusual is hidden inside.

<img width="1195" height="402" alt="Ảnh chụp màn hình 2026-11-12 223051" src="https://github.com/user-attachments/assets/bfabcf3d-e3b3-41c5-aea0-7588b61de24e" />

<img width="1225" height="265" alt="Ảnh chụp màn hình 2026-11-12 223135" src="https://github.com/user-attachments/assets/859681f1-3ef1-412a-8b9f-6fb1b060af7e" />

With exiftool, we don't find anything noteworthy. However, with zsteg, we spot two strange character strings (q3Q5uSUus and QsSwqUSQw7Ssq), though decoding them in CyberChef yields nothing—even when combining both strings. On the second line, however, zsteg detects an unusual executable/binary file hidden within the least significant bits (LSB) in the abgr channel order. Therefore, we upload this image to an open-source deep analysis tool, Aperi'Solve.

<img width="1917" height="902" alt="Ảnh chụp màn hình 2026-11-12 224009" src="https://github.com/user-attachments/assets/5b2a4dea-c8fa-44d0-af02-3d9c178f2a42" />

Reviewing all the results analyzed by the tool, we find the flag hidden in the Red color channel:

<img width="1696" height="891" alt="Ảnh chụp màn hình 2026-11-12 224114" src="https://github.com/user-attachments/assets/439b2108-d049-4a73-8c72-406ba2c6773a" />
