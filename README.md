# Homelab

Personal infrastructure running on repurposed hardware. The homelab 
serves as both a development platform and a production environment for 
projects I actually use.

<img width="1440" height="1058" alt="image" src="https://github.com/user-attachments/assets/109e0a61-2f94-4cb8-8991-9e2006023a76" />


---

## Hardware

| Device | Role |
|---|---|
| Nokia fiber modem | ISP-provided, fiber → Ethernet |
| Consumer router | DHCP, firewall, port forwarding |
| Proxmox server | Repurposed Linux server, now a hypervisor |
| Workstation | AMD Ryzen 7 5700X · 40GB DDR4 · Win10 — daily driver |

---

## Proxmox

The server runs Proxmox VE, hosting all VMs and containers on local 
hardware. Each workload is isolated in its own VM.

| VM | Purpose |
|---|---|
| GitHub Actions runner | Self-hosted CI/CD runner for [ThreatAggregator](https://github.com/AyyyStew/ThreatAggregator) — receives push events, runs tests, builds and pushes Docker images, deploys via SSH |
| Docker VM | Hosts the Threat Aggregator container in production |
| Celery worker | Distributed compute node for the [Candidate Chess](https://github.com/AyyyStew/candidate-chess) position generation pipeline — runs Stockfish evaluation jobs dispatched via Redis |
| Wazuh SIEM | Centralized security monitoring — agents on all VMs ship logs for threat detection and alerting |
| Game servers | Minecraft, Terraria, Valheim, Palworld — port-forwarded for remote access |
| Ubuntu desktop | Linux environment for sysadmin practice and tooling experiments |

---

## CI/CD integration

The GitHub Actions runner VM is the operational core of this lab. On 
every push to ThreatAggregator's main branch:

```
GitHub push event
  → self-hosted runner picks up job
  → pytest
  → docker build
  → docker push → Docker Hub
  → SSH deploy → Docker VM restarts container
```

The runner and the deployment target are both VMs on the same Proxmox 
host, managed independently.

---

## Security monitoring

Wazuh agents run on all VMs and ship logs to the Wazuh SIEM VM. 
Provides centralized visibility across the homelab — failed auth 
attempts, file integrity, process monitoring.

The Docker VM also runs the Threat Aggregator, which feeds a PowerShell 
firewall automation script (`automateAddingToWindowsFirewall.ps1`) that 
blocks known malicious IPs on the workstation via Windows Defender 
Firewall rules, updated on a daily Task Scheduler job.

---

## Projects running here

- [ThreatAggregator](https://github.com/AyyyStew/ThreatAggregator) — 
  IoC feed aggregator API, containerized, continuously deployed
- [Candidate Chess](https://github.com/AyyyStew/candidate-chess) — 
  Celery worker participates in offline position generation pipeline
