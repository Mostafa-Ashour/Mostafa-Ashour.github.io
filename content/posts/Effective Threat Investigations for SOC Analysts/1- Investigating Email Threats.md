---
title: "1  Investigating Email Threats"
date: 2026-09-01T19:55:00+03:00
draft: false
toc: false
images:
tags:
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
index=main "victim@lab.local" ("Passed CLEAN" OR postfix/smtpd)
```

- This query filters out the noise and pulls up the exact mail server transaction logs for emails received by the victim.

#### Query Breakdown

- ***`index=main`***—Directs Splunk to search only within the `main` index, which is the default storage database where your lab's syslog data is being forwarded.
- ***`"victim@lab.local"`***—A free-text keyword search. It filters the results to only show log events that contain this exact email address anywhere in the text.
- ***`("Passed CLEAN" OR smtpd)`***—A logical filter that narrows the search down to specific mail server actions. It requires the log to contain _either_:
    - ***`"Passed CLEAN"`***—A tag content filter confirming the email was scanned and allowed through without being blocked.
	- ***`smtpd`***—The tag for the Postfix SMTP daemon, showing the exact moment the server received the inbound email connection from Kali.

## Step 1: Investigating the Email Sender Domain and SMTP Server Reputation

- You need to identify the true origin of the email to see if the sending server is malicious.

- In this step, I'm looking for two things:
	- Email Sender Domain.
	- SMTP Server IP Address.

```SPL
index=main smtpd "victim@lab.local"
| rex field=_raw "RCPT from [^\[]*\[(?<src_ip>[^\]]+)\]"
| stats count by helo, src_ip
| table helo, src_ip, count
```
![investigation_1](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/1-%20Investigating%20Email%20Threats/investigation_1.png)

- The `helo` field declares the self-declared hostname or domain used by the sending server (attacker vm) during SMTP Handshake.
- The `src_ip` is extracted from raw logs as it's the SMTP Server IP Address.

- Take each value, and search them on MXToolBox Online tool to check their reputation.
> [!note]
> Since both of them are self-made (by me), you might not find them having a bad reputation, I've just made them for testing.

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

## Step 4: Investigating Suspicious Email Content

- We've agreed that the attacker has two methods to either gain initial access to our system or to harvest the victim's credentials, which are by supplying the victim with either a Link or an Attachment.

- Either methods used, we should analyze them.

- In case of supplying a Link, you should scan it on tools like `urlscan.io` to check if it's has a bad reputation or not.
- The same thing in case the attacker has supplied the victim with a malicious attachment.

---
