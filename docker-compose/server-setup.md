# 服务器初始化配置记录

## 基本信息

| 项目 | 值 |
|------|-----|
| 服务器 IP | 43.226.44.151 |
| SSH 端口 | 2002 |
| SSH 用户 | OnePiece |
| 操作日期 | 2026-07-27 |

---

## 1. 创建系统用户

```bash
useradd -m -s /bin/bash OnePiece
echo "OnePiece:Zhy26825.+8" | chpasswd
```

## 2. 配置 sudo 免密权限

```bash
echo "OnePiece ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/OnePiece
chmod 0440 /etc/sudoers.d/OnePiece
```

## 3. 放行端口（修改 SSH 端口前必须先执行）



### 服务器防火墙
```bash
# firewalld — 修改 SSH 端口前先放行新端口
firewall-cmd --add-port=2002/tcp --permanent && firewall-cmd --reload
```

> **重要：必须在修改 SSH 端口之前放行 2002 端口，否则当前连接会中断且无法重连。**

## 4. 修改 SSH 端口为 2002

```bash
sed -i 's/^#\?Port.*/Port 2002/' /etc/ssh/sshd_config
systemctl restart sshd
```

## 5. 禁止 root 远程登录

```bash
sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd
```

## 6. 安装 Docker

### 安装 Docker

```bash
bash <(curl -sSL https://linuxmirrors.cn/docker.sh)
```

```

### 配置 Docker

```bash
sudo systemctl enable docker && sudo systemctl start docker
sudo usermod -aG docker OnePiece
```

验证：

```bash
docker --version
docker run hello-world
```

## 7. 配置 Docker 使用权限

```bash
groupadd docker              # 若已存在会提示
usermod -aG docker OnePiece
sudo systemctl restart docker
```

## 8. 配置 SSH 免密登录

### 本地公钥
- 私钥：`C:\Users\Administrator\.ssh\WorkComputer`
- 公钥：`ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICHoRZMVgHok3fX5kJGQxqnV2im5OlBPSDF4pQdk+f4W OnePiece-WorkComputer`

### 服务器配置
```bash
mkdir -p ~/.ssh
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICHoRZMVgHok3fX5kJGQxqnV2im5OlBPSDF4pQdk+f4W OnePiece-WorkComputer' >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 验证连接
```bash
ssh -p 2002 -i ~/.ssh/WorkComputer OnePiece@43.226.44.151
```

## 9. 修改主机名

```bash
sudo hostnamectl set-hostname Squid
```

## 10. 配置 Fail2ban

### 防火墙放行 SSH 端口（安装前必须执行）

```bash
# fail2ban 启动后会封禁所有未白名单的 IP
# 必须在安装/启动前放行 2002 端口，否则会被自己封禁
sudo firewall-cmd --permanent --add-port=2002/tcp && sudo firewall-cmd --reload
```

### 安装

CentOS 8 已 EOL，基础仓库无 fail2ban。通过下载 RPM 包强制安装（跳过依赖检查）：

```bash
cd /tmp
mkdir -p /tmp/fail2ban
cd /tmp/fail2ban
sudo dnf download fail2ban fail2ban-server fail2ban-firewalld fail2ban-systemd --destdir=/tmp/fail2ban
sudo rpm -ivh --nodeps *.rpm
```

### 配置

创建 `/etc/fail2ban/jail.local`：

```ini
[DEFAULT]
bantime = 3600
findtime = 300
maxretry = 5
ignoreip = 127.0.0.1/8 ::1
banaction = firewallcmd-ipset
backend = systemd

[sshd]
enabled = true
port = 2002
filter = sshd
logpath = /var/log/secure
```

启用并启动：

```bash
sudo systemctl enable fail2ban && sudo systemctl start fail2ban
```

查看状态：

```bash
sudo systemctl status fail2ban
sudo fail2ban-client status sshd
```

---

## 后续可用连接命令

```bash
ssh -p 2002 -i ~/.ssh/WorkComputer OnePiece@43.226.44.151
```

## 注意事项

- 修改 SSH 端口前，**务必**先在云安全组和防火墙放行对应端口，否则连接会永久丢失
- Fail2ban 安装并启动前，**务必**先通过 `firewall-cmd` 放行 2002 端口，否则会被自己封禁
- 禁用了 root 远程登录后，如需 root 权限可通过 OnePiece 用户 sudo 切换
- Docker 用户组变更后需重新登录生效
- CentOS 8 已于 2021 年 EOL，建议后续迁移至 Rocky Linux 9 / AlmaLinux 9
