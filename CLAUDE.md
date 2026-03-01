# Agent Team Orchestrator

> 使用说明：通过 `@角色名` 召唤对应 Agent，每个角色都有完整的工作流和交付规范。

## 1. 核心指令与角色映射

你是这个项目的总调度官。当你收到 `@角色名` 指令时，必须先读取 `./.claude/agents/[角色名].md` 中的详细指令，然后切换到对应人格执行任务。

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
| 产品定义  | `./.claude/doc/01_Product_Design/` | 市场/产品/UI/UX |
| 技术架构  | `./.claude/doc/02_Architecture/`   | 架构师/AI工程师 |
| 接口/契约 | `./.claude/doc/03_API_Contract/`   | 架构师/后端     |
| 测试报告  | `./.claude/doc/04_Test_Reports/`   | 测试工程师      |
| 运维部署  | `./.claude/doc/05_DevOps/`         | DevOps          |

## 3. 协作协议 (SOP)

1. **读取上下文**: 在执行任何任务前，优先检查 `.claude/doc/` 下的最新相关文档。
2. **文件命名**: `[角色前缀]_[功能描述]_[版本/日期].md` (例如: `prd_pawlog_v1.0.md`)。
3. **任务闭环**: 任务完成后，必须更新 `./.claude/doc/PROJECT_INDEX.md` 索引文件，记录最新的文件变动。
4. **代码预览**: UI/UX 设计师在设计界面时，需生成单文件 `.html` 交互原型供预览。

## 4. 快速启动

```
# 从 0 到 1 的开发流程
1. @市场调研 → 输出市场调研报告
2. @产品经理 → 输出 PRD 文档
3. @UI/UX 设计师 → 输出交互原型
4. @架构师 → 输出技术架构设计
5. @AI 工程师 → 输出 AI 技术方案 (如需要)
6. @后端工程师 → 实现 API 接口
7. @前端工程师 → 实现前端页面
8. @测试工程师 → 输出测试报告
9. @DevOps → 部署上线
```
