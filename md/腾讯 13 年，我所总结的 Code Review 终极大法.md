> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/HoFSNCd1U3eoUqYaQiEgwQ)

> 一文梳理 Code Review 方法与实践

# 关注并星标腾讯云开发者

# 每周 1 | 鹅厂工程师带你审判技术

# 第 3 期 | 林强：Code Review 我都 CR 些什么

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95GDtt9cMfb2cZnk583WdLtJiczZoEN3IGA3TFfIhXPNkzSn5ibzCCLWriayKNdvhMoga4hzKibqNZtBQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95GDtt9cMfb2cZnk583WdLtNzIs4DWZkHmfGiaCupDgVFhE7buBxnbo6E1GfKJLUicmPERNevr2kYiag/640?wx_fmt=png)

谚语曰: “Talk Is Cheap, Show Me The Code”。知易行难，知行合一难。嘴里要讲出来总是轻松，把别人讲过的话记住，组织一下语言，再讲出来，很容易。设计理念你可能道听途说了一些，以为自己掌握了，但是你会做么？有能力去思考、改进自己当前的实践方式和实践中的代码细节么？不客气地说，很多人仅仅是知道并且认同了某个设计理念，进而产生了一种虚假的安心感——自己的技术并不差。但是他根本没有去实践这些设计理念，甚至根本实践不了这些设计理念，从结果来说，他懂不懂这些道理 / 理念，有什么差别？变成了自欺欺人。  

代码，是设计理念落地的地方，是技术的呈现和根本。同学们可以在 review 过程中做到落地沟通，不再是空对空的讨论，可以在实际问题中产生思考的碰撞，互相学习，大家都掌握团队里积累出来最好的实践方式！当然，如果 leader 没有时间写代码，仅仅是通过 review 代码指出团队内其他同学某些实践方式不好，需要给出建议的话，leader 本身也需要沉淀很多相关思考。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95GDtt9cMfb2cZnk583WdLtF39xYibUPiaA6XtNz56hJm2o4mG8LAt8YfiaXqGTuPjGdoicuNMtXMN8pw/640?wx_fmt=png)

我这里先给一个我自己的总结：所谓架构师，就是掌握大量设计理念和原则、落地到各种语言及附带工具链（生态）下的实践方法、垂直行业模型理解，定制系统模型设计和工程实践规范细则，进而控制 30 万 + 行代码项目的开发便利性、可维护性、可测试性、运营质量的资深研发群体。

厉害的技术人，主要可以分为下面几个方向：  

**奇技淫巧：**掌握很多技巧，以及发现技巧一系列思路的人。很多编程大赛比的就是这个，但是这个方向对软件工程用处似乎并不是很大。

**领域奠基：**比如约翰 · 卡马克，他创造出了现代计算机图形高效渲染的方法论。先不提假设没有他，之后会不会有人发明该方法，从结果上看他就是第一个发明者。1999 年，卡马克登上了美国时代杂志评选出来的科技领域 50 大影响力人物榜单，并且名列第 10 位。但是类似的殿堂级位置，凤毛麟角，不够大家分，也没我们的事儿。

**理论研究：**八十年代李开复博士坚持采用隐含马尔可夫模型的框架，成功地开发了世界上第一个大词汇量连续语音识别系统 Sphinx。我辈工程师，好像擅长这个的很少。

**产品成功：**小龙哥是标杆。

**最佳实践：**按照上面架构师的定义，这个是大家都可以做到的。在这条路上走得好，就能为任何公司构建技术团队，组织建设高质量的系统。

从上面的讨论中可以看出，我们普通工程师的进化之路，就是不断打磨最佳实践方法论、落地细节。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe94fohv4px9Jbfx6bWBXL0N5vMx1DEEHe7cArCO06MqkAULE6AI7NE5dvyCbJXg4WEsg0gHPfeBS6w/640?wx_fmt=png)

在讨论什么代码是好代码之前，我们先讨论什么是不好的。计算机是人造的学科，我们自己制造了很多问题，进而去思考解法。  

####    重复的代码

```
func BatchGetQQTinyWithAdmin(ctx context.Context, adminUin uint64, friendUin []uint64) (
  adminTiny uint64, sig []byte, frdTiny map[uint64]uint64, err error) {
  var friendAccountList []*basedef.AccountInfo
  for _, v := range friendUin {
    friendAccountList = append(friendAccountList, &basedef.AccountInfo{
      AccountType: proto.String(def.StrQQU),
      Userid:      proto.String(fmt.Sprint(v)),
    })
  }
  req := &cmd0xb91.ReqBody{
    Appid:       proto.Uint32(model.DocAppID),
    CheckMethod: proto.String(CheckQQ),
    AdminAccount: &basedef.AccountInfo{
      AccountType: proto.String(def.StrQQU),
      Userid:      proto.String(fmt.Sprint(adminUin)),
    },
    FriendAccountList: friendAccountList,
  }

```

因为最开始协议设计得不好，第一个使用接口的人，没有类似上面这个函数的代码，自己实现了一个嵌入逻辑代码的填写请求结构结构体的代码，在最开始效果还可以。但当有第二、第三个人干了类似的事情，我们将无法重构这个协议，只能麻烦地向前兼容。并且每个同学，都要理解一遍上面这个协议怎么填，理解有问题，就触发 bug。或者如果某个错误的理解普遍存在，我们就得找到所有这些重复的片段，去修改一遍。

当你要读一个数据，发现两个地方都有，不知道该选哪个。当你要实现一个功能，发现两个 rpc 接口、两个函数都能做到，不知道该选哪个。你有面临过这样的 “人生难题” 么？其实怎么选已经不重要了，你写的代码已经在走向 shit 的道路上迈出了坚实的一步。

####    早期有效的决策不再有效

很多时候，我们第一版代码写出来，是没有太大的问题的。比如下面这个代码：

```
func (s *FilePrivilegeStore) Update(key def.PrivilegeKey,
  clear, isMerge bool, subtract []*access.AccessInfo, increment []*access.AccessInfo,
  policy *uint32, adv *access.AdvPolicy, shareKey string, importQQGroupID uint64) error {
  info, err := s.Get(key)
  if err != nil {
    return err
  }
  incOnlyModify := update(info, &key, clear, subtract, 
    increment, policy, adv, shareKey, importQQGroupID)
  stat := statAndUpdateAccessInfo(info)
  if !incOnlyModify {
    if stat.groupNumber > model.FilePrivilegeGroupMax {
      return errors.Errorf(errors.PrivilegeGroupLimit, 
        "group num %d larger than limit %d",
        stat.groupNumber, model.FilePrivilegeGroupMax)
    }
  }
  if !isMerge {
    if key.DomainID == uint64(access.SPECIAL_FOLDER_DOMAIN_ID) &&
      len(info.AccessInfos) > model.FilePrivilegeMaxFolderNum {
      return errors.Errorf(errors.PrivilegeFolderLimit, 
        "folder owner num %d larger than limit %d",
        len(info.AccessInfos), model.FilePrivilegeMaxFolderNum)
    }
    if len(info.AccessInfos) > model.FilePrivilegeMaxNum {
      return errors.Errorf(errors.PrivilegeUserLimit, 
        "file owner num %d larger than limit %d",
        len(info.AccessInfos), model.FilePrivilegeMaxNum)
    }
  }
  pbDataSt := infoToData(info, &key)
  var updateBuf []byte
  if updateBuf, err = proto.Marshal(pbDataSt); err != nil {
    return errors.Wrapf(err, errors.MarshalPBError,
      "FilePrivilegeStore.Update Marshal data error, key[%v]", key)
  }
  if err = s.setCKV(generateKey(&key), updateBuf); err != nil {
    return errors.Wrapf(err, errors.Code(err),
      "FilePrivilegeStore.Update setCKV error, key[%v]", key)
  }
  return nil
}

```

现在看这个代码挺好的，长度没超过 80 行，逻辑比较清晰。但是当 isMerge 这里判断逻辑，如果加入更多的逻辑，把局部行数撑到 50 行以上，这个函数味道就坏了。由此出现两个问题：

函数内代码不在一个逻辑层次上，阅读代码，本来在阅读着顶层逻辑，突然就掉入了长达 50 行的 isMerge 的逻辑处理细节，还没看完，读者已经忘了前面的代码讲了什么，需要来回看，挑战自己大脑的 cache 尺寸。

代码有问题后，再新加代码的同学，是改还是不改前人写好的代码呢？要不要往里面填屎，堆屎山呢？出 bug 谁来背？这是一个灵魂拷问。

####    对合理性没有苛求

“两种写法都 ok，你随便挑一种吧”，“我这样也没什么吧”，这是我经常听到的话。

```
func (i *IPGetter) Get(cardName string) string {
  i.l.RLock()
  ip, found := i.m[cardName]
  i.l.RUnlock()
  if found {
    return ip
  }
  i.l.Lock()
  var err error
  ip, err = getNetIP(cardName)
  if err == nil {
    i.m[cardName] = ip
  }
  i.l.Unlock()
  return ip
}

```

i.l.Unlock() 可以放在当前的位置，也可以放在 i.l.Lock() 下面，做成 defer。两种在最初构造的时候好像都行，这个时候很多同学态度就变得不坚决。实际上，这里必须是 defer 的。

```
  i.l.Lock()
  defer i.l.Unlock()
  var err error
  ip, err = getNetIP(cardName)
  if err != nil {
    return "127.0.0.1"
  }
  i.m[cardName] = ip
  return ip

```

这样的修改，是极有可能发生的，它还是要变成 defer，那为什么不一开始就是 defer，进入最合理的状态？不一开始就进入最合理的状态，在后续协作中，其他同学很可能犯错！

####    总是面向对象 / 总喜欢封装

我是软件工程科班出身，学的第一门编程语言是 C++，教材是《C++ 程序设计: 程序设计和面向对象设计入门 (第 3 版)》 。当时自己读完教材，初入程序设计之门，对于里面讲的 “封装”，惊为天人，多么美妙的设计啊；面向对象，多么智慧的设计啊。但是这些年来，我看到了大牛 “云风” 对于 “毕业生使用 mysql api 就喜欢搞个 class 封装再用” 的嘲讽；看到了各种莫名其妙的 class 定义；体会到了经常要去看一个莫名其妙的继承树，必须要把整个继承树整体读明白才能确认一个细小的逻辑分支；多次体会到了需要辛苦地压抑住抵触情绪，去细读一个自作聪明的被封装的代码，确认我的 bug。除了 UI 类场景，我认为应该少用继承、多用组合。

```
template<class _PKG_TYPE>
class CSuperAction : public CSuperActionBase {
  public:
    typedef _PKG_TYPE pkg_type;
    typedef CSuperAction<pkg_type> this_type;
    ...
}

```

这是 sspp 的代码。CSuperAction 和 CSuperActionBase，一会儿 super，一会儿又 base，Super 和 SuperBase 是在怎样的两个抽象层次上，不通读代码，没人能读明白。我想确认任何细节，都要把多个层次的代码都通读了，有什么封装性可言？

好，你说是作者没有把 class name 取好。那问题是，你能取好么？一个刚入职的新人同学能把 class name、class 树设计得好么？即使是对简单的业务模型，也需要无数次 “坏” 的对象抽象实践，才能培养出一个具有合格的 class 抽象能力的同学，这对于大型却松散的团队协作，难道不是破坏性的？已经有了一套继承树，想要添加功能就只能在这个继承树里添加，以前的继承树不再适合新的需求，这个继承树上所有的 class，以及使用它们的地方，你都去改？不，是个正常人都会放弃，然后开始堆屎山。

封装，就是我可以不关心实现，但是做一个稳定的系统，每一层设计都可能出问题。abi，总有合适的用法和不合适的用法，真的存在我们能完全不关心封装的部分是怎么实现的？不，你不能。bug 和性能问题，常常就出现在，你选择了错误的用法去使用一个封装好的函数。即使是  Android、iOS 的 api，Golang、Java 现成的 api，我们常常都要去探究实现，才能把 api 用好。那我们是不是该一上来，就做一个透明性很强的函数才更为合理？使用者想知道细节，进来吧，我的实现很易读，你看看就明白，使用时不会迷路！对于逻辑复杂的函数，我们还要强调函数内部工作方式 “可以让读者在大脑里想象呈现完整过程” 的可显性，让使用者轻松读懂，有把握，使用时，不迷路！

####    根本没有设计

这个最可怕，所有需求，上手就是一顿撸。“设计是什么东西？我一个文件 5w 行，一个函数 5k 行，干不完需求？” 从第一行代码开始，就是无设计的，随意地踩着满地的泥坑。对于旁人的眼光没有感觉，一个人独舞，产出的代码完成了需求，毁灭了接手自己代码的人本该用来休息的夜晚~ 这个就不举例了，每个同学应该都能在自己的项目里发现这种代码。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe94fohv4px9Jbfx6bWBXL0N5FzLo0jCP05Zp8J4icxpdExT3iaCkFKvH6up4qtE8DlxTGzd0uKJv9PgQ/640?wx_fmt=png)

同学们常常听演讲、公开课，就喜欢听一些细枝末节的 “干货”。这没有问题。但是你干了几年活，学习了多少干货知识点？构建起自己的技术思考 “面”，进入立体的 “工程思维”，把技术细节和系统要满足的需求在思考上连接起来了么？当听到一个需求的时候，你能思考到自己的 code package 该怎么组织，函数该怎么组织了么？

技术点要怎么和需求连接起来呢？答案很简单，你需要在时间里总结，总结出一些明确的原则、思维过程。思考怎么去总结，特别像是在思考哲学问题。从一些琐碎的细节中，由具体情况上升到一些原则、公理。同时，大家在接受原则时，不应该是接受和记住原则本身，而应该是解构原则，让这个原则在自己这里重新推理一遍，自己完全掌握这个原则的适用范围。

再进一步具体地说，对于工程最佳实践的形而上的思考过程，就是：

把工程实践中遇到的问题，从**问题类型**和**解法类型**两个角度去归类，总结出一些有限适用的原则，就从点到了面。把诸多总结出的原则，组合应用到自己的项目代码中，就是把多个面结合起来构建了一套立体的最佳实践的方案。当你这套方案能适应 30w + 行代码的项目，超过 30 人的项目，你就入门架构师了！当你这个项目是多端、多语言，代码量超过 300w 行，参与人数超过 300 人，代码质量依然很高，代码依然在高效地自我迭代，每天消除掉过时的代码，填充高质量的替换旧代码和新生的代码。恭喜你，你已经是一个很高级的架构师了！再进一步，你对某个业务模型有独到或者全面的理解，构建了一套行业第一的解决方案，结合刚才高质量实现的能力，实现了这么一个项目，恭喜你，你已经成为专家工程师了！

那么，我们要从头开始积累思考和总结吗？不，有一本书叫做《 Unix 编程艺术》，我在不同的时期分别读了 3 遍，文中的很多原则，正好就能作为 code review 时大家判定代码质量的准绳。但在那之前，我得讲一下另外一个很重要的话题，模型设计。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe94fohv4px9Jbfx6bWBXL0N5ddHeLEDEKrK2ib3llqXhsd75vnAKwCd8XRc6CCUrNIQB7PMEGQuaThg/640?wx_fmt=png)

没读过 oauth2.0 RFC，就去设计第三方授权登陆的人，终归还要再发明一个蹩脚的 oauth。

2012 年我刚毕业，和一个去了广州联通公司的华南理工毕业生聊天。当时他说他工作很不开心，因为工作里不经常写代码，而且认为自己有 ACM 竞赛金牌级的算法熟练度 + 对 CPP 代码的熟悉，写下一个个指针操作内存，还有什么程序写不出来，什么事情做不好。当时我觉得，挺有道理，编程工具在手，我什么事情做不了？

现在我会告诉他，复杂如 Linux 操作系统、Chromium 引擎、Windows Office，你做不了。原因是他根本没进入软件工程的工程世界，不是会搬砖就能修出港珠澳大桥。但是这么回答并不好，举证用的论据离我们太遥远了。我现在会回答，你做不了，简单如一个权限系统，你知道怎么做么？堆积一堆逻辑层次一维展开的 if else？简单如一个共享文件管理，你知道怎么做么？堆积一堆逻辑层次一维展开的 if else？你公司有上万台服务器，你要怎么写一个管理平台？堆积一堆逻辑层次一维展开的 if else？

上来就是干，能实现上面提到的三个看似简单的需求么？想一想，亚马逊、腾讯云们折腾了多少年，最后才找到了容器 + Kubernetes 的大杀器。这里需要谷歌多少年在 BORG 系统上的实践，提出了优秀的服务编排领域模型。权限领域，有 RBAC、DAC、MAC 等等模型，到了业务，又会有细节的不同。如 Domain Driven Design 说的，没有良好的领域思考和模型抽象，逻辑复杂度就是 n^2 指数级的，你得写多少 if else，得思考多少可能的 if 路径，来 cover 所有的不合符预期的情况。你必须要有 Domain 思考探索、model 拆解 / 抽象 / 构建的能力。有人问过我，要怎么有效地获得这个能力？这个问题我没能回答，就像是在问我，怎么才能获得 MIT 博士的学术能力？我无法回答。唯一回答就是，进入某个领域，就是首先去看前人的思考，站在前人的肩膀上，再用上自己的通识能力，去进一步思考。至于怎么建立好的通识思考能力，可能得去常青藤读个书吧 :），或者就在工程实践中思考和锻炼自己的这个能力！

同时，基于 model 设计的代码，能更好地适应产品经理不断变更的需求。比如说，一个 calendar（日历）应用，随便想想，不要太简单！以 “userid_date” 为 key 记录一个用户的每日安排不就完成了么？只往前走一步，设计了一个任务，上限分发给 100w 个人，创建这么一个任务，是往 100w 个人下面添加一条记录？你得改掉之前的设计，换 db。再往前走一步，要拉出某个用户和某个人一起要参与的所有事务，是把两个人的所有任务来做 join？好像还行。如果是和 100 个人一起参与的所有任务呢？100 个人的任务来 join？不现实了吧。好，你引入一个群组 id，那么，你最开始的 “userid_date” 为 key 的设计，是不是又要修改和做数据迁移了？经常来一个需求，你就得把系统推翻重来，或者根本就只能拒绝用户的需求，这样的战斗力，还好意思叫自己工程师？你一开始就应该思考自己面对的业务领域，思考自己的日历应用可能的模型边界，把可能要做的能力都拿进来思考，构建一个 model，设计一套通用的 store 层接口，基于通用接口的逻辑代码。当产品不断发展，就是不停往模型里填内容，而不是推翻重来。思考模型边界，构建模型细节，就是两个很重要的能力，也是绝大多数产品经理不具备的能力。你面对产品经理时，就听取他们出于对用户体验负责思考出的需求点，到你自己这里，用一个完整的模型去涵盖这些零碎的点。

model 设计，是形而上思考中的一个方面，也是一个特别重要的方面。在这里，强烈推荐大家去精读几遍《Unix 编程艺术》这本经典著作，书中提到的 KISS 原则、组合原则、吝啬原则、透明性原则、通俗原则、缄默原则、 补救原则等，都是随着软件工程不断发展，历久弥新的经典原则，值得反复揣摩深思。在自己的 coding/code review 中，站在巨人的肩膀上去思考。不重复地发现经典力学，而是往相对论挺进。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95GDtt9cMfb2cZnk583WdLtMzTtQ9bHdQiaFK4tiaibZuf7nEW1l6ia2wp8iaQqwBicPmRGicXluRO25Ykwg/640?wx_fmt=png)

前文列举了很多大的原则和思考方向，这里再来给大家简单列举几个细节执行点。下面主要针对 Golang 语言，另外我一时也想不全我所执行的所有细则，这就是我强调 “原则” 的重要性，原则是可枚举的。

▶︎ 对于代码格式规范，100% 严格执行，眼中容不得一点沙。

▶︎ 文件绝不能超过 800 行，超过一定要思考怎么拆文件。工程思维，就在于拆文件的时候积累。

▶︎ 函数对决不能超过 80 行，超过一定要思考怎么拆函数，思考函数分组，层次。工程思维，就在于拆文件的时候积累。

▶︎ 代码嵌套层次不能超过 4 层，超过了就得改。多想想能不能 early return。工程思维，就在于拆文件的时候积累。

```
if !needContinue {
  doA()
  return
} else {
  doB()
  return
}

```

```
if !needContinue {
  doA()
  return
}
doB()
return

```

下面这个就是 early return，把两端代码从逻辑上解耦了。

▶︎ 从目录、package、文件、struct、function 一层层下来 ，信息一定不能出现冗余。比如 file.FileProperty 这种定义。只有每个 “定语” 只出现在一个位置，才为 “做好逻辑、定义分组 / 分层” 提供了可能性。

▶︎ 多用多级目录来组织代码所承载的信息，即使某一些中间目录只有一个子目录。

▶︎ 随着代码的扩展，老的代码违反了一些设计原则，应该立即原地局部重构，维持住代码质量不滑坡。比如：拆文件、拆函数、用 Session 来保存一个复杂的流程型函数的所有信息、重新调整目录结构。

▶︎ 基于上一点考虑，我们应该尽量让项目的代码有一定的组织、层次关系。我个人的当前实践是除了特别通用的代码，都放在一个 git 里。特别通用、修改少的代码，逐渐独立出 git，作为子 git 连接到当前项目 git，让 Golang 的 Refactor 特性、各种 Refactor 工具能帮助我们快速、安全局部重构。

▶︎ 自己的项目代码，应该有一个内生的层级和逻辑关系。flat 平铺展开是非常不利于代码复用的。怎么复用、怎么组织复用，肯定会变成 “人生难题”。

▶︎ 如果被 review 的代码虽然简短，但是你看了一眼却发现不咋懂，那就一定有问题。自己看不出来，就找高级别的同学交流。这是你与他人共同 review 代码、共同成长的宝贵时刻。

▶︎ 日志要少打，要打日志就要把关键索引信息带上，必要的日志必须打。

▶︎ 有疑问就立即问，不要怕问错，让代码作者给出解释，不要怕问出低级问题。

▶︎ 不要说 “建议”，提问题就直接提，有错误就得改！

▶︎ 请积极使用 trpc。总是要和老板站在一起！只有和老板达成的对于代码质量建设的共识，才能在团队里更好地做好代码质量建设。

▶︎ 消灭重复！消灭重复！消灭重复！

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95GDtt9cMfb2cZnk583WdLtUepXvSRfib2ZczHibmvY7q0aiah7fadzDEbvchO9iaeaFrzGibM97j1Ky4Q/640?wx_fmt=png)

最后我来为 “主干开发” 多说一句话。道理很简单，只有每次被 review 代码不到 500 行，reviewer 才能快速地看完，而且几乎不会看漏。超过 500 行，reviewer 就不能仔细看，只能大概浏览了。而且让你调整 500 行代码内的逻辑比调整 3000 行甚至更多的代码，容易很多，降低不仅仅是 6 倍，而是一到两个数量级。有问题在刚出现的时候就调整了，不会给被 review 的人带来大的修改负担。

关于 CI(continuous integration)，还有很多好的资料和书籍，大家应该及时去学习学习。

![](https://mmbiz.qpic.cn/mmbiz_png/VY8SELNGe95GDtt9cMfb2cZnk583WdLtugzO7xt3CctJVSMdnbOTiae2splY0XEpx965HVTIIlRiauTl5eUAPvPQ/640?wx_fmt=png)

建议大家把《Unix 编程艺术》这本书找出来读一读。你们已经积累了大量的代码实践，亟需对 “工程性” 做思考总结。很多工程方法论都过时了，这本书的内容是例外中的例外，它所表达出的内容没有因为软件技术的更替而过时。

佛教禅宗讲 “不立文字”（不立文字，教外别传，直指人心，见性成佛），很多道理和感悟是不能用文字传达的。大家常常因为 “自己听说过、知道某个道理” 而产生一种安心感，认为 “我懂了这个道理”，但是自己却不能在实践中做到。知易行难，知道却做不到，在工程实践里，就和 “不懂这个道理” 没有任何区别。

曾经我面试过一个别的公司的总监，讲得好像一套一套，代码拉出来遛一遛，根本就没做到，仅仅会道听途说。他在工程实践上的探索前路可以说已经基本断绝了。我只能祝君能做好向上管理，走自己的纯管理道路吧。请不要再说自己对技术有追求，是个技术人了！

所以大家不仅仅是看看我这篇文章，而是在实践中去不断践行和积累自己的 “教外别传” 吧。

《Software Engineering at Google》也是一本必读好书，在这里推荐给大家。

-End-

原创作者｜林强

📢你是怎么做 Code Review 的？你对 Code Review 有什么看法欢迎留言。我们将挑选一则最有创意的答案，为其留言者送出腾讯定制便捷通勤袋。9 月 4 日中午 12 点开奖。