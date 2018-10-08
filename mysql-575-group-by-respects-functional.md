最近碰到个“MySQL5.7的问题”，Error Code: 1055
Expression #4 of SELECT list is not in GROUP BY clause and contains nonaggregated column '***' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by，google问题的时候发现了两篇博客，翻译一下。

#MySQL 5.7.5: GROUP BY respects functional dependencies!

[From](http://rpbouman.blogspot.ae/2014/09/mysql-575-group-by-respects-functional.html) 

GROUP BY尊重函数依赖！

Today, Oracle announced the availability of the Development Milestone Release 15 of MySQL 5.7.5. The tagline for this release promises "Enhanced Database Performance and Manageability". That may sound rather generic, the actual list of changes and improvements is simply *huge*, and includes many items that I personally find rather exciting! Perhaps I'm mistaken but I think this may be one of the largest number of changes packed into a MySQL point release that I've witnessed in a long time. The list of changes includes improvements such as:

今天，Oracle宣布开发里程碑 MySQL 5.7.5 Release 15可用了。这个发布版的标语许诺“增强数据库的性能和可管理性”。 这个可能听起来相当普通，这实际的改变和提升列表是很大的，并且包含了许多条我个人认为相当兴奋的。或许是我弄错了，但是我认为这个可能是我见证的几次变动里边最大的一个。这些改变里边包含的改进，比如：

* InnoDB improvements: Simplified tablespace recovery, support for spatial indexes, dynamic configuration of the innodb_buffer_pool_size parameter(!), more efficient creation and rebuilding of indexes ("sorted index build")

InnoDB改进，简化了表空间恢复，支持空间索引，动态配置innodb_buffer_pool_size参数，更高效的创建和索引重建（排序的索引创建） 

* Several general improvements for spatial data, such as support for open polygons and functions to manipulate geohash values and GeoJSON documents  

几个空间数据的普通改进，比如对开放多边形和操作geohash values和GeoJSON文档的函数支持

* performance_schema additions and improvements, such as a new user_variables_by_thread table, addition of WORK_COMPLETED and WORK_ESTIMATED columns to the stage event tables, improvements to the wait event tables, instrumentation for InnoDB memory allocation in the memory summary tables. 

performance_schema新增与改进，比如，一个新的user_variables_by_thread表，新增了WORK_COMPLETED，WORK_ESTIMATED列到stage event 表，改进了wait event tables，InnoDB内存分配的指令在in the memory summary tables。

* Quite a bunch of optimizer improvements, such as better cost estimation for semi-join materialization, configurable cost model (by way of the mysql.server_cost and mysql.engine_cost system tables) and more exact index statistics.  

相当多的优化改进，比如更好的消耗评估对于semi-join实物化，配置消耗模型(by way of the mysql.server_cost and mysql.engine_cost system tables)，和更准确的索引统计。    

* Many improvements and additions that make replication more robust  

许多改进和新增使复制跟健壮。

* A more sane default SQL mode and GROUP BY behaviour  

更健全的默认SQL mode和GROUP BY行为。

This is very far from an exhaustive list; It really is not an exaggeration to say that there is much much more than I can cover here. Just see for yourself. 

这里跟详尽的更新列表还差很多，毫不夸张的说，这里有太多我包含不到的。自己看吧。  

Now, one of the changes I'd like to highlight in this post is the improved GROUP BY support. 

现在，我想要在这篇博客中重点谈及的一个改变是改进了GROUP BY的支持。  

## GROUP BY behavior before MySQL 5.7.5
MySQL 5.7.5之前的GROUP BY行为  

More than 7 years ago, I wrote an article on this blog called [Debunking GROUP BY Myths](http://rpbouman.blogspot.nl/2007/05/debunking-group-by-myths.html). The article is basically an explanation of the syntax and semantics of the SQL GROUP BY clause, with a focus on MySQL particular non-standard implementation.   

七年多以前，我写了一篇博客叫做[Debunking GROUP BY Myths](http://rpbouman.blogspot.nl/2007/05/debunking-group-by-myths.html).这篇文章基本上是SQL GROUP BY字句的语法，语义的解释，关注的是MySQL独有的不标准的SQL实现。  

Before MySQL 5.7.5, MySQL would by default always aggregate over the list of expressions that appear in the GROUP BY-clause, even if the SELECT-list contains non-aggregate expressions that do not also appear in the GROUP BY-clause. In the final resultset, MySQL would produce one of the available values for such non-aggregate expressions and the result would basically not be deterministic from the user's point of view.   

MySQL 5.7.5之前，MySQL默认总是聚合出现在GROUP BY字句里边的字段，即使SELECT里边包含不是GROUP BY里边的非聚合表达式。在最终的返回结果中，MySQL将产生从可用的值里边选一个为这个非聚合表达式，结果将基本上不是确定的，从用户的观点看。  

This behavior is not standard: SQL92 states that any non-aggregate expressions appearing in the SELECT-list must appear in the GROUP BY-clause; SQL99 and on state that any non-aggregate expressions that appear in the SELECT-list must be functionally dependent upon the list of expressions appearing in the GROUP BY. In this context, "functionally dependent" simply means that for each unique combination of values returned by the expressions that make up the GROUP BY-clause, the non-aggregate expression necessarily yields exactly one value. (This concept is further explained and illustrated in my original article.)   





