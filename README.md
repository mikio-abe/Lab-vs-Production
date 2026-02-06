# Lab vs Production

Differences between this verification lab and production SASE/SD-WAN deployments.

---

## 🔬Overview

This lab is designed for **concept verification and skill demonstration**, not production replication. Key simplifications were made to focus on overlay behavior rather than infrastructure scalability.

**【日本語サマリ】**

本ラボは概念検証とスキル実証を目的としており、本番環境の完全再現ではありません。<BR>
インフラのスケーラビリティではなく、Overlay動作の検証に焦点を当てています。

---

## 📊Comparison Summary

| Component | Lab | Production |
|-----------|-----|------------|
| **Site-to-Site (SASE)** | WireGuard (UDP 4960) | BGP over IPsec (Magic WAN) |
| **ZTNA Topology** | POP as Aggregation Node | Direct WARP on Endpoints |
| **MPLS Core** | PE direct connect | P routers + Route Reflectors |
| **FortiGate License** | Evaluation (10 policy limit) | Full license |
| **Brownout** | tc command (latency injection) | Natural occurrence |
| **Path Priority** | SASE primary (for demo) | MPLS primary (typical) |

---

## 🔗Site-to-Site Connectivity

### Lab: WireGuard

```
POP1 ─── WireGuard (UDP 4960) ─── POP2
　　　　　　Linux to Linux
```

- Linux POP cannot run BGP over IPsec natively
- WireGuard provides simple UDP tunnel
- Sufficient for carrying SD-WAN IPsec traffic

### Production: BGP over IPsec

```
Site A ────── BGP over IPsec ────── Site B
           (Magic WAN / Tunnel)
```

- Dynamic route exchange between sites
- Cloudflare Magic WAN or similar service
- Native BGP support on edge routers

**【日本語サマリ】**

ラボではLinux POPでBGP over IPsecが使用できないためWireGuardで代替。本番ではMagic WAN等でBGP over IPsecを使用し、動的ルート交換を実現。

---

## 🏗MPLS Infrastructure

### Lab: Simplified Topology

```
CE1 ────── PE1 ═══════ PE2 ────── CE2
       　 (direct connection)
```

- No P (Provider) routers
- No Route Reflectors
- PE-to-PE direct iBGP

### Production: Full Topology

```
CE1 ─── PE1 ─── P ─── P ─── PE2 ─── CE2
                 \   /
                  RR
```

- P routers for core scalability
- Route Reflectors for BGP scalability
- RSVP-TE for traffic engineering (optional)
- QoS policies for traffic classification

**【日本語サマリ】**

ラボではPルータとRoute Reflectorを省略しPE直結構成。本番ではPルータでコアスケーラビリティ、RRでBGPスケーラビリティを確保。

---

## 🔧FortiGate Configuration

### Lab: Evaluation License

| Limitation | Impact |
|------------|--------|
| 10 Firewall Policies | Policy consolidation required |
| Limited features | Some features restricted |

Workaround: Combined policies using sdwan-zone instead of individual interfaces.

### Production: Full License

- Unlimited policies
- Full feature set
- HA (High Availability) support

**【日本語サマリ】**

ラボでは評価版ライセンス（Policy 10個制限）のため、sdwan-zoneを使用したポリシー統合で対応。本番では制限なし。

---

## ⚡Brownout Simulation

### Lab: Manual Injection

```bash
# Add 300ms latency to SASE path
tc qdisc add dev eth0 root netem delay 300ms
```

- Predictable timing for demonstration
- Controlled degradation level
- Repeatable test conditions

### Production: Natural Occurrence

- ISP congestion
- Network equipment issues
- Weather/physical damage
- Unpredictable timing and severity

**【日本語サマリ】**

ラボではtcコマンドで遅延を注入しBrownoutをシミュレート。本番ではISP輻輳、機器障害、物理的損傷等で自然発生。

---

## 🔀Path Priority Design

### Lab: SASE Primary

| Priority | Path | Reason |
|----------|------|--------|
| Primary | SASE | Brownout demo target |
| Backup | MPLS | Failover destination |

SASE path is primary so that latency injection triggers visible failover to MPLS.

### Production: MPLS Primary (Typical)

| Priority | Path | Reason |
|----------|------|--------|
| Primary | MPLS | Low latency, SLA guaranteed |
| Backup | Internet/SASE | Cost-effective backup |

MPLS typically preferred for enterprise traffic due to predictable performance.

**【日本語サマリ】**

ラボではBrownoutデモのためSASEをプライマリに設定。本番では通常MPLSをプライマリとし、低遅延・SLA保証を活用。

---

## 🛡ZTNA Topology

### Lab: POP as Aggregation Node

```
FG1 ─── POP1 (Linux) ─── WARP ─── Cloudflare ─── Internet
         ↑
    Aggregation Node
    (EVE-NG constraint)
```

- EVE-NG cannot run WARP client directly on FortiGate
- Linux POP inserted between FG and Cloudflare
- POP acts as ZTNA aggregation point for the site

### Production: Direct ZTNA from LAN

```
FG (SD-WAN/Firewall)
         │
         └─── LAN
               │
               ├─── Server (WARP/ZTNA)
               ├─── Server (WARP/ZTNA)
               └─── Endpoint (WARP/ZTNA)
                         │
                         ▼
                    Cloudflare
```

- WARP client installed directly on servers/endpoints
- Each device connects to Cloudflare individually
- FortiGate provides SD-WAN path selection, not ZTNA aggregation

**【日本語サマリ】**

ラボではEVE-NGの制約でFortiGateにWARPを直接実行できないため、POPをAggregation Nodeとして設置しZTNA接続。本番ではFG配下のLANに接続されたサーバ/Endpointに直接WARPをインストールし、個別にCloudflareへZTNA接続。

---

## ✅What Transfers to Production

Despite simplifications, the following skills and concepts transfer directly:

| Skill | Lab Validation | Production Application |
|-------|----------------|------------------------|
| SD-WAN Health Check | ✓ Configured and tested | Same configuration |
| SLA-based Failover | ✓ Demonstrated | Same behavior |
| SASE Policy (SWG/ZTNA) | ✓ Configured and tested | Same configuration |
| Troubleshooting methodology | ✓ Layer-by-layer approach | Same approach |
| FortiGate CLI | ✓ diagnose commands | Same commands |
| Cloudflare Dashboard | ✓ Gateway Logs analysis | Same interface |

**【日本語サマリ】**

ラボで検証したSD-WAN Health Check、SLAフェイルオーバー、SASEポリシー、トラブルシューティング手法、CLI/Dashboard操作は本番環境でもそのまま適用可能。

---

## 🔗Related Components

- [SASE-ZeroTrust](https://github.com/mikio-abe/SASE-ZeroTrust) - Security policy
- [SD-WAN](https://github.com/mikio-abe/SD-WAN) - Path selection
- [Enterprise-SP](https://github.com/mikio-abe/Enterprise-SP) - MPLS underlay
- [Troubleshooting](https://github.com/mikio-abe/Troubleshooting) - Verification methods

---

*Part of SASE × SD-WAN Verification Lab*
