---
title: SQL语句根据条件批量修改排序
tags:
  - Sql
categories:
  - 后端
abbrlink: eeb3b058
cover: /assets/image/BE009.jpg
poster:
  headline: SQL语句根据条件批量修改排序
  caption: 根据查询出来的条件批量修改序号自增
  color: white
date: 2023-12-31 11:29:15
---

#### **需求**
项目中有一个需求：文件夹可拖动操作，这就需要在数据库中有个排序的字段(order_num)，但目前我的数据库这个字段是后加的，创建出来就是null，现在要给每个数据后加上序号，数据少还行，百万条数据就不可能手改了。

#### **想法**
首先，我想到的是在sql中用for循环去做，后来查了下资料发现还有更简单的方法：
根据既定条件查询踹数据后，批量修改字段参数代码

#### **实现**
这里以**navicat**为例：

![](https://coscdn.banxx.cn/blog/PixPin_2023-12-30_13-55-35.png)
将下方代码粘贴上去
```sql
	/*下面这三行就是相当于定义变量   变量名，变量类型，默认值 */
 DECLARE  no_more_record int DEFAULT 0;
 DECLARE  akey int;
	/*首先这里对游标进行定义*/
	/*cur_record 也是一个随便取的名字 vanti_logistics.v_tag-->表名*/
 DECLARE  cur_record CURSOR FOR  
	 SELECT id from vanti_logistics.v_tag WHERE parent_id = 0;   
	/*这个是个条件处理,针对NOT FOUND的条件,当没有记录时赋值为1*/
	/* 判断上面你写的 查询语句是否有返回的数据 */
 DECLARE  CONTINUE HANDLER FOR NOT FOUND  SET  no_more_record = 1;
	/*接着使用OPEN打开游标*/
 OPEN  cur_record;
	/*把第一行数据写入变量中,游标也随之指向了记录的第一行*/
	/* 这里就相当于把上面的你写的select查询的查询结果第一条赋值给akey这个变量 */
 FETCH  cur_record INTO akey;
	/* 这里是一个循环判断 */
 WHILE no_more_record != 1 DO
		/* 设置默认初始值 设置0就从1开始计数 */
    SET @order_num = 0;
    /* 你所要循环执行的修改语句 */
    UPDATE vanti_logistics.v_tag 
		SET order_num = ( SELECT @order_num := @order_num + 1 ) 
		WHERE parent_id = akey;
    /* 一次循环结束再次取出下一个游标值 */
    FETCH  cur_record INTO akey;
 END WHILE;
	/*用完后记得用CLOSE把资源释放掉*/
 CLOSE  cur_record; 
SELECT 0;

```
如图所示保存并运行即可
![](https://coscdn.banxx.cn/blog/PixPin_2023-12-30_13-59-52.png)

不过目前的方法只适用于两级层级，多级的就将第一步查询语句条件去掉查所有就好，速度依然很快