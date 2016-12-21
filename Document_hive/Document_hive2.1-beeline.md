title: hive2.1-beeline
notebook: 技术相关
tags: hive2.1, beeline

[TOC]

# Beeline 命令行
HiveServer2 支持命令行Beeline。 他是一个jdbc的客户端，基于[SQLLine CLi](http://sqlline.sourceforge.net/)

Beeline shell 可以运行在内嵌模式和远程模式俩种情况下。内嵌模式类似于Hive CLi，直接运行在hiveserver2 上。 而远程模式则通过Beeline连接一个远程的HiveServer2服务

### Beeline Example

    % bin/beeline
    Hive version 0.11.0-SNAPSHOT by Apache
    beeline> !connect jdbc:hive2://localhost:10000 scott tiger
    !connect jdbc:hive2://localhost:10000 scott tiger
    Connecting to jdbc:hive2://localhost:10000
    Connected to: Hive (version 0.10.0)
    Driver: Hive (version 0.10.0-SNAPSHOT)
    Transaction isolation: TRANSACTION_REPEATABLE_READ


    0: jdbc:hive2://localhost:10000> show tables;
    show tables;
    +-------------------+
    |     tab_name      |
    +-------------------+
    | primitives        |
    | src               |
    | src1              |
    | src_json          |
    | src_sequencefile  |
    | src_thrift        |
    | srcbucket         |
    | srcbucket2        |
    | srcpart           |
    +-------------------+
    9 rows selected (1.079 seconds)


在命令行上可以指定连接参数

    % beeline -u jdbc:hive2://localhost:10000/default -n scott -w password_file
    Hive version 0.11.0-SNAPSHOT by Apache

    Connecting to jdbc:hive2://localhost:10000/default

### Beeline Commands

| 命令   |  描述 |
|-------|-------|
| ```!<SQLLine command> ```|  SQLLine Commands相关命令见[地址](http://sqlline.sourceforge.net/.)

### Beeline Commands

| 命令 |  描述 |
|-----|-------|
|```reset```|   重置相关配置到默认值 |
|```set <key>=<value>```| 给指定的配置设置参数值， 如果变量拼写错误，beeline不会报错|
|```set``` |  列出所有被用户覆盖的配置信息 |
| ```set -v``` | 列出所有的hadoop和hive的配置信息|
|```add FILE[S] <filepath> <filepath>*``` ```add JARS[S] <filepath> <filepath>* ``` ``` add ARCHIVE[S] <filepath> <filepath>* ``` | 向distributed cache 的资源列表中添加一个或者多个file，jars，或者archive|
|```add FILE[s] <ivyurl> <ivyurls>*``` ``` add JARS[S] <ivyurl> <ivyurl>* ``` ```add ARCHIVE[S] <ivyurl> <ivyurl>*``` | 使用ivy url 想distributed cache的资源列表中添加一个或者多个file，jars或者archive
| ```list FILE[S]``` ```list JAR[S]``` ```list ARCHIVE[S]```| 列出已经被添加到distributed cache中的资源列表|
|```delete FILE[S] <filepath> <filepath>*``` ```delete JARS[S] <filepath> <filepath>* ``` ``` delete ARCHIVE[S] <filepath> <filepath>* ``` | 从distributed cache中删除资源
|```delete FILE[s] <ivyurl> <ivyurls>*``` ``` delete JARS[S] <ivyurl> <ivyurl>* ``` ```delete ARCHIVE[S] <ivyurl> <ivyurl>*``` | 从distributed cache中删除资源
| ```reload``` | 通过配置```hive.reloadable.aux.jars.path ```参数来改变其值，可以增加，删除和更新jar 文件|
| ```dfs <dfs command>``` |  执行dfs 命令 |
| ``` <query string> ``` |  执行hive查询并返回结果|


eg:

    0: jdbc:hive2://thadoop-uelrcx-host1:10000> dfs -ls /;
    +----------------------------------------------------------------------+--+
    |                              DFS Output                              |
    +----------------------------------------------------------------------+--+
    | Found 4 items                                                        |
    | drwxrwxrwx   - hadoop supergroup          0 2016-10-09 12:02 /hbase  |
    | drwxrwxrwx   - hadoop supergroup          0 2016-11-09 17:42 /test   |
    | drwxrwxrwx   - hadoop supergroup          0 2016-11-17 10:32 /tmp    |
    | drwxrwxrwx   - hadoop supergroup          0 2016-10-31 15:28 /user   |
    +----------------------------------------------------------------------+--+

### Beeline Command Options

| 参数 | 描述 | Usage |
|------|------|------|
|``-u <database url>`` |  用于连接的jdbc url.| beeline -u db_URL |
|```-r```|  重新连接最后一次连接的jdbc url| beeline -r|
|```-n <username>```|  beeline 连接时使用的用户名 |beeline -n valid_user|
|```-p <password>```|  beeline 连接时使用的密码  | beeline -p valid_password|
|```-d <driver class> ```| 使用的drive class |beeline -d driver_class|
|```-e <query>    ``` |查询语句 |  beeline -e "query_string" |
|```-f <file> ```|需要执行的脚本文件 | beeline -f  filepath |
|```-i (or)  --init <file or files>``` | 初始化脚本 | beeline -i /tmp/initfile |
|```-w (or) --password-file <password file>```| 读取密码的文件|        |
|```-a (or) --authType <auth type>```| 授权文件     |           |
|```--property-file <file>``` |  配置文件 | beeline --property-file /tmp/a|
|```--hiveconf property=value```| 设置特定的hive配置 | beeline --hiveconf prop1=value1|
|```--hivevar name=value```|  hive 变量名和值 | beeline --hivevar var1=value1 |
|```--color=[true/false]```|用于控制是否显示颜色，默认为false| beeline --color=true |
|```--showHeader=[true/false]```| 查询结果是否显示表头 | beeline --showHeader=false|
|```--headerInterval=ROWS```| 表头重复展现 | beeline --headerInterval=50 |
|```--fastConnect=[true/false]```|When connecting, skip building a list of all tables and columns for tab-completion of HiveQL statements (true) or build the list (false). Default is true.|beeline --fastConnect=false|
|```--autoCommit=[true/false]```|自动提交事务| beeline --autoCommit=true|
|```--verbose=[true/false]```|是否展现冗长的调试信息| beeline --verbose=true|
|```--showWarnings=[true/false]```|是否展现警告信息| beeline --showWarnings=true|
|```--showDbInPrompt=[true/false]```|是否显示当前的数据库| beeline --showDbInPrompt=true|
|```--showNestedErrs=[true/false]```|是否展现嵌套的错误| beeline --showNestedErrs=true|
|```--numberFormat=[pattern]```|格式化数据| beeline --numberFormat="#,###,##0.00"|
|```--force=[true/false]```|在遇到错误时是否继续执行| beeline--force=true |
|```--maxWidth=MAXWIDTH```| 控制台清理数据前的最大宽度| beeline --maxWidth=150|
|```--maxColumnWidth=MAXCOLWIDTH```| 最大列数|  beeline --maxColumnWidth=25 |
|```--silent=[true/false]```| 压缩展现的消息数量 | beeline --silent=true |
|```--autosave=[true/false]```| 自动保存偏好设置 | beeline --autosave=true |
|```--outputformat=[table/vertical/csv/tsv/dsv/csv2/tsv2]```| 结果展现的格式 | beeline --outputformat=tsv |
|```--truncateTable=[true/false]```| 在控制台展现超出长度会清楚掉 |         |
|```--delimiterForDSV= DELIMITER```| 分隔字符 |    |
|```--isolation=LEVEL```|  事务的隔离级别 |  beeline --isolation=TRANSACTION_SERIALIZABLE |
|```--nullemptystring=[true/false]```|  将pull转为字符串或者NULL           | beeline --nullemptystring=false |
|```--incremental=[true/false]```|  结果是否被缓存 |             |
|```--help```| 显示usage  信息 | beeline --help  |
