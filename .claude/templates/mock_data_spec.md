# Mock 数据规范 - [项目名称]

> 版本：v2.0（简化版）
> 日期：2026-03-02
> 作者：@后端工程师

---

## 1. Mock 数据规则

**核心规则**：
- 后端工程师在 QG-02 通过后，必须提供 Mock 数据给前端
- Mock 数据格式必须与真实 API 响应一致
- 前端工程师优先使用后端提供的 Mock 数据

**Mock 数据清单**：
| 接口 | Mock 文件 | 状态 | 负责人 |
|------|----------|------|--------|
| GET /api/v1/users | `mock/users_list.json` | ☐ 待提供 / ☐ 已完成 | @后端工程师 |
| POST /api/v1/login | `mock/login.json` | ☐ 待提供 / ☐ 已完成 | @后端工程师 |

---

## 2. Mock 数据存放位置

### 2.1 目录结构

```
.claude/doc/03_API_Contract/
├── mock/
│   ├── users_list.json        # 用户列表 Mock 数据
│   ├── user_detail.json       # 用户详情 Mock 数据
│   ├── login_response.json    # 登录响应 Mock 数据
│   └── ...
├── api_[模块名]_v[版本号].md  # API 合同
└── README.md                  # Mock 数据说明
```

### 2.2 Mock 数据命名规范

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| 列表数据 | `[资源]_list.json` | `users_list.json` |
| 详情数据 | `[资源]_detail.json` | `user_detail.json` |
| 操作响应 | `[操作]_response.json` | `login_response.json` |

---

## 3. Mock 数据格式规范

### 3.1 基本格式

Mock 数据格式必须与真实 API 响应格式一致：

```json
{
  "code": 0,
  "data": {
    "list": [],
    "total": 0,
    "page": 1,
    "pageSize": 20,
    "totalPages": 0
  },
  "message": "success"
}
```

### 3.2 分页数据格式

```json
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": "1",
        "name": "示例数据 1"
      },
      {
        "id": "2",
        "name": "示例数据 2"
      }
    ],
    "total": 100,
    "page": 1,
    "pageSize": 20,
    "totalPages": 5
  },
  "message": "success"
}
```

---

## 4. Mock Server 方案

### 4.1 方案选择

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| **json-server** | 零代码、支持 RESTful | 功能简单 | 快速搭建、CRUD 场景 |
| **Mock.js** | 支持数据模板、随机数 | 需要额外配置 | 复杂数据场景 |

### 4.2 json-server 使用示例

**安装**：
```bash
npm install -g json-server
```

**启动 Mock Server**：
```bash
json-server --watch .claude/doc/03_API_Contract/mock/db.json --port 3001
```

---

## 5. Mock 数据切换真实 API

### 5.1 切换时机

| 条件 | 说明 |
|------|------|
| 后端 API 完成 80% 以上 | 核心接口可用 |
| 联调阶段开始 | 自动进入联调 |

### 5.2 切换步骤

1. **前端配置 API 基础 URL**：
   ```javascript
   // 开发环境（Mock）
   const API_BASE_URL = 'http://localhost:3001/api/v1';

   // 真实环境
   const API_BASE_URL = 'http://localhost:3000/api/v1';
   ```

2. **逐一验证接口连通性**

3. **记录问题并修复**

---

## 6. Mock 数据检查清单

### 6.1 后端工程师自检

- [ ] Mock 数据格式与 API 合同一致
- [ ] 核心接口的 Mock 数据已完成
- [ ] Mock Server 可以正常启动
- [ ] Mock 数据已提交到版本控制

### 6.2 前端工程师确认

- [ ] Mock Server 可以正常访问
- [ ] Mock 数据格式符合前端需求
- [ ] 已集成 Mock 数据到前端项目
- [ ] 了解如何切换到真实 API

---

**Path: `.claude/templates/mock_data_spec.md`**
