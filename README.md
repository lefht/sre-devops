# SRE/DEVOPS

# Linux skills & commands checklist for SRE / DevOps

This is a compact reference grouped by topic. Learn commands, options, and practice scenarios for each area.

- Basics / shell
  - bash / sh: variable, subshells, command substitution: $(...), `...`
  - history, ctrl+r, alias, functions, env, export
  - piping & redirection: |, >, >>, 2>, &>, <, heredoc
  - job control: &, fg, bg, jobs, disown
  - tmux / screen

  - Examples:
    ```bash
    # variable and command substitution
    user=$(whoami); echo "current user: $user"
    # simple pipeline and redirection
    ps aux | grep nginx > /tmp/nginx-processes.txt
    # background job and bring to foreground
    sleep 300 & jobs; fg %1
    ```

- File & text processing
  - ls, cp, mv, rm, mkdir, rmdir, stat
  - find, locate, updatedb
  - grep, egrep, fgrep, ripgrep (rg)
  - awk, sed, cut, sort, uniq, tr, paste
  - head, tail, less, more, wc
  - xargs, tee
  - file, strings, hexdump, od

  - Examples:
    ```bash
    # find and act on files
    find /var/log -name '*.log' -mtime +30 -print
    # extract column and sort/uniq
    awk -F: '{print $1}' /etc/passwd | sort | uniq -c | sort -nr | head
    # quick text search
    rg -n "ERROR" /srv/app || true
    ```

- Permissions & users
  - chmod, chown, chgrp, umask
  - passwd, useradd, usermod, userdel, groupadd
  - su, sudo, visudo, sudoers
  - setfacl / getfacl, setfacl default ACLs

  - Examples:
    ```bash
    # change owner and mode
    sudo chown appuser:appgroup /srv/app && sudo chmod 750 /srv/app
    # add user and set shell
    sudo useradd -m -s /bin/bash deployer && sudo passwd deployer
    # set an ACL
    sudo setfacl -m u:ci:rwx /srv/build
    ```

- Processes & systemd
  - ps, top, htop, pstree
  - nice, renice, ionice
  - kill, killall, pkill
  - systemctl (start/stop/status/enable/disable/restart)
  - journalctl (logs filtering, -u service, -f, --since/--until)
  - service, chkconfig (legacy)

  - Examples:
    ```bash
    # check and restart a service
    sudo systemctl status nginx && sudo systemctl restart nginx
    # follow journal for a unit
    sudo journalctl -u myapp.service -f --since "10 minutes ago"
    # kill by name
    pkill -f "node server.js"
    ```

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

  - Examples:
    ```bash
    # check local interfaces and routes
    ip addr show && ip route show
    # test connectivity and HTTP endpoint
    curl -I https://example.com
    # capture 100 packets on eth0
    sudo tcpdump -i eth0 -c 100 -w /tmp/capture.pcap
    ```

- Disks & filesystems
  - df, du, lsblk, fdisk, cfdisk, parted
  - mount, umount, /etc/fstab
  - blkid, tune2fs, dumpe2fs
  - mkfs.*, fsck
  - LVM: pvcreate, vgcreate, lvcreate, lvextend, pvs/vgs/lvs
  - mdadm (software RAID), btrfs, xfs tools

  - Examples:
    ```bash
    # check disk usage and big directories
    df -hT / && du -sh /var/log/* | sort -h
    # list block devices and partitions
    lsblk -f
    # extend LVM logical volume (example)
    sudo lvextend -L +10G /dev/vg0/lv_root && sudo resize2fs /dev/vg0/lv_root
    ```

- Storage & backup
  - rsync, scp, sftp
  - tar, gzip, bz2, xz, zstd
  - dd, pv
  - borg, restic, duplicity, rclone

  - Examples:
    ```bash
    # rsync incremental copy (dry run)
    rsync -av --delete --dry-run /srv/data/ backup:/srv/data/
    # create compressed tarball
    tar -czf /tmp/site-$(date +%F).tar.gz /srv/site
    # restic backup (example)
    restic -r s3:s3.amazonaws.com/myrepo backup /etc
    ```

- Package management & building
  - apt, apt-get, dpkg
  - yum, dnf, rpm
  - pacman, zypper (as applicable)
  - make, cmake, gcc, strip

  - Examples:
    ```bash
    # Debian package install and inspect
    sudo apt update && sudo apt install -y htop
    dpkg -l | rg htop
    # build a simple C program
    gcc -O2 -o hello hello.c
    ```

- Monitoring & observability
  - top/htop, vmstat, iostat (sysstat), sar
  - free, uptime
  - dstat, iotop
  - collectd, Prometheus (node_exporter), Grafana basics
  - journalctl, /var/log, logrotate, rsyslog, syslog-ng
  - curl + health checks, blackbox exporter

  - Examples:
    ```bash
    # check memory and load
    free -m; uptime
    # quick Prometheus node exporter run (container)
    docker run -d --name node_exporter -p 9100:9100 prom/node-exporter
    # inspect last logs for a service
    sudo journalctl -u postgres -n 200 --no-pager
    ```

- Performance & debugging
  - strace, ltrace
  - perf, bpftrace
  - lsof, ss, netstat
  - ftrace, systemtap (advanced)
  - ulimit, sysctl, /proc tuning

  - Examples:
    ```bash
    # trace syscalls of a process
    sudo strace -p 1234 -o /tmp/strace.out
    # list open files for a PID
    sudo lsof -p 1234
    # show socket listeners
    ss -tulpen
    ```

- Containers & orchestration
  - docker / podman: run, exec, build, images, ps, logs, cp, network, volumes
  - docker-compose
  - kubectl: get, describe, logs, exec, apply, port-forward
  - helm, kubeadm basics
  - ctr / crictl for containerd, crictl exec/logs

  - Examples:
    ```bash
    # run and inspect a container
    docker run -d --name nginx-test -p 8080:80 nginx:stable
    docker logs -f nginx-test
    # kubectl basic
    kubectl get pods -A
    kubectl logs deployment/myapp -c web --tail=200
    ```

- CI/CD & automation
  - git (clone, commit, branch, rebase, cherry-pick, bisect), git hooks
  - Jenkins / GitLab CI / GitHub Actions basics (running jobs, logs)
  - ansible: ad-hoc, playbooks; terraform basics
  - shell scripting: robust scripts, set -euo pipefail, trap

  - Examples:
    ```bash
    # simple git workflow
    git checkout -b feature/xyz && git add . && git commit -m "feat: xyz"
    # run an Ansible ad-hoc command
    ansible all -i inventory -m ping
    # safe bash header for scripts
    #!/usr/bin/env bash
    set -euo pipefail
    ```

- Security & auditing
  - openssl, ssh-keygen, ssh-agent, ssh-copy-id
  - fail2ban, auditd
  - SELinux (getenforce, setenforce, semanage, restorecon) or AppArmor tools
  - chkrootkit, rkhunter (awareness)
  - tcpwrappers basics, nmap

  - Examples:
    ```bash
    # generate and copy SSH key
    ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
    ssh-copy-id -i ~/.ssh/id_ed25519.pub deployer@host
    # check SELinux status
    getenforce || sudo getenforce
    ```

- High-availability & clustering
  - keepalived, haproxy, nginx (reverse proxy), pacemaker/corosync basics
  - rsync + fencing strategies, DR drills

  - Examples:
    ```bash
    # test HA proxy config and reload
    sudo haproxy -c -f /etc/haproxy/haproxy.cfg && sudo systemctl reload haproxy
    # quick rsync for DR
    rsync -avz --delete /srv/ important-backup:/srv/
    ```

- Troubleshooting playbook items (commands to run when things break)
  - check service: systemctl status <svc>, journalctl -u <svc> -n 200 -f
  - check processes: ps aux | grep, top/htop
  - check disk: df -h, du -sh /var/log/*
  - check network: ip route, ss -tnlp, netstat -tulpen
  - check open files: lsof -p <pid> or lsof /path
  - check kernel messages: dmesg | tail -n 200
  - reproducible capturing: tcpdump -w /tmp/capture.pcap
  - collect system info: uname -a, cat /etc/os-release, free -m, vmstat 1 5

  - Examples:
    ```bash
    # basic playbook steps
    sudo systemctl status mysvc || sudo journalctl -u mysvc -n 200 -f
    df -h /var && du -sh /var/log/* | sort -h | tail
    # capture network for 60s
    sudo tcpdump -i eth0 -w /tmp/trace.pcap -G 60 -W 1
    ```

- Misc & ergonomics
  - scripting best practices, logging, exit codes
  - cron / crontab, at
  - environment management: /etc/profile, ~/.bashrc, /etc/environment
  - system snapshots, rollback strategies (filesystem or VM-level)
  - versioning of infra (IaC), runbooks, runbook automation

  - Examples:
    ```bash
    # add a cron job for root
    sudo crontab -l | { cat; echo "0 3 * * * /usr/local/bin/backup.sh"; } | sudo crontab -
    # safe script logging
    exec > >(tee -a /var/log/myscript.log) 2>&1
    ```

# Snippets (one-line examples per section)

- Basics: `user=$(whoami) && echo "user:$user"`
- File & text processing: `rg -n "ERROR" /var/log || true`
- Permissions & users: `sudo chown appuser:appgroup /srv/app && sudo chmod 750 /srv/app`
- Processes & systemd: `sudo systemctl restart nginx && sudo systemctl status nginx --no-pager`
- Networking: `curl -I https://example.com`
- Disks & filesystems: `df -h / && du -sh /var/log | sort -h | tail`
- Storage & backup: `rsync -avz /srv/data/ backup:/srv/data/`
- Package management & building: `sudo apt update && sudo apt install -y htop`
- Monitoring & observability: `free -m; uptime`
- Performance & debugging: `sudo strace -p 1234 -o /tmp/strace.out`
- Containers & orchestration: `docker run -d --name nginx-test -p 8080:80 nginx:stable`
- CI/CD & automation: `git checkout -b feature/xyz && git commit --allow-empty -m "wip"`
- Security & auditing: `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""`
- High-availability & clustering: `sudo haproxy -c -f /etc/haproxy/haproxy.cfg && sudo systemctl reload haproxy`
- Troubleshooting playbook items: `sudo journalctl -u mysvc -n 200 --no-pager`
- Misc & ergonomics: `sudo crontab -l | { cat; echo "0 3 * * * /usr/local/bin/backup.sh"; } | sudo crontab -`

