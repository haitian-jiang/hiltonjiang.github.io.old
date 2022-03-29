---
title: PyG source code reading
category: GNN
date: 2022-3-6
---



### Load Data

```python
loader = DataLoader(dataset, batch_size=32, shuffle=False)
for batch in loader:
    batch
```

batch中为子图，batch_size 为子图的个数

cora中使用batch.train_mask用来区分训练节点（已知label）和其他节点（如cora）中，train_mask为[0:140]，val_mask为[140:640]，test_mask为[1708:2708]

对于ogb中的数据集，可以试用`from torch_geometric_autoscale.utils import index2mask`将数据集划分的index转换成mask

```python
split_idx = dataset.get_idx_split()
data.train_mask = index2mask(split_idx['train'], data.num_nodes)
data.val_mask = index2mask(split_idx['valid'], data.num_nodes)
data.test_mask = index2mask(split_idx['test'], data.num_nodes)
```







### [ogbn-papers100M](https://ogb.stanford.edu/docs/nodeprop/#ogbn-papers100M) 数据集

总节点个数：111,059,956

训练集节点个数：1207179

验证集节点个数：125265

测试集节点个数：214338

特征维度：128

节点种类：172

`data.y` 为`torch.FloatTensor`类型，无label的数据均用nan表示。





### PyGAS

`ScalableGNN`基类中包含`History`，初始化`ScalableGNN`时的参数`device`指定的是`History`的设备

`ScalableGNN`中的`pool`位于模型相同的设备上（GPU）

PyGAS需要使用`SparseTensor`作为邻接矩阵类型，是因为metis后重排边的顺序只实现了稀疏矩阵版本。
