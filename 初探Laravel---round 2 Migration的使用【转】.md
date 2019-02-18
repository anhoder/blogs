# 初探Laravel---round 2 Migration的使用【转】

<span style="color:red;">**\[文章转载自简书---饥渴计科极客杰铿，有所修改，[传送门](https://www.jianshu.com/p/a94391a78ace)\]**</span>

*Mission Start!*

## 1、创建Migration
创建新表的迁移文件：

```sh
php artisan make:migration create_users_table --create=users
```
创建修改表的迁移文件：

```sh
php artisan make:migration add_votes_to_users_table --table=users
```

命令执行完成会在/database/migrations中创建一些迁移文件，在这些文件中可以对表进行设计和修改。

## 2、创建表的列

如：

```php
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
});
```

以下为**部分**Migration方法及其对应的数据库数据类型

| 命令 | 描述 |
| --- | --- |
| mediumInteger($column, $autoIncrement = false, $unsigned = false) | MEDIUMINT类型 |
| integer($column, $autoIncrement = false, $unsigned = false) | INT类型 |
| bigInteger($column, $autoIncrement = false, $unsigned = false) | BIGINT类型 |
| increments($column) | 自增ID，INT类型 |
| bigIncrements($column) | 自增ID，BIGINT类型 |
| binary($column) | BLOB类型 |
| boolean($column) | BOOLEAN类型 |
| char($column, $length = null) | CHAR类型 |
| date($column) | DATE类型 |
| dateTime($column, $precision = 0) | DATETIME类型 |
| decimal($column, $total = 8, \$places = 2) | DECIMAL类型 |
| double($column, $total = null, \$places = null) | DOUBLE类型 |
| enum($column, array $allowed) | ENUM类型 |
| json($column) | JSON类型 |
| string($column, $length = null)  | VARCHAR类型 |

**字段默认为NOT NULL，若需要可为NULL请使用nullable()方法，如：$table->string('name', 20)->nullable();**


以上为部分方法，更多内容请查看这些方法的定义文件---Blueprint.php

## 3、修改表的列

将部分方法放在代码中演示

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 30)->change();               // 将长度修改为30
    $table->string('name', 30)->nullable()->change();   // 将name列修改为允许NULL
    $table->remameColumn('name', 'user_name');          // name列名修改为user_name，暂不支持enum类型列的重命名
    $table->dropColumn('name');                         // 删除列
    $table->dropColumn('name', 'name2', 'name3');       // 删除多个列
});
```

*Mission Complete!*

