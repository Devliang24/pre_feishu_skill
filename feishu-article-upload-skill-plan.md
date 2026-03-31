# 飞书文章上传 Skill 实现计划

## Context

用户需要创建一个 Claude Code skill，用于将文章上传到飞书个人文档库。使用 `lark-cli` 工具来操作飞书文档 API，参考 https://code.claude.com/docs/zh-CN/skills 中的 Skills 规范。

**目标目录**: `/Users/liang/ai-work/feishu_skill/`（当前为空目录）

**核心需求**:
1. 在飞书个人文档库（my_library）下创建文件夹
2. 将文章上传到指定文件夹中

## 关键发现

### lark-cli 命令
- **创建文档**: `lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library`
  - `--folder-token <token>` 可指定父文件夹（用于在子文件夹中创建文档）
- **更新文档**: `lark-cli docs +update --doc <URL> --markdown "内容" --mode overwrite`
- **获取文档**: `lark-cli docs +fetch --doc <URL>`
- **创建文件夹**: `lark-cli api POST /open-apis/drive/v1/files/create_folder --data '{"name": "文件夹名", "folder_token": "<父文件夹token>"}'`
  - 需要有效的父文件夹 token（不能为空）
- **已安装路径**: `/opt/homebrew/bin/lark-cli`
- **认证状态**: 有效（用户: 用户369356）

### Skills 规范要点
- SKILL.md 包含 YAML frontmatter + Markdown 内容
- frontmatter 字段: `name`, `description`, `disable-model-invocation`, `allowed-tools`
- 使用 `!<command>` 语法注入动态上下文
- SKILL.md 保持在 500 行以下，详细文档放 references/

## 文件结构

```
/Users/liang/ai-work/feishu_skill/
├── SKILL.md                        # 主 skill 文件（必需）
└── references/
    └── lark_cli_reference.md       # lark-cli 详细命令参考
```

## 实现步骤

### 步骤 1: 创建 SKILL.md

**文件**: `/Users/liang/ai-work/feishu_skill/SKILL.md`

内容要点：
1. **frontmatter**:
   - `name`: feishu-article-upload
   - `description`: 飞书文档管理 skill。在飞书个人文档库创建文件夹、上传文章、更新文档。当用户说"上传到飞书"、"保存到飞书"、"转飞书文档"、"在飞书创建文件夹"、"创建飞书文件夹"时使用。
   - `disable-model-invocation`: true（用户手动触发，防止误上传）
   - `allowed-tools`: Bash(lark-cli *)

2. **SKILL.md 主体内容**:
   - 介绍 skill 用途
   - 列出 lark-cli 核心命令用法：
     - `docs +create`: 创建文档（支持 --wiki-space my_library 或 --folder-token）
     - `docs +update`: 更新文档
     - `api POST /open-apis/drive/v1/files/create_folder`: 创建文件夹
   - 提供工作流程指导：
     1. 识别用户意图（创建文件夹/上传文档/更新文档）
     2. 创建文件夹时，需要用户提供父文件夹 token（可使用 --dry-run 测试）
     3. 创建文档时使用 --wiki-space my_library 或指定 --folder-token
     4. 返回文档链接
   - 包含示例对话
   - 引用 references/lark_cli_reference.md

### 步骤 2: 创建 lark_cli_reference.md

**文件**: `/Users/liang/ai-work/feishu_skill/references/lark_cli_reference.md`

内容：
- lark-cli docs +create 命令详细用法
- lark-cli docs +update 命令详细用法
- lark-cli api 命令用法（用于创建文件夹）
- --folder-token vs --wiki-space 参数说明
- --mode 选项（overwrite, append, insert_before 等）
- 故障排除指南

## SKILL.md 示例内容

```markdown
---
name: feishu-article-upload
description: 飞书文档管理 skill。在飞书个人文档库创建文件夹、上传文章、更新文档。当用户说"上传到飞书"、"保存到飞书"、"转飞书文档"、"在飞书创建文件夹"、"创建飞书文件夹"时使用。
disable-model-invocation: true
allowed-tools: Bash(lark-cli *)
---

# 飞书文章上传

使用 lark-cli 管理飞书个人文档库中的文档。

## 核心命令

### 创建文档
在个人文档库根目录创建：
!`lark-cli docs +create --help`

在指定文件夹创建：
!`lark-cli docs +create --dry-run --title "示例" --folder-token <folder_token>`

### 创建文件夹
使用 Drive API 创建文件夹（需要父文件夹 token）：
!`lark-cli api POST /open-apis/drive/v1/files/create_folder --dry-run --data '{"name": "示例文件夹", "folder_token": "<token>"}'`

### 更新文档
!`lark-cli docs +update --help`

## 工作流程

### 场景 1: 创建文件夹
1. 用户要求"在飞书创建文件夹 XXX"
2. 询问父文件夹 token（默认可为空，在根目录创建）
3. 执行创建命令
4. 返回结果

### 场景 2: 上传文档到根目录
1. 用户要求上传文章
2. 执行: `lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library`
3. 返回文档 URL

### 场景 3: 上传文档到指定文件夹
1. 用户提供目标文件夹 token
2. 执行: `lark-cli docs +create --title "标题" --markdown "内容" --folder-token <folder-id>`
3. 返回文档 URL

### 场景 4: 更新文档
1. 用户提供文档 URL 要求更新
2. 执行: `lark-cli docs +update --doc <URL> --markdown "新内容" --mode overwrite`
3. 返回更新结果

## 参考文档
详细命令参考见 [lark_cli_reference.md](references/lark_cli_reference.md)
```

## 验证方法

1. **检查认证状态**: `lark-cli doctor`
2. **测试创建文档**: `lark-cli docs +create --title "测试" --markdown "# 测试" --wiki-space my_library`
3. **测试创建文件夹**: `lark-cli api POST /open-apis/drive/v1/files/create_folder --data '{"name": "测试文件夹"}' --dry-run`
4. **测试 skill**: 在 Claude Code 中输入 `/feishu-article-upload` 触发 skill
5. **检查文档库**: 访问飞书文档确认文档已创建

## 关键文件

| 文件 | 路径 |
|------|------|
| SKILL.md | /Users/liang/ai-work/feishu_skill/SKILL.md |
| lark_cli_reference.md | /Users/liang/ai-work/feishu_skill/references/lark_cli_reference.md |
| lark-cli | /opt/homebrew/bin/lark-cli |

## 注意事项

1. **创建文件夹**: 需要父文件夹 token。如果为空，在根目录创建（可能需要特定权限）
2. **文档创建到根目录**: 使用 `--wiki-space my_library` 即可
3. **文档创建到子文件夹**: 需要先获取目标文件夹的 token，然后使用 `--folder-token`
4. **权限**: 确保 lark-cli 已认证（`lark-cli auth status`）