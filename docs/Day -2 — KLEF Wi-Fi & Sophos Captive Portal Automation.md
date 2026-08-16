# Day -2 — KLEF Wi-Fi & Sophos Captive Portal Automation

## Goal

Configure the Raspberry Pi 5 to connect directly to the **KLEF hostel Wi-Fi** without using my laptop hotspot and automatically authenticate through the **Sophos captive portal** after boot.

## What I did

- Scanned for available Wi-Fi networks using NetworkManager
- Identified **KLEF** as the hostel Wi-Fi
- Confirmed that KLEF is an open Wi-Fi network
- Created a persistent KLEF Wi-Fi connection using NetworkManager
- Enabled automatic Wi-Fi connection at boot
- Configured high autoconnect priority for KLEF
- Stored Sophos credentials securely in `/etc/default/kl-sophos`
- Created a systemd service for automatic Sophos authentication
- Enabled SSH to start automatically at boot
- Installed and enabled Avahi for `.local` hostname access
- Verified that `Bhavyesh.local` could be accessed from Windows
- Switched the Pi from the laptop hotspot to KLEF Wi-Fi
- Confirmed the Pi received a KLEF network IP address
- Rebooted the Pi without a monitor or keyboard
- Confirmed the Pi automatically reconnected to KLEF after reboot
- Debugged the Sophos captive portal login
- Inspected the Sophos portal JavaScript to find the actual login request
- Updated the login request to use the correct Sophos `login.xml` endpoint
- Added the required Sophos login parameters:
  - `mode=191`
  - `username`
  - `password`
  - `a` timestamp
  - `producttype=0`
- Restarted the Sophos systemd service
- Verified that automatic Sophos authentication succeeded
- Verified unrestricted Internet access after authentication
- Confirmed that `apt update` can work once the captive portal is authenticated

## Final Setup

```text
KLEF Hostel Wi-Fi
        │
        ▼
Raspberry Pi 5
        │
        ├── NetworkManager
        │       └── Automatic KLEF connection
        │
        ├── systemd
        │       └── kl-sophos.service
        │
        ├── Sophos Captive Portal
        │       └── Automatic authentication
        │
        ├── SSH
        │       └── Remote access
        │
        └── Avahi
                └── Bhavyesh.local

## Commands Learned

```bash
nmcli device wifi list
nmcli device wifi rescan
nmcli connection show
nmcli connection add
nmcli connection modify
nmcli connection up

systemctl enable
systemctl start
systemctl restart
systemctl status
systemctl is-enabled
systemctl daemon-reload

journalctl
curl
grep
hostname
hostname -I

** Important Configuration **

KLEF Wi-Fi
    SSID: KLEF
    Security: Open
    Autoconnect: Enabled
    Autoconnect Priority: 100

## Sophos Service

Service: kl-sophos.service
Script: /opt/kl-sophos/sophos-login.sh
Credentials: /etc/default/kl-sophos

## SSH / Hostname

Hostname: Bhavyesh
mDNS hostname: Bhavyesh.local

** Verification **

    After reboot, the Raspberry Pi successfully:

        Connected to KLEF automatically
        Obtained an IP address
        Started the Sophos service
        Detected the captive portal
        Submitted the credentials
        Successfully authenticated
        Obtained unrestricted Internet access
        Remained accessible through SSH

    Internet authentication was verified using:

        curl -sS -L --max-redirs 8 --max-time 10 \
        -o /dev/null \
        -w '%{http_code} %{url_effective}\n' \
        http://clients3.google.com/generate_204

    Successful result:

        204 http://clients3.google.com/generate_204

## Key Lesson

A captive portal is separate from Wi-Fi authentication.

The KLEF Wi-Fi itself is open, so NetworkManager only needs to connect to the SSID. Internet access is then controlled by the Sophos captive portal, which requires a separate login.

Using systemd allows the Sophos authentication script to run automatically after the network becomes available, making the Raspberry Pi fully headless and independent of the laptop hotspot.