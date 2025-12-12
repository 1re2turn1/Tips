# Linux 常见问题 Linux Common Pitfalls

记录使用Linux系统时容易遇到的问题和解决方案。

## 1. 权限问题 Permission Issues

### 常见权限错误
```bash
# 错误: Permission denied
# 原因: 没有执行权限或访问权限
```

### 解决方案
```bash
# 给文件添加执行权限
chmod +x script.sh

# 修改文件所有者
sudo chown user:group file.txt

# 修改目录及其内容的权限
chmod -R 755 /path/to/directory

# 理解权限数字
# 4 = 读 (r)
# 2 = 写 (w)
# 1 = 执行 (x)
# 7 = 4+2+1 = rwx
# 6 = 4+2 = rw-
# 5 = 4+1 = r-x
```

### 查看和理解权限
```bash
# 列出详细权限
ls -l file.txt
# -rw-r--r-- 1 user group 1234 Dec 12 10:00 file.txt
# 第一个字符: 文件类型 (- 文件, d 目录, l 链接)
# 接下来9个字符: 权限 (所有者 组 其他人)

# 查看目录权限
ls -ld /path/to/directory
```

### ⚠️ 避免使用777权限
```bash
# 危险！给所有人完全权限
chmod 777 file.txt  # 不推荐

# 更好的做法：只给需要的权限
chmod 644 file.txt  # 所有者可读写，其他人只读
chmod 755 script.sh  # 所有者可读写执行，其他人可读执行
```

## 2. 磁盘空间问题 Disk Space Issues

### 检查磁盘使用情况
```bash
# 查看磁盘使用情况
df -h

# 查看目录大小
du -sh /path/to/directory

# 查找大文件
find / -type f -size +100M 2>/dev/null | xargs ls -lh

# 查看当前目录下各子目录大小
du -h --max-depth=1 | sort -hr

# 查找最大的10个文件
find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -n 10
```

### 清理空间
```bash
# 清理APT缓存 (Ubuntu/Debian)
sudo apt-get clean
sudo apt-get autoclean
sudo apt-get autoremove

# 清理journald日志
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=100M

# 清理临时文件
sudo rm -rf /tmp/*
sudo rm -rf /var/tmp/*

# 清理旧内核 (Ubuntu)
sudo apt-get autoremove --purge
```

## 3. 进程管理 Process Management

### 查找和终止进程
```bash
# 查找进程
ps aux | grep process_name
pgrep process_name
pidof process_name

# 查看进程树
pstree -p

# 终止进程
kill <PID>
kill -9 <PID>  # 强制终止
killall process_name
pkill process_name

# 查看占用端口的进程
sudo lsof -i :8080
sudo netstat -tulpn | grep 8080
sudo ss -tulpn | grep 8080
```

### 后台运行和管理
```bash
# 后台运行命令
command &

# 将前台进程切换到后台
# Ctrl+Z (暂停)
bg  # 继续在后台运行

# 查看后台任务
jobs

# 将后台任务切换到前台
fg %1

# 使进程在退出终端后继续运行
nohup command > output.log 2>&1 &

# 使用screen或tmux
screen -S session_name
# Ctrl+A, D (分离会话)
screen -r session_name  # 重新连接
```

## 4. 网络问题 Network Issues

### 网络诊断
```bash
# 测试网络连接
ping google.com
ping -c 4 8.8.8.8  # 只ping 4次

# 追踪路由
traceroute google.com
mtr google.com  # 更好的traceroute

# 测试端口连接
telnet host 80
nc -zv host 80

# 查看网络接口
ip addr
ifconfig  # 老命令

# 查看路由表
ip route
route -n

# 查看网络统计
netstat -s
ss -s
```

### 刷新DNS缓存
```bash
# 测试DNS解析
nslookup google.com
dig google.com
host google.com

# 刷新DNS缓存
# Ubuntu 18.04+ (推荐使用resolvectl)
sudo resolvectl flush-caches
# 或旧命令
sudo systemd-resolve --flush-caches

# 查看DNS配置
cat /etc/resolv.conf

# 临时更改DNS
sudo nano /etc/resolv.conf
# 添加
# nameserver 8.8.8.8
# nameserver 8.8.4.4
```

### 防火墙问题
```bash
# UFW (Ubuntu)
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw enable

# iptables
sudo iptables -L
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# firewalld (CentOS/RHEL)
sudo firewall-cmd --state
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload
```

## 5. 软件包管理 Package Management

### APT常见问题 (Ubuntu/Debian)
```bash
# 修复损坏的包
sudo apt-get install -f
sudo dpkg --configure -a

# 更新包列表
sudo apt-get update

# 升级所有包
sudo apt-get upgrade
sudo apt-get dist-upgrade

# 搜索包
apt-cache search package_name
apt search package_name

# 查看包信息
apt-cache show package_name
apt show package_name

# 清理
sudo apt-get autoremove
sudo apt-get autoclean
```

### 锁定文件问题
```bash
# 错误: Could not get lock /var/lib/dpkg/lock-frontend

# 查找占用的进程
sudo lsof /var/lib/dpkg/lock-frontend

# 等待其他包管理器完成，或强制删除锁文件（谨慎！）
sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock*

# 重新配置
sudo dpkg --configure -a
```

## 6. 文件系统问题 Filesystem Issues

### 文件系统检查和修复
```bash
# 检查磁盘错误（需要卸载或在单用户模式）
sudo fsck /dev/sda1

# 强制检查（下次重启时）
sudo touch /forcefsck

# 查看文件系统类型
df -T
lsblk -f
```

### 挂载问题
```bash
# 查看已挂载的文件系统
mount
df -h

# 挂载设备
sudo mount /dev/sdb1 /mnt/usb

# 卸载设备
sudo umount /mnt/usb

# 如果设备繁忙
lsof +f -- /mnt/usb
sudo umount -l /mnt/usb  # 懒卸载

# 自动挂载配置
sudo nano /etc/fstab
# 示例：
# /dev/sdb1 /mnt/data ext4 defaults 0 2
```

## 7. 环境变量问题 Environment Variables

### 设置环境变量
```bash
# 临时设置（当前会话）
export PATH=$PATH:/new/path
export MY_VAR="value"

# 永久设置（当前用户）
echo 'export PATH=$PATH:/new/path' >> ~/.bashrc
source ~/.bashrc

# 系统级别
sudo nano /etc/environment
# 或
sudo nano /etc/profile.d/custom.sh
```

### 查看环境变量
```bash
# 查看所有环境变量
printenv
env

# 查看特定变量
echo $PATH
echo $HOME
printenv PATH
```

### PATH问题
```bash
# 临时添加到PATH
export PATH=$PATH:/usr/local/bin

# 查看PATH
echo $PATH

# 恢复默认PATH（如果搞乱了）
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

## 8. SSH问题 SSH Issues

### SSH连接问题
```bash
# 详细输出调试信息
ssh -vvv user@host

# 常见问题：权限太开放
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys

# 生成SSH密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 复制公钥到服务器
ssh-copy-id user@host
# 或手动
cat ~/.ssh/id_rsa.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### SSH配置
```bash
# 编辑SSH配置
nano ~/.ssh/config

# 示例配置
Host myserver
    HostName 192.168.1.100
    User username
    Port 22
    IdentityFile ~/.ssh/id_rsa
    ServerAliveInterval 60

# 使用
ssh myserver
```

## 9. 系统性能问题 System Performance

### 监控系统资源
```bash
# CPU和内存使用
top
htop  # 更友好的界面

# 实时监控
watch -n 1 free -h

# 查看系统负载
uptime
w

# 查看内存使用
free -h
vmstat 1

# 查看IO使用
iostat
iotop  # 需要安装

# 查看网络使用
iftop
nethogs
```

### 性能分析
```bash
# 查找CPU使用率高的进程
ps aux --sort=-%cpu | head -n 10

# 查找内存使用率高的进程
ps aux --sort=-%mem | head -n 10

# 系统调用跟踪
strace -p <PID>

# 性能分析
perf top
```

## 10. 日志和调试 Logging and Debugging

### 查看系统日志
```bash
# 系统日志
sudo journalctl

# 最近的日志
sudo journalctl -n 50

# 实时查看日志
sudo journalctl -f

# 特定服务的日志
sudo journalctl -u nginx
sudo journalctl -u ssh

# 特定时间范围
sudo journalctl --since "2024-01-01" --until "2024-01-02"
sudo journalctl --since "1 hour ago"

# 传统日志文件
tail -f /var/log/syslog
tail -f /var/log/auth.log
```

### 应用日志
```bash
# Apache
tail -f /var/log/apache2/error.log

# Nginx
tail -f /var/log/nginx/error.log

# 自定义日志
tail -f /var/log/myapp/app.log

# 搜索日志
grep "error" /var/log/syslog
grep -i "failed" /var/log/auth.log
```

## 最佳实践 Best Practices

1. **定期备份重要数据**
```bash
# 使用rsync备份
rsync -avz --delete /source/ /backup/

# 使用tar打包
tar -czf backup-$(date +%Y%m%d).tar.gz /path/to/data
```

2. **使用别名提高效率**
```bash
# 在~/.bashrc中添加
alias ll='ls -lah'
alias ..='cd ..'
alias grep='grep --color=auto'
alias update='sudo apt-get update && sudo apt-get upgrade'
```

3. **定期更新系统**
```bash
# Ubuntu/Debian
sudo apt-get update && sudo apt-get upgrade

# CentOS/RHEL
sudo yum update
```

4. **监控系统安全**
```bash
# 查看登录失败记录
sudo grep "Failed password" /var/log/auth.log

# 查看最近登录
last
lastb  # 失败的登录尝试
```

5. **使用sudo而不是root登录**
```bash
# 添加用户到sudo组
sudo usermod -aG sudo username

# 配置sudo无需密码（谨慎！）
sudo visudo
# 添加: username ALL=(ALL) NOPASSWD: ALL
```

## 紧急恢复 Emergency Recovery

### 单用户模式
1. 重启系统
2. 在GRUB菜单按'e'编辑
3. 找到linux行，在末尾添加 `single` 或 `init=/bin/bash`
4. 按Ctrl+X启动

### 忘记root密码
1. 进入单用户模式
2. 重新挂载根文件系统为读写：`mount -o remount,rw /`
3. 修改密码：`passwd root`
4. 重启：`reboot -f`

## 标签 Tags
`linux` `troubleshooting` `system-admin` `常见问题` `运维`
