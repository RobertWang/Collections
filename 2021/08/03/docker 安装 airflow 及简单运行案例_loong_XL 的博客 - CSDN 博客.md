> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_42357472/article/details/118392084)

参考：https://github.com/puckel/docker-airflow  
https://blog.csdn.net/qq_40133908/article/details/96722925

```
 1、安装运行：
docker run -d -p 8080:8080 puckel/docker-airflow
2、web打开查看

```

![](https://img-blog.csdnimg.cn/20210701174910957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM1NzQ3Mg==,size_16,color_FFFFFF,t_70)

```
3、root权限进入容器
docker exec -it -u root  3ad***1b1 /bin/bash
4、安装vim
apt-get update
apt-get install vim

5、初始化容器
airflow initdb
如果出现异常：irflow.exceptions.AirflowException: Could not create Fernet object: Incorrect padding
解决办法：
python -c "from cryptography.fernet import Fernet;print(Fernet.generate_key().decode())"

然后会显示一行字符串。

然后执行：export AIRFLOW__CORE__FERNET_KEY=string

再重新：airflow initdb

```

![](https://img-blog.csdnimg.cn/20210701175458620.png)

```
6、拷贝案例进容器/usr/local/airflow/dags目录下，因为airflow.cfg里有写
dags_folder = /usr/local/airflow/dags

docker cp /o本机/demo.py 容器名或id:/usr/local/airflow/dags


```

demo.py

```
from datetime import datetime, timedelta

from airflow import DAG
from airflow.utils import dates
from airflow.utils.helpers import chain
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator


def default_options():
    default_args = {
        'owner': 'airflow',  # 拥有者名称
        'start_date': dates.days_ago(1),  # 第一次开始执行的时间，为 UTC 时间
        'retries': 1,  # 失败重试次数
        'retry_delay': timedelta(seconds=5)  # 失败重试间隔
    }
    return default_args


# 定义DAG
def test1(dag):
    t = "pwd"
    # operator 支持多种类型， 这里使用 BashOperator
    task = BashOperator(
        task_id='test1',  # task_id
        bash_command=t,  # 指定要执行的命令
        dag=dag  # 指定归属的dag
    )
    return task


def hello_world_1():
    current_time = str(datetime.today())
    print('hello world at {}'.format(current_time))


def test2(dag):
    # PythonOperator
    task = PythonOperator(
        task_id='test2',
        python_callable=hello_world_1,  # 指定要执行的函数
        dag=dag)
    return task


def test3(dag):
    t = "date"
    task = BashOperator(
        task_id='test3',
        bash_command=t,
        dag=dag)
    return task


with DAG(
        'test_task',  # dag_id
        default_args=default_options(),  # 指定默认参数
        schedule_interval="*/3 * * * *"  # 执行周期
) as d:
    task1 = test1(d)
    task2 = test2(d)
    task3 = test3(d)
    task1 >> task2 >> task3
    #chain(task1, task2, task3)  # 指定执行顺序

```

```
7、容器里运行脚本
python ~/airflow/dags/tutorial.py

```

![](https://img-blog.csdnimg.cn/20210701193636606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM1NzQ3Mg==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20210702092840923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjM1NzQ3Mg==,size_16,color_FFFFFF,t_70)