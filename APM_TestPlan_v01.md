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

## 2. Testing Scope
1. Sustain 1 Gbps throughput while serving 500 / 1500 concurrent SSLVPN connections on the two target hardware platforms.
2. Sustain the same load continuously for 1 hour without performance degradation, resource leak, session loss, or abnormal log/alert/warning/error events.
3. Preserves service continuity after an HA failover

## 3. Test environment

### 3.1 Device under test (DUT)

| ID | CPU | Memory | OS | ZTA/APM license | 
|---|---|---|---|---|
| HW Platform #1 | Hagon CPU 3250 | 64 GB RAM | Kylin | YK-BIG-APM-VE-1G-V23 |
| HW Platform #2 | Hagon CPU 5280 | 64 GB RAM | Kylin | YK-BIG-APM-VE-5G-V23 |

### 3.2 Software / firmware baseline

| Item | HW Platform #1 | HW Platform #2 |
|---|---|---|
| License model & capacity | YK-BIG-APM-VE-1G-V23 | YK-BIG-APM-VE-5G-V23 |
| ZTA/APM software | _record_ | _record_ |
| Kylin OS version | _record_ | _record_ |
| NIC model / driver / firmware | _record_ | _record_ |
| CPU cores / threads, CPU governor | _record_ | _record_ |

### 3.3 Supporting equipment

| Role | Description | Purpose |
|---|---|---|
| SSLVPN client simulator | Load generator capable of establishing 500 / 1500 concurrent SSLVPN tunnels (e.g. traffic generator appliance or a farm of Linux hosts running the APM SSLVPN client in headless mode) | Build concurrent tunnel load |
| Traffic generator / backend server | iperf3 (multi-stream) or HTTP load tool on the protected (internal) network | Generate 1 Gbps payload through the tunnels |
| Switch | 1 GbE / 10 GbE non-blocking L2 switch, no oversubscription on test paths | Connect client side and server side |
| Management / monitoring host | Collects CPU/memory samples, syslog, SNMP, test logs | Data collection and evidence |
| Time source | Common NTP server for DUT, generators and monitoring host | Correlate metrics with log timestamps |

### 3.4 Monitoring & data collection

| Metric | Source / method | Sampling interval |
|---|---|---|
| CPU loading (total, per-core, user/sys/si-softirq) | `top -b`, `mpstat -P ALL 10`, `sar -u`, APM CLI/GUI dashboard | 10 s |
| Memory usage (used, free, cached, swap) | `free -m`, `sar -r`, `vmstat 10`, APM CLI/GUI dashboard | 10 s |
| Load average | `uptime`, `sar -q` | 10 s |
| Throughput (Rx/Tx per interface, aggregate) | Traffic generator report + `sar -n DEV 10` on DUT | 10 s |
| Concurrent SSLVPN session count | APM session/statistics page or CLI | 1 min |
| Interface errors / drops | `ip -s link`, `ethtool -S` | Start & end of each run |
| Connection setup success/failure, disconnects | Load generator report + APM logs | Continuous |
| Logs / alerts / warnings / errors | APM system & SSLVPN logs, `journalctl -p warning`, `dmesg`, syslog forwarded to monitoring host | Continuous (reviewed after each run) |

## 4. High level topology

```
                        ┌──────────────────────────────┐
                        │   Monitoring / Mgmt Host     │
                        │  (metrics, syslog, results)  │
                        └───────────────┬──────────────┘
                                        │ Mgmt network
                                        │
 ┌───────────────────────┐   ┌───────────┴───────────┐   ┌──────────────────────────┐
 │  SSLVPN Client        │   │        DUT: APM       │   │  Backend / Traffic       │
 │  Load Generator       │   │  OS: Kylin            │   │  Server                  │
 │  500 / 1500 tunnels   │   │                       │   │  (iperf3 / HTTP server)  │
 │                       │   │  HW #1: Hagon 3250,   │   │                          │
 │  ── untrusted side ───┼───┤   64GB, YK-BIG-APM-   ├───┼─── trusted side ──       │
 │      (WAN / public)   │ L2│   VE-1G-V23           │ L2│      (LAN / internal)    │
 │                       │ SW│  HW #2: Hagon 5280,   │ SW│                          │
 │                       │   │   64GB, YK-BIG-APM-   │   │                          │
 │                       │   │   VE-5G-V23           │   │                          │
 └───────────────────────┘   └───────────────────────┘   └──────────────────────────┘
        │                                                          │
        └──────────── SSLVPN tunnels (TLS) ────────────────────────┘
                   payload target: 1 Gbps aggregate
```

| Link | Segment | Speed | Notes |
|---|---|---|---|
| Client LG → Switch A → DUT untrusted port | Encrypted SSLVPN traffic | ≥ 1 GbE (10 GbE preferred to avoid link saturation) | Carries tunnel + TLS overhead |
| DUT trusted port → Switch B → Backend server | Clear-text payload | ≥ 1 GbE (10 GbE preferred) | Carries decrypted payload |
| DUT / LG / server → Mgmt switch | Management | 1 GbE | Out-of-band; must not carry test traffic |

> Note: if the physical link is exactly 1 GbE, the tunnel overhead (TLS + encapsulation) makes a 1 Gbps *payload* result unreachable on the encrypted side. Use 10 GbE ports on the untrusted side, or define 1 Gbps as line-rate including overhead — see Section 9, Assumption A2.

## 5. Test configuration / parameters

| Parameter | Value |
|---|---|
| Tunnel type | SSLVPN (TLS), default cipher suite of the APM build |
| Authentication | Local user database (bulk-provisioned test accounts), no MFA |
| Concurrency levels | 500 (Platform #1 & #2), 1500 (Platform #2) |
| Target aggregate throughput | 1 Gbps (bidirectional profile as defined per test case) |
| Traffic profile | TCP, iperf3 multi-stream, payload 1400 B (fits tunnel MTU without fragmentation) |
| Per-tunnel rate (500 sessions) | ~2 Mbps |
| Per-tunnel rate (1500 sessions) | ~0.68 Mbps |
| Ramp-up | 50 connections/s until target concurrency reached |
| Hold / measurement time (throughput test) | 5 minutes steady state after ramp-up |
| Hold / measurement time (stability test) | 60 minutes steady state |
| Logging level on DUT | Default/production level (do not raise to debug — it perturbs performance) |

## 6. Test items

### 6.1 Test case summary

| Test ID | Category | Platform | Concurrent SSLVPN | Target throughput | Duration | Ref. |
|---|---|---|---|---|---|---|
| TC-TP-01 | Throughput | HW Platform #1 (Hagon 3250 / YK-BIG-APM-VE-1G-V23) | 500 | 1 Gbps | 5 min steady state | §6.3 |
| TC-TP-02 | Throughput | HW Platform #2 (Hagon 5280 / YK-BIG-APM-VE-5G-V23) | 500 | 1 Gbps | 5 min steady state | §6.3 |
| TC-TP-03 | Throughput | HW Platform #2 | 1500 | 1 Gbps | 5 min steady state | §6.3 |
| TC-ST-01 | Stability | HW Platform #1 | 500 | 1 Gbps | 60 min | §6.4 |
| TC-ST-02 | Stability | HW Platform #2 | 500 | 1 Gbps | 60 min | §6.4 |
| TC-ST-03 | Stability | HW Platform #2 | 1500 | 1 Gbps | 60 min | §6.4 |

### 6.2 Common pre-conditions

| # | Pre-condition |
|---|---|
| 1 | DUT installed with the baseline Kylin OS + APM build; versions recorded per §3.2 |
| 2 | Correct license applied and validated (1G model on Platform #1, 5G model on Platform #2) |
| 3 | Enough test user accounts provisioned (≥ 1600) and SSLVPN portal/profile configured |
| 4 | Interfaces up at expected speed/duplex; no interface errors before start |
| 5 | NTP synchronised on DUT, load generator, backend server, monitoring host |
| 6 | Baseline idle measurement taken: CPU %, memory usage, session count = 0 |
| 7 | Logs rotated/cleared (or start timestamp noted) so only test-window events are analysed |
| 8 | Monitoring collectors started before traffic starts, stopped after traffic stops |

### 6.3 Throughput test procedure (TC-TP-01 … TC-TP-03)

| Step | Action | Expected result / data to record |
|---|---|---|
| 1 | Record idle baseline CPU %, memory usage, interface counters | Baseline captured |
| 2 | Start metric collectors (`mpstat`, `sar`, `free`, interface stats) at 10 s interval | Collection running |
| 3 | Ramp up SSLVPN connections to the target count (500 or 1500) at 50 conn/s | All tunnels established; login/tunnel failure count = 0; session count on APM matches target |
| 4 | Start payload traffic and raise aggregate rate to 1 Gbps | Traffic ramps without tunnel drops |
| 5 | Hold steady state for 5 minutes | Aggregate throughput sustained at target |
| 6 | Sample and record CPU loading and memory usage throughout | Avg / peak CPU %, avg / peak memory usage |
| 7 | Record throughput (Rx/Tx, aggregate), packet loss, retransmissions, latency if available | Values recorded |
| 8 | Record concurrent session count at start / mid / end of steady state | Count stable at target |
| 9 | Stop traffic, disconnect all tunnels | Clean teardown; session count returns to 0 |
| 10 | Collect interface error/drop counters | No new errors/drops |
| 11 | Export and review DUT logs for the test window (system, SSLVPN, kernel) | Record every alert / warning / error with timestamp and text; expected: none of severity ≥ warning related to the DUT under load |
| 12 | Allow the DUT to idle 5 minutes, then re-record CPU/memory | Resources return to near baseline |

### 6.4 Stability test procedure (TC-ST-01 … TC-ST-03)

Same setup as the corresponding throughput case, with the steady state extended to **60 minutes**.

| Step | Action | Expected result / data to record |
|---|---|---|
| 1 | Repeat §6.3 steps 1–4 to reach target concurrency and 1 Gbps | Load established |
| 2 | Hold steady state for 60 minutes without intervention | No tunnel drops, no throughput degradation |
| 3 | Record CPU loading and memory usage continuously (10 s samples); report per-10-minute averages and overall avg/peak | Trend table completed (§8.3) |
| 4 | Record throughput per 10-minute interval | Sustained ≥ target throughout |
| 5 | Record concurrent session count per 10-minute interval | Remains at target (no silent disconnect/re-login churn) |
| 6 | Monitor logs live and export at end of run | All alerts / warnings / errors captured with timestamps |
| 7 | At end of run, check memory trend for leak (compare 10 min vs 60 min mark) | Memory growth within tolerance (see §7) |
| 8 | Stop traffic, disconnect all tunnels, verify clean teardown and resource release | Sessions = 0; CPU/memory return to near baseline |
| 9 | Verify DUT remains fully manageable (GUI/CLI responsive, no restart of services) | No crash, no service restart, no reboot, uptime unchanged |

## 7. Pass / fail criteria

| # | Criterion | Threshold | Applies to |
|---|---|---|---|
| PF-1 | All targeted SSLVPN connections established and maintained | 100% of 500 / 1500; 0 unexpected disconnects | All cases |
| PF-2 | Sustained aggregate throughput | ≥ 1 Gbps (≥ 95% of target, i.e. ≥ 0.95 Gbps, accepted as pass with note) | All cases |
| PF-3 | Packet loss on payload traffic | ≤ 0.01% | All cases |
| PF-4 | CPU loading during steady state | Average ≤ 80%, peak ≤ 90% (recorded in all cases; treated as observation if design target differs) | All cases |
| PF-5 | Memory usage during steady state | ≤ 80% of 64 GB | All cases |
| PF-6 | Memory leak | Growth between the 10-minute and 60-minute marks ≤ 2% of total RAM, with no monotonic upward trend | Stability cases |
| PF-7 | Throughput degradation over the 1-hour run | Last 10-minute average ≥ 98% of first 10-minute average | Stability cases |
| PF-8 | Logs | No `error` / `critical` / `alert` severity event attributable to the DUT during the test window. Warnings are recorded and assessed; any warning indicating resource exhaustion, session drop or crypto failure is a fail | All cases |
| PF-9 | Stability / availability | No service crash, service restart, watchdog event or reboot; management plane responsive throughout | All cases |
| PF-10 | Interface counters | No increment of CRC/error/drop counters on test interfaces | All cases |

A test case is **PASS** only when every applicable criterion is met; otherwise **FAIL**, with observed values, logs and the failing criterion recorded.

## 8. Result record templates

### 8.1 Environment record (one per platform)

| Item | Value |
|---|---|
| Platform ID / model | |
| APM build / license | |
| Kylin version / kernel | |
| Test date & tester | |

### 8.2 Throughput test results

| Test ID | Platform | Sessions | Sessions established / target | Avg throughput (Gbps) | Peak throughput (Gbps) | Packet loss (%) | CPU avg (%) | CPU peak (%) | Mem avg (GB / %) | Mem peak (GB / %) | Log events (E/W/A count) | Result |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| TC-TP-01 | #1 | 500 | | | | | | | | | | |
| TC-TP-02 | #2 | 500 | | | | | | | | | | |
| TC-TP-03 | #2 | 1500 | | | | | | | | | | |

### 8.3 Stability test results (1 hour) — per test case

| Elapsed | Sessions | Throughput (Gbps) | CPU (%) | Memory (GB / %) | Load avg | Drops / errors | Log events |
|---|---|---|---|---|---|---|---|
| 00:00 (baseline, idle) | 0 | 0 | | | | | |
| 00:10 | | | | | | | |
| 00:20 | | | | | | | |
| 00:30 | | | | | | | |
| 00:40 | | | | | | | |
| 00:50 | | | | | | | |
| 01:00 | | | | | | | |
| Post-run idle (+5 min) | 0 | 0 | | | | | |
| **Avg / Peak** | | | | | | | |

### 8.4 Log / alert / warning / error record

| # | Timestamp | Platform / Test ID | Source (system / SSLVPN / kernel) | Severity | Message | Impact assessment | Disposition (accept / defect ID) |
|---|---|---|---|---|---|---|---|
| 1 | | | | | | | |

### 8.5 Defect summary

| Defect ID | Test ID | Severity | Description | Status |
|---|---|---|---|---|
| | | | | |

## 9. Assumptions, dependencies and risks

### 9.1 Assumptions

| ID | Assumption | Action if invalid |
|---|---|---|
| A1 | "1 Gbps throughput" means aggregate payload throughput through the SSLVPN tunnels, measured bidirectionally as the sum of Rx+Tx at the DUT, using TCP traffic | Confirm the measurement definition (payload vs line rate, unidirectional vs bidirectional) before execution and update §5 |
| A2 | Test-side links are 10 GbE (or 1 GbE is accepted as line-rate-limited), so TLS/encapsulation overhead does not cap the payload result | Re-cable to 10 GbE or restate the target as "1 Gbps including tunnel overhead" |
| A3 | Platform #1 carries a 1 Gbps-class license (YK-BIG-APM-VE-1G-V23), so 1 Gbps is at the licensed ceiling and CPU headroom may be minimal | Record CPU as an observation rather than a hard fail on Platform #1 if the vendor spec allows high utilisation at rated throughput |
| A4 | The load generator can open 1500 concurrent SSLVPN tunnels and is not itself the bottleneck | Validate generator capacity in a pre-test dry run; scale out client hosts if needed |
| A5 | Default (production) logging level is used for all runs | Any deviation is recorded with the result |

### 9.2 Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Load generator becomes the bottleneck (CPU/TLS on client side) | Understated DUT throughput | Dry-run calibration; monitor generator CPU; distribute across multiple hosts |
| 1 GbE link saturation with tunnel overhead | Target unreachable | Use 10 GbE on untrusted side (A2) |
| Licence capacity or session limit reached at 1500 sessions | Test blocked | Verify licensed session/throughput limits before execution |
| Debug-level logging left enabled | Inflated CPU, disk fill, skewed results | Verify log level in pre-conditions |
| Clock skew between DUT and monitoring host | Cannot correlate logs with metric spikes | NTP enforced (pre-condition 5) |
| Shared lab network carrying other traffic | Noisy results | Dedicated isolated test VLANs / switches |

### 9.3 Entry / exit criteria

| Type | Criteria |
|---|---|
| Entry | Build installed and licensed on both platforms; topology cabled and verified; dry run of 10 sessions + 100 Mbps passes; monitoring pipeline verified |
| Exit | All 6 test cases executed; CPU/memory recorded for each; all log events triaged; results and defects documented; test report reviewed and signed off |

## 10. Schedule (indicative)

| Activity | Duration |
|---|---|
| Environment setup, baseline recording, dry run | 0.5 day |
| TC-TP-01 … TC-TP-03 (throughput, incl. setup/teardown per case) | 0.5 day |
| TC-ST-01 … TC-ST-03 (3 × 1 h soak + setup/teardown + log review) | 1 day |
| Data consolidation, log triage, report | 0.5 day |
| **Total** | **~2.5 days** |

## 11. Deliverables

| Deliverable | Content |
|---|---|
| Test report | Completed tables from §8, pass/fail per case, conclusions |
| Raw metric files | `mpstat` / `sar` / `free` / `vmstat` / interface counter logs per case |
| Traffic generator reports | Throughput, loss, session establishment statistics |
| DUT log archives | System, SSLVPN and kernel logs covering each test window |
| Screenshots | APM dashboard session count and resource graphs per case |

## 12. Sign-off

| Role | Name | Signature | Date |
|---|---|---|---|
| Test engineer | | | |
| Test lead / QA | | | |
| Product owner | | | |
````

**Notes on choices I made (please confirm or correct):**

1. **Throughput definition** — I defined 1 Gbps as aggregate payload throughput (Rx+Tx at the DUT) with TCP traffic. This is the single biggest variable in this plan; unidirectional vs bidirectional, and payload vs line rate, can change results by ~2×. See Assumption A1.
2. **Link speed** — At exactly 1 GbE on the encrypted side you can't reach 1 Gbps of payload because of TLS/encapsulation overhead, so I recommended 10 GbE test links (Assumption A2 / topology note).
3. **Platform #1 vs license** — `YK-BIG-APM-VE-1G-V23` looks like a 1 Gbps-class license, so TC-TP-01/TC-ST-01 run right at the rated ceiling; I softened the CPU criterion there via Assumption A3.
4. **Thresholds** (CPU ≤ 80% avg, memory ≤ 80%, loss ≤ 0.01%, ≤ 2% memory growth) are industry-typical defaults — replace them with your product spec numbers if you have them.
5. I also added **TC-ST-01** (1-hour soak on Platform #1 at 500 sessions) since "run the above test for 1 hour" implies all three throughput configurations; drop it if only Platform #2 needs soaking.

I kept this in the chat rather than writing a file, per this workspace's convention. Say the word and I'll save it as e.g. `apm-test-plan/APM_Throughput_Stability_Test_Plan.md`, or fold in your real thresholds, tool names and a Mermaid version of the topology diagram.
