<div align="center">

![nahj-net-automation banner](doc/assets/banner.jpg)

![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=flat&logo=ansible&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![GNS3](https://img.shields.io/badge/GNS3-Network%20Sim-blue?style=flat)

</div>

---

## What This Project Does

We are automating two things:

1. **Network Device Configuration** — Ansible pushes configuration tasks to a Cisco IOS router running in GNS3
2. **Linux System Info Collection** — Ansible gathers system details from a Linux host and generates a formatted Markdown report

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

## Team

| Member | Role | Branch |
|--------|------|--------|
| Omar *(Leader)* | Docker, repo setup, Ansible Vault, CI pipeline | `feature/setup` |
| Moqbel | Network config — IP, user account, banner | `feature/net-identity` |
| Ali | Network config — interface description, static route, device info | `feature/net-routing` |
| Knan | Linux sysinfo — hostname, date/time, CPU, memory, disk | `feature/linux-facts` |
| Ahmed | Linux sysinfo — logged-in users, top 5 processes, Jinja2 report | `feature/linux-activity` |

---

## What Gets Automated

### Part 1 — Cisco IOS Router (GNS3)

| Task | Playbook |
|------|----------|
| Configure IP address on interface | `playbooks/net_identity.yml` |
| Create local user account | `playbooks/net_identity.yml` |
| Set login banner (MOTD) | `playbooks/net_identity.yml` |
| Add interface description | `playbooks/net_routing.yml` |
| Add static route | `playbooks/net_routing.yml` |
| Retrieve device information | `playbooks/net_routing.yml` |

### Part 2 — Linux Host

| Info Collected | Playbook |
|---------------|----------|
| Hostname | `playbooks/linux_facts.yml` |
| Current date and time | `playbooks/linux_facts.yml` |
| CPU information | `playbooks/linux_facts.yml` |
| Memory usage | `playbooks/linux_facts.yml` |
| Disk usage | `playbooks/linux_facts.yml` |
| Logged-in users | `playbooks/linux_activity.yml` |
| Top 5 processes by CPU | `playbooks/linux_activity.yml` |

> Output is saved as a formatted `report.md` via a Jinja2 template.

---

## Prerequisites

- [Git](https://git-scm.com/downloads)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) — enable WSL2 on Windows
- [VMware Workstation](https://www.vmware.com/products/workstation-pro.html) with a Cisco CSR1000v image loaded

> Ansible runs inside the Docker container — no local install needed.

---

## Project Structure

```
nahj-net-automation/
├── .github/
│   └── workflows/
│       └── ci.yml               # ansible-lint on every PR
├── doc/
│   ├── assets/
│   │   ├── banner.jpg
│   │   └── archet.png
│   ├── overview.md
│   └── phases.md
├── docker/
│   └── Dockerfile               # Ansible control node
├── inventory/
│   └── hosts.yml                # target hosts (no plain-text passwords)
├── vars/
│   └── vault.yml                # Ansible Vault encrypted credentials
├── templates/
│   └── report.j2                # Jinja2 template for sysinfo report
├── playbooks/
│   ├── site.yml                 # master playbook — runs everything
│   ├── net_identity.yml         # IP, user, banner
│   ├── net_routing.yml          # interface desc, static route, device info
│   ├── linux_facts.yml          # hostname, date/time, CPU, memory, disk
│   └── linux_activity.yml       # logged-in users, top 5 processes + report
├── reflections/                 # personal reflection reports
└── README.md
```

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/Mhdomer/nahj-net-automation.git
cd nahj-net-automation
```

### 2. Start the Linux target host

```bash
docker run -d \
  --name nahj-linux \
  -e PUID=1000 -e PGID=1000 \
  -e PASSWORD_ACCESS=true \
  -e USER_NAME=nahj \
  -e USER_PASSWORD=group-nahj \
  -p 2222:2222 \
  lscr.io/linuxserver/openssh-server
```

### 3. Start the Cisco router

Boot your Cisco CSR1000v in VMware Workstation. Once it's up, confirm SSH is reachable on port 22. Update `ansible_host` in `inventory/hosts.yml` to match your router's IP if it differs.

### 4. Enter the Ansible control node

```bash
docker compose run --rm ansible bash
```

### 5. Set up the Ansible Vault password

```bash
echo "nahj" > /tmp/.vault_pass
```

### 6. Run the full automation

```bash
ansible-playbook -i inventory/hosts.yml playbooks/site.yml --vault-password-file /tmp/.vault_pass
```

Or run a specific playbook:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/net_identity.yml --vault-password-file /tmp/.vault_pass
ansible-playbook -i inventory/hosts.yml playbooks/linux_facts.yml --vault-password-file /tmp/.vault_pass
```

The report is saved to `playbooks/report.md` after the Linux playbooks run.

---

## Git Workflow

```
main  (protected — no direct pushes)
 ├── feature/setup             ← Omar
 ├── feature/net-identity      ← Moqbel
 ├── feature/net-routing       ← Ali
 ├── feature/linux-facts       ← Knan
 └── feature/linux-activity    ← Ahmed
```

1. Work on your own branch
2. Commit regularly — at least 3 commits across different days
3. Open a Pull Request to `main`

---

## CI Pipeline

Every Pull Request triggers a GitHub Actions workflow that runs `ansible-lint` on all playbooks. Fix any lint warnings before requesting a review.

---

## Course Info

**Course:** SECR3253 Network Programming  
**University:** Universiti Teknologi Malaysia (UTM)  
**Semester:** 2025/2026-2