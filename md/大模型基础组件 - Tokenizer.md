> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/651430181)

Tokenizer 分词算法是 NLP 大模型最基础的组件，基于 Tokenizer 可以将文本转换成独立的 token 列表，进而转换成输入的向量成为计算机可以理解的输入形式。本文将对分词器进行系统梳理，包括分词模型的演化路径，可用的工具，并**手推**每个 tokenizer 的具体实现。

**速览**
------

1. 根据不同的切分粒度可以把 tokenizer 分为: 基于词的切分，基于字的切分和基于 subword 的切分。 基于 subword 的切分是目前的主流切分方式。  
2. subword 的切分包括: BPE(/BBPE), WordPiece 和 Unigram 三种分词模型。其中 WordPiece 可以认为是一种特殊的 BPE。  
3. 完整的分词流程包括：文本归一化，预切分，基于分词模型的切分，后处理。  
4. SentencePiece 是一个分词工具，内置 BEP 等多种分词方法，基于 Unicode 编码并且将空格视为特殊的 token。这是当前大模型的主流分词方案。

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>分词方法</th><th>典型模型</th></tr><tr><td>BPE</td><td>GPT, GPT-2, GPT-J, GPT-Neo, RoBERTa, BART, LLaMA, ChatGLM-6B, Baichuan</td></tr><tr><td>WordPiece</td><td>BERT, DistilBERT，MobileBERT</td></tr><tr><td>Unigram</td><td>AlBERT, T5, mBART, XLNet</td></tr></tbody></table>

**1. 基于 subword 的切分**
---------------------

基于 subword 的切分能很好平衡基于词切分和基于字切分的优缺点，也是目前主流最主流的切分方式。

但是基于词和字的切分都会存在一定的问题，直接应用的效果比较差。

基于词的切分，会造成:

*   词表规模过大
*   一定会存在 UNK，造成信息丢失
*   不能学习到词缀之间的关系，例如：dog 与 dogs，happy 与 unhappy

基于字的切分，会造成:

*   每个 token 的信息密度低
*   序列过长，解码效率很低

所以基于词和基于字的切分方式是两个极端，其优缺点也是互补的。而折中的 subword 就是一种相对平衡的方案。

subword 的基本切分原则是：

*   高频词依旧切分成完整的整词
*   低频词被切分成有意义的子词，例如 dogs => [dog, ##s]

基于 subword 的切分可以实现：

*   词表规模适中，解码效率较高
*   不存在 UNK，信息不丢失
*   能学习到词缀之间的关系

基于 subword 的切分包括：BPE，WordPiece 和 Unigram 三种分词模型。

**2. 切分流程**
-----------

Tokenizer 包括训练和推理两个环节。训练阶段指得是从语料中获取一个分词器模型。推理阶段指的是给定一个句子，基于分词模型切分成一连串的 token。

基本的流程如图所示，包括归一化，预分词，基于分词模型的切分，后处理 4 个步骤。

![](https://pic2.zhimg.com/v2-651f237fb96410b1000c94fa85645c1d_b.jpg)

**2.1. 归一化**
------------

这是最基础的文本清洗，包括删除多余的换行和空格，转小写，移除音调等。例如：

```
input: Héllò hôw are ü?
normalization: hello how are u?

```

HuggingFace tokenizer 的实现： [https://huggingface.co/docs/tokenizers/api/normalizers](https://link.zhihu.com/?target=https%3A//huggingface.co/docs/tokenizers/api/normalizers)

**2.2. 预分词**
------------

预分词阶段会把句子切分成更小的 “词” 单元。可以基于空格或者标点进行切分。 不同的 tokenizer 的实现细节是不一样的。例如:

```
input: Hello, how are  you?

pre-tokenize:
[BERT]: [('Hello', (0, 5)), (',', (5, 6)), ('how', (7, 10)), ('are', (11, 14)), ('you', (16, 19)), ('?', (19, 20))]

[GPT2]: [('Hello', (0, 5)), (',', (5, 6)), ('Ġhow', (6, 10)), ('Ġare', (10, 14)), ('Ġ', (14, 15)), ('Ġyou', (15, 19)), ('?', (19, 20))]

[t5]: [('▁Hello,', (0, 6)), ('▁how', (7, 10)), ('▁are', (11, 14)), ('▁you?', (16, 20))] 


```

可以看到 BERT 的 tokenizer 就是直接基于空格和标点进行切分。 GPT2 也是基于空格和标签，但是空格会保留成特殊字符 “Ġ”。 T5 则只基于空格进行切分，标点不会切分。并且空格会保留成特殊字符 "▁"，并且句子开头也会添加特殊字符 "▁"。

HuggingFace tokenizer 的实现： [https://huggingface.co/docs/tokenizers/api/pre-tokenizers](https://link.zhihu.com/?target=https%3A//huggingface.co/docs/tokenizers/api/pre-tokenizers)

**2.3. 基于分词模型的切分**
------------------

这里指的就是不同分词模型具体的切分方式。分词模型包括：BPE，WordPiece 和 Unigram 三种分词模型。

HuggingFace tokenizer 的实现： [https://huggingface.co/docs/tokenizers/api/models](https://link.zhihu.com/?target=https%3A//huggingface.co/docs/tokenizers/api/models)

**2.4. 后处理**
------------

后处理阶段会包括一些特殊的分词逻辑，例如添加 sepcial token：[CLS],[SEP] 等。 HuggingFace tokenizer 的实现： [https://huggingface.co/docs/tokenizers/api/post-processors](https://link.zhihu.com/?target=https%3A//huggingface.co/docs/tokenizers/api/post-processors)

**3.BPE**
---------

Byte-Pair Encoding(BPE) 是最广泛采用的 subword 分词器。

*   训练方法：从字符级的小词表出发，训练产生合并规则以及一个词表
*   编码方法：将文本切分成字符，再应用训练阶段获得的合并规则
*   经典模型：GPT, GPT-2, RoBERTa, BART, LLaMA, ChatGLM 等

**3.1. 训练阶段**
-------------

在训练环节，目标是给定语料，通过训练算法，生成合并规则和词表。 BPE 算法是从一个字符级别的词表为基础，合并 pair 并添加到词表中，逐步形成大词表。合并规则为选择相邻 pair 词频最大的进行合并。

下面我们进行手工的实现。

假定训练的语料 (已归一化处理) 为 4 个句子。

```
corpus = [
    "This is the Hugging Face Course.",
    "This chapter is about tokenization.",
    "This section shows several tokenizer algorithms.",
    "Hopefully, you will be able to understand how they are trained and generate tokens.",
]

```

首先进行预切分处理。这里采用 gpt2 的预切分逻辑。 具体会按照空格和标点进行切分，并且空格会保留成特殊的字符 “Ġ”。

```
from transformers import AutoTokenizer

# init pre tokenize function
gpt2_tokenizer = AutoTokenizer.from_pretrained("gpt2")
pre_tokenize_function = gpt2_tokenizer.backend_tokenizer.pre_tokenizer.pre_tokenize_str

# pre tokenize
pre_tokenized_corpus = [pre_tokenize_str(text) for text in corpus]


```

获得的 pre_tokenized_corpus 如下，每个单元分别为 [word, (start_index, end_index)]

```
[
    [('This', (0, 4)), ('Ġis', (4, 7)), ('Ġthe', (7, 11)), ('ĠHugging', (11, 19)), ('ĠFace', (19, 24)), ('ĠCourse', (24, 31)), ('.', (31, 32))], 
    [('This', (0, 4)), ('Ġchapter', (4, 12)), ('Ġis', (12, 15)), ('Ġabout', (15, 21)), ('Ġtokenization', (21, 34)), ('.', (34, 35))], 
    [('This', (0, 4)), ('Ġsection', (4, 12)), ('Ġshows', (12, 18)), ('Ġseveral', (18, 26)), ('Ġtokenizer', (26, 36)), ('Ġalgorithms', (36, 47)), ('.', (47, 48))], 
    [('Hopefully', (0, 9)), (',', (9, 10)), ('Ġyou', (10, 14)), ('Ġwill', (14, 19)), ('Ġbe', (19, 22)), ('Ġable', (22, 27)), ('Ġto', (27, 30)), ('Ġunderstand', (30, 41)), ('Ġhow', (41, 45)), ('Ġthey', (45, 50)), ('Ġare', (50, 54)), ('Ġtrained', (54, 62)), ('Ġand', (62, 66)), ('Ġgenerate', (66, 75)), ('Ġtokens', (75, 82)), ('.', (82, 83))]
]


```

进一步统计每个整词的词频

```
word2count = defaultdict(int)
for split_text in pre_tokenized_corpus:
    for word, _ in split_text:
        word2count[word] += 1


```

获得 word2count 如下

```
defaultdict(<class 'int'>, {'This': 3, 'Ġis': 2, 'Ġthe': 1, 'ĠHugging': 1, 'ĠFace': 1, 'ĠCourse': 1, '.': 4, 'Ġchapter': 1, 'Ġabout': 1, 'Ġtokenization': 1, 'Ġsection': 1, 'Ġshows': 1, 'Ġseveral': 1, 'Ġtokenizer': 1, 'Ġalgorithms': 1, 'Hopefully': 1, ',': 1, 'Ġyou': 1, 'Ġwill': 1, 'Ġbe': 1, 'Ġable': 1, 'Ġto': 1, 'Ġunderstand': 1, 'Ġhow': 1, 'Ġthey': 1, 'Ġare': 1, 'Ġtrained': 1, 'Ġand': 1, 'Ġgenerate': 1, 'Ġtokens': 1})


```

因为 BPE 是从字符级别的小词表，逐步合并成大词表，所以需要先获得字符级别的小词表。

```
vocab_set = set()
for word in word2count:
    vocab_set.update(list(word))
vocabs = list(vocab_set)


```

获得的初始小词表 vocabs 如下:

```
['i', 't', 'p', 'o', 'r', 'm', 'e', ',', 'y', 'v', 'Ġ', 'F', 'a', 'C', 'H', '.', 'f', 'l', 'u', 'c', 'T', 'k', 'h', 'z', 'd', 'g', 'w', 'n', 's', 'b']


```

基于小词表就可以对每个整词进行切分

```
word2splits = {word: [c for c in word] for word in word2count}

'This': ['T', 'h', 'i', 's'], 
'Ġis': ['Ġ', 'i', 's'], 
'Ġthe': ['Ġ', 't', 'h', 'e'], 
...
'Ġand': ['Ġ', 'a', 'n', 'd'], 
'Ġgenerate': ['Ġ', 'g', 'e', 'n', 'e', 'r', 'a', 't', 'e'], 
'Ġtokens': ['Ġ', 't', 'o', 'k', 'e', 'n', 's']


```

基于 word2splits 统计 vocabs 中相邻两个 pair 的词频 pair2count

```
def _compute_pair2score(word2splits, word2count):
    pair2count = defaultdict(int)
    for word, word_count in word2count.items():
        split = word2splits[word]
        if len(split) == 1:
            continue
        for i in range(len(split) - 1):
            pair = (split[i], split[i + 1])
            pair2count[pair] += word_count
    return pair2count


```

获得 pair2count 如下：

```
defaultdict(<class 'int'>, {('T', 'h'): 3, ('h', 'i'): 3, ('i', 's'): 5, ('Ġ', 'i'): 2, ('Ġ', 't'): 7, ('t', 'h'): 3, ..., ('n', 's'): 1})


```

统计当前频率最高的相邻 pair

```
def _compute_most_score_pair(pair2count):
    best_pair = None
    max_freq = None
    for pair, freq in pair2count.items():
        if max_freq is None or max_freq < freq:
            best_pair = pair
            max_freq = freq
    return best_pair


```

经过统计，当前频率最高的 pair 为: ('Ġ', 't')， 频率为 7 次。 将 ('Ġ', 't') 合并成一个词并添加到词表中。同时在合并规则中添加 ('Ġ', 't') 这条合并规则。

```
merge_rules = []
best_pair = self._compute_most_score_pair(pair2score)
vocabs.append(best_pair[0] + best_pair[1])
merge_rules.append(best_pair)


```

此时的 vocab 词表更新成:

```
['i', 't', 'p', 'o', 'r', 'm', 'e', ',', 'y', 'v', 'Ġ', 'F', 'a', 'C', 'H', '.', 'f', 'l', 'u', 'c', 'T', 'k', 'h', 'z', 'd', 'g', 'w', 'n', 's', 'b', 
'Ġt']


```

根据更新后的 vocab 重新对 word2count 进行切分。具体实现上，可以直接在旧的 word2split 上应用新的合并规则 ('Ġ', 't')

```
def _merge_pair(a, b, word2splits):
    new_word2splits = dict()
    for word, split in word2splits.items():
        if len(split) == 1:
            new_word2splits[word] = split
            continue
        i = 0
        while i < len(split) - 1:
            if split[i] == a and split[i + 1] == b:
                split = split[:i] + [a + b] + split[i + 2:]
            else:
                i += 1
        new_word2splits[word] = split
    return new_word2splits


```

从而获得新的 word2split

```
{'This': ['T', 'h', 'i', 's'], 
'Ġis': ['Ġ', 'i', 's'], 
'Ġthe': ['Ġt', 'h', 'e'], 
'ĠHugging': ['Ġ', 'H', 'u', 'g', 'g', 'i', 'n', 'g'],
...
'Ġtokens': ['Ġt', 'o', 'k', 'e', 'n', 's']}


```

可以看到新的 word2split 中已经包含了新的词 "Ġt"。

重复上述循环直到整个词表的大小达到预先设定的词表大小。

```
while len(vocabs) < vocab_size:
    pair2score = self._compute_pair2score(word2splits, word2count)
    best_pair = self._compute_most_score_pair(pair2score)
    vocabs.append(best_pair[0] + best_pair[1])
    merge_rules.append(best_pair)
    word2splits = self._merge_pair(best_pair[0], best_pair[1], word2splits)


```

假定最终词表的大小为 50，经过上述迭代后我们获得的词表和合并规则如下：

```
vocabs = ['i', 't', 'p', 'o', 'r', 'm', 'e', ',', 'y', 'v', 'Ġ', 'F', 'a', 'C', 'H', '.', 'f', 'l', 'u', 'c', 'T', 'k', 'h', 'z', 'd', 'g', 'w', 'n', 's', 'b', 'Ġt', 'is', 'er', 'Ġa', 'Ġto', 'en', 'Th', 'This', 'ou', 'se', 'Ġtok', 'Ġtoken', 'nd', 'Ġis', 'Ġth', 'Ġthe', 'in', 'Ġab', 'Ġtokeni', 'Ġtokeniz']

merge_rules = [('Ġ', 't'), ('i', 's'), ('e', 'r'), ('Ġ', 'a'), ('Ġt', 'o'), ('e', 'n'), ('T', 'h'), ('Th', 'is'), ('o', 'u'), ('s', 'e'), ('Ġto', 'k'), ('Ġtok', 'en'), ('n', 'd'), ('Ġ', 'is'), ('Ġt', 'h'), ('Ġth', 'e'), ('i', 'n'), ('Ġa', 'b'), ('Ġtoken', 'i'), ('Ġtokeni', 'z')]


```

至此我们就根据给定的语料完成了 BPE 分词器的训练。

**3.2. 推理阶段**
-------------

在推理阶段，给定一个句子，我们需要将其切分成一个 token 的序列。 具体实现上需要先对句子进行预分词并切分成字符级别的序列，然后根据合并规则进行合并。

```
def tokenize(self, text: str) -> List[str]:
    # pre tokenize
    words = [word for word, _ in self.pre_tokenize_str(text)]
    # split into char level
    splits = [[c for c in word] for word in words]
    # apply merge rules
    for merge_rule in self.merge_rules:
        for index, split in enumerate(splits):
            i = 0
            while i < len(split) - 1:
                if split[i] == merge_rule[0] and split[i + 1] == merge_rule[1]:
                    split = split[:i] + ["".join(merge_rule)] + split[i + 2:]
                else:
                    i += 1
            splits[index] = split
    return sum(splits, [])


```

例如

```
>>> tokenize("This is not a token.")
>>> ['This', 'Ġis', 'Ġ', 'n', 'o', 't', 'Ġa', 'Ġtoken', '.']


```

**3.3. BBPE**
-------------

![](https://pic3.zhimg.com/v2-6be86b9910c22e8ef6a6ef0f7d3c337e_b.jpg)

通过这种方式可以更好的处理跨语言和不常见字符的特殊问题 (例如，颜文字)，相比传统的 BPE 更节省词表空间（同等词表大小效果更好），每个 token 也能获得更充分的训练。

但是在解码阶段，一个 byte 序列可能解码后不是一个合法的字符序列，这里需要采用动态规划的算法进行解码，使其能解码出尽可能多的合法字符。具体算法如下： 假定$f(k)$f(k) 表示字符序列$B_{1,k}$B_{1,k} 最大能解码的合法字符数量，$f(k)$f(k) 有最优的子结构：

$f(k) = max_{t=1,2,3,4}{f(k-t) + g(k-t+1, k)}$f(k) = max_{t=1,2,3,4}{f(k-t) + g(k-t+1, k)}

这里如果$B_{i,j}$B_{i,j} 为一个合法字符$g(i,j) = 1$g(i,j) = 1，否则$g(i,j) = 0$g(i,j) = 0。

**4. WordPiece**
----------------

WordPiece 分词与 BPE 非常类似，只是在训练阶段合并 pair 的策略不是 pair 的频率而是互信息。

$socre = log(p(ab)) - (log(p(a)) + log(p(b))) = log(p(ab)/p(a)p(b))$socre = log(p(ab)) - (log(p(a)) + log(p(b))) = log(p(ab)/p(a)p(b))

这里的动机是一个 pair 的频率很高，但是其中 pair 的一部分的频率更高，这时候不一定需要进行该 pair 的合并。 而如果一个 pair 的频率很高，并且这个 pair 的两个部分都是只出现在这个 pair 中，就说明这个 pair 很值得合并。

*   训练方法：从字符级的小词表出发，训练产生合并规则以及一个词表
*   编码方法：将文本切分成词，对每个词在词表中进行最大前向匹配
*   经典模型：BERT 及其系列 DistilBERT，MobileBERT 等

**4.1. 训练阶段**
-------------

在训练环节，给定语料，通过训练算法，生成最终的词表。 WordPiece 算法也是从一个字符级别的词表为基础，逐步扩充成大词表。合并规则为选择相邻 pair 互信息最大的进行合并。

下面进行具体手工实现。

假定训练的语料 (已归一化处理) 为

```
corpus = [
    "This is the Hugging Face Course.",
    "This chapter is about tokenization.",
    "This section shows several tokenizer algorithms.",
    "Hopefully, you will be able to understand how they are trained and generate tokens.",
]


```

首先进行预切分处理。这里采用 BERT 的预切分逻辑。具体会按照空格和标点进行切分。

```
from transformers import AutoTokenizer

# init pre tokenize function
bert_tokenizer = AutoTokenizer.from_pretrained("bert-base-cased")
pre_tokenize_function = bert_tokenizer.backend_tokenizer.pre_tokenizer.pre_tokenize_str

# pre tokenize
pre_tokenized_corpus = [pre_tokenize_str(text) for text in corpus]


```

获得的 pre_tokenized_corpus 如下，每个单元分别为 [word, (start_index, end_index)]

```
[
    [('This', (0, 4)), ('is', (5, 7)), ('the', (8, 11)), ('Hugging', (12, 19)), ('Face', (20, 24)), ('Course', (25, 31)), ('.', (31, 32))], 
    [('This', (0, 4)), ('chapter', (5, 12)), ('is', (13, 15)), ('about', (16, 21)), ('tokenization', (22, 34)), ('.', (34, 35))], 
    [('This', (0, 4)), ('section', (5, 12)), ('shows', (13, 18)), ('several', (19, 26)), ('tokenizer', (27, 36)), ('algorithms', (37, 47)), ('.', (47, 48))], 
    [('Hopefully', (0, 9)), (',', (9, 10)), ('you', (11, 14)), ('will', (15, 19)), ('be', (20, 22)), ('able', (23, 27)), ('to', (28, 30)), ('understand', (31, 41)), ('how', (42, 45)), ('they', (46, 50)), ('are', (51, 54)), ('trained', (55, 62)), ('and', (63, 66)), ('generate', (67, 75)), ('tokens', (76, 82)), ('.', (82, 83))]
]


```

进一步统计词频

```
word2count = defaultdict(int)
for split_text in pre_tokenized_corpus:
    for word, _ in split_text:
        word2count[word] += 1


```

获得 word2count 如下

```
defaultdict(<class 'int'>, {'This': 3, 'is': 2, 'the': 1, 'Hugging': 1, 'Face': 1, 'Course': 1, '.': 4, 'chapter': 1, 'about': 1, 'tokenization': 1, 'section': 1, 'shows': 1, 'several': 1, 'tokenizer': 1, 'algorithms': 1, 'Hopefully': 1, ',': 1, 'you': 1, 'will': 1, 'be': 1, 'able': 1, 'to': 1, 'understand': 1, 'how': 1, 'they': 1, 'are': 1, 'trained': 1, 'and': 1, 'generate': 1, 'tokens': 1})


```

因为 WordPiece 同样是从字符级别的小词表，逐步合并成大词表，所以先获得字符级别的小词表。注意这里如果字符不是不一个词的开始，需要添加上特殊字符 "##"。

```
vocab_set = set()
for word in word2count:
    vocab_set.add(word[0])
    vocab_set.update(['##' + c for c in word[1:]])
vocabs = list(vocab_set)


```

获得的初始小词表 vocabs 如下:

```
['##a', '##b', '##c', '##d', '##e', '##f', '##g', '##h', '##i', '##k', '##l', '##m', '##n', '##o', '##p', '##r', '##s', '##t', '##u', '##v', '##w', '##y', '##z', ',', '.', 'C', 'F', 'H', 'T', 'a', 'b', 'c', 'g', 'h', 'i', 's', 't', 'u', 'w', 'y']


```

基于小词表对每个词进行切分

```
word2splits = {word: [word[0]] + ['##' + c for c in word[1:]] for word in word2count}

{'This': ['T', '##h', '##i', '##s'], 
'is': ['i', '##s'], 
'the': ['t', '##h', '##e'], 
'Hugging': ['H', '##u', '##g', '##g', '##i', '##n', '##g'], 
...
'generate': ['g', '##e', '##n', '##e', '##r', '##a', '##t', '##e'], 
'tokens': ['t', '##o', '##k', '##e', '##n', '##s']}


```

进一步统计 vocabs 中相邻两个 pair 的互信息

```
def _compute_pair2score(word2splits, word2count):
    """
    计算每个pair的分数
    score=(freq_of_pair)/(freq_of_first_element×freq_of_second_element)
    :return:
    """
    vocab2count = defaultdict(int)
    pair2count = defaultdict(int)
    for word, word_count in word2count.items():
        splits = word2splits[word]
        if len(splits) == 1:
            vocab2count[splits[0]] += word_count
            continue
        for i in range(len(splits) - 1):
            pair = (splits[i], splits[i + 1])
            vocab2count[splits[i]] += word_count
            pair2count[pair] += word_count
        vocab2count[splits[-1]] += word_count
    scores = {
        pair: freq / (vocab2count[pair[0]] * vocab2count[pair[1]])
        for pair, freq in pair2count.items()
    }
    return scores


```

获得每个 pair 的互信息如下：

```
{('T', '##h'): 0.125, 
('##h', '##i'): 0.03409090909090909, 
('##i', '##s'): 0.02727272727272727, 
('a', '##b'): 0.2,
...
('##n', '##s'): 0.00909090909090909}


```

统计出互信息最高的相邻 pair

```
def _compute_most_score_pair(pair2score):
    best_pair = None
    max_score = None
    for pair, score in pair2score.items():
        if max_score is None or max_score < score:
            best_pair = pair
            max_score = score
    return best_pair


```

此时互信息最高的 pair 为: ('a', '##b') 将 ('a', '##b') 合并成一个词'ab'并添加到词表中

```
best_pair = self._compute_most_score_pair(pair2score)
vocabs.append(best_pair[0] + best_pair[1])


```

这样 vocab 词表更新成:

```
['##a', '##b', '##c', '##d', '##e', '##f', '##g', '##h', '##i', '##k', '##l', '##m', '##n', '##o', '##p', '##r', '##s', '##t', '##u', '##v', '##w', '##y', '##z', ',', '.', 'C', 'F', 'H', 'T', 'a', 'b', 'c', 'g', 'h', 'i', 's', 't', 'u', 'w', 'y', 
'ab']


```

根据更新的 vocab 重新对 word2count 进行切分。

```
def _merge_pair(a, b, word2splits):
    new_word2splits = dict()
    for word, split in word2splits.items():
        if len(split) == 1:
            new_word2splits[word] = split
            continue
        i = 0
        while i < len(split) - 1:
            if split[i] == a and split[i + 1] == b:
                merge = a + b[2:] if b.startswith("##") else a + b
                split = split[:i] + [merge] + split[i + 2:]
            else:
                i += 1
        new_word2splits[word] = split
    return new_word2splits


```

获得新的 word2split

```
{'This': ['T', '##h', '##i', '##s'], 
'is': ['i', '##s'], 'the': ['t', '##h', '##e'], 
'Hugging': ['H', '##u', '##g', '##g', '##i', '##n', '##g'], 
'about': ['ab', '##o', '##u', '##t'], 
'tokens': ['t', '##o', '##k', '##e', '##n', '##s']}


```

可以看到新的 word2split 中已经包含了新的词 "ab"。

重复上述步骤，直到整个词表的大小达到预先设定的词表大小。

```
while len(vocabs) < vocab_size:
    pair2score = self._compute_pair2score(word2splits, word2count)
    best_pair = self._compute_most_score_pair(pair2score)
    word2splits = self._merge_pair(best_pair[0], best_pair[1], word2splits)
    new_token = best_pair[0] + best_pair[1][2:] if best_pair[1].startswith('##') else best_pair[1]
    vocabs.append(new_token)


```

假定最终词表的大小为 70，经过上述迭代后我们获得的词表如下：

```
vocabs = ['##a', '##b', '##c', '##ct', '##d', '##e', '##f', '##fu', '##ful', '##full', '##fully', '##g', '##h', '##hm', '##i', '##k', '##l', '##m', '##n', '##o', '##p', '##r', '##s', '##t', '##thm', '##thms', '##u', '##ut', '##v', '##w', '##y', '##z', '##za', '##zat', ',', '.', 'C', 'F', 'Fa', 'Fac', 'H', 'Hu', 'Hug', 'Hugg', 'T', 'Th', 'a', 'ab', 'b', 'c', 'ch', 'cha', 'chap', 'chapt', 'g', 'h', 'i', 'is', 's', 'sh', 't', 'th', 'u', 'w', 'y', '[CLS]', '[MASK]', '[PAD]', '[SEP]', '[UNK]']


```

注意词表中添加了特殊的 token：[CLS], [MASK], [PAD], [SEP], [UNK] 至此我们就根据给定的语料完成了 WordPiece 分词器的训练。

**4.2. 推理阶段**
-------------

在推理阶段，给定一个句子，需要将其切分成一个 token 的序列。 具体实现上需要先对句子进行预分词，然后对每个词进行在词表中进行最大前向的匹配。如果词表中不存在则为 UNK。

```
def _encode_word(self, word):
    tokens = []
    while len(word) > 0:
        i = len(word)
        while i > 0 and word[:i] not in self.vocabs:
            i -= 1
        if i == 0:
            return ["[UNK]"]
        tokens.append(word[:i])
        word = word[i:]
        if len(word) > 0:
            word = f"##{word}"
    return tokens

def tokenize(self, text):
    words = [word for word, _ in self.pre_tokenize_str(text)]
    encoded_words = [self._encode_word(word) for word in words]
    return sum(encoded_words, [])


```

例如

```
>>> tokenize("This is the Hugging Face course!")
>>> ['Th', '##i', '##s', 'is', 'th', '##e', 'Hugg', '##i', '##n', '##g', 'Fac', '##e', 'c', '##o', '##u', '##r', '##s', '##e', '[UNK]']


```

**5. Unigram**
--------------

Unigram 分词与 BPE 和 WordPiece 不同，是基于一个大词表逐步裁剪成一个小词表。 通过 Unigram 语言模型计算删除不同 subword 造成的损失来衡量 subword 的重要性，保留重要性较高的子词。

*   训练方法：从包含字符和全部子词的大词表出发，通过训练逐步裁剪出一个小词表，并且每个词都有自己的分数。
*   编码方法：将文本切分成词，对每个词基于 Viterbi 算法求解出最佳解码路径。
*   经典模型：AlBERT, T5, mBART, Big Bird, XLNet

**5.1. 训练阶段**
-------------

在训练环节，目标是给定语料，通过训练算法，生成最终的词表，并且每个词有自己的概率值。 Unigram 算法是从大词表为基础，逐步裁剪成小词表。裁剪规则是根据 Unigram 语言模型的打分依次裁剪重要度相对较低的词。

下面进行具体手工实现。

假定训练的语料 (已归一化处理) 为

```
corpus = [
    "This is the Hugging Face Course.",
    "This chapter is about tokenization.",
    "This section shows several tokenizer algorithms.",
    "Hopefully, you will be able to understand how they are trained and generate tokens.",
]


```

首先进行预切分处理。这里采用 xlnet 的预切分逻辑。具体会按照空格进行切分，标点不会切分。并且空格会保留成特殊字符 "▁"，句子开头也会添加特殊字符 "▁"。

```
from transformers import AutoTokenizer

# init pre tokenize function
xlnet_tokenizer = AutoTokenizer.from_pretrained("xlnet-base-cased")
pre_tokenize_function = xlnet_tokenizer.backend_tokenizer.pre_tokenizer.pre_tokenize_str

# pre tokenize
pre_tokenized_corpus = [pre_tokenize_str(text) for text in corpus]


```

获得的 pre_tokenized_corpus 如下，每个单元分别为 [word, (start_index, end_index)]

```
[
    [('▁This', (0, 4)), ('▁is', (5, 7)), ('▁the', (8, 11)), ('▁Hugging', (12, 19)), ('▁Face', (20, 24)), ('▁Course.', (25, 32))], 
    [('▁This', (0, 4)), ('▁chapter', (5, 12)), ('▁is', (13, 15)), ('▁about', (16, 21)), ('▁tokenization.', (22, 35))], 
    [('▁This', (0, 4)), ('▁section', (5, 12)), ('▁shows', (13, 18)), ('▁several', (19, 26)), ('▁tokenizer', (27, 36)), ('▁algorithms.', (37, 48))], 
    [('▁Hopefully,', (0, 10)), ('▁you', (11, 14)), ('▁will', (15, 19)), ('▁be', (20, 22)), ('▁able', (23, 27)), ('▁to', (28, 30)), ('▁understand', (31, 41)), ('▁how', (42, 45)), ('▁they', (46, 50)), ('▁are', (51, 54)), ('▁trained', (55, 62)), ('▁and', (63, 66)), ('▁generate', (67, 75)), ('▁tokens.', (76, 83))]
]


```

进一步统计词频

```
word2count = defaultdict(int)
for split_text in pre_tokenized_corpus:
    for word, _ in split_text:
        word2count[word] += 1


```

获得 word2count 如下

```
defaultdict(<class 'int'>, {'▁This': 3, '▁is': 2, '▁the': 1, '▁Hugging': 1, '▁Face': 1, '▁Course.': 1, '▁chapter': 1, '▁about': 1, '▁tokenization.': 1, '▁section': 1, '▁shows': 1, '▁several': 1, '▁tokenizer': 1, '▁algorithms.': 1, '▁Hopefully,': 1, '▁you': 1, '▁will': 1, '▁be': 1, '▁able': 1, '▁to': 1, '▁understand': 1, '▁how': 1, '▁they': 1, '▁are': 1, '▁trained': 1, '▁and': 1, '▁generate': 1, '▁tokens.': 1})


```

统计词表的全部子词，并统计词频。取前 300 个词，构成最初的大词表。为了避免 OOV，char 级别的词均需要保留。

```
char2count = defaultdict(int)
sub_word2count = defaultdict(int)
for word, count in word2count.items():
    for i in range(len(word)):
        char2count[word[i]] += count
        for j in range(i + 2, len(word) + 1):
            sub_word2count[word[i:j]] += count
sorted_sub_words = sorted(sub_word2count.items(), key=lambda x: x[1], reverse=True)
# init a large vocab with 300
tokens = list(char2count.items()) + sorted_sub_words[: 300 - len(char2count)]


```

获得的初始小词表 vocabs 如下:

```
[('▁', 31), ('T', 3), ('h', 9), ('i', 13), ('s', 13), ...,  ('several', 1)]


```

进一步统计每个子词的概率，并转换成 Unigram 里的 loss 贡献

```
token2count = {token: count for token, count in tokens}
total_count = sum([count for token, count in token2count.items()])
model = {token: -log(count / total_count) for token, count in token2count.items()}

model = {
    '▁': 2.952892114877499, 
    'T': 5.288267030694535, 
    'h': 4.189654742026425, 
    ..., 
    'sever': 6.386879319362645, 
    'severa': 6.386879319362645, 
    'several': 6.386879319362645
}


```

基于每个子词的 loss 以及 Viterbi 算法就可以求解出，输入的一个词的最佳分词路径。即整体语言模型的 loss 最小。词的长度为 N，解码的时间复杂度为 O(N^2)。

```
def _encode_word(word, model):
    best_segmentations = [{"start": 0, "score": 1}] + [{"start": None, "score": None} for _ in range(len(word))]
    for start_idx in range(len(word)):
        # This should be properly filled by the previous steps of the loop
        best_score_at_start = best_segmentations[start_idx]["score"]
        for end_idx in range(start_idx + 1, len(word) + 1):
            token = word[start_idx:end_idx]
            if token in model and best_score_at_start is not None:
                score = model[token] + best_score_at_start
                # If we have found a better segmentation (lower score) ending at end_idx
                if (
                        best_segmentations[end_idx]["score"] is None
                        or best_segmentations[end_idx]["score"] > score
                ):
                    best_segmentations[end_idx] = {"start": start_idx, "score": score}
    segmentation = best_segmentations[-1]
    if segmentation["score"] is None:
        # We did not find a tokenization of the word -> unknown
        return ["<unk>"], None
    score = segmentation["score"]
    start = segmentation["start"]
    end = len(word)
    tokens = []
    while start != 0:
        tokens.insert(0, word[start:end])
        next_start = best_segmentations[start]["start"]
        end = start
        start = next_start
    tokens.insert(0, word[start:end])
    return tokens, score


```

例如：

```
>>> tokenize("This")
>>> (['This'], 6.288267030694535)
>>> tokenize("this")
>>>(['t', 'his'], 10.03608902044192)


```

基于上述的函数，可以获得任一个词的分词路径，以及 loss。这样就可以计算整个语料上的 loss。

```
def _compute_loss(self, model, word2count):
    loss = 0
    for word, freq in word2count.items():
        _, word_loss = self._encode_word(word, model)
        loss += freq * word_loss
    return loss


```

尝试移除 model 中的一个子词，并计算移除后新的 model 在全部语料上的 loss，从而获得这个子词的 score，即删除这个子词使得 loss 新增的量。

```
def _compute_scores(self, model, word2count):
    scores = {}
    model_loss = self._compute_loss(model, word2count)
    for token, score in model.items():
        # We always keep tokens of length 1
        if len(token) == 1:
            continue
        model_without_token = copy.deepcopy(model)
        _ = model_without_token.pop(token)
        scores[token] = self._compute_loss(model_without_token, word2count) - model_loss
    return scores

scores = self._compute_scores(model, word2count)


```

为了提升迭代效率，批量删除前 10% 的结果，即让整体 loss 增量最小的前 10% 的词。(删除这些词对整体 loss 的影响不大。)

```
sorted_scores = sorted(scores.items(), key=lambda x: x[1])
# Remove percent_to_remove tokens with the lowest scores.
for i in range(int(len(model) * 0.1)):
    _ = token2count.pop(sorted_scores[i][0])


```

获得新的词表后，重新计算每个词的概率，获得新的模型。并重复以上步骤，直到裁剪到词表大小符合要求。

```
while len(model) > vocab_size:
    scores = self._compute_scores(model, word2count)
    sorted_scores = sorted(scores.items(), key=lambda x: x[1])
    # Remove percent_to_remove tokens with the lowest scores.
    for i in range(int(len(model) * percent_to_remove)):
        _ = token2count.pop(sorted_scores[i][0])
    total_count = sum([freq for token, freq in token2count.items()])
    model = {token: -log(count / total_count) for token, count in token2count.items()}


```

假定预设的词表的大小为 100，经过上述迭代后我们获得词表如下:

```
model = {
    '▁': 2.318585434340487, 
    'T': 4.653960350157523, 
    'h': 3.5553480614894135, 
    'i': 3.1876232813640963, 
    ...
    'seve': 5.752572638825633, 
    'sever': 5.752572638825633, 
    'severa': 5.752572638825633, 
    'several': 5.752572638825633
}


```

**5.2. 推理阶段**
-------------

在推理阶段，给定一个句子，需要将其切分成一个 token 的序列。 具体实现上先对句子进行预分词，然后对每个词基于 Viterbi 算法进行解码。

```
def tokenize(self, text):
    words = [word for word, _ in self.pre_tokenize_str(text)]
    encoded_words = [self._encode_word(word, self.model)[0] for word in words]
    return sum(encoded_words, [])


```

例如

```
>>> tokenize("This is the Hugging Face course!")
>>> ['▁This', '▁is', '▁the', '▁Hugging', '▁Face', '▁', 'c', 'ou', 'r', 's', 'e', '.']


```

基于 Viterbi 的切分获得的是最佳切分，基于 unigram 可以实现一个句子的多种切分方式，并且可以获得每种切分路径的打分。

**6. SentencePiece**
--------------------

[SentencePiece](https://link.zhihu.com/?target=https%3A//github.com/google/sentencepiece) 是 Google 出的一个分词工具:

*   内置 BPE，Unigram，char 和 word 的分词方法
*   无需预分词，以 unicode 方式直接编码整个句子，空格会被特殊编码为▁
*   相比传统实现进行优化，分词速度速度更快

当前主流的大模型都是基于 sentencepiece 实现，例如 ChatGLM 的 tokenizer。

```
...
class TextTokenizer:
    def __init__(self, model_path):
        self.sp = spm.SentencePieceProcessor()
        self.sp.Load(model_path)
        self.num_tokens = self.sp.vocab_size()

    def encode(self, text):
        return self.sp.EncodeAsIds(text)

    def decode(self, ids: List[int]):
        return self.sp.DecodeIds(ids)
...


```

[https://huggingface.co/THUDM/chatglm-6b/blob/main/tokenization_chatglm.py#L21](https://link.zhihu.com/?target=https%3A//huggingface.co/THUDM/chatglm-6b/blob/main/tokenization_chatglm.py%23L21)

**6.1. byte 回退**
----------------

当 SentencePiece 在训练 BPE 的时开启`--byte_fallback`, 在效果上类似 BBPE，遇到 UNK 会继续按照 byte 进行进一步的切分。参见：[https://github.com/google/sentencepiece/issues/621](https://link.zhihu.com/?target=https%3A//github.com/google/sentencepiece/issues/621) 具体实现上是将 <0x00> ... <0xFF > 这 256 个 token 添加到词表中。

分析 ChatGLM 的模型，可以发现 ChatGLM 就是开启了`--byte_fallback`

```
from sentencepiece import sentencepiece_model_pb2

m = sentencepiece_model_pb2.ModelProto()
with open('chatglm-6b/ice_text.model', 'rb') as f:
    m.ParseFromString(f.read())
print('ChatGLM tokenizer\n\n'+str(m.trainer_spec))


```

output：

```
ChatGLM tokenizer

input: "/root/train_cn_en.json"
model_prefix: "new_ice_unigram"
vocab_size: 130000
character_coverage: 0.9998999834060669
split_digits: true
user_defined_symbols: "<n>"
byte_fallback: true
pad_id: 3
train_extremely_large_corpus: true


```

可以看到`byte_fallback: true`

同样的方法，可以验证 LLaMA, ChatGLM-6B, Baichuan 这些大模型都是基于 sentencepiece 实现的 BPE 的分词算法，并且采用 byte 回退。

**参考**
------

*   [HuggingFace tokenizer tutorial](https://link.zhihu.com/?target=https%3A//huggingface.co/learn/nlp-course/chapter6/1)
*   [google/sentencepiece](https://link.zhihu.com/?target=https%3A//github.com/google/sentencepiece/)
*   [BPE: Neural Machine Translation of Rare Words with Subword Units](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1508.07909)
*   [BBPE: Neural Machine Translation with Byte-Level Subwords](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1909.03341.pdf)
*   [Unigram: Subword Regularization: Improving Neural Network Translation Models with Multiple Subword Candidates](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1804.10959)
*   [SentencePiece: A simple and language independent subword tokenizer and detokenizer for Neural Text Processing](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1808.06226)