## Mysql

### binlog 原理 

记录mysql的数据更新或潜在更新

mysql里面的主从复制依赖binglog



master-slave

读写分离， master 写 ， salve读， 增加读操作的性能



Master                                  slave

binlog                                     IO thread   SQL thread

​						relay log (中转日志)



3种模式  ： statement 记录sql语句   row  记录记录数据  mixed  按语句判断





### Mysql的主从模式实现

**数据库的版本5.7版本**

**安装以后文件对应的目录**

mysql的数据文件和二进制文件： /var/lib/mysql/

mysql的配置文件： /etc/my.cnf

mysql的日志文件： /var/log/mysql.log

 

## 140 为master

1.   创建一个用户’repl’,并且允许其他服务器可以通过该用户远程访问master，通过该用户去读取二进制数据，实现数据同步

Create user repl identified by ‘repl； repl用户必须具有REPLICATION SLAVE权限，除此之外其他权限都不需要

GRANT REPLICATION SLAVE ON *.* TO ‘repl’@’%’ IDENTIFIED BY ‘repl’ ; 

2.   修改140 my.cnf配置文件，在[mysqld] 下添加如下配置

log-bin=mysql-bin //启用二进制日志文件

server-id=130 服务器唯一ID 

3.   重启数据库 systemctl restart mysqld 

4.   登录到数据库，通过show master status  查看master的状态信息

## 142 为slave

1.   修改142 my.cnf配置文件， 在[mysqld]下增加如下配置

server-id=132  服务器id，唯一

relay-log=slave-relay-bin

relay-log-index=slave-relay-bin.index

read_only=1

2.   重启数据库： systemctl restart mysqld

3.   连接到数据库客户端，通过如下命令建立同步连接

change master to master_host=’192.168.11.140’, master_port=3306,master_user=’repl’,master_password=’repl’,**master_log_file=’mysql-bin.000001’,master_log_pos=0;**

  红色部分从master的show master status可以找到对应的值，不能随便写。

4.   执行 start slave

5.   show slave status\G;查看slave服务器状态，当如下两个线程状态为yes，表示主从复制配置成功

**Slave_IO_Running=Yes**

**Slave_SQL_Running=Yes**



## 主从同步的原理

![1532218626921](C:\Users\Arlen\AppData\Local\Temp\1532218626921.png)



1.   master记录二进制日志。在每个事务更新数据完成之前，master在二日志记录这些改变。MySQL将事务串行的写入二进制日志，即使事务中的语句都是交叉执行的。在事件写入二进制日志完成后，master通知存储引擎提交事务 2.   slave将master的binary log拷贝到它自己的中继日志。首先，slave开始一个工作线程——I/O线程。I/O线程在master上打开一个普通的连接，然后开始binlog dump process。Binlog dump process从master的二进制日志中读取事件，如果已经跟上master，它会睡眠并等待master产生新的事件。I/O线程将这些事件写入中继日志 3.   SQL线程从中继日志读取事件，并重放其中的事件而更新slave的数据，使其与master中的数据一致 

binlog： 用来记录mysql的数据更新或者潜在更新（update xxx where id=x effect row 0）;

文件内容存储：/var/lib/mysql

 

mysqlbinlog --base64-output=decode-rows -v mysql-bin.000001  查看binlog的内容

 

## binlog的格式

**statement** **： 基于sql语句的模式。update table set name =””; effect row 1000； uuid、now() other function**

row： 基于行模式; 存在1000条数据变更；  记录修改以后每一条记录变化的值

mixed: 混合模式，由mysql自动判断处理

修改binlog_formater,通过在mysql客户端输入如下命令可以修改

set global binlog_format=’row/mixed/statement’;

或者在vim /etc/my.cnf  的[mysqld]下增加binlog_format=‘mixed’





主从同步的延时问题

### 主从同步延迟是怎么产生的

1.   当master库tps比较高的时候，产生的DDL数量超过slave一个sql线程所能承受的范围，或者slave的大型query语句产生锁等待

2.   网络传输： bin文件的传输延迟

3.   磁盘的读写耗时：文件通知更新、磁盘读取延迟、磁盘写入延迟

### 解决方案

1.   在数据库和应用层增加缓存处理，优先从缓存中读取数据

2.   减少slave同步延迟，可以修改slave库sync_binlog属性； 

sync_binlog=0  文件系统来调度把binlog_cache刷新到磁盘

sync_binlog=n  

3.   增加延时监控

Nagios做网络监控

mk-heartbeat



1.   Mysql的主从配置
2.   了解binlog及主从复制原理

### MySQL索引 B+tree



![1532236305928](C:\Users\Arlen\AppData\Local\Temp\1532236305928.png)



![1532236349075](C:\Users\Arlen\AppData\Local\Temp\1532236349075.png)



二叉树和红黑树高度不可控，但是B+tree可以控制高度，建议高度是3-5层。

B+用叶子节点存储数据，IO渐进复杂度是恒定的。



索引好坏的标准  IO渐进复杂度



### Mysql底层实现

#### MyIsam

3个文件

test_myisam.frm  表的元数据

test_myisam.myd  数据

test_myisam.myi 索引

#### Innodb

2个文件

test_innodb.idb

test_innodb.frm 



![1532236918843](E:\gupao\workspace\typora\1532236918843.png)



BTree 每个节点都存数据， B+节点只有叶子节点存数据，B+叶子节点有指针连起来，查询多个 ：id>1方便



已id为索引的B+tree， 非聚集索引

![1532237670075](C:\Users\Arlen\AppData\Local\Temp\1532237670075.png)

![1532237736048](C:\Users\Arlen\AppData\Local\Temp\1532237736048.png)



聚集索引：以索引来组织数据的一种形式，

Innodb不需要索引文件

innodb创建表的时候，允许不指定主键，但是会自动选一列作为主键，uniq的一列。 如果实在已有一列name，他会帮你自动生成一列作为索引。



![1532238670085](C:\Users\Arlen\AppData\Local\Temp\1532238670085.png)



建立name的索引

select * from user where name = 'james'

![1532239194340](C:\Users\Arlen\AppData\Local\Temp\1532239194340.png)





Innodb作为engine的情况下，为什么用自增id作为主键？

插入层面：B+tree会每次都在最后一个节点插入，索引树以一种几乎填满的方式向后延续，保持物理存储的连续，减少碎片。物理连续性最大。

查询层面： 效率更高



数据库索引使用B+树，为什么？ 

B+树的磁盘读写代价更低

​                   B+树内部节点没有指向关键字的指针，内部节点更小。

​                   B+树的查询效率更加稳定，任何数据的查找必须走一条根节点到叶子节点的路，所有关键字查询路径长度相同，效率相当。

B+实际数据用一个链表链在一起，是得遍历整颗树只要遍历叶子节点就行

B树在提高IO性能的同时没有解决元素遍历效率低下的问题，B+树只要去遍历叶子节点就可以实现整棵树的遍历。