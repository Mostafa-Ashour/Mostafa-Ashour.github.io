---
title: "2 Email Header Analysis"
date: 2026-09-19T17:24:25+03:00
draft: false
toc: false
images:
tags:
  - Email Headers
---

- Email Flow and Header Analysis

# Email Flow

- ***Email Flow*** means the path/hops the email passes through when it's sent from the Sender until it's delivered to the Recipient.

![email_flow](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/2-%20Email%20Header%20Analysis/email_flow.png)


- ***MUA (Mail User Agent)***—The agent used by the client to send the email. For instance, Outlook or browsers like Microsoft Edge.
- ***MSA (Mail Submission Agent)***—the agent that will receive the mail after the client has submitted it to the MUA.
- ***MTA (Mail Transfer Agent)***—this is the mail server that receives the mail from the sender's MSA and passes it to one/several MTA servers until the mail is delivered to the recipient MX Server. It can also be called Relay Server.
- ***MX (Mail Exchange)***—This is the email server responsible of receiving messages for a particular domain after it has been transferred from one MTA to another until it reaches the recipient MX Server. This type of servers is identified by MX Records in the DNS records of the specified Domain.
- ***MDA (Mail Delivery Agent)***—this server is responsible of providing the recipient with the email sent to him after a successful authentication.

---

# Email Authentication

- ***Email Authentication*** is the process of checking that the sender's identity and domain are not spoofed before delivering the email to the recipient.

### How does Email Authentication works?

- The authentication of emails rely on the domain owner, he must ensure that their domain is not spoofed or impersonated by a malicious actor.
- Therefore, the process is going to be something like this:
	- the domain owner must add email authentication protocol records to it's DNS records and rules to be followed in order to check the legitimacy of received emails.
	- Whenever any email server receives any email from that specific domain it tries to authenticate it using the published authentication protocols and rules 
	- Finally, the receiving mail server determines the legitimacy of the received email and whether to deliver it to the recipient or to drop it.

## Email Authentication Protocols

- We've three Email Authentication Protocols:
	- SPF.
	- DKIM.
	- DMARC.

### SPF (Sender Policy Framework)

- This email authentication protocol provides a DNS TXT Record in the specified Domain's DNS Record that specifies which IP Addresses or Hostnames are allowed/authorized to send emails for and on behalf of this domain.

- According to this authentication protocol, it allows the receiving email server to:
	- Retrieve the sender domain's SPF record in it's DNS Records which contains authorized IP Address or Hostname.
	- Look up the IP Address or Hostname sent with the email message.
	- Cross-reference both results.
	- If they match, the receiving email server passes the received email, if not, it will block it.

![SPF_authentication_protocol](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/2-%20Email%20Header%20Analysis/SPF_authentication_protocol.png)

### SPF Record Example

```txt
v=spf1 ip4:192.0.2.0/24 include:_spf.google.com -all
```
- `v=spf1`—specifies the version of the SPF Protocol used.
- `ip4:192.0.2.0/24`—Specifies that this IPv4 range is authorized to send emails for the specified domain.
- `include:_spf.google.com`—Authorizes Google’s mail servers to send emails on behalf of the domain.
- `-all`—this indicates a "Hard Fail" which means that if the email sender IP Address or Hostname doesn't match the domain's SPF records, the email will be rejected.

### SPF example from an email header

```txt
Received-SPF: pass (google.com: domain of sender@example.com designates 192.0.2.1 as permitted sender) client-ip=192.0.2.1;
```

- This is an example of SPF Protocol existing in the received email header.
- As you notice, that the IP Address `192.0.2.1` is specified in the SPF records of the email sender domain to be authorized to send emails on behalf of the email sender domain.

## DKIM (DomainKeys Identified Email)

- ***DKIM*** is an Encryption Methodology or Digital Signature.
- It's added to email header as email authentication to prevent spoofing.

### DKIM Algorithm

1. Hash the Email Message Body.
2. Encrypts the hashed email message body., alongside with a list of email header parameters using the private key.
3. The public key is published in the DNS records of the signer's domain.
4. For the recipient email server, it'll retrieve the public key from the domain's DNS records and verify whether the signature is valid or not.

- DKIM Authentication Process
![dkim_authentication_process](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/2-%20Email%20Header%20Analysis/dkim_authentication_process.png)

### DKIM Signature Example from Email Header


```txt
DKIM-Signature: v=1; a=rsa-sha256; d=example.com; s=selector1;
  c=relaxed/simple;q=dns/txt; i=@example.com; t=1255993973;
  h=From:Sender:Reply-To:Subject:Date:Message-Id:To:Cc:
  MIME-Version:Content-Type:Content-Transfer-Encoding:Content-ID:
  Content-Description:Resent-Date:Resent-From:Resent-Sender:
  Resent-To:Resent-Cc:Resent-Message-ID:In-Reply-To:References:List-Id:
  List-Help:List-Unsubscribe:List-Subscribe:List-Post:List-Owner:
  List-Archive; bh=+7qxGePcmmrtZAIVQAtkSSGHfQ/ftNuvUTWJ3vXC9Zc=;
  b=dB85+qM+If1KGQmqMLNpqLgNtUaG5dhGjYjQD6/QXtXmViJx8tf9gLEjcHr+musL
  CAvr0Fsn1DA3ZLLlUxpf4ARCRftffOIp55Mj5EtDMm8v1UkcuftmgY9vcvpoPLDz
  yV/7EG/Gnmat6N7qrlC5Ip5sQqSaaaDwenkvOGGuDqs=;
```

- ***v=1***—Version, which is always 1.
- ***a=rsa-sha256***—This value refers to the algorithm that's used for both encryption and hashing to generate the digital signature.
- ***d=example.com***—This is the domain that is claiming to be authorized to send the email. This is where the receiving mail servers will fetch the public key to check the signature validity from.
- ***s=selector1***—This defines which value is used in the DNS lookup to determine which public key was used to sign the message. In this case public key would be stored in `selector1._domainkey.dkim-test1.com`.
- ***c=relaxed/simple***—this field refers to the canonicalization algorithm that determines how the body and the header are prepared for either the hashing/encryption algorithm. The first part is for the header and the second is for the body. `relaxed` means that certain modifications can be made to the email while preserving the essential content of the message. `simple` this canonicalization algorithm removes trailing spaces white spaces from email header fields and doesn't modify the message body before hashing.
- ***t=1255993973***—The epoch timestamp marking when the message was signed.
- ***h***—the value of this field is a colon-separated list of email headers included in the signature.
- ***bh***—This is the base64 encoded hash of the message body after it has been canonicalized via the method in `c` and then run through the hash function in `a`.
- ***b***—Finally, this is the DKIM signature itself. Everything else up to this point has been meta information on how this value was calculated.

## DMARC (Domain-Based Message Authentication, Reporting, and Conformance)

- It's an Email Authentication, Policy, and Reporting protocol, which depends on the SPF and DKIM authentication protocols results. 
- If one of them or both of them fail, DMARC will do two things:
	- Apply predefined policies by the Sender Domain Owner.
	- Reports the violation to the Sender Domain Owner.
- DMARC policies are published in the Sender Domain's DNS records as TXT record.

![dmarc_policy](/Effective%20Threat%20Investigations%20for%20SOC%20Analysts/2-%20Email%20Header%20Analysis/dmarc_policy.png)

### DMARC Example

```txt
"v=DMARC1;p=reject;pct=100;rua=mailto:mailmaster@example.com"
```

- ***`v`***—this field holds the version of the DMARC the domain uses.
- ***`p`***—this field's value tells the email recipient server which policy to apply in case that email fails authentication.
- ***`pct`***—the value of emails subjected to the policy. In our case, 100% of emails will be rejected if it fails authentication.
- ***`rua`***—specifies the URI of the mailbox that will receive DMARC reports.

### Authentication Results of an Email

```txt
Authentication-Results: mx.google.com;
       dkim=pass header.i=@valimail.com header.s=google2048 header.b=Z8L6tjHb;
       spf=pass (google.com: domain of [redacted]@valimail.com designates 209.85.220.41 as permitted sender) smtp.mailfrom=[redacted]@valimail.com;
       dmarc=pass (p=REJECT sp=REJECT dis=NONE) header.from=valimail.com
```

---

# Email Authentication Analysis

### In case of Success

```txt
Authentication-Results: mx.google.com;
       spf=pass (google.com: domain of sender@example.com designates
192.0.2.1 as permitted sender) smtp.mailfrom=sender@example.com;
       dkim=pass header.i=@example.com header.s=selector1 header.b=Gw+yUxcC;
       dmarc=pass (p=NONE) header.from=example.com
```

### In case of Failure

```txt
Authentication-Results: mx.google.com;
       spf=fail (google.com: domain of sender@example.com does not designate
192.0.2.1 as permitted sender) smtp.mailfrom=sender@example.com;
       dkim=fail (signature did not verify) header.i=@example.com header.s=selector1 header.b=Gw+yUxcC;
       dmarc=fail (p=QUARANTINE) header.from=example.com
```

---

# Email Header Analysis Example

```txt
Delivered-To: <REDACTED-RECIPIENT>@gmail.com
Received: by 2002:a17:907:985b:b0:c29:4c8d:3190 with SMTP id <REDACTED-ID>;
        Tue, 15 Sep 2026 03:10:10 -0700 (PDT)
X-Received: by 2002:a05:690e:130d:b0:671:4af3:8de with SMTP id <REDACTED-ID>;
        Tue, 15 Sep 2026 03:10:10 -0700 (PDT)
ARC-Seal: i=1; a=rsa-sha256; t=1789467010; cv=none;
        d=google.com; s=arc-20260327;
        b=WFr9PmMvkE0KyuMN3AoLDS5dDdSucVsgpE...[truncated]
ARC-Message-Signature: i=1; a=rsa-sha256; c=relaxed/relaxed; d=google.com; s=arc-20260327;
        h=to:from:subject:message-id:gmsai:date:mime-version:dkim-signature;
        bh=6+T61JzDwDj/Z+Ic5GCUMwq0JXe4JGL73herfFAVy+o=;
        fh=Hlz6O2ltZMIp395qayYjj6VRNrpMD5nex6cPGW/qsa0=;
        b=Lrv8Ri+2OOAtXhkpNLJ/2uzLoPjPXyS+506H...[truncated];
        dara=google.com
ARC-Authentication-Results: i=1; mx.google.com;
       dkim=pass header.i=@google.com header.s=20251104 header.b=uJtm9Hhi;
       spf=pass (google.com: domain of <REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com designates 209.85.220.69 as permitted sender) smtp.mailfrom=<REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com;
       dmarc=pass (p=REJECT sp=REJECT dis=NONE) header.from=google.com;
Return-Path: <<REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com>
Received: from mail-sor-f69.google.com (mail-sor-f69.google.com. [209.85.220.69])
        by mx.google.com with SMTPS id <REDACTED-ID>
        for <<REDACTED-RECIPIENT>@gmail.com>
        (Google Transport Security);
        Tue, 15 Sep 2026 03:10:10 -0700 (PDT)
Received-SPF: pass (google.com: domain of <REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com designates 209.85.220.69 as permitted sender) client-ip=209.85.220.69;
Authentication-Results: mx.google.com;
       dkim=pass header.i=@google.com header.s=20251104 header.b=uJtm9Hhi;
       spf=pass (google.com: domain of <REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com designates 209.85.220.69 as permitted sender) smtp.mailfrom=<REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com;
       dmarc=pass (p=REJECT sp=REJECT dis=NONE) header.from=google.com;
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=google.com; s=20251104; t=1789467010; x=1790071810; dara=google.com;
        h=content-type:to:from:subject:message-id:gmsai:date:mime-version
         :from:to:cc:subject:date:message-id:reply-to:content-type;
        bh=6+T61JzDwDj/Z+Ic5GCUMwq0JXe4JGL73herfFAVy+o=;
        b=uJtm9Hhi5DNbItz96W4CMtA+bgiJj5BDJs...[truncated]
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20260707; t=1789467010; x=1790071810;
        h=content-type:to:from:subject:message-id:gmsai:date:mime-version
         :x-gm-message-state:from:to:cc:subject:date:message-id:reply-to
         :content-type;
        bh=6+T61JzDwDj/Z+Ic5GCUMwq0JXe4JGL73herfFAVy+o=;
        b=ricQwXLGyaha2iC58JrEM9ofd7L1oAz5vA...[truncated]
X-Gm-Message-State: <REDACTED-TOKEN>
MIME-Version: 1.0
X-Received: by 2002:a05:690e:13c9:b0:671:3f16:cd3 with SMTP id <REDACTED-ID>;
        Tue, 15 Sep 2026 03:10:10 -0700 (PDT)
Date: Tue, 15 Sep 2026 03:10:10 -0700
gmsai: true
X-Google-Notification-Metadata: <REDACTED-TOKEN>
X-Notifications: <REDACTED-TOKEN>
X-Notifications-Bounce-Info: <REDACTED-TOKEN>
Message-ID: <REDACTED-MESSAGE-ID>@notifications.google.com
Subject: Get started with Google services for families by inviting family members
From: Google <families-noreply@google.com>
To: <REDACTED-RECIPIENT>@gmail.com
Content-Type: multipart/alternative; boundary="0000000000007ae7f0065b82c1e3"

--0000000000007ae7f0065b82c1e3
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 7bit

Hi Mostafa,

This is a sample message used for email header analysis.

Regards,
Sample Sender

--0000000000007ae7f0065b82c1e3
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: 7bit

<html>
<body>
<p>Hi Mostafa,</p>
<p>This is a sample message used for email header analysis.</p>
<p>Regards,<br>Sample Sender</p>
</body>
</html>

--0000000000007ae7f0065b82c1e3--
```

### First Part

- Email Headers are read from Bottom to Top.
- Firstly, you'll find:
	- Email Content.
	- Content-Type, To, From, and Subject fields.
- Which is the first part of the email headers where the sender normally inputs through his Mail User Agent, for instant, Outlook.
```txt
Message-ID: <REDACTED-MESSAGE-ID>@notifications.google.com
Subject: Get started with Google services for families by inviting family members
From: Google <families-noreply@google.com>
To: <REDACTED-RECIPIENT>@gmail.com
Content-Type: multipart/alternative; boundary="0000000000007ae7f0065b82c1e3"

--0000000000007ae7f0065b82c1e3
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 7bit

Hi Mostafa,

This is a sample message used for email header analysis.

Regards,
Sample Sender

--0000000000007ae7f0065b82c1e3
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: 7bit

<html>
<body>
<p>Hi Mostafa,</p>
<p>This is a sample message used for email header analysis.</p>
<p>Regards,<br>Sample Sender</p>
</body>
</html>

--0000000000007ae7f0065b82c1e3--
```

### Second Part

- **X-headers are custom, non-standard email headers.**
- Mail servers, spam filters, and email clients add them to record extra information that the official email standards don't define.

- Take `X-Received` for example, `X-Received` is a Google-specific header that logs an internal hop inside Google's own systems.
- In our example:
```txt
X-Received: by 2002:a05:690e:130d:b0:671:4af3:8de with SMTP id ...;
        Tue, 15 Sep 2026 03:10:10 -0700 (PDT)
```
- **`by 2002:a05:...`**—This is the internal Google system that received the message. It looks like an IPv6 address, but it is an internal identifier, not a real public IP, so it doesn't help with tracing the sender.
- **The trailing ID**—This is an internal identifier for that step.
- **The timestamp**—This shows when that internal handoff happened.

### Third Part

- You'll find the Authentication Results which upon it will result the action taken on that email.
- In our case, our email passed authentication protocols specified by google.
```txt
Authentication-Results: mx.google.com;
       dkim=pass header.i=@google.com header.s=20251104 header.b=uJtm9Hhi;
       spf=pass (google.com: domain of <REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com designates 209.85.220.69 as permitted sender) smtp.mailfrom=<REDACTED-VERP-ADDRESS>@chime-notifications.bounces.google.com;
       dmarc=pass (p=REJECT sp=REJECT dis=NONE) header.from=google.com;
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
```

---
