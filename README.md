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