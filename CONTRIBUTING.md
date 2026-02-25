# 贡献指南：DeerFlow

感谢你对 DeerFlow 的兴趣！本指南将帮助你搭建开发环境并理解我们的开发流程。

## 开发环境配置

我们提供两种开发环境。为获得最稳定、最省心的体验，**推荐使用 Docker**。

### 方案一：Docker 开发（推荐）

Docker 提供一致、隔离的环境，并预装所有依赖。你无需在本机安装 Node.js、Python 或 nginx。

#### 前置条件

- Docker Desktop 或 Docker Engine
- pnpm（用于缓存优化）

#### 配置步骤

1. **配置应用**：
   ```bash
   # Copy example configuration
   cp config.example.yaml config.yaml

   # Set your API keys
   export OPENAI_API_KEY="your-key-here"
   # or edit config.yaml directly
   ```

2. **初始化 Docker 环境**（仅第一次需要）：
   ```bash
   make docker-init
   ```
   将会执行：
   - 构建 Docker 镜像
   - 安装前端依赖（pnpm）
   - 安装后端依赖（uv）
   - 与宿主机共享 pnpm 缓存以加速构建

3. **启动开发服务**：
   ```bash
   make docker-start
   ```
   所有服务以热更新模式启动：
   - 前端修改会自动刷新
   - 后端修改会触发自动重启
   - LangGraph 服务支持热更新

4. **访问应用**：
   - Web 界面：http://localhost:2026
   - API Gateway：http://localhost:2026/api/*
   - LangGraph：http://localhost:2026/api/langgraph/*

#### Docker 常用命令

```bash
# Build the custom k3s image (with pre-cached sandbox image)
make docker-init
# Start all services in Docker (localhost:2026)
make docker-start
# Stop Docker development services
make docker-stop
# View Docker development logs
make docker-logs
# View Docker frontend logs
make docker-logs-frontend
# View Docker gateway logs
make docker-logs-gateway
```

#### Docker 架构

```
Host Machine
  ↓
Docker Compose (deer-flow-dev)
  ├→ nginx (port 2026) ← Reverse proxy
  ├→ web (port 3000) ← Frontend with hot-reload
  ├→ api (port 8001) ← Gateway API with hot-reload
  └→ langgraph (port 2024) ← LangGraph server with hot-reload
```

**Docker 开发的优势**：
- ✅ 多机器一致的开发环境
- ✅ 无需本地安装 Node.js、Python、nginx
- ✅ 依赖与服务隔离
- ✅ 清理与重置方便
- ✅ 所有服务支持热更新
- ✅ 更接近生产环境

### 方案二：本地开发

如果你希望直接在本机运行服务：

#### 前置条件

确认已安装所有必需工具：

```bash
make check
```

必需工具：
- Node.js 22+
- pnpm
- uv（Python 包管理器）
- nginx

#### 配置步骤

1. **配置应用**（同 Docker 方案）

2. **安装依赖**：
   ```bash
   make install
   ```

3. **运行开发服务器**（使用 nginx 启动所有服务）：
   ```bash
   make dev
   ```

4. **访问应用**：
   - Web 界面：http://localhost:2026
   - 所有 API 请求会自动通过 nginx 代理

#### 手动控制服务

如果你需要分别启动各服务：

1. **启动后端服务**：
   ```bash
   # Terminal 1: Start LangGraph Server (port 2024)
   cd backend
   make dev

   # Terminal 2: Start Gateway API (port 8001)
   cd backend
   make gateway

   # Terminal 3: Start Frontend (port 3000)
   cd frontend
   pnpm dev
   ```

2. **启动 nginx**：
   ```bash
   make nginx
   # or directly: nginx -c $(pwd)/docker/nginx/nginx.local.conf -g 'daemon off;'
   ```

3. **访问应用**：
   - Web 界面：http://localhost:2026

#### Nginx 配置说明

nginx 配置提供：
- 2026 端口的统一入口
- 将 `/api/langgraph/*` 转发到 LangGraph 服务（2024）
- 将其他 `/api/*` 转发到 Gateway API（8001）
- 将非 API 请求转发到前端（3000）
- 统一处理 CORS
- 支持 SSE/流式响应
- 对长时间请求优化超时设置

## 项目结构

```
deer-flow/
├── config.example.yaml      # Configuration template
├── extensions_config.example.json  # MCP and Skills configuration template
├── Makefile                 # Build and development commands
├── scripts/
│   └── docker.sh           # Docker management script
├── docker/
│   ├── docker-compose-dev.yaml  # Docker Compose configuration
│   └── nginx/
│       ├── nginx.conf      # Nginx config for Docker
│       └── nginx.local.conf # Nginx config for local dev
├── backend/                 # Backend application
│   ├── src/
│   │   ├── gateway/        # Gateway API (port 8001)
│   │   ├── agents/         # LangGraph agents (port 2024)
│   │   ├── mcp/            # Model Context Protocol integration
│   │   ├── skills/         # Skills system
│   │   └── sandbox/        # Sandbox execution
│   ├── docs/               # Backend documentation
│   └── Makefile            # Backend commands
├── frontend/               # Frontend application
│   └── Makefile            # Frontend commands
└── skills/                 # Agent skills
    ├── public/             # Public skills
    └── custom/             # Custom skills
```

## 架构

```
Browser
  ↓
Nginx (port 2026) ← Unified entry point
  ├→ Frontend (port 3000) ← / (non-API requests)
  ├→ Gateway API (port 8001) ← /api/models, /api/mcp, /api/skills, /api/threads/*/artifacts
  └→ LangGraph Server (port 2024) ← /api/langgraph/* (agent interactions)
```

## 开发流程

1. **创建功能分支**：
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **在热更新环境中进行开发修改**

3. **充分测试你的修改**

4. **提交改动**：
   ```bash
   git add .
   git commit -m "feat: description of your changes"
   ```

5. **推送并创建 Pull Request**：
   ```bash
   git push origin feature/your-feature-name
   ```

## 测试

```bash
# Backend tests
cd backend
uv run pytest

# Frontend tests
cd frontend
pnpm test
```

## 代码风格

- **后端（Python）**：使用 `ruff` 进行 lint 与格式化
- **前端（TypeScript）**：使用 ESLint 与 Prettier

## 文档

- [配置指南](backend/docs/CONFIGURATION.md) - 安装与配置
- [架构概览](backend/CLAUDE.md) - 技术架构
- [MCP 配置指南](MCP_SETUP.md) - Model Context Protocol 配置

## 需要帮助？

- 查看已有 [Issues](https://github.com/bytedance/deer-flow/issues)
- 阅读 [Documentation](backend/docs/)
- 在 [Discussions](https://github.com/bytedance/deer-flow/discussions) 中提问

## 许可证

当你向 DeerFlow 贡献代码时，即表示你同意你的贡献内容遵循 [MIT License](./LICENSE)。
