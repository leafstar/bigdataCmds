# Redis commands

1. strings

```
set mykey 10
get mykey
incr mykey
incrby mykey 100
decr mykey
decrby mykey
mset a 10 b 20 c 30
mget a b c
exists mykey
del mykey
type mykey
expire mykey 5
ttl mykey
set mykey 10 ex 10
set my key 100 xx/nx
```

2. lists

```
lpush/rpush mykey 1 2 3
lrange mykey 0 -1
ltrim mykey 1 2
lpop/rpop mykey
brpop/blpop # block 

lpushx newlist 1 # only when newlist exists

rpoplpush
```

+ producer can call lpush, consumer can call lpop.

+ empty key will be automatically destroyed

3. hashes

```
hmset player:1 name ame age 23 team lgd 
hget player:1 name
hgetall player:1
hmget

hincrby
```

4. sets

```
sadd myset 1 2 3 4 5 6
smember myset
sismember myset 1 
scard myset
sinter/sunion/sdiff
sunionstore
srandmember deck 10
spop

```

5. sorted sets

```
zadd hackers 1940 "aaa"// add&update 
zrange/zrevrange [withscores]
zrangebyscore/zremrangebyscore
zrank/zrevrank
```

+ lexicographical scores

  ```
  zrangebylex
  ...
  ```

6. bitmaps

an 8-bit bitmap : _ _ _ _ _ _ _ _

store user info inside it :

best player of 2019: _ _ _ _ 1_ _ _ (user 5 set to 1)

best player of 2020: _ _ _ _ _ _ _ 1 (user 5 and 8 set to 1)

use bitop OR to get best players in the history.

```
setbit bestPlayer:2019 5 1
setbit bestPlayer:2020 8 1
bitop OR bestPlayerInHist bestPlayer:2019 bestPlayer:2020
```

7. hyperloglog

```
pfadd
pfcount
```

