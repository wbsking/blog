Title: MySQL主从配置
Date: 2014-08-28
Category: other
Tag: mysql, 主从复制

### 工作原理

通过MySQL的主从配置，可以实现双机热备和读写分离等等。MySQL的主从复制主要是根据下面三个步骤工作的；

1. master将改变记录到二进制日志（binary log）中；
2. slave将master的binary log拷贝到中继日志（relay log）；
3. slave重做中继日志中的事件，将改变反映到自己的数据库中。

因此我们可以使用主从配置实现master的实时备份，同时当master出现故障时可以将从机切换为主机进行工作。

### 配置

1. master配置

    在master的my.cnf中增加如下配置：
        
        server-id = 1          // 主服务器的ID值
        log-bin = mysql-bin    // 二进制日志

2. slave配置

    在slave的my.cnf中增加如下配置：
        
        log_bin = mysql-bin
        server_id = 2                    // 必须唯一
        relay_log = mysql-relay-bin      // 中继日志
        log_slave_updates = 1
        read_only = 1

3. 查看master的二进制日志位置

     在master上运行命令：
         
        mysql> show master status;
        +------------------+-----------+--------------+------------------+-------------------+
        | File             | Position  | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
        +------------------+-----------+--------------+------------------+-------------------+
        | mysql-bin.000010 | 122164543 |              |                  |                   |
        +------------------+-----------+--------------+------------------+-------------------+
        1 row in set (0.00 sec)

    通过上述命令可以看到master当前的日志在mysql-bin.000010，位置122164543处。我们需要记住这两个值，供下一步使用。

4. slave中设置master

        mysql> CHANGE MASTER TO MASTER_HOST='$MASTER_IP', MASTER_USER='root', MASTER_PASSWORD='123456', MASTER_LOG_FILE='mysql-bin.000010', MASTER_LOG_POS=122164543;         

5. 启动slave

     在slave中执行：
        
            mysql> start slave;

6. 查看状态

    在slave上查看同步状态：
    
            mysql> show slave status\G;
            
            ***
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            ***
            Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it

   如果slave正常运行则Slave_IO_Running和Slave_SQL_Running都必须是Yes。
   
至此我们的slave已经可以正常运行了，当然在实际执行上述步骤前我们需要手动将master上的数据copy至slave上，可以使用mysqldump或其它工具来完成。

### 延时复制

在上述的主从配置中master上的所有更改会立刻反映到slave上，如果我们使用slave做为备份，想像一下出现下述情况怎么办。

在某个下午小明刚刚从午睡中醒来，迷迷糊糊将鼠标移到了数据库管理软件上选中某个数据库点击了右键，并选择了delete，然后我们的数据库从master上消失了，在0.001秒后，数据库也从slave上消失了......

针对上述的情况mysql实现了延时复制的功能，可以人为的将salve滞后master一定的时间。在MySQL 5.6中支持了delay的功能，使用起来也比较简单：

    mysql> stop slave;
    mysql> change master to master_delay=86400;   //以秒为单位
    mysql> stop slave;
    
 文档可见，[点我](http://dev.mysql.com/doc/refman/5.6/en/replication-delayed.html)
    


