# Ubuntu 配置记录

## 1. 安装 opencode

```bash
# 1. 安装 curl
sudo apt update
sudo apt install -y curl

# 安装新版 Node 20
cd /tmp
curl -fsSL -o node.tar.xz https://nodejs.org/dist/v20.15.0/node-v20.15.0-linux-x64.tar.xz

  方案一:淘宝(npmmirror)镜像
  curl -fL -o /tmp/node.tar.xz https://npmmirror.com/mirrors/node/v20.15.0/node-v20.15.0-linux-x64.tar.xz

  方案二:华为云镜像
  curl -fL -o /tmp/node.tar.xz https://mirrors.huaweicloud.com/nodejs/v20.15.0/node-v20.15.0-linux-x64.tar.xz

  方案三:腾讯云镜像
  curl -fL -o /tmp/node.tar.xz https://mirrors.cloud.tencent.com/nodejs-release/v20.15.0/node-v20.15.0-linux-x64.tar.xz

  下载完成后验证一下文件:
  ls -lh /tmp/node.tar.xz   # 大约 23MB,太小就说明下载不完整

  然后继续解压:
  sudo tar -xJf /tmp/node.tar.xz -C /usr/local/node --strip-components=1
  /usr/local/node/bin/node -v


sudo mkdir -p /usr/local/node
sudo tar -xJf node.tar.xz -C /usr/local/node --strip-components=1
echo 'export PATH=/usr/local/node/bin:$PATH' >> ~/.bashrc
export PATH=/usr/local/node/bin:$PATH

# 安装 opencode（用户目录，免 sudo）
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
export PATH="$HOME/.npm-global/bin:$PATH"
npm i -g opencode-ai

# 验证
opencode --version
```

运行：`opencode`

## 2. VSCode 远程连接 (SSH)

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
ip a    # 查 IP
hostname -I
whoami
ssh zzr@192.168.1.100
```

Windows VSCode：装扩展 **Remote-SSH** → `Ctrl+Shift+P` → `Remote-SSH: Connect to Host` → 输入 `zrx@<IP>` → 输密码。

## 3. Samba 共享目录

让 Windows 通过"映射网络驱动器"访问 Ubuntu 目录。

```bash
# 1. 装 Samba
sudo apt update
sudo apt install -y samba samba-common

# 2. 建共享目录（放在主目录下）
mkdir -p /home/zrx/share

# 3. 配置共享（追加到 /etc/samba/smb.conf）
# 推荐用 nano 手动编辑，最稳：
sudo nano /etc/samba/smb.conf
# 文件末尾追加：
#   [Share]
#   comment = Shared Folder
#   path = /home/zrx/share
#   valid users = zrx
#   directory mask = 0775
#   create mask = 0775
#   public = no
#   writable = yes
#   available = yes
#   browseable = yes
# Ctrl+O 保存，Ctrl+X 退出

# 或用 tee + heredoc（需整块复制粘贴，不能逐行）：
sudo tee -a /etc/samba/smb.conf > /dev/null <<'EOF'

[Share]
comment = Shared Folder
path = /home/zrx/share
valid users = zrx
directory mask = 0775
create mask = 0775
public = no
writable = yes
available = yes
browseable = yes
EOF

# 4. 设 Samba 用户密码（独立于 Ubuntu 登录密码）
sudo smbpasswd -a zrx

# 5. 检查配置语法（必做，能提前发现错误）
sudo testparm

# 6. 重启服务
sudo service smbd restart
sudo systemctl is-active smbd
```

Windows 侧：右键"此电脑" → 映射网络驱动器 → 文件夹填 `\\<Ubuntu IP>\share` → 输 `zrx` + Samba 密码。

### 踩坑记录

- **heredoc 逐行粘贴会卡住**：`<<'EOF'` 要整块复制粘贴，逐行输会一直等输入，Ctrl+C 退出。
- **单行 printf 命令折行会写坏配置**：终端粘贴长命令时折行，导致 `directory mask = 0775` 被拆成两行（`directory mask =` 和 `0775` 分开），smbd 起不来。
- **报错识别**：`testparm` 报 `Invalid octal number directory mask` = 配置文件里 `directory mask = 0775` 被折行拆断。
- **修复**：把拆断的两行合并成一行：
  ```bash
  sudo sed -i '/^directory mask =$/,/^  0775$/c\directory mask = 0775' /etc/samba/smb.conf
  sudo testparm   # 显示 Loaded services file OK. 即通过
  sudo service smbd restart
  ```
