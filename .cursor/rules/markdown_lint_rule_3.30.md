---
description: 
globs: 
alwaysApply: true
---
# Markdown Lint 规则

## 描述

此规则文件用于确保所有 Markdown 文档严格遵循 markdownlint 规范，提高文档质量和一致性。

## 规则内容

在处理 Markdown 文件时，请严格遵循以下规范：

1. **文档结构规则**
   - MD001：标题级别应该只递增一级
   - MD003：标题样式应保持一致（使用 ATX 风格）
   - MD022：标题前后应有空行
   - MD023：标题前不应有空格
   - MD025：文档中只应有一个一级标题
   - MD041：文件应以标题开始
   - MD036：不应使用强调代替标题

2. **空格与空行规则**
   - MD009：行尾不应有多余空格
   - MD010：不使用硬 Tab，应使用空格
   - MD012：多个连续空行应减少为一个
   - MD027：引用块标记后只应有一个空格
   - MD030：列表标记后应有一个空格
   - MD037：强调标记内不应有空格
   - MD038：代码段标记内不应有空格
   - MD039：链接文本内不应有空格
   - MD047：文件应以空行结束

3. **列表与段落规则**
   - MD004：无序列表样式应保持一致
   - MD005：列表缩进应保持一致
   - MD007：无序列表缩进应为 2 或 4 个空格
   - MD028：引用块之间不应只有空行
   - MD029：有序列表编号应采用递增方式
   - MD030：列表标记后应有一个空格
   - MD032：列表项前后应有空行

4. **代码与链接规则**
   - MD011：链接语法不应颠倒
   - MD031：代码块前后应有空行
   - MD035：水平分隔线样式应保持一致
   - MD040：代码块应指定语言
   - MD042：不应有空链接

5. **最佳实践**
   - 代码块应使用三个反引号（```）而非缩进，并指定语言类型
   - 文档中的链接和图片应使用标准 Markdown 语法，避免使用 HTML 标签
   - 确保文档结构清晰，标题层级合理，没有跳级
   - 使用标准的 Markdown 语法，避免使用特定平台的扩展语法

## 示例

```markdown
# 文档标题

## 第一部分

这是一段正文，包含**加粗**和*斜体*文本。

- 列表项 1
- 列表项 2
  - 嵌套列表项
  - 另一个嵌套列表项

## 第二部分

这是一段包含[链接](https://example.com)的文本。

```python
def hello_world():
    print("Hello, world!")
```

## 参考

- [markdownlint 规则详情](mdc:https:/github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md)
- [Markdown 语法指南](mdc:https:/www.markdownguide.org)
- [GitHub Markdown 指南](mdc:https:/docs.github.com/cn/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
