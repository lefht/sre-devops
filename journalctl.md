# journalctl — systemd journal deep dive (beginner → advanced)

This document explains systemd's journal and the `journalctl` utility from first principles to advanced usage with practical examples.

---

## 1) What is the systemd journal?

- The systemd journal is a structured, binary log storage maintained by systemd-journald.
- It collects messages from the kernel, init system, services, stdout/stderr of services, and optionally syslog daemons.
- Stored as binary files (usually under /run/log/journal for volatile or /var/log/journal for persistent).
- Journald stores structured fields (timestamp, unit, PID, uid, message, and many metadata fields) enabling precise filtering.

Why use journalctl?
- Structured logs: fields allow powerful, accurate queries.
- Unified source: kernel + services + stdout/stderr in one place.
- Flexible output formats (JSON, short, verbose) for human and machine consumption.
- Rotation and rate-limiting capabilities.

---

## 2) Anatomy of a journal entry (fields)
Common fields you will see with `-o verbose` or JSON:
- __REALTIME_TIMESTAMP__: epoch time in microseconds.
- __MONOTONIC_TIMESTAMP__: boot-relative microseconds.
- _PID, _UID, _GID_: process and user IDs.
- _COMM_: executable name.
- _EXE_: full path to executable binary.
- _SYSTEMD_UNIT_: systemd unit that generated the entry.
- _TRANSPORT_: origin (syslog, kernel, journal, stdout).
- SYSLOG_IDENTIFIER, MESSAGE, PRIORITY, SEQUENCE, _HOSTNAME, _MACHINE_ID.
Use these fields for precise filtering in queries.

---

## 3) Basic usage — viewing logs

- Show all logs (oldest → newest):
  journalctl

- Show newest entries first:
  journalctl -r

- Show last N entries:
  journalctl -n 200

- Follow live logs (like tail -f):
  journalctl -f

Examples:
- View system boot logs:
  journalctl -b

- View logs for a specific boot (boot IDs from `journalctl --list-boots`):
  journalctl -b -1   # previous boot

- View logs for a specific unit (service):
  journalctl -u nginx.service
  Explanation: Filters entries to those with _SYSTEMD_UNIT=nginx.service.

---

## 4) Time filtering

- Since / until:
  journalctl --since "2025-01-01 10:00" --until "2025-01-01 12:00"
  Short forms:
  journalctl --since "1 hour ago"
  journalctl --since "today"

- Combine with unit:
  journalctl -u myapp.service --since "2025-10-01" --until "2025-10-02"

Examples:
- Show the last hour of logs for sshd:
  journalctl -u sshd.service --since "1 hour ago"

Notes:
- Time parsing is flexible; ISO 8601 works well.
- Use UTC or local time consistently when correlating multiple systems.

---

## 5) Priority (severity) filtering

- Priorities: 0 (emerg) → 7 (debug). Common names: emerg, alert, crit, err, warning, notice, info, debug.
- Show only errors and above:
  journalctl -p err
- Combine with unit:
  journalctl -u nginx -p warning --since "30 minutes ago"

Examples:
- Inspect errors for a unit:
  journalctl -u myapp -p err --since "1 hour ago"

---

## 6) Field-based filtering (powerful and structured)

You can filter by any field name:
- By PID:
  journalctl _PID=1234
- By executable:
  journalctl _EXE=/usr/bin/python3
- By user ID:
  journalctl _UID=1000
- By syslog identifier:
  journalctl SYSLOG_IDENTIFIER=sshd

Combine multiple filters (AND semantics):
  journalctl _SYSTEMD_UNIT=sshd.service _UID=0 --since "today"

Negation uses the `!` operator (systemd v239+):
  journalctl _SYSTEMD_UNIT=!cron.service

Examples:
- Find messages from a specific binary:
  journalctl _EXE=/usr/sbin/apache2 --since "yesterday"

---

## 7) Output formats

- short (default): human friendly
- short-iso, short-precise: include ISO timestamps
- verbose: full field list
- json, json-pretty, json-sse: machine-readable JSON
- export: binary export for backup/import

Examples:
- JSON output suitable for parsing:
  journalctl -u myapp -o json-pretty --since "1 hour ago" > myapp-logs.json

- Verbose (see all fields):
  journalctl -u myapp -o verbose -n 50

---

## 8) Paging and colors

- `journalctl` pipes through your pager (LESS) by default. Use:
  journalctl --no-pager   # disable paging
- Preserve colors:
  journalctl --no-pager --output=short-iso   # colors depend on environment

You can set environment variables for pager behavior:
  export SYSTEMD_LESS=FRXMK  # preserves colors and disables certain features

---

## 9) Boot analysis and boot-specific commands

- List boots:
  journalctl --list-boots
  Output example: each boot has an index like 0 (current), -1, -2, plus boot ID and timestamp.

- Show logs for a specific boot:
  journalctl -b -2   # two boots ago

- Show only messages from kernel during boot:
  journalctl -k -b

- Boot time analysis (systemd-analyze):
  systemd-analyze blame
  systemd-analyze critical-chain

Combine with journal:
  journalctl -b -u some-critical-service --no-pager

---

## 10) Exporting, saving and rotating logs

- Disk usage:
  journalctl --disk-usage

- Vacuum (delete old data by size/time):
  journalctl --vacuum-size=2G
  journalctl --vacuum-time=2weeks

- Export to a binary archive (portable):
  journalctl --output=export > /tmp/journal.export
  Restore on another host:
  systemd-journal-remote / some import tool (advanced workflows).

Best practice:
- Configure persistent journald (see next section) for production so logs survive reboots.

---

## 11) Configuring journald (persistent storage, limits, rate-limiting)

Configuration file:
/etc/systemd/journald.conf and drop-ins in /etc/systemd/journald.conf.d/
Key options:
- Storage=auto|persistent|volatile|none
  - persistent uses /var/log/journal (recommended for servers).
- SystemMaxUse=, SystemKeepFree=, SystemMaxFileSize=, SystemMaxFiles=
  - control how much disk the journal may use.
- MaxRetentionSec=, MaxFileSec= (newer systemd)
- RateLimitIntervalSec=, RateLimitBurst= (throttle noisy services)
- ForwardToSyslog= yes/no (forward messages to syslog daemon)
- Compress= yes/no (enable compression on journal files)

Example /etc/systemd/journald.conf (minimal):
  [Journal]
  Storage=persistent
  SystemMaxUse=2G
  RateLimitIntervalSec=30s
  RateLimitBurst=1000
  Compress=yes

After changes:
  sudo systemctl restart systemd-journald

Notes:
- Enabling persistent storage: create /var/log/journal (systemd will write).
  sudo mkdir -p /var/log/journal && sudo systemd-tmpfiles --create --prefix /var/log/journal

---

## 12) Permissions and security

- Journal files usually owned by root:adm or root:systemd-journal depending on distro.
- Grant read access without root: add user to `systemd-journal` or `adm` group:
  sudo usermod -aG systemd-journal alice
- Sensitive logs: remember that journals may contain secrets printed to logs; restrict access appropriately.

---

## 13) Troubleshooting scenarios (practical examples)

A. Service failing on start — get recent logs and errno:
  sudo systemctl start mysvc
  sudo journalctl -u mysvc -n 200 --no-pager
  sudo journalctl -u mysvc -p err --since "1 hour ago"

B. High disk usage due to journal:
  journalctl --disk-usage
  journalctl --vacuum-size=1G   # shrink to 1G
  # or edit journald.conf SystemMaxUse and restart journald

C. Find which unit produced a message:
  journalctl | rg "specific error text" -n 20
  # then view full fields
  journalctl -o verbose _SYSTEMD_UNIT=found.unit

D. Correlate logs across services by timestamp:
  journalctl --since "2025-10-29 10:12" --until "2025-10-29 10:15" -o short-iso

E. Track down kernel OOPS or panics:
  journalctl -k --since "10 minutes ago" -o short-precise

F. Capture logs for incident postmortem:
  sudo journalctl -u mysvc --since "2025-10-25" --until "2025-10-26" -o json-pretty > mysvc-logs.json

---

## 14) Advanced features

- Export to JSON for ingestion into ELK/Fluentd:
  journalctl -u mysvc -o json | jq -c '.'
  Use `journald` forwarders (systemd-journal-remote, journal-gatewayd) to ship logs.

- Remote reading and upload:
  - Tools: systemd-journal-remote (accepts uploads), journal-upload (client), journal-gatewayd (HTTP gateway).
  Example: Configure a central log server to receive uploaded journals from clients (see systemd-journal-remote docs).

- Using the Journal API in applications:
  - libsystemd provides sd-journal APIs for direct writes and queries.
  - Use `systemd-cat` to write to the journal:
    echo "hello" | systemd-cat -t myscript -p info

- Journal cursors:
  - Each entry has a cursor; store it to resume reading from that exact position.
  - Example:
    CURSOR=$(journalctl -u mysvc -n1 --show-cursor --no-pager | tail -n1 | sed -n 's/^-- cursor: //p')
    journalctl -c "$CURSOR" -f

- Fields and machine-readable selectors:
  - Use `_MACHINE_ID`, `_HOSTNAME` to filter across imported logs.
  - Example: logs for a specific container or machine:
    journalctl _MACHINE_ID=abcdef1234567890

- SELinux contexts & journal:
  - SELinux denials are logged in journal; use `ausearch` or filter message for AVC.
    journalctl | rg "avc:"

---

## 15) Performance & operational tips

- Avoid excessive verbosity in production; set appropriate log levels for services.
- Use rate-limiting in journald.conf to prevent log floods.
- Rotate and vacuum regularly, or enforce SystemMaxUse to prevent disk exhaustion.
- For high-volume logging, forward to a dedicated logging stack (Fluentd/Logstash/Vector) and limit local retention.
- Use structured logging in applications (JSON to stdout) and parse JSON in the logging pipeline; journal keeps the original MESSAGE plus structured fields.

---

## 16) Examples — from beginner to advanced (compact list)

- Beginner
  - View current logs:
    journalctl -n 100
  - Follow logs:
    journalctl -f

- Intermediate
  - Show last hour errors from nginx:
    journalctl -u nginx -p err --since "1 hour ago"
  - Export logs to JSON:
    journalctl -u myapp -o json-pretty --since "yesterday" > /tmp/myapp.json

- Advanced
  - Resume reading journal from saved cursor:
    CUR=$(journalctl -u myapp -n1 --show-cursor | sed -n 's/^-- cursor: //p')
    journalctl -c "$CUR" -f
  - Ship journals to a central server (high-level):
    # On central: run journal-gatewayd or systemd-journal-remote
    # On client: use journal-upload or configure remote forwarding
  - Use field-based extraction and jq transformation:
    journalctl -u myapp -o json | jq -r '.MESSAGE'

---

## 17) Common pitfalls

- Relying solely on stdout/stderr without structured fields makes correlation harder.
- Assuming journald logs are text — they are binary; use journalctl to read them.
- Not configuring persistent storage on servers — logs lost after reboot.
- Forgetting log rotation limits — journal may fill disk if not constrained.

---

## 18) Further reading and references

- `man journalctl`
- `man systemd-journald.service`
- `man journald.conf`
- systemd official documentation: https://www.freedesktop.org/wiki/Software/systemd/
- For programmatic access: sd-journal API docs.

---

End of journalctl guide.
