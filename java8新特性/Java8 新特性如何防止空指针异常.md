# Java8 新特性如何防止空指针异常

要说 Java 编程中哪个异常是你印象最深刻的，那 `NullPointerException` 空指针可以说是臭名昭著的。不要说初级程序员会碰到， 即使是中级，专家级程序员稍不留神，就会掉入这个坑里。

`Null` 引用的发明者 [Tony Hoare](http://en.wikipedia.org/wiki/Tony_Hoare) 曾在 2009 年作出道歉声明，声明中表示，到目前为止，空指针异常大约给企业已造成数十亿美元的损失。

下面是 Tony Hoare 的原话：

> 我将 Null 引用的设计称为是一个数十亿美元的错误。1965 那年，我正在用面向对象语言(ALGOL W) 设计首个功能全面的系统。当时我的考量是，确保所有被使用的引用都是安全的，编译器会自动进行检查。但是，我没有抵住诱惑，加入了 Null 引用，仅仅是为了实现起来省事。这之后，它导致了数不清的 bug、错误和系统崩溃，也为企业导致了不可估量的损失。

事已至此，我们必须学会面对它。So, 我们要如何防止空指针异常呢？

唯一的办法就是对可能为 Null 的对象添加检查。但是 Null 检查是繁琐且痛苦的。所以一些比较新的语言为了处理 Null 检查，特意添加了特殊的语法，如[空合并运算符](http://en.wikipedia.org/wiki/Null_coalescing_operator)。

> 在 [Groovy](http://groovy-lang.org/operators.html#_elvis_operator) 或 [Kotlin](http://kotlinlang.org/docs/reference/null-safety.html) 这样的语言中也被称为 `Elvis` 运算符。

不幸的是，在老版本的 Java 中并没有提供这样的语法糖。Java8 中在这方面做了改进。所以，这篇文章就特意来介绍一下如何在 Java8 中利用新特性来编写防止 `NullPointerException`的发生。

## Java8 中如何加强对 Null 对象的检查？

在上篇文章 [Java8 新特性指导手册](https://www.exception.site/course/3/chapter/1) 中简单的提了一下如何通过 `Optional` 类来对对象做空校验。接下来，我们再细说一下：

![img](https://exception-image-bucket.oss-cn-hangzhou.aliyuncs.com/154838376789518)

在业务系统中，对象中嵌套对象是经常发生的场景，如下示例代码：

```java
// 最外层对象
class Outer {
    Nested nested;
    Nested getNested() {
        return nested;
    }
}
// 第二层对象
class Nested {
    Inner inner;
    Inner getInner() {
        return inner;
    }
}
// 最底层对象
class Inner {
    String foo;
    String getFoo() {
        return foo;
    }
}
```

业务中，假设我们需要获取 `Outer` 对象对底层的 `Inner` 中的 `foo` 属性，我们必须写一堆的非空校验，来防止发生 `NullPointerException`：

```java
// 繁琐的代码
Outer outer = new Outer();
if (outer != null && outer.nested != null && outer.nested.inner != null) {
    System.out.println(outer.nested.inner.foo);
}
```

## 通过 Optional

在 Java8 中，我们有更优雅的解决方式，那就是使用 `Optional`是说，我们可以在一行代码中，进行流水式的 `map` 操作。而 **map 方法内部会自动进行空校验**：

```java
Optional.of(new Outer())
    .map(Outer::getNested)
    .map(Nested::getInner)
    .map(Inner::getFoo
    .ifPresent(System.out::println); // 如果不为空，最终输出 foo 的值
```

## 通过 suppiler 函数自定义增强 API

上面这种方式个人感觉还是有点啰嗦，我们可以利用 `suppiler` 函数来出一个终极解决方案：

```java
public static <T> Optional<T> resolve(Supplier<T> resolver) {
    try {
        T result = resolver.get();
        return Optional.ofNullable(result);
    }
    catch (NullPointerException e) {
        // 可能会抛出空指针异常，直接返回一个空的 Optional 对象
        return Optional.empty();
    }
}
```

利用上面的 `resolve` 方法来重构上述的非空校验代码段：

```java
Outer obj = new Outer();
// 直接调用 resolve 方法，内部做空指针的处理
resolve(() -> obj.getNested().getInner().getFoo());
    .ifPresent(System.out::println); // 如果不为空，最终输出 foo 的值
```

## Optional 方法的使用

JDK1.8 版本的 Optional 总共有17个方法, 除去两个私有的构造方法, 除去equals, hashCode, toString三个方法外, 我们来分析一下剩下12个 Optional 提供的方法.
**先来看三个构造 Optional 实例的方法**

- empty 方法返回一个不包含值的 Optional 实例, 注意不保证返回的 empty 是单例, 不要用 == 比较.

```java
public static<T> Optional<T> empty();
```

- 返回一个 Optional 实例, 代表指定的非空值, 如果传入 null 会立刻抛出空指针异常

```java
public static <T> Optional<T> of(T value);
```

- 返回一个 Optional 实例, 如果指定非空值则实例包含非空值, 如果传入 null 返回不包含值的 empty

```java
public static <T> Optional<T> ofNullable(T value);
```

**下面来看 Optional 中两个让人唾弃的方法, 不推荐使用**

- isPresent 用来判断实例是否包含值, 如果不包含非空值返回 false, 否则返回 true

```java
public boolean isPresent();
```

- get 方法, 如果实例包含值则返回当前值, 否则抛出 NoSushElementException 异常.

```java
public T get();
```

为什么不推荐调用上面两个方法呢, 因为容易写出下方代码, 比原先判断 if null 的代码还脏.

```java
Integer result;
if (optional.isPresent()) {
    result = optional.get();
}
```

**接下里我们来看 Optional 中真正有用的7个方法**

- ifPresent 方法作用是当实例包含值时, 来执行传入的 Consumer, 比如调用一些其他方法.

```java
public void ifPresent(Consumer<? super T> consumer);
```

- filter 方法用于过滤不符合条件的值, 接收一个 Predicate 参数, 如果符合条件返回代表值的 Optional 实例, 否则返回 empty

```java
public Optional<T> filter(Predicate<? super T> predicate)
```

- map 方法是链式调用避免空指针的核心方法, 当实例包含值时, 对值执行传入的 Function 逻辑, 并返回一个代表结果值的新的 Optional 实例. 这也意味着返回的结果依旧可以继续调用 map 方法, 而不需要空指针判断.方法签名以及示例如下:

```java
public<U> Optional<U> map(Function<? super T, ? extends U> mapper);
// 如果 outer 包含的 nested 对象或者 nested 包含的 inner 对象为空
// 传统方式 每一个 get 方法都要进行空判断, 否则有可能抛出空指针异常
// 用 Optional map 实现链式调用, 而不需要判断 null
// 任何一层对象为 null 时, 变量获得的都是指定默认值 null
String name = Optional.of(outter).map(Outter::getNested)
    .map(Nested::getInner)
    .map(Inner::getName).orElse(null);
```

- flatMap 方法与 map 方法作用一致, 不过 flatMap 接收的参数 Function 要求返回一个 Optional 实例, 并且 flatMap 方法直接返回该结果, 而不对结果包装一层 Optional, 适用于 Optional 包含的值也是 Optional, 可以进行多层 Optional 的合并.

```java
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper);
```

**接下来看三个也比较重要的方法, orElse, orElseGet, orElseThrow**

- 如果实例包含值, 那么返回这个值, 否则返回指定的默认值, 如null

```java
public T orElse(T other);
```

- 如果实例包含值, 返回这个值, 否则调用传入的 Supplier 参数生产一个值.

```java
public T orElseGet(Supplier<? extends T> other);
```

- 如果实例不包含值, 调用传入的 Supplier 参数, 生成一个异常实例并抛出.这个方法通常与全局异常处理器一起使用, 当参数或者其他情况获取不到值时, 抛出自定义异常, 由异常处理器处理成通用返回结果, 返回给前端.

```java
public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier);
```

## 实际开发中 Optional 的应用

在使用 Repository 操作 Es 时, findById 通过 id 查询数据库记录, 返回结果为 Optional, 此时无需 if null 判断, 利用 orElseThrow , 不存在就抛出异常, 配合异常处理器渲染通用结果和错误提示信息返回给前端处理.

```java
Optional<ApiManager> byId = apiManagerRepository
		.findById(apiManagerDTO.getId());
ApiManager oldApiManager = byId
		.orElseThrow(
		() -> new SysException(SysCodeEnum.API_MANAGER_DOES_NOT_EXIST)
		);
```

## 最后

你需要知道的是，上面这两个解决方案并没传统的 `null` 检查性能那么高效。但在绝大部分业务场景下，舍弃那么一丢丢的性能来方便编码，是完全可取， 除非是那种对性能有严格要求的场景，我们才不建议使用。

> 个人觉得，真要拿这点性能说事，还不如去优化优化 sql 语句，业务逻辑等。

