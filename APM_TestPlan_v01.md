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

Verify that the ZTA/APM appliance meets its published throughput licence capability, remains stable under sustained full-rate load, and preserves service continuity after an HA failover.

## 2. Testing Items
+ Sustain 1 Gbps throughput while serving 500 / 1500 concurrent SSLVPN connections.
+ Sustain the same load continuously for 1 hour over a single/few concurrent SSLVPN connections.
+ Preserves service continuity after an HA failover.

## 3. Test environment

### 3.1 Device under test (DUT)

| ID | CPU | Memory | OS | ZTA/APM license | 
|---|---|---|---|---|
| HW Platform #1 | Hagon CPU 3250 | 64 GB RAM | Kylin | YK-BIG-APM-VE-1G-V23 |
| HW Platform #2 | Hagon CPU 5280 | 64 GB RAM | Kylin | YK-BIG-APM-VE-5G-V23 |

### 3.2 Software / Firmware baseline

| Item | HW Platform #1 | HW Platform #2 |
|---|---|---|
| License model & capacity | YK-BIG-APM-VE-1G-V23 | YK-BIG-APM-VE-5G-V23 |
| ZTA/APM software |  |  |
| Kylin OS version |  |  |
| NIC model / driver / firmware |  |  |
| CPU Model / Cores / Threads |  |  |

### 3.3 Test Tools

| Items | Description | Purpose |
|---|---|---|
| SSLVPN client simulator | Generator capable of establishing SSLVPN tunnels | Build 500 / 1500 concurrent tunnels |
| Traffic generator | iperf3 (multi-stream) or HTTP load tool | Generate 1 Gbps payload through the tunnels |

## 4. High level topology

```
 ┌───────────────────────┐   ┌───────────────────────┐   ┌──────────────────────────┐
 │  SSLVPN Client        │   │  DUT: ZTA/APM.        │   │                          │
 │  Load Generator       │   │  OS: Kylin            │   │                          │
 │  500 / 1500 tunnels   │   │                       │   │                          │
 │                       │   │  HW #1: Hygon 3250    │   │                          │
 │                       ┼───┤                       ├───┼                          │
 │                       │   │  HW #2: Hagon 5280    │   │                          │
 │  Client               │   │                       │   │  Backend Server.         │
 │  (iPerf)              │   │                       │   │  (iperf)                 │
 │                       │   │                       │   │                          │
 └───────────────────────┘   └───────────────────────┘   └──────────────────────────┘
        │                                                          │
        └──────────── SSLVPN tunnels (TLS) ────────────────────────┘
                   payload target: 1 Gbps aggregate
```

## 5. Test items

### 5.1 Test case summary

| Test ID | Category | Platform | Concurrent SSLVPN | Target throughput | Duration |
|---|---|---|---|---|---|
| TP-01 | Throughput | HW Platform #1 (Hagon 3250 / YK-BIG-APM-VE-1G-V23) | 500 | 1 Gbps | 5 min steady state |
| TP-02 | Throughput | HW Platform #2 (Hagon 5280 / YK-BIG-APM-VE-5G-V23) | 500 | 1 Gbps | 5 min steady state |
| TP-03 | Throughput | HW Platform #2 (Hagon 5280 / YK-BIG-APM-VE-5G-V23) | 1500 | 1 Gbps | 5 min steady state |
| ST-01 | Stability | HW Platform #1 (Hagon 3250 / YK-BIG-APM-VE-1G-V23) | 500 | 1 Gbps | 60 min |
| ST-02 | Stability | HW Platform #2 (Hagon 5280 / YK-BIG-APM-VE-5G-V23) | 500 | 1 Gbps | 60 min |
| ST-03 | Stability | HW Platform #2  (Hagon 5280 / YK-BIG-APM-VE-5G-V23)| 1500 | 1 Gbps | 60 min |
| HA-01 | HA Failover | HW Platform #1,#2  | 500 | 1 Gbps | 5 min steady state |

## 6. Result record templates

### 6.1 Throughput test results

| Test ID | Platform | Sessions | Sessions established | Avg throughput (Gbps) | Peak throughput (Gbps) | CPU avg (%) | CPU peak (%) | Mem avg (GB / %) | Mem peak (GB / %) | Log events (E/W/A count) | Result |
|---|---|---|---|---|---|---|---|---|---|---|---|
| TP-01 | #1 | 500 CCU | | | | | | | | | |
| TP-02 | #2 | 500 CCU | | | | | | | | | |
| TP-03 | #2 | 1500 CCU | | | | | | | | | |

### 6.2 Stability test results (1 hour)

| Test ID | Platform | Sessions | Throughput | CPU avg (%) | Memory (%) | Log events | Result |
|---|---|---|---|---|---|---|---|
| ST-01 | #1 | 500 CCU | | | | | | | | | |
| ST-02 | #2 | 500 CCU | | | | | | | | | |
| ST-03 | #2 | 1500 CCU | | | | | | | | | |

### 6.3 HA Failover test results
| Test ID | Platform | Log events | Result |
|---|---|---|---|
| HA-01 | #1, #2 | | |

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
````
