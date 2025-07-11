# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PostgreSQL High Availability (HA) experiment using Podman containers on RockyLinux VMs. The goal is to set up and test a PostgreSQL Master/Slave configuration with failover capabilities.

## Environment Setup

- **Target Platform**: RockyLinux (ARM-based for development on macOS)
- **Container Runtime**: Podman with rootless containers
- **PostgreSQL Version**: 16
- **Orchestration**: Ansible for VM provisioning and configuration
- **Virtualization**: UTM on macOS for local development

## Key Components

- **VMs**: Two RockyLinux VMs (`pg1` as master, `pg2` as slave)
- **Networking**: Bridge network with fixed IP addresses
- **Authentication**: SSH key-based authentication using `podman-postgres-ha-experiment` keypair
- **Integration**: Podman/systemd integration for service management

## Development Workflow

This project is primarily infrastructure-focused using Ansible playbooks. The main development activities involve:

1. **VM Setup**: Creating and configuring RockyLinux VMs with UTM
2. **Network Configuration**: Setting up bridge networking with fixed IPs using `nmcli`
3. **Ansible Playbooks**: Creating automation for PostgreSQL HA setup
4. **Testing**: Failover scenarios and backup/restore procedures

## Prerequisites

```bash
brew install ansible utm
```

## SSH Configuration

The project uses a dedicated SSH key for VM access:
- Private key: `~/.ssh/podman-postgres-ha-experiment`
- Public key: `~/.ssh/podman-postgres-ha-experiment.pub`

## Directory Structure

- `downloads/`: Contains Rocky Linux ISO files (excluded from git)
- `ansible/`: Ansible playbooks and configurations (currently empty)
- `readme.md`: Detailed project documentation and setup instructions

## Testing Scenarios

The project focuses on testing:
- Master/Slave replication setup
- Failover scenarios
- Backup and restore procedures
- Rootless container deployment
- Systemd service integration