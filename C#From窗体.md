#### 数据库连接

DAL层（数据访问层）一般用于数据库（ sql server ）的连接和操作。

1.**连接数据库**:

- 首先，调用 `connection()` 方法来创建一个 `SqlConnection` 对象并连接到数据库。

```c#
public SqlConnection connection() //SqlConnection是 .NET Framework 提供的一个用于连接到 SQL Server 数据库的类。它表示一个打开的连接。
{
    string scStr = @"Data Source=.\;Initial Catalog=Demo;User Id=sa;Password=000000;";
    /*scStr是连接字符串
    	Data Source=.\: 这指定了数据库服务器的位置，. 表示本地服务器。
		Initial Catalog=Demo: 这指定了要连接的数据库名称，这里是 "Demo"。
		User Id=sa;Password=000000;: 这指定了连接数据库所需的用户名和密码*/
    SqlConnection sc = new SqlConnection(scStr);
    //创建一个SqlConnection对象sc传参scStr
    sc.Open();
    //打开数据库连接
    return sc;
    //返回SqlConnection对象
```

2.**执行 SQL 命令**:

- 当调用 `Command(string sql)` 方法时，它会创建一个 `SqlCommand` 对象，该对象与数据库连接建立，并且接受一个 SQL 命令字符串作为参数。

```c#
public SqlCommand Command(string sql) //方法接受一个参数 sql，该参数是一个字符串，表示要执行的 SQL 命令。
{
    SqlCommand scom = new SqlCommand(sql,connection());
    //在方法内部，首先创建一个 SqlCommand 对象 scom。
    /*构造 SqlCommand 对象时，传入两个参数：
第一个参数是 sql，即要执行的 SQL 命令字符串。
第二个参数是通过调用 connection() 方法返回的 SqlConnection 对象，它用于指定此 SqlCommand 对象所属的数据库连接。*/
    return scom;
    //创建完成 SqlCommand 对象后，将其赋值给局部变量 scom，然后将其返回。
}
```

这个方法的主要作用是封装了创建 `SqlCommand` 对象的过程，并指定了该对象所属的数据库连接。调用该方法可以方便地获取一个已经配置好的 `SqlCommand` 对象，可以直接用于执行 SQL 命令。

3.**执行查询并返回数据**:

- 当调用 `DataAdapter(string sql)` 方法时，它会先调用 `Command(sql)` 方法来创建一个 `SqlCommand` 对象，然后使用该对象创建一个 `SqlDataAdapter` 对象，用于执行查询并填充一个 `DataSet`，最后返回该 `DataSet` 对象。

```c#
public SqlDataAdapter DataAdapter(string sql) //方法接受一个参数 sql，该参数是一个字符串，表示要执行的 SQL 命令。
{
    SqlCommand scom = this.Command(sql);
    //创建SqlCommand对象：根据传入的sql命令创建并配置一个SqlCommand对象。返回的SqlCommand对象赋值给局部变量scom
    SqlDataAdapter sqlDataApt = new SqlDataAdapter(scom);//使用上一步得到的 scom 对象，创建一个 SqlDataAdapter 对象 sqlDataApt。在创建 SqlDataAdapter 对象时，传入 scom 对象作为参数，以便指定要执行的 SQL 命令。
    return sqlDataApt;  
    //创建完成 SqlDataAdapter 对象后，将其赋值给局部变量 sqlDataApt，然后将其返回。
}
```

这个方法的主要作用是封装了创建 `SqlDataAdapter` 对象的过程，并配置了其要执行的 SQL 命令。调用该方法可以方便地获取一个已经配置好的 `SqlDataAdapter` 对象，可以直接用于执行查询并填充数据集。

4.**释放资源**:

- 在 `dataset(string sql)` 方法中，使用 `using` 语句确保在使用完 `SqlDataAdapter` 后释放相关资源。

```c#
public DataSet dataset(string sql) //方法接受一个参数 sql，该参数是一个字符串，表示要执行的查询 SQL 命令。
{
    using (SqlDataAdapter SAT = this.DataAdapter(sql))//这个方法的作用是根据传入的查询 SQL 命令创建并配置一个 SqlDataAdapter 对象SAT。
    {
        DataSet ds = new DataSet();
        //在 using 语句中创建一个 DataSet 对象 ds，该对象用于存储查询结果。
        SAT.Fill(ds);
        //使用上一步得到的 SqlDataAdapter 对象 SAT，调用其 Fill 方法来执行查询并填充数据到 DataSet 对象 ds 中。
        return ds;
        //填充完成后，将 DataSet 对象 ds 返回给调用者。
    }
    //using 语句确保在 DataSet 对象使用完毕后及时释放资源。
}
```

这个方法的主要作用是执行查询并将查询结果填充到一个 `DataSet` 对象中，然后返回该 `DataSet` 对象。使用 `using` 语句确保了 `SqlDataAdapter` 对象在使用完毕后会被及时释放，避免资源泄漏。

5.**执行增删改查操作**:

- `Excute(string sql)` 方法用于执行增删改查操作，它会调用 `Command(sql)` 方法创建一个 `SqlCommand` 对象，并调用 `ExecuteNonQuery()` 方法执行 SQL 命令，并返回受影响的行数。

```c#
public int Excute(string sql) //方法接受一个参数 sql，该参数是一个字符串，表示要执行的 SQL 命令。
{
    return Command(sql).ExecuteNonQuery();
    //根据传入的 SQL 命令字符串创建并配置一个 SqlCommand 对象,调用其 ExecuteNonQuery 方法来执行 SQL 命令。ExecuteNonQuery 方法用于执行对数据库的增、删、改操作，并返回受影响的行数(int)。
}
```

这个方法的主要作用是执行给定的 SQL 命令，并返回执行后受影响的行数。

- `read(string sql)` 方法用于执行查询语句，并返回一个 `SqlDataReader` 对象，该对象包含查询结果的数据。

```c#
public SqlDataReader read(string sql) //方法接受一个参数 sql，该参数是一个字符串，表示要执行的查询 SQL 命令
{
    return Command(sql).ExecuteReader();
    //根据传入的 SQL 命令字符串创建并配置一个 SqlCommand 对象,ExecuteReader 方法用于执行查询，并返回一个用于逐行读取查询结果的 SqlDataReader 对象(只进只读,只能按顺序不能随机访问行或修改数据，快速而轻量)。
}
```

这个方法的主要作用是执行查询并返回一个 `SqlDataReader` 对象，该对象可以逐行读取查询结果。



tips:在使用时，可以按照业务需求调用这些方法来执行相应的数据库操作。例如，如果需要执行一个查询语句并获取结果集，可以调用 `dataset(string sql)` 方法；如果需要执行一个更新操作，可以调用 `Excute(string sql)` 方法。