## Overview算法笔记

## 1.Derivation

+ people
+ animal
+ transportation
+ natural pheonomena

## 2.Peoprocessing

### 2.1 noise filtering

#### 	Mean(Median) filter

> use n-1 predecessors to calculate the mean/median. 

> good for individual, need big window size for consecutive noise points.

> can result in bigger error.

#### 	Kalman and Particle filters  (53号论文)

> Kalman use speed. Assume linear model+ Gaussian noise.
>
> Particle: step1: generate P points from the initial distribution; 
>
> ​               step2: importance sampling.
>
> ​			    step3: importance weight.
>
> ​                step4: selection
>
> ​				step5:weight sum = $\sum_{i=1}^{P}\omega_i^{(j)}\hat x^j$

#### 	Heuristic based outlier detection

points with speed larger than threshold are removed.

### 2.2 stay point detection

dist threshold/ time span threshold.

从第一个点开始，不断加入点直到他们的距离超过阈值，形成一个簇。从不在簇中的点开始继续detect。（论文中还用timespan来测一下，一个簇的anchor point和最后一个点时间长度是否大于阈值）。

改进版本：一个簇形成之后，从簇中第二个点继续跑算法。相当于sliding size减小到1。

### 2.3 compression 

+ 两种measure error的方式。 1.perpendicular euclidean distance，2. time synchronized euclidean distance. 
+ 后者assume 匀速直线运动

**Douglas Peucker：**

找到error最大的点，分开，两边继续。从N^2提升到了NlogN



**Online：**

sliding window. keep adding points until error exceed some bound. reserve it and continue

open window:  keep adding…. find the max error point as the new anchor point.

[84]. new data point important or not.



**semantic meaning**

[17] Chen proposed TS.  firstly divide it into walking/non-walking segments. point weight：转向+邻居距离。

另一些：drop所有anchor point直到现在位置的中间点，只要follow shortest path。需要 map-matching

PRESS时空分别压缩。 空间压缩用short code来代表常用的路段。



### 2.4 Traj segmentation

+ time interval

time interval>threshold : split 

+ shape of traj

 turing points(head of direction turned over a threshold)

Douglus Peucker.

MDL: [51]   L(H),  L(D|H)

+ semantic meaning

**stay points. **

if estimate velocity, remove stay points.

if wanna compare similarity between users, only use the stay points.

**transportation modes**

walk vs non-walk.



### 2.5 MapMatching

- additional information used

  + geometric: shape of indiv link.
  + topological algorithms: 
    + connectivity of road network
    + Frechet distance (dist between GPS sequence & candidate road sequence)
  + probablistic alg:
    
  + deal with noisy, low sampling rate trajs
    
  + advanced.

    

- range of sampling points

  + local  通常 online
  + global offline. entire trajs - road networks

  



【54】算法： 对某个点p，画个圆，圆内的segment都是candidite，然后投影到每个segment上，生成ci，每个ci都是candidate point，用高斯分布计算概率。然后同时进行概率传递，传递的概率是两个ci的euclidean dist除以路网dist。

local+transition确定最好的点。



## 3. Data management



###  3.1 Indexing and retrieval



#### > 3 approaches of range queries.

+ time as the third cood.  3D Rtree , 3D Query box.  

  + good for recent hours, bad for long time span. overlaps.  
  + <span style="color:red">不是特别清楚frequently update index</span>
  + can be improved, but not perfect anyway.

+ Rtree index for each interval（time slot）

  + query来了就找time slot，找到了再找spatial info

+ 先通过spatial 分成gird。 对每个grid里的数据建立时间索引。

  + CSE tree 。
  + 每个segment都有一个2d index （startTime，endTime），这些point被b+ tree存起来。
  + 查询时，先找所有相关grid，然后再去b+tree 找到segments，最后merge这些segments的ID以及始末时间。

  

#### > KNN Queries.

+ point query
  + order may matter
  + 更加在乎traj是否跟query点的连接性更好，而不是shape
+ traj query
  + 相似度
  + 要retrieve 车子在某条路上的轨迹
    + 把路网看作traj，算相似度
    + 把query traj 分开成segments，然后用mma去匹配road。
    + road & traj relationship stored in a suffix tree-like index。
      + 每个node是road segment，每个edge代表path。
      + 每个node都存着我这个road segment都有哪些traj走过 $T_{r_i}$，走了多久 $t_{r_i}$。
      + 很显然适合recent，因为每加一个traj，就得加很多index

### 3.2 Distance & Similarities of Trajectories

+ 单个点q，就找轨迹上最近的那个点。 $D(q,A) = min_{p\in A}D(q,p)$ 

+ 如果是多个点的集合Q，用一个exponential sum来表示
  + $D(Q,A) = \sum_{q\in Q} e^{D(q,A)}$
  + 如果计算相似度，那么把指数变成负的，这样近的pair就会有更大的weight。远的点weight就会非常小。
  + $S(Q,A) = \sum_{q\in Q} e^{-D(q,A)}$

+ 两个轨迹间的距离：
  + CPD(A, B) 直接用最近点代表轨迹间距离
  + SPD(A, B) 两两算距离相加 （assume ab length 相同）
  + DTW用来解决SPD假设不成立的问题，允许点被重复使用
  + LCSS 允许跳过某些噪点。
  + 感觉跟sequence alignment算法非常接近啊。。怎么就是新的算法了呢
  + KBCT 允许重复点也允许跳过。
+ 两个轨迹片段的距离
  + minimun bounding rectangles. 
    + D(B1,B2) = 最短的两点距离，one from each rectangle。
  + $D_{Haus}$ weighted sum of $d_{\perp}+d_{\parallel}+d_{\theta}$



## 4 Uncertainty

### 4.1.1 modeling uncertainty. indi PDF or stochastic process

### 4.1.2 Path inference 

​     most likely K routes.  complementary trajs.



两种：

1. 路网

跟路网匹配的区别：1，路网匹配只是单个traj，这个是多个traj互补。2，这边的采样率可以非常低，在mma中是不可想象的。

2. free space

起始点相邻，结束点相同，起始点的grid可以连成region，反之亦然



### 4.2 privacy





## 5 pattern mining

1. moving together pattern，2. trajectory clustering 3.sequential patterns 4.periodic patterns

### 5.1 moving together pattern

flock ： 某个形状的聚在一起连续的k时间

convoy：loose，density够就行

swarm：进一步loose，在k时间内就行，不一定要连续。

travel companion： online version of convoy and swarm



gathering pattern：再loose，允许member逐渐出现



### 5.2 trajectory clustering

难以把 spatial and temporal properties encode到feature vector里面。

主要针对free space，因为road map的case可以用mma以及graph clustering混合解决。

回到正题，可以用regression mixed model + em，但是考虑到移动的物体一般不会走完全程，所以我们把traj切成line segments。然后build group，基于D_Haus。

都用了micro and macro cluster。 一个主要发现是：新的数据只会影响这个数据接收到的区域，不会影响远的地区。 

### 5.3 Mining Sequential Patterns from Trajectories

A: l1->l2->l7->l4    B: l1->l2->l4

sequential trajectory pattern   作用： travel recommendation； next location predication； user similarity； traj compression



1. define location in a sequence.  location tagged with unique identity. 

但是在gps系统中，每个点都具有一对GPS坐标，这些坐标在每个模式实例中都不会完全重复。导致两个traj不能直接对比。而且点非常多，会有很多cost。

#### 解决方式： 5.3.1 sequential pattern mining in a free space

+ Line-simplication-based methods: 
  + 道格拉斯简化

+ cluster-based
  + 轨迹点聚在一起变成簇。一个簇就是一个location。
  + 若在乎semantic meaning。 可以把staypoints 表示成region of interest。



**5.3.2路网sequential pattern mining**

1. mma map到road network上。-> traj就变成了一堆road segment ID的集合，可以用string表示
2. LCSS和Suffix Tree就可以来做sequential pattern mining了。
3. Suffix Tree  edge上的数字代表有多少traj traverse了这个edge。build完之后可以在On时间内找到support given a threshold。dataset大的话，需要限制深度

### 5.4 mining periodical patterns from trajectories

帮助压缩traj 以及 预测。

一堆pattern asynchronous pattern/ surprising periodic pattern/ pattern with gap penalties.

由于fuzziness， 现有的方法不适用。 现实生活中情况非常复杂，interleaving peiods，partial time span。。noises and outliers

解决： two-stage detection。 

stage1: detect a few reference spots(density-based clusterign algs like KDE). 

然后就可以根据这些reference spots把traj转化成binary time series （in to/ out of reference spot）然后用FT，and some autocorrelation methods, the value of periods can be calculated.   

stage2: hierarchical clustering algs. summarizes periodic behaviors from partial movement sequences.



## 6. trajectory classification

分类：motions/transportation modes/human activities

很有价值emm。



3steps

+ Step1: segmentation
+ step2: extract feature from each seg
+ step3: build model to classify each seg. DBN/ HMM Conditional Random Field(CRF).  这些算法都有传递性。

149 147.



gps 通过crm-based mma 到street patches，再通过classification到 activities，a1,a2…(driving,walking,sleep) 并且identify significant places like P1， P2（home，work，bus stop..）



DBN-based inference  model。 s(ignal) -> l(ocation) -> a(ctivity) -> G(oal)



## 7. Anomalies Detection From Trajs

### 7.1 Detecting outlier trajectories

如果一段不能被cluster，大概就是outlier。

### 7.2 identify anomalous events by trajs

Liu 【62】 把一天分成时间箱。对每个link识别三个feature：某个时间箱里经过这个link的所有车数量；这些vehicles的占比（同样的dest region）；以及占比（同样的origin）。三个特征跟前几天进行比较，计算出每个feature最小的distort。构建三维的feature vector，然后用mahalanobis distance 来 measure extreme points/outliers

Pan Zheng【76】通过司机的行路习惯来检测异常，跟之前完全不一样的路线，就说明有异常。

然后跟社交网络上的关键词匹配



Pang【77，78】用likelihood ratio test 。 把城市分成网格，然后找跟预期严重不符合的（比如交通器具的数量），likelihood ratio statistic会掉到 $\chi^2$的尾巴上的，就是异常。



## 8. transfer  trajectory to other representations

###  8.1 From Trajectory to graph



#### 8.1.1 road network setting 

1. 直接把road network 的 intersection作为node，road seg作为edge，然后把traj map到road network上，这样根据traj所携带的信息，就可以计算一系列的东西，速度，traffic volume，是否有anomaly等等。



2. **landmark** 把topk visited road segment 作为 landmark nodes， 经过两个landmark的traj作为edges

two-stage routing alg is used to find the fastest driving path.  search landmark graph for a rough route(represented by a sequence of landmarks). and then find a route connecting consecutive landmarks.



3. build a region graph.

   image segmentation-based algorithm [128]

   通过主干道分成region 作为node，两个region间通勤比较多的就作为edge。

   转换之后用skyline alg 提取connection不好的region pair，意思是 走得慢，交通量大。

   

#### 8.1.2 free space settings

two major steps:

1. identify key locations as vertexes by clustering method. 
2. connect vertices based on trajs passing two locations.

**Travel recommendation** 

+ 从traj检测停留点，把不同人的停留点cluster成location
+ 建立一个bipartite graph，以及一个routable的graph between locations
  + bipartite graph里面的edge代表user去过那个location。
  + A HITS (Hypertext Induced Topic Search)-based model is then employed to infer the interest level of a location。**（authority score）** 以及 travel knowledge of a user**（hub score）**
  + 根据score可以排出 top k most interesting locations
  +  
  + ![image-20200918135913745](/Users/muxingwang/Library/Application Support/typora-user-images/image-20200918135913745.png)
  + 这个图，考虑三个factor来计算edge的importance（representativeness）
    + source location的authority score weighted by the probability that people move out by this edge
    + dest location的authority score weighted by the probability that people move in by this edge
    + the hub score of the users who have traveled this edge
    + http://pi.math.cornell.edu/~mec/Winter2009/RalucaRemus/Lecture4/lecture4.html 关于HITS的
    + sum起来得到edge的总分数
+ 【120】【121】提出了一个best traveling route算法。
+ 还有一些算法。。



Estimating the similarity

hierarchical 

![image-20200918142529306](/Users/muxingwang/Library/Application Support/typora-user-images/image-20200918142529306.png)

Xiao 把这个概念延伸，对不同的城市间的人做similarity comparison。 把location标记成POI。

### 8.2 From Traj to Matrix

三个问题： 1. row是啥 2. column是啥 3.entry是啥

#### travel recommendation

+ row代表user， col代表location， entry代表访问次数
+ collaborative filtering model用来预测

另外有一些matrix factorization的方法. 

<span style="color:red ">拓展一下： </span>

$X_{m*n}$ location-activity matrix.  entry是发生次数。

Yml  location-feature matrix 

Znn 从网上搜，activity之间的关联。 然后model uwv 三个matrix 用gradient descent的方式。

拿到矩阵之后就可以用uv直接model出x。进行推荐。





#### traffic condition estimation

论文【89】 。 这边的overview不好懂

#### anomaly

把traj 转化成 region graph。 两个matrix L和A 。L row是link，col是time interval。 entry是多少辆车在ti经过lj。   A row是link ，col是path， entry是indicator，link是否包含在此path内。

首先用PCA（principal component analysis） detect异常link。 用vector b表示。然后solve Ax=b，来得到跟path的relation

Tensor 先不看了，过于hardcore。如果需要要专门去看detailed paper



