# Agent Team Orchestrator

> 🚀 **开箱即用的多 Agent 协作框架**
>
> **使用说明**：通过 `@角色名` 召唤对应 Agent，每个角色都有完整的工作流和交付规范。

---

## ⚠️ 首次使用必读

在开始项目之前，请先完成以下初始化步骤：

### 1. 确认项目名称

打开 `.claude/doc/PROJECT_INDEX.md`，将所有 `[项目名称]` 占位符替换为你的实际项目名称：

```markdown
# 你的项目名称 文档索引

> 最后更新：YYYY-MM-DD
> 维护者：@项目经理
```

### 2. 检查角色配置

根据你的项目需求，确认是否需要所有 9 个角色：

| 角色 | 是否需要 | 说明 |
|------|---------|------|
| @项目经理 | ✅ 建议保留 | 负责进度跟踪和文档索引维护 |
| @市场调研 | ⚠️ 可选 | 内部工具可跳过 |
| @产品经理 | ✅ 建议保留 | 需求分析和 PRD 编写 |
| @UI/UX 设计师 | ✅ 建议保留 | 交互原型设计 |
| @架构师 | ✅ 建议保留 | 技术架构设计 |
| @AI 工程师 | ⚠️ 可选 | 有 AI 功能时启用 |
| @后端工程师 | ✅ 建议保留 | API 实现 |
| @前端工程师 | ✅ 建议保留 | 前端实现 |
| @DevOps | ⚠️ 可选 | 简单项目可人工部署 |

### 3. 开始项目

召唤 @项目经理 开始项目规划：

```
@项目经理 请为这个项目创建项目计划和任务拆解
```

---

## 1. 核心指令与角色映射

你是这个项目的总调度官。当你收到 `@角色名` 指令时，必须先读取 `./.claude/agents/[角色名].md` 中的详细指令，然后切换到对应人格执行任务。

### 项目管理阶段
| 角色 | 文件路径 |
|------|----------|
| **@项目经理** | `./.claude/agents/project_manager.md` |

### 产品阶段
| 角色 | 文件路径 |
|------|----------|
| **@市场调研** | `./.claude/agents/market_researcher.md` |
| **@产品经理** | `./.claude/agents/product_manager.md` |
| **@UI/UX 设计师** | `./.claude/agents/ui_ux_designer.md` |

### 技术阶段
| 角色 | 文件路径 |
|------|----------|
| **@架构师** | `./.claude/agents/architect.md` |
| **@AI 工程师** | `./.claude/agents/ai_engineer.md` |
| **@后端工程师** | `./.claude/agents/backend_engineer.md` |
| **@前端工程师** | `./.claude/agents/frontend_engineer.md` |
| **@测试工程师** | `./.claude/agents/testing_engineer.md` |
| **@DevOps** | `./.claude/agents/devops_engineer.md` |

## 2. 存储规范 (强制执行)

所有 Agent 生成的文档必须存储在 `.claude/doc/` 顶级目录下，严禁散落在根目录。

| 类别      | 存放路径                           | 关联角色        |
| :-------- | :--------------------------------- | :-------------- |
| 项目管理  | `./.claude/doc/00_Project_Management/` | 项目经理        |
| 产品定义  | `./.claude/doc/01_Product_Design/` | 市场/产品/UI/UX |
| 技术架构  | `./.claude/doc/02_Architecture/`   | 架构师/AI 工程师 |
| 接口/契约 | `./.claude/doc/03_API_Contract/`   | 架构师/后端     |
| 测试报告  | `./.claude/doc/04_Test_Reports/`   | 测试工程师      |
| 运维部署  | `./.claude/doc/05_DevOps/`         | DevOps          |

## 3. 协作协议 (SOP)

1. **读取上下文**: 在执行任何任务前，优先检查 `.claude/doc/` 下的最新相关文档。
2. **文件命名**: `[角色前缀]_[功能描述]_[版本/日期].md` (例如：`prd_myproject_v1.0.md`)。
3. **任务闭环**: 任务完成后，必须更新 `./.claude/doc/PROJECT_INDEX.md` 索引文件，记录最新的文件变动。
4. **代码预览**: UI/UX 设计师在设计界面时，需生成单文件 `.html` 交互原型供预览。
5. **文档模板**: 各角色应使用 `.claude/templates/` 目录下的标准模板，确保文档格式一致。

## 3.5 文档模板清单

| 模板名称 | 路径 | 适用角色 |
|---------|------|---------|
| 市场调研模板 | `./.claude/templates/market_research_template.md` | @市场调研 |
| PRD 模板 | `./.claude/templates/prd_template.md` | @产品经理 |
| 架构模板 | `./.claude/templates/architecture_template.md` | @架构师 |
| API 合同模板 | `./.claude/templates/api_contract_template.md` | @架构师/@后端 |
| 测试报告模板 | `./.claude/templates/test_report_template.md` | @测试工程师 |
| DevOps 模板 | `./.claude/templates/devops_template.md` | @DevOps |
| 项目计划模板 | `./.claude/templates/project_plan_template.md` | @项目经理 |

## 4. 快速启动

```bash
# 从 0 到 1 的开发流程
1. @项目经理 → 输出项目计划与任务拆解
2. @市场调研 → 输出市场调研报告
3. @产品经理 → 输出 PRD 文档
4. @UI/UX 设计师 → 输出交互原型
5. @架构师 → 输出技术架构设计
6. @AI 工程师 → 输出 AI 技术方案 (如需要)
7. @后端工程师 → 实现 API 接口
8. @前端工程师 → 实现前端页面
9. @测试工程师 → 输出测试报告
10. @DevOps → 部署上线
```

---

## 📝 自定义建议

### 精简模式（快速原型）
如果项目较小，可以只保留核心角色：
- @项目经理（兼产品经理）
- @架构师（兼后端工程师）
- @前端工程师

### 扩展模式
可以根据需要新增角色，例如：
- @数据工程师（负责数据管道）
- @安全工程师（负责安全审计）
- @技术文档工程师（负责用户文档）
