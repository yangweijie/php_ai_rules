~~~html


{
    if (empty($day)) {
        return false;
    }

    $openingDays = ['friday', 'saturday', 'sunday'];

    return in_array(strtolower($day), $openingDays, true);
}
~~~

### 避免深层嵌套，尽早返回 (part 2)

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E6%B7%B1%E5%B1%82%E5%B5%8C%E5%A5%97%E5%B0%BD%E6%97%A9%E8%BF%94%E5%9B%9E-part-2)

**糟糕的:**

~~~html
function fibonacci(int $n)
{
    if ($n < 50) {
        if ($n !== 0) {
            if ($n !== 1) {
                return fibonacci($n - 1) + fibonacci($n - 2);
            }
            return 1;
        }
        return 0;
    }
    return 'Not supported';
}
~~~

**好:**

~~~html
function fibonacci(int $n): int
{
    if ($n === 0 || $n === 1) {
        return $n;
    }

    if ($n >= 50) {
        throw new Exception('Not supported');
    }

    return fibonacci($n - 1) + fibonacci($n - 2);
}
~~~

### 少用无意义的变量名

[](https://github.com/php-cpm/clean-code-php#%E5%B0%91%E7%94%A8%E6%97%A0%E6%84%8F%E4%B9%89%E7%9A%84%E5%8F%98%E9%87%8F%E5%90%8D)

别让读你的代码的人猜你写的变量是什么意思。 写清楚好过模糊不清。

**坏:**

~~~html
$l = ['Austin', 'New York', 'San Francisco'];

for ($i = 0; $i < count($l); $i++) {
    $li = $l[$i];
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
  // 等等, `$li` 又代表什么?
    dispatch($li);
}
~~~

**好:**

~~~html
$locations = ['Austin', 'New York', 'San Francisco'];

foreach ($locations as $location) {
    doStuff();
    doSomeOtherStuff();
    // ...
    // ...
    // ...
    dispatch($location);
}
~~~

### 不要添加不必要上下文

[](https://github.com/php-cpm/clean-code-php#%E4%B8%8D%E8%A6%81%E6%B7%BB%E5%8A%A0%E4%B8%8D%E5%BF%85%E8%A6%81%E4%B8%8A%E4%B8%8B%E6%96%87)

如果从你的类名、对象名已经可以得知一些信息，就别再在变量名里重复。

**坏:**

~~~html
class Car
{
    public $carMake;

    public $carModel;

    public $carColor;

    //...
}
~~~

**好:**

~~~html
class Car
{
    public $make;

    public $model;

    public $color;

    //...
}
~~~

## 表达式

[](https://github.com/php-cpm/clean-code-php#%E8%A1%A8%E8%BE%BE%E5%BC%8F)

### [使用恒等式](http://php.net/manual/en/language.operators.comparison.php)

[](https://github.com/php-cpm/clean-code-php#%E4%BD%BF%E7%94%A8%E6%81%92%E7%AD%89%E5%BC%8F)

**不好:**

简易对比会将字符串转为整形

~~~html
$a = '42';
$b = 42;

if ($a != $b) {
   //这里始终执行不到
}
~~~

对比 $a != $b 返回了`FALSE`但应该返回`TRUE`! 字符串 '42' 跟整数 42 不相等

**好:**

使用恒等判断检查类型和数据

~~~html
$a = '42';
$b = 42;

if ($a !== $b) {
    // The expression is verified
}
~~~

The comparison`$a !== $b`returns`TRUE`.

### Null合并运算符

[](https://github.com/php-cpm/clean-code-php#null%E5%90%88%E5%B9%B6%E8%BF%90%E7%AE%97%E7%AC%A6)

Null合并运算符是[PHP 7新特性](https://www.php.net/manual/en/migration70.new-features.php). Null合并运算符`??`是用来简化判断`isset()`的语法糖。如果第一个操作数存在且不为`null`则返回；否则返回第二个操作数。

**不好:**

~~~html
if (isset($_GET['name'])) {
    $name = $_GET['name'];
} elseif (isset($_POST['name'])) {
    $name = $_POST['name'];
} else {
    $name = 'nobody';
}
~~~

**好:**

~~~html
$name = $_GET['name'] ?? $_POST['name'] ?? 'nobody';
~~~


## 函数

[](https://github.com/php-cpm/clean-code-php#%E5%87%BD%E6%95%B0)

### 合理使用参数默认值，没必要在方法里再做默认值检测

[](https://github.com/php-cpm/clean-code-php#%E5%90%88%E7%90%86%E4%BD%BF%E7%94%A8%E5%8F%82%E6%95%B0%E9%BB%98%E8%AE%A4%E5%80%BC%E6%B2%A1%E5%BF%85%E8%A6%81%E5%9C%A8%E6%96%B9%E6%B3%95%E9%87%8C%E5%86%8D%E5%81%9A%E9%BB%98%E8%AE%A4%E5%80%BC%E6%A3%80%E6%B5%8B)

**不好:**

不好，`$breweryName`可能为`NULL`.

~~~html
function createMicrobrewery($breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
~~~

**还行:**

比上一个好理解一些，但最好能控制变量的值

~~~html
function createMicrobrewery($name = null): void
{
    $breweryName = $name ?: 'Hipster Brew Co.';
    // ...
}
~~~

**好:**

如果你的程序只支持 PHP 7+, 那你可以用[type hinting](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)保证变量`$breweryName`不是`NULL`.

~~~html
function createMicrobrewery(string $breweryName = 'Hipster Brew Co.'): void
{
    // ...
}
~~~

### 函数参数（最好少于2个）

[](https://github.com/php-cpm/clean-code-php#%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0%E6%9C%80%E5%A5%BD%E5%B0%91%E4%BA%8E2%E4%B8%AA)

限制函数参数个数极其重要，这样测试你的函数容易点。有超过3个可选参数参数导致一个爆炸式组合增长，你会有成吨独立参数情形要测试。

无参数是理想情况。1个或2个都可以，最好避免3个。再多就需要加固了。通常如果你的函数有超过两个参数，说明他要处理的事太多了。 如果必须要传入很多数据，建议封装一个高级别对象作为参数。

**坏:**

~~~html
class Questionnaire
{
    public function __construct(
        string $firstname,
        string $lastname,
        string $patronymic,
        string $region,
        string $district,
        string $city,
        string $phone,
        string $email
    ) {
        // ...
    }
}
~~~

**好:**

~~~html
class Name
{
    private $firstname;

    private $lastname;

    private $patronymic;

    public function __construct(string $firstname, string $lastname, string $patronymic)
    {
        $this->firstname = $firstname;
        $this->lastname = $lastname;
        $this->patronymic = $patronymic;
    }

    // getters ...
}

class City
{
    private $region;

    private $district;

    private $city;

    public function __construct(string $region, string $district, string $city)
    {
        $this->region = $region;
        $this->district = $district;
        $this->city = $city;
    }

    // getters ...
}

class Contact
{
    private $phone;

    private $email;

    public function __construct(string $phone, string $email)
    {
        $this->phone = $phone;
        $this->email = $email;
    }

    // getters ...
}

class Questionnaire
{
    public function __construct(Name $name, City $city, Contact $contact)
    {
        // ...
    }
}
~~~


### 函数名应体现他做了什么事

[](https://github.com/php-cpm/clean-code-php#%E5%87%BD%E6%95%B0%E5%90%8D%E5%BA%94%E4%BD%93%E7%8E%B0%E4%BB%96%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88%E4%BA%8B)

**坏:**

~~~html
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 啥？handle处理一个消息干嘛了？是往一个文件里写吗？
$message->handle();
~~~

**好:**

~~~html
class Email
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// 简单明了
$message->send();
~~~

### 函数里应当只有一层抽象abstraction

[](https://github.com/php-cpm/clean-code-php#%E5%87%BD%E6%95%B0%E9%87%8C%E5%BA%94%E5%BD%93%E5%8F%AA%E6%9C%89%E4%B8%80%E5%B1%82%E6%8A%BD%E8%B1%A1abstraction)

当你抽象层次过多时时，函数处理的事情太多了。需要拆分功能来提高可重用性和易用性，以便简化测试。 （译者注：这里从示例代码看应该是指嵌套过多）

**坏:**

~~~html
function parseBetterPHPAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }

    foreach ($ast as $node) {
        // parse...
    }
}
~~~

**坏:**

我们把一些方法从循环中提取出来，但是`parseBetterJSAlternative()`方法还是很复杂，而且不利于测试。

~~~html
function tokenize(string $code): array
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterPHPAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // 解析逻辑...
    }
}
~~~

**好:**

最好的解决方案是把`parseBetterPHPAlternative()`方法的依赖移除。

~~~html
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterPHPAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // 解析逻辑...
        }
    }
}
~~~

### 不要用flag作为函数的参数

[](https://github.com/php-cpm/clean-code-php#%E4%B8%8D%E8%A6%81%E7%94%A8flag%E4%BD%9C%E4%B8%BA%E5%87%BD%E6%95%B0%E7%9A%84%E5%8F%82%E6%95%B0)

flag就是在告诉大家，这个方法里处理很多事。前面刚说过，一个函数应当只做一件事。 把不同flag的代码拆分到多个函数里。

**坏:**

~~~html
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/' . $name);
    } else {
        touch($name);
    }
}
~~~

**好:**

~~~html
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/' . $name);
}
~~~

### 避免副作用

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E5%89%AF%E4%BD%9C%E7%94%A8)

一个函数做了比获取一个值然后返回另外一个值或值们会产生副作用如果。副作用可能是写入一个文件，修改某些全局变量或者偶然的把你全部的钱给了陌生人。

现在，你的确需要在一个程序或者场合里要有副作用，像之前的例子，你也许需要写一个文件。你想要做的是把你做这些的地方集中起来。不要用几个函数和类来写入一个特定的文件。用一个服务来做它，一个只有一个。

重点是避免常见陷阱比如对象间共享无结构的数据，使用可以写入任何的可变数据类型，不集中处理副作用发生的地方。如果你做了这些你就会比大多数程序员快乐。

**坏:**

~~~html
// Global variable referenced by following function.
// If we had another function that used this name, now it'd be an array and it could break it.
$name = 'Ryan McDermott';

function splitIntoFirstAndLastName(): void
{
    global $name;

    $name = explode(' ', $name);
}

splitIntoFirstAndLastName();

var_dump($name);
// ['Ryan', 'McDermott'];
~~~

**好:**

~~~html
function splitIntoFirstAndLastName(string $name): array
{
    return explode(' ', $name);
}

$name = 'Ryan McDermott';
$newName = splitIntoFirstAndLastName($name);

var_dump($name);
// 'Ryan McDermott';

var_dump($newName);
// ['Ryan', 'McDermott'];
~~~

### 不要写全局函数

[](https://github.com/php-cpm/clean-code-php#%E4%B8%8D%E8%A6%81%E5%86%99%E5%85%A8%E5%B1%80%E5%87%BD%E6%95%B0)

在大多数语言中污染全局变量是一个坏的实践，因为你可能和其他类库冲突 并且调用你api的人直到他们捕获异常才知道踩坑了。让我们思考一种场景： 如果你想配置一个数组，你可能会写一个全局函数`config()`，但是他可能 和试着做同样事的其他类库冲突。

**坏:**

~~~html
function config(): array
{
    return [
        'foo' => 'bar',
    ];
}
~~~

**好:**

~~~html
class Configuration
{
    private $configuration = [];

    public function __construct(array $configuration)
    {
        $this->configuration = $configuration;
    }

    public function get(string $key): ?string
    {
        // null coalescing operator
        return $this->configuration[$key] ?? null;
    }
}
~~~

加载配置并创建`Configuration`类的实例

~~~html
$configuration = new Configuration([
    'foo' => 'bar',
]);
~~~

现在你必须在程序中用`Configuration`的实例了

### 不要使用单例模式

[](https://github.com/php-cpm/clean-code-php#%E4%B8%8D%E8%A6%81%E4%BD%BF%E7%94%A8%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F)

单例是一种[反模式](https://en.wikipedia.org/wiki/Singleton_pattern). 以下是解释：Paraphrased from Brian Button:

1.  总是被用成全局实例。They are generally used as a**global instance**, why is that so bad? Because**you hide the dependencies**of your application in your code, instead of exposing them through the interfaces. Making something global to avoid passing it around is a[code smell](https://en.wikipedia.org/wiki/Code_smell).
2.  违反了[单一响应原则](https://github.com/php-cpm/clean-code-php/blob/master)They violate the[single responsibility principle](https://github.com/php-cpm/clean-code-php#single-responsibility-principle-srp): by virtue of the fact that**they control their own creation and lifecycle**.
3.  导致代码强耦合They inherently cause code to be tightly[coupled](https://en.wikipedia.org/wiki/Coupling_%28computer_programming%29). This makes faking them out under**test rather difficult**in many cases.
4.  在整个程序的生命周期中始终携带状态。They carry state around for the lifetime of the application. Another hit to testing since**you can end up with a situation where tests need to be ordered**which is a big no for unit tests. Why? Because each unit test should be independent from the other.

这里有一篇非常好的讨论单例模式的\[根本问题(([http://misko.hevery.com/2008/08/25/root-cause-of-singletons/)的文章，是](http://misko.hevery.com/2008/08/25/root-cause-of-singletons/)%E7%9A%84%E6%96%87%E7%AB%A0%EF%BC%8C%E6%98%AF)[Misko Hevery](http://misko.hevery.com/about/)写的。

**坏:**

~~~html
class DBConnection
{
    private static $instance;

    private function __construct(string $dsn)
    {
        // ...
    }

    public static function getInstance(): self
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }

        return self::$instance;
    }

    // ...
}

$singleton = DBConnection::getInstance();
~~~

**好:**

~~~html
class DBConnection
{
    public function __construct(string $dsn)
    {
        // ...
    }

    // ...
}
~~~

创建`DBConnection`类的实例并通过[DSN](http://php.net/manual/en/pdo.construct.php#refsect1-pdo.construct-parameters)配置.

~~~html
$connection = new DBConnection($dsn);
~~~

现在你必须在程序中 使用`DBConnection`的实例了

### 封装条件语句

[](https://github.com/php-cpm/clean-code-php#%E5%B0%81%E8%A3%85%E6%9D%A1%E4%BB%B6%E8%AF%AD%E5%8F%A5)

**坏:**

~~~html
if ($article->state === 'published') {
    // ...
}
~~~

**好:**

~~~html
if ($article->isPublished()) {
    // ...
}
~~~

### 避免用反义条件判断

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E7%94%A8%E5%8F%8D%E4%B9%89%E6%9D%A1%E4%BB%B6%E5%88%A4%E6%96%AD)

**坏:**

~~~html
function isDOMNodeNotPresent(DOMNode $node): bool
{
    // ...
}

if (! isDOMNodeNotPresent($node)) {
    // ...
}
~~~

**好:**

~~~html
function isDOMNodePresent(DOMNode $node): bool
{
    // ...
}

if (isDOMNodePresent($node)) {
    // ...
}
~~~

### 避免条件判断

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E6%9D%A1%E4%BB%B6%E5%88%A4%E6%96%AD)

这看起来像一个不可能任务。当人们第一次听到这句话是都会这么说。 "没有`if语句`我还能做啥？" 答案是你可以使用多态来实现多种场景 的相同任务。第二个问题很常见， “这么做可以，但为什么我要这么做？” 答案是前面我们学过的一个Clean Code原则：一个函数应当只做一件事。 当你有很多含有`if`语句的类和函数时,你的函数做了不止一件事。 记住，只做一件事。

**坏:**

~~~html
class Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
~~~

**好:**

~~~html
interface Airplane
{
    // ...

    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    // ...

    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
~~~

### 避免类型检查 (part 1)

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E7%B1%BB%E5%9E%8B%E6%A3%80%E6%9F%A5-part-1)

PHP是弱类型的,这意味着你的函数可以接收任何类型的参数。 有时候你为这自由所痛苦并且在你的函数渐渐尝试类型检查。 有很多方法去避免这么做。第一种是统一API。

**坏:**

~~~html
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
~~~

**好:**

~~~html
function travelToTexas(Vehicle $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
~~~

### 避免类型检查 (part 2)

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E7%B1%BB%E5%9E%8B%E6%A3%80%E6%9F%A5-part-2)

如果你正使用基本原始值比如字符串、整形和数组，要求版本是PHP 7+，不用多态，需要类型检测， 那你应当考虑[类型声明](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)或者严格模式。 提供了基于标准PHP语法的静态类型。 手动检查类型的问题是做好了需要好多的废话，好像为了安全就可以不顾损失可读性。 保持你的PHP 整洁，写好测试，做好代码回顾。做不到就用PHP严格类型声明和严格模式来确保安全。

**坏:**

~~~html
function combine($val1, $val2): int
{
    if (! is_numeric($val1) || ! is_numeric($val2)) {
        throw new Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
~~~

**好:**

~~~html
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
~~~

### 移除僵尸代码

[](https://github.com/php-cpm/clean-code-php#%E7%A7%BB%E9%99%A4%E5%83%B5%E5%B0%B8%E4%BB%A3%E7%A0%81)

僵尸代码和重复代码一样坏。没有理由保留在你的代码库中。如果从来没被调用过，就删掉！ 因为还在代码版本库里，因此很安全。

**坏:**

~~~html
function oldRequestModule(string $url): void
{
    // ...
}

function newRequestModule(string $url): void
{
    // ...
}

$request = newRequestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
~~~

**好:**

~~~html
function requestModule(string $url): void
{
    // ...
}

$request = requestModule($requestUrl);
inventoryTracker('apples', $request, 'www.inventory-awesome.io');
~~~

## 对象和数据结构

[](https://github.com/php-cpm/clean-code-php#%E5%AF%B9%E8%B1%A1%E5%92%8C%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)

### 使用 getters 和 setters

[](https://github.com/php-cpm/clean-code-php#%E4%BD%BF%E7%94%A8-getters-%E5%92%8C-setters)

在PHP中你可以对方法使用`public`,`protected`,`private`来控制对象属性的变更。

*   当你想对对象属性做获取之外的操作时，你不需要在代码中去寻找并修改每一个该属性访问方法
*   当有`set`对应的属性方法时，易于增加参数的验证
*   封装内部的表示
*   使用set*和get*时，易于增加日志和错误控制
*   继承当前类时，可以复写默认的方法功能
*   当对象属性是从远端服务器获取时，get\*，set\*易于使用延迟加载

此外，这样的方式也符合OOP开发中的[开闭原则](https://github.com/php-cpm/clean-code-php#%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99)

**坏:**

~~~html
class BankAccount
{
    public $balance = 1000;
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->balance -= 100;
~~~

**好:**

~~~html
class BankAccount
{
    private $balance;

    public function __construct(int $balance = 1000)
    {
      $this->balance = $balance;
    }

    public function withdraw(int $amount): void
    {
        if ($amount > $this->balance) {
            throw new \Exception('Amount greater than available balance.');
        }

        $this->balance -= $amount;
    }

    public function deposit(int $amount): void
    {
        $this->balance += $amount;
    }

    public function getBalance(): int
    {
        return $this->balance;
    }
}

$bankAccount = new BankAccount();

// Buy shoes...
$bankAccount->withdraw($shoesPrice);

// Get balance
$balance = $bankAccount->getBalance();
~~~

### 给对象使用私有或受保护的成员变量

[](https://github.com/php-cpm/clean-code-php#%E7%BB%99%E5%AF%B9%E8%B1%A1%E4%BD%BF%E7%94%A8%E7%A7%81%E6%9C%89%E6%88%96%E5%8F%97%E4%BF%9D%E6%8A%A4%E7%9A%84%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F)

*   对`public`方法和属性进行修改非常危险，因为外部代码容易依赖他，而你没办法控制。**对之修改影响所有这个类的使用者。**`public`methods and properties are most dangerous for changes, because some outside code may easily rely on them and you can't control what code relies on them.**Modifications in class are dangerous for all users of class.**
*   对`protected`的修改跟对`public`修改差不多危险，因为他们对子类可用，他俩的唯一区别就是可调用的位置不一样，**对之修改影响所有集成这个类的地方。**`protected`modifier are as dangerous as public, because they are available in scope of any child class. This effectively means that difference between public and protected is only in access mechanism, but encapsulation guarantee remains the same.**Modifications in class are dangerous for all descendant classes.**
*   对`private`的修改保证了这部分代码**只会影响当前类**`private`modifier guarantees that code is**dangerous to modify only in boundaries of single class**(you are safe for modifications and you won't have[Jenga effect](http://www.urbandictionary.com/define.php?term=Jengaphobia&defid=2494196)).

所以，当你需要控制类里的代码可以被访问时才用`public/protected`，其他时候都用`private`。

可以读一读这篇[博客文章](http://fabien.potencier.org/pragmatism-over-theory-protected-vs-private.html)，[Fabien Potencier](https://github.com/fabpot)写的.

**坏:**

~~~html
class Employee
{
    public $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$employee = new Employee('John Doe');
// Employee name: John Doe
echo 'Employee name: ' . $employee->name;
~~~

**好:**

~~~html
class Employee
{
    private $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }

    public function getName(): string
    {
        return $this->name;
    }
}

$employee = new Employee('John Doe');
// Employee name: John Doe
echo 'Employee name: ' . $employee->getName();
~~~

## 类

[](https://github.com/php-cpm/clean-code-php#%E7%B1%BB)

### 少用继承多用组合

[](https://github.com/php-cpm/clean-code-php#%E5%B0%91%E7%94%A8%E7%BB%A7%E6%89%BF%E5%A4%9A%E7%94%A8%E7%BB%84%E5%90%88)

正如 the Gang of Four 所著的[*设计模式*](https://en.wikipedia.org/wiki/Design_Patterns)之前所说， 我们应该尽量优先选择组合而不是继承的方式。使用继承和组合都有很多好处。 这个准则的主要意义在于当你本能的使用继承时，试着思考一下`组合`是否能更好对你的需求建模。 在一些情况下，是这样的。

接下来你或许会想，“那我应该在什么时候使用继承？” 答案依赖于你的问题，当然下面有一些何时继承比组合更好的说明：

1.  你的继承表达了“是一个”而不是“有一个”的关系（人类-》动物，用户-》用户详情）
2.  你可以复用基类的代码（人类可以像动物一样移动）
3.  你想通过修改基类对所有派生类做全局的修改（当动物移动时，修改她们的能量消耗）

**糟糕的:**

~~~html
class Employee
{
    private $name;

    private $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    // ...
}


// 不好，因为 Employees "有" taxdata
// 而 EmployeeTaxData 不是 Employee 类型的


class EmployeeTaxData extends Employee
{
    private $ssn;

    private $salary;

    public function __construct(string $name, string $email, string $ssn, string $salary)
    {
        parent::__construct($name, $email);

        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}
~~~

**好:**

~~~html
class EmployeeTaxData
{
    private $ssn;

    private $salary;

    public function __construct(string $ssn, string $salary)
    {
        $this->ssn = $ssn;
        $this->salary = $salary;
    }

    // ...
}

class Employee
{
    private $name;

    private $email;

    private $taxData;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function setTaxData(EmployeeTaxData $taxData): void
    {
        $this->taxData = $taxData;
    }

    // ...
}
~~~

### 避免连贯接口

[](https://github.com/php-cpm/clean-code-php#%E9%81%BF%E5%85%8D%E8%BF%9E%E8%B4%AF%E6%8E%A5%E5%8F%A3)

[连贯接口Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)是一种 旨在提高面向对象编程时代码可读性的API设计模式，他基于[方法链Method chaining](https://en.wikipedia.org/wiki/Method_chaining)

有上下文的地方可以降低代码复杂度，例如[PHPUnit Mock Builder](https://phpunit.de/manual/current/en/test-doubles.html)和[Doctrine Query Builder](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html)，更多的情况会带来较大代价：

While there can be some contexts, frequently builder objects, where this pattern reduces the verbosity of the code (for example the[PHPUnit Mock Builder](https://phpunit.de/manual/current/en/test-doubles.html)or the[Doctrine Query Builder](http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html)), more often it comes at some costs:

1.  破坏了[对象封装](https://en.wikipedia.org/wiki/Encapsulation_%28object-oriented_programming%29)
2.  破坏了[装饰器模式](https://en.wikipedia.org/wiki/Decorator_pattern)
3.  在测试组件中不好做[mock](https://en.wikipedia.org/wiki/Mock_object)
4.  导致提交的diff不好阅读

了解更多请阅读[连贯接口为什么不好](https://ocramius.github.io/blog/fluent-interfaces-are-evil/)，作者[Marco Pivetta](https://github.com/Ocramius).

**坏:**

~~~html
class Car
{
    private $make = 'Honda';

    private $model = 'Accord';

    private $color = 'white';

    public function setMake(string $make): self
    {
        $this->make = $make;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setModel(string $model): self
    {
        $this->model = $model;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function setColor(string $color): self
    {
        $this->color = $color;

        // NOTE: Returning this for chaining
        return $this;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = (new Car())
    ->setColor('pink')
    ->setMake('Ford')
    ->setModel('F-150')
    ->dump();
~~~

**好:**

~~~html
class Car
{
    private $make = 'Honda';

    private $model = 'Accord';

    private $color = 'white';

    public function setMake(string $make): void
    {
        $this->make = $make;
    }

    public function setModel(string $model): void
    {
        $this->model = $model;
    }

    public function setColor(string $color): void
    {
        $this->color = $color;
    }

    public function dump(): void
    {
        var_dump($this->make, $this->model, $this->color);
    }
}

$car = new Car();
$car->setColor('pink');
$car->setMake('Ford');
$car->setModel('F-150');
$car->dump();
~~~

### 推荐使用 final 类

[](https://github.com/php-cpm/clean-code-php#%E6%8E%A8%E8%8D%90%E4%BD%BF%E7%94%A8-final-%E7%B1%BB)

能用时尽量使用`final`关键字:

1.  阻止不受控的继承链
2.  鼓励[组合](https://github.com/php-cpm/clean-code-php#%E5%B0%91%E7%94%A8%E7%BB%A7%E6%89%BF%E5%A4%9A%E7%94%A8%E7%BB%84%E5%90%88).
3.  鼓励[单一职责模式](https://github.com/php-cpm/clean-code-php#%E5%8D%95%E4%B8%80%E8%81%8C%E8%B4%A3%E6%A8%A1%E5%BC%8F).
4.  鼓励开发者用你的公开方法而非通过继承类获取受保护方法的访问权限.
5.  使得在不破坏使用你的类的应用的情况下修改代码成为可能.

The only condition is that your class should implement an interface and no other public methods are defined.

For more informations you can read[the blog post](https://ocramius.github.io/blog/when-to-declare-classes-final/)on this topic written by[Marco Pivetta (Ocramius)](https://ocramius.github.io/).

**Bad:**

~~~html
final class Car
{
    private $color;

    public function __construct($color)
    {
        $this->color = $color;
    }

    /**
     * @return string The color of the vehicle
     */
    public function getColor()
    {
        return $this->color;
    }
}
~~~

**Good:**

~~~html
interface Vehicle
{
    /**
     * @return string The color of the vehicle
     */
    public function getColor();
}

final class Car implements Vehicle
{
    private $color;

    public function __construct($color)
    {
        $this->color = $color;
    }

    public function getColor()
    {
        return $this->color;
    }
}
~~~

## SOLID

[](https://github.com/php-cpm/clean-code-php#solid)

**SOLID**是Michael Feathers推荐的便于记忆的首字母简写，它代表了Robert Martin命名的最重要的五个面对对象编码设计原则

*   [S: 单一职责原则 (SRP)](https://github.com/php-cpm/clean-code-php#%E8%81%8C%E8%B4%A3%E5%8E%9F%E5%88%99)
*   [O: 开闭原则 (OCP)](https://github.com/php-cpm/clean-code-php#%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99)
*   [L: 里氏替换原则 (LSP)](https://github.com/php-cpm/clean-code-php#%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99)
*   [I: 接口隔离原则 (ISP)](https://github.com/php-cpm/clean-code-php#%E6%8E%A5%E5%8F%A3%E9%9A%94%E7%A6%BB%E5%8E%9F%E5%88%99)
*   [D: 依赖倒置原则 (DIP)](https://github.com/php-cpm/clean-code-php#%E4%BE%9D%E8%B5%96%E5%80%92%E7%BD%AE%E5%8E%9F%E5%88%99)

### 单一职责原则

[](https://github.com/php-cpm/clean-code-php#%E5%8D%95%E4%B8%80%E8%81%8C%E8%B4%A3%E5%8E%9F%E5%88%99)

Single Responsibility Principle (SRP)

正如在Clean Code所述，"修改一个类应该只为一个理由"。 人们总是易于用一堆方法塞满一个类，如同我们只能在飞机上 只能携带一个行李箱（把所有的东西都塞到箱子里）。这样做 的问题是：从概念上这样的类不是高内聚的，并且留下了很多 理由去修改它。将你需要修改类的次数降低到最小很重要。 这是因为，当有很多方法在类中时，修改其中一处，你很难知 晓在代码库中哪些依赖的模块会被影响到。

**坏:**

~~~html
class UserSettings
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function changeSettings(array $settings): void
    {
        if ($this->verifyCredentials()) {
            // ...
        }
    }

    private function verifyCredentials(): bool
    {
        // ...
    }
}
~~~

**好:**

~~~html
class UserAuth
{
    private $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function verifyCredentials(): bool
    {
        // ...
    }
}

class UserSettings
{
    private $user;

    private $auth;

    public function __construct(User $user)
    {
        $this->user = $user;
        $this->auth = new UserAuth($user);
    }

    public function changeSettings(array $settings): void
    {
        if ($this->auth->verifyCredentials()) {
            // ...
        }
    }
}
~~~


### 开闭原则

[](https://github.com/php-cpm/clean-code-php#%E5%BC%80%E9%97%AD%E5%8E%9F%E5%88%99)

Open/Closed Principle (OCP)

正如Bertrand Meyer所述，"软件的工件（ classes, modules, functions 等） 应该对扩展开放，对修改关闭。" 然而这句话意味着什么呢？这个原则大体上表示你 应该允许在不改变已有代码的情况下增加新的功能

**坏:**

~~~html
abstract class Adapter
{
    protected $name;

    public function getName(): string
    {
        return $this->name;
    }
}

class AjaxAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'ajaxAdapter';
    }
}

class NodeAdapter extends Adapter
{
    public function __construct()
    {
        parent::__construct();

        $this->name = 'nodeAdapter';
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        $adapterName = $this->adapter->getName();

        if ($adapterName === 'ajaxAdapter') {
            return $this->makeAjaxCall($url);
        } elseif ($adapterName === 'httpNodeAdapter') {
            return $this->makeHttpCall($url);
        }
    }

    private function makeAjaxCall(string $url): Promise
    {
        // request and return promise
    }

    private function makeHttpCall(string $url): Promise
    {
        // request and return promise
    }
}
~~~

**好:**

~~~html
interface Adapter
{
    public function request(string $url): Promise;
}

class AjaxAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class NodeAdapter implements Adapter
{
    public function request(string $url): Promise
    {
        // request and return promise
    }
}

class HttpRequester
{
    private $adapter;

    public function __construct(Adapter $adapter)
    {
        $this->adapter = $adapter;
    }

    public function fetch(string $url): Promise
    {
        return $this->adapter->request($url);
    }
}
~~~

### 里氏替换原则

[](https://github.com/php-cpm/clean-code-php#%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99)

Liskov Substitution Principle (LSP)

这是一个简单的原则，却用了一个不好理解的术语。它的正式定义是 "如果S是T的子类型，那么在不改变程序原有既定属性（检查、执行 任务等）的前提下，任何T类型的对象都可以使用S类型的对象替代 （例如，使用S的对象可以替代T的对象）" 这个定义更难理解:-)。

对这个概念最好的解释是：如果你有一个父类和一个子类，在不改变 原有结果正确性的前提下父类和子类可以互换。这个听起来依旧让人 有些迷惑，所以让我们来看一个经典的正方形-长方形的例子。从数学 上讲，正方形是一种长方形，但是当你的模型通过继承使用了"is-a" 的关系时，就不对了。

**坏:**

~~~html
class Rectangle
{
    protected $width = 0;

    protected $height = 0;

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}

function printArea(Rectangle $rectangle): void
{
    $rectangle->setWidth(4);
    $rectangle->setHeight(5);

    // BAD: Will return 25 for Square. Should be 20.
    echo sprintf('%s has area %d.', get_class($rectangle), $rectangle->getArea()) . PHP_EOL;
}

$rectangles = [new Rectangle(), new Square()];

foreach ($rectangles as $rectangle) {
    printArea($rectangle);
}
~~~

**好:**

最好是将这两种四边形分别对待，用一个适合两种类型的更通用子类型来代替。

尽管正方形和长方形看起来很相似，但他们是不同的。 正方形更接近菱形，而长方形更接近平行四边形。但他们不是子类型。 尽管相似，正方形、长方形、菱形、平行四边形都是有自己属性的不同形状。

~~~html
interface Shape
{
    public function getArea(): int;
}

class Rectangle implements Shape
{
    private $width = 0;
    private $height = 0;

    public function __construct(int $width, int $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square implements Shape
{
    private $length = 0;

    public function __construct(int $length)
    {
        $this->length = $length;
    }

    public function getArea(): int
    {
        return $this->length ** 2;
    }
}

function printArea(Shape $shape): void
{
    echo sprintf('%s has area %d.', get_class($shape), $shape->getArea()).PHP_EOL;
}

$shapes = [new Rectangle(4, 5), new Square(5)];

foreach ($shapes as $shape) {
    printArea($shape);
}
~~~

### 接口隔离原则

[](https://github.com/php-cpm/clean-code-php#%E6%8E%A5%E5%8F%A3%E9%9A%94%E7%A6%BB%E5%8E%9F%E5%88%99)

Interface Segregation Principle (ISP)

接口隔离原则表示："调用方不应该被强制依赖于他不需要的接口"

有一个清晰的例子来说明示范这条原则。当一个类需要一个大量的设置项， 为了方便不会要求调用方去设置大量的选项，因为在通常他们不需要所有的 设置项。使设置项可选有助于我们避免产生"胖接口"

**坏:**

~~~html
interface Employee
{
    public function work(): void;

    public function eat(): void;
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        // ...... eating in lunch break
    }
}

class RobotEmployee implements Employee
{
    public function work(): void
    {
        //.... working much more
    }

    public function eat(): void
    {
        //.... robot can't eat, but it must implement this method
    }
}
~~~

**好:**

不是每一个工人都是雇员，但是每一个雇员都是一个工人

~~~html
interface Workable
{
    public function work(): void;
}

interface Feedable
{
    public function eat(): void;
}

interface Employee extends Feedable, Workable
{
}

class HumanEmployee implements Employee
{
    public function work(): void
    {
        // ....working
    }

    public function eat(): void
    {
        //.... eating in lunch break
    }
}

// robot can only work
class RobotEmployee implements Workable
{
    public function work(): void
    {
        // ....working
    }
}
~~~


### 依赖倒置原则

[](https://github.com/php-cpm/clean-code-php#%E4%BE%9D%E8%B5%96%E5%80%92%E7%BD%AE%E5%8E%9F%E5%88%99)

Dependency Inversion Principle (DIP)

这条原则说明两个基本的要点：

1.  高阶的模块不应该依赖低阶的模块，它们都应该依赖于抽象
2.  抽象不应该依赖于实现，实现应该依赖于抽象

这条起初看起来有点晦涩难懂，但是如果你使用过 PHP 框架（例如 Symfony），你应该见过 依赖注入（DI），它是对这个概念的实现。虽然它们不是完全相等的概念，依赖倒置原则使高阶模块 与低阶模块的实现细节和创建分离。可以使用依赖注入（DI）这种方式来实现它。最大的好处 是它使模块之间解耦。耦合会导致你难于重构，它是一种非常糟糕的的开发模式。

**坏:**

~~~html
class Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot extends Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
~~~

**好:**

~~~html
interface Employee
{
    public function work(): void;
}

class Human implements Employee
{
    public function work(): void
    {
        // ....working
    }
}

class Robot implements Employee
{
    public function work(): void
    {
        //.... working much more
    }
}

class Manager
{
    private $employee;

    public function __construct(Employee $employee)
    {
        $this->employee = $employee;
    }

    public function manage(): void
    {
        $this->employee->work();
    }
}
~~~

## 别写重复代码 (DRY)

[](https://github.com/php-cpm/clean-code-php#%E5%88%AB%E5%86%99%E9%87%8D%E5%A4%8D%E4%BB%A3%E7%A0%81-dry)

试着去遵循[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)原则.

尽你最大的努力去避免复制代码，它是一种非常糟糕的行为，复制代码 通常意味着当你需要变更一些逻辑时，你需要修改不止一处。

试想一下，如果你在经营一家餐厅并且你在记录你仓库的进销记录：所有 的土豆，洋葱，大蒜，辣椒等。如果你有多个列表来管理进销记录，当你 用其中一些土豆做菜时你需要更新所有的列表。如果你只有一个列表的话 只有一个地方需要更新。

通常情况下你复制代码是应该有两个或者多个略微不同的逻辑，它们大多数 都是一样的，但是由于它们的区别致使你必须有两个或者多个隔离的但大部 分相同的方法，移除重复的代码意味着用一个function/module/class创 建一个能处理差异的抽象。

用对抽象非常关键，这正是为什么你必须学习遵守在[类](https://github.com/php-cpm/clean-code-php#%E7%B1%BB)章节写 的SOLID原则，不合理的抽象比复制代码更糟糕，所以务必谨慎！说了这么多， 如果你能设计一个合理的抽象，那就这么干！别写重复代码，否则你会发现 任何时候当你想修改一个逻辑时你必须修改多个地方。

**坏:**

~~~html
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [$expectedSalary, $experience, $githubLink];

        render($data);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [$expectedSalary, $experience, $githubLink];

        render($data);
    }
}
~~~

**好:**

~~~html
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [$expectedSalary, $experience, $githubLink];

        render($data);
    }
}
~~~

**极好:**

最好让代码紧凑一点

~~~html
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        render([$employee->calculateExpectedSalary(), $employee->getExperience(), $employee->getGithubLink()]);
    }
}
~~~