# 初探Laravel---round 4 多对多关系模型

<!-- more -->

*Mission Start!*

刚接触到这个东西时有点懵逼，这什么呀？干嘛的？网上找了一些博客和文档，对这个东东有了初步的认识，所以写个 Blog 记录一下，同时加深自己的理解。

在Laravel的官方文档中，其用了`User-Role`关系作为例子，用户可能是多个角色，一个角色也有多个用户。作为一个合格的学僧，我就用`Student-Course`举个例子吧。

## 一、建立表

### students表
单个表的简历过程不是本节重点，不赘述。表的结构和数据为：

| id | name |
| --- | --- |
| 1 | Alan |
| 2 | Jane |
| 3 | Jamie |
| 4 | William |

### courses表
course表的结构与数据为：

| id | name |
| --- | --- |
| 1 | Math |
| 2 | English |
| 3 | Chinese |
| 4 | PHP |

### 关联表

使用命令`php artisan make:migration course_student_table --create=course_student`建立迁移文件。

> *关联表的命名规范：<span style="color:red;">① 表名需包含相关表表名；② 根据单词首字母排序；③ 单词一律小写；④ 单词之间用下划线连接</span>*

course_student表就是我们的关联表，称为 `pivot` 表（翻译：pivot, 枢轴; 中心点，中枢; [物] 支点，支枢;）。该表中需包含两个外键，分别是student_id、course_id，每个外键对应一个表。我们可以在这个表中添加更多字段，如：若添加 `score` 字段，需要使用到 `withPivot('col1', 'col2')` ，代码如下：

```php
public function courses()
{
    return $this->belongsToMany('App\Course')
            ->withPivot('score')
            ->withTimestamps();
} 
```

关联表的结构为：

| id | student_id | course_id | score |
| --- | --- | --- | --- |

## 二、建立多对多模型

接下来建立多对多关系，可以选择在任意一个主表中进行定义，取决于你的具体需求。

* 例如，获取一个学生的所有课程，则在 `/app/Student.php` 中创建多对多关系模型：

   ```php
   class Student extends Model
   {
       public function courses()
       {
           return $this->belongsToMany('App\Course');
       }
   }
   ```

* 再如：获取一门课程的全部选课学生，则在 `/app/Course.php` 中创建多对多关系模型：

   ```php
   class Course extends Model
   {
       public function students()
       {
           return $this->belongsToMany('App\Student');
       }
   }
   ```

> 需要注意的是：
>    
> <span style="color:red;">① 若关联表名未按照 Laravel 的默认方式进行命名，则需在belongsToMany()中传入第二个参数。</span>如：关联表名若为student_and_course，相关代码则应为 `return $this->belongsToMany('App\Student', 'student_and_course');`。
>    
> <span style="color:red;">② 若关联表中外键键名未采用默认规范进行命名，则需使用belongsToMany()的第三、四个参数。</span>如：若关联表中的外键名为s_id、c_id，相关代码则应为 `return $this->belongsToMany('App\Student', 'student_and_course', 's_id', 'c_id');`。
>     
> <span style="color:red;">③ 若关联方法名不是curses或students，则需要将对应的方法名作为第五个参数传入到belongToMany。</span>如：若方法名为 `courses_test()` ，则对应代码为 `return $this->belongsToMany('App\Student', 'student_and_course', 's_id', 'c_id', 'courses_test');`


## 三、多对多数据模型操作

### 1. 添加记录---attach()
① id为1的学生添加一门id为5的课程：

```php
$courses = Student::find($id = 1)->courses();
$courses->attach($id = 5);
```
② id为2的学生添加id分别1, 2, 3, 4的多门课程：

```php
$courses = Student::find($id = 2)->courses();
$courses->attach([$id1 = 1, $id2 = 2, $id3 = 3, $id4 = 4]);
```
③ 向id为1的学生添加id为5的课程成绩80（即，添加额外的属性列数据）：

```php
$courses = Student::find($id = 1)->courses();
$courses->attach($id = 1, ['score' => 80]); // 需要添加更多属性数据，往第二个参数数组中添加键值即可
```
④ 向id分别为1, 5, 12的学生添加id为2的课程成绩都为85：

```php
$courses = Student::find($id = 1)->courses();
$courses->attach([$id1 = 1, $id2 = 5, $id3 = 12], ['score' => 85]);
```

### 2. 删除记录---detach()

① 删除id为1的学生的所有课程记录：

```php
$courses = Student::find($id = 1)->courses();
$courses->detach();
```
② 删除id为1的学生课程为1, 2, 3, 4的课程：

```php
$courses = Student::find($id = 1)->courses();
$courses->detach([1, 2, 3, 4]);
```

### 3. 查询记录
① 输出id为1的学生的所有课程名（在Student中查询courses表的name字段）：

```php
$courses = Student::find($id = 1)->courses;     // 没有括号
foreach ($courses => $course) {
    echo $course->name, ' ';
}
```
② 输出id为1的学生的所有课程成绩（在Student中查询中间表course_student的score字段）：

```php
$courses = Student::find($id = 1)->courses;
foreach ($courses => $course) {
    echo $course->pivot->course, ' ';
}
```
③ 输出id为1的学生的id为2的课程成绩（在Student中对中间表course_student进行条件查询）：

```php
$courses = Student::find($id = 1)->courses()->newPivotStatement(); // 返回Builder对象
$res = $courses
    ->where('student_id', 1)
    ->where('course_id', 2)
    ->first();
echo $res->score;

```
### 4. 同步---sync()

删除id为1的学生的除1, 2, 3号外的所有课程：

```php
$courses = Student::find($id = 1)->courses();
$course->sync([1, 2, 3]);
```

> **以上方法参数中的变量（如：$id、$id1等）只起提示作用，可去除**
> 
> **顺便附上官方文档的传送门：[英文](https://docs.golaravel.com/docs/5.6/eloquent-relationships/#many-to-many)、[中文](http://laravelacademy.org/post/8867.html#toc_5)**

*Mission Complete!*

