# ZTA/APM Test Plan

|  |  |
|---|---|
| Document title | ZTA/APM Throughput and Stability Test Plan |
| Version | 1.0 (Draft) |
| Date | 2026-09-17 |
| Operating system | Kylin OS |
| SSL Accerlation | No |
| HW Platform | Xinchuang Hygon CPU |

### Revision history

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-09-17 | Boris | Initial draft |

---

## 1. Purpose

Verify that the ZTA/APM appliance meets its **published throughput licence capability**, **remains stable under sustained full-rate load**, and **preserves service continuity after an HA failover**.

## 2. Testing Items
1. Sustain **1 Gbps throughput** while serving **500 / 1500 concurrent SSLVPN connections**.
2. Sustain the same **load continuously for 1 hour** over a single/few concurrent SSLVPN connections.
3. Preserves service continuity after an **HA failover**.

## 3. Test environment

### 3.1 Device under test (DUT)

| ID | CPU | Memory | OS | ZTA/APM license | 
|---|---|---|---|---|
| HW Platform #1 | Hygon **CPU 3250** | 64 GB RAM | Kylin | YK-APM-VE-**1G**-V23 |
| HW Platform #2 | Hygon **CPU 5280** | 64 GB RAM | Kylin | YK-APM-VE-**5G**-V23 |

### 3.2 Software / Firmware baseline

| Item | HW Platform #1 | HW Platform #2 |
|---|---|---|
| License model & capacity | YK-APM-VE-**1G**-V23 | YK-APM-VE-**5G**-V23 |
| ZTA/APM software version |  |  |
| Kylin OS version |  |  |
| NIC model / driver / firmware |  |  |
| CPU Model / Cores / Threads |  |  |

### 3.3 Test Tools & Environment

| Items | Description | Purpose |
|---|---|---|
| Linux with docker | For SSLVPN client simulator |  |
| SSLVPN client simulator | Generator capable of establishing SSLVPN tunnels | Build 500 / 1500 concurrent tunnels |
| Linux Client | iperf3 or HTTP load tool | Generate 1 Gbps payload through a SSLVPN tunnel |
| Linux Server | iperf3 or HTTP load tool | Generate 1 Gbps payload through a SSLVPN tunnel |

## 4. High level topology

```
 ┌───────────────────────┐   ┌───────────────────────┐   ┌──────────────────────────┐
 │ Container Based       │   │                       │   │                          │
 │ SSLVPN Conn Generator ┼───┤  DUT: ZTA/APM         │   │                          │
 │ 500 / 1500 tunnels    │   │  OS: Kylin            │   │                          │
 ┌───────────────────────┐   │                       │   │                          │
 │ Linux with Docker     │   │  HW #1: Hygon 3250    │   │                          │
 └───────────────────────┘   │                       ├───┼                          │
 ┌───────────────────────┐   │  HW #2: Hygon 5280    │   │                          │
 │                       │   │                       │   │                          │
 │ Linux Client(s)       ┼───┤                       │   │  Backend Server(s)       │
 │ (iPerf)               │   │                       │   │  (iperf)                 │
 └───────────────────────┘   └───────────────────────┘   └──────────────────────────┘
        │                                                          │
        └────────────────────── SSLVPN tunnels (TLS) ──────────────┘
                                  1 Gbps aggregate
```

## 5. Test items

### 5.1 Test case summary

| Test ID | Category | HW Platform | Concurrent SSLVPN | Target throughput | Duration |
|---|---|---|---|---|---|
| TP-01 | Throughput | HW #1 (Hygon 3250 / YK-APM-VE-1G-V23) | 500 | 1 Gbps | 5 min |
| TP-02 | Throughput | HW #2 (Hygon 5280 / YK-APM-VE-5G-V23) | 500 | 1 Gbps | 5 min |
| TP-03 | Throughput | HW #2 (Hygon 5280 / YK-APM-VE-5G-V23) | 1500 | 1 Gbps | 5 min |
| ST-01 | Stability | HW #1 (Hygon 3250 / YK-APM-VE-1G-V23) | 500 | 1 Gbps | 60 min |
| ST-02 | Stability | HW #2 (Hygon 5280 / YK-APM-VE-5G-V23) | 500 | 1 Gbps | 60 min |
| ST-03 | Stability | HW #2  (Hygon 5280 / YK-APM-VE-5G-V23)| 1500 | 1 Gbps | 60 min |
| HA-01 | HA Failover | HW #1,#2  | 500 | 1 Gbps | 5 min |

## 6. Result record templates

### 6.1 Throughput test results

| Test ID | HW Platform | Sessions Established | Avg Throughput (Gbps) | Peak Throughput (Gbps) | CPU Avg (%) | CPU Peak (%) | Mem Avg (%) | Mem Peak (%) | Result |
|---|---|---|---|---|---|---|---|---|---|
| TP-01 | #1 | 500 CCU | | | | | | | |
| TP-02 | #2 | 500 CCU | | | | | | | |
| TP-03 | #2 | 1500 CCU | | | | | | | |

### 6.2 Stability test results (1 hour)

| Test ID | Platform | Sessions | Throughput | CPU avg (%) | Memory (%) | Result |
|---|---|---|---|---|---|---|
| ST-01 | #1 | 500 CCU | | | | | | | | |
| ST-02 | #2 | 500 CCU | | | | | | | | |
| ST-03 | #2 | 1500 CCU | | | | | | | | |

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

## 8. CCU Simulator
+ Video demo “Yunke-CCU-Test_v1.mp4”
+ Download “yunke-ccu-v6.tar.zip” image to your docker machine
+ Create a Linux Virtual Machine
 + I used Ubuntu-24.04-live-server-amd64.iso image
+ Install Docker on the Linux Virtual Machine
 + https://docs.docker.com/engine/install/
+ Decompress & Import yunke-ccu-v6.tar
```
decompress yunke-ccu-v6.tar.zip
docker load –input yunke-ccu-v6.tar
```
 
+ Create a SSLVPN account on Yunke device
+ create a Virtual-Server “https://192.168.100.200” for SSLVPN testing (The SSLVPN script is hardcode with this IP)
 + username/password = yunke
 + the container image hardcoded the above IP and VPN credential
 
+ SSH to Yunke device and keep monitoring VPN connection status
```watch tmsh show apm license```
 
+ SSH to Yunke device and keep monitoring /var/log/apm
```tail -f /var/log/apm | egrep -i ‘license’```
 
+ Remove all containers which created by previous test
```docker container prune```
 
+ Start testing by spin up 600+ containers
```for i in {1..600}; do echo "### $i ###"; docker run -it yunke-ccu:v6; done```
