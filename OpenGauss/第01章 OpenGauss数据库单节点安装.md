## OpenGauss数据库单节点安装

### 1.更新yum源为华为云源
```shell
[root@localhost ~]# cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
[root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo https://repo.huaweicloud.com/repository/conf/CentOS-7-reg.repo
```

### 2.清除原有yum缓存并更新缓存
```shell
[root@localhost ~]# yum clean all
[root@localhost ~]# yum makecache
```

### 3.安装系统软件依赖
```shell
[root@localhost ~]# yum install libaio-devel -y
[root@localhost ~]# yum install flex -y
[root@localhost ~]# yum install bison -y
[root@localhost ~]# yum install ncurses-devel -y
[root@localhost ~]# yum install glibc-devel -y
[root@localhost ~]# yum install patch -y
[root@localhost ~]# yum install redhat-lsb-core -y
[root@localhost ~]# yum install readline-devel -y
[root@localhost ~]# yum install bzip2 -y 
[root@localhost ~]# yum install net-tools -y 
[root@localhost ~]# yum install sysstat -y 
```

### 4.安装Python

```shell
## 安装python3
[root@localhost ~]# yum install python3
## 查看Python版本
[root@localhost ~]# python3 -V
```

### 5.系统环境设置

#### 5.1 设置防火墙

常规的做法应该是建防火墙例外，现在为了快速搭建，直接关闭防火墙

```shell
[root@localhost ~]# systemctl status firewalld

[root@localhost ~]# systemctl stop firewalld

[root@localhost ~]# systemctl disable firewalld
```

#### 5.2 关闭SELINUX

选择临时关闭或者永久关闭SELinux。

##### 5.2.1 临时关闭SELinux

```shell
[root@localhost ~]# setenforce 0
```

##### 5.2.2 永久关闭SElinux

```shell
[root@localhost ~]# vi /etc/selinux/config
```

找到SELINUX=enforcing，按i进入编辑模式，将参数修改为 SELINUX=disabled 修改完成后，按下键盘Esc键，执行命令:wq，保存并退出文件。

```shell
## 重启服务器
[root@localhost ~]# shutdown -r now
```

重启后，运行命令getenforce，验证SELinux状态为disabled，表明SELinux已关闭。

#### 5.3 设置时区和时间

```shell
[root@localhost ~]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

#### 5.4 更新hostname
```shell
[root@localhost ~]# hostname
	localhost.localdomain
	
[root@localhost ~]# hostnamectl set-hostname opengauss-201

[root@localhost ~]# hostname
	opengauss-201
	
## 配置hosts解析
[root@localhost ~]# echo "192.168.1.201 opengauss-201">>/etc/hosts

[root@localhost ~]# ping opengauss-201
	PING opengauss-201 (192.168.1.201) 56(84) bytes of data.
	64 bytes from opengauss-201 (192.168.1.201): icmp_seq=1 ttl=64 time=0.015 ms
	64 bytes from opengauss-201 (192.168.1.201): icmp_seq=2 ttl=64 time=0.132 ms
	64 bytes from opengauss-201 (192.168.1.201): icmp_seq=3 ttl=64 time=0.045 ms
	......
```

#### 5.5 设置UTF-8字符集
```shell
[root@localhost ~]# vim /etc/locale.conf
	LANG="zh_CN.UTF-8"

[root@localhost ~]# vim /etc/profile
	## 新增字符集
	export LANG=zh_CN.UTF-8
```

#### 5.6 修改系统参数sysctl.conf

```shell
[root@opengauss-201 ~]# vim /etc/sysctl.conf
  net.ipv4.tcp_max_tw_buckets = 10000
  net.ipv4.tcp_tw_reuse = 1
  net.ipv4.tcp_tw_recycle = 1
  net.ipv4.tcp_keepalive_time = 30
  # add
  net.ipv4.tcp_keepalive_probes = 9
  net.ipv4.tcp_keepalive_intvl = 30

  # add
  net.ipv4.tcp_retries1 = 5
  net.ipv4.tcp_syn_retries = 5
  net.ipv4.tcp_synack_retries = 5

  net.ipv4.tcp_retries2 = 12
  net.ipv4.ip_local_reserved_ports = 15400-15407,20050-20057

  # add
  net.ipv4.tcp_syncookies = 1

  net.core.wmem_max = 21299200
  net.core.rmem_max = 21299200
  net.core.wmem_default = 21299200
  net.core.rmem_default = 21299200
  net.sctp.sctp_mem = 94500000 915000000 927000000
  net.sctp.sctp_rmem = 8192 250000 16777216
  net.sctp.sctp_wmem = 8192 250000 16777216
  kernel.sem = 250 6400000 1000 25600
  # add
  net.ipv4.ip_local_port_range = 26000-65535
  net.ipv4.tcp_fin_timeout = 60
  net.ipv4.tcp_sack = 1
  net.ipv4.tcp_timestamps = 1

  net.ipv4.tcp_rmem = 8192 250000 16777216
  net.ipv4.tcp_wmem = 8192 250000 16777216
  vm.min_free_kbytes = 812390
  # add
  vm.overcommit_memory = 0
  vm.extfrag_threshold = 500
  vm.overcommit_ratio = 90

  net.core.netdev_max_backlog = 65535
  net.ipv4.tcp_max_syn_backlog = 65535
  net.core.somaxconn = 65535
  kernel.shmall = 1152921504606846720
  kernel.shmmax = 18446744073709551615
  
## 使得系统参数生效  
[root@opengauss-201 ~]# sysctl -p
```

#### 5.7 修改文件句柄

```shell
[root@opengauss-201 ~]# echo "* soft nofile 1000000" >>/etc/security/limits.conf

[root@opengauss-201 ~]# echo "* hard nofile 1000000" >>/etc/security/limits.conf
 
## 重启服务器
[root@opengauss-201 ~]# shutdown -r now
```

#### 5.8 修改系统版本显示

```shell
[root@opengauss-201 ~]# echo "CentOS Linux release 7.6.1810 (Core)">/etc/centos-release
```

### 6.OpenGauss安装

#### 6.1 下载OpenGauss安装包

```shell
[root@opengauss-201 ~]# cd /usr/local/src

[root@opengauss-201 src]# wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/3.0.0/x86/openGauss-3.0.0-CentOS-64bit-all.tar.gz
```

#### 6.2 解压文件

```shell
[root@opengauss-201 src]# mkdir -p /opt/software/openGauss

[root@opengauss-201 src]# chmod 755 -R /opt/software

[root@opengauss-201 src]# cd /opt/software/openGauss

[root@opengauss-201 openGauss]# mv /usr/local/src/openGauss-3.0.0-CentOS-64bit-all.tar.gz .

[root@opengauss-201 openGauss]# tar -zxvf openGauss-3.0.0-CentOS-64bit-all.tar.gz

[root@opengauss-201 openGauss]# tar -zxvf openGauss-3.0.0-CentOS-64bit-om.tar.gz 
```

#### 6.3 编辑XML文件

```shell
[root@opengauss-201 openGauss]# vi cluster_config.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ROOT>
    <!-- openGauss整体信息 -->
    <CLUSTER>
        <!-- 数据库名称 -->
        <PARAM name="clusterName" value="openGauss" />
        <!-- 数据库节点名称(hostname) -->
        <PARAM name="nodeNames" value="opengauss-201" />
        <!-- 数据库安装目录-->
        <PARAM name="gaussdbAppPath" value="/opt/huawei/install/app" />
        <!-- 日志目录-->
        <PARAM name="gaussdbLogPath" value="/var/log/omm" />
        <!-- 临时文件目录-->
        <PARAM name="tmpMppdbPath" value="/opt/huawei/tmp" />
        <!-- 数据库工具目录-->
        <PARAM name="gaussdbToolPath" value="/opt/huawei/install/om" />
        <!-- 数据库core文件目录-->
        <PARAM name="corePath" value="/opt/huawei/corefile" />
        <!-- 节点IP，与数据库节点名称列表一一对应 -->
        <PARAM name="backIp1s" value="192.168.1.201"/>    
    </CLUSTER>
    <!-- 每台服务器上的节点部署信息 -->
    <DEVICELIST>
        <!-- 节点1上的部署信息 -->
        <DEVICE sn="opengauss-201">
            <!-- 节点1的主机名称 -->
            <PARAM name="name" value="opengauss-201"/>
            <!-- 节点1所在的AZ及AZ优先级 -->
            <PARAM name="azName" value="AZ1"/>
            <PARAM name="azPriority" value="1"/>
            <!-- 节点1的IP，如果服务器只有一个网卡可用，将backIP1和sshIP1配置成同一个IP -->
            <PARAM name="backIp1" value="192.168.1.201"/>
            <PARAM name="sshIp1" value="192.168.1.201"/>
               
            <!--dbnode-->
            <PARAM name="dataNum" value="1"/>
            <PARAM name="dataPortBase" value="15400"/>
            <PARAM name="dataNode1" value="/opt/huawei/install/data/dn"/>
            <PARAM name="dataNode1_syncNum" value="0"/>
        </DEVICE>
    </DEVICELIST>
</ROOT>
```

#### 6.4 创建数据库组

```shell
[root@opengauss-201 openGauss]# groupadd dbgrp
```

#### 6.5 预安装OpenGauss

```shell
[root@opengauss-201 openGauss]# cd script/

## 预安装
[root@opengauss-201 script]# ./gs_preinstall -U omm -G dbgrp -X /opt/software/openGauss/cluster_config.xml

在执行过程中需要手动输入以下指令跟密码：
......
Are you sure you want to create the user[omm] (yes/no)? yes
......
Please enter password for cluster user.
Password:(输入密码：jeesaas@liubo)
Please enter password for cluster user again.
Password:(再次输入密码:jeesaas@liubo)
......
Set ARM Optimization.
No need to set ARM Optimization.
Fixing server package owner.
Setting finish flag.
Successfully set finish flag.
Preinstallation succeeded.
```

#### 6.6 正式安装OpenGauss

```shell
## 切换到omm用户
[root@opengauss-201 script]# su - omm

[omm@opengauss-201 ~]$ gs_install -X /opt/software/openGauss/cluster_config.xml --gsinit-parameter="--locale=zh_CN.utf8"
......
Please enter password for database:(输入密码：Gauss@123)
Please repeat for database:(输入密码：Gauss@123)
......
Check consistence of memCheck and coresCheck on database nodes.
Configuring pg_hba on all nodes.
Configuration is completed.
Successfully started cluster.
Successfully installed application.
end deploy..
```

#### 6.7 查看进程

```shell
[omm@opengauss-201 ~]$ ps -ef|grep gauss
	omm        3548 114161  0 19:12 pts/0    00:00:00 grep --color=auto gauss
	omm      129515      1  5 19:09 ?        00:00:07 /opt/huawei/install/app/bin/gaussdb -D /opt/huawei/install/data/dn
```

#### 6.8 查看数据库状态

```shell
[omm@opengauss-201 ~]$ gs_om -t status
	-----------------------------------------------------------------------

	cluster_name    : openGauss
	cluster_state   : Normal
	redistributing  : No

	-----------------------------------------------------------------------
```
#### 6.9 单节点集群启停

```sql
## 单节点集群停止
[omm@opengauss-201 ~]$ gs_om -t stop
	Stopping cluster.
	=========================================
	Successfully stopped cluster.
	=========================================
	End stop cluster.

## 单节点集群启动
[omm@opengauss-201 ~]$ gs_om -t start
	Starting cluster.
	=========================================
	[SUCCESS] opengauss-201
	for g_instance.attr.attr_storage.cstore_buffers (1024 Mbytes) or shared memory (4456 Mbytes) is larger.
	=========================================
	Successfully started.
```

### 7.数据库操作

#### 7.1 gsql连接数据库

我们在cluster_config.xml文件中配置的dataPortBase是15400，所以，我们可以通过以下命令连接数据库。

```shell
## 连接数据库
[omm@opengauss-201 ~]$ gsql -d postgres -p 15400

## 查看数据库列表
openGauss=# \l
                             List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | Access privileges 
-----------+-------+----------+------------+------------+-------------------
 postgres  | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | 
 template0 | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | =c/omm           +
           |       |          |            |            | omm=CTc/omm
 template1 | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | =c/omm           +
           |       |          |            |            | omm=CTc/omm
(3 rows)
```

#### 7.2 创建数据库

```sql
## 创建数据库
openGauss=# CREATE DATABASE testdb WITH ENCODING 'UTF-8' template = template0;
	CREATE DATABASE

## 查看数据库列表
openGauss=# \l
                             List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | Access privileges 
-----------+-------+----------+------------+------------+-------------------
 postgres  | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | 
 template0 | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | =c/omm           +
           |       |          |            |            | omm=CTc/omm
 template1 | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | =c/omm           +
           |       |          |            |            | omm=CTc/omm
 testdb    | omm   | UTF8     | zh_CN.utf8 | zh_CN.utf8 | 
(4 rows)
```

#### 7.3 切换数据库

```sql
openGauss=# \c testdb
	Non-SSL connection (SSL connection is recommended when requiring high-security)
	You are now connected to database "testdb" as user "omm".
testdb=# 
```

#### 7.4 数据库常见操作

```sql
# 创建新表 
CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);
# 插入数据 
INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');
# 选择记录 
SELECT * FROM user_tbl;
# 更新数据 
UPDATE user_tbl set name = '李四' WHERE name = '张三';
# 删除记录 
DELETE FROM user_tbl WHERE name = '李四' ;
# 添加栏位 
ALTER TABLE user_tbl ADD email VARCHAR(40);
# 更新结构 
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;
# 更名栏位 
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;
# 删除栏位 
ALTER TABLE user_tbl DROP COLUMN email;
# 表格更名 
ALTER TABLE user_tbl RENAME TO backup_tbl;
# 删除表格 
DROP TABLE IF EXISTS backup_tbl;
```

#### 7.5 退出

```sql
testdb=# \q

[omm@opengauss-201 ~]$ 
```

#### 7.6 数据库软件卸载

```shell
[root@opengauss-201 ~]# /opt/software/openGauss/script/gs_uninstall --delete-date -L 

[root@opengauss-201 ~]# rm -rf /opt/huawei
```

### 8 远程访问

安装OpenGauss数据库之后，默认是只接受本地访问连接。如果想在其他主机上访问OpenGauss数据库服务器，就需要进行相应的配置。

配置远程连接OpenGauss数据库的步骤很简单，只需要修改data目录下的pg_hba.conf和postgresql.conf配置文件。

- pg_hba.conf：配置对数据库的访问权限；

- postgresql.conf：配置PostgreSQL数据库服务器的相应的参数。

#### 8.1 更加加密方式

定位到#listen_addresses = 'localhost'。PostgreSQL安装完成后，默认只接受来自本机localhost的连接请求。

将行开头都#去掉，将行内容修改为listen_addresses = '*' 来允许数据库服务器监听来自任何主机的连接请求！

```shell
[omm@opengauss-201 ~]$ vim /opt/huawei/install/data/dn/postgresql.conf
	## 并将原本的password_encryption_type = 2更换为password_encryption_type = 0(使用md5加密)。         
	#Password storage type, 0 is md5 for PG, 1 is sha256 + md5, 2 is sha256 only
	password_encryption_type = 0
```

#### 8.2 添加白名单

对`pg_hba.conf`进行修改,如果不考虑安全性，也可以用0.0.0.0对所有IP地址进行开放，并将后面的trust替换成md5加密方式。

```shell
[omm@opengauss-201 ~]$ vim /opt/huawei/install/data/dn/pg_hba.conf
	# Allow replication connections from localhost, by a user with the
	# replication privilege.
	#local   replication     omm                                trust
	#host    replication     omm        127.0.0.1/32            trust
	#host    replication     omm        ::1/128                 trust
	host     all             all        192.168.1.201/32        md5

## 停止服务
[omm@opengauss-201 ~]$ gs_om -t stop

## 启动服务
[omm@opengauss-201 ~]$ gs_om -t start
```

#### 8.3 创建远程登录用户

```sql
## 超级管理员登录
[omm@opengauss-201 ~]$ gsql -d postgres -p 15400

## 创建数据库
openGauss=> CREATE DATABASE malodb WITH ENCODING 'UTF-8' template = template0;

## 切换数据库
openGauss=# \c malodb

## 新增用户(用户跟Schema关系我们后续章节再专门介绍)
malodb=# create user malo_auth with password "Gauss@123"; 

## 将malodb数据库下malo_auth schema 的表都授权于 malo
malodb=# grant all privileges on all tables in schema malo_auth to malo_auth;

## 将malodb数据库授权给malo用户
malodb=# grant all privileges on database malodb to malo_auth;

## 解决部分客户端(JetBrains DataGrid报错)
malodb=# GRANT ALL PRIVILEGES ON pg_user TO malo_auth; 

## 退出
malodb=# \q

## 新用户登录数据库
[omm@opengauss-201 ~]$ gsql -U malo -p 15400 -d testdb
	Password for user malo: 
	gsql ((openGauss 3.0.0 build 02c14696) compiled at 2022-04-01 18:12:34 commit 0 last mr  )
	Non-SSL connection (SSL connection is recommended when requiring high-security)
	Type "help" for help.

	testdb=> 
```

### 附录

| 序号 |     类型     | 目录                        | 说明 |
| :--: | :----------: | :-------------------------- | :--: |
|  1   | 软件安装目录 | /opt/huawei/install/app     |      |
|  2   | 数据存放目录 | /opt/huawei/install/data/dn |      |
|  3   | 日志存放目录 | /var/log/omm                |      |
|  4   | 系统工具目录 | /opt/huawei/install/om      |      |
|  5   | core文件目录 | /opt/huawei/corefile        |      |
|  6   | 临时文件目录 | /opt/huawei/tmp             |      |
