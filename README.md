# SRE/DEVOPS

# Linux skills & commands checklist for SRE / DevOps

This is a compact reference grouped by topic. Learn commands, options, and practice scenarios for each area.

- Basics / shell
  - Examples:
    ```bash
    # variable and command substitution
    user=$(whoami); echo "current user: $user"
    # simple pipeline and redirection
    ps aux | grep nginx > /tmp/nginx-processes.txt
    # background job and bring to foreground
    sleep 300 & jobs; fg %1
    ```
  - Explanation:
    - Use command substitution to capture output into variables (quick env checks).
    - Pipe output to grep and redirect to a file for inspection or later analysis.
    - Run long tasks in background and manage them with job control.

- File & text processing
  - Examples:
    ```bash
    # find and act on files
    find /var/log -name '*.log' -mtime +30 -print
    # extract column and sort/uniq
    awk -F: '{print $1}' /etc/passwd | sort | uniq -c | sort -nr | head
    # quick text search
    rg -n "ERROR" /srv/app || true
    ```
  - Explanation:
    - find filters files by name and age for cleanup/archival.
    - awk + sort + uniq is useful for frequency analysis (e.g., users, keys).
    - ripgrep (rg) is a fast recursive search tool; || true avoids non-zero exit when no matches.

- Permissions & users
  - Examples:
    ```bash
    # change owner and mode
    sudo chown appuser:appgroup /srv/app && sudo chmod 750 /srv/app
    # add user and set shell
    sudo useradd -m -s /bin/bash deployer && sudo passwd deployer
    # set an ACL
    sudo setfacl -m u:ci:rwx /srv/build
    ```
  - Explanation:
    - chown/chmod set identity and access; prefer least privilege for app dirs.
    - useradd with -m creates a home and sets a shell; follow with passwd or key setup.
    - setfacl allows per-user permissions without changing primary ownership.

- Processes & systemd
  - Examples:
    ```bash
    # check and restart a service
    sudo systemctl status nginx && sudo systemctl restart nginx
    # follow journal for a unit
    sudo journalctl -u myapp.service -f --since "10 minutes ago"
    # kill by name
    pkill -f "node server.js"
    ```
  - Explanation:
    - Use systemctl for lifecycle ops and quick status checks.
    - journalctl -f streams logs in real time for the unit; --since helps narrow scope.
    - pkill -f matches the full command line; use carefully to avoid collateral termination.

- Networking
  - Examples:
    ```bash
    # check local interfaces and routes
    ip addr show && ip route show
    # test connectivity and HTTP endpoint
    curl -I https://example.com
    # capture 100 packets on eth0
    sudo tcpdump -i eth0 -c 100 -w /tmp/capture.pcap
    ```
  - Explanation:
    - ip shows interface addresses and routing for troubleshooting connectivity.
    - curl -I validates HTTP reachability and response headers quickly.
    - tcpdump captures packets for offline analysis with Wireshark/tshark.

- Disks & filesystems
  - Examples:
    ```bash
    # check disk usage and big directories
    df -hT / && du -sh /var/log/* | sort -h
    # list block devices and partitions
    lsblk -f
    # extend LVM logical volume (example)
    sudo lvextend -L +10G /dev/vg0/lv_root && sudo resize2fs /dev/vg0/lv_root
    ```
  - Explanation:
    - df/du find capacity and the biggest consumers.
    - lsblk shows device topology and filesystem types.
    - LVM extend needs logical volume resize and filesystem grow; commands vary by FS.

- Storage & backup
  - Examples:
    ```bash
    # rsync incremental copy (dry run)
    rsync -av --delete --dry-run /srv/data/ backup:/srv/data/
    # create compressed tarball
    tar -czf /tmp/site-$(date +%F).tar.gz /srv/site
    # restic backup (example)
    restic -r s3:s3.amazonaws.com/myrepo backup /etc
    ```
  - Explanation:
    - rsync is efficient for incremental syncs; use --dry-run to validate.
    - tar + compression creates portable archives for snapshotting data.
    - restic/borg provide deduped, encrypted backups to remote repos.

- Package management & building
  - Examples:
    ```bash
    # Debian package install and inspect
    sudo apt update && sudo apt install -y htop
    dpkg -l | rg htop
    # build a simple C program
    gcc -O2 -o hello hello.c
    ```
  - Explanation:
    - Refresh package lists before installs; verify with dpkg or rpm queries.
    - Build tools are needed for compiling from source or CI build steps.

- Monitoring & observability
  - Examples:
    ```bash
    # check memory and load
    free -m; uptime
    # quick Prometheus node exporter run (container)
    docker run -d --name node_exporter -p 9100:9100 prom/node-exporter
    # inspect last logs for a service
    sudo journalctl -u postgres -n 200 --no-pager
    ```
  - Explanation:
    - free/uptime provide immediate resource and load context.
    - node_exporter exposes metrics for Prometheus; run in container for testing.
    - journalctl extracts service logs for incident analysis.

- Performance & debugging
  - Examples:
    ```bash
    # trace syscalls of a process
    sudo strace -p 1234 -o /tmp/strace.out
    # list open files for a PID
    sudo lsof -p 1234
    # show socket listeners
    ss -tulpen
    ```
  - Explanation:
    - strace helps find failing syscalls or blocking I/O.
    - lsof reveals file/socket handles that may indicate leaks.
    - ss lists listening sockets and owning processes.

- Containers & orchestration
  - Examples:
    ```bash
    # run and inspect a container
    docker run -d --name nginx-test -p 8080:80 nginx:stable
    docker logs -f nginx-test
    # kubectl basic
    kubectl get pods -A
    kubectl logs deployment/myapp -c web --tail=200
    ```
  - Explanation:
    - Run ephemeral containers for smoke tests; check logs to validate behavior.
    - kubectl is primary for cluster debugging; use namespace/all flags when needed.

- CI/CD & automation
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
  - Explanation:
    - Branching and small commits improve traceability in CI.
    - Ansible ad-hoc commands validate connectivity/target state quickly.
    - set -euo pipefail makes scripts fail fast and safer in automation.

- Security & auditing
  - Examples:
    ```bash
    # generate and copy SSH key
    ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
    ssh-copy-id -i ~/.ssh/id_ed25519.pub deployer@host
    # check SELinux status
    getenforce || sudo getenforce
    ```
  - Explanation:
    - Use modern key types (ed25519) and distribute public keys rather than passwords.
    - Check SELinux/AppArmor to avoid unexpected permission denials during troubleshooting.

- High-availability & clustering
  - Examples:
    ```bash
    # test HA proxy config and reload
    sudo haproxy -c -f /etc/haproxy/haproxy.cfg && sudo systemctl reload haproxy
    # quick rsync for DR
    rsync -avz --delete /srv/ important-backup:/srv/
    ```
  - Explanation:
    - Validate HA/proxy configs before reload to avoid downtime.
    - Use rsync (with fencing and access controls) as a simple DR synchronization tool.

- Troubleshooting playbook items (commands to run when things break)
  - Examples:
    ```bash
    # basic playbook steps
    sudo systemctl status mysvc || sudo journalctl -u mysvc -n 200 -f
    df -h /var && du -sh /var/log/* | sort -h | tail
    # capture network for 60s
    sudo tcpdump -i eth0 -w /tmp/trace.pcap -G 60 -W 1
    ```
  - Explanation:
    - Start with service status and recent logs; then check disk, processes, and network.
    - Capture short packet traces for reproducible network issues.

- Misc & ergonomics
  - Examples:
    ```bash
    # add a cron job for root
    sudo crontab -l | { cat; echo "0 3 * * * /usr/local/bin/backup.sh"; } | sudo crontab -
    # safe script logging
    exec > >(tee -a /var/log/myscript.log) 2>&1
    ```
  - Explanation:
    - Manage scheduled tasks centrally and test cron entries.
    - Route script output to logs for auditing and easier debugging.

# Snippets (one-line examples per section)

- Basics: `user=$(whoami) && echo "user:$user"` — prints the current shell user (quick identity check).
- File & text processing: `rg -n "ERROR" /var/log || true` — search logs for "ERROR" with line numbers, return success even if none found.
- Permissions & users: `sudo chown appuser:appgroup /srv/app && sudo chmod 750 /srv/app` — set owner/group and restrictive permissions for an app directory.
- Processes & systemd: `sudo systemctl restart nginx && sudo systemctl status nginx --no-pager` — restart nginx and show immediate status output.
- Networking: `curl -I https://example.com` — fetch HTTP headers to validate reachability and response code.
- Disks & filesystems: `df -h / && du -sh /var/log | sort -h | tail` — show root usage and the largest entries under /var/log.
- Storage & backup: `rsync -avz /srv/data/ backup:/srv/data/` — perform an efficient, compressed sync to a remote backup host.
- Package management & building: `sudo apt update && sudo apt install -y htop` — refresh package lists and install a monitoring tool.
- Monitoring & observability: `free -m; uptime` — quick memory and load snapshot for capacity checks.
- Performance & debugging: `sudo strace -p 1234 -o /tmp/strace.out` — record syscalls of PID 1234 for debugging.
- Containers & orchestration: `docker run -d --name nginx-test -p 8080:80 nginx:stable` — launch a test nginx container bound to localhost:8080.
- CI/CD & automation: `git checkout -b feature/xyz && git commit --allow-empty -m "wip"` — create a feature branch and record a WIP commit.
- Security & auditing: `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""` — create a new ed25519 SSH keypair without a passphrase.
- High-availability & clustering: `sudo haproxy -c -f /etc/haproxy/haproxy.cfg && sudo systemctl reload haproxy` — validate HAProxy config and reload service if valid.
- Troubleshooting playbook items: `sudo journalctl -u mysvc -n 200 --no-pager` — view recent logs for a failing service for quick diagnostics.
- Misc & ergonomics: `sudo crontab -l | { cat; echo "0 3 * * * /usr/local/bin/backup.sh"; } | sudo crontab -` — append a daily root cron job safely.

