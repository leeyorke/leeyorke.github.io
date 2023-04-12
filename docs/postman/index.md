# postman脚本使用正则提取变量

### 遇到的坑
假设接口resp响应如下，我们要提取里面的sessionId ，因为resp里面包含`\`字符。所以需要在`\`前面加`\`形成`\\`才能正确提取

```bash
{
  "status": "success",
  "data": "{\"title\":\"\",\"sessionName\":\"zentaosid\",\"sessionID\":\"qr4taa6q43evbder1iistqre5u\",\"rand\":100,\"pager\":null}",
  "md5": "4a199d04a0a2df3376e67213957a1347"
}
```
### 使用字面量正则提取
`/"sessionID\\":\\"(.*?)\\"/`
```javascript
resp = pm.response.text();
console.log("提取的resp为:",resp);
sessionId = resp.match(/"sessionID\\":\\"(.*?)\\"/)[1];
// 设置为全局变量
pm.globals.set("ssesionId", sessionId);
console.log("提取的sessionId为:",sessionId);
```
### 使用构造函数正则
`sessionID\\\\":\\\\"(.*?)\\\\"`
```javascript
resp = pm.response.text();
console.log("提取的resp为:",resp);
pt = new RegExp('sessionID\\\\":\\\\"(.*?)\\\\",');
console.log("正则为:",pt.toString());
sessionId = resp.match(pt)[1];
pm.globals.set("ssesionId", sessionId);
console.log("提取的sessionId为:",sessionId);
```
### 后记
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)上说：
>当使用构造函数创造正则对象时，需要常规的字符转义规则（在前面加反斜杠 \）。

以下是等价的
```javascript
var re = new RegExp("\\w+");
var re = /\w+/;
```
就是说如果使用了元字符
>比如 \W、\S之类的

字面量正则里面不需要加`\`转义，比如：`/\w+/`，构造函数就需要写成这种：

```javascript
pattern  = new RegExp("\\w+")
```

但如果正则里面包含`$`, ` (`,  `)`,  `*`,  `+`,  `.`,  `[`,  `]`,  `?`,  `\`,  `^`,  `{`,  `}`,  `|`。则
* 字面量正则需要加`\`转义。如`/"sessionID\\":\\"(.*?)\\"/`
* 构造函数正则需要加`\\\`，也就是三个`\`。

因为

```javascript
pattern = new RegExp('sessionID\\\\":\\\\"(.*?)\\\\",')
pattern_str = pattern.toString()

// 输出结果为
/sessionID\\":\\"(.*?)\\"/
```
而字符串的match函数接受的是一个字面量正则，如果你写成

```javascript
pattern = new RegExp('sessionID\\":\\"(.*?)\\",')
```
`pattern.toString()`后会是`/sessionID\":\"(.*?)\"/`，匹配结果为null。

故
在postman中使用正则提取变量，如果你的接口响应里存在转义字符，还是使用字面量正则好一些。

