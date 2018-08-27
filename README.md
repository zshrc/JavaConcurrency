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
## Join


## Wait and Notify
