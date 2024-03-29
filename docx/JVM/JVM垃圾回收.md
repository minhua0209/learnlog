## JVM垃圾处理
### 哪些内存需要回收
- Java堆和方法区

###判断对象生死的算法
####引用计数算法
- 描述
   - 给对象添加一个引用计数器
   - 每有一个地方引用它，计数器加 1
   - 引用失效时，计数器减 1
   - 计数器值为 0 的对象不再可用
- 缺点
   - 无法解决循环引用问题

####可达性分析算法（主流）
- 描述
   - 从"GC Root"对象作为起点开始向下搜索，走过的路径称为引用链（Reference Chain）
   - 从"GC Root"开始，不可达的对象被判为不可用
- Java中可做"GC Root"的对象
   - 栈中（本地变量表中的reference）
      - 虚拟机栈中，栈帧中的本地变量表引用的对象
      - 本地方法栈中，JNI 引用的对象（native方法）
   - 方法区
      - 类的静态属性引用的对象
      - 常量引用的对象

即便如此，一个对象**也不是**一旦被判为不可达，就立即死去的，宣告一个的死亡需要经过**两次标记**过程 

### 四种引用类型
- **强引用**：像`Object obj = new Object()`这种，只要强引用还存在，垃圾收集器就永远不会回收掉被引用的对象
- **软引用**：用来引用还存在但非必须的对象。对于软引用对象，在 OOM 前，虚拟机会把这些对象列入回收范围中进行第二次回收，如果这次回收后，内存还是不够用，就 OOM。实现类`SoftReference`
- **弱引用**：被弱引用引用的对象只能生存到下一次垃圾收集前，一旦发生垃圾收集，被弱引用所引用的对象就会被清掉。实现类：`WeakReference`
- **虚引用**：幽灵引用，对对象没有半毛钱影响，甚至不能用来取得一个对象的实例。它唯一的用途就是：当一个虚引用引用的对象被回收时，系统会收到这个对象被回收了的通知。实现类：`PhantomReference`

### 对象死亡的两次标记过程
- 当发现对象不可达后，该对象被第一次标记，并进行是否有必要执行`finalize()`方法的判断
   - 不需要执行：对象没有覆盖`finalize()`方法，或`finalize()`方法已被执行过（`finalize()`只被执行一次）
   - 需要执行：将该对象放置在一个队列中，稍后由一个虚拟机自动创建的低优先级线程执行
- `finalize()`方法是对象逃脱死亡的最后一次机会，不过虚拟机不保证等待`finalize()`方法执行结束，也就是说，虚拟机只触发`finalize()`方法的执行，如果这个方法要执行超久，那么虚拟机并不等待它执行结束，所以最好不要手动调用这个方法。
- `finalize()`方法能做的，try-finally 都能做，所以忘了这个方法吧

###方法区的回收
- 废弃常量：例如一个字符串 "abc"，当没有任何引用指向 "abc" 时，就是废弃常量了
- 无用的类，满足三个条件
   - 该类的所有实例已被回收，Java 堆中不存在该类的任何实例
   - 加载该类的 Classloader 已被回收
   - 该类的 Class 对象没有被任何地方引用，即无法在任何地方通过反射访问该类的方法

   
###垃圾收集算法
####标记-清除算法(基础)
- 描述
   - 先标记需要回收的对象
   - 标记后，统一回收被标记对象
- 缺点
   - 标记和清除效率不高
   - 标记清除后会产生大量不连续的内存碎片，导致以后为较大的对象分配内存时找不到足够的连续内存，会提前触发另一次 GC
   
####复制算法(解决效率问题)
- 描述
   - 将可用内存分为大小相等的两块，每次只使用其中一块
   - 当一块内存用完时，将这块内存上还存活的对象复制到另一块内存上去，将这一块内存全部清理掉  
- 缺点
   - 可用内存缩小为原来的一半
- 改进方法：
   - 新生代中的对象 98% 都是朝生夕死的，所以不需要按照 1:1 的比例对内存进行划分
   - 内存划分为：
      - 1 块比较大的`Eden`区
      - 2 块较小的`Survivor`区
   - 每次使用`Eden`区和 1 块`Survivor`区
   - 回收时，将以上 2 部分区域中的存活对象复制到另一块`Survivor`区中，然后将以上两部分区域清空
   - JVM 参数设置`-XX:SurvivorRatio=8`表示`Eden`区大小 / 1 块`Survivor`区大小 = 8 

   ####标记-整理算法(解决空间碎片问题)
   - 描述
      - 标记方法与 “标记 - 清除算法” 一样
      - 标记完后，将所有存活对象向一端移动，然后直接清理掉边界以外的内存
   - 缺点：存在效率问题，适合老年代

   #### 分代收集算法(进化)
   - **新生代**：GC过后只有少量对象存活 — 复制算法 
   - **老年代**：GC过后对象存活率高 — 标记-整理算法

   
   ###HotSpot虚拟机中GC算法实现
   - OopMap遍历：HotSpot 使用了一种OopMap的数据结构，OopMap记录了栈上本地变量到堆上对象的引用关系，GC的时候就不用遍历整个栈只遍历每个栈的OopMap就行了
   - 安全点：一种更新OopMap的方法
      - 更新OopMap代价比较大，因此在安全点更新
      - 选取安全点方式(长时间执行的指令处)：指令的复用，例如：方法调用、循环跳转、异常跳转等
   - GC停顿：如果在分析可达性的过程中，对象的引用关系还在变来变去，那是不可能得到正确的分析结果的，所以在枚举根节点时也是必须要停顿的
      - 抢先式中断：
         - 先中断所有线程
         - 发现有线程没中断在安全点，恢复它，让它跑到安全点
      - 主动式中断：(主要使用)
         - 设置一个中断标记
         - 每个线程到达安全点时，检查这个中断标记，选择是否中断自己
  - 安全区域：指在一段代码片段之中，引用关系不会发生变化，因此在这个区域中的任意位置开始 GC 都是安全的

  ###7个垃圾收集器
  - Serial / ParNew 搭配 Serial Old 收集器
  - Parallel 搭配 Parallel Scavenge 收集器
  - CMS 收集器
  - G1 收集器
  
  [文章摘抄自开源中国](https://my.oschina.net/u/3728792/blog/3070471)
     
       