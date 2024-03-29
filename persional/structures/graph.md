# 图
1. 邻接矩阵：使用一个二维数组 G[N][N]存储图，如果顶点 Vi 和 顶点 Vj 之间有边，则 G[Vi][Vj] = 1 或 weight。邻接矩阵是对称的。
2. 邻接表：图的一种链式存储结构：对于图 G 中每个顶点 Vi，把所有邻接于 Vi 的顶点 Vj 链成一个单链表，这个单链表称为顶点 Vi 的邻接表。
3. 接表占用空间少，适合存储稀疏图；邻接矩阵适合存储稠密图。如果需要直接判断任意两个结点之间是否有边连接，可能也要用邻接矩阵。

## 图的类型
1. 完全图：任意两个顶点都相连的图称为完全图，又分为无向完全图和有向完全图。
2. 连通图：在无向图中，若任意两个顶点vivi与vjvj都有路径相通，则称该无向图为连通图。
3. 强连通图：在有向图中，若任意两个顶点vivi与vjvj都有路径相通，则称该有向图为强连通图。
4. 连通网：带权值的连通图叫做连通网。

## 图的遍历
1. 深度优先搜索遍历(DFS)
    基本步骤：
   - 从图中某个顶点v0出发，首先访问v0;
   - 访问结点v0的第一个邻接点，以这个邻接点vt作为一个新节点，访问vt所有邻接点。直到以vt出发的所有节点都被访问到，回溯到v0的下一个未被访问过的邻接点，以这个邻结点为新节点，重复上述步骤。直到图中所有与v0相通的所有节点都被访问到。
   - 若此时图中仍有未被访问的结点，则另选图中的一个未被访问的顶点作为起始点。重复深度优先搜索过程，直到图中的所有节点均被访问过。

    ![img.png](img_2.png)

2. 广度优先搜索遍历(BFS)
   基本步骤：
   - 从图中某个顶点v0出发，首先访问v0;
   - 依次访问v0的各个未被访问的邻接点。
   - 依次从上述邻接点出发，访问他们的各个未被访问的邻接点。始终保证一点：如果vi在vk之前被访问，则vi的邻接点应在vk的邻接点之前被访问。重复上述步骤，直到所有顶点都被访问到。
   - 如果还有顶点未被访问到，则随机选择一个作为起始点，重复上述过程，直到图中所有顶点都被访问到。
   - **提示：**为了按照优先访问顶点的次序，访问其邻接点，所以需要建立一个优先队列（先进先出）。
   ![img.png](img_3.png)

