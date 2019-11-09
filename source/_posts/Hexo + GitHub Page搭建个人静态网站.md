---
layout: hexo
title: Hexo + GitHub Page搭建个人静态网站
date: 2019-11-06 17:38:09
tags:
- 博客
- Hexo
categories:
- web前端
---
## Github设置
### 创建仓库名称是username.github.io
```
用户名是lihuacai168
那么仓库名对应就是lihuacai168.github.io
```
### Github部署测试
- 进入项目的`settings`找到`GitHub Pages`,会看到
```
# 项目还在准备状态,直接打开链接是显示404
Your site is ready at https://lihuacai168.github.io/
```
- 回到项目`Code`页面,增加`index.md`文件,去到`settings`的`GitHub Pages`,会看到
```
# 项目已经发布了,打开链接就能正常查看
Your site is published at https://lihuacai168.github.io/
```

## `Hexo`安装,调试,部署
```
npm install -g hexo
```

### `Hexo`本地调试
```
mkdir myblog # 创建博客项目文件夹
hexo init # 初始化项目
hexo g # 根据当前目录下文件生成静态网页
hexo s # 启动服务器
```

### `Hexo`部署到`GitHub`
```
hexo clean  # 清理缓存

hexo g # 根据当前目录下文件生成静态网页

hexo d  # 推送本地编译好的代码代GitHub仓库的master分支
```

## 使用`Git`仓库管理`Hexo`项目
### 初始化仓库
```
cd myblog # 进入项目文件夹

git init # 初始化git仓库
```
### 提交到本地仓库`dev`分支
`GitHub Page`默认使用`master`分支的代码来部署.
所以需要用`dev`分支保存平时写文章的代码
```
git branch -b dev  #　创建dev分支,并且换到dev分支  
git add .
git commit -m '提交所有代码'
```

### 配置远程仓库地址
```
# 增加远程仓库地址,别名是origin
git remote add origin git@github.com:lihuacai168/lihuacai168.github.io.git  
```

### 推送代码到远程dev分支
```
git branch --set-upstream-to=origin/dev dev  # 本地dev分支追踪远程分支,只需第一次提交时操作
git pull origin dev  # 推送之前先拉取远程代码 
git push origin dev  # 本地dev分支推送到远程dev分支
```

## 安装主题
主题的路径统一放在`myblog\themes`目录下  
因为安装主题需要修改里面的配置文件,所以最好先在`GitHub`上`fork`一份这个主题的代码,不然无法提交主题的代码  
下面以`next`主题为例
### `fork`主题代码 
在`GitHub`上`fork`主题`next`的仓库  
![fork](fork-next.jpg)

### 拉取远程`fork`的仓库,加入到本地仓库的子模块
在`myblog`根目录执行
```
git submodule add git@github.com:lihuacai168/hexo-theme-next.git themes/next
```
- `git submodule add`是git仓库增加子模块
- `git@github.com:lihuacai168/hexo-theme-next.git`是`fork`主题的远程git地址
- `themes/next` 指定子模块在本地的目录名称