> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zengkefu/p/5606696.html)

linux 诡异的半连接 (SYN_RECV) 队列长度(一)
-------------------------------

>> 转载请注明来源：[飘零的代码 piao2010 ’s blog](http://www.piao2010.com/ "飘零的代码 piao2010 ’s blog")，谢谢！^_^  
>> 本文链接地址：[linux 诡异的半连接 (SYN_RECV) 队列长度(一)](http://www.piao2010.com/linux%e8%af%a1%e5%bc%82%e7%9a%84%e5%8d%8a%e8%bf%9e%e6%8e%a5syn_recv%e9%98%9f%e5%88%97%e9%95%bf%e5%ba%a6%e4%b8%80 "linux诡异的半连接(SYN_RECV)队列长度(一)")

最近在学习 TCP 方面的基础知识，对于古老的 SYN Flood 也有了更多认识。SYN Flood 利用的是 TCP 协议缺陷，发送大量伪造的 TCP 连接请求，从而使得被攻击方资源耗尽（CPU 满负荷或内存不足）的攻击方式。  
SYN Flood 的原理简单，实现也不复杂，而且网上有许多现成的程序。

我在两台虚拟机上 (虚拟机 C 攻击虚拟机 S) 做测试，S 上跑了 apache 监听 80 端口，用 C 对 S 的 80 端口发送 SYN Flood，在无任何防护的情况下攻击效果显著。用 netstat 可以看见 80 端口存在大量的半连接状态(SYN_RECV)，用 tcpdump 抓包可以看见大量伪造 IP 发来的 SYN 连接，S 也不断回复 SYN+ACK 给对方，可惜对方并不存在 (如果存在则 S 会收到 RST 这样就失去效果了)，所以会超时重传。  
这个时候如果有正常客户 A 请求 S 的 80 端口，它的 SYN 包就被 S 丢弃了，因为半连接队列已经满了，达到攻击目的。

对于 SYN Flood 的防御一般会提到修改 net.ipv4.tcp_synack_retries, net.ipv4.tcp_syncookies, net.ipv4.tcp_max_syn_backlog  
目的就是减小 SYN+ACK 重传次数，增加半连接队列长度，启用 syn cookie。  
当 S 开启 syn cookie 的时候情况会缓解，一旦半连接队列满了系统就会启用 syn cookie 功能，同时在 / var/log/messages 记录 kernel: possible SYN flooding on port 80. Sending cookies.  
但也不是可以完全防御的，如果说攻击瞬间并发量足够大，毕竟 S 的 CPU 内存有限，一般大公司都有专业的防火墙设备来应对。

其中对于 net.ipv4.tcp_max_syn_backlog 的描述一般都称为半连接队列的长度，但在我实际测试的过程中却发现 SYN_RECV 状态的数量与 net.ipv4.tcp_max_syn_backlog 设置的值相差甚远。  
S 系统配置如下：  
net.ipv4.tcp_synack_retries = 5  
net.ipv4.tcp_syncookies = 0  
net.ipv4.tcp_max_syn_backlog = 4096  
但 SYN_RECV 状态的数量却只有 256

于是就开始相关资料，首先想到的是 TCP/IP 详解卷 1 中提到的 backlog，man 2 listen:  
int listen(int sockfd, int backlog);  
The backlog parameter defines the maximum length the queue of pending connections may grow to. If a connection request arrives with  
the queue full the client may receive an error with an indication of ECONNREFUSED or, if the underlying protocol supports retransmis-  
sion, the request may be ignored so that retries succeed.

NOTES  
The behaviour of the backlog parameter on TCP sockets changed with Linux 2.2. Now it specifies the queue length for completely estab-  
lished sockets waiting to be accepted, instead of the number of incomplete connection requests. The maximum length of the queue for  
incomplete sockets can be set using the tcp_max_syn_backlog sysctl. When syncookies are enabled there is no logical maximum length  
and this sysctl setting is ignored. See tcp(7) for more information.

可见 backlog 在 Linux 2.2 之后表示的是已完成三次握手但还未被应用程序 accept 的队列长度。

man 7 tcp:  
tcp_max_syn_backlog (integer; default: see below)  
The maximum number of queued connection requests which have still not received an acknowledgement from the connecting client.  
If this number is exceeded, the kernel will begin dropping requests. The default value of 256 is increased to 1024 when the  
memory present in the system is adequate or greater (>= 128Mb), and reduced to 128 for those systems with very low memory (<=  
32Mb). It is recommended that if this needs to be increased above 1024, TCP_SYNQ_HSIZE in include/net/tcp.h be modified to  
keep TCP_SYNQ_HSIZE*16<=tcp_max_syn_backlog, and the kernel be recompiled.

可见 tcp_max_syn_backlog 确实是半连接队列的长度，那为何会不准呢？  
这时候正好让同事也在两台机器上测试了一下，得到的数据居然与 tcp_max_syn_backlog 完全一致。  
开始怀疑是系统哪个地方配置有问题，又发现一个可疑的配置 net.core.somaxconn 它是 listen 的第二个参数 int backlog 的上限值，如果程序里的 backlog 大于  
net.core.somaxconn 的话就会取 net.core.somaxconn 的值。S 系统的 net.core.somaxconn = 128

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">//file:net/socket.c
&nbsp;
SYSCALL_DEFINE2(listen, int, fd, int, backlog)
{
	struct socket *sock;
	int err, fput_needed;
	int somaxconn;
&nbsp;
	sock = sockfd_lookup_light(fd, &amp;err, &amp;fput_needed);
	if (sock) {
		somaxconn = sock_net(sock-&gt;sk)-&gt;core.sysctl_somaxconn;
		//上限不超过somaxconn
		if ((unsigned)backlog &gt; somaxconn)
			backlog = somaxconn;
&nbsp;
		err = security_socket_listen(sock, backlog);
		if (!err)
			err = sock-&gt;ops-&gt;listen(sock, backlog);
&nbsp;
		fput_light(sock-&gt;file, fput_needed);
	}
	return err;
}</pre></td></tr></tbody></table>

查了 apache 文档关于 ListenBackLog 指令的说明，默认值是 511. 可见最终全连接队列 (backlog) 应该是 net.core.somaxconn = 128  
证实这点比较容易，用[慢连接攻击](http://www.piao2010.com/%E4%B8%80%E6%AC%A1%E5%88%86%E4%BA%AB%E5%BC%95%E5%8F%91%E7%9A%84%E8%A1%80%E6%A1%88-http-post-denial-of-service)测试观察到虚拟机 S 的 80 端口 ESTABLISHED 状态最大数量 384  
正好等于 256(apache prefork 模式 MaxClients 即 apache 可以响应的最大并发连接数) + 128(backlog 即已完成三次握手等待 apache accept 的最大连接数)。说明全连接队列长度等于 min(backlog,somaxconn);  
好久没写这么多文字了，下回 [linux 诡异的半连接 (SYN_RECV) 队列长度(二)](http://www.piao2010.com/linux%E8%AF%A1%E5%BC%82%E7%9A%84%E5%8D%8A%E8%BF%9E%E6%8E%A5syn_recv%E9%98%9F%E5%88%97%E9%95%BF%E5%BA%A6%E4%BA%8C) 继续

 

 

 

linux 诡异的半连接 (SYN_RECV) 队列长度(二)
-------------------------------

>> 转载请注明来源：[飘零的代码 piao2010 ’s blog](http://www.piao2010.com/ "飘零的代码 piao2010 ’s blog")，谢谢！^_^  
>> 本文链接地址：[linux 诡异的半连接 (SYN_RECV) 队列长度(二)](http://www.piao2010.com/linux%e8%af%a1%e5%bc%82%e7%9a%84%e5%8d%8a%e8%bf%9e%e6%8e%a5syn_recv%e9%98%9f%e5%88%97%e9%95%bf%e5%ba%a6%e4%ba%8c "linux诡异的半连接(SYN_RECV)队列长度(二)")

继续上回：我们已经确认了全连接队列的长度计算，接下来继续寻找半连接队列长度。  
试着慢慢减小 tcp_max_syn_backlog 的值，但还是看不到半连接状态数量的变化。  
实在没什么思路，只能 Google 之，搜出来的基本都是关于 SYN Flood 的文章，难道没同学关注过半连接队列的长度吗？  
困扰数日终于在某个夜晚被我找一篇题为[《关于半连接队列的释疑》](http://blog.chinaunix.net/space.php?uid=20357359&do=blog&id=1963498)的文章，激动呐。根据作者提供的思路我开始翻代码，注意我用的内核版本 2.6.32，不同版本代码也有差异。  
首先定位到 tcp_v4_conn_request 函数，在文件 netipv4tcp_ipv4.c 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">int tcp_v4_conn_request(struct sock *sk, struct sk_buff *skb)
{
	struct inet_request_sock *ireq;
	struct tcp_options_received tmp_opt;
	struct request_sock *req;
	__be32 saddr = ip_hdr(skb)-&gt;saddr;
	__be32 daddr = ip_hdr(skb)-&gt;daddr;
	__u32 isn = TCP_SKB_CB(skb)-&gt;when;
	struct dst_entry *dst = NULL;
#ifdef CONFIG_SYN_COOKIES
	int want_cookie = 0;
#else
#define want_cookie 0 /* Argh, why doesn't gcc optimize this :( */
#endif
&nbsp;
	/* Never answer to SYNs send to broadcast or multicast */
	if (skb-&gt;rtable-&gt;rt_flags &amp; (RTCF_BROADCAST | RTCF_MULTICAST))
		goto drop;
&nbsp;
	/* TW buckets are converted to open requests without
	 * limitations, they conserve resources and peer is
	 * evidently real one.
	 */
&nbsp;
	//关键函数inet_csk_reqsk_queue_is_full
	if (inet_csk_reqsk_queue_is_full(sk) &amp;&amp; !isn) {
#ifdef CONFIG_SYN_COOKIES
		if (sysctl_tcp_syncookies) {
			want_cookie = 1;
		} else
#endif
		goto drop;
	}
&nbsp;
	/* Accept backlog is full. If we have already queued enough
	 * of warm entries in syn queue, drop request. It is better than
	 * clogging syn queue with openreqs with exponentially increasing
	 * timeout.
	 */
	if (sk_acceptq_is_full(sk) &amp;&amp; inet_csk_reqsk_queue_young(sk) &gt; 1)
		goto drop;
&nbsp;
	req = inet_reqsk_alloc(&amp;tcp_request_sock_ops);
	if (!req)
		goto drop;
省略N多代码</pre></td></tr></tbody></table>

跟进关键函数 inet_csk_reqsk_queue_is_full，在文件 includenetinet_connection_sock.h 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">static inline int inet_csk_reqsk_queue_is_full(const struct sock *sk)
{
	return reqsk_queue_is_full(&amp;inet_csk(sk)-&gt;icsk_accept_queue);
}</pre></td></tr></tbody></table>

跟进关键函数 reqsk_queue_is_full，在文件 includenetrequest_sock.h 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">static inline int reqsk_queue_is_full(const struct request_sock_queue *queue)
{
        //注意这里是用&gt;&gt;(右移)来判断的，不是大于号
	return queue-&gt;listen_opt-&gt;qlen &gt;&gt; queue-&gt;listen_opt-&gt;max_qlen_log;
}</pre></td></tr></tbody></table>

查找 qlen 和 max_qlen_log 的定义，在文件 includenetrequest_sock.h 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">/** struct listen_sock - listen state
 *
 * @max_qlen_log - log_2 of maximal queued SYNs/REQUESTs
 */
struct listen_sock {
	u8			max_qlen_log;// 2^max_qlen_log = 半连接队列最大长度
	/* 3 bytes hole, try to use */
	int			qlen;//全连接队列的当前长度
	int			qlen_young;
	int			clock_hand;
	u32			hash_rnd;
	u32			nr_table_entries;
	struct request_sock	*syn_table[0];
};</pre></td></tr></tbody></table>

可见关键是如何计算 max_qlen_log，[前一篇博客](http://www.piao2010.com/linux%E8%AF%A1%E5%BC%82%E7%9A%84%E5%8D%8A%E8%BF%9E%E6%8E%A5syn_recv%E9%98%9F%E5%88%97%E9%95%BF%E5%BA%A6%E4%B8%80)提到了 listen 的系统调用：

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">//file:net/socket.c
&nbsp;
SYSCALL_DEFINE2(listen, int, fd, int, backlog)
{
	struct socket *sock;
	int err, fput_needed;
	int somaxconn;
&nbsp;
	sock = sockfd_lookup_light(fd, &amp;err, &amp;fput_needed);
	if (sock) {
		somaxconn = sock_net(sock-&gt;sk)-&gt;core.sysctl_somaxconn;
		//上限不超过somaxconn
		if ((unsigned)backlog &gt; somaxconn)
			backlog = somaxconn;
&nbsp;
		err = security_socket_listen(sock, backlog);
		if (!err)
		        //这里是关键。
			err = sock-&gt;ops-&gt;listen(sock, backlog);
&nbsp;
		fput_light(sock-&gt;file, fput_needed);
	}
	return err;
}</pre></td></tr></tbody></table>

sock->ops->listen 其实是 inet_listen，在文件 netipv4af_inet.c 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cs">int inet_listen(struct socket *sock, int backlog)
{
	struct sock *sk = sock-&gt;sk;
	unsigned char old_state;
	int err;
&nbsp;
	lock_sock(sk);
&nbsp;
	err = -EINVAL;
	if (sock-&gt;state != SS_UNCONNECTED || sock-&gt;type != SOCK_STREAM)
		goto out;
&nbsp;
	old_state = sk-&gt;sk_state;
	if (!((1 &lt;&lt; old_state) &amp; (TCPF_CLOSE | TCPF_LISTEN)))
		goto out;
&nbsp;
	/* Really, if the socket is already in listen state
	 * we can only allow the backlog to be adjusted.
	 */
	if (old_state != TCP_LISTEN) {
	        //关键函数inet_csk_listen_start
		err = inet_csk_listen_start(sk, backlog);
		if (err)
			goto out;
	}
	sk-&gt;sk_max_ack_backlog = backlog;
	err = 0;
&nbsp;
out:
	release_sock(sk);
	return err;
}</pre></td></tr></tbody></table>

跟进 inet_csk_listen_start，在文件 netipv4inet_connection_sock.c 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">int inet_csk_listen_start(struct sock *sk, const int nr_table_entries)
{
	struct inet_sock *inet = inet_sk(sk);
	struct inet_connection_sock *icsk = inet_csk(sk);
&nbsp;
	//关键函数reqsk_queue_alloc
	int rc = reqsk_queue_alloc(&amp;icsk-&gt;icsk_accept_queue, nr_table_entries);
  //后面省略
}</pre></td></tr></tbody></table>

跟进 reqsk_queue_alloc，在文件 netcorerequest_sock.c 中。

[?](http://www.ericbess.com/ericblog/2008/03/03/wp-codebox/#examples "WP-CodeBox HowTo?")View Code C  <table><tbody><tr><td><pre class="hljs cpp">int reqsk_queue_alloc(struct request_sock_queue *queue,
		      unsigned int nr_table_entries)
{
	size_t lopt_size = sizeof(struct listen_sock);
	struct listen_sock *lopt;
&nbsp;
  //这里开始影响到nr_table_entries的取值，内核版本小于2.6.20的话nr_table_entries是不会修改的
        nr_table_entries = min_t(u32, nr_table_entries, sysctl_max_syn_backlog);
	nr_table_entries = max_t(u32, nr_table_entries, 8);
	nr_table_entries = roundup_pow_of_two(nr_table_entries + 1);
&nbsp;
	//nr_table_entries到这里已经确定
	lopt_size += nr_table_entries * sizeof(struct request_sock *);
	if (lopt_size &gt; PAGE_SIZE)
		lopt = __vmalloc(lopt_size,
			GFP_KERNEL | __GFP_HIGHMEM | __GFP_ZERO,
			PAGE_KERNEL);
	else
		lopt = kzalloc(lopt_size, GFP_KERNEL);
	if (lopt == NULL)
		return -ENOMEM;
&nbsp;
  //这里确定了lopt-&gt;max_qlen_log的值
	for (lopt-&gt;max_qlen_log = 3;
	     (1 &lt;&lt; lopt-&gt;max_qlen_log) &lt; nr_table_entries;//内核版本小于2.6.20的话这里是sysctl_max_syn_backlog
	     lopt-&gt;max_qlen_log++);
&nbsp;
	get_random_bytes(&amp;lopt-&gt;hash_rnd, sizeof(lopt-&gt;hash_rnd));
	rwlock_init(&amp;queue-&gt;syn_wait_lock);
	queue-&gt;rskq_accept_head = NULL;
	lopt-&gt;nr_table_entries = nr_table_entries;
&nbsp;
	write_lock_bh(&amp;queue-&gt;syn_wait_lock);
	queue-&gt;listen_opt = lopt;
	write_unlock_bh(&amp;queue-&gt;syn_wait_lock);
&nbsp;
	return 0;
}</pre></td></tr></tbody></table>

代码到此为止，然后我们计算一下为何在虚拟机 S 上的 SYN_RECV 状态数量会是 256

nr_table_entries = listen 的第二个参数 int backlog ，上限是系统的 somaxconn  
若 somaxconn = 128 sysctl_max_syn_backlog = 4096 backlog = 511 则 nr_table_entries = 128

nr_table_entries = min_t(u32, nr_table_entries, sysctl_max_syn_backlog);  
取两者较小的一个 nr_table_entries = 128

nr_table_entries = max_t(u32, nr_table_entries, 8);  
取两者较大的一个 nr_table_entries = 128

nr_table_entries = roundup_pow_of_two(nr_table_entries + 1); //roundup_pow_of_two - round the given value up to nearest power of two  
roundup_pow_of_two(128 + 1) = 256

for (lopt->max_qlen_log = 3; (1 << lopt->max_qlen_log) < nr_table_entries; lopt->max_qlen_log++);  
max_qlen_log = 8

判断半连接队列是否满 queue->listen_opt->qlen >> queue->listen_opt->max_qlen_log;  
queue->listen_opt->qlen = 256 时 reqsk_queue_is_full 返回 1 , 进入 drop  
所以 queue->listen_opt->qlen 取值 0~255， 因此 SYN_RECV 状态数量会是 256

另外同事的测试结果为何与我的不同？  
因为内核版本小于 2.6.20 的话 max_qlen_log 是直接由 sysctl_max_syn_backlog 决定的，所以半连接队列的长度就是等于 sysctl_max_syn_backlog  
文章有点长，不过总算是把问题给解决了。这里要特别感谢雨哥 ([博客](http://blog.chinaunix.net/space.php?uid=20043340))，很多代码是他带着我分析的。