#### Summary of Resource Requirements
```txt
Resource	Minimum Required
--------    ----------------
Memory      27 GB
CPUs        4
Root disk   95 GB
Data disk   145 GB
```


#### Login to Github Enterprise

```link
https://enterprise.github.com/login
```
Now complete your signup


#### Goto Software Download Link:

```link
https://enterprise.github.com/releases/3.18.1/download
```
Then select `GitHub On-premises` -> `OpenStack KVM (QCOW2)` -> `Download for OpenStack KVM (QCOW2)`

**OR**

Login your Linux (Ubuntu) server
```bash
aria2c -x16 -s16 -c "https://github-enterprise.s3.amazonaws.com/kvm/releases/github-enterprise-3.18.1.qcow2"
```

Wait for Download


#### Stop and Remove Any Previous GitHub Enterprise Instance
If you had a previous `.qcow2` GitHub Enterprise image running (e.g., version `3.14.0`), stop and delete it first.

```bash
# List all running VMs
sudo virsh list --all
```
Find the old VM name (for example: `github-enterprise`).
Then destroy and undefine it:

```bash
sudo virsh destroy github-enterprise --graceful
sudo virsh undefine github-enterprise
```

#### Install KVM, libvirt, and virt-install

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
```
Check if KVM is available:

```bash
virsh list --all
```

#### Move the New Image to a Proper Location
```bash
sudo mkdir -p /var/lib/libvirt/images/
sudo mv /path/github-enterprise-3.18.1.qcow2 /var/lib/libvirt/images/
```
#### Create secondary data disk (`≥145 GB`):
```bash
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/github-enterprise-data.qcow2 150G
```

#### Create GitHub Enterprise VM
```bash
sudo virt-install \
  --name github-enterprise \
  --memory 27648 \
  --vcpus 4 \
  --disk path=/var/lib/libvirt/images/github-enterprise-3.18.1.qcow2,format=qcow2,bus=virtio \
  --disk path=/var/lib/libvirt/images/github-enterprise-data.qcow2,format=qcow2,bus=virtio \
  --os-variant ubuntu22.04 \
  --import \
  --network network=default \
  --graphics none
```
**✅ Explanation:**
- `--memory 27648` → 27 GB RAM
- `--vcpus 4` → minimum recommended
- `--disk` ...-data.qcow2 → secondary storage
- `--import` → use downloaded .qcow2 directly
- `--network network=default` → default libvirt NAT network
- `--graphics none` → headless mode

#### Verify VM and Disks
```bash
sudo virsh list --all
sudo virsh domblklist github-enterprise
sudo virsh domifaddr github-enterprise
```
Expected output:
- `vda` → root disk
- `vdb` → secondary disk
- IP for web access (e.g., 192.168.122.xxx)

#### Connect to VM Console
```bash
sudo virsh console github-enterprise
```
- Escape: `Ctrl + ]`
- Inside VM, verify:

```bash
lsblk        # verify disks vda & vdb
free -h      # verify memory ≥ 27GB
```

#### Access GitHub Enterprise Web Interface
```cpp
https://<VM_IP>:8443
```
**OR**
```bash
curl https://<VM_IP>:8443
```
- Ignore SSL warnings temporarily (`self-signed`)
- Preflight checks should now pass ✅
