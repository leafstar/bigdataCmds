

# GeoMesa

  ‘<SimpleFeatureType>’   ‘<SimpleFeature>’

```scala
val params = Map(HBaseDataStoreParams.HBaseCatalogParam.key -> “mycatalog”)
val store = DataStoreFinder.getDataStore(params.asJava)

store.dispose()
```



```scala
String spec = "name:String,dtg:Date,*start:Point,end:Point"
SimpleFeatureType sft = SimpleFeatureTypes.createType("mySft")
sft.getUserData().put("geomesa.indices.enabled","z3")
sft.getUserData().put("geomesa.indices.enabled","z3:start:dtg,z3:end:dtg,attr:name")
```

```scala
sft.getUserData().put("geomesa.fid.uuid",true)
```



```scala
sft.getDescriptor(“geom”).getUserData().put(“precision”,4)
```





```scala
sft.getUserData().put("geomesa.z.splits",4)
```



```scala
sft.getUserData().put("geomesa.z3.interval","month")
```

cardinality

```scala
sft.getDescriptor("name").getUserData().put("index", "true");
sft.getDescriptor("name").getUserData().put("cardinality", "high");
```



cached stats . top K,min,max

```scala
sft.getDescriptor("name").getUserData().put("keep-stats", "true");
```



优先设置时间index

```scala
sft.getUserData().put("geomesa.temporal.priority", "true");
```



mixed geometry type

```scala
sft.getUserData().put("geomesa.mixed.geometries", "true");
```



geomesa-site.xml generally has lower priority than cmdline . However, we can set <final> tag to be true to specify top priority of xml configs.







