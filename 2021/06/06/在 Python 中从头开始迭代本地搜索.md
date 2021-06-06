> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjUyNDQ5OA==&mid=2653575829&idx=1&sn=23a344808fbfb887e12568d0e4cb81a2&chksm=806e7428b719fd3e169c9d66eff549b8c00cfb10163c735ab784c2e8f53fccbabb35e848f556&mpshare=1&scene=1&srcid=0604iYMFgfNzyQKscUOXCG3i&sharer_sharetime=1622819233361&sharer_shareid=7fece245937ac96f04f0fb8e1311fff1#rd)

迭代局部搜索是一种随机全局优化算法。它涉及将本地搜索算法重复应用于先前找到的好的解决方案的修改版本。这样，它就像是具有随机重启算法的随机爬山的巧妙版本。

该算法背后的直觉是，随机重新启动可以帮助找到问题中的许多局部最优值，并且更好的局部最优值通常接近于其他局部最优值。因此，对现有局部最优值的适度扰动可能会为优化问题找到更好甚至最好的解决方案。

在本教程中，您将发现如何从头开始实现迭代的本地搜索算法。完成本教程后，您将知道：

*   迭代本地搜索是一种随机全局搜索优化算法，它是具有随机重启功能的随机爬山的更智能版本。
    
*   如何从头开始随机重启随机爬山。
    
*   如何实现并将迭代的局部搜索算法应用于非线性目标函数。
    

**教程概述**

本教程分为五个部分。他们是：

*   什么是迭代本地搜索
    
*   客观目标函数
    
*   随机爬山算法
    
*   随机重新开始的随机爬山
    
*   迭代局部搜索算法
    

**什么是迭代本地搜索**

迭代本地搜索（简称 ILS）是一种随机的全局搜索优化算法。它与随机爬山和随机爬山随机开始有关。

随机爬山是一种本地搜索算法，它涉及对现有解决方案进行随机修改，并且仅当修改产生比当前工作解决方案更好的结果时，才接受修改。

通常，本地搜索算法会陷入本地最优状态。解决此问题的一种方法是从新的随机选择的起点重新开始搜索。重新启动过程可以执行多次，也可以在固定数量的功能评估之后触发，或者在给定数量的算法迭代中看不到进一步的改善时，可以触发重新启动过程。该算法称为随机重新启动的随机爬山。

迭代的本地搜索类似于具有随机重启的随机爬坡，除了不是为每次重启选择随机的起点，而是根据迄今为止在更广泛的搜索中找到的最佳点的修改版本来选择一个点。到目前为止，最佳解决方案的扰动就像是搜索空间中向新区域的大幅跃迁，而随机爬山算法产生的扰动要小得多，仅限于搜索空间的特定区域。这允许在两个级别上执行搜索。爬山算法是一种本地搜索，用于从特定的候选解决方案或搜索空间区域中获取最大收益，并且重新启动方法允许探索搜索空间的不同区域。这样，迭代局部搜索算法可在搜索空间中探索多个局部最优，从而增加了定位全局最优的可能性。尽管可以通过在搜索空间中使用不同的步长将其应用于连续功能优化，但迭代局部搜索是针对组合优化问题（如旅行推销员问题（TSP））提出的：爬坡的步幅较小，爬坡的步幅较大随机重启。既然我们熟悉了迭代本地搜索算法，那么让我们探索如何从头开始实现该算法。

**客观目标函数**

首先，让我们定义一个渠道优化问题，作为实现 “迭代本地搜索” 算法的基础。Ackley 函数是多模式目标函数的一个示例，该函数具有单个全局最优值和多个局部最优值，可能会卡住局部搜索。因此，需要全局优化技术。这是一个二维目标函数，其全局最佳值为[0,0]，其值为 0.0。下面的示例实现了 Ackley，并创建了一个三维表面图，显示了全局最优值和多个局部最优值。

```
# ackley multimodal function
from numpy import arange
from numpy import exp
from numpy import sqrt
from numpy import cos
from numpy import e
from numpy import pi
from numpy import meshgrid
from matplotlib import pyplot
from mpl_toolkits.mplot3d import Axes3D
 
# objective function
def objective(x, y):
 return -20.0 * exp(-0.2 * sqrt(0.5 * (x**2 + y**2))) - exp(0.5 * (cos(2 * pi * x) + cos(2 * pi * y))) + e + 20
 
# define range for input
r_min, r_max = -5.0, 5.0
# sample input range uniformly at 0.1 increments
xaxis = arange(r_min, r_max, 0.1)
yaxis = arange(r_min, r_max, 0.1)
# create a mesh from the axis
x, y = meshgrid(xaxis, yaxis)
# compute targets
results = objective(x, y)
# create a surface plot with the jet color scheme
figure = pyplot.figure()
axis = figure.gca(projection='3d')
axis.plot_surface(x, y, results, cmap='jet')
# show the plot
pyplot.show()
```

运行示例将创建 Ackley 函数的表面图，以显示大量的局部最优值。

![](https://mmbiz.qpic.cn/mmbiz_png/5mt0ewv9OS3OQNs8sT5RRTfTC2EsXN7FGerCffkviaSsj7snWPWG61nCVBGMXsSf22U3vW1vmrpxUKjX6BAQNvQ/640?wx_fmt=png)

我们将以此为基础来实现和比较简单的随机爬山算法，随机重启的随机爬山算法以及最终迭代的本地搜索。我们希望随机爬山算法容易陷入局部极小值。我们希望随机爬山并重新启动可以找到许多本地最小值，并且如果配置得当，我们希望迭代本地搜索比任何一种方法在此问题上的执行效果都更好。

**随机爬山算法**

迭代本地搜索算法的核心是本地搜索，在本教程中，我们将为此目的使用随机爬山算法。随机爬山算法涉及到首先生成一个随机的起点和当前的工作解决方案，然后生成当前工作解决方案的扰动版本，如果它们优于当前的工作解决方案，则接受它们。假设我们正在研究连续优化问题，则解决方案是目标函数要评估的值的向量，在这种情况下，该向量是二维空间中以 - 5 和 5 为边界的点。我们可以通过以均匀的概率分布对搜索空间进行采样来生成随机点。例如：

```
# generate a random point in the search space
solution = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
```

我们可以使用高斯概率分布，当前解决方案中当前值的平均值以及由超参数控制的标准偏差来生成当前正在工作的解决方案的扰动版本，该超参数控制允许搜索从当前工作解决方案进行多远的探索。

我们将此超参数称为 “step_size”，例如：

```
# generate a perturbed version of a current working solution
candidate = solution + randn(len(bounds)) * step_size
```

重要的是，我们必须检查生成的解决方案是否在搜索空间内。

这可以通过一个名为`in_bounds（）`的自定义函数来实现，该函数采用候选解和搜索空间的边界，如果该点位于搜索空间中，则返回 True，否则返回 False。

```
# check if a point is within the bounds of the search
def in_bounds(point, bounds):
 # enumerate all dimensions of the point
 for d in range(len(bounds)):
  # check if out of bounds for this dimension
  if point[d] < bounds[d, 0] or point[d] > bounds[d, 1]:
   return False
 return True
```

然后可以在爬坡期间调用此函数，以确认新点在搜索空间的边界内，如果没有，则可以生成新点。

结合在一起，下面的函数`hillclimbing（）`实现了随机爬山局部搜索算法。它以目标函数的名称，问题的范围，迭代次数和步长为参数，并返回最佳解决方案及其评估。

```
# hill climbing local search algorithm
def hillclimbing(objective, bounds, n_iterations, step_size):
 # generate an initial point
 solution = None
 while solution is None or not in_bounds(solution, bounds):
  solution = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
 # evaluate the initial point
 solution_eval = objective(solution)
 # run the hill climb
 for i in range(n_iterations):
  # take a step
  candidate = None
  while candidate is None or not in_bounds(candidate, bounds):
   candidate = solution + randn(len(bounds)) * step_size
  # evaluate candidate point
  candidte_eval = objective(candidate)
  # check if we should keep the new point
  if candidte_eval <= solution_eval:
   # store the new point
   solution, solution_eval = candidate, candidte_eval
   # report progress
   print('>%d f(%s) = %.5f' % (i, solution, solution_eval))
 return [solution, solution_eval]
```

我们可以在 Ackley 函数上测试该算法。

我们将为伪随机数生成器固定种子，以确保每次运行代码时都得到相同的结果。

该算法将运行 1,000 次迭代，步长为 0.05 个单位。经过一些反复试验后，才选择了这两个超参数。

运行结束时，我们将报告找到的最佳解决方案。

```
# seed the pseudorandom number generator
seed(1)
# define range for input
bounds = asarray([[-5.0, 5.0], [-5.0, 5.0]])
# define the total iterations
n_iterations = 1000
# define the maximum step size
step_size = 0.05
# perform the hill climbing search
best, score = hillclimbing(objective, bounds, n_iterations, step_size)
print('Done!')
print('f(%s) = %f' % (best, score))
```

结合在一起，下面列出了将随机爬山算法应用于 Ackley 目标函数的完整示例。

```
# hill climbing search of the ackley objective function
from numpy import asarray
from numpy import exp
from numpy import sqrt
from numpy import cos
from numpy import e
from numpy import pi
from numpy.random import randn
from numpy.random import rand
from numpy.random import seed
 
# objective function
def objective(v):
 x, y = v
 return -20.0 * exp(-0.2 * sqrt(0.5 * (x**2 + y**2))) - exp(0.5 * (cos(2 * pi * x) + cos(2 * pi * y))) + e + 20
 
# check if a point is within the bounds of the search
def in_bounds(point, bounds):
 # enumerate all dimensions of the point
 for d in range(len(bounds)):
  # check if out of bounds for this dimension
  if point[d] < bounds[d, 0] or point[d] > bounds[d, 1]:
   return False
 return True
 
# hill climbing local search algorithm
def hillclimbing(objective, bounds, n_iterations, step_size):
 # generate an initial point
 solution = None
 while solution is None or not in_bounds(solution, bounds):
  solution = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
 # evaluate the initial point
 solution_eval = objective(solution)
 # run the hill climb
 for i in range(n_iterations):
  # take a step
  candidate = None
  while candidate is None or not in_bounds(candidate, bounds):
   candidate = solution + randn(len(bounds)) * step_size
  # evaluate candidate point
  candidte_eval = objective(candidate)
  # check if we should keep the new point
  if candidte_eval <= solution_eval:
   # store the new point
   solution, solution_eval = candidate, candidte_eval
   # report progress
   print('>%d f(%s) = %.5f' % (i, solution, solution_eval))
 return [solution, solution_eval]
 
# seed the pseudorandom number generator
seed(1)
# define range for input
bounds = asarray([[-5.0, 5.0], [-5.0, 5.0]])
# define the total iterations
n_iterations = 1000
# define the maximum step size
step_size = 0.05
# perform the hill climbing search
best, score = hillclimbing(objective, bounds, n_iterations, step_size)
print('Done!')
print('f(%s) = %f' % (best, score))
```

运行示例将对目标函数执行随机爬山搜索。搜索过程中发现的每个改进都会报告出来，然后在搜索结束时报告最佳解决方案。

注意：由于算法或评估程序的随机性，或者数值精度的差异，您的结果可能会有所不同。考虑运行该示例几次并比较平均结果。

在这种情况下，我们可以看到搜索过程中约有 13 处改进，最终解决方案约为 f（-0.981，1.965），得出的评估值为 5.381，与 f（0.0，0.0）= 0 相去甚远。

```
>0 f([-0.85618854 2.1495965 ]) = 6.46986
>1 f([-0.81291816 2.03451957]) = 6.07149
>5 f([-0.82903902 2.01531685]) = 5.93526
>7 f([-0.83766043 1.97142393]) = 5.82047
>9 f([-0.89269139 2.02866012]) = 5.68283
>12 f([-0.8988359 1.98187164]) = 5.55899
>13 f([-0.9122303 2.00838942]) = 5.55566
>14 f([-0.94681334 1.98855174]) = 5.43024
>15 f([-0.98117198 1.94629146]) = 5.39010
>23 f([-0.97516403 1.97715161]) = 5.38735
>39 f([-0.98628044 1.96711371]) = 5.38241
>362 f([-0.9808789 1.96858459]) = 5.38233
>629 f([-0.98102417 1.96555308]) = 5.38194
Done!
f([-0.98102417 1.96555308]) = 5.381939
```

**随机重新开始的随机爬山**

具有随机重启功能的随机爬山算法涉及重复运行随机爬山算法并跟踪找到的最佳解决方案。首先，让我们修改 hillclimbing（）函数以获取搜索的起点，而不是随机生成它。这将在以后实现迭代本地搜索算法时有所帮助。

```
# hill climbing local search algorithm
def hillclimbing(objective, bounds, n_iterations, step_size, start_pt):
 # store the initial point
 solution = start_pt
 # evaluate the initial point
 solution_eval = objective(solution)
 # run the hill climb
 for i in range(n_iterations):
  # take a step
  candidate = None
  while candidate is None or not in_bounds(candidate, bounds):
   candidate = solution + randn(len(bounds)) * step_size
  # evaluate candidate point
  candidte_eval = objective(candidate)
  # check if we should keep the new point
  if candidte_eval <= solution_eval:
   # store the new point
   solution, solution_eval = candidate, candidte_eval
 return [solution, solution_eval]
```

接下来，我们可以通过重复调用`hillclimbing（）`函数一定的次数来实现随机重启算法。每次通话时，我们都会为爬山搜索生成一个随机选择的新起点。

```
# generate a random initial point for the search
start_pt = None
while start_pt is None or not in_bounds(start_pt, bounds):
 start_pt = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
# perform a stochastic hill climbing search
solution, solution_eval = hillclimbing(objective, bounds, n_iter, step_size, start_pt)
```

然后，我们可以检查结果并将其保留，以使其比我们到目前为止所看到的任何搜索结果都要好。

```
# check for new best
if solution_eval < best_eval:
 best, best_eval = solution, solution_eval
print('Restart %d, best: f(%s) = %.5f' % (n, best, best_eval))
```

结合在一起，`random_restarts（）`函数实现了具有随机重启功能的随机爬山算法。

```
# hill climbing with random restarts algorithm
def random_restarts(objective, bounds, n_iter, step_size, n_restarts):
 best, best_eval = None, 1e+10
 # enumerate restarts
 for n in range(n_restarts):
  # generate a random initial point for the search
  start_pt = None
  while start_pt is None or not in_bounds(start_pt, bounds):
   start_pt = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
  # perform a stochastic hill climbing search
  solution, solution_eval = hillclimbing(objective, bounds, n_iter, step_size, start_pt)
  # check for new best
  if solution_eval < best_eval:
   best, best_eval = solution, solution_eval
   print('Restart %d, best: f(%s) = %.5f' % (n, best, best_eval))
 return [best, best_eval]
```

然后，我们可以将此算法应用于 Ackley 目标函数。在这种情况下，我们会将随机重启的数量限制为任意选择的 30 次。

下面列出了完整的示例。

```
# hill climbing search with random restarts of the ackley objective function
from numpy import asarray
from numpy import exp
from numpy import sqrt
from numpy import cos
from numpy import e
from numpy import pi
from numpy.random import randn
from numpy.random import rand
from numpy.random import seed
 
# objective function
def objective(v):
 x, y = v
 return -20.0 * exp(-0.2 * sqrt(0.5 * (x**2 + y**2))) - exp(0.5 * (cos(2 * pi * x) + cos(2 * pi * y))) + e + 20
 
# check if a point is within the bounds of the search
def in_bounds(point, bounds):
 # enumerate all dimensions of the point
 for d in range(len(bounds)):
  # check if out of bounds for this dimension
  if point[d] < bounds[d, 0] or point[d] > bounds[d, 1]:
   return False
 return True
 
# hill climbing local search algorithm
def hillclimbing(objective, bounds, n_iterations, step_size, start_pt):
 # store the initial point
 solution = start_pt
 # evaluate the initial point
 solution_eval = objective(solution)
 # run the hill climb
 for i in range(n_iterations):
  # take a step
  candidate = None
  while candidate is None or not in_bounds(candidate, bounds):
   candidate = solution + randn(len(bounds)) * step_size
  # evaluate candidate point
  candidte_eval = objective(candidate)
  # check if we should keep the new point
  if candidte_eval <= solution_eval:
   # store the new point
   solution, solution_eval = candidate, candidte_eval
 return [solution, solution_eval]
 
# hill climbing with random restarts algorithm
def random_restarts(objective, bounds, n_iter, step_size, n_restarts):
 best, best_eval = None, 1e+10
 # enumerate restarts
 for n in range(n_restarts):
  # generate a random initial point for the search
  start_pt = None
  while start_pt is None or not in_bounds(start_pt, bounds):
   start_pt = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
  # perform a stochastic hill climbing search
  solution, solution_eval = hillclimbing(objective, bounds, n_iter, step_size, start_pt)
  # check for new best
  if solution_eval < best_eval:
   best, best_eval = solution, solution_eval
   print('Restart %d, best: f(%s) = %.5f' % (n, best, best_eval))
 return [best, best_eval]
 
# seed the pseudorandom number generator
seed(1)
# define range for input
bounds = asarray([[-5.0, 5.0], [-5.0, 5.0]])
# define the total iterations
n_iter = 1000
# define the maximum step size
step_size = 0.05
# total number of random restarts
n_restarts = 30
# perform the hill climbing search
best, score = random_restarts(objective, bounds, n_iter, step_size, n_restarts)
print('Done!')
print('f(%s) = %f' % (best, score))
```

运行该示例将执行随机爬山，并随机重启以查找 Ackley 目标函数。每次发现改进的整体解决方案时，都会进行报告，并汇总通过搜索找到的最终最佳解决方案。

注意：由于算法或评估程序的随机性，或者数值精度的差异，您的结果可能会有所不同。考虑运行该示例几次并比较平均结果。

在这种情况下，我们可以看到搜索过程中的三处改进，发现的最佳解决方案约为 f（0.002，0.002），其评估值为大约 0.009，这比单次爬山算法要好得多。

```
Restart 0, best: f([-0.98102417 1.96555308]) = 5.38194
Restart 2, best: f([1.96522236 0.98120013]) = 5.38191
Restart 4, best: f([0.00223194 0.00258853]) = 0.00998
Done!
f([0.00223194 0.00258853]) = 0.009978
```

接下来，让我们看看如何实现迭代的本地搜索算法。

**迭代局部搜索算法**

迭代本地搜索算法是具有随机重启算法的随机爬坡的改进版本。重要的区别在于，随机爬山算法的每种应用的起点都是到目前为止找到的最佳点的一种扰动版本。我们可以通过使用 random_restarts（）函数作为起点来实现此算法。每次重新启动迭代时，我们可以生成到目前为止找到的最佳解决方案的修改版本，而不是随机的起点。这可以通过使用步长超参数来实现，就像在随机爬山者中使用的一样。在这种情况下，考虑到搜索空间中较大的扰动，将使用较大的步长值。

```
# generate an initial point as a perturbed version of the last best
start_pt = None
while start_pt is None or not in_bounds(start_pt, bounds):
 start_pt = best + randn(len(bounds)) * p_size
```

结合在一起，下面定义了`iterated_local_search（）`函数。

```
# iterated local search algorithm
def iterated_local_search(objective, bounds, n_iter, step_size, n_restarts, p_size):
 # define starting point
 best = None
 while best is None or not in_bounds(best, bounds):
  best = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
 # evaluate current best point
 best_eval = objective(best)
 # enumerate restarts
 for n in range(n_restarts):
  # generate an initial point as a perturbed version of the last best
  start_pt = None
  while start_pt is None or not in_bounds(start_pt, bounds):
   start_pt = best + randn(len(bounds)) * p_size
  # perform a stochastic hill climbing search
  solution, solution_eval = hillclimbing(objective, bounds, n_iter, step_size, start_pt)
  # check for new best
  if solution_eval < best_eval:
   best, best_eval = solution, solution_eval
   print('Restart %d, best: f(%s) = %.5f' % (n, best, best_eval))
 return [best, best_eval]
```

然后，我们可以将该算法应用于 Ackley 目标函数。在这种情况下，我们将使用较大的步长值 1.0 进行随机重启，这是在经过反复试验后选择的。

下面列出了完整的示例。

```
# iterated local search of the ackley objective function
from numpy import asarray
from numpy import exp
from numpy import sqrt
from numpy import cos
from numpy import e
from numpy import pi
from numpy.random import randn
from numpy.random import rand
from numpy.random import seed
 
# objective function
def objective(v):
 x, y = v
 return -20.0 * exp(-0.2 * sqrt(0.5 * (x**2 + y**2))) - exp(0.5 * (cos(2 * pi * x) + cos(2 * pi * y))) + e + 20
 
# check if a point is within the bounds of the search
def in_bounds(point, bounds):
 # enumerate all dimensions of the point
 for d in range(len(bounds)):
  # check if out of bounds for this dimension
  if point[d] < bounds[d, 0] or point[d] > bounds[d, 1]:
   return False
 return True
 
# hill climbing local search algorithm
def hillclimbing(objective, bounds, n_iterations, step_size, start_pt):
 # store the initial point
 solution = start_pt
 # evaluate the initial point
 solution_eval = objective(solution)
 # run the hill climb
 for i in range(n_iterations):
  # take a step
  candidate = None
  while candidate is None or not in_bounds(candidate, bounds):
   candidate = solution + randn(len(bounds)) * step_size
  # evaluate candidate point
  candidte_eval = objective(candidate)
  # check if we should keep the new point
  if candidte_eval <= solution_eval:
   # store the new point
   solution, solution_eval = candidate, candidte_eval
 return [solution, solution_eval]
 
# iterated local search algorithm
def iterated_local_search(objective, bounds, n_iter, step_size, n_restarts, p_size):
 # define starting point
 best = None
 while best is None or not in_bounds(best, bounds):
  best = bounds[:, 0] + rand(len(bounds)) * (bounds[:, 1] - bounds[:, 0])
 # evaluate current best point
 best_eval = objective(best)
 # enumerate restarts
 for n in range(n_restarts):
  # generate an initial point as a perturbed version of the last best
  start_pt = None
  while start_pt is None or not in_bounds(start_pt, bounds):
   start_pt = best + randn(len(bounds)) * p_size
  # perform a stochastic hill climbing search
  solution, solution_eval = hillclimbing(objective, bounds, n_iter, step_size, start_pt)
  # check for new best
  if solution_eval < best_eval:
   best, best_eval = solution, solution_eval
   print('Restart %d, best: f(%s) = %.5f' % (n, best, best_eval))
 return [best, best_eval]
 
# seed the pseudorandom number generator
seed(1)
# define range for input
bounds = asarray([[-5.0, 5.0], [-5.0, 5.0]])
# define the total iterations
n_iter = 1000
# define the maximum step size
s_size = 0.05
# total number of random restarts
n_restarts = 30
# perturbation step size
p_size = 1.0
# perform the hill climbing search
best, score = iterated_local_search(objective, bounds, n_iter, s_size, n_restarts, p_size)
print('Done!')
print('f(%s) = %f' % (best, score))
```

运行该示例将对 Ackley 目标函数执行 “迭代本地搜索”。

每次发现改进的整体解决方案时，都会进行报告，并在运行结束时汇总通过搜索找到的最终最佳解决方案。

注意：由于算法或评估程序的随机性，或者数值精度的差异，您的结果可能会有所不同。考虑运行该示例几次并比较平均结果。

在这种情况下，我们可以在搜索过程中看到四个改进，发现的最佳解决方案是两个非常小的输入，它们接近于零，其估计值为 0.0003，这比单次爬山或爬山都要好。登山者重新启动。

```
Restart 0, best: f([-0.96775653 0.96853129]) = 3.57447
Restart 3, best: f([-4.50618519e-04 9.51020713e-01]) = 2.57996
Restart 5, best: f([ 0.00137423 -0.00047059]) = 0.00416
Restart 22, best: f([ 1.16431936e-04 -3.31358206e-06]) = 0.00033
Done!
f([ 1.16431936e-04 -3.31358206e-06]) = 0.000330
```

Blog: http://yishuihancheng.blog.csdn.net