---
name: feishu-article-upload
version: 1.5.0
description: 飞书文档管理 skill。将文章、Excel 表格同步到本地后上传到飞书个人文档库，创建文档和表格。当用户提供文件路径或 URL 并说"上传到飞书"、"保存到飞书"、"转飞书文档"、"上传表格到飞书"、"把 Excel 传到飞书"、"创建飞书表格"时自动触发。
allowed-tools: Bash(lark-cli *), Read, Glob
---

# 飞书文档上传

使用 lark-cli 管理飞书个人文档库中的文档和表格。

## 核心功能

- 文档管理：创建文档、更新文档、获取文档
- 表格管理：同步本地文件后，读取 Excel/CSV，创建飞书表格

## 工作流程

### 重要：先同步到本地

上传文件前，**必须先将文件同步到本地**，确保文件存在且可读。

**同步来源**：
1. 用户直接提供文件路径（如 `@Token计费测试用例.xlsx`）
2. 用户提供 URL（如网盘链接、GitHub 地址等）

**同步方式**：
- 本地文件：使用 `Read` 或 `Glob` 工具读取
- URL 文件：使用 `Bash` + `curl` 下载

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

### Excel/CSV 文件上传流程

**步骤**：

1. **同步文件到本地**
   - 本地文件：直接读取
   - URL 文件：用 curl 下载

2. **读取文件内容**
   - 使用 Python pandas 读取 Excel/CSV

3. **创建 Sheets 电子表格**

4. **写入数据**

### 具体命令

**下载远程文件（如网盘、GitHub）**：
```bash
# 下载文件到本地
curl -L -o 本地文件名.xlsx "文件URL"

# 示例：下载 GitHub  releases 文件
curl -L -o data.xlsx "https://github.com/user/repo/releases/download/v1.0/data.xlsx"
```

**读取 Excel/CSV 文件**：
```bash
python3 -c "
import pandas as pd
import json

df = pd.read_excel('本地文件路径.xlsx')
# 转为 2D 数组
data = [df.columns.tolist()] + df.values.tolist()
print(json.dumps(data, ensure_ascii=False))
"
```

**创建表格并写入**：
```bash
# 1. 创建电子表格
lark-cli sheets +create --title "表格标题"

# 2. 获取 sheet-id
lark-cli sheets +info --spreadsheet-token <token>

# 3. 写入数据
lark-cli sheets +write \
  --spreadsheet-token <token> \
  --sheet-id <sheet_id> \
  --values '[["列1", "列2"], ["值1", "值2"]]'
```

## 工作流程详解

### 场景 1: 上传本地文件

1. 用户提供本地文件路径（如 `@Token计费测试用例.xlsx`）
2. 使用 `Read` 确认文件存在
3. 使用 Python 读取内容
4. 创建 Sheets 并写入
5. 返回链接

### 场景 2: 上传远程 URL 文件

1. 用户提供 URL（如网盘链接、GitHub 地址）
2. 使用 curl 下载到本地临时目录
3. 使用 Python 读取内容
4. 创建 Sheets 并写入
5. 清理临时文件
6. 返回链接

### 场景 3: 更新文档

1. 用户提供文档 URL 要求更新
2. 从用户提供提取新内容
3. 执行: `lark-cli docs +update --doc <URL> --markdown "新内容" --mode overwrite`
4. 返回更新结果

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
| 下载文件 | `curl -L -o 本地文件名.xlsx "文件URL"` |

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

## 注意事项

### 1. 先同步到本地

**重要**：上传前必须确保文件在本地存在。远程 URL 必须先下载。

### 2. Sheets 创建位置

`lark-cli sheets +create` **只能在根目录创建**，返回链接后用户可手动拖动到目标位置。

### 3. 认证要求

部分操作需要特定 scope：
- `docs:doc` - 文档操作
- `sheets:spreadsheet` - Sheets 操作

如果提示权限不足，重新授权：
```bash
lark-cli auth login --scope "docs:doc,sheets:spreadsheet"
```

### 4. 临时文件处理

下载的远程文件使用完毕后可删除：
```bash
rm 本地文件名.xlsx
```

## 参考文档

详细命令参考见 [lark_cli_reference.md](references/lark_cli_reference.md)
