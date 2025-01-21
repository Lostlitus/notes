# Concurrency control

The word *concurrency* refers to situations in which multiple instruction
streams are interleaved, due to multiprocessor parallelism, thread switching,
or interrupts. Method to ensure correctness under concurrency is called
*concurrency control*.

A *race condition* is a situation in which a memory location is accessed
concurrently, and at least one access is a write. Data is consistent before and
after modification, but may temporary loses its consistency during the
modification. We name this kind of code segement as *critical section*. If
someone reads the intermediate data modified in critical section and uses it,
bug grows.

## Locks

A usual way to avoid races is to use a lock. There are various kinds of locks,
each has its own pros and cons. But the common part is that they aim to ensure
serialization. We are going to talk about the some most common ones here.

### Start with spinlock

A spinlock is made up with a `locked` state and two functions, `acquire` and
`release`. The `acquire` function sets the `locked` to 1 if it is 0. The
`release` function sets the state back to 0 if it is 1.

```C
struct spinlock lock;

void worker()
{
        acquire(&lock);
        // Working.
        release(&lock);
}
```

Function `acquire` is blocked until `locked` is set to 0, so only one worker
here can go next. Only after function `release` is invoked, can one blocked
worker be released.

The implementation of `acquire` looks similar to this.

```C
void acquire(struct spinlock *lock)
{
        while (1) {
                if (lock->locked == 0) {
                        lock->locked = 1;
                        break;
                }
        }
}
```

Yes, it just keeps checking the `locked` state in a loop without stop. However,
the implementation above is incorrect. What if two workers find the state is
set to 0 in the same time? They both break from the loop, leads to multiple
workers working together.

The problem here is that reading and writing to the state are splitted into two
step. To solve this, hardware provides a instruction to swap the state with a
given value and return the state. And language provides portable interface
which boils down to this kind of instruction depending on the platform. For
example in C, we have `__sync_lock_test_and_set`.  Now we have a better
`acquire` like this.

```C
void acquire(struct spinlock *lock)
{
        while (1) {
                if (__sync_lock_test_and_set(&lock->locked, 1) == 0) {
                        break;
                }
        }
}
```

When state is set to 1(i.e. acquired by others), the instruction swaps 1 to the
state and returns the state value 1. So it keeps looping. When set to 0, on the
contrary, it sets the state to 1 and gets 0. So it breaks from loop.

This is the reason why this kind of lock is called spinlock, since it keeps
spinning (swapping) on waiting.

Nevertheless, there still exists one problem. Compiler and CPU may reorder our
instructions, so *memory barrier* is needed to tell them not to go across the
line of `acquire`.  Or the work may be executed before `acquire` is invoked in
actual.

```C
void acquire(struct spinlock *lock)
{
        while (1) {
                if (__sync_lock_test_and_set(&lock->locked, 1) == 0) {
                        break;
                }
        }

        __sync_synchronize();
}
```

The `__sync_synchronize` here is the portable interface of memory barrier from
C library.

The same as function `release`.

```C
void release(struct spinlock *lock)
{
        __sync_synchronize();

        __sync_lock_release(&lock);
}
```

Most language does not force compiler to implement an assignment as atomic. So
we replace statement like `lock->locked = 0` with portable atomic assignment
interface in C, the `__sync_lock_release` above.

One first read the implementation of spinlock may be shocked by how it wastes
CPU resource since it keeps spining. However, it's affordable if the spining
last for a short time. For example, a thread acquires the spinlock, finishes
its job in several nanosecond and releases the spinlock. In this case, other
threads may only wait for several nanosecond as well. So not be too nervous to
use spinlock in code.

### From sleep to condition variable

If the code is doing some long-wait job in critical section like disk IO,
spinlock is inappropriate. The spinlock will be held for a long time (from
several millisecond to an hour) and other threads just block on wait.

```C
static spinlock lock;
static int resource = 0;

extern int longWaitJob();

void producer()
{
        // Producing
        acquire(&lock);
        resource += longWaitJob();
        release(&lock);
}

void consumer()
{
        // Consuming
        while (1) {
                acquire(&lock)
                if (resource != 0) {
                        resource = 0;
                }
                release(&lock);
        }
}
```

The loop in `consumer` keeps consuming CPU resource but does nothing until the
`producer` finishes. Actually, it's OK even if it waits for a long time, but it
should release the CPU to others.  So here comes `sleep`.

`sleep` works with `wakeup`. What `sleep` does is that it yields the CPU to
others and sets itself as *sleeping*, then kernel won't schedule CPU for this
sleeping thread until someone call `wakeup` to change the state to *runable*.
The interface of `sleep` and `wakeup` may look wired the first time one see
them, so now we give it by building them step by step. Say they does not need
any argument.

```C
static int resource = 0;

extern int longWaitJob();

void producer()
{
        // Producing
        resource += longWaitJob();
        wakeup();
}

void consumer()
{
        // Consuming
        if (resource == 0) {
                sleep();
        }
        resource = 0;
}
```

The problem of CPU resource wasting is solved. But two new problems appear. The
first is both `wakeup` and `sleep` need a channel-like argument to communicate,
or `wakeup` does not know which `sleep` thread to wake.

```C
static const int chan = 123;
static int resource = 0;

extern int longWaitJob();

void producer()
{
        // Producing
        resource += longWaitJob();
        wakeup(chan);
}

void consumer()
{
        // Consuming
        if (resource == 0) {
                sleep(chan);
        }
        resource = 0;
}
```

This way, `sleep` indicates it is sleeping in channel `chan` and `wakeup` says
it wakes threads in channel `chan`.

Another problem here is that the critical section here is not protected, so
race occurs. For example, consumer may find `resource == 0` returns false, but
right before it invokes `sleep(chan)`, producer add resource and invokes
`wakeup(chan)`, then consumer would sleep forever. This case is called *lost
wake-up*.  We should add lock to serialize read and wirite. But adding spinlock
here is not correct as well.

```C
static spinlock lock;
static const int chan = 123;
static int resource = 0;

extern int longWaitJob();

void producer()
{
        // Producing
        acquire(&lock)
        resource += longWaitJob();
        wakeup(chan);
        release(&lock)
}

void consumer()
{
        // Consuming
        acquire(&lock)
        if (resource == 0) {
                sleep(chan);
        }
        resource = 0;
        release(&lock)
}
```

If the statement `resource == 0` return true, then consumer holds the lock
forever.  And producer would never be able to reach line `wakeup(chan);` since
it blocks on line `acquire(&lock);` forever.

The point is to hold the lock before sleep but release it after sleep. So here
comes new interface.

```C
static spinlock lock;
static const int chan = 123;
static int resource = 0;

extern int longWaitJob();

void producer()
{
        // Producing
        acquire(&lock)
        resource += longWaitJob();
        wakeup(chan);
        release(&lock)
}

void consumer()
{
        // Consuming
        acquire(&lock)
        if (resource == 0) {
                sleep(chan, &lock);
        }
        resource = 0;
        release(&lock)
}
```

We pass the lock as argument to sleep and it would release the lock before
actually yielding the CPU.

Now we only has one more case. If there are multiple producers and consumers,
invoking `wakeup` may wake all consumers up (called *thundering herd*), but
only one consumer gets the resource, other consumers suffer *spurious
wakeup* and return the function with nothing done. So we add a loop in
addition to resolve spurious wakeup.

```C
static spinlock lock;
static const int chan = 123;
static int resource = 0;

extern int longWaitJob();

void producer()
{
        // Producing
        acquire(&lock)
        resource += longWaitJob();
        wakeup(chan);
        release(&lock)
}

void consumer()
{
        // Consuming
        acquire(&lock)
        while (resource == 0) {
                sleep(chan, &lock);
        }
        resource = 0;
        release(&lock)
}
```

Now only one consumer would consume the resource, others just loop again to
wait for next producer.

That's all about `sleep` and `wakeup`. The modern implementation of these two
may differ, since it needs optimizations. But the core is idential.  Checkes
the section below.

### Talk from condition variable

`condition variable` is considered as a modern interface of `sleep` & `wakeup`
pair.  For example, it uses a better data structure than `chan`, like a wait
queue.  This way kernel needs not to scan the whole thread list to wake up
sleeping threads, instead, it fetches from the queue. Other optimizations
including accepting the condition as an argument, so user needs not to wrap the
sleep with a loop explicitly.  Besides, it allows to choose between waking one
consumer up or waking all consumers up.  All these optimizations are achieved
by C++ `std::condition_variable`.

```C++
static std::mutex lock;
static std::condition_variable cv;
static int resource = 0;

extern int longWaitJob();

void producer() {
        // Producing
        std::unique_lock<std::mutex> uniLock(lock);

        resource += longWaitJob();
        cv.notify_one();
}

void consumer() {
        // Consuming
        std::unique_lock uniLock(lock);

        cv.wait(uniLock, []{return resource != 0});

        resource = 0;
}
```

The `notify_one` here only wakes one consumer up. If waking all consumers up is
needed, `notify_all` is available.

Other than these performance and convenience optimizations.  Condition variable
has safety optimizations as well.

We see that `wait` accepts an exception safe lock wrapper type
`std::unique_lock<std::mutex>` instead of a bare lock, which type obeys RAII.
In real world, code may throw an exception after the bare lock is acquired. If
the exception handling code does not release the lock manually, other threads
will be blocked forever. With exception safe lock wrapper, at least the bare
lock would be released correctly.

Another point here is that we use `std::mutex` instead of spinlock, why? Well,
the most straightforward reason is because C++ does not provide spinlock. But
why?  The quite shocking answer is that spinlock is considered harmful in user
space.  Detail talk is as follows.

### Do not use spinlock

Like we talked above, spinlock just keeps spining when waiting someone to
release the lock. But during this period, the OS scheduler does not know this
thread is waiting.  Instead, the scheduler may even consider this thread as
doing some heavy job, since it never stops. As a result, the remaining time
slice assigned to this spining thread is wasted.

One may think that it is acceptable like we talked above in spinlock section,
since after this the scheduler would arrange CPU to other threads and finally
the one holding the lock would have chance to release it. Then the spining
thread is freed.

In real world, however, the case can get even worse. Say we have several
threads sharing the same spinlock, and one thread is preempted before releasing
the spinlock and scheduler arranges CPU to other threads. In this case, the
origin thread owning the lock may need to wait all the others to finish wasting
all their time slice in spining. Yes, in this round of schedule, all threads
does nothing. They just spining like a drug addict.

And even even worse. The cases above wasting time, but in the end it would be
the origin thread's turn to release the lock. That is, though the program is
poor in performance, it works. But disaster happens when these threads are
granted with priority.  Same scenario, but the thread holding the lock has low
priority and the threads count is more than CPU count. Then scheduler would
never assign CPU to this thread since there are other high-priority threads, so
the lock would never be released, so all other threads suck. The server now
does nothing but increasing your electricity bill. This is called *prioroty
inversion*.

What we talked above happens in user space, since kernel space does not has a
scheduler to automatically switch context. Kernel switches context manually,
so it could switch to the thread that owns the lock. But there still exists one
problem. Interrupt is the exception, it can preempt kernel thread. If a kernel
thread is interrupted after acquiring the lock, and the interrupt handler
acquires the lock as well, then *deadlock* occurs.  So modern kernel spinlock
would disable the interrupt until the lock is released. (This policy is applied
on other kind of kernel lock as well)

So it's OK to use spinlock in kernel space, but what should we use to replace
spinlock in user space?

### Use modern synchronization primitives to replace spinlock in user space

All the problems above come from one point that kernel does not know the
spining thread is blocked, or it would assgin CPU to the thread owning the
lock. So modern synchronization primitives tell information about it to kernel,
and problem solved.

#### 1. Mutex

Actually, spinlock is one kind of implementation of `mutex`. But modern mutex
chooses a hybrid implementation for mutex. It spins like spinlock for a while,
and if lock is still held by others, it works like `sleep` talked above.  So
scheduler would not arrange CPU for it, no waste anymore. When you need
spinlock, replace it with mutex.

#### 2. Condition variable

As we talked above, condition variable is the modern interface for user to
achieve `sleep` & `wakeup` stuff. It yields CPU immediately after invoking
`wait`.  So compared to mutex, it wouldn't spin. Just like the case `sleep`
replaces spinlock, condition variable replaces spinlock when the critical
section would have a long-time job.

#### 3. Semaphore

`semaphore` is basically a mutex with a count initialized higher than 0. When
acquiring, minuses the count by 1 and if it's already 0, block on wait. When
releasing, increases the count by 1. It works like condition variable, that is,
the block thread would sleep instead of spin. But its use case differs. It's
mostly used in the case that resource is limited. For example, thread pool,
queued requests should block until there is one idle thread.
