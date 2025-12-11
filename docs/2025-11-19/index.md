# sunflower演进之路|用例model设计


本文档介绍 `sunflower` 测试框架的用例结构组成。

## 基本组件
`sunflower` 测试用例文档由以下三个基本组件构成。
```yaml
config:
background:
testcases:
```

### `config`
`config` 通过 `!include` 标签用来引用 `api` 文档路径。 `api` 也是用yaml编写，需要在测试前定义好，
作用是定义好接口测试的各种配置。

### `background`
`background` 下为 `list` 形式, 作用是定义整个文档的测试背景，在它下面可以设置各种组件，目前可以设置：
- vars
- sql
- script
- send
- request

**vars**
用于定义测试一组环境变量。
```yaml
-   vars:
        tmp_mobile: 15640079705
        tmp_password: 123456
```

**sql**
用于SQL操作。
```yaml
-   sql:
        description: 查询
        database: knight
        expresion: |
            select username from users;
```
**script**
用于执行一个自定义脚本，该脚本由用户自行编写。
```yaml
-   script:
        call: http.send
        kwargs:
            method: get
            url: '/user_types/'
            headers:
                Content-Type: "application/json;charset=UTF-8"
```
**request**
用于发送一个请求。
```yaml
-   request:
        method: post
        url: '/users/'
        json: |
            {
                "username": "{{pm.name}}",
                "user_types": ["{{SA}}"],
                "mobile": "{{tmp_mobile}}",
                "email": "{{pm.email}}",
                "password": "{{tmp_password}}",
                "is_active": true
            }
```
**send**
用于发送一个请求, 快捷方式。
```yaml
-   send: !include ../api/turin.user_types.list.yaml
```
### `testcases`

`testcases` 下为 `list` 形式，可以编写数个测试用例。
一个测试用例结构组成如下：
```yaml
-   scenario: 正常登录场景
    tag: normal,login
    description: 用新创建的用户名密码登录，期望登录成功
    ref: tokyo.accounts.login.func.001
    precondition:
    request:
        json: |
            {
                "mobile": "{{tmp_mobile}}",
                "password": "{{tmp_password}}"
            }
    postcondition:
        -   export:
                select: json
                vars:
                    user_id: data.profile.id
                    username: data.profile.username
        -   assertion:
                description: 校验http状态码等于200
                select: json
                expect: [ _code_, '==', 200]
        -   assertion:
                description: 校验接口耗时小于200ms
                select: json
                expect: [ _time_, '<', 200]
        -   assertion:
                description: 校验接口响应状态码等于0
                select: json
                expect: [ status_code, '==', 0]
        -   assertion:
                description: 校验user_id正确
                select: json
                expect: [ 'data.profile.mobile', 'seq', '{{tmp_mobile}}']
        -   assertion:
                description: 校验username正确
                select: json
                expect: [ 'data.profile.username', '==', '{{username}}']

```

- scenario： 测试用例表标题
- tag：为此用例设置一个或多个tag
- description：详细描述该用例的用来做什么
- ref：关联的测试需求编号
- precondition：前置条件。可以设置 vars, sql, script, send, request等子组件。
- request：测试数据，即接口请求body
- postcondition：后置条件。可以设置 assertion, export, sql, script, send, request等子组件


一个完整的 `sunflower` 测试用例文档如下：
```yaml
config: !include ../api/turin.accounts.login.yaml
background:
-   vars:
        tmp_mobile: 15640079705
        tmp_password: 123456
-   request:
        method: post
        url: '/users/'
        json: |
            {
                "username": "{{pm.name}}",
                "user_types": ["{{SA}}"],
                "mobile": "{{tmp_mobile}}",
                "email": "{{pm.email}}",
                "password": "{{tmp_password}}",
                "is_active": true
            }
testcases:
##################################################################
-   scenario: 正常登录场景
    tag: normal,login
    description: 用新创建的用户名密码登录，期望登录成功
    # 需求关联
    ref: tokyo.accounts.login.func.001
    precondition:
    request:
        json: |
            {
                "mobile": "{{tmp_mobile}}",
                "password": "{{tmp_password}}"
            }
    postcondition:
        -   export:
                select: json
                vars:
                    user_id: data.profile.id
                    username: data.profile.username
        -   assertion:
                description: 校验http状态码等于200
                select: json
                expect: [ _code_, '==', 200]
        -   assertion:
                description: 校验接口耗时小于200ms
                select: json
                expect: [ _time_, '<', 200]
        -   assertion:
                description: 校验接口响应状态码等于0
                select: json
                expect: [ status_code, '==', 0]
        -   assertion:
                description: 校验user_id正确
                select: json
                expect: [ 'data.profile.mobile', 'seq', '{{tmp_mobile}}']
        -   assertion:
                description: 校验username正确
                select: json
                expect: [ 'data.profile.username', '==', '{{username}}']

```

## 子组件
`sunflower` 测试用例文档由各种组件组成。组件均可以在测试用例中复用，意味在测试用例中的设置数量
可以为1~N个。
### vars
变量组件。该组件将为 `sunflower` 测试设置环境变量。
- 在 vars 内设置的变量将会存储在sunflower上下文(context.e)中。
- 在 vars 内设置的变量都可以在测试用例文档中引用, 以 `"{{var}}"` 形式。


```yaml
vars:
    foo: 2023
    bar: ok
```

### export
变量提取组件。该组件用于从前置步骤(sql, script, send, request)中提取变量,
存储在sunflower上下文(context.e)。组件语法如下：
```yaml
-   export:
        select: json
        vars:
            user_id: data.profile.id
            username: data.profile.username
```
- select: 从 json, text，xml中提取变量
- vars: key 为变量名，value为提取表达式。
    - 若select为json，则表达式为jmespath。
    - 若select为text，则表达式为正则表达式。
    - 若select为xml，则表达式为xpath。

从 export 提取的变量，都可以在后续步骤中以 `{{}}` 形式使用。

与 `vars` 组件区别：
|序号|vars|export|
|-|-|-|
|1|key 为变量名, value为人为定义的变量值|key 为变量名。value为 jmespath 表达式|
|2|手动定义的环境变量|从前置步骤中提取的所需数据，设置为环境变量|

## 断言
### 断言组件
断言组件书写形式如下：
```yaml
-   assertion:
        description: 校验http状态码等于200
        select: json
        expect: [ _code_, '==', 200]
```

- description: 描述断言的对象及目的
- select: 选择从 json, text，xml中提取对应变量及进行断言。
- expect: 断言表达式。[实际结果, 断言条件, 期望结果]
    - 实际结果可以是：jmespath、xpath、正则表达式之一
    - 断言条件见下方详述
    - 期望结果为手动填写

### 断言条件

`sunflower` 中, 断言表达式一般写为 `[actual, condition, expect]`, `condition` 支持以下判断条件：
- **eq**: 等于，一般用来判断整型。
- **neq**: 不等于，一般用来判断整型。
- **lt**: 小于，一般用来判断整型。
- **lt**: 小于等于，一般用来判断整型。
- **gt**: 大于，一般用来判断整型。
- **gteq**: 大于等于，一般用来判断整型。
- **seq**: 等于，一般用来判断字符串。
- **leq**: 长度等于，一般用来判断字符串长度。
- **lgteq**: 长度大于等于，一般用来判断字符串长度。
- **llt**: 长度小于，一般用来判断字符串长度。
- **llteq**: 长度小于等于，一般用来判断字符串长度。
- **sfeq**: 长度模糊相等。
- **scot**: 字符串包含, 判断期望结果包含在实际结果中。
- **cot**: 通用包含, 判断期望结果包含在实际结果中，支持类型 `list`, `tuple`, `str`, `bytes`。
- **cod**: 通用包含, 判断实际结果包含在期望结果中，支持类型 `list`, `tuple`, `str`, `bytes`。
- **sw**: 开头判断, 判断实际结果的开头是否与期望结果开头相等。
- **ew**: 结尾判断, 判断实际结果的结尾是否与期望结果开头相等。
