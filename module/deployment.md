# 部署指南

本文档提供详细的 Docker 和 Docker Compose 部署说明。

## Docker 部署

Docker 是最简单快速的部署方式，一条命令即可启动服务。

### 快速开始

```bash
docker run -d \
  --name config-flow \
  -p 80:80 \
  -v $(pwd)/data:/data \
  -e ADMIN_USERNAME=admin \
  -e ADMIN_PASSWORD=admin123 \
  -e JWT_SECRET_KEY=your-secret-key-please-change-in-production \
  dooix/config-flow:latest
```

访问 `http://localhost` 即可使用。

> 请把 `ADMIN_PASSWORD` 和 `JWT_SECRET_KEY` 替换为更安全的值，生产环境务必更新凭据。

### 参数说明

**端口映射：**
- `-p 80:80` - 将容器的 80 端口映射到宿主机
- 容器内部 Nginx 暴露 80 端口，Flask API 运行在内部 5001 端口
- 自定义端口：`-p 8080:80` 映射到 8080 端口

**数据持久化：**
- `-v $(pwd)/data:/data` - 挂载数据目录
- 配置数据存储在 `/data/config.json`
- 规则文件存储在 `/data/rules/`
- 数据持久化确保重启容器不丢失数据

**容器管理：**
- `--name config-flow` - 容器名称
- `-d` - 后台运行

### 常用操作

```bash
# 查看状态
docker ps | grep config-flow

# 查看日志
docker logs config-flow
docker logs -f config-flow  # 实时日志

# 容器管理
docker stop config-flow
docker start config-flow
docker restart config-flow

# 删除容器
docker stop config-flow
docker rm config-flow

# 更新版本
docker stop config-flow
docker rm config-flow
docker pull dooix/config-flow:latest
docker run -d --name config-flow -p 80:80 -v $(pwd)/data:/data \
  -e ADMIN_USERNAME=admin -e ADMIN_PASSWORD=admin123 -e JWT_SECRET_KEY=your-secret-key-please-change-in-production dooix/config-flow:latest
```

### 自定义端口

```bash
# 8080 端口
docker run -d --name config-flow -p 8080:80 -v $(pwd)/data:/data \
  -e ADMIN_USERNAME=admin -e ADMIN_PASSWORD=admin123 -e JWT_SECRET_KEY=your-secret-key-please-change-in-production dooix/config-flow:latest

# 3000 端口
docker run -d --name config-flow -p 3000:80 -v $(pwd)/data:/data \
  -e ADMIN_USERNAME=admin -e ADMIN_PASSWORD=admin123 -e JWT_SECRET_KEY=your-secret-key-please-change-in-production dooix/config-flow:latest
```

## Docker Compose 部署

Docker Compose 适合长期维护和管理。

### 快速开始

**1. 创建 docker-compose.yml**

```yaml
version: '3.8'

services:
  config-flow:
    image: dooix/config-flow:latest
    container_name: config-flow
    ports:
      - "80:80"
    volumes:
      - ./data:/data
    environment:
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=admin123
      - JWT_SECRET_KEY=your-secret-key-please-change-in-production
    restart: unless-stopped
```

**2. 启动服务**

```bash
docker-compose up -d
```

访问 `http://localhost` 即可使用。

> 请把 `ADMIN_PASSWORD` 和 `JWT_SECRET_KEY` 替换为更安全的值，生产环境务必更新凭据。

### 配置说明

- **version**: Docker Compose 文件格式版本
- **image**: Docker 镜像，`latest` 为最新版本
- **ports**: 端口映射，格式为 `宿主机:容器`
- **volumes**: 数据目录挂载，支持相对路径或绝对路径
- **restart**: 重启策略
  - `unless-stopped`: 除非手动停止，否则自动重启
  - `always`: 始终重启
  - `no`: 不自动重启
  - `on-failure`: 仅失败时重启

### 常用操作

```bash
# 启动服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs
docker-compose logs -f  # 实时日志

# 服务管理
docker-compose stop
docker-compose start
docker-compose restart

# 停止并删除
docker-compose down

# 更新版本
docker-compose pull
docker-compose up -d
```

### 配置示例

**自定义端口：**
```yaml
version: '3.8'

services:
  config-flow:
    image: dooix/config-flow:latest
    container_name: config-flow
    ports:
      - "8080:80"  # 使用 8080 端口
    volumes:
      - ./data:/data
    environment:
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=admin123
      - JWT_SECRET_KEY=your-secret-key-please-change-in-production
    restart: unless-stopped
```

**指定版本：**
```yaml
version: '3.8'

services:
  config-flow:
    image: dooix/config-flow:v1.0.1  # 使用特定版本
    container_name: config-flow
    ports:
      - "80:80"
    volumes:
      - ./data:/data
    environment:
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=admin123
      - JWT_SECRET_KEY=your-secret-key-please-change-in-production
    restart: unless-stopped
```

**绝对路径挂载：**
```yaml
version: '3.8'

services:
  config-flow:
    image: dooix/config-flow:latest
    container_name: config-flow
    ports:
      - "80:80"
    volumes:
      - /opt/config-flow/data:/data  # 绝对路径
    environment:
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=admin123
      - JWT_SECRET_KEY=your-secret-key-please-change-in-production
    restart: unless-stopped
```

## 部署验证

### 1. 检查容器状态

```bash
# Docker
docker ps | grep config-flow

# Docker Compose
docker-compose ps
```

确认状态为 `Up`

### 2. 检查端口

```bash
netstat -tuln | grep 80
# 或
ss -tuln | grep 80
```

### 3. 访问界面

- 本地：`http://localhost` 或 `http://localhost:端口号`
- 远程：`http://服务器IP` 或 `http://服务器IP:端口号`

### 4. 检查数据目录

```bash
ls -la data/
```

应该看到：
```
data/
├── config.json    # 配置文件
└── rules/         # 规则目录
```

## 故障排查

### 容器无法启动

**检查端口占用：**
```bash
netstat -tuln | grep 80
lsof -i :80
```

**解决方案：**
```bash
# 修改映射端口
docker run -d --name config-flow -p 8080:80 -v $(pwd)/data:/data \
  -e ADMIN_USERNAME=admin -e ADMIN_PASSWORD=admin123 -e JWT_SECRET_KEY=your-secret-key-please-change-in-production dooix/config-flow:latest
```

**检查 Docker 服务：**
```bash
systemctl status docker
```

### 无法访问界面

**检查日志：**
```bash
docker logs config-flow
```

**检查防火墙（CentOS/RHEL）：**
```bash
firewall-cmd --list-ports
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --reload
```

**检查防火墙（Ubuntu/Debian）：**
```bash
ufw status
ufw allow 80/tcp
```

### 数据丢失

**确认挂载：**
```bash
docker inspect config-flow | grep Mounts -A 10
```

**解决方案：**
- 始终使用 `-v` 挂载数据目录
- 定期备份 `data/config.json`
- 使用 Web 界面"导出配置"功能

## 生产环境建议

### 安全配置
- 配置 HTTPS（使用 Nginx/Caddy 反向代理）
- 添加访问认证
- 限制 IP 访问范围

### 性能优化
- 使用 SSD 存储数据
- 定期清理 Docker 日志
- 设置容器资源限制

### 监控告警
- 配置 Docker 健康检查
- 设置日志收集
- 添加服务监控

### 备份策略
- 定期备份 data 目录
- 使用配置导出功能
- 保留多版本备份
