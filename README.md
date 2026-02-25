# 🦌 DeerFlow - 2.0

DeerFlow（**D**eep **E**xploration and **E**fficient **R**esearch **Flow**）是一个开源的**超级智能体运行框架**，它通过可扩展的**技能**编排**子代理**、**记忆**与**沙箱**，几乎可以完成任何任务。

https://github.com/user-attachments/assets/a8bcadc4-e040-4cf2-8fda-dd768b999c18

> [!NOTE]
> **DeerFlow 2.0 是一次从零重写。**它与 v1 没有任何代码共享。如果你在寻找原始的 Deep Research 框架，请访问 [`1.x` 分支](https://github.com/bytedance/deer-flow/tree/main-1.x) —— 仍然欢迎在那边贡献。当前主要开发已迁移到 2.0。

## 官方网站

在官网了解更多信息并查看**真实演示**。

**[deerflow.tech](https://deerflow.tech/)**

---

## 目录

- [快速开始](#快速开始)
- [沙箱模式](#沙箱模式)
- [从深度研究到超级智能体框架](#从深度研究到超级智能体框架)
- [核心特性](#核心特性)
  - [技能与工具](#技能与工具)
  - [子代理](#子代理)
  - [沙箱与文件系统](#沙箱与文件系统)
  - [上下文工程](#上下文工程)
  - [长期记忆](#长期记忆)
- [推荐模型](#推荐模型)
- [文档](#文档)
- [贡献](#贡献)
- [许可证](#许可证)
- [致谢](#致谢)
- [Star 历史](#star-历史)

## 快速开始

### 配置

1. **克隆 DeerFlow 仓库**

   ```bash
   git clone https://github.com/bytedance/deer-flow.git
   cd deer-flow
   ```

2. **生成本地配置文件**

   在项目根目录（`deer-flow/`）运行：

   ```bash
   make config
   ```

   该命令会基于示例模板生成本地配置文件。

3. **配置你偏好的模型**

   编辑 `config.yaml` 并至少定义一个模型：

   ```yaml
   models:
     - name: gpt-4                       # Internal identifier
       display_name: GPT-4               # Human-readable name
       use: langchain_openai:ChatOpenAI  # LangChain class path
       model: gpt-4                      # Model identifier for API
       api_key: $OPENAI_API_KEY          # API key (recommended: use env var)
       max_tokens: 4096                  # Maximum tokens per request
       temperature: 0.7                  # Sampling temperature
   ```

4. **为已配置的模型设置 API Key**

   选择以下任一方式：

- 方案 A：编辑根目录下的 `.env` 文件（推荐）


   ```bash
   TAVILY_API_KEY=your-tavily-api-key
   OPENAI_API_KEY=your-openai-api-key
   # Add other provider keys as needed
   ```

- 方案 B：在你的 shell 中导出环境变量

   ```bash
   export OPENAI_API_KEY=your-openai-api-key
   ```

- 方案 C：直接在 `config.yaml` 中填写（生产环境不推荐）

   ```yaml
   models:
     - name: gpt-4
       api_key: your-actual-api-key-here  # Replace placeholder
   ```

### 运行应用

#### 方案一：Docker（推荐）

最快的启动方式，并确保环境一致性：

1. **初始化并启动**：
   ```bash
   make docker-init    # Pull sandbox image (Only once or when image updates)
   make docker-start   # Start all services and watch for code changes
   ```

2. **访问**：http://localhost:2026

详细的 Docker 开发指南见 [CONTRIBUTING.md](CONTRIBUTING.md)。

#### 方案二：本地开发

如果你希望在本机直接运行服务：

1. **检查前置依赖**：
   ```bash
   make check  # Verifies Node.js 22+, pnpm, uv, nginx
   ```

2. **（可选）提前拉取沙箱镜像**：
   ```bash
   # Recommended if using Docker/Container-based sandbox
   make setup-sandbox
   ```

3. **启动服务**：
   ```bash
   make dev
   ```

4. **访问**：http://localhost:2026

### 高级
#### 沙箱模式

DeerFlow 支持多种沙箱执行方式：
- **本地执行**（在宿主机直接运行沙箱代码）
- **Docker 执行**（在隔离的 Docker 容器中运行）
- **Kubernetes + Docker 执行**（通过 provisioner 服务在 K8s Pod 中运行）

可参考 [沙箱配置指南](backend/docs/CONFIGURATION.md#sandbox) 配置你的模式。

#### MCP 服务器

DeerFlow 支持可配置的 MCP 服务器与技能扩展能力。
详细说明见 [MCP 服务器指南](backend/docs/MCP_SERVER.md)。

## 从深度研究到超级智能体框架

DeerFlow 起初是一个 Deep Research 框架——但社区把它带到了更远的地方。自发布以来，开发者用它构建数据流水线、生成幻灯片、搭建仪表盘、自动化内容流程，远超我们的预期。

这让我们意识到：DeerFlow 不只是研究工具，它是一个**运行框架**——为智能体提供真正能“干活”的基础设施。

于是我们从零重写。

DeerFlow 2.0 不再是需要你自行拼装的框架，而是一个开箱即用、完全可扩展的超级智能体运行框架。它基于 LangGraph 和 LangChain，内置文件系统、记忆、技能、沙箱执行以及复杂任务的规划与子代理能力。

你可以直接使用，也可以按需拆改。

## 核心特性

### 技能与工具

技能让 DeerFlow 能做*几乎所有事情*。

标准的 Agent Skill 是结构化能力模块——一个定义流程、最佳实践与参考资源的 Markdown 文件。DeerFlow 内置研究、报告生成、幻灯片制作、网页生成、图像/视频生成等技能。更强大的是可扩展性：你可以添加自定义技能、替换内置技能，或把多个技能组合成复合流程。

技能按需渐进加载——只有在任务需要时才加载，这能显著节省上下文窗口，适配更“省 token”的模型。

工具也采用相同理念。DeerFlow 提供核心工具集——网页搜索、网页抓取、文件操作、bash 执行等，并支持通过 MCP 服务器和 Python 函数扩展自定义工具。你可以替换或新增任何工具。

```
# Paths inside the sandbox container
/mnt/skills/public
├── research/SKILL.md
├── report-generation/SKILL.md
├── slide-creation/SKILL.md
├── web-page/SKILL.md
└── image-generation/SKILL.md

/mnt/skills/custom
└── your-custom-skill/SKILL.md      ← yours
```

### 子代理

复杂任务很难一次完成，DeerFlow 会将其拆解。

主代理可以随时创建子代理——每个子代理拥有自己的上下文、工具与终止条件。子代理在可行时并行执行，产出结构化结果后再由主代理统一综合。

这让 DeerFlow 能处理耗时分钟到小时的任务：一次研究可能分发给十多个子代理，各自探索不同角度，最后汇总成一份报告、一个网站或一套带可视化的幻灯片。一个框架，多只手。

### 沙箱与文件系统

DeerFlow 不只是“说它能做”，它真的有自己的电脑。

每个任务运行在隔离的 Docker 容器中，拥有完整文件系统——技能、工作区、上传与输出。代理可以读写和编辑文件、执行 bash 命令、生成代码、查看图片。全程沙箱化、可审计、会话间零污染。

这就是“可用工具的聊天机器人”和“拥有实际执行环境的智能体”之间的本质区别。

```
# Paths inside the sandbox container
/mnt/user-data/
├── uploads/          ← your files
├── workspace/        ← agents' working directory
└── outputs/          ← final deliverables
```

### 上下文工程

**子代理隔离上下文**：每个子代理运行在完全隔离的上下文中，无法看到主代理或其他子代理的上下文，确保它能专注于自己的任务，不被其他上下文干扰。

**摘要与压缩**：在单次会话内，DeerFlow 会积极管理上下文——总结已完成的子任务，将中间结果写入文件系统，压缩当前不再关键的信息，从而在长链条任务里保持清晰而不爆上下文。

### 长期记忆

大多数智能体在对话结束后会遗忘一切，但 DeerFlow 会记住。

跨会话持久化记忆会保存你的个人资料、偏好和积累的知识。使用得越多，它越懂你——你的写作风格、技术栈和常用流程。记忆存储在本地，完全由你掌控。

## 推荐模型

DeerFlow 对模型无绑定——只要兼容 OpenAI 风格 API 的 LLM 都可以使用。最佳体验来自具备以下能力的模型：

- **长上下文窗口**（10 万+ token）用于深度研究与多步骤任务
- **推理能力**用于自适应规划与复杂拆解
- **多模态输入**用于图像理解与视频解析
- **强工具调用能力**用于可靠的函数调用与结构化输出

## 文档

- [贡献指南](CONTRIBUTING.md) - 开发环境配置与工作流程
- [配置指南](backend/docs/CONFIGURATION.md) - 安装与配置说明
- [架构概览](backend/CLAUDE.md) - 技术架构细节
- [后端架构](backend/README.md) - 后端架构与 API 参考

## 贡献

欢迎贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 获取开发环境、工作流程与贡献规范。

## 许可证

本项目为开源项目，遵循 [MIT License](./LICENSE)。

## 致谢

DeerFlow 建立在开源社区的卓越成果之上。我们由衷感谢所有为 DeerFlow 贡献过的项目与作者，真正“站在巨人的肩膀上”。

特别感谢以下项目的重大贡献：

- **[LangChain](https://github.com/langchain-ai/langchain)**：其出色框架支撑了 LLM 交互与链式调用，实现无缝集成。
- **[LangGraph](https://github.com/langchain-ai/langgraph)**：其在多智能体编排方面的创新使 DeerFlow 的复杂工作流成为可能。

这些项目体现了开源协作的巨大力量，我们很荣幸构建在其基础之上。

### 关键贡献者

衷心感谢 `DeerFlow` 的核心作者，他们的愿景、热情与投入让项目得以落地：

- **[Daniel Walnut](https://github.com/hetaoBackend/)**
- **[Henry Li](https://github.com/magiccube/)**

你们的坚定投入与专业能力，是 DeerFlow 成功的关键驱动力。能够与你们同行，我们深感荣幸。

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=bytedance/deer-flow&type=Date)](https://star-history.com/#bytedance/deer-flow&Date)
