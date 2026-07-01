---
id: topic-note-concurrency-part3
aliases: []
tags: []
---

# :material-pencil: Topic Note Part 3: ExecutorService, Thread Pools & ForkJoin

> **Course:** Mastering Java Concurrency and Multithreading — Tim Buchalka (Udemy)
> **Section:** 21 — Mastering Java Concurrency and Multithreading
> **Lectures:** 15–20
> **Status:** :material-check-circle: Complete

---

## :material-target: Learning Objectives

By the end of this part, you should be able to:

- [x] Explain what `ExecutorService` is and why it replaces manual thread creation
- [x] Use `execute(Runnable)` vs `submit(Callable)` and explain the difference
- [x] Create and manage `SingleThreadExecutor`, `FixedThreadPool`, and `CachedThreadPool`
- [x] Shut down an executor gracefully with `shutdown()` + `awaitTermination()`
- [x] Use `Callable<T>` and `Future<T>` to retrieve results from concurrent tasks
- [x] Use `invokeAll()` and `invokeAny()` for bulk task submission
- [x] Schedule tasks with `ScheduledExecutorService` (`schedule`, `scheduleAtFixedRate`, `scheduleWithFixedDelay`)
- [x] Explain the work-stealing algorithm in `WorkStealingPool` / `ForkJoinPool`
- [x] Write `RecursiveTask<T>` for divide-and-conquer parallel computation

---

## :material-pool: 1. `ExecutorService` — The Thread Pool Abstraction

### Why Not Raw Threads?

```mermaid
flowchart LR
    subgraph RAW["❌ Raw Threads"]
        R1["new Thread(task1).start()"] --> RC["Creates OS thread\n(expensive: ~1-2MB stack)"]
        RC --> RD["Destroyed after task\n(wasted creation cost)"]
    end
    subgraph POOL["✅ ExecutorService / Thread Pool"]
        P1["executor.submit(task1)"] --> QUEUE["Task Queue"]
        QUEUE --> T1["Worker Thread 1\n(reused!)"]
        QUEUE --> T2["Worker Thread 2\n(reused!)"]
        T1 --> IDLE["Idle → waits for next task"]
        T2 --> IDLE
    end

    style RAW fill:#dc5c59,color:#fff
    style POOL fill:#4caf7c,color:#fff
    style IDLE fill:#4a6fa5,color:#fff
```

**Thread pool advantages:**
- **Reuse** — worker threads are not destroyed after a task; they wait for the next task
- **Bounded resources** — limit the number of concurrent threads (no unbounded thread explosion)
- **Task queue** — excess tasks wait in a queue, not blocked callers
- **Lifecycle management** — clean shutdown, graceful termination

### The `ExecutorService` Interface

```java
public interface ExecutorService extends Executor {
    void execute(Runnable task);             // Fire-and-forget (no return value)
    <T> Future<T> submit(Callable<T> task); // Return Future<T> for result retrieval
    <T> Future<T> submit(Runnable task, T result); // Runnable with forced result
    Future<?> submit(Runnable task);        // Runnable wrapped in Future

    <T> List<Future<T>> invokeAll(Collection<Callable<T>> tasks) throws InterruptedException;
    <T> T invokeAny(Collection<Callable<T>> tasks) throws InterruptedException, ExecutionException;

    void shutdown();                         // Graceful — no new tasks, drain queue
    List<Runnable> shutdownNow();            // Aggressive — interrupt running threads
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    boolean isShutdown();
    boolean isTerminated();
}
```

---

## :material-server-network: 2. Factory Methods — `Executors` Class

### `newSingleThreadExecutor()`

```java
var executor = Executors.newSingleThreadExecutor(
    new ColorThreadFactory(ThreadColor.ANSI_BLUE)  // Custom thread factory
);

executor.execute(Main::countDown);   // Submit Runnable

executor.shutdown();
boolean done = executor.awaitTermination(500, TimeUnit.MILLISECONDS);
if (done) System.out.println("Finished!");
```

```mermaid
flowchart LR
    T1["Task 1"] --> SQ["Single Queue"]
    T2["Task 2"] --> SQ
    T3["Task 3"] --> SQ
    SQ --> W1["Single Worker Thread\n(serialized execution)"]

    style SQ fill:#4a6fa5,color:#fff
    style W1 fill:#4caf7c,color:#fff
```

**Use when:** Tasks must execute in submission order, one at a time (e.g., database write serialization).

### `newFixedThreadPool(n)`

```java
int poolSize = 3;
var executor = Executors.newFixedThreadPool(poolSize, new ColorThreadFactory());

for (int i = 0; i < 10; i++) {
    executor.execute(Main::countDown);  // 10 tasks, only 3 threads
}
executor.shutdown();
```

```mermaid
flowchart LR
    Q["Task Queue\n(10 tasks)"] --> W1["Worker 1\n(runs tasks 1, 4, 7, 10)"]
    Q --> W2["Worker 2\n(runs tasks 2, 5, 8)"]
    Q --> W3["Worker 3\n(runs tasks 3, 6, 9)"]

    style Q fill:#4a6fa5,color:#fff
    style W1 fill:#4caf7c,color:#fff
    style W2 fill:#4caf7c,color:#fff
    style W3 fill:#4caf7c,color:#fff
```

**Use when:** CPU-bound tasks where `n ≈ Runtime.getRuntime().availableProcessors()`.

### `newCachedThreadPool()`

```java
var executor = Executors.newCachedThreadPool();
// Creates new threads on demand; reuses idle threads (60s idle timeout)
// Good for short-lived async tasks; bad for long-running tasks (unbounded growth)
```

### `ThreadFactory` — Custom Thread Configuration

```java
class ColorThreadFactory implements ThreadFactory {
    private String threadName;
    private int colorValue = 1;

    public ColorThreadFactory(ThreadColor color) {
        this.threadName = color.name();
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r);
        thread.setName(threadName);  // Name based on color enum
        return thread;
    }
}

// Usage
var blueExecutor = Executors.newSingleThreadExecutor(
    new ColorThreadFactory(ThreadColor.ANSI_BLUE)
);
```

---

## :material-cash-check: 3. `Callable<T>` and `Future<T>`

### `Callable` vs `Runnable`

| | `Runnable` | `Callable<T>` |
|---|---|---|
| **Return** | `void` | `T` (any type) |
| **Exception** | Cannot declare checked exceptions | Can throw `Exception` |
| **Use with** | `execute()` or `submit()` | `submit()` only |
| **SAM** | `void run()` | `T call() throws Exception` |

```java
Callable<Integer> sumTask = () -> {
    int sum = 0;
    for (int i = 1; i <= 10; i++) sum += i;
    return sum;  // Returns a value!
};

Future<Integer> future = executor.submit(sumTask);
```

### `Future<T>` — Handle for Async Results

```java
Future<Integer> future = executor.submit(sumTask);

future.isDone();              // true if completed (normally, cancelled, or exception)
future.isCancelled();         // true if cancelled before completion
future.cancel(true);          // Cancel; true = interrupt if running

Integer result = future.get();                              // Block until done
Integer result2 = future.get(500, TimeUnit.MILLISECONDS);  // Block with timeout
```

!!! warning "`Future.get()` Can Throw"
    - `ExecutionException` — if `call()` threw an exception (wraps it)
    - `InterruptedException` — if the waiting thread was interrupted
    - `TimeoutException` — if `get(timeout, unit)` expired
    Always handle all three.

### `invokeAll()` — Submit All, Wait for All

```java
List<Callable<Integer>> tasks = List.of(
    () -> sum(1, 10, 1, "red"),
    () -> sum(10, 100, 10, "blue"),
    () -> sum(2, 20, 2, "green")
);

var multiExecutor = Executors.newCachedThreadPool();
try {
    List<Future<Integer>> results = multiExecutor.invokeAll(tasks);
    // Blocks until ALL tasks complete (or exception/interrupt)

    for (var result : results) {
        System.out.println(result.get(500, TimeUnit.MILLISECONDS));
    }
} catch (InterruptedException | TimeoutException | ExecutionException e) {
    throw new RuntimeException(e);
} finally {
    multiExecutor.shutdown();
}
```

```mermaid
sequenceDiagram
    participant MAIN as Main Thread
    participant E as ExecutorService
    participant T1 as Task 1
    participant T2 as Task 2
    participant T3 as Task 3

    MAIN->>E: invokeAll([t1, t2, t3])
    E->>T1: submit
    E->>T2: submit
    E->>T3: submit
    Note over MAIN: BLOCKED waiting for all
    T1-->>E: result1
    T2-->>E: result2
    T3-->>E: result3
    E-->>MAIN: List<Future> (all done)
    MAIN->>MAIN: future.get() for each
```

### `invokeAny()` — Submit All, Take First Winner

```java
// Returns the result of the FIRST task to complete successfully
// All other tasks are cancelled
Integer firstResult = multiExecutor.invokeAny(tasks);
System.out.println("First to finish: " + firstResult);
```

**Use case:** Redundant computation — submit the same job to multiple threads, take whoever finishes first (e.g., querying multiple data sources).

---

## :material-executor: 4. Shutdown — Graceful vs Immediate

```mermaid
flowchart TD
    RUNNING["ExecutorService\n(RUNNING)"] --> SD["shutdown()"]
    RUNNING --> SDN["shutdownNow()"]

    SD --> SHUTDOWN["SHUTDOWN state\n• No new tasks accepted\n• Queued tasks still run\n• Running tasks complete"]
    SHUTDOWN --> AT["awaitTermination(timeout)"]
    AT --> TERMINATED["TERMINATED\n(all tasks done)"]

    SDN --> INTERRUPT["Interrupt running tasks\nReturn unstarted tasks list"]
    INTERRUPT --> TERMINATED

    style RUNNING fill:#4caf7c,color:#fff
    style SHUTDOWN fill:#e8933a,color:#fff
    style TERMINATED fill:#3d59a1,color:#fff
    style INTERRUPT fill:#dc5c59,color:#fff
```

```java
executor.shutdown();   // Signal shutdown (non-blocking)
try {
    // Wait up to 10 seconds for all tasks to finish
    if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
        executor.shutdownNow();   // Force if timeout exceeded
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

---

## :material-clock-fast: 5. `ScheduledExecutorService` — Timed Tasks

### Factory

```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(4);
// OR single-threaded:
ScheduledExecutorService single = Executors.newSingleThreadScheduledExecutor();
```

### Three Scheduling Modes

```java
Runnable task = () -> System.out.println("Running at: " + ZonedDateTime.now());

// 1. Run ONCE after a delay
scheduler.schedule(task, 5, TimeUnit.SECONDS);

// 2. Fixed rate — start every N seconds from the first start
// (If task takes longer than period, next run starts immediately after previous ends)
ScheduledFuture<?> fixed = scheduler.scheduleAtFixedRate(
    task,
    2,    // initialDelay
    2,    // period (between START of each execution)
    TimeUnit.SECONDS
);

// 3. Fixed delay — wait N seconds AFTER previous execution ends
scheduler.scheduleWithFixedDelay(
    task,
    2,    // initialDelay
    3,    // delay (between END of last execution and START of next)
    TimeUnit.SECONDS
);
```

```mermaid
flowchart LR
    subgraph RATE["scheduleAtFixedRate(period=2s)"]
        SR1["Run 1\n(0-5s)"] --> SR2["Run 2\n(immediately!)"]
        SR2 --> SR3["Run 3\n(2s after Run 1 start)"]
    end
    subgraph DELAY["scheduleWithFixedDelay(delay=2s)"]
        SD1["Run 1\n(0-5s)"] --> WAIT["Wait 2s"] --> SD2["Run 2\n(7s)"]
    end

    style RATE fill:#4a6fa5,color:#fff
    style DELAY fill:#4caf7c,color:#fff
```

!!! info "Rate vs Delay"
    - `scheduleAtFixedRate`: Period is measured from **start of previous execution**. If a task runs longer than the period, the next run starts immediately (no accumulation of missed runs).
    - `scheduleWithFixedDelay`: Delay is measured from **end of previous execution**. Guarantees a rest period between runs regardless of task duration.

### Cancelling Scheduled Tasks

```java
ScheduledFuture<?> task = scheduler.scheduleAtFixedRate(job, 0, 2, TimeUnit.SECONDS);

// Check and cancel after 10 seconds
long start = System.currentTimeMillis();
while (!task.isDone()) {
    Thread.sleep(2000);
    if ((System.currentTimeMillis() - start) / 1000 > 10) {
        task.cancel(true);   // Cancel with interrupt
    }
}
scheduler.shutdown();
```

---

## :material-fork: 6. `WorkStealingPool` & `ForkJoinPool`

### Work-Stealing Algorithm

```mermaid
flowchart TD
    FJP["ForkJoinPool\n(parallelism = CPU cores)"]
    FJP --> W1["Worker 1\nDeque: [A, B, C]"]
    FJP --> W2["Worker 2\nDeque: [D, E]"]
    FJP --> W3["Worker 3\nDeque: [ ] IDLE"]

    W3 -->|"STEAL from tail of W1"| STOLEN["Executes C from W1's queue"]

    style FJP fill:#3d59a1,color:#fff
    style W1 fill:#4caf7c,color:#fff
    style W2 fill:#4caf7c,color:#fff
    style W3 fill:#e8933a,color:#fff
    style STOLEN fill:#7b68ae,color:#fff
```

**Work stealing:** Each worker has its own double-ended queue (deque). Workers process their own tasks from the front. Idle workers **steal tasks from the TAIL** of other workers' queues — minimizing contention (owner takes front, thieves take back).

### `newWorkStealingPool()`

```java
// Creates a ForkJoinPool with parallelism = available CPU cores
var executor = Executors.newWorkStealingPool();

// Equivalent to:
new ForkJoinPool(Runtime.getRuntime().availableProcessors());

System.out.println("CPUs: " + Runtime.getRuntime().availableProcessors());
System.out.println("Parallelism: " + ForkJoinPool.commonPool().getParallelism());
```

### `RecursiveTask<T>` — Divide and Conquer

```java
class RecursiveSumTask extends RecursiveTask<Long> {
    private final long[] numbers;
    private final int start, end, division;

    @Override
    protected Long compute() {
        // Base case: small enough to compute directly
        if ((end - start) <= (numbers.length / division)) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += numbers[i];
            return sum;
        }
        // Recursive case: split into subtasks
        int mid = (start + end) / 2;
        RecursiveSumTask left  = new RecursiveSumTask(numbers, start, mid, division);
        RecursiveSumTask right = new RecursiveSumTask(numbers, mid, end, division);

        left.fork();   // Async submit to pool
        right.fork();  // Async submit to pool

        return left.join() + right.join();  // Wait for results + combine
    }
}

// Usage
ForkJoinPool pool = ForkJoinPool.commonPool();
RecursiveTask<Long> task = new RecursiveSumTask(numbers, 0, numbers.length, 2);
long result = pool.invoke(task);   // submit + wait for root task
```

```mermaid
flowchart TD
    ROOT["compute(0..100000)"] --> LEFT["compute(0..50000)"]
    ROOT --> RIGHT["compute(50000..100000)"]
    LEFT --> LL["compute(0..25000)\n→ sum directly"]
    LEFT --> LR["compute(25000..50000)\n→ sum directly"]
    RIGHT --> RL["compute(50000..75000)\n→ sum directly"]
    RIGHT --> RR["compute(75000..100000)\n→ sum directly"]

    style ROOT fill:#3d59a1,color:#fff
    style LEFT fill:#4a6fa5,color:#fff
    style RIGHT fill:#4a6fa5,color:#fff
    style LL fill:#4caf7c,color:#fff
    style LR fill:#4caf7c,color:#fff
    style RL fill:#4caf7c,color:#fff
    style RR fill:#4caf7c,color:#fff
```

### `RecursiveAction` vs `RecursiveTask<T>`

| | `RecursiveTask<T>` | `RecursiveAction` |
|---|---|---|
| **Returns** | `T` (a computed value) | `void` |
| **Use for** | Divide-and-conquer computation | Parallel operations with side effects |
| **Example** | Sum an array in parallel | Sort an array in parallel |

---

## :material-compare: 7. Choosing the Right Executor

```mermaid
flowchart TD
    Q["What type of task?"]
    Q --> IO["I/O bound\n(DB, network, file)"] --> CACHED["newCachedThreadPool\nor\nnewFixedThreadPool(high-n)"]
    Q --> CPU["CPU bound\n(computation)"] --> FIXED["newFixedThreadPool(n)\nn = availableProcessors()"]
    Q --> ORDER["Sequential order required"] --> SINGLE["newSingleThreadExecutor"]
    Q --> SCHED["Periodic / scheduled"] --> SCHEX["newScheduledThreadPool"]
    Q --> DIVCON["Divide-and-conquer\nrecursive computation"] --> FJP2["ForkJoinPool /\nnewWorkStealingPool"]

    style CACHED fill:#4a6fa5,color:#fff
    style FIXED fill:#4caf7c,color:#fff
    style SINGLE fill:#7b68ae,color:#fff
    style SCHEX fill:#e8933a,color:#fff
    style FJP2 fill:#3d59a1,color:#fff
```

---

## :material-help-circle: Questions Explored

- [x] Why is creating a new thread for every task inefficient?
- [x] What's the difference between `execute()` and `submit()`?
- [x] What states can a `Future` be in?
- [x] What exceptions can `Future.get()` throw and why?
- [x] What's the difference between `invokeAll()` and `invokeAny()`?
- [x] What's the difference between `shutdown()` and `shutdownNow()`?
- [x] What's the difference between `scheduleAtFixedRate` and `scheduleWithFixedDelay`?
- [x] How does the work-stealing algorithm prevent idle threads?
- [x] What's the difference between `RecursiveTask` and `RecursiveAction`?
- [x] When should you use `ForkJoinPool` vs `FixedThreadPool`?

---

## :material-navigation: Related Notes

| Part | Topic | Link |
|:----:|-------|------|
| 1 | Threads, States & Memory Model | [← Part 1](topic-note.md) |
| 2 | Synchronization & Locks | [← Part 2](topic-note-part2.md) |
| 3 | ExecutorService & Thread Pools | **You are here** |
| 4 | Parallel Streams & Concurrent Collections | [Part 4 →](topic-note-part4.md) |
| 5 | Thread Hazards, Atomics & WatchService | [Part 5 →](topic-note-part5.md) |

---

*Last Updated: 2026-07-01*
