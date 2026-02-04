# MinIO Ansible Role

---

## Table of Contents

1. [Overview](#1-overview)
2. [Supported Operating Systems](#2-supported-operating-systems)
3. [Prerequisites & Known Limitations](#3-prerequisites--known-limitations)
4. [Role Structure](#4-role-structure)
5. [Configuration Overview](#5-configuration-overview)
6. [Installation Flow](#6-installation-flow)
7. [Running the Playbook](#7-running-the-playbook)
8. [Validation & Testing](#8-validation--testing)
9. [Best Practices Followed](#9-best-practices-followed)
10. [Troubleshooting](#10-troubleshooting)
11. [Conclusion](#11-conclusion)
12. [References](#12-references)
13. [Author](#13-author)

---

## 1. Overview

**MinIO** is an open-source, high-performance, **S3-compatible object storage platform**.

It is widely used for:

* Cloud-native and platform services
* CI/CD artifacts and build outputs
* Backup, restore, and archival workloads
* Observability data (metrics, logs, traces)
* Stateful workloads requiring object storage

This Ansible role provides a **standardized, production-oriented MinIO deployment** with:

* Single-node and **distributed (HA) mode**
* Strict persistent storage validation
* TLS / HTTPS support
* Optional DNS and Load Balancer integration
* Prometheus-compatible metrics exposure
* Optional OIDC / SSO integration
* Declarative user and policy management using `mc`

Official documentation:
[https://min.io/docs/](https://min.io/docs/)

---

## 2. Supported Operating Systems

| OS Family | Versions                             |
| --------- | ------------------------------------ |
| Debian    | Ubuntu 20.04+, Debian 10+            |
| RedHat    | RHEL 8+, CentOS 8+, Rocky, AlmaLinux |

OS compatibility is validated during role execution.

---

## 3. Prerequisites & Known Limitations

### System Requirements

| Requirement | Description                           |
| ----------- | ------------------------------------- |
| RAM         | 4 GB minimum (HA: 8 GB recommended)   |
| CPU         | 2 vCPU or more                        |
| Disk        | Dedicated mounted filesystem required |
| Network     | Open API & Console ports              |

### Python & Ansible Requirements

| Package | Purpose            |
| ------- | ------------------ |
| ansible | Playbook execution |

Installation:

```bash
pip install ansible
```

Recommended environment:

```bash
python3 -m venv ansible-venv
source ansible-venv/bin/activate
pip install ansible
```

### Known Platform Limitations

The following are **MinIO platform constraints**, not role limitations:

* Filesystems and disks are **not provisioned** by this role
* Persistent storage **must be mounted prior to execution**
* Distributed (HA) mode requires **minimum 4 endpoints** for quorum
* TLS certificates **must exist on the host**
* DNS is optional, but strictly validated when enabled
* Root credentials **must be injected externally** (Vault / CI secrets)

---

## 4. Role Structure

```
roles/
└── minio/
    ├── defaults/main.yml
    ├── vars/main.yml
    ├── handlers/main.yml
    ├── tasks/
    │   ├── main.yml
    │   ├── prerequisites.yml
    │   ├── install.yml
    │   ├── configure.yml
    │   ├── security.yml
    │   ├── observability.yml
    │   ├── service.yml
    │   ├── access.yml
    │   └── verify.yml
    ├── templates/
    │   ├── minio.env.j2
    │   ├── minio.service.j2
    │   └── policies/
    │       ├── read-only.json.j2
    │       └── write-all.json.j2
    └── meta/main.yml

inventory.ini  
playbook.yml  
group_vars/
```

---

## 5. Configuration Overview

MinIO configuration is fully **variable-driven** and defined in
`roles/minio/defaults/main.yml`.

| Configuration Area | Description                               |
| ------------------ | ----------------------------------------- |
| Core Runtime       | Binary, service, ports                    |
| Storage            | Data directories and mount validation     |
| TLS / HTTPS        | Certificate-based encryption              |
| DNS                | Optional FQDN / LB endpoint identity      |
| High Availability  | Distributed MinIO mode                    |
| Security           | Non-root execution & permission hardening |
| Observability      | Prometheus metrics exposure               |
| Access Management  | Users and policies via `mc`               |
| SSO                | Optional OIDC integration                 |

---

## 6. Installation Flow

High-level execution flow:

1. Validate OS compatibility
2. Validate persistent storage mounts
3. Validate credentials and HA constraints
4. Create MinIO system user and group
5. Download MinIO binary and client (`mc`)
6. Prepare data and configuration directories
7. Render runtime environment configuration
8. Validate TLS and OIDC inputs
9. Install systemd service
10. Enable and start MinIO
11. Configure users and policies (optional)
12. Verify service readiness and endpoints

<img width="524" height="736" alt="image" src="https://github.com/user-attachments/assets/dd9ab9ea-7b39-46bd-9e79-49ee7fd21728" />

---

## 7. Running the Playbook

```bash
ansible-playbook -i inventory.ini playbook.yml
```
---

## 8. Validation & Testing

### Service Status

```bash
sudo systemctl status minio
```
---

### Browser & API Access

```
https://<MINIO_HOST>:8443
```

---

### Metrics Validation

```bash
curl -k https://<MINIO_HOST>:9000/minio/v2/metrics/cluster
```
---

## 9. Best Practices Followed

| Practice              | Description             |
| --------------------- | ----------------------- |
| OS awareness          | Explicit OS validation  |
| Strict storage checks | Prevents data loss      |
| Non-root execution    | Security hardening      |
| TLS-first design      | Secure-by-default       |
| Variable-driven       | No hardcoded values     |
| Idempotent            | Safe re-runs            |
| HA guardrails         | Enforced quorum rules   |
| Local admin access    | `mc` bound to localhost |

---

## 10. Troubleshooting

| Issue                    | Fix                            |
| ------------------------ | ------------------------------ |
| MinIO not starting       | `journalctl -u minio -n 50`    |
| TLS errors               | Verify cert paths & ownership  |
| Mount validation failure | Check `findmnt` output         |
| HA assertion failure     | Ensure 4+ endpoints            |
| Metrics not scraping     | Validate public metrics config |
| Console redirect issues  | Verify redirect URL            |

---

## 11. Conclusion

This role provides a **reusable, secure, and HA-aware MinIO foundation**.By externalizing configuration and enforcing safety checks, it enables consistent MinIO deployments across environments while aligning with platform-engineering best practices.

---

## 12. References

| Purpose                 | Link                                                                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| MinIO Documentation     | [https://min.io/docs/](https://min.io/docs/)                                                                                                                                                     |
| MinIO Distributed Guide | [https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-distributed.html](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-distributed.html) |
| Ansible Documentation   | [https://docs.ansible.com](https://docs.ansible.com)                                                                                                                                             |

---

## 13. Author

**Author:** Divya Mishra

**Last Updated:** 4-Feb-2026

---
