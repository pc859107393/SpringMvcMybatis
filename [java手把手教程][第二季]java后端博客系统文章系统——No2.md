这是博客系统的第三章，也是坚持写这个系列文章的第三个月了。在这期间我建立了全栈技术交流群，感谢一路鼓励我的朋友们。也要感谢我的大学导师，是他们在我需要的时候，告诉我做人的品质。

今天这一篇，主要是关于上一张的编码实现。为什么我要单路分离出来？因为做事要分先后，明白道理，执行才能确定无误。

我们还是从老套路（Dao→Service→Controller）做起来，让我们先看看数据存储相关的东西吧。

项目github地址：https://github.com/pc859107393/SpringMvcMybatis

我的简书首页是：http://www.jianshu.com/users/86b79c50cfb3/latest_articles

上一期是：[优雅的SpringMvc+Mybatis应用（七）](http://www.jianshu.com/p/def0076976aa)

---
#### wordpress做的文章存储

在上次我们已经看过了wordpress的数据库模型（有朋友问我什么是逆向分析，拿着别人的产品逆向推导这就是逆向分析），我们可以很清楚的看到数据库关于文章存储的两张表，它们分别存储了文章的主体信息和文章的其他信息，具体的我们再看看数据库模型：

![第二章文章系统-wordpress数据库模型](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第二章文章系统-wordpress数据库模型.png)

在上面的途中，我们很明显的看到数据库关于文章的存储主要分为两张表：

- wp_posts 存放文章主体信息
- wp_postmeta   存放文章的附加信息

我们先看看wp_posts的主要结构：

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

上面的数据库表，也是根据表生成的sql语句，当然注释是我加上去的。

可能看到这里很多朋友说我们现在只看到了表结构，而且又是你添加的注释，你这想怎么忽悠就怎么忽悠。既然这样，我们不妨一看数据库。

![我的博客第三章文章系统-数据库截图片段](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第三章文章系统-数据库截图片段.png)

嗯，上面的图需要放大后才看得清楚==  这个有点尴尬。
从上面的图中我们可以看到如下关键信息：

| id | post_author | post_date | post_content | post_title | post_status | post_type | post_mime_type |
| -- | -- | -- | -- | -- | -- | -- | -- |
| 286 | 1 | 2016-11-22 18:51:37 | 这是文章内容 | Android-MVP架构 | publish | post |  |
| 277 | 1 | 2016-11-08 00:37:07 |  | ssm应用七-访问列表-流程图 | inherit | attachment | image/png |
| 23 | 1 | 2015-09-26 22:55:30 |  | YKT主要界面示例--源码 | inherit | attachment | application/rar |

关键信息就和上面的类似了，为什么我挑选这三个：
- 博客的系统，主要的就是文章存储和多媒体资源存储
- 上面三个分别代表了文章、图片、rar压缩包
- 上面这个缩略表，刚好提取了目前我们可能会用到区分的不同信息

没时间解释了，我们直接分析上面的数据库表中的字段。
- 文章
- post_content一般有内容
- post_status 会有多种状态
- post_type 指向文章 post
- post_mime_type为空
- 多媒体文件
- post_content 为空
- post_status一般是inherit
- post_type 一般是 attachment
- post_mime_type 一般为具体的文件类型，如：	image/png、	application/rar

所以根据上面的一些信息，我们可以开始实现我们文章系统下面的文章列表接口了，首先按照老规矩实现Dao层：

```
import cn.acheng1314.domain.PostBean;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;

import java.io.Serializable;
import java.util.List;

/**
* Created by 程 on 2016/11/27.
*/
@Repository
public interface PostDao extends Dao<PostBean> {

@Override
int add(PostBean postBean);

@Override
int del(PostBean postBean);

@Override
int update(PostBean postBean);

@Override
PostBean findOneById(Serializable Id);

@Override
List<PostBean> findAll();


List<PostBean> findAllNew();

/**
* 分页查询
*
* @param offset 起始位置
* @param limit  每页数量
* @return
*/
List<PostBean> findAllPublish(@Param("offset") int offset, @Param("limit") int limit);

/**
* 获取总的条数
*/
int getAllCount();

List<PostBean> getAllPostDateCount();
}

```

其实上面的接口我们可以看到和以前的差不多，毕竟数据库操作就是一些基本的增删改查。没道理的，我们必须接着实现mapper，mapper如下：

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.acheng1314.dao.PostDao">
<select id="findAllPublish" resultType="cn.acheng1314.domain.PostBean">
SELECT
`ID`,`post_title`,`post_date`,`post_content`
FROM
`wp_posts`
WHERE
`post_type`='post'
AND
`post_status`='publish'
ORDER BY
`ID`
DESC
LIMIT #{offset}, #{limit}
</select>

<select id="findAllNew" resultType="cn.acheng1314.domain.PostBean">
SELECT
`ID`,`post_title`
FROM
`wp_posts`
WHERE
`post_type`='post'
AND
`post_status`='publish'
ORDER BY
`ID`
DESC
LIMIT 1, 10
</select>

<select id="getAllCount" resultType="int">
SELECT
COUNT(*)
FROM
`wp_posts`;
</select>

<select id="getAllPostDateCount" resultType="cn.acheng1314.domain.PostBean">
SELECT `post_date`,`ID` FROM `wp_posts`
WHERE
`post_type`='post'
AND
`post_status`='publish'
ORDER BY `post_date` DESC
</select>
</mapper>
```

我们上面唯一需要注意的就是我们的文章查询的时候，必须指定`post_type`='post'和`post_status`='publish'，这样我们首页展示的文章列表就是公开的文章。

每次按照套路来，大家都会知道我这边Dao层完成后，就应该进行Service层的操作，那么我们看下这里我们的Service层是怎么回事。

```
import cn.acheng1314.domain.DateCountBean;
import cn.acheng1314.domain.PostBean;
import cn.acheng1314.service.BaseService;

import java.util.List;

/**
* Created by 程 on 2016/11/27.
*/
public interface PostService extends BaseService<PostBean> {
@Override
void add(PostBean postBean) throws Exception;

@Override
List<PostBean> findAll(int pageNum, int pageSize);

List<PostBean> findAllPublish(int pageNum, int pageSize);

/**
* 获取总条数
* @return  获取总条数
*/
int getAllCount();

/**
* 获取热点文章
* @return
*/
List<PostBean> findAllNew();

/**
* 获取所有文章的日期归档
* @return  返回归档信息
*/
List<DateCountBean> getAllPostDateCount();
}


import cn.acheng1314.dao.PostDao;
import cn.acheng1314.domain.DateCountBean;
import cn.acheng1314.domain.PostBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

/**
* Created by 程 on 2016/11/27.
*/
@Service("postService")
public class PostServiceImpl implements PostService {

@Autowired
private PostDao dao;

@Override
public void add(PostBean postBean) throws Exception {

}

@Override
public List<PostBean> findAll(int pageNum, int pageSize) {
return null;
}

@Override
public List<PostBean> findAllPublish(int pageNum, int pageSize) {
//因为数据库内容是从第一条出的数据，所以我们查询的 起始位置 = 页码 * 条数 + 1；
pageNum -= 1;
return dao.findAllPublish(pageNum * pageSize + 1, pageSize);
}

@Override
public int getAllCount() {
return dao.getAllCount();
}

@Override
public List<PostBean> findAllNew() {
return dao.findAllNew();
}

@Override
public List<DateCountBean> getAllPostDateCount() {
/*
* 这里存入的json数据为：
*  [ {"date": "2015-11-16", "idList": [ "75", "73"] } ]
*  解释一下：日期归档本身应该是个下拉列表。下拉列表中的某个item包含了这个日期有：文章数量，文章ID
*/
List<PostBean> tmpList = new ArrayList<>(); //创建日期归档的数据集合
if (null != dao.getAllPostDateCount()
&& !dao.getAllPostDateCount().isEmpty()) {  //从dao层获取的文章日期和ID的集合不为空
tmpList.addAll(dao.getAllPostDateCount());  //先将获取的数据存入缓存变量中，避免多次读取数据库
List<DateCountBean> myDateCount = new ArrayList<>();    //创建一个日期归档的集合格式和上面所诉的json数据格式相同
//也就是外层是一个集合，里面是多个对象
for (PostBean tmpBean : tmpList) {  //遍历获取文章信息数据
DateCountBean dateCountBean = new DateCountBean();  //创建文章信息缓存的对象
if (!myDateCount.isEmpty() &&
DateFormat.getDateInstance().format(tmpBean.getPostDate().getTime()).equals(myDateCount.get(myDateCount.size() - 1).getDate())) {
//上一个日期和当前日期的相同，则只需存入ID
myDateCount.get(myDateCount.size() - 1).getIdList().add(tmpBean.getId());
} else {    //集合为或者上一条的日期和当前的日期不相同，添加一条数据
//把文章缓存信息添加到集合中
dateCountBean.setDate(DateFormat.getDateInstance().format(tmpBean.getPostDate().getTime()));    //日期格式化，把date格式化为String，也就是2015-09-28 00:01:07 ==> 2015-09-28
List<String> idList = new ArrayList<>();
idList.add(tmpBean.getId());
dateCountBean.setIdList(idList);
myDateCount.add(dateCountBean);
}
}
return myDateCount;
} else return null;
}
}

```

在上面我这里在一个Service层的一个接口写这么多代码？对的，没错，我们强调的是Service层用来做数据驱动，那么我们需要在Service层完成一些基本数据的组织。所以说我们最后首页的数据如下：

```
{
"code": 1,
"msg": "success",
"data": {
"date": [
{
"date": "2016-5-19",
"idList": [
"192",
"191"
]
},
{
"date": "2016-3-30",
"idList": [
"187"
]
}
],
"posts": [
{
"id": "282",
"postDate": "Nov 16, 2016 12:51:13 AM",
"postContent": "多角色控制思路整理</h3>\r\n关于多角色控制，起始用户角色按照用户职能分工，一般来说思路如下",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"
},
{
"id": "278",
"postDate": "Nov 9, 2016 8:46:18 PM",
"postContent": "其实分页列表也没什么，重点在于做出<strong>列表局部刷新，减少页面请求</strong>。\r\n\r\n我们先要新建一个页面用来显示列表，由于我们的后台网页结构基本已经固定，所以我们在后台主页那边设定一个访问入口，然后链接上我们的网页。这里我把左边的一个菜单改成了列表，具体效果如图：",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（七）"
}
],
"newPosts": [
{
"id": "282",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"
},
{
"id": "278",
"postTitle": "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（七）"
},
{
"id": "192",
"postTitle": "正如雨下"
}
],
"totalNum": 19,
"pageNum": 1,
"pageSize": 10
}
}
```

**由于后台数据存储的是富文本，所以我们能看到有很多网页标签。**

光是这样说明也是没有多少实际意义的，我们仍然需要晒一晒具体的Controller的方法的代码，如下：

```
@RequestMapping("/main")
public ModelAndView frontMain() {
ModelAndView view = new ModelAndView("frontMain");
view.addObject("homeJson", getHomeJson(null));  //把首页需要的json数据直接扔到view里面，在下面的js代码中可以看到如何使用
return view;
}

@RequestMapping(value = "/home"
, produces = "application/json; charset=utf-8")
@ResponseBody
public Object getHomeJson(User user) {
if (null != user) {
//埋点，AOP日志记录
}
HomeBean homeBean = new HomeBean(); //首页内容
HomeBean.DataBean dataBean = new HomeBean.DataBean();   //首页下面的Data内容对象
try {
int toalNum; //总页码

toalNum = postService.getAllCount();    //先把总条数赋值给总页数，作为缓存变量，减少下面算法的查找次数

toalNum = toalNum % 10 > 0 ? toalNum / 10 + 1 : toalNum / 10;     //在每页固定条数下能不能分页完成，有余则加一页码

List<PostBean> postsData = postService.findAllPublish(1, 10);   //首页下面的文章内容
List<PostBean> newData = postService.findAllNew();   //首页下面的文章内容
if (null == postsData || postsData.isEmpty()) {
dataBean.setPosts(null);
} else {
dataBean.setPosts(postsData);   //首页文章列表信息设定
}
if (null == newData || newData.isEmpty()) {
dataBean.setNewPosts(null);
} else {
dataBean.setNewPosts(newData);   //首页文章列表信息设定
}
List<DateCountBean> allPostDateCount = postService.getAllPostDateCount();
if (null != allPostDateCount && !allPostDateCount.isEmpty()) {
dataBean.setDate(allPostDateCount);
} else {
dataBean.setDate(null);
}
dataBean.setPageNum(1);
dataBean.setPageSize(10);
dataBean.setTotalNum(toalNum);
homeBean.setData(dataBean);
homeBean.setCode(ResponseObj.OK);
homeBean.setMsg(ResponseList.OK_STR);
return new GsonUtils().toJson(homeBean);
} catch (Exception e) {
e.printStackTrace();
//查询失败
homeBean.setCode(ResponseObj.FAILED);
homeBean.setMsg(ResponseList.FAILED_STR);
return new GsonUtils().toJson(homeBean);
}
}

```

注意看我上面代码的Try-Catch处理，我这里基本目标是保证程序功能正常，不会因为我这边的异常而产生其他错误信息。

那我们都看到了首页上面的一些数据，那么我们现在是不是需要查看前端页面的完成呢？此处不必惊慌，前端页面的完成，我们是不会少的，而且这一期完成后的代码，我也一样会同步到github。

现在我们需要的是把前台页面列表加载出来并且实现局部刷新。so，我们需要先获取到前台页面，具体代码省略，我们展示核心代码：

```
<!--从modelAndView的名为“homeJson”的Object中获取数据，并且转换位json的字符串，然后再转换位json对象-->
var homeJsonStr = JSON.stringify(${homeJson});
var homeJsonObj = JSON.parse(homeJsonStr);
var pageNum = homeJsonObj.data.pageNum; //获取当前页码
var pageSize = homeJsonObj.data.pageSize;   //获取页面长度
var totalNum = homeJsonObj.data.totalNum;   //获取总得页码
if (homeJsonObj.code == 1) {    //获取数据成功
pagefn = doT.template($("#pagetmpl").html());   //初始化列表模板
updateList(homeJsonObj.data.posts);   //更新数据
} else {
alert("获取数据失败！请联系管理员");
}

function updateList(data) {
$("#pagetmpl").empty(); //清空模板数据
$("#blog-table-list").html(pagefn(data));   //加入数据到模板
}

function goToNextPage() {
pageNum = parseInt(pageNum) + 1;
$.ajax({
type: "POST",
url: '<c:url value="/front/findPublishPost"/>',
data: {pageNum: pageNum, pageSize: pageSize},
dataType: 'json', //当这里指定为json的时候，获取到了数据后会自己解析的，只需要 返回值.字段名称 就能使用了
cache: false,
success: function (data) {
if (data.code == 1) {
updateList(data.data);
pageNum = data.pageNum;
$("#log-controller-now").html(pageNum);
}
}
});
}

function goToLastPage() {
pageNum = parseInt(pageNum) - 1;
$.ajax({
type: "POST",
url: '<c:url value="/front/findPublishPost"/>',
data: {pageNum: pageNum, pageSize: pageSize},
dataType: 'json', //当这里指定为json的时候，获取到了数据后会自己解析的，只需要 返回值.字段名称 就能使用了
cache: false,
success: function (data) {
if (data.code == 1) {
updateList(data.data);
pageNum = data.pageNum;
$("#log-controller-now").html(pageNum);
}
}
});
}
```

当然上面的代码，必须有jquery、doT.min.js和json2.js才能运行。

最后出来的整体效果，也就和上一章的截图类似，具体图片就不再截图了。稍后上传本章的代码到[github](https://github.com/pc859107393/SpringMvcMybatis)
