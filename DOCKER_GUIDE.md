# Docker 使用指南

## Docker 简介

Docker 是一个开源的容器化平台，可以将应用程序及其依赖项打包到轻量级、可移植的容器中，确保应用在任何环境中都能一致运行。

## 核心概念

### 1. 镜像 (Image)
- 只读模板，包含创建容器所需的所有内容（代码、运行时、库、环境变量、配置文件）
- 类似于虚拟机的快照

### 2. 容器 (Container)
- 镜像的运行实例
- 轻量级、独立的可执行软件包
- 包含运行应用所需的一切

### 3. Dockerfile
- 用于构建镜像的文本文件
- 包含一系列指令和命令

### 4. Docker Compose
- 用于定义和运行多容器应用的工具
- 使用 YAML 文件配置应用服务

## 安装 Docker

### macOS
```bash
# 下载并安装 Docker Desktop for Mac
# https://www.docker.com/products/docker-desktop

# 验证安装
docker --version
docker-compose --version
```

### Linux (Ubuntu/Debian)
```bash
# 更新包索引
sudo apt-get update

# 安装依赖
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

# 添加 Docker 官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 添加 Docker 仓库
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# 安装 Docker
sudo apt-get update
sudo apt-get install docker-ce

# 启动 Docker 服务
sudo systemctl start docker
sudo systemctl enable docker

# 将当前用户添加到 docker 组（避免每次使用 sudo）
sudo usermod -aG docker $USER
```

## 基本命令

### 镜像操作

```bash
# 搜索镜像
docker search ubuntu

# 拉取镜像
docker pull ubuntu:latest
docker pull nginx:alpine

# 查看本地镜像
docker images
docker image ls

# 删除镜像
docker rmi ubuntu:latest
docker image rm ubuntu:latest

# 构建镜像
docker build -t myapp:v1.0 .

# 查看镜像详情
docker inspect ubuntu:latest

# 导出镜像
docker save -o ubuntu.tar ubuntu:latest

# 导入镜像
docker load -i ubuntu.tar
```

### 容器操作

```bash
# 运行容器
docker run ubuntu echo "Hello Docker"
docker run -it ubuntu /bin/bash  # 交互式运行
docker run -d nginx  # 后台运行
docker run -d -p 8080:80 nginx  # 端口映射
docker run -d --name my-nginx nginx  # 指定容器名称

# 查看运行中的容器
docker ps
docker container ls

# 查看所有容器（包括停止的）
docker ps -a

# 停止容器
docker stop <container_id>
docker stop my-nginx

# 启动已停止的容器
docker start <container_id>

# 重启容器
docker restart <container_id>

# 删除容器
docker rm <container_id>
docker rm -f <container_id>  # 强制删除运行中的容器

# 进入运行中的容器
docker exec -it <container_id> /bin/bash
docker attach <container_id>

# 查看容器日志
docker logs <container_id>
docker logs -f <container_id>  # 实时查看

# 查看容器资源使用情况
docker stats <container_id>

# 查看容器进程
docker top <container_id>

# 复制文件
docker cp file.txt <container_id>:/path/to/destination
docker cp <container_id>:/path/to/file.txt ./
```

### 网络操作

```bash
# 查看网络
docker network ls

# 创建网络
docker network create my-network

# 连接容器到网络
docker network connect my-network <container_id>

# 断开容器网络
docker network disconnect my-network <container_id>

# 删除网络
docker network rm my-network
```

### 数据卷操作

```bash
# 查看数据卷
docker volume ls

# 创建数据卷
docker volume create my-volume

# 查看数据卷详情
docker volume inspect my-volume

# 删除数据卷
docker volume rm my-volume

# 使用数据卷运行容器
docker run -d -v my-volume:/app/data nginx
```

## Dockerfile 编写

### 基本结构

```dockerfile
# 基础镜像
FROM ubuntu:20.04

# 维护者信息
LABEL maintainer="your-email@example.com"

# 设置环境变量
ENV APP_HOME=/app
ENV NODE_ENV=production

# 设置工作目录
WORKDIR $APP_HOME

# 复制文件
COPY . .

# 安装依赖
RUN apt-get update && apt-get install -y \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# 安装应用依赖
RUN npm install

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# 启动命令
CMD ["npm", "start"]
```

### 常用指令

- `FROM`: 指定基础镜像
- `LABEL`: 添加元数据
- `ENV`: 设置环境变量
- `WORKDIR`: 设置工作目录
- `COPY`: 复制文件或目录
- `ADD`: 添加文件（支持 URL 和自动解压 tar）
- `RUN`: 执行命令
- `EXPOSE`: 声明端口
- `CMD`: 容器启动时执行的命令
- `ENTRYPOINT`: 配置容器为可执行程序
- `HEALTHCHECK`: 健康检查
- `ARG`: 构建时的变量
- `VOLUME`: 创建挂载点

### 最佳实践

```dockerfile
# 使用多阶段构建减小镜像大小
FROM node:14 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

# 使用 .dockerignore 排除不需要的文件
# .dockerignore 文件内容：
# node_modules
# npm-debug.log
# .git
# .env
# README.md
```

## Docker Compose

### docker-compose.yml 示例

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      - db
    volumes:
      - ./app:/app
    networks:
      - app-network

  db:
    image: postgres:13
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Docker Compose 命令

```bash
# 启动服务
docker-compose up
docker-compose up -d  # 后台运行

# 停止服务
docker-compose down

# 停止并删除数据卷
docker-compose down -v

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs
docker-compose logs -f web

# 重启服务
docker-compose restart

# 构建服务
docker-compose build

# 进入容器
docker-compose exec web /bin/bash

# 运行一次性命令
docker-compose run web npm install

# 暂停/恢复服务
docker-compose pause
docker-compose unpause

# 查看服务进程
docker-compose top
```

## 实用场景示例

### 1. 运行 Nginx Web 服务器

```bash
# 创建目录
mkdir -p ~/nginx/html

# 创建测试页面
echo "<h1>Hello Docker Nginx</h1>" > ~/nginx/html/index.html

# 运行 Nginx 容器
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v ~/nginx/html:/usr/share/nginx/html \
  nginx:alpine

# 访问 http://localhost:8080
```

### 2. 运行 MySQL 数据库

```bash
docker run -d \
  --name mysql-db \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=mydb \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

# 连接到 MySQL
docker exec -it mysql-db mysql -uroot -prootpassword
```

### 3. 运行 Python 应用

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

```bash
# 构建并运行
docker build -t my-python-app .
docker run -d -p 5000:5000 my-python-app
```

### 4. 运行 Node.js 应用

```dockerfile
# Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

USER node
CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
```

## 清理 Docker 资源

```bash
# 删除所有停止的容器
docker container prune

# 删除所有未使用的镜像
docker image prune -a

# 删除所有未使用的数据卷
docker volume prune

# 删除所有未使用的网络
docker network prune

# 一键清理所有未使用的资源
docker system prune -a

# 查看磁盘使用情况
docker system df
```

## 调试技巧

```bash
# 查看容器详细信息
docker inspect <container_id>

# 查看容器实时日志
docker logs -f --tail 100 <container_id>

# 查看容器资源使用
docker stats <container_id>

# 导出容器文件系统
docker export <container_id> > container.tar

# 查看镜像历史
docker history <image_name>

# 保存容器为镜像
docker commit <container_id> my-new-image:v1.0
```

## 安全最佳实践

1. **使用官方镜像**：优先使用 Docker Hub 官方镜像
2. **指定镜像版本**：避免使用 `latest` 标签
3. **最小化镜像**：使用 Alpine 版本减小攻击面
4. **不使用 root 用户**：在 Dockerfile 中指定非 root 用户
5. **扫描漏洞**：使用 `docker scan` 检查镜像漏洞
6. **限制资源**：使用 `--cpus` 和 `--memory` 限制容器资源
7. **使用只读文件系统**：`--read-only` 标志
8. **不存储敏感信息**：使用 Docker Secrets 或环境变量

```bash
# 示例：安全运行容器
docker run -d \
  --name secure-app \
  --read-only \
  --cap-drop ALL \
  --cap-add CHOWN \
  --pids-limit 100 \
  --memory="512m" \
  --cpus="0.5" \
  myapp:latest
```

## 常见问题解决

### 1. 权限问题
```bash
# 将用户添加到 docker 组
sudo usermod -aG docker $USER
# 重新登录生效
```

### 2. 磁盘空间不足
```bash
# 清理未使用的资源
docker system prune -a --volumes

# 移动 Docker 数据目录
sudo systemctl stop docker
sudo mv /var/lib/docker /new/path
sudo ln -s /new/path /var/lib/docker
sudo systemctl start docker
```

### 3. 网络问题
```bash
# 重建 Docker 网络
docker network prune
docker network create bridge

# 使用 host 网络模式
docker run --network host myapp
```

### 4. 容器无法启动
```bash
# 查看详细错误信息
docker logs <container_id>

# 交互式调试
docker run -it --entrypoint /bin/bash myapp
```

## 监控和日志

```bash
# 查看容器资源使用
docker stats

# 查看容器进程
docker top <container_id>

# 查看容器端口映射
docker port <container_id>

# 实时监控事件
docker events

# 使用 Prometheus + Grafana 监控
# 使用 ELK Stack 收集日志
```

## 进阶主题

### 1. 多阶段构建
减小最终镜像大小，分离构建和运行环境

### 2. Docker Swarm
Docker 原生的容器编排工具

### 3. Kubernetes
生产级容器编排平台

### 4. CI/CD 集成
与 Jenkins、GitLab CI、GitHub Actions 集成

### 5. 私有镜像仓库
搭建 Harbor、Nexus 等私有镜像仓库

## 参考资源

- [Docker 官方文档](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Compose 文档](https://docs.docker.com/compose/)
- [Dockerfile 最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Docker 安全最佳实践](https://docs.docker.com/engine/security/)
