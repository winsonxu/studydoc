# mysql innodb 突然启动不起来，原因意然是这个，12步解决

生产环境中的mysql突然启动不了，查了原因是innodb库错误，以前就遇到过这个问题，稀里糊涂的没解决，结果导致大量数据丢失。这些又遇到这个问题，果断把那个有问题的数据库移动了别的地方，启动了mysql使用。然后正好赶上中秋节假期，所以花了两天时间认真查资料，一点点的解决问题。

因为我是用docker做了一个沙箱,但是启动不起来,在这里上面浪费了半天的时间。这个跳过，直接说一下思路和过程。有什么问题可以微信我（winsonhsu)备注mysql技术交流

整个过程需要三个库 we7  we7_old  we7_tmp
1.备份数据库文件夹we7
2.设置数据库为恢复模式innodb_force_recovery = 3
3.启动数据库we7
4.此时可以看到库里的innodb数据表都没有显示出来.只有myiasm表
4.创建一个新库we7_tmp
5.使用navcat的tools>>data transfer 库建从we7到新库we7_tmp 这样就把myiasm库都复制过去了
6.还原一个过去备份的数据库we7_old,用navcat里的tool>>structure synchronization工具进行架构比较,只比较表,得到差异后只复制增加的表语句,得到innodb的表结构 innodb.sql
7.在we7_tmp中执行innodb.sql
8.将复制过来的.ibd文件与.frm文件发生联系。具体就是在控制台执行下面命令：
for f in $(ls *.ibd); do echo "alter table" $(basename $f .ibd) " import tablespace;" >> import.sql; done
得到import.sql文件等一下使用

恢复表数据需要首先将原先的.ibd文件与原先的.frm文件解除绑定，具体就是在控制台执行下面命令：
for f in $(ls *.ibd); do echo "alter table" $(basename $f .ibd) " discard tablespace;" >> discard.sql; done
得到discard.sql语句等一下使用


9在we7_tmp库中执行discard.sql  
10.将we7文件夹中的ibd文件全部拷贝过来
 cp /www/backup/we7_test/*.ibd /www/docker/mysql/data/we7_test/
设置权限为全部可以读写
11.在we7_tmp库中执行import.sql

12.设置完成后删除we7和we7_old数据库  将we7_tmp数据库重命名为we7  写一个rename.sh
```shell
#!/bin/bash
# 假设将we7_tmp数据库名改为we7
# MyISAM直接更改数据库目录下的文件即可

mysql -uroot -pmD4Yyxrb3tcLDLFw -e 'create database if not exists we7_test'
list_table=$(mysql -uroot -pmD4Yyxrb3tcLDLFw -Nse "select table_name from information_schema.TABLES where TABLE_SCHEMA='we7_tmp'")

for table in $list_table
do
    mysql -uroot -pmD4Yyxrb3tcLDLFw -e "rename table we7_tmp.$table to we7_test.$table"
done
```
执行./rename.sh
数据库修复完成，全部数据都回来了