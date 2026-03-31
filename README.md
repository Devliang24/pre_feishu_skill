# feishu-article-upload

飞书文档管理 Claude Code Skill，支持将文章、Excel/CSV 表格上传到飞书个人文档库，自动创建文档或表格。

---

## 目录

- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [安装详解](#安装详解)
- [使用说明](#使用说明)
- [常用命令速查](#常用命令速查)
- [参数详解](#参数详解)
- [注意事项](#注意事项)
- [文件结构](#文件结构)
- [版本历史](#版本历史)

---

## 功能特性

### 核心功能

| 功能 | 说明 |
|------|------|
| 📄 创建文档 | 将 Markdown 文章创建为飞书文档 |
| 📝 更新文档 | 覆盖、追加或插入内容到已有文档 |
| 📊 创建表格 | 将 Excel/CSV 文件转换为飞书表格 |
| 🔍 获取文档 | 读取已有文档的内容 |

### Base 多维表格 vs Sheets 电子表格

| 特性 | Base 多维表格 | Sheets 电子表格 |
|------|---------------|----------------|
| 适用场景 | 结构化数据、记录管理 | 数值计算、数据分析 |
| 字段类型 | 多种类型（文本、数字、人员等） | 仅文本和数字 |
| 视图 | 多种视图（表格、看板等） | 单一表格 |
| 关联 | 支持多表关联 | 不支持 |
| 创建方式 | 需要 base token | 直接创建 |

**推荐**：大多数场景使用 Sheets 电子表格，操作更简单。

---

## 快速开始

### 1. 安装 lark-cli

```bash
npm install -g @larksuite/cli
```

### 2. 认证

```bash
lark-cli auth login --scope "docs:doc,base:base,sheets:spreadsheet"
```

### 3. 开始使用

在 Claude Code 中直接说：

- "上传到飞书" - 上传文章
- "上传表格到飞书" - 上传 Excel/CSV 文件
- "创建飞书表格" - 创建新表格

---

## 安装详解

### 前置要求

| 软件 | 版本 | 说明 |
|------|------|------|
| Node.js | >= 14 | 用于运行 lark-cli |
| npm/yarn | 最新版 | 包管理器 |
| Python | >= 3.7 | 用于读取 Excel/CSV 文件 |
| pandas | 最新版 | Python 数据处理库 |

### 安装步骤

#### 方法一：克隆仓库（推荐）

```bash
git clone https://github.com/Devliang24/pre_feishu_skill.git ~/.claude/skills/feishu-article-upload
```

#### 方法二：手动复制

```bash
mkdir -p ~/.claude/skills/feishu-article-upload
# 克隆后复制
cp -r SKILL.md references/ ~/.claude/skills/feishu-article-upload/
```

### 安装 Python 依赖（用于处理 Excel/CSV）

```bash
pip3 install pandas openpyxl
```

### 认证配置

```bash
# 查看认证状态
lark-cli doctor

# 重新认证（如权限不足）
lark-cli auth login --scope "docs:doc,base:base,sheets:spreadsheet"
```

---

## 使用说明

### 触发方式

在 Claude Code 对话中直接说：

| 触发语句 | 操作 |
|----------|------|
| "上传到飞书" | 创建新文档 |
| "保存到飞书" | 创建新文档 |
| "转飞书文档" | 创建新文档 |
| "上传表格到飞书" | 创建表格 |
| "把 Excel 传到飞书" | 创建表格 |
| "创建飞书表格" | 创建表格 |

### 上传文档

#### 操作步骤

1. 用户提供文章内容
2. 提取标题和正文
3. 执行创建命令
4. 返回文档链接

#### 示例

**输入**：
```
上传到飞书：# 我的文章
这是文章正文内容。
```

**执行命令**：
```bash
lark-cli docs +create --title "我的文章" --markdown "# 我的文章\n\n这是文章正文内容。" --wiki-space my_library
```

**输出**：
```
文档创建成功！
链接：https://www.feishu.cn/docx/xxxxx
```

### 上传表格

#### 操作步骤

1. 用户提供 Excel/CSV 文件路径
2. 使用 Python 读取文件内容
3. 创建 Sheets 电子表格
4. 写入数据
5. 返回表格链接

#### 示例

**输入**：
```
上传这个表格到飞书：@data.xlsx
```

**执行步骤**：
```bash
# 1. 读取 Excel 文件
python3 -c "
import pandas as pd
import json
df = pd.read_excel('data.xlsx')
data = [df.columns.tolist()] + df.values.tolist()
print(json.dumps(data, ensure_ascii=False))
"

# 2. 创建表格
lark-cli sheets +create --title "数据表"

# 3. 获取 sheet-id
lark-cli sheets +info --spreadsheet-token <token>

# 4. 写入数据
lark-cli sheets +write --spreadsheet-token <token> --sheet-id <sheet-id> --values '[["列1", "列2"], ["值1", "值2"]]'
```

**输出**：
```
表格创建成功！
链接：https://xxx.feishu.cn/sheets/xxxxx
```

### 更新文档

#### 操作步骤

1. 用户提供文档 URL 和新内容
2. 执行更新命令
3. 返回更新结果

#### 示例

**输入**：
```
更新这个文档：https://www.feishu.cn/docx/xxxxx
新内容：# 更新后的标题\n\n这是更新后的内容。
```

**执行命令**：
```bash
lark-cli docs +update --doc "https://www.feishu.cn/docx/xxxxx" --markdown "# 更新后的标题\n\n这是更新后的内容。" --mode overwrite
```

---

## 常用命令速查

### 文档操作

| 操作 | 命令 |
|------|------|
| 创建文档 | `lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library` |
| 更新文档 | `lark-cli docs +update --doc <URL> --markdown "内容" --mode overwrite` |
| 获取文档 | `lark-cli docs +fetch --doc <URL>` |

### 表格操作

| 操作 | 命令 |
|------|------|
| 创建 Sheets | `lark-cli sheets +create --title "标题"` |
| 获取 sheet-id | `lark-cli sheets +info --spreadsheet-token <token>` |
| 写入数据 | `lark-cli sheets +write --spreadsheet-token <token> --sheet-id <id> --values '[[...]]'` |
| 创建 Base | `lark-cli base +table-create --base-token <token> --name "表名"` |
| 插入记录 | `lark-cli base +record-upsert --base-token <token> --table-id <id> --json '{"fields": {...}}'` |

### 系统命令

| 操作 | 命令 |
|------|------|
| 检查认证 | `lark-cli doctor` |
| 查看帮助 | `lark-cli --help` |

---

## 参数详解

### docs +create

| 参数 | 必填 | 说明 |
|------|------|------|
| `--title` | 是 | 文档标题 |
| `--markdown` | 是 | Markdown 格式内容 |
| `--wiki-space my_library` | 是 | 固定值，表示个人文档库 |

### docs +update

| 参数 | 必填 | 说明 |
|------|------|------|
| `--doc` | 是 | 文档 URL 或 token |
| `--markdown` | 是 | 新内容 |
| `--mode` | 否 | 模式：overwrite/append/insert_before/insert_after |

### sheets +create

| 参数 | 必填 | 说明 |
|------|------|------|
| `--title` | 是 | 表格标题 |

### sheets +write

| 参数 | 必填 | 说明 |
|------|------|------|
| `--spreadsheet-token` | 是 | 表格 token |
| `--sheet-id` | 是 | 工作表 ID |
| `--values` | 是 | 2D 数组 JSON |

### base +table-create

| 参数 | 必填 | 说明 |
|------|------|------|
| `--base-token` | 是 | Base token |
| `--name` | 是 | 表格名称 |

---

## 注意事项

### 1. Sheets 创建位置

`lark-cli sheets +create` **只能在根目录创建**，不支持指定文件夹。创建后可手动拖动到目标文件夹。

### 2. 认证权限

如遇到权限不足错误，重新授权：

```bash
lark-cli auth login --scope "docs:doc,base:base,sheets:spreadsheet"
```

### 3. Excel/CSV 读取

读取 Excel 需要 Python 和 pandas：

```bash
pip3 install pandas openpyxl
```

### 4. Base 需要已有 token

Base 多维表格需要先有 base token，无法直接创建空白 base。

---

## 文件结构

```
feishu-article-upload/
├── SKILL.md                      # Skill 主文件（Claude Code 读取）
├── README.md                     # 本文档
└── references/
    └── lark_cli_reference.md     # lark-cli 详细命令参考
```

---

## 版本历史

| 版本 | 更新内容 |
|------|----------|
| v1.4.0 | 移除文件夹创建功能，精简文档 |
| v1.3.0 | 增加 Sheets 文件夹限制说明 |
| v1.2.0 | 支持 Excel/CSV 上传 |
| v1.1.0 | 支持表格操作 |
| v1.0.0 | 初始版本，支持文档操作 |

---

## License

MIT
