# Confluence

> Confluence是一个专业的企业知识管理与协同软件，也可以用于构建企业wiki。使用简单，但它强大的编辑和站点管理特征能够帮助团队成员之间共享信息、文档协作、集体讨论，信息推送

## 数据库设置

不建议将数据库部署在Docker容器，推荐使用云数据库或者物理机数据库。

### 文档

[Confluence Data Center and Server documentation](https://confluence.atlassian.com/doc/confluence-data-center-and-server-documentation-135922.html)

[Database Configuration](https://confluence.atlassian.com/doc/database-configuration-159764.html)

### 数据库设置

选择安装的Confluence版本，阅读[Database Setup For MySQL](https://confluence.atlassian.com/doc/database-setup-for-mysql-128747.html)后，修改[Mysql配置文件](https://dev.mysql.com/doc/refman/5.7/en/option-files.html)，本文以Mysql 8.0为例

```
[mysqld]
...
character-set-server=utf8mb4 
collation-server=utf8mb4_bin
default-storage-engine=INNODB
max_allowed_packet=256M 
innodb_log_file_size=2GB
transaction-isolation=READ-COMMITTED
binlog_format=row
log-bin-trust-function-creators = 1
// 如果为Mysql5.7，关闭derived_merge能优化仪表板加载缓慢
optimizer_switch = derived_merge=off
...
```

如果`sql_mode = NO_AUTO_VALUE_ON_ZERO`，请删除此选项

### 创建数据库&用户

- 创建数据库


```mysql
CREATE DATABASE <database-name> CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
```

- 创建用户

```mysql
CREATE user '<confluenceuser>'@'localhost' IDENTIFIED BY '<password>';
```

**如果 Confluence 与数据库不在同一台服务器上运行（或者是Docker用户），请用 Confluence 服务器的主机名或 IP 地址替换 localhost（也可以使用`%`，表示允许所有host）**

- 授权

```mysql
GRANT ALL PRIVILEGES ON <database-name>.* TO '<confluenceuser>'@'localhost' WITH GRANT OPTION;
```

## Docker Compose

### 文档

镜像：[atlassian/confluence-server](https://hub.docker.com/r/atlassian/confluence-server)

破解插件：~~[atlassian-agent](https://gitee.com/pengzhile/atlassian-agent)~~ 项目已被私有，无法访问

### 准备工具

#### 破解插件

~~[atlassian-agent.jar](https://gitee.com/pengzhile/atlassian-agent/attach_files/283102/download/atlassian-agent-v1.2.3.zip)~~ 直接使用仓库内`atlassian-agent.jar`

#### 数据库驱动

官方镜像并没有内置MySQL driver，需要自行下载：[Database JDBC Drivers](https://confluence.atlassian.com/doc/database-jdbc-drivers-171742.html)。

Mysql 8.0 下载：[mysql-connector-java-8.0.22.jar](https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-8.0.22.zip)

Mysql 5.7 下载：[mysql-connector-java-5.1.48.jar](https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-5.1.48.zip)


#### docker-compose.yml

```yaml
version: '3'
services:
    confluence:
        image: "atlassian/confluence-server"
        volumes: 
            - ./atlassian-agent.jar:/var/atlassian/atlassian-agent.jar
            - ./mysql-connector-java-8.0.22.jar:/opt/atlassian/confluence/confluence/WEB-INF/lib/mysql-connector-java-8.0.22.jar
            - ~/your-confluence-home:/var/atlassian/application-data/confluence
        environment:
            - JAVA_OPTS="-javaagent:/var/atlassian/atlassian-agent.jar"
            - JVM_MINIMUM_MEMORY=2048m
            - JVM_MAXIMUM_MEMORY=2048m
            - JVM_RESERVED_CODE_CACHE_SIZE=512m
        ports: 
            - "8090:8090"
        restart: always
```

默认内存分配为1024m，如果需要覆盖 Confluence Server 的默认内存分配，可以通过环境变量`JVM_MINIMUM_MEMORY`、`JVM_MAXIMUM_MEMORY`、`JVM_RESERVED_CODE_CACHE_SIZE` 控制最小堆(Xms)和最大堆(Xmx)。

已上传Github：[aladdinding/Confluence-and-Jira](https://github.com/aladdinding/Confluence-and-Jira)

### 运行

```
docker-compose up -d
```

查看日志，发现`========= agent working =========`则插件正常运行

## 初始化配置

#### 破解

![](https://img.aladdinding.cn/confluence1.png)

复制 Server ID `BT5W-KP7Q-31DT-PTNG`，使用容器内的Java环境，进入存放`atlassian-agent.jar`目录，运行下方命令生成Key

```bash
java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p 'conf' -o http://localhost:8090 -s BT5W-KP7Q-31DT-PTNG
```

#### 设置数据库

这里使用的宿主机搭建的Mysql 8.0，一路下一步即可

![](https://img.aladdinding.cn/confluence2.png)

#### 查看授权细节

![](https://img.aladdinding.cn/confluence3.png)

## 插件破解

第三方插件将其应用密钥/插件关键字作为-p参数。如：-p 'com.valiantys.spreadsheets'

```
java -jar atlassian-agent.jar -d -m mytest@mytest.com -n BAT -p 'com.valiantys.spreadsheets' -o http://localhost:8090 -s BDMK-KXF1-H7GV-F7QG
```

查找新应用，选择你想要的应用插件，点击免费使用，点击接受&安装，进入管理应用页面，粘贴生成的许可证，点击更新完成破解

![](https://img.aladdinding.cn/confluence4.png)

# JIRA

> JIRA是Atlassian公司出品的项目与事务跟踪工具，被广泛应用于缺陷跟踪、客户服务、需求收集、流程审批、任务跟踪、项目跟踪和敏捷管理等工作领域。

Jira搭建流程和Confluence类似，这里不再赘述，附上相关内容

### 文档

[Jira Software Data Center and Server documentation](https://confluence.atlassian.com/jirasoftwareserver)

[Connecting Jira applications to a database](https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-a-database-938846850.html)

### 破解命令

```
java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p 'jira' -o http://localhost:8080 -s BT5W-KP7Q-31DT-PTNG
```

### 配置Confluence与Jira用户数据对接

进入Jira选择用户管理 > Jira用户服务器 > 添加应用程序

![](https://img.aladdinding.cn/jira1.png)

进入Confluence > 用户管理 > 用户目录 > 添加目录（目录类型为：Atlassian Jira）

![](https://img.aladdinding.cn/jira2.png)

将JIRA Server 顺序顶置最上，点击同步即可将Jira用户信息同步到Confluence

![](https://img.aladdinding.cn/jira3.png)

如果是通过备份还原的Jira及Confluence可以直接禁用之前的用户目录然后移除。其他应用程序关联等设置比较简单，自行操作。



# 异常记录

- Confluence重启后一段时间内无响应

```
confluence_1 | WARNING: An illegal reflective access operation has occurred
confluence_1 | WARNING: Illegal reflective access by com.atlassian.hibernate.adapter.proxy.BytecodeProviderImpl_ImplementV2Proxy (file:/opt/atlassian/confluence/confluence/WEB-INF/lib/hibernate.adapter-1.0.3.jar) to field java.lang.reflect.Field.modifiers
confluence_1 | WARNING: Please consider reporting this to the maintainers of com.atlassian.hibernate.adapter.proxy.BytecodeProviderImpl_ImplementV2Proxy
confluence_1 | WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
confluence_1 | WARNING: All illegal access operations will be denied in a future release
confluence_1 | Security framework of XStream not explicitly initialized, using predefined black list on your own risk.
```

类似问题：

https://community.atlassian.com/t5/Confluence-questions/Confluence-no-longer-responds-Debugging-articles/qaq-p/1404597

https://community.atlassian.com/t5/Confluence-questions/Illegal-reflective-access-by-BytecodeProviderImpl/qaq-p/1255035

https://community.atlassian.com/t5/Confluence-questions/hibernate-adapter-1-0-3-jar/qaq-p/1281057

结论：

- 等待一段时间会自动运行正常
- 非法反射访问错误是Java9添加的，所以尝试在Java8上运行Confluence，不过官方最新镜像都是Java11了！

