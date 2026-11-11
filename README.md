# 绿软 Agent 技能（lvruan-skill）

[English](#english)

让你的 AI Agent（Claude Code、Codex、OpenClaw、Cursor 等）直接用上 [绿软 lvruan.com](https://lvruan.com) 的软件库：
按名字或需求找软件、拿下载地址、看每天新收录的 AI 软件、提交你自己的软件并跟进审核。

**免费使用**：注册即可，免费账号每月 200 积分；看最新软件、浏览分类、AI 每日推送、提交软件都不扣积分。

## 最快的用法：一句话

不用安装任何东西，直接对你的 Agent 说：

```
学习这个 skill，获得搜索软件、查询并下载绿软软件的能力：https://lvruan.com/skill-zh.md
```

英文版是 `https://lvruan.com/skill.md`。第一次用时 Agent 会引导你登录或自动注册拿到令牌。

## 装成本地 skill

本仓库的 `skills/` 目录就是站上那两份技能的副本：

| 目录 | 语言 |
| ---- | ---- |
| `skills/lvruan-software-zh/` | 中文 |
| `skills/lvruan-software/` | 英文 |

两份的 skill 名都是 `lvruan-software`，**二选一装一份即可**。

**用 skills CLI**（支持 Claude Code、Codex、Cursor 等多种 Agent）：

```bash
npx skills add clinchcc/lvruan-skill
```

**手动复制**，以 Claude Code 为例：

```bash
git clone https://github.com/clinchcc/lvruan-skill.git
mkdir -p ~/.claude/skills
cp -r lvruan-skill/skills/lvruan-software-zh ~/.claude/skills/lvruan-software
```

Codex 放到 `~/.codex/skills/`，其他 Agent 放到它读取 skill 的目录。站上的版本随时更新，本仓库会定期同步；以
https://lvruan.com/skill-zh.md 为准。

## 能做什么

直接说需求就行，不用记命令：

- 「有没有能本地跑视频生成的 AI 软件？」→ 按需求搜索，只推荐站里真有的，附下载地址
- 「哪里下载 Everything？」→ 搜索并给出下载链接
- 「最近有什么新软件」「AI 分类里有什么」→ 最新、分类、标签列表
- 「每天给我推新的 AI 软件」→ 每天查一次，有新收录才提醒
- 「我想发布我的软件」→ 收集标题、logo、截图、下载地址、简介，提交审核
- 「我提交的软件审核过了吗」「还有多少积分」

## 令牌与积分

- 令牌保存在 `~/.lvruan/config.json`（或环境变量 `LVRUAN_API_KEY`），请求头 `Authorization: Bearer sk_...`。
- 也可以登录后在 https://lvruan.com/settings/apikeys 复制令牌，把那段消息直接贴给 Agent。
- 搜索、详情等查询每次 1 积分，失败退回；免费账号每月 200 积分，订阅用户每月再加 400。
- `/api/v1/latest`、`/api/v1/categories`、`/api/v1/digest/ai`、`/api/v1/me`、提交缺失软件、上传和发布都不扣积分。

## 主要接口

Base URL：`https://lvruan.com/api/v1`，响应格式 `{ "code": 0, "message": "ok", "data": ... }`。

| 用途 | 接口 |
| ---- | ---- |
| 注册拿令牌 | `POST /register` |
| 搜索软件和文章 | `GET /search?q=&lang=zh\|en` |
| 软件详情 / 下载地址 | `GET /apps/{id}`、`GET /apps/{id}/download` |
| 最新软件 | `GET /latest` |
| 分类 | `GET /categories`、`GET /categories/{slug}` |
| 标签 | `GET /tags`、`GET /tags/{id}` |
| AI 每日推送 | `GET /digest/ai` |
| 登记没找到的软件 | `POST /requests` |
| 上传图片 | `POST /uploads` |
| 提交软件 / 我的提交 | `POST /apps`、`GET /my/apps` |
| 积分余额 | `GET /me` |

完整的调用方式和规则见 skill 文件本身。

## 规则

- 不收录、不提交：破解、注册机、绕过授权的补丁、恶意软件、VPN 和代理工具。
- 用户没要求时，Agent 不会替用户下载或安装任何东西。
- Agent 不会把令牌回显给用户。

---

## English

Give your AI agent (Claude Code, Codex, OpenClaw, Cursor…) the [lvruan.com](https://lvruan.com/en) software
directory: find software by name or by need, get download links, follow the daily digest of new AI software, and
submit your own software and track its review.

**Free to use**: a free account gets 200 credits a month; the newest list, categories, the AI digest and publishing
cost nothing.

**One line, nothing to install** — tell your agent:

```
Learn this skill to search lvruan.com for software and get download links: https://lvruan.com/skill.md
```

**Or install it locally** (pick one of the two copies; both are named `lvruan-software`):

```bash
npx skills add clinchcc/lvruan-skill
# or
git clone https://github.com/clinchcc/lvruan-skill.git
cp -r lvruan-skill/skills/lvruan-software ~/.claude/skills/lvruan-software
```

The live version at https://lvruan.com/skill.md is the source of truth; this repository mirrors it.

## License

MIT
