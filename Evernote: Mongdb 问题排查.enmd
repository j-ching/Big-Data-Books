---
title: Mongdb 问题排查
tags: []
notebook: 技术相关
---

  1. mongostat 查看mongo的各指标

  


insert query update delete getmore command % dirty % used flushes vsize res qr|qw ar|aw netIn netOut conn time 1. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:21+08:00 2. *0 *0 *0 *0 0 1|0 0.0 0.0 1 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:22+08:00 3. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:23+08:00 4. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:24+08:00 5. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:25+08:00 6. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:26+08:00 7. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:27+08:00 8. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:28+08:00 9. *0 *0 *0 *0 0 1|0 0.0 0.0 0 2.5G 41.0M 0|0 0|0 79b 17k 1 2016-08-10T23:21:29+08:00

2\. db.currentOp()

3\. db.serverStatus()
    
    
    
    {
        "host" : "1fe08fc00117",
        "version" : "3.0.9",
        "process" : "mongos",
        "pid" : NumberLong(18),
        "uptime" : 3660400,
        "uptimeMillis" : NumberLong("3660399710"),
        "uptimeEstimate" : 3637506,
        "localTime" : ISODate("2016-08-10T15:47:34.493Z"),
        "asserts" : {
            "regular" : 0,
            "warning" : 0,
            "msg" : 0,
            "user" : 279060,
            "rollovers" : 0
        },
        "connections" : {
            "current" : 12074,
            "available" : 7926,
            "totalCreated" : NumberLong(3728227)
        },
        "extra_info" : {
            "note" : "fields vary by platform",
            "heap_usage_bytes" : 51808568,
            "page_faults" : 192
        },
        "network" : {
            "bytesIn" : NumberLong("32295400422"),
            "bytesOut" : NumberLong("43987376572"),
            "numRequests" : NumberLong(402198394)
        },
        "opcounters" : {
            "insert" : 179105,
            "query" : 125255536,
            "update" : 218895,
            "delete" : 5670,
            "getmore" : 3,
            "command" : 276452245
        },
        "mem" : {
            "bits" : 64,
            "resident" : 1241,
            "virtual" : 13497,
            "supported" : true
        },
        "metrics" : {
            "commands" : {
                "<UNKNOWN>" : NumberLong(2),
                "addShard" : {
                    "failed" : NumberLong(5),
                    "total" : NumberLong(8)
                },
                "aggregate" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(84)
                },
                "authenticate" : {
                    "failed" : NumberLong(4),
                    "total" : NumberLong(4)
                },
                "buildInfo" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(3658407)
                },
                "collStats" : {
                    "failed" : NumberLong(1),
                    "total" : NumberLong(649)
                },
                "count" : {
                    "failed" : NumberLong(9),
                    "total" : NumberLong(17641776)
                },
                "createIndexes" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(22)
                },
                "createUser" : {
                    "failed" : NumberLong(1),
                    "total" : NumberLong(4)
                },
                "dbStats" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(63)
                },
                "delete" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(5671)
                },
                "drop" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(9)
                },
                "dropDatabase" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(1)
                },
                "dropUser" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(2)
                },
                "enableSharding" : {
                    "failed" : NumberLong(2),
                    "total" : NumberLong(4)
                },
                "eval" : {
                    "failed" : NumberLong(6),
                    "total" : NumberLong(752)
                },
                "getCmdLineOpts" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(31)
                },
                "getLastError" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(24)
                },
                "getLog" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(27)
                },
                "getnonce" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(60972)
                },
                "insert" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(167251)
                },
                "isMaster" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(119164442)
                },
                "listCollections" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(232)
                },
                "listDatabases" : {
                    "failed" : NumberLong(1),
                    "total" : NumberLong(161)
                },
                "listIndexes" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(647)
                },
                "listShards" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(2)
                },
                "ping" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(124823862)
                },
                "profile" : {
                    "failed" : NumberLong(1),
                    "total" : NumberLong(1)
                },
                "replSetGetStatus" : {
                    "failed" : NumberLong(569),
                    "total" : NumberLong(569)
                },
                "rolesInfo" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(1)
                },
                "saslContinue" : {
                    "failed" : NumberLong(61661),
                    "total" : NumberLong(7378461)
                },
                "saslStart" : {
                    "failed" : NumberLong(21),
                    "total" : NumberLong(3720082)
                },
                "serverStatus" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(45)
                },
                "shardCollection" : {
                    "failed" : NumberLong(1),
                    "total" : NumberLong(2)
                },
                "update" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(218895)
                },
                "usersInfo" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(8)
                },
                "whatsmyuri" : {
                    "failed" : NumberLong(0),
                    "total" : NumberLong(760)
                }
            },
            "cursor" : {
                "open" : {
                    "multiTarget" : NumberLong(0),
                    "singleTarget" : NumberLong(50),
                    "total" : NumberLong(50)
                }
            },
            "getLastError" : {
                "wtime" : {
                    "num" : 0,
                    "totalMillis" : 0
                }
            }
        },
        "ok" : 1
    }
    
    
    
http://blog.csdn.net/hjxhjh/article/details/12611195