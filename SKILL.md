---
name: nku-dev-workflow
description: 按角色化流程驱动全栈项目（后端 + 前端）从需求到上线的每一步。7 个子 Agent 顺序执行，同时读取后端和前端两个知识库。
---

# nku-dev-workflow — 研发流程编排

面向全栈项目（Spring Boot 后端 + Vue3/React 前端）的 AI 驱动研发流程。主 Agent 负责编排调度，按顺序启动 7 个子 Agent，同时读取后端和前端两个知识库来理解全貌。

## 触发条件

| 用户说法 | 效果 |
|---------|------|
| "帮我实现 xxx 功能" / "开发 xxx" / "实现 xxx" | 主 Agent 解析需求，依次启动 7 个子 Agent 完成全流程 |
| "自动跑完" / "继续" / "不需要确认" | 在任一确认点说，后续步骤跳过确认连续执行 |

## 配置文件

执行前先读取 `config.json`，获取：
- `developer` — 开发者信息（git 提交用）
- `repos` — 后端和前端各自的仓库路径、构建/测试/运行命令
- `knowledge` — 后端和前端知识库文档路径
- `agent_mode.confirm_each_step` — 是否每步确认

## 前置依赖：双知识库加载

在启动研发流程之前，必须确认两个知识库都已就绪：

1. 读取 `knowledge.backend` → `_overview.md`，检查是否有实际内容
2. 读取 `knowledge.frontend` → `_overview.md`，检查是否有实际内容
3. 若某一侧知识库为空（模板状态），提示用户先执行 `nku-dev-knowledge` 学习该侧仓库
4. 若知识库已就绪，**同时加载后端和前端全部知识文档作为上下文**

## 需求分析与知识库联动

主 Agent 接收到需求后，先根据两个知识库判断需求涉及的范围：

| 需求涉及 | 加载知识库 | 子 Agent 调整 |
|---------|-----------|-------------|
| 仅后端 | backend 知识库 | Agent 按后端分层执行 |
| 仅前端 | frontend 知识库 | Agent 按前端分层执行 |
| 全栈（后端 + 前端） | 两个知识库 | Agent 同时处理后端和前端改动 |

## 流程编排

```
用户输入需求
    ↓
主 Agent 读取后端知识库 + 前端知识库 → 启动 ① Analyst
    → 输出改动清单（区分后端/前端） → ⏸️ 用户确认
    ↓
主 Agent 启动 ② Coder
    → feature 分支 → 按分层逐文件实现（先后端再前端或并行） → 按逻辑单元 commit
    ↓
主 Agent 启动 ③ Compiler
    → 后端 mvn compile + 前端 npm run build → 确认 0 错误
    ↓
主 Agent 启动 ④ Tester
    → 后端 mvn test + 前端 vitest/jest → 启动服务 → 接口/E2E 验证 → 数据库验证
    ↓
主 Agent 启动 ⑤ Reviewer
    → AI 自查：后端分层规范/事务/SQL/安全 + 前端组件规范/状态管理/性能/安全
    ↓
主 Agent 启动 ⑥ Deployer
    → rebase main → merge → 打 tag → 构建部署 → 健康检查 → ⏸️ 用户确认
    ↓
主 Agent 启动 ⑦ Learner
    → 总结本次改动 → 按后端/前端分别调用 nku-dev-knowledge 增量更新 → git push
```

## 执行规则

1. **每个子 Agent 执行完毕后默认暂停**，等待用户确认。用户说"自动跑完"则跳过后续确认连续执行。
2. **子 Agent 失败时暂停**，不继续后续步骤。
3. **每个子 Agent 的产出写入 `runs/<task-id>/<role>.md`**，用于断点续跑和回溯。
4. **全栈需求时**，analyst.md 和 coder.md 需要区分 `## 后端改动` 和 `## 前端改动`。

## 断点续跑

流程中断后重新启动时，检查 `runs/<task-id>/` 目录下的角色产出文件：

```
恢复逻辑：
  runs/<task-id>/
  ├── analyst.md  存在  → ① 已完成
  ├── coder.md    存在  → ② 已完成
  ├── compiler.md 存在  → ③ 已完成
  ├── tester.md   存在  → ④ 已完成
  ├── reviewer.md 存在  → ⑤ 已完成
  ├── deployer.md 存在  → ⑥ 已完成
  └── learner.md  存在  → 流程全部完成
                   不存在  → 从对应角色继续
```

## 子 Agent 调用

每个子 Agent 通过读取 `agents/<role>/SKILL.md` 获取角色定义和详细指令。主 Agent 负责：
1. 准备该角色所需的输入（之前步骤的产出文档、**后端和前端两份知识库文档**）
2. 将输入和角色定义传给子 Agent
3. 收集子 Agent 的产出，写入 `runs/<task-id>/<role>.md`
4. 根据产出决定继续或暂停
