金牌xxg订单列表SQL
背景
如下SQL执行慢


分析
 先开始看以为是join 的表太多了 然后把连表都去掉了 发现还是慢，explain 看了一下 发现
 a表 使用的createTime 索引，为啥用这个索引不知道， create_time 倒序 其实是和主键倒序应该是一样，然后改成 order by a.id desc limit 0,20 
时间只用了 0.188 秒（原来是 19 秒多 为啥 ？ 不知道） 然后explain 看下
我们可以看到 使用的 xxg 这个 索引 ref 是 const 也就是用二级索引 等值 查询的 ref 为const，然后把这个语句强制使用 xxg_id 为索引 发现也很快
explain 看下 只是比使用 order by id 多了一个 Using filesort（也就是mysql 不能使用索引进行排序）因为order by id 主键本身就是有序 的
ORDER BY 两种方式
1、利用索引（因为索引本身是有序的） 
2、如果没法用到索引那么就把数据查出来以后然后自己再做一次排序
为什么他不使用xxg_id 这个索引（瞎猜 可能是mysql 觉得 filesort 的成本要大于基于create为索引的成本 有时候mysql优化器也不是很准） 
扩展
order by 可以用到索引的时候示例：
1、有索引 key a_b_c
2、order by a || order by ab || order by abc 也就是最左原则
3、和where 列 同时 使用     where a = s and b = b order by c desc 也是最左原则
Comments:
MySQL 设计策略：如果内存足够，就多利用内存，减少磁盘访问
MySQL还可能会使用rowid排序，如果MySQL认为查询数据的单行数据太大，超过max_length_for_sort_data，就会将“全字段排序”算法更换为“rowid排序”
全字段排序 vs rowid 排序
如果 MySQL 实在是担心排序内存太小
会影响排序效率，才会采用 rowid 排序算法
这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据
如果 MySQL 认为内存足够大
会优先选择全字段排序，把需要的字段都放到 sort_buffer 中
这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据
结论：
对于 InnoDB 表来说，rowid 排序会要求回表多造成磁盘读，因此不会被优先选择
