

> 本文档记录 DeepSeek Harness（dsh）从安装、插件管理、故障排除到卸载的完整操作流程，适用于 WSL / Linux 环境。

---

## 1. 安装

### 1.1 快速安装（npx 方式）

前置条件：已安装 [Node.js](https://nodejs.org/)（建议 18 及以上版本）。

```bash
# 通过 npx 直接启动 Web UI，无需全局安装
npx @deepseek-ai/dsh web
```

- 默认在 `http://127.0.0.1:3080` 启动 Web 服务并自动打开浏览器。
- 若通过 SSH 远程启动，本地端口转发地址由 SSH 客户端/编辑器接管，终端只会打印 host URL。
- 附加 `--no-open` 参数可仅启动服务、不打开浏览器：

```bash
npx @deepseek-ai/dsh web --no-open
```

> 详细 Web UI 使用指南见 [官方文档](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/index.md)。

### 1.2 从源码构建

适用于需要二次开发或 npx 拉取失败的场景。

```bash
# 克隆仓库
git clone https://github.com/deepseek-ai/deepseek-harness.git

# 若遇到 SSL 证书验证失败（常见于公司内网/代理环境），跳过证书校验
git -c http.sslVerify=false clone https://github.com/deepseek-ai/deepseek-harness.git

# 进入项目目录
cd deepseek-harness

# 安装项目依赖（使用 pnpm）
pnpm install

# 编译项目产物（生成 dist / 原生二进制等）
pnpm run build

# 使用已编译产物启动 Web UI（不会重新 build）
pnpm dsh web
```

> `pnpm run build` 负责产出可运行文件；`pnpm dsh web` 直接消费这些产物，避免重复编译。

---

## 2. 插件管理

dsh 支持以 `--profile web` 方式管理 Web 端插件。

### 2.1 安装插件

```bash
# 为 web profile 添加 dshmarket 插件
dsh plugin --profile web add dshmarket
```

### 2.2 移除插件

```bash
# 移除 dsh-web-ui-all（@linxin666 维护的 Web UI 扩展）
pnpm dsh plugin --profile web remove @linxin666/dsh-web-ui-all

# 移除 dsh-agent-teams（@nanmicoder 维护的多 Agent 协作插件）
pnpm dsh plugin --profile web remove @nanmicoder/dsh-agent-teams

# 移除 dsh-vision-router（视觉路由插件）
pnpm dsh plugin --profile web remove dsh-vision-router
```

> 移除后建议重启一次 `pnpm dsh web` 使变更生效。

---

## 3. 故障排除

### 3.1 重启 WSL 后重试

**症状**：dsh 启动后无响应、端口被占用、或 WSL 内部网络异常。

**处理**：

```bash
# 在 Windows 终端（PowerShell / CMD）中执行，彻底终止 WSL
wsl --terminate <distro-name>

# 重新进入 WSL
wsl -d <distro-name>
```

等待 10–30 秒让网络栈重新初始化，再执行 `pnpm dsh web`。

### 3.2 重新编译原生依赖（最常见的段错误解法）

**症状**：运行时出现 `Segmentation fault`、`error while loading shared libraries`、或 `node-gyp` 相关报错。

**原因**：`esbuild`、`swc`、`sharp`、`node-sass` 等包含 C/C++ 预编译二进制的包，在 Node.js 版本切换或系统库更新后会与当前环境不匹配。

**处理**：

```bash
# 强制对所有原生依赖执行 rebuild，重新编译/链接二进制
pnpm rebuild
```

完成后再次运行：

```bash
pnpm dsh web
```

### 3.3 彻底清理并重新安装（"核弹"级清理）

**症状**：`pnpm rebuild` 仍无效，怀疑 pnpm 全局 store 或 lock 文件已损坏。

**处理**：

```bash
# 将 registry 指向国内镜像，避免网络超时
pnpm config set registry https://registry.npmmirror.com

# 1. 删除项目本地依赖目录
rm -rf node_modules

# 2. 清理 pnpm 全局 store 中不再被任何项目引用的包
pnpm store prune

# 3. 删除 lock 文件，强制重新解析依赖树(可选)
rm -f pnpm-lock.yaml

# 4. 全新安装依赖
pnpm install

# 5. 重新构建并启动
pnpm run build
pnpm dsh web
```

> ⚠️ 此操作会丢弃当前 lock 文件，依赖版本可能漂移，确认无误后再提交。

---

## 4. 卸载

如需彻底移除 DeepSeek Harness 及所有用户数据：

```bash
# 删除源码 / 安装目录（替换为实际路径）
rm -rf ~/XXXX/deepseek-harness/

# 删除 dsh 用户配置、缓存、会话数据
rm -rf ~/.dsh

# 删除 pnpm 全局 store 中 dsh 相关的包缓存
rm -rf ~/.local/share/pnpm/store/v11
```

完成后重新执行第 1 章的安装步骤即可从零恢复。

---

## 5. 参考

- 项目仓库：[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)
- Web UI 指南：[docs/user/guide](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/user/guide/index.md)