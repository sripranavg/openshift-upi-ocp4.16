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
Place OpenShift CLI binaries:

bash
Copy
Edit
cp openshift-install /usr/local/bin/
cp oc /usr/local/bin/
chmod +x /usr/local/bin/openshift-install /usr/local/bin/oc
2. Create install-config.yaml
bash
Copy
Edit
mkdir ~/ocp-install && cd ~/ocp-install
openshift-install create install-config --dir=.
Replace with:

yaml
Copy
Edit
apiVersion: v1
baseDomain: inficsocp.com
metadata:
  name: lab
compute:
  - name: worker
    replicas: 0
controlPlane:
  name: master
  replicas: 3
networking:
  networkType: OVNKubernetes
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '<your_pull_secret>'
sshKey: '<your_ssh_key>'
3. Generate Manifests & Ignition
bash
Copy
Edit
openshift-install create manifests --dir=.
openshift-install create ignition-configs --dir=.
Files created:

bootstrap.ign

master.ign

worker.ign

4. Host Ignition Files
bash
Copy
Edit
mkdir -p /var/www/html/ocp
cp *.ign /var/www/html/ocp/
restorecon -R /var/www/html/ocp
systemctl restart httpd
5. Configure DNS (Bind)
Sample entries:

css
Copy
Edit
api.lab.inficsocp.com.     IN A <load_balancer_vip>
api-int.lab.inficsocp.com. IN A <load_balancer_vip>
*.apps.lab.inficsocp.com.  IN A <apps_vip>
master-0.lab.inficsocp.com. IN A <IP>
worker-0.lab.inficsocp.com. IN A <IP>
6. Set Up Load Balancer (HAProxy example)
Proxy 6443 to master IPs and 80/443 to worker/storage IPs.

7. Boot Nodes with Ignition
Bootstrap node → bootstrap.ign

Masters (3) → master.ign

Workers (10) + Storage (4) → worker.ign

8. Monitor Bootstrap
bash
Copy
Edit
openshift-install wait-for bootstrap-complete --dir=.
Once complete, shut down bootstrap node.

9. Approve CSRs
bash
Copy
Edit
export KUBECONFIG=~/ocp-install/auth/kubeconfig
oc get csr
oc adm certificate approve <csr>
10. Label Storage Nodes
bash
Copy
Edit
oc label node storage-0 node-role.kubernetes.io/storage=""
(Optional):

bash
Copy
Edit
oc taint node storage-0 node-role.kubernetes.io/storage=:NoSchedule
11. Complete Installation
bash
Copy
Edit
openshift-install wait-for install-complete --dir=.
✅ Done! Your OpenShift UPI Cluster is Ready.
