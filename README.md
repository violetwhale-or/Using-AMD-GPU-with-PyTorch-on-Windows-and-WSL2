# 在 Windows WSL 2 中配置 AMD ROCm + PyTorch 环境指南

> **硬件**: AMD Radeon RX 7800 XT | **系统**: Windows 11 + WSL 2 Ubuntu 22.04 | **ROCm**: 7.2.1 | **PyTorch**: 2.9.1

---

## 📋 系统要求

| 项目 | 要求 |
|------|------|
| Windows 版本 | Windows 11 或 Windows 10 版本 21H2+ |
| 显卡 | AMD RDNA 3 架构（RX 7000 系列） |
| 内存 | 建议 16GB+ |
| 磁盘空间 | 建议 50GB+ 可用空间 |

---

## 目录

1. [安装/升级 WSL](#1-安装升级-wsl)
2. [将 Ubuntu 安装到指定盘符](#2-将-ubuntu-安装到指定盘符)
3. [设置用户名和密码](#3-设置用户名和密码)
4. [更换清华源](#4-更换清华源)
5. [安装基础依赖](#5-安装基础依赖)
6. [添加 ROCm 7.2.1 仓库](#6-添加-rocm-721-仓库)
7. [安装 ROCm 7.2.1](#7-安装-rocm-721)
8. [安装 librocdxg（WSL GPU 支持）](#8-安装-librocdxgwsl-gpu-支持)
9. [配置环境变量](#9-配置环境变量)
10. [验证 ROCm 安装](#10-验证-rocm-安装)
11. [安装 Miniconda](#11-安装-miniconda)
12. [配置 Conda 和 Pip 国内镜像源](#12-配置-conda-和-pip-国内镜像源)
13. [创建 Conda 虚拟环境](#13-创建-conda-虚拟环境)
14. [安装 PyTorch for ROCm 7.2](#14-安装-pytorch-for-rocm-72)
15. [验证安装](#15-验证安装)

---

## 1. 安装/升级 WSL

以**管理员身份**打开 PowerShell，执行：

```powershell
wsl --install -d Ubuntu-22.04
```

> **说明**：如果电脑内没有 WSL 环境或版本较低，此命令会优先升级 WSL 版本。例如：
> ```
> 正在下载: 适用于 Linux 的 Windows 子系统 2.7.8
> 正在安装: 适用于 Linux 的 Windows 子系统 2.7.8
> 已安装 适用于 Linux 的 Windows 子系统 2.7.8。
> ```

> ⚠️ **安装完成后必须重启电脑**，以启用 VirtualMachinePlatform 功能。

---

## 2. 将 Ubuntu 安装到指定盘符

重启后，以管理员身份打开 PowerShell，执行：

```powershell
wsl --install -d Ubuntu-22.04 --location D:\WSL\Ubuntu-22.04
```

> **说明**：`--location` 后可替换为任意盘符路径，不指定则默认安装到 C 盘。

下载安装结束后会弹出 Ubuntu 欢迎页面，**切回 PowerShell 或 Windows Terminal 继续后续操作**（WSL 已在后台运行）。

---

## 3. 设置用户名和密码

在欢迎页面中，系统会提示：

```
Create a default Unix user account: User
```

- `User` 会自动填充为 Windows 用户名，直接按 `Enter` 确认。
- 输入密码时**不会显示任何字符**（包括星号），这是正常的安全机制，直接输入后按 `Enter`，再确认一次。

完成后，切换到 Linux 主目录：

```bash
cd ~
```

---

## 4. 更换清华源

```bash
# 备份原配置
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak.$(date +%Y%m%d)

# 写入清华镜像源
sudo tee /etc/apt/sources.list << 'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
EOF

sudo apt update
```

> **说明**：`jammy` 是 Ubuntu 22.04 的代号。如使用 Ubuntu 24.04，需将 `jammy` 替换为 `noble`。

---

## 5. 安装基础依赖

```bash
sudo apt install -y \
    build-essential \
    cmake \
    git \
    wget \
    curl \
    gnupg2 \
    software-properties-common \
    libjpeg-dev \
    python3-dev \
    python3-pip \
    pkg-config \
    libnuma-dev \
    libdrm-dev \
    aria2
```

> **说明**：这些是系统级编译工具和 ROCm 依赖库，不会污染 Python 环境。

---

## 6. 添加 ROCm 7.2.1 仓库

```bash
# 添加 AMD GPG 密钥
wget -qO - https://repo.radeon.com/rocm/rocm.gpg.key | sudo apt-key add -

# 添加 ROCm 7.2.1 仓库
echo 'deb [arch=amd64] https://repo.radeon.com/rocm/apt/7.2.1 jammy main' | sudo tee /etc/apt/sources.list.d/rocm.list

# 设置 apt 优先级，强制使用 AMD 官方仓库
# （解决 Ubuntu universe 中旧版 ROCm 5.0.0 包冲突）
sudo tee /etc/apt/preferences.d/rocm-pin-1001 << 'EOF'
Package: *
Pin: origin repo.radeon.com
Pin-Priority: 1001
EOF

sudo apt update
```

---

## 7. 安装 ROCm 7.2.1

```bash
sudo apt install -y rocm-dev rocm-libs
```

如果报错，可使用指定版本安装：

```bash
sudo apt install -y \
    rocm-cmake=0.14.0.70201-81~22.04 \
    rocm-device-libs=1.0.0.70201-81~22.04 \
    rocm-utils=7.2.1.70201-81~22.04 \
    rocm-dev \
    rocm-libs
```

---

## 8. 安装 librocdxg（WSL GPU 支持）

```bash
cd /tmp
wget https://github.com/ROCm/librocdxg/releases/download/v1.1.2/rocdxg-roct_1.1.2_amd64.deb
sudo dpkg -i rocdxg-roct_1.1.2_amd64.deb
sudo apt --fix-broken install -y
```

### ⚠️ 常见问题：GitHub 地址被解析为 127.0.0.1

如果执行 `wget` 时出现以下错误：

```
Resolving github.com (github.com)... 127.0.0.1
Connecting to github.com (github.com)|127.0.0.1|:443... failed: Connection refused.
```

**解决方案**：

1. **检查 hosts 文件**：
   ```bash
   cat /etc/hosts | grep github
   ```
   如果有 `127.0.0.1 github.com` 条目，用 `sudo nano /etc/hosts` 注释或删除该行。

2. **检查 DNS 配置**：
   ```bash
   dig @8.8.8.8 github.com
   ```
   应返回正确 IP（如 `20.205.243.166`）。

3. **备用方案——手动下载**：
   在 Windows 浏览器中打开：
   ```
   https://github.com/ROCm/librocdxg/releases/download/v1.1.2/rocdxg-roct_1.1.2_amd64.deb
   ```
   下载完成后，在 Windows 文件资源管理器地址栏输入 `\\wsl.localhost\Ubuntu\home\你的用户名\`，将 `.deb` 文件拖入该目录。

   然后在 WSL 终端执行：
   ```bash
   sudo dpkg -i ~/rocdxg-roct_1.1.2_amd64.deb
   sudo apt --fix-broken install -y
   ```

4. **验证文件已就位**：
   ```bash
   ls -la ~/rocdxg-roct_1.1.2_amd64.deb
   ```
   如看到文件大小（约 166KB），则成功。

---

## 9. 配置环境变量

```bash
cat >> ~/.bashrc << 'EOF'

# ROCm
export PATH=/opt/rocm/bin:/opt/rocm/hip/bin:$PATH
export HSA_ENABLE_DXG_DETECTION=1
EOF

source ~/.bashrc
```

---

## 10. 验证 ROCm 安装

```bash
rocminfo | grep "Marketing Name"
```

预期输出应包含你的显卡型号，例如：

```
Marketing Name: AMD Radeon RX 7800 XT
```

也可通过 `rocm-smi` 查看：

```bash
/opt/rocm/bin/rocm-smi --showproductname
```

---

## 11. 安装 Miniconda

```bash
# 下载 Miniconda（清华镜像）
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh

# 安装
bash miniconda.sh -b -p $HOME/miniconda3

# 初始化
~/miniconda3/bin/conda init bash
source ~/.bashrc
```

看到命令行前有 `(base)` 则表示 conda 正常启动。

---

## 12. 配置 Conda 和 Pip 国内镜像源

### Conda 清华镜像源

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch
conda config --set show_channel_urls yes
```

> **提示**：如果提示 `already in 'channels' list, moving to the top`，属于正常信息，无需处理。

### Pip 清华镜像源

```bash
mkdir -p ~/.config/pip
cat > ~/.config/pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF
```

---

## 13. 创建 Conda 虚拟环境

```bash
conda create -n amdpy310 python=3.10 -y
conda activate amdpy310
```

> **注意**：第一次创建环境时可能会弹出 Anaconda 服务条款确认，输入 `a`（accept）即可。  
> 如需永久跳过，执行：`conda config --set accept_terms_of_service true`

---

## 14. 安装 PyTorch for ROCm 7.2

```bash
mkdir -p ~/pytorch_whl && cd ~/pytorch_whl
```

### 下载各组件

```bash
# torch 2.9.1（约 1.6GB）
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/torch-2.9.1%2Brocm7.2.0.lw.git7e1940d4-cp310-cp310-linux_x86_64.whl

# torchvision 0.24.0（约 2.9MB）
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/torchvision-0.24.0%2Brocm7.2.0.gitb919bd0c-cp310-cp310-linux_x86_64.whl

# torchaudio 2.9.0（约 487KB）
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/torchaudio-2.9.0%2Brocm7.2.0.gite3c6ee2b-cp310-cp310-linux_x86_64.whl

# triton（PyTorch 依赖，约 287MB）
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-7.2/triton-3.5.1%2Brocm7.2.0.gita272dfa8-cp310-cp310-linux_x86_64.whl
```

### 本地安装

```bash
cd ~/pytorch_whl
pip install ./*.whl
```

---

## 15. 验证安装

```bash
python -c "
import torch
print(f'PyTorch版本: {torch.__version__}')
print(f'ROCm/HIP版本: {torch.version.hip}')
print(f'GPU可用: {torch.cuda.is_available()}')
print(f'GPU数量: {torch.cuda.device_count()}')
if torch.cuda.is_available():
    print(f'GPU名称: {torch.cuda.get_device_name(0)}')
    x = torch.rand(1000, 1000).cuda()
    y = torch.rand(1000, 1000).cuda()
    z = x @ y
    print('GPU 矩阵乘法测试通过')
"
```

### 预期输出

```
PyTorch版本: 2.9.1+rocm7.2.0.lw.git7e1940d4
ROCm/HIP版本: 7.2.x
GPU可用: True
GPU数量: 1
GPU名称: AMD Radeon RX 7800 XT
GPU 矩阵乘法测试通过
```

---



## 🔧 附录：常用命令速查

| 操作 | 命令 |
|------|------|
| 进入 WSL | `wsl` |
| 指定发行版进入 | `wsl -d Ubuntu-22.04` |
| 关闭 WSL | `wsl --shutdown` |
| 查看 WSL 发行版列表 | `wsl -l -v` |
| 激活 conda 环境 | `conda activate amdpy310` |
| 退出 conda 环境 | `conda deactivate` |
| 查看 GPU 状态 | `/opt/rocm/bin/rocm-smi` |
| VS Code 连接 WSL | `code .`（在 WSL 终端中执行） |

---
