# HikariCP

## 常用配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://1.12.236.101:3306/jpa_test?serverTimezone=GMT%2B8&characterEncoding=utf8&useSSL=false&useUnicode=true
    username: root
    password: 21qaz@!QAZ
    hikari:
      # 是否自动提交,默认为 True
      auto-commit: true
      # 客户端创建连接等待超时时间,如果时间内没有获取连接则抛异常,默认 30 秒
      connection-timeout: 30000
      # 连接允许最长空闲时间,如果超时,则会被关闭，默认为 600 秒
      # 如果idleTimeout+1秒>maxLifetime 且 maxLifetime>0，则会被重置为0
      # 如果idleTimeout=0则表示空闲的连接在连接池中永远不被移除
      # 由定时线程每隔30秒检测一次，实际移除的时间时可能是idleTimeout+30秒，平均idleTimeout+15秒
      # idle-timeout: 600000

      # 会按照该时间间隔发送PING给数据库，JDBC4以上直接发送二进制数据，或者JDBC4以下发送connectionTestQuery
      # 防止它被数据库或网络基础设施超时，默认值为：0（禁用）
      keepalive-time: 0
      # 当达到这个时间后连接会被标记被移除，如果连接没有被使用会被删除。
      # 如果连接正在使用会在使用完成后被其他线程获取时被删除
      # 值为 0 表示没有最大生命周期，默认值：1800000（30 分钟）
      # 该值需要设置为比任何数据库或基础设施强加的连接时间限制短几秒。
      # 设置为0或过大，可能导致 no operations allowed after connection closed 异常。
      max-lifetime: 1800000
      # 该值不要设置
      # connectionTestQuery
      # 该值不要设置，默认与 maximumPoolSize 保持一致
      # minimumIdle
      # 连接池达到的最大大小，包括空闲和使用中的连接，默认值：10。
      maximumPoolSize: 20
```

maximumPoolSize = ((core_count * 2)+ effective_spindle_count),effective_spindle_count为磁盘阵列的硬盘数。
考虑整体系统性能，考虑线程执行需要的等待时间，设计合理的线程数目。但是，不要过度配置数据库。

增大连接池大小可以缓解池锁问题，**但是扩大池之前是可以先检查一下应用层面能够调优，不要直接调整连接池大小**。
pool size = Tn x (Cm - 1) + 1，T n是线程的最大数量，C m是单个线程持有的同时连接的最大数量，这是避免池锁的最低限度。

## 核心组件

### HikariDataSource

实现 DataSource 标准接口，作为连接池的门面，负责校验配置参数，在初始化或者第一次获取连接时创建连接池。

```java
public class HikariDataSource extends HikariConfig implements DataSource, Closeable
{
   private static final Logger LOGGER = LoggerFactory.getLogger(HikariDataSource.class);

   private final AtomicBoolean isShutdown = new AtomicBoolean();

   private final HikariPool fastPathPool;
   private volatile HikariPool pool;

   public HikariDataSource()
   {
      super();
      fastPathPool = null;
   }

   public HikariDataSource(HikariConfig configuration)
   {
      configuration.validate();
      configuration.copyStateTo(this);
      //初始化连接池
      pool = fastPathPool = new HikariPool(this);
   }

  @Override
   public Connection getConnection() throws SQLException
   {
      if (isClosed()) {
         throw new SQLException("HikariDataSource " + this + " has been closed.");
      }

      if (fastPathPool != null) {
         return fastPathPool.getConnection();
      }
      //初始化连接池
      HikariPool result = pool;
      if (result == null) {
         synchronized (this) {
            result = pool;
            if (result == null) {
               validate();
               LOGGER.info("{} - Starting...", getPoolName());
               try {
                  pool = result = new HikariPool(this);
                  this.seal();
               }
               catch (PoolInitializationException pie) {
                  if (pie.getCause() instanceof SQLException) {
                     throw (SQLException) pie.getCause();
                  }
                  else {
                     throw pie;
                  }
               }
               LOGGER.info("{} - Start completed.", getPoolName());
            }
         }
      }

      return result.getConnection();
   }
}
```

## PoolBase

HikariPool 的父类，提供与 Connection 相关的基础方法，主要以下几个方面：
1. 初始化原生DataSource，可能是 DriverDataSource、JNDI、其他 dataSourceClassName（通过HikariConfig配置），可使用getUnwrappedDataSource()访问。
2. 通过DataSource创建Connection，修饰符为private，外部不能访问。
3. 判断Connection是否存活。
4. 关闭Connection。
5. 创建 PoolEntry，修饰符为空，提供给子类访问。PoolEntry 是 Connection 的一对一封装。
6. 定义回收PoolEntry的抽象方法。

```java
abstract class PoolBase
{
    // 三种初始化数据源的方式
    private void initializeDataSource()
    {
        final String jdbcUrl = config.getJdbcUrl();
        final String username = config.getUsername();
        final String password = config.getPassword();
        final String dsClassName = config.getDataSourceClassName();
        final String driverClassName = config.getDriverClassName();
        final String dataSourceJNDI = config.getDataSourceJNDI();
        final Properties dataSourceProperties = config.getDataSourceProperties();

        DataSource ds = config.getDataSource();
        if (dsClassName != null && ds == null) {
            ds = createInstance(dsClassName, DataSource.class);
            PropertyElf.setTargetFromProperties(ds, dataSourceProperties);
        }
        else if (jdbcUrl != null && ds == null) {
            ds = new DriverDataSource(jdbcUrl, driverClassName, dataSourceProperties, username, password);
        }
        else if (dataSourceJNDI != null && ds == null) {
            try {
                InitialContext ic = new InitialContext();
                ds = (DataSource) ic.lookup(dataSourceJNDI);
            } catch (NamingException e) {
                throw new PoolInitializationException(e);
            }
        }

        if (ds != null) {
            setLoginTimeout(ds);
            createNetworkTimeoutExecutor(ds, dsClassName, jdbcUrl);
        }

        this.dataSource = ds;
    }

    // 创建PoolEntry
    PoolEntry newPoolEntry() throws Exception
    {
        return new PoolEntry(newConnection(), this, isReadOnly, isAutoCommit);
    }
}
```

## HikariPool

负责对连接资源(即PoolEntry)进行管理，
