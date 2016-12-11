转眼到了博客系统第二章了。这一张我们主要介绍文章系统。毕竟博客系统的核心就是文章的发布和阅读。闲话不多说，老规矩走起来。

[[手把手教程][JavaWeb]第一季点击这里查看所有文章](http://www.jianshu.com/notebooks/4409922/latest)。当然，也可以直接访问[我的博客](http://acheng1314.cn)。

#### 工具
- IDE为**idea16**
- JDK环境为**1.8**
- gradle构建，版本：2.14.1
- Mysql版本为**5.5.27**
- Tomcat版本为**7.0.52**
- 流程图绘制（xmind）
- 建模分析软件**PowerDesigner16.5**

---
> 思维导图

按照前面我们**第二季第一章**阐述的，我们需要先了解我们这个文章系统的整个功能模块组合，也就是我们的思维导图，只有这样才能实现整体功能的架设。

![第二章文章系统思维导图](http://acheng1314.cn/wp-content/uploads/2016/11/我的博客第二章文章系统思维导图.png)

其实在上面的系统中，我已经把前端用户的文章查阅功能排除掉的。为什么我这里会单独排掉前端的查阅呢？前端的文章查阅功能基本在后端的所有文章中已经有体现相应功能。大概功能如下：
- 前端文章查阅
- 文章列表
- 文章归档
- 文章分类
- 文章详情

> 流程图

![第二章文章阅读-发布流程图](http://acheng1314.cn/wp-content/uploads/2016/11/我的博客第二章文章阅读-发布流程图.png)

在上面的流程图中，我们可以看到我们清楚的把业务流程描述出来了。可能很多哥们会说我们有其他不一样的方式，或者类似的方式但是实现比现在的强势，这个无可否认。但是我认为这个是别人项目中存在且我使用的很符合个人习惯的东西。好的东西要学习，不友好的东西我们需要自己改进。

首先我们访问站点的方式只有访问主页，然后才会有web应用的展示，也就是说我们网站的首页是我们web应用的总入口。

而我们主页的功能也是需要围绕我们的中心——博客来制作，这样才能达到我们建设这个后端的目的。所以首页元素需要有以下方面：

- 文章列表
- 文章归类
- 作者介绍
- 热门文章
- 最高评论
- 最近动态
- 联系信息（二维码）
- 标签导航
- 等···


> 数据流图

![第二章文章系统-数据流图](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第二章文章系统-数据流图.png)

为什么我们需要数据流图，我们不是为了软件工程二故意做这个数据流图。而是数据流图能清晰的表明我们这些流程中需要哪些关键的东西，能在一定程度上反应业务逻辑。所以我们做这个还是有意义。在上面我们可以看到在我们程序流转的过程中，我们需要知道具体的文章ID才能进行详情查看操作，所以我们在拿到列表的时候就需要把文章ID拿到，同时文章归档的依据信息，也需要拿到，大概需要哪些简单的东西，具体跟下面首页的json数据相关。具体的首页预想效果如下图：

![第二章文章系统-博客样图](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第二章文章系统-博客样图.png)

当然具体的[原作者的博客请看这里](https://www.vtrois.com/)。原作者的导航栏在右边，个人喜好，所以改到左边。根据这一张图，我们也能看到大概的功能如下：

- 博客文章列表展示
- 作者信息展示
- 最新、热点、随机文章
- *日期归档导航
- 标签导航

> 数据来源

按照第二季开发标准来说，前端页面展示的数据都是尽量从服务器接口获得，将前后端解耦。所以按照通用接口标准来说，我们首页数据需要JSON的标准数据。分析可得，我们的json格式大概如下：

``` json
{"code":1,
"msg":"success",
"data":{
"posts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"totalNum":20,
"author":{},
"newPosts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"hotPosts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"randomPosts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"tag":{},
"date":{}
}
}
```

可能一些朋友看到这里就会迷糊了，你的json数据的实体类型怎么来的呢？其实我们一开始就提过我们的数据库是wordpress的数据库，也就是数据内容是来自我的个人博客系统上面的数据库。所以我们需要看看wrodpress的博客系统上面文章表的结构和内容才能推测是表中字段及其分布各有什么意义。具体的数据库表结构如下：

```
DROP TABLE IF EXISTS `wp_posts`;
CREATE TABLE `wp_posts` (
`ID` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`post_author` bigint(20) unsigned NOT NULL DEFAULT '0' COMMENT '作者ID',
`post_date` datetime NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '文章创建时间',
`post_date_gmt` datetime NOT NULL DEFAULT '0000-00-00 00:00:00' COMMENT '文章最近修改时间',
`post_content` longtext COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '文章内容',
`post_title` text COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '文章标题',
`post_excerpt` text COLLATE utf8mb4_unicode_ci NOT NULL,
`post_status` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'publish' COMMENT '文章状态',
`comment_status` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'open' COMMENT '评论状态',
`ping_status` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'open' COMMENT 'ping状态',
`post_password` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '文章密码',
`post_name` varchar(200) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '文章名字',
`to_ping` text COLLATE utf8mb4_unicode_ci NOT NULL,
`pinged` text COLLATE utf8mb4_unicode_ci NOT NULL,
`post_modified` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',
`post_modified_gmt` datetime NOT NULL DEFAULT '0000-00-00 00:00:00',
`post_content_filtered` longtext COLLATE utf8mb4_unicode_ci NOT NULL,
`post_parent` bigint(20) unsigned NOT NULL DEFAULT '0',
`guid` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
`menu_order` int(11) NOT NULL DEFAULT '0',
`post_type` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'post' COMMENT '文章类型',
`post_mime_type` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '' COMMENT '文件类型',
`comment_count` bigint(20) NOT NULL DEFAULT '0' COMMENT '评论数',
PRIMARY KEY (`ID`),
KEY `type_status_date` (`post_type`,`post_status`,`post_date`,`ID`),
KEY `post_parent` (`post_parent`),
KEY `post_author` (`post_author`),
KEY `post_name` (`post_name`(191))
) ENGINE=InnoDB AUTO_INCREMENT=289 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
从上面的文章信息表中我们可以看到这一张表只是用来存储所有的文章的基本信息，但是文章的一些其他信息都是没有的，比如说：

- 评论
- 特色图片
- 文章归档
- 等···

一般来说，我们的常规思路是需要将这些信息关联在一起的，而且这个思路也是没错的。但是可能有的实现我们并没有较好的设计思想，所以我们可以简单的把数据库逆向到模型。所以闲话不多说，直接在有wrodpress环境的电脑上面链接数据库，打开wordpress数据库，选择逆向到模型。那么，数据库逆向模型如下所示：

![第二章文章系统-wordpress数据库模型](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第二章文章系统-wordpress数据库模型.png)

从上面的数据库模型中我们可以看出维持wordpress中心的有几张表，如下：

- wp_posts  文章基础信息表
- wp_postmeta   文章扩展数据表
- wp_comments   评论基本表
- wp_commentmeta    评论扩展表
- wp_links  链接表
- wp_options    设置信息表
- wp_users  用户信息表
- wp_usermeta   用户信息扩展表

为什么我说上面这几张表是核心表呢？首先我们可以看到这几张表都是存储了博客系统的一些基本的东西。接着我们可以看到这些各个表中一些关联的表都是有彼此的键对应其他表的主键，所以看到这里大家可能也就心里有数。

所以上面我们的json信息中的实体类型该怎么设定也就是很明显的，必须对应数据库字段嘛。既然都这样了，那我们是不是也可以进一步猜想出其他的json内容呢？

> 日期归档

文章按照日期归档相信很多人都看到过，大概样子就是一个下拉列表中显示年月日后面加上数量，大概样子如下（节约流量，不上图）：
- 请选择日期 ↓ 
- 所有
- 2016年11月12日（2）
- 2016年11月15日（1）
- 2016年10月28日（3）

我们要把这样的效果做出来，其实可以直接把文章信息传递给前台让前端完成。但是数据量过多的时候，网络传输也就相对吃力，所以我们还是直接后端处理，将网络传输的数据最精简。

那么我们简单的首页集合的数据应该如下所示了：

```
{"code":1,
"msg":"success",
"data":{
"posts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"totalNum":20,
"author":{},
"newPosts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"hotPosts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"randomPosts":[
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"},
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "文章内容",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"}
],
"tag":{},
"date": [
{
"date": "2016-11-22",
"idList": [
"286"
]
},
{
"date": "2016-5-19",
"idList": [
"192",
"191"
]
}
]
}
}
```
这里应该有朋友可能会问，为啥你的date（根据日期归档）的json数据这么奇怪呢？

其实我们最直接的可以看到，在上面的日期归档的json中，日期可以很直观的看出来，同时idList中把文章ID也是展示出来的，所以我们根据ID和日期都还是可以互相参考的，同时ID的数量可以让我们明白每个日期有多少篇文章。

既然我们在上面把基本的首页框架数据归类，写出的json接口，同时通过逆向开发的思路等把项目我们需要使用的一些模型图完成了，这样接下来就是具体编码的事情。 具体的编码问题，且听下回分解。

---
> 福利：用户密码算法

核心算法：SHA-256

步骤：

- 注册用户
- 客户端进行16位MD5小写加密
- 生成随机的salt
- 将密码和salt进行SHA-256加密
- 数据库存入用户信息和对应的salt


---
这一期，我们把文章系统一些做了基础的分析，下一期我们需要完成wordpress数据库内容分析和文章系统模块开发，和文章的撰写相关的东西。其实经过上一季的一些东西我们能明白，项目开发中的一些基本思想，但是可能我们最终目的是倚赖wordpress的博客。所以在实际开发中，我们可以参考别人的完成并加以列用。
