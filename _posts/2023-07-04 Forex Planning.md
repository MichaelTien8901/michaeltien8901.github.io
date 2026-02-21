# Forex Project Planning
<!-- TOC -->

- [Forex Project Planning](#forex-project-planning)
        - [Migration in Entity Framework Core](#migration-in-entity-framework-core)
        - [For Create SQLite with EF Code First](#for-create-sqlite-with-ef-code-first)
        - [Reverse engineering Scaffold-DbContext](#reverse-engineering-scaffold-dbcontext)
            - [For SQL authentication](#for-sql-authentication)
            - [For Windows authentication](#for-windows-authentication)
    - [WebAPI](#webapi)
        - [Tutorials](#tutorials)
        - [dotnet-aspnet-codegenerator](#dotnet-aspnet-codegenerator)
            - [Synopsis](#synopsis)
            - [Description](#description)
            - [Arguments](#arguments)
        - [Authentication and Authorization](#authentication-and-authorization)
            - [LDAP](#ldap)
        - [Sql Server Report Service](#sql-server-report-service)
        - [Docker](#docker)
    - [Sub-Projects](#sub-projects)
        - [Membership Management](#membership-management)
            - [zero-length string POST or PUT problem](#zero-length-string-post-or-put-problem)
                - [Problem: Error happened when POST or PUT with EFentity framework](#problem-error-happened-when-post-or-put-with-efentity-framework)
                - [Reason](#reason)
                - [Use T-SQL to delete constraints with name SSMA_CC and end with disallow_zero_length](#use-t-sql-to-delete-constraints-with-name-ssma_cc-and-end-with-disallow_zero_length)
            - [Add additional column of table Account to connect to users in authentication database](#add-additional-column-of-table-account-to-connect-to-users-in-authentication-database)
        - [Name Service](#name-service)
        - [Broker](#broker)
        - [Back Office](#back-office)
        - [Accounting](#accounting)
    - [System Architecture](#system-architecture)
    - [Database Sanitize](#database-sanitize)

<!-- /TOC -->
```PM
Scaffold-DbContext [-Connection] [-Provider] [-OutputDir] [-Context] [-Schemas>] [-Tables <tables>] 
                    [-DataAnnotations] [-Force] [-Project] [-StartupProject] [<CommonParameters>]

```

In Visual Studio, select menu Tools -> NuGet Package Manger -> Package Manger Console and run the following command:

```PM
PM> Scaffold-DbContext "Server=.\SQLExpress;Database=SchoolDB;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables dbo.Users dbo.roles
```

### Migration in Entity Framework Core

Migration is a way to keep the database schema in sync with the EF Core model by preserving data.

EF Core API builds the EF Core model from the domain (entity) classes and EF Core migrations will create or update the database schema based on the EF Core model.

Whenever you change the domain classes, you need to run migration to keep the database schema up to date.

EF Core migrations are a set of commands which you can execute in NuGet Package Manager Console or in dotnet Command Line Interface (CLI).

| PMC Command                      | dotnet CLI command     | Usage                                               |
|:---------------------------------|:-----------------------|:----------------------------------------------------|
| add-migration `<migration name>` | Add `<migration name>` | Creates a migration by adding a migration snapshot. |
| Remove-migration                 | Remove                 | Removes the last migration snapshot.                |
| update-database                  | Update                 | Updates the database schema based on the last migration snapshot. |
| Script-migration                 | Script                 | Generates a SQL script using all the migration snapshots. |

* Adding a Migration

    * Package Manager Console

```PM Console
PM> add-migration MyFirstMigration
```

    * dotnet Command Line Interface, 

```dotnet
> dotnet ef migrations add MyFirstMigration
```

* Creating or Updating the Database

    * Package Manager Console

```PM
PM> Update-Database
```

    * dotnet command line interface

```dotnet
dotnet ef database update
```

* Revert a Migration

    Suppose you changed your domain class and created the second migration named MySecondMigration using the add-migration command and applied this migration to the database using the Update command. But, for some reason, you want to revert the database to the previous state. In this case, use the update-database <migration name> command to revert the database to the specified previous migration snapshot.

    * Package Manager Console

```pm
PM>Update-database MyFirstMigration
```

   * CLI

```cli
> dotnet ef database update MyFirstMigration
```

### For Create SQLite with EF Code First

* Nuget add libraries

```
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="7.0.15" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="7.0.15"/>
    <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="7.0.15" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="7.0.15"/>

```
* Create model class

```c#
namespace TicketService.Models
{
   public class CounterModel
   {
      public int Id { get; set; }
      public string Name { get; set; } = string.Empty;
      public int Value { get; set; } = 0;
   }
}

```

* Create DbContext

```c#
using Microsoft.EntityFrameworkCore;
using System.Diagnostics.Metrics;

namespace TicketService.Models
{
   public class CounterDbContext: DbContext
   {
      public CounterDbContext(DbContextOptions<CounterDbContext> options) : base(options)
      {
      }

      //public DbSet<CounterModel> Counters { get; set; }
      public DbSet<CounterModel> Counters => Set<CounterModel>();
   }
}

```

* Define Connection String in appsettings.json

```json
  "ConnectionStrings": {
    "UniqueCounterConnection": "Data Source=.\\sqlite\\counter.sqlite"
  },
```

* Add connection in program.cs

```c#
builder.Services.AddDbContext<CounterDbContext>(options =>
   options.UseSqlite(configuration.GetConnectionString("UniqueCounterConnection")));
```

* In Package manager, make sure to be at the correct project folder.  Use `dir`, `cd`
* Use add-migration and update-database to create sqlite database 

```PM
PM> add-migration -Context CounterDbContext InitialDb
PM> update-database -Context CounterDbContext
```

* Sync up the ticket number 

```console
curl -X PUT -H "Content-Type: application/json" -d "42" https://localhost:{your_port}/api/UniqueInteger/id1
```

### Reverse engineering (Scaffold-DbContext)

Generates a DbContext and entity type classes for a specified database. This is called reverse engineering.

#### For SQL authentication

```pm
PM> Scaffold-DbContext "Server=.;Initial Catalog=FOREXNAMEDB;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;user id=sa;password=p@ssw0rd;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -OutPutDir Models
```

#### For `Windows authentication`

Replace 'user id' and 'password' with "Integrated Security=SSPI;" in the first parameter `connection string`

* Generate entity framework models for `FOREXNAMEDB`, `FOREXOFAC`, `FOREXBANNEDSERACH`

```pm
PM> Scaffold-DbContext "Data Source=localhost;Initial Catalog=FOREXNAMEDB;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;Integrated Security=SSPI;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -OutPutDir Models

PM> Scaffold-DbContext "Data Source=localhost;Initial Catalog=FOREXOFAC;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;Integrated Security=SSPI;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -OutPutDir Models

PM> Scaffold-DbContext "Data Source=localhost;Initial Catalog=FOREXBANNEDSEARCH;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;Integrated Security=SSPI;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -OutPutDir Models

```

* Generate MemberService entity framework models from `FOREXDB`

```pm
PM> Scaffold-DbContext "Data Source=localhost;Initial Catalog=FOREXDB;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;Integrated Security=SSPI;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -Tables Account,Broker,Employee -OutPutDir Models 
```

## WebAPI

### Tutorials

* [Tutorial: Create a web API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-7.0&tabs=visual-studio)

* [RestFul Readme](D:\projects\2023\Restful\TodoApi\Readme.md)

* [Designing a Web Api Using Database first Approach](https://www.youtube.com/watch?v=EROpDwbpGLA&list=PLZ1x30w8XcXMI1jujASGQuwuXv3IFr1Zq&ab_channel=BenjaminFadina)

* [Pro ASP.NET Core 6](https://github.com/Apress/pro-asp.net-core-6)

* [Pro ASP.NET Core 7](https://github.com/ManningBooks/pro-asp.net-core-7)

### dotnet-aspnet-codegenerator

* [Tutorial](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/tools/dotnet-aspnet-codegenerator?view=aspnetcore-7.0)

#### Synopsis

```CLI
dotnet aspnet-codegenerator [arguments] [-p|--project] [-n|--nuget-package-dir] [-c|--configuration] [-tfm|--target-framework] [-b|--build-base-path] [--no-build] 
dotnet aspnet-codegenerator [-h|--help]
```

#### Description

The dotnet aspnet-codegenerator global command runs the ASP.NET Core code generator and scaffolding engine.

#### Arguments

| Generator | Operation              |
|-----------|------------------------|
| area      | Scaffolds an Area      |
| controller| Scaffolds a controller |
| identity  | Scaffolds Identity     |
| razorpage | Scaffolds Razor Pages  |
| view      | Scaffolds a view       |

* Scaffold `Controller`,

  * Run the following command:

```
    dotnet-aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers

```

* Scaffolding Controller with Visual Studio

    1. In solution explorer, under `controller` folder, right click to add `New Scaffolded Item`-> select Common->API->`API Controller with actions, using Entity Framework`, then press `Add` button.  
    2. In the `Add API Controller with actions, using Entity Frame`, select `Data Context class`, and `Model class`, and confirm `Controller Name`, press `Add` button.
    3. After controller class successfully added, add more controller class by repeat 1, 2. 

### Authentication and Authorization

* [Authentication and Authorization in ASP.NET Web API](https://learn.microsoft.com/en-us/aspnet/web-api/overview/security/authentication-and-authorization-in-aspnet-web-api)

* [Use cookie authentication without ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie?view=aspnetcore-7.0#sign-out)
*[Generate JWT token Asp.Net Core Web API(video)](https://www.youtube.com/watch?v=wd5RQfrnaUU&t=19s&ab_channel=TipsByAnil)

#### LDAP

* [Deploy LDAP directory service with OpenLDAP Docker](https://medium.com/rahasak/deploy-ldap-directory-service-with-openldap-docker-8d9f438f1216)
* [A docker image to run OpenLDAP](https://github.com/osixia/docker-openldap)
* [LDAP authentication in ASP.NET Core MVC](https://decovar.dev/blog/2022/06/16/dotnet-ldap-authentication/)
* [Apache Directory Studio](https://directory.apache.org/studio/)

### Sql Server Report Service

* Report Viewer Control
  * [Use the WinForms ReportViewer Control](https://learn.microsoft.com/en-us/sql/reporting-services/application-integration/using-the-winforms-reportviewer-control?view=sql-server-ver16)
  * [Integrate Reporting Services Using Report Viewer Controls](https://learn.microsoft.com/en-us/sql/reporting-services/application-integration/integrating-reporting-services-using-reportviewer-controls?view=sql-server-ver16)

* SQL Server Report Services
  * [What is SQL Server Reporting Services (SSRS)?](https://learn.microsoft.com/en-us/sql/reporting-services/create-deploy-and-manage-mobile-and-paginated-reports?view=sql-server-ver16)
  * [SQL Server Reporting Services features supported by editions](https://learn.microsoft.com/en-us/sql/reporting-services/reporting-services-features-supported-by-the-editions-of-sql-server-2016?view=sql-server-ver16)

### Docker

* SQL Server Express in Docker
  * [Docker Express: Running a local SQL Server Express](https://medium.com/agilix/docker-express-running-a-local-sql-server-express-204890cff699)
  * [Microsoft SQL Server - Ubuntu based images](https://hub.docker.com/_/microsoft-mssql-server)
  * [Quickstart: Run SQL Server Linux Container image with Docker](https://learn.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver16&pivots=cs1-bash)

## Sub-Projects

### Membership Management

* Roles: Broker, Back Office, Account, Admin, Employee

#### zero-length string POST or PUT problem

##### Problem: Error happened when POST or PUT with EF(entity framework)

* Debug: use Swagger to test POST or PUT API, find the error message like,  

`The INSERT statement conflicted with the CHECK constraint "SSMA_CC$T_ART$artImg$disallow_zero_length".`

* Solution:  need to delete the constraints

```SQL
ALTER TABLE DROP CONSTRAINT SSMA_CC$T_ART$artImg$disallow_zero_length
```

in a query window.

##### Reason

* [MS SQL Migration Assistant handling MS Access zero-length string](https://braingrabber.wordpress.com/2013/01/07/ms-sql-migration-assistant-handling-ms-access-zero-length-string/)

The pitfall of Migration Assistant is the assumptions that will be made automatically. Always triple check (application, business and data wise) your constraints before you blindly apply them on your own data model. I thought applying a Zero-length check on a varchar(50) column was a good idea. The same column in Access had data type Text and the options “Allow Zero Length” & “Required” flagged no. So SSMA generated this constraint based on the “Allow Zero Length” option.

##### Use T-SQL to delete constraints with name `SSMA_CC` and end with `disallow_zero_length`

* This is generated by ChatGPT, with 3 update for errors and mistakes.

* Before exec the SQL, backup database first

```sql

USE FOREXDB; -- Replace "FOREXDB" with your actual database name

DECLARE @tableSchema NVARCHAR(255)
DECLARE @tableName NVARCHAR(255)
DECLARE @constraintName NVARCHAR(255)
DECLARE @sql NVARCHAR(MAX)

DECLARE constraintCursor CURSOR FOR
SELECT TABLE_SCHEMA, TABLE_NAME, CONSTRAINT_NAME
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE CONSTRAINT_NAME LIKE 'SSMA_CC%disallow_zero_length'

OPEN constraintCursor
FETCH NEXT FROM constraintCursor INTO @tableSchema, @tableName, @constraintName

WHILE @@FETCH_STATUS = 0
BEGIN
    SET @sql = 'ALTER TABLE ' + 
               QUOTENAME(@tableSchema) + '.' + 
               QUOTENAME(@tableName) + 
               ' DROP CONSTRAINT ' + QUOTENAME(@constraintName)

    EXEC sp_executesql @sql

    FETCH NEXT FROM constraintCursor INTO @tableSchema, @tableName, @constraintName
END

CLOSE constraintCursor
DEALLOCATE constraintCursor

```

#### Add additional column of table `Account` to connect to users in authentication database

* Add database table column in database management

```sql
USE FOREXDB; -- Use the database

-- Add the UserId column to the Account table
ALTER TABLE Account
ADD UserId NVARCHAR(450) NULL;

```

* Re-Generate MemberService entity framework models from `FOREXDB`, with Force flag

```pm
PM> Scaffold-DbContext "Data Source=localhost;Initial Catalog=FOREXDB;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;Integrated Security=SSPI;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -Force -Tables Account,Broker,Employee -OutPutDir Models 

```

* Re-Generate MemberService for additional 'Customer', and 'BusinessTbl'

```pm
PM> Scaffold-DbContext "Data Source=localhost;Initial Catalog=FOREXDB;Persist Security Info=False;MultipleActiveResultSets=True;Encrypt=False;Integrated Security=SSPI;TrustServerCertificate=true;Connection Timeout=30" Microsoft.EntityFrameworkCore.SqlServer -Force -Tables Account,Broker,Employee,Customer,BusinessTbl -OutPutDir Models 
```


### Name Service

* Banned List: From official, or private(by self)
* Checking changes and Update

### Broker

### Back Office

### Accounting

## System Architecture

* Backend
  * Database: MS SQL
  * Server: C# MicroService REST Web Service

* Frontend
  * WinForm
  * ASP.NET

## Database Sanitize

* MS-SQL database is converted be SSMA (SQL server Migration Assistant from Access)
* The database table contain extra column named "SSMA_TimeStamp".  We use the followging SQL to delete this column in each table.

File `DEL_SSMA_TimeStamp.SQL`

```sql
USE FOREXDB; -- Switch to the FOREXDB database

DECLARE @ColumnName NVARCHAR(128)
SET @ColumnName = 'SSMA_TimeStamp' -- Replace 'SSMA' with the column name you want to drop

DECLARE @TableName NVARCHAR(128)
DECLARE @SQL NVARCHAR(MAX)

-- DECLARE TableCursor CURSOR FOR
-- SELECT TABLE_NAME
-- FROM INFORMATION_SCHEMA.COLUMNS
-- WHERE COLUMN_NAME = @ColumnName

DECLARE TableCursor CURSOR FOR
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME = @ColumnName
AND TABLE_NAME NOT IN (
    SELECT TABLE_NAME 
    FROM INFORMATION_SCHEMA.VIEWS
)

OPEN TableCursor
FETCH NEXT FROM TableCursor INTO @TableName

WHILE @@FETCH_STATUS = 0
BEGIN
    -- Build the dynamic SQL statement to delete the specified column
    SET @SQL = N'ALTER TABLE ' + QUOTENAME(@TableName) + N' DROP COLUMN ' + QUOTENAME(@ColumnName) + N';'
    -- Print the current table being processed
    PRINT 'Processing table: ' + @TableName
    EXEC sp_executesql @SQL

    FETCH NEXT FROM TableCursor INTO @TableName
END

CLOSE TableCursor
DEALLOCATE TableCursor

```


