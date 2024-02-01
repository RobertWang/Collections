> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653562524&idx=1&sn=d6ddb33cfaa675c686d69c124267847b&chksm=8139b704b64e3e12ac665da938d087c57e8d7089197aa1b9221a6d7f4945dd709eb86e893028&mpshare=1&scene=1&srcid=02017EMmMqxHX8PtknSWuErx&sharer_shareinfo=ad2756d17bd3569800db5f9a6be4fd86&sharer_shareinfo_first=ad2756d17bd3569800db5f9a6be4fd86#rd)

变更已然是一个老生常谈的话题了，相信大家都不陌生。在技术标准里面，ITIL 的每个版本里面，都有他的篇幅描述，在 ITIL 中对于变更管理的核心是通过合理的流程和步骤来确保变更的有效管理，最小化潜在风险，保障服务的稳定性和可靠性。更多的是在强调对于将要发起的变更流程的规划、控制、评估和记录。但是缺少对于变更执行过程的过程管控，这里涉及对于过程对象的定义、过程自身的定义和过程的干预等等。

在笔者与业界一些公司的沟通交流中也发现，目前大家对于变更的重视程度已然达到了顶峰。在实践过程中，基本上是围绕变更方案评审，变更发起前控制（重大活动 / 节假日等关键时间控制），变更事件记录这些开展变更管理工作。

这种做法应对传统的 IT 架构是足够的，但是在目前围绕云原生 + 微服务的这套复杂技术体系来看，存在着诸多问题，诸如：

*   评审围绕流程驱动，在评审环节依靠人的经验来确保方案的正确性，过于依赖经验并难以持续保证效果
    
*   由于资源的稀缺性，难以确保每个变更都能够纳入评审，仅能覆盖部门高优重保的大型变更计划
    
*   评审行为仅能约束变更的发起，无法保障变更执行过程按照要求切实有效执行，比如强制分区灰度要求，强制观察时间要求等
    
*   基于人操作的变更和平台操作的变更，围绕技术平台的基础设施和业务平台的配置变更，散落在各个地方，各自对于变更用了各自的流程定义和描述，难以收归一起管理和分析
    
*   单纯的变更记录，难以支撑当下复杂场景下由某个边缘变更、底层变更导致的级联故障定位
    
*   由于缺乏完整的变更管控技术框架，难以实现公司内部所有变更的平台托管，即无法知晓到底存在多少变更行为，哪些地方会发起变更，每次变更具体是变了什么东西，变更影响具体影响了什么对象和范围等
    

面对以上诸多局限，我们秉承技术问题还需要技术解法的原则，将变更作为一个独立的技术域，抽象和定义了变更的对象概念，规划和设计了变更管控技术框架，围绕这个框架打通从变更元信息定义、变更平台 / 场景接入、变更防控到变更分析的全流程链路，来实现变更的一体化和多层次管控。

下面我们从变更的定义出发，看整个变更管控体系是如何设计和落地的。

**什么是变更？**

**变更定义**

日常变更无处不在，那到底什么才是稳定性领域中的变更呢？通过对行业理论和实践总结的梳理，我们认为变更是指任何对服务运行状态造成影响的行为。这些影响可能源自研发、SRE、产品或运营的各种活动。从本质上来讲，稳定性问题通常是服务从一种稳定状态转变为不稳定状态的熵增过程。

**变更生命周期**

在上面对于变更的定义里面，我们明确了几个关键的点，服务，状态变化和过程。一个变更的生命周期，其实就是对于变更过程从变更发起、执行到结束的描述和管理。传统中，一个完整的变更流程包含变更计划、变更预审、变更评审、变更执行和变更校验等环节。在变更执行环节，往往会围绕变更对象的范围，拆分多个批次进行变更的灰度化，减少因异常变更熵增带来的影响面过大问题。

**变更平台 / 场景**

关于状态变化和变更过程的描述，我们采用了 “变更平台” 和“变更场景”这一概念。变更平台是指发起变更的平台，其负责提供具体变更的功能。变更平台会托管一系列变更场景，以数据库管控平台为例，它涵盖了新建数据库、扩容实例、SQL DML 变更等多个变更场景。而 “变更场景” 则详细描述了特定变更过程，包括基础信息（变更描述、变更级别、变更对象、归属对象等）和管控信息（变更内容、变更批次、变更防御项）。通过这些详细描述，我们可以清晰了解是谁在何时何地执行了何种操作，带来了哪些变化，类似于代码中的函数定义，明确入参、出参、操作对象和调用方等详细信息。一旦这些信息被清晰结构化，就可以围绕 CMDB 实现平台之间的变更关联，从而具备后续变更风险预测和智能检测分析的能力。

**为何要管控？**

开篇也介绍了变更对线上稳定性的影响，故障的大概率成因是由于变更导致的。在变更过程中，不论是引入新的代码，还是调整新的配置都是有可能导致服务异常和故障发生的。基于此，我们非常有必要对变更建立体系化的管控能力，以期通过管控，减少变更过程中引入的风险和隐患，防止变更风险的扩大。

**管控思路**

对于变更需要进行管控，在这点上行业的认知都是一致的。但是在如何进行管控上，大家采用的方案是不同的。这里我们将变更作为一个独立的管控对象，通过平台化、工程化的手段进行完整的管控。基于前面的概念抽象，我们将这些概念转化到具体的变更对象模型上， 通过定义标准的变更信息结构和交互协议，提供统一的变更信息归集、变更感知 & 订阅 & 检索、变更强流程执行、变更防御配置 & 检测 & 阻断和应急逃生等能力，确保公司内部涉及变更的平台能够标准化、轻量化的接入和纳管，实现公司层面基架 & 业务系统在变更领域的统一化管控，以期在灾难来临时能够快速识别变更风险，降低故障影响。

对于变更的管控，虽然行业都认同其重要性，但在具体的管控方法上存在差异。我们将变更视为一个独立的管控对象，通过平台化和工程化手段实施全面的管控。基于前述概念的抽象，我们将这些理念转化为具体的变更对象模型。通过定义标准的变更信息结构和交互协议，我们提供了统一的变更信息归集、变更感知、订阅、检索、强化流程执行、防御配置、检测、阻断以及应急逃生等能力。这确保了公司内部涉及变更的平台能够以标准化、轻量化的方式接入和纳管。公司层面的基架和业务系统在变更领域实现了统一的管控，旨在在灾难来临时能够迅速识别变更风险，减少故障的影响。

**管什么？**

在进行变更管理之前，各个平台对自己平台所承载的变更行为都各自的定义，审计等操作，并且更多是以脚本或者代码的方式存在。在管什么阶段，我们需要明确我们的管理对象和管理范畴，首先我们要管理变更的模型，然后去管理变更的过程，以此来实现变更元信息的统一，为后续变更实例数据的统一建模打下基础。

**变更定义模型**

针对不同角色对变更的理解以及不同业务部门在语义的定义上是一定存在差异的。在 B 站，应用变更、配置变更和基础设施变更分别由不同平台承载。由于这几种变更场景对变更信息的定义存在差异，因此实现不同类型变更统一管控的目标需要抽象出一个通用的变更信息模型。

这个通用的变更信息模型将会作为一个中介层，跨越各种平台和部门，以统一的方式描述和定义变更信息。这种模型可能包括变更的基本属性、关联对象、影响范围、变更级别、审核流程以及执行步骤等。通过这样的模型，不同平台和部门能够以统一的语义来描述变更，提高了跨部门和跨平台之间的理解和协作。

在该模型的支持下，可以建立通用的变更管理标准和规范，确保不同类型的变更都能够遵循相同的管控流程和标准。这样一来，即使不同类型的变更由不同的平台承载，也能够在变更信息的定义和管理上保持一致性，促进变更管控的统一性和协调性。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnuHjEbSerqEOq2oHW9t08K2xDRGISQbkqOkgccDicThALA2VK6Vf2v6Q/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

我们将变更信息模型分为两个部分, 包括基本信息和管控信息. 其中基本信息包含:

1.  **变更方式**：用以标识具体的变更平台和平台唯一标识
    
2.  **变更对象**：包括变更对象类型 (应用, 数据库, 服务器等) 和变更对象唯一标识
    
3.  **变更环境**：变更所生效的环境，比如 UAT、PRE 预发布、PROD 环境等
    
4.  **变更时间**：变更具体执行的时间
    
5.  **变更人员**：变更实际的操作人
    
6.  **变更内容描述**：对变更操作的详细描述，包括 release notes 和详情描述，对于传达变更目的和实施细节非常重要。
    

而管控信息则包含:

1.  **变更诉求**：即期望通过变更来达成的目的，包括日常迭代、BUG 修复、故障处理等;
    
2.  **变更场景**：指具体的变更动作，包括迭代发布、配置发布、服务器上下线等;
    
3.  **变更范围**：即变更在线上环境的生效范围，包括机器分批生效模式以及流量比例分批生效等;
    
4.  **变更影响**：指本次变更影响的服务和基础设施
    

综合而言，这样的变更信息模型为实现多类型变更的统一管控提供了坚实的基础。标准化的信息模型使得后续的变更感知、变更预检、变更防御以及变更审计等操作都能够基于统一的信息结构进行，有助于降低管理复杂性，提高协同效率。此外，对于其他技术风险领域的能力建设，这个模型也为故障定位、根因分析等方面提供了通用的数据结构和框架。

**变更流模型**

一个变更的生效往往是基于一定范围和流程的，因此我们需要围绕这个过程进行变更流模型的定义。通过流模型定义，我们可以抽象出两类基本的变更流模型，分别适用于逐步生效的变更和无法拆分的变更：

1. 逐步生效的变更流模型：

a. 按机器分批生效：将变更分为若干批次，在每个批次中逐步将变更应用到不同的机器上。这种方式允许在不同的批次上实施更多的风险管控，以便更精细地监控和评估变更对系统的影响。

b. 按流量比例分批生效：允许按照预定的流量比例逐步引导用户访问到经过变更的系统。这种方式同样可以在不同阶段上实施风险控制，适用于需要更细粒度控制用户流量的情况。

2. 无法拆分的变更流模型：即一次性生效模型，适用于某些无法拆分或拆分成本较高的变更场景，例如 DB 配置类型变更。在这种模型下，变更控制防御能力主要集中在配置变更前后。

这两类变更流模型提供了在不同变更场景下进行管控的灵活性，使得团队可以基于当下情况选择合适的管控等级进行改造接入。逐步生效的模型允许更细粒度地控制变更的推进生效过程，细粒度控制风险，降低风险影响面。一次性生效的模型适用于那些不能分阶段应用的变更，强调在整个变更生命周期中的风险控制。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnFZ49nDSLP7JgmlasqGNblOwlHbrtm7nd0x9ic8c9SKwqjPxCtr4CDOg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

上图就是我们的变更流模型。在我们定义的变更流模型里, 任意的变更都包含一个初始化节点, 一个结束节点以及 0 或多个变更批次节点. 同时基于这样的变更流模型, 我们可以在节点上设置管控规则, 包括变更初始化的准入检查, 批次的前置后置校验以及变更结束后的后置检查.

该变更流模型包含了初始化节点、结束节点以及可能存在的多个变更批次节点，这提供了对变更生命周期的明确定义。同时，在这些节点上设置管控规则和加入防御项可以帮助确保变更按照预期顺利进行，这些管控规则通常涵盖以下方面：

1.  变更初始化的准入检查：在初始化节点上，执行各种准入检查，以验证变更的合规性、准备情况以及对环境的影响。这可以包括权限检查、配置验证、资源可用性检查等。确保变更前的准备工作符合要求是成功变更的关键。
    
2.  批次的前置后置校验：对于每个变更批次节点，可以设置前置和后置校验规则，以确保批次的执行前后系统状态的一致性和稳定性。前置校验可能涉及先决条件的确认，而后置校验可能涉及变更完成后的状态验证。
    
3.  变更结束后的后置检查：在结束节点上执行后置检查，以验证变更是否成功完成，并确保系统处于期望的状态。这可能包括验证配置变更是否正确应用、系统功能是否正常运行以及服务是否可用等。
    

通过在节点上设置这些管控规则，您的变更流模型变得更加完善和可控。这些规则有助于确保变更过程中的合规性、稳定性和一致性，从而减少风险，提高变更的成功率。同时，也有助于在变更过程中追踪问题、定位故障以及进行事后审计。

**控什么？**

明确了管理的对象、范围和过程，下面就是要控什么了。在这一章节，我们将基于上述的变更定义模型和流模型介绍控制的范围和控制等级描述。

**管控等级**

针对变更流模型，我们结合变更的生效范围，采用差异化的管控策略。对于常规的运维操作，通常以实例为单位进行批次生效，包含多个变更批次节点。相反，一些配置类的变更，如数据库变更，通常是直接生效的，不涉及变更批次节点的划分。在我们的做法中，我们参考了开源项目 AlterShield[1] 对变更代际的定义，并引入了管控等级的概念以便于内部推广和理解。基于这一框架，我们将变更划分为五个等级，以更精准地管理变更的生命周期。

<table resolved="" width="677"><colgroup><col><col><col><col><col></colgroup><tbody><tr><td colspan="1" rowspan="1" width="51" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px; background-color: rgb(255, 254, 213);"><p><strong>管控等级</strong></p></td><td colspan="1" rowspan="1" width="115" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px; background-color: rgb(255, 254, 213);"><p><strong>含义</strong></p></td><td colspan="1" rowspan="1" width="121" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px; background-color: rgb(255, 254, 213);"><p><strong>变更节点数量</strong></p></td><td colspan="1" rowspan="1" width="92" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px; background-color: rgb(255, 254, 213);"><p><strong>生命周期定义</strong></p></td><td colspan="1" rowspan="1" width="190" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px; background-color: rgb(255, 254, 213);"><p><strong>使用场景</strong></p></td></tr><tr><td colspan="1" rowspan="1" width="16" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>GO</p></td><td colspan="1" rowspan="1" width="115" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>仅做事件同步，无管控能力</p></td><td colspan="1" rowspan="1" width="79" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;" class=""><p class="">一个</p></td><td colspan="1" rowspan="1" width="92" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>仅有一个事件同步节点</p></td><td colspan="1" rowspan="1" width="210" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>适用于无任何风险的变更，但数据需要提供给相关人员做检索、审计等场景</p></td></tr><tr><td colspan="1" rowspan="1" width="16" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>G1</p></td><td colspan="1" rowspan="1" width="115" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>仅做单节点的变更前后切面管控</p></td><td colspan="1" rowspan="1" width="79" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;" class=""><p class="">一个</p></td><td colspan="1" rowspan="1" width="92" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>生命周期中仅有一个变更节点</p></td><td colspan="1" rowspan="1" width="210" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>适用于低风险变更，无需复杂风险管控能力，仅做事前准入性和事后完整性检查的场景</p></td></tr><tr><td colspan="1" rowspan="1" width="16" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>G2</p></td><td colspan="1" rowspan="1" width="115" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>有完整的变更工单，且工单下关联了至少 1 个批次的子节点</p></td><td colspan="1" rowspan="1" width="79" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;" class=""><p>多个</p></td><td colspan="1" rowspan="1" width="92" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>生命周期分为四个阶段 (工单开始&gt; 各批次节点开始&gt;各批次节点结束&gt;工单结束)</p></td><td colspan="1" rowspan="1" width="210" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>适用于多数逐步生效的变更，并需要配套进行风险管控的场景</p></td></tr><tr><td colspan="1" rowspan="1" width="16" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>G3</p></td><td colspan="1" rowspan="1" width="115" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>在完整的变更工单感知基础上，增加了变更提单阶段的感知</p></td><td colspan="1" rowspan="1" width="79" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;" class=""><p>多个</p></td><td colspan="1" rowspan="1" width="92" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>生命周期在 G2 等级的基础上，前置增加了变更提单阶段</p></td><td colspan="1" rowspan="1" width="210" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>适用于非技术人员或专人代理执行变更，需要系统辅助进行事前风险分析的场景</p></td></tr><tr><td colspan="1" rowspan="1" width="16" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>G4</p></td><td colspan="1" rowspan="1" width="115" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>在变更提单感知的基础上，增加了变更无人值守的决策能力</p></td><td colspan="1" rowspan="1" width="79" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p class="">多个</p></td><td colspan="1" rowspan="1" width="92" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>生命周期在 G3 等级的基础上，变更提单后增加了无人值守决策阶段</p></td><td colspan="1" rowspan="1" width="210" align="center" valign="middle" data-style="padding-top: 7px; padding-bottom: 7px; outline: 0px; word-break: break-all; hyphens: auto; border-color: rgb(193, 199, 208); vertical-align: top; min-width: 8px;"><p>适用于需要系统进行无人值守代理执行变更的场景</p></td></tr></tbody></table>

每个管控等级的变更场景在满足前一级别的基础之上衍生出更多的变更要求和管控切面。基于标准的变更信息模型以及变更等级，我们就可以将变更管控单独抽离出来, 在不同的平台业务上构建统一的变更管控平台, 使得 SRE 团队能够更专注于变更的风险防控与治理, 同时平台业务团队更专注于平台研发本身。

在每个管控等级中，随着等级的提升，变更场景会在满足前一级别基础上呈现更多的变更需求和管控层面。借助标准的变更信息模型和各个等级的定义，我们能够将变更管控独立出来，构建一个名为 ChangePilot 的统一变更管控平台（后文简称，ChangePilot），适用于不同平台业务。

这样的平台可以实现以下益处：

*   标准化的变更管控：基于变更信息模型和不同等级的定义，该平台能够提供标准化的变更管控流程和规则，确保各级变更的执行符合标准和最佳实践。
    
*   专注于核心任务：SRE 团队可以将精力集中在风险管理和治理方面，通过该平台更有效地监控和管理变更的影响。
    
*   业务团队专注开发：平台业务团队则能更专注于平台的研发工作，不必过多关注变更管控流程，从而提高开发效率和创新能力。
    
*   跨平台适用性：ChangePilot 平台能够适用于不同的平台业务，提供通用的变更管控解决方案，促进公司范围内的标准化和协作。
    
*   合理投入：针对不同的场景定义场景最合适的接入管控等级，避免不必要的接入和改造投入。
    

通过构建这样的变更管控平台，公司能够更有效地管理不同等级变更的流程，并在风险控制方面更有针对性，同时允许不同团队专注于其核心职能，提高整体的生产力和效率。

**平台总览**

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMn31eJJG05M1w9nRCiaYUsgekpYiccPEsP30maEB0ZoEYtrm09mNBvz4Tw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/1BMf5Ir754THapCFaWOjeKLbQNVQJDMncV8p78VaCZLIsek5bqN0o0ZicZNRAQgd6JJn8O4X2LkiaxY6rVOHoWrg/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

在技术层面，我们主要将一个变更场景整体做了如下定义：

```
type ChangeScene struct {
    // 归属变更平台
    Platform
    // 关联变更资源类型
    SourceType
    // 基础信息(变更内容)Schema
    ContentSchema
    // 管控信息(批次内容)Schema
    StepSchema
    // 检查项
    []Navigation
    ...
}

```

可以看到变更场景定义了一类变更，包括归属的变更平台，操作的变更资源，管控时所需的基础信息 Schema 和管控信息 Schema 以及检查项等。每一个变更都基于具体的变更场景进行实例化，执行变更场景定义的管控动作和信息交互。因此, 进行变更管控的第一件事就是定义需要管控的变更场景，变更源平台在接入自身平台后需要在 ChangePilot 上托管自身的变更场景后才能进行实际的变更管控。

**变更行为控制**

在 ChangePilot 基于前述的变更流模型提供了以下接口：

变更源平台需要在变更流中的每一个节点和 ChangePilot 交互

1.  在用户创建变更单之后, 执行 `InitChange` 初始化变更, ChangePilot 按照场景的定义执行准入检查;
    
2.  在批次开始时, 执行 `StartChangeStep` 开始变更批次, ChangePilot 执行批次前置校验;
    
3.  在批次结束后, 执行 `EndChangeStep` 结束变更批次, ChangePilot 执行批次后置校验;
    
4.  在变更结束后, 执行 `FinishChange` 结束变更, 更新变更最终状态, ChangePilot 执行后置检查; 
    

只有通过预先定义在变更场景之上的检查后才可以进行下一步的变更流程。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnfoVuNDczo22sPibqnugeAzzVe8wsaEo8ssdhreOKTjmNw5qY2sXbWVQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

 **变更防控建设**

在定义完变更场景后，变更源平台就可以基于变更场景来控制具体的变更行为。通过在变更场景中增加防御检查项来实现，在变更前的预检、变更中的批次检查和变更后校验等行为。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnudyNuYKgrj4oibkBKCoaeUV9HX3BGrRL3ewjgDE4QHW8iaoQTS1wsJmw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

在变更流模型中, 我们把每个节点的风险管控统一抽象为检查项, 检查项的基本定义如下:

```
type Navigation interface {
    ParamProvider() func(context.Context, Info) (Param, error)
    SpecProvider() func(context.Context, Info) (Spec, error)
    Execute(context.Context, Param, Spec) (Result, error)
}

```

从定义中可以看到, 检查项主要分为两个部分,  ParamProvider 负责从变更信息 (Info) 中生成检查参数(Param), SpecProvider 负责从变更信息中生成检查规范(Spec). 最后, 通过检查参数和检查规范得到检查结果(Result). 基于这样的抽象, 我们可以灵活的集成不同的检查项, 满足复杂的业务场景. 对于通用的变更检查项, 由 ChangePilot 集成为内置检查, 目前集成了大多数变更场景需求的核心检查项, 例如服务的 SLO 检查, 变更规范校验, 封网检查等。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnmKC5TnC9ibN7ictMaGAEpKyia1ibOlEhTl4XhyxeeFEVMcdoOsfMLvQiaSw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

**变更规范校验**

B 站在安全生产专项实践的过程中, 落地了一套强制灰度策略, 限制了变更需要在多个流量阶段强制灰度观察.  ChangePilot 集成了变更规范校验, 在变更批次前, 对将要执行的变更计划进行校验, 是否满足强制灰度策略, 满则无法继续当前变更批次.

**SLO 检查**

B 站对于应用业务核心场景有着一套较为完善的 SLO 体系, 在 SLO 不达标时, 触发高准确率和高召回率的 SLO 告警.  因此, ChangePilot 集成了 SLO 校验作为标准的检查项. 在应用的变更中, 如果某一个 SLO 指标下跌到阈值以下, 表明本次变更存在较大的风险, 需要进行干预.

**三方检查项**

除了内置的变更检查项外, ChangePilot 还提供了标准的三方检查项接入模式, 任何业务都能够自行接入满足自身业务需求的检查项, 并且作用于不同的变更场景. 三方检查项由需求平台实现 SpecProvider 并在 ChangePilot 定义结果解析表达式, 以标准的协议调用集成到 ChangePilot 中. 

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnCQ1GJUKTgHByiaxCIBDjcLruWA9eaLUQEk6hTVNcZu4Pmupvc1n7b6Q/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

 **变更内容感知**

基于标准的变更信息模型和变更场景的定义，我们能够以更高的精细度感知变更所涉及的内容。ChangePilot 本身提供了多维度的变更检索功能，其中包括基于变更内容的全文检索，允许用户通过关键词对变更信息进行精准检索。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnq9PWIeoB3OCdTIawrRlgLicPOiaqqEvVLcgOywurdVFticLzFczdLLUtQ/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

为了满足下游变更的精细化消费需求，ChangePilot 提供了灵活的变更订阅机制，支持使用 IM 消息、webhook 以及消息队列等多种方式，用户可以按照 CMDB 统一的元信息路径进行订阅。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnfS6HttXVAPJlItRDnEbBmtOXSVKibUWV6Z041DMwqlbP3HvcppzZ4Rg/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

同时，下游平台可以基于 ChangePilot 的变更事件进一步提升变更感知的细致程度。例如，可以利用变更事件进行服务画像的分析，从而更好地了解服务的变更影响。此外，基于变更搜索的功能，下游平台还可以关联应用告警，从而实现问题的快速定位。

**变更智能分析**

**变更关联分析**

通过对变更故障的深入分析，我们发现很大一部分异常并非源自本服务的变更，而是由上下游依赖或更基础的设施变更所引起的。为了解决这一挑战，ChangePilot 利用了 trace 和 CMDB 资源拓扑信息，以关联和聚合应用服务的变更。

具体而言，我们利用 trace 分析获取了应用服务的上下游关系，同时通过 CMDB 资源拓扑信息分析得到了应用服务所管理的资源架构，包括存储层（如数据库集群、缓存集群等）、计算层（包括容器、物理机实例）以及网关层等关键资源信息。随后，通过统一的资源标识符进行检索，我们能够有效地聚合变更信息。

这种方法使我们能够更全面地了解应用服务的生态系统，包括其上下游关系和底层资源架构。通过整合这些信息，ChangePilot 能够追踪并识别变更对整个服务生态系统的潜在影响。这不仅有助于准确定位变更引起的异常，还提供了更深层次的可视化和理解，有助于提高故障排查的效率和准确性。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMngzSw0vH8E09icEDDFMGfshnJ3LAicKpzU3zia9Gd4LBJ5nkcwzicYToSAw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/1BMf5Ir754THapCFaWOjeKLbQNVQJDMnEiaB6x8BGvHr4CO3R4iaO8AJkYiaer3sIraM4DP4MKyYvf1v0JDANumpw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

**业务接入实践**

**运维变更场景**

应用发布作为核心的变更场景之一，对于 B 站而言主要分为容器发布和物理机发布两种类型。ChangePilot 针对这两种发布类型进行了全面的管控。在应用发布过程中，我们按照 G2 的变更管控等级进行接入，并努力收集变更过程中的详细信息，包括当前变更的具体变化（如配置版本、镜像版本）以及当前批次发布的新实例等。然而，G2 变更管控等级的接入对于变更源平台原有的工作流程存在一定的侵入性，因为变更源平台必须在变更流程的各个节点分别与 ChangePilot 进行交互。目前的实践是通过接口调用的方式进行接入，但未来我们将提供统一的 SDK 来简化变更源平台的接入流程，从而帮助它们更快速、更方便地接入 ChangePilot。

这样的 SDK 将为变更源平台提供更简洁、更一致的接入方式，降低接入的复杂度和侵入性，使得变更源平台能够更高效地与 ChangePilot 进行集成。这将有助于提高变更管控的整体效率，并使得管控过程更加顺畅、无缝地融入到变更源平台的工作流程中。

**业务变更场景**

另一个重要的接入管控场景是业务网关配置的修改，包括变更接口列表和网关配置参数等。业务网关服务在提供统一鉴权、流量治理、多活单元化等多方面能力时扮演着关键的角色，因此对其配置的发布变更也是按照 G2 的变更管控等级进行接入。

在这一场景中，ChangePilot 通过与业务网关服务的接入，实现对配置修改的全面管控。这包括了对变更接口列表和网关配置参数的监测和记录，确保任何对这些关键配置的修改都能够得到妥善的管理和跟踪。类似于应用发布场景，业务网关配置的变更同样需要符合 G2 变更管控等级的要求，确保在整个变更过程中保持透明度和可追溯性。通过这样的管控机制，我们能够有效地防范潜在的问题，确保业务网关服务的配置修改是可控且安全的。

**实践思考**

**风险与效率**

变更管控的核心矛盾在于风险和效率, 因为变更管控的核心是通过损失部分效率来降低风险, 达成 "慢就是快". 然而实践的时候却无法按照最严格要求进行建设, 一刀切式的管控变更. 一方面, 变更管控会在一定程度上导致效率的损失; 另一方面, 在某些应急的场景下, 变更管控可能会降低恢复的速度.

**应急逃生豁免**

基于此，我们单独开辟了逃生能力通道，确保应用在故障场景可以快速响应，绕行非必要防御策略。在逃生策略方面，我们采用了紧急和常态化两种策略:

*   **绿色通道**：主要用于快速止损，仅豁免 1 个小时，即开即生效。开启绿色通道后，以私信的方式发送给操作人、应用研发、应用负责人，同时支持订阅方式将消息发送到工作群, 确保变更的感知和事后审计。
    
*   **白名单**：主要用于长时间豁免管控规则，一般用于活动期间的保障场景下一旦出现问题需要快速止损的情况，需要事前提交申请，并审批通过。
    

**总结与展望**

B 站的变更管控目前仍处于发展完善阶段, 目前已经覆盖核心变更平台的大部分场景. 未来一方面要继续提高变更管控的覆盖, 另一方面要引入更精确, 更智能的检查项, 例如, 通过时间序列异常检测技术检测对比变更前和变更后的指标时序, 发现潜在异常; 通过相似度算法检测变更中, 变更后日志的异常情况等。同时, 目前 LLM 技术如火如荼, 如何把 LLM 技术与变更管控结合, 做更智能化的变更感知和分析也是后续发展方向。

B 站的变更管控目前正处于不断发展和完善的阶段，当下已经成功覆盖了核心变更平台的核心变更场景。未来的发展方向将集中在两个主要方面：

*   进一步提高变更管控的覆盖范围：在提高覆盖范围方面，我们将持续扩展变更管控的适用场景，确保更全面地覆盖各个关键领域。这包括对更多变更平台的支持以及对不同变更场景的深入了解，以满足业务的多样化需求。
    
*   引入更为精确和智能的防御检查项：为了进一步提升变更管控的智能程度，我们计划引入更为精确的防御检查项。其中，我们将采用时间序列异常检测技术，通过对比变更前后的指标时序，发现潜在的异常情况。另外，我们还将使用相似度算法，对变更中和变更后的日志进行异常检测，以更准确地捕捉可能存在的问题。
    
*   新方向新技术的探索：同样值得注意的是，当前 LLM 技术蓬勃发展，我们计划将 LLM 技术与变更管控结合，以实现更智能化的变更感知和分析。这意味着我们将充分利用 LLM 技术提供的实时性和敏感性，更早地发现和解决变更可能引起的潜在问题，从而提高整体的系统稳定性和性能。
    

通过这些未来的发展方向，我们期望 B 站的变更管控系统能够更全面、更智能地适应复杂的业务环境，实现对变更的更为准确和及时的感知与防控。

**参考**

[1] AlterShield, Change-Related Failures Terminator（_https://altershield.io/zh-CN/_）

**参考阅读：**

*   [RocksDB 在 vivo 消息推送系统中的实践](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653562440&idx=1&sn=43d470f342727918d458f0a04e11d88a&chksm=8139b7d0b64e3ec6b9d005b38741f6902cb27753ab24d7d02985e7093826aebee64c20161afa&scene=21#wechat_redirect)  
    
*   [从变更 license 协议，到联合创始人离职，是时候考虑 Consul 的国产化平替方案了](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653562450&idx=1&sn=e3713e29168a3d92dd63cdd2a3668efa&chksm=8139b7cab64e3edc5c6031ffba89ba813cd2d56d710bc4d55996d6d99d8155deb8059a03dafc&scene=21#wechat_redirect)
    
*   [一文浅谈 CodeReview 中的一些思考](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653562453&idx=1&sn=99bc2c617123a9987b8c88917ab7d334&chksm=8139b7cdb64e3edbd28d5fc4c8a07c31f84f0bf51b55d12e29c2e4cdd717163004c92d4e125e&scene=21#wechat_redirect)
    
*   [放弃 PHP 转投 Go，10 万行代码重构升级一步到位！](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653562481&idx=1&sn=3e43d347d82221019273e24d4adf7ae4&chksm=8139b7e9b64e3eff903471f9e627913947dbaa6f8c5ef4ca54b27ee40088990deace9576bc71&scene=21#wechat_redirect)