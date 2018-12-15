# java-util-concurrent

## CopyOnWriteArrayList
    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);

            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
    
    http://cs.oswego.edu/pipermail/concurrency-interest/2010-February/006886.html
    https://stackoverflow.com/questions/28772539/why-setarray-method-call-required-in-copyonwritearraylist
    
    
The reason this code calls setArray even when it's apparently unnecessary is so that it establishes an invariant for this method that it always performs a volatile write to this array. This establishes happens-before semantics with other threads that perform volatile reads from this array. This is important because the volatile write-read semantics apply to reads and writes other than those of the volatile field itself. Specifically, writes to other (non-volatile) fields before a volatile write happen-before reads from those other fields after a volatile read of the same volatile variable. 
    
    
    第17-18行的意义：
    这个hb意义何在？如下例子：a为非volatile的某基本类型变量,coal为CopyOnWriteArrayList对象
t1:
x:a = calValue;
y:coal.set….
———————
t2:
m:coal.get…
n:int tmp = a;

假设存在以上场景，如果能保证只会存在这样的轨迹：x,y,m,n.根据上述java API文档中的约定有hb(y,m),根据线程内的操作相关规定有hb(x,y),hb(m,n)，根据hb的传递性读写a变量就有hb(x,n)，所以t1对a的写操作对t2中a的读操作可见。如果CopyOnWriteArrayList的set的else里没有setArray(elements)的话，hb(y,m)就不再有了，上述的可见性也就无法保证。


## ArrayList

## HashMap

## ConcurrentHashMap


## InterruptedException
我们讨论运行线程t1的状态。中断t1的线程是t2
### 怎么产生的呢？
1、如果t1在运行阻塞方法（sleep/wait/poll），那么会产生InterruptedException。并且isInterrupted状态已经清空。
2、如果t1仅仅在跑计算，那么会设置isInterrupted会设置为true
### 如何处理呢？
1、可以简单的往上抛。
2、如果不能继续往上层抛异常的话，那么应该手动恢复状态Thread.currentThread().interrupt（）。其他其他线程t2、t3能通过t1.isInterrpted来判断，t1是成功跑完任务了呢，还是被啥坏家伙异常中断了。


