## 问题

网上最流行的安装方法是：

```
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/dockercompose-$(
uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

安装过程中发现无法下载，可能是墙的原因。

另一种安装方案：

```
yum install epel-release 
yum install python-pip.noarch
pip install docker-compose
```

安装报错，原因是Centos8使用python3，应该用下面的命令：

```
yum install python3-pip.noarch
pip3 install docker-compose
```

安装还是报错:

error: command 'gcc' failed with exit status 1

继续解决：

```
yum install python-devel
```

搞定：

![image-20220518105351334](C:\Users\Brandon\AppData\Roaming\Typora\typora-user-images\image-20220518105351334.png)