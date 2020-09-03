# bigdataCmds

# Spark read HDFS
* as rdds
```
    val rddFromFile = spark.sparkContext.textFile("hdfs://nn1home:8020/text01.txt")
    val rddWhole = spark.sparkContext.wholeTextFiles("hdfs://nn1home:8020/text01.txt")
```
  
* as DFs
```
    val df:DataFrame = spark.read.text("hdfs://nn1home:8020/text01.txt")
    val ds:Dataset[String] = spark.read.textFile("hdfs://nn1home:8020/text01.txt")
```    
    

  
