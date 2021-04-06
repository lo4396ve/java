
# JDBC
## 1、概念
Java数据库连接，（Java Database Connectivity，简称JDBC）是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。

使用Java程序访问数据库时，Java代码并不是直接通过TCP连接去访问数据库，而是通过JDBC接口来访问，而JDBC接口则通过JDBC驱动来实现真正对数据库的访问。

从代码来看，Java标准库自带的JDBC接口其实就是定义了一组接口，而某个具体的JDBC驱动其实就是实现了这些接口的类

## 2、JDBC链接
两个重要的类
* DriverManager（驱动管理对象）：主要功能注册驱动告诉程序该使用哪一个数据库驱动和获取数据库连接
* Connection（数据库连接对象）：主要功能包括获取执行sql的对象和事务管理

JDBC链接示例

```
String JDBC_URL = "jdbc:mysql://localhost:3306/dataBaseName";
String JDBC_USER = "root";
String JDBC_PASSWORD = "password";
// 获取连接会getConnectio方法自动扫描classpath，找到所有的JDBC驱动，然后根据传入的URL自动挑选一个合适的驱动。
Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
// 数据库操作...
// 关闭连接:
conn.close();
```
## 3、JDBC操作
### 3.1 查询
JDBC提供的Statemen和 PreparedStatement接口主要是用来发送 SQL 命令或 PL/SQL 命令到数据库，并从数据库接收数据。
* Statement：不会初始化SQL，Statement 接口不接受参数，适用于运行静态 SQL 语句。 
* PreparedStatement：会先初始化SQL，接受参数先把这个SQL提交到数据库中进行预处理，多次使用可提高效率。比Statement更安全，而且更快

对数据库的查询操作，一般需要返回查询结果，在程序中，JDBC提供了ResultSet接口来专门处理查询结果集。

使用Statement：
```
...
Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
Statement statement = conn.createStatement();
User login(String name, String pass) {
    ...
    Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
    Statement statement = conn.createStatement();
    ResultSet rs = statement.executeQuery("SELECT * FROM user WHERE username='" + name + "' AND pass='" + pass + "'");

    while (rs.next()) {
        String name = rs.getLong(1); // 索引从1开始
        String pass = rs.getLong(2);
    }
    ...
}
```

使用Statement拼字符串非常容易引发SQL注入的问，参数name和pass通常都是Web页面用户输入后由程序接收到的，如果用户的输入是一个精心构造的字符串，就可以拼出意想不到的SQL，例如：name = "bob' OR pass=", pass = " OR pass='"，最后拼接出来SQL为
```
SELECT * FROM user WHERE username='bob' OR pass=' AND pass=' OR pass=''
```

PreparedStatement就可以避免SQL注入的问题：
```
User login(String name, String pass) {
    ...
    String sql = "SELECT * FROM user WHERE username=? AND pass=?";
    PreparedStatement ps = conn.prepareStatement(sql);
    ps.setObject(1, name);
    ps.setObject(2, pass);
    ResultSet rs = ps.executeQuery();
    while (rs.next()) {
        String name = rs.getLong(1); // 索引从1开始
        String pass = rs.getLong(2);
    }
    ...
}
```

使用占位符？表示SQL语句的参数，用setObject方法为占位符设置预期的值，最后再调用executeQuery执行完整的SQL命令。

严格来讲，为了防止SQL注入，所有的程序都应该避免使用Statement来操作数据库。

### 3.2 插入
#### 3.2.1 非自增主键
```
PreparedStatement ps = conn.prepareStatement("INSERT INTO user (id, username, pass) VALUES (?,?,?)");
ps.setObject(1, 111); // 注意：索引从1开始
ps.setObject(2, "张三");
ps.setObject(3, "passsword");
int n = ps.executeUpdate(); // 返回入的记录数量 1
```
与查询不同的是使用的不再是SELECT而是INSERT，最后执行的不是executeQuery()，而是executeUpdate()。executeUpdate返回值是int，表示插入的记录数量。此处总是1，因为只插入了一条记录。

#### 3.2.2 插入自增主键
如果数据库的表设置了自增主键(id)，那么在执行INSERT语句时，并不需要指定主键，数据库会自动分配主键。对于使用自增主键的程序，有个额外的步骤，就是如何获取插入后的自增主键的值。

获取自增主键的正确写法是在创建PreparedStatement的时候，指定一个RETURN_GENERATED_KEYS标志位，表示JDBC驱动必须返回插入的自增主键

```
PreparedStatement ps = conn.prepareStatement("INSERT INTO user (username, pass) VALUES (?,?)",Statement.RETURN_GENERATED_KEYS);
ps.setObject(1, "张三"); // 注意：索引从1开始
ps.setObject(2, "passsword");
int n = ps.executeUpdate(); // 返回入的记录数量 1
try (ResultSet rs = ps.getGeneratedKeys()) {
    if (rs.next()) {
        long id = rs.getLong(1); // 注意：索引从1开始
    }
}
```

需要注意两个地方：
1. 调用prepareStatement()时，第二个参数必须传入常量Statement.RETURN_GENERATED_KEYS，否则JDBC驱动不会返回自增主键
2. 执行executeUpdate()方法后，必须调用getGeneratedKeys()获取一个ResultSet对象，这个对象包含了数据库自动生成的主键的值，读取该对象的每一行来获取自增主键的值。如果一次插入多条记录，那么这个ResultSet对象就会有多行返回值。如果插入时有多列自增，那么ResultSet对象的每一行都会对应多个自增值（自增列不一定必须是主键）。

### 3.3 更新
```
PreparedStatement ps = conn.prepareStatement("UPDATE user SET username=? WHERE id=?");
ps.setObject(1, "李四"); // 注意：索引从1开始
ps.setObject(2, 111);
int n = ps.executeUpdate(); // 返回更新的行数
```

### 3.4 删除
```
PreparedStatement ps = conn.prepareStatement("DELETE FROM user WHERE id=?");
ps.setObject(1, 111); // 注意：索引从1开始
int n = ps.executeUpdate(); // 返回删除的行数
```

## 4、事务
### 4.1 特性
数据库事务（Transaction），多条SQL命令执行，要么都成功，要么都失败，只要有一条命令失败，都需要回滚操作。
数据库事务具有ACID特性：
* **原子性（Atomicity）** 事务是最小单位。一个事务中的所有操作，要么全部完成，要么全部不完成
* **一致性（Consistency）** 执行事务是使得数据库从一个一致性状态到另一个一致性状态，如果事务最终没有被提交，那么事务所做的修改也不会保存到数据库中
* **隔离性（Isolation）** 涉及到隔离级别的概念，当多个事务同时对数据库操作，通过设置不同的隔离级别实施不同的处理方案
* **持久性（Durability）** 事务一旦被提交，那么对数据库的修改会被永久的保存

### 4.2 隔离级别
#### 4.2.1 未提交读（READ UNCOMMITTED）
两个事务同时操作数据库，但是其中一个事务A可以读到另一个未提交事务B修改过的数据。这种隔离级别就称为未提交读。

如果事务B又进行了回滚，那么事务A就相当于读取到了不存在的数据，这种现象称为脏读。这种隔离级别非常不安全，尽量不要使用。

READ UNCOMMITTED隔离级别的事务实现方式就是直接读取记录的最新版本。

#### 4.2.2 已提交读（READ COMMITTED）
一个事务只能读到另一个已经提交的事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值，这种隔离级别称为已提交读。

已提交读不存在脏读的情况，由于每次其他事务提交的修改该事务都能获取到最新值，所以这种隔离级别下会存在某一事务对同一条记录每次获取的值不一样，这种现象称为不可重复读（即一个事务中多次读的数据可能都不一样）。
#### 4.2.3 可重复读（REPEATABLE READ）
**默认隔离级别** ，一个事务只能读到其他已经提交的事务修改过的数据，但是第一次读过某条记录后，即使其他事务修改了该记录的值并且提交，该事务之后再读该条记录时，读到的仍是第一次读到的值，而不是最新值，这种隔离级别就称为可重复读

也就是在一个事务里，对某条记录只要读过一次，后面再读都是第一次读到的值，不再变化，保证在一个事务中多次读同一条记录每次都得到同样的值。

READ COMMITTED和REPEATABLE READ隔离级别的实现方式比较复杂，涉及到版本链和ReadView的概念。
#### 4.2.4 串行化（SERIALIZABLE）
以上3种隔离级别都允许对同一条记录进行读-读、读-写、写-读的并发操作，如果我们不允许读-写、写-读的并发操作。如果一个事务对数据库进行读写操作时有其他事务正在对数据库操作，该事务会处于等待状态，直到其他事务操作结束该事务再进行操作。这种隔离级别称为串行化。

串行化利用锁来实现，当一个事务对数据库某一天记录操作时，会给这条记录加锁，锁的作用就是防止其他事务再对这条记录操作，直到该事务提交才会释放锁，其他事务才可以读写这条记录。这种隔离级别可能导致大量超时和锁争用的问题，实际中很少使用这个隔离级别。


### 4.3 使用事务
JDBC在执行SQL命令时，每次执行完一句SQL就会自动commit提交。如果要把多条SQL包裹在一个数据库事务中执行，就要先关闭JDBC的自动提交，然后手动提交和回滚。
```
Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
try{
  conn.setAutoCommit(false);  // 关闭自动提交

  PreparedStatement ps = conn.prepareStatement("DELETE FROM user WHERE id=1");
  int n = ps.executeUpdate(); // 执行第一个SQL
  PreparedStatement ps = conn.prepareStatement("DELETE FROM user WHERE id=2");
  int n = ps.executeUpdate(); // 执行第二个SQL

  conn.commit();  // 提交事务

} catch(SQLException e) {
  // 回滚事务:
  conn.rollback();
} finally {
  conn.setAutoCommit(true);
  conn.close();
}
```

## 5、Batch批量操作
如果要把user表中的每个人的年龄加一，可以使用循环的方式执行多次SQL，这种方式效率比较低，可以使用batch批量执行。
```
PreparedStatement ps = conn.prepareStatement("INSERT INTO user (id, name, age) VALUES (?, ?, ?)");
for (User user : userList) {
    ps.setInt(1, user.id);
    ps.setString(2, user.name);
    ps.setInt(3, user.age + 1);
    ps.addBatch(); // 添加到batch
}

int[] ns = ps.executeBatch();
for (int n : ns) {
    System.out.println(n + " inserted."); // batch中每个SQL执行的结果数量
}
```




