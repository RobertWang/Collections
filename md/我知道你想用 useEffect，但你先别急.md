> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247537056&idx=1&sn=6458059e158e779f3558ad64ef4b51fd&chksm=e92a6cafde5de5b95ac9f71666c422ca6be5a69a2e7ae4abc011ae16c5a7cef5cb24769f5d6f&mpshare=1&scene=1&srcid=02027osIzRIW9pKrmx80caCp&sharer_shareinfo=2d737c9290564719f9b04063cfa1c98c&sharer_shareinfo_first=2d737c9290564719f9b04063cfa1c98c#rd)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKoTzV6Pb6MRQcWglvtKDZvFJYBLLDquB36QGuST9YGcbUFHW9OV1J43qHt860sUwK2iazokm6c7bw/640?wx_fmt=jpeg&from=appmsg)  

阿里妹导读

useEffect 是 React 提供给我们的一个 “逃生舱”，是 React 的纯函数式世界通往命令式世界的 “逃生通道”，选择合适的时机使用 useEffect 会让我们的代码既优雅又高效，反之会造成不必要的负担。

> 本文 95% 来自官方文档，均为官方推荐的写法或者做法。

你是不是也遇到过这个情况？

你希望状态 name 变化后，发起一次请求，于是你用了 useEffect，这简直不要太适合：

```
useEffect(()=>{
  query();
},[name])

```

随着迭代，需求变更，name 也会被其他地方给修改，但是他只是临时修改，并不需要发起一次请求。

幸运的是，你发现了这个问题，因为之前 name 相关的修改地方太多了，所以不敢动，于是你用 if 处理：

```
useEffect(()=>{
  if(isReFresh){
    query();
  }
},[name])

```

很好，没毛病，不会重复发起请求了。

后面需求又改了，现在 age 改变也要重新请求，很好，这不就是依赖数组的用处吗，加进去就好了：

```
useEffect(()=>{
  if(isReFresh){
    query();
  }
},[name,age])

```

随着时间推移，你逐渐发现：

*   「是否发送请求」与「if 条件」相关
    
*   「是否发送请求」还与「name、age 等依赖项」相关
    
*   「name、age 等依赖项」又与「很多需求」相关
    

根本分不清到底什么时候会发送请求，一直在堆屎...

Effect 的生命周期

https://react.dev/learn/lifecycle-of-reactive-effects

Effect 和其他的组件生命周期不太一样，组件的生命周期通常是：

*   当组件被添加到屏幕上时 --- mount
    
*   当组件接收到新的 props 或者 state 时（通常是为了响应式交互） --- update
    
*   当组件从屏幕上被移除时 --- unmount
    

这是设计组件生命周期的好办法，但是并不适用于 Effect。Effect 只能做两件事情：

*   开始**同步**一些内容
    
*   稍后停止**同步**
    

如果你的 Effect 取决于 props 和 state 的变化，那么这个循环可能会发生多次。

React 也提供了相应的 linter 规则来检测你是否正确的指定了 Effect 的依赖，以保证你的 Effect 可以正确的同步最新的 props 和 state。

为什么要少用 Effect？

**修改数据以更新渲染时：**当你更新状态的时候，React 会首先调用你的组件函数来计算屏幕上应该显示什么，然后 React 会将这些变化 “提交” 到 DOM 去渲染更新屏幕。接着 React 会执行你的 Effects，如果你的 Effects 也马上更新了 State，此时也将从头开始重新启动整个过程。

**用户事件：**当某一个数据变更时，在 Effect 运行时你可能无法准确的得知用户做了什么事情，只知道数据发生了变更。（依赖项变化 ->useEffect->if 判断 ==> 开头提到的死循环问题）

在 React 官方文档的 ESCAPE HATCHES 章节中，我们发现 effect 占据了一半的文档，这个章节翻译成中文是逃生舱口，只有在危险的时候才会进入逃生舱口，effect 亦是如此，只有在不得已的时候将其作为最后的逃生舱。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKHQTxia7KibL3bJpXic5vaW9TEwnUElI1qDiaDrlWNlFMtrssoBk0uD0YK5OZiagTVHNnBhvMUU0viawNg/640?wx_fmt=png&from=appmsg)

哪些情况可以不用 Effect？

> Don’t rush to add Effects to your components. Keep in mind that Effects are typically used to “step out” of your React code and synchronize with some external system. This includes browser APIs, third-party widgets, network, and so on. If your Effect only adjusts some state based on other state, you might not need an Effect.
> 
> 不要急于向组件添加 Effect。请记住，Effects 通常用于 “跳出”React 代码并与某些外部系统同步，你可以把 Effect 看作从 React 的纯函数式世界通往命令式世界的逃生通道。这包括浏览器 API、第三方小部件、网络等。如果您的效果仅根据其他状态调整某些状态，则您可能不需要 Effect。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKHQTxia7KibL3bJpXic5vaW9Ta1UUqcrhFW53uz9ZpxpbEVg98N8Rkz59FWtUUJYgKicsbldpXaMCsow/640?wx_fmt=png&from=appmsg)

从 Effect-Usage 的第二个小节开始，React 官方就在说 Effect 只是一个逃生舱，你可能不需要用它，和 Effect 相关的文章，我感觉官方每时每刻都在重复**强调少用，别用，换一个方法，就怕你滥用....**

所以，大家可以从以下几个 case 开始考虑，是不是不要用 useEffect 了，但是阅读之前请注意，不要根据标题来断章取义的认为标题的这种情况就完全不适合用 useEffect，可能不适合用 useEffect 的仅在内容描述的某一些 case 下，而其他的情况还需要再分析考虑。

**一、依赖 props 或者 state** 

```
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // 🔴 Avoid：冗余的状态和不必要的Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}

```

没必要这么写，既复杂又低效。这种情况就命中了我前面说的他会从头开始重新渲染整个页面。

```
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ Good: 重渲染期间计算
  const fullName = firstName + ' ' + lastName;
  // ...
}

```

**当你需要计算某些值，而恰好计算的参数都是可以从 Props 或者 State 中获取的，不要把他放在 State 中，相对应的，可以在渲染期间计算它。**这使得：

*   代码更快（避免了额外的 “级联” 更新）
    
*   更简单（删除了一些代码）
    
*   更不容易出错（避免了由于不同状态变量彼此不同步而导致的错误）
    

非 state 数据是否可以？

答：不可以

这个就涉及到 React 本身的更新渲染机制，也就是我们为什么要用 useState 来 set 数据。如果不用 useState，是不会触发 React 本身的更新的。

**二、缓存昂贵的计算** 

如何判断本次计算是否昂贵？

一般而言，除非您要创建或遍历数千个对象，否则它可能并不昂贵。

如果记录的总时间加起来很大（比如 1 毫秒或更多），那么记住该计算可能是有意义的。

可以利用如下的方式来测算：

```
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');

```

```
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // 🔴 Avoid: 冗余的状态和不必要的Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);
  // ...
}

```

与前面的示例一样，这既不必要又低效。

```
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ Good：当getFilteredTodos执行的不慢的时候，这个写法很好
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}

```

通常情况下，这种写法已经是很优秀的写法了！但是如果 getFilteredTodos 很慢，或者你有很多其他的待办事项会导致 props 或者 state 变化的非常的频繁。在这种情况下，如果你不想重新计算的话，可以将昂贵的计算包裹在 useMemo 的 Hook 中来缓存。

> 注意：useMemo 不会让第一次渲染更快。它只会帮助您跳过不必要的更新工作。

```
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ Good：除非 todos 或 filter 改变了，否则不会重新执行这个方法
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}

```

**三、当 props 改变时重置所有 state** 

这个 ProfilePage 组件接收一个 userId。该页面包含评论输入，您使用评论状态变量来保存其值。当您从一个配置文件导航到另一个配置文件时，评论状态不会重置。因此，很容易不小心在错误的用户个人资料上发表评论。所以你考虑用 useEffect 来清空 state：

```
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');
  // 🔴 Avoid: prop 改变时用useEffect来重置state
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}

```

其实大家都可以感受得到，这个写法非常的低效且一看就不是正确的解法。如果你有多个 state，每多一个 state 你都要加一次清空。而且 ProfilePage 及其子项将首先使用过时值呈现，然后再次呈现清空后的值。并且如果这个组件是嵌套的，你也需要一个一个清除嵌套组件的 state，显然不是最优解。

相反的，你可以用 key 来标识每一个组件，来告诉 React，每一个用户的个人资料在概念上是不同的：

```
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}
function Profile({ userId }) {
  // ✅ Good: comment包括其他的state在key改变时会自动的重置
  const [comment, setComment] = useState('');
  // ...
}

```

通常，React 在同一组件的同一个地方渲染时，React 会保留状态。

**四、当 props 改变时重置部分 state**

可能也有一些场景，比如说你只希望在 props 改变时，只修改部分的 state，你可能会想到用 useEffect 来清除部分，如下：

```
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);
  // 🔴 Avoid: prop 改变时用useEffect来重置state
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}

```

这里的问题和第三点说的一致，而且 setSelection(null) 调用将导致再次重新渲染 List 及其子组件，再次重新启动整个过程。

第一步还是删除 useEffect，考虑在渲染期间直接调整状态：

```
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);
  // 🟢 Better: 渲染改变时调整state
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}

```

带着一些疑问来看一下这个写法：

*   为什么要 items !== prevItems，不写会怎么样？
    
*   这种写法优势在哪里，好在哪里？
    
*   ...
    

Storing information from previous renders [2]

*   它必须在 prevCount !== count 之类的条件内，并且条件内必须有 setPrevCount(count) 之类的调用。否则，您的组件重复循环渲染，直到它崩溃。此外，你只能像这样更新当前渲染组件的状态。在渲染过程中调用另一个组件的 set 函数是错误的。最后，你的 set 调用应该仍然更新状态而不是整体替换改变 [3]。
    
*   当您在渲染期间调用 set 函数时，React 将在您的组件以返回语句退出后立即重新渲染该组件，然后再渲染子组件。这样，子组件就不需要渲染两次。组件函数的其余部分仍将执行（结果将被丢弃）。
    

即使这种写法比 useEffect 更好，但是大部分组件也不会这么写，不管你怎么做，基于属性或其他状态调整状态都会使你的数据流更难理解和调试。仍然不建议用这个写法。遇到这类问题，我们还是优先考虑将这个问题归到其余的两个解法中来：  

*   能否重置所有状态
    
*   在渲染中计算结果
    

```
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: 在渲染期间计算一切
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}

```

这种写法更好，因为它的结果是计算出来的，每次渲染都会重新计算一次结果。对大多数项目遇到这种情况来说，这个显然是更优解。

**五、在事件处理程序之间共享逻辑** 

假设你有一个带有两个按钮（购买和结帐）的产品页面，这两个按钮都可以让你购买该产品。你希望在用户将产品放入购物车时显示通知。在两个按钮的点击处理程序中调用 showNotification() 感觉是重复的，因此你可能想将此逻辑放在副作用中：

```
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: 将特定事件逻辑放在Effect中
  useEffect(() => {
    if (product.isInCart) {
      // 展示通知
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);
  function handleBuyClick() {
    addToCart(product);
  }
  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}

```

在这里用 useEffect 是没必要的，而且可能会导致一些 bug，例如，假设你的应用程序在页面重新加载之间 “记住” 了购物车。如果你将产品添加到购物车一次并刷新页面，通知将再次出现。每次你刷新该产品的页面时，它都会继续出现。这是因为 product.isInCart 在页面加载时已经是 true，所以上面的 effect 将调用 showNotification()。

**当您不确定某些代码是应该在 Effect 中还是在事件处理程序中时，问问自己为什么这段代码需要运行。只有当代码应该在组件被展示给用户再运行时，才应该使用** **useEffect****。**在这个例子中，通知应该出现是因为用户按下了按钮，而不是因为页面被显示了！正确的应该是将对应的时间逻辑放在特定事件被处理时才触发：

```
function ProductPage({ product, addToCart }) {
  // ✅ Good: 特定事件逻辑只有在事件被处理时才触发
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }
  function handleBuyClick() {
    buyProduct();
  }
  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}

```

**六、POST 请求**

useEffect 会发起两次请求

```
useEffect(() => {
  logVisit(url); // Sends a POST request
}, [url]);

```

在 dev 环境中，logVisit 将针对每个 url 调用两次，因此你可能想尝试修复它。 **我们建议保持此代码不变。** 与前面的示例一样，运行一次和运行两次之间没有用户可见的行为差异。从实际的角度来看，logVisit 不应在开发中执行任何操作，因为你不希望来自开发机器的日志影响生产指标。每次你保存其文件时，你的组件都会 remounts，因此无论如何它都会记录开发中的额外访问。  

在 production 环境中，不会访问两次。

假设你有一个 Form 组件，会发送 2 组请求。一组是在组件 mount 的时候发起，另一种是在点击提交按钮时发起：

我们可以分析一下这两个：

*   第一个请求是合理的，因为触发请求的原因就是表单已经显示渲染，需要获取一次（在 dev 会触发两次）
    
*   第二个不合理，第二个注册并不是由表单渲染而引起的请求，只是在某个特定条件下（比如：用户按下按钮发起请求），此时应该放在特定的事件请求方法中
    

**七、计算链** 

有时你可能会想链接副作用，每个副作用都根据其他状态调整一个状态：

如果用 useEffect 来处理的话，会有两个问题：

*   低效：组件（及其子级）必须在链中的每个 set 调用之间重新渲染。最坏的情况下渲染是这样的：**setCard→ 渲染 → setGoldCardCount → 渲染 → setRound → 渲染 → setIsGameOver → 渲染**
    
*   迭代能力和维护性差：假设后续有新需求，需要时空回溯，获取游戏的每个动作历史。你会通过将每个状态变量更新为过去的一个值来实现。然而，每次修改 card 就会触发一次 Effect 链并且更改你需要显示的数据，这种代码往往是僵硬且脆弱的。
    

在这种情况下，最好在渲染期间计算你能做什么，并在事件处理程序中调整状态：

这样效率更高。此外，如果打算做一个查看游戏历史的方法，现在能够将每个状态变量设置为过去的值，而不会触发调整每个其他值的 Effect 链。如果您需要在多个事件处理程序之间重用逻辑，您可以提取一个函数并从这些处理程序中调用它。

在事件处理内部，状态表现的会像快照。举个例子，在你调用 setRound(round + 1) 时，round 值会同步的更新给用户，但是如果你想要在方法里面拿到更新后的值，需要像这样 const nextRound = round + 1 手动定义它。

说到这里，你如果觉得如果涉及到计算链，就先摒弃 useEffect，那就错了。并不是所有的计算链都不适合在 useEffect 中。举个例子，如果说你有多个 dropdown，且下一个 dropdown 的内容依赖于上一个 dropdown 选择的内容，此时用 useEffect 来处理计算链是合适的，因为此时我们的数据是和后端交互的，从服务端中获取数据，在这种情况下，由于获取到的数据是异步的，需要等待请求结果，所以用 effect 反而可以保证数据的同步性。这样做的好处是，无论哪一个下拉框的选项发生了变化，都会触发相应的 Effect 钩子函数，保证了整个表单的数据同步性。

**八、初始化应用程序**

一些逻辑应该只在应用程序加载时运行一次

你可能想将它放在顶层组件的副作用中：

我想如果前面你有认真读这篇文章，你应该知道问题。前面提到了，在 dev 环境下，useEffect 会触发两次。这就可能会导致问题，就以这个代码为例，可能会导致身份令牌验证失效，因为可能这个方法就是用来调用一次的，再次请求可能就拿不到身份信息了。

虽然这个问题只在 dev 环境中会被触发，但是还是要尽量保证一致性，便于调试和重用。如果某些逻辑必须在每次应用加载时运行一次而不是每次组件安装时运行一次，请添加一个顶级变量来跟踪它是否已经执行：

您还可以在模块初始化期间和应用程序呈现之前运行它：

尽量慎用此模式，即使这个组件最终没被渲染，当你的组件被导入时就会执行一次。应尽量将应用程序范围的初始化逻辑保留到根组件模块（如 App.js）或应用程序的入口点。

**九、通知父级状态变更**

假设你正在编写一个具有内部 isOn 状态的 Toggle 组件，该状态可以是 true 或 false。有几种不同的方式来切换它（通过单击或拖动）。你希望在 Toggle 内部状态发生变化时通知父级，因此你公开了一个 onChange 事件并从副作用中调用它：

和之前一样，这个做法并不理想，Toggle 更新状态 ->React 更新屏幕 ->React 运行 Effect-> 调用 onChange-> 父组件更新状态 -> 重渲染（这里要启动另一个渲染通道）。

如果可以的话，最好一次性完成所有事情。

通过这种方法，Toggle 组件及其父组件都会在事件期间更新其状态。React 将来自不同组件的更新一起批量处理，因此只有一个渲染通道。

这个做法也叫 “状态提升”，将状态提升到父组件中，让父组件通过切换状态来完全控制 Toggle，这意味着父组件将必须包含更多逻辑，但总体上需要担心的状态会更少。每当您尝试保持两个不同的状态变量同步时，请尝试提升状态！

**十、将数据传递给父级** 

这个操作是可以实现数据传递，不存在写法问题。但是在 React 中，数据是从父组件传递到子组件的单向数据流动。当你发现错误时，可以沿着组件链追踪信息的来源，直到你发现某个组件有错误属性或者状态。当子组件在副作用中更新其父组件的状态时，数据流变得很难追踪。由于子级和父组件都需要相同的数据，因此让父级获取该数据，然后将其传递给子级才是正确的做法：

**十一、订阅外部存储**

有时，你的组件可能需要订阅 React 状态之外的一些数据。此数据可能来自第三方库或内置浏览器 API。由于此数据可能会在 React 不知情的情况下发生变化，因此你需要手动为你的组件订阅它。这通常通过 useEffect 完成，例如：

在这里，该组件订阅了一个外部数据存储（在本例中为浏览器 navigator.onLine API）。由于此 API 在服务器上不存在（因此它不能用于初始 HTML），因此最初将状态设置为 true。每当该数据存储的值在浏览器中发生变化时，组件都会更新其状态。

这么用其实也很正常，一般也不会挑什么毛病。但是，我们有更好的解决方案，React 中有一个专门的用户订阅外部存储的钩子，useSyncExternalStore：

与使用副作用手动将可变数据同步到 React 状态相比，这种方法更不容易出错。通常，你会像上面的 useOnlineStatus() 一样编写一个自定义钩子，这样你就不需要在各个组件中重复此代码。

**十二、获取数据**

前面说了这么多可以不用的 useEffect，我就当大家都认可了，但是大家心里肯定有一个是觉得一定要放在 useEffect 里面的，那就是**获取数据**

> 事先说明，并不是所有获取数据都不可以直接在 useEffect 中，请看下文分析，部分场景是不适合直接在 useEffect 中...

```
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    // 🔴 Avoid：没有清除逻辑的获取数据
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);
  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

```

肯定有人会质疑，这有啥问题？请求参数（query）或者页码（page）改变发起一次请求，不就该放在 useEffect 里面吗？

但是我现在要说的是，你**不需要**把这个逻辑放在**事件处理**函数中。

这可能看起来与之前需要将逻辑放入事件处理函数中的示例相矛盾！但是，请考虑到打字事件并不是获取数据的主要原因。搜索输入框的值经常从 URL 中预填充，用户可以在不关心输入框的情况下导航到后退和前进页面。

page 和 query 的来源其实并不重要。只要该组件可见，你就需要通过当前 page 和 query 的值，保持 results 和网络数据的同步。这就是为什么这里是一个 Effect 的原因。

然而，大家在写这类请求的时候一定会碰到一个问题，那就是用户快速输入，会频繁发起请求，但是无法保证返回的数据顺序。由于 setState 是最后调用的，将会错误地显示结果。这种情况被称为 “竞态条件”：两个不同的请求 “相互竞争”，并以与你预期不符的顺序返回。

你可能会考虑加上 debounce，或者加上清理函数来忽略较早的返回结果。这确保了当你在 Effect 中获取数据时，除了最后一次请求的所有返回结果都将被忽略。

处理竞态条件并不是实现数据获取的唯一难点。你可能还需要考虑：

*   缓存响应结果（使用户点击后退按钮时可以立即看到先前的屏幕内容）
    
*   如何在服务端获取数据（使服务端初始渲染的 HTML 中包含获取到的内容而不是加载动画）
    
*   以及如何避免网络瀑布（使子组件不必等待每个父组件的数据获取完毕后才开始获取数据）
    

如果你不使用框架（也不想开发自己的框架），但希望使从 Effect 中获取数据更符合人类直觉，请考虑像这个例子一样，将获取逻辑提取到一个自定义 Hook 中：

一般来说，当你不得不编写 Effect 时，请留意是否可以将某段功能提取到专门的内置 API 或一个更具声明性的自定义 Hook 中，比如上面的 useData。你会发现组件中的原始 useEffect 调用越少，维护应用将变得更加容易。

那我应该在什么时候用 Effect？

https://react.dev/reference/react/useEffect#usage

内容太多，大家可以作为后续自己学习的内容...

回到开始

当你希望 name 变化之后发起请求，你首先应该问自己：

**「状态 name 变化，接下来需要发起请求」**

还是

**「某个用户行为需要发起请求，请求依赖状态 name 作为参数」**

如果是后者，我们应当将这个请求和用户的操作关联，而不是放在 useEffect 当中。

```
<Button onClick={(e)=>{query(e)}}>点击发起请求</Button>
```

经过这样修改，**「状态 name 变化」**与**「发送请求」**之间不再有因果关系，后续对**状态 name** 的修改不会再有**「无意间触发请求」**的顾虑。

诸如此类的问题，大家期望使用 useEffect 的时候，都应该考虑一下，是否在前面提到的十二个 case 其中之一，是否有更好的写法。

总结

useEffect 是 React 提供给我们的一个 “逃生舱”，是 React 的纯函数式世界通往命令式世界的 “逃生通道”，选择合适的时机使用 useEffect 会让我们的代码既优雅又高效，反之会造成不必要的负担。

以下情况可能考虑不要使用 useEffect

*   如果你可以在渲染期间计算某些内容，则不需要使用 Effect。
    
*   想要缓存昂贵的计算，请使用 useMemo 而不是 useEffect。
    
*   想要重置整个组件树的 state，请传入不同的 key。
    
*   想要在 prop 变化时重置某些特定的 state，请在渲染期间处理。
    
*   组件 显示 时就需要执行的代码应该放在 Effect 中，否则应该放在事件处理函数中。
    
*   如果你需要更新多个组件的 state，最好在单个事件处理函数中处理。
    
*   当你尝试在不同组件中同步 state 变量时，请考虑状态提升。
    
*   你可以使用 Effect 获取数据，但你需要实现清除逻辑以避免竞态条件。
    

**参考：**
-------

[1] https://react.dev/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state

[2] https://react.dev/reference/react/useState#storing-information-from-previous-renders

[3] https://react.dev/reference/react/useState#updating-objects-and-arrays-in-state

[4] https://react.dev/learn/state-as-a-snapshot

[5] https://react.dev/learn/queueing-a-series-of-state-updates

[6] https://react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development

[7] https://react.nodejs.cn/reference/react/useSyncExternalStore