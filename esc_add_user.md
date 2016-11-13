# 系统安装过程

双十一剁手，在阿里买了一台ecs服务器，系统 Ubuntu 14.04 64位,购买的时候默认root用户，创建了root密码。

进入新服务器，考虑安全性，准备完成以下服务器配置目标：

* 创建自己的用户
* 能使用sudo权限
* 禁用root ssh登录
* 使用rsa public key免密码登录


## 创建自己的用户

#### 1. 创建用户
* 命令

```
# useradd -d /home/wan -m wan
```

* 参数说明   

```
-d, --home HOME_DIR
	The new user will be created using HOME_DIR as the value for the user's login directory.
	
-m, --create-home
	Create the user's home directory if it does not exist.
```

创建完成后，查看/home下，ls -l查看了一下目录权限，所有者和所属组都是wan

```
# ls -al
total 12
drwxr-xr-x  3 root root 4096 Nov 13 13:41 .
drwxr-xr-x 22 root root 4096 Nov 11 01:41 ..
drwxr-xr-x  2 wan  wan  4096 Nov 13 13:41 wan
```

#### 2. 设置密码
* 命令

```
# passwd wan
Enter new UNIX password:
Retype new UNIX password:
```

打开一个新窗口，使用新用户登录，登录成功，可是界面有点问题

```
Welcome to aliyun Elastic Compute Service!

$
```

考虑应该是shell设置问题

#### 3. 修改默认shell

查看一下当前使用shell

```
$ echo $SHELL
/bin/sh
```

使用的sh，平时自己用的最多的是bash，考虑修改成bash。

* 命令

```
$ chsh -s /bin/bash
Password:
```

* 参数说明

```
chsh - change login shell

-s, --shell SHELL
	The name of the user's new login shell.
	
```

这里也可以使用root用户，修改/etc/passwd文件，在自己用户后面添加对应shell

```
# vi /etc/passwd

添加如下：
wan:x:1000:1000::/home/wan:/bin/bash
```

退出重新登录后

```
Welcome to aliyun Elastic Compute Service!

Last login: Sun Nov 13 14:06:46 2016
wan@hostname:~$ 
wan@hostname:~$ echo $SHELL
/bin/bash
```


## 使用sudo权限

通过sudo，我们能把某些超级权限有针对性的下放，并且不需要普通用户知道root密码,所以sudo相对于权限无限制性的su来说，还是比较安全的，所以sudo也能被称为受限制的su；另外sudo 是需要授权许可的，所以也被称为授权许可的su；

sudo 执行命令的流程是当前用户切换到root（或其它指定切换到的用户)，然后以root（或其它指定的切换到的用户）身份执行命令，执行完成后，直接退回到当前用户；而这些的前提是要通过sudo的配置文件/etc/sudoers来进行授权；


#### 方式1（添加到sudo组）

修改/etc/group文件，查找sudo开头的记录行，在该行后添加自己用户名，多个用户名之间用英文逗号隔开，如下：

```
# vi /etc/group

添加如下：
sudo:x:27:wan	
```

修改完成后重新登录wan用户

```
Last login: Sun Nov 13 14:18:24 2016
wan@hostname:~$ id
uid=1000(wan) gid=1000(wan) groups=1000(wan),27(sudo)
```

添加成功，如果想直接修改所属组，可以使用usermod命令，命令如下：

```
root@hostname:/# usermod -g sudo wan
```

usermod 与直接修改文件区别：usermod是修改用户所属组；直接修改文件，则是将用户添加到sudo组中

测试sudo命令

```
wan@hostname:~$ sudo su
[sudo] password for wan:
root@hostname:/home/wan#
```

但是中间需要输入自己的用户名，因为服务器是自己玩的，所以安全级别不要这么高，所以设置sudo不需要输入密码

设置sudo组不需要密码

修改/etc/sudoers，添加sudo不需要密码

```
# visudo

修改后效果如下
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL

ctrl+x退出
```

####方式2 添加用户到sudo配置文件中
修改/etc/sudoers，添加wan用户配置，并设置不需要使用密码

```
# visudo

在user privilege中添加如下
wan    ALL=(ALL:ALL) NOPASSWD: ALL
```


## 禁用root ssh登录
修改ssh配置文件 /etc/ssh/sshd_config,设置PermitRootLogin为no

```
# vi /etc/ssh/sshd_config

PermitRootLogin no
```

重启ssh服务

```
# service ssh restart
```

退出root用户，测试配置是否生效

```
ssh root@host
root@host's password:
Permission denied, please try again.
```

##使用rsa public key免密码登录

#### 1. 设置ssh支持RSA认证登录登录

```
# vi /etc/ssh/sshd_config
```

取消下面两行注释(前面的＃号）

```
RSAAuthentication yes
PubkeyAuthentication yes
```

重启ssh服务

```
# service ssh restart
```

#### 2. 设置自己的公钥
如果在自己电脑上有公钥的话，则直接上传就好啦，公钥位置 ~/.ssh/id_rsa.pub.如果没有，根据自己系统不同，网上有对应生成密钥方法，这里提供参考命令

```
ssh-keygen –t rsa –P ''
```

拷贝本地pub key到远端服务器（也可以手工打开文件，复制粘贴），服务器上无~/.ssh目录的话，可自己创建

```
scp ~/.ssh/id_rsa.pub wan@host:~/.ssh
```

将上传公钥添加到授权公钥列表中

```
$ cat cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

如果authorized_keys是首次生成，需要设置其权限为600

```
$ chmod 600 ~/.ssh/authorized_keys
```

测试是否可以免密码登录

```
ssh wan@host
Welcome to Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-86-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
New release '16.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Welcome to aliyun Elastic Compute Service!

Last login: Sun Nov 13 15:44:07 2016 
wan@hostname:~$
```

服务器账号设置完成，后续会自己安装php，nginx，redis，mysql等搭建一个简单的应用服务器
