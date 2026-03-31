# lark-cli 命令参考

本文档提供 lark-cli 针对飞书文档和表格操作的详细命令参考。

## 安装与认证

### 安装
```bash
npm install -g @larksuite/cli
# 或使用 npx
npx @larksuite/cli
```

### 认证
```bash
# 登录认证
lark-cli auth login

# 检查认证状态
lark-cli doctor

# 查看当前认证信息
lark-cli auth status
```

## 文档操作 (docs)

### docs +create - 创建文档

```bash
lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library
```

**参数说明**:
- `--title <string>`: 文档标题（必需）
- `--markdown <string>`: Markdown 格式的文档内容（Lark-flavored）
- `--wiki-space <id>`: 知识空间 ID，使用 `my_library` 表示个人文档库
- `--folder-token <token>`: 父文件夹 token，用于在指定文件夹中创建
- `--wiki-node <token>`: 知识库节点 token
- `--as <type>`: 身份类型 (user|bot)，默认 user

**示例**:

创建到个人文档库根目录：
```bash
lark-cli docs +create \
  --title "我的文档" \
  --markdown "# 标题\n\n正文内容" \
  --wiki-space my_library
```

创建到指定文件夹：
```bash
lark-cli docs +create \
  --title "子文档" \
  --markdown "# 内容" \
  --folder-token fldcnxxxxxx
```

### docs +update - 更新文档

```bash
lark-cli docs +update --doc <URL或token> --markdown "内容" --mode overwrite
```

**参数说明**:
- `--doc <string>`: 文档 URL 或 token（必需）
- `--markdown <string>`: 新的 Markdown 内容
- `--mode <type>`: 更新模式
  - `overwrite`: 完全覆盖原内容（默认）
  - `append`: 在末尾追加
  - `insert_before`: 在指定位置前插入
  - `insert_after`: 在指定位置后插入
  - `replace_range`: 替换指定范围
  - `delete_range`: 删除指定范围
- `--selection-by-title <title>`: 通过标题定位位置
- `--selection-with-ellipsis <text>`: 通过内容定位（格式: "开始...结束")

**示例**:

完全覆盖：
```bash
lark-cli docs +update \
  --doc "https://www.feishu.cn/wiki/xxxxx" \
  --markdown "# 新内容" \
  --mode overwrite
```

追加内容：
```bash
lark-cli docs +update \
  --doc doxcnxxxxxx \
  --markdown "\n## 新章节\n\n追加的内容" \
  --mode append
```

### docs +fetch - 获取文档内容

```bash
lark-cli docs +fetch --doc <URL或token>
```

### docs +search - 搜索文档

```bash
lark-cli docs +search --query "关键词"
```

## 文件夹操作 (drive API)

### 创建文件夹

```bash
lark-cli api POST /open-apis/drive/v1/files/create_folder \
  --data '{"name": "文件夹名", "folder_token": "<父文件夹token>"}'
```

**参数说明**:
- `name`: 新文件夹名称（必需）
- `folder_token`: 父文件夹 token（为空时在根目录创建）

**示例**:

在根目录创建文件夹：
```bash
lark-cli api POST /open-apis/drive/v1/files/create_folder \
  --data '{"name": "我的文件夹", "folder_token": ""}'
```

### 上传文件到 Drive

```bash
lark-cli drive +upload --file ./本地文件.txt --folder-token <token>
```

- 最大文件大小: 20MB

## Base 多维表格操作 (base)

### base +table-create - 创建表格

```bash
lark-cli base +table-create --base-token <token> --name "表名"
```

**参数说明**:
- `--base-token <token>`: Base token（必需）
- `--name <string>`: 表格名称（必需）
- `--fields <json>`: 字段定义 JSON 数组（可选）
- `--view <json>`: 视图定义 JSON（可选）

**示例**:

创建简单表格：
```bash
lark-cli base +table-create \
  --base-token bsptxxxxxx \
  --name "数据表"
```

### base +table-list - 列出表格

```bash
lark-cli base +table-list --base-token <token>
```

### base +record-upsert - 插入/更新记录

```bash
lark-cli base +record-upsert \
  --base-token <token> \
  --table-id <table_id> \
  --json '{"fields": {"列1": "值1", "列2": "值2"}}'
```

**参数说明**:
- `--base-token <token>`: Base token（必需）
- `--table-id <id>`: 表格 ID 或名称（必需）
- `--json <json>`: 记录 JSON 对象（必需）

**示例**:

插入新记录：
```bash
lark-cli base +record-upsert \
  --base-token bsptxxxxxx \
  --table-id "数据表" \
  --json '{"fields": {"姓名": "张三", "年龄": 25, "部门": "技术部"}}'
```

### base +record-list - 列出记录

```bash
lark-cli base +record-list --base-token <token> --table-id <table_id>
```

### base +record-delete - 删除记录

```bash
lark-cli base +record-delete \
  --base-token <token> \
  --table-id <table_id> \
  --record-id <record_id>
```

## Sheets 电子表格操作 (sheets)

### sheets +create - 创建电子表格

```bash
lark-cli sheets +create --title "表格标题"
```

**参数说明**:
- `--title <string>`: 电子表格标题（必需）

**示例**:
```bash
lark-cli sheets +create --title "销售数据"
```

### sheets +info - 获取表格信息

```bash
lark-cli sheets +info --spreadsheet-token <token>
```

返回表格 ID（sheetId）用于后续写入操作。

### sheets +write - 写入数据

```bash
lark-cli sheets +write \
  --spreadsheet-token <token> \
  --sheet-id <sheet_id> \
  --values '[["标题1", "标题2"], ["值1", "值2"]]'
```

**参数说明**:
- `--spreadsheet-token <token>`: 电子表格 token（必需）
- `--sheet-id <id>`: 工作表 ID（必需）
- `--values <json>`: 2D 数组 JSON（必需）
- `--range <string>`: 写入范围（可选，如 "A1:C5"）

**示例**:

写入标题和一行数据：
```bash
lark-cli sheets +write \
  --spreadsheet-token shtcnxxxxxx \
  --sheet-id "0" \
  --values '[["姓名", "年龄", "部门"], ["张三", "25", "技术部"]]'
```

### sheets +read - 读取数据

```bash
lark-cli sheets +read \
  --spreadsheet-token <token> \
  --sheet-id <sheet_id> \
  --range "A1:C10"
```

### sheets +append - 追加行

```bash
lark-cli sheets +append \
  --spreadsheet-token <token> \
  --sheet-id <sheet_id> \
  --values '[["张三", "25", "技术部"]]'
```

## 通用参数

| 参数 | 说明 |
|------|------|
| `--as <type>` | 身份类型: user|bot|auto |
| `--format <fmt>` | 输出格式: json|ndjson|table|csv|pretty |
| `--page-all` | 自动分页获取所有数据 |
| `--page-size <n>` | 每页数量 |
| `--page-limit <n>` | 最大页数 |
| `-o, --output <path>` | 输出文件路径 |
| `--dry-run` | 仅打印请求，不执行 |
| `--params <json>` | URL 查询参数 |
| `--data <json>` | 请求体 JSON |

## 常见问题

### 1. 认证失败

```bash
# 重新登录
lark-cli auth login

# 检查问题
lark-cli doctor
```

### 2. 权限不足

某些操作需要特定 scope，登录时添加：
```bash
lark-cli auth login --scope "drive:drive,docs:doc,base:base,sheets:spreadsheet"
```

### 3. 获取 Base/Sheets Token

- 从 URL 中提取：飞书 Base/Sheets URL 中的 token
- Base URL 格式: `https://xxx.feishu.cn/base/xxxxx`
- Sheets URL 格式: `https://xxx.feishu.cn/sheets/xxxxx`

### 4. Base vs Sheets 选择

| 特性 | Base 多维表格 | Sheets 电子表格 |
|------|---------------|----------------|
| 适用场景 | 结构化数据、记录管理 | 数值计算、数据分析 |
| 字段类型 | 多种类型（文本、数字、人员等） | 仅文本和数字 |
| 视图 | 多种视图（表格、看板等） | 单一表格 |
| 关联 | 支持多表关联 | 不支持 |

## Excel/CSV 文件读取

### 使用 Python 读取 Excel

```bash
# 读取 Excel 文件（需要 pandas 和 openpyxl）
python3 -c "
import pandas as pd
import json

df = pd.read_excel('文件路径.xlsx')
data = [df.columns.tolist()] + df.values.tolist()
print(json.dumps(data, ensure_ascii=False))
"

# 读取 CSV 文件
python3 -c "
import pandas as pd
import json

df = pd.read_csv('文件路径.csv')
data = [df.columns.tolist()] + df.values.tolist()
print(json.dumps(data, ensure_ascii=False))
"
```

### Python 依赖安装

```bash
pip3 install pandas openpyxl
```

## 参考链接

- [lark-cli GitHub](https://github.com/larksuite/cli)
- [飞书开放平台文档](https://open.feishu.cn/document/)
- [Base API 文档](https://open.feishu.cn/document/ukTMukTMukTM/uUDN04SN0QjL1QDN/bitable-v1/app-table)
- [Sheets API 文档](https://open.feishu.cn/document/ukTMukTMukTM/uUDN04SN0QjL1QDN/sheets-v3/spreadsheet-sheet)
