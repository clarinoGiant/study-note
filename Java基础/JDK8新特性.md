### Optional

 Optional 类既可以含有对象也可以为空 

在线JDK说明： https://www.matools.com/file/manual/jdk_api_1.8_google/java/util/Optional.html#orElseThrow-java.util.function.Supplier-

#### 1. 构造函数

```java
private Optional() // 构建一个空的 Optional 实例。实例中的 value == null
private Optional(T var1) // 构建一个Optional 实例。实例中的 value == var1。var1为NULL会抛NPE.
```

#### 2. 获取 Optional 实例方法

```java
public static  Optional empty() // 获取一个 Optional 空实例。
public static  Optional of(T var0) // 获取一个 Optional 实例。var0为NULL会抛NPE。
public static  Optional ofNullable(T var0) // 获取一个 Optional 实例。var0为 NULL 会返回一个 Optional 空实例。
```

#### 3.获取 Optional 实例中的 value

```java
public T get()  // value 为空抛NoSuchElementException。
public T orElse(T var1) // value 不为 NULL 则返回 value ；为 NULL 返回 var1。
public T orElseGet(Supplier<? extends T> var1) // value 不为 NULL 则返回 value ；为 NULL 执行 Supplier。
public <X extends Throwable> T orElseThrow(Supplier<? extends X> var1) throws X 
// value 不为 NULL 则返回 value ；为 NULL 执行 var1。
```

#### 4.判断 Optional 是否存在

```java
public boolean isPresent()  // 即返回 Optional 实例中的 value 是否为 NULL。
public void ifPresent(Consumer<? super T> var1) // value 不为 NULL 则执行 var1。
```

#### 5.其他 Lambda 表达式的操作

 如果当前对象为NULL，则返回自己或者一个 Optional 空实例。不 NULL 则执行后面的 Lambda，得到返回结果的Optional 实例 

```java
public Optional<T> filter(Predicate<? super T> var1)：
public <U> Optional<U> map(Function<? super T, ? extends U> var1)：
public <U> Optional<U> flatMap(Function<? super T, Optional<U>> var1)
```

#### 6.代码样例

```java
Optional<User> emptyOpt = Optional.empty();
Optional<User> opt = Optional.of(user);  // 不可为NULL，否则NPE
Optional<User> opt = Optional.ofNullable(user);

Optional<String> opt = Optional.ofNullable(name);
assertEquals("John", opt.get());  // value 为空抛NoSuchElementException

User user = null;
User user2 = new User("anna@gmail.com", "1234");
User result = Optional.ofNullable(user).orElse(user2);
User result = Optional.ofNullable(user).orElseGet( () -> user2); // user为空执行Supplier

User result = Optional.ofNullable(user)
      .orElseThrow( () -> new IllegalArgumentException());

User user = new User("anna@gmail.com", "1234");
String email = Optional.ofNullable(user)
      .map(u -> u.getEmail()).orElse("default@gmail.com");
// map() 对值应用(调用)作为参数的函数，

User user = new User("anna@gmail.com", "1234");
user.setPosition("Developer");
String position = Optional.ofNullable(user)
      .flatMap(u -> u.getPosition()).orElse("default");

// filter() 接受一个 Predicate 参数，返回测试结果为 true 的值。如果测试结果为 false，会返回一个空的 Optional。
User user = new User("anna@gmail.com", "1234");
    Optional<User> result = Optional.ofNullable(user)
      .filter(u -> u.getEmail() != null && u.getEmail().contains("@"));
```

具体场景

```java
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}
```

==优化后==

1. 重构类，使其 getter 方法返回 *Optional* 引用：

```java
public class User {
    private Address address;

    public Optional<Address> getAddress() {
        return Optional.ofNullable(address);
    }

    // ...
}
public class Address {
    private Country country;

    public Optional<Country> getCountry() {
        return Optional.ofNullable(country);
    }

    // ...
}
```

2. 具体业务代码

```java
String result = Optional.ofNullable(user)
  .flatMap(User::getAddress)
  .flatMap(Address::getCountry)
  .map(Country::getIsocode)
  .orElse("default");
```

