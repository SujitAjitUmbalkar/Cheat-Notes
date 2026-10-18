# 1. Introduction to Threads in Java

## 1.1 Program vs Process vs Thread

- **Program:** A set of instructions stored in a file.
- **Process:** A program that is currently executing. It has its own memory space and resources.
- **Thread:** A path of execution within a process.

### Example

When a Java application runs:

- Program → Java code.
- Process → Running Java application.
- Thread → Executes a particular task inside the application.

Threads share the process's memory but have their own stacks and execution states.

## 1.2 Single-threaded vs Multithreaded Processes

**Single-threaded process:**
- Executes tasks using one thread.
- Tasks generally execute sequentially.

**Multithreaded process:**
- Uses multiple threads within the same process.
- Threads can make progress concurrently.
- Can improve responsiveness and throughput.

**Concurrency:** Multiple tasks make progress during overlapping periods.

**Parallelism:** Multiple tasks execute at the same time on different CPU execution resources.

## 1.3 Thread States in Java

Java defines six thread states through `Thread.State`.

| State | Meaning |
|---|---|
| `NEW` | Thread created but not started. |
| `RUNNABLE` | Thread is executing or eligible to execute. |
| `BLOCKED` | Waiting to acquire a monitor lock. |
| `WAITING` | Waiting indefinitely for another action. |
| `TIMED_WAITING` | Waiting for a specified time. |
| `TERMINATED` | Thread execution has finished. |

### Important methods

## Important Thread Methods in Java

- **`start()`** — Starts a new thread of execution. The JVM schedules the new thread to execute its `run()` method independently of the thread that called `start()`. The calling thread can continue executing without waiting for the new thread to finish.

- **`run()`** — Contains the task's code that the thread executes. When `start()` is called, the new thread executes `run()`. However, if you call `run()` directly like a normal method, the code executes on the current thread, and no new thread is created.

- **`sleep(time)`** — Pauses the currently executing thread for the specified duration. For example, `Thread.sleep(5000)` pauses the current thread for approximately 5 seconds. During this time, the thread does not execute its task, allowing other threads to run. It throws `InterruptedException` if the thread is interrupted while sleeping.

- **`join()`** — Makes the thread calling `join()` wait until the target thread finishes its execution. For example, `workerThread.join()` called from the `main` thread blocks `main` until `workerThread` terminates. Use it when one thread must wait for another thread's result or completion; avoid it when the calling thread needs to continue without waiting.

- **`getState()`** — Returns the current state of a thread, such as `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, or `TERMINATED`. It is useful for understanding a thread's lifecycle while debugging. The returned state is only a snapshot and may change immediately afterward.

- **`currentThread()`** — Returns a reference to the thread that is currently executing the code. For example, `Thread.currentThread().getName()` returns the current thread's name, and `Thread.currentThread().getState()` returns its current Java state.

- **`getName()`** — Returns the name assigned to a thread. Thread names are useful for identifying which thread is executing a log statement, especially when debugging multithreaded applications.

- **`interrupt()`** — Sends an interruption request to a thread. If the thread is sleeping or waiting in an interruptible operation, it may receive an `InterruptedException`. Interruption does not forcibly terminate the thread; the thread's code should handle the request appropriately.

**Remember:** `RUNNABLE` includes both running and ready-to-run conditions.

## 1.4 Java Threads Before and After Java 21

Java 21 introduced virtual threads as a standard feature.

### Platform Threads

- Closely associated with operating-system threads.
- Consume more resources per thread.
- Suitable for CPU-intensive tasks and general-purpose work.
- The number that can be created efficiently is limited by system resources.

### Virtual Threads (Java 21+)

- Lightweight threads managed by the Java runtime.
- Allow a large number of concurrent tasks.
- Particularly useful for tasks waiting for I/O, such as database queries and HTTP requests.
- Do not automatically make CPU-intensive calculations faster.

### CPU and Threads

An 8-core CPU can execute up to 8 threads simultaneously if each core runs one thread and no additional hardware threads are available.

More threads can exist than the CPU can execute simultaneously. The operating system schedules them for CPU time.

**Remember:** Virtual threads improve scalability, especially for tasks that spend much of their time waiting.

## 1.5 Runnable Interface — Java 1

`Runnable` is a functional interface used to define a task that does not return a result.

Its single abstract method is:

```java
public void run();
```

### Example

```java
Runnable task = () -> {
    System.out.println("Task running");
};

Thread thread = new Thread(task);
thread.start();
```

- `Runnable` defines the task.
- `Thread` executes the task on a new thread when `start()` is called.
- `run()` returns no value.
- `Runnable.run()` cannot declare checked exceptions in its method signature.

## 1.6 Callable Interface — Java 5

`Callable<V>` is a functional interface introduced in Java 5 in the `java.util.concurrent` package.

Its single abstract method is:

```java
V call() throws Exception;
```

- `V` represents the return type.
- `call()` returns a result.
- It can declare checked exceptions.
- It is typically executed through an `ExecutorService`.

### Example

```java
import java.util.concurrent.*;

ExecutorService executor =
        Executors.newSingleThreadExecutor();

try {
    Callable<Integer> task = () -> 10 + 20;

    Future<Integer> future = executor.submit(task);

    System.out.println(future.get()); // 30
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
} catch (ExecutionException e) {
    e.printStackTrace();
} finally {
    executor.shutdown();
}
```

### Understanding Future

`Future<V>` represents the result of an asynchronous computation.

- `submit()` submits the task to the executor.
- `future.get()` waits for completion if necessary and retrieves the result.
- `shutdown()` stops accepting new tasks and allows submitted tasks to finish.

## 1.7 Runnable vs Callable

| Feature | Runnable | Callable |
|---|---|---|
| Method | `run()` | `call()` |
| Return value | No | Yes, type `V` |
| Checked exceptions | Cannot declare them | Can throw `Exception` |
| Typical execution | `Thread` or `ExecutorService` | `ExecutorService` |
| Result through `Future` | Possible when submitted to an executor, but no task result | Yes, via `Future<V>` |

### When to use which?

- Use `Runnable` when a task performs an action without returning a result.
- Use `Callable<V>` when a task must return a result or declare checked exceptions.

## Quick Revision

1. A program is code; a process is a running program; a thread is an execution path inside a process.
2. A process can contain multiple threads.
3. Java has six official thread states.
4. `start()` starts a new thread; directly calling `run()` does not.
5. Platform threads are associated with OS threads; virtual threads are lightweight and managed by the Java runtime.
6. `Runnable` uses `run()` and has no return value.
7. `Callable<V>` uses `call()` and returns a value of type `V`.
8. `Future<V>` allows you to retrieve a task's result using `get()`.
9. `ExecutorService` manages task execution and can be shut down using `shutdown()`.
