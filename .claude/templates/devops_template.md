# DevOps 部署文档 - [项目名称]

> 版本：v1.0
> 日期：YYYY-MM-DD
> 作者：@DevOps

---

## 1. 基础设施概览

### 1.1 架构图

```mermaid
graph TB
    A[用户] --> B[负载均衡器]
    B --> C[应用服务器 1]
    B --> D[应用服务器 2]
    C --> E[数据库主]
    D --> F[数据库从]
```

### 1.2 资源清单
| 资源 | 规格 | 数量 | 用途 |
|-----|------|-----|------|
| ECS | 4C8G | 2 | 应用服务器 |
| RDS | 4C16G | 1 | 数据库 |
| Redis | 2G | 1 | 缓存 |

---

## 2. CI/CD 流水线

### 2.1 流水线配置

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: npm ci && npm run build
      - name: Test
        run: npm test
      - name: Deploy
        run: ./deploy.sh
```

### 2.2 分支策略
| 分支 | 环境 | 触发条件 |
|-----|------|---------|
| feature/* | Dev | PR 创建 |
| develop | Staging | 合并入 develop |
| main | Prod | 打 Tag _release |

### 2.3 回滚机制
```bash
# 回滚命令
kubectl rollout undo deployment/[app-name]
```

---

## 3. 容器化配置

### 3.1 Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### 3.2 Kubernetes 配置

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [app-name]
spec:
  replicas: 3
  selector:
    matchLabels:
      app: [app-name]
  template:
    spec:
      containers:
      - name: [app-name]
        image: [registry]/[app-name]:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

---

## 4. 监控与告警

### 4.1 监控指标
| 指标 | 阈值 | 告警级别 |
|-----|------|---------|
| CPU 使用率 | > 80% | Warning |
| 内存使用率 | > 85% | Warning |
| 错误率 | > 1% | Critical |
| 响应时间 P95 | > 1s | Warning |

### 4.2 告警渠道
- Slack: #alerts
- 邮件：team@company.com
- 短信：[紧急联系人]

---

## 5. 环境配置

### 5.1 环境变量

```bash
# .env.production
NODE_ENV=production
DATABASE_URL=postgres://user:pass@host:5432/db
REDIS_URL=redis://host:6379
JWT_SECRET=[secret]
```

### 5.2 密钥管理
- 使用 AWS Secrets Manager / HashiCorp Vault
- 禁止硬编码敏感信息

---

## 6. 灾难恢复

### 6.1 备份策略
| 数据类型 | 频率 | 保留期 |
|---------|------|-------|
| 数据库 | 每日 | 30 天 |
| 文件存储 | 实时 | - |

### 6.2 故障转移
1. 检测故障（健康检查失败）
2. 自动切换到备用节点
3. 通知运维团队

### 6.3 恢复演练计划
- [ ] 每季度进行一次故障转移演练
- [ ] 每半年进行一次数据恢复演练

---

## 7. 运维巡检清单

### 7.1 每日巡检
- [ ] 检查监控面板
- [ ] 查看错误日志
- [ ] 确认备份完成

### 7.2 每周报告
- [ ] 输出 `ops_week报_YYYYMMDD.md`
- [ ] 分析性能趋势
- [ ] 更新容量规划

---

**Path: `.claude/doc/05_DevOps/devops_[项目简称]_[文档类型]_[日期].md`**
