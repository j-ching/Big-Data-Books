title: 渠道反作弊
notebook: 糖豆
tags: 机器学习,渠道反作弊

[TOC]

# 数据准备
  反作弊分析， 由用户的静态数据和行为数据出发，统计新增用户的关键特征，用来区分作弊用户与反作弊用户
## 数据源
### 用户注册数据
用户的注册信息, 是用户的基本信息，用于描述用户的静态特征(mysql的user表)

### 底层服务数据
底层服务数据，包括了用户在使用App过程中产生的访问底层数据接口的信息(hive的user_action表)

## 数据预处理
通过将用户注册数据和底层服务数据的整理，抽取出一个月的新增用户的关键特征， 作为我们的样本集

|  序号  |  字段             |  类型          |    字段描述   |
|-------|-------------------|---------------|--------------|
|0	    |u_diu				|string			|设备唯一号,android--imei,ios--IDFV              |
|1	    |is_reg				|int			|激活后是否注册：0未注册，1注册              |
|2	    |avatar				|int			|激活后是否设置头像：0否，1是              |
|3	    |type				|int			|激活后登录账号类型：1微信，2qq，3手机，0未注册              |
|4	    |signature			|int			|激活后是否设置个性签名：0否，1是              |
|5	    |space_pic			|int			|激活后是否设置背景空间：0否，1是              |
|6	    |follow_ucnt		|int			|激活后关注用户的个数              |
|7	    |manufacture		|string			|激活时的厂商              |
|8	    |device				|string			|激活时的ios:设备型号              |
|9	    |os_version			|string			|操作系统版本号              |
|10     |dic_f				|string			|激活时的客户端渠道代码              |
|11     |div_f				|string			|激活时的客户端版本              |
|12     |hour_f				|int			|激活的时间——小时              |
|13     |resolution			|string			|屏幕分辨率：高*宽              |
|14     |u_client			|string			|客户端类型              |
|15     |u_nettype			|string			|网络类型              |
|16     |u_province			|string			|用户省份              |
|17     |u_city				|string			|用户城市              |
|18     |u_netop			|string			|网络运营商              |
|19     |u_agent			|string			|客户端agent              |
|20     |u_xff				|string			|客户端ip              |
|21     |search_cnt			|int			|搜索次数              |
|22     |sug_cnt			|int			|搜索提示次数              |
|23     |homepage_cnt		|int			|首页打开次数              |
|24     |hp_hot_cnt			|int			|首页-热门打开次数              |
|25     |hp_dancemusic_cnt	|int			|首页-舞曲打开次数              |
|26     |hp_talent_cnt		|int			|首页-达人打开次数              |
|27     |hp_category_cnt	|int			|首页-分类打开次数              |
|28     |hp_new_cnt			|int			|首页-最新打开次数              |
|29     |hp_follow_cnt		|int			|首页-我的关注模块点击次数              |
|30     |hp_greatest_cnt	|int			|首页-每日精选模块点击次数              |
|31     |hp_live_cnt		|int			|首页-糖豆生活模块点击次数              |
|32     |hp_stage_cnt		|int			|首页-糖粉舞台模块点击次数              |
|33     |hp_more_cnt		|int			|首页-加载更多点击次数              |
|34     |feaddback_cnt		|int			|糖小豆反馈次数              |
|35     |showdance_cnt		|int			|秀舞打开次数              |
|36     |upload_cnt			|int			|上传视频次数              |
|37     |serv_cnt			|int			|接口调用次数              |
|38     |serv_unique_cnt	|int			|接口调用个数              |
|39     |vcnt				|int			|观看视频的个数              |
|40     |vv					|int			|观看次数              |
|41     |vp					|int			|播放进度              |
|42     |vst				|int			|观看时长              |
|43     |vc					|int			|评论次数              |
|44     |vsf				|int			|送花次数              |
|45     |vf					|int			|收藏次数              |
|46     |vs					|int			|分享次数              |
|47     |vd					|int			|下载次数              |
|48     |max_stepid			|int			|激活当日最大步数              |
|49     |agent_abnormal		|int			|user agent 是否异常，1是0否              |
|50     |tag				|int			|0:负样本 1:正样本               |


### 样本标量化
在模型训练时，要求将所有的类别性特征转为数据值特征。spark提供了多种用于特征转换的模型，这里我们使用```StringIndexer```模型， 他会将类别行的特征转为从```0```开始的数值型特征， 对应关系由类别出现的频率来决定， 如出现频率最多的值会转为```0```, 出现频率第二的值会转为```1```, 以此类推


将如下的类别型特征转化为数值型特征

|类别特征         |数值特征  |
|-------|-----------|
|manufacture  |   manufacture_index  |
|device       |   device_index  |
|os_version   |   os_version_index  |
|dic_f        |   dic_f_index  |
|div_f        |   div_f_index  |
|resolution   |   resolution_index  |
|u_client     |   u_client_index  |
|u_nettype    |   u_nettype_index  |
|u_province   |   u_province_index  |
|u_city       |   u_city_index  |
|u_netop      |   u_netop_index  |
|u_agent      |   u_agent_index  |
|u_xff        |   u_xff_index  |

将特征中所有的数值型特征转为Double型

|  字段 |   数据类型  |
|--------|----------|
|is_reg | toDouble |
|avatar | toDouble |
|type | toDouble |
|signature | toDouble |
|space_pic | toDouble |
|follow_ucnt | toDouble |
|manufacture_index | toDouble |
|device_index | toDouble |
|os_version_index | toDouble |
|dic_f_index | toDouble |
|div_f_index | toDouble |
|hour_f | toDouble |
|resolution_index | toDouble |
|u_client | toDouble |
|u_nettype_index | toDouble |
|u_province_index | toDouble |
|u_city_index | toDouble |
|u_netop_index | toDouble |
|u_agent_index | toDouble |
|u_xff_index | toDouble |
|search_cnt | toDouble |
|sug_cnt | toDouble |
|homepage_cnt | toDouble |
|hp_hot_cnt | toDouble |
|hp_dancemusic_cnt | toDouble |
|hp_talent_cnt | toDouble |
|hp_category_cnt | toDouble |
|hp_new_cnt | toDouble |
|hp_follow_cnt | toDouble |
|hp_greatest_cnt | toDouble |
|hp_live_cnt | toDouble |
|hp_stage_cnt | toDouble |
|hp_more_cnt | toDouble |
|feaddback_cnt | toDouble |
|showdance_cnt | toDouble |
|upload_cnt | toDouble |
|serv_cnt | toDouble |
|serv_unique_cnt | toDouble |
|vcnt | toDouble |
|vv | toDouble |
|vp | toDouble |
|vst | toDouble |
|vc | toDouble |
|vsf | toDouble |
|vf | toDouble |
|vs | toDouble |
|vd | toDouble |

### 构造特征向量
将标量化之后的特征组合为特征向量, 形成标准的LIBSVM数据文件, 作为标准的样本集

|  labels  | features |
|----------|----------|
|  1.0 | 7:3.0 8:421.0 9:8.0 10:8.0 11:19.0 12:1.0 13:2.0 15:4.0 16:51.0 18:620.0 19:423735.0 22:1.0 29:2.0 36:53.0 37:15.0 38:4.0 39:4.0 40:22.0 41:133.0 |
|  1.0 | 7:3.0 8:68.0 10:1.0 11:22.0 12:2.0 13:2.0 15:20.0 16:284.0 17:3.0 18:64.0 19:157721.0 22:1.0 24:1.0 36:25.0 37:13.0 38:2.0 39:2.0 40:15.0 41:72.0|
|......| ......|

# 模型选择与训练
## 模型选择
### 常用的分类模型
反作弊的场景是典型的分类问题，可以使用的分类问题有如下几种

| 模型 | 中文名 |
|-----|-----|
| Logistic regression  |  逻辑回归    |
| Decision tree classifier | 决策树    |
| Random forest classifier |   随机森林  |
| Gradient-boosted tree classifier |     |
| Multilayer perceptron classifier |     |
| One-vs-Rest classifier |   一对多SVM分类？  |
| Naive Bayes |  朴素贝叶斯    |

### 模型的比较
目前仅选用```决策树```来对数据做分析， 稍后会对各种分类模型做测试，选择最优的模型

## 模型训练及检验
样本数据共有```576056```， 正样本```312247```， 负样本```263809```, 将整个样本集7-3分开，7份用于训练模型， 剩余3份用于对模型进行检验

| 决策树模型参数      |  检验样本数| 正样本数| 负样本数| 预测正 | 预测负 | 正预测正 | 负预测正| 负预测负| 正预测负| 准确率 |精确度   |  召回率 | F 值 |
|-------------|-----------|--------|-------|-------|-------|---------|--------|-------|--------|-------|-------|----------------|
|(gini,10,10) | 172793    | 93453  | 79340 | 94400 | 78393 | 85290   | 9110   | 70230 | 8163   |0.9000364598102932 | 0.903495763 |0.912651279 |  0.8999883453209875   |
|(gini,10,300)| 172966    | 93553  | 79413 | 93709 | 79257 | 85253   | 8456   | 70957 | 8300   |0.9031254697454991 | 0.909763203 |0.911280237 |  0.903118198576006    |
|(gini,20,10) | 172982    | 93470  | 79512 | 93250 | 79732 | 86917   | 6333   | 73179 | 6553   |0.9255067001190875 | 0.932085791 |0.929891944 |  0.9255142720996273   |
|(gini,20,300)| 172886    | 93425  | 79461 | 93873 | 79013 | 87298   | 6575   | 72886 | 6127   |0.9265296206748956 | 0.929958561 |0.934417982 |  0.9265136389596025   |
|(entropy,10,10) | 172885 | 93442  | 79443 | 93984 | 78901 | 85057   | 8927   | 70516 | 8385   |0.8998640714926107 | 0.905015747 |0.910265191 |  0.8998374794289492   |
|(entropy,10,300)| 172861 | 93454  | 79407 | 93923 | 78938 | 85086   | 8837   | 70570 | 8368   |0.9004691630847906 | 0.90591229  |0.910458621 |  0.9004463251503265   |
|(entropy,20,10) | 172788 | 93461  | 79327 | 93456 | 79332 | 86491   | 6965   | 72362 | 6970   |0.9193520383359955 | 0.92547295  |0.925423439 |  0.9193522304510111   |
|(entropy,20,300)| 172919 | 93394  | 79525 | 93657 | 79262 | 86815   | 6842   | 72683 | 6579   |0.9223856256397504 | 0.926946197 |0.929556503 |  0.922375913239748    |

决策树参数：
+ **impurity** 计算不纯度的度量方式， 有gini不纯度和熵(entropy)
+ **depth** 树的最大深度
+ **bins** 决策树的规则集称为桶， 桶的个数

模型度量指标
+ **准确率(accuracy)** 是指预测正确(预测为正且确实为正或者预测为负且确实为负)的样本占所有预测样本的比例
+ **精确度(precision)** 是指预测为正且确实为正的样本数占所有预测为正的样本数的比例
+ **召回率(recall)** 是指预测为正且确实为正的样本数站所有本来为正的样本数的比例
+ **F值** F值 = 精确度 * 召回率 * 2 / (精确度 + 召回率) 精确度和召回率的调和平均值，用于整体反应预测的结果

从检验的结果来看， (gini,20,10)和(gini,20,300) 俩种参数情况下，效果要比其他几种情况下要好，(gini,20,300) 虽然要比 (gini,20,10)在准确率，召回率上都要高一些，但由于bins=300的时候，计算的复杂程度会大大增加，效率较低，所以选```  (gini,20,10)```的模型

# 问题和改进
1. 目前虽然对于类别性特征做了标量化，转为了数值型特征，并没有对其做特殊的处理，整个模型训练中都将其作为数值特征处理了， 给模型带来了偏差和不可解释性， 需要对此做处理
2. 单一的决策树模型，并不能完全的反应事实特征， 会引入随机森林等更多分类模型，来对效果做比较
