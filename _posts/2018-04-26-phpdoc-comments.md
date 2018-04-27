---
layout: post
title: 如何使用 PHPDoc 写注释
image: /assets/php/phpdoc.logo.png
category: [ php ]
---

注释是一定要写的，但要怎么写才能使开发者与工具都看得懂呢？PHPDoc是目前PHP注释的标准，提供了统一的注释写法，并且获得 PhpStorm 的支持，PHPUnit也使用PHPDoc，事实上 ThinKPHP 也是使用 PHPDoc 格式来写注释。

## 为什么要写注释？

1. 使用人话解释
使用人话描述显示逻辑、商业逻辑与资料库逻辑，避免日后需求改变须维护时，自己或其他人看不懂而无法维护。
2. 让 IDE 得知型别
PHP 是弱型别语言，可以使用 `type hint`，也可使用 `duck type`，当使用 `duck type` 时，只有在 runtime 才能得知型别，IDE 无法从 design time 得知型别，因此无法实现代码自动完成与代码检查，必须依赖 PHPDoc 描述类和变量，这也是PHP比其他语言都更依赖注释的原因。

## 如何建立 PHPDoc？

1. 在 PhpStorm 输入 `/**`，然后按下 `↩`，PhpStorm 会自动依据当时的游标的位置产生适当的 `PHPDoc blocks`。
2. ***Code->Generate*** 或按 `Ctrl + N`，会产生 `Generate` 选单，选择 `PHPDoc Blocks`。

![](/asset/php/phpdoc000.png)

3. 在适当时机按下 `Alt + ↩`，会出现 `Generate PHPDoc for ...`。如刚建立完 class, property 或 method 时。

![](/assets/php/phpdoc001.png)

## 文件注释

新建 PHP Class 文件时，PhpStorm 默认会在文件头部插入文件注释。

![](/assets/php/phpdoc002.png)

```
Post model 的 repository
```
第1行使用中文描述整个文件(整个 class)的功能。

之后会有一空行。

### @version

```
@version 0.1.0
```

目前代码的版本，一开始为0.1.0。建议采用 Major.Minor.Patch 的版本命名方式，也就是 Breaks.Features.Fixes。

### @author
```
@author slobber nandiw@purecode.cn
```

代码建立时的原作者。第一个参数为姓名，第二个参数为 email。

### @date
```
@date 11/22/15
```

代码的建立日期。

### @since
```
@since 0.1.0 11/22/15 slobber: 呵呵
```
代码的改版记录。第一个参数为版本号，第二个参数为修改日期，第三个参数为修改者姓名，后面加上`:`，再加上简易的修改内容。

若有很多版本，就继续加上多个 @since。

## Class 注释
建立 PHP Class 时，PhpStorm 自动会在 class 之前插入 class 注释。

![](/assets/php/phpdoc003.png)

```
Class PostRepository
```

第1行PhpStorm预设使用Class加上class名称当注释，由于one class per file原则，所以class的注释就相当于档案的注释，因此不用再特别叙述。

之后会空一行。

### @package

```
@package App\Repositories
```

描述class所属的namespace，PhpStorm会自动建立。

### @method

```
namespace Illuminate\Support;

/**
 * @method Fluent first()
 * @method Fluent after($column)
 * @method Fluent change()
 * @method Fluent nullable()
 * @method Fluent unsigned()
 * @method Fluent unique()
 * @method Fluent index()
 * @method Fluent primary()
 * @method Fluent default($value)
 * @method Fluent onUpdate($value)
 * @method Fluent onDelete($value)
 * @method Fluent references($value)
 * @method Fluent on($value)
 */
class Fluent {}
```

平常不会使用，除非使用 `__call()` 或 `__callStatic()` 动态产生method，由于这种写法 PhpStorm 的 auto complete 抓不到，因次要使用 PHPDoc 特别描述。

### @property
```
/**
 * Company
 *
 * @property integer $id
 * @property string $name
 * @property string $pairing_code
 * @property integer $nation_region_id
 * (略)
*/
```
平常不会使用，除非使用 `__get()` 与 `__set()` 动态产生 property，由于这种写法 PhpStorm 的 auto complete 抓不到，因此要使用 PHPDoc 特别描述。

## Property注释

当建立property时，在property上方输入 ```/** + ↩```，PhpStorm 会自动产生 property 的注释。

![](/assets/php/phpdoc004.png)

### @var

```
@var Post 注入的post model
```
对每个 property 做注释。

第1个参数为 property 的类型，由于 PHP 允许 duck type，这会导致 PhpStorm 的 auto complete 无法执行，所以一定要要输入类型。

第2个参数为 property 的注释，使用中文描述该 property 的意义。

## Method注释

1. 当建立 method 时，在 method 名称后面按 `Alt + ↩`，PhpStorm 会自动根据参数创建 method 的 PHPDoc blocks。
2. 当使用 `Ctrl + shift + I` 实现 interface 的 method 时，只要 interface 有写好注释，只要在 Add PHPDoc 选择 Copy from base class，就可以将 interface 所写好的注释複製过来。


```
回传最新3笔文章
```
第1行使用中文叙述整个method的功能。

之后会空一行。

### @param
```
@param string $field 排序栏位
```
对method的每个参数做注释。

第1个参数为传入参数的类别。
第2个参数为传入参数的变量。
第3个参数为传入参数的描述。

### @return
```
@return Collection 最新3笔文章
```
对method的返回值做注释。

第1个参数为返回值的型别。
第2个参数为返回值的描述。

> 为什么 @param 与 @return 对 PhpStorm 的 Auto Complete 与 Inspection 这么重要？
> 由于 PHP 允许 duck type，因此 method 的参数可以不写类型，但这样使得 PhpStorm 无法在 design time 得知该参数的类型，因此 auto complete 与 inspection 无法执行，所以建议在 PHPDoc block 一定要加上 @param 描述参数类型，@return 描述返回值类i系那个。此外，若参数为对象，也建议加上 class 或 interface 的 type hint，一来 PhpStorm 的 auto complete与 inspection 可以抓到，二来在 unit test 时，可以使用 service container 的 dependency injection。

### @throws
```
@throws InvalidArgumentException 若提供的参数不是array类型
```
若method有使用exception，可使用@throws描述。

第1个参数为exception的类型。
第2个参数为该exception的描述。

### @todo
```
@todo 此段代码尚未完成，建议有时间时加以重构
```
若method尚有功能未完成，可写在 @todo 里。

### @since
若之后该method因bug或需求变更而需要维护，使用@since描述其变更，与挡头注释的@since写法一样。

### @deprecated
若该API或public method即将在未来版本停用，可加上@depricated，后面加上注释，PhpStorm对于@depricated有特别的支持。

![](/assets/php/phpdoc007.png)

调用时 method 上会加删除线。

![](/assets/php/phpdoc008.png)

## Method 内注释

当 PhpStorm 无法得知该变数的型别，而导致auto complete与inspection无法执行时，就必须自行使用@var去描述该变量的类型。

![](/assets/php/phpdoc006.png)

### @var
最典型的用法是 ThinkPHP 的 Model 传回一个对象，但我们知道他本质上是某个类型的 model，但 PhpStorm 并不知道，因此我们必须自行使用 @var 加以描述。
