# NATION-STATE CYBER WEAPONS ARSENAL

> **Live Web Page:** [`https://[your-github-username].github.io/nationstate-cyber-tools`](https://[your-github-username].github.io/nationstate-cyber-tools)

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

## 6. COMMERCIAL SPYWARE VENDORS

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

## 7. THE ZERO-DAY MARKET ECONOMY

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
