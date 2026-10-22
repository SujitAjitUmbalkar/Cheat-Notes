# Java Executor Framework: 

The Java Executor Framework helps me submit tasks and lets Java manage how those tasks run. Instead of creating and controlling a thread for every task myself, I can use an executor and, commonly, a thread pool. I should remember the difference between a task (the work), a thread (the worker), and a thread pool (reusable workers).

## Syllabus covered

* Multithreading basics and why the Executor Framework is needed
* Executor interface and `execute(Runnable)`
* `ExecutorService`, `submit()`, `Callable`, and `Future`
* `ThreadPoolExecutor`, workers, queues, pool sizing, and rejection
* Creating pools with `Executors` and `ThreadPoolExecutor`
* `ScheduledExecutorService`, delayed tasks, fixed rate, and fixed delay
* Shutdown, complete examples, comparisons, and common mistakes

---

## 1. Multithreading basics

A thread is a path of execution inside a program. I can think of a thread as a worker. A task is the work the worker has to do. Multithreading means a program can use multiple threads so different tasks can make progress concurrently. Depending on the computer and scheduling, threads may run at the same time on different CPU cores or take turns.

### Why do we need it?

For example, an application may download a file while updating a progress bar. If everything were done by one thread, one activity could make another wait. Multiple threads can help keep work responsive and handle several tasks.

### Creating a thread manually

```java
public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            System.out.println("Worker is doing a task");
        });

        t1.start(); // Starts a new thread
    }
}

```

*Important:* `start()` starts a separate thread. Calling `run()` directly would just call the method on the current thread; it would not start a new thread.

### Why not create a new thread for every task?

Creating and managing a very large number of threads costs memory and CPU time. Too many threads can hurt performance. A thread pool reuses worker threads, so a worker can finish one task and then take another.

---

## 2. Java Executor Framework

The Executor Framework was introduced in JDK 5 and is in `java.util.concurrent`. It separates the task I want to run from the way it is executed. I submit work to an executor; the implementation decides whether to use a new thread, a pooled thread, or another execution strategy.

### Basic flow

1. I define a task, commonly as a `Runnable` or `Callable`.
2. I submit the task to an executor.
3. The executor implementation arranges execution.
4. If it uses a thread pool, a worker executes the task and may later be reused.

*Key memory line:* I describe what work should happen; the executor manages how it gets executed.

---

## 3. Executor interface

`Executor` is the basic interface. It has `execute(Runnable command)`. This method submits a Runnable for execution, but it does not return a result or a `Future`.

```java
void execute(Runnable command);

```

### Runnable example

```java
Runnable task = () -> {
    System.out.println("Hello from the task");
};

Executor executor = command -> new Thread(command).start();
executor.execute(task);

```

In this example, I made a simple custom `Executor` that starts a new thread for each task. Another `Executor` implementation could reuse a pooled thread or run the task in the caller's thread. So `execute()` does not guarantee that a new thread is created.

### What is Runnable?

`Runnable` represents work that has a `run()` method and does not return a result through the Runnable interface. It is commonly written with a lambda when the work is short.

---

## 4. ExecutorService interface

`ExecutorService` extends `Executor`. It still supports `execute()`, and adds more methods for submitting tasks, retrieving results, and managing shutdown.

| Method | My understanding |
| --- | --- |
| `execute(Runnable)` | Submit a Runnable; no Future is returned. |
| `submit(Runnable)` | Submit work and receive a `Future<?>`. |
| `submit(Callable<T>)` | Submit work that returns `T` and receive `Future<T>`. |
| `shutdown()` | Stop accepting new tasks and allow submitted tasks to finish. |
| `shutdownNow()` | Attempt to interrupt running tasks and return tasks that never started. |
| `isShutdown()` | Check whether shutdown has been initiated. |
| `isTerminated()` | Check whether shutdown has completed and all tasks are finished. |
| `awaitTermination(time, unit)` | Wait up to a time limit for the executor to terminate. |
| `invokeAll(tasks)` | Submit a collection of tasks and wait for them to complete. |
| `invokeAny(tasks)` | Return one successful completed result, subject to the method's contract. |

### execute() vs submit()

| Feature | execute() | submit() |
| --- | --- | --- |
| Task type | Runnable | Runnable or Callable |
| Returns Future? | No | Yes |
| Get result through Future? | No | Yes, if the task produces a result |
| Failure observation | Depends on executor/execution context | Task failure can be observed from `Future.get()` |

I use `execute()` when I only need to submit work. I use `submit()` when I may need a `Future` to check completion, cancel, or retrieve a result.
---
# NOTE

### Note: Instantiating `Executor` vs. `ExecutorService` in Java

---

###  The Core Rule: Interfaces vs. Concrete Classes

* Both `Executor` and `ExecutorService` are Java **interfaces**, meaning they cannot be instantiated directly using the `new` keyword (doing so triggers a compilation error stating they are abstract and cannot be instantiated).
* To use them, developers must either supply a custom implementation or utilize Java's built-in concrete implementation classes.

###  Instantiating the `Executor` Interface

* Because `Executor` is a **Functional Interface** containing only a single abstract execution method, it can be instantiated very simply.
* **Anonymous Inner Classes:** Developers can define an inline, unnamed implementation of the interface directly where it is needed, overriding its single method to dictate how commands run (such as spawning a brand-new thread for every task).
* **Lambda Expressions:** Given its single-method nature, a lambda expression can replace the anonymous class entirely, providing a concise and clean way to implement the interface on the fly.


###  Instantiating the `ExecutorService` Interface

* In contrast to `Executor`, `ExecutorService` contains multiple abstract methods responsible for managing a comprehensive lifecycle (including task submission, shutdown sequences, future tracking, and queue management).
* **Why Lambdas and Anonymous Classes Fail:** Lambdas cannot be used because the interface has multiple abstract methods. While writing an anonymous inner class is syntactically possible, it is highly impractical because it forces the developer to manually write boilerplate overrides for dozens of distinct lifecycle and invocation methods.
* **The Correct Approach (Concrete Classes):** `ExecutorService` should always be instantiated using Java's built-in concrete implementation classes from the concurrent package:
* **`ThreadPoolExecutor`:** The standard, highly customizable class used to manage core and maximum pool sizes, task queueing, and idle thread timeouts.
* **`ScheduledThreadPoolExecutor`:** A specialized subclass tailored specifically for delayed or recurring task scheduling.
* **`ForkJoinPool`:** An optimization-focused class designed for work-stealing algorithms and recursive, divide-and-conquer workloads.

---

## 5. Runnable, Callable, and Future

### Runnable

```java
Runnable task = () -> {
    System.out.println("Processing order");
};

```

Runnable's method is `run()`. It does not return a value through its interface and cannot declare checked exceptions in `run()`.

### Callable

```java
Callable<Integer> task = () -> {
    return 10 + 20;
};

```

Callable's method is `call()`. It returns a value of type `T` and can throw checked exceptions.

### Future

`Future` is like a handle for a result that may be ready later. The task may be running, finished, failed, or cancelled. Calling `get()` waits for the result if it is not ready yet.

| Future method | Meaning |
| --- | --- |
| `get()` | Wait for and retrieve the result. |
| `get(timeout, unit)` | Wait for a limited time. |
| `isDone()` | Check whether the task is complete. |
| `isCancelled()` | Check whether it was cancelled. |
| `cancel(boolean)` | Attempt to cancel the task. |

### Runnable + Future example

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService service =
                Executors.newSingleThreadExecutor();

        Callable<Integer> task = () -> 10 + 20;
        Future<Integer> future = service.submit(task);

        try {
            System.out.println(future.get()); // 30
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            System.err.println("Task failed: " + e.getCause());
        } finally {
            service.shutdown();
        }
    }
}

```

`get()` may throw `InterruptedException` if the waiting thread is interrupted, and `ExecutionException` if the task failed. If I need a timeout, I can use `get(timeout, unit)`, which may throw `TimeoutException`.

---

## 6. ThreadPoolExecutor

`ThreadPoolExecutor` is a common implementation of `ExecutorService`. It manages worker threads and a task queue. A worker executes a task; when it finishes, the worker can be reused for another task.

### Important configuration

| Setting | Meaning |
| --- | --- |
| `corePoolSize` | Number of core workers the pool generally maintains. |
| `maximumPoolSize` | Maximum number of worker threads allowed. |
| `keepAliveTime` | How long eligible idle workers beyond the core size can remain alive. |
| `workQueue` | Holds tasks waiting to be executed. |
| `ThreadFactory` | Creates worker threads. |
| `RejectedExecutionHandler` | Defines what happens when a task cannot be accepted. |

### How tasks are generally handled

1. If fewer than `corePoolSize` workers are running, the executor generally creates a worker for a new task.
2. Once core workers are busy, the executor tries to put new tasks in the queue.
3. If the queue cannot accept a task, the executor may create more workers up to `maximumPoolSize`.
4. If it cannot queue the task or create another worker, the rejection policy is applied.

This is the standard `ThreadPoolExecutor` behavior. The exact behavior depends on the queue type, settings, and rejection handler. An unbounded queue usually means the pool does not need to grow beyond the core size in normal operation.

### Example: configure ThreadPoolExecutor

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ThreadPoolExecutor executor =
            new ThreadPoolExecutor(
                2, // core pool size
                4, // maximum pool size
                30,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(10)
            );

        for (int i = 1; i <= 6; i++) {
            final int taskId = i;
            executor.execute(() -> {
                System.out.println(
                    "Task " + taskId + " on " +
                    Thread.currentThread().getName()
                );
            });
        }

        executor.shutdown();
    }
}

```

This pool has two core workers, can grow to four workers when the queue is full, and has a queue capacity of ten. The default rejection policy is `AbortPolicy`. Output order is not guaranteed.

### Rejection policies

| Policy | What it does |
| --- | --- |
| `AbortPolicy` | Throws `RejectedExecutionException`. |
| `CallerRunsPolicy` | Runs the task in the submitting thread if the executor is not shut down. |
| `DiscardPolicy` | Silently discards the rejected task. |
| `DiscardOldestPolicy` | Removes the oldest queued task and retries submission if the executor is not shut down. |

import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        // Example using CallerRunsPolicy
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            1,                      // corePoolSize
            1,                      // maximumPoolSize
            0L,                     // keepAliveTime
            TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(1), // small queue to trigger rejection
            new ThreadPoolExecutor.CallerRunsPolicy() // Specify the rejection policy here
        );

        // Other options you can pass as the last argument:
        // new ThreadPoolExecutor.AbortPolicy()
        // new ThreadPoolExecutor.DiscardPolicy()
        // new ThreadPoolExecutor.DiscardOldestPolicy()

        for (int i = 1; i <= 3; i++) {
            final int taskId = i;
            executor.execute(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Task " + taskId + " running on " + Thread.currentThread().getName());
            });
        }

        executor.shutdown();
    }
}

---

## 7. Creating pools with Executors

`Executors` is a utility class with factory methods. They make common executor configurations easier to create.
when you use these , default configuration of Pool and Task Queue is Used internally.

### A. Fixed thread pool

```java
ExecutorService executor = Executors.newFixedThreadPool(5);

```

Creates a pool with five core workers and a shared unbounded queue. It limits the number of concurrently executing tasks to five, but the waiting queue can grow if tasks arrive faster than they finish.

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor =
                Executors.newFixedThreadPool(5);

        for (int i = 1; i <= 10; i++) {
            final int id = i;
            executor.execute(() ->
                    System.out.println("Task " + id));
        }

        executor.shutdown();
    }
}

```

### B. Single-thread executor

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

```

Uses one worker to execute tasks sequentially. Tasks submitted normally are executed in submission order. The executor can replace its worker if it terminates unexpectedly.

### C. Scheduled thread pool

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);

```

Supports delayed and periodic tasks.

---

## 8. ScheduledExecutorService

`ScheduledExecutorService` extends `ExecutorService`. It lets me run a task after a delay or repeat it using a schedule.

### A. schedule(): run once after a delay

```java
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(1);

scheduler.schedule(
    () -> System.out.println("Runs after 5 seconds"),
    5,
    TimeUnit.SECONDS
);

scheduler.shutdown();

```

The task becomes eligible after about five seconds; the exact start time can be later due to scheduling or system load. `shutdown()` allows already submitted delayed tasks to run.

### B. scheduleAtFixedRate(): regular cadence

```java
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(1);

scheduler.scheduleAtFixedRate(
    () -> System.out.println("Checking system"),
    0, // initial delay
    3, // period
    TimeUnit.SECONDS
);

```

The first execution has no initial delay. Later executions follow a cadence based on scheduled start times. If a run takes longer than the period, the next run starts late; the same periodic task does not overlap with itself.

### C. scheduleWithFixedDelay(): wait after completion

```java
scheduler.scheduleWithFixedDelay(
    () -> System.out.println("Cleaning files"),
    0,
    3,
    TimeUnit.SECONDS
);

```

The next run is scheduled after the previous run finishes and the specified delay passes. If a task takes five seconds and the delay is three seconds, the next run begins about eight seconds after the previous run started.

### Fixed rate vs fixed delay

| Point | Fixed rate | Fixed delay |
| --- | --- | --- |
| Method | `scheduleAtFixedRate()` | `scheduleWithFixedDelay()` |
| Timing | Based on scheduled start times | Delay after completion |
| Slow task | Later runs can start late | Delay is added after each run |
| Overlapping same periodic task | No | No |
| Exception escapes the task | Later runs are suppressed | Later runs are suppressed |

If a periodic execution throws an exception that escapes from the task, later executions are suppressed. For a periodic task that must continue, handle expected exceptions inside the task and log them.

---

## 9. Shutting down correctly

| Method | Meaning |
| --- | --- |
| `shutdown()` | Stop accepting new tasks and allow previously submitted tasks to finish. |
| `shutdownNow()` | Attempt to interrupt running tasks and return tasks that have not started. |
| `awaitTermination()` | Wait for termination up to a time limit. |

`shutdown()` does not itself wait for completion. `shutdownNow()` is only an attempt to stop active tasks; tasks that ignore interruption may continue.

```java
executor.shutdown();

try {
    if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}

```

---

## 10. Complete example: processing orders

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        ExecutorService executor =
                Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 6; i++) {
            final int orderId = i;

            executor.submit(() -> {
                System.out.println(
                    "Processing order " + orderId +
                    " on " +
                    Thread.currentThread().getName()
                );
            });
        }

        executor.shutdown();
    }
}

```

I submit six tasks to a pool of three workers. At most three tasks execute concurrently. Other tasks wait until workers are free. The output order may vary. `shutdown()` allows submitted tasks to finish.

---

## 11. Quick comparison table

| Concept | Remember it as |
| --- | --- |
| `Executor` | Basic task execution abstraction. |
| `ExecutorService` | Executor + submit/results + lifecycle management. |
| `ThreadPoolExecutor` | Configurable pool of worker threads and queue. |
| `Executors` | Utility factory class for creating common executor types. |
| `ScheduledExecutorService` | Schedules delayed and repeated work. |
| `Runnable` | Task without a result through its interface. |
| `Callable` | Task that returns a value and may throw checked exceptions. |
| `Future` | Handle for checking, cancelling, or retrieving task outcome. |

---

## 12. Common mistakes

* A task is not the same thing as a thread.
* `execute()` does not guarantee creation of a new thread.
* `submit()` returns a `Future`; it does not necessarily wait for the result.
* `Future.get()` can block and can throw exceptions.
* `shutdown()` initiates shutdown but does not wait for termination.
* `shutdownNow()` cannot guarantee that every running task stops.
* A fixed thread pool commonly uses an unbounded queue, which can grow very large.
* Fixed rate is based on scheduled start times; fixed delay is measured after completion.
* An exception escaping from a periodic task suppresses its future executions.
* The exact output order of tasks on multiple workers is not guaranteed.

---

## 13. Final summary in my own words

I use the Executor Framework when I want Java to manage task execution instead of manually managing every thread. Executor gives me `execute()`. `ExecutorService` adds `submit()`, `Future` support, and shutdown methods. `Runnable` is for work that does not return a value through its interface, while `Callable` can return a value. `ThreadPoolExecutor` manages worker threads, a queue, pool sizes, and rejection behavior. `Executors` provides factory methods such as `newFixedThreadPool()`, `newSingleThreadExecutor()`, and `newScheduledThreadPool()`. `ScheduledExecutorService` is for delayed or repeated tasks. At the end, I should shut down executors correctly.

---

## 14. Practice questions

1. What is the difference between a task and a thread?
2. Why is a thread pool useful?
3. What is the difference between `execute()` and `submit()`?
4. How are `Runnable` and `Callable` different?
5. What does `Future.get()` do?
6. What are `corePoolSize` and `maximumPoolSize`?
7. What happens when a bounded task queue is full?
8. Compare `newFixedThreadPool()` and `newSingleThreadExecutor()`.
9. What is the difference between `scheduleAtFixedRate()` and `scheduleWithFixedDelay()`?
10. What is the difference between `shutdown()` and `shutdownNow()`?

---


import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {
        
        // 1. Manual Thread Creation
        Thread manualThread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("1. Manual thread running on " + Thread.currentThread().getName());
            }
        });
        manualThread.start();

        // 2. Custom Executor Interface Implementation
        Executor customExecutor = new Executor() {
            @Override
            public void execute(Runnable command) {
                new Thread(command).start();
            }
        };
        customExecutor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("2. Custom Executor running on " + Thread.currentThread().getName());
            }
        });

        // 3. Executors Factory Methods (Fixed Thread Pool & Single Thread Executor)
        ExecutorService fixedPool = Executors.newFixedThreadPool(2);
        ExecutorService singlePool = Executors.newSingleThreadExecutor();

        fixedPool.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("3. Fixed thread pool task running on " + Thread.currentThread().getName());
            }
        });

        singlePool.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("3. Single thread executor task running on " + Thread.currentThread().getName());
            }
        });

        // 4. ThreadPoolExecutor with custom configuration and CallerRunsPolicy rejection handler
        ThreadPoolExecutor detailedThreadPool = new ThreadPoolExecutor(
            1,                                    // corePoolSize
            1,                                    // maximumPoolSize
            10L,                                  // keepAliveTime
            TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(1),          // workQueue
            new ThreadPoolExecutor.CallerRunsPolicy() // Rejection Policy
        );

        detailedThreadPool.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println("4. ThreadPoolExecutor task running on " + Thread.currentThread().getName());
            }
        });

        // 5. Callable, Future, and Future Methods (isDone, get)
        ExecutorService callableService = Executors.newSingleThreadExecutor();
        Callable<Integer> callableTask = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                Thread.sleep(100);
                return 100 + 200;
            }
        };
        Future<Integer> future = callableService.submit(callableTask);
        try {
            System.out.println("5. Future isDone before completion: " + future.isDone());
            System.out.println("5. Callable result via Future.get(): " + future.get());
            System.out.println("5. Future isDone after completion: " + future.isDone());
        } catch (Exception e) {
            e.printStackTrace();
        }

        // 6. ScheduledExecutorService (schedule, scheduleAtFixedRate, scheduleWithFixedDelay)
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
        
        // Single delayed execution
        scheduler.schedule(new Runnable() {
            @Override
            public void run() {
                System.out.println("6. Scheduled delayed task executed.");
            }
        }, 1, TimeUnit.SECONDS);

        // Fixed rate execution
        scheduler.scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println("6. Scheduled fixed rate task executed.");
            }
        }, 0, 2, TimeUnit.SECONDS);

        // Fixed delay execution
        scheduler.scheduleWithFixedDelay(new Runnable() {
            @Override
            public void run() {
                System.out.println("6. Scheduled fixed delay task executed.");
            }
        }, 0, 2, TimeUnit.SECONDS);

        // Let periodic tasks run briefly for demonstration
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        // 7. Graceful Shutdown & awaitTermination
        fixedPool.shutdown();
        singlePool.shutdown();
        detailedThreadPool.shutdown();
        callableService.shutdown();
        
        // Shutting down scheduler using shutdownNow to stop recurring tasks cleanly
        scheduler.shutdownNow();

        try {
            if (!fixedPool.awaitTermination(3, TimeUnit.SECONDS)) {
                fixedPool.shutdownNow();
            }
        } catch (InterruptedException e) {
            fixedPool.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
