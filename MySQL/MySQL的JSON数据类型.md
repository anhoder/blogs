# MySQL的JSON数据类型

<!-- more -->

MySQL在5.7.8版本中增加了对json数据的支持，而不再是需要使用字符串形式进行存储。下面简单介绍下MySQL对json的操作：

<h2>1、数据类型--json</h2>

MySQL使用的字段数据类型就是<span style="color: blue;">json</span>，例如（字段test_json）：

```sql
create table `test_json`(
    id int not null auto_increment primary key, 
    test_json json not null
);
```


<h2>2、MySQL的json相关函数</h2>

<h3>① json_array()</h3>
<strong>作用：</strong>将数组转化为json格式数据。

<strong>语法：</strong>

```sql
/*为下一行打辅助*/
json_array(value1[, value2[, value3[,...]]])
```

<strong>例如：</strong>

```sql
insert into `test_json` values (
    null,
    json_array('Alan','Jane','Jack','Rose')
);
```

<strong>结果：</strong>

```sql
+----+----------------------------------+
| id | test_json                        |
+----+----------------------------------+
|  1 | ["Alan", "Jane", "Jack", "Rose"] |
+----+----------------------------------+
```

<h3>② json_object()</h3>
<strong>作用：</strong>将对象转化为json格式数据。

<strong>语法：</strong>

```sql
/*为下一行打辅助*/
json_object(key1, value1[, key2, value2[, key3, value3[, ...]]]);
```

<strong>例如：</strong>

```sql
insert into `test_json` values (
    null,
    json_object('name', 'Alan', 'age', '18', 'desc', 'handsome')
);
```

<strong>结果：</strong>

```sql
+----+-------------------------------------------------+
| id | test_json                                       |
+----+-------------------------------------------------+
|  2 | {"age": 18, "desc": "handsome", "name": "Alan"} |
+----+-------------------------------------------------+
```

<h3>③ json_quote()</h3>
<strong>作用：</strong>将字符串转化为json数据格式（为字符串增加双引号以及为引号增加转义字符）
<strong>语法：</strong>

```sql
/*为下一行打辅助*/
json_quote(value);
```

<strong>例如：</strong>

```sql
insert into `test_json` values(
    null,
    json_quote("Hello World!")
),(
    null,
    json_quote('He say, "Hello World!"')
);
```

<strong>结果：</strong>

```sql
+----+----------------------------+
| id | test_json                  |
+----+----------------------------+
|  6 | "Hello World!"             |
|  7 | "He Say, \"Hello World!\"" |
+----+----------------------------+
```

<h3>④ json_unquote()</h3>
<strong>作用：</strong>与json_quote()作用相反
<strong>语法：</strong>

```sql
/*为下一行打辅助*/
json_unquote(value);
```

<strong>例如：</strong>

```sql
select id, json_unquote(test_json) as test_json 
from `test_json` 
where id=6 or id=7;
```

<strong>结果：</strong>

```sql
+----+------------------------+
| id | test_json              |
+----+------------------------+
|  6 | Hello World!           |
|  7 | He Say, "Hello World!" |
+----+------------------------+
```

<h3>⑤ json_merge()</h3>
<strong>作用：</strong>将多个json文本合并成一个json文本
<strong>语法：</strong>

```sql
/*助攻行*/
json_merge(json1, json2[,json3[, json4[, ...]]]);
```

<strong>例如：</strong>

```sql
insert into `test_json` values(
    null,
    json_merge('["Alan","Jane"]', '["Jack", "Rose"]', '["other"]')
);
```

<strong>结果：</strong>

```sql
+----+-------------------------------------------+
| id | test_json                                 |
+----+-------------------------------------------+
| 10 | ["Alan", "Jane", "Jack", "Rose", "other"] |
+----+-------------------------------------------+
```

<h3>⑥ json_valid()</h3>
<strong>作用：</strong>判断json格式是否正确
<strong>语法：</strong>

```sql
/*助攻行*/
json_valid(json);
```

<strong>例如：</strong>

```sql
/*助攻行*/
select *,json_valid(test_json) as IsValid from test_json;
```

<strong>结果：</strong>

```sql
+----+-------------------------------------------------+---------+
| id | test_json                                       | IsValid |
+----+-------------------------------------------------+---------+
|  1 | ["Alan", "Jane", "Jack", "Rose"]                |       1 |
|  2 | {"age": 18, "desc": "handsome", "name": "Alan"} |       1 |
|  4 | "Hello World!"                                  |       1 |
|  5 | "He Say, 'Hello World!'"                        |       1 |
|  6 | "Hello World!"                                  |       1 |
|  7 | "He Say, \"Hello World!\""                      |       1 |
|  8 | ["Alan", "Jane", "Jack", "Rose", "other"]       |       1 |
|  9 | ["Alan", "Jane", "Jack", "Rose", "other"]       |       1 |
| 10 | ["Alan", "Jane", "Jack", "Rose", "other"]       |       1 |
+----+-------------------------------------------------+---------+
```

<h3>⑦ json_type()</h3>
<strong>作用：</strong>判断json文本的数据类型
<strong>语法：</strong>

```sql
/*助攻行*/
json_type(jsonData);
```

<strong>例如：</strong>

```sql
/*助攻行*/
select *,json_type(test_json) as json_type from test_json;
```

<strong>结果：</strong>

```
+----+-------------------------------------------------+-----------+
| id | test_json                                       | json_type |
+----+-------------------------------------------------+-----------+
|  1 | ["Alan", "Jane", "Jack", "Rose"]                | ARRAY     |
|  2 | {"age": 18, "desc": "handsome", "name": "Alan"} | OBJECT    |
|  4 | "Hello World!"                                  | STRING    |
|  5 | "He Say, 'Hello World!'"                        | STRING    |
|  6 | "Hello World!"                                  | STRING    |
|  7 | "He Say, \"Hello World!\""                      | STRING    |
|  8 | ["Alan", "Jane", "Jack", "Rose", "other"]       | ARRAY     |
|  9 | ["Alan", "Jane", "Jack", "Rose", "other"]       | ARRAY     |
| 10 | ["Alan", "Jane", "Jack", "Rose", "other"]       | ARRAY     |
+----+-------------------------------------------------+-----------+
```

<span style="color:#ed0617;">暂时只写这7个函数，MySQL还有许多关于json的函数，例如：json_append(), json_array_append(), json_array_insert(), json_insert(),  json_remove(), json_replace(), json_set() 等</span>