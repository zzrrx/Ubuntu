# Ubuntu 环境配置记录

本仓库记录在 Ubuntu 虚拟机上的实际操作步骤：安装 AI 编程工具 opencode、配置 VSCode SSH 远程连接。

环境：Ubuntu 22.04 (jammy)，x86_64，VMware 虚拟机。

---

## 1. 安装 opencode

> 坑点记录：
> - 系统自带的 node 是 12.22.9，opencode 需要 node ≥ 18，必须换新版
> - `opencode.ai` 和 `npmmirror.com` 都连不上（网络出口受限），但 `nodejs.org` 和 `registry.npmjs.org` 通
> - 本机网络能通官方源的，可以直接用官方下载

### 1.1 下载安装新版 Node (v20)

```bash
# 确认架构
uname -m

# 下载 node 20 二进制（nodejs.org 可访问）
cd /tmp
curl -fsSL -o node.tar.xz https://nodejs.org/dist/v20.15.0/node-v20.15.0-linux-x64.tar.xz

# 解压到 /usr/local/node
sudo mkdir -p /usr/local/node
sudo tar -xJf node.tar.xz -C /usr/local/node --strip-components=1

# 加入 PATH（写入 ~/.bashrc 永久生效）
echo 'export PATH=/usr/local/node/bin:$PATH' >> ~/.bashrc
export PATH=/usr/local/node/bin:$PATH

# 验证
node -v   # 应显示 v20.15.0
```

### 1.2 用 npm 安装 opencode（装到用户目录，免 sudo）

```bash
# 设置 npm 全局安装目录为用户目录
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc

# 当前终端手动加载新 PATH
export PATH="$HOME/.npm-global/bin:$PATH"

# 安装 opencode
npm i -g opencode-ai

# 验证
opencode --version
```

> 若 `opencode` 提示"未找到命令"，说明 PATH 没加载，执行：
> ```bash
> export PATH="$HOME/.npm-global/bin:$PATH"
> ```

### 1.3 运行

```bash
opencode
```

首次运行会引导配置模型 API key。

### 备用方案：官方安装脚本

```bash
curl -fsSL https://opencode.ai/install | bash
```

> 注意：此脚本需要能访问 `opencode.ai`，网络不通时会卡死（本机实测不通）。

---

## 2. VSCode 远程连接 Ubuntu (SSH)

### 2.1 虚拟机侧：开启 SSH 服务

```bash
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
sudo systemctl is-active ssh    # 输出 active 即成功
ip a                            # 查看 IP 地址
```

### 2.2 Windows VSCode 侧

1. 安装扩展 **Remote - SSH**（发布者 Microsoft）
2. `Ctrl+Shift+P` → `Remote-SSH: Connect to Host`
3. 输入 `zrx@<虚拟机IP>`，回车，输入密码
4. host key 提示选 Continue

### 2.3 排查

- 先在 Windows `cmd` 里 ping 虚拟机 IP，确认网络通
- 虚拟机有两个网卡（如 `192.168.1.83` 和 `192.168.71.130`），哪个 ping 通就连哪个
- NAT 网络模式主机连不到虚拟机，需在虚拟机软件里配置端口转发

---

## 3. 参考命令速查

```bash
# 测网络连通性（exit=0 表示通）
curl -I --connect-timeout 6 https://nodejs.org
curl -I --connect-timeout 6 https://registry.npmjs.org
```
