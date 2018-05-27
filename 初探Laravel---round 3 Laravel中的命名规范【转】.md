# 初探Laravel---round 3 Laravel中的命名规范【转】

<span style="color:red;">**\[文章转载自CSDN---多多说happy的博客，有所修改，[传送门](https://blog.csdn.net/da_guo_li/article/details/78680767)\]**</span>

*Mission Start!*

## 一、数据库命名

* 建议使用复数名词作为表名，如： `users` 
* 数据库及表名均使用小写英文字母，单词之间采用下划线连接，并以首字母排序，如： `course_student` 
* 多对多关系的中间表用单数作为表名，如： `course_student` 
* 不要使用关键字作为数据库或表的名字（ `null` 、 `select` 等）

## 二、文件命名

### Migration

* 迁移文件的文件名采用 `动词 + _TableName + _table` 的形式，如： `php artisan make:migration create_users_table` 
* 上述迁移文件文件名的 `TableName` 与迁移文件中的表名保持一致，如：创建迁移文件时使用参数指定表名 `php artisan make:migration create_users_table --create=users`，或者在文件中指明：
  
  ```sh
  Schema::create('users', function (Blueprint $table) {
  
  });
  ```

### Model

* 一律使用单数形式，如： `User` 、 `Question` 

### Seeder

*  `ModelName + TableSeeder` ，如： `UserTableSeeder` 

### resource下的文件

* 所有目录命名一律小写，单字间以下划线"_"分割


*Mission Complete!*

