> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/jhno1/p/14980599.html)

作者：[@青青子衿悠悠我心](https://home.cnblogs.com/u/jhno1/)  
本文为作者原创，转载请注明出处：[https://www.cnblogs.com/jhno1/p/14980599.html](https://www.cnblogs.com/jhno1/p/14980599.html)

一、 SQLite 版本过低
--------------

```
#1.报错信息：
[root@stg-airflow001 ~]$ airflow db init
Traceback (most recent call last):
  File "/usr/local/python3/bin/airflow", line 5, in <module>
    from airflow.__main__ import main
  File "/usr/local/python3.8.6/lib/python3.8/site-packages/airflow/__init__.py", line 34, in <module>
    from airflow import settings
  File "/usr/local/python3.8.6/lib/python3.8/site-packages/airflow/settings.py", line 35, in <module>
    from airflow.configuration import AIRFLOW_HOME, WEBSERVER_CONFIG, conf  # NOQA F401
  File "/usr/local/python3.8.6/lib/python3.8/site-packages/airflow/configuration.py", line 1115, in <module>
    conf = initialize_config()
  File "/usr/local/python3.8.6/lib/python3.8/site-packages/airflow/configuration.py", line 877, in initialize_config
    conf.validate()
  File "/usr/local/python3.8.6/lib/python3.8/site-packages/airflow/configuration.py", line 202, in validate
    self._validate_config_dependencies()
  File "/usr/local/python3.8.6/lib/python3.8/site-packages/airflow/configuration.py", line 242, in _validate_config_dependencies
    raise AirflowConfigException(
airflow.exceptions.AirflowConfigException: error: sqlite C library version too old (< 3.15.0). See https://airflow.apache.org/docs/apache-airflow/2.1.0/howto/set-up-database.rst#setting-up-a-sqlite-database

#2.升级SQLite
从https://sqlite.org/下载源代码，在本地制作并安装。
1）下载源码
[root@stg-airflow001 ~]$ wget https://www.sqlite.org/2019/sqlite-autoconf-3290000.tar.gz

2） 编译
[root@stg-airflow001 ~]$ tar zxvf sqlite-autoconf-3290000.tar.gz 
[root@stg-airflow001 ~]$ cd sqlite-autoconf-3290000/
[root@stg-airflow001 ~/sqlite-autoconf-3290000]$ ./configure --prefix=/usr/local
[root@stg-airflow001 ~/sqlite-autoconf-3290000]$ make && make install

3）替换系统低版本 sqlite3
[root@stg-airflow001 ~/sqlite-autoconf-3290000]$ cd 
[root@stg-airflow001 ~]$ mv /usr/bin/sqlite3  /usr/bin/sqlite3_old
[root@stg-airflow001 ~]$ ln -s /usr/local/bin/sqlite3   /usr/bin/sqlite3
[root@stg-airflow001 ~$ echo "/usr/local/lib" > /etc/ld.so.conf.d/sqlite3.conf
[root@stg-airflow001 ~]$ ldconfig
[root@stg-airflow001 ~]$ sqlite3 -version
3.29.0 2019-07-10 17:32:03 fc82b73eaac8b36950e527f12c4b5dc1e147e6f4ad2217ae43ad82882a88bfa6

#3.airflow官网升级SQLite方案
先决条件：您将需要wget, tar, gzip,``gcc`` make, 和expect来使升级过程正常工作。
yum -y install wget tar gzip gcc make expect

从https://sqlite.org/下载源代码，在本地制作并安装。
wget https://www.sqlite.org/src/tarball/sqlite.tar.gz
tar xzf sqlite.tar.gz
cd sqlite/
export CFLAGS="-DSQLITE_ENABLE_FTS3 \
    -DSQLITE_ENABLE_FTS3_PARENTHESIS \
    -DSQLITE_ENABLE_FTS4 \
    -DSQLITE_ENABLE_FTS5 \
    -DSQLITE_ENABLE_JSON1 \
    -DSQLITE_ENABLE_LOAD_EXTENSION \
    -DSQLITE_ENABLE_RTREE \
    -DSQLITE_ENABLE_STAT4 \
    -DSQLITE_ENABLE_UPDATE_DELETE_LIMIT \
    -DSQLITE_SOUNDEX \
    -DSQLITE_TEMP_STORE=3 \
    -DSQLITE_USE_URI \
    -O2 \
    -fPIC"
export PREFIX="/usr/local"
LIBS="-lm" ./configure --disable-tcl --enable-shared --enable-tempstore=always --prefix="$PREFIX"
make
make install

安装后添加/usr/local/lib到库路径
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH


```

二、mysqlclient 找不到
-----------------

```
#1.报错信息：
[root@stg-airflow001 ~]$ airflow db init
Traceback (most recent call last):
  File "/usr/local/python-3.8.6b/python3.8/site-packages/MySQLdb/__init__.py", line 18, in <module>
    from . import _mysql
ImportError: libmysqlclient.so.21: cannot open shared object file: No such file or directory

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/local/python3/bin/airflow", line 5, in <module>
    from airflow.__main__ import main
  File "/usr/local/python-3.8.6b/python3.8/site-packages/airflow/__init__.py", line 46, in <module>
    settings.initialize()
  File "/usr/local/python-3.8.6b/python3.8/site-packages/airflowttings.py", line 447, in initialize
    configure_adapters()
  File "/usr/local/python-3.8.6b/python3.8/site-packages/airflowttings.py", line 327, in configure_adapters
    import MySQLdb.converters
  File "/usr/local/python-3.8.6b/python3.8/site-packages/MySQLdb/__init__.py", line 24, in <module>
    version_info, _mysql.version_info, _mysql.__file__
NameError: name '_mysql' is not defined

#2.报错解决
1）查看libmysqlclient位置
[root@stg-airflow001 ~]$ find / -name "libmysqlclient.so.*"
/root/mysql-8.0.24dbrary_output_directorybmysqlclient.so.21
/usr/local/mysql/lib/libmysqlclient.so.21
或者
/usr/local/mysql/lib/libmysqlclient.so.18

[root@stg-airflow001 ~]$ ls -ld /usr/lib64/
dr-xr-xr-x. 50 root root 36864 Dec 23 03:40 /usr/lib64/

2）做软连接
[root@stg-airflow001 ~]$ ln -s /usr/local/mysql/lib/libmysqlclient.so.21 /usr/lib64/libmysqlclient.so.21
或者
[root@stg-airflow001 ~]$ ln -s /usr/local/mysql/lib/libmysqlclient.so.18 /usr/lib64/libmysqlclient.so.18 

3）导入模块
[root@m01 ~]# python3
Python 3.8.6 (default, Jun 23 2021, 18:48:14) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import MySQLdb 
>>> exit()


```

三、模块 pymysql 未找到
----------------

```
#1.报错信息
Broken DAG: [/root/airflow/dags/aiinsight_sdata_job.py] Traceback (most recent call last):
  File "/root/airflow/dags/utils/getFiles.py", line 5, in <module>
    from conf.mysql_client import get_db
  File "/root/airflow/dags/conf/mysql_client.py", line 6, in <module>
    import pymysql
ModuleNotFoundError: No module named 'pymysql'

#2.报错解决
root@stg-airflow001 ~]$ pip3 install  pymysql
Collecting pymysql
  Using cached PyMySQL-1.0.2-py3-none-any.whl (43 kB)
Installing collected packages: pymysql
Successfully installed pymysql-1.0.2
WARNING: Running pip as root will break packages and permissions. You should install packages reliably by using venv: https://pip.pypa.io/warnings/venv



```

四、模块 sshtunnel 未找到
------------------

```
#1.报错信息
Broken DAG: [/root/airflow/dags/lebel_crm_table_etl.py] Traceback (most recent call last):
  File "/root/airflow/dags/lebel/CRM_WECHAT_FANS_BASIC_INFO.py", line 8, in <module>
    from conf import config
  File "/root/airflow/dags/conf/config.py", line 7, in <module>
    from sshtunnel import SSHTunnelForwarder
ModuleNotFoundError: No module named 'sshtunnel'

#2.报错解决
[root@stg-airflow001 ~]$ pip3 install  sshtunnel
Collecting sshtunnel
  Downloading sshtunnel-0.4.0-py2.py3-none-any.whl (24 kB)
Collecting paramiko>=2.7.2
  Using cached paramiko-2.7.2-py2.py3-none-any.whl (206 kB)
Collecting pynacl>=1.0.1
  Downloading PyNaCl-1.4.0-cp35-abi3-manylinux1_x86_64.whl (961 kB)
     |████████████████████████████████| 961 kB 46 kB/s 
Collecting bcrypt>=3.1.3
  Using cached bcrypt-3.2.0-cp36-abi3-manylinux2010_x86_64.whl (63 kB)
Requirement already satisfied: cryptography>=2.5 in /usr/local/python3.8.6/lib/python3.8/site-packages (from paramiko>=2.7.2->sshtunnel) (3.4.7)
Requirement already satisfied: cffi>=1.1 in /usr/local/python3.8.6/lib/python3.8/site-packages (from bcrypt>=3.1.3->paramiko>=2.7.2->sshtunnel) (1.14.5)
Requirement already satisfied: six>=1.4.1 in /usr/local/python3.8.6/lib/python3.8/site-packages (from bcrypt>=3.1.3->paramiko>=2.7.2->sshtunnel) (1.16.0)
Requirement already satisfied: pycparser in /usr/local/python3.8.6/lib/python3.8/site-packages (from cffi>=1.1->bcrypt>=3.1.3->paramiko>=2.7.2->sshtunnel) (2.20)
Installing collected packages: pynacl, bcrypt, paramiko, sshtunnel
Successfully installed bcrypt-3.2.0 paramiko-2.7.2 pynacl-1.4.0 sshtunnel-0.4.0
WARNING: Running pip as root will break packages and permissions. You should install packages reliably by using venv: https://pip.pypa.io/warnings/venv


```

五、模块 gunicorn 找不到
-----------------

```
#1.报错信息
6月 28 11:06:50 stg-airflow001 airflow[24153]: FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn'

#2.报错解决
[root@stg-airflow001 ~/airflow]$ which  gunicorn
/usr/local/python3/bin/gunicorn

[root@stg-airflow001 ~/airflow]$ ln -fs /usr/local/python3.8.6/bin/gunicorn /bin/gunicorn


```

```
如果您觉得阅读本文对您有帮助，请点一下“推荐”按钮，您的“推荐”将是我最大的写作动力！欢迎各位转载，但是未经作者本人同意，转载文章之后必须在文章页面明显位置给出作者和原文连接，否则保留追究法律责任的权利。

```