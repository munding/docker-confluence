> JIRA是Atlassian公司出品的项目与事务跟踪工具，被广泛应用于缺陷跟踪、客户服务、需求收集、流程审批、任务跟踪、项目跟踪和敏捷管理等工作领域。
> Confluence是一个专业的企业知识管理与协同软件，也可以用于构建企业wiki。使用简单，但它强大的编辑和站点管理特征能够帮助团队成员之间共享信息、文档协作、集体讨论，信息推送。

# 准备工具

1. Jira镜像：[Docker Hub链接](https://hub.docker.com/r/cptactionhank/atlassian-jira-software)
2. Confluence镜像：[Docker Hub链接](https://hub.docker.com/r/cptactionhank/atlassian-confluence)
3. Mysql镜像：[Docker Hub链接](https://hub.docker.com/_/mysql)
4. 破解工具：[下载链接](https://gitee.com/pengzhile/atlassian-agent/releases)

至于[Docker](https://docs.docker.com/engine/install/)和[Docker Compose](https://docs.docker.com/compose/install/)的安装这里就不再赘述，Docker官方文档讲解的很清楚。

PS：Docker官网下载慢的话可以使用[ 清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)，Docker镜像拉取慢的话可以使用阿里云的容器镜像服务[镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

# 制作镜像

## Jira

目录结构

```
Jira/
    atlassian-agent.jar
    Dockerfile
```

Dockerfile

```dockerfile
FROM cptactionhank/atlassian-jira-software:latest

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/jira/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh
```

构建镜像（latest版本为8.8）

```bash
docker build -t aladdinding/jira:8.8 .
```

## Confluence

目录结构

```
Confluence/
    atlassian-agent.jar
    Dockerfile
```

Dockerfile

```dockerfile
FROM cptactionhank/atlassian-confluence:latest

USER root

# 将代理破解包加入容器
COPY "atlassian-agent.jar" /opt/atlassian/confluence/

# 设置启动加载代理包
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```

构建镜像（latest版本为7.4）

```bash
docker build -t aladdinding/confluence:7.4 .
```

## Mysql

目录结构

```
Mysql/
    Dockerfile
    my.cnf
```

my.cnf（运行Jira和Confluence的Mysql相关配置），各个版本的Jira和Confluence的数据库设置可能不太一样，具体查看官方文档：

[Connecting Jira applications to MySQL 5.7](https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-mysql-5-7-966063305.html)

[Database Setup For MySQL](https://confluence.atlassian.com/conf74/database-setup-for-mysql-1003129360.html)

```
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_bin
default-storage-engine=INNODB
max_allowed_packet=512M
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
```

Dockerfile

```dockerfile
FROM mysql:5.7

USER root

# 使用自定义的配置文件
COPY "my.cnf" /etc/mysql/mysql.conf.d
```

创建镜像

```bash
docker build -t aladdinding/mysql:5.7 .
```

# Docker Composeq启动服务

docker-compose.yml

```yaml
version: "3"
services:
    mysql:
        image: "aladdinding/mysql:5.7"
        ports: 
        - "3306:3306"
        volumes: 
        - /data/atlassian-mysql:/var/lib/mysql
        environment: 
        - MYSQL_ROOT_PASSWORD=atlassian
        restart: always
    confluence:
        image: "aladdinding/confluence:7.4"
        ports: 
        - "8601:8090"
        volumes: 
        - ~/confluence:/var/atlassian/confluence
        environment: 
        - TZ="Asia/Shanghai"
        links:
        - mysql
        restart: always
    jira:
        image: "aladdinding/jira:8.8"
        ports: 
        - "8600:8080"
        volumes: 
        - ~/jira:/var/atlassian/jira
        environment: 
        - TZ="Asia/Shanghai"
        links:
        - mysql
        restart: always
```

docker-compose后台启动

```bash
docker-compose up -d
```

# 创建Jira和Confluence数据库及用户

进入启动的Mysql容器

```
docker exec -it ${容器id} /bin/bash
```

创建数据库及用户

```sql
# Jira
CREATE DATABASE jira CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on jira.* to 'jira'@'%' identified by 'jira';

# Confluence
CREATE DATABASE confluence CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on confluence.* to 'confluence'@'%' identified by 'confluence';
```

# 初始化及破解Jira

浏览器访问 IP:8601，如：http://192.168.0.181:8600/

<img src="https://img.aladdinding.cn/jira-1.png"  />

主机直接填写mysql就行，--link启动容器docker已经做好了映射，可以自行到mysql容器内运行`cat /etc/hosts`查看

![](https://img.aladdinding.cn/jira-2.png)

设置属性

![](https://img.aladdinding.cn/jira-3.png)

获取服务器 ID，在存放`atlassian-agent.jar`的目录下运行以下命令完成破解获取许可证（可以使用容器内的java环境）

![](https://img.aladdinding.cn/jira-4.png)

注意替换邮箱，公司名称，访问地址，服务器ID等

```bash
java -jar atlassian-agent.jar -d -m mytest@mytest.com -n BAT -p 'jira' -o http://192.168.0.181 -s BDMK-KXF1-H7GV-F7QG
```

如下为我生成的许可证：


```
====================================================
=======        Atlassian Crack Agent         =======
=======           https://zhile.io           =======
=======          QQ Group: 30347511          =======
====================================================

Your license code(Don't copy this line!!!): 

AAABnQ0ODAoPeJyNUl1PgzAUfedXkPhcpBDcR9JEBVQibCrT+NqxO1fDCrkt0/nrLYPFr2UxaZq0u
efcc889JznXdsa3tufblI6DYOz6dpjPbM/1XOsFAeSqqmtAJxUFSAWzbQ0TvgYWTrMsfgiTi9QKE
bgWlYy4BtYCiRsQz7eOQCJQBYq6RbFHWYq10LCwyw5gz7f2SutajU9PP1aiBEdUVsaF1CC5LCB+r
wVu+27DEXEH5livAvleZbwQHfUkTbJkFkfWpFnPAafLRwWoGKF7cUe4aqwWTaGd9kFUtdRvHMH5Q
3SklhdabIBpbOCHl9//j8CNKh6CmRq70t6eJ9O4Hc6z8mb+ZeOuJN7wstktgy15qXr630RTfOFSq
K6uddoYTUeeQ8+GjuvQIbXCSmqjMjaul0yD0uft5RTVumM87MI/58o1x1ZLp7BfRBKxNInyeEJSG
oxMiM6o61M/+LHXQ1HKATeABn4ZZbfk9vmKkpvB9RO5GtxfH0rw32zcNVisuILf+f0O3rlXo1D9e
EYoOyC2922n8fJi9glBKSZMMCwCFDgDVu1zI/dsSeAFI+Hdvr0Nd3gxAhQsGkV9fRduoZjPDwdyD
BQBVAk4zg==X02k0
```

将生成的许可证复制到页面，完成破解

![](https://img.aladdinding.cn/jira-5.png)

安装完成后查看版本和许可证

![](https://img.aladdinding.cn/jira-6.png)

# 初始化及破解Confluence

浏览器访问 IP:8601，如：http://192.168.0.181:8601/

![](https://img.aladdinding.cn/confluence-1.png)

按照流程下一步，到授权码页面，和Jira激活方法一样，使用以下命令，获取授权码后粘贴，点击下一步

```bash
java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p 'conf' -o http://192.168.0.181 -s BOH6-4HVK-CBCV-BRNO
```

![](https://img.aladdinding.cn/confluence-2.png)

避免可能出现的中文乱码，数据库URL修改如下，主机名为mysql，在设置Jira中提及过，这里不在赘述

```
jdbc:mysql://mysql/confluence?useUnicode=true&characterEncoding=utf8
```

![](https://img.aladdinding.cn/confluence-3.png)

选择示范站点（如果是还原用户也可以先选择示范站点，进入Confluence也会有备份还原的选项）

![](https://img.aladdinding.cn/confluence-4.png)

可以先选择 在Confluence中管理用户与组，在后面的章节会讲如何与Jira对接

![](https://img.aladdinding.cn/confluence-5.png)

进入Confluence，查看授权码细节

![](https://img.aladdinding.cn/confluence-6.png)

# 配置Confluence与Jira用户数据对接

进入JIra选择用户管理-Jira用户服务器-新建应用程序

![](https://img.aladdinding.cn/conf%26jira-1.png)

填写应用程序名和密码，ip地址172.18.0.1为Docker默认网络模式bridge下的宿主机IP

![](https://img.aladdinding.cn/conf%26jira-2.png)

进入Confluence选择用户目录-添加用户目录（目录类型为：Atlassian Jira）

![](https://img.aladdinding.cn/conf&jira-3.png)

填写Jira服务器URL，刚才在Jira用户服务器中设置的应用程序名称和应用程序密码，点击下方的测试设置，然后点击测试保存

![](https://img.aladdinding.cn/conf&jira-4.png)

将JIRA Server 顺序顶置最上，点击同步即可将Jira用户信息同步到Confluence

![](https://img.aladdinding.cn/conf&jira-5.png)

如果是通过备份还原的Jira及Confluence可以直接禁用之前的用户目录然后移除。其他应用程序关联等设置比较简单，自行操作。

# 插件破解

第三方插件将其应用密钥/插件关键字作为-p参数。如：-p 'com.valiantys.spreadsheets'

```
java -jar atlassian-agent.jar -d -m mytest@mytest.com -n BAT -p 'com.valiantys.spreadsheets' -o http://192.168.0.181 -s BDMK-KXF1-H7GV-F7QG
```

查找新应用，选择你想要的应用插件，点击免费使用，点击接受&安装，进入管理应用页面，粘贴生成的许可证，点击更新完成破解

![](https://img.aladdinding.cn/confapp-1.png)

# 异常记录

**Jira 不支持数据库排序规则“utf8mb4_bin”和表排序规则“utf8mb4_bin”。或者是Jira 不支持表排序规则：“utf8mb4_bin”。Jira 支持数据库排序规则：“utf8_bin”。之类的故障诊断信息。**

更改yourDB以适合您的数据库名称：

```sql
ALTER DATABASE yourDB CHARACTER SET utf8 COLLATE utf8_bin
```

更改表格排序
以下查询将生成一系列ALTER TABLE语句，然后您必须对数据库运行该语句。更改yourDB以适合您的数据库名称：

```sql
SELECT CONCAT('ALTER TABLE ',  table_name, ' CHARACTER SET utf8 COLLATE utf8_bin;')
FROM information_schema.TABLES AS T, information_schema.`COLLATION_CHARACTER_SET_APPLICABILITY` AS C
WHERE C.collation_name = T.table_collation
AND T.table_schema = 'yourDB'
AND
(
    C.CHARACTER_SET_NAME != 'utf8'
    OR
    C.COLLATION_NAME != 'utf8_bin'
);
```

更改列整理
以下查询（一varchar列用于列，一varchar列用于非列）将生成一系列ALTER TABLE语句，然后您必须针对数据库运行这些语句。更改yourDB以适合您的数据库名称：

```sql
SELECT CONCAT('ALTER TABLE `', table_name, '` MODIFY `', column_name, '` ', DATA_TYPE, '(', CHARACTER_MAXIMUM_LENGTH, ') CHARACTER SET UTF8 COLLATE utf8_bin', (CASE WHEN IS_NULLABLE = 'NO' THEN ' NOT NULL' ELSE '' END), ';')
FROM information_schema.COLUMNS 
WHERE TABLE_SCHEMA = 'yourDB'
AND DATA_TYPE = 'varchar'
AND
(
    CHARACTER_SET_NAME != 'utf8'
    OR
    COLLATION_NAME != 'utf8_bin'
);
```

**com.atlassian.crowd.exception.ApplicationPermissionException: Forbidden (403) 加载页面时发生 "403 - Forbidden" 错误 client.forbidden.exception 转换到Jira主页***

Jira服务器IP地址填写错误，尤其是使用Docker搭建的用户，如果不是就去看log日志吧。。。
