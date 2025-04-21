# voldemort-cisco-implant
In-the-wild malware sample masquerading as Cisco Webex – April 2025
## 🧠 Overview

- Masquerades as: `CiscoCollabHost.exe`, `CiscoSparkLauncher.dll`
- Installation: `AppData\Local\CiscoSparkLauncher`
- Persistence: Scheduled Task (user-level)
- Behavior: Likely DLL sideloading, stealth backdoor tactics
- Suspected C2: Abuses cloud infrastructure (Google Sheets, Cloudflare IPs)
- File size: ~600MB — possibly sandbox evasion by design

## 🛠️ Observed Files

- `CiscoCollabHost.exx`
- `CiscoSparkLauncher.dl_`
- Multiple log/config files

All files are renamed to prevent accidental execution.

## 🧬 Execution Chain: From File to Fileless

While the sample initially appears as a standard (albeit shady) `.exe`, deeper forensic analysis reveals a sophisticated infection chain designed to transition from a visible loader to a stealth, memory-only implant.

### What we observed:

1. **Initial Loader (CiscoCollabHost.exe)**  
   - Dropped in: `AppData\Local\CiscoSparkLauncher\`  
   - Size: ~600MB  
   - Masquerades as Cisco Webex component  
   - Registers a Scheduled Task under user context for persistence  
   - Triggers no immediate AV alert

2. **Payload Injection into `services.exe`**  
   - Upon execution, the loader injects code into the trusted Windows process `services.exe`  
   - No child processes visible  
   - Memory threads begin spawning rogue `svchost.exe` instances under user (`snake`) rather than SYSTEM

3. **Memory-Resident Behavior**  
   - Injected processes (`svchost.exe`) had:  
     - No command line  
     - No file path  
     - Outbound traffic to suspicious cloud-hosted C2s  
   - Attempts to terminate the processes resulted in immediate respawn  
   - Indicates presence of **injected watchdog logic** or persistence via `services.exe`'s memory space

4. **Trigger-Based Detection**  
   - Windows Defender did not flag the sample during initial execution  
   - Only upon **renaming** or **tracing execution behavior** did Defender detect and label it as:  
     - `Trojan:Win32/Bearfoos.B!ml`  
     - `Trojan:Script/Wacatac.C!ml`

This behavior matches advanced APT-grade implants:
- Use a **clean loader** to pass static scans
- Escalate to **memory-only persistence**
- Avoid common IOCs like registry keys, startup folders, or visible file paths

> In short: This thing starts as a file, lives in memory, and **doesn’t want to be found again**.

🔒 If you’re seeing rogue `svchost.exe` processes under your own user — you may already be hosting a guest.

## 📦 Sample Files

All renamed binaries are available directly in this repo under `artifacts/`.  
Do NOT run these. They are for research and analysis only.

- `CiscoCollabHost.exx`
- `CiscoSparkLauncher.dl_`
- Log files

## ⚠️ Disclaimer

This sample is provided **strictly for research and educational purposes**.  
Do not execute the binaries. Use in sandboxes or analysis environments only.

The behavior and operational signature **strongly resembles tools disclosed in the 2017 Vault 7 leak**, associated with a certain U.S. intelligence agency known for global surveillance operations.

Whether this sample is a fork, evolution, or unreported sibling remains unclear — but its stealth, impersonation, and delivery method are consistent with state-grade implants historically linked to **Langley-based interests**.

## 📩 Contact

If you're a researcher, analyst, or journalist interested in further context or forensic logs, feel free to open an issue or email anonymously.

---

