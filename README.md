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

## 4. 电鸿 (PowerHarmony) 环境工具链

SDK 编译依赖的环境，Ubuntu 22.04 适配。**命令必须分块执行**——单条长命令粘贴时会折行，后面的包全被当成独立命令而装不上。

```bash
# 基础（已装就跳过）
sudo apt update
sudo apt install -y binutils binutils-dev git git-lfs gnupg flex bison gperf build-essential zip curl zlib1g-dev

# 交叉编译链（apt 自带即 10.3 版，符合文档要求）
sudo apt install -y gcc-multilib g++-multilib gcc-arm-none-eabi
arm-none-eabi-gcc --version    # 验证

# 32 位库
sudo apt install -y libc6-dev-i386 lib32ncurses6 lib32z1 ccache

# 编译依赖工具
sudo apt install -y libxml2-utils xsltproc bc gnutls-bin python3-pip ruby genext2fs device-tree-compiler libffi-dev pkg-config libssl-dev libelf-dev libdwarf-dev

# 烧录/文档工具
sudo apt install -y u-boot-tools mtd-utils cpio doxygen openjdk-17-jre-headless texinfo mtools default-jre default-jdk

# 杂项
sudo apt install -y wget scons rsync git-core libxml2-dev xxd
sudo apt install -y libglib2.0-dev libpixman-1-dev kmod jfsutils reiserfsprogs xfsprogs squashfs-tools quota ppp vim locales libtinfo-dev

# 补充（文档 p7 清单里剩余的，GUI/图形相关，需要才装）
sudo apt install -y libgl1-mesa-dev x11proto-core-dev libx11-dev liblz4-tool apt-utils grsync pcmciautils
sudo apt install -y libxinerama-dev libxcursor-dev libxrandr-dev libxi-dev libncursesw5-dev libstdc++6
```

验证：

```bash
python3 --version   # 要 >= 3.8，实测 3.10.12（文档要求 python3.8，22.04 无此包，用 3.10 即可）
java -version       # 要 >= 8，实测 17（文档要求 java8，22.04 用 17 替代）
```

### 坑

- **长命令折行**：粘贴后后面的包名变成独立命令报"未找到命令"，要分块短命令执行。
- **`gcc-arm-linux-gnueabi` 装不上**：依赖 `gcc-11-arm-linux-gnueabi (>= 11.2)`，即使底层装好元包仍报冲突。**MCU 编译用不到它**，直接跳过，交叉编译链用 `gcc-arm-none-eabi`。
- **`e2fsprogs` 报未找到命令**：折行误报，系统内置，无需装。
- **文档要求的包在 22.04 不存在**：`python3.8`/`python3.8-distutils`（用系统 python3.10）、`lib32ncurses5-dev`（用 lib32ncurses6）、`libtinfo5`/`libncurses5`/`libncursesw5`（改名 `-dev` 后缀）、`openjdk-8`（用 openjdk-17）、`liblz4-tool` 等。文档基于 18.04，22.04 按包名适配。

### 下一步（需先拿到 SDK 源码）

源码到手后：解压 → 代码根目录装 hb → `./build/prebuilts_download.sh` → `hb set` 选产品 → `hb build -f`。
