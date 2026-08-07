# Ubuntu 配置记录

## 1. 安装 opencode

```bash
# 1. 安装 curl
sudo apt update
sudo apt install -y curl

# 安装新版 Node 20
cd /tmp
curl -fsSL -o node.tar.xz https://nodejs.org/dist/v20.15.0/node-v20.15.0-linux-x64.tar.xz
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
```

Windows VSCode：装扩展 **Remote-SSH** → `Ctrl+Shift+P` → `Remote-SSH: Connect to Host` → 输入 `zrx@<IP>` → 输密码。
