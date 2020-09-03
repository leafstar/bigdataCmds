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

# Spark read Hbase
    

  
