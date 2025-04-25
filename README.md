# voldemort-cisco-implant  
In-the-wild Vault 7–style multi-stage memory-resident implant leveraging fake Cisco components  
First observed: April 2025  

## Overview

- Masquerades as:  
  - `CiscoCollabHost.exe` (mid-stage injector)  
  - `CiscoSparkLauncher.dll` (possibly unused decoy or placeholder)

- Initial Infection Vector:  
  - `ai.exe` spawned from `WINWORD.EXE` (macro-enabled document)  
  - Likely responsible for dropping or injecting other components

- Installation Path: 
  - `AppData\Local\CiscoSparkLauncher\`

- Persistence Mechanisms:
  - CCXProcess.exe: Long-lived memory-resident watchdog or beacon  
  - Scheduled Task (user context) — neutralized early, no longer present  
  - Hijacked legitimate Windows services:  
    - `DoSvc`, `AppXSvc`, `WaaSMedicSvc`, `wscsvc`

- Behavior Summary: 
  - Mid-stage payload (`CiscoCollabHost.exe`) injects into `services.exe`  
  - Spawns multiple stealth `svchost.exe` clones:  
    - No binary path or command-line  
    - Protected Process Light (PPL) level  
    - Active outbound TLS traffic to public cloud infrastructure  
  - Fully evades Microsoft Defender at all stages  
  - Likely inspired by CIA Vault 7 (Hive-like structure, but no redundancy)

- Suspected C2 Infrastructure: 
  - Encrypted TLS beaconing over port 443  
  - Targets include CDN endpoints (e.g. Cloudflare, Azure, Google)  
  - Matches behavioral patterns of **Swindle/Honeycomb-style** proxy+C2 split

## Observed Files

- `CiscoCollabHost.exx`
- `CiscoSparkLauncher.dl_`
- Multiple log/config files

All files are renamed to prevent accidental execution.

### What We Observed (Vault 7–Style Architecture)

#### 1. Root Injector (`ai.exe`)
- Spawned by: `WINWORD.EXE` (macro document)
- Behavior:
  - One instance terminated but still shows up in memory scans
  - Another instance remains active
  - No visible file path or command-line
- Assessment:
  - Strong candidate as Stage 0 root injector
  - Likely dropped or injected the mid-stage components

---

#### 2. Mid-Stage Loader (`CiscoCollabHost.exe`)
- Location:`AppData\Local\CiscoSparkLauncher\`
- Size: ~600MB
- Behavior:
  - Masquerades as a Cisco Webex component
  - Was active during the svchost.exe spawning phase
  - Once killed, it did not return — not redundant
- Assessment:
  - Likely used to inject into `services.exe`
  - Or to trigger hijacked service persistence (e.g. DoSvc)

---

#### 3. Beacon/Watchdog (`CCXProcess.exe`)
- Status: Persistent and memory-resident
- Behavior:
  - Survives reboot
  - Does not re-launch `CiscoCollabHost.exe` or svchosts (as observed)
- Assessment:
  - Likely acts as a WMI-triggered beacon or monitoring agent
  - Matches behavior of Vault 7 Hive’s Beacon component

---

#### 4. Payload Injection (`services.exe` → `svchost.exe` Clones)
- Behavior:
  - `services.exe` (PID 800) spawns memory-only `svchost.exe` clones
  - These clones are bound to real services:
    - `DoSvc`, `AppXSvc`, `WaaSMedicSvc`, `wscsvc`
  - Traits of injected `svchost.exe` instances:
    - No command line
    - No binary path
    - 0–1 threads (abnormal for legitimate services)
    - TLS/443 outbound traffic to Azure/CDN IPs
    - PPL-protected: unkillable even from SYSTEM

---

#### 5. Undetected by Microsoft Defender
- No alerts triggered during any phase:
  - Scheduled Task registration
  - Execution or injection into `services.exe`
  - svchost.exe beaconing
- Implant remains invisible to Defender even under active monitoring

---

### Summary

This malware demonstrates:
- Modular staging
- Memory-resident payload
- No disk presence 
- Legitimate service hijack
- No fallback redundancy (unlike full Vault 7 Hive)

TTPs Match APT-Grade Implants:
- Uses a document-based loader (`ai.exe`)
- Deploys mid-stage agents (`CCXProcess.exe`, `CiscoCollabHost.exe`)
- Injects into `services.exe` to spawn untraceable `svchost.exe` implants
- Fully evades userland AV (e.g., Microsoft Defender)

> _"If you're seeing `svchost.exe` tied to `DoSvc` or `AppXSvc` with no path, no command-line, and live network activity — you're likely already hosting a guest."_ 


# Infection Chain Diagram (Vault 7–Inspired Architecture)

LEGEND:
[Component]            = Executable or process
(Behavior)             = Action observed or inferred
{Notes}                = Confidence level: Confirmed 🟢 / Likely 🟡 / Inferred ⚪

--------------------------------------------------------------
                        USER ACTION
--------------------------------------------------------------
        [WINWORD.EXE] 
               │
               ▼
        [ai.exe]  🟡
        (Spawned by Word macro) 
        (Remains active / appears post-exit)
        {Candidate Root Injector – may have started the chain}
               │
               ├────────────┬────────────┐
               ▼            ▼            ▼
     [CCXProcess.exe]  [CiscoCollabHost.exe]  [services.exe]
         🟡                  🟢                  🟢
    (Beacon / Watchdog) (Mid-stage injector) (Legit system process)
    (Survives reboot)   (No longer present)  (Used to spawn svchost)
    {Likely drops/injects}   {Previously launched implants}

               ▼
--------------------------------------------------------------
                   FINAL PAYLOAD STAGE
--------------------------------------------------------------
        [svchost.exe clones] 🟢
        (PIDs: 6016, 11720, 14640, 14684, etc.)
        (Memory-resident, no cmdline/path, PPL-protected)
        (Linked to: DoSvc, AppXSvc, WaaSMedicSvc, wscsvc)
        {Confirmed implant behavior – Vault 7–style}

               ▼
       Encrypted TLS/443 Communication 🟢
       (To CDN/Azure – suspected Swindle proxy layer)
               ▼
     [Remote C2 Server – "Honeycomb" equivalent] ⚪
     (Destination unknown, but behavior matches Vault 7)

--------------------------------------------------------------

✔ CCXProcess.exe → Likely persistent, watchdog-style role
✔ CiscoCollabHost.exe → One-time loader, not redundant
✔ ai.exe → Strong root injector candidate (spawned by WINWORD.EXE)
✔ svchost.exe clones → Final-stage memory-only implants
✔ No redundant fallback detected = not full Vault 7 Hive


## Sample Files

All renamed binaries are available directly in this repo under `artifacts/`.  
Do NOT run these. They are for research and analysis only.

- `CiscoCollabHost.exx`
- `CiscoSparkLauncher.dl_`
- Log files

## Vault 7 Parallels

| Capability            | Voldemort Implant                              | Vault 7 Reference                        |
|-----------------------|------------------------------------------------|------------------------------------------|
| Memory-only           | svchost.exe lives entirely in RAM              | Athena, Leona, HammerDrill               |
| Process injection     | Injects into services.exe                      | Common to most implants                  |
| Respawn logic         | CCXProcess.exe may act as watchdog             | Leona, RATscreamer, HIVE                 |
| No path / cmdline     | svchost.exe clones lack disk trace             | Athena, DerStarke                        |
| Fake identity         | Poses as Cisco and Adobe software              | HIVE (TLS cert spoofing, mimic domains)  |
| Cloud C2              | TLS to Azure/CDN IPs                           | HIVE, Pandemic                           |
| AV evasion            | Undetected by Microsoft Defender               | Core Vault 7 objective                   |
| Modular design        | ai.exe → CiscoCollabHost.exe → CCXProcess.exe | Athena, HIVE modular architecture        |
| Service hijack        | Persists via DoSvc, AppXSvc, WaaSMedicSvc      | Seen in HIVE, AnnaChapman                |

## Disclaimer

This sample is provided **strictly for research and educational purposes**.  
Do not execute the binaries. Use in sandboxes or analysis environments only.
This is work in progress and we welcome colaboration 

The behavior and operational signature **strongly resembles tools disclosed in the 2017 Vault 7 leak**.

## Contact

If you're a researcher, analyst, or journalist interested in further context or forensic logs, feel free to open an issue or email anonymously.

---

