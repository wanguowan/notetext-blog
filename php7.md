#小白安装PHP7

## 目录
[系统说明](#0)  
[1. 下载源码包](#1)   
[2. apt命令](#2)   
[3. PHP安装前系统包安装](#3)  
	[3.1 安装bison](#3.1)   
	[3.2 安装re2c](#3.2)   
[4. 扩展说明](#4)   
[5. 编译安装PHP7](#5)   
	[5.1 最终编译参数](#5.1)    
	[5.2 过程中扩展包依赖错误及对应安装](#5.2)    
	[5.3 编译](#5.3)   
	[5.4 配置](#5.4)  
	[5.5 验证](#5.5)     
[总结](#6)  

<h2 id='0'>系统说明</h2>

操作系统：Ubuntu 14.04.4 LTS (GNU/Linux 3.13.0-86-generic x86_64)  
PHP版本：PHP-7.0.13


<h2 id='1'> 下载源码包</h2>

因为小白安装，所以各种查询，最终在php.net找到[版本说明及下载地址][1]

选择中国的下载地址进行下载，在服务器上执行下面命令下载

```
$ wget http://cn2.php.net/get/php-7.0.13.tar.gz/from/this/mirror 
```

下载完成后重命名并解压

```
$ mv mirror php7.tar.gz
$ tar zxvf php7.tar.gz
```

因为自己没有编译安装过PHP，纯小白，所以流程参考网上，链接如下

[安装PHP7][2]  
[php.net UNIX系统下的安装][3]

但是对于各种参数，感觉完全按照流程安装下来，没有什么收获，所以对比网友安装过程及官网中标注的安装前的依赖包进行安装

<h2 id='2'> apt命令</h2>

后面安装系统库需要使用apt命令，基本使用方法如下，详情请参考[apt][4]

```
apt-get install	<libname>		//	安装包
apt update						//	更新源
apt-cache search <libname>		//	查找包
```


<h2 id='3'> PHP安装前系统包安装</h2>

bison是什么鬼，re2c是什么鬼，[php.net中][3]说明编译需要这两个包。上网查了一番，原来bison与re2c是php的语法分析器.[[详情]][5]   [[bison]][6]

知道bison和re2c是做什么用的之后，开始安装

<h3 id='3.1'> 安装bison</h3>

命令: 
  
```
wan@hostname:~/php-7.0.13$ apt-cache search bison

wan@hostname:~/php-7.0.13$ sudo apt-get install bison
```

安装完后立马查看bison版本，查看是否安装成功

```
wan@hostname:~/php-7.0.13$ bison --help

wan@hostname:~/php-7.0.13$ bison -V 
bison (GNU Bison) 3.0.2
Written by Robert Corbett and Richard Stallman.

Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

<h3 id='3.2'>安装re2c</h3>

命令：

```
wan@hostname:~/php-7.0.13$ apt-cache search re2c

wan@hostname:~/php-7.0.13$ sudo apt-get install re2c
```

根据参考中网友安装过程，不安装gcc，安装re2c会报错，说好的报错呢，难道系统已经有gcc了？果断命令：

```
wan@onlywan:~/php-7.0.13$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/4.8/lto-wrapper
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 4.8.4-2ubuntu1~14.04.1' --with-bugurl=file:///usr/share/doc/gcc-4.8/README.Bugs --enable-languages=c,c++,java,go,d,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-4.8 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.8 --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-gnu-unique-object --disable-libmudflap --enable-plugin --with-system-zlib --disable-browser-plugin --enable-java-awt=gtk --enable-gtk-cairo --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-4.8-amd64/jre --enable-java-home --with-jvm-root-dir=/usr/lib/jvm/java-1.5.0-gcj-4.8-amd64 --with-jvm-jar-dir=/usr/lib/jvm-exports/java-1.5.0-gcj-4.8-amd64 --with-arch-directory=amd64 --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --enable-objc-gc --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.1)
```
这下就放心了。   
下一步，研究扩展，其他php.net中说的环境如果有报错，再安装也不迟


<h2 id='4'> 扩展说明</h2>

前一段时间公司测试服务器搭建一个工程，使用laravel框架，composer install的时候，报出一个缺少依赖包的错误，细节忘了，大概记得缺少fileinfo依赖。小白我第一次安装扩展呀，摆好姿势，各种兴奋，可以搞点以前没搞过的了。各种查询，结果安装fileinfo。然后各种被虐：pecl安装扩展有warning，结果make失败；然后去pecl官网，下载源码，一路下来，make还是报错。然后怀疑是pecl warning的问题，然后各种查询，解决了warning的问题，重新来，make还是报错。查询过程中，看到有人说在php源码中编译安装扩展。最后发现root用户目录下有源码，然后进入到源码中，抱着试一试的心态，竟然成功了。这么简单的事情，竟然搞了快一下午。

有了上面的经历，决定一次性准备好所有可能需要的扩展

开始研究扩展（所谓研究就是对着参考的大神的列表，一个一个研究是干啥用的）

####--with-config-file-path=PATH   
Set the path in which to look for php.ini [PREFIX/lib]

####--with-mcrypt  
加密支持扩展库[详情][7]  

#### mysql数据库连接扩展
--with-mysql=mysqlnd  
--with-mysqli=mysqlnd  
--with-pdo-mysql=mysqlnd   

```
wan@hostname:~/php-7.0.13$ ./configure --help | grep mysql   
  --with-mysqli=FILE      Include MySQLi support.  FILE is the path
                          to mysql_config.  If no value or mysqlnd is passed   
  --enable-embedded-mysqli   
  --with-mysql-sock=SOCKPATH   
  --with-pdo-mysql=DIR    PDO: MySQL support. DIR is the MySQL base directory
                          If no value or mysqlnd is passed as DIR, the
  --enable-mysqlnd        Enable mysqlnd explicitly, will be done implicitly
  --disable-mysqlnd-compression-support
                          Disable support for the MySQL compressed protocol in mysqlnd
  --with-zlib-dir=DIR     mysqlnd: Set the path to libz install prefix
  
```
  
其中--with-mysql在PHP7中已经废弃。
而两外两个配置中使用的mysqlnd，
  
查询mysqlnd是什么，原来是zend自己开发的mysql驱动，来避免可能存在的版权问题，而且貌似还更快 [详情][8]

果断使用mysqlnd

####--with-gd   
php处理图形的扩展库[详情][9]


####--with-iconv
文件编码转化库[详情][10]

####--with-zlib
数据压缩用的函式库[详情][11]

####--enable-bcmath
科学计算包

####--enable-shmop
PHP读写创建删除UNIX共享内存段[详情][12]

####--enable-sysvsem
PHP进程信号控制[详情][13]

####--enable-inline-optimization
无该配置。默认开启；对应有关闭配置 --disable-inline-optimization

####--enable-mbregex
多字节正则功能  multibyte regex support   
无该配置。默认开启；对应有关闭配置 --disable-mbregex

####--enable-mbstring       
Enable multibyte string support   
在计算中文字符串长度的时候，使用过～

####--enable-gd-native-ttf  
GD: Enable TrueType string function  
TrueType一种字体，中文名称全真字体 [详情][14]

####--enable-pcntl
PHP进程控制[详情][15]

####--enable-sockets        
Enable sockets support

####--enable-xml   
无该配置。默认开启该功能 --disable-xml 
XML support

####--with-xmlrpc=DIR    
Include XMLRPC-EPI support

####--enable-zip    
Include Zip read/write support

####--enable-soap   
Enable SOAP support
简单对象访问协议［Simple Object Access Protocol］[详情][17]


####--without-pear          
Do not install PEAR   
貌似一个管理扩展工具，不是很清楚，感觉没啥用[详情][18]


####--with-gettext=DIR    
Include GNU gettext support   
GNU Gettext 开源多语组件[详情][16]


####--enable-session
无该配置；默认支持   --disable-session       Disable session support

####--with-curl=DIR         
Include cURL support   
利用URL语法在命令行方式下工作的开源文件传输工具[详情][19]


####--with-jpeg-dir=DIR     
GD: Set the path to libjpeg install prefix

####--with-freetype-dir
字体引擎[详情][20]

####--enable-opcache
无此配置，php opcache默认打开 关闭参数--disable-opcache       Disable Zend OPcache support
[opcache详情][21]


<h2 id='5'> 编译安装PHP7</h2>

<h3 id='5.1'>最终编译参数</h3>

```
wan@hostname:~/php-7.0.13$ ./configure --prefix=/usr/local/php7 \
--with-config-file-path=/usr/local/php7/etc \
--with-mcrypt=/usr/include \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-gd \
--with-iconv \
--with-zlib \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-mbregex \
--enable-fpm \
--enable-mbstring \
--enable-gd-native-ttf \
--with-openssl \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--without-pear \
--with-gettext \
--enable-session \
--with-curl \
--with-jpeg-dir \
--with-freetype-dir
```

<h3 id='5.2'>过程中扩展包依赖错误及对应安装</h3>

#### 错误：
configure: error: xml2-config not found. Please check your libxml2 installation.

解决方法：

```
wan@hostname:~/php-7.0.13$ apt-cache search libxml2
wan@hostname:~/php-7.0.13$ sudo apt-get install libxml++2.6-dev
```

#### 错误：
configure: error: Cannot find OpenSSL's <evp.h>

解决方法：

```
wan@hostname:~/php-7.0.13$ apt-cache search openssl
wan@hostname:~/php-7.0.13$ sudo apt-get install openssl
wan@hostname:~/php-7.0.13$ sudo apt-get install pkg-config
wan@hostname:~/php-7.0.13$ sudo apt-get install libssl-dev
```

#### 错误：
configure: error: Please reinstall the libcurl distribution -
    easy.h should be in <curl-dir>/include/curl/
    
解决方法：

```
wan@hostname:~/php-7.0.13$ sudo apt-get install curl
```

#### 错误
configure: error: Please reinstall the libcurl distribution -
    easy.h should be in <curl-dir>/include/curl/
    
解决方法：

```
wan@hostname:~/php-7.0.13$ sudo apt-get install libcurl4-gnutls-dev
```

#### 错误
configure: error: jpeglib.h not found.

解决方法：

```
wan@hostname:~/php-7.0.13$ sudo apt-get install libjpeg-dev
```

#### 错误
configure: error: png.h not found.

解决方法：

```
wan@hostname:~/php-7.0.13$ sudo apt-get install libpng++-dev
```

#### 错误
configure: error: freetype-config not found.

解决方法：

```
wan@hostname:~/php-7.0.13$ sudo apt-get install libfreetype6-dev
```

#### 错误
configure: error: mcrypt.h not found. Please reinstall libmcrypt.

解决方法：

```
wan@hostname:~/php-7.0.13$ sudo apt-get install libmcrypt-dev
```


经过半天折腾终于看到了   
Thank you for using PHP.

<h3 id='5.3'>编译</h3>

```
wan@hostname:~/php-7.0.13$ make && sudo make install
```

<h3 id='5.4'>配置</h3>

```
wan@hostname:~/php-7.0.13$ cp php.ini-production /usr/local/php7/etc/php.ini  
wan@hostname:~/php-7.0.13$ cp sapi/fpm/init.d.php-fpm /etc/init.d/php7-fpm  
wan@hostname:~/php-7.0.13$ chmod +x /etc/init.d/php7-fpm  
wan@hostname:~/php-7.0.13$ cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf  
wan@hostname:~/php-7.0.13$ cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf
wan@hostname:~/php-7.0.13$ cd /usr/local/bin
wan@hostname:~/php-7.0.13$ sudo ln -s ../php7/bin/php
```
<h3 id='5.5'>验证</h3>

```
wan@hostman:~$ php -v
PHP 7.0.13 (cli) (built: Nov 19 2016 17:22:48) ( NTS )
Copyright (c) 1997-2016 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
```


<h2 id='6'> 总结</h2>
因是第一次安装，所以各种问题都是解决后记录，可能有些细节忘记记录。也可能有些解决方法不正确等等，还望大家指正，多多交流。

整个过程花费很长时间，包括整理这片文章，感谢那些在网上分享自己经验的人

后续会继续研究配置nginx，fpm，mysql等。如果感兴趣，可以继续关注后面的记录


------------
[1]: http://www.php.net/downloads.php
[2]: http://blog.csdn.net/molaifeng/article/details/50178367
[3]: http://www.php.net/install.unix
[4]: http://blog.csdn.net/heihuifeng/article/details/6610899
[5]: http://blog.csdn.net/siren0203/article/details/8085813
[6]: http://baike.baidu.com/link?url=O9SlNKRxIplDrCCVlTbbQmUExaxa_45vvCCC6abnL6nI7IpBO2k7dOO3zFkv05FAoQe422-fY1PY_tyufPath4ww1UnGa8Z_iCwh9bCBZh_
[7]: http://baike.baidu.com/link?url=tVQjwb0RMlS4T1RqgWdQ8h67pHAVxaGE7K9TXhxby_-0YEzQvaARIy0cHsdeelQljjlQf7xmP_EHU8ADLndUTAMqYXTTqN22J4Prat3NcaO
[8]: http://baike.baidu.com/link?url=KBuL61fLXI7Vl0uR3rukFNsp6vLflWzcS9tzvI5ZeQU8BefFvaDdEKpbeM_N96JdBpidhAOTLQXWxnrbmcLMpa
[9]: http://baike.baidu.com/link?url=9xmfpht-0HW73X0E5Vi_9CpxPMZZGLV3UnA6padZzNCcFuitI39xFuRNW0buQ1kX_dPb4Esr4XTiK60qGXlbTjFaTuoBMtj3EbPgML-Utr\_
[10]: http://baike.baidu.com/link?url=KEJAkPankryNZB8Ox7XLU_NnDcQ9cZBjadlrnsQ7fTDOS_N8bvsNGBadnuFHX_RTT60HuOOAzvqMR8ZgDWXiL\_
[11]: http://baike.baidu.com/link?url=knT0hlIc8TDi1Zk6z9LM47AkzOnKHjd-LtPmUI-sVrK_SR3-fMHGssbyZSkWHN-8Saa62n_rLsJIuWYSOPnINK
[12]: http://baike.baidu.com/link?url=pFYdd_Eg07go1Tl_BXTWHE9MGyET5Z_NTKgZBFIOFiSWXBAnYwRGqfLrWSuXUKd_IsyzrWUpMnzvweR21aLHCK
[13]: https://segmentfault.com/a/1190000005042263
[14]: http://baike.baidu.com/link?url=LwUsb0A4KfU-yf6fARhq3VhQBwsOHBrYZEIX99ADKvDfFMsjbWLMWhpHKXnLXyE_ehnl4utEeRj4XWiHc5iIxILyQq7L7VlczqoHsXlRdnTu1fJF8qf7baOseYYgL1lf
[15]: http://php.net/manual/zh/intro.pcntl.php
[16]: http://blog.csdn.net/gykthh/article/details/43192711
[17]: http://baike.baidu.com/link?url=r0l_5L_SboStcLYANYD3vdvxSPMXxlCWzSXpZpKc3zQseQ6URnxXIs3WBq1fnHpyOtrmyABSiI2nuFknqrH58WjfOKYj_lcRCcvdK6zjpcoUS3-TmDDUAMJNuiOiWvryUtIA7F3jg0TVNEpM-yOhnSEmOyzyREm7vPTdeX0ziFBcM0qyOsydsoEQuZ_wVa3GTF26UUTtJrJS5BZNJGIk-C4iQZC-mDMUIrK6xZ6KUCq
[18]: https://www.zhihu.com/question/50826568?sort=created
[19]: http://baike.baidu.com/link?url=F1Gepdiknu2usTAPHc1lLyEvIdy9kVWUyuyh_O6s8f1dN3XIXVQ65Jvjn2-iuXsGzg-cojeT1RgT9pMtZ4ErXq
[20]: http://baike.baidu.com/link?url=GHbteHw2HziQ-3oLLQvf9JrJgAFAKNkGKeohiFxkSqr6xf_8DbPJHO518OmxOxwFyH2xqWChqgqBIEnfsLq5ztcFyFnTWtdmQf5LYEN34Vm
[21]: http://baike.baidu.com/link?url=OD2bgMysKeC5XPJgs4zQjUtJeKnfmakBFOgjyyKQ1KVNzl8e7TGENEs0VGfjhOtQd89h44vZ7OVNAb2O8Xp-aGNNKJGUxJjTgNsEWCfJ9dO