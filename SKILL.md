---
name: feishu-article-upload
version: 1.4.0
description: 飞书文档管理 skill。将文章、Excel 表格上传到飞书个人文档库，创建文档和表格。当用户提供文件路径或 URL 并说"上传到飞书"、"保存到飞书"、"转飞书文档"、"上传表格到飞书"、"把 Excel 传到飞书"、"创建飞书表格"时自动触发。
allowed-tools: Bash(lark-cli *), Read, Glob
---

# 飞书文档上传

使用 lark-cli 管理飞书个人文档库中的文档和表格。

## 核心功能

- 文档管理：创建文档、更新文档、获取文档
- 表格管理：读取 Excel/CSV 文件，创建 Base 多维表格或 Sheets 电子表格

## 文档操作

### 创建文档

在个人文档库根目录创建：
```bash
lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library
```

### 更新文档

```bash
lark-cli docs +update --doc <URL或token> --markdown "新内容" --mode overwrite
```

### 获取文档内容

```bash
lark-cli docs +fetch --doc <URL或token>
```

## 表格操作

### Excel/CSV 文件转换为飞书表格

**推荐方式：使用 Sheets 电子表格**

1. 读取 Excel/CSV 文件内容
2. 创建 Sheets 电子表格
3. 获取 sheet-id
4. 写入数据

```bash
# 读取 Excel 文件（使用 Python pandas）
python3 -c "import pandas as pd; df = pd.read_excel('文件路径.xlsx'); print(df.to_json(orient='records'))"

# 创建电子表格
lark-cli sheets +create --title "表格标题"

# 写入数据（2D 数组）
lark-cli sheets +write \
  --spreadsheet-token <token> \
  --sheet-id <sheet_id> \
  --values '[["列1", "列2"], ["值1", "值2"]]'
```

### 方式一：Base 多维表格

创建表格：
```bash
lark-cli base +table-create --base-token <token> --name "表名"
```

插入/更新记录：
```bash
lark-cli base +record-upsert \
  --base-token <token> \
  --table-id <table_id> \
  --json '{"fields": {"列1": "值1", "列2": "值2"}}'
```

### 方式二：Sheets 电子表格

创建电子表格：
```bash
lark-cli sheets +create --title "表格标题"
```

获取表格信息（含 sheet-id）：
```bash
lark-cli sheets +info --spreadsheet-token <token>
```

写入数据（2D 数组）：
```bash
lark-cli sheets +write \
  --spreadsheet-token <token> \
  --sheet-id <sheet_id> \
  --values '[["标题1", "标题2"], ["值1", "值2"]]'
```

## 工作流程

### 场景 1: 上传文档到根目录

1. 用户要求上传文章
2. 从用户提供的内容中提取标题和正文
3. 执行: `lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library`
4. 返回文档 URL

### 场景 2: 更新文档

1. 用户提供文档 URL 要求更新
2. 从用户提供提取新内容
3. 执行: `lark-cli docs +update --doc <URL> --markdown "新内容" --mode overwrite`
4. 返回更新结果

### 场景 3: Excel/CSV 文件上传到飞书

1. 用户提供文件路径（如 `@Token计费测试用例.xlsx`）
2. 使用 Python 读取 Excel/CSV 文件
3. 提取表头和数据行
4. 创建 Sheets 电子表格
5. 获取 sheet-id
6. 写入数据
7. 返回表格链接

**Python 读取 Excel 示例**:
```python
import pandas as pd
import json

df = pd.read_excel('文件路径.xlsx')
# 转为 2D 数组
data = [df.columns.tolist()] + df.values.tolist()
print(json.dumps(data, ensure_ascii=False))
```

## 常用命令速查

| 操作 | 命令 |
|------|------|
| 检查认证 | `lark-cli doctor` |
| 创建文档 | `lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library` |
| 更新文档 | `lark-cli docs +update --doc <URL> --markdown "内容" --mode overwrite` |
| 获取文档 | `lark-cli docs +fetch --doc <URL>` |
| 创建 Sheets | `lark-cli sheets +create --title "标题"` |
| 获取 sheet-id | `lark-cli sheets +info --spreadsheet-token <token>` |
| 写入 Sheets 数据 | `lark-cli sheets +write --spreadsheet-token <token> --sheet-id <id> --values '[[...]]'` |
| 创建 Base 表格 | `lark-cli base +table-create --base-token <token> --name "表名"` |
| 插入 Base 记录 | `lark-cli base +record-upsert --base-token <token> --table-id <id> --json '{"fields": {...}}'` |

## 参数说明

### docs +create 参数
- `--title`: 文档标题
- `--markdown`: Markdown 格式的文档内容
- `--wiki-space my_library`: 创建到个人文档库

### docs +update 参数
- `--doc <URL或token>`: 文档 URL 或 token
- `--markdown <内容>`: 新的 Markdown 内容
- `--mode`: 更新模式 (overwrite/append/insert_before/insert_after)

### sheets +create 参数
- `--title <string>`: 电子表格标题（必需）

### sheets +write 参数
- `--spreadsheet-token <token>`: 电子表格 token（必需）
- `--sheet-id <id>`: 工作表 ID（必需）
- `--values <json>`: 2D 数组 JSON

### base +table-create 参数
- `--base-token <token>`: Base token（必需）
- `--name <string>`: 表格名称（必需）

### base +record-upsert 参数
- `--base-token <token>`: Base token（必需）
- `--table-id <id>`: 表格 ID 或名称（必需）
- `--json <json>`: 记录 JSON，包含 fields 字段

## 注意事项

### Sheets 创建位置

`lark-cli sheets +create` **只能在根目录创建**，返回链接后用户可手动拖动到目标位置。

### 认证要求

部分操作需要特定 scope：
- `docs:doc` - 文档操作
- `base:base` - Base 表格操作
- `sheets:spreadsheet` - Sheets 操作

如果提示权限不足，重新授权：
```bash
lark-cli auth login --scope "docs:doc,base:base,sheets:spreadsheet"
```

## 参考文档

详细命令参考见 [lark_cli_reference.md](references/lark_cli_reference.md)
