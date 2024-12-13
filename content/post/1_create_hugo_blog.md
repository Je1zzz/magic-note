---
title: "Hugo + Github pages制作个人学习博客"    # 标题，去掉横短线病转换为标题格式
date: 2024-12-13T21:39:54+08:00                                               # 发布日期
tags: ["😁无用小笔记"]                                                      # 分类和标记，用于过滤
author: "zhm"                                                  # 作者
# author: ["Me", "You"] # multiple authors
showToc: true                                                   # 显示目录
TocOpen: false                                                  # 默认展开
draft: false                                                    # 是否为草稿（True则会发布）
hidemeta: false                                                 # 隐藏元信息（作者、发布日期等）
comments: false                                                 # 是否comments
description: ""                                                 # 文章描述
canonicalURL: "https://canonical.url/to/page"                   # idk
disableShare: false                                             # 禁止分享
disableHLJS: false                                              # 禁用代码高亮
hideSummary: false                                              # 隐藏文章摘要
searchHidden: false                                             # 在search里隐藏文章
ShowReadingTime: true                                           # 显示阅读时间
ShowBreadCrumbs: true                                           # 显示面包屑导航
ShowPostNavLinks: true                                          # 显示文章导航（下一篇，上一篇）
ShowWordCount: true                                             # 字数统计
ShowRssButtonInSectionTermList: true                            # idk
UseHugoToc: true                                                  # 使用Hugo生成的目录
math: true    
editPost:
    URL: "https://github.com/Je1zzz/magic-note/issues"
    Text: "Suggest Changes" # edit text
    appendFilePath: false # to append file path to Edit link  
---

### Todo:
- [ ] 图像不能显示

## 1. 下载Hugo

进入网址 [Releases · gohugoio/hugo](https://github.com/gohugoio/hugo/releases)，下载相关文件

![](/1_make_blog/image1.png)

安装完成后重启终端，输入 ``hugo version`` 可以查看下载的Hugo版本信息 

## 2. 初始化Hugo博客

在存储笔记文件的根目录下，输入

```python
hugo new site <your_blog_name> --format yml
cd <your_blog_name>
git init
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

随后将目录下的 <hugo.yml> 输入 ``theme: PaperMod``

注意：

- ``--format``不能输入``--f``

- 更多主题自行探索 [Complete List | Hugo Themes](https://themes.gohugo.io/) 

## 3. Github配置

### 3.1 新建仓库并关联

先新建一个repository，选择``Public``公开

然后将项目关联到远程，以下为web关联，也可以选择更稳定的ssh：

```python
git remote add origin https://github.com/username/<your_blog_name>.git
git branch -M main
```

### 3.2 创建文章并测试

```python
hugo new post/test.md
hugo server -D
```

![](/1_make_blog/2024-12-13-20-45-10-image.png)

![](/1_make_blog/2024-12-13-20-45-30-image.png)

### 3.3 新建分支

在Github项目仓中，创建新的分支

![](/1_make_blog/2024-12-13-20-46-49-image.png)

## 4. 部署Github Action

### 4.1 填写deploy.yml文件

在项目的< hugo.yml >中，将baseURL设置为 

```
baseURL: https://<username>.github.io/<your_blog_name>/
```

注意： 最后的斜杠也不能省去

并在目录下创建 .github/workflows/deploy.yml

![](/1_make_blog/2024-12-13-20-51-04-image.png)

填入如下：

```yml
name: deploy

on:
    push:
      branches: ["main"]
    workflow_dispatch:
    # schedule:
    #     # Runs everyday at 8:00 AM
    #     - cron: "0 0 * * *"

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

# Default to bash
defaults:
  run:
    shell: bash

jobs:
    # BUild job
    build:
        runs-on: ubuntu-latest
        env:
          HUGO_VERSION: latest
          TZ: America/Los_Angeles
        steps:
            - name: Checkout
              uses: actions/checkout@v3
              with:
                  submodules: true  # Fetch Hugo themes (true OR recursive)
                  fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

            - name: Setup Hugo
              id: pages
              uses: peaceiris/actions-hugo@v2
              with:
                hugo-version: latest
                extended: true

            - name: Build Hugo
              env:
                # For maximum backward compatibility with Hugo modules
                HUGO_ENVIRONMENT: production
                HUGO_ENV: production
              run: hugo --minify

            - name: Deploy Web
              id: deployment
              uses: peaceiris/actions-gh-pages@v3
              with:
                  PERSONAL_TOKEN: ${{ secrets.PERSONAL_TOKEN }}
                  EXTERNAL_REPOSITORY: Je1zzz/magic-note
                  PUBLISH_BRANCH: gh-pages
                  PUBLISH_DIR: ./public
                  commit_message: ${{ github.event.head_commit.message }}
```

注意：需要更改的是最后的 < EXTERNAL_REPOSITORY >。

### 4.2 设置token

打开网址[Personal Access Tokens (Classic)](https://github.com/settings/tokens)，生成token后复制

![](/1_make_blog/2024-12-13-20-57-27-image.png)

然后在github的项目设置中写入secret

![](/1_make_blog/2024-12-13-20-59-53-image.png)

## 5. 测试

先设置当前分支 `main` 跟踪远程的 `origin/main` 分支

```
git branch --set-upstream-to=origin/main main
```

先合并不相关目录

```
git pull origin main --allow-unrelated-histories
```

上传

```
git add .
git commit -m "first commit"
git push
```

可以在Actions看见已经成功了

![](/1_make_blog/2024-12-13-21-09-24-image.png)

打开网址

![](/1_make_blog/2024-12-13-21-10-12-image.png)

## 6. 上传新笔记

在archetypes中创建一个<post.md>，复制以下（模板）：

```
---
title: "{{ .File.ContentBaseName | title }}"    # 标题，去掉横短线病转换为标题格式
date: {{ .Date }}                                               # 发布日期
tags: []                                                      # 分类和标记，用于过滤
author: "zhm"                                                  # 作者
# author: ["Me", "You"] # multiple authors
showToc: true                                                   # 显示目录
TocOpen: false                                                  # 默认展开
draft: false                                                    # 是否为草稿（True则会发布）
hidemeta: false                                                 # 隐藏元信息（作者、发布日期等）
comments: false                                                 # 是否comments
description: ""                                                 # 文章描述
canonicalURL: "https://canonical.url/to/page"                   # idk
disableShare: false                                             # 禁止分享
disableHLJS: false                                              # 禁用代码高亮
hideSummary: false                                              # 隐藏文章摘要
searchHidden: false                                             # 在search里隐藏文章
ShowReadingTime: true                                           # 显示阅读时间
ShowBreadCrumbs: true                                           # 显示面包屑导航
ShowPostNavLinks: true                                          # 显示文章导航（下一篇，上一篇）
ShowWordCount: true                                             # 字数统计
ShowRssButtonInSectionTermList: true                            # idk
UseHugoToc: true                                                  # 使用Hugo生成的目录
math: true       
---
```

然后使用``hugo new post/xx.md`` 生成一篇文章，复制里面的头信息

将笔记已以下形式存储好，一个md文件以及一个专门存储该md文件的图像的文件夹

![](picture_saved/2024-12-13-21-13-43-image.png)

然后将md文件拖入到post里，再将img文件拖入到static里。复制头信息到最前面， 并更改文件中图像文件的格式，可以按`CTRL+F2`一起修改路径，修改为``/folder_name/xx.png``

![](picture_saved/2024-12-13-21-34-46-image.png)



随后进行预览和上传

```
# 预览
hugo server -D


# 上传
git add .
git commit -m "update passage XX"
git push
```

## 参考

[1] [Hugo如何在markdown里引用本地图片 - Jincheng9's blog](https://jincheng9.github.io/post/hugo-add-img/)

[2] [hugo+Stack 搭建个人博客](https://hyrtee.github.io/2023/start-blog/#%E5%AE%89%E8%A3%85-hugo)
