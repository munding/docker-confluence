<!--
 * @Author: Aladdin
 * @Date: 2020-05-18 16:03:09
 * @LastEditTime: 2020-05-20 16:44:37
 * @FilePath: /Atlassian/README.md
 * @Description: file description
--> 
docker run -itd --name confluence -p 18010:8090 -e TZ="Asia/Shanghai" -v /opt/confluence:/var/atlassian/confluence kuaidaili/confluence:latest

java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p conf -o http://192.168.0.181 -s BWOI-VA2G-XZVB-K8G3


docker run -itd --name mysql -v /opt/docker-mysql/var/lib/mysql:/var/lib/mysql -v /opt/docker-mysql/mysql.conf.d:/etc/mysql/mysql.conf.d -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 mysql:5.7

# 创建jira数据库及用户
CREATE DATABASE jira CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on jira.* to 'jira'@'%' identified by 'jira';

# 创建confluence数据库及用户
CREATE DATABASE confluence CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on confluence.* to 'confluence'@'%' identified by 'confluence';

# 以在my.cnf中配置
# confluence要求设置事务级别为READ-COMMITTED
set global tx_isolation='READ-COMMITTED';


java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p 'net.customware.confluence.plugin.composition' -o http://192.168.0.181 -s BK1V-66W9-PIR0-IL8T
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
docker build -t kuaidaili/confluence:latest .
docker build -t kuaidaili/jira:latest .

java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p conf -o http://39.105.141.110 -s BA7G-8EQE-Z8W6-MJ6N
