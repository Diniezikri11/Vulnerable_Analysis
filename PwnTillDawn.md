### LAB WEEK 3: PwnTillDawn 1st Box

1. open terminal and run `openvpn PwnTillDawn.ovpn`.
2. Scan the network in the range `10.150.150.10-254`.
3. Pick your desire ip address like `10.150.150.242`

**RECON:**

1. `nmap -F 10.150.150.242` to see available open ports.
2. common open port found: **53** (domain), **80** (http), **445** (smb)

<img width="1498" height="378" alt="image" src="https://github.com/user-attachments/assets/2764af9f-aef2-4cdf-aa8b-8ce055ddc951" />

**ENUMERATION:**

There are port 80 that open so go to browser and type the ip 10.150.150.242
<img width="1500" height="960" alt="image" src="https://github.com/user-attachments/assets/d5a360a0-1c3b-4516-96b0-cbae9fb593a4" />
2.Found vulnerable code  in the images ispect
<img width="1507" height="1070" alt="image" src="https://github.com/user-attachments/assets/3d7a1ad8-de78-4c04-9965-2ad4dc84d69f" />

**EXPLOIT**
1. use metasploit to exploit port 445
2. run msfconsole to open metasploit console
<img width="1497" height="447" alt="image" src="https://github.com/user-attachments/assets/83ad490a-8cd0-434e-910c-5eb3ff67d04f" />
I ran an Nmap scan with the smb-vuln-ms17-010 script against 10.150.150.242. The results confirmed the target is vulnerable to EternalBlue (CVE-2017-0143). This vulnerability allows for unauthenticated Remote Code Execution because the target is running an unpatched version of SMBv1.

<img width="1024" height="453" alt="image" src="https://github.com/user-attachments/assets/9834e0a3-7d54-4c17-b9c8-6806c14e43f9" />

After confirming the MS17-010 vulnerability with Nmap, I searched Metasploit for matching modules. I identified ms17_010_eternalblue and ms17_010_psexec as the primary exploits available to gain remote access to the target.

<img width="859" height="458" alt="image" src="https://github.com/user-attachments/assets/179412ae-58d5-4843-8bdf-294d2b6b17e9" />

use command to select the EternalBlue exploit module. This tells Metasploit that we want to target the specific MS17-010 vulnerability we found during the Nmap scan. Once selected, Metasploit automatically loads a "payload" (in this case, a Meterpreter shell), which is the tool that will give us control once we break in.

Next, we set the RHOST (Remote Host). This is the IP address of the victim's computer.
set the RPORT (Remote Port) to 445

We then set the LHOST (Local Host). This is the most critical part of a Reverse Shell.

Before launching the attack, we run the options command. This displays a summary table of all our settings. We do this to double-check


<img width="1500" height="706" alt="image" src="https://github.com/user-attachments/assets/0c2db208-f724-4da0-a23a-d7ab43884cce" />


After configuring the exploit settings, I executed the EternalBlue module, which successfully bypassed the target's security to establish a Meterpreter session with "NT AUTHORITY\SYSTEM" privileges on the Windows Server 2008 R2 machine.

<img width="1493" height="861" alt="image" src="https://github.com/user-attachments/assets/aff3d4fa-a0d7-4e60-9d12-602e6f05d2d5" />

navigate to Admin folder: cd C:\\Users\\Administrator.GNBUSCA-W054
navigate to Desktop to find the flag: cd ~/Dekstop.
list all the file in Dekstop: ls.
found 1 file name FLAG34.txt.
open the file: cat FLAG34.txt
