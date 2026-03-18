*This project has been created as part of the 42 curriculum by itanvuia*

# 🖥️ Born2beRoot

> **System Administration**
> 
> *"Nobody ever administered a server into production by clicking 'Next' sixteen times."*
> 
> — Every greybeard sysadmin, at some point

![42 School](https://img.shields.io/badge/42-000000?style=for-the-badge&logo=42&logoColor=white)
![Grade](https://img.shields.io/badge/Grade-100%2F100-success?style=for-the-badge)
[![Debian](https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white)](https://www.debian.org/)
[![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)](https://www.virtualbox.org/)
[![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)](https://www.linux.org/)
[![Shell](https://img.shields.io/badge/Shell_Script-737373?style=for-the-badge&logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/)
---

## 📖 About this Project

Born2beRoot is the first serious encounter with system administration. The objective is to configure a **Debian** virtual machine from scratch under strict security constraints, no GUI, no training wheels. By the end of it you should be able to explain every running service, every open port, and every policy decision as confidently as you explain a pointer dereference.

This README doubles as an **evaluation preparation guide**. If you can walk through every section below without reaching for Google, you're ready.

**OS chosen:** Debian (latest stable)
**Bonus completed:** Partitioning only (LVM encrypted layout matching the bonus schema)

---

## 📑 Table of Contents  
  
- [🤔 Why Debian](#-why-debian-debian-vs-rocky-linux)  
- [💻 Virtual Machine Fundamentals](#-virtual-machine-fundamentals-virtualbox-vs-utm)  
- [💾 Partitioning & LVM (Bonus Layout)](#-partitioning--lvm-bonus-layout)  
- [🔐 sudo Configuration](#-sudo-configuration)  
- [👥 User & Group Management](#-user--group-management)  
- [🔑 Password Policy](#-password-policy)  
- [🌐 SSH Hardening](#-ssh-hardening)  
- [🧱 UFW Firewall](#-ufw-firewall)  
- [🛡️ AppArmor](#️-apparmor)  
- [📜 The monitoring.sh Script](#-the-monitoringsh-script)  
- [⏰ Cron](#-cron)  
- [✅ Evaluation Quick-Reference](#-evaluation-quick-reference)  
- [📦 Submission](#-submission)  
  
---

## 📖 Description

Born2beRoot is a **system administration** project from the 42 curriculum. The objective is to configure a **Debian** virtual machine from scratch under strict security constraints, no GUI, no training wheels. By the end of it you should be able to explain every running service, every open port, and every policy decision as confidently as you explain a pointer dereference.

This README doubles as an **evaluation preparation guide**. If you can walk through every section below without reaching for Google, you're ready.

**OS chosen:** Debian (latest stable)
**Hypervisor:** VirtualBox
**Bonus completed:** Partitioning only (LVM encrypted layout matching the bonus schema)

### 🏗️ Main design choices

- **Partitioning:** Bonus-level encrypted LVM layout with separate logical volumes for `/`, `/home`, `/var`, `/srv`, `/tmp`, `/var/log`, and `swap` — each isolated to limit blast radius of disk-filling attacks and to enable independent resizing
- **Security policies:** PAM-based password complexity via `pam_pwquality`, 30-day expiry with 2-day minimum change interval, sudo limited to 3 attempts with full I/O logging
- **User management:** Non-root user in `user42` and `sudo` groups, root SSH login disabled
- **Services:** SSH on port 4242, UFW firewall allowing only port 4242, AppArmor in enforce mode

The four comparisons required by the subject are covered in their respective sections below: Debian vs Rocky Linux, VirtualBox vs UTM, UFW vs firewalld, and AppArmor vs SELinux.

---

## 🔧 Instructions

### Prerequisites

- **VirtualBox** (or UTM if VirtualBox is unavailable) installed on your host machine
- A **Debian ISO** — latest stable release, downloaded from [debian.org](https://www.debian.org/distrib/)

### Installation

1. Create a new VM in VirtualBox (no GUI server — X.org, Wayland, etc. are forbidden)
2. Allocate at least 1 CPU, 1 GB RAM, and a VDI disk sized to accommodate the bonus partition layout
3. Boot from the Debian ISO and follow the guided installer, selecting **encrypted LVM** partitioning when prompted
4. After installation, configure all mandatory services (SSH, UFW, AppArmor, sudo, password policy, cron) as documented in the sections below

### Running the VM

```bash
# Start the VM from VirtualBox GUI or headless:
VBoxManage startvm "Born2beRoot" --type headless

# Connect via SSH from the host:
ssh <username>@127.0.0.1 -p 4242
```

Port forwarding must be configured in VirtualBox: host port 4242 → guest port 4242 (Settings → Network → Advanced → Port Forwarding).

### Verifying the setup

```bash
# OS and kernel
cat /etc/os-release

# Partitions
lsblk

# Services
sudo systemctl status ssh
sudo ufw status verbose
sudo aa-status
```

---

## 📚 Resources

### References

- [Debian Administrator's Handbook](https://www.debian.org/doc/manuals/debian-handbook/) — comprehensive guide to Debian system administration
- [ArchWiki — LVM](https://wiki.archlinux.org/title/LVM) — one of the best LVM references regardless of distro
- [ArchWiki — dm-crypt / LUKS](https://wiki.archlinux.org/title/Dm-crypt) — in-depth encryption documentation
- [Ubuntu Server Guide — UFW](https://ubuntu.com/server/docs/firewalls) — practical UFW usage and examples
- [Debian Wiki — AppArmor](https://wiki.debian.org/AppArmor) — Debian-specific AppArmor setup and profile management
- [man sudoers](https://www.sudo.ws/docs/man/sudoers.man/) — the authoritative reference for sudo configuration
- [CronGuru](https://crontab.guru/) — interactive crontab expression editor

### 🤖 AI usage

`TODO: Fill in how AI was used (or not) for this project.`

---

## 🤔 Why Debian — Debian vs Rocky Linux

The subject permits either **Debian** or **Rocky Linux**. I went with Debian. Here's the reasoning, and here's what your evaluator will want to hear:

**Debian** is a community-driven distribution. It has no corporate parent dictating its roadmap. It prioritises stability above all, packages in the `stable` branch have been through `unstable` and `testing` before release, which means they've been battle-tested. Its package manager ecosystem (`dpkg`, `apt`, `aptitude`) is one of the most mature in the Linux world.

**Rocky Linux** is a downstream rebuild of Red Hat Enterprise Linux (RHEL), born after CentOS shifted to CentOS Stream. It uses `dnf`/`rpm` for package management and **SELinux** for mandatory access control instead of AppArmor. Setting it up for this project is significantly more complex (and KDump configuration is explicitly waived in the subject).

### ⚖️ Debian vs Rocky Linux

| | **Debian** | **Rocky Linux** |
|---|---|---|
| **Governance** | Community-driven (Debian Project) | Community enterprise rebuild of RHEL |
| **Package manager** | `dpkg` / `apt` / `aptitude` | `rpm` / `dnf` |
| **MAC system** | AppArmor (path-based) | SELinux (label-based) |
| **Firewall** | UFW (iptables front-end) | firewalld (zone-based) |
| **Release model** | Stable / Testing / Unstable tiers | Point releases tracking RHEL |
| **Target audience** | General-purpose, universally popular | Enterprise / production servers |
| **Complexity for B2bR** | Lower — recommended by the subject | Higher — SELinux & firewalld config |

**Evaluation question — apt vs aptitude:**
Both are front-ends to `dpkg`. `apt` is the modern CLI tool (clean output, progress bars, simpler syntax). `aptitude` is an older, higher-level tool with a TUI (ncurses interface) and smarter dependency resolution. It can suggest multiple solutions when a dependency conflict arises, whereas `apt` will typically just refuse. In practice, `apt` is what you'll use 99% of the time on a modern Debian system. The subject asks you to know the difference, not to prefer one.

---

## 💻 Virtual Machine Fundamentals — VirtualBox vs UTM

A virtual machine is a software emulation of a physical computer. A **hypervisor** (in our case, VirtualBox or UTM) sits between the VM and the host hardware, allocating CPU cycles, memory, and I/O to each guest OS as though it were running on bare metal.

**Type 1 hypervisors** (ESXi, Xen, Hyper-V) run directly on hardware; these are what you find in data centres. **Type 2 hypervisors** (VirtualBox, VMware Workstation) run on top of a host OS; these are what we use on our workstations.

Why does this matter? Because your evaluator may ask you to explain *what* a VM is and *why* we use one. The short answer: isolation, reproducibility, and the ability to snapshot an entire operating system state. If you break something, you roll back. You can't do that with bare metal without considerably more effort.

### ⚖️ VirtualBox vs UTM

| | **VirtualBox** | **UTM** |
|---|---|---|
| **Platform** | Cross-platform (Windows, macOS Intel, Linux) | macOS only (native Apple Silicon support) |
| **Hypervisor type** | Type 2 (runs on host OS) | Type 2 wrapper around Apple's Hypervisor.framework / QEMU |
| **Architecture** | x86/x86_64 virtualisation | ARM native + x86 emulation via QEMU |
| **Performance** | Near-native on x86 hardware | Near-native on ARM; slower for x86 emulation |
| **Disk format** | `.vdi` | `.qcow2` |
| **Snapshot support** | Built-in snapshot manager | Built-in snapshot/save state |
| **Use case for B2bR** | Default choice — subject assumes VirtualBox | Alternative for Apple Silicon Macs that can't run VirtualBox |

I used **VirtualBox** because it's the default specified by the subject and I'm working on x86 hardware.

---

## 💾 Partitioning & LVM (Bonus Layout)

This is the **bonus partitioning scheme**. The subject provides a reference diagram; the layout below matches it.

### 🔍 What is LVM?

**Logical Volume Management** is an abstraction layer between your physical disks and the file systems the OS sees. It introduces three concepts:

- **PV (Physical Volume):** The actual disk or partition (e.g., `/dev/sda5`).
- **VG (Volume Group):** A pool of storage created from one or more PVs. Think of it as a virtual disk.
- **LV (Logical Volume):** A carved-out chunk of a VG that you format and mount. Think of it as a virtual partition.

The power of LVM is flexibility. You can resize volumes, add disks to a VG on the fly, and create snapshots, none of which are possible (or at least easy) with traditional MBR/GPT partitions.

### 🔒 What about encryption?

The subject mandates **at least 2 encrypted partitions using LVM**. This is achieved via **LUKS** (Linux Unified Key Setup), which encrypts the physical volume before LVM sees it. At boot, the system asks for the passphrase, decrypts the PV, and then LVM assembles the volume group and logical volumes as normal.

### 🔍 Verifying your layout

```bash
lsblk
```

Your output should show the bonus structure: `sda` split into `sda1` (boot), `sda2` (extended), `sda5` (LUKS-encrypted PV), with logical volumes for `root`, `swap`, `home`, `var`, `srv`, `tmp`, and `var--log` carved out of the encrypted VG.

The key thing evaluators look for: **the LV names, mount points, and the fact that the volumes sit on top of an encrypted container**. If `lsblk` shows `crypt` in the type column, you're good.

### 🧠 Why separate partitions?

This is a real-world best practice. Isolating `/var/log` prevents a log-flooding attack from filling your root filesystem. Separating `/home` means a full reinstall doesn't nuke user data. Putting `/tmp` on its own volume lets you mount it with `nosuid` and `noexec` flags. Each decision has a security or operational rationale.

---

## 🔐 sudo Configuration

`sudo` ("superuser do") lets permitted users execute commands as root (or as another user) without sharing the root password. It's the gatekeeper between unprivileged and privileged execution.

### 📋 Subject requirements and their rationale

All sudo configuration lives in `/etc/sudoers` (edited via `visudo`) or in drop-in files under `/etc/sudoers.d/`.

| Requirement | Implementation | Why |
|---|---|---|
| Max 3 auth attempts | `Defaults passwd_tries=3` | Limits brute-force attempts in an interactive session |
| Custom error message | `Defaults badpass_message="<your message>"` | Just a subject requirement, in production you'd keep defaults |
| Log inputs and outputs | `Defaults log_input, log_output` | Full audit trail of what was run and what it produced |
| Log directory | `Defaults iolog_dir="/var/log/sudo"` | Centralises sudo audit logs |
| TTY required | `Defaults requiretty` | Prevents sudo from being invoked by background daemons or scripts without a terminal, reduces attack surface |
| Restricted PATH | `Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"` | Prevents PATH injection attacks where a malicious binary in a user-controlled directory shadows a system command |

### 🔍 Verification

```bash
sudo visudo                           # check the file (read-only if unsure)
ls -la /var/log/sudo/                 # confirm log directory exists
cat /var/log/sudo/*/log               # check sudo command history
sudo ls                               # run something, then re-check the log
```

---

## 👥 User & Group Management

The subject requires:

- A user with **your 42 login** as username
- That user must belong to groups **user42** and **sudo**

### 📋 Key commands

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

The `-aG` flag in `usermod` is critical: `-a` means *append* (don't replace existing groups), `-G` specifies the supplementary group. Forgetting `-a` will remove the user from all other supplementary groups

---

## 🔑 Password Policy

The password policy is split across two mechanisms: **login.defs** for age/expiry rules and **PAM** (specifically `pam_pwquality`) for complexity rules.

### ⏳ Age policy — `/etc/login.defs`

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

### 🔒 Complexity policy — `/etc/pam.d/common-password`

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

## 🌐 SSH Hardening

**SSH** (Secure Shell) provides encrypted remote access to the server. It replaces insecure protocols like Telnet and rlogin.

### 📋 Subject requirements

1. SSH runs on **port 4242** only (not the default 22)
2. **Root login via SSH is disabled**

### ⚙️ Configuration — `/etc/ssh/sshd_config`

```
Port 4242
PermitRootLogin no
```

After editing, restart the service:

```bash
sudo systemctl restart ssh
sudo systemctl status ssh              # confirm it's active on 4242
```

### 🔍 Verification during evaluation

```bash
# From the HOST machine (not the VM):
ssh <username>@127.0.0.1 -p 4242      # should work

# Attempt root login — must be denied:
ssh root@127.0.0.1 -p 4242            # should fail
```

For this to work, you need to set up **port forwarding** in VirtualBox: host port 4242 → guest port 4242 (Settings → Network → Advanced → Port Forwarding).

### 🧠 Why change the port?

Port 22 is the default SSH port. Every automated scanner on the internet hammers port 22 first. Changing it doesn't make SSH more secure cryptographically, but it massively reduces noise from brute-force bots. This is **security through obscurity** and not a primary defence, but a sensible layer.

---

## 🧱 UFW Firewall — UFW vs firewalld

**UFW** (Uncomplicated Firewall) is a front-end for `iptables`/`nftables`. It simplifies rule management for common use cases.

### 📋 Subject requirements

- UFW must be active at boot
- Only port **4242** is open

### ⚙️ Key commands

```bash
sudo ufw status verbose                # check status and default policies
sudo ufw status numbered               # list rules with numbers (useful for deletion)

# During evaluation — add and remove a port:
sudo ufw allow 8080                    # open port 8080
sudo ufw status numbered               # confirm it's listed
sudo ufw delete <rule_number>          # remove it (do this for both v4 and v6 rules)
```

### 🧠 What's happening under the hood?

UFW translates your simple `allow/deny` rules into `iptables` chains (or `nft` rules on newer kernels). When a packet arrives at the network interface, the kernel walks these chains and decides whether to ACCEPT, DROP, or REJECT the packet. The default policy for UFW is to **deny incoming** and **allow outgoing** — a sane baseline for any server.

### ⚖️ UFW vs firewalld

| | **UFW** | **firewalld** |
|---|---|---|
| **Used on** | Debian, Ubuntu | Rocky, Fedora, RHEL |
| **Backend** | iptables / nftables | nftables (iptables legacy fallback) |
| **Model** | Simple allow/deny rules per port/service | Zone-based — interfaces are assigned to zones with different trust levels |
| **Complexity** | Very low — ideal for single-purpose servers | Higher — powerful for multi-interface setups with different trust boundaries |
| **Runtime changes** | Requires `ufw reload` for some changes | Supports runtime vs permanent rules (apply without restart) |
| **Config style** | CLI commands → flat rule files | CLI (`firewall-cmd`) or XML zone files |

I used **UFW** because the subject mandates it for Debian. firewalld's zone model is more powerful but overkill for a single-port server.

---

## 🛡️ AppArmor — AppArmor vs SELinux

**AppArmor** is a Linux Security Module (LSM) that provides **Mandatory Access Control (MAC)**. Unlike traditional UNIX permissions (Discretionary Access Control — the user decides who can access their files), MAC is enforced by the kernel based on security policies, regardless of what the user wants.

AppArmor works with **profiles** — each profile defines what a specific program is allowed to access (files, network, capabilities). Profiles can run in:

- **Enforce mode** — violations are blocked and logged
- **Complain mode** — violations are logged but allowed

### 🔍 Verification

```bash
sudo aa-status                         # list loaded profiles and their modes
sudo systemctl status apparmor         # confirm it's running
```

AppArmor must be running at startup. On Debian, it's enabled by default.

### ⚖️ AppArmor vs SELinux

| | **AppArmor** | **SELinux** |
|---|---|---|
| **Used on** | Debian, Ubuntu, SUSE | Rocky, Fedora, RHEL, CentOS |
| **Model** | Path-based — rules reference file paths | Label-based — every file, process, and port gets a security context label |
| **Granularity** | Per-application profiles | System-wide policy covering all processes |
| **Learning curve** | Lower — profiles are human-readable and easy to write | Steep — requires understanding of type enforcement, roles, and booleans |
| **Default behaviour** | Unconfined processes are unrestricted | All processes are confined by default under the targeted policy |
| **Profile creation** | `aa-genprof` / `aa-logprof` to generate from observed behaviour | `audit2allow` to generate rules from denial logs |
| **Flexibility** | Simpler but less granular | Extremely granular — the gold standard for high-security environments |

I used **AppArmor** because it's the default MAC system on Debian and the subject requires it. SELinux offers finer-grained control but is significantly harder to configure and debug — it's the reason the subject warns that Rocky setup is complex.

---

## 📜 The monitoring.sh Script

The subject requires a bash script that broadcasts system information to all terminals every 10 minutes via `wall`.

### 📂 Script location

```bash
/usr/local/bin/monitoring.sh
```

## Monitoring Script Breakdown

The monitoring script (`monitoring.sh`) is executed every 10 minutes by a root cron job and broadcasts system information to all logged-in terminals using `wall`.

### Architecture

```bash
arch=$(uname -a)
```

`uname -a` prints all available system identification info: kernel name, hostname, kernel release, kernel version, machine hardware, processor type, hardware platform, and operating system. This single command gives a complete snapshot of the system identity.

### CPU Physical

```bash
cpuf=$(grep "physical id" /proc/cpuinfo | wc -l)
```

`/proc/cpuinfo` is a virtual file maintained by the kernel that describes each CPU core the system sees. Each entry has a `physical id` field identifying which physical chip the core belongs to. Counting these lines gives the number of physical CPU sockets. On a VirtualBox VM this will typically be `0` (since VirtualBox emulates a single socket starting at id `0`, and `grep` still matches that one line, though the count may vary depending on VM configuration).

### CPU Virtual

```bash
cpuv=$(grep "processor" /proc/cpuinfo | wc -l)
```

Every logical processing unit (vCPU) gets its own `processor` entry in `/proc/cpuinfo`. This includes all cores and hyperthreads. The count here reflects how many vCPUs were allocated to the VM in VirtualBox settings.

### RAM

```bash
ram_total=$(free --mega | awk '$1 == "Mem:" {print $2}')
ram_use=$(free --mega | awk '$1 == "Mem:" {print $3}')
ram_percent=$(free --mega | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
```

`free --mega` displays memory statistics in megabytes (base 10, i.e. 1 MB = 1,000,000 bytes). The `awk` filter targets the `Mem:` row specifically:
- `$2` is total physical RAM
- `$3` is currently used RAM (excluding buffers/cache)
- The percentage is calculated inline by dividing used by total and multiplying by 100, formatted to two decimal places

### Disk Usage

```bash
disk_total=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_t += $2} END {printf ("%.1fGb\n"), disk_t/1024}')
disk_use=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} END {print disk_u}')
disk_percent=$(df -m | grep "/dev/" | grep -v "/boot" | awk '{disk_u += $3} {disk_t+= $2} END {printf("%d"), disk_u/disk_t*100}')
```

`df -m` reports filesystem disk space in megabytes. The pipeline filters for real device-backed filesystems (`/dev/`) while excluding the boot partition (`/boot`), since the subject is only interested in main storage.

- `disk_total` accumulates the total size (`$2`) of all matching partitions and converts from MB to GB
- `disk_use` accumulates the used space (`$3`) across all matching partitions, left in MB
- `disk_percent` calculates the overall usage percentage by summing both used and total across all partitions before dividing, this gives a weighted average rather than averaging percentages, which would be misleading if partitions are different sizes

### CPU Load

```bash
cpul=$(vmstat 1 2 | tail -1 | awk '{printf $15}')
cpu_op=$(expr 100 - $cpul)
cpu_fin=$(printf "%.1f" $cpu_op)
```

`vmstat 1 2` takes two samples one second apart. The first sample is a summary since boot (less useful), so `tail -1` grabs only the second, which reflects current activity. Column 15 (`$15`) is the `id` (idle) percentage how much of the CPU is doing nothing. Subtracting from 100 gives the actual load. The result is formatted to one decimal place.

### Last Boot

```bash
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
```

`who -b` prints the time of the last system boot. The output looks like `system boot 2026-03-12 10:10`, so awk filters for the line starting with `system` and extracts the date (`$3`) and time (`$4`).

### LVM Use

```bash
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)
```

`lsblk` lists all block devices and their types. If any device has type `lvm` (Logical Volume Manager), the count will be greater than zero and the script prints `yes`. LVM is a storage abstraction layer that allows resizing, snapshotting, and flexible management of disk partitions

### TCP Connections

```bash
tcpc=$(ss -ta | grep ESTAB | wc -l)
```

`ss -ta` lists all TCP sockets (`-t` for TCP, `-a` for all states). Filtering for `ESTAB` counts only established connections,these are active, two-way communication channels between the VM and other hosts. This excludes sockets in listening, waiting, or closing states.

### User Log

```bash
ulog=$(users | wc -w)
```

`users` prints a space-separated list of currently logged-in usernames (one entry per session, so the same user logged in twice appears twice). `wc -w` counts the words, giving the total number of active login sessions.

### Network

```bash
ip=$(hostname -I)
mac=$(ip link | grep "link/ether" | awk '{print $2}')
```

`hostname -I` prints all non-loopback IP addresses assigned to the machine. `ip link` lists network interfaces with their properties; `link/ether` lines contain the MAC (Media Access Control) address, which is the unique hardware identifier burned into the network adapter. The MAC address is fixed and identifies the physical device, while the IP address is assigned by the network and can change.

### Sudo Commands

```bash
cmnd=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
```

`journalctl` queries the systemd journal (the centralised system log). `_COMM=sudo` filters for entries generated by the `sudo` binary, and `grep COMMAND` narrows to lines that record an actual command execution (as opposed to authentication attempts or session openings). This gives a count of how many commands have been run with elevated privileges since the journal began recording.

### Broadcasting

```bash
wall "Architecture: $arch
    CPU physical: $cpuf
    ..."
```

`wall` (write all) sends a message to every terminal listed in `/var/run/utmp`. This includes the VM console (tty) and any active SSH sessions (pts). The cron daemon executes this script every 10 minutes as root, ensuring all logged-in users see the system status regardless of which terminal they're connected through.


### 📋 What it displays

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

### 🧠 Key evaluation questions about the script

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

## ⏰ Cron

**Cron** is a time-based job scheduler. The cron daemon (`crond`) reads **crontab** files and executes commands at specified intervals.

### 📋 Crontab syntax

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

### ⚙️ Editing crontab

```bash
sudo crontab -u root -e               # edit root's crontab
sudo crontab -u root -l               # list root's crontab
```

---

## SSH `wall` Broadcast Fix
 
By default, the monitoring script's `wall` broadcast may not appear in SSH sessions. This happens because the `/var/run/utmp` file (which tracks active terminal sessions) may not exist on the system. Without it, `wall` has no way to discover which terminals are active and skips them.
 
### Diagnosing the Issue
 
```bash
# Check if your SSH session is registered
who
 
# If your session doesn't appear, check for the utmp file
ls -l /var/run/utmp
```
 
### Solution
 
Create the missing utmp file with the correct ownership and permissions:
 
```bash
sudo touch /var/run/utmp
```
Creates the empty utmp file. This is the database that login-tracking tools (`who`, `w`, `wall`, `last`) read from to determine which users are logged in and on which terminals. Without this file, none of these tools can function.
 
```bash
sudo chmod 664 /var/run/utmp
```
Sets read/write for the owner (`root`) and the group (`utmp`), and read-only for everyone else. The group write permission is critical, it allows PAM session modules and login utilities (which run as members of the `utmp` group) to register and deregister terminal sessions when users log in and out.
 
```bash
sudo chown root:utmp /var/run/utmp
```
Assigns ownership to `root` with the `utmp` group. This is the standard ownership on Debian systems. The `utmp` group exists specifically so that session-tracking processes can write to this file without needing root privileges.
 
After creating the file, disconnect and reconnect the SSH session. Verify with `who` that the session now appears, and the next `wall` broadcast from the monitoring cron job should be visible in both the VM console and the SSH terminal.

---

## ✅ Evaluation Quick-Reference

This is your pre-defence checklist. Run through these commands in order before your evaluator arrives.

### 🔍 Preliminary checks

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

### ⚙️ Service checks

```bash
sudo systemctl status ssh              # active, port 4242
sudo ufw status verbose                # active, only 4242 open
sudo aa-status                         # AppArmor loaded with profiles in enforce mode
```

### 👥 User & group checks

```bash
id <your_login>                        # should show user42 and sudo groups
getent group sudo
getent group user42
```

### 🔑 Password policy checks

```bash
sudo chage -l <your_login>            # verify age policy
sudo chage -l root                     # root too
cat /etc/pam.d/common-password         # verify pam_pwquality line
```

### 🔐 Sudo checks

```bash
sudo visudo                            # review the rules (or cat /etc/sudoers.d/<file>)
ls /var/log/sudo/                      # log directory exists
sudo ls                                # run something, then verify it was logged
```

### 🏷️ Hostname modification (evaluator will ask)

```bash
sudo hostnamectl set-hostname <new_hostname>
sudo nano /etc/hosts                   # update the hostname here too
sudo reboot                            # confirm it persists
# Then restore original hostname the same way
```

### 👤 User creation (evaluator will ask)

```bash
sudo adduser <new_user>                # follow prompts, use a compliant password
sudo groupadd evaluating
sudo usermod -aG evaluating <new_user>
getent group evaluating                # verify
```

### 🧱 UFW rule addition/removal (evaluator will ask)

```bash
sudo ufw allow 8080
sudo ufw status numbered
sudo ufw delete <rule_number>          # delete both IPv4 and IPv6 entries
sudo ufw status numbered               # confirm removal
```

### 🌐 SSH connection test

```bash
# From host terminal:
ssh <your_login>@127.0.0.1 -p 4242    # should succeed
ssh root@127.0.0.1 -p 4242            # should be denied
```

### 📜 Monitoring script

```bash
cat /usr/local/bin/monitoring.sh       # be ready to explain every line
sudo crontab -u root -e               # show the cron job, change interval if asked
```

---

## 📦 Submission

The files submitted to the Git repository are `README.md` and `signature.txt`, the latter containing the **SHA-1 hash** of the `.vdi` (or `.qcow2`) virtual disk image.

```bash
# On the HOST machine, navigate to the VM storage directory:
# Linux:   ~/VirtualBox VMs/
# macOS:   ~/VirtualBox VMs/  (or ~/Library/Containers/com.utmapp.UTM/Data/Documents/)
# Windows: %HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\

sha1sum <your_vm_name>.vdi             # Linux
shasum <your_vm_name>.vdi              # macOS
certUtil -hashfile <your_vm_name>.vdi sha1  # Windows
```

Paste the resulting hash into `signature.txt` at the root of your repo.

### ⚠️ Critical warnings

- Any change to the VM after generating the signature **will alter the hash**. Either duplicate the VM disk file or use VirtualBox's snapshot feature so you can restore the exact state that matches your submitted signature.
- **No snapshots may exist at the beginning of each evaluation.** A snapshot dedicated to the defence will be created and deleted at the end. Test the snapshot workflow before submitting.
- The VM itself must **never** be included in the Git repository.

---

## 🧠 Final Notes

Born2beRoot isn't really about memorising commands. It's about understanding *why* each configuration exists. Every `Defaults` line in your sudoers file, every `pam_pwquality` parameter, every firewall rule, they all map to a real threat model. Your evaluator isn't checking whether you can type `sudo ufw status`. They're checking whether you understand what happens when you do, and what would happen if you hadn't configured it.

Know your system. Defend your choices.

---

## 🎓 42 School Evaluation

**Grade:** N/A/100 ✅
**Bonus:** [YES]  
**Evaluation Date:** [N/A]

## 📝 License

This project is part of the 42 School curriculum. Feel free to reference this guide for learning purposes, but please complete your own 42 projects independently to get the full educational benefit.


**Author:** Alex Tanvuia (Ionut Tanvuia)

**42 Login:** itanvuia

**School:** 42 London

**Project Completed:** [February 2026]

[![42 Profile](https://img.shields.io/badge/42_Profile-itanvuia-000000?style=flat-square&logo=42)](https://profile.intra.42.fr/)

*Part of my journey through 42 School's peer-learning curriculum. Check out my other projects on my [GitHub profile](https://github.com/Axel92803)!*
