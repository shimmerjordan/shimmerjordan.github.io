---
title: MVC-MVP-MVVM
date: 2020-03-10 12:42:20
tags: 
	- Web开发
	- 体系结构
	- MVC
	- MVP
	- MVVE
categories: Web体系结构
id: MVC-MVP-MVVM
---
复杂的软件必须有清晰合理的架构，否则难以开发与维护。MVC（Model-View-Controller）是最常见的软件架构之一，广泛应用于业界。在MVC的基础上衍生出了MVP以及MVVM架构，这些名词均是为了解决图形界面应用程序复杂性管理问题而产生的应用架构模式。

本文分别对其相关特性做了简要分析，并从图形角度简析其间区别与联系。
<!--more-->

# UI程序面临问题

图形界面的应用程序提供用户可视化的操作界面，为其提供数据和信息。用户的输入行为会导致一些业务逻辑的触发，可能会变更应用程序的数据。而数据的变更自然需要用户界面的同步更新。例如用户对数据进行筛选，那么程序执行完筛选的业务逻辑后，需要将符合筛选条件的数据同步反馈到界面上。

在开发应用程序时，为更好管理其复杂性，我们通常基于职责分离的思想对应用程序进行分层。

# MVC
MVC的全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，是一种软件设计典范。它是用一种业务逻辑、数据与界面显示分离的方法来组织代码，将众多的业务逻辑聚集到一个部件里面，在需要改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑，达到减少编码的时间。

MVC开始是存在于桌面程序中的，M是指业务模型，V是指用户界面，C则是控制器。

## 使用的MVC的目的
在于将M和V的实现代码分离，从而使同一个程序可以使用不同的表现形式。

比如Windows系统资源管理器文件夹内容的显示方式，下面两张图中上图为详细信息显示方式，下图边为中等图标显示方式，文件的内容并没有改变，改变的是显示的方式。不管用户使用何种类型的显示方式，文件的内容并没有改变，达到M和V分离的目的。

![列表显示](https://i.imgur.com/ICdm20S.jpg)  
![中等图显示](https://i.imgur.com/eKE78zT.jpg)

## M含义
M即Model模型是指模型表示业务规则。在MVC的三个部件中，模型拥有最多的处理任务。被模型返回的数据是中立的，模型与数据格式无关，这样一个模型能为多个视图提供数据，由于应用于模型的代码只需写一次就可以被多个视图重用，所以减少了代码的重复性。

## V含义
V即View视图是指用户看到并与之交互的界面。比如由html元素组成的网页界面，或者软件的客户端界面。MVC的好处之一在于它能为应用程序处理很多不同的视图。在视图中其实没有真正的处理发生，它只是作为一种输出数据并允许用户操纵的方式。

## C含义
C即Controller控制器是指控制器接受用户的输入并调用模型和视图去完成用户的需求，控制器本身不输出任何东西和做任何处理。它只是接收请求并决定调用哪个模型构件去处理请求，然后再确定用哪个视图来显示返回的数据。

下图说明了三者之间的调用关系。

![](https://i.imgur.com/P2HZqXz.jpg)

用户首先在界面中进行人机交互，然后请求发送到控制器，控制器根据请求类型和请求的指令发送到相应的模型，模型可以与数据库进行交互，进行增删改查操作，完成之后，根据业务的逻辑选择相应的视图进行显示，此时用户获得此次交互的反馈信息，用户可以进行下一步交互，如此循环。

## 简化互动模式
简而言之，接受用户指令时，MVC可以分为两种方式。一种是通过View接受指令，传递给controller：

![](https://i.imgur.com/zJtlFDK.jpg)

另一种是直接通过Controller接受指令：

![](https://i.imgur.com/ddxZ2D1.jpg)

实际项目旺旺采用更领过的方式，以Backbone.js为例：

![](https://i.imgur.com/nUNQ604.jpg)


1. 用户可以向 View 发送指令（DOM 事件），再由 View 直接要求 Model 改变状态  
2. 用户也可以直接向 Controller 发送指令（改变 URL 触发 hashChange 事件），再由 Controller 发送给 View  
3. Controller 非常薄，只起到路由的作用，而 View 非常厚，业务逻辑都部署在 View。所以，Backbone 索性取消了 Controller，只保留一个 Router（路由器）

## MVC例一
最典型的MVC就是jsp + servlet + javabean模式。

JavaBean作为模型，既可以作为数据模型来封装业务数据，又可以作为业务逻辑模型来包含应用的业务操作。其中，数据模型用来存储或传递业务数据，而业务逻辑模型接收到控制器传过来的模型更新请求后，执行特定的业务逻辑处理，然后返回相应的执行结果。

JSP作为表现层，负责提供页面为用户展示数据，提供相应的表单（Form）来用于用户的请求，并在适当的时候（点击按钮）向控制器发出请求来请求模型进行更新。

Serlvet作为控制器，用来接收用户提交的请求，然后获取请求中的数据，将之转换为业务模型需要的数据模型，然后调用业务模型相应的业务方法进行更新，同时根据业务执行结果来选择要返回的视图。

## MVC例二
Struts2框架：Struts2是基于MVC的轻量级的web应用框架。Struts2的应用范围是Web应用，注重将Web应用领域的日常工作和常见问题抽象化，提供一个平台帮助快速的完成Web应用开发。基于Struts2开发的Web应用自然就能实现MVC，Struts2着力于在MVC的各个部分为开发提供相应帮助。

### 代码解释

 **Login.html**

```html
<body>
	<form id="form1" name="form1" action="action/Login.action" method="post">
		登录<br>
		用户名：<input name="username" type"text"><br>
		密码：<input name="password" type="password"><br>
		<input type="submit" value="登录">
	</form>
</body>
```

 **Login.java**

```java
if(u.equals("1") && p.equals("1")) {
	return "Success";
} else {
	return "Error";
}
```

 **Struts.xml**

```html
<struts>
	<package name="deflut" namespace="/action" extends="struts-default">
		<action name="Login" class="com.dc365.s2.Login">
			<result name="Success">../Success.jsp</result>
			<result name="Error">../Error.jsp</result>
		</action>
	</package>
</struts>
```

用户首先在Login.html中输入用户名和密码，点击登陆，此时根据action的路径，在struts.xml中找到对应的Login，然后根据对应的class的路径进入相应的login.Java，在这里判断之后，返回success或error，然后根据struts.xml中的result值，指向相应的jsp页面。

![](https://i.imgur.com/2e3Qvzp.jpg)

#### 控制器——filterdispatcher
从上面这张图来看，用户请求首先到达前端控制器FilterDispatcher。FilterDispatcher负责根据用户提交的URL和struts.xml中的配置，来选择合适的动作(Action)，让这个Action来处理用户的请求。FilterDispatcher其实是一个过滤器（Filter，servlet规范中的一种web组件），它是Struts2核心包里已经做好的类，不需要我们去开发，只是要在项目的web.xml中配置一下即可。FilterDispatcher体现了J2EE核心设计模式中的前端控制器模式。

#### 动作——Action
在用户请求经过FilterDispatcher之后，被分发到了合适的动作Action对象。Action负责把用户请求中的参数组装成合适的数据模型，并调用相应的业务逻辑进行真正的功能处理，获取下一个视图展示所需要的数据。Struts2的Action，相比于别的web框架的动作处理，它实现了与Servlet API的解耦，使得Action里面不需要再直接去引用和使用HttpServletRequest与HttpServletResponse等接口。因而使得Action的单元测试更加简单，而且强大的类型转换也使得我们少做了很多重复的工作。

#### 视图——Result
视图结果用来把动作中获取到的数据展现给用户。在Struts2中有多种优秀的结果展示方式，常规的jsp，模板freemarker、velocity，还有各种其它专业的展示方式，如图表jfreechart、报表JasperReports、将XML转化为HTML的XSLT等等。而且各种视图结果在同一个工程里面可以混合出现。

## MVC例三
**ASP.NET MVC**
ASP.NET项目也含有Controllers，Models和Views目录，体现了MVC编程思想，这里不再赘述。
几个需要特别关注的关键点：

1. View是把控制权交移给Controller，自己不执行业务逻辑。  
2. Controller执行业务逻辑并且操作Model，但不会直接操作View，可以说它是对View无知的。  
3. View和Model的同步消息是通过观察者模式进行，而同步操作是由View自己请求Model的数据然后对视图进行更新。
	

特别注意的是MVC模式的精髓在于第三点：`Model的更新是通过观察者模式告知View的`具体表现形式可以是Pub/Sub或者是触发Events。  

通过观察者模式的好处就是：不同的MVC三角关系可能会有共同的Model，一个MVC三角中的Controller操作了Model以后，两个MVC三角的View都会接受到通知，然后更新自己。保持了依赖同一块Model的不同View显示数据的实时性和准确性。我们每天都在用的观察者模式，在几十年前就已经被大神们整合到MVC的架构当中。

## MVC的优点：
### 1.耦合性低
视图层和业务层分离，这样就允许更改视图层代码而不用重新编译模型和控制器代码，同样，一个应用的业务流程或者业务规则的改变只需要改动MVC的模型层即可。因为模型与控制器和视图相分离，所以很容易改变应用程序的数据层和业务规则。

### 2.重用性高
MVC模式允许使用各种不同样式的视图来访问同一个服务器端的代码，因为多个视图能共享一个模型，它包括任何WEB（HTTP）浏览器或者无线浏览器（wap），比如，用户可以通过电脑也可通过手机来订购某样产品，虽然订购的方式不一样，但处理订购产品的方式是一样的。由于模型返回的数据没有进行格式化，所以同样的构件能被不同的界面使用。

### 3.部署快，生命周期成本低
MVC使开发和维护用户接口的技术含量降低。使用MVC模式使开发时间得到相当大的缩减，它使程序员（Java开发人员）集中精力于业务逻辑，界面程序员（HTML和JSP开发人员）集中精力于表现形式上。

### 4.可维护性高
分离视图层和业务逻辑层也使得WEB应用更易于维护和修改。

## MVC的缺点：
### 1.完全理解MVC比较复杂。
由于MVC模式提出的时间不长，加上同学们的实践经验不足，所以完全理解并掌握MVC不是一个很容易的过程。

### 2.调试困难。
因为模型和视图要严格的分离，这样也给调试应用程序带来了一定的困难，每个构件在使用之前都需要经过彻底的测试。

Controller测试困难。因为视图同步操作是由View自己执行，而View只能在有UI的环境下运行。在没有UI环境下对Controller进行单元测试的时候，Controller业务逻辑的正确性是无法验证的：Controller更新Model的时候，无法对View的更新操作进行断言。

### 3.不适合小型，中等规模的应用程序
在一个中小型的应用程序中，强制性的使用MVC进行开发，往往会花费大量时间，并且不能体现MVC的优势，同时会使开发变得繁琐。

### 4.增加系统结构和实现的复杂性
对于简单的界面，严格遵循MVC，使模型、视图与控制器分离，会增加结构的复杂性，并可能产生过多的更新操作，降低运行效率。


### 5.视图与控制器间的过于紧密的连接并且降低了视图对模型数据的访问
视图与控制器是相互分离，但却是联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用。

### 6.View无法组件化
View是强依赖特定的Model的，如果需要把这个View抽出来作为一个另外一个应用程序可复用的组件就困难了。因为不同程序的的Domain Model是不一样的

# MVP
MVP全称Model-View-Presenter，是MVC模式的一种变体。

## 分类
MVP有两种：  
1. Passive View  
2. Supervising Controller

这里讨论大多数的情况：Passive View

## MVP（Passive View）的依赖关系

MVP模式把MVC模式中的Controller换成了Presenter。MVP层次之间的依赖关系如下：

![](https://i.imgur.com/V8rlnOZ.jpg)

MVP打破了View原来对于Model的依赖，其余的依赖关系和MVC模式一致。

## MVP（Passive View）的调用关系

既然View对Model的依赖被打破了，那View如何同步Model的变更？看看MVP的调用关系：

![](https://i.imgur.com/fzL6YPZ.jpg)

1. 各部分之间的通信，都是双向的。
2. View 与 Model 不发生联系，都通过 Presenter 传递。
3. View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里。

和MVC模式一样，用户对View的操作都会从View交移给Presenter。Presenter同样的会执行相应的业务逻辑，并且对Model进行相应的操作；而这时候Model也是通过观察者模式把自己变更的消息传递出去，但是是传给Presenter而不是View。Presenter获取到Model变更的消息以后，**通过View提供的接口更新界面**。

在MVC模式中，Activity应该是属于View这一层。而实质上，它既承担了View，同时也包含一些Controller的东西在里面。这对于开发与维护来说不太友好，耦合度大高了。把Activity的View和Controller抽离出来就变成了View和Presenter，这就是MVP模式。

## MVP（Passive View）关键点

1. View不再负责同步的逻辑，而是由Presenter负责。Presenter中既有业务逻辑也有同步逻辑。
2. View需要提供操作界面的接口给Presenter进行调用。（关键）

对比在MVC中，Controller是不能操作View的，View也没有提供相应的接口；而在MVP当中，Presenter可以操作View，View需要提供一组对界面操作的接口给Presenter进行调用；Model仍然通过事件广播自己的变更，但由Presenter监听而不是View。

## MVP（Passive View）的优缺点

### 优点

1. 便于测试。Presenter对View是通过接口进行，在对Presenter进行不依赖UI环境的单元测试的时候。可以通过Mock一个View对象，这个对象只需要实现了View的接口即可。然后依赖注入到Presenter中，单元测试的时候就可以完整的测试Presenter业务逻辑的正确性。。
2. View可以进行组件化。在MVP当中，View不依赖Model。这样就可以让View从特定的业务场景中脱离出来，可以说View可以做到对业务逻辑完全无知。它只需要提供一系列接口提供给上层操作。这样就可以做到高度可复用的View组件。
### 缺点

1. Presenter中除了业务逻辑以外，还有大量的View->Model，Model->View的手动同步逻辑，造成Presenter比较笨重，维护起来比较困难。

## MVP（Supervising Controller）
上面讲的是MVP的Passive View模式，该模式下View非常Passive，它几乎什么都不知道，Presenter让它干什么它就干什么。而Supervising Controller模式中，Presenter会把一部分简单的同步逻辑交给View自己去做，Presenter只负责比较复杂的、高层次的UI操作，所以可以把它看成一个Supervising Controller。

Supervising Controller模式下的依赖和调用关系：

![](https://i.imgur.com/klYGsZh.jpg)

# MVVM
MVVM的依赖关系和MVP依赖，只不过是把P换成了VM。

![](https://i.imgur.com/dBgDcqp.jpg)

下图展示了iOS下的MVC是如何拆分成MVVM的：

![](https://i.imgur.com/HJ5B0VL.gif)

## MVVM的调用关系
MVVM的调用关系和MVP一样。但是，在ViewModel当中会有一个叫Binder，或者是Data-binding engine的东西。以前全部由Presenter负责的View和Model之间数据同步操作交由给Binder处理。你只需要在View的模版语法当中，指令式地声明View上的显示的内容是和Model的哪一块数据绑定的。当ViewModel对进行Model更新的时候，Binder会自动把数据更新到View上去，当用户对View进行操作（例如表单输入），Binder也会自动把数据更新到Model上去。这种方式称为：Two-way data-binding，双向数据绑定。可以简单而不恰当地理解为一个模版引擎，但是会根据数据变更实时渲染。
![](https://i.imgur.com/ySqPoFu.jpg)

也就是说，MVVM把View和Model的同步逻辑自动化了。以前Presenter负责的View和Model同步不再手动地进行操作，而是交由框架所提供的Binder进行负责。只需要告诉Binder，View显示的数据对应的是Model哪一部分即可。

这里有一个JavaScript MVVM的例子，因为MVVM需要Binder引擎。所以例子中使用了一个MVVM的库：Vue.js。

## MVVM的优缺点
### 优点：

1. 提高可维护性。解决了MVP大量的手动View和Model同步的问题，提供双向绑定机制。提高了代码的可维护性。
2. 简化测试。因为同步逻辑是交由Binder做的，View跟着Model同时变更，所以只需要保证Model的正确性，View就正确。大大减少了对View同步更新的测试。
### 缺点：

1. 过于简单的图形界面不适用，或说牛刀杀鸡。
2. 对于大型的图形应用程序，视图状态较多，ViewModel的构建和维护的成本都会比较高。
3. 数据绑定的声明是指令式地写在View的模版当中的，这些内容是没办法去打断点debug的。

关于MVVN与DOM对比的详解，可以参考[廖雪峰的博客](https://www.liaoxuefeng.com/wiki/1022910821149312/1108898947791072 "廖雪峰的博客")

而深入解析的博客链接可参考[西木优子的博客](https://www.jianshu.com/p/2ad25e2769b5 "西木柚子的博客")