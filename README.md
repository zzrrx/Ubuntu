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

## 3. 电鸿 (PowerHarmony) 环境搭建

Samba 共享、环境工具链、SDK 解压、编译、烧录的完整步骤见 **[电鸿操作系统环境搭建.md](电鸿操作系统环境搭建.md)**。
