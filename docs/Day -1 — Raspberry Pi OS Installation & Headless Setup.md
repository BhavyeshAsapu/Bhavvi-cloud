# Day -1 — Raspberry Pi OS Installation & Headless Setup

## Goal

Install **Raspberry Pi OS Lite (64-bit)** on my Raspberry Pi 5 using my portable SSD and configure it for **headless operation** without an external monitor or keyboard.

## Hardware Used

* Raspberry Pi 5
* Portable 512 GB SSD
* Windows laptop
* Laptop's mobile hotspot
* USB connection for the SSD

## What I Did

### 1. Prepared the Portable SSD

* My portable SSD initially had two partitions:

  * EFI partition
  * Data partition
* Confirmed that Raspberry Pi Imager can overwrite the existing partition layout.
* Did **not** manually merge the EFI and Data partitions.
* Used the entire SSD as the Raspberry Pi OS storage device.

> **Important:** Writing Raspberry Pi OS to the SSD overwrites the existing partitions and data on the selected drive.

### 2. Installed Raspberry Pi OS Lite

Used **Raspberry Pi Imager** to install:

```text
Raspberry Pi OS Lite (64-bit)
```

Target device:

```text
Raspberry Pi 5
```

Storage:

```text
Portable SSD
```

### 3. Configured a Headless Installation

Since I don't currently have an external monitor for the Pi, I configured the OS before writing it to the SSD.

Configured:

```text
Username: bhavyesh_bhavvi
```

Configured a password for the Linux user.

Configured Wi-Fi:

```text
SSID: Raspberry BRO
Band: 2.4 GHz
```

The `Raspberry BRO` network is my Windows laptop's mobile hotspot.

### 4. Booted the Raspberry Pi

Connected the SSD to the Raspberry Pi 5 and booted it without a monitor.

The Pi successfully connected to my laptop's hotspot.

Windows showed the connected device with an IP address such as:

```text
192.168.137.237
```

### 5. Tested Network Connectivity

Initially, the Pi appeared in the Windows hotspot connected-device list.

Tested connectivity using:

```powershell
ping 192.168.137.237
```

Also tested whether SSH was reachable:

```powershell
Test-NetConnection 192.168.137.237 -Port 22
```

At one point the Pi disconnected from the hotspot, so I power-cycled the Pi and waited for it to reconnect.

### 6. Tested SSH

Attempted SSH access:

```powershell
ssh pi@192.168.137.237
```

This confirmed that the SSH service was reachable, but the `pi` username was incorrect.

Checked the Raspberry Pi Imager configuration and found that my actual username was:

```text
bhavyesh_bhavvi
```

Therefore the correct SSH command is:

```powershell
ssh bhavyesh_bhavvi@192.168.137.237
```

### 7. Fixed Raspberry Pi Imager Credentials

Discovered that Raspberry Pi Imager had previously saved password values.

To avoid authentication problems, I reconfigured the installation with explicit:

* Username
* Password
* Password confirmation
* Wi-Fi SSID
* Wi-Fi password

I then re-imaged the SSD with the corrected configuration.

## Commands Learned

### SSH

```bash
ssh username@ip-address
```

Example:

```bash
ssh bhavyesh_bhavvi@192.168.137.237
```

### Ping

Used to test whether the Raspberry Pi is reachable over the network:

```powershell
ping 192.168.137.237
```

### Test SSH Port

Used to check whether port 22 is reachable:

```powershell
Test-NetConnection 192.168.137.237 -Port 22
```

### ARP

Used to inspect devices known to the local network:

```powershell
arp -a
```

## Problems Encountered

### Problem 1 — Pi disappeared from hotspot

The Pi initially connected to the Windows hotspot but later disconnected.

**Solution:**

* Kept the hotspot enabled
* Power-cycled the Raspberry Pi
* Waited for it to reconnect
* Checked the connected-device list again

### Problem 2 — SSH password rejected

Initially tried:

```bash
ssh pi@192.168.137.237
```

The password was rejected.

The actual configured username was discovered to be:

```text
bhavyesh_bhavvi
```

### Problem 3 — Raspberry Pi Imager saved password

The Imager showed a previously saved password value, which caused confusion during configuration.

**Solution:**

Explicitly re-entered the desired password and confirmation before re-imaging.

## Important Lessons

* Raspberry Pi OS Lite is suitable for a **headless Raspberry Pi server**.
* A monitor is not required if SSH and networking are configured during imaging.
* Raspberry Pi Imager can configure the username, password, Wi-Fi and SSH before the first boot.
* The Linux username is different from the Wi-Fi SSID.
* The Raspberry Pi's IP address can change when using a hotspot.
* Windows hotspot can show connected devices even when they temporarily become unreachable.
* SSH uses **TCP port 22** by default.
* A successful SSH connection reaching the password prompt means the network connection and SSH service are working.
* When typing an SSH password, characters are intentionally not displayed.
* Always verify the target drive before writing an OS image because existing partitions and data can be overwritten.

## Current Setup

```text
Raspberry Pi: Raspberry Pi 5
OS: Raspberry Pi OS Lite (64-bit)
Storage: Portable 512 GB SSD
Network: Windows Mobile Hotspot
Wi-Fi Band: 2.4 GHz
Username: bhavyesh_bhavvi
Access Method: SSH
Monitor: Not required
```

## Next Day

### Day 0 — First Boot & Linux Setup

Planned tasks:

* Verify Raspberry Pi OS version
* Verify 64-bit architecture
* Check CPU and RAM
* Check SSD and partitions
* Update the system
* Configure hostname
* Configure SSH properly
* Generate/use SSH keys
* Learn basic Linux server commands
* Check network configuration
* Prepare the Pi for always-on server use
