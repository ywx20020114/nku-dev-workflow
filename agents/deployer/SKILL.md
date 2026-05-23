---
name: deployer
description: 部署上线。后端 rebase → merge → tag → jar 部署 → 健康检查，前端 rebase → merge → tag → 构建 → 部署静态资源。失败回滚。
---

# ⑥ Deployer — 部署上线

## 输入

由主 Agent 传入：
- 评审通过确认（reviewer.md）
- `config.json` → `repos`（部署命令和健康检查 URL）
- 版本号

## 执行步骤

### 1. 后端部署（如涉及后端改动）

```bash
cd <backend_repo>
git checkout main && git pull origin main
git checkout feature/<功能>
git rebase main
git checkout main
git merge feature/<功能> --no-ff
git tag -a v<version> -m "Release v<version>: <功能描述>"
mvn package -DskipTests
```

部署 jar 包到服务器，重启服务。

健康检查：
```bash
curl -s http://<host>:<port>/actuator/health
# 预期: {"status":"UP"}
```

### 2. 前端部署（如涉及前端改动）

```bash
cd <frontend_repo>
git checkout main && git pull origin main
git checkout feature/<功能>
git rebase main
git checkout main
git merge feature/<功能> --no-ff
git tag -a v<version> -m "Release v<version>: <功能描述>"
npm run build
```

部署 `dist/` 目录到服务器 / CDN / OSS。

健康检查：
```bash
curl -s http://<frontend_url>
# 预期: 200 OK，页面正常渲染
```

### 3. 推送

```bash
# 后端和前端分别
git push origin main
git push origin --tags
```

## 失败处理

部署失败或健康检查不通过时：
1. 回滚到合并前状态
2. 报告具体错误
3. 等待用户决策

## 产出格式

产出写入 `runs/<task-id>/deployer.md`：

```markdown
# 部署报告

## 版本信息
- Tag: v<version>
- 发布时间: <时间>

## 后端部署（如有）
- Merge Commit: <hash>
- 构建: ✅/❌
- 部署: ✅/❌
- 健康检查: ✅/❌

## 前端部署（如有）
- Merge Commit: <hash>
- 构建: ✅/❌
- 部署: ✅/❌
- 健康检查: ✅/❌

## 结论
- 状态: ✅ 全部成功 / ❌ 存在失败（已回滚）
```
