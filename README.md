# TechBlog 🚀

> **记录我在本地环境中“折腾”前沿技术的实验笔记。**
> 涵盖 Large Language Models (LLM), AI Agents, ROS (Robot Operating System) 及其他新兴开发工具。这里没有枯燥的官方文档，只有**真实的环境配置步骤、踩过的深坑、以及实战心得**。

## 📜 关于这个仓库 (About)

在这个技术迭代以天为单位的时代，实践出真知，这个仓库的主要目的是不让这难得的真知被忘记得太快。`TechBlog` 是我个人的**技术实验室日志**，记录我将潮流技术落地到个人电脑（Windows / Linux / macOS）的全过程。

我的目标很明确：
1.  **复现性**：确保读者（包括未来的我）能按照步骤复现环境。
2.  **避坑指南**：详细记录那些导致进程崩溃、内存溢出或依赖冲突的“坑”。
3.  **工程化思考**：不仅是 Hello World，更关注如何在本地高效开发 Agent 或与物理/虚拟机器人交互。

## 🛠 技术栈与关注领域 (Tech Stack & Interests)

### 1. Large Language Models (LLM)
- **推理引擎**：vLLM, Ollama, Local LLM 部署优化
- **应用层**：LangChain, LlamaIndex
- **微调与量化**：QLoRA, GGUF 格式探索

### 2. AI Agents
- **框架**：CrewAI, AutoGen, LangGraph
- **场景**：自主代码生成、多智能体协作、RAG 系统构建

### 3. Robotics (ROS / ROS2)
- **系统**：ROS 1 (Noetic) / ROS 2 (Humble/Iron)
- **环境**：X-ARM, 机械臂仿真、SLAM 算法本地化
- **集成**：ROS <-> LLM 交互接口（让大模型驱动机器人）

### 4. DevOps & Tools
- **容器化**：Docker 在 ROS 和 LLM 环境中的应用
- **版本控制**：Git 工作流


### 5. And More ……

- **很多很多**：我愿意，有时间尝试的。

## 📂 目录结构 (Directory Structure)

先不定目录结构，也许写得多了就有目录结构了。

## 🚨 避坑指南 (Pitfall Avoidance)

*这里是该仓库的灵魂所在。我们在折腾过程中遇到的经典错误及解决方案：*

### 💥 典型场景 1：LLM 本地部署内存不足
- **现象**：加载 7B 模型时 OOM (Out Of Memory)。
- **原因**：未启用量化或 Batch Size 过大。
- **解决**：使用 `--quantized` 参数，或调整 `max_context_length`。
-> 详细步骤见 [[llm/ollama_setup]]

### 💥 典型场景 2：ROS2 与 Python 环境冲突
- **现象**：`import rclpy` 报错 `ModuleNotFoundError` 或链接错误。
- **原因**：系统 Python 与虚拟环境混用，或 `LD_LIBRARY_PATH` 未正确设置。
- **解决**：严格使用 `conda` 或 `venv` 隔离 ROS2 的 Python 依赖。
-> 详细步骤见 [[ros/ros2_wsl2_setup]]

### 💥 典型场景 3：Agent 工具调用死循环
- **现象**：Agent 反复调用同一工具，Token 耗尽。
- **原因**：提示词（Prompt）缺乏明确的停止条件或反馈机制。
- **解决**：引入 “Human-in-the-loop” 或设置最大迭代次数 `max_iterations`。
-> 详细步骤见 [[agents/crewai_vs_langchain]]

## 📝 如何阅读 (How to Read)

1.  **按概率搜索**：看缘分几何。
2.  **看时间线**：技术更新极快，请留意每篇笔记的 `Last Updated` 日期。

## 📄 开源许可 (License)

本仓库内容基于 [MIT License](./LICENSE) 开源。你可以自由使用、修改和分发，但请保留原始声明。

---

**⚠️ 免责声明**：
本仓库中的实验记录可能包含不稳定的配置或破坏性的命令（尤其是涉及 Docker 和 系统级 Python 环境时）。请在使用 `sudo` 或修改系统文件前务必备份！

## 🤝 合作与联系

如果你对将 LLM 应用于机器人控制感兴趣，或者想分享你的“折腾”心得，欢迎提交 PR 或 Issue 交流。

- **Email**: [holmes83@163.com]


