#编译Docker镜像:
```
 docker build --force-rm -t wdmsyf/xampp .
```
#启动依赖的MySQL容器
```
docker run -d --name alpine-mysql -p 3366:3306 -v /data1/db/alpine-mysql:/app -e MYSQL_DATABASE=admin -e MYSQL_USER=itisme -e MYSQL_P
ASSWORD=it'sme -e MYSQL_ROOT_PASSWORD=111111 wdmsyf/mysql-alpine
```
以上命令中参数的含义:
MYSQL_USER: 具有全库权限的用户
MYSQL_PASSWORD: MYSQL_USER 用户对应的密码
MYSQL_ROOT_PASSWORD: 数据库root用户密码

#运行镜像:
www存储在宿主机
```
docker run --name xampp_my -p 4022:22 -p 4080-4089:80-89 -d -v ~/my_htdocs:/www wdmsyf/xampp
```
其中：

 - xampp_my是容器名称 4022是映射到宿主机的SSH端口，在外部可以通过 ssh <宿主机地址> -p 4022   
   登录到xampp_my容器
 - 4080-4089:80-89是容器中80到89端口映射到4080到4089，一个容器最多可以承载10个web应用
 - ~/my_htdocs是宿主机上对应的web应用的根目录，~/my_htdocs目录的下一级目录都可以作为一个web应用。~/my_htdocs/root是默认web目录
 - 目录~/my_htdocs需要你自己在宿主机上建立，系统不会自动建该目录。你也可以使用其他目录，只需在上面命令行中相应修改即可

#使用自定义apache参数:
```
docker run --name xampp_my -p 4022:22 -p 4080-4089:80-89 -d -v ~/my_htdocs:/www  -v ~/my_apache_conf:/opt/lampp/apache2/conf.d wdmsyf/xampp
```
其中：
~/my_apache_conf 是宿主机上对应的apache配置文件目录

#xampp链接到MySQl容器
```
docker run --name xampp_my -p 4022:22 -p 4080-4089:80-89 -d -v ~/my_htdocs:/www -v ~/my_apache_conf:/opt/lampp/apache2/conf.d --link alpine-mysql wdmsyf/xampp
```
其中：
apline-mysql是mysql容器的名称


#Bash连接到容器:
```
docker exec -ti xampp_my bash
```

#重启xampp服务:
```
docker exec xampp_my /opt/lampp/lampp restartapache
```

#SSH:
SSH root 用户默认密码是 root，对外暴露端口由启动命令行决定，例如：4022
```
ssh root@localhost -p 4022
```


----------

#FAQ：
1、在连接两个容器后，如果执行 mysql，会报错：TERM environment variable not set.
答：执行
```
export TERM=dumb
```
或
```
$ echo "export TERM=dumb" >> ~/.bashrc
$ source ~/.bashrc
```
参考资料:
[https://github.com/dockerfile/mariadb/issues/3][1]


2、如何从docker外导入数据：
答：
```
 docker exec -i CONTAINER_ID /bin/bash -c "export TERM=xterm && mysql -proot -uroot database" < import.sql
```

3、如何配置phpmyadmin的登录策略；
答：未经配置的phpMyAdmin很不安全，容易受到攻击，或者根本无法正常使用，phpMyAdmin有3种授权模式：

 - cookie: 显示一个web登录页面，输入mysql的用户名和密码，然后进入管理界面
 - http: 显示1个windows登录框，输入mysql的用户名和密码，然和进入管理
 - config: 把mysql用户名和密码直接填入config.inc.php，不显示登录界面，直接进入管理界面

phpMyAdmin的配置文件名为config.inc.php，各版本的config.inc.php修改方法如下：

 - 2.6以前版本：将config.inc.sample.php改为 config.inc.php
 - 2.7版本：将config.default.php改为config.inc.php
 - 2.8版本：用配置脚本 ‘/script/setup.php’生成配置文件，生成的文件拷贝下来，手动存为config.inc.php
