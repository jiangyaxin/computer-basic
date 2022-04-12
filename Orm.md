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
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。                                                                                                         | SIMPLE REUSE BATCH                              | SIMPLE                                                |
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

### ResultMap

如果 select 返回值中字段与 javabean 属性一样，可使用 ResultType 来配置，在框架底层会自动生成 ResultMap 来映射，例如：

```xml
<select id="selectUsers" resultType="com.someapp.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

属性：

* id：当前命名空间中的唯一标识。
* type：ResultMap 对应的 javabean，类的完全限定名 或者 一个类型别名，例如：在配置文件配置`<typeAliases><package name="domain.blog"/></typeAliases>`。
* autoMapping：定义 ResultMap 是否自动映射结果，会覆盖全局的 autoMappingBehavior ，默认未设置。

子标签：

1. id & result：配置映射关系。
   属性：
   * property：javabean 中对应的属性。
   * column：数据库的列名或者别名。
   * javaType：javabean 的类名。
   * jdbcType：JDBC 的类型。
   * typeHandler：覆盖默认的类型。

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
