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


## Join


## Wait and Notify
