# 5.hbase的Java编程
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

## 源码
```
package hbase;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

import org.apache.commons.configuration.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.HColumnDescriptor;
import org.apache.hadoop.hbase.HTableDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Delete;
import org.apache.hadoop.hbase.client.Get;
import org.apache.hadoop.hbase.client.HBaseAdmin;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.client.Scan;
import org.apache.hadoop.hbase.filter.BinaryComparator;
import org.apache.hadoop.hbase.filter.BinaryPrefixComparator;
import org.apache.hadoop.hbase.filter.CompareFilter;
import org.apache.hadoop.hbase.filter.Filter;
import org.apache.hadoop.hbase.filter.RowFilter;
import org.apache.hadoop.hbase.filter.SubstringComparator;
import org.apache.hadoop.hbase.mapreduce.IndexBuilder.Map;
import org.apache.hadoop.hbase.protobuf.generated.ZooKeeperProtos.Table;
import org.apache.hadoop.hbase.util.Bytes;

public class Hbase_test {
	
	private static org.apache.hadoop.conf.Configuration conf;

	static {
		conf =HBaseConfiguration.create();
		//((org.apache.hadoop.conf.Configuration) conf).set("hbase.zookeeper.quorum", "master");
	    ((org.apache.hadoop.conf.Configuration) conf).set("hbase.zookeeper.property.clientPort", "2181");
	}
	public static boolean create(String tableName, String columnFamily) {
	    HBaseAdmin admin = null;
	    try {
	        admin = new HBaseAdmin((org.apache.hadoop.conf.Configuration) conf);
	        if (admin.tableExists(tableName)) {
	            System.out.println(tableName + " exists!");
	            return false;
	        } else {
	            // 逗号分隔，可以有多个columnFamily
	            String[] cfArr = columnFamily.split(",");
	            HColumnDescriptor[] hcDes = new HColumnDescriptor[cfArr.length];
	            for (int i = 0; i < cfArr.length; i++) {
	                hcDes[i] = new HColumnDescriptor(cfArr[i]);
	            }
	            HTableDescriptor tblDes = new HTableDescriptor(TableName.valueOf(tableName));
	            for (HColumnDescriptor hc : hcDes) {
	                tblDes.addFamily(hc);
	            }
	            admin.createTable(tblDes);
	            System.out.println(tableName + " create successfully！");
	            return true;
	        }
	    } catch (IOException e) {
	        e.printStackTrace();
	        return false;
	    }
	}
	
	public static boolean put(String tableName, String rowkey, String columnFamily, String qualifier, String value) {
	    try {
	        HTable table = new HTable(conf, tableName);
	        org.apache.hadoop.hbase.client.Put put = new org.apache.hadoop.hbase.client.Put(rowkey.getBytes());
	        put.add(columnFamily.getBytes(), qualifier.getBytes(), value.getBytes());
	        table.put(put);
	        System.out.println("put successfully！ " + rowkey + "," + columnFamily + "," + qualifier + "," + value);
	    } catch (IOException e) {
	        e.printStackTrace();
	        return false;
	    }
	    return true;
	}

	public static boolean delete(String tableName) {
	    System.out.println("delete table " + tableName);
	    try {
	        HBaseAdmin admin = new HBaseAdmin(conf);
	        if (admin.tableExists(tableName)) {
	            admin.disableTable(tableName);
	            admin.deleteTable(tableName);
	        }
	        return true;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return false;
	    }
	}
	
	public static Result getResult(String tableName, String rowkey) {
	    System.out.println("get result. table=" + tableName + " rowkey=" + rowkey);
	    try {
	        HTable table = new HTable(conf, tableName);
	        Get get = new Get(rowkey.getBytes());
	        return table.get(get);
	    } catch (IOException e) {
	        e.printStackTrace();
	        return null;
	    }
	}
	
	public static byte[] get(String tableName, String rowkey, String qualifier) {
	    System.out.println("get result. table=" + tableName + " rowkey=" + rowkey + " qualifier=" + qualifier);
	    Result result = getResult(tableName, rowkey);
	    if (result != null && result.listCells() != null) {
	        for (Cell cell : result.listCells()) {
	            String key = Bytes.toString(CellUtil.cloneQualifier(cell));
	            if (key.equals(qualifier)) {
	                String value = Bytes.toString(CellUtil.cloneValue(cell));
	                System.out.println(key + " => " + value);
	                return CellUtil.cloneValue(cell);
	            }
	        }
	    }
	    return null;
	}
	
	private static HashMap<String, Object> result2Map(Result result) {
	    HashMap<String, Object> ret = new HashMap<String, Object>();
	    if (result != null && result.listCells() != null) {
	        for (Cell cell : result.listCells()) {
	            String key = Bytes.toString(CellUtil.cloneQualifier(cell));
	            String value = Bytes.toString(CellUtil.cloneValue(cell));
	            //System.out.println(key + " => " + value);
	            ret.put(key, value);
	        }
	    }
	    return ret;
	}
	
	public static List<Map> scan(String tableName) {
	    System.out.println("scan table " + tableName);
	    try {
	        HTable table = new HTable(conf, tableName);
	        Scan scan = new Scan();
	        ResultScanner rs = table.getScanner(scan);
	        List<Map> resList = new ArrayList<Map>();
	        for (Result r : rs) {
	            HashMap<String, Object> m = result2Map(r);
	            StringBuilder sb = new StringBuilder();
	            for(String k : m.keySet()) {
	                sb.append(k).append("=>").append(m.get(k)).append(" ");
	            }
	            //System.out.println(sb.toString());
	            System.out.println(m);
	        }
	        return resList;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return null;
	    }
	}
	
	public static List<String> list() {
	    System.out.println("list tables.");
	    try {
	        HBaseAdmin admin = new HBaseAdmin(conf);
	        TableName[] tableNames = admin.listTableNames();
	        List<String> tblArr = new ArrayList<String>();
	        for (int i = 0; i < tableNames.length; i++) {
	            tblArr.add(tableNames[i].getNameAsString());
	            System.out.println("Table: " + tableNames[i].getNameAsString());
	        }
	        return tblArr;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return null;
	    }
	}
	
	public static boolean deleteQualifier(String tableName, String rowkey, String columnFamily, String qualifier) {
	    System.out.println("delete qualifier. table=" + tableName
	            + " rowkey=" + rowkey + " cf=" + columnFamily + " qualifier=" + qualifier);
	    try {
	        HBaseAdmin admin = new HBaseAdmin(conf);
	        if (admin.tableExists(tableName)) {
	            HTable table = new HTable(conf, tableName);
	            Delete delete = new Delete(rowkey.getBytes());
	            delete.deleteColumn(columnFamily.getBytes(), qualifier.getBytes());
	            table.delete(delete);
	        }
	        return true;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return false;
	    }
	}
	
	public static boolean deleteRow(String tableName, String rowkey) {
	    System.out.println("delete row. table=" + tableName + " rowkey=" + rowkey);
	    try {
	        HBaseAdmin admin = new HBaseAdmin(conf);
	        if (admin.tableExists(tableName)) {
	            HTable table = new HTable(conf, tableName);
	            Delete delete = new Delete(rowkey.getBytes());
	            table.delete(delete);
	        }
	        System.out.println(tableName + ", " + rowkey + " delete successfully!");
	        return true;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return false;
	    }
	}
	
	public static boolean deleteColumnFamily(String tableName, String columnFamily) {
	    System.out.println("delete column family. table=" + tableName + " cf=" + columnFamily);
	    try {
	        HBaseAdmin admin = new HBaseAdmin(conf);
	        if (admin.tableExists(tableName)) {
	            admin.disableTable(tableName);
	            admin.deleteColumn(tableName, columnFamily);
	            admin.enableTable(tableName);
	        }
	        return true;
	    } catch (IOException e) {
	        e.printStackTrace();
	        return false;
	    }
	}
	
	public static void split(String tableName,int number,int timeout) throws Exception{  
	    HBaseAdmin hAdmin = new HBaseAdmin(conf);  
	    HTable hTable = new HTable(conf,tableName);  
	    int oldsize = 0;  
	    long time = System.currentTimeMillis();  
	    while(true){  
	        int size = hTable.getRegionLocations().size();  
	        //logger.info("the region number="+size);  
	        if(size>=number) break;  
	        if(size !=oldsize){  
	            hAdmin.split(hTable.getTableName());  
	            oldsize = size;  
	        }else if(System.currentTimeMillis()-time>timeout){  
	            break;  
	        }  
	        Thread.sleep(1000*10);  
	    }  
	}  
	
    public static void WithRowFilter(String tableName,String row) throws IOException {
    	HTable table = new HTable(conf, tableName);
        Scan scan = new Scan();
        System.out.println("包含有"+row+"的行");
        Filter filter3 = new RowFilter(CompareFilter.CompareOp.EQUAL,new SubstringComparator(row));
        scan.setFilter(filter3);
        ResultScanner scanner3 = table.getScanner(scan);
        for (Result res : scanner3) {
                System.out.println(res);
        }
        scanner3.close();
    }
    
    public static void lessequalRowFilter(String tableName,String row) throws IOException {
    	HTable table = new HTable(conf, tableName);
        Scan scan = new Scan();
        System.out.println("小于等于"+row+"的行");
        Filter filter3 = new RowFilter(CompareFilter.CompareOp.EQUAL,new BinaryComparator(row.getBytes()));
        scan.setFilter(filter3);
        ResultScanner scanner3 = table.getScanner(scan);
        for (Result res : scanner3) {
                System.out.println(res);
        }
        scanner3.close();
    }
    
    public static void beginwithRowFilter(String tableName,String row) throws IOException {
    	HTable table = new HTable(conf, tableName);
        Scan scan = new Scan();
        System.out.println("正则结尾为"+row+"的行");
        Filter filter3 = new RowFilter(CompareFilter.CompareOp.EQUAL,new BinaryPrefixComparator(row.getBytes()));
        scan.setFilter(filter3);
        ResultScanner scanner3 = table.getScanner(scan);
        for (Result res : scanner3) {
                System.out.println(res);
        }
        scanner3.close();
    }
	
		public static void main(String[] args) throws Exception {
			create("students", "ID,Description,Courses,Home");//create table
			put("students", "001", "Description", "Name", "Li Lei");//Insert a piece of data
			put("students", "001", "Description", "Height", "176");
			put("students", "001", "Courses", "Chinese", "80");
			put("students", "002", "Home", "Province", "Bei Jing");
			put("students", "003", "Home", "Province", "Shang Hai");
			put("students", "002", "Description", "Name", "Han Meimei");
			put("students", "002", "Description", "Height", "183");
			put("students", "003", "Description", "Height", "162");
			put("students", "003", "Description", "Name", "Xiao Ming");
			scan("students");//Query table data
			//delete("students");//delete table
			//System.out.println(getResult("test1","row1"));//single query, appointed row
			WithRowFilter("test1","row2");//Batch query with row name
			lessequalRowFilter("test1","row2");
			beginwithRowFilter("test1","row2");
			//System.out.println(get("test1","row1","field1"));//search the value of the appointed row and column
			//list();//get all the list
			//deleteQualifier("test1", "row1", "cf1", "field1");//delete appointed column
			//deleteRow("test1", "row1");//delete appointed row
			//deleteColumnFamily("test1","cf1");//delete appointed column family
			//split("test1",2,1000);//Slitting table
			
		    }
		
}
```
