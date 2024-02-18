---
title: Java Dynamic Proxy Introduction
date: 2022-04-01 19:41:34
tags:
  - Java
  - Proxy
categories:
  - Java
id: ynamic-proxy
---

# 目标：

1. 理解动态代理：基于反射机制

2. 什么是动态代理，动态代理能做什么

<!-- more -->

# 情景

> 代理模式：为其它对象提供一种代理以控制对这个对象的访问。换句话说，使用代理对象，是为了在不修改目标对象的基础上，增强业务逻辑。

客户类真正**想要访问**的对象是目标对象，但客户类真正**可以访问**的对象是代理对象。

1. 在开发中会有这样的情况，有A类，本来是调用B类的方法完成某个功能，但是B不让A直接调用。因此需要在Ａ和B之间创建一个C代理，使得Ａ能够通过C访问B。

2. 一个工程如果依赖另一个工程给的接口，但是另一个工程的接口不稳定，经常变更协议，就可以使用一个代理，接口变更时，只需要修改代理，不需要一一修改业务代码。从这个意义上说，所有调外界的接口，我们都可以这么做，不让外界的代码对我们的代码有侵入，这叫防御式编程。

# JDK动态代理

jdk动态代理是jre提供给我们的类库，可以直接使用，不依赖第三方。先看下jdk动态代理的使用代码，再理解原理。

## 案例

1. 首先有个`Star`接口类，有`sing`、`dance`两个功能：
   
   ```java
   package proxy; 
   public interface Star{
       String sing(String name);    
       String dance(String name);
   }
   ```

2. 再有一个`Star`实现类`LiuDeHua`：
   
   ```java
   package proxy; 
   public class LiuDeHua implements Star{   
       @Override
       public String sing(String name){
            System.out.println("给我一杯忘情水");
           return "唱完" ;
       }
   
       @Override
       public String dance(String name){
           System.out.println("开心的马骝");
           return "跳完" ;
       }
   }
   ```

3. 明星演出前需要有人收钱，由于要准备演出，自己不做这个工作，一般交给一个经纪人。便于理解，它的名字以`Proxy`结尾，但他不是代理类，原因是它没有实现我们的明星接口，无法对外服务，它仅仅是一个**wrapper**。
   
   ```java
   package proxy;
   
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   
   public class StarProxy implements InvocationHandler{
       // 目标类，也就是被代理对象
       private Object target;   
       public void setTarget(Object target){
           this.target = target;
       }
   
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
           // 这里可以做增强
           System.out.println("收钱");        
           Object result = method.invoke(target, args);        
           return result;
       }
   
       // 生成代理类
       public Object CreatProxyedObj() {
           return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
       }   
   }
   ```

上述例子中，方法`CreatProxyedObj`返回的对象才是我们的代理类，它需要三个参数，前两个参数的意思是在同一个`classloader`下通过接口创建出一个对象，该对象需要一个属性，也就是**第三个参数**，它是一个`InvocationHandler`。需要注意的是这个`CreatProxyedObj`方法不一定非得在我们的`StarProxy`类中，往往放在一个工厂类中。

上述代理的代码使用过程一般如下：

1. new一个目标对象

2. new一个`InvocationHandler`，将目标对象`set`进去

3. 通过`CreatProxyedObj`创建代理对象，强转为目标对象的接口类型即可使用，实际上生成的代理对象实现了目标接口。
   
   ```java
       Star ldh = new LiuDeHua();
       StarProxy proxy = new StarProxy();
       proxy.setTarget(ldh); 
       Object obj = proxy.CreatProxyedObj();
       Star star = (Star)obj;
   ```

`Proxy`（jdk类库提供）根据B的接口生成一个实现类，我们称之为C，它就是动态代理类（该类型是 `$Proxy+数字` 的“新的类型”）。

生成过程是：由于拿到了接口，便可以获知接口的所有信息（主要是方法的定义），也就能声明一个新的类型去实现该接口的所有方法。这些方法显然都是“虚”的，它调用另一个对象的方法。当然这个被调用的对象不能是对象B，如果是对象B，我们就没法增强了，等于饶了一圈又回来了。

所以它调用的是B的包装类，这个包装类需要我们来实现，但是jdk给出了约束，它必须实现`InvocationHandler`，上述例子中就是`StarProxy`， 这个接口里面有个方法，它是所有`Target`的**所有方法**的调用入口（`invoke`），**调用之前我们可以加自己的代码增强**。

再看下我们的实现，我们在`InvocationHandler`里调用了对象B（`target`）的方法，调用之前增强了B的方法。

```java
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
        // 这里增强
        System.out.println("收钱");   
        Object result = method.invoke(target, args);        
        return result;
    }
```

所以可以这么认为：C代理了`InvocationHandler`，`InvocationHandler`代理了我们的类B，**两级代理**。

## 小结

整个JDK动态代理的秘密也就这些。简单一句话，动态代理就是要生成一个包装类对象，由于代理的**对象是动态**的，所以叫动态代理。

由于我们需要增强，这个增强是需要留给开发人员开发代码的，因此代理类不能直接包含被代理对象，而是一个`InvocationHandler`，该`InvocationHandler`包含被代理对象，并负责分发请求给被代理对象，分发前后均可以做增强。从原理可以看出，JDK动态代理是“对象”的代理。

## 动态代理原理

下面看下动态代理类到底如何调用的`InvocationHandler`的，为什么`InvocationHandler`的一个`invoke`方法能为分发`target`的**所有方法**。C中的部分代码示例如下，通过反编译生成后的代码查看，摘自[JDK 动态代理（AOP）使用及实现原理分析](https://blog.csdn.net/jiankunking/article/details/52143504)。`Proxy`创造的C是自己（`Proxy`）的子类，且实现了B的接口，一般都是这么修饰的：

```java
public final class XXX extends Proxy implements XXX
```

### 方法代码如下：

```java
public final String SayHello(String paramString){
    try{
        return (String)this.h.invoke(this, m4, new Object[] { paramString });
    }
    catch (Error|RuntimeException localError)                          {
        throw localError;
    }
    catch (Throwable localThrowable){
        throw new UndeclaredThrowableException(localThrowable);
    }
```

可以看到，C中的方法全部通过调用`h`实现，其中`h`就是`InvocationHandler`，是我们在生成C时传递的第三个参数。这里还有个关键就是`SayHello`方法（业务方法）跟调用`invoke`方法时传递的参数`m4`一定要是一一对应的。但是这些对我们来说都是透明的，由`Proxy`在`newProxyInstance`时保证的。留心看到C在`invoke``时把自己`this`传递了过去，`InvocationHandler`的``invoke`的第一个方法也就是我们的动态代理实例类，业务上有需要就可以使用它。（所以千万不要在`invoke`方法里把请求分发给第一个参数，否则很明显就死循环了）

### C类中有B中所有方法的成员变量

```java
  private static Method m1;
  private static Method m3;
  private static Method m4;
  private static Method m2;
  private static Method m0;
```

这些变量在static静态代码块初始化，这些变量是在调用`invocationhander`时必要的入参，也让我们依稀看到`Proxy`在生成C时留下的痕迹。

```java
static{
    try{
        m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
        m3 = Class.forName("jiankunking.Subject").getMethod("SayGoodBye", new Class[0]);
        m4 = Class.forName("jiankunking.Subject").getMethod("SayHello", new Class[] { Class.forName("java.lang.String") });
        m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
        m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        return;
    }
    catch (NoSuchMethodException localNoSuchMethodException){
        throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
    }
    catch (ClassNotFoundException localClassNotFoundException){
        throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
    }
  }
```

# CGLIB动态代理

## 案例

我们了解到，“代理”的目的是构造一个和被代理的对象有同样行为的对象，一个对象的行为是在类中定义的，对象只是类的实例。所以构造代理，**不一定非得通过持有、包装对象**这一种方式。

通过“继承”可以**继承父类**所有的公开方法，然后可以重写这些方法，在重写时对这些方法增强，这就是cglib的思想。根据里氏代换原则（LSP），父类需要出现的地方，子类可以出现，所以cglib实现的代理也是可以被正常使用的。

先看代码：

```java
package proxy;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class CglibProxy implements MethodInterceptor{
    // 根据一个类型产生代理类，此方法不要求一定放在MethodInterceptor中
    public Object CreatProxyedObj(Class<?> clazz){
        Enhancer enhancer = new Enhancer();     
        enhancer.setSuperclass(clazz);        
        enhancer.setCallback(this);        
        return enhancer.create();
    }

    @Override
    public Object intercept(Object arg0, Method arg1, Object[] arg2, MethodProxy arg3) throws Throwable{
        // 这里增强
        System.out.println("收钱");        
        return arg3.invokeSuper(arg0, arg2);
    } 
}
```

从代码可以看出，它和jdk动态代理有所不同，对外表现上看`CreatProxyedObj`，它只需要一个类型`clazz`就可以产生一个代理对象， 所以说是“类的代理”，且创造的对象通过打印类型发现也是一个新的类型。不同于jdk动态代理，**jdk动态代理要求对象必须实现接口（三个参数的第二个参数），cglib对此没有要求。**

cglib的原理是这样，它生成一个继承B的类型C（代理类），这个代理类持有一个`MethodInterceptor`，我们`setCallback`时传入的。 C重写**所有**B中的方法（方法名一致），然后在C中，构建名叫`"CGLIB"+"父类方法名"`的方法（下面叫cglib方法，所有非`private`的方法都会被构建），方法体里只有一句话`super.方法名()`，可以简单的认为保持了对父类方法的一个引用，方便调用。

这样的话，C中就有了重写方法、cglib方法、父类方法（不可见），还有一个统一的拦截方法（增强方法`intercept`）。其中重写方法和cglib方法肯定是有映射关系的。

C的重写方法是外界调用的入口（LSP原则），它调用`MethodInterceptor`的`intercept`方法，调用时会传递四个参数，第一个参数传递的是`this`，代表代理类本身，第二个参数标示拦截的方法，第三个参数是入参，第四个参数是cglib方法，`intercept`方法完成增强后，我们调用cglib方法间接调用父类方法完成整个方法链的调用。

## Tip

1. `intercept`的四个参数，为什么我们使用的是`arg3`而不是`arg1`?
   
   因为如果我们通过反射`arg1.invoke(arg0, ...)`这种方式是无法调用到父类的方法的，子类有方法重写，隐藏了父类的方法，父类的方法已经不可见，如果硬调`arg1.invoke(arg0, ...)`很明显会死循环。
   
   所以调用的是cglib开头的方法，但是，我们使用`arg3`也不是简单的`invoke`，而是用的`invokeSuper`方法，这是因为cglib采用了*fastclass*机制，不仅巧妙的避开了调不到父类方法的问题，还加速了方法的调用。
   
   *fastclass*基本原理是，给每个方法编号，通过编号找到方法执行。避免了通过反射调用。

## 小结

对比JDK动态代理，cglib依然需要一个第三者分发请求。只不过jdk动态代理分发给了目标对象，cglib最终分发给了自己，通过给method编号完成调用。cglib是继承的极致发挥，本身还是很简单的，只是*fastclass*需要另行理解。

# 测试

```java
public static void main(String[] args) {
    int times = 1000000;

    Star ldh = new LiuDeHua();
    StarProxy proxy = new StarProxy();
    proxy.setTarget(ldh);

    long time1 = System.currentTimeMillis();
    Star star = (Star)proxy.CreatProxyedObj();
    long time2 = System.currentTimeMillis();
    System.out.println("jdk创建时间：" + (time2 - time1));

    CglibProxy proxy2 = new CglibProxy();
    long time5 = System.currentTimeMillis();
    Star star2 = (Star)proxy2.CreatProxyedObj(LiuDeHua.class);
    long time6 = System.currentTimeMillis();
    System.out.println("cglib创建时间：" + (time6 - time5));

    long time3 = System.currentTimeMillis();
    for (int i = 1; i <= times; i++) {
        star.sing("ss");
        star.dance("ss");
    }
    long time4 = System.currentTimeMillis();
    System.out.println("jdk执行时间" + (time4 - time3));

    long time7 = System.currentTimeMillis();
    for (int i = 1; i <= times; i++) {
        star2.sing("ss");

        star2.dance("ss");
    }

    long time8 = System.currentTimeMillis();
    System.out.println("cglib执行时间" + (time8 - time7));   
}
```

经测试，jdk**创建**对象的速度远大于cglib，这是由于cglib创建对象时需要操作字节码。cglib**执行**速度略大于jdk，所以比较适合单例模式。

另外由于CGLIB的大部分类是直接对Java字节码进行操作，这样生成的类会在Java的永久堆中。如果动态代理操作过多，容易造成**永久堆满**，触发`OutOfMemory`异常。

Spring默认使用jdk动态代理，如果类没有接口，则使用cglib。
