# MyBlockingqueue

```Java
// 对于put来说,阻塞条件: 队列为满
// 对于take来说,阻塞条件: 队列为空 

// 当前代码中,put和take两种操作不会wait(等待条件相反)

// notify只能随机唤醒一个等待的线程,不能做到精准
// 要想精准,就必须使用不同的锁对象
// 想唤醒t1,就t1.notify 让t1进行o1.wait
// 想唤醒t2,就o2.notify,让t2进行o2.wait
class MyBlockingQueue{
    // [head tail)表示队列中的元素范围
    // tail表示所要放的元素的位置
    // 入队列,tail++
    // 出队列,head++
    private int[] data = new int[1000];
    private int size = 0;
    private int head = 0;
    private int tail = 0;
    public Object locker = new Object();

    // 入队列
    public void put(int value){
        synchronized (locker){
            if(size == data.length){
                // 队列满了
                //return;
                try {
                    locker.wait();  // 针对那个对象加锁, 就使用哪个对象wait,如果针对this加锁,那么就是this.wait
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            // 把新的元素放到tail位置上
            data[tail] = value;
            tail++;

            // 如果tail到达末尾,将tail置为0
            // 第一种方式
            if(tail >= data.length) tail = 0;
            // 第二种方式(不推荐)
            // 理由如下
            // 1. 非常不直观, 可读性差
            // 2. 对于计算机;来说,开销要更大,
            //    计算机除法的速度是不如比较操
            //    作的
            // tail %= tail % data.length;

            size++;
            locker.notify();
        }
    }

    // 出队列
    public  Integer take() {
        synchronized (locker) {
            if (size == 0) {
                //return null;
                try {
                    locker.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            int res = data[head];
            head++;
            if (head >= data.length) head = 0;
            size--;
            // take 成功之后,就唤醒put中的等待
            locker.notify();
            return res;
        }
    }
}
```

