> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JyYTZ1wcnMwtIapAWsm80Q)

可重用的列别名
=======

```
WITH intro_cte AS (
    SELECT
        'These are the voyages of the starship Enterprise...' AS intro
), starship_loc_cte AS (
    SELECT
        intro,
        instr(intro, 'starship') AS starship_loc
    FROM intro_cte
)
SELECT
    intro,
    starship_loc,
    substr(intro, starship_loc + len('starship') + 1) AS trimmed_intro
FROM starship_loc_cte;

```

```
SELECT 
     'These are the voyages of the starship Enterprise...' AS intro,
     instr(intro, 'starship') AS starship_loc,
     substr(intro, starship_loc + len('starship') + 1) AS trimmed_intro;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">intro</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">starship_loc</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">trimmed_intro</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">These are the voyages of the starship Enterprise…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">30</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">Enterprise…</td></tr></tbody></table>

动态列选择
=====

不再需要预先知道所有列名称！ DuckDB 可以根据正则表达式模式匹配、EXCLUDE 或 REPLACE 修饰符，甚至 lambda 函数来选择甚至修改列（有关详细信息，请参阅下面有关 lambda 函数的部分！）。

让我们来看看有关《星际迷航》第一季的一些事实。 使用 DuckDB 的 httpfs 扩展，我们可以直接从 GitHub 查询 csv 数据集。 它有几列，所以让我们描述一下它。

```
INSTALL httpfs;
LOAD httpfs;
CREATE TABLE trek_facts AS
    SELECT * FROM 'https://raw.githubusercontent.com/Alex-Monahan/example_datasets/main/Star_Trek-Season_1.csv';

DESCRIBE trek_facts;

```

带有正则表达式的 COLUMNS()
==================

COLUMNS 表达式可以接受作为正则表达式的字符串参数，并将返回与该模式匹配的所有列名称。 第一季中 Warp 发生了怎样的变化？ 让我们检查一下包含单词 warp 的任何列名称。

```
SELECT
    episode_num,
    COLUMNS('.*warp.*')
FROM trek_facts;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">episode_num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">cnt_warp_speed_orders</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">highest_warp_speed_issued</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">0</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">0</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">0</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">3</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">0</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">…</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">27</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">28</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">0</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">0</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">29</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">8</td></tr></tbody></table>

COLUMNS 表达式还可以由其他函数包装，以将这些函数应用于每个选定的列。 让我们简化上面的查询来查看所有剧集的最大值：

```
SELECT
    MAX(COLUMNS('.*warp.*'))
FROM trek_facts;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.cnt_warp_speed_orders)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.highest_warp_speed_issued)</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">5</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">8</td></tr></tbody></table>

我们还可以创建一个适用于多个列的 WHERE 子句。 所有列都必须符合过滤条件，这相当于用 AND 将它们组合起来。 哪些剧集至少有 2 个曲速等级且曲速等级至少为 2？

```
SELECT
    episode_num,
    COLUMNS('.*warp.*')
FROM trek_facts
WHERE
    COLUMNS('.*warp.*') >= 2;
    -- cnt_warp_speed_orders >= 2 
    -- AND 
    -- highest_warp_speed_issued >= 2

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">episode_num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">cnt_warp_speed_orders</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">highest_warp_speed_issued</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">14</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">3</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">7</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">17</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">7</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">18</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">8</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">29</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">8</td></tr></tbody></table>

带 EXCLUDE 和 REPLACE 的 COLUMNS()
===============================

在对各个列应用计算之前，也可以排除或替换它们。 例如，由于我们的数据集仅包含第 1 季，因此我们不需要查找该列的最大值。 这是非常不合逻辑的。

```
SELECT
    MAX(COLUMNS(* EXCLUDE season_num))
FROM trek_facts;


```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.episode_num)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.aired_date)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.cnt_kirk_hookups)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.bool_enterprise_saved_the_day)</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">29</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1967-04-13</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td></tr></tbody></table>

**REPLACE** 语法在应用于动态列集时也很有用。 在此示例中，我们希望在查找每列中的最大值之前将日期转换为时间戳。 以前，这需要整个子查询或 CTE 来预处理该单个列！

```
SELECT
    MAX(COLUMNS(* REPLACE aired_date::timestamp AS aired_date))
FROM trek_facts;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.season_num)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.episode_num)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(aired_date :=CAST(aired_date AS TIMESTAMP))</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">max(trek_facts.bool_enterprise_saved_the_day)</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">29</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1967-04-13 00:00:00</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">…</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1</td></tr></tbody></table>

带有 lambda 函数的 COLUMNS()
=======================

查询动态列集的最灵活方法是通过 lambda 函数。 这允许将任何匹配条件应用于列的名称，而不仅仅是正则表达式。 请参阅下面有关 lambda 函数的更多详细信息。

例如，如果使用 **LIKE** 语法更舒服，我们可以选择与 LIKE 模式匹配的列，而不是与正则表达式匹配的列。

```
SELECT
    episode_num,
    COLUMNS(col -> col LIKE '%warp%')
FROM trek_facts
WHERE
    COLUMNS(col -> col LIKE '%warp%') >= 2;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">episode_num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">cnt_warp_speed_orders</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">highest_warp_speed_issued</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">14</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">3</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">7</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">17</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">7</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">18</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">8</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">29</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">8</td></tr></tbody></table>

自动 JSON 到嵌套类型转换
===============

本系列的第一部分提到 JSON 点表示法参考作为未来的工作。 然而，团队走得更远！ 现在，JSON 可以自动解析为 DuckDB 的原生类型，而不是使用点表示法引用 JSON 类型的列，从而显着加快性能、压缩以及友好的点表示法！

首先，安装并加载 **httpfs** 和 **json** 扩展（如果它们未与您使用的客户端捆绑在一起）。 然后直接查询远程 JSON 文件，就好像它是一个表一样！

```
INSTALL httpfs;
LOAD httpfs;
INSTALL json;
LOAD json;

SELECT 
     starfleet[10].model AS starship 
FROM 'https://raw.githubusercontent.com/vlad-saling/star-trek-ipsum/master/src/content/content.json';

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">starship</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">USS Farragut - NCC-1647 - Ship on which James Kirk served as a phaser station operator. Attacked by the Dikironium Cloud Creature, killing half the crew. ad.</td></tr></tbody></table>

现在我们来看看一些新的 SQL 功能，这些功能超出了上一篇文章的想法！

SELECT 语句中先使用 FROM
==================

构建查询时需要知道的第一件事是数据来自哪里。 那么为什么是 SELECT 语句中的第二个子句呢？ 不再！ DuckDB 正在构建 SQL，因为它应该一直如此 - 将 FROM 子句放在第一位！ 这解决了有关 SQL 的最长期的抱怨之一，DuckDB 团队在 2 天内实施了它。

```
FROM my_table SELECT my_column;

```

不仅如此，SELECT 语句可以完全删除，DuckDB 将假设所有列都应该被 SELECTed。 现在查看表格就这么简单：

```
FROM my_table;
-- SELECT * FROM my_table

```

其他语句（例如 COPY）也得到简化。

```
COPY (FROM trek_facts) TO 'phaser_filled_facts.parquet';

```

除了节省击键次数和保持开发流程状态之外，这还有一个额外的好处：当开始选择要查询的列时，自动完成将拥有更多上下文。

请注意，此语法是完全可选的，因此 SELECT * FROM 键盘快捷键是安全的，即使它们已过时

函数链
===

许多 SQL 博客建议使用 CTE 而不是子查询。 除其他好处外，它们更具可读性。 操作被划分为离散的块，并且可以按从上到下的顺序阅读它们，而不是强迫读者从里到外部阅读。

DuckDB 为每个标量函数实现了相同的可解释性改进！ 使用点运算符将函数链接在一起，就像在 Python 中一样。 链中的先前表达式用作后续函数的第一个参数。

```
SELECT 
     ('Make it so')
          .UPPER()
          .string_split(' ')
          .list_aggr('string_agg','.')
          .concat('.') AS im_not_messing_around_number_one;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">im_not_messing_around_number_one</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">MAKE.IT.SO.</td></tr></tbody></table>

现在将其与旧方法进行比较......

```
SELECT 
     concat(
          list_aggr(
               string_split(
                    UPPER('Make it stop'),
               ' '),
          'string_agg','.'),
     '.') AS oof;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">oof</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">MAKE.IT.STOP.</td></tr></tbody></table>

通过名称 Union
==========

DuckDB 旨在融合最好的数据库和数据框架。 这种新语法的灵感来自于 Pandas 中的 concat 函数。 列不是根据列位置垂直堆叠表，而是按名称匹配并相应地堆叠。 只需将 UNION 替换为 **UNION BY NAME** 或将 **UNION ALL** 替换为 **UNION ALL BY NAME** 即可。

例如，我们必须在《下一代》中添加一些新的外来物种谚语：

```
CREATE TABLE proverbs AS
     SELECT 
          'Revenge is a dish best served cold' AS klingon_proverb 
     UNION ALL BY NAME 
     SELECT 
          'You will be assimilated' AS borg_proverb,
          'If winning is not important, why keep score?' AS klingon_proverb;

FROM proverbs;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">klingon_proverb</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">borg_proverb</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">Revenge is a dish best served cold</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">NULL</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">If winning is not important, why keep score?</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">You will be assimilated</td></tr></tbody></table>

这种方法还有其他好处。 如上所示，不仅可以组合具有不同列顺序的表，而且可以完全组合具有不同列数的表。 这在架构迁移时非常有用，对于 DuckDB 的多文件读取功能特别有用。

按名称插入
=====

SQL 中列顺序严格的另一种常见情况是向表中插入数据时。 列必须与顺序完全匹配，或者所有列名称必须在查询中的两个位置重复。

相反，在插入时在表名后面添加关键字 BY NAME。 可以按任何顺序插入表中列的任何子集。

```
INSERT INTO proverbs BY NAME 
     SELECT 'Resistance is futile' AS borg_proverb;

SELECT * FROM proverbs;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">klingon_proverb</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">borg_proverb</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">Revenge is a dish best served cold</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">NULL</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">If winning is not important, why keep score?</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">You will be assimilated</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">NULL</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">Resistance is futile</td></tr></tbody></table>

动态 PIVOT 和 UNPIVOT
==================

从历史上看，数据库不太适合旋转操作。 然而 DuckDB 的 PIVOT 和 UNPIVOT 子句可以创建或堆叠动态列名称，以实现真正灵活的旋转功能！ 除了灵活性之外，DuckDB 还提供 SQL 标准语法和更友好的简写。

让我们看一下地球与罗慕伦战争刚开始时的一些采购预测数据：

```
CREATE TABLE purchases (item VARCHAR, year INT, count INT);

INSERT INTO purchases
    VALUES ('phasers', 2155, 1035), ('phasers', 2156, 25039), ('phasers', 2157, 95000),
           ('photon torpedoes', 2155, 255), ('photon torpedoes', 2156, 17899), ('photon torpedoes', 2157, 87492);

FROM purchases;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">item</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">year</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">count</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2155</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1035</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2156</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">25039</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2157</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">95000</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2155</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">255</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2156</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">17899</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2157</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">87492</td></tr></tbody></table>

如果每年的数据在视觉上很接近，那么比较我们的相位器需求和光子鱼雷需求就更容易。 让我们将其转变为更友好的格式！ 每年应该收到自己的列（但不需要在查询中指定每年！），我们想要总结总计数，并且我们仍然希望为每个项目保留一个单独的组（行）。

```
CREATE TABLE pivoted_purchases AS
     PIVOT purchases 
          ON year 
          USING SUM(count) 
          GROUP BY item;

FROM pivoted_purchases;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">item</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">2155</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">2156</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">2157</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1035</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">25039</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">95000</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">255</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">17899</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">87492</td></tr></tbody></table>

现在想象一下相反的情况。 工程部门的斯科蒂一直在直观地分析并手动构建他的采购预测。 他更喜欢旋转的东西，这样更容易阅读。 现在您需要将其放回到数据库中！ 这场战争可能会持续一段时间，所以明年你可能需要再次这样做。 让我们编写一个 UNPIVOT 查询来返回可以处理任何年份的原始格式。

COLUMNS 表达式将使用除 item 之外的所有列。 堆叠后，包含来自 pivoted_purchases 的列名称的列应重命名为 year，并且这些列中的值表示计数。 结果是与原始数据集相同的数据集。

```
UNPIVOT pivoted_purchases
     ON COLUMNS(* EXCLUDE item)
     INTO
          NAME year
          VALUE count;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">item</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">year</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">count</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2155</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">1035</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2156</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">25039</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">phasers</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2157</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">95000</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2155</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">255</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2156</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">17899</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">photon torpedoes</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2157</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">87492</td></tr></tbody></table>

我们的 DuckDB 0.8.0 公告帖子中包含了更多示例，并且 PIVOT 和 UNPIVOT 文档页面突出显示了更复杂的查询。

列出 lambda 函数
============

列出 lambda 允许将操作应用于列表中的每个项目。 这些不需要预先定义 - 它们是在查询中动态创建的。

在此示例中，将 lambda 函数与 list_transform 函数结合使用来缩短每艘官方船舶名称。

```
SELECT 
     (['Enterprise NCC-1701', 'Voyager NCC-74656', 'Discovery NCC-1031'])
          .list_transform(x -> x.string_split(' ')[1]) AS short_name;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">ship_name</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">[Enterprise, Voyager, Discovery]</td></tr></tbody></table>

Lambda 还可用于过滤列表中的项目。 lambda 返回一个布尔值列表，list_filter 函数使用该列表来选择特定项目。 contains 函数使用前面描述的函数链。

```
SELECT 
     (['Enterprise NCC-1701', 'Voyager NCC-74656', 'Discovery NCC-1031'])
          .list_filter(x -> x.contains('1701')) AS the_original;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">the_original</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">[Enterprise NCC-1701]</td></tr></tbody></table>

列表推导式
=====

如果有一个简单的语法来修改和过滤列表怎么办？ DuckDB 从 Python 的列表推导式方法中汲取灵感，极大地简化了上述示例。 列表推导式是语法糖 - 这些查询在幕后被重写为 lambda 表达式！

在括号内，首先指定所需的转换，然后指示应迭代哪个列表，最后包括过滤条件。

```
SELECT 
     [x.string_split(' ')[1] 
     FOR x IN ['Enterprise NCC-1701', 'Voyager NCC-74656', 'Discovery NCC-1031'] 
     IF x.contains('1701')] AS ready_to_boldly_go;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">ready_to_boldly_go</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">[Enterprise]</td></tr></tbody></table>

展开 struct.*
===========

DuckDB 中的结构是一组键 / 值对。 结构体存储为每个键一个单独的列。 因此，将结构分解为单独的列在计算上很容易，而且现在语法上也很简单！ 这是允许 SQL 处理动态列名的另一个示例。

```
WITH damage_report AS (
     SELECT {'gold_casualties':5, 'blue_casualties':15, 'red_casualties': 10000} AS casualties
) 
FROM damage_report
SELECT 
     casualties.*;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">gold_casualties</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">blue_casualties</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">red_casualties</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">5</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">15</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">10000</td></tr></tbody></table>

自动创建 struct
===========

DuckDB 提供了一种将任何表转换为单列结构的简单方法。 不要选择列名，而是选择表名本身。

```
WITH officers AS (
     SELECT 'Captain' AS rank, 'Jean-Luc Picard' AS name 
     UNION ALL 
     SELECT 'Lieutenant Commander', 'Data'
) 
FROM officers 
SELECT officers;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">officers</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">{‘rank’: Captain, ‘name’: Jean-Luc Picard}</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">{‘rank’: Lieutenant Commander, ‘name’: Data}</td></tr></tbody></table>

Union 数据类型
==========

DuckDB 利用强类型来提供高性能并提高数据质量。 然而，DuckDB 也尽可能宽容地使用隐式转换等方法来避免总是必须在数据类型之间进行转换。

DuckDB 实现灵活性的另一种方式是新的 UNION 数据类型。 UNION 数据类型允许单个列包含多种类型的值。 这可以被认为是对 SQLite 灵活的数据类型规则的 “选择加入”（与 SQLite 最近宣布的严格表相反的方向）。

默认情况下，DuckDB 在将表组合在一起时会寻找数据类型的公分母。 以下查询产生 VARCHAR 列：

```
SELECT 'The Motion Picture' AS movie UNION ALL 
SELECT 2 UNION ALL 
SELECT 3 UNION ALL 
SELECT 4 UNION ALL 
SELECT 5 UNION ALL 
SELECT 6 UNION ALL 
SELECT 'First Contact';

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">movie</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">varchar</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">The Motion Picture</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">First Contact</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">6</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">5</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">4</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">3</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td></tr></tbody></table>

```
CREATE TABLE movies (
     movie UNION(num INT, name VARCHAR)
);
INSERT INTO movies 
     VALUES ('The Motion Picture'), (2), (3), (4), (5), (6), ('First Contact');

FROM movies 
SELECT 
     movie,
     union_tag(movie) AS type,
     movie.name,
     movie.num;

```

<table width="319"><thead data-style="line-height: 1.75; background: rgba(0, 0, 0, 0.05); font-weight: bold; color: rgb(63, 63, 63);"><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">movie</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">type</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">name</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223);">num</td></tr></thead><tbody><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">union(num integer, name varchar)</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">varchar</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">varchar</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">int32</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">The Motion Picture</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">name</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">The Motion Picture</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">2</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">3</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">3</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">4</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">4</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">5</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">5</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">6</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">num</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">6</td></tr><tr><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">First Contact</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">name</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);">First Contact</td><td data-style="padding: 0.25em 0.5em; line-height: 1.75; border-color: rgb(223, 223, 223); color: rgb(63, 63, 63);"><br></td></tr></tbody></table>

额外的友好功能
=======

其他几个友好的功能值得一提，其中一些功能强大到足以保证他们自己的博客文章。

DuckDB 借鉴了 Pandas 中的描述函数，并实现了 SUMMARIZE 关键字，该关键字将计算数据集中每一列的各种统计信息，以实现快速、高级的概述。 只需将 SUMMARIZE 添加到任何表或 SELECT 语句之前即可。

DuckDB 添加了更多将表连接在一起的方法，使表达常见计算变得更加容易。 LATERAL、ASOF、SEMI 和 ANTI 连接等一些连接存在于其他系统中，但在 DuckDB 中具有高性能实现。 DuckDB 还添加了一个新的 POSITIONAL 连接，该连接按每个表中的行号进行组合，以匹配常用的 Pandas 连接行号索引的功能。 有关详细信息，请参阅 JOIN 文档，并查看描述 DuckDB 最先进的 ASOF 连接的博客文章！

总结和未来工作
=======

DuckDB 的目标是成为最容易使用的数据库。 进程内、零依赖性和强类型化等基本架构决策有助于实现这一目标，但其 SQL 方言的友好性也有很大影响。 通过扩展行业标准 PostgreSQL 方言，DuckDB 旨在提供最简单的方法来表达所需的数据转换。 这些变化的范围从改变 SELECT 语句的古老子句顺序以 FROM 开始，允许以一种全新的方式使用链接函数，到高级嵌套数据类型计算（如列表推导式）。 这些功能均在 0.8.1 版本中提供。

更友好的 SQL 的未来工作包括：

*   • 具有超过 1 个参数的 Lambda 函数，例如 list_zip
    
*   • 下划线作为数字分隔符（例如：1_000_000 而不是 1000000）
    
*   • 扩展用户体验，包括自动加载
    
*   • 文件通配的改进