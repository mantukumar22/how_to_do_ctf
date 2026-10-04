# CTF (Capture The Flag) — Overview

## What is a CTF?
A CTF is a cybersecurity competition where participants solve security challenges (exploiting bugs, cracking crypto, reversing binaries, analyzing traffic, etc.) to find hidden strings called "flags" (e.g. `flag{...}`). CTFs teach real offensive/defensive skills in a legal, controlled sandbox — used for learning, recruiting, and benchmarking skill.

## Types of CTFs

| Type | Description |
|---|---|
| **Jeopardy** | Independent challenges grouped by category (Web, Pwn, Crypto, Forensics, Reverse, Misc). Solve any in any order for points. Most common for beginners. |
| **Attack-Defense** | Each team runs a vulnerable service; you patch your own while exploiting others' live. Fast-paced, team-heavy. |
| **Mixed/King of the Hill** | Hybrid — maintain control of a shared vulnerable box/service longer than rivals. |
| **Boot2Root** | Single VM/host, find a path from initial access to root (e.g. HackTheBox, VulnHub, TryHackMe). |

## Common Categories (Jeopardy-style)
- **Web** – SQLi, XSS, SSRF, auth bypass, deserialization
- **Pwn / Binary Exploitation** – buffer overflows, ROP, heap exploitation
- **Reverse Engineering** – disassembling/decompiling binaries to find logic/flags
- **Crypto** – breaking weak ciphers, encoding, key recovery
- **Forensics** – memory dumps, disk images, file carving, steganography
- **Network/Pcap** – analyzing packet captures for leaked data
- **OSINT** – finding info from public sources
- **Misc** – scripting, puzzles, odd one-offs

## How the Process Generally Works
1. **Recon** – identify what you're given (IP, binary, pcap, source code, URL) and gather info passively/actively.
2. **Enumeration** – find attack surface (open ports, endpoints, functions, file structure).
3. **Vulnerability Identification** – map enumerated surface to known weaknesses/bug classes.
4. **Exploitation** – craft and deliver a payload/technique to leverage the vulnerability.
5. **Post-Exploitation / Extraction** – read files, escalate privileges, or decode output to reveal the flag.
6. **Documentation** – write up what you did (helps learning and often required for scoring on some platforms).

## Ground Rules
- Only attack systems you're authorized to (official CTF infra, your own labs, or platforms like HTB/TryHackMe/PicoCTF).
- Never point these techniques at systems you don't own or lack explicit permission to test — doing so is illegal.
- Keep notes as you go; CTFs reward methodical documentation over random guessing.

See `ctf_walkthrough.md` for a step-by-step recon-to-flag command reference.
