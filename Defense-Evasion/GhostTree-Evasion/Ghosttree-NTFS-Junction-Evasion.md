## Table of Contents

1. [Introduction](#1-introduction)
2. [NTFS Reparse Points Fundamentals](#2-ntfs-reparse-points-fundamentals)
3. [Junctions vs Symbolic Links](#3-junctions-vs-symbolic-links)
4. [Why Junctions Became Attractive for Evasion](#4-why-junctions-became-attractive-for-evasion)
5. [Understanding the Windows MAX\_PATH Limitation](#5-understanding-the-windows-max_path-limitation)
6. [From GhostBranch to GhostTree](#6-from-ghostbranch-to-ghosttree)
7. [Understanding the "126 Levels" Limitation](#7-understanding-the-126-levels-limitation)
8. [Practical Lab Scenario](#8-practical-lab-scenario)
9. [Detection & Defensive Considerations](#9-detection--defensive-considerations)
10. [Final Thoughts](#10-final-thoughts)

---

## 1. Introduction

Recently, a filesystem-based defense evasion concept referred to as **GhostTree** started gaining attention across the cybersecurity community. Unlike traditional evasion methods that rely on encryption, shellcode obfuscation, or memory corruption, GhostTree abuses the logical behavior of **filesystem traversal** itself.

### What GhostTree Is NOT

> One important distinction should be made early.

GhostTree is **not**:

- a malware family
- a privilege escalation technique
- a kernel exploit
- a memory corruption vulnerability
- a classic signature bypass

It also does **not** introduce a new NTFS primitive or filesystem capability.

### What GhostTree IS

GhostTree is more accurately described as a **methodology** that combines:

- existing NTFS features
- recursive traversal behavior
- filesystem graph abuse

...into a practical evasion-oriented technique.

The attacker does not exploit Windows directly. Instead, they abuse **legitimate NTFS functionality** intentionally designed by Microsoft for compatibility, storage management, and path redirection purposes.

The core idea is to create **recursive directory structures** capable of confusing or heavily stressing recursive scanning engines. In poorly handled scenarios, the scanner may spend excessive resources traversing recursive paths before properly reaching or analyzing the actual payload.

Importantly:

- there is **no encryption** involved
- **no hidden shellcode** involved
- **no exploit primitive** required for the traversal logic itself

The payload may exist normally on disk — **the issue is the traversal behavior surrounding it**.

### Background

Researchers from **Varonis** demonstrated how specially crafted NTFS junction structures could trigger recursive traversal conditions capable of:

- degrading scanning performance
- creating excessive CPU, memory, and I/O consumption

The behavior was also demonstrated against **Microsoft Defender**. Although Microsoft reportedly did not initially classify the issue as crossing a formal security boundary, patches and traversal handling improvements were later introduced to mitigate recursive scanning instability.

> **Note:** The underlying concepts are not entirely new. Techniques involving NTFS Junctions, Reparse Points, Recursive Traversal Loops, and Path Explosion Conditions have existed conceptually for years. What made GhostTree notable was the **operational framing** and **practical demonstration** of these concepts against defensive tooling.

---

## 2. NTFS Reparse Points Fundamentals

To properly understand GhostTree, it is first necessary to understand the Windows filesystem feature that makes the technique possible.

### What Is a Reparse Point?

In NTFS, a **reparse point** allows Windows to transparently redirect filesystem access from one path to another. Applications often do not realize this redirection is happening internally.

Windows introduced this mechanism primarily for three purposes:

<p align="center">
<table>
  <thead>
    <tr>
      <th>Purpose</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Compatibility</strong></td>
      <td>Legacy apps with hardcoded paths can be redirected transparently without code changes</td>
    </tr>
    <tr>
      <td><strong>Storage Optimization</strong></td>
      <td>Large datasets can be relocated across volumes without breaking path expectations</td>
    </tr>
    <tr>
      <td><strong>Path Redirection</strong></td>
      <td>Applications continue functioning normally even if the real storage location changes</td>
    </tr>
  </tbody>
</table>
</p>

### Compatibility — Example

An application may expect its files inside:

```
C:\ProgramData\App
```

But administrators may later decide to move the actual data elsewhere. Instead of rewriting the application, Windows can transparently redirect the original path to another location.

### Storage Optimization — Example

```
C:\Logs  →  D:\ArchivedLogs
```

Applications continue using `C:\Logs` while storage is managed on `D:\`.

### What Is an NTFS Junction?

A **Junction** is a specific type of NTFS reparse point used for redirecting one **directory** to another **local directory**.

```
C:\Logs  →  D:\ArchivedLogs
```

If an application accesses `C:\Logs`, Windows transparently redirects the request without the application necessarily realizing it.

### Junction Limitations

<p align="center">
<table>
  <thead>
    <tr>
      <th>Constraint</th>
      <th>Details</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Local-only</strong></td>
      <td>A junction can only point to another local path on the same system — no UNC/network paths</td>
    </tr>
    <tr>
      <td><strong>Directories only</strong></td>
      <td>Junctions cannot point directly to files</td>
    </tr>
    <tr>
      <td><strong>Absolute paths required</strong></td>
      <td>NTFS stores a canonical destination path — relative paths are not valid</td>
    </tr>
  </tbody>
</table>
</p>

```cmd
# Valid
mklink /J Child C:\Parent

# Invalid
mklink /J Child ..\Parent
```

---

## 3. Junctions vs Symbolic Links

Here is a visual comparison of the key differences between Junctions and Symbolic Links : 

<p align="center">
  <img src="https://github.com/Nullon1/Threat-Intel-Hub/blob/main/Defense-Evasion/GhostTree-Evasion/images/Junctions%20VS%20Symbolics.png">
</p>

The most important difference from an attacker's perspective is **privilege requirements**.

- Creating **symbolic links** traditionally required elevated privileges or developer mode configuration.
- **Junctions**, however, are significantly less restricted.

An attacker with **normal write permissions** can often create recursive junction structures **without administrator access**. This is one of the primary reasons junctions became attractive for filesystem-based evasion scenarios.

---

## 4. Why Junctions Became Attractive for Evasion

From an attacker's perspective, NTFS Junctions have several characteristics that make them useful:

- Native Windows functionality — no external tools required
- Usually **no administrator privileges** required
- Low-friction creation via `mklink /J`
- Compatibility with existing software
- Ability to introduce **recursive traversal loops**

### The Core Problem

Historically, many defensive products treated filesystem traversal as a **simple recursive tree**. That assumption becomes dangerous once recursive redirection loops are introduced.

**Normal structure — a tree:**

```
Parent
├── Child
└── File.exe
```

**Junction-abused structure — a graph with cycles:**

```
Parent
├── Child1  →  Parent
├── Child2  →  Parent
└── Payload.exe
```

The filesystem no longer behaves like a tree — **it behaves like a graph containing cycles**.

> Humans immediately recognize the loop. Defensive software must **explicitly detect** it.

---

## 5. Understanding the Windows MAX_PATH Limitation

### The Historical Limit

Historically, many Win32 APIs used a path length limit of **260 characters**.

This value originated from older Windows and MS-DOS design assumptions and includes:

- Drive letter
- Directory names
- File name
- Null terminator

### The Real Story

> **NTFS itself was never the real limitation.**

The limitation came primarily from **legacy Win32 APIs**. Modern Windows supports extended-length paths using the `\\?\` prefix, which can approach **32,767 characters**.

However, backward compatibility keeps many legacy assumptions alive across tooling and scanners.

### What MAX_PATH Actually Affects

<p align="center">
<table>
  <thead>
    <tr>
      <th>Factor</th>
      <th>Impact</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Traversal depth</strong></td>
      <td>Limits how deep legacy tools can recurse</td>
    </tr>
    <tr>
      <td><strong>Recursive expansion</strong></td>
      <td>Influences how path strings grow per level</td>
    </tr>
    <tr>
      <td><strong>Legacy scanner assumptions</strong></td>
      <td>Older implementations may truncate or fail silently</td>
    </tr>
    <tr>
      <td><strong>Path-handling bugs</strong></td>
      <td>Unexpected behavior at boundary conditions</td>
    </tr>
  </tbody>
</table>
</p>

> **Important:** Recursive filesystem loops are not fundamentally caused by MAX_PATH. The real issue is **graph recursion and cycle handling**.

---

## 6. From GhostBranch to GhostTree

A simple recursive loop might look like:

```
C:\Parent\Child  →  C:\Parent
```

GhostTree becomes more interesting once **multiple recursive child nodes** are introduced:

```
C:\Parent
├── Child1  →  C:\Parent
├── Child2  →  C:\Parent
├── Child3  →  C:\Parent
└── Payload.exe
```

Instead of a single recursive path, the scanner now faces **multiple recursive branches** — a condition researchers commonly describe as:

> **Path Explosion**

The filesystem effectively behaves like a **recursively expanding graph** rather than a traditional directory hierarchy.

---

## 7. Understanding the "126 Levels" Limitation

> **Note:** The values discussed below are theoretical approximations intended to illustrate recursive growth behavior. They should not be interpreted as official NTFS limits or guarantees of real-world scanner behavior.

### Where Does "126" Come From?

The commonly cited **"126 levels"** figure is not an official NTFS limitation. It is a simplified approximation derived from historical MAX_PATH constraints under optimized directory naming assumptions.

If directory names are reduced to a single character:

```
\A\A\A\A\...
```

Each level consumes approximately **2 characters** (name + separator).

```
260 / 2 ≈ 130
```

After accounting for drive letter, formatting, and terminators, the practical figure commonly discussed becomes approximately **126 levels**.

### Why Depth Alone Is Not the Problem

<p align="center">
<table>
  <thead>
    <tr>
      <th>Factor</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Depth</strong></td>
      <td>Limits how far down any single path can go</td>
    </tr>
    <tr>
      <td><strong>Branching</strong></td>
      <td>Multiplies the number of traversal paths exponentially</td>
    </tr>
  </tbody>
</table>
</p>

Under a simplified theoretical model, a **dual-node recursive structure** can produce approximately:

```
2^126  logically distinct traversal paths
```

These paths **do not physically exist on disk**. The figure represents the number of possible traversal combinations within the recursive graph and is primarily useful for illustrating **exponential growth behavior**.

### Real-World Safeguards

In practice, modern scanners commonly implement:

- Recursion watchdogs
- Cycle detection
- Traversal limits
- Visited-node tracking (via File IDs / MFT Record Numbers)

These mechanisms often prevent scanners from exploring every theoretical traversal branch — but their quality **varies significantly across implementations**.

---

## 8. Practical Lab Scenario

The following example demonstrates the logical structure behind GhostTree in a controlled lab-style scenario for **defensive understanding only**.

<p align="center">
  <img src="https://github.com/Nullon1/Threat-Intel-Hub/blob/main/Defense-Evasion/GhostTree-Evasion/images/Flow.png">
</p>

### Structure

```
C:\Parent
├── Child1  →  Junction to C:\Parent
├── Child2  →  Junction to C:\Parent
└── Payload.exe
```

### Setup Commands

```cmd
mklink /J Child1 C:\Parent
mklink /J Child2 C:\Parent
```

### What Happens During a Scan

1. Scanner begins at `C:\Parent`
2. It encounters `Child1`, `Child2`, and `Payload.exe`
3. It enters `C:\Parent\Child1` — Windows redirects back into `C:\Parent`
4. The scanner may now re-enumerate `Child1`, `Child2`, and `Payload.exe` again
5. This cycle repeats

Each traversal iteration may trigger:

<p align="center">
<table>
  <thead>
    <tr>
      <th>Operation</th>
      <th>Resource Cost</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Directory enumeration</td>
      <td>I/O</td>
    </tr>
    <tr>
      <td>Hash calculation</td>
      <td>CPU</td>
    </tr>
    <tr>
      <td>Metadata inspection</td>
      <td>Memory</td>
    </tr>
    <tr>
      <td>Permission validation</td>
      <td>CPU + I/O</td>
    </tr>
    <tr>
      <td>Memory allocation</td>
      <td>Memory</td>
    </tr>
    <tr>
      <td>Scan job scheduling</td>
      <td>CPU</td>
    </tr>
  </tbody>
</table>
</p>

Depending on traversal protections and cycle-awareness, recursive structures may significantly **degrade scanning performance** or create **traversal instability**.

---

## 9. Detection & Defensive Considerations

GhostTree demonstrates why defensive products should treat filesystem traversal as **graph analysis** rather than assuming every directory structure is a safe recursive tree.

### Modern Detection Approaches

Modern cycle-aware scanners track underlying filesystem objects using:

- **File IDs**
- **MFT Record Numbers**
- **Object IDs**
- Other filesystem-specific identifiers

This allows a scanner to recognize when the **same filesystem object** has already been visited, even if it is reached through a different logical path.

### Important Caveats

> The existence of modern cycle-detection mechanisms should not be interpreted as universal immunity.

- Traversal protection quality **varies significantly** across implementations
- Recursive traversal issues were demonstrated against **Microsoft Defender** before traversal-handling improvements were introduced
- Traversal logic remains **implementation-dependent** and has historically been a source of reliability issues across multiple software categories

---

## 10. Final Thoughts

GhostTree is interesting not because it introduces a new vulnerability class, but because it highlights a recurring problem in defensive engineering:

> **Filesystems are not always simple trees.**

Once reparse points, junctions, and recursive references enter the picture, the filesystem begins behaving more like a **graph**. Any security product that assumes every path expansion leads to a completely new location risks spending resources analyzing the same underlying objects repeatedly.

### Key Takeaways

<p align="center">
<table>
  <thead>
    <tr>
      <th>Perspective</th>
      <th>Takeaway</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Attackers</strong></td>
      <td>Junctions are low-privilege, native, and capable of creating traversal instability without any exploit</td>
    </tr>
    <tr>
      <td><strong>Defenders</strong></td>
      <td>Cycle detection, object-level tracking, and traversal limits are essential — not optional</td>
    </tr>
    <tr>
      <td><strong>Engineers</strong></td>
      <td>Test scanning engines against graph-like structures, not just linear directory trees</td>
    </tr>
  </tbody>
</table>
</p>

The practical impact of GhostTree should not be overstated — modern security products commonly implement recursion limits, cycle detection, and other safeguards. But the Microsoft Defender case demonstrated that traversal logic can still become a source of operational issues when recursive filesystem structures are not handled correctly.

For defenders, GhostTree serves as a useful reminder:

> Security products must understand the **structure** they are analyzing — not just the **paths** they are presented with.

In many cases, the challenge is not detecting a malicious file. **The challenge is reaching it efficiently without getting lost in the filesystem graph along the way.**

---

*This document is intended for defensive research and educational purposes only.*
