<div align="center">

![nahj-net-automation banner](doc/assets/banner.jpg)

![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=flat&logo=ansible&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![VMware](https://img.shields.io/badge/VMware-607078?style=flat&logo=vmware&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white)

</div>

---

## What This Project Does

We automate the configuration and monitoring of a real multi-host network using Ansible — all from a single command inside a Docker container.

1. **Cisco CSR1000v Router** — Ansible configures 3 real GigabitEthernet interfaces, creates a user account, sets a login banner, adds interface descriptions, configures a static route, and retrieves live device info
2. **Linux Monitoring Host** — Ansible collects hostname, date/time, CPU, memory, disk, logged-in users, and top processes, then generates a Markdown report via Jinja2
3. **LAN End Devices** — Ansible SSHes into 4 end devices split across two LAN segments (pc1, laptop1 on LAN1 · server1, pc2 on LAN2) and generates a per-device report for each

Credentials are encrypted with **Ansible Vault** — no plain-text passwords anywhere in this repo. A **GitHub Actions CI pipeline** runs `ansible-lint` on every Pull Request before it merges.

---

## Architecture

<div align="center">

![Project Architecture](doc/assets/archet.png)

</div>

## Network Topology

<div align="center">

![Network Topology](doc/assets/network%20topology.png)

</div>

---

## Network Design

| Interface | Subnet | Role |
|-----------|--------|------|
| GigabitEthernet1 | 192.168.229.0/24 | Management — Ansible connects here |
| GigabitEthernet2 | 192.168.10.0/24 | LAN1 — pc1, laptop1 |
| GigabitEthernet3 | 192.168.20.0/24 | LAN2 — server1, pc2 |

---

## Team

| Member | Role | Branch |
|--------|------|--------|
| Omar *(Leader)* | Docker, inventory, Ansible Vault, CI pipeline, site.yml | `feature/setup` |
| Moqbel | Router config — IP addressing, user account, MOTD banner | `feature/net-identity` |
| Ali | Router config — interface descriptions, static route, device info | `feature/net-routing` |
| Knan | Linux sysinfo — hostname, date/time, CPU, memory, disk | `feature/linux-facts` |
| Ahmed | Linux sysinfo — logged-in users, top processes, Jinja2 report | `feature/linux-activity` |

---

## What Gets Automated

### Router — Cisco CSR1000v (VMware Workstation)

| Task | Playbook |
|------|----------|
| Configure IP on GigabitEthernet2 (LAN1) and GigabitEthernet3 (LAN2) | `net_identity.yml` |
| Create local user account | `net_identity.yml` |
| Set login banner (MOTD) | `net_identity.yml` |
| Set interface descriptions on Gi2 and Gi3 | `net_routing.yml` |
| Add static route (LAN2 via GigabitEthernet3) | `net_routing.yml` |
| Retrieve show run / show version / show ip int brief | `net_routing.yml` |

### Linux Monitoring Host

| Info Collected | Playbook |
|----------------|----------|
| Hostname | `linux_facts.yml` |
| Date and time | `linux_facts.yml` |
| CPU architecture and core count | `linux_facts.yml` |
| Memory usage | `linux_facts.yml` |
| Disk usage per mount | `linux_facts.yml` |
| Logged-in users | `linux_activity.yml` |
| Top 5 processes by CPU | `linux_activity.yml` |

> Output saved as `playbooks/report.md` via Jinja2 (`templates/report.j2`)

### LAN End Devices (pc1 · laptop1 · server1 · pc2)

| Info Collected | Playbook |
|----------------|----------|
| Device role and LAN segment | `lan_devices.yml` |
| CPU architecture and core count | `lan_devices.yml` |
| Memory usage | `lan_devices.yml` |
| Top 3 processes by CPU | `lan_devices.yml` |

> One report per device: `playbooks/report_pc1.md`, `report_laptop1.md`, `report_server1.md`, `report_pc2.md`

---

## Prerequisites

- [Git](https://git-scm.com/downloads)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) — enable WSL2 on Windows
- [VMware Workstation](https://www.vmware.com/products/workstation-pro.html) with Cisco CSR1000v loaded and VMnet2/VMnet3 configured

> Ansible runs inside the Docker container — no local install needed.

---

## Project Structure

```
nahj-net-automation/
├── .github/
│   └── workflows/
│       └── ci.yml                   # ansible-lint on every PR
├── doc/
│   ├── assets/
│   │   ├── banner.jpg
│   │   ├── archet.png               # architecture diagram
│   │   └── network topology.png     # network topology diagram
│   ├── sam/
│   │   └── CSR1000v_Multi_Interface_Writeup.md
│   ├── overview.md
│   └── phases.md
├── docker/
│   └── Dockerfile                   # Ansible control node image
├── inventory/
│   ├── hosts.yml                    # all target hosts
│   └── group_vars/
│       └── all/
│           └── vault.yml            # auto-loaded encrypted vars
├── vars/
│   └── vault.yml                    # Ansible Vault encrypted credentials
├── templates/
│   ├── report.j2                    # Jinja2 template — linux_host report
│   └── device_report.j2             # Jinja2 template — per LAN device report
├── playbooks/
│   ├── site.yml                     # master playbook — runs everything
│   ├── net_identity.yml             # router: IP, user, banner
│   ├── net_routing.yml              # router: descriptions, static route, device info
│   ├── linux_facts.yml              # linux_host: hostname, CPU, memory, disk
│   ├── linux_activity.yml           # linux_host: users, processes, report
│   └── lan_devices.yml              # LAN devices: info collection + per-device reports
├── reflections/                     # personal reflection PDFs
└── README.md
```

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/Mhdomer/nahj-net-automation.git
cd nahj-net-automation
```

### 2. Start the Linux monitoring host

```bash
docker run -d \
  --name nahj-linux \
  -e PUID=1000 -e PGID=1000 \
  -e PASSWORD_ACCESS=true \
  -e USER_NAME=nahj \
  -e USER_PASSWORD=group-nahj \
  -p 2222:2222 \
  lscr.io/linuxserver/openssh-server

docker exec nahj-linux apk add python3
```

### 3. Start the LAN end device containers

```bash
for name_port in "pc1:2223" "laptop1:2224" "server1:2225" "pc2:2226"; do
  name="${name_port%%:*}"
  port="${name_port##*:}"
  docker run -d --name "$name" \
    -e PUID=1000 -e PGID=1000 \
    -e PASSWORD_ACCESS=true \
    -e USER_NAME=nahj \
    -e USER_PASSWORD=group-nahj \
    -p "$port":2222 \
    lscr.io/linuxserver/openssh-server
  docker exec "$name" apk add python3
done
```

### 4. Start the Cisco CSR1000v

Boot the CSR1000v in VMware Workstation. Confirm it has 3 NICs assigned — VMnet1 (management), VMnet2 (LAN1), VMnet3 (LAN2). Update `ansible_host` in `inventory/hosts.yml` if your router IP differs from `192.168.229.129`.

### 5. Enter the Ansible control node

```bash
docker compose run --rm ansible bash
```

### 6. Set up the Ansible Vault password

```bash
echo "nahj" > /tmp/.vault_pass
```

### 7. Run the full automation

```bash
ansible-playbook -i inventory/hosts.yml playbooks/site.yml --vault-password-file /tmp/.vault_pass
```

Or run a specific playbook:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/net_identity.yml --vault-password-file /tmp/.vault_pass
ansible-playbook -i inventory/hosts.yml playbooks/lan_devices.yml  --vault-password-file /tmp/.vault_pass
```

Reports are saved to `playbooks/report.md` and `playbooks/report_<device>.md`.

---

## Inventory Structure

```
all
├── routers        → cisco_router (CSR-Nahj · 192.168.229.129)
├── linux          → linux_host   (Docker · port 2222)
└── lan_devices    → pc1 (2223) · laptop1 (2224) · server1 (2225) · pc2 (2226)
```

---

## CI Pipeline

Every Pull Request triggers a GitHub Actions workflow that runs `ansible-lint` on all playbooks. All playbooks must pass lint before merging to `main`.

---

## Course Info

**Course:** SECR3253 Network Programming
**University:** Universiti Teknologi Malaysia (UTM)
**Semester:** 2025/2026-2
