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

## 🔗 Sample Download (For Research Use Only)

📦 **Download (Mega.nz)**:  
[https://mega.nz/file/Fg5h2TbB#f07IwoptVhl2rfrRvUJ0iO5A1vqPTmg1ZhYKawOnkxU](https://mega.nz/file/Fg5h2TbB#f07IwoptVhl2rfrRvUJ0iO5A1vqPTmg1ZhYKawOnkxU)  
🔐 **Password**: `infected`

## ⚠️ Disclaimer

This sample is provided **strictly for research and educational purposes**.  
Do not execute the binaries. Use in sandboxes or analysis environments only.

The behavior and operational signature **strongly resembles tools disclosed in the 2017 Vault 7 leak**, associated with a certain U.S. intelligence agency known for global surveillance operations.

Whether this sample is a fork, evolution, or unreported sibling remains unclear — but its stealth, impersonation, and delivery method are consistent with state-grade implants historically linked to **Langley-based interests**.

## 📩 Contact

If you're a researcher, analyst, or journalist interested in further context or forensic logs, feel free to open an issue or email anonymously.

---

