# MyThreadPool

```java
import javafx.concurrent.Worker;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.PriorityBlockingQueue;

/**
 * @author sgj
 * @create 2022-09-11 16:32
 */
public class Demo26 {
    public static void main(String[] args) {
        MyThreadPool myThreadPool = new MyThreadPool(10);
        for (int i = 0; i < 10; i++) {
            myThreadPool.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println("hello threadPool");
                }
            });
        }
    }
}

class MyThreadPool{
    // 1. 创建一个任务,直接使用Runnable,不需要额外创建类了
    // 2. 使用一个数据结构来组织若干个任务
    private BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>();
    // 3. 描述一个线程,工作线程的功能就是从任务队列中任务并执行
    static class Worker extends Thread{
        // 当前线程池中有若干个worker线程, 这些线程内部都持有了上述的任务队列
        private BlockingQueue<Runnable> queue = null;

        public Worker(BlockingQueue<Runnable> queue){
            this.queue = queue;
        }

        @Override
        public void run() {
            // 就需要能够拿到上面的队列
            while(true){
                Runnable runnable = null;
                try {
                    // 循环的去获取任务队列中任务
                    // 如果队列为空,就直接阻塞,如果队列非空, 就获取到里面的内容
                    runnable = queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 获取之后,就执行任务
                runnable.run();
            }
        }
    }
    // 4. 创建一个数据结构来组织若干个线程
    private List<Thread> workers = new ArrayList<>();

    public MyThreadPool(int n){
        // 在构造方法中,创建出若干个线程, 放到上述的数组中
        for(int i = 0; i<= n - 1; i++){
            Worker worker = new Worker(queue);
            worker.start();
            workers.add(worker);
        }
    }

    // 5. 创建一个方法,允许程序员来放任务到线程池中
    public void submit(Runnable runnable){
        try {
            queue.put(runnable);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

