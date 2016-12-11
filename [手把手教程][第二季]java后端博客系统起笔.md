转眼间时间就从9月份到现在的十一月份了。这段时间说实话做的有意义的事情太少。现在还是单身···

闲话直接跳过了，嗯，手把手教程第二季已经来了，第一季就不用再写什么第一季汇总资源之类的记录了，直接扔出第一季的总集合地址。
[[手把手教程][JavaWeb]第一季点击这里查看所有文章](http://www.jianshu.com/notebooks/4409922/latest)。当然，也可以直接访问[我的博客](http://acheng1314.cn)。

最近一直在想怎么搞的更好，怎样描述能更加简单直观的解决问题。第一季我们采用了以下的描述方法：
- 列表
- 画流程图
- 贴效果图
- 语言描述
- 直接贴代码

第二季我考虑适当的引入一些软件工程的概念，以及常用的思维模式的一些实现，大概想做一些下面的东西：
- [思维导图](http://baike.baidu.com/link?url=bWRI4YoX8CE7krCZgFBNA7V_YJ2xe6jqg6sqyijipOJUSk_vNb7SUXBxz0KNHb7PKW9fSWThFyHaCfLe4n4Oy2C0C3pNTwYxPIFb4kZOX-Bq4M74XINBEGLmEYcXP4OT)
- [流程图](http://baike.baidu.com/link?url=Sa6_MwnUpZ8kmoX7Gj9IJnG7zgSAjxSYV4ZV-NSVilCIW7UN3xWYX4rKGLssfBTt3eL0ujH5rHFSKc8iP2lE8rIbu3FwBehdG68Oucpp6k41GW5Tr8k9lxEl5YqWXd5a)
- [数据流图](http://baike.baidu.com/link?url=w0ek92zthobLfoZ1a7Cl5_ivun6bGSIpy7r98y66LzIcLLhQQB7p4OHWGTLKJPSe9oGiqRBenvgHzN_DK2VnUe24VW1lxz43MyvsKhh3li3PsupYkAeYnSqei79B4Bxx)
- [E-R图](http://baike.baidu.com/link?url=uK05RHdstOYc9iEI0ep_IKr0FspwXsV2XBg9dal1IKesLm4PuzffTT-YKVleDb0nmZ7-Asoxddh-zvQhOVEc1zpvVmjU6oEgLxyluRBjVSa)
- [UML建模](http://baike.baidu.com/link?url=9FrEX5BNxDkzyDZHxvfItP3mdPPgAzkBBckm9ZWkwaX1c0mSn6KJdQ55Y5ZxuanxsaeqZvKPydq0-QEgcOuy99wMA7SWx5oxkYEVWVjHuTe9B5F-Emsl72JgjnpQ3RFe)

说实话上面的这些东西，在实际开发中我们可能不是每次开发都准备这些东西，但是我们在平时可以考虑把这些东西都准备一下，到了一些时候我们的脑袋里自然会有这些相关的概念浮现。而且这样分析程序组织结构和执行流程对我们每个人的成长也已有利的，所以希望同学们能一起互勉。

----
软件工程讲究的是以工程学的角度来控制软件的研发。核心目的是：提高效率降低成本。我们在实际开发中如何体现这些东西呢？

> 思维导图

为什么要把思维导图放在最前面？思维导图又叫心智图，是表达发散性思维的有效的图形思维工具，是一种将放射性思考具体化的方法，是一种图像式思维的工具以及一种利用图像式思考辅助工具。简单思维导图如下：

![我的博客第一章第一图](http://acheng1314.cn/wp-content/uploads/2016/11/我的博客第一章第一图.png)

上面这个图是我画的一个关于文章系统设计的图(中间有小瑕疵，将就的看=,=)，这个就是我们常用的思维导图的作用之一，能帮助我们理清思路和功能结构。具体的思维导图我们就不再多做介绍了，在上面的链接中都可以查看，思维导图推荐的工具是xmind。

> 流程图

流程图相对来说是我们现在相对更加熟悉的东西，在前面的第一季的文章中我们能看到很多关于流程图的绘画。流程图是流经一个系统的信息流、观点流或部件流的图形代表，它以特定的图形符号加上说明来表示事物执行流程。

> 数据流图

数据流图：简称DFD（Data Flow Diagram），它从数据传递和加工角度，以图形方式来表达系统的逻辑功能、数据在系统内部的逻辑流向和逻辑变换过程，是结构化系统分析方法的主要表达工具及用于表示软件模型的一种图示方法。

- 指明数据存在的数据符号，这些数据符号也可指明该数据所使用的媒体；
- 指明对数据执行的处理的处理符号，这些符号也可指明该处理所用到的机器功能；
- 指明几个处理和（或）数据媒体之间的数据流的流线符号；
- 便于读、写数据流程图的特殊符号。

![简单的数据流图实例](http://p.blog.csdn.net/images/p_blog_csdn_net/turkeyzhou/EntryImages/20100106/4.jpg)

数据流图虽然说在名字上面听起来有点类似流程图，但是实际上两者差异还是较大，同时我们可以很明显的看到数据流图把程序执行的数据流转示意表现的很清楚，所以我们也需要他来帮我们完成一些事情。

> E-R图

E-R图：实体-联系图(Entity Relationship Diagram)，提供了表示实体类型、属性和联系的方法，用来描述现实世界的概念模型。

> UML建模

UML建模技术就是用模型元素来组建整个系统的模型，模型元素包括系统中的类、类和类之间的关联、类的实例相互配合实现系统的动态行为等。

UML是面向对象开发中一种通用的图形化建模语言。面向对象的分析主要在加强对问题空间和系统任务的理解、改进各方交流、与需求保持一致和支持软件重用等4个方面比较突出，因此也成为现在主流的建模方法（在IDEA中我们能看到项目对应的Uml模型）。

相对于其他的图示，我更加喜欢UML建模，他能很生动形象的表现出各个类、接口之间的关系，如下图：

![泛型接口的实现和接口继承](http://acheng1314.cn/wp-content/uploads/2016/11/我的博客第一章第二图泛型接口的实现和接口继承.png)

![javaBean实现Serializable接口](http://acheng1314.cn/wp-content/uploads/2016/11/我的博客第一章第三图javaBean实现Serializable接口.png)

上面的第一张图中我们可以看到是我的UserDao继承了BaseDao并且将泛型T具体化为User。
``` java
public interface UserDao extends Dao<User> {
    int add(User user);

    int del(User user);

    int update(User user);

    User findOneById(Serializable Id);

    List<User> findAll();

    void updateLoginSession(@Param("sessionId") String sessionId, @Param("loginId") String loginId);

    void addSessionId(String id);
}
```
同理可得，我们的PostDao也是继承BaseDao并且将泛型T具体化为PostBean。

第二张图中，实际就是我们的User和PostBean这两个javaBean，他们同时实现了接口Serializable。

上面两张图中我们可以看到：
- 类或者接口的继承用实线箭头表示
- 类实现接口用虚线箭头表示
- 泛型具体化也是用实线箭头表示
- 类使用淡蓝色方框表示
- 接口使用淡紫色方框表示

具体的一些东西我们后面再详细介绍，现在大概明白即可（当然老司机肯定是直接跳过）。

----

> 倚赖wordpress数据库的博客系统

这一季我们的正式目标是做一个博客系统，然后倚赖的是以前的wordpress博客的数据库。这几天大概整理了功能如下：

![博客系统整体结构图](http://acheng1314.cn/wp-content/uploads/2016/11/我的博客第一章第四图博客系统整体结构图.png)

为什么说打算做这一个东西，主要是因为首先我个人的博客被人家刷评论了，第二点是博客一直被人攻击，想用自己的系统来和别人斗智斗勇看看。

做重要的是想自己作一些属于自己的东西，留下一些记录的痕迹。

这个第一期只能说不算开篇的开篇吧，在后面的文章中可能我们很多时候更多是怎么样去引导思维这样子做事，而不是怎么样去编码。

希望在这新的一季里面我们能有更多的收获，一起加油吧。