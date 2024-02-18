---
title: SPFA算法详解
date: 2020-02-13 11:36:55
tags: 
	- 算法
	- 最短路径优化
	- 负环解决
categories: 算法
id: intro-SPFA
---
# SPFA算法介绍 
SPFA算法全程为，Shortest Path Faster Algorithm，为Bellman-Ford算法的队列优化算分的别称，其最坏情况的复杂度与Bellman-Ford相同，为O(VE)  
与BFS算法比较，复杂度相对稳定。但在稠密图中复杂度比迪杰斯特拉算法差。  
对SPFA的一个很直观的理解就是由无权图的BFS转化而来。在无权图中，BFS首先到达的顶点所经历的路径一定是最短路(也就是经过的最少顶点数)，所以此时利用数组记录节点访问可以使每个顶点只进队一次。但在带权图中，最先到达的顶点所计算出来的路径不一定是最短路。一个解决方法是放弃数组，此时所需时间自然就是指数级的。所以我们不能放弃数组，而是在处理一个已经在队列中且当前所得的路径比原来更好的顶点时，直接更新最优解。  

<!--more-->

# 算法优点  
1. 时间复杂度低于普通Dijkstra以及Ford（一般情况下）
2. 能够计算负权图问题
3. 能够判断是否有负环

# 算法思想
我们用数组记录每个节点的最短路径估计值，用邻接表来存储图G。  
这里采取的方式是动态逼近法：  
　　1. 设立一个先进先出的队列用来存储待优化的节点。
　　2. 优化时每次取出队首节点u,并且用u点当前的最短路径估计值对离开u点所指向的节点v进行松弛操作，如果v点的最短路径估计值有所调整，且v点不在当前的队列中，就将v点放入队尾
　　3. 这样不断从队列中取出节点来进行松弛操作，直至队列空为止
事实上，如果一个节点被放入队列n次，那么即证明存在负环。

# 实现方法 
1. 存入图。可以使用{% post_link 前向星与链式前向星 链式前向星 %}或者vector；
2. 开一个队列，先将开始的节点放入；
3. 每次从队列中取出一个节点x，遍历与x相通的y节点，查询比对 dis[y] 和 dis[x]+edge[i].w  
　　如果dis[y] < dis[x]+edge[i].w，说明需要更新操作。
　　　　1) 存入最短路  
　　　　2) 由于改变了原有的长度，所以需要往后更新，与这个节点项链的最短路（即：判断是否存在队列，在就不需要重复，不在就加入队列，等待更新）  
　　　　3) 在这期间可以记录这个节点的仅对次数，判断是否存在负环（如果某个点进入队列的次数超过n即存在）  
4. 直到队空。

# 模拟过程
<div align=center><img src="https://i.imgur.com/V2L4gUM.png"></div>

首先建立起始点a到其余各点的最短路径表格  

![](https://i.imgur.com/cXnijXy.jpg)

首先源点a入队，当队列非空时：  
- 队首元素（a）出队，对以a为起始点的所有边的终点依次进行松弛操作（此处有b，c，d三个点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/dvKLisF.jpg"></div>

在松弛时三个点的路径估计值变小了，而这些点队列中都没有出现，这些点需要入队，此时，队列中新入队了三个节点b，c，d（表格中30应为∞，编辑错误）

- 队首元素b点出队，对以b为起始点的所有终点依次进行松弛操作（此时只有e点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/0OxHknG.jpg"></div>

在最短路径中，e的最短路径估值也变小了，e在队列中不存在，因此e也要入队，此时队列中的元素为c，d，e

- 队首元素c点出队，对以c为起始点的所有边的终点依次进行松弛操作（此处有e,f两个点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/8in2A1V.jpg"></div>

在最短路径表中，e，f的最短路径估值变小了，e在队列中存在，f不存在。因此e不用入队了，f要入队，此时队列中的元素为d，e，f

- 队首元素d点出队，对以d为起始点的所有边的终点依次进行松弛操作（此处只有g这个点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/ft0AfIb.jpg"></div>

在最短路径表中，g的最短路径估值没有变小（松弛不成功），没有新结点入队，队列中元素为f，g

- 队首元素f点出队，对以f为起始点的所有边的终点依次进行松弛操作（此处有d，e，g三个点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/CiU4zsB.jpg"></div>

在最短路径表中，e，g的最短路径估值又变小，队列中无e点，e入队，队列中存在g这个点，g不用入队，此时队列中元素为g，e

- 队首元素g点出队，对以g为起始点的所有边的终点依次进行松弛操作（此处只有b点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/Zd63QDN.jpg></div>

在最短路径表中，b的最短路径估值又变小，队列中无b点，b入队，此时队列中元素为e，b队首元素e点出队，对以e为起始点的所有边的终点依次进行松弛操作（此处只有g这个点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/mjSGsIN.jpg"></div>

在最短路径表中，g的最短路径估值没变化（松弛不成功），此时队列中元素为b

- 队首元素b点出队，对以b为起始点的所有边的终点依次进行松弛操作（此处只有e这个点），此时路径表格状态为：  
<div align=center><img src="https://i.imgur.com/EdyYHof.jpg"></div>

在最短路径表中，e的最短路径估值没变化（松弛不成功），此时队列为空了

最终a到g的最短路径为 14

# SPFA核心代码

```C++
bool SPFA(int s, int n) {
	queue <int> q;
	memset(vis, inf, sizeof(vis));
	memset(ven, 0, sizeof(ven));
	memset(nums, 0, sizeof(nums));
	vis[s] = 0;			//初始化距离 
	ven[s] = 1, nums[s]++;	//标记s节点在队列，队列次数+1 
	q.push(s);
	while (!q.empty()) {
		int x = q.front();
		q.pop();		//出队 
		ven[x] = 0;		//标记不在队列 
		for (int i = pre[x]; ~i; i = a[i].next) {	//遍历与x节点连通的点 
		
			int y = a[i].y;
			if (vis[y] > vis[x] + a[i].time) { //更新 
			
				vis[y] = vis[x] + a[i].time;
				if (!ven[y]) {
					//由于更新了节点，所以后续以这个为基础的最短路，也要更新下
					//所以如果在队列就不用加入，不在的话加入更新后续节点 
					q.push(y);
					ven[y] = 1;	//标记这个节点在队列中 
					nums[y]++;	//记录加入次数 
					if (nums[y] > n)	//加入超过n次说明存在负圈，直接返回 
						return false;
				}
			}
		}
	}
	return true;
}
```

# 关于差分约束和最短路  
SPFA通常用于计算最短路来解决差分约束问题，关于差分约束和最短路可见{% post_link 差分约束与最短路 差分约束与最短路 %} 

# 例题 
{% post_link CCF-201809-4-再卖菜 CCF 201809-4 再卖菜 %}



