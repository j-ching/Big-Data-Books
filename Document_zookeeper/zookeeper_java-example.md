# A Simple Watch Client
我们开发一个简单的watch客户端，用于监控zk上的node的改变。

# 条件
客户端需要如下条件：

+ 需要如下参数
    + zk服务地址
    + znode名称，也就是用于监控的节点名称
    + 带参的执行程序
+ 获取关联znode的数据，启动执行
+ 如果znode改变，客户端重新获取数据，重新执行
+ 如果znode消失，客户端关闭执行

# 程序设计
一般的，zk应用分为俩部分，一部分用于维持连接，一部分用于监控数据。 在本实例中，Executor用于维护zk的连接，DataMonitor则用于监控zk树上的数据。同时Executor作为主线程，包含了主要的执行逻辑。

# Executor
executor 对象是主要的容器，它包含了zookeeper对象，DataMonitor 

     public static void main(String[] args) {
        if (args.length < 4) {
            System.err
                    .println("USAGE: Executor hostPort znode filename program [args ...]");
            System.exit(2);
        }
        String hostPort = args[0];
        String znode = args[1];
        String filename = args[2];
        String exec[] = new String[args.length - 3];
        System.arraycopy(args, 3, exec, 0, exec.length);
        try {
            new Executor(hostPort, znode, filename, exec).run();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public Executor(String hostPort, String znode, String filename,
            String exec[]) throws KeeperException, IOException {
        this.filename = filename;
        this.exec = exec;
        zk = new ZooKeeper(hostPort, 3000, this);
        dm = new DataMonitor(zk, znode, null, this);
    }

    public void run() {
        try {
            synchronized (this) {
                while (!dm.dead) {
                    wait();
                }
            }
        } catch (InterruptedException e) {
        }
    }
    
