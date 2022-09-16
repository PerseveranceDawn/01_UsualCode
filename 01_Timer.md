# 01_定时器

```Java
// 1. 描述一个任务:runnable + time
// 2. 使用优先级队列来组织若干个任务 PriorityBlockingQueue
// 3. 实现shedule方法来注册任务到队列中
// 4. 创建一个扫描线程,这个扫描线程不停地获取队首元素,并且判定时间是否到达
// 另外注意,让MyTask类能够支持比较,以及注意解决这里的忙等问题
package lesson03;

import java.util.Timer;
import java.util.TimerTask;

/**
 * @author sgj
 * @create 2022-09-11 8:34
 */
public class demo23 {
    public static void main(String[] args) {
        // Timer内部是有专门的线程,来负责执行注册的任务的
        Timer timer = new Timer();
        timer.schedule(new TimerTask(){
            @Override
            public void run() {
                System.out.println("hello timer");
            }
        }, 3000);
        System.out.println("main");
    }
}

// Timer内部需要什么东西
// 1. 管理很多任务
// 2. 执行时间到了的任务

// 管理任务
// 1) 描述任务
//    创建一个专门的类来表示一个定时器中的任务(TimerTask)

// 2) 组织任务(使用一定的数据结构把一些任务给放到一起)
//    通过一定的数据结构来组织

// 3) 执行时间到了任务
//    需要先执行时间最靠前的任务
//    就需要有一个线程,不停的区间当前优先队列的队首元素,
//    看看当前最靠前的这个任务是不是时间到了

package lesson03;

import java.util.PriorityQueue;
import java.util.concurrent.PriorityBlockingQueue;

/**
 * @author sgj
 * @create 2022-09-11 9:18
 */
public class demo24 {
    public static void main(String[] args) {
        MyTimer myTimer = new MyTimer();
        myTimer.schedule(new Runnable() {
            @Override
            public void run() {
                System.out.println("hello timer");
            }
        }, 3000);

        System.out.println("main");
    }
}

// 创建一个类,表示一个任务
class MyTask implements Comparable<MyTask>{
    // 具体任务要干啥
    private Runnable runnable;

    // 任务具体啥时候干,保存任务要执行的时间戳
    private long time;

    // after 是一个时间间隔,不是绝对的时间戳的值
    public MyTask(Runnable runnable, long after){
        this.runnable = runnable;
        this.time = System.currentTimeMillis() + after;
    }

    public void run(){
        runnable.run();
    }

    public long getTime(){
        return time;
    }

    @Override
    public int compareTo(MyTask o) {
        // 升序排
        return (int)(this.time - o.time);
    }
}

class MyTimer{
    // 定时器内部要能够存放多个任务
    // 此处的队列要考虑线程安全问题,可能在多个线程里面进行注册任务
    // 同时还有一个专门的线程来获取任务执行,此处的队列需要注意线程
    // 安全问题
    // PriorityBlockingQueue是带有优先级和阻塞效果(线程安全)的队列
    private PriorityBlockingQueue<MyTask> queue = new PriorityBlockingQueue<>();

    public void schedule(Runnable runnable, long delay){
        MyTask task = new MyTask(runnable, delay);
        queue.put(task);
        // 每次任务插入成功之后,都唤醒一下扫描线程,让线程重新检查一下队首的任务看是否时间到要执行
        synchronized (locker){
            locker.notify();
        }
    }

    private Object locker = new Object();

    public MyTimer(){
        Thread t = new Thread(() ->{
            while(true){
                try {
                    // 先取出队首元素
                    MyTask task = queue.take();  // 如果队列中任务是空着的,就还好,这个线程就会在这里阻塞
                                                 // 就怕队列中的任务不空,并且对首任务的时间还没到,就会一
                                                 // 直取,然后再放回队列,上述操作叫做"忙等",这种操作是非常浪费资源的
                    // 再比较一下当前这个任务的时间到了吗
                    long curTime = System.currentTimeMillis();
                    if(curTime < task.getTime()){
                        // 时间没到, 要把任务在加回到队列中
                        queue.put(task);
                        // 指定一个等待时间
                        synchronized (locker){
                            // 这个wait方式,是等待一定时间,自动被唤醒,和join(time)类似
                            // 这里不用sleep,是因为当前线程不会交出CPU的执行权
                            locker.wait(task.getTime() - curTime);
                        }

                    }else{
                        // 时间到了, 执行任务
                        task.run();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t.start();
    }
}
```

