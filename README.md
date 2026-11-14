# Rocky Linux CIS Hardening Lab

This project demonstrates a simple Ansible-based hardening baseline for Rocky Linux 9 nodes running in a KVM/libvirt lab environment. The goal is to automate common CIS-aligned security controls for enterprise Linux systems, including package updates, SSH hardening, service reduction, kernel tuning, and password aging enforcement.

## Overview

The lab consists of a three-node environment:

- vm01
- vm02
- vm03

The inventory file defines the managed hosts and the Ansible credentials used to connect to them.

## Project structure

- `inventory.ini` — host inventory and connection settings
- `site.yaml` — Ansible playbook that applies the hardening changes
- `README.md` — project documentation

## Environment

- OS: Rocky Linux 9
- Hypervisor: KVM / libvirt
- Automation: Ansible
- Security baseline: CIS-inspired hardening tasks

## What the playbook does

The playbook currently performs the following tasks:

- Updates all installed packages with `dnf`
- Checks whether a reboot is required after package changes
- Hardens SSH by enforcing:
  - `PermitRootLogin prohibit-password`
  - `PasswordAuthentication no`
- Stops and disables unnecessary services when present
- Applies kernel hardening with `sysctl`:
  - `net.ipv4.ip_forward = 0`
  - `fs.suid_dumpable = 0`
- Configures password aging in `/etc/login.defs`:
  - `PASS_MAX_DAYS 90`
  - `PASS_MIN_DAYS 7`
  - `PASS_WARN_AGE 7`
- Restarts `sshd` when SSH configuration changes

## Prerequisites

Before running the lab, ensure the following are in place:

- Rocky Linux 9 hosts are reachable over SSH
- Ansible is installed on the control node
- Root or sudo access is available to the managed systems
- The inventory file contains the correct host IPs and credentials

## Inventory

The inventory file defines the target group and connection variables:

```ini
[rocky_nodes]
vm01 ansible_host=192.168.122.66
vm02 ansible_host=192.168.122.138
vm03 ansible_host=192.168.122.45

[all:vars]
ansible_user=root
ansible_password=secret
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

## Usage

Run the playbook from the project root:

```bash
ansible-playbook -i inventory.ini site.yaml
```

To validate connectivity before applying changes:

```bash
ansible -i inventory.ini rocky_nodes -m ping
```

## Example run

A typical execution looks like this:

```text
PLAY [rocky_nodes]

TASK [Gathering Facts]
ok: [vm01]
ok: [vm02]
ok: [vm03]

TASK [Update all system packages]
ok: [vm01]
ok: [vm02]
ok: [vm03]

TASK [Hardening SSH configuration]
ok: [vm01]
ok: [vm02]
ok: [vm03]

TASK [CIS Benchmark - Kernel hardening via sysctl]
changed: [vm01]
changed: [vm02]
changed: [vm03]

PLAY RECAP
vm01: ok=6 changed=2 ...
vm02: ok=6 changed=2 ...
vm03: ok=6 changed=2 ...
```

## Notes and considerations

- Some services listed for shutdown may not exist on every Rocky Linux base image. Those cases are handled gracefully with `ignore_errors: true`.
- The playbook is intentionally a baseline hardening lab and is not a full CIS benchmark implementation.
- After package updates, a reboot may still be required depending on the system state and kernel changes.
- This lab is useful as a foundation for expanding into more advanced controls, such as firewall rules, audit settings, file permissions, and centralized logging.

## Future improvements

Possible next steps for this lab include:

- Adding SELinux policy enforcement checks
- Configuring a firewall baseline with `firewalld` or `nftables`
- Enforcing file permission standards and audit policies
- Storing secrets securely with Ansible Vault
- Splitting the playbook into role-based components for better reuse

## License

This project is intended for educational and lab use. Modify and extend it based on your environment and security requirements.
