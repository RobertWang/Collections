> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JOGTxJhsVca3FoC-639tlg)

> RDB 通过周期性保存数据集快照, 实现 Redis 的数据持久化存储和主从快速副本同步。

一、Redis RDB 使用场景
================

RDB 主要用于两方面:

1.  数据持久化。将内存中的数据集快照保存到磁盘中, 实现断电重启后数据恢复。
    
2.  主从复制。主节点生成 RDB 文件发送给从节点, 用于初次全量复制或连接重建时快速同步。
    

这两种场景下, RDB 都可以提供比 AOF 方式更好的恢复速度。

二、RDB 配置启动
==========

### RDB 的相关配置

1.  save: 设置触发 RDB 条件, 比如默认 900 秒内至少 1 个 key 变更。
    
2.  dbfilename: 设置 RDB 文件名称, 默认 dump.rdb。
    
3.  dir: 设置 RDB 文件保存目录。
    
4.  stop-writes-on-bgsave-error: 后台保存错误时是否停止写入。
    

通过配置保存条件, 可以控制 RDB 持久化的频率。

### save 默认配置

如果配置文件没有配置 save, 服务启动时相当于配置以下保存策略

```
# 距离上次生产rdb 1小时以上并且至少发生1次写操作, 开启bg save生产rdb
save 3600 1   # After 3600 seconds (an hour) if at least 1 key changed
# 距离上次生产rdb 5分钟以上并且至少发生300次写操作, 开启bg save生产rdb
save 300 100  # * After 300 seconds (5 minutes) if at least 100 keys changed
# 距离上次生产rdb 1分钟以上并且至少发生10000次写操作, 开启bg save生产rdb
save 60 10000  # After 60 seconds if at least 10000 keys changed

```

三、RDB 工作原理
==========

1.  保存时, Redis 创建 RDB 文件的子进程。
    
2.  RDB 子进程遍历内存键值对, 编码后写入临时文件。
    
3.  替换旧 RDB 文件, 更新生成时间戳。
    
4.  主节点生成 RDB 传给从节点, 从节点接收加载到内存中。
    

RDB 通过创建子进程实现保存, 最大程度减少性能影响。

四、RDB 源码剖析
==========

redis 源码版本：6.2.5 本文从以下几个场景带着大家走读代码：

*   启动时 加载 rdb 文件
    
*   定时启动子进程保存 rdb
    
*   save 命令和 bgsave 命令
    
*   diskless 模式 rdb 处理
    

### rdb 相关的变量

在进入具体场景分析前，先来认识下与 rdb 相关的变量

```
// path:src/server.h
struct redisServer {
//...
/* RDB persistence */
    // rdb相关配置项和全局变量
    // 写操作地址dirty数值
    long long dirty;                /* Changes to DB from the last save */
    // 开启后台进程前保存当时ditry值
    // 通过dirty-dirty_before_bgsave可以知道在后台子进程生成rdb的过程中
    // kv 是否有产生了多少新的变化
    long long dirty_before_bgsave;  /* Used to restore dirty on failed BGSAVE */
    // bg save 保存策略
    // saveparam 包含seconds,代表上次rdb save距离当前至少多少时间就可以再次 save rdb
    // saveparam 包含change, 即dirty数值至少大于多少时 就可以再次 save rdb
    // 用来判断是否可以开启新的一次 bg save rdb
    struct saveparam *saveparams;   /* Save points array for RDB */
    // saveparams 数组长度
    int saveparamslen;              /* Number of saving points */
    // 保存rdb 文件名
    char *rdb_filename;             /* Name of RDB file */
    // 是否使用压缩
    int rdb_compression;            /* Use compression in RDB? */
    // 保存生成rdb过程中的checksum
    int rdb_checksum;               /* Use RDB checksum? */
    // 是否同步删除rdb文件(实例没有持久化时)
    int rdb_del_sync_files;         /* Remove RDB files used only for SYNC if
                                       the instance does not use persistence. */
    // 最后一次save rdb时间戳
    time_t lastsave;                /* Unix time of last successful save */
    //// 最后一次 参数 bg save rdb时间戳
    time_t lastbgsave_try;          /* Unix time of last attempted bgsave */
    // 最后一个异步生成rdb 使用的时间
    time_t rdb_save_time_last;      /* Time used by last RDB save run. */
    // 当前生产rdb 的开始时间
    time_t rdb_save_time_start;     /* Current RDB save start time. */
    // 设置为1,代表没有其他子进程可以拉起bg save
    // server_cron 会定时检测
    int rdb_bgsave_scheduled;       /* BGSAVE when possible if true. */
    // rdb 类型
    // #define RDB_CHILD_TYPE_NONE 0
    // #define RDB_CHILD_TYPE_DISK 1     /* RDB is written to disk. */
    // #define RDB_CHILD_TYPE_SOCKET 2   /* RDB is written to slave socket. */
    int rdb_child_type;             /* Type of save by active child. */
    // 记录最后一次bg save 成功或者失败
    int lastbgsave_status;          /* C_OK or C_ERR */
    // 如果被设置, 即不能bgsave 就不能写
    // 即lastbgsave_status 为err 并且stop_writes_on_bgsave_err被设置
    // 无法执行写命令
    int stop_writes_on_bgsave_err;  /* Don't allow writes if can't BGSAVE */
    // 主进程读取子进程发过来的rdb fd
    int rdb_pipe_read;              /* RDB pipe used to transfer the rdb data */
                                    /* to the parent process in diskless repl. */
    // 主进程通知子进程退出的fd(RDB_CHILD_TYPE_SOCKET模式下有用)
    int rdb_child_exit_pipe;        /* Used by the diskless parent allow child exit. */
    // 保存此次RDB_CHILD_TYPE_SOCKET时, 需要把rdb内容发送给那些副本(从)的客户端
    connection **rdb_pipe_conns;    /* Connections which are currently the */、
    // rdb_pipe_conns 数量
    int rdb_pipe_numconns;          /* target of diskless rdb fork child. */
    // 记录多少conns需要等待写事件继续发送rdb
    int rdb_pipe_numconns_writing;  /* Number of rdb conns with pending writes. */
    // 从子进程读取的rdb内存保存在rdb_pipe_buff
    char *rdb_pipe_buff;            /* In diskless replication, this buffer holds data */
    // rdb_pipe_buff的大小
    int rdb_pipe_bufflen;           /* that was read from the the rdb pipe. */
    // 测试用
    int rdb_key_save_delay;         /* Delay in microseconds between keys while
                                     * writing the RDB. (for testings). negative
                                     * value means fractions of microsecons (on average). */
    // 测试用
    int key_load_delay;             /* Delay in microseconds between keys while
                                     * loading aof or rdb. (for testings). negative
                                     * value means fractions of microsecons (on average). */
    // 通过的子进程向主进程发送信息用的管道
    /* Pipe and data structures for child -> parent info sharing. */
    int child_info_pipe[2];         /* Pipe used to write the child_info_data. */
    int child_info_nread;           /* Num of bytes of the last read from pipe */
    //...
}

```

### 启动时加载 rdb 文件

```
// path: src/rdb.c
/* Mark that we are loading in the global state and setup the fields
 * needed to provide loading stats. */
// 开始加载rdb, 设置一些全局状态
void startLoading(size_t size, int rdbflags) {
    /* Load the DB */
    server.loading = 1; // 设置loading标记
    server.loading_start_time = time(NULL); // 开始时间
    server.loading_loaded_bytes = 0; // 已经加载字节数
    server.loading_total_bytes = size; // 总共的字节数
    server.loading_rdb_used_mem = 0; // 使用内容
    blockingOperationStarts();
    /* Fire the loading modules start event. */
    // 通知modlues 事件
    int subevent;
    if (rdbflags & RDBFLAGS_AOF_PREAMBLE)
        subevent = REDISMODULE_SUBEVENT_LOADING_AOF_START;
    else if(rdbflags & RDBFLAGS_REPLICATION)
        subevent = REDISMODULE_SUBEVENT_LOADING_REPL_START;
    else
        subevent = REDISMODULE_SUBEVENT_LOADING_RDB_START;
    moduleFireServerEvent(REDISMODULE_EVENT_LOADING,subevent,NULL);
}
/* Mark that we are loading in the global state and setup the fields
 * needed to provide loading stats.
 * 'filename' is optional and used for rdb-check on error */
// 开始从文件加载
void startLoadingFile(FILE *fp, char* filename, int rdbflags) {
    struct stat sb;
    // 判断文件是否存在
    if (fstat(fileno(fp), &sb) == -1)
        sb.st_size = 0;
    // 记录加载文件名
    rdbFileBeingLoaded = filename;
    // 调用startLoading 设置一些全局状态
    startLoading(sb.st_size, rdbflags);
}
/* Refresh the loading progress info */
// 刷新加载的进度 和内存使用量
void loadingProgress(off_t pos) {
    server.loading_loaded_bytes = pos;
    if (server.stat_peak_memory < zmalloc_used_memory())
        server.stat_peak_memory = zmalloc_used_memory();
}
/* Loading finished */
// 停止加载，设置一些全局变量
void stopLoading(int success) {
    server.loading = 0;
    blockingOperationEnds();
    rdbFileBeingLoaded = NULL;
    /* Fire the loading modules end event. */
    moduleFireServerEvent(REDISMODULE_EVENT_LOADING,
                          success?
                            REDISMODULE_SUBEVENT_LOADING_ENDED:
                            REDISMODULE_SUBEVENT_LOADING_FAILED,
                          NULL);
}
/* Load a "type" in RDB format, that is a one byte unsigned integer.
 * This function is not only used to load object types, but also special
 * "types" like the end-of-file type, the EXPIRE type, and so forth. */
// 读取1字节内容作为type
int rdbLoadType(rio *rdb) {
    unsigned char type;
    if (rioRead(rdb,&type,1) == 0) return -1;
    return type;
}
/* Load a string object from an RDB file according to flags:
 *
 * RDB_LOAD_NONE (no flags): load an RDB object, unencoded.
 * RDB_LOAD_ENC: If the returned type is a Redis object, try to
 *               encode it in a special way to be more memory
 *               efficient. When this flag is passed the function
 *               no longer guarantees that obj->ptr is an SDS string.
 * RDB_LOAD_PLAIN: Return a plain string allocated with zmalloc()
 *                 instead of a Redis object with an sds in it.
 * RDB_LOAD_SDS: Return an SDS string instead of a Redis object.
 *
 * On I/O error NULL is returned.
 */
// 从rdb 加载string
void *rdbGenericLoadStringObject(rio *rdb, int flags, size_t *lenptr) {
    int encode = flags & RDB_LOAD_ENC;
    int plain = flags & RDB_LOAD_PLAIN;
    int sds = flags & RDB_LOAD_SDS;
    int isencoded;
    unsigned long long len;
    // 先读长度, isencoded代表是RDB_ENCVAL编码
    len = rdbLoadLen(rdb,&isencoded);
    if (len == RDB_LENERR) return NULL;
    // 如果是RDB_ENCVAL, 具体读取类型
    if (isencoded) {
        switch(len) {
        case RDB_ENC_INT8:
        case RDB_ENC_INT16:
        case RDB_ENC_INT32:
            // string压缩为int,需要反解为string
            return rdbLoadIntegerObject(rdb,len,flags,lenptr);
        case RDB_ENC_LZF:
            // 压缩为Lzf的字符串,解压缩为正常字符串
            return rdbLoadLzfStringObject(rdb,flags,lenptr);
        default:
            rdbReportCorruptRDB("Unknown RDB string encoding type %llu",len);
            return NULL;
        }
    }
    // 没有压缩,直接读取原始字节返回
    // 根据flag 返回不同的类型
    if (plain || sds) {
        void *buf = plain ? ztrymalloc(len) : sdstrynewlen(SDS_NOINIT,len);
        if (!buf) {
            serverLog(server.loading? LL_WARNING: LL_VERBOSE, "rdbGenericLoadStringObject failed allocating %llu bytes", len);
            return NULL;
        }
        if (lenptr) *lenptr = len;
        if (len && rioRead(rdb,buf,len) == 0) {
            if (plain)
                zfree(buf);
            else
                sdsfree(buf);
            return NULL;
        }
        return buf;
    } else {
        robj *o = encode ? createStringObject(SDS_NOINIT,len) :
                           createRawStringObject(SDS_NOINIT,len);
        if (len && rioRead(rdb,o->ptr,len) == 0) {
            decrRefCount(o);
            return NULL;
        }
        return o;
    }
}
// 从rdb读取string
// 要求返回obj-raw
robj *rdbLoadStringObject(rio *rdb) {
    return rdbGenericLoadStringObject(rdb,RDB_LOAD_NONE,NULL);
}
// 从rdb读取string
// 要求返回obj(带压缩)
robj *rdbLoadEncodedStringObject(rio *rdb) {
    return rdbGenericLoadStringObject(rdb,RDB_LOAD_ENC,NULL);
}
/* Load a Redis object of the specified type from the specified file.
 * On success a newly allocated object is returned, otherwise NULL. */
// 加载rdb object
robj *rdbLoadObject(int rdbtype, rio *rdb, sds key) {
    robj *o = NULL, *ele, *dec;
    uint64_t len;
    unsigned int i;
    int deep_integrity_validation = server.sanitize_dump_payload == SANITIZE_DUMP_YES;
    if (server.sanitize_dump_payload == SANITIZE_DUMP_CLIENTS) {
        /* Skip sanitization when loading (an RDB), or getting a RESTORE command
         * from either the master or a client using an ACL user with the skip-sanitize-payload flag. */
        int skip = server.loading ||
            (server.current_client && (server.current_client->flags & CLIENT_MASTER));
        if (!skip && server.current_client && server.current_client->user)
            skip = !!(server.current_client->user->flags & USER_FLAG_SANITIZE_PAYLOAD_SKIP);
        deep_integrity_validation = !skip;
    }
    // 判断RDB_Type
    // 根据不同type进行价值
    if (rdbtype == RDB_TYPE_STRING) {
        /* Read string value */
        if ((o = rdbLoadEncodedStringObject(rdb)) == NULL) return NULL;
        o = tryObjectEncoding(o);
    } else if (rdbtype == RDB_TYPE_LIST) {
        /* Read list value */
        // list, 先读取数量
        if ((len = rdbLoadLen(rdb,NULL)) == RDB_LENERR) return NULL;
        o = createQuicklistObject();
        quicklistSetOptions(o->ptr, server.list_max_ziplist_size,
                            server.list_compress_depth);
        /* Load every single element of the list */
        // 逐个读取
        while(len--) {
            if ((ele = rdbLoadEncodedStringObject(rdb)) == NULL) {
                decrRefCount(o);
                return NULL;
            }
            dec = getDecodedObject(ele);
            size_t len = sdslen(dec->ptr);
            quicklistPushTail(o->ptr, dec->ptr, len);
            decrRefCount(dec);
            decrRefCount(ele);
        }
    } else if (rdbtype == RDB_TYPE_SET) {
        // Set 读取长度
        /* Read Set value */
        if ((len = rdbLoadLen(rdb,NULL)) == RDB_LENERR) return NULL;
        /* Use a regular set when there are too many entries. */
        // 数量是否超过压缩大小,创建不同对象
        if (len > server.set_max_intset_entries) {
            o = createSetObject();
            /* It's faster to expand the dict to the right size asap in order
             * to avoid rehashing */
            if (len > DICT_HT_INITIAL_SIZE && dictTryExpand(o->ptr,len) != DICT_OK) {
                rdbReportCorruptRDB("OOM in dictTryExpand %llu", (unsigned long long)len);
                decrRefCount(o);
                return NULL;
            }
        } else {
            o = createIntsetObject();
        }
        /* Load every single element of the set */
        // 逐个读取原始
        for (i = 0; i < len; i++) {
            long long llval;
            sds sdsele;
            if ((sdsele = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL) {
                decrRefCount(o);
                return NULL;
            }
            // 判断是否是压缩做不同逻辑
            if (o->encoding == OBJ_ENCODING_INTSET) {
                /* Fetch integer value from element. */
                if (isSdsRepresentableAsLongLong(sdsele,&llval) == C_OK) {
                    uint8_t success;
                    o->ptr = intsetAdd(o->ptr,llval,&success);
                    if (!success) {
                        rdbReportCorruptRDB("Duplicate set members detected");
                        decrRefCount(o);
                        sdsfree(sdsele);
                        return NULL;
                    }
                } else {
                    setTypeConvert(o,OBJ_ENCODING_HT);
                    if (dictTryExpand(o->ptr,len) != DICT_OK) {
                        rdbReportCorruptRDB("OOM in dictTryExpand %llu", (unsigned long long)len);
                        sdsfree(sdsele);
                        decrRefCount(o);
                        return NULL;
                    }
                }
            }
            /* This will also be called when the set was just converted
             * to a regular hash table encoded set. */
            if (o->encoding == OBJ_ENCODING_HT) {
                if (dictAdd((dict*)o->ptr,sdsele,NULL) != DICT_OK) {
                    rdbReportCorruptRDB("Duplicate set members detected");
                    decrRefCount(o);
                    sdsfree(sdsele);
                    return NULL;
                }
            } else {
                sdsfree(sdsele);
            }
        }
    } else if (rdbtype == RDB_TYPE_ZSET_2 || rdbtype == RDB_TYPE_ZSET) {
        /* Read list/set value. */
        uint64_t zsetlen;
        size_t maxelelen = 0;
        zset *zs;
        // zset, 读取元素数量
        if ((zsetlen = rdbLoadLen(rdb,NULL)) == RDB_LENERR) return NULL;
        o = createZsetObject();
        zs = o->ptr;
        if (zsetlen > DICT_HT_INITIAL_SIZE && dictTryExpand(zs->dict,zsetlen) != DICT_OK) {
            rdbReportCorruptRDB("OOM in dictTryExpand %llu", (unsigned long long)zsetlen);
            decrRefCount(o);
            return NULL;
        }
        /* Load every single element of the sorted set. */
        // 逐个元素加载
        while(zsetlen--) {
            sds sdsele;
            double score;
            zskiplistNode *znode;
            if ((sdsele = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL) {
                decrRefCount(o);
                return NULL;
            }
            if (rdbtype == RDB_TYPE_ZSET_2) {
                if (rdbLoadBinaryDoubleValue(rdb,&score) == -1) {
                    decrRefCount(o);
                    sdsfree(sdsele);
                    return NULL;
                }
            } else {
                if (rdbLoadDoubleValue(rdb,&score) == -1) {
                    decrRefCount(o);
                    sdsfree(sdsele);
                    return NULL;
                }
            }
            /* Don't care about integer-encoded strings. */
            if (sdslen(sdsele) > maxelelen) maxelelen = sdslen(sdsele);
            znode = zslInsert(zs->zsl,score,sdsele);
            if (dictAdd(zs->dict,sdsele,&znode->score) != DICT_OK) {
                rdbReportCorruptRDB("Duplicate zset fields detected");
                decrRefCount(o);
                /* no need to free 'sdsele', will be released by zslFree together with 'o' */
                return NULL;
            }
        }
        /* Convert *after* loading, since sorted sets are not stored ordered. */
        if (zsetLength(o) <= server.zset_max_ziplist_entries &&
            maxelelen <= server.zset_max_ziplist_value)
                zsetConvert(o,OBJ_ENCODING_ZIPLIST);
    } else if (rdbtype == RDB_TYPE_HASH) {
        uint64_t len;
        int ret;
        sds field, value;
        dict *dupSearchDict = NULL;
        // 加载hash 元素数量
        len = rdbLoadLen(rdb, NULL);
        if (len == RDB_LENERR) return NULL;
        o = createHashObject();
        // 是否要用压缩类型
        /* Too many entries? Use a hash table right from the start. */
        if (len > server.hash_max_ziplist_entries)
            hashTypeConvert(o, OBJ_ENCODING_HT);
        else if (deep_integrity_validation) {
            /* In this mode, we need to guarantee that the server won't crash
             * later when the ziplist is converted to a dict.
             * Create a set (dict with no values) to for a dup search.
             * We can dismiss it as soon as we convert the ziplist to a hash. */
            dupSearchDict = dictCreate(&hashDictType, NULL);
        }
        /* Load every field and value into the ziplist */
        // OBJ_ENCODING_ZIPLIST类型处理
        while (o->encoding == OBJ_ENCODING_ZIPLIST && len > 0) {
            len--;
            /* Load raw strings */
            if ((field = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL) {
                decrRefCount(o);
                if (dupSearchDict) dictRelease(dupSearchDict);
                return NULL;
            }
            if ((value = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL) {
                sdsfree(field);
                decrRefCount(o);
                if (dupSearchDict) dictRelease(dupSearchDict);
                return NULL;
            }
            if (dupSearchDict) {
                sds field_dup = sdsdup(field);
                if (dictAdd(dupSearchDict, field_dup, NULL) != DICT_OK) {
                    rdbReportCorruptRDB("Hash with dup elements");
                    dictRelease(dupSearchDict);
                    decrRefCount(o);
                    sdsfree(field_dup);
                    sdsfree(field);
                    sdsfree(value);
                    return NULL;
                }
            }
            /* Add pair to ziplist */
            o->ptr = ziplistPush(o->ptr, (unsigned char*)field,
                    sdslen(field), ZIPLIST_TAIL);
            o->ptr = ziplistPush(o->ptr, (unsigned char*)value,
                    sdslen(value), ZIPLIST_TAIL);
            /* Convert to hash table if size threshold is exceeded */
            if (sdslen(field) > server.hash_max_ziplist_value ||
                sdslen(value) > server.hash_max_ziplist_value)
            {
                sdsfree(field);
                sdsfree(value);
                hashTypeConvert(o, OBJ_ENCODING_HT);
                break;
            }
            sdsfree(field);
            sdsfree(value);
        }
        if (dupSearchDict) {
            /* We no longer need this, from now on the entries are added
             * to a dict so the check is performed implicitly. */
            dictRelease(dupSearchDict);
            dupSearchDict = NULL;
        }
        if (o->encoding == OBJ_ENCODING_HT && len > DICT_HT_INITIAL_SIZE) {
            if (dictTryExpand(o->ptr,len) != DICT_OK) {
                rdbReportCorruptRDB("OOM in dictTryExpand %llu", (unsigned long long)len);
                decrRefCount(o);
                return NULL;
            }
        }
        /* Load remaining fields and values into the hash table */
        // OBJ_ENCODING_HT类型处理
        while (o->encoding == OBJ_ENCODING_HT && len > 0) {
            len--;
            /* Load encoded strings */
            if ((field = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL) {
                decrRefCount(o);
                return NULL;
            }
            if ((value = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL) {
                sdsfree(field);
                decrRefCount(o);
                return NULL;
            }
            /* Add pair to hash table */
            ret = dictAdd((dict*)o->ptr, field, value);
            if (ret == DICT_ERR) {
                rdbReportCorruptRDB("Duplicate hash fields detected");
                sdsfree(value);
                sdsfree(field);
                decrRefCount(o);
                return NULL;
            }
        }
        /* All pairs should be read by now */
        serverAssert(len == 0);
    } else if (rdbtype == RDB_TYPE_LIST_QUICKLIST) {
        // list,先加载梳理
        if ((len = rdbLoadLen(rdb,NULL)) == RDB_LENERR) return NULL;
        o = createQuicklistObject();
        quicklistSetOptions(o->ptr, server.list_max_ziplist_size,
                            server.list_compress_depth);
        // 逐个元素加载
        while (len--) {
            size_t encoded_len;
            unsigned char *zl =
                rdbGenericLoadStringObject(rdb,RDB_LOAD_PLAIN,&encoded_len);
            if (zl == NULL) {
                decrRefCount(o);
                return NULL;
            }
            if (deep_integrity_validation) server.stat_dump_payload_sanitizations++;
            if (!ziplistValidateIntegrity(zl, encoded_len, deep_integrity_validation, NULL, NULL)) {
                rdbReportCorruptRDB("Ziplist integrity check failed.");
                decrRefCount(o);
                zfree(zl);
                return NULL;
            }
            quicklistAppendZiplist(o->ptr, zl);
        }
    } else if (rdbtype == RDB_TYPE_HASH_ZIPMAP  ||
               rdbtype == RDB_TYPE_LIST_ZIPLIST ||
               rdbtype == RDB_TYPE_SET_INTSET   ||
               rdbtype == RDB_TYPE_ZSET_ZIPLIST ||
               rdbtype == RDB_TYPE_HASH_ZIPLIST)
    {
        // rdb 里保存的就是压缩类型的字节流
        size_t encoded_len;
        // 把字节流读出来
        unsigned char *encoded =
            rdbGenericLoadStringObject(rdb,RDB_LOAD_PLAIN,&encoded_len);
        if (encoded == NULL) return NULL;
        o = createObject(OBJ_STRING,encoded); /* Obj type fixed below. */
        /* Fix the object encoding, and make sure to convert the encoded
         * data type into the base type if accordingly to the current
         * configuration there are too many elements in the encoded data
         * type. Note that we only check the length and not max element
         * size as this is an O(N) scan. Eventually everything will get
         * converted. */
        // 根据rdb 不同压缩类型处理
        switch(rdbtype) {
            case RDB_TYPE_HASH_ZIPMAP:
                // hash 类型处理
                /* Since we don't keep zipmaps anymore, the rdb loading for these
                 * is O(n) anyway, use `deep` validation. */
                if (!zipmapValidateIntegrity(encoded, encoded_len, 1)) {
                    rdbReportCorruptRDB("Zipmap integrity check failed.");
                    zfree(encoded);
                    o->ptr = NULL;
                    decrRefCount(o);
                    return NULL;
                }
                /* Convert to ziplist encoded hash. This must be deprecated
                 * when loading dumps created by Redis 2.4 gets deprecated. */
                {
                    unsigned char *zl = ziplistNew();
                    unsigned char *zi = zipmapRewind(o->ptr);
                    unsigned char *fstr, *vstr;
                    unsigned int flen, vlen;
                    unsigned int maxlen = 0;
                    dict *dupSearchDict = dictCreate(&hashDictType, NULL);
                    while ((zi = zipmapNext(zi, &fstr, &flen, &vstr, &vlen)) != NULL) {
                        if (flen > maxlen) maxlen = flen;
                        if (vlen > maxlen) maxlen = vlen;
                        zl = ziplistPush(zl, fstr, flen, ZIPLIST_TAIL);
                        zl = ziplistPush(zl, vstr, vlen, ZIPLIST_TAIL);
                        /* search for duplicate records */
                        sds field = sdstrynewlen(fstr, flen);
                        if (!field || dictAdd(dupSearchDict, field, NULL) != DICT_OK) {
                            rdbReportCorruptRDB("Hash zipmap with dup elements, or big length (%u)", flen);
                            dictRelease(dupSearchDict);
                            sdsfree(field);
                            zfree(encoded);
                            o->ptr = NULL;
                            decrRefCount(o);
                            return NULL;
                        }
                    }
                    dictRelease(dupSearchDict);
                    zfree(o->ptr);
                    o->ptr = zl;
                    o->type = OBJ_HASH;
                    o->encoding = OBJ_ENCODING_ZIPLIST;
                    if (hashTypeLength(o) > server.hash_max_ziplist_entries ||
                        maxlen > server.hash_max_ziplist_value)
                    {
                        hashTypeConvert(o, OBJ_ENCODING_HT);
                    }
                }
                break;
            case RDB_TYPE_LIST_ZIPLIST:
                // list 类型处理
                if (deep_integrity_validation) server.stat_dump_payload_sanitizations++;
                // 检测是否有效list
                if (!ziplistValidateIntegrity(encoded, encoded_len, deep_integrity_validation, NULL, NULL)) {
                    rdbReportCorruptRDB("List ziplist integrity check failed.");
                    zfree(encoded);
                    o->ptr = NULL;
                    decrRefCount(o);
                    return NULL;
                }
                // 有效，直接使用二进制字节流
                o->type = OBJ_LIST;
                o->encoding = OBJ_ENCODING_ZIPLIST;
                listTypeConvert(o,OBJ_ENCODING_QUICKLIST);
                break;
            case RDB_TYPE_SET_INTSET:
                // set 类型处理
                if (deep_integrity_validation) server.stat_dump_payload_sanitizations++;
                // 检测是否有效list
                if (!intsetValidateIntegrity(encoded, encoded_len, deep_integrity_validation)) {
                    rdbReportCorruptRDB("Intset integrity check failed.");
                    zfree(encoded);
                    o->ptr = NULL;
                    decrRefCount(o);
                    return NULL;
                }
                 // 有效，直接使用二进制字节流
                o->type = OBJ_SET;
                o->encoding = OBJ_ENCODING_INTSET;
                if (intsetLen(o->ptr) > server.set_max_intset_entries)
                    // 超出压缩要求大小,转化成dict
                    setTypeConvert(o,OBJ_ENCODING_HT);
                break;
            case RDB_TYPE_ZSET_ZIPLIST:
                // zset 类型处理
                if (deep_integrity_validation) server.stat_dump_payload_sanitizations++;
                // 检测是否有效
                if (!zsetZiplistValidateIntegrity(encoded, encoded_len, deep_integrity_validation)) {
                    rdbReportCorruptRDB("Zset ziplist integrity check failed.");
                    zfree(encoded);
                    o->ptr = NULL;
                    decrRefCount(o);
                    return NULL;
                }
                // 有效，直接使用二进制字节流
                o->type = OBJ_ZSET;
                o->encoding = OBJ_ENCODING_ZIPLIST;
                if (zsetLength(o) > server.zset_max_ziplist_entries)
                     // 超出压缩要求大小,转化成skiplist
                    zsetConvert(o,OBJ_ENCODING_SKIPLIST);
                break;
            case RDB_TYPE_HASH_ZIPLIST:
                // hash 类型处理
                if (deep_integrity_validation) server.stat_dump_payload_sanitizations++;
                // 检测是否有效
                if (!hashZiplistValidateIntegrity(encoded, encoded_len, deep_integrity_validation)) {
                    rdbReportCorruptRDB("Hash ziplist integrity check failed.");
                    zfree(encoded);
                    o->ptr = NULL;
                    decrRefCount(o);
                    return NULL;
                }
                // 有效，直接使用二进制字节流
                o->type = OBJ_HASH;
                o->encoding = OBJ_ENCODING_ZIPLIST;
                if (hashTypeLength(o) > server.hash_max_ziplist_entries)
                     // 超出压缩要求大小,转化成HT
                    hashTypeConvert(o, OBJ_ENCODING_HT);
                break;
            default:
                /* totally unreachable */
                rdbReportCorruptRDB("Unknown RDB encoding type %d",rdbtype);
                break;
        }
    } else if (rdbtype == RDB_TYPE_STREAM_LISTPACKS) {
        // stream 类型处理
        // ... 
    } else if (rdbtype == RDB_TYPE_MODULE || rdbtype == RDB_TYPE_MODULE_2) {
        // MODULE 类型处理
        //  // ...
    } else {
        rdbReportReadError("Unknown RDB encoding type %d",rdbtype);
        return NULL;
    }
    return o;
}
/* Track loading progress in order to serve client's from time to time
   and if needed calculate rdb checksum  */
// rdb 加载进度回调, 设置在rio中,由rioh回调用
void rdbLoadProgressCallback(rio *r, const void *buf, size_t len) {
    if (server.rdb_checksum)
        //更新rdb的checksum
        rioGenericUpdateChecksum(r, buf, len);
    if (server.loading_process_events_interval_bytes &&
        (r->processed_bytes + len)/server.loading_process_events_interval_bytes > r->processed_bytes/server.loading_process_events_interval_bytes)
    {
       // 连着主redis,并且处于接收rdb阶段
        if (server.masterhost && server.repl_state == REPL_STATE_TRANSFER)
            // 回复mastere 告诉它我们收到并加载rdb内容
            replicationSendNewlineToMaster();
        // 设置加载进度
        loadingProgress(r->processed_bytes);
         // 定时做一些实践处理(epoll_wait，处理客户端读写事件)
        processEventsWhileBlocked();
        processModuleLoadingProgressEvent(0);
    }
}
/* Load an RDB file from the rio stream 'rdb'. On success C_OK is returned,
 * otherwise C_ERR is returned and 'errno' is set accordingly. */
// 从rio(文件fd或者socket fd)中加载 rdb 到内存db中
// 循环读取文件内容, 解析成redis kv等不同变量和指令,加载到内存中
int rdbLoadRio(rio *rdb, int rdbflags, rdbSaveInfo *rsi) {
    uint64_t dbid;
    int type, rdbver;
    redisDb *db = server.db+0;
    char buf[1024];
    // 设置rio回调
    rdb->update_cksum = rdbLoadProgressCallback;
    rdb->max_processing_chunk = server.loading_process_events_interval_bytes;
    // 读取magic
    if (rioRead(rdb,buf,9) == 0) goto eoferr;
    buf[9] = '\0';
    if (memcmp(buf,"REDIS",5) != 0) {
        serverLog(LL_WARNING,"Wrong signature trying to load DB from file");
        errno = EINVAL;
        return C_ERR;
    }
    // 获取rdb 版本
    rdbver = atoi(buf+5);
    if (rdbver < 1 || rdbver > RDB_VERSION) {
        serverLog(LL_WARNING,"Can't handle RDB format version %d",rdbver);
        errno = EINVAL;
        return C_ERR;
    }
    /* Key-specific attributes, set by opcodes before the key type. */
    long long lru_idle = -1, lfu_freq = -1, expiretime = -1, now = mstime();
    long long lru_clock = LRU_CLOCK();
    while(1) {
        sds key;
        robj *val;
        /* Read type. */
        // 读取一个字节的type
        if ((type = rdbLoadType(rdb)) == -1) goto eoferr;
        // 根据不同类型做不同处理
        /* Handle special types. */
        if (type == RDB_OPCODE_EXPIRETIME) {
            /* EXPIRETIME: load an expire associated with the next key
             * to load. Note that after loading an expire we need to
             * load the actual type, and continue. */
            // 超时类型,读取时间戳
            expiretime = rdbLoadTime(rdb);
            expiretime *= 1000;
            if (rioGetReadError(rdb)) goto eoferr;
            continue; /* Read next opcode. */
        } else if (type == RDB_OPCODE_EXPIRETIME_MS) {
            /* EXPIRETIME_MS: milliseconds precision expire times introduced
             * with RDB v3. Like EXPIRETIME but no with more precision. */
            // 超时类型,读取微秒时间戳
            expiretime = rdbLoadMillisecondTime(rdb,rdbver);
            if (rioGetReadError(rdb)) goto eoferr;
            continue; /* Read next opcode. */
        } else if (type == RDB_OPCODE_FREQ) {
            /* FREQ: LFU frequency. */
            // LFU 数值
            uint8_t byte;
            if (rioRead(rdb,&byte,1) == 0) goto eoferr;
            lfu_freq = byte;
            continue; /* Read next opcode. */
        } else if (type == RDB_OPCODE_IDLE) {
            /* IDLE: LRU idle time. */
            // LRU 时间
            uint64_t qword;
            // 读取时间戳
            if ((qword = rdbLoadLen(rdb,NULL)) == RDB_LENERR) goto eoferr;
            lru_idle = qword;
            continue; /* Read next opcode. */
        } else if (type == RDB_OPCODE_EOF) {
            // rdb 文件结束了
            /* EOF: End of file, exit the main loop. */
            break;
        } else if (type == RDB_OPCODE_SELECTDB) {
            // db序号, 读取并且设置当前db序号
            /* SELECTDB: Select the specified database. */
            if ((dbid = rdbLoadLen(rdb,NULL)) == RDB_LENERR) goto eoferr;
            if (dbid >= (unsigned)server.dbnum) {
                serverLog(LL_WARNING,
                    "FATAL: Data file was created with a Redis "
                    "server configured to handle more than %d "
                    "databases. Exiting\n", server.dbnum);
                exit(1);
            }
            db = server.db+dbid;
            continue; /* Read next opcode. */
        } else if (type == RDB_OPCODE_RESIZEDB) {
            /* RESIZEDB: Hint about the size of the keys in the currently
             * selected data base, in order to avoid useless rehashing. */
            uint64_t db_size, expires_size;
            // 具体db 的dict和expires 大小
            if ((db_size = rdbLoadLen(rdb,NULL)) == RDB_LENERR)
                goto eoferr;
            if ((expires_size = rdbLoadLen(rdb,NULL)) == RDB_LENERR)
                goto eoferr;
            dictExpand(db->dict,db_size);
            dictExpand(db->expires,expires_size);
            continue; /* Read next opcode. */
        } else if (type == RDB_OPCODE_AUX) {
            // 全局变量
            /* AUX: generic string-string fields. Use to add state to RDB
             * which is backward compatible. Implementations of RDB loading
             * are required to skip AUX fields they don't understand.
             *
             * An AUX field is composed of two strings: key and value. */
            robj *auxkey, *auxval;
            if ((auxkey = rdbLoadStringObject(rdb)) == NULL) goto eoferr;
            if ((auxval = rdbLoadStringObject(rdb)) == NULL) goto eoferr;
            if (((char*)auxkey->ptr)[0] == '%') {
                /* All the fields with a name staring with '%' are considered
                 * information fields and are logged at startup with a log
                 * level of NOTICE. */
                serverLog(LL_NOTICE,"RDB '%s': %s",
                    (char*)auxkey->ptr,
                    (char*)auxval->ptr);
            } else if (!strcasecmp(auxkey->ptr,"repl-stream-db")) {
                if (rsi) rsi->repl_stream_db = atoi(auxval->ptr);
            } else if (!strcasecmp(auxkey->ptr,"repl-id")) {
                if (rsi && sdslen(auxval->ptr) == CONFIG_RUN_ID_SIZE) {
                    memcpy(rsi->repl_id,auxval->ptr,CONFIG_RUN_ID_SIZE+1);
                    rsi->repl_id_is_set = 1;
                }
            } else if (!strcasecmp(auxkey->ptr,"repl-offset")) {
                if (rsi) rsi->repl_offset = strtoll(auxval->ptr,NULL,10);
            } else if (!strcasecmp(auxkey->ptr,"lua")) {
                /* Load the script back in memory. */
                if (luaCreateFunction(NULL,server.lua,auxval) == NULL) {
                    rdbReportCorruptRDB(
                        "Can't load Lua script from RDB file! "
                        "BODY: %s", (char*)auxval->ptr);
                }
            } else if (!strcasecmp(auxkey->ptr,"redis-ver")) {
                serverLog(LL_NOTICE,"Loading RDB produced by version %s",
                    (char*)auxval->ptr);
            } else if (!strcasecmp(auxkey->ptr,"ctime")) {
                time_t age = time(NULL)-strtol(auxval->ptr,NULL,10);
                if (age < 0) age = 0;
                serverLog(LL_NOTICE,"RDB age %ld seconds",
                    (unsigned long) age);
            } else if (!strcasecmp(auxkey->ptr,"used-mem")) {
                long long usedmem = strtoll(auxval->ptr,NULL,10);
                serverLog(LL_NOTICE,"RDB memory usage when created %.2f Mb",
                    (double) usedmem / (1024*1024));
                server.loading_rdb_used_mem = usedmem;
            } else if (!strcasecmp(auxkey->ptr,"aof-preamble")) {
                long long haspreamble = strtoll(auxval->ptr,NULL,10);
                if (haspreamble) serverLog(LL_NOTICE,"RDB has an AOF tail");
            } else if (!strcasecmp(auxkey->ptr,"redis-bits")) {
                /* Just ignored. */
            } else {
                /* We ignore fields we don't understand, as by AUX field
                 * contract. */
                serverLog(LL_DEBUG,"Unrecognized RDB AUX field: '%s'",
                    (char*)auxkey->ptr);
            }
            decrRefCount(auxkey);
            decrRefCount(auxval);
            continue; /* Read type again. */
        } else if (type == RDB_OPCODE_MODULE_AUX) {
            // MODULE相关变量
            /* Load module data that is not related to the Redis key space.
             * Such data can be potentially be stored both before and after the
             * RDB keys-values section. */
            uint64_t moduleid = rdbLoadLen(rdb,NULL);
            int when_opcode = rdbLoadLen(rdb,NULL);
            int when = rdbLoadLen(rdb,NULL);
            if (rioGetReadError(rdb)) goto eoferr;
            if (when_opcode != RDB_MODULE_OPCODE_UINT) {
                rdbReportReadError("bad when_opcode");
                goto eoferr;
            }
            moduleType *mt = moduleTypeLookupModuleByID(moduleid);
            char name[10];
            moduleTypeNameByID(name,moduleid);
            if (!rdbCheckMode && mt == NULL) {
                /* Unknown module. */
                serverLog(LL_WARNING,"The RDB file contains AUX module data I can't load: no matching module '%s'", name);
                exit(1);
            } else if (!rdbCheckMode && mt != NULL) {
                if (!mt->aux_load) {
                    /* Module doesn't support AUX. */
                    serverLog(LL_WARNING,"The RDB file contains module AUX data, but the module '%s' doesn't seem to support it.", name);
                    exit(1);
                }
                RedisModuleIO io;
                moduleInitIOContext(io,mt,rdb,NULL);
                io.ver = 2;
                /* Call the rdb_load method of the module providing the 10 bit
                 * encoding version in the lower 10 bits of the module ID. */
                if (mt->aux_load(&io,moduleid&1023, when) != REDISMODULE_OK || io.error) {
                    moduleTypeNameByID(name,moduleid);
                    serverLog(LL_WARNING,"The RDB file contains module AUX data for the module type '%s', that the responsible module is not able to load. Check for modules log above for additional clues.", name);
                    goto eoferr;
                }
                if (io.ctx) {
                    moduleFreeContext(io.ctx);
                    zfree(io.ctx);
                }
                uint64_t eof = rdbLoadLen(rdb,NULL);
                if (eof != RDB_MODULE_OPCODE_EOF) {
                    serverLog(LL_WARNING,"The RDB file contains module AUX data for the module '%s' that is not terminated by the proper module value EOF marker", name);
                    goto eoferr;
                }
                continue;
            } else {
                /* RDB check mode. */
                robj *aux = rdbLoadCheckModuleValue(rdb,name);
                decrRefCount(aux);
                continue; /* Read next opcode. */
            }
        }
        // 来到这里就是读取 kv
        /* Read key */
        if ((key = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL)) == NULL)
            goto eoferr;
        /* Read value */
        // 加载value, 次数type是具体的value 类型
        if ((val = rdbLoadObject(type,rdb,key)) == NULL) {
            sdsfree(key);
            goto eoferr;
        }
        /* Check if the key already expired. This function is used when loading
         * an RDB file from disk, either at startup, or when an RDB was
         * received from the master. In the latter case, the master is
         * responsible for key expiry. If we would expire keys here, the
         * snapshot taken by the master may not be reflected on the slave.
         * Similarly if the RDB is the preamble of an AOF file, we want to
         * load all the keys as they are, since the log of operations later
         * assume to work in an exact keyspace state. */
        if (iAmMaster() &&
            !(rdbflags&RDBFLAGS_AOF_PREAMBLE) &&
            expiretime != -1 && expiretime < now)
        {
            // 本身是主,并且不是aof混合模式, 过期时间小于现在(已经过期,直接删掉)
            sdsfree(key);
            decrRefCount(val);
        } else {
            // 把key val 条件到对应db中
            robj keyobj;
            initStaticStringObject(keyobj,key);
            /* Add the new object in the hash table */
            int added = dbAddRDBLoad(db,key,val);
            if (!added) {
                // 现在db已经有了,不合理,看是否能够容忍
                if (rdbflags & RDBFLAGS_ALLOW_DUP) {
                    /* This flag is useful for DEBUG RELOAD special modes.
                     * When it's set we allow new keys to replace the current
                     * keys with the same name. */
                    // 异步删除
                    dbSyncDelete(db,&keyobj);
                    // 再设置
                    dbAddRDBLoad(db,key,val);
                } else {
                    // 不能容忍直接报错
                    serverLog(LL_WARNING,
                        "RDB has duplicated key '%s' in DB %d",key,db->id);
                    serverPanic("Duplicated key found in RDB file");
                }
            }
            // 设置过期时间
            /* Set the expire time if needed */
            if (expiretime != -1) {
                setExpire(NULL,db,&keyobj,expiretime);
            }
            // 设置LRU 和LFU相关链表
            /* Set usage information (for eviction). */
            objectSetLRUOrLFU(val,lfu_freq,lru_idle,lru_clock,1000);
            // 产生 module event
            /* call key space notification on key loaded for modules only */
            moduleNotifyKeyspaceEvent(NOTIFY_LOADED, "loaded", &keyobj, db->id);
        }
        /* Loading the database more slowly is useful in order to test
         * certain edge cases. */
        if (server.key_load_delay)
            debugDelay(server.key_load_delay);
        /* Reset the state that is key-specified and is populated by
         * opcodes before the key, so that we start from scratch again. */
        expiretime = -1;
        lfu_freq = -1;
        lru_idle = -1;
    }
    // rdb 版本> 5, 校验checksum
    /* Verify the checksum if RDB version is >= 5 */
    if (rdbver >= 5) {
        uint64_t cksum, expected = rdb->cksum;
        if (rioRead(rdb,&cksum,8) == 0) goto eoferr;
        if (server.rdb_checksum && !server.skip_checksum_validation) {
            memrev64ifbe(&cksum);
            if (cksum == 0) {
                serverLog(LL_WARNING,"RDB file was saved with checksum disabled: no check performed.");
            } else if (cksum != expected) {
                serverLog(LL_WARNING,"Wrong RDB checksum expected: (%llx) but "
                    "got (%llx). Aborting now.",
                        (unsigned long long)expected,
                        (unsigned long long)cksum);
                rdbReportCorruptRDB("RDB CRC error");
                return C_ERR;
            }
        }
    }
    return C_OK;
    /* Unexpected end of file is handled here calling rdbReportReadError():
     * this will in turn either abort Redis in most cases, or if we are loading
     * the RDB file from a socket during initial SYNC (diskless replica mode),
     * we'll report the error to the caller, so that we can retry. */
eoferr:
    serverLog(LL_WARNING,
        "Short read or OOM loading DB. Unrecoverable error, aborting now.");
    rdbReportReadError("Unexpected EOF reading RDB file");
    return C_ERR;
}
/* Like rdbLoadRio() but takes a filename instead of a rio stream. The
 * filename is open for reading and a rio stream object created in order
 * to do the actual loading. Moreover the ETA displayed in the INFO
 * output is initialized and finalized.
 *
 * If you pass an 'rsi' structure initialied with RDB_SAVE_OPTION_INIT, the
 * loading code will fiil the information fields in the structure. */
// wrap rdbLoadRio()
// 从一个文件加载内容
int rdbLoad(char *filename, rdbSaveInfo *rsi, int rdbflags) {
    FILE *fp;
    rio rdb;
    int retval;
    // 打开文件
    if ((fp = fopen(filename,"r")) == NULL) return C_ERR;
    // 设置开始加载的全局状态
    startLoadingFile(fp, filename,rdbflags);
    // 用文件fd 初始化rio
    rioInitWithFile(&rdb,fp);
    // 调用rdbLoadRio加载
    retval = rdbLoadRio(&rdb,rdbflags,rsi);
    fclose(fp);
    stopLoading(retval==C_OK);
    return retval;
}
// path: src/server.c
/* Function called at startup to load RDB or AOF file in memory. */
// InitServerLast(); 之后被调用
void loadDataFromDisk(void) {
    long long start = ustime();
    if (server.aof_state == AOF_ON) {
        // 开启AOF时, 加载AOF文件
        if (loadAppendOnlyFile(server.aof_filename) == C_OK)
            serverLog(LL_NOTICE,"DB loaded from append only file: %.3f seconds",(float)(ustime()-start)/1000000);
    } else {
        // 加载rdb
        rdbSaveInfo rsi = RDB_SAVE_INFO_INIT;
        errno = 0; /* Prevent a stale value from affecting error checking */
        // 从rdb 加载 内容
        if (rdbLoad(server.rdb_filename,&rsi,RDBFLAGS_NONE) == C_OK) {
            // 加载成功
            // 打印加载耗时
            serverLog(LL_NOTICE,"DB loaded from disk: %.3f seconds",
                (float)(ustime()-start)/1000000);
            /* Restore the replication ID / offset from the RDB file. */
            // 是主节点
            if ((server.masterhost ||
                (server.cluster_enabled &&
                nodeIsSlave(server.cluster->myself))) &&
                rsi.repl_id_is_set &&
                rsi.repl_offset != -1 &&
                /* Note that older implementations may save a repl_stream_db
                 * of -1 inside the RDB file in a wrong way, see more
                 * information in function rdbPopulateSaveInfo. */
                rsi.repl_stream_db != -1)
            {
                // 保存自己的replid
                memcpy(server.replid,rsi.repl_id,sizeof(server.replid));
                server.master_repl_offset = rsi.repl_offset;
                /* If we are a slave, create a cached master from this
                 * information, in order to allow partial resynchronizations
                 * with masters. */
                replicationCacheMasterUsingMyself();
                // 切换到rdb 最后操作的db序号
                selectDb(server.cached_master,rsi.repl_stream_db);
            }
        } else if (errno != ENOENT) {
            // rdb文件存在但是加载出错, 报错退出
            serverLog(LL_WARNING,"Fatal error loading the DB: %s. Exiting.",strerror(errno));
            exit(1);
        }
        // 不存在就不加载,正常启动
    }
}
int main(int argc, char **argv) {
    // ...
    ACLLoadUsersAtStartup();
    InitServerLast();
    // 从磁盘加载AOF或者RDB
    loadDataFromDisk();
    // ...
}

```

启动加载 RDB 的调用链:

```
int main(int argc, char **argv) 
    - loadDataFromDisk(); // 从磁盘加载AOF或者RDB
        - int rdbLoad(char *filename, rdbSaveInfo *rsi, int rdbflags)  //从 文件加载
              // 从rio(文件fd或者socket fd)中加载 rdb 到内存db中
            - int rdbLoadRio(rio *rdb, int rdbflags, rdbSaveInfo *rsi) 
                // 循环读取文件内容, 解析成redis kv等不同变量和指令,加载到内存中
                - while(1) 
                    - int rdbLoadType(rio *rdb) // 读取kv 的value类型
                    -  key = rdbGenericLoadStringObject(rdb,RDB_LOAD_SDS,NULL))、 /* Read key */
                    -  val = rdbLoadObject(type,rdb,key)) // 加载value, 次数type是具体的value 类型
                        - // 判断RDB_Type, 根据不同type加载到内存中

```

### 定时启动子进程保存 rdb

###### 启动子进程生成 rdb

```
//path: src/rdb.c
/* Save a Redis object.
 * Returns -1 on error, number of bytes written on success. */
// rdb保存一个具体的Redis object
ssize_t rdbSaveObject(rio *rdb, robj *o, robj *key) {
    ssize_t n = 0, nwritten = 0;
    if (o->type == OBJ_STRING) {
        /* Save a string value */
        // string就保存string
        if ((n = rdbSaveStringObject(rdb,o)) == -1) return -1;
        nwritten += n;
    } else if (o->type == OBJ_LIST) {
        /* Save a list value */
        // 链表,先写入长度
        // 再依次写入具体节点
        if (o->encoding == OBJ_ENCODING_QUICKLIST) {
            quicklist *ql = o->ptr;
            quicklistNode *node = ql->head;
            if ((n = rdbSaveLen(rdb,ql->len)) == -1) return -1;
            nwritten += n;
            while(node) {
                // 判断是否是压缩类型, 调用不同string写入方案
                if (quicklistNodeIsCompressed(node)) {
                    void *data;
                    size_t compress_len = quicklistGetLzf(node, &data);
                    if ((n = rdbSaveLzfBlob(rdb,data,compress_len,node->sz)) == -1) return -1;
                    nwritten += n;
                } else {
                    if ((n = rdbSaveRawString(rdb,node->zl,node->sz)) == -1) return -1;
                    nwritten += n;
                }
                node = node->next;
            }
        } else {
            serverPanic("Unknown list encoding");
        }
    } else if (o->type == OBJ_SET) {
        /* Save a set value */
        if (o->encoding == OBJ_ENCODING_HT) {
            dict *set = o->ptr;
            dictIterator *di = dictGetIterator(set);
            dictEntry *de;
            // 先写set长度
            if ((n = rdbSaveLen(rdb,dictSize(set))) == -1) {
                dictReleaseIterator(di);
                return -1;
            }
            nwritten += n;
            // 遍历set
            while((de = dictNext(di)) != NULL) {
                sds ele = dictGetKey(de);
                // 写元素string
                if ((n = rdbSaveRawString(rdb,(unsigned char*)ele,sdslen(ele)))
                    == -1)
                {
                    dictReleaseIterator(di);
                    return -1;
                }
                nwritten += n;
            }
            dictReleaseIterator(di);
        } else if (o->encoding == OBJ_ENCODING_INTSET) {
            size_t l = intsetBlobLen((intset*)o->ptr);
            // 压缩的INTSET, 直接写进去
            if ((n = rdbSaveRawString(rdb,o->ptr,l)) == -1) return -1;
            nwritten += n;
        } else {
            serverPanic("Unknown set encoding");
        }
    } else if (o->type == OBJ_ZSET) {
        /* Save a sorted set value */
        if (o->encoding == OBJ_ENCODING_ZIPLIST) {
            size_t l = ziplistBlobLen((unsigned char*)o->ptr);
            // 压缩的类型,直接写进去
            if ((n = rdbSaveRawString(rdb,o->ptr,l)) == -1) return -1;
            nwritten += n;
        } else if (o->encoding == OBJ_ENCODING_SKIPLIST) {
            zset *zs = o->ptr;
            zskiplist *zsl = zs->zsl;
            // 先写数量
            if ((n = rdbSaveLen(rdb,zsl->length)) == -1) return -1;
            nwritten += n;
            /* We save the skiplist elements from the greatest to the smallest
             * (that's trivial since the elements are already ordered in the
             * skiplist): this improves the load process, since the next loaded
             * element will always be the smaller, so adding to the skiplist
             * will always immediately stop at the head, making the insertion
             * O(1) instead of O(log(N)). */
            zskiplistNode *zn = zsl->tail;
            // 逐个key val遍历
            while (zn != NULL) {
                // 保存key
                if ((n = rdbSaveRawString(rdb,
                    (unsigned char*)zn->ele,sdslen(zn->ele))) == -1)
                {
                    return -1;
                }
                nwritten += n;
                // 保存value
                if ((n = rdbSaveBinaryDoubleValue(rdb,zn->score)) == -1)
                    return -1;
                nwritten += n;
                zn = zn->backward;
            }
        } else {
            serverPanic("Unknown sorted set encoding");
        }
    } else if (o->type == OBJ_HASH) {
        /* Save a hash value */
        if (o->encoding == OBJ_ENCODING_ZIPLIST) {
            // 压缩类型直接写字节流
            size_t l = ziplistBlobLen((unsigned char*)o->ptr);
            if ((n = rdbSaveRawString(rdb,o->ptr,l)) == -1) return -1;
            nwritten += n;
        } else if (o->encoding == OBJ_ENCODING_HT) {
            dictIterator *di = dictGetIterator(o->ptr);
            dictEntry *de;
            // 先写元素数量
            if ((n = rdbSaveLen(rdb,dictSize((dict*)o->ptr))) == -1) {
                dictReleaseIterator(di);
                return -1;
            }
            nwritten += n;
            // 遍历元素
            while((de = dictNext(di)) != NULL) {
                sds field = dictGetKey(de);
                sds value = dictGetVal(de);
                // 写元素
                if ((n = rdbSaveRawString(rdb,(unsigned char*)field,
                        sdslen(field))) == -1)
                {
                    dictReleaseIterator(di);
                    return -1;
                }
                nwritten += n;
                if ((n = rdbSaveRawString(rdb,(unsigned char*)value,
                        sdslen(value))) == -1)
                {
                    dictReleaseIterator(di);
                    return -1;
                }
                nwritten += n;
            }
            dictReleaseIterator(di);
        } else {
            serverPanic("Unknown hash encoding");
        }
    } else if (o->type == OBJ_STREAM) {
        // stream 类型
        // ...
    }else if (o->type == OBJ_MODULE) {
        // ...
    }else {
        serverPanic("Unknown object type");
    }
    return nwritten;
}
/* Save a key-value pair, with expire time, type, key, value.
 * On error -1 is returned.
 * On success if the key was actually saved 1 is returned. */
// 保存k,v, 并且保持过期时间
int rdbSaveKeyValuePair(rio *rdb, robj *key, robj *val, long long expiretime) {
    int savelru = server.maxmemory_policy & MAXMEMORY_FLAG_LRU;
    int savelfu = server.maxmemory_policy & MAXMEMORY_FLAG_LFU;
    /* Save the expire time */
    if (expiretime != -1) {
        // 过期时间不为0,先写一个过期的opcode
        // 再把过期时间写进去
        if (rdbSaveType(rdb,RDB_OPCODE_EXPIRETIME_MS) == -1) return -1;
        if (rdbSaveMillisecondTime(rdb,expiretime) == -1) return -1;
    }
    /* Save the LRU info. */
    // 保存LRU时,把object分配时间也写进去
    if (savelru) {
        uint64_t idletime = estimateObjectIdleTime(val);
        idletime /= 1000; /* Using seconds is enough and requires less space.*/
        if (rdbSaveType(rdb,RDB_OPCODE_IDLE) == -1) return -1;
        if (rdbSaveLen(rdb,idletime) == -1) return -1;
    }
    /* Save the LFU info. */
    // 保存LFU时,把object访问次数也写进去
    if (savelfu) {
        uint8_t buf[1];
        buf[0] = LFUDecrAndReturn(val);
        /* We can encode this in exactly two bytes: the opcode and an 8
         * bit counter, since the frequency is logarithmic with a 0-255 range.
         * Note that we do not store the halving time because to reset it
         * a single time when loading does not affect the frequency much. */
        if (rdbSaveType(rdb,RDB_OPCODE_FREQ) == -1) return -1;
        if (rdbWriteRaw(rdb,buf,1) == -1) return -1;
    }
    /* Save type, key, value */
    // 写val类型比如RDB_TYPE_STRING, RDB_TYPE_ZSET_2
    if (rdbSaveObjectType(rdb,val) == -1) return -1;
    // 将key写进去(key都是string)
    if (rdbSaveStringObject(rdb,key) == -1) return -1;
    // 将val object 写到rdb
    if (rdbSaveObject(rdb,val,key) == -1) return -1;
    /* Delay return if required (for testing) */
    if (server.rdb_key_save_delay)
        debugDelay(server.rdb_key_save_delay);
    return 1;
}
/* Produces a dump of the database in RDB format sending it to the specified
 * Redis I/O channel. On success C_OK is returned, otherwise C_ERR
 * is returned and part of the output, or all the output, can be
 * missing because of I/O errors.
 *
 * When the function returns C_ERR and if 'error' is not NULL, the
 * integer pointed by 'error' is set to the value of errno just after the I/O
 * error. */
// 生成rdb 文件
// 首先写magic
// 写全局的变量信息
// 逐个扫描db,写当前db 序号; 扫描db 的所有kv, 写kv
// 最后写eof 和 cr64 checksum
int rdbSaveRio(rio *rdb, int *error, int rdbflags, rdbSaveInfo *rsi) {
    dictIterator *di = NULL;
    dictEntry *de;
    char magic[10];
    uint64_t cksum;
    size_t processed = 0;
    int j;
    long key_count = 0;
    long long info_updated_time = 0;
    char *pname = (rdbflags & RDBFLAGS_AOF_PREAMBLE) ? "AOF rewrite" :  "RDB";
    if (server.rdb_checksum)
        rdb->update_cksum = rioGenericUpdateChecksum;
    snprintf(magic,sizeof(magic),"REDIS%04d",RDB_VERSION);
    if (rdbWriteRaw(rdb,magic,9) == -1) goto werr;
    if (rdbSaveInfoAuxFields(rdb,rdbflags,rsi) == -1) goto werr;
    if (rdbSaveModulesAux(rdb, REDISMODULE_AUX_BEFORE_RDB) == -1) goto werr;
    // 逐个db保存k v
    for (j = 0; j < server.dbnum; j++) {
        redisDb *db = server.db+j;
        dict *d = db->dict;
        if (dictSize(d) == 0) continue;
        di = dictGetSafeIterator(d);
        /* Write the SELECT DB opcode */
        // 保存目前db序号
        if (rdbSaveType(rdb,RDB_OPCODE_SELECTDB) == -1) goto werr;
        if (rdbSaveLen(rdb,j) == -1) goto werr;
        /* Write the RESIZE DB opcode. */
        uint64_t db_size, expires_size;
        db_size = dictSize(db->dict);
        expires_size = dictSize(db->expires);
        // 保存当前db 的dict大小 和expires大小
        if (rdbSaveType(rdb,RDB_OPCODE_RESIZEDB) == -1) goto werr;
        if (rdbSaveLen(rdb,db_size) == -1) goto werr;
        if (rdbSaveLen(rdb,expires_size) == -1) goto werr;
        /* Iterate this DB writing every entry */
        // 遍历db 的kv，逐个保存
        while((de = dictNext(di)) != NULL) {
            sds keystr = dictGetKey(de);
            robj key, *o = dictGetVal(de);
            long long expire;
            initStaticStringObject(key,keystr);
            expire = getExpire(db,&key);
            if (rdbSaveKeyValuePair(rdb,&key,o,expire) == -1) goto werr;
            /* When this RDB is produced as part of an AOF rewrite, move
             * accumulated diff from parent to child while rewriting in
             * order to have a smaller final write. */
            // rdb 和aof混合模式,及时读取父进程的rewrite buf
            if (rdbflags & RDBFLAGS_AOF_PREAMBLE &&
                rdb->processed_bytes > processed+AOF_READ_DIFF_INTERVAL_BYTES)
            {
                processed = rdb->processed_bytes;
                aofReadDiffFromParent();
            }
            /* Update child info every 1 second (approximately).
             * in order to avoid calling mstime() on each iteration, we will
             * check the diff every 1024 keys */
            // 定时向父进程发送子进程进度信息
            if ((key_count++ & 1023) == 0) {
                long long now = mstime();
                if (now - info_updated_time >= 1000) {
                    sendChildInfo(CHILD_INFO_TYPE_CURRENT_INFO, key_count, pname);
                    info_updated_time = now;
                }
            }
        }
        dictReleaseIterator(di);
        di = NULL; /* So that we don't release it again on error. */
    }
    /* If we are storing the replication information on disk, persist
     * the script cache as well: on successful PSYNC after a restart, we need
     * to be able to process any EVALSHA inside the replication backlog the
     * master will send us. */
    if (rsi && dictSize(server.lua_scripts)) {
        di = dictGetIterator(server.lua_scripts);
        while((de = dictNext(di)) != NULL) {
            robj *body = dictGetVal(de);
            if (rdbSaveAuxField(rdb,"lua",3,body->ptr,sdslen(body->ptr)) == -1)
                goto werr;
        }
        dictReleaseIterator(di);
        di = NULL; /* So that we don't release it again on error. */
    }
    // 保存一些其他全局信息
    if (rdbSaveModulesAux(rdb, REDISMODULE_AUX_AFTER_RDB) == -1) goto werr;
    /* EOF opcode */
    // 写eof 代表rdb文件末尾
    if (rdbSaveType(rdb,RDB_OPCODE_EOF) == -1) goto werr;
    /* CRC64 checksum. It will be zero if checksum computation is disabled, the
     * loading code skips the check in this case. */
    cksum = rdb->cksum;
    memrev64ifbe(&cksum);
    // 写CRC64 checksum,用于校验
    if (rioWrite(rdb,&cksum,8) == 0) goto werr;
    return C_OK;
werr:
    if (error) *error = errno;
    if (di) dictReleaseIterator(di);
    return C_ERR;
}
/* Save the DB on disk. Return C_ERR on error, C_OK on success. */
// 功能:生成rdb 并且保存在文件 filename
// 首先创建一个临时文件, 调用rioInitWithFile初始化rio
// 再调用rdbSaveRio生成rdb(往rio写内容),实际上写到临时文件
// 最后将临时文件重命名为filename
int rdbSave(char *filename, rdbSaveInfo *rsi) {
    char tmpfile[256];
    char cwd[MAXPATHLEN]; /* Current working dir path for error messages. */
    FILE *fp = NULL;
    rio rdb;
    int error = 0;
    // 创建一个临时文件
    snprintf(tmpfile,256,"temp-%d.rdb", (int) getpid());
    fp = fopen(tmpfile,"w");
    if (!fp) {
        char *cwdp = getcwd(cwd,MAXPATHLEN);
        serverLog(LL_WARNING,
            "Failed opening the RDB file %s (in server root dir %s) "
            "for saving: %s",
            filename,
            cwdp ? cwdp : "unknown",
            strerror(errno));
        return C_ERR;
    }
    // 调用rioInitWithFile初始化rio
    rioInitWithFile(&rdb,fp);
    startSaving(RDBFLAGS_NONE);
    if (server.rdb_save_incremental_fsync)
        rioSetAutoSync(&rdb,REDIS_AUTOSYNC_BYTES);
    // 再调用rdbSaveRio生成rdb(往rio写内容),实际上写到临时文件
    if (rdbSaveRio(&rdb,&error,RDBFLAGS_NONE,rsi) == C_ERR) {
        errno = error;
        goto werr;
    }
    // 刷盘
    /* Make sure data will not remain on the OS's output buffers */
    if (fflush(fp)) goto werr;
    if (fsync(fileno(fp))) goto werr;
    if (fclose(fp)) { fp = NULL; goto werr; }
    fp = NULL;
    /* Use RENAME to make sure the DB file is changed atomically only
     * if the generate DB file is ok. */
    // 最后将临时文件重命名为filename
    if (rename(tmpfile,filename) == -1) {
        char *cwdp = getcwd(cwd,MAXPATHLEN);
        serverLog(LL_WARNING,
            "Error moving temp DB file %s on the final "
            "destination %s (in server root dir %s): %s",
            tmpfile,
            filename,
            cwdp ? cwdp : "unknown",
            strerror(errno));
        unlink(tmpfile);
        stopSaving(0);
        return C_ERR;
    }
    serverLog(LL_NOTICE,"DB saved on disk");
    server.dirty = 0;
    server.lastsave = time(NULL);
    server.lastbgsave_status = C_OK;
    stopSaving(1);
    return C_OK;
werr:
    serverLog(LL_WARNING,"Write error saving DB on disk: %s", strerror(errno));
    if (fp) fclose(fp);
    unlink(tmpfile);
    stopSaving(0);
    return C_ERR;
}
// 开一个后台进程生成rdb,并且保持到文件中
int rdbSaveBackground(char *filename, rdbSaveInfo *rsi) {
    pid_t childpid;
    // 判断已经有子进程, 返回错误
    if (hasActiveChildProcess()) return C_ERR;
    server.dirty_before_bgsave = server.dirty;
    server.lastbgsave_try = time(NULL);
    // fork 一个子进程
    if ((childpid = redisFork(CHILD_TYPE_RDB)) == 0) {
        int retval;
        /* Child */
        // 设置进程名字 cpu亲和性
        redisSetProcTitle("redis-rdb-bgsave");
        redisSetCpuAffinity(server.bgsave_cpulist);
        // 调用rdbSave 生成rdb
        retval = rdbSave(filename,rsi);
        if (retval == C_OK) {
            // 通知父进程rdb已经完成
            sendChildCowInfo(CHILD_INFO_TYPE_RDB_COW_SIZE, "RDB");
        }
        exitFromChild((retval == C_OK) ? 0 : 1);
    } else {
        /* Parent */
        if (childpid == -1) {
            server.lastbgsave_status = C_ERR;
            serverLog(LL_WARNING,"Can't save in background: fork: %s",
                strerror(errno));
            return C_ERR;
        }
        serverLog(LL_NOTICE,"Background saving started by pid %ld",(long) childpid);
        server.rdb_save_time_start = time(NULL);
        server.rdb_child_type = RDB_CHILD_TYPE_DISK;
        return C_OK;
    }
    return C_OK; /* unreached */
}
// path:src/server.c
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
// ...
    /* Check if a background saving or AOF rewrite in progress terminated. */
    if (hasActiveChildProcess() || ldbPendingChildren())
    {
        run_with_period(1000) receiveChildInfo();
        checkChildrenDone();
    } else {
        /* If there is not a background saving/rewrite in progress check if
         * we have to save/rewrite now. */
        // 遍历rdb 保存(生成)策略
        for (j = 0; j < server.saveparamslen; j++) {
            struct saveparam *sp = server.saveparams+j;
            /* Save if we reached the given amount of changes,
             * the given amount of seconds, and if the latest bgsave was
             * successful or if, in case of an error, at least
             * CONFIG_BGSAVE_RETRY_DELAY seconds already elapsed. */
            // 达到时间要求
            // change数量要求
            // 并且(上次bg save正常 或者 已经有CONFIG_BGSAVE_RETRY_DELAY的时间没有bgsave)
            if (server.dirty >= sp->changes &&
                server.unixtime-server.lastsave > sp->seconds &&
                (server.unixtime-server.lastbgsave_try >
                 CONFIG_BGSAVE_RETRY_DELAY ||
                 server.lastbgsave_status == C_OK))
            {
                serverLog(LL_NOTICE,"%d changes in %d seconds. Saving...",
                    sp->changes, (int)sp->seconds);
                rdbSaveInfo rsi, *rsiptr;
                rsiptr = rdbPopulateSaveInfo(&rsi);
                // 开启bg save
                rdbSaveBackground(server.rdb_filename,rsiptr);
                break;
            }
        }
        // ...
    }
// ...
}

```

启动子进程保存 rdb 文件的调用链为:

```
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) // cron,定时执行
    -int rdbSaveBackground(char *filename, rdbSaveInfo *rsi)// 没有其他子进程,变量bg save策略,如果达到要求，开启bg save
        - int redisFork(int purpose) //fork子进程
            - int rdbSave(char *filename, rdbSaveInfo *rsi) //子进程调用此生成rdb 并且保存在文件 filename
                - fp = fopen(tmpfile,"w");// 创建一个临时文件,获得句柄
                - rioInitWithFile(&rdb,fp);// 使用fd初始化rio
                - int rdbSaveRio(rio *rdb, int *error, int rdbflags, rdbSaveInfo *rsi)// 生成rdb 文件
                    - // 首先写magic
                    - // 写全局的变量信息
                    - for (j = 0; j < server.dbnum; j++)// 逐个扫描db,
                        - rdbSaveType(rdb,RDB_OPCODE_SELECTDB) //写当前db 序号;
                        - while(***) //扫描db 的所有kv, 写kv
                            - int rdbSaveKeyValuePair(rio *rdb, robj *key, robj *val,long long expiretime)// 保存k&v,并且保持过期时间
                                - rdbSaveObjectType(rdb,val) // 写val类型比如RDB_TYPE_STRING, RDB_TYPE_ZSET_2
                                - rdbSaveStringObject(rdb,key) // 将key写进去(key都是string)
                                - rdbSaveObject(rdb,val,key) // 将val object 写到rdb
                                    - rdb保存一个具体的Redis object
                - fflush(fp) // 到这里rdb生成完了,刷盘
                - rename(tmpfile,filename) // 最后将临时文件重命名为filename
                - exitFromChild((retval == C_OK) ? 0 : 1);// 退出子进程

```

###### 子进程处理完毕后主进程处理

```
//path: src/rdb.c
/* A background saving child (BGSAVE) terminated its work. Handle this.
 * This function covers the case of actual BGSAVEs. */
// bg save 保存完rdb 后主线程调用handle处理
static void backgroundSaveDoneHandlerDisk(int exitcode, int bysignal) {
    if (!bysignal && exitcode == 0) {
        // 保存rdb 成功
        serverLog(LL_NOTICE,
            "Background saving terminated with success");
        // rdb子进程开启后产生的change数量
        server.dirty = server.dirty - server.dirty_before_bgsave;
        // 记录最后保存时间
        server.lastsave = time(NULL);
        server.lastbgsave_status = C_OK;
    } else if (!bysignal && exitcode != 0) {
        // 保存失败
        serverLog(LL_WARNING, "Background saving error");
        server.lastbgsave_status = C_ERR;
    } else {
        mstime_t latency;
        // 主动被信号kill
        serverLog(LL_WARNING,
            "Background saving terminated by signal %d", bysignal);
        latencyStartMonitor(latency);
        // 删除rdb临时文件
        rdbRemoveTempFile(server.child_pid, 0);
        latencyEndMonitor(latency);
        latencyAddSampleIfNeeded("rdb-unlink-temp-file",latency);
        /* SIGUSR1 is whitelisted, so we have a way to kill a child without
         * triggering an error condition. */
        if (bysignal != SIGUSR1)
            server.lastbgsave_status = C_ERR;
    }
}
/* A background saving child (BGSAVE) terminated its work. Handle this.
 * This function covers the case of RDB -> Slaves socket transfers for
 * diskless replication. */
// bgsave保存rdb发送给副本执行完成后, 主线程会执行这个handle
static void backgroundSaveDoneHandlerSocket(int exitcode, int bysignal) {
    if (!bysignal && exitcode == 0) {
        // 成功
        serverLog(LL_NOTICE,
            "Background RDB transfer terminated with success");
    } else if (!bysignal && exitcode != 0) {
        // 异常
        serverLog(LL_WARNING, "Background transfer error");
    } else {
        // 被信号kill
        serverLog(LL_WARNING,
            "Background transfer terminated by signal %d", bysignal);
    }
    // 关闭通讯句柄
    if (server.rdb_child_exit_pipe!=-1)
        close(server.rdb_child_exit_pipe);
    // 删除子进程给父(主进程)发送rdb的fd
    aeDeleteFileEvent(server.el, server.rdb_pipe_read, AE_READABLE);
    close(server.rdb_pipe_read);
    server.rdb_child_exit_pipe = -1;
    server.rdb_pipe_read = -1;
    zfree(server.rdb_pipe_conns);
    server.rdb_pipe_conns = NULL;
    server.rdb_pipe_numconns = 0;
    server.rdb_pipe_numconns_writing = 0;
    //释放缓冲器: 从子进程读取rdb内容缓冲区
    zfree(server.rdb_pipe_buff);
    server.rdb_pipe_buff = NULL;
    server.rdb_pipe_bufflen = 0;
}
/* When a background RDB saving/transfer terminates, call the right handler. */
// bg rdb 子进程完成后,会调用这个handle
// handle在根据具体的类型: 存盘, socket调用不同的handle处理
void backgroundSaveDoneHandler(int exitcode, int bysignal) {
    int type = server.rdb_child_type;
    switch(server.rdb_child_type) {
    // rdb 保存在磁盘文件的子进程结束
    // 本节聚焦于 RDB_CHILD_TYPE_DISK的处理
    case RDB_CHILD_TYPE_DISK:
        backgroundSaveDoneHandlerDisk(exitcode,bysignal);
        break;
    case RDB_CHILD_TYPE_SOCKET:
        backgroundSaveDoneHandlerSocket(exitcode,bysignal);
        break;
    default:
        serverPanic("Unknown RDB child type.");
        break;
    }
    server.rdb_child_type = RDB_CHILD_TYPE_NONE;
    server.rdb_save_time_last = time(NULL)-server.rdb_save_time_start;
    server.rdb_save_time_start = -1;
    /* Possibly there are slaves waiting for a BGSAVE in order to be served
     * (the first stage of SYNC is a bulk transfer of dump.rdb) */
    // 处理接收rdb 的副本
    // 如果是diskless, 子进程运行完毕也说明发送完毕
    // 非diskless,监听副本tcp fd写事件发送 rdb
    updateSlavesWaitingBgsave((!bysignal && exitcode == 0) ? C_OK : C_ERR, type);
}
// 判断子进程是否退出并做处理
void checkChildrenDone(void) {
    int statloc = 0;
    pid_t pid;
    // 调用 waitpid 获取子进程状态
    if ((pid = waitpid(-1, &statloc, WNOHANG)) != 0) {
        int exitcode = WIFEXITED(statloc) ? WEXITSTATUS(statloc) : -1;
        int bysignal = 0;
        if (WIFSIGNALED(statloc)) bysignal = WTERMSIG(statloc);
        /* sigKillChildHandler catches the signal and calls exit(), but we
         * must make sure not to flag lastbgsave_status, etc incorrectly.
         * We could directly terminate the child process via SIGUSR1
         * without handling it */
        if (exitcode == SERVER_CHILD_NOERROR_RETVAL) {
            bysignal = SIGUSR1;
            exitcode = 1;
        }
        if (pid == -1) {
            serverLog(LL_WARNING,"waitpid() returned an error: %s. "
                "child_type: %s, child_pid = %d",
                strerror(errno),
                strChildType(server.child_type),
                (int) server.child_pid);
        } else if (pid == server.child_pid) {
            if (server.child_type == CHILD_TYPE_RDB) {
                // rdb 子进程退出处理
                backgroundSaveDoneHandler(exitcode, bysignal);
            } else if (server.child_type == CHILD_TYPE_AOF) {
                // 收到aof rewrite子进程退出
                // 执行aof rewrite后主进程处理
                backgroundRewriteDoneHandler(exitcode, bysignal);
            } else if (server.child_type == CHILD_TYPE_MODULE) {
                ModuleForkDoneHandler(exitcode, bysignal);
            } else {
                serverPanic("Unknown child type %d for child pid %d", server.child_type, server.child_pid);
                exit(1);
            }
            if (!bysignal && exitcode == 0) receiveChildInfo();
            resetChildState();
        } else {
            if (!ldbRemoveChild(pid)) {
                serverLog(LL_WARNING,
                          "Warning, detected child with unmatched pid: %ld",
                          (long) pid);
            }
        }
        /* start any pending forks immediately. */
        // 处理副本, 告诉他们rdb发送完毕(RDB_CHILD_TYPE_SOCKET)
        // 或者开始发送rdb(RDB_CHILD_TYPE_DISK)
        replicationStartPendingFork();
    }
}
// path:src/server.c
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
// ...
    /* Check if a background saving or AOF rewrite in progress terminated. */
    if (hasActiveChildProcess() || ldbPendingChildren())
    {
        run_with_period(1000) receiveChildInfo();
        checkChildrenDone();
    } else {
        //...
    }
// ...
}

```

父进程处理 bg save(文件) 进程结束的调用链为:

```
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) //cron 定时执行
    - void checkChildrenDone(void) // 判断子进程是否退出并做处理
        - void backgroundSaveDoneHandler(int exitcode, int bysignal) // bg rdb 子进程完成后,会调用这个handle
            - void backgroundSaveDoneHandlerDisk(int exitcode, int bysignal) // bg save(disk) 保存完rdb 后主线程调用handle处理
                - server.dirty = server.dirty - server.dirty_before_bgsave; // rdb子进程开启后产生的change数量
                - server.lastsave = time(NULL); // 记录最后保存时间     

```

### save 命令和 bgsave 命令

```
// path: src/rdb.c
// save命令,即同步执行保存rdb文件
// 阻塞主进程
void saveCommand(client *c) {
    if (server.child_type == CHILD_TYPE_RDB) {
        addReplyError(c,"Background save already in progress");
        return;
    }
    rdbSaveInfo rsi, *rsiptr;
    rsiptr = rdbPopulateSaveInfo(&rsi);
    // 直接调用rdbSave同步保存
    if (rdbSave(server.rdb_filename,rsiptr) == C_OK) {
        addReply(c,shared.ok);
    } else {
        addReplyErrorObject(c,shared.err);
    }
}
/* BGSAVE [SCHEDULE] */
// save命令,即同步执行保存rdb文件
// 不阻塞主进程(fork子进程处理)
void bgsaveCommand(client *c) {
    int schedule = 0;
    /* The SCHEDULE option changes the behavior of BGSAVE when an AOF rewrite
     * is in progress. Instead of returning an error a BGSAVE gets scheduled. */
    if (c->argc > 1) {
        if (c->argc == 2 && !strcasecmp(c->argv[1]->ptr,"schedule")) {
            schedule = 1;
        } else {
            addReplyErrorObject(c,shared.syntaxerr);
            return;
        }
    }
    rdbSaveInfo rsi, *rsiptr;
    rsiptr = rdbPopulateSaveInfo(&rsi);
    // 已经有偶rdb子进程了,报错
    if (server.child_type == CHILD_TYPE_RDB) {
        addReplyError(c,"Background save already in progress");
    } else if (hasActiveChildProcess()) {
        // 存在其他子进程,判断是否可以等会执行
        if (schedule) {
             // 存在其他子进程,设置server.rdb_bgsave_scheduled
            // serverCron定期检查没有子进程并且rdb_bgsave_scheduled==1
            // 调用rdbSaveBackground 开启子进程保存rdb
            server.rdb_bgsave_scheduled = 1;
            addReplyStatus(c,"Background saving scheduled");
        } else {
            addReplyError(c,
            "Another child process is active (AOF?): can't BGSAVE right now. "
            "Use BGSAVE SCHEDULE in order to schedule a BGSAVE whenever "
            "possible.");
        }
    } else if (rdbSaveBackground(server.rdb_filename,rsiptr) == C_OK) {
        // 调用rdbSaveBackground 开启子进程保存rdb
        addReplyStatus(c,"Background saving started");
    } else {
        addReplyErrorObject(c,shared.err);
    }
}
// path:src/server.c
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) {
// ...
     /* Start a scheduled BGSAVE if the corresponding flag is set. This is
     * useful when we are forced to postpone a BGSAVE because an AOF
     * rewrite is in progress.
     *
     * Note: this code must be after the replicationCron() call above so
     * make sure when refactoring this file to keep this order. This is useful
     * because we want to give priority to RDB savings for replication. */
    // 没有子进程
    // 并且被设置了调度执行(bgsave command)
    if (!hasActiveChildProcess() &&
        server.rdb_bgsave_scheduled &&
        (server.unixtime-server.lastbgsave_try > CONFIG_BGSAVE_RETRY_DELAY ||
         server.lastbgsave_status == C_OK))
    {
        rdbSaveInfo rsi, *rsiptr;
        rsiptr = rdbPopulateSaveInfo(&rsi);
        // 调用dbSaveBackground 产生子进程save rdb
        if (rdbSaveBackground(server.rdb_filename,rsiptr) == C_OK)
            server.rdb_bgsave_scheduled = 0;
    }
// ...
}

```

###### save

*   save 命令直接在主进程遍历 db 生成 rdb
    

###### bgsave

*   在当前没有子进程运行时, 直接调用 rdbSaveBackground 开启子进程保存 rdb
    
*   如果存在其他子进程, 设置 server.rdbbgsavescheduled, 即子进程结束后执行 bg save
    
*   serverCron 定期检查没有子进程并且 rdbbgsavescheduled 为 1 时调用 rdbSaveBackground 开启子进程保存 rdb
    

### diskless 模式 rdb 处理

diskless 模式只用于副本全量同步, 本节聚焦关于 diskless rdb 处理 (主从同步细节后续文章会细讲)

###### 启动子进程处理 diskless rdb

```
// path: src/rdb
/* This is just a wrapper to rdbSaveRio() that additionally adds a prefix
 * and a suffix to the generated RDB dump. The prefix is:
 *
 * $EOF:<40 bytes unguessable hex string>\r\n
 *
 * While the suffix is the 40 bytes hex string we announced in the prefix.
 * This way processes receiving the payload can understand when it ends
 * without doing any processing of the content. */
// wrap rdbSaveRio()
// 在rdb 前进前缀 和后缀
// 可以方便识别文件什么时候结束而不必加载执行rdb
int rdbSaveRioWithEOFMark(rio *rdb, int *error, rdbSaveInfo *rsi) {
    char eofmark[RDB_EOF_MARK_SIZE];
    startSaving(RDBFLAGS_REPLICATION);
    getRandomHexChars(eofmark,RDB_EOF_MARK_SIZE);
    if (error) *error = 0;
    if (rioWrite(rdb,"$EOF:",5) == 0) goto werr;
    if (rioWrite(rdb,eofmark,RDB_EOF_MARK_SIZE) == 0) goto werr;
    if (rioWrite(rdb,"\r\n",2) == 0) goto werr;
    if (rdbSaveRio(rdb,error,RDBFLAGS_NONE,rsi) == C_ERR) goto werr;
    if (rioWrite(rdb,eofmark,RDB_EOF_MARK_SIZE) == 0) goto werr;
    stopSaving(1);
    return C_OK;
werr: /* Write error. */
    /* Set 'error' only if not already set by rdbSaveRio() call. */
    if (error && *error == 0) *error = errno;
    stopSaving(0);
    return C_ERR;
}
/* Spawn an RDB child that writes the RDB to the sockets of the slaves
 * that are currently in SLAVE_STATE_WAIT_BGSAVE_START state. */
// 生成rdb 通过pipe fd 发给主进程, 主机程再将rdb通过socket发送给副本
int rdbSaveToSlavesSockets(rdbSaveInfo *rsi) {
    listNode *ln;
    listIter li;
    pid_t childpid;
    int pipefds[2], rdb_pipe_write, safe_to_exit_pipe;
    // 存在子进程,保存
    if (hasActiveChildProcess()) return C_ERR;
    /* Even if the previous fork child exited, don't start a new one until we
     * drained the pipe. */
    if (server.rdb_pipe_conns) return C_ERR;
    /* Before to fork, create a pipe that is used to transfer the rdb bytes to
     * the parent, we can't let it write directly to the sockets, since in case
     * of TLS we must let the parent handle a continuous TLS state when the
     * child terminates and parent takes over. */
    if (pipe(pipefds) == -1) return C_ERR;
    // 创建父子进程通讯的句柄(发送rdb内容)
    server.rdb_pipe_read = pipefds[0]; /* read end */
    rdb_pipe_write = pipefds[1]; /* write end */
    // 不阻塞
    anetNonBlock(NULL, server.rdb_pipe_read);
    /* create another pipe that is used by the parent to signal to the child
     * that it can exit. */
    // 创建用于父进程通知子进程是否退出的fd
    if (pipe(pipefds) == -1) {
        close(rdb_pipe_write);
        close(server.rdb_pipe_read);
        return C_ERR;
    }
    safe_to_exit_pipe = pipefds[0]; /* read end */
    server.rdb_child_exit_pipe = pipefds[1]; /* write end */
    /* Collect the connections of the replicas we want to transfer
     * the RDB to, which are i WAIT_BGSAVE_START state. */
    server.rdb_pipe_conns = zmalloc(sizeof(connection *)*listLength(server.slaves));
    server.rdb_pipe_numconns = 0;
    server.rdb_pipe_numconns_writing = 0;
    // 遍历当前副本列表,找出那些SLAVE_STATE_WAIT_BGSAVE_START的副本, 告诉他们准备接收全量rdb
    listRewind(server.slaves,&li);
    while((ln = listNext(&li))) {
        client *slave = ln->value;
        // 处于SLAVE_STATE_WAIT_BGSAVE_START,即等待全量加载rdb
        if (slave->replstate == SLAVE_STATE_WAIT_BGSAVE_START) {
            // 保存在server.rdb_pipe_conns
            // 后续收到子进程rdb buf直接遍历发送
            server.rdb_pipe_conns[server.rdb_pipe_numconns++] = slave->conn;
            // 告诉他们准备接收全量rdb
            // 并且设置为SLAVE_STATE_WAIT_BGSAVE_END
            replicationSetupSlaveForFullResync(slave,getPsyncInitialOffset());
        }
    }
    // 创建子进程用于保存rdb
    /* Create the child process. */
    if ((childpid = redisFork(CHILD_TYPE_RDB)) == 0) {
        /* Child */
        int retval, dummy;
        rio rdb;
        // 初始化rio,写的fd其实是与父进程通讯的fd
        rioInitWithFd(&rdb,rdb_pipe_write);
        // 设置进程名字和cpu亲和性
        redisSetProcTitle("redis-rdb-to-slaves");
        redisSetCpuAffinity(server.bgsave_cpulist);
        // 扫描db,生成rdb,写入rio
        retval = rdbSaveRioWithEOFMark(&rdb,NULL,rsi);
        if (retval == C_OK && rioFlush(&rdb) == 0)
            retval = C_ERR;
        // 发送结束消息
        if (retval == C_OK) {
            sendChildCowInfo(CHILD_INFO_TYPE_RDB_COW_SIZE, "RDB");
        }
        // 释放rio
        rioFreeFd(&rdb);
        /* wake up the reader, tell it we're done. */
        close(rdb_pipe_write);
        close(server.rdb_child_exit_pipe); /* close write end so that we can detect the close on the parent. */
        /* hold exit until the parent tells us it's safe. we're not expecting
         * to read anything, just get the error when the pipe is closed. */
        // 等待父进程通知退出
        // 父进程读取完最后rdb内容,读到eof就会关闭safe_to_exit_pipe
        dummy = read(safe_to_exit_pipe, pipefds, 1);
        UNUSED(dummy);
        exitFromChild((retval == C_OK) ? 0 : 1);
    } else {
        /* Parent */
        close(safe_to_exit_pipe);
        if (childpid == -1) {
            // 创建子进程失败
            serverLog(LL_WARNING,"Can't save in background: fork: %s",
                strerror(errno));
            /* Undo the state change. The caller will perform cleanup on
             * all the slaves in BGSAVE_START state, but an early call to
             * replicationSetupSlaveForFullResync() turned it into BGSAVE_END */
            // 遍历对应副本,重新设置为接收完整rdb的副本状态为SLAVE_STATE_WAIT_BGSAVE_START
            listRewind(server.slaves,&li);
            while((ln = listNext(&li))) {
                client *slave = ln->value;
                if (slave->replstate == SLAVE_STATE_WAIT_BGSAVE_END) {
                    slave->replstate = SLAVE_STATE_WAIT_BGSAVE_START;
                }
            }
            // 关闭通讯句柄
            close(rdb_pipe_write);
            close(server.rdb_pipe_read);
            zfree(server.rdb_pipe_conns);
            server.rdb_pipe_conns = NULL;
            server.rdb_pipe_numconns = 0;
            server.rdb_pipe_numconns_writing = 0;
        } else {
            serverLog(LL_NOTICE,"Background RDB transfer started by pid %ld",
                (long) childpid);
            server.rdb_save_time_start = time(NULL);
            server.rdb_child_type = RDB_CHILD_TYPE_SOCKET;
            close(rdb_pipe_write); /* close write in parent so that it can detect the close on the child. */
            // 创建成功, 设置监听rdb_pipe_read的hanlder 接收子进程发送过来的rdb 内容
            if (aeCreateFileEvent(server.el, server.rdb_pipe_read, AE_READABLE, rdbPipeReadHandler,NULL) == AE_ERR) {
                serverPanic("Unrecoverable error creating server.rdb_pipe_read file event.");
            }
        }
        return (childpid == -1) ? C_ERR : C_OK;
    }
    return C_OK; /* Unreached. */
}

```

diskless 启动子进程处理调用链为:

```
int rdbSaveToSlavesSockets(rdbSaveInfo *rsi)// 生成rdb 通过pipe fd 发给主进程, 主机程再将rdb通过socket发送给副本
    - server.rdb_pipe_read = pipefds[0]; rdb_pipe_write = pipefds[1]; // 创建子进程向父进程发送rdb的 pipe
    - safe_to_exit_pipe = pipefds[0]; server.rdb_child_exit_pipe = pipefds[1]; // 创建用于父进程通知子进程是否退出的fd
    - while() //遍历副本client,找出那些SLAVE_STATE_WAIT_BGSAVE_START的副本, 告诉他们准备接收全量rdb
        - 将符合添加的副本客户端保存在server.rdb_pipe_conns中 //后续收到子进程rdb buf直接遍历发送
        - 发生协议告诉他们准备接收全量rdb,并且设置为SLAVE_STATE_WAIT_BGSAVE_END
    - int redisFork(int purpose) //fork子进程
        - rioInitWithFd(&rdb,rdb_pipe_write); // 初始化rio, rdb 写的fd其实是与父进程通讯的pipe fd
        - int rdbSaveRioWithEOFMark(rio *rdb, int *error, rdbSaveInfo *rsi)// 在rdb 前进前缀 和后缀,可以方便识别文件结束而不必加载rdb
            - 往rio(rdb)写前缀
            - int rdbSaveRio(rio *rdb, int *error, int rdbflags, rdbSaveInfo *rsi)// 生成rdb 文件
                    - // 首先写magic
                    - // 写全局的变量信息
                    - for (j = 0; j < server.dbnum; j++)// 逐个扫描db,
                        - rdbSaveType(rdb,RDB_OPCODE_SELECTDB) //写当前db 序号;
                        - while(***) //扫描db 的所有kv, 写kv
                            - int rdbSaveKeyValuePair(rio *rdb, robj *key, robj *val,long long expiretime)// 保存k&v,并且保持过期时间
                                - rdbSaveObjectType(rdb,val) // 写val类型比如RDB_TYPE_STRING, RDB_TYPE_ZSET_2
                                - rdbSaveStringObject(rdb,key) // 将key写进去(key都是string)
                                - rdbSaveObject(rdb,val,key) // 将val object 写到rdb
                                    - rdb保存一个具体的Redis object
            - 往rio(rdb)写后缀
        - close(rdb_pipe_write);// 关闭pipe写句柄,告诉父进程rdb结束了(EOF)
        - dummy = read(safe_to_exit_pipe, pipefds, 1); // 父进程读取完最后rdb内容,读到eof就会关闭safe_to_exit_pipe
        - exitFromChild((retval == C_OK) ? 0 : 1);// 退出子进程

```

跟 diskless 模式跟生成 rdb 文件默认, 启动子进程处理的核心区别是:

*   创建子进程向父进程发送 rdb 内容 pipe
    
*   创建父进程告诉子进程可以正常退出的 pipe
    
*   确定那些副本 client 会接受本次 rdb 内容
    
*   调用 rdbSaveRioWithEOFMark 生成有前后缀的 rdb, 方便副本收 rdb 知道什么时候接受完
    
*   子进程通过关闭向父进程发送 rdb 内容 pipe 的 fd 告诉父进程 rdb 生成完毕
    
*   子进程等待父进程通知才退出
    

###### diskless 父进程接收 rdb

```
// path: src/replication.c
// 主进程收到子进程的rdb内容时 调用】
// 将读取到的rdb 内容发送给本次接收rdb的副本client
/* Called in diskless master, when there's data to read from the child's rdb pipe */
void rdbPipeReadHandler(struct aeEventLoop *eventLoop, int fd, void *clientData, int mask) {
    UNUSED(mask);
    UNUSED(clientData);
    UNUSED(eventLoop);
    int i;
    if (!server.rdb_pipe_buff)
        server.rdb_pipe_buff = zmalloc(PROTO_IOBUF_LEN);
    serverAssert(server.rdb_pipe_numconns_writing==0);
    while (1) {
        server.rdb_pipe_bufflen = read(fd, server.rdb_pipe_buff, PROTO_IOBUF_LEN);
        if (server.rdb_pipe_bufflen < 0) {
            if (errno == EAGAIN || errno == EWOULDBLOCK)
                return;
            serverLog(LL_WARNING,"Diskless rdb transfer, read error sending DB to replicas: %s", strerror(errno));
            // 读取rdb内容异常
            // 直接关闭本次接收rdb的副本 client
            for (i=0; i < server.rdb_pipe_numconns; i++) {
                connection *conn = server.rdb_pipe_conns[i];
                if (!conn)
                    continue;
                client *slave = connGetPrivateData(conn);
                freeClient(slave);
                server.rdb_pipe_conns[i] = NULL;
            }
            // kill rbd子进程
            killRDBChild();
            return;
        }
        if (server.rdb_pipe_bufflen == 0) {
            //收到字节数为0,代表子进程rdb已经发生完毕
            /* EOF - write end was closed. */
            int stillUp = 0;
            // 删除监听读事件
            aeDeleteFileEvent(server.el, server.rdb_pipe_read, AE_READABLE);
            for (i=0; i < server.rdb_pipe_numconns; i++)
            {
                connection *conn = server.rdb_pipe_conns[i];
                if (!conn)
                    continue;
                stillUp++;
            }
            serverLog(LL_WARNING,"Diskless rdb transfer, done reading from pipe, %d replicas still up.", stillUp);
            /* Now that the replicas have finished reading, notify the child that it's safe to exit. 
             * When the server detectes the child has exited, it can mark the replica as online, and
             * start streaming the replication buffers. */
            // 关闭db_child_exit_pipe
            // 告诉子进程正常退出
            close(server.rdb_child_exit_pipe);
            server.rdb_child_exit_pipe = -1;
            return;
        }
        // 将读取到的rdb 内容发送给本次接收rdb的副本
        int stillAlive = 0;
        for (i=0; i < server.rdb_pipe_numconns; i++)
        {
            int nwritten;
            connection *conn = server.rdb_pipe_conns[i];
            if (!conn)
                continue;
            // 尽量给副本tcp发生内容
            client *slave = connGetPrivateData(conn);
            if ((nwritten = connWrite(conn, server.rdb_pipe_buff, server.rdb_pipe_bufflen)) == -1) {
                if (connGetState(conn) != CONN_STATE_CONNECTED) {
                    serverLog(LL_WARNING,"Diskless rdb transfer, write error sending DB to replica: %s",
                        connGetLastError(conn));
                    freeClient(slave);
                    server.rdb_pipe_conns[i] = NULL;
                    continue;
                }
                /* An error and still in connected state, is equivalent to EAGAIN */
                slave->repldboff = 0;
            } else {
                /* Note: when use diskless replication, 'repldboff' is the offset
                 * of 'rdb_pipe_buff' sent rather than the offset of entire RDB. */
                slave->repldboff = nwritten;
                atomicIncr(server.stat_net_output_bytes, nwritten);
            }
            /* If we were unable to write all the data to one of the replicas,
             * setup write handler (and disable pipe read handler, below) */
            // 写到不可写, 设置监听副本客户端可写事件继续发送
            if (nwritten != server.rdb_pipe_bufflen) {
                slave->repl_last_partial_write = server.unixtime;
                server.rdb_pipe_numconns_writing++;
                connSetWriteHandler(conn, rdbPipeWriteHandler);
            }
            stillAlive++;
        }
        if (stillAlive == 0) {
            serverLog(LL_WARNING,"Diskless rdb transfer, last replica dropped, killing fork child.");
            // 如果需要接收的副本客户端都异常了,不用继续生成rdb了
            // 直接kill子进程
            killRDBChild();
        }
        /*  Remove the pipe read handler if at least one write handler was set. */
        if (server.rdb_pipe_numconns_writing || stillAlive == 0) {
            aeDeleteFileEvent(server.el, server.rdb_pipe_read, AE_READABLE);
            break;
        }
    }
}
// path: src/rdb.c
/* Spawn an RDB child that writes the RDB to the sockets of the slaves
 * that are currently in SLAVE_STATE_WAIT_BGSAVE_START state. */
// 生成rdb 通过socke发送给副本
int rdbSaveToSlavesSockets(rdbSaveInfo *rsi) {
    listNode *ln;
    // 创建父子进程通讯的pipe等初始化工作
    //...
    // 创建子进程用于保存rdb
    /* Create the child process. */
    if ((childpid = redisFork(CHILD_TYPE_RDB)) == 0) {
        // 子进程处理逻辑
        //....
    } else {
        /* Parent */
        close(safe_to_exit_pipe);
        if (childpid == -1) {
            // 创建子进程失败处理逻辑
            // 关闭句柄,记录错误之类
            // ..。
        } else {
            serverLog(LL_NOTICE,"Background RDB transfer started by pid %ld",
                (long) childpid);
            server.rdb_save_time_start = time(NULL);
            server.rdb_child_type = RDB_CHILD_TYPE_SOCKET;
            close(rdb_pipe_write); /* close write in parent so that it can detect the close on the child. */
            // 创建成功, 设置监听rdb_pipe_read的hanlder 接收子进程发送过来的rdb 内容
            if (aeCreateFileEvent(server.el, server.rdb_pipe_read, AE_READABLE, rdbPipeReadHandler,NULL) == AE_ERR) {
                serverPanic("Unrecoverable error creating server.rdb_pipe_read file event.");
            }
        }
        return (childpid == -1) ? C_ERR : C_OK;
    }
    return C_OK; /* Unreached. */
}

```

父进程接收 rdb 内容 handle 设置：

*   在 rdbSaveToSlavesSockets 创建子进程成功后, 设置 server.rdbpiperead 的读时间处理 handle 为 rdbPipeReadHandler
    

rdbPipeReadHandler 处理如下:

*   通过 server.rdbpiperead 读取 rdb 内容保存到 server.rdb_buf 中
    
*   将读取到的 rdb 内容发送给本次接收 rdb 的副本 client 们
    
*   某些副本 client tcp 已经不可写, 监听其可写事件, 设置 rdbPipeWriteHandler, client 可写时继续发送 rdb
    
*   如果父进程读到子进程 pipe eof 内容, 删除对 server.rdbpiperead 监听, 关闭 server.rdbchildexit_pipe 告诉子进程正常退出
    

###### diskless rdb 结束后父进程处理

```
// path src/rdb.c
/* A background saving child (BGSAVE) terminated its work. Handle this.
 * This function covers the case of RDB -> Slaves socket transfers for
 * diskless replication. */
// bgsave保存rdb发送给副本执行完成后, 主线程会执行这个handle
static void backgroundSaveDoneHandlerSocket(int exitcode, int bysignal) {
    if (!bysignal && exitcode == 0) {
        // 成功
        serverLog(LL_NOTICE,
            "Background RDB transfer terminated with success");
    } else if (!bysignal && exitcode != 0) {
        // 异常
        serverLog(LL_WARNING, "Background transfer error");
    } else {
        // 被信号kill
        serverLog(LL_WARNING,
            "Background transfer terminated by signal %d", bysignal);
    }
    // 关闭通讯句柄
    if (server.rdb_child_exit_pipe!=-1)
        close(server.rdb_child_exit_pipe);
    // 删除子进程给父(主进程)发送rdb的fd
    aeDeleteFileEvent(server.el, server.rdb_pipe_read, AE_READABLE);
    close(server.rdb_pipe_read);
    server.rdb_child_exit_pipe = -1;
    server.rdb_pipe_read = -1;
    zfree(server.rdb_pipe_conns);
    server.rdb_pipe_conns = NULL;
    server.rdb_pipe_numconns = 0;
    server.rdb_pipe_numconns_writing = 0;
    //释放缓冲器: 从子进程读取rdb内容缓冲区
    zfree(server.rdb_pipe_buff);
    server.rdb_pipe_buff = NULL;
    server.rdb_pipe_bufflen = 0;
}

```

父进程处理 bg save(文件) 进程结束的调用链为:

```
int serverCron(struct aeEventLoop *eventLoop, long long id, void *clientData) //cron 定时执行
    - void checkChildrenDone(void) // 判断子进程是否退出并做处理
        - void backgroundSaveDoneHandler(int exitcode, int bysignal) // bg rdb 子进程完成后,会调用这个handle
            - void backgroundSaveDoneHandlerSocket(int exitcode, int bysignal) // bg save(diskless) 保存完rdb 后主线程调用handle处理
                - close(server.rdb_pipe_read); //关闭读取rdb的pipe句柄
                - zfree(server.rdb_pipe_conns); // 回收本次接受rdb的数组内存
                - zfree(server.rdb_pipe_buff); // 回收接受rdb用的buf

```