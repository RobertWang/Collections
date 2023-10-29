> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u014253011/article/details/81089421)

**P2P（Peer to Peer）对等网络**

P2P 技术属于覆盖层网络 (Overlay Network) 的范畴，是相对于客户机 / [服务器](http://www.cww.net.cn/tech/%B7%FE%CE%F1%C6%F7) (C/S) 模式来说的一种网络信息[交换](http://www.cww.net.cn/techClass1)方式。在 C/S 模式中，数据的分发采用专门的服务器，多个客户端都从此服务器获取数据。

优点是：数据的一致性容易控制，系统也容易管理。

缺点是：因为服务器的个数只有一个 (即便有多个也非常有限)，系统容易出现单一失效点；单一服务器面对众多的客户端，由于 CPU 能力、内存大小、网络带宽的限制，可同时服务的客户端非常有限，可扩展性差。

P2P 技术正是为了解决这些问题而提出来的一种对等网络结构。在 P2P 网络中，每个节点既可以从其他节点得到服务，也可以向其他节点提供服务。这样，庞大的终端资源被利用起来，一举解决了 C/S 模式中的两个弊端。  

P2P 应用软件主要包括文件分发软件、语音服务软件、流媒体软件。目前 P2P 应用种类多、形式多样，没有统一的[网络协议](https://so.csdn.net/so/search?q=%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE&spm=1001.2101.3001.7020)标准，其体系结构和组织形式也在不断发展。

**对等网络的基本结构**

**![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)  
**

**（1）集中式对等网络**（Napster、QQ）

集中式对等网络基于中央目录服务器，为网络中各节目提供目录查询服务，传输内容无需再经过中央服务器。这种网络，结构比较简单，中央服务器的负担大大降低。但由于仍存在中央节点，容易形成传输瓶颈，扩展性也比较差，不适合大型网络。但由于目录集中管理，对于小型网络的管理和控制上倒是一种可选择方案。

**（2）无结构分布式网络（**Gnutella**）**

无结构分布式网络与集中式的最显著区别在于，它没有中央服务器，所有结点通过与相邻节点间的通信，接入整个网络。在无结构的网络中，节点采用一种查询包的机制来搜索需要的资源。具体的方式为，某节点将包含查询内容的查询包发送到与之相邻的节点，该查询包以扩散的方式在网络中蔓延，由于这样的方式如果不加节制，会造成消息泛滥，因此一般会设置一个适当的生存时间（TTL），在查询的过程中递减，当 TTL 值为 0 时，将不再继续发送。

   这种无结构的方式，组织方式比较松散，节点的加入与离开比较自由，当查询热门内容时，很容易就能找到，但如果需求的内容比较冷门，较小的 TTL 不容易找到，而较大的 TTL 值又容易引起较大的查询流量，尤其当网络范围扩展到一定规模时，即使限制的 TTL 值较小，仍然会引起流量的剧增。但当网络中存在一些拥有丰富资源的所谓的类服务器节点时，可显著提高查询的效率。

**（3）结构化分布式网络（**第三代 P2P Pastry、Tapestry、Chord、CAN）

    结构化分布式网络，是近几年基于分布式哈希表（Distributed Hash Table）技术的研究成果。它的基本思想是将网络中所有的资源整理成一张巨大的表，表内包含资源的关键字和所存放结点的地址，然后将这张表分割后分别存储到网络中的每一结点中去。当用户在网络中搜索相应的资源时，它将能发现存储与关键词对应的哈希表内容所存放的结点，在该结点中存储了包含所需资源的结点地址，然后发起搜索的结点根据这些地址信息，与对应结点连接并传输资源。这是一种技术上比较先进的对等网络，它具有高度结构化，高可扩展性，结点的加入与离开比较自由。这种方式适合比较大型的网络。

对等网络经典结构

  (1)DHT 结构

    分布式哈希表 (DHT)[1] 是一种功能强大的工具，它的提出引起了学术界一股研究 DHT 的热潮。虽然 DHT 具有各种各样的实现方式，但是具有共同的特征，即都是一个环行拓扑结构，在这个结构里每个节点具有一个唯一的节点标识 (ID)，节点 ID 是一个 128 位的哈希值。每个节点都在路由表里保存了其他前驱、后继节点的 ID。如图 1(a) 所示。通过这些路由信息，可以方便地找到其他节点。这种结构多用于文件共享和作为底层结构用于流媒体[传输](http://www.cww.net.cn/techClass0) [2]。

    (2) 树形结构     P2P 网络树形结构如图 1(b) 所示。在这种结构中，所有的节点都被组织在一棵树中，树根只有子节点，树叶只有父节点，其他节点既有子节点也有父节点。信息的流向沿着树枝流动。最初的树形结构多用于 P2P 流媒体直播 [3-4]。     (3) 网状结构

    网状结构如图 1(c)所示，又叫无结构。顾名思义，这种结构中，所有的节点无规则地连在一起，没有稳定的关系，没有父子关系。网状结构 [5] 为 P2P 提供了最大的容忍性、动态适应性，在流媒体直播和点播应用中取得了极大的成功。当网络变得很大时，常常会引入超级节点的概念，超级节点可以和任何一种以上结构结合起来组成新的结构，如 KaZaA[6]。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)  
  
**P2P 技术应用**

(1) 分布式科学计算  
P2P 技术可以使得众多终端的 CPU 资源联合起来，服务于一个共同的计算。这种计算一般是计算量巨大、数据极多、耗时很长的科学计算。在每次计算过程中，任务 (包括逻辑与数据等) 被划分成多个片，被分配到参与科学计算的 P2P 节点机器上。在不影响原有计算机使用的前提下，人们利用分散的 CPU 资源完成计算任务，并将结果返回给一个或多个服务器，将众多结果进行整合，以得到最终结果。  
(2) 文件共享  
BitTorrent 是一种无结构的网络协议。除了 BitTorrent 之外，还有不少著名的无结构化的 P2P 文件共享协议，典型的有 Gnutella[8] 和 KaZaA[6]。  
(3) 流媒体直播  
(4) 流媒体点播  
(5)IP 层语音通信  
Skype 采取类似 KaZaA 的拓扑结构，在网络中选取一些超级节点。在通信双方直连效果不好时，一些合适的超级节点则担当起其中转节点的角色，为通信双方创建中转连接，并转发相应的语音通信包。  

**典型 P2P 应用的机制分析**  
      分析典型的 P2P 应用机制可以深入了解 P2P 的原理。本节将对文件分发、流媒体应用、语音服务 3 个领域中具有代表性的软件机制进行详细的分析。对于这些软件的分析有助于理解 P2P 技术的原理和把握 P2P 技术未来发展的趋势。

**BitTorrent  
**      BitTorrent 软件用户首先从 Web 服务器上获得下载文件的种子文件，种子文件中包含下载文件名及数据部分的哈希值，还包含一个或者多个的索引 (Tracker) 服务器地址。它的工作过程如下：客户端向索引服务器发一个超文本传输协议 (HTTP) 的 GET 请求，并把它自己的私有信息和下载文件的哈希值放在 GET 的参数中；索引服务器根据请求的哈希值查找内部的数据字典，随机地返回正在下载该文件的一组节点，客户端连接这些节点，下载需要的文件片段。因此可以将索引服务器的文件下载过程简单地分成两个部分：与索引服务器通信的 HTTP，与其他客户端通信并传输数据的协议，我们称为 BitTorrent 对等协议。BitTorrent 软件的工作原理如图 4 所示。BitTorrent 协议也处在不断变化中，可以通过数据报协议 (UDP) 和 DHT 的方法获得可用的传输节点信息，而不是仅仅通过原有的 HTTP，这种方法使得 BitTorrent 应用更加灵活，提高 BitTorrent 用户的下载体验。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**eMule  
**      eMule 软件基于 eDonkey 协议改进后的协议，同时兼容 eDonkey 协议。每个 eMule 客户端都预先设置好了一个服务器列表和一个本地共享文件列表，客户端通过 TCP 连接到 eMule 服务器进行登录，得到想要的文件的信息以及可用的客户端的信息。一个客户端可以从多个其他的 EMule 客户端下载同一个文件，并从不同的客户端取得不同的数据片段。eMule 同时扩展了 eDonkey 的能力，允许客户端之间互相交换关于服务器、其他客户端和文件的信息。eMule 服务器不保存任何文件，它只是文件位置信息的中心索引。eMule 客户端一启动就会自动使用传输控制协议 (TCP) 连接到 eMule 服务器上。服务器给客户端提供一个客户端标识(ID)，它仅在客户端服务器连接的生命周期内有效。连接建立后，客户端把其共享的文件列表发送给服务器。服务器将这个列表保存在内部数据库内。eMule 客户端也会发送请求下载列表。连接建立以后，eMule 服务器给客户端返回一个列表，包括哪些客户端可以提供请求文件的下载。然后，客户端再和它们主动建立连接下载文件。图 5 所示为 eMule 的工作原理。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)  

      eMule 基本原理与 BitTorrent 类似，客户端通过索引服务器获得文件下载信息。eMule 同时允许客户端之间传递服务器信息，BitTorrent 只能通过索引服务器或者 DHT 获得。eMule 共享的是整个文件目录，而 BitTorrent 只共享下载任务，这使得 BitTorrent 更适合分发热门文件，eMule 倾向于一般热门文件的下载。

**迅雷  
**      迅雷是一款新型的基于多资源多线程技术的下载软件，迅雷拥有比目前用户常用的下载软件快 7～10 倍的下载速度。迅雷的技术主要分成两个部分，一部分是对现有 Internet 下载资源的搜索和整合，将现有 Internet 上的下载资源进行校验，将相同校验值的统一资源定位 (URL) 信息进行聚合。当用户点击某个下载连接时，迅雷服务器按照一定的策略返回该 URL 信息所在聚合的子集，并将该用户的信息返回给迅雷服务器。另一部分是迅雷客户端通过多资源多线程下载所需要的文件，提高下载速率。迅雷高速稳定下载的根本原因在于同时整合多个稳定服务器的资源实现多资源多线程的数据传输。多资源多线程技术使得迅雷在不降低用户体验的前提下，对服务器资源进行均衡，有效降低了服务器负载。  

      每个用户在网上下载的文件都会在迅雷的服务器中进行数据记录，如有其他用户再下载同样的文件，迅雷的服务器会在它的数据库中搜索曾经下载过这些文件的用户，服务器再连接这些用户，通过用户已下载文件中的记录进行判断，如用户下载文件中仍存在此文件 (文件如改名或改变保存位置则无效)，用户将在不知不觉中扮演下载中间服务角色，上传文件。

**PPLive  
**      PPLive 软件的工作机制和 BitTorrent 十分类似，PPLive 将视频文件分成大小相等的片段，第三方提供播放的视频源，用户启矾 PPLive 以后，从 PPLive 服务器获得频道的列表，用户点击感兴趣的频道，然后从其他节点获得数据文件，使用流媒体实时传输协议 (RTP) 和实时传输控制协议 (RTCP) 进行数据的传输和控制。将数据下载到本地主机后，开放本地端口作为视频服务器，PPLive 的客户端播放器连接此端口，任何同一个局域网内的用户都可以通过连接这个地址收看到点播的节目。图 6 所示为 PPLive 的工作原理示意图。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

**Skype  
**      Skype 是网络语音沟通工具。它可以提供免费高清晰的语音对话，也可以用来拨打国内国际长途，还具备即时通讯所需的其他功能，比如文件传输、文字聊天等。Skype 是在 KaZaA 的基础上开发的，就像 KaZaA 一样，Skype 本身也是基于覆盖层的 P2P 网络，在它里面有两种类型的节点：普通节点和超级节点。普通节点是能传输语音和消息的一个功能实体；超级节点则类似于普通节点的网络网关，所有的普通节点必须与超级节点连接，并向 Skype 的登陆服务器注册它自己来加入 Skype 网络。Skype 的登陆服务器上存有用户名和密码，并且授权特定的用户加入 Skype 网络，图 7 所示为 Skype 的体系结构 [18]。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)  

      Skype 的另一个突出特点就是能够穿越地址转换设备和防火墙。Skype 能够在最小传输带宽 32 kb/s 的网络上提供高质量的语音。Skype 是使用 P2P 语音服务的代表。由于其具有超清晰语音质量、极强的穿透防火墙能力、免费多方通话以及高保密性等优点，成为互联网上使用最多的 P2P 应用之一。

**P2P 实现的原理**首先先介绍一些基本概念：     NAT(Network Address Translators)，网络地址转换：网络地址转换是在 IP 地址日益缺乏的情况下产生的，它的主要目的就是为了能够地址重用。NAT 从历史发展上分为两大类，基本的 NAT 和 NAPT(Network Address/Port Translator)。     最先提出的是基本的 NAT(peakflys 注：刚开始其实只是路由器上的一个功能模块)，它的产生基于如下事实：一个私有网络（域）中的节点中只有很少的节点需要与外网连接（这是在上世纪 90 年代中期提出的）。那么这个子网中其实只有少数的节点需要全球唯一的 IP 地址，其他的节点的 IP 地址应该是可以重用的。 因此，基本的 NAT 实现的功能很简单，在子网内使用一个保留的 IP 子网段，这些 IP 对外是不可见的。子网内只有少数一些 IP 地址可以对应到真正全球唯一的 IP 地址。如果这些节点需要访问外部网络，那么基本 NAT 就负责将这个节点的子网内 IP 转化为一个全球唯一的 IP 然后发送出去。(基本的 NAT 会改变 IP 包中的原 IP 地址，但是不会改变 IP 包中的端口) 关于基本的 NAT 可以参看 RFC 1631 另外一种 NAT 叫做 NAPT，从名称上我们也可以看得出，NAPT 不但会改变经过这个 NAT 设备的 IP 数据报的 IP 地址，还会改变 IP 数据报的 TCP/UDP 端口。基本 NAT 的设备可能我们见的不多（基本已经淘汰了），NAPT 才是我们真正需要关注的。看下图：  
![](http://www.cppblog.com/images/cppblog_com/peakflys/p2p.JPG)  
有一个私有网络 10.*.*.*，Client A 是其中的一台计算机，这个网络的网关（一个 NAT 设备）的外网 IP 是 155.99.25.11(应该还有一个内网的 IP 地址，比如 10.0.0.10)。如果 Client A 中的某个进程（这个进程创建了一个 UDP Socket, 这个 Socket 绑定 1234 端口）想访问外网主机 18.181.0.31 的 1235 端口，那么当数据包通过 NAT 时会发生什么事情呢？ 首先 NAT 会改变这个数据包的原 IP 地址，改为 155.99.25.11。接着 NAT 会为这个传输创建一个 Session（Session 是一个抽象的概念，如果是 TCP，也许 Session 是由一个 SYN 包开始，以一个 FIN 包结束。而 UDP 呢，以这个 IP 的这个端口的第一个 UDP 开始，结束呢，呵呵，也许是几分钟，也许是几小时，这要看具体的实现了）并且给这个 Session 分配一个端口，比如 62000，然后改变这个数据包的源端口为 62000。所以本来是 （10.0.0.1:1234->18.181.0.31:1235）的数据包到了互联网上变为了（155.99.25.11:62000->18.181.0.31:1235）。 一旦 NAT 创建了一个 Session 后，NAT 会记住 62000 端口对应的是 10.0.0.1 的 1234 端口，以后从 18.181.0.31 发送到 62000 端口的数据会被 NAT 自动的转发到 10.0.0.1 上。（注意：这里是说 18.181.0.31 发送到 62000 端口的数据会被转发，其他的 IP 发送到这个端口的数据将被 NAT 抛弃）这样 Client A 就与 Server S1 建立以了一个连接。 上面的是一些基础知识，下面的才是关键的部分了。 看看下面的情况：  
![](http://www.cppblog.com/images/cppblog_com/peakflys/p2p2.JPG)  
接上面的例子，如果 Client A 的原来那个 Socket(绑定了 1234 端口的那个 UDP Socket) 又接着向另外一个 Server S2 发送了一个 UDP 包，那么这个 UDP 包在通过 NAT 时会怎么样呢？ 这时可能会有两种情况发生，一种是 NAT 再次创建一个 Session，并且再次为这个 Session 分配一个端口号（比如：62001）。另外一种是 NAT 再次创建一个 Session，但是不会新分配一个端口号，而是用原来分配的端口号 62000。前一种 NAT 叫做 Symmetric NAT，后一种叫做 Cone NAT。如果你的 NAT 刚好是第一种，那么很可能会有很多 P2P 软件失灵。（可以庆幸的是，现在绝大多数的 NAT 属于后者，即 Cone NAT） peakflys 注：Cone NAT 具体又分为 3 种： (1) 全圆锥 ( Full Cone) : NAT 把所有来自相同内部 IP 地址和端口的请求映射到相同的外部 IP 地址和端口。任何一个外部主机均可通过该映射发送 IP 包到该内部主机。 (2) 限制性圆锥 (Restricted Cone) : NAT 把所有来自相同内部 IP 地址和端口的请求映射到相同的外部 IP 地址和端口。但是, 只有当内部主机先给 IP 地址为 X 的外部主机发送 IP 包, 该外部主机才能向该内部主机发送 IP 包。 (3) 端口限制性圆锥 ( Port Restricted Cone) : 端口限制性圆锥与限制性圆锥类似, 只是多了端口号的限制, 即只有内部主机先向 IP 地址为 X, 端口号为 P 的外部主机发送 1 个 IP 包, 该外部主机才能够把源端口号为 P 的 IP 包发送给该内部主机。 好了，我们看到，通过 NAT, 子网内的计算机向外连结是很容易的（NAT 相当于透明的，子网内的和外网的计算机不用知道 NAT 的情况）。 但是如果外部的计算机想访问子网内的计算机就比较困难了（而这正是 P2P 所需要的）。 那么我们如果想从外部发送一个数据报给内网的计算机有什么办法呢？首先，我们必须在内网的 NAT 上打上一个 “洞”（也就是前面我们说的在 NAT 上建立一个 Session），这个洞不能由外部来打，只能由内网内的主机来打。而且这个洞是有方向的，比如从内部某台主机（比如：192.168.0.10）向外部的某个 IP(比如：219.237.60.1) 发送一个 UDP 包，那么就在这个内网的 NAT 设备上打了一个方向为 219.237.60.1 的“洞”，（这就是称为 UDP Hole Punching 的技术）以后 219.237.60.1 就可以通过这个洞与内网的 192.168.0.10 联系了。（但是其他的 IP 不能利用这个洞）。  
**P2P 的常用实现**一、普通的直连式 P2P 实现 通过上面的理论，实现两个内网的主机通讯就差最后一步了：那就是鸡生蛋还是蛋生鸡的问题了，两边都无法主动发出连接请求，谁也不知道谁的公网地址，那我们如何来打这个洞呢？我们需要一个中间人来联系这两个内网主机。 现在我们来看看一个 P2P 软件的流程，以下图为例： ![](http://www.cppblog.com/images/cppblog_com/peakflys/P2P_test.jpg) 首先，Client A 登录服务器，NAT A 为这次的 Session 分配了一个端口 60000，那么 Server S 收到的 Client A 的地址是 202.187.45.3:60000，这就是 Client A 的外网地址了。同样，Client B 登录 Server S，NAT B 给此次 Session 分配的端口是 40000，那么 Server S 收到的 B 的地址是 187.34.1.56:40000。 此时，Client A 与 Client B 都可以与 Server S 通信了。如果 Client A 此时想直接发送信息给 Client B，那么他可以从 Server S 那儿获得 B 的公网地址 187.34.1.56:40000，是不是 Client A 向这个地址发送信息 Client B 就能收到了呢？答案是不行，因为如果这样发送信息，NAT B 会将这个信息丢弃（因为这样的信息是不请自来的，为了安全，大多数 NAT 都会执行丢弃动作）。现在我们需要的是在 NAT B 上打一个方向为 202.187.45.3（即 Client A 的外网地址）的洞，那么 Client A 发送到 187.34.1.56:40000 的信息, Client B 就能收到了。这个打洞命令由谁来发呢？自然是 Server S。 总结一下这个过程：如果 Client A 想向 Client B 发送信息，那么 Client A 发送命令给 Server S，请求 Server S 命令 Client B 向 Client A 方向打洞。然后 Client A 就可以通过 Client B 的外网 地址与 Client B 通信了。 注意：以上过程只适合于 Cone NAT 的情况，如果是 Symmetric NAT，那么当 Client B 向 Client A 打洞的端口已经重新分配了，Client B 将无法知道这个端口（如果 Symmetric NAT 的端口是顺序分配的，那么我们或许可以猜测这个端口号，可是由于可能导致失败的因素太多，这种情况下一般放弃 P2P  —peakflys）。  
二、STUN 方式的 P2P 实现 STUN 是 RFC3489 规定的一种 NAT 穿透方式，它采用辅助的方法探测 NAT 的 IP 和端口。毫无疑问的，它对穿越早期的 NAT 起了巨大的作用，并且还将继续在 NAT 穿透中占有一席之地。 STUN 的探测过程需要有一个公网 IP 的 STUN server，在 NAT 后面的 UAC 必须和此 server 配合，互相之间发送若干个 UDP 数据包。UDP 包中包含有 UAC 需要了解的信息，比如 NAT 外网 IP，PORT 等等。UAC 通过是否得到这个 UDP 包和包中的数据判断自己的 NAT 类型。 假设有如下 UAC（B），NAT（A），SERVER（C），UAC 的 IP 为 IPB，NAT 的 IP 为 IPA ，SERVER 的 IP 为 IPC1 、IPC2。请注意，服务器 C 有两个 IP，后面你会理解为什么需要两个 IP。 (1)NAT 的探测过程 STEP1：B 向 C 的 IPC1 的 port1 端口发送一个 UDP 包。C 收到这个包后，会把它收到包的源 IP 和 port 写到 UDP 包中，然后把此包通过 IP1C 和 port1 发还给 B。这个 IP 和 port 也就是 NAT 的外网 IP 和 port，也就是说你在 STEP1 中就得到了 NAT 的外网 IP。 熟悉 NAT 工作原理的应该都知道，C 返回给 B 的这个 UDP 包 B 一定收到。如果在你的应用中，向一个 STUN 服务器发送数据包后，你没有收到 STUN 的任何回应包，那只有两种可能：1、STUN 服务器不存在，或者你弄错了 port。2、你的 NAT 设备拒绝一切 UDP 包从外部向内部通过，如果排除防火墙限制规则，那么这样的 NAT 设备如果存在，那肯定是坏了„„ 当 B 收到此 UDP 后，把此 UDP 中的 IP 和自己的 IP 做比较，如果是一样的，就说明自己是在公网，下步 NAT 将去探测防火墙类型，就不多说了 (下面有图)。如果不一样，说明有 NAT 的存在，系统进行 STEP2 的操作。 STEP2：B 向 C 的 IPC1 发送一个 UDP 包，请求 C 通过另外一个 IPC2 和 PORT（不同与 SETP1 的 IP1）向 B 返回一个 UDP 数据包（现在知道为什么 C 要有两个 IP 了吧，为了检测 cone NAT 的类型）。 我们来分析一下，如果 B 收到了这个数据包，那说明什么？说明 NAT 来着不拒，不对数据包进行任何过滤，这也就是 STUN 标准中的 full cone NAT。遗憾的是，full cone nat 太少了，这也意味着你能收到这个数据包的可能性不大。如果没收到，那么系统进行 STEP3 的操作。 STEP3：B 向 C 的 IPC2 的 port2 发送一个数据包，C 收到数据包后，把它收到包的源 IP 和 port 写到 UDP 包中，然后通过自己的 IPC2 和 port2 把此包发还给 B。 和 step1 一样，B 肯定能收到这个回应 UDP 包。此包中的 port 是我们最关心的数据，下面我们来分析： 如果这个 port 和 step1 中的 port 一样，那么可以肯定这个 NAT 是个 CONE NAT，否则是对称 NAT。道理很简单：根据对称 NAT 的规则，当目的地址的 IP 和 port 有任何一个改变，那么 NAT 都会重新分配一个 port 使用，而在 step3 中，和 step1 对应，我们改变了 IP 和 port。因此，如果是对称 NAT, 那这两个 port 肯定是不同的。 如果在你的应用中，到此步的时候 PORT 是不同的，那就只能放弃 P2P 了，原因同上面实现中的一样。如果不同，那么只剩下了 restrict cone 和 port restrict cone。系统用 step4 探测是是那一种。 STEP4：B 向 C 的 IP2 的一个端口 PD 发送一个数据请求包，要求 C 用 IP2 和不同于 PD 的 port 返回一个数据包给 B。 我们来分析结果：如果 B 收到了，那也就意味着只要 IP 相同，即使 port 不同，NAT 也允许 UDP 包通过。显然这是 restrict cone NAT。如果没收到，没别的好说，port restrict NAT. 协议实现的算法运行图如下：  
![](http://www.cppblog.com/images/cppblog_com/peakflys/STUN.JPG) 一旦路经到达红色节点时，UDP 的沟通是没有可能性的 (peakflys 注：准备来说除了包被防火墙 blocked 之外，其他情况也是有可能建立 P2P 的，只是代价太大，一般放弃)。一旦通过黄色或是绿色的节点，就有连接的可能。 最终通过 STUN 服务器得到自己的 NAT 类型和公网 IP、Port，以后建立 P2P 时就非常容易了。

参考文章：

P2P 技术原理：[http://www.360doc.com/content/14/0305/17/8285430_357987074.shtml](http://www.360doc.com/content/14/0305/17/8285430_357987074.shtml)

P2P 技术现状及发展未来：[http://www.zte.com.cn/cndata/magazine/zte_communications/2007/6/magazine/200712](http://www.zte.com.cn/cndata/magazine/zte_communications/2007/6/magazine/200712/t20071220_150722.html)

P2P 原理及其常用的实现方式：[http://www.cppblog.com/peakflys/archive/2013/01/25/197562.html](http://www.cppblog.com/peakflys/archive/2013/01/25/197562.html)