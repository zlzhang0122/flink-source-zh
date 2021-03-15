### Idle State Retention Time

在Flink SQL的开发中，新手开发者特别容易遇到的一个问题就是状态的无限制暴增，并由此导致的checkpoint慢甚至是失败问题，
以及容器化场景下的内存超用被kill的问题。在Flink Streaming的开发中，从Flink 1.6开始可以通过State TTL机制来设置
状态的过期时间和策略，虽然目前来说该设置不一定能及时的清除失效的状态，而且目前也只支持Processing Time时间(Event 
Time时间的目前已经在规划中)，但是总的来说还是特别有效的。那么在Flink SQL场景下，该怎么办呢？这就要说到Flink 提供
的另一个失效状态清除机制--Idle State Retention Time。

它目前主要针对的是Table API和SQL模式下的持续查询和聚合语句，主要是通过借助于Query Configuration配置项来使其生效。
我们知道如果一个持续查询语句没有时间窗口的定义，那么从理论上来说，它是会无限的计算下去的，由此可能导致的问题就是随着时
间的不断增长，其内存中的状态也会越来越多，就会导致快照越来越大，同时占用的内存也会越来越多，最终超出超出内存上限作业被
kill。如果我们在存放每个状态时就能给其设置一个过期时间和定时器，如果在状态存放之后每次被访问过时就更新其过期时间，如果
一直未被访问，就表明该状态已不再需要，在定时器触发时对其进行清理，这样不久可以确保无效的状态都可以得到及时的清理吗？当然
这个方案存在的问题就是过期时间不太好确定。

FLink SQL中通过StreamQueryConfig的withIdleStateRetentionTime()方法，来为其设置最长和最短的清理时间，这样就可以
保证状态最终都被被清除，最长和最短清理时间的差距最少为5分钟，这样能够避免大量状态数据在同一瞬间过期，从而对系统造成压力。