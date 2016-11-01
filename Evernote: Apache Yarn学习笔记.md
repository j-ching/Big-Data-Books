title: Apache Yarn学习笔记
notebook: 技术相关
tags: yarn

Yarn 最基本的想法是将资源管理的功能与job调度与监控的功能分放在不同的领域。 以此为基础， 就产生了全局的ResourceManager(RM)和针对每个应用的ApplicationMaster(AM)。这里的应用包括单独的job或者多个job的有向无环图。
ResourceManager 和 NadeManager 构成了数据计算框架。 ResourcerManager 拥有决定系统中所有应用享有资源的最终权利。 NodeManager是每天机器的客户端框架， 它的职责是管理containers, 监控资源的使用情况(cpu, memory, disk, network)以及将这些状态上报给ResourceManager/Scheduler上。

每个应用都会有一个ApplicationMaster, 它的任务时向ResourceManager申请资源， 与NodeManager一同执行任务并对其监控。

ResourceManager有俩大部分： Scheduler和ApplicationManager

Scheduler的职责是为各种运行的应用分配资源, 使其满足大致相同的容量，队列等。 Scheduler是一个单纯的调度器，它并不包含对应用的监控和状态的跟踪， 同样，它也不负责由于应用错误或者硬件错误等造成的失败任务的重启。 Scheduler只负责应用资源的申请， 这种资源被抽象为Container(包含memory，cpu,disk, network等)

Scheduler包含一个附加的策略，主要负责对不同的queue，application中的资源进行分区。 目前example中使用的策略有 CapacityScheduler 和 FairScheduler

ApplicationsManager职责包括接收job的提交， 判断执行应用的ApplicationMaster的第一个container， 以及重启失败的ApplicationMaster。

每一个ApplicationMaster的职责是从Scheduler中申请合适的资源container， 跟踪他们的状态以及监控他们的进程。

1. 在hadoop2.x 的MapReduce 与之前版本维持了API compatibility 规则，也就意味了，之前的mrjob 不需要代码，只需要重新编译一下就可以在最新的yarn上运行
2. yarn依赖 ReservationSystem 支持了资源预定（resource reservation ）， 这就允许用户事先指定所需要的资源数目， 为比较重要的job的执行预留资源。

+ ResourceManager
	* schedule  根据容量，队列等将系统中的资源分配给各个正在运行的应用
	* ApplicationManager 接收job的提交， 判断执行应用的ApplicationMaster的第一个container， 以及重启失败的ApplicationMaster
+ NodeManager
	* container 对资源的封装，包括memory, disk, cpu, network等
	* AppMaster Scheduler中申请合适的资源container， 跟踪他们的状态以及监控他们的进程， 同时向ResourceManager上报这些状态

## 应用提交

构造YarnClient实例，当yarnClient启动后，后遭application context， 准备包含有applicationMaster的第一个container， 然后提交应用。 你需要提交如下信息： 应用运行时所需的本地可用的jars信息， 需要被执行的命令， 以及系统环境变量的设置。 你还需要指定ApplicationMaster加载的线程数。

ResourceManager在分配好的container中加载一个applicationMaster, applicationMaster 与yarn集群通讯，驱动应用程序执行。 ApplicationMaster是以异步的方式执行这些操作。
在application 加载的过程中， ApplicationMaster主要任务是：

1. 与Resourcemanager通讯，申请进一步的containers资源
2. 当container资源分配后，联系NodeManagers将应用加载到container

任务：

1. 通过AMRMClientAsync实例来异步执行任务， 实例需要在AMRMClientAsync.CallbackHandler中指明一个事件驱动的方法。 然后将该时间设置到client中
2. 加载一个可运行的实例，然后加载一个分配好的container资源。 加载container的同时， AM指定了ContainerLaunchContext， 来接收命令行和系统环境等的参数

在application执行过程中， ApplicationMaster通过NMClientAsync与NodeManagers通讯。 所有的containers通过NMClientAsync.CallbackHandler来驱动。 一个典型的回调驱动可以驱动客户端start， stop，status update和error。 ApplicationMaster也会将执行的进度通过AMRMClientAsync.CallbackHandler的getProgress()告诉给ResourceManager。

+ Client<-->ResourceManager
By using YarnClient objects.
+ ApplicationMaster<-->ResourceManager
By using AMRMClientAsync objects, handling events asynchronously by AMRMClientAsync.CallbackHandler
+ ApplicationMaster<-->NodeManager
Launch containers. Communicate with NodeManagers by using NMClientAsync objects, handling container events by NMClientAsync.CallbackHandler


## Writing a Simple Yarn Application

+ 构造并启动yarn-client

	```
	YarnClient yarnClient = YarnClient.createYarnClient();
	yarnClient.init(conf);
	yarnClient.start();

	```
