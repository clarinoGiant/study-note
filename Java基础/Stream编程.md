# Stream编程

参考：java/util/stream/package-summary.html

# 1. Stream基础知识

## 1.1 Stream与collection区别



- No storage. A stream is not a data structure that stores elements; instead, it conveys elements from a source such as a data structure, an array, a generator function, or an I/O channel, through a pipeline of computational operations.

- Functional in nature. An operation on a stream produces a result, but does not modify its source. For example, filtering a `Stream` obtained from a collection produces a new `Stream` without the filtered elements, rather than removing elements from the source collection.
- Laziness-seeking. Many stream operations, such as filtering, mapping, or duplicate removal, can be implemented lazily, exposing opportunities for optimization. For example, "find the first `String` with three consecutive vowels" need not examine all the input strings. Stream operations are divided into intermediate (`Stream`-producing) operations and terminal (value- or side-effect-producing) operations. Intermediate operations are always lazy.

- Possibly unbounded. While collections have a finite size, streams need not. Short-circuiting operations such as `limit(n)` or `findFirst()` can allow computations on infinite streams to complete in finite time.
- Consumable. The elements of a stream are only visited once during the life of a stream. Like an [`Iterator`](../Iterator.html), a new stream must be generated to revisit the same elements of the source.

## 1.2 流的操作与pipeline

Stream operations are divided into ==intermediate== and ==terminal== operations, and are combined to form *stream pipelines*.

A stream pipeline consists of

- a source (such as a `Collection`, an array, a generator function, or an I/O channel); 
- followed by zero or more intermediate operations such as `Stream.filter` or `Stream.map`; 
- a terminal operation such as `Stream.forEach` or `Stream.reduce`.

Intermediate operations return a new stream. They are always *lazy*; executing an intermediate operation such as `filter()` does not actually perform any filtering, but instead creates a new stream that, when traversed, contains the elements of the initial stream that match the given predicate. Traversal of the pipeline source does not begin until the terminal operation of the pipeline is executed.

Terminal operations, such as `Stream.forEach` or `IntStream.sum`, may traverse the stream to produce a result or a side-effect. After the terminal operation is performed, the stream pipeline is considered consumed, and can no longer be used; if you need to traverse the same data source again, you must return to the data source to get a new stream. In almost all cases, terminal operations are *eager*, completing their traversal of the data source and processing of the pipeline before returning. Only the terminal operations `iterator()` and `spliterator()` are not; these are provided as an "escape hatch" to enable arbitrary client-controlled pipeline traversals in the event that the existing operations are not sufficient to the task.

## 1.3 Parallelism

All streams operations can execute either in serial or in parallel.

`Collection` has methods [`Collection.stream()`](../Collection.html#stream()) and [`Collection.parallelStream()`](../Collection.html#parallelStream()),



## 1.4 怎么得到Stream

- From a [`Collection`](../Collection.html) via the `stream()` and `parallelStream()` methods;
- From an array via [`Arrays.stream(Object[\])`](../Arrays.html#stream(T[]));
- From static factory methods on the stream classes, such as [`Stream.of(Object[\])`](Stream.html#of(T...)), [`IntStream.range(int, int)`](IntStream.html#range(int,int)) or [`Stream.iterate(Object, UnaryOperator)`](Stream.html#iterate(T,java.util.function.UnaryOperator));
- The lines of a file can be obtained from [`BufferedReader.lines()`](../../io/BufferedReader.html#lines());
- Streams of file paths can be obtained from methods in [`Files`](../../nio/file/Files.html);
- Streams of random numbers can be obtained from [`Random.ints()`](../Random.html#ints());
- Numerous other stream-bearing methods in the JDK, including [`BitSet.stream()`](../BitSet.html#stream()), [`Pattern.splitAsStream(java.lang.CharSequence)`](../regex/Pattern.html#splitAsStream(java.lang.CharSequence)), and [`JarFile.stream()`](../jar/JarFile.html#stream()).



# 2. API定义

## 2.1 中间处理



## 2.2 规约型





# 3. 样例

```java
int sum = widgets.parallelStream()
                      .filter(b -> b.getColor() == RED)
                      .mapToInt(b -> b.getWeight())
                      .sum();
// reduce操作
int sum = numbers.stream().reduce(0, Integer::sum);
int sum = numbers.parallelStream().reduce(0, Integer::sum);

// 
OptionalInt heaviest = widgets.parallelStream()
                               .mapToInt(Widget::getWeight)
                               .max();
```

*In its more general form, a reduce operation on elements of type <T> yielding a result of type <U> requires three parameters:*

```java
<U> U reduce(U identity,
              BiFunction<U, ? super T, U> accumulator,
              BinaryOperator<U> combiner);
// identity: both an initial seed value for the reduction and a default result if there are no input elements.
// The accumulator function takes a partial result and the next element, and produces a new partial result.
// The combiner function combines two partial results to produce a new partial result.

int sumOfWeights = widgets.stream()
                           .reduce(0,
                                   (sum, b) -> sum + b.getWeight(),
                                   Integer::sum);

Mutable reduction
<R> R collect(Supplier<R> supplier,
           BiConsumer<R, ? super T> accumulator,
           BiConsumer<R, R> combiner);
// a supplier function to construct new instances of the result container, 
// an accumulator function to incorporate an input element into a result container,
// a combining function to merge the contents of one result container into another. 

List<String> strings = stream.map(Object::toString)
                                  .collect(Collectors.toList());
Collector<Employee, ?, Integer> summingSalaries
         = Collectors.summingInt(Employee::getSalary);

Map<Department, Integer> salariesByDept
 = employees.stream().collect(Collectors.groupingBy(Employee::getDepartment,
                                                    summingSalaries));
// 声明无序：BaseStream.unordered()
Map<Buyer, List<Transaction>> salesByBuyer
         = txns.parallelStream()
               .unordered()
               .collect(groupingByConcurrent(Transaction::getBuyer));

```







# 时间日期