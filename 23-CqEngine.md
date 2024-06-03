## 查询的类型

* `ComparativeQuery` ：比较查询，主要的实现 `Max`、`Min`、`LongestPrefix`，其中 Max、Min的字段需要实现 `Comparable`。
* `SimpleQuery` ：简单查询，主要的实现 `All`、`Between`、`Equal`、`ExistsIn`、`GreaterThan`、`Has`、`In`、`LessThan`、`None`、`StringContains`、`StringEndsWith`、`StringIsContainedIn`、`StringIsPrefixOf`、`StringMatchesRegex`、`StringStartsWith`。
* `LogicalQuery` ：逻辑查询，主要的实现 `And`、`Not`、`Or`。

使用的方法：通过DSL构建查询，DSL的门面是 `QueryFactory`，如果需要自定义查询，可自行实现 `SimpleQuery` 、`SimpleComparativeQuery` 、`LogicalQuery` 或继承其子类，然后扩展 `QueryFactory` 。

## 索引

### 常用的索引类型

* `UniqueIndex` ：唯一索引，一个集合只能存在一个，单字段索引，时间复杂度 O(1)， 只支持 Equal 和 In 查询。
* `HashIndex` : 哈希索引，一个集合可以存在多个，单字段索引，时间复杂度 O(1)， 只支持 Equal 、In 、Has 查询。
* `CompoundIndex` ：复合索引，一个集合可以存在多个，多字段索引，时间复杂度 O(1)，只适用于 And 查询，并且子查询只能是 Equal 查询，不包含 LogicalQuery、ComparativeQuery。
* `NavigableIndex` ：区间索引，排序索引，索引有序，一个集合可以存在多个，单字段索引，底层数据结构为 `ConcurrentSkipListMap`，访问区间时零拷贝，时间复杂度 O(1)，支持 `Equal` 、`In` 、`LessThan` 、`GreaterThan` 、`Between` 、`Has` 查询，使用方式：

````java
public class CqEngineTest {
    public void test() {
        IndexedCollection<Car> cars = new ConcurrentIndexedCollection<>();
        // 1. 使用区间查询的字段实现 Comparable
        // 2. 添加区间索引
        cars.addIndex(NavigableIndex.onAttribute(Car.CAR_ID));
        // 3. 设置阀值为 1.0 使其走区间索引
        Thresholds thresholds = new Thresholds(Arrays.asList(new Threshold(EngineThresholds.INDEX_ORDERING_SELECTIVITY, 1.0D)));
        // 4. 设置区间字段为排序字段
        OrderByOption<Car> orderBy = orderBy(descending(Car.CAR_ID));
        QueryOptions queryOption = queryOptions(orderBy, thresholds);
        // 5. 开启日志，可以不用设置
        queryOption.put(QueryLog.class, true);
        cars.retrieve(lessThan(Car.CAR_ID, 2)).forEach(System.out::println);
    }
}
````

* `StandingQuery` ：定义查询索引，可以针对 Query 建立索引，时间复杂度 O(1)，自定义程度最大，Query 最好设置为 常量，原因是索引命中默认使用Object的默认Equal方法，即比较内存地址，使用常量防止修改不能命中索引。

### 查询算法

* 先使用 排序索引(例如 `NavigableIndex`) 筛选区间，再将结果集通过 `Query` 条件过滤，时间复杂度为 `O(区间筛选后的结果集 * SimpleQuery数量)`
* 先使用 `Query` 条件查询，再使用 `List.sort` 进行快速排序，时间复杂度为 `O(Min(子查询结果集的数量) * SimpleQuery数量 + 快排时间)`

### 索引储存数据结构

```java
public class CollectionQueryEngine<O> implements QueryEngineInternal<O> {
    // NavigableIndex、HashIndex
    private final ConcurrentMap<Attribute<O, ?>, Set<Index<O>>> attributeIndexes = new ConcurrentHashMap<Attribute<O, ?>, Set<Index<O>>>();
    // UniqueIndex
    private final ConcurrentMap<Attribute<O, ?>, Index<O>> uniqueIndexes = new ConcurrentHashMap<Attribute<O, ?>, Index<O>>();
    // CompoundIndex
    private final ConcurrentMap<CompoundAttribute<O>, CompoundIndex<O>> compoundIndexes = new ConcurrentHashMap<CompoundAttribute<O>, CompoundIndex<O>>();
    // StandingQuery
    private final ConcurrentMap<Query<O>, Index<O>> standingQueryIndexes = new ConcurrentHashMap<Query<O>, Index<O>>();
}
```

### 索引命中顺序

* StandingQuery

```java
public class CollectionQueryEngine<O> implements QueryEngineInternal<O> {

    ResultSet<O> retrieveRecursive(Query<O> query, final QueryOptions queryOptions) {
        // retrieveFromStandingQueryIndexIfAvailable 实现为 Index<O> standingQueryIndex = standingQueryIndexes.get(query);
        ResultSet<O> resultSetFromStandingQueryIndex = retrieveFromStandingQueryIndexIfAvailable(query, queryOptions);
        if (resultSetFromStandingQueryIndex != null) {
            return resultSetFromStandingQueryIndex;
        }
        // .... 其他类型查询
    }

}
```

* 对于 `SimpleQuery` 会使用 `StandingQuery > UniqueIndex > HashIndex`

```java

public class CollectionQueryEngine<O> implements QueryEngineInternal<O> {

    <A> ResultSet<O> retrieveSimpleQuery(SimpleQuery<O, A> query, QueryOptions queryOptions) {
        // StandingQuery
        ResultSet<O> lowestCostResultSet = retrieveFromStandingQueryIndexIfAvailable(query, queryOptions);
        if (lowestCostResultSet != null) {
            return lowestCostResultSet;
        }
        // UniqueIndex
        Index<O> uniqueIndex = uniqueIndexes.get(query.getAttribute());
        if (uniqueIndex != null && uniqueIndex.supportsQuery(query, queryOptions)) {
            return uniqueIndex.retrieve(query, queryOptions);
        }
        // HashIndex
        Iterable<Index<O>> indexesOnAttribute = getIndexesOnAttribute(query.getAttribute());
        for (Index<O> index : indexesOnAttribute) {
            if (index.supportsQuery(query, queryOptions)) {
                //...
            }
        }
    }
}

```

* 对于 `LogicalQuery` 会将其子查询中`SimpleQuery`拆分成多次查询，选择最小结果集遍历，合并其他结果集取交集，对子查询中 `LogicalQuery` 进行递归调用查询。

```java
public class CollectionQueryEngine<O> implements QueryEngineInternal<O> {
    
    ResultSet<O> retrieveRecursive(Query<O> query, final QueryOptions queryOptions) {
        // ...
        
        Iterable<ResultSet<O>> resultSetsToMerge = new Iterable<ResultSet<O>>() {
            @Override
            public Iterator<ResultSet<O>> iterator() {
                return new UnmodifiableIterator<ResultSet<O>>() {

                    boolean needToProcessSimpleQueries = and.hasSimpleQueries();
                    boolean needToProcessComparativeQueries = and.hasComparativeQueries();
                    Iterator<LogicalQuery<O>> logicalQueriesIterator = and.getLogicalQueries().iterator();

                    @Override
                    public boolean hasNext() {
                        return needToProcessSimpleQueries || needToProcessComparativeQueries || logicalQueriesIterator.hasNext();
                    }

                    @Override
                    public ResultSet<O> next() {
                        if (needToProcessSimpleQueries) {
                            needToProcessSimpleQueries = false;
                            // 处理子查询中 SimpleQuery
                            return retrieveIntersectionOfSimpleQueries(and.getSimpleQueries(), queryOptions, indexMergeStrategyEnabled);
                        }
                        if (needToProcessComparativeQueries) {
                            needToProcessComparativeQueries = false;
                            // Retrieve results for comparative queries from indexes...
                            return retrieveIntersectionOfComparativeQueries(and.getComparativeQueries(), queryOptions);
                        }
                        // 递归处理子查询中 LogicalQuery
                        return retrieveRecursive(logicalQueriesIterator.next(), queryOptions);
                    }
                };
            }
        };
        return new ResultSetIntersection<O>(resultSetsToMerge, query, queryOptions, useIndexMergeStrategy);
    }
}
````

* `LogicalQuery` 中存在一种特殊查询，可以使用 `CompoundIndex`，即 父查询为 `And`，子查询全是 `Equal`，索引优先级小于 `StandingQuery`，大于 `HashIndex`。

### 使用注意事项

* 查询条件中使用的`Attribute`类型需要与索引的对象类型一直，例如索引的字段类型使用 `SimpleAttribute` ，但是查询时使 `ReflectiveAttribute` ，会导致索引无法命中。
* `BigDecimal` 无法使用 `Equal` 查询，因为底层调用 `BigDecimal` 的 `Equal` 比较，导致匹配不是，`BigDecimal` 需要调用 `compareTo` 匹配，可以将利用其他条件筛选出后使用 `Stream` 过滤，或者新建将 `BigDecimal` 转换为 `String` 的`SimpleAttribute` 进行 `Equal`查询。
* 对于变动的字段最好不要创建索引，因为索引的计算是在 `add` 时完成，如果 `add` 之后直接修改对象属性，会导致索引没有重建，出现查询错误，或者调用 `update` 方法先删除再添加，单影响性能。

### 索引使用示例

* 对于所有的查询都可以使用 `StandingQuery` 方式，达到 O(1) 效果，但是每一种查询都建一个索引占用大量内存，最好使用在特别复杂的查询。
* `And(Equal(A,1),Equal(B,1),Equal(C,1))`，使用 `CompoundIndex(A，B，C)`,时间复杂度 O(1)。
* `Not(And(Equal(A,1),Equal(B,1),Equal(C,1)))`，时间复杂度 O(n)，最好将其该写为 `Or(And(),And(),And())`，或者使用 `StandingQuery` 。
* `And(Equal(A,1),Equal(B,1),And(Equal(C,1)，Equal(D,1)))`，使用 `HashIndex(A)`、`HashIndex(B)`、`CompoundIndex(C，D)`，时间复杂度为 `O(Min(HashIndex(A)、HashIndex(B)、CompoundIndex(C，D) * 3)`
* `in(A,[1,2,3])`，使用 `HashIndex(A)`，时间复杂度 O(1)。
* `And(Equal(A,1),Equal(B,1),greaterThan(C,1))`,将查询改写为 `And(Equal(A,1),Equal(B,1)` ,使用 `CompoundIndex(A，B)`，再使用 `stream` 流过滤。
