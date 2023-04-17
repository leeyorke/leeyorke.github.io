# 创建hugo项目
新建一个名为`myblog`的项目，并进入这个目录。
```bash
hugo new site myblog
cd myblog
```
hugo会为你新建一个基本的hugo项目，生成的目录结构如下：

```bash
./myblog
│  config.toml
│
├─archetypes
│      default.md
│
├─content
├─data
├─layouts
├─public
├─static
└─themes
```
# 下载主题
hugo有很多漂亮的主题，我们可以进入[https://themes.gohugo.io/](https://themes.gohugo.io/)选择喜欢的主题。这里我选择的是 [LoveIt](https://hugoloveit.com/zh-cn/theme-documentation-basics/)。
下载主题到 themes 目录下
```bash
 git clone https://github.com/dillonzq/LoveIt.git themes/LoveIt
```
初始化你的项目目录为 git 仓库, 并且把主题仓库作为你的网站目录的子模块:

```bash
git init
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```
拷贝`themes\LoveIt\exampleSite`目录下所有文件，将其复制到`myblog`下。
git add添加所有新建文件到本地仓库，提交commit
# 构建网站
命令行输入hugo构建
```bash
hugo
```
此时会在当前目录生成public、resources两个新文件夹。
# 启动hugo服务预览

```bash
hugo server
```
访问`http://localhost:1313/`，一个基本的博客就建好了。
# 修改config.toml
这是博客的配置文件，[点击查看如何配置](https://hugoloveit.com/zh-cn/theme-documentation-basics/#site-configuration)
# 推送到GitHub
* 在github新建一个仓库名为`用户名.github.io`；
* 修改myblog/public目录为docs。
* 推送myblog项目到仓库
* 打开仓库设置，开启pages,Branch设置为master/docs
* 访问`https://用户名.github.io`。
# 更新文章流程
创建文章
hugo new posts/[文章名]/index.zh-cn.md
```bash
hugo new posts/docker-command/index.zh-cn.md
```
渲染
```bash
hugo
```
本地预览
```bash
hugo server
```
添加、提交、推送
```bash
git add

git commit -m "commit_message"

git push
```
# ❗ FAQ
* themes 文件夹下的东西千万不要改，容易导致 GitHub action 构建失败
* 如何给博客配置评论系统？
  * 注册 Utterances 👉 [点击查看](https://roife.github.io/2021/02/12/use-utterances-for-comment/))
  * 修改 config.toml
  ```toml
  [params.page.comment.utterances]
        enable = true
        # owner/repo
        repo = "xxx/xxx.github.io"
        issueTerm = "pathname"
        label = "comment"
        lightTheme = "github-light"
        darkTheme = "github-dark"
  * ```