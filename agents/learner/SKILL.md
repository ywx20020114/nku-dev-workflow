---
name: learner
description: 经验沉淀。汇总本次全栈改动，按后端/前端分别增量更新对应的知识库，git commit + push 保持知识库同步。
---

# ⑦ Learner — 经验沉淀

## 输入

由主 Agent 传入：
- 部署成功确认（deployer.md）
- 本次流程全部产出文档 + 双知识库文档路径

## 执行步骤

### 1. 汇总本次改动

从 `analyst.md` 和 `coder.md` 提取，**区分后端和前端**：

**后端改动**：
- 新增/修改的 Controller / Service / Mapper / Entity
- 新增/修改的表结构

**前端改动**：
- 新增/修改的 Pages / Components / Store / API / Router
- 新增的依赖或配置变更

### 2. 增量更新知识库

按 `nku-dev-knowledge` 的增量更新规则，**分别更新后端和前端知识库**：

**后端知识库更新**（`docs/backend/`）：

| 本次改动涉及 | 更新文档 |
|------------|---------|
| 新增/删除 Controller | `_overview.md` + `modules/controller.md` |
| 新增/删除 Service | `modules/service.md` |
| 新增/删除 Mapper | `modules/mapper.md` |
| 新增/删除 Entity / 表结构变更 | `_overview.md` + `modules/entity.md` |
| pom.xml / application.yml 变更 | `server.md` |
| Config 类变更 | `modules/config.md` |

**前端知识库更新**（`docs/frontend/`）：

| 本次改动涉及 | 更新文档 |
|------------|---------|
| 新增/删除页面 | `_overview.md` + `modules/pages.md` |
| 新增/删除路由 | `_overview.md` + `modules/router.md` |
| 新增/删除组件 | `modules/components.md` |
| Store 变更 | `modules/store.md` |
| API 函数变更 | `modules/api.md` |
| 工具函数变更 | `modules/utils.md` |
| package.json / 配置变更 | `server.md` |

### 3. 推送知识库

```bash
cd <nku-dev-knowledge 目录>
git add docs/backend/ docs/frontend/
git commit -m "docs: 增量更新 — <功能描述>"
git push origin main
```

## 失败处理

git push 失败时，先 `git pull --rebase` 尝试自动合并，仍失败则保留本地 commit，提示用户手动处理。

## 产出格式

产出写入 `runs/<task-id>/learner.md`：

```markdown
# 经验沉淀报告

## 本次改动总结

### 后端改动
| 分层 | 文件 | 改动类型 | 说明 |
|------|------|---------|------|

### 前端改动
| 分层 | 文件 | 改动类型 | 说明 |
|------|------|---------|------|

## 知识库更新记录

### 后端知识库
| 文档 | 更新类型 | 内容摘要 |
|------|---------|---------|

### 前端知识库
| 文档 | 更新类型 | 内容摘要 |
|------|---------|---------|

## 推送状态
- 状态: ✅ / ⚠️
- Commit: <hash>
```
