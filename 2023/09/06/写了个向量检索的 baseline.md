> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sDAOENuXzn3aanym4n7sDA)

最近大模型的有关工作又把搜索，尤其是向量检索这块的工作带火了一把，检索在整个大模型环境中的意义在于，通过检索的方式能够快速找到可能匹配的知识点，通过输入给到大模型后，大模型能够进行抽取而给出更有针对性的回复，即实现常言的 “外挂知识库”。因此，我写了一套单机的向量召回方案（含向量模型推理），方便大家理解和使用。

为了方便大家理解，大家最好对向量召回中两个重要组件的概念有一定了解，一个是向量表征，一个是向量召回，其实这是两件事，可以看看我这篇文章：[心法利器 [16] | 向量表征和向量召回](http://mp.weixin.qq.com/s?__biz=MjM5ODkzMzMwMQ==&mid=2650422983&idx=4&sn=e96deacff90e46bfd07a64a8507a02b7&chksm=becdb89d89ba318b6f3cc80c7e2cc5957fa9c5449bab3f3a5fad747a6a69916c4db9662956e9&scene=21#wechat_redirect)

项目目录结构
------

首先看看文件的结构。

```
|-- main_faq_searcher.py
|-- model
|   |-- model.py
|   `-- simcse_model.py
|-- script
|   `-- run_build_index.py
`-- searcher
    |-- vec_index.py
    `-- vec_searcher.py


```

这里可以看做有 4 个大模块：

*   模型相关，用来进行向量化推理，我这使用的是 simcse 模型。
    
*   检索器，用于构造索引并执行检索。
    
*   脚本内有一个`run_build_index`，适用于构造索引，即在检索启动之前先把数据灌入的过程。
    
*   `main_faq_searcher`主程序，启动之后就可以进行检索了。
    

模型模块
----

模型这里我分了两个文件，simcse_model 是和新模型，这里是只管模型和推理的，而因为向量检索式依赖相似度计算的，所以我又用`model`包了一层，方便特别的计算，同时切换模型也会比较简单。这里我选用的是我自己比较喜欢用的 simcse，这个已经不算新东西了，我了解到最近因为大模型在这个方向又涌现了很多好的方案，大家也可以更换进行尝试。

首先是`simcse_model.py`，引用我带了链接，用的是一位大佬的模型，方便进行向量化。

```
import torch
import torch.nn as nn

from tqdm import tqdm
from transformers import BertConfig, BertModel

class SimcseModel(nn.Module):
    # https://blog.csdn.net/qq_44193969/article/details/126981581
    def __init__(self, pretrained_bert_path, drop_out, pooling="cls") -> None:
        super(SimcseModel, self).__init__()

        self.pretrained_bert_path = pretrained_bert_path
        config = BertConfig.from_pretrained(self.pretrained_bert_path)
        config.attention_probs_dropout_prob = drop_out
        config.hidden_dropout_prob = drop_out
        self.bert = BertModel.from_pretrained(self.pretrained_bert_path, config=config)
        self.pooling = pooling
    
    def forward(self, input_ids, attention_mask, token_type_ids):
        out = self.bert(input_ids, attention_mask, token_type_ids, output_hidden_states=True)

        if self.pooling == "cls":
            return out.last_hidden_state[:, 0]
        if self.pooling == "pooler":
            return out.pooler_output
        if self.pooling == 'last-avg':
            last = out.last_hidden_state.transpose(1, 2)
            return torch.avg_pool1d(last, kernel_size=last.shape[-1]).squeeze(-1)
        if self.pooling == 'first-last-avg':
            first = out.hidden_states[1].transpose(1, 2)
            last = out.hidden_states[-1].transpose(1, 2)
            first_avg = torch.avg_pool1d(first, kernel_size=last.shape[-1]).squeeze(-1)
            last_avg = torch.avg_pool1d(last, kernel_size=last.shape[-1]).squeeze(-1)
            avg = torch.cat((first_avg.unsqueeze(1), last_avg.unsqueeze(1)), dim=1)
            return torch.avg_pool1d(avg.transpose(1, 2), kernel_size=2).squeeze(-1)


```

然后是`model.py`，这个旨在包裹模型，并且给出模型预测的一些特定功能，再者也给切换模型带来一些方便，这里提供了

```
import torch
import torch.nn as nn
import torch.nn.functional as F

from transformers import BertTokenizer

from model.simcse_model import SimcseModel

class VectorizeModel:
    def __init__(self, ptm_model_path) -> None:
        self.tokenizer = BertTokenizer.from_pretrained(ptm_model_path)
        self.model = SimcseModel(pretrained_bert_path=ptm_model_path, pooling="cls")
        self.DEVICE = torch.device('cuda' if torch.cuda.is_available() else "cpu")
        self.model.to(self.DEVICE)
    
    def predict_vec(self,query):
        # 预测向量
        q_id = self.tokenizer(query, max_length = 200, truncation=True, padding="max_length", return_tensor='pt')
        with torch.no_grad():
            q_id_input_ids = q_id["input_ids"].squeeze(1).to(self.DEVICE)
            q_id_attention_mask = q_id["attention_mask"].squeeze(1).to(self.DEVICE)
            q_id_token_type_ids = q_id["token_type_ids"].squeeze(1).to(self.DEVICE)
            q_id_pred = self.model(q_id_input_ids, q_id_attention_mask, q_id_token_type_ids)
        return q_id_pred[0]
    
    def predict(self,q1, q2):
        # 预测两个句子的相似度
        q1_v = self.predict_vec(q1)
        q2_v = self.predict_vec(q2)
        sim = F.cosine_similarity(q1_v, q2_v, dim=-1)
        return sim


```

向量检索器
-----

### 科普一下检索里的关键概念

开始之前，想给不太理解检索的同学科普几个关键的概念，索引、倒排和正排，以及为什么我们需要倒排和正排。

首先先给大家解释倒排，抛开向量检索，先说字面检索，首先了解为什么我们搜 “倒排”，能够出很多有关倒排索引的文章，是因为底层有一套 kv 结构，和这个就叫做倒排，key 是切好的词汇，value 是包含这个词汇的所有文档的 title，即：

```
{
 "倒排":["搜索引擎概述之倒排索引 - 知乎","倒排索引简介","什么是倒排","倒排索引 | Elasticsearch: 权威指南 | Elastic", ...],
 "搜索":["搜狗搜索","搜索(汉语词语) - 百度百科", ....],
 "索引":["搜索引擎概述之倒排索引 - 知乎","倒排索引简介","倒排索引 | Elasticsearch: 权威指南 | Elastic", "索引 - 百度百科"...]
 ...
}


```

我们只需要找到你的检索词，把所有 value 都给你弄出来，这就叫做查询到了，然而随着库的变大，我们肯定不能把输入的每个字和库里面的做逐一匹配：

```
query = "倒排"
result = []
for index_key in database:
    if index_key == query:
        result.extend(database[index_key])


```

时间复杂度肯定就有问题（O(n)），不要小看这个线性复杂度，当库里面有千万甚至更多的内容时，线性复杂度也远远不够，我们就要用特定的数据结构来降低检索的时间复杂度，甚至不惜牺牲空间复杂度，对字面的，会考虑 trie 树等结构，可以把对数据条目数的复杂度降低到常数级，这些结构，我把他叫做**索引**。

至于正排，则是存的对应内容的详情的，例如这个：

```
[{
    "title":"搜索引擎概述之倒排索引 - 知乎",
    "docs"："xxxxxxxxxx",
    "insert_time":"2023081315550000"
},{
    "title":"倒排索引 - 百度文库",
    "docs"："xxxxxxxxxx",
    "insert_time":"2023081316550000"
}]


```

我们搜的时候，可能是针对 title 搜的，然而，我们没必要也不可以把别的和查询无关的信息也存到索引中，因此，我们构造了一个额外的数据结构，这样：

```
{"id1":{
    "title":"搜索引擎概述之倒排索引 - 知乎",
    "docs"："xxxxxxxxxx",
    "insert_time":"2023081315550000"
},"id2":{
    "title":"倒排索引 - 百度文库",
    "docs"："xxxxxxxxxx",
    "insert_time":"2023081316550000"
}}


```

当我们通过倒排查到了 id1 后，来这个新的数据结构里面，通过 id1 这个钥匙就能找到这个文档的详情，并且可以展示给用户了，这个结构，就是正排。

好了，这块的科普点到为止，更多有兴趣的内容，可以看《信息检索导论》以及《这就是搜索引擎》这两本书。

### 回到代码

向量检索器我这里同样分成了两块，构造了简单的向量索引，并包裹了一层向量检索器，向量检索器（`vec_searcher.py`）内提供了必要的检索和存储的功能，索引内核则是`vec_index.py`，当然了，也是为了方便切换，甚至可以切换成分布式的方案。

首先是向量索引内核，单机的选择，比较常见的是 annoy、hnswlib 之类的，这里我使用的是 ngt，是我自己用起来比较顺手的一个吧，另外两个大家也可以按需选择使用，写在 vec_index 里面就行了，这里总共提到了 3 个有关内容，非常推荐大家主动去学习下具体的使用方法：

*   annoy：https://github.com/spotify/annoy
    
*   hnswlib：https://github.com/nmslib/hnswlib
    
*   ngtpy：https://github.com/yahoojapan/NGT（只支持 linux）
    

当然了，数据多了单机肯定不够的，要上分布式的方案，诸如 elasticsearch、faiss、milvus 等，有兴趣大家可以了解下，此处我给大家弄的是一个简单的单机方案。

这里需要注意，这 3 个包都只是提供特殊的索引结构罢了，即类似 trie 树这种，而倒排和正排这种完整的检索结构，还需要我们单独去写的。现在首先先把这索引结构的使用弄明白：

```
import ngtpy

class VecIndex:
    def __init__(self) -> None:
        self.index = ""
    
    def build(self, output_path, index_dim):
        ngtpy.create(output_path, index_dim, distance_type="Consine", edge_size_for_creation=40, edge_size_for_search=80) # 余弦距离，同时也给出一些别的必要参数。
        self.index = ngtpy.Index(output_path)
    
    def insert(self, vec):
        self.index.insert(vec)
    
    def batch_insert(self, vecs):
        self.index.batch_insert(vecs)
        self.index.save()
    
    def load(self, path):
        self.index = ngtpy.Index(path)
    
    def search(self, query, num):
        # id, distance
        return self.index.search(query, size = num)


```

要做检索，要 4 个核心功能：构造、插入、加载、检索，单机层面可能还会包含保存（从内存转到本地），这个基本就是围绕 ngt 的基本功能来写的，整体而言还是比较简单的（我觉得甚至可以写一个基类了）。

然后就是 searcher，这就是我说的要构造倒排和正排了。

```
import os, json
from searcher.vec_index import VecIndex

class VecSearcher:
    def __init__(self):
        self.invert_index = "" # 检索倒排，使用的是索引是VecIndex
        self.forward_index = [] # 检索正排，实质上只是个list，通过ID获取对应的内容
        self.FORWARD_IDX_FORMAT_PATH = "{}/for" # 此处我自己是想把正排也放在和ngt索引一个位置，所以特地弄了个这个格式化的路径

    def build(self, output_path, index_dim):
        # 初始化
        index_name = "faq_index"
        index_path = os.path.join(output_path, index_name)
        self.output_path = index_path
        self.invert_index = VecIndex()
        self.invert_index.build(self.output_path, index_dim=index_dim)
    
    def batch_insert(self, vecs, docs):
        self.invert_index.batch_insert(vecs)
        self.invert_index.index.save()

        self.forward_index.extend(docs)
    
    def save(self):
        with open(self.FORWARD_IDX_FORMAT_PATH.format(self.output_path), "w") as f:
            for data in self.forward_index:
                f.write("{}\n".format(json.dumps(data, ensure_ascii=False)))
    
    def load(self, path):
        self.invert_index = VecIndex()
        self.invert_index.load(path)

        self.forward_index = []
        with open(self.FORWARD_IDX_FORMAT.format(path)) as f:
            for line in f:
                self.forward_index.append(line.strip())
    
    def search(self, vecs, nums = 5):
        search_res = self.invert_index.search(vecs, nums)
        result = []
        for idx, distance in search_res:
            result.append(self.forward_index[idx], distance)
        return result


```

这个就需要展开来讲了，我选重点的点出。

*   build：初始化出一个完整的空索引，只是定义了索引中的必要内容，基础的如维度、存储路径等，复杂的针对索引底层还有很多超参。
    
*   batch_insert：开始加载数据，一方面批量化把索引数据存到 index 里面，另一方面构造一个正排。
    
*   search：进行查询，查的是 index，index 返回的是检索到的 TopN 个最佳结果的 id，以及对应的距离，id 需要去正排里面取，才能够取到完整的所有内容。
    

灌数据
---

灌数据是一个离线过程，如果是一次性的或者手动的，那就准备一个脚本就行，脚本样例我写在这里：

```
import json
from tqdm import tqdm
# from loguru import logger
from search.vec_searcher import VecSearcher
from model import VectorizeModel

docs_path = "./data/docs_data20230813.data" # question \001 answer(json)
MODEL_PATH = "/download/berts/simcse-chinese-roberta-wwm-ext"
# 加载模型
vec_model = VectorizeModel(MODEL_PATH)

# 基础数据加载
docs_data = []
with open(docs_path) as f:
    for line in f:
        ll = line.strip().split("\001")
        ll_dict = json.loads(ll[1])
        docs_data.append([ll_dict])

# 推理向量
vecs = []
for q in tqdm(docs_data):
    vecs.append(vec_model.pedict_vec(q["question"]).cpu().numpy())

# 存入
vec_searcher = VecSearcher()
vec_searcher.build("index", 768)
vec_searcher.batch_insert(vecs, docs_data)
vec_searcher.save()

# 构造完成后的测试
q = "你好啊"
q_vec = vec_model.predict_vec(q).cpu().numpy()
vec_searcher.search(q_vec)


```

这里的代码可以看到非常简单，经历加载模型、数据加载、推理向量、存入和测试几个流程，只要流程明白，就不会很难了。

当然了，如果有定时更新、实时 / 被动更新之类的流程，那就配合用不同的工具来做就行。

*   例如定时更新，可以考虑 apschedule。
    
*   实时或者被动更新，这可以把这个脚本打包成一个服务，当收到请求的时候则进行更新。
    

faq 全流程检索类
----------

有了前面两个关键组件和构造好的索引，那我们就可以用来做向量检索了，有关网络服务这个我就不赘述了，这里我就把这个关键类给写出来：

```
import json
from search.vec_searcher import VecSearcher
from model import VectorizeModel

class FAQ:
    def __init__(self, model_path, vec_search_path):
        self.vec_model = VectorizeModel(model_path)

        self.vec_searcher = VecSearcher()
        self.vec_searcher.load(vec_search_path)
    
    def search(self, query, nums=3):
        q_vec = self.vec_model.predict_vec(query).cpu().numpy()
        result = self.vec_searcher.search(q_vec, nums)
        return result


```

其实就是加载和提供检索函数，因为前面的流程拆解的比较好，所以这个类就非常的简洁。

小结
--

本文给大家弄了一个简单的向量检索项目方案，包含了必要的向量推理和检索引擎，让大家可以快速理解并使用，值得注意的是，这只是一个 baseline，距离最终的合格效果以及最匹配在线的需求与性能还有一定距离，后续大家可能还要在这基础上做一些优化（我还是叠个甲吧）。