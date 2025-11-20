---
title: "sunflower演进之路|用例model设计"
date: 2025-11-19T11:59:17+08:00
lastmod:
draft: false
tags: ["测试", "测试用例", 'python']
categories:
description: ""
---

本文档介绍 `sunflower` 测试框架的用例组成。

## 基类
`sunflower` 测试用例文档由以下三个基类构成。
```yaml
config:
background:
testcases:
```

### `config` 基类
`config` 会引用 `api` 文档中的配置。 `api` 也是用yaml编写，需要在测试前定义好。作用是定义好接口测试
的文档。

### `background` 基类
`background` 作用是定义整个文档的测试背景，下面可以设置各种组件，目前可以设置：
- environment_variable
- variable
- sql
- script
- wait

### `testcases` 基类

`testcases` 下为 `list` 形式，可以编写数个测试用例。

一个完整的 `sunflower` 测试用例文档如下：
```yaml
config: !include ../api/tokyo.accounts.login.yaml
background:
    environment_variable:
        mobile: 12344445555
        password: 123456
    variable:
        foo: 2023
        bar: ok

testcases:
-   scenario: 验证用正确的手机号和密码，登录成功
    tag: ['P1', 'login', 'login_normal']
    description: |
        作为一个企业管理员
        我想要输入正确的手机号和密码
        以便登录成功
    ref: tokyo.accounts.func.01.01
    precondition:
        # 环境变量组件：设置环境变量
        environment_variable:
            mobile: 12344445555
            password: 123456
        # 临时变量组件：设置临时变量
        variable:
            foo: 2023
            bar: ok
        # 脚本组件：调用一个Py脚本
        script:
        # 数据库组件：数据库操作
        sql:
        # 延时组件：延时操作
        wait:

    postcondition:
    steps:
    -   request:
            headers:
                Content-Type: "application/json;charset=UTF-8"
            json: |
                {
                    "mobile": "{{mobile}}",
                    "password": "{{password}}"
                }
```

## 组件
`sunflower` 测试用例文档由各种组件组成。组件均可以在测试用例中复用，意味在测试用例中的设置数量
可以为1~N个。
### environment_variable
环境变量组件。该组件将为 `sunflower` 测试设置环境变量。环境变量的作用域为整个测试环境，当测试环境变化时，
该变量将自动销毁。

```yaml
environment_variable:
    mobile: 12344445555
    password: 123456
    foo: test
    bar: var
```