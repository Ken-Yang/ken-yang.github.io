---
layout: post
title: "What's MySQL partition"
description: ""
category: 
tags: [mysql, partition]
---
{% include JB/setup %}


最近在面試幾個candidate，都說自己有MySQL tuning的經驗，
但進一步問下去，發現連MySQL怎麼用BTree放index，或者怎麼使用partition or sharding都不了解，所以打算來寫一系列的MySQL文章，這篇就先挑簡單的partition來寫。

---
### 1. What's Partition and Why?
---

簡單說，Partition是將你的table，根據你的配置去divide成好幾個part；最重要的是，資料都是在同一個DB裡面，這對application來說，可以無感的使用它，卻又享有它的優點。</br>

那麼為何要用partition呢？partition的優點在於：

- `Partition Pruning`：基於已經知道資料在哪個partition的條件下，讓你的SQL operation變快，這包含了`select, insert, update ,delete`
- `刪除資料`：當你的table很大時，刪除是非常耗時的一件工作，有了partition以後，你可以直接drop該partition，且時間上可以快上非常多。

</br>


<!--more-->


---
### 2. When to use partition?
---

而使用partition的時機點，取決於資料量的大小，如果當你的資料只有幾千筆時，其實做partition的意義就不大了。
</br>
不過partition也不是萬靈丹，當你的資料量成長到一定的數量時，可能也意味著你的application使用人數也成長了，當你所有的操作都在同一個DB上時，partition也是救不了你的application，所以才會有進階的做法，就是`sharding`，但`sharding`就比較複雜了，不管是對application還是replication來說，所以這就等下一篇有機會再寫吧。
</br>
簡單說，基於下面的條件你就可以使用partition：

- 如果你的table很大
- 如果你知道你一定會去查詢會被partition的column
- 如果你想要快速的刪除歷史資料


</br>

---
### 3. Horizontal vs Vertical
---

Partition分為兩種類型：

1. Horizontal：同一個row資料可以存在不同的partition上
2. Vertical：不同的column存在不同的partition上

目前MySQL只支援horizontal partition。


</br>

---
### 4. Types of MySQL partition
---

而上面有說到，你可以配置你想要MySQL怎麼擺放你的資料，而配置的種類如下：

1. Key(column)：deleted by partition not works
2. Hash(INT expression)：deleted by partition not works
3. RANGE(INT expression) or RANGE COLUMNS()
4. LIST(INT expression) or LIST COLUMNS()

接下來會ㄧ一說明各個的差別。


</br>


#### 4-1. Key partitioning


如果你是使用key partitioning的話，MySQL會用它自己的hash function算出你這筆資料要放哪裡。</br>
key的用法為`key(column)`，裡面必須為table中的某個欄位，範例如下：


```sql
CREATE TABLE t (
	id INT, 
	create_time DATETIME
)    
PARTITION BY KEY(create_time)
PARTITIONS 10;
```


上面的範例，就會根據data的create_time，去divide出10個partition，然後當塞入資料時，會決定要放置到哪個partition。
有二點要注意，key只能使用column當作判斷指標，不能使用expression，以及如果使用key的話，無法根據partition進行刪除。



</br>

#### 4-2. Hash partitioning


Hash的用法為`HASH(INT expression)`，裡面一定要是

- Int
- expression且是回傳Int，如`MONTH(now())`


```sql
CREATE TABLE t (
	id INT, 
	create_time DATETIME
)    
PARTITION BY HASH(MONTH(create_time))
PARTITIONS 12;
```

上面的範例，就會根據data的create_time，去divide出12個partition。然後HASH一樣無法根據partition進行刪除。



</br>

#### 4-3. LIST partitioning


LIST的用法為`LIST(INT expression)`，參數也一定要是

- Int
- expression且是回傳Int，如`MONTH(now())`


```sql
CREATE TABLE t (
	id INT, 
	create_time DATETIME
)    
PARTITION BY LIST(DAYOFWEEK(create_time)) (
	PARTITION pMon VALUES IN (1),
	PARTITION pTue VALUES IN (2),
	PARTITION pWed VALUES IN (3),
	PARTITION pThu VALUES IN (4)		
);
```

上面的範例，就會根據data的DAYOFWEEK(create_time)，去divide出4個partition。


</br>

#### 4-4.  Range partitioning


Range的用法為`Range(INT expression)`，參數也一定要是

- Int
- expression且是回傳Int，如`YEAR(now())`


```sql
CREATE TABLE t (
	id INT, 
	create_time DATETIME
)    
PARTITION BY RANGE(YEAR(create_time)) (
	PARTITION p2017 VALUES LESS THAN (2018),
	PARTITION p2018 VALUES LESS THAN (2019),
	PARTITION p2019 VALUES LESS THAN (2020)
);
```

上面的範例，就會根據data的YEAR(create_time)，簡單說，就是會根據年份去divide出4個partition。
</br>
使用Range，有二點要注意，必須由低往高去設定，以及會無法insert超出range的資料。


</br>

---
### 5. Manage partition
---

上面已經說明怎麼去設置不同type of partition了，接著要說明怎麼`新增` and `刪除` partition，

#### 5-1. Drop partition

前面有說到，當table資料量很大時，delete會是一個很耗時的工作，但可以透過drop partition的方式達到相同目的，
當然前提是你知道你的資料在哪個partition上。

```sql
ALTER TABLE t DROP PARTITION p2019;
```


#### 5-2. Add partition

由於一開始在建立range or list partition時，不一定會一次建好所有的partition，
以上面的`range`範例來說，我們也只建立了近3年的資料，但如果到了2020以後，就會無法`insert`了，
所以勢必得動態add partition。

```sql
ALTER TABLE t ADD PARTITION (PARTITION p2020 VALUES LESS THAN (2021));
```


</br>

---
### 6. Partition Pruning
---

接著要講partition的其中一個優點，就是partition pruning，先建立二張table，一張有partition，另外一張則沒有，
然後依樣塞入一些年份不同的資料。


```sql
CREATE TABLE t_partioned (
	id INT, 
	create_time DATETIME
)    
PARTITION BY RANGE(YEAR(create_time)) (
	PARTITION p2017 VALUES LESS THAN (2018),
	PARTITION p2018 VALUES LESS THAN (2019),
	PARTITION p2019 VALUES LESS THAN (2020)
);
insert into t_partioned values(1,now(3) + interval 0 year);
insert into t_partioned values(2,now(3) + interval 0 year);
insert into t_partioned values(3,now(3) + interval 0 year);
insert into t_partioned values(4,now(3) + interval 1 year);
insert into t_partioned values(5,now(3) + interval 1 year);
insert into t_partioned values(6,now(3) + interval 1 year);
insert into t_partioned values(7,now(3) + interval 1 year);
insert into t_partioned values(8,now(3) + interval 2 year);
insert into t_partioned values(9,now(3) + interval 2 year);
insert into t_partioned values(10,now(3) + interval 2 year);
insert into t_partioned values(11,now(3) + interval 2 year);



CREATE TABLE t_no_partioned (
	id INT, 
	create_time DATETIME
);
insert into t_no_partioned values(1,now(3) + interval 0 year);
insert into t_no_partioned values(2,now(3) + interval 0 year);
insert into t_no_partioned values(3,now(3) + interval 0 year);
insert into t_no_partioned values(4,now(3) + interval 1 year);
insert into t_no_partioned values(5,now(3) + interval 1 year);
insert into t_no_partioned values(6,now(3) + interval 1 year);
insert into t_no_partioned values(7,now(3) + interval 1 year);
insert into t_no_partioned values(8,now(3) + interval 2 year);
insert into t_no_partioned values(9,now(3) + interval 2 year);
insert into t_no_partioned values(10,now(3) + interval 2 year);
insert into t_no_partioned values(11,now(3) + interval 2 year);
```

</br>

然後我們可以用MySQL的`explain`去觀察select這2張table時，會有什麼變化，以下面的例子來看，
可以發現當查詢有partition的table時，也只去scan 7 rows，且直接從partition p2017, p2018去獲取資料；
反之去查詢沒有partition的table時，是整張table都掃了一次，才找到要的資料。


```sql
mysql>  explain select * from t_partioned where create_time > now() and create_time < now() + interval 1 year;
+----+-------------+-------------+-------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table       | partitions  | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------------+-------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_partioned | p2017,p2018 | ALL  | NULL          | NULL | NULL    | NULL |    7 |    14.29 | Using where |
+----+-------------+-------------+-------------+------+---------------+------+---------+------+------+----------+-------------+

mysql> explain select * from t_no_partioned where create_time > now() and create_time < now() + interval 1 year;
+----+-------------+----------------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table          | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+----------------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | t_no_partioned | NULL       | ALL  | NULL          | NULL | NULL    | NULL |   11 |    11.11 | Using where |
+----+-------------+----------------+------------+------+---------------+------+---------+------+------+----------+-------------+
```

</br>
但是要做到partition pruning，根據不同的配置是有不同的條件的：

1. `PARTITION BY KEY(id)`
	- WHERE id = 'xxxx';
2. `PARTITION BY HASH(MONTH(create_time))`
	- WHERE create_time = '12';
3. `PARTITION BY LIST(DAYOFWEEK(create_time))`
	- WHERE create_time = DAYOFWEEK(now());
3. `PARTITION BY RANGE(YEAR(create_time))`
	- WHERE create_time = now();
	- WHERE create_time between now() AND now() + interval 1 year;
	- WHERE create_time > now();




</br>

---
### 7. Sub partition
---

sub partition就是可以讓你在partition中再一次地切割，不過sub partition有先限制，

- 只有`RANGE` and `LIST`是可以使用sub partition的
- sub partition type只能是`HASH` and `KEY`

以下面的例子來看，我們主要的partition是以RANGE(id)為主，然後有四個partition（p0,p1,p2,p3），每個底下又根據KEY(name)各切了2個，所以這例子來看，總共有2x4=8個partitions。

```sql
CREATE TABLE t (
	id INT NOT NULL AUTO_INCREMENT,
	name VARCHAR(25) NOT NULL,
	PRIMARY KEY pk (id, name)
) 
PARTITION BY RANGE(id)
SUBPARTITION BY KEY (name)
SUBPARTITIONS 2 (
    PARTITION p0 VALUES LESS THAN (5),
    PARTITION p1 VALUES LESS THAN (10),
    PARTITION p2 VALUES LESS THAN (15),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

</br>
查詢時，可以直接下

```sql
SELECT id FROM t PARTITION (p0sp1);
```



</br>

---
### 8. Conclusion
---

上面是partition的基本知識，這裡彙整一些重點：

- partition不是萬靈丹，當用戶數多時，還是得考慮sharding其他方式
- 如果你的table很大，可以考慮使用
- 如果你一定會查詢會被partition的column，可以考慮使用
- 如果你想要快速的刪除歷史資料，可以考慮使用
- Type of KEY and HASH不能根據partition去刪除資料
- 只有RANGE and LIST可以使用sub partition





</br>
</br>
</br>













</font>