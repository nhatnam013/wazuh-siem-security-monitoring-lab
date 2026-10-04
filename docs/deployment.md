# Deployment Guide

## 1. Overview

This document describes the deployment process for the Wazuh SIEM monitoring laboratory.

The deployment consists of:

- Wazuh Manager deployed on AWS EC2
- Wazuh Indexer
- Wazuh Dashboard
- Ubuntu Web Server
- Wazuh Agent installed on the Web Server
- Kali Linux used for controlled security testing

The objective is to establish a functional security monitoring pipeline in which endpoint logs are collected by the Wazuh Agent and forwarded to the Wazuh Manager for analysis and visualization.

---

## 2. Deployment Architecture

The main deployment flow is:

```text
                    AWS Cloud
        ┌───────────────────────────────┐
        │                               │
        │     Wazuh Manager EC2         │
        │                               │
        │  ┌─────────────────────────┐  │
        │  │ Wazuh Manager           │  │
        │  │ Wazuh Indexer           │  │
        │  │ Wazuh Dashboard         │  │
        │  └─────────────────────────┘  │
        │                               │
        └───────────────┬───────────────┘
                        │
                        │ Wazuh Agent
                        │ Security Events
                        │
                        ▼
              ┌─────────────────────┐
              │ Ubuntu Web Server   │
              │                     │
              │ Apache2             │
              │ WordPress           │
              │ MySQL / PHP         │
              │ Wazuh Agent         │
              └─────────────────────┘
                        ▲
                        │
                        │ HTTP Requests
                        │
              ┌─────────┴─────────┐
              │    Kali Linux     │
              │ Security Testing  │
              │ Nikto / Hydra     │
              └───────────────────┘
```

The Wazuh Manager provides centralized monitoring, while the Ubuntu Web Server acts as the monitored endpoint.

---

## 3. Wazuh Manager Deployment

### 3.1 AWS EC2 Instance

The Wazuh Manager was deployed on an AWS EC2 instance.

The laboratory configuration uses:

| Resource | Configuration |
|---|---|
| Platform | AWS EC2 |
| CPU | 2 vCPU |
| Memory | 8 GB RAM |
| Operating System | Ubuntu |
| Role | Wazuh All-in-One |
| Wazuh Components | Manager, Indexer, Dashboard |

The Wazuh All-in-One deployment combines the main Wazuh components on a single server for laboratory and demonstration purposes.

This configuration is intended for a small-scale security monitoring environment rather than a production deployment.

---

## 4. Wazuh All-in-One Installation

The Wazuh All-in-One environment was installed on the AWS EC2 instance.

The deployment includes:

```text
Wazuh Manager
      │
      ├── Wazuh Indexer
      │
      └── Wazuh Dashboard
```

After installation, the Wazuh Dashboard was accessed through the configured HTTPS service to verify that the Wazuh platform was operational.

The installation was considered successful after the following components became available:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

---

## 5. AWS Security Group Configuration

The EC2 Security Group was configured to allow the network traffic required by the Wazuh environment and administrative access.

The main ports used in the laboratory are:

| Port | Protocol | Purpose |
|---|---|---|
| `22` | TCP | SSH administration |
| `443` | TCP | Wazuh Dashboard HTTPS |
| `1514` | TCP/UDP | Wazuh Agent communication |
| `1515` | TCP | Wazuh Agent enrollment |

Only the required ports should be exposed.

For a production deployment, administrative and Wazuh communication ports should be restricted to trusted source addresses whenever possible.

---

## 6. Ubuntu Web Server Deployment

A separate Ubuntu EC2 instance was deployed as the monitored Web Server.

The Web Server provides the application environment used for security monitoring and controlled attack simulations.

The main services include:

```text
Ubuntu
│
├── Apache2
├── PHP
├── MySQL
├── WordPress
└── Wazuh Agent
```

The Web Server uses a smaller instance configuration because its primary role is to generate application and endpoint telemetry for the SIEM.

---

## 7. Apache Web Server

Apache2 was installed and enabled on the Ubuntu Web Server.

The default Apache service was initially used to verify basic HTTP connectivity.

The service can be checked with:

```bash
sudo systemctl status apache2
```

A successful service status confirms that Apache is running.

The Web Server generates HTTP access and error logs that can be monitored by the Wazuh Agent.

Typical Apache log locations include:

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
```

These logs provide useful telemetry for detecting suspicious web activity.

---

## 8. WordPress Application

WordPress was deployed on the Ubuntu Web Server as the application target for the controlled security testing scenarios.

The application provides a realistic web environment for testing:

- Web reconnaissance
- HTTP request monitoring
- Login brute-force detection
- Application-level security events

The WordPress login endpoint used during testing was:

```text
/wp-login.php
```

---

## 9. Wazuh Agent Deployment

The Wazuh Agent was installed on the Ubuntu Web Server.

The purpose of the agent is to collect endpoint and application security information and forward events to the Wazuh Manager.

The deployment flow is:

```text
Ubuntu Web Server
       │
       │
       ▼
Wazuh Agent
       │
       │ Security Events
       ▼
Wazuh Manager
```

The Agent was configured to communicate with the Wazuh Manager using the Manager's network address.

---

## 10. Agent Configuration

The Wazuh Agent configuration was updated to point to the Wazuh Manager.

The configuration file is:

```text
/var/ossec/etc/ossec.conf
```

The Manager address is defined in the agent configuration.

After configuration changes, the Wazuh Agent service was restarted.

Example:

```bash
sudo systemctl restart wazuh-agent
```

The service status can then be verified with:

```bash
sudo systemctl status wazuh-agent
```

---

## 11. Agent Registration and Connectivity

After installing and configuring the Wazuh Agent, the agent was registered with the Wazuh Manager.

The Wazuh Dashboard was then used to verify the agent connection.

The expected result is:

```text
Agent Status: Active
```

An active status confirms that the endpoint is successfully communicating with the Wazuh Manager.

The agent connectivity was successfully validated before performing the attack simulations.

---

## 12. Log Collection Verification

After the Agent became active, endpoint logs were verified on the Wazuh Manager.

The Wazuh environment stores collected events under the Wazuh log directories.

The archived event log can be inspected with:

```bash
sudo tail -f /var/ossec/logs/archives/archives.log
```

The appearance of endpoint events confirms that the monitoring pipeline is receiving data from the monitored system.

The basic monitoring pipeline is therefore:

```text
Web Server
    │
    │ Apache / System Logs
    ▼
Wazuh Agent
    │
    │ Agent Communication
    ▼
Wazuh Manager
    │
    ▼
Wazuh Indexer
    │
    ▼
Wazuh Dashboard
```

---

## 13. Monitoring Validation

Before conducting security testing, the following conditions were verified:

- Wazuh Manager was operational.
- Wazuh Dashboard was accessible.
- Wazuh Agent was installed on the Web Server.
- The Agent was registered with the Manager.
- The Agent status was shown as Active.
- Security events were visible on the Wazuh platform.
- Apache was running on the monitored Web Server.

This validation ensures that the monitoring infrastructure is functional before generating attack traffic.

---

## 14. Security Testing Environment

Kali Linux was used as the controlled security testing system.

The testing environment contains:

```text
Kali Linux
    │
    ├── Nikto
    │
    └── Hydra
          │
          ▼
   Ubuntu Web Server
          │
          ▼
     Wazuh Agent
          │
          ▼
    Wazuh Manager
```

The security testing was performed only against the laboratory Web Server.

---

## 15. Nikto Testing

Nikto was used to perform web server reconnaissance and vulnerability scanning against the monitored Web Server.

The purpose of the test was to generate abnormal HTTP traffic and verify that Wazuh could identify the resulting activity.

The simplified workflow is:

```text
Kali Linux
    │
    │ Nikto Scan
    ▼
Ubuntu Web Server
    │
    │ Apache Logs
    ▼
Wazuh Agent
    │
    ▼
Wazuh Manager
    │
    ▼
Security Alert
```

The Wazuh environment detected abnormal HTTP GET activity generated during the scan.

The corresponding detection was associated with:

```text
Rule ID: 31101
Level: 5/10
```

The resulting alert was reviewed through the Wazuh Dashboard.

---

## 16. Hydra Testing

Hydra was used to simulate repeated login attempts against the WordPress login endpoint.

The target endpoint was:

```text
/wp-login.php
```

The purpose of the test was to generate repeated authentication requests and verify Wazuh's ability to identify brute-force activity.

The simplified workflow is:

```text
Kali Linux
    │
    │ Hydra
    ▼
/wp-login.php
    │
    │ Repeated HTTP POST
    ▼
Ubuntu Web Server
    │
    ▼
Wazuh Agent
    │
    ▼
Wazuh Manager
    │
    │ Rule 40501
    ▼
High-Level Security Alert
```

The resulting detection was associated with:

```text
Rule ID: 40501
Level: 10/10
```

The alert was reviewed through the Wazuh Dashboard.

---

## 17. Troubleshooting

During the laboratory deployment, troubleshooting was performed when required to ensure communication between the monitored endpoint and the Wazuh Manager.

Common validation points include:

### Check Wazuh Agent service

```bash
sudo systemctl status wazuh-agent
```

### Restart the Agent

```bash
sudo systemctl restart wazuh-agent
```

### Check Agent logs

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

### Check collected events

```bash
sudo tail -f /var/ossec/logs/archives/archives.log
```

### Check Apache service

```bash
sudo systemctl status apache2
```

These checks help identify whether an issue is related to the endpoint service, network connectivity, log collection, or the Wazuh monitoring pipeline.

---

## 18. Final Deployment State

After completing the deployment and validation process, the laboratory reached the following state:

```text
                         ┌────────────────────────┐
                         │      AWS EC2            │
                         │                         │
                         │  Wazuh All-in-One       │
                         │  ┌───────────────────┐  │
                         │  │ Wazuh Manager     │  │
                         │  │ Wazuh Indexer     │  │
                         │  │ Wazuh Dashboard   │  │
                         │  └───────────────────┘  │
                         └───────────┬────────────┘
                                     │
                                     │ Wazuh Agent
                                     │
                                     ▼
                         ┌────────────────────────┐
                         │ Ubuntu Web Server      │
                         │                        │
                         │ Apache2                │
                         │ WordPress              │
                         │ MySQL / PHP            │
                         │ Wazuh Agent            │
                         └───────────▲────────────┘
                                     │
                              Attack Traffic
                                     │
                         ┌───────────┴────────────┐
                         │      Kali Linux        │
                         │                        │
                         │  Nikto / Hydra         │
                         └────────────────────────┘
```

The final environment successfully demonstrates:

- Centralized endpoint monitoring
- Wazuh Agent deployment
- Agent-to-Manager communication
- Security log collection
- Web activity monitoring
- Attack simulation
- Security alert generation
- Basic SOC-style investigation

---

## 19. Deployment Result

The deployment successfully established a functional Wazuh-based SIEM monitoring environment.

The Web Server was successfully connected to the Wazuh Manager through the Wazuh Agent, and security events were visible through the Wazuh monitoring platform.

The subsequent Nikto and Hydra simulations generated detectable security events, demonstrating the complete workflow from attack activity to centralized security alerting.

This deployment provides the foundation for the detection scenarios documented in:

```text
docs/detection-scenarios.md
```

---

## 20. Security Considerations

This project is a controlled security laboratory.

The following security practices should be applied when adapting the architecture for a production environment:

- Restrict AWS Security Group source addresses.
- Avoid exposing administrative ports to the entire Internet.
- Use strong credentials and multi-factor authentication.
- Protect Wazuh Dashboard access.
- Keep operating systems and Wazuh components updated.
- Avoid storing credentials, API keys, or private keys in the repository.
- Use least-privilege access wherever possible.
- Separate production monitoring infrastructure from attack-testing environments.

All security testing in this project was performed against systems under the project's control.
