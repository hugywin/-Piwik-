![Piwik](http://wemedia.yidian-inc.com:9000/piwik/plugins/Morpheus/images/logo.svg)
# Linux 环境 安装Piwik
1. 安装Apache
2. 安装PHP7
3. 安装Piwik

######（ps: 由于木有root权限....前方一大波坑等着我）
#### 登录服务器
`ssh user@127.0.0.1`

os: CentOS Linux-7.2.1511-Core-x86_64-3.10.0-327.3.1.el7.x86_64

服务器没有外网，选择scp拷贝上传

`scp xxx.tar.gz user@127.0.0.1:~/file`

（压缩解压：）`tar cvf a.tar a` `tar xvf a.tar`

***
## 安装Apach
(老大反复强调木有root权限， 我完全没有在意)

根据依赖安装

1. apr-1.5.2 [http://ftp.jaist.ac.jp/pub/apache//apr/apr-1.5.2.tar.gz](http://ftp.jaist.ac.jp/pub/apache//apr/apr-1.5.2.tar.gz)
2. apr-util-1.5.4 [http://ftp.jaist.ac.jp/pub/apache//apr/apr-util-1.5.4.tar.bz2](http://ftp.jaist.ac.jp/pub/apache//apr/apr-util-1.5.4.tar.bz2)
3. pcre [http://jaist.dl.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz](http://jaist.dl.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz)

安装顺序apr -> apr-util ，当然pcre可以乱入（顺序不重要）；

（源码安装过程，配置 -> 编译 -> 安装 三部曲；）

主要说配置：

进入解压后的apr文件包：

    [worker@worker]# `./configure --prefix=/home/lib/apr/`(这里配置的是安装路径)
    # `make`
    # `make intall`

进入解压后的apr-util文件包：

     [worker@worker]# `./configure --prefix=/home/lib/apr-util -with-apr=/home/lib/apr/bin/apr-1-config`（配置apr-util安装路径，并关联apr文件——确保路径正确）
    # `make`
    # `make intall`

进入解压后的pcre文件包：

    [worker@worker]# `./configure --prefix=/home/lib/pcre`（配置安装路径）
    # `make`
    # `make intall`


依赖安装完成， 安装httpd

进入解压后的httpd文件包:

	[worker@worker]# `./configure --prefix=/home/lib/apache/ \
	--sysconfdir=/home/lib/httpd \ //指定Apache服务器的配置文件存放位置
	--with-apr=/home/lib/apr \
	--with-apr-util=/home/lib/apr-util/ \
	--with-pcre=/home/lib/pcre/  \
	--enable-so \ //以动态共享对象(DSO)编译---记得要加，否则以后手动修改配置文件加载新的模块，比如，不配置的话，安装好PHP后，要手动修改conf来loadmodule……
	--enable-deflate=shared \ //缩小传输编码的支持
	--enable-expires=shared \ //期满头控制
	--enable-rewrite=shared \ //基于规则的URL操控
	--enable-static-support //建立一个静态链接版本的支持`
	# `make`
	# `make intall`

---------悲剧的开始 make 关联组件在 /usr/lib 下获取， 没有root权限的坑我先跳了---------
-------此处过程省略1W字-----
在同事帮助下找到了解决方案-httpd安装目录下读安装文件

`vi INSTALL` -- 细细品读吧

```
Consider if you want to use a previously installed APR and APR-Util (such as those provided with many OSes) or if you need to use the APR and APR-Util from the apr.apache.org project. If the latter, download the latest versions and unpack them to ./srclib/apr and ./srclib/apr-util (no version numbers in the directory names) and use ./configure's --with-included-apr option

```
* 将apr、apr-util 安装包拷贝到 httpd /srclib 目录
  ` cp -r /home/file/httpd-2.4.18/apr /home/file/apr`

编译安装：

	`./configure --prefix=/home/worker/lib/apache/ --sysconfdir=/home/worker/lib/httpd/ --with-included-apr --with-pcre=/home/worker/lib/pcre --enable-so  --enable-deflate=shared --e`
	# make && make install

#### 验证Apache安装成功
在`/bin`目录配置软链

`ln -s ../lib/apache/bin/apachectl`

拷贝httpd.conf文件

` cp /home/lib/httpd/httpd.conf /home/etc/httpd.conf `

* apachectl -f /home/etc/httpd.conf #启动服务
* apachectl stop  # 关闭服务
* apachectl restart # 重启服务

mkdir home/code 安装目录下增加 index.php

	<?php
		phpinfo();

修改httpd.conf 文件

	listen 9000 # 修改端口，默认80端口
	DocumentRoot "/home/code"
	<Directory "/home/code">
		AllowOverride None
		DirectoryIndex index.html index.php
		Require all granted
	</Directory>



启动服务 `apachectl -f /home/etc/httpd.conf `
访问 xxx:9000/index.php  可以看到网页上显示Apache服务器信息。
-------------------安装完成30%--------

####此处入门了vi

	常用指令
	:w   保存文件但不退出vi
	:w file 将修改另外保存到file中，不退出vi
	:w!   强制保存，不推出vi
	:wq  保存文件并退出vi
	:wq! 强制保存文件，并退出vi
	q:  不保存文件，退出vi
	:q! 不保存文件，强制退出vi
	:e! 放弃所有修改，从上次保存文件开始再编辑
	i + esc 就可以愉快的玩耍了

## 安装PHP7

官网找源文件下载

php: `http://cn2.php.net/distributions/php-7.0.14.tar.gz`
doc: `http://cn2.php.net/distributions/manual/php_manual_zh.html.gz`

跟着文档一步一步

	Unix 系统下的 Apache 2.x
	从上面列出的地方获取 Apache 源码包，然后解压：

	gzip -d httpd-2_x_NN.tar.gz
	tar -xf httpd-2_x_NN.tar
	同样，获取 PHP 源码包并解压：

	gunzip php-NN.tar.gz
	tar -xf php-NN.tar
	编译并安装 Apache。请参考 Apache 安装文档了解编译 Apache 的更多细节。

	cd httpd-2_x_NN
	./configure --enable-so
	make
	make install
	现在已经将 Apache 2.x.NN 安装在 /usr/local/apache2。本安装支持可装载模块 和标准的 MPM prefork。之后，可以使用如下命令启动 Apache 服务器：

	/usr/local/apache2/bin/apachectl start
	如果成功，可以停止 Apache 服务器并继续安装 PHP：
	/usr/local/apache2/bin/apachectl stop
	现在需要配置并编译 PHP。在这里可以用各种各样的参数来自定义 PHP，例如启动哪些扩展功能包的支持等。用 ./configure --help 命令可以列出当前可用的所有参数。在此例中，将给出一个在有 MySQL 支持的 Apache 2 上进行配置的范例。

	如果按照上面的说明从源代码编译了 Apache，下面的例子会正确匹配 apxs 的路径。如果通过其他方式安装了 Apache，需要相应的调整 apxs 的路径。注意，在有些发行版本中，可能将 apxs 更名为 apxs2。

	cd ../php-NN
	./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql
	make
	make install
	如果决定在安装后改变配置选项，只需重复最后的三步 configure，make，以及 make install，然后需要重新启动 Apache 使新模块生效。Apache 不需要重新编译。

	请注意，除非明确有提示，否则“make install”命令将安装 PEAR、各种 PHP 工具诸如 phpize，并安装 PHP CLI 等等。

	配置 php.ini

	cp php.ini-development /usr/local/lib/php.ini
	可以编辑 php.ini 来设置 PHP 运行时的选项。如果想要把此文件放到另外的位置，需要在步骤 5 添加 --with-config-file-path=/path 选项。

	如果选择了 php.ini-production，请务必阅读其中的变更列表，它们将影响 PHP 的执行。

	编辑 httpd.conf 文件以调用 PHP 模块。LoadModule 达式右边的路径必须指向系统中的 PHP 模块。以上的 make install 命令可能已经完成了这些，但务必要检查。

	LoadModule php5_module modules/libphp5.so
	告知 Apache 将特定的扩展名解析成 PHP，例如，让 Apache 将扩展名 .php 解析成 PHP。为了避免潜在的危险，例如上传或者创建类似 exploit.php.jpg 的文件并被当做 PHP 执行，我们不再使用 Apache 的 AddType 指令来设置。参考下面的例子，你可以简单的将需要的扩展名解释为 PHP。我们演示为增加.php。

	<FilesMatch \.php$>
    	SetHandler application/x-httpd-php
	</FilesMatch>
	或者，你也想将 .php，.php2，.php3，.php4，.php5，.php6，以及 .phtml 文件都当做 PHP 来运行，我们无需额外的设置，仅需按照下面这样来：

	<FilesMatch "\.ph(p[2-6]?|tml)$">
    	SetHandler application/x-httpd-php
	</FilesMatch>
	然后，可以将 .phps 文件由 PHP 源码过滤器处理，使得其在显示时可以高亮源码，设置如下：

	<FilesMatch "\.phps$">
  	  SetHandler application/x-httpd-php-source
	</FilesMatch>
	mod_rewrite 也有助于将那些不需要运行的 .php 文件的源码高亮显示，而并不需要将他们更名为 .phps 文件：

	RewriteEngine On
	RewriteRule (.*\.php)s$ $1 [H=application/x-httpd-php-source]
		不要在正式生产运营的系统上启动 PHP 源码过滤器，因为这可能泄露系统机密或者嵌入的代码中的敏感信息。

	按照通常的方式启动 Apache 服务：

	/usr/local/apache2/bin/apachectl start
	或者

	service httpd restart

root权限的重要性又出现了， 老办法继续读`INSTALL`文件， 虽然英文太差，  那也自能怪自己罗！  

    The second method involves providing the `DESTDIR' variable.  For
    example, `make install DESTDIR=/alternate/directory' will prepend
    `/alternate/directory' before all installation names.

第二次安装php

	./configure --prefix=/home/worker/lib/php7 --with-apxs2=/home/worker/lib/apache/bin/apxs --with-mysqli
	make

	make install DESTDIR=/home/worker/lib

cp php.ini-development /home/lib/php7/lib/php.ini

编辑 httpd.conf
`LoadModule php7_module modules/libphp7.so`


成功了， 兴奋ing


## 安装Piwik

5-minute Piwik Installation，那就5分钟结束吧

下载piwik [piwik.org](piwik.org)

安装教程 [http://piwik.org/docs/installation/](http://piwik.org/docs/installation/)

1. 解压piwik 到`/home/code`
2. 浏览器访问 xxxx:9000/piwik/index.php
一切都很愉快的， next>> ....

提示缺少依赖 libxml, pdo/mysql (修改php.ini)



http://blog.csdn.net/duck_arrow/article/details/23347565

http://blog.csdn.net/u013032788/article/details/46729003

http://www.cnblogs.com/mazefeng/p/3357607.html


https://www.freetype.org/contact.html

https://sourceforge.net/directory/os:mac/?q=libjpeg

http://www.cnblogs.com/endv/p/4537278.html

http://blog.csdn.net/dodott/article/details/49665753
