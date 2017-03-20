---
layout: post
title: QingdaoU开源OJ后期使用与维护
data: 2017-03-20 21:52
category: "OJ维护"
---

本文介绍了关于QingDaoU开源OJ的一种数据迁移的方法,要求两台服务器的数据不能有交叉,否则被植入数据的那台的冲突数据会被抛弃.

---
# 拥有数据的服务器

## ssh登录存有数据的服务器
```
ssh username@ip
```
## 查看web_server的容器ID
```
swjtuacmer@swjtuacmer-ubuntu:~
sudo docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                         NAMES
86762ccac25f        qduoj/judger          "/bin/sh -c 'bash ..."   3 hours ago         Up 3 hours          0.0.0.0:8085->8080/tcp        judger_judger_1
9c13f35c3746        nginx                 "nginx -g 'daemon ..."   2 months ago        Up 5 hours          0.0.0.0:80->80/tcp, 443/tcp   ojwebserver_nginx_1
a24e56cc5911        qduoj/oj_web_server   "/bin/sh -c 'bash ..."   2 months ago        Up 3 hours          127.0.0.1:8080->8080/tcp      ojwebserver_oj_web_server_1
58f6cd86e207        mysql                 "docker-entrypoint..."   3 months ago        Up 5 hours          3306/tcp                      ojwebserver_mysql_1
fdf5ae29470f        redis                 "docker-entrypoint..."   3 months ago        Up 5 hours          6379/tcp                      ojwebserver_redis_1

```

## 进入web_server容器

```
swjtuacmer@swjtuacmer-ubuntu:~
sudo docker exec -i -t a24e56cc5911 /bin/bash
```

## 导出用户/题目/比赛等信息 并退出
```
root@a24e56cc5911:/code# python manage.py dumpdata >oj_data.json
root@a24e56cc5911:/code# exit
```

## 进入OnlineJudge目录 传输导出的信息
```
cd /home/OnlineJudge
sudo scp oj_data.json username@接受数据的服务器ip:/home/ubuntu/
```

## 进入数据文件夹的父文件夹 将数据文件夹发送至接受这些数据的服务器上
注意发送数据的时候不能直接发送到接受数据服务器的/home目录下,因为我们在/home下没有权限
```
cd /home
sudo scp -r /home/test_case username@接受数据的服务器ip:/home/ubuntu/
```
因为数据比较多 所以这个过程会非常长 所以如果只有没几个题目的数据是新的话，只需要传输新加的那几个文件夹就可以了，不用把整个test_case文件夹传过去

如果有图片资源要更新，则还需要传输upload文件夹，当然在确定是哪几个图片的情况下，则只需要传输这几个文件

```
sudo scp -r /home/upload username@接受数据的服务器ip:/home/ubuntu/
```
在传输完test_case ,oj_data.json, upload这些数据以后，退出这台服务器

# 拥有数据的服务器操作到这里就结束了
---
# 接受数据的服务器

## ssh登录接收数据服务器

```
ssh username@ip
```

## 将接收的测试数据，图片资源更新到本地

```
sudo cp -rf /home/ubuntu/test_case/* /home/test_case
sudo cp -rf /home/ubuntu/upload/* /home/upload
```
## 将用户/题目/比赛数据移至OnlineJudge文件夹下
```
sudo mv /home/ubuntu/oj_data.json /home/OnlineJudge
```

查询oj_web_server容器的ID并进入容器
```
ubuntu@VM-88-210-ubuntu:~$ sudo docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                         NAMES
aebd027ba0de        qduoj/judger          "/bin/sh -c 'bash ..."   About an hour ago   Up About an hour    0.0.0.0:8085->8080/tcp        judger_judger_1
b808ca027430        nginx                 "nginx -g 'daemon ..."   2 hours ago         Up About an hour    0.0.0.0:80->80/tcp, 443/tcp   ojwebserver_nginx_1
d6213b395107        qduoj/oj_web_server   "/bin/sh -c 'bash ..."   2 hours ago         Up About an hour    127.0.0.1:8080->8080/tcp      ojwebserver_oj_web_server_1
241773a1e436        redis                 "docker-entrypoint..."   2 hours ago         Up About an hour    6379/tcp                      ojwebserver_redis_1
d1359b98354b        mysql                 "docker-entrypoint..."   2 hours ago         Up About an hour    3306/tcp                      ojwebserver_mysql_1
ubuntu@VM-88-210-ubuntu:~$ sudo docker exec -i -t d6213b395107 /bin/bash
```

## 覆盖数据并退出

```
root@d6213b395107:/code# python manage.py loaddata oj_data.json
root@d6213b395107:/code# exit
```

## 数据导入后的一些处理
由于那个导入的json里包含了判题服务器的信息,所以需要登录OJ后台去检查一下判题服务器的配置(IP,TOKEN等)
关于TOKEN的查看
Token在OnlineJudge/dockerfile/judger/docker-compose.yml 文件里查看

而IP则用下面的方法查看
```
sudo docker inspect --format='{{json .NetworkSettings.Networks}}' {JUDGER_CONTAINER_ID}
```
其中{JUDGER_CONTAINER_ID}是judger在docker容器里的ID,请用docker ps -a查看.


如果出现Invlaid Token的话就说明配置出问题了,或者是这一题的数据有问题了(比如没有数据之类的问题)
要求网页后台的Token和IP是与上述方法获得的Token与IP是一致的

---
# 缺陷
1.这是数据的替换，不是合并，所以当两个OJ的数据有冲突时，接收数据的OJ的数据会被抹掉
2.由于本人能力有限，暂时不会移植submission(用户提交)部分，所以这一部分的数据只有原来的,不会被更新，也就是说，新添加的比赛里的提交，都是没有的，但是能看到排名等情况。

# To be updated
1.submission这一部分的数据是在mysql容器里的，但是不知道为什么..我不知道我的服务器的mysql的密码，如果有密码的话其实可以mysqldump把submission给迁移过去，submission所在的目录是
/home/data/mysql/oj_submission
在mysql容器下的路径是 
/var/lib/mysql
2.过几天来更批量导入题库
