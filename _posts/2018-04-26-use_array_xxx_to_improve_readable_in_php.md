---
layout: post
title: PHP 中使用 array_* 提升代码可读性
image: /assets/php/use_array_xxx.logo.png
category: [ php ]
---


在PHP中，数组作为一个语言的基础组成部分，发挥着及其重要的作用，可以当作列表，也可以当作字典，而我们平常使用数组的过程中充斥着 for、foreach，这样操作虽然能满足我们的需求，但是对未来的代码维护来说并不健康，因为等过一段时间再看时，会发现除了知道是循环，为什么这么写就很难回忆起来了，更何况这代码有可能是我写的呢.....

所以，类似于 C# 的 lambda 表达式，或者 Java 的匿名函数，以及 Objective-C 的 block， 我们在 PHP 中可以使用匿名函数来实现一些数组操作，具体涉及到的函数包括：

* array_filter
* array_map
* array_walk
* array_reduce

这4个函数威力巨大， 在处理列表数组方面可以完全替换掉for、foreach、while这些循环控制语句， 这也是函数式编程方式在PHP的一部份体现。

## 1. array_filter 函数

```php
$data = [
    ['id' => 1, 'name' => '张三', 'gender' => 'male'],
    ['id' => 2, 'name' => '李四', 'gender' => 'male'],
    ['id' => 3, 'name' => '王五', 'gender' => 'male'],
    ['id' => 4, 'name' => '赵六', 'gender' => 'male'],
    ['id' => 5, 'name' => '甲', 'gender' => 'female'],
    ['id' => 6, 'name' => '乙', 'gender' => 'female'],
    ['id' => 7, 'name' => '丙', 'gender' => 'female'],
    ['id' => 8, 'name' => '丁', 'gender' => 'female'],
];
$result = [];
foreach($data as $item) {
    if($item['gender'] == 'female') {
        array_push($result, $item);
    }
}
```
这段代码比较好理解，将数组中性别字段为女的数据项提取出来。 整段代码的逻辑大致如下

    1.定义result数组， 用来存放结果
    2.循环数组， 对每一个数据项进行条件判断， 查看其中的性别字段是否为 female
    3.如符合条件则放入result数组中

这是原汁原味的命令式程序代码。

如果data变量中的数据并非存放于php数组中， 而是存在于关系数库的表之中， 那何取得性别为女的数据结果呢？ 对于程序员来说这貌似是一个更加简单的问题，一句SQL语句就搞定了:

```sql
SELECT * FROM data WHERE gender = 'female';
```
显然， 利用SQL查询数据更加方便，意途也更加清晰，毕间一个SQL表达 式就将所有的程序逻辑都给表达了现来。这句SQL只表达了：“我需要性别为 female 的数据，至于怎么拿， 我不管”， 除了结果， 其它的它一概不知。
这样的结构在 PHP 中通过匿名函数也完全可以实现。

```php
$result = array_filter($data, function($item) {
    return $item['gender'] == 'female';
}
```

利用array_filter函数，可以轻松的完成这个任务， 仔细观察一下， 是不是原来的程序逻辑都不见了，包括定义数组、循环、条件判断这些都不见了，逻辑方面是只剩下了一个性别比较语句，这对于代码所实现的功能一目了然。 和上面的SQL比较一下， 这里的性别判断语句就是SQL中where子句后面的条件判断， 而array_filter函数其实就是SQL中的where子句。 这就是SQL语句面向结果编程的逻辑原封不变的在PHP中的体现，也就是时下最流行的“声明性编程”或者也称为“表达式编程”。

此外， 代码中性别判断语句所在的位置称之为 lambda 表达式， 更通俗一些的叫法是匿名函数。不难看出， 在 SQL 的 where 条件中编写条件判断远不如在匿名函数中写 PHP 代码来的灵活,在 where 条件中只能执行 or 和 and 逻辑，而在 PHP 匿名函数中可以随便怎么写，只要函数的返回值是个布尔值就可以了，这也是 PHP 声明性编程优于SQL声明性编程的地方。

另外，为了更加通用，具体的性别很有可能是在匿名函数外设置的，而 array_filter 的 callback 函数只支持一个参数，在语法层面上我们需要用 use 命令实现导入外层变量：

```php
$gender = 'female';
$result = array_filter($data, function($item) use($gender){
    return $item['gender'] == $gender;
}
```

## 2. array_map函数 

再来看一个例子

```php
$data = [
    ['id' => 1, 'name' => '张三', 'gender' => 'male'],
    ['id' => 2, 'name' => '李四', 'gender' => 'male'],
    ['id' => 3, 'name' => '王五', 'gender' => 'male'],
    ['id' => 4, 'name' => '赵六', 'gender' => 'male'],
    ['id' => 5, 'name' => '甲', 'gender' => 'female'],
    ['id' => 6, 'name' => '乙', 'gender' => 'female'],
    ['id' => 7, 'name' => '丙', 'gender' => 'female'],
    ['id' => 8, 'name' => '丁', 'gender' => 'female'],
];
$result = [];
foreach($data as $item) {
    $item['姓名'] = $item['name'];
    unset($item['name']);
    $item['性别'] = $item['gender'] == 'male' ? '男' : '女';
    unset($item['gender']);
    array_push($result, $item);
}
```

数据中的性别字段是中文的，值也是中文的， 现在想把字段名和字段值都改为英文的, 就可以用上面这段代码实现， 至于实现的逻辑这里不赘述了。

下面是利用SQL的实现方式

```sql
SELECT id, name AS 姓名, CASE gender WHEN 'male' THEN '男' ELSE '女' END AS 性别
```
SQL中case when语句好像不太好看， 但是不影响整体逻辑的表达。 将这段SQL转换成PHP的方式实现

```php
$result = array_map($data, function($item) {
    return ['id'=>$item['id'], '姓名'=>$item['name'], '性别'=>$item['gender'] == 'male' ? '男' : '女';
}
```

相比之前的PHP实现， 是不是简洁明了了许多。

在这里使用到了 array_map 函数 。 在 SQL 语句中以 select 语句最为常用， select 的字面意思是“选择”，而 select 语句也被称之为选择查询， 事实上从关系数据库的角度来说，select 被称之为“投影”， 并不是查询什么的。 换言之， select 语句只是将 SQL 的查询结果以一定的方式（选字段、计算值等等）提取出来了。 PHP 中的 array_map 表达的也是这层意思， “映射”与“投影”完全是一种意思的不同表达。

## 3. array_walk

## 4. array_reduce


当然，可读性提升的副作用就是性能的下降，建议以上代码写在注释里，方便后人理解循环的逻辑吧。X_X
