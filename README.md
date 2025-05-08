# OpenShift 4.16 - UPI Installation Guide (Helper Node Based)

This guide outlines a step-by-step process to deploy **OpenShift 4.16** using the **User Provisioned Infrastructure (UPI)** method with a **helper node**. The setup includes:
- 🖥️ 3 Masters
- ⚙️ 10 Workers
- 🗄️ 4 Storage Nodes
- 🧰 1 Helper Node (DNS, HTTP, Load Balancer, PXE optional)

---

## 📦 Environment Overview

| Component       | Count | Purpose                                |
|----------------|-------|----------------------------------------|
| Master Nodes    | 3     | Control Plane                          |
| Worker Nodes    | 10    | Application Workloads                  |
| Storage Nodes   | 4     | Ceph/ODF (labeled separately)          |
| Helper Node     | 1     | Bootstraps everything (infra services) |

---

## 🛠️ Prerequisites

- RHEL 8/9 or compatible OS on the helper
- `openshift-install` & `oc` CLI tools (4.16)
- Static IPs & DNS hostnames for all nodes
- Access to Red Hat Pull Secret

---

## 📋 Step-by-Step Instructions

### 1. Prepare Helper Node

```bash
dnf install -y httpd bind-utils vim wget net-tools unzip
systemctl enable --now httpd
