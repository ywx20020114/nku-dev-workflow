---
name: compiler
description: 编译验证。仅编译目标服务。后端 mvn compile + package，前端 npm run build。最多重试 3 次。
---

# ③ Compiler — 编译验证

## 输入

由主 Agent 传入：
- `coder.md`
- 目标服务的构建命令（`config.json` → `services[target].build_cmd`）
- 目标服务的 local_path

## 执行步骤

### 1. 后端编译（目标服务 type=backend）

```bash
cd <目标服务 local_path>
mvn compile
```

- 确认 `BUILD SUCCESS`，0 编译错误
- 检查 stderr 和编译警告

```bash
mvn package -DskipTests
```

- 确认 jar 包生成

### 2. 前端编译（目标服务 type=frontend）

```bash
cd <目标服务 local_path>
npm run build
```

- 确认 0 构建错误

### 3. 重试机制

最多重试 3 次。编译失败时分析错误并尝试修复。

## 关键约束

- **只编译目标服务**，不编译其他服务
- 如果编译错误来自与其他服务的接口不匹配（如 DTO 变更），记录但不停下

## 产出格式

产出写入 `runs/<task-id>/compiler.md`：

```markdown
# 编译验证报告

## 目标服务
- 名称: <service-name>
- 类型: backend / frontend

## 编译结果
- mvn compile / npm run build: ✅ / ❌
- 重试次数: <N>

## 错误列表（如有）
| 错误 | 文件 | 原因 | 修复 |
|------|------|------|------|

## 结论
```
