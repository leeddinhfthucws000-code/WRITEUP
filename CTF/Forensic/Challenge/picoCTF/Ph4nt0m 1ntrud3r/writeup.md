# Author: Prince Niyonshuti N
# Category: Forensics
# Easy

___

First, open `myNetworkTraffic.pcap` in Wireshark.

<img width="1917" height="1020" alt="Ảnh chụp màn hình 2026-11-17 104252" src="https://github.com/user-attachments/assets/93f9190e-c141-4e04-ba22-320c6a94c4b4" />

At first glance, the file contains 22 packets, link type 228 (Raw IPv4, no Ethernet header), and all of them are TCP. All 22 packets share the same characteristics:
 
| Field | Value |
| --- | --- |
| Source -> Destination | 10.200.0.2:20 -> 192.168.1.2:80 |
| TCP flags | SYN |
| Seq / Ack | 0 / 0 |
| Window | 8192 |
| TTL | 64 |
| IP ID | 1 |
| Payload length | 4, 8 or 12 bytes |
 
Something unusual stands out: a SYN packet normally carries no data, but here every packet has a payload that looks like Base64 (ending with `=` or `==`). For example, the hex dump of Frame 1 shows the payload bytes `44 46 4f 69 57 58 45 3d`, which is the ASCII string `DFOiWXE=`.
 
<img width="973" height="437" alt="Ảnh chụp màn hình 2026-11-17 104624" src="https://github.com/user-attachments/assets/999f05ff-a799-4576-8dec-b2a5207324e3" />
 
Next, we decode the Base64 payload of each packet separately with CyberChef. Some packets decode to readable text, while others return meaningless characters. For example, Frame 1 (`DFOiWXE=`) decodes to garbage:
 
<img width="957" height="872" alt="Ảnh chụp màn hình 2026-11-17 105328" src="https://github.com/user-attachments/assets/4140676c-5007-42de-b161-d9f6965ccb7c" />

while the payload `ezF0X3c0cw==` decodes to `{1t_w4s`, which looks like a piece of the flag:
 
<img width="957" height="867" alt="Ảnh chụp màn hình 2026-11-17 105438" src="https://github.com/user-attachments/assets/d6fe5086-c037-4112-bcc7-d8b641202ac5" />

The hint says time is essential, so the packets were probably shuffled and we need to put them back in chronological order. In Wireshark we can do this by clicking the Time column to sort.
 
<img width="1917" height="1020" alt="Ảnh chụp màn hình 2026-11-17 105628" src="https://github.com/user-attachments/assets/2207055c-a792-415f-b42d-bd798cea20b3" />

After sorting, the Time column contains negative values. This happens because Wireshark uses Frame 1 as time zero, while the packets inside the file are not stored in chronological order, so the packets that arrived before Frame 1 get negative times. This is direct evidence that the packets were shuffled.
 
Looking at the result, all the meaningless packets are at the top, and Frame 1 itself is the last junk packet (as we saw when decoding it above). Right below Frame 1 there are 7 consecutive packets that decode to readable ASCII. We decode them one by one in chronological order and get:
 
| Order | Frame | Base64 | Decoded |
| --- | --- | --- | --- |
| 1 | 18 | `YWNhZGVteQ==` | `academy` |
| 2 | 11 | `ezF0X3c0cw==` | `{1t_w4s` |
| 3 | 12 | `bnRfdGg0dA==` | `nt_th4t` |
| 4 | 4 | `XzM0c3lfdA==` | `_34sy_t` |
| 5 | 21 | `YmhfNHJfYQ==` | `bh_4r_a` |
| 6 | 16 | `ODEzOWE0MQ==` | `8139a41` |
| 7 | 17 | `fQ==` | `}` |
 
Joining the 7 parts together gives us the flag:
 
___
