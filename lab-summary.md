# 🧠 Sadeed Home Lab – Final Summary v2 (October 2025)

## ⚙️ Server & Role Inventory

### 🟢 HP DL380 Gen10 (2U, SFF)
**Role(s):**
- Main Compute / Production Hypervisor  
- High-IOPS VM tier (critical services)

**Specs**
- CPU: 2 × Xeon Gold 6140 (36 threads each)  
- RAM: 256 GB DDR4 RDIMM  
- Storage:  
  - 5 × Samsung PM863A / similar 480 GB enterprise SATA SSDs  
  - 2 × WD Red 1 TB NAS SSDs  
  - ≈ 4.5–5 TB raw total  
- Connectivity: 10 GbE core  
- Status: Always On (24/7)

**Purpose**
Production workloads — virtualization, AI/ML training, AD/DNS, VPN, and file services.

---

### 🟢 Dell PowerEdge R730 (2U, LFF)
**Role(s):**
- Live Storage Node (TrueNAS/Unraid)  
- Central capacity pool and iSCSI/NFS exporter

**Specs**
- CPU: 2 × Xeon E5-2620 v2  
- RAM: 128 GB DDR4 ECC  
- Storage:  
  - 8 × 6 TB SAS 3.5″ (≈ 48 TB raw)  
  - RAIDZ2 → ≈ 36–40 TB usable  
  - Dual M.2 BOSS-S1 boot mirror  
- Connectivity: 10 GbE  
- Status: Always On

**Purpose**
Serves all VMs and backup storage.  
Acts as the “data lake” and main iSCSI/NFS target.

💸 **Cost:** $900 CAD  
💰 **Market Value:** ~$1,800–2,000 CAD  
✅ **Savings:** ~50%

---

### 🟢 HP DL360 Gen10 (1U, SFF) — “Option D Node”
**Role(s):**
- DR / Failover Node  
- Test & Dev Environment

**Specs**
- CPU: Xeon Silver  
- RAM: 32 GB DDR4  
- Storage: 8 × 2 TB SAS3 10K RPM drives (Seagate DKSSHJ/J1R8SS)  
  - 16 TB raw total  
- Status: Warm Standby (on demand)

**Purpose**
Runs replicated or dev workloads; can assume production role in downtime.

💸 **Cost:** $280 (drives) + $400 (server) ≈ $680 CAD  
💰 **Market Value:** ~$1,000–1,200 CAD  
✅ **Savings:** ~45%

**RAID Decision (Pending):**
| Layout | Pros | Cons | Best Use |
|---------|------|------|----------|
| **RAID10** | High IOPS, fast rebuild, ideal for VM failover | Less usable (~8 TB) | Recommended for DR performance |
| **RAIDZ2** | More capacity (~11–12 TB) | Slower rebuild, less IOPS | Snapshot storage / archive |

🧭 **Recommendation:** RAID10 — gives real-time failover capability for critical workloads.

---

### 🟡 HP DL380 Gen9 (2U, LFF) — New Purchase
**Role(s):**
- Cold Backup / Offline Mirror  
- Spare-parts donor for Gen9 fleet  
- Periodic backup/sync system

**Specs**
- CPU: 2 × Xeon E5-2620 v3  
- RAM: 128 GB DDR4  
- Storage:
  - 8 × 4 TB SATA + 4 × 3 TB SAS = 44 TB installed  
  - + 4 × 3 TB SAS loose spares → 56 TB total raw inventory  
- Rails included  
- Status: Off most of the week; powered on for syncs

**Purpose**
Offline backup and cold redundancy layer (low-power, air-gapped).

💸 **Cost:** $600 CAD  
💰 **Market Value:** ~$1,200–1,600 CAD  
✅ **Savings:** ~55–60%

---

### 🟡 HP DL360 Gen9 (1U, SFF)
**Role Options**
1. **Utility Node / Control Plane**
   - Run PBS (Proxmox Backup Server), monitoring (Zabbix/Grafana), DNS, syslog, or management VLAN router.
   - Pros: Keeps management layer isolated.  
   - Cons: +60–80 W idle power draw.

2. **Sell and Reinvest**
   - Market resale: ~$400 CAD after adding a 480 GB SSD.
   - Funds could buy:
     - Boot SSDs for other nodes
     - 10 Gb NIC for DL380 Gen9
     - Extra enterprise SSDs for DL380 Gen10 consistency

**Recommendation:**  
If minimizing idle power → sell.  
If isolating management functions → keep and repurpose.

---

## 💾 Storage Architecture Overview

| Tier | Server | Drives | RAID | Usable | Role |
|------|---------|---------|-------|--------|------|
| **SSD Tier** | DL380 Gen10 | 7× SSDs (480 GB–1 TB) | RAID10 | ~4 TB | Production VM & databases |
| **SAS3 Tier** | DL360 Gen10 | 8× 2 TB 10 K SAS | RAID10 | ~8 TB | DR / Failover workloads |
| **HDD Tier** | R730 | 8× 6 TB SAS | RAIDZ2 | ~36–40 TB | Main shared storage |
| **Cold Backup** | DL380 Gen9 | 44 TB HDDs | RAID6 | ~32 TB | Offline sync backup |

---

## 🌐 Network & Rack Infrastructure

| Device | Role | Notes | Cost (CAD) |
|---------|------|-------|------------|
| **Netgear ProSafe XS712T** | Core 10 GbE switch | 12× RJ45 10G + 2× SFP+ | 140 |
| **TP-Link JetStream TL-SG2428P** | PoE switch | 24× PoE+ (250 W) + 4 SFP | 150 |
| **TP-Link VPN Router (Multi-WAN)** | Gateway router | VPN + edge routing | 70 |
| **HP Aruba 7010** | Secondary / VLAN testing | Redundant routing layer | 70 |
| **Total Networking** |  |  | **430 CAD** |

**Rack & Power**
| Component | Notes | Cost (CAD) |
|------------|--------|------------|
| **APC AR2105BLK (25U)** | Main rack | 450 |
| **UPS (Costco model)** | Backup power | 250 |
| **1U PDUs (2x)** | Power distribution | 100 |
| **Cat6A Cabling + Patch Panel** | 10 GbE backbone | 220 |
| **Total Power/Rack** |  | **1,020 CAD** |

---

## 🔋 Power & Tiered Operation
| Mode | Nodes Active | Power Draw (approx.) | Notes |
|-------|---------------|----------------------|-------|
| **Daily Ops** | DL380 G10 + R730 | ~320 W | Always on |
| **DR / Maintenance** | + DL360 G10 | +120 W | Warm standby |
| **Full Sync / Backup** | + DL380 G9 | +250 W | Occasional only |
| **Management Only** | DL360 G9 | 60 W | Optional if retained |

---

## 💰 Cost Analysis

| Category | Your Cost (CAD) | Market Value (CAD) | Savings |
|-----------|-----------------|--------------------|----------|
| **Servers & Drives** | ~2,780 | ~6,200 | ~55% |
| **Networking Gear** | 430 | 900–1,200 | ~50% |
| **Rack & Power** | 1,020 | 1,200 | ~15% |
| **Total** | **≈ 4,230 CAD** | **≈ 9,100 CAD (used market)** | **≈ 54% savings** |
| *(Retail equivalent ≈ $11,000–12,000 CAD → ≈ 65% savings)* |

---

## 🧩 Role Tiers Summary

| Tier | Purpose | Nodes |
|------|----------|-------|
| **Tier 1 – Production** | Core workloads (VMs, AI, AD, etc.) | HP DL380 Gen10 + Dell R730 |
| **Tier 2 – DR/Dev** | Failover & testing | HP DL360 Gen10 |
| **Tier 3 – Cold Backup** | Offline redundancy | HP DL380 Gen9 |
| **Tier 4 – Utility / PBS** | Optional mgmt plane | HP DL360 Gen9 (optional) |

---

## 🧭 Recommendations & Next Steps

- ✅ Standardize boot drives with mirrored enterprise SSDs across all nodes.  
- ✅ Add 10 Gb NIC to DL380 Gen9 (cold backup).  
- ✅ Keep DL380 Gen9 powered down except for backup syncs.  
- ✅ Use DL360 Gen10 as RAID10 failover node (Option D).  
- ⚖️ Decide whether to sell or repurpose DL360 Gen9.  

---

**Final Takeaway:**  
You’ve built a **four-tier, data-center-grade private cluster** for **≈ 4 K CAD** — worth roughly **9–12 K CAD** on the open market.  
Balanced compute, IOPS, and capacity across Gen10 performance, Gen9 resilience, and a 10 Gb backbone — all under 500 W in normal operation.

