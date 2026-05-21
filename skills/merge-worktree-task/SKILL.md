---
name: merge-worktree-task
description: 当用户要求在主会话中合并某个 worktree 任务分支、读取 WORKTREE_MERGE_NOTE.md 并把副分支合入当前主分支时使用。
---

# 合并 Worktree 任务

## 作用

在主会话中读取指定 worktree 的 `WORKTREE_MERGE_NOTE.md`，检查副分支和主分支状态，然后把副分支合并到当前主分支。

这个 skill 只负责合并，不负责删除 worktree，不负责删除副分支，也不负责清理工作区。合并完成后必须停下，等待用户运行项目或测试确认。

## 适用场景

- 用户明确要求合并某个 worktree 任务。
- 用户提供了 worktree 路径或分支名。
- worktree 任务已经调用 `finish-worktree-task` 生成了 `WORKTREE_MERGE_NOTE.md`。
- 主会话准备把副分支合入当前主分支。

不适用：

- 在 worktree 工作窗口里继续开发。
- 任务还没完成、没有合并说明、或者用户只是想查看状态。
- 用户要求删除 worktree 或删除分支。清理不属于本 skill。

## 硬规则

- 必须在主仓库 / 主会话中执行，不要在目标 worktree 里合并自己。
- 必须读取目标 worktree 的 `WORKTREE_MERGE_NOTE.md`。
- 必须检查目标 worktree 的当前分支、最近提交和 `git status --short`。
- 必须检查主仓库当前分支和 `git status --short`。
- 主仓库工作区不干净时，暂停并询问用户，不要合并。
- 目标 worktree 有未提交改动时，暂停并询问用户，不要合并。
- 合并前必须向用户展示摘要、风险和将执行的命令，并获得明确确认。
- 默认使用普通 merge：`git merge <branch>`。
- 不要自动 push。
- 不要删除 worktree。
- 不要删除副分支。
- 不要执行 `git worktree remove`。
- 不要执行 `git branch -d` 或 `git branch -D`。
- 合并成功后必须停止，提醒用户运行项目或测试确认。

## 工作流程

1. 确定主仓库根目录：
   - `git rev-parse --show-toplevel`
2. 确认当前主仓库状态：
   - `git branch --show-current`
   - `git status --short`
   - `git log --oneline -n 5`
3. 根据用户提供的信息定位目标 worktree：
   - 如果用户提供路径，直接检查该路径。
   - 如果用户只提供分支名，用 `git worktree list` 查找对应 worktree。
4. 读取目标 worktree 信息：
   - `WORKTREE_MERGE_NOTE.md`
   - `git -C <worktree-path> branch --show-current`
   - `git -C <worktree-path> status --short`
   - `git -C <worktree-path> log --oneline -n 5`
5. 检查是否可合并：
   - 主仓库工作区必须干净。
   - 目标 worktree 工作区必须干净。
   - 目标分支必须明确。
   - 合并说明中不能存在明确的“不应合并”阻塞项。
6. 输出合并前摘要并询问用户确认。
7. 用户确认后，在主仓库执行：
   - `git merge <branch>`
8. 如果合并成功，输出合并结果并停止。
9. 如果出现冲突，进入冲突处理流程。

## 合并前摘要格式

````markdown
准备合并 worktree 任务：

- Worktree：
- 副分支：
- 当前主分支：
- 最近提交：
- 工作区状态：
- 合并说明摘要：
- 验证状态：
- 风险 / 待确认：

将执行命令：
```bash
git merge <branch>
```

请确认是否执行合并。
````

## 冲突处理规则

如果合并出现冲突：

- 不要擅自选择任一方。
- 不要自动 `git merge --abort`。
- 先列出冲突文件并读取冲突内容。
- 分析主分支版本和 worktree 分支版本各自意图。
- 必要时可以调用相关 skill 辅助分析、调试或验证。
- 必须给用户 2-3 个解决方案，并说明每个方案的后果。
- 等用户选择后，再按用户选择解决冲突。
- 冲突解决后继续完成 merge。
- 合并完成后仍不得删除 worktree 或分支。

### 冲突方案说明格式

```markdown
合并出现冲突，等待你选择解决方案。

## 冲突文件
-

## 冲突分析
### 文件：
- 主分支意图：
- Worktree 分支意图：
- 冲突点：

## 可选方案
1. 方案 A：
   - 做法：
   - 保留：
   - 放弃：
   - 影响：
   - 需要验证：

2. 方案 B：
   - 做法：
   - 保留：
   - 放弃：
   - 影响：
   - 需要验证：

3. 方案 C：
   - 做法：
   - 保留：
   - 放弃：
   - 影响：
   - 需要验证：

请选择方案后，我再继续解决冲突。
```

## 合并成功回复格式

```markdown
合并已完成。

- 合并分支：
- 目标分支：
- 新提交：
- 冲突：
- 验证状态：

我没有删除 worktree，也没有删除副分支。

下一步请运行项目或测试确认结果。确认没问题后，你可以再明确要求清理 worktree / 分支。
```

## 注意事项

- `WORKTREE_MERGE_NOTE.md` 是合并前的重要输入，但不能替代主会话的最终检查。
- 如果合并说明和实际 git 状态不一致，暂停并让用户确认。
- 如果用户要求 squash、ff-only 或其他合并方式，先说明命令和影响，再等用户确认。
- 如果验证失败，按项目规则处理；不要因为 merge 成功就声称任务完成。
