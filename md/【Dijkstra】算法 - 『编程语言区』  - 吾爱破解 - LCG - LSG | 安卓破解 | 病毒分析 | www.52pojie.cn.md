> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1450741&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline) ![](https://avatar.52pojie.cn/data/avatar/001/38/02/63_avatar_middle.jpg)Lthero  **迪克斯特拉算法**  
**非教学帖子，只作自我记录  
**  
**详解**  
**https://www.codespeedy.com/how-to-implement-dijkstras-shortest-path-algorithm-in-python/**  
**wiki**  
**https://en.m.wikipedia.org/wiki/Dijkstra%27s_algorithm**  
[Python] _纯文本查看_ _复制代码_

```
graph = {
    'A': {'B': 1, 'C': 4, 'D': 2},
    'B': {'A': 9, 'E': 5},
    'C': {'A': 4, 'F': 15},
    'D': {'A': 10, 'F': 7},
    'E': {'B': 3, 'J': 7},
    'F': {'C': 11, 'D': 14, 'K': 3, 'G': 9},
    'G': {'F': 12, 'I': 4},
    'H': {'J': 13},
    'I': {'G': 6, 'J': 7},
    'J': {'H': 2, 'I': 4},
    'K': {'F': 6}
}
source = input('输入起点 ')
# 记录来路  起点来路设置为起点
came_from = {source:source}
# 记录已经遍历的结点
passed_points = [source]
# 记录起点到其它结点的最小值
queue = {}
for i in graph:
    queue[i] = float('inf')
    came_from[i] = None
# 起点设置为0
queue[source] = 0
# 排序
queue = sorted(queue.items(), key=lambda x: x[1])
while queue:
    # pop出边权重最小的结点
    min_point = queue.pop(0)
    # 格式转换，下面取出数据
    queue = dict(queue)
    # 遍历这个结点的邻接表
    for i in graph[min_point[0]]:
        # 如果i还没遍历
        if i not in passed_points:
            # 计算如果走这个结点的花费
            comp = min_point[1] + graph[min_point[0]][i]
            # 如果这个花费更小（比原来的路）
            if queue[i] > comp:
                # 更新花费
                queue[i] = comp
                # 更新来路
                came_from[i] = (min_point[0], comp)
                # 当前结点放到已经遍历的中
                passed_points.append(i)
    # 重新排序
    queue = sorted(queue.items(), key=lambda x: x[1])
# print(came_from)
dest = input('输入终点 ')
final_path = []
p = dest
print('总共花费 ', came_from[p][1], ' 单位')
 
# 输出结果
while came_from[p] != source:
    final_path.append(came_from[p][0])
    p = came_from[p][0]
for i in final_path[::-1]:
    print(i, end=' ')
```

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)小小宇 6  还是 Floyd 简单些 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) ljb 牛 爱了，爱了![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)nanaqilin  好像听说过这个算法，但是太久了不知道做啥用的了 ![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif) lotus136 小白有点看不懂。![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)machinemaker  数据结构 oh 头疼![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)月清晖  哈哈 9 年前数学建模用过这个算法，特么 9 年了。。。。