FROM debian:jessie 
MAINTAINER Sheng Youfu<wdmsyf@sina.com>
 
ENV DEBIAN_FRONTEND noninteractive 
RUN apt-get update --fix-missing
# curl是下载xampp安装程序必须的；net-tools给xampp提供netstat命令 
RUN apt-get -y install curl net-tools

#RUN curl -o xampp-linux-installer.run "https://downloadsapachefriends.global.ssl.fastly.net/xampp-files/5.6.21/xampp-linux-x64-5.6.21-0-installer.run?from_af=true"
RUN curl -o xampp-linux-installer.run "https://jaist.dl.sourceforge.net/project/xampp/XAMPP%20Linux/5.6.21/xampp-linux-x64-5.6.21-0-installer.run"
RUN chmod +x xampp-linux-installer.run
RUN bash -c './xampp-linux-installer.run'
RUN ln -sf /opt/lampp/lampp /usr/bin/lampp

# 移除xampp安装程序
RUN rm ./xampp-linux-installer.run

# 启用web(移除apache的安全检查) 
RUN sed -i.bak s'/Require local/Require all granted/g' /opt/lampp/etc/extra/httpd-xampp.conf

#在根目录下创建www，并把原来的htdocs目录改成www目录的链接，www目录下的每一个目录可以映射为apache的一个vhost
#www目录里不要放置任何应用
RUN mkdir /www && \
    mkdir /www/root && \
    mv /opt/lampp/htdocs/* /www/root && \
    rm -rf /opt/lampp/htdocs && \
    ln -s /www /opt/lampp/htdocs

#修改原始的httpd.conf文件，把配置文件目录转移到新目录，以便于映射到宿主机的目录
#修改apache默认的应用目录
RUN sed -i.bak s'#etc/extra#etc/httpd.d/extra#g' /opt/lampp/etc/httpd.conf && \
	sed -i.bak s'#/opt/lampp/htdocs#/opt/lampp/htdocs/root/#g' /opt/lampp/etc/httpd.conf

#把apache的配置文件移入单独的文件夹，以便于映射到宿主机上
RUN mkdir /opt/lampp/etc/httpd.d && \
	mv /opt/lampp/etc/extra /opt/lampp/etc/httpd.d/ && \
	mv /opt/lampp/etc/httpd.conf* /opt/lampp/etc/httpd.d/

RUN echo "Include /opt/lampp/etc/httpd.d/httpd.conf" > /opt/lampp/etc/httpd.conf

# Enable includes of several configuration files
#RUN mkdir /opt/lampp/apache2/conf.d && \
#    echo "IncludeOptional /opt/lampp/apache2/conf.d/*.conf" >> /opt/lampp/etc/httpd.conf

# 创建一个apache配置文件的映射目录
RUN ln -s /opt/lampp/etc/httpd.d /httpd.d

# SSH server 
RUN apt-get install -y -q supervisor openssh-server
RUN mkdir -p /var/run/sshd

#Endable XDEBUG extension
RUN echo "" >> /opt/lampp/etc/php.ini && \
    echo "[xdebug]" >> /opt/lampp/etc/php.ini && \
    echo "zend_extension = /opt/lampp/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.default_enable = On" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.collect_params = On" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_enable = 1" >> /opt/lampp/etc/php.ini && \
    echo "#如果开启此，将忽略下面的 xdebug.remote_host 的参数" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_connect_back = Off" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_host = 127.0.0.1" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_port = 9000" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_handler = dbgp" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_mode = req" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.remote_autostart = no" >> /opt/lampp/etc/php.ini && \
    echo "xdebug.idekey="PHPSTORM"" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.profiler_append = 0" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.profiler_enable = 0" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.profiler_enable_trigger = 0" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.profiler_output_dir = "/data1/logs"" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.profiler_output_name = "xdebug.profiler.out.%t-%s"" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.trace_output_dir = "/data1/logs"" >> /opt/lampp/etc/php.ini && \
    echo "; xdebug.remote_log=/data1/logs/xdebug.log" >> /opt/lampp/etc/php.ini

# 创建php配置文件目录并把配置文件移入
RUN mkdir /opt/lampp/etc/php.d && \
	mv /opt/lampp/etc/php.ini* /opt/lampp/etc/php.d

#修改xampp启动脚本，重新指定php.ini文件的位置
RUN sed -i.bak s'#export XAMPP_ROOT#export XAMPP_ROOT\nexport PHP_INI_SCAN_DIR=$XAMPP_ROOT/etc/php.d#g' /opt/lampp/xampp

# Output supervisor config file to start openssh-server 
RUN echo "[program:openssh-server]" >> /etc/supervisor/conf.d/supervisord-openssh-server.conf && \
	echo "command=/usr/sbin/sshd -D" >> /etc/supervisor/conf.d/supervisord-openssh-server.conf && \
	echo "numprocs=1" >> /etc/supervisor/conf.d/supervisord-openssh-server.conf && \
	echo "autostart=true" >> /etc/supervisor/conf.d/supervisord-openssh-server.conf && \
	echo "autorestart=true" >> /etc/supervisor/conf.d/supervisord-openssh-server.conf

# 允许root用户使用密码登录 
# root password is: root 
RUN sed -ri 's/PermitRootLogin without-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

# Set root password 
# password hash generated using this command: openssl passwd -1 -salt xampp root 
#RUN sed -ri 's/root\:\*/root\:\$1\$xampp\$5\/7SXMYAMmS68bAy94B5f\./g' /etc/shadow
# password: passw0rdt00r
RUN sed -ri 's/root\:\*/root\:\$1\$xampp\$3zdDfqh\/GurztSQ8DfJcy\//g' /etc/shadow

# Few handy utilities which are nice to have 
#RUN apt-get -y install nano vim less --no-install-recommends
RUN apt-get -y install vim less --no-install-recommends

RUN apt-get clean

#设置TERM环境变量，以免在执行MySQL出错
RUN echo "export TERM=dumb" >> ~/.bashrc
#RUN source ~/.bashrc


VOLUME [ "/var/log/mysql/", "/var/log/apache2/" ]

# 对外暴露MySQL端口
EXPOSE 3306
# 对外暴露sshd端口
EXPOSE 22 
# 每个容器可对外暴露从80至89共10个http端口，以便于扩展
EXPOSE 80
EXPOSE 81
EXPOSE 82
EXPOSE 83
EXPOSE 84
EXPOSE 85
EXPOSE 86
EXPOSE 87
EXPOSE 88
EXPOSE 89

# 生成启动脚本

#RUN echo '/opt/lampp/lampp start' >> /startup.sh
#RUN echo '/opt/lampp/lampp startmysql' >> /startup.sh
#RUN echo '/opt/lampp/lampp startftp' >> /startup.sh
# 只启动apache，MySQl由单独的容器提供
RUN echo '/opt/lampp/lampp startapache' >> /startup.sh

RUN echo '/usr/bin/supervisord -n' >> /startup.sh

CMD ["sh", "/startup.sh"]


