# FILE ANALYSIS (Comprehensive Guide to Malicious Document Analysis for Digital Forensics)
## Overview
Malicious documents (Office files, RTF, etc.) are one of the most common initial access vectors in phishing campaigns. Attackers abuse legitimate document features such as Macros, remote templates, DDE and embedded objects to execute code on the victim's machine. This guide covers the file format structures, the attack vectors, the static and dynamic analysis workflow, and the tools investigators use to dissect suspicious documents.
 
---
## 1. Office File Format Structures
 
### 1.1. Document Format Classification
* **Description:** Microsoft Office documents belong to two structural format families:
  * **OLE / CFBF (Compound File Binary Format):** Classic binary format (Office 1997 – 2003) for `.doc`, `.xls`, `.ppt`. It works like a file system inside a file, made of **Streams** (data streams) and **Storages** (storage directories).
  * **OOXML (Office Open XML):** Modern format (Office 2007+) for `.docx`, `.xlsx`, `.pptm`, `.docm`, etc. It is a ZIP package containing XML files, images, text, and related resources, plus the `_rels` relationship schema.
* **Forensic Value:** The format decides the toolchain. ZIP-based files are unpacked with `unzip`, while binary OLE files need dedicated parsers such as `oledump.py` or `olevba`.

| Characteristic | OLE / CFBF (Legacy) | OOXML (Modern) |
| :--- | :--- | :--- |
| **File Extension** | `.doc`, `.xls`, `.ppt` | `.docx`, `.docm`, `.xlsx`, `.xlsm` |
| **Structural Nature** | Binary (Streams / Storages) | ZIP-compressed file containing XML |
| **Compatibility** | Office 1997 – 2003 | Office 2007 and later |
| **Macro Capability** | Allowed directly inside the binary file | Requires a separate format (`.docm`, `.xlsm`) |
 
### 1.2. OOXML Structure Analysis (Kali Linux)
* **Description:** Because OOXML files are ZIP archives, every component can be extracted on Linux:
```bash
  unzip sample.docx -d extracted_doc
```
* **Key Components:**
  * `docProps/`: Properties and metadata. `core.xml` stores author info and creation/modification time; `app.xml` stores the version of the application that created the file.
  * `_rels/`: Relationship map between elements. `_rels/.rels` and `word/_rels/document.xml.rels` are the priority locations to check for **Template Injection**.
  * `word/` (or `xl/`, `ppt/` depending on the app): Main content (`document.xml`), fonts, embedded images, and `vbaProject.bin` (Macro code) if present.
* **Forensic Value:** Reveals authorship, timestamps, remote links, and hidden Macro containers without ever opening the file in Office.
* **Reference:** `digital-forensics-lab/Lab 03 - vonderchild/digital-forensics-lab` on GitHub.
---
## 2. VBA Macro Analysis
 
### 2.1. Concept and Nature of Macros
* **Description:** A Macro is not a standalone tool but a feature built into Microsoft Office (Word, Excel, PowerPoint), written in **VBA (Visual Basic for Applications)**.
* **Legitimate Use:** Automating office work, complex data manipulation, or calculations based on predefined scripts.
* **Attacker Abuse:** Exploiting VBA's auto-execution. Once the victim clicks **"Enable Content"** (or **"Enable Macro"**), auto-run functions can download payloads, encrypt data, or interfere with the system.
* **Scope:** Only applies to Office files or compatible suites (`.docm`, `.xlsm`, `.pptm`, legacy `.doc`, `.xls`). It does not apply to plain text (`.txt`) or `.pdf`.
### 2.2. Static Inspection with oletools (Advanced)
* **Description:** The **oletools** suite provides the standard commands for initial triage.
* **Key Commands:**
```bash
  # Quick scan: VBA Macros, encryption, embedded OLE objects
  oleid sample.doc
 
  # Extract and analyze VBA source code
  olevba sample.doc
```
* **What `olevba` Flags Automatically:**

| Category | Examples | Forensic Value |
| :--- | :--- | :--- |
| **Auto-exec Triggers** | `AutoOpen`, `Document_Open`, `Workbook_Open` | Code that runs without further user interaction. |
| **Obfuscation Indicators** | Hidden strings, hexadecimal encoding, XOR | Attempts to evade static scanners. |
| **Suspicious I/O & Shell Calls** | `Shell`, `WScript.Shell`, `URLDownloadToFile`, `CreateObject` | Command execution, file download, or dropper behavior. |
 
---
## 3. Advanced Attack Vectors Beyond VBA Macros
 
### 3.1. Template Injection
* **Description:** The attacker edits `word/_rels/document.xml.rels` and changes the `Target` of `attachedTemplate` so it points to a remote server (C2) instead of a local template.
* **Why It Is Dangerous:** The delivered `.docx` contains no Macro, so antivirus and firewalls easily let it pass. When opened, Word silently connects out, downloads a malicious template (usually `.dotm`), and runs it without the victim noticing.
* **Where to Look:** `word/_rels/document.xml.rels`, `_rels/.rels`.
### 3.2. Dynamic Data Exchange (DDE)
* **Description:** DDE is an old Windows protocol that lets applications exchange data (e.g., Word updating data from an open Excel table). Attackers insert DDE **Field Codes** directly into the document structure.
* **Why It Is Dangerous:** It needs **no VBA Macro**. On open, Word executes the DDE command and directly calls processes such as `cmd.exe` or `powershell.exe` to download malware or take control of the machine.
### 3.3. OLE Object Embedding / Packager
* **Description:** The attacker embeds a malicious executable (`.exe`, `.bat`, `.vbs`, `.lnk`) inside an OLE/OOXML document and disguises the icon as a PDF, an image, or a harmless Word file.
* **Why It Is Dangerous:** It relies on social engineering. The victim double-clicks the fake icon, and the system extracts and runs the payload underneath.
### Attack Vector Summary
 
| Vector | Needs VBA Macro? | Trigger | Key Artifact to Inspect |
| :--- | :--- | :--- | :--- |
| **VBA Macro** | Yes | User clicks "Enable Content" | `vbaProject.bin` |
| **Template Injection** | No (remote template has it) | Opening the document | `document.xml.rels` |
| **DDE** | No | Opening the document | Field codes in `document.xml` |
| **OLE / Packager** | No | User double-clicks embedded object | Embedded OLE objects |
 
---
## 4. Legacy OLE Formats and Other Formats
 
### 4.1. Legacy OLE Binary Files with `oledump.py` (Advanced)
* **Description:** Old formats (`.doc`, `.xls`, `.ppt`) use a monolithic binary structure, so `unzip` cannot extract them. `oledump.py` analyzes the internal structure instead.
* **Key Functions:**
  * **List Streams:** Lists every data stream in the file.
  * **Detect Macros:** Streams containing VBA source code are marked with `M` or `m`.
  * **Extract Code:** View or dump the VBA source of one stream (`<stream_id>` is the stream's ordinal number).
```bash
  # List all streams
  oledump.py sample.doc
 
  # Dump VBA source from a specific stream
  oledump.py -s <stream_id> -v sample.doc
```
 
### 4.2. Rich Text Format (`.rtf`)
* **Description:** RTF is often abused to deliver Remote Code Execution (RCE) exploits against Word's memory. Classic examples: **CVE-2017-11882** and **CVE-2017-0199**.
* **Forensic Value:** Hidden embedded OLE objects inside the RTF often carry the actual exploit or payload.
* **Analysis Tool:** `rtfobj` (part of oletools) scans, parses, and extracts malicious embedded OLE objects.
```bash
  rtfobj sample.rtf
```
---

## 5. Tools Summary
 
| Tool | Target | Purpose |
| :--- | :--- | :--- |
| **unzip** | OOXML files | Extract the ZIP package to inspect XML, `_rels`, `vbaProject.bin`. |
| **oleid** | OLE / OOXML | Quick triage: Macros, encryption, embedded objects. |
| **olevba** | OLE / OOXML | Extract VBA code, flag auto-exec, obfuscation, suspicious calls. |
| **oledump.py** | Legacy OLE | List streams, detect Macro streams, dump VBA code. |
| **rtfobj** | RTF | Extract embedded OLE objects from RTF files. |
| **pcodedump.py** | VBA Stomped files | Recover P-Code when source code is stripped. |
| **Procmon / Process Explorer** | Dynamic analysis | Monitor processes, registry, file, and network activity. |

---

# STEGANOGRAPHY 
## Overview
Steganography is the practice of hiding secret information (text, images, files) inside an ordinary-looking carrier file. Audio and image files are the most common carriers, and they appear frequently in digital forensics investigations and CTF (Capture The Flag) challenges. This guide covers the main techniques and the tools used to detect and extract hidden data.
 
---
## 1. Audio Steganography
 
### 1.1. Technique Classification
* **Description:** Audio Steganography hides a secret message (text, images, or files) inside a digital audio file without significantly affecting the human listening experience. It is mainly divided into two schools of thought, based on how the data is manipulated:
  * **Spectrogram Steganography:** The frequency (pitch) and amplitude (volume) of the audio are modified over time to encode information into the soundwave. Normal listening reveals only faint static or minor background noise. When the file is loaded into analysis software (Sonic Visualiser or Audacity) and switched to Spectrogram mode, the modified frequency bands form an image or text containing the hidden message (flag).
  * **LSB (Least Significant Bit) Steganography:** Digital audio is a stream of 0s and 1s, so the least significant bits of the audio samples can be replaced with the bits of the secret payload. The LSB has the lowest weight, so the change is imperceptible to the human ear. Visual spectrogram analysis is **ineffective** here; the payload must be extracted with dedicated command-line decoding tools (such as Steghide on Kali Linux).
* **Forensic Value:** The technique decides the method. If the message is visible in a spectrogram, use visual analysis; if not, suspect LSB or an appended payload and move on to extraction tools.

| Characteristic | Spectrogram Steganography | LSB Steganography |
| :--- | :--- | :--- |
| **Where data is hidden** | Frequency / amplitude over time | Least significant bits of audio samples |
| **Human perception** | Faint static or background noise | Not perceptible |
| **Detection method** | Visual (Spectrogram view) | Statistical / tool-based decoding |
| **Typical tools** | Sonic Visualiser, Audacity | Steghide, DeepSound, SilentEye |
 
### 1.2. Audio Analysis Tools Overview
* **Description:** The most common tools for analyzing suspicious audio files:

| Tool Name | Functionality & Usage |
| :--- | :--- |
| **Audacity** | Edit, play, and visualize audio; switch to Spectrogram mode to detect hidden images, text, or embedded signals within sound waves. |
| **Sonic Visualiser** | In-depth, highly detailed spectrogram analysis, revealing complex frequency bands that standard media players cannot display. |
| **QSSTV** | Decode image signals transmitted over audio waves (Slow-Scan Television); specialized in recovering images from strange audio tracks in CTF challenges. |
| **Steghide** | Command-line tool to embed or extract secret payloads (text, compressed files) inside audio (or image) files using a passphrase. |
| **HxD (Hex Editor)** | Inspect raw binary code, verify Header/Footer signatures of audio files, and identify appended files (EOF - End of File). |
| **Baudline** | Ultra-fast time-frequency spectrum analysis, digital signal processing (DSP), and real-time spectrogram viewing. |
| **SoX (Sound eXchange)** | Convert audio formats, apply frequency filters, and quickly analyze technical file attributes via command line. |
| **GNU Radio** | DSP toolkit for analyzing complex waveforms, radio transmissions, and advanced audio signals. |
| **SilentEye / DeepSound** | GUI software for rapidly hiding or extracting secret data within common audio formats such as WAV. |
 
### 1.3. Audio Analysis Workflow (Kali Linux)
* **Description:** A typical triage order for a suspicious audio file:
```bash
# 1. Check basic file attributes and metadata
file sample.wav
exiftool sample.wav
soxi sample.wav
 
# 2. Look for readable strings and appended data
strings sample.wav | less
binwalk sample.wav
 
# 3. Generate a spectrogram image with SoX (or open in Sonic Visualiser / Audacity)
sox sample.wav -n spectrogram -o spectrogram.png
 
# 4. Try to extract an LSB / Steghide payload (a passphrase may be required)
steghide info sample.wav
steghide extract -sf sample.wav
```
* **Forensic Value:** Spectrogram review catches visual payloads, while `strings`, `binwalk`, and `steghide` cover appended files and LSB-embedded data.
---
## 2. Image Steganography
 
### 2.1. Technique Classification
* **Description:** Image Steganography embeds secret information directly into the pixel data of a digital image. Since altering image bits changes color values, there are two main strategies:
  * **Invisible Bit Modification (Standard LSB):** Only modifies the least significant bits of the RGB color channels, so the tint variation is completely imperceptible to human vision.
  * **Intentional Visual Modification:** Alters bits in fixed pixel areas to form hidden patterns or contrast bands that only become visible when contrast or color filters are boosted to maximum levels.
* **Forensic Value:** Steganalysis (forensic analysis of images) relies on CLI tools on Linux environments such as Kali Linux. The image format decides which tool to use.

| Image Format | Recommended Tool | Reason |
| :--- | :--- | :--- |
| **PNG** | `zsteg` | Lossless format preserves LSB data |
| **JPEG** | `steghide` | Lossy compression destroys plain LSB data |
| **Any** | `exiftool` | Checks metadata outside the pixel data |
 
### 2.2. Image Analysis Tools (Kali Linux)
* **PNG Format (zsteg):** A Ruby-based tool that automatically scans all color planes and LSB bit positions, extracting hidden strings and data directly to the terminal.
```bash
sudo gem install zsteg
zsteg -a image.png
```
* **JPEG Format (steghide):** JPEG compression is lossy and degrades plain LSB data, so `steghide` is used to embed and extract secret payloads based on graph-theory value-swapping algorithms.
```bash
sudo apt install steghide
steghide info image.jpg
steghide extract -sf image.jpg
```
* **Metadata Analysis (exiftool):** A standard utility to inspect file headers and metadata tags (such as `Comment`, `Artist`, or `Description`) and check whether secret flags are stored in plain text outside the pixel data.
```bash
exiftool image.png
```
 
### 2.3. Additional Checks
* **Description:** Quick checks that complement the tools above:
```bash
# Look for readable strings and files appended after the image end marker
strings image.png | less
binwalk image.png
 
# Inspect raw bytes and header / footer signatures
xxd image.png | head
```
* **Forensic Value:** Mismatched headers/footers or extra data after the end-of-file marker often indicate an appended payload.
---
## 3. Summary
 
| Carrier | Technique | Detection / Extraction |
| :--- | :--- | :--- |
| **Audio** | Spectrogram | Sonic Visualiser, Audacity (Spectrogram mode) |
| **Audio** | LSB | Steghide, DeepSound, SilentEye |
| **Audio** | Slow-Scan TV signal | QSSTV |
| **Image (PNG)** | LSB | `zsteg` |
| **Image (JPEG)** | Value swapping | `steghide` |
| **Image (any)** | Metadata | `exiftool` |
 
