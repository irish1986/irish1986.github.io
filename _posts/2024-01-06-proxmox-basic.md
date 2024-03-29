---
layout: post
title: "Proxmox VE 8.1 Quick Start Guide"
author: irish1986
date: 2024-01-06 00:00:00 +0000
categories:
  - proxmox
  - tutorial
tags:
  - core system
  - proxmox
  - proxmox 8
  - setup
  - tutorial
---

Proxmox VE 8.1 was release [late in November of 2023](https://www.proxmox.com/en/about/press-releases/proxmox-virtual-environment-8-1).  This iteration is based on [Debian Bookworm](https://www.debian.org/News/2023/20230610) bringing a slew a new features like defaulting to Linux Kernel 6.5.

This guide is a complete step by step bare metal installation of Proxmox and configuration as per my homelab.  

## Why Proxmox VE for Homelab?

### Proxmox Consideration and Recommendations

## Get started

### Creating Proxmox USB Boot Media

### Installing Proxmox VE 8.1

1. Accept EULA
2. Select disk (ext4) for single drive
3. Set local
    * Country : Canada
    * Time zone : America/Toronto
    * Keyboard : US English
4. Set credentials
    * Country : str82THE.str82THE.
    * Email : <simon.harvey.86@gmail.com>
5. Setup network
    * FQDN : pve-x.lan
    * IP : 192.168.10.2x/24
    * Netmask: 255.255.255.0
    * Gateway : 192.168.10.1
    * DNS : 192.168.10.1

## Proxmox Post-Install Configuration

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```

### Update (Optional)

### AMD/Intel Microcode Update (Optional)

AMD and Intel releases new microcode for their processors from time to time. This is different from BIOS firmware, as the microcode runs inside the processor. It can fix CPU bugs or make other changes, as needed. You can use the following tteck script to download the latest AMD/Intel microcode and install it. A full system reboot will be needed for the microcode to take effect.

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/microcode.sh)"
```

After the Proxmox host reboots you can run the following command to see if any microcode update is active. Not all CPUs need microcode updates, so you may well not see anything listed.

```bash
journalctl -k | grep -E "microcode" | head -n 1
```

### Optimize CPU Power (Optional)

## Security

### Users & Groups

### Token

### Proxmox Let’s Encrypt SSL Cert (Optional)

### Proxmox Two Factor Setup (Optional)

## Network

### Realtek R8125B (Optional)

```bash
cd /tmp
wget <https://github.com/awesometic/realtek-r8125-dkms/releases/download/9.012.03-2/realtek-r8125-dkms_9.012.03-2_amd64.deb>
sudo dpkg -i realtek-r8125-dkms*.deb
sudo apt install --fix-broken

lsmod | grep -i r8169
lspci -k

sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && sudo apt autoclean -y
```

### VLAN Enable Proxmox (Optional)

## Storage

### Local vs Local-LVM

1. Navigate to "datacenter -> storage"
2. Delete the "local-lvm" storage
3. SSH into your server and run the follwing commands

```shell
lvremove /dev/pve/data
lvresize -l +100%FREE /dev/pve/root
resize2fs /dev/mapper/pve-root
```

### NFS Shares

### CEPH Storage

1. Wipe disk
    ceph-volume lvm zap /dev/nvme0n1 --destroy

## Device Passthrough (Optional)

### Intel iGPU (Optional)

```bash
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"

update-grub

nano /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
update-initramfs -u -k all

reboot
```

### nvidia GPU (Optional)

### AMD GPU (Optional)

### Google Coral TPU (Optional)

A very popular NVR solution that integrates well with Home Assistant is Frigate. It provides fantastic object and person detection from your camera streams. The Frigate project is a container, so it’s easy to deploy.

Special hardware can be used in order to improve the image detection and general performance.  A Google Coral Tensor Processing Unit (TPU) is a very common fairly affordable option.  This device is availabe for purchase in a few configuration sch as USB, PCI-E and m.2 (A+E Key) PCI-E.  Two of my Proxmox host have m.2 (A+E Key) Google Coral TPU installed so here how to enable this device passthrough.

First on the Proxmox host, we need to modify the configuration of the Proxmox host where the Coral TPU is installed.  Login to Proxmox and open a shell.

1. Modify the GRUB configuration by running the following command:

```bash
nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_aspm=off initcall_blacklist=sysfb_init"
```

```bash
nano /etc/default/grub
update-grub
```

```bash
nano /etc/modules
vfio
vfio_iommu_type1
vfio_pci

kvmgt
xengt
vfio-mdev
```

```bash
update-initramfs -u -k all
```

```bash
nano /etc/modprobe.d/blacklist-apex.conf
blacklist gasket
blacklist apex
options vfio-pci ids=1ac1:089a
```

```bash
update-initramfs -u -k all
lsmod | grep apex
reboot
```

### PCI-E Device (Optional)

I don't have any other devices at the moment to passthrought from the host but soon I shall have a HBA adapter.  I am not certain if I'll run TrueNAS bare metal or under Proxmox.

## Clustering

1. Log in to the first Proxmox server, select Datacenter, then Cluster, and select Create Cluster.
2. Give the cluster a name, then select create. The cluster will then be created and you’ll be able to join it from other Proxmox instances.
3. We will create three total rules for UDP ports 5404, 5405, and TCP port 22. (optional)
4. On the device you just set up the cluster with (pve-test in my example), select Join Information under Cluster.
5. The join information will be displayed. You will need both, the Fingerprint and Join Information to join the cluster. Select Copy Information, then open your second Proxmox node.
6. On the second Proxmox node, select Datacenter, Cluster, and Join Cluster.
7. Create external quorum devices

pvecm create pve-cluster
pvecm status
pvecm join <ip-address-of-the-main-node>

### High Availability

1. Create HA Groups
2. Create Disable HA

### Create Pools

1. Create HA Groups
2. Create Disable HA

### Wake On Lan

1. Create HA Groups
2. Create Disable HA

## Backup

### Promox Backup Server

1. Create HA Groups
2. Create Disable HA

## Monitoring

### Metric Server

### Notifications (Optional)

### SMART Monitoring (Optional)

## Workload

### Terraform

1. Create Terraform user
2. Create API Tokens
3. Create template VM

### Admin

1. Create Terraform user

### K3S

1. Create Terraform user

### Home Assistance (HAOS)

1. Create Terraform user
