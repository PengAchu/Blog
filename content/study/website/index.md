---
title: 个人网站搭建
date: 2026-04-29T00:00:00+08:00
draft: false
tags: ["study","website","Hugo","Blowfish"]
---

## 前置准备

安装Hugo、Git、Go（非必要，使用blowfish自带的CLI才需要），准备 github 账号，建一个 repository用于关联本地仓库



## 搭建过程

### 搭建本地网站

```powershell
// 用 hugo 指令建立网站文件夹mysite
hugo new site mysite

// 下载 blowfish 主题
cd themes
git clone https://github.com/nunocoracao/blowfish.git
cd ..

// 删除 .git 文件，本次搭建未采用 submodule 的方式，因为对 blowfish 的 layout 做了修改，后续上传github repository时需要一起上传 themes 的 blowfish 文件夹，而 submodule 方式则会在 repository 关联引用未修改的https://github.com/nunocoracao/blowfish.git中的 blowfish，这会导致对 blowfish 的修改失效
Remove-Item -Recurse -Force  themes/blowfish/.git

// 初始化网站，可查看效果
hugo server
```


### 构建workflow

```powershell
// 建workflow
mkdir .github/workflows
// 这里用的 VScode 编辑文件，hugo.yaml的内容可参考https://blowfish.page/zh-cn/docs/hosting-deployment/
code .github/workflows/hugo.yaml
```
```powershell
# .github/workflows/hugo.yaml

name: GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  build-deploy:
    runs-on: ubuntu-24.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: gh-pages
          publish_dir: ./public
```


### 上传到github

```powershell
// 上传项目到github
git init
git branch -M main
git config --global user.name yourname
git config --global user.email youremail
git add .
git commit -m "modification message"
// 这一步要输入账密，github密码如果不行可以创一个通行密钥，用通行密钥登录
git push -u origin main
```



### 部署个人主页

上传后在action部分会有workflow运作，首先产生gh-page branch，然后在setting->page部分设置branch为gh-page，回到action会看到workflow继续运作部署个人主页，等一会完成后即可进入个人主页。

若产生gh-page branch可能是权限问题（本人踩的坑），setting->action->general->workflow permissions 中选择 Read and Write Permission。



## 关于blowfish主题的修改

由于不满意于blowfish的极少数页面布局点，故对layout部分做了一些修改，相关文件均在blowfish/partials/下



### homepage的背景填充

```powershell
// blowfish/partials/home/background.html的img设置代码
// 对class的h和w做了修改，均设为为full，以全填充图片，本人背景图比例正好适合所以这样设置
<img
	id="background-image"
    class="nozoom mt-0 mr-0 mb-0 ml-0 h-full w-full object-cover"
    src="{{ $homepageImage.RelPermalink }}"
    role="presentation"
    {{ if $style }}style="{{ $style | safeCSS }}"{{ end }}>
```



### 导航栏长度与logo调整

```powershell
// blowfish/layouts/partials/header/basic.html
// logo的大小形状调整如下
<img
  src="{{ $logo.RelPermalink }}"
  class="logo h-12 w-12 object-cover object-center nozoom rounded-full"
  alt="">

// blowfish/layouts/partials/header/components/desktop-menu.html
// 导航栏长度调整如下，修改第一行h为16即可
<nav class="flex items-center gap-x-5 h-16">

// blowfish/layouts/partials/header/components/desktop-menu.html
// 嵌套菜单中子项居中，修改pt和p值一致
<div class="menuhide">
	<div class="pt-5 p-5 rounded-xl backdrop-blur shadow-2xl bg-neutral/25 dark:bg-neutral-800/25">
```



