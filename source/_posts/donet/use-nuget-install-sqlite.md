---
title: 使用Nuget 安装 SQLite 小型数据库
date: 2017-10-29 22:07:04
tags: C#
---
# 说明
记录一下 使用Nuget 安装 SQLite 小型数据库，并记录使用示例

## 操作步骤如下
### 第1步 下载安装方式
1. 去nuget 直接下载[System.Data.SQLite.dll](https://www.nuget.org/packages?q=sqlite)然后引用
2. 在VS 工具 –> Nuget包管理器 –> 程序包管理器控制台 安装dll
```bash
# 输入命令安装，也可以使用可视化工具安装
Install-Package System.Data.SQLite -Version 1.0.105.2
```
### 第2步 新建Sqlite 数据库
1. 使用CodeFirst 创建实体类 要求和表的字段一一对应 其他特性不列举
```C#
/// <summary>
/// 实体类
/// </summary>
[Table("表名")]
public class TestTable
{
    [Key]
    public String guid { get; set; }

    public String field01 {get; set;}

    public String field02 {get; set;}
    //。。。。
}
```
2. 新建ConnectionString链接信息和DBContext类  
```xml
<!-- app.config 配置-->
<!-- 更改或添加providers -->
 <providers>
      <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      <provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
      <provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6 " />
    </providers>

<!-- 添加 connectionString  -->
<connectionStrings>
     <add name="SQLiteDbContext" connectionString="Data Source=CarInfoDB.sqlite" providerName="System.Data.SQLite.EF6" />
</connectionStrings>
```

```csharp
//C# 代码配置
 public class SQLiteDbContext : DbContext
{
    // 可以使用 base 指定链接名 ， 也可以不指定但类名 要和链接名一致
    //public SQLiteDbContext() : base("SQLiteDbContext") {
    //}

    public DbSet<TestTable> TestTableEntities { get; set; }
}

```

### 第3步 测试是否自动生成sqlite数据库
```csharp
public void TestConn() {
    SQLiteDbContext dbCxt = new SQLiteDbContext();
    var res = dbCxt.TestTableEntities.Where(m => true).Count();
    System.Console.WriteLine(res);
}
```

