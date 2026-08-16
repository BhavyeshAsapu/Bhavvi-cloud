# Day 3 — SSH Setup

## Goal

Configure secure, key-based SSH access to the Raspberry Pi from both my Mac and Asus laptop, allowing the Pi to run headless without a monitor or keyboard.

## What I did

* Verified that SSH was enabled and running on the Raspberry Pi.
* Connected from my Mac to the Raspberry Pi using SSH.
* Initially used the Pi's IP address, then configured access using the hostname `bhavyesh.local`.
* Confirmed that the Pi's IP address can change because it is assigned through DHCP.
* Generated an ED25519 SSH key pair on my Mac.
* Added the Mac's public SSH key to the Raspberry Pi.
* Verified key-based SSH login from the Mac.
* Disabled SSH password authentication on the Raspberry Pi.
* Verified the SSH configuration before restarting the SSH service.
* Restarted the SSH service and confirmed that key-based login still worked.
* Generated/used an existing ED25519 SSH key on my Asus laptop.
* Added the Asus public SSH key to the Raspberry Pi's `authorized_keys`.
* Verified SSH access from the Asus laptop to the Raspberry Pi.
* Confirmed that both the Mac and Asus can access the Pi using SSH keys.
* Removed the need for a monitor and keyboard by switching the Pi to headless operation.

## SSH Setup

### Raspberry Pi

Hostname:

```text
bhavyesh.local
```

Username:

```text
bhavyesh_bhavvi
```

SSH configuration:

```text
PubkeyAuthentication yes
PasswordAuthentication no
```

SSH authorized keys:

```text
~/.ssh/authorized_keys
```

The Pi now accepts SSH connections using authorized public keys instead of account passwords.

## Mac → Raspberry Pi

SSH command:

```bash
ssh bhavyesh_bhavvi@bhavyesh.local
```

Mac SSH key:

```text
~/.ssh/id_ed25519
```

Mac public key:

```text
~/.ssh/id_ed25519.pub
```

Generated the key using:

```bash
ssh-keygen -t ed25519
```

Added the public key using:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub bhavyesh_bhavvi@bhavyesh.local
```

Verified that I could log in without entering the Pi account password.

## Asus → Raspberry Pi

Verified that OpenSSH was installed on Windows:

```powershell
ssh -V
```

Generated/used an ED25519 SSH key:

```powershell
ssh-keygen -t ed25519
```

Asus SSH key location:

```text
C:\Users\ASUS\.ssh\id_ed25519
```

Asus public key:

```text
C:\Users\ASUS\.ssh\id_ed25519.pub
```

Added the Asus public key to:

```text
~/.ssh/authorized_keys
```

Verified the connection:

```powershell
ssh bhavyesh_bhavvi@bhavyesh.local
```

The Asus can now access the Pi using SSH key authentication.

## SSH Security Configuration

Before disabling password authentication, verified the current configuration:

```bash
sudo sshd -T | grep -E 'passwordauthentication|pubkeyauthentication'
```

Initially:

```text
pubkeyauthentication yes
passwordauthentication yes
```

Found that password authentication was enabled by:

```text
/etc/ssh/sshd_config.d/50-cloud-init.conf
```

Changed:

```text
PasswordAuthentication yes
```

to:

```text
PasswordAuthentication no
```

Verified the configuration:

```bash
sudo sshd -t
```

Then checked the effective SSH configuration:

```bash
sudo sshd -T | grep -E 'passwordauthentication|pubkeyauthentication'
```

Final result:

```text
pubkeyauthentication yes
passwordauthentication no
```

Restarted SSH:

```bash
sudo systemctl restart ssh
```

Then opened a new SSH connection from the Mac and confirmed that key-based authentication still worked.

## Commands Learned

```bash
ssh
ssh-keygen
ssh-copy-id
sshd
sshd -t
sshd -T
systemctl
nano
chmod
hostname
hostname -I
```

### Useful SSH Commands

Connect to Pi:

```bash
ssh bhavyesh_bhavvi@bhavyesh.local
```

Generate an ED25519 key:

```bash
ssh-keygen -t ed25519
```

Copy public key to a server:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub bhavyesh_bhavvi@bhavyesh.local
```

Check SSH service:

```bash
sudo systemctl status ssh
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

Validate SSH configuration:

```bash
sudo sshd -t
```

View effective SSH configuration:

```bash
sudo sshd -T
```

Check SSH authentication settings:

```bash
sudo sshd -T | grep -E 'passwordauthentication|pubkeyauthentication'
```

View SSH authorized keys:

```bash
cat ~/.ssh/authorized_keys
```

Set secure SSH directory permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

## Important Files

### Mac

Private key:

```text
~/.ssh/id_ed25519
```

Public key:

```text
~/.ssh/id_ed25519.pub
```

Known hosts:

```text
~/.ssh/known_hosts
```

### Asus

Private key:

```text
C:\Users\ASUS\.ssh\id_ed25519
```

Public key:

```text
C:\Users\ASUS\.ssh\id_ed25519.pub
```

### Raspberry Pi

Authorized public keys:

```text
~/.ssh/authorized_keys
```

SSH configuration:

```text
/etc/ssh/sshd_config
```

Cloud-init SSH configuration:

```text
/etc/ssh/sshd_config.d/50-cloud-init.conf
```

## Final Setup

```text
                Raspberry Pi 5
                 Bhavyesh
              bhavyesh.local
                     │
          ~/.ssh/authorized_keys
                ┌────┴────┐
                │         │
             Mac key   Asus key
                │         │
                ▼         ▼
               Mac       Asus
```

Both devices can now securely connect to the Raspberry Pi using SSH keys.

Password-based SSH authentication is disabled.

The Raspberry Pi can now operate **headless**, so a monitor and keyboard are no longer required for normal administration.

## Result

**Day 3 completed successfully.** ✅

I configured secure SSH access from both my Mac and Asus laptop to the Raspberry Pi, switched SSH authentication to public-key authentication, disabled password authentication, and verified that both devices can connect successfully.

The Raspberry Pi is now ready to be managed remotely as an always-on headless server.
