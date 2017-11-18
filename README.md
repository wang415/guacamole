# guacamole
【Guacamole#1】Guacamole介绍及安装
一、Guacamole介绍
Guacamole是一个提供了基于HTML5 web应用程序的远程桌面代理服务器。通过使用Guacamole服务器，我们很轻松的在浏览器上远程访问Guacamole代理的主机。
 
我们可以在浏览器访问Guacamole页面的时候，此时，浏览器会通过HTTP使用Guacamole协议与Guacamole 服务器中的Web服务器进行连接。Guacamole Web应用会从用户的请求中读取Guacamole协议，并将其转发给guacd（本地Guacamole代理）。Guacd根据web 应用转发过来的Guacamole协议来代替用户连接到远程桌面服务器。在Guacamole Web应用与guacd进行通信的时候，两者均不需要知道实际使用的远程桌面协议是什么，即协议不可知性。
在Guacamole的组成中主要包含如下三部分：
1.	Guacamole协议是用于远程显示和事件传输的协议，不实现特定的桌面环境支持，实现了现有远程桌面的超集。
2.	guacd是Guacamole的核心，guacd也不了解任何具体的远程桌面协议，而是实现了通过web应用转发的Guacamole协议来确定哪些协议需要加载，哪些参数必须传递给它。
3.	web应用程序是Guacamole与用户进行交互的部分。Apache提供了基于Java的编写的Web应用程序，但是这并不代表Guacamole 只支持Java。Guacamole是一个API。
二、Guacamole Server安装
采用的服务器是在VMware WorkStation上运行的CentOS 7.3作为Guacamole Server，同时在VMware WorkStation上安装两台测试虚拟机，操作系统为Ubuntu 17 和Windows 10虚拟机各一台。此次主要安装的是Guacamole 0.9.13.
1、安装必要环境
这些必要的环境主要是指编译Guacamole Server时所需要的工具及其依赖项所需要的软件源。

#yum -y install epel-release
#rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
#rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
#yum update -y
#yum -y install wget
#yum -y groupinstall "Development Tools"

2、安装依赖项
我安装的依赖项包括必须安装的依赖项和可选依赖项的全部
#yum -y install cairo-devel libjpeg-turbo-devel libpng-devel uuid-devel freerdp-devel pango-devel libssh2-devel libtelnet-devel libvncserver-devel pulseaudio-libs-devel openssl-devel libvorbis-devel libwebp-devel ffmpeg-devel 

3、获取、编译和安装Guacamole Server
#cd /tmp
#tar xzvf guacamole-server-0.9.13-incubating.tar.gz
#cd guacamole-server-0.9.13-incubating
#autoreconf -fi
#./configure --with-init-dir=/etc/init.d
#make
#make install
#ldconfig

Configure  一定要通过  guacd 是后台服务   guacenc 是解码录像文件
 

4、启动guacd
#service guacd start
三、Guacamole Client安装
Guacamole Client的安装其实是相当简单的主要就是安装支持Java的Web服务器和Guacamole Client war包。
#yum -y install java-1.8.0-openjdk tomcat
#cd /tmp
#cp guacamole-0.9.13-incubating.war /var/lib/tomcat/webapps/guacamole.war

四、Guacamole配置
Guacamole默认从其自己的配置文件目录中读取配置文件，仅在找不到配置文件的情况下才会使用类路径。Guacamole查找配置文件的目录的顺序如下：
a.	在系统属性guacamole.home中指定的目录 。
b.	在环境变量中指定的目录 GUACAMOLE_HOME。（env, /etc/profile）
c.	该目录.guacamole位于运行servlet容器的用户的主目录中。(Tomcat根目录)
GUACAMOLE_HOME目录中主要有Guacamole主配置文件guacamole.properties，Guacamole Logback的日志记录系统配置文件logback.xml，Guacamole扩展的安装位置extensions文件夹，Guacamole扩展所需的库lib文件夹。
在我这次搭建中主要使用到了默认的认证方式user-mapping.xml，guacamole.properties配置如下。

guacd-hostname: localhost
guacd-port:     4822
basic-user-mapping: /etc/guacamole/user-mapping.xml
user-mapping: /etc/guacamole/user-mapping.xml
采用了默认认证方式user-mapping.xml，具体配置如下。
<user-mapping>
    <authorize username="admin" password="admin">
        <connection name="CentOS 7 TigerVNC">
            <protocol>vnc</protocol>
            <param name="hostname">localhost</param>
            <param name="port">5901</param>
            <param name="password">123456</param>
        </connection>
        <connection name="CentOS SSH">
        <protocol>ssh</protocol>
            <param name="hostname">localhost</param>
    </connection>

        <connection name="Windows 10(Test)">
            <protocol>rdp</protocol>
            <param name="hostname">192.168.199.231</param>
            <param name="port">3389</param>
            <param name="security">tls</param>
            <param name="ignore-cert">true</param> 
        </connection>
        <connection name="Ubuntu x11vnc">
            <protocol>vnc</protocol>
            <param name="hostname">192.168.199.141</param>
            <param name="port">5900</param>
            <param name="password">123456</param>
        </connection>
    </authorize>
</user-mapping>
在user-mapping.xml中<authorize>标签添加一个用户，在该标签中username参数是用户登录名，password是用户登录密码，encoding是加密方式（该参数为可选）。
<authorize username ="USER" password ="PASS"> 
    ... 
</ authorize>

在<authorize>标签中添加一个<connection>标签，即为用户添加一个连接，参数name是连接名称。
<connection name ="NAME">
    ...
 </connection>

Gucamole配置具体操作如下：
1、创建GUACAMOLE_HOME和配置文件
#mkdir /etc/guacamole
#mkdir /etc/guacamole/lib
#mkdir /etc/guacamole/extensions
#vim /etc/guacamole/guacamole.properties
#vim /etc/guacamole/user-mapping.xml

2、配置Guacamole
#mkdir /user/shares/tomcat/.guacamole
#ln -s /etc/guacamole/guacamole.properties /user/shares/tomcat/.guacamole

#chmod -R 777 /etc/guacamole

#firewall-cmd --zone=public --add-port=8080/tcp --permanent
#firewall-cmd --reload
#service tomcat start


五、测试
在浏览器中打开Guacamole Web应用，地址为http://Guacamole_Server_IP:8080/guacamole
Guacamole登录界面
下图是“CentOS 7 TigerVNC”连接结果，主要测试TigerVNC,采用VNC协议。
 
CentOS 7 TigerVNC
下图是“Windows 10(Test)”连接的结果展示，测试RDP。
 
Windows 10(Test)
下图是”CentOS SSH”连接结果，测试SSH。
 
CentOS SSH
下图是“Ubuntu x11vnc”，测试x11vnc。
 
Ubuntu x11vnc
六、总结
Guacamole对RDP、VNC以及SSH支持还是比较完美的。我仅仅简单地测试了RDP、VNC和SSH三种协议以及一些比较简单的功能。在后期的文章中将会详细介绍连接参数、常见错误分析以及其他验证方式。
MySQL 连接配置
1、创建GUACAMOLE_HOME和配置文件
#mkdir /etc/guacamole
#mkdir /etc/guacamole/lib
#mkdir /etc/guacamole/extensions
#vim /etc/guacamole/guacamole.properties


2、配置cp guacamole-auth 文件
Cd /tmp/
Tar -zxf guacamole-auth-jdbc-0.9.13-incubating.tar.gz
Cd guacamole-auth-jdbc-0.9.13-incubating
Cp mysql/guacamole-auth-jdbc-mysql-0.9.13-incubating.jar /etc/guacamole/extensions
Cd /tmp/
Cp mysql-connector-java-5.1.43-bin.jar /etc/guacamole/lib

3、编辑配置文件guacamole.properties
mysql-hostname: localhost
mysql-port: 3306
mysql-database: guacamole_db
mysql-username: guacamole_user
mysql-password: DongDong_password
4、安装Mysql 
rpm -qa | grep mariadb
rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
yum install vim libaio net-tools
yum -y install mariadb-server mariadb mariadb-devel 
systemctl start mariadb
systemctl enable mariadb
firewall-cmd --permanent --add-service mysql
systemctl restart firewalld.service
iptables -L -n|grep 3306
mysql_secure_installation    配置mysql 
firewall-cmd --permanent --add-service mysql
systemctl restart firewalld.service
iptables -L -n|grep 3306
2、创建guacamole 数据库
Mysql -uroot -p
CREATE DATABASE guacamole_db;
CREATE USER 'guacamole_user'@'localhost' IDENTIFIED BY 'some_password';
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'guacamole_user'@'localhost';
FLUSH PRIVILEGES;
quit

3、导入guacamole 数据库
Cd /tmp/
Cd guacamole-auth-jdbc-0.9.13-incubating
ls schema/
cat schema/*.sql | mysql -u root -p guacamole_db
4、重启服务
service guacd start
vi /etc/tomcat7/server.xml
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <Context path="" docBase="../webapps/guacamole.war"/>
service tomcat7 restart
 
六、测试
到此可以配置用户及查看history 
 
以下配置是主机的录像配置
 
3、文件不能直接查看需要进行转换
guacenc -s 1920x1080 -r 4000000 Bastion.mp4

 
上传文件下载 
Ctrl Alt+Shift
 

 
 
Copy 文字
Ctrl Alt+Shift
 
 
 




