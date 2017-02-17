#### [java手把手教程][第二季]java后端博客系统文章系统——No5

停更了一个月后，我们再次开始更新。具体原因只能说是过年事情太多，幼时不知努力，长大了却又事事缠身。

这一期主要是根据WordPress执行的效果来观察数据库，从而分析WordPress的程序设计。

项目github地址：https://github.com/pc859107393/SpringMvcMybatis

我的简书首页是：http://www.jianshu.com/users/86b79c50cfb3/latest_articles

上一期是：[[手把手教程][第二季]java 后端博客系统文章系统——No4](http://acheng1314.cn/?p=328)

![行走的java全栈](http://acheng1314.cn/wp-content/uploads/2016/10/行走的java全栈群二维码.png)

#### 工具
- IDE为**idea16**
- JDK环境为**1.8**
- gradle构建，版本：2.14.1
- Mysql版本为**5.5.27**
- Tomcat版本为**7.0.52**
- 流程图绘制（xmind）
- 建模分析软件**PowerDesigner16.5**
- 数据库工具MySQLWorkBench，版本：6.3.7build

#### 本期目标

- 根据WordPress的工作进行程序设计分析
- 完成文章保存和草稿保存相关程序流程分析

> 根据WordPress文章保存和草稿保存分析程序设计

首先我们打开WordPress登录到控制台后随便保存草稿和文章，然后导出数据库中posts表增加内容如下：

```json
{
"RECORDS":[
{
"ID":"329",
"post_author":"1",
"post_date":"02-16-2017 09:57:29",
"post_date_gmt":"02-16-2017 01:57:29",
"post_title":"[java 手把手教程][第二季]java 后端博客系统文章系统——No4",
"post_excerpt":"",
"post_status":"inherit",
"comment_status":"closed",
"comment_count":"0",
"ping_status":"closed",
"post_password":"",
"post_name":"328-revision-v1",
"to_ping":"",
"pinged":"",
"post_modified":"02-16-2017 09:57:29",
"post_modified_gmt":"02-16-2017 01:57:29",
"post_content_filtered":"",
"post_parent":"328",
"guid":"http://acheng1314.cn/?p=329",
"menu_order":"0",
"post_type":"revision",
"post_mime_type":"",
"comment_count(2)":"0"
},
{
"ID":"328",
"post_author":"1",
"post_date":"02-16-2017 09:58:19",
"post_date_gmt":"02-16-2017 01:58:19",
"post_title":"[java 手把手教程][第二季]java 后端博客系统文章系统——No4",
"post_excerpt":"",
"post_status":"publish",
"comment_status":"open",
"comment_count":"0",
"ping_status":"open",
"post_password":"",
"post_name":"java-%e6%89%8b%e6%8a%8a%e6%89%8b%e6%95%99%e7%a8%8b%e7%ac%ac%e4%ba%8c%e5%ad%a3java-%e5%90%8e%e7%ab%af%e5%8d%9a%e5%ae%a2%e7%b3%bb%e7%bb%9f%e6%96%87%e7%ab%a0%e7%b3%bb%e7%bb%9f-no4",
"to_ping":"",
"pinged":"",
"post_modified":"02-16-2017 09:58:19",
"post_modified_gmt":"02-16-2017 01:58:19",
"post_content_filtered":"",
"post_parent":"0",
"guid":"http://acheng1314.cn/?p=328",
"menu_order":"0",
"post_type":"post",
"post_mime_type":"",
"comment_count(2)":"0"
},
{
"ID":"327",
"post_author":"1",
"post_date":"02-14-2017 23:20:15",
"post_date_gmt":"02-14-2017 15:20:15",
"post_title":"我的草稿",
"post_excerpt":"",
"post_status":"inherit",
"comment_status":"closed",
"comment_count":"0",
"ping_status":"closed",
"post_password":"",
"post_name":"323-revision-v1",
"to_ping":"",
"pinged":"",
"post_modified":"02-14-2017 23:20:15",
"post_modified_gmt":"02-14-2017 15:20:15",
"post_content_filtered":"",
"post_parent":"323",
"guid":"http://acheng1314.cn/?p=327",
"menu_order":"0",
"post_type":"revision",
"post_mime_type":"",
"comment_count(2)":"0"
},
{
"ID":"326",
"post_author":"1",
"post_date":"02-14-2017 23:20:01",
"post_date_gmt":"02-14-2017 15:20:01",
"post_title":"",
"post_excerpt":"",
"post_status":"inherit",
"comment_status":"closed",
"comment_count":"0",
"ping_status":"closed",
"post_password":"",
"post_name":"323-revision-v1",
"to_ping":"",
"pinged":"",
"post_modified":"02-14-2017 23:20:01",
"post_modified_gmt":"02-14-2017 15:20:01",
"post_content_filtered":"",
"post_parent":"323",
"guid":"http://acheng1314.cn/?p=326",
"menu_order":"0",
"post_type":"revision",
"post_mime_type":"",
"comment_count(2)":"0"
},
{
"ID":"325",
"post_author":"1",
"post_date":"02-10-2017 22:34:41",
"post_date_gmt":"00-00-00 00:00:00",
"post_title":"自动草稿",
"post_excerpt":"",
"post_status":"auto-draft",
"comment_status":"open",
"comment_count":"0",
"ping_status":"open",
"post_password":"",
"post_name":"",
"to_ping":"",
"pinged":"",
"post_modified":"02-10-2017 22:34:41",
"post_modified_gmt":"00-00-00 00:00:00",
"post_content_filtered":"",
"post_parent":"0",
"guid":"http://acheng1314.cn/?p=325",
"menu_order":"0",
"post_type":"post",
"post_mime_type":"",
"comment_count(2)":"0"
},
{
"ID":"323",
"post_author":"1",
"post_date":"02-14-2017 23:20:15",
"post_date_gmt":"00-00-00 00:00:00",
"post_title":"我的草稿",
"post_excerpt":"",
"post_status":"draft",
"comment_status":"open",
"comment_count":"0",
"ping_status":"open",
"post_password":"",
"post_name":"",
"to_ping":"",
"pinged":"",
"post_modified":"02-14-2017 23:20:15",
"post_modified_gmt":"02-14-2017 15:20:15",
"post_content_filtered":"",
"post_parent":"0",
"guid":"http://acheng1314.cn/?p=323",
"menu_order":"0",
"post_type":"post",
"post_mime_type":"",
"comment_count(2)":"0"
}
]
}
```

在上面的数据中我们已经删除了文章内容的数据（数据量太大，不方便查阅）。然后我们仔细分析上面的json数据，我们可以得出结论如下：

1. 文章：
    - ID为329和328的表示文章，且为同一篇文章（编辑完成立即发布）。
    - 不同字段为：
        * ID
        * post_date
        * post_date_gmt
        * post_status
        * comment_status
        * ping_status
        * post_name
        * post_modified
        * post_modified_gmt
        * post_parent
        * post_type
    - 

通过上面的对比我们大致可以得出这样一个结论：

- 文章编辑完成发布后，会留下一个初始版本的记录和一个正式发布版本的记录。
- 正式发布的文章和文章历史记录的主要区别如下：

```json
---->正式发布
"post_status":"publish",
"comment_status":"open",
"ping_status":"open",
"post_name":"java-%e6%89%8b%e6%8a%8a%e6%89%8b%e6%95%99%e7%a8%8b%e7%ac%ac%e4%ba%8c%e5%ad%a3java-%e5%90%8e%e7%ab%af%e5%8d%9a%e5%ae%a2%e7%b3%bb%e7%bb%9f%e6%96%87%e7%ab%a0%e7%b3%bb%e7%bb%9f-no4",
"post_type":"post",
"post_parent":"0",
---->历史记录
"post_status":"inherit",
"comment_status":"closed",
"ping_status":"closed",
"post_name":"328-revision-v1",
"post_type":"revision",
"post_parent":"328",
```

2. 草稿：
    - ID为323、325、326、327的均为草稿，且为同一篇草稿。
    - 具体的不同区别也和上面的类似，所以说我们可以自行整理下即可。

3. 小结：

- 文章和草稿都是有完整的版本记录。
- 文章和草稿的格式类似。
- 草稿分为自动草稿和手动草稿。
- 版本记录也是完整的记录，只是一些关键的字段改变了下。

> 文章分组相关分析

```sql
SELECT
  `ID`,
  `post_title`,
  `post_date`,
  `post_content`
FROM
  `wp_posts`
WHERE
  `post_type` = 'post'
  AND
  `post_status` = 'publish'
ORDER BY
  `ID`
```

上面的语句能够查找出来公开的文章，文章ID一目了然。

同时我们观察数据库可以得出跟文章的归类相关的数据库有：

- wp_terms
- wp_term_taxonomy
- wp_term_relationships

但是这么多表都是文章分类相关的东西，那么文章分类又分为什么些呢？按照WordPress的简单构架支撑大量的数据来看，那么我们可以肯定文章标签和目录分类肯定是在一起的。所以我们先看最根本的wp_terms。

| term_id | name | slug | term_group |
|--|--|--|--|
|1|java web|java-web|0|
|2|C语言学习|how2use_c|0|
|3|Android开发|makeandroid|0|
|4|综合总结|all_log|0|
|5|个人生活|myself_life|0|
|6|post-format-aside|post-format-aside|0|
|7|转载|from_others|0|
|8|Android Coder|android-coder|0|
|9|友情链接|%e5%8f%8b%e6%83%85%e9%93%be%e6%8e%a5|0|
|10|JavaWeb|javaweb|0|
|11|java web|java-web|0|
|12|Spring|spring|0|
|13|Mybatis|mybatis|0|
|14|java后端|java%e5%90%8e%e7%ab%af|0|
|15|JavaWeb|javaweb|0|
|16|MySQL数据库|mysql%e6%95%b0%e6%8d%ae%e5%ba%93|0|
|17|全栈教程|%e5%85%a8%e6%a0%88%e6%95%99%e7%a8%8b|0|
|18|java|java|0|

上面这张表是我线上服务器上面的wp_term表，可能我们暂时不明白什么意思，不过问题不大。我们接着看wp_term_taxonomy。

| term_taxonomy_id | term_id | taxonomy | description | parent | count |
|--|--|--|--|--|--|
|1|1|category||0|17|
|2|2|category||0|0|
|3|3|category||0|20|
|4|4|category||0|6|
|5|5|category||0|1|
|6|6|post_format||0|44|
|7|7|category||0|8|
|8|8|link_category||0|2|
|9|9|nav_menu||0|0|
|10|10|link_category||0|0|
|11|11|post_tag||0|2|
|12|12|post_tag||0|4|
|13|13|post_tag||0|4|
|14|14|post_tag||0|3|
|15|15|post_tag||0|2|
|16|16|post_tag||0|3|
|17|17|post_tag||0|3|
|18|18|post_tag||0|1|

通过上面这种表我们就可以明白了term_id所对应的name分别是什么用的，他们分别有文章分组、文章标签、链接标记等。

但是说这么多都没把上面文章的文章分类在哪找到，所以我们接着看wp_term_relationships表里面的东西。

| object_id | term_taxonomy_id | term_order |
|--|--|--|
|1|8|0|
|2|8|0|
|9|4|0|
|9|6|0|
|11|4|0|
|11|6|0|
|16|3|0|
|16|6|0|
|···|···|···|

表里面数据还有很多此处暂时省略。

上面表中的object_id顾名思义就是说对象的ID，说明它不单是文章也还有其他分类的信息。

我们再看看我们线上的wp_posts（文章）表，里面的简略内容如下：

| ID | post_title | post_date | post_content |
|--|--|--|--|
|9|IT产品文档列表|2015-09-22 16:44:48|内容省略···|
|11|建站伊始，一切从头再来，本站用wordpress搭建，基于PHP。|2015-09-22 16:45:32|···|

其实数据不需要那么多，我们只需要一丢丢数据简单对比就能知道结果了。
- 文章ID为9和11的文章的term_taxonomy_id分别为：4、6、4、6
- term_taxonomy_id为4和6的term_id和taxonomy分别为：

| term_id | taxonomy |
|--|--|
|category|post_format|

- 最后我们在wp_terms这个表中可以看到term_id分别为4和6的分别是

| term_id | name | slug | term_group |
|--|--|--|--|
|4|综合总结|all_log|0|
|6|post-format-aside|post-format-aside|0|

所以最后我们通过这样就可以明白分类信息的大概查找结构，文章分类的大概查找如下：

文章id ➡ wp_term_relationships中的object_id对应的term_taxonomy_id ➡ wp_term_taxonomy的ID可以看到分别是什么分类同时可以查找到term_id ➡ 最后在wp_term表中根据term_id可以查找到具体的名称。

至此分类信息基本查找完成。


> 总结

1. 文章和草稿只是一些关键信息的不同
2. 文章和草稿都有完整的历史记录
3. 文章分类在文章关系表中
4. 文章关系表包含了文章目录、文章标签等
5. 文章其他属性都可以通过先在WordPress上面执行后逆向观察数据库窥到一二