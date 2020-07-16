---
layout:     post
title:      【推荐算法实战（二）】PersonalRank
subtitle:   个性化召回算法LFM
date:       2020-07-14 22.15
author:     Raserting_L
header-img: img/artical/OIP (1).jpg
catalog: true
tags:
    - 推荐算法






---



# 一、PersonalRank算法背景与物理意义



## 1. 背景

1. 用户行为很容易表示为图

   图这种数据结构有两个基本的概念：顶点和边。

   

   在实际的个性化推荐系统中，无论是信息流场景、电商场景或者是O2O场景，用户无论是点击、购买、分享、评论等等的行为都是在user和item两个顶点之间搭起了一条连接边，构成了图的基本要素。

   

   实际上这里user与item构成的图是二分图，后面会介绍二分图的概念以及结合具体的例子展示如何将用户行为转换为图。

   

2. 图推荐在个性化推荐领域效果显著

## 2. 二分图

​	二分图又称为二部图，是图论中的一种特殊模型。设G=(V,E)是一个无向图，如果顶点V可分割为两个互不相交的子集（A,B），并且图中的每条边（i,j）所关联的两个顶点 i 和 j 分别属于这两个不同的顶点集（i in A, j in B）,则称图G为一个二分图。

​	在推荐系统中，user、item恰好满足两种独立的集合，并且用户行为总是从user顶点到item顶点集合，所以由推荐系统中user和item之间构成的图就是二分图。

​	接下来结合具体实例讲解如何将用户的行为转化为二分图。假设某推荐系统中有4个用户：A B C D，以及从日志（log）中发现对如下item有过行为：**user A 对 item a、b、d有过行为，userB 对 item a、c有过行为，userC对 item b、e有过行为，userD 对 item c、d有过行为。**

那么可以得到如下图：

![](https://upload-images.jianshu.io/upload_images/11172737-4c1433d18fb835ac.png?imageMogr2/auto-orient/strip|imageView2/2/w/385/format/webp)

## 3. 物理意义

​	下面从物理意义的角度来分析一下，从二分图上如何分析出来item集合对user的重要程度。	



​	1. 两个顶点之间连通的路径数

​		如果要比较两个item顶点对固定user的重要程度，只需分别看一下user到两个item顶点的路径数，**路径		数越多的顶点越重要**。

​	2. 两个顶点之间连通的路径长度

​		同样路径数的情况下，**总路径长度越短的顶点越重要**。	

​	3. 两个顶点之间连通路径经过顶点的出度

​		如user A对item a、b、d有过行为，即为有条连接边，则A的出度为3。如果前两项都相同，则两个item		对固定user的重要程度可以通过比较经过顶点的所有的出度和来得出，如果**出度和越小则越重要**。



**举例：**对于上图的A来说，c和e哪个更重要?

​	1.分别有几条路径连接？

​		首先看A-c 之间有几条路径连通：分别是A-a-B-c，A-d-D-c 两条路径连通。	

​		再来看A-e 之间有几条路径连通：A-b-C-e一条路径

​		从这一角度出发，可以知道 c 比 e 重要。

​	2.连通路径的长度分别是多少？

​		首先看A-c 之间有几条路径连通：分别是A-a-B-c，A-d-D-c ，长度都为3

​		再来看A-e 之间有几条路径连通：A-b-C-e长度为3

​	3.连通路径的经过顶点出度分别是多少?

​		首先看A-a-B-c这条路径：A出度是3，a出度是2，B出度是2，c出度是2

​		再看A-d-D-c这条路径：A出度是3，d出度是2，D出度是2，c出度是2

​		再看A-b-C-e这条路劲：A出度是3，b出度是2，C出度是2，e出度是1

​		这里虽然 e 的出度和更小，但是由于1中 c 有两条路径，且1的优先级更高，所以还是应该推荐 c。

# 二、PersonalRank数学公式推导

## 1. 算法文字阐述

​	对用户A进行个性化推荐，从用户A节点开始在用户-物品二分图random walk，以alpha的概率从A的出边中等概率选择一条游走过去，到达该顶点后（举例顶点a），有alpha的概率继续从顶点a的出边中等概率选择一条继续游走到下一个节点，或者（1-alpha）的概率回到顶点A，多次迭代。直到各顶点对于用户A的重要度收敛。



## 2. 算法的数学公式

![](https://upload-images.jianshu.io/upload_images/11172737-a6ee74c9df0c769e.png?imageMogr2/auto-orient/strip|imageView2/2/w/927/format/webp)

​	

​	把不同item对user的重要程度描述为**PR值**。



​	**举例：**为了便于理解，把A作为固定起点。user A的PR值初始化为1，其余节点的PR值初始化为0。

​	这里使用 a 节点和 A 节点阐述公式：



​		从user A出发有3条边，等概率，即1/3的概率到节点a；user B以1/2的概率选择了a。结合描述看一下公式的上半部分（即v != v_A那一行）：对于不是A节点的PR值，也就是 a 的PR值，那么首先要找到连接该顶点节点，同时分别计算他们PR值的几分之几贡献到要求节点的PR值。那么A将自己PR值的1/3贡献给了 a ，B将自己PR值得1/2贡献给了 a，分别求和，乘alpha，得到 a 的PR值。

​		接下来看下半部分：如果要求A节点本身的PR值，首先知道任意节点都会以（1-alpha）的概率回到本身，那么对于一些本来就与A节点相连的节点，比如这里的 a 节点或者 b 节点，它们除了以（1-alpha）的概率直接回到A以外，还可以以alpha的概率从自己的出边中等概率的选择与A相邻的这条边，比如这里的 a 节点，可以以1/2的概率选择回到A节点，所以就构成了下半部分的前后两个部分。

​		经过分析可以发现，personal rank算法求item对固定user的PR值，需要每次迭代都在全图范围内迭代，时间复杂度在工业界实际算法落地的时候是不能接受的，所以要让尽可能多的user并行迭代。结合之前许多其他算法训练的工业界实现，很容易想到矩阵化实现，下面看personal rank算法的矩阵化实现。



## 3. 算法抽象矩阵式

$$
r=(1-a)r_0+ \alpha M^Tr
$$

$$
M_{ij}=\frac {1}{|out(i)|} j \in out(i)else 0
$$

​	假设这里共有m个user，n个item。



​	r是m+n行，1列的矩阵，表示其余顶点对该固定顶点的PR值。当然得到了这个，就得到了固定顶点下其余所有顶点的重要程度排序，这里只需要排出m个user节点。

​	r0 是m+n行，1列的矩阵，负责选取某一节点是固定节点，它的数值只有一行为1，其余行全为0。为1的行，即为选取了该行对应的顶点为固定顶点。那么得到的就是该固定顶点下，其余节点对该固定节点的重要程度的排序。

​	M 是 m+n行 * m+n列的矩阵，也就是行包含了所有的节点，列也包含了所有的节点。 它是转移矩阵，数值定义如下：1.第一行第二列的数值距离，如果第一行对应index的顶点由出边连接到了第二列对应index的顶点，那么该值就为第一行顶点的出度的倒数（也就是转移概率）；如果没有连接边，那么就是0。（所以这么M矩阵一般为稀疏矩阵）



​	最后我们通过对上面第一个式子进行矩阵变换，将 **r** 换到单独的左边：
$$
（E- \alpha M^T)*r=(1-a)r_0
$$

$$
最终得到： r=(E- \alpha M^T)^{-1}(1- \alpha)r_0
$$

​	刚才说过，r_0是m+n行，1列的矩阵，它能够选取固定的顶点，得到固定顶点的推荐结果。如果将r_0变为（m+n）*（m+n）的矩阵，也就得到了所有顶点的推荐结果。

​	由于得到的推荐结果是考虑顶点之间的PR值的顺序关系，并非一个绝对数值，所以可以将（1- alpha）舍去。所以
$$
(E- \alpha M^T)^{-1}
$$
即为所有顶点的推荐结果。每一位表示该顶点下，其余顶点对于该顶点的PR值。

​	但是，需要注意的是，每一个user能够行为的item毕竟是少数，所以这里的M矩阵是稀疏矩阵，则上式同样也是稀疏矩阵。

# 三、 代码解析



## 1. 直接使用2.2公式计算

首先介绍的是直接使用公式进行计算（与之后的使用矩阵相对立）。



首先我们需要将数据读入并且将其变为图的形式：

```python
def get_graph_from_data(input_file):
    """
    Args:
        input_file :user-item图文件
    Return:
        dict形如: {UserA:{itemb:1, itemc:1}, itemb:{UserA:1}} 键是userid/itemid，值为相应的有联系的itemid/userid:
    """

    score_thr = 4
    graph = {}


    if not os.path.exists(input_file):
        return graph
    fp = open(input_file)
    linenum = 0
    for line in fp:
        if linenum == 0:
            linenum += 1
            continue
        item = line.strip().split(',')
        if len(item) < 3:
            continue
        userid, itemid, rating = item[0], "item_"+item[1], item[2]
        if float(rating) < score_thr:
            continue

        if userid not in graph:
            graph[userid] = {}
        graph[userid][itemid] = 1
        if itemid not in graph:
            graph[itemid] = {}
        graph[itemid][userid] = 1
    fp.close()
    return graph



# 调用
get_graph_from_data("./data/ratings.txt")["1"]
```

得到了数据并且转化为图结构之后，我们可以开始计算其PR值，也就是PersonalRank模型

```python
def personal_rank(graph, root, alpha, iter_num, recom_num = 10):
    """
    Args:
        graph:      user-item图
        root:       给哪个user推荐
        alpha:      以alpha概率选择往下随机游走，1-alpha概率回到起点
        iter_num:   迭代次数
        recom_num:  给用户推荐的图片数目
    Return:
        dict :  {itemid:pr值}
    """

    #首先将root顶点的pr值初始化为1，其余全为0
    rank = {}
    rank = {point:0 for point in graph}
    rank[root] = 1 

    recom_result = {}

    for iter_index in range(iter_num):
        temp_rank = {}
        temp_rank = {point:0 for point in graph}
        for out_point, out_dict in graph.items():
            for inner_point, value in out_dict.items():
                temp_rank[inner_point] += round(alpha * rank[out_point] / len(out_dict), 4)
                if inner_point == root:
                    temp_rank[inner_point] += round(1 - alpha, 4)
        
        if temp_rank == rank: # 迭代以此之后和之前的完全相同，说明收敛了可以退出了
            break 
        rank = temp_rank
    for res in sorted(rank.items(), key = lambda v: v[1], reverse=True):
        point, pr_score = res[0], res[1]
        if len(point.split('_')) < 2:  #如果是user顶点就不做任何事
            continue
        if point in graph[root]: # 如果这个顶点已经被root节点访问过，那就不需要推荐了
            continue
        if recom_num <= 0:
            break
        recom_num -= 1
        recom_result[point] = pr_score
    return recom_result
    

    
    
# 下面进行测试，得到一个用户的推荐列表
user = "1"
alpha = 0.6
iter_num = 10
recom_num = 10
graph = get_graph_from_data("./data/ratings.txt")
print(personal_rank(graph, user, alpha, iter_num, recom_num))
```



为了能够更直观看到效果，像上一节一样，将编号与电影名相对应并且打印出来

```python
# 下面引入之前的读取movie数据的函数来把结果与电影进行映射
# 获取movie文件内容
def get_item_info(input_file):
    """
    get item info[title, genre]
    Args(参数):
        input_file:item info file
    Return(输出):
        a dict:{key:itemid, value:[title, genre]}
    """
    if not os.path.exists(input_file):
        return {}
    item_info = {}
    linenum = 0 
    fp = open(input_file)
    for line in fp:
        if linenum == 0:
            linenum += 1
            continue
        item = line.strip().split(',')  # strip方法为移除字符串头尾的指定字符，默认为空格和换行符
        if len(item) < 3:
            continue
        elif len(item) == 3:
            itemid, title, genre = item[0], item[1], item[2]
        elif len(item) > 3:
            itemid = item[0]
            genre = item[-1]
            title = ",".join(item[1:-1])
            
        item_info.update({itemid:[title, genre]})
        
    fp.close()

    return item_info

```

接下来就可以进行测试了

```python
# 下面进行测试，得到一个用户的推荐列表
user = "1"
alpha = 0.6
iter_num = 10
recom_num = 100
graph = get_graph_from_data("./data/ratings.txt")

item_info = get_item_info("./data/movies.txt")

recom_result_base = personal_rank(graph, user, alpha, iter_num, recom_num)

print("已经点击过的：")
for itemid in graph.get(user):
    print(item_info.get(itemid.split('_')[1]), "\t")

print("推荐的可能喜欢的：")
for itemid in recom_result:
    print(item_info.get(itemid.split('_')[1]), "\t", recom_result.get(itemid))
```

## 2. 接下来看看矩阵形式下的PersonalRank的训练方法



首先也是读入数据，并且将其转化为矩阵的形式存储，但是这里注意，由于访问数据是很稀疏的，所以我们需要对其使用稀疏矩阵的处理方法，需要用到几个模块如下：

```python
from scipy.sparse import coo_matrix  #之后建立稀疏矩阵用
from scipy.sparse.linalg import gmres # 之后解非齐次线性方程组
```

先是得到数据转化为矩阵的方式存储

```python
def graph_to_mat(graph):
    """
    Args:
        graph: 上面的得到的 user-item图字典
    Return：
        Mat：一个稀疏矩阵
        List: 一个包括所有顶点的列表：包括用户/物品
        dict: 一个字典，由 user/item节点到索引数值index的映射
    """

    vertex = list(graph.keys()) # 获得所有的顶点，包括用户与物品
    address_dict = {}  # 建立一个由 user/item节点到索引数值index的映射

    # 定义行索引，列索引，数值，来存储稀疏矩阵
    total_len = len(vertex)
    row = []
    col = []
    data = []

    for index in range(len(vertex)):
        address_dict.update({vertex[index] : index })
    
    for element_i in graph:
        weight = round(1/len(graph[element_i]), 3)
        row_index = address_dict.get(element_i)
        for element_j in graph.get(element_i):
            col_index = address_dict.get(element_j)

            row.append(row_index)
            col.append(col_index)
            data.append(weight)

    row = np.array(row)
    col = np.array(col)
    data = np.array(data)

    # 下面得到稀疏矩阵
    M = coo_matrix ((data, (row, col)), shape=(total_len, total_len))

    return M, vertex, address_dict
```

然后根据公式，对矩阵进行计算
$$
（E- \alpha M^T)
$$

```python
# 根据公式可知，需要得到单位矩阵减去alpha*m矩阵转置的差
# 下面这个函数就是进行这个操作
def mat_T(m_mat, vertex, alpha): 
    """
    Args:
        m_mat: m矩阵
        vertex: 一个包括所有顶点的列表：包括用户/物品
        alpha：随机游走的概率
    Return：
        M: 一个稀疏矩阵
    """
    total_len = len(vertex)
    row = []
    col = []
    data = []

    for index in range(total_len):
        row.append(index)
        col.append(index)
        data.append(1)

    row = np.array(row)
    col = np.array(col)
    data = np.array(data)

    eye_t = coo_matrix ((data, (row, col)), shape=(total_len, total_len)) #得到一个单位矩阵
    
    # 对于稀疏矩阵运算，我们使用tocsr()更快
    return eye_t.tocsr() - alpha * m_mat.tocsr().transpose()
```

之后就可以开始解非齐次线性方程组了，得到最后的r

```python
# 下面是PersonalRank矩阵式的计算

def personal_rank_mat(graph, root, alpha, recom_num = 10):
    """
    Args:
        graph:      user-item图
        root:       给哪个user推荐
        alpha:      以alpha概率选择往下随机游走，1-alpha概率回到起点
        iter_num:   迭代次数
        recom_num:  给用户推荐的图片数目
    Return:
        dict :  {itemid:pr值}
    """
    score_dict = {}
    recom_dict = {}

    # 首先得到m矩阵和顶点集合以及顶点到index的映射
    m, vertex, address_dict = graph_to_mat(graph)
    if root not in address_dict:
        return{}
    m_T = mat_T(m, vertex, alpha)
    # 接下来我们要得到m_T的逆矩阵，需要用到scipy.sparse.linalg里的gmres模块
    # 虽然上述模块文档中表示可以解A*x=b的线性方程且b可以是n*n的矩阵，但是实际使用中好像只能解n*1的矩阵
    #先对r0进行初始化，也就是把root的位置pr值变为1，其余为0
    index = address_dict.get(root)
    r0 = [0 for i in range(len(vertex))]
    r0[index] = 1
    r0 = np.array(r0)
    # 开始解方程 ,得到一个元组，这个元组的第一维度是一个数组表示所有顶点对固定顶点root的PR值
    res = gmres(m_T, r0, tol=1e-8)[0]

    for index in range(len(res)): #将PR值进行处理与itemid形成一个键值对
        point = vertex[index]
        if len(point.strip().split('_')) < 2 :  # 是user的话，不需要处理
            continue
        if point in graph.get(root): # 如果user已经访问过的也不要处理
            continue
        score_dict.update({point:round(res[index], 3)})
    
    #下面将pr值排序返回
    for val in sorted(score_dict.items(), key = lambda v:v[1], reverse=True)[:recom_num]:
        recom_dict.update({val[0]:val[1]})  # {item : PR}
    
    return recom_dict

```

最后进行一下测试

```python
# 下面测试矩阵的方法计算推荐
# 下面进行测试，得到一个用户的推荐列表
user = "1"
alpha = 0.6
recom_num = 100
graph = get_graph_from_data("./data/ratings.txt")

item_info = get_item_info("./data/movies.txt")

recom_result_mat = personal_rank_mat(graph, user, alpha, recom_num)

print("已经点击过的：")
for itemid in graph.get(user):
    print(item_info.get(itemid.split('_')[1]), "\t")

print("推荐的可能喜欢的：")
for itemid in recom_result:
    print(item_info.get(itemid.split('_')[1]), "\t", recom_result.get(itemid))
```



## 3. 两种方法比较

对于两种方式得到的推荐结果进行比较：

```python
# 下面将之前的基础方法和后面的矩阵方法比较看看二者推荐结果有多少是相同的(topk取10，为5个相同，topk为100时，为76个相同)
num = 0
for ele in recom_result_base:
    if ele in recom_result_mat:
        #print (ele)
        num += 1
print(num)
```



对于TopK为10时，相同数目为5

对于TopK为100时，相同数目为76.





写在后面: 虽然最后得到了推荐结果，但是可以发现我们每次计算基本上都对所有的user和item进行了一边遍历但是在实际使用中，面对海量数据，我们不能再用这么简单粗暴的方法，实际上我们可以用Hadoop和Spark等工具进行计算。