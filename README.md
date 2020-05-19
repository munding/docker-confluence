<!--
 * @Author: Aladdin
 * @Date: 2020-05-18 16:03:09
 * @LastEditTime: 2020-05-19 11:46:06
 * @FilePath: /Atlassian/README.md
 * @Description: file description
--> 
docker run -itd --name confluence -p 18010:8090 -e TZ="Asia/Shanghai" -v /Users/traveler/confluence.wzlinux.com:/var/atlassian/confluence kuaidaili/confluence:latest

java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p conf -o http://192.168.0.181 -s B8PL-IEY4-4NWQ-S4AV


docker run -itd --name mysql -v ~/DatabaseData/mysql/data:/var/lib/mysql -v ~/DatabaseData/mysql/mysql.conf.d:/etc/mysql/mysql.conf.d -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 mysql:5.7

# 创建jira数据库及用户
CREATE DATABASE jiradb CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on jiradb.* to 'jirauser'@'%' identified by 'jirauser';

# 创建confluence数据库及用户
CREATE DATABASE confdb CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
grant all on confdb.* to 'confuser'@'%' identified by 'confuser';

# 以在my.cnf中配置
# confluence要求设置事务级别为READ-COMMITTED
set global tx_isolation='READ-COMMITTED';


java -jar atlassian-agent.jar -d -m test@test.com -n BAT -p 'net.customware.confluence.plugin.composition' -o http://192.168.0.181 -s BCS2-DYDV-ECV8-ZOVH
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common