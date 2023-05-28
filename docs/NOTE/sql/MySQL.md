# MySQL

主要记录在Python下使用数据库的经历。

## 安装 MySQL

我安装的是MySQL 8.0.29 （macOS Monterey 12.2.1），官网下载即可。



## 添加配置文件

/usr/local/mysql/etc/my.cnf

```bash
[mysqld]

port = 3306

basedir=/usr/local/mysql

datadir=/usr/local/mysql/data
```



## 启动/关闭 MySQL

1. 安装完成后，可以使用访达 -> 系统偏好设置中的MySQL点击 Start/Stop MySQL Server进行启动和关闭。

2. 使用MySQL的脚本程序/usr/local/mysql-8.0.29-macos12-x86_64/support-files/mysql.server。终端中输入

   ```bash
   #  启动
   $ sudo /usr/local/mysql-8.0.29-macos12-x86_64/support-files/mysql.server start
   
   # 关闭
   $ sudo /usr/local/mysql-8.0.29-macos12-x86_64/support-files/mysql.server stop
   
   
   # 添加环境变量后，可以简化命令，方法：
   
   # 使用命令打开环境变量文件
   $ vim ~/.bash_profile
   
   # 插入下行text
   export PATH=${PATH}:/usr/local/mysql/support-files/
   
   # 以后就可以直接
   $ sudo mysql.server start
   $ sudo mysql.server stop
   ```

   

## PyMySQL

### 安装

```
pip install PyMySQL
```

### 创建数据库

```sql
$ mysql -u root -p

-- terminal 登录 root 后可使用下行语句进行创建（1.不存在则创建；2.默认编码utf-8）
mysql> CREATE DATABASE IF NOT EXISTS yourDataBaseName DEFAULT CHARSET utf8 COLLATE utf8_general_ci;

-- 选择数据库，并创建表
mysql> use socketLearn;
Database changed

 mysql> create table users(
     id int auto_increment primary key,  -- 主键 & 自增
     username varchar(8) not null,      -- 最大长度为8个字符 & 不允许为空
     password varchar(32) not null,
     dpk varchar(256)
)default charset=utf8;

mysql> desc users;
+----------+--------------+------+-----+---------+----------------+
| Field    | Type         | Null | Key | Default | Extra          |
+----------+--------------+------+-----+---------+----------------+
| id       | int          | NO   | PRI | NULL    | auto_increment |
| username | varchar(8)   | NO   |     | NULL    |                |
| password | varchar(32)  | NO   |     | NULL    |                |
| dpk      | varchar(256) | YES  |     | NULL    |                |
+----------+--------------+------+-----+---------+----------------+
4 rows in set (0.01 sec)
```

### 插入数据

试着插入几条测试数据

```sql
mysql> insert into users(username, password) values("Alice", "123");
Query OK, 1 row affected (0.01 sec)

mysql> insert into users(username, password) values("Bob", "123");
Query OK, 1 row affected (0.00 sec)
```













## 有关 Error

---

1. `RuntimeError: 'cryptography' package is required for sha256_password or caching_sha2_password auth methods`

突然，MySQL登录不了了。上面是pycharm的报错。

该错误原因是需要cryptography包，`pip install cryptography`就可以了。

----



