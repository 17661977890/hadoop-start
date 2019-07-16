# hadoop-start
再centos 7 虚拟机搭建伪分布式hadoop集群 版本hadoop 2.7.7

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
* 配置hadoop的环境变量 vi etc/profile 还是参考上个链接
* 输入 hadoop 命令验证是否成功


#### 修改hadoop的配置文件，还是参考上个链接 
   
  hadoop 配置文件默认是本地模式，我们修改四个配置文件，这些文件都位于/usr/local/hadoop-2.7.7/etc/hadoop 目录下。
```bash

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
```
#### （4）启动hadoop集群:

* ==> 启动之前，格式化文件:你安装hadoop目录（usr/local/hadoop/）下执行命令 bin/hdfs namenode -format
  没有报错日志标识成功：
  
  ![image](https://github.com/17661977890/hadoop-start/blob/master/image/%E5%9B%BE%E7%89%871.png)
  
* ==> 一次性启动：还是安装目录下执行命令：  sbin/start-all.sh 启动 hadoop

* ==> 输入命令 jps  查看进程，如图所示，表示正在启动
  
  ![image](https://github.com/17661977890/hadoop-start/blob/master/image/%E5%9B%BE%E7%89%872.png)
  
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
