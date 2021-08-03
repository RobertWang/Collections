> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/xy-ouyang/p/12894955.html)

目录：

1、[centos7 安装 apache-airflow](#title1)  
　　1.1、[下载安装 centos7](#title1.1)  
　　1.2、[centos7 安装 python3](#title1.2)  
　　1.3、[centos7 安装 mysql](#title1.3)  
　　1.4、[安装 airflow](#title1.4)  
　　1.5、[初始化数据库表（默认使用本地的 sqlite 数据库）](#title1.5)  
　　1.6、[启动 airflow webserver](#title1.6)  
2、[airflow 安装可能遇到的问题](#title2)  
　　2.1、[pip 安装 airflow 依赖时报版本不对的解决](#title2.1)  
　　2.2、[解决 Python.h：No such file or directory](#title2.2)  
　　2.3、[启动 airflow webserver 报错：FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn'](#title2.3)

1、centos7 安装 apache-airflow    [<-- 返回目录](#content)
---------------------------------------------------

### 1.1、下载安装 centos7    [<-- 返回目录](#content)

[](https://www.cnblogs.com/xy-ouyang/p/12951109.html)　　参考：[vmware 安装 centos7 及网络配置](https://www.cnblogs.com/xy-ouyang/p/12951109.html)[](https://www.cnblogs.com/xy-ouyang/p/12951109.html)

### 1.2、centos7 安装 python3    [<-- 返回目录](#content)

[](https://www.cnblogs.com/xy-ouyang/p/12957249.html)　　参考：[Centos7 安装 python3 并与 python2 共存, 以及安装 pip(pip3)](https://www.cnblogs.com/xy-ouyang/p/12957249.html)[](https://www.cnblogs.com/xy-ouyang/p/12957249.html)

　　安装相关依赖

```
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel

```

　　./configure --prefix=/usr/local/python3.6.8 报错

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527110206201-664471190.png)

　　安装 gcc 编译环境：yum -y install gcc

　　查看 gcc 版本

 ![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527110629925-812061112.png)

　　开始安装

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
# 下载解压
cd /opt
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
tar -xvf Python-3.6.8.tgz
mv Python-3.6.8 python-3.6.8 # 在local目录下创建python3.6.8目录
mkdir /usr/local/python3.6.8
# 配置安装目录
cd /opt/python-3.6.8
./configure --prefix=/usr/local/python3.6.8
# 编译安装
make && make install
# 配置python3软连接
ln -s /usr/local/python3.6.8/bin/python3  /usr/bin/python3
# 配置pip3软连接
ln -s /usr/local/python3.6.8/bin/pip3 /usr/bin/pip3
# 升级pip
pip3 install --upgrade pip -i https://pypi.douban.com/simple

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
# 查看已安装的包
pip3 freeze
pip3 list
# 检查哪些包需要更新
pip3 list outdated

```

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527111307503-1830092300.png)

### 1.3、centos7 安装 mysql    [<-- 返回目录](#content)

参考：[CentOS7 安装 MySQL（完整版）](https://blog.csdn.net/qq_36582604/article/details/80526287)

　　查看编码

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527114128419-778744497.png)

### 为 firewalld 添加开放 mysql 端口 3306 和 airflow webserver 8080

### [root@localhost ~]# firewall-cmd --zone=public --add-port=3306/tcp --permanent

重新载入

```
[root@localhost ~]# firewall-cmd --reload

```

telnet 查看 centos7 的 3306 端口是否开放

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527114541904-171706441.png)

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527114656165-367075441.png)

### 1.4、安装 airflow    [<-- 返回目录](#content)

配置 AIRFLOW_HOME

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527122621548-79362154.png)

 配置生效: source /etc/profile

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527122837735-2062608980.png)

开始安装

```
pip3 install apache-airflow -i https://pypi.douban.com/simple

```

安装日志

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@localairflow ~]# pip3 install apache-airflow -i https://pypi.douban.com/simple
Looking in indexes: https://pypi.douban.com/simple
Collecting apache-airflow
  Downloading https://pypi.doubanio.com/packages/f7/32/600475e46b90f983ff093187425725cad9f3303665161328cd17d9ba974c/apache_airflow-1.10.10-py2.py3-none-any.whl (4.7MB)
    100% |????????????????????????????????| 4.7MB 637kB/s 
Collecting zope.deprecation<5.0,>=4.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/f9/26/b935bbf9d27e898b87d80e7873a0200cebf239253d0afe7a59f82fe90fff/zope.deprecation-4.4.0-py2.py3-none-any.whl
Collecting lazy-object-proxy~=1.3 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/0b/dd/b1e3407e9e6913cf178e506cd0dee818e58694d9a5cd1984e3f6a8b9a10f/lazy_object_proxy-1.4.3-cp36-cp36m-manylinux1_x86_64.whl (55kB)
    100% |????????????????????????????????| 61kB 1.0MB/s 
Collecting tenacity==4.12.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/75/1b/46a6a7b7c2b16811665ea09b7e63e7e6b7f9b5dedf2d0ba67e029668403c/tenacity-4.12.0-py2.py3-none-any.whl
Collecting flask-caching<1.4.0,>=1.3.3 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/88/f1/7bbd68a4d79eb53c33863a17926f14268749b897c8a77ab589f2e9117d47/Flask_Caching-1.3.3-py2.py3-none-any.whl
Collecting requests<3,>=2.20.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/1a/70/1935c770cb3be6e3a8b78ced23d7e0f3b187f5cbfab4749523ed65d7c9b1/requests-2.23.0-py2.py3-none-any.whl (58kB)
    100% |????????????????????????????????| 61kB 1.2MB/s 
Collecting colorlog==4.0.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/68/4d/892728b0c14547224f0ac40884e722a3d00cb54e7a146aea0b3186806c9e/colorlog-4.0.2-py2.py3-none-any.whl
Collecting pendulum==1.4.4 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/30/47/02f04abed54918d2a3f1da602a8254247670b2e1a99b4b1f02734a27e71e/pendulum-1.4.4-cp36-cp36m-manylinux1_x86_64.whl (123kB)
    100% |????????????????????????????????| 133kB 670kB/s 
Collecting attrs~=19.3 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/a2/db/4313ab3be961f7a763066401fb77f7748373b6094076ae2bda2806988af6/attrs-19.3.0-py2.py3-none-any.whl
Collecting funcsigs<2.0.0,>=1.0.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/69/cb/f5be453359271714c01b9bd06126eaf2e368f1fddfff30818754b5ac2328/funcsigs-1.0.2-py2.py3-none-any.whl
Collecting pygments<3.0,>=2.0.1 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/2d/68/106af3ae51daf807e9cdcba6a90e518954eb8b70341cee52995540a53ead/Pygments-2.6.1-py3-none-any.whl (914kB)
    100% |????????????????????????????????| 921kB 1.1MB/s 
Collecting flask-admin==1.5.4 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/03/4e/a92e3c5cee3fce68fda877b437ed4657df3a0168091f6d2cab6c94d932e3/Flask-Admin-1.5.4.tar.gz (1.7MB)
    100% |????????????????????????????????| 1.7MB 1.1MB/s 
Collecting flask-login<0.5,>=0.3 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c1/ff/bd9a4d2d81bf0c07d9e53e8cd3d675c56553719bbefd372df69bf1b3c1e4/Flask-Login-0.4.1.tar.gz
Collecting flask-wtf<0.15,>=0.14.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/36/a9/8c01171066bd7a524ee005d81bb4a8aa446ab178043a1ad6cb5dc8f0bd83/Flask_WTF-0.14.3-py2.py3-none-any.whl
Collecting jinja2<2.11.0,>=2.10.1 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/65/e0/eb35e762802015cab1ccee04e8a277b03f1d8e53da3ec3106882ec42558b/Jinja2-2.10.3-py2.py3-none-any.whl (125kB)
    100% |????????????????????????????????| 133kB 1.6MB/s 
Collecting flask-swagger==0.2.13 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/68/97/4e31ac3dc4a44a4b7487eab8404a68c871b57a15811e189862d0bf0c5b55/flask-swagger-0.2.13.tar.gz
Collecting setproctitle<2,>=1.1.8 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/5a/0d/dc0d2234aacba6cf1a729964383e3452c52096dc695581248b548786f2b3/setproctitle-1.1.10.tar.gz
Collecting croniter<0.4,>=0.3.17 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/56/02/94e5b63bb6c287fbda8a5693f15898a2b6f56cbdfaf5a5c1c0109d18d062/croniter-0.3.31-py2.py3-none-any.whl
Collecting termcolor==1.1.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/8a/48/a76be51647d0eb9f10e2a4511bf3ffb8cc1e6b14e9e4fab46173aa79f981/termcolor-1.1.0.tar.gz
Collecting configparser<3.6.0,>=3.5.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/55/c0/e0206081eaad646c6f5e2dc266edf457110d9031b363518d3264880e675d/configparser-3.5.3-py3-none-any.whl
Collecting typing-extensions>=3.7.4; python_version < "3.8" (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/0c/0e/3f026d0645d699e7320b59952146d56ad7c374e9cd72cd16e7c74e657a0f/typing_extensions-3.7.4.2-py3-none-any.whl
Collecting psutil<6.0.0,>=4.2.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c4/b8/3512f0e93e0db23a71d82485ba256071ebef99b227351f0f5540f744af41/psutil-5.7.0.tar.gz (449kB)
    100% |????????????????????????????????| 450kB 3.9MB/s 
Collecting pandas<1.0.0,>=0.17.1 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/52/3f/f6a428599e0d4497e1595030965b5ba455fd8ade6e977e3c819973c4b41d/pandas-0.25.3-cp36-cp36m-manylinux1_x86_64.whl (10.4MB)
    100% |????????????????????????????????| 10.4MB 768kB/s 
Collecting python-daemon<2.2,>=2.1.1 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/f4/59/816004688f8e8602526553cd96226f34657ce4a86daa2240c3eebb0568a3/python_daemon-2.1.2-py2.py3-none-any.whl
Collecting json-merge-patch==0.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/39/62/3b783faabac9a099877397d8f7a7cc862a03fbf9fb1b90d414ea7c6bb096/json-merge-patch-0.2.tar.gz
Collecting sqlalchemy~=1.3 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/b3/f5/c929eeb441efcb04eb7c5b6461500427e5dfab9bf980da165bc5f85cf659/SQLAlchemy-1.3.17-cp36-cp36m-manylinux1_x86_64.whl (1.2MB)
    100% |????????????????????????????????| 1.2MB 1.6MB/s 
Collecting alembic<2.0,>=1.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/60/1e/cabc75a189de0fbb2841d0975243e59bde8b7822bacbb95008ac6fe9ad47/alembic-1.4.2.tar.gz (1.1MB)
    100% |????????????????????????????????| 1.1MB 2.0MB/s 
  Installing build dependencies ... done
Collecting future<0.19,>=0.16.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/45/0b/38b06fd9b92dc2b68d58b75f900e97884c45bedd2ff83203d933cf5851c9/future-0.18.2.tar.gz (829kB)
    100% |????????????????????????????????| 829kB 2.0MB/s 
Collecting iso8601>=0.1.12 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/ef/57/7162609dab394d38bbc7077b7ba0a6f10fb09d8b7701ea56fa1edc0c4345/iso8601-0.1.12-py2.py3-none-any.whl
Collecting tzlocal<2.0.0,>=1.4 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/cb/89/e3687d3ed99bc882793f82634e9824e62499fdfdc4b1ae39e211c5b05017/tzlocal-1.5.1.tar.gz
Collecting dill<0.4,>=0.2.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c7/11/345f3173809cea7f1a193bfbf02403fff250a3360e0e118a1630985e547d/dill-0.3.1.1.tar.gz (151kB)
    100% |????????????????????????????????| 153kB 2.5MB/s 
Collecting sqlalchemy-jsonfield~=0.9; python_version >= "3.5" (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/d3/70/f636dd6f9e6bd38e64ba398a24c717af141d9771ccfd79b174b989d5a624/SQLAlchemy_JSONField-0.9.0-py2.py3-none-any.whl
Collecting text-unidecode==1.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/79/42/d717cc2b4520fb09e45b344b1b0b4e81aa672001dd128c180fabc655c341/text_unidecode-1.2-py2.py3-none-any.whl (77kB)
    100% |????????????????????????????????| 81kB 2.0MB/s 
Collecting argcomplete~=1.10 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/82/7d/455e149c28c320044cb763c23af375bd77d52baca041f611f5c2b4865cf4/argcomplete-1.11.1-py2.py3-none-any.whl
Collecting markdown<3.0,>=2.5.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/6d/7d/488b90f470b96531a3f5788cf12a93332f543dbab13c423a5e7ce96a0493/Markdown-2.6.11-py2.py3-none-any.whl (78kB)
    100% |????????????????????????????????| 81kB 3.3MB/s 
Collecting jsonschema~=3.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c5/8f/51e89ce52a085483359217bc72cdbf6e75ee595d5b1d4b5ade40c7e018b8/jsonschema-3.2.0-py2.py3-none-any.whl (56kB)
    100% |????????????????????????????????| 61kB 2.2MB/s 
Collecting python-dateutil<3,>=2.3 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/d4/70/d60450c3dd48ef87586924207ae8907090de0b306af2bce5d134d78615cb/python_dateutil-2.8.1-py2.py3-none-any.whl (227kB)
    100% |????????????????????????????????| 235kB 2.5MB/s 
Collecting flask-appbuilder~=2.2; python_version >= "3.6" (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/e8/3e/52fef38e20829f105c0dd7137aa319530f94449f7dfd1c70c0b2296c917e/Flask_AppBuilder-2.3.4-py3-none-any.whl (1.7MB)
    100% |????????????????????????????????| 1.7MB 545kB/s 
Collecting graphviz>=0.12 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/83/cc/c62100906d30f95d46451c15eb407da7db201e30f42008f3643945910373/graphviz-0.14-py2.py3-none-any.whl
Collecting cached-property~=1.5 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/3b/86/85c1be2e8db9e13ef9a350aecd6dea292bd612fa288c2f40d035bb750ded/cached_property-1.5.1-py2.py3-none-any.whl
Collecting unicodecsv>=0.14.1 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/6f/a4/691ab63b17505a26096608cc309960b5a6bdf39e4ba1a793d5f9b1a53270/unicodecsv-0.14.1.tar.gz
Collecting gunicorn<20.0,>=19.5.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/5f/54/c15f2c243c19074cbf06ce6c48732d99aec825487f87e57e86e9a22990f2/gunicorn-19.10.0-py2.py3-none-any.whl (113kB)
    100% |????????????????????????????????| 122kB 1.3MB/s 
Collecting tabulate<0.9,>=0.7.5 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c4/f4/770ae9385990f5a19a91431163d262182d3203662ea2b5739d0fcfc080f1/tabulate-0.8.7-py3-none-any.whl
Collecting werkzeug<1.0.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c2/e4/a859d2fe516f466642fa5c6054fd9646271f9da26b0cac0d2f37fc858c8f/Werkzeug-0.16.1-py2.py3-none-any.whl (327kB)
    100% |????????????????????????????????| 327kB 991kB/s 
Collecting thrift>=0.9.2 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/97/1e/3284d19d7be99305eda145b8aa46b0c33244e4a496ec66440dac19f8274d/thrift-0.13.0.tar.gz (59kB)
    100% |????????????????????????????????| 61kB 1.9MB/s 
Collecting cattrs~=0.9 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/0e/4f/165edbb76d21cf95a0c9003df85cfaca419d64ede766498201ef720d3104/cattrs-0.9.2-py2.py3-none-any.whl (61kB)
    100% |????????????????????????????????| 61kB 1.4MB/s 
Collecting flask<2.0,>=1.1.0 (from apache-airflow)
  Downloading https://pypi.doubanio.com/packages/f2/28/2a03252dfb9ebf377f40fba6a7841b47083260bf8bd8e737b0c6952df83f/Flask-1.1.2-py2.py3-none-any.whl (94kB)
    100% |????????????????????????????????| 102kB 1.6MB/s 
Requirement already satisfied: setuptools in /usr/local/python3.6.8/lib/python3.6/site-packages (from zope.deprecation<5.0,>=4.0->apache-airflow) (40.6.2)
Collecting six>=1.9.0 (from tenacity==4.12.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/ee/ff/48bde5c0f013094d729fe4b0316ba2a24774b3ff1c52d924a8a4cb04078a/six-1.15.0-py2.py3-none-any.whl
Collecting certifi>=2017.4.17 (from requests<3,>=2.20.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/57/2b/26e37a4b034800c960a00c4e1b3d9ca5d7014e983e6e729e33ea2f36426c/certifi-2020.4.5.1-py2.py3-none-any.whl (157kB)
    100% |????????????????????????????????| 163kB 1.4MB/s 
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 (from requests<3,>=2.20.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/e1/e5/df302e8017440f111c11cc41a6b432838672f5a70aa29227bf58149dc72f/urllib3-1.25.9-py2.py3-none-any.whl (126kB)
    100% |????????????????????????????????| 133kB 1.4MB/s 
Collecting chardet<4,>=3.0.2 (from requests<3,>=2.20.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl (133kB)
    100% |????????????????????????????????| 143kB 5.7MB/s 
Collecting idna<3,>=2.5 (from requests<3,>=2.20.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/89/e3/afebe61c546d18fb1709a61bee788254b40e736cff7271c7de5de2dc4128/idna-2.9-py2.py3-none-any.whl (58kB)
    100% |????????????????????????????????| 61kB 10.1MB/s 
Collecting pytzdata>=2018.3.0.0 (from pendulum==1.4.4->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/7f/f9/cdd39831b0465285c02ed90cfbf0db25bb951cb2a35ded0e69222375bea3/pytzdata-2019.3-py2.py3-none-any.whl (489kB)
    100% |????????????????????????????????| 491kB 5.2MB/s 
Collecting wtforms (from flask-admin==1.5.4->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/f0/1b/af089b3d54955e0a3b0045ddd40773c2ef5dc7375ccffd09366ce469b755/WTForms-2.3.1-py2.py3-none-any.whl (169kB)
    100% |????????????????????????????????| 174kB 7.6MB/s 
Collecting itsdangerous (from flask-wtf<0.15,>=0.14.2->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from jinja2<2.11.0,>=2.10.1->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/b2/5f/23e0023be6bb885d00ffbefad2942bc51a620328ee910f64abe5a8d18dd1/MarkupSafe-1.1.1-cp36-cp36m-manylinux1_x86_64.whl
Collecting PyYAML>=3.0 (from flask-swagger==0.2.13->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/64/c2/b80047c7ac2478f9501676c988a5411ed5572f35d1beff9cae07d321512c/PyYAML-5.3.1.tar.gz (269kB)
    100% |????????????????????????????????| 276kB 8.2MB/s 
Collecting pytz>=2017.2 (from pandas<1.0.0,>=0.17.1->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/4f/a4/879454d49688e2fad93e59d7d4efda580b783c745fd2ec2a3adf87b0808d/pytz-2020.1-py2.py3-none-any.whl (510kB)
    100% |????????????????????????????????| 512kB 5.9MB/s 
Collecting numpy>=1.13.3 (from pandas<1.0.0,>=0.17.1->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/03/27/e35e7c6e6a52fab9fcc64fc2b20c6b516eba930bb02b10ace3b38200d3ab/numpy-1.18.4-cp36-cp36m-manylinux1_x86_64.whl (20.2MB)
    100% |????????????????????????????????| 20.2MB 638kB/s 
Collecting docutils (from python-daemon<2.2,>=2.1.1->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/81/44/8a15e45ffa96e6cf82956dd8d7af9e666357e16b0d93b253903475ee947f/docutils-0.16-py2.py3-none-any.whl (548kB)
    100% |????????????????????????????????| 552kB 1.2MB/s 
Collecting lockfile>=0.10 (from python-daemon<2.2,>=2.1.1->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c8/22/9460e311f340cb62d26a38c419b1381b8593b0bb6b5d1f056938b086d362/lockfile-0.12.2-py2.py3-none-any.whl
Collecting Mako (from alembic<2.0,>=1.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/50/78/f6ade1e18aebda570eed33b7c534378d9659351cadce2fcbc7b31be5f615/Mako-1.1.2-py2.py3-none-any.whl (75kB)
    100% |????????????????????????????????| 81kB 2.7MB/s 
Collecting python-editor>=0.3 (from alembic<2.0,>=1.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c6/d3/201fc3abe391bbae6606e6f1d598c15d367033332bd54352b12f35513717/python_editor-1.0.4-py3-none-any.whl
Collecting typing>=3.6; python_version < "3.7" (from sqlalchemy-jsonfield~=0.9; python_version >= "3.5"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/fe/2e/b480ee1b75e6d17d2993738670e75c1feeb9ff7f64452153cf018051cc92/typing-3.7.4.1-py3-none-any.whl
Collecting importlib-metadata<2,>=0.23; python_version == "3.6" (from argcomplete~=1.10->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/ad/e4/891bfcaf868ccabc619942f27940c77a8a4b45fd8367098955bb7e152fb1/importlib_metadata-1.6.0-py2.py3-none-any.whl
Collecting pyrsistent>=0.14.0 (from jsonschema~=3.0->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/9f/0d/cbca4d0bbc5671822a59f270e4ce3f2195f8a899c97d0d5abb81b191efb5/pyrsistent-0.16.0.tar.gz (108kB)
    100% |????????????????????????????????| 112kB 2.3MB/s 
Collecting email-validator<2,>=1.0.5 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/8b/f5/26dc56e8e5b3441e766c8c359be9a28d2355902ab8b2140a2d5988da675e/email_validator-1.1.1-py2.py3-none-any.whl
Collecting click<8,>=6.7 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/d2/3d/fa76db83bf75c4f8d338c2fd15c8d33fdd7ad23a9b5e57eb6c5de26b430e/click-7.1.2-py2.py3-none-any.whl (82kB)
    100% |????????????????????????????????| 92kB 2.3MB/s 
Collecting Flask-OpenID<2,>=1.2.5 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/d1/a2/9d1fba3287a65f81b9d1c09c4f7cb16f8ea4988b1bc97ffea0d60983338f/Flask-OpenID-1.2.5.tar.gz (43kB)
    100% |????????????????????????????????| 51kB 814kB/s 
Collecting PyJWT>=1.7.1 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/87/8b/6a9f14b5f781697e51259d81657e6048fd31a113229cf346880bb7545565/PyJWT-1.7.1-py2.py3-none-any.whl
Collecting marshmallow-sqlalchemy<1,>=0.16.1 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/80/b8/8419ae927fbd16bced8eb0640a171931260857efbf3b4acbadaa71400190/marshmallow_sqlalchemy-0.23.0-py2.py3-none-any.whl
Collecting Flask-SQLAlchemy<3,>=2.4 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/6a/a3/7b8427bb41aa9348670645932c31811b898684010c51fa8db0a82d56fbdb/Flask_SQLAlchemy-2.4.2-py2.py3-none-any.whl
Collecting Flask-JWT-Extended<4,>=3.18 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/7a/ee/bb1c679b44ebec5c64626d632186d890cd95fffb63c1bded85d979a80724/Flask-JWT-Extended-3.24.1.tar.gz
Collecting colorama<1,>=0.3.9 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c9/dc/45cdef1b4d119eb96316b3117e6d5708a08029992b2fee2c143c7a0a5cc5/colorama-0.4.3-py2.py3-none-any.whl
Collecting marshmallow<3.0.0,>=2.18.0 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/8d/5a/c8288b3fa34bfc5afeee56454db7c239b9d2f492c36172dafabf95c780af/marshmallow-2.21.0-py2.py3-none-any.whl (50kB)
    100% |????????????????????????????????| 51kB 3.5MB/s 
Collecting apispec[yaml]<2,>=1.1.1 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/38/7a/20500edb2e77b8a6f6b677ed6a7b6d97b696fcc477374d36d83883462863/apispec-1.3.3-py2.py3-none-any.whl
Collecting sqlalchemy-utils<1,>=0.32.21 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/aa/d3/f397b61a2eee34d023e7c1f2519b6e5a7058f3d857461eaa539cc38e1e85/SQLAlchemy-Utils-0.36.6.tar.gz (131kB)
    100% |????????????????????????????????| 133kB 1.8MB/s 
Collecting Flask-Babel<2,>=1 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/76/a4/0115c7c520125853037fc1d6b3da132a526949640e27a699a13e05ec7593/Flask_Babel-1.0.0-py3-none-any.whl
Collecting marshmallow-enum<2,>=1.4.1 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/c6/59/ef3a3dc499be447098d4a89399beb869f813fee1b5a57d5d79dee2c1bf51/marshmallow_enum-1.5.1-py2.py3-none-any.whl
Collecting prison<1.0.0,>=0.1.3 (from flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/56/3d/10c7dfd83613dadc8388bdef8e8000ccfb2984134050ce9bea6a00c94296/prison-0.1.3-py2.py3-none-any.whl
Collecting zipp>=0.5 (from importlib-metadata<2,>=0.23; python_version == "3.6"->argcomplete~=1.10->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/b2/34/bfcb43cc0ba81f527bc4f40ef41ba2ff4080e047acb0586b56b3d017ace4/zipp-3.1.0-py3-none-any.whl
Collecting dnspython>=1.15.0 (from email-validator<2,>=1.0.5->flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/ec/d3/3aa0e7213ef72b8585747aa0e271a9523e713813b9a20177ebe1e939deb0/dnspython-1.16.0-py2.py3-none-any.whl (188kB)
    100% |????????????????????????????????| 194kB 1.6MB/s 
Collecting python3-openid>=2.0 (from Flask-OpenID<2,>=1.2.5->flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/bd/de/52c5699f52dcee3037db587196dcaf63ffedf5fbeba3183afe9b21a3a89f/python3_openid-3.1.0-py3-none-any.whl (130kB)
    100% |????????????????????????????????| 133kB 2.0MB/s 
Collecting Babel>=2.3 (from Flask-Babel<2,>=1->flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/15/a1/522dccd23e5d2e47aed4b6a16795b8213e3272c7506e625f2425ad025a19/Babel-2.8.0-py2.py3-none-any.whl (8.6MB)
    100% |????????????????????????????????| 8.6MB 3.5MB/s 
Collecting defusedxml (from python3-openid>=2.0->Flask-OpenID<2,>=1.2.5->flask-appbuilder~=2.2; python_version >= "3.6"->apache-airflow)
  Downloading https://pypi.doubanio.com/packages/06/74/9b387472866358ebc08732de3da6dc48e44b0aacd2ddaa5cb85ab7e986a2/defusedxml-0.6.0-py2.py3-none-any.whl
Installing collected packages: zope.deprecation, lazy-object-proxy, six, tenacity, werkzeug, itsdangerous, MarkupSafe, jinja2, click, flask, flask-caching, certifi, urllib3, chardet, idna, requests, colorlog, python-dateutil, pytzdata, pytz, tzlocal, pendulum, attrs, funcsigs, pygments, wtforms, flask-admin, flask-login, flask-wtf, PyYAML, flask-swagger, setproctitle, croniter, termcolor, configparser, typing-extensions, psutil, numpy, pandas, docutils, lockfile, python-daemon, json-merge-patch, sqlalchemy, Mako, python-editor, alembic, future, iso8601, dill, typing, sqlalchemy-jsonfield, text-unidecode, zipp, importlib-metadata, argcomplete, markdown, pyrsistent, jsonschema, dnspython, email-validator, defusedxml, python3-openid, Flask-OpenID, PyJWT, marshmallow, marshmallow-sqlalchemy, Flask-SQLAlchemy, Flask-JWT-Extended, colorama, apispec, sqlalchemy-utils, Babel, Flask-Babel, marshmallow-enum, prison, flask-appbuilder, graphviz, cached-property, unicodecsv, gunicorn, tabulate, thrift, cattrs, apache-airflow
  Running setup.py install for tzlocal ... done
  Running setup.py install for flask-admin ... done
  Running setup.py install for flask-login ... done
  Running setup.py install for PyYAML ... done
  Running setup.py install for flask-swagger ... done
  Running setup.py install for setproctitle ... done
  Running setup.py install for termcolor ... done
  Running setup.py install for psutil ... done
  Running setup.py install for json-merge-patch ... done
  Running setup.py install for alembic ... done
  Running setup.py install for future ... done
  Running setup.py install for dill ... done
  Running setup.py install for pyrsistent ... done
  Running setup.py install for Flask-OpenID ... done
  Running setup.py install for Flask-JWT-Extended ... done
  Running setup.py install for sqlalchemy-utils ... done
  Running setup.py install for unicodecsv ... done
  Running setup.py install for thrift ... done
Successfully installed Babel-2.8.0 Flask-Babel-1.0.0 Flask-JWT-Extended-3.24.1 Flask-OpenID-1.2.5 Flask-SQLAlchemy-2.4.2 Mako-1.1.2 MarkupSafe-1.1.1 PyJWT-1.7.1 PyYAML-5.3.1 alembic-1.4.2 apache-airflow-1.10.10 apispec-1.3.3 argcomplete-1.11.1 attrs-19.3.0 cached-property-1.5.1 cattrs-0.9.2 certifi-2020.4.5.1 chardet-3.0.4 click-7.1.2 colorama-0.4.3 colorlog-4.0.2 configparser-3.5.3 croniter-0.3.31 defusedxml-0.6.0 dill-0.3.1.1 dnspython-1.16.0 docutils-0.16 email-validator-1.1.1 flask-1.1.2 flask-admin-1.5.4 flask-appbuilder-2.3.4 flask-caching-1.3.3 flask-login-0.4.1 flask-swagger-0.2.13 flask-wtf-0.14.3 funcsigs-1.0.2 future-0.18.2 graphviz-0.14 gunicorn-19.10.0 idna-2.9 importlib-metadata-1.6.0 iso8601-0.1.12 itsdangerous-1.1.0 jinja2-2.10.3 json-merge-patch-0.2 jsonschema-3.2.0 lazy-object-proxy-1.4.3 lockfile-0.12.2 markdown-2.6.11 marshmallow-2.21.0 marshmallow-enum-1.5.1 marshmallow-sqlalchemy-0.23.0 numpy-1.18.4 pandas-0.25.3 pendulum-1.4.4 prison-0.1.3 psutil-5.7.0 pygments-2.6.1 pyrsistent-0.16.0 python-daemon-2.1.2 python-dateutil-2.8.1 python-editor-1.0.4 python3-openid-3.1.0 pytz-2020.1 pytzdata-2019.3 requests-2.23.0 setproctitle-1.1.10 six-1.15.0 sqlalchemy-1.3.17 sqlalchemy-jsonfield-0.9.0 sqlalchemy-utils-0.36.6 tabulate-0.8.7 tenacity-4.12.0 termcolor-1.1.0 text-unidecode-1.2 thrift-0.13.0 typing-3.7.4.1 typing-extensions-3.7.4.2 tzlocal-1.5.1 unicodecsv-0.14.1 urllib3-1.25.9 werkzeug-0.16.1 wtforms-2.3.1 zipp-3.1.0 zope.deprecation-4.4.0
You are using pip version 18.1, however version 20.2b1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
[root@localairflow ~]#

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

### 1.5、初始化数据库表（默认使用本地的 sqlite 数据库）    [<-- 返回目录](#content)

pip3 安装 airflow 路径

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527130126031-1667779220.png)

airflow 命令在 /usr/local/python3.6.8/bin 目录下

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527130211877-1663676079.png)

初始化：  ./airflow initdb

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@localairflow bin]# ./airflow initdb
DB: sqlite:////usr/local/airflow/airflow.db
[2020-05-27 09:59:53,776] {db.py:378} INFO - Creating tables
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> e3a246e0dc1, current schema
INFO  [alembic.runtime.migration] Running upgrade e3a246e0dc1 -> 1507a7289a2f, create is_encrypted
/usr/local/python3.6.8/lib/python3.6/site-packages/alembic/ddl/sqlite.py:41: UserWarning: Skipping unsupported ALTER for creation of implicit constraintPlease refer to the batch mode feature which allows for SQLite migrations using a copy-and-move strategy.
  "Skipping unsupported ALTER for "
INFO  [alembic.runtime.migration] Running upgrade 1507a7289a2f -> 13eb55f81627, maintain history for compatibility with earlier migrations
INFO  [alembic.runtime.migration] Running upgrade 13eb55f81627 -> 338e90f54d61, More logging into task_instance
INFO  [alembic.runtime.migration] Running upgrade 338e90f54d61 -> 52d714495f0, job_id indices
INFO  [alembic.runtime.migration] Running upgrade 52d714495f0 -> 502898887f84, Adding extra to Log
INFO  [alembic.runtime.migration] Running upgrade 502898887f84 -> 1b38cef5b76e, add dagrun
INFO  [alembic.runtime.migration] Running upgrade 1b38cef5b76e -> 2e541a1dcfed, task_duration
INFO  [alembic.runtime.migration] Running upgrade 2e541a1dcfed -> 40e67319e3a9, dagrun_config
INFO  [alembic.runtime.migration] Running upgrade 40e67319e3a9 -> 561833c1c74b, add password column to user
INFO  [alembic.runtime.migration] Running upgrade 561833c1c74b -> 4446e08588, dagrun start end
INFO  [alembic.runtime.migration] Running upgrade 4446e08588 -> bbc73705a13e, Add notification_sent column to sla_miss
INFO  [alembic.runtime.migration] Running upgrade bbc73705a13e -> bba5a7cfc896, Add a column to track the encryption state of the 'Extra' field in connection
INFO  [alembic.runtime.migration] Running upgrade bba5a7cfc896 -> 1968acfc09e3, add is_encrypted column to variable table
INFO  [alembic.runtime.migration] Running upgrade 1968acfc09e3 -> 2e82aab8ef20, rename user table
INFO  [alembic.runtime.migration] Running upgrade 2e82aab8ef20 -> 211e584da130, add TI state index
INFO  [alembic.runtime.migration] Running upgrade 211e584da130 -> 64de9cddf6c9, add task fails journal table
INFO  [alembic.runtime.migration] Running upgrade 64de9cddf6c9 -> f2ca10b85618, add dag_stats table
INFO  [alembic.runtime.migration] Running upgrade f2ca10b85618 -> 4addfa1236f1, Add fractional seconds to mysql tables
INFO  [alembic.runtime.migration] Running upgrade 4addfa1236f1 -> 8504051e801b, xcom dag task indices
INFO  [alembic.runtime.migration] Running upgrade 8504051e801b -> 5e7d17757c7a, add pid field to TaskInstance
INFO  [alembic.runtime.migration] Running upgrade 5e7d17757c7a -> 127d2bf2dfa7, Add dag_id/state index on dag_run table
INFO  [alembic.runtime.migration] Running upgrade 127d2bf2dfa7 -> cc1e65623dc7, add max tries column to task instance
INFO  [alembic.runtime.migration] Running upgrade cc1e65623dc7 -> bdaa763e6c56, Make xcom value column a large binary
INFO  [alembic.runtime.migration] Running upgrade bdaa763e6c56 -> 947454bf1dff, add ti job_id index
INFO  [alembic.runtime.migration] Running upgrade 947454bf1dff -> d2ae31099d61, Increase text size for MySQL (not relevant for other DBs' text types)
INFO  [alembic.runtime.migration] Running upgrade d2ae31099d61 -> 0e2a74e0fc9f, Add time zone awareness
INFO  [alembic.runtime.migration] Running upgrade d2ae31099d61 -> 33ae817a1ff4, kubernetes_resource_checkpointing
INFO  [alembic.runtime.migration] Running upgrade 33ae817a1ff4 -> 27c6a30d7c24, kubernetes_resource_checkpointing
INFO  [alembic.runtime.migration] Running upgrade 27c6a30d7c24 -> 86770d1215c0, add kubernetes scheduler uniqueness
INFO  [alembic.runtime.migration] Running upgrade 86770d1215c0, 0e2a74e0fc9f -> 05f30312d566, merge heads
INFO  [alembic.runtime.migration] Running upgrade 05f30312d566 -> f23433877c24, fix mysql not null constraint
INFO  [alembic.runtime.migration] Running upgrade f23433877c24 -> 856955da8476, fix sqlite foreign key
INFO  [alembic.runtime.migration] Running upgrade 856955da8476 -> 9635ae0956e7, index-faskfail
INFO  [alembic.runtime.migration] Running upgrade 9635ae0956e7 -> dd25f486b8ea, add idx_log_dag
INFO  [alembic.runtime.migration] Running upgrade dd25f486b8ea -> bf00311e1990, add index to taskinstance
INFO  [alembic.runtime.migration] Running upgrade 9635ae0956e7 -> 0a2a5b66e19d, add task_reschedule table
INFO  [alembic.runtime.migration] Running upgrade 0a2a5b66e19d, bf00311e1990 -> 03bc53e68815, merge_heads_2
INFO  [alembic.runtime.migration] Running upgrade 03bc53e68815 -> 41f5f12752f8, add superuser field
INFO  [alembic.runtime.migration] Running upgrade 41f5f12752f8 -> c8ffec048a3b, add fields to dag
INFO  [alembic.runtime.migration] Running upgrade c8ffec048a3b -> dd4ecb8fbee3, Add schedule interval to dag
INFO  [alembic.runtime.migration] Running upgrade dd4ecb8fbee3 -> 939bb1e647c8, task reschedule fk on cascade delete
INFO  [alembic.runtime.migration] Running upgrade 939bb1e647c8 -> 6e96a59344a4, Make TaskInstance.pool not nullable
INFO  [alembic.runtime.migration] Running upgrade 6e96a59344a4 -> d38e04c12aa2, add serialized_dag table
Revision ID: d38e04c12aa2
Revises: 6e96a59344a4
Create Date: 2019-08-01 14:39:35.616417
INFO  [alembic.runtime.migration] Running upgrade d38e04c12aa2 -> b3b105409875, add root_dag_id to DAG
Revision ID: b3b105409875
Revises: d38e04c12aa2
Create Date: 2019-09-28 23:20:01.744775
INFO  [alembic.runtime.migration] Running upgrade 6e96a59344a4 -> 74effc47d867, change datetime to datetime2(6) on MSSQL tables
INFO  [alembic.runtime.migration] Running upgrade 939bb1e647c8 -> 004c1210f153, increase queue name size limit
INFO  [alembic.runtime.migration] Running upgrade c8ffec048a3b -> a56c9515abdc, Remove dag_stat table
INFO  [alembic.runtime.migration] Running upgrade a56c9515abdc, 004c1210f153, 74effc47d867, b3b105409875 -> 08364691d074, Merge the four heads back together
INFO  [alembic.runtime.migration] Running upgrade 08364691d074 -> fe461863935f, increase_length_for_connection_password
INFO  [alembic.runtime.migration] Running upgrade fe461863935f -> 7939bcff74ba, Add DagTags table
INFO  [alembic.runtime.migration] Running upgrade 7939bcff74ba -> a4c2fd67d16b, add pool_slots field to task_instance
INFO  [alembic.runtime.migration] Running upgrade a4c2fd67d16b -> 852ae6c715af, Add RenderedTaskInstanceFields table
INFO  [alembic.runtime.migration] Running upgrade 852ae6c715af -> 952da73b5eff, add dag_code table
WARNI [airflow.utils.log.logging_mixin.LoggingMixin] cryptography not found - values will not be stored encrypted.
Done.
[root@localairflow bin]# cd /usr/local/
[root@localairflow local]# ls
airflow  bin  etc  games  include  lib  lib64  libexec  python3.6.8  sbin  share  src

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

查看生成的文件

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527130620395-964222209.png)

创建软连接：

```
ln -s /usr/local/python3.6.8/bin/airflow /usr/bin/airflow
ln /usr/local/python3.6.8/bin/gunicorn /usr/bin/gunicorn

```

### 1.6、启动 airflow webserver    [<-- 返回目录](#content)

```
airflow webserver
airflow scheduler

```

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527133015463-823925599.png)

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527132516015-1077389612.png)

访问：http://192.168.213.200:8080/admin/

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200527132041813-1087174041.png)

2、airflow 安装可能遇到的问题    [<-- 返回目录](#content)
-------------------------------------------

### 2.1、pip 安装 airflow 依赖时报版本不对的解决    [<-- 返回目录](#content)

首先卸载已经安装的版本

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200515144930429-1847351650.png)

 重新下载

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200515145017268-998263076.png)

 再次安装 airflow 指令，顺利完成安装：

```
sudo pip install apache-airflow -i https://pypi.douban.com/simple

```

### 2.2、解决 Python.h：No such file or directory    [<-- 返回目录](#content)

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200526224506134-829703537.png)

解决：yum install python-devel  或 yum install python3-devel

![](https://img2020.cnblogs.com/blog/1615446/202005/1615446-20200526224808222-2071591560.png)

### 2.3、启动 airflow webserver 报错：FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn'    [<-- 返回目录](#content)

[启动 airflow webserver 报错：FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn'](https://www.cnblogs.com/lwglinux/p/7100400.html)

执行 gunicorn 启动时，因为在 PATH 中找不到该命令报错。

创建 gunicorn 软连接

ln –fs /home/admin/python3.6/bin/gunicorn/bin/gunicorn /bin/gunicorn

或者将 /home/admin/pytthon3.6/bin 添加到 PATH，export PATH=${PATH}:/home/admin/python3.6/bin

 ---