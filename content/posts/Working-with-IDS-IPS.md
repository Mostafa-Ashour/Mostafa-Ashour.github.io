---
title: "Working With IDS IPS"
date: 2026-04-16T21:46:20+02:00
draft: false
toc: false
images:
tags:
  - untagged
---

# Introduction To IDS/IPS

- ***Network security monitoring (NSM)*** relies on `Intrusion Detection Systems (IDS)` and `Intrusion Prevention Systems (IPS)` to:
  - Identify potential Threats.
  - Mitigate their impact.
- An `Intrusion Detection System (IDS)`:
  - Monitors network or system activity for malicious behavior and policy violations.
  - Reporting to a management station to provide network visibility.
- While an `IDS` alerts to intrusions, it does not prevent them.

- The IDS operates in two main modes:
  - Signature-Based Detection.
  - Anomaly-Based Detection.
- In `signature-based detection`:
  - The IDS recognizes bad patterns, such as malware signatures and previously identified attack patterns.
  - However, this method is limited to only known threats.
- For this reason, `anomaly-based detection` is implemented:
  - Which establishes a baseline of normal behavior and sends an alert when it detects behavior deviating from this baseline.
  - It's a more proactive approach but is susceptible to false positives, hence why both methods are used to balance each other out.

- An `IPS`, positioned behind the firewall, actively prevents detected threats, providing an additional layer of protection by not only alerting to intrusions but also actively stops them from entering.
- An `IPS` also operates in a similar mode to IDS, offering both `signature-based` and `anomaly-based detection`.
- Once a potential threat is detected, it takes actions such as:
  - Dropping malicious packets.
  - Blocking network traffic.
  - Resetting the connection.
- The goal is to interrupt or halt activities that are deemed dangerous to our network or against our policy.

- IDS and IPS are typically deployed on the internal network side, behind the firewall, to:
  - Maximize visibility of relevant traffic.
  - Effectively protect resources.

- ***Intrusion Detection Systems (IDS)***:
  - Passively monitor network traffic for threats and generate alerts.
  - Positioned behind the firewall, they analyze traffic that has bypassed initial defenses, focusing on complex threats.
- ***Intrusion Prevention Systems (IPS)***:
  - Actively block detected threats.
  - Deployed inline, typically directly behind the firewall, IPS do both monitor and block malicious traffic.

- Intrusion Detection/Prevention Systems (IDS/IPS) deployment depends on network requirements and traffic monitoring needs.
- These systems can be implemented at the host level (HIDS/HIPS) to monitor individual host traffic for suspicious activity.
- Placement of these systems is integral to a defense-in-depth strategy.
- The specific architecture will vary based on:
  - Network characteristics/nature.
  - Data sensitivity.
  - The threat landscape.

## IDS/IPS Updates

- Moreover, to ensure these systems perform at their best, consistently:
  - Update them with the latest threat signatures.
  - Fine-tune their anomaly detection algorithms.
- This requires a diligent, ongoing effort from our security team, but it's absolutely essential given the continually evolving threat landscape.

- Security Information and Event Management (SIEM) systems are also crucial.
- By aggregating logs from IDS/IPS and other network devices, SIEMs:
  - Correlate events and use advanced analytics to detect complex attacks.
  - Providing a unified security view for rapid threat response.

---

# Suricata Fundamentals

- Suricata is a robust, open-source network security engine for IDS, IPS, and NSM.
- Developed and maintained by the OISF, it showcases the power of community-led, non-profit projects.

- Suricata dissect/analyze all network traffic to find potential malicious activity.
- It can broadly:
	- Assess the network's health/condition.
	- Also investigate specific application-level interactions.
- Suricata uses a detailed set of rules to guide its analysis, helping it identify possible threats.
- It is designed for high-speed performance on standard and specialized hardware, making it a very efficient tool.

## Suricata Operation Modes

- Suricata operates in four (4) distinct modes.

#### `Intrusion Detection System (IDS) mode`

- In this mode, Suricata:
	- Passively monitors network traffic.
	- Identifying potential attacks without intervention.
- This enhances network visibility and accelerates response times by providing an in-depth view of network activities, though it does not offer direct protection.

#### `Intrusion Prevention System (IPS) mode`

- In this mode, Suricata proactively inspects all network traffic, allowing only approved traffic to access the internal network.
- This strengthens security by preventing attacks before they reach the internal network.
- However, deploying Suricata in IPS mode:
	- Requires thorough understanding of network landscape to avoid blocking legitimate traffic.
	- Each rule requires careful testing.
- While enhancing security, this inspection process may introduce latency, as it requires Suricata to inspect and process every packet of inbound traffic before letting it in.

#### `Intrusion Detection Prevention System (IDPS) mode`

- This mode brings together the best of both IDS and IPS.
- While Suricata continues to passively monitor traffic, it possesses the ability to actively transmit RST packets in response to abnormal activities.
- This mode strikes a balance between active protection and maintaining low latency, crucial for seamless network operations.

#### `Network Security Monitoring (NSM) mode`

- Suricata functions as a comprehensive/dedicated logging tool/mechanism, recording all network traffic for later security incident investigation.
- While it doesn't actively analyze or prevent traffic, it provides a wealth of data despite the high volume of data generated.

## Suricata Inputs

- Regarding Suricata inputs, there are two main categories:

1. `Offline Input`—Reading PCAP in the `LibPCAP` file format files allows for post-capture packet processing, facilitating both forensic data analysis and rule set experimentation.
2. `Live Input`:
	1. `LibPCAP` enables live input by reading packets directly from network interfaces, but it suffers from performance and load-balancing issues.
	2. For inline operations, `NFQ` and `AF_PACKET` are alternatives.
	3. `NFQ`, a Linux-specific inline IPS mode, integrates with `IPTables` to redirect packets to Suricata for inspection, requiring drop rules for effective blocking.
	4. `AF_PACKET` offers better performance and multi-threading than `LibPCAP`, but it lacks compatibility with older Linux versions and cannot be used inline on routing machines.

## Suricata Outputs

- Suricata creates multiple outputs, including:
	- Logs.
	- Alerts.
	- Additional network-related data such as DNS requests and network flows.
- One of the most critical outputs is `EVE`, a JSON formatted log that records a wide range of event types including alerts, HTTP, DNS, TLS metadata, drop, SMTP metadata, flow, netflow, and more.
- Tools such as `Logstash` can easily consume this output, facilitating data analysis.

- We might encounter `Unified2` Suricata output, which is essentially a Snort binary alert format, enabling integration with other software that leverages Unified2.
- Any Unified2 output can be read using Snort’s `u2spewfoo` tool, which is a straightforward and effective method to gain insight into the alert data.

## Configuring Suricata & Custom Rules

- To be able to get an overview of all the rule files:
```shell
ls -lah /etc/suricata/rules/
```
```
total 27M
drwxr-xr-x 2 root root 4.0K Jun 28 12:10 .
drwxr-xr-x 3 root root 4.0K Jul  4 14:44 ..
-rw-r--r-- 1 root root  31K Jun 27 20:55 3coresec.rules
-rw-r--r-- 1 root root 1.9K Jun 15 05:51 app-layer-events.rules
-rw-r--r-- 1 root root 2.1K Jun 27 20:55 botcc.portgrouped.rules
-rw-r--r-- 1 root root  27K Jun 27 20:55 botcc.rules
-rw-r--r-- 1 root root 109K Jun 27 20:55 ciarmy.rules
-rw-r--r-- 1 root root  12K Jun 27 20:55 compromised.rules
-rw-r--r-- 1 root root  21K Jun 15 05:51 decoder-events.rules
-rw-r--r-- 1 root root  468 Jun 15 05:51 dhcp-events.rules
-rw-r--r-- 1 root root 1.2K Jun 15 05:51 dnp3-events.rules
-rw-r--r-- 1 root root 1.2K Jun 15 05:51 dns-events.rules
-rw-r--r-- 1 root root  32K Jun 27 20:55 drop.rules
-rw-r--r-- 1 root root 2.7K Jun 27 20:55 dshield.rules
-rw-r--r-- 1 root root 365K Jun 27 20:55 emerging-activex.rules
-rw-r--r-- 1 root root 613K Jun 27 20:55 emerging-adware_pup.rules
-rw-r--r-- 1 root root 650K Jun 27 20:55 emerging-attack_response.rules
-rw-r--r-- 1 root root  33K Jun 27 20:55 emerging-chat.rules
-rw-r--r-- 1 root root  19K Jun 27 20:55 emerging-coinminer.rules
-rw-r--r-- 1 root root 119K Jun 27 20:55 emerging-current_events.rules
-rw-r--r-- 1 root root 1.7M Jun 27 20:55 emerging-deleted.rules
-rw-r--r-- 1 root root  20K Jun 27 20:55 emerging-dns.rules
-rw-r--r-- 1 root root  62K Jun 27 20:55 emerging-dos.rules
-rw-r--r-- 1 root root 606K Jun 27 20:55 emerging-exploit_kit.rules
-rw-r--r-- 1 root root 1.1M Jun 27 20:55 emerging-exploit.rules
-rw-r--r-- 1 root root  45K Jun 27 20:55 emerging-ftp.rules
-rw-r--r-- 1 root root  37K Jun 27 20:55 emerging-games.rules
-rw-r--r-- 1 root root 572K Jun 27 20:55 emerging-hunting.rules
-rw-r--r-- 1 root root  18K Jun 27 20:55 emerging-icmp_info.rules
-rw-r--r-- 1 root root  11K Jun 27 20:55 emerging-icmp.rules
-rw-r--r-- 1 root root  15K Jun 27 20:55 emerging-imap.rules
-rw-r--r-- 1 root root  11K Jun 27 20:55 emerging-inappropriate.rules
-rw-r--r-- 1 root root 2.9M Jun 27 20:55 emerging-info.rules
-rw-r--r-- 1 root root  48K Jun 27 20:55 emerging-ja3.rules
-rw-r--r-- 1 root root 8.2M Jun 27 20:55 emerging-malware.rules
---SNIP---
```

- The rules can be seen in a straightforward list format and be inspected to understand their functionality, as follows.
```shell
more /etc/suricata/rules/emerging-malware.rules
```
```
# Emerging Threats
#
# This distribution may contain rules under two different licenses.
#
#  Rules with sids 1 through 3464, and 100000000 through 100000908 are under the GPLv2.
#  A copy of that license is available at http://www.gnu.org/licenses/gpl-2.0.html
#
#  Rules with sids 2000000 through 2799999 are from Emerging Threats and are covered under the BSD License
#  as follows:
#
#*************************************************************
#  Copyright (c) 2003-2022, Emerging Threats
#  All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without modification, are permitted provided that the
#  following conditions are met:
#
#  * Redistributions of source code must retain the above copyright notice, this list of conditions and the following
#    disclaimer.
#  * Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the
#    following disclaimer in the documentation and/or other materials provided with the distribution.
#  * Neither the name of the nor the names of its contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS AS IS AND ANY EXPRESS OR IMPLIED WARRANTIES,
#  INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
#  DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
#  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
#  SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
#  WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE
#  USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#*************************************************************
#
#
#
#

# This Ruleset is EmergingThreats Open optimized for suricata-5.0-enhanced.

#alert tcp $HOME_NET any -> $EXTERNAL_NET any (msg:"ET MALWARE Psyb0t joining an IRC Channel"; flow:established,to_server; flowbits:isset,is_proto_irc; content:"
JOIN #mipsel"; reference:url,www.adam.com.au/bogaurd/PSYB0T.pdf; reference:url,doc.emergingthreats.net/2009172; classtype:trojan-activity; sid:2009172; rev:2; me
tadata:created_at 2010_07_30, updated_at 2010_07_30;)

alert tcp $HOME_NET any -> $EXTERNAL_NET 25 (msg:"ET MALWARE SC-KeyLog Keylogger Installed - Sending Initial Email Report"; flow:established,to_server; content:"
Installation of SC-KeyLog on host "; nocase; content:"<p>You will receive a log report every "; nocase; reference:url,www.soft-central.net/keylog.php; reference:
url,doc.emergingthreats.net/2002979; classtype:trojan-activity; sid:2002979; rev:4; metadata:created_at 2010_07_30, updated_at 2010_07_30;)

alert tcp $HOME_NET any -> $EXTERNAL_NET 25 (msg:"ET MALWARE SC-KeyLog Keylogger Installed - Sending Log Email Report"; flow:established,to_server; content:"SC-K
eyLog log report"; nocase; content:"See attached file"; nocase; content:".log"; nocase; reference:url,www.soft-central.net/keylog.php; reference:url,doc.emerging
threats.net/2008348; classtype:trojan-activity; sid:2008348; rev:2; metadata:created_at 2010_07_30, updated_at 2010_07_30;)
---SNIP---
```

- Rules might be commented out, meaning they aren't loaded and don't affect the system.
- This usually happens when:
	- A new version of the rule comes into play.
	- If the threat associated with the rule becomes outdated or irrelevant.

- Each rule usually involves specific variables, such as `$HOME_NET` and `$EXTERNAL_NET`.
- The rule examines traffic from the IP addresses specified in the `$HOME_NET` variable heading towards the IP addresses in the `$EXTERNAL_NET` variable.

- These variables can be defined in the `suricata.yaml` configuration file.
```shell
more /etc/suricata/suricata.yaml
```
```
%YAML 1.1
---

# Suricata configuration file. In addition to the comments describing all
# options in this file, full documentation can be found at:
# https://suricata.readthedocs.io/en/latest/configuration/suricata-yaml.html

# This configuration file generated by Suricata 6.0.13.
suricata-version: "6.0"

##
## Step 1: Inform Suricata about your network
##

vars:
  # more specific is better for alert accuracy and performance
  address-groups:
    HOME_NET: "[10.0.0.0/8]"
    #HOME_NET: "[192.168.0.0/16]"
    #HOME_NET: "[10.0.0.0/8]"
    #HOME_NET: "[172.16.0.0/12]"
    #HOME_NET: "any"

    EXTERNAL_NET: "!$HOME_NET"
    #EXTERNAL_NET: "any"

    HTTP_SERVERS: "$HOME_NET"
    SMTP_SERVERS: "$HOME_NET"
    SQL_SERVERS: "$HOME_NET"
    DNS_SERVERS: "$HOME_NET"
    TELNET_SERVERS: "$HOME_NET"
    AIM_SERVERS: "$EXTERNAL_NET"
    DC_SERVERS: "$HOME_NET"
    DNP3_SERVER: "$HOME_NET"
---SNIP---
```

- This allows us to:
	- Customize these variables according to our specific network environment.
	- Even define our own variables.

- Finally, to configure Suricata to load signatures from a custom rules file, such as `local.rules`, execute the following.
```shell
sudo vim /etc/suricata/suricata.yaml
```
1. Add `<path-to-rules-file>/local.rules` to `rule-files:`.
2. Press the `Esc` key.
3. Enter `:wq` and then, press the `Enter` key.

## Hands-on With Suricata Inputs

- With Suricata inputs, we can experiment with both offline and live input:

1. For offline input (reading PCAP files - [suspicious.pcap](https://dingtoffee.medium.com/holiday-hack-challenge-2022-writeup-stage-2-recover-the-tolkien-ring-3f76fcd7dab9) in this case):
	- The following command needs to be executed.
	- Then Suricata will create various logs (mainly `eve.json`, `fast.log`, and `stats.log`).
```shell
suricata -r /home/htb-student/pcaps/suspicious.pcap
```
```
5/7/2023 -- 13:35:51 - <Notice> - This is Suricata version 6.0.13 RELEASE running in USER mode
5/7/2023 -- 13:35:51 - <Notice> - all 3 packet processing threads, 4 management threads initialized, engine started.
5/7/2023 -- 13:35:51 - <Notice> - Signal Received.  Stopping engine.
5/7/2023 -- 13:35:51 - <Notice> - Pcap-file module read 1 files, 5172 packets, 3941260 bytes
```

- An alternative command can be executed to bypass checksums (`-k` flag) and log in a different directory (`-l` flag).
```shell
suricata -r /home/htb-student/pcaps/suspicious.pcap -k none -l .
```
- Command Explanation:
	- `suricata`—Runs the Suricata IDS/IPS engine to analyze network traffic.
	- `-r suspicious.pcap`—Reads and analyzes packets from the specified PCAP file instead of live capture.
	- `-k none`—Disables checksum validation so no packets are ignored due to invalid checksums.
	- `-l .`—Saves all output logs (alerts, events, stats) in the current working directory.

2. For live input, we can try Suricata’s (Live) `LibPCAP` mode as follows.

- Obtain relevant NIC:
```shell
ifconfig
```

```shell
sudo suricata --pcap=ens160 -vv
```
```shell
[sudo] password for user:
5/7/2023 -- 13:44:01 - <Notice> - This is Suricata version 6.0.13 RELEASE running in SYSTEM mode
5/7/2023 -- 13:44:01 - <Info> - CPUs/cores online: 2
5/7/2023 -- 13:44:01 - <Info> - Setting engine mode to IDS mode by default
5/7/2023 -- 13:44:01 - <Info> - Found an MTU of 1500 for 'ens160'
5/7/2023 -- 13:44:01 - <Info> - Found an MTU of 1500 for 'ens160'
5/7/2023 -- 13:44:01 - <Info> - fast output device (regular) initialized: fast.log
5/7/2023 -- 13:44:01 - <Info> - eve-log output device (regular) initialized: eve.json
5/7/2023 -- 13:44:01 - <Info> - stats output device (regular) initialized: stats.log
5/7/2023 -- 13:44:01 - <Info> - Running in live mode, activating unix socket
5/7/2023 -- 13:44:01 - <Perf> - using shared mpm ctx' for http_uri
---SNIP---
```

3. For Suricata in Inline (`NFQ`) mode, the following command should be executed first.
```shell
sudo iptables -I FORWARD -j NFQUEUE
```

- Then, we should be able to execute the following.
```shell
sudo suricata -q 0
```
```
5/7/2023 -- 13:52:38 - <Notice> - This is Suricata version 6.0.13 RELEASE running in SYSTEM mode
5/7/2023 -- 13:52:39 - <Notice> - all 4 packet processing threads, 4 management threads initialized, engine started.
```

- Moreover, to try Suricata in `IDS` mode with `AF_PACKET` input, execute one of the below.
```shell
sudo suricata -i ens160
```
```
5/7/2023 -- 13:53:35 - <Notice> - This is Suricata version 6.0.13 RELEASE running in SYSTEM mode
5/7/2023 -- 13:53:35 - <Notice> - all 1 packet processing threads, 4 management threads initialized, engine started.
```

```shell
sudo suricata --af-packet=ens160
```
```
5/7/2023 -- 13:54:34 - <Notice> - This is Suricata version 6.0.13 RELEASE running in SYSTEM mode
5/7/2023 -- 13:54:34 - <Notice> - all 1 packet processing threads, 4 management threads initialized, engine started.
```

- To observe Suricata dealing with "live" traffic, let's:
	- Establish an additional SSH connection.
	- Then utilize `tcpreplay` to replay network traffic from a PCAP file (`suspicious.pcap` in this case).
```shell
sudo tcpreplay -i ens160 <path-to-pcap-file>/suspicious.pcap
```
```
^C User interrupt...
sendpacket_abort
Actual: 730 packets (663801 bytes) sent in 22.84 seconds
Rated: 29060.3 Bps, 0.232 Mbps, 31.95 pps
Statistics for network device: ens160
        Successful packets:        729
        Failed packets:            0
        Truncated packets:         0
        Retried packets (ENOBUFS): 0
        Retried packets (EAGAIN):  0
```

- Then, feel free to terminate both tcpreplay and Suricata. The logs from the observed (replayed) traffic will be available at `/var/log/suricata`.

- The `-i` option helps Suricata choose the best input option. In the case of Linux, the best input option is `AF_PACKET`.
- If pcap mode is needed, the `--pcap` option is recommended.
- Configuration of (Live) `LibPCAP` can be achieved via the `suricata.yaml` file, including settings for buffer size, BPF or tcpdump filters, checksum validation, threads, promiscuous mode, snap length, etc.

## Hands-on With Suricata Outputs

- Suricata logs:
	- Stored by default in `/var/log/suricata` and accessible with root privileges.
	- Include `eve.json`, `fast.log`, and `stats.log`.
- These files offer crucial insights into network activity. Let's examine each one.

1. `eve.json`—This file contains Suricata's  recommended output, including JSON objects with each object including information like `timestamps`, `flow_id`, and `event_type`.
2. Inspect `old_eve.json` in `/var/log/suricata` for examples.
```shell
less /var/log/suricata/old_eve.json
```
```json
{"timestamp":"2023-07-06T08:34:24.526482+0000","event_type":"stats","stats":{"uptime":8,"capture":{"kernel_packets":4,"kernel_drops":0,"errors":0},"decoder":{"pkts":3,"bytes":212,"invalid":0,"ipv4":0,"ipv6":1,"ethernet":3,"chdlc":0,"raw":0,"null":0,"sll":0,"tcp":0,"udp":0,"sctp":0,"icmpv4":0,"icmpv6":1,"ppp":0,"pppoe":0,"geneve":0,"gre":0,"vlan":0,"vlan_qinq":0,"vxlan":0,"vntag":0,"ieee8021ah":0,"teredo":0,"ipv4_in_ipv6":0,"ipv6_in_ipv6":0,"mpls":0,"avg_pkt_size":70,"max_pkt_size":110,"max_mac_addrs_src":0,"max_mac_addrs_dst":0,"erspan":0,"event":{"ipv4":{"pkt_too_small":0,"hlen_too_small":0,"iplen_smaller_than_hlen":0,"trunc_pkt":0,"opt_invalid":0,"opt_invalid_len":0,"opt_malformed":0,"opt_pad_required":0,"opt_eol_required":0,"opt_duplicate":0,"opt_unknown":0,"wrong_ip_version":0,"icmpv6":0,"frag_pkt_too_large":0,"frag_overlap":0,"frag_ignored":0},"icmpv4":{"pkt_too_small":0,"unknown_type":0,"unknown_code":0,"ipv4_trunc_pkt":0,"ipv4_unknown_ver":0},"icmpv6":{"unknown_type":0,"unknown_code":0,"pkt_too_small":0,"ipv6_unknown_version":0,"ipv6_trunc_pkt":0,"mld_message_with_invalid_hl":0,"unassigned_type":0,"experimentation_type":0},"ipv6":{"pkt_too_small":0,"trunc_pkt":0,"trunc_exthdr":0,"exthdr_dupl_fh":0,"exthdr_useless_fh":0,"exthdr_dupl_rh":0,"exthdr_dupl_hh":0,"exthdr_dupl_dh":0,"exthdr_dupl_ah":0,"exthdr_dupl_eh":0,"exthdr_invalid_optlen":0,"wrong_ip_version":0,"exthdr_ah_res_not_null":0,"hopopts_unknown_opt":0,"hopopts_only_padding":0,"dstopts_unknown_opt":0,"dstopts_only_padding":0,"rh_type_0":0,"zero_len_padn":0,"fh_non_zero_reserved_field":0,"data_after_none_header":0,"unknown_next_header":0,"icmpv4":0,"frag_pkt_too_large":0,"frag_overlap":0,"frag_invalid_length":0,"frag_ignored":0,"ipv4_in_ipv6_too_small":0,"ipv4_in_ipv6_wrong_version":0,"ipv6_in_ipv6_too_small":0,"ipv6_in_ipv6_wrong_version":0},"tcp":{"pkt_too_small":0,"hlen_too_small":0,"invalid_optlen":0,"opt_invalid_len":0,"opt_duplicate":0},"udp":{"pkt_too_small":0,"hlen_too_small":0,"hlen_invalid":0,"len_invalid":0},"sll":{"pkt_too_small":0},"ethernet":{"pkt_too_small":0},"ppp":{"pkt_too_small":0,"vju_pkt_too_small":0,"ip4_pkt_too_small":0,"ip6_pkt_too_small":0,"wrong_type":0,"unsup_proto":0},"chdlc":{"pkt_too_small":0}},"too_many_layers":0},"tcp":{"syn":0,"synack":0,"rst":0,"sessions":0,"ssn_memcap_drop":0,"pseudo":0,"pseudo_failed":0,"invalid_checksum":0,"midstream_pickups":0,"pkt_on_wrong_thread":0,"segment_memcap_drop":0,"stream_depth_reached":0,"reassembly_gap":0,"overlap":0,"overlap_diff_data":0,"insert_data_normal_fail":0,"insert_data_overlap_fail":0,"insert_list_fail":0,"memuse":606208,"reassembly_memuse":98304}, "app_layer":{"flow":{"http":0,"ftp":0,"smtp":0,"tls":0,"ssh":0,"imap":0,"smb":0,"dcerpc_tcp":0,"dns_tcp":0,"nfs_tcp":0,"ntp":0,"ftp-data":0,"tftp":0,"ikev2":0,"krb5_tcp":0,"dhcp":0,"snmp":0,"sip":0,"rfb":0,"mqtt":0,"rdp":0,"failed_tcp":0,"dcerpc_udp":0,"dns_udp":0,"nfs_udp":0,"krb5_udp":0,"failed_udp":0},"http":{"memuse":0,"memcap":0},"ftp":{"memuse":0,"memcap":0},"file_store":{"open_files":0}}}
---SNIP---
```

- If we wish to filter out only alert events, for example, we can utilize the [jqlang/jq: Command-line JSON processor](https://github.com/jqlang/jq) command-line JSON processor as follows.
```shell
cat /var/log/suricata/old_eve.json | jq -c 'select(.event_type == "alert")'
```
```json
{"timestamp":"2023-07-06T08:34:35.003163+0000","flow_id":1959965318909019,"in_iface":"ens160","event_type":"alert","src_ip":"10.9.24.101","src_port":51833,"d est_ip":"10.9.24.1","dest_port":53,"proto":"UDP","tx_id":0,"alert":{"action":"allowed","gid":1,"signature_id":1,"rev":0,"signature":"Known bad DNS lookup, possible Dridex infection","category":"","severity":3},"dns":{"query":[{"type":"query","id":6430,"rrname":"adv.epostoday.uk","rrtype":"A","tx_id":0,"opcode":0}    ]},"app_proto":"dns","flow":{"pkts_toserver":1,"pkts_toclient":0,"bytes_toserver":76,"bytes_toclient":0,"start":"2023-07-06T08:34:35.003163+0000"}}
```

- If we wish to identify the earliest DNS event, for example, we can utilize the `jq` command-line JSON processor as follows.
```shell
cat /var/log/suricata/old_eve.json | jq -c 'select(.event_type == "dns")' | head -1 | jq .
```
```json
{
 "timestamp": "2023-07-06T08:34:35.003163+0000",
 "flow_id": 1959965318909019,
 "in_iface": "ens160",
  "event_type": "dns",
  "src_ip": "10.9.24.101",
 "src_port": 51833,
 "dest_ip": "10.9.24.1",
 "dest_port": 53,
 "proto": "UDP",
 "dns": {
  "type": "query",
  "id": 6430,
 "rrname": "adv.epostoday.uk",
 "rrtype": "A",
 "tx_id": 0,
    "opcode": 0
 }
}
```
- Command Explanation:
	- `cat /var/log/suricata/old_eve.json`—Reads the Suricata JSON log file containing recorded network events.  
	- `|`—Pipes the output of one command to the next command.  
	- `jq -c 'select(.event_type == "dns")'`—Filters only DNS-related events from the JSON and outputs each match in compact (single-line) format.  
	 - `|`—Passes the filtered DNS events to the next command.  
	- `head -1`—Selects only the first DNS event from the filtered results.  
	- `|`—Sends that single event to the final command.  
	- `jq .`—Pretty-prints the JSON output in a readable, formatted structure.

- We can also use similar commands to filter for other event types like TLS and SSH.  

- `flow_id`:
	- Suricata assigns a unique `flow_id` to each network flow—a set of IP packets traversing a network interface in a specific direction between source and destination endpoints.
	- This `flow_id` enables tracking and correlating events within the EVE JSON log, associating alerts, network transactions, and packets to provide a cohesive view of a specific communication channel.

- `pcap_cnt`:
	- The `pcap_cnt` counter tracks the processing order of each packet in Suricata, whether from live network traffic or a PCAP file.
	- This:
		- Allows tracing packets back to their original sequence.
		- Aiding in:
			- Understanding the order/sequence of network events as they occurred.
			- Precisely pinpoint when an alert was triggered in relation to other packets.
			- which can provide valuable context in an investigation.

2. `fast.log`—This is a text-based log format that records alerts only and is enabled by default. Try inspecting the content of `old_fast.log` residing at `/var/log/suricata` as follows.
```shell
cat /var/log/suricata/old_fast.log
```
```
07/06/2023-08:34:35.003163  [**] [1:1:0] Known bad DNS lookup, possible Dridex infection [**] [Classification: (null)] [Priority: 3] {UDP} 10.9.24.101:51833 -> 10.9.24.1:53
```

3. `stats.log`—This is a human-readable statistics log, which can be particularly useful while debugging Suricata deployments. Try inspecting the content of `old_stats.log` residing at `/var/log/suricata` as follows.
```shell
cat /var/log/suricata/old_stats.log
```
```
------------------------------------------------------------------------------------
Date: 7/6/2023 -- 08:34:24 (uptime: 0d, 00h 00m 08s)
------------------------------------------------------------------------------------
Counter                                       | TM Name                   | Value
------------------------------------------------------------------------------------
capture.kernel_packets                        | Total                     | 4
decoder.pkts                                  | Total                     | 3
decoder.bytes                                 | Total                     | 212
decoder.ipv6                                  | Total                     | 1
decoder.ethernet                              | Total                     | 3
decoder.icmpv6                                | Total                     | 1
decoder.avg_pkt_size                          | Total                     | 70
decoder.max_pkt_size                          | Total                     | 110
flow.icmpv6                                   | Total                     | 1
flow.wrk.spare_sync_avg                       | Total                     | 100
flow.wrk.spare_sync                           | Total                     | 1
flow.mgr.full_hash_pass                       | Total                     | 1
flow.spare                                    | Total                     | 9900
tcp.memuse                                    | Total                     | 606208
tcp.reassembly_memuse                         | Total                     | 98304
flow.memuse                                   | Total                     | 7394304
------------------------------------------------------------------------------------
---SNIP---
```

- To focus the output strategy
	- Deactivate comprehensive EVE output.
	- Selectively activate specific outputs instead.
- For example, activating `http-log` generates a new http.log file for each Suricata run that encounters HTTP events.

## Hands-on With Suricata Outputs - File Extraction

- Suricata has a powerful but often overlooked/underused feature: [file extraction](https://docs.suricata.io/en/suricata-6.0.13/file-extraction/file-extraction.html).
- This feature lets you capture and save files that are transferred using various protocols.
- This can be very useful for threat hunting, forensics, or simply data analysis.

- To enable file extraction in Suricata, modify the `file-store` section of the `suricata.yaml` configuration file as follows:
	- set `version` to `2`.
	- `enabled` to `yes`.
	- `force-filestore` to `yes`.
```
file-store:
  version: 2
  enabled: yes
  force-filestore: yes
```

- In the same `file-store` section, we define where Suricata stores the extracted files.
- We set the `dir` option to the directory of our choice.

- As a quick exercise, let's enable file extraction and run Suricata on the `vm-2.pcap` file from [www.netresec.com](https://www.netresec.com/?page=PcapFiles).
- As Suricata's documentation specifies, file extraction requires a specific/explicit rule defining when and what files to extract. It is not an automatic process.
- The simplest rule we can add to our `local.rules` file to experiment with file extraction is the following.
```txt
alert http any any -> any any (msg:"FILE store all"; filestore; sid:2; rev:1;)
```

- If we configured Suricata correctly, multiple files will be stored inside the `filestore` directory.

- Run Suricata on the `<path-to-pcap-file>/vm-2.pcap` file.
```shell
suricata -r <path-to-pcap-file>/vm-2.pcap
```

- We will notice that `eve.json`, `fast.log`, `stats.log`, and `suricata.log` were created, alongside a new directory called `filestore`.
- `filestore`'s content in terms of the files it contains can be inspected as follows.
```shell
cd filestore
find . -type f
```
```
./fb/fb20d18d00c806deafe14859052072aecfb9f46be6210acfce80289740f2e20e
./21/214306c98a3483048d6a69eec6bf3b50497363bc2c98ed3cd954203ec52455e5
./21/21742fc621f83041db2e47b0899f5aea6caa00a4b67dbff0aae823e6817c5433
./26/2694f69c4abf2471e09f6263f66eb675a0ca6ce58050647dcdcfebaf69f11ff4
./2c/2ca1a0cd9d8727279f0ba99fd051e1c0acd621448ad4362e1c9fc78700015228
./7d/7d4c00f96f38e0ffd89bc2d69005c4212ef577354cc97d632a09f51b2d37f877
./6b/6b7fee8a4b813b6405361db2e70a4f5a213b34875dd2793667519117d8ca0e4e
./2e/2e2cb2cac099f08bc51abba263d9e3f8ac7176b54039cc30bbd4a45cfa769018
---SNIP---
```
- `find . -type f`—Searches recursively starting from the current directory (`.`) and lists all files (`-type f`), excluding directories and other types like links.

- Suricata's `file-store` module, as documented, saves extracted files in its own log directory (default: `filestore`, within the main logging directory).
- Files are named using the SHA256 hash of their contents and stored in subdirectories `00` to `ff`, based on the first two characters of the SHA256 hash.
- For example, a file with a SHA256 hash starting with `f9bc6d...` would be located in `filestore/f9`.

- If we wanted to inspect, for example, the `/21/21742fc621f83041db2e47b0899f5aea6caa00a4b67dbff0aae823e6817c5433` file inside the `filestore` directory, we could use the `xxd` tool as follows.
```shell
cd filestore
xxd ./21/21742fc621f83041db2e47b0899f5aea6caa00a4b67dbff0aae823e6817c5433 | head
```
```
00000000: 4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
00000010: b800 0000 0000 0000 4000 0000 e907 0000  ........@.......
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 8000 0000  ................
00000040: 0e1f ba0e 00b4 09cd 21b8 014c cd21 5468  ........!..L.!Th
00000050: 6973 2070 726f 6772 616d 2063 616e 6e6f  is program canno
00000060: 7420 6265 2072 756e 2069 6e20 444f 5320  t be run in DOS
00000070: 6d6f 6465 2e0d 0d0a 2400 0000 0000 0000  mode....$.......
00000080: 5045 0000 4c01 0300 fc90 8448 0000 0000  PE..L......H....
00000090: 0000 0000 e000 0f01 0b01 0600 00d0 0000  ................
```

- In this case, the file was a Windows executable based on the file's header. More about the MS-DOS EXE format can be found in following resource [MZ](https://wiki.osdev.org/MZ).

## Live Rule Reloading Feature & Updating Suricata Rulesets

- Suricata's live rule reloading is a crucial feature that enables uninterrupted traffic inspection by updating the ruleset without downtime, ensuring continuous monitoring and minimizing missed malicious activity.

- To enable live rule reloading in Suricata, we need to configure our Suricata configuration file (`suricata.yaml`).
- In the `suricata.yaml` file, we should locate the `detect-engine` section and set the value of the `reload` parameter to `true`. It looks something like this:
```
detect-engine:
  - reload: true
```

- Proceed to execute the following `kill` command, which will signal the Suricata process (determined by `$(pidof suricata)`) to refresh its rule set without necessitating a complete restart.
```shell
sudo kill -usr2 $(pidof suricata)
```

- This modification tells Suricata to:
	- Check for changes in the ruleset periodically.
	- Apply them without needing to restart the service.

- Updating Suricata's ruleset can be performed using the `suricata-update` tool.
- We can perform a simple update to the Suricata ruleset using the following command.
```shell
sudo suricata-update
```
```
6/7/2023 -- 06:46:44 - <Info> -- Using data-directory /var/lib/suricata.
6/7/2023 -- 06:46:44 - <Info> -- Using Suricata configuration /etc/suricata/suricata.yaml
6/7/2023 -- 06:46:44 - <Info> -- Using /etc/suricata/rules for Suricata provided rules.
6/7/2023 -- 06:46:44 - <Info> -- Found Suricata version 6.0.13 at /usr/bin/suricata.
6/7/2023 -- 06:46:44 - <Info> -- Loading /etc/suricata/suricata.yaml
6/7/2023 -- 06:46:44 - <Info> -- Disabling rules for protocol http2
6/7/2023 -- 06:46:44 - <Info> -- Disabling rules for protocol modbus
6/7/2023 -- 06:46:44 - <Info> -- Disabling rules for protocol dnp3
6/7/2023 -- 06:46:44 - <Info> -- Disabling rules for protocol enip
6/7/2023 -- 06:46:44 - <Info> -- No sources configured, will use Emerging Threats Open
6/7/2023 -- 06:46:44 - <Info> -- Fetching https://rules.emergingthreats.net/open/suricata-6.0.13/emerging.rules.tar.gz.
 100% - 3963342/3963342
6/7/2023 -- 06:46:45 - <Info> -- Done.
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/app-layer-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/decoder-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/dhcp-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/dnp3-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/dns-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/stream-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Loading distribution rule file /etc/suricata/rules/tls-events.rules
6/7/2023 -- 06:46:45 - <Info> -- Ignoring file rules/emerging-deleted.rules
6/7/2023 -- 06:46:48 - <Info> -- Loaded 43453 rules.
6/7/2023 -- 06:46:48 - <Info> -- Disabled 14 rules.
6/7/2023 -- 06:46:48 - <Info> -- Creating directory /var/lib/suricata/rules.
6/7/2023 -- 06:46:48 - <Info> -- Backing up current rules.
6/7/2023 -- 06:46:49 - <Info> -- Writing rules to /var/lib/suricata/rules/suricata.rules: total: 43453; enabled: 34465; added: 43453; removed 0; modified: 0
6/7/2023 -- 06:46:49 - <Info> -- Writing /var/lib/suricata/rules/classification.config
6/7/2023 -- 06:46:49 - <Info> -- Testing with suricata -T.
6/7/2023 -- 06:47:11 - <Info> -- Done.
```

- The output shows that `suricata-update` successfully connected to [https://rules.emergingthreats.net/open/](https://rules.emergingthreats.net/open/) and saved the downloaded rules to `/var/lib/suricata/rules/` directory.

- Next, we'll use the following command to list all ruleset providers.
```shell
sudo suricata-update list-sources
```
```
6/7/2023 -- 06:59:29 - <Info> -- Using data-directory /var/lib/suricata.
6/7/2023 -- 06:59:29 - <Info> -- Using Suricata configuration /etc/suricata/suricata.yaml
6/7/2023 -- 06:59:29 - <Info> -- Using /etc/suricata/rules for Suricata provided rules.
6/7/2023 -- 06:59:29 - <Info> -- Found Suricata version 6.0.13 at /usr/bin/suricata.
6/7/2023 -- 06:59:29 - <Info> -- No source index found, running update-sources
6/7/2023 -- 06:59:29 - <Info> -- Downloading https://www.openinfosecfoundation.org/rules/index.yaml
6/7/2023 -- 06:59:29 - <Info> -- Adding all sources
6/7/2023 -- 06:59:29 - <Info> -- Saved /var/lib/suricata/update/cache/index.yaml
Name: et/open
  Vendor: Proofpoint
  Summary: Emerging Threats Open Ruleset
  License: MIT
Name: et/pro
  Vendor: Proofpoint
  Summary: Emerging Threats Pro Ruleset
  License: Commercial
  Replaces: et/open
  Parameters: secret-code
  Subscription: https://www.proofpoint.com/us/threat-insight/et-pro-ruleset
Name: oisf/trafficid
  Vendor: OISF
  Summary: Suricata Traffic ID ruleset
  License: MIT
Name: scwx/enhanced
  Vendor: Secureworks
  Summary: Secureworks suricata-enhanced ruleset
  License: Commercial
  Parameters: secret-code
  Subscription: https://www.secureworks.com/contact/ (Please reference CTU Countermeasures)
Name: scwx/malware
  Vendor: Secureworks
  Summary: Secureworks suricata-malware ruleset
  License: Commercial
  Parameters: secret-code
  Subscription: https://www.secureworks.com/contact/ (Please reference CTU Countermeasures)
Name: scwx/security
  Vendor: Secureworks
  Summary: Secureworks suricata-security ruleset
  License: Commercial
  Parameters: secret-code
  Subscription: https://www.secureworks.com/contact/ (Please reference CTU Countermeasures)
Name: sslbl/ssl-fp-blacklist
  Vendor: Abuse.ch
  Summary: Abuse.ch SSL Blacklist
  License: Non-Commercial
Name: sslbl/ja3-fingerprints
  Vendor: Abuse.ch
  Summary: Abuse.ch Suricata JA3 Fingerprint Ruleset
  License: Non-Commercial
Name: etnetera/aggressive
  Vendor: Etnetera a.s.
  Summary: Etnetera aggressive IP blacklist
  License: MIT
Name: tgreen/hunting
  Vendor: tgreen
  Summary: Threat hunting rules
  License: GPLv3
Name: malsilo/win-malware
  Vendor: malsilo
  Summary: Commodity malware rules
  License: MIT
Name: stamus/lateral
  Vendor: Stamus Networks
  Summary: Lateral movement rules
  License: GPL-3.0-only
```

- To show how Suricata fetches rulesets, we'll use the `et/open` rulesets as an example.

- Now, we'll run a command to retrieve and enable these `et/open` rulesets in Suricata.
```shell
sudo suricata-update enable-source et/open
```
```
6/7/2023 -- 07:02:08 - <Info> -- Using data-directory /var/lib/suricata.
6/7/2023 -- 07:02:08 - <Info> -- Using Suricata configuration /etc/suricata/suricata.yaml
6/7/2023 -- 07:02:08 - <Info> -- Using /etc/suricata/rules for Suricata provided rules.
6/7/2023 -- 07:02:08 - <Info> -- Found Suricata version 6.0.13 at /usr/bin/suricata.
6/7/2023 -- 07:02:08 - <Info> -- Creating directory /var/lib/suricata/update/sources
6/7/2023 -- 07:02:08 - <Info> -- Source et/open enabled
```

- Lastly, let's reissue the `suricata-update` command to load the newly acquired ruleset.
```shell
sudo suricata-update
```

- A Suricata service restart may also be required.
```shell
sudo systemctl restart suricata
```

## Validating Suricata's Configuration

- Validation of Suricata's configuration is also an essential part of maintaining the robustness of our IDS/IPS setup.
- To validate the configuration, we can use the `-T` option provided by the Suricata command.
- This command runs a test to check if:
	- The configuration file is valid.
	- And All files referenced in the configuration are accessible.

- Here is how we can do this.
```shell
sudo suricata -T -c /etc/suricata/suricata.yaml
```
```
6/7/2023 -- 07:13:29 - <Info> - Running suricata under test mode
6/7/2023 -- 07:13:29 - <Notice> - This is Suricata version 6.0.13 RELEASE running in SYSTEM mode
6/7/2023 -- 07:13:29 - <Notice> - Configuration provided was successfully loaded. Exiting.
```

## Suricata Documentation

- Suricata is an incredibly versatile tool with extensive functionality.
- We highly recommend exploring [Suricata's documentation](https://docs.suricata.io/) to gain a deeper understanding of its capabilities.

## Suricata Key Features

Key features that bolster Suricata's effectiveness include:

- Deep packet inspection and packet capture logging
- Anomaly detection and Network Security Monitoring
- Intrusion Detection and Prevention, with a hybrid mode available
- Lua scripting
- Geographic IP identification (GeoIP)
- Full IPv4 and IPv6 support
- IP reputation
- File extraction
- Advanced protocol inspection
- Multitenancy

- **Note**:
	- Suricata can also be used to detect "non-standard/anomalous" traffic.
	- We can leverage strategies outlined in Suricata's [Protocol Anomalies Detection page](https://redmine.openinfosecfoundation.org/projects/suricata/wiki/Protocol_Anomalies_Detection).
	- This approach enhances our visibility into unusual or non-compliant behavior within our network, thus augmenting our security posture.

---

# Suricata Rule Development Part 1

- Suricata rules serve as directives, instructing the engine to watch for specific markers in network traffic and generate notifications upon detection.
- These rules are not limited to identifying malicious activity; they also provide network defenders with valuable insights into network events. Rule specificity can be adjusted, requiring a balance to accurately identify threats, such as malware variants, while minimizing false positives.
- Rules are often based on cybersecurity community intelligence, but because each rule consumes processing power and memory, Suricata offers guidelines for effective rule writing.

## Suricata Rule Anatomy

- A sample Suricata rule.
```shell
action protocol from_ip port -> to_ip port (msg:"Known malicious behavior, possible X malware infection"; content:"some thing"; content:"some other thing"; sid:10000001; rev:1;)
```

#### Header (`action protocol from_ip port -> to_ip port` part)

- The header of a rule specifies/encapsulates:
	- What the rule does/ the intended action of the rule and the protocol it applies to.
	- It also includes:
		- IP addresses information (`src` & `dst`).
		- Port numbers information  (`src` & `dst`).
		- An arrow showing the direction of network traffic (`->` or `<-` or `<>`).

- `action` instructs Suricata on what steps to take if the contents match.
- This could range from:
	- Generating an alert (`alert`).
	- Logging the traffic without an alert (`log`).
	- Ignoring the packet (`pass`).
	- Dropping the packet in IPS mode (`drop`).
	- Sending TCP RST packets (`reject`).

- `protocol` can vary, including `tcp`, `udp`, `icmp`, `ip`, `http`, `tls`, `smb`, or `dns`.

- Traffic directionality is declared using:
	- `rule host variables` (such as `$HOME_NET`, `$EXTERNAL_NET`, etc. that we saw inside `suricata.yaml`).
	- `rule direction`.
- The direction arrow between the two IP-Port pairs informs Suricata about the traffic flow. Examples:
	- Outbound: `$HOME_NET any -> $EXTERNAL_NET 9090`
	- Inbound: `$EXTERNAL_NET any -> $HOME_NET 8443`
	- Bidirectional: `$EXTERNAL_NET any <> $HOME_NET any`

- `Rule ports` define the ports at which the traffic for this rule will be evaluated. Examples:
	- `alert tcp $HOME_NET any -> $EXTERNAL_NET 9443`
	- `alert tcp $HOME_NET any -> $EXTERNAL_NET $UNCOMMON_PORTS`
		- $UNCOMMON_PORTS can be defined inside `suricata.yaml`
	- `alert tcp $HOME_NET any -> $EXTERNAL_NET [8443,8080,7001:7002,!8443]`

#### Rule message & content (`(msg:"Known malicious behavior, possible X malware infection"; content:"some thing"; content:"some other thing";` part)

- The `rule message & content` section specifies the message shown to analysts when a specific activity is detected.
- `Content` refers to the parts/segments of the network traffic that are important/essential for identifying these activities/detections.

- `Rule message (msg)`:
	- It's an arbitrary text displayed when the rule is triggered.
	- Ideally, the rule messages we create should contain details about malware architecture, family, and action.

- `flow`:
	- Identifies the originator and responder.
	- Always remember, when crafting rules, to have the engine monitor `established` TCP sessions.
	- ***Connection Direction (`to_server` / `from_client`)***—These keywords direct Suricata to inspect only traffic originating from the request initiator, ignoring server responses.
	- ***Response Direction (`to_client` / `from_server`)***—These keywords focus on traffic from the responder to the requester, allowing inspection of server responses like malicious files or command and control instructions.
	- ***The `Established` State***—The `established` keyword in Suricata ensures inspection begins only after a successful TCP three-way handshake, preventing false positives by verifying connection establishment.
	- You can combine both Connection/Response Direction with the Established State.

- `dsize`:
	- Matches using the payload size of the packet.
	- It relies on TCP segment length, not the total packet length.
    - Example: `alert http any any -> any any (msg:"Large HTTP response"; dsize:>10000; content:"HTTP/1.1 200 OK"; sid:2003;)`

- `Rule content` comprises unique values that help identify specific network traffic or activities. Suricata matches these unique content values in packets for detection.
	- Example: `content:"User-Agent|3a 20|Go-http-client/1.1|0d 0a|Accept-Encoding|3a 20|gzip";`
		- `|3a 20|`—This is the hexadecimal code for a colon (:) followed by a space. It's used to find that specific sequence of characters within the data of a packet or packet payload.
		- `|0d 0a|`—This represents the hexadecimal representation of the characters `\r\n`, which signifies the end of a line in HTTP headers.

- `Rule Buffers`:
	- Improve efficiency by avoiding full packet searches for content matches, saving time and resources.
	- For more information, visit [https://suricata.readthedocs.io/en/latest/rules/http-keywords.html](https://suricata.readthedocs.io/en/latest/rules/http-keywords.html)
	- Example: `alert http any any -> any any (http.accept; content:"image/gif"; sid:1;)`
		- `http.accept`—This sticky buffer is used to match the HTTP Accept header, containing only the header value. The carriage return and newline characters (`\r\n`) following the header are not included in this buffer.

- `Rule options` act as additional modifiers to aid detection, helping Suricata locate the exact location of contents.
	- `nocase` ensures rules are not bypassed through case changes.
	    - Example: `alert tcp any any -> any any (msg:"Detect HTTP traffic with user agent Mozilla"; content:"User-Agent: Mozilla"; nocase; sid:8001;)`
	- `offset` informs Suricata about the start position inside the packet for matching.
	    - Example: `alert tcp any any -> any any (msg:"Detect specific protocol command"; content:"|01 02 03|"; offset:0; depth:5; sid:3003;)`
	        - This rule triggers an alert when Suricata detects a specific protocol command represented by the byte sequence `|01 02 03|` in the TCP payload. The `offset:0` keyword sets the content match to start from the beginning of the payload, and `depth:5` specifies a length of five bytes to be considered for matching.

- `distance`:
	- Specifies the minimum number of bytes between the end of the previous content match and the start of the next, and is only applicable when multiple `content` keywords are used in a rule.
	- It's often paired with `within` to define a search window.
	- For example: `alert http any any -> any any (msg:"Detect suspicious URL path after HTTP method"; content:"GET"; offset:0; depth:4; content:"/admin"; distance:10; within:80; sid:3001;)`.
	- In this rule, Suricata first matches `GET` within the first four bytes.
	- `distance:10` then requires at least 10 bytes before the search for `/admin` begins, while `within:80` limits this search to the subsequent 80 bytes.
	- Thus, `/admin` must start 10-80 bytes after `GET` to trigger the rule.

#### Rule metadata (`sid:10000001; rev:1;` part)

- `Reference`—points us to the original source of information behind a rule.
- `SID` (The signature ID)—is a unique number that helps rule writers organize and differentiate their rules.
- `revision`—indicates the rule's version, showing how it has changed and improved over time.

#### PCRE

##### 1. What is PCRE?

- PCRE (Perl Compatible Regular Expressions) is a powerful language used to find complex patterns in text that standard keyword searches can’t handle.
- In Suricata, it acts as a "fine-tooth comb" to verify specific data formats after general filters have already matched.

##### 2. Basic Syntax Rules

- A PCRE expression is always enclosed in **forward slashes** (`/pattern/`), with **flags** (like `i` for case-insensitive) placed immediately after the closing slash.
- Specialized characters (like `$` or `.`) must be "escaped" with a backslash (`\`) if you want to search for the literal character itself.

##### 3. Using Anchors

- Anchors like `^` (start of line) and `$` (end of line) tell the engine exactly where the pattern must appear.
- These are written inside the slashes to lock the search to a specific boundary, preventing matches in the middle of unrelated data.

##### 4. The `!` (Negation) Operator

- Placing an exclamation mark **before** the PCRE string (`!"/pattern/"`) tells Suricata to trigger the rule only if the pattern is **not** found.
- This is useful for identifying "abnormal" traffic that deviates from a known safe format.

##### 5. Essential Flags: `i` and `RP`

- The `i` flag makes the search ignore case differences.
- The `RP` (Relative Postprocessor) flag tells Suricata to begin searching with the PCRE (Perl Compatible Regular Expressions) starting from the exact end position of the previous `content` match. This makes sure the rule checks the data in the correct order.

##### 6. Efficiency Best Practices

- You should **never** use PCRE alone; always pair it with a `content` keyword first.
- Suricata uses fast hardware-based matching for `content` to quickly discard safe traffic, only using the slower, more "expensive" PCRE engine when a potential threat is found.

- For those who seek to broaden their understanding of Suricata rules and delve deeper into rule development, the following resource serves as a comprehensive guide: [https://docs.suricata.io/en/latest/rules/index.html](https://docs.suricata.io/en/latest/rules/index.html).

## IDS/IPS Rule Development Approaches

- When it comes to creating rules for Intrusion Detection Systems (IDS) and Intrusion Prevention Systems (IPS), there's an art and a science behind it.
- It requires a comprehensive understanding of:
	- Network Protocols.
	- Malware Behaviors.
	- System Vulnerabilities.
	- The threat landscape in general.

- Signature-based detection, a traditional IDS/IPS method, identifies malware by matching unique network traffic elements to known signatures.
- These signatures can range from simple packet payload patterns, like specific commands or strings, to complex patterns involving multiple packets or characteristics.
- While effective against known threats, this approach is limited in detecting novel malware lacking established signatures.

- Alternatively, ***Anomaly- or Behavior-Based Detection*** identifies malware by characteristic actions, such as consistent HTTP response sizes or beaconing intervals.
- Other indicators include high data transfer volumes and unusual port usage.
- This approach can detect zero-day attacks and novel threats missed by signature-based methods, but it often suffers from higher false-positive rates due to the variability of network activity.

- ***Stateful Protocol Analysis***, another method for crafting IDS/IPS rules:
	- Tracks network protocol states.
	- Compares observed behaviors to expected state transitions.
- This allows identification of deviations indicative of malicious activity.

## Suricata Rule Development Example 1: Detecting PowerShell Empire

```shell
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"ET MALWARE Possible PowerShell Empire Activity Outbound"; flow:established,to_server; content:"GET"; http_method; content:"/"; http_uri; depth:1; pcre:"/^(?:login\/process|admin\/get|news)\.php$/RU"; content:"session="; http_cookie; pcre:"/^(?:[A-Z0-9+/]{4})*(?:[A-Z0-9+/]{2}==|[A-Z0-9+/]{3}=|[A-Z0-9+/]{4})$/CRi"; content:"Mozilla|2f|5.0|20 28|Windows|20|NT|20|6.1"; http_user_agent; http_start; content:".php|20|HTTP|2f|1.1|0d 0a|Cookie|3a 20|session="; fast_pattern; http_header_names; content:!"Referer"; content:!"Cache"; content:!"Accept"; sid:2027512; rev:1;)
```

- The Suricata rule above is designed to detect possible outbound activity from [PowerShell Empire](https://github.com/EmpireProject/Empire), a common post-exploitation framework used by attackers.
- Let's break down the important parts of this rule to understand its workings.

#### Rule Explanation

```
alert http $HOME_NET any -> $EXTERNAL_NET any
```
- **Action**—`alert` (Generate an alert and log the packet).
- **Protocol**—`http` (Specifically inspects web traffic).
- **Direction**—From your internal network (`$HOME_NET`) to the outside world (`$EXTERNAL_NET`). This signifies **outbound C2 traffic** (a compromised machine "calling home").

```
flow:established,to_server; content:"GET"; http_method;
```
- **Flow**—The rule only triggers on an active, established connection moving toward the server.
- **Method**—It looks specifically for an HTTP **GET** request, which Empire agents use to check for new tasks from the attacker.

```
content:"/"; http_uri; depth:1; pcre:"/^(?:login\/process|admin\/get|news)\.php$/RU";
```
- It looks for common default paths used by PowerShell Empire:
	- `/login/process.php`
	- `/admin/get.php`
	- `/news.php`
- The **PCRE** (Perl Compatible Regular Expression) ensures the URI matches one of these three patterns exactly.

```
content:"Mozilla|2f|5.0|20 28|Windows|20|NT|20|6.1"; http_user_agent;
```
- The rule looks for a specific **User-Agent** string.
	- The hex code `|20|` represents a space, and `|2f|` is a forward slash.
	- Translated, it looks for: `Mozilla/5.0 (Windows NT 6.1`.
	- **Note**—This is an older User-Agent (Windows 7). Attackers often forget to update these defaults, making them easy to spot.

```
content:!".php|20|HTTP|2f|1.1|0d 0a|Cookie|3a 20|session="; fast_pattern;` `content:!"Referer"; content:!"Cache"; content:!"Accept";
```
- This section looks for things that **should not** be there or look "weird":
	- **Strict Header Order**—It checks for a very specific sequence of characters in the HTTP header.
	- **Missing Headers (`!`)**—Real web browsers almost always send a `Referer`, `Cache-Control`, or `Accept` header. Empire’s default traffic often omits these to save space, which is a major "red flag" for defenders.

- Useful Resources:
	- [https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78](https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78)
    - [https://www.keysight.com/blogs/tech/nwvs/2021/06/16/empire-c2-networking-into-the-dark-side](https://www.keysight.com/blogs/tech/nwvs/2021/06/16/empire-c2-networking-into-the-dark-side)
    - [https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78](https://github.com/EmpireProject/Empire/blob/master/data/agent/agent.ps1#L78)

- This rule alerts on outbound HTTP GET requests containing a specific pattern in the URI, cookie, or user-agent, excluding certain headers.
```shell
sudo suricata -r /home/htb-student/pcaps/psempire.pcap -l . -k none
```
```
15/7/2023 -- 03:57:42 - <Notice> - This is Suricata version 4.0.0-beta1 RELEASE
15/7/2023 -- 03:57:42 - <Notice> - all 5 packet processing threads, 4 management threads initialized, engine started.
15/7/2023 -- 03:57:42 - <Notice> - Signal Received.  Stopping engine.
15/7/2023 -- 03:57:42 - <Notice> - Pcap-file module read 511 packets, 101523 bytes
```

```shell
cat fast.log
```
```
11/21/2017-05:04:53.950737  [**] [1:2027512:1] ET MALWARE Possible PowerShell Empire Activity Outbound [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.56.14:50447 -> 51.15.197.127:80
11/21/2017-05:04:01.308390  [**] [1:2027512:1] ET MALWARE Possible PowerShell Empire Activity Outbound [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.56.14:50436 -> 51.15.197.127:80
11/21/2017-05:05:20.249515  [**] [1:2027512:1] ET MALWARE Possible PowerShell Empire Activity Outbound [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.56.14:50452 -> 51.15.197.127:80
11/21/2017-05:05:56.849190  [**] [1:2027512:1] ET MALWARE Possible PowerShell Empire Activity Outbound [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.56.14:50459 -> 51.15.197.127:80
11/21/2017-05:06:02.062235  [**] [1:2027512:1] ET MALWARE Possible PowerShell Empire Activity Outbound [**] [Classification: (null)]
---SNIP---
```

## Suricata Rule Development Example 2: Detecting Covenant

```shell
alert tcp any any -> $HOME_NET any (msg:"detected by body"; content:"<title>Hello World!</title>"; detection_filter: track by_src, count 4 , seconds 10; priority:1; sid:3000011;)
```

- **Rule source**: [Signature-based IDS for Encrypted C2 Traffic Detection - Eduardo Macedo](https://repositorio-aberto.up.pt/bitstream/10216/142718/2/572020.pdf)

- The (inefficient) Suricata rule above is designed to detect certain variations of [Covenant](https://github.com/cobbr/Covenant), another common post-exploitation framework used by attackers.
- `Covenant` is an open-source Command and Control (C2) framework. Its underpinnings can be explored via the following repository.[https://github.com/cobbr/Covenant/blob/master/Covenant/Data/Profiles/DefaultHttpProfile.yaml#L35](https://github.com/cobbr/Covenant/blob/master/Covenant/Data/Profiles/DefaultHttpProfile.yaml#L35)

#### Rule Explanation

```
alert tcp any any -> $HOME_NET any
```
- **`alert`**—The action. Suricata will generate a log entry when triggered.
- **`tcp`**—It is looking specifically at TCP traffic (web traffic, SSH, etc.).
- **`any any -> $HOME_NET any`**—It watches traffic coming from **any** IP and **any** port, destined for your defined **home network** on **any** port.

```
(msg:"detected by body"; content:"<title>Hello World!</title>"; ...)
```
- **`msg`**—The label that will appear in your logs.
- **`content`**—The "fingerprint". Suricata is scanning the raw TCP data for the specific string `<title>Hello World!</title>`. This indicates the rule is looking for a specific HTML page title.

```
detection_filter: track by_src, count 4, seconds 10;
```
- **`track by_src`**—It remembers the IP address of the sender.
- **`count 4`** & **`seconds 10`**—The alert only triggers once the sender has sent that specific content **4 times within a 10-second window**.
- This is often used to detect **brute-force attacks** or **automated scanners**.

```
priority:1; sid:3000011;
```
- **`priority:1`**—This marks it as high importance (1 is highest, 4 is lowest).
- **`sid:3000011`**—The **Signature ID**. This is a unique serial number for the rule. IDs in the 3,000,000 range are typically reserved for custom, local rules.

- A high-priority alert triggers when four or more TCP connections originate from the same source IP to an internal host within 10 seconds, each containing the payload `<title>Hello World!</title>`.
```shell
sudo suricata -r /home/htb-student/pcaps/covenant.pcap -l . -k none
```
```
15/7/2023 -- 04:47:15 - <Notice> - This is Suricata version 4.0.0-beta1 RELEASE
15/7/2023 -- 04:47:15 - <Notice> - all 5 packet processing threads, 4 management threads initialized, engine started.
15/7/2023 -- 04:47:16 - <Notice> - Signal Received.  Stopping engine.
15/7/2023 -- 04:47:16 - <Notice> - Pcap-file module read 27384 packets, 3125549 bytes
```

```shell
cat fast.log
```
```
01/21/2021-06:38:51.250048  [**] [1:3000011:0] detected by body [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:50366
01/21/2021-06:40:55.021993  [**] [1:3000011:0] detected by body [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:50375
01/21/2021-06:36:21.280144  [**] [1:3000011:0] detected by body [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:50358
01/21/2021-06:41:53.395248  [**] [1:3000011:0] detected by body [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:50378
01/21/2021-06:42:21.582624  [**] [1:3000011:0] detected by body [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:50379
01/21/2021-06:41:25.215525  [**] [1:3000011:0] detected by body [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:50377
---SNIP---
```

- Invest some time in scrutinizing [https://github.com/cobbr/Covenant/blob/master/Covenant/Data/Profiles/DefaultHttpProfile.yaml](https://github.com/cobbr/Covenant/blob/master/Covenant/Data/Profiles/DefaultHttpProfile.yaml), the `covenant.pcap` file using `Wireshark`, and these newly found rule to comprehend how they work. These rules may yield false-positive results, and hence for optimal performance, it's advisable to integrate them with other detection rules.

## Suricata Rule Development Example 3: Detecting Covenant (Using Analytics)

```shell
alert tcp $HOME_NET any -> any any (msg:"detected by size and counter"; dsize:312; detection_filter: track by_src, count 3 , seconds 10; priority:1; sid:3000001;)
```

- The important parts of this rule to understand its workings.
	- `dsize:312;`: This instructs Suricata to look for TCP traffic with a data payload size of exactly 312 bytes.
	- `detection_filter: track by_src, count 3 , seconds 10;`: it will only trigger an alert if it detects the same rule hit at least 3 times (`count 3`) within a 10-second window (`seconds 10`).

- Run Suricata on `/home/htb-student/pcaps/covenant.pcap`.
```shell
sudo suricata -r <path-to-pcap-file>/covenant.pcap -l . -k none
```
```
15/7/2023 -- 05:29:19 - <Notice> - This is Suricata version 4.0.0-beta1 RELEASE
15/7/2023 -- 05:29:19 - <Notice> - all 5 packet processing threads, 4 management threads initialized, engine started.
15/7/2023 -- 05:29:20 - <Notice> - Signal Received.  Stopping engine.
15/7/2023 -- 05:29:20 - <Notice> - Pcap-file module read 27384 packets, 3125549 bytes
```

```shell
cat fast.log
```
```
01/21/2021-06:45:21.609212  [**] [1:3000001:0] detected by size and counter [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:
50386
01/21/2021-06:48:49.965761  [**] [1:3000001:0] detected by size and counter [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:
50395
01/21/2021-06:42:49.682887  [**] [1:3000001:0] detected by size and counter [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:
50380
01/21/2021-06:49:20.143398  [**] [1:3000001:0] detected by size and counter [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:
50396
01/21/2021-06:50:49.706170  [**] [1:3000001:0] detected by size and counter [**] [Classification: (null)] [Priority: 1] {TCP} 157.230.93.100:80 -> 10.0.0.61:
50400
---SNIP---
```

## Suricata Rule Development Example 4: Detecting Sliver

```shell
alert tcp any any -> any any (msg:"Sliver C2 Implant Detected"; content:"POST"; pcre:"/\/(php|api|upload|actions|rest|v1|oauth2callback|authenticate|oauth2|oauth|auth|database|db|namespaces)(.*?)((login|signin|api|samples|rpc|index|admin|register|sign-up)\.php)\?[a-z_]{1,2}=[a-z0-9]{1,10}/i"; sid:1000007; rev:1;)
```

- **Rule source**: [https://www.bilibili.com/read/cv19510951/](https://www.bilibili.com/read/cv19510951/)

The Suricata rule above is designed to detect certain variations of [Sliver](https://github.com/BishopFox/sliver), yet another common post-exploitation framework used by attackers.

- `content:"POST";`: This option instructs Suricata to look for TCP traffic containing the string "POST".
- `Sliver` is an open-source Command and Control (C2) framework. Its underpinnings can be explored via the following repository.[https://github.com/BishopFox/sliver/blob/master/server/configs/http-c2.go#L294](https://github.com/BishopFox/sliver/blob/master/server/configs/http-c2.go#L294)

#### Rule Explanation

```
alert tcp any any -> any any
```
- The rule monitors **all TCP traffic** moving in **any direction** (inbound or outbound).
- Since Sliver implants (the "infected" machines) usually communicate with a C2 server over the internet, this ensures no matter where the server is hosted, the traffic is inspected.

```
pcre:"/\/(php|api|upload|actions|rest|v1|oauth2callback|authenticate|oauth2|oauth|auth|database|db|namespaces)(.*?)((login|signin|api|samples|rpc|index|admin|register|sign-up)\.php)\?[a-z_]{1,2}=[a-z0-9]{1,10}/i";
```
- It looks for common-looking API paths like `/php`, `/api`, `/rest`, or `/db`.
- It looks for a standard-looking script name like `login.php`, `index.php`, or `admin.php`.
- It looks for a very specific, short parameter at the end, like `?a=x1y2z3`.
    - `[a-z_]{1,2}`: A 1 or 2 letter variable name.
    - `[a-z0-9]{1,10}`: A random-looking string of letters and numbers up to 10 characters long.
- **The `/i` flag:** This makes the search **case-insensitive**.

```shell
sudo suricata -r /home/htb-student/pcaps/sliver.pcap -l . -k none
```
```
16/7/2023 -- 02:27:50 - <Notice> - This is Suricata version 4.0.0-beta1 RELEASE
16/7/2023 -- 02:27:50 - <Notice> - all 5 packet processing threads, 4 management threads initialized, engine started.
16/7/2023 -- 02:27:50 - <Notice> - Signal Received.  Stopping engine.
16/7/2023 -- 02:27:50 - <Notice> - Pcap-file module read 36 packets, 18851 bytes
```

```shell
cat fast.log
```
```
01/23/2023-15:14:46.988537  [**] [1:1000002:1] Sliver C2 Implant Detected - POST [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.4.90:50681 -> 192.168.4.85:80
01/23/2023-15:14:47.321224  [**] [1:1000002:1] Sliver C2 Implant Detected - POST [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.4.90:50684 -> 192.168.4.85:80
01/23/2023-15:14:48.074797  [**] [1:1000002:1] Sliver C2 Implant Detected - POST [**] [Classification: (null)] [Priority: 3] {TCP} 192.168.4.90:50687 -> 192.168.4.85:80
```

- Another rule for detecting Silver
```
alert tcp any any -> any any (msg:"Sliver C2 Implant Detected - Cookie"; content:"Set-Cookie"; pcre:"/(PHPSESSID|SID|SSID|APISID|csrf-state|AWSALBCORS)\=[a-z0-9]{32}\;/"; sid:1000003; rev:1;)
```

- Let's break down the important parts of this rule to understand its workings.
	- `content:"Set-Cookie";`—This option instructs Suricata to look for TCP traffic containing the string .`Set-Cookie`
	- `pcre:"/(PHPSESSID|SID|SSID|APISID|csrf-state|AWSALBCORS)\=[a-z0-9]{32}\;/";`—This is a regular expression used to identify specific cookie-setting patterns in the traffic. It matches the header when it's setting specific cookie names (PHPSESSID, SID, SSID, APISID, csrf-state, AWSALBCORS) with a value that's a 32-character alphanumeric string.`Set-Cookie`

---

# Suricata Rule Development Part 2 (Encrypted Traffic)

- Encrypted traffic presents a key challenge in network security, hindering (pose significant obstacles):
	- Effective traffic analysis.
	- The development of reliable IDS/IPS rules.

- To detect security threats, we can analyze SSL/TLS certificates and JA3 fingerprints.

- SSL/TLS certificates are exchanged during the initial handshake of an SSL/TLS secure connection.
- These certificates contain unencrypted information such as:
	- Who issued the certificate.
	- When it was issued.
	- When it expires.
	- the subject (containing information about who the certificate is for and the domain name).
- Suspicious or malicious domains might utilize SSL/TLS certificates with anomalous or unique characteristics.
- Recognizing these anomalies in SSL/TLS certificates can be a stepping stone to crafting effective Suricata rules.

- Further, we can also utilize the [JA3](https://github.com/salesforce/ja3) hash:
	- ***JA3*** is a fingerprinting technique that creates a unique identifier for each SSL/TLS client.
	- The JA3 Hash works by combining details from the client hello packet exchanged during the SSL/TLS handshake.
	- This combination generates a hash value, called the JA3 hash, which can be unique to specific malware families or suspicious software.
	- These JA3 hashes are useful for creating/formulating detection rules for encrypted network traffic.

## Suricata Rule Development Example 5: Detecting Dridex (TLS Encrypted)

```
alert tls $EXTERNAL_NET any -> $HOME_NET any (msg:"ET MALWARE ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex)"; flow:established,from_server; content:"|16|"; content:"|0b|"; within:8; byte_test:3,<,1200,0,relative; content:"|03 02 01 02 02 09 00|"; fast_pattern; content:"|30 09 06 03 55 04 06 13 02|"; distance:0; pcre:"/^[A-Z]{2}/R"; content:"|55 04 07|"; distance:0; content:"|55 04 0a|"; distance:0; pcre:"/^.{2}[A-Z][a-z]{3,}\s(?:[A-Z][a-z]{3,}\s)?(?:[A-Z](?:[A-Za-z]{0,4}?[A-Z]|(?:\.[A-Za-z]){1,3})|[A-Z]?[a-z]+|[a-z](?:\.[A-Za-z]){1,3})\.?[01]/Rs"; content:"|55 04 03|"; distance:0; byte_test:1,>,13,1,relative; content:!"www."; distance:2; within:4; pcre:"/^.{2}(?P<CN>(?:(?:\d?[A-Z]?|[A-Z]?\d?)(?:[a-z]{3,20}|[a-z]{3,6}[0-9_][a-z]{3,6})\.){0,2}?(?:\d?[A-Z]?|[A-Z]?\d?)[a-z]{3,}(?:[0-9_-][a-z]{3,})?\.(?!com|org|net|tv)[a-z]{2,9})[01].*?(?P=CN)[01]/Rs"; content:!"|2a 86 48 86 f7 0d 01 09 01|"; content:!"GoDaddy"; sid:2023476; rev:5;)
```

### Rule Explanation

```
alert tls $EXTERNAL_NET any -> $HOME_NET any
```
- It's watching an external server responding/reaching to one of your internal machines.

```
flow:established,from_server
```
- The rule only triggers after a connection is made and the server is sending its "Certificate" packet.

```
content:"|16|"; content:"|0b|"; within:8; byte_test:3,<,1200,0,relative;
```
- `|16|`—This is the hex code for a TLS "Handshake" record.
- `|0b|`—This is the hex code for the "Certificate" message.
- `byte_test`—It checks if the certificate is smaller than 1200 bytes. Real certificates (like Google's) are usually much larger because they have long chains of trust. Malicious ones are often small and "self-signed."

```
content:"|30 09 06 03 55 04 06 13 02|"; distance:0; pcre:"/^[A-Z]{2}/R";
```
- This looks for the **Country Code** field in the certificate.
- `pcre:"/^[A-Z]{2}/R"`—It ensures there are exactly two uppercase letters (e.g., US, UK, RU).

```
pcre:"/^.{2}[A-Z][a-z]{3,}\s.../Rs"
```
```
pcre:"/^.{2}(?P<CN>...)\.(?!com|org|net|tv)[a-z]{2,9})[01].*?(?P=CN)[01]/Rs"
```

- **Pseudo-Randomness**—Dridex certificates use a specific formula to generate fake names (e.g., `John Doe LLC` or `RandomService.xyz`). The regex checks for specific patterns of capitalization and word length that humans don't usually use.
- Non-Standard TLDs—It explicitly looks for domains that **do NOT** end in `.com`, `.org`, `.net`, or `.tv`.

```
content:!"www."; content:!"GoDaddy";
```
- `!` (NOT)—If the certificate contains "www." or was issued by "GoDaddy," the rule stops.
- This prevents alerting on legitimate, popular websites that might accidentally share a minor characteristic with the malware.

- Give this [resource on Dridex SSL certificates](https://unit42.paloaltonetworks.com/wireshark-tutorial-dridex-infection-traffic/) a look.

- The mentioned OIDs (Object Identifiers) are part of the X.509 standard for PKI and are used to uniquely identify the types of fields contained within certificates.

- Execute Suricata on the file:
```shell
sudo suricata -r <path-to-pcap-file>/dridex.pcap -l . -k none
```
```
15/7/2023 -- 20:34:11 - <Notice> - This is Suricata version 6.0.13 RELEASE running in USER mode
15/7/2023 -- 20:34:11 - <Notice> - all 3 packet processing threads, 4 management threads initialized, engine started.
15/7/2023 -- 20:34:11 - <Notice> - Signal Received.  Stopping engine.
15/7/2023 -- 20:34:11 - <Notice> - Pcap-file module read 1 files, 3683 packets, 3276706 bytes
```

```shell
cat fast.log
```
```
07/09/2019-18:26:31.480302  [**] [1:2023476:5] ET MALWARE ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex) [**] [Classification: (null)] [P             riority: 3] {TCP} 188.166.156.241:443 -> 10.7.9.101:49206
07/09/2019-18:26:33.937036  [**] [1:2023476:5] ET MALWARE ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex) [**] [Classification: (null)] [P             riority: 3] {TCP} 188.166.156.241:443 -> 10.7.9.101:49207
07/09/2019-18:26:39.373287  [**] [1:2023476:5] ET MALWARE ABUSE.CH SSL Blacklist Malicious SSL certificate detected (Dridex) [**] [Classification: (null)] [P             riority: 3] {TCP} 188.166.156.241:443 -> 10.7.9.101:49208
---SNIP---
```

## Suricata Rule Development Example 6: Detecting Sliver (TLS Encrypted)

```
alert tls any any -> any any (msg:"Sliver C2 SSL"; ja3.hash; content:"473cd7cb9faa642487833865d516e578"; sid:1002; rev:1;)
```

- The Suricata rule above is designed to detect certain variations of [Sliver](https://github.com/BishopFox/sliver) whenever it identifies a TLS connection with a specific JA3 hash.

- The JA3 hash can be calculated as follows.
```shell
ja3 -a --json <path-to-pcap-file>/sliverenc.pcap
```
```json
[
    {
        "destination_ip": "23.152.0.91",
        "destination_port": 443,
        "ja3": "771,49195-49199-49196-49200-52393-52392-49161-49171-49162-49172-156-157-47-53-49170-10-4865-4866-4867,0-5-10-11-13-65281-18-43-51,29-23-24-25,0",
        "ja3_digest": "473cd7cb9faa642487833865d516e578",
        "source_ip": "10.10.20.101",
        "source_port": 53222,
        "timestamp": 1634749464.600896
    },
    {
        "destination_ip": "23.152.0.91",
        "destination_port": 443,
        "ja3": "771,49195-49199-49196-49200-52393-52392-49161-49171-49162-49172-156-157-47-53-49170-10-4865-4866-4867,0-5-10-11-13-65281-18-43-51,29-23-24-25,0",
        "ja3_digest": "473cd7cb9faa642487833865d516e578",
        "source_ip": "10.10.20.101",
        "source_port": 53225,
        "timestamp": 1634749465.069819
    },
    {
        "destination_ip": "23.152.0.91",
        "destination_port": 443,
        "ja3": "771,49195-49199-49196-49200-52393-52392-49161-49171-49162-49172-156-157-47-53-49170-10-4865-4866-4867,0-5-10-11-13-65281-18-43-51,29-23-24-25,0",
        "ja3_digest": "473cd7cb9faa642487833865d516e578",
        "source_ip": "10.10.20.101",
        "source_port": 53229,
        "timestamp": 1634749585.240773
    },
---SNIP---
```

- Execute Suricata on the file.
```shell
sudo suricata -r <path-to-pcap-file>/sliverenc.pcap -l . -k none
```
```
15/7/2023 -- 22:30:37 - <Notice> - This is Suricata version 6.0.13 RELEASE running in USER mode
15/7/2023 -- 22:30:37 - <Notice> - all 3 packet processing threads, 4 management threads initialized, engine started.
15/7/2023 -- 22:30:37 - <Notice> - Signal Received.  Stopping engine.
15/7/2023 -- 22:30:37 - <Notice> - Pcap-file module read 1 files, 15547 packets, 11904606 bytes
```

```shell
cat fast.log
```
```
10/20/2021-17:04:25.166658  [**] [1:1002:1] Sliver C2 SSL [**] [Classification: (null)] [Priority: 3] {TCP} 10.10.20.101:53225 -> 23.152.0.91:443
10/20/2021-17:07:25.315183  [**] [1:1002:1] Sliver C2 SSL [**] [Classification: (null)] [Priority: 3] {TCP} 10.10.20.101:53231 -> 23.152.0.91:443
10/20/2021-17:04:24.700690  [**] [1:1002:1] Sliver C2 SSL [**] [Classification: (null)] [Priority: 3] {TCP} 10.10.20.101:53222 -> 23.152.0.91:443
10/20/2021-17:06:25.328173  [**] [1:1002:1] Sliver C2 SSL [**] [Classification: (null)] [Priority: 3] {TCP} 10.10.20.101:53229 -> 23.152.0.91:443
10/20/2021-17:10:25.311929  [**] [1:1002:1] Sliver C2 SSL [**] [Classification: (null)] [Priority: 3] {TCP} 10.10.20.101:53234 -> 23.152.0.91:443
---SNIP---
```

---


# Snort Fundamentals

- Snort is an open-source intrusion detection system (IDS) and intrusion prevention system (IPS) that can also function as a packet logger/sniffer, similar to Suricata.
- It inspects network traffic to identify and log all activity, providing a comprehensive view and detailed logs of application layer transactions.
- Snort requires rule sets to define inspection parameters and identify specific items of interest.
- It is designed to operate efficiently on both general-purpose and custom hardware.

## Snort Operation Modes

- Snort typically operates in the following modes:
    - Inline IDS/IPS.
    - Passive IDS.
    - Network-based IDS.
    - Host-based IDS (however, Snort is not ideally a host-based IDS. We would recommend opting for more specialized tools for this.)

**According to [Snort's documentation](https://docs.snort.org/start/inspection)**:

"With certain DAQ modules, Snort is able to utilize two different modes of operation: `passive` and `inline`. `Passive mode` gives Snort the ability to observe and detect traffic on a network interface, but it prevents outright blocking of traffic. `Inline mode` on the other hand, does give Snort the ability to block traffic if a particular packet warrants such an event.

Snort will infer the particular mode of operation based on the options used at the command line. For example, reading from a pcap file with the `-r` option or listening on an interface with `-i` will cause Snort to run in passive mode by default. If the DAQ supports inline, however, then users can specify the `-Q` flag to run Snort inline.

One DAQ module that supports inline mode is `afpacket`, which is a module that gives Snort access to packets received on Linux network devices."

## Snort Architecture

- In order for Snort to transition from a simple packet sniffer to a robust IDS, several key components were added:
    - `Preprocessor`.
    - `Detection Engine`.
    - `Logging and Alerting System`.
    - Various `Output modules`.

1. The packet sniffer extracts network traffic, decodes packet structures, and forwards raw packets to the `Preprocessors`.
2. `Snort Preprocessors` analyze packets to identify their type or behavior, utilizing plugins like the HTTP preprocessor or the `port_scan` preprocessor. These plugins, configured in `snort.lua`, pass their findings to the `Detection Engine`.
3. The `Detection Engine` compares packets against a predefined rule set. Matches trigger information to be sent to the Logging and Alerting System.
4. The `Logging and Alerting System` and `Output modules` record events or trigger alerts based on rule actions. Logs are typically stored in `syslog`, `unified2` formats, or directly in a database. Output modules are configured in `snort.lua`.

- Snort high-level architecture
![Snort_high-level_architecture](./attachments/Snort_high-level_architecture.png)

## Snort Configuration & Validating Snort's Configuration

- Snort 3 offers pre-configured `snort.lua` and `snort_defaults.lua` files for quick and easy setup, providing a standard configuration framework for users.

- The `snort.lua` file serves as the principal configuration file for Snort.
- This file contains the following sections:
    - Network variables
    - Decoder configuration
    - Base detection engine configuration
    - Dynamic library configuration
    - Preprocessor configuration
    - Output plugin configuration
    - Rule set customization
    - Preprocessor and decoder rule set customization
    - Shared object rule set customization

- Let's browse the `snort.lua` file residing in this section's target as follows.
```shell
sudo more /root/snorty/etc/snort/snort.lua
```
```
---------------------------------------------------------------------------
-- Snort++ configuration
---------------------------------------------------------------------------

-- there are over 200 modules available to tune your policy.
-- many can be used with defaults w/o any explicit configuration.
-- use this conf as a template for your specific configuration.

-- 1. configure defaults
-- 2. configure inspection
-- 3. configure bindings
-- 4. configure performance
-- 5. configure detection
-- 6. configure filters
-- 7. configure outputs
-- 8. configure tweaks

---------------------------------------------------------------------------
-- 1. configure defaults
---------------------------------------------------------------------------

-- HOME_NET and EXTERNAL_NET must be set now
-- setup the network addresses you are protecting
HOME_NET = 'any'

-- set up the external network addresses.
-- (leave as "any" in most situations)
EXTERNAL_NET = 'any'

include 'snort_defaults.lua'

---------------------------------------------------------------------------
-- 2. configure inspection
---------------------------------------------------------------------------

-- mod = { } uses internal defaults
-- you can see them with snort --help-module mod

-- mod = default_mod uses external defaults
-- you can see them in snort_defaults.lua

-- the following are quite capable with defaults:

stream = { }
stream_ip = { }
stream_icmp = { }
stream_tcp = { }
stream_udp = { }
stream_user = { }
stream_file = { }
---SNIP---
```

- Enabling and fine-tuning Snort `modules` is a significant aspect of the configuration process.
- To explore the complete list and get a brief description of all Snort 3 modules, you can use the following command.
```shell
snort --help-modules
```
```
ack (ips_option): rule option to match on TCP ack numbers
active (basic): configure responses
address_space_selector (policy_selector): configure traffic processing based on address space
alert_csv (logger): output event in csv format
alert_fast (logger): output event with brief text format
alert_full (logger): output event with full packet dump
alert_json (logger): output event in json format
alert_syslog (logger): output event to syslog
alert_talos (logger): output event in Talos alert format
---SNIP---
```

- These modules are enabled and configured in the `snort.lua` configuration file using Lua table formats/literals.
- If a module is set up as an empty table, it means it's using its built-in "default" settings.
- You can use the following command to see these default settings.
```shell
snort --help-config arp_spoof
```
```
ip4 arp_spoof.hosts[].ip: host ip address
mac arp_spoof.hosts[].mac: host mac address
```

- Passing (and validating) configuration files to Snort can be done as follows.
```shell
snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq
```
```
--------------------------------------------------
o")~   Snort++ 3.1.64.0
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
        output
        ips
        classifications
        references
        binder
        file_id
        ftp_server
        smtp
        port_scan
        gtp_inspect
        dce_smb
        s7commplus
        modbus
        ssh
        active
        alerts
        daq
        decode
        host_cache
        host_tracker
        hosts
        network
        packets
        process
        search_engine
        so_proxy
        stream_icmp
        normalizer
        stream
        stream_ip
        stream_tcp
        stream_udp
        stream_user
        stream_file
        arp_spoof
        back_orifice
        dns
        imap
        netflow
        pop
        rpc_decode
        sip
        ssl
        telnet
        cip
        dnp3
        iec104
        mms
        dce_tcp
        dce_udp
        dce_http_proxy
        dce_http_server
        ftp_client
        ftp_data
        http_inspect
        http2_inspect
        file_policy
        js_norm
        appid
        wizard
        trace
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0     208       0     208    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 208
               text rules: 208
            option chains: 208
            chain headers: 1
--------------------------------------------------
service rule counts          to-srv  to-cli
                  file_id:      208     208
                    total:      208     208
--------------------------------------------------
fast pattern groups
                to_server: 1
                to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
                instances: 2
                 patterns: 416
            pattern chars: 2508
               num states: 1778
         num match states: 370
             memory scale: KB
             total memory: 68.5879
           pattern memory: 18.6973
        match list memory: 27.3281
        transition memory: 22.3125
appid: MaxRss diff: 3084
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to passive.

Snort successfully validated the configuration (with 0 warnings).
o")~   Snort exiting
```

- Snort 3 needs to know the location of `LibDAQ`.
- `LibDAQ`, or the "Data Acquisition Library," is a layer that simplifies how modules connect to/communicate with network data sources (both hardware and software).

## Snort Inputs

- To observe Snort in action, the easiest method is to execute it against a packet capture file.
- By providing the name of the pcap file as an argument to the `-r` option in the command line, Snort will process the file accordingly.
```shell
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r <path-to-pcap-file>icmp.pcap
```
```
--------------------------------------------------
o")~   Snort++ 3.1.64.0
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
---SNIP---
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0     208       0     208    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 208
               text rules: 208
            option chains: 208
            chain headers: 1
--------------------------------------------------
service rule counts          to-srv  to-cli
                  file_id:      208     208
                    total:      208     208
--------------------------------------------------
fast pattern groups
                to_server: 1
                to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
                instances: 2
                 patterns: 416
            pattern chars: 2508
               num states: 1778
         num match states: 370
             memory scale: KB
             total memory: 68.5879
           pattern memory: 18.6973
        match list memory: 27.3281
        transition memory: 22.3125
appid: MaxRss diff: 3024
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to read-file.
Commencing packet processing
++ [0] /home/htb-student/pcaps/icmp.pcap
-- [0] /home/htb-student/pcaps/icmp.pcap
--------------------------------------------------
Packet Statistics
--------------------------------------------------
daq
                    pcaps: 1
                 received: 8
                 analyzed: 8
                    allow: 8
                 rx_bytes: 592
--------------------------------------------------
codec
                    total: 8            (100.000%)
                      eth: 8            (100.000%)
                    icmp4: 8            (100.000%)
                     ipv4: 8            (100.000%)
--------------------------------------------------
Module Statistics
--------------------------------------------------
appid
                  packets: 8
        processed_packets: 8
           total_sessions: 1
--------------------------------------------------
binder
                new_flows: 1
                 inspects: 1
--------------------------------------------------
detection
                 analyzed: 8
--------------------------------------------------
port_scan
                  packets: 8
                 trackers: 2
--------------------------------------------------
stream
                    flows: 1
--------------------------------------------------
stream_icmp
                 sessions: 1
                      max: 1
                  created: 1
                 released: 1
--------------------------------------------------
Summary Statistics
--------------------------------------------------
timing
                  runtime: 00:00:00
                  seconds: 0.033229
                 pkts/sec: 241
o")~   Snort exiting
```

- Snort also has the capability to listen on active network interfaces.
- To specify this behavior, you can utilize the `-i` option followed by the names of the interfaces on which Snort should run.
```shell
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -i ens160
```
```
--------------------------------------------------
o")~   Snort++ 3.1.64.0
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
---SNIP---
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0     208       0     208    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 208
               text rules: 208
            option chains: 208
            chain headers: 1
--------------------------------------------------
service rule counts          to-srv  to-cli
                  file_id:      208     208
                    total:      208     208
--------------------------------------------------
fast pattern groups
                to_server: 1
                to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
                instances: 2
                 patterns: 416
            pattern chars: 2508
               num states: 1778
         num match states: 370
             memory scale: KB
             total memory: 68.5879
           pattern memory: 18.6973
        match list memory: 27.3281
        transition memory: 22.3125
appid: MaxRss diff: 2820
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to passive.
Commencing packet processing
++ [0] ens160
^C** caught int signal
== stopping
-- [0] ens160
--------------------------------------------------
Packet Statistics
--------------------------------------------------
daq
                 received: 33
                 analyzed: 33
                    allow: 33
                     idle: 9
                 rx_bytes: 2756
--------------------------------------------------
codec
                    total: 33           (100.000%)
                 discards: 2            (  6.061%)
                      arp: 13           ( 39.394%)
                      eth: 33           (100.000%)
                    icmp4: 12           ( 36.364%)
                    icmp6: 3            (  9.091%)
                     ipv4: 17           ( 51.515%)
                     ipv6: 3            (  9.091%)
                      tcp: 5            ( 15.152%)
--------------------------------------------------
Module Statistics
--------------------------------------------------
appid
                  packets: 18
        processed_packets: 16
          ignored_packets: 2
           total_sessions: 3
--------------------------------------------------
arp_spoof
                  packets: 13
--------------------------------------------------
binder
              raw_packets: 15
                new_flows: 3
                 inspects: 18
--------------------------------------------------
detection
                 analyzed: 33
--------------------------------------------------
port_scan
                  packets: 20
                 trackers: 8
--------------------------------------------------
stream
                    flows: 3
--------------------------------------------------
stream_icmp
                 sessions: 2
                      max: 2
                  created: 2
                 released: 2
--------------------------------------------------
stream_tcp
                 sessions: 1
                      max: 1
                  created: 1
                 released: 1
             instantiated: 1
                   setups: 1
            data_trackers: 1
              segs_queued: 1
            segs_released: 1
                 max_segs: 1
                max_bytes: 64
--------------------------------------------------
tcp
        bad_tcp4_checksum: 2
--------------------------------------------------
Appid Statistics
--------------------------------------------------
detected apps and services
              Application: Services   Clients    Users      Payloads   Misc       Referred
                  unknown: 1          0          0          0          0          0
--------------------------------------------------
Summary Statistics
--------------------------------------------------
process
                  signals: 1
--------------------------------------------------
timing
                  runtime: 00:00:25
                  seconds: 25.182182
                 pkts/sec: 1
o")~   Snort exiting
```

## Snort Rules

- Snort rules, similar to Suricata rules, consist of a `Rule Header` and `Rule Options`.
- Despite their similarities, we recommend consulting [https://docs.snort.org/](https://docs.snort.org/) and [https://docs.suricata.io/en/latest/rules/differences-from-snort.html](https://docs.suricata.io/en/latest/rules/differences-from-snort.html) for Snort rule writing guidance.
- The latest Snort rules are available on the Snort or Emerging Threats websites.

- In Snort deployments, rules can be directly integrated into the `snort.lua` configuration file using the `ips` module.
```shell
sudo vim /root/snorty/etc/snort/snort.lua
```
```
---SNIP---
ips =
{
    -- use this to enable decoder and inspector alerts
    --enable_builtin_rules = true,

    -- use include for rules files; be sure to set your path
    -- note that rules files can include other rules files
    -- (see also related path vars at the top of snort_defaults.lua)

    { variables = default_variables, include = '/home/htb-student/local.rules' }
}
---SNIP---
```

- Snort automatically loads "included" rules.
- Alternatively, rules can be incorporated directly from the command line using:
    - The `-R` option to specify a single specific rules file to be utilized by snort.
    - the `--rule-path` option to specify a path to the directory containing rules files. This enables us to provide Snort with a directory containing multiple rules files.

## Snort Outputs

In our Snort deployment, we may encounter a significant amount of data.
To provide a summary of the core output types, let's explore the key aspects:

- `Basic Statistics`—Upon shutdown, Snort generates various counts based on the configuration and processed traffic. This includes:
    - `Packet Statistics`—It includes information from the DAQ and decoders, such as the number of received packets and UDP packets.
    - `Module Statistics`—Each module keeps track of activity through peg counts, indicating the frequency of observed or performed actions. Examples include the count of processed HTTP GET requests and trimmed TCP reset packets.
    - `File Statistics`—This section provides a breakdown of file types, bytes, and signatures.
    - `Summary Statistics`—It encompasses the total runtime for packet processing, packets per second, and, if configured, profiling data.

- `Alerts`—When rules are configured, it is necessary to enable alerting (using the `-A` option) to view the details of detection events. There are multiple types of alert outputs available, including:
    - `-A cmg`—This option combines `-A fast -d -e` and displays alert information along with packet headers and payload.
    - `-A u2`—This option is equivalent to `-A unified2` and logs events and triggering packets in a binary file, which can be used for post-processing with other tools.
    - `-A csv`—This option outputs fields in comma-separated value format, providing customization options and facilitating pcap analysis.
    - ***Main Objective***—the `-A` option is used to enable and control how detection alerts are displayed or logged, allowing you to view and analyze security events in different output formats.

- To discover the available alert types, we can execute the following command.
```shell
snort --list-plugins | grep logger
```
```
logger::alert_csv v0 static
logger::alert_fast v0 static
logger::alert_full v0 static
logger::alert_json v0 static
logger::alert_syslog v0 static
logger::alert_talos v0 static
logger::alert_unixsock v0 static
logger::log_codecs v0 static
logger::log_hext v0 static
logger::log_pcap v0 static
logger::unified2 v0 static
```

- `Performance Statistics`—In addition to the outputs above, the `perf_monitor` module can be configured to capture a customizable set of `peg` counts during runtime. This data can be used by an external program to monitor Snort's activity without interrupting its operations. Furthermore, the `profiler` module tracks time and space usage by modules and rules, which helps optimize system performance. The profiler output is displayed under `Summary Statistics` during shutdown.

- Let's see an example of the `cmg` output.
```shell
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r <path-to-pcap-file>/icmp.pcap -A cmg
```
```
--------------------------------------------------
o")~   Snort++ 3.1.64.0
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
--SNIP---
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
Loading /home/htb-student/local.rules:
Finished /home/htb-student/local.rules:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0     209       0     209    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 209
               text rules: 209
            option chains: 209
            chain headers: 2
--------------------------------------------------
port rule counts
             tcp     udp    icmp      ip
     any       0       0       1       0
   total       0       0       1       0
--------------------------------------------------
service rule counts          to-srv  to-cli
                  file_id:      208     208
                    total:      208     208
--------------------------------------------------
fast pattern groups
                to_server: 1
                to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
                instances: 2
                 patterns: 416
            pattern chars: 2508
               num states: 1778
         num match states: 370
             memory scale: KB
             total memory: 68.5879
           pattern memory: 18.6973
        match list memory: 27.3281
        transition memory: 22.3125
appid: MaxRss diff: 3024
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to read-file.
Commencing packet processing
++ [0] /home/htb-student/pcaps/icmp.pcap
06/19-08:45:56.838904 [**] [1:1000001:1] "ICMP test" [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.158.139 -> 174.137.42.77
00:0C:29:34:0B:DE -> 00:50:56:E0:14:49 type:0x800 len:0x4A
192.168.158.139 -> 174.137.42.77 ICMP TTL:128 TOS:0x0 ID:55107 IpLen:20 DgmLen:60
Type:8  Code:0  ID:512   Seq:8448  ECHO

snort.raw[32]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
61 62 63 64 65 66 67 68  69 6A 6B 6C 6D 6E 6F 70  abcdefgh ijklmnop
71 72 73 74 75 76 77 61  62 63 64 65 66 67 68 69  qrstuvwa bcdefghi
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

06/19-08:45:57.055699 [**] [1:1000001:1] "ICMP test" [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 174.137.42.77 -> 192.168.158.139
00:50:56:E0:14:49 -> 00:0C:29:34:0B:DE type:0x800 len:0x4A
174.137.42.77 -> 192.168.158.139 ICMP TTL:128 TOS:0x0 ID:30433 IpLen:20 DgmLen:60
Type:0  Code:0  ID:512  Seq:8448  ECHO REPLY

snort.raw[32]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
61 62 63 64 65 66 67 68  69 6A 6B 6C 6D 6E 6F 70  abcdefgh ijklmnop
71 72 73 74 75 76 77 61  62 63 64 65 66 67 68 69  qrstuvwa bcdefghi
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

06/19-08:45:57.840049 [**] [1:1000001:1] "ICMP test" [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.158.139 -> 174.137.42.77
00:0C:29:34:0B:DE -> 00:50:56:E0:14:49 type:0x800 len:0x4A
192.168.158.139 -> 174.137.42.77 ICMP TTL:128 TOS:0x0 ID:55110 IpLen:20 DgmLen:60
Type:8  Code:0  ID:512   Seq:8704  ECHO

snort.raw[32]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
61 62 63 64 65 66 67 68  69 6A 6B 6C 6D 6E 6F 70  abcdefgh ijklmnop
71 72 73 74 75 76 77 61  62 63 64 65 66 67 68 69  qrstuvwa bcdefghi
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

---SNIP---

06/19-08:46:00.042354 [**] [1:1000001:1] "ICMP test" [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 174.137.42.77 -> 192.168.158.139
00:50:56:E0:14:49 -> 00:0C:29:34:0B:DE type:0x800 len:0x4A
174.137.42.77 -> 192.168.158.139 ICMP TTL:128 TOS:0x0 ID:30453 IpLen:20 DgmLen:60
Type:0  Code:0  ID:512  Seq:9216  ECHO REPLY

snort.raw[32]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
61 62 63 64 65 66 67 68  69 6A 6B 6C 6D 6E 6F 70  abcdefgh ijklmnop
71 72 73 74 75 76 77 61  62 63 64 65 66 67 68 69  qrstuvwa bcdefghi
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

-- [0] /home/htb-student/pcaps/icmp.pcap
--------------------------------------------------
Packet Statistics
--------------------------------------------------
daq
                    pcaps: 1
                 received: 8
                 analyzed: 8
                    allow: 8
                 rx_bytes: 592
--------------------------------------------------
codec
                    total: 8            (100.000%)
                      eth: 8            (100.000%)
                    icmp4: 8            (100.000%)
                     ipv4: 8            (100.000%)
--------------------------------------------------

---SNIP---

Summary Statistics
--------------------------------------------------
timing
                  runtime: 00:00:00
                  seconds: 0.042473
                 pkts/sec: 188
o")~   Snort exiting
```

- The same command but using a `.rules` files that may not be "included" in `snort.lua` is the following.
```shell
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r /home/htb-student/pcaps/icmp.pcap -R <path-to-pcap-file>local.rules -A cmg
```
```
--------------------------------------------------
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
--SNIP---
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
Loading /home/htb-student/local.rules:
Finished /home/htb-student/local.rules:
--------------------------------------------------
ips policies rule stats
             id  loaded  shared enabled    file
              0     209       0     209    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
      total rules loaded: 209
              text rules: 209
           option chains: 209
           chain headers: 2
--------------------------------------------------
port rule counts
            tcp     udp    icmp      ip
    any       0       0       1       0
  total       0       0       1       0
--------------------------------------------------
service rule counts          to-srv  to-cli
                 file_id:      208     208
                   total:      208     208
--------------------------------------------------
fast pattern groups
               to_server: 1
               to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
               instances: 2
                patterns: 416
           pattern chars: 2508
              num states: 1778
        num match states: 370
            memory scale: KB
            total memory: 68.5879
          pattern memory: 18.6973
       match list memory: 27.3281
       transition memory: 22.3125
appid: MaxRss diff: 3024
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to read-file.
Commencing packet processing
++ [0] /home/htb-student/pcaps/icmp.pcap
06/19-08:45:56.838904 [**] [1:1000001:1] "ICMP test" [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 192.168.158.139 -> 174.137.42.77
00:0C:29:34:0B:DE -> 00:50:56:E0:14:49 type:0x800 len:0x4A
192.168.158.139 -> 174.137.42.77 ICMP TTL:128 TOS:0x0 ID:55107 IpLen:20 DgmLen:60
Type:8  Code:0  ID:512   Seq:8448  ECHO

snort.raw[32]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
61 62 63 64 65 66 67 68  69 6A 6B 6C 6D 6E 6F 70  abcdefgh ijklmnop
71 72 73 74 75 76 77 61  62 63 64 65 66 67 68 69  qrstuvwa bcdefghi
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

---SNIP---

06/19-08:46:00.042354 [**] [1:1000001:1] "ICMP test" [**] [Classification: Generic ICMP event] [Priority: 3] {ICMP} 174.137.42.77 -> 192.168.158.139
00:50:56:E0:14:49 -> 00:0C:29:34:0B:DE type:0x800 len:0x4A
174.137.42.77 -> 192.168.158.139 ICMP TTL:128 TOS:0x0 ID:30453 IpLen:20 DgmLen:60
Type:0  Code:0  ID:512  Seq:9216  ECHO REPLY

snort.raw[32]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
61 62 63 64 65 66 67 68  69 6A 6B 6C 6D 6E 6F 70  abcdefgh ijklmnop
71 72 73 74 75 76 77 61  62 63 64 65 66 67 68 69  qrstuvwa bcdefghi
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

-- [0] /home/htb-student/pcaps/icmp.pcap
--------------------------------------------------
Packet Statistics
--------------------------------------------------
daq
                   pcaps: 1
                received: 8
                analyzed: 8
                   allow: 8
                rx_bytes: 592
--------------------------------------------------
codec
                   total: 8            (100.000%)
                     eth: 8            (100.000%)
                   icmp4: 8            (100.000%)
                    ipv4: 8            (100.000%)
--------------------------------------------------
Module Statistics
--------------------------------------------------
appid
                 packets: 8
       processed_packets: 8
          total_sessions: 1
--------------------------------------------------
binder
               new_flows: 1
                inspects: 1
--------------------------------------------------
detection
                analyzed: 8
              hard_evals: 8
                  alerts: 8
            total_alerts: 8
                  logged: 8
--------------------------------------------------
port_scan
                 packets: 8
                trackers: 2
--------------------------------------------------
search_engine
        qualified_events: 8
--------------------------------------------------
stream
                   flows: 1
--------------------------------------------------
stream_icmp
                sessions: 1
                     max: 1
                 created: 1
                released: 1
--------------------------------------------------
Summary Statistics
--------------------------------------------------
timing
                 runtime: 00:00:00
                 seconds: 0.042473
                pkts/sec: 188
o")~   Snort exiting
```

## Snort Key Features

- Key features that bolster Snort's effectiveness include:
    - Deep packet inspection, packet capture, and logging
    - Intrusion detection
    - Network Security Monitoring
    - Anomaly detection
    - Support for multiple tenants
    - Both IPv6 and IPv4 are supported

---

# Snort Rule Development

- Snort rules are powerful tools for identifying/flagging potential malicious network activity.
- While similar to Suricata rules in their structure (header and options), we recommend studying Snort rule writing using the following resources: [https://docs.snort.org/](https://docs.snort.org/), [https://docs.suricata.io/en/latest/rules/differences-from-snort.html](https://docs.suricata.io/en/latest/rules/differences-from-snort.html).

- Let's move forward and explore some examples of crafting Snort rules to counter real-world malware threats.

## Snort Rule Development Example 1: Detecting Ursnif (Inefficiently)

```
alert tcp any any -> any any (msg:"Possible Ursnif C2 Activity"; flow:established,to_server; content:"/images/", depth 12; content:"_2F"; content:"_2B"; content:"User-Agent|3a 20|Mozilla/4.0 (compatible|3b| MSIE 8.0|3b| Windows NT"; content:!"Accept"; content:!"Cookie|3a|"; content:!"Referer|3a|"; sid:1000002; rev:1;)
```

- This Snort rule detects a variation of [Ursnif](https://www.fortinet.com/blog/threat-research/ursnif-variant-spreading-word-document) malware but is inefficient due to missed HTTP sticky buffers.
- It operates on established, client-to-server TCP connections (`flow:established,to_server;`).
- The rule searches for:
    - "`/images/`" within the first 12 bytes of the payload (`content:"/images/", depth 12;`).
    - "`_2F`" and "`_2B`" anywhere in the payload (`content:"_2F";` and `content:"_2B";`).
- It also identifies a specific `User-Agent` string, using hexadecimal representations for "`:`" and "`;`" (`content:"User-Agent|3a 20|Mozilla/4.0 (compatible|3b| MSIE 8.0|3b| Windows NT";`).
- Finally, it checks for the absence of "`Accept`", "`Cookie:`", and "`Referer:`" HTTP headers (`content:!"Accept"; content:!"Cookie|3a|"; content:!"Referer|3a|";`).

- Run Snort against the `ursnif.pcap` file.
```shell
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -R <path-to-rule-file>/local.rules -r <path-to-pcap-file>/ursnif.pcap -A cmg
```
```
--------------------------------------------------
o")~   Snort++ 3.1.64.0
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
        hosts
---SNIP---
        host_tracker
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
Loading /home/htb-student/local.rules:
Finished /home/htb-student/local.rules:
Loading rule args:
Loading /home/htb-student/local.rules:
Finished /home/htb-student/local.rules:
Finished rule args:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0     210       2     210    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 210
          duplicate rules: 2
               text rules: 210
            option chains: 210
            chain headers: 5
--------------------------------------------------
port rule counts
             tcp     udp    icmp      ip
     any       1       0       1       0
   total       1       0       1       0
--------------------------------------------------
service rule counts          to-srv  to-cli
                  file_id:      208     208
                    total:      208     208
--------------------------------------------------
fast pattern groups
                      any: 2
                to_server: 1
                to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
                instances: 3
                 patterns: 417
            pattern chars: 2566
               num states: 1836
         num match states: 371
             memory scale: KB
             total memory: 70.9639
           pattern memory: 18.792
        match list memory: 27.8125
        transition memory: 23.9844
appid: MaxRss diff: 3024
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to read-file.
Commencing packet processing
++ [0] /home/htb-student/pcaps/ursnif.pcap
07/21-19:27:47.161230 [**] [1:1000002:1] "Possible Ursnif C2 Activity" [**] [Priority: 0] {TCP} 10.10.10.104:49191 -> 192.42.116.41:80
00:1F:E2:10:8B:C9 -> 00:0C:29:C9:67:00 type:0x800 len:0x18C
10.10.10.104:49191 -> 192.42.116.41:80 TCP TTL:128 TOS:0x0 ID:20640 IpLen:20 DgmLen:382 DF
***AP*** Seq: 0x12E06BB  Ack: 0xE061E225  Win: 0x4029  TcpLen: 20

snort.raw[342]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
47 45 54 20 2F 69 6D 61  67 65 73 2F 70 32 52 55  GET /ima ges/p2RU
52 68 5F 32 2F 42 6B 32  76 72 31 4F 59 52 46 31  Rh_2/Bk2 vr1OYRF1
57 47 75 35 6E 67 5F 32  46 73 66 73 2F 57 4F 57  WGu5ng_2 Fsfs/WOW
72 47 54 45 54 79 4B 2F  37 4D 7A 4D 5F 32 42 6E  rGTETyK/ 7MzM_2Bn
47 5A 51 52 32 6A 73 67  50 2F 73 5F 32 42 70 53  GZQR2jsg P/s_2BpS
34 31 4B 37 41 67 2F 4F  75 4A 6A 51 66 41 61 63  41K7Ag/O uJjQfAac
32 64 2F 76 6D 46 5F 32  46 31 4B 42 50 72 4B 5F  2d/vmF_2 F1KBPrK_
32 2F 46 36 36 38 32 64  67 64 69 61 47 31 7A 75  2/F6682d gdiaG1zu
56 43 7A 37 47 68 64 2F  62 4C 37 36 57 66 35 71  VCz7Ghd/ bL76Wf5q
64 71 77 56 35 76 7A 52  2F 32 65 31 41 38 79 42  dqwV5vzR /2e1A8yB
49 64 6B 6D 49 5F 32 42  2F 6F 67 79 7A 4E 55 57  IdkmI_2B /ogyzNUW
33 47 72 2F 67 4B 42 5A  57 58 78 2E 67 69 66 20  3Gr/gKBZ WXx.gif
48 54 54 50 2F 31 2E 31  0D 0A 55 73 65 72 2D 41  HTTP/1.1 ..User-A
67 65 6E 74 3A 20 4D 6F  7A 69 6C 6C 61 2F 34 2E  gent: Mo zilla/4.
30 20 28 63 6F 6D 70 61  74 69 62 6C 65 3B 20 4D  0 (compa tible; M
53 49 45 20 38 2E 30 3B  20 57 69 6E 64 6F 77 73  SIE 8.0;  Windows
20 4E 54 20 36 2E 31 29  0D 0A 48 6F 73 74 3A 20   NT 6.1) ..Host:
62 6C 75 65 77 61 74 65  72 73 74 6F 6E 65 2E 72  bluewate rstone.r
75 0D 0A 43 6F 6E 6E 65  63 74 69 6F 6E 3A 20 4B  u..Conne ction: K
65 65 70 2D 41 6C 69 76  65 0D 0A 43 61 63 68 65  eep-Aliv e..Cache
2D 43 6F 6E 74 72 6F 6C  3A 20 6E 6F 2D 63 61 63  -Control : no-cac
68 65 0D 0A 0D 0A                                 he....
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

-- [0] /home/htb-student/pcaps/ursnif.pcap
--------------------------------------------------
Packet Statistics
--------------------------------------------------
daq
                    pcaps: 1
                 received: 49
                 analyzed: 49
                    allow: 49
                 rx_bytes: 5925
--------------------------------------------------
codec
                    total: 49           (100.000%)
                      eth: 49           (100.000%)
                     ipv4: 49           (100.000%)
                      tcp: 16           ( 32.653%)
                      udp: 33           ( 67.347%)
--------------------------------------------------
Module Statistics
--------------------------------------------------
appid
                  packets: 49
        processed_packets: 49
           total_sessions: 15
       service_cache_adds: 5
             bytes_in_use: 760
             items_in_use: 5
--------------------------------------------------
back_orifice
                  packets: 33
--------------------------------------------------
binder
                new_flows: 12
          service_changes: 2
                 inspects: 12
--------------------------------------------------
detection
                 analyzed: 49
             raw_searches: 2
          cooked_searches: 1
             pkt_searches: 3
            file_searches: 1
                   alerts: 1
             total_alerts: 1
                   logged: 1
--------------------------------------------------
dns
                  packets: 19
                 requests: 12
                responses: 7
--------------------------------------------------
file_id
              total_files: 1
          total_file_data: 304
     max_concurrent_files: 1
--------------------------------------------------
http_inspect
                    flows: 2
                    scans: 9
              reassembles: 9
              inspections: 9
                 requests: 2
                responses: 2
             get_requests: 1
            post_requests: 1
           request_bodies: 1
  max_concurrent_sessions: 2
              total_bytes: 1603
--------------------------------------------------
port_scan
                  packets: 49
                 trackers: 7
--------------------------------------------------
search_engine
               max_queued: 1
            total_flushed: 2
            total_inserts: 2
             total_unique: 2
     non_qualified_events: 1
         qualified_events: 1
           searched_bytes: 2192
--------------------------------------------------
stream
                    flows: 12
             total_prunes: 7
idle_prunes_proto_timeout: 7
--------------------------------------------------
stream_tcp
                 sessions: 2
                      max: 2
                  created: 2
                 released: 2
             instantiated: 2
                   setups: 2
                 restarts: 2
             syn_trackers: 2
              segs_queued: 4
            segs_released: 4
                segs_used: 4
          rebuilt_packets: 9
            rebuilt_bytes: 1627
                     syns: 2
                 syn_acks: 2
                   resets: 2
                 max_segs: 1
                max_bytes: 981
--------------------------------------------------
stream_udp
                 sessions: 10
                      max: 10
                  created: 13
                 released: 13
                 timeouts: 3
              total_bytes: 1964
--------------------------------------------------
wizard
                tcp_scans: 2
                 tcp_hits: 2
                udp_scans: 3
               udp_misses: 3
--------------------------------------------------
Appid Statistics
--------------------------------------------------
detected apps and services
              Application: Services   Clients    Users      Payloads   Misc       Referred
                  unknown: 11         9          0          2          0          0
--------------------------------------------------
Summary Statistics
--------------------------------------------------
timing
                  runtime: 00:00:00
                  seconds: 0.039200
                 pkts/sec: 1250
                Mbits/sec: 1
o")~   Snort exiting
```

## Snort Rule Development Example 2: Detecting Cerber

```
alert udp $HOME_NET any -> $EXTERNAL_NET any (msg:"Possible Cerber Check-in"; dsize:9; content:"hi", depth 2, fast_pattern; pcre:"/^[af0-9]{7}$/R"; detection_filter:track by_src, count 1, seconds 60; sid:2816763; rev:4;)
```

- The Snort rule above is designed to detect certain variations of [Cerber](https://blog.checkpoint.com/research/14959/) malware.
- Let's break down the important parts of this rule to understand its workings.

- `$HOME_NET any -> $EXTERNAL_NET any` signifies the rule applies to any `UDP` traffic going from any port in the home network to any port on external networks.
- `dsize:9;`: This is a condition that restricts the rule to UDP datagrams that have a payload data size of exactly `9` bytes.
- `content:"hi", depth 2, fast_pattern;`: This checks the payload's first `2` bytes for the string `hi`. The `fast_pattern` modifier makes the pattern matcher search for this pattern before any others in the rule, optimizing the rule's performance.
- `pcre:"/^[af0-9]{7}$/R";`: This stands for Perl Compatible Regular Expressions. The rule is looking for seven hexadecimal characters (from the set `a-f` and `0-9`) starting at the beginning of the payload (after the `hi`), and this should be the complete payload (signified by the `$` end anchor).
- `detection_filter:track by_src, count 1, seconds 60;`: The `detection_filter` keyword in Snort rule language is used to suppress alerts unless a certain threshold of matched events occurs within a specified time frame. In this rule, the filter is set to track by source IP (`by_src`), with a count of `1` and within a time frame of `60` seconds. This means that the rule will trigger an alert only if it matches more than one event (specifically, more than count events which is `1` here) from the same source IP address within `60` seconds.

- To test this Snort rule, add it in a file called `local.rules`, then run Snort against the `cerber.pcap` file.
```shell
sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -R <path-to-rules-file> -r <path-to-pcap-file> -A cmg
```
```
--------------------------------------------------
o")~   Snort++ 3.1.64.0
--------------------------------------------------
Loading /root/snorty/etc/snort/snort.lua:
Loading snort_defaults.lua:
Finished snort_defaults.lua:
        output
---SNIP---
        trace
Finished /root/snorty/etc/snort/snort.lua:
Loading file_id.rules_file:
Loading file_magic.rules:
Finished file_magic.rules:
Finished file_id.rules_file:
Loading /home/htb-student/local.rules:
Finished /home/htb-student/local.rules:
Loading rule args:
Loading /home/htb-student/local.rules:
Finished /home/htb-student/local.rules:
Finished rule args:
--------------------------------------------------
ips policies rule stats
              id  loaded  shared enabled    file
               0     210       2     210    /root/snorty/etc/snort/snort.lua
--------------------------------------------------
rule counts
       total rules loaded: 210
          duplicate rules: 2
               text rules: 210
            option chains: 210
            chain headers: 5
--------------------------------------------------
port rule counts
             tcp     udp    icmp      ip
     any       0       1       1       0
   total       0       1       1       0
--------------------------------------------------
service rule counts          to-srv  to-cli
                  file_id:      208     208
                    total:      208     208
--------------------------------------------------
fast pattern groups
                      any: 2
                to_server: 1
                to_client: 1
--------------------------------------------------
search engine (ac_bnfa)
                instances: 3
                 patterns: 417
            pattern chars: 2511
               num states: 1781
         num match states: 371
             memory scale: KB
             total memory: 69.8359
           pattern memory: 18.7383
        match list memory: 27.3828
        transition memory: 23.3398
appid: MaxRss diff: 3024
appid: patterns loaded: 300
--------------------------------------------------
pcap DAQ configured to read-file.
Commencing packet processing
++ [0] /home/htb-student/pcaps/cerber.pcap
07/22-02:17:56.486663 [**] [1:2816763:4] "Possible Cerber Check-in" [**] [Priority: 0] {UDP} 10.0.2.15:1046 -> 31.184.234.1:6892
08:00:27:A9:8C:97 -> 52:54:00:12:35:02 type:0x800 len:0x3C
10.0.2.15:1046 -> 31.184.234.1:6892 UDP TTL:128 TOS:0x0 ID:83 IpLen:20 DgmLen:37
Len: 9

snort.raw[9]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
68 69 30 30 37 32 38 39  35                       hi007289 5
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -

07/22-02:17:56.486795 [**] [1:2816763:4] "Possible Cerber Check-in" [**] [Priority: 0] {UDP} 10.0.2.15:1046 -> 31.184.234.2:6892
08:00:27:A9:8C:97 -> 52:54:00:12:35:02 type:0x800 len:0x3C
10.0.2.15:1046 -> 31.184.234.2:6892 UDP TTL:128 TOS:0x0 ID:84 IpLen:20 DgmLen:37
Len: 9

snort.raw[9]:
- - - - - - - - - - - -  - - - - - - - - - - - -  - - - - - - - - -
68 69 30 30 37 32 38 39  35                       hi007289 5
---SNIP---
-- [0] /home/htb-student/pcaps/cerber.pcap
--------------------------------------------------
Packet Statistics
--------------------------------------------------
daq
                    pcaps: 1
                 received: 1035
                 analyzed: 1035
                    allow: 1035
                 rx_bytes: 65672
--------------------------------------------------
codec
                    total: 1035         (100.000%)
                      eth: 1035         (100.000%)
                     ipv4: 1035         (100.000%)
                      tcp: 9            (  0.870%)
                      udp: 1026         ( 99.130%)
--------------------------------------------------
Module Statistics
--------------------------------------------------
appid
                  packets: 1035
        processed_packets: 1035
           total_sessions: 1026
       service_cache_adds: 514
             bytes_in_use: 78128
             items_in_use: 514
--------------------------------------------------
back_orifice
                  packets: 514
--------------------------------------------------
binder
                new_flows: 1026
          service_changes: 1
                 inspects: 1026
--------------------------------------------------
detection
                 analyzed: 1035
             raw_searches: 1026
             pkt_searches: 1026
            file_searches: 1
                   alerts: 511
             total_alerts: 511
                   logged: 511
--------------------------------------------------
dns
                  packets: 2
                 requests: 1
                responses: 1
--------------------------------------------------
file_id
              total_files: 1
          total_file_data: 164
     max_concurrent_files: 1
--------------------------------------------------
http_inspect
                    flows: 1
                    scans: 5
              reassembles: 5
              inspections: 5
                 requests: 1
                responses: 1
             get_requests: 1
  max_concurrent_sessions: 1
              total_bytes: 462
--------------------------------------------------
pcre
               pcre_rules: 2
              pcre_native: 2
--------------------------------------------------
port_scan
                  packets: 1035
                 trackers: 516
--------------------------------------------------
search_engine
               max_queued: 1
            total_flushed: 512
            total_inserts: 512
             total_unique: 512
     non_qualified_events: 1
         qualified_events: 511
           searched_bytes: 17146
--------------------------------------------------
stream
                    flows: 1026
--------------------------------------------------
stream_tcp
                 sessions: 1
                      max: 1
                  created: 1
                 released: 1
             instantiated: 1
                   setups: 1
                 restarts: 1
             syn_trackers: 1
              segs_queued: 2
            segs_released: 2
                segs_used: 2
          rebuilt_packets: 5
            rebuilt_bytes: 474
                     syns: 1
                 syn_acks: 1
                     fins: 1
                 max_segs: 1
                max_bytes: 435
--------------------------------------------------
stream_udp
                 sessions: 1025
                      max: 1025
                  created: 1025
                 released: 1025
              total_bytes: 16982
--------------------------------------------------
wizard
                tcp_scans: 1
                 tcp_hits: 1
                udp_scans: 1024
               udp_misses: 1024
--------------------------------------------------
Appid Statistics
--------------------------------------------------
detected apps and services
              Application: Services   Clients    Users      Payloads   Misc       Referred
                  unknown: 2          1          0          1          0          0
--------------------------------------------------
Summary Statistics
--------------------------------------------------
timing
                  runtime: 00:01:11
                  seconds: 71.153373
                 pkts/sec: 15
o")~   Snort exiting
```

- When using Wireshark to apply this rule, the used filter:
```
udp && udp.length == 17 && udp contains "hi"
```
![suricata_example_2_1](./attachments/suricata_example_2_1.png)
- You'll notice that nearly half of the traffic is duplicates of each other:
  - All have length 9.
  - From same source to a destination sequential subnet.
  - And contain the same message data—`hi0072895`.

## Snort Rule Development Example 3: Detecting Patchwork

```
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"OISF TROJAN Targeted AutoIt FileStealer/Downloader CnC Beacon"; flow:established,to_server; http_method; content:"POST"; http_uri; content:".php?profile="; http_client_body; content:"ddager=", depth 7; http_client_body; content:"&r1=", distance 0; http_header; content:!"Accept"; http_header; content:!"Referer|3a|"; sid:10000006; rev:1;)
```

#### The "Who and Where"

- **`alert http $HOME_NET any -> $EXTERNAL_NET any`**—This tells the system to watch for HTTP traffic (web traffic) leaving your internal network (`$HOME_NET`) and heading out to the internet (`$EXTERNAL_NET`).

- **`flow:established,to_server`**—This ignores the initial connection handshake (the 3-way handshake) and only looks at the actual data being sent from the infected computer to the attacker's Command & Control (C2) server.

#### Packet Content

- For this rule to trigger, the web request must meet **all** of the following criteria.
- If even one is missing, the rule stays silent:

##### 1. The Method and Location

- **`http_method; content:"POST";`**—The malware isn't just "looking" at a page; it’s "posting" data (uploading information).
- **`http_uri; content:".php?profile=";`**—The destination URL on the attacker's server must end in `.php?profile=`. This is a common way for malware to send "ID" info about the infected computer.

##### 2. The Body (The "Cargo")

- The rule looks inside the data being uploaded (`http_client_body`) for specific markers:
  - **`content:"ddager=", depth 7;`**—The very first 7 bytes of the data must be `ddager=`. This is likely a hardcoded variable name the attacker used in their code.
  - **`content:"&r1=", distance 0;`**—Immediately following the first variable, it looks for `&r1=`. This structure is typical for a web form sending multiple pieces of stolen data.

##### 3. The "Missing" Headers (The Stealth Check)

- This is the most clever part of the rule.
- Legitimate web browsers almost always send certain "headers" to identify themselves.
- This malware is poorly coded or trying to be minimal, so the rule looks for the **absence** of these headers:
  - **`content:!"Accept";`**—It triggers if the `Accept` header is **missing**.
  - **`content:!"Referer|3a|";`**—It triggers if the `Referer` header (which shows which site you came from) is **missing**.

- Invest some time in scrutinizing both the `patchwork.pcap` file using `Wireshark` and this rule to comprehend how it works.
- Wireshark Applied Filter:
```
http && ip.src == 192.168.1.37 && http.request.method == POST && http.request.uri contains ".php?profile=" && http.file_data contains "ddager=" && http.file_data contains "&r1=" && !http.accept && !http.referer
```
![snort_example_3_1.png](./attachments/snort_example_3_1.png)

## Snort Rule Development Example 4: Detecting Patchwork (SSL)

```
alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"Patchwork SSL Cert Detected"; flow:established,from_server; content:"|55 04 03|"; content:"|08|toigetgf", distance 1, within 9; classtype:trojan-activity; sid:10000008; rev:1;)
```

- `flow:established,from_server;`—Matches established traffic flows originating from the server.
- `content:"|55 04 03|";`—Searches for the hex sequence `55 04 03`, representing the ASN.1 tag for the "common name" field in an X.509 certificate (often the domain name).
- `content:"|08|toigetgf", distance 1, within 9;`: Locates the string `toigetgf` starting 1 byte after the previous content match, within a 9-byte window.

- Wireshark filter used:
```
tls.handshake.certificate && frame contains 55:04:03 && frame contains 6f:69:67:65:74:67:66
```
![snort_example_4_1.png](./attachments/snort_example_4_1.png)

---


# Zeek Fundamentals

- Zeek is an open-source network traffic analyzer used to detect suspicious or malicious activity by scrutinizing network traffic.
- Beyond security, it aids in troubleshooting network issues and performing network measurements.
- Once deployed, Zeek provides defensive cybersecurity teams with extensive log files detailing network activity, including connection records and application-layer transcripts (e.g., DNS, HTTP).
- Zeek also offers built-in functions for analyzing and detecting network activities.

- Zeek's powerful scripting language allows users to create custom scripts, similar to Suricata rules, enabling adaptable network analysis and intrusion detection.
- Unlike signature-based IDSs, Zeek's script ability and ability to run on standard hardware facilitate:
  - Semantic misuse detection.
  - Anomaly detection.
  - Behavioral analysis.

## Zeek's Operation Modes

- Zeek operates in the following modes.

#### Fully passive traffic analysis

- Zeek acts like a silent observer on a network tap or span port, meaning it monitors traffic without interfering with the flow of data.
- This ensures that even if Zeek experiences a failure, your network connectivity remains completely unaffected.

#### `libpcap` interface for packet capture

- Zeek uses the standard `libpcap` library to "listen" to the network interface, allowing it to capture raw data packets directly from the hardware.
- It translates these messy, raw packets into high-level, readable logs that describe network events (like HTTP requests or TLS handshakes).

#### Real-time and offline (e.g., PCAP-based) analysis

- It can analyze live traffic as it happens to provide instant visibility, or it can ingest recorded **PCAP files** for forensic investigations.
- This allows security teams to "rewind the tape" and analyze past security incidents with the same depth as live events.

#### Cluster support for large-scale deployments

- To handle massive amounts of data (like a 100Gbps backbone), Zeek can be distributed across multiple servers in a cluster.
- A "manager" node coordinates the work, while multiple "worker" nodes split the traffic load, ensuring the system stays performant under heavy pressure.

## Zeek's Architecture

- Zeek's architecture comprises two primary components:
  - The `event engine` (or core).
    - The `script interpreter`.

#### Event Engine

- Think of the **Event Engine** as a neutral translator.
- It sits between:
  - The raw, messy data of the network.
  - The human-readable reports you actually want to see.

- The ***Event Engine***:
  - Processes incoming network data.
  - Then Converts it into meaningful events.
- In Zeek, these events describe network activity without making any judgments or interpretations.
- They simply tell us what is happening.
- For example:
  - An HTTP request is turned into an `http_request` event.
  - This event includes all the details of the request but doesn't analyze it or determine if it's malicious, such as checking if a port is associated with malware.

#### Script Interpreter

- Zeek's script interpreter analyzes network traffic using event handlers written in the Zeek scripting language.
- These scripts implement the site's security policy by defining actions to be taken upon detection of certain events.
- Zeek's core queues events for processing in the order they are received.
- Most events are defined in `.bif` files within the `/scripts/base/bif/plugins/ directory`.
- Refer to: [https://docs.zeek.org/en/stable/scripts/base/bif/](https://docs.zeek.org/en/stable/scripts/base/bif) for more information.

## Zeek Logs

- When Zeek analyzes a PCAP file offline, it saves logs to the current directory.

- Common Zeek logs include:
  - `conn.log`—This log provides details about IP, TCP, UDP, and ICMP connections.
  - `dns.log`—Here, you'll find the details of DNS queries and responses.
  - `http.log`—This log captures the details of HTTP requests and responses.
  - `ftp.log`—Details of FTP requests and responses are logged here.
  - `smtp.log`—This log covers SMTP transactions, such as sender and recipient details.

- For example, the `http.log` contains valuable data such as:
  - `host`—The HTTP domain/IP.
  - `uri`—The HTTP URI.
  - `referrer`—The referrer of the HTTP request.
  - `user_agent`—The client's user agent.
  - `status_code`—The HTTP status code.

- For a more exhaustive list of common Zeek logs and their respective fields, refer to the following resource: [https://docs.zeek.org/en/master/logs/index.html](https://docs.zeek.org/en/master/logs/index.html)

- Zeek, by default:
  - Compresses log files hourly using `gzip`.
  - Archives them in `YYYY-MM-DD` directories.
- For working with these compressed logs, consider using tools like `gzcat` or `zgrep`.

- You can find examples on how to work with gzip-compressed Zeek logs at this link: [https://blog.rapid7.com/2016/06/02/working-with-bro-logs-queries-by-example/](https://blog.rapid7.com/2016/06/02/working-with-bro-logs-queries-by-example/)

- While standard Unix commands like `cat` and `grep` can process Zeek logs, Zeek provides the `zeek-cut` utility.
- This tool:
  - Accepts Zeek log files as standard input via pipelines or stream redirection.
  - Outputs specified columns.

- If you're interested in exploring Zeek examples, use cases, and learning the basics of writing Zeek scripts, take a look at the following link: [https://docs.zeek.org/en/stable/examples/index.html](https://docs.zeek.org/en/stable/examples/index.html)

- For a quick start guide to Zeek, refer to the following link: [https://docs.zeek.org/en/stable/quickstart/index.html](https://docs.zeek.org/en/stable/quickstart/index.html)

## Zeek Key Features

- Network activity logging.
- Application protocol analysis (HTTP, DNS, FTP, etc.).
- File content inspection.
- IPv6 support.
- Tunnel detection.
- Protocol sanity checks.
- IDS-like pattern matching.
- Powerful scripting for analysis and network state management.
- Structured ASCII logs with ElasticSearch/DataSeries backends.
- Real-time external input integration.
- External C library for Zeek events.
- External process triggering.

---

# Intrusion Detection With Zeek

- Zeek, formerly Bro, is a powerful and flexible network security monitoring tool that provides deep insights into network traffic.
- Its rich logs and extensive scripting capabilities enable customized intrusion detection and continuous improvement of our security posture.

## Intrusion Detection With Zeek Example 1: Detecting Beaconing Malware

- Beaconing is a malware communication method/process where it periodically contacts its command and control (C2) server:
  - To receive instructions.
  - For data exfiltration.

- Analyzing connection logs (`conn.log`) for:
  - Repetitive connections to the same destination IP or domain.
  - Constant data size.
  - Connection timing.
- This will look for patterns in outbound traffic, and reveal potential beaconing behavior.

```shell
/usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/psempire.pcap
```

```shell
cat conn.log
```
![zeek_example_1_1](./attachments/zeek_example_1_1.png)

| **Field**      | **Value**           | **What it means**                                                                                                                   |
| -------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **ts**         | `1511269439.125289` | **The Timestamp:** Exactly when the connection started (in Unix time).                                                              |
| **uid**        | `CuQYC98rE69BBb7jb` | **Unique ID:** A "tracking number" Zeek gives this specific conversation so you can find it in other logs (like the `http.log`).    |
| **id.orig_h**  | `192.168.56.14`     | **Originator (Source):** The IP address of the device that started the connection (the "caller").                                   |
| **id.resp_h**  | `51.15.197.127`     | **Responder (Destination):** The IP address that answered the request (the "receiver").                                             |
| **id.resp_p**  | `80`                | **Port:** They connected on Port 80, which is the standard port for **unencrypted web traffic (HTTP).**                             |
| **proto**      | `tcp`               | **Protocol:** They used TCP, which is like a "certified mail" delivery system for data.                                             |
| **duration**   | `2.186789`          | **Time Spent:** The entire conversation lasted about **2.18 seconds.**                                                              |
| **orig_bytes** | `204`               | **Data Sent:** The source device sent **204 bytes** (likely a small web request).                                                   |
| **resp_bytes** | `5557`              | **Data Received:** The destination sent back **5,557 bytes** (likely the actual webpage content).                                   |
| **conn_state** | `SF`                | **Status:** "SF" stands for **S**ynchronize and **F**inished. This means the connection started and ended perfectly with no errors. |

- Traffic to `51.15.197.127:80` occurs approximately every 5 seconds, suggesting malware beaconing.
- The `psempire.pcap` file contains traffic related to PowerShell Empire, which beacons every 5 seconds by default.

- Using this filter in Wireshark, will make the 5 seconds span more clear.
```
http && ip.src == 192.168.56.14
```
![zeek_example_1](./attachments/zeek_example_1_2.png)


## Intrusion Detection With Zeek Example 2: Detecting DNS Exfiltration

- Zeek aids in detecting data exfiltration, which is often difficult to identify due to its resemblance/mimicking to normal network traffic.
- Zeek's deeper network traffic analysis allows for the identification of unusual data transfers. Specifically, `files.log` can reveal large data volumes sent to atypical external destinations or over non-standard ports.
- The `http.log` and `dns.log` can expose covert (hidden or invisible) exfiltration channels like DNS tunneling or suspicious HTTP POST requests.
- Additionally, Zeek's file reassembly capability, irrespective of the protocol, helps determine the type/nature of exfiltrated data.

```shell
/usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/dnsexfil.pcapng
```

- Let's extract the requested (sub)domains using `zeek-cut`.
```shell
cat dns.log | /usr/local/zeek/bin/zeek-cut query | cut -d . -f1-7
```
```
safebrowsing.google.com
456c54f2.blue.letsgohunt.online
456c54f2.blue.letsgohunt.online
wpad.windomain.local
456c54f2.blue.letsgohunt.online
www.180.0c9a5671.456c54f2.blue.letsgohunt.online
www.1204192da26d109d4.1c9a5671.456c54f2.blue.letsgohunt.online
www.11a1855b98d2b71c3.2c9a5671.456c54f2.blue.letsgohunt.online
www.1f5e94740470d0157.9c9a5671.456c54f2.blue.letsgohunt.online
www.114cbea690a81874a.ac9a5671.456c54f2.blue.letsgohunt.online
www.10db7634eade0b736.bc9a5671.456c54f2.blue.letsgohunt.online
www.1d5aee37e1c25ba02.cc9a5671.456c54f2.blue.letsgohunt.online
cdn.5600553f0.456c54f2.blue.letsgohunt.online
cdn.d600553f0.456c54f2.blue.letsgohunt.online
cdn.e600553f0.456c54f2.blue.letsgohunt.online
cdn.f600553f0.456c54f2.blue.letsgohunt.online
cdn.10600553f0.456c54f2.blue.letsgohunt.online
post.140.0346c53ab.456c54f2.blue.letsgohunt.online
post.10bb13b53.1346c53ab.456c54f2.blue.letsgohunt.online
post.104fb3984.2346c53ab.456c54f2.blue.letsgohunt.online
post.1bdfe1d1e.3346c53ab.456c54f2.blue.letsgohunt.online
post.19f3acfa6.4346c53ab.456c54f2.blue.letsgohunt.online
kohaqbopxlq.windomain.local
kohaqbopxlq.windomain.local
456c54f2.blue.letsgohunt.online
wpad.windomain.local
456c54f2.blue.letsgohunt.online
456c54f2.blue.letsgohunt.online
cdn.013821c34.456c54f2.blue.letsgohunt.online
cdn.b13821c34.456c54f2.blue.letsgohunt.online
cdn.c13821c34.456c54f2.blue.letsgohunt.online
456c54f2.blue.letsgohunt.online
456c54f2.blue.letsgohunt.online
456c54f2.blue.letsgohunt.online
456c54f2.blue.letsgohunt.online
post.1dbcb3f1b.37467121d5.456c54f2.blue.letsgohunt.online
---SNIP---
```
- Command Explanation:
  - `cat dns.log`—This opens the Zeek DNS log file and feeds its raw text into the next part of the command.
  - `| /usr/local/zeek/bin/zeek-cut query`—This uses Zeek’s specialized tool to isolate and extract only the `query` column while ignoring all other metadata like timestamps or IP addresses.
  - `| cut -d . -f1-7`—This uses the period as a separator to keep only the first seven segments of each domain name, effectively stripping off any trailing data.

- The domain `letsgohunt.online` has a high number of subdomains, similar to cloud providers.
- However, it's worth noting that interactions with dozens or even hundreds of subdomains are generally not considered typical behavior.

## Intrusion Detection With Zeek Example 3: Detecting TLS Exfiltration

- Let's now go over an example of detecting data exfiltration over TLS.
```shell
/usr/local/zeek/bin/zeek -C -r <path-to-pcap-file>
```

- Let's narrow things down by using `zeek-cut` one more time.

- **One-liner source**: [activecountermeasures](https://www.activecountermeasures.com/)
```shell
cat conn.log | /usr/local/zeek/bin/zeek-cut id.orig_h id.resp_h orig_bytes | sort | grep -v -e '^$' | grep -v '-' | datamash -g 1,2 sum 3 | sort -k 3 -rn | head -10
```
```
10.0.10.100     192.168.151.181 270775912
10.0.10.100     10.0.10.1       0
```

- `cat conn.log`—This command reads the `conn.log` file, which Zeek generates to record network connections.
- `/usr/local/zeek/bin/zeek-cut id.orig_h id.resp_h orig_bytes`—we're extracting the following fields:
  - `id.orig_h` (originating host).
  - `id.resp_h` (responding host).
  - `orig_bytes` (number of bytes sent by the originating host).
- `sort`—Sort arranges the output from the previous command, defaulting to ascending order based on the first field (`id.orig_h` in our case).
- `grep -v -e '^$'`—Filters out empty lines:
  - `-v` inverts the selection (it will show you everything except empty lines).
  - `-e` enables regular expression matching, and `^$` matches empty lines.
- `grep -v '-'`—This command filters lines containing a dash (`-`), which in Zeek logs typically indicates a missing value or undefined field.
- `datamash -g 1,2 sum 3`—`datamash` is a command-line tool for numeric, textual, and statistical operations. The option `-g 1,2 sum 3` groups the output by the first two fields (originating and responding host IP addresses) and calculates the sum of the third field (bytes sent) for each group.
- `sort -k 3 -rn`—Sorts the output by the numerical value of the third field (sum of `orig_bytes` for each IP address pair) (`-k 3`) in descending order (`-rn`).
- `head -10`—This command displays the top 10 IP address pairs ranked by total bytes sent from originating to responding host.

- Approximately `270 MB` of data was sent to `192.168.151.181`.

## Intrusion Detection With Zeek Example 4: Detecting PsExec

- PsExec, a Sysinternals tool, is commonly used for remote administration in Active Directory environments, but its powerful capabilities also make it a favorite preference of adversaries for remote code execution attacks.

- To illustrate a typical attack sequence, let's consider this:
  - An attacker transfers the binary file `PSEXESVC.exe` to a target machine using the `ADMIN$` share, a special shared folder used in Windows networks, via the SMB (Server Message Block) protocol.
  - Following this, the attacker remotely launches this file as a temporary service by utilizing the `IPC$` share, another special shared resource that enables Inter-Process Communication.

- We can identify SMB transfers and the typical use of `PsExec` using Zeek's `smb_files.log`, `dce_rpc.log`, and `smb_mapping.log`.

---
