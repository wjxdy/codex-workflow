---
name: finish-worktree-task
description: 当 worktree 任务开发完成或准备交给主会话合并时使用，用于检查改动并生成主会话合并所需的说明文件。
---

# 完成 Worktree 任务

## 作用

在 worktree 工作区内收尾当前任务，生成给主会话看的 `WORKTREE_MERGE_NOTE.md`。主会话后续根据这个文件判断如何 review、测试、合并和清理 worktree。

这个 skill 不负责执行合并，不负责删除 worktree，也不负责切回主分支。它只负责检查当前 worktree 的真实状态，并写出清楚的合并说明。

## 文件

在当前 worktree 根目录生成或更新：

- `WORKTREE_MERGE_NOTE.md`

这个文件只存在于 worktree 任务目录中，用来告诉主会话：

- 这个分支解决了什么问题。
- 改了哪些范围。
- 合并前要看哪些文件。
- 跑过哪些验证。
- 还有哪些风险、冲突或待确认点。
- 合并后要做什么。

## 硬规则

- 必须确认当前目录是 git worktree。
- 必须读取当前分支、最近提交和 `git status --short`。
- 必须读取改动概览：`git diff --stat`；如已有提交，也读取最近提交：`git log --oneline -n 5`。
- 不要编造验证结果。没有跑过的命令必须写“未运行”。
- 不要自动 merge 到主分支。
- 不要自动删除 worktree 或分支。
- 不要自动 push，除非用户明确要求并按 `AGENTS.md` 危险操作规则确认。
- 如果工作区还有未提交改动，要在合并说明中明确标出。

## 工作流程

1. 确定当前 worktree 根目录：
   - `git rev-parse --show-toplevel`
2. 读取当前分支：
   - `git branch --show-current`
3. 读取最近提交：
   - `git log --oneline -n 5`
4. 读取工作区状态：
   - `git status --short`
5. 读取改动概览：
   - `git diff --stat`
   - 如有 staged 改动，也读取 `git diff --cached --stat`
6. 根据当前会话、git 状态和实际文件改动，生成 `WORKTREE_MERGE_NOTE.md`。
7. 如任务还未提交，提醒用户是否需要先提交；不要擅自提交。
8. 完成后告诉用户：
   - 合并说明文件路径
   - 当前分支
   - 是否还有未提交改动
   - 建议主会话下一步怎么做

## WORKTREE_MERGE_NOTE.md 模板

````markdown
# Worktree 合并说明

## 基本信息
- 生成时间：
- Worktree 路径：
- 分支名：
- 目标合并分支：
- 最近提交：
- 工作区是否干净：

## 任务目标
-

## 合并前必须知道
-

## 主要改动范围
| 路径 / 模块 | 改动类型 | 说明 | 主会话 review 重点 |
|---|---|---|---|
|  |  |  |  |

## 关键文件
| 文件 | 为什么重要 | 合并时注意 |
|---|---|---|
|  |  |  |

## Git 状态
### 最近提交
```text
粘贴 git log --oneline -n 5 的结果
```

### 工作区状态
```text
粘贴 git status --short 的结果
```

### 改动概览
```text
粘贴 git diff --stat / git diff --cached --stat 的结果
```

## 验证状态
| 命令 | 结果 | 备注 |
|---|---|---|
| 未运行 | 未运行 | 如未验证，必须明确写未运行 |

## 合并建议
- 建议合并方式：
- 是否建议 squash：
- 是否需要先跑测试：
- 是否依赖其他分支：
- 是否有冲突风险：

## 不应合并 / 需要排除
-

## 合并后建议
-

## 给主会话的下一步
1.
2.
3.
````

## 完成回复格式

```markdown
已生成 worktree 合并说明：`WORKTREE_MERGE_NOTE.md`

- 当前分支：
- 工作区状态：
- 关键改动：
- 验证状态：
- 主会话建议：

请回到主会话读取该文件，再决定 review、测试、merge 和清理 worktree。
```

## 注意事项

- `WORKTREE_MERGE_NOTE.md` 应该描述真实状态，不是宣传稿。
- 如果 worktree 还没完成，也可以生成说明，但必须标明“未完成”和剩余事项。
- 主会话合并前应再次检查 git 状态和测试结果，不要只依赖说明文件。
