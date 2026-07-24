---
name: server-admin
description: 管理服务器（Squid, CentOS 8, SSH: OnePiece@43.226.44.151:2002）。处理用户管理、权限配置、系统监控等基础运维。
trigger: 用户询问服务器状态、管理用户和权限、查看系统资源、SSH 连接问题，或任何新服务器初始化任务。
---

## 服务器概览

### 当前活动服务器 (43.226.44.151)
- **主机名**: localhost.localdomain
- **操作系统**: CentOS 8 (内核 4.18.0-348.el8_5.x86_64)
- **SSH 别名**: Squid
- **SSH 端口**: 2002
- **SSH 密钥**: `~/.ssh/WorkComputer`（公司电脑，已配置免密登录）
- **用户**: root（密钥+密码 `Zhy26825.+8`）、OnePiece（uid=1000, wheel组, sudo NOPASSWD, 密钥免密）
- **资源**: 4x Intel Xeon E5-2690 v2 @ 3.00GHz, 3.6GB 内存, 17GB 磁盘 (约 14% 已用)
- **运行时间**: ~6 天

> **注意**: 此为新装空服务器，Docker 未安装，无任何业务服务。43.226.46.171 已停用，不再使用。

## SSH 连接信息

### 连接命令
```bash
# 密钥连接（推荐）
ssh -i $HOME/.ssh/WorkComputer -p 2002 root@43.226.44.151
ssh -i $HOME/.ssh/WorkComputer -p 2002 OnePiece@43.226.44.151

# 密码连接
ssh -o StrictHostKeyChecking=no -p 2002 OnePiece@43.226.44.151   # 首次需输入密码 Zhy26825.+8
```

### SSH 注意事项
- Windows 本地 `ssh` 可能被 `~/.sbx-denybin/ssh.bat` 拦截，若输出乱码使用完整路径。
- 已知 PQ KEX 警告：OpenSSH 版本较旧，提示 "store now, decrypt later" 风险，不影响使用。
- SSH 别名 `OnePiece` 指向 46.171，已失效。连接 151 需直接用 IP 或重新配置别名。

## 常用操作

### 快速健康检查
```bash
ssh -i $HOME/.ssh/WorkComputer -p 2002 root@43.226.44.151 'hostname; uptime; free -h | grep Mem; df -h /; ss -tlnp'
```

### 创建新用户
```bash
ssh -i $HOME/.ssh/WorkComputer -p 2002 root@43.226.44.151 '
useradd -m -s /bin/bash <用户名>
echo "<用户名>:<密码>" | chpasswd
mkdir -p /home/<用户名>/.ssh && cp /root/.ssh/authorized_keys /home/<用户名>/.ssh/authorized_keys
chown -R <用户名>:<用户名> /home/<用户名>/.ssh
chmod 700 /home/<用户名>/.ssh && chmod 600 /home/<用户名>/.ssh/authorized_keys
echo "<用户名> ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/<用户名>
chmod 440 /etc/sudoers.d/<用户名>
'
```

### 安装软件包
```bash
ssh -i $HOME/.ssh/WorkComputer -p 2002 root@43.226.44.151 'yum install -y <包名>'
```

### 修改 SSH 配置
```bash
ssh -i $HOME/.ssh/WorkComputer -p 2002 root@43.226.44.151 'sed -i "" /etc/ssh/sshd_config -e "s/^PermitRootLogin yes/PermitRootLogin no/" && systemctl restart sshd'
```

## 当前已配置用户
| 用户 | 角色 | 认证方式 | sudo |
|---|---|---|---|
| root | 管理员 | 密钥 + 密码 | N/A |
| OnePiece | 管理员 | 密钥 + 密码 | NOPASSWD: ALL |
