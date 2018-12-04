---
layout: 'n'
title: 理解MySQL底层B+tree索引机制
date: 2018-12-04 20:12:07
tags: 
- MySQL
- 性能优化
- B+tree
- 索引
categories: 性能优化
typora-copy-images-to: 理解MySQL底层B-tree索引机制
typora-root-url: 理解MySQL底层B-tree索引机制
---

# 一、初识Mysql体系结构

**整体结构图**

![img](/wps4EA1.tmp.jpg)

- 1，Connectors 

  接入方 支持协议很多

- 2，Management Serveices & Utilities： 

  系统管理和控制工具

  例如：备份恢复，mysql复制集群等

- 3，Connection Pool 

  连接池：管理缓冲用户连接、用户名、密码、权限校验、线程处理等需要缓存的需求 

- 4，SQL Interface

  SQL接口：接受用户的SQL命令，并且返回用户需要查询的结果。比如select from就是调用SQL Interface

- 5，Parser

   解析器，SQL命令传递到解析器的时候会被解析器验证和解析。解析器是由Lex和YACC实现的。

- 6，Optimizer

  查询优化器，SQL语句在查询之前会使用查询优化器对查询进行优化

- 7，Cache和Buffer（高速缓存区）

   查询缓存，如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。 

- 8，Pluggable Storage Engines

  插件式存储引擎。存储引擎是MySql中具体的与文件打交道的子系统。也是Mysql最具有特色的一个地方。 Mysql的存储引擎是插件式的。

- 9，File System  

  文件系统，数据、日志（redo，undo）、索引、错误日志、查询记录、慢查询等

**运行时机理图**

![img](/wps5B52.tmp.png)

# 二、理解Mysql底层B+tree索引机制

写在前面的话

***正确***的创建***合适***的索引，是提升数据库查询性能的***基础***



## **索引是什么？**

索引定义：

索引是为了加速对表中数据行的检索而创建的一种分散存储的**数据结构**

![120421194082031](/120421194082031.png)

## 为什么要用索引？

索引意义：

​     索引能极大的减少存储引擎需要扫描的数据量

​     索引可以把随机IO变成顺序IO

​     索引可以帮助我们在进行分组、排序等操作时，避免使用临时表

## 为什么是B+Tree？

二叉查找树	

平衡二叉树  

多路平衡查找树 

加强版多路平衡查找树

 

详细了解上面的数据结构可以参见[帖子](https://www.cnblogs.com/vianzhang/p/7922426.html)

### 二叉查找树，Binary Search Tree

每个分支最多有两个（路）

![120421194082038](/120421194082038.Png)

### 平衡二叉查找树，Balance binary search tree

平衡二叉树（AVL树）在符合二叉查找树的条件下，还满足任何节点的两个子树的高度最大差为1。

先了解下磁盘的相关知识。

系统从磁盘读取数据到内存时是以磁盘块（block）为基本单位的，位于同一个磁盘块中的数据会被一次性读取出来，而不是需要什么取什么。



X < 10 --> P1

X = 10 --> 命中

X > 10 --> P2



![120421194082041](/120421194082041.Png)

一个磁盘块：

​	绿色：关键字 （这里关键字就比如是id = 10）

​	蓝色：数据区（比如图上为 id  为 10 整条记录 id 、name 、age  、 phoneNum）

​	粉色：P1 P2 子节点的引用

模拟查找关键字id = 8的过程：

​	1.从根节点开始找磁盘块1，读入内存，  发现  8 < 10 --> P1 ，一次IO操作

​	2.此时P1指向磁盘块2，读入内存，  	     发现 8 > 5 --> P2， 一次IO操作

​	3.此时P2指向磁盘块5，读入内存，	     发现 8 = 8，命中 ，一次IO操作

​	（这里的IO操作理解为读取一个磁盘块，包括关键字、数据区、子节点引用）

 上面索引到id为8的记录，需要3次IO操作、3次内存查找。毕竟上面是举例子，假如我们的老师teacher表数据量非常大，那id就很多，这样这个平衡二叉树的高度就很大，假如很不幸我们要找的id = 8的记录，在这颗树最大的高度，那这样的命中索引，需要很多次IO操作。

可想而知，这种索引的规则太随机，MySQL肯定不是采用的这样

**小结**

它太深了
​	数据处的（高）深度决定着他的IO操作次数，IO操作耗时大
它太小了
​	每一个磁盘块（节点/页）保存的数据量太小了
​	没有很好的利用操作磁盘IO的数据交换特性，
​	也没有利用好磁盘IO的预读能力（空间局部性原理），从而带来频繁的IO操作

## 多路平衡查找树，B-Tree

B-Tree是为磁盘等外存储设备设计的一种平衡查找树。



***绝对平衡树*** 

B-Tree中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个3阶的B-Tree： 

X < 17 --> P1
X = 17 命中
17< X < 35 --> P2
X = 35 命中
X > 35 --> P3

![120421194082048](/120421194082048.Png)

模拟查找关键字的过程：(省略)

B-Tree相对于AVLTree缩减了节点个数，使每次磁盘I/O取到内存的数据都发挥了作用，从而提高了查询效率。

## 加强版多路平衡查找树 B+Tree

B+Tree是在B-Tree基础上的一种优化，使其更适合实现外存储索引结构，InnoDB存储引擎就是用B+Tree实现其索引结构。



MySQL的B+Tree

左闭合B+Tree：
1 <= X < 28 --> P1
28 <= X < 66 --> P2
66 <= X --> P3

![120421194082052](/120421194082052.Png)

从上一节中的B-Tree结构图中可以看到每个节点中不仅包含数据的key值（关键字，子节点引用），还有data值（一整条记录）。而每一个页的存储空间是有限的，如果data数据较大时将会导致每个节点（即一个页）能存储的key的数量很小，当存储的数据量很大时同样会导致B-Tree的深度较大，增大查询时的磁盘I/O次数，进而影响查询效率。在B+Tree中，所有数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子节点上只存储key值信息，这样可以大大加大每个节点存储的key值数量，降低B+Tree的高度。



**B+Tree与B-Tree的区别**

1、B+节点关键字搜索采用闭合区间
2、B+非叶节点不保存数据相关信息，只保存关键字和子节点的引用
3、B+关键字对应的数据保存在叶子节点中
4、B+叶子节点是顺序排列的，并且相邻节点具有顺序引用的关系



## 为什么选用B+Tree ？

B+树是B-树的变种（PLUS版）多路绝对平衡查找树，他拥有B-树的优势
B+树扫库、扫表能力更强
B+树的磁盘读写能力更强
B+树的排序能力更强
B+树的查询效率更加稳定（仁者见仁、智者见智）

# Mysql B+Tree索引体现形式

## Mysql B+Tree索引体现形式 - Myisam

MyISAM是默认[存储引擎](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E)（Mysql5.1前）。它基于更老的ISAM代码，但有很多有用的扩展。（注意MySQL 5.1不支持ISAM）。 每个MyISAM在磁盘上存储成三个文件，每一个文件的名字均以表的名字开始，扩展名指出文件类型。

![120421194082063](/120421194082063.Png)

.frm文件存储表定义；

·MYD (MYData)文件存储表的数据；

.MYI (MYIndex)文件存储表的索引。

简介（[百度百科](https://baike.baidu.com/item/myisam/8970102?fr=aladdin)）

​	`要明确表示你想要用一个MyISAM表格，请用ENGINE表选项指出来：`

​	`CREATE TABLE t (i INT) ENGINE = MYISAM;`

​	`注释：老版本的MySQL使用TYPE而不是ENGINE（例如，TYPE = MYISAM）。MySQL 5.1为向下兼容而支持这个语法，但TYPE现在被轻视，而ENGINE是首先的用法。`

​	`一般地，ENGINE选项是不必要的；除非默认已经被改变了，InnoDB是默认[`[存储引擎](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E)`]（Mysql 5.1后）。`

​	`你可以用myisamchk工具来检查或修复MyISAM表。请参阅MySQL 5.1参考手册5.9.5.6节，“使用myisamchk做崩溃恢复”。你也可以用myisampack来压缩MyISAM表，让它们占更少的空间。请参阅MySQL 5.1参考手册8.2节，“myisampack，产生压缩、只读的MyISAM表”。`



![120421194082067](/120421194082067.Png)

## Mysql B+Tree索引体现形式 - innodb

InnoDB，是MySQL的数据库引擎之一，为[MySQL AB](https://baike.baidu.com/item/MySQL%20AB)发布binary的标准之一。InnoDB由Innobase Oy公司所开发，2006年五月时由[甲骨文公司](https://baike.baidu.com/item/%E7%94%B2%E9%AA%A8%E6%96%87%E5%85%AC%E5%8F%B8/430115)并购。与传统的[ISAM](https://baike.baidu.com/item/ISAM)与[MyISAM](https://baike.baidu.com/item/MyISAM)相比，InnoDB的最大特色就是支持了[ACID](https://baike.baidu.com/item/ACID/10738)兼容的[事务](https://baike.baidu.com/item/%E4%BA%8B%E5%8A%A1/5945882)（Transaction）功能，类似于[PostgreSQL](https://baike.baidu.com/item/PostgreSQL)。

![120421194082071](/120421194082071.Png)

简介（[百度百科](https://baike.baidu.com/item/innodb/8970025?fr=aladdin)）

​	事务型数据库的首选引擎，支持ACID事务，支持行级锁定。InnoDB是为处理巨大数据量时的最大性能设计。InnoDB存储引擎完全与MySQL服务器整合，InnoDB存储引擎为在主内存中缓存数据和索引而维持它自己的缓冲池。InnoDB存储它的表&索引在一个表空间中，表空间可以包含数个文件（或原始磁盘分区）。这与MyISAM表不同，比如在MyISAM表中每个表被存在分离的文件中。InnoDB 表可以是任何尺寸，即使在文件尺寸被限制为2GB的操作系统上。InnoDB默认地被包含在MySQL二进制分发中。Windows Essentials installer使InnoDB成为Windows上MySQL的默认表。

![120421194082075](/120421194082075-1543934044732.Png)

MySQL的innodb是以主键索引来组织数据结构的，加入你见表的时候，没有主键，innodb会自动隐式的建立一个主键索引，只是你外界看不见而已。

**辅助索引**命中的只是**主键索引**的**关键字**



## Innodb VS Myisam

![120421194082079](/120421194082079.Png)



Innodb: 假如你频繁的修改、更新辅助索引，主键索引是不需要重排序的

//TODO 这里描述的待补充，还没理解透彻

# 索引知识补充

列的离散性 count(distinct col) : count(col)

找出离散性最好的列？

![120421194082084](/120421194082084.Png)

name的离散性最大，越大离散性越好
结论：
​	离散性越高
​	选择性就越好

## 最左匹配原则

对索引中关键字进行计算（对比），一定是从左往右依次进行，且不可跳过

![120421194082088](/120421194082088.Png)

## 联合索引？so seay

单列索引
​	节点中关键字[name]
联合索引
​	节点中关键字[name,phoneNum]
**单列索引是特殊的联合索引**

联合索引列选择原则
1，经常用的列优先 【最左匹配原则】
2，选择性（离散度）高的列优先【离散度高原则】
3，宽度小的列优先【最少空间原则】

**机灵的李二狗**

经排查发现最常用的sql语句：

```mysql
Select * from users where name = ? ;
Select * from users where name = ? and phoneNum = ?;
```

机灵的李二狗的解决方案：

```mysql
create index idx_name on users(name);#冗余索引，根本用不上
create index idx_name_phoneNum on users(name,phoneNum);
```

## 覆盖索引

如果查询列可通过索引节点中的关键字直接返回，则该索引称之为
覆盖索引。
覆盖索引可减少数据库IO，将随机IO变为顺序IO，可提高查询性能

例：

```mysql
create index idx_name_phoneNum on users(name,phoneNum);

select name,phoneNum form users where name = "张三"；
```

在索引的子节点命中了 返回的列，就不需要再继续往B+Tree的底部叶子节点 读取数据了，减少了IO操作，提高了查询速度。

这就是为什么不建议大家select * from table 的原因，需要什么列就查什么列，可能会命中覆盖索引，这样就会极大的提高查询效率。

# 现在，你能都理解了么？

索引列的数据长度能少则少。

索引一定不是越多越好，越全越好，一定是建合适的。

匹配列前缀可用到索引 like 9999%，like %9999%、like %9999用不到索引；

Where 条件中 not in 和 <>操作无法使用索引；（<>在主键列上操作可能会使用索引，//TODO 后面在测试）

匹配范围值，order by 也可用到索引；

多用指定列查询，只返回自己想到的数据列，少用select *；

联合索引中如果不是按照索引最左列开始查找，无法使用索引；

联合索引中精确匹配最左前列并范围匹配另外一列可以用到索引；

联合索引中如果查询中有某个列的范围查询，则其右边的所有列都无法使用索引；