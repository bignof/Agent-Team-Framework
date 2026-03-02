# 环境配置规范 - [项目名称]

> 版本：v1.0
> 日期：YYYY-MM-DD
> 作者：@架构师
> 维护：@架构师 / @DevOps
>
> ℹ️ **多技术栈说明**：本模板以 Node.js 生态为示例。如项目使用 Java/Go/Python 等其他技术栈，请将环境变量文件、启动脚本和依赖配置替换为对应技术栈的等价方案（如 `.properties`、`application.yml`、`.toml` 等）。Docker Compose 和中间件配置与技术栈无关，可直接使用。

---

## 1. 中间件清单

### 1.1 开发环境配置

| 中间件        | 版本  | 本地端口   | Docker 镜像                 | 账号/密码          | 说明      |
| ------------- | ----- | ---------- | --------------------------- | ------------------ | --------- |
| MySQL         | 8.0+  | 3306       | mysql:8.0                   | root/root123       | 主数据库  |
| PostgreSQL    | 15+   | 5432       | postgres:15                 | postgres/pg123     | 可选      |
| Redis         | 7.0+  | 6379       | redis:7-alpine              | -                  | 缓存/会话 |
| MongoDB       | 6.0+  | 27017      | mongo:6                     | mongo/mongo123     | 文档存储  |
| RabbitMQ      | 3.12+ | 5672/15672 | rabbitmq:3.12-management    | guest/guest        | 消息队列  |
| Kafka         | 3.5+  | 9092/9093  | confluentinc/cp-kafka:7.4.0 | -                  | 事件流    |
| Elasticsearch | 8.9+  | 9200/9300  | elasticsearch:8.9           | elastic/elastic123 | 搜索/日志 |

### 1.2 必装中间件（根据项目选择）

- [ ] MySQL 8.0+
- [ ] Redis 7.0+
- [ ] RabbitMQ 3.12+

### 1.3 可选中间件

- [ ] Elasticsearch（搜索需求时）
- [ ] Kafka（高吞吐消息场景）
- [ ] MongoDB（文档存储需求）

---

## 2. Docker Compose 配置

### 2.1 开发环境 docker-compose.yml

```yaml
# docker-compose.yml
# 注：新版 Docker Compose 已不需要 version 字段

services:
  # MySQL 数据库
  mysql:
    image: mysql:8.0
    container_name: ${PROJECT_NAME}_mysql
    ports:
      - '3306:3306'
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-root123}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-dev_db}
      MYSQL_USER: ${MYSQL_USER:-dev}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-dev123}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - dev_network
    healthcheck:
      test: ['CMD', 'mysqladmin', 'ping', '-h', 'localhost']
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 缓存
  redis:
    image: redis:7-alpine
    container_name: ${PROJECT_NAME}_redis
    ports:
      - '6379:6379'
    volumes:
      - redis_data:/data
    networks:
      - dev_network
    command: redis-server --appendonly yes
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 10s
      timeout: 5s
      retries: 5

  # RabbitMQ 消息队列
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: ${PROJECT_NAME}_rabbitmq
    ports:
      - '5672:5672' # AMQP 端口
      - '15672:15672' # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER:-guest}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASS:-guest}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - dev_network
    healthcheck:
      test: ['CMD', 'rabbitmq-diagnostics', 'check_running']
      interval: 30s
      timeout: 10s
      retries: 5

networks:
  dev_network:
    driver: bridge

volumes:
  mysql_data:
  redis_data:
  rabbitmq_data:
```

### 2.2 启动脚本

```bash
# scripts/start-dev.sh
#!/bin/bash

# 加载环境变量
if [ -f .env.development ]; then
  export $(cat .env.development | grep -v '^#' | xargs)
fi

# 启动容器
docker-compose up -d

# 等待服务就绪
echo "Waiting for MySQL..."
until mysql -h localhost -P 3306 -u root -proot123 -e "SELECT 1" &> /dev/null; do
  sleep 2
done
echo "MySQL is ready!"

echo "Waiting for Redis..."
until redis-cli ping &> /dev/null; do
  sleep 1
done
echo "Redis is ready!"

echo "All services are up and running!"
```

---

## 3. 环境变量规范

### 3.1 开发环境 (.env.development)

```bash
# ==================== 项目配置 ====================
PROJECT_NAME=[project-name]
NODE_ENV=development

# ==================== 应用配置 ====================
APP_PORT=3000
APP_HOST=localhost
API_PREFIX=/api/v1

# ==================== 数据库配置 ====================
DATABASE_HOST=localhost
DATABASE_PORT=3306
DATABASE_NAME=dev_db
DATABASE_USER=dev
DATABASE_PASSWORD=dev123
DATABASE_URL=mysql://dev:dev123@localhost:3306/dev_db?charset=utf8mb4

# ==================== Redis 配置 ====================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_URL=redis://localhost:6379/0

# ==================== RabbitMQ 配置 ====================
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_URL=amqp://guest:guest@localhost:5672/

# ==================== JWT 配置 ====================
JWT_SECRET=your-dev-jwt-secret-change-in-production
JWT_EXPIRES_IN=7d
JWT_ISSUER=your-app-name

# ==================== 日志配置 ====================
LOG_LEVEL=debug
LOG_FORMAT=dev

# ==================== CORS 配置 ====================
CORS_ORIGIN=http://localhost:5173
CORS_CREDENTIALS=true
```

### 3.2 预发布环境 (.env.staging)

```bash
# ==================== 项目配置 ====================
PROJECT_NAME=[project-name]
NODE_ENV=staging

# ==================== 应用配置 ====================
APP_PORT=3000
APP_HOST=0.0.0.0
API_PREFIX=/api/v1

# ==================== 数据库配置 ====================
# 生产环境使用环境变量注入，不要硬编码
DATABASE_HOST=${DB_HOST}
DATABASE_PORT=3306
DATABASE_NAME=${DB_NAME}
DATABASE_USER=${DB_USER}
DATABASE_PASSWORD=${DB_PASSWORD}
DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:3306/${DB_NAME}?charset=utf8mb4

# ==================== Redis 配置 ====================
REDIS_HOST=${REDIS_HOST}
REDIS_PORT=6379
REDIS_PASSWORD=${REDIS_PASSWORD}
REDIS_URL=redis://:${REDIS_PASSWORD}@${REDIS_HOST}:6379/0

# ==================== RabbitMQ 配置 ====================
RABBITMQ_HOST=${RABBITMQ_HOST}
RABBITMQ_PORT=5672
RABBITMQ_USER=${RABBITMQ_USER}
RABBITMQ_PASSWORD=${RABBITMQ_PASSWORD}
RABBITMQ_URL=amqp://${RABBITMQ_USER}:${RABBITMQ_PASSWORD}@${RABBITMQ_HOST}:5672/

# ==================== JWT 配置 ====================
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRES_IN=7d
JWT_ISSUER=your-app-name

# ==================== 日志配置 ====================
LOG_LEVEL=info
LOG_FORMAT=json

# ==================== CORS 配置 ====================
CORS_ORIGIN=https://staging.your-app.com
CORS_CREDENTIALS=true
```

### 3.3 生产环境 (.env.production)

```bash
# ==================== 项目配置 ====================
PROJECT_NAME=[project-name]
NODE_ENV=production

# ==================== 应用配置 ====================
APP_PORT=3000
APP_HOST=0.0.0.0
API_PREFIX=/api/v1

# ==================== 数据库配置 ====================
# 所有敏感配置通过密钥管理服务注入（AWS Secrets Manager / HashiCorp Vault）
DATABASE_HOST=${DB_HOST}
DATABASE_PORT=3306
DATABASE_NAME=${DB_NAME}
DATABASE_USER=${DB_USER}
DATABASE_PASSWORD=${DB_PASSWORD}
DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:3306/${DB_NAME}?charset=utf8mb4

# ==================== Redis 配置 ====================
REDIS_HOST=${REDIS_HOST}
REDIS_PORT=6379
REDIS_PASSWORD=${REDIS_PASSWORD}
REDIS_URL=redis://:${REDIS_PASSWORD}@${REDIS_HOST}:6379/0

# ==================== RabbitMQ 配置 ====================
RABBITMQ_HOST=${RABBITMQ_HOST}
RABBITMQ_PORT=5672
RABBITMQ_USER=${RABBITMQ_USER}
RABBITMQ_PASSWORD=${RABBITMQ_PASSWORD}
RABBITMQ_URL=amqp://${RABBITMQ_USER}:${RABBITMQ_PASSWORD}@${RABBITMQ_HOST}:5672/

# ==================== JWT 配置 ====================
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRES_IN=7d
JWT_ISSUER=your-app-name

# ==================== 日志配置 ====================
LOG_LEVEL=warn
LOG_FORMAT=json

# ==================== CORS 配置 ====================
CORS_ORIGIN=https://your-app.com
CORS_CREDENTIALS=true
```

### 3.4 环境变量示例文件 (.env.example)

```bash
# ==================== 项目配置 ====================
PROJECT_NAME=your-project-name
NODE_ENV=development

# ==================== 应用配置 ====================
APP_PORT=3000
APP_HOST=localhost
API_PREFIX=/api/v1

# ==================== 数据库配置 ====================
DATABASE_HOST=localhost
DATABASE_PORT=3306
DATABASE_NAME=your_db
DATABASE_USER=your_user
DATABASE_PASSWORD=your_password
DATABASE_URL=mysql://your_user:your_password@localhost:3306/your_db

# ==================== Redis 配置 ====================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_URL=redis://localhost:6379/0

# ==================== RabbitMQ 配置 ====================
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=your_user
RABBITMQ_PASSWORD=your_password
RABBITMQ_URL=amqp://your_user:your_password@localhost:5672/

# ==================== JWT 配置 ====================
JWT_SECRET=generate-a-secure-secret-with-openssl-rand-base64-32
JWT_EXPIRES_IN=7d
JWT_ISSUER=your-app-name

# ==================== 日志配置 ====================
LOG_LEVEL=debug
LOG_FORMAT=dev

# ==================== CORS 配置 ====================
CORS_ORIGIN=http://localhost:5173
CORS_CREDENTIALS=true
```

---

## 4. 配置传递说明

### 4.1 给后端工程师的配置信息

| 配置项        | 用途                 | 文件位置           |
| ------------- | -------------------- | ------------------ |
| 数据库连接    | ORM/数据库客户端配置 | `.env.development` |
| Redis 连接    | 缓存/会话存储        | `.env.development` |
| RabbitMQ 连接 | 消息队列/事件驱动    | `.env.development` |
| JWT 密钥      | 身份认证             | `.env.development` |

**后端工程师需要实现：**

1. 读取环境变量的配置加载器（config/loader.ts）
2. 环境变量验证（使用 Zod 或 Joi）
3. 默认值配置（config/default.ts）

### 4.2 给前端工程师的配置信息

| 配置项       | 用途     | 说明                        |
| ------------ | -------- | --------------------------- |
| API_BASE_URL | API 地址 | 开发：http://localhost:3000 |
| API_PREFIX   | API 前缀 | /api/v1                     |
| CORS_ORIGIN  | 允许的源 | 开发：http://localhost:5173 |

### 4.3 给 DevOps 的配置信息

| 配置项          | 用途             | 传递方式            |
| --------------- | ---------------- | ------------------- |
| 生产数据库配置  | 生产 RDS 连接    | AWS Secrets Manager |
| 生产 Redis 配置 | ElastiCache/自建 | 环境变量注入        |
| JWT_SECRET      | 生产密钥         | 密钥管理服务        |
| 日志配置        | 生产日志级别     | 环境变量            |

---

## 5. 中间件配置详情

### 5.1 MySQL 配置

```sql
-- 初始化脚本 scripts/init.sql
CREATE DATABASE IF NOT EXISTS `dev_db` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'dev'@'%' IDENTIFIED BY 'dev123';
GRANT ALL PRIVILEGES ON `dev_db`.* TO 'dev'@'%';
FLUSH PRIVILEGES;
```

### 5.2 Redis 配置

```conf
# Redis 配置覆盖（可选）
# 开发环境不需要特殊配置，使用默认即可
# 如需自定义，创建 docker-compose.yml 中的 command 覆盖
```

### 5.3 RabbitMQ 配置

```yaml
# RabbitMQ 配置（可选）
# 定义队列、交换器、绑定关系
# 建议在应用启动时通过代码自动创建
```

---

## 6. 快速启动指南

### 6.1 首次启动

```bash
# 1. 复制环境变量示例文件
cp .env.example .env.development

# 2. 启动中间件容器
docker-compose up -d

# 3. 验证服务状态
docker-compose ps

# 4. 查看服务日志
docker-compose logs -f mysql
docker-compose logs -f redis
docker-compose logs -f rabbitmq
```

### 6.2 常用命令

```bash
# 重启所有服务
docker-compose restart

# 重启单个服务
docker-compose restart mysql

# 停止所有服务
docker-compose down

# 清空数据（危险操作）
docker-compose down -v

# 查看容器状态
docker-compose ps

# 进入容器
docker-compose exec mysql bash
docker-compose exec redis redis-cli
```

---

## 7. 配置变更记录

| 版本 | 日期       | 变更内容 | 作者    |
| ---- | ---------- | -------- | ------- |
| v1.0 | YYYY-MM-DD | 初始版本 | @架构师 |

---

**Path: `.claude/doc/02_Architecture/config_[项目简称]_v[版本号].md`**
