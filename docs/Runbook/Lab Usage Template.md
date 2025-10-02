# Lab Usage Template

> For all VMs in the **9000+ range** (Sandbox / Lab)

## Basic Info
- **VMID:** 900X
- **Name:** labX
- **FQDN:** labX.home.lab
- **VLAN:** 40 (10.20.40.0/24)
- **IP:** 10.20.40.X
- **Memory:** 4 GB (expand as needed)
- **CPU:** 2 vCPU (expand as needed)
- **OS:** Debian / Ubuntu / Other

---

## Intended Purpose
- [ ] Linux learning / package testing  
- [ ] Docker / Compose practice  
- [ ] Security tools / Pentesting  
- [ ] CTF / vulnerable apps  
- [ ] Networking experiments  
- [ ] Other: __________________________  

---

## Setup Checklist
- [ ] Installed base OS (Debian preferred)  
- [ ] Set static IP on VLAN 40 (10.20.40.x)  
- [ ] Added to DNS (`labX.home.lab`)  
- [ ] Installed guest tools (QEMU agent)  
- [ ] Initial snapshot created (“clean base”)  

---

## Experiment Notes
- **Experiment Name:** __________________________  
- **Tools Installed:** __________________________  
- **Configs Changed:** __________________________  
- **Known Breaks:** __________________________  

---

## Cleanup / Reset
- [ ] Revert to snapshot (if broken beyond repair)  
- [ ] Export config if useful for future  
- [ ] Archive notes into Obsidian/Git  
- [ ] Delete VM if not needed  

---

## Snapshots (Recommended)
- **Base clean install** → after OS + updates  
- **Configured baseline** → after Docker/Kali/tools installed  
- **Pre-experiment** → before trying new tools/exploits  

---

## Notes
- VLAN 40 ensures complete isolation from LAN.  
- Use `iptables`/firewall rules on OPNsense if extra segmentation is needed.  
- Do not bridge VLAN 40 into VLAN 10 unless explicitly required for a test.  
