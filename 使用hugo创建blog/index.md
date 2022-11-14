# 创建Blog流程

# 使用hugo创建Blog

---

## 1.创建流程

---

### 1.1 下载 hugo

```
//使用homebrew下载hugo
brew install hugo
```

---

### 1.2 快速创建
```
hugo new site mysite
```

---

### 1.3 下载主题
```
//在刚刚创建的mysite/themes文件下打开终端
//LoveIt为当前作者选择的主题
git clone https://github.com/dillonzq/LoveIt.git LoveIt
```
---

### 1.4 阅读主题demo或者文档

---

### 1.5 快速创建写作文档
执行以下命令，在mysite/content下创建posts/my-first-post.md文档
```
hugo new posts/my-first-post.md
```
按照想要你想要编辑的方式，去改写my-first-post.md文档  
在启用drafts参数的条件下开启Hugo内置的服务器。
```
hugo server -D
```

---

### 1.6 上传github
在github创建后缀为github.io的仓库  
将hugo的baseUrl设置为GitHub仓库的地址  
并将主题设置为当前主题  
```
hugo --theme=LoveIt --baseUrl="http://zxlkgf.github.io/" --buildDrafts
```
再将public文件夹上传到github即可  

---

### 1.7 hugo基本使用
使用方法:  
  hugo  
  hugo [flags]  
  hugo [command]  
  hugo [command] [flags]  

例如 command:  
  new         为你的站点创建新的内容  
  server      一个高性能的web服务器  

节选的 flags:  
  -D, --buildDrafts                包括被标记为draft的文章  
  -E, --buildExpired               包括已过期的文章  
  -F, --buildFuture                包括将在未来发布的文章  

例子:  
  hugo -D                          生成静态文件并包括draft为true的文章  
  hugo new post/new-content.md     新建一篇文章  
  hugo new site mysite             新建一个称为mysite的站点  
  hugo server --buildExpired       启动服务器并包括已过期的文章  

---

