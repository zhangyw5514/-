# 分布式唯一ID
## UUID
- UUID是通用唯一识别码（Universally Unique Identifier)的缩写，开放软件基金会(OSF)规范定义了包括网卡MAC地址、时间戳、名字空间（Namespace）、随机或伪随机数、时序等元素。利用这些元素来生成UUID。  
- UUID是由128位二进制组成，一般转换成十六进制，然后用String表示。
### 优点
- 通过本地生成，没有经过网络I/O，性能较快
- 无序，无法预测它的生成顺序。(当然这个也是他的缺点之一)
### 缺点
- 一般使用32位的UUID，数字字母都有，只能使用String存储
- 信息不安全：基于MAC地址生成UUID的算法可能会造成MAC地址泄露，这个漏洞曾被用于寻找梅丽莎病毒的制作者位置
- 作为MySQL主键非常不适合:
  - MySQL官方有明确的建议主键要尽量越短越好[4]，36个字符长度的UUID不符合要求
  > All indexes other than the clustered index are known as secondary indexes. In InnoDB, each record in a secondary index contains the primary key columns for the row, as well as the columns specified for the secondary index. InnoDB uses this primary key value to search for the row in the clustered index.*** If the primary key is long, the secondary indexes use more space, so it is advantageous to have a short primary key***.
  - 对MySQL索引不利：如果作为数据库主键，在InnoDB引擎下，UUID的无序性可能会引起数据位置频繁变动，严重影响性能。
- 无法生成递增有序的数字
### 适用场景
- 不需要担心过多的空间占用，以及不需要生成有递增趋势的数字
  
## 数据库主键自增
### 优点
- 简单方便，有序递增，方便排序和分页
### 缺点
- 分库分表会带来问题，需要进行改造。
- 并发性能不高，受限于数据库的性能。
- 简单递增容易被其他人猜测利用，比如你有一个用户服务用的递增，那么其他人可以根据分析注册的用户ID来得到当天你的服务有多少人注册，从而就能猜测出你这个服务当前的一个大概状况。
- 数据库宕机服务不可用。
### 适用场景
- 根据上面可以总结出来，当数据量不多，并发性能不高的时候这个很适合，比如一些to B的业务，商家注册这些，商家注册和用户注册不是一个数量级的，所以可以数据库主键递增。如果对顺序递增强依赖，那么也可以使用数据库主键自增。
  
## Redis
- Redis中有两个命令:Incr和IncrBy,都是原子性的操作。
### 优点
- 性能会比数据库要好
- 可以满足有序递增
### 缺点
- 由于Redis是存储在内存中，即使有RDB和AOF，但是仍然存在数据丢失的风险，可能会造成ID重复
- 强依赖于Redis，Redis故障时服务不可用
### 适用场景
- 由于其性能比数据库好，但是有可能会出现ID重复和不稳定，这一块如果可以接受那么就可以使用。也适用于到了某个时间，比如每天都刷新ID，那么这个ID就需要重置，通过(Incr Today)，每天都会从0开始加。
  
## Snowflake算法
- 1bit(无用位) + 41bit(时间戳) + 10bit(workId) + 10bit(序列号)
### 优点
- 毫秒数在高位，自增序列在低位，整个ID都是趋势递增的。
- 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的。
- 可以根据自身业务特性分配bit位，非常灵活。
### 缺点
- 强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态。
### 应用举例
- MongoDB ObjectID
### 适用场景
- 当我们需要无序不能被猜测的ID，并且需要一定高性能，且需要long型，那么就可以使用我们雪花算法。比如常见的订单ID，用雪花算法别人就无法猜测你每天的订单量是多少。

## 美团开源分布式ID生成服务——Leaf
- 分为号段模式和Snowflake模式
### 适用场景
- Leaf-segment方案可以生成趋势递增的ID，同时ID号是可计算的，不适用于订单ID生成场景，比如竞对在两天中午12点分别下单，通过订单id号相减就能大致计算出公司一天的订单量，这个是不能忍受的
- Leaf-Snowflake方案对时钟要求比较敏感，内部也提供了解决方案



