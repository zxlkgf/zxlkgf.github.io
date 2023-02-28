# GitAction_Hugo流程和问题


# 关于Hugo在GitAction部署期间的问题
参看网站:  
1.[利用GitHub Action实现Hugo博客在GitHub Pages自动部署](https://lucumt.info/post/hugo/using-github-action-to-auto-build-deploy/)(Access day:2023/02/28)

2.[Hugo + GitHub Action，搭建你的博客自动发布系统](https://sspai.com/post/73512)(Access day:2023/02/28)

## hugo使用请参照
创建Blog流程

## 问题1 hugo调用的主题是一个Git仓库
如果hugo调用的主题是一个Git仓库的，可以使用以下命令，将主题的Git仓库作为你的仓库的子目录，它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。
```git
git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt
```
默认情况下，子模块会将子项目放到一个与仓库同名的目录中，即“XXX”。 如果你想要放到其他地方，那么可以在命令结尾添加一个不同的路径。如果这时运行 git status，注意到新的.gitmodules文件。该配置文件保存了项目URL与已经拉取的本地目录之间的映射。
```git
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   .gitmodules
    new file:   XXX
```
.gitmodules的配置具体为下：
```git
[submodule "themes/LoveIt"]
	path = themes/LoveIt
	url = https://github.com/dillonzq/LoveIt.git
	branch = release
```

## 问题2 未知但解决的问题
错误为以下问题
```git
 /usr/bin/git push origin main
  fatal: unable to access 'https://x-access-token:***@github.com/zxlkgf/zxlkgf.github.io.git/': URL using bad/illegal format or missing URL
  Error: Action failed with "The process '/usr/bin/git' failed with exit code 128
```
原先在YML中使用的是PERSONAL_TOKEN,不知道为什么报错，原因目前未知。
查看以下网页，发现可以改名尝试。
[GitHub Actions: actions/checkout で、repository not found エラーが出るときの回避策](https://zenn.dev/m_norii/articles/349b9ce0260631)
遂改为ZXL_BLOG_TOKEN之后问题解决。

YML文件如下

```git
name: deploy

on:
    push:
    workflow_dispatch:
    schedule:
        # Runs everyday at 8:00 AM
        - cron: "0 0 * * *"

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
              uses: actions/checkout@v2
              with:
                  submodules: true
                  fetch-depth: 0

            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2
              with:
                  hugo-version: "latest"
                  extended: true

            - name: Build Web
              run: hugo

            - name: Deploy Web
              uses: peaceiris/actions-gh-pages@v3
              with:
                  PERSONAL_TOKEN: ${{ secrets.ZXL_BLOG_TOKEN }}
                  EXTERNAL_REPOSITORY: zxlkgf/zxlkgf.github.io
                  PUBLISH_BRANCH: main
                  PUBLISH_DIR: ./public
                  commit_message: ${{ github.event.head_commit.message }}
```
