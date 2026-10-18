# Xinchuang AWAF Test Plan

|  |  |
|---|---|
| Document title | Xinchuang AWAF Throughput and Stability Test Plan |
| Version | 1.0 |
| Date | 2026-10-15 |
| Operating system | **Kylin OS** |
| SSL Accerlation | **No** |
| HW Platform | **Hygon CPU (3250 / 5280)** |

### Revision history

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-10-15 | Boris | 1st draft |


---

## 1. Purpose

Verify that the AWAF appliance meets its 
1. **published throughput licence capability**,
2. **remains stable under sustained full-rate load**, and
3. **preserves service continuity after an HA failover**.

## 2a. Testing Items - Performance & Stabliity
1. Sustain **1 or 3 Gbps throughput**.
2. Sustain the same **load continuously for 1 hour** over a single/few concurrent SSLVPN connections.
3. Preserves service continuity after an **HA failover**.

## 2b. Testing Items - General Use Cases
1. AWAF for JSON (transparent mode)
> + Enable all signatures
> > + Security Policy : [Download](./IRDTAXP_policy01_ipv4_443_enable_all_signature_add_json_profile.xml)
> + Disable all signature
> > + Security Policy : IRDTAXP_policy01_ipv4_443_disable_all_signature_add_json_profile
2. AWAF for JSON (blocking mode)
> + Enable all signatures
> > + Security Policy : IRDTAXP_policy01_ipv4_443_enable_all_signature_add_json_profile
> + Optimized signatures only
> > + Security Policy : IRDTAXP_policy01_ipv4_443_optimized_add_json_profile
> + No json signature
> > + Security Policy : IRDTAXP_policy01_ipv4_443_no_json_add_json_profile
> + Disable all signature
> > + Security Policy : IRDTAXP_policy01_ipv4_443_disable_all_signature_add_json_profile
3. AWAF Event logs
> + GUI Requests Log
> + Syslog
4. Attack Signatures Update
> + AWAF/ASM Attack Signatures
> + Bot Signatures
> + Browser Challenges
> + Server Technologies
> + Threat Campaigns
5. Packet Capture
> + 


## 3. Test environment

### 3.1 Device under test (DUT)

| ID | CPU | Memory | OS | AWAF license | 
|---|---|---|---|---|
| HW Platform #1 | Hygon **CPU 3250** | 64 GB RAM | Kylin | YK-BIG-AWF-VE-**1G**-V23 |
| HW Platform #2 | Hygon **CPU 5280** | 64 GB RAM | Kylin | YK-BIG-AWF-VE-**3G**-V23 |

### 3.2 Software / Firmware baseline

| Item | HW Platform #1 | HW Platform #2 |
|---|---|---|
| License model & capacity | YK-BIG-AWF-VE-**1G**-V23 | YK-BIG-AWF-VE-**3G**-V23 |
| AWAF software version | 17.5.1.x | 17.5.1.x |
| Kylin OS version |  |  |
| NIC model / driver / firmware |  |  |
| CPU Model / Cores / Threads |  |  |

### 3.3 Test Tools & Environment

| Items | Description | Purpose |
|---|---|---|
| Test Client | HTTPs load tool or apache benchmark tool (ab) | Generate 1Gbps or 3 Gbps HTTPs payload |
| Test Server | HTTPs load tool or apache benchmark tool (ab) | Generate 1Gbps or 3 Gbps HTTPs payload |

## 4. High level topology

```
                             ┌───────────────────────┐   ┌──────────────────────────┐
                             │  DUT: ZTA/APM         │   │                          │
                             │  HW #1: Hygon 3250    │   │                          │
                             │  HW #1: Hygon 5280    │   │                          │
 ┌───────────────────────┐   │                       │   │  Testing Web Server(s)   │
 │ Testing Client(s)     ┼───┤ 1G/10G                ├───┼  1G/10G                  │
 │ (Apache benchmark)    │   │                       │   │                          │
 └───────────────────────┘   └───────────────────────┘   └──────────────────────────┘
```

## 5. Test items

### 5.1 Test case summary

| Test ID | Category | HW Platform | Enforcement Mode | Target throughput | Duration |
|---|---|---|---|---|---|
| TP-01a | Throughput | HW #1 (Hygon 3250 / YK-BIG-AWF-VE-**1G**-V23) | Transparent | 1 Gbps | 5 min |
| TP-01b | Throughput | HW #1 (Hygon 3250 / YK-BIG-AWF-VE-**1G**-V23) | Blocking | 1 Gbps | 5 min |
| TP-02a | Throughput | HW #2 (Hygon 5280 / YK-BIG-AWF-VE-**3G**-V23) | Transparent | 3 Gbps | 5 min |
| TP-02b | Throughput | HW #2 (Hygon 5280 / YK-BIG-AWF-VE-**3G**-V23) | Blocking | 3 Gbps | 5 min |
| ST-01 | Stability | HW #1 (Hygon 3250 / YK-BIG-AWF-VE-**1G**-V23) | Blocking | 1 Gbps | 60 min |
| ST-02 | Stability | HW #2 (Hygon 5280 / YK-BIG-AWF-VE-**3G**-V23) | Blocking | 3 Gbps | 60 min |
| HA-01 | HA Failover | HW #1,#2  | Blocking | 1 Gbps | 5 min |

## 6. Result record templates

### 6.1 Throughput test results

| Test ID | HW Platform | Throughput (Gbps) | Ctrl CPU Avg (%) | Ctrl CPU Peak (%) | Sys CPU Avg (%) | Sys CPU Peak (%) | Mem Avg (%) | Mem Peak (%) | Result |
|---|---|---|---|---|---|---|---|---|---|
| TP-01a | #1 | | | | | | | | |
| TP-01b | #1 | | | | | | | | |
| TP-02a | #2 | | | | | | | | |
| TP-02b | #2 | | | | | | | | |

### 6.2 Stability test results (1 hour)

| Test ID | Platform | Throughput | Ctrl CPU avg (%) | Sys CPU avg (%) | Memory (%) | Result |
|---|---|---|---|---|---|---|
| ST-01 | #1 | | | | | | | | | |
| ST-02 | #2 | | | | | | | | | |

### 6.3 HA Failover test results
| Test ID | Platform | Result |
|---|---|---|
| HA-01 | #1, #2 | |

### 6.4 Log / alert / warning / error record

| # | Timestamp | Test ID | Source (System / SSLVPN / Kernel) | Severity | Message |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |

## 7. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Test engineer | | | |
| Test lead / QA | | | |
| Product owner | | | |

## 8. CCU Generator
Video demo “Yunke-CCU-Test_v1.mp4”
> + ```https://fileupload.f5xc.website/index.php/s/vK2HblXM3FRHBMp```

Create a Linux Virtual Machine.
> + ```e.g. Ubuntu-24.04-live-server-amd64.iso image```

Install Docker on the Linux Virtual Machine.
> + ```https://docs.docker.com/engine/install/```

Download “yunke-ccu-v6.tar.zip” image to your docker machine
> + ```https://fileupload.f5xc.website/index.php/s/K1W8BcCDDTj7dz4```

Decompress & Import yunke-ccu-v6.tar.
> + ```decompress yunke-ccu-v6.tar.zip```
> + ```docker load –input yunke-ccu-v6.tar```

Create a Virtual-Server “https://192.168.100.200” for SSLVPN testing.  Set SSLVPN idle timeout to 7200.
> + ```The CCU generator hardcoded with this IP and VPN credential (username/password = yunke)```

Start testing by spin up 505+ or 1505+ containers.
> + ```for i in {1..505}; do echo "### $i ###"; docker run -it yunke-ccu:v6; done``` (May need 45 mins)
> + ```for i in {1..1505}; do echo "### $i ###"; docker run -it yunke-ccu:v6; done``` (May need 45x3 mins)

SSH to Yunke device and keep monitoring VPN connection status.
> + ```watch tmsh show apm license```

SSH to Yunke device and keep monitoring /var/log/apm.
> + ```tail -f /var/log/apm | egrep -i 'license'```

Remove all containers which created by previous test. 
> + ```docker container prune```
