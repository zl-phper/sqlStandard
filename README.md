# 基础规范
- 表存储引擎推荐使用InnoDB  
- 表字符集默认使用utf8，必要时候使用utf8mb4
<p>
    <font color=#FF0000 >
 解读： 

（1）通用，无乱码风险，汉字3字节，英文1字节

（2）utf8mb4是utf8的超集，有存储4字节例如表情符号时，使用它
    </font>
</p>

- 不推荐使用存储过程，视图，触发器，Event 
<p>
    <font color=#FF0000 >
 解读： 

（1）对数据库性能影响较大，互联网业务，能让程序干的事情，不要交到数据库

（2）调试，排错，迁移都比较困难，扩展性较差
    </font>
</p> 
    
# 命名规范
- 库名，表名，列名推荐用小写，采用下划线分隔 相同业务的表最好加上同样的前缀
- 库名，表名，列名最好见名知义，长度不要超过32字符 

<p>
    <font color=#FF0000 >
 解读： 

（1） mysql允许的最大字节为64
    </font>
</p>

# 表设计规范 
- 最好为你的表加上一个主键
<p>
    <font color=#FF0000 >
 解读：

https://imysql.com/?p=1234?p=1234
    </font>
</p>

- 禁止使用外键，如果要保证完整性，应由应用程序实现
<p>
    <font color=#FF0000 >
 解读：
（1） 互联网业务中不需要外键 完成性应该由自己程序保证
    </font>
</p>

- 建议将大字段，访问频度低的字段拆分到单独的表中存储，分离冷热数据 
<p>
    <font color=#FF0000 >
 解读：
（1） 大字段不频繁更新的字段单独存储
    </font>
</p>

# 列设计规范
- 根据业务区分使用tinyint/int/bigint，分别会占用1/4/8字节
- 根据业务区分使用char/varchar 
<p>
    <font color=#FF0000 >
 解读：

（1）字段长度固定，或者长度近似的业务场景，适合使用char，能够减少碎片，查询性能高

（2）字段长度相差较大，或者更新较少的业务场景，适合使用varchar，能够减少空间
    </font>
</p>

- 根据业务区分使用datetime/timestamp
<p>
    <font color=#FF0000 >
 解读：

（1）前者占用5个字节，后者占用4个字节，存储年使用YEAR，存储日期使用DATE，存储时间使用datetime，  timestamp 最大的长度 2038-01-19 03:14:07
    </font>
</p> 
 
 
- 禁止存储文件、图片内容

- 必须把字段定义为NOT NULL并设默认值
<p>
    <font color=#FF0000 >
 解读：

（1） hulk 也提交不上去 

（2）NULL的列使用索引，索引统计，值都更加复杂，MySQL更难优化

（3）NULL需要更多的存储空间
 
（4）https://www.cnblogs.com/mr-wuxiansheng/p/11578881.html
    </font>
</p> 

- 使用varchar(20)存储手机号，不要使用整数
<p>
    <font color=#FF0000 >
 解读：

（1） 如果你喜欢你可以用int 试试（特别是有符号的）
    </font>
</p> 

- 使用TINYINT来代替ENUM
<p>
    <font color=#FF0000 >
 解读：

（1） ENUM增加新值要进行DDL操作 虽然他们占据的空间一样
    </font>
</p> 

- 小数类型为decimal，禁止使用float和double。
<p>
    <font color=#FF0000 >
 解读：

（1） float和double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不

正确的结果。如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储。
    </font>
</p> 

# 索引规范 
- 唯一索引使用uniq_[字段名]来命名
- 非唯一索引使用idx_[字段名]来命名
- 禁止在区分度不高的字段建立索引
 </font>
</p> 


<p>
    <font color=#FF0000 >
 解读：

（1） 性别上建立索引本身也没什么意义
    </font>
</p> 
- 单张表索引数量建议控制在5个以内
    </font>
</p> 


<p>
    <font color=#FF0000 >
 解读：

（1） 索引过多肯定会影响写入

（2） hulk 也不让提交

（3） mysql挑选不对最优的索引
    </font>
</p> 

- 非必要不要进行JOIN查询，如果要进行JOIN查询，被JOIN的字段必须类型相同，并建立索引 
<p>
    <font color=#FF0000 >
 解读：

（1） join列数据类型不一致会导致扫表
 
（2） 最好拆成多条sql （尤其是分区表上 jion太恐怖）
    </font>
</p>

- 理解组合索引最左前缀原则，避免重复建设索引，如果建立了(a,b,c)，相当于建立了(a), (a,b), (a,b,c) 最好把区分度高的放在前面
- 不建议在频繁更新的字段上建立索引
- 业务上具有唯一特性的字段，即使是组合字段，也最好建成唯一索引。 
<p>
    <font color=#FF0000 >
 解读：

（1） 虽然你肯定会在程序中非常完美的校验，只要没有唯一索引根据墨菲定律，

必然有脏数据产生。
    </font>
</p>

- 在varchar字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度。 


# SQL规范

- 禁止使用select *，只获取必要字段
<p>
    <font color=#FF0000 >
 解读：

（1） 一般会被扣工资
 
（2） select *会增加cpu/io/内存/带宽的消耗

（3）指定字段能有效利用索引覆盖

（4）指定字段查询，在表结构变更时，能保证对应用程序无影响 
    </font>
</p>

- insert必须指定字段，禁止使用insert into T values()
<p>
    <font color=#FF0000 >
 解读：

（1） 指定字段插入，在表结构变更时，能保证对应用程序无影响
    </font>
</p>

- 禁止在where条件列使用函数或者表达式
<p>
    <font color=#FF0000 >
 解读：

（1） 回导致索引失效
    </font>
</p>

- 如果有order by的场景，请注意利用索引的有序性。order by最后的字段是组合索 引的一部分，并且放在索引组合顺序的最后，避免出现file_sort的情况，影响查询性能。 

<p>
    <font color=#FF0000 >
 解读：

（1） where a=? and b=? order by c;索引：a_b_c
    </font>
</p>

- 禁止使用属性隐式转换，使用相同类型进行比较（会导致索引失效）
-    NOT、!=、<>、!<、!>、NOT IN、NOT LIKE、%开头
    这些操作都用不到索引，导致全表扫描
- 利用覆盖索引来进行查询操作，来避免回表操作
- explain 你的sql 语句
- 删除和修改记录时，要先select，避免出现误删除，确认无误才能 提交执行。最好开个事物 
- 更新表结构时候 注释要记得一起更改
