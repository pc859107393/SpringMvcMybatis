又是新的一期了，这一期我们把前面文章于都相关的基本地方都完成吧。

每次新写一章都会有很多话说，到头来觉得这些话好像无关痛痒，毕竟我们做技术的只需要技术足够，除非不只做技术才会想更多。

说实话，最近经历了很多事情，只想说：时间才是最长的告白。

项目github地址：https://github.com/pc859107393/SpringMvcMybatis

我的简书首页是：http://www.jianshu.com/users/86b79c50cfb3/latest_articles

上一期是：[[手把手教程][第二季]java 后端博客系统文章系统——No2](https://gold.xitu.io/post/584dacd3a22b9d0058e22a17)

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

- 文章阅读前端页面全部完成
- 根据页面框架进行解耦
    - 页面附属信息
    - 文章信息

#### 文章系统前端页面

文章系统作为我们博客系统中重要的一环，我们需要的不仅仅是文章系统，更多的是可以理解成一个自媒体平台，我们的核心价值通过这个体现出来了，才能把其他的东西做好。

上次我们的文章中可以看到前端页面的一些东西，主要是：
- 文章列表（文章内容）
- 文章分类导航
- 标签聚合导航
- 站点基本信息
- 等···

具体显示信息如图所示：

![我的博客第四章文章系统-首页结构](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第四章文章系统-首页结构.png)

从上面的截图中我们可以看到我们的页面大概结构，页面头、页面尾和页面中间的内容，那么出于便利考虑我们需要把头尾单独抽取出来存放，页面其他的内容我们需要根据需要处理。现在我们先不考虑那么多，我们只是基于程序合理建设的角度来说，我们需要把页面上面动态变化的信息都独立做成接口来供外部调用，然后一般不怎么变化的东西我们就直接固化到页面中，即是说：
- 中间的列表我们采用分页加载，全部动态从接口获取
- 上面的一些其他变化的信息我们从请求的时候就附加给它

所以，我们需要把前面的首页接口重写一下。

首先，我们给首页获取数据的接口打上过时的标记。
```
/**
     * 获取主页的json数据，按照道理讲这里应该根据页面结构拆分组合的
     *
     * @param user 用户信息
     * @return 返回首页json
     */
    @RequestMapping(value = "/home"
            , produces = "application/json; charset=utf-8")
    @ResponseBody
    @Deprecated
    public Object getHomeJson(User user) {
        //此处代码省略
    }
```

既然我们已经把首页的设置为过时，那么新的接口必须对照着做一个，那么我们需要怎么处理呢？按照前面的思路来讲，我们现在需要根据需求将我们页面信息拆分成多个接口，首先需要把左边我们圈出来的部分整合到一起，那么我们需要先把个人信息分类导航和标签聚合这几个独立出来，所以得我们直接上代码。

```
    /**
     * 返回主页面
     *
     * @return
     */
    @RequestMapping("/main")
    public ModelAndView frontMain(HttpServletRequest request) throws Exception {
        ModelAndView view = new ModelAndView("frontMain");
        //把数据存入前端，避免前端再次发起网络请求
        view.addObject("framJson", getFramJson());
        view.addObject("postsJson", findPublishPost(request, 1, 10));
        return view;
    }

    /**
     * 页面框架的变化信息
     * 1、个人信息
     * 2、最新热点随机文章信息
     * 3、标签信息
     *
     * @return
     */
    @RequestMapping(value = "/getFramJson"
            , produces = "application/json; charset=utf-8")
    @ResponseBody
    public Object getFramJson() {
        HomeBean homeBean = new HomeBean(); //首页内容
        HomeBean.DataBean dataBean = new HomeBean.DataBean();   //首页下面的Data内容对象
        try {
            int toalNum = postService.getAllCount();    //先把总条数赋值给总页数，作为缓存变量，减少下面算法的查找次数
            toalNum = toalNum % 10 > 0 ? toalNum / 10 + 1 : toalNum / 10;     //在每页固定条数下能不能分页完成，有余则加一页码

            List<PostBean> newData = postService.findAllNew();
            if (null == newData || newData.isEmpty()) {
                //页面上面推荐的文章信息不为空
                dataBean.setNewPosts(null);
                dataBean.setHotPosts(null);
                dataBean.setRandomPosts(null);
            } else {
                //首页文章列表信息设定
                dataBean.setNewPosts(newData);
                dataBean.setHotPosts(newData);
                dataBean.setRandomPosts(newData);
            }
            //日期归档
            List<DateCountBean> allPostDateCount = postService.getAllPostDateCount();
            if (null != allPostDateCount && !allPostDateCount.isEmpty()) {
                dataBean.setDate(allPostDateCount);
            } else {
                dataBean.setDate(null);
            }
            //设置作者信息
            List<HashMap<String, String>> userMeta = userService.getUserMeta(1);
            dataBean.setAuthor(userMeta);

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

说实话感觉上面没啥解释的，说白了就是将数据按照一定的结构组合起来，具体展示的json如何，我们可以直接在下面看：

```
{
  "msg" : "success",
  "data" : {
    "author" : [
      {
        "meta_key" : "nickname",
        "meta_value" : "雨下一整夜"
      },
      {
        "meta_key" : "description",
        "meta_value" : "我想在最好的时候遇见最美的你。那是最美。"
      },
      {
        "meta_key" : "managenav-menuscolumnshidden",
        "meta_value" : "a:2:{i:0;s:11:\"css-classes\";i:1;s:11:\"description\";}"
      }
    ],
    "pageNum" : 0,
    "pageSize" : 0,
    "newPosts" : [
      {
        "id" : "286",
        "postTitle" : "Android-MVP架构"
      },
      {
        "id" : "282",
        "postTitle" : "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"
      }
    ],
    "date" : [
      {
        "date" : "2015-11-13",
        "idList" : [
          "71",
          "69",
          "67"
        ]
      },
      {
        "date" : "2015-10-15",
        "idList" : [
          "48",
          "46"
        ]
      }
    ],
    "randomPosts" : [
      {
        "id" : "286",
        "postTitle" : "Android-MVP架构"
      },
      {
        "id" : "282",
        "postTitle" : "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"
      }
    ],
    "hotPosts" : [
      {
        "id" : "286",
        "postTitle" : "Android-MVP架构"
      },
      {
        "id" : "282",
        "postTitle" : "[手把手教程][JavaWeb]优雅的SpringMvc+Mybatis应用（八）"
      }
    ],
    "totalNum" : 0
  },
  "code" : 1
}
```

通过上面我们组织的json，我们可以很清晰明了的看到我们的数据结构是根据页面结构来组合的，所以我们需要数据的时候对应着取值就可以解决问题。

说了这么多后端的接口，我们现在需要拿数据去前台展示，所以我们需要在前端获取数据。前台数据展示还是使用doT.min.js来展示，代码如下：

```

    var framJsonStr = JSON.stringify(${framJson});
    var framJsonObj = JSON.parse(framJsonStr);
    var postsJsonStr = JSON.stringify(${postsJson});
    var postsJsonObj = JSON.parse(postsJsonStr);
    var pageNum = postsJsonObj.pageNum;
    var pageSize = postsJsonObj.pageSize;
    var totalNum = postsJsonObj.totalNum;
    var authorDes = "<p class=\"text-center\">" + framJsonObj.data.author[0].meta_value + "<br/>" + framJsonObj.data.author[1].meta_value + "</p>";
    document.getElementById("authorDescription").innerHTML = authorDes;

    if (framJsonObj.code == 1) {
        pagefn = doT.template($("#listHot").html());   //初始化列表模板
        updateHotList(framJsonObj.data.hotPosts);   //更新数据
        updateNewList(framJsonObj.data.newPosts);   //更新数据
        updateRandList(framJsonObj.data.randomPosts);   //更新数据
    }

    function updateHotList(data) {
        $("#listHot").empty(); //清空模板数据
        $("#hotList").html(pagefn(data));   //加入数据到模板
    }

    function updateNewList(data) {
        $("#listNew").empty(); //清空模板数据
        $("#newList").html(pagefn(data));   //加入数据到模板
    }

    function updateRandList(data) {
        $("#listRand").empty(); //清空模板数据
        $("#randList").html(pagefn(data));   //加入数据到模板
    }

    //开始加载列表数据
    if (postsJsonObj.code == 1) {
        pagefn = doT.template($("#pagetmpl").html());   //初始化列表模板
        updateList(postsJsonObj.data);   //更新数据
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

上面我们可以看到pagefn用了几次，这个是doT的关键词，意思是设置模板。

```
<!--顯示文章列表-->
<div id="blog-table-list">
	<script id="pagetmpl" type="text/x-dot-template">
		{{ for(var i=0; i
		< it.length ; i++){ }} <a href="<c:url value=" /front/post/{{=it[i].id}} "/>">
			<div style="width:100%;height: 2px"></div>
			<%--<div class="kratos-entry-thumb clearfix">--%>
			<%--<img class="kratos-entry-thumb"--%>
			<%--src="<c:url value="/static/images/kratos-update.png"/>">--%>
			<%--</div>--%>
			<div class="kratos-post-inner">
				<header class="kratos-entry-header clearfix">
					<h1 class="kratos-entry-title">{{=it[i].postTitle}}</h1>
				</header>
				<div class="kratos-entry-content clearfix">
					<p>{{=it[i].postTitle}}</p>
				</div>
			</div>
			<div style="background: #CCCCCC;width:100%;height: 2px"></div>
			</a>
			{{ } }}
	</script>
</div>

<!-- 展示文章導航 -->
<div class="tab-content">
	<div class="tab-pane fade" id="newest">
		<ul class="list-group" id="newList">
			<script id="listNew" type="text/x-dot-template">
				{{ for(var i=0; i
				< it.length ; i++){ }} 
				<a class="list-group-item visible-lg"
				    title="{{=it[i].postTitle}}" 
				    href="<c:url value=" /front/post/{{=it[i].id}} "/>" 
				    rel="bookmark">
				    <i class="fa fa-book"> {{=it[i].postTitle}}</i> 
				</a>
					{{ } }}</script>
		</ul>
	</div>
	<div class="tab-pane fade  in active" id="hot">
		<ul class="list-group" id="hotList">
			<script id="listHot" type="text/x-dot-template">
				{{ for(var i=0; i
				< it.length ; i++){ }} 
				<a class="list-group-item visible-lg"
				    title="{{=it[i].postTitle}}" 
				    href="<c:url value=" /front/post/{{=it[i].id}} "/>" 
				    rel="bookmark">
				<i class="fa fa-book"> {{=it[i].postTitle}}</i> 
				</a>
					{{ } }}</script>
		</ul>
	</div>
	<div class="tab-pane fade" id="rand">
		<ul class="list-group" id="randList">
			<script id="listRand" type="text/x-dot-template">
				{{ for(var i=0; i
				< it.length ; i++){ }} 
				<a class="list-group-item visible-lg"
				    title="{{=it[i].postTitle}}" 
				    href="<c:url value=" /front/post/{{=it[i].id}} "/>" 
				    rel="bookmark">
				    <i class="fa fa-book"> {{=it[i].postTitle}}</i> 
				</a>
					{{ } }}</script>
		</ul>
	</div>
</div>Ï
```

其實在上面的代碼中我們可以看到doT模板和其他的都差不多，無非就是按照固定的格式組裝数据，反正就是网页怎么写的，然后把格式套上，然后按照格式输出就行了。

做到这里后，我们就能看到做出的结果是什么样子的了。效果图暂时不上，大家后面自行下载项目运行就知道了。

既然这里做了，那么我们势必要做一下项目的文章详情。文章详情我们应该怎么办呢？ 我们需要**通过关键数据去查找到具体的文章信息**。

我们可以看到上面的json数据中包含一个id的字段，然后我们对照数据库会看到id和数据库的id也是对应的。所以我们需要用接口实现通过ID查找数据库对应的文章信息。思路有了，那么代码实现就是很容易的，直接代码如下：
```
    /**
     * RESTful风格的文章页面
     * @RequestMapping(path = "/post/{postId}", method = RequestMethod.GET)
     * 通过上面的语句配置访问路径/post/后面指定是文档ID，在下面的方法参数中配置注解@PathVariable可以自动赋值，然后获取数据。
     * @param postId 文章ID
     * @return 返回文章页面
     */
    @RequestMapping(path = "/post/{postId}", method = RequestMethod.GET)
    public ModelAndView getPostView(@PathVariable int postId) {
        ModelAndView resultView = new ModelAndView("frontPost");
        resultView.addObject("framJson", getFramJson());
        resultView.addObject("postJson", getPostById(postId));
        return resultView;
    }
    
    
    /**
     * 根据文章ID获取文章内容
     *
     * @param postId 文章ID
     * @return 返回文章ID对应的文章内容
     */
    @RequestMapping(value = "/getPost"
            , produces = "application/json; charset=utf-8")
    @ResponseBody
    public Object getPostById(int postId) {
        ResponseObj<Object> responseObj = new ResponseObj<>();
        try {
            PostBean postBean = postService.findPostById(postId);
            if (null == postBean) {
                responseObj.setCode(ResponseObj.EMPUTY);
                responseObj.setMsg(ResponseObj.EMPUTY_STR);
            } else {
                responseObj.setCode(ResponseObj.OK);
                responseObj.setMsg(ResponseObj.OK_STR);
                responseObj.setData(postBean);
            }
            return new GsonUtils().toJson(responseObj);
        } catch (Exception e) {
            e.printStackTrace();
            responseObj.setCode(ResponseObj.FAILED);
            responseObj.setMsg(ResponseObj.FAILED_STR);
            return new GsonUtils().toJson(responseObj);
        }
    }
    

```

上面的代码一个是RESTful风格请求地址的文章页面，一个是api接口访问地址。下面分别是Service和Dao层的代码。

```
    /**
    *这是Service
    */
    @Override
    public PostBean findPostById(Serializable postId) {
        return dao.findOneById(postId);
    }
    
    /**
    *这是Dao
    */
    @Override
    PostBean findOneById(Serializable postId);
    
    <!-- 这是mapper -->
    <select id="findOneById" resultType="cn.acheng1314.domain.PostBean">
        SELECT
          `ID`,`post_title`,`post_date`,`post_content`
        FROM
          `wp_posts`
        WHERE
        `ID`=#{postId}
        AND
        `post_status`='publish'

    </select>
```

最后页面的样子如下，页面展示代码也就是获取到内容直接输出，具体代码就不贴了，影响文章篇幅。大家可以直接到我的GitHub上面下载。

![我的博客第四章文章系统-文章页面](http://acheng1314.cn/wp-content/uploads/2016/12/我的博客第四章文章系统-文章页面.png)

到目前为止，我们前端的页面基本完成后面是需要把页面头和页面尾独立出来，然后我们这种部分方便定制。

那么目前前端页面相关的东西基本完成，这一期也结束了，后面我们就是配置系统后端了。加油，有兴趣额的一起来吧。