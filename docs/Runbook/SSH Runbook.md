# Post-Install SSH & User Setup (Debian/Ubuntu VMs)

## 1. Install SSH server (if not present)
```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh

2. Create or configure main user

# Create user (skip if already exists)
sudo adduser ellie

# Add to sudo group
sudo usermod -aG sudo ellie

3. Set password

sudo passwd ellie

4. Enable password authentication (temporary)

(default cloud images often disable it)

sudo mkdir -p /etc/ssh/sshd_config.d
cat << 'EOF' | sudo tee /etc/ssh/sshd_config.d/99-password.conf
PasswordAuthentication yes
KbdInteractiveAuthentication yes
PermitRootLogin prohibit-password
EOF
sudo systemctl restart ssh

5. Add user to Docker group (for apps VM)

sudo usermod -aG docker ellie

(log out and back in for this to take effect)
6. (Optional) Install QEMU Guest Agent

(so Proxmox shows VM IP in Summary tab)

sudo apt install -y qemu-guest-agent
sudo systemctl enable --now qemu-guest-agent

Notes

    Password login is temporary.

    Once keys are set up, disable password auth in SSH:

PasswordAuthentication no

Root login remains disabled except via console.