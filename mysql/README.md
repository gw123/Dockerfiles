# 说明
基于apline的mysql镜像，编译后占用空间很小（60多M）

# 编译
```
docker build -t wdmsyf/alpine-mysql .
```

#运行
```
docker run -it --rm -v $(pwd):/app -p 3306:3306 wdmsyf/alpine-mysql
```
其中：
以当前目录作为mysql的数据存储目录（见my.cnf中配置）


# 用法
```
docker run -it --name alpine-mysql -p 3306:3306 -v $(pwd):/app -e MYSQL_DATABASE=admin -e MYSQL_USER=itisme -e MYSQL_PASSWORD=it'sme -e MYSQL_ROOT_PASSWORD=111111 wdmsyf/alpine-mysql
```
执行完毕创建的容器名称为alpine-mysql，
会自动创建一个名为admin的数据库，其用户是itisme，密码是it'sme，
并设置了root密码（默认为111111)

