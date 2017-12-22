[toc]

---

### 1 垃圾回收中的重要概念

#### 1.1 定义

In computer science, garbage collection (GC) is a form of automatic memory management. The garbage collector, or just collector, attempts to reclaim garbage, or memory occupied by objects that are no longer in use by the program. Garbage collection was invented by John McCarthy around 1959 to simplify manual memory management in Lisp. （引用自[维基百科](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))）

#### 1.2 GC性能的评价标准

- 吞吐量：是指单位时间内是有多少时间是用来运行user application的。GC占用的时间过多，就会导致吞吐量较低。
- 最大暂停时间：基本上所有的垃圾回收算法，都会在执行GC的过程中，暂停user application。如果暂停时间过长，必然会影响用户体验，尤其是那些交互性较强的应用。
- 堆使用效率：影响堆使用效率的主要有两个因素，一个是对象头部大小，一个是堆的用法。一般来说，堆的头部越大，存储的信息越多，那么GC的效率就会越高，吞吐量什么的也会有更佳的表现。但是，我们必须明白，对象头越小越好。另外，不同的算法对于堆的不同用法，也会导致堆使用效率差距非常大。比如复制算法，用户应用只能使用一般的堆大小。GC是自动管理内存的，如果因为GC导致过量占用堆，那么就是本末倒置了。
- 访问的局部性：具有引用关系的对象之间很可能存在连续访问的情况。因此，把具有引用关系的对象安排在堆中较劲的位置，可以充分利用内存访问局部性。有的GC算法会根据引用关系重排对象，比如复制算法。
- 等等等等

设计垃圾回收算法时，折中无处不在。较大的吞吐量和较短的最大暂停时间往往不可兼得。

---

### 2 常见的GC算法

#### 2.1 标记-清除算法  STW


```
mark_sweep(){
    mark()
    sweep()
}
```

分两个阶段，整个过程需要STW：

- mark phase：时间复杂度跟活动对象数量成正比
- sweep phase：时间复杂度跟堆的大小成正比

sweep阶段回收的空间会连接到空闲链表上，分配空间时从空闲链表分配，因此分配空间时间复杂度可能会有点高，即需要遍历整个空闲链表。

优点是实现简单，并且与保守式GC兼容（即没有移动对象） 

缺点也非常明显：

- 存在内存碎片
- 分配速度较慢：使用多个空闲链表
- 与写时复制技术不兼容：位图标记法
- sweep操作时间复杂度同堆大小成正比：延迟清除法

延迟清除法：没有空闲链表。定义一个全局变量sweeping，从sweeping开始遍历分配新空间，遍历的开始位置处于上一次lazy_sweep操作发现的分块的右边。如果当前分块mark=true，则取消标记。如果mark=false并且分开大小大于申请空间，则分配给他。
当遍历到堆末尾时，需要将sweeping设置为heap_start，并且需要重新标记。

#### 2.2 引用计数法

在标记-清除等GC算法中，没有分块可用时，mutator会调用下面的函数启动GC回收空间，以便进行分配：


```
garbage_collect(){
    ...
}
```
然而引用计数法是没有启动GC的语句的，它与mutator的执行密切相关，它在mutator的执行过程中通过增减引用计数器的值来实现实时的内存管理和垃圾回收。

引用计数器的更新主要有两个场景：

- 分配新对象时
- 更新指针时

如果引用计数值变为0，则会立即连接到空闲链表上去。分配空间时，从空闲链表分配，分配失败，则直接失败返回。

优点是：

- 最大暂停时间短
- 可即刻回收垃圾

缺点是：

- 引用计数值的增减处理繁重，吞吐量低
- 循环引用
- 计数器占用很多位

#### 2.3 复制算法 STW

GC复制算法将堆空间分成大小相等的两块：from空间和to空间。新对象只能从from空间分配。

当from空间没有可用空间时，则会启动GC，将from空间的活动对象复制到to空间，同时将from和to的身份互换。因此其时间复杂度，与活动对象的数量成正比，而与堆的大小无关。

复制时，是从根对象递归遍历复制过去的。因此，我们定义了一个free变量记录下从这个位置分配可用空间，分配空间的时间复杂度为常数。

优点：

- 因为时间复杂度与活动对象数量成正比，而与堆大小无关。所以，吞吐量优秀。
- 分配速度快
- 不会发生碎片化
- 访问的局部性原理

缺点是：

- 堆的使用效率低下
- 移动对象，与保守式GC不兼容


#### 2.4 标记-压缩算法  STW

标记-压缩算法时将标记-清除和复制算法相结合的产物。

标记阶段跟标记-清除算法一样，从根引用的活动变量开始遍历。时间复杂度同活动对象的数量成正比。

而压缩阶段则需要遍历整个堆，按照之前的排列顺序压缩到堆的一侧去。

压缩阶段需要遍历三次堆：

- 第一次遍历，需要找出计算出所有的活动对象需要移动到哪个位置去
- 第二次遍历，需要重写根指针，重写活动对象的指针
- 第三次遍历，移动对象到目标位置

优点是堆的利用率很高，没有碎片。缺点也是灰常的明显，压缩的时间复杂度太高，而且与堆的大小成正比。

#### 2.5 分代垃圾回收

分代垃圾回收在对象中引入了“年龄”的概念，通过优先回收容易成为垃圾的对象，提高GC的效率。

我们把刚生成的对象称为**新生代对象**，到达一定年龄的对象称为**老年代对象**。

在新生代空间执行的GC，称为**minor GC**。在老年代空间执行的GC，称为**major GC**。

堆的结构：

![image](https://github.com/shenlanse/learning-note/raw/master/images/mem.jpg)


新生成对象分配的空间都是从Eden区分配的。当Eden区满了的时候，就会触发minor GC，将Eden区和From区的活动对象都复制到To，然后交互To和From的身份。

复制的时候如果超过一定的年龄或者To空间不足（即Eden和From的活动对象占用的空间超过了To），那么直接复制到老年代空间。如果老年代空间不足则会触发major GC。

那些大于Eden空间的对象，一般也不会直接失败，而是直接分配到老年代空间。实际的实现，一般是超过一定大小，则会将其分配到老年代空间。

新生代空间和老年代空间采用不同的GC算法。新生代采用复制算法，老年代采用标记-压缩算法或者标记-清除算法。


---

### 3 Golang的垃圾回收算法

#### 3.1 三色标记法

golang采用的是并发的三色标记清除算法（Tri-color marking）。

- 白色：还没搜索的对象
- 灰色：正在搜索的对象
- 黑色：搜索完成的对象

GC开始前所有的对象都是白色对象。GC开始时，会将从根能够直接引用的对象标记为灰色，并且放到堆栈里。

然后，灰色对象会被依次弹出栈，其子对象也被涂成灰色，压入栈。当其所有的子对象都变成灰色后，就会把这个对象涂成黑色。

当GC结束时，活动对象全部为黑色对象，垃圾则为白色对象，回收白色对象即可。

主要分为四个阶段：

- root_scan：STW
- mark...mark...mark...mark...
- mark termination：STW
- sweep...sweep...sweep...

接下来分别介绍一下上述的四个不同阶段。

##### 3.1.1 root scan

根查找阶段需要STW。找出能够从根直接引用的对象，标记为灰色，压入栈。


```
root_scan_phase(){
    for(r : $roots){
        mark(*r)
    }
    $gc_phase = GC_MARK
}

mark(obj){
    if(obj.mark == false){
        obj.mark = true
        push(obj,$mark_stack)
    }
}
```
当我们把所有直接从根引用的对象涂成了灰色时，root scan阶段就结束了，mutator（即user application）会继续执行。

##### 3.1.2 mark 和 mark termination

mark是分多次运行的，即增量式的mark，不需要STW，它和mutator交替运行。它主要是弹出栈里面的对象，将其子对象涂成灰色，压入栈，然后把这个对象涂成黑色。重复这个过程，直到栈为空。

mark termination则是需要STW的。它会root_rescan，然后重新执行mark。

然而，mark阶段与mutator并发运行存在一个问题，可能误把活动对象当做垃圾对象回收。

比如下面的情况：

![image](https://github.com/shenlanse/learning-note/raw/master/images/barrier.jpg)

第二张图，创建了从黑色对象到白色对象的引用。第三张图，删除了从灰色对象到白色对象的引用。这个时候就会导致C被误认为垃圾而回收。

为了避免这种情况的发生，需要引入write barrier。

最常用的write barrier是由Dijkstra提出的insertion style write barrier。


```

write_barrier(obj,field,newobj){
    if(newobj.mark == false){
        newobj.mark = true
        push(newobj,$mark_stack)
    }
    *field = newobj
}

```
即如果新引用的对象是白色对象，则直接把它涂为灰色：
![image](https://github.com/shenlanse/learning-note/raw/master/images/barrier2.jpg)


mark和mark termination的伪代码为：


```
incremental_mark_phase(){
    for(i : 1...MARK_MAX){
        // 分多次mark，不需要STW
        if(is_empty($mark_stack) == false){
            obj = pop($mark_stack)
            for(child : children(obj)){
                mark(*child)
            }
        }else{
            // mark termination，需要STW
            for(r : roots){
                mark(*r)
            }
            while(is_empty($mark_stack) == false){
                obj = pop($mark_stack)
                for(child : children(obj)){
                    mark(*child)
                }
            }
            $gc_phase = GC_SWEEP
            $sweeping = $heap_start
            return
        }
    }
}


```



##### 3.1.3 sweep

sweep也是分多次的，增量式的回收垃圾，跟mutator交替运行。跟标记-清除算法的实现基本一致，也是需要遍历整个堆，将白色对象挂到空闲链表上，黑色对象取消mark标记。

##### 3.1.4 分配新对象

从空闲链表分配。

#### 3.2 golang为什么没有采用压缩算法和分代算法

有点高深，想深入了解的可以参考[golang-nuts](https://groups.google.com/forum/#!topic/golang-nuts/KJiyv2mV2pU)上面的讨论。


---

### 4 Golang垃圾回收的相关参数

#### 4.1 触发GC

gc触发的时机：2分钟或者内存占用达到一个阈值（当前堆内存占用是上次gc后对内存占用的两倍，当GOGC=100时）

```
 # 表示当前应用占用的内存是上次GC时占用内存的两倍时，触发GC
export GOGC=100
```


#### 4.2 查看GC信息



```
export GODEBUG=gctrace=1
```


可以查看gctrace信息。

举例：


```
gc 1 @0.008s 6%: 0.071+2.0+0.080 ms clock, 0.21+0.22/1.9/1.9+0.24 ms cpu, 4->4->3 MB, 5 MB goal, 4 P
# command-line-arguments
gc 1 @0.001s 16%: 0.071+3.3+0.060 ms clock, 0.21+0.17/2.9/0.36+0.18 ms cpu, 4->4->4 MB, 5 MB goal, 4 P
gc 2 @0.016s 8%: 0.020+6.0+0.070 ms clock, 0.082+0.094/3.9/2.2+0.28 ms cpu, 8->9->8 MB, 9 MB goal, 4 P
gc 3 @0.046s 7%: 0.019+7.3+0.062 ms clock, 0.076+0.089/7.1/7.0+0.24 ms cpu, 14->16->14 MB, 17 MB goal, 4 P
gc 4 @0.092s 8%: 0.015+24+0.10 ms clock, 0.060+0.10/24/0.75+0.42 ms cpu, 25->27->24 MB, 28 MB goal, 4 P

```
每个字段表示什么信息可以参考 [golang doc](https://godoc.org/runtime)

---

### 5 如何提高GC的性能

Golang的GC算法是固定的，用户无法去配置采用什么算法，也没法像Java一样配置年轻代、老年代的空间比例等。golang的GC相关的配置参数只有一个，即GOGC，用来表示触发GC的条件。

目前来看，提高GC效率我们唯一能做的就是减少垃圾的产生。所以说，这一章称为提高GC的性能也不太合适。下面我们就主要讨论一下，在golang中如何减少垃圾的产生，有哪些需要注意的方面。

#### 5.1 golang中的内存分配

参考官网[Frequently Asked Questions (FAQ)](https://golang.org/doc/faq#stack_or_heap)

How do I know whether a variable is allocated on the heap or the stack? 

> From a correctness standpoint, you don't need to know. Each variable in Go exists as long as there are references to it. The storage location chosen by the implementation is irrelevant to the semantics of the language.
> 
> The storage location does have an effect on writing efficient programs. When possible, the Go compilers will allocate variables that are local to a function in that function's stack frame. However, if the compiler cannot prove that the variable is not referenced after the function returns, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors. Also, if a local variable is very large, it might make more sense to store it on the heap rather than the stack.
> 
> In the current compilers, if a variable has its address taken, that variable is a candidate for allocation on the heap. However, a basic escape analysis recognizes some cases when such variables will not live past the return from the function and can reside on the stack.


我们看一个例子有个直观的认识：


```
1 package main
2 
3 import ()
4 
5 func foo() *int {
6     var x int
7     return &x
8 }
9 
10 func bar() int {
11     x := new(int)
12     *x = 1
13     return *x
14 }
15 
16 func big() {
17     x := make([]int,0,20)
18     y := make([]int,0,20000)
19 
20     len := 10
21     z := make([]int,0,len)
22 }
23 
24 func main() {
25  
26 }
```



```
# go build -gcflags='-m -l' test.go

./test.go:7:12: &x escapes to heap
./test.go:6:9: moved to heap: x
./test.go:11:13: bar new(int) does not escape
./test.go:18:14: make([]int, 0, 20000) escapes to heap
./test.go:21:14: make([]int, 0, len) escapes to heap
./test.go:17:14: big make([]int, 0, 20) does not escape
./test.go:17:23: x declared and not used
./test.go:18:23: y declared and not used
./test.go:21:23: z declared and not used
```


#### 5.2 sync.Pool对象池

sync.Pool主要是为了重用对象，一方面缩短了申请空间的时间，另一方面，还减轻了GC的压力。不过它是一个临时对象池，为什么这么说呢？因为对象池中的对象会被GC回收。所以说，有状态的对象，比如数据库连接是不能够用sync.Pool来实现的。

>  use sync.Pool if you frequently allocate many objects of the same type and you want to save some allocation and garbage collection overhead. However, in the current implementation, any unreferenced sync.Pool objects are removed at each garbage collection cycle, so you can't use this as a long-lived free-list of objects. If you want a free-list that maintains objects between GC runs, you'll still have to build that yourself. This is only to reuse allocated objects between garbage collection cycles.


sync.Pool主要有两个方法：


```
func (p *Pool) Get() interface{} {
    ...
}
func (p *Pool) Put(x interface{}) {
    ...
}

```

Get方法是指从临时对象池中申请对象，put是指把不再使用的对象返回对象池，以便后续重用。如果我们在使用Get申请新对象时pool中没有可用的对象，那么就会返回nil，除非设置了sync.Pool的New func：


```
type Pool struct {

    ...
    
	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() interface{}
}
```


另外，我们不能对从对象池申请到的对象值做任何假设，可能是New新生成的，可能是被某个协程修改过放回来的。


一个比较好的使用sync.Pool的例子：


```
var DEFAULT_SYNC_POOL *SyncPool

func NewPool() *SyncPool {
	DEFAULT_SYNC_POOL = NewSyncPool(
		5,     
		30000, 
		2,     
	)
	return DEFAULT_SYNC_POOL
}

func Alloc(size int) []int64 {
	return DEFAULT_SYNC_POOL.Alloc(size)
}

func Free(mem []int64) {
	DEFAULT_SYNC_POOL.Free(mem)
}

// SyncPool is a sync.Pool base slab allocation memory pool
type SyncPool struct {
	classes     []sync.Pool
	classesSize []int
	minSize     int
	maxSize     int
}

func NewSyncPool(minSize, maxSize, factor int) *SyncPool {
	n := 0
	for chunkSize := minSize; chunkSize <= maxSize; chunkSize *= factor {
		n++
	}
	pool := &SyncPool{
		make([]sync.Pool, n),
		make([]int, n),
		minSize, maxSize,
	}
	n = 0
	for chunkSize := minSize; chunkSize <= maxSize; chunkSize *= factor {
		pool.classesSize[n] = chunkSize
		pool.classes[n].New = func(size int) func() interface{} {
			return func() interface{} {
				buf := make([]int64, size)
				return &buf
			}
		}(chunkSize)
		n++
	}
	return pool
}

func (pool *SyncPool) Alloc(size int) []int64 {
	if size <= pool.maxSize {
		for i := 0; i < len(pool.classesSize); i++ {
			if pool.classesSize[i] >= size {
				mem := pool.classes[i].Get().(*[]int64)
				// return (*mem)[:size]
				return (*mem)[:0]
			}
		}
	}
	return make([]int64, 0, size)
}

func (pool *SyncPool) Free(mem []int64) {
	if size := cap(mem); size <= pool.maxSize {
		for i := 0; i < len(pool.classesSize); i++ {
			if pool.classesSize[i] >= size {
				pool.classes[i].Put(&mem)
				return
			}
		}
	}
}
```


有一个开源的通用golang对象池实现，有兴趣的可以研究一下：[Go Commons Pool](https://github.com/jolestar/go-commons-pool)，在此不再赘述。


#### 5.3 append

我们先看一下append的基本用法。

```
nums:=make([]int,0,10)
```

创建切片，len=0，cap=10，底层实际上分配了10个元素大小的空间。在没有append数据的情况下，不能直接使用nums[index]。

---

```
nums:=make([]int,5,10)
```
创建切片，len=5，cap=10，底层实际上分配了10个元素大小的空间。在没有append数据的情况下，可以直接使用nums[index]，index的范围是[0,4]。执行append操作的时候是从index=5的位置开始存储的。


---


```
nums := make([]int,5)
```
如果没有指定capacity，那么cap与len相等。

---


```
nums = append(nums,10)
```
执行append操作的时候，nums的地址可能会改变，因此需要利用其返回值重新设置nums。至于nums的地址会不会改变，取决于还没有空间来存储新的数据，如果没有空闲空间了，那就需要申请cap*2的空间，将数据复制过去。

因此，我们在使用append操作的时候，最好是设置一个比较合理的cap值，即根据自己的应用场景预申请大小合适的空间，避免无谓的不断重新申请新空间，这样可以减少GC的压力。

由append导致的内存飙升和GC压力过大这个问题，需要特别注意一下。


### 参考文献

> 1 https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)    
> 2 http://legendtkl.com/2017/04/28/golang-gc/    
> 3 https://github.com/golang/proposal/blob/master/design/17503-eliminate-rescan.md    
> 4 https://lengzzz.com/note/gc-in-golang    
> 5 https://making.pusher.com/golangs-real-time-gc-in-theory-and-practice/    
> 6 https://blog.twitch.tv/gos-march-to-low-latency-gc-a6fa96f06eb7    
> 7 https://golang.org/doc/faq    
> 8 《垃圾回收的算法与实现》 中村成杨 相川光. 编著