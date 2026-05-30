# 1. Introduction

Recently, a filesystem-based defense evasion concept referred to as GhostTree started gaining attention across the cybersecurity community. Unlike traditional evasion methods that rely on encryption, shellcode obfuscation, or memory corruption, GhostTree abuses the logical behavior of filesystem traversal itself.

One important distinction should be made early:

GhostTree is not:

- a malware family
- a privilege escalation technique
- a kernel exploit
- a memory corruption vulnerability
- a classic signature bypass

GhostTree also does not introduce a new NTFS primitive or filesystem capability.

Instead, it is more accurately described as a methodology that combines existing NTFS features, recursive traversal behavior, and filesystem graph abuse into a practical evasion-oriented technique.

The attacker does not exploit Windows directly. Instead, they abuse legitimate NTFS functionality intentionally designed by Microsoft for compatibility, storage management, and path redirection purposes.

The core idea is to create recursive directory structures capable of confusing or heavily stressing recursive scanning engines. In poorly handled scenarios, the scanner may spend excessive resources traversing recursive paths before properly reaching or analyzing the actual payload.

Importantly:

- there is no encryption involved
- no hidden shellcode involved
- and no exploit primitive required for the traversal logic itself

The payload may exist normally on disk.

The issue is the traversal behavior surrounding it.

Researchers from Varonis demonstrated how specially crafted NTFS junction structures could trigger recursive traversal conditions capable of degrading scanning performance and creating excessive CPU, memory, and I/O consumption.

The behavior was also demonstrated against Microsoft Defender. Although Microsoft reportedly did not initially classify the issue as crossing a formal security boundary, patches and traversal handling improvements were later introduced to mitigate recursive scanning instability associated with these filesystem loop conditions.

It is also important to note that the underlying concepts are not entirely new. Techniques involving:

- NTFS Junctions
- Reparse Points
- Recursive Traversal Loops
- Path Explosion Conditions

have existed conceptually for years.

What made GhostTree notable was the operational framing and practical demonstration of these concepts against defensive tooling.

# 2. NTFS Reparse Points Fundamentals

To properly understand GhostTree, it is first necessary to understand the Windows filesystem feature that makes the technique possible.

The core concept revolves around something called a Reparse Point.

In NTFS, a reparse point allows Windows to transparently redirect filesystem access from one path to another.

Applications often do not realize this redirection is happening internally.

Windows introduced this mechanism primarily for:

- Compatibility
- Storage Optimization
- Seamless Path Redirection

## Compatibility

Many legacy applications were written with hardcoded assumptions about where data should exist.

For example, an application may expect its files inside:

C:\ProgramData\App

But administrators may later decide to move the actual data elsewhere.

Instead of rewriting the application, Windows can transparently redirect the original path to another location.

## Storage Optimization

Reparse points also help relocate large datasets between storage volumes without breaking application path expectations.

Example:

C:\Logs

may internally redirect to:

D:\ArchivedLogs

Applications continue using the original path while storage is managed elsewhere.

## Seamless Path Redirection

Applications can continue functioning normally even if the real storage location changes underneath them.

The application still believes it is interacting with:

C:\Application\Data

while NTFS silently redirects requests internally.

This transparent redirection behavior is exactly what GhostTree abuses against recursive scanning engines.

## What Is an NTFS Junction?

A Junction is a specific type of NTFS reparse point used for redirecting one directory to another local directory.

Example:

C:\Logs -> D:\ArchivedLogs

If an application accesses:

C:\Logs

Windows transparently redirects the request without the application necessarily realizing it.

## Important Junction Limitations

### 1. Junctions Are Local-Only

A junction can only point to another local path on the same system.

Valid:

C:\Data -> D:\Backup

Invalid:

C:\Data -> \\RemoteServer\Share

### 2. Junctions Work Only With Directories

Junctions cannot point directly to files.

Valid:

C:\Folder -> C:\AnotherFolder

Invalid:

report.docx -> backup.docx

### 3. Junctions Require Absolute Paths

Example:

mklink /J Child C:\Parent

instead of:

mklink /J Child ..\Parent

NTFS stores a canonical destination path for the reparse object, which is why Windows expects an exact target location.

# 3. Junctions vs Symbolic Links

| Feature | Junction | Symbolic Link |
|----------|----------|----------|
| Requires Administrator Privileges | Usually No | Often Yes |
| Works With Files | No | Yes |
| Works With Directories | Yes | Yes |
| Supports Remote Paths | No | Yes |
| Primary Purpose | Local Directory Redirection | Flexible Filesystem Linking |

The most important difference from an attacker's perspective is privilege requirements.

Creating symbolic links traditionally required elevated privileges or developer mode configuration.

Junctions, however, are significantly less restricted.

An attacker with normal write permissions can often create recursive junction structures without administrator access.

This is one of the primary reasons junctions became attractive for filesystem-based evasion scenarios.

# 4. Why Junctions Became Attractive for Evasion

From an attacker’s perspective, NTFS Junctions have several characteristics that make them useful for filesystem-based evasion.

- Native Windows functionality
- Usually no administrator privileges required
- Low-friction creation
- Compatibility with existing software
- Ability to introduce recursive traversal loops

Historically, many defensive products treated filesystem traversal as a simple recursive tree.

That assumption becomes dangerous once recursive redirection loops are introduced.

Instead of:

Parent
├── Child
└── File.exe

the scanner encounters:

Parent
├── Child1 -> Parent
├── Child2 -> Parent
└── Payload.exe

The filesystem no longer behaves like a tree.

It behaves like a graph containing cycles.

Humans immediately recognize the loop.

Defensive software must explicitly detect it.

# 5. Understanding the Windows MAX_PATH Limitation

At this point, an important question appears:

If Windows limits path length, how can attackers still create extremely deep recursive traversal structures?

The answer starts with MAX_PATH.

Historically, many Win32 APIs used a path length limit of:

260 characters

This value originated from older Windows and MS-DOS design assumptions.

The 260-character limit includes:

- Drive letter
- Directory names
- File name
- Null terminator

Importantly:

NTFS itself was never the real limitation.

The limitation came primarily from legacy Win32 APIs.

Modern Windows supports extended-length paths such as:

\\?\

which can approach:

32,767 characters

However, backward compatibility keeps many legacy assumptions alive.

It is also important to understand that recursive filesystem loops are not fundamentally caused by MAX_PATH.

The real issue is graph recursion and cycle handling.

MAX_PATH mainly influences:

- Traversal depth
- Recursive expansion behavior
- Legacy scanner assumptions
- Older path-handling implementations

# 6. From GhostBranch to GhostTree

A simple recursive loop might look like:

C:\Parent\Child -> C:\Parent

GhostTree becomes more interesting once multiple recursive child nodes are introduced.

Example:

C:\Parent
├── Child1 -> C:\Parent
├── Child2 -> C:\Parent
├── Child3 -> C:\Parent
└── Payload.exe

Instead of a single recursive path, the scanner faces multiple recursive branches.

Researchers commonly describe this as:

Path Explosion

The filesystem effectively behaves like a recursively expanding graph rather than a traditional directory hierarchy.

# 7. Understanding the "126 Levels" Limitation

> **Note**
>
> The values discussed below are theoretical approximations intended to illustrate recursive growth behavior.
>
> They should not be interpreted as official NTFS limits or guarantees of real-world scanner behavior.

Many discussions reference the commonly cited "126 levels" figure.

This value is not an official NTFS limitation.

Rather, it is a simplified approximation derived from historical MAX_PATH constraints under highly optimized directory naming assumptions.

If directory names are reduced to:

A

each level consumes approximately:

- 1 character for the directory name
- 1 character for the backslash separator

Result:

\A

consumes roughly 2 characters.

A simplified calculation becomes:

260 / 2 ≈ 130

After accounting for:

- C:\
- Internal formatting
- Terminators

the practical value often discussed becomes approximately:

126 levels

However, depth alone is not the most important problem.

Branching is.

Under a simplified theoretical model, a dual-node recursive structure can produce approximately:

2^126

logically distinct traversal paths.

These paths do not physically exist on disk.

The figure represents the number of possible traversal combinations within the recursive graph structure and is primarily useful for illustrating exponential growth behavior rather than predicting real-world scanner behavior.

In practice:

- Recursion watchdogs
- Cycle detection
- Traversal limits
- Caching
- Visited-node tracking

often prevent scanners from exploring every theoretical traversal branch.

# 8. Practical Lab Scenario

The following example demonstrates the logical structure behind GhostTree in a controlled lab-style scenario for defensive understanding.

Structure:

C:\Parent
├── Child1 -> Junction to C:\Parent
├── Child2 -> Junction to C:\Parent
└── Payload.exe

Example:

mklink /J Child1 C:\Parent
mklink /J Child2 C:\Parent

The scanner begins with:

C:\Parent

and encounters:

- Child1
- Child2
- Payload.exe

When it enters:

C:\Parent\Child1

Windows redirects traversal back into:

C:\Parent

again.

The scanner may repeatedly:

- Enumerate directories
- Calculate hashes
- Inspect metadata
- Validate permissions
- Allocate memory
- Schedule scanning operations

All of these operations consume resources.

Depending on traversal protections and cycle-awareness implementation, recursive structures may significantly degrade scanning performance or create traversal instability.

# 9. Detection & Defensive Considerations

GhostTree demonstrates why defensive products should treat filesystem traversal as graph analysis rather than assuming every directory structure is a safe recursive tree.

Modern scanners typically do not rely solely on path strings.

Many implementations track underlying filesystem objects using:

- File IDs
- MFT Record Numbers
- Object IDs
- Other filesystem-specific identifiers

This allows cycle-aware scanners to recognize when the same filesystem object has already been visited, even if it is reached through a different logical path.

However, traversal protection quality varies significantly across implementations.

The existence of modern cycle-detection mechanisms should not be interpreted as universal immunity.

Traversal logic remains implementation-dependent and has historically been a source of reliability issues across multiple software categories.

This point is particularly relevant because recursive traversal issues were demonstrated against Microsoft Defender before traversal-handling improvements and mitigations were introduced.

# 10. Final Thoughts

GhostTree is interesting not because it introduces a new vulnerability class, but because it highlights a recurring problem in defensive engineering:

filesystems are not always simple trees.

Once reparse points, junctions, and recursive references enter the picture, the filesystem begins behaving more like a graph. Any security product that assumes every path expansion leads to a completely new location risks spending resources analyzing the same underlying objects repeatedly.

The practical impact of GhostTree should not be overstated. Modern security products commonly implement recursion limits, cycle detection, object tracking, and other safeguards designed to prevent uncontrolled traversal behavior.

At the same time, the Microsoft Defender case demonstrated that traversal logic can still become a source of operational issues when recursive filesystem structures are not handled correctly.

For defenders, GhostTree serves as a useful reminder that security products must understand the structure they are analyzing, not just the paths they are presented with.

In many cases, the challenge is not detecting a malicious file.

The challenge is reaching it efficiently without getting lost in the filesystem graph along the way.
