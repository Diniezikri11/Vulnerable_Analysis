<img width="940" height="594" alt="image" src="https://github.com/user-attachments/assets/64f43b65-5c92-412f-a5b5-c342c8a3d88e" />

## 🛠️ Question 1: Analyzing `packet1.pcap` to Extract the Hidden Flag

### 1. Protocol Filtering
* **Action:** Open `packet1.pcap` inside the Wireshark interface and input the following token into the green display filter bar:

`icmp`

* **Purpose:** This isolates all Internet Control Message Protocol (ICMP) echo requests and responses (ping queries), filtering out unrelated background data transmissions.

### 2. Anomaly Detection & Identification
While reviewing the filtered ICMP frame list, a severe technical anomaly stands out at **Packet 37**:
* **Frame Length Variance:** Every standard ping request/reply package (Packets 27–36, 38) carries a precise length of 98 bytes. Packet 37 drops down sharply to 70 bytes.
* **Header Configurations:** Baseline traffic increments smoothly via specific tracking identifiers (e.g., id=0x7a93, sequence counters near 18/4608). Packet 37 completely resets its sequence metadata, presenting a static signature of id=0x0001 with a sequence counter configuration of seq=0/0.
* **TTL Flag Alteration:** The Time-to-Live (TTL) structural signature shifts from the standard baseline value of ttl=64 down to ttl=63.*
* 
*<img width="940" height="416" alt="image" src="https://github.com/user-attachments/assets/14878dd8-7d86-4ae0-8115-f653c599f065" />

*   ### 3. Data Extraction (Hex & ASCII Dump Window)
By highlighting Packet 37 and shifting focus to the bottom-right **Packet Bytes Pane**, we can audit the payload's raw binary signature. Standard ping frames use standard character strings as diagnostic padding. 

In Packet 37, this padding is stripped out and replaced with a base-64 text block. Reading the characters vertically downward across memory offsets 0020, 0030, and 0040 yields the complete string vector:

`U1VDVEYyMDIze2FpX2lzX2Nvb2x9`

### 4. Decoding via CyberChef
* **Cryptographic Scheme:** Base64 (Alphanumeric uppercase, lowercase, integers, and length divisible by 4).
* **Execution Details:**
  * **Input String:** `U1VDVEYyMDIze2FpX2lzX2Nvb2x9`
  * **Applied Recipe:** From Base64
  * **Decrypted Output:** `SUCTF2023{ai_is_cool}`

### 🏁 Captured Flag

`SUCTF2023{ai_is_cool}`

---

<img width="940" height="647" alt="image" src="https://github.com/user-attachments/assets/cf213c47-9196-4380-a4b2-8c128a9007bb" />


## 🔒 Question 2: Analyzing `packet2.pcap` to Extract the Hidden Flag

### 1. File Inspection & Application Filtering
* **Action:** Load `packet2.pcap` into Wireshark and replace the filter string with:

`ftp`

* **Purpose:** Filters out ambient noise to explicitly track File Transfer Protocol commands and responses traversing the network.

### 2. Hunting the Command Stream
Auditing the FTP command responses reveals a client probing filesystem architectures. At **Packet 205**, an explicit file request tracking handle is flagged:

`Request: SIZE global_thermonuclear_war.gamerules.txt`

<img width="940" height="584" alt="image" src="https://github.com/user-attachments/assets/ede68245-fdd2-4640-8eea-fa97e9ebb8e3" />

### 3. Reconstructing the Transmission (TCP Stream Follow)
Because FTP handles commands (Port 21) and file data payloads (Port 20 or dynamic high ports) over separate channels, we clear the FTP filter to display the passive raw socket handshakes.
1. Clear the filter bar and press Enter.
2. Locate the sequential grey TCP transactions running right after the file inquiry (starting around **Packet 211**).
3. **Right-Click** the data packet -> Select **Follow** -> Click **TCP Stream**.

A dedicated plain text popup panel exposes the raw internal text configuration parameters of `global_thermonuclear_war.gamerules.txt`, revealing a hidden redirection link:

`https://tinyurl.com/yr5zprz4`

<img width="940" height="550" alt="image" src="https://github.com/user-attachments/assets/f20ae67e-8fa3-4319-9ba0-7a83a4ba8f19" />

### 4. Decoding the Pigpen Cipher
Navigating to the document URL reveals a series of geometric substitution symbols known historically as a **Pigpen Cipher** (or Masonic grid cipher). 

Mapping the shapes to the respective tic-tac-toe grids and X-frames containing dot parameters yields the following matching string characters


### 🏁 Captured Flag

`SUCTF2023{EXMACHINAAVA}`

## 🔍 Question 3: Interpreting Nmap Scan Results

### 3.1 Exploitation Capabilities by Port
* **Port 21/tcp (FTP - vsftpd 2.3.4):** Attackers can attempt anonymous path transversals, brute-force service authentication vectors, or target application backdoors to gain unauthorized file system access.
* **Port 22/tcp (SSH - OpenSSH 5.3p1):** Attackers can launch dictionaries to compromise root/administrator passwords or trace timing anomalies to harvest user account profiles.
* **Port 80/tcp (HTTP - Apache 2.2.8):** Attackers can script directory busters to find unindexed file paths or target structural injection flaws inside legacy web applications.
* **Port 139/tcp (NetBIOS-SSN):** Exposes corporate asset structural groupings, endpoint identities, and configuration records.
* **Port 445/tcp (SMB - Windows 7 Professional 7601 SP1):** Enables remote network exploitation to compromise memory parsing parameters or capture internal credential validation hashes.

### 3.2 Key Vulnerabilities Present
* **vsftpd 2.3.4 -> CVE-2011-2523:** Contains an upstream compromise backdoor. Injecting a smiley face sequence `:)` inside the username field opens an unauthenticated interactive root command shell on dynamic backup listening port 6200.
* **OpenSSH 5.3p1 -> CVE-2016-6210:** Susceptible to a timing side-channel flaw allowing active directory profile harvesting.
* **Apache 2.2.8 -> CVE-2007-6750:** Critically vulnerable to the **Slowloris DoS** attack vector, which hangs and crashes the service via lingering incomplete HTTP header requests.
* **Windows 7 SP1 (Port 445) -> CVE-2017-0144 (MS17-010):** Susceptible to the **EternalBlue** exploit module within the legacy SMBv1 protocol.

### 3.3 Highest Risk Assessment
**Port 445 running Windows 7 SP1 (MS17-010)** represents the ultimate security risk threat layer.

**Justification:** Unlike application-isolated flaws, EternalBlue targets the operating system's kernel stack layer. It requires **zero valid credentials/user interaction** and grants full execution privilege control (`NT AUTHORITY\SYSTEM`). It is a wormable vulnerability capable of automating lateral ransomware propagation.

### 3.4 Target Attack Path Scenario
1. **Initial Shell:** Attacker targets Port 445 using Metasploit (`exploit/windows/smb/ms17_010_eternalblue`) to inherit an active SYSTEM shell.
2. **Credential Harvesting:** Attacker extracts stored plain-text admin credentials from memory arrays inside the Local Security Authority Subsystem Service (`lsass.exe`) using post-exploitation tool sets.
3. **Lateral Movement:** Attacker re-uses the stolen administrative password vector to authenticate directly across Port 22 (SSH) on neighboring systems, compromising the network zone.

### 3.5 Remediation Ledger
* **Windows 7 Platform:** Deploy patch **MS17-010** immediately, disable **SMBv1** protocols, and upgrade the OS architecture to Windows 10/11 due to End-of-Life lifecycle status.
* **vsftpd 2.3.4 Service:** Reinstall pristine verified packages and shift to modern upstream stable binary deployments.
* **Apache & OpenSSH:** Execute administrative server configuration updates (`apt upgrade` / `yum update`) to transition both daemons to modern, patched LTS instances.
* **Network Perimeter:** Implement structural firewall Access Control Lists (ACLs) restricting access to administrative ports from external public segments.

---

## 🏷️ Question 4: Passive OS Fingerprinting Analysis (TTL)

Operating systems initialize network frames using standard distinct baseline values for **Time-To-Live (TTL)** parameters inside ICMP echo header metrics.

* **Image 1 Classification: Linux / Unix**
  * *Technical Signature:* Logs register a baseline metric of `ttl=64`, which matches default Linux network stacks.
* **Image 2 Classification: Cisco Infrastructure Node / Enterprise Unix (Solaris)**
  * *Technical Signature:* The Wireshark protocol parser window highlights a hard metric of `Time to live: 255`, indicating core routing/switching platforms.
* **Image 3 Classification: Microsoft Windows Server/Desktop**
  * *Technical Signature:* Logs record a signature metric of `ttl=128`, confirming standard Windows OS architecture configuration parameters.

---

## 🐱 Question 5: Apache Tomcat Ghostcat Vulnerability Analysis

Analyzing a standard Nessus parsing report centered on the **Ghostcat** bug yields the following indicators of compromise (IoC):

1. **Target Vulnerable Port:** `8009 / tcp`
2. **Target Network Protocol:** `AJP13` (Apache JServ Protocol version 1.3)
3. **Vulnerability Severity Metric:** CVSS v3.0 Base Rating Score: `9.8 (Critical)`
4. **Public Exploit Matrices:** * Standalone Automation Script: **Exploit-DB ID 48143** (Python File Inclusion script)
   * Metasploit Exploitation Framework Module: `auxiliary/admin/http/tomcat_ghostcat`
5. **Universal Security Reference Identifier:** `CVE-2020-1938`
