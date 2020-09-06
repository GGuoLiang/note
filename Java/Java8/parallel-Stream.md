# ParallelStream

**并行流就是一个把内容拆分成多个数据块，用不同线程分别处理每个数据块的流。**对收集源调用parallelStream方法就能将集合转换为并行流。

## 并行流

### 并行流和顺序流转换

parallel 和 sequential

~~~java
Integer reduce = Stream.iterate(0, n -> n + 2).limit(10000).reduce(1, Integer::sum);
// 将顺序流转化为并行流
Integer reduce1 = Stream.iterate(0, n -> n + 2).limit(10000).parallel().reduce(1, Integer::sum);
// 将并行流转为顺序流
Integer reduce2 = Stream.iterate(0, n -> n + 2).limit(10000).parallel().map(integer -> integer + 2).sequential().reduce(1, Integer::sum);
~~~

最后一次parallel或sequential调用会影响整个流水线

> 配置并行流使用的线程池：
>
> 1. 并行流内部使用了默认的ForkJoinPool。它默认的线程数量就是你的处理器数量，这个值是由Runtime.getRuntime().availableProcessors()得到的。
>
> 2. 可以通过系统属性java.util.concurrent.ForkJoinPool.common.parallelism来修改线程池大小
>
>    ~~~java
>    System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","12");
>    System.out.println( System.getProperty("java.util.concurrent.ForkJoinPool.common.parallelism"));
>    ~~~
>
> 3. 这是一个全局设置，因此它会对代码中所有的并行流产生影响。反过来说，目前我们还无法专为某个并行流指定这个值。一般而言，让ForkJoinPool的大小等于处理器数量是个不错的默认值，除非你有很充足的理由，否则强烈建议你不要修改它。
>
> 

### 正确的姿势使用并行流

**并行流并不总是比顺序流快**。所以正确的姿势使用并行流是尤为重要的，不然适得其反。

决定某个特定情况下是否有必要使用并行流。可以参考一下几点建议

1. 如果有疑问，测量。并行流有时候会和你的直觉不一致，所以在考虑选择顺序流还是并行流时，很重要的建议就是用适当的基准来检查其性能。

2. 留意装箱。自动装箱和拆箱操作会大大降低性能。Java 8中有原始类型流（IntStream、LongStream和DoubleStream）来避免这种操作，但凡有可能都应该用这些流

3. 有些操作本身在并行流上的性能就比顺序流差。特别是limit和findFirst等依赖于元素顺序的操作，它们在并行流上执行的代价非常大。例如，findAny会比findFirst性能好，因为它不一定要按顺序来执行。你总是可以调用unordered方法来把有序流变成无序流。那么，如果你需要流中的N个元素而不是专门要前N个的话，对无序并行流调用limit可能会比单个有序流（比如数据源是一个List）更高效。

4. 考虑流的操作流水线的总计算成本。设N是要处理的元素的总数，Q是一个元素通过流水线的大致处理成本，则N*Q就是这个对成本的一个粗略的定性估计。Q值较高就意味着使用并行流时性能好的可能性比较大。

5. 对于较小的数据量，选择并行流几乎从来都不是一个好的决定。并行处理少数几个元素的好处还抵不上并行化造成的额外开销。

6. 考虑流背后的数据结构是否易于分解。例如，ArrayList的拆分效率比LinkedList高得多，因为前者用不着遍历就可以平均拆分，后者则必须遍历。另外，用range工厂方法创建的原始类型流也可以快速分解。可以参考一下表格：

   | 数据源          | 性能 |
   | --------------- | ---- |
   | ArrayList       | 极佳 |
   | LinkedList      | 差   |
   | IntStrean.range | 极佳 |
   | Strean.iterate  | 差   |
   | HashSet         | 好   |
   | TreeSet         | 好   |

   

7. 流自身的特点以及流水线中的中间操作修改流的方式，都可能会改变分解过程的性能。例如，一个SIZED流可以分成大小相等的两部分，这样每个部分都可以比较高效地并行处理，但筛选操作可能丢弃的元素个数无法预测，从而导致流本身的大小未知。

8. 还要考虑终端操作中合并步骤的代价是大是小（例如Collector中的combiner方法）。如果这一步代价很大，那么组合每个子流产生的部分结果所付出的代价就可能会超出通过并行流得到的性能提升。

## 分支/合并框架

分支/合并框架的目的是以递归方式将可以并行的任务拆分成更小的任务，然后将每个子任务的结果合并起来生成整体结果。它是ExecutorService接口的一个实现，它把子任务分配给线程池（称为ForkJoinPool）中的工作线程。

