# Redis实践

## 点赞功能

**需求**

- 显示点赞数量
- 判断用户是否点过赞，用于去重，必须的判断
- 显示个人点赞列表，一般在用户中心
- 显示文章点赞列表

### Mysql实现

```sql
-- 文章表
create table post {        
post_id int(11) NOT NULL AUTO_INCREMENT,        
......        
star_num int(11) COMMENT '点赞数量'
}

-- 用户表
create table user {        
user_id int(11) NOT NULL AUTO_INCREMENT,        
......        
star_num int(11) COMMENT '点赞数量'
}

-- 点赞表

create table star {        
id int(11) NOT NULL AUTO_INCREMENT,        
post_id,        
user_id,        
......
}
```

**常用的查询：**

查询用户点赞过的文章 `select post_id from star where user_id=?`

查询文章的点赞用户 `select user_id from star where post_id=?`

点赞数量可以通过定时异步统计更新到post和user 表中。

数据量不大的时候，这种设计基本可以满足需求了，

**缺点：**

数据量大时，一张表在查询时压力巨大，需要分表，而不论用post_id还是user_id来hash分表都与我们的需求有冲突，唯一的办法就是做两个表冗余。这增加了存储空间和维护工作量，还可能有一致性问题。

### Redis实现

#### 场景a ：显示点赞数量

在点赞的地方，只是显示一个点赞数量，能区分用户是否点赞过，一般用户不关心这个列表，这个场景只要一个数字就可以了，当数量比较大时，一般显示为"7k" ,"10W" 这样。

以文章id为key

```java
//以文章id=888为例 127.0.0.1:6379[2]
> set star:tid:888 898 
//设置点赞数量 OK 127.0.0.1:6379[2]
> incr star:tid:888 
//实现数量自增 (integer) 899
```

#### 场景b：点赞去重，避免重复点赞

要实现这个需求，必须有文章点赞的uid列表，以uid为key

### 场景c：一般在用户中心，可以看到用户自己的点赞列表

这个需求可以使用场景b的数据来实现。

### 场景d：文章的点赞列表

### 类似场景b，以文章id为key

```
//以文章id=888为例 127.0.0.1:6379[2]> sadd star:list:tid:888 123 456 789  //点赞uid列表 (integer) 3 127.0.0.1:6379[2]> sismember star:list:tid:888 456  //判断是否点赞 (integer) 1
```

可能有人觉得，点赞列表没人关心，存储又会浪费大量资源，不如不存！但是，这个数据是必须要有的。两点：

- 去重。点赞数可以不精确，但去重必须是精确的，
- 另外一个社交产品，用户行为的一点一滴都需要记录，对于后续的用户行为分析和数据挖掘都是有意义的。

上面使用string存储的用户点赞数量，除了string，还可以用hash来存储，对文章id分块，每100个存到一个hash，分别存入hash table，每个文章id为hash的一个key，value存储点赞的用户id，如果点赞用户很多，避免id过多产生性能问题，可以单列出来，用sorted set结构保存，热点的毕竟是少数。

## 3. 数据一致性

redis作为storage使用时，一定要做好数据的持久化，必须开启 rdb 和 aof，这会导致业务只能使用一半的机器内存，所以要做好容量的监控，及时扩容。