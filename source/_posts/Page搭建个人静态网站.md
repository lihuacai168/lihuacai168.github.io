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

## Hexo安装,调试,部署
```
npm install -g hexo
```

### Hexo本地调试
```
mkdir myblog # 创建博客项目文件夹
hexo init # 初始化项目
hexo g # 根据当前目录下文件生成静态网页
hexo s # 启动服务器
``` 
### Hexo部署到GitHub