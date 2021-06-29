> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?src=11×tamp=1624949883&ver=3159&signature=-PE0kTN84XjjwA1l2F7o8qOsQHC8rt48PqCBycsuQ1T4mYykO6RAjQRIRrNv5gusz5hAKeIVSnRsPJFJ95du8AmPBNG855hhKFbUfsNu0cw0c99S2wfARwTQTEPsJ9Xl&new=1)

给你一个用字符数组 tasks 表示的 CPU 需要执行的任务列表。其中每个字母表示一种不同种类的任务。任务可以以任意顺序执行，并且每个任务都可以在 1 个单位时间内执行完。在任何一个单位时间，CPU 可以完成一个任务，或者处于待命状态。

输入：tasks = ["A","A","A","B","B","B"], n = 0

输出：6

解释：在这种情况下，任何大小为 6 的排列都可以满足要求，因为 n = 0

["A","A","A","B","B","B"]

["A","B","A","B","A","B"]

["B","B","B","A","A","A"]

...

诸如此类

示例 3：

输入：tasks = ["A","A","A","A","A","A","B","C","D","E","F","G"], n = 2

输出：16

解释：一种可能的解决方案是：

     A -> B -> C -> A -> D -> E -> A -> F -> G -> A -> (待命) -> (待命) -> A -> (待命) -> (待命) -> A

提示：

1 <= task.length <= 104

tasks[i] 是大写英文字母

n 的取值范围为 [0, 100]

解题思路：

1，如果连续 n+1 个位置没有重复的任务，就不需要待命，所以需不需要待命，由次数最多的任务决定。  

2，我们假设，次数最多的任务的次数为 maxCnt，可以建模一个二维表格，宽度为 n+1，高度为 maxCnt  

![](https://mmbiz.qpic.cn/mmbiz_png/vcGM5GwZRgz4psy3IlNEHSPDXFXDWUMmysG1z1IIicOkym3ogDHJwl43icCWEwowTvXjHaSBa99uTyH4goqfTXxg/640?wx_fmt=png)

3，分两种情况：  

A，不同的任务可以将（maxCnt-1）*（n+1）的位置填满，这个时候，没有空的位置，也就是没有任务需要待命

B，不同的任务，不能将（maxCnt-1）*（n+1）的位置填满，这个时候，空的位置，就是待命状态。  

4，针对情况 A，说明不需要待命，所以需要时间就是任务数  

5，针对情况 B，不考虑 maxCnt-1 行，第 maxCnt 行的任务数，就是次数为 maxCnt 的任务数量 maxCntNum。所以总的次数为（maxCnt-1）*（n+1）+maxCntNum

倒数第二行开始，按照反向列优先的顺序（即先放入靠左侧的列，同一列中先放入下方的行），依次放入每一种任务，并且同一种任务需要连续地填入。对于任意一种任务而言，一定不会被放入同一行两次（否则说明该任务的执行次数大于等于 maxCnt），并且由于我们是按照列优先的顺序放入这些任务，因此任意两个相邻的任务之间要么间隔 n（例如上图中位于同一列的相同任务），要么间隔 n+1

6，结果是两者中较大者  

代码实现

```
func leastInterval(tasks []byte, n int) int {
   m:=make(map[byte]int)
   for _,t:=range tasks{
       m[t]++
   }
   maxCnt,maxCntNum:=0,0
   for _,c:=range m{
       if c>maxCnt{
           maxCnt=c
           maxCntNum=1
       }else if c==maxCnt{
           maxCntNum++
       }
   }
   if num:=(maxCnt-1)*(n+1)+maxCntNum;num>len(tasks){
       return num
   }
   return  len(tasks)
}

```