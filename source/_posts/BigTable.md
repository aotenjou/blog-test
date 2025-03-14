---
title: BigTable
date: 2025-03-14 15:23:52
tags:
- 读论文
---
<!--more-->

## bigtable：Google 设计的分布式结构化数据表

### Bigtable 的设计动机:

**需要存储的数据种类繁多**,包括URL、网页内容、用户的个性化设置在内的数据都是Google需要经常处理的
**需要存储的数据种类繁多海量的服务请求**,Google运行着目前世界上最繁忙的系统,它每时每刻处理的客户服务请求数量是普通的系统根本无法承受的.
**商用数据库无法满足需求**,一方面现有商用数据库的设计着眼点在于其通用性。另一方面对于底层系统的完全掌控会给后期的系统维护、升级带来极大的便利


### Bigtable 数据的存储格式

    `Bigtable is a sparse, distributed, persistent multidimensional sorted map.`

- Persistent：一个表是一个包含海量 Key-Value 键值对的 Map，数据是持久化存储的；
- Distributed：这个大的 Map 需要支持多个分区来实现分布式；
- Multidimensional Sorted Map：这个 Map 按照 Row Key 进行排序，这个 Key 是一个由 {Row Key, Column Key, Timestamp} 组成的多维结构；
- Sparse：每一行列的组成并不是严格的结构，而是稀疏的，也就是说，行与行可以由不同的列组成：

Bigtable 是一个*分布式, 多维, 映射表*. 表中的数据通过*一个行关键字（Row Key）、一个列关键字（Column Key）以及一个时间戳（Time Stamp）进行索引*. 
在Bigtable中一共有三级索引. 行关键字为第一级索引，列关键字为第二级索引，时间戳为第三级索引。
**物理结构上基于 Map 实现**。

### Bigtable的Webc存储逻辑可以表示为：

    (row:string, column:string, time:int64)→string

![](https://pic1.imgdb.cn/item/67d3e14588c538a9b5bd5685.png)

##### Row Key
    行关键字可以是任意字符串，最大容量为 64KB，但是在大多数场景下，字节数只有 10～100 Bytes 左右。Bigtable 按照 Row key 的字典序组织数据。
    什么是字典顺序？ASCII 码表中的后面的字符比前面的字符大，比如 c>a，因为 Row Key 本质就是字符串，因此可以使用字典顺序进行排序。
    利用这个特性可以通过选择合适的行关键字，使数据访问具有良好的局部性。如 Webtable 中，通过将反转的 URL 作为行关键字，可以将同一个域名下的网页聚集在一起。
    **网站的反转**指的是 www.google.com 反转为 com.google.www，类似于 Java 中的 package 的命名。因为 URL 解析过程本身就是从后往前解析的，这符合 URL 的使用逻辑。另一方面，方便域名管理，将**同一个域名下的子域名网页能聚集**在一起。
    拿 www.github.com 作为一个 URL 的一个例子，为 www 开头的 URL 建立集群的意义并不大（没有区分度，相当于没有建集群），但是将 com.github 域名建立集群就有一定的使用用途了。
    注意：在 Bigtable 中仅仅涉及一个 Row key 的读/写操作是原子的。

##### Tablet
    在 Bigtable 中，Row Key 相同的数据可以有非常多，为此 Bigtable 中表的行区间需要动态划分（也就是横向进行数据分区，横向的意思便是将表横着切），每个行区间称为一个 Tablet（子表）。Tablet 是 Bigtable 数据分布和负载均衡的基本单位，不同的子表可以有不同的大小。为了限制 Tablet 的移动成本与恢复成本，每个子表默认的最大尺寸为 200 MB。Tablet 是一个连续的 Row Key 区间，当 Tablet 的数据量增长到一定大小后可以自动分裂为两个 Tablet。同时 Bigtable 也支持多个连续的 Tablet 合并为一个大的 Tablet。

##### Column Key 与 Column Family
Column Key 一般都表示一种数据类型，Column Key 的集合称作 Column Family(列族)。**存储在同一 Column Family 下的数据属于同一种类型，Column Family 下的数据被压缩在一起保存。**
    Column Family 是 access control（访问控制）、disk and memory accounting（磁盘和内存计算）的基本单元。数据在被存储之前必须先确定其 Column Family，然后才能确定具体的 Column Key，并且表中的 Column Family 不宜过多，通常几百个。但 Column key 的个数并不进行限制，可以有无限多个。在 Bigtable 中列关键字的命名语法为：**family:qualifier 即 "列族:限定词"**，列族名称必须是可打印的字符串，限定词则可以是任意字符串。如 Webtable 中名为 anchor 的列族，该列族的每一个列关键字代表一个锚链接；anchor 列族的限定词是引用网页的站点名，每列的数据项是链接文本。
##### TimeStamp
    Bigtable 中的表项可以包含同一数据的不同版本，采用时间戳进行索引。时间戳是 64 位整型，既可以由系统赋值也可由用户指定。时间戳通常以 us（微秒）为单位。时间戳既可以由 Bigtable 进行分配，也可以由客户端进行分配，如果应用程序希望避免冲突，应当生产唯一的时间戳。
    表项的不同版本按照时间戳倒序排列（大的在前，时间戳越大表明数据加入的时间越晚），即最新的数据排在最前面，因而每次查询会先读到最新版本。为了简化多版本数据的管理，每个列族都有两个设置参数用于版本的自动回收，用户可以指定保存最近 N 个版本，或保留足够新的版本(如最近 7 天的内容)。
    在 Bigtable 论文的 Webtable 例子中，contents family 存储的时间戳是网络爬虫抓取页面的时间，表中的回收机制可以选择保留任一页面的最近 3 个版本。




























