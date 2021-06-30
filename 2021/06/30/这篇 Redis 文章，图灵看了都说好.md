> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/k8agEub4qmhm3kX_TpETrA)

> 2007 年，他和朋友一起创建了一个网站。为了解决这个网站的负载问题，他自己定制了一个数据库。2009 年开发的，这个是 Redis。这位意大利程序员是萨尔瓦托勒 · 桑菲利波 (Salvatore Sanfilippo)，他被称为 Redis 之父，更广为人知的名字是 Antirez。

### 一、Redis 简介

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFoKdRTxPCsUjpHJhjFhuPnicOUVdZfspC4cG3mC0jV36OibZN1s4b0sLA/640?wx_fmt=jpeg)

### 二、内存模型

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFzfsW38OIiaG3OkXJVJL3lOYyxqatLWZiaXFrJSG2Fsko13bs7D7HcGJw/640?wx_fmt=jpeg)

1.  used_memory：从 Redis 角度使用了多少内存，即 Redis 分配器分配的内存总量（单位是字节），**包括使用的虚拟内存**（即 swap）；
    
2.  used_memory_rss：从操作系统角度实际使用量，即 Redis 进程占据操作系统的内存（单位是字节），**包括进程运行本身需要的内存、内存碎片**等，**不包括虚拟内存**。一般情况下 used_memory_rss 都要比 used_memory 大，因为 Redis 频繁删除读写等操作使得内存碎片较多，而虚拟内存的使用一般是非极端情况下是不怎么使用的；
    
3.  mem_fragmentation_ratio：即内存碎片比率，该值是 used_memory_rss / used_memory 的比值；mem_fragmentation_ratio 一般大于 1，且该值越大，内存碎片比例越大。如果 mem_fragmentation_ratio<1，说明 Redis 使用了虚拟内存，由于虚拟内存的媒介是磁盘，比内存速度要慢很多，当这种情况出现时，应该及时排查，如果内存不足应该及时处理，如增加 Redis 节点、增加 Redis 服务器的内存、优化应用等。一般来说，mem_fragmentation_ratio 在 1.03 左右是比较健康的状态（对于 jemalloc 来说）；
    
4.  mem_allocator：即 Redis 使用的内存分配器，一般默认是 jemalloc。
    

#### Redis 内存划分

1.  数据
    

2.  进程本身运行需要的内存
    

3.  缓冲内存
    

缓冲内存包括：

*   客户端缓冲区：存储客户端连接的输入输出缓冲；
    
*   复制积压缓冲区：用于部分复制功能；
    
*   aof_buf：用于在进行 AOF 重写时，保存最近的写入命令。
    

这部分内存由 jemalloc 分配，因此会统计在 used_memory 中。

4.  内存碎片
    

内存碎片是 Redis 在数据更改频繁分配、回收物理内存过程中产生的。

内存碎片不会统计在 used_memory 中。

#### Redis 数据存储的细节

下面看一张经典的图。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFICVxiaVEJYclTkaiapssnicYB2zTccY8gY7fNlX2yohYOiaAAhbunQmqaA/640?wx_fmt=jpeg)

1.  jemalloc
    

无论是 DictEntry 对象，还是 RedisObject、SDS 对象，都需要内存分配器（如 jemalloc）分配内存进行存储。Redis 在编译时便会指定内存分配器；内存分配器可以是 libc 、jemalloc 或者 tcmalloc，默认是 jemalloc。当 Redis 存储数据时，会选择大小最合适的内存块进行存储。

jemalloc 划分的内存单元如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFUCyo8yjBgMyI2bvYicyEgu6yz7YgDfhqYkw4rKc0V7vjdNmQbxM1suw/640?wx_fmt=jpeg)

2.  dictEntry
    

每个 dictEntry 都保存着一个键值对，key 值保存一个 sds 结构体，value 值保存一个 redisObject 结构体。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFI09ubvibKyVRruWbw5AckbCrAEUv12rPhiblqRiavHNArS5icbuknJZQBA/640?wx_fmt=jpeg)

3.  redisObject
    

前面说到，Redis 对象有 5 种类型；无论是哪种类型，Redis 都不会直接存储，而是通过 RedisObject 对象进行存储。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFPe5KtVJ2MKjrEDe8ltwOMLMMen2FjsTrQ9Z2LePVytJw9m9VM6iaPtw/640?wx_fmt=jpeg)

RedisObject 的每个字段的含义和作用如下：

*   type
    

type 字段表示对象的类型，占 4 个比特；目前包括 REDIS_STRING(字符串)、REDIS_LIST (列表)、REDIS_HASH(哈希)、REDIS_SET(集合)、REDIS_ZSET(有序集合)。

*   encoding
    

encoding 表示对象的内部编码，占 4 个比特 (redis-3.0)。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFyBfPvpzOJas7FqtnfAtERvhZroP7NwicZqO1Bulsia9S5UtLVIRFJvoA/640?wx_fmt=jpeg)

*   lru
    

lru 记录的是对象最后一次被命令程序访问的时间，占据的比特数不同的版本有所不同（如 4.0 版本占 24 比特，2.6 版本占 22 比特）。

*   refcount
    

refcount 记录的是该对象被引用的次数，类型为整型。refcount 的作用，主要在于对象的引用计数和内存回收：

*   当创建新对象时，refcount 初始化为 1；
    
*   当有新程序使用该对象时，refcount 加 1；
    
*   当对象不再被一个新程序使用时，refcount 减 1；
    
*   当 refcount 变为 0 时，对象占用的内存会被释放。
    

Redis 中被多次使用的对象 (refcount>1) 称为共享对象。Redis 为了节省内存，当有一些对象重复出现时，新的程序不会创建新的对象，而是仍然使用原来的对象。这个被重复使用的对象，就是共享对象。目前共享对象仅支持整数值的字符串对象。

**共享对象的具体实现**

Redis 的共享对象目前只支持整数值的字符串对象。之所以如此，实际上是对内存和 CPU（时间）的平衡：共享对象虽然会降低内存消耗，但是判断两个对象是否相等却需要消耗额外的时间。

对于整数值，判断操作复杂度为 O(1)；

对于普通字符串，判断复杂度为 O(n)；

而对于哈希、列表、集合和有序集合，判断的复杂度为 O(n^2)。

虽然共享对象只能是整数值的字符串对象，但是 5 种类型都可能使用共享对象（如哈希、列表等的元素可以使用）。

就目前的实现来说，Redis 服务器在初始化时，会创建 10000 个字符串对象，值分别是 0~9999 的整数值；当 Redis 需要使用值为 0~9999 的字符串对象时，可以直接使用这些共享对象。10000 这个数字可以通过调整参数 REDIS_SHARED_INTEGERS（4.0 中是 OBJ_SHARED_INTEGERS）的值进行改变。

*   ptr
    

ptr 指针指向具体的数据，如前面的例子中，set hello world，ptr 指向包含字符串 world 的 SDS。

4.  SDS
    

Redis 没有直接使用 C 字符串 (即以空字符‘\0’结尾的字符数组) 作为默认的字符串表示，而是使用了 SDS。SDS 是简单动态字符串 (Simple Dynamic String) 的缩写。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFvooGEHedRSGMQV8AjkWvS6xY5kvpFHCJx8nT7stJIhADccKv3R26Hw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFJbDAgS42V0YOPVEAO52Wa23EUAmMfZANJkEF3W6decNRdMbSyrs9vQ/640?wx_fmt=jpeg)

通过 SDS 的结构可以看出，buf 数组的长度 = free+len+1（其中 1 表示字符串结尾的空字符）；所以，一个 SDS 结构占据的空间为：free 所占长度 + len 所占长度 + buf 数组的长度 = 4+4+free+len+1=free+len+9。

**为什么使用 SDS 而不直接使用 C 字符串？**

SDS 在 C 字符串的基础上加入了 free 和 len 字段，带来了很多好处：

*   获取字符串长度：SDS 是 O(1)，C 字符串是 O(n)。
    
*   缓冲区溢出：使用 C 字符串的 API 时，如果字符串长度增加（如 strcat 操作）而忘记重新分配内存，很容易造成缓冲区的溢出；而 SDS 由于记录了长度，相应的 API 在可能造成缓冲区溢出时会自动重新分配内存，杜绝了缓冲区溢出。
    
*   修改字符串时内存的重分配：对于 C 字符串，如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。而对于 SDS，由于可以记录 len 和 free，因此解除了字符串长度和空间数组长度之间的关联，可以在此基础上进行优化——空间预分配策略（即分配内存时比实际需要的多）使得字符串长度增大时重新分配内存的概率大大减小；惰性空间释放策略使得字符串长度减小时重新分配内存的概率大大减小。
    
*   存取二进制数据：SDS 可以，C 字符串不可以。因为 C 字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此 C 字符串无法正确存取；而 SDS 以字符串长度 len 来作为字符串结束标识，因此没有这个问题。
    

### 三、持久化 Persistence

持久化的功能：Redis 是内存数据库，数据都是存储在内存中，为了避免进程退出导致数据的永久丢失，需要定期将 Redis 中的数据以某种形式（数据或命令）从内存保存到硬盘。当下次 Redis 重启时，利用持久化文件实现数据恢复。除此之外，为了进行灾难备份，可以将持久化文件拷贝到一个远程位置。

Redis 持久化分为 RDB 持久化和 AOF 持久化，前者将当前数据保存到硬盘，后者则是将每次执行的写命令保存到硬盘（类似于 MySQL 的 Binlog）。由于 AOF 持久化的实时性更好，即当进程意外退出时丢失的数据更少，因此 AOF 是目前主流的持久化方式，不过 RDB 持久化仍然有其用武之地。

#### RDB 持久化

RDB(Redis Database)持久化方式能够在指定的时间间隔能对你的数据进行快照存储。一般通过 bgsave 命令会创建一个子进程，由子进程来负责创建 RDB 文件，父进程 (即 Redis 主进程) 则继续处理请求。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFoV7uO367HQhWDICfNumicZ0bc6TdHTgF0wbDJMjCkia8hDYO1ibMicqIAA/640?wx_fmt=jpeg)

图片中的 5 个步骤所进行的操作如下：

1.  Redis 父进程首先判断：当前是否在执行 save，或 bgsave/bgrewriteaof（后面会详细介绍该命令）的子进程，如果在执行则 bgsave 命令直接返回。bgsave/bgrewriteaof 的子进程不能同时执行，主要是基于性能方面的考虑：两个并发的子进程同时执行大量的磁盘写操作，可能引起严重的性能问题。
    
2.  父进程执行 fork 操作创建子进程，这个过程中父进程是阻塞的，Redis 不能执行来自客户端的任何命令；
    
3.  父进程 fork 后，bgsave 命令返回”Background saving started” 信息并不再阻塞父进程，并可以响应其他命令；
    
4.  子进程进程对内存数据生成快照文件；
    
5.  子进程发送信号给父进程表示完成，父进程更新统计信息。
    

**这里补充一下第 4 点是如何生成 RDB 文件的。一定有读者也有疑问：在同步到磁盘和持续写入这个过程是如何处理数据不一致的情况呢？生成快照 RDB 文件时是否会对业务产生影响？**

1.  通过 fork 创建的子进程能够获得和父进程完全相同的内存空间，父进程对内存的修改对于子进程是不可见的，两者不会相互影响；
    
2.  通过 fork 创建子进程时不会立刻触发大量内存的拷贝，采用的是写时拷贝 COW (Copy On Write)。内核只为新生成的子进程创建虚拟空间结构，它们来复制于父进程的虚拟究竟结构，但是不为这些段分配物理内存，它们共享父进程的物理空间，当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间；
    

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFFj3eEElMHJ1fVZBxuGokmnEtt8FMd8QiaKia0DwQvVib5XXbU2Wqn8kFA/640?wx_fmt=jpeg)

#### AOF 持久化

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFKZhjqWAl6qTKSpgHYpuAaPXfibpfK54vclqH46EH7mLPyKXYqficegZA/640?wx_fmt=jpeg)

AOF(Append Only File) 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF 命令以 redis 协议追加保存每次写的操作到文件末尾。Redis 还能对 AOF 文件进行后台重写, 使得 AOF 文件的体积不至于过大。

AOF 的执行流程包括：

#### 命令追加 (append)

Redis 先将写命令追加到缓冲区 aof_buf，而不是直接写入文件，主要是为了避免每次有写命令都直接写入硬盘，导致硬盘 IO 成为 Redis 负载的瓶颈。

#### 文件写入 (write) 和文件同步(sync)

根据不同的同步策略将 aof_buf 中的内容同步到硬盘；

Linux 操作系统中为了提升性能，使用了页缓存（page cache）。当我们将 aof_buf 的内容写到磁盘上时，此时数据并没有真正的落盘，而是在 page cache 中，为了将 page cache 中的数据真正落盘，需要执行 fsync / fdatasync 命令来强制刷盘。这边的文件同步做的就是刷盘操作，或者叫文件刷盘可能更容易理解一些。

AOF 缓存区的同步文件策略由参数 appendfsync 控制，各个值的含义如下：

*   always：命令写入 aof_buf 后立即调用系统 write 操作和系统 fsync 操作同步到 AOF 文件，fsync 完成后线程返回。这种情况下，每次有写命令都要同步到 AOF 文件，硬盘 IO 成为性能瓶颈，Redis 只能支持大约几百 TPS 写入，严重降低了 Redis 的性能；即便是使用固态硬盘（SSD），每秒大约也只能处理几万个命令，而且会大大降低 SSD 的寿命。**可靠性较高，数据基本不丢失。**
    
*   no：命令写入 aof_buf 后调用系统 write 操作，不对 AOF 文件做 fsync 同步；同步由操作系统负责，通常同步周期为 30 秒。这种情况下，文件同步的时间不可控，且缓冲区中堆积的数据会很多，数据安全性无法保证。
    
*   everysec：命令写入 aof_buf 后调用系统 write 操作，write 完成后线程返回；fsync 同步文件操作由专门的线程每秒调用一次。everysec 是前述两种策略的折中，是性能和数据安全性的平衡，因此是 Redis 的默认配置，也是我们推荐的配置。
    

**有同学可能会疑问为什么 always 策略还是不能 100% 保障数据不丢失，例如在开启 AOF 的情况下，有一条写命令，Redis 在写命令执行完，写 aof_buf 未成功的情况下宕机了?**

不能，Redis 就不能 100% 保证数据不丢失。

```
void flushAppendOnlyFile(int force) {
    ssize_t nwritten;
    int sync_in_progress = 0;
    mstime_t latency;

    if (sdslen(server.aof_buf) == 0) return;

    if (server.aof_fsync == AOF_FSYNC_EVERYSEC)
        sync_in_progress = bioPendingJobsOfType(REDIS_BIO_AOF_FSYNC) != 0;

    if (server.aof_fsync == AOF_FSYNC_EVERYSEC && !force) {
        /* With this append fsync policy we do background fsyncing.
         * If the fsync is still in progress we can try to delay
         * the write for a couple of seconds. */
        if (sync_in_progress) {
            if (server.aof_flush_postponed_start == 0) {
                /* No previous write postponing, remember that we are
                 * postponing the flush and return. */
                server.aof_flush_postponed_start = server.unixtime;
                return;
            } else if (server.unixtime - server.aof_flush_postponed_start < 2) {
                /* We were already waiting for fsync to finish, but for less
                 * than two seconds this is still ok. Postpone again. */
                return;
            }
            /* Otherwise fall trough, and go write since we can't wait
             * over two seconds. */
            server.aof_delayed_fsync++;
            redisLog(REDIS_NOTICE,"Asynchronous AOF fsync is taking too long (disk is busy?). Writing the AOF buffer without waiting for fsync to complete, this may slow down Redis.");
        }
    }
    /* We want to perform a single write. This should be guaranteed atomic
     * at least if the filesystem we are writing is a real physical one.
     * While this will save us against the server being killed I don't think
     * there is much to do about the whole server stopping for power problems
     * or alike */

    latencyStartMonitor(latency);
    nwritten = write(server.aof_fd,server.aof_buf,sdslen(server.aof_buf));
    latencyEndMonitor(latency);
    /* We want to capture different events for delayed writes:
     * when the delay happens with a pending fsync, or with a saving child
     * active, and when the above two conditions are missing.
     * We also use an additional event name to save all samples which is
     * useful for graphing / monitoring purposes. */
    if (sync_in_progress) {
        latencyAddSampleIfNeeded("aof-write-pending-fsync",latency);
    } else if (server.aof_child_pid != -1 || server.rdb_child_pid != -1) {
        latencyAddSampleIfNeeded("aof-write-active-child",latency);
    } else {
        latencyAddSampleIfNeeded("aof-write-alone",latency);
    }
    latencyAddSampleIfNeeded("aof-write",latency);

    /* We performed the write so reset the postponed flush sentinel to zero. */
    server.aof_flush_postponed_start = 0;

    if (nwritten != (signed)sdslen(server.aof_buf)) {
        static time_t last_write_error_log = 0;
        int can_log = 0;

        /* Limit logging rate to 1 line per AOF_WRITE_LOG_ERROR_RATE seconds. */
        if ((server.unixtime - last_write_error_log) > AOF_WRITE_LOG_ERROR_RATE) {
            can_log = 1;
            last_write_error_log = server.unixtime;
        }

        /* Log the AOF write error and record the error code. */
        if (nwritten == -1) {
            if (can_log) {
                redisLog(REDIS_WARNING,"Error writing to the AOF file: %s",
                    strerror(errno));
                server.aof_last_write_errno = errno;
            }
        } else {
            if (can_log) {
                redisLog(REDIS_WARNING,"Short write while writing to "
                                       "the AOF file: (nwritten=%lld, "
                                       "expected=%lld)",
                                       (long long)nwritten,
                                       (long long)sdslen(server.aof_buf));
            }

            if (ftruncate(server.aof_fd, server.aof_current_size) == -1) {
                if (can_log) {
                    redisLog(REDIS_WARNING, "Could not remove short write "
                             "from the append-only file.  Redis may refuse "
                             "to load the AOF the next time it starts.  "
                             "ftruncate: %s", strerror(errno));
                }
            } else {
                /* If the ftruncate() succeeded we can set nwritten to
                 * -1 since there is no longer partial data into the AOF. */
                nwritten = -1;
            }
            server.aof_last_write_errno = ENOSPC;
        }

        /* Handle the AOF write error. */
        if (server.aof_fsync == AOF_FSYNC_ALWAYS) {
            /* We can't recover when the fsync policy is ALWAYS since the
             * reply for the client is already in the output buffers, and we
             * have the contract with the user that on acknowledged write data
             * is synced on disk. */
            redisLog(REDIS_WARNING,"Can't recover from AOF write error when the AOF fsync policy is 'always'. Exiting...");
            exit(1);
        } else {
            /* Recover from failed write leaving data into the buffer. However
             * set an error to stop accepting writes as long as the error
             * condition is not cleared. */
            server.aof_last_write_status = REDIS_ERR;

            /* Trim the sds buffer if there was a partial write, and there
             * was no way to undo it with ftruncate(2). */
            if (nwritten > 0) {
                server.aof_current_size += nwritten;
                sdsrange(server.aof_buf,nwritten,-1);
            }
            return; /* We'll try again on the next call... */
        }
    } else {
        /* Successful write(2). If AOF was in error state, restore the
         * OK state and log the event. */
        if (server.aof_last_write_status == REDIS_ERR) {
            redisLog(REDIS_WARNING,
                "AOF write error looks solved, Redis can write again.");
            server.aof_last_write_status = REDIS_OK;
        }
    }
    server.aof_current_size += nwritten;

    /* Re-use AOF buffer when it is small enough. The maximum comes from the
     * arena size of 4k minus some overhead (but is otherwise arbitrary). */
    if ((sdslen(server.aof_buf)+sdsavail(server.aof_buf)) < 4000) {
        sdsclear(server.aof_buf);
    } else {
        sdsfree(server.aof_buf);
        server.aof_buf = sdsempty();
    }

    /* Don't fsync if no-appendfsync-on-rewrite is set to yes and there are
     * children doing I/O in the background. */
    if (server.aof_no_fsync_on_rewrite &&
        (server.aof_child_pid != -1 || server.rdb_child_pid != -1))
            return;

    /* Perform the fsync if needed. */
    if (server.aof_fsync == AOF_FSYNC_ALWAYS) {
        /* aof_fsync is defined as fdatasync() for Linux in order to avoid
         * flushing metadata. */
        latencyStartMonitor(latency);
        aof_fsync(server.aof_fd); /* Let's try to get this data on the disk */
        latencyEndMonitor(latency);
        latencyAddSampleIfNeeded("aof-fsync-always",latency);
        server.aof_last_fsync = server.unixtime;
    } else if ((server.aof_fsync == AOF_FSYNC_EVERYSEC &&
                server.unixtime > server.aof_last_fsync)) {
        if (!sync_in_progress) aof_background_fsync(server.aof_fd);
        server.aof_last_fsync = server.unixtime;
    }
}


```

那么从上面 redis-3.0 的源码及上下文

```
if (server.aof_fsync == AOF_FSYNC_ALWAYS)


```

分析得出，其实我们每次执行客户端命令的时候操作并没有写到 aof 文件中，只是写到了 aof_buf 内存当中，只有当下一个事件来临时，才会去 fsync 到 disk 中，从 redis 的这种策略上我们也可以看出，redis 和 mysql 在数据持久化之间的区别，redis 的数据持久化仅仅就是一个附带功能，并不是其主要功能。

结论：**Redis 即使在配制 appendfsync=always 的策略下，还是会丢失一个事件循环的数据。**

#### 文件重写 (rewrite)

定期重写 AOF 文件，达到压缩的目的。

AOF 重写是 AOF 持久化的一个机制，用来压缩 AOF 文件，通过 fork 一个子进程，重新写一个新的 AOF 文件，该次重写不是读取旧的 AOF 文件进行复制，而是读取内存中的 Redis 数据库，重写一份 AOF 文件，有点类似于 RDB 的快照方式。

文件重写之所以能够压缩 AOF 文件，原因在于：

*   过期的数据不再写入文件
    
*   无效的命令不再写入文件：如有些数据被重复设值 (set mykey v1, set mykey v2)、有些数据被删除了(sadd myset v1, del myset) 等等
    
*   多条命令可以合并为一个：如 sadd myset v1, sadd myset v2, sadd myset v3 可以合并为 sadd myset v1 v2 v3。不过为了防止单条命令过大造成客户端缓冲区溢出，对于 list、set、hash、zset 类型的 key，并不一定只使用一条命令；而是以某个常量为界将命令拆分为多条。这个常量在 redis.h/REDIS_AOF_REWRITE_ITEMS_PER_CMD 中定义，不可更改，3.0 版本中值是 64。
    

##### 文件重写时机

相关参数：

*   aof_current_size：表示当前 AOF 文件空间
    
*   aof_base_size：表示上一次重写后 AOF 文件空间
    
*   auto-aof-rewrite-min-size: 表示运行 AOF 重写时文件的最小体积，默认为 64MB
    
*   auto-aof-rewrite-percentage: 表示当前 AOF 重写时文件空间（aof_current_size）超过上一次重写后 AOF 文件空间（aof_base_size）的比值多少后会重写。
    

同时满足下面两个条件，则触发 AOF 重写机制：

*   aof_current_size 大于 auto-aof-rewrite-min-size
    
*   当前 AOF 相比上一次 AOF 的增长率:(aof_current_size - aof_base_size)/aof_base_size 大于或等于 auto-aof-rewrite-percentage
    

##### 文件重写流程

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFelQlkulZU7WxWACapfzn0R2Y7uHGiceVia3seaDiakw0iasq5Yzibr2UYMw/640?wx_fmt=jpeg)

流程说明：

1.  执行 AOF 重写请求。
    

如果当前进程正在执行 bgrewriteaof 重写，请求不执行。

如果当前进程正在执行 bgsave 操作，重写命令延迟到 bgsave 完成之后再执行。

2.  父进程执行 fork 创建子进程，开销等同于 bgsave 过程。
    

3.1 主进程 fork 操作完成后，继续响应其它命令。所有修改命令依然写入 AOF 文件缓冲区并根据 appendfsync 策略同步到磁盘，保证原有 AOF 机制正确性。

3.2 由于 fork 操作运用写时复制技术，子进程只能共享 fork 操作时的内存数据由于父进程依然响应命令，Redis 使用 “AOF” 重写缓冲区保存这部分新数据，防止新的 AOF 文件生成期间丢失这部分数据。

4.  子进程依据内存快照，按照命令合并规则写入到新的 AOF 文件。每次批量写入硬盘数据量由配置 aof-rewrite-incremental-fsync 控制，默认为 32MB，防止单次刷盘数据过多造成硬盘阻塞。
    

5.1 新 AOF 文件写入完成后，子进程发送信号给父进程，父进程更新统计信息。

5.2 父进程把 AOF 重写缓冲区的数据写入到新的 AOF 文件。

5.3 使用新的 AOF 文件替换老的 AOF 文件，完成 AOF 重写。

**Redis 为什么考虑使用 AOF 而不是 WAL 呢?** 

很多数据库都是采用的 Write Ahead Log（WAL）写前日志，其特点就是先把修改的数据记录到日志中，再进行写数据的提交，可以方便通过日志进行数据恢复。

但是 Redis 采用的却是 AOF（Append Only File）写后日志，特点就是先执行写命令，把数据写入内存中，再记录日志。

如果先让系统执行命令，只有命令能执行成功，才会被记录到日志中。因此，Redis 使用写后日志这种形式，可以避免出现记录错误命令的情况。

另外还有一个原因就是：AOF 是在命令执行后才记录日志，所以不会阻塞当前的写操作。

### 四、复制 Replication

主从复制过程大体可以分为 3 个阶段：连接建立阶段（即准备阶段）、数据同步阶段、命令传播阶段；下面分别进行介绍。

#### 连接建立阶段

1.  保存主节点信息
    
2.  建立 socket 连接
    
3.  发送 ping 命令
    
4.  身份验证
    
5.  发送从节点端口信息
    

#### 数据同步阶段

执行流程：

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFziczzEuXubH6pNPsx0Vwia4fOFItsc1r5cnRHtok7zT3hPk23XP3NLMw/640?wx_fmt=jpeg)

1.  全量复制（完整重同步）
    
    Redis 通过 psync 命令进行全量复制的过程如下：
    

*   从节点判断无法进行部分复制，向主节点发送全量复制的请求；或从节点发送部分复制的请求，但主节点判断无法进行部分复制；具体判断过程需要在讲述了部分复制原理后再介绍；
    
*   主节点收到全量复制的命令后，执行 bgsave，在后台生成 RDB 文件，并使用一个缓冲区（称为复制缓冲区）记录从现在开始执行的所有写命令；
    
*   主节点的 bgsave 执行完成后，将 RDB 文件发送给从节点；**从节点首先清除自己的旧数据，然后载入接收的** RDB 文件，将数据库状态更新至主节点执行 bgsave 时的数据库状态；
    
*   主节点将前述复制缓冲区中的所有写命令发送给从节点，从节点执行这些写命令，将数据库状态更新至主节点的最新状态；
    
*   如果从节点开启了 AOF，则会触发 bgrewriteaof 的执行，从而保证 AOF 文件更新至主节点的最新状态。
    

3.  部分复制（部分重同步）
    

*   复制偏移量
    

主节点和从节点分别维护一个复制偏移量（offset），代表的是**主节点向从节点传递的字节数**；主节点每次向从节点传播 N 个字节数据时，主节点的 offset 增加 N；从节点每次收到主节点传来的 N 个字节数据时，从节点的 offset 增加 N。

*   复制积压缓冲区
    

复制积压缓冲区是由主节点维护的、固定长度的、先进先出 (FIFO) 队列，默认大小 1MB；当主节点开始有从节点时创建，其作用是备份主节点最近发送给从节点的数据。注意，无论主节点有一个还是多个从节点，都只需要一个复制积压缓冲区。

*   服务器运行 ID(runid)
    

每个 Redis 节点 (无论主从)，在启动时都会自动生成一个随机 ID(每次启动都不一样)，由 40 个随机的十六进制字符组成；runid 用来唯一识别一个 Redis 节点。

#### 命令传播阶段

主 -> 从：PING。

每隔指定的时间，**主节点会向从节点发送** PING 命令，这个 PING 命令的作用，主要是为了让从节点进行超时判断。

从 -> 主：REPLCONF ACK

在命令传播阶段，**从节点会向主节点发送** REPLCONF ACK 命令，频率是每秒 1 次；命令格式为：REPLCONF ACK {offset}，其中 offset 指从节点保存的复制偏移量。

### 五、架构模式

#### 哨兵模式

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFnWHtbsIb7vs6pfdpEaQrarGfWolnIqLWlx2UOEuGoULFWTPmeUY1Vg/640?wx_fmt=jpeg)

##### 哨兵模式工作原理

1.  每个 Sentinel 以每秒一次的频率向它所知的 Master，Slave 以及其他 Sentinel 节点发送一个 `PING` 命令；
    
2.  如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过配置文件 `own-after-milliseconds` 选项所指定的值，则这个实例会被 Sentinel 标记为**主观下线**；
    
3.  如果一个 Master 被标记为主观下线，那么正在监视这个 Master 的所有 Sentinel 要以每秒一次的频率确认 Master 是否真的进入主观下线状态；
    
4.  当有**足够数量的 Sentinel**（大于等于配置文件指定的值）在**指定的时间范围内确认** Master 的确进入了主观下线状态，则 Master 会被标记为**客观下线**；
    
5.  如果 Master 处于 **ODOWN 状态**，则投票自动选出新的主节点。将剩余的从节点指向新的主节点继续进行数据复制；
    
6.  在正常情况下，每个 Sentinel 会以每 10 秒一次的频率向它已知的所有 Master，Slave 发送 `INFO` 命令；当 Master 被 Sentinel 标记为客观下线时，Sentinel 向已下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次；
    
7.  若没有足够数量的 Sentinel 同意 Master 已经下线，Master 的客观下线状态就会被移除。若 Master 重新向 Sentinel 的 PING 命令返回有效回复，Master 的主观下线状态就会被移除。
    

**主要缺陷**：单个节点的写能力，存储能力受到单机的限制，动态扩容困难复杂。

#### 集群模式

为了解决哨兵模式存储受单机的限制，这里引入分片概念。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFaCAHvMg5MEcGbaEFGBErEHOVIFtwAiblnrXto1Eb1C5YmicBDJUVN1icw/640?wx_fmt=jpeg)

##### 分片

Redis Cluster 采用**虚拟哈希槽分区**，所有的键根据哈希函数映射到 0 ~ 16383 整数槽内，计算公式：`HASH_SLOT = CRC16(key) % 16384`。每一个节点负责维护一部分槽以及槽所映射的键值数据。

![](https://mmbiz.qpic.cn/mmbiz_jpg/j3gficicyOvavGcroOiaKHttrg1xAAVX8SFTfMSDdemiaQRicmWicbYfvWETEgZeK6bCvxv8Vf1qUVVg5Yibu5Nn7XicyQ/640?wx_fmt=jpeg)

Redis Cluster 提供了灵活的节点扩容和缩容方案。在不影响集群对外服务的情况下，可以为集群添加节点进行扩容也可以下线部分节点进行缩容。可以说，槽是 Redis Cluster 管理数据的基本单位，集群伸缩就是槽和数据在节点之间的移动。

**参考文献**

*   [https://redis.io/topics/replication](https://redis.io/topics/replication)
    
*   [https://redis.io/topics/persistence](https://redis.io/topics/persistence)
    
*   [https://www.cnblogs.com/mrhelloworld/p/redis-architecture.html](https://www.cnblogs.com/mrhelloworld/p/redis-architecture.html)
    
*   [https://mp.weixin.qq.com/s/V2MKyMKtCO1skgWWXBnrgg](https://mp.weixin.qq.com/s?__biz=MzkwOTIxNDQ3OA==&mid=2247533213&idx=1&sn=96abbf1f568b88c2408cbfbd50b41f95&scene=21#wechat_redirect)
    
*   [https://www.cnblogs.com/biyeymyhjob/archive/2012/07/20/2601655.html](https://www.cnblogs.com/biyeymyhjob/archive/2012/07/20/2601655.html)
    
*   [https://www.cnblogs.com/kismetv/p/8654978.html](https://www.cnblogs.com/kismetv/p/8654978.html)
    
*   [https://zhuanlan.zhihu.com/p/187596888](https://zhuanlan.zhihu.com/p/187596888)
    
*   [https://cloud.tencent.com/developer/article/1633077](https://cloud.tencent.com/developer/article/1633077)
    
*   [https://www.cnblogs.com/yangmingxianshen/p/8373205.html](https://www.cnblogs.com/yangmingxianshen/p/8373205.html)
    

近期好文：  

[四种代码洁癖类型，程序员看了直呼内行](http://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649761200&idx=1&sn=d66bc549ab8267425e4c918600fed74b&chksm=beccb6cb89bb3fdd1ebc0b326998e9c407dcd4e0a07e42b39a5d87808dc874acd8c76eae279b&scene=21#wechat_redirect)  

[一文深入理解 Kubernetes](http://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649760964&idx=1&sn=8416dc47639381bfba2391d3d1a25022&chksm=beccb7bf89bb3ea9f7145b73bce49353de1df64a9c21579fbb4e2c8d582bf176a00328c92da9&scene=21#wechat_redirect)  

[从 0 到 1 实现浏览器端沙盒运行环境](http://mp.weixin.qq.com/s?__biz=MjM5ODYwMjI2MA==&mid=2649760697&idx=1&sn=94738a1bb83487f16a5e256aa3d0d212&chksm=beccb0c289bb39d4bb4ce3f1638f09478beb459beeea4e2156b52f4a76334ac36c55bdc75bfc&scene=21#wechat_redirect)