
####  Docsify
一个神奇的文档网站生成器。

#### 概述
`docsify` 可以快速帮你生成文档网站。不同于 `GitBook`、`Hexo` 的地方是它不会生成静态的 `.html` 文件，所有转换工作都是在运行时。如果你想要开始使用它，只需要创建一个 `index.html` 就可以开始编写文档并直接部署在 `GitHub Pages`。

查看快速开始了解详情:[https://docsify.js.org/#/zh-cn/](https://docsify.js.org/#/zh-cn/)

#### 安装 docsify-cli
建一个文件夹 `cloud-doc`
```
C:\Users\cloud-doc (master)
输入命令：npm i docsify-cli -g
> Thank you for using docsify!
If you rely on this package, please consider supporting our open collective:
> https://opencollective.com/docsify/donate

npm WARN ws@7.4.5 requires a peer of bufferutil@^4.0.1 but none is installed. You must install peer dependencies yourself.
npm WARN ws@7.4.5 requires a peer of utf-8-validate@^5.0.2 but none is installed. You must install peer dependencies yourself.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@2.3.2 (node_modules\docsify-cli\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@2.3.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ docsify-cli@4.4.3
added 208 packages from 91 contributors in 13.718s
```
#### 初始化项目
```
docsify init ./docs
```
#### 开始写文档
初始化成功后，可以看到 `./docs` 目录下创建的几个文件

- index.html 入口文件
- README.md 会做为主页内容渲染
- nojekyll 用于阻止 GitHub Pages 忽略掉下划线开头的文件
直接编辑 `docs/README.md` 就能更新文档内容，当然也可以添加更多页面。

#### 本地预览
通过运行 `docsify serve` 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 `http://localhost:3000` 。
#### 配置主题
参考 [https://docsify.js.org/#/zh-cn/themes](https://docsify.js.org/#/zh-cn/themes)

[docsify-themeable](https://jhildenbiddle.github.io/docsify-themeable/#/) 一个用于docsify的，简单到令人愉悦的主题系统，参考[https://codesandbox.io/s/xv36w4695o?file=/sidebar.md:0-167](https://codesandbox.io/s/xv36w4695o?file=/sidebar.md:0-167)
直接粘贴复制它的文件，然后用 `vscode` 打开，浏览器可以实时查看到效果
#### 部署 Github
接下来自己修修改改改，然后上传到 `github`，访问 [https://javakangpan.github.io/Blog/#/](https://javakangpan.github.io/Blog/#/)
这里简单的介绍了如何使用，了解更多需要自己去探索。