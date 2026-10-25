# linux-init-systems-lab
Hands-on Linux labs exploring SysVinit, OpenRC, runit, and systemd, including the boot process, PID 1, service management, troubleshooting, and Linux internals for aspiring Linux Kernel Test Engineers.

# 🐧 Linux Init Systems Lab

**Understanding how Linux boots, starts services, and manages processes.**

![Linux](https://img.shields.io/badge/Platform-Linux-black?logo=linux)
![Learning](https://img.shields.io/badge/Focus-Linux%20Internals-blue)
![Labs](https://img.shields.io/badge/Approach-Hands--On-green)
![Status](https://img.shields.io/badge/Project-Learning%20in%20Progress-orange)

## 🎯 Hook — What happens after the Linux kernel boots?

The Linux kernel initializes the system, but it does not independently manage every userspace service.

After the kernel starts userspace, the first process in the initial PID namespace normally becomes responsible for bringing up the rest of the system.

This repository explores the software responsible for that work: **Linux init systems and service managers.**

## 🚨 Problem — Why does this matter?

Many Linux learners know commands such as `ls`, `cd`, `ps`, and `systemctl` without fully understanding how Linux starts.

Questions worth investigating include:

- What is PID 1?
- What happens after the kernel starts its first userspace process?
- How does Linux start networking, SSH, logging, and other services?
- Why might a system boot successfully while a service fails?
- What is the difference between a kernel failure and a userspace startup failure?
- Why do different Linux distributions use different init systems?

Understanding these concepts helps with Linux troubleshooting, system administration, and Linux Kernel Test Engineering.

## 🧠 Explain — What is an init system?

An init system is userspace software responsible for starting and managing system services and coordinating parts of the startup process.

Four implementations explored in this repository are:

| Init system | Approach | Key concept |
|---|---|---|
| SysVinit | Traditional runlevels and scripts | Sequential service startup and runlevel management |
| OpenRC | Dependency-aware service management | Runlevels and service dependencies |
| runit | Lightweight service supervision | Monitoring and restarting services |
| systemd | Integrated service and system management | Units, dependencies, targets, timers, and journaling |

These systems are alternatives, not four components that must all run together.

### 1. SysVinit

SysVinit follows the traditional Unix-style initialization model.

Important concepts:
- Runlevels
- Init scripts
- `/etc/inittab` on applicable systems
- `/etc/init.d/`
- Startup ordering

Example:

```bash
service ssh start
service ssh status
```

Command availability and service names vary by distribution.

### 2. OpenRC

OpenRC provides dependency-based service management and runlevels.

Important concepts:
- Service scripts
- Dependency relationships
- Default and other runlevels
- Service startup and shutdown

Example:

```bash
rc-service sshd status
rc-update show
```

These commands apply to systems configured with OpenRC. The service may be named `ssh` rather than `sshd` on some distributions.

### 3. runit

runit is a lightweight init and service supervision system.

Important concepts:
- Service supervision
- Service run scripts
- Process monitoring
- Restart behavior
- Service lifecycle

Example:

```bash
sv status /etc/service/sshd
```

The service path is illustrative; actual service directories depend on the distribution and configuration.

### 4. systemd

systemd is a widely used init and system-management implementation.

Important concepts:
- Units and unit files
- Services and targets
- Dependency relationships
- Socket activation and timers
- Journald logging
- Service restart policies

Example:

```bash
systemctl status ssh
systemctl --failed
journalctl -b -p warning
```

On systems where the SSH service is named `sshd`, use that unit name instead.

## 🔬 Demonstrate — Hands-on Linux labs

The labs begin with safe inspection commands and progress toward service troubleshooting.

### Lab 1: Identify PID 1

```bash
ps -p 1 -o pid,comm,args
```

**Objective:** Identify the first userspace process visible in the initial PID namespace.

Questions:
- What is its PID?
- What executable is running?
- Is it systemd or another init implementation?

### Lab 2: Inspect the init executable

```bash
ls -l /sbin/init
readlink -f /sbin/init
```

**Objective:** Investigate how the distribution exposes its init executable.

Note: `/sbin/init` may be a symlink, a directly installed executable, or absent in some environments.

### Lab 3: Identify the Linux kernel version

```bash
uname -r
uname -a
```

**Objective:** Record the running kernel version and architecture.

Explain why the kernel version alone does not tell you which init system is running.

### Lab 4: Inspect running processes

```bash
ps -eo pid,ppid,comm --sort=pid | head -n 20
```

**Objective:** Observe process IDs and parent process IDs.

Investigate how process relationships help you understand userspace startup.

### Lab 5: Inspect services on systemd

```bash
systemctl list-units --type=service
systemctl --failed
systemctl status ssh
```

**Objective:** List loaded service units, identify failed units, and inspect an individual service.

A service may be inactive without being broken. Interpret its expected state before declaring a defect.

### Lab 6: Inspect boot logs

```bash
journalctl -b
journalctl -b -p err
journalctl -k -b
```

**Objective:** Examine system logs and kernel messages from the current boot.

Compare:
- Kernel messages
- Userspace startup messages
- Service failures
- Timestamps and event ordering

These commands assume systemd's journal is available.

### Lab 7: Investigate a startup failure

Use a disposable virtual machine or a purpose-built test service.

1. Record the baseline service state.
2. Read the service configuration and dependencies.
3. Collect relevant logs.
4. Identify the failure condition.
5. Reproduce the issue in a controlled environment.
6. Apply one change at a time.
7. Repeat the test.
8. Document the root cause and evidence.

Never disable essential production services merely to practise troubleshooting.

## 🧪 Kernel testing connection

Init systems operate in userspace, but their failures can reveal important system-level problems.

| Observation | Possible investigation |
|---|---|
| Kernel panic during boot | Kernel logs, configuration, hardware or virtual hardware |
| Root filesystem cannot be mounted | Kernel storage drivers, initramfs, filesystem and boot parameters |
| Kernel boots but networking does not start | Driver state, interface configuration, service dependencies and logs |
| A service repeatedly exits | Service configuration, permissions, dependencies and kernel messages |
| PID 1 fails to start | Init executable, root filesystem, userspace libraries and boot configuration |

These are diagnostic starting points, not definitive root causes.

## 📚 Learning objectives

By completing this repository, I aim to:

- Explain the Linux boot process.
- Identify PID 1 and understand its role.
- Compare SysVinit, OpenRC, runit, and systemd.
- Inspect service states and process relationships.
- Read kernel and userspace logs.
- Reproduce and document startup failures.
- Develop structured troubleshooting and defect-reporting skills.
- Build a foundation for Linux Kernel Test Engineering.

## 🗂️ Suggested repository structure

```text
linux-init-systems-lab/
├── README.md
├── docs/
│   ├── linux-boot-process.md
│   ├── pid-1-explained.md
│   └── init-systems-comparison.md
├── labs/
│   ├── 01-identify-pid1.md
│   ├── 02-inspect-init.md
│   ├── 03-process-investigation.md
│   ├── 04-systemd-services.md
│   ├── 05-boot-log-analysis.md
│   └── 06-service-failure-investigation.md
├── troubleshooting/
│   └── defect-report-template.md
├── scripts/
│   └── collect-system-info.sh
└── LICENSE
```

## 📝 Defect report template

When investigating a reproducible issue, record:

- **Title:** A short description of the failure
- **Environment:** Distribution, version, kernel version, init system, and VM configuration
- **Prerequisites:** Required packages, configuration, and starting state
- **Steps to reproduce:** Exact commands and actions
- **Expected result:** What should happen
- **Actual result:** What happened instead
- **Evidence:** Logs, exit codes, screenshots, and configuration details
- **Reproducibility:** Whether the issue occurs consistently
- **Workaround:** Any verified temporary mitigation
- **Conclusion:** Confirmed cause or remaining hypotheses

Separate observed facts from assumptions. Never report an unverified hypothesis as a confirmed root cause.

## 💡 Lesson

The Linux kernel and the init system have different responsibilities.

The kernel manages core operating-system mechanisms, while the init system coordinates userspace startup and service management.

Learning to distinguish between these layers is essential for effective Linux debugging and kernel testing.

**Build → Break → Analyze → Repeat → Document.**

## 🚀 Call to action

If you are learning Linux internals, system administration, or kernel testing:

- ⭐ Star this repository.
- 🍴 Fork it and complete the labs.
- 🧪 Reproduce problems in a disposable VM.
- 📝 Document your findings.
- 🤝 Contribute corrections and additional exercises.

Follow the project as I develop my practical skills toward becoming a Linux Kernel Test Engineer.

**No system is safe. Linux is the backbone of all technology infrastructures.** 🐧🔐

---

*This is a learning project. All experiments should be conducted on systems you own or are authorized to test.*
