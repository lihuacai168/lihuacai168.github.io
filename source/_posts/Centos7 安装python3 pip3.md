---
layout: centos7
title: Centos7 安装python3 pip3
date: 2019-11-07 10:13:24
tags:
- Python
- Centos7
- Pip
categories:
- Python
---

# 安装python3.6
- 安装 python36
```python
yum install python36 -y
```
- 查看版本
安装完成,查看一下python版本是否正常
```python
python36 --version
```
- 创建软链
```python
ln -s /usr/bin/python3.6 /usr/bin/py3
ln -s /usr/bin/python3.6 /usr/bin/python3
```
# 安装pip3
- 安装依赖
```bash
yum install openssl-devel -y
yum install zlib-devel -y
```
- 安装 setuptools
```
wget --no-check-certificate https://pypi.python.org/packages/source/s/setuptools/setuptools-19.6.tar.gz#md5=c607dd118eae682c44ed146367a17e26
tar -zxvf setuptools-19.6.tar.gz
cd setuptools-19.6
python36 setup.py build
python36 setup.py install
```
- 安装 pip3
```
wget https://github.com/pypa/pip/archive/9.0.1.tar.gz
tar -zvxf 9.0.1.tar.gz
cd pip-9.0.1
python36 setup.py install
```
- 查看 pip3版本
```
pip --version
```
- 升级pip3
```
pip3 install --upgrade pip
```