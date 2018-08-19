# JavaConcurrency

## Start new thread

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

## Volatile
