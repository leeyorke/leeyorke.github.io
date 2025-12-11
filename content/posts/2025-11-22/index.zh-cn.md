---
title: "一日一技|yaml中的多行字符串"
date: 2025-11-22T08:12:37+08:00
lastmod:
draft: false
tags: ["yaml"]
categories:
description: ""
---

`yaml` 中多行字符串的各种写法。

## 原始yaml
```yaml
paragraph:
  Blank lines denote
  paragraph breaks
alias:
  bar: baz
```
转成json
```json
{
  "paragraph": "Blank lines denote paragraph breaks",
  "alias": {
    "bar": "baz"
  }
}
```
---
## `|`
作用：`|` 会给**每一行**末尾加上 `\n` 字符串。

示例
```yaml
paragraph: |
  Blank lines denote
  paragraph breaks
alias:
  bar: baz
```
转成json
```json
{
  "paragraph": "Blank lines denote\nparagraph breaks\n",
  "alias": {
    "bar": "baz"
  }
}
```
---
## `|-`
作用：`|-` 在 `|` 的基础上会将最后一行末尾的 `\n` 去掉。

示例
```yaml
paragraph: |-
  Blank lines denote
  paragraph breaks
alias:
  bar: baz
```
转成json
```json
{
  "paragraph": "Blank lines denote\nparagraph breaks",
  "alias": {
    "bar": "baz"
  }
}
```
---
## `>`
作用：`>` 会给**最后一行**末尾加上 `\n` 字符串。

示例
```yaml
paragraph: >
  Blank lines denote
  paragraph breaks
alias:
  bar: baz
```
转成json
```json
{
  "paragraph": "Blank lines denote paragraph breaks\n",
  "alias": {
    "bar": "baz"
  }
}
```
---
## `>-`
作用：`>-` 在 `>` 基础上会将**最后一行**末尾 `\n` 去掉。

示例
```yaml
paragraph: >-
  Blank lines denote
  paragraph breaks
alias:
  bar: baz
```
转成json
```json
{
  "paragraph": "Blank lines denote paragraph breaks",
  "alias": {
    "bar": "baz"
  }
}
```
---
## 总结
|操作符|含义|等效于|备注|
|--|--|--|--|
|`\|`|给每一行的末尾加上`\n`|无|无|
|`\|+`|给每一行的末尾加上`\n`|`\|`|不常用|
|`\|-`|去掉最后一行末尾的`\n`|无|常用|
|`>`|给最后一行的末尾加上`\n`|无|常用|
|`>+`|给最后一行的末尾加上`\n`|`>`|不常用|
|`>-`|去掉最后一行的末尾加上`\n`|什么符号也不加|不常用|

常用：
- `|`
- `>`
- `|-`