---
tags:
  - nahj
  - overview
  - network-automation
---

# Project Overview — Group Nahj

**Course:** SECR3253 Network Programming  
**Assignment:** Group Assignment (5%)  
**Deadline:** 6 July 2026, 9:00 AM  
**Group Name:** Nahj  
**Group Size:** 5 members  
**Repository:** https://github.com/Mhdomer/nahj-net-automation

> [!info] Related Files
> [[phases]] — where we are right now and what's next

---

## What We Are Building

We are building a **network automation project** that does two things:

1. **Automate the configuration of a virtual network router** — using Ansible to push configurations to a Cisco IOS router running inside GNS3.
2. **Automatically collect and display Linux system information** — using Ansible to gather details from a Linux host.

Everything is containerized using **Docker**, so every team member runs the exact same environment regardless of their machine.

---

## Why These Tools

| Tool | Why We Use It |
|------|---------------|
| **GNS3** | Simulates a real Cisco IOS router on our local machines — no physical hardware needed |
| **Ansible** | Automates all configuration tasks and info gathering — we write YAML, it does the work |
| **Docker** | Packages our Ansible control node so the environment is identical for everyone |
| **Ansible Vault** | Encrypts our credentials so no passwords are ever stored in plain text |
| **GitHub Actions** | Runs automated checks on every Pull Request to keep our code clean |
| **GitHub** | Hosts our repository, tracks contributions, and manages our workflow through branches and pull requests |

---

## What Our Automation Does

### Part 1 — Network Device Configuration

We configure a Cisco IOS router running in GNS3 with the following:

- **IP Address** — assign an IP to a router interface
- **User Account** — create a local user on the device
- **Banner Message** — set a login banner (MOTD)
- **Interface Description** — add a description label to an interface
- **Static Route** — add a static routing entry
- **Retrieve Device Information** — pull and display the running configuration and device details

### Part 2 — Linux System Information Collection

We automatically collect and display the following from a Linux host:

- Hostname
- Current date and time
- CPU information
- Memory usage
- Disk usage
- Logged-in users
- Top 5 running processes by CPU usage

The output is generated as a **formatted Markdown report** using a Jinja2 template — not just raw terminal output.

---

## What Makes Our Project Stand Out

We are adding three layers on top of the base requirements that most groups won't bother with:

> [!tip] Ansible Vault — Credential Security
> We do **not** store passwords or credentials in plain text anywhere in the repository. All sensitive values (router password, user account credentials) are encrypted using Ansible Vault. This is standard DevSecOps practice and immediately signals that we understand secure automation.

> [!tip] Jinja2 Report Generation
> Instead of dumping raw terminal output, our Linux sysinfo playbook generates a clean, formatted **`report.md`** file using a Jinja2 template. The report presents all collected system information in a structured, readable format — it looks like a finished tool, not a script.

> [!tip] GitHub Actions CI Pipeline
> We have a CI workflow (`.github/workflows/ci.yml`) that automatically runs **`ansible-lint`** on every Pull Request. This means every contribution is checked for quality before it merges into `main`. Our repo reflects an actively maintained, professional project.

---

## How We Work Together

We follow a **branch-based Git workflow**:

- The `main` branch is our stable branch — no one pushes directly to it
- Each member works on their own branch (e.g. `feature/net-identity`, `feature/linux-facts`)
- We open **Pull Requests** to merge into `main` — the group leader reviews and approves
- We commit regularly across multiple days — not one big commit at the end

> [!warning] Important
> The Git commit history is worth **30% of our grade**. Commits must be spread across real working days, not dumped in one night.

---

## Who Owns What

| Member | Responsibility | Branch | Files |
|--------|---------------|--------|-------|
| Member 1 (Leader) | Docker setup, repo structure, Vault setup, CI pipeline, README | `feature/setup` | `Dockerfile`, `docker-compose.yml`, `inventory/hosts.yml`, `playbooks/site.yml`, `.github/workflows/ci.yml` |
| Member 2 | Network config — IP address, user account, banner | `feature/net-identity` | `playbooks/net_identity.yml` |
| Member 3 | Network config — interface description, static route, device info | `feature/net-routing` | `playbooks/net_routing.yml` |
| Member 4 | Linux sysinfo — hostname, date/time, CPU, memory, disk | `feature/linux-facts` | `playbooks/linux_facts.yml` |
| Member 5 | Linux sysinfo — logged-in users, top 5 processes + Jinja2 report | `feature/linux-activity` | `playbooks/linux_activity.yml`, `templates/report.j2` |

---

## Project Folder Structure

```
nahj-net-automation/
├── .github/
│   └── workflows/
│       └── ci.yml               ← runs ansible-lint on every PR
├── doc/
│   ├── overview.md              ← this file
│   └── phases.md                ← project phases and current progress
├── docker/
│   └── Dockerfile               ← Ansible control node container
├── inventory/
│   └── hosts.yml                ← GNS3 router + Linux host targets (no plain-text passwords)
├── vars/
│   └── vault.yml                ← Ansible Vault encrypted credentials
├── templates/
│   └── report.j2                ← Jinja2 template for sysinfo report
├── playbooks/
│   ├── site.yml                 ← master playbook (runs everything)
│   ├── net_identity.yml         ← IP, user, banner
│   ├── net_routing.yml          ← interface desc, static route, device info
│   ├── linux_facts.yml          ← hostname, date/time, CPU, memory, disk
│   └── linux_activity.yml       ← logged-in users, top 5 processes + report generation
├── reflections/
│   └── <name>_reflection.docx  ← each member's 2-page reflection
└── README.md                    ← how to set up and run everything
```

---

## What Each Member Needs to Install

Before starting, every member must have the following installed:

1. **Git** — for version control and committing work
2. **Docker Desktop** — runs our Ansible control node container (enable WSL2 on Windows)
3. **GNS3** — simulates the Cisco IOS router we configure
4. **VS Code** (recommended) with extensions:
   - `ms-vscode-remote.remote-containers`
   - `redhat.ansible`
5. **A GitHub account** — send your username to the group leader to be added as a collaborator

> [!note]
> Ansible does **not** need to be installed locally. It runs inside the Docker container.

---

## Timeline

| Date | Milestone |
|------|-----------|
| June 21–25 | Repo created, all members added, tools installed, skeleton files committed |
| June 26 – July 1 | Each member builds and tests their playbook on their own branch |
| July 2–3 | PRs merged, full integration test, README finalized |
| July 4–5 | Personal reflections written, final review |
| **July 6, 9 AM** | **Submit GitHub repo URL** |