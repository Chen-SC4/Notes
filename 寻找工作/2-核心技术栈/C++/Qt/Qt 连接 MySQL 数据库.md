# Qt 连接 MySQL 数据库

## 1. 准备工作

在进行数据库连接前，需要检查 `Qt` 是否包含有 `Sql` 模块，以及能否正确加载 `Sql Driver`。

1. 确保 `Qt Maintance Tool` 中勾选了 `Src` 和一种编译工具链。
	- ![](Qt%20连接%20MySQL%20数据库.assets/file-20250328132231394.png)
2. 运行 `qDebug() << QSqlDatabase::drivers();` 这行代码，保证输出不是 `()`，即至少有 `Sqlite` 的驱动。
	- `("QSQLITE", "QODBC", "QODBC3", "QPSQL", "QPSQL7")`
	- [QSqlDatabase::drivers() 输出为空](https://blog.csdn.net/xqf222/article/details/128565067)
	- 最后的方法是重装 `Qt`

在上面的两点都有保证之后，准备工作就算完成了。

## 2. 编译 MySQL 驱动

在 `Qt` 的 `Qt\5.14.2\Src\qtbase\src\plugins\sqldrivers\mysql` 目录下找到 `mysql.pro` 文件，用 `Qt Creator` 打开。

1. 将 `QMAKE_USE` 这一行注释掉
2. 添加 `INCLUDEPATH += $$quote($(MySQL的安装路径)/MySQL Server 8.0/include)`
3. 添加 `DEPENDPATH += $$quote($(MySQL的安装路径)/MySQL Server 8.0/include)`
4. 添加 `LIBS += -L$$quote($(MySQL的安装路径)/MySQL Server 8.0/lib) -llibmysql`

然后点击 `构建` (编译)。此时可能会提示：找不到 `qtsqldrivers-config.pri`，此时需要打开 `qsqldriverbase.pri` 文件，将其中的 `include($$shadowed($$PWD)/qtsqldrivers-config.pri)` 改为 `include(./configure.pri)`

然后再次编译，此时可能会提示 `No Such File xxx/qsqlmysql.dll`。不用理会，可以全局搜索一下 `qsqlmysql.dll`，如果能找到就说明生成成功了。

## 3. 使用/安装 MySQL 驱动

将得到的 `qsqlmysql.dll` 复制到 `$(Qt安装路径)\$(编译工具链)\plugins\sqldrivers` 目录下。

将 `$(MySQL安装路径)\MySQL Server 8.0\lib` 下的 `libmysql.dll` 复制到 `$(Qt安装路径)\$(编译工具链)\bin` 目录下。

然后再次运行 `qDebug() << QSqlDatabase::drivers();`，输出中应该会有：`QMYSQL`。