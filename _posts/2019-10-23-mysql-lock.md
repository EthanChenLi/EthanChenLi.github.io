---
layout: post
title:  "乐观锁+版本号解决锁竞争问题"
date:   2019-10-23 11:23:00 +0800
categories: default
tags: mysql
comments: 1
---
### 乐观锁+版本号解决锁竞争问题

  在高并发的场景下，经常会遇到这种情况：
A请求过来，查询出来一条数据，进行update操作，与此同时B请求也在这个时候过来，对这条数据进行查询，并进行操作。此时就会出现B在A之后进行查询操作，但是实际B的数据却被A覆盖。

这种情况并不少见，有时候会为了避免这种情况，我们会引入锁来解决这种问题，常见的比如使用ReetrantLock或者synchronized同步块等悲观锁的方式来实现，其实在这里，我们也可以尝试使用乐观锁+版本号的方式来进行处理。

可以举个更具体的案例来说明一下。
```sql
select num from store where id='1';
```
此时获取到num=99
这时候B请求进来后，也发起查询
```sql
select num from store where id='1';
```
此时由于A请求尚未进行update，B请求获取到的num也等于99。
这时候A进行update操作
```sql
update store set num =${num} +1 where id='1';
```
这时候写入数据库的num即为100
此时B请求也发起了更新操作：
```sql
update store set num =${num} +1 where id='1';
```
这时候我们的预期本应该是101的，但是实际上B又在数据库写入了100.

---

#### 解决方案：
这时候我们可以引入一个版本号的概念，来巧妙的解决这个问题。

A请求
```sql
select num，version from store where id='1';
```
此时假设num=99，version=1,这时候B依然发起select请求：
```sql
select num，version from store where id='1';
```
没有意外，B取到的version也是1，这时候A进行update写入，此时：
```sql
update store set num =${num} +1,version=${version}+1 where id='1' and version='1'
```
由于A进行行级锁，此时version也变成了2。
B这时候发起操作，由于select的时候，版本号依然为1，此时实际B无法进行update的
```sql
update store set num =${num} +1,version=${version}+1 where id='1' and version='1'
```
我们可以看到由于在数据库的版本号已经变成2,了，实际version=1会让B请求的更新数据不能成功。

----
###### 转载至CSDN。
###### 原文链接：https://blog.csdn.net/wueryan/article/details/85239700