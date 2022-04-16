# MyBatis

### 配置文件

#### Setting


| 设置名                           | 描述                                                                                                                                                                                                                                              | 有效值                                          | 默认值                                                |
| ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------- |
| cacheEnabled                     | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。                                                                                                                                                                                          | true                                            | false                                                 |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置`fetchType` 属性来覆盖该项的开关状态。                                                                                                                           | true                                            | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考`lazyLoadTriggerMethods`)。                                                                                                                        | true                                            | false                                                 |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。                                                                                                                                                                                              | true                                            | false                                                 |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。                                                                                                                                              | true                                            | false                                                 |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。                                                                                             | true                                            | false                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。                                                                             | NONE, PARTIAL, FULL                             | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。`NONE`: 不做任何反应 `WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`） `FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                          | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新，可以调用session.flushStatements()来立即执行，或者在方法上使用@flush注解                                  | SIMPLE REUSE BATCH                              | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。                                                                                                                                                                                              | 任意正整数                                      | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。                                                                                                                                                               | 任意正整数                                      | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                                                                                                                                                                                                          | FORWARD_ONLY                                    | SCROLL_SENSITIVE                                      |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。                                                                                                                                                                           | true                                            | false                                                 |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。                                                                                                                                                                 | true                                            | false                                                 |
| mapUnderscoreToCamelCase         | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。                                                                                                                                                            | true                                            | false                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。                         | SESSION                                         | STATEMENT                                             |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。                                                                                        | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。 | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                                                                                                                                                                                                              | 用逗号分隔的方法列表。                          | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                                                                                                                                                                                                             | 一个类型别名或全限定类名。                      | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认`TypeHandler` 。（新增于 3.4.5）                                                                                                                                                                                              | 一个类型别名或全限定类名。                      | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。                                                    | true                                            | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回`null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2）                                                                                    | true                                            | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                                                                                                                                                                                                               | 任何字符串                                      | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。                                                                                                                                                                                             | SLF4J                                           | LOG4J(deprecated since 3.5.9)                         |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。                                                                                                                                                                                                 | CGLIB                                           | JAVASSIST                                             |
| vfsImpl                          | 指定 VFS 的实现                                                                                                                                                                                                                                   | 自定义 VFS 的实现的类全限定名，以逗号分隔。     | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上`-parameters` 选项。（新增于 3.4.1）                                                                                                             | true                                            | false                                                 |
| configurationFactory             | 指定一个提供`Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3）                                         | 一个类型别名或完全限定类名。                    | 未设置                                                |
| shrinkWhitespacesInSql           | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5)                                                                                                                                                                   | true                                            | false                                                 |
| defaultSqlProviderType           | Specifies an sql provider class that holds provider method (Since 3.5.6). This class apply to the`type`(or `value`) attribute on sql provider annotation(e.g. `@SelectProvider`), when these attribute was omitted.                               | A type alias or fully qualified class name      | Not set                                               |
| nullableOnForEach                | Specifies the default value of 'nullable' attribute on 'foreach' tag. (Since 3.5.9)                                                                                                                                                               | true                                            | false                                                 |

#### TypeAliases

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>

# 或
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
# 可在对应的 javabean 中使用 @Alias，默认使用 首字母小写 来作为别名

# 或在 aplication.properties 中配置
mybatis.type-aliases-package=com.test.model
```

默认别名：


| 别名       | 映射的类型 |
| ------------ | ------------ |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

#### TypeHandlers

可继承`org.apache.ibatis.type.TypeHandler`或`org.apache.ibatis.type.BaseTypeHandler`来实现，并使用`@MappedTypes`和`@MappedJdbcTypes`注解来配置转换映射关系，然后在配置文件中配置，例如：

```xml
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
# 或
<typeHandlers>
  <package name="org.mybatis.example"/>
</typeHandlers>
# 或在 aplication.properties 中配置
mybatis.type-handlers-package=org.mybatis.example
```

对于枚举类型的自定义：

```java
public interface BaseBizEnum {
    Integer getCode();
}

public enum AgreementType implements BaseBizEnum{
    /***/
    QUICK_PAY(1,"免密支付"),
    ;

    private final Integer code;
    @Getter
    private final String desc;
    private static Map<Integer,AgreementType> itemMap = Maps.newHashMap();
    static {
        for (AgreementType typeEnum : AgreementType.values()) {
            itemMap.put(typeEnum.getCode(),typeEnum);
        }
    }
    AgreementType(Integer code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    //重写了接口BaseBizEnum的方法
    @Override
    public Integer getCode() {
        return code;
    }
    public static AgreementType ofNullable(Integer code) {
        return itemMap.get(code);
    }
}

public class BizEnumTypeHandler<E extends BaseBizEnum> extends BaseTypeHandler<E> {

    private Class<E> type;

    //初始化时定义枚举和code的映射关系
    private final Map<Integer,E> enumsMap = new HashMap<>();

    public BizEnumTypeHandler(Class<E> type) {
        if (type == null) {
            throw new IllegalArgumentException("Type argument cannot be null");
        }
        this.type = type;
        for (E enumConstant : type.getEnumConstants()) {
            enumsMap.put(enumConstant.getCode(),enumConstant);
        }
        if (this.enumsMap.size() == 0) {
            throw new IllegalArgumentException(type.getSimpleName() + " does not represent an enum type.");
        }
    }

    //在请求Sql执行时转换参数
    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, E e, JdbcType jdbcType) throws SQLException {
        preparedStatement.setInt(i,e.getCode());
    }

    //处理返回结果
    @Override
    public E getNullableResult(ResultSet resultSet, String columnName) throws SQLException {
        if (resultSet.wasNull()) {
            return null;
        }
        int code = resultSet.getInt(columnName);
        return getEnum(code);
    }

    private E getEnum(Integer code) {
        try {
            return getEnumByValue(code);
        } catch (Exception ex) {
            throw new IllegalArgumentException(
                    "Cannot convert " + code + " to " + type.getSimpleName() + " by ordinal value.", ex);
        }
    }

    protected E getEnumByValue(Integer code) {
        return enumsMap.get(code);
    }

    @Override
    public E getNullableResult(ResultSet resultSet, int columnIndex) throws SQLException {
        if (resultSet.wasNull()) {
            return null;
        }
        int code = resultSet.getInt(columnIndex);
        return getEnum(code);
    }

    @Override
    public E getNullableResult(CallableStatement callableStatement, int columnIndex) throws SQLException {
        if (callableStatement.wasNull()) {
            return null;
        }
        int code = callableStatement.getInt(columnIndex);
        return getEnum(code);
    }
}

@MappedTypes(value = {AgreementType.class})
@MappedJdbcTypes(value = {JdbcType.INTEGER,JdbcType.TINYINT,JdbcType.SMALLINT})
public class AgreementTypeEnumTypeHandler extends BizEnumTypeHandler<AgreementType>{
    public AgreementTypeEnumTypeHandler() {
        super(AgreementType.class);
    }
}
```


| 类型处理器                   | Java 类型                       | JDBC 类型                                                                  |
| ------------------------------ | --------------------------------- | ---------------------------------------------------------------------------- |
| `BooleanTypeHandler`         | `java.lang.Boolean`, `boolean`  | 数据库兼容的`BOOLEAN`                                                      |
| `ByteTypeHandler`            | `java.lang.Byte`, `byte`        | 数据库兼容的`NUMERIC` 或 `BYTE`                                            |
| `ShortTypeHandler`           | `java.lang.Short`, `short`      | 数据库兼容的`NUMERIC` 或 `SMALLINT`                                        |
| `IntegerTypeHandler`         | `java.lang.Integer`, `int`      | 数据库兼容的`NUMERIC` 或 `INTEGER`                                         |
| `LongTypeHandler`            | `java.lang.Long`, `long`        | 数据库兼容的`NUMERIC` 或 `BIGINT`                                          |
| `FloatTypeHandler`           | `java.lang.Float`, `float`      | 数据库兼容的`NUMERIC` 或 `FLOAT`                                           |
| `DoubleTypeHandler`          | `java.lang.Double`, `double`    | 数据库兼容的`NUMERIC` 或 `DOUBLE`                                          |
| `BigDecimalTypeHandler`      | `java.math.BigDecimal`          | 数据库兼容的`NUMERIC` 或 `DECIMAL`                                         |
| `StringTypeHandler`          | `java.lang.String`              | `CHAR`, `VARCHAR`                                                          |
| `ClobReaderTypeHandler`      | `java.io.Reader`                | -                                                                          |
| `ClobTypeHandler`            | `java.lang.String`              | `CLOB`, `LONGVARCHAR`                                                      |
| `NStringTypeHandler`         | `java.lang.String`              | `NVARCHAR`, `NCHAR`                                                        |
| `NClobTypeHandler`           | `java.lang.String`              | `NCLOB`                                                                    |
| `BlobInputStreamTypeHandler` | `java.io.InputStream`           | -                                                                          |
| `ByteArrayTypeHandler`       | `byte[]`                        | 数据库兼容的字节流类型                                                     |
| `BlobTypeHandler`            | `byte[]`                        | `BLOB`, `LONGVARBINARY`                                                    |
| `DateTypeHandler`            | `java.util.Date`                | `TIMESTAMP`                                                                |
| `DateOnlyTypeHandler`        | `java.util.Date`                | `DATE`                                                                     |
| `TimeOnlyTypeHandler`        | `java.util.Date`                | `TIME`                                                                     |
| `SqlTimestampTypeHandler`    | `java.sql.Timestamp`            | `TIMESTAMP`                                                                |
| `SqlDateTypeHandler`         | `java.sql.Date`                 | `DATE`                                                                     |
| `SqlTimeTypeHandler`         | `java.sql.Time`                 | `TIME`                                                                     |
| `ObjectTypeHandler`          | Any                             | `OTHER` 或未指定类型                                                       |
| `EnumTypeHandler`            | Enumeration Type                | VARCHAR 或任何兼容的字符串类型，用来存储枚举的名称（而不是索引序数值）     |
| `EnumOrdinalTypeHandler`     | Enumeration Type                | 任何兼容的`NUMERIC` 或 `DOUBLE` 类型，用来存储枚举的序数值（而不是名称）。 |
| `SqlxmlTypeHandler`          | `java.lang.String`              | `SQLXML`                                                                   |
| `InstantTypeHandler`         | `java.time.Instant`             | `TIMESTAMP`                                                                |
| `LocalDateTimeTypeHandler`   | `java.time.LocalDateTime`       | `TIMESTAMP`                                                                |
| `LocalDateTypeHandler`       | `java.time.LocalDate`           | `DATE`                                                                     |
| `LocalTimeTypeHandler`       | `java.time.LocalTime`           | `TIME`                                                                     |
| `OffsetDateTimeTypeHandler`  | `java.time.OffsetDateTime`      | `TIMESTAMP`                                                                |
| `OffsetTimeTypeHandler`      | `java.time.OffsetTime`          | `TIME`                                                                     |
| `ZonedDateTimeTypeHandler`   | `java.time.ZonedDateTime`       | `TIMESTAMP`                                                                |
| `YearTypeHandler`            | `java.time.Year`                | `INTEGER`                                                                  |
| `MonthTypeHandler`           | `java.time.Month`               | `INTEGER`                                                                  |
| `YearMonthTypeHandler`       | `java.time.YearMonth`           | `VARCHAR` 或 `LONGVARCHAR`                                                 |
| `JapaneseDateTypeHandler`    | `java.time.chrono.JapaneseDate` | `DATE`                                                                     |

#### ObjectFactory

每次 MyBatis 创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成实例化工作。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法

#### Plugins

实现 `org.apache.ibatis.plugin.Interceptor` 接口，并使用注解：

```java
@Intercepts(
        {
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
        }
)
public class PageInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
    ...
    }
    @Override
    public Object plugin(Object target) {
    ...
    }
    @Override
    public void setProperties(Properties properties) {
    ...
    }
}
```

1. type：表示拦截的节点，Executor、ParameterHandler、ResultSetHandler、StatementHandler。
2. method：节点中调用的方法。

* Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
* ParameterHandler (getParameterObject, setParameters)
* ResultSetHandler (handleResultSets, handleOutputParameters)
* StatementHandler (prepare, parameterize, batch, update, query)

3. arg：调用方法对应的参数,可以查询源码中对应的类查找.

最后加入到配置文件：

```xml
<plugins>
    <plugin interceptor="org.format.mybatis.cache.interceptor.ExamplePlugin"></plugin>
</plugins>
```

#### Mappers

```xml
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
# 或在 aplication.properties 中配置
mybatis.mapper-locations=classpath*:com/open/**/dao/*.xml
```

##### Cache

自定义缓存可以实现 org.apache.ibatis.cache.Cache 。

属性：

* `type`：cache使用的类型，默认是`PerpetualCache`。
* `eviction`： 定义回收的策略，常见的有FIFO，LRU。
* `flushInterval`： 配置一定时间自动刷新缓存，单位是毫秒。
* `size`： 最多缓存对象的个数。
* `readOnly`： 默认配置可读写(false)，则需要对应的实体类能够序列化，是通过序列化返回缓存对象的拷贝，若配置只读(true)，会直接返回缓存中的实例，当更改该实例时会造成缓存修改，所以该实例禁止修改。
* `blocking`： 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。

多命名空间共享相同的缓存配置和实例：`<cache-ref namespace="com.someone.application.data.SomeMapper"/>`

更新缓存的默认配置，不需要设置：

```xml
<select ... flushCache="false" useCache="true"/>
<insert ... flushCache="true"/>
<update ... flushCache="true"/>
<delete ... flushCache="true"/>
```

##### Select


| 属性          | 描述                                                                                                                                                                                                      |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id            | 在命名空间中唯一的标识符，可以被用来引用这条语句。                                                                                                                                                        |
| parameterType | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。                                             |
| resultType    | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。                                    |
| resultMap     | 对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。                                         |
| flushCache    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。                                                                                                                     |
| useCache      | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。                                                                                                             |
| timeout       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。                                                                                               |
| fetchSize     | 这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。                                                                                        |
| statementType | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。                                                                       |
| resultSetType | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。                                                                               |
| databaseId    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。                                                |
| resultOrdered | 这个设置仅针对嵌套结果 select 语句：如果为 true，将会假设包含了嵌套结果集或是分组，当返回一个主结果行时，就不会产生对前面结果集的引用。 这就使得在获取嵌套结果集的时候不至于内存不够用。默认值：`false`。 |
| resultSets    | 这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。                                                                                          |

##### Insert、Update、Delete


| 属性             | 描述                                                                                                                                                                                                                      |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id               | 在命名空间中唯一的标识符，可以被用来引用这条语句。                                                                                                                                                                        |
| parameterType    | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。                                                             |
| flushCache       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。                                                                                                  |
| timeout          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。                                                                                                               |
| statementType    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。                                                                                       |
| useGeneratedKeys | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。                      |
| keyProperty      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| keyColumn        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。                                    |
| databaseId       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。                                                                |

##### Sql

可重用的代码片段，例如：

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>

# 在 include 元素的 refid 属性或内部语句中使用属性值例如：
<sql id="sometable">
  ${prefix}Table
</sql>

<sql id="someinclude">
  from
    <include refid="${include_target}"/>
</sql>

<select id="select" resultType="map">
  select
    field1, field2, field3
  <include refid="someinclude">
    <property name="prefix" value="Some"/>
    <property name="include_target" value="sometable"/>
  </include>
</select>
```

###### 动态SQL

* if：test使用 OGNL 表达式，property ！= null and property != '' 用于字符串判断不为空。

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <if test="title != null">
      AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
      AND author_name like #{author.name}
    </if>
  </select>
  ```

  常用 OGNL ：

  e1 or e2
  e1 and e2
  e1 == e2,e1 eq e2
  e1 != e2,e1 neq e2
  e1 lt e2
  e1 lte e2
  e1 in e2
  e1 not in e2
  e1 + e2,e1 * e2,e1/e2,e1 - e2,e1%e2
  !e,not e
  e.method(args)：调用对象方法
  e.property：对象属性值
  e1[ e2 ]：按索引取值，List,数组和Map
  @class@method(args)：调用类的静态方法
  @class@field：调用类的静态字段值
* choose、when、otherwise：相当于switch

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <choose>
      <when test="title != null">
        AND title like #{title}
      </when>
      <when test="author != null and author.name != null">
        AND author_name like #{author.name}
      </when>
      <otherwise>
        AND featured = 1
      </otherwise>
    </choose>
  </select>
  ```
* where：若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

  ```xml
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG
    <where>
      <if test="state != null">
           state = #{state}
      </if>
      <if test="title != null">
          AND title like #{title}
      </if>
      <if test="author != null and author.name != null">
          AND author_name like #{author.name}
      </if>
    </where>
  </select>
  ```
* trim：

  属性：

  * prefix：会将该值放在 `<trim>`元素内容 前面。
  * prefixOverrides：会将 `<trim>`元素内容 前面等于 该值的内容 替换为空字符。
  * suffix：会将该值放在 `<trim>`元素内容 后面。
  * suffixOverrides：会将 `<trim>`元素内容 后面等于 该值的内容 替换为空字符。

  ```xml
  <select id="getUserList3" resultMap="RESULT_MAP">
     select `user_id`, `name`, `age`, `sex` from `user`
     <trim prefix="where" suffixOverrides="and">
        <choose>
           <when test="username != null and username != ''">
              `name` like #{username} and
           </when>
           <when test="age != null">
              `age` >= #{age} and
           </when>
           <otherwise>
              `user_id` is not null and
           </otherwise>
        </choose>
     </trim>
  </select>
  ```
* set：用于update语句，会自动删掉 最后额外的逗号

  ```xml
  <update id="updateAuthorIfNecessary">
    update Author
      <set>
        <if test="username != null">username=#{username},</if>
        <if test="password != null">password=#{password},</if>
        <if test="email != null">email=#{email},</if>
        <if test="bio != null">bio=#{bio}</if>
      </set>
    where id=#{id}
  </update>
  ```
* foreach：用于循环。

  属性：

  * collection：循环的对象。
  * item：每个元素。
  * index：循环下标。
  * open：开始字符。
  * separator：分隔符。
  * close：结束字符。

  ```xml
  <select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM POST P
    <where>
      <foreach item="item" index="index" collection="list"
          open="ID in (" separator="," close=")" nullable="true">
            #{item}
      </foreach>
    </where>
  </select>
  ```
* bind：使用 OGNL 表达式创建一个变量。

  ```xml
  <select id="selectBlogsLike" resultType="Blog">
    <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
    SELECT * FROM BLOG
    WHERE title LIKE #{pattern}
  </select>
  ```
* 多数据支持

  ```xml
  <insert id="insert">
    <selectKey keyProperty="id" resultType="int" order="BEFORE">
      <if test="_databaseId == 'oracle'">
        select seq_users.nextval from dual
      </if>
      <if test="_databaseId == 'db2'">
        select nextval for seq_users from sysibm.sysdummy1"
      </if>
    </selectKey>
    insert into users values (#{id}, #{name})
  </insert>
  ```

##### 参数

#{value}：会使用 PreparedStatement 的占位符 ? 传入，例如：#{property,javaType=int,jdbcType=NUMERIC}。

${value}：会直接替换，value值可以是 OGNL 表达式。

##### ResultMap

如果 select 返回值中字段与 javabean 属性一样，可使用 ResultType 来配置，在框架底层会自动生成 ResultMap 来映射，例如：

```xml
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

默认情况下，配置文件中 autoMappingBehavior 使用默认值`PARTIAL`，会自动映射结果，如果`mapUnderscoreToCamelCase`设置为true，还可以按照数据库`_`分隔，javabean 驼峰进行映射。

属性：

* id：当前命名空间中的唯一标识。
* type：ResultMap 对应的 javabean，类的完全限定名 或者 一个类型别名，例如：在配置文件配置`<typeAliases><package name="domain.blog"/></typeAliases>`。
* autoMapping：定义 ResultMap 是否自动映射结果，会覆盖全局的 autoMappingBehavior ，默认未设置。
* extends：继承其他 ResultMap 属性。

子标签：

1. id & result：配置映射关系。
   属性：

   * property：javabean 中对应的属性。
   * column：数据库的列名或者别名。
   * javaType：javabean 的类名。
   * jdbcType：JDBC 的类型。
   * typeHandler：覆盖默认的类型。
2. constructor：不使用属性的set方法填充属性，可用来创建不可变类。

   子标签：

   * idArg：ID参数，和 id & result 属性一样。
   * arg：普通参数，和 id & result 属性一样。
3. association：嵌套映射另外一个类，一对一。

   子标签：id & result
   属性：

   * property：javabean 中对应的属性。
   * javaType：该属性对应的类名。

     嵌套结果映射：映射 leftjoin 结果
   * resultMap：可以直接引用另外一个resultMap。
   * columnPrefix: 可以给resultMap中每个column加个前缀。

     嵌套select查询：将查询结果作为参数进行第二次查询
   * select：将 column 属性作为参数传递给该select，多个参数可以使用`column="{prop1=col1,prop2=col2}"`，会造成n+1问题，不建议使用
   * column：和 select 一起使用。
   * fetchType：和 select 一起使用，有效值为 `lazy` 和 `eager`。 指定属性后，将在映射中忽略全局配置参数 `lazyLoadingEnabled`

     关联多个结果集：返回多个结果集，在结果集中进行关联
   * resultSet：为每个结果集指定一个名字，如：resultSets="blogs,authors"
   * column：父类型，与 子类型对应。
   * foreignColumn：子类型，例如：

     ```xml
     <select id="selectBlog" resultSets="blogs,authors" resultMap="blogResult">
       SELECT * FROM BLOG WHERE ID = #{id};
       SELECT * FROM AUTHOR WHERE ID = #{id};
     </select>

     <resultMap id="blogResult" type="Blog">
       <id property="id" column="id" />
       <result property="title" column="title"/>
       <association property="author" javaType="Author" resultSet="authors" column="author_id" foreignColumn="id">
         <id property="id" column="id"/>
         <result property="username" column="username"/>
         <result property="password" column="password"/>
         <result property="email" column="email"/>
         <result property="bio" column="bio"/>
       </association>
     </resultMap>
     ```
4. collection：嵌套另外一个集合，一对多。

   子标签：id & result
   属性：

   * property：javabean 中对应的属性。
   * javaType：该属性对应的集合名，例如 ArrayList。
   * ofType：集合中对象名，例如 集合为 `List<Post>` ,该值为 Post。

     嵌套结果映射：映射 leftjoin 结果
   * resultMap：可以直接引用另外一个resultMap。
   * columnPrefix: 可以给resultMap中每个column加个前缀。

     嵌套select查询：将查询结果作为参数进行第二次查询
   * select：将 column 属性作为参数传递给该select，多个参数可以使用`column="{prop1=col1,prop2=col2}"`，会造成n+1问题，不建议使用
   * column：和 select 一起使用。

     关联多个结果集：返回多个结果集，在结果集中进行关联
   * resultSet：为每个结果集指定一个名字，如：resultSets="blogs,authors"
   * column：父类型，与 子类型对应。
   * foreignColumn：子类型，例如：

     ```xml
     <select id="selectBlog" resultSets="blogs,authors" resultMap="blogResult" statementType="CALLABLE">
       SELECT * FROM BLOG WHERE ID = #{id};
       SELECT * FROM AUTHOR WHERE ID = #{id};
     </select>

     <resultMap id="blogResult" type="Blog">
       <id property="id" column="id" />
       <result property="title" column="title"/>
       <collection property="posts" ofType="Post" resultSet="posts" column="id" foreignColumn="blog_id">
         <id property="id" column="id"/>
         <result property="subject" column="subject"/>
         <result property="body" column="body"/>
       </collection>
     </resultMap>
     ```
5. discriminator：类似于switch语句
   子标签：case ，如果不能匹配任何一个 case，只会使用 discriminator 以外的映射，discriminator 的结果映射将被忽略。
   属性：

   * javaType：该属性对应的类名。
   * column：switch的字段，例如：

     ```xml
     resultMap id="vehicleResult" type="Vehicle">
       <id property="id" column="id" />
       <result property="vin" column="vin"/>
       <result property="year" column="year"/>
       <result property="make" column="make"/>
       <result property="model" column="model"/>
       <result property="color" column="color"/>
       <discriminator javaType="int" column="vehicle_type">
         <case value="1" resultMap="carResult"/>
         <case value="2" resultMap="truckResult"/>
         <case value="3" resultMap="vanResult"/>
         <case value="4" resultMap="suvResult"/>
       </discriminator>
     </resultMap>
     ```

### 基本使用要点

1. Mapper 接口和 XML文件关联时，命名空间 namespace 的值需要配置成接口的全限定名称，例如:

```java
package tk.mybatis.simple.mapper;
@Mapper
public interface UserMapper{
}
```

```xml
<mapper namespace="tk.mybatis.simple.mapper.UserMapper">
</mapper>
```
