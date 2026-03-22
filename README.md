# my-awesome-skills

## skillhub in China
> https://skillhub.tencent.com/#categories

install skillhub client and use skillhub to manage skills for agents
```shell
# install cli only
curl -fsSL https://skillhub-1388575217.cos.ap-guangzhou.myqcloud.com/install/install.sh | bash  -s -- --no-skills

# search skill
skillhub search find-skills

# install skill
skillhub install find-skills # download skill to $HOME/skills defaultly
skillhub install find-skills --dir /path/to/skills # download skill to /path/to/skills
```

## some useful skills for my work

1. [code-review](https://github.com/google-gemini/gemini-cli/tree/main/.gemini/skills/code-reviewer)

简介：
该 Skill 旨在引导 AI 开展专业且全面的代码审查工作。它既支持审查本地代码改动（包括已暂存和未暂存的变更），也可审查远程代码合并请求（Pull Request，简称 PR）。审查的核心目标是保障代码的正确性、可维护性，并确保代码符合项目既定的规范标准。

应用场景：
- 审查远程 PR
  当你完成功能开发或问题修复并提交 PR 后，可发起 AI 审查请求。你只需提供 PR 编号或 URL（例如："Review PR #123"），AI 会自动检出（checkout）该 PR 的代码，运行项目预设的检查脚本（如 npm run preflight），同时阅读 PR 描述与评论以理解开发目标，随后对代码开展深度分析并给出反馈。
- 审查本地代码变更
  若你希望在提交代码或创建 PR 前，先对本地修改进行审查，只需发出 “审查我的代码” 等类似指令即可，无需提供 PR 相关信息；AI 会通过 git status、git diff 等命令，检查工作区中已暂存（staged）和未暂存（unstaged）的代码改动，进而对这些变更进行分析并反馈。
- 提供深度分析与结构化反馈
  无论是审查远程 PR 还是本地代码变更，AI 都会从多维度开展深度的代码质量分析，涵盖正确性、可维护性、可读性与执行效率、安全性与测试完整性等维度。最终，AI 会以结构化形式输出反馈，内容包括总体概述、具体发现（关键问题、改进建议）以及明确的结论（如批准合并或要求修改）。

2. [code fix and linting](https://github.com/facebook/react/tree/main/.claude/skills/fix)

简介：
这个 skill 的核心作用是自动化地修复代码格式并检查代码规范（linting）错误 。它通过执行两个关键命令来保证代码质量：
- yarn prettier：自动格式化已修改的文件，统一代码风格。
- yarn linc：检查代码中是否存在 linting 错误（这些是 Prettier 无法修复的，例如未使用的变量、逻辑错误等），这些错误通常会导致持续集成（CI）失败。
最终目标是确保代码在提交前符合项目规范，从而顺利通过 CI/CD 流程。

应用场景：
- 提交代码前的预防性检查
  在你完成编码，执行 git commit 之前，运行该 Skill，让 AI 自动清理代码格式，并提示任何需要手动修复的 linting 错误。
- 修复已发现的 linting 或格式问题
  当你在编码过程中或接手他人代码时，发现当前工作区内存在明显的格式混乱或 linting 错误提示（例如，IDE 的警告），可以立即运行该 Skill，快速解决当前已知的代码质量问题，从而在开发过程中保持代码的整洁和可读性。
- 解决持续集成（CI）失败问题
  当一个提交被推送到服务器后，CI 流水线报告了因 linting 或格式错误导致的失败。此时你可以在本地对应的分支上运行此 Skill，让 AI 自动修复格式问题，并列出需要手动更正的 linting 错误，帮助你快速定位并解决问题，然后提交修复。

  这种方式比较浪费 token，建议在必要时使用。
  对于 linting 类的任务，或者静态分析任务，可以通过 pre-commit 加上其他工具的方式来实现。

3. [update-docs](https://github.com/vercel/next.js/blob/canary/.agents/skills/update-docs/SKILL.md)

简介：
该 Skill 是一套用于更新 Next.js 项目文档的引导式工作流，核心作用是帮助你根据源代码的变更，来分析、更新和创建相关的文档，确保代码和文档保持同步。它特别为审查 Pull Request (PR) 时的文档完整性检查而设计，通过一系列标准化的步骤来规范文档的修改过程。

应用场景：
- 分析代码变更对文档的影响
  提交代码变更后，可以调用该 Skill 来分析哪些文档文件需要更新。
  它会通过 git diff 命令检查你的分支与 canary 分支之间的差异，并根据预定义的映射关系 (references/CODE-TO-DOCS-MAPPING.md)，找出与变更的代码文件相对应的文档文件。
- 更新现有的文档
  对于已经存在的文档，当其对应的功能或 API 发生变化时（例如组件新增了 props、函数行为变更），该 Skill 会引导你更新现有文档。
  它会提示你如何添加或修改 props 表格、更新代码示例、添加废弃通知等，并遵循项目固有的文档规范（例如，使用 <AppOnly> / <PagesOnly> 来区分不同路由的内容）。
- 为新功能创建脚手架文档
  当你在项目中添加了一个全新的功能时（例如一个新的组件、函数或配置项），该 Skill 可以帮你快速创建符合规范的新文档。
  它为不同类型的文档（如 API 参考、指南）提供了标准模板，确保新文档的结构、命名和元信息（Frontmatter）都符合项目要求。

4. [find-skills](https://github.com/vercel-labs/skills/tree/main/skills/find-skills)

简介：
该 Skill 主要作用帮助你发现并安装 Agent Skill。
它依托名为 skills 的命令行工具（CLI），让你可以从开放的 Agent Skill 生态中搜索、安装与管理各类模块化技能包；这些技能可扩展 Agent 能力，为其补充特定领域知识、标准化工作流与工具能力。

应用场景：
- 探索未知的 Skill
  当你希望 Agent 帮忙处理某个特定领域的任务，但不确定 Agent 是否具备相应能力时，可以使用此 Skill 进行探索。例如，当你询问 “你能帮我评审代码吗？” 或 “如何为我的项目生成文档？” 时，该 Skill 会被激活，主动在技能市场中搜索与 “代码评审” 或“ 文档生成” 相关的能力，并将找到的可用技能呈现给你.
- 查找特定的 Skill
  当你明确知道需要一个 Skill 来解决特定问题，但不知道具体是哪个 Skill 时，可以主动调用此 Skill 进行精确查找。例如，你可以直接说 “帮我找一个用于 React 性能优化的 skill”，该 Skill 会将 “React 性能优化” 作为关键词进行搜索，并返回最匹配的技能选项，如 “vercel-react-best-practices”。
- 提供可执行的 Skill 安装建议
  当该 Skill 找到一个或多个匹配的 skill 后，它会自动整理并输出一份标准化的推荐信息。这份信息不仅包含技能的名称和功能简介 ，还会提供 一键安装指令 (npx skills add ...) 以及指向技能详情页的官方链接 。

5. [requirement-analysis and specification](./skills/requirement-analysis-and-specification/SKILL.md)
简介：需求分析规格说明书的编写

6. [session-learning](./skills/session-learning/SKILL.md)

简介：
对已经安装的 skill 进行梳理，针对上下文中重复性的问题进行总结和提炼，更新 skill 或者添加新的 skill。

应用场景：
```
// case 1:
User: 这个问题已经解决成功了，请帮我总结一下经验，看看是否需要再加新的 skills

// case 2:
User: 这个问题已经解决成功了，你有用到什么 skills 吗？
```

7. [write-guide](https://github.com/vercel/next.js/blob/canary/.agents/skills/write-guide/SKILL.md)

8. [write-api-reference](https://github.com/vercel/next.js/blob/canary/.agents/skills/write-api-reference/SKILL.md)

9. [skill creator](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)。这个 skill 与 session-learning 类似，创建，修改和改进现有技能。

# usage
不同 agent 读取 skill 的默认目录不同，最好是只维护一个 skill，其他 agent 可以通过链接来使用该 skill。

可以按照 project，在 project 中创建 .agent/skills/ 目录，来存储该 project 的 skill，然后 .claude/.trae/.cursor 等目录中都使用软链接的方式指向该目录。