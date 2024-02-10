---

author = "yifanLiao"
title = "Rich Content"
date = "2024-02-10"
description = "A brief description of use method"
tags = [
    "markdown",
    "web",
]

---

# Blog部署及修改方法

## Part1:网站选择及部署

1. 通过[Hugo选择适合自己风格的网站](https://themes.gohugo.io/)
2. 进入github项目后点击fork加入自己项目列表中
3. 进入[Netfily](app.netlify.com),选择github账号登录，选择你的网页项目后进行构建

## Part2:部署后搭建自己的网站

URL:https://windliao.netlify.app/

或者点击*[Liao's Blog](https://windliao.netlify.app/)*

### 方法：拉取项目到本地进行更改

```
git clone [url] #克隆到本地

git add *
git commit -m "change"
git push [url] master #git自动更新最新版本项目文件
#之后网页自动渲染
```

### 头像修改

path：*hugo-theme-stack/assets/img*

### 个人简介/侧边栏修改

path1:*exampleSite/config.yaml*

path2：*exampleSite/content/page/about/index.md*

> 可能出现部署失败情况，一般是构建报错

### 发布文章及修改

* path:*exampleSite/content/post*

* 使用*index.md*渲染网页，图片放在同一文件，引用使用相对路径

* [Markdown语法大全](https://markdown.com.cn/basic-syntax)

​	