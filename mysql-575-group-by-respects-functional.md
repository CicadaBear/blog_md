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



* Quite a bunch of optimizer improvements, such as better cost estimation for semi-join materialization, configurable cost model (by way of the mysql.server_cost and mysql.engine_cost system tables) and more exact index statistics.  

* Many improvements and additions that make replication more robust  


* A more sane default SQL mode and GROUP BY behaviour  



