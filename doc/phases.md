---
tags:
  - nahj
  - phases
  - progress
---

# Project Phases — Group Nahj

> [!info] Related Files
> [[overview]] — full project scope, folder structure, and tool decisions

**Repository:** https://github.com/Mhdomer/nahj-net-automation  
**Last Updated:** 2026-06-21  
**Current Phase:** Phase 1 — Setup

---

## Phase Overview

```
Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5
 Setup     Build    Enhance   Integrate  Submit
  🔄         ⏳        ⏳        ⏳        ⏳
```

---

## Phase 1 — Setup & Onboarding
**Target:** June 21–25  
**Status:** 🔄 In Progress

This phase is about getting everyone aligned and the repo ready before any code is written.

### Tasks
- [ ] Create GitHub repo (`nahj-net-automation`) — *Leader*
- [ ] Add all 5 members as collaborators — *Leader*
- [ ] Push initial skeleton folder structure — *Leader*
- [ ] Each member installs: Git, Docker Desktop, GNS3, VS Code
- [ ] Each member accepts GitHub invite and clones the repo
- [ ] Each member creates their own branch
- [ ] GNS3 lab topology decided and shared with group — *Leader*
- [ ] Cisco IOS image confirmed working in GNS3 — *Leader*

> [!warning] Blocker Risk
> GNS3 + Cisco IOS image setup can be time-consuming. The leader should confirm the lab is working before June 25 so members aren't blocked waiting for it.

---

## Phase 2 — Core Development
**Target:** June 26 – July 1  
**Status:** ⏳ Not Started

Each member works independently on their own branch and file. No waiting on others.

### Member 2 — `feature/net-identity`
- [ ] Write `playbooks/net_identity.yml`
- [ ] Task: Configure IP address on router interface
- [ ] Task: Create local user account on router
- [ ] Task: Set login banner (MOTD)
- [ ] Test against GNS3 router, confirm config applied
- [ ] Commit at least 3 times across different days
- [ ] Open Pull Request to `main`

### Member 3 — `feature/net-routing`
- [ ] Write `playbooks/net_routing.yml`
- [ ] Task: Add interface description
- [ ] Task: Add static route
- [ ] Task: Retrieve and display device information (`show running-config`, `show version`)
- [ ] Test against GNS3 router, confirm output
- [ ] Commit at least 3 times across different days
- [ ] Open Pull Request to `main`

### Member 4 — `feature/linux-facts`
- [ ] Write `playbooks/linux_facts.yml`
- [ ] Task: Collect hostname
- [ ] Task: Collect current date and time
- [ ] Task: Collect CPU information
- [ ] Task: Collect memory usage
- [ ] Task: Collect disk usage
- [ ] Test against Linux host, confirm data collected
- [ ] Commit at least 3 times across different days
- [ ] Open Pull Request to `main`

### Member 5 — `feature/linux-activity`
- [ ] Write `playbooks/linux_activity.yml`
- [ ] Task: Collect logged-in users
- [ ] Task: Collect top 5 processes by CPU usage
- [ ] Write `templates/report.j2` — Jinja2 template for the final report
- [ ] Integrate report generation into playbook (outputs `report.md`)
- [ ] Test report output, confirm it looks clean
- [ ] Commit at least 3 times across different days
- [ ] Open Pull Request to `main`

### Member 1 (Leader) — `feature/setup`
- [ ] Write `docker/Dockerfile` — Ansible control node with `cisco.ios` collection
- [ ] Write `docker-compose.yml`
- [ ] Write `inventory/hosts.yml` — targets GNS3 router + Linux host (no plain-text passwords)
- [ ] Write `vars/vault.yml` — Ansible Vault encrypted credentials
- [ ] Write `playbooks/site.yml` — master playbook that imports all others
- [ ] Write `.github/workflows/ci.yml` — ansible-lint on every PR
- [ ] Open Pull Request to `main`

---

## Phase 3 — Enhancements
**Target:** June 28 – July 1 (overlaps Phase 2)  
**Status:** ⏳ Not Started

These run in parallel with Phase 2, owned by the leader.

### Tasks
- [ ] **Ansible Vault** — encrypt router password and user credentials in `vars/vault.yml`, update `hosts.yml` to reference vault variables instead of plain text
- [ ] **Jinja2 Report** — finalize `templates/report.j2`, verify `report.md` output is well-formatted
- [ ] **GitHub Actions CI** — confirm `ci.yml` runs `ansible-lint` successfully on a test PR

> [!tip] Why These Matter
> Most groups will have passwords in plain text and raw terminal output. These three additions are what separates our project from everyone else without adding scope.

---

## Phase 4 — Integration & Testing
**Target:** July 2–3  
**Status:** ⏳ Not Started

All PRs merged. We run the full project end-to-end and fix any issues.

### Tasks
- [ ] Merge all PRs into `main` — *Leader reviews each*
- [ ] Run `playbooks/site.yml` end-to-end against GNS3 + Linux host
- [ ] Confirm all 6 network config tasks applied correctly on router
- [ ] Confirm `report.md` generated correctly with all 7 Linux sysinfo items
- [ ] Confirm GitHub Actions CI passes on `main`
- [ ] Finalize `README.md` — setup instructions, how to run, how to decrypt Vault
- [ ] Take screenshots/recordings for evidence

---

## Phase 5 — Documentation & Submission
**Target:** July 4–6  
**Status:** ⏳ Not Started

### Tasks
- [ ] Each member writes their 2-page personal reflection (`reflections/<name>_reflection.docx`)
- [ ] Final check: all 5 members have meaningful commits across multiple days
- [ ] Final check: README is complete and accurate
- [ ] Final check: no plain-text passwords anywhere in the repo
- [ ] Submit GitHub repo URL — **before 9 AM, July 6**

> [!warning] Deadline
> **July 6, 2026 at 9:00 AM** — do not leave reflections to the last night.

---

## Progress Summary

| Phase | Status | Owner | Due |
|-------|--------|-------|-----|
| Phase 1 — Setup | 🔄 In Progress | All members | June 25 |
| Phase 2 — Core Dev | ⏳ Not Started | Each member owns their file | July 1 |
| Phase 3 — Enhancements | ⏳ Not Started | Leader | July 1 |
| Phase 4 — Integration | ⏳ Not Started | Leader + all test | July 3 |
| Phase 5 — Submission | ⏳ Not Started | All members | July 6 |

---

## Status Key

| Icon | Meaning |
|------|---------|
| ✅ | Done |
| 🔄 | In Progress |
| ⏳ | Not Started |
| 🚫 | Blocked |