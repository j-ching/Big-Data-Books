[TOC]

## 通过hive 外部表(external table) 来做关联

### user_core_track

```

CREATE EXTERNAL TABLE user_core_track_hbase (rowkey String, d_diu string, d_session string, d_step string, d_time string, d_event String, dt String)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:d_diu,f:d_session,f:d_step,f:d_time,f:d_event,f:dt")
TBLPROPERTIES("hbase.table.name" = "user_core_track");

```


```

insert overwrite table user_core_track_hbase
select concat(d_diu, '_', d_session, '_', d_step), d_diu,d_session,d_step,d_time,d_event,dt
from user_core_track where dt = '2016-07-14’ and d_diu <> ''
limit 10

```


```
CREATE EXTERNAL TABLE user_info_hbase(
rowkey String,
u_diu string,
u_diu2 string,
u_diu3 string,
u_uid string,
u_uuid string,
u_hash string,
u_xinge string,
u_token string,
u_div_f string,
u_div string,
u_dic_f string,
u_dic string,
u_client string,
u_timestamp_f string,
u_timestamp string,
u_netop_f string,
u_netop string,
u_province_f string,
u_province string,
u_city_f string,
u_city string,
u_manufacture string,
u_model string,
u_device string,
u_width string,
u_height string,
u_fresh string,
u_active string,
u_tag string,
u_bigger_json string,
dt string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:u_diu,f:u_diu2,f:u_diu3,f:u_uid,f:u_uuid,f:u_hash,f:u_xinge,f:u_token,f:u_div_f,f:u_div,f:u_dic_f,f:u_dic,f:u_client,f:u_timestamp_f,f:u_timestamp,f:u_netop_f,f:u_netop,f:u_province_f,f:u_province,f:u_city_f,f:u_city,f:u_manufacture,f:u_model,f:u_device,f:u_width,f:u_height,f:u_fresh,f:u_active,f:u_tag,f:u_bigger_json,f:dt")
TBLPROPERTIES("hbase.table.name" = "user_info");
```



### user_info

```
insert overwrite table user_info_hbase
select u_diu,u_diu,u_diu2,u_diu3,u_uid,u_uuid,u_hash,u_xinge,u_token,u_div_f,u_div,u_dic_f,u_dic,u_client,u_timestamp_f,u_timestamp,u_netop_f,u_netop,u_province_f,u_province,u_city_f,u_city,u_manufacture,u_model,u_device,u_width,u_height,u_fresh,u_active,u_tag,u_bigger_json,dt
from user_info
where u_diu <> ''
and dt = '2016-07-28'
```

**导入的问题： hbase表会重复**

### user_core_act

```
CREATE EXTERNAL TABLE user_core_act_hbase (
rowkey string,
d_diu string,
d_dt string,
m_interface_count string,
m_interface_distinct string,
m_search_all string,
m_searh_top string,
m_searh_radar string,
m_searh_tag string,
m_online_play string,
m_offline_play string,
m_comment string,
m_upload string,
m_download string,
m_like string,
m_praise string,
m_share string,
m_txd string,
m_space string,
m_shot string,
m_team_all string,
m_follow string,
m_unfollow string,
m_reverse0 string,
m_reverse1 string,
m_reverse2 string,
m_reverse3 string,
m_reverse4 string,
m_reverse5 string,
m_reverse6 string,
m_reverse7 string,
m_reverse8 string,
m_reverse9 string,
dt string
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:d_diu,f:d_dt,f:m_interface_count,f:m_interface_distinct,f:m_search_all,f:m_searh_top,f:m_searh_radar,f:m_searh_tag,f:m_online_play,f:m_offline_play,f:m_comment,f:m_upload,f:m_download,f:m_like,f:m_praise,f:m_share,f:m_txd,f:m_space,f:m_shot,f:m_team_all,f:m_follow,f:m_unfollow,f:m_reverse0,f:m_reverse1,f:m_reverse2,f:m_reverse3,f:m_reverse4,f:m_reverse5,f:m_reverse6,f:m_reverse7,f:m_reverse8,f:m_reverse9,f:dt")
TBLPROPERTIES("hbase.table.name" = "user_core_act");
```

亲，大家都在看%s老师的%s(舞蹈),你也快来看看吧>>

```
insert overwrite table user_core_act_hbase
select concat(d_diu, '_', d_dt), d_diu,d_dt,m_interface_count,m_interface_distinct,m_search_all,m_searh_top,m_searh_radar,m_searh_tag,m_online_play,m_offline_play,m_comment,m_upload,m_download,m_like,m_praise,m_share,m_txd,m_space,m_shot,m_team_all,m_follow,m_unfollow,m_reverse0,m_reverse1,m_reverse2,m_reverse3,m_reverse4,m_reverse5,m_reverse6,m_reverse7,m_reverse8,m_reverse9,dt
from user_core_act where dt = '2016-07-19' and d_diu <> ''
limit 1;
```

### push_video_recommend_online

```
CREATE EXTERNAL TABLE push_video_recommend_online
(
rowkey String,
xinge string,
client int,
vid int,
vid_name String,
vuid_name String,
v_score double,
vchild_category String,
dt String
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:xinge,f:client,f:vid,f:vid_name,f:vuid_name,f:v_score,f:vchild_category,f:dt")
TBLPROPERTIES("hbase.table.name" = "push_video_recommend_online");
```

```
insert overwrite table push_video_recommend_online
select concat('2016-08-31', '_', xinge), xinge,client,vid,vid_name, v_uid_name, v_score, v_child_category, '2016-08-31'
from da.push_video_recommend
```

### test_push_video_recommend_final

```
CREATE EXTERNAL TABLE test_push_video_recommend_final
(
rowkey String,
orignrowkey String,
xinges string,
client int,
vid int,
vid_name String,
vuid_name String,
dt String
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:xinge,f:xinges,f:client,f:vid,f:vid_name,f:vuid_name,f:dt")
TBLPROPERTIES("hbase.table.name" = "test_push_video_recommend_final");
```

```
insert overwrite table  test_push_video_recommend_final
select concat('2016-09-02', '_', rowkey), xinges,client,vid,vid_name, v_uid_name, '2016-09-02'
from da.push_video_recommend
where xinge in (
'7972e6ca4a2cae3c5f5b5eb7491708f4c3039193241fd5a378e3b579f1459e02',
'df3be3ebf5f897dd326c4ba6d14a32ef6f334c8f65d96318dd1d220cb543ce86',
'83570c73f3c0258d6c737018a91f24d3749c5cc42a56459ef708b21f1b30d2c6',
'52ae5eb6a823ddddead0f87fc882bf656752e78233edb7a58f0bbf844be7dbe0',
'c692e9d1ed725faf48d1999b873ce52501ec28e9',
'39dfda1281de44fd5703058c2b64e76a95bcf0eb5af0d823ba0a826d782fec32',
'4f40e3881cb4203b2642e88d948f8fa765cf27c6f58c35cd6f705b2bf935db38',
'ba2fd796dfa058574ef9ba80d55e87dc0ecc1623',
'ffd26e18b0563a6e49649eb262645af6471fcd22',
'ed53ea674108003b4cbcaa73c7ce092a67136d18')
```

```
insert overwrite table  test_push_video_recommend_final
select concat('2016-09-08', '_', rowkey), rowkey, xinges,client,vid,vid_name, v_uid_name, '2016-09-08'
from(
select
	floor(rank/1000) group_id,
	vid,
	vid_name  ,
	v_uid_name ,
	client ,
	max(xinge) rowkey,
	concat_ws(',',collect_set(xinge)) xinges
from
	(
	select
	xinge ,
	client ,
	vid,
	vid_name  ,
	v_uid_name ,
	ROW_NUMBER() OVER  (PARTITION BY vid,client ORDER BY xinge DESC) rank
	from push_video_recommend
	where xinge in (
		'44dc25fcefe0f9bc4d608007dc9dd77de6b76591',
		'412db9331e1cf50b47d8b492ea69de2a81b7d0a9',
		'454d483617096ea54713bb214f19bfde53773eea',
		'90f3314ecc33a58d491e99289c3af696c11dc8c2',
		'dd810d94d044b0ab41d28bfbff887d434deb3b11',
		'83570c73f3c0258d6c737018a91f24d3749c5cc42a56459ef708b21f1b30d2c6')
	) a
	group by
	floor(rank/1000)  ,·
	vid,
	vid_name  ,
	v_uid_name ,
	client) t
;

```

```
CREATE EXTERNAL TABLE push_video_recommend_online_20160904
(
rowkey String,
orignrowkey String,
xinges string,
client int,
vid int,
vid_name String,
vuid_name String,
dt String
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:xinge,f:xinges,f:client,f:vid,f:vid_name,f:vuid_name,f:dt")
TBLPROPERTIES("hbase.table.name" = "push_video_recommend_online_20160904");
```

```
insert overwrite table  push_video_recommend_online_20160908
select concat('2016-09-08', '_', rowkey),rowkey, xinges,client,vid,vid_name, v_uid_name, '2016-09-08'
from push_video_recommend_final_daily

```

## hbase_push_all_device

```
CREATE EXTERNAL TABLE hbase_push_all_device
(
rowkey String,
xinge String,
xinges string,
client int
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:xinge,f:xinges,f:client")
TBLPROPERTIES("hbase.table.name" = "hbase_push_all_device");

```

```
insert overwrite table  hbase_push_all_device
select rowkey,rowkey, xinges,client
from push_all_device

```

```
CREATE EXTERNAL TABLE test_push_video_recommend_final_new
(
rowkey String,
orignrowkey String,
xinges string,
client int,
vid int,
vid_name String,
vuid_name String,
dt String
)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:xinge,f:xinges,f:client,f:vid,f:vid_name,f:vuid_name,f:dt")
TBLPROPERTIES("hbase.table.name" = "test_push_video_recommend_final_new");
```

```
insert overwrite table  test_push_video_recommend_final_new
select concat('20160908', '_', orignrowkey),orignrowkey, xinges,client,vid,vid_name, vuid_name, '20160908'
from test_push_video_recommend_final

```


