1. SKILL.md

---
name: feishu-to-obsidian
version: 1.0.0
description: "飞书内容导入 Obsidian：支持飞书云文档、妙记、电子表格等飞书内容自动获取并保存到本地 Obsidian vault"
---

# 飞书内容导入 Obsidian

将飞书云空间中的内容自动获取并保存到本地 Obsidian vault，用于知识管理和个人笔记归档。

## 适用场景

当用户需要将飞书内容导入到 Obsidian 时使用本 skill：
- 飞书云文档→ Obsidian Markdown
- 飞书妙记→ Obsidian 会议纪要
- 飞书电子表格→ Obsidian Markdown 表格
- 飞书多维表格→ Obsidian Markdown

## 前置条件

**CRITICAL — 执行前 MUST 先用 Read 工具读取 [`../lark-shared/SKILL.md`](../lark-shared/SKILL.md)**
了解飞书 CLI 的认证、权限处理、全局参数和安全规则。

## 核心流程

### 1. 获取飞书内容

根据用户提供的飞书链接类型，调用对应的飞书 CLI 命令：

| 内容类型 | 识别特征 | 命令 |
|---------|---------|------|
| 云文档 | `/docx/` 或 `/wiki/` | `lark-cli docs +fetch --api-version v2 --doc <url> --doc-format markdown` |
| 妙记 | `/minutes/` | `lark-cli vc +notes --minute-tokens <token>` |
| 电子表格 | `/sheets/` | `lark-cli sheets +export --token <token> --format markdown` |
| 多维表格 | `/base/` | `lark-cli base records list ...` |

### 2. 处理内容

- 清理飞书特有标签（如 `<readonly-block>`、`<cite>` 等）
- 保留核心内容结构
- 提取元数据（标题、日期、参会人等）

### 3. 保存到 Obsidian

默认 vault 路径：`~/Documents/Obsidian Vault/`

文件命名规则：`标题-日期.md`

如：`组织AI化推进工作脑暴会-2026-05-06.md`

## 使用示例

```bash
# 用户给一个飞书文档链接
# AI 自动识别类型 → 获取内容 → 清理格式 → 保存到 Obsidian
Shortcuts
无快捷命令，本 skill 主要作为 AI Agent 的编排逻辑使用。

配置
用户首次使用时，需要确认 Obsidian vault 路径：

默认：~/Documents/Obsidian Vault/
可自定义：在 ~/.claude/skills/feishu-to-obsidian/config.json 中配置
参数
参数	说明	默认值
vault_path	Obsidian vault 路径	~/Documents/Obsidian Vault/
naming_rule	文件命名规则	title-date
create_folder	是否创建子文件夹	false
注意事项
飞书 CLI 需已登录：lark-cli auth login
确保有对应内容的访问权限
大文件可能需要较长处理时间
依赖技能
lark-doc — 云文档操作
lark-minutes — 妙记操作
lark-sheets — 电子表格操作
lark-base — 多维表格操作


---

## 2. config.json

```json
{
  "vault_path": "~/Documents/Obsidian Vault/",
  "naming_rule": "title-date",
  "create_folder": false,
  "add_frontmatter": true,
  "frontmatter_template": "---\nsource: feishu\nurl: {{url}}\ntype: {{type}}\ndate: {{date}}\n---\n\n"
}
3. references/workflow.md

# 飞书内容导入 Obsidian 工作流

## 完整流程

用户输入飞书链接
↓
识别链接类型
↓
调用对应飞书 CLI 命令
↓
获取内容（Markdown/XML）
↓
清理格式、提取元数据
↓
生成文件名（标题-日期.md）
↓
保存到 Obsidian vault
↓
确认保存成功



## 链接类型识别

| URL 模式 | 类型 | 处理命令 |
|----------|------|----------|
| `https://*.feishu.cn/docx/*` | 云文档 | `docs +fetch` |
| `https://*.feishu.cn/wiki/*` | 知识库文档 | `docs +fetch` |
| `https://*.feishu.cn/minutes/*` | 妙记 | `vc +notes` |
| `https://*.feishu.cn/sheets/*` | 电子表格 | `sheets +export` |
| `https://*.feishu.cn/base/*` | 多维表格 | `base records list` |

## 内容清理规则

### 移除的标签

- `<readonly-block type="isv">`
- `<readonly-block type="meeting_notes_qa">`
- 空的 `<cite>` 标签（保留有内容的）

### 保留的标签

- `<blockquote>` → Markdown 引用
- `<grid>` → Markdown 表格或保留
- `<whiteboard>` → 转为链接说明

### 元数据提取

从内容中提取：
- **标题**：`<title>` 标签或文档标题
- **日期**：会议时间、创建时间或 URL 推断
- **参会人**：`<cite type="user">` 标签
- **相关链接**：文档内嵌入的其他飞书链接

## 文件命名

```javascript
// 规则：标题-日期.md
// 示例：组织AI化推进工作脑暴会-2026-05-06.md

function generateFilename(title, date) {
    const cleanTitle = title.replace(/[\\/:*?"<>|]/g, '');
    const dateStr = formatDate(date); // YYYY-MM-DD
    return `${cleanTitle}-${dateStr}.md`;
}
Obsidian Front Matter（可选）
可在文件头部添加 YAML front matter：


---
source: feishu
url: https://xxx.feishu.cn/docx/xxx
type: meeting_notes
date: 2026-05-06
tags: [会议, AI, 组织]
---
错误处理
错误类型	处理方式
权限不足	提示用户 lark-cli auth login
文档不存在	确认 URL 正确性
Vault 路径不存在	创建目录或提示用户
文件名冲突	添加时间戳或序号


---

## 4. README.md

```markdown
# 飞书内容导入 Obsidian Skill

> 将飞书云文档、妙记、电子表格等内容一键导入到本地 Obsidian vault

## 功能

- 支持飞书云文档
- 支持飞书妙记
- 支持飞书电子表格
- 自动清理格式、提取元数据
- 按规则命名文件

## 安装

```bash
npx skills add wangniannian09/feishu-to-obsidian -g
前置要求
已安装 lark-cli
已登录飞书：lark-cli auth login
有对应飞书内容的访问权限
使用方法
直接给 AI 一个飞书链接：


帮我把这个飞书文档保存到 Obsidian：
https://xxx.feishu.cn/docx/Nna0dFSE9oXHHYxO245cAI8hn1g
AI 会自动：

识别链接类型
获取内容
清理格式
保存到你的 Obsidian vault
配置
编辑 config.json 自定义设置：


{
  "vault_path": "~/Documents/Obsidian Vault/",
  "naming_rule": "title-date",
  "create_folder": false,
  "add_frontmatter": true
}
文件命名规则
规则	格式	示例
title-date	标题-日期.md	组织AI化推进-2026-05-06.md
date-title	日期-标题.md	2026-05-06-组织AI化推进.md
title	标题.md	组织AI化推进.md
License
MIT
