# Codex 工作流配置

这是一套面向 Codex 的个人工作流配置，用来让 Codex 在长期项目中更稳定地协作、记录进度、交接上下文，并在上下文清空后快速恢复理解。

这套配置的核心目标不是让 AI “自动乱跑”，而是让它在合适的时候更守规矩：

- 默认使用中文沟通和写文档。
- 按任务复杂度区分咨询、小改、中等改动和大改。
- 长期项目维护清晰的项目记忆文件。
- 上下文过满时可以生成详细交接文档。
- 重开会话后可以恢复上下文，但不会自动继续开发，除非用户明确要求。
- 危险操作必须先说明风险并获得用户确认。

## 目录结构

```text
codex-workflow/
├── AGENTS.md
├── README.md
└── skills/
    ├── create-worktree-task/
    │   └── SKILL.md
    ├── finish-worktree-task/
    │   └── SKILL.md
    ├── merge-worktree-task/
    │   └── SKILL.md
    ├── project-memory/
    │   └── SKILL.md
    ├── session-handoff/
    │   └── SKILL.md
    └── restore-context/
        └── SKILL.md
```

## 文件说明

### AGENTS.md

全局行为规则，定义 Codex 的默认工作方式。

主要包括：

- 默认中文回复和中文文档产出。
- 咨询类任务直接回答，改动类任务按开发流程处理。
- P0 / P1 / P2 任务分级。
- 验证要求。
- 项目记忆文件规则。
- Git 提交与推送规范。
- 危险操作与敏感文件规则。

### skills/project-memory

用于初始化和维护项目级记忆文件。

它维护两个文件：

- `PROJECT_PROGRESS.md`：记录项目当前阶段、已完成内容、关键决策、技术/结构备注、最近进展。
- `PROJECT_TODO.md`：只记录尚未完成的下一步、进行中事项、待确认、阻塞和后续可做事项。

适合在进入长期项目、初始化项目记忆、检查项目进度时使用。

### skills/session-handoff

用于在上下文过满、准备重开会话或暂停开发时，生成详细交接文档。

它维护：

- `SESSION_HANDOFF.md`

这个文件会记录：

- 当前任务。
- 用户原始目标。
- 已完成内容。
- 当前工作区状态。
- 关键文件和每个文件的下一步。
- 当前实现思路。
- 未完成事项。
- 已运行和未运行的验证命令。
- 阻塞、风险和待确认点。

它只负责写交接文档，不负责清空上下文，也不负责继续开发。

### skills/restore-context

用于重开会话后恢复上下文认知。

它会读取：

- `PROJECT_PROGRESS.md`
- `PROJECT_TODO.md`
- `SESSION_HANDOFF.md`
- git 当前分支、最近提交、工作区状态

它的重点是帮助 Codex 知道“上个会话在干什么、做到哪里、下一步可能是什么”。

重要边界：

- 调用 `restore-context` 不代表授权继续开发。
- 除非用户明确说“恢复后继续开发”，否则它必须在恢复摘要后停止，等待用户下一条指令。

### skills/create-worktree-task

用于主会话创建独立 worktree 和任务分支。

适合在一个项目中拆出可以独立开发的任务时使用。它会检查主仓库状态，创建可读的分支名和 worktree 目录，然后停止，提醒用户在 Codex 客户端中新开会话进入该 worktree。

它不负责在新 worktree 中开发，也不负责合并或清理。

### skills/finish-worktree-task

用于 worktree 工作窗口完成任务后，生成主会话合并需要看的说明文件：

- `WORKTREE_MERGE_NOTE.md`

这个文件记录真实改动、关键文件、验证状态、风险、合并建议和主会话下一步。

它不负责执行合并，不删除 worktree，也不删除分支。`WORKTREE_MERGE_NOTE.md` 只是临时合并说明文件，不能合入目标分支。

### skills/merge-worktree-task

用于主会话读取 `WORKTREE_MERGE_NOTE.md` 并合并某个 worktree 任务分支。

合并前它会检查主仓库和目标 worktree 状态，展示摘要、风险和将执行的命令，并等待用户确认。

重要边界：

- 合并成功后停止，等待用户运行项目或测试确认。
- 不删除 worktree。
- 不删除副分支。
- 遇到冲突时，不擅自选择任一方；必须分析方案和后果，让用户选择后再解决。

## 项目中的三个记忆文件

这套工作流在具体项目里通常会使用三个 Markdown 文件：

```text
PROJECT_PROGRESS.md
PROJECT_TODO.md
SESSION_HANDOFF.md
```

### PROJECT_PROGRESS.md

长期状态文件，记录项目已经走到哪里。

适合保存：

- 当前阶段。
- 已完成内容。
- 关键决策。
- 技术和结构备注。
- 最近一次进展。

### PROJECT_TODO.md

待办文件，只保存“还没有完成”的事情。

已完成事项不要长期保留在这里，也不要只打勾留着。任务完成后，应把结果写入 `PROJECT_PROGRESS.md`，并从 `PROJECT_TODO.md` 删除对应事项。

### SESSION_HANDOFF.md

会话交接文件，记录当前会话的详细现场。

它可以比前两个文件详细很多，因为它是给下一次恢复上下文用的临时现场记录。不要把这些大量细节塞进 `PROJECT_PROGRESS.md` 或 `PROJECT_TODO.md`。

## 推荐使用流程

### 1. 进入一个长期项目

可以对 Codex 说：

```text
调用 project-memory，初始化这个项目的记忆文件。
```

Codex 会检查项目根目录是否存在：

- `PROJECT_PROGRESS.md`
- `PROJECT_TODO.md`

如果缺失，会按 skill 模板初始化。

### 2. 日常开发

正常让 Codex 按需求工作即可。

当任务推进、阶段变化、阻塞解决时，Codex 应该提醒同步项目记忆文件。

如果发现不属于当前任务范围的问题，Codex 不应擅自扩大范围，而应先说明问题和影响，让用户决定：

- 立即处理。
- 记录到 `PROJECT_TODO.md`，本次不处理。
- 忽略。

### 3. 上下文过满，准备清空或重开

可以对 Codex 说：

```text
调用 session-handoff，记录当前会话，准备清空上下文。
```

Codex 会生成或覆盖：

```text
SESSION_HANDOFF.md
```

写完后，就可以结束当前会话或清空上下文。

### 4. 重开会话后恢复上下文

可以对 Codex 说：

```text
调用 restore-context，恢复这个项目的上下文。
```

Codex 会读取三个项目文件和 git 状态，然后输出恢复摘要。

默认情况下，它不会继续开发。恢复完成后可以再说：

```text
继续开发。
```

如果希望恢复后直接继续，也可以明确说：

```text
调用 restore-context，恢复后继续开发。
```

### 5. 拆分并行 worktree 任务

在主会话中可以说：

```text
调用 create-worktree-task，为支付模块重构创建一个 worktree。
```

创建完成后，在 Codex 客户端中新开会话，选择新 worktree 路径作为工作目录，只处理该任务。

### 6. worktree 任务完成后生成合并说明

在 worktree 工作窗口中可以说：

```text
调用 finish-worktree-task，生成合并说明。
```

它会生成：

```text
WORKTREE_MERGE_NOTE.md
```

然后回到主会话。

### 7. 主会话合并 worktree 分支

在主会话中可以说：

```text
调用 merge-worktree-task，合并 <worktree 路径或分支名>。
```

主会话会读取 `WORKTREE_MERGE_NOTE.md`，检查状态，展示合并摘要，并在用户确认后执行合并。

合并时必须排除 `WORKTREE_MERGE_NOTE.md`，不能把这个临时说明文件合入主分支。

合并后不会删除 worktree 或分支。需要你运行项目或测试确认没问题后，再单独下指令清理。

## 安装方式

把本仓库中的文件复制到 Codex 配置目录：

```bash
cp AGENTS.md ~/.codex/AGENTS.md
mkdir -p ~/.codex/skills
cp -R skills/create-worktree-task ~/.codex/skills/
cp -R skills/finish-worktree-task ~/.codex/skills/
cp -R skills/merge-worktree-task ~/.codex/skills/
cp -R skills/project-memory ~/.codex/skills/
cp -R skills/session-handoff ~/.codex/skills/
cp -R skills/restore-context ~/.codex/skills/
```

安装后，重新打开 Codex 会话即可使用。

## 安全边界

这套工作流特别强调危险操作要先问用户。

以下操作必须先说明命令、原因、影响范围和替代方案，用户明确同意后才能执行：

- force push。
- push / merge 到 `main` 或 `master`。
- `git reset --hard`。
- `git clean -fd`。
- 删除未合并分支。
- 合并或关闭 PR。
- 修改 Git 全局配置或 CI/CD 配置。
- 涉及真实秘密值的敏感文件修改。

敏感文件包括 `.env`、凭据、SSH 私钥、证书和云配置等。能使用占位符、示例值或编辑建议解决时，优先不用真实敏感内容。

## 设计原则

- `AGENTS.md` 只放全局规则，不重复维护 skill 模板。
- 模板和具体流程放在对应 skill 中。
- `PROJECT_TODO.md` 保持干净，只放未完成事项。
- `SESSION_HANDOFF.md` 可以详细，用来承接上下文。
- `restore-context` 只恢复认知，不默认继续开发。
- worktree 拆分开发由主会话创建，工作会话开发，主会话合并。
- 合并成功不等于可以清理；worktree 和副分支必须等用户验证后再单独清理。
- `WORKTREE_MERGE_NOTE.md` 只供主会话合并前阅读，合并时必须排除，不能进入目标分支。
- 所有文档、计划、总结、待办和项目记忆默认使用中文。
