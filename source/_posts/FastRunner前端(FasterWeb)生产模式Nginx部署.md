---
title: FastRunner前端(FasterWeb)生产模式Nginx部署
date: 2019-11-08 00:22:11
tags:
- FasterWeb
- Web前端
- Vue

categories:
- FasterWeb

---
# 1.安装nginx
```
yum install -y nginx
```
# 2.启动nginx
```
nginx 
```
# 3.增加fasterweb配置文件
```
# 进入FasterWeb项目根目录
cd /home/faster/FasterWeb/

# 把配置文件复制到nginx默认配置目录下
cp default.conf /etc/nginx/conf.d/fasterweb.conf
```
# 4.测试nginx配置文件
```
nginx -t
```

# 5.重新加载nginx配置文件
```
nginx -s reload
```
# 6.构建生产环境包 即dist目录
```
npm run build 
```
# 7.把dist目录复制到fasterweb.conf配置lacation root目录中  
```
cp -r dist/* /usr/share/nginx/html/
```
# 8.访问
```
http://sever_ip:8080/fastrunner/login
```