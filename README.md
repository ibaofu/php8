#1.联合类型 rfc
考虑到 PHP 动态语言类型的特性，现在很多情况下，联合类型都是很有用的。联合类型是两个或者多个类型的集合，表示可以使用其中任何一个类型。

public function foo(Foo|Bar $input): int|float;
请注意，联合类型中不包含 void，因为 void 表示的含义是 “根本没有返回值”。 另外，可以使用 |null 或者现有的 ? 表示法来表示包含 nullable 的联合体 ：

public function foo(Foo|null $foo): void;

public function bar(?Bar $bar): void;
JIT rfc
JIT — just in time — 编译器虽然不总是在 Web 请求的上下文中，但是有望显着地提高性能。目前还没有完成任何准确的基准测试，但是肯定会到来。

如果您想进一步了解 JIT 对 PHP 的作用，可以阅读我写过的另一篇文章此处。

属性 rfc
属性在其他语言中通常被称为 注解 ，提供一种在无需解析文档块的情况下将元数据添加到类中的方法。

快速浏览一下，这里有一份来自 RFC 的属性示例：

use App\Attributes\ExampleAttribute;

<<ExampleAttribute>>
class Foo
{
    <<ExampleAttribute>>
    public const FOO = 'foo';

    <<ExampleAttribute>>
    public $x;

    <<ExampleAttribute>>
    public function foo(<<ExampleAttribute>> $bar) { }
}





<<PhpAttribute>>
class ExampleAttribute
{
    public $value;

    public function __construct($value)
    {
        $this->value = $value;
    }
}
如果您想深入了解属性如何工作以及如何构建自己的属性，您可以在此博客上阅读有关深入属性的信息。

#2.新增 static 返回类型 rfc
尽管已经可以返回 self，但是 static 直到 PHP 8 才是有效的返回类型 。考虑到 PHP 具有动态类型的性质，此功能对于许多开发人员将非常有用。

class Foo
{
    public function test(): static
    {
        return new static();
    }
}
#3.新增 mixed 类型 rfc
有人可能将其称为必要的邪恶：mixed 类型让许多人感觉十分混乱。然而，有一个很好的论据支持去实现它：缺少类型在 PHP 中会导致很多情况：

函数不返回任何内容或返回空值
我们需要多种类型的一种类型
我们需要的是 PHP 中不能进行类型提示的类型
因为上述原因，添加 mixed 类型是一件很棒的事儿。mixed 本身代表下列类型中的任一类型：

array
bool
callable
int
float
null
object
resource
string
请注意，mixed 不仅仅可以用来作为返回类型，还可以用作参数和属性类型。



另外，还需要注意，因为 mixed 类型已经包括了 null，因此 mixed 类型不可为空。下面的代码会触发致命错误：

// 致命错误：混合类型不能为空，null已经是混合类型的一部分。
function bar(): ?mixed {}
throw 表达式 rfc
该 RFC 将 throw 从一个语句更改为一个表达式，这使得可以在很多新地方抛出异常：

$triggerError = fn () => throw new MyError();

$foo = $bar['offset'] ?? throw new OffsetDoesNotExist('offset');
弱映射 rfc



基于在 PHP 7.4 中新增的 弱引用 RFC，PHP 8 中新增了 WeakMaps（弱映射）的实现。 WeakMaps（弱映射）在保持对一些对象的引用的同时，并不会组织这些对象被垃圾回收机制处理 。

以 ORM 为例，它们通常实现保存对实体类的引用的缓存，从而提高实体类之间关联的性能。 只要缓存中存在对这些实体类的引用，那么这些类就无法被垃圾回收机制回收，尽管除了缓存中，已经没有别处再引用这些实体类，它们依然不会被垃圾处理机制处理。

如果这个缓存层使用了弱引用和弱映射，那么 PHP 将会在这些实体类没有任何其他引用时，对其进行垃圾回收。 尤其是对于 ORMs，它可以管理一个请求中的数百个 (如果不是数千个) 实体；弱映射可以提供一种更好的、对资源更友好的方式来处理这些对象。

下面是弱映射基本的例子，摘抄自 RFC ：

class Foo 
{
    private WeakMap $cache;

    public function getSomethingWithCaching(object $obj): object
    {
        return $this->cache[$obj]
           ??= $this->computeSomethingExpensive($obj);
    }
}
允许对对象使用 ::class rfc


一个很小但是很有用的新特性：现在可以在对象上使用 :: class ，而不必在对象上使用 get_class() ，它的工作方式跟 get_class() 相同。

$foo = new Foo();

var_dump($foo::class);
Non-capturing catches rfc


在 PHP 8 之前，无论何时你想要捕获一个异常，你都需要先将其存储到一个变量中，不管这个变量你是否会用到。通过 Non-capturing catches 你可以忽略变量，所以替换下面的代码：

try {
    // Something goes wrong
} catch (MySpecialException $exception) {
    Log::error("Something went wrong");
}


你现在可以这么做：

try {
    // Something goes wrong
} catch (MySpecialException) {
    Log::error("Something went wrong");
}
请注意，必须始终指定类型，不允许将 catch 留空，如果你想要捕获所有类型的异常和错误，需要使用 Throwable 作为捕获类型。

参数列表中的尾部逗号 rfc
当调用函数时已经支持尾部逗号，但是参数列表中仍然缺少尾随逗号支持。现在 PHP8 中允许这样做，这意味着您可以执行以下操作：

public function(
    string $parameterA,
    int $parameterB,
    Foo $objectfoo,
) {
    // …
}
从接口创建 DateTime 对象
你已经可以使用 DateTime::createFromImmutable($immutableDateTime) 从 DateTimeImmutable 对象创建一个 DateTime 对象， 而另一种方法则更加取巧。通过添加 DateTime::createFromInterface() 和 DatetimeImmutable::createFromInterface() 现在有一种通用的方法可以将 DateTime 和 DatetimeImmutable 对象相互转换。

DateTime::createFromInterface(DateTimeInterface $other);

DateTimeImmutable::createFromInterface(DateTimeInterface $other);
#4.新增 Stringable 接口 rfc
Stringable 接口可用于键入提示任何字符串或实现__ toString() 的内容。此外，每当一个类实现__ toString() 时，它就会自动实现后台接口，而无需手动实现。

class Foo
{
    public function __toString(): string
    {
        return 'foo';
    }
}

function bar(Stringable $stringable) { /* … */ }

bar(new Foo());
bar('abc');
#5.新增 str_contains() 函数 rfc
有些人可能会说这是早该发生的，但我们最终不必再依赖 strpos 来知道一个字符串是否包含另一个字符串。

无需这样做：

if (strpos('string with lots of words', 'words') !== false) { /* … */ }
你可以这样做：

if (str_contains('string with lots of words', 'words')) { /* … */ }
新增 str_starts_with() 和 str_ends_with() 函数 rfc
这是另外两个早该出现的函数，现在已在核心函数中添加了这两个函数。

str_starts_with('haystack', 'hay'); // true
str_ends_with('haystack', 'stack'); // true
#6.新增 fdiv() 函数 pr
新的 fdiv() 函数的作用类似于 fmod() 和 intdiv() 函数，它们可以除以 0。视情况而定，将得到 INF，-INF 或 NAN。



#7.新增 get_debug_type() 函数 rfc
get_debug_type() 返回变量的类型，听起来好像跟 gettype() 的作用一样啊？get_debug_type() 可以为数组，字符串，匿名类和对象返回更有用的输出信息。

例如，在类 \ Foo \ Bar 上调用 gettype() 将返回 object，而使用 get_debug_type() 将返回类名。

如下表：


可以在 RFC 中找到 get_debug_type() 和 gettype() 之间的差异的完整列表。

#8.新增 get_resource_id() 函数 pr
资源是 PHP 中的特殊变量，指的是外部资源。一个示例是 MySQL 连接，另一个是文件句柄。

这些资源中的每一个都分配有一个 ID，然而在这之前，如果想获取某资源的 ID，唯一方法是将资源转换为 int：

$resourceId = (int) $resource;
PHP 8 添加了 get_resource_id() 函数，使此操作更加明显且类型安全：

$resourceId = get_resource_id($resource);
Traits 改进中的抽象方法 rfc
Traits 可以指定必须由使用它们的类所实现的抽象方法。需要注意的是： 在 PHP 8 之前，尚未验证这些方法已经实现的标识。以下内容有效：

trait Test {
    abstract public function test(int $input): int;
}

class UsesTrait
{
    use Test;

    public function test($input)
    {
        return $input;
    }
}
当使用 Traits 并实现其抽象方法时，PHP 8 将执行适当的方法进行标识验证抽象方法是否确实被实现。这意味着您需要编写以下代码：

class UsesTrait
{
    use Test;

    public function test(int $input): int
    {
        return $input;
    }
}
token_get_all() rfc 的对象实现

token_get_all() 函数返回一个值数组，该 RFC 使用 PhpToken :: getAll() 方法新增了 PhpToken 类。此实现适用于对象而不是普通值。它消耗更少的内存，并且更易于阅读。



可变语法调整 rfc
在 RFC 中：“统一变量语法 RFC 解决了 PHP 变量语法中的许多不一致之处。该 RFC 旨在解决一小部分被忽略的情况。”

内部函数的类型注解 externals
许多人 投入 了为所有内部函数添加适当的类型注释的工作。这是一个长期存在的问题，最终可以通过以前版本中对 PHP 所做的所有更改来解决。这意味着内部函数和方法将在反射中具有完整的类型信息。

#9.重大变化
如前所述：这是一个重大更新，因此会有重大变化。最好的办法是查看 升级 文档中所列的重大变化的完整列表。

许多这些突破性的更改在以前的 7.* 版本中已被弃用，因此如果你多年来一直保持 PHP 在最新状态，升级到 PHP 8 应该没那么难。

一致的类型错误 rfc
之前版本在出现类型错误时，PHP 中的用户定义函数已经会抛出 TypeErrors，但是内部函数不会这么做，而是发出警告并返回 null。从 PHP 8 开始，内部函数的行为已变得和用户定义函数一致。

重新分类的引擎警告 rfc
许多以前仅触发警告或通知的错误已转换为适当的错误。以下警告已更改。

变量未定义：Error 异常代替通知
数组索引未定义：警告代替通知
除以零：DivisionByZeroError 异常代替警告
尝试添加 / 移除非对象的属性 '% s' ：Error 异常代替警告
尝试修改非对象的属性 '% s' ：Error 异常代替警告
尝试分配非对象的属性 '% s' ：Error 异常代替警告
从空值创建默认对象：Error 异常代替警告
尝试获取非对象的属性 '% s' ：警告代替通知
未定义的属性：% s::$% s：警告代替通知
无法添加元素到数组，因为下一个元素已被占用：Error 异常代替警告
无法在非数组变量中销毁偏移量：Error 异常代替警告
无法将标量值用作数组：Error 异常代替警告
只有数组和 Traversables 可以被解包：TypeError 异常代替警告
为 foreach () 提供了无效的参数：TypeError 异常代替警告
偏移量类型非法：TypeError 异常代替警告
isset 或 empty 中的偏移量类型非法：TypeError 异常代替警告
unset 中的偏移量类型非法：TypeError 异常代替警告
数组到字符串的转换：警告代替通知
资源 ID#% d 用作偏移量，转换为整数 (% d)：警告代替通知
发生字符串偏移量转换：警告代替通知
未初始化的字符串偏移量：% d：警告代替通知
无法将空字符串分配给字符串偏移量：Error 异常代替警告
提供的资源不是有效的流资源：TypeError 异常代替警告


@ 运算符不再使致命错误不提醒
此更改可能会使 PHP 8 之前的版本被 @ 隐藏的错误再次显示出来。请确保在生产服务器上设置了 display_errors=Off ！

默认错误报告级别
现在的默认错误报告级别是 E_ALL 而不是之前的除 E_NOTICE 和 E_DEPRECATED 的所有内容。这意味着可能会弹出许多错误，这些错误以前曾被忽略，尽管在 PHP 8 之前的版本中可能已经存在。

默认 PDO 错误模式 rfc
根据 RFC：当前 PDO 的默认错误模式为静默。这意味着当出现 SQL 错误时，除非开发人员实现了自己的显式错误处

理，否则不会发出任何错误或警告，也不会引发任何异常。

此 RFC 将在 PHP 8 中将默认 PDO 错误模式 改为 PDO::ERRMODE_EXCEPTION 。

串联优先级 rfc
在 PHP 7.4 中已废弃的同时，此变更开始生效。如果你像这样子书写：

echo "sum: " . $a + $b;
PHP 以前会如是理解：

echo ("sum: " . $a) + $b;
PHP 8 将这么做故理解为此：

echo "sum: " . ($a + $b);
更严格的算术和位运算类型检查 rfc
PHP 8 以前，算术或位运算符用于数组、资源或对象是可接受的。现在不再可接受，并会抛出一个 类型错误 ：

[] % [42];
$object + 4;
反射方法签名变更
反射类的 3 个方法签名已变更：

ReflectionClass::newInstance($args);
ReflectionFunction::invoke($args);
ReflectionMethod::invoke($object, $args);
现在已变成：

ReflectionClass::newInstance(...$args);
ReflectionFunction::invoke(...$args);
ReflectionMethod::invoke($object, ...$args);
升级指南指定，如果要扩展这些类，并且仍想同时支持 PHP 7 和 PHP 8，则允许以下签名：

ReflectionClass::newInstance($arg = null, ...$args);
ReflectionFunction::invoke($arg = null, ...$args);
ReflectionMethod::invoke($object, $arg = null, ...$args);
几个弃用
在 PHP 7. * 的开发期间，添加了几个弃用版本，这些弃用已于 PHP 8 最终确定。

PHP 7.2 中的弃用
PHP 7.3 中的弃用
PHP 7.4 中的弃用
原文地址：https://stitcher.io/blog/new-in-php-8
译文地址：https://learnku.com/php/t/44904
