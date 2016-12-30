# 网络攻击的方式
+ **DOS** 是Denial of Service的简称，即拒绝服务，造成DoS的攻击行为被称为DoS攻击，其目的是使计算机或网络无法提供正常的服务。最常见的DoS攻击有计算机网络带宽攻击和连通性攻击
+ **R2L** unauthorized access from a remote machine, e.g. guessing password;
+ **U2R**  unauthorized access to local superuser (root) privileges, e.g., various ``buffer overflow'' attacks;
+ **probing** surveillance and other probing, e.g., port scanning.

# 特性分类

## 网络连接的基本特征

序号    |     特征        |    描述                                         |        类型
-------|----------------|-------------------------------------------------|----------------
1      | duration       |  连接长度(秒)                                     |  连续
2      | protocol_type  |协议类型，如 TCP，UDP                               | 离散
3      | service        |目的站网络服务，如 HTTP，Telnet 等                   | 离散
4      | flag           |连接的状态 (正常或错误)                              | 离散
5      | src_bytes      |源到目的站的数据字节数                               |  连续
6      | dst_bytes      |目的站到源站的数据字节数                              | 连续
7      | land           |如果连接从/到相同的主机/端口，则 为 1;否则为 0          | 离散
8      | wrong_fragment |错误段 (fragment) 的数量                            | 连续
9      | urgent         |urgent 包的数量                                    | 连续

## 网络连接基于内容的特征