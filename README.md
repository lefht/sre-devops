# SRE/DEVOPS

# Linux skills & commands checklist for SRE / DevOps

This is a compact reference grouped by topic. Learn commands, options, and practice scenarios for each area.

- Basics / shell
  - bash / sh: variable, subshells, command substitution: $(...), `...`
  - history, ctrl+r, alias, functions, env, export
  - piping & redirection: |, >, >>, 2>, &>, <, heredoc
  - job control: &, fg, bg, jobs, disown
  - tmux / screen

- File & text processing
  - ls, cp, mv, rm, mkdir, rmdir, stat
  - find, locate, updatedb
  - grep, egrep, fgrep, ripgrep (rg)
  - awk, sed, cut, sort, uniq, tr, paste
  - head, tail, less, more, wc
  - xargs, tee
  - file, strings, hexdump, od

- Permissions & users
  - chmod, chown, chgrp, umask
  - passwd, useradd, usermod, userdel, groupadd
  - su, sudo, visudo, sudoers
  - setfacl / getfacl, setfacl default ACLs

- Processes & systemd
  - ps, top, htop, pstree
  - nice, renice, ionice
  - kill, killall, pkill
  - systemctl (start/stop/status/enable/disable/restart)
  - journalctl (logs filtering, -u service, -f, --since/--until)
  - service, chkconfig (legacy)

- Networking
  - ip (ip addr, ip link, ip route), ss, netstat (legacy)
  - ifconfig (legacy), ethtool, nmcli
  - ping, traceroute, mtr
  - nslookup, dig, host
  - curl, wget, httpie
  - nc / netcat, ncat, socat
  - tcpdump, tshark, wireshark
  - iptables / nftables, ufw, firewalld
  - ipvsadm (for LVS), conntrack

- Disks & filesystems
  - df, du, lsblk, fdisk, cfdisk, parted
  - mount, umount, /etc/fstab
  - blkid, tune2fs, dumpe2fs
  - mkfs.*, fsck
  - LVM: pvcreate, vgcreate, lvcreate, lvextend, pvs/vgs/lvs
  - mdadm (software RAID), btrfs, xfs tools

- Storage & backup
  - rsync, scp, sftp
  - tar, gzip, bz2, xz, zstd
  - dd, pv
  - borg, restic, duplicity, rclone

- Package management & building
  - apt, apt-get, dpkg
  - yum, dnf, rpm
  - pacman, zypper (as applicable)
  - make, cmake, gcc, strip

- Monitoring & observability
  - top/htop, vmstat, iostat (sysstat), sar
  - free, uptime
  - dstat, iotop
  - collectd, Prometheus (node_exporter), Grafana basics
  - journalctl, /var/log, logrotate, rsyslog, syslog-ng
  - curl + health checks, blackbox exporter

- Performance & debugging
  - strace, ltrace
  - perf, bpftrace
  - lsof, ss, netstat
  - ftrace, systemtap (advanced)
  - ulimit, sysctl, /proc tuning

- Containers & orchestration
  - docker / podman: run, exec, build, images, ps, logs, cp, network, volumes
  - docker-compose
  - kubectl: get, describe, logs, exec, apply, port-forward
  - helm, kubeadm basics
  - ctr / crictl for containerd, crictl exec/logs

- CI/CD & automation
  - git (clone, commit, branch, rebase, cherry-pick, bisect), git hooks
  - Jenkins / GitLab CI / GitHub Actions basics (running jobs, logs)
  - ansible: ad-hoc, playbooks; terraform basics
  - shell scripting: robust scripts, set -euo pipefail, trap

- Security & auditing
  - openssl, ssh-keygen, ssh-agent, ssh-copy-id
  - fail2ban, auditd
  - SELinux (getenforce, setenforce, semanage, restorecon) or AppArmor tools
  - chkrootkit, rkhunter (awareness)
  - tcpwrappers basics, nmap

- High-availability & clustering
  - keepalived, haproxy, nginx (reverse proxy), pacemaker/corosync basics
  - rsync + fencing strategies, DR drills

- Troubleshooting playbook items (commands to run when things break)
  - check service: systemctl status <svc>, journalctl -u <svc> -n 200 -f
  - check processes: ps aux | grep, top/htop
  - check disk: df -h, du -sh /var/log/*
  - check network: ip route, ss -tnlp, netstat -tulpen
  - check open files: lsof -p <pid> or lsof /path
  - check kernel messages: dmesg | tail -n 200
  - reproducible capturing: tcpdump -w /tmp/capture.pcap
  - collect system info: uname -a, cat /etc/os-release, free -m, vmstat 1 5

- Misc & ergonomics
  - scripting best practices, logging, exit codes
  - cron / crontab, at
  - environment management: /etc/profile, ~/.bashrc, /etc/environment
  - system snapshots, rollback strategies (filesystem or VM-level)
  - versioning of infra (IaC), runbooks, runbook automation

Quick learning exercises:
- Create a small LAMP stack, create a systemd unit for the app, add health checks.
- Break and recover: fill disk, then free space and explain steps.
- Containerize an app, deploy to Kubernetes, expose, and collect logs/metrics.
- Write an idempotent Ansible playbook to install and configure a service.

Use this checklist as a living document — add commands, options, and examples as you practice and encounter real incidents.