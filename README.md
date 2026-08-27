# WAZUH SOC LAB

## Overview

This project builds an on-premises Wazuh SIEM environment to develop practical SOC analyst experience through centralized security monitoring across Windows and Linux endpoints within the existing **DeleDFIR.local Active Directory lab**. The environment covers Wazuh server and Dashboard deployment, endpoint telemetry collection and validation, custom dashboards and detection rules, File Integrity Monitoring (FIM), Active Response, and end-to-end security investigations. The lab is being developed as a repeatable SOC environment that can be extended with additional detections, threat simulations, investigations, and automation through **Tines, AI, Slack, and human-approved blocking workflows**.

---

# PART 1 — Wazuh SIEM Deployment & Archive Telemetry

## Objective

Deploy and configure Wazuh as the centralized SIEM for the existing DeleDFIR.local lab to establish reliable security telemetry collection and investigation visibility.

## Scope & Assumptions

This is a controlled VirtualBox lab extending the existing DeleDFIR.local Active Directory environment with a dedicated Ubuntu Wazuh server at `192.168.56.120`, focused on SIEM deployment, configuration, and archive telemetry collection before endpoint integration.

## Skills

- SIEM deployment and administration
- Linux system administration
- Security monitoring, telemetry collection, and validation
- Troubleshooting and configuration management
- Service health validation
- Evidence-based technical analysis
- SOC documentation

## Tools

- **Wazuh** — Centralized SIEM for deploying the monitoring platform and collecting archive telemetry.
- **Ubuntu Linux** — Hosts the Wazuh server and provides the platform for configuration and administration.
- **VirtualBox** — Hosts the isolated Wazuh lab environment.
- **Filebeat** — Collects and forwards Wazuh archive telemetry to the Wazuh Indexer.
- **SSH** — Provides remote access from the host to the Wazuh server for administration.


## Steps

<img src="01_Screenshots/Wazuh-Archives-Hits.png">

Configured the Wazuh server and archive telemetry pipeline to retain non-alerting security events, created the `wazuh-archives-*` index pattern, and verified **1,663 events** in Wazuh Discover to confirm the telemetry was successfully indexed and available for investigation.

---

## Challenges & Troubleshooting

The deployment encountered missing credentials, Filebeat configuration and certificate issues, and initially incomplete Wazuh components, with Filebeat configuration tests, service-status checks, certificate inspection, installation-file inspection, and Indexer health checks used to identify the underlying problems.

The issues were resolved by retrieving the required credentials and certificates, correcting the Filebeat and Wazuh configuration, initializing the Wazuh Indexer and Dashboard components, and restarting the affected services.

---

## Summary

**Security Decision:** Archive telemetry was enabled because retaining non-alerting events provides additional investigative evidence that would otherwise be excluded from alert-focused monitoring.

**Validation:** Successful Dashboard access, healthy Indexer status, operational Wazuh services, valid Filebeat configuration, and **1,663 indexed archive events** confirmed that the initial SIEM deployment and archive-collection pipeline were functioning correctly.

## Operational Impact

The deployment establishes a functional centralized SIEM foundation with reliable archive telemetry, improving security visibility and providing a validated platform for subsequent endpoint monitoring and detection capabilities.

---


# Part 2: Windows & Ubuntu Agents + Sysmon

- **Objective:** 
Extend centralized security monitoring by onboarding Windows and Linux endpoints and integrating Sysmon to reduce blind spots in process and network activity.

- **Scope & Assumptions:** 
Lab simulation with two Windows endpoints (DC1, WS01), 2 Ubuntu endpoints (DFIR‑Linux), and a Wazuh server from Part 1 for centralized monitoring.

- **Skills:** 
SIEM administration | endpoint telemetry deployment | Windows/Linux administration | security monitoring | log analysis | troubleshooting | evidence validation | security documentation.

- **Tools:** 
Wazuh Manager/Dashboard for centralized monitoring | Wazuh Agents for endpoint collection | Microsoft Sysmon for deeper telemetry | SSH/PowerShell for deployment | VirtualBox for lab environment.

- **Steps:**

<img src="02_Screenshots/Wazuh_Agent_Coverage.png">

**Agent Deployment & Coverage:** Enrolled DC1, WS01, and DFIR-Linux into Wazuh and verified centralized telemetry from all monitored systems.

<img src="02_Screenshots/DC1-WS01-Linux-active-agents.png">

**Windows Sysmon Integration:** Configured Wazuh to collect the Sysmon Operational channel from DC1 and WS01 and verified Sysmon process and network telemetry in the archive.

**Linux Sysmon Integration:** Installed Sysmon for Linux on DFIR-Linux, applied the `collect-all.xml` configuration, and verified Sysmon events were written to `/var/log/syslog` and forwarded to Wazuh.

**Telemetry Validation:** Executed `uname` on DFIR-Linux and confirmed Wazuh captured the activity as a Linux Sysmon event with `/usr/bin/uname` as the process image.

- **Challenges & Troubleshooting:** 
Windows Sysmon telemetry required explicit `ossec.conf` configuration before events appeared, and restarting the Wazuh Agent resolved the issue; Linux Sysmon required installation and local log validation before telemetry was confirmed in Wazuh.

- **Summary:**
  - **Investigation Findings:** Wazuh archive confirmed telemetry from all endpoints systems—DC1, WS01, DFIR-Linux, and ubuntu-s2—with Sysmon process and network events.
  - **Security Decision:** Sysmon was added to strengthen endpoint visibility and support deeper investigations.
  - **Validation:** Wazuh Discover confirmed active agent coverage and successful Sysmon telemetry, including the captured Linux `uname` execution.

- **Operational Impact:** 
Centralized Sysmon telemetry improved endpoint visibility and strengthened incident-response readiness.