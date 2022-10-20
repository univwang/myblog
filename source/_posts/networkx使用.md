---
title: networkx使用
date: 2022-10-20 19:42:18
tags: [python, 图]
excerpt: 创建图并进行计算
categories: python
index_img: /img/index_img/5.png
banner_img: /img/banner_img/background6.jpg
---

>networkx是Python的一个包，用于构建和操作复杂的图结构，提供分析图的算法。图是由顶点、边和可选的属性构成的数据结构，顶点表示数据，边是由两个顶点唯一确定的，表示两个顶点之间的关系。顶点和边也可以拥有更多的属性，以存储更多的信息。

<a class="btn" target="_blank" rel="noopener" style="font-size:20px; color: green" href="https://www.osgeo.cn/networkx/install.html" title="github">Networkx使用文档</a>

<a class="btn" target="_blank" rel="noopener" style="font-size:20px; color: green" href="https://networkx.org/documentation/stable/reference/index.html
" title="github">Networkx英文文档</a>

<p class="note note-primary">networkx版本为2.87</p>

## 基础用法

### 创建图

![](https://raw.githubusercontent.com/univwang/img/main/20221020204000.png)

```python
import networkx as nx
G = nx.Graph()
```

### 图的节点

```python
g.add_node(1)
g.add_nodes_from([2,3,4])
g.add_node(1,name='n1',weight=1)
g.nodes()
```

```python 
g.remove_node(node_ID)
g.remove_nodes_from(nodes_list)
```

```python
g.nodes[1]['name'] = 'n1 new'
```


## 图的边

```python
g.add_edge(2,3)
g.add_edges_from([(1,2),(1,3)])
g.edges()

g.add_edge(1, 2, weight=4.7, relationship='renew')
g.add_weighted_edges_from([(1,2,0.125),(1,3,0.75),(2,4,1.2),(3,4,0.375)])
g.add_edges_from([(1,2,{'color':'blue'}), (2,3,{'weight':8})])

```


边是两个顶点的ID属性构成的元组，通过 edge=(node1,node2) 来标识边，进而从图中找到边：
```python
g.remove_edge(edge)
g.remove_edges_from(edges_list)
```


```python
g.edges(data=True) #带着属性
g[1][2]
```

```python 
g[1][2]['weight'] = 4.7
g.edge[1][2]['weight'] = 4
g[1][2].update({"weight": 4.7})
g.edges[1, 2].update({"weight": 4.7})
```

## 图的最短路求解

$dijkstra_path(G, source, target, weight='weight')$

使用Dijkstra方法计算图中两个节点之间的最短加权路径。返回从源到目标的最短加权路径$g$。默认情况下，求解最短路使用的是边权`weight`。


<dl class="field-list">
<dt class="field-odd">参数</dt>
<dd class="field-odd"><dl>
<dt><strong>G</strong><span class="classifier">网络X图表</span></dt><dd></dd>
<dt><strong>source</strong><span class="classifier">结点</span></dt><dd><p>起始节点</p>
</dd>
<dt><strong>target</strong><span class="classifier">结点</span></dt><dd><p>结束节点</p>
</dd>
<dt><strong>weight</strong><span class="classifier">字符串或函数</span></dt><dd><p>如果这是一个字符串，则边权重将通过具有此关键字的边属性(即边连接的权重)进行访问 <code class="xref py py-obj docutils literal notranslate"><span class="pre">u</span></code> 至 <code class="xref py py-obj docutils literal notranslate"><span class="pre">v</span></code> 将会是 <code class="docutils literal notranslate"><span class="pre">G.edges[u,</span> <span class="pre">v][weight]</span></code> )。如果不存在这样的边属性，则假定边的权重为1。</p>
<p>如果这是一个函数，则边的权重是函数返回的值。函数必须只接受三个位置参数：边的两个端点和该边的边属性字典。函数必须返回一个数字。</p>
</dd>
</dl>
</dd>
<dt class="field-even">返回</dt>
<dd class="field-even"><dl class="simple">
<dt><strong>path</strong><span class="classifier">列表</span></dt><dd><p>最短路径中的节点列表。</p>
</dd>
</dl>
</dd>
<dt class="field-odd">Raises:</dt>
<dd class="field-odd"><dl class="simple">
<dt>NodeNotFound</dt><dd><p>如果 <code class="xref py py-obj docutils literal notranslate"><span class="pre">source</span></code> 不在 <code class="xref py py-obj docutils literal notranslate"><span class="pre">G</span></code> .</p>
</dd>
<dt>NetworkXNoPath</dt><dd><p>如果在源和目标之间没有路径存在。</p>
</dd>
</dl>
</dd>
</dl>


```python
# 比较函数模板
def cmp(u, v, d):
    #字典对象，如果不存在key返回1
    node_u_wt = G.nodes[u].get('pow', 1)
    node_v_wt = G.nodes[v].get('pow', 1)
    edge_wt = d.get('weights', 1)
    return node_u_wt + node_v_wt + edge_wt
```