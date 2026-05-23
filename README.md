# nku-dev-workflow — 研发流程编排

面向多服务全栈项目的 AI 驱动研发流程 Skill。读取全部服务知识库理解全局上下文，7 角色顺序执行，仅针对**目标服务**开发。

## 核心设计

```
knowledge_base/              workflow 行为
├── user-service/     ← 读取（上下文）
├── order-service/    ← 读取（上下文）  
├── payment-service/  ← 目标服务！开发+编译+测试+部署只针对它
└── admin-frontend/   ← 读取（上下文）
```

- **读全部**：加载所有服务的知识文档，理解全貌和服务间调用关系
- **改一个**：编译、测试、部署只针对目标服务，其他服务代码不动

## 核心能力

- **多服务全栈**：`services` 数组管理 N 个后端服务 + N 个前端项目
- **全貌感知**：加载全部知识库，Analyst 自动定位目标服务，了解跨服务调用
- **7 角色流水线**：Analyst → Coder → Compiler → Tester → Reviewer → Deployer → Learner
- **目标聚焦**：只对目标服务执行编译/测试/部署，不影响其他服务
- **断点续跑**：流程中断后自动从断点恢复

## 目录结构

```
nku-dev-workflow/
├── SKILL.md              # 主 Skill：编排调度 + 目标服务定位
├── config.json           # services 数组 + knowledge_base 路径
├── README.md
├── agents/               # 7 个子 Agent
│   ├── analyst/SKILL.md  # ① 加载全部知识库，定位目标服务，输出改动清单
│   ├── coder/SKILL.md    # ② 仅对目标服务编码
│   ├── compiler/SKILL.md # ③ 仅编译目标服务
│   ├── tester/SKILL.md   # ④ 仅测试目标服务（含联调验证）
│   ├── reviewer/SKILL.md # ⑤ 评审目标服务 + 跨服务调用检查
│   ├── deployer/SKILL.md # ⑥ 仅部署目标服务
│   └── learner/SKILL.md  # ⑦ 仅更新目标服务的知识文档
└── runs/<task-id>/       # 运行记录
```

## 快速开始

### 1. 配置

编辑 `config.json`：

```json
{
  "developer": { "name": "你的名字", "email": "your@email.com" },
  "knowledge_base": "../nku-dev-knowledge/docs",
  "services": [
    {
      "name": "user-service",
      "type": "backend",
      "local_path": "/path/to/user-service",
      "build_cmd": "mvn package -DskipTests",
      "test_cmd": "mvn test",
      "health_check_url": "http://localhost:8081/actuator/health"
    },
    {
      "name": "admin-frontend",
      "type": "frontend",
      "local_path": "/path/to/admin-frontend",
      "build_cmd": "npm run build",
      "test_cmd": "npm run test",
      "health_check_url": "http://localhost:5173"
    }
  ]
}
```

### 2. 前置条件

确保已通过 `nku-dev-knowledge` 学习所有服务，`knowledge_base/` 下有对应的知识文档。

### 3. 触发研发

| 说法 | 效果 |
|------|------|
| `帮我在 user-service 中实现 xxx` | 明确目标服务，直接开始 |
| `帮我实现用户管理 CRUD` | 主 Agent 从知识库自动推断目标服务 |
| `自动跑完` | 跳过确认连续执行 |

## 目标服务定位

| 需求特征 | 定位方式 |
|---------|---------|
| 用户明确指定了 service | 直接使用 |
| 提到某个服务的 Entity/表 | 该服务即为目标 |
| 提到某接口路径 | 查找哪个 Controller 暴露了该路径 |
| 无法判断 | 列出候选服务，询问用户 |
| 涉及多个后端服务 | 提示拆分需求，一次只改一个 |

## 与 nku-dev-knowledge 的关系

```
nku-dev-knowledge (学习)            nku-dev-workflow (执行)
        │                                  │
        │  docs/user-service/ ──────────→  │  读取（理解 user-service）
        │  docs/order-service/ ─────────→  │  读取（理解上下游接口）
        │  docs/admin-frontend/ ────────→  │  读取（理解前端架构）
        │                                  │
        │  ⑦ Learner 调用增量更新 ←────────  │  只更新目标服务的文档
```
