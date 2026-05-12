# furniture-psd-generator

家具投标插图生成工作流 Hermes Skill

用 gpt-image-1.5 分层生图（原生透明背景）+ PIL 布局合成 + psd-tools 打包为 Photoshop 可直接编辑的 PSD 文件。

## 安装

```bash
hermes skills install https://github.com/Mark7766/furniture-psd-generator
```

## 功能

- 调用 302.ai gpt-image-1.5 生成透明背景 PNG（无需抠图）
- PIL 代码控制各图层的缩放比例和位置
- psd-tools 打包为多图层 PSD，PS 直接打开可编辑

## 依赖

```bash
pip install psd-tools Pillow requests
```

需要 302.ai API Key：https://302.ai

---

## 🤖 Multi-Platform AI Tool Installation Guide
## 🤖 多平台 AI 工具接入指南

### 核心原理 / Core Principle

> **将 `SKILL.md` 的内容复制为对应 AI 工具的 system prompt 或 rules 文件，即可在任意支持自定义指令的 AI 编程工具中使用本工作流。**
>
> **Copy the content of `SKILL.md` as the system prompt or rules file for your AI tool to use this workflow in any AI coding assistant that supports custom instructions.**

本 skill 的所有工作流逻辑、提示词和代码模板都包含在 `SKILL.md` 中。只要 AI 工具能读取该文件（或其内容），就能完整复现家具 PSD 生成能力。

All workflow logic, prompts, and code templates are contained in `SKILL.md`. Any AI tool that can read this file (or its contents) can fully reproduce the furniture PSD generation capability.

---

### 1. 🌊 Windsurf (Codeium)

**配置文件**：`.windsurfrules`（项目根目录）

```bash
# 在你的项目根目录创建规则文件
cp /path/to/SKILL.md .windsurfrules
# 或者将 SKILL.md 内容追加到已有的 .windsurfrules
cat SKILL.md >> .windsurfrules
```

Windsurf 会自动读取项目根目录的 `.windsurfrules` 文件，将其作为 Cascade AI 的上下文规则。

> **.windsurfrules** is loaded automatically by Windsurf's Cascade AI as project-level context rules.

---

### 2. 🖱️ Cursor

**配置方式**：`.cursorrules`（项目根目录）或 Cursor Rules UI

**方法一：文件方式**
```bash
cp /path/to/SKILL.md .cursorrules
```

**方法二：Cursor Rules UI**
1. 打开 Cursor → Settings → Rules for AI
2. 将 `SKILL.md` 全文粘贴到 "User Rules" 或 "Project Rules" 中
3. 保存后立即生效

> Cursor reads `.cursorrules` from the project root automatically. You can also use **Cursor Settings → Rules for AI** to paste the skill content as persistent instructions.

---

### 3. 🤖 Claude Code (Anthropic CLI)

**配置方式一**：`CLAUDE.md`（推荐，项目级）

```bash
# 将 SKILL.md 内容写入项目的 CLAUDE.md
cp SKILL.md CLAUDE.md
# 或追加到已有 CLAUDE.md
cat SKILL.md >> CLAUDE.md
```

**配置方式二**：`--add-dir` 参数

```bash
# 启动 Claude Code 时指定包含 SKILL.md 的目录
claude --add-dir /path/to/furniture-psd-generator
```

**配置方式三**：全局用户级配置

```bash
# 写入用户级 CLAUDE.md（对所有项目生效）
cat SKILL.md >> ~/.claude/CLAUDE.md
```

> Claude Code automatically reads `CLAUDE.md` in the project root and parent directories. Use `--add-dir` to include any directory containing `SKILL.md` as additional context.

---

### 4. 🔷 VS Code + GitHub Copilot

**配置文件**：`.github/copilot-instructions.md`（项目级）

```bash
mkdir -p .github
cp /path/to/SKILL.md .github/copilot-instructions.md
```

**或使用 Workspace Instructions**：
1. 打开 VS Code → Settings (JSON)
2. 添加配置：
```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "SKILL.md"
    }
  ]
}
```

> GitHub Copilot in VS Code reads `.github/copilot-instructions.md` as repository-level custom instructions automatically applied to all Copilot Chat interactions.

---

### 5. 🟡 Trae (字节跳动 / ByteDance)

**官网**：[trae.ai](https://trae.ai)

**配置方式：Rules / Prompt 配置**

1. 打开 Trae → 左下角设置图标 → **Rules**（规则）
2. 在 "Project Rules" 中新建规则，将 `SKILL.md` 全文粘贴
3. 或在对话前通过 **Prompt** 面板粘贴内容作为 system prompt

**Project Rules 文件方式**（如支持）：
```bash
# 在项目根目录创建 .trae/rules.md 或查看 Trae 文档确认路径
mkdir -p .trae
cp SKILL.md .trae/rules.md
```

> Trae supports project-level Rules configuration. Paste the `SKILL.md` content in the Rules panel, or check Trae's latest documentation for the supported rules file path in your project.

---

### 6. 🟢 OpenCode

**配置文件**：`AGENTS.md`（项目根目录）

```bash
cp SKILL.md AGENTS.md
# 或追加到已有 AGENTS.md
cat SKILL.md >> AGENTS.md
```

OpenCode 在启动时自动读取项目根目录的 `AGENTS.md` 文件作为 agent 上下文。

> OpenCode reads `AGENTS.md` from the project root as the agent context file.

---

### 7. 🔴 腾讯 AI 编程工具（腾讯云 AI 代码助手 / CodeBuddy）

**配置方式：自定义 Prompt / 规则**

**方法一：对话注入**
在开始对话时，将 `SKILL.md` 全文作为第一条消息发送，告知 AI 工具本次工作流规范。

**方法二：项目规则文件**（若 CodeBuddy 支持）
```bash
# 查看 CodeBuddy 文档确认规则文件路径，常见路径：
cp SKILL.md .codecbuddy/instructions.md
# 或
cp SKILL.md .ai/instructions.md
```

**方法三：IDE 插件设置**
1. 打开 VS Code / JetBrains IDE 中的腾讯云 AI 代码助手插件
2. 进入插件设置 → 自定义 Prompt / System Prompt
3. 粘贴 `SKILL.md` 内容

> For Tencent CodeBuddy, paste the `SKILL.md` content as a custom system prompt in the plugin settings, or send it as the initial context message before starting your workflow conversation.

---

### 8. 🟠 阿里云 AI 编程工具（通义灵码 / Tongyi Lingma）

**配置方式：自定义指令 / 项目规则**

**方法一：IDE 插件自定义指令**
1. 打开 VS Code / JetBrains 中的通义灵码插件
2. 进入插件设置 → **自定义指令**（Custom Instructions）
3. 将 `SKILL.md` 全文粘贴到自定义指令框中

**方法二：项目配置文件**（若通义灵码支持项目规则）
```bash
# 常见路径（以实际文档为准）
cp SKILL.md .lingma/instructions.md
# 或
cp SKILL.md .tongyi/rules.md
```

**方法三：对话上下文注入**
```
# 在通义灵码对话开始时发送：
请按照以下工作流规范执行任务：
[粘贴 SKILL.md 全文]
```

> For Tongyi Lingma (通义灵码), use the Custom Instructions setting in the IDE plugin, or paste the `SKILL.md` content at the beginning of your conversation as context.

---

### 9. 🌐 Universal Method / 通用方法

**适用于所有支持 system prompt 或自定义指令的 AI 工具 / For any AI tool that supports system prompts or custom instructions:**

**步骤 / Steps:**

1. 打开 `SKILL.md` 文件，复制全部内容
2. 粘贴到目标工具的 system prompt、自定义指令或规则配置中
3. 开始对话，直接描述你的家具投标需求

---

1. Open `SKILL.md` and copy all content
2. Paste into the target tool's system prompt, custom instructions, or rules configuration
3. Start chatting — describe your furniture bidding illustration needs directly

**常见配置路径速查表 / Quick Reference for Rules File Paths:**

| AI 工具 | 配置文件路径 |
|---------|------------|
| Windsurf | `.windsurfrules` |
| Cursor | `.cursorrules` |
| Claude Code | `CLAUDE.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| OpenCode | `AGENTS.md` |
| Trae | Rules 面板（UI配置）|
| 通义灵码 | 插件自定义指令（UI配置）|
| CodeBuddy | 插件 System Prompt（UI配置）|
| 任意支持 AGENTS.md 的框架 | `AGENTS.md` |
| 任意支持 CLAUDE.md 的框架 | `CLAUDE.md` |

**示例对话 / Example Prompt After Setup:**

```
帮我生成一套现代简约风格客厅家具投标插图，
包含：沙发、茶几、装饰植物三个图层，
最终输出 1024x1024 的多图层 PSD 文件。
```

```
Generate a modern minimalist living room furniture bidding illustration,
with layers for: sofa, coffee table, and decorative plant.
Output as a 1024x1024 multi-layer PSD file.
```
