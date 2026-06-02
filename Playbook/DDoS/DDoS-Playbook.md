# DDoS Attack Detection & Incident Response Playbook

Version: 3.0
Last Updated: June 2026

---

# Purpose

This playbook provides a structured methodology for detecting, analyzing, containing, mitigating, recovering from, and documenting Distributed Denial of Service (DDoS) attacks.

Objectives:

* Detect DDoS activity as early as possible
* Minimize business disruption
* Protect critical services
* Support coordinated incident response
* Preserve evidence for investigation
* Improve future detection and mitigation capabilities

---

# Scope

This playbook applies to:

* Internet-facing web applications
* Customer portals
* Public APIs
* VPN gateways
* DNS infrastructure
* CDN-protected services
* Edge firewalls
* Load balancers

Supported data sources:

* Firewall Logs
* WAF Logs
* CDN Logs
* Load Balancer Logs
* NetFlow / sFlow
* IDS / IPS Alerts
* SIEM Platforms
* ISP Notifications
* Infrastructure Monitoring Systems

---

# Safety Rules

Mandatory Requirements:

* Never block production traffic without authorization unless emergency mitigation criteria are met.
* Never modify routing policies without proper approval.
* Every action must be documented.
* Every approval must be recorded.
* Escalate immediately when uncertainty exists.

Required Documentation:

* Timestamp
* Analyst
* Action Taken
* Reason
* Approver
* Outcome

---

# Severity Classification

| Severity | Description                                 |
| -------- | ------------------------------------------- |
| SEV-1    | Service outage or critical business impact  |
| SEV-2    | Significant service degradation             |
| SEV-3    | Localized operational impact                |
| SEV-4    | Suspicious activity requiring investigation |

---

# Authority Matrix

| Severity | Approval Authority                |
| -------- | --------------------------------- |
| SEV-1    | Incident Commander or SOC Manager |
| SEV-2    | SOC Lead                          |
| SEV-3    | Senior Analyst                    |
| SEV-4    | Assigned Analyst                  |

Emergency mitigation actions may be initiated without prior approval when active service disruption exists, provided that documentation and notification occur immediately afterward.

---

# Escalation Criteria

Immediate escalation is required for:

* Customer-facing outages
* VPN outages
* DNS outages
* Bandwidth saturation
* Revenue-generating service disruption
* Critical infrastructure impact
* Multi-vector attacks

---

# Baseline Requirements

Detection and response activities depend on established organizational baselines.

Organizations should maintain:

* Average inbound bandwidth
* Peak inbound bandwidth
* 95th percentile bandwidth
* Requests per second (RPS)
* Connections per second (CPS)
* Unique source IP counts
* HTTP error rate baselines
* Authentication success ratios

Detection thresholds must be periodically reviewed and adjusted according to business requirements.

---

# Threat Assumptions

Attackers may utilize:

## Volumetric Attacks

* UDP Flood
* ICMP Flood
* DNS Amplification
* NTP Amplification
* SSDP Amplification
* Memcached Amplification

## Protocol Attacks

* SYN Flood
* ACK Flood
* Fragmentation Attacks
* TCP State Exhaustion

## Application Layer Attacks

* HTTP GET Flood
* HTTP POST Flood
* Slowloris
* RUDY
* API Abuse
* Bot-Based Attacks

## Multi-Vector Attacks

Simultaneous use of multiple attack techniques targeting different infrastructure layers.

---

# Incident Activation Criteria

Activate this playbook when one or more conditions are observed:

* Significant deviation from established traffic baselines
* Sustained abnormal connection growth
* Unexpected increase in unique source IPs
* Service degradation or outage
* ISP attack notification
* CDN or WAF DDoS alerts
* Infrastructure resource exhaustion

---

# Phase 1 – Detection

## Goal

Identify potential DDoS activity and determine whether further investigation is required.

## Detection Logic

Potential indicators include:

* Abnormal bandwidth utilization
* Rapid connection growth
* Elevated request rates
* Increased error responses
* Resource exhaustion
* Significant growth in source diversity
* Unusual protocol distribution

### Example Indicators

Bandwidth:

* Traffic significantly exceeds established baseline

Connection Activity:

* Connection growth exceeds historical norms

Source Diversity:

* Sudden increase in unique external IP addresses

Application Activity:

* Elevated HTTP request rates
* Increased 429, 503, or 504 responses

Infrastructure Indicators:

* CPU exhaustion
* Memory pressure
* Connection table saturation

> **Technical Note – HTTP Error Indicators**
>
> Elevated HTTP error responses may indicate application-layer attacks or severe infrastructure stress.
>
> Common examples:
>
> - **429 Too Many Requests**: Rate limiting is being triggered due to excessive request volume.
> - **503 Service Unavailable**: Backend services are unable to handle incoming demand.
> - **504 Gateway Timeout**: Upstream services are failing to respond within expected time limits.
>
> These indicators should be evaluated together with traffic volume, request patterns, and infrastructure metrics before declaring a DDoS incident.

---

# False Positive Validation

Before declaring a DDoS incident verify with:

* NOC
* Operations Team
* Application Team
* Network Engineering

Check for:

* Planned maintenance
* Product launches
* Marketing campaigns
* Seasonal demand increases
* Infrastructure changes
* Customer notifications

---

# Flash Crowd Assessment

Determine whether increased traffic is legitimate.

Indicators of legitimate traffic:

* Normal User-Agent distribution
* Expected geographic distribution
* Successful authentication behavior
* Consistent session duration
* Expected customer activity patterns

Indicators of malicious traffic:

* Excessive request repetition
* Abnormal User-Agent patterns
* Excessive error generation
* Automated interaction indicators
* Suspicious session behavior

---

# Phase 2 – Scoping

## Goal

Determine:

* Attack type
* Scale
* Target assets
* Geographic distribution
* Business impact

## Document

* Attack start time
* Affected services
* Estimated attack volume
* Targeted ports
* Protocols involved
* Unique attacker count
* Geographic distribution
* Business impact

---

# Attack Classification

## Volumetric Attack

Characteristics:

* Bandwidth saturation
* Amplification behavior
* High packet volume

Preferred Mitigations:

* CDN Protection
* ISP Scrubbing
* Traffic Diversion

## Protocol Attack

Characteristics:

* Connection exhaustion
* State table exhaustion
* TCP abuse

Preferred Mitigations:

* SYN Protection
* FlowSpec
* ACL Controls

> **Technical Note – State Table Exhaustion**
>
> Network devices such as firewalls, load balancers, and servers maintain a state table to track active TCP sessions.
>
> During SYN Flood attacks, attackers generate large numbers of incomplete TCP handshakes, consuming available state table entries.
>
> When the table becomes exhausted, legitimate users may be unable to establish new connections even though the service itself remains operational.
>
> Common indicators include:
>
> - Large numbers of connections in SYN_RECV state
> - Increased connection failures
> - Firewall or load balancer capacity alerts
> - State table utilization approaching maximum capacity
>
> Example Linux validation:
>
> ```bash
> netstat -an | grep SYN_RECV | wc -l
> ```
>
> A significant increase in half-open TCP sessions may indicate SYN Flood activity.

## Application Layer Attack

Characteristics:

* HTTP abuse
* API abuse
* Resource exhaustion

Preferred Mitigations:

* WAF Controls
* Rate Limiting
* Bot Protection
* Challenge Mechanisms

---

# Advanced Application Layer Analysis

Investigate:

* User-Agent distribution
* JA3 fingerprints
* JA4 fingerprints
* Cookie reuse patterns
* Session creation rates
* Session completion ratios
* Authentication behavior
* Bot detection alerts
* API consumption patterns

---

# Decision Matrix

Bandwidth Saturation

→ Escalate to ISP

HTTP Flood

→ Enable WAF protections

API Abuse

→ Apply API rate limiting

Small Attacker Population

→ Consider targeted blocking

Large Distributed Population

→ Prioritize upstream mitigation

Multi-Vector Attack

→ Escalate to Incident Commander

---

# Phase 3 – Containment & Mitigation

## Goal

Reduce attack impact while minimizing business disruption.

## Mitigation Priority

### 1. CDN / DDoS Protection

Examples:

* Cloudflare
* Akamai
* Imperva
* Fastly

### 2. WAF Controls

Apply:

* Rate Limiting
* Bot Protection
* Challenge Pages
* API Throttling

### 3. Firewall Controls

Apply:

* Rate Limiting
* Connection Limits
* Service-Based Restrictions

### 4. Geographic Restrictions

Use only when justified by business requirements.

### 5. Source Blocking

Use only for confirmed attacker populations.

### 6. ISP Mitigation

Request:

* Traffic Scrubbing
* FlowSpec
* Upstream Filtering
* Traffic Diversion
* RTBH

> **Technical Note – Traffic Scrubbing**
>
> Traffic scrubbing is a mitigation service provided by an ISP or DDoS protection provider.
>
> During large volumetric attacks, traffic is redirected through a scrubbing center where malicious packets are filtered before reaching the protected environment.
>
> Benefits:
>
> - Reduces bandwidth saturation
> - Preserves service availability
> - Removes malicious traffic upstream
> - Protects infrastructure from large-scale attacks
>
> Scrubbing services are typically required for attacks that exceed the mitigation capacity of local infrastructure.

---

# RTBH Guidance

Use RTBH only when:

* Service preservation is no longer possible
* Target asset can be sacrificed temporarily
* Attack traffic threatens additional services

> **Technical Note – RTBH (Remote Triggered Black Hole)**
>
> RTBH is a defensive technique used to discard traffic before it reaches the protected network.
>
> Unlike granular mitigation methods, RTBH intentionally drops all traffic destined for a targeted host or prefix.
>
> Because both legitimate and malicious traffic are discarded, RTBH should only be used when service preservation is no longer possible and infrastructure protection becomes the primary objective.

---

# FlowSpec Guidance

Use FlowSpec when attack characteristics can be filtered through granular traffic controls without impacting legitimate traffic.

> **Technical Note – BGP FlowSpec**
>
> FlowSpec is an extension to BGP that allows network operators to distribute traffic-filtering rules across routers.
>
> Unlike traditional blackholing, FlowSpec enables selective filtering based on characteristics such as:
>
> - Source IP
> - Destination IP
> - Protocol
> - Port
> - Packet size
>
> This allows malicious traffic to be filtered while preserving legitimate traffic whenever possible.

---

# Monitoring Loop

During active mitigation:

Every 5–10 minutes:

* Review traffic metrics
* Validate service availability
* Measure attack volume
* Verify mitigation effectiveness

Success Criteria:

* Traffic approaches baseline
* Services remain available
* Customer impact is minimized

Continue monitoring for at least 30 minutes after stabilization.

---

# Phase 4 – Business Protection

Priority Order:

1. Customer Portals
2. Revenue APIs
3. VPN Services
4. DNS Services
5. Email Infrastructure
6. Internal Services

## Business Impact Assessment

Document:

* Affected services
* Affected users
* Downtime
* Revenue impact
* SLA impact

---

# Customer Communication

Coordinate with:

* Management
* Public Relations
* Customer Support

Only approved communications should be released externally.

---

# Phase 5 – Recovery

## Goal

Return infrastructure to normal operation safely.

Actions:

* Validate service availability
* Verify customer access
* Remove temporary controls
* Confirm monitoring coverage
* Validate traffic normalization

Recovery should begin only after traffic remains stable for an appropriate observation period.

---

# Recovery Validation Checklist

* Service availability confirmed
* Performance metrics normalized
* Error rates normalized
* Monitoring alerts stabilized
* Temporary controls reviewed
* Business owner approval obtained

---

# Phase 6 – Evidence Collection & Handoff

Collect:

* Firewall Logs
* WAF Logs
* CDN Logs
* NetFlow Data
* ISP Tickets
* Change Records
* Dashboards
* Screenshots
* Search Results
* Approval Records

> Note: Validate log availability and completeness. DDoS incidents may reveal logging gaps, missing telemetry, or monitoring blind spots requiring remediation.

## Incident Timeline

Document:

* Detection
* Escalation
* Mitigation
* Stabilization
* Recovery
* Closure

## Handoff Summary

Include:

* Incident window
* Attack type
* Peak volume
* Peak requests per second
* Unique attackers
* Target systems
* Actions taken
* Approvals
* Current status

Status Categories:

* Active
* Contained
* Monitoring
* Resolved

---

# Phase 7 – Post-Incident Review

Complete within 24-36 hours.

## Root Cause Review

Evaluate:

* Attack effectiveness
* Detection effectiveness
* Escalation effectiveness
* Mitigation effectiveness
* Control failures
* Process gaps

## Lessons Learned

Document:

* What worked well
* What failed
* Recommended improvements
* Required engineering changes

---

# Metrics of Success

Track:

* Time to Detect (TTD)
* Time to Escalate (TTE)
* Time to Mitigate (TTM)
* Customer Downtime
* Revenue Impact
* Number of Affected Systems

---

# Common Mistakes

Don't:

* Block entire countries without approval
* Disable security controls
* Make routing changes without rollback plans
* Assume all traffic spikes are malicious
* Ignore business impact
* Remove evidence before closure

---

# Hardening Recommendations

* Always-On CDN Protection
* DDoS Scrubbing Agreement with ISP
* WAF Deployment
* FlowSpec Readiness
* BGP Diversion Procedures
* Quarterly Tabletop Exercises
* Annual DDoS Simulations
* Continuous Baseline Tuning
* Automated Alerting
* Regular Incident Response Reviews
