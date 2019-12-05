



## 软件安装



### MySQL:5.7

1. 下载镜像

```shell
docker pull mysql:5.7
```

2. 运行容器

分解命令

```shell
docker run 
-p 3306:3306 
--name mysql 
--restart=always 
-v $PWD/conf:/etc/mysql/conf.d 
-v $PWD/logs:/logs
-v $PWD/data:/mysql_data 
-e MYSQL_ROOT_PASSWORD=123456 
-d mysql:5.7 
--lower_case_table_names=1
```

复制用命令

```shell
docker run -p 3306:3306 --name mysql --restart=always -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/mysql_data -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 --lower_case_table_names=1
```

```shell
docker run -p 3306:3306 --name mysql --restart=always -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7 --lower_case_table_names=1
```



 

echo "lower_case_table_names=1" >> /etc/mysql/mysql.conf.d/mysqld.cnf

mysql -uroot -p123456 -e "show variables like '%case_table%'"  



C C#/Db D D#/Eb E F F#/Gb G G#/Ab A A#/Bb B C





C      大三, 小三 43

Cm   小三, 大三 34

C0     

C+





### RabbitMQ

Docker运行RabbitMQ容器:

```shell
docker run --name rabbitmq --hostname rabbitmq --restart=always -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=123456 -d rabbitmq:management
```



















