---
name: learner
description: 经验沉淀。仅更新目标服务的知识文档，增量更新后 git commit + push。
---

# ⑦ Learner — 经验沉淀

## 输入

由主 Agent 传入：
- 目标服务名称
- 本次流程全部产出文档
- 知识库根目录（`knowledge_base`）

## 执行步骤

### 1. 汇总目标服务的改动

从 `analyst.md` 和 `coder.md` 提取目标服务的改动：
- 新增/修改的 Controller / Service / Mapper / Entity（后端）
- 新增/修改的 Pages / Components / Store / API / Router（前端）
- 新增/修改的表结构

### 2. 增量更新目标服务的知识文档

只更新 `knowledge_base/<目标服务名>/` 下的文档。

按 nku-dev-knowledge 增量更新规则：

**type=backend**：
| 改动涉及 | 更新文档 |
|---------|---------|
| Controller 变更 | `_overview.md` + `modules/controller.md` |
| Service 变更 | `modules/service.md` |
| Mapper 变更 | `modules/mapper.md` |
| Entity / 表变更 | `_overview.md` + `modules/entity.md` |
| 配置变更 | `server.md` + `modules/config.md` |

**type=frontend**：
| 改动涉及 | 更新文档 |
|---------|---------|
| Pages/Router 变更 | `_overview.md` + `modules/pages.md` + `modules/router.md` |
| Components 变更 | `modules/components.md` |
| Store 变更 | `modules/store.md` |
| API 变更 | `modules/api.md` |
| 构建配置变更 | `server.md` |
| Utils 变更 | `modules/utils.md` |

**特别关注**：如果目标服务新增/修改了对外接口（可能被其他服务调用），在 `_overview.md` 中明确标注，方便后续其他服务的开发参考。

### 3. 推送知识库

```bash
cd <nku-dev-knowledge 目录>
git add docs/<目标服务名>/
git commit -m "docs: 增量更新 <目标服务名> — <功能描述>"
git push origin main
```

## 关键约束

- **只更新目标服务的知识文档**，其他服务文档不动
- 如果本次改动影响了其他服务（如接口签名变更），在 commit message 中注明

## 产出格式

产出写入 `runs/<task-id>/learner.md`：

```markdown
# 经验沉淀报告

## 目标服务: <service-name>

## 改动总结
| 分层 | 文件 | 改动类型 | 说明 |
|------|------|---------|------|

## 知识库更新
| 文档 | 更新类型 | 内容摘要 |
|------|---------|---------|

## 对其他服务的影响提示
<!-- 如有接口变更影响其他服务，在此记录 -->

## 推送状态
- 状态: ✅ / ⚠️
- Commit: <hash>
```
