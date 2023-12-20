> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51cto.com](https://www.51cto.com/article/776943.html)

> 最近十年，Elasticsearch 已经成为了最受欢迎的开源检索引擎，并沉淀了大量的实践案例及优化总结。

最近十年，Elasticsearch 已经成为了最受欢迎的开源检索引擎，并沉淀了大量的实践案例及优化总结。在本文中，我们尽可能全面地总结了 Elasticsearch 日常开发中的一些重要实践 & 避坑指南，希望能为大家提供 Elasticsearch 使用上的一些借鉴点，欢迎讨论！

一、前言
----

本文分享了在工作中关于 ElasticSearch 的一些使用建议。和其他更偏向手册化更注重结论的文章不同，本文将一定程度上阐述部分建议背后的原理及使用姿势参考，避免流于表面，只知其然而不知其所以然。如有不当的地方，欢迎指正！

二、查询相关
------

### 充分利用缓存

*   分片查询缓存（Shard Request Cache）

ES 层面的缓存实现，封装在 IndicesRequestCache 类中。缓存的 Key 是整个客户端请求，缓存内容为单个分片的查询结果。主要作用是对聚合的缓存，查询结果中被缓存的内容主要包括：Aggregations（聚合结果）、Hits.total、以及 Suggestions 等。

并非所有的分片级查询都会被缓存。只有客户端查询请求中 size=0 的情况下才会被缓存。其他不被缓存的条件还包括 Scroll、设置了 Profile 属性，查询类型不是 QUERY_THEN_FETCH，以及设置了 requestCache=false 等。另外一些存在不确定性的查询例如：范围查询带有 Now，由于它是毫秒级别的，缓存下来没有意义，类似的还有在脚本查询中使用了 Math.random() 等函数的查询也不会进行缓存。

当有新的 Segment 写入到分片后，缓存会失效，因为之前的缓存结果已经无法代表整个分片的查询结果。所以分片每次 Refresh 之后，缓存会被清除。

*   节点查询缓存 / 过滤器缓存（Node Query Cache /Filter Cache)

Lucene 层面的缓存实现，封装在 LRUQueryCache 类中，默认开启。缓存的是某个 Filter 子查询语句在一个 Segment 上的查询结果。

并非所有的 Filter 查询都会被缓存。对于体积较小的 Segment 不会建立 Query Cache，因为他们很快会被合并。Segment 的 Doc 数量需要大于 10000，并且占整个分片的 3% 以上才会走 Cache 策略（参考：缓存）。

当 Segment 合并的时候，被删除的 Segment 其关联 Cache 会失效。

#### 01. 使用过滤器上下文（Filter）替代查询上下文（Query）。

*   Filter 不会进行打分操作，而 Must 会。
*   Filter 查询可以被缓存，从而提高查询性能。

正例：

```
// 创建BoolQueryBuilder
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();


    // 构建过滤器上下文
    boolQuery.filter(QueryBuilders.termQuery("field", "value"));
```

反例：

```
// 创建BoolQueryBuilder
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();


    // 构建查询上下文
    boolQuery.must(QueryBuilders.termQuery("field1", "value1"));
```

#### 02. 只关注聚合结果而不关注文档细节时，Size 设置为 0 利用分片查询缓存。

参考示例：

```
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();


    // 添加聚合查询
    sourceBuilder.aggregation(
        AggregationBuilders.terms("term_agg").field("field")
            .subAggregation(AggregationBuilders.sum("sum_agg").field("field"))
        );


    // 设置size为0，只返回聚合结果而不返回文档
    sourceBuilder.size(0);
```

#### 03. 日期范围查询使用绝对时间值。

日期字段上使用 Now，一般来说不会被缓存，因为匹配到的时间一直在变化。因此， 可以从业务的角度来考虑是否一定要用 Now，尽量使用绝对时间值，不需要解析相对时间表达式且利用 Query Cache 能够提高查询效率。例如时间范围查询中使用 Now/h，使用小时级别的单位，可以让缓存在 1 小时内都可能被访问到。

正例：

```
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();


        // 获取当前日期并格式化为绝对时间值
        LocalDateTime now = LocalDateTime.now();
        DateTimeFormatter formatter = DateTimeFormatter.ISO_DATE;
        String currentDate = now.format(formatter);


        // 创建日期范围查询
        sourceBuilder.query(QueryBuilders.rangeQuery("date_field")
                .gte("2022-01-01")
                .lte(currentDate));
```

```
反例：
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();


        // 创建日期范围查询，使用相对时间值
        sourceBuilder.query(QueryBuilders.rangeQuery("date_field")
                .gte("now-7d")
                .lte("now"));
```

### 聚合查询

#### 04. 避免多层聚合嵌套查询。

聚合查询的中间结果和最终结果都会在内存中进行，嵌套过多，会导致内存耗尽。

如：

```
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();


        // 创建主要查询
        sourceBuilder.query(QueryBuilders.matchAllQuery());


        // 创建第一层聚合
        TermsAggregationBuilder termAggBuilder1 = AggregationBuilders.terms("term_agg1").field("field_name1");


        // 创建第二层聚合
        TermsAggregationBuilder termAggBuilder2 = AggregationBuilders.terms("term_agg2").field("field_name2");
        termAggBuilder1.subAggregation(termAggBuilder2);


        // 创建第三层聚合
        TermsAggregationBuilder termAggBuilder3 = AggregationBuilders.terms("term_agg3").field("field_name3");
        termAggBuilder2.subAggregation(termAggBuilder3);


        sourceBuilder.aggregation(termAggBuilder1);
```

#### 05. 嵌套查询建议使用 Composite 聚合查询方式。

对于常见的 Group by A,B,C 这种多维度 Groupby 查询，嵌套聚合的性能很差，嵌套聚合被设计为在每个桶内进行指标计算，对于平铺的 Group by 来说有存在很多冗余计算，另外在 Meta 字段上的序列化反序列化代价也非常大，这类 Group by 替换为 Composite 可以将查询速度提升 2 倍左右。

正例：

```
// 创建Composite Aggregation构建器
        CompositeAggregationBuilder compositeAggregationBuilder = AggregationBuilders
                .composite("group_by_A_B_C")
                .sources(
                        AggregationBuilders.terms("group_by_A").field("fieldA.keyword"),
                        AggregationBuilders.terms("group_by_B").field("fieldB.keyword"),
                        AggregationBuilders.terms("group_by_C").field("fieldC.keyword")
                );


        // 创建查询条件
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder()
                .query(QueryBuilders.matchAllQuery())
                .aggregation(compositeAggregationBuilder)
                .size(0);
```

反例：

```
// 创建Terms Aggregation构建器，按照字段A分组
        TermsAggregationBuilder termsAggregationA = AggregationBuilders.terms("group_by_A").field("fieldA.keyword");


        // 在字段A的基础上创建Terms Aggregation构建器，按照字段B分组
        TermsAggregationBuilder termsAggregationB = AggregationBuilders.terms("group_by_B").field("fieldB.keyword");


        // 在字段B的基础上创建Terms Aggregation构建器，按照字段C分组
        TermsAggregationBuilder termsAggregationC = AggregationBuilders.terms("group_by_C").field("fieldC.keyword");


        // 将字段C的聚合添加到字段B的聚合中
        termsAggregationB.subAggregation(termsAggregationC);


        // 将字段B的聚合添加到字段A的聚合中
        termsAggregationA.subAggregation(termsAggregationB);


        // 创建查询条件
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder()
                .query(QueryBuilders.matchAllQuery())
                .aggregation(termsAggregationA)
                .size(0);
```

#### 06. 避免大聚合查询。

聚合查询的中间结果和最终结果都会在内存中进行，数据量太大会导致内存耗尽。

#### 07. 高基数场景嵌套聚合查询建议使用 BFS 搜索。

聚合是在 ES 内存完成的。当一个聚合操作包含了嵌套的聚合操作时，每个嵌套的聚合操作都会使用上一级聚合操作中构建出的桶作为输入，然后根据自己的聚合条件再进行桶的进一步分组。这样对于每一层嵌套，都会再次动态构建一组新的聚合桶。在高基数场景，嵌套聚合操作会导致聚合桶数量随着嵌套层数的增加指数级增长，最终结果就是占用 ES 大量内存，从而导致 OOM 的情况发生。

默认情况下，ES 使用 DFS（深度优先）搜索。深度优先先构建完整的树，然后修剪无用节点。BFS（广度优先）先执行第一层聚合，再继续下一层聚合之前会先做修剪。

在聚合查询中，使用广度优先算法需要在每个桶级别上缓存文档数据，然后在剪枝阶段后向子聚合重放这些文档。因此，广度优先算法的内存消耗取决于每个桶中的文档数量。对于许多聚合查询，每个桶中的文档数量都非常大，聚合可能会有数千或数十万个文档。

但是，有大量桶但每个桶中文档数量相对较少的情况下，使用广度优先算法能更加高效地利用内存资源，而且可以让我们构建更加复杂的聚合查询。虽然可能会产生大量的桶，但每个桶中只有相对较少的文档，因此使用广度优先搜索算法可以更加节约内存。

参考示例：

```
searchSourceBuilder.aggregation(
        AggregationBuilders.terms("brandIds")
                .collectMode(Aggregator.SubAggCollectionMode.BREADTH_FIRST)
                .field("brandId")
                .size(2000)
                .order(BucketOrder.key(true))
);
```

#### 08. 避免对 text 字段类型使用聚合查询。

*   text 的 Fielddata 会加大对内存的占用，如有需求使用，建议使用 Keyword。

#### 09. 不建议使用 bucket_sort 进行聚合深分页查询。

ES 的高 Cardinality 聚合查询非常消耗内存，超过百万基数的聚合很容易导致节点内存不够用以至 OOM。

bucket_sort 使用桶排序算法，性能问题主要是由于它需要在内存中缓存所有的文档和聚合桶，然后才能进行排序和分页，随着文档数量增多和分页深度增加，性能会逐渐变差，有深分页问题。因为桶排序需要对所有文档进行整体排序，所以它的时间复杂度是 O(NlogN)，其中 N 是文档总数。

目前 Elasticsearch 支持聚合分页（滚动聚合）的目前只有复合聚合 (Composite Aggregation) 一种。滚动的方式类似于 SearchAfter。聚合时指定一个复合键，然后每个分片都按照这个复合键进行排序和聚合，不需要在内存中缓存所有文档和桶，而是可以每次返回一页的数据。

反例：使用 bucket_sort 深分页 RT 达到 5000ms+

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
  BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
  boolQuery.filter(QueryBuilders.termQuery(EsNewApplyDocumentFields.IS_DEL, 0));
  TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("spuIdAgg").field("spuId").order(BucketOrder.key(false)).size(pageNum*pageSize);
  termsAggregationBuilder.subAggregation(new BucketSortPipelineAggregationBuilder("spuBucket",null).from((pageNum-1)*pageSize).size(pageSize));  searchSourceBuilder.query(boolQuery).aggregation(termsAggregationBuilder).size(0);
```

正例：使用 Composite Aggregation 优化后深分页查询：423ms

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        boolQuery.filter(QueryBuilders.termQuery(EsNewApplyDocumentFields.IS_DEL, 0));
        CompositeAggregationBuilder compositeBuilder = new CompositeAggregationBuilder(
                "spuIdAgg",
                Collections.singletonList(new TermsValuesSourceBuilder("spuId").field("spuId").order("desc"))
        ).aggregateAfter(ImmutableMap.of("spuId", "603030")).size(20);
        searchSourceBuilder.query(boolQuery).aggregation(compositeBuilder).aggregation(totalAgg).size(0);
```

### 分页

#### 10. 避免使用 from+size 方式。

ES 中深度翻页排序的花费会随着分页的深度而成倍增长，分页搜索不会单独 “Cache”。每次分页的请求都是一次重新搜索的过程，而不是从第一次搜索的结果中获取。如果数据特别大对 CPU 和内存的消耗会非常巨大甚至会导致 OOM。

#### 11. 避免高实时性 & 大结果集场景使用 Scroll 方式。

基于快照的上下文。实时性高的业务场景不建议使用。大结果集场景将生成大量 Scroll 上下文，可能导致内存消耗过大，建议使用 SearcheAfter 方式。

思考：对于 Scroll 和 SearchAfter 的选用怎么看？两者分别适用于哪种场景？SearchAfter 可以完全替代 Scroll 吗？

Scroll 维护一份当前索引段的快照，适用于非实时滚动遍历全量数据查询，但大量 Contexts 占用堆内存的代价较高；7.10 引入的新特性 Search After + PIT，查询本质是利用前向页面的一组排序之检索匹配下一页，从而保证数据一致性；8.10 官方文档明确指出不再建议使用 Scroll API 进行深分页。如果分页检索超过 Top10000+ 推荐使用 PIT + Search After。

#### 12. SearchAfter 分页 / Scroll ID/ 遍历索引中的数据指定 Sort 字段要保证唯一性，否则会造成分页 / 遍历数据不完整或重复。

#### 13. 建议指定业务字段排序，不要采用默认打分排序。

ES 默认使用 “_score” 字段按评分排序。如在使用 Scroll API 获取数据时，如果没有特殊的排序需求，推荐使用 "sort":"_doc" 让 ES 按索引顺序返回命中文档，可以节省排序开销。原因如下：

*   使用非文档 ID 排序，会导致每次查询 ES 需要在每个分片记住上次返回的最后一个文档，然后下次查询中会对之前已经返回的文档进行忽略过滤，同时在协调节点进行排序操作。文档 ID 排序则不需要上述操作。
*   对于文档 ID 排序，ES 内部进行了特殊优化，性能表现更优。

#### 14. Scroll 查询确保显式调用 clearScroll() 方法清除 Scroll ID。

否则会导致 ES 在过期时间前无法释放 Scroll 结果集占用的内存资源，同时也会占用默认 3000 个 Scroll 查询的容量，导致 too many scroll ID 的查询拒绝报错，影响业务。

### 其他

#### 15. 注意 Must 和 Should 同时出现在语句里的时候，Should 会失效；注意 Must 和 Should 同时出现在同一层级的 bool 查询时，Should 查询会失效。

正例：

```
{"query":{ "bool":{
            "must":[
                {"bool":{
                        "must":[
                            {
                                "term":{
                                    "status.keyword":"1"
                           } }]}},
                {"bool":{
                        "should":[
                            {"term":{
                                    "tag.keyword":"1"
  } } ] }}]}}}
```

反例：

```
{"query":{
        "bool":{
            "must":[
                {
                    "term":{
                        "status.keyword":"1"
                    }}],
            "should":[
                {
                    "term":{
                        "tag.keyword":"1"
                    }
   }]}}}
```

因为 Elasticsearch 中的索引名称是全局可见的，可以通过查询所有索引的方式来枚举某个集群中的所有索引名称。可以通过在 Elasticsearch 配置文件中设置 action.destructive_requires_name 参数来禁止查询 indexName-*。

#### 17. 脚本使用 Stored 方式，避免使用 Inline 方式。

对于固定结构的 Script，使用 Stored 方式，把脚本通过 Kibana 存入 ES 集群，降低重复编译脚本带来的性能损耗。

正例：

```
第1步：通过stored方式，建script模版：
POST _script/activity_discount_price
{
  "script":{
        "lang":"painless",
        "source":"doc.xxx.value * params.discount"
  }
}


第2步：调用script脚本模版：cal_activity_discount
GET index/_search
{
  "script_fields": {
    "discount_price": {
      "script": {
           "id": "activity_discount_price",
           "params":{
               "discount": 0.8
           }
}}}}
```

反例：

```
//直接inline方式，请求中传入脚本：
GET index/_search
{
  "script_fields": {
    "activity_discount_price": {
      "script": {
           "source":"doc.xxx.value * 0.8"
      }
    }
  }
}
```

#### 18. 避免使用 _all 字段。

_all 字段包含了所有的索引字段，如果没有获取原始文档数据的需求，可通过设置 Includes、Excludes 属性来定义放入 _source 的字段。_all 默认将写入的字段拼接成一个大的字符串，并对该字段进行分词，用于支持整个 Doc 的全文检索，“_all” 字段在查询时占用更多的 CPU，同时占用更多的磁盘存储空间，默认为 “false”，不建议开启该字段和使用。

#### 19. 建议用 Get 查询替换 Search 查询。

GET/MGET 直接根据文档 ID 从正排索引中获取内容。Search 不指定_id，根据关键词从倒排索引中获取内容。

#### 20. 避免进行多索引查询。

反例：

```
GET /index1,index2,index3/_search
{
  "query": {
    "match_all": {}
  }
}
```

#### 21. 避免单次召回大量数据，建议使用 _source_includes 和 _source_excludes 参数来包含或排除字段。

大型文档尤其有用，部分字段检索可以节省网络开销。

参考示例：

```
// 创建SearchSourceBuilder，并设置查询条件
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.query(QueryBuilders.matchAllQuery());


        // 设置要包含的字段
        String[] includes = {"field1", "field2"};
        sourceBuilder.fetchSource(includes, Strings.EMPTY_ARRAY);


        // 设置要排除的字段
        String[] excludes = {"field3"};
        sourceBuilder.fetchSource(Strings.EMPTY_ARRAY, excludes);
```

#### 22. 避免使用 Wildcard 进行中缀模糊查询。

ES 官方文档并不推荐使用 Wildcard 来进行中缀模糊的查询，原因在于 ES 内部为了加速这种带有通配符查询，会将输入的字符串 Pattern 构建成一个 DFA (Deterministic Finite Automaton)，而带有通配符的 Pattern 构造出来的 DFA 可能会很复杂，开销很大。

建议使用 ES 官方在 7.9 推出的一种专门用来解决模糊查询慢的 Wildcard 字段类型。与 Text 字段相比，它不会将文本看作是标点符号分割的单词集合；与 Keyword 字段比，它在中缀搜索场景下具有无与伦比的查询速度，且对输入没有大小限制，这是 Keyword 类型无法相比的。

#### 23. 避免使用 Scripting。

Painless 脚本语言语法相对简单，灵活度高，安全性高，性能高（相对于其他脚本，但是其性能比 DSL 要低）。不适用于非复杂业务，一般 DSL 能解决大部分的问题，解决不了的用类似 Painless 等脚本语言。主要性能影响如下：单次查询或更新耗时增加，脚本的执行时间相比于其他查询和更新操作可能会更长，因为在执行脚本之前需要对其进行词法分析、语法分析和代码编译等预处理工作。

#### 24. 避免使用脚本查询（Script Query）计算动态字段，建议在索引时计算并在文档中添加该字段。

例如，我们有一个包含大量用户信息的索引，我们需要查询以 "1234" 开头的所有用户。运行一个脚本查询如 "source":“doc[‘num’].value.startsWith（‘1234’）”。这个查询非常耗费资源，索引时考虑添加 “num_prefix” 的 keyword 字段，然后查询 "name_prefix":“1234”。

三、写入相关
------

#### 25. 避免代码中或手工直接 Refresh 操作。

合理设置索引 Settings/Refresh_Interval 时间，通过系统完成 Refresh 动作。

#### 26. 避免单个文档过大。

鉴于默认 http.max_content_length 设置为 100MB，Elasticsearch 将拒绝索引任何大于该值的文档。

#### 27. 写入数据不指定 Doc_ID，让 ES 自动生成。

索引具有显式 ID 的文档时 ES 在写入过程中会多一步判断的过程，即检查具有相同 ID 的文档是否已经存在于相同的分片中，随着索引增长而变得更加昂贵。

#### 28. 合理使用 Bulk API 批量写。

大数据量写入时可以使用 Bulk，但是请求响应的耗时会增加，即使连接断开，ES 集群内部也仍然在执行。高速大批量数据写入时，可能造成集群短时间内响应缓慢甚至假死的的情况。

*   可以通过性能测试确定最佳数量，官方建议大约 5-15mb。
*   超时时间需要足够长，建议 60s 以上。
*   写入端尽量将数据轮询打到不同节点上。

#### 29. 脚本刷大量数据，写入前调大 Refresh Interval，不建议将副本分片为 0，待写入完成后再调回来。

副本分片重新加入节点会触发副分片恢复 Recovery 流程，如果是大分片会影响集群性能。

四、索引创建
------

### 分片

#### 30. 副本分片数大于等于 1。

高可用性保证。增加副本数可以一定程度上提高搜索性能；但会降低写入性能，建议每个主分片对应 1-2 个副本分片即可。

#### 31. 官方建议单分片限制最大数据条数不超过 2^32 - 1。

#### 32. 索引主分片数量不要设置过大。

ES 创建好索引后，一般情况下不再动态调整主分片数量。

每个分片本质上就是一个 Lucene 索引，因此会消耗相应的文件句柄、内存和 CPU 资源。

ES 使用词频统计来计算相关性，当然这些统计也会分配到各个分片上，如果在大量分片上只维护了很少的数据，则将导致最终的文档相关性较差。

一般来说，我们遵循一些原则：

*   读场景较多则可以设置少一点，写场景则可以设置多一些。
*   控制每个分片占用的硬盘容量不超过 ES 的最大 JVM 的堆空间设置（32G），因此，如果索引的总容量在 200G 左右，那分片大小在 7-8 个左右即可。
*   考虑一下 Node 数量，一般一个节点对应一台物理机，如果分片数远大于节点数，则一个节点上存在多个分片，一旦该节点故障，即使保持了 1 个以上的副本，同样有可能会导致数据丢失，集群无法恢复。所以， 一般都设置分片数不超过节点数的 3 倍。

#### 33. 单个分片数据量不要超过 50GB。

单个索引的规模控制在 1TB 以内，单个分片大小控制在 30 ~ 50GB ，Docs 数控制在 10 亿内，如果超过建议滚动。

Mapping 设计

#### 34. 避免使用字段动态映射功能，指定具体字段类型，子类型（若需要），分词器（特别有场景需要）。

#### 35. 对于不需要分词的字符串字段，使用 Keyword 类型而不是 Text 类型。

#### 36. ES 默认字段个数最大 1000，建议不要超过 100。

单个 Doc 在建立索引时的运算复杂度，最大的因素不在于 Doc 的字节数或者说某个字段 Value 的长度，而是字段的数量。例如在满负载的写入压力测试中，Mapping 相同的情况下，一个有 10 个字段，200 字节的 Doc， 通过增加某些字段 Value 的长度到 500 字节，写入 ES 时速度下降很少，而如果字段数增加到 20，即使整个 Doc 字节数没增加多少，写入速度也会降低一倍。

#### 37. 对于不索引字段，Index 属性设置为 False。

在下面的例子中，Title 字段的 Index 属性被设置为 False，表示该字段不会被包含在索引中。而 Content 字段的 Index 属性默认为 True，表示该字段会被包含在索引中。需要注意的是，即使 Index 属性被设置为 False，该字段仍然会被保存在文档中，可以被查询和聚合。

参考示例：

```
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "index": false
      },
      "content": {
        "type": "text"
      }
    }
  }
}
```

#### 38. 避免使用 Nested 或 Parent/Child。

Nested Query 慢，Parent/Child Query 更慢，针对 1 个 Document，每一个 Nested Field 都会生成一个独立的 Document，这将使 Doc 数量剧增，影响查询效率尤其是 JOIN 的效率。因此能在 Mapping 设计阶段搞定的（大宽表设计或采用比较 Smart 的数据结构），就不要用父子关系的 Mapping。如果一定要使用 Nested Fields，保证 Nested Fields 字段不能过多，目前 ES 默认限制是 Index.mapping.nested_fields.limit=50。不建议使用 Nested，那有什么方式来解决 ES 无法 JOIN 的问题？主要有几种实现方式：

*   在文档建模上尽可能在设计时将业务转化有关联关系的文档形式，使用扁平的文档模型。
*   独立索引存储，实际业务层分多次请求实现。
*   通过宽表冗余存储避免关联。
*   否则 Nested 和 Parent/Child 存储对性能均有一定影响，由于 Nested 更新子文档时需要 Reindex 整个文档，所以对写入性能影响较大，适用于 1 对 n（n 较小）场景；Parent/Child 存储在相同 Type 中，写入相比 Nested 性能高，用于 1 对 n（n 较大）场景，但比 Nested 查询更慢，官网说是 5-10 倍左右。

#### 39. 避免使用 Norms。

Norm 是索引评分因子，如果不用按评分对文档进行排序，设置为 “False”。

参考示例：

```
"title": {"type": "string","norms": {"enabled": false}}
```

对于 Text 类型的字段而言，默认开启了 Norms，而 Keyword 类型的字段则默认关闭了 Norms。

开启 Norms 之后，每篇文档的每个字段需要一个字节存储 Norms。对于 Text 类型的字段而言是默认开启 Norms 的，因此对于不需要评分的 Text 类型的字段，可以禁用 Norms。

#### 40. 对不需要进行聚合 / 排序的字段禁用列存 Doc_Values。

面向列的方式存储，主要用户排序、聚合和访问脚本中字段值等数据访问场景。几乎所有字段类型都支持 Doc_Values，值得注意的是，需要分析的字符串字段除外。默认情况下，所有支持 Doc_Values 的字段都启用了这个功能。如果确定不需要对字段进行排序或聚合，或从脚本访问字段值，则可以禁用此功能以减少冗余存储成本。

### Keyword 和 Numeric 的选择

Keyword 类型的主要缺点是在聚合的时候需要构建全局序数，而数值类型则不用。但低基数字段通常会命中大量结果集，例如性别，使用 Numeric 则会在构建 Bitset 上产生很高的代价。  

综上所述，在类型选择上可以参考下面的原则：

*   在仅查询的情况下，如果有 Range 查询需求，使用 Numeric，否则使用 KeyWord。
*   在仅聚合的情况下，如果明确字段是低基数的，使用 Keyword 配合 Execution_hint:map，其他情况使用 Numeric。
*   剩下 Term 查询 + 聚合的场景，需要综合考虑 Numeric 类型 Term 查询构建 BitSet 和 Keyword 类型构建全局序数哪个代价更大，需要看实际场景，但是目前所知的最坏情况下，构建 Bitset 会导致 CPU 跑满，构建全局序数的主要问题是带来的查询延迟，也会给 JVM 带来一些压力。

#### 41. 对于极少使用 Range 查询的数字值，使用 Keyword 类型。

并非所有数值数据都应映射为数值字段数据类型。Elasticsearch 为查询优化数字字段，例如 Integer or long。如果不需要范围查找，对于 Term 查询而言，Keyword 比 Integer 性能更好。

#### 42. 对于有频繁且较为固定的 Range 查询字段，增加 Keyword 类型 Pre-Indexing 字段。

如果对字段的大多数查询在一个固定的范围上运行 Range 聚合，那么可以增加一个 Keyword 类型的字段，通过将范围 “Pre-Indexing” 到索引中并使用 Terms 聚合来加快聚合速度。

#### 43. 对需要聚合查询的高基数 Keyword 字段启用 Eager_Global_Ordinals。

```
参考：eager_global_ordinals
```

序号（Ordinals）用于在 Keyword 字段上运行 Terms 聚合。序号用一个自增数值表示，ES 维护这个自增数字与实际值的映射关系，并为每一数值分配一个 Bucket，映射关系是 Segment 级别的。

但是做聚合操作时往往需要结合多个 Segment 的结果，而每个 Segment 的 Ordinals 映射关系是不一致的，所以 ES 会在每个分片上创建全局序号（Global Ordinals）结构 ，一个全局统一的映射，维护全局的 Ordinal 与每个 Segment 的 Ordinal 的映射关系。

默认情况下，Global Ordinals 默认是延时构建，在第一次查询如 Term Aggregation 使用到时才会构建。因为 ES 不知道哪些字段将用于 Terms 聚合，哪些字段不会。对于基数大的字段，构建成本较大。

启用 eager_global_ordinals 后，Elasticsearch 会在分片构建时预先计算出全局词项表，以便在查询时能够更快地加载和使用。但启用 eager_global_ordinals 后，每次执行 Refresh 操作都会构建 Global Ordinals，相当于把搜索时候花费的构建成本转移到写入时，所以会对写入效率有一定的影响，可以配合增大索引的 Refresh Interval 来使用。

参考示例：

```
PUT index
{
    "mappings": {
        "type": { 
            "properties": {
                "foo": {
                    "type": "keyword",
                    "eager_global_ordinals" : true
                }
            }
        }
    }
}
```

五、总结
----

最近十年，Elasticsearch 已经成为了最受欢迎的开源检索引擎，并沉淀了大量的实践案例及优化总结。在本文中，我们尽可能全面地总结了 Elasticsearch 日常开发中的一些重要实践 & 避坑指南，希望能为大家提供 Elasticsearch 使用上的一些借鉴点，欢迎讨论！

参考文章：

1.《Elasticsearch 源码解析与优化实战》

2.《Elasticsearch 权威指南》

3.https://www.easyice.cn/archives/367

4.https://www.elastic.co/guide/en/elasticsearch/guide/current/filter-caching.html#_independent_query_caching

5.https://www.elastic.co/guide/cn/elasticsearch/guide/current/_preventing_combinatorial_explosions.html

6.https://www.elastic.co/guide/en/elasticsearch/reference/current/eager-global-ordinals.html