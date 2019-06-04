```java
public class Processor implements Runnable {
    private String strVal = null;
    private String name;
    private Thread thread;
    private boolean available = false;

    public Processor(int id) {
        this.name = "processor[" + id + "]";
    }


    synchronized public void assign(String strVal) {
        while (available) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println(e);
            }
        }

        this.strVal = strVal;
        this.available = true;

        notifyAll();
    }

    synchronized private String await() {
        while (!available) {
            try {
                wait();
            } catch (InterruptedException e) {
                System.out.println(e);
            }
        }

        available = false;
        String v = this.strVal;
        notifyAll();
        return v;
    }


    @Override
    public void run() {
        while (true) {
            String str = await();
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                System.out.print(e);
            }
            System.out.println("thread " + name + " get string: " + str);
        }
    }

    public void start() {
        Thread thread = new Thread(this, name);
        //thread.setDaemon(true);
        thread.start();
    }
}

```





```java
import java.util.Stack;

public class Main {
    public static void main(String[] argv) {
        System.out.println("hello world");
        Stack processors = new Stack();
        for (int i = 0; i < 1; i++) {
            Processor p = new Processor(i);
            p.start();
            processors.push(p);
        }

        Processor processor = (Processor) processors.pop();

        for (int i = 100; i < 105; i++) {
            processor.assign("strvalue:" + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.print(e);
            }
        }
    }
}
```

