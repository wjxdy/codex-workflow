---
name: create-worktree-task
description: 当用户要为一个可独立开发的任务创建 git worktree、任务分支或隔离工作区时使用，尤其是主会话准备拆分并行任务时。
---

# 创建 Worktree 任务

## 作用

从主工作区创建一个独立的 git worktree 和任务分支，让用户可以在 Codex 客户端中打开新的会话窗口，在隔离目录里开发独立任务。

这个 skill 只负责创建隔离工作区，不负责在新 worktree 里开发，不负责合并，也不负责删除 worktree。

## 适用场景

- 用户发现当前工作可以拆成两个或多个独立模块。
- 用户想让不同 Codex 窗口分别处理不同任务。
- 用户明确要求创建 worktree、任务分支、隔离开发目录。
- 主会话准备调度多个工作窗口并行开发。

不适用：

- 只是普通单任务开发。
- 当前任务还没有清晰边界。
- 用户只是想创建普通 git branch，而不是 worktree。

## 硬规则

- 创建前必须确认当前目录是 git 仓库。
- 创建前必须读取当前分支和 `git status --short`。
- 如果主工作区存在未提交改动，先向用户说明风险并询问如何处理；不要擅自 stash、commit、reset 或继续创建。
- 分支名和 worktree 目录名必须可读、可追踪，不要使用随机名。
- 默认把 worktree 放在主仓库的同级目录。
- 创建后停止，不要自动进入新 worktree 开发。
- 不要自动合并、删除 worktree 或删除分支。
- 危险操作按 `AGENTS.md` 执行：先说明风险，用户明确同意后再做。

## 命名规则

根据任务类型生成分支名：

- 新功能：`feat/<task-slug>`
- 修复：`fix/<task-slug>`
- 重构：`refactor/<task-slug>`
- 文档 / 配置 / 工作流：`docs/<task-slug>` 或 `chore/<task-slug>`

`task-slug` 使用小写英文、数字和连字符，例如：

- `feat/payment-flow`
- `fix/login-session`
- `chore/codex-workflow`

worktree 目录默认使用：

```text
../<repo-name>-<task-slug>
```

如果目标目录已存在，必须换一个目录名或询问用户，不要覆盖。

## 工作流程

1. 确定当前仓库根目录：
   - `git rev-parse --show-toplevel`
2. 读取当前分支：
   - `git branch --show-current`
3. 检查工作区：
   - `git status --short`
4. 如果工作区不干净，暂停并询问用户如何处理。
5. 根据用户描述拟定：
   - 任务名
   - 分支名
   - worktree 路径
6. 向用户说明将执行的命令，确认后执行：
   - `git worktree add <worktree-path> -b <branch-name>`
7. 在新 worktree 中检查：
   - 当前分支
   - `git status --short`
8. 回复用户：
   - worktree 路径
   - 分支名
   - 建议在 Codex 客户端中新开会话并选择该目录
   - 工作窗口完成后调用 `finish-worktree-task`

## 建议命令

```bash
git worktree add ../<repo-name>-<task-slug> -b <branch-name>
```

如果用户明确要求从特定 base 分支创建：

```bash
git worktree add ../<repo-name>-<task-slug> -b <branch-name> <base-branch>
```

## 创建后回复格式

```markdown
已创建 worktree 任务。

- 分支：
- 路径：
- 来源分支：
- 当前状态：

下一步：
1. 在 Codex 客户端中新开一个会话。
2. 选择 worktree 路径作为工作目录。
3. 在新会话里只处理这个任务。
4. 任务完成后调用 `finish-worktree-task` 生成合并说明。
```

## 注意事项

- 主会话负责创建、调度、最终合并。
- 工作会话负责在对应 worktree 中开发。
- 合并说明文件不在创建时生成；应在任务完成后由 `finish-worktree-task` 根据真实改动生成。
