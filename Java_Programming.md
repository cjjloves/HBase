# 1.hbase的java编程
## 配置
在hbase-site.xml增加配置  
```
	<property>
                <name>hbase.zookeeper.property.clientPort</name>
                <value>2181</value>
	</property>
```
## 相关jar包导入
参考网站：http://www.aboutyun.com/thread-8401-1-1.html
## 代码说明
### 1. 创建表
参数：表名、列族
```
public static boolean create(String tableName, String columnFamily)
```
### 2. 插入一行数据
参数：表名、行、列族、列、值
```
public static boolean put(String tableName, String rowkey, String columnFamily, String qualifier, String value) 
```
### 3. 输出表信息
参数：表名
```
public static List<Map> scan(String tableName)
```
### 4. 以行删除数据
参数：表名、行
```
public static boolean deleteRow(String tableName, String rowkey)
```
### 5. 以列族删除数据
参数：表明、列族
```
public static boolean deleteColumnFamily(String tableName, String columnFamily)
```
### 6. 删除一条数据
参数：表名、行、列族、值
```
public static boolean deleteQualifier(String tableName, String rowkey, String columnFamily, String qualifier)
```
### 7. 删除表
参数：表名
```
public static boolean delete(String tableName)
```
### 8. 单条查询
参数：表名、行
```
public static Result getResult(String tableName, String rowkey)
```
### 9. 范围查询
参数：表名、行
```
//包含row的行
public static void WithRowFilter(String tableName,String row)
//小于等于row的行
public static void lessequalRowFilter(String tableName,String row)
//正则结尾为row的行
public static void beginwithRowFilter(String tableName,String row)
```
### 10. 按指定值进行查询
参数：表名、行、值
```
public static byte[] get(String tableName, String rowkey, String qualifier)
```
### 11. 得到所有表名
无参数
```
public static List<String> list()
```
### 12. 切分表
参数：表名、切分数、时间范围
```
public static void split(String tableName,int number,int timeout)
```
## 程序建表、输出表信息截图
![Image text](https://raw.github.com/cjjloves/Homework6/master/pictures/code.JPG)  
![Image text](https://raw.github.com/cjjloves/Homework6/master/pictures/result.JPG)
