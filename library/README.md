# library 跨项目、团队公共资源库

这是一个为多个项目及团队提供公共资源支持的仓库。    

> 注意
1. 放入此处的文件是否满足提供多项目使用的条件

2. 此项目的文件改动必须向上兼容，不允许再未经讨论的情况下进行大的重构

3. 提交项目代码时正常提交即可，但是需要手动将此目录推送至公共资源仓库

4. 不收录任何与业务相关的逻辑组件，如附带有业务逻辑代码组件放至components文件夹作为项目内公共组件；   
  如有特殊基础类情况，则不可写死业务数据，以尽可能的传入参数、slot、函数回调来完成相关逻辑

> 提交公共资源子仓库时，在项目根路径执行以下命令

Git Submodules 介绍（通俗易懂，总结了工作完全够用的 submodule 命令）
===============================================
为什么有 submodules？
----------------

1.  **解决公共代码问题**。如果某些文件，在项目A和项目B中都会用到，例如组件库，那么这些文件可以作为 submodules 来管理，减少重复代码。（当然，该场景下npm包是另一解决方案，你需要选择一种方案。）
2.  **解决团队维护难题**。如果一个大项目是一个大 Git 仓库，需要统一编译，不同的模块由不同团队维护，放在同一个 Git 仓库有诸多难处：例如多个团队的 MR 混在一起、权限难以区分等。这种情况即使公司内网 Git 权限做的足够精细，仓库管理员的学习成本也会很高，很难深度使用这种高级功能。为了解决多团队维护的难题，Git Submodules 也能大展身手，它可以让每个团队负责的模块就是一个 Git 仓库，这些 Git 仓库都被包含在同一个主 Git 项目下。（当然，微前端、微服务是另一种解决方案，你需要选择一种方案。）

了解 Git Submodules
-----------------

有2个概念：主项目、submodule（子模块）。这两者各自都是完整的 Git 仓库。

### 如何让一个Git仓库变为另一个Git仓库的 submodule

1.  创建Git仓库A。
2.  创建Git仓库B。
3.  在Git仓库A中，通过​`​git submodule add ...(仓库B的地址，即git clone时后面那串东西)​`​​，可以把仓库B当作仓库A的submodule，此时A就成了主项目。【注：B也可以做A的主项目，通过在仓库B执行​`​git submodule add ...(A地址)​`​即可，因为二者都是完整Git仓库，在建立父子关系前，没有差异的。】

**注意事项**

*   执行操作后，会在当前父项目下新建个文件夹，名字就是 submodule 仓库的名字。这个文件夹里面的内容，**是 submodule 对应 Git 仓库的完整代码。** 
*   如果你希望换个名字，或者换个路径（例如放在某个更深的目录下），也是允许的，需要后面增加个路径参数，例如​`​git submodule add ...(仓库地址) src/B(你希望 submodule 位于的文件夹路径)​`​

### submodule 的父子关系存在哪里

**关系是保存在主项目的 Git 仓库中。** 

被当作 submodule 的 Git 仓库，其实不知道自己变成了 submodule，它更不知道爸爸们有谁。（意思是，当你打开某个被当作 submodule 的 Git 仓库首页时，或者拉下这个仓库时，没有任何痕迹表明它是个submodule。因为父子信息不存在这里，只存在爸爸那里。）

### submodule 的父子关系信息怎么存

#### .gitmodules 文件

父子关系的信息保存在主项目的 ​`​.gitmodules​`​ 文件，如果不是新加 submodule，**这个文件通常不必改变了，因为信息比较固定。** 

![](https://s2.51cto.com/images/blog/202210/22000818_6352c3f2c2dde19781.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

#### submodule 的版本号

主项目还保存了对应 submodule 的版本号（commit id），**没有冗余存储 submodule 的代码**。

![](https://s2.51cto.com/images/blog/202210/22000818_6352c3f2d163428772.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

可以看到，这其实是个跳转到另一个仓库的链接，指明了具体的 commit id。

**这个版本号，是需要经常变更的。** 

submodule 开发常用操作
----------------

### 如何给 submodule 仓库提交更新

#### 方法一，像普通仓库一样更新

之前说过，submodule 仓库也是个普通的 Git 仓库，它并不知道自己有多少个爸爸。

我们可以直接​`​git clone xxx​`​这个仓库，像给普通 Git 仓库提交更新一样，去更新它。

#### 方法二，在主项目中更新

例如主项目在文件夹​`​A​`​，A里面包含：

*   ​`​.git​`​文件夹
*   ​`​READMD.md​`​主项目的ReadMe文件。
*   ​`​B​`​文件夹，是个 submodule。

我们可以进入B文件夹​`​cd B​`​​，你会发现在B中，也可以执行​`​git status​`​​等命令，此时的​`​git​`​命令都会是针对仓库B的，你可以在这里切换分支、提交更新，这时候，提交的都是submodule的变更。

> 这是更推荐的方式。因为很多时候 submodule 是对主项目仓库有依赖的。可能它是个组件库，没有运行环境，需要在主项目中debug。这点非常方便！

**注意事项**

当你在文件夹B中做commit后，文件夹B里面就有了新的 commit id。此时主项目A中所记录的 submodule 的commit id也会更新。所以，你​`​cd ..​`​回到文件夹A后，会发现A有变更了，变更内容是：旧commid id变成了新的commit id。

![](https://s2.51cto.com/images/blog/202210/22000818_6352c3f2de5f851268.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

下面是​`​git diff​`​：

![](https://s2.51cto.com/images/blog/202210/22000819_6352c3f30e24011279.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

### 如何在主项目仓库，拉取 submodule 的更新

#### 方法一，cd submodule 后 git pull

在 submodule 中，所有git操作就当作一个普通的 Git 仓库就行，你可以切换分支、提交代码、拉取更新等。

这个方法，你可以拉取到 submodule 的master最新代码。但是如果这时候的commit id跟主项目里记录的 submodule 的 commit id 不一致，你会在主项目仓库看到diff，你可能需要提交主项目更新。

#### 方法二，主项目执行git submodule update --remote \[submodule文件夹相对路径\]

这个方法会自动拉取submodule的主分支（通常叫master或main）的最新版本。效果跟方法一一致。

如果你不带参数​`​[submodule文件夹相对路径]​`​，就会更新所有 submodules。

**注意事项，更新后需提交主项目变更。** 

当我们更新子项目后，相当于是把主项目记录的 submodule 的 commit id 给更新了，需要提交下主项目的变更。

#### 方法三，主项目执行 git submodule update \[submodule文件夹相对路径\]

注意，这个方法会使 submodule 的分支处于主项目里指定的 commit id。可能并不是拉 submodule 的 master 最新代码。

所以，这种方法仅适用于，当主仓库里记录的 submodule 的 commit id 已经是最新的（可能被其他同事提交过）。或者你期望 submodule 跟主仓库记录的保持一致时，也可以使用该方法。

### 如何 clone 包含 submodule 的仓库

#### 方法一，按需clone submodule

1.  先​`​git clone 主项目仓库​`​并进入主项目文件夹，这时候submodule的文件夹都是空的。
2.  执行​`​git submodule init [submodule的文件夹的相对路径]​`​。
3.  执行​`​git submodule update [submodule的文件夹的相对路径]​`​。

这就按需clone了submodule。什么时候有用呢？跨团队协作某个主项目时，一些其它团队的submodule我们没必要安装，就不必执行​`​init​`​​和​`​update​`​了。

**合并第2、3步骤**

第2、3步可以合并。使用以下命令：

登录后复制

```


git submodule update --init \[submodule的文件夹的相对路径\]

*   1.


```

注意顺序，​`​--init​`​​跟​`​[submodule的文件夹的相对路径]​`​的位置不可以调换噢。

![](https://s2.51cto.com/images/blog/202210/22000819_6352c3f32042e75013.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

#### 方法二，一次性clone所有 submodule

1.  先​`​git clone 主项目仓库​`​，这时候submodule的文件夹都是空的。
2.  执行​`​git submodule init​`​。
3.  执行​`​git submodule update​`​。

只要不写submodule，那么就一次性检查该主项目的所有submodule，都拉下来。

![](https://s2.51cto.com/images/blog/202210/22000819_6352c3f333f3989495.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

**合并第2、3步骤**

登录后复制

```


git submodule update --init

*   1.


```

**合并第1、2、3步骤**

登录后复制

```


git clone \--recurse-submodules \[主项目Git仓库地址\]

*   1.


```

![](https://s2.51cto.com/images/blog/202210/22000819_6352c3f34882f51766.webp?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=/format,webp/resize,m_fixed,w_1184)

### 如何创建 submodule 关系

cd到主项目，执行：

登录后复制

```


git submodule add ...(另一仓库地址，即git clone时后面那串东西)

*   1.


```

下面可以指定 submodule 放到哪个子文件夹

登录后复制

```


git submodule add ...(另一仓库地址) \[(可选，submodule下载的路径)\]

*   1.


```

更多资料
----

通过官方文档，你可以了解到更多场景，但是我从来没使用过其它场景了，因为用不到。本文描述的完全满足了我所有日常使用场景。

而高级场景会导致协作变困难，因为不是所有开发者都懂这些更复杂的命令和配置：

*   嵌套的submodule怎么快速拉取？（就是有个父项目有儿子、也有孙子、还有祖孙子等等，通过​`​--recurse-submodules​`​​或​`​--recursive​`​参数）。
*   通过配置​`​git config -f .gitmodules submodule.子模块文件夹相对目录.branch 子模块分支名​`​​，使得每次执行​`​git submodule update --remote​`​时，追踪任意指定的子模块分支（而非默认的主分支master）。
*   通过​`​foreach​`​命令可以方便的给所有子模块依次执行一样的命令。

想了解，可以阅读 Git Submodules 官方文档，搜索命令相关关键字：

*   英文：www.git-scm.com/book/en/v2/…
*   中文：git-scm.com/book/zh/v2/…

