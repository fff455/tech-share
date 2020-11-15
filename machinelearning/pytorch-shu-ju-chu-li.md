# PyTorch数据处理

PyTorch对数据处理有一套标准的接口操作，本篇对其进行一个总结，方便在使用PyTorch进行数据预处理和数据提取时使用。

PyTorch的几个重要的和数据处理相关的类都在torch.utils.data包中。

## DataLoader

PyTorch数据处理的核心是DataLoader类。通过以下语句引入。

```python
from torch.utils.data import DataLoader

# DataLoader初始化参数

dataloader = DataLoader(dataset, batch_size=1, shuffle=False, sampler=None, 
                        batch_sampler=None, num_workers=0, collate_fn=None, 
                        pin_memory=False, drop_last=False, timeout=0, 
                        worker_init_fn=None)
```

代码中给出了DataLoader类可以提供的参数。

* dataset 用于提取数据的dataset，通常是继承torch.utils.data.Dataset的对象。
* batch\_size 进行训练每个batch的样本数量。
* shuffle 是否在每个训练epoch打乱数据顺序。
* sampler 采样器，用于定义随机获得每个batch的采样策略，如果设置了sampler就必须设置shuffle=False。
* batch\_sampler 和batch\_size+shuffle+sampler+drop\_last起到作用类似，通常不使用。
* num\_workers 定义是否开启多子进程数据加载。
* pin\_memory 设置为True时可以提高将cpu上的Tensor转到GPU上的速率，但会提高内存消耗。
* collated\_fn 将一个样本列表组装成一个batch的中间处理函数，可自定义。
* drop\_last 是否去除最后一个没有达到batch\_size大小的batch
* timeout 如果设置成正数，会在超时后停止batch采集作业。
* worker\_init\_fn workers的初始化函数

其中我们主要通过实现dataset sampler collate\_fn三个参数来实现自定义数据加载器的功能。

## dataset

dataset参数必须继承Dataset类。

其中Dataset类实际上是用于形成Map类型的datasets。另一种IterableDataset用于流式数据加载的场景，不是很常见。

Dataset类的子类需要实现两个函数：

1. \_\_getitem\_\_\(idx\)  该函数表示根据输入的索引idx，从数据中提取一个样本返回。
2. \_\_len\_\_\(\)  该函数表示返回数据样本的总数

一个简单的已经将数据按照样本一条条排布的数据集如下所示，但实际上，这个类赋予了我们很多的灵活性，可以在函数中进行很多额外的检查和操作。

```python
from torch.utils.data import Dataset

class MyDataset(Dataset):

    def __init__(self, total_data):
        # total_data是一个10000*10的numpy.ndarray类型数据，其中前9列是特征，最后一列是标签
        self.features = total_data[:, :9]
        self.label = total_data[:, -1]

    def __getitem__(self, idx):
        y = self.features[idx]
        x = self.label[idx]

        return {'x':x, 'y':y}

    def __len__(self):
        return len(self.features)
```

有时候我们也可以调用PyTorch已经实现好的TensorDataset、ConcatDataset等类，避免自己实现的麻烦。

## sampler

sampler需要继承torch.utils.data.Sampler类，一般要以data\_source作为参数，负责传递索引给Dataset对象。

PyTorch中已经提供的几种采样器名称和功能如下：

1. torch.utils.data.SequentialSampler\(data\_source\) 按顺序进行采样。
2. torch.utils.data.RandomSampler\(data\_source, replacement=False, num\_samples=None\) 根据参数进行一定数量的有或者无放回随机采样。
3. torch.utils.data.SubsetRandomSampler\(indices\) 在给定的索引列表中进行再次采样。
4. torch.utils.data.WeightedRandomSampler\(weights, num\_samples, replacement=True\) 按照权重进行随机采样。

sampler需要实现两个函数 \__len\_\(\)和\_\_iter\_\(\)，作用分别是给出数据总长度和该次迭代返回的索引值。

下面我们实现一个根据数据集大小决定是否进行随机采样的sampler。

```python
from torch.utils.data import Sampler
import numpy as np

class MySampler(Sampler):

    def __init__(self, data_source):
        self.data_source = data_source

    def __iter__(self):
        if len(self.data_source) >= 10000:
            return iter(range(len(self.data_source)))
        else:
            return np.random.randint(len(self.data_source))

    def __len__(self):
        return len(self.data_source)
```

## collate\_fn

collate\_fn是一个callable的对象，其实就是一个函数。这个函数的输入是由sampler按照index在dataset中提取出的batch\_size个样本组成的list列表。

collate\_fn负责将这个list中的元素组装成一个完整的Tensor矩阵。为此默认的collate\_fn函数具有以下三个性质：

1. 总是会在将每个样本组合起来后，在最左边添加一个大小为batch\_size的新维度。
2. 自动将numpy数组转化成PyTorch的Tensor类型数据。
3. 会保留数据结构，例如在dataset节中，我们将输出结果设置成一个有x和y两个key的字典值。collate\_fn将会保留这个字典形式的结构，只是将每个key的value替换成batch\_size长度的Tensor。

一个自定义的在第二个维度上做stack操作的collate\_fn如下（假设每个样本格式为字典）:

```python
import torch

def my_collate_fn(data)
    x_list = [data[i]['x'] for i in range(len(data))]
    y_list = [data[i]['y'] for i in range(len(data))]

    x_tensor = torch.FloatTensor(x_list)
    y_tensor = torch.FloatTensor(y_list)

    return {'x': x_tensor, 'y': y_tensor}
```

总结来说，只需要熟练掌握自定义和使用dataset、sampler、collate\_fn的方法，可以满足我们各种实验的需求，也会使得我们的代码更加有条理，提高实验效率。

