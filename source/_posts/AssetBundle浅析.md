---
title: AssetBundle浅析
date: 2024-8-23 19:30:12
tags: 
	- Unity
	- Assetbundle
categories: Unity
id: assetbundle-intro
---

**AssetBundle系统提供了一种压缩文件的格式，可以把一个到多个文件进行索引和序列化。**

Unity项目在交付安装之后，会通过AssetBundle对不包含代码的资源进行更新。这就允许开发人员先提交一个小的应用程序包，将运行时内存压力降到最低，并有选择地加载针对不同终端用户设备优化后的内容。

<!--more-->

# AssetBundle原理
## AssetBundle结构
总的来说，AssetBundle就像传统的压缩包一样，由两个部分组成：包头和数据段。

包头包含有关AssetBundle的信息，比如标识符、压缩类型和内容清单。清单是一个以Objects name为键的查找表。每个条目都提供一个字节索引，用来指示该Objects在AssetBundle数据段的位置。在大多数平台上，这个查找表是用平衡搜索树实现的。具体来说，Windows和OSX派生平台（包括iOS）都采用了红黑树。因此，构建清单所需的时间会随着AssetBundle中Assets的数量增加而线性增加。

数据段包含通过序列化AssetBundle中的Assets而生成的原始数据。如果指定LZMA为压缩方案，则对所有序列化Assets后的完整字节数组进行压缩。如果指定了LZ4，则单独压缩单独Assets的字节。如果不使用压缩，数据段将保持为原始字节流。

> 在Unity 5.3之前，是无法对AssetBundle中单独Objects进行压缩的。因此，如果在5.3之前的Unity版本被要求从压缩的AssetBundle中读取一个或多个对象时，Unity必须解压整个AssetBundle。通常，Unity会缓存AssetBundle的解压缩副本，以提高在相同AssetBundle上的后续加载请求的加载性能。

## 加载AssetBundle

AssetBundles可以通过四个不同的API进行加载。但受限于两个标准，这四个API的行为是不同的。2个标准如下：
- AssetBundles的压缩方式：LZMA、LZ4、还是未压缩的。
- AssetBundles的加载平台。

而四个API分别是：
- `AssetBundle.LoadFromMemory(Async optional)`
- `AssetBundle.LoadFromFile(Async optional)`
- UnityWebRequest's `DownloadHandlerAssetBundle`
- `WWW.LoadFromCacheOrDownload` (on Unity 5.6 or older)

### 4个API的区别

**AssetBundle.LoadFromMemory(Async)**  
Unity的建议是——**不要使用这个API**。

`LoadFromMemory(Async)` 是从托管代码的字节数组里加载AssetBundle。也就是说你要提前用其它的方式将资源的二进制数组加入到内存中。然后该接口会将源数据从托管代码字节数组复制到新分配的、连续的本机内存块中。

但如果AssetBundle使用了LZMA压缩类型，它将在复制时解压AssetBundle。而未压缩和LZ4压缩类型的AssetBundle将逐字节的完整复制。

之所以不建议使用该API是因为，此API消耗的最大内存量将至少是AssetBundle的两倍：本机内存中的一个副本，和`LoadFromMemory(Async)`从托管字节数组中复制的一个副本。

因此，从通过此API创建的AssetBundle加载的资产将在内存中冗余三次：一次在托管代码字节数组中，一次在AssetBundle的栈内存副本中，第三次在GPU或系统内存中，用于Asset本身。

注意：在Unity 5.3.3之前，这个API被称为`AssetBundle.CreateFromMemory`。但功能没有改变。

**AssetBundle.LoadFromFile(Async)**  
LoadFromFile是一种高效的API，用于从本地存储（如硬盘或SD卡）加载未压缩或LZ4压缩格式的AssetBundle。

在桌面独立平台、控制台和移动平台上，API将只加载AssetBundle的头部，并将剩余的数据留在磁盘上。

AssetBundle的Objects会按需加载，比如：加载方法（例如：AssetBundle.Load）被调用或其InstanceID被间接引用的时候。在这种情况下，不会消耗过多的内存。

但在Editor环境下，API还是会把整个AssetBundle加载到内存中，就像读取磁盘上的字节和使用`AssetBundle.LoadFromMemoryAsync`一样。

如果在Editor中对项目进行了分析，此API可能会导致在AssetBundle加载期间出现内存尖峰。但这不应影响设备上的性能，在做优化之前，这些尖峰应该在设备上重新再测试一遍。

要注意，这个API只针对未压缩或LZ4压缩格式，因为前面说过了，如果使用LZMA压缩，它是针对整个生成后的数据包进行压缩的，所以在未解压之前是无法拿到AssetBundle的头信息的。

注意：这里曾经有过一个历史遗留问题，即在Unity 5.3或更老版本的Android设备上，当试图从Streaming Assets路径加载AssetBundles时，此API将失败。这个问题已在Unity 5.4中解决。

在Unity 5.3之前，这个API被称为`AssetBundle.CreateFromFile`。其功能没有改变。

**AssetBundleDownloadHandler**  
`DownloadHandlerAssetBundle`的操作是通过UnityWebRequest的API来完成的。

UnityWebRequest API允许开发人员精确地指定Unity应如何处理下载的数据，并允许开发人员消除不必要的内存使用。使用UnityWebRequest下载AssetBundle的最简单方法是调用`UnityWebRequest.GetAssetBundle`。

就实战项目而言，最有意思的类是`DownloadHandlerAssetBundle`。它使用工作线程，将下载的数据流存储到一个固定大小的缓冲区中，然后根据下载处理程序的配置方式将缓冲数据放到临时存储或AssetBundle缓存中。

所有这些操作都发生在非托管代码中，消除了增加堆内存的风险。此外，该下载处理程序并不会保留所有下载字节的栈内存副本，从而进一步减少了下载AssetBundle的内存开销。

LZMA压缩的AssetBundles将在下载和缓存的时候更改为LZ4压缩。这个可以通过设置`Caching.CompressionEnable`属性来更改。

如果将缓存信息提供给UnityWebRequest对象，一旦有请求的AssetBundle已经存在于Unity的缓存中，那么AssetBundle将立即可用，并且此API的行为将会与`AssetBundle.LoadFromFile`相同操作。

在Unity 5.6之前，UnityWebRequest系统使用了一个固定的工作线程池和一个内部作业系统来防止过多的并发下载，并且线程池的大小是不可配置的。在Unity 5.6中，这些安全措施已经被删除，以便适应更现代化的硬件，并允许更快地访问HTTP响应代码和报头。

**WWW.LoadFromCacheOrDownload**  
这是一个很古老的API了，从Unity 2017.1开始，就只是简单地包装了UnityWebRequest。因此，使用Unity 2017.1或更高版本的开发者应该直接使用UnityWebRequest来工作。Unity已经放弃了对改接口的维护，并可能在未来的某个版本中移除。

所以下面说的这些内容只适合于Unity 5.6或更老的版本。

`WWW.LoadFromCacheOrDownload`允许从远程服务器和本地存储加载对象。也可以通过文件URL从本地存储加载文件。如果AssetBundle存在于Unity Cache中，则此API的行为将与`AssetBundle.LoadFromFile`完全相同。

如果AssetBundle尚未缓存，则`WWW.LoadFromCacheOrDownload`会将从它的源文件读取AssetBundle。如果AssetBundle被压缩过，它会使用工作线程进行解压缩并写入缓存中。否则，它将通过工作线程直接写入缓存。

在缓存AssetBundle之后，`WWW.LoadFromCacheOrDownload`将从缓存的、解压缩的AssetBundle加载头信息。然后，和`AssetBundle.LoadFromFile`加载AssetBundle行为相同。

此缓存会在`WWW.LoadFromCacheOrDownload`和UnityWebRequest之间共享。一个API下载的任何AssetBundle也可以通过另一个API获得。

虽然数据将通过固定大小的缓冲区解压缩并写入缓存，但WWW对象会在本机内存中保留AssetBundle字节的完整副本。这个额外副本被保留的原因是因为要支持WWW.bytes字节属性。

由于在WWW对象中缓存AssetBundle的字节的内存开销，所以，实际项目开发中AssetBundles应该要保持较少的体积以便减少内存。

与UnityWebRequest不同的是，每次调用这个API都会产生一个新的工作线程。因此，在手机等内存有限的平台上，最好限定一次只能下载一个AssetBundle，以避免内存激增。而在其它平台也要小心创建过多的线程。如果需要下载5个以上的AssetBundles，建议在脚本代码中创建和管理下载队列，以确保只有少数几个AssetBundle同时下载。

**建议**
1. 一般来说，只要有可能，就应该使用`AssetBundle.LoadFromFile`。这个API在速度、磁盘使用和运行时内存使用方面是最有效的。
2. 对于必须下载或热更新AssetBundles的项目，强烈建议对使用Unity 5.3或更高版本的项目使用UnityWebRequest，对于使用Unity 5.2或更老版本的项目使用`WWW.LoadFromCacheOrDownload`。
3. 当使用UnityWebRequest或`WWW.LoadFromCacheOrDownload`时，要确保下载程序代码在加载AssetBundle后正确地调用Dispose。另外，C#的using语句是确保WWW或UnityWebRequest被安全处理的最方便的方法。
4. 对于需要独特的、特定的缓存或下载需求的大项目，可以考虑使用自定义的下载器。编写自定义下载程序是一项重要并且复杂的任务，任何自定义的下载程序都应该与`AssetBundle.LoadFromFile`保持兼容。

## 从AssetBundle中加载Assets

到这里，我们已经能够获得AssetBundles了，那么接下来就是要从AssetBundles里获取Assets。

Unity提供了三个不同的API从AssetBundles加载`UnityEngine.Objects`，这些API都绑定到AssetBundle对象上，并且这些API具有同步和异步变体：
- `LoadAsset (LoadAssetAsync)`
- `LoadAllAssets (LoadAllAssetsAsync)`
- `LoadAssetWithSubAssets (LoadAssetWithSubAssetsAsync)`

并且这些API的同步版本总是比异步版本快至少一个帧（其实是因为异步版本为了确保异步，都至少延迟了1帧），异步加载每帧会加载多个对象，直到它们的时间切片切出。

加载多个独立的`UnityEngine.Objects`时应使用LoadAllAsset。并且只有在需要加载AssetBundle中的大多数或所有对象时，才应该使用它。与其它两个API相比，LoadAllAsset比对LoadAsset的多个单独调用略快一些。因此，如果要加载的Asset数量很大，但如果需要一次性加载不到三分之二的AssetBundle，则要考虑将AssetBundle拆分为多个较小的包，再使用LoadAllAsset。

加载包含多个嵌入式对象的复合Asset时，应使用LoadAssetWithSubAsset，例如嵌入动画的FBX模型或嵌入多个精灵的sprite图集。也就是说，如果需要加载的对象都来自同一Asset，但与许多其它无关对象一起存储在AssetBundle中，则使用此API。

任何其它情况，请使用LoadAsset或LoadAssetAsync。

### 低层级的加载细节  
Object加载是在主线程上执行，但数据从工作线程上的存储中读取。任何不触碰Unity系统中线程敏感部分（脚本、图形）的工作都将在工作线程上转换。例如，VBO将从网格创建，纹理将被解压等等。

从Unity 5.3开始，Object加载就被并行化了。在工作线程上反序列化、处理和集成多个Object。当一个Object完成加载时，它的Awake回调将被调用，该对象的其余部分将在下一个帧中对UnityEngine可用。

同步`AssetBundle.Load`方法将暂停主线程，直到Object加载完成。但它们也会加载时间切片的Object，以便Object集成不会占用太多的毫秒帧时间。应用程序属性设置毫秒数的属性为`Application.backgroundLoadingPriority`。  
- `ThreadPriority.High`：每帧最多50毫秒  
- `ThreadPriority.Normal`：每帧最多10毫秒
- `ThreadPriority.BelowNormal`：每帧最多4毫秒  
- `ThreadPriority.Low`：每帧最多2毫秒

从Unity 5.2开始，加载多个对象的时候，会一直进行直到达到对象加载的帧时间限制为止。假设所有其它因素相等，Asset加载API的异步变体将总是比同步版本花费更长的时间，因为发出异步调用和对象之间有最小的一帧延迟。

### AssetBundle依赖项
根据运行时环境的不同，使用两个不同的API自动跟踪AssetBundles之间的依赖关系。在UnityEditor中，可以通过AssetDatabaseAPI查询AssetBundle依赖项。AssetBundles分配和依赖项可以通过AssetImportAPI访问和更改。在运行时，Unity提供了一个可选的API，通过基于ScriptableObject的AssetBundleManifest API加载在AssetBundle构建过程中生成的依赖信息。

当一个或多个AssetBundle的UnityEngine.Objects引用了一个或者多个其它AssetBundle的UnityEngine.Objects，那么这个AssetBundle就会依赖于另外的AssetBundle。AssetBundles充当由它包含的每个对象的FileGUID和LocalID标识的源数据。

因为一个对象是在其Instance ID第一次被间接引用时加载的，而且由于一个对象在加载其AssetBundle时被分配了一个有效的Instance ID，所以加载AssetBundles的顺序并不重要。相反，在加载对象本身之前，重要的是加载包含对象依赖关系的所有AssetBundles。Unity不会尝试在加载父AssetBundle时自动加载任何子AssetBundle。

**示例**
假设Material A引用Texture B。Material A被打包到AssetBundle1中，Texture B被打包到AssetBundle2中：
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/AssetBundle浅析/image-20240825190138057.png)
在本用例中，AssetBundle2必须在Material A从AssetBundle1中加载之前先加载。这并不意味着AssetBundle 2必须在AssetBundle 1之前加载，或者Texture B必须从AssetBundle 2中显式加载。在将Material A从AssetBundle 1加载之前加载AssetBundle 2就足够了。

简单来说就是AssetBundle之间的加载没有先后，但是Asset的加载有。

有关AssetBundle依赖项的详细信息，请[参阅手册页](https://docs.unity3d.com/Manual/AssetBundles-Dependencies.html?_ga=2.168691873.1408835506.1571651191-1030292064.1564583003)。

### AssetBundle manifests
当使用`BuildPiine.BuildAssetBundles` API执行AssetBundle构建管线时，Unity会序列化一个包含每个AssetBundle依赖项信息的对象。此数据存储在单独的AssetBundle中，其中包含AssetBundleManifest类型的单个对象。

此Asset将存储在与构建AssetBundles的父目录同名的AssetBundle中。如果一个项目将其AssetBundles构建到位于`(Projectroot)/Build/Client/`的文件夹中，那么包含清单的AssetBundle将被保存为`(Projectroot)/build/client/Client.manifest`。

包含Manifest的AssetBundle可以像任何其它AssetBundle一样加载、缓存和卸载。

AssetBundleManifest对象本身提供GetAllAssetBundles API来列出与清单同时构建的所有AssetBundles，以及查询特定AssetBundle的依赖项的两个方法：
- `AssetBundleManifest.GetAllDependencies`返回AssetBundle的所有层次依赖项，其中包括AssetBundle的直接子级、其子级的依赖项等。
- `AssetBundleManifest.GetDirectDependations`只返回AssetBundle的直接子级

请注意，这两个API分配的都是字符串数组。因此，最好是在性能要求不敏感的时候使用。

**建议**  
在多数情况下，最好在玩家进入应用程序的性能关键区域（如主游戏关卡或世界）之前加载尽可能多的所需对象。这在移动平台上尤为重要，因为在移动平台上，访问本地存储的速度很慢，并且在运行时加载和卸载对象会触发垃圾回收。

# AssetBundle实践
这一节讨论下在实际使用AssetBundles的过程中遇到的问题以及解决方案。
## 管理已经加载的Assets

在一些内存敏感的环境中，严格控制加载对象的大小和数量是至关重要的。当对象从活动场景中被移除时，Unity并不会自动卸载对象。Assets的清理工作是在特定时间触发的，当然也可以手动触发（其实就是GC了）。

AssetBundle必须要小心管理。在本地存储（通过Unity缓存或通过`AssetBundle.LoadFromFile`加载的文件）里的文件支持的AssetBundle具有最小的内存开销，很少超过几十K字节。但是，如果存在大量的AssetBundles，这种开销仍然会成为问题。

由于大多数项目允许用户重复体验游戏内容（例如重新打一个关卡），因此了解何时加载或卸载AssetBundle就变得非常重要。如果AssetBundle卸载不当，则会导致内存中的对象产生冗余副本。而在某些情况下，不适当地卸载AssetBundles也会导致不良行为，例如纹理丢失。

在管理Asset和AssetBundle时，最重要的一点是调用`AssetBundle.unload`时的方式，unload参数为true或false。

此API将卸载正在调用的AssetBundle的包头信息。unload参数决定是否也卸载从此AssetBundle实例化的所有对象。如果设置为true，那么从AssetBundle创建的所有对象也将立即卸载！即使它们目前正在活动场景中被引用。

举个例子，假设Material M是从AssetBundle AB加载的，并且假设M当前在活动场景中。
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/AssetBundle浅析/image-20240825190831764.png)

如果调用了`AssetBundle.Unload(True)`，则M将从场景中移除，销毁并卸载。但是，如果调用`AssetBundle.Unload(False)`，则AB的包头信息将被卸载，但M将保持在场景中，并且仍然是可用的。调用`AssetBundle.Unload(False)`破坏了M和AB之间的链接。如果AB稍后再次加载，则AB中包含的对象的新副本将会被加载到内存中。
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/AssetBundle浅析/image-20240825190855794.png)
如果AB稍后再次加载，将会重新加载AssetBundle的头信息的新副本。然而，M不是从这个新的AB拷贝加载的。Unity并没有在AB和M的新副本之间建立任何联系。
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/AssetBundle浅析/image-20240825190909336.png)
如果调用`AssetBundle.LoadAsset()`来重新加载M，Unity将不会将旧的M副本作为为AB中数据的实例。因此，Unity将加载一个新的副本的M，所以此时将会有两个相同的副本M在现场。
![github pic](https://gcore.jsdelivr.net/gh/shimmerjordan/pic_bed@obsidian-assets/AssetBundle浅析/image-20240825190920372.png)
对于大多数项目来说，这样的结果是不可取的。大多数项目应该使用`AssetBundle.Unload(True)`，并采用一种方法来确保对象不被复制。两种常见的方法是：  
1. 在应用程序的生命周期内定义一个合适的节点，并在此期间卸载不需要的AssetBundle，例如在关卡切换或加载屏幕期间。这是最简单和最常见的选择。
2. 维护单个对象的引用计数，并仅当所有组成对象都未使用时才卸载AssetBundles。这允许应用程序在不重复内存的情况下卸载和重新加载单个对象。

如果应用程序必须使用`AssetBundle.Unload(False)`，那么只能通过两种方式卸载各个对象：  
1. 在场景和代码中消除对不需要的对象的所有引用。完成后，调用`Resources.UnusedAsset`。
2. 非附加加载场景。这将销毁当前场景中的所有对象并调用`Resources.UnusedAsset`。

如果项目有明确定义的节点，玩家可以等待对象加载和卸载，例如在游戏模式或关卡之间切换，则可以根据需要卸载尽可能多的对象和加载新对象。

最简单的方法是将项目的离散块打包到场景中，然后将这些场景与所有依赖项一起构建到AssetBundles中。然后，应用程序可以切换到“loading”场景，从而完全卸载包含旧场景的AssetBundles，然后加载包含新场景的AssetBundles。

虽然这是最简单的流程，但却需要很复杂的AssetBundles管理。由于每个项目是不同的，Unity尚没有提供通用的AssetBundles设计模式。

在决定如何将对象分组到AssetBundles时，如果必须同时加载或更新对象，则通常最好将对象捆绑到AssetBundles中。例如，考虑一个角色扮演游戏。个别的地图和裁剪场景可以按场景分组成AssetBundles，但有些对象可能会存在于很多其它场景则不能划分进去。AssetBundles可以用来提供肖像画，游戏中的UI，以及不同的角色模型和纹理。这些稍后需要的Objects和Assets可以分组到第二组AssetBundles中，这些Assets在启动时加载，并在应用程序的生命周期内保持加载状态。

另一个问题可能会出现，如果Unity必须在AssetBundle卸载后从AssetBundle中重新加载一个对象。在这种情况下，重新加载将失败，该对象将以(Missing)对象的形式出现在Unity编辑器的层次结构中。

这主要会发生在Unity失去并试图恢复对其图形上下文的控制时，例如当移动应用程序被挂起或用户锁定他们的PC时。在这种情况下，Unity必须重新上传纹理和渲染到GPU。如果这些Asset的源AssetBundle不可用，则应用程序会将场景中的相关对象呈现为洋红色。

## 发布

客户端发布项目的AssetBundles有两种基本方法：和项目一起安装或安装后再下载它们。

在安装时还是安装后交付AssetBundles，取决于项目将运行的平台的功能和限制。移动项目通常选择先安装后下载，以减少初始安装大小，并保持低于相关平台下载的大小限制（比如苹果商店和谷歌商店会对4G模式下最大能下载的包做限制）。控制台和PC项目通常在初始安装时附带AssetBundles。

良好的架构应该允许项目在安装后再进行资源热更，而不用管AssetBundles最初是如何发布的。有关这方面的更多信息，请参见“[Unity 手册](https://docs.unity3d.com/Manual/AssetBundles-Patching.html?_ga=2.161080382.2009813986.1571904029-1030292064.1564583003)”中的“Patching with AssetBundles”一节。

### 随项目发布
与项目一起发布AssetBundles是最简单的，因为它不需要额外的代码进行下载和管理。项目在安装时包含AssetBundles，有两个主要原因：
- 减少项目构建时间并允许更简单的迭代开发。如果这些AssetBundles不需要和应用程序本身分开更新，可以通过将AssetBundles存储在Streaming Assets中，将AssetBundles包含在应用程序中。请参阅下面的 Streaming Assets部分。
- 发布可更新内容的初始修订版。这通常是为了节省终端用户在最初安装后的时间，或者作为以后修补的基础。Streaming Assets在这种情况下并不理想。但是，如果不选择编写自定义下载和缓存系统，则可以从Streaming Assets将可更新内容的初始修订加载到Unity缓存中（请参阅下面的缓存启动部分）。

### 安装后下载 
将AssetBundles传递到移动设备的最好方法是在应用程序安装后再下载它们。这就允许在安装后再更新游戏内容，而不必强迫用户重新下载整个应用程序。在许多平台上，应用程序二进制文件必须经过昂贵而漫长的重新认证过程。因此，开发一个良好的分离下载系统是至关重要的。

交付AssetBundles最简单的方法是将它们放在某个Web服务器上，并通过UnityWebRequest发布。Unity将自动在本地存储上缓存下载的AssetBundles。如果下载的AssetBundle是LZMA压缩的，那么AssetBundle将以未压缩或重新压缩的形式存储在缓存中，就像LZ 4一样（依赖`Caching.compressionEnabled`设置），以便将来更快地加载。如果下载的包是LZ 4压缩的，AssetBundles将被压缩存储。如果缓存被填满，Unity将从缓存中删除最近使用最少的AssetBundle。

通常建议在允许的情况下使用UnityWebRequest，或者只有在使用Unity 5.2或更老版本时才使用`WWW.LoadFromCacheOrDownload`。只有当内置API的内存消耗、缓存行为或性能对于特定项目是不可接受的时候，或者项目必须运行特定于平台的代码才能满足其需求时，才需要对自定义下载系统进行扩展。

可能会阻碍使用UnityWebRequest或`WWW.LoadFromCacheOrDownload`的情况示例：
- 当需要对AssetBundle缓存进行颗粒度控制时。
- 当项目需要实现自定义压缩策略时。
- 当项目希望使用特定于平台的API来满足某些需求时，例如在不活动时才加载流数据。比如，使用IOS的后台任务API下载数据。
- 当AssetBundles必须在不具备SSL支持条件(如PC)的平台上通过SSL交付时时候。

### 建立缓存  
Unity有一个内置的AssetBundle缓存系统，可以用来缓存通过UnityWebRequest API下载的AssetBundles，该API的重载会接受一个AssetBundle版本号作为参数。这个数字不是存储在AssetBundles里的，也不是由AssetBundles系统生成的。

缓存系统跟踪传递给UnityWebRequest的最后一个版本号。当使用版本号调用此API时，缓存系统通过比较版本号来检查是否存在缓存的AssetBundle。如果这些数字匹配，系统将加载缓存的AssetBundle。如果数字不匹配，或者没有缓存的AssetBundle，那么Unity将下载一个新的副本。此新副本将与新的版本号相关联。

**缓存系统中的AssetBundle只通过它们的文件名来标识**，而不是通过下载它们的完整URL。这意味着具有相同文件名的AssetBundle可以存储在多个不同的位置，例如CDN。只要文件名相同，缓存系统就会将它们识别为相同的AssetBundle。

每个应用程序都需要确定将版本号分给AssetBundles的考量，然后将这些编号传递给UnityWebRequest。数字可能来自某些唯一标识符，例如crc值。请注意，虽然`AssetBundleManifest.GetAssetBundleHash()`也可用于此目的，但我们不建议将此函数用于版本控制，因为它只提供了估计，而不是真正的哈希计算。

# AssetBundle编译过程
## MenuItem

通过调用MenuItem对应的打包函数，对应到AssetBundle下的Build方法，实质上执行的是BuildAssetBundleStage重写了父类IStage的Process方法，主要通过以下步骤启动AB打包：
```csharp          
RestoreCombinedNameIndexPool();
RestoreOutGameCombinedNameIndexPool();
Build();
LinkAssetEntriesToBundleIds();
LinkLevelScenesToBundleIds();
```
其中主要还是`Build()`函数是核心，其他的无非是在做初始化和资源链接

## Build

因为魂斗罗项目比较特殊，项目组在开发工程里就把所有资源全放进了Resource目录，所以在AB构建的时候需要不断建立一些链接来提取需要构建进AB包的资源。除了前面的步骤外，`Build()`函数内也有例如以下的链接函数：

```cs
LinkGameModeToCommonBundleIds();
LinkAssetEntryToCommonBundleIds();
LinkControllerToHallModel();
```

除此之外，还可以用`AssetBundleStatisticsHelper.RecordPackedSize();`记录打包的大小。然后就是关键的调用引擎的AsseBundle构建函数了：

```cs
//如果内存不够的话，统一构建AssetBundle会报out of memory
BuildPipeline.BuildAssetBundles(exportPath, buildMap,
    buildOptions,
    EditorUserBuildSettings.activeBuildTarget);
// 因此采用下面逐个构建的方式，采用下面的方式buildOptions需要加上DontCompile
// Unity2017之后没有DontCompile的选项了，就不用管了
// foreach (var bundle in buildMap)
// {
//      BuildPipeline.BuildAssetBundles(exportPath,
//         new AssetBundleBuild[] { bundle, },
//         buildOptions,
//         EditorUserBuildSettings.activeBuildTarget);
// }
```
函数的最后还会执行一个`GenExtBundleInfo`的函数，主要用于生成AB包的MD5，在后续步骤中进行校验

## BuildAssetBundles

由`public static AssetBundleManifest BuildAssetBundles`实际通过指定平台后的`internal`同名方法再调用`BuildAssetBundlesInternal`启动AB构建的，这个方法在`Editor\Mono\BuildPipeline.bindings.cs`中只做了extern的外部声明，实际实现在`Editor\Src\BuildPipeline\BuildAssetBundle.cpp`中的静态方法。

### BuildAssetBundlesInternal

AB构建的根入口，每一个异常终止的分支都会调用`SendBuildAssetBundlesAnalytics`发送错误报告并返回`NULL`
1. 判断是否支持该平台的构建，不支持就异常处理
2. 因为不支持在Unity运行态时build AB，所以需要先终止，然后这个方法里会用`SetApplicationIsPlaying`相当于抢占锁了
3. 做一些例如License、Variants、BuildOptions是否存在且合法的判断，一样是不合法就异常处理
4. 保存previous平台的一些信息，保证build完AB之后可以顺利切换当前平台（如果需要的话）
5. 通过`CalculateAssetBundlesToBeBuilt`这个**比较复杂的函数**提取需要构建的AB资源
6. 通过`BuildAssetBundleHelper.cpp`中的`CheckSceneAssetBundles`方法检查并标记场景文件
7. 通过之前随AB包打出来的Hash对比现在的预打包的AB包，大小不一就说明有变化需要重新build，同时赋值一些例如压缩方法、额外流式场景之类的buildOptions
8. 判断`GenerateAssemblyTypeInfos`能否正确返回，否则异常处理之
9. 预编译一些例如质量设置等的配置项，注意这里维护了一个`m_ManagersToReset`列表，用`push_back`和`back`方法取新的当前对象。这种列表的维护理想在引擎的很多地方都存在，例如协程等需要记录上下文的场景
10. （核心构建入口）对AB资源的各个子序列调用`BuildAssetBundle`开始构建，收集完asset内所有对象后开始调用同名的静态函数进行Build，详见下文子章节
11. 用`core::hash_map<SInt32, AssetDatabase::AssetBundleFullName>`按照AB资源名称进行缓存，避免`AssetDatabase::GetAssetBundleName`的多次调用，那样会降低效率
12. 确定这个scene asset放在正确位置后，判断是否是最后一个asset bundle。如果是最后一个可以用后处理机制来优化，避免不必要的重复编译脚本。当然，出现异常Abort也走这个后处理：
    ```c++
    RestoreBuildTarget(previousTargetGroup, previousTargetPlatform, previousSubtarget);
    CleanupAfterBuildPlayer(kPlayerAssembliesPath);
    report.SetBuildResult(BuildReporting::kResultFailed);
    report.EndBuildStep(buildStepBundleToken);
    EndPlayerBuilding(report, sceneBackups, buildStepAllSceneBundles);
    CleanupAssetBundleBuild();
    SendBuildAssetBundlesAnalytics(tr("Aborting Build Due to Errors"));
    ```

13. 每次循环中`MonitoredGarbageCollectIfHighMemoryUsage`的`UnloadUnusedAssetsImmediate`可以动态卸载不需要的assts来节省内存，同时监控GC后的内存变化。
14. 调用`CleanupAfterBuildPlayer`，恢复配置到未指定平台

### BuildAssetBundle

核心的AssetBundle迭代构建入口
1. ` report.SetBuildResult(BuildReporting::kResultFailed);`先假设失败，看来作者是悲观派
2. 根据设置开始创建各种需要的文件夹，包括存放AB的、inernalPath、临时路径
3. 随后为packaging步骤预备bundle文件，但实际上暂时并未加载到内存
4. 在运行`CalculateBuildCompressionSettingsFromLegacyBuildAssetBundleOptions`后调用`BuildAssetBundleArchiveFile`编译archive文件，这一个函数内：
   1. 给scene根据index排序，准备编译header的配置以及压缩设置（我们一般用LZ4）
   2. 然后一个场景一个场景添加到archive
        > 在此过程中，我们访问了前面申明的内存区间中的地址，如果尝试访问内存地址出现异常，大概率就是这里分配出来的内存被占用/被释放/超出机器极限。可以先尝试设置4倍的虚拟内存再尝试，虚拟内存再大效果可能会变差，需要过多CPU时间去定位映射出来的虚拟内存地址。

   3. 做一些文件迁移出临时文件夹、文件重定位、大小统计的活