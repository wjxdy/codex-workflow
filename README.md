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

## 安装方式

把本仓库中的文件复制到 Codex 配置目录：

```bash
cp AGENTS.md ~/.codex/AGENTS.md
mkdir -p ~/.codex/skills
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
- 所有文档、计划、总结、待办和项目记忆默认使用中文。
