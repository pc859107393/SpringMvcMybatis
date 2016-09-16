#### [手把手教程][JavaWeb]优雅的SpringMvc+Mybatis整合之路
手把手教你整合最优雅SSM框架：SpringMVC + Spring + MyBatis
- 前面网友说我为啥很久不更新博客了,我告诉他们我准备潜修.其实是我的博客被人批评是在记流水账(~一脸尴尬~).
- 再次安利一波,博客地址:[acheng1314.cn](http://acheng1314.cn/)
- 本文中的图片用了个人服务器存储,网速较慢,各位老司机耐心等待.

#### 工具
- IDE为**idea15**
- JDK环境为**1.8**
- maven版本为**maven3**

#### 目标
- 完成基本的SpringMVC + Spring + MyBatis框架整合
- 数据库使用mysql
- 加入阿里巴巴的druid数据库连接池
- 使用gson作为json解析工具
- 实现日志输出
- maven依赖的版本管理

#### 优点
```
此处省略若干字,观众们请脑补.
```

----

#### SSM框架整合配置

前面说了这么多,现在开始正式的干货.

##### 第一步: 使用idea的maven创建一个基本的web工程.
- 打开Idea在欢迎界面选择创建一个新的Project或者是(在菜单界面选择:New→Project),这是会出现一个界面如下图所示:

![maven新建WebApp项目第一步](http://acheng1314.cn/wp-content/uploads/2016/09/maven新建WebApp项目第一步.png)

- 如上图所示,我们需要勾选的地方已经使用红色框标注出来.
    - 最左边的是**maven**,是我们需要使用的项目构建工具.
    - 勾选右边上面的**Create from archetype**,我们才能在下面选择我们需要构建成什么类型的项目.
    - 接着我们选中**maven-archetype-webapp**,这时候我们的项目类型就确定为是web项目.
    - 需要注意一点,我上面图中没标注出来的**Project SDK**,这里是选择我们开发的JDK版本.

- 点击next后,如下图所示:

![maven新建WebApp项目第二步](http://acheng1314.cn/wp-content/uploads/2016/09/maven新建WebApp项目第二步.png)

- 上面图中,我们需要注意地方如下:
    - **GroupId**也就是我们常说的组织ID,也可以理解为我们**应用程序的包名**
    - **ArtifactId**是我们常说的产品名称(同一个组织下面可以有多个产品),也可以当作是我们的**当前项目名称**
    - **Version**顾名思义就是版本号
    - 最下面的红色框中,Previous==>返回上一步,Next==>下一步,Cancel==>取消,Help==>帮助
    
- 接下来,我们继续点击Next后,如下图所示:

![maven新建WebApp项目第三步](http://acheng1314.cn/wp-content/uploads/2016/09/maven新建WebApp项目第三步.png)

- 上面途中没啥好说的,圈出来部分就是我们的**Maven目录**.继续next后,如下图所示:

![maven新建WebApp项目第四步](http://acheng1314.cn/wp-content/uploads/2016/09/maven新建WebApp项目第四步.png)

- 上面选中部分,**Project name**为**项目名称**,**Project location**是项目的**存储位置**(~右边的省略号意味着可以选择位置~).
- 接下来我们**点击Finish**,我们新建基本的web项目的步骤就完成了.
- 这时候在Idea主窗口的右下角部分,我们可以看到一个滚动条在执行,说明我们的项目正在build中.右上角有一个提示框如下图所示:

![maven新建WebApp项目完成后的自动导入提示框](http://acheng1314.cn/wp-content/uploads/2016/09/maven新建WebApp项目完成后的自动导入提示框.png)

- 这个提示框大概意思是:Maven项目需要被导入.我建议勾选:**Enable Auto-Import**(~自动导入~)

此处,使用Idea创建一个**Maven依赖的基本的WebApp项目**已经完成.

----
#### 框架整合前的准备工作.
- 整理项目文件组织结构.
    - 通过观察目录结构,我们可以发现,需要的目录不齐全,我们需要手动补齐.初始结构图如下:
    
        ![maven新建WebApp项目完成后的目录结构](http://acheng1314.cn/wp-content/uploads/2016/09/maven新建WebApp项目完成后的目录结构.png)
    - 我们需要的主体结构图应如下:

        ![WebApp项目整合框架前的目录结构](http://acheng1314.cn/wp-content/uploads/2016/09/WebApp项目整合框架前的目录结构.png)
        
        #### 需要的主体结构目录解释:
        -----
        |  目录名称  |  说明  |
        | ---- | ----|
        | src | 源码、资源等文件的根目录|
        | ↓ main | 项目开发主要目录之一,可以放java代码和一些资源文件. |
        | ↓↓java | 开发的主要的java代码存放目录 |
        | ↓↓↓cn.acheng1314 | 我的应用程序的包名 |
        | ↓↓resources | 开发中的主要的资源文件存放目录 |
        | ↓↓sql | 开发中主要的sql语句文件存放目录 |
        | ↓↓webapp | web页面和其他web配置、资源文件存放目录 |
        | ↓ test | 项目开发中的测试模块存放路径,包含java代码和资源文件. |
        | ↓↓java | 测试代码存放目录 |
        | ↓↓resources | 测试资源文件存放目录 |
    - 配置目录:
        - 创建main目录下的java目录(用于存放java源代码)
        
            ![WebApp目录调整第一步创建java源代码目录](http://acheng1314.cn/wp-content/uploads/2016/09/WebApp目录调整第一步创建java源代码目录.png)
            
            我们先**右键点击main目录**,接着选中**New**→**Directory**,在弹出的对话框中输入java.
        - 接着我们需要把java目录标记为源文目录.
            
            ![WebApp目录调整第二步标记java目录为资源目录](http://acheng1314.cn/wp-content/uploads/2016/09/WebApp目录调整第二步标记java目录为资源目录.png)
            
            我们先**右键点击java**,然后选择**Mark Directory As**→**Sources Root**
            
            接着我们在src目录下创建test目录(注意: **test目录和main目录同级**),以及test下面的java和resources目录,分别标记为源文件目录和资源文件目录
            
            **值得注意的是sql目录为普通文件目录**
            
- 根据目标明白我们**需要哪些支援库**,具体结果如下:
    
    
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <groupId>cn.acheng1314</groupId>  <!-- 对应前面设置的GroupId -->
        <artifactId>SSM_LOG</artifactId>   <!-- 前面设置的artifactId-->
        <packaging>war</packaging>  <!-- 打包方式war -->
        <version>1.0-SNAPSHOT</version> <!-- 版本号 -->
        <name>SSM_LOG Maven Webapp</name> <!-- 显示名字 -->
        <url>http://maven.apache.org</url>
        <dependencies>  <!-- 远程依赖库 -->
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>3.8.1</version>
                <scope>test</scope>
            </dependency>
            <!-- 1.日志 -->
            <!--&lt;!&ndash; 实现slf4j接口并整合 &ndash;&gt;-->
            <!--<dependency>-->
            <!--<groupId>ch.qos.logback</groupId>-->
            <!--<artifactId>logback-classic</artifactId>-->
            <!--<version>1.1.1</version>-->
            <!--</dependency>-->
            <!--log4j2支持-->
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-core</artifactId>
                <version>${org.apache.logging.log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.logging.log4j</groupId>
                <artifactId>log4j-api</artifactId>
                <version>${org.apache.logging.log4j.version}</version>
            </dependency>
    
    
            <!-- 2.数据库 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
                <scope>runtime</scope>
            </dependency>
            <!--druid==>阿里巴巴数据库连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${com.alibaba.druid.version}</version>
            </dependency>
            <!--<dependency>-->
            <!--<groupId>c3p0</groupId>-->
            <!--<artifactId>c3p0</artifactId>-->
            <!--<version>0.9.1.2</version>-->
            <!--</dependency>-->
    
            <!-- DAO: MyBatis -->
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis</artifactId>
                <version>${com.mybatis.mybatis.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis</groupId>
                <artifactId>mybatis-spring</artifactId>
                <version>${com.mybatis.mybatis_spring.version}</version>
            </dependency>
    
            <!-- 3.Servlet web -->
            <dependency>
                <groupId>taglibs</groupId>
                <artifactId>standard</artifactId>
                <version>1.1.2</version>
            </dependency>
            <dependency>
                <groupId>jstl</groupId>
                <artifactId>jstl</artifactId>
                <version>1.2</version>
            </dependency>
            <!--json工具-->
            <!--<dependency>-->
            <!--<groupId>com.fasterxml.jackson.core</groupId>-->
            <!--<artifactId>jackson-databind</artifactId>-->
            <!--<version>2.5.4</version>-->
            <!--</dependency>-->
            <dependency>
                <groupId>com.google.code.gson</groupId>
                <artifactId>gson</artifactId>
                <version>${com.google.gson.version}</version>
            </dependency>
            <!--Servlet版本设置-->
            <dependency>
                <groupId>javax.servlet</groupId>
                <artifactId>javax.servlet-api</artifactId>
                <version>${javax.servlet.version}</version>
            </dependency>
    
            <!-- 4.Spring -->
            <!-- 1)Spring核心 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <!-- 2)Spring DAO层 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-jdbc</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-tx</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <!-- 3)Spring web -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-webmvc</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
            <!-- 4)Spring test -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${org.springframework.version}</version>
            </dependency>
    
            <!-- redis客户端:Jedis -->
            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
                <version>${redis.clients.version}</version>
            </dependency>
            <dependency>
                <groupId>com.dyuproject.protostuff</groupId>
                <artifactId>protostuff-core</artifactId>
                <version>${com.dyuproject.protostuff.version}</version>
            </dependency>
            <dependency>
                <groupId>com.dyuproject.protostuff</groupId>
                <artifactId>protostuff-runtime</artifactId>
                <version>${com.dyuproject.protostuff.version}</version>
            </dependency>
    
            <!-- Map工具类 -->
            <dependency>
                <groupId>commons-collections</groupId>
                <artifactId>commons-collections</artifactId>
                <version>3.2.2</version>
            </dependency>
    
            <!--文件上传工具-->
            <dependency>
                <groupId>commons-fileupload</groupId>
                <artifactId>commons-fileupload</artifactId>
                <version>1.3.2</version>
            </dependency>
            <dependency>
                <groupId>commons-io</groupId>
                <artifactId>commons-io</artifactId>
                <version>2.5</version>
            </dependency>
    
        </dependencies>
    
        <!-- 配置可变版本号,也就是常说的版本管理 （Spring、SpringMvc、Mybatis、Gson、Druid） -->
        <!-- 要针对某个依赖进行升级的时候只需要更改下面对应的版本号 -->
        <!-- 在上面使用版本号的时候需要用固定格式,如: ${包名.version} -->
        <properties>
            <org.apache.logging.log4j.version>2.6.2</org.apache.logging.log4j.version>
            <mysql.version>5.1.37</mysql.version>
            <com.alibaba.druid.version>1.0.25</com.alibaba.druid.version>
            <com.mybatis.mybatis.version>3.4.1</com.mybatis.mybatis.version>
            <com.mybatis.mybatis_spring.version>1.3.0</com.mybatis.mybatis_spring.version>
            <com.google.gson.version>2.7</com.google.gson.version>
            <javax.servlet.version>3.1.0</javax.servlet.version>
            <org.springframework.version>4.3.2.RELEASE</org.springframework.version>
            <redis.clients.version>2.7.3</redis.clients.version>
            <com.dyuproject.protostuff.version>1.0.8</com.dyuproject.protostuff.version>
            <developer.organization><![CDATA[scengine]]></developer.organization>
        </properties>
        
        <!-- 构建项目的最终名称 -->
        <build>
            <finalName>SSM_LOG</finalName>
        </build>
    </project>
    
    

----
#### 整合框架

在上面,我们已经把基本的目录配置好了,现在我们在已经依赖了项目支援库,接下来我们需要做的是开始**整合Spring+SpringMvc+Mybatis**

我们先**打开webapp目录下面的WEB-INF目录中的web.xml文件**,web.xml文件是整合web项目的配置中心.我们在web.xml中加入如下内容:

```
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1" metadata-complete="true">
         
    <!--默认的首页-->
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
    
    <!-- 如果是用maven命令生成的xml，需要修改servlet版本为3.1 -->
    <!-- 配置DispatcherServlet -->
    <servlet>
        <display-name>SSM_LOG</display-name>    <!-- 项目名称 -->
        <servlet-name>mvc-dispatcher</servlet-name> <!-- mvc调度器 -->
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置springMVC需要加载的配置文件
            spring-dao.xml,spring-service.xml,spring-web.xml
            Mybatis - > spring -> springmvc
         -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-*.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <!-- 默认匹配所有的静态资源,此处配置出错,会产生错误500 -->
        <url-pattern>/js/*</url-pattern>
        <url-pattern>/css/*</url-pattern>
        <url-pattern>/images/*</url-pattern>
        <url-pattern>/fonts/*</url-pattern>
    </servlet-mapping>

    <!--druid ==> WEB方式监控配置-->
    <servlet>
        <servlet-name>DruidStatView</servlet-name>
        <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DruidStatView</servlet-name>
        <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>
    <filter>
        <filter-name>druidWebStatFilter</filter-name>
        <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
        <init-param>
            <param-name>exclusions</param-name>
            <param-value>/public/*,*.js,*.css,/druid*,*.jsp,*.swf</param-value>
        </init-param>
        <init-param>
            <param-name>principalSessionName</param-name>
            <param-value>sessionInfo</param-value>
        </init-param>
        <init-param>
            <param-name>profileEnable</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>druidWebStatFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

#### 快捷生成spring目录
- 在上面的```<param-value>classpath:spring/spring-*.xml</param-value>```处,我们选中前面一个spring,按下Alt+Enter自动生成spring目录.
- spring目录位于src→main→resources下.

#### 在spring目录下创建spring相关的控制文件
- spring-dao.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
        <!-- 配置整合mybatis过程 -->
        <!-- 1.配置数据库相关参数properties的属性：${url} -->
        <!-- 使用数据库配置文件解耦 -->
        <context:property-placeholder location="classpath:jdbc.properties"/>
    
        <!-- 下面的druid配置都是基本配置,具体优化设置可以上网查询,也可以去github上面直接搜索druid -->
        <!-- 2.数据库连接池 -->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
              init-method="init" destroy-method="close">
            <!-- 配置连接池属性 -->
            <property name="driverClassName" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
    
            <!-- 配置初始化大小、最小、最大 -->
            <property name="initialSize" value="1" />
            <property name="minIdle" value="1" />
            <property name="maxActive" value="10" />
    
            <!-- 配置获取连接等待超时的时间 -->
            <property name="maxWait" value="10000" />
    
            <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
            <property name="timeBetweenEvictionRunsMillis" value="60000" />
    
            <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
            <property name="minEvictableIdleTimeMillis" value="300000" />
    
            <property name="testWhileIdle" value="true" />
    
            <!-- 这里建议配置为TRUE，防止取到的连接不可用 -->
            <property name="testOnBorrow" value="true" />
            <property name="testOnReturn" value="false" />
    
            <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
            <property name="poolPreparedStatements" value="true" />
            <property name="maxPoolPreparedStatementPerConnectionSize"
                      value="20" />
    
            <!-- 这里配置提交方式，默认就是TRUE，可以不用配置 -->
    
            <property name="defaultAutoCommit" value="true" />
    
            <!-- 验证连接有效与否的SQL，不同的数据配置不同 -->
            <property name="validationQuery" value="select 1 " />
            <property name="filters" value="stat" />
            <property name="proxyFilters">
                <list>
                    <ref bean="logFilter" />
                </list>
            </property>
        </bean>
    
        <!-- 3.配置SqlSessionFactory对象 -->
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <!-- 注入数据库连接池 -->
            <property name="dataSource" ref="dataSource"/>
            <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
            <property name="configLocation" value="classpath:mybatis-config.xml"/>
            <!-- 扫描entity包 使用别名 -->
            <!-- cn.acheng1314是我的应用程序的包名,你们需要使用你们自己的包名,也就是前面我们提到过的GroupId -->
            <property name="typeAliasesPackage" value="cn.acheng1314.domain"/>
            <!-- 扫描sql配置文件:mapper需要的xml文件 -->
            <property name="mapperLocations" value="classpath:mapper/*.xml"/>
        </bean>
    
        <!-- 4.配置扫描Dao接口包，动态实现Dao接口，注入到spring容器中 -->
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <!-- 注入sqlSessionFactory -->
            <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
            <!-- 给出需要扫描Dao接口包 -->
            <property name="basePackage" value="cn.acheng1314.dao"/>
        </bean>
    
        <!-- 上面的druid的配置 -->
        <bean id="logFilter" class="com.alibaba.druid.filter.logging.Slf4jLogFilter">
            <property name="statementExecutableSqlLogEnable" value="false" />
        </bean>
    </beans>
    ```

上面的配置中,肯定会出现报错的情况,这时候我们只需要选中报错的地方按下Alt+Enter就能生成相关的资源.

- spring-service.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
        
        <!-- 扫描service包下所有使用注解的类型 -->
        <!-- cn.acheng1314为我们应用的包名,当然也是我们前面提到过的GroupId -->
        <context:component-scan base-package="cn.acheng1314.service" />
    
        <!-- 配置事务管理器 -->
        <bean id="transactionManager"
              class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <!-- 注入数据库连接池 -->
            <property name="dataSource" ref="dataSource" />
        </bean>
    
        <!-- 配置基于注解的声明式事务 -->
        <tx:annotation-driven transaction-manager="transactionManager" />
    </beans>
    ```

上面的配置中,肯定会出现报错的情况,这时候我们只需要选中报错的地方按下Alt+Enter就能生成相关的资源.

**基本的spring系列和druid**已经配置完毕. 接着我们需要解决上面自动生成的一些问题.基本配置截图如下:

![基本的spring配置和druid配置后截图](http://acheng1314.cn/wp-content/uploads/2016/09/基本的spring配置和druid配置后截图.png)

现在我们会发现我们的jdbc.properties和mybatis-config.xml文件都是空的,我们需要继续写入内容.

jdbc.properties是数据库连接的配置文件.如下:

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3307/wordpress?useUnicode=true&characterEncoding=utf8
jdbc.username=数据库用户名
jdbc.password=数据库用户名对应的密码
```

上面的jdbc.driver为数据库连接的驱动,jdbc.url为数据库的连接地址.

mybatis-config.xml 顾名思义是mybatis的配置文件,如下:

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置全局属性 -->
    <settings>
        <!-- 使用jdbc的getGeneratedKeys获取数据库自增主键值 -->
        <setting name="useGeneratedKeys" value="true" />

        <!-- 使用列别名替换列名 默认:true -->
        <setting name="useColumnLabel" value="true" />

        <!-- 开启驼峰命名转换:Table{create_time} -> Entity{createTime} -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
    </settings>
</configuration>
```

配置完成上面的东西后,大体需要的我们已经完成了.但是,我们会看到我们的日志记录还没有配置,上面我们采用了log4j2,通过查看官网文档,我们发现只需要在资源目录下面添加一个默认的配置文件即可,如下:

配置文件文件名: **log4j2.xml** , 存放目录为**src**→**main**→**resources**
```
<?xml version="1.0" encoding="UTF-8"?>
<!-- status=debug 可以查看log4j的装配过程 -->
<configuration status="off" monitorInterval="1800">
    <properties>
        <!--日志目录-->
        <property name="LOG_HOME">/logs/webLog</property>
        <!-- 日志备份目录 -->
        <property name="BACKUP_HOME">{LOG_HOME}/backup</property>
        <property name="STAT_NAME">stat</property>
        <property name="SERVER_NAME">global</property>
    </properties>
    <appenders>
        <!-- 定义控制台输出 -->
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n"/>
        </Console>
        <!-- 程序员调试日志 -->
        <RollingRandomAccessFile name="DevLog" fileName="${LOG_HOME}/${SERVER_NAME}"
                                 filePattern="${LOG_HOME}/${SERVER_NAME}.%d{yyyy-MM-dd-HH}.log">
            <PatternLayout pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingRandomAccessFile>
        <!-- 游戏产品数据分析日志 -->
        <RollingRandomAccessFile name="ProductLog"
                                 fileName="${LOG_HOME}/${SERVER_NAME}_${STAT_NAME}"
                                 filePattern="${LOG_HOME}/${SERVER_NAME}_${STAT_NAME}.%d{yyyy-MM-dd-HH}.log">
            <PatternLayout
                    pattern="%date{yyyy-MM-dd HH:mm:ss.SSS} %level [%thread][%file:%line] - %msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"
                                           modulate="true"/>
            </Policies>
        </RollingRandomAccessFile>
    </appenders>
    <loggers>
        <!-- 3rdparty Loggers -->
        <logger name="org.springframework.core" level="info">
        </logger>
        <logger name="org.springframework.beans" level="info">
        </logger>
        <logger name="org.springframework.context" level="info">
        </logger>
        <logger name="org.springframework.web" level="info">
        </logger>
        <logger name="org.jboss.netty" level="warn">
        </logger>
        <logger name="org.apache.http" level="warn">
        </logger>
        <logger name="com.mchange.v2" level="warn">
        </logger>
        <!-- Game Stat  logger -->
        <logger name="com.u9.global.service.log" level="info"
                additivity="false">
            <appender-ref ref="ProductLog"/>
        </logger>
        <!-- Root Logger -->
        <root level="DEBUG">
            <appender-ref ref="DevLog"/>
            <appender-ref ref="Console"/>
        </root>
    </loggers>
</configuration>
```

----
至此,我们的基本配置就完成了,结果如下图所示:

![ssm框架整合完毕截图](http://acheng1314.cn/wp-content/uploads/2016/09/ssm框架整合完毕截图.png)

具体基本配置完毕,下面我们需要进行实际演练方可知道效果,也能根据实际效果检查配置有没有出现问题.至于实际演练如何,且听下回分解.
