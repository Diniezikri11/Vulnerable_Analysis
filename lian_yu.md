### Phase 1: Reconnaissance (Nmap)
"The initial Nmap scan revealed open ports 21, 22, 80, and 111, prompting a strategic enumeration of FTP and HTTP services to uncover the hidden credentials required for SSH access and final flag retrieval."
<img width="1227" height="775" alt="image" src="https://github.com/user-attachments/assets/ded345b9-1194-4000-a2f4-5ca5ade0b1b3" />

### Run Gobuster
"Gobuster revealed /island as the key directory (accessible) and /server-status as forbidden; the next step is to explore /island further with ffuf to uncover hidden files leading to your flag."
<img width="886" height="298" alt="image" src="https://github.com/user-attachments/assets/a1d32f00-2bb9-4c4d-9f5e-94e027c13019" />
<img width="768" height="536" alt="image" src="https://github.com/user-attachments/assets/cc6e926e-c5ab-42ac-a573-b5aca47fbebf" />


### Phase: Web Enumeration (Deep Directory Fuzzing)
You used ffuf on http://10.48.177.251/island/FUZZ with a wordlist to brute‑force hidden directories; the valid hits with status 200 revealed accessible paths inside /island, which must be explored next to extract credentials or clues leading to SSH login and the user.txt flag.
<img width="1241" height="680" alt="image" src="https://github.com/user-attachments/assets/262d2ee4-7dbf-4ae9-8a4e-9e215d4ead08" />
<img width="759" height="540" alt="image" src="https://github.com/user-attachments/assets/5370cf1a-0f07-4c21-a8cb-c2b9cb24e5c5" />



### Phase: Targeted Web Enumeration (File Fuzzing)
This step belongs to the Targeted Web Enumeration phase, where you fuzz specific file patterns (.ticket) inside /island/2100/ to reveal hidden files that lead toward the next credential or flag.
<img width="769" height="521" alt="image" src="https://github.com/user-attachments/assets/a215b544-80ac-488f-8f31-e516064f43fa" />

### Phase: Data Decoding (Credential Extraction)
This step belongs to the Data Decoding phase, where CyberChef is used to convert Base58‑encoded text into a usable password (!#th3h00d) for progressing toward SSH login or flag retrieval.
<img width="782" height="548" alt="image" src="https://github.com/user-attachments/assets/42644119-3a9f-43f4-b4c6-69394a5b9678" />
### File Retrieval Phase
In that step, you connected to the FTP server, listed available files, downloaded them, and exited — all of which fall under retrieving files for later analysis

<img width="1232" height="717" alt="image" src="https://github.com/user-attachments/assets/2d2dc267-53ea-4d9a-9508-26a152576600" />

### Steganography Extraction and Directory Verification
I successfully utilized the StegSeek tool to crack the "password" passphrase and extract hidden data from the file aa.jpg into a new output file named aa.jpg.out, which I subsequently verified by listing the directory contents using the ls -la command.

<img width="895" height="771" alt="image" src="https://github.com/user-attachments/assets/ab9b5106-60a5-4139-9577-48016b949236" />
## File Extraction and Content Analysis
I unzipped the previously extracted aa.jpg.out file to reveal the documents passwd.txt and shado, then viewed their contents using the cat command to uncover a hidden note about "Lian_Yu" and a specific string "M3tahuman."
<img width="1232" height="520" alt="image" src="https://github.com/user-attachments/assets/f633fe19-78a6-4207-a301-1a7f3011eb6d" />

## Remote Access and Flag Retrieval
I successfully established an SSH connection to the remote server as the user "slade," where I retrieved the user flag from user.txt and discovered a hint regarding "root Privileges" and a "Secret_Mission" inside the hidden .Important file.
<img width="1234" height="773" alt="image" src="https://github.com/user-attachments/assets/9be0ffa2-0760-4d30-8a8b-94cccb842482" />

I identified that the user "slade" had sudo permissions to run pkexec, which I exploited to spawn a root shell and successfully gain full administrative access to the system as indicated by the change to the "root" user identity.
<img width="1241" height="521" alt="image" src="https://github.com/user-attachments/assets/58dbbb49-0238-4c01-8412-8865b196a7a6" />

<img width="1225" height="250" alt="image" src="https://github.com/user-attachments/assets/7254b043-c7fb-4af7-ad84-3902fb9b2e10" />

