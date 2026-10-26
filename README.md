<div align="center">

  <strong>MISL RAT</strong>

  <pre>
  

         __             _,-"~^"-.
       _// )      _,-"~`         `.
     ." ( /`"-,-"`                 ;
    / 6                             ;
   /           ,             ,-"     ;
  (,__.--.      \           /        ;
   //'   /`-.\   |          |        `._________
     _.-'_/`  )  )--...,,,___\     \-----------,)
   ((("~` _.-'.-'           __`-.   )         //
         ((("`             (((---~"`         //
                                            ((________________
                                            `----""""~~~~^^^```

  </pre>

  <strong>Modular remote administration toolkit — red-team &amp; lab use</strong>

</div>

<p align="center">
  <img src="https://img.shields.io/badge/HACKTOOL-ff2d55?style=for-the-badge&logo=target&logoColor=white" alt="HACKTOOL">
  <img src="https://img.shields.io/badge/C++-00599C?style=for-the-badge&logo=cplusplus&logoColor=white" alt="C++">
  <img src="https://img.shields.io/badge/OPEN--SOURCE-22c55e?style=for-the-badge&logo=opensourceinitiative&logoColor=white" alt="OPEN-SOURCE">
</p>

---

A modular remote administration toolkit for red-team engagements and
lab environments. Three components, two executables and one server:


- **misl.exe** — agent. Runs on the target machine, connects to C2,
  executes commands, returns results.
- **misl-operator.exe** — operator console. Windows desktop app
  (Win32 + Direct2D), connects to C2, manages agents.
- **misl-c2** — command-and-control server. Routes commands between
  operator and agents, stores state in SQLite.

---

## Features

### Remote control

- **Live screen capture** — grab the target's screen to PNG on demand,
  optional auto-refresh for near-real-time monitoring.
- **Camera snapshots** — pull a frame from the default or selected
  camera device; supports multiple devices where the driver exposes them.
- **File manager** — browse remote drives, upload, download, delete,
  and execute files. Two-pane layout, drag & drop, progress and cancel.
- **Interactive terminal** — run `cmd.exe` commands on the target,
  stream stdout/stderr back in real time, command history.
- **Process manager** — list processes with PID, user, RAM and CPU,
  search, kill, start new ones.
- **Keylogger** — low-level global keyboard hook, live stream in the
  operator, clear buffer, export to `.txt`.
- **System control** — shutdown, restart, lock, list disks with usage,
  list local users.

### Operator console

- Native Windows desktop app (Win32 + Direct2D), single EXE, `/MT`,
  no external DLLs.
- Dark theme, sidebar with agent list, online/offline status, search.
- Tabs per agent: Overview, Screen, Camera, Files, Terminal, Processes,
  Keylog, System.
- Toast notifications, progress bars, confirmation for destructive
  actions, hotkeys (`Ctrl+K` search, `Ctrl+Enter` run command).
- Config stored encrypted with DPAPI, never plaintext.
- Auth: token + TOTP (2FA).

### Agent

- Single EXE, `/MT`, no installer, no external dependencies.
- Modules: screenshot, camera, files, shell, process, keylog, system.
- Transport: WebSocket over TLS (WinHTTP WebSocket API), reconnect
  with exponential backoff + jitter, randomized heartbeat.
- Crypto: ECDH (Curve25519) key exchange + AES-256-GCM session.
- Evasion: string encryption (XOR/RC4), indirect syscalls, AMSI patch,
  ETW patch, anti-VM, anti-debug.
- Anti-forensics: in-memory logs, wipe on exit, optional PE header
  wipe, temp file cleanup.
- Persistence (optional, off by default): HKCU Run, Task Scheduler,
  WMI subscription.

### C2 server

- Python 3.11+, FastAPI + WebSocket.
- Endpoints: `/ws/agent`, `/ws/operator`, REST for agents, commands,
  upload, download.
- SQLite storage for agents, commands, files, keylogs.
- Operator auth: token + TOTP; agent auth: shared secret embedded at
  build time.
- Logs never contain the operator's IP.
- Listens on `localhost` only; exposed through a reverse proxy.

### Anonymity & operator protection

| Layer | Hides from |
|---|---|
| CDN fronting (Cloudflare/Fastly) | Origin IP from target and network analysts |
| Only 443 exposed, internal ports localhost | Port scanning, service fingerprinting |
| Random WS path `/api/v1/stream/<token>` | Automated scanners probing `/ws` |
| Headers masked as a normal API | Traffic fingerprinting by `Server:` header |
| Agent binary: C2 domain XOR/RC4-encrypted | Static string extraction from the binary |
| Fallback channels (DNS TXT, ICMP, Telegram) | Primary C2 blocked by network policy |
| Operator via Tor (.onion) or WireGuard | Operator's real IP from C2 host logs |
| Firewall: only 443 (and 80 for ACME) | Port scans, lateral movement |
| C2 in Docker, read-only FS | Post-compromise tampering |
| Anonymous domain registration, crypto payment | Registrar subpoena linking domain to identity |
| TLS 1.3 + ECDH + AES-256-GCM | Passive TLS interception (MITM with a forged cert) |

**What can still expose you:** timing analysis, packet-size
correlation, DNS leaks, CDN subpoena, traffic correlation between
operator and agent sides, host forensics (memory artifacts, unpatched
ETW providers, crash dumps, prefetch, amcache). The layers above
reduce surface, they do not grant legal anonymity.

---

## Components

### Agent — `misl.exe`

| Module | What it does | APIs | Limits |
|---|---|---|---|
| screenshot | Captures the screen to PNG | BitBlt, GDI+ | Single monitor; multi-monitor needs per-display capture |
| camera | Grabs a frame from the default camera | DirectShow / Media Foundation | Depends on device; some drivers refuse |
| files | List, upload, download, delete, execute | FindFirstFile, CreateFile | Chunked transfer; large files are slow over WSS |
| shell | Runs `cmd.exe /c <line>`, returns stdout/stderr | CreateProcess, pipes | Inherits agent's integrity level |
| process | List, kill, start processes | Toolhelp32, OpenProcess, TerminateProcess | Killing protected processes requires higher privileges |
| keylog | Low-level keyboard hook, buffered | SetWindowsHookEx(WH_KEYBOARD_LL) | Global hook; some secure-input fields are not captured |
| system | Shutdown, restart, lock, list disks/users | ExitWindowsEx, GetLogicalDrives, NetUserEnum | Shutdown/restart require SeShutdownPrivilege |

### Operator — `misl-operator.exe`

Windows desktop console. Win32 + Direct2D/DirectWrite, WinHTTP
WebSocket. Single EXE, `/MT`, no external DLLs.

### C2 — `misl-c2`

Python 3.11+, FastAPI + WebSocket.

---
## Contact Us: 
Telegram: @GazkisMeneger_BOT


## The developers aren't responsible for your actions. The program doesn’t promise full protection or anonymity, use it at your own risk.


