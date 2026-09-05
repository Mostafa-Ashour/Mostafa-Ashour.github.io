---
title: "1  Investigating Email Threats"
date: 2026-09-01T19:55:00+03:00
draft: false
toc: false
images:
tags:
  - Email Threat Vectors
  - Email Threats
  - Email Spoofing
  - Phishing
---

# Architecture Overview

- A Home Lab for Chapter 1: Investigating Email Threats.

## Network Topology

![home_lab_topology](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/home_lab_topology.png)

- All components reside on an isolated internal network (`192.168.150.0/24`) to ensure complete lab safety while maintaining realistic cross-tier communications:
	* **Subnet**—`192.168.150.0/24`
	* **Domain Name**—`lab.local`
	* **Mail FQDN**—`mail.lab.local`

## Node Inventory & Specifications

| Node / Hostname                    | Management IP    | Operating System | Primary Software & Services                                                 | Role / Purpose                                                                 |
| :--------------------------------- | :--------------- | :--------------- | :-------------------------------------------------------------------------- | :----------------------------------------------------------------------------- |
| **Splunk SIEM** (`siem-solution`)  | `192.168.150.10` | Ubuntu Server    | Splunk Enterprise (UDP `514` / `1514`)                                      | Centralized log collection, indexing, and threat detection.                    |
| **Services VM** (`mail.lab.local`) | `192.168.150.20` | Ubuntu Server    | iRedMail (Postfix, Dovecot, Nginx, Roundcube, MariaDB, Fail2ban), `rsyslog` | Corporate mail environment hosting victim mailboxes and sending syslog events. |
| **Windows Victim** (`Win-Victim`)  | `192.168.150.30` | Windows 10       | Splunk Universal Forwarder, Sysmon, Mail Client / Web Browser               | Monitored victim endpoint consuming emails and interacting with payloads.      |
| **Kali Attacker** (`kali`)         | `192.168.150.50` | Ubuntu           | `swaks` (SMTP Swiss Army Knife), Gophish Framework                          | Red Team / Threat Emulation node generating spoofed and phishing emails.       |

---

## How to start services and check connectivity

### 1. Virtual Machine Startup Order

- To prevent connection timeouts and service errors, boot your Virtual Machines in this exact sequence:

```text
1. Splunk SIEM (192.168.150.10)   --> Start 1st (Collector must be ready)
2. Services VM (192.168.150.20)   --> Start 2nd (Mail Infrastructure)
3. Windows Victim (192.168.150.30) --> Start 3rd (Client Endpoint)
4. Kali Attacker (Dynamic IP)     --> Start 4th (Attack Platform)
```

### 2. Service Management (Starting & Verification)

#### A. Splunk SIEM VM (`192.168.150.10`)

**Start Service:**
```bash
sudo /opt/splunk/bin/splunk start --run-as-root
```

**Verify Service Status:**
```bash
sudo /opt/splunk/bin/splunk status --run-as-root
```

**Verify Listening Ports:**
```bash
sudo netstat -tulnp | grep -E '8000|514|9997'
```
*(Check that port `8000` for Web UI, `514` for Syslog, and `9997` for Windows Forwarder are active).*

#### B. Services VM - iRedMail (`192.168.150.20`)

**Start Services (if not set to auto-start):**
```bash
sudo systemctl start postfix dovecot nginx mariadb rsyslog
```

**Verify Service Status:**
```bash
sudo systemctl status postfix dovecot nginx mariadb rsyslog --no-pager
```

**Restart Log Forwarding Service:**
```bash
sudo systemctl restart rsyslog
```

#### C. Windows Victim VM (`192.168.150.30`)

**Verify Splunk Universal Forwarder Service (PowerShell):**
```powershell
Get-Service -Name SplunkForwarder
```

**Start Service (if stopped):**
```powershell
Start-Service -Name SplunkForwarder
```

#### D. Attacker VM

**Start Gophish Service (Optional):**
```bash
sudo gophish
```

### 3. Cross-Node Connectivity & Inter-Service Verification

- Execute these verification checks to ensure all nodes can communicate with each other.

#### Step 1: Network Ping Matrix (ICMP Reachability)

- Run `ping` tests between nodes to ensure basic network connectivity across `192.168.150.0/24`:

* **From Services VM (`.20`):**
  ```bash
  ping -c 2 192.168.150.10   # Ping Splunk SIEM
  ping -c 2 192.168.150.30   # Ping Windows Victim
  ```

* **From Kali Attacker:**
  ```bash
  ping -c 2 192.168.150.20   # Ping Services VM
  ping -c 2 192.168.150.10   # Ping Splunk SIEM
  ```

#### Step 2: Service Port Reachability Checks

Verify that application ports are open and accepting remote requests across VMs using `nc` (netcat) or PowerShell:

| Source Node        | Target Node & Port | Command                                                      | Expected Output           |
| :----------------- | :----------------- | :----------------------------------------------------------- | :------------------------ |
| **Services VM**    | Splunk UDP `514`   | `nc -zuv 192.168.150.10 514`                                 | `Connection succeeded!`   |
| **Windows Victim** | Services HTTP `80` | `Test-NetConnection -ComputerName 192.168.150.20 -Port 80`   | `TcpTestSucceeded : True` |
| **Windows Victim** | Splunk TCP `9997`  | `Test-NetConnection -ComputerName 192.168.150.10 -Port 9997` | `TcpTestSucceeded : True` |
| **Kali Attacker**  | Services SMTP `25` | `nc -zv 192.168.150.20 25`                                   | `Connection open`         |

#### Step 3: End-to-End Log Telemetry Pipeline Test

##### A. Test Mail Syslog Forwarding (Services VM -> Splunk)
1. Send a manual syslog payload from the **Services VM**:
   ```bash
   logger -p mail.info "HOMELAB STARTUP TEST: rsyslog integration verified"
   ```
2. Open Splunk Web (`http://192.168.150.10:8000`) and search:
   ```spl
   index=main "HOMELAB STARTUP TEST"
   ```

##### B. Test Windows Endpoint Telemetry (Windows -> Splunk)
1. Trigger a test Security Event on the **Windows Victim** via PowerShell:
   ```powershell
   Write-EventLog -LogName Application -Source "Application Error" -EventID 999 -EntryType Information -Message "HOMELAB STARTUP TEST: Windows Universal Forwarder active"
   ```
2. Search in Splunk Web:
   ```spl
   index=main host="Win-Victim" "HOMELAB STARTUP TEST"
   ```

---

# Attack Scenario and Experimentation

- For now, I'm gonna stick with the four Email Threat Types stated by the first chapter (might add more in the future), which are:
	1. Spearphishing Attachments.
	2. Spearphishing Links.
	3. Blackmail Email.
	4. Business Email Compromise.

## 1. Spearphishing Attachment

#### Main Idea

- The Main Idea of Spearphishing Attachment that the attacker send a malicious attachment to the victim and trick him into clicking/interacting with it.
- The purpose of the attacker is to either:
	- Gain initial foothold/access into the victim's system and subsequently into the corporate's network.
	- Harvest the victim's credentials.
- Multiple attachments types can be used, for instance, Microsoft Office Documents, PDF files, or HTML Files, etc.

#### Attack Scenario

- I'm gonna send an Invoice to the victim by the HR email in order to trick him to click on the attached file.

- Template used by attacker:
![spearphishing_attachment_1](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_attachment_1.png)

- HR Profile:
![spearphishing_attachment_2](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_attachment_2.png)

- Victim Email added to a group:
![spearphishing_attachment_3](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_attachment_3.png)

- Preparing the campaign to launch it.
![spearphishing_attachment_4](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_attachment_4.png)

- The Victim has received the Email from the HR.
![spearphishing_attachment_victim_received_email](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_attachment_victim_received_email.png)

- The victim clicking the/interacting with attachment depends on his security awareness or trainings.

## 2. Spearphishing Link

#### Main Idea

- Spearphishing Link's idea is the same as Spearphishing Attachment but instead of sending an malicious attachment within the email, the attacker will send a malicious link.
- After the victim clicking the/interacting with the malicious link, the attacker's purpose is to either:
	- The link is pointing to an attacker controlled server, hence downloading a malware on the victim's host and therefore gaining initial access to the victim's system.
	- The link is pointing to a landing page, a login form maybe, and this login form will look like the corporate's login form, therefore the victim will not notice any difference and might enter his own credentials. If so, the attacker will harvest the victim's credentials.

#### Attack Scenario

- Firstly, create the landing page, a vital constraint that the created landing page must be typical to the corporate's login page, therefore the victim will not notice any difference.
![spearphishing_link_1](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_link_1.png)

- Create a customized template, where the IT Support send an Urgent Password Verification Check to the Victim with the crafted Landing Page created in the previous step.
![spearphishing_link_2](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_link_2.png)

- Customize the campaign then launch it.
![spearphishing_link_3](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_link_3.png)

- The email has been sent to the victim.
![spearphishing_link_4](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_link_4.png)

- The victim has clicked the link.
![spearphishing_link_5](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_link_5.png)

- You'll notice that the attacker has received the victim's creds after he has entered them in the landing page.
![spearphishing_link_6](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/spearphishing_link_6.png)

## 3. Blackmail Email

#### Main Idea

- The main idea here, that the attacker will send an email to the victim (doesn't has to be a spoofed email) indicating that his system is compromised (might not be compromised but the attacker is trying to trick him) and that the attacker has exfiltrated sexual or confidential data from the victim's system.
- And for the attacker to not publish these data, he'll demand a payment.

- For the victim to believe that the attacker has exfiltrated sensitive data from his device, the attacker has two methods to prove infections:
	1. Screenshots from the breached data (the attacker can have it from previous breaches he has bought it previously, for instance) or from the victim's machine (if the attacker has actually compromised the victim's machine).
	2. Spoofing the target victim's email address, where the attacker will trick the victim into believing that the email came from his own email address.

#### Attack Scenario

- Create a customized template for Blackmail Email purpose.
![blackmail_email_1](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/blackmail_email_1.png)

- Create the campaign and launch it.
![blackmail_email_2](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/blackmail_email_2.png)

- The victim has received the email, and he either pay or not, this depends on the investigations of the security team.
![blackmail_email_3](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/blackmail_email_3.png)

## 4. Business Email Compromise (BEC)

#### Main Idea

- It has the main idea as the blackmail email, but instead of making the claim that you've exfiltrated the victim's data and demand payments; the attacker will spoof the CEO's Email, for instance and it can be the email of any important person in the company, then send an email to the finance team, for instance, demanding a bank transaction for any reason.
- Take into consideration any variations of this scenario.

---

# Logs Investigations

#### Note

- In some investigation steps, I couldn't apply it manually.
- Because it's a local lab, therefore applying it will not yield a meaningful results.

## Initial Discovery

- Firstly, you must isolate the suspicious email, which usually begins with a user report (e.g., `victim@lab.local` reporting a strange invoice) or by hunting for anomalous inbound traffic.

```SPL
index=main "victim@lab.local"
```

- Firstly, I want to analyze traffic that includes our employee email, which is `victim@lab.local`.
![investigation_2](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_2.png)

- You'll notice that there's something called ***Queue Code***, which is **unique tracking identifier** for a specific email message as it moves through the mail system.
- I can use the queue code field with our victim's email to track all email sent to him.

- To check how many email the victim has received:
```SPL
index=main "victim@lab.local" Queue-ID
```
![investigation_3](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_3.png)
- You'll notice that he has received four emails.

- Therefore, I want to extract those four Queue-IDs for further analysis:
```SPL
index=main "victim@lab.local"
| rex "Queue-ID:\s*(?<Queue_ID>[A-Za-z0-9]+)"
| table Queue_ID
| where Queue_ID != ""
```
```txt
4hYZQw153nz6yXV
4hYbqV5BrJz6ybn
4hYbRT4SYvz6ybf
4hYb1q4gy5z6ybg
```
![investigation_4](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_4.png)
#### Query Breakdown

- ***`index=main`***—This tells Splunk to search only inside the default `main` database, which is where all your lab's syslog data is forwarded and stored.
- ***`"victim@lab.local"`***—This acts as a strict keyword filter. It ensures Splunk only returns log events that contain this exact email address anywhere in the text. Which is our main focus right now.

## Step 1: Investigating the Email Sender Domain and SMTP Server Reputation

- You need to identify the true origin of the email to see if the sending server is malicious.

- In this step, I'm looking for two things:
	- Email Sender Domain.
	- SMTP Server IP Address.

```SPL
index=main "victim@lab.local"
| rex field=_raw "RCPT from [^\[]*\[(?<src_ip>[^\]]+)\]"
| stats count by helo, src_ip
| table helo, src_ip, count
```
![investigation_1](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_1.png)

- The `helo` field declares the self-declared hostname or domain used by the sending server (attacker vm) during SMTP Handshake.
- The `src_ip` is extracted from raw logs as it's the SMTP Server IP Address.

- Take each value, and search them on MXToolBox Online tool to check their reputation.

- Checking Email Sender Domain (in normal case, the tool will output the hostname and corresponding IP Address).
![investigation_5](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_5.png)

- When checking the email domain, if you found the IP Address found using our Search Query, therefore it's normal and not suspicious.
- But if not, you'll have to check it against blacklisted IP Addresses.
![investigation_6](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_6.png)

- Since it's a home-lab and the network is private, you'll not find any thing suspicious.

## Step 2: Spoofing Validation

- Since you've both the SMTP Server IP Address, and the Email Sender Domain.
- Therefore, you're going to do 2 things:
	- Check the `MX Record` (which is a type of DNS Record that tells the internet is responsible for accepting email on behalf of your domain) for the Email Sender Domain through MXToolBox.
	- Check the `WHOIS Record` (which is a public directory entry that details who owns and manages: domain names, IP Addresses, and network blocks) for the SMTP Server IP Address that sent the email.
- Finally, cross reference both results in order to see if attacker has spoofed the Email Domain or not.

## Step 3: Email Sender Behavior

- If the previous two steps couldn't validate to you that the email is suspicious, you can ask yourself the following questions:

1. Have the recipient received previous Emails from the same Email Sender or it's Domain?
2. Check whether the same email subject formula is being used by the email sender?
3. Check whether the email subject formula is related to their duties/job or not?

- In order to analyze the Email Sender Behavior; let's return to the four email that was sent to our victim and analyze them.
```SPL
index=main "victim@lab.local" Queue-ID
```
![investigation_7](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_7.png)
- In this case, I'll focus on the subject field (marked inside the red rectangle).

- To Extract it:
```SPL
index=main "victim@lab.local" Queue-ID
| rex "Subject:\s*\"(?<Email_Subject>[^\"]+)\""
| table Email_Subject
```
![investigation_8](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_8.png)
- I want to add the email sender address, in order to map the email sender and be able to analyze it's behavior.
```SPL
index=main "victim@lab.local" Queue-ID
| rex "Subject:\s*\"(?<Email_Subject>[A-Za-z0-9])\""
| rename helo as Email_Sender
| table Email_Sender, Email_Subject
```
![investigation_9](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_9.png)
- Since that the first 3 email subjects are suspicious enough, to validate that the emails sent from `attacker.lab.local` are malicious.
- But I wanted to add the email sender that appeared to our victim in our search query:
```SPL
index=main "victim@lab.local" Queue-ID
| rex "Subject:\s*\"(?<Email_Subject>[^\"]+)\""
| rex "From:\s*<*(?<Spoofed_Email_Sender>[^>]+)"
| rename helo as Real_Email_Sender
| table Real_Email_Sender, Spoofed_Email_Sender, Email_Subject
```
![investigation_10](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_10.png)
- Since every email the attacker spoofs the email sender address, and the Email Subjects are suspicious.
- Therefore, I can consider this email sender as malicious.

## Step 4: Investigating Suspicious Email Content

- We've agreed that the attacker has two methods to either gain initial access to our system or to harvest the victim's credentials, which are by supplying the victim with either a Link or an Attachment.

- Either methods used, we should analyze them.

### In case of Spearphishing Attachment

- I'm going to utilize the following query to know the time when each email was sent to our victim.
- Then I'm gonna analyze Event Codes generated on the victim's machine in order to know if any process related to file interaction happened in the same time the email was sent or after the email was sent by a small interval of time.

```SPL
index=main "victim@lab.local" Queue-ID
| rex "Subject:\s*\"(?<Email_Subject>[^\"]+)\""
| rex "Queue-ID:\s*(?<queue_id>[A-Za-z0-9]+)"
| table _time, queue_id, Email_Subject
```
![investigation_11](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_11.png)

- I'm going to know which Event Codes generated on the Victim's host.
```SPL
index=main host=Win-victim
| stats count by EventCode
| table EventCode, count
| sort EventCode
```
![investigation_12](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_12.png)

- The following are the only event codes related to a user interacting with an attachment:
	- **EventCode 1 — Process Creation**—Logs when a user or application launches a file or script, capturing execution details like the parent process and command-line parameters.
	- **EventCode 15 — FileCreateStreamHash**—Logs when a file is saved or downloaded from an untrusted source (like webmail or a browser), capturing the file's hash, target path, and source URL via the Windows _Mark of the Web_ stream.
	- **EventCode 27 — FileBlockExecutable**—Logs when Sysmon detects and prevents an executable file from being dropped or written to disk.

- Begin your time trimming, and begin to compare between the times the emails were sent and the time the event codes were generated.
```SPL
index=main host=Win-victim (EventCode=1 OR EventCode=15 OR EventCode=27) earliest=1725122249 latest=1788210000
| stats count by EventCode
| table count, EventCode
| sort - count
```
![investigation_13](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_13.png)
- Since the timeframe between the first sent email and the last email is small, therefore I stated:
	- The earliest time for logs as the time for the first sent email.
	- While the latest time you can make it after the last sent email with a considerable amount of time, because it's not mandatory that the victim has interacted with the attached attachment/link once he's received the email from the attacker.

- From now on, I'm gonna proceed with event code 15, because it's the logic event code to be used when you want to detect if the user has saved/downloaded an attachment from untrusted source, in our case an email.
- I'm gonna use this `sourcetype` `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` as event code 15 is logged under it.
![investigation_14](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_14.png)

- I'm going to extract the following fields in order to be able to analyze the attachment.
	- The two hashes if you wanted to check it on any online tool.
	- The Link to attachment if you wanted to download the document and analyze it either manually, or via any online tool.
	- The target file name and the EventCode.
```SPL
index=main sourcetype="XmlWinEventLog" "<EventID>15</EventID>" earliest=1725122249 latest=1788210000
| rex "EventID>(?<EventCode>[0-9]+)"
| rex "TargetFilename'>(?<TargetFilename>[A-Za-z0-9.:\\\s*]+)"
| rex "MD5=(?<MD5_Hash>[A-Za-z0-9]+)"
| rex "SHA256=(?<SHA256_Hash>[A-Za-z0-9]+)"
| rex "HostUrl=(?<Link_to_attachment>[^ ]+)"
| eval TargetFilename=replace(TargetFilename, ":Zone\.Identifier$", "")
| stats values(EventCode) as EventCode, values(Link_to_attachment) as Link_to_attachment, values(MD5_Hash) as MD5_Hash, values(SHA256_Hash) as SHA256_Hash by TargetFilename
```
![investigation_15](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_15.png)

---
