# 拉取代码操作
```
git clone git@github.com:lihuacai168/lihuacai168.github.io.git
git submodule init && git submodule update

#下面这一句的效果和上面三条命令的效果是一样的,多加了个参数  `--recursive`
git clone git@github.com:lihuacai168/lihuacai168.github.io.git --recursive
```

# 增加子模块的方式
```
# 指定路径为themes/next,子仓库的名字就是next,不然会是远程仓库的名字hexo-theme-next
git submodule add git@github.com:lihuacai168/hexo-theme-next.git themes/next

```
