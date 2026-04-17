---
title: "Snort"
date: 2026-04-17T14:34:05+02:00
draft: false
toc: false
images:
tags:
  - untagged
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
![Snort_high-level_architecture](/Working%20With%20IDS-IPS/attachments/Snort_high-level_architecture.png)

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
![suricata_example_2_1](/Working%20With%20IDS-IPS/attachments/suricata_example_2_1.png)
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
![snort_example_3_1.png](/Working%20With%20IDS-IPS/attachments/snort_example_3_1.png)

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
![snort_example_4_1.png](/Working%20With%20IDS-IPS/attachments/snort_example_4_1.png)

---
