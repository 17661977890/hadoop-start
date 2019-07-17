# hadoop-start
再centos 7 虚拟机搭建伪分布式hadoop集群 版本hadoop 2.7.7

初识hadoop:
Hadoop实现了一个分布式文件系统（Hadoop Distributed File System），简称HDFS。有高容错性的特点，并且设计用来部署在低廉的（low-cost）硬件上；而且它提供高吞吐量来访问应用程序的数据，适合那些有着超大数据集（large data set）的应用程序。

Hadoop的框架最核心的设计就是：HDFS和MapReduce。HDFS为海量的数据提供了存储，而MapReduce则为海量的数据提供了计算。

### 准备工作：
* （1）虚拟机centos7 和 jdk 的安装：（自行百度--桥接模式，所需镜像在下面百度网盘中）

**链接：https://pan.baidu.com/s/1LsXEGHr3ejcbFEczG8ai4A 
  提取码：iagq 
  复制这段内容后打开百度网盘手机App，操作更方便哦**
  
* 安装注意事项： 记得安装时，各个设置（特别是桌面GUI配置，方便操作，也可以顺便自动下载openjdk，或者自行下载）
* 不管是默认还是自行下载的jdk 都要 配置环境变量 
* 切换管理员 su 输入你安装系统时配置的root密码 
* 编辑文件，配置java环境变量 vi etc/profile ,* 参考链接 ： https://github.com/17661977890/interview-docs/blob/master/docs/hadoop/hadoop01.md
* 注意要修改位自己安装jdk的目录，如果是默认安装的，可以通过命令寻找：
* 1.which java (定位到java的可执行路径) 
* 2.ls -lrt /usr/bin/java 
* 3.ls -lrt /etc/alternatives/java -----去后面绿色路径（截止到bin之前就可以） 


#### hadoop 2.7.7 的安装与配置：
* 安装地址 ： http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.7.7
* 本地解压缩文件，并重命名为 hadoop，方便使用。拷贝到centos系统中，hadoop 目录的完整路径是“/usr/local/hadoop”。
* 配置hadoop的环境变量 vi /etc/profile 还是参考上个链接
* 输入 hadoop 命令验证是否成功-----如果你配置玩环境变量还是找不到命令
* 执行一下这个命令：export PATH=$PATH:/usr/local/hadoop/bin---这样重启系统=>命令：reboot 以后,会发现环境变量失效,因为只适用于当前终端,所以怎么办呢,看下面,熟读文章

**当时因为这个环境变量晕了很久,我明明配置了,但是为什么hadoop 命令找不到 ,每次要执行export PATH=$PATH:/usr/local/hadoop/bin 才可以.
原因找到了:
(1)得益于以为大佬的推荐文章:https://blog.csdn.net/sfhawx/article/details/49969321  
给你讲述 login shell 和 no login shell 的区别,加载不同文件来读取环境配置变量,以及如何正确使用 切换root 命令.
(2)确实配置有误,配置修改参考连接:https://www.cnblogs.com/liulala2017/p/9519705.html
在 ~/.bashrc 或 /etc/profile这个文件修改 export PATH=<你要加入的路径1>:<你要加入的路径2>: ...... :$PATH    $PATH这个是放在最后的,不像上一个当前终端使用的在前边**

* 废话不说,正确配置如下:
  
  如果你是su 切换root 用户,non-login shell只会读取~/.bashrc这个文件,不会读取/etc/profile,那就修改这个文件,并且source一下,修改内容如下
  如果你是su - 切换root 用户,login shell 那就会读取/etc/profile  修改此文件即可,然后source 但是因为系统整体性,不是必要不用改,
  如果你是直接用普通用户,hadoop 命令是不行的,不会读取~/.bashrc文件. 用su -l 普通用户名切换普通用户,才会读取~/.bashrc 但是会有权限问题
   --日志:-bash: /usr/local/hadoop/bin/hadoop: 权限不够
  ![image]() 
```bash
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
export HADOOP_HOME=/usr/local/hadoop
export JAVA_BIN=$JAVA_HOME/bin
export JAVA_LIB=$JAVA_HOME/lib
export CLASSPATH=.:$JAVA_LIB/tools.jar:$JAVA_LIB/dt.jar
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$JAVA_HOME/bin.:$HADOOP_HOME/bin.:$HADOOP_HOME/sbin:$PATH 

# 之前最后PATH配置一直是export PATH=$PATH:$JAVA_HOME/bin.:$HADOOP_HOME/bin.:$HADOOP_HOME/sbin  --这是不对的
```


#### 修改hadoop的配置文件，还是参考上个链接 
   
  * hadoop 配置文件默认是本地模式，我们修改6个配置文件，这些文件都位于/usr/local/hadoop/etc/hadoop 目录下。
  * **下面的配置中的localhost 可能需要改为虚拟机的ip或者主机名，貌似localhost哪里会有问题，我启动时候都该我自己的ip了,如果是动态可以将ip改为静态的**
  * 注意配置虚拟机本机ip 与 主机名的映射关系cat /etc/hosts 查看有没有配置，没有则配置 vi /etc/hosts 格式：ip 主机名  
  * 没有上述配置通常执行hdfs 的shell 命令会报错：访问hdfs主节点失败Call From xxxx to xxxx:9000 failed
  * 主机名查询命令 ： hostname
  
```bash
不了解 vi vim 文件编辑命令的，这里大概说一下：vi 文件名：进入编辑文件 i 开始编辑  esc 退出编辑模式 :wq 保存并退出 :w 保存 :q 只是退出

# vi hadoop-env.sh  
设为自己的jdk路径
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
 
# vi core-site.xml

 <configuration>
    <property>
       <name>fs.defaultFS</name>
       <value>hdfs://localhost:9000</value>
       <description>HDFS 的访问路径</description>
    </property>
    <property>
       <name>hadoop.tmp.dir</name>
       <value>/usr/local/hadoop-2.7.7/tmp/hadoop</value>
       <description>hadoop 的运行临时文件的主目录</description>
    </property>
 </configuration>

# vi hdfs-site.xml
  
 <configuration>
    <property>
      <name>dfs.replication</name>
      <value>1</value>
    </property>
    <property>
          <name>dfs.client.use.datanode.hostname</name>
          <value>true</value>
  </property>
  <property>
          <name>dfs.datanode.use.datanode.hostname</name>
          <value>true</value>
  </property>
  <property>
      <name>dfs.permissions</name>
      <value>false</value>
  </property>
 </configuration>



# vi mapred-site.xml
# 注意事项： 2.7.7版本 MapReduce 配置文件是 mapred-site.xml.template 要重命名为 mapred-site.xml (命令参考：mv a b)

  <configuration>
    <property>
        <name>mapred.job.tracker</name>
        <value>localhost:9001</value>
        <description>JobTracker 的访问路径</description>
     </property>
     <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
  </configuration>

# vi yarn-site.xml  (sun.com是我的主机名，你们要设置自己的)

  <configuration>
         <property>
             <!--指定yarn的老大resourcemanager的地址-->
             <name>yarn.resourcemanager.hostname</name>
             <value>sun.com</value>
         </property>
         <property>
             <!--NodeManager获取数据的方式-->
             <name>yarn.nodemanager.aux-services</name>
             <value>mapreduce_shuffle</value>
         </property>
</configuration>

# vi slaves 设置主机名（之前是localhost，貌似可以不改）
#slaves这个文件指定DataNode在哪台机器上。这个文件在hadoop-2.7.7中存在，但是在hadoop-3.1.0里没有这个文件，我怀疑是该文件改名为workers了。当搭建分布式hadoop集群时，需要修改这个文件，配置DataNode在哪台机器上。

  sun.com
  
```
#### （4）启动hadoop集群:

* ==> 启动之前，格式化文件:你安装hadoop目录（usr/local/hadoop/）下执行命令 bin/hdfs namenode -format
  没有报错日志标识成功：
  打印日志成功：Storage directory /usr/local/hadoop-2.7.7/tmp/dfs/name has been successfully formatted
  
  ![image](https://github.com/17661977890/hadoop-start/blob/master/image/%E5%9B%BE%E7%89%871.png)
  
* ==> 一次性启动：还是安装目录下执行命令：  sbin/start-all.sh 启动 hadoop

* ==> 输入命令 jps  查看进程，如图所示，表示正在启动，如果缺少namenode 单独启动一下：hadoop-daemon.sh start namenode
* ==> 踩完各种坑后正常日志如下：(上述少节点启动问题不会出现)
  
  ![image](https://github.com/17661977890/hadoop-start/blob/master/image/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190717114838.png)
  
  可以看到上图正在启动进程，分 别是 namenode、datanode、secondarynamenode、jobtracker、tasktracker，一共 5 个，待执行 完毕后，并不意味着这 5 个  进程成功启动，上面仅仅表示系统正在启动进程而已
虚拟机内部访问 http://192.168.1.191:50070 查看hadoop服务，访问集群中的所有应用程序的默认端口号为8088。使用以下URL访问该服务 http://192.168.1.191:8088

如果想要本地访问虚拟机内部的hadoop集群服务，需要将本地和虚拟机的防火墙全部关闭，本地就不再介绍，虚拟机关闭防火墙的几个操作命令：
```bash

firewall-cmd --state ##查看防火墙状态，是否是running 
systemctl status firewalld.service ##查看防火墙状态
systemctl start firewalld.service ##启动防火墙
systemctl stop firewalld.service ##临时关闭防火墙
systemctl enable firewalld.service ##设置开机启动防火墙
systemctl disable firewalld.service ##设置禁止开机启动防火墙

systemctl status iptables 查看防火墙状态

关闭IPV4和IPV6的防火墙服务
service iptables stop
service ip6tables stop
设置防火墙服务开机不启动
chkconfig iptables off
chkconfig ip6tables off
```
 ![image](https://github.com/17661977890/hadoop-start/blob/master/image/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190716143421.png)
 ![image](https://github.com/17661977890/hadoop-start/blob/master/image/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190716143450.png)
  
* (5) 有时候因为hadoop版本不同，配置会有不同，就像上述文件要重命名一样
 参考链接：https://www.cnblogs.com/zhengna/p/9316424.html 本文只是以此了解，不按此配置操作，具体其余配置可以自行百度
