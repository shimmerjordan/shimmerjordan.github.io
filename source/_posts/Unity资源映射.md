---
title: Unity资源映射
date: 2024-8-22 19:30:12
tags: 
	- Unity
	- Asset
categories: Unity
id: unity-asset-resources
---

Unity工程（空工程）包含`Assets`, `Library`, `Packages`, `ProjectSettings`，其作用分别是：
- **Assets**  
    Unity工程实际的资源目录，所有项目用到的资源、代码、配置、库等原始资源只有放置在这个文件夹才会被Unity认可和处理。
- **Library**  
    存放Unity处理完毕的资源，大部分的资源导入到Assets目录之后，还需要通过Unity转化成Unity认可的文件，转化后的文件会存储在这个目录。会在Unity打开时自动检查和转化，不需要备份。如果涉及到多平台开发，建议分开存放目录，以免每次切平台转化Library耗时过久。
- **Packages**  
    2018以后新增的目录，用于管理Unity分离的packages组件。
- **ProjectSettings**  
    用于存放Unity的各种项目设定。

<!--more-->

至于AssetBundle，[官评](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)：**AssetBundles是一个包含了特殊平台、非代码形式Assets的归档文件。** 这里有几个重要信息，首先它是一个归档文件（即捆绑形式的文件类型），其次它拥有平台的差异性，再次它不包含代码，最后它存储的是Unity的Assets。
# Assets初涉
## 识别与引用

作为资产文件，Assets有非常多的类型。比如：材质球、纹理贴图、音频文件、FBX文件、各种动画、配置或者Clip文件等等。

### 1. Assets和Objects
Assets指Unity的资产，可以意指为Unity的Projects窗口里看到的单个文件（或者文件夹）。而Objects这里我们指的是从UnityEngine.Object继承的对象，它其实是一个可以序列化的数据，用来描述一个特定的资源的实例。它可以代表任何Unity引擎所支持的类型，比如：Mesh、Sprite、AudioClip or AnimationClip等等。

大多数的Objects都是Unity内置支持的，但有两种除外：
- **ScriptableObject**  
    用来提供给开发者进行自定义数据格式的类型。从该类继承的格式，都可以像Unity的原生类型一样进行序列和反序列化，并且可以从Unity的Inspector窗口进行操作。
- **MonoBehaviour**  
    提供了一个指向MonoScript的转换器。MonoScript是一个Unity内部的数据类型，它不是可执行代码，但是会在特定的命名空间和程序集下，保持对某个或者特殊脚本的引用。
    
Assets和Objects之间是一对多的关系，比如一个Prefab我们可以认为是一个Asset，但是这个Prefab里可以包含很多个Objects，比如：如果是一个UGUI的Prefab，里面就可能会有很多个Text、Button、Image等组件。

### 2. File GUIDs和Local IDs
熟悉Unity的人都知道，UnityEngine.Objects之间是可以互相引用的。这就会存在一个问题，这些互相引用的Objects有可能是在同一个Asset里，也有可能是在不同的Assets里。

> 比如：UGUI的一个Image需要引用一张Sprite Atlas里的Sprite，这就要求Unity必须有健壮的资源标识，能稳定地处理不同资源的引用关系。除此之外，Unity还必须考虑这些资源标识应该与平台无关，不能让开发者在切换平台的时候还需要关注资源的引用关系，毕竟它自己是一个跨平台部署的引擎。

基于这些特定的需求，Unity把序列化拆分成两个表达部分。第一部分叫做File GUID，标识这个资产的位置，这个GUID是由Unity根据内部算法自动生成的，并且存放在原始文件的同目录、同名但是后缀为`.meta`的文件里。这也是为什么变更`.asset`文件之后需要和`.meta`文件一同提交仓库。

**这里需要注意几个点:**
- 第一次导入资源的时候Unity会自动生成。
- 在Unity的面板里移动位置，Unity会自动帮你同步.meta文件。
- 在Unity打开的情况下，单独删除.meta，Unity可以确保重新生成的GUID和现有的一样。
- 在Unity关闭的情况下，移动或者删除.meta文件，Unity无法恢复到原有的GUID，也就是说引用会丢失。

确定了资产文件之后，还需要一个Local IDs来表示当前的Objects在资产里的唯一标识。File GUID确保了资产在整个Unity工程里唯一，Local ID确保Objects在资产里唯一，这样就可以通过二者的组合去快速找到对应的引用。

Unity还在内部维护了一张资产GUID和路径的映射表，每当有新的资源进入工程，或者删除了某些资源，又或者调整了资源路径，Unity的编辑器都会自动修改这张映射表以便正确地记录资产位置。所以如果.meta文件丢失或者重新生成了不一样的GUID，Unity就会丢失引用，在工程内的表现就是某个脚本显示“Missing”，或者某些贴图材质的丢失导致场景出现粉红色。

### Library存储资源
前面提到非Unity支持的格式，需要由导入器进行资源转换，这也涉及到了File GUID。之所以需要对资源转换和存储，也是为了方便下一次启动的时候不需要再处理资源，比较每次导入资源是巨耗时的操作。简单来讲，所有的转换结果都会存储在`Library/metadata/`目录下，以File GUID的前两位进行命名的文件夹里。比如：
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/Unity%E8%B5%84%E6%BA%90%E6%98%A0%E5%B0%84/image-20240825161855237.png)
**注意：原生支持的Assets资源也会有同样的存储过程，只是不需要再用导入器进行转化而已。**

### 4. Instance ID
File GUID和Local ID确实已经能够在编辑器模式下帮助Unity完成它的规划了，与平台无关、快速定位和维护资源位置以及引用关系。但若投入到运行时，则还有比较大的性能问题。也就是说运行时还是需要一个表现更好的系统。于是Unity又弄了一套缓存（还记得前面那套缓存嘛？是用来记录GUID和文件的路径关系的）。

PersistentManager用来把File GUIDs和Local IDs转化为一个简单的、Session唯一的整数，这些整数就是Instance ID。Instance ID很简单，就是一个递增的整数，每当有新对象需要在缓存里注册的时候，简单的递增就行。

简单来说：PersistentManager会维护Instance ID和File GUID、Local ID的映射关系，定位Object源数据的位置以及维护内存中（如果有）Object的实例。只要系统解析到一个Instance ID，就能快速找到代表这个Instance ID的已加载的对象。如果Object没有被加载，File GUID和Local ID也可以快速地定位到指定的Asset资源从而即时进行资源加载。

具体可以参考来自腾讯GAD的：[《程序丨入门必看：Unity资源加载及管理》](https://mp.weixin.qq.com/s/0XFQt8LmqoTxxst_kKDMjw?)

# 资源生命周期

上文介绍了Unity的Asset在编辑器和运行时的关联和引用关系。接下来我们还要关注一下这些资源的生命周期，以及在内存中的管理方式，以便能更好地管理加载时长和内存占用。

**1、Object加载**  
当Unity的应用程序启动的时候，PersistentManager的缓存系统会对项目立刻要用到的数据（比如：启动场景里的这些或者它的依赖项），以及所有包含在Resources目录的Objects进行初始化。如果在运行时导入了Asset或者从AssetBundles（比如：远程下载下来的）加载Object都会产生新的Instance ID。

另外Object在满足下列条件的情况时会自动加载，比如：
- 某个Object的Instance ID被间接引用了
- Object当前没有被加载进内存
- 可以定位到Object的源位置（File GUID 和 Local ID）

另外，如果File GUID和Local ID没有Instance ID，或者有Instance ID，但是对应的Objects已经被卸载了，并且这个Instance ID引用了无效的File GUID和Local ID，那么这个Objects的引用会被保留，但是实际Objects不会被加载。在Unity的编辑器里会显示为：“（Missing）”引用，而在运行时根据Objects类型不一样，有可能会是空指针，有可能会丢失网格或者纹理贴图导致场景或者物体显示粉红色。

**2、Object卸载**  
除了加载之外，Objects会在一些特定情况下被卸载。

（1）当没有使用的Asset在执行清理的时候，会自动卸载对应的Object。一般是由切场景或者手动调用了`Resources.UnloadUnusedAssets`的API的时候触发的。但是这个过程只会卸载那些没有任何引用的Objects。

（2）从Resources目录下加载的Objects可以通过调用`Resources.UnloadAsset` API进行显式的卸载。但这些Objects的Instance ID会保持有效，并且仍然会包含有效的File GUID 和LocalID。当任何Mono的变量或者其它Objects持有了被`Resources.UnloadAsset`卸载的Objects的引用之后，这个Object在被直接或者间接引用之后会马上被加载。

（3）从AssetBundles里得到的Objects在执行了`AssetBundle.Unload(true)` API之后，会立刻自动的被卸载，并且这会立刻让这些Objects的File GUID、Local ID以及Instance ID立马失效。任何试图访问它的操作都会触发一个NullReferenceException。但如果调用的是`AssetBundle.Unload(false)` API，那么生命周期内的Objects不会随着AssetBundle一起被销毁，但是Unity会中断File GUID 、Local ID和对应Object的Instance IDs之间的联系，也就是说，如果这些Objects在未来的某些时候被销毁了，那么当再次对这些Objects进行引用的时候，是没法再自动进行重加载的。

另外，如果Objects中断了它和源AssetBundle的联系之后，那么再次加载相同Asset，Unity也不会复用先前加载的Objects，而是会重新创建Instance ID，也就是说内存里会有多份冗余的资源。

# Resource浅析

前面我们也说了，Unity处理资源就2种方式，Resources和AssetBundles。一般来说，正式的商业项目，对外发布资源的时候都是使用AssetBundles的方式进行。但是AssetBundles的方式有很多缺点，比如：
- 无法直观地看到包内的资源情况。
- 异步加载，需要写比较繁琐的回调处理。
- 调试的时候，无法通过Hierarchy直接定位到资源。
- 使用之前需要花费时间进行打包，尤其是在开发的时候，调整资源频繁，如果忘记打包可能导致Bug。

所以Resources能被广泛地使用，很大原因是它的使用非常简单，并且是同步加载。但真相是什么样的呢？

## 官方答复

Unity官方对于Resources使用的建议是：不！要！使！用！它！

Yes，出于几点原因，Unity并不希望大家**过度使用**Resources，是因为：
- Resources内的资源会增加应用程序的启动时间和构建时长。
- Resources内的资源无法增量更新，这是现在手机游戏开发的致命点。

所以官方建议，使用AssetBundles。

一般情况是我们把需要随包构建的资源放在Resources下，结合AssetBundle进行热更新。这种思路有多种实现方法，魂斗罗而言，是把所有资源全部一股脑塞进Resources，然后在构建脚本中进行AB打包和Resources分拣。

## 哪些情况Resources

Resources有它的致命性缺点，但是存在即合理。它还是有它的一些使用场景的，比如：
- 某些资源是项目整个生命周期都必须要用的。
- 有些很重要，但是却不怎么占内存的。
- 不怎么需要变化，并且不需要进行平台差异化处理的。
- 用于系统启动时候最小引导的。

另外还有一种情况就是，当你要快速完成Demo，可以使用Resources可以节省时间。但是要注意，一旦你决定要把既定的工程用于正式生产的时候，请一定把它用AssetBundles重写。

## Resources的序列化

如前文所说，构建项目的时候，所有的Resources目录下的文件会被合并为一个序列化文件。该文件会有自己的metadata信息和索引信息。内部用红黑树实现资源查找，用于索引相应的File GUID和Local ID，并且它还要记录在序列化文件中的偏移量。

官方的实际测试数据，一个拥有10000个Assets的Resources目录，在低端移动设备上的初始化需要5-10秒甚至更长。但其实，这些Assets并不会在一开始就全部用到。

## 开发中的替代方案

前面把Resources和AssetBundles不便的地方都说了一遍，那么有没有一种方案，既可以在开发时候快速加载，又没有Resources那些缺点呢？有的，其实就是AssetDatabase。

这是一个Unity在编辑器模式下的资源加载类，它提供了常规资源的Create、Delete、Save、Load等常用接口，并且是同步加载。所以我们只需要自己写一个资源管理类，用宏区分Editor模式，在Editor直接使用AssetDatabase进行资源加载，然后模拟一个异步回调，让它看起来跟AssetBundles 加载流程相似，然后在非Editor模式下，调用正常的AssetBundles加载就可以了。
