# hbase下载与配置
下载网站:http://www.apache.org/dyn/closer.cgi/hbase/  
选择http://mirrors.shuosc.org/apache/hbase/  
选择stable  
选择hbase-1.2.6-bin.tar.gz进行下载  
下载完成后，命令tar xzvf hbase-1.2.6-bin.tar.gz进行解压  
命令sudo chmod 777 hbase-1.2.6赋予权限  
## 配置
### 对conf/hbase-env.sh进行配置  
export JAVA_HOME=/home/u2/java/jdk1.8.0_144  
export HBASE_MANAGES_ZK=true  
### 对conf/hbase-site.xml进行配置
单机模式下配置  
```
<configuration>
	<property>
		<name>hbase.rootdir</name>
		<value>file:///home/u2/testuser/hbase</value>
	</property>
	<property>
		<name>hbase.zookeeper.property.dataDir</name>
		<value>/home/u2/testuser/zookeeper</value>
	</property>
</configuration>
```
伪分布式下配置  
```
<configuration>
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<property>
		<name>hbase.root.dir</name>
		<value>hdfs://localhost:9000/hbase</value>
	</property>
	<property>
		<name>hbase.zookeeeper.property.dataDir</name>
		<value>/home/u2/hbase/testuser/zookeeper</value>
	</property>
</configuration>
```
## 伪分布式hbase
启动dfs
```
bin/hdfs namenode -format  
sbin/start-dfs.sh  
```
![image](https://github.com/cjjloves/Homework6/edit/master/pictures/start-dfs.JPG)
启动hbase
```
bin/start-hbase.sh  
```
![image](https://github.com/cjjloves/Homework6/edit/master/pictures/start-hbase.JPG)
进入shell模式
```
bin/hbase shell  
```
关闭hbase
```
./bin/stop-hbase.sh  
```
关闭dfs
```
sbin/stop-dfs.sh  
```
## 遇到的问题
在运行list命令时出错  
![image](https://github.com/cjjloves/Homework6/edit/master/pictures/problems.JPG)
解决方法：将conf/hbase-site.xml中的配置hbase.rootdir改为hbase.root.dir
