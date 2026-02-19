# Proxmox Lab Infrastructure Training

A comprehensive 15-day training project demonstrating enterprise infrastructure setup from bare metal to production-ready services.

## About This Project

This is a comprehensive training project demonstrating enterprise infrastructure setup using:

- **Virtualization:** Proxmox VE 8.2 with ZFS storage
- **Operating System:** Ubuntu Server 24.04 LTS
- **Collaboration:** Rocket.Chat 6.x, Nextcloud 28.x
- **Identity Management:** Keycloak 24.0.0, OpenLDAP 2.5.x
- **Security:** CIS benchmarks, Fail2Ban, SSL/TLS
- **Monitoring:** Prometheus 2.51.0, Grafana 10.x

**Note:** This documentation uses `example.com` as a placeholder domain per RFC 2606 standards. In a production environment, this would be replaced with your actual domain.

**Training Timeline:** 15 days from bare metal to production-ready infrastructure

## Skills Demonstrated

- Infrastructure as Code principles
- Enterprise authentication and SSO (Single Sign-On)
- Security hardening and compliance
- Technical documentation and knowledge management
- Virtualization and container orchestration
- Monitoring and observability
- Backup and disaster recovery

## Architecture Overview

Infrastructure consists of:
- 1 Proxmox host with ZFS storage
- 6 Virtual Machines running Ubuntu 24.04 LTS
- Centralized authentication via Keycloak + OpenLDAP
- Reverse proxy with SSL termination
- Comprehensive monitoring stack

## Documentation Structure

- **01-preparation/**: Physical server setup (Days 1)
- **02-vm-creation/**: Virtual machine creation (Day 2)
- **03-software/**: Service installation (Days 3-8)
- **04-security/**: Hardening and security (Days 9-10)
- **05-monitoring/**: Observability setup (Day 11)
- **06-maintenance/**: Operations and maintenance
- **07-diagrams/**: Architecture diagrams

## Live Documentation

View the full documentation at: https://dmaljkovic.github.io/proxmox-lab/

## Technologies Used

- Proxmox VE 8.2, Ubuntu Server 24.04 LTS
- Docker 24.x, Rocket.Chat 6.x, Nextcloud 28.x
- Keycloak 24.0.0, OpenLDAP 2.5.x
- Nginx 1.24, Prometheus 2.51.0, Grafana 10.x
- MkDocs with Material theme

## Author

**dmaljkovic** - Infrastructure Training Project
