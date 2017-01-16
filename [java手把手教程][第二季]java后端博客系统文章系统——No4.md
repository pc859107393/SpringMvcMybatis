转眼间第二季来到了第五章，也是我们博客系统的第四章。前段时间因为个人私事较多，项目停更了两期，但是这都不是问题，我们继续接着走下去。毕竟承诺的事情就得完成。

这一期我们的目标是完成后端博客系统的博客发布功能。

按照我们前面的设定，我们的后端博客系统需要完成最简单的博文发布，我们也得有后台管理界面，同时需要将用户权限这些都附带上，但是由于时间关系，我们后端默认账户就是管理员吧，毕竟这一期的重点是实现博客的发布。

---

> 博文发布系统

我们需要发布博文，那么后端必不可少的是登录和发布系统，至于其他的我们可以先缓一缓，毕竟我也没想好后端页面怎么设计，嘿嘿。

前面我看了一下，确实是完美兼容WordPress还是有很多难度，毕竟很多技术细节我们并不知道，不过，至少说目前我们已经兼容了博客文章，剩下的只需要一点点的适配就能大概完成任务。

不多说了，我们先完成后端登录功能。

#### 后端登录

后端登录，我们不可能说一味的兼容WordPress，还有一些技术上面的设计理念可能也不是那么那啥，所以我们需要拿出一些自己的玩意。首先还是老规矩，从Dao→Service→Controller。

- Dao按照老规矩就是对数据库的操作，所以我们只需要写上接口和mapper就行了。
- Service层还是一样进行单元数据操作。
- Controller是web应用的入口地点。
有了上面的这些我们只需要进行一个登录验证，也就是前面说过的密码规则验证，不过具体代码如下：

```java
@RequestMapping(value = "/login"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = RequestMethod.POST)
    @ApiOperation(
            value = "用户登录"
            , notes = "用户登录的接口，输入用户名和密码进行登录"
            , response = User.class)
    @ResponseBody
    public Object userLogin(HttpServletRequest request,
                            @ApiParam(value = "用户名不能为空，否则不允许登录"
                                    , required = true) @RequestParam("userLogin") String userLogin,
                            @ApiParam(value = "用户密码不能为空且必须为16位小写MD5，否则不允许登录"
                                    , required = true) @RequestParam("userPass") String userPass) {
        ResponseObj<User> responseObj = new ResponseObj<>();
        User user;
        if (PublicUtil.isJsonRequest(request)) {    //确认是否是post的json提交
            try {
                user = new GsonUtils().jsonRequest2Bean(request.getInputStream(), User.class);  //转换为指定类型的对象
                userLogin = user.getUserLogin();
                userPass = user.getUserPass();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        if (StringUtils.isEmpty(userLogin) || StringUtils.isEmpty(userPass)) {
            responseObj.setCode(ResponseObj.FAILED);
            responseObj.setMsg("用户名和密码不能为空！请检查！");
            return new GsonUtils().toJson(responseObj);
        }

        user = userService.findOneById(userLogin);

        if (null == user) {
            responseObj.setCode(ResponseObj.EMPUTY);
            responseObj.setMsg("用户不存在！请检查!");
            return new GsonUtils().toJson(responseObj);
        }

        userPass = userPass.toLowerCase();  //将大写md5转换为小写md5

        if (userPass.length() > 16 && userPass.length() == 32) {    //32位小写转换为16位小写
            userPass = userPass.substring(8, 24).toLowerCase();
        } else if (userPass.length() > 32) {
            responseObj.setCode(ResponseObj.FAILED);
            responseObj.setMsg("密码不规范！请检查！");
            return new GsonUtils().toJson(responseObj);
        }

        String encryptPassword = EncryptUtils.encryptPassword(userPass, user.getUserActivationKey());

        if (!encryptPassword.equals(user.getUserPass())) {
            responseObj.setCode(ResponseObj.FAILED);
            responseObj.setMsg("用户名和密码不匹配！请检查！");
            return new GsonUtils().toJson(responseObj);
        }

        user.setUserPass(request.getSession().getId()); //将sessionId放入用户信息中
        user.setUserActivationKey("");  //将用户注册的salt清空
        user.setUserUrl("/user/endSupport/index");

        responseObj.setData(user);
        responseObj.setCode(ResponseObj.OK);
        responseObj.setMsg("登录成功");

        return new GsonUtils().toJson(responseObj);
    }
```
虽然说很多东西我们在前端或者是客户端已经做了限制，但是为了防止别人搞事情我们还是需要这样做才行。

#### Spring-Fox，Api测试页面

什么是Spring-Fox呢？Springfox的前身是swagger-springmvc，是一个开源的API doc框架，可以将我们的Controller的方法以文档的形式展现。

为什么我们要大费周章的做这些呢？
- 它可以帮助我们归类web访问入口
- 它可以整理接口
- 它可以···

确实语言描述是我的弱点，不过呢，我这个理工直男癌就需要直截了当的说出来，没时间解释，直接上图。

![我的博客第五章-API_doc框架](http://acheng1314.cn/wp-content/uploads/2017/01/我的博客第五章-API_doc框架.png)

正如上面的截图所示，我们首先应该找到对应的spring-fox的说明文档，然后仔细一看网上分为两个版本，一个是开源中国的引入说明，一个是[Spring-Fox官方的使用说明](http://springfox.github.io/springfox/docs/current/)，那么肯定选择[官方](http://springfox.github.io/springfox/docs/current/)的。

按照官方文档，我们简单总结一下：
- 版本选择（Release或者Snapshot，推荐使用Release）
- 依赖引入（maven或者gradle）
- swagger设置
- 重要细节（Spring-Fox官方文档中没有指明！！！）

按照官方文档说明的是，他们的demo是在SpringBoot下面实现的，现在我们需要单一的拆分出来，可以看成我们的项目就是Spring-Mvc，所以一些细节需要改变，当然当中一个很重要的细节官方文档也是没有指明，所以看官们且看我细细道来。

> 引入依赖资源

首先我们引入引来资源，通读全文最基本的依赖是：springfox-swagger、springfox-swagger-ui，所以我们直接老规矩，在gradle的配置文件中引入依赖：

```
compile "io.springfox:springfox-swagger2:2.6.1"
compile 'io.springfox:springfox-swagger-ui:2.6.1'
```

在引入上面的基本依赖后，我们查看他们关联的依赖可以发现这些依赖里面还有引入jackson，这个时候我们可以选择提升我们的Jackson或者不管他们也行，不过我还是吧Jackson的版本提升了：

```
compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.8.4'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-annotations', version: '2.8.4'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: '2.8.4'
```
> swagger设置

根据官方文档我们可以看到有一个swagger设置需要先引入后，才能让我们设定的东西生效，所以我们先引入设置：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * Created by mac on 2017/1/8.
 */
@Configuration  //说明这个是spring的设置
@EnableWebMvc   //不是SpringBoot需要引入这个
@EnableSwagger2 //开启Swagger2
@ComponentScan("cn.acheng1314.controller")  //指定被扫描Controller的位置
public class Swagger2SpringMvc extends WebMvcConfigurerAdapter {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)  //Docket，Springfox的私有API设置初始化为Swagger2
                .select()
//                .apis(RequestHandlerSelectors.basePackage("cn.acheng1314.controller"))    //此处设置扫描包和上面设置的扫描包一样效果
                .paths(PathSelectors.any())
                .build()
                .apiInfo(new ApiInfoBuilder()  //设置API文档的主体说明
                        .title("博客Apis")
                        .description("雨下一整夜的博客APIs")
                        .version("1.01")
                        .termsOfServiceUrl("http://acheng1314.cn/")
                        .build())
//                .pathMapping("/")
//                .genericModelSubstitutes(ResponseEntity.class)
//                .alternateTypeRules(
//                        newRule(typeResolver.resolve(DeferredResult.class,
//                                typeResolver.resolve(ResponseEntity.class, WildcardType.class)),
//                                typeResolver.resolve(WildcardType.class)))
                ;
    }

}

```

设置完成上面的东西后，我们需要干什么呢？  上面我们很明显的看到我们 **@configuration是一个spring框架的注解** ，顾名思义肯定就应该是一个spring的设置。同时我们可以在idea的编辑器中看到类名有一层淡黄色的标记，然后选中类名按下代码提示(Alt+Enter)会有提示告诉我们这个设置没有使用，然后自动完成后会给我们自动添加到Spring的ApplicationContext中作为CodeContext使用。

当然，上面的是懒人做事这样做的后果会是导致apiInfo的设置不能生效。

那么正常一点的应该是怎么做呢？按照Spring的思想来说，我们需要在Spring的设置文件中直接引入bean。所以我们在spring-web.xml中插入对应的bean，如下：

```xml
<!--SpringFox设置引入-->
<bean id="SpringFox" class="cn.acheng1314.Swagger2SpringMvc"/>
```
通过这样的在spring的配置文件中设置后，我们感觉应该是能用的，所以我们可以先跑起来看看。

> 重要细节，我有特殊的装X技巧

按照官方文档我们完全设定了后，我们可以看到就算我们在代码中引入了配置后，一样的在里面不能看到网页的接口列表（只看得到上面的标题栏，下面的是空白），然后我们仔细的查看网页的请求会发现
```
http://localhost:8080/swagger-resources/configuration/ui
```
这个请求是404。说实话这个错误困扰了我很久，同时这个问题前面我处理的时候还是有一系列的综合问题，后面整个工程师重建后完成的。

但是现在这个问题简单直接找到问题所在了，那就是在我们spring的设置中，关于web的设置我们都是在spring-web.xml中完成的，同时里面的东西我们需要改动一下才能适应现在的需要，如下：

```
<!-- 4.扫描web相关的bean -->
    <context:component-scan base-package="cn.acheng1314.controller">
        <!-- 制定扫包规则 ,只扫描使用@Controller注解的JAVA类 -->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

<!--上面的是原来的设置，现在我们新的代码如下：-->
<!-- 4.扫描web相关的bean -->
    <context:component-scan base-package="cn.acheng1314.controller,springfox">
        <!-- 制定扫包规则 ,只扫描使用@Controller注解的JAVA类 -->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```
也就是说，我们除了要在Spring的配置文件中引入bean来初始化swagger相关的东西以外，我们还需要在web扫描那里添加springfox的扫描。所以我们spring-fox的设置相关的完成了。

> Spring-Fox的使用

从前面的学习中我们可以明白我们所有的网络请求都是在controller中来实现的，所以我们这里需要通过对controller做适当的修改才能实现SpringFox的使用。具体的直接仍代码上来，大家详细的看看就行了，不需要什么深入钻研。

```java

import cn.acheng1314.domain.*;
import cn.acheng1314.domain.Response.HomeBean;
import cn.acheng1314.service.postService.PostService;
import cn.acheng1314.service.userService.UserService;
import cn.acheng1314.utils.GsonUtils;
import cn.acheng1314.utils.PublicUtil;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;
import java.util.List;

import static org.springframework.http.MediaType.APPLICATION_JSON_UTF8_VALUE;

/**
 * 前端页面的控制器，博客系统前端页面相对较少，所以都扔在这里了
 * Created by 程 on 2016/11/25.
 */
@Controller
@RequestMapping("/front")
@Api(value = "/front", description = "前台页面")
public class FrontWebController {

    @Autowired
    private PostService postService;
    @Autowired
    private UserService userService;

    /**
     * 返回主页面
     *
     * @return
     */
    @RequestMapping(value = "/main", method = RequestMethod.GET)
    @ApiOperation(
            value = "打开首页界面"
            , notes = "首页web界面，js模板加载网页数据")
    public ModelAndView frontMain(HttpServletRequest request) throws Exception {
        ModelAndView view = new ModelAndView("frontMain");
        view.addObject("framJson", getFramJson());
        view.addObject("postsJson", findPublishPost(request, 1, 10));
        return view;
    }

    /**
     * 获取文章分页列表
     *
     * @param request  用户请求
     * @param pageNum  当前页码
     * @param pageSize 每一页的长度
     * @return
     * @throws Exception
     */
    @RequestMapping(value = "/findPublishPost"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ResponseBody
    @ApiOperation(  //接口描述
            value = "获取文章分页列表"  //功能简介
            , notes = "返回文章列表，分页加载" //接口功能说明
            , response = PostBean.class,    //返回数据的值说明
            responseContainer = "List") //返回数据类型说明
    public Object findPublishPost(HttpServletRequest request,
                                  @ApiParam(value = "当前页码,默认不能为空，否则为1",
                                          required = true,    //参数是否必传
                                          defaultValue = "1" //参数默认值为1
//            ,allowableValues = "available,pending,sold"   //暂时未知，待查阅文章
//            ,allowMultiple = true    //是否允许allowMultiple类型的参数
                                  ) @RequestParam("pageNum")
                                          int pageNum,
                                  @ApiParam(value = "每一页的长度,默认不能为空，否则列表条目数量为10",
                                          required = true,    //参数是否必传
                                          defaultValue = "10" //参数默认值为1
//            ,allowableValues = "available,pending,sold"   //暂时未知，待查阅文章
//            ,allowMultiple = true    //是否允许allowMultiple类型的参数
                                  ) @RequestParam("pageSize")
                                          int pageSize) throws Exception {
        PageSplit page;
        ResponseList<PostBean> list = new ResponseList<>();

        if (PublicUtil.isJsonRequest(request)) {    //确认是否是post的json提交
            page = new GsonUtils().jsonRequest2Bean(request.getInputStream(), PageSplit.class);  //转换为指定类型的对象
            pageNum = page.getPageNum();
            pageSize = page.getPageSize();
        }
        if (pageNum <= 0) {
            pageNum = 1;
        }
        if (pageSize == 0) {
            pageSize = 10;
        }

        try {
            int toalNum; //总页码
            toalNum = postService.getAllCount();    //先把总条数赋值给总页数，作为缓存变量，减少下面算法的查找次数

            toalNum = toalNum % pageSize > 0 ? toalNum / pageSize + 1 : toalNum / pageSize;     //在每页固定条数下能不能分页完成，有余则加一页码

            List<PostBean> tmp = postService.findAllPublish(pageNum, pageSize);
            if (null == tmp || tmp.size() == 0) {
                list.setCode(ResponseList.EMPUTY);
                list.setMsg(ResponseList.EMPUTY_STR);
                return new GsonUtils().toJson(list);
            }
            list.setCode(ResponseList.OK);
            list.setMsg(ResponseList.OK_STR);
            list.setPageNum(pageNum);
            list.setTotalNum(toalNum);
            list.setPageSize(pageSize);
            list.setData(tmp);
            return new GsonUtils().toJson(list);
        } catch (Exception e) {
            e.printStackTrace();
            //查询失败
            list.setCode(ResponseObj.FAILED);
            list.setMsg(ResponseList.FAILED_STR);
            return new GsonUtils().toJson(list);
        }

    }

    /**
     * 查找最近的文章
     *
     * @return
     */
    @RequestMapping(value = "/findNewPost"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ApiOperation(
            value = "获取最近文章"
            , notes = "获取最近文章的json，具体字段请参照输出的json数据"
            , response = PostBean.class)
    @ResponseBody
    public Object findNewPost() {
        ResponseObj<Object> responseObj = new ResponseObj<>();
        try {
            List<PostBean> allNew = postService.findAllNew();
            if (null == allNew || allNew.isEmpty()) {
                responseObj.setCode(ResponseObj.EMPUTY);
                responseObj.setMsg(ResponseObj.EMPUTY_STR);
            } else {
                responseObj.setCode(ResponseObj.OK);
                responseObj.setMsg(ResponseObj.OK_STR);
                responseObj.setData(allNew);
            }
            return new GsonUtils().toJson(responseObj);
        } catch (Exception e) {
            e.printStackTrace();
            responseObj.setCode(ResponseObj.FAILED);
            responseObj.setMsg(ResponseObj.FAILED_STR);
            return new GsonUtils().toJson(responseObj);
        }
    }

    /**
     * 获取热点文章信息
     *
     * @return
     */
    @RequestMapping(value = "/findHotPost"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ApiOperation(
            value = "获取最热文章"
            , notes = "获取最热文章的json，具体字段请参照输出的json数据"
            , response = PostBean.class)
    @ResponseBody
    public Object findHotPost() {
        return findNewPost();
    }

    /**
     * 获取随机文章信息
     *
     * @return 返回json
     */
    @RequestMapping(value = "/findRandomPost"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ApiOperation(
            value = "随机获取文章"
            , notes = "随机获取文章的json，具体字段请参照输出的json数据"
            , response = PostBean.class)
    @ResponseBody
    public Object findRandomPost() {
        return findNewPost();
    }

    /**
     * 获取主页的json数据，按照道理讲这里应该根据页面结构拆分组合的
     *
     * @param user 用户信息
     * @return 返回首页json
     */
    @RequestMapping(value = "/home"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ResponseBody
    @Deprecated
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
                dataBean.setHotPosts(null);
                dataBean.setRandomPosts(null);
            } else {
                dataBean.setNewPosts(newData);   //首页文章列表信息设定
                dataBean.setHotPosts(newData);
                dataBean.setRandomPosts(newData);
            }
            List<DateCountBean> allPostDateCount = postService.getAllPostDateCount();
            if (null != allPostDateCount && !allPostDateCount.isEmpty()) {
                dataBean.setDate(allPostDateCount);
            } else {
                dataBean.setDate(null);
            }
            //设置作者信息
            List<HashMap<String, String>> userMeta = userService.getUserMeta(1);
            dataBean.setAuthor(userMeta);

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

    /**
     * 页面框架的变化信息
     * 1、个人信息
     * 2、最新热点随机文章信息
     * 3、标签信息
     *
     * @return
     */
    @RequestMapping(value = "/getFramJson"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ApiOperation(
            value = "获取主题框架的json"
            , notes = "整个页面的主体框架的json数据")
    @ResponseBody
    public Object getFramJson() {
        HomeBean homeBean = new HomeBean(); //首页内容
        HomeBean.DataBean dataBean = new HomeBean.DataBean();   //首页下面的Data内容对象
        try {
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

    /**
     * 根据作者的ID获取作者的信息
     *
     * @param userId 作者ID
     * @return 返回作者的json信息
     */
    @RequestMapping(value = "/getAuthorInfo"
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ApiOperation(
            value = "获取作者信息"
            , notes = "获取作者基本信息的json，具体字段请参照输出的json数据"
            , response = PostBean.class)
    @ResponseBody
    public Object getAuthorJson(int userId) {
        ResponseObj<Object> responseObj = new ResponseObj<>();
        try {
            List<HashMap<String, String>> userMeta = userService.getUserMeta(userId);
            if (null == userMeta || userMeta.isEmpty()) {
                responseObj.setCode(ResponseObj.EMPUTY);
                responseObj.setMsg(ResponseObj.EMPUTY_STR);
            } else {
                responseObj.setCode(ResponseObj.OK);
                responseObj.setMsg(ResponseObj.OK_STR);
                responseObj.setData(userMeta);
            }
            return new GsonUtils().toJson(responseObj);
        } catch (Exception e) {
            e.printStackTrace();
            responseObj.setCode(ResponseObj.FAILED);
            responseObj.setMsg(ResponseObj.FAILED_STR);
            return new GsonUtils().toJson(responseObj);
        }
    }

    /**
     * RESTful风格的文章页面
     *
     * @param postId 文章ID
     * @return 返回文章页面
     */
    @RequestMapping(path = "/post/{postId}", method = RequestMethod.GET)
    @ApiOperation(
            value = "打开文章详情web界面"
            , notes = "文章详情web界面，js模板加载网页数据")
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
            , produces = {APPLICATION_JSON_UTF8_VALUE}
            , method = {RequestMethod.GET, RequestMethod.POST})
    @ApiOperation(
            value = "根据id获取文章json"
            , notes = "根据文章的ID获取文章的详情json"
            , response = PostBean.class)
    @ResponseBody
    public Object getPostById(
            @ApiParam(value = "文章ID", required = true)
            @RequestParam("postId")
                    int postId) {
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

}

```

至此我们的Spring-Fox简单实用已经完成，后续的操作我们在需要的地方再查找资料就行了。

#### 总结
本期项目都是简单的介绍了一些东西，主要有：
- 登录密码校验规则（MD5→SHA256）
- Spring-Fox的引入
- Spring-Fox在非springBoot中的使用
- Spring-Fox的使用

这一期本来是上一周就应该完成的，但是回家懒癌发作又拖了一周，期间女朋友还各种生病，每天也没心思写代码，很对不起大家了，后面我会更加努力，也感谢一些哥们在我博客上面的留言鼓励，谢谢大家。