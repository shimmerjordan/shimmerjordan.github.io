---
title: Python可视化csv数据实例
date: 2020-6-17 20:21:00
tags: 
    - Python
    - 数据可视化
categories: Python初级应用
id: py_csv_primary
---

​		Comma Separated Values，简称CSV，它是一种以逗号分隔数值的文件类型。在数据库或电子表格中，它是最常见的导入导出格式，它以一种简单而明了的方式存储和共享数据，CSV文件通常以纯文本的方式存储数据表。本篇文章记录了Python在csv格式的数据处理中的简单应用实例，并填补了一些曾经学习中的坑。

<!--more-->

# 1、数据预处理

​		首先考虑到数据集中可能存在数据缺失与重复，这里我们需要对数据集进行预处理，这里我选用了[国家数据网](http://data.stats.gov.cn/easyquery.htm?cn=C01)中的“指标/能源/综合能源平衡表”，筛选条件为20年。在获取到其csv文件后发现，其原数据的保存形式如下图所示：

![NZpdCF.jpg](https://s1.ax1x.com/2020/06/17/NZpdCF.jpg)

​		首先使用Excel的倒置粘贴对其进行坐标的倒置操作，另存为data.csv并使用NotePad++打开后其csv数据显示如下（部分）：

![NZpq58.jpg](https://s1.ax1x.com/2020/06/17/NZpq58.jpg)

​		可以看到回收能这一条目在2011-2015年没有数据记录，这里需要对其进行丢失数据的预处理。这里可以参考机器学习中的数据处理方式，由于这里丢失数据的规模较小，因此排除了删除特征项的处理方案。这里采用了`sklearn.preprocessing`的`Imputer`方法进行缺失值的处理，可以选择平均数，众数或者中位数来填充缺失的值。数据预处理核心代码如下：

```python
import pandas as pd

filename = 'data.csv'
df = pd.read_csv(filename, encoding='gbk') #解决'utf-8' codec can't decode byte 0xb1 in position 2: invalid start byte

print(df)
# 输出缺失值的个数
missing = df.isnull().sum()
print(missing)
print(missing.values, type(missing.values))
print(missing.values[1:])  # 可以索引取值，但是不是列表，方法属性不一样
print(dir(missing.values))

print(df.dropna())  # 按行删除有缺失值
print(df.dropna(axis=1))  # 删除列中有缺失值的列
print(df.dropna(how="all"))  # 删除那些行全是缺失值的

from sklearn.preprocessing import Imputer

#strategy表示采用何种策略，有mean，median， most_frequent
#axis=0, 表示在列上面进行操作， axis=1表示在行上面进行操作
imr = Imputer(missing_values='', strategy='mean', axis=1)

## 以后进行transform的时候，直接将这些值填充过去就可以了
imr = imr.fit(df)
imputed_data = imr.transform(df.values)
print(imputed_data)
```

​		至此，关于数据预处理的部分成功完结，由于数据集并未涉及重复数据，这里不再赘述。

# 2、CSV数据文件可视化

## 2.1 数据结构分析

​		首先我们需要对csv的数据结构做一个简单的分析，可以看到预处理后的数据集第一行是年份信息，每一列是各类消耗途径的信息。这里我们考虑以时间为轴，针对各类消耗途径进行分析。因此在前面预处理数据的阶段进行了数据的转置，以此方便后续基于逐行读取的`csv.reader(file)`进行数组的截取。

## 2.2 数据的读取

​		这里引入csv库，利用其中的`csv.reader(file)`进行数据的读取，由于在数据预处理阶段已经将csv文件中冗余的说明文字去除，这里直接采用`next(csv.reader(file))`即可获取第一行数据并默认保存为属性信息。

```python
import csv

filename = 'data.csv'
with open(filename) as f:
    reader = csv.reader(f)
    header_row = next(reader)
    
    # 存储相关消耗类型
    year, total, produce_once, recycle, impo, expo = [], [], [], [], [], []
    ann_inven_diff, total_custom, agri_custom, indus_custom = [], [], [], []
    constru_custom, traf_custom, accomm_custom, other_custom = [], [], [], []
    live_custom, termin_custom, indus_termin_custom, source_loss = [], [], [], []
    cok_loss, oil_loss, total_loss, mean_diff = [], [], [], []
    
    del(header_row[0])
        
    for row in reader:
        year.append(row[0])
        total.append(int(row[1]))
        produce_once.append(int(row[2]))
        recycle.append(row[3])
        impo.append(int(row[4]))
        expo.append(int(row[5]))
        ann_inven_diff.append(int(row[6]))
        total_custom.append(int(row[7]))
        agri_custom.append(int(row[8]))
        indus_custom.append(int(row[9]))
        constru_custom.append(int(row[10]))
        traf_custom.append(int(row[11]))
        accomm_custom.append(int(row[12]))
        other_custom.append(int(row[13]))
        live_custom.append(int(row[14]))
        termin_custom.append(int(row[15]))
        indus_termin_custom.append(int(row[16]))
        source_loss.append(int(row[17]))
        cok_loss.append(int(row[18]))
        oil_loss.append(int(row[19]))
        total_loss.append(int(row[20]))
        mean_diff.append(int(row[21]))
```

​		针对每一次逐行读取，以数组下标为索引，逐个读取各年份的同类型消耗的数据，存放在消耗类型的列表中。例如这里的`year[]`存放String类型的年份信息，`total[]`存放总消耗量等等。

## 2.3 数据可视化

​		这里利用`matplotlib`库对csv数据进行数据可视化操作，考虑到依据年份有同类消耗的走势数据分析层面，也有多类型走势对比角度，另一方面也可以针对同一年份不同消耗类型的占比分析角度。因此我采用了组合折线图和饼状图的形式实现数据的可视化操作。核心代码以及相关注解如下（这段代码与上面的数据读取代码同文件）：

```python
from matplotlib import pyplot  as plt

plt.rcParams['font.sans-serif']=['SimHei'] #显示中文标签
plt.rcParams['axes.unicode_minus']=False   #这两行需要手动设置

	fig=plt.figure(dpi=300,figsize=(15,8))
    # 将各个列表传个plot()方法
    plt.plot(year,total,c='red',label='可供消费的能源总量(万吨标准煤)')
    plt.plot(year,produce_once,c='aliceblue',label='一次能源生产量(万吨标准煤)')
    plt.plot(year,recycle,c='aqua',label='回收能(万吨标准煤)')
    plt.plot(year,impo,c='azure',label='进口量(万吨标准煤)')
    plt.plot(year,expo,c='blanchedalmond',label='出口量(-)(万吨标准煤)')
    plt.plot(year,ann_inven_diff,c='chartreuse',label='年初年末库存差额(万吨标准煤)')
    plt.plot(year,total_custom,c='crimson',label='能源消费总量(万吨标准煤)')
    plt.plot(year,agri_custom,c='darkorange',label='农、林、牧、渔、水利业消费总量(万吨标准煤)')
    plt.plot(year,indus_custom,c='deepskyblue',label='工业消费总量(万吨标准煤)')
    plt.plot(year,constru_custom,c='gold',label='建筑业消费总量(万吨标准煤)')
    plt.plot(year,total_loss,c='lime',label='损失量(万吨标准煤)')
    plt.plot(year,mean_diff,c='violet',label='平衡差额(万吨标准煤)')
    
    # 设置图形的格式
    plt.title('2000-2017综合能源平衡折线图（部分）',fontsize=24)
    plt.xlabel('year',fontsize=26)
    
    # 绘制斜线日期标签
    fig.autofmt_xdate()
    plt.ylabel('万吨标准煤',fontsize=26)
    plt.tick_params(axis='both',which='major',labelsize=10)
    plt.legend(loc=0,ncol=2)
    plt.show()

    # 2000年各消耗途径占比
    labels = ['农、林、牧、渔、水利业消费总量','工业消费总量','建筑业消费总量',\
              '交通运输、仓储和邮政业消费总量','生活消费总量',\
                  '其他行业消费总量','批发、零售业和住宿、餐饮业消费总量']
    sizes_2000 = [agri_custom[0], indus_custom[0], constru_custom[0],\
             traf_custom[0],live_custom[0] , other_custom[0],\
                 accomm_custom[0]]
    explode = (0,0,0,0.1,0,0,0)
    plt.figure(dpi=300,figsize=(15,8))
    plt.pie(sizes_2000,explode=explode,labels=labels,autopct='%1.1f%%',\
            shadow=False,textprops={'fontsize':18,'color':'black'}, startangle=150)
    plt.legend(loc="upper left",fontsize=10,bbox_to_anchor=(1.1,1.05),\
               ncol=1,borderaxespad=0.3)
    plt.title("饼图示例-2000年损耗占比",fontsize=24)
               
            
    # 2017年损耗占比
    sizes_2017 = [agri_custom[17], indus_custom[17], constru_custom[17],\
             traf_custom[17], live_custom[17] , other_custom[17],\
                accomm_custom[17]]
    explode = (0,0,0,0.1,0,0,0)
    plt.figure(dpi=300,figsize=(15,8))
    plt.pie(sizes_2017,explode=explode,labels=labels,autopct='%1.1f%%'\
            ,textprops={'fontsize':18,'color':'black'},shadow=True,startangle=150)
    plt.title("饼图示例-2017年损耗占比",fontsize=24)
    plt.legend(loc="upper left",fontsize=10,bbox_to_anchor=(1.1,1.05),ncol=1,borderaxespad=0.3)
    plt.show()   
```

​		这里遇到了包括中文标签乱码、`'utf-8' codec can't decode byte 0xb1 in position 2: invalid start byte`、`ValueError: x and y must have same first dimension, but have shapes (21,) and (18,)`等诸多问题。其解决方案在以上的代码注释中已经说明，这里不再赘述。

## 2.4 输出可视化成果

​		在填补了输出图像像素、标签等问题后，三张用于展示的图像如下所示：

![NZZReg.png](https://s1.ax1x.com/2020/06/17/NZZReg.png)

![NZZrWt.png](https://s1.ax1x.com/2020/06/17/NZZrWt.png)

![NZZySP.png](https://s1.ax1x.com/2020/06/17/NZZySP.png)

​		这里可以在绘图代码中对绘图参数进行修改，例如这里设置了两个饼图的不同阴影效果。另一方面，在折线图中由于值域的跨越较大，部分变化不明显，这里可以在源码中注释掉部分值域较大的数据集来对特定参数进行数据比对。

# 3、数据分析

​		不知道怎么分析，那就胡乱分析一波。总体来说，各类能源消费量在每年均呈现上涨趋势。可以看到，从2000-2017年中，基于生活基础建设类的能源消耗比例有一定程度的减小，而包括工业、农业等生产业的能源消耗有相应的比例提升。这和我国提出一带一路等国际化方针政策有些千丝万缕的关系，各类产业链的间接强化直接影响到了相应能源的消耗量。