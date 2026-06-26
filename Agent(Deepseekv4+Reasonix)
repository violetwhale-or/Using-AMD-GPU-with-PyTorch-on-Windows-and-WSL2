
---

# Agent 工具接入指南（以 DeepSeek V4 + Reasonix 为例）

> **环境**: WSL 2 Ubuntu 22.04 / 24.04 | **Node.js**: 24.x LTS | **Agent**: Reasonix (DeepSeek 官方编程助手)

---

## 📋 前置要求

| 项目 | 要求 |
|------|------|
| 操作系统 | WSL 2 Ubuntu 22.04 / 24.04 |
| Node.js | 版本 ≥ 22（推荐 24.x LTS） |
| 网络 | 能正常访问 DeepSeek API 服务 |
| API Key | 已申请 DeepSeek 官方 API Key |

---

## 目录

1. [安装 Node.js 24.x](#1-安装-nodejs-24x)
2. [验证 Node.js 安装](#2-验证-nodejs-安装)
3. [安装并运行 Reasonix](#3-安装并运行-reasonix)
4. [配置 DeepSeek API Key](#4-配置-deepseek-api-key)
5. [验证 Reasonix 是否正常工作](#5-验证-reasonix-是否正常工作)

---

## 1. 安装 Node.js 24.x

在 WSL 2 终端中执行以下命令：

```bash
# 1. 安装 NodeSource 仓库的 GPG 密钥和依赖工具
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings

# 2. 下载并导入 NodeSource GPG 密钥
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

# 3. 添加 Node.js 24.x 的 APT 仓库源
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_24.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

# 4. 更新软件包索引并安装 Node.js
sudo apt update
sudo apt install -y nodejs
```

> **说明**：
> - 此方法通过 NodeSource 官方仓库安装，是 Linux 下安装 Node.js 的推荐方式。
> - Node.js 24.x 为 LTS（长期支持）版本，稳定且适合生产使用。
> - 如网络受限，可将 `deb.nodesource.com` 替换为国内镜像（如 `mirrors.ustc.edu.cn/nodesource/deb`）。

---

## 2. 验证 Node.js 安装

安装完成后，执行以下命令验证版本：

```bash
node --version
# 预期输出: v24.18.0（或更高 24.x 版本）

npm --version
# 预期输出: 10.x.x（或更高）
```

如果正确显示版本号，则 Node.js 安装成功。

---

## 3. 安装并运行 Reasonix

Reasonix 是 DeepSeek 官方推出的编程 Agent 工具，可通过 `npx` 直接运行，无需全局安装。

### 3.1 进入项目目录

```bash
cd ~/your_project_directory
```

> **建议**：在存放代码的项目目录下运行 Reasonix，以便 Agent 能正确读取项目上下文。

### 3.2 一键运行

```bash
npx reasonix code
```

首次运行时，`npx` 会自动下载 Reasonix 及其依赖包，等待片刻即可。

---

## 4. 配置 DeepSeek API Key

首次运行 `npx reasonix code` 后，终端会提示你输入 API Key：

```
Please enter your DeepSeek API Key: 
```

**操作步骤**：

1. 登录 [DeepSeek 平台](https://platform.deepseek.com/) 获取你的 API Key。
2. 在提示符处粘贴 API Key（格式通常为 `sk-xxxxxxxxxxxxxxxxxxxxxxxx`），按 `Enter` 确认。
3. Reasonix 会自动保存 API Key 到本地配置文件（`~/.reasonix/config.json`），下次使用无需重复输入。

> **安全提示**：
> - API Key 保存在本地，不会上传到任何第三方服务器。
> - 如不慎泄露，请立即在 DeepSeek 平台重新生成新的 API Key。

---

## 5. 验证 Reasonix 是否正常工作

配置完成后，Reasonix 会进入交互模式。你可以尝试以下测试：

### 5.1 简单问答测试

在 Reasonix 交互界面输入：

```
请解释一下 Python 中的装饰器（decorator）是什么
```

如果 Reasonix 能正常返回解释内容，说明接入成功。

### 5.2 代码生成测试

```
用 Python 写一个快速排序函数
```

Reasonix 应能生成相应的代码片段。

---

## 至此配置完成

---

## 🔧 附录：常用命令速查

| 操作 | 命令 |
|------|------|
| 查看 Node.js 版本 | `node --version` |
| 查看 npm 版本 | `npm --version` |
| 运行 Reasonix | `npx reasonix code` |
| 清理 Reasonix 缓存 | `npx reasonix code --clear-cache` |
| 更新 Reasonix 到最新版 | `npx reasonix code@latest` |
| 查看 API Key 配置状态 | `cat ~/.reasonix/config.json` |

---

## ❓ 常见问题排查

### Q1: `npx: command not found`

**原因**：Node.js 未正确安装或 `npx` 未添加到 PATH。

**解决方案**：
```bash
# 确认 Node.js 已安装
which node

# 手动安装 npx
npm install -g npx
```

### Q2: 提示 `API Key 无效` 或 `401 Unauthorized`

**原因**：API Key 不正确或已过期。

**解决方案**：
1. 登录 DeepSeek 平台重新生成 API Key。
2. 删除本地配置文件重新配置：
   ```bash
   rm ~/.reasonix/config.json
   npx reasonix code
   ```

