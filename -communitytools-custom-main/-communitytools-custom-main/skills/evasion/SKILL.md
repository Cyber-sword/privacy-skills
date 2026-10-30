---
name: evasion
description: AV/EDR evasion and detection bypass — AMSI, ETW, API unhooking, process injection, AppLocker bypass, LOLBins, shellcode obfuscation.
---

# Evasion

Bypass defensive controls to execute payloads and maintain access without triggering AV/EDR/SIEM.

## Techniques

| Category | Vectors |
|----------|---------|
| **Script-level bypass** | PowerShell execution policy, AMSI patch, ScriptBlock log disable |
| **Telemetry blind spots** | ETW patch, API unhooking (Hell's Gate), direct syscalls |
| **Execution without drop** | CLR injection, reflective loading, execute-assembly |
| **Process trust abuse** | PPID spoofing, process hollowing, APC injection |
| **Policy bypass** | AppLocker via LOLBins, regsvr32/mshta/InstallUtil/MSBuild |
| **Payload concealment** | XOR/AES/RC4 shellcode, DLL sideloading, DLL hijacking |

## Workflow

1. Identify defensive stack (AV vendor, EDR, PowerShell logging, AppLocker policy)
2. Select bypass matching the control — AMSI before PS execution, ETW before in-memory tools
3. Inject/execute payload without touching disk where possible
4. Verify evasion: test against defender logs, not just AV scan

## Reference

- `reference/amsi-etw-bypass.md` — AMSI patch, ETW blind, API unhooking, PS logging disable
- `reference/process-injection.md` — DLL injection, hollowing, APC, CLR/execute-assembly, PPID spoof
- `reference/lolbins-applocker.md` — AppLocker bypass, LOLBins, DLL hijacking/sideloading
- `reference/shellcode-obfuscation.md` — XOR/AES/RC4 encryption, polymorphic encoding
