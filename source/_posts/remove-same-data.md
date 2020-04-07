---
title: 关于数据库去除重复数据
date: 2015.11.21 14:08:39
tags: [mysql]
categories: 后端
---

关于数据库去除重复数据已经是一个老问题了，甚至面试也会经常被问到这部分的问题。
最老的方法就是建立中间表了，由于我对数据库不是很熟悉，没有深入研究过，所以也不能一一列举方法。
目前想记录的一个东西是最近我爬取数据的时候，有大概几万条数据，但是重复率很高的情况，可以利用distinct等去筛选出来，但是假设我不想写代码，从数据库层面去操作的话，用distinct没办法配合delete操作，而且我懒，不愿意去建立中间表这些。
在翻找资料的过程中，终于找到了一种方法，如下：
```
DELETE
FROM
	table_name
WHERE
	id NOT IN (
select * from(
		SELECT
			min(id) AS id
		FROM
			table_name
		GROUP BY
			field)b
	)
```
但是这种去重效率并不高。
