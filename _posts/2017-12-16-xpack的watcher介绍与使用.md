# Watcher

------

Watcher已经自动被集成到X-pack里面了。
X-pack提供了一个API来创建，管理和测试watches。
一个watch描述了一个单一的告警，并能够包含众多的通知动作。

+ Schedule 时间表
	时间表用来运行一个query和检查状态

—+ Query
	要运行的query相当于condition的输入语句。watches支持所有elasticsearch查询语法，包括聚合查询

+ Condition
	Condition决定了是否执行actions，可以使用简单的conditions，也可以使用脚本来应对复杂的场景

+ Action
	可以有多种actions，如发送邮件，或者通过webhook将数据发送到第三方系统，或所引出query的结果

所有的watches历史都存储在Elasticsearch索引中。

**watches运行的权限较高，使用内建的watcher_admin或其他角色来manager_watcher，要确定用户是被信任的**


## Getting Started with Watcher

编排(Schedule)watches AND 定义input

Schedule Triggler
首先应该同步所有nodes和cluster的时间

watcher提供的triggler
+ hourly
+ daily
+ weekly
+ monthly
+ yearly
+ cron
+ interval

