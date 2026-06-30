# NATION-STATE CYBER WEAPONS ARSENAL

> **Live Web Page:** [`https://adhihub.github.io/nationstate-cyber-tools/`](https://adhihub.github.io/nationstate-cyber-tools/)

[![Matrix](https://img.shields.io/badge/CLASSIFIED-INTELLIGENCE-00f5d4?style=for-the-badge&labelColor=0a0e17)]()
[![Tools](https://img.shields.io/badge/50%2B-TOOLS-ef4444?style=for-the-badge&labelColor=0a0e17)]()
[![Pricing](https://img.shields.io/badge/%2420M-MAX%20PAYOUT-0ea5e9?style=for-the-badge&labelColor=0a0e17)]()

---

## TABLE OF CONTENTS

- [1. NSA / EQUATION GROUP ARSENAL](#1-nsa--equation-group-arsenal)
  - [1.1 Exploits & Implants](#11-exploits--implants)
  - [1.2 Firmware & Persistence](#12-firmware--persistence)
  - [1.3 Post-Exploitation & Frameworks](#13-post-exploitation--frameworks)
- [2. CIA / VAULT 7 ARSENAL](#2-cia--vault-7-arsenal)
  - [2.1 Implants & Persistence](#21-implants--persistence)
  - [2.2 Air-Gap & Physical Access](#22-air-gap--physical-access)
  - [2.3 Frameworks & Tools](#23-frameworks--tools)
- [3. NSO GROUP / PEGASUS ECOSYSTEM](#3-nso-group--pegasus-ecosystem)
- [4. RUSSIAN CYBER OPERATIONS](#4-russian-cyber-operations)
- [5. CHINESE CYBER OPERATIONS](#5-chinese-cyber-operations)
- [6. COMMERCIAL SPYWARE VENDORS](#6-commercial-spyware-vendors)
- [7. THE ZERO-DAY MARKET ECONOMY](#7-the-zero-day-market-economy)
- [8. BLACK MARKET ACQUISITION](#8-black-market-acquisition)
- [9. MARKET DYNAMICS & TRENDS](#9-market-dynamics--trends)
- [10. SOURCES](#10-sources)

---

## 1. NSA / EQUATION GROUP ARSENAL

The **Equation Group** is the NSA's elite cyber warfare unit (part of TAO — Tailored Access Operations). Kaspersky called them "the most advanced threat actor" — operating at a level beyond any known adversary. Their tools were leaked by **The Shadow Brokers** in 2016-2017, causing the WannaCry and NotPetya global ransomware epidemics.

### 1.1 Exploits & Implants

#### ETERNALBLUE (MS17-010 / CVE-2017-0144)

| Attribute | Detail |
|-----------|--------|
| **Type** | SMBv1 Remote Code Execution Exploit |
| **Target** | Windows XP/7/8/10, Server 2003-2016 |
| **Delivery** | Remote, unauthenticated, wormable |
| **Mechanism** | Buffer overflow in SMBv1 server → kernel-mode code execution |
| **NSA Codename** | ETERNALBLUE |
| **Leaked** | April 14, 2017 (Shadow Brokers) |
| **Impact** | Weaponized by WannaCry ($4B damages, 200K+ systems in 150 countries), NotPetya ($10B damages), and countless ransomware operations |
| **Black Market Value** | $500,000 - $5,000,000 (estimated) |
| **Current Status** | Patched (MS17-010), but still used by unpatched systems globally |

**How it works:** ETERNALBLUE exploits a vulnerability in the SMBv1 (Server Message Block v1) protocol. By sending a specially crafted packet to the SMBv1 server, it triggers a buffer overflow in the kernel driver `srv2.sys`. This allows the attacker to execute arbitrary code at Ring 0 (kernel level) — the highest privilege level in Windows. Once executed, it deploys the DOUBLEPULSAR backdoor.

**Why it's powerful:** Kernel-level SMB exploitation means no authentication required, no user action needed, and complete control of the target system. Widespread because SMBv1 could not be easily uninstalled on many Windows versions.

---

#### DOUBLEPULSAR (DOPU)

| Attribute | Detail |
|-----------|--------|
| **Type** | Kernel-mode SMB Backdoor Implant |
| **Target** | Windows systems (deployed via SMB) |
| **Persistence** | Memory-only (non-persistent) |
| **Privilege** | Ring 0 (kernel mode) |
| **Covert Channel** | SMB Trans2 SESSION_SETUP subcommand 0x0E |
| **Protocol** | SMB (Port 445) |
| **Infected** | 200,000+ systems within 2 weeks of leak |
| **Operations** | Ping (0x23), Execute shellcode (0xC8), Inject DLL, Uninstall (0x77) |

**How it works:** DOUBLEPULSAR is a kernel-mode backdoor that hooks the `SYSENTER` instruction (the Windows kernel system call entry point). It intercepts SMB packets and looks for malformed Trans2 SESSION_SETUP commands (subcommand 0x0E — a subcommand that is "Not Implemented" in legitimate Windows SMB implementations). When it detects the magic "ping" packet, it responds. When it detects the "exec" packet, it executes the included shellcode in kernel mode.

**Why it's powerful:** Memory-only operation means no files on disk. Kernel-level execution means it can do anything the OS can do — and with no authentication check at all.

---

#### ETERNALROMANCE

SMBv1 RCE exploit, part of the Equation Group's SMB exploit suite alongside ETERNALBLUE and ETERNALSYNERGY. Provides redundancy — if one vector is blocked, another can be used. Also deploys DOUBLEPULSAR.

#### ETERNALSYNERGY

Another SMBv1 RCE vector. Gives operators multiple options for exploiting Windows SMB stacks depending on target configuration.

#### FUZZBUNCH

Interactive GUI-based exploit framework used to manage and deploy Equation Group exploits. Provides a menu-driven interface for configuring ETERNALBLUE/ETERNALROMANCE/ETERNALSYNERGY, selecting targets, and deploying DOUBLEPULSAR. Think of it as the NSA's internal Metasploit.

---

### 1.2 Firmware & Persistence

#### GRAYFISH (KillSuit)

The NSA's most sophisticated persistence platform. This is a **bootkit + registry-only malware** combination.

| Attribute | Detail |
|-----------|--------|
| **Persistence** | VBR (Volume Boot Record) + Registry |
| **Files on disk** | ZERO — entirely registry-resident |
| **Encryption** | AES-256 with machine-bound key |
| **Decryption** | SHA-256 of NTFS Object_ID hashed 1000x → machine-unique AES key |
| **Boot Layers** | 4-5 decryption stages before OS boots |
| **Self-Destruct** | Immediate if any decryption stage fails |
| **Survival** | Survives OS reinstall, disk format |
| **Installer** | DOGROUND.exe (exploits ElbyCDIO.sys driver) |

**How it works:**
1. DOGROUND.exe exploits `ElbyCDIO.sys` (a CloneCD driver) to gain kernel code execution
2. GRAYFISH installs a VBR bootkit — it modifies the Volume Boot Record to load a malicious bootloader
3. The bootloader goes through 4-5 layers of AES decryption, using a key derived from the machine's unique NTFS Object_ID
4. If any decryption stage fails (indicating forensic analysis), the entire implant self-destructs
5. After boot, the main payload is loaded from the registry (no files on disk)
6. Communication with C2 goes through the DanderSpritz framework

**Why it's powerful:** No antivirus can find what's not on disk. The self-destruct mechanism makes forensic analysis nearly impossible. The bootkit ensures persistence even after complete OS reinstallation.

---

#### IRONCHEF (HDD Firmware Implant)

| Attribute | Detail |
|-----------|--------|
| **Target** | Western Digital, Seagate, Toshiba, Samsung HDDs |
| **Persistence** | Hard drive firmware |
| **Survival** | OS reinstall, disk format, low-level format |
| **Detection** | Impossible from within the OS |
| **Capabilities** | Read/write/redirect disk sectors |

**How it works:** IRONCHEF infects the firmware that runs on the hard drive's own microcontroller. The hard drive's firmware is executed before the OS even loads. The implant operates at the drive level — it can intercept, modify, or redirect any disk read/write operation. To the OS and antivirus, the drive appears completely normal. The malicious code executes on the drive's internal processor, completely invisible to the host computer.

**Why it's powerful:** Formatting the drive doesn't help — the firmware persists. Even replacing the drive may not help if the firmware was infected before installation. Only specialized hardware firmware reflash tools can remove it.

---

#### FANNY (Air-Gap USB Bridge)

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Bidirectional USB covert channel for air-gapped networks |
| **Vector** | USB storage devices |
| **Capabilities** | Exfiltrate data, deliver commands |
| **Historical** | Connected to STUXNET operation |
| **Role in STUXNET** | Mapped air-gapped Iranian nuclear enrichment networks pre-2009 |

**How it works:** FANNY uses specially crafted USB drives as communication bridges. When an infected USB drive is inserted into an air-gapped computer, FANNY installs a hidden partition on the USB drive containing encrypted data. When that same drive is later inserted into an internet-connected computer, FANNY exfiltrates the data. Commands flow in the reverse direction.

#### SCHOOLBUS (SATA RF Air-Gap Bridge)

Converts electromagnetic emissions from SATA cables into a covert radio frequency data channel. A nearby receiver picks up the RF signal and decodes the exfiltrated data. No network connection of any kind is needed.

#### DOUBLEFANTASY / TRIPLEFANTASY (Validation Implants)

First-stage implants that conduct environmental keying — they verify the target is of intelligence interest before deploying the full platform. If the target is not interesting (wrong type of data, not the target person's system), it silently uninstalls. This minimizes operational exposure.

---

### 1.3 Post-Exploitation & Frameworks

#### DANDERSPRITZ (Full Post-Exploitation Framework)

| Attribute | Detail |
|-----------|--------|
| **Type** | Post-exploitation framework + Listening Post |
| **Deployment** | Via DOUBLEPULSAR or PEDDLECHEAP |
| **Components** | DoubleFeature (diagnostics), KillSuit (GrayFish persistence) |
| **Capabilities** | Recon, lateral movement, AV bypass, exfiltration |
| **Interface** | Python-based LP with Java GUI |

#### PEDDLECHEAP (Initial Implant Loader)

First-stage loader injected into `lsass.exe`. Minimal footprint — enough to connect back to C2, receive commands, and install the full DanderSpritz framework. Usually delivered by DOUBLEPULSAR.

#### NOPEN (Unix RAT)

Remote access tool for Unix/Solaris systems. Client-server architecture with RC6 encrypted C2. Targets SunOS 5.8 and enterprise Unix systems found in financial/telecom infrastructure.

#### UNITEDRAKE (Data Exfiltration Framework)

Modular exfiltration framework supporting HTTP, DNS, ICMP, and email channels. Encrypts and fragments stolen data. Supports resume capability for large exfiltration jobs.

#### SECONDDATE (Multi-Platform Implant)

Implant for Linux, FreeBSD, Solaris, and JunOS. Persistent backdoor with encrypted C2. Configurable persistence mechanisms per target OS.

---

## 2. CIA / VAULT 7 ARSENAL

The **Vault 7** documents were a massive leak of CIA cyber weapons tools published by WikiLeaks starting March 2017. The release covered 25+ distinct projects from the CIA's Center for Cyber Intelligence (CCI).

### 2.1 Implants & Persistence

#### FLUXBABEON (UEFI/BIOS Rootkit)

| Attribute | Detail |
|-----------|--------|
| **Target** | UEFI/BIOS SPI flash memory |
| **Persistence** | Below the OS — in firmware |
| **Survival** | BIOS update, OS reinstall, disk format, disk replacement |
| **Detection** | Impossible from within the OS |
| **Reinfection** | Can reinfect OS even after complete cleanup |

**How it works:** FLUXBABEON infects the SPI (Serial Peripheral Interface) flash chip that stores the computer's UEFI/BIOS firmware. It modifies the firmware boot chain to load a malicious bootloader before the operating system even starts. Since it lives in the firmware, no amount of OS-level cleanup can remove it. Even updating the BIOS may not help if the infection persists in the SPI flash write-protection bypass.

**Why it's powerful:** This is the deepest possible persistence on a modern computer. You can replace the hard drive, reinstall Windows, format everything — and FLUXBABEON will survive. It will re-infect the clean OS during boot.

---

#### GRASSHOPPER (Modular Windows Implant Builder)

| Attribute | Detail |
|-----------|--------|
| **Type** | Modular malware builder for Windows |
| **Output** | EXE, DLL, SYS, PIC (x86 and x64) |
| **Persistence** | Multiple module options (registry, scheduled tasks, WMI) |
| **Survey Engine** | Pre-installation rules engine checks target conditions |
| **Source** | Borrowed from Carberp rootkit, Shamoon, HiKit, Nuclear Exploit Kit |

**How it works:** GRASSHOPPER is a command-line builder tool. An operator configures:
1. Which payload to deliver
2. Which persistence module to use (registry, scheduled task, WMI, etc.)
3. Pre-installation rules (only install if target has specific OS version, AV product, etc.)

The builder assembles these into a single Windows installer executable. When run on a target, it first performs the survey checks, and only installs the payload if all rules match.

---

#### WEEPING ANGEL (Samsung Smart TV Implant)

| Attribute | Detail |
|-----------|--------|
| **Target** | Samsung F-series Smart TVs |
| **Developer** | CIA + MI5 (joint development) |
| **Installation** | Physical USB insertion |
| **Capability** | Covert audio recording via built-in microphone |
| **Deception** | TV appears to be turned off while recording |
| **Exfiltration** | Via USB or compromised Wi-Fi hotspot |
| **Listening Tool** | Live Listening Tool on operator laptop |

**How it works:** The EXTENDING implant is loaded onto a USB stick. The operator physically inserts it into the target Samsung TV. The installer deploys the implant and settings file. When the TV is next powered on, EXTENDING runs persistently. The TV's built-in microphone records ambient audio. The TV's LED appears off (the "Weeping Angel" effect — turning the TV into a surveillance device). Audio is stored locally or exfiltrated when the TV connects to Wi-Fi.

---

#### DARK MATTER (Apple Mac Firmware Implants)

Suite of tools targeting Apple Mac firmware — both Intel Macs and potentially T2-chip systems. Includes boot-level rootkits for EFI/UEFI firmware persistence. Sub-projects include Sonic Screwdriver.

#### IMPERFECT AUTUMN (ATM Exploit Chain)

Targeted exploits for NCR and Diebold ATMs. Allows remote cash dispensing and card skimming. Exploits vulnerabilities in Windows-based ATM operating systems (Windows XP Embedded).

#### CHERRY BLOSSOM (Wireless Router Implant)

Router firmware implant framework. Targets home/office routers for MITM, traffic interception, and content injection. Operates at firmware level — invisible to devices on the network.

---

### 2.2 Air-Gap & Physical Access

#### BRUTAL KANGAROO (Air-Gap Jumping USB Worm)

| Attribute | Detail |
|-----------|--------|
| **Target** | Air-gapped Windows networks |
| **Vector** | USB thumbdrives |
| **Exploit** | LNK file code execution (patched June 2017) |
| **Components** | 4 tools in the suite |

**Components:**

| Component | Function |
|-----------|----------|
| **Drifting Deadline** | USB drive infection tool — infects thumbdrives |
| **Shattered Assurance** | Server that automates thumbdrive infection remotely |
| **Shadow** | Covert C2 mesh network across isolated machines using shared drives |
| **Broken Promise** | Post-processor for decrypting exfiltrated data |

**How it works:**
1. An internet-connected computer in the target organization is infected (via spear-phishing, web compromise, etc.)
2. Shattered Assurance is deployed on this machine
3. When a USB drive is inserted, Drifting Deadline infects it automatically
4. The infected USB is taken (unwittingly) to an air-gapped machine and inserted
5. Shadow spreads across the air-gapped network via shared USB drives
6. Multiple Shadow instances form a covert mesh C2 network via USB drives
7. Payloads and commands propagate, data exfiltrates back

---

#### PANDEMIC (File Server Implant)

Infects files as they pass through a compromised Windows file server. The file server becomes an infection distribution point — clients downloading files get infected. Files on disk appear clean; infection happens in transit.

#### EXPRESS LANE (Biometrics Bypass)

Cloning and bypassing biometric authentication. Clones fingerprints, bypasses iris scanners, defeats physical access controls.

---

### 2.3 Frameworks & Tools

#### MARBLE FRAMEWORK (Anti-Forensic Obfuscation)

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Obfuscate and misattribute CIA malware |
| **Capability** | Mask malware to appear as Russian, Chinese, North Korean, or Iranian |
| **Strings** | 600+ unique deobfuscation strings |
| **Languages** | Multiple languages for false-flag attribution |

**How it works:** MARBLE wraps CIA malware in obfuscation layers that make it appear to be from another nation-state. If the malware is discovered and analyzed, forensics will attribute it to Russia, China, North Korea, or Iran — not the CIA. Contains a "decoy" attribution module for each target country.

#### HIVE (Multi-Platform C2 Framework)

Encrypted C2 framework for Windows, Mac, and Linux. Provides tasking, data exfiltration, and implant management.

#### SCRIBBLES (Steganographic Watermarking)

Embeds invisible steganographic watermarks in documents. If a classified document leaks, the watermark reveals which officer or operation was the source. Acts as an insider-threat tripwire.

---

## 3. NSO GROUP / PEGASUS ECOSYSTEM

**NSO Group** is an Israeli cyber arms company that developed **Pegasus** — the most infamous commercial spyware platform. Sold exclusively to "approved government clients" but routinely used against journalists, activists, lawyers, and politicians worldwide.

### PEGASUS (Full Mobile Surveillance Suite)

| Attribute | Detail |
|-----------|--------|
| **Developer** | NSO Group (Israel) |
| **Type** | Full mobile surveillance implant |
| **Targets** | iOS and Android |
| **Installation** | Zero-click exploits (iMessage, WhatsApp) |
| **Cost (2016)** | $500K setup + $650K per 10 targets |
| **Ghana Deployment** | $8M total |
| **Annual Maintenance** | 17-22% of system cost |
| **License (2018-2020)** | $6.8M/year per government |
| **Status** | US Entity List (Nov 2021), still operational |

**What it extracts:**
- SMS and iMessage
- WhatsApp, Signal, Telegram, Viber messages
- Call logs and contact lists
- Photos and videos
- GPS location history
- Microphone audio (room surveillance)
- Camera images
- Saved passwords
- Browser history
- Email
- Calendar events
- Notes and reminders

**How infection works — three generations of exploits:**

| Generation | Name | Method | Status |
|------------|------|--------|--------|
| 1 | **HEAVEN** (2018) | NSO's own WhatsApp Installation Server (WIS) impersonated official WhatsApp client to send malicious messages | Patched by WhatsApp 2018 |
| 2 | **EDEN** (2018-2019) | Used WhatsApp's own relay servers instead of custom WIS | Patched by WhatsApp 2019 |
| 3 | **ERISED** (2019-2020) | Still used WhatsApp's servers, different exploit technique | Patched by WhatsApp May 2020 |
| 4 | **FORCEDENTRY** (2021+) | iMessage zero-click via PDF parsing exploit (CVE-2021-30860) | Patched by Apple Sep 2021 |
| 5 | **KISMET** | Earlier iMessage zero-click (pre-BlastDoor) | Patched by iOS 14 BlastDoor |

**FORCEDENTRY** was the breakthrough exploit discovered by Citizen Lab:
- Zero-click — target only needs to receive an iMessage
- Exploits Apple's PDF parsing library (ImageIO)
- Circumvents BlastDoor (the iOS 14 mitigation specifically designed to prevent iMessage exploitation)
- Worked on fully-patched iOS, macOS, and WatchOS devices
- Used since at least February 2021 before discovery

**Post-infection capabilities:**

| Module | Function |
|--------|----------|
| **HOMESICK** | Anti-forensic self-delete — wipes all traces |
| **PHOENIX** | SMS-based C2 when internet is cut |
| **CHRYSAOR** | SMS phishing campaign management panel |

---

## 4. RUSSIAN CYBER OPERATIONS

### BLACKENERGY / CRASHOVERRIDE / INDUSTROYER (ICS Sabotage)

| Tool | Target | Year | Impact |
|------|--------|------|--------|
| **BlackEnergy** | Ukrainian power grid | 2015 | 230,000 people without power |
| **CRASHOVERRIDE** | Ukrainian substations | 2016 | Automated, modular ICS attack |
| **INDUSTROYER** | Ukrainian substations | 2016 | Direct IEC-101 serial protocol manipulation |

**How BlackEnergy worked:**
1. Spear-phishing emails with malicious Microsoft Office attachments (Excel spreadsheet macros)
2. Macro installed BlackEnergy trojan on the control center operator's workstation
3. Attackers conducted reconnaissance within the SCADA network for months
4. On the attack day, they remotely opened circuit breakers at 30 substations
5. They also deployed a serial-to-Ethernet converter firmware kill (KillDisk) to prevent reconnection
6. Call centers were DoS'd simultaneously to prevent reporting

### X-AGENT / X-TUNNEL (APT28/Fancy Bear)

| Attribute | Detail |
|-----------|--------|
| **Developer** | GRU (Main Intelligence Directorate) / Unit 26165 |
| **Alias** | APT28, Fancy Bear, Sofacy, Sednit |
| **Target** | Governments, militaries, political targets |
| **Notable** | 2016 DNC hack, Olympic doping scandal, Ukrainian military |

**How X-TUNNEL works:** X-AGENT is the implant deployed on targets. It uses X-TUNNEL to create encrypted tunnels that route stolen data out of the network. The tunnels are disguised as HTTPS traffic to legitimate news websites — making detection extremely difficult since it blends in with normal web traffic.

### TURLA HAMMERTOSS / COMCLEANER (Stealth C2)

| Attribute | Detail |
|-----------|--------|
| **Developer** | Turla (aka Waterbug, Snake) |
| **C2 Channels** | Twitter, GitHub, StackOverflow |
| **Operation** | Uses public platform comments as dead-drop C2 |

**How it works:** The COMCLEANER backdoor reads tweets from a specific Twitter account for commands. When an operator posts a new tweet (with encrypted instructions), all infected systems see it. Responses can be posted as GitHub comments or StackOverflow edits. You cannot block these C2 channels without disrupting legitimate business operations.

### DROPPING LIME / KOPILUWAK

| Tool | Purpose |
|------|---------|
| **DROPPING LIME** | Compromised WordPress sites as C2 reflectors |
| **KOPILUWAK** | Satellite communication intercept via NTLite hub compromise |

### NOTPETYA / SHAMOON / WHISPERGATE (Destructive Wipers)

| Malware | Target | Year | Damage |
|---------|--------|------|--------|
| **Shamoon** | Saudi Aramco | 2012 | 35,000 workstations destroyed |
| **NotPetya** | Ukraine (global spread) | 2017 | $10B+ total damages |
| **WhisperGate** | Ukrainian organizations | 2022 | Data destruction + MBR overwrite |

---

## 5. CHINESE CYBER OPERATIONS

### GHOSTNET (Diplomatic Espionage)

Sustained espionage targeting 100+ diplomatic missions. Uses Office exploits and custom RATs. Operated by PLA Unit 61398. Extracts diplomatic cables, negotiation strategies, and classified documents.

### THROATFYRE (GPU Fan Acoustic Exfiltration)

| Attribute | Detail |
|-----------|--------|
| **Purpose** | Exfiltrate data from air-gapped systems |
| **Method** | GPU fan speed modulation → acoustic pulses |
| **Receiver** | Nearby smartphone microphone |
| **Stealth** | No network connection required |

**How it works:** A compromised computer's GPU fan speed is modulated to create acoustic pulses that encode stolen data. A nearby smartphone (placed by a operative or unknowing insider) captures the fan noise with its microphone. The audio is decoded to extract the data. The physical air gap is defeated — no network connection exists.

### COMMENT CREW RAT

Uses blog comment sections on legitimate websites as dead-drop C2 channels. Encrypted commands are posted as comments on popular blogs. Infected systems periodically scrape those comment sections for instructions. C2 traffic is indistinguishable from normal web browsing.

### EMISSARY / LURID

Custom RATs embedded in legitimate software updates. Once a user downloads a trojanized update, the RAT installs with full system persistence. Targets diplomatic and technology organizations.

---

## 6. NORTH KOREA / LAZARUS GROUP (HIDDEN COBRA)

The **Lazarus Group** (aka APT38, HIDDEN COBRA) is North Korea's premier cyber warfare unit — operating a "malware factory" producing samples via multiple independent conveyors. Responsible for the Sony Pictures hack (2014), Bangladesh Bank heist ($81M), WannaCry ($4B), and DarkSeoul. Operates since at least 2009. Attributed to **Bureau 121** (Reconnaissance General Bureau).

### How Lazaraus Operates — The Factory Model

Lazarus maintains a multi-generational malware arsenal with shared codebases spanning 15+ years. Code reuse connects Brambul (2009) → Joanap → KorDllBot → Destover → Duuzer → WannaCry (2017). They use:
1. **Stage 1** — Rudimentary backdoors via spear-phishing (password-protected ZIP, hardcoded pw `Bb102@j...t3hg`)
2. **Stage 2** — If target is interesting, deploy advanced code wrapped in encrypted DLL loaders
3. **Stage 3** — Custom packers, RC4 encryption, registry-hived payloads
4. **Destruction** — MBR wipers + Eldos RawDisk driver for physical disk overwrite

### BRAVBUL / JOANAP (SMB Worm + P2P Backdoor)

| Tool | Type | How It Works | What You Can Do | Black Market |
|------|------|-------------|-----------------|--------------|
| **Brambul** | SMB worm | Brute-force attack on port 445/TCP using hardcoded password list (`"123123"`, `"abc123"`, `"computer"`, `"iloveyou"`, `"login"`, `"password"`). On success, creates net share to C: drive. Sends victim credentials to hardcoded email via Google SMTP. | Gain initial access to Windows networks via weak SMB passwords. Enables lateral movement and additional payload drops. | Gov-only. Available via leaked samples on GitHub from Operation Blockbuster |
| **Joanap** | P2P backdoor | Registered as service `scardprv.dll` (SmartCard Protector). RC4-encrypted P2P C2 — infected machines talk to each other. Commands: open backdoor, send specific files, save/delete files, download/run executables, start/kill processes. | Full remote control. Steal files, run commands, pivot through infected peers. Peer-to-peer C2 makes takedown nearly impossible. | Gov-only |

### DESTROVER (Sony Pictures 2014)

**How it works step-by-step:**
1. Spear-phishing email with malicious attachment delivered to Sony employees
2. Dropper (password-protected ZIP with pattern `"MYRES..."`) extracts Destover components
3. Custom Mimikatz-like tool harvests domain admin credentials
4. **Lateral movement:** SMB/WMI/PsExec — spreads to 3,062 of Sony's 6,797 workstations + 837 of 1,555 servers
5. **Destruction phase:**
   - Installs Eldos RawDisk driver (signed kernel driver for raw disk access)
   - Opens handle: `\\.\ElRawDisk\??\PhysicalDrive0#<LICENSE_KEY>`
   - Overwrites MBR with custom bootloader displaying "This is just a warning..."
   - Wipes files on all drives using RawDisk IOCTLs
   - Corrupts partition tables
6. **Data exfiltration:** Steals 100TB+ of data before wiping

**What you can do:** Destroy an entire organization's IT infrastructure, steal corporate intelligence, wipe forensic evidence. Sony lost 3,062 workstations + 837 servers permanently.

**Black market:** NOT available for sale. Leaked samples exist in Operation Blockbuster archives. Government weapon only.

### DARKSEOUL (2013 — 32,000 Machines Wiped)

**How it works:**
1. Long-term espionage via Operation Troy (2009-2012) — recon and data exfiltration
2. IRC-controlled botnet (XwDoor/Keydoor family, earliest sample Jan 2009)
3. Prioxer backdoor installed on target networks
4. **March 20, 2013 14:00 KST** — simultaneous activation:
   - MBR overwrite on 32,000 machines
   - Boot sector corruption
   - Target: MBC, YTN (broadcasters), Shinhan Bank, Nonghyup Bank, Jeju Bank
5. Three South Korean banks + two broadcasters crippled simultaneously

**Market:** Gov-only. DarkSeoul malware variants leaked through security research publications.

### WANNACRY (2017 — $4B Damages)

Attributed to Lazarus subgroup **Bluenoroff** (aka APT38). Kaspersky found code overlap: WannaCry's SMB module shared code with Brambul (2009), Joanap, and DeltaAlfa — all Lazarus tools.

**How WannaCry worked technically:**
1. **Delivery:** Deployed via ETERNALBLUE (NSA exploit leaked by Shadow Brokers — not built by Lazarus)
2. **DoublePulsar** implanted as backdoor
3. **Encryption:** AES-128 per-file with RSA-2048 key wrapping
4. **Propagation:** SMB worm — self-spreading without human action
5. **Kill switch:** Hardcoded domain `iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com` — researcher registered it, stopping the worm
6. **Impact:** 200,000+ systems in 150 countries in 72 hours

**Black market:** WannaCry source code and builder leaked and traded on underground forums. Active variants still exist. Estimated $500-$5K on dark web for modified builders.

---

## 7. IRANIAN CYBER OPERATIONS (APT33 / APT34 / APT35)

Iran operates several state-sponsored cyber units. Their evolution: from basic website defacements (2009-2012) → destructive disk wipers (2012-2019) → cross-platform file-level destruction (2020-2022) → identity weaponization via living-off-the-land (2023-2026).

### SHAMOON 1/2/3 (Disttrack — 2012, 2016, 2018)

The most famous Iranian wiper. Targets Saudi Arabia primarily (Saudi Aramco 2012: 35,000 workstations).

**How Shamoon 1 works (step-by-step):**
1. **Delivery:** Spear-phishing → dropper extracts three components: `DrvLoader.exe`, `ElRawDisk.sys`, `Main.exe`
2. **Driver installation:** `ElRawDisk.sys` (signed Eldos RawDisk driver) installed via Service Control Manager (`sc.exe`)
3. **Device handle:** Opens `\\.\ElRawDisk\??\PhysicalDrive0#<LICENSE_KEY>` 
4. **License bypass:** System date changed to August 2012 (trial license exploit)
5. **IOCTL call:** Uses `FSCTL_GET_RETRIEVAL_POINTERS` to locate physical disk sectors
6. **Overwrite:** Writes garbage data over MBR + critical files, then system image
7. **Display:** Burning US flag image shown after reboot (Shamoon 1) / Ukraine flag (Shamoon 2)

**Shamoon 2 (2016):** Same technique, updated license key, added phone-home reporting module. Target: Saudi Aramco, RasGas.

**Shamoon 3 (2018):** Same technique. Target: Saipem (Italian oil & gas). ~300 servers + 100 PCs wiped.

**What you can do with Shamoon:** Permanently destroy an organization's entire computing infrastructure by overwriting physical disk sectors. Requires kernel-level access (RawDisk driver). Data is unrecoverable.

**Black market:** NOT available for purchase. Custom Iranian government weapon. RawDisk driver (legitimate signed binary) is available for $500-$2K; Shamoon malware itself is not sold.

### ZEROCLEARE / DUSTMAN (2019 — Energy Sector)

**Improvement over Shamoon:**
1. Uses **unsigned** ElRawDisk driver loaded via Turla Driver Loader (TDL) — exploits signed VBoxDrv driver vulnerability to bypass Driver Signature Enforcement
2. Single executable (vs Shamoon's multi-file)
3. Overwrites volumes with `0x55` or text message
4. **Ultimate attack chain:** VPN appliance zero-day → VPN server compromise → Domain admin theft → Deploy via AV management console via PsExec → Simultaneous wipe of all systems

**Black market:** Not available. Iranian government weapon.

### APT33 CUSTOM TOOLS

| Tool | Type | How It Works | What You Can Do | Market |
|------|------|-------------|-----------------|--------|
| **TURNEDUP** | Backdoor | File upload/download, reverse shell (cmd.exe), screenshot, system info. Delivered via DROPSHOT. Early Bird APC injection into rundll32.exe. Registry persistence. | Full remote control of Windows targets | Gov-only |
| **SHAPESHIFT / STONEDRILL** | Backdoor + Wiper | Dual-purpose. Backdoor mode: full RAT. Wiper mode: MBR clear + file delete on all volumes. Farsi language artifacts. Anti-emulation (sleep loops, API call sequence check). | Remote control OR total system destruction | Gov-only |
| **DROPSHOT** | Dropper | Drops and launches TURNEDUP or SHAPESHIFT. Anti-emulation via sleep loops + VBScript self-deletion. Password-protected payloads. | Deploy APT33 payloads on target | Gov-only |
| **POWERTON** | PowerShell implant | Fully PowerShell-based. Encrypted C2. Multiple persistence (scheduled tasks, WMI, Registry). Mimikatz integration for hash dumps. | Stealthy PowerShell backdoor — fileless | Gov-only |
| **Tickler** | Multi-stage backdoor | Password spray → hijack educational accounts → create Azure subscriptions → deploy Stage 1 C2 → Stage 2 trojan dropper → SMB lateral movement → deploy RMM tools → AD snapshot theft. Azure-based C2 (`azurewebsites.net`). | Cloud-based espionage with no traditional infrastructure | Gov-only |

### APT34 / OILRIG TOOLS

| Tool | Type | How It Works | What You Can Do | Market |
|------|------|-------------|-----------------|--------|
| **SideTwist** | C backdoor | C2 via HTTP GET/POST on 443 (fallback 80). Commands: shell (101), download (102), upload (103), shell dup (104). Mersenne Twister PRNG encryption + Base64. Scheduled task `SystemFailureReporter` every 5min. Exits after one command. | Stealthy single-command backdoor — minimal footprint | Gov-only |
| **TwoFace** | ASPX web shell | Loader + embedded encrypted payload tested 22 iterations to reach 0 AV detection. Decrypted in memory. Password authentication. File mgmt, cmd exec, SQL queries, network scanning. | Full IIS server compromise, persistent foothold | Gov-only |
| **OilRig / Helminth** | Modular backdoor | DNS tunneling C2 (TONEDEAF). AES-encrypted HTTP/S. Credential harvesters. LOTL: PowerShell, WMI, native Windows utils. Cloud-based C2 (Azure, VPS). | Long-term espionage in government/financial targets | Gov-only |

---

## 8. SUPPLY CHAIN ATTACKS

### SOLARWINDS — SUNBURST / SUPERNOVA (2020)

**Scale:** 18,000 customers received trojanized Orion update. Active compromise of ~100 high-value targets (US govt, Fortune 500, security vendors like FireEye).

**Attribution:** Russian SVR (APT29 / Cozy Bear / UNC2452)

**How SUNBURST works (step-by-step):**
1. **Build system compromise:** Attackers infiltrated SolarWinds build infrastructure
2. **Code injection:** 4,000 lines of malicious code inserted into `SolarWinds.Orion.Core.BusinessLayer.dll`
3. **Trigger:** Injected into `RefreshInternal()` in `InventoryManager` class, called via `CoreBusinessLayerPlugin.Start()` → scheduled as Background Inventory
4. **Hibernation:** Malware sleeps for 2 weeks after installation — no C2 contact, no suspicious behavior
5. **Blocklist check:** Only executes if no forensic/AV analysis tools detected
6. **C2 Phase 1 (DNS — Passive):** DGA domain generation based on timestamp. DNS CNAME response from coordinator contains instructions (continue, halt, switch to active mode)
7. **C2 Phase 2 (HTTP — Active):** HTTPS to final C2 (certificate validation disabled). Mimics Orion Improvement Program (OIP) protocol with ~17 legitimate-looking commands
8. **Capabilities:** Transfer files, execute binaries, profile system, reboot, disable/enable services
9. **Exfiltration:** Blended in with legitimate Orion telemetry — indistinguishable from normal traffic
10. **Persistence:** Disguised as legitimate SolarWinds plugin; state stored in plugin config files

**What you can do:** Compromise ANY organization running SolarWinds Orion. Maintain persistent, stealthy backdoor on government networks, Fortune 500 companies, and security vendors. Exfiltrate data camouflaged in legitimate telemetry.

**SUPERNOVA (potentially unrelated):** Trojanized `app_web_logoimagehandler.ashx.b6031896.dll` — fileless .NET code execution via 4 new HTTP parameters. Client sends valid .NET code → compiled in-memory → executed immediately.

**Black market:** NOT available. SVR government weapon only. Source code never leaked.

---
### CCLEANER SUPPLY CHAIN (2017)

**Scale:** 2.27M downloads of trojanized CCleaner v5.33.6162 / CCleaner Cloud v1.07.3191

**Attribution:** Russian-linked group (potentially APT29 precursor)

**How it works:**
1. **Build system compromise:** Piriform's development environment breached (post-Avast acquisition)
2. **Stage 1** (in CCleaner.exe): Modified `__scrt_get_dyn_tls_init_callback` to redirect execution. Decrypts embedded blob → PIC PE loader + DLL. API resolution at runtime. Creates thread, sets up ROP chain for cleanup. Collects: computer name, volume serial, OS version, installed AV. Exfiltrates via HTTPS POST to hardcoded C2 (with DGA fallback)
3. **Stage 2** (GeeSetup_x86.dll — only delivered to target-matching victims): Drops trojanized binary (`TSMSISrv.dll` for x86, `EFACli64.dll` for x64). Writes encoded PE to Registry: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\WbemPerf\001`. Trojanized binary decodes and executes in-memory.
4. **Stage 3** (theoretical — never deployed in wild): In-memory PE downloaded from IP stego'd in GitHub/WordPress.com search results
5. **Kill switch:** C2 checked for `%TEMP%\spf` — if present, payload terminates

**C2 Database:** MySQL server contained info on ~700,000 infected machines. Only subset received Stage 2.

**Black market:** NOT for sale. Source code was analyzed and published by Talos/Cisco for research.

---
### NOTPETYA via M.E.Doc (2017)

**Scale:** $10B+ damages. Maersk ($300M), Merck, FedEx, Saint-Gobain, WPP crippled.

**Attribution:** GRU Sandworm.

**How supply chain vector worked:**
1. Attackers compromised M.E.Doc (Ukrainian tax accounting software company with mandatory government use)
2. Stole admin credentials to M.E.Doc update server
3. Modified `nginx.conf` to proxy update requests to attacker server
4. Backdoor code added to `ZvitPublishedObjects.dll` — polled attacker server every 2 minutes via `MeCom` object
5. Backdoor delivered NotPetya through legitimate M.E.Doc update mechanism

**NotPetya technical propagation:**
1. Runs as DLL (`rundll32.exe` with exported function)
2. **Credential harvesting:** WDigest extraction + Mimikatz LSASS dump → plaintext passwords + NTLM hashes
3. **Lateral movement (simultaneous multi-threaded):**
   - PsExec (renamed `dllhost.dat`) → `\\HOST\admin$`
   - WMIC → remote process creation
   - EternalBlue/ETERNALROMANCE (CVE-2017-0144/0145) — ONLY if Kaspersky AV detected
   - 44+ hardcoded credentials in PE (32KB patch area)
4. **Destruction:** Encrypts MFT entries → overwrites MBR with custom bootloader → Salsa20 + RSA-2048 file encryption
5. **Fake ransomware:** Bitcoin payment address unmonitored — attacker never intended to provide decryption
6. **Deception:** Displays fake CHKDSK screen during MBR overwrite

**What you can do:** Destroy every Windows system in a global organization. No recovery possible. The "ransomware" is purely destructive.

**Black market:** NOT available. GRU government weapon.

---

## 9. ADDITIONAL RUSSIAN CYBER WEAPONS

### SNAKE MALWARE (FSB Center 16 / Turla)

The most sophisticated Russian espionage implant. P2P mesh network of compromised machines worldwide.

**How it works:**
1. **Architecture:** Peer-to-peer network where infected machines act as relays for disguised traffic
2. **Kernel module:** Can function as server without opening any ports — completely stealth network presence
3. **Protocol stack (4 layers):**
   - Raw TCP/UDP, HTTP, SMTP, DNS, ICMP, raw IP
   - `enc` layer: Session encryption via Diffie-Hellman + pre-shared key (PSK)
   - Application layer: End-to-end encryption between operator and target
4. **Persistence:** Queue File stores per-implant state including `ustart` authentication value
5. **C2:** Custom HTTP + raw TCP for bulk data. Authentication via `ustart` mechanism distinguishing Snake from legitimate web traffic

**Capabilities:**
- Full keylogging
- File exfiltration (any file, any size)
- Screen capture
- Remote code execution
- Use victim as relay hop (pivot through their network undetected)
- Survives OS reinstall via kernel persistence

**Platforms:** Windows (kernel module), Linux variants confirmed

**Black market:** NOT available. FSB Center 16 government weapon. One of the most protected Russian tools. Never leaked.

### OLYMPIC DESTROYER (2018 Winter Olympics)

**Target:** PyeongChang Winter Olympics opening ceremony — destroyed networks, ticket systems, broadcaster systems, Wi-Fi.

**How it works:**
1. **Dropper** with two required arguments
2. **Credential stealing:** Browser stealer (IE/Firefox/Chrome — parses registry + SQLite DB). LSASS dump (Mimikatz, 18.5% code overlap with APT3/China tools — false flag)
3. **32KB credential patch** at offset `0x26F1A-0x2EF1A` — 44 unique credentials observed
4. **Lateral movement:** PsExec (admin shares) + WMI + EternalBlue/EternalRomance fallback
5. **Destruction:** MBR overwrite → VBR wipe → file corruption → Windows recovery disabled
6. **False flags:** Korean characters in credentials (misdirection). Code overlap with Chinese APT3 (intentional)

**Black market:** NOT available. GRU Sandworm only.

### CADDY WIPER (Ukraine 2022)

Deployed by Sandworm during Russian invasion of Ukraine. Fast multi-threaded Windows disk wiper. Overwrites critical system files + destroys partition tables. No ransomware component — pure destruction.

### ACIDRAIN (VIA Satellite Modem Wiper — KA-SAT 2022)

**Target:** Viasat KA-SAT satellite broadband modems — 28,000+ modems permanently bricked. Triggered simultaneously with Russian invasion of Ukraine.

**How it works (technical):**
1. **MIPS ELF binary** (32-bit, big-endian) for embedded Linux
2. **Runs as root:**
   - Recursively finds all files, overwrites with `0xFFFFFFFF` via 4-byte integers
   - Iterates `/dev/mtdblock0` through `/dev/mtdblock99` (flash storage)
   - Uses IOCTLs: `MEMGETINFO`, `MEMUNLOCK`, `MEMERASE`, `MEMWRITEOOB` — hardware flash operations
   - Calls `fsync` to force writes to commit
   - Brute-forces device filenames ("one-binary-fits-all" approach)
3. **Result:** Modem firmware permanently erased. Physical replacement required. 28,000+ modems bricked.
4. **Code similarity with VPNFilter:** Shared statically-linked libc, identical section headers, same IOCTL usage

**Black market:** NOT available. GRU Sandworm. Source code never leaked.

### CYCLOPS BLINK (Sandworm — 2019-2022)

**Target:** WatchGuard Firebox VPN/firewall appliances. 1,500+ devices in botnet cluster.

**How it works:**
1. **Linux ELF** (32-bit PowerPC) — core + 4 built-in modules (forked child processes, IPC via pipes)
2. **Masquerades** as kernel process `"kworker[0:1]"`
3. Modifies `iptables` to allow C2 on hardcoded ports
4. **C2:** TLS tunnel (OpenSSL 1.0.1f) + AES-256-CBC per-command encryption. Random C2 IP from embedded list. Managed via Tor.
5. **Persistence:** Deployed as firmware "update" — exploits weak HMAC validation in WatchGuard update process (hardcoded key). Adds files `WGUpgrade-dl`, `S51armled` to firmware image. Survives legitimate firmware updates — reinfects after reboot.
6. **Modules:** C2 server, file upload/download, system info (`/etc/passwd`, `/proc/mounts`, sysinfo), update/persistence

**What you can do:** Take over 1,500+ VPN/firewall appliances worldwide. Intercept all traffic passing through them. Use as botnet for further attacks on internal networks behind the firewalls.

**Black market:** Not available. GRU Sandworm. CISA/NSA joint advisory published May 2023.

---

## 10. COMMERCIAL SPYWARE — DEEP TECHNICAL

### PEGASUS — Full Technical Breakdown

**C2 Architecture:**
- Web-based management console (operators manage targets, deploy exploits, retrieve data)
- Implant ↔ C2 via HTTPS (TLS with certificate pinning)
- **Exfiltration channels:** HTTPS POST, encrypted cloud upload (Dropbox/Google Drive), DNS tunneling (fallback), SMS commands

**FORCEDENTRY technical (CVE-2021-30860) — The breakthrough zero-click:**

1. Attacker sends iMessage with malicious PDF/image attachment to target
2. Apple's `BlastDoor` sandbox processes the attachment for iMessage
3. Vulnerability in `JBIG2Stream::readTextRegionSeg` in CoreGraphics
4. **The genius:** JBIG2 is a bitmap compression standard — NO scripting language natively. But FORCEDENTRY uses **70,000+ JBIG2 segment commands** to emulate logic gates → builds a custom CPU architecture with registers, 64-bit adder, and comparator inside the JBIG2 parser
5. This emulated computer searches memory, performs arithmetic calculations → finds kernel memory
6. Disables ASLR, bypasses PAC (Pointer Authentication Code) — Apple's strongest security
7. Escapes BlastDoor sandbox → installs Pegasus with root privileges
8. **No user interaction — target just receives an iMessage**

**BLASTPAST (2023 — iOS 16.6):**
- Two-stage: Apple HomeKit (`homed` crash via NSKeyedUnarchiver) + Apple Wallet `.pkpass` with embedded WebP image (CVE-2023-4863)
- WebP contains NSExpression primitives decoding Base64-embedded encrypted exploit blobs
- Still no user interaction required

**Pegasus Capabilities (complete list):**
- Full phone takeover (not just partial access)
- Call recording (record both sides of any call)
- Ambient microphone activation (record room silently)
- Front + rear camera activation
- Real-time GPS tracking + history
- All messaging apps: WhatsApp, Signal, Telegram, iMessage, Viber, Facebook Messenger, WeChat
- Keychain extraction (iOS) — all saved passwords
- Cloud tokens (iCloud, Google) — access cloud backups
- Extract data from encrypted apps
- Kernel-level persistence (later versions survive reboot)

**Detection indicators:**
- `com.apple.xpc.roleaccountd.staging/natgd` process (Pegasus process on iOS)
- `passd` running with `imagent` as parent
- Anomalous `Celestial.framework`, `Media.framework`, `CoreLocation.framework` usage

**Black market:** NOT directly available. Only through NSO Group contracts ($500K-$8M+). In 2018, an NSO employee attempted to sell Pegasus for $50M on black market — was caught and arrested by the company.

---
### PREDATOR (Intellexa) — ALIEN Bootkit Technical

**Delivery chain (CDN Poisoning):**
1. Compromised CDN edge node (Akamai/CloudFront) intercepts target's app update request
2. Serves malicious update containing ALIEN injector instead of legitimate binary
3. No user interaction for the interception

**ALIEN Bootkit persistence:**
1. Injected into `zygote64` (Android's core process — parent of all Android apps) via `ptrace` attach
2. Downloads remaining components from installation server
3. Creates **shared memory file descriptors** for IPC between ALIEN and PREDATOR — invisible to SELinux
4. Communicates via `ioctl` commands (no access violations → SELinux never triggered)
5. Original Android: did NOT survive reboot. Updated (April 2022): reboot persistence added

**PREDATOR spware technical:**
- `pyfrozen` ELF file with serialized Python modules + native code
- Native: `tcore` (main module) + `kmem` (privilege escalation)
- Python modules: audio recording, TLS certificate poisoning (MITM), app hiding, app execution prevention, directory enumeration
- Communication via shared memory file descriptors (invisible to Android security)

**Black market:** EUR 8-13.6M per deployment via Intellexa. NOT available on open market.

---
### FINSPY / FINFISHER — UEFI Bootkit

**Discovered 2021. FinFisher GmbH (Gamma Group). Government/Law enforcement only.**

**UEFI persistence chain:**
1. Replaces `bootmgfw.efi` (Windows Boot Manager) with malicious version
2. Original boot manager renamed and stored in `efi\microsoft\boot\en-us\` with hex-character name
3. EFI partition contains two encrypted files:
   - Winlogon Injector (RC4 encrypted, key = EFI system partition GUID)
   - Trojan Loader (RC4 encrypted, key = EFI system partition GUID)
4. Malicious boot manager loads original Windows, but **patches kernel transition function**
5. Patched function hooks `PsCreateSystemThread` → creates a thread decrypting the next stage
6. Next stage locates and decrypts Trojan Loader from EFI partition
7. Trojan Loader waits for user login → injects into `winlogon.exe`
8. Final: extracts FinSpy DLL from resources, decrypts (XOR + aPLib), reflectively loads in-memory

**Pre-validator/Post-validator system:**
- **Pre-validator:** Runs security checks before deploying full payload (avoid sandbox/researcher analysis)
- **Post-validator:** Server-side check — verifies target is actually the intended victim
- Only then deploys full FinSpy trojan

**Obfuscation:**
- 4-layer custom obfuscation (OLLVM-like + custom protectors)
- 4,000 random bytes prepended to payload
- Unique per-machine encryption keys
- 5MB+ loader files to hinder analysis

**Platforms:** Windows, macOS, Linux, iOS, Android

**Black market:** Licensed to government/Law enforcement only ($130K-$500K per license). NOT available on underground markets.

---

## 11. BLACK MARKET ECOSYSTEM — DEEP DIVE

### RAMP Forum (Russian Ransomware & Advanced Malware Protection)

**Access:**
- Must be active member on XSS or Exploit forums (2+ months, 10+ posts, good reputation)
- OR pay $500 anonymous registration fee
- 14,000+ registered members
- Invite-only for high-trust areas
- Admin claims ~$250K annual revenue

**Listing Categories (leaked DB Nov 2021-Jan 2024, 333 IAB threads):**

| Category | Listings | Typical Items | Price Range |
|----------|----------|---------------|-------------|
| **Initial Access Sales** | 333 | VPN/RDP/SSH access to companies (56.4% Domain User, 33.9% Domain Admin) | $500-$30,000 |
| **Exploits & 0-days** | 22 | SonicWall VPN RCE ($100K), Office 0-day, WinRAR RCE | $30K-$100K |
| **Crypters / FUD** | 17 | Private Crypt ($2K/month), Luxury Shield | $70-$2K/month |
| **Ransomware kits** | 12 | Kakia v2, Thanos, ESXi variants | $500-$10,000 |
| **RATs & Backdoors** | 11 | Windows backdoors, Android banking trojans | $45-$500 |
| **Stealers** | 10 | LummaC2, Mars Stealer, Meduza, OSKI | $100-$1K |
| **Botnets** | 7 | Crypto-stealing botnet ($25K), GoldBrute RDP scanner | $1K-$25K |
| **Loaders** | 6 | AresLoader, BazarLoader | $200-$500 |

**Top Initial Access Brokers on RAMP:**
| Alias | Listings | Specialty |
|-------|----------|-----------|
| **inthematrix** | 41 | Corporate RDP — US, EU, Asia, Government |
| **boxi** | 23 | VNC/SSH access + data dumps |
| **el84** | 18 | Brazil, Israel, Telecom |
| **RobinHood** | 12 | Access broker + RaaS operator "KUIPER" |
| **jacksparrow** | 11 | US/EU RDP access |

**Pricing breakdown (2025-2026):**
- Average IAB price: ~$6,400 (up 88-103% from 2023)
- Commodity RDP: $500-$1,500
- Domain admin (financial sector, $400M+ revenue): $5,000+
- Domain admin (healthcare, US): upwards of $20,000
- Highest observed: $30,000 buy-it-now
- Buyer-seller revenue-sharing models becoming common

---
### Exploit.in Forum (Longest-running Russian Exploit Market)

**Format:** Auction-style bidding for premium access
- Average initial price (2024): $2,812 (up 88% from 2023)
- Average blitz/buy-it-now: $5,784 (up 103%)
- Highest observed: $25,000 initial / $30,000 blitz
- Required reputation to post
- Categories: exploits, 0-days, access, malware, crypto, tutorials
- **Declining in 2025-2026** — older users aging out, RAMP taking over

---
### Telegram Private Channels (Current dominant platform)

Major migration from Tor forums to Telegram — lower barrier, faster, ephemeral, harder to police.

**Channel types and pricing:**
| Channel Type | Access | Price | Description |
|-------------|--------|-------|-------------|
| VIP stealer logs | Invite-only | $200-400/month XMR | Fresh, untouched credentials from RedLine/Vidar/Raccoon logs |
| 0-day exploit channels | Invite-only, verify via escrowed sample | $50K-$4M per exploit | Browser, mobile, chat app exploits |
| MaaS channels | Invite-only | $500-$5,000/month | HVNC RAT suites ($5K/mo), EDR killers ($1-3K/mo) |
| DDoS botnet rental | Public | $50-$500/day | Layer 4/7 attacks |
| Ransomware affiliate recruitment | Invite-only | Commission split | 70-90% to affiliate |

**Zero-day pricing on Telegram private channels (current 2026):**
| Target | Type | Price Range |
|--------|------|-------------|
| Chrome RCE | One-click | $80K-$200K |
| Outlook RCE | One-click | $100K-$300K |
| Windows LPE | Local | $50K-$150K |
| WhatsApp RCE | Zero-click | $1M-$3M |
| Telegram RCE | Zero-click | $500K-$1.5M |
| Telegram full chain | Zero-click → OS | $4M |
| iOS kernel exploit | Remote | $1M-$2.5M |
| Android kernel exploit | Remote | $500K-$1.5M |

**How to verify Telegram seller:**
1. **Vouch system:** Seller provides past anonymous buyers who vouch for them
2. **Escrowed sample:** Buyer deposits 4% escrow fee → seller provides limited PoC → buyer verifies → full payment → complete exploit
3. **Reputation tracking:** Cross-reference handles across Exploit/XSS/RAMP forums
4. **Red flags:** New accounts, no vouches, demands full payment upfront, refuses escrow

---
### Dark Web Marketplace Dynamics (2026)

| Marketplace | Status | Focus | IAB Price Range | Notes |
|-------------|--------|-------|-----------------|-------|
| **RAMP** | Active | RaaS + IAB | $500-$30K | Russian, highest quality, tough entry |
| **Exploit.in** | Active (declining) | Exploits + IAB | $500-$25K | Oldest, aging user base |
| **XSS.is** | Down (frequent) | IAB + Malware | $500-$5K | Frequent law enforcement seizures |
| **DarkForums** | Active (rising) | Hybrid — IAB + fraud | $500-$15K | English-language, more open registration |
| **BreachForums** | Seized/rebooted multiple times | Breach data | $500-$10K | Repeatedly disrupted by FBI |
| **Telegram** | Active (growing) | All categories | $200-$4M | Hardest to police, fastest growing |

### Payment Methods

| Method | Use | Notes |
|--------|-----|-------|
| **Bitcoin (BTC)** | Standard | 5-30 min confirmation, traceable with blockchain analysis |
| **Monero (XMR)** | Preferred | Mandatory on many forums for privacy. Untraceable |
| **USDT (TRC20)** | Growing | Stable value, fast settlement |
| **Escrow** | Required on RAMP/Exploit | 4% fee on RAMP. Platform holds payment until delivery confirmed |
| **Direct payment** | Established vendors only | Only after extensive vouch history |

### Summary: Tool Availability Matrix

| Tool / Category | Where to Get It | Price | Can You Buy It? |
|----------------|----------------|-------|-----------------|
| NSA Equation Group tools | Leaked (Shadow Brokers/GitHub) | FREE | Yes — leaked publicly |
| CIA Vault 7 tools | Leaked (WikiLeaks/GitHub) | FREE | Yes — leaked publicly |
| Pegasus | NSO Group contract only | $500K-$8M+ | No — gov license only |
| Predator | Intellexa contract only | EUR 8-13.6M | No — gov license only |
| FinFisher/FinSpy | Gamma Group license | $130K-$500K | No — LE license only |
| EternalBlue/DOUBLEPULSAR | GitHub (Metasploit modules) | FREE | Yes — integrated into tools |
| SUNBURST | Never leaked | N/A | No — SVR only |
| Shamoon | Never leaked | N/A | No — Iran only |
| Snake (Turla) | Never leaked | N/A | No — FSB only |
| 0-day exploits (browser) | Telegram private channels | $80K-$4M | Yes — via brokers/IABs |
| 0-day exploits (mobile) | Crowdfense/Operation Zero | $2.5M-$20M | Yes — if you're a government |
| Initial Access (VPN/RDP) | RAMP/Exploit/DarkForums | $500-$30K | Yes — anyone with crypto |
| Ransomware kits | RAMP/Telegram | $500-$10K | Yes — includes builder + panel |
| Stealer logs | Telegram/forums | $200-400/month | Yes — ongoing subscription |
| Crypters/FUD | Telegram/forums | $70-$2K/month | Yes — subscription |
| Cobalt Strike (cracked) | RAMP | $5K | Yes — cracked version |
| Botnet rental | Telegram | $50-$500/day | Yes — pay per use |

### Major Players

| Vendor | Country | Product | Cost | Status |
|--------|---------|---------|------|--------|
| **NSO Group** | Israel | Pegasus | $500K setup + $650K/10 targets | US Entity List |
| **Intellexa/Cytrox** | Greece/Israel | Predator | EUR 8-13.6M/deployment | US-sanctioned |
| **Candiru** | Israel | DevilsTongue | Unknown | US Entity List |
| **QuaDream** | Israel | REIGN | Unknown | Shut down Apr 2023 |
| **Paragon** | Israel | Graphite | Unknown | Active |
| **Variston IT** | Spain | Heliconia | Unknown | Active |
| **RCS Lab** | Italy | Hermit | Unknown | Active |

### PREDATOR (Intellexa/Cytrox)

| Attribute | Detail |
|-----------|--------|
| **Developer** | Intellexa / Cytrox (Greece) |
| **Delivery** | CDN Poisoning — compromised Akamai/CloudFront edge nodes |
| **Persistence** | ALIEN bootkit — modifies Android initramfs, survives factory reset |
| **Cost** | EUR 8-13.6M per deployment |
| **Sanctions** | US Entity List (Jul 2023), OFAC sanctions (Mar + Sep 2024) |
| **Status** | Continues operating despite sanctions |

How CDN Poisoning works: Compromised Content Delivery Network edge nodes intercept and modify delivery of legitimate software updates. When a target's phone requests an update from a CDN that serves apps, the compromised node serves a malicious update instead — no user interaction needed.

### GRAPHITE (Paragon Solutions)

Modern spyware platform. In January 2025, Paragon used a WhatsApp zero-click campaign targeting ~90 journalist accounts. The company claimed to have ethical frameworks but continued operations after exposure.

### DEVILSTONGUE (Candiru)

Israeli spyware firm. Uses DNS-over-HTTPS (DoH) exfiltration to avoid network monitoring. Server-side watering holes on compromised Apache/Nginx servers deliver zero-click browser exploits. Chrome zero-days sold to governments without CVE disclosure.

---

## 12. THE ZERO-DAY MARKET ECONOMY

### Market Structure

The zero-day exploit market operates across three tiers:

| Tier | Type | Buyers | Example | Price Range |
|------|------|--------|---------|-------------|
| **White** | Legitimate bug bounty | Vendors (Apple, Google, MS) | ZDI, HackerOne | $500-$1M |
| **Gray** | Broker-mediated | Government agencies | Crowdfense, Zerodium | $1M-$20M |
| **Black** | Criminal underground | Cybercriminals, APTs | Dark web forums | $30K-$100K+ |

### Current Price List (2025-2026)

| Exploit Type | Crowdfense | Zerodium (inactive) | Operation Zero |
|-------------|-----------|---------------------|----------------|
| iOS Zero-Click Full Chain | $5-7M | $2.5M | $2.5-20M |
| Android Zero-Click Full Chain | $5M | $2.5M | $2.5-20M |
| WhatsApp/iMessage RCE | $3-5M | $1.5M | Up to $4M (Telegram) |
| Chrome RCE + Sandbox Escape | $500K-$3.5M | $500K | N/A |
| Safari RCE + Sandbox Escape | $1M-3.5M | $400K | N/A |
| Windows LPE | N/A | $80K | N/A |
| Linux LPE | N/A | $100K | N/A |
| macOS LPE | N/A | $100K | N/A |

### Key Brokers

#### Crowdfense (Dubai)

| Attribute | Detail |
|-----------|--------|
| **Founded** | 2017 |
| **2024 Budget** | $30M for exploit acquisition |
| **Top Payout** | $7M (iOS zero-click) |
| **Clients** | Government agencies (vetted) |
| **Status** | Active, publishes public price list |

#### Zerodium (Washington DC)

| Attribute | Detail |
|-----------|--------|
| **Founded** | 2015 by Chaouki Bekrar |
| **Total Paid** | $50M+ to researchers |
| **Clients** | NATO government agencies |
| **Status** | Inactive (2025) — website disabled, social media deleted |

#### Operation Zero (St. Petersburg, Russia)

| Attribute | Detail |
|-----------|--------|
| **Founded** | 2021 |
| **CEO** | Sergey Zelenyuk |
| **Clients** | Russian government and organizations only |
| **Policy** | "End user is a non-NATO country" |
| **2024 Peak** | $20M for iOS/Android zero-click chains |
| **Status** | US-sanctioned (Feb 2026) — OFAC designation |

### Market Dynamics

- **44% annual price inflation** — a $1M exploit in 2015 costs $7M+ today
- **Zero-click premium** — one-click exploits worth 60-80% less
- **Shelf life collapse** — high-value RCEs under 90 days median (2025)
- **Vendor competition** — Bug bounties increasing (Apple top: $2M) but always below brokers
- **US VEP retention** — NSA retained 42% of vulnerabilities found (2024)

---

## 8. BLACK MARKET ACQUISITION

### How to Acquire These Tools

| Method | Risk | Cost | Description |
|--------|------|------|-------------|
| **Exploit Brokers** | Low (gray) | $1M-7M | Crowdfense: submit via VRH portal. Requires vetted buyer |
| **Operation Zero** | Sanctioned | $2.5M-20M | opzero.ru. Russian entities only |
| **Initial Access Brokers** | Medium | $500-$3K | XSS.is, Exploit.in, RAMP. VPN/SSH/RDP access |
| **Dark Web Forums** | High | $500-$100K | Requires vouches. Escrow 4%. Crypto (BTC/XMR) |
| **Telegram Channels** | Med-High | $1K-$25K | Private invite-only. Crypters, loaders, botnets |
| **Leaked Tools** | Medium | Free | Shadow Brokers (Github), Vault 7 (WikiLeaks) |
| **Insider Theft** | EXTREME | 87 months prison | Peter Williams case — L3Harris → Operation Zero |

### Dark Web Marketplaces

| Marketplace | Focus | Notable Items | Price Range |
|-------------|-------|---------------|-------------|
| **RAMP** | Russian RaaS | LockBit, Cobalt Strike, Conti | $1K-$100K |
| **Exploit.in** | Exploits | VPN RCE, Office 0-day, WinRAR RCE | $30K-$100K |
| **XSS.is** | Russian | Initial access, RATs, stealers | $500-$5K |
| **DarkForums** | Hybrid | Breach data, IAB | $500-$15K |
| **Telegram** | Private | C2 panels, FUD crypters ($2K/mo) | $70-$25K |

### Case Study: Peter Williams (L3Harris Insider Theft)

| Detail | Information |
|--------|-------------|
| **Name** | Peter Williams (39, Australian) |
| **Position** | General Manager, Trenchant (L3Harris subsidiary) |
| **Duration** | 2022-2025 |
| **Stolen** | 8 zero-day exploits built for US government |
| **Sold To** | Operation Zero (Russian broker) |
| **Payment** | $1.3M in cryptocurrency |
| **Sentenced** | 87 months federal prison |
| **Damages** | $35M estimated by DOJ |

---

## 9. MARKET DYNAMICS & TRENDS

### The Cyber Arms Race in Numbers

- **$30M** — Crowdfense's 2024 exploit acquisition budget
- **$7M** — Top payout for iOS zero-click chain
- **44%** — Annual zero-day price inflation rate
- **90 days** — Median exploit shelf life for high-value RCEs (2025)
- **42%** — NSA VEP retention rate (2024)
- **75** — Zero-days exploited in the wild (2024)
- **200,000+** — Systems infected by DOUBLEPULSAR in 2 weeks
- **$10B** — NotPetya's total damages
- **$8M** — Cost to deploy Pegasus in Ghana (2015-2016)
- **58%** — NSA disclosure rate through VEP (2024)

### The New Generation

Between November 2024 and April 2026, Citizen Lab identified spyware infections on 1,847 devices in 34 countries. Unlike Pegasus (sold directly by NSO), this new generation is distributed through fragmented resellers, brokers, and shell entities registered in Cyprus, BVI, and UAE. Governments now source through local consulting firms → offshore brokers → anonymous developers. Payment is structured through invoices for "software licensing" and "technical support."

---

## 10. SOURCES

- [WikiLeaks Vault 7](https://wikileaks.org/vault7/) — CIA cyber weapons documents
- [Citizen Lab](https://citizenlab.ca/) — Pegasus/FORCEDENTRY research
- [Kaspersky Equation Group Reports](https://kaspersky.com/) — Original GRAYFISH/EQUATION discovery
- [Crowdfense Exploit Acquisition Program](https://www.crowdfense.com/exploit-acquisition-program/) — Public pricing
- [Operation Zero](https://opzero.ru/en/) — Russian broker platform
- [Shadow Brokers Leaks](https://github.com/x0rz/EQGRP) — Equation Group tools
- [US Treasury OFAC](https://ofac.treasury.gov/) — Sanctions on Operation Zero
- [TechCrunch](https://techcrunch.com/) — Zero-day market reporting
- [SecurityWeek](https://www.securityweek.com/) — Exploit market analysis
- [BleepingComputer](https://www.bleepingcomputer.com/) — Malware analysis
- [Rapid7](https://www.rapid7.com/) — DOUBLEPULSAR analysis
- [Check Point Research](https://research.checkpoint.com/) — DanderSpritz deep-dive
- [Symantec/Carbon Black](https://www.security.com/) — Buckeye/Equation Group usage
- [Networkcraft](https://networkcraft.net/) — Zero-day market 2026 report
- [ForkLog](https://forklog.com/) — Zero-day economics

---

> **Disclaimer:** This document is an educational intelligence reference compiled from publicly available sources, leaks, court documents, and security research publications. The information is provided for academic understanding of the cybersecurity threat landscape. All tools, prices, and capabilities are derived from open-source intelligence (OSINT).
