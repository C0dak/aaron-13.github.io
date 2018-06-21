# kube-shell安装

------

## kube-shell
kube-shell可以为kubectl提供自动命令提示和补全

有以下特征：
- 命令提示
- 自动补全
- 使用tab键列出可选对象
- vim模式

## 系统Python环境升级
系统自带的Python环境为2.7.5，使用pip安装kube-shell需要使用Python2.7.10+的版本。


```
下载最新的Python
wget https://www.python.org/ftp/python/2.7.14/Python-2.7.15.tgz
解压缩并编译
安装相关编译依赖的组件，模块
yum install gcc* openssl openssl-devel ncurses-devel.x86_64bzip2-devel sqlite-devel python-devel zlib

修改yum中的python版本
/usr/bin/yum
/usr/libexec/urlgrabber-ext-down
< #!/usr/bin/python
---
> #!/usr/bin/python2.7

rm -f /usr/bin/python
ln -sv /usr/local/python/bin/python /usr/bin/python

下载pip
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
ln -sv /usr/local/python/bin/pip /usr/bin/pip

pip install kube-shell

ln -sv /usr/local/python/bin/kube-shell /usr/bin/kube-shell
```


