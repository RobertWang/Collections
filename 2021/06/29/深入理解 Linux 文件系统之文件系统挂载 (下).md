> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAwMDQ1MjAzOQ==&mid=2247484079&idx=2&sn=486459d87468309cbae46d9e9c714082&chksm=9ae9f607ad9e7f1197cc0921e6702484aacc951ecabf033d9aa2ab0887e68ead0238bb0e1bbc&cur_album_id=1892049888444514305&scene=189#rd)

本文为文件系统挂载专题文章的第二篇，主要介绍如何通过挂载实例关联挂载点和超级块并添加到全局文件系统树。
===================================================

4. 添加到全局文件系统树
=============

4.1 do_new_mount_fc
-------------------

```
do_new_mount  //fs/namespace.c
->do_new_mount_fc
 ->     struct vfsmount *mnt;                      
    ->   struct mountpoint *mp;                     
 ->  struct super_block *sb = fc->root->d_sb;  //获得vfs的超级块 （之前已经构建好）
 >  mnt = vfs_create_mount(fc);   //为一个已配置的超级块 分配mount实例
    ->    mp = lock_mount(mountpoint);  //寻找挂载点 如果挂载目录是挂载点（已经有文件系统挂载其上），则将最后一次挂载的文件系统根目录作为挂载点    
 ->  do_add_mount(real_mount(mnt), mp, mountpoint, mnt_flags);  //关联挂载点 加入全局文件系统树
 


```

4.2 vfs_create_mount 源码分析
-------------------------

```
vfs_create_mount
    ->     mnt = alloc_vfsmnt(fc->source ?: "none");  //分配mount实例 
    ->     mnt->mnt.mnt_sb         = fc->root->d_sb;  //mount关联超级块 (使用vfsmount关联)
    ->    mnt->mnt.mnt_root       = dget(fc->root);   //mount关联根dentry (使用vfsmount关联)
    ->    mnt->mnt_mountpoint     = mnt->mnt.mnt_root; // mount关联挂载点 （临时指向根dentry，后面会指向真正的挂载点，以至于对用户可见）
    ->  mnt->mnt_parent         = mnt;  //父挂载指向自己 (临时指向 后面会设置)              
    ->     return &mnt->mnt;  //返回内嵌的vfsmount



```

注：老内核使用的是 vfsmount 来描述文件系统的一次挂载，现在内核都使用 mount 来描述，而 vfsmount 被内嵌到 mount 中，主要来描述文件系统的超级块和跟 dentry。

vfs_create_mount 之后 vfs 对象数据结构之间关系图如下：
--------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_jpg/ljlJkD7iaUC2XJwxqpPAaEDks0lvnVlMvDYjDt4x5ib3zQYro0cEUFqbMsnQl5w0ya18xkAdMf6WRVdTFwkkicNcw/640?wx_fmt=jpeg)

4.3 lock_mount 源码分析
-------------------

lock_mount 是最不好理解的函数，下面详细讲解：

-> mp = lock_mount(mountpoint); 

// 不只是加锁， 通过传来的 挂载点的 path（vfsmout, dentry 二元组)，来查找最后一次挂载的文件系统的根 dentry 作为即将挂载文件系统的挂载点

我们看下这个函数

-> 这个函数主要从挂载点的 path(即是挂载目录的 path 结构，如挂载到 / mnt 下, path 为 mnt 的 path) 来找到真正的挂载点 两种情况：

1. 如果挂载点的 path 是正常的目录，原来不是挂载点，则直接返回这个目录的 dentry 作为挂载点（mountpoint 的 m_dentry 会指向挂载点的 dentry）

2. 如果挂载点的 path 不是正常的目录，原来就是挂载点，说明这个目录已经有其他的文件系统挂载，那么它会查找最后一个挂载到这个目录的文件系统的根 dentry, 作为真正的挂载点。

我们打开这个黑匣子看一下：首先传递来的 path 是一个表示要解析的挂载目录 [vfsmount,dentry] 二元组，如我们要挂载到 /mnt  （path 即为 < mnt 所在文件系统的 vfsmount, mnt 的 dentry>）

```
//include/linux/path.h  描述一个路径
struct path { 
        struct vfsmount *mnt;
        struct dentry *dentry;
} __randomize_layout;

 //fs/mount.h       描述一个挂载点
struct mountpoint {       
        struct hlist_node m_hash; 
        struct dentry *m_dentry;  
        struct hlist_head m_list; 
        int m_count;              
};                                


static struct mountpoint *lock_mount(struct path *path)           
{                                                                 
        struct vfsmount *mnt;                                     
        struct dentry *dentry = path->dentry; //获得挂载目录的dentry                     
retry:                                                            
        inode_lock(dentry->d_inode);     //写方式申请 inode的读写信号量                        
        if (unlikely(cant_mount(dentry))) { //判断挂载目录能否被挂载
                inode_unlock(dentry->d_inode);                    
                return ERR_PTR(-ENOENT);                          
        }                                                         
        namespace_lock();  //写方式申请 命名空间读写信号量                                      
        mnt = lookup_mnt(path); //查找挂载在path上的第一个子mount    //！！！重点函数，后面分析 ！！！                               
        if (likely(!mnt)) { // mnt为空 说明没有文件系统挂载在这个path上  是我们要找的目标 
    
    //1.如果dentry之前是挂载点 则从mountpoint hash表 查找mountpoint （dentry计算hash）
    // 2. 如果dentry之前不是挂载点 分配mountpoint 加入mountpoint hash表（dentry计算hash）,设置dentry为挂载点
                struct mountpoint *mp = get_mountpoint(dentry); //！！！重点函数,后面会分析 ！！！
      
                if (IS_ERR(mp)) {                                 
                        namespace_unlock();                       
                        inode_unlock(dentry->d_inode);            
                        return mp;                                
                }                                                 
                return mp; //返回找到的挂载点实例 （这个挂载点的dentry之前没有被挂载）                                      
        }
        namespace_unlock(); //释放命名空间读写信号量
        inode_unlock(path->dentry->d_inode); //释放 inode的读写信号量 
        path_put(path);
        path->mnt = mnt; // path->mnt指向找到的vfsmount                                          
        dentry = path->dentry = dget(mnt->mnt_root); //path->dentry指向找到的vfsmount的根dentry！
        goto retry; //继续查找下一个挂载
}


```

### 1）get_mountpoint 源码分析

```
static struct mountpoint *get_mountpoint(struct dentry *dentry)                      
{                                                                                    
        struct mountpoint *mp, *new = NULL;                                          
        int ret;                                                                     
                                                                                     
        if (d_mountpoint(dentry)) {  //dentry为挂载点 （当dentry为挂载点时 会设置dentry->d_flags 的DCACHE_MOUNTED标志）                                                
                /* might be worth a WARN_ON() */                                     
                if (d_unlinked(dentry))                                              
                        return ERR_PTR(-ENOENT);                                     
mountpoint:                                                                          
                read_seqlock_excl(&mount_lock);                                      
                mp = lookup_mountpoint(dentry); // 从mountpoint hash表 查找mountpoint （dentry计算hash）                                    
                read_sequnlock_excl(&mount_lock);                                    
                if (mp)                                                              
                        goto done;  //找到直接返回mountpoint实例                                               
        }                                                                            
                                                                                     
        if (!new)    //mountpoint哈希表中没有找到    则分配                                                             
                new = kmalloc(sizeof(struct mountpoint), GFP_KERNEL);             
        if (!new)                                                                    
                return ERR_PTR(-ENOMEM);                                             
                                                                                     
                                                                                     
        /* Exactly one processes may set d_mounted */                                
        ret = d_set_mounted(dentry);   //设置dentry为挂载点
   ->dentry->d_flags |= DCACHE_MOUNTED;  //设置挂载点标志很重要  路径名查找时发现为挂载点则会步进到相关文件系统的跟dentry
                                                                                     
        /* Someone else set d_mounted? */                                            
        if (ret == -EBUSY)                                                           
                goto mountpoint;                                                     
                                                                                     
        /* The dentry is not available as a mountpoint? */                           
        mp = ERR_PTR(ret);                                                           
        if (ret)                                                                     
                goto done;                                                           
                                                              
        /* Add the new mountpoint to the hash table */        
        read_seqlock_excl(&mount_lock);                       
        new->m_dentry = dget(dentry);  //设置mountpoint实例  的m_dentry 指向dentry                       
        new->m_count = 1;                                      
        hlist_add_head(&new->m_hash, mp_hash(dentry));   // mountpoint实例添加到 mountpoint_hashtable     
        INIT_HLIST_HEAD(&new->m_list);  //初始化 挂载链表   mount实例会加入到这个链表                      
        read_sequnlock_excl(&mount_lock);                     
                                                              
        mp = new;   //指向挂载点                                          
        new = NULL;                                           
done:                                                         
        kfree(new);                                           
        return mp;  //返回挂载点                                           
}                                                             


```

### 2）lookup_mnt 源码分析

它在文件系统挂载和路径名查找都会使用到，作用为查找挂载在这个 path 下的第一个子 vfsmount 实例。

-> 文件系统挂载场景中，使用它查找合适的 vfsmount 实例作为父 vfsmount。        

-> 路径名查找场景中，使用它查找一个合适的 vfsmount 实例作为下一级路径名解析起点的 vfsmount。

```
//fs/namespace.c
lookup_mnt(const struct path *path)
->  struct mount *child_mnt;
 struct vfsmount *m;     
 child_mnt = __lookup_mnt(path->mnt, path->dentry);  //委托__lookup_mnt
 m = child_mnt ? &child_mnt->mnt : NULL; //返回mount实例的vfsmount实例 或NULL            
 return m;

->
struct mount *__lookup_mnt(struct vfsmount *mnt, struct dentry *dentry)        
{               
        struct hlist_head *head = m_hash(mnt, dentry);  // 根据 父vfsmount实例 和 挂载点的dentry查找  mount_hashtable的一个哈希表项
        struct mount *p;
        
        hlist_for_each_entry_rcu(p, head, mnt_hash)  //从哈希表项对应的链表中查找   遍历链表的每个节点
                if (&p->mnt_parent->mnt == mnt && p->mnt_mountpoint == dentry) //节点的mount实例的父mount为mnt 且mount实例的挂载点为 dentry
                        return p; //找到返回mount实例
        return NULL;  //没找到返回NULL
}



```

### 3）lock_mount 情景分析

1)lock_mount 传递的 path 之前不是挂载点：

调用链为：

lock_mount 

  ->mnt = lookup_mnt(path) // 没有子 mount 返回 NULL

  ->mp = get_mountpoint(dentry) // 分配 mountpoint 加入 mountpoint hash 表（dentry 计算 hash）, 设置 dentry 为挂载点

  ->return mp  // 返回找到的挂载点实例

2)lock_mount 传递的 path 之前是挂载点：

我们现在执行  mount -t ext2 /dev/sda4 /mnt

之前 /mnt 的挂载情况

mount /dev/sda1 /mnt   （1） 

mount /dev/sda2 /mnt   （2） 

mount /dev/sda3 /mnt （3）

调用链为：

lock_mount 

 ->mnt = lookup_mnt(path) // 返回（1）的 mount 实例

  ->path->mnt = mnt  // 下一次查找的 path->mnt 赋值（1）的 mount 实例

  ->dentry = path->dentry = dget(mnt->mnt_root) // // 下一次查找 path->dentry 赋值（1）的根 dentry

  ->mnt = lookup_mnt(path) // 返回（2）的 mount 实例 

  ->path->mnt = mnt  // 下一次查找的  path->mnt 赋值（2）的 mount 实例

  ->dentry = path->dentry = dget(mnt->mnt_root) // // 下一次查找 path->dentry 赋值（2）的根 dentry

  ->mnt = lookup_mnt(path) // 返回（3）的 mount 实例

  ->path->mnt = mnt  // 下一次查找的  path->mnt 赋值（3）的 mount 实例

  ->dentry = path->dentry = dget(mnt->mnt_root) // // 下一次查找 path->dentry 赋值（3）的根 dentry

-> mnt = lookup_mnt(path) // 没有子 mount 返回 NULL

->mp = get_mountpoint(dentry) // 分配 mountpoint 加入 mountpoint hash 表（dentry 计算 hash）, 设置 dentry 为挂载点（（3）的根 dentry 作为挂载点）

->return mp  // 返回找到的挂载点实例 (也就是最后一次挂载（3） 文件系统的根 dentry)

4.4 do_add_mount 源码分析
---------------------

准备好了挂载点之后，接下来子 mount 实例关联挂载点以及添加子 mount 实例到全局的文件系统挂载树中。

```
do_add_mount  //添加mount到全局的文件系统挂载树中
->struct mount *parent = real_mount(path->mnt); //获得父挂载点的挂载实例
->graft_tree(newmnt, parent, mp)
 -> mnt_set_mountpoint(dest_mnt, dest_mp, source_mnt)
  ->   child_mnt->mnt_mountpoint = mp->m_dentry;   //关联子mount到挂载点的dentry                 
    child_mnt->mnt_parent = mnt; //子mount->mnt_parent指向父mount
    child_mnt->mnt_mp = mp; //子mount->mnt_mp指向挂载点
    hlist_add_head(&child_mnt->mnt_mp_list, &mp->m_list); //mount添加到挂载点链表

 ->commit_tree //提交挂载树
  ->__attach_mnt(mnt, parent)
   ->  hlist_add_head_rcu(&mnt->mnt_hash,
       ¦  m_hash(&parent->mnt, mnt->mnt_mountpoint));  //添加到mount hash表 ，通过父挂载点的vfsmount和挂载点的dentry作为索引(如上面示例中的<(3)的vfsmount , (3)的根dentry>)
    list_add_tail(&mnt->mnt_child, &parent->mnt_mounts);  //添加到父mount链表


```

上面说了一大堆，主要为了实现：

**将 mount 实例与挂载点联系起来（会将 mount 实例加入到 mount 哈希表，父文件系统的 vfsmount 和真正的挂载点的 dentry 组成的二元组为索引，路径名查找时便于查找），以及 mount 实例与文件系统的跟 dentry 联系起来（路径名查找的时候便于沿着跟 dentry 来访问这个文件系统的所有文件）。**

do_add_mount  之后 vfs 对象数据结构之间关系图（/mnt 之前不是挂载点情况）如下：
---------------------------------------------------

![](https://mmbiz.qpic.cn/mmbiz_jpg/ljlJkD7iaUC2XJwxqpPAaEDks0lvnVlMvj2zuTuKgicMODeodXVPQyC6sricf7eIKrDNKGE8ge3qlTiabEIAcxgJfA/640?wx_fmt=jpeg)

5. mount 的应用
============

上面几章我们分析了文件系统挂载的主要流程，创建并关联了各个 vfs 的对象，为了打开文件等路径名查找时做准备。

5.1 路径名查找到挂载点源码分析
-----------------

```
//fs/namei.c 查找一个路径分量
walk_component  
->step_into
 ->handle_mounts
  ->traverse_mounts
   ->flags = smp_load_acquire(&path->dentry->d_flags); //获得dentry标志
   ->__traverse_mounts(path, flags, jumped, count, lookup_flags); //查找挂载点  返回不再是挂载点的path
     -> {
     
       while (flags & DCACHE_MANAGED_DENTRY) {    //找到的dentry是挂载点 则继续查找 ，不是则退出循环
        ...
        if (flags & DCACHE_MOUNTED) {   // something's mounted on it..  是挂载点 和上面lock_mount分析逻辑类似 
         //不断的查找挂载点  直到最后查找到的dentry不是挂载点 退出循环
         struct vfsmount *mounted = lookup_mnt(path);  //查找第一个子mount          
         if (mounted) {          // ... in our namespace         
           dput(path->dentry);                            
           if (need_mntput)
             mntput(path->mnt);
           path->mnt = mounted; //path->mnt赋值子mount
           path->dentry = dget(mounted->mnt_root); //path->dentry 赋值子mount的根dentry （是挂载点就会步进到挂载点的跟dentry）
           // here we know it's positive
           flags = path->dentry->d_flags; //获得dentry标志
           need_mntput = true;
           continue; //继续查找
        }
        ...

       }
       
       //最终返回找到的path（它不再是挂载点），后面继续查找下一个路径
      }



```

5.2 举例说明
--------

我们做以下的路径名查找：/mnt/test/text.txt

/mnt/ 目录挂载情况为

mount /dev/sda1 /mnt   （1）

mount /dev/sda2 /mnt  （2）

mount /dev/sda3 /mnt  （3）

test/text.txt 文件在 /dev/sda3 上  

则路径名查找时，查找到 mnt 的 dentry 发现它是挂载点，就会依次查找（1）的根目录 ->（2）的根目录 ->（3）的根目录， 最终将（3）的 vfsmount 和 根目录的 dentry 填写到 path，进行下一步的查找, 最终查找到 /dev/sda3 上的 text.txt 文件。

注：一个目录被文件系统挂载时，原来目录中包含的其他子目录或文件会被隐藏。  

6. 挂载图解
=======

为了便于讲解图示中各个实例表示如下：

Xyn  --->  X 表示哪个实例对象   如：mount 实例使用 M 表示（第一个大小字母） dentry 使用 D 表示  inode 使用 I 表示  super_block 用 S 表示 vfsmount 使用 V 表示

 y 表示是父文件系统中的实例对象还是子文件系统中  如：p（parent）表示父文件系统中实例对象  c（child）表示子文件系统中实例对象

 n 区分同一种对象的不同实例

例如：Dc1 表示子文件系统中一个 dentry 对象

### 1）mount、super_block、file_system_type 三者关系图解

![](https://mmbiz.qpic.cn/mmbiz_jpg/ljlJkD7iaUC2XJwxqpPAaEDks0lvnVlMvHJBXVsIZyRWpCpzjHOt4D7CPbicIXdIjjmHGqN3FRU2j8eHRJDxBOug/640?wx_fmt=jpeg)

  

解释：mount 实例、super_block 实例、file_system_type 实例三种层级逐渐升高，即一个 file_system_type 实例会包含多个 super_block 实例，一个 super_block 实例会包含多个 mount 实例。一种 file_system_type 必须先被注册到系统中来宣誓这种文件系统存在，主要提供此类文件系统的挂载和卸载方法等，注册即是加入全局的 file_systems 链表，等到有块设备上的文件系统要挂载时就会根据挂载时传递的文件系统类型名查找 file_system_type 实例，如果查找到，就会调用它的挂载方法进行挂载。首先，在 file_systems 实例的 super_block 链表中查找有没有 super_block 实例已经被创建，如果有就不需要从磁盘读取（这就是一个块设备上的文件系统挂载到多个目录上只有一个 super_block 实例的原因），如果没有从磁盘读取并加入对应的 file_systems 实例的 super_block 链表。而每次挂载都会创建一个 mount 实例来联系挂载点和 super_block 实例，并以（父 vfsmount, 挂载点 dentry）为索引加入到全局 mount 哈希表，便于后面访问这个挂载点的文件系统时的路径名查找。

### 2）父子文件系统挂载关系图解

![](https://mmbiz.qpic.cn/mmbiz_jpg/ljlJkD7iaUC2XJwxqpPAaEDks0lvnVlMvXLib8LkjWe0ZlZKtO02OMDEzibRpvmibafXktwDFMpC2lqHTCb2gmjS8Q/640?wx_fmt=jpeg)

解释：图中 / dev/sda1 中的子文件系统挂载到父文件系统的 / mnt 目录下。当挂载的时候会创建 mount、super_block、跟 inode、跟 dentry 四大数据结构并建立相互关系，将子文件系统的 mount 加入到 (Vp, Dp3) 二元组为索引的 mount 哈希表中，通过设置 mnt 的目录项（Dp3）的 DCACHE_MOUNTED 来将其标记为挂载点，并与父文件系统建立亲缘关系挂载就完成了。

当需要访问子文件系统中的某个文件时，就会通过路径名各个分量解析到 mnt 目录，发现其为挂载点，就会通过 (Vp, Dp3) 二元组在 mount 哈希表中找到子文件系统的 mount 实例(Mc), 然后就会从子文件系统的跟 dentry（Dc1）开始往下继续查找，最终访问到子文件系统上的文件。

### 2）单个文件系统多挂载点关系图解

![](https://mmbiz.qpic.cn/mmbiz_jpg/ljlJkD7iaUC2XJwxqpPAaEDks0lvnVlMvHKnARRl2nZ3YNdgmhQNWSwl4txMgNYsN8AqWMbyUFGUnQ46zpIhmBQ/640?wx_fmt=jpeg)

解释：图中将 / dev/sda1 中的文件系统分别挂载到父文件系统的 / mnt/a 和 / mnt/b 目录下。当第一次挂载到 / mnt/a 时，会创建 mount、super_block、跟 inode、跟 dentry 四大数据结构 (分别对应与 Mc1、Sc、Dc1、Ic) 并建立相互关系，将子文件系统的 Mc1 加入到 (Vp, Dp3) 二元组为索引的 mount 哈希表中，通过设置 / mnt/a 的目录项的 DCACHE_MOUNTED 来将其标记为挂载点，并与父文件系统建立亲缘关系挂载就完成了。然后挂载到 / mnt/b 时, Sc、Dc1、Ic 已经创建好不需要再创建，内存中只会有一份，会创建 Mc2 来关联 super_block 和第二次的挂载点，建立这几个数据结构关系，将子文件系统的 Mc2 加入到 (Vp, Dp4) 二元组为索引的 mount 哈希表中，通过设置 / mnt/b 的目录项的 DCACHE_MOUNTED 来将其标记为挂载点，并与父文件系统建立亲缘关系挂载就完成了。

当需要访问子文件系统中的某个文件时，就会通过路径名各个分量解析到 / mnt/a 目录，发现其为挂载点，就会通过 (Vp, Dp3) 在 mount 哈希表中找到子文件系统的 Mc1, 然后就会从子文件系统的 Dc1 开始往下继续查找，最终访问到子文件系统上的文件。同样，如果解析到 / mnt/b 目录, 发现其为挂载点，就会通过 (Vp, Dp4) 在 mount 哈希表中找到子文件系统的 Mc2, 然后就会从子文件系统的 Dc1 开始往下继续查找，最终访问到子文件系统上的文件。可以发现，同一个块设备上的文件系统挂载到不同的目录上，相关联的 super_block 和跟 dentry 是一样的，这保证了无论从哪个挂载点开始路径名查找都访问到的是同一个文件系统上的文件。

### 3）多文件系统单挂载点关系图解

![](https://mmbiz.qpic.cn/mmbiz_jpg/ljlJkD7iaUC2XJwxqpPAaEDks0lvnVlMvnhK60aicfuGtKXujMe01v1kkkyxFU8r8TpYCbzjgsJYncwgX4aPWf1Q/640?wx_fmt=jpeg)

解释：最后我们来看多文件系统单挂载点的情况，图中先将块设备 / dev/sda1 中的子文件系统 1 挂载到 / mnt 目录，然后再将块设备 / dev/sdb1 中的子文件系统 2 挂载到 / mnt 目录上。

当子文件系统 1 挂载的时候，会创建 mount、super_block、跟 inode、跟 dentry 四大数据结构 (分别对应与 Mc1、Sc1、Dc1、Ic1) 并建立相互关系，将子文件系统的 Mc1 加入到 (Vp, Dp3) 二元组为索引的 mount 哈希表中，通过设置 / mnt 的目录项的 DCACHE_MOUNTED 来将其标记为挂载点，并与父文件系统建立亲缘关系挂载就完成了。

当子文件系统 2 挂载的时候，会创建 mount、super_block、跟 inode、跟 dentry 四大数据结构 (分别对应与 Mc2、Sc2、Dc4、Ic2) 并建立相互关系，这个时候会发现 / mnt 目录是挂载点，则会将子文件系统 1 的根目录（Dc1）作为文件系统 2 的挂载点，将子文件系统的 Mc2 加入到 (Vc1, Dc1) 二元组为索引的 mount 哈希表中，通过设置 Dc1 的 DCACHE_MOUNTED 来将其标记为挂载点，并与父文件系统建立亲缘关系挂载就完成了。

这个时候，子文件系统 1 已经被子文件系统 2 隐藏起来了，当路径名查找到 / mnt 目录时，发现其为挂载点，则通过 (Vp, Dp3) 二元组为索引在 mount 哈希表中找到 Mc1，会转向文件系统 1 的跟目录（Dc1）开始往下继续查找，发现 Dc1 也是挂载点，则 (通过 Vc1, Dc1) 二元组为索引在 mount 哈希表中找到 Mc2, 会转向文件系统 1 的跟目录（Dc4）开始往下继续查找，于是就访问到了文件系统 2 中的文件。除非，文件系统 2 被卸载，文件系统 1 的跟 dentry（Dc1）不再是挂载点，这个时候文件系统 1 中的文件才能再次被访问到。

7. 总结
=====

Linux 中，块设备上的文件系统只有挂载到内存的目录树中的一个目录下，用户进程才能访问，而挂载是创建数据结构关联块设备上的文件系统和挂载点，使得路径名查找的时候能够通过挂载点目录访问到挂载在其下的文件系统。

7.1 挂载主要步骤
----------

1.vfs_get_tree 调用具体文件系统的获取填充超级块方法 (fs_context_operations.get_tree 或者 file_system_type.mount)， 在内存构建 super_block，然后构建根 inode 和根 dentry(磁盘文件系统可能需要从磁盘读取磁盘超级块构建内存的 super_block，从磁盘读取根 inode 构建内存的 inode)。2.do_new_mount_fc 对于每次挂载都会分配 mount 实例，用于关联挂载点到文件系统。当一个要挂载的目录不是挂载点，会设置这个目录的 dentry 为挂载点，然后 mount 实例记录这个挂载点。当一个要挂载的目录是挂载点（之前已经有文件系统被挂载到这个目录），那么新挂载的文件系统将挂载到这个目录最后一次挂载的文件系统的根 dentry, 之前挂载的文件系统的文件都被隐藏（当子挂载被卸载，原来的文件系统的文件才可见）。

7.2 文件系统的用户可见性
--------------

**只对内核内部可见：**不需要将文件系统关联到一个挂载点，内核通过文件系统的 super_block 等结构即可访问到文件系统的文件（如 bdev，sockfs）。

**对于用户可见：**需要将文件系统关联到一个挂载点，就需要通过给定的挂载点目录名找到真正的挂载点，然后进行挂载操作， 挂载的实质是：通过 mount 实例的 mnt_mountpoint 关联真正的挂载点 dentry, 然后建立父 mount 关系，mount 实例加入到全局的 mount hash table（通过父 vfsmount 和真正的挂载点 dentry 作为 hash 索引），然后用户打开文件的时候通过路径名查找解析各个目录分量，当发现一个目录是挂载点时，就会步进到最后一次挂载到这个目录的文件系统的根 dentry 中继续查找，知道根 dentry 就可以继续查找到这个文件系统的任何文件。

7.3 几条重要规律
----------

1）文件系统被挂载后都会有以下几大 vfs 对象被创建：

super_block  
mount

根 inode 

根 dentry

注：其中 mount 为纯软件构造的对象（内嵌 vfsmount 对象），其他对象视文件系统类型，可能涉及到磁盘操作。

super_block    超级块实例，描述一个文件系统的信息，有的需要磁盘读取在内存中填充来构建（如磁盘文件系统），有的直接内存中填充来构建。

mount     挂载实例，描述一个文件系统的一次挂载，主要关联一个文件系统到挂载点，为路径名查找做重要准备工作。

根 inode    每个文件系统都会有根 inode，有的需要磁盘读取在内存中填充来构建（如磁盘文件系统，根 inode 号已知），有的直接内存中填充来构建。

根 dentry   每个文件系统都会有根 dentry，根据根 inode 来构建，路径名查找时会步进到文件系统的根 dentry 来访问这个文件系统的文件。

2）一个目录可以被多个文件系统挂载。第一次挂载是直接挂载这个目录上，新挂载的文件系统实际上是挂载在上一个文件系统的根 dentry 上。

3）一个目录被多个文件系统挂载时，新挂载导致之前的挂载被隐藏。

4）一个目录被文件系统挂载时，原来目录中包含的其他子目录或文件被隐藏。

5）每次挂载都会有一个 mount 实例描述本次挂载。

6）一个快设备上的文件系统可以被挂载到多个目录，有多个 mount 实例，但是只会有一个 super_block、根 dentry 和根 inode。

7）mount 实例用于关联挂载点 dentry 和文件系统，起到路径名查找时 “路由” 的作用。

8）挂载一个文件系统必须保证所要挂载的文件系统类型已经被注册。

9）挂载时会查询文件系统类型的 fs_type->fs_supers 链表，检查是否已经有 super_block 被加入链表，如果没有才会分配并读磁盘超级块填充。

10）对象层次：一个 fs_type->fs_supers 链表可以挂接属于同一个文件系统的被挂载的超级块，超级块链表可以挂接属于同一个超级块的 mount 实例 fs_type ->  super_block -> mount 从高到低的包含层次。  

参考文档: 《存储技术原理分析  基于 Linux2.6 内核源代码》