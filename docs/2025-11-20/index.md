# hugo 如何显示 mermaid 流程图


`hugo` 默认是不显示 `mermaid` 流程图的, 如果写了 `mermaid` 代码, 不会自动渲染，这就需要自行配置下。

## windows 下配置流程

**第一步**。在博客根目录的 `layouts` 目录下创建 `_default` 目录，拷贝主题下对应的内容

```PowerShell
cd layouts
mkdir _default
cp themes\LoveIt\layouts\_default\baseof.html layouts\_default
```

**第二步**。打开 `baseof.html` 文件，在 `</body>` 标签最后面增加以下内容

```javascript
{{ if .Store.Get "hasMermaid" }}
  <script type="module">
    import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.esm.min.mjs';
    mermaid.initialize({ startOnLoad: true });
  </script>
{{ end }}
```

**第三步**。在 `layouts/_default` 目录下创建 `_markup` 目录及文件

```PowerShell
cd layouts\_default\
mkdir _markup
cd _markup
echo.> render-codeblock-mermaid.html
```

**第四步**。复制以下内容到 `render-codeblock-mermaid.html` 文件中
```html
<pre class="mermaid">
  {{- .Inner | safeHTML }}
</pre>
{{ .Page.Store.Set "hasMermaid" true }}
```

**第五步**。打开 `assets\css\_custom.scss` 文件添加以下内容，使 `mermaid` 流程图能居中显示
```css
/* 流程图居中显示 */
.flowchart {
    display: block;
    margin-left: auto;
    margin-right: auto;
    max-width: 100%;
}
```

## 参考
* [Hugo支持渲染mermaid图](https://bytejog.com/posts/hugo/mermaid/)
