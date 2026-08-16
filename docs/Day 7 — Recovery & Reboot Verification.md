# Day 7 — Recovery & Reboot Verification

## Goal

Verify that the Raspberry Pi can reboot and return to a reliable server state without manual fixing.

## Phase 1 Requirement

> Phase 1 is complete when the Pi can reboot and come back as a reliable server without manual fixing.

## What I did

* Recorded the server state before reboot
* Verified the Raspberry Pi hostname and IP address
* Verified the SSD and filesystem layout
* Verified the `/srv` server directory structure
* Verified UFW was active before reboot
* Rebooted the Raspberry Pi
* Reconnected through SSH after reboot
* Verified the network after reboot
* Verified the SSD/root filesystem after reboot
* Verified `/boot/firmware` after reboot
* Verified the `/srv` directory structure after reboot
* Verified UFW automatically started after reboot
* Checked for failed systemd services
* Confirmed there were no failed services
* Confirmed the server recovered without manual fixing

## Pre-Reboot State

### Hostname

```bash
hostname
```

Result:

```text
Bhavyesh
```

### IP Address

```bash
hostname -I
```

Result:

```text
10.46.2.72
```

### SSD Layout

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINTS
```

The Raspberry Pi boots from the SSD:

```text
sda
├── sda1  512M   vfat   bootfs   /boot/firmware
└── sda2  465.3G ext4   rootfs   /
```

The SSD is approximately 465.8 GB.

### Root Filesystem

```text
/dev/sda2 → /
```

### Boot Filesystem

```text
/dev/sda1 → /boot/firmware
```

## Server Directory Verification

Before reboot:

```bash
ls -la /srv
```

Result:

```text
/srv/
├── apps/
├── backups/
├── config/
├── data/
├── docker/
└── logs/
```

## Firewall Before Reboot

```bash
sudo ufw status
```

Result:

```text
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

## Reboot Test

Rebooted the Raspberry Pi using:

```bash
sudo reboot
```

The existing SSH session disconnected as expected.

Waited for the Raspberry Pi to boot and reconnect to the network.

## SSH Recovery

From the Mac, connected using:

```bash
ssh bhavyesh_bhavvi@bhavyesh.local
```

SSH connection succeeded.

This confirmed that:

* Avahi/mDNS was working
* `bhavyesh.local` resolved correctly
* SSH started automatically
* SSH port 22 was accessible
* Public-key authentication continued to work
* The non-root administration workflow continued to work

## Network Verification

After reboot:

```bash
hostname -I
```

Result:

```text
10.46.2.72
```

The Pi recovered the same IP address.

```text
Network recovery: SUCCESS
```

## SSD Verification

After reboot:

```bash
lsblk -f
```

Result:

```text
sda
├── sda1  vfat  bootfs  /boot/firmware
└── sda2  ext4  rootfs  /
```

The root filesystem was correctly mounted:

```text
/dev/sda2 → /
```

The boot filesystem was correctly mounted:

```text
/dev/sda1 → /boot/firmware
```

## Disk Usage Verification

```bash
df -h
```

Important mounts:

```text
/dev/sda2  → /
/dev/sda1  → /boot/firmware
```

The SSD was available normally after reboot.

## `/srv` Verification

After reboot:

```bash
ls -la /srv
```

The complete server directory structure was still present:

```text
/srv/
├── apps/
├── backups/
├── config/
├── data/
├── docker/
└── logs/
```

## Firewall Recovery

After reboot:

```bash
sudo ufw status
```

Result:

```text
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

This confirmed that UFW automatically started after reboot.

## Systemd Service Recovery

Checked for failed services:

```bash
systemctl --failed
```

Result:

```text
0 loaded units listed.
```

No systemd services were in a failed state.

## Recovery Verification

| Component               | Result       |
| ----------------------- | ------------ |
| Raspberry Pi reboot     | ✅ Successful |
| SSH after reboot        | ✅ Working    |
| `.local` hostname       | ✅ Working    |
| Network                 | ✅ Working    |
| IP address              | ✅ 10.46.2.72 |
| SSD boot                | ✅ Working    |
| Root filesystem         | ✅ Mounted    |
| Boot filesystem         | ✅ Mounted    |
| `/srv` structure        | ✅ Present    |
| UFW                     | ✅ Active     |
| SSH firewall rule       | ✅ Present    |
| Systemd failed services | ✅ None       |

## Recovery Procedure

The basic recovery procedure for the Raspberry Pi is:

```text
1. Power on or reboot the Raspberry Pi
2. Wait for the operating system to boot
3. Wait for the network to initialize
4. Connect using SSH:
   ssh bhavyesh_bhavvi@bhavyesh.local
5. Verify the filesystem:
   lsblk -f
6. Verify disk mounts:
   df -h
7. Verify server directories:
   ls -la /srv
8. Verify firewall:
   sudo ufw status
9. Check failed services:
   systemctl --failed
```

## Result

The Raspberry Pi successfully rebooted and recovered without manual intervention.

After reboot:

* SSH was available
* `.local` hostname resolution worked
* Network connectivity was restored
* The SSD booted correctly
* `/` was mounted correctly
* `/boot/firmware` was mounted correctly
* `/srv` directories were intact
* UFW was active
* SSH remained allowed through the firewall
* No systemd services were failed

## Phase 1 Status

**Phase 1 — Complete ✅**

The Raspberry Pi has demonstrated that it can reboot and return to a reliable server state without manual fixing.
