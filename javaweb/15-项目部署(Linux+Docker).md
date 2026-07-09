# 15 - 项目部署（Linux + Docker）

## 1. Linux 基础

### 1.1 常用命令

| 分类 | 命令 | 说明 |
|------|------|------|
| 目录操作 | `cd` `ls` `pwd` `mkdir` `rm -rf` | 切换/查看/创建/删除 |
| 文件操作 | `touch` `cat` `vi/vim` `cp` `mv` `rm` | 创建/查看/编辑/复制/移动/删除 |
| 文件查看 | `tail -f` `grep` `less` `head` | 实时查看/搜索/分页 |
| 权限 | `chmod 755` `chown` | 修改权限/所有者 |
| 进程 | `ps -ef` `kill -9` `netstat -tlnp` | 查看进程/杀进程/看端口 |
| 压缩 | `tar -zxvf` `tar -zcvf` | 解压/压缩 |
| 系统 | `top` `df -h` `free -h` | 系统负载/磁盘/内存 |

**vim 基本操作**：

```
普通模式：dd(删行) yy(复制) p(粘贴) /搜索 n(下一个) :wq(保存退出) :q!(强制退出)
插入模式：按 i 进入，Esc 退出
```

### 1.2 安装软件 — yum/dnf

```bash
# CentOS / RHEL
yum install -y nginx            # 安装
yum remove nginx                # 卸载
systemctl start nginx           # 启动
systemctl enable nginx          # 开机自启
systemctl status nginx          # 查看状态

# Ubuntu / Debian 用 apt
apt update && apt install -y nginx
```

---

## 2. 项目部署流程

### 2.1 传统部署（Jar 包）

```bash
# 1. 打包
mvn clean package -DskipTests
# → target/tlias-web-1.0.jar

# 2. 上传到服务器
scp target/tlias-web-1.0.jar root@server:/opt/tlias/

# 3. 服务器上运行
java -jar tlias-web-1.0.jar                    # 前台运行
nohup java -jar tlias-web-1.0.jar > app.log &  # 后台运行
# nohup: 忽略挂断信号（关闭终端不杀进程）
# &: 后台运行
# > app.log: 输出到日志文件

# 4. 停止
ps -ef | grep java                    # 找到进程 ID
kill -9 <PID>

# 5. 查看日志
tail -f app.log
```

### 2.2 环境变量与多环境

```bash
# 启动时指定环境
java -jar tlias-web-1.0.jar --spring.profiles.active=prod

# 或者通过环境变量
export SPRING_PROFILES_ACTIVE=prod
java -jar tlias-web-1.0.jar
```

### 2.3 Nginx 反向代理

```nginx
# /etc/nginx/conf.d/tlias.conf
server {
    listen       80;
    server_name  tlias.example.com;

    # 前端静态资源
    location / {
        root   /opt/tlias/dist;
        index  index.html;
        try_files $uri $uri/ /index.html;  # Vue History 模式
    }

    # 后端 API 代理
    location /api/ {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

```
用户 → Nginx(:80) → 前端页面（/ → /opt/tlias/dist/index.html）
                  → 后端API（/api/ → http://localhost:8080/）
```

---

## 3. Docker 部署

### 3.1 Docker 核心概念

```
镜像 (Image)   → 集装箱   → 应用的"安装包"（只读模板）
容器 (Container) → 运行的集装箱 → 镜像的运行实例
仓库 (Registry) → 集装箱码头 → 存储和分发镜像（Docker Hub / 阿里云）
```

### 3.2 常用命令

```bash
# 镜像
docker pull openjdk:17         # 拉取镜像
docker images                  # 查看本地镜像
docker rmi <image_id>          # 删除镜像

# 容器
docker run -d -p 8080:8080 --name tlias-app openjdk:17 java -jar app.jar
docker ps                      # 查看运行中的容器
docker ps -a                   # 查看所有容器（含停止的）
docker stop <container>        # 停止
docker start <container>       # 启动
docker restart <container>     # 重启
docker rm <container>          # 删除
docker logs -f <container>     # 查看日志
docker exec -it <container> /bin/bash  # 进入容器
```

### 3.3 Dockerfile — 构建自定义镜像

```dockerfile
# Dockerfile
FROM openjdk:17-jdk-alpine
LABEL maintainer="tlias@example.com"

# 设置工作目录
WORKDIR /app

# 复制 jar 包
COPY target/tlias-web-1.0.jar app.jar

# 暴露端口
EXPOSE 8080

# 启动命令
ENTRYPOINT ["java", "-jar", "app.jar", "--spring.profiles.active=prod"]
```

```bash
# 构建镜像
docker build -t tlias-app:1.0 .

# 运行容器
docker run -d -p 8080:8080 --name tlias tlias-app:1.0

# 浏览器访问 http://服务器IP:8080
```

### 3.4 Docker Compose — 编排多容器

```yaml
# docker-compose.yml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: tlias-mysql
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: tlias
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./sql/init.sql:/docker-entrypoint-initdb.d/init.sql

  app:
    build: .
    container_name: tlias-app
    ports:
      - "8080:8080"
    depends_on:
      - mysql
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/tlias
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 123456

  nginx:
    image: nginx:alpine
    container_name: tlias-nginx
    ports:
      - "80:80"
    volumes:
      - ./dist:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

volumes:
  mysql-data:
```

```bash
# 一键启动所有服务
docker compose up -d

# 查看运行状态
docker compose ps

# 查看日志
docker compose logs -f

# 一键停止
docker compose down
```

---

## 4. Docker vs 传统部署

| | 传统部署 | Docker |
|------|----------|--------|
| **环境一致性** | ❌ "我电脑上能跑啊" | ✅ 镜像统一环境 |
| **部署速度** | 安装 JDK、MySQL... | `docker run` 秒级启动 |
| **迁移** | 复杂，每台机器重新配 | 镜像一次构建，到处运行 |
| **资源利用** | 一台服务器装一个应用 | 一台服务器可运行多个容器 |
| **学习成本** | 低 | 中 |

---

## 5. 总结

| 知识点 | 核心要点 |
|--------|----------|
| Linux 常用命令 | `cd` `ls` `cat` `tail` `ps` `netstat` `tar` `vi` `chmod` |
| 后台运行 | `nohup java -jar xxx.jar > app.log &` |
| Nginx 反向代理 | 前端 `/` → `dist/`，后端 `/api/` → SpringBoot |
| Docker 镜像 | `docker pull/build` |
| Docker 容器 | `docker run/stop/start/rm/logs/exec` |
| Dockerfile | `FROM` `WORKDIR` `COPY` `EXPOSE` `ENTRYPOINT` |
| Docker Compose | 多容器一键编排，`up -d` 启动，`down` 停止 |
