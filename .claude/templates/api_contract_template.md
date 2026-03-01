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
| 认证方式 | Bearer Token (JWT) |
| 数据格式 | JSON (application/json) |
| 速率限制 | 100 请求/分钟 (可配置) |

### 1.2 接口列表
| 方法 | 路径 | 描述 | 权限 | 速率限制 |
|-----|------|------|------|---------|
| GET | `/[资源]` | 获取列表 | 用户 | 100/min |
| GET | `/[资源]/{id}` | 获取详情 | 用户 | 200/min |
| POST | `/[资源]` | 创建资源 | 用户 | 50/min |
| PUT | `/[资源]/{id}` | 更新资源 | 用户 | 50/min |
| DELETE | `/[资源]/{id}` | 删除资源 | 管理员 | 20/min |

---

## 2. OpenAPI 3.0 规范

### 2.1 OpenAPI YAML 定义

```yaml
openapi: 3.0.3
info:
  title: [模块名称] API
  description: [模块] 的 RESTful API 接口规范
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: https://api.example.com/api/v1
    description: 生产环境
  - url: https://staging-api.example.com/api/v1
    description: 预发布环境
  - url: http://localhost:3000/api/v1
    description: 开发环境

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT Token，通过 Authorization header 传递

    rateLimit:
      type: rateLimit
      description: API 速率限制

  schemas:
    # 通用响应结构
    SuccessResponse:
      type: object
      properties:
        code:
          type: integer
          example: 0
          description: 成功码，0 表示成功
        data:
          type: object
          description: 响应数据
        message:
          type: string
          example: success

    ErrorResponse:
      type: object
      properties:
        code:
          type: integer
          description: 错误码
        error:
          type: string
          description: 错误类型
        message:
          type: string
          description: 错误信息
        details:
          type: array
          items:
            type: object
          description: 详细错误信息（参数验证失败时）

    # 分页结构
    PaginatedResponse:
      type: object
      properties:
        code:
          type: integer
          example: 0
        data:
          type: object
          properties:
            list:
              type: array
              items:
                $ref: '#/components/schemas/[资源]'
            total:
              type: integer
              example: 100
            page:
              type: integer
              example: 1
            pageSize:
              type: integer
              example: 20
            totalPages:
              type: integer
              example: 5
        message:
          type: string
          example: success

    # [资源] 数据模型
    [资源]:
      type: object
      required:
        - id
        - name
        - createdAt
      properties:
        id:
          type: string
          format: uuid
          example: "550e8400-e29b-41d4-a716-446655440000"
          description: 资源唯一标识符
        name:
          type: string
          example: "示例资源"
          description: 资源名称
        description:
          type: string
          example: "这是资源描述"
          description: 资源描述
        status:
          type: string
          enum: [active, inactive, pending, deleted]
          example: active
          description: 资源状态
        metadata:
          type: object
          description: 扩展元数据
        createdAt:
          type: string
          format: date-time
          example: "2024-01-01T00:00:00Z"
          description: 创建时间
        updatedAt:
          type: string
          format: date-time
          example: "2024-01-01T00:00:00Z"
          description: 更新时间
        createdBy:
          type: string
          format: uuid
          description: 创建者 ID

    # 创建请求
    Create[资源]Request:
      type: object
      required:
        - name
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
          example: "新资源"
        description:
          type: string
          maxLength: 500
          example: "资源描述"
        status:
          type: string
          enum: [active, inactive, pending]
          default: active

    # 更新请求
    Update[资源]Request:
      type: object
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        description:
          type: string
          maxLength: 500
        status:
          type: string
          enum: [active, inactive, pending, deleted]

paths:
  # 获取列表
  /[资源]:
    get:
      summary: 获取 [资源] 列表
      description: 支持分页、排序、筛选
      operationId: list[资源]
      tags:
        - [资源]
      security:
        - bearerAuth: []
      parameters:
        - name: page
          in: query
          description: 页码，从 1 开始
          schema:
            type: integer
            default: 1
            minimum: 1
        - name: limit
          in: query
          description: 每页数量
          schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
        - name: sortBy
          in: query
          description: 排序字段
          schema:
            type: string
            default: createdAt
        - name: order
          in: query
          description: 排序方向
          schema:
            type: string
            enum: [asc, desc]
            default: desc
        - name: status
          in: query
          description: 状态筛选
          schema:
            type: string
            enum: [active, inactive, pending, deleted]
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PaginatedResponse'
        '401':
          description: 未认证
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '429':
          description: 请求频率超限
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

    post:
      summary: 创建 [资源]
      description: 创建新的资源记录
      operationId: create[资源]
      tags:
        - [资源]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Create[资源]Request'
      responses:
        '201':
          description: 创建成功
          content:
            application/json:
              schema:
                allOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - type: object
                    properties:
                      data:
                        $ref: '#/components/schemas/[资源]'
        '400':
          description: 参数验证失败
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          description: 未认证

  # 获取详情
  /[资源]/{id}:
    get:
      summary: 获取 [资源] 详情
      operationId: get[资源]
      tags:
        - [资源]
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          description: 资源 ID
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                allOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - type: object
                    properties:
                      data:
                        $ref: '#/components/schemas/[资源]'
        '404':
          description: 资源不存在
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

    put:
      summary: 更新 [资源]
      operationId: update[资源]
      tags:
        - [资源]
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Update[资源]Request'
      responses:
        '200':
          description: 更新成功
          content:
            application/json:
              schema:
                allOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - type: object
                    properties:
                      data:
                        $ref: '#/components/schemas/[资源]'
        '404':
          description: 资源不存在

    delete:
      summary: 删除 [资源]
      operationId: delete[资源]
      tags:
        - [资源]
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '204':
          description: 删除成功
        '404':
          description: 资源不存在
        '403':
          description: 无权限
```

---

## 3. 认证与授权

### 3.1 认证方式
- **类型**: JWT Bearer Token
- **Header**: `Authorization: Bearer <token>`
- **Token 获取**: 通过 `/api/v1/auth/login` 接口获取
- **Token 有效期**: Access Token 7 天，Refresh Token 30 天

### 3.2 Token 刷新机制
```http
POST /api/v1/auth/refresh
Content-Type: application/json
Authorization: Bearer <access_token>

{
  "refreshToken": "<refresh_token>"
}
```

### 3.3 权限级别
| 权限级别 | 说明 | 接口示例 |
|---------|------|---------|
| PUBLIC | 无需认证 | `/api/v1/health` |
| USER | 需要登录用户 | `GET /api/v1/[资源]` |
| ADMIN | 需要管理员角色 | `DELETE /api/v1/[资源]/{id}` |

---

## 4. 速率限制

### 4.1 限制策略
| 接口类型 | 限制 | 说明 |
|---------|------|------|
| 查询接口 | 100 请求/分钟 | GET 请求 |
| 写入接口 | 50 请求/分钟 | POST/PUT 请求 |
| 删除接口 | 20 请求/分钟 | DELETE 请求 |
| 认证接口 | 10 请求/分钟 | 登录/注册 |

### 4.2 限流响应
```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
Retry-After: 60

{
  "code": 429001,
  "error": "Rate Limit Exceeded",
  "message": "请求频率超限，请在 60 秒后重试",
  "retryAfter": 60
}
```

---

## 5. 分页规范

### 5.1 分页参数
| 参数 | 类型 | 默认值 | 说明 |
|-----|------|--------|------|
| page | integer | 1 | 页码，从 1 开始 |
| limit | integer | 20 | 每页数量，最大 100 |
| sortBy | string | createdAt | 排序字段 |
| order | string | desc | 排序方向 (asc/desc) |

### 5.2 分页响应
```json
{
  "code": 0,
  "data": {
    "list": [...],
    "total": 100,
    "page": 1,
    "pageSize": 20,
    "totalPages": 5
  },
  "message": "success"
}
```

---

## 6. 错误码规范

### 6.1 错误码分类
| 错误码范围 | 分类 | 说明 |
|-----------|------|------|
| 40000-40099 | 客户端错误 | 参数验证、请求格式等 |
| 40100-40199 | 认证错误 | Token 过期、无效等 |
| 40300-40399 | 授权错误 | 无权限访问 |
| 40400-40499 | 资源错误 | 资源不存在 |
| 42900-42999 | 限流错误 | 请求频率超限 |
| 50000-50099 | 服务器错误 | 内部异常 |

### 6.2 通用错误码
| 错误码 | HTTP 状态码 | 说明 | 处理建议 |
|-------|-----------|------|---------|
| 40001 | 400 | 参数验证失败 | 检查请求参数格式 |
| 40002 | 400 | 请求体过大 | 减小请求体大小 |
| 40101 | 401 | Token 缺失 | 在 Header 中添加 Authorization |
| 40102 | 401 | Token 过期 | 使用 Refresh Token 刷新 |
| 40103 | 401 | Token 无效 | 重新登录获取 Token |
| 40301 | 403 | 无权限访问 | 检查用户角色权限 |
| 40401 | 404 | 资源不存在 | 检查资源 ID 是否正确 |
| 42901 | 429 | 请求频率超限 | 等待后重试 |
| 50001 | 500 | 服务器内部错误 | 联系技术支持 |

### 6.3 错误响应格式
```json
{
  "code": 40001,
  "error": "Validation Error",
  "message": "请求参数验证失败",
  "details": [
    {
      "field": "name",
      "message": "name 不能为空",
      "code": "REQUIRED"
    },
    {
      "field": "email",
      "message": "邮箱格式不正确",
      "code": "INVALID_FORMAT"
    }
  ]
}
```

---

## 7. 请求/响应验证规则

### 7.1 请求头要求
```http
Content-Type: application/json
Authorization: Bearer <token>
X-Request-ID: <uuid>  # 用于链路追踪
```

### 7.2 响应头
```http
Content-Type: application/json
X-Request-ID: <uuid>  # 与请求 ID 一致
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

### 7.3 数据验证规则
| 字段类型 | 验证规则 | 说明 |
|---------|---------|------|
| string | minLength, maxLength, pattern | 长度限制、正则匹配 |
| integer | minimum, maximum | 数值范围 |
| array | minItems, maxItems | 数组长度 |
| object | required, properties | 必填字段、属性定义 |
| email | format: email | 邮箱格式 |
| date-time | format: date-time | ISO 8601 格式 |

---

## 8. 变更记录

| 版本 | 日期 | 变更内容 | 作者 | 审批 |
|-----|------|---------|------|------|
| v1.0 | YYYY-MM-DD | 初始版本 | @架构师 | |
| v1.1 | | [变更说明] | | |

---

## 9. 附录

### 9.1 健康检查
```http
GET /api/v1/health

Response:
{
  "status": "healthy",
  "timestamp": "2024-01-01T00:00:00Z",
  "services": {
    "database": "connected",
    "cache": "connected"
  }
}
```

### 9.2 SDK 使用示例

**JavaScript/TypeScript**:
```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: '/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`
  }
});

// 获取列表
const list = await api.get('/[资源]', {
  params: { page: 1, limit: 20 }
});

// 创建资源
const created = await api.post('/[资源]', {
  name: '新资源',
  description: '描述'
});
```

---

**Path: `.claude/doc/03_API_Contract/api_[模块名]_v[版本号].md`**
