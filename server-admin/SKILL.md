---
name: server-admin
description: 管理服务器（Squid, CentOS 8, SSH: OnePiece@43.226.44.151:2002）。处理用户管理、权限配置、系统监控、Docker 部署等基础运维。
trigger: 用户询问服务器状态、管理用户和权限、查看系统资源、SSH 连接问题、Docker 部署或任何服务器维护任务。
---

## 服务器概览

### 当前活动服务器 (43.226.44.151) — Squid
- **主机名**: localhost.localdomain  
- **操作系统**: CentOS 8 (内核 4.18.0-348.el8_5.x86_64) ⚠️ EOL 2021-12，建议迁移至 Rocky Linux 9
- **SSH 别名**: `Squid`
- **SSH 端口**: 2002
- **SSH 密钥**: `~/.ssh/WorkComputer`（公司电脑，已配置免密登录）
- **用户**: root（密钥+密码 `Zhy26825.+8`）、OnePiece（uid=1000, wheel组, sudo NOPASSWD, 密钥免密）
- **资源**: 4x Intel Xeon E5-2690 v2 @ 3.00GHz, 3.6GB 内存, ~14% 磁盘已用
- **运行时间**: ~6 天

> **注意**: 43.226.46.171（SSH 别名 OnePiece）已停用，不再使用。

## SSH 连接信息

### 连接命令（推荐密钥认证）
```bash
# OnePiece 用户（sudo NOPASSWD）
ssh -i ~/.ssh/WorkComputer -p 2002 OnePiece@43.226.44.151

# root 用户
ssh -i ~/.ssh/WorkComputer -p 2002 root@43.226.44.151
```

### 密码连接（首次使用）
```bash
ssh -o StrictHostKeyChecking=no -p 2002 OnePiece@43.226.44.151
# 输入密码: Zhy26825.+8
```

### SSH 注意事项
- Windows 本地 `ssh` 可能被 `~/.sbx-denybin/ssh.bat` 拦截，若输出乱码使用完整路径
- 已知 PQ KEX 警告：OpenSSH 版本较旧，提示 "store now, decrypt later" 风险，不影响正常使用
- SSH 别名 `OnePiece` 指向已停用的 46.171，**不能用于连接 43.226.44.151**，请直接用 IP 连接

## 常用操作

### 快速健康检查
```bash
ssh -i ~/.ssh/WorkComputer -p 2002 OnePiece@43.226.44.151 '
hostname; uptime; free -h | grep Mem; df -h /; ss -tlnp
'
```

### 创建新用户
```bash
ssh -i ~/.ssh/WorkComputer -p 2002 OnePiece@43.226.44.151 'sudo useradd -m -s /bin/bash <用户名>
echo "<用户名>:<密码>" | chpasswd
mkdir -p /home/<用户名>/.ssh && cp /root/.ssh/authorized_keys /home/<用户名>/.ssh/authorized_keys
chown -R <用户名>:<用户名> /home/<用户名>/.ssh
chmod 700 /home/<用户名>/.ssh && chmod 600 /home/<用户名>/.ssh/authorized_keys
echo "<用户名> ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/<用户名>
chmod 440 /etc/sudoers.d/<用户名>'
```

### 修改 SSH 配置（示例：禁止 root 登录）
```bash
ssh -i ~/.ssh/WorkComputer -p 2002 OnePiece@43.226.44.151 'sudo sed -i "s/^#\?PermitRootLogin.*/PermitRootLogin no/" /etc/ssh/sshd_config && sudo systemctl restart sshd'
```

## Docker 相关（按需求补充）

### 安装 Docker（CentOS 8 依赖冲突时）
参考 `docs/docker-compose/server-setup.md` 中的完整安装流程，或联系作者获取适配脚本。

### Docker Compose 部署示例：Redis
```bash
# 1. 创建目录结构
mkdir -p ~/docker-compose/redis/redis-data

# 2. 写入 docker-compose.yml
cat > ~/docker-compose/redis/docker-compose.yml << 'EOF'
version: "3.8"
services:
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: ["redis-server", "--requirepass", "YOUR_PASSWORD", "--appendonly", "yes"]
    ports:
      - "6379:6379"
    volumes:
      - ./redis-data:/data
networks:
  default:
    driver: bridge
EOF

# 3. 启动服务（OnePiece 有 NOPASSWD）
cd ~/docker-compose/redis && sudo docker compose up -d

# 4. 检查状态
sudo docker ps -f name=redis

# 5. 放行防火墙端口
sudo firewall-cmd --permanent --add-port=6379/tcp && sudo firewall-cmd --reload

# 6. 连接测试
sudo docker exec -it redis redis-cli -a YOUR_PASSWORD PONG
# 返回: PONG
```

**注意事项：** 密码含特殊字符（如 `+`、`.`）时，在 shell 中需适当转义或使用引号保护；生产环境建议使用外部 Vault 管理密码而非硬编码。

## 当前已配置用户

| 用户 | 角色 | 认证方式 | sudo |
|------|------|----------|------|
| root | 管理员 | 密钥 + 密码 (`Zhy26825.+8`) | N/A |
| OnePiece | 管理员 | 密钥 + 密码 | NOPASSWD: ALL |
