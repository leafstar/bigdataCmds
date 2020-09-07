# bigdataCmds

# Spark read HDFS
* as rdds
```
    val rddFromFile = spark.sparkContext.textFile("hdfs://nn1home:8020/text01.txt")
    val rddWhole = spark.sparkContext.wholeTextFiles("hdfs://nn1home:8020/text01.txt")
```
  
* as DFs

 * 反射机制推断rdd模式
```
    // 1.create case class for implicit transformation.
    case class Person(name: String, age: Long)
    val DF = sc.textFile("path_name").
        map(_.split(",")).
        map(attr=>Person(attr(0), attr(1).trim.toInt)).
        toDF()
    // 2.注册临时表
    DF.createOrReplaceTempView("peeple")  //随便起个名字
    // 3.查询
    val res = spark.sql("select name, age from people where age > 20")
    res.show()
```    
 * 编程方式定义
```
    // 1. 创建schema
    val fields = Array(StructField("name", StringType, true),
                       StructField("age", IntegerType, true))
    val schema = StructType(fields)
    
    // 2. 创建row
    val peopleRDD = sc.textFile("path_name")
        //convert rdd to object ROW
    val rowRDD = peopleRDD.map(lines=>lines.split(",")).
        map(attr=>Row(attr(0),attr(1).trim.toInt))
    // 3. 链接 schema和rows
    val peopleDF = spark.createDataFrame(rowRDD, schema)
    
    // 4. 注册临时表
    peopleDF.createOrReplaceTempView("people")
    
    // 5.查询
    
    val res = spark.sql("select name, age from people where age > 20")
    
    res.show()
    
```

# Spark read Hive
```
import java.io.File

import org.apache.spark.sql.{Row, SaveMode, SparkSession}

case class Record(key: Int, value: String)

// warehouseLocation points to the default location for managed databases and tables
val warehouseLocation = new File("spark-warehouse").getAbsolutePath

val spark = SparkSession
  .builder()
  .appName("Spark Hive Example")
  .config("spark.sql.warehouse.dir", warehouseLocation)
  .enableHiveSupport()
  .getOrCreate()

import spark.implicits._
import spark.sql

sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING) USING hive")
sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

// Queries are expressed in HiveQL
sql("SELECT * FROM src").show()
// +---+-------+
// |key|  value|
// +---+-------+
// |238|val_238|
// | 86| val_86|
// |311|val_311|
// ...

```
# Spark read Hbase
    
```
def catalog = s"""{
    |"table":{"namespace":"default", "name":"Contacts"},
    |"rowkey":"key",
    |"columns":{
    |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
    |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
    |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
    |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
    |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
    |}
|}""".stripMargin

```
# 11

```
def withCatalog(cat: String): DataFrame = {
    spark.sqlContext
    .read
    .options(Map(HBaseTableCatalog.tableCatalog->cat))
    .format("org.apache.spark.sql.execution.datasources.hbase")
    .load()
 }

```
# sparkSession.read()返回一个dataFrameReader class
+ csv
+ jdbc
+ json
+ orc
+ parquet
+ text

# 连接jdbc数据库　
+ 读数据
```
val jdbcDF = spark.read.format("jdbc")
	.option("url","jdbc:mysql://localhost:3306/spark")
	.option("driver","com.mysql.jdbc.Driver")
	.option("dtable", "student")
	.option("user", "root")
	.option("password", "....")
	.load()
jdbcDF.show()
```

+ 写数据
```
import java.util.Properties
import org.apache.spark.sql.types._
import org.apache.spark.sql.Row

val studentRDD = spark.sparkContext.parallelize(Array("3 LEO M 26","4 LEO2 M 27")).map(._split(" "))
val schema = StructType(list(StructField("id", IntegerType, true),StructField("NAME", StringType, true),StructField("gender",Stringtype,true),StructField("age",IntegerType,true)))

val RowRDD = studentRDD.map(p=> Row(p(0).trim.toInt,p(1).trim,p(2).trim,p(3).trim.toInt))
val studentDF = spark.createDataFrame(rowRDD,schema)

//保存jdbc连接参数
val prop = new Properties()
prop.put("user","root")
prop.put("password","xxx")
prop.put("driver","com.mysql.jdbc.Driver")

//连接数据库，采用append模式
studentDF.write.mode("append").jdbc("jdbc:mysql://localhost:3306/spark","spark.student", prop)
```
