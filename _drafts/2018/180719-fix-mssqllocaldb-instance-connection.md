# 修复连接本地 MSSQL LocalDB Instance 连接错误

　　昨天需要在某 .NET 项目中做 Add-Migration 和 update-database 操作，然而还没开始就遇到了问题——本地数据库 LocalDB 无法连接。其弹窗报错为：

![Instance error](/assets/images/blog/mssqllocaldb-instance-error.png)

对 .NET 和 MSSQL 开发不是很熟悉，于是上网找了下解决方法。

## 删除注册表中的 UserInstance

　　打开注册表，定位到:

```txt
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Microsoft SQL Server\UserInstances
```

然后删除以 GUID 命名的对应报错 Instance 的项。

　　再重新连接，就没问题了。

　　参考：

- [MSSQLLocalDB Instance Issue](https://social.msdn.microsoft.com/Forums/windowsapps/en-US/aabaa5b7-05c8-4c0b-a559-bacbef0a41f4/mssqllocaldb-instance-issue?forum=sqlexpress)
- [LocalDB parent instance version invalid: MSSQL13E.LOCALDB](https://stackoverflow.com/questions/40022742/localdb-parent-instance-version-invalid-mssql13e-localdb)

## 版本冲突问题

　　还有一个问题，是在解决上面问题过程中偶然发现的——当我在命令行中查看 LocalDB 版本时，报错：

```txt
> sqlllocaldb v
Windows API call "RegGetValueW" returned error code: 0
```

　　网上一搜，也是注册表的问题——版本号冲突：

```txt
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL13E.LOCALDB\MSSQLServer\CurrentVersion
```

这里的版本是 13.1.4001.0，而：

```txt
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server Local DB\Installed Versions
```

的版本则是 13.0。将 13.0 修改为 13.1，问题解决。

　　参考：

- [SQL 2016 sp1 SQLLocalDB versions errors with "Windows API call "RegGetValueW" returned error code: 0."](https://social.msdn.microsoft.com/Forums/sqlserver/en-US/1257bf26-6ab0-416d-bf26-34f128f42248/sql-2016-sp1-sqllocaldb-versions-errors-with-quotwindows-api-call-quotreggetvaluewquot?forum=sqlexpress)

　　……

　　真是麻烦啊。

1807191254