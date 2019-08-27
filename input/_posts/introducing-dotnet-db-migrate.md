layout: post
Id: b118e273-96c6-428b-aec3-5cda12ede69c
Published: 2018-09-15
Title: Introducing dotnet-db-migrate
Lead: A .NET Core global tool is a special NuGet package that contains a console application. Here I create a global tool for database migrations using db-up.
Author: Rich Tasker
permalink: /2018/09/15/introducing-dotnet-db-migrate
Tags:
  - db-up
  - Dotnet Core
  - Global Tools
---

### Introduction

I am a big fan of Db-up, if you haven't used it or want to learn more about it you can find more info about the project on their [GitHub](https://github.com/DbUp/DbUp).

When I have a solution that has an SQL DB as a data store I inevitably end up creating a `Migrations` project and setting up db-up. *For example* https://dbup.readthedocs.io/en/latest/#getting-started.

#### .Net Core Global Tools

I started doing that exact same thing on a personal home automation project. 
But then along came [dotnet core global tools](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools). I found Nate McMaster's [article](https://natemcmaster.com/blog/2018/05/12/dotnet-global-tools/) on how to get start writing your own global tools. This gave me the idea to wrap up my migrations project into a global tool so I would never have to write the same project again.

### dotnet-db-migrate

You can find dotnet-db-migrate on [GitHub](https://github.com/ritasker/dotnet-db-migrate) and [Nuget](https://www.nuget.org/packages/dotnet-db-migrate/).

#### Installing

You will need to have at least [.Net Core 2.1](https://www.microsoft.com/net/download) installed. Once installed, run...

```
dotnet tool install -g dotnet-db-migrate
```


#### Usage

Once installed, running `dotnet db-migrate -h` will print the following help information.
```
A tool to deploy changes to SQL databases.

Usage: dotnet-db-migrate [arguments] [options]

Arguments:
  ConnectionString    Required. The connection details for a database.

Options:
  -h|--help           Show help information
  -p|--provider       Optional. The connection provider. Default: mssql
  -s|--scripts        Optional. The path to the migration scripts. Default: scripts/
  --ensure-db-exists  Optional. Create the database if it doesn't exist. Default: false
```

Here is an example using a Azure SQL DB.
`dotnet db-migrate "Server=tcp:db-migrate.database.windows.net,1433;Initial Catalog=db-migrate;User ID=ritasker;Password=SuperSecure10;" -s ./scripts/MSSQL`

**Console Output:**
```
Beginning database upgrade
Checking whether journal table exists..
Journal table does not exist
Executing Database Server script '201806182121-Create-Contacts.sql'
Checking whether journal table exists..
Creating the [SchemaVersions] table
The [SchemaVersions] table has been created
Upgrade successful
Success!
```

And here are the tables the migrator created.

![database tables](https://github.com/ritasker/blog-post-images/raw/master/intro-dotnet-db-migrate/db-tables.png)


### Wrapping Up

Currently dotnet-db-migrate only supports MS-SQL and PostgreSQL. The tool is open source and I am accepting pull requests if you want to extend it to other databases.

I am really pleased with dotnet-db-migrate and the introduction of global tools. I can't wait to see what other tools people produce that help us develop better software.
