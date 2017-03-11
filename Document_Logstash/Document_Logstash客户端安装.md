[TOC]


# LogStash Agent 安装配置

  1. bin/logstash -e '配置信息'
  2. bin/logstash -f 配置文件

# LogStash Pipeline

LogStash Pipeline 包括一个或者多个input, filter 和output。 启动logstash实例的时候，可以通过-f 来指定pipeline 的配置文件 pipeline中input和output是必须要有的，filter是可选的。 input插件负责从数据源获取数据，filter插件按要求处理数据，output插件将处理好的数据写到相应的目标存储。


    # The # character at the beginning of a line indicates a comment. Use
    # comments to describe your configuration.
    input {
    }
    # The filter part of this file is commented out to indicate that it is
    # optional.
    # filter {
    #
    # }
    output {
    }



上述的代码框架只包含了input 和 output的空实现，并没有包含实际的功能。 我们可以自己去定义input， filter和output， 来实现我们自己想要实现的功能

**这里我们实现一个将apache 日志收容到es的例子**

## 配置一个File类型的input插件


    input {
        file {
            path => "/path/to/logstash-tutorial.log"
            start_position => beginning
            ignore_older => 0
        }
    }



  1. file input默认监控文件信息的变化，与unix 中的tail -f 类似, 收集整个文件的内容，也可以配置start_position指定数据收集的开始位置。
  2. file input默认会忽略掉一天之前的修改内容， 也可以通过修改配置ignore_older关闭掉这个限制 path 指向你所要收集的文件路径

# 配置一个agent， 数据来自nginx的access log,  筛选出request请求日志后，分别输出到kafka用于实时监控展现和hdfs用于离线计算和展现

1. 在logstash客户端安装webhdfs插件 ``` bin/logstash-plugin install logstash-output-webhdfs ```
2. 配置文件logstash.conf

	input {
	    file {
	        path => [ "/data/log/nginx/access.log" ]
	        start_position => "end"
	        codec => "json"
	    }
	}
	filter{
	    urldecode {
	        field=>"request"
	    }
	}
	output {
	    kafka {
	       bootstrap_servers => "ukafka-ghz0cc-1-bj04.service.ucloud.cn:9092,ukafka-ghz0cc-2-bj04.service.ucloud.cn:9092,ukafka-ghz0cc-3-bj04.service.ucloud.cn:9092,ukafka-ghz0cc-4-bj04.service.ucloud.cn:9092"
	       topic_id => "logstash-tv"
	    }

	    webhdfs {
	       host => "thadoop-uelrcx-host1"
	       path => "/user/logstash/dt=%{+YYYY-MM-dd}/10_19_38_10/logstash-%{+HH}.log"
	       user => "hadoop"
	       codec => json
	    }
	}
3. logstash默认将数据源读取的游标值记录在$HOME/.sincedb*, 如果希望从头开始读，需要删除该文件，或者重新设置 ```sincedb_path```
4. logstash日期种类


