# API 合同 - [模块名称]

> 版本：v1.0
> 最后更新：YYYY-MM-DD
> 作者：@架构师 / @后端工程师

---

## 1. API 概览

### 1.1 基础信息
| 项目 | 值 |
|-----|------|
| Base URL | `/api/v1/[模块]` |
| 认证方式 | Bearer Token |
| 数据格式 | JSON |

### 1.2 接口列表
| 方法 | 路径 | 描述 | 权限 |
|-----|------|------|------|
| GET | `/[资源]` | 获取列表 | 用户 |
| GET | `/[资源]/{id}` | 获取详情 | 用户 |
| POST | `/[资源]` | 创建资源 | 用户 |
| PUT | `/[资源]/{id}` | 更新资源 | 用户 |
| DELETE | `/[资源]/{id}` | 删除资源 | 管理员 |

---

## 2. 数据模型

### 2.1 [资源模型]

```typescript
interface Resource {
  id: string;
  name: string;
  createdAt: string;
  updatedAt: string;
  // ...
}
```

### 2.2 请求/响应示例

#### GET /[资源]

**请求**
```http
GET /api/v1/[资源]?page=1&limit=20
Authorization: Bearer <token>
```

**响应 (200 OK)**
```json
{
  "code": 0,
  "data": {
    "list": [],
    "total": 100,
    "page": 1,
    "pageSize": 20
  },
  "message": "success"
}
```

---

## 3. 错误码

| 错误码 | HTTP 状态码 | 说明 |
|-------|-----------|------|
| 40001 | 400 | 参数验证失败 |
| 40101 | 401 | Token 过期 |
| 40301 | 403 | 无权限 |
| 40401 | 404 | 资源不存在 |
| 50001 | 500 | 服务器内部错误 |

---

## 4. 变更记录

| 版本 | 日期 | 变更内容 | 作者 |
|-----|------|---------|------|
| v1.0 | | 初始版本 | |

---

**Path: `.claude/doc/03_API_Contract/api_[模块名]_v[版本号].md`**
