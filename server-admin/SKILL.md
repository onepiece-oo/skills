---
name: server-admin
description: Administer the production server (Squid, Ubuntu 24.04, SSH: OnePiece@43.226.46.171:2002). Handles Docker container management, service health checks, log inspection, config edits, and deployment tasks.
trigger: User asks to check server status, manage Docker containers, restart services, check logs, deploy SpringBoot apps, manage MySQL/Redis/Nginx/Nacos/RabbitMQ, or any server administration task.
---

## Server Overview
- **Hostname**: Squid
- **OS**: Ubuntu 24.04.4 LTS (Noble Numbat)
- **Kernel**: Linux 6.8.0-106-generic x86_64
- **SSH Alias**: OnePiece
- **SSH Port**: 2002
- **SSH Key**: ~/.ssh/id_ed25519
- **User**: OnePiece (uid=1001, sudo group, docker group)
- **Resources**: 4 CPU, 3.8GB RAM, 68GB disk (24% used)
- **Uptime**: ~97 days

## SSH Command Template
**Important**: Local `ssh` command may be intercepted by `~/.sbx-denybin/ssh.bat` wrapper on Windows. If you get garbled output like `off\r\nexit /b 1\r\n`, use the full path:
`C:\Windows\System32\OpenSSH\ssh.exe -i $HOME/.ssh/id_ed25519 -p 2002 OnePiece@43.226.46.171 '<command>'`
Or prefer: `ssh OnePiece '<command>'` if alias resolves cleanly.

## Docker Services

### Running Containers
| Container | Image | Ports | Status |
|---|---|---|---|
| dkd-backend | dkd:v1 | 8081->8080/tcp | Up 2 months |
| nginx | nginx:alpine | 80:80, 443:443 | Up 2 months |
| emqx-mqtt | emqx/emqx:5.8.3 | 1883, 18083 | Up (MQTT + Dashboard) |
| mysql8 | mysql:8.0 | 33066->3306/tcp | May be stopped (see below) |
| redis7 | redis:8.0-alpine | 6379:6379 | May be restarting |
| nacos-standalone | nacos/nacos-server:v3.0.3 | 8080, 8848, 9848, 9849 | May be stopped |
| rabbitmq3 | rabbitmq:3.13-management | 5672, 15672 | May be stopped |

### Docker Compose Projects
Compose files may not be in expected locations. Check:
- `~/docker` — may contain saved container listings (vim history recorded), NOT a compose file
- Look for compose files: `ssh OnePiece 'find ~ -name "docker-compose*" -not -path "*/node_modules/*" 2>/dev/null'`
- Previous known locations: `/mysql-compose/`, `/redis-compose/`, `/nginx-compose/`, `/rabbit-compose/`, `/nacos-docker/` (may not exist)

### Docker Commands
- List running: `ssh OnePiece 'docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'`
- List all: `ssh OnePiece 'docker ps -a --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'`
- Logs: `ssh OnePiece 'docker logs <container> --tail 50 2>&1'`
- Exec: `ssh OnePiece 'docker exec <container> <command>'`
- Restart: `ssh OnePiece 'docker restart <container>'`
- Stop: `ssh OnePiece 'docker stop <container>'`
- Start: `ssh OnePiece 'docker start <container>'` (useful for stopped containers that were started directly, not via compose)
- Compose up: `ssh OnePiece 'sudo docker compose -f <path> up -d'`
- Compose down: `ssh OnePiece 'sudo docker compose -f <path> down'`
- Images: `ssh OnePiece 'docker images -a --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"'`

### Container Management Notes
- **mysql8**: Was exited (137/OOM kill) 2 months ago. Use `docker start mysql8` to restart, not compose up.
- **redis7**: May be in restart loop. Check logs with `docker logs redis7`.
- **nacos-standalone**: May have been stopped. Use `docker start nacos-standalone`.
- **dkd-backend** and **nginx**: Independent containers (not managed by compose), have been running 2+ months.

## Service Credentials

### MySQL
- **Port**: 33066 (host) -> 3306 (container)
- **Root Password**: Zhy26825.+8
- **Nacos DB User**: OnePiece / Zhy26825.+8
- **Nacos DB Name**: sknacos
- **Business Databases**: `one`, `restaurant`, `sknacos`
- **Connect**: `ssh OnePiece 'docker exec mysql8 mysql -u root -p'"'"'Zhy26825.+8'"'"' -e "<sql>"'`
- Note: escape + in passwords for shell, use single quotes around password

### Redis
- **Port**: 6379
- **Password**: Zhy26825.+8
- **Config**: /redis-compose/redis.conf (keyspace notifications enabled, AOF yes)
- **Issue**: Redis container may be in restart loop - check logs with `docker logs redis7`
- **Audit Script**: /redis-compose/watch_redis_audit.sh (monitors SET/DEL/HSET etc.)

### Nacos
- **Ports**: 8848 (core), 9848-9849 (gRPC), 8080 (management)
- **Auth Token**: Ui0YwTlSg4dAE96B8lI1BVDs/0mOuFxPPITFHx+vdIQ=
- **Mode**: standalone
- **Data Source**: Direct JDBC to 43.226.46.171:33066/sknacos

### EMQX (MQTT Broker)
- **Compose**: `~/mqtt-compose/docker-compose.yml`
- **Env**: `~/mqtt-compose/.env` (hidden file)
- **Container**: emqx-mqtt (emqx/emqx:5.8.3)
- **Ports**: 1883 (MQTT TCP), 18083 (Dashboard HTTP)
- **Dashboard User**: admin / Admin@1234
- **Note**: Dashboard has no firewall protection — always access via SSH tunnel:
  `ssh -L 18083:localhost:18083 OnePiece@43.226.46.171`
- **Password reset** (EMQX enforces complexity: 8+ chars, mixed types):
  `ssh OnePiece 'docker exec emqx-mqtt /opt/emqx/bin/emqx ctl admins passwd admin <new_password>'`
- **Env var note**: `EMQX_DASHBOARD_DEFAULT_PASSWORD` only applies on first startup. If container already created users, use CLI reset instead.

### Nginx
- **Ports**: 80 (HTTP), 443 (HTTPS)
- **Proxy**: /prod-api/ and /stage-api/ -> http://43.226.46.171:8081/
- **Static Root**: /usr/share/nginx/html (from ./html volume)
- **Config**: /nginx-compose/nginx-conf/nginx.conf

### RabbitMQ
- **Ports**: 5672 (AMQP), 15672 (Management UI)
- **Credentials**: hutongsk / hutongsk123
- **Image**: rabbitmq:3.13-management

## SpringBoot App
- **JAR**: ~/springboot/dkd-admin.jar (~100MB)
- **Port**: 8081 (listening)
- **Version**: v3.8.7 (后台管理框架)
- **Dockerfile**: ~/springboot/Dockerfile (eclipse-temurin:11-jre-alpine)
- **Health**: Requires auth (401 on /info and /actuator/health)
- **Build**: No Maven/Gradle on server - build locally, deploy JAR

## System Services
- **fail2ban**: Running (no active jails detected via CLI)
- **postfix**: Running (SMTP on port 25)
- **containerd**: Running
- **docker**: Running
- **No crontab** for OnePiece user
- **No nginx/mysql/maven binaries** on host (all via Docker)
- **Java/Node**: Not installed on host (only in containers)
- **Python 3.12.3**: Available
- **Tools**: jq, tmux, htop, git, curl, wget, mysql client

## Common Operations

### Quick Health Check
```
ssh OnePiece 'echo "===CONTAINERS==="; docker ps --format "table {{.Names}}\t{{.Status}}"; echo "===PORTS==="; ss -tlnp | grep -E ":(80|443|33066|6379|8080|8081|8848|5672|15672|1883|18083) "; echo "===RESOURCES==="; free -h | grep Mem; df -h / | tail -1'
```

### Fix Redis Restart Loop
```
ssh OnePiece 'docker logs redis7 --tail 30'
ssh OnePiece 'docker stop redis7 && docker rm redis7'
ssh OnePiece 'sudo docker compose -f /redis-compose/docker-compose.yml up -d'
```

### Deploy SpringBoot App
```
# 1. Upload new JAR
scp -i $HOME/.ssh/id_ed25519 -P 2002 ./target/dkd-admin.jar OnePiece@43.226.46.171:~/springboot/dkd-admin.jar
# 2. Restart if needed
ssh OnePiece 'docker restart <app-container>'
```

### Check Disk Space
```
ssh OnePiece 'du -sh /var/lib/docker/overlay2 2>/dev/null; du -sh /root/mysql-compose/mysql-data 2>/dev/null; du -sh /root/redis-compose/redis-data 2>/dev/null'
```

### View Recent Logs
```
ssh OnePiece 'journalctl -u docker --no-pager -n 30'
ssh OnePiece 'docker logs nacos-standalone --tail 30'
ssh OnePiece 'docker logs mysql8 --tail 30'
ssh OnePiece 'tail -50 /root/nginx-compose/logs/access.log 2>/dev/null'
```

## Security Notes
- Redis is exposed on port 6379 to all interfaces - vulnerable to external attacks (seen brute force attempts in logs)
- MySQL port 33066 exposed publicly
- Nacos management port 8080 exposed
- No UFW firewall configured
- fail2ban has no active jails for SSH/postfix
- Consider restricting exposed ports or adding firewall rules

## Warnings
- **Do NOT modify** Redis password or MySQL password in configs without user confirmation
- **Do NOT run** `docker system prune` without asking - may remove needed images
- **Do NOT** restart nacos-standalone or mysql8 without confirming no impact
- Nacos auth is enabled - API calls need proper token
- RabbitMQ and Nginx compose projects exist but containers may not be running
- **Container state changes**: Containers may be started/stopped outside of compose. Always check `docker ps -a` before assuming compose-managed state.
