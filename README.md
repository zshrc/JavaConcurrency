
# JavaConcurrency

## Start new thread

Thread is a subclass implements Runnable interface.

<b>Option 1: implement Runnable interface - Preferred way </b>

```Java
Class Runner implements Runnable {
    @Override
    public void run() {
        // do something
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new Runner());
        t1.start();
    }
}
```

<b>Option 1.b: anonymous class</b>

```Java
Class Application {
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {// do something}
        };
        t1.start();
    }
}
```
<b>Option 2: extend Thread class </b>

```Java
Class Runner extends Thread {
    @Override
    public void run() {// do something}

    public static void main(String[] args) {
        Runner t1 = new Runner();
        t1.start();
    }
}
```

## Volatile keyword
If you write a program like this:
```Java
class Program implements Runnable {

    private boolean shouldRun = true;
    
    @Override
    public void run() {
        while (shouldRun) {
            Thread.sleep(5000);
        }
    }
    
    public void shutdown() {
        shouldRun = false;
    }
}

Class App {
    public static void main(String[] args) {
        Program p1 = new Program();
        Thread t1 = new Thread(p1);
        t1.start();
        new Scanner(System.in).nextLine();
        p1.shutdown();
    }
}
```

Program never shuts down, because variable shouldRun is written into CPU cache and even its value changed, it's never being checked again. To make it to be read from 'main memory' instead of cache, we should add 'volatile' variable:

```Java
private volatile boolean shouldRun = true;

```


## Synchronize

<b> Synchronized method</b>
```Java
public synchronized void increment(){ i++;}
```

<b> Intrinsic lock </b>
Every object has an intrinsic lock associated with it. Each thread that needs to exclusively and consistently access to an object's field has to acquire this lock before accessing them and release intrinsic lock when it's done with them.
```Java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```
<b> Static class</b>  
In the case that a static method has a synchronized keyword, the thread actually acquires the class object associated with this Class. Therefore access to a class's static fields is controlled by a lock that's distinct from the lock for any instance of a class.

<b> Reentrant Synchronization </b>  
A thread cannot acquire lock owned by another thread, but it can acquire the lock it already owns. Allowing a thread to acquire a lock more than once enables reentrant synchronization.

## ThreadPool

First, understand Executor and Executors:

<b>Executor</b> (interface) executes submitted input runnable: ```void execute(Runnable command)```  
<b>ExecutorService</b> (interface) extends Executor interface, provides method to manage termination and return a Future object for tracking one or more asynchronous tasks.  
<b>Executors</b> factory and utility methods for Executor: ```Executors.newFixedThreadPool(int nThreads)```  

Example:
```Java
class App {
    public static void main(Stringp[] args) {
    
        ExecutorService executor = Executors.newFixedThreadPool(2);
        for (int i = 0; i < 5; i++) {
            executor.submit(new Runnable() {});
        }
        executor.shutdown();
        System.out.println("All tasks submitted.");
        
        try {
            executor.awaitTermination(1, TimeUnit.DAYS);
        } catch (InterruptedException e) {
        }
        
        System.out.println("All tasks completed.");
    }
}

```
## CountdownLatch
An synchronization aid that allows one or more threads waits for a set of operations performed by other threads completes.

A CountdownLatch initializes with a <i>given</i> count; <i>await</i> block until the current count reaches zero after <i>countDown()</i>, after which all threads are released and any subsequent operations of await return immediately.

```Java
Class Process {
    private CountdownLatch latch;
    
    public Processor(CountdownLatch latch) {
        this.latch = latch;
    }
    public void process() throws InterruptedException {
        System.out.println("Submitted");
        Thread.sleep(3000);
        latch.countDown();
     }
 }

Class app {
    public static void main(String[] args) throw InterruptedException {
        CountdownLatch latch = new CountdownLatch(3);
        ExecutorService executor = Executors.newFixedThreadPool(3);
        for (int i = 0; i < 3; i++) {
            executor.submit(new Processor(latch);
        }
        latch.await();
        System.out.println("Finished");
    }
}

```

## Join
Allows one thread to wait for completion of another. If ```t``` is a thread object whose Thread is currently running, t.join causes current thread pause until t's thread terminates.

```
public class App {
    private BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            produce();
        }
        Thread t2 = new Thread(new Runnable() {
            consume();
        }
        
        t1.start();
        t2.start();
        
        t1.join();
        t2.join();
    }
    
    public static void produce() {
        while(true) {
            queue.put(new Random().nextInt(100));
        }
     }
     
     public static void consume() {
        while(true) {
            if (new Random().nextInt(10) == 0) {
                queue.take();
            }
        }
     }
}
```

## Wait and Notify
```wait()``` it causes one thread to wait until another thread invokes ```notify()``` or ```notifyAll()``` method for this object. It doesn't consume a lot of system resources unlike while loop checking for a flag. <b>You can only call it within ```synchronized``` lock block</b>. ```notify()``` can only be called within a synchronized block. ```notify``` does not relinquish the lock by itself so you want to relinguish the lock imediately after notify is called; otherwise this thread will keep holding the lock and the other thread won't be waken up until this thread is done.

```
public class Process {
    public void produce() {
        synchronized(this) {
            System.out.println("starting producer");
            wait();
            System.out.println("resumed");
        }
    }
        
    public void consume() {
        Scanner scanner = new Scanner(System.in);
        Thread.sleep(2000);
        synchronized(this) {
            System.out.println("waiting for press key");
            scanner.nextLine();
            System.out.println("key pressed");
            notify();
            // if we put a Thread.sleep(5000), the other thread cannot resume until 5 seconds later
        }
    }
}
```
## Wait and Notify in an producer/consumer example

```
class Procedure {
    LinkedList<Integer> queue = new LinkedList<>();
    Object lock = new Object();
    public void produce() {
        int value = 0;
        while(true) {
            synchronized(lock) {
                while (queue.size() == LIMIT) {
                    lock.wait();
                }
                queue.add(value++);
                lock.notify();
            }
        }
    }
    
    public void consume() {
        while(true) {
            synchronized(lock) {
                while(queue.size() == 0) {
                    lock.wait();
                }
                queue.removeFirst();
                lock.notify();
            }
        }
    }

}

```
Why there is a while loop check for queue size? Check answer here: https://stackoverflow.com/questions/1038007/why-should-wait-always-be-called-inside-a-loop

Basically it prevents spurious thread from calling notify() and causing array OutOfIndex error.

## Re-entrantLock
```ReentrantLock``` is similar to intrinsic lock, with extended capability. A ReentrantLock is unstructured, unlike synchronized constructs -- i.e. you don't need to use a block structure for locking and can even hold a lock across methods. An example:

```
public Condition newCondition()
```
Returns a Condition instance for use with this Lock instance.
The returned Condition instance supports the same usages as do the Object monitor methods (wait, notify, and notifyAll) when used with the built-in monitor lock.

1. If this <b>lock is not held when any of the Condition waiting or signalling methods are called</b>, then an IllegalMonitorStateException is thrown.
2. When the condition waiting methods are called the lock is released and, before they return, the lock is reacquired and the lock hold count restored to what it was when the method was called.
3. If a thread is interrupted while waiting then the wait will terminate, an InterruptedException will be thrown, and the thread's interrupted status will be cleared.
4. Waiting threads are signalled in FIFO order.
5. The ordering of lock reacquisition for threads returning from waiting methods is the same as for threads initially acquiring the lock, which is in the default case not specified, but for fair locks favors those threads that have been waiting the longest.

If condition.signalAll() is callled, all waiting threads become runnable.

```
private ReentrantLock lock;

public void foo() {
  ...
  lock.lock();
  ...
}

public void bar() {
  ...
  lock.unlock();
  ...
}
```

```
class Runner {
    private ReentrantLock lock = new ReentrantLock();
    private Condition cond = lock.newCondition();
    private count = 0;

    public imcrement() {
        for (int i = 0; i < 10000; i++) {
            count++;
       }
    }
    
    public void thread1() {
        lock.lock();
        System.out.println("Waiting...");
        cond.await(); // hand over the lock
        
        try {
            imcrement();
        } finally {
            lock.unlock();
        }
    }
    
    public void thread2() {
        Thread.sleep(1000);
        lock.lock();
        System.out.println("Press the enter key");
        new Scanner(System.in).nextLine();
        System.out.println("get the key");
        
        try {
            increment();
            cond.signal();  // equivalent to notify()
        } finally {
            lock.unlock();
        }
    }
}
```

## Semaphore
A counting semaphore. Conceptually, a Semaphore maintains a set of permits. Each acquire blocks until a permis is available, and then takes it; each release() adds a permit, potentially releases a blocking acquirer. 

Semaphore is often used to restrict the number threads can acquire to access some resources.

```
class Connection {
    private Connection instance = new Connection();
    private Semaphore semaphore = new Semaphore(10, true);
    private int connection = 10;
    
    private Connection() {}
    public static Connection getInstance() { return instance; }
    public void connect() {
        semaphore.acquire();
        try{
            synchronized(this) {
                connection++;
                System.out.println("Current connections: " + connections);
            }
            Thread.sleep(1000);
            synchronized(this) {
                connection--;
            }
        } finally {
            semaphore.release();
        }
    }
}

class App {
    public static void main(String[] args) {
        ExecutorService service = ExecutorService.newCachedThreadPool();
        for (int i = 0; i < 200; i++) {
            service.submit(new Runnable() {
                @Override
                public void run() {
                    Connection.getInstance().connect();
                }
            }
        }
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.DAYS);
    }
}
