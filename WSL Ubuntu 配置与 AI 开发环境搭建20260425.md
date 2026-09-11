# 完整指南：WSL Ubuntu 配置与 AI 开发环境搭建

本文档汇集了在 Windows Subsystem for Linux (WSL) 中配置 Ubuntu 环境的完整流程，涵盖了从系统迁移、文件互访、Python 环境配置、深度学习框架 (PyTorch) 安装，到本地大模型 (Ollama) 部署及 AI 工具 (Claude Code) 和 ROS2 的安装指南。

## 1. WSL 的安装、迁移与基础操作

### 1.1 安装 WSL

如果你还没有安装 WSL，请打开管理员权限的 **PowerShell** 并运行以下命令：

```
wsl --install
```

默认情况下，这会安装 Ubuntu 并将其放置在 C 盘。

### 1.2 将 Ubuntu 从 C 盘迁移至其他磁盘

当 C 盘空间告急时，可以将 WSL 迁移到其他磁盘（例如 D 盘）：

1. **关闭正在运行的 Ubuntu**：
    
    ```
    wsl --list --verbose
    wsl --terminate Ubuntu
    ```
    
2. **导出系统镜像**（以备份到 `D:\WSL_Backup` 为例）：
    
    ```
    wsl --export Ubuntu D:\WSL_Backup\ubuntu_backup.tar
    ```
    
3. **注销原有系统（释放 C 盘空间）**：
    
    ```
    wsl --unregister Ubuntu
    ```
    
4. **导入到新磁盘**（以导入到 `D:\WSL\Ubuntu` 为例）：
    
    ```
    wsl --import Ubuntu D:\WSL\Ubuntu D:\WSL_Backup\ubuntu_backup.tar
    ```
    
5. **恢复默认用户**（重要：迁移后默认是 root 用户）：
    
    ```
    ubuntu config --default-user <你的用户名>
    ```
    
    _如果命令不生效，可以在进入 WSL 后通过修改 `/etc/wsl.conf` 添加 `[user]\ndefault=<你的用户名>` 来固定默认用户。_
    

### 1.3 WSL 常用基础管理命令

日常使用中，你经常会在 PowerShell 中用到以下指令来管理 WSL：

- **查看当前所有子系统及其状态**：
    
    ```
    wsl -l -v
    ```
    
- **彻底关闭并重启 WSL 服务**（当遇到卡顿、修改注册表或需让配置文件生效时非常有用）：
    
    ```
    wsl --shutdown
    ```
    
    执行后重新输入 `wsl` 即可重新启动系统。
    

### 1.4 在 Windows 中访问 WSL 文件目录 (新增)

微软深度集成了 Windows 与 WSL 的文件系统，你不需要复杂的挂载即可轻松互相访问。

**方法一：通过文件资源管理器（最直观）**

1. 打开 Windows 文件资源管理器 (File Explorer)。
    
2. 在左侧导航栏找到带有 **小企鹅图标** 的 “Linux” 选项。点击进入即可看到 `Ubuntu` 文件夹及完整的根目录（如 `/home/username`）。 _小贴士：如果没看到企鹅图标，可以直接在地址栏输入 `\\wsl$` 然后按回车，你会看到所有正在运行的 Linux 分发版。_
    

**方法二：通过命令行快速打开（最快捷）** 如果你正在 Ubuntu 终端里，想立刻在 Windows 中打开当前所在的 Linux 目录，输入以下命令（注意 `.` 前面有空格）：

```
explorer.exe .
```

**方法三：在 VS Code 中访问（开发者首选）** 在 Ubuntu 终端的任意项目目录下输入：

```
code .
```

VS Code 会自动启动并在 Windows 界面中打开 Linux 里的代码文件夹（需要安装 WSL 扩展），你可以直接在 Windows 下高效编写代码。

⚠️ **重要警告：绝对不要反向操作！** 虽然通过 `\\wsl$` 访问很安全，但 **绝对不要** 顺着 Windows 的 C 盘路径（如 `C:\Users\...\AppData\Local\Packages...`）去找底层隐藏的 Linux 镜像文件并直接修改。直接修改底层镜像会导致 Linux 权限混乱甚至系统彻底崩溃！

## 2. Python 环境配置：pip 与 venv

在 Ubuntu 中进行 AI 开发，良好的包管理习惯至关重要。强烈建议使用虚拟环境（venv）而不是在系统全局安装包。

### 2.1 安装 pip 与 venv 模块

更新包列表并安装 Python3 的 pip 和 venv 工具：

```
sudo apt update
sudo apt install python3-pip python3-venv
```

### 2.2 创建与激活虚拟环境

1. **创建虚拟环境**（在你的项目目录下，`myenv` 为环境名称）：
    
    ```
    python3 -m venv myenv
    ```
    
2. **激活虚拟环境**：
    
    ```
    source myenv/bin/activate
    ```
    
    _激活后，命令行开头会出现 `(myenv)` 字样。此时使用 `pip install` 安装的包将只存在于该环境中。_
    
3. **退出虚拟环境**：
    
    ```
    deactivate
    ```
    

## 3. PyTorch 与 GPU 硬件加速配置

### 3.1 验证 WSL 的 GPU 支持 (重要)

在 WSL 中运行 AI 框架和模型，强烈建议使用显卡加速。现代的 WSL2 已经原生支持将 Windows 端的 GPU 映射到 Linux 中。 在 Windows 侧装好最新的 NVIDIA 显卡驱动后，请在 Ubuntu 终端输入以下命令检查 GPU 状态：

```
nvidia-smi
```

- **成功**：如果你能看到一张包含显卡型号、显存和驱动版本的信息表格，说明 GPU 已成功打通。
    
- **失败**：如果提示 `command not found` 或无法通信，请检查 Windows 端的 NVIDIA 驱动是否为最新版。
    

### 3.2 PyTorch 在虚拟环境中的安装

在确认 GPU 可用并激活了虚拟环境 `(myenv)` 后，你可以安装 PyTorch：

1. 运行 PyTorch 官方安装命令（这里以支持 CUDA 12.1 为例）：
    
    ```
    pip install torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/cu121](https://download.pytorch.org/whl/cu121)
    ```
    
    _如果你的电脑没有 NVIDIA 显卡，可以安装 CPU 版本：_
    
    ```
    pip install torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/cpu](https://download.pytorch.org/whl/cpu)
    ```
    
2. **验证深度学习框架是否成功调用显卡**：
    
    ```
    python -c "import torch; print(torch.__version__); print('GPU 可用:', torch.cuda.is_available())"
    ```
    
    如果输出 `GPU 可用: True`，则说明配置大功告成。
    

## 4. Ollama 安装与 Cherry Studio 联动

Ollama 是目前最流行的本地大模型运行工具，结合 Windows 端的 Cherry Studio，可以获得极佳的 AI 交互体验。

### 4.1 在 WSL 中安装 Ollama 并下载模型

1. **运行官方一键安装脚本**：
    
    ```
curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
    ```
    
2. **下载并运行大语言模型**（以深度求索的推理模型 **deepseek-reasoner:7B** 为例）： 在终端中输入以下指令直接运行：
    
    ```
    ollama run deepseek-reasoner:7b
    ```
    
    _初次运行时，Ollama 会自动开始 `pulling manifest` 并分块下载模型文件（7B 模型大约需要 4.7GB 的空间）。下载完成后，终端会自动变成一个对话输入框，此时你就可以直接在命令行里与模型对话了。输入 `/bye` 可以退出。_
    

### 4.2 配置 Ollama 允许 Windows 访问 (联动准备)

默认情况下，Ollama 只监听 `127.0.0.1`，我们需要修改配置让 Windows 侧的客户端能够访问它。

1. 创建并编辑 systemd 配置文件：
    
    ```
    sudo mkdir -p /etc/systemd/system/ollama.service.d
    sudo nano /etc/systemd/system/ollama.service.d/override.conf
    ```
    
2. 输入以下内容以允许跨域和监听所有 IP：
    
    ```
    [Service]
    Environment="OLLAMA_HOST=0.0.0.0"
    Environment="OLLAMA_ORIGINS=*"
    ```
    
3. 重载配置并重启服务：
    
    ```
    sudo systemctl daemon-reload
    sudo systemctl restart ollama
    ```
    

### 4.3 在 Cherry Studio 中配置

1. 打开 Windows 端的 Cherry Studio。
    
2. 进入 **设置 -> 模型服务 -> Ollama**。
    
3. 将 API 地址修改为 `http://127.0.0.1:11434`。 _(如果连接失败，可以在 WSL 中运行 `hostname -I` 查看 IP，并将地址替换为 `http://<WSL的IP>:11434`)_。
    
4. 点击“检查”，成功后即可在应用内选择刚才下载的 `deepseek-reasoner:7b` 模型进行带图形界面的问答。
    

## 5. 基于“鱼香ROS”的 ROS 2 安装

“鱼香ROS” 是国内非常流行的 ROS 一键安装工具，自动处理了换源、依赖和环境变量等繁琐步骤。

1. **运行一键安装脚本**：
    
    ```
    wget [http://fishros.com/install](http://fishros.com/install) -O fishros && . fishros
    ```
    
2. **按照屏幕提示选择**：
    
    - 选择安装 ROS 2。
        
    - 选择对应的 Ubuntu 版本代号（如 22.04 对应 Humble）。
        
    - 选择桌面版 (Desktop) 或基础版 (Base)。
        
3. **验证安装**： 安装完成后，脚本会自动将 `source /opt/ros/humble/setup.bash` 写入你的 `.bashrc` 文件。重新打开终端后输入：
    
    ```
    ros2
    ```
    
    如果能看到命令行帮助文档，说明安装成功。
    

## 6. Claude Code 安装过程

Claude Code 是 Anthropic 推出的强大命令行 AI 编程助手。**请注意，官方目前已弃用旧版的 npm 安装方式，强烈建议使用原生安装程序。**

1. **环境要求**：支持 macOS、Linux (含 WSL) 和 Windows。你需要有 Claude 的订阅 (Pro, Teams, 等) 才能使用。
    
2. **在 WSL 中安装（原生安装法）**： 打开 WSL 终端，运行以下命令：
    
    ```
    curl -fsSL [https://claude.ai/install.sh](https://claude.ai/install.sh) | bash
    ```
    
3. **启动与鉴权**： 进入你的代码项目目录，输入：
    
    ```
    claude
    ```
    
    首次运行会弹出一个链接（或要求你在浏览器中打开），登录你的 Anthropic 账号进行授权即可开始体验终端里的 AI 编程。