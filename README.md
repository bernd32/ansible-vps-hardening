# Debian VPS Simple & Basic Initial Hardening Playbook

This Ansible playbook is designed to take a freshly rented, insecure Debian/Ubuntu VPS (accessible only via `root` with a password) and transform it into a secure, hardened environment in a single run.

## What this playbook does:
1. **System Updates:** Updates all system packages to their latest versions.
2. **Installs Dependencies:** Installs `sudo`, `ufw` (Firewall), `fail2ban` (Intrusion Prevention), and `unattended-upgrades`.
3. **User Management:** Creates a secure, non-root user and grants them passwordless `sudo` access (ideal for future Ansible automation).
4. **SSH Keys:** Injects your local public SSH key into the new user's `authorized_keys`.
5. **Firewall:** Enables UFW, blocking all incoming traffic except SSH.
6. **Intrusion Prevention:** Enables Fail2Ban to automatically temporarily ban IPs that attempt to brute-force SSH.
7. **Locks the Doors (SSH Hardening):** Disables `root` login and completely disables password-based authentication.

---

## ⚠️ Important Nuances & Safety Features

* **Anti-Lockout Validation:** The playbook uses `validate: /usr/sbin/sshd -t -f %s` before applying changes to the SSH daemon. If the playbook makes a syntax error, SSH will refuse the change rather than crashing and locking you out. Or make a snapshot of your system before running this playbook. 
* **Sudoers Best Practices:** Instead of modifying the main `/etc/sudoers` file (which can be risky and overwrite OS defaults), this playbook safely places a dedicated file in `/etc/sudoers.d/`.

---

## Requirements

**On your Local Machine:**
* [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) installed.
* `sshpass` installed (required for Ansible to log in with a password the very first time). 
  * *Ubuntu/Debian:* `sudo apt install sshpass`
  * *macOS:* `brew install sshpass`
* An SSH Keypair. If you don't have one, generate it:
  `ssh-keygen -t ed25519 -C "vps-admin"`

**On your Remote VPS:**
* A fresh Debian or Ubuntu installation.
* You must know the server's IP address and the `root` password.

---

## How to use this Playbook

### Step 1: Configure Variables
1. Edit `inventory.ini` and replace `192.168.88.70` with your actual VPS IP address.
2. Edit `vars.yml` and adjust the variables to your liking:
   * `new_user`: The name of the user you want to create (e.g., `adminuser`).
   * `ssh_public_key_path`: The path to your **local** public key (e.g., `~/.ssh/id_ed25519.pub`).

### Step 2: Establish a "Lifeline" (CRITICAL SAFETY STEP)
Because this playbook disables password login, if your SSH key is invalid, you could lock yourself out. 
Before running the playbook, open a terminal window and SSH into your VPS:
```bash
ssh root@<YOUR_VPS_IP>
