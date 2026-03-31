# feishu-article-upload

飞书文档管理 Claude Code Skill，支持上传文章、Excel 表格到飞书个人文档库。

## 功能

- 📄 **文档管理**：创建文档、更新文档、获取文档
- 📊 **表格管理**：读取 Excel/CSV 文件，创建飞书表格

## 安装

### 方法一：复制文件（推荐）

```bash
mkdir -p ~/.claude/skills/feishu-article-upload
cp -r SKILL.md references/ ~/.claude/skills/feishu-article-upload/
```

### 方法二：从 GitHub 克隆

```bash
git clone https://github.com/Devliang24/pre_feishu_skill.git ~/.claude/skills/feishu-article-upload
```

## 使用前提

### 安装 lark-cli

```bash
npm install -g @larksuite/cli
```

### 认证

```bash
lark-cli auth login --scope "docs:doc,base:base,sheets:spreadsheet"
lark-cli doctor  # 检查认证状态
```

## 使用方法

在 Claude Code 中直接说：

- "上传到飞书" - 上传文章
- "上传表格到飞书" - 上传 Excel/CSV 文件
- "把 Excel 传到飞书" - 上传 Excel 文件
- "创建飞书表格" - 创建表格

## 命令速查

| 操作 | 命令 |
|------|------|
| 创建文档 | `lark-cli docs +create --title "标题" --markdown "内容" --wiki-space my_library` |
| 创建 Sheets | `lark-cli sheets +create --title "标题"` |
| 更新文档 | `lark-cli docs +update --doc <URL> --markdown "内容" --mode overwrite` |
| 写入数据 | `lark-cli sheets +write --spreadsheet-token <token> --sheet-id <id> --values '[[...]]'` |

## 版本

- **v1.4.0** - 移除文件夹创建功能，精简文档

## License

MIT
