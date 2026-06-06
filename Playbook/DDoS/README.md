# DDoS Incident Response Playbook

A structured, phase-based playbook for detecting, containing, and recovering from DDoS attacks.

---

## What's inside

The playbook covers the full incident lifecycle across 7 phases:

1. **Detection** — baselines, indicators, false positive validation
2. **Scoping** — attack type, scale, business impact
3. **Containment & Mitigation** — CDN, WAF, firewall, ISP, FlowSpec, RTBH
4. **Business Protection** — service priority, impact assessment, communications
5. **Recovery** — validation checklist, safe restoration
6. **Evidence Collection** — logs, timelines, handoff summary
7. **Post-Incident Review** — root cause, lessons learned

Also includes severity classification, authority matrix, escalation criteria, and hardening recommendations.

---

## Attack types covered

| Category | Examples |
|---|---|
| Volumetric | UDP Flood, DNS/NTP/Memcached Amplification |
| Protocol | SYN Flood, ACK Flood, TCP State Exhaustion |
| Application Layer | HTTP Flood, Slowloris, API Abuse, Bot Attacks |
| Multi-Vector | Simultaneous layered attacks |

---

## How to use this

This is an operational reference, not a checklist to follow blindly.

- Read the **Safety Rules** section before taking any mitigation action
- Follow the **Authority Matrix** — know who needs to approve what
- Use the **Decision Matrix** in Phase 2 to pick the right mitigation path
- Document every action with timestamp, analyst, reason, and outcome

---

## License

For internal use. Adapt to your organization's infrastructure and approval workflows before deploying operationally.
