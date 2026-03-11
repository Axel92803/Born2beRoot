# 🖥️ Born2beRoot

**42 London** · System Administration · Rank 01

> *"Nobody ever administered a server into production by clicking 'Next' sixteen times."*
> — Every greybeard sysadmin, at some point

---

## About this Project

Born2beRoot is the first serious encounter with system administration. The objective is to configure a **Debian** virtual machine from scratch under strict security constraints — no GUI, no training wheels. By the end of it you should be able to explain every running service, every open port, and every policy decision as confidently as you explain a pointer dereference.

This README doubles as an **evaluation preparation guide**. If you can walk through every section below without reaching for Google, you're ready.

**OS chosen:** Debian (latest stable)
**Bonus completed:** Partitioning only (LVM encrypted layout matching the bonus schema)

---

## Table of Contents

1. [Why Debian](#1-why-debian)
2. [Virtual Machine Fundamentals](#2-virtual-machine-fundamentals)
3. [Partitioning & LVM (Bonus Layout)](#3-partitioning--lvm-bonus-layout)
4. [sudo Configuration](#4-sudo-configuration)
5. [User & Group Management](#5-user--group-management)
6. [Password Policy](#6-password-policy)
7. [SSH Hardening](#7-ssh-hardening)
8. [UFW Firewall](#8-ufw-firewall)
9. [AppArmor](#9-apparmor)
10. [The monitoring.sh Script](#10-the-monitoringsh-script)
11. [Cron](#11-cron)
12. [Evaluation Quick-Reference](#12-evaluation-quick-reference)
13. [Submission](#13-submission)

---

## 1. Why Debian

The subject permits either **Debian** or **Rocky Linux**. I went with Debian. Here's the reasoning, and here's what your evaluator will want to hear:

**Debian** is a community-driven distribution. It has no corporate parent dictating its roadmap. It prioritises stability above all — packages in the `stable` branch have been through `unstable` and `testing` before release, which means they've been battle-tested. Its package manager ecosystem (`dpkg`, `apt`, `aptitude`) is one of the most mature in the Linux world.

**Rocky Linux** is a downstream rebuild of Red Hat Enterprise Linux (RHEL), born after CentOS shifted to CentOS Stream. It uses `dnf`/`rpm` for package management and **SELinux** for mandatory access control instead of AppArmor. Setting it up for this project is significantly more complex (and KDump configuration is explicitly waived in the subject).

**Evaluation question — apt vs aptitude:**
Both are front-ends to `dpkg`. `apt` is the modern CLI tool (clean output, progress bars, simpler syntax). `aptitude` is an older, higher-level tool with a TUI (ncurses interface) and smarter dependency resolution — it can suggest multiple solutions when a dependency conflict arises, whereas `apt` will typically just refuse. In practice, `apt` is what you'll use 99% of the time on a modern Debian system. The subject asks you to know the difference, not to prefer one.

---

## 2. Virtual Machine Fundamentals

A virtual machine is a software emulation of a physical computer. A **hypervisor** (in our case, VirtualBox or UTM) sits between the VM and the host hardware, allocating CPU cycles, memory, and I/O to each guest OS as though it were running on bare metal.

**Type 1 hypervisors** (ESXi, Xen, Hyper-V) run directly on hardware — these are what you find in data centres. **Type 2 hypervisors** (VirtualBox, VMware Workstation) run on top of a host OS — these are what we use on our workstations.

Why does this matter? Because your evaluator may ask you to explain *what* a VM is and *why* we use one. The short answer: isolation, reproducibility, and the ability to snapshot an entire operating system state. If you break something, you roll back. You can't do that with bare metal without considerably more effort.

---

## 3. Partitioning & LVM (Bonus Layout)

This is the **bonus partitioning scheme**. The subject provides a reference diagram; the layout below matches it.

### What is LVM?

**Logical Volume Management** is an abstraction layer between your physical disks and the file systems the OS sees. It introduces three concepts:

- **PV (Physical Volume):** The actual disk or partition (e.g., `/dev/sda5`).
- **VG (Volume Group):** A pool of storage created from one or more PVs. Think of it as a virtual disk.
- **LV (Logical Volume):** A carved-out chunk of a VG that you format and mount. Think of it as a virtual partition.

The power of LVM is flexibility. You can resize volumes, add disks to a VG on the fly, and create snapshots — none of which are possible (or at least easy) with traditional MBR/GPT partitions.

### What about encryption?

The subject mandates **at least 2 encrypted partitions using LVM**. This is achieved via **LUKS** (Linux Unified Key Setup), which encrypts the physical volume before LVM sees it. At boot, the system asks for the passphrase, decrypts the PV, and then LVM assembles the volume group and logical volumes as normal.

### Verifying your layout

```bash
lsblk
```

Your output should show the bonus structure: `sda` split into `sda1` (boot), `sda2` (extended), `sda5` (LUKS-encrypted PV), with logical volumes for `root`, `swap`, `home`, `var`, `srv`, `tmp`, and `var--log` carved out of the encrypted VG.

The key thing evaluators look for: **the LV names, mount points, and the fact that the volumes sit on top of an encrypted container**. If `lsblk` shows `crypt` in the type column, you're good.

### Why separate partitions?

This is a real-world best practice. Isolating `/var/log` prevents a log-flooding attack from filling your root filesystem. Separating `/home` means a full reinstall doesn't nuke user data. Putting `/tmp` on its own volume lets you mount it with `nosuid` and `noexec` flags. Each decision has a security or operational rationale.

---

## 4. sudo Configuration

`sudo` ("superuser do") lets permitted users execute commands as root — or as another user — without sharing the root password. It's the gatekeeper between unprivileged and privileged execution.

### Subject requirements and their rationale

All sudo configuration lives in `/etc/sudoers` (edited via `visudo`) or in drop-in files under `/etc/sudoers.d/`.

| Requirement | Implementation | Why |
|---|---|---|
| Max 3 auth attempts | `Defaults passwd_tries=3` | Limits brute-force attempts in an interactive session |
| Custom error message | `Defaults badpass_message="<your message>"` | Just a subject requirement — in production you'd keep defaults |
| Log inputs and outputs | `Defaults log_input, log_output` | Full audit trail of what was run and what it produced |
| Log directory | `Defaults iolog_dir="/var/log/sudo"` | Centralises sudo audit logs |
| TTY required | `Defaults requiretty` | Prevents sudo from being invoked by background daemons or scripts without a terminal — reduces attack surface |
| Restricted PATH | `Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"` | Prevents PATH injection attacks where a malicious binary in a user-controlled directory shadows a system command |

### Verification

```bash
sudo visudo                           # check the file (read-only if unsure)
ls -la /var/log/sudo/                 # confirm log directory exists
cat /var/log/sudo/*/log               # check sudo command history
sudo ls                               # run something, then re-check the log
```

---

## 5. User & Group Management

The subject requires:

- A user with **your 42 login** as username
- That user must belong to groups **user42** and **sudo**

### Key commands

```bash
# Check existing groups for your user
groups <username>
id <username>

# Verify group membership explicitly
getent group sudo
getent group user42

# During evaluation — create a new user
sudo adduser <evaluator_username>

# Create a new group and assign the user
sudo groupadd evaluating
sudo usermod -aG evaluating <evaluator_username>

# Verify
getent group evaluating
```

The `-aG` flag in `usermod` is critical: `-a` means *append* (don't replace existing groups), `-G` specifies the supplementary group. Forgetting `-a` will remove the user from all other supplementary groups — a classic mistake.

---

## 6. Password Policy

The password policy is split across two mechanisms: **login.defs** for age/expiry rules and **PAM** (specifically `pam_pwquality`) for complexity rules.

### Age policy — `/etc/login.defs`

```
PASS_MAX_DAYS   30      # Password expires every 30 days
PASS_MIN_DAYS   2       # Minimum 2 days between changes
PASS_WARN_AGE   7       # Warning 7 days before expiry
```

**Important:** Changing `login.defs` only affects *newly created* users. For existing users (including root and your login), apply retroactively with:

```bash
sudo chage -M 30 -m 2 -W 7 <username>
sudo chage -l <username>              # verify
```

### Complexity policy — `/etc/pam.d/common-password`

This is handled by `libpam-pwquality`. The relevant line:

```
password requisite pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

| Parameter | Meaning |
|---|---|
| `minlen=10` | Minimum 10 characters |
| `ucredit=-1` | At least 1 uppercase letter |
| `lcredit=-1` | At least 1 lowercase letter |
| `dcredit=-1` | At least 1 digit |
| `maxrepeat=3` | No more than 3 consecutive identical characters |
| `reject_username` | Password must not contain the username |
| `difok=7` | At least 7 characters must differ from the old password (does not apply to root) |
| `enforce_for_root` | Root must also comply |

After configuring these, **change all existing passwords** (root included) to comply with the new policy.

---

## 7. SSH Hardening

**SSH** (Secure Shell) provides encrypted remote access to the server. It replaces insecure protocols like Telnet and rlogin.

### Subject requirements

1. SSH runs on **port 4242** only (not the default 22)
2. **Root login via SSH is disabled**

### Configuration — `/etc/ssh/sshd_config`

```
Port 4242
PermitRootLogin no
```

After editing, restart the service:

```bash
sudo systemctl restart ssh
sudo systemctl status ssh              # confirm it's active on 4242
```

### Verification during evaluation

```bash
# From the HOST machine (not the VM):
ssh <username>@127.0.0.1 -p 4242      # should work

# Attempt root login — must be denied:
ssh root@127.0.0.1 -p 4242            # should fail
```

For this to work, you need to set up **port forwarding** in VirtualBox: host port 4242 → guest port 4242 (Settings → Network → Advanced → Port Forwarding).

### Why change the port?

Port 22 is the default SSH port. Every automated scanner on the internet hammers port 22 first. Changing it doesn't make SSH more secure cryptographically, but it massively reduces noise from brute-force bots. This is **security through obscurity** — not a primary defence, but a sensible layer. In a production environment, you'd combine this with key-based auth and fail2ban.

---

## 8. UFW Firewall

**UFW** (Uncomplicated Firewall) is a front-end for `iptables`/`nftables`. It simplifies rule management for common use cases.

### Subject requirements

- UFW must be active at boot
- Only port **4242** is open

### Key commands

```bash
sudo ufw status verbose                # check status and default policies
sudo ufw status numbered               # list rules with numbers (useful for deletion)

# During evaluation — add and remove a port:
sudo ufw allow 8080                    # open port 8080
sudo ufw status numbered               # confirm it's listed
sudo ufw delete <rule_number>          # remove it (do this for both v4 and v6 rules)
```

### What's happening under the hood?

UFW translates your simple `allow/deny` rules into `iptables` chains (or `nft` rules on newer kernels). When a packet arrives at the network interface, the kernel walks these chains and decides whether to ACCEPT, DROP, or REJECT the packet. The default policy for UFW is to deny incoming and allow outgoing — a sane baseline for any server.

---

## 9. AppArmor

**AppArmor** is a Linux Security Module (LSM) that provides **Mandatory Access Control (MAC)**. Unlike traditional UNIX permissions (Discretionary Access Control — the user decides who can access their files), MAC is enforced by the kernel based on security policies, regardless of what the user wants.

AppArmor works with **profiles** — each profile defines what a specific program is allowed to access (files, network, capabilities). Profiles can run in **enforce** mode (violations are blocked and logged) or **complain** mode (violations are logged but allowed).

### Verification

```bash
sudo aa-status                         # list loaded profiles and their modes
sudo systemctl status apparmor         # confirm it's running
```

AppArmor must be running at startup. On Debian, it is enabled by default. If your evaluator asks the difference between AppArmor and SELinux: AppArmor uses **path-based** rules (it cares about file paths), while SELinux uses **label-based** rules (every file, process, and port gets a security label). SELinux is more granular but significantly harder to configure — hence why Rocky is the harder choice for this project.

---

## 10. The monitoring.sh Script

The subject requires a bash script that broadcasts system information to all terminals every 10 minutes via `wall`.

### Script location

```bash
/usr/local/bin/monitoring.sh
```

### What it displays

| Metric | How it's gathered |
|---|---|
| Architecture & kernel | `uname -a` |
| Physical CPUs | `grep "physical id" /proc/cpuinfo \| sort \| uniq \| wc -l` |
| Virtual CPUs | `grep "^processor" /proc/cpuinfo \| wc -l` |
| RAM usage | `free -m` parsed with `awk` |
| Disk usage | `df -BM` / `df -BG` parsed with `awk` |
| CPU load | `top -bn1` parsed for user + system percentage |
| Last boot | `who -b` parsed with `awk` |
| LVM active | Check `lsblk` output for `lvm` type entries |
| TCP connections | `ss -neopt state established \| wc -l` |
| Logged users | `users \| wc -w` |
| IP & MAC | `hostname -I` and `ip link show` |
| Sudo commands | `journalctl _COMM=sudo \| grep COMMAND \| wc -l` |

### Key evaluation questions about the script

**"How do you stop the script without modifying it?"**

You stop the **cron job**, not the script itself:
```bash
sudo crontab -u root -e
# Comment out or delete the line that calls monitoring.sh
```

Alternatively, `sudo systemctl stop cron` will stop the cron daemon entirely, but that's a heavier hammer.

**"Change it to run every minute."**

```bash
sudo crontab -u root -e
# Change: */10 * * * * /usr/local/bin/monitoring.sh
# To:     */1  * * * * /usr/local/bin/monitoring.sh
```

---

## 11. Cron

**Cron** is a time-based job scheduler. The cron daemon (`crond`) reads **crontab** files and executes commands at specified intervals.

### Crontab syntax

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 7, 0 and 7 = Sunday)
│ │ │ │ │
* * * * * command_to_execute
```

`*/10 * * * *` means "every 10 minutes, every hour, every day." The `*/n` syntax means "every nth interval."

### Editing crontab

```bash
sudo crontab -u root -e               # edit root's crontab
sudo crontab -u root -l               # list root's crontab
```

---

## 12. Evaluation Quick-Reference

This is your pre-defence checklist. Run through these commands in order before your evaluator arrives.

### Preliminary checks

```bash
# Verify no graphical server is running
ls /usr/bin/*session                   # should NOT show a display manager
systemctl list-units --type=service | grep -i display

# Verify hostname
hostnamectl                            # should show <your_login>42

# Verify partitions match bonus layout
lsblk

# Verify OS
cat /etc/os-release                    # should show Debian
```

### Service checks

```bash
sudo systemctl status ssh              # active, port 4242
sudo ufw status verbose                # active, only 4242 open
sudo aa-status                         # AppArmor loaded with profiles in enforce mode
```

### User & group checks

```bash
id <your_login>                        # should show user42 and sudo groups
getent group sudo
getent group user42
```

### Password policy checks

```bash
sudo chage -l <your_login>            # verify age policy
sudo chage -l root                     # root too
cat /etc/pam.d/common-password         # verify pam_pwquality line
```

### Sudo checks

```bash
sudo visudo                            # review the rules (or cat /etc/sudoers.d/<file>)
ls /var/log/sudo/                      # log directory exists
sudo ls                                # run something, then verify it was logged
```

### Hostname modification (evaluator will ask)

```bash
sudo hostnamectl set-hostname <new_hostname>
sudo nano /etc/hosts                   # update the hostname here too
sudo reboot                            # confirm it persists
# Then restore original hostname the same way
```

### User creation (evaluator will ask)

```bash
sudo adduser <new_user>                # follow prompts, use a compliant password
sudo groupadd evaluating
sudo usermod -aG evaluating <new_user>
getent group evaluating                # verify
```

### UFW rule addition/removal (evaluator will ask)

```bash
sudo ufw allow 8080
sudo ufw status numbered
sudo ufw delete <rule_number>          # delete both IPv4 and IPv6 entries
sudo ufw status numbered               # confirm removal
```

### SSH connection test

```bash
# From host terminal:
ssh <your_login>@127.0.0.1 -p 4242    # should succeed
ssh root@127.0.0.1 -p 4242            # should be denied
```

### Monitoring script

```bash
cat /usr/local/bin/monitoring.sh       # be ready to explain every line
sudo crontab -u root -e               # show the cron job, change interval if asked
```

---

## 13. Submission

The only file submitted to the Git repository is `signature.txt`, containing the SHA-1 hash of the `.vdi` (or `.qcow2`) virtual disk image.

```bash
# On the HOST machine, navigate to the VM storage directory:
# Linux:   ~/VirtualBox VMs/
# macOS:   ~/VirtualBox VMs/  (or ~/Library/Containers/com.utmapp.UTM/Data/Documents/)
# Windows: %HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\

sha1sum <your_vm_name>.vdi             # Linux
shasum <your_vm_name>.vdi              # macOS
```

Paste the resulting hash into `signature.txt` at the root of your repo.

**Critical warning:** Any change to the VM after generating the signature will alter the hash. Either duplicate the VM before your evaluation or use VirtualBox's **snapshot/save state** feature so you can restore the exact state that matches your submitted signature.

---

## Final Notes

Born2beRoot isn't really about memorising commands. It's about understanding *why* each configuration exists. Every `Defaults` line in your sudoers file, every `pam_pwquality` parameter, every firewall rule — they all map to a real threat model. Your evaluator isn't checking whether you can type `sudo ufw status`. They're checking whether you understand what happens when you do, and what would happen if you hadn't configured it.

Know your system. Defend your choices.

---

*Project by [itanvuia](https://github.com/Axel92803) · 42 London · 2026*