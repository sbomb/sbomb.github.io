# PostgreSQL源码安装

## 源码编译

环境准备

```sh
yum install -y gcc *xml2* *readline*dev* *zlib* *perl*dev* *python*dev* bison flexs
```



编译

```sh
./configure --prefix=/path
make
make install
adduser postgres
mkdir /usr/local/pgsql/data
chown postgres /usr/local/pgsql/data                             # 授权postgres拥有data目录
su - postgres                                                          # 切换用户
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data                   # 初始化数据库
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start  # 启动数据库
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l logfile stop   # 停止数据库
```



## 数据库自动启动

1. 自动启动

   `vi /etc/rc.d/rc.local`

   添加：

   `su - postgres -c 'pg_ctl start -D /postgres/data -l logfile'为该文件添加执行权限chmod +x /etc/rc.d/rc.local`

2.  使用systemctl

   ```sh
   [Unit]
   Description=postgresql
   After=network.target
   	
   [Service]
   Type=forking
   User=postgres
   Group=postgres
   ExecStart=/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data  start 
   ExecReload=/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data  restart 
   ExecStop=/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data  stop 
   PrivateTmp=true
   	
   [Install]
   WantedBy=multi-user.target
   ```

   PS:
   
   在设置自启动时，主要 -l logfile 参数，并不是postgrel的变量，所以，此时自启动的脚本无法自动启动。



## 修改配置文件

在PGDATA目录下生成配置文件pg_hba.conf和postgresql.conf 

其中：

postgresql.conf文件主要是配置文件

主要修改监听地址和端口

listen_address="*"

port=5432

pg_hba.conf文件为授权文件，控制用户访问方式

```
# "local" is for Unix domain socket connections only
local all all trust
# IPv4 local connections:
host all all 127.0.0.1/32 trust
# IPv6 local connections:
host all all ::1/128 trust
```

## 修改用户密码

```
[postgres@localhost postgresql-9.6beta1]$ /opt/pg9.6/bin/psql -U postgres
psql (9.6beta1)
Type "help" for help.

postgres=# alter user postgres with password 'password';
ALTER ROLE
postgres=# \q
```

## 创建用户

```sh
create user testUser WITH PASSWORD '*****'   ;				#创建用户
#将数据库testDB授权给testUser;
GRANT ALL PRIVILEGES ON DATABASE testDB TO testUser ;				
#将数据库中的所有表都授权给该用户，如果无此授权，则当前用户无读写权限
GRANT ALL PRIVILEGES ON all tables in schema public TO testUser ;	

## 授权某个表
GRANT SELECT ON TABLE mytable TO testUser;					#授权查询
```

## 创建数据库

```sh
#创建数据库，如demo：
CREATE DATABASE testDB OWNER testUser;
# 将demo数据库的所有权限都赋予tom用户：
GRANT ALL PRIVILEGES ON DATABASE testDB TO testUser;
```

