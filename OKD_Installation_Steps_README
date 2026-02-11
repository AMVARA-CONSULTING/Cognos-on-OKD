# OKD Installation Steps on Ubuntu 24.04 Server (POC)

This README provides step-by-step installation instructions for installing **OKD (OpenShift Kubernetes Distribution)** on an **Ubuntu 24.04 LTS** server for a **POC deployment** (IBM Cognos use case).

Your server specs are excellent for this installation:

- Virtualization enabled (`vmx/svm` detected)
- KVM acceleration supported
- Ubuntu 24.04 LTS
- 62GB RAM, 40 vCPU

Recommended deployment type:

✅ **Single Node OKD Cluster (SNO)**

This is the best option for POC since it avoids multi-node load balancer complexity.

---

# 1️⃣ Prerequisites Checklist

## Firewall / Ports
Ensure required ports are open:

```bash
sudo ufw allow 6443/tcp
sudo ufw allow 22623/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 53/udp
sudo ufw allow 53/tcp
sudo ufw allow 9000:9999/tcp
sudo ufw reload
```

For POC, you can disable firewall completely:

```bash
sudo ufw disable
```

---

# 2️⃣ Create DNS Records (Very Important)

You need a domain, for example:

- `okd.lab.local`

DNS records required:

| Record | Target |
|--------|--------|
| api.okd.lab.local | your server IP |
| api-int.okd.lab.local | your server IP |
| *.apps.okd.lab.local | your server IP |

Example:

```
api.okd.lab.local     -> 192.168.1.50
api-int.okd.lab.local -> 192.168.1.50
*.apps.okd.lab.local  -> 192.168.1.50
```

If you do not have a DNS server, you can use `/etc/hosts` for testing (not ideal but works for POC).

---

# 3️⃣ Install Required Tools

```bash
sudo apt update
sudo apt install -y curl wget jq podman openssh-client
```

---

# 4️⃣ Download OKD Installer + OC Client

Go to OKD releases:

https://github.com/okd-project/okd/releases

Example download commands (replace with latest version URLs):

```bash
mkdir okd
cd okd

wget https://github.com/okd-project/okd/releases/download/4.15.0-0.okd-2024-xx-xx/openshift-install-linux.tar.gz
wget https://github.com/okd-project/okd/releases/download/4.15.0-0.okd-2024-xx-xx/openshift-client-linux.tar.gz
```

Extract and install:

```bash
tar -xvf openshift-install-linux.tar.gz
tar -xvf openshift-client-linux.tar.gz

sudo mv openshift-install /usr/local/bin/
sudo mv oc kubectl /usr/local/bin/
```

Verify:

```bash
openshift-install version
oc version
```

---

# 5️⃣ Create Installation Directory

```bash
mkdir ~/okd-sno
cd ~/okd-sno
```

---

# 6️⃣ Create install-config.yaml

Run:

```bash
openshift-install create install-config --dir=.
```

It will ask for:

- Base domain (example: `lab.local`)
- Cluster name (example: `okd`)
- Platform: **none** (bare metal style)
- SSH public key
- Pull secret

⚠ OKD requires a pull secret. You can use a RedHat pull secret (free account) or OKD community pull secret.

---

# 7️⃣ Modify install-config.yaml for Single Node

Edit:

```bash
nano install-config.yaml
```

Modify the cluster node count:

```yaml
controlPlane:
  name: master
  replicas: 1

compute:
- name: worker
  replicas: 0
```

This creates a **Single Node OKD cluster**.

---

# 8️⃣ Generate Ignition Configs

```bash
openshift-install create ignition-configs --dir=.
```

This will generate:

- bootstrap.ign
- master.ign
- worker.ign

---

# 9️⃣ Download Fedora CoreOS / OKD CoreOS ISO

OKD runs on CoreOS. Your Ubuntu server will host OKD inside a VM.

Download Fedora CoreOS ISO:

```bash
wget https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/latest/x86_64/fedora-coreos-*.iso
```

---

# 🔟 Create VM in KVM for OKD Node

Install libvirt + KVM tools:

```bash
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients virt-manager bridge-utils
sudo systemctl enable --now libvirtd
```

Create VM with virt-install:

```bash
sudo virt-install --name okd-sno --ram 32768 --vcpus 8 --disk path=/var/lib/libvirt/images/okd-sno.qcow2,size=200 --os-variant fedora-coreos-stable --network network=default --graphics none --console pty,target_type=serial --cdrom fedora-coreos.iso
```

---

# 1️⃣1️⃣ Inject Ignition into CoreOS VM

Serve ignition files using Python HTTP server:

```bash
cd ~/okd-sno
python3 -m http.server 8080
```

During CoreOS installation, provide ignition URL:

```
http://<ubuntu-server-ip>:8080/master.ign
```

---

# 1️⃣2️⃣ Start Cluster Installation

Once the VM is booting with ignition:

```bash
openshift-install wait-for bootstrap-complete --dir=. --log-level=info
```

Then:

```bash
openshift-install wait-for install-complete --dir=. --log-level=info
```

---

# 1️⃣3️⃣ Export Kubeconfig and Login

```bash
export KUBECONFIG=~/okd-sno/auth/kubeconfig
oc get nodes
```

Expected output:

- 1 master node in Ready state

---

# 1️⃣4️⃣ Open Web Console

Console URL will look like:

```
https://console-openshift-console.apps.okd.lab.local
```

Get admin password:

```bash
cat ~/okd-sno/auth/kubeadmin-password
```

---

# ✅ Post Installation (Important for IBM Cognos)

IBM Cognos often requires relaxed security context.

Create namespace:

```bash
oc new-project cognos
```

Allow anyuid SCC:

```bash
oc adm policy add-scc-to-user anyuid -z default -n cognos
```

---

# ⭐ Recommended VM Specs for OKD SNO

Suggested allocation:

- 8 vCPU
- 32GB RAM
- 200GB disk

This leaves enough resources for:

- Cognos pods
- Database pods
- other workloads

---

# 🎉 Summary

Your server is fully compatible for OKD installation:

- virtualization enabled
- KVM supported
- sufficient CPU/RAM/disk

OKD SNO is the best option for your IBM Cognos POC.

---

**End of README**
