# flannel之启动

### 启动参数
etcd-endpoints:指定etcd集群的路径
etcd-prefix:指定etcd的文件路径
etcd-keyfile:指定SSL key file
etcd-certfile
etcd-cafile
etcd-username
listen:指定监听的端口
remote:作为client运行时，连接的远程服务器的ip:port
remote-keyfile:SSL key
remote-certfile:
remote-cafile:
kube-subnet-mgr:使用Kubernetes API申请子网信息，而不是使用etcd或者flannel-server

##总入口函数做的几件事情：
####1.解析命令行参数，如果在命令行参数中没用指定启动的参数，flannel会尝试从环境变量中获取启动参数   
####2.配置获取子网的途径：三个途径：1.通过远程的flannel server,2.通过kubernetes API;3.通过ETCD  
####3.注册SIGINT和SIGTERM信号，这里注册信号量的机制和posix c不同，首先需要创建一个os.Signal的chan,然后，让这个channel跟系统的SIGINT和SIGTERM相关联。如果系统向fannel发起SIGINT和SIGTERM信号，这个channel就会捕捉到，一旦捕捉到，flannel进程要进行一些收尾动作，然后退出。这里需要重点理解golang是怎么处理系统的信号量的。
####4.启动flannel:flannel有两种启动模式：1.作为普通的client,2.作为flannel server;如果是作为flannel client启动，那么需要将第二步获取的获取子网途径加入到runFunC中。在paas2.0的环境中flannel一般是作为client运行的。

## flannel client模式
前文提到的flannel获取子网信息的三个途径：1.flannel server;2.kubernetes API；3.etcd;
