# 记一次：恢复Mysql数据

0x06：项目起初对框架不熟悉而导致 

<!-- more -->

前言：起初想通过全局sql日志恢复，后面查看mysql配置并没有打开全局日志，但是打开了 bin-log,及二进制日志，平均每个1G左右。

开始方案：据官方了解，bin-log 可以通过 mysqlbinlog 工具解析
启动方案：
- 通过 解析base64`-v --base64-output=decode-rows`并过滤只提取 `--database` 库的日志.
    ```shell
    mysqlbinlog -v --base64-output=decode-rows --database=databasename mysql-bin.000016 > 16.log
    
- 当前目录下的 `mysql-bin.000016` 并输出到当前 `16.log`文件中.

    ```text
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
    /*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
    DELIMITER /*!*/;
    # at 4
    #200115  0:52:33 server id 10100332  end_log_pos 123 CRC32 0xf47ac891 	Start: binlog v 4, server v 5.7.21-enterprise-commercial-advanced-log created 200115  0:52:33
    # at 123
    #200115  0:52:33 server id 10100332  end_log_pos 154 CRC32 0xcd2ae247 	Previous-GTIDs
    # [empty]
    # at 154
    .....
    
- 最后提取`INSERT INTO `语句大概是这样的：

    ```text
    # at 507657591
    # ...
    ### INSERT INTO `database`.`tabel`
    ### SET
    ###   @1=146
    ###   @2=61
    ###   @3=NULL
    ###   .....
    
- 上面的 `@` 符号后面的数字就是字段的下标，从`1`开始.
- 最后解析了`76G`日志
  
后记：凡事三思而后行!
  
