## 使用

```java
@Slf4j
public class DisruptorTests {

    @Test
    public void disruptorTest() throws Exception {
        EventFactory<Truck<Integer>> truckFactory = Truck::new;
        ThreadFactory threadFactory = new NamingThreadFactory("disruptor");
        WaitStrategy waitStrategy = new YieldingWaitStrategy();
        int ringSize = 2 << 10;
        Disruptor<Truck<Integer>> disruptor = new Disruptor<>(truckFactory, ringSize, threadFactory, ProducerType.MULTI, waitStrategy);
  
        // 注意消费者需要将 event 中的引用赋值为 null ，防止内存溢出
        WorkHandler<Truck<Integer>> consumer1 = truck -> log.info("consumer1 ------ {}", truck.getData());
        WorkHandler<Truck<Integer>> consumer2 = truck -> log.info("consumer2 ------ {}", truck.getData());
        WorkHandler<Truck<Integer>> consumer3 = truck -> log.info("consumer3 ------ {}", truck.getData());
        EventHandler<Truck<Integer>> consumer4 = (truck, sequence, endOfBatch) -> log.info("consumer4 ------ {}", truck.getData());
        disruptor.handleEventsWithWorkerPool(consumer1, consumer2, consumer3).then(consumer4);
        disruptor.start();

        // ProducerType.MULTI 类型可以多线程发布
        for (int i = 0; i < 20; i++) {
            Integer data = i;
            disruptor.publishEvent((truck, sequence) -> truck.load(data));
        }

        disruptor.shutdown();
    }
}

@Builder
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class Truck<T> {

    private T data;

    public void load(T data) {
        this.data = data;
    }
}
```

## 概念

* `RingBuffer`：环形队列，负责缓冲生产者生产的消息，包含一个 `EventFactory` 和 一个 `Sequencer`。
* `EventFactory`：事件工厂，`RingBuffer` 初始化时会调用事件工厂生成所有的事件载体，事件载体可以认为是 装货的卡车，至于装什么事件取决于生产者往卡车里放什么事件。
* `Sequencer`：生产者序列号产生器，每一个事件都有对应的序列号，`RingBuffer` 中生产者的同步机制，包含 生产者的序列号(`cursor`) 和 消费者的序列号(`gatingSequences`) 的引用。
* `Sequence`：序列号，使用 CAS 的方式增长，并且使用填充属性解决伪共享的问题。
* `SequenceBarrier`：消费者序列号屏障，由 `Sequencer` 创建 被 `EventProcesser` 持有，用来判断消费者是否可以拿到待处理的事件(生产是否到位等)，具体逻辑由 `WaitStrategy` 来实现。
* `EventProcesser`：消费者，通过 `SequenceBarrier` 获取消费的序列号从而消费对应事件。
* `WaitStrategy`: 消费者获取序列号的策略,比如阻塞等待唤醒的`BlockingWaitStrategy`,疯狂压榨cpu自旋追求响应时间的`YieldingWaitStrategy`。
* `Disruptor`：DSL API，聚合生产者和消费者操作，也是我们主要使用的API。

## 核心思想

1. 获取序列号、发布序列号二阶段提交+cas操作替代重量级的锁
2. 环形队列预创建对象,避免频繁的gc。
3. 序列号缓存填充,伪缓存共享。
4. 环形队列数组索引计算，使用位运算替代取模运算

## 组件

### Sequence

将对象填充到128个字节，解决伪共享问题，可以解决大小为64字节缓存行伪共享问题，不能解决128字节。

```java
class LhsPadding
{
    protected long p1, p2, p3, p4, p5, p6, p7;
}

class Value extends LhsPadding
{
    protected volatile long value;
}

class RhsPadding extends Value
{
    protected long p9, p10, p11, p12, p13, p14, p15;
}
public class Sequence extends RhsPadding
{
    static final long INITIAL_VALUE = -1L;
    private static final Unsafe UNSAFE;
    private static final long VALUE_OFFSET;

    static
    {
        UNSAFE = Util.getUnsafe();
        try
        {
            VALUE_OFFSET = UNSAFE.objectFieldOffset(Value.class.getDeclaredField("value"));
        }
        catch (final Exception e)
        {
            throw new RuntimeException(e);
        }
    }
}
```

### RingBuffer

`RingBuffer` 的访问量也是极高的，也进行缓存填充。

创建 `RingBuffer` 时已经创建好所有的 event，当消费完成后需要将event中的引用设置为 null ，防止 `BufferSize` 太长导致占用大量内存，导致内存溢出。

可用 `RingBuffer` 创建 `SequenceBarrier`，该 `SequenceBarrier` 使用 `cursor`（生产者序列号）作为依赖序列号，避免消费者消费未生产的事件。

实现 `Cursored`，获取生产者序列号。

实现 `EventSequencer`，可通过 `sequence` 获取事件。

实现 `EventSink`，可发布事件。

```java
abstract class RingBufferPad
{
    protected long p1, p2, p3, p4, p5, p6, p7;
}

abstract class RingBufferFields<E> extends RingBufferPad
{
    private static final int BUFFER_PAD;
    private static final long REF_ARRAY_BASE;
    private static final int REF_ELEMENT_SHIFT;
    private static final Unsafe UNSAFE = Util.getUnsafe();

    static
    {
        final int scale = UNSAFE.arrayIndexScale(Object[].class);
        if (4 == scale)
        {
            REF_ELEMENT_SHIFT = 2;
        }
        else if (8 == scale)
        {
            REF_ELEMENT_SHIFT = 3;
        }
        else
        {
            throw new IllegalStateException("Unknown pointer size");
        }
        BUFFER_PAD = 128 / scale;
        // Including the buffer pad in the array base offset
        REF_ARRAY_BASE = UNSAFE.arrayBaseOffset(Object[].class) + (BUFFER_PAD << REF_ELEMENT_SHIFT);
    }

    private final long indexMask;
    private final Object[] entries;
    protected final int bufferSize;
    protected final Sequencer sequencer;
}

public final class RingBuffer<E> extends RingBufferFields<E> implements Cursored, EventSequencer<E>, EventSink<E>
{
    public static final long INITIAL_CURSOR_VALUE = Sequence.INITIAL_VALUE;
    protected long p1, p2, p3, p4, p5, p6, p7;
}
```

### Sequencer

生产者序列号器，负责产生生产者事件的序列号，持有消费者序列号的引用，有单线程(`SingleProducerSequencer`)和多线程(`MultiProducerSequencer`)两个版本。

实现 `Sequenced`，可获取下一个序列号，发布序列号等功能。

```java
public interface Sequencer extends Cursored, Sequenced
{
    long INITIAL_CURSOR_VALUE = -1L;

    void claim(long sequence);

    boolean isAvailable(long sequence);

    void addGatingSequences(Sequence... gatingSequences);

    boolean removeGatingSequence(Sequence sequence);

    SequenceBarrier newBarrier(Sequence... sequencesToTrack);

    long getMinimumSequence();

    long getHighestPublishedSequence(long nextSequence, long availableSequence);

    <T> EventPoller<T> newPoller(DataProvider<T> provider, Sequence... gatingSequences);
}
```

### SingleProducerSequencer

填充并不是只能对单个Long类型变量，多个变量也可以，即填充之后就能够保证多个变量在一个缓存行中，且这些变量永远不会和其他变量在同一个缓存行中。

`next` 返回下一个序列号，这时候未发布，消费者不能消费，需要调用 `publish` 将 `cursor` 设置为改值时，消费者才能消费，实现两阶段提交。

`cachedValue` 负责缓存消费者的最小值，不一定是最小值，但一定比最小值小，防止生产者越界。

`SequenceBarrier` 由 `AbstractSequencer` 产生。

```java
public abstract class AbstractSequencer implements Sequencer
{
    private static final AtomicReferenceFieldUpdater<AbstractSequencer, Sequence[]> SEQUENCE_UPDATER =
        AtomicReferenceFieldUpdater.newUpdater(AbstractSequencer.class, Sequence[].class, "gatingSequences");

    protected final int bufferSize;
    protected final WaitStrategy waitStrategy;
    // 生产者序列号
    protected final Sequence cursor = new Sequence(Sequencer.INITIAL_CURSOR_VALUE);
    // 消费者序列号
    protected volatile Sequence[] gatingSequences = new Sequence[0];
}

abstract class SingleProducerSequencerPad extends AbstractSequencer
{
    protected long p1, p2, p3, p4, p5, p6, p7;

    SingleProducerSequencerPad(int bufferSize, WaitStrategy waitStrategy)
    {
        super(bufferSize, waitStrategy);
    }
}

abstract class SingleProducerSequencerFields extends SingleProducerSequencerPad
{
    SingleProducerSequencerFields(int bufferSize, WaitStrategy waitStrategy)
    {
        super(bufferSize, waitStrategy);
    }

    /**
     * Set to -1 as sequence starting point
     */
    long nextValue = Sequence.INITIAL_VALUE;
    long cachedValue = Sequence.INITIAL_VALUE;
}

public final class SingleProducerSequencer extends SingleProducerSequencerFields
{
    protected long p1, p2, p3, p4, p5, p6, p7;
}
```

### MultiProducerSequencer

`gatingSequenceCache` 同样也是缓存消费者的最小值，但类型是 `Sequence`，由于这时候生产者是多线程的，`Sequence` 的 set 方法最终调用 `putLongVolatile` 方法，实现了 `Volatile` 语言保证多线程可见。

`availableBuffer` 用来标记 `sequence` 是否能被消费者消费， 默认值为 -1，当 `sequence` 被发布时变为 0。

```java
public final class MultiProducerSequencer extends AbstractSequencer
{
    private static final Unsafe UNSAFE = Util.getUnsafe();
    private static final long BASE = UNSAFE.arrayBaseOffset(int[].class);
    private static final long SCALE = UNSAFE.arrayIndexScale(int[].class);

    private final Sequence gatingSequenceCache = new Sequence(Sequencer.INITIAL_CURSOR_VALUE);

    // availableBuffer tracks the state of each ringbuffer slot
    // see below for more details on the approach
    private final int[] availableBuffer;
    private final int indexMask;
    private final int indexShift;
}
```

### SequenceBarrier

负责控制消费者需要等依赖的序列号发布后才能消费，并且保证消费的顺序性。

```java
final class ProcessingSequenceBarrier implements SequenceBarrier
{
    private final WaitStrategy waitStrategy;
    private final Sequence dependentSequence;
    private volatile boolean alerted = false;
    private final Sequence cursorSequence;
    private final Sequencer sequencer;

    ProcessingSequenceBarrier(
        final Sequencer sequencer,
        final WaitStrategy waitStrategy,
        final Sequence cursorSequence,
        final Sequence[] dependentSequences)
    {
        this.sequencer = sequencer;
        this.waitStrategy = waitStrategy;
        this.cursorSequence = cursorSequence;
        // 当 Disruptor 调用 handleEventsWith、handleEventsWithWorkerPool 时 dependentSequences 为 new Sequence[0] ，表示依赖 cursor 即生产者的序列号
        if (0 == dependentSequences.length)
        {
            dependentSequence = cursorSequence;
        }
        // 当 EventHandlerGroup 调用 then、thenHandleEventsWithWorkerPool 时 dependentSequences 为 前一个阶段的消费者 序列数组，即依赖 前一个阶段消费者
        else
        {
            dependentSequence = new FixedSequenceGroup(dependentSequences);
        }
    }
}
```

### EventProcessor

消费者，实现 `Runable` ，通过调用 `halt` 来停止消费者。

#### BatchEventProcessor

1. `BatchEventProcessor`可以处理超时，可以处理中断，可以通过用户实现的异常处理类处理异常，发生异常之后再次启动，不会漏消费，也不会重复消费。
2. 多个 `BatchEventProcessor` 并行时互不干扰，如果 生产者生产 10 个事件，每个消费者都会消费 10 个事件。

#### WorkProcessor & WorkerPool

一个 `WorkerPool` 由多个 `WorkProcessor` 组成，共享一个 `workSequence`，可形成互斥消费，如果 生产者生产 10 个事件，一个 `WorkerPool` 中每个消费者消费一部分，所有消费者消费的事件总数为 10

### WaitStrategy

1. `BlockingWaitStrategy`: 默认策略，使用锁进行数据监控和线程等待、唤醒，最节省CPU，但由于需要线程切换在高并发场景下性能最糟糕。
2. `SleepingWaitStrategy`: 会进行自旋等待，如果不成功会调用yield让出CPU，并最终使用 `LockSupport.parkNanos(可配置时间)` 进行线程休眠，以确保不占用过多CPU，可能会产生较高的平均延时，对延时要求不是特别高的场合，对生产者线程影响最小，典型场景就是异步日志。
3. `YieldingWaitStrategy`：用于低延时环境，内部是一个 yield 的死循环，会消耗大量的CPU资源，相当于绑定该核心，最好有多于该消费者线程数量的核心数量。
4. `BusySpinWaitStrategy`：相比 `YieldingWaitStrategy` 更加疯狂，不会让出CPU，吃掉所有的CPU资源，所以物理核必须要多于消费者数目，即使存在逻辑核也会影响无法工作。
5. `LiteBlockingWaitStrategy`：基于 `BlockingWaitStrategy` 的轻量级等待策略，在没有锁竞争的时候会省去唤醒操作。
6. `TimeoutBlockingWaitStrategy`：基于 `BlockingWaitStrategy` 的带超时策略，超时后会抛出异常。
7. `LiteTimeoutBlockingWaitStrategy`：基于 `BlockingWaitStrategy` 的轻量级带超时策略，在没有锁竞争的时候会省去唤醒操作，超时后会抛出异常。
