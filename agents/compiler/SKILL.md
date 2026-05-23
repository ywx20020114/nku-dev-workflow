---
name: compiler
description: 编译验证。后端执行 mvn compile + mvn package，前端执行 npm run build。确认 0 错误，失败最多重试 3 次。
---

# ③ Compiler — 编译验证

## 输入

由主 Agent 传入：
- `coder.md` 分支名和涉及范围
- 后端构建命令（`config.json` → `repos.backend`）
- 前端构建命令（`config.json` → `repos.frontend`，如涉及前端）

## 执行步骤

### 1. 后端编译检查（如涉及后端改动）

```bash
cd <backend_repo>
mvn compile
```

- 确认 `BUILD SUCCESS`，0 编译错误
- 检查 stderr 和编译警告

```bash
mvn package -DskipTests
```

- 确认 `BUILD SUCCESS`
- 确认 jar 包生成

### 2. 前端编译检查（如涉及前端改动）

```bash
cd <frontend_repo>
npm run build   # 或 pnpm build
```

- 确认 0 构建错误
- 检查构建输出和警告

### 3. 重试机制

任一编译失败时，读取错误信息，分析原因并尝试修复，**最多重试 3 次**。

常见错误及处理：

**后端**：
- 缺少依赖类 → 检查 import 和类路径
- 方法签名不匹配 → 检查接口与实现
- 语法错误 → 修正语法

**前端**：
- TypeScript 类型错误 → 修正类型定义
- 未使用的变量/导入 → 清理
- 模块找不到 → 检查 import 路径

## 失败处理

重试 3 次仍失败时，收集全部错误日志，暂停流程。

## 产出格式

产出写入 `runs/<task-id>/compiler.md`：

```markdown
# 编译验证报告

## 后端编译
- 状态: ✅ / ❌ / 不涉及
- 重试次数: <N>
- 编译输出: ...

## 前端编译
- 状态: ✅ / ❌ / 不涉及
- 重试次数: <N>
- 编译输出: ...

## 错误列表（如有）

| 侧 | 错误 | 文件 | 原因 | 修复 |
|----|------|------|------|------|

## 结论
- 状态: ✅ 全部通过 / ❌ 存在错误
```
