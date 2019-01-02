# MybatisCodeTree
Mybatis核心源码技术研究


<pre>
          
          Mybatis的持久化解决方案是将用户从原始的JDBC访问中解放出来，用户只需要定义需要
      操作的SQL语句，无需关注底层的JDBC操作，就可以以面向对象的方式来进行持久化层操作，
      底层数据库连接的获取，数据访问的实现，事务控制等都无需用户关心，从而将应用层从底层
      的JDBC/JTA API抽取出来，通过配置文件管理JDBC连接，让Mybatis解决持久化的实现.
</pre>

<pre>
SqlSessionFactory

          SqlSessionFactory是Mybatis的关键对象，它是个单个数据库映射关系经过编译后的
      内存镜像。

          每一个MyBatis的应用程序都以一个SqlSessionFactory对象的实例为核心.同时
      SqlSessionFactory也是线程安全的,SqlSessionFactory一旦被创建,应该在应用执行期间
      都存在.在应用运行期间不要重复创建多次,建议使用单例模式.SqlSessionFactory是创
      建SqlSession的工厂

          public interface SqlSessionFactory {
			    SqlSession openSession();
			
			    SqlSession openSession(boolean var1);
			
			    SqlSession openSession(Connection var1);
			
			    SqlSession openSession(TransactionIsolationLevel var1);
			
			    SqlSession openSession(ExecutorType var1);
			
			    SqlSession openSession(ExecutorType var1, boolean var2);
			
			    SqlSession openSession(ExecutorType var1, TransactionIsolationLevel var2);
			
			    SqlSession openSession(ExecutorType var1, Connection var2);
			
			    Configuration getConfiguration();
		}
</pre>

<pre>
SqlSession

          SqlSession是Mybatis的关键对象，是进行持久化操作的独享，类似于JDBC中的
      Connection。它是应用程序与持久层之间执行交互操作的一个单线程对象，也是Mybatis
      执行持久化操作的关键对象，SqlSession对象完全包含以数据库为背景的所有执行SQL
      操作的方法，它的底层封装了JDBC连接，可以用SqlSession实例来直接执行被映射的
      SQL语句，每个线程都应该有它自己的SqlSession实例，SqlSession的实例不能被共享，
      同时SqlSession也是线程不安全的，绝对不能将SqlSession实例的引用放在一个类的
      静态字段甚至是实例字段中，也绝对不能将SqlSession实例的引用放在任何类型的管理
      范围中，比如Servlet当中的HttpSession对象中，使用完SqlSession之后关闭
      Session很重要，应该确保使用finally块来关闭它。

      public interface SqlSession extends Closeable {

		  <T> T selectOne(String statement);
		
		  <T> T selectOne(String statement, Object parameter);
		
		  <E> List<E> selectList(String statement);
		
		  <E> List<E> selectList(String statement, Object parameter);
		
		  <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds);
		
		  <K, V> Map<K, V> selectMap(String statement, String mapKey);
		
		  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey);
		
		  <K, V> Map<K, V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowBounds);
		
		  void select(String statement, Object parameter, ResultHandler handler);
		
		  void select(String statement, ResultHandler handler);
		
		  void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler);
		
		  int insert(String statement);
		
		  int insert(String statement, Object parameter);
		
		  int update(String statement);
		
		  int update(String statement, Object parameter);
		
		  int delete(String statement);
		
		  int delete(String statement, Object parameter);
		
		  void commit();
		
		  void commit(boolean force);
		
		  void rollback();
		
		  void rollback(boolean force);
		
		  List<BatchResult> flushStatements();
		
		  void close();
		
		  void clearCache();
		
		  Configuration getConfiguration();
		
		  <T> T getMapper(Class<T> type);
		
		  Connection getConnection();
	 }
</pre>

<pre>
SqlSessionFactory / SqlSession实现过程
     
      Mybatis框架主要是围绕着SqlSessionFactory进行的，创建过程大概如下：
 
          1）定义一个Configuration对象，其中包含数据源，事务，mapper文件资源以及影响
             数据库行为属性的设置settings
          2）通过配置对象，则可创建一个SqlSessionFactoryBuilder对象。
          3）通过SqlSessionFactoryBuilder获取SqlSessionFactory的实例。
          5）SqlSessionFactory的实例可以获得操作数据的SqlSession实例，通过这个实例对数
             据库进行操作。
</pre>

![](https://i.imgur.com/zl1Idg8.png)

![](https://i.imgur.com/MPMF9Vb.png)

<pre>
MapperScannerConfigurer

      MapperScannerConfigurer是Spring和Mybatis整合的mybatis-spring.jar包中提供的一个类。

      MapperFactoryBean的出现为了代替手工使用SqlSessionDaoSupport或SqlSessionTemplate编写数据访问对象(DAO)的代码,使用动态代理实现。

      官方文档中的配置：

		org.mybatis.spring.sample.mapper.UserMapper是一个接口，我们创建一
        个MapperFactoryBean实例，然后注入这个接口和sqlSessionFactory（mybatis中提供
        的SqlSessionFactory接口，MapperFactoryBean会使用SqlSessionFactory创建SqlSession）
        这两个属性。
		
		之后想使用这个UserMapper接口的话，直接通过spring注入这个bean，然后就可以直接使用了，
        spring内部会创建一个这个接口的动态代理。
		
		当发现要使用多个MapperFactoryBean的时候，一个一个定义肯定非常麻烦，于是mybatis-spring
        提供了MapperScannerConfigurer这个类，它将会查找类路径下的映射器并自动将它们创建
        成MapperFactoryBean。
		
		这段配置会扫描org.mybatis.spring.sample.mapper下的所有接口，然后创建各自接口的动态代
        理类。
</pre>

Spring与Mybatis整合的MapperScannerConfigurer处理过程源码分析
https://www.cnblogs.com/fangjian0423/p/spring-mybatis-MapperScannerConfigurer-analysis.html

![](https://i.imgur.com/1y7Ii7f.png)

<pre>
Mybatis主要成员

      1) configuration
         Mybatis所有的配置信息都保存在Configuration对象中，配置文件中的大部分类都会存储
         在该类对象中。
      2）Sqlsession
         作为Mybatis工作的主要顶层API，标识和数据库交互时的会话，完成必要数据库增删改查功能。
      3）Executor
         Mybatis执行器，是Mybatis调度的核心，负责SQL语句的生成和查询缓存的维护。
      5）StatementHandler
         封装JDBC statement操作，负责对JDBC statement的操作，如设置参数等。
      6）ParameterHandler
         负责对用户传递的参数转换成jdbc statement所对应的数据类型
      7）ResultSetHandler
         负责将jdbc返回的ResultSet结果集对象转换成List类型的集合
      8）TypeHandler
         负责java数据类型和jdbc数据类型之间的映射和转换
      9）MappedStatement
         MappedStatement维护一条(select|insert|update|delete|query)节点的封装
      10）sqlsource
         负责根据用户传递的parameterObject，动态生成sql语句，将信息封装到BoundSql对象中，
         并返回
      11）BoundSql
         标识动态生成的sql语句以及相应的参数信心
</pre>

###Mybatis缓存

![](https://i.imgur.com/k2G3iL3.png)

<pre>
一级缓存是SqlSession级别的缓存，每个SqlSession对象都有一个哈希表用于缓存数据，不同
SqlSession对象之间缓存不共享。同一个SqlSession对象对象执行2遍相同的SQL查询，在第一次查询执
行完毕后将结果缓存起来，这样第二遍查询就不用向数据库查询了，直接返回缓存结果即可。MyBatis默认
是开启一级缓存的。

二级缓存是mapper级别的缓存，二级缓存是跨SqlSession的，多个SqlSession对象可以共享同一个二级
缓存。不同的SqlSession对象执行两次相同的SQL语句，第一次会将查询结果进行缓存，第二次查询直接返
回二级缓存中的结果即可。MyBatis默认是不开启二级缓存的，可以在配置文件中使用如下配置来开启二级
缓存：
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>

当SQL语句进行更新操作(删除/添加/更新)时，会清空对应的缓存，保证缓存中存储的都是最新的数据。
MyBatis的二级缓存对细粒度的数据级别的缓存实现不友好，比如如下需求：对商品信息进行缓存，由于商
品信息查询访问量大，但是要求用户每次都能查询最新的商品信息，此时如果使用mybatis的二级缓存就无
法实现当一个商品变化时只刷新该商品的缓存信息而不刷新其它商品的信息，因为mybaits的二级缓存区域
以mapper为单位划分，当一个商品信息变化会将所有商品信息的缓存数据全部清空。解决此类问题需要在业
务层根据需求对数据有针对性缓存，具体业务具体实现。
</pre>

MyBatis框架及原理分析

https://www.cnblogs.com/luoxn28/p/6417892.html