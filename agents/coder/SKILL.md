---
name: coder
description: 代码实现。仅对目标服务进行开发，参考全部知识库理解服务间调用约定。
---

# ② Coder — 代码实现

## 输入

由主 Agent 传入：
- `analyst.md` 改动清单
- **全部服务的知识文档**（了解其他服务的接口约定、DTO 结构）
- 目标服务的名称、local_path、开发者信息

## 执行步骤

### 1. 创建 feature 分支（仅目标服务）

```bash
cd <目标服务 local_path>
git checkout main && git pull origin main
git checkout -b feature/<功能描述>
```

### 2. 按分层顺序实现（仅目标服务）

**后端服务（type=backend）**：
1. 数据库 DDL → Entity → Mapper → Service → ServiceImpl → Controller → DTO/VO

**前端项目（type=frontend）**：
1. API 层 → Store → Components → Pages → Router

### 3. 参考其他服务知识库

- 调用其他服务时，查阅该服务的知识文档确认接口路径和参数格式
- Feign 客户端 / API 函数的命名和 DTO 结构与目标服务保持一致
- 不修改其他服务的代码

### 4. 遵守代码风格

与目标服务已有代码风格保持一致。

### 5. 按逻辑单元 commit

每完成一个分层单元就 commit，commit 格式：
```
feat: <功能描述> - <分层>
```

## 产出格式

产出写入 `runs/<task-id>/coder.md`：

```markdown
# 代码实现记录

## 目标服务
- 名称: <service-name>
- 分支: feature/<功能>

## 实现文件清单
| 文件 | 改动类型 | 说明 |
|------|---------|------|

## Commit 记录
| Hash | 消息 |
|------|------|

## 跨服务调用说明（如有）
<!-- 本次改动涉及调用哪些其他服务，使用了什么接口 -->
```
