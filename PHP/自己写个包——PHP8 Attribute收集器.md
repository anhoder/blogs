# 自己写个包——PHP8 Attribute收集器

PHP8发布已经有一段时间了，新增了不少特性，其中我觉得比较好用的：一个是函数可以指定参数名传参，另外一个就是**注解Attribute**。

## 注解

在PHP8之前，PHP语法上是不支持注解的，只能通过反射来获取类、方法或属性的注释，然后通过解析注释来获取需要的元数据。最具代表性的就是注解包[doctrine/annotations](https://github.com/doctrine/annotations)，很多框架都有在用。

使用方法：

```php
use Doctrine\Common\Annotations\Annotation\Attribute;
use Doctrine\Common\Annotations\Annotation\Attributes;
use Doctrine\Common\Annotations\Annotation\Target;

/**
 * @Annotation
 * @Target("CLASS")
 * @Attributes({
 *     @Attribute("name", type="string", required=true),
 *     @Attribute("age", type="int")
 * })
 */
class Student
{
    /**
     * @var string
     */
    private $name;
    
    /**
     * @var int
     */
    private $age
}

/**
 * @Student(name="anhoder", age=18)
 */
class Anhoder
{
    
}
```

而在PHP8中，注解Attribute是这样的：

```php
#[Attribute(Attribute::TARGET_CLASS)]
class Student
{
    public function __construct(private string $name, private int $age)
    {
    }
}

#[Student('anhoder', 18)]
class Anhoder
{

}
```

## 收集注解Attribute

PHP8的Attribute数据也需要通过反射来手动获取：

```php
$reflection = new ReflectionClass(Anhoder::class);
var_dump($reflection->getAttributes());
// array(1) {
//   [0]=>
//   object(ReflectionAttribute)#3 (0) {
//   }
// }
```

我们可以实现一个Attribute收集器，用于自动收集项目中或其他composer包中的注解信息。

### 1. 获取所有包的根目录及namespace

通过composer提供的ClassLoader自动加载类，我们获取到所有包的命名空间及对应的根目录：

```php
// 获取Composer的自动加载类
function getComposerLoader()
{
    $loaders = spl_autoload_functions();

    foreach ($loaders as $loader) {
        if (is_array($loader) && isset($loader[0]) && $loader[0] instanceof ClassLoader) {
            return $loader[0];
        }
    }
    
    throw new NotFoundException('Composer class loader');
}

$composerLoader = getComposerLoader();

// 获取命令空间及目录
$composerLoader->getPrefixesPsr4();
```

### 2. 加载每个包下的AnnotationConfig类

我们约定：在每个需要收集注解的包下，都放一个AnnotationConfig类，并实现AnnotationConfigInterface接口。用于获取该包下需要扫描的目录及命名空间。例如：

```php
class AnnotationConfig implements AnnotationConfigInterface
{

    /**
     * @inheritDoc
     */
    public static function getAnnotationConfigs(): array
    {

        return [
            'scanDirs' => [
                __NAMESPACE__ => __DIR__,
            ],
        ];
    }
}
```

在以上约定条件下，我们需要遍历第一步获取到的根目录，判断其目录下是否有AnnotationConfig类，若有则调用getAnnotationConfigs方法获取其配置。

### 3. 解析、收集注解

最后一步，就是遍历每个目录下的文件及其对应的类，然后通过反射获取到类、方法、属性及常量的注解信息，并将这些信息存入容器数组中。

```php
$namespace = rtrim($namespace, '\\');
$iterator = new RecursiveDirectoryIterator($dir);

foreach ($iterator as $splFileInfo) {
    $basename = $splFileInfo->getBasename();
    
    if (!$splFileInfo->isFile() || $splFileInfo->getExtension() != 'php') {
        // not php file
        continue;
    }

    // PHP File
    $className = $splFileInfo->getBasename('.' . $splFileInfo->getExtension());
    $class = "{$namespace}\\{$className}";
    if (class_exists($class)) {
        $reflection = new ReflectionClass($class);
        $attributes = $reflection->getAttributes();
    }
}
```

## 封装为composer包

以上代码都是简单的示例，实际还需要考虑更多，所以我将其做成了一个composer包——[anhoder/annotations-collector](https://github.com/anhoder/annotations-collector)。简单的使用：

1. 安装
    ```sh
    composer require anhoder/annotations-collector
    ```
2. 创建 `AnnotationConfig.php` 文件到你项目的根目录

    ```php
    class AnnotationConfig implements AnnotationConfigInterface
    {
    
        public static function getAnnotationConfigs(): array
        {
            return [
                // The dirs need to be scanned
                'scanDirs' => [
                    __NAMESPACE__ => __DIR__,
                ],
            ];
        }
    }
    ```
3. 添加Annotation及AnnotationHandler.

    ```php
    // Annotation
    #[Attribute(Attribute::TARGET_CLASS)]
    class ClassAnnotation
    {
        public const TEST = 'test';
    
        private string $test;
    
        public function __construct(string $test)
        {
            $this->test = $test;
        }
    }
    
    // AnnotationHandler
    #[AnnotationHandler(ClassAnnotation::class)]
    class ClassAnnotationHandler extends AbstractAnnotationHandler
    {
        public function handle()
        {
            // Your logic.
            var_dump($this);
        }
    }
    ```
4. 开始扫描

    ```php
    AnnotationHelper::scan();
    ```

## 附

由于注解需要配合反射来使用，其相比正常代码性能较差，而且在php-fpm模式下，每次请求都需要重新获取注解信息，所以**不推荐在php-fpm下使用注解**，其**更适合一些常驻内存型的项目**，例如：Swoole、WorkerMan等。