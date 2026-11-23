# RAM (Memory) Forensics
 
Notes on acquiring and analyzing a Windows memory dump for forensic investigation.
 
## 1. Memory Acquisition
 
Tool: **DumpIt** — a lightweight Windows CLI utility.
 
```bash
# Run on the target Windows host (as Administrator)
D:\TOOL\DumpIt\x64> DumpIt.exe
# press 'y' to confirm -> produces a raw memory image (.mem / .raw)
```
 
## 2. Memory Analysis
 
Analysis is done with either:
 
- **Volatility3** (Linux/Ubuntu, CLI) — `cd volatility3`, then run plugin commands.
- **Volatility Workbench** (GUI) — load the image file and pick a plugin from the list.
### `windows.info`
 
Identifies the OS version, architecture (x86/x64) and kernel build of the host that produced the dump. Run this **first** so Volatility loads the correct symbol table before any other plugin.
 
```bash
python3 vol.py -f memory.raw windows.info
```
 
### `windows.pslist`
 
Lists the processes that were running at the moment the memory was captured.
 
| Field | Returned value | Meaning |
|---|---|---|
| PID | positive integer (e.g. `4`, `76`, `380`, `552`, `620`) | Unique identifier of the process on Windows |
| PPID (Parent Process ID) | positive integer (e.g. `4`, `520`, `744`) | ID of the parent process; important for spotting process injection or impersonation |
| ImageFileName | executable name (e.g. `System`, `smss.exe`, `lsass.exe`, `svchost.exe`) | Name of the program/binary behind the process |
| Offset (V) | hex address starting with `0x` | Virtual address of the `EPROCESS` structure representing the process in the image |
| Threads | integer or `N/A` | Number of active threads owned by the process |
| Handles | integer or `-` | Number of system resources (files, registry keys, network ports...) the process has open |
| SessionId | integer (`0`, `1`...) or `N/A` | Session the process belongs to (`0` = background services, `1+` = interactive user session) |
| Wow64 | `True` / `False` | Whether a 32-bit process is running under the WOW64 layer on 64-bit Windows |
| CreateTime | UTC timestamp (e.g. `2026-06-18 18:55:59.000000 UTC`) | Exact time the process was created |
| ExitTime | UTC timestamp or `N/A` | Time the process exited; `N/A` means it was still running at capture time |
| File output | status text (e.g. `Disabled`) | Whether automatic extraction of the process executable is enabled |
 
```bash
python3 vol.py -f memory.raw windows.pslist
```
 
### `windows.netscan`
 
Shows the network connections present in the memory image.
 
| Field | Returned value | Meaning |
|---|---|---|
| Offset | hex address (e.g. `0xaa083a7bfe98`) | Physical/virtual location of the network object in the image; used for deeper cross-referencing |
| Proto | `TCPv4`, `TCPv6`, `UDPv4`, `UDPv6` | Protocol used by the socket |
| LocalAddr | local IP (e.g. `0.0.0.0`, `127.0.0.1`, `-`) | IP of the analyzed machine; `0.0.0.0` means listening on all interfaces |
| LocalPort | port number or `-` | Local port open for serving/listening |
| ForeignAddr | remote IP (e.g. `0.0.0.0`, or `*`) | IP of the remote host; key for spotting a C2 server |
| ForeignPort | remote port or `*` | Port on the remote host |
| State | `LISTENING`, `ESTABLISHED`, or blank/`N/A` | `LISTENING` = waiting for a connection, `ESTABLISHED` = actively transferring data; blank is common for UDP (connectionless) |
| PID | integer or `-` | PID of the process that owns the connection |
| Owner | process name or `-` | Program responsible for the port/connection |
| Created | timestamp or blank | Exact time the socket object was created |
 
```bash
python3 vol.py -f memory.raw windows.netscan
```
 
### `windows.cmdline`
 
Displays the full command line each process was launched with — useful for spotting suspicious commands (base64-encoded payloads, remote downloads, hidden PowerShell, etc.).
 
```bash
python3 vol.py -f memory.raw windows.cmdline
```
 
### `windows.malfind`
 
Scans process memory regions for signs of code injection. Commonly used to catch malware hiding inside a legitimate process.
 
```bash
python3 vol.py -f memory.raw windows.malfind
```
 
### `windows.filescan` + `windows.dumpfiles`
 
`filescan` lists file objects referenced in memory, including files already deleted from disk. `dumpfiles` extracts those files for further analysis — very useful for recovering evidence (or a hidden flag) still cached in RAM.
 
```bash
python3 vol.py -f memory.raw windows.filescan
python3 vol.py -f memory.raw windows.dumpfiles --virtaddr <offset>
```
 
