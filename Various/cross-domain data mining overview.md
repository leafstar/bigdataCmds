# Methodologies for Cross-Domain Data Fusion

## 1. Intro

three categories:

+ stage-based
  + 比如把城市划分成regions，然后根据human mobility data来detect规划的不好的地方。not well connected
+ feature level-based
  + DNN。 data->new feature representation
+ semantic meaning-based
  + multi-view learning-based
    + different datasets (features) fed into different models,later merged together.
    + Eg. Co-training
  + similarity-based
    + eg. coupled collaborative filtering/context-aware-CF
    + different datasets are modeled by different matrices with common dimensions.
    + eg. Manifold alignment
  + probablistic dependency-based
    + BN, Markov random field
  + transfer learning-based
    + transfer

## 2. Related work

### 2.1 relation to traditional data integration

传统：转化成consistent的schema， duplicate detection 之后merge

现在：knowledge fusion instead of schema mapping

### 2.2 heterogeneous information network

eg. node: author, conference, paper …. edge: an anthor published a paper, a paper presented in a conference.

算法：rank and cluster. \[58\]\[59\] 

虽然能表示不同类型的node，edge，但还是无法表示不同的domain。

## 3 Stage-Based Data Fusion Methods

不同的阶段用不同的dataset，所以在consistency方面没什么要求。

Eg1: 

+ map segementation method, partition a city into regions.
+ GPS trajs are then mapped onto the regions to formulate a region graph. Node: region. Edge: commutes
+ region graph blends knowledge from road network and taxi trajs.
+ mining tasks can be carries out, such as improper design of road network.

Eg2:

+ Hierachical clustering.
  + traj->staypoint->pois->user similarity.

Eg 3:

+ driver routing behaviour significantly differ from history-> anomaly + tweets(social media keywords &locations )…..



## 4. Feature-Level Based Data Fusion



