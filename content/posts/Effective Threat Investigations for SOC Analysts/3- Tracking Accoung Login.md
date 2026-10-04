---
title: "3  Tracking Accoung Login"
date: 2026-10-04T18:35:50+03:00
draft: false
toc: false
cover: "/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/3.%20thumb.jpg"
tags:
  - NTLM Protocol
  - Kerberos Protocol
  - User Logon Attempts
---

- Almost Everything and every action in a Windows Environment is mapped to an account and logged.

# 1. How to track Account Loggings

- I need to be able to detect and investigate compromised account activities.
- I'll be able to do so, by firstly tracking the account Login.

- In a windows environment, Login Event (either succeeded of failed) is recorded as event log.
- This event log includes valuable information, for instance:
	- Timestamp.
	- Account Name.
	- Authentication Method.
	- etc.

### In case of a Successful Login

- The Event Id responsible for recording successful logins is `Event Id 4624(S): An account was successfully logged on`.
- Microsoft docs: [4624(S) An account was successfully logged on](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624).

![event-4624](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event-4624.png)

- Windows Logon Type:
![windows_logon_type](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/windows_logon_type.png)

### In case of a failed Login

- The Event Id responsible for recording successful logins is `# 4625(F): An account failed to log on`.
- Microsoft docs: [4625(F) An account failed to log on](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4625)

![event_id_4625](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event_id_4625.png)

![windows_logon_failure_status_and_substatus_codes](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/windows_logon_failure_status_and_substatus_codes.png)

### In case of a blocked account

- Depending on the company security policy, after a number of failed logins attempts, the account will be locked for a certain time or until unlocked by the system administrator which is also predefined by the company security policy.

- Windows log the account lock event in `Event ID 4740(S): A user account was locked out`.
- Windows docs: [4740(S) A user account was locked out](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4740).

![event_id_4740](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event_id_4740.png)

### How to know if the successfully logged in user is an Administrator

- Since Administrative privileged accounts are of top sensitive accounts.
- Therefore, tracking this accounts' actions are important.

- To know if the newly logged in user is an Administrator.
- After checking the presented creds of the user and `Event ID 4624` is generated.

- Another event indicating that this account has privileges, which is `Event ID 4672(S): Special privileges assigned to new logon`.
- Windows docs: [4672(S) Special privileges assigned to new logon](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4672).
- which means that the newly logged in user is granted special privileges, ofc according to the company's security policy.

![event_id_4672](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event_id_4672.png)

### Logon Sessions Tracking

- After knowing the state of the user whois trying to log in, either a successful or failed login attempt.
- In case of a successful login attempt, I want to track this login session specifically when this user is going to log off.

- In case I want to track logon sessions, I want to know:
	- When the session started (the time of login, event 4624).
	- When the session ended (the time of logoff, event 4647 or 4634).
	- Compare timestamps between both events.

- Windows docs for both events:
	- [4647(S) User initiated logoff](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4647).
	- [4634(S) An account was logged off](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4634).

- To track the session timeframe, you firstly need to identify the `Logon ID`, which is a unique identifier for each logon session, from event id 4624.
- Then compare the timestamp between the Login event and Logoff Event.

***Event ID 4647***

![event_id_4647](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event_id_4647.png)

***Event ID 4634***

![event-4634](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event-4634.png)

---

# 2. Login Validation Events

- After going through logon events, we want to know the result of the presented credentials by the user, are they valid or invalid.
- Here where `Login Validation Events` comes into play.

- We have two scenarios:
	- ***`Domain Account Authentication`***—where the Domain Controller serves as the authentication server, and logs the login validation events.
	- ***`Local Account Authentication`***—where the workstation itself serves as the authentication server which authenticates the presented credentials using the SAM database, and the logs of the login validation events are logged on the workstation itself.

### Local Account Authentication

- Previously discussed Logon Event Logs are logged locally on the workstation that was presented the credentials in both cases.

### Domain Account Authentication

- In case of Domain Account Authentication, there're two authentication protocols to use one of them:
	- NTLM Protocol.
	- Kerberos Protocol.
- Based on the used authentication protocol, different logs will exist on the DC.

#### NTLM Protocol

- For both successful and failed logon events, Microsoft Logs only one Event which is ***`Event ID 4776(S, F): The computer attempted to validate the credentials for an account`***.
- Microsoft docs: [4776(S, F) The computer attempted to validate the credentials for an account](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4776).

![event-4776](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event-4776.png)

- The Error Code is equivalent to the sub-status code of failed login or event Id 4625.

#### Kerberos Protocol

![Kerb_auth](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/Kerb_auth.png)

- There're three event IDs generated here:

1. ***`4768(S, F): A Kerberos authentication ticket (TGT) was requested`:***
	- This Event records either the TGT request has succeeded or failed.
	- But in our case it'll represent a **Ticket Granting Ticker (TGT)** is created successfully.
	- Windows docs: [4768(S, F) A Kerberos authentication ticket (TGT) was requested](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768).
![event_id_4768](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event_id_4768.png)

2. ***`Event ID 4769(S, F): A Kerberos service ticket was requested.`:***
	- This event generates every time Key Distribution Center gets a Kerberos Ticket Granting Service (TGS) ticket request.
	- This event is generated only on domain controllers.
![event_id_4769](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event_id_4769.png)

3. ***`Event ID 4771(F): Kerberos pre-authentication failed`:***
	- This event record pre-authentication failures, which means that the DC failed to validate the provided credentials.
	- Therefore, the DC won't grant TGT or TGS.
![event-4771](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/event-4771.png)

- Failure Codes

| Failure Code | Description                                                |
| :----------- | ---------------------------------------------------------- |
| `0x6`        | Client not found in Kerberos database                      |
| `0x9`        | Password must reset                                        |
| `0x12`       | Account Disabled/Expired/locked-out, or out of logon hours |
| `0x17`       | Password Expired                                           |
| `0x18`       | Wrong Password                                             |
| `0x20`       | Ticket Expired                                             |

---

# Logs Investigations

## 1. Track Account Logons

### Track Successful Logon Attempts

- Firstly, I want to check successful logon activities for possible anomalies.
- SPL used:
```SPL
index=botsv3 sourcetype=WinEventLog EventCode=4624
| stats count by Logon_Type
| sort - count
```
![investigation_1](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_1.png)

- As you'll notice that Logon Type 5 is the dominant type across all other types.
- ***Logon Type 5:***
	- This is activated when the Windows Service Control Manager initiates a service using a specific user or an integrated service account like LOCAL SERVICE or NETWORK SERVICE.
	- Highly noisy and usually benign, but can be a red flag is a standard user account generates these type of logon, because it can indicate persistence via newly registered malicious service.

- ***Logon Type 2***—indicates interactive logon, the user is logging on at the local console.

- I'm going to check the process that initiated the logon activity:
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4624
| stats count by Process_Name
| sort - count
```
![investigation_2](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_2.png)

- Check fields from the Successful Logon Screenshot.
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4624
| eval user=mvindex(Account_Name,1), Logon_ID=mvindex(Logon_ID, 1)
| table _time host user Logon_ID Logon_Type Source_Network_address Workstation_Name
| sort _time
```
![investigation_3](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_3.png)

### Track Failed Logon Attempts

- Firstly, I've noticed that there's only 5 failed logon events.
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4625
```
![investigation_4](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_4.png)

- Tracking accounts which failed the authentication:
```SPL
index=botsv3 sourcetype="WinEventLog" EventCode=4625
| eval user=mvindex(Account_Name,1)
| stats count values(Logon_Type) as types by user Source_Network_Address host
| table user host types Source_Network_Address count
| sort - count
```
![investigation_5](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_5.png)

- Checking the failure reason:
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4625
| eval user=mvindex(Account_Name,1)
| stats count by user host Logon_Type Failure_Reason Sub_Status
| where isnotnull(user) AND isnotnull(Logon_Type)
```
![investigation_6](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_6.png)

### Tracking Blocked Account

- I'm gonna track Event ID 4740:
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4740
```
![investigation_7](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_7.png)

- Since there's no results to our search queries, therefore there's no blocked accounts to investigate in.

### Tracking Administrator Login

- Track Event ID 4672, which indicates that the logon is granted special privileges:
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4672
| stats count by Account_Name ComputerName
| sort - count
```
![investigation_8](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_8.png)

- I want to see administrative account logon, therefor I'm going to neglect service accounts.
- Note: I'm just traversing the logs not investigating any attack technique.
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4672
| stats count by Account_Name ComputerName
| where NOT match(Account_Name,"^(SYSTEM|LOCAL SERVICE|NETWORK SERVICE|DWM-\d+|UMFD-\d+)$") AND NOT match(Account_Name,"\$$")
| sort - count
```
![investigation_9](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_9.png)

### Logon Session Tracking

- Firstly, I need to identify two things:
	- The Target User Name of the user I want to track his logon session.
	- The corresponding Logon ID.
```SPL
index="botsv3" sourcetype="WinEventLog" EventCode=4624
| eval user=mvindex(Account_Name,1), logon_id=mvindex(Logon_ID,1)
| where NOT match(user,"^(SYSTEM|LOCAL SERVICE|NETWORK SERVICE|DWM-\d+|UMFD-\d+)$") AND NOT match(user,"\\$$")
| table _time user logon_id Logon_Type
```
![investigation_10](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_10.png)

- Let's make an assumption that I want to know the session duration for the first successful logon attempt.
- Therefore:
	- Take the logon ID for that specific logon event.
	- Search using it for events 4647 or 4634.
	- Take the timestamp for the event result.
	- Finally, you have the starting and ending timestamps, and you can calculate the logon session timing.
```SPL
index="botsv3" sourcetype="WinEventLog" Logon_ID="0x7797E" EventCode IN (4647 , 4634)
| table _time Account_Name Logon_ID
```
![investigation_11](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_11.png)

## 2. Logon Validation Events

### NTLM Protocol

- Let's see the:
	- Logon Account.
	- Source Workstation
	- Error Code
```SPL
index=t1110_003 EventCode=4776
| stats count by Logon_Account Source_Workstation Error_Code 
| sort - count
```
![investigation_12](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_12.png)

- Let's see the most occurred error codes and look at their relevant meaning from the table.
```SPL
index=t1110_003 EventCode=4776
| stats count by Error_Code 
| sort - count
```
![investigation_13](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_13.png)
- I've noticed that:
	- 80 logon attempts with wrong password.
	- 40 logon attempts with wrong username.

- Let's see which accounts are responsible for those failed logon attempts.
```SPL
index=t1110_003 sourcetype=* EventCode=4776
| stats count by Logon_Account Error_Code
| sort - count
```[cite: 3]
```
![investigation_14](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_14.png)

### Kerberos Protocol

- Traversing Event Codes 4768,4769
```SPL
index=t1110_003 sourcetype=* EventCode IN (4768,4769)
| stats count by EventCode Account_Name Client_Address Result_Code
| sort - count
```
![investigation_15](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_15.png)

- Traversing Pre-Authentication Failures:
```SPL
index=t1110_003 sourcetype=* EventCode=4771
| stats count by EventCode Account_Name Client_Address Failure_Code
| sort - count
```
![investigation_16](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/3-%20Tracking%20Account%20Login/investigation_16.png)

---
