---
name: coder
description: 代码实现。按分层惯例逐层实现，同时支持后端（Entity → Mapper → Service → Controller）和前端（API → Store → Components → Pages → Router）。
---

# ② Coder — 代码实现

## 输入

由主 Agent 传入：
- `analyst.md` 改动清单（区分后端/前端）
- 后端仓库本地路径（`config.json` → `repos.backend.local_path`）
- 前端仓库本地路径（`config.json` → `repos.frontend.local_path`，如涉及前端）
- 开发者信息（`config.json` → `developer`）

## 执行步骤

### 1. 创建 feature 分支

在后端和/或前端仓库分别创建 `feature/<功能描述>` 分支：

```bash
# 后端
cd <backend_repo>
git checkout main && git pull origin main
git checkout -b feature/<功能描述>

# 前端（如涉及）
cd <frontend_repo>
git checkout main && git pull origin main
git checkout -b feature/<功能描述>
```

### 2. 后端实现（如涉及）

按 Spring Boot 分层顺序：

1. **数据库 DDL** → `src/main/resources/db/migration/`
2. **Entity** — 实体类，使用项目已有 ORM 注解风格
3. **Mapper** — 数据访问接口
4. **Service 接口** — 业务方法签名
5. **ServiceImpl** — 业务逻辑实现
6. **Controller** — REST 接口
7. **DTO/VO** — 数据传输对象（如需要）

### 3. 前端实现（如涉及）

按前端分层顺序：

1. **API 层** — `src/api/` 新增接口调用函数
2. **Store 层** — `src/stores/` 新增/更新状态管理（如需要）
3. **Components** — `src/components/` 新增/修改组件
4. **Pages** — `src/views/` 新增/修改页面
5. **Router** — `src/router/` 注册新路由

### 4. 遵守已有代码风格

- 后端：命名规范、注解风格、返回值包装、异常处理与项目保持一致
- 前端：组件写法（Composition API / Options API）、状态管理写法、CSS 方案与项目保持一致

### 5. 按逻辑单元 commit

每完成一个完整的分层单元就 commit，区分后端和前端：

```
feat: <功能描述> - 后端 Entity + Mapper
feat: <功能描述> - 后端 Service + Controller
feat: <功能描述> - 前端 API + Store
feat: <功能描述> - 前端 Pages + Router
```

## 产出格式

产出写入 `runs/<task-id>/coder.md`：

```markdown
# 代码实现记录

## 后端改动（如有）
- 分支: feature/<功能>
- 基于: main

### 实现文件清单
| 文件 | 改动类型 | 说明 |
|------|---------|------|

### Commit 记录
| Hash | 消息 |
|------|------|

## 前端改动（如有）
- 分支: feature/<功能>
- 基于: main

### 实现文件清单
| 文件 | 改动类型 | 说明 |
|------|---------|------|

### Commit 记录
| Hash | 消息 |
|------|------|
```
