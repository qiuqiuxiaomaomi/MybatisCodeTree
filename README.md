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