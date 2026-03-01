# Agent Team Orchestrator

> 🚀 **开箱即用的多 Agent 协作框架**
>
> **使用说明**：通过 `@角色名` 召唤对应 Agent，每个角色都有完整的工作流和交付规范。

---

## 自动初始化检查（每次对话优先执行）

**当你开始任何对话时，必须先检查项目是否已初始化：**

### 1. 检查步骤

读取 `.claude/doc/PROJECT_INDEX.md` 文件，判断是否包含以下占位符：
- `[项目名称]`
- `[请修改此处]`
- `YYYY-MM-DD`
- `[请补充项目描述]`

**如果包含任一占位符** → 项目未初始化，需要执行初始化

### 2. 未初始化时的处理

如果检测到项目未初始化，直接询问用户：

```
🔔 检测到项目尚未初始化！

请告诉我：
1. 项目名称是什么？
2. 项目描述是什么？（可选）
```

### 3. 执行初始化

收到用户回复的项目名称和描述后，**直接使用 Write 工具更新文件**：

1. **更新 `PROJECT_INDEX.md`**：
   - 将 `[项目名称]` 替换为实际名称
   - 将 `[请修改此处]` 替换为实际名称
   - 将 `[请补充项目描述]` 替换为实际描述
   - 将 `YYYY-MM-DD` 替换为当前日期

2. **创建文档目录**（如不存在）：
   - `.claude/doc/00_Project_Management/`
   - `.claude/doc/01_Product_Design/`
   - `.claude/doc/01_Product_Design/prototypes/`
   - `.claude/doc/02_Architecture/`
   - `.claude/doc/03_API_Contract/`
   - `.claude/doc/04_Test_Reports/`
   - `.claude/doc/05_DevOps/`

3. **告知用户完成**：
   ```
   ✅ 项目初始化完成！

   项目名称：[名称]
   项目描述：[描述]

   现在可以开始使用了：
   - @项目经理 - 项目规划
   - @产品经理 - PRD 编写
   - @UI/UX 设计师 - 原型设计
   - @架构师 - 技术架构
   - ...（其他角色）
   ```

### 4. 已初始化情况

如果项目已初始化（无占位符），直接告知用户可以使用：

```
✅ 项目已就绪：[项目名称]

可以使用以下角色：
- @项目经理、@产品经理、@UI/UX 设计师
- @架构师、@AI 工程师、@后端工程师
- @前端工程师、@测试工程师、@DevOps

输入 @角色名 开始工作！
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

### 3.1 核心原则

**Agent Team 与传统开发的区别**：
- **无固定周期**：Agent 24/7 工作，无 Sprint/站会概念，任务驱动而非时间驱动
- **异步交接**：通过文档索引和质量卡点进行交接，无需同步会议
- **文档即契约**：所有决策和变更必须记录在文档中，而非口头沟通
- **用户即 PM**：用户负责审批关键节点（PRD、原型、架构），Agent 负责执行

### 3.2 执行规范

1. **读取上下文**: 在执行任何任务前，优先检查 `.claude/doc/` 下的最新相关文档。
2. **文件命名**: `[角色前缀]_[功能描述]_[版本/日期].md` (例如：`prd_myproject_v1.0.md`)。
3. **任务闭环**: 任务完成后，必须更新 `./.claude/doc/PROJECT_INDEX.md` 索引文件，记录最新的文件变动。
4. **代码预览**: UI/UX 设计师在设计界面时，需生成单文件 `.html` 交互原型供预览。
5. **文档模板**: 各角色应使用 `.claude/templates/` 目录下的标准模板，确保文档格式一致。
6. **配置传递**（重要）:
   - @架构师 完成架构设计后，必须创建 `config_[项目简称]_v1.0.md` 配置文档
   - @后端工程师 必须读取配置文档获取数据库/中间件配置
   - @DevOps 必须读取配置文档获取生产环境配置方案
   - 配置文档包含：中间件清单、Docker Compose 配置、环境变量模板
7. **质量卡点**（新增）:
   - 每个阶段完成后，必须通过对应的质量卡点检查才能流转到下一阶段的开始
   - 质量卡点检查清单位于 `.claude/templates/quality_gate.md`
   - 卡点状态记录在 `PROJECT_INDEX.md` 中

## 3.5 文档模板清单

| 模板名称 | 路径 | 适用角色 |
|---------|------|---------|
| 市场调研模板 | `./.claude/templates/market_research_template.md` | @市场调研 |
| PRD 模板 | `./.claude/templates/prd_template.md` | @产品经理 |
| 架构模板 | `./.claude/templates/architecture_template.md` | @架构师 |
| **配置规范模板** | `./.claude/templates/config_template.md` | @架构师 |
| API 合同模板 | `./.claude/templates/api_contract_template.md` | @架构师/@后端 |
| 测试报告模板 | `./.claude/templates/test_report_template.md` | @测试工程师 |
| DevOps 模板 | `./.claude/templates/devops_template.md` | @DevOps |
| 项目计划模板 | `./.claude/templates/project_plan_template.md` | @项目经理 |
| **质量卡点清单** | `./.claude/templates/quality_gate.md` | 所有角色 |

## 4. 快速启动

### 4.1 项目初始化（首次使用）

首次使用时，Claude Code 会自动检测项目是否已初始化。如果未初始化，会提示你输入项目名称和描述。

### 4.2 从 0 到 1 的开发流程

```bash
# 标准开发流程（包含质量卡点）
1. @项目经理 → 输出项目计划与任务拆解
   └─ 通过 QG-01 检查（项目规划 → 产品需求）
2. @市场调研 → 输出市场调研报告（可选）
3. @产品经理 → 输出 PRD 文档 + **页面线框图**
   └─ 通过 QG-02 检查（产品需求 → 架构设计）
   └─ 通过 QG-02.5 检查（产品需求 → 交互原型）
4. @UI/UX 设计师 → 输出交互原型（**基于产品经理线框图**进行高保真设计）
   └─ **原型评审**（@产品经理确认界面是否符合产品意图）
   └─ 通过 QG-03.5 检查（交互原型 → 架构设计/开发）
5. @架构师 → 输出技术架构设计 + **环境配置规范**
   └─ 通过 QG-03 检查（架构设计 → 代码实现）
6. @AI 工程师 → 输出 AI 技术方案 (如需要)
7. @后端工程师 → 实现 API 接口（读取架构师的配置文档）
   └─ 通过 QG-04 检查（代码实现 → 测试）
8. @前端工程师 → 实现前端页面
   └─ 通过 QG-04 检查（代码实现 → 测试）
9. @测试工程师 → 输出测试报告
   └─ 通过 QG-05 检查（测试 → 部署）
10. @DevOps → 部署上线（读取架构师的配置文档）
```

**质量卡点说明**：
- 每个卡点检查通过后才能进入下一阶段
- 检查清单位于 `.claude/templates/quality_gate.md`
- 每个角色完成任务后必须在对应目录生成文档

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
