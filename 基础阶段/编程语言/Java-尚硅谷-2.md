# 第8章 多线程

## 8.1 程序、进程、线程理解

1、程序(programm)

概念：是为完成特定任务、用某种语言编写的一组指令的集合。即指一段静态的代码。

2、进程(process)

概念：程序的一次执行过程，或是正在运行的一个程序。

说明：进程作为资源分配的单位，系统在运行时会为每个进程分配不同的内存区域

3、线程(thread)

概念：进程可进一步细化为线程，是一个程序内部的一条执行路径。

说明：线程作为调度和执行的单位，每个线程拥独立的运行栈和程序计数器(pc)，线程切换的开销小。

 ![image-20210221213001794](java-尚.assets/image-20210221213001794.png)

 补充：

 内存结构：

  ![image-20210221213020239](java-尚.assets/image-20210221213020239.png)

 进程可以细化为多个线程。

 每个线程，拥有自己独立的：栈、程序计数器

 多个线程，共享同一个进程中的结构：方法区、堆。

## 8.2 并行和并发

1、单核CPU与多核CPU的理解

单核CPU，其实是一种假的多线程，因为在一个时间单元内，也只能执行一个线程的任务。例如：虽然有多车道，但是收费站只有一个工作人员在收费，只有收了费才能通过，那么CPU就好比收费人员。如果某个人不想交钱，那么收费人员可以把他“挂起”（晾着他，等他想通了，准备好了钱，再去收费。）但是因为CPU时间单元特别短，因此感觉不出来。如果是多核的话，才能更好的发挥多线程的效率。（现在的服务器都是多核的）一个Java应用程序java.exe，其实至少三个线程：main()主线程，gc()垃圾回收线程，异常处理线程。当然如果发生异常，会影响主线程。

2、并行与并发的理解

并行：多个CPU同时执行多个任务。比如：多个人同时做不同的事。

 并发：一个CPU(采用时间片)同时执行多个任务。比如：秒杀、多个人做同一件事



## 8.3 创建多线程的两种方式

 方式一：继承Thread类的方式：
 ```sh
  1. 创建一个继承于Thread类的子类
  2. 重写Thread类的run() -- 将此线程执行的操作声明在run()中
  3. 创建Thread类的子类的对象
  4. 通过此对象调用start()：①启动当前线程 ② 调用当前线程的run()
 ```

 说明两个问题：

 问题一：我们启动一个线程，必须调用start()，不能调用run()的方式启动线程。

 问题二：如果再启动一个线程，必须重新创建一个Thread子类的对象，调用此对象的start().

 方式二：实现Runnable接口的方式：
 ```sh
  1. 创建一个实现了Runnable接口的类
  2. 实现类去实现Runnable中的抽象方法：run()
  3. 创建实现类的对象
  4. 将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
  5. 通过Thread类的对象调用start()
 ```

 两种方式的对比：
 ```sh
  开发中：优先选择：实现Runnable接口的方式
  原因：
  1. 实现的方式没类的单继承性的局限性
  2. 实现的方式更适合来处理多个线程共享数据的情况。
      
  联系：public class Thread implements Runnable
  相同点：两种方式都需要重写run(),将线程要执行的逻辑声明在run()中。
       目前两种方式，要想启动线程，都是调用的Thread类中的start()。
 ```

 

## 8.4 Thread常用的方法

 Thread类中的常用的方法:

 ```sh
  1. start():启动当前线程；调用当前线程的run()
  2. run(): 通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中
  3. currentThread():静态方法，返回执行当前代码的线程
  4. getName():获取当前线程的名字
  5. setName():设置当前线程的名字
  6. yield():释放当前cpu的执行权
  7. join():在线程a中调用线程b的join(),此时线程a就进入阻塞状态，直到线程b完全执行完以后，线程a才结束阻塞状态。
  8. stop():已过时。当执行此方法时，强制结束当前线程。
  9. sleep(long millitime):让当前线程“睡眠”指定的millitime毫秒。在指定的millitime毫秒时间内，当前线程是阻塞状态。
  10. isAlive():判断当前线程是否存活
 
  线程的优先级：
  1.
  MAX_PRIORITY：10
  MIN _PRIORITY：1
  NORM_PRIORITY：5  --默认优先级
  2.如何获取和设置当前线程的优先级：
  getPriority():获取线程的优先级
  setPriority(int p):设置线程的优先级
   
 说明：高优先级的线程要抢占低优先级线程cpu的执行权。但是只是从概率上讲，高优先级的线程高概率的情况下被执行。并不意味着只当高优先级的线程执行完以后，低优先级的线程才执行。
 
 
 线程通信：wait() / notify() / notifyAll() :此三个方法定义在Object类中的。
 ```

 补充：线程的分类

 一种是守护线程，一种是用户线程。



## 8.5 Thread生命周期

 图示：

 ![image-20210221213614753](java-尚.assets/image-20210221213614753.png)

 说明：

 1.生命周期关注两个概念：状态、相应的方法

 2.关注：状态a--状态b:哪些方法执行了（回调方法）
         某个方法主动调用：状态a--状态b

 3.阻塞：临时状态，不可以作为最终状态

 死亡：最终状态。

## 8.6 线程的同步机制

 1.背景

 例子：创建个窗口卖票，总票数为100张.使用实现Runnable接口的方式

 ```sh
  1.问题：卖票过程中，出现了重票、错票 --出现了线程的安全问题
  2.问题出现的原因：当某个线程操作车票的过程中，尚未操作完成时，其他线程参与进来，也操作车票。
  3.如何解决：当一个线程a在操作ticket的时候，其他线程不能参与进来。直到线程a操作完ticket时，其他线程才可以开始操作ticket。这种情况即使线程a出现了阻塞，也不能被改变。
 ```

 2.Java解决方案：同步机制

 在Java中，我们通过同步机制，来解决线程的安全问题。

 ```sh
 方式一：同步代码块
 
  synchronized(同步监视器){
  //需要被同步的代码
  }
 
  说明：1.操作共享数据的代码，即为需要被同步的代码。  --不能包含代码多了，也不能包含代码少了。
  2.共享数据：多个线程共同操作的变量。比如：ticket就是共享数据。
  3.同步监视器，俗称：锁。任何一个类的对象，都可以充当锁。
  要求：多个线程必须要共用同一把锁。
   
  补充：在实现Runnable接口创建多线程的方式中，我们可以考虑使用this充当同步监视器。
       在继承Thread类创建多线程的方式中，慎用this充当同步监视器，考虑使用当前类充当同步监视器。
 
  方式二：同步方法
  如果操作共享数据的代码完整的声明在一个方法中，我们不妨将此方法声明同步的。
  关于同步方法的总结：
  1. 同步方法仍然涉及到同步监视器，只是不需要我们显式的声明。
  2. 非静态的同步方法，同步监视器是：this
  静态的同步方法，同步监视器是：当前类本身
 
   方式三：Lock锁  --- JDK5.0新增
 
 
  1. 面试题：synchronized 与 Lock的异同？
  相同：二者都可以解决线程安全问题
  不同：synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器
 
  Lock需要手动的启动同步（lock()，同时结束同步也需要手动的实现（unlock()）使用的优先顺序：
  Lock --- 同步代码块（已经进入了方法体，分配了相应资源 ) --- 同步方法（在方法体之外)
 ```

 3.利弊

 ```sh
 同步的方式，解决了线程的安全问题。---好处
 操作同步代码时，只能一个线程参与，其他线程等待。相当于是一个单线程的过程，效率低。
 ```

 4.面试题

 ```sh
 面试题：Java是如何解决线程安全问题的，有几种方式？并对比几种方式的不同
 面试题：synchronized和Lock方式解决线程安全问题的对比
 ```

 

### 8.6.1 线程安全单利模式（懒汉式）

 使用同步机制将单例模式中的懒汉式改写为线程安全的。

 ```java
 class Bank{
     private Bank(){}
     private static Bank instance = null;
     public static Bank getInstance(){
     //方式一：效率稍差
     // synchronized (Bank.class) {
     //if(instance == null){
     //
     // instance = new Bank();
     //  }
     //  return instance;
 // }
     //方式二：效率更高
    
         if(instance == null){
             synchronized (Bank.class) {
                 if(instance == null){
                     instance = new Bank();
                 }
             }
         }
         return instance;
     }  
 }
 ```
 面试题：写一个线程安全的单例模式。

 ```sh
 饿汉式。
 懒汉式：上面提供的。
 ```

 

### 8.6.2 死锁问题

 1.死锁的理解：

 不同的线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁

 2.说明：
 ```sh
  1出现死锁后，不会出现异常，不会出现提示，只是所的线程都处于阻塞状态，无法继续
  2我们使用同步时，要避免出现死锁。
 ```

 3.举例：


 ```java
 public static void main(String[] args) {
     StringBuffer s1 = new StringBuffer();
     StringBuffer s2 = new StringBuffer();
     new Thread(){
         @Override
         public void run() {
             synchronized (s1){
                 s1.append("a");
                 s2.append("1");
                 try {
                     Thread.sleep(100);
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
                 synchronized (s2){
                     s1.append("b");
                     s2.append("2");
                     System.out.println(s1);
                     System.out.println(s2);
                 }
             }
         }
     }.start();
     new Thread(new Runnable() {
         @Override
         public void run() {
             synchronized (s2){
             s1.append("c");
             s2.append("3");
             try {
                 Thread.sleep(100);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
 
             synchronized (s1){
                 s1.append("d");
                 s2.append("4");
                 System.out.println(s1);
                 System.out.println(s2);
             }
             }
         }
     }).start();
 }
 ```


## 8.7 线程通信

 1.线程通信涉及到的三个方法：
 ```sh
  wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。
  notify():一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。
  notifyAll():一旦执行此方法，就会唤醒所有被wait的线程。
 ```

 2.说明：

 ```sh
  1.wait()，notify()，notifyAll()三个方法必须使用在同步代码块或同步方法中。
  2.wait()，notify()，notifyAll()三个方法的调用者必须是同步代码块或同步方法中的同步监视器。
  否则，会出现IllegalMonitorStateException异常
  3.wait()，notify()，notifyAll()三个方法是定义在java.lang.Object类中。
 ```

 3.面试题：

 面试题：sleep() 和 wait()的异同？

 ```sh
  1.相同点：一旦执行方法，都可以使得当前的线程进入阻塞状态。
  2.不同点：1）两个方法声明的位置不同：Thread类中声明sleep() , Object类中声明wait()
  2）调用的要求不同：sleep()可以在任何需要的场景下调用。 wait()必须使用在同步代码块或同步方法中
  3）关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，wait()会释放锁。
 ```

 

 4.小结释放锁的操作：

 ![image-20210221214923342](java-尚.assets/image-20210221214923342.png)

 小结不会释放锁的操作：

 ![image-20210221214930604](java-尚.assets/image-20210221214930604.png)

## 8.8 JDK5.0新增创建线程的方式

 新增方式一：实现Callable接口。 --- JDK 5.0新增

 ```java
 //1.创建一个实现Callable的实现类
 class NumThread implements Callable{
     //2.实现call方法，将此线程需要执行的操作声明在call()中
     @Override
     public Object call() throws Exception {
         int sum = 0;
         for (int i = 1; i <= 100; i++) {
             if(i % 2 == 0){
                 System.out.println(i);
                 sum += i;
             }
         }
         return sum;
     }
 }
 
 
 public class ThreadNew {
     public static void main(String[] args) {
         //3.创建Callable接口实现类的对象
         NumThread numThread = new NumThread();
         //4.将此Callable接口实现类的对象作为传递到FutureTask构造器中，创建FutureTask的对象
         FutureTask futureTask = new FutureTask(numThread);
         //5.将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()
         new Thread(futureTask).start();
         try {
             //6.获取Callable中call方法的返回值
             //get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。
             Object sum = futureTask.get();
             System.out.println("总和为：" + sum);
         } catch (InterruptedException e) {
             e.printStackTrace();
         } catch (ExecutionException e) {
             e.printStackTrace();
         }
     }
 }
 ```

 ```sh
 说明：
  如何理解实现Callable接口的方式创建多线程比实现Runnable接口创建多线程方式强大？
  1. call()可以返回值的。
  2. call()可以抛出异常，被外面的操作捕获，获取异常的信息
  3. Callable是支持泛型的
 ```

 

 新增方式二：使用线程池

 ```java
 class NumberThread implements Runnable{
     @Override
     public void run() {
         for(int i = 0;i <= 100;i++){
             if(i % 2 == 0){
                 System.out.println(Thread.currentThread().getName() + ": " + i);
             }
         }
     }
 }
 
 class NumberThread1 implements Runnable{
     @Override
     public void run() {
 
         for(int i = 0;i <= 100;i++){
             if(i % 2 != 0){
                 System.out.println(Thread.currentThread().getName() + ": " + i);
             }
         }
     }
 }
 
 public class ThreadPool {public static void main(String[] args) {
     //1. 提供指定线程数量的线程池
     ExecutorService service = Executors.newFixedThreadPool(10);
     ThreadPoolExecutor service1 = (ThreadPoolExecutor) service;
     //设置线程池的属性
     //System.out.println(service.getClass());
     //service1.setCorePoolSize(15);
     //service1.setKeepAliveTime();
     //2.执行指定的线程的操作。需要提供实现Runnable接口或Callable接口实现类的对象
     service.execute(new NumberThread());//适合适用于Runnable
     service.execute(new NumberThread1());//适合适用于Runnable
     //service.submit(Callable callable);//适合使用于Callable
     //3.关闭连接池
     service.shutdown();
     }
 }
 ```
说明：

 ```sh
  好处：
  1.提高响应速度（减少了创建新线程的时间）
  2.降低资源消耗（重复利用线程池中线程，不需要每次都创建）
  3.便于线程管理
  corePoolSize：核心池的大小
  maximumPoolSize：最大线程数
  keepAliveTime：线程没任务时最多保持多长时间后会终止
 ```

 面试题：Java中多线程的创建有几种方式？四种。



# 第9章 java常用类

## 9.1 String类

 java.lang.String类的使用

 1.概述

 ```sh
 String:字符串，使用一对""引起来表示。
 1.String声明为final的，不可被继承
 2.String实现了Serializable接口：表示字符串是支持序列化的。
         实现了Comparable接口：表示String可以比较大小
 3.String内部定义了final char[] value用于存储字符串数据
 4.通过字面量的方式（区别于new给一个字符串赋值，此时的字符串值声明在字符串常量池中)。
 5.字符串常量池中是不会存储相同内容(使用String类的equals()比较，返回true)的字符串的。
 ```

 2.String的不可变性

 2.1 说明

 ```sh
 1.当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
 2.当对现的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
 3.当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
 ```

 2.2 代码举例

 ```java
 String s1 = "abc";//字面量的定义方式
 String s2 = "abc";
 s1 = "hello";
 
 System.out.println(s1 == s2);//比较s1和s2的地址值
 
 System.out.println(s1);//hello
 System.out.println(s2);//abc
 
 System.out.println("");
 
 String s3 = "abc";
 s3 += "def";
 System.out.println(s3);//abcdef
 System.out.println(s2);
 
 System.out.println("");
 
 String s4 = "abc";
 String s5 = s4.replace('a', 'm');
 System.out.println(s4);//abc
 System.out.println(s5);//mbc
 ```

 2.3 图示

 ![image-20210221222321836](java-尚.assets/image-20210221222321836.png)

 3.String实例化的不同方式

 3.1 方式说明

 方式一：通过字面量定义的方式

 方式二：通过new + 构造器的方式

 3.2 代码举例

 ```java
 //通过字面量定义的方式：此时的s1和s2的数据javaEE声明在方法区中的字符串常量池中。
 String s1 = "javaEE";
 String s2 = "javaEE";
 //通过new + 构造器的方式:此时的s3和s4保存的地址值，是数据在堆空间中开辟空间以后对应的地址值。
 String s3 = new String("javaEE");
 String s4 = new String("javaEE");
 
 System.out.println(s1 == s2);//true
 System.out.println(s1 == s3);//false
 System.out.println(s1 == s4);//false
 System.out.println(s3 == s4);//false
 ```

 3.3 面试题

 ```sh
 String s = new String("abc");方式创建对象，在内存中创建了几个对象？
 两个:一个是堆空间中new结构，另一个是char[]对应的常量池中的数据："abc"
 ```

 3.4 图示

 ![image-20210221222419749](java-尚.assets/image-20210221222419749.png)

 4.字符串拼接方式赋值的对比

 4.1 说明

 ```sh
 1.常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。
 2.只要其中一个是变量，结果就在堆中。
 3.如果拼接的结果调用intern()方法，返回值就在常量池中
 ```

 4.2 代码举例

 ```java
 String s1 = "javaEE";
 String s2 = "hadoop";
 
 String s3 = "javaEEhadoop";
 String s4 = "javaEE" + "hadoop";
 String s5 = s1 + "hadoop";
 String s6 = "javaEE" + s2;
 String s7 = s1 + s2;
 
 System.out.println(s3 == s4);//true
 System.out.println(s3 == s5);//false
 System.out.println(s3 == s6);//false
 System.out.println(s3 == s7);//false
 System.out.println(s5 == s6);//false
 System.out.println(s5 == s7);//false
 System.out.println(s6 == s7);//false
 
 String s8 = s6.intern();//返回值得到的s8使用的常量值中已经存在的“javaEEhadoop”
 System.out.println(s3 == s8);//true
 
 
 
 String s1 = "javaEEhadoop";
 String s2 = "javaEE";
 String s3 = s2 + "hadoop";
 System.out.println(s1 == s3);//false
 
 final String s4 = "javaEE";//s4:常量
 String s5 = s4 + "hadoop";
 System.out.println(s1 == s5);//true
 ```

 

 5.常用方法：

 ```sh
 int length()：返回字符串的长度： return value.length
 char charAt(int index)： 返回某索引处的字符return value[index]
 boolean isEmpty()：判断是否是空字符串：return value.length == 0
 String toLowerCase()：使用默认语言环境，将 String 中的所字符转换为小写
 String toUpperCase()：使用默认语言环境，将 String 中的所字符转换为大写
 String trim()：返回字符串的副本，忽略前导空白和尾部空白
 boolean equals(Object obj)：比较字符串的内容是否相同
 boolean equalsIgnoreCase(String anotherString)：与equals方法类似，忽略大小写
 String concat(String str)：将指定字符串连接到此字符串的结尾。 等价于用“+”
 int compareTo(String anotherString)：比较两个字符串的大小
 String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
 String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
 
 boolean endsWith(String suffix)：测试此字符串是否以指定的后缀结束
 boolean startsWith(String prefix)：测试此字符串是否以指定的前缀开始
 boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始
 
 boolean contains(CharSequence s)：当且仅当此字符串包含指定的 char 值序列时，返回 true
 int indexOf(String str)：返回指定子字符串在此字符串中第一次出现处的索引
 int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
 int lastIndexOf(String str)：返回指定子字符串在此字符串中最右边出现处的索引
 int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索
 
 注：indexOf和lastIndexOf方法如果未找到都是返回-1
 
 替换：
 String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所 oldChar 得到的。
 String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所匹配字面值目标序列的子字符串。
 String replaceAll(String regex, String replacement)：使用给定的 replacement 替换此字符串所匹配给定的正则表达式的子字符串。
 String replaceFirst(String regex, String replacement)：使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串。
 匹配:
 boolean matches(String regex)：告知此字符串是否匹配给定的正则表达式。
 切片：
 String[] split(String regex)：根据给定正则表达式的匹配拆分此字符串。
 String[] split(String regex, int limit)：根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中。
 ```

 6.String与其它结构的转换	 

 6.1 与基本数据类型、包装类之间的转换

  ```sh
String --> 基本数据类型、包装类：调用包装类的静态方法：parseXxx(str)
基本数据类型、包装类 --> String:调用String重载的valueOf(xxx)
  ```

 ```java
  @Test
  public void test1(){
      String str1 = "123";
      //int num = (int)str1;//错误的
      int num = Integer.parseInt(str1);
      String str2 = String.valueOf(num);//"123"
      String str3 = num + "";
      System.out.println(str1 == str3);
  }
 ```
 6.2 与字符数组之间的转换

 ```sh
 String -- char[]:调用String的toCharArray()
 char[] -- String:调用String的构造器
 ```

 ```java
 @Test
 public void test2(){
     String str1 = "abc123";  //题目： a21cb3
     char[] charArray = str1.toCharArray();
     for (int i = 0; i < charArray.length; i++) {
         System.out.println(charArray[i]);
     }
     char[] arr = new char[]{'h','e','l','l','o'};
     String str2 = new String(arr);
     System.out.println(str2);
 }
 ```
 6.3 与字节数组之间的转换

 ```sh
 编码：String --> byte[]:调用String的getBytes()
 解码：byte[] --> String:调用String的构造器
 
 编码：字符串 -->字节  (看得懂 -->看不懂的二进制数据)
 解码：编码的逆过程，字节 --> 字符串 （看不懂的二进制数据 --> 看得懂
 
 说明：解码时，要求解码使用的字符集必须与编码时使用的字符集一致，否则会出现乱码。
 ```

 ```java
 @Test
 public void test3() throws UnsupportedEncodingException {
     String str1 = "abc123中国";
     byte[] bytes = str1.getBytes();//使用默认的字符集，进行编码。
     System.out.println(Arrays.toString(bytes));
     byte[] gbks = str1.getBytes("gbk");//使用gbk字符集进行编码。
     System.out.println(Arrays.toString(gbks));
     
     System.out.println("");
     String str2 = new String(bytes);//使用默认的字符集，进行解码。
     System.out.println(str2);
     String str3 = new String(gbks);
     System.out.println(str3);//出现乱码。原因：编码集和解码集不一致！
     String str4 = new String(gbks, "gbk");
     System.out.println(str4);//没出现乱码。原因：编码集和解码集一致！
 }
 ```

 6.4 与StringBuffer、StringBuilder之间的转换

 ```sh
 String --> StringBuffer、StringBuilder:调用StringBuffer、StringBuilder构造器
 StringBuffer、StringBuilder -->String:①调用String构造器；②StringBuffer、StringBuilder的toString()
 ```

 7.JVM中字符串常量池存放位置说明：

 ```sh
 jdk 1.6 (jdk 6.0 ,java 6.0):字符串常量池存储在方法区（永久区）
 jdk 1.7:字符串常量池存储在堆空间
 jdk 1.8:字符串常量池存储在方法区（元空间）
 ```

 8.常见算法题目的考查：

 ```sh
 1）模拟一个trim方法，去除字符串两端的空格。
 2）将一个字符串进行反转。将字符串中指定部分进行反转。比如“abcdefg”反转为”abfedcg”
 3）获取一个字符串在另一个字符串中出现的次数。
       比如：获取“ ab”在 “abkkcadkabkebfkabkskab” 中出现的次数
 4）获取两个字符串中最大相同子串。比如：
    str1 = "abcwerthelloyuiodef“;str2 = "cvhellobnm"
    提示：将短的那个串进行长度依次递减的子串与较长的串比较。
 5）对字符串中字符进行自然顺序排序。
 提示：
 1字符串变成字符数组。
 2对数组排序，择，冒泡，Arrays.sort();
 3将排序后的数组变成字符串。
 ```

 

## 9.2 StringBuffer、StringBuilder

 1.String、StringBuffer、StringBuilder三者的对比

 ```sh
 String:不可变的字符序列；底层使用char[]存储
 StringBuffer:可变的字符序列；线程安全的，效率低；底层使用char[]存储
 StringBuilder:可变的字符序列；jdk5.0新增的，线程不安全的，效率高；底层使用char[]存储
 ```

 2.StringBuffer与StringBuilder的内存解析

 ```java
 //以StringBuffer为例：
 String str = new String();//char[] value = new char[0];
 String str1 = new String("abc");//char[] value = new char[]{'a','b','c'};
 
 StringBuffer sb1 = new StringBuffer();//char[] value = new char[16];底层创建了一个长度是16的数组。
 System.out.println(sb1.length());//
 sb1.append('a');//value[0] = 'a';
 sb1.append('b');//value[1] = 'b';
 
 StringBuffer sb2 = new StringBuffer("abc");//char[] value = new char["abc".length() + 16];
 ```

问题1. System.out.println(sb2.length());//3

问题2. 扩容问题:如果要添加的数据底层数组盛不下了，那就需要扩容底层的数组。
         默认情况下，扩容为原来容量的2倍 + 2，同时将原数组中的元素复制到新的数组中。
         指导意义：开发中建议大家使用：StringBuffer(int capacity) 或 StringBuilder(int capacity)

 3.对比String、StringBuffer、StringBuilder三者的执行效率

 ```sh
 从高到低排列：StringBuilder  StringBuffer  String
 ```

 4.StringBuffer、StringBuilder中的常用方法

 ```sh
 增：append(xxx)
 删：delete(int start,int end)
 改：setCharAt(int n ,char ch) / replace(int start, int end, String str)
 查：charAt(int n )
 插：insert(int offset, xxx)
 长度：length();
 遍历：for() + charAt() / toString()
 ```

 

## 9.3 JDK8之前的日期时间API

1.获取系统当前时间：System类中的currentTimeMillis()

```java
long time = System.currentTimeMillis();
//返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差。
//称为时间戳
System.out.println(time);
```

2.java.util.Date类与java.sql.Date类


```java
/**
java.util.Date类
        |---java.sql.Date类
 1.两个构造器的使用
     构造器一：Date()：创建一个对应当前时间的Date对象
     构造器二：创建指定毫秒数的Date对象
 2.两个方法的使用
     toString():显示当前的年、月、日、时、分、秒
     getTime():获取当前Date对象对应的毫秒数。（时间戳）
 3. java.sql.Date对应着数据库中的日期类型的变量
    如何实例化
    如何将java.util.Date对象转换为java.sql.Date对象
*/
@Test
public void test2(){
    //构造器一：Date()：创建一个对应当前时间的Date对象
    Date date1 = new Date();
    System.out.println(date1.toString());//Sat Feb 16 16:35:31 GMT+08:00 2019
    System.out.println(date1.getTime());//1550306204104
    //构造器二：创建指定毫秒数的Date对象
    Date date2 = new Date(155030620410L);
    System.out.println(date2.toString());
    //创建java.sql.Date对象
    java.sql.Date date3 = new java.sql.Date(35235325345L);
    System.out.println(date3);//1971-02-13

    //如何将java.util.Date对象转换为java.sql.Date对象
    //情况一：
    //        Date date4 = new java.sql.Date(2343243242323L);
    //        java.sql.Date date5 = (java.sql.Date) date4;
    //情况二：
    Date date6 = new Date();
    java.sql.Date date7 = new java.sql.Date(date6.getTime());
}
```

3.java.text.SimpleDataFormat类

```sh
SimpleDateFormat对日期Date类的格式化和解析
1.两个操作：
1.1 格式化：日期 ---字符串
1.2 解析：格式化的逆过程，字符串 --- 日期
2.SimpleDateFormat的实例化:new + 构造器
```

```java
//照指定的方式格式化和解析：调用带参的构造器
//        SimpleDateFormat sdf1 = new SimpleDateFormat("yyyyy.MMMMM.dd GGG hh:mm aaa");
        SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        //格式化
        String format1 = sdf1.format(date);
        System.out.println(format1);//2019-02-18 11:48:27
        //解析:要求字符串必须是符合SimpleDateFormat识别的格式(通过构造器参数体现),
        //否则，抛异常
        Date date2 = sdf1.parse("2020-02-18 11:48:27");
        System.out.println(date2);
```

```java
//小练习：
/**
练习一：字符串"2020-09-08"转换为java.sql.Date
练习二："天打渔两天晒网"   1990-01-01  xxxx-xx-xx 打渔？晒网？

举例：2020-09-08 ？ 总天数
总天数 % 5 == 1,2,3 : 打渔
总天数 % 5 == 4,0 : 晒网

总天数的计算？
方式一：( date2.getTime() - date1.getTime()) / (1000  60  60  24) + 1
方式二：1990-01-01  -- 2019-12-31  +  2020-01-01 --2020-09-08
 */
@Test
public void testExer() throws ParseException {
    String birth = "2020-09-08";
    SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd");
    Date date = sdf1.parse(birth);//        System.out.println(date);
    java.sql.Date birthDate = new java.sql.Date(date.getTime());
    System.out.println(birthDate);
}
```
4.Calendar类：日历类、抽象类
		

```java
   //1.实例化
   //方式一：创建其子类（GregorianCalendar的对象
   //方式二：调用其静态方法getInstance()
    Calendar calendar = Calendar.getInstance();
   //System.out.println(calendar.getClass());
   //2.常用方法
    //get()
    int days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);
    System.out.println(calendar.get(Calendar.DAY_OF_YEAR));

    //set()
    //calendar可变性
    calendar.set(Calendar.DAY_OF_MONTH,22);
    days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);

    //add()
    calendar.add(Calendar.DAY_OF_MONTH,-3);
    days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);

    //getTime():日历类--- Date
    Date date = calendar.getTime();
    System.out.println(date);

    //setTime():Date --- 日历类
    Date date1 = new Date();
    calendar.setTime(date1);
    days = calendar.get(Calendar.DAY_OF_MONTH);
    System.out.println(days);
```




## 9.4 JDK8中新的时间API

 1.日期时间API的迭代：

 ```sh
 第一代：jdk 1.0 Date类
 第二代：jdk 1.1 Calendar类，一定程度上替换Date类
 第三代：jdk 1.8 提出了新的一套API
 ```

 2.前两代存在的问题举例：

 ```sh
 可变性：像日期和时间这样的类应该是不可变的。
 偏移性：Date中的年份是从1900开始的，而月份都从0开始。
 格式化：格式化只对Date用，Calendar则不行。
 此外，它们也不是线程安全的；不能处理闰秒等。
 ```

 3.java 8 中新的日期时间API涉及到的包

 ![image-20210221223845644](java-尚.assets/image-20210221223845644.png)

 4.本地日期、本地时间、本地日期时间的使用：LocalDate / LocalTime / LocalDateTime

 4.1 说明：

 ```properties
 ① 分别表示使用 ISO-8601日历系统的日期、时间、日期和时间。它们提供了简单的本地日期或时间，并不包含当前的时间信息，也不包含与时区相关的信息。
 ② LocalDateTime相较于LocalDate、LocalTime，使用频率要高
 ③ 类似于Calendar
 ```

 4.2 常用方法：

 ![image-20210221223919339](java-尚.assets/image-20210221223919339.png)

 5.时间点：Instant

 5.1 说明：

 ```properties
 ① 时间线上的一个瞬时点。 概念上讲，它只是简单的表示自1970年1月1日0时0分0秒（UTC开始的秒数。）
 ② 类似于 java.util.Date类
 ```

 5.2 常用方法：

 ![image-20210221223946245](java-尚.assets/image-20210221223946245.png)

 6.日期时间格式化类：DateTimeFormatter

 6.1 说明：

 ```properties
 ① 格式化或解析日期、时间
 ② 类似于SimpleDateFormat
 ```

 6.2 常用方法：

  ① 实例化方式：

预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME

本地化相关的格式。如：ofLocalizedDateTime(FormatStyle.LONG)

自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)

  ② 常用方法：

  ![image-20210221224040522](java-尚.assets/image-20210221224040522.png)

 特别的：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)

 ```java
 //  重点：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)
 DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
 //格式化
 String str4 = formatter3.format(LocalDateTime.now());
 System.out.println(str4);//2019-02-18 03:52:09
 
 //解析
 TemporalAccessor accessor = formatter3.parse("2019-02-18 03:52:09");
 System.out.println(accessor);
 ```

 7.其它API的使用 （不讲）

 7.1 带时区的日期时间：ZonedDateTime / ZoneId 
 举例：

 ```java
 	// ZoneId:类中包含了所的时区信息
 	@Test
 	public void test1(){
 		//getAvailableZoneIds():获取所的ZoneId
 		Set<String zoneIds = ZoneId.getAvailableZoneIds();
 		for(String s : zoneIds){
 			System.out.println(s);
 		}
 		System.out.println();
 	//获取“Asia/Tokyo”时区对应的时间
 	LocalDateTime localDateTime = LocalDateTime.now(ZoneId.of("Asia/Tokyo"));
 	System.out.println(localDateTime);}
 	//ZonedDateTime:带时区的日期时间
 	@Test
 	public void test2(){
 		//now():获取本时区的ZonedDateTime对象
 		ZonedDateTime zonedDateTime = ZonedDateTime.now();
 		System.out.println(zonedDateTime);
 		//now(ZoneId id):获取指定时区的ZonedDateTime对象
 		ZonedDateTime zonedDateTime1 = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));
 		System.out.println(zonedDateTime1);
 	}
 ```

 7.2 时间间隔：Duration--用于计算两个“时间”间隔，以秒和纳秒为基准

 举例：
 ```java
 	@Test
 	public void test3(){
 		LocalTime localTime = LocalTime.now();
 		LocalTime localTime1 = LocalTime.of(15, 23, 32);
 		//between():静态方法，返回Duration对象，表示两个时间的间隔
 		Duration duration = Duration.between(localTime1, localTime);
 		System.out.println(duration);
 	
         System.out.println(duration.getSeconds());
 	
         System.out.println(duration.getNano());
 	
 	
         LocalDateTime localDateTime = LocalDateTime.of(2016, 6, 12, 15, 23, 32);
 	
         LocalDateTime localDateTime1 = LocalDateTime.of(2017, 6, 12, 15, 23, 32);
 	
 	
         Duration duration1 = Duration.between(localDateTime1, localDateTime);
 	
         System.out.println(duration1.toDays());
 	
 }
 ```
 7.3 日期间隔：Period --用于计算两个“日期”间隔，以年、月、日衡量

 举例：

 ```java
 	@Test
 	public void test4(){
 		LocalDate localDate = LocalDate.now();
 		LocalDate localDate1 = LocalDate.of(2028, 3, 18);
 	
         Period period = Period.between(localDate, localDate1);
 	
         System.out.println(period);
 	
 	
         System.out.println(period.getYears());
 	
         System.out.println(period.getMonths());
 	
         System.out.println(period.getDays());
 	
 	
         Period period1 = period.withYears(2);
 	
         System.out.println(period1);
 	
 }
 ```
 7.4 日期时间校正器：TemporalAdjuster

 举例：

 ```java
 	@Test
 	public void test5(){
 		//获取当前日期的下一个周日是哪天？
 		TemporalAdjuster temporalAdjuster = TemporalAdjusters.next(DayOfWeek.SUNDAY);
 	
         LocalDateTime localDateTime = LocalDateTime.now().with(temporalAdjuster);
 	
         System.out.println(localDateTime);
 	
 	
         //获取下一个工作日是哪天？
 	
         LocalDate localDate = LocalDate.now().with(new TemporalAdjuster(){
 
 		@Override
 		public Temporal adjustInto(Temporal temporal) {
 			LocalDate date = (LocalDate)temporal;
 			if(date.getDayOfWeek().equals(DayOfWeek.FRIDAY)){
 				return date.plusDays(3);
 			}else if(date.getDayOfWeek().equals(DayOfWeek.SATURDAY)){
 				return date.plusDays(2);
 			}else{
 				return date.plusDays(1);
 			}
 				
 		}
 		
 	});
 	
 	System.out.println("下一个工作日是：" + localDate);
 }
 ```

## 9.5 java比较器

 1.Java比较器的使用背景：

 ```sh
 Java中的对象，正常情况下，只能进行比较：==  或  != 。不能使用  或 < 的但是在开发场景中，我们需要对多个对象进行排序，言外之意，就需要比较对象的大小。如何实现？使用两个接口中的任何一个：Comparable 或 Comparator
 ```

 2.自然排序：使用Comparable接口

 2.1 说明

 ```sh
 1.像String、包装类等实现了Comparable接口，重写了compareTo(obj)方法，给出了比较两个对象大小的方式。
 2.像String、包装类重写compareTo()方法以后，进行了从小到大的排列
 3. 重写compareTo(obj)的规则：
    如果当前对象this大于形参对象obj，则返回正整数，
    如果当前对象this小于形参对象obj，则返回负整数，
    如果当前对象this等于形参对象obj，则返回零。
 4. 对于自定义类来说，如果需要排序，我们可以让自定义类实现Comparable接口，重写compareTo(obj)方法。在compareTo(obj)方法中指明如何排序
 ```

 2.2 自定义类代码举例：

 ```java
 public class Goods implements  Comparable{
 private String name;
 private double price;
 
 //指明商品比较大小的方式:照价格从低到高排序,再照产品名称从高到低排序
 @Override
 public int compareTo(Object o) {
 //        System.out.println("");
         if(o instanceof Goods){
             Goods goods = (Goods)o;
             //方式一：
             if(this.price  goods.price){
                 return 1;
             }else if(this.price < goods.price){
                 return -1;
             }else{
 //                return 0;
                return -this.name.compareTo(goods.name);
             }
             //方式二：
 //           return Double.compare(this.price,goods.price);
         }
 //        return 0;
         throw new RuntimeException("传入的数据类型不一致！");
     }
 // getter、setter、toString()、构造器：省略
 }
 ```

 3.定制排序：使用Comparator接口

 3.1 说明

 ```sh
 1.背景：
 当元素的类型没实现java.lang.Comparable接口而又不方便修改代码，或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，那么可以考虑使用 Comparator 的对象来排序
 
 2.重写compare(Object o1,Object o2)方法，比较o1和o2的大小：
 如果方法返回正整数，则表示o1大于o2；
 如果返回0，表示相等；
 返回负整数，表示o1小于o2。
 ```

 3.2 代码举例：

 ```java
 Comparator com = new Comparator() {
     //指明商品比较大小的方式:照产品名称从低到高排序,再照价格从高到低排序
     @Override
     public int compare(Object o1, Object o2) {
         if(o1 instanceof Goods && o2 instanceof Goods){
             Goods g1 = (Goods)o1;
             Goods g2 = (Goods)o2;
             if(g1.getName().equals(g2.getName())){
                 return -Double.compare(g1.getPrice(),g2.getPrice());
             }else{
                 return g1.getName().compareTo(g2.getName());
             }
         }
         throw new RuntimeException("输入的数据类型不一致");
     }
 }
 ```

 使用：

 ```java
 Arrays.sort(goods,com);
 Collections.sort(coll,com);
 new TreeSet(com);
 ```

 

 4.两种排序方式对比

 ```sh
     Comparable接口的方式一旦一定，保证Comparable接口实现类的对象在任何位置都可以比较大小。
     Comparator接口属于临时性的比较。
 ```

 



## 9.6 其他类

 1.System类

 - System类代表系统，系统级的很多属性和控制方法都放置在该类的内部。该类位于java.lang包。
 - 由于该类的构造器是private的，所以无法创建该类的对象，也就是无法实例化该类。其内部的成员变量和成员方法都是static的，所以也可以很方便的进行调用。
 - 方法：
   native long currentTimeMillis()
   void exit(int status)
   void gc()
   String getProperty(String key)

 2.Math类

 java.lang.Math提供了一系列静态方法用于科学计算。其方法的参数和返回值类型一般为double型。

 3.BigInteger类、BigDecimal类

 说明：

 ① java.math包的BigInteger可以表示不可变的任意精度的整数。

 ② 要求数字精度比较高，用到java.math.BigDecimal类

 代码举例：

 ![image-20210221224715079](java-尚.assets/image-20210221224715079.png)



# 第10章 枚举类和注解

## 10.1 枚举类的使用

 1.枚举类的说明：

 ```sh
  1.枚举类的理解：类的对象只有有限个，确定的。我们称此类为枚举类
  2.当需要定义一组常量时，强烈建议使用枚举类
  3.如果枚举类中只一个对象，则可以作为单例模式的实现方式。
 ```

 2.如何自定义枚举类？步骤：

 ```java
 //自定义枚举类
 class Season{
  //1.声明Season对象的属性:private final修饰
  private final String seasonName;
  private final String seasonDesc;
 
  //2.私化类的构造器,并给对象属性赋值
  private Season(String seasonName,String seasonDesc){
      this.seasonName = seasonName;
      this.seasonDesc = seasonDesc;
  }
 
  //3.提供当前枚举类的多个对象：public static final的
  public static final Season SPRING = new Season("春天","春暖花开");
  public static final Season SUMMER = new Season("夏天","夏日炎炎");
  public static final Season AUTUMN = new Season("秋天","秋高气爽");
  public static final Season WINTER = new Season("冬天","冰天雪地");
 
  //4.其他诉求1：获取枚举类对象的属性
  public String getSeasonName() {
      return seasonName;
  }
 
  public String getSeasonDesc() {
      return seasonDesc;
  }
  //4.其他诉求1：提供toString()
  @Override
  public String toString() {
      return "Season{" +
              "seasonName='" + seasonName + '\'' +
              ", seasonDesc='" + seasonDesc + '\'' +
              '}';
  }
 }
 ```

 

 3.jdk 5.0 新增使用enum定义枚举类。步骤：

 ```java
 //使用enum关键字枚举类
 enum Season1 {
  //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
  SPRING("春天","春暖花开"),
  SUMMER("夏天","夏日炎炎"),
  AUTUMN("秋天","秋高气爽"),
  WINTER("冬天","冰天雪地");
 
  //2.声明Season对象的属性:private final修饰
  private final String seasonName;
  private final String seasonDesc;
 
  //2.私化类的构造器,并给对象属性赋值
 
  private Season1(String seasonName,String seasonDesc){
      this.seasonName = seasonName;
      this.seasonDesc = seasonDesc;
  }
 
  //4.其他诉求1：获取枚举类对象的属性
  public String getSeasonName() {
      return seasonName;
  }
 
  public String getSeasonDesc() {
      return seasonDesc;
  }
 
 }
 ```

 

 4.使用enum定义枚举类之后，枚举类常用方法：（继承于java.lang.Enum类）

 ```java
 Season1 summer = Season1.SUMMER;
 //toString():返回枚举类对象的名称
 System.out.println(summer.toString());
 //System.out.println(Season1.class.getSuperclass());
 System.out.println("");
 //values():返回所的枚举类对象构成的数组
 Season1[] values = Season1.values();
 for(int i = 0;i < values.length;i++){
     System.out.println(values[i]);
 }
 System.out.println("");
 Thread.State[] values1 = Thread.State.values();
 for (int i = 0; i < values1.length; i++) {
     System.out.println(values1[i]);
 }
 //valueOf(String objName):返回枚举类中对象名是objName的对象。
 Season1 winter = Season1.valueOf("WINTER");
 //如果没objName的枚举类对象，则抛异常：IllegalArgumentException
 //Season1 winter = Season1.valueOf("WINTER1");
 System.out.println(winter);
 ```
 5.使用enum定义枚举类之后，如何让枚举类对象分别实现接口：

 ```java
 interface Info{
  void show();
 }
 
 //使用enum关键字枚举类
 enum Season1 implements Info{
     //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
     SPRING("春天","春暖花开"){
         @Override
         public void show() {
             System.out.println("春天在哪里？");
         }
     },
     SUMMER("夏天","夏日炎炎"){
         @Override
         public void show() {
             System.out.println("宁夏");
         }
     },
     AUTUMN("秋天","秋高气爽"){
         @Override
         public void show() {
             System.out.println("秋天不回来");
         }
     },
     WINTER("冬天","冰天雪地"){
         @Override
         public void show() {
             System.out.println("大约在冬季");
         }
     };
 }
 ```

 

## 10.2 注解的使用

 1.注解的理解

 ```sh
 ① jdk 5.0 新增的功能
 ② Annotation 其实就是代码里的特殊标记, 这些标记可以在编译, 类加载, 运行时被读取, 并执行相应的处理。通过使用 Annotation,程序员可以在不改变原逻辑的情况下, 在源文件中嵌入一些补充信息。
  ③在JavaSE中，注解的使用目的比较简单，例如标记过时的功能，忽略警告等。在JavaEE/Android
  中注解占据了更重要的角色，例如用来配置应用程序的任何切面，代替JavaEE旧版中所遗留的繁冗
  代码和XML配置等。
 
 框架 = 注解 + 反射机制 + 设计模式
 ```

 2.注解的使用示例

 ```sh
  示例一：生成文档相关的注解
  示例二：在编译时进行格式检查(JDK内置的个基本注解)
   @Override: 限定重写父类方法, 该注解只能用于方法
   @Deprecated: 用于表示所修饰的元素(类, 方法等)已过时。通常是因为所修饰的结构危险或存在更好的择
   @SuppressWarnings: 抑制编译器警告
 
   示例：跟踪代码依赖性，实现替代配置文件功能
 ```

 3.如何自定义注解：参照@SuppressWarnings定义

 ```sh
   ① 注解声明为：@interface
   ② 内部定义成员，通常使用value表示
   ③ 可以指定成员的默认值，使用default定义
   ④ 如果自定义注解没成员，表明是一个标识作用。
 
 
 说明：
 如果注解有成员，在使用注解时，需要指明成员的值。
 自定义注解必须配上注解的信息处理流程(使用反射)才意义。
 自定义注解通过都会指明两个元注解：Retention、Target
 ```

 代码举例：

 ```java
 @Inherited
 @Repeatable(MyAnnotations.class)
 @Retention(RetentionPolicy.RUNTIME)
 @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE,TYPE_PARAMETER,TYPE_USE})
 public @interface MyAnnotation {
     String value() default "hello";
 }
 ```
 4.元注解 ：对现有的注解进行解释说明的注解。 

 ```sh
 jdk 提供的4种元注解：
 Retention：指定所修饰的 Annotation 的生命周期：SOURCE\CLASS（默认行为\RUNTIME只声明为RUNTIME生命周期的注解，才能通过反射获取。
 Target:用于指定被修饰的 Annotation 能用于修饰哪些程序元素出现的频率较低
 Documented:表示所修饰的注解在被javadoc解析时，保留下来。
 Inherited:被它修饰的 Annotation 将具继承性。
 
 ---类比：元数据的概念：String name = "Tom";
 ```

 5.如何获取注解信息:通过发射来进行获取、调用。

 前提：要求此注解的元注解Retention中声明的生命周期状态为：RUNTIME.

 6.JDK8中注解的新特性：可重复注解、类型注解

 ```sh
 6.1 可重复注解：
 ① 在MyAnnotation上声明@Repeatable，成员值为MyAnnotations.class
 ② MyAnnotation的Target和Retention等元注解与MyAnnotations相同。
 
 6.2 类型注解：
 ElementType.TYPE_PARAMETER 表示该注解能写在类型变量的声明语句中（如：泛型声明。
 ElementType.TYPE_USE 表示该注解能写在使用类型的任何语句中。
 ```

 



# 第11章 JAVA集合

## 11.1 数组与集合

1、集合与数组存储数据概述：

   ```properties
   集合、数组都是对多个数据进行存储操作的结构，简称Java容器。
   说明：此时的存储，主要指的是内存层面的存储，不涉及到持久化的存储（.txt,.jpg,.avi，数据库中)
   ```

2、数组存储的特点：

 ```properties
 一旦初始化以后，其长度就确定了。
 数组一旦定义好，其元素的类型也就确定了。我们也就只能操作指定类型的数据了。
 比如：String[] arr;int[] arr1;Object[] arr2;
 ```

 3、数组存储的弊端：

 ```properties
  一旦初始化以后，其长度就不可修改。
 数组中提供的方法非常限，对于添加、删除、插入数据等操作，非常不便，同时效率不高。
 获取数组中实际元素的个数的需求，数组没有现成的属性或方法可用
 数组存储数据的特点：有序、可重复。对于无序、不可重复的需求，不能满足。
 ```

4、集合存储的优点：

解决数组存储数据方面的弊端。

## 11.2 Collection接口

1、单列集合框架结构

 ```sh
 |----Collection接口：单列集合，用来存储一个一个的对象
 
           |----List接口：存储序的、可重复的数据。  --“动态”数组
           |----ArrayList、LinkedList、Vector
            
           |----Set接口：存储无序的、不可重复的数据   --高中讲的“集合”
           |----HashSet、LinkedHashSet、TreeSet
 ```

 对应图示：

![image-20210222205445608](java-尚.assets/image-20210222205445608.png)

 2、Collection接口常用方法：

 ```java
 add(Object obj),
 addAll(Collection coll),
 size(),
 isEmpty(),
 clear();
 contains(Object obj),
 containsAll(Collection coll),
 remove(Object obj),
 removeAll(Collection coll),
 retainsAll(Collection coll),
 equals(Object obj);
 hasCode(),
 toArray(),
 iterator();
 ```

3、Collection集合与数组间的转换

 ```java
 //集合 ---数组：toArray()
 Object[] arr = coll.toArray();
 for(int i = 0;i < arr.length;i++){
     System.out.println(arr[i]);
 }
 
 //拓展：数组 ---集合:调用Arrays类的静态方法asList(T ... t)
 List<String list = Arrays.asList(new String[]{"AA", "BB", "CC"});
 System.out.println(list);
 
 List arr1 = Arrays.asList(new int[]{123, 456});
 System.out.println(arr1.size());//1
 
 List arr2 = Arrays.asList(new Integer[]{123, 456});
 System.out.println(arr2.size());//2
 ```

 4、使用Collection集合存储对象，要求对象所属的类满足：

 ```sh
 向Collection接口的实现类的对象中添加数据obj时，要求obj所在类要重写equals().
 ```

 5、本章节对大家的要求：

 ```sh
 层次一：选择合适的集合类去实现数据的保存，调用其内部的相关方法。
 
 层次二：不同的集合类底层的数据结构为何？如何实现数据的操作的：增删改查等。
 ```



## 11.3 Iterator接口与foreach循环

1、遍历Collection的两种方式：

 ```sh
 ① 使用迭代器Iterator 
 ② foreach循环（或增强for循环）
 ```

2、java.utils包下定义的迭代器接口：Iterator

2.1、说明：

 ```sh
 Iterator对象称为迭代器(设计模式的一种)，主要用于遍历 Collection 集合中的元素。
 GOF给迭代器模式的定义为：提供一种方法访问一个容器(container)对象中各个元素，而又不需暴露该对象的内部细节。迭代器模式，就是为容器而生。
 ```

2.2、作用：遍历集合Collectiton元素

2.3、如何获取实例：coll.iterator()返回一个迭代器实例

2.4、遍历的代码实现：

 ```java
 Iterator iterator = coll.iterator();
 //hasNext():判断是否还下一个元素
 while(iterator.hasNext()){
     //next():①指针下移 ②将下移以后集合位置上的元素返回
     System.out.println(iterator.next());
 }
 ```

2.5、图示说明：

  ![image-20210222205813348](java-尚.assets/image-20210222205813348.png)

2.6、 remove()的使用：

 ```java
  //测试Iterator中的remove()
 //如果还未调用next()或在上一次调用 next 方法之后已经调用了 remove 方法，再调用remove都会报IllegalStateException。
 //内部定义了remove(),可以在遍历的时候，删除集合中的元素。此方法不同于集合直接调用remove()
 @Test
 public void test3(){
     Collection coll = new ArrayList();
     coll.add(123);
     coll.add(456);
     coll.add(new Person("Jerry",20));
     coll.add(new String("Tom"));
     coll.add(false);
     //删除集合中"Tom"
     Iterator iterator = coll.iterator();
     while (iterator.hasNext()){
         //iterator.remove();
         Object obj = iterator.next();
         if("Tom".equals(obj)){
             iterator.remove();
             //iterator.remove();
         }
     }
     //遍历集合
     iterator = coll.iterator();
     while (iterator.hasNext()){
         System.out.println(iterator.next());
     }
 }   
 ```

3、jdk5.0新特性--增强for循环：(foreach循环)

1.遍历集合举例：

 ```java
 @Test
 public void test1(){
     Collection coll = new ArrayList();
     coll.add(123);
     coll.add(456);
     coll.add(new Person("Jerry",20));
     coll.add(new String("Tom"));
     coll.add(false);
     //for(集合元素的类型 局部变量 : 集合对象)
     for(Object obj : coll){
         System.out.println(obj);
     }
 }
 ```
 说明：

 内部仍然调用了迭代器。

 2.遍历数组举例：

 ```java
 @Test
 public void test2(){
     int[] arr = new int[]{1,2,3,4,5,6};
     //for(数组元素的类型 局部变量 : 数组对象)
     for(int i : arr){
         System.out.println(i);
     }
 }
 ```

## 11.4 Collection子接口：List接口

存储的数据特点：存储序的、可重复的数据。

常用方法：(记住)

 ```sh
 增：add(Object obj)
 删：remove(int index) / remove(Object obj)
 改：set(int index, Object ele)
 查：get(int index)
 插：add(int index, Object ele)
 长度：size()
 遍历：
 ① Iterator迭代器方式
 ② 增强for循环
 ③ 普通的循环
 ```

​    3、常用实现类：

   ```sh
    |----Collection接口：单列集合，用来存储一个一个的对象
   	 |----List接口：存储序的、可重复的数据。  --“动态”数组,替换原的数组
   	 |----ArrayList：作为List接口的主要实现类；线程不安全的，效率高；底层使用Object[]elementData存储
    	  |----LinkedList：对于频繁的插入、删除操作，使用此类效率比ArrayList高；底层使用双向链表存储
            |----Vector：作为List接口的古老实现类；线程安全的，效率低；底层使用Object[] elementData存储
   ```

   4、源码分析(难点)

4.1 、ArrayList的源码分析：

 ```sh
 jdk 7情况下
    ArrayList list = new ArrayList();//底层创建了长度是10的Object[]数组elementData
    list.add(123);//elementData[0] = new Integer(123);
    ...
    list.add(11);//如果此次的添加导致底层elementData数组容量不够，则扩容。
    默认情况下，扩容为原来的容量的1.5倍，同时需要将原有数组中的数据复制到新的数组中。
     
    结论：建议开发中使用带参的构造器：ArrayList list = new ArrayList(int capacity)
     
 jdk 8中ArrayList的变化：
    ArrayList list = new ArrayList();//底层Object[] elementData初始化为{}.并没创建长度为10的数组
     
    list.add(123);//第一次调用add()时，底层才创建了长度10的数组，并将数据123添加到elementData[0]
    ...
    后续的添加和扩容操作与jdk 7 无异。
 
 小结：jdk7中的ArrayList的对象的创建类似于单例的饿汉式，而jdk8中的ArrayList的对象的创建类似于单例的懒汉式，延迟了数组的创建，节省内存。
 ```

 4.2 LinkedList的源码分析：

 ```java
  LinkedList list = new LinkedList(); 内部声明了Node类型的first和last属性，默认值为null
  list.add(123);//将123封装到Node中，创建了Node对象。
 
 //其中，Node定义为：体现了LinkedList的双向链表的说法
  private static class Node<E {
        E item;
        Node<E next;
        Node<E prev;
 
        Node(Node<E prev, E element, Node<E next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
        }
    }
 ```

 4.3 Vector的源码分析：

 ```sh
 jdk7和jdk8中通过Vector()构造器创建对象时，底层都创建了长度为10的数组。
 在扩容方面，默认扩容为原来的数组长度的2倍。
 ```

 5、存储的元素的要求：

添加的对象，所在的类要重写equals()方法

[面试题]

 ```sh
 面试题：ArrayList、LinkedList、Vector者的异同？
 同：三个类都是实现了List接口，存储数据的特点相同：存储序的、可重复的数据
 不同：见上（第3部分+第4部分）
 ```

## 11.5 Collection子接口：Set接口

1、存储的数据特点：无序的、不可重复的元素

具体的：

   ```sh
   以HashSet为例说明：
   1. 无序性：不等于随机性。存储的数据在底层数组中并非照数组索引的顺序添加，而是根据数据的哈希值决定的。
   2. 不可重复性：保证添加的元素照equals()判断时，不能返回true.即：相同的元素只能添加一个。
   ```

   2、元素添加过程：(以HashSet为例)

 ```sh
  我们向HashSet中添加元素a,首先调用元素a所在类的hashCode()方法，计算元素a的哈希值，此哈希值接着通过某种算法计算出在HashSet底层数组中的存放位置（即为：索引位置，判断数组此位置上是否已经元素：如果此位置上没其他元素，则元素a添加成功。 ---情况1
        如果此位置上其他元素b(或以链表形式存在的多个元素，则比较元素a与元素b的hash值：
        如果hash值不相同，则元素a添加成功。---情况2
        如果hash值相同，进而需要调用元素a所在类的equals()方法：
                   equals()返回true,元素a添加失败
                   equals()返回false,则元素a添加成功。---情况2
    
    对于添加成功的情况2和情况3而言：元素a 与已经存在指定索引位置上数据以链表的方式存储。
    jdk 7 :元素a放到数组中，指向原来的元素。
    jdk 8 :原来的元素在数组中，指向元素a
    
 总结：七上八下
 HashSet底层：数组+链表的结构。（前提：jdk7)
 ```

​    3、常用方法

   ```sh
   Set接口中没额外定义新的方法，使用的都是Collection中声明过的方法。
   ```

   4、常用实现类：

   ```sh
    |----Collection接口：单列集合，用来存储一个一个的对象
   
             |----Set接口：存储无序的、不可重复的数据   --高中讲的“集合”
             |----HashSet：作为Set接口的主要实现类；线程不安全的；可以存储null值
             |----LinkedHashSet：作为HashSet的子类；遍历其内部数据时，可以按照添加的顺序遍历
             在添加数据的同时，每个数据还维护了两个引用，记录此数据前一个数据和后一个数据。对于频繁的遍历操作，LinkedHashSet效率高于HashSet.
             |----TreeSet：可以照添加对象的指定属性，进行排序。
   ```

   5、存储对象所在类的要求：

   ```sh
   HashSet/LinkedHashSet:
   
   要求：向Set(主要指：HashSet、LinkedHashSet)中添加的数据，其所在的类一定要重写hashCode()和equals()
   要求：重写的hashCode()和equals()尽可能保持一致性：相等的对象必须具有相等的散列码
   
   重写两个方法的小技巧：对象中用作 equals() 方法比较的 Field，都应该用来计算 hashCode 值。
        
   
   TreeSet:
   1.自然排序中，比较两个对象是否相同的标准为：compareTo()返回0.不再是equals().
   2.定制排序中，比较两个对象是否相同的标准为：compare()返回0.不再是equals().
   ```

   6、TreeSet的使用

6.1 使用说明:

   ```sh
   1.向TreeSet中添加的数据，要求是相同类的对象。
   2.两种排序方式：自然排序（实现Comparable接口 和 定制排序（Comparator）
   ```

   6.2 常用的排序方式:

   ```java
    //方式一：自然排序
      @Test
          public void test1(){
              TreeSet set = new TreeSet();
   
       //失败：不能添加不同类的对象
   
   //        set.add(123);
   //        set.add(456);
   //        set.add("AA");
   //        set.add(new User("Tom",12));
   
       //举例一：
   
   //        set.add(34);
   //        set.add(-34);
   //        set.add(43);
   //        set.add(11);
   //        set.add(8);
   
   //举例二：
       set.add(new User("Tom",12));
       set.add(new User("Jerry",32));
       set.add(new User("Jim",2));
       set.add(new User("Mike",65));
       set.add(new User("Jack",33));
       set.add(new User("Jack",56));
   
      Iterator iterator = set.iterator();
       while(iterator.hasNext()){
           System.out.println(iterator.next());
       }
   
   }
   
   //方式二：定制排序
       @Test
       public void test2(){
           Comparator com = new Comparator() {
               //照年龄从小到大排列
               @Override
               public int compare(Object o1, Object o2) {
                   if(o1 instanceof User && o2 instanceof User){
                       User u1 = (User)o1;
                       User u2 = (User)o2;
                       return Integer.compare(u1.getAge(),u2.getAge());
                   }else{
                       throw new RuntimeException("输入的数据类型不匹配");
                   }
               }
           };
   
    TreeSet set = new TreeSet(com);
       set.add(new User("Tom",12));
       set.add(new User("Jerry",32));
       set.add(new User("Jim",2));
       set.add(new User("Mike",65));
       set.add(new User("Mary",33));
       set.add(new User("Jack",33));
       set.add(new User("Jack",56));
   
   Iterator iterator = set.iterator();
           while(iterator.hasNext()){
               System.out.println(iterator.next());
           }
       }
   ```


## 11.6 Map接口

 双列集合框架：Map

 1.常用实现类结构

 ```sh
 |----Map:双列数据，存储key-value对的数据   ---类似于高中的函数：y = f(x)
 
        |----HashMap:作为Map的主要实现类；线程不安全的，效率高；存储null的key和value
        |----LinkedHashMap:保证在遍历map元素时，可以照添加的顺序实现遍历。
        原因：在原的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。
        对于频繁的遍历操作，此类执行效率高于HashMap。
        |----TreeMap:保证照添加的key-value对进行排序，实现排序遍历。此时考虑key的自然排序或定制排序
        底层使用红黑树
        |----Hashtable:作为古老的实现类；线程安全的，效率低；不能存储null的key和value
        |----Properties:常用来处理配置文件。key和value都是String类型
         
         
        HashMap的底层：数组+链表  （jdk7及之前)
        数组+链表+红黑树 （jdk 8)
 
 [面试题]
 
   1. HashMap的底层实现原理？
   2. HashMap 和 Hashtable的异同？
   3. CurrentHashMap 与 Hashtable的异同？（暂时不讲)
 ```

 2.存储结构的理解：

 ```sh
 Map中的key:无序的、不可重复的，使用Set存储所的key  --- key所在的类要重写equals()和hashCode() （以HashMap为例)
 Map中的value:无序的、可重复的，使用Collection存储所的value ---value所在的类要重写equals()
 一个键值对：key-value构成了一个Entry对象。
 Map中的entry:无序的、不可重复的，使用Set存储所的entry
 ```

 图示：

 ![image-20210222213203977](java-尚.assets/image-20210222213203977.png)

 3.常用方法

 ```sh
  添加：put(Object key,Object value)
  删除：remove(Object key)
  修改：put(Object key,Object value)
  查询：get(Object key)
  长度：size()
  遍历：keySet() / values() / entrySet()
 ```

 4.内存结构说明：（难点）

 4.1 HashMap在jdk7中实现原理：

 ```sh
 HashMap map = new HashMap():
 
       在实例化以后，底层创建了长度是16的一维数组Entry[] table。
       ...可能已经执行过多次put...
       map.put(key1,value1):
       首先，调用key1所在类的hashCode()计算key1哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置。
       如果此位置上的数据为空，此时的key1-value1添加成功。 ----情况1
       如果此位置上的数据不为空，(意味着此位置上存在一个或多个数据(以链表形式存在)),比较key1和已经存在的一个或多个数据的哈希值：
       如果key1的哈希值与已经存在的数据的哈希值都不相同，此时key1-value1添加成功。----情况2
       如果key1的哈希值和已经存在的某一个数据(key2-value2)的哈希值相同，继续比较：调用key1所在类的equals(key2)方法，比较：
       如果equals()返回false:此时key1-value1添加成功。----情况3
       如果equals()返回true:使用value1替换value2。
        
       补充：关于情况2和情况3：此时key1-value1和原来的数据以链表的方式存储。
        
       在不断的添加过程中，会涉及到扩容问题，当超出临界值(且要存放的位置非空)时，扩容。默认的扩容方式：扩容为原来容量的2倍，并将原的数据复制过来。
 ```

 4.2 HashMap在jdk8中相较于jdk7在底层实现方面的不同：

 ```sh
 1. new HashMap():底层没创建一个长度为16的数组
 2. jdk 8底层的数组是：Node[],而非Entry[]
 3. 首次调用put()方法时，底层创建长度为16的数组
 4. jdk7底层结构只：数组+链表。jdk8中底层结构：数组+链表+红黑树。
    4.1 形成链表时，七上八下（jdk7:新的元素指向旧的元素。jdk8：旧的元素指向新的元素）
    4.2 当数组的某一个索引位置上的元素以链表形式存在的数据个数  8 且当前数组的长度  64时，此时此索引位置上的所数据改为使用红黑树存储。
 ```

  4.3 HashMap底层典型属性的属性的说明：

 ```sh
 DEFAULT_INITIAL_CAPACITY : HashMap的默认容量，16
 DEFAULT_LOAD_FACTOR：HashMap的默认加载因子：0.75
 threshold：扩容的临界值，=容量填充因子：16  0.75 = 12
 TREEIFY_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树:8
 MIN_TREEIFY_CAPACITY：桶中的Node被树化时最小的hash表容量:64
 ```

 4.4 LinkedHashMap的底层实现原理(了解)

 ```sh
 LinkedHashMap底层使用的结构与HashMap相同，因为LinkedHashMap继承于HashMap.
 区别就在于：LinkedHashMap内部提供了Entry，替换HashMap中的Node.
 ```

  ![image-20210222213352971](java-尚.assets/image-20210222213352971.png)

 5.TreeMap的使用

 ```sh
 //向TreeMap中添加key-value，要求key必须是由同一个类创建的对象
 //因为要照key进行排序：自然排序 、定制排序
 ```

 6.使用Properties读取配置文件

 ```java
 //Properties:常用来处理配置文件。key和value都是String类型
 public static void main(String[] args)  {
     FileInputStream fis = null;
     try {
         Properties pros = new Properties();
     fis = new FileInputStream("jdbc.properties");
     pros.load(fis);//加载流对应的文件
 
     String name = pros.getProperty("name");
     String password = pros.getProperty("password");
 
     System.out.println("name = " + name + ", password = " + password);
 } catch (IOException e) {
     e.printStackTrace();
 } finally {
     if(fis != null){
         try {
             fis.close();
         } catch (IOException e) {
             e.printStackTrace();
         }
     }
 }
 ```

## 11.7 Collections工具类

 Collections工具类

 1.作用：操作Collection和Map的工具类

 2.常用方法：

 ```sh
 reverse(List)：反转 List 中元素的顺序
 shuffle(List)：对 List 集合元素进行随机排序
 sort(List)：根据元素的自然顺序对指定 List 集合元素升序排序
 sort(List，Comparator)：根据指定的 Comparator 产生的顺序对 List 集合元素进行排序
 swap(List，int， int)：将指定 list 集合中的 i 处元素和 j 处元素进行交换
 Object max(Collection)：根据元素的自然顺序，返回给定集合中的最大元素
 Object max(Collection，Comparator)：根据 Comparator 指定的顺序，返回给定集合中的最大元素
 Object min(Collection)
 Object min(Collection，Comparator)
 int frequency(Collection，Object)：返回指定集合中指定元素的出现次数
 void copy(List dest,List src)：将src中的内容复制到dest中
 boolean replaceAll(List list，Object oldVal，Object newVal)：使用新值替换 List 对象的所旧值
 ```

  ![image-20210222213556856](java-尚.assets/image-20210222213556856.png)


 说明：ArrayList和HashMap都是线程不安全的，如果程序要求线程安全，我们可以将ArrayList、HashMap转换为线程的。使用synchronizedList(List list） 和 synchronizedMap(Map map）

 3.面试题：

面试题：Collection 和 Collections的区别？

## 11.8 数据结构简述

 1.数据结构概述

 ```sh
 数据结构（Data Structure是一门和计算机硬件与软件都密切相关的学科，它的研究重点是在计算机的程序设计领域中探讨如何在计算机中组织和存储数据并进行高效率的运用，涉及的内容包含：数据的逻辑关系、数据的存储结构、排序算法（Algorithm）、查找（或搜索）等。
 ```

 2.数据结构与算法的理解

 ```sh
 程序能否快速而高效地完成预定的任务，取决于是否选对了数据结构，而程序是否能清楚而正确地把问题解决，则取决于算法。
 所以大家认为：“Algorithms + Data Structures = Programs”（出自：Pascal之父Nicklaus Wirth）
 总结：算法是为了解决实际问题而设计的，数据结构是算法需要处理的问题载体。
 ```

 3.数据结构的研究对象

 3.1 数据间的逻辑结构

 ![image-20210222214151599](java-尚.assets/image-20210222214151599.png)

 3.2 数据的存储结构：

 线性表（顺序表、链表、栈、队列）

 树

 图

 说明：习惯上把顺序表和链表看做基本数据结构（或真实数据结构）

 习惯上把栈、队列、树、图看做抽象数据类型，简称ADT

 4.使用详情见思维导图：

 《附录：尚硅谷\_宋红康_数据结构概述-Java版.xmind》



# 第12章 泛型

## 12.1 泛型的理解

 1.泛型的概念

 ```sh
 所谓泛型，就是允许在定义类、接口时通过一个标识表示类中某个属性的类型或者是某个方法的返
 回值及参数类型。这个类型参数将在使用时（例如，继承或实现这个接口，用这个类型声明变量、
 创建对象时确定（即传入实际的类型参数，也称为类型实参）。
 ```

 2.泛型的引入背景

 ```sh
 集合容器类在设计阶段/声明阶段不能确定这个容器到底实际存的是什么类型的对象，所以在JDK1.5之前只能把元素类型设计为Object，JDK1.5之后使用泛型来解决。因为这个时候除了元素的类型不确定，其他的部分是确定的，例如关于这个元素如何保存，如何管理等是确定的，因此此时把元素的类型设计成一个参数，这个类型参数叫做泛型。Collection<E，List<E，ArrayList<E   这个<E就是类型参数，即泛型。
 ```

## 12.2 泛型在集合中的使用

 1. 在集合中使用泛型之前的例子

    
    ```java
    @Test
     public void test1(){
         ArrayList list = new ArrayList();
         //需求：存放学生的成绩
         list.add(78);
         list.add(76);
      list.add(89);
      list.add(88);
         //问题一：类型不安全
    //        list.add("Tom");
    for(Object score : list){
      //问题二：强转时，可能出现ClassCastException
         int stuScore = (Integer) score;
    
         System.out.println(stuScore);
    
     } 
     }
    ```
    

 ​       图示：

 ![image-20210222214316848](java-尚.assets/image-20210222214316848.png)


 2. 在集合中使用泛型例子1

    
    ```java
    @Test
    public void test2(){
        ArrayList<Integer list =  new ArrayList<Integer();
        list.add(78);
        list.add(87);
        list.add(99);
        list.add(65);
        //编译时，就会进行类型检查，保证数据的安全
        // list.add("Tom");
    
        //方式一：
        // for(Integer score : list){
        //避免了强转操作
        //  int stuScore = score;
        // System.out.println(stuScore);
    // }
         //方式二：
         Iterator<Integer iterator = list.iterator();
         while(iterator.hasNext()){
             int stuScore = iterator.next();
             System.out.println(stuScore);
      }
    }
    ```

![image-20210222214419591](java-尚.assets/image-20210222214419591.png)

 3. 在集合中使用泛型例子2

    
    ```java
    //在集合中使用泛型的情况：以HashMap为例
     @Test
     public void test3(){
         //  Map<String,Integer map = new HashMap<String,Integer();
         //jdk7新特性：类型推断
         Map<String,Integer map = new HashMap<();
         map.put("Tom",87);
         map.put("Jerry",87);
         map.put("Jack",67);
         //map.put(123,"ABC");
         //泛型的嵌套
         Set<Map.Entry<String,Integer entry = map.entrySet();
         Iterator<Map.Entry<String, Integer iterator = entry.iterator();
         while(iterator.hasNext()){
            Map.Entry<String, Integer e = iterator.next();
            String key = e.getKey();
            Integer value = e.getValue();
            System.out.println(key + "----" + value);
        }
    }    
    ```


 4. 集合中使用泛型总结：

 ```sh
   ① 集合接口或集合类在jdk5.0时都修改为带泛型的结构。
   ② 在实例化集合类时，可以指明具体的泛型类型
   ③ 指明完以后，在集合类或接口中凡是定义类或接口时，内部结构（比如：方法、构造器、属性等）使用到类的泛型的位置，都指定为实例化的泛型类型。
   比如：add(E e)  ---实例化以后：add(Integer e)
   ④ 注意点：泛型的类型必须是类，不能是基本数据类型。需要用到基本数据类型的位置，拿包装类替换
   ⑤ 如果实例化时，没指明泛型的类型。默认类型为java.lang.Object类型。
 ```

 

## 12.3 自定义泛型类、泛型接口、泛型方法

 1.举例:

【Order.java】

 ```java
 public class Order<T {
     String orderName;
     int orderId;
     //类的内部结构就可以使用类的泛型
     T orderT;
 
 public Order(){
     //编译不通过
     //  T[] arr = new T[10];
     //编译通过
     T[] arr = (T[]) new Object[10];
 }
     public Order(String orderName,int orderId,T orderT){
         this.orderName = orderName;
         this.orderId = orderId;
         this.orderT = orderT;
     }
     //如下的个方法都不是泛型方法
     public T getOrderT(){
         return orderT;
     }
     public void setOrderT(T orderT){
         this.orderT = orderT;
     }
     @Override
     public String toString() {
         return "Order{" +
             "orderName='" + orderName + '\'' +
             ", orderId=" + orderId +
             ", orderT=" + orderT +
             '}';
     }
     //静态方法中不能使用类的泛型。
     //public static void show(T orderT){
     // System.out.println(orderT);
 //}
     public void show(){
     //编译不通过
     //try{
 //     }catch(T t){
 //
 //   }
 }
 
 //泛型方法：在方法中出现了泛型的结构，泛型参数与类的泛型参数没任何关系。
 //换句话说，泛型方法所属的类是不是泛型类都没关系。
 //泛型方法，可以声明为静态的。原因：泛型参数是在调用方法时确定的。并非在实例化类时确定。
 public static <E  List<E copyFromArrayToList(E[] arr){
     ArrayList<E list = new ArrayList<();
     for(E e : arr){
         list.add(e);
     }
     return list;
 }
 }
 
 ```
 【SubOrder.java】

 ```java
 public class SubOrder extends Order<Integer {
     //SubOrder:不是泛型类
     public static <E List<E copyFromArrayToList(E[] arr){
         ArrayList<E list = new ArrayList<();
         for(E e : arr){
             list.add(e);
         }
         return list;
     }
  }
 //实例化时，如下的代码是错误的
 SubOrder<Integer o = new SubOrder<();
 ```


【SubOrder1.java】

 ```java
 public class SubOrder1<T extends Order<T {
     //SubOrder1<T:仍然是泛型类
 }
 ```

 【测试】


 ```java
 @Test
 public void test1(){
     //如果定义了泛型类，实例化没指明类的泛型，则认为此泛型类型为Object类型
     //要求：如果大家定义了类是带泛型的，建议在实例化时要指明类的泛型。
     Order order = new Order();
     order.setOrderT(123);
     order.setOrderT("ABC");
     //建议：实例化时指明类的泛型
     Order<String order1 = new Order<String("orderAA",1001,"order:AA");
     order1.setOrderT("AA:hello");
 }
 
 @Test
 public void test2(){
     SubOrder sub1 = new SubOrder();
     //由于子类在继承带泛型的父类时，指明了泛型类型。则实例化子类对象时，不再需要指明泛型。
     sub1.setOrderT(1122);
     SubOrder1<String sub2 = new SubOrder1<();
     sub2.setOrderT("order2...");
 }
 
 @Test
 public void test3(){
     ArrayList<String list1 = null;
     ArrayList<Integer list2 = new ArrayList<Integer();
     //泛型不同的引用不能相互赋值。
     //        list1 = list2;
      Person p1 = null;
     Person p2 = null;
     p1 = p2;  
  }
 
 //测试泛型方法
 @Test
 public void test4(){
     Order<String order = new Order<();
     Integer[] arr = new Integer[]{1,2,3,4};
     //泛型方法在调用时，指明泛型参数的类型。
     List<Integer list = order.copyFromArrayToList(arr);
     System.out.println(list);
 }
 ```


 2.注意点：

 ![image-20210222214929456](java-尚.assets/image-20210222214929456.png)

 ![image-20210222214941383](java-尚.assets/image-20210222214941383.png)

 3.应用场景举例：

 【DAO.java】:定义了操作数据库中的表的通用操作。   ORM思想(数据库中的表和Java中的类对应)


 ```java
 public class DAO<T {//表的共性操作的DAO
     //添加一条记录
     public void add(T t){}
     //删除一条记录
     public boolean remove(int index){
         return false;
 }
 
 //修改一条记录
 public void update(int index,T t){}
 
 //查询一条记录
 public T getIndex(int index){
 
     return null;
 }
 
 //查询多条记录
 public List<T getForList(int index){
 
     return null;
 }
 
 //泛型方法
 //举例：获取表中一共有多少条记录？获取最大的员工入职时间？
 public <E E getValue(){
 
     return null;
 }}
 ```

 【CustomerDAO.java】:

 ```java
 public class CustomerDAO extends DAO<Customer{//只能操作某一个表的DAO
 }
 ```

 

 【StudentDAO.java】:

 ```java
 public class StudentDAO extends DAO<Student {//只能操作某一个表的DAO
 }
 ```

 



## 12.4 泛型在继承上的体现

 泛型在继承上的体现:

 ```java
   /**
        1. 泛型在继承方面的体现
   虽然类A是类B的父类，但是G<A 和G<B二者不具备子父类关系，二者是并列关系。
 
    补充：类A是类B的父类，A<G 是 B<G 的父类
 */
 @Test
 public void test1(){
 
     Object obj = null;
     String str = null;
     obj = str;
 
     Object[] arr1 = null;
     String[] arr2 = null;
     arr1 = arr2;
     //编译不通过
     //        Date date = new Date();
 //        str = date;
         List<Object list1 = null;
         List<String list2 = new ArrayList<String();
         //此时的list1和list2的类型不具子父类关系
         //编译不通过
 //        list1 = list2;
         /
         反证法：
         假设list1 = list2;
            list1.add(123);导致混入非String的数据。出错。
            /
 
     show(list1);
     show1(list2);
 
 }   
 public void show1(List<String list){
 
 }
 
 public void show(List<Object list){
 
 }
 
 @Test
 public void test2(){
 
     AbstractList<String list1 = null;
     List<String list2 = null;
     ArrayList<String list3 = null;
 
     list1 = list3;
     list2 = list3;
 
     List<String list4 = new ArrayList<();
 
 }
 ```




## 12.5 通配符

 1.通配符的使用

 ```java
 /**
     通配符的使用
        通配符：?
 
        类A是类B的父类，G<A和G<B是没关系的，二者共同的父类是：G<?
     */
 
     @Test
     public void test3(){
         List<Object list1 = null;
         List<String list2 = null;
 
         List<? list = null;
 
         list = list1;
         list = list2;
         //编译通过
 //        print(list1);
 //        print(list2);
 
 
         //
         List<String list3 = new ArrayList<();
         list3.add("AA");
         list3.add("BB");
         list3.add("CC");
         list = list3;
         //添加(写入)：对于List<?就不能向其内部添加数据。
         //除了添加null之外。
 //        list.add("DD");
 //        list.add('?');
 
         list.add(null);
 
         //获取(读取)：允许读取数据，读取的数据类型为Object。
         Object o = list.get(0);
         System.out.println(o);
 
 
     }
 
     public void print(List<? list){
         Iterator<? iterator = list.iterator();
         while(iterator.hasNext()){
             Object obj = iterator.next();
             System.out.println(obj);
         }
     }
 ```

 2.涉及通配符的集合的数据的写入和读取:

 见上	

 3.有限制条件的通配符的使用

 ```java
 /
     限制条件的通配符的使用。
         ? extends A:
                 G<? extends A 可以作为G<A和G<B的父类，其中B是A的子类
 
         ? super A:
                 G<? super A 可以作为G<A和G<B的父类，其中B是A的父类
 
      /
     @Test
     public void test4(){
 
         List<? extends Person list1 = null;
         List<? super Person list2 = null;
 
         List<Student list3 = new ArrayList<Student();
         List<Person list4 = new ArrayList<Person();
         List<Object list5 = new ArrayList<Object();
 
         list1 = list3;
         list1 = list4;
 //        list1 = list5;
 
 //        list2 = list3;
         list2 = list4;
         list2 = list5;
 
         //读取数据：
         list1 = list3;
         Person p = list1.get(0);
         //编译不通过
         //Student s = list1.get(0);
 
         list2 = list4;
         Object obj = list2.get(0);
         ////编译不通过
 //        Person obj = list2.get(0);
 
         //写入数据：
         //编译不通过
 //        list1.add(new Student());
 
         //编译通过
         list2.add(new Person());
         list2.add(new Student());
 
     }	 
 
 ```

 

# 第13章 IO流

## 13.1 File类使用

 1.File类的理解
 ```sh
  1. File类的一个对象，代表一个文件或一个文件目录(俗称：文件夹)
  2. File类声明在java.io包下
  3. File类中涉及到关于文件或文件目录的创建、删除、重命名、修改时间、文件大小等方法，
  并未涉及到写入或读取文件内容的操作。如果需要读取或写入文件内容，必须使用IO流来完成。
  4. 后续File类的对象常会作为参数传递到流的构造器中，指明读取或写入的"终点".
 ```

 2.File的实例化

 2.1 常用构造器

 ```sh
 File(String filePath)
 File(String parentPath,String childPath)
 File(File parentFile,String childPath)
 ```

 2.2 路径的分类

 ```sh
 相对路径：相较于某个路径下，指明的路径。
 绝对路径：包含盘符在内的文件或文件目录的路径
 
 说明：
 IDEA中：
 如果大家开发使用JUnit中的单元测试方法测试，相对路径即为当前Module下。
 如果大家使用main()测试，相对路径即为当前的Project下。
 Eclipse中：
 不管使用单元测试方法还是使用main()测试，相对路径都是当前的Project下。
 ```

 

 2.3 路径分隔符

 ```sh
 windows和DOS系统默认使用“\”来表示
 UNIX和URL使用“/”来表示
 ```

 

 3.File类的常用方法

 ![image-20210222215605904](java-尚.assets/image-20210222215605904.png)

 ![image-20210222215618919](java-尚.assets/image-20210222215618919.png)

 ![image-20210222215628351](java-尚.assets/image-20210222215628351.png)



## 13.2 IO流概述

 1.流的分类

 ```sh
  1.操作数据单位：字节流、字符流
  2.数据的流向：输入流、输出流
  3.流的角色：节点流、处理流
 ```

 图示：

 ![image-20210222215829167](java-尚.assets/image-20210222215829167.png)

 2.流的体系结构

 ![image-20210222215847230](java-尚.assets/image-20210222215847230.png)

 说明：红框对应的是IO流中的4个抽象基类。

 蓝框的流需要大家重点关注。

 3.重点说明的几个流结构

 ![image-20210222215924949](java-尚.assets/image-20210222215924949.png)

 4.输入、输出的标准化过程

 4.1 输入过程

 ```sh
 ① 创建File类的对象，指明读取的数据的来源。（要求此文件一定要存在）
 ② 创建相应的输入流，将File类的对象作为参数，传入流的构造器中
 ③ 具体的读入过程：
     创建相应的byte[] 或 char[]。
 ④ 关闭流资源
 说明：程序中出现的异常需要使用try-catch-finally处理。
 ```

 4.2 输出过程

 ```sh
 ① 创建File类的对象，指明写出的数据的位置。（不要求此文件一定要存在）
 ② 创建相应的输出流，将File类的对象作为参数，传入流的构造器中
 ③ 具体的写出过程：
     write(char[]/byte[] buffer,0,len)
 ④ 关闭流资源
 说明：程序中出现的异常需要使用try-catch-finally处理。
 ```

 



## 13.3 节点流（或文件流）

 1.FileReader/FileWriter的使用：

 1.1 FileReader的使用

 ```java
 /**
 将day09下的hello.txt文件内容读入程序中，并输出到控制台
 
 说明点：
 1. read()的理解：返回读入的一个字符。如果达到文件末尾，返回-1
 2. 异常的处理：为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
 3. 读入的文件一定要存在，否则就会报FileNotFoundException。
 
  */
 @Test
     public void testFileReader1()  {
         FileReader fr = null;
         try {
             //1.File类的实例化
             File file = new File("hello.txt");
 
             //2.FileReader流的实例化
             fr = new FileReader(file);
 
             //3.读入的操作
             //read(char[] cbuf):返回每次读入cbuf数组中的字符的个数。如果达到文件末尾，返回-1
             char[] cbuf = new char[5];
             int len;
             while((len = fr.read(cbuf)) != -1){
                 //方式一：
                 //错误的写法
 //                for(int i = 0;i < cbuf.length;i++){
 //                    System.out.print(cbuf[i]);
 //                }
                 //正确的写法
 //                for(int i = 0;i < len;i++){
 //                    System.out.print(cbuf[i]);
 //                }
                 //方式二：
                 //错误的写法,对应着方式一的错误的写法
 //                String str = new String(cbuf);
 //                System.out.print(str);
                 //正确的写法
                 String str = new String(cbuf,0,len);
                 System.out.print(str);
             }
         } catch (IOException e) {
             e.printStackTrace();
         } finally {
             if(fr != null){
                 //4.资源的关闭
                 try {
                     fr.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
 
             }
         }
 
     }
 
 ```

 1.2 FileWriter的使用

 ```java
 /**
 从内存中写出数据到硬盘的文件里。
 
 说明：
 1. 输出操作，对应的File可以不存在的。并不会报异常
 2.
      File对应的硬盘中的文件如果不存在，在输出的过程中，会自动创建此文件。
      File对应的硬盘中的文件如果存在：
            如果流使用的构造器是：FileWriter(file,false) / FileWriter(file):对原文件的覆盖
            如果流使用的构造器是：FileWriter(file,true):不会对原文件覆盖，而是在原文件基础上追加内容
 
  */
 @Test
 public void testFileWriter() {
     FileWriter fw = null;
     try {
         //1.提供File类的对象，指明写出到的文件
         File file = new File("hello1.txt");
 
         //2.提供FileWriter的对象，用于数据的写出
         fw = new FileWriter(file,false);
 
         //3.写出的操作
         fw.write("I have a dream!\n");
         fw.write("you need to have a dream!");
     } catch (IOException e) {
         e.printStackTrace();
     } finally {
         //4.流资源的关闭
         if(fw != null){
 
             try {
                 fw.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
         }
     }
 }
 ```

 1.3 文本文件的复制：

 ```java
 @Test
     public void testFileReaderFileWriter() {
         FileReader fr = null;
         FileWriter fw = null;
         try {
             //1.创建File类的对象，指明读入和写出的文件
             File srcFile = new File("hello.txt");
             File destFile = new File("hello2.txt");
 
             //不能使用字符流来处理图片等字节数据
 //            File srcFile = new File("爱情与友情.jpg");
 //            File destFile = new File("爱情与友情1.jpg");
 
 
             //2.创建输入流和输出流的对象
              fr = new FileReader(srcFile);
             fw = new FileWriter(destFile);
 
 
             //3.数据的读入和写出操作
             char[] cbuf = new char[5];
             int len;//记录每次读入到cbuf数组中的字符的个数
             while((len = fr.read(cbuf)) != -1){
                 //每次写出len个字符
                 fw.write(cbuf,0,len);
 
             }
         } catch (IOException e) {
             e.printStackTrace();
         } finally {
             //4.关闭流资源
             //方式一：
 //            try {
 //                if(fw != null)
 //                    fw.close();
 //            } catch (IOException e) {
 //                e.printStackTrace();
 //            }finally{
 //                try {
 //                    if(fr != null)
 //                        fr.close();
 //                } catch (IOException e) {
 //                    e.printStackTrace();
 //                }
 //            }
             //方式二：
             try {
                 if(fw != null)
                     fw.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
             try {
                 if(fr != null)
                     fr.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
         }
 
     }
 
 ```

 2.FileInputStream / FileOutputStream的使用：

 ```java
  1. 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
  2. 对于非文本文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)，使用字节流处理
 /
 实现对图片的复制操作
  /
 @Test
 public void testFileInputOutputStream()  {
     FileInputStream fis = null;
     FileOutputStream fos = null;
     try {
         //1.造文件
         File srcFile = new File("爱情与友情.jpg");
         File destFile = new File("爱情与友情2.jpg");
 
         //2.造流
         fis = new FileInputStream(srcFile);
         fos = new FileOutputStream(destFile);
 
         //3.复制的过程
         byte[] buffer = new byte[5];
         int len;
         while((len = fis.read(buffer)) != -1){
             fos.write(buffer,0,len);
         }
 
     } catch (IOException e) {
         e.printStackTrace();
     } finally {
         if(fos != null){
             //4.关闭流
             try {
                 fos.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
         }
         if(fis != null){
             try {
                 fis.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
         }
     }
 
 }
 
 ```

## 13.4 缓冲流

 1.缓冲流涉及到的类：

 ```properties
  BufferedInputStream
  BufferedOutputStream
  BufferedReader
  BufferedWriter
 ```

 2.作用：

 作用：提供流的读取、写入的速度,提高读写速度的原因：内部提供了一个缓冲区。默认情况下是8kb

 ![image-20210224211049425](java-尚.assets/image-20210224211049425.png)

 3.典型代码

 3.1 使用BufferedInputStream和BufferedOutputStream:处理非文本文件

 ```java
 //实现文件复制的方法
     public void copyFileWithBuffered(String srcPath,String destPath){
         BufferedInputStream bis = null;
         BufferedOutputStream bos = null;
 
         try {
             //1.造文件
             File srcFile = new File(srcPath);
             File destFile = new File(destPath);
             //2.造流
             //2.1 造节点流
             FileInputStream fis = new FileInputStream((srcFile));
             FileOutputStream fos = new FileOutputStream(destFile);
             //2.2 造缓冲流
             bis = new BufferedInputStream(fis);
             bos = new BufferedOutputStream(fos);
 
             //3.复制的细节：读取、写入
             byte[] buffer = new byte[1024];
             int len;
             while((len = bis.read(buffer)) != -1){
                 bos.write(buffer,0,len);
             }
         } catch (IOException e) {
             e.printStackTrace();
         } finally {
             //4.资源关闭
             //要求：先关闭外层的流，再关闭内层的流
             if(bos != null){
                 try {
                     bos.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
 
             }
             if(bis != null){
                 try {
                     bis.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
 
             }
             //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
 //        fos.close();
 //        fis.close();
         }
     }
 ```

 3.2 使用BufferedReader和BufferedWriter：处理文本文件

 ```java
 @Test
     public void testBufferedReaderBufferedWriter(){
         BufferedReader br = null;
         BufferedWriter bw = null;
         try {
             //创建文件和相应的流
             br = new BufferedReader(new FileReader(new File("dbcp.txt")));
             bw = new BufferedWriter(new FileWriter(new File("dbcp1.txt")));
 
             //读写操作
             //方式一：使用char[]数组
 //            char[] cbuf = new char[1024];
 //            int len;
 //            while((len = br.read(cbuf)) != -1){
 //                bw.write(cbuf,0,len);
 //    //            bw.flush();
 //            }
 
             //方式二：使用String
             String data;
             while((data = br.readLine()) != null){
                 //方法一：
 //                bw.write(data + "\n");//data中不包含换行符
                 //方法二：
                 bw.write(data);//data中不包含换行符
                 bw.newLine();//提供换行的操作
 
             }
 
 
         } catch (IOException e) {
             e.printStackTrace();
         } finally {
             //关闭资源
             if(bw != null){
 
                 try {
                     bw.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
             if(br != null){
                 try {
                     br.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
 
             }
         }
 
     }
 
 ```

 

## 13.5 转换流的使用

 1.转换流涉及到的类：属于字符流

 ```sh
 InputStreamReader：将一个字节的输入流转换为字符的输入流
 解码：字节、字节数组  ---字符数组、字符串
 
 OutputStreamWriter：将一个字符的输出流转换为字节的输出流
 编码：字符数组、字符串 --- 字节、字节数组
 
 说明：编码决定了解码的方式
 ```

 2.作用：提供字节流与字符流之间的转换

 3.图示：

  ![image-20210224211253494](java-尚.assets/image-20210224211253494.png)

 4.典型实现：

 ```java
 @Test
     public void test1() throws IOException {
 
         FileInputStream fis = new FileInputStream("dbcp.txt");
 //        InputStreamReader isr = new InputStreamReader(fis);//使用系统默认的字符集
         //参数2指明了字符集，具体使用哪个字符集，取决于文件dbcp.txt保存时使用的字符集
         InputStreamReader isr = new InputStreamReader(fis,"UTF-8");//使用系统默认的字符集
 
         char[] cbuf = new char[20];
         int len;
         while((len = isr.read(cbuf)) != -1){
             String str = new String(cbuf,0,len);
             System.out.print(str);
         }
 
         isr.close();
 
     }
 
 /**
 此时处理异常的话，仍然应该使用try-catch-finally
 综合使用InputStreamReader和OutputStreamWriter
  */
 @Test
 public void test2() throws Exception {
     //1.造文件、造流
     File file1 = new File("dbcp.txt");
     File file2 = new File("dbcp_gbk.txt");
 
     FileInputStream fis = new FileInputStream(file1);
     FileOutputStream fos = new FileOutputStream(file2);
 
     InputStreamReader isr = new InputStreamReader(fis,"utf-8");
     OutputStreamWriter osw = new OutputStreamWriter(fos,"gbk");
 
     //2.读写过程
     char[] cbuf = new char[20];
     int len;
     while((len = isr.read(cbuf)) != -1){
         osw.write(cbuf,0,len);
     }
 
     //3.关闭资源
     isr.close();
     osw.close();
 }
 ```

 5.说明：

 //文件编码的方式（比如：GBK），决定了解析时使用的字符集（也只能是GBK）。

 

### 13.5.1 编码集

 1.常见的编码表

 ```properties
 ASCII：美国标准信息交换码。
 用一个字节的7位可以表示。
 ISO8859-1：拉丁码表。欧洲码表
 用一个字节的8位表示。
 GB2312：中国的中文编码表。最多两个字节编码所有字符
 GBK：中国的中文编码表升级，融合了更多的中文文字符号。最多两个字节编码
 Unicode：国际标准码，融合了目前人类使用的所字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示。
 UTF-8：变长的编码方式，可用1-4个字节来表示一个字符。
 ```

 

 2.对后面学习的启示

 客户端/浏览器端    <----  后台(java,GO,Python,Node.js,php)   <---- 数据库

 要求前前后后使用的字符集都要统一：UTF-8.

## 13.6 其他的流的使用

 标准的输入输出流：

 ```sh
 System.in:标准的输入流，默认从键盘输入
 System.out:标准的输出流，默认从控制台输出
 
 修改默认的输入和输出行为：
 System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入和输出的流。
 
 ```

 打印流：

 ```sh
 PrintStream 和PrintWriter
 说明：
 提供了一系列重载的print()和println()方法，用于多种数据类型的输出
 System.out返回的是PrintStream的实例
 ```

  数据流：

 DataInputStream 和 DataOutputStream

 作用：用于读取或写出基本数据类型的变量或字符串

 示例代码：

 ```java
 /**
 练习：将内存中的字符串、基本数据类型的变量写出到文件中。
 
 注意：处理异常的话，仍然应该使用try-catch-finally.
  */
 @Test
 public void test3() throws IOException {
     //1.
     DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.txt"));
     //2.
     dos.writeUTF("刘建辰");
     dos.flush();//刷新操作，将内存中的数据写入文件
     dos.writeInt(23);
     dos.flush();
     dos.writeBoolean(true);
     dos.flush();
     //3.
     dos.close();
 
 
 }
 /
 将文件中存储的基本数据类型变量和字符串读取到内存中，保存在变量中。
 
 注意点：读取不同类型的数据的顺序要与当初写入文件时，保存的数据的顺序一致！
 
  /
 @Test
 public void test4() throws IOException {
     //1.
     DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));
     //2.
     String name = dis.readUTF();
     int age = dis.readInt();
     boolean isMale = dis.readBoolean();
 
     System.out.println("name = " + name);
     System.out.println("age = " + age);
     System.out.println("isMale = " + isMale);
 
     //3.
     dis.close();
 
 }
 
 ```

 

## 13.7 对象流的使用

 1.对象流： 

 ObjectInputStream 和 ObjectOutputStream

 2.作用：

 ObjectOutputStream:内存中的对象---存储中的文件、通过网络传输出去：序列化过程

 ObjectInputStream:存储中的文件、通过网络接收过来 ---内存中的对象：反序列化过程

 3.对象的序列化机制：

 对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。//当其它程序获取了这种二进制流，就可以恢复成原来的Java对象

 4.序列化代码实现：

 ```java
 @Test
 public void testObjectOutputStream(){
     ObjectOutputStream oos = null;
 
     try {
         //1.
         oos = new ObjectOutputStream(new FileOutputStream("object.dat"));
         //2.
         oos.writeObject(new String("我爱北京天安门"));
         oos.flush();//刷新操作
 
          oos.writeObject(new Person("王铭",23));
         oos.flush();
 
         oos.writeObject(new Person("张学良",23,1001,new Account(5000)));
         oos.flush();
 
     } catch (IOException e) {
         e.printStackTrace();
     } finally {
         if(oos != null){
             //3.
             try {
                 oos.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
         }
     }
 
 }
 ```

 5.反序列化代码实现：

 ```java
 @Test
 public void testObjectInputStream(){
     ObjectInputStream ois = null;
     try {
         ois = new ObjectInputStream(new FileInputStream("object.dat"));
 
         Object obj = ois.readObject();
         String str = (String) obj;
 
         Person p = (Person) ois.readObject();
         Person p1 = (Person) ois.readObject();
 
         System.out.println(str);
         System.out.println(p);
         System.out.println(p1);
 
     } catch (IOException e) {
         e.printStackTrace();
     } catch (ClassNotFoundException e) {
         e.printStackTrace();
     } finally {
         if(ois != null){
             try {
                 ois.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
         }
     }
 
 
 
 }
 ```

 6.实现序列化的对象所属的类需要满足：

1.需要实现接口：Serializable

2.当前类提供一个全局常量：serialVersionUID

3.除了当前Person类需要实现Serializable接口之外，还必须保证其内部所属性也必须是可序列化的。（默认情况下，基本数据类型可序列化）

补充：ObjectOutputStream和ObjectInputStream不能序列化static和transient修饰的成员变量


## 13.8 RandomAccessFile的使用

 1.随机存取文件流：RandomAccessFile

 2.使用说明：

 ```sh
  1.RandomAccessFile直接继承于java.lang.Object类，实现了DataInput和DataOutput接口
  2.RandomAccessFile既可以作为一个输入流，又可以作为一个输出流
 
  3.如果RandomAccessFile作为输出流时，写出到的文件如果不存在，则在执行过程中自动创建。
    如果写出到的文件存在，则会对原文件内容进行覆盖。（默认情况下，从头覆盖）
 
  4. 可以通过相关的操作，实现RandomAccessFile“插入”数据的效果。seek(int pos)
 ```

 3.典型代码1：

 ```java
 @Test
 public void test1() {
 
     RandomAccessFile raf1 = null;
     RandomAccessFile raf2 = null;
     try {
         //1.
         raf1 = new RandomAccessFile(new File("爱情与友情.jpg"),"r");
         raf2 = new RandomAccessFile(new File("爱情与友情1.jpg"),"rw");
         //2.
         byte[] buffer = new byte[1024];
         int len;
         while((len = raf1.read(buffer)) != -1){
             raf2.write(buffer,0,len);
         }
     } catch (IOException e) {
         e.printStackTrace();
     } finally {
         //3.
         if(raf1 != null){
             try {
                 raf1.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
         }
         if(raf2 != null){
             try {
                 raf2.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
 
         }
     }
 }
 
 ```

 典型代码2：

 ```java
 /**
 使用RandomAccessFile实现数据的插入效果
  */
 @Test
 public void test3() throws IOException {
 
     RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");
 
     raf1.seek(3);//将指针调到角标为3的位置
     //保存指针3后面的所数据到StringBuilder中
     StringBuilder builder = new StringBuilder((int) new File("hello.txt").length());
     byte[] buffer = new byte[20];
     int len;
     while((len = raf1.read(buffer)) != -1){
         builder.append(new String(buffer,0,len)) ;
     }
     //调回指针，写入“xyz”
     raf1.seek(3);
     raf1.write("xyz".getBytes());
 
     //将StringBuilder中的数据写入到文件中
     raf1.write(builder.toString().getBytes());
 
     raf1.close();
 
     //思考：将StringBuilder替换为ByteArrayOutputStream
 }
 
 ```

 

## 13.9 Path、Paths、Files的使用

 1.NIO的使用说明：

 ```sh
 Java NIO (New IO，Non-Blocking IO)是从Java 1.4版本开始引入的一套新的IO API，可以替代标准的Java 
 IO AP。
 NIO与原来的IO同样的作用和目的，但是使用的方式完全不同，NIO支持面向缓冲区的(IO是面向流的)、基于
 通道的IO操作。
 NIO将以更加高效的方式进行文件的读写操作。
 随着 JDK 7 的发布，Java对NIO进行了极大的扩展，增强了对文件处理和文件系统特性的支持，以至于我们称他们为 NIO.2。
 ```

 2.Path的使用 ---jdk7提供

 2.1Path的说明：

 Path替换原有的File类。

 2.2如何实例化：

  ![image-20210224212633676](java-尚.assets/image-20210224212633676.png)

 2.3常用方法：

 ![image-20210224212701972](java-尚.assets/image-20210224212701972.png)

 3.Files工具类 ---jdk7提供

 3.1作用：

 操作文件或文件目录的工具类

 3.2常用方法：

 ![image-20210224212726361](java-尚.assets/image-20210224212726361.png)

 ![image-20210224212734929](java-尚.assets/image-20210224212734929.png)



# 第14章 网络编程

## 14.1 InetAdress类的使用

 一、实现网络通信需要解决的两个问题

 ```sh
  1.如何准确地定位网络上一台或多台主机；定位主机上的特定的应用
  2.找到主机后如何可靠高效地进行数据传输
 ```

 二、网络通信的两个要素：

 ```sh
  1.对应问题一：IP和端口号
  2.对应问题二：提供网络通信协议：TCP/IP参考模型（应用层、传输层、网络层、物理+数据链路层）
 
 ```

 三、通信要素一：IP和端口号

 ```sh
 1.IP的理解
  1. IP:唯一的标识 Internet 上的计算机（通信实体）
  2. 在Java中使用InetAddress类代表IP
  3. IP分类：IPv4 和 IPv6 ; 万维网 和 局域网
  4. 域名:   www.baidu.com   www.mi.com  www.sina.com  www.jd.com
            
      域名解析：域名容易记忆，当在连接网络时输入一个主机的域名后，域名服务器(DNS)负责将域名转化成IP地址，这样才能和主机建立连接。 -------域名解析
  5. 本地回路地址：127.0.0.1 对应着：localhost
 
 2.InetAddress类:此类的一个对象就代表着一个具体的IP地址
 2.1实例化
 getByName(String host) 、 getLocalHost()
 
 2.2常用方法
 getHostName() / getHostAddress()
 
 3.端口号：正在计算机上运行的进程。
  要求：不同的进程不同的端口号
  范围：被规定为一个 16 位的整数 0~65535。
 
 端口号与IP地址的组合得出一个网络套接字：Socket
 ```

 四、通信要素二：网络通信协议

 1.分型模型

 ![image-20210224213119810](java-尚.assets/image-20210224213119810.png)

 2.TCP和UDP的区别

  ![image-20210224213213705](java-尚.assets/image-20210224213213705.png)

 3.TCP三次握手和四次挥手

 ![image-20210224213328153](java-尚.assets/image-20210224213328153.png)

 ![image-20210224213336993](java-尚.assets/image-20210224213336993.png)



## 14.2 TCP编程模型

 代码示例1：客户端发送信息给服务端，服务端将数据显示在控制台上

 ```java
 //客户端
     @Test
     public void client()  {
         Socket socket = null;
         OutputStream os = null;
         try {
             //1.创建Socket对象，指明服务器端的ip和端口号
             InetAddress inet = InetAddress.getByName("192.168.14.100");
             socket = new Socket(inet,8899);
             //2.获取一个输出流，用于输出数据
             os = socket.getOutputStream();
             //3.写出数据的操作
             os.write("你好，我是客户端mm".getBytes());
         } catch (IOException e) {
             e.printStackTrace();
         } finally {
             //4.资源的关闭
             if(os != null){
                 try {
                     os.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
 
             }
             if(socket != null){
                 try {
                     socket.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
 
             }
         }
 
 
 
     }
     //服务端
     @Test
     public void server()  {
 
         ServerSocket ss = null;
         Socket socket = null;
         InputStream is = null;
         ByteArrayOutputStream baos = null;
         try {
             //1.创建服务器端的ServerSocket，指明自己的端口号
             ss = new ServerSocket(8899);
             //2.调用accept()表示接收来自于客户端的socket
             socket = ss.accept();
             //3.获取输入流
             is = socket.getInputStream();
 
             //不建议这样写，可能会乱码
 //        byte[] buffer = new byte[1024];
 //        int len;
 //        while((len = is.read(buffer)) != -1){
 //            String str = new String(buffer,0,len);
 //            System.out.print(str);
 //        }
             //4.读取输入流中的数据
             baos = new ByteArrayOutputStream();
             byte[] buffer = new byte[5];
             int len;
             while((len = is.read(buffer)) != -1){
                 baos.write(buffer,0,len);
             }
 
             System.out.println(baos.toString());
 
             System.out.println("收到了来自于：" + socket.getInetAddress().getHostAddress() + "的数据");
 
         } catch (IOException e) {
             e.printStackTrace();
         } finally {
             if(baos != null){
                 //5.关闭资源
                 try {
                     baos.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
             if(is != null){
                 try {
                     is.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
             if(socket != null){
                 try {
                     socket.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
             if(ss != null){
                 try {
                     ss.close();
                 } catch (IOException e) {
                     e.printStackTrace();
                 }
             }
 
         }
 
     }
 
 ```

 代码示例2：客户端发送文件给服务端，服务端将文件保存在本地。

 ```java
 /**
 这里涉及到的异常，应该使用try-catch-finally处理
  */
 @Test
 public void client() throws IOException {
     //1.
     Socket socket = new Socket(InetAddress.getByName("127.0.0.1"),9090);
     //2.
     OutputStream os = socket.getOutputStream();
     //3.
     FileInputStream fis = new FileInputStream(new File("beauty.jpg"));
     //4.
     byte[] buffer = new byte[1024];
     int len;
     while((len = fis.read(buffer)) != -1){
         os.write(buffer,0,len);
     }
     //5.
     fis.close();
     os.close();
     socket.close();
 }
 
 /
 这里涉及到的异常，应该使用try-catch-finally处理
  /
 @Test
 public void server() throws IOException {
     //1.
     ServerSocket ss = new ServerSocket(9090);
     //2.
     Socket socket = ss.accept();
     //3.
     InputStream is = socket.getInputStream();
     //4.
     FileOutputStream fos = new FileOutputStream(new File("beauty1.jpg"));
     //5.
     byte[] buffer = new byte[1024];
     int len;
     while((len = is.read(buffer)) != -1){
         fos.write(buffer,0,len);
     }
     //6.
     fos.close();
     is.close();
     socket.close();
     ss.close();
 
 }
 ```

 代码示例3：从客户端发送文件给服务端，服务端保存到本地。并返回“发送成功”给客户端。并关闭相应的连接。

 ```java
 /**
     这里涉及到的异常，应该使用try-catch-finally处理
     */
 @Test
 public void client() throws IOException {
     //1.
     Socket socket = new Socket(InetAddress.getByName("127.0.0.1"),9090);
     //2.
     OutputStream os = socket.getOutputStream();
     //3.
     FileInputStream fis = new FileInputStream(new File("beauty.jpg"));
     //4.
     byte[] buffer = new byte[1024];
     int len;
     while((len = fis.read(buffer)) != -1){
         os.write(buffer,0,len);
     }
     //关闭数据的输出
     socket.shutdownOutput();
 
     //5.接收来自于服务器端的数据，并显示到控制台上
     InputStream is = socket.getInputStream();
     ByteArrayOutputStream baos = new ByteArrayOutputStream();
     byte[] bufferr = new byte[20];
     int len1;
     while((len1 = is.read(buffer)) != -1){
         baos.write(buffer,0,len1);
     }
 
     System.out.println(baos.toString());
 
     //6.
     fis.close();
     os.close();
     socket.close();
     baos.close();
 }
 
 /**
 这里涉及到的异常，应该使用try-catch-finally处理
  */
 @Test
 public void server() throws IOException {
     //1.
     ServerSocket ss = new ServerSocket(9090);
     //2.
     Socket socket = ss.accept();
     //3.
     InputStream is = socket.getInputStream();
     //4.
     FileOutputStream fos = new FileOutputStream(new File("beauty2.jpg"));
     //5.
     byte[] buffer = new byte[1024];
     int len;
     while((len = is.read(buffer)) != -1){
         fos.write(buffer,0,len);
     }
 
     System.out.println("图片传输完成");
 
     //6.服务器端给予客户端反馈
     OutputStream os = socket.getOutputStream();
     os.write("你好，美女，照片我已收到，非常漂亮！".getBytes());
 
     //7.
     fos.close();
     is.close();
     socket.close();
     ss.close();
     os.close();
 
 }
 
 ```

 



## 14.3 UDP网络编程

 代码示例：

 ```java
 //发送端
 @Test
 public void sender() throws IOException {
 
     DatagramSocket socket = new DatagramSocket();
 
 
 
     String str = "我是UDP方式发送的导弹";
     byte[] data = str.getBytes();
     InetAddress inet = InetAddress.getLocalHost();
     DatagramPacket packet = new DatagramPacket(data,0,data.length,inet,9090);
 
     socket.send(packet);
 
     socket.close();
 
 }
 //接收端
 @Test
 public void receiver() throws IOException {
 
     DatagramSocket socket = new DatagramSocket(9090);
 
     byte[] buffer = new byte[100];
     DatagramPacket packet = new DatagramPacket(buffer,0,buffer.length);
 
     socket.receive(packet);
 
     System.out.println(new String(packet.getData(),0,packet.getLength()));
 
     socket.close();
 }
 
 
 ```

 

## 14.4 URL编程

 1.URL(Uniform Resource Locator)的理解:

 统一资源定位符，对应着互联网的某一资源地址

 2.URL的5个基本结构：
 ```sh
  http://localhost:8080/examples/beauty.jpg?username=Tom
 
  协议   主机名    端口号  资源地址           参数列表
 ```

 3.如何实例化:

URL url = new URL("http://localhost:8080/examples/beauty.jpg?username=Tom");

 4.常用方法：

  ![image-20210224213809190](java-尚.assets/image-20210224213809190.png)

 5.可以读取、下载对应的url资源：

 ```java
 public static void main(String[] args) {
 
     HttpURLConnection urlConnection = null;
     InputStream is = null;
     FileOutputStream fos = null;
     try {
         URL url = new URL("http://localhost:8080/examples/beauty.jpg");
 
         urlConnection = (HttpURLConnection) url.openConnection();
 
         urlConnection.connect();
 
         is = urlConnection.getInputStream();
         fos = new FileOutputStream("day10\\beauty3.jpg");
 
         byte[] buffer = new byte[1024];
         int len;
         while((len = is.read(buffer)) != -1){
             fos.write(buffer,0,len);
         }
 
         System.out.println("下载完成");
     } catch (IOException e) {
         e.printStackTrace();
     } finally {
         //关闭资源
         if(is != null){
             try {
                 is.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
         }
         if(fos != null){
             try {
                 fos.close();
             } catch (IOException e) {
                 e.printStackTrace();
             }
         }
         if(urlConnection != null){
             urlConnection.disconnect();
         }
     }
 }
 
 
 ```

 



# 第15章 Java的反射机制

## 15.1 反射概述

 1.本章的主要内容

  ![image-20210224214007399](java-尚.assets/image-20210224214007399.png)

 2.关于反射的理解

  Reflection（反射)是被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。

  框架 = 反射 + 注解 + 设计模式。

 3.体会反射机制的“动态性”

 ```java
 //体会反射的动态性
 @Test
 public void test2(){
 
     for(int i = 0;i < 100;i++){
         int num = new Random().nextInt(3);//0,1,2
         String classPath = "";
         switch(num){
             case 0:
                 classPath = "java.util.Date";
                 break;
             case 1:
                 classPath = "java.lang.Object";
                 break;
             case 2:
                 classPath = "com.atguigu.java.Person";
                 break;
         }
 
         try {
             Object obj = getInstance(classPath);
             System.out.println(obj);
         } catch (Exception e) {
             e.printStackTrace();
         }
     }
 
 
 
 }
 
 /**
 创建一个指定类的对象。
 classPath:指定类的全类名
  */
 public Object getInstance(String classPath) throws Exception {
    Class clazz =  Class.forName(classPath);
    return clazz.newInstance();
 }
 
 ```

 4.反射机制能提供的功能

  ![image-20210224214110904](java-尚.assets/image-20210224214110904.png)

 5.相关API

 java.lang.Class:反射的源头

 java.lang.reflect.Method

 java.lang.reflect.Field

 java.lang.reflect.Constructor

 ....

## 15.2 Class类的理解与获取Class的实例

 1.Class类的理解

 ```sh
 1.类的加载过程：
 程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)。
 接着我们使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件
 加载到内存中。此过程就称为类的加载。加载到内存中的类，我们就称为运行时类，此
 运行时类，就作为Class的一个实例。
 2.换句话说，Class的实例就对应着一个运行时类。
 3.加载到内存中的运行时类，会缓存一定的时间。在此时间之内，我们可以通过不同的方式
 来获取此运行时类。
 ```

 2.获取Class实例的几种方式：（前三种方式需要掌握）

 ```java
 	//方式一：调用运行时类的属性：.class
         Class clazz1 = Person.class;
         System.out.println(clazz1);
         //方式二：通过运行时类的对象,调用getClass()
         Person p1 = new Person();
         Class clazz2 = p1.getClass();
         System.out.println(clazz2);
 
         //方式三：调用Class的静态方法：forName(String classPath)
         Class clazz3 = Class.forName("com.atguigu.java.Person");
 //        clazz3 = Class.forName("java.lang.String");
         System.out.println(clazz3);
 
         System.out.println(clazz1 == clazz2);
         System.out.println(clazz1 == clazz3);
 
         //方式四：使用类的加载器：ClassLoader  (了解)
         ClassLoader classLoader = ReflectionTest.class.getClassLoader();
         Class clazz4 = classLoader.loadClass("com.atguigu.java.Person");
         System.out.println(clazz4);
 
         System.out.println(clazz1 == clazz4);
 ```

 3.总结：创建类的对象的方式?

 方式一：new + 构造器

 方式二：要创建Xxx类的对象，可以考虑：Xxx、Xxxs、XxxFactory、XxxBuilder类中查看是否有

 静态方法的存在。可以调用其静态方法，创建Xxx对象。

 方式三：通过反射

 4.Class实例可以是哪些结构的说明

 ![image-20210224214404616](java-尚.assets/image-20210224214404616.png)

## 15.3 了解ClassLoader

 1.类的加载过程----了解

 ![image-20210224214453791](java-尚.assets/image-20210224214453791.png)

 2.类的加载器的作用

 ![image-20210224214510295](java-尚.assets/image-20210224214510295.png)

 3.类的加载器的分类

 ![image-20210224214526344](java-尚.assets/image-20210224214526344.png)

 4.Java类编译、运行的执行的流程

 ![image-20210224214546639](java-尚.assets/image-20210224214546639.png)

 5.使用Classloader加载src目录下的配置文件

 ```java
 @Test
     public void test2() throws Exception {
 
         Properties pros =  new Properties();
         //此时的文件默认在当前的module下。
         //读取配置文件的方式一：
 //        FileInputStream fis = new FileInputStream("jdbc.properties");
 //        FileInputStream fis = new FileInputStream("src\\jdbc1.properties");
 //        pros.load(fis);
 
         //读取配置文件的方式二：使用ClassLoader
         //配置文件默认识别为：当前module的src下
         ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
         InputStream is = classLoader.getResourceAsStream("jdbc1.properties");
         pros.load(is);
 
         String user = pros.getProperty("user");
         String password = pros.getProperty("password");
         System.out.println("user = " + user + ",password = " + password);
 
 
 
     }
 
 ```

 

## 15.4 反射应用一：创建运行时类的对象

 1.代码举例

 ```java
 Class<Person clazz = Person.class;
 
 Person obj = clazz.newInstance();
 System.out.println(obj);
 ```

 2.说明

 ```properties
 newInstance():调用此方法，创建对应的运行时类的对象。内部调用了运行时类的空参的构造器。
 
 要想此方法正常的创建运行时类的对象，要求：
 1.运行时类必须提供空参的构造器
 2.空参的构造器的访问权限得够。通常，设置为public。
 
 
 在javabean中要求提供一个public的空参构造器。原因：
 1.便于通过反射，创建运行时类的对象
 2.便于子类继承此运行时类时，默认调用super()时，保证父类此构造器
 ```

 

## 15.5 反射应用二：获取运行时类的完整结构

 我们可以通过反射，获取对应的运行时类中所有的属性、方法、构造器、父类、接口、父类的泛型、包、注解、异常等。。。。

典型代码：

 ```java
 @Test
 public void test1(){
 
     Class clazz = Person.class;
 
     //获取属性结构
     //getFields():获取当前运行时类及其父类中声明为public访问权限的属性
     Field[] fields = clazz.getFields();
     for(Field f : fields){
         System.out.println(f);
     }
     System.out.println();
 
     //getDeclaredFields():获取当前运行时类中声明的所属性。（不包含父类中声明的属性
     Field[] declaredFields = clazz.getDeclaredFields();
     for(Field f : declaredFields){
         System.out.println(f);
     }
 }
 
 @Test
 public void test1(){
 
     Class clazz = Person.class;
 
     //getMethods():获取当前运行时类及其所父类中声明为public权限的方法
     Method[] methods = clazz.getMethods();
     for(Method m : methods){
         System.out.println(m);
     }
     System.out.println();
     //getDeclaredMethods():获取当前运行时类中声明的所方法。（不包含父类中声明的方法
     Method[] declaredMethods = clazz.getDeclaredMethods();
     for(Method m : declaredMethods){
         System.out.println(m);
     }
 }
 
 /**
     获取构造器结构
 
      */
     @Test
     public void test1(){
 
         Class clazz = Person.class;
         //getConstructors():获取当前运行时类中声明为public的构造器
         Constructor[] constructors = clazz.getConstructors();
         for(Constructor c : constructors){
             System.out.println(c);
         }
 
         System.out.println();
         //getDeclaredConstructors():获取当前运行时类中声明的所的构造器
         Constructor[] declaredConstructors = clazz.getDeclaredConstructors();
         for(Constructor c : declaredConstructors){
             System.out.println(c);
         }
 
     }
 
     /**
     获取运行时类的父类
 
      */
     @Test
     public void test2(){
         Class clazz = Person.class;
 
         Class superclass = clazz.getSuperclass();
         System.out.println(superclass);
     }
 
     /**
     获取运行时类的带泛型的父类
 
      */
     @Test
     public void test3(){
         Class clazz = Person.class;
 
         Type genericSuperclass = clazz.getGenericSuperclass();
         System.out.println(genericSuperclass);
     }
 
     /**
     获取运行时类的带泛型的父类的泛型
 
     代码：逻辑性代码  vs 功能性代码
      */
     @Test
     public void test4(){
         Class clazz = Person.class;
 
         Type genericSuperclass = clazz.getGenericSuperclass();
         ParameterizedType paramType = (ParameterizedType) genericSuperclass;
         //获取泛型类型
         Type[] actualTypeArguments = paramType.getActualTypeArguments();
 //        System.out.println(actualTypeArguments[0].getTypeName());
         System.out.println(((Class)actualTypeArguments[0]).getName());
     }
 
     /**
     获取运行时类实现的接口
     */
     @Test
     public void test5(){
         Class clazz = Person.class;
 
         Class[] interfaces = clazz.getInterfaces();
         for(Class c : interfaces){
             System.out.println(c);
         }
 
         System.out.println();
         //获取运行时类的父类实现的接口
         Class[] interfaces1 = clazz.getSuperclass().getInterfaces();
         for(Class c : interfaces1){
             System.out.println(c);
         }
 
     }
     /**
         获取运行时类所在的包
 
      */
     @Test
     public void test6(){
         Class clazz = Person.class;
 
         Package pack = clazz.getPackage();
         System.out.println(pack);
     }
 
     /**
         获取运行时类声明的注解
 
      */
     @Test
     public void test7(){
         Class clazz = Person.class;
 
         Annotation[] annotations = clazz.getAnnotations();
         for(Annotation annos : annotations){
             System.out.println(annos);
         }
     }
 ```

 

## 15.6 反射应用三：调用运行时类的指定结构

 调用指定的属性：

 ```java
 @Test
 public void testField1() throws Exception {
     Class clazz = Person.class;
 
     //创建运行时类的对象
     Person p = (Person) clazz.newInstance();
 
     //1. getDeclaredField(String fieldName):获取运行时类中指定变量名的属性
     Field name = clazz.getDeclaredField("name");
 
     //2.保证当前属性是可访问的
     name.setAccessible(true);
     //3.获取、设置指定对象的此属性值
     name.set(p,"Tom");
 
     System.out.println(name.get(p));
 }
 调用指定的方法：
  @Test
     public void testMethod() throws Exception {
 
         Class clazz = Person.class;
 
         //创建运行时类的对象
         Person p = (Person) clazz.newInstance();
 
         /**
         1.获取指定的某个方法
         getDeclaredMethod():参数1 ：指明获取的方法的名称  参数2：指明获取的方法的形参列表
         */
         Method show = clazz.getDeclaredMethod("show", String.class);
         //2.保证当前方法是可访问的
         show.setAccessible(true);
 
         /**
         3. 调用方法的invoke():参数1：方法的调用者  参数2：给方法形参赋值的实参
         invoke()的返回值即为对应类中调用的方法的返回值。
          */
         Object returnValue = show.invoke(p,"CHN"); //String nation = p.show("CHN");
         System.out.println(returnValue);
 
         System.out.println("如何调用静态方法");
 
         // private static void showDesc()
 
         Method showDesc = clazz.getDeclaredMethod("showDesc");
         showDesc.setAccessible(true);
         //如果调用的运行时类中的方法没返回值，则此invoke()返回null
 //        Object returnVal = showDesc.invoke(null);
         Object returnVal = showDesc.invoke(Person.class);
         System.out.println(returnVal);//null
 
     }
 
 调用指定的构造器：
 @Test
 public void testConstructor() throws Exception {
     Class clazz = Person.class;
 
     //private Person(String name)
     /**
     1.获取指定的构造器
     getDeclaredConstructor():参数：指明构造器的参数列表
      */
 
     Constructor constructor = clazz.getDeclaredConstructor(String.class);
 
     //2.保证此构造器是可访问的
     constructor.setAccessible(true);
 
     //3.调用此构造器创建运行时类的对象
     Person per = (Person) constructor.newInstance("Tom");
     System.out.println(per);
 
 }
 ```

 

## 15.7 反射应用四：动态代理

 1.代理模式的原理：

 使用一个代理将对象包装起来, 然后用该代理对象取代原始对象。任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。 

 2.静态代理

 2.1 举例：

 实现Runnable接口的方法创建多线程。

 ```java
 Class MyThread implements Runnable{} //相当于被代理类
 Class Thread implements Runnable{} //相当于代理类
 main(){
 MyThread t = new MyThread();
 Thread thread = new Thread(t);
 thread.start();//启动线程；调用线程的run()
 }
 ```

 2.2 静态代理的缺点：

① 代理类和目标对象的类都是在编译期间确定下来，不利于程序的扩展。

② 每一个代理类只能为一个接口服务，这样一来程序开发中必然产生过多的代理。

 3.动态代理的特点：
 动态代理是指客户通过代理类来调用其它对象的方法，并且是在程序运行时根据需要动态创建目标类的代理对象。

 4.动态代理的实现

 4.1 需要解决的两个主要问题：

 ```properties
 问题一：如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象。 
 （通过Proxy.newProxyInstance()实现）
 问题二：当通过代理类的对象调用方法a时，如何动态的去调用被代理类中的同名方法a。
 (通过InvocationHandler接口的实现类及其方法invoke())
 ```

 4.2 代码实现：

 ```java
 /**
  
   动态代理的举例
  
   @author shkstart
   @create 2019 上午 10:18
  */
 
 interface Human{
 
     String getBelief();
 
     void eat(String food);
 
 }
 //被代理类
 class SuperMan implements Human{
 
 
     @Override
     public String getBelief() {
         return "I believe I can fly!";
     }
 
     @Override
     public void eat(String food) {
         System.out.println("我喜欢吃" + food);
     }
 }
 
 class HumanUtil{
 
     public void method1(){
         System.out.println("====================通用方法一====================");
 
     }
 
     public void method2(){
         System.out.println("====================通用方法二====================");
     }
 
 }
 
 
 class ProxyFactory{
     //调用此方法，返回一个代理类的对象。解决问题一
     public static Object getProxyInstance(Object obj){//obj:被代理类的对象
         MyInvocationHandler handler = new MyInvocationHandler();
 
         handler.bind(obj);
 
         return Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),handler);
     }
 
 }
 
 class MyInvocationHandler implements InvocationHandler{
 
     private Object obj;//需要使用被代理类的对象进行赋值
 
     public void bind(Object obj){
         this.obj = obj;
     }
 
     //当我们通过代理类的对象，调用方法a时，就会自动的调用如下的方法：invoke()
     //将被代理类要执行的方法a的功能就声明在invoke()中
     @Override
     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
 
         HumanUtil util = new HumanUtil();
         util.method1();
 
         //method:即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
         //obj:被代理类的对象
         Object returnValue = method.invoke(obj,args);
 
         util.method2();
 
         //上述方法的返回值就作为当前类中的invoke()的返回值。
         return returnValue;
 
     }
 }
 
 public class ProxyTest {
 
     public static void main(String[] args) {
         SuperMan superMan = new SuperMan();
         //proxyInstance:代理类的对象
         Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
         //当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
         String belief = proxyInstance.getBelief();
         System.out.println(belief);
         proxyInstance.eat("四川麻辣烫");
 
         System.out.println("");
 
         NikeClothFactory nikeClothFactory = new NikeClothFactory();
 
         ClothFactory proxyClothFactory = (ClothFactory) ProxyFactory.getProxyInstance(nikeClothFactory);
 
         proxyClothFactory.produceCloth();
 
     }
 }
 ```

 体会：反射的动态性。



# 第16章 Java8新特性

## 16.1 Java8新特征概述

 ![image-20210224215245344](java-尚.assets/image-20210224215245344.png)



 ![image-20210224215257151](java-尚.assets/image-20210224215257151.png)



## 16.2 Lambda表达式

 1.Lambda表达式使用前后的对比：

 举例一：

 ```java
 @Test
 public void test1(){
 
     Runnable r1 = new Runnable() {
         @Override
         public void run() {
             System.out.println("我爱北京天安门");
         }
     };
 
     r1.run();
 
     System.out.println("");
 
     Runnable r2 = () - System.out.println("我爱北京故宫");
 
     r2.run();
 }
 
 //举例二：
 @Test
 public void test2(){
 
     Comparator<Integer com1 = new Comparator<Integer() {
         @Override
         public int compare(Integer o1, Integer o2) {
             return Integer.compare(o1,o2);
         }
     };
 
     int compare1 = com1.compare(12,21);
     System.out.println(compare1);
 
     System.out.println("");
     //Lambda表达式的写法
     Comparator<Integer com2 = (o1,o2) - Integer.compare(o1,o2);
 
     int compare2 = com2.compare(32,21);
     System.out.println(compare2);
 
 
     System.out.println("");
     //方法引用
     Comparator<Integer com3 = Integer :: compare;
 
     int compare3 = com3.compare(32,21);
     System.out.println(compare3);
 }
 ```

 2.Lambda表达式的基本语法：

 ```sh
  1.举例： (o1,o2) - Integer.compare(o1,o2);
  2.格式：
       - :lambda操作符 或 箭头操作符
       -左边：lambda形参列表 （其实就是接口中的抽象方法的形参列表
       -右边：lambda体 （其实就是重写的抽象方法的方法体
 ```

 3.如何使用：分为六种情况

  ![image-20210224215455564](java-尚.assets/image-20210224215455564.png)

  ![image-20210224215509095](java-尚.assets/image-20210224215509095.png)

 总结六种情况：

 ```sh
 -左边：lambda形参列表的参数类型可以省略(类型推断)；如果lambda形参列表只一个参数，其一对()也可以省略
 -右边：lambda体应该使用一对{}包裹；如果lambda体只一条执行语句（可能是return语句，省略这一对{}和return关键字
 ```

 

## 16.3 函数式接口

 1.函数式接口的使用说明

 ```sh
  如果一个接口中，只声明了一个抽象方法，则此接口就称为函数式接口。
  我们可以在一个接口上使用 @FunctionalInterface 注解，这样做可以检查它是否是一个函数式接口。
  Lambda表达式的本质：作为函数式接口的实例
 ```

 2.Java8中关于Lambda表达式提供的4个基本的函数式接口：

 具体使用：

  ![image-20210224215644571](java-尚.assets/image-20210224215644571.png)

 3.总结

 3.1 何时使用lambda表达式？

 当需要对一个函数式接口实例化的时候，可以使用lambda表达式。

 3.2 何时使用给定的函数式接口？

 如果我们开发中需要定义一个函数式接口，首先看看在已有的jdk提供的函数式接口是否提供了
 能满足需求的函数式接口。如果有，则直接调用即可，不需要自己再自定义了。



## 16.4 方法引用

 1.理解：

 ```sh
 方法引用可以看做是Lambda表达式深层次的表达。换句话说，方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法。
 ```

 2.使用情境：

 ```sh
 当要传递给Lambda体的操作，已经实现的方法了，可以使用方法引用！
 ```

 3.格式：

 ```sj
 类(或对象) :: 方法名
 ```

 4.分为如下的三种情况：

 ```sh
     情况1     对象 :: 非静态方法
     情况2     类 :: 静态方法
 
     情况3     类 :: 非静态方法
 ```

 5.要求：

 ```sh
  要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同！（针对于情况1和情况2）
   当函数式接口方法的第一个参数是需要引用方法的调用者，并且第二个参数是需要引用方法的参数(或无参数)时：ClassName::methodName（针对于情况3）
 ```

 6.使用建议：

 如果给函数式接口提供实例，恰好满足方法引用的使用情境，大家就可以考虑使用方法引用给函数式接口提供实例。如果大家不熟悉方法引用，那么还可以使用lambda表达式。

 7.使用举例：

 ```java
 // 情况一：对象 :: 实例方法
 //Consumer中的void accept(T t)
 //PrintStream中的void println(T t)
 @Test
 public void test1() {
 	Consumer<String con1 = str - System.out.println(str);
 	con1.accept("北京");
 
 	System.out.println("");
 	PrintStream ps = System.out;
 	Consumer<String con2 = ps::println;
 	con2.accept("beijing");
 }
 
 //Supplier中的T get()
 //Employee中的String getName()
 @Test
 public void test2() {
 	Employee emp = new Employee(1001,"Tom",23,5600);
 
 	Supplier<String sup1 = () - emp.getName();
 	System.out.println(sup1.get());
 
 	System.out.println("");
 	Supplier<String sup2 = emp::getName;
 	System.out.println(sup2.get());
 
 }
 
 // 情况二：类 :: 静态方法
 //Comparator中的int compare(T t1,T t2)
 //Integer中的int compare(T t1,T t2)
 @Test
 public void test3() {
 	Comparator<Integer com1 = (t1,t2) - Integer.compare(t1,t2);
 	System.out.println(com1.compare(12,21));
 
 	System.out.println("");
 
 	Comparator<Integer com2 = Integer::compare;
 	System.out.println(com2.compare(12,3));
 
 }
 
 //Function中的R apply(T t)
 //Math中的Long round(Double d)
 @Test
 public void test4() {
 	Function<Double,Long func = new Function<Double, Long() {
 		@Override
 		public Long apply(Double d) {
 			return Math.round(d);
 		}
 	};
 
 	System.out.println("");
 
 	Function<Double,Long func1 = d - Math.round(d);
 	System.out.println(func1.apply(12.3));
 
 	System.out.println("");
 
 	Function<Double,Long func2 = Math::round;
 	System.out.println(func2.apply(12.6));
 }
 
 // 情况：类 :: 实例方法  (难度)
 // Comparator中的int comapre(T t1,T t2)
 // String中的int t1.compareTo(t2)
 @Test
 public void test5() {
 	Comparator<String com1 = (s1,s2) - s1.compareTo(s2);
 	System.out.println(com1.compare("abc","abd"));
 
 	System.out.println("");
 
 	Comparator<String com2 = String :: compareTo;
 	System.out.println(com2.compare("abd","abm"));
 }
 
 //BiPredicate中的boolean test(T t1, T t2);
 //String中的boolean t1.equals(t2)
 @Test
 public void test6() {
 	BiPredicate<String,String pre1 = (s1,s2) - s1.equals(s2);
 	System.out.println(pre1.test("abc","abc"));
 
 	System.out.println("");
 	BiPredicate<String,String pre2 = String :: equals;
 	System.out.println(pre2.test("abc","abd"));
 }
 
 // Function中的R apply(T t)
 // Employee中的String getName();
 @Test
 public void test7() {
 	Employee employee = new Employee(1001, "Jerry", 23, 6000);
 
 
 	Function<Employee,String func1 = e - e.getName();
 	System.out.println(func1.apply(employee));
 
 	System.out.println("");
 
 	Function<Employee,String func2 = Employee::getName;
 	System.out.println(func2.apply(employee));
 
 
 }
 ```

 

## 16.5 构造器引用与数组引用

 1.构造器引用格式：

 类名::new

 2.构造器引用使用要求：

 ```sh
 和方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致。抽象方法的返回值类型即为构造器所属的类的类型
 ```

 3.构造器引用举例：

 ```java
 //Supplier中的T get()
    //Employee的空参构造器：Employee()
    @Test
    public void test1(){
 
        Supplier<Employee sup = new Supplier<Employee() {
            @Override
            public Employee get() {
                return new Employee();
            }
        };
        System.out.println("");
 
        Supplier<Employee  sup1 = () - new Employee();
        System.out.println(sup1.get());
 
        System.out.println("");
 
        Supplier<Employee  sup2 = Employee :: new;
        System.out.println(sup2.get());
    }
 
 //Function中的R apply(T t)
    @Test
    public void test2(){
        Function<Integer,Employee func1 = id - new Employee(id);
        Employee employee = func1.apply(1001);
        System.out.println(employee);
 
        System.out.println("");
 
        Function<Integer,Employee func2 = Employee :: new;
        Employee employee1 = func2.apply(1002);
        System.out.println(employee1);
 
    }
 
 //BiFunction中的R apply(T t,U u)
    @Test
    public void test3(){
        BiFunction<Integer,String,Employee func1 = (id,name) - new Employee(id,name);
        System.out.println(func1.apply(1001,"Tom"));
 
        System.out.println("");
 
        BiFunction<Integer,String,Employee func2 = Employee :: new;
        System.out.println(func2.apply(1002,"Tom"));
 
    }
 ```

 4.数组引用格式：

 数组类型[] :: new

 5.数组引用举例：

 ```java
 //Function中的R apply(T t)
 @Test
 public void test4(){
     Function<Integer,String[] func1 = length - new String[length];
     String[] arr1 = func1.apply(5);
     System.out.println(Arrays.toString(arr1));
 
     System.out.println("");
 
     Function<Integer,String[] func2 = String[] :: new;
     String[] arr2 = func2.apply(10);
     System.out.println(Arrays.toString(arr2));
 
 }
 ```

 

## 16.6 Stream API

 1.Stream API的理解：

 1.1 Stream关注的是对数据的运算，与CPU打交道
 集合关注的是数据的存储，与内存打交道

 1.2 java8提供了一套api,使用这套api可以对内存中的数据进行过滤、排序、映射、归约等操作。类似于sql对数据库中表的相关操作。

 2.注意点：

 ```sh
  ①Stream 自己不会存储元素。
  ②Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
  ③Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。
 ```

 3.Stream的使用流程：

 ```sh
  ① Stream的实例化
  ② 一系列的中间操作（过滤、映射、...)
  ③ 终止操作
 ```

 4.使用流程的注意点：

 ```sh
  4.1 一个中间操作链，对数据源的数据进行处理
  4.2 一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用
 ```

 5.步骤一：Stream实例化

 ```java
 //创建 Stream方式一：通过集合
     @Test
     public void test1(){
         List<Employee employees = EmployeeData.getEmployees();
 
 //        default Stream<E stream() : 返回一个顺序流
         Stream<Employee stream = employees.stream();
 
 //        default Stream<E parallelStream() : 返回一个并行流
         Stream<Employee parallelStream = employees.parallelStream();
 
     }
 
     //创建 Stream方式二：通过数组
     @Test
     public void test2(){
         int[] arr = new int[]{1,2,3,4,5,6};
         //调用Arrays类的static <T Stream<T stream(T[] array): 返回一个流
         IntStream stream = Arrays.stream(arr);
 
         Employee e1 = new Employee(1001,"Tom");
         Employee e2 = new Employee(1002,"Jerry");
         Employee[] arr1 = new Employee[]{e1,e2};
         Stream<Employee stream1 = Arrays.stream(arr1);
 
     }
     //创建 Stream方式三：通过Stream的of()
     @Test
     public void test3(){
 
         Stream<Integer stream = Stream.of(1, 2, 3, 4, 5, 6);
 
     }
 
     //创建 Stream方式四：创建无限流
     @Test
     public void test4(){
 
 //      迭代
 //      public static<T Stream<T iterate(final T seed, final UnaryOperator<T f)
         //遍历前10个偶数
         Stream.iterate(0, t - t + 2).limit(10).forEach(System.out::println);
 
 
 //      生成
 //      public static<T Stream<T generate(Supplier<T s)
         Stream.generate(Math::random).limit(10).forEach(System.out::println);
 
     }
 ```

 6.步骤二：中间操作

 ![image-20210224220350743](java-尚.assets/image-20210224220350743.png)

 ![image-20210224220359159](java-尚.assets/image-20210224220359159.png)

 ![image-20210224220407382](java-尚.assets/image-20210224220407382.png)

 7.步骤三：终止操作

 ![image-20210224220425478](java-尚.assets/image-20210224220425478.png)

 ![image-20210224220434726](java-尚.assets/image-20210224220434726.png)

 ![image-20210224220442671](java-尚.assets/image-20210224220442671.png)

 ![image-20210224220452175](java-尚.assets/image-20210224220452175.png)

 Collector需要使用Collectors提供实例。

 ![image-20210224220510134](java-尚.assets/image-20210224220510134.png)



## 16.7 Optional类的使用

 java.util.Optional类

 1.理解：为了解决java中的空指针问题而生！

 ```sh
 Optional<T 类(java.util.Optional) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null
 ，表示这个值不存在。原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避
 免空指针异常。
 ```

 2.常用方法：

 ```java
 @Test
     public void test1(){
         //empty():创建的Optional对象内部的value = null
         Optional<Object op1 = Optional.empty();
         if(!op1.isPresent()){//Optional封装的数据是否包含数据
             System.out.println("数据为空");
 
         }
         System.out.println(op1);
         System.out.println(op1.isPresent());
         //如果Optional封装的数据value为空，则get()报错。否则，value不为空时，返回value.
 //        System.out.println(op1.get());
 
     }
 
     @Test
     public void test2(){
         String str = "hello";
 //        str = null;
         //of(T t):封装数据t生成Optional对象。要求t非空，否则报错。
         Optional<String op1 = Optional.of(str);
         //get()通常与of()方法搭配使用。用于获取内部的封装的数据value
         String str1 = op1.get();
         System.out.println(str1);
 
     }
 
     @Test
     public void test3(){
         String str = "beijing";
         str = null;
         //ofNullable(T t) ：封装数据t赋给Optional内部的value。不要求t非空
         Optional<String op1 = Optional.ofNullable(str);
         //orElse(T t1):如果Optional内部的value非空，则返回此value值。如果
         //value为空，则返回t1.
         String str2 = op1.orElse("shanghai");
 
         System.out.println(str2);//
 
 
     }
 ```

 3.典型练习：

 能保证如下的方法执行中不会出现空指针的异常。

 ```java
 //使用Optional类的getGirlName():
 public String getGirlName2(Boy boy){
 
     Optional<Boy boyOptional = Optional.ofNullable(boy);
     //此时的boy1一定非空
     Boy boy1 = boyOptional.orElse(new Boy(new Girl("迪丽热巴")));
 
     Girl girl = boy1.getGirl();
 
     Optional<Girl girlOptional = Optional.ofNullable(girl);
     //girl1一定非空
     Girl girl1 = girlOptional.orElse(new Girl("古力娜扎"));
 
     return girl1.getName();
 }
 
 @Test
 public void test5(){
     Boy boy = null;
     boy = new Boy();
     boy = new Boy(new Girl("苍老师"));
     String girlName = getGirlName2(boy);
     System.out.println(girlName);
 
 }
 ```

 

# 第17章 Java9&10&11新特性

略







