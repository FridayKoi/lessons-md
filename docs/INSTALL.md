# 安装指南

错题本本体是一个 markdown 文件（`LESSONS.md`），放在**每个项目的根目录**。skill 是提炼和查阅它的工具，装在**你的 AI 编程工具**里（用户级，装一次全局可用）。

## 三档模式：你配置到哪一档，就是哪一档

没有配置开关，档位由"你装了什么"决定，随时可以升级：

| 档位 | 怎么开启 | 效果 | 适合 |
|---|---|---|---|
| 🥉 被动 | 只装 skill | 你说"把这个坑记一下"，AI 才提炼入库 | 先试用 |
| 🥈 半自动（推荐） | 装 skill + AGENTS.md 接线 | AI 在复盘时机**主动提醒**你，你点头它才动手 | 日常使用。哪些错值得记的判断权留给人，防止错题本被鸡毛蒜皮灌满 |
| 🥇 全自动 | 再加各工具的 hook 配置 | 会话结束自动复盘，零人工 | 重度使用。hook 是各工具私有机制，见下方对应小节 |

## 第一步：初始化项目（所有工具通用）

把 [`LESSONS.template.md`](../LESSONS.template.md) 复制到项目根目录，改名为 `LESSONS.md`。

## 第二步：接线 —— 让 AI 每次会话都读错题本

**工具清单远不止下面这些**——`AGENTS.md` 是跨工具正式标准（Linux Foundation / Agentic AI Foundation），凡是支持它的工具都无需额外配置。清单会过时，以各工具官方文档为准。

| 工具 | 接线方式 |
|---|---|
| Codex CLI / Cursor / Copilot / Windsurf / Zed / Warp / RooCode / Aider / Devin / DeepSeek Harness (DSH) | 原生读 `AGENTS.md`，✅ 零配置 |
| Gemini CLI | 默认读 `GEMINI.md`；在 `.gemini/settings.json` 里加 `"contextFileName": "AGENTS.md"`，或把下面内容贴进 `GEMINI.md` |
| Claude Code | 读 `CLAUDE.md`，把下面内容贴进去（或把 `CLAUDE.md` 软链到 `AGENTS.md`） |
| ZCode | 读 `AGENTS.md`（项目根目录或 `~/.zcode/AGENTS.md`），贴下面内容；项目里没有就新建，或用内置 `/init` 命令生成后再追加 |
| TraeCode（Trae / TraeWork） | 支持读根目录 `AGENTS.md` 和 `CLAUDE.md`（两者各有独立开关，**默认都关闭**）：到 设置 → 规则与记忆 → 导入设置，打开"将 AGENTS.md 包含在上下文中"即可（推荐 AGENTS.md，跨工具通用；两个都开没必要），再贴下面内容。也可用其全局/项目两级"规则"功能作为替代接线 |
| WorkBuddy | 未确认读项目级 `AGENTS.md`。用 设置 → 个性化 → **自定义指令** 贴下面内容（对所有任务全局生效，上限 1500 字，下面这段放得下）。注意其"长期记忆记录"是跨项目的个人偏好，与项目级错题本不是一回事，不要混用 |
| 其他/未知工具 | 找到它读的指令文件（通常叫 AGENTS/CLAUDE/RULES 之类），贴下面内容 |

> 注：WorkBuddy、TraeWork 的"记忆/长期记忆"功能是跨会话的个人偏好记忆，与项目级的错题本互补而不替代——记忆记住"你是谁、你的习惯"，错题本记住"在这个项目里什么不能做"。

要贴的内容（一个标题 + 4 条规则，整块复制）：

```markdown
## Mistake Notebook（错题本）
- 开始任何任务前，先看项目根目录是否存在 LESSONS.md：存在则通读全部条目，并回复一行"已读错题本（N 条）"作为确认（没有这行确认就视为没读）；不存在则跳过本节
- 准备执行的操作与某条目的"触发场景"匹配时，先重读该条目再动手；🔴 禁令条目无例外，违反前必须停下说明
- 会话结束或完成一个阶段性任务后，主动询问用户是否运行 /retro 复盘
- 踩坑提炼一律写入项目根目录的 LESSONS.md（按其条目格式，含复发日期）；不要写入自动记忆等其他文件
```

## 第三步：安装 skill（可选，决定你停在几档）

skill 的格式（带 frontmatter 的 `SKILL.md`）已是事实标准，多数工具通用：

| 工具 | 复制到哪 | 全自动档 |
|---|---|---|
| Claude Code | `~/.claude/skills/` | ✅ 支持 hooks，见下 |
| ZCode | 用户级：`~/.zcode/skills/`（如 `C:\Users\<你>\.zcode\skills\`）；项目级：`<项目>/.zcode/skills/`；跨工具共享：`~/.agents/skills/` | 视版本 |
| Codex / Cursor / TraeCode / 其他 | 没有统一的 skill 目录就用兜底：把 [`skills/mistake-retro/SKILL.md`](../skills/mistake-retro/SKILL.md) 正文（去掉顶部 frontmatter）当 prompt 发给 AI，效果等同。注：TraeCode 侧边栏有"技能与命令"入口，原生兼容 SKILL.md 与否待验证 | 视工具 |

Claude Code 全自动档示例（`~/.claude/settings.json` 的 hooks 中加 SessionEnd，用提示词驱动复盘确认）：

```json
{
  "hooks": {
    "SessionEnd": [
      { "hooks": [ { "type": "command", "command": "echo 会话已结束，如本次有踩坑请运行 /retro 复盘" } ] }
    ]
  }
}
```

> 注：hook 只能发提醒，真正的提炼仍由 AI 完成——故意的，见上方"半自动"档的理由。

## 验证装好了

在一个测试项目里让 AI 做点小事，然后运行 `/retro`（或说"复盘一下"），应该看到它创建或更新了 `LESSONS.md` 并输出变更摘要。

## 建议：把 LESSONS.md 提交进 git

错题本是项目知识的一部分，和代码一起提交、随仓库共享，团队成员（和他们的 AI）都能受益。如果条目里有机器路径等隐私信息，记得让 AI 写条目时脱敏（这一点也可以写进你项目的 CLAUDE.md 里）。

## 实在不会弄？让 AI 替你装

安装这件事本身就是 AI 最擅长的活。在你的项目里打开任意 AI 编程工具，把下面整段发给它（把尖括号里的两处换成你的实际情况）：

```markdown
请帮我在当前项目里安装 lessons-md 错题本（AI 编程错题本，仓库在 <本地路径或 GitHub 地址>。步骤：

1. 读取该仓库的 docs/INSTALL.md，了解完整安装说明
2. 确认我用的 AI 工具是 <工具名，如 ZCode / Claude Code / Codex>，查出它的 skills 目录
   和它读的指令文件（AGENTS.md / CLAUDE.md 等）
3. 执行安装：把 skills/mistake-retro 和 skills/mistake-recall 复制到我的 skills 目录；
   在当前项目根目录创建 LESSONS.md（内容取仓库里的 LESSONS.template.md）；
   在指令文件里追加"Mistake Notebook"小节（内容取 INSTALL.md，若已有同名小节则跳过）
4. 装完告诉我：哪些文件被改动/创建了，以及我怎么验证装好了
```

它做完后，你只需要核对它报告的文件清单和 INSTALL.md 说的是否一致。

