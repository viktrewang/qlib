# Qlib Dev Container 使用指南

这是为 Qlib 项目配置的 Docker 开发容器，允许你在完全隔离的环境中开发和测试 Qlib。

## 前置要求

- **Docker Desktop** for Windows（已安装 ✓）
- **VS Code**
- **VS Code Remote - Containers** 扩展

## 快速开始

### 1. 在 VS Code 中打开项目

```bash
# 在 Windows 宿主机上
cd d:\github\qlib
code .
```

### 2. 连接到容器

- 打开 VS Code 命令面板（`Ctrl+Shift+P`）
- 搜索并运行：`Remote-Containers: Reopen in Container`
- VS Code 将自动构建镜像并启动容器

首次构建可能需要 5-10 分钟，因为需要安装所有依赖。

### 3. 验证环境

在容器终端中运行：

```bash
# 检查 Python 版本
python --version

# 检查 Qlib 安装
python -c "import qlib; print('Qlib installed successfully')"

# 运行测试
pytest tests/ -v
```

## 主要配置说明

### devcontainer.json
- **service**: `qlib` - 使用 docker-compose 中定义的服务
- **workspaceFolder**: `/workspace` - 容器内的工作目录
- **forwardPorts**: 8888, 8080, 5000 - 转发用于 Jupyter、Web 服务和调试的端口
- **postCreateCommand**: 自动安装 qlib 及开发依赖

### docker-compose.yml
- **构建**: 从 `.devcontainer/Dockerfile` 构建镜像
- **卷挂载**: 
  - 项目目录映射到 `/workspace`
  - 缓存卷用于 pip 和 conda 缓存
- **资源限制**: CPU 最多 2 核，内存最多 4GB

### Dockerfile
- **基础镜像**: `continuumio/miniconda3:latest`
- **Python 版本**: 3.10（可通过 `docker-compose.yml` 的 `PYTHON_VERSION` ARG 修改）
- **预装包**:
  - Qlib 核心依赖（numpy, pandas, scipy 等）
  - 开发工具（pytest, black, pylint 等）
  - Jupyter Lab

## 常用操作

### 启动 Jupyter Notebook

在容器终端中：

```bash
jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser --allow-root
```

然后在浏览器中打开：`http://localhost:8888`

### 运行测试

```bash
# 运行所有测试
pytest tests/

# 运行特定测试文件
pytest tests/test_workflow.py -v

# 运行特定测试并显示覆盖率
pytest tests/ --cov=qlib --cov-report=html
```

### 安装额外的包

```bash
pip install <package-name>
```

新安装的包会被持久化到 Docker 卷中。

### 交互式 Python 调试

```bash
ipython
# 或使用 IPython 调试器
ipdb
```

### 查看容器信息

```bash
# 查看容器运行状态
docker ps

# 查看容器日志
docker logs qlib-dev

# 连接到正在运行的容器
docker exec -it qlib-dev /bin/bash
```

## 目录结构

```
.devcontainer/
├── devcontainer.json      # VS Code Dev Container 配置
├── docker-compose.yml     # Docker Compose 配置
├── Dockerfile             # 容器镜像定义
└── .dockerignore          # Docker 构建忽略文件
```

## 性能优化

### 卷挂载性能（Windows 上）

Windows 上的文件系统操作可能较慢。以下优化措施已应用：

1. **缓存卷**: 使用命名卷存储 pip 和 conda 缓存，避免宿主机文件系统操作
2. **`:cached` 标志**: 减少从容器到宿主机的文件同步延迟

### 额外优化建议

如果性能仍然较慢，可以：

- 增加 `.dockerignore` 中的忽略模式
- 在 `docker-compose.yml` 中调整资源限制
- 使用 WSL 2 作为 Docker Desktop 的后端

## 故障排除

### 容器无法启动

```bash
# 检查 Docker 日志
docker logs qlib-dev

# 重建容器
docker-compose -f .devcontainer/docker-compose.yml build --no-cache
```

### Qlib 安装失败

如果 C++ 扩展编译失败：

```bash
# 确保安装了编译工具
apt-get update && apt-get install -y build-essential cmake

# 重新安装 qlib
pip install -e . --force-reinstall --no-cache-dir
```

### 连接超时

如果 Jupyter 连接超时，尝试：

```bash
# 检查端口是否正确转发
docker ps

# 手动启动 Jupyter
jupyter notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser
```

## 进阶配置

### 修改 Python 版本

编辑 `docker-compose.yml`：

```yaml
args:
  PYTHON_VERSION: "3.11"  # 改为 3.8, 3.9, 3.11, 3.12 等
```

然后重建容器。

### 添加系统包

编辑 `.devcontainer/Dockerfile`，在 `apt-get install` 行添加包名：

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    git \
    # 添加你需要的包
    ...
```

### 自定义 Jupyter 配置

编辑 Dockerfile 中的 Jupyter 配置部分：

```dockerfile
RUN jupyter notebook --generate-config && \
    echo "c.NotebookApp.default_url = '/lab'" >> ~/.jupyter/jupyter_notebook_config.py
```

## 资源清理

### 停止容器

```bash
docker-compose -f .devcontainer/docker-compose.yml down
```

### 删除镜像

```bash
docker rmi qlib-dev
```

### 完全清理

```bash
docker-compose -f .devcontainer/docker-compose.yml down -v  # 删除卷
docker system prune -a  # 清理未使用的镜像和容器
```

## 进一步阅读

- [VS Code Dev Containers 文档](https://code.visualstudio.com/docs/remote/containers)
- [Docker 官方文档](https://docs.docker.com/)
- [Qlib 官方文档](https://qlib.readthedocs.io/)

## 许可证

按照 Qlib 项目许可证。
