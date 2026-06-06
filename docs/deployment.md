# 学生成绩管理系统 - 部署文档

> **文档版本：** V1.0  
> **创建日期：** 2026-06-06  
> **负责人：** DevOps Agent  
> **文档状态：** 已完成

---

## 1. 部署概述

### 1.1 部署架构

本系统采用 Docker 容器化部署，包含以下服务：

| 服务 | 技术栈 | 端口 | 说明 |
|------|--------|------|------|
| **后端服务** | FastAPI + Uvicorn | 8000 | RESTful API 服务 |
| **前端服务** | Vue 3 + Nginx | 80 | 静态资源服务 + API 代理 |
| **数据库** | SQLite | - | 嵌入式数据库，数据持久化到 Docker Volume |

### 1.2 系统要求

| 要求项 | 最低配置 | 推荐配置 |
|--------|---------|---------|
| **操作系统** | Linux / macOS / Windows (WSL2) | Ubuntu 22.04 LTS |
| **CPU** | 2 核 | 4 核 |
| **内存** | 2 GB | 4 GB |
| **磁盘** | 10 GB | 20 GB |
| **Docker** | 20.10+ | 最新稳定版 |
| **Docker Compose** | 2.0+ | 最新稳定版 |

---

## 2. 快速部署

### 2.1 一键部署（推荐）

```bash
# 1. 克隆项目
git clone <repository-url>
cd student-grade-system

# 2. 配置环境变量
cp deployment/.env.example deployment/.env
# 编辑 .env 文件，根据需要修改配置

# 3. 一键部署
cd deployment
make deploy

# 4. 健康检查
make health
```

### 2.2 访问应用

| 服务 | 地址 | 说明 |
|------|------|------|
| **前端页面** | http://localhost | 用户界面 |
| **后端 API** | http://localhost:8000 | API 服务 |
| **API 文档** | http://localhost:8000/docs | Swagger UI |
| **健康检查** | http://localhost:8000/health | 后端健康状态 |

---

## 3. 详细部署步骤

### 3.1 环境准备

#### 3.1.1 安装 Docker

**Ubuntu/Debian:**
```bash
# 更新包索引
sudo apt-get update

# 安装依赖
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

# 添加 Docker 官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker 仓库
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 将当前用户添加到 docker 组
sudo usermod -aG docker $USER
newgrp docker

# 验证安装
docker --version
docker compose version
```

**macOS:**
```bash
# 使用 Homebrew 安装
brew install --cask docker

# 或者下载 Docker Desktop for Mac
# https://www.docker.com/products/docker-desktop/
```

**Windows:**
```bash
# 下载 Docker Desktop for Windows
# https://www.docker.com/products/docker-desktop/

# 确保启用 WSL2 后端
```

#### 3.1.2 验证 Docker 安装

```bash
# 检查 Docker 版本
docker --version

# 检查 Docker Compose 版本
docker compose version

# 运行测试容器
docker run hello-world
```

### 3.2 项目配置

#### 3.2.1 克隆项目

```bash
git clone <repository-url>
cd student-grade-system
```

#### 3.2.2 配置环境变量

```bash
# 进入部署目录
cd deployment

# 复制环境变量模板
cp .env.example .env

# 编辑环境变量
nano .env  # 或使用其他编辑器
```

**环境变量说明：**

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `APP_NAME` | 学生成绩管理系统 | 应用名称 |
| `APP_VERSION` | 1.0.0 | 应用版本 |
| `DEBUG` | false | 调试模式 |
| `DATABASE_URL` | sqlite:///./data/grades.db | 数据库连接字符串 |
| `BACKEND_PORT` | 8000 | 后端服务端口 |
| `FRONTEND_PORT` | 80 | 前端服务端口 |
| `LOG_LEVEL` | INFO | 日志级别 |

#### 3.2.3 构建镜像

```bash
# 构建所有镜像
make build

# 或者使用 docker-compose
docker-compose build
```

### 3.3 启动服务

```bash
# 启动所有服务（后台运行）
make up

# 或者使用 docker-compose
docker-compose up -d
```

### 3.4 验证部署

```bash
# 查看容器状态
make ps

# 健康检查
make health

# 查看日志
make logs
```

---

## 4. 开发环境部署

### 4.1 启动开发环境

```bash
# 进入部署目录
cd deployment

# 启动开发环境
make dev

# 查看开发环境日志
make dev-logs
```

### 4.2 开发环境访问

| 服务 | 地址 | 说明 |
|------|------|------|
| **前端（Vite）** | http://localhost:5173 | 开发服务器（支持热更新） |
| **后端 API** | http://localhost:8000 | API 服务（支持热更新） |
| **API 文档** | http://localhost:8000/docs | Swagger UI |

### 4.3 开发环境特性

- ✅ 代码热更新：修改代码后自动刷新
- ✅ 卷挂载：本地代码同步到容器
- ✅ 调试模式：详细的错误日志

### 4.4 停止开发环境

```bash
make dev-down
```

---

## 5. 常用操作

### 5.1 服务管理

```bash
# 启动服务
make up

# 停止服务
make down

# 重启服务
make restart

# 查看容器状态
make ps

# 查看资源使用
make stats
```

### 5.2 日志管理

```bash
# 查看所有服务日志（实时）
make logs

# 查看后端日志
make logs-backend

# 查看前端日志
make logs-frontend
```

### 5.3 容器操作

```bash
# 进入后端容器
make backend-shell

# 进入前端容器
make frontend-shell
```

### 5.4 数据库管理

```bash
# 备份数据库
make backup

# 恢复数据库
make restore FILE=grades.db.backup.20260101120000
```

### 5.5 更新部署

```bash
# 拉取最新代码并重新部署
make update
```

---

## 6. 目录结构

```
student-grade-system/
├── src/                            # 后端源代码
├── frontend/                       # 前端源代码
│   ├── src/
│   ├── package.json
│   └── vite.config.ts
├── data/                           # 数据目录（持久化）
├── docs/                           # 文档目录
├── deployment/                     # 部署配置目录
│   ├── Dockerfile                  # 后端 Docker 配置
│   ├── .dockerignore               # 后端 Docker 忽略文件
│   ├── docker-entrypoint.sh        # 后端入口脚本
│   ├── frontend/                   # 前端部署配置
│   │   ├── Dockerfile              # 前端 Docker 配置
│   │   ├── .dockerignore           # 前端 Docker 忽略文件
│   │   └── nginx.conf              # Nginx 配置
│   ├── docker-compose.yml          # 生产环境配置
│   ├── docker-compose.dev.yml      # 开发环境配置
│   ├── .env.example                # 环境变量模板
│   ├── Makefile                    # 常用命令
│   └── docs/
│       └── deployment.md           # 部署文档（本文件）
├── requirements.txt                # Python 依赖
└── README.md                       # 项目说明
```

---

## 7. 配置说明

### 7.1 Nginx 配置

Nginx 配置文件位于 `deployment/frontend/nginx.conf`，主要功能：

| 功能 | 说明 |
|------|------|
| **Gzip 压缩** | 减少传输体积，提升加载速度 |
| **静态资源缓存** | 浏览器缓存 1 年，减少请求 |
| **API 代理** | 将 `/api/` 请求代理到后端服务 |
| **Vue Router** | 支持 History 模式路由 |
| **健康检查** | `/health` 端点用于容器健康检查 |

### 7.2 Docker Compose 配置

**生产环境 (`docker-compose.yml`)：**
- 后端服务：FastAPI + Uvicorn
- 前端服务：Nginx 静态服务
- 数据持久化：Docker Volume
- 健康检查：自动重启

**开发环境 (`docker-compose.dev.yml`)：**
- 代码卷挂载：支持热更新
- 调试模式：详细日志
- 开发端口：5173 (Vite) + 8000 (API)

---

## 8. 故障排查

### 8.1 常见问题

#### 问题 1：端口被占用

**错误信息：**
```
Error: Bind for 0.0.0.0:8000 failed: port is already allocated
```

**解决方案：**
```bash
# 查找占用端口的进程
lsof -i :8000
# 或
netstat -tulpn | grep 8000

# 停止占用端口的进程，或修改 .env 中的端口配置
BACKEND_PORT=8001
```

#### 问题 2：Docker 构建失败

**错误信息：**
```
ERROR: failed to solve: process "/bin/sh -c pip install ..." did not complete successfully
```

**解决方案：**
```bash
# 清理 Docker 缓存
docker system prune -f

# 重新构建
make build

# 如果网络问题，可以配置国内镜像源
# 在 Dockerfile 中添加：
# RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 问题 3：容器启动失败

**错误信息：**
```
Error: Container exited with code 1
```

**解决方案：**
```bash
# 查看容器日志
make logs-backend
# 或
make logs-frontend

# 进入容器调试
make backend-shell
```

#### 问题 4：数据库初始化失败

**错误信息：**
```
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) unable to open database file
```

**解决方案：**
```bash
# 检查数据目录权限
ls -la data/

# 确保数据目录存在
mkdir -p data

# 检查 Docker Volume
docker volume ls
docker volume inspect student-grade-data
```

#### 问题 5：前端无法访问后端 API

**错误信息：**
```
GET http://localhost/api/v1/students net::ERR_CONNECTION_REFUSED
```

**解决方案：**
```bash
# 检查后端服务是否正常
curl http://localhost:8000/health

# 检查 Nginx 配置
make frontend-shell
cat /etc/nginx/conf.d/default.conf

# 检查容器网络
docker network ls
docker network inspect student-grade-network
```

### 8.2 日志查看

```bash
# 查看所有服务日志
make logs

# 查看特定服务日志
docker-compose logs -f backend
docker-compose logs -f frontend

# 查看最近 100 行日志
docker-compose logs --tail=100 backend
```

### 8.3 容器调试

```bash
# 进入后端容器
make backend-shell

# 在容器内检查环境变量
env

# 在容器内测试 API
curl http://localhost:8000/health

# 退出容器
exit
```

---

## 9. 性能优化

### 9.1 镜像优化

- 使用多阶段构建减小镜像体积
- 使用 Alpine 基础镜像
- 合并 RUN 指令减少层数
- 使用 `.dockerignore` 排除不需要的文件

### 9.2 构建缓存

```bash
# 使用 BuildKit 加速构建
DOCKER_BUILDKIT=1 docker-compose build

# 清理构建缓存
docker builder prune
```

### 9.3 运行时优化

- 配置适当的健康检查间隔
- 使用 Docker Volume 持久化数据
- 配置日志轮转避免磁盘占满

---

## 10. 安全建议

### 10.1 容器安全

- 使用非 root 用户运行应用
- 定期更新基础镜像
- 扫描镜像漏洞
- 限制容器资源使用

### 10.2 网络安全

- 使用内部网络隔离服务
- 仅暴露必要端口
- 配置防火墙规则
- 使用 HTTPS（生产环境）

### 10.3 数据安全

- 定期备份数据库
- 加密敏感配置
- 使用 Docker Secrets 管理密钥

---

## 11. 备份与恢复

### 11.1 数据备份

```bash
# 使用 Makefile 备份
make backup

# 手动备份
docker-compose exec backend cp /app/data/grades.db /app/data/grades.db.backup.$(date +%Y%m%d%H%M%S)

# 从容器复制备份到宿主机
docker cp student-grade-backend:/app/data/grades.db.backup.20260101120000 ./backup/
```

### 11.2 数据恢复

```bash
# 使用 Makefile 恢复
make restore FILE=grades.db.backup.20260101120000

# 手动恢复
docker cp ./backup/grades.db.backup.20260101120000 student-grade-backend:/app/data/grades.db
docker-compose restart backend
```

### 11.3 自动备份

可以使用 cron 定时任务实现自动备份：

```bash
# 编辑 crontab
crontab -e

# 添加每日凌晨 2 点备份
0 2 * * * cd /path/to/student-grade-system/deployment && make backup
```

---

## 12. 监控与运维

### 12.1 健康检查

```bash
# 手动健康检查
make health

# 检查后端
curl http://localhost:8000/health

# 检查前端
curl http://localhost:80/health
```

### 12.2 资源监控

```bash
# 查看容器资源使用
make stats

# 实时监控
docker stats
```

### 12.3 日志管理

```bash
# 配置日志轮转（在 docker-compose.yml 中）
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

---

## 附录

### A. Makefile 命令速查

| 命令 | 说明 |
|------|------|
| `make help` | 显示帮助信息 |
| `make build` | 构建所有镜像 |
| `make up` | 启动所有服务 |
| `make down` | 停止所有服务 |
| `make restart` | 重启所有服务 |
| `make logs` | 查看所有日志 |
| `make dev` | 启动开发环境 |
| `make dev-down` | 停止开发环境 |
| `make ps` | 查看容器状态 |
| `make health` | 健康检查 |
| `make backup` | 备份数据库 |
| `make update` | 更新部署 |
| `make clean` | 清理所有资源 |

### B. Docker Compose 命令速查

| 命令 | 说明 |
|------|------|
| `docker-compose up -d` | 后台启动服务 |
| `docker-compose down` | 停止并删除容器 |
| `docker-compose ps` | 查看容器状态 |
| `docker-compose logs -f` | 实时查看日志 |
| `docker-compose exec backend bash` | 进入后端容器 |
| `docker-compose build` | 构建镜像 |
| `docker-compose pull` | 拉取最新镜像 |

### C. 相关文档

- [架构文档](./architecture.md) - 系统架构设计
- [API 文档](./api-spec.md) - API 接口规范
- [产品需求文档](./prd.md) - 产品需求说明

---

> **文档结束**  
> 如有问题，请联系 DevOps 团队
