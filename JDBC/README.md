
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