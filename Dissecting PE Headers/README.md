# TryHackMe — Dissecting PE Headers

**Category:** Malware Analysis
**Difficulty:** Medium
**Platform:** TryHackMe

---

## Overview

The **Portable Executable (PE)** format is the standard executable file format used by Windows for applications, DLLs, and other executable components.

Understanding the PE structure is an important skill in **malware analysis and reverse engineering**, as PE headers provide information about:

* File architecture
* Entry point
* Sections
* Memory permissions
* Imported APIs
* Resources
* Relocation information
* Potential packing and obfuscation

This room focuses on analyzing PE files using tools such as `pe-tree`, `pecheck`, and `wxHexEditor`.

---

# Task 1 — Introduction

Windows executables commonly use the `.exe` extension. These files use the **Portable Executable (PE)** format.

The PE format is based on the **Common Object File Format (COFF)** and defines how Windows executable files are structured and loaded into memory.

From a malware-analysis perspective, understanding this structure allows analysts to extract useful information from a binary without executing it.

### Topics Covered

* PE file structure
* PE headers
* Reading PE header fields
* PE sections
* Imported APIs
* Identifying packed executables

> **Prerequisite:** The room recommends completing **Intro to Malware Analysis** before starting this room.

### Answer

```text
No answer needed
```

---

# Task 2 — Overview of PE Headers

A PE file is ultimately stored on disk as a sequence of bytes.

Opening the file with a hex editor exposes these bytes as hexadecimal values. Although this provides the raw data, manually interpreting an entire PE file is impractical.

Tools such as `pe-tree` make the structure easier to analyze.

### Important PE Structures

The main structures examined in this room are:

```text
IMAGE_DOS_HEADER
IMAGE_NT_HEADERS
FILE_HEADER
OPTIONAL_HEADER
IMAGE_SECTION_HEADER
IMAGE_IMPORT_DESCRIPTOR
```

A simplified view of the PE structure is:

```text
PE File
│
├── IMAGE_DOS_HEADER
│
├── DOS_STUB
│
├── IMAGE_NT_HEADERS
│   │
│   ├── Signature
│   ├── FILE_HEADER
│   └── OPTIONAL_HEADER
│
├── IMAGE_SECTION_HEADER
│   ├── .text
│   ├── .data
│   ├── .rdata
│   └── .rsrc
│
└── Import Information
```

These structures are represented using **C-style structures (`STRUCT`)** containing multiple fields.

> **Key takeaway:** The objective is to understand the PE format rather than become dependent on a specific analysis tool.

### Answer

**Question:** What data type are the PE headers?

```text
STRUCT
```

---

# Task 3 — IMAGE_DOS_HEADER and DOS_STUB

## IMAGE_DOS_HEADER

The `IMAGE_DOS_HEADER` occupies the first **64 bytes** of a PE file.

One of its most recognizable fields is the `e_magic` value.

At the beginning of a PE file, the raw bytes are:

```text
4D 5A
```

When interpreted as ASCII:

```text
MZ
```

### MZ Signature

`MZ` refers to **Mark Zbikowski**, one of the Microsoft architects associated with the MS-DOS executable format.

The signature identifies the file as an executable format compatible with the PE loading process.

In `pe-tree`, the value is represented as:

```text
e_magic = 0x5A4D
```

The apparent reversal is caused by **little-endian byte ordering**.

---

## e_lfanew

Another important field in the DOS header is:

```text
e_lfanew
```

This field contains the **file offset of the `IMAGE_NT_HEADERS`**.

For example:

```text
e_lfanew = 0x000000D8
```

means the NT headers begin at:

```text
0x000000D8
```

in the file.

This field is particularly useful when manually navigating a PE file in a hex editor.

---

## DOS_STUB

Immediately following the DOS header is the **DOS Stub**.

A common message stored within the DOS Stub is:

```text
This program cannot be run in DOS mode.
```

The DOS Stub exists primarily for backward compatibility with DOS environments.

If the PE executable is executed in an incompatible DOS environment, the stub displays the message instead of executing the Windows program.

---

## Entropy

PE-analysis tools may also display **entropy**.

Entropy measures the randomness of data:

```text
Low entropy  → More predictable data
High entropy → More random-looking data
```

High entropy can be an indicator of:

* Compression
* Encryption
* Packing
* Obfuscation

However, entropy should never be treated as definitive proof of packing by itself.

### Answers

| Question                             | Answer           |
| ------------------------------------ | ---------------- |
| Size of `IMAGE_DOS_HEADER`           | `64 bytes`       |
| What does MZ stand for?              | `Mark Zbikowski` |
| Variable containing NT header offset | `e_lfanew`       |
| NT header address of `zmsuz3pinwl`   | `0x000000F8`     |

---

# Task 4 — IMAGE_NT_HEADERS

The `IMAGE_NT_HEADERS` contains important information required to interpret the PE file.

Its main components are:

```text
IMAGE_NT_HEADERS
│
├── Signature
├── FILE_HEADER
└── OPTIONAL_HEADER
```

---

## PE Signature

The NT headers begin at the offset specified by:

```text
IMAGE_DOS_HEADER.e_lfanew
```

The first four bytes are:

```text
50 45 00 00
```

The first two bytes correspond to:

```text
PE
```

This signature identifies the beginning of the PE/NT header structure.

---

## FILE_HEADER

The `FILE_HEADER` contains information describing the PE file.

Important fields include:

| Field                  | Purpose                        |
| ---------------------- | ------------------------------ |
| `Machine`              | Target CPU architecture        |
| `NumberOfSections`     | Number of PE sections          |
| `TimeDateStamp`        | Compilation timestamp          |
| `PointerToSymbolTable` | COFF symbol table location     |
| `NumberOfSymbols`      | Number of COFF symbols         |
| `SizeOfOptionalHeader` | Size of the optional header    |
| `Characteristics`      | Properties and flags of the PE |

### Machine

The `Machine` field identifies the target architecture.

For example:

```text
i386
```

indicates a **32-bit Intel architecture**.

### NumberOfSections

This specifies how many sections are present in the PE file.

These sections may contain:

* Executable code
* Data
* Resources
* Import information
* Relocation information

### TimeDateStamp

The `TimeDateStamp` stores the timestamp associated with the binary.

### Characteristics

This field contains flags describing properties of the PE, such as whether it is:

* An executable image
* Designed for a specific architecture
* Using relocation information
* Containing debugging or symbol information

### Answers

| Question                                                       | Answer                        |
| -------------------------------------------------------------- | ----------------------------- |
| In the attached VM, there is a file                                                            |
| Desktop\Samples\zmsuz3pinwl. Open this file in pe-tree.        |  `32-bit machine`             |
| Is this PE file compiled for a 32-bit machine or a 64-bit                                      |
| machine?                                                                                       |
|                                                                                                |                                
| Timestamp of the file                                          | `Wed Mar 9 12:27:49 2022 UTC` |

---

# Task 5 — OPTIONAL_HEADER

The `OPTIONAL_HEADER` is a major component of `IMAGE_NT_HEADERS`.

Despite its name, it contains several critical fields used by Windows to load and execute the PE.

---

## Magic

The `Magic` field identifies whether the PE uses the 32-bit or 64-bit PE format.

| Magic    | Architecture |
| -------- | ------------ |
| `0x010B` | 32-bit PE32  |
| `0x020B` | 64-bit PE32+ |

Therefore:

```text
0x010B → 32-bit
0x020B → 64-bit
```

---

## AddressOfEntryPoint

`AddressOfEntryPoint` specifies the **Relative Virtual Address (RVA)** where execution begins.

It identifies the location of the first code executed when the PE is loaded.

Conceptually:

```text
Entry Point Address
        ↓
ImageBase + EntryPoint RVA
```

The entry point is particularly important during malware reverse engineering because it provides the starting point for examining program execution.

---

## ImageBase

`ImageBase` specifies the preferred virtual memory address at which the PE should be loaded.

A common default for 32-bit executables is:

```text
0x00400000
```

If the preferred address cannot be used, relocation information may be required.

---

## BaseOfCode and BaseOfData

`BaseOfCode` specifies the location of the code section relative to the image base.

`BaseOfData` specifies the location of the data section in PE32 files.

> **Note:** `BaseOfData` is not present in PE32+ headers.

---

## Subsystem

The `Subsystem` field identifies the environment required by the executable.

Common values include:

```text
0x0002 → WINDOWS_GUI
0x0003 → WINDOWS_CUI
```

`WINDOWS_CUI` refers to a console-based Windows application.

---

## DataDirectory

The `DataDirectory` contains references to important PE data structures.

These directories can provide information about:

* Imports
* Exports
* Resources
* Relocations
* TLS
* Debug information

Import information is particularly useful during malware analysis because imported APIs can provide clues about potential functionality.

### Answers

| Question                           | Answer               |
| ---------------------------------- | -------------------- |
| Field identifying 32/64-bit format | `Magic`              |
| Magic value for 64-bit             | `0x020B`             |
| Subsystem of `zmsuz3pinwl`         | `0x0003 WINDOWS_CUI` |

---

# Task 6 — IMAGE_SECTION_HEADER

PE files divide their data into multiple **sections**.

Each section can contain different types of information required by the executable.

The `IMAGE_SECTION_HEADER` describes these sections.

---

## Common PE Sections

| Section  | Typical Purpose             |
| -------- | --------------------------- |
| `.text`  | Executable code             |
| `.data`  | Initialized read/write data |
| `.rdata` | Read-only data              |
| `.idata` | Import-related information  |
| `.reloc` | Relocation information      |
| `.rsrc`  | Resources                   |
| `.ndata` | Uninitialized data          |

---

## .text

The `.text` section generally contains executable code.

Typical permissions:

```text
READ
EXECUTE
```

It normally does not require write permissions.

---

## .data

The `.data` section commonly contains initialized writable data.

Typical permissions:

```text
READ
WRITE
```

---

## .rdata / .idata

These sections may contain read-only data and import-related information.

Import information allows the executable to use functionality provided by external DLLs.

---

## .reloc

The `.reloc` section contains relocation information used when the PE cannot be loaded at its preferred address.

---

## .rsrc

The `.rsrc` section contains application resources such as:

* Icons
* Images
* Dialogs
* Menus
* Version information

---

## Important Section Header Fields

| Field             | Description                        |
| ----------------- | ---------------------------------- |
| `VirtualAddress`  | RVA of the section in memory       |
| `VirtualSize`     | Size of the section in memory      |
| `SizeOfRawData`   | Size of the section on disk        |
| `Characteristics` | Section permissions and properties |

### Section Permissions

Common characteristics include:

```text
READ
WRITE
EXECUTE
```

A section with:

```text
READ + WRITE + EXECUTE
```

is worth investigating because writable executable memory can be associated with unusual code-loading or unpacking behavior.

However, permissions alone do not prove that a file is malicious or packed.

### Answers

| Question                            | Answer                                                    |
| ----------------------------------- | --------------------------------------------------------- |
| Number of sections in `zmsuz3pinwl` | `7`                                                       |
| `.rsrc` characteristics             | `0xE0000040 INITIALIZED_DATA \| EXECUTE \| READ \| WRITE` |

---

# Task 7 — IMAGE_IMPORT_DESCRIPTOR

Windows executables frequently rely on functionality provided by external DLLs rather than implementing everything internally.

The `IMAGE_IMPORT_DESCRIPTOR` contains information about the PE's imported DLLs and functions.

---

## Why Imports Matter

Imported APIs can provide useful clues about the potential functionality of a binary.

For example:

```text
CreateFile
```

may indicate file-related operations.

```text
CreateProcessW
```

may indicate process creation.

```text
CreateDirectoryW
```

may indicate directory creation.

```text
WriteFile
```

may indicate file-writing activity.

> **Important:** An imported API indicates that the program has access to that functionality. It does not, by itself, prove that the function is actually executed.

---

## Imported DLLs

The `redline` sample imports functions from DLLs including:

```text
ADVAPI32.dll
SHELL32.dll
ole32.dll
COMCTL32.dll
USER32.dll
```

These DLLs provide Windows API functionality to the executable.

---

## Import Address Table

Two important fields involved in import resolution are:

```text
OriginalFirstThunk
FirstThunk
```

Windows uses these structures when resolving imported functions and constructing the **Import Address Table (IAT)**.

The IAT ultimately contains the addresses used by the executable to call imported functions.

### Answer

**Question:** `redline` imports `CreateWindowExW`. Which DLL provides this function?

```text
User32.dll
```

---

# Task 8 — Packing and Identifying Packed Executables

## What Is Packing?

A **packer** transforms a PE executable so that its original code and data are more difficult to analyze statically.

When the packed program executes, an unpacking routine can reconstruct the original code in memory.

Packing can be used legitimately for:

* Software protection
* Compression
* Anti-reverse-engineering

Malware authors may also use packers to:

* Obfuscate malicious code
* Hide strings
* Evade signature-based detection
* Complicate static analysis

---

## Identifying Packed Executables

There is no single definitive indicator of packing.

Instead, analysts look for multiple suspicious characteristics.

---

### 1. Unusual Section Names

Typical PE sections include:

```text
.text
.data
.rdata
.rsrc
.reloc
```

Packed executables may contain:

* Unusual section names
* Random-looking names
* Empty section names
* Unexpected section layouts

The `zmsuz3pinwl` sample contains unconventional/unnamed sections.

---

### 2. High Entropy

Packed or encrypted data often has high entropy.

For example:

```text
Entropy: 7.999788
```

is very close to the theoretical maximum of `8`.

This suggests highly random-looking data.

> **Note:** High entropy can indicate packing or encryption, but it is not conclusive by itself.

---

### 3. Multiple Executable Sections

Normal PE files commonly place executable code in `.text`.

If several sections have:

```text
READ
WRITE
EXECUTE
```

permissions, the file deserves further investigation.

This can occur because an unpacking routine needs to write reconstructed code into memory before executing it.

---

### 4. SizeOfRawData vs VirtualSize

Two useful fields are:

```text
SizeOfRawData
VirtualSize
```

`SizeOfRawData` represents the section size on disk.

`VirtualSize` represents the required size when loaded into memory.

A significant difference between these values can be suspicious, particularly when combined with writable and executable section permissions.

---

### 5. Very Few Imports

Packed executables often expose fewer imports than their unpacked counterparts.

Functions frequently associated with runtime API resolution include:

```text
GetProcAddress
GetModuleHandleA
LoadLibraryA
```

These functions allow a program to dynamically locate and load APIs during execution.

Consequently, the actual functionality of the binary may not be obvious from its import table.

---

## pecheck Analysis

The `pecheck` utility can be used to inspect PE sections and calculate entropy.

Example:

```bash
cd Desktop/Samples
pecheck zmsuz3pinwl
```

The output can reveal:

```text
Entropy
IMAGE_SECTION_HEADER
VirtualAddress
VirtualSize
SizeOfRawData
Characteristics
```

For example:

```text
Characteristics: 0xE0000040
```

corresponds to:

```text
IMAGE_SCN_CNT_INITIALIZED_DATA
IMAGE_SCN_MEM_EXECUTE
IMAGE_SCN_MEM_READ
IMAGE_SCN_MEM_WRITE
```

This indicates that the section is:

```text
READ + WRITE + EXECUTE
```

and contains initialized data.

---

## Packed Executable Indicators

| Indicator                                         | Why It Matters                               |
| ------------------------------------------------- | -------------------------------------------- |
| Unusual section names                             | May indicate a non-standard PE layout        |
| Empty/random section names                        | Can be associated with packers               |
| High entropy                                      | May indicate compression or encryption       |
| Multiple executable sections                      | May indicate unpacking/runtime code          |
| RWX sections                                      | Worth investigating for runtime modification |
| Large `VirtualSize` vs `SizeOfRawData` difference | May indicate runtime unpacking               |
| Very few imports                                  | Functionality may be resolved dynamically    |

### Answer

**Question:** Which file in `Desktop/Samples` appears to be packed?

```text
zmsuz3pinwl
```

---

# Task 9 — Conclusion

This room provided an introduction to **Windows PE file analysis** and demonstrated how PE metadata can assist malware analysts during static analysis.

## Key Takeaways

* `IMAGE_DOS_HEADER` contains the DOS header and `MZ` signature.
* `e_lfanew` points to the `IMAGE_NT_HEADERS`.
* `IMAGE_NT_HEADERS` contains the `FILE_HEADER` and `OPTIONAL_HEADER`.
* `Magic` identifies whether the PE is 32-bit or 64-bit.
* `AddressOfEntryPoint` identifies where execution begins.
* `IMAGE_SECTION_HEADER` describes the PE's sections and their permissions.
* Imported APIs can provide clues about potential program functionality.
* Entropy can help identify compressed, encrypted, or packed data.
* Unusual sections, RWX permissions, high entropy, and limited imports can indicate packing.
* Multiple indicators should be combined before concluding that a binary is packed.

> **Key takeaway:** Understanding the PE format allows a malware analyst to extract valuable information from a Windows binary before executing it.

---

# Tools Used

```text
pe-tree
pecheck
wxHexEditor
Hex Editor
```

---

# TryHackMe Answers

| Task | Question                        | Answer                                                    |
| ---- | ------------------------------- | --------------------------------------------------------- |
| 1    | Intro to Malware Analysis       | `No answer needed`                                        |
| 2    | PE header data type             | `STRUCT`                                                  |
| 3    | `IMAGE_DOS_HEADER` size         | `64`                                                      |
| 3    | MZ stands for                   | `Mark Zbikowski`                                          |
| 3    | NT header offset variable       | `e_lfanew`                                                |
| 3    | `zmsuz3pinwl` NT header address | `0x000000F8`                                              |
| 4    | Architecture                    | `32-bit machine`                                          |
| 4    | TimeDateStamp of the file       | `0x62289d45 Wed Mar  9 12:27:49 2022 UTC`                                              |
| 5    | Architecture field              | `Magic`                                                   |
| 5    | 64-bit Magic value              | `0x020B`                                                  |
| 5    | Subsystem                       | `0x0003 WINDOWS_CUI`                                      |
| 6    | Number of sections              | `7`                                                       |
| 6    | `.rsrc` characteristics         | `0xE0000040 INITIALIZED_DATA \| EXECUTE \| READ \| WRITE` |
| 7    | `CreateWindowExW` DLL           | `User32.dll`                                              |
| 8    | Packed executable               | `zmsuz3pinwl`                                             |
| 9    | Social channels                 | `No answer needed`                                        |

---

# Skills Practiced

```text
Windows PE Analysis
Static Malware Analysis
PE Header Analysis
Hexadecimal Analysis
Windows Internals
API Import Analysis
Entropy Analysis
Packed Executable Detection
Reverse Engineering Fundamentals
```
