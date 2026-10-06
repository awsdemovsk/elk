# 05 – Linux Server Logs

## What to collect
| File | Distro | Purpose |
|---|---|---|
| `/var/log/syslog` | Debian/Ubuntu | General system messages |
| `/var/log/messages` | RHEL/CentOS/Amazon Linux | General system messages |
| `/var/log/auth.log` | Debian/Ubuntu | SSH, sudo, logins |
| `/var/log/secure` | RHEL family | SSH, sudo, logins |
| `/var/log/nginx/access.log` | any | Web traffic |
| `/var/log/nginx/error.log` | any | Web errors |

Also consider: `journalctl` (use `type: journald` input), `/var/log/kern.log`, `/var/log/audit/audit.log`.

## Install Filebeat (Ubuntu)
```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.19.0-amd64.deb
sudo dpkg -i filebeat-8.19.0-amd64.deb
sudo cp filebeat/linux/filebeat-system.yml /etc/filebeat/filebeat.yml
sudo filebeat test config
sudo filebeat test output
sudo systemctl enable --now filebeat
sudo journalctl -u filebeat -f
```

## Config
See [`filebeat/linux/filebeat-system.yml`](../filebeat/linux/filebeat-system.yml) and the modules variant [`modules-nginx-system.yml`](../filebeat/linux/modules-nginx-system.yml).

Using modules (gives parsed fields + dashboards):
```bash
sudo filebeat modules enable system nginx
sudo filebeat setup -e
```

## Journald alternative
```yaml
filebeat.inputs:
  - type: journald
    id: journald
```

## Permissions
Filebeat runs as root via systemd, so `auth.log` is readable. In containers, mount `/var/log` read-only.

## Generate test events
```bash
logger "elk-test message"
ssh invalid@localhost        # failed auth
sudo -k; sudo ls             # sudo events
```
