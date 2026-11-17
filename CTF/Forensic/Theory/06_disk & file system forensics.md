# DISK & FILE SYSTEM FORENSICS 
## Overview
Disk & File System Forensics is a foundational discipline of Digital Forensics focused on the acquisition, preservation, recovery, and examination of artifacts residing on non-volatile storage media (HDDs, SSDs, USB flash drives, memory cards, and virtual disk images) to uncover admissible evidence during Incident Response (IR) investigations and Capture The Flag (CTF) challenges.

---

## 1. Core Forensic Workflow & Physical Storage Architecture

### 1.1. Golden Rules of Forensic Disk Examination
* **Description:** Strict procedural controls required to maintain evidence admissibility and prevent accidental metadata alteration.
  * **Never Operate on Original Media:** All forensic examination must be conducted exclusively on a verified bit-stream copy (Forensic Image). Mounting an active volume natively on Windows or Linux alters timestamps and journal states.
  * **Enforce Write Protection:** Always deploy a certified Hardware Write Blocker (or mount images strictly in `read-only` / `ro,noload` mode) during acquisition and triage.
  * **Verify Cryptographic Integrity:** Compute and document hash digests (`MD5`, `SHA-1`, `SHA-256`) prior to and immediately following imaging to validate zero bit-level modification.

### 1.2. Forensic Disk Acquisition Formats
* **Description:** Unlike standard logical copies that skip deleted entries and unallocated space, forensic acquisition performs a sequential bit-stream copy from `LBA 0` to the final physical sector.
* **Forensic Value:** Preserves unallocated clusters, file slack, partition gaps, and corrupted file system structures.

| Image Format | File Extension | Structural Nature | Compression & Metadata |
| :--- | :--- | :--- | :--- |
| **Raw / DD Image** | `.dd`, `.raw`, `.img` | Exact `1:1` bit-for-bit binary replica matching physical drive size | No compression; no embedded case metadata |
| **Expert Witness Format** | `.E01` | Industry-standard EnCase/FTK segmented forensic container | Lossless compression; stores hashes & chain-of-custody logs |
| **Advanced Forensic Format** | `.AFF`, `.AFF4` | Open-source, extensible forensic image container | High-speed compression; flexible metadata streams |

### 1.3. Storage Allocation Units & Latent Data Regions
* **Sector:** The smallest addressable physical unit on magnetic platters or NAND flash media, historically `512 bytes` (modern Advanced Format drives use `4096 bytes` / `4KB`).
* **Cluster (Windows) / Block (Linux):** The smallest logical allocation unit managed by a file system, consisting of contiguous sectors (e.g., `1 cluster = 8 sectors = 4096 bytes`).
* **Unallocated Space:** Clusters or blocks not currently indexed by any active file record; acts as the primary repository for deleted file payloads prior to overwrite.
* **Slack Space (File Slack):** Unused trailing bytes within an allocated cluster when a file's logical size is smaller than the cluster boundary (e.g., a `1,000-byte` file in a `4,096-byte` cluster leaves `3,096 bytes` of slack space containing residual data from prior allocations).

### 1.4. Partitioning Schemes: MBR vs. GPT
* **Description:** Low-level disk structures defining volume boundaries, boot code, and partition types.

| Characteristic | MBR (Master Boot Record) | GPT (GUID Partition Table) |
| :--- | :--- | :--- |
| **Storage Location** | First sector of disk (`LBA 0` — initial `512 bytes`) | `LBA 0` (Protective MBR); `LBA 1` (GPT Header); `LBA 2–33` (Entries); mirrored Backup GPT at end of disk |
| **Partition Capacity** | Maximum `4` Primary Partitions (or `3` Primary + `1` Extended containing Logical volumes) | Up to `128` primary partitions by default on Windows without extended/logical workarounds |
| **Maximum Disk Size** | `2 TB` maximum (`32-bit` LBA addressing with `512-byte` sectors) | Up to `9.4 ZB` theoretical limit (`64-bit` LBA addressing) |
| **Magic Signature** | Trailing `2 bytes` at offset `0x1FE` are always `0x55 0xAA` | ASCII string `EFI PART` (`45 46 49 20 50 41 52 54`) at start of `LBA 1` (offset `0x200`) |

---

## 2. Anatomy of the NTFS File System (Windows)

### 2.1. Master File Table (`$MFT`) & Record Structure
* **Description:** NTFS operates on the principle that *"everything on the volume is a file—including the file system's own structural metadata."*
  * **Record Size & Magic Header:** Every file and directory has at least one fixed-size `1024-byte` (`2-sector`) MFT Record beginning with the magic signature `FILE` (`46 49 4C 45`).
  * **System Metadata Records (`0–15`):** Reserved for core structures including `$MFT` (`Record 0`), `$MFTMirr` (`Record 1`), `$LogFile` (`Record 2`), `$Volume` (`Record 3`), Root Directory `.` (`Record 5`), and `$Bitmap` (`Record 6`).
  * **Allocation Flags (Offset `0x16`):** `0x00` (Deleted File), `0x01` (Active File), `0x02` (Deleted Directory), `0x03` (Active Directory).

### 2.2. Key MFT Record Attributes
* **`$STANDARD_INFORMATION` (`0x10`):** Stores primary `MACB` timestamps (Creation, Modified, MFT Modified, Last Accessed), DOS file attributes (`Read-only`, `Hidden`, `System`), and Security ID. Vulnerable to user-mode timestomping.
* **`$FILE_NAME` (`0x30`):** Stores `UTF-16LE` filename, Parent Directory MFT Record Reference, logical/physical size, and a secondary kernel-managed set of `MACB` timestamps resistant to standard API tampering.
* **`$DATA` (`0x80`):** Holds actual file content:
  * **Resident Data:** Payloads smaller than `~700 bytes` are embedded directly inside the `1024-byte` MFT Record without consuming external clusters.
  * **Non-Resident Data:** Larger files store cluster pointers (**Data Runs**) referencing external allocated clusters across the volume.
* **`$INDEX_ROOT` (`0x90`) & `$INDEX_ALLOCATION` (`0xA0`):** Maintain B-tree directory index structures mapping child files and subdirectories.

### 2.3. Alternate Data Streams (ADS) & Transaction Journals
* **Alternate Data Streams (ADS):** Allows a single file to contain multiple `$DATA` attributes (e.g., `benign.txt:hidden_payload.exe`). Adversaries abuse ADS to conceal malware or staged data while keeping the host file's visible size unchanged in Windows Explorer.
* **`$LogFile` & `$UsnJrnl` (Change Journal):** Transactional logs tracking file creation, modification, renaming, and deletion events—critical for proving file existence even after MFT records are overwritten.

---

## 3. Anatomy of the Ext4 File System (Linux)

### 3.1. Structural Components
* **Superblock:** Located at a fixed offset of `1024 bytes` from the start of the partition; stores global metadata (total Inode count, total Block count, block size, inodes per group, mount state). Redundant backup Superblocks are mirrored across specific Block Groups for recovery.
* **Inode (Index Node):** Analogous to an MFT Record; uniquely identified by an **Inode Number** and stores file permissions (`mode`), `UID`/`GID`, byte size, timestamps (`atime`, `mtime`, `ctime`, `crtime`), and **Extent trees** pointing to data blocks. **Inodes do not store filenames.**
* **Directory Entry (`Dentry`):** A directory file acting as a lookup table mapping `[Filename]` strings to `[Inode Numbers]`, enabling multiple filenames (**Hard Links**) to reference the same underlying Inode.
* **Extents & Journaling (`jbd2`):** Ext4 uses Extent trees to map contiguous block ranges efficiently, while the `jbd2` journal records pending metadata/data transactions.

---

## 4. Data Recovery & Artifact Analysis Techniques

### 4.1. Mechanics of File Deletion
* **Description:** Executing `Shift + Delete` (Windows) or `rm` (Linux) unlinks metadata pointers rather than zeroing out physical data blocks to preserve I/O performance.
  * **NTFS Deletion:** Flips the MFT Record allocation flag from `0x01` to `0x00` and marks corresponding clusters as free in `$Bitmap`. Data runs remain intact until the MFT slot is reused.
  * **Ext4 Deletion:** Unlinks the directory entry, clears bits in the Inode/Block Bitmaps, and—on modern Linux kernels—zeroes out Extent pointers inside the deleted Inode, making metadata-based recovery of large files difficult without carving.

### 4.2. File Carving & Magic Byte Signatures
* **Description:** Signature-driven recovery scanning raw binary streams across disk images or unallocated space using known **Header** and **Footer** markers, independent of corrupted or missing file system metadata.

| File Format | Header (Magic Bytes) | Footer (Trailer) | Forensic Identification Notes |
| :--- | :--- | :--- | :--- |
| **JPEG / JPG** | `FF D8 FF E0` / `FF D8 FF E1` | `FF D9` | Primary target in unallocated image recovery tasks |
| **PNG** | `89 50 4E 47 0D 0A 1A 0A` | `49 45 4E 44 AE 42 60 82` | Footer matches the `IEND` chunk and its CRC checksum |
| **PDF** | `25 50 44 46` (`%PDF-`) | `25 25 45 4F 46` (`%%EOF`) | Incremental saves may produce multiple `%%EOF` markers |
| **ZIP / DOCX / XLSX** | `50 4B 03 04` (`PK..`) | `50 4B 05 06` (`+ 18 bytes`) | Modern Office Open XML documents are ZIP containers |
| **SQLite Database** | `53 51 4C 69 74 65 20 66` | *No fixed footer* | Stores browser history, messaging logs, and app states |
| **Windows PE (`.exe`)** | `4D 5A` (`MZ`) | *No fixed footer* | Includes DOS stub string `This program cannot be run in DOS mode` |

### 4.3. Temporal Artifact Analysis (`MACB` Timestamps) & Timestomping
* **M — Modified:** Updated when the file's internal data payload is altered.
* **A — Accessed:** Updated when the file is read or opened (disabled by default on modern Windows volumes to minimize SSD write wear).
* **C — Changed / MFT Modified:** Updated when permissions, ownership, filename, or MFT/Inode metadata attributes change.
* **B — Birth / Created:** Timestamp recording when the file was instantiated on the current volume.
* **Detecting Timestomping (Anti-Forensics):**
  * Adversaries use utilities like `timestomp` to backdate malicious binaries, typically modifying only `$STANDARD_INFORMATION` (`0x10`).
  * Cross-check `$STANDARD_INFORMATION` against `$FILE_NAME` (`0x30`): if `$STANDARD_INFORMATION` creation time predates `$FILE_NAME` creation time, or if nanosecond sub-second fields are zero-padded (`.0000000`), timestomping is strongly indicated.

---

## 5. Essential Disk Forensics Toolchain

| Category | Representative Tools | Operational Use Cases in IR & CTF |
| :--- | :--- | :--- |
| **Disk Acquisition** | `FTK Imager`, `dc3dd`, `dd`, `Magnet DumpIt` | Create verified bit-stream images (`.dd`, `.E01`), preview partitions, and extract locked system files (`$MFT`, Registry hives) |
| **Forensic Suites** | `Autopsy`, `The Sleuth Kit` (`fls`, `icat`, `mmls`, `fsstat`) | Enumerate partition tables, recover deleted files, generate super-timelines, and perform indexed keyword searches |
| **NTFS / MFT Parsers** | `MFTECmd`, `AnalyzeMFT`, `Timeline Explorer` | Parse raw `$MFT` records into CSV timelines to filter file activity and detect timestomping anomalies |
| **Hex Editors** | `HxD`, `010 Editor`, `Bless` | Inspect raw `MBR`/`GPT` structures byte-by-byte, repair corrupted magic headers, and examine file slack |
| **File Carving** | `Foremost`, `Scalpel`, `PhotoRec`, `Binwalk` | Carve embedded or deleted files from raw binary streams using custom header/footer definitions |
