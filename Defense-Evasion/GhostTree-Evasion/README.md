# GhostTree — NTFS Junction-Based Defense Evasion

> **Category:** Defense Evasion / Filesystem Abuse  
> **Platform:** Windows (NTFS)  
> **Privileges Required:** Standard user (no admin needed)  
> **Related Concepts:** Reparse Points, NTFS Junctions, Recursive Traversal, Path Explosion

---

## Overview

**GhostTree** is a filesystem-based defense evasion methodology that abuses native NTFS Junction (Reparse Point) functionality to create recursive directory loops. These loops are capable of confusing, stalling, or exhausting recursive scanning engines — including EDR and AV products — before they ever reach the actual payload on disk.

This is **not** a malware family, kernel exploit, or privilege escalation technique. It requires no shellcode, no encryption, and no administrator rights. It abuses legitimate Windows filesystem behavior by design.

> Originally demonstrated by **Varonis researchers**, who showed the technique could degrade scanning performance and create traversal instability — including against Microsoft Defender.

---

## How It Works

NTFS Junctions allow a directory to transparently redirect to another location. When two or more junctions inside a parent directory point **back to that same parent**, a recursive loop is formed. A scanner entering the directory will keep re-enumerating the same structure indefinitely.

```
C:\Parent
├── Child1  →  Junction → C:\Parent
├── Child2  →  Junction → C:\Parent
└── Payload.exe
```

Each traversal level re-expands into the full structure, producing exponential path combinations — a condition known as **path explosion**.

### NTFS Junction Loop — Visual

![GhostTree NTFS Junction Loop Structure](https://github.com/Nullon1/Threat-Intel-Hub/blob/main/Defense-Evasion/GhostTree-Evasion/images/Flow.png)

The diagram above illustrates how `Child1` and `Child2` junction back to `C:\Parent`, causing scanners to recurse infinitely until the path length limit (~260 chars) is hit — never reaching `Program.exe`.

---

## Why It's Effective

| Factor | Detail |
|---|---|
| No elevated privileges | Junctions can be created by standard users via `mklink /J` |
| No exotic tooling | Built into Windows — no third-party dependencies |
| No payload obfuscation | The file exists normally on disk; the issue is reaching it |
| Exponential traversal cost | Dual-node loop produces ~2¹²⁶ logical traversal paths |
| Resource exhaustion | Each traversal cycle burns CPU, memory, and I/O |

---

## Disclaimer

This research is intended **for defensive and educational purposes only**. The concepts documented here are derived from publicly available security research. No exploit code or weaponized tooling is provided. Use this knowledge to improve detection engineering and scanner resilience.
