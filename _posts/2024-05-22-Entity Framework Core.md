# Entity Framework Core

## 数据库连接

| **数据库**  | **Nuget程序包**                                       |
| ----------- | ----------------------------------------------------- |
| SQL Server  | Microsoft.EntityFrameworkCore.SqlServer               |
| MySQL       | MySql.Data.EntityFrameworkCore（官方版，不建议使用）  |
| MySQL       | Pomelo.EntityFrameworkCore(第三方提供，Bug少建议使用) |
| PostgreSQL  | Npgsql.EntityFrameworkCore.PostgreSQL                 |
| SQLite      | Microsoft.EntityFrameworkCore.SQLite                  |
| SQL Compact | Microsoft.EntityFrameworkCore.SQLite                  |
| In-memory   | Microsoft.EntityFrameworkCore.InMemory                |



### 连接MySQL

#### 安装Nuget包

首先要安装如下Nuget包：

- Pomelo.EntityFrameworkCore.MySql
- Microsoft.EntityFrameworkCore



#### 注入DbContext

##### appsetting.json配置

```json
{
  "ConnectionString": "server=43.**.3.252;database=learning-csharp;uid=***;password=***;port=3306;",
  "Logging": {
    // ...
  }
}
```



##### 启动类配置

```c#
string mysqlConnectStr = builder.Configuration["ConnectionString"];
builder.Services.AddDbContext<DatabaseDbContext>(options =>
{
    options.UseMySql(
        mysqlConnectStr,
        ServerVersion.AutoDetect(mysqlConnectStr),
        options => options.EnableRetryOnFailure(
            maxRetryCount: 3,
            maxRetryDelay: TimeSpan.FromSeconds(10),
            errorNumbersToAdd: null
        )
    );
});
```





## ORM应用

### 实体类

```c#
[Table("database")]
public class DatabaseEntity
{
    [Key]
    public long Id { get; set; }
    public string Name { get; set; }
}
```



### DbContext定义

```c#
public class DatabaseDbContext : DbContext
{
    public DatabaseDbContext(DbContextOptions<DatabaseDbContext> options) : base(options)
    {

    }

    public DbSet<DatabaseEntity> DatabaseEntities { get; set; }
}
```



### DbSet使用

```c#
[ApiController]
[Route("[controller]/[action]")]
public class DatabaseController
{
    private readonly DatabaseDbContext DatabaseDbContext;
    public DatabaseController(DatabaseDbContext databaseDbContext)
    {
        DatabaseDbContext = databaseDbContext;
    }

    [HttpGet]
    public ApiResult<List<DatabaseEntity>> List()
    {
        List<DatabaseEntity> databaseEntities = DatabaseDbContext.DatabaseEntities.ToList();
        return ApiResult<List<DatabaseEntity>>.Success(databaseEntities);
    }
}
```



## DbContext

DbContext是实体类和数据库之间的桥梁， DbContext主要负责与数据交互，主要作用：

- **查询**：将LINQ-to-Entities查询转换为SQL查询并将其发送到数据库。
- **更改跟踪**：跟踪实体在从数据库查询后发生的更改。
- **持久化数据**：根据实体的状态对数据库执行插入，更新和删除操作。
- **缓存**：默认提供一级缓存。它存储在上下文类生命周期中已经被检索的实体。
- **管理关系**：在Db-First或Model-First方法中使用CSDL，MSL和SSDL管理关系，并以Code-First方法使用流畅的API配置。
- **对象实现**：将来自数据库的原始数据转换为实体对象。



### 方法

Entry:获取DbEntityEntry给定的实体。该条目提供访问更改实体的跟踪信息和操作。
SavaChange:对已添加，已修改或已删除状态的实体的数据库执行INSERT，UPDATE或DELETE命令。
SaveChangesAsync: SaveChanges（）的异步方法
Set: 创建一个DbSet可以用来查询和保存实例的TEntity。
OnModelCreating 重写此方法以进一步配置通过DbSet派生上下文中属性中公开的实体类型按惯例发现的模型。



## DbSet

