# WAZUH SOC LAB

## Overview

This project builds an on-premises Wazuh SIEM to develop SOC analyst skills through centralized monitoring of Windows and Linux endpoints. It integrates with the **DeleDFIR.local Active Directory** domain and Ubuntu systems to generate and analyze security telemetry. The lab covers Wazuh server and dashboard deployment, telemetry validation, custom detection rules, File Integrity Monitoring, Active Response, and full security investigations. Designed as a repeatable SOC environment, it can be extended with detections, threat simulations, investigations, and automation using **Tines, AI, Slack, and human-approved blocking workflows**.

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

---


# Part 3 — Controlled Activity & Log Analysis

## Objective
Simulate Windows/Linux security events and analyze Wazuh telemetry to demonstrate detection and correlation of authentication, account changes, group membership modifications, and SSH activity, with risk assessment applied.

## Scope & Assumptions

Conducted in a controlled lab using the existing **DeleDFIR.local Active Directory environment (DC1 and WS01)** and an **Ubuntu endpoint**. Wazuh served as the centralized monitoring platform for telemetry collection and analysis.

## Skills
- SIEM log analysis & correlation
- Windows/Linux telemetry interpretation (Event ID, SSH)
- SID/RID auditing & group activity trails
- Session tracking & risk assessment

## Tools
- **Wazuh:** Centralized collection, search, filtering, and analysis of Windows and Linux security telemetry.
- **Windows Event Logs:** Account creation, account deletion, logon, and group-membership activity.
- **Ubuntu/Linux:** SSH authentication and session telemetry.
- **PowerShell / Windows CLI:** Controlled account and group activity generation.
- **SSH:** Controlled Linux authentication activity.
- **Sysmon telemetry:** Session-based activity correlation.

## Steps

### Event ID 4726 — Account Deletion
<img src="03_Screenshots/WS01_4726.png">

The **Subject**, identified by **RID 500**, was the Administrator account that executed the action, while the **Target Account**, identified by **RID 1012**, was the `student1` account. By parsing the **Subject SID** and correlating the RID, I confirmed the actor as **Administrator**. I then correlated this event with the controlled deletion of `student1` on WS01 in the **DELEDFIR domain**, validating that the activity matched the expected lab simulation.

### Event ID 4720 — Account Creation
<img src="03_Screenshots/WS01_4720.png">

Interpreted **Event ID 4720** as a user-account creation event, identified the **Subject (RID 500)** as the account that performed the action and the **New Account (RID 1012)** as the account created, parsed the **Subject SID** and used the RID to identify the **Administrator account**, reviewed the account attributes including Primary Group ID `513` as **Domain Users** and New UAC Value `0x15` as indicating Account Disabled, Password Not Required, and Normal Account, and correlated the event with the controlled `student1` account creation performed on WS01 in the **DELEDFIR domain**.

### Event ID 4624 — Successful Logon
<img src="03_Screenshots/WS01_4624.png">

Interpreted **Event ID 4624** as a successful user logon, identified the New Logon account as **`DELEDFIR\john.smith`** and **RID 1239** as the relative identifier at the end of the account SID, identified Logon Type `7` as a re-logon/unlock event, reviewed Logon ID `0x87E1C9` as a correlation point for the logon session, identified **`lsass.exe`** as the process handling the authentication and Negotiate as the authentication package, and correlated the logon with WS01 and the **DELEDFIR domain**.

### Event ID 4732 — Local Group Membership
<img src="03_Screenshots/WS01_4732.png">

Interpreted **Event ID 4732** as a member being added to a security-enabled local group, identified the **Subject (RID 500)** as the account that performed the action and the **Member (RID 1012)** as the account added, used the **Member SID/RID** to identify the affected account as **`student1`** because the Member **Account Name** field was not populated, identified the target local group as Users in the Builtin domain, and correlated the event with the controlled `student1` group-membership change performed on WS01 by `DELEDFIR\Administrator`.

### Linux SSH Authentication

<img src="03_Screenshots/SSH_Session_opened.png">

Identified a **connection reset by invalid user** for `fakeuser` from 192.168.56.1  on port `52603`, identified a **failed password attempt** for fakeuser from the same source IP and port, searched for `dfir AND accepted` and identified a successful SSH authentication for **`dfir`** from 192.168.56.1 on port **`52395`**, and interpreted the **Accepted password** event as successful authentication.

### Linux SSH Session Lifecycle

<img src="03_Screenshots/SSH_Session_Closed.png">

Searched for **`dfir AND session`** to investigate the SSH session lifecycle, confirmed the session was established through the pam_unix(sshd:session): session opened event, used the shared **`sshd` process ID `2345`** to correlate the session-open and session-close events, and confirmed the SSH session started at **`02:37:59`** and closed at **`02:57:08`**.


## Challenges & Troubleshooting
Initial Windows account-management activity did not produce the expected Security events in Wazuh; investigation showed that WS01 was not auditing User Account Management for Success and Failure, so the required audit policy was enabled with `auditpol` and the activity was repeated.

During the Linux investigation, SSH telemetry was initially searched against the Wazuh server instead of the Linux endpoint because the VM IPs were confused; correcting the target to the Linux endpoint restored the expected SSH telemetry and enabled session correlation.

## Summary

**Investigation Findings:** Evidence from Wazuh showed controlled account creation, deletion, local-group membership changes, successful Windows logon activity, and a complete Linux SSH authentication/session lifecycle, including failed authentication followed by successful access and session termination.

**Investigation Rationale:** Wazuh was used as the central investigation point because correlating endpoint authentication, account-management, group-membership, session, and command telemetry provides a single evidence trail for identifying potentially unauthorized activity.

**Validation:** Repeating the controlled activities after correcting the Windows audit configuration and Linux endpoint targeting confirmed that Wazuh captured the expected telemetry, including Windows Events `4720`, `4726`, `4624`, `4732` and Linux SSH authentication/session events.

## Operational Impact

The investigation improved centralized security visibility by enabling analysts to audit and trace account changes, authentication activity, privilege changes, and SSH sessions from correlated endpoint telemetry.