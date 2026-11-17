# Wireshark
 
A self-study guide you can read and use right away. Written for anyone who wants to capture packets, read them, and hunt for flags inside pcap files.
 
## Table of Contents
 
- [0. Preparation](#0-preparation)
- [1. Interface Basics](#1-interface-basics)
- [2. Capturing Packets](#2-capturing-packets)
- [3. Display Filters](#3-display-filters)
- [4. Analysis Tools](#4-analysis-tools)
- [5. Extracting Files from a Pcap](#5-extracting-files-from-a-pcap)
- [6. Decrypting TLS and Wi-Fi](#6-decrypting-tls-and-wi-fi)
- [7. Where Flags Hide in Pcaps](#7-where-flags-hide-in-pcaps)
- [8. Everyday Network Analysis](#8-everyday-network-analysis)
- [9. tshark, the Command-Line Wireshark](#9-tshark-the-command-line-wireshark)
- [10. A Workflow for CTF Pcaps](#10-a-workflow-for-ctf-pcaps)
- [11. Shortcuts and Common Pitfalls](#11-shortcuts-and-common-pitfalls)
- [12. Resources and Practice](#12-resources-and-practice)
## 0. Preparation
 
- Wireshark is currently on the 4.6.x branch (4.6.9 was released in September 2026). Download it from [wireshark.org/download](https://www.wireshark.org/download/) and keep it updated, since recent releases fixed quite a few security bugs in the dissectors and file parsers.
- For the same reason, open untrusted pcaps (CTF files, malware samples) inside a virtual machine, not directly on your main machine.
- Only capture traffic on networks you own or have permission to test.
- Companion tools: **tshark** (installed with Wireshark), [CyberChef](https://gchq.github.io/CyberChef/) (decoding), `xxd`, `file`, `strings`, `binwalk`.
## 1. Interface Basics
 
Remember three panes and you are good to go:
 
- **Packet List** (top): one row per packet, showing number, time, source, destination, protocol, length and Info.
- **Packet Details** (middle): a protocol tree that peels the packet layer by layer: Frame → Ethernet → IP → TCP/UDP → application. Click a field and the matching bytes light up in the pane below.
- **Packet Bytes** (bottom): hex and ASCII view of the packet.
- **Filter bar**: green = valid syntax, red = invalid, yellow = valid but suspicious. Black or red rows in the Packet List usually mean trouble (retransmissions, RST, checksum errors).
One-time setup that saves you pain later:
 
- `View → Time Display Format`: pick *Seconds Since Previous Displayed Packet* or *Date and Time of Day*, depending on what you need.
- `View → Name Resolution`: turn off name resolving so you see real IPs and ports, with less noise.
- Right-click any field (for example `http.host`) → **Apply as Column** to put it in the packet list.
- `Edit → Configuration Profiles`: create a dedicated CTF profile so your columns and filter buttons do not get mixed up with other work.
## 2. Capturing Packets
 
Pick an interface with a wiggling activity graph (Wi-Fi, Ethernet, Loopback, docker0, ...) and double-click it to start. `Ctrl+E` stops/restarts the capture, `Ctrl+S` saves (pcapng by default), and `Ctrl+K` opens Capture Options for promiscuous mode, ring buffers for long captures, and so on.
 
There are two kinds of filters. Do not mix them up:
 
| | Capture filter | Display filter |
|---|---|---|
| Used | Before capturing | After capturing |
| Syntax | BPF: `host 10.0.0.5`, `port 80`, `not port 22`, `net 192.168.1.0/24` | Wireshark's own: `ip.addr == 10.0.0.5`, `tcp.port == 80` |
| Filtered packets | Lost for good | Only hidden |
 
Advice: capture broadly and filter later with display filters (unless the network is extremely busy). On a headless server, use `sudo tcpdump -i eth0 -w out.pcap port 80`, then copy the file over and open it in Wireshark.
 
## 3. Display Filters
 
This is the most important skill in Wireshark.
 
Syntax: comparison operators `== != > < >= <= contains matches in {}`; logical operators `and or not` (or `&& || !`). Start typing and Wireshark suggests field names. If you are lazy, right-click a field → **Apply as Filter → Selected** and it writes the filter for you. The full field list lives in the Display Filter Reference (linked at the end).
 
| Goal | Filter |
|---|---|
| One host (both directions) | `ip.addr == 192.168.1.10` |
| Source / destination only | `ip.src == x`, `ip.dst == x` |
| A whole subnet | `ip.addr == 10.0.0.0/24` |
| By port | `tcp.port == 80`, `udp.port == 53`, `tcp.port in {80 443 8080}` |
| Remove noise | `not (arp or dns or mdns or ssdp)` |
| HTTP requests | `http.request` |
| POST / URL / host | `http.request.method == "POST"`, `http.request.uri contains "login"`, `http.host contains "abc"` |
| FTP login | `ftp.request.command in {"USER" "PASS"}` |
| What DNS asked for | `dns.qry.name contains "evil"` |
| TLS Client Hello / site name | `tls.handshake.type == 1`, `tls.handshake.extensions_server_name` |
| Search text in any byte | `frame contains "flag"` |
| Case-insensitive regex | `frame matches "(?i)ctf[{]"` |
| Packets with a payload (skip bare ACKs) | `tcp.len > 0` |
| Payload Wireshark cannot identify | `data` |
| One TCP stream | `tcp.stream == 5` |
| Initial SYN (new connection / scan) | `tcp.flags.syn == 1 and tcp.flags.ack == 0` |
| RST | `tcp.flags.reset == 1` |
| Any TCP warning | `tcp.analysis.flags` |
| Packet range | `frame.number >= 100 and frame.number <= 200` |
 
A small note: since Wireshark 4.0, `ip.addr != x` correctly means "no address equals x". With older docs or older versions, write `not (ip.addr == x)` to be safe. If you reuse a filter often, click the `+` on the right side of the filter bar to save it as a button.
 
## 4. Analysis Tools
 
### Follow Stream
 
Your number-one weapon. Right-click a packet → **Follow → TCP Stream** (`Ctrl+Alt+Shift+T`). You get the whole conversation: red is client → server, blue is server → client. At the bottom, choose *Show as* ASCII / Raw / Hex Dump / C Arrays, and use **Save as** to store the raw bytes (useful for rebuilding files). There are equivalents for UDP, TLS and HTTP. After you close the window, Wireshark leaves a `tcp.stream eq N` filter applied, so remember to hit **Clear** when you want to see everything again. Otherwise you will think the pcap only has 20 packets.
 
### Statistics menu
 
- **Capture File Properties**: duration, packet count, file hash.
- **Protocol Hierarchy**: percentage per protocol. Anything with a strange share (lots of ICMP, too much DNS, a protocol nobody recognizes) deserves a closer look.
- **Conversations / Endpoints**: who talks to whom and how many bytes. Sort by Bytes to find the "chattiest" pair.
- **I/O Graph**: packets over time. A spike suggests a flood or scan; a regular rhythm may be a beacon.
- **HTTP → Requests**, **DNS**: quick summaries.
### Analyze menu
 
- **Expert Information**: collects all errors and warnings (retransmissions, duplicate IPs, ...). Open it early so you miss nothing.
- **Decode As**: force Wireshark to read a port as another protocol. Example: HTTP running on port 8888 but shown as plain TCP → Decode As, port 8888 → HTTP.
### Searching and marking
 
- `Ctrl+F`: search by display filter, hex, *String* or *Regular Expression*; choose **Packet bytes** to search inside the data. Also try *Narrow & Wide* (UTF-16), since Windows data often uses wide strings.
- `Ctrl+G` jumps to packet N, `Ctrl+M` marks a packet, `Ctrl+D` ignores it. Right-click → **Packet Comment** to leave notes.
## 5. Extracting Files from a Pcap
 
- `File → Export Objects → HTTP / SMB / TFTP / IMF / DICOM`: lists every file that was transferred; hit Save All. CTF authors love hiding images, zips and PDFs here.
- For other channels (FTP data, custom TCP): Follow Stream → *Show as Raw* → **Save as**. For a single packet: select the Data field → `File → Export Packet Bytes` (`Ctrl+Shift+X`).
- After saving, check with `file out.bin` and `xxd out.bin | head`. Quick identification by magic bytes:
| Type | Starts with |
|---|---|
| PNG | `89 50 4E 47` |
| JPEG | `FF D8 FF` |
| ZIP / docx / jar | `50 4B 03 04` |
| PDF | `25 50 44 46` (`%PDF`) |
| gzip | `1F 8B` |
| ELF | `7F 45 4C 46` |
 
If a file saved as raw from an HTTP stream will not open, it usually still has the HTTP header in front (up to the blank line `\r\n\r\n`). Cut it off and try again. Export Objects does this step for you.
 
## 6. Decrypting TLS and Wi-Fi
 
**TLS/HTTPS**: by default you only see the Client Hello (the SNI tells you which site), the certificate, and then encrypted *Application Data*. To read the content you need one of the following:
 
1. **Keylog file** (the best option, works with ECDHE too): `Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename`, then point it at the `.log` file. To generate your own keylog, set the environment variable `SSLKEYLOGFILE=~/keys.log` and launch Chrome/Firefox from that same terminal.
2. **RSA private key**: same TLS preferences page, *RSA keys list*. This only works if the session used RSA key exchange. ECDHE/DHE (forward secrecy) cannot be decrypted this way.
Once decrypted, a *Decrypted TLS* tab appears under Packet Bytes and the `http` filter starts matching. If a CTF challenge ships with a keylog file or a `.pem`, this is clearly the intended path.
 
**Wi-Fi WPA2**: you need a pcap that contains the 4-way handshake (the `eapol` filter should show 4 packets). Go to `Preferences → Protocols → IEEE 802.11 → Decryption keys → Add`, choose `wpa-pwd`, and enter `password:SSID`. No password? Try cracking the handshake with aircrack-ng or hashcat.
 
## 7. Where Flags Hide in Pcaps
 
**Plaintext (HTTP, FTP, Telnet, SMTP/POP3).** Follow Stream and you will see the credentials. With HTTP Basic Auth, the header `Authorization: Basic dXNlcjpwYXNz` is just base64 for `user:pass`. You can also try `Tools → Credentials` to have Wireshark collect them for you.
 
**Base64/hex inside payloads.** Extremely common: each packet carries a small fragment, and you stitch them together and decode. Filter with `tcp.len > 0` (or `data`). If the fragments are scattered, use tshark to pull the payloads out (section 9), join them in the right order (by packet number or `tcp.seq`, skipping retransmissions), then decode with `base64 -d` or CyberChef (its *Magic* operation can guess for you). Recognition tip: it only contains `A-Z a-z 0-9 + /` and often ends with `=`. `flag{` aligned on a 3-byte boundary encodes to `ZmxhZ3s`, but if it is shifted you get something different, so decode everything first and search afterwards.
 
**DNS tunneling/exfiltration.** Lots of queries to the same odd domain with long subdomains that look like hex/base32, or TXT responses. Try `dns.qry.name.len > 40` and `dns.qry.type == 16`. Join the labels together and decode.
 
**ICMP.** Data smuggled inside echo request/reply payloads: look for pings larger than usual and extract the `data.data` field.
 
**USB.** Keyboard captures use the `usb.capdata` field (or `usbhid.data`, depending on the version). Each report is 8 bytes: the first byte is the modifier (`02` = Shift), the third byte is the HID keycode (`04` = a, `05` = b, ...). Dump it with tshark and map the codes back to characters. For mouse captures, plot the coordinates.
 
**Hidden in headers.** One character per packet in `ip.id`, `ip.ttl`, `tcp.srcport`, `tcp.seq`, ... Extract that column with tshark and convert the numbers to ASCII.
 
**Files sent over the network.** See section 5. Once you have the file, there may be another layer: `exiftool`, `binwalk`, `strings`, and `zsteg`/`steghide` for images.
 
**Genuinely encrypted traffic.** For TLS see section 6. For SSH, give up on the content and look at metadata instead (IPs, SNI, sizes, timing).
 
**Custom protocols.** If you only see *Data*, try Decode As or Follow Stream, then guess the format (magic bytes, length fields, simple encodings such as single-byte XOR).
 
## 8. Everyday Network Analysis
 
TCP in 60 seconds: a connection opens with **SYN → SYN/ACK → ACK**; it closes politely with **FIN** and abruptly (or by refusal) with **RST**; **PSH** means "there is data, push it up now". Wireshark shows relative seq/ack numbers to keep things readable (`Preferences → Protocols → TCP → Relative sequence numbers`). Every connection gets a `tcp.stream` number.
 
| Symptom | What to look for |
|---|---|
| Port scan | One IP sending many SYNs to many ports of one target; closed ports answer RST, open ports answer SYN/ACK. Filter the initial SYNs, then check Conversations |
| SYN flood | A huge number of SYNs with no completed handshakes; spike in the I/O Graph |
| ARP spoofing | Expert Info reports a duplicate IP, `arp.duplicate-address-detected`, a stream of unsolicited ARP replies (`arp.opcode == 2`), one IP mapped to two MACs |
| Slow or lossy network | `tcp.analysis.retransmission`, `tcp.analysis.duplicate_ack`, `tcp.analysis.zero_window`; slow DNS: `dns.time > 1` |
| Password guessing | Many consecutive POST /login requests from one IP, responses of 401/403 followed by a 200 |
| Suspicious outbound connections | Conversations sorted by Bytes, unfamiliar IPs; packets sent at regular intervals |
 
## 9. tshark, the Command-Line Wireshark
 
Installed alongside Wireshark. Use it when a pcap is huge, when you need to extract fields in bulk, or when you want to plug the output into scripts. Flags worth remembering: `-r` read file, `-Y` display filter, `-T fields -e <field>` choose columns, `-q` do not print every packet, `-z` statistics, `-o` override a preference.
 
```bash
# Table of HTTP requests: packet number, source, host, URI
tshark -r cap.pcap -Y 'http.request' -T fields -e frame.number -e ip.src -e http.host -e http.request.uri
 
# Follow TCP stream 0 as ASCII
tshark -r cap.pcap -q -z follow,tcp,ascii,0
 
# Export HTTP objects to a directory
tshark -r cap.pcap --export-objects http,./out
 
# Join TCP payloads and decode hex (tr strips ':' in case your version prints it)
tshark -r cap.pcap -Y 'tcp.len > 0' -T fields -e tcp.payload | tr -d ':' | xxd -r -p
 
# If the payload pieces are base64:
tshark -r cap.pcap -Y 'tcp.len > 0' -T fields -e tcp.payload | tr -d ':' | xxd -r -p | tr -d '\n' | base64 -d
 
# List every queried domain (spot DNS exfiltration)
tshark -r cap.pcap -Y 'dns.flags.response == 0' -T fields -e dns.qry.name
 
# Statistics
tshark -r cap.pcap -q -z io,phs      # protocol hierarchy
tshark -r cap.pcap -q -z conv,tcp    # TCP conversations
 
# Decrypt TLS with a keylog file
tshark -r cap.pcap -o tls.keylog_file:keys.log -Y http -T fields -e http.request.full_uri
```
 
## 10. A Workflow for CTF Pcaps
 
1. Open the file and check **Capture File Properties** and **Protocol Hierarchy**. Is there HTTP, DNS, FTP, USB, or an unfamiliar protocol?
2. **Conversations**: pin down the one or two main conversations.
3. Quick sweep with `frame contains "flag"` or the competition's flag format (`CTF{`). Also try `Ctrl+F` with *Narrow & Wide*.
4. Not found yet? Go protocol by protocol: HTTP → Export Objects + Follow; FTP/Telnet → Follow; DNS → look for odd queries; `tcp.len > 0` or `data` → inspect payloads.
5. Spot a weird string (base64/hex/base32)? Throw it into CyberChef.
6. See TLS? Look for a keylog file or private key shipped with the challenge.
7. Got a file? Save it, then `file`, `binwalk`, `strings`, `exiftool`.
8. Still stuck: **Expert Information**, packets with unusual size or port, header fields (`ip.id`, `ttl`, ...), *malformed* packets.
9. Mark packets (`Ctrl+M`) and write Packet Comments so you do not lose your place.
## 11. Shortcuts and Common Pitfalls
 
| Key | Action |
|---|---|
| `Ctrl+/` | Jump to the filter bar |
| `Ctrl+F` / `Ctrl+G` | Find / go to packet N |
| `Ctrl+M` / `Ctrl+D` | Mark / ignore packet |
| `Ctrl+Alt+Shift+T` | Follow TCP stream (`U` = UDP, `S` = TLS, `H` = HTTP) |
| `Ctrl+Shift+X` | Export Packet Bytes |
| `Ctrl+E` / `Ctrl+K` | Start or stop capture / Capture Options |
 
Common pitfalls:
 
- Red filter bar: a wrong field name, or missing double quotes around a string (`http.host == "abc"`).
- Forgetting to Clear the filter after Follow Stream.
- Seeing *TCP segment of a reassembled PDU*: the data was split across several packets, and the full content sits in the last packet of the chain (or just use Follow Stream).
- Garbage in HTTP data: it may be gzip-compressed. Wireshark has a *Decompressed Entity Body* tab, and Export Objects decompresses automatically.
- Unusual port, so Wireshark cannot guess the protocol: use Decode As.
- Capture truncated by snaplen (Frame *length* larger than *captured length*): the tail of each packet is gone for good.
- Big file making the GUI lag: turn off name resolution, use display filters, or switch to tshark.
## 12. Resources and Practice
 
- [Wireshark User's Guide](https://www.wireshark.org/docs/)
- [Display Filter Reference](https://www.wireshark.org/docs/dfref/) (look up field names)
- [Sample captures](https://wiki.wireshark.org/SampleCaptures) to practice on
- [CyberChef](https://gchq.github.io/CyberChef/)
- [Malware-Traffic-Analysis.net](https://www.malware-traffic-analysis.net/) for real-world pcaps (open them in a VM)
- [picoCTF](https://picoctf.org/) to test yourself (the Forensics category has plenty of pcap challenges)
Self-test exercises (finish them all and you are in good shape):
 
1. Download an HTTP sample capture, find the POST request, and extract the username and password.
2. Use Export Objects to pull out an image and verify it with `file`.
3. Use tshark to print every queried domain, then pick out the most suspicious one.
4. Capture your own machine while browsing, set `SSLKEYLOGFILE`, and try decrypting the HTTPS traffic.
 
