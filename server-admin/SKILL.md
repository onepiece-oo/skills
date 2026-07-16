---
name: server-admin
description: 管理生产服务器（Squid, Ubuntu 24.04, SSH: OnePiece@43.226.46.171:2002）。处理 Docker 容器管理、服务健康检查、日志查看、配置编辑和部署任务。
trigger: 用户询问服务器状态、管理 Docker 容器、重启服务、查看日志、部署 SpringBoot 应用、管理 MySQL/Redis/Nginx/Nacos/RabbitMQ，或任何服务器管理任务。
---

## 服务器概览
- **主机名**: Squid
- **操作系统**: Ubuntu 24.04.4 LTS (Noble Numbat)
- **内核**: Linux 6.8.0-106-generic x86_64
- **SSH 别名**: OnePiece
- **SSH 端口**: 2002
- **SSH 密钥**: ~/.ssh/id_ed25519（个人电脑）/ ~/.ssh/WorkComputer（公司电脑）
- **用户**: OnePiece (uid=1001, sudo 组, docker 组)
- **资源**: 4 CPU, 3.8GB 内存, 68GB 磁盘 (约 30% 已用)
- **运行时间**: ~100 天

## SSH 连接信息

### 个人电脑
- **SSH 密钥**: `~/.ssh/id_ed25519`
- **连接命令**: `ssh OnePiece '<command>'` 或 `ssh -i $HOME/.ssh/id_ed25519 -p 2002 OnePiece@43.226.46.171 '<command>'`

### 公司电脑
- **SSH 密钥**: `~/.ssh/WorkComputer`
- **连接命令**: `ssh -i $HOME/.ssh/WorkComputer -p 2002 OnePiece@43.226.46.171 '<command>'`

## SSH 命令模板
**注意**: Windows 本地 `ssh` 可能被 `~/.sbx-denybin/ssh.bat` 拦截。如果输出乱码如 `off\r\nexit /b 1\r\n`，使用完整路径：
`C:\Windows\System32\OpenSSH\ssh.exe -i $HOME/.ssh/id_ed25519 -p 2002 OnePiece@43.226.46.171 '<command>'`
或优先使用：`ssh OnePiece '<command>'`（别名能正常解析时）。

## Docker 容器

### 容器列表
| 容器 | 镜像 | 端口 | 状态 | 管理方式 |
|---|---|---|---|---|
| dkd-backend | dkd:v1 | 8081->8080/tcp | 独立容器，非 compose 管理 | `docker restart dkd-backend` |
| emqx-mqtt | emqx/emqx:5.8.3 | 1883, 18083 | `~/mqtt-compose/` | compose |
| mysql8 | mysql:8.0 | 33066->3306/tcp | `/home/OnePiece/docker-compose/mysql-redis-compose/` | 与 redis7 共用 compose |
| redis7 | redis:8.0-alpine | 6379:6379 | 同上 | 同上 |
| nginx | nginx:alpine | 80:80, 443:443 | `/home/OnePiece/docker-compose/nginx-compose/` | compose |
| nacos-standalone | nacos/nacos-server:v3.0.3 | 8080, 8848, 9848, 9849 | `/home/OnePiece/docker-compose/nacos-docker/` | compose |
| rabbitmq3 | rabbitmq:3.13-management | 5672, 15672 | `/home/OnePiece/docker-compose/rabbit-compose/` | compose |

### Docker 常用命令
- 查看运行中：`ssh OnePiece 'docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'`
- 查看所有：`ssh OnePiece 'docker ps -a --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'`
- 查看日志：`ssh OnePiece 'docker logs <容器名> --tail 50 2>&1'`
- 进入容器：`ssh OnePiece 'docker exec -it <容器名> sh'`
- 重启：`ssh OnePiece 'docker restart <容器名>'`
- 停止：`ssh OnePiece 'docker stop <容器名>'`
- 启动：`ssh OnePiece 'docker start <容器名>'`（独立启动的容器用此方式）
- Compose 启动：`ssh OnePiece 'cd /home/OnePiece/docker-compose/<项目目录> && sudo docker compose up -d'`
- Compose 停止：`ssh OnePiece 'cd /home/OnePiece/docker-compose/<项目目录> && sudo docker compose down'`
- 镜像列表：`ssh OnePiece 'docker images -a --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"'`

### 容器管理注意事项
- **dkd-backend** 和 **nginx** 是独立容器（非 compose 管理），直接通过 `docker start/stop/restart` 控制。
- 所有 compose 项目统一存放在 `/home/OnePiece/docker-compose/` 下。
- 旧 `/mysql-compose/`、`/redis-compose/` 目录已删除，数据已迁移。

## 服务凭证

### MySQL
- **端口**: 33066 (宿主机) -> 3306 (容器)
- **Root 密码**: Zhy26825.+8（仅限 localhost 连接）
- **业务用户**: onepiece / Zhy26825.+8（仅限 localhost 连接）
- **Nacos DB**: OnePiece / Zhy26825.+8 / sknacos
- **业务数据库**: `one`, `restaurant`, `sknacos`
- **安全变更**: root 用户和 OnePiece 用户的 `%` 主机权限已删除，禁止远程登录
- **连接命令**: `ssh OnePiece 'docker exec mysql8 mysql -u root -p'"'"'Zhy26825.+8'"'"' -e "<SQL>"'`
- **注意**: shell 中转义 + 号，密码用单引号包裹

### Redis
- **端口**: 6379
- **密码**: Zhy26825.+8（requirepass，兼容旧客户端）
- **ACL 用户**: onepiece / Zhy26825.+8（拥有全部权限和所有 key 访问）
- **配置**: `/home/OnePiece/docker-compose/mysql-redis-compose/redis.conf`（AOF 开启）
- **ACL 持久化**: `/home/OnePiece/docker-compose/mysql-redis-compose/users.acl`
- **镜像**: redis:8.0-alpine，加载了 5 个模块（Search, Bloom, TimeSeries, VectorSet, ReJSON）
- **已知问题**: 内存碎片率 ~15（used_memory 1.1MB 但 RSS 17MB），端口公网暴露曾遭暴力破解
- **连接 onepiece 用户**: `redis-cli -u redis://onepiece:Zhy26825.+8@127.0.0.1:6379`

### Nacos
- **端口**: 8848 (核心), 9848-9849 (gRPC), 8080 (管理)
- **认证 Token**: Ui0YwTlSg4dAE96B8lI1BVDs/0mOuFxPPITFHx+vdIQ=
- **模式**: standalone
- **数据源**: 直连 JDBC 到 43.226.46.171:33066/sknacos

### EMQX (MQTT 代理)
- **Compose**: `~/mqtt-compose/docker-compose.yml`
- **环境变量**: `~/mqtt-compose/.env`（隐藏文件）
- **容器**: emqx-mqtt (emqx/emqx:5.8.3)
- **端口**: 1883 (MQTT TCP), 18083 (Dashboard HTTP)
- **Dashboard 账号**: admin / Admin@1234
- **注意**: Dashboard 无防火墙保护 — 必须通过 SSH 隧道访问：
  `ssh -L 18083:localhost:18083 OnePiece@43.226.46.171`
- **密码重置**（EMQX 强制复杂度：8+ 字符，混合类型）：
  `ssh OnePiece 'docker exec emqx-mqtt /opt/emqx/bin/emqx ctl admins passwd admin <新密码>'`
- **环境变量说明**: `EMQX_DASHBOARD_DEFAULT_PASSWORD` 仅在首次启动时生效。如果容器已创建过用户，需用 CLI 重置。

### Nginx
- **端口**: 80 (HTTP), 443 (HTTPS)
- **代理**: /prod-api/ 和 /stage-api/ -> http://43.226.46.171:8081/
- **静态文件根目录**: /usr/share/nginx/html（来自 ./html 卷）
- **配置**: `/home/OnePiece/docker-compose/nginx-compose/nginx-conf/nginx.conf`

### RabbitMQ
- **端口**: 5672 (AMQP), 15672 (Management UI)
- **账号**: hutongsk / hutongsk123
- **镜像**: rabbitmq:3.13-management

## SpringBoot 应用
- **JAR**: ~/springboot/dkd-admin.jar (~100MB)
- **端口**: 8081（监听中）
- **版本**: v3.8.7（后台管理框架）
- **Dockerfile**: ~/springboot/Dockerfile (eclipse-temurin:11-jre-alpine)
- **健康检查**: 需要认证（/info 和 /actuator/health 返回 401）
- **构建**: 服务器无 Maven/Gradle — 本地构建后部署 JAR

## 系统服务
- **fail2ban**: 运行中（无活跃 jail）
- **postfix**: 运行中（SMTP 端口 25）
- **containerd**: 运行中
- **docker**: 运行中
- **OnePiece 用户无 crontab**
- **宿主机无 nginx/mysql/maven 二进制**（全部通过 Docker）
- **Java/Node**: 宿主机未安装（仅在容器中）
- **Python 3.12.3**: 可用
- **工具**: jq, tmux, htop, git, curl, wget, mysql client

## 常用操作

### 快速健康检查
```
ssh OnePiece 'echo "===容器==="; docker ps --format "table {{.Names}}\t{{.Status}}"; echo "===端口==="; ss -tlnp | grep -E ":(80|443|33066|6379|8080|8081|8848|5672|15672|18083) "; echo "===资源==="; free -h | grep Mem; df -h / | tail -1'
```

### 部署 SpringBoot 应用
```bash
# 1. 上传新 JAR
scp -i $HOME/.ssh/id_ed25519 -P 2002 ./target/dkd-admin.jar OnePiece@43.226.46.171:~/springboot/dkd-admin.jar
# 2. 重启容器
ssh OnePiece 'docker restart dkd-backend'
```

### 查看磁盘空间
```
ssh OnePiece 'du -sh /var/lib/docker/overlay2 2>/dev/null; du -sh /home/OnePiece/docker-compose/*/mysql-data 2>/dev/null; du -sh /home/OnePiece/docker-compose/*/redis-data 2>/dev/null'
```

### 查看近期日志
```
ssh OnePiece 'journalctl -u docker --no-pager -n 30'
ssh OnePiece 'docker logs nacos-standalone --tail 30'
ssh OnePiece 'docker logs mysql8 --tail 30'
ssh OnePiece 'tail -50 /home/OnePiece/docker-compose/nginx-compose/logs/access.log 2>/dev/null'
```

### 迁移 Docker Compose 项目
所有 compose 项目统一存放在 `/home/OnePiece/docker-compose/` 下。迁移步骤：
1. 停止相关容器：`docker stop <容器名>`
2. 移动目录：`sudo mv /旧路径/<项目> /home/OnePiece/docker-compose/`
3. 修正权限：`sudo chown -R OnePiece:OnePiece /home/OnePiece/docker-compose/<项目>`
4. 启动：`cd /home/OnePiece/docker-compose/<项目> && sudo docker compose up -d`
5. 验证：`docker ps` 检查容器状态

### MySQL 用户管理
```bash
# 创建新用户
ssh OnePiece 'docker exec mysql8 mysql -u root -p'"'"'Zhy26825.+8'"'"' -e "CREATE USER '\''<用户名>'\''@'\''localhost'\'' IDENTIFIED BY '\''<密码>'\''; GRANT ALL PRIVILEGES ON *.* TO '\''<用户名>'\''@'\''localhost'\''; DROP USER IF EXISTS '\''<用户名>'\''@'\''%'\''; FLUSH PRIVILEGES;"'

# 查看所有用户及主机
ssh OnePiece 'docker exec mysql8 mysql -u root -p'"'"'Zhy26825.+8'"'"' -e "SELECT user, host FROM mysql.user;"'

# 禁止 root 远程登录
ssh OnePiece 'docker exec mysql8 mysql -u root -p'"'"'Zhy26825.+8'"'"' -e "DROP USER IF EXISTS '\''root'\''@'\''%'\''; DROP USER IF EXISTS '\'''\''@'\''%'\''; FLUSH PRIVILEGES;"'
```

### Redis ACL 管理
```bash
# 添加 ACL 用户
ssh OnePiece 'docker exec redis7 redis-cli -a Zhy26825.+8 ACL SETUSER <用户名> on ">密码" ~* +@all'

# 查看用户
ssh OnePiece 'docker exec redis7 redis-cli -a Zhy26825.+8 ACL GETUSER <用户名>'

# 测试登录
ssh OnePiece 'docker exec redis7 redis-cli -u redis://<用户名>:<密码>@127.0.0.1:6379 ping'

# ACL 持久化：在 redis.conf 中添加 aclfile /usr/local/etc/redis/users.acl，并在 compose 中挂载 ./users.acl
```

## 安全注意事项
- Redis 6379 端口公网暴露 — 曾遭受暴力破解攻击
- MySQL 33066 端口公网暴露 — root 和 onepiece 用户已限制为仅 localhost 连接
- Nacos 8080 管理端口暴露
- 未配置 UFW 防火墙
- fail2ban 无活跃 jail（SSH/postfix）
- 建议：考虑限制暴露端口或添加防火墙规则

## 警告
- **不要擅自修改** Redis 或 MySQL 密码，需先确认
- **不要执行** `docker system prune`，可能删除需要的镜像
- **不要随意重启** nacos-standalone 或 mysql8，需确认不影响其他服务
- Nacos 已启用认证 — API 调用需要正确 token
- **容器状态变化**：容器可能在 compose 之外被启动/停止。操作前始终检查 `docker ps -a`
- 根目录 `/` 应保持整洁，所有 compose 项目在 `/home/OnePiece/docker-compose/` 下
- `/home/OnePiece/` 下不再有 `mysql-compose/`, `redis-compose/`, `rabbit-compose/`, `mqtt-compose/` 等散落目录
