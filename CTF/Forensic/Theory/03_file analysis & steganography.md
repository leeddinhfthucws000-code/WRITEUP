# Comprehensive Guide to Malicious Document Analysis for Digital Forensics
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
