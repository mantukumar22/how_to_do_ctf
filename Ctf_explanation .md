# CTF Explained — An Experienced Player's Mental Model

> Written like a 5-year CTF player explaining how they actually think, not just what commands to run.
> Pairs with `README.md` (overview) and `ctf_walkthrough.md` (step-by-step commands).

---

## 1. The Mindset First (this matters more than tools)

**What a 5-year player keeps in their head on every single challenge:**

- **"What category is this, really?"** — The stated category (Web/Pwn/Crypto) can be a decoy. A "Web" challenge might really be a crypto puzzle hidden in a cookie. Read the whole challenge before running anything.
- **Trust the filename and file size.** A 2KB "image.png" that's listed as 2MB is hiding something (steganography/binwalk territory). A `.pcap` with one giant stream is worth `follow TCP stream` before anything fancy.
- **Flags have a format — use it.** Most platforms tell you the flag format upfront (`flag{...}`, `CTF{...}`, `picoCTF{...}`). Grep for that pattern everywhere, always. It saves hours.
- **Don't rabbit-hole.** If you're 30–45 min into one approach with zero progress, step back, re-read the challenge description, and try a completely different angle. The description almost always has a clue you missed.
- **Points = difficulty, not time.** A 500-point challenge isn't "500 minutes of work" — sometimes it's one clever insight. Don't assume effort correlates with points.
- **Check challenge comments/solves count** (if platform shows it) — if 200 teams solved something you think is "unsolvable," you're overcomplicating it.
- **Save everything.** Every command, every output, every half-working idea — in a separate file per challenge. You WILL need to backtrack.
- **Diff is your best friend.** Comparing a modified file against the "normal" version of that file type (a patched binary vs stock libc, a tampered image vs a clean one) finds the needle fast.

---

## 2. Environment Setup — What Must Exist Before You Start

| Item | Why it matters |
|---|---|
| **Kali Linux / ParrotOS VM** (or Docker container) | Comes with 90% of tools pre-installed. Don't CTF from a bare OS. |
| **A dedicated CTF working folder structure** | Keeps challenges from bleeding into each other (see structure below). |
| **VPN client** (OpenVPN/WireGuard) | Most CTFs (HTB, THM, official competitions) require VPN access to the challenge network. |
| **Burp Suite (Community is fine)** | Non-negotiable for any Web category. |
| **A note-taking tool** (Obsidian, CherryTree, or just markdown files) | For writeups and tracking partial progress. |
| **Python 3 + pip + pwntools + requests** | 80% of exploit scripting uses these. |
| **A wordlists folder** (`/usr/share/wordlists`, SecLists cloned) | Directory brute-forcing and password cracking need these. |

### Recommended folder structure per CTF
```
ctf_event_name/
├── web/
│   └── chall1/
│       ├── files/        <- downloaded/given files, never modified
│       ├── notes.md       <- your working notes
│       └── solve.py       <- final/working exploit script
├── pwn/
├── crypto/
├── forensics/
├── rev/
└── misc/
```
**Why this matters:** keep the original given files untouched in `files/` — you will mess up a binary or image while testing, and you need a clean copy to fall back to.

---

## 3. The Tool Belt — What Experienced Players Actually Install

### Recon / Network
- `nmap` — port/service scanning
- `rustscan` — fast port discovery (feeds into nmap)
- `netcat (nc)` / `socat` — raw connections, shell catching
- `Wireshark` / `tshark` — packet analysis

### Web
- `Burp Suite` — intercept/modify/replay requests
- `gobuster` / `ffuf` / `dirsearch` — content discovery
- `sqlmap` — automated SQL injection
- `whatweb` / `wappalyzer` — fingerprinting
- Browser DevTools (Network + Console tabs) — always open

### Binary / Reverse Engineering
- `Ghidra` — free, full decompiler (industry standard)
- `radare2` / `rizin` + `cutter` (GUI) — alternative/lighter RE suite
- `gdb` + `pwndbg` or `GEF` plugin — debugging with exploit-dev helpers
- `checksec` — binary protection check
- `strings`, `file`, `objdump`, `ltrace`, `strace` — quick static/dynamic inspection

### Pwn / Exploitation
- `pwntools` (Python library) — exploit scripting framework
- `one_gadget`, `ROPgadget` — finding gadgets for ROP chains
- `patchelf` — change a binary's linked libc for local testing

### Crypto
- `CyberChef` (web tool) — "the Swiss army knife," try this before writing custom code
- `openssl` — quick cipher/cert operations
- `RsaCtfTool` — automated RSA attacks
- Python `pycryptodome` — custom crypto scripting

### Forensics
- `exiftool` — metadata extraction
- `binwalk` — embedded file/firmware extraction
- `steghide`, `zsteg`, `stegsolve` — steganography
- `volatility3` — memory dump (RAM) analysis
- `foremost` / `photorec` — file carving from raw disk images
- `Autopsy` — full disk forensics GUI

### OSINT
- `theHarvester` — email/subdomain gathering
- Google dorking, Shodan, Censys — public exposure searches
- `exiftool` again — photo metadata often reveals location/device

### Password/Hash Cracking
- `hashcat` (GPU-accelerated, preferred) 
- `john` (John the Ripper, CPU-based, good for format auto-detection)
- `hash-identifier` / `hashid` — figure out what hash type you even have

### Privilege Escalation (boot2root style)
- `linpeas.sh` / `winpeas.exe` — automated enumeration
- `pspy` — process monitoring without root (catches cron jobs)
- GTFOBins (website, not a tool) — look up privesc via allowed sudo binaries

---

## 4. What Files Typically "Hide" the Challenge — Know Where to Look

| File type given | First things an experienced player checks |
|---|---|
| **Image (.png/.jpg/.bmp)** | `exiftool`, `binwalk -e`, `strings`, then `zsteg`/`steghide` (LSB steganography), check image dimensions for appended data |
| **PDF** | `exiftool`, `pdf-parser`, check for embedded JS or attachments (`pdfdetach`) |
| **ZIP/Archive** | Check for password (`zip2john` + `john`), nested archives (archive-in-archive is common) |
| **Binary (ELF/PE)** | `file`, `checksec`, `strings`, then Ghidra/r2 for logic, `ltrace`/`strace` for runtime behavior |
| **PCAP** | Follow TCP/HTTP streams in Wireshark, export objects (File → Export Objects), check for FTP/Telnet creds in cleartext |
| **Memory dump (.raw/.mem/.vmem)** | `volatility3 -f FILE windows.info` (or linux.info) to ID the profile, then `pslist`, `filescan`, `dumpfiles` |
| **Disk image (.dd/.img)** | Mount read-only or use `foremost`/`Autopsy`, check deleted files and slack space |
| **Source code dump** | `grep -rn "flag\|password\|secret\|key"` across the whole repo first, check `.git/` history if present (`git log -p`) |
| **Encrypted/encoded text blob** | Try CyberChef "Magic" wand first — auto-detects base64/hex/rot13/gzip chains instantly |

---

## 5. Step-by-Step: How an Experienced Player Actually Approaches a Fresh Challenge

1. **Read the full challenge text twice.** Note any hints, flag format, and attached files. (90 seconds, saves hours.)
2. **Identify file type(s) given.** `file *` on everything in the challenge folder immediately.
3. **Run baseline recon tools** matching the category (nmap for web/pwn with a host; exiftool+binwalk for forensics files; checksec+file for binaries).
4. **Google the exact challenge title/wording** (sometimes challenges reuse known CVEs or public writeup patterns — not cheating, just efficient recon) *within allowed platform rules*.
5. **Form a hypothesis** — "I think this is a buffer overflow because checksec shows no canary and the binary reads unbounded input."
6. **Test the hypothesis small.** Don't write the full exploit yet — prove the crash/bug exists first (e.g., send a long string and confirm segfault).
7. **Build the exploit/solution incrementally**, testing at each stage (e.g., confirm offset before building the full ROP chain).
8. **Validate locally before going remote** (for pwn — always test against your local copy of the binary before firing at the remote server; remote attempts are often rate-limited or logged).
9. **Extract/format the flag exactly as required** (case-sensitive, exact brackets — copy-paste, don't retype).
10. **Submit, then immediately write 3 lines of notes** on how you solved it — your future self in a later competition will thank you.

---

## 6. Common Mistakes Even Experienced Players Watch For

- **Overwriting the original challenge file** while testing — always work on a copy.
- **Forgetting `-oN`/output flags on nmap** — don't lose scan results.
- **Ignoring HTTP response headers** — flags and hints hide in headers (`Set-Cookie`, custom `X-` headers) more often than people expect.
- **Not checking source/`view-source`** on web challenges before brute-forcing.
- **Assuming local exploit offsets match remote** — different libc version on remote server breaks pwn exploits; always check with `file`/`ldd` or provided Docker/libc files.
- **Submitting the flag with wrapping text** (e.g. submitting `Found it: flag{...}` instead of just `flag{...}`).
- **Not reading challenge point value/category tags** — "easy" tagged as "hard" due to a trick, or vice versa — re-calibrate effort accordingly.
- **Skipping `strings` on anything** — it's cheap, instant, and finds flags left in plaintext embarrassingly often.

---

## 7. Quick Mental Checklist (keep this pinned while solving)

- [ ] Did I read the full description + title for hints?
- [ ] Do I know the exact flag format?
- [ ] Have I run `file` on every given file?
- [ ] Am I working on a COPY of the original file?
- [ ] Have I checked metadata (`exiftool`) before deep analysis?
- [ ] Have I tried the "dumb" check (strings/grep for flag pattern) before the complex one?
- [ ] Is there a public writeup/CVE for this exact service/version?
- [ ] Have I saved my commands/notes as I go?
- [ ] Am I stuck >40 min? → Step back, re-read, try different category angle.
- [ ] Did I copy the flag exactly, no extra text, before submitting?

---

## 8. Ethics & Ground Rules (always applies)

- Only test targets explicitly provided by the CTF platform/organizers, or systems you own.
- Never reuse CTF exploitation techniques against real-world/production systems without authorization — that crosses from "practice" into illegal activity.
- If a CTF rule says "no attacking the scoreboard/infra directly," respect it — scoreboard attacks often get you disqualified or banned.
- Respect "no DoS" rules — don't run aggressive brute-force/flood scripts against shared competition infrastructure.
