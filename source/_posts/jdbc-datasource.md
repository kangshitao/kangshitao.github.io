---
title: JDBC与Druid数据库连接池
excerpt: JDBC简介，Druid数据库连接池、Apache-DBUtils工具类的用法
mathjax: true
date: 2021-04-30 15:58:59
tags: ['数据库','MySQL','SQL','Java']
categories: 数据库
keywords: 数据库,MySQL,JDBC,数据库连接池,Druid,DBUtils
---

# 一、传统数据库连接

## 1、JDBC简介

**JDBC**(Java Database Connectivity)是**独立于特定数据库管理系统、通用的SQL数据库存取和操作的公共接口**（一组API），定义了用来访问数据库的标准Java类库（**java.sql,javax.sql**），使用这些类库可以以一种**标准**的方法、方便地访问数据库资源。

开发人员只需要**面向JDBC的接口编程**即可，不同数据库厂商各自需要实现JDBC接口，即为不同的数据库驱动。

使用JDBC连接并操作数据库的步骤：

* 导入`java.sql`和所使用的数据库的驱动，即`.jar`包
* 加载并注册驱动程序
* 创建`Connection`对象，建立连接
* 创建`Statement`或`PreparedStatement`对象
* 执行SQL语句，实现CRUD操作，如果是查询操作，会返回结果集`ResultSet`并需要处理。
* 关闭`Statement`/`PreparedStatement`对象和`Connection`连接，如果有`ResultSet`，也需要关闭。



## 2、数据库连接

`java.sql.Driver`接口是所有JDBC驱动程序需要实现的接口，数据库厂商需要实现此接口。程序中使用驱动程序管理器类(`java.sql.DriverManager`)调用这些`Driver`实现（即注册驱动）。

创建连接的步骤：

* **加载并读取配置相关信息**，如数据库用户名，密码，驱动的类名，驱动URL

* **加载驱动**，即实例化Driver，一般不主动实例化，而是使用反射的方式，根据加载的JDBC驱动的类名，加载驱动。如`Class.forName(“com.mysql.jdbc.Driver”);`

* **注册驱动**:

  * 方式一，使用`DriverManager.registerDriver(com.mysql.jdbc.Driver)`注册驱动.

  * 方式二，通常不手动注册，因为 `Driver `接口的驱动程序类**都**包含了静态代码块，在这个静态代码块中，会调用 `DriverManager.registerDriver() `方法来注册自身的一个实例。比如MySQL的`Driver`实现类的源码如下

    ```java
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    ```

    

* **获取连接**。调用`DriverManager` 类的静态 `getConnection()` 方法建立到数据库的连接。

>url用于标识一个被注册的驱动程序，驱动程序管理器通过这个 URL 选择正确的驱动程序，从而建立到数据库的连接。JDBC URL由三部分组成，即`jdbc:子协议:子名称`，子名称用于定位具体的数据库，包含主机名、端口号、数据库名。比如对于MySQL来说，其URL编写格式为：jdbc:mysql://[主机名称:mysql的服务端口号]/数据库名称[?参数=值&参数=值]，例如test数据库，可以写为：`jdbc:mysql://localhost:3306/test?characterEncoding=utf-8`，如果有编码问题，可以设置连接时的参数。



使用`DriverManager` 获取数据库连接举例：

```java
public  void testConnection() throws Exception {
    //1.加载配置文件
    InputStream is = ConnectionTest.class.getClassLoader().
        getResourceAsStream("jdbc.properties");
    roperties pros = new Properties();
    pros.load(is);
    
    //2.读取配置信息
    String user = pros.getProperty("user");
    String password = pros.getProperty("password");
    String url = pros.getProperty("url");
    String driverClass = pros.getProperty("driverClass");

    //3.加载驱动(包含了实例化Driver和注册驱动两步)
    Class.forName(driverClass);
    /*
    上述步骤通过反射获取Driver实现类的Class实例，目的是在此过程中调用
    对应实现类的静态代码块，代码块中实例化Driver，并注册了驱动
    */

    //4.获取连接
    Connection conn = DriverManager.getConnection(url,user,password);
    System.out.println(conn);
}
```

其中`jdbc.properties`配置文件需要在src目录下，内容如下：

```properties
user=root 
password=abc123
url=jdbc:mysql://localhost:3306/test?characterEncoding=utf-8
driverClass=com.mysql.jdbc.Driver
```

这里操作符两边不能用空格。



# 二、使用PreparedStatement实现CRUD操作

在` java.sql` 包中有 3 个接口分别定义了对数据库的调用的不同方式：

- `Statement`：用于执行静态 SQL 语句并返回它所生成结果的对象。 
- `PrepatedStatement`：SQL 语句被预编译并存储在此对象中，可以使用此对象多次高效地执行该语句。
- `CallableStatement`：用于执行 SQL 存储过程



## 1、PreparedStatement介绍

- 可以通过调用 Connection 对象的 **preparedStatement(String sql)** 方法获取 PreparedStatement 对象

- **PreparedStatement 接口是 Statement 的子接口，它表示一条预编译过的 SQL 语句**

- PreparedStatement 对象所代表的 SQL 语句中的参数用`?`来表示，调用 PreparedStatement 对象的 `setXxx()` 方法来设置这些参数，`setXxx()` 方法有两个参数，第一个参数是要设置的 SQL 语句中的参数的索引(从`1`开始)，第二个是设置的 SQL 语句中的参数的值。

  

## 2、PreparedStatement的使用

**PreparedStatement 和 Statement对比**：

- PreparedStatement是Statement的子类，二者都能实现对数据库的CRUD操作
- S‘tatement使用字符串拼接操作，繁琐。而使用PreparedStatement的代码可读性和可维护性强。
- **PreparedStatement 对SQL语句预编译**，能最大可能提高性能：
  - DBServer会对**预编译**语句提供性能优化。因为**预编译语句有可能被重复调用**，所以语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。
  - statement语句中，即使是相同操作，但因为数据内容不一样导致整个语句本身不能匹配，没有缓存语句的意义。事实是没有数据库会对普通语句编译后的执行代码缓存。这样每执行一次都要对传入的语句编译一次，效率降低。
- Statement存在SQL注入问题，PreparedStatement 使用预编译和`?`占位符传值，可以防止此问题 。
- 此外，PreparedStatement还能够实现对`Blob`数据的操作，并且对于批量插入比较高效。



举例1：使用PreparedStatement实现增、删、改操作

```java
/*
我们将上述提到的数据库连接操作，以及资源关闭操作，封装到JDBCUtils工具类中，
即以下的几个静态方法：
public static Connection getConnection()：返回一个Connection实例
public static void closeResource(Connection c,Statement ps)：关闭资源
public static void closeResource(Connection c,Statement ps,ResultSet rs)：关闭资源
*/
public class PreparedStatementTest{
    //适用于不同的表的增、删、改操作
	public void update(String sql,Object ... args){
		Connection conn = null;
		PreparedStatement ps = null;
		try {
			//1.获取数据库的连接
			conn = JDBCUtils.getConnection();
			
			//2.获取PreparedStatement的实例 (或：预编译sql语句)
			ps = conn.prepareStatement(sql);
			//3.填充占位符
			for(int i = 0;i < args.length;i++){
				ps.setObject(i + 1, args[i]);
			}
			//4.执行sql语句
			ps.execute();
		} catch (Exception e) {
			e.printStackTrace();
		}finally{
			//5.关闭资源
			JDBCUtils.closeResource(conn, ps);
		}
	}
}
```



举例2：使用PreparedStatement实现查询操作：

```java
public class prepareStatementTest{
    // 通用的针对于不同表的查询:返回一个对象
	public <T> T getInstance(Class<T> clazz, String sql, Object... args) {
		Connection conn = null;
		PreparedStatement ps = null;
		ResultSet rs = null;
		try {
			// 1.获取数据库连接
			conn = JDBCUtils.getConnection();
			// 2.预编译sql语句，得到PreparedStatement对象
			ps = conn.prepareStatement(sql);
			// 3.填充占位符
			for (int i = 0; i < args.length; i++) {
				ps.setObject(i + 1, args[i]);
			}
			// 4.执行executeQuery(),得到结果集：ResultSet
			rs = ps.executeQuery();
			// 5.得到结果集的元数据：ResultSetMetaData
			ResultSetMetaData rsmd = rs.getMetaData();
			// 6.1通过ResultSetMetaData得到columnCount,columnLabel；通过ResultSet得到列值
			int columnCount = rsmd.getColumnCount();
			if (rs.next()) {
				T t = clazz.newInstance();
				for (int i = 0; i < columnCount; i++) {// 遍历每一个列
					// 获取列值
					Object columnVal = rs.getObject(i + 1);
					// 获取列的别名:列的别名必须和类的属性相同
					String columnLabel = rsmd.getColumnLabel(i + 1);
					// 6.2使用反射，给对象的相应属性赋值
					Field field = clazz.getDeclaredField(columnLabel);
					field.setAccessible(true);
					field.set(t, columnVal);
				}
				return t;
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {// 7.关闭资源
			JDBCUtils.closeResource(conn, ps, rs);
		}
		return null;
	}
}
```



如果考虑事务操作，则不应该在方法中获取和关闭连接，需要在调用之前传入`Connection`对象，对于事务的操作可以简化为以下步骤：

```java
public void testJDBCTransaction() {
	Connection conn = null;
	try {
		// 1.获取数据库连接
		conn = JDBCUtils.getConnection();
		// 2.开启事务,关闭自动提交
		conn.setAutoCommit(false);
		// 3.进行数据库操作，这里的两个SQL操作组成一个事务
		String sql1 = "update user_table set balance = balance - 100 where user = ?";
		update(conn, sql1, "AA");  //执行事务
		// 模拟网络异常。如果有异常，事务会回滚
		//System.out.println(10 / 0);

		String sql2 = "update user_table set balance = balance + 100 where user = ?";
		update(conn, sql2, "BB");
		// 4.若没有异常，则提交事务
		conn.commit();
	} catch (Exception e) {
		e.printStackTrace();
		// 5.若有异常，则回滚事务
		try {
			conn.rollback();
		} catch (SQLException e1) {
			e1.printStackTrace();
		}
    } finally {
        try {
			//6.恢复自动提交
			conn.setAutoCommit(true);
		} catch (SQLException e) {
			e.printStackTrace();
		}
        //7.关闭连接
		JDBCUtils.closeResource(conn, null, null); 
    }  
}

//使用事务以后的方法需要改写，将连接对象作为参数传入，其余内容不变
//对于查询操作的方法同样如此。
public void update(Connection conn,String sql, Object... args) {
	PreparedStatement ps = null;
	try {
		ps = conn.prepareStatement(sql);
		for (int i = 0; i < args.length; i++) {
			ps.setObject(i + 1, args[i]);
		}
		ps.execute();
	} catch (Exception e) {
		e.printStackTrace();
	} finally {
		JDBCUtils.closeResource(null, ps);
	}
}
```

`Connection`的对象也可以调用`setTransactionIsolation()`方法设置事务的隔离级别

## 3、资源释放

执行完操作后，必须要释放资源：

* 释放`ResultSet`， `Statement`，`Connection`。
* 数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统宕机。Connection的使用原则是**尽量晚创建，早释放**。
* 可以在finally中关闭，保证及时其他代码出现异常，资源也一定能被关闭。  



## 4、总结

JDBC涉及到的两种思想和两种技术：

* 两种思想

  * 面向接口编程的思想
  * ORM思想(object relational mapping)
    * 一个数据表对应一个java类
    * 表中的一条记录对应java类的一个对象
    * 表中的一个字段对应java类的一个属性

  > SQL查询的结果集中列的别名，必须要和类的属性名相同。

* 两种技术

  * JDBC结果集的元数据：`ResultSetMetaData`，作用为
    * 获取结果集列数：`getColumnCount()`
    * 获取列的别名：`getColumnLabed()`
  * 通过反射，创建指定类的对象，获取指定的属性并赋值。





# 三、DAO的使用

**DAO（Data Access Object）**：访问数据信息的类和接口，只包括了对数据的CRUD（Create、Retrival、Update、Delete）操作，而不包含任何业务相关的信息。有时也称作：BaseDAO

作用：为了实现功能的模块化，更有利于代码的维护和升级。

使用DAO主要包括三部分：

* 将DAO定义为一个抽象类，包含了CRUD基本操作，适用于所有表。
* 对每个表，定义一个接口，接口规范了当前表的不同功能。
* 每个表对应一个实现类，此类继承DAO抽象类，并实现了此表对应的接口。

最后，我们只需要调用实现类的方法，满足不同的需求。

比如，对于`Customer`类，数据库中对应customers表，使用DAO及其相关实现类如下

 `DAO.java`文件类，定义抽象类：

```java
/*
 * DAO: data(base) access object
 * 封装了针对于数据表的通用的操作
 */
public abstract class BaseDAO<T> {
    private Class<T> clazz = null;
    //需要获取BaseDAO子类的带泛型父类的泛型，根据此泛型得知要操作的类。
    //可以在代码块或者构造器中获取。
    public BaseDAO(){
        //带泛型的父类，即BaseDAO<Customer>
        Type genericSuperclass = this.getClass().getGenericSuperclass();
        ParameterizedType paramType = (ParameterizedType) genericSuperclass;
        //获取父类的泛型，即Customer
        Type[] actualTypeArguments = paramType.getActualTypeArguments();
        clazz = (Class<T>) actualTypeArguments[0];//数组的第一个值就是Customer
    }
    
    // 通用的增删改操作
    public int update(Connection conn, String sql, Object... args) {
        PreparedStatement ps = null;
        try {
            // 1.预编译sql语句，返回PreparedStatement的实例
            ps = conn.prepareStatement(sql);
            // 2.填充占位符
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i + 1, args[i]);// 小心参数声明错误！！
            }
            // 3.执行
            return ps.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {// 4.资源的关闭
            JDBCUtils.closeResource(null, ps);
        }
        return 0;
    }
    
    // 通用的查询操作，用于返回数据表中的一条记录
    public T getInstance(Connection conn, String sql, Object... args) {
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i + 1, args[i]);
            }
            rs = ps.executeQuery();
            // 获取结果集的元数据 :ResultSetMetaData
            ResultSetMetaData rsmd = rs.getMetaData();
            // 通过ResultSetMetaData获取结果集中的列数
            int columnCount = rsmd.getColumnCount();
            if (rs.next()) {
                T t = clazz.getConstructor().newInstance();
                // 处理结果集一行数据中的每一个列
                for (int i = 0; i < columnCount; i++) {
                    // 获取列值
                    Object columValue = rs.getObject(i + 1);
                    // 获取每个列的列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    // 给t对象指定的columnName属性，赋值为columValue：通过反射
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t, columValue);
                }
                return t;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.closeResource(null, ps, rs);
        }
        return null;
    }
    // 通用的查询操作，用于返回数据表中的多条记录构成的集合
    public List<T> getForList(Connection conn, String sql, Object... args) {
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                ps.setObject(i + 1, args[i]);
            }
            rs = ps.executeQuery();
            // 获取结果集的元数据 :ResultSetMetaData
            ResultSetMetaData rsmd = rs.getMetaData();
            // 通过ResultSetMetaData获取结果集中的列数
            int columnCount = rsmd.getColumnCount();
            // 创建集合对象
            ArrayList<T> list = new ArrayList<T>();
            while (rs.next()) {
                T t = clazz.newInstance();
                // 处理结果集一行数据中的每一个列:给t对象指定的属性赋值
                for (int i = 0; i < columnCount; i++) {
                    // 获取列值
                    Object columValue = rs.getObject(i + 1);
                    // 获取每个列的列名
                    String columnLabel = rsmd.getColumnLabel(i + 1);
                    // 给t对象指定的columnName属性，赋值为columValue：通过反射
                    Field field = clazz.getDeclaredField(columnLabel);
                    field.setAccessible(true);
                    field.set(t, columValue);
                }
                list.add(t);
            }
            return list;
        }
        catch (Exception e) {e.printStackTrace();}
        finally {JDBCUtils.closeResource(null, ps, rs);}
        return null;
    }
    
    //用于查询特殊值的通用的方法
    //由于不同列的类型不同，使用泛型方法
    public <E> E getValue(Connection conn,String sql,Object...args){
        PreparedStatement ps = null;
        ResultSet rs = null;
        try {
            ps = conn.prepareStatement(sql);
            for(int i = 0;i < args.length;i++){
                ps.setObject(i + 1, args[i]);
            }
            rs = ps.executeQuery();
            if(rs.next()){
                return (E) rs.getObject(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally{
            JDBCUtils.closeResource(null, ps, rs);
        }
        return null;
    }
}
```

`CustomerDAO.java`文件，定义接口：

```java
/*
 * 此接口用于规范针对于customers表的常用操作
 */
public interface CustomerDAO {
    //将cust对象添加到数据库中
    void insert(Connection conn,Customer cust);
    
    //针对指定的id，删除表中的一条记录
    void deleteById(Connection conn,int id);
    
    //去修改数据表中指定的记录
    void update(Connection conn,Customer cust);
    
    //针对指定的id查询得到对应的Customer对象
    Customer getCustomerById(Connection conn,int id);
    
    //查询表中的所有记录构成的集合
    List<Customer> getAll(Connection conn);
    
    //返回数据表中的数据的条目数
    Long getCount(Connection conn);
    
    //返回数据表中最大的生日
    Date getMaxBirth(Connection conn);
}
//可以根据需求，定义不同功能的方法
```



`CustmoerDAOImpl.java`文件，实现接口，继承抽象类：

```java
/*
对于customer表，定义一个实现类，实现接口中的所有方法。
实现接口方法时，调用的是DAO抽象类中的方法。
*/
public class CustomerDAOImpl extends BaseDAO<Customer> implements CustomerDAO{
    @Override
    public void insert(Connection conn, Customer cust) {
        String sql = "insert into customers(name,email,birth)values(?,?,?)";
        update(conn, sql,cust.getName(),cust.getEmail(),cust.getBirth());
    }
    
    @Override
    public void deleteById(Connection conn, int id) {
        String sql = "delete from customers where id = ?";
        update(conn, sql, id);
    }
    
    @Override
    public void update(Connection conn, Customer cust) {
        String sql = "update customers set name=?,email=?,birth=? where id=?";
        update(conn,sql,cust.getName(),cust.getEmail(),cust.getBirth(),cust.getId());
    }
    
    @Override
    public Customer getCustomerById(Connection conn, int id) {
        String sql = "select id,name,email,birth from customers where id = ?";
        Customer customer = getInstance(conn,sql,id);
        return customer;
    }
    
    @Override
    public List<Customer> getAll(Connection conn) {
        String sql = "select id,name,email,birth from customers";
        List<Customer> list = getForList(conn,sql);
        return list;
    }
    
    @Override
    public Long getCount(Connection conn) {
        String sql = "select count(*) from customers";
        return getValue(conn, sql);
    }
    
    @Override
    public Date getMaxBirth(Connection conn) {
        String sql = "select max(birth) from customers";
        return getValue(conn, sql);
    }
}
```





# 四、数据库连接池

## 1、连接池简介

传统的数据库连接存在的问题：

- 普通的JDBC数据库连接使用 DriverManager 来获取，每次向数据库建立连接的时候都要将 Connection 加载到内存中，再验证用户名和密码。需要数据库连接的时候，就向数据库要求一个，执行完成后再断开连接。这样的方式将会消耗大量的资源和时间。**数据库的连接资源并没有得到很好的重复利用。**若同时有几百人甚至几千人在线，频繁的进行数据库连接操作将占用很多的系统资源，严重的甚至会造成服务器的崩溃。
- **对于每一次数据库连接，使用完后都得断开。**否则，如果程序出现异常而未能关闭，将会导致数据库系统中的内存泄漏，最终将导致重启数据库。
- **这种开发不能控制被创建的连接对象数**，系统资源会被毫无顾及的分配出去，如连接过多，也可能导致内存泄漏，服务器崩溃。



为了解决以上的问题，可以采用数据库连接池技术：

- **数据库连接池的基本思想**：就是为数据库连接建立一个“缓冲池”。预先在缓冲池中放入一定数量的连接，当需要建立数据库连接时，只需从“缓冲池”中取出一个，使用完毕之后再放回去。

- **数据库连接池**负责分配、管理和释放数据库连接，它**允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个**。
- 数据库连接池在**初始化**时将创建一定数量的数据库连接放到连接池中，这些数据库连接的数量是由**最小数据库连接数来设定**的。无论这些数据库连接是否被使用，连接池都将一直保证至少拥有这么多的连接数量。连接池的**最大数据库连接数量**限定了这个连接池能占有的最大连接数，当应用程序向连接池请求的连接数超过最大连接数量时，这些请求将被加入到等待队列中。



数据库连接池的优点：

* **资源重用**。由于数据库连接得以重用，避免了频繁创建和释放连接引起的大量性能开销。在减少系统消耗的基础上，另一方面也增加了系统运行环境的平稳性。
* **更快的系统反应速度**。数据库连接池在初始化过程中，往往已经创建了若干数据库连接置于连接池中备用。此时连接的初始化工作均已完成。对于业务请求处理而言，直接利用现有可用连接，避免了数据库连接初始化和释放过程的时间开销，从而减少了系统的响应时间
* **新的资源分配手段，控制每个应用的最大连接数**。对于多应用共享同一数据库的系统而言，可在应用层通过数据库连接池的配置，实现某一应用最大可用数据库连接数的限制，避免某一应用独占所有的数据库资源
* **统一的连接管理，避免数据库连接泄漏**。在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免了常规数据库连接操作中可能出现的资源泄露



JDBC 的数据库连接池使用`javax.sql.DataSource` 来表示，`DataSource` 只是一个接口，该接口通常由服务器(Weblogic, WebSphere, Tomcat)提供实现，也有一些开源组织提供实现：

- **DBCP** 是Apache提供的数据库连接池。tomcat 服务器自带dbcp数据库连接池。**速度相对c3p0较快**，但因自身存在BUG，Hibernate3已不再提供支持。
- **C3P0** 是一个开源组织提供的一个数据库连接池，**速度相对较慢，稳定性还可以。**hibernate官方推荐使用
- **Proxool** 是sourceforge下的一个开源项目数据库连接池，有监控连接池状态的功能，**稳定性较c3p0差一点**
- **BoneCP** 是一个开源组织提供的数据库连接池，速度快
- **Druid** 是阿里提供的数据库连接池，据说是集DBCP 、C3P0 、Proxool 优点于一身的数据库连接池，但是速度不确定是否有BoneCP快



`DataSource` 通常被称为数据源，它包含**连接池**和**连接池管理**两个部分，习惯上也经常把` DataSource` 称为**连接池**。

- **DataSource用来取代DriverManager来获取Connection，获取速度快，同时可以大幅度提高数据库访问速度。**
- **整个应用只需要创建一个数据源（连接池）即可。**
- 当数据库访问结束后，程序还是像以前一样关闭数据库连接：`conn.close();`但`conn.close()`操作并没有关闭数据库的物理连接，它仅仅把数据库连接释放，归还给了数据库连接池（由busy状态变为free状态）。



## 2、Druid数据库连接池

Druid是阿里巴巴开源平台上一个数据库连接池实现，它结合了C3P0、DBCP、Proxool等DB池的优点，同时加入了日志监控，可以很好的监控DB池连接和SQL的执行情况，可以说是针对监控而生的DB连接池，**是目前最好的连接池之一。**

连接池的一些配置参数：

| **配置**                      | **缺省** | **说明**                                                     |
| :---------------------------- | -------- | :----------------------------------------------------------- |
| name                          |          | 如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是：”DataSource-” + System.identityHashCode(this) |
| url                           |          | 连接数据库的url，不同数据库不一样。例如：mysql : jdbc:mysql://10.20.153.104:3306/druid2 |
| username                      |          | 连接数据库的用户名                                           |
| password                      |          | 连接数据库的密码。                                           |
| driverClassName               |          | 根据url自动识别   这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置) |
| initialSize                   | 0        | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                     | 8        | 同时维持的最大连接数                                         |
| maxIdle                       | 8        | 最大的空闲连接数。已废弃。类似于JVM中的Xmx                   |
| minIdle                       | 0        | 最小的空闲连接数。类似于JVM中的Xmn                           |
| minEvictableIdleTimeMillis    | 1800000L | 池中的空闲连接在一段时间内一直空闲，被逐出连接池的时间       |
| maxWait                       | -1       | 获取连接时最大等待时间，单位毫秒。-1表示无限等待。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |
| poolPreparedStatements        | false    | 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭。 |
| maxOpenPreparedStatements     | -1       | 开启池的prepared后的同时最大连接数。要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 |
| testOnBorrow                  | true     | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。 |
| testOnReturn                  | false    | 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                 | false    | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。 |
| timeBetweenEvictionRunsMillis |          | 有两个含义： 1)Destroy线程会检测连接的间隔时间2)testWhileIdle的判断依据，详 |
| connectionInitSqls            |          | 物理连接初始化的时候执行的sql                                |



Druid数据库连接池使用案例：

```java
//只需要创建一个连接池，因此写在静态代码块中
private static DataSource source;
static{
    try {
        Properties pros = new Properties();
        FileInputStream is = new FileInputStream("src/druid.properties");
        pros.load(is);
        //创建一个dbcp连接池
        source = BasicDataSourceFactory.createDataSource(pros);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
public static Connection getConnection3() throws SQLException {
    Connection conn = source.getConnection();
    return conn;
}
```

其中`druid.properties`配置文件内容如下：

```properties
url=jdbc:mysql:///test?characterEncoding=utf8
username=root
password=abc123
driverClassName=com.mysql.jdbc.Driver
initialSize=10
maxActive=10
```



# 五、Apache-DBUtils工具类

`commons-dbutils`是 Apache 组织提供的一个开源 JDBC工具类库，它是对JDBC的简单封装，学习成本极低，并且使用`dbutils`能极大简化jdbc编码的工作量，同时也不会影响程序的性能。

API介绍：

- `org.apache.commons.dbutils.QueryRunner`该类简单化了SQL查询，与`ResultSetHandler`搭配使用，可以完成大部分数据库操作。`QueryRunner`的主要方法有：
  - update：执行插入、更新、删除操作
  - insert：只支持insert语句
  - query：查询操作
- `org.apache.commons.dbutils.ResultSetHandler`该接口用于处理`ResultSet`，将数据按要求转换为另一种形式。并且提供了一个单独的方法：`Object handle (java.sql.ResultSet.rs)`。该接口的主要实现类有：
  - ArrayHandler：把结果集中的第一行数据转成对象数组。
  - ArrayListHandler：把结果集中的每一行数据都转成一个数组，再存放到List中。
  - **BeanHandler：**将结果集中的第一行数据封装到一个对应的JavaBean实例中。
  - **BeanListHandler：**将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到List里。
  - ColumnListHandler：将结果集中某一列的数据存放到List中。
  - KeyedHandler(name)：将结果集中的每一行数据都封装到一个Map里，再把这些map再存到一个map里，其key为指定的key。
  - **MapHandler：**将结果集中的第一行数据封装到一个Map里，key是列名，value就是对应的值。
  - **MapListHandler：**将结果集中的每一行数据都封装到一个Map里，然后再存放到List
  - **ScalarHandler：**查询单个值对象
- 工具类`org.apache.commons.dbutils.DbUtils`用于关闭连接、装载JDBC驱动程序等操作。里面的所有方法都是静态的。比如以下方法：
  - `public static void close(xxx) throws java.sql.SQLException`，关闭指定的资源。
  - `public static void closeQuietly(xxx)`: 这一类方法不仅能在Connection、Statement和ResultSet为NULL情况下避免关闭，还能隐藏一些在程序中抛出的SQLEeception。
  - `public static void commitAndClose(Connection conn)throws SQLException`： 用来提交连接的事务，然后关闭连接
  - `public static void commitAndCloseQuietly(Connection conn)`： 用来提交连接，然后关闭连接，并且在关闭连接时不抛出SQL异常。 
  - `public static void rollback(Connection conn)throws SQLException`：允许conn为null，因为方法内部做了判断
  - `public static void rollbackAndClose(Connection conn)throws SQLException`
  - `rollbackAndCloseQuietly(Connection)`
  - `public static boolean loadDriver(java.lang.String driverClassName`)：这一方装载并注册JDBC驱动程序，如果成功就返回true。使用该方法，不需要捕捉这个异常ClassNotFoundException



`DBUtils`的使用案例

```java
public class dbutilsTest{
    /*
    BeanHandler：是ResultSetHandler接口的实现类，用于封装表中的一条记录
    */
    @Test
    public void testQuery1(){
        Connection conn = null;
        try{
            conn = JDBCUtils.getConnection();
            QueryRunner runner = new QueryRunner();
            String sql = "select id,name,email,birth from customers where id=?";
            BeanHandler<Customer> handler = new BeanHandler<>(Customer.class);
            Customer query = runner.query(conn, sql, handler, 22);
            System.out.println(query);
        }catch (SQLException e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    /*
    MapHandler：是ResultSetHandler接口的实现类，对应表中的一条记录
    将字段及字段的值作为map中的key和value
    */
    @Test
    public void testQuery3(){
        Connection conn = null;
        try{
            conn = JDBCUtils.getConnection();
            QueryRunner runner = new QueryRunner();
            String sql = "select id,name,email,birth from customers where id=?";
            MapHandler handler = new MapHandler();
            Map<String, Object> query = runner.query(conn, sql, handler, 22);
            System.out.println(query);
        }catch (SQLException e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    /*
    ScalarHandler用于查询特殊值
    */
    @Test
    public void testQuery5(){
        Connection conn = null;
        try{
            conn = JDBCUtils.getConnection();
            QueryRunner runner = new QueryRunner();
            String sql = "select Max(birth) from customers";
            ScalarHandler scalarHandler = new ScalarHandler();
            //返回sql.date类型，转为util.date型
            Date date = (Date)runner.query(conn, sql, scalarHandler); 
            System.out.println(date);
        }catch (SQLException e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
    /*
    自定义ResultSetHandler实现类
    */
    @Test
    public void testQuery7(){
        Connection conn = null;
        try{
            conn = JDBCUtils.getConnection3();
            QueryRunner runner = new QueryRunner();
            String sql = "select id,name,email,birth from customers where id=?";
            ResultSetHandler<Customer> handler = new ResultSetHandler<Customer>() {
                @Override
                public Customer handle(ResultSet rs) throws SQLException {
                    return "自定义结果";
                }
            };
            Customer query = runner.query(conn, sql, handler, 23);//返回sql.date类型，转为util.date型
            System.out.println(query);
        }catch (SQLException e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeResource(conn,null);
        }
    }
}
```





# 六、总结

# 1、JDBC总结

至此，回顾使用JDBC编程的几个主要步骤，主要总结为三步：创建连接、CRUD操作、关闭资源。

可以发现，我们使用数据库连接池代替了传统的连接方式，使用DBUtils工具类代替了手动创建Statement对象实现CRUD的操作，和关闭资源的操作。

JDBC的流程可以总结为下（考虑事务）：

```java
public void testUpdateWithTx() {
    Connection conn = null;
    try {
        /*1.获取连接的操作
        方式一：手写的连接：JDBCUtils.getConnection();
        方式二：使用数据库连接池：C3P0;DBCP;Druid
        */
        /*2.对数据表进行CRUD操作
        方式一：使用PreparedStatement对象
        public void update(Connection conn,String sql,Object ... args){}
        public T getInstance(Connection conn,String sql,Object ... args){}
        方式二：使用dbutils提供的jar包中提供的QueryRunner类
        */
        //提交数据
        conn.commit();
    } catch (Exception e) {
        e.printStackTrace();
        try {//回滚数据
            conn.rollback();
        } catch (SQLException e1) {e1.printStackTrace();}
    }finally{
        /*3.关闭连接等操作
        方式一：JDBCUtils.closeResource();
        方式二：使用dbutils中的DbUtils工具类提供的关闭资源的方法
        */
    }
}
```



# 2、常见问题总结

问题1：**数据库连接池工作原理和实现方案?**

* 工作原理：JAVA EE服务器启动时会建立一定数量的池连接，并一直维持不少于此数目的池连接(minIdle)。客户端程序需要连接时，池驱动程序会返回一个未使用的池连接，并将其标记为busy状态。如果当前没有空闲连接，池驱动程序就新建一定数量的连接，新建连接的数量有配置参数决定。当使用的池连接调用完成后，池驱动程序将此连接标记为free，其他调用就可以使用这个连接。
* 实现方案：**连接池使用集合来进行装载，返回的Connection是原始Connection的代理，代理Connection的close方法，当调用close方法时，不是真正关闭物理连接，而是把它代理的Connection对象放回到连接池中，等待下一次重复利用。**

***



问题2：**JDBC是如何实现Java程序和JDBC驱动的松耦合的？**

通过制定接口，数据库厂商来实现。我们只要通过接口调用即可，**所有的操作都是通过JDBC接口完成的**，而驱动只有在通过Class.forName反射机制来加载的时候才会出现。——面向接口编程

***



问题3：**PreparedStatement有什么缺点，如何解决**

缺点：不能直接用它来执行in条件语句。

解决方案：

- 分别进行单条查询——性能很差，不推荐。
- 使用存储过程——这取决于数据库的实现，不是所有数据库都支持。
- 动态生成PreparedStatement——不能享受PreparedStatement的缓存带来的好处。
- 在PreparedStatement查询中使用NULL值——如果知道输入变量的最大个数的话，这是个不错的办法，扩展一下还可以支持无限参数。

***



问题4：**JDBC的ResultSet是什么 **

- 在查询数据库后会返回一个ResultSet，它像是查询结果集的一张数据表。
- ResultSet对象维护了一个游标，指向当前的数据行。开始的时候这个游标指向的是第一行。如果调用了ResultSet的next()方法游标会下移一行，如果没有更多的数据了，next()方法会返回false。可以在for循环中用它来遍历数据集。
- 默认的ResultSet是不能更新的，游标也只能往下移。也就是说只能从第一行到最后一行遍历一遍。不过也可以创建可以回滚或者可更新的ResultSet。
- 当生成ResultSet的Statement对象要关闭或者重新执行或是获取下一个ResultSet的时候，ResultSet对象也会自动关闭。
- 可以通过ResultSet的`get`方法，传入列名或者从1开始的序号来获取列数据。

***



问题5：**JDBC的DataSource是什么，有什么好处**

DataSource即数据源，它是定义在`javax.sql`中的一个接口，跟DriverManager相比，它的功能要更强大。DataSource也被称为数据库连接池。除了提供连接外，它还能够管理连接对象，提供如下功能：

- **缓存PreparedStatement以便更快的执行**
- **可以设置连接超时时间**
- **提供日志记录的功能**
- **ResultSet大小的最大阈值设置**
- **通过JNDI的支持，可以为servlet容器提供连接池的功能**

***



问题6：**JDBC中存在哪些不同类型的锁**

从广义上讲，有两种锁机制来防止多个用户同时操作引起的数据损坏。

- **乐观锁**：只有当更新数据的时候才会锁定记录。
- **悲观锁**：从查询到更新和提交整个过程都会对数据记录进行加锁。



更多问题总结参考：[JDBC面试题都在这里](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483745&idx=1&sn=c5e8d9ad63878aa5bc3dcdbb903d8899&chksm=ebd74060dca0c976345e93c1304b1d1d609e804665a3c2c338cbaa6903eee01477ca47c9645b#rd)