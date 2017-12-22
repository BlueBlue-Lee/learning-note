[toc]

---

### 1 ���������е���Ҫ����

#### 1.1 ����

In computer science, garbage collection (GC) is a form of automatic memory management. The garbage collector, or just collector, attempts to reclaim garbage, or memory occupied by objects that are no longer in use by the program. Garbage collection was invented by John McCarthy around 1959 to simplify manual memory management in Lisp. ��������[ά���ٿ�](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science))��

#### 1.2 GC���ܵ����۱�׼

- ����������ָ��λʱ�������ж���ʱ������������user application�ġ�GCռ�õ�ʱ����࣬�ͻᵼ���������ϵ͡�
- �����ͣʱ�䣺���������е����������㷨��������ִ��GC�Ĺ����У���ͣuser application�������ͣʱ���������Ȼ��Ӱ���û����飬��������Щ�����Խ�ǿ��Ӧ�á�
- ��ʹ��Ч�ʣ�Ӱ���ʹ��Ч�ʵ���Ҫ���������أ�һ���Ƕ���ͷ����С��һ���Ƕѵ��÷���һ����˵���ѵ�ͷ��Խ�󣬴洢����ϢԽ�࣬��ôGC��Ч�ʾͻ�Խ�ߣ�������ʲô��Ҳ���и��ѵı��֡����ǣ����Ǳ������ף�����ͷԽСԽ�á����⣬��ͬ���㷨���ڶѵĲ�ͬ�÷���Ҳ�ᵼ�¶�ʹ��Ч�ʲ��ǳ��󡣱��縴���㷨���û�Ӧ��ֻ��ʹ��һ��ĶѴ�С��GC���Զ������ڴ�ģ������ΪGC���¹���ռ�öѣ���ô���Ǳ�ĩ�����ˡ�
- ���ʵľֲ��ԣ��������ù�ϵ�Ķ���֮��ܿ��ܴ����������ʵ��������ˣ��Ѿ������ù�ϵ�Ķ������ڶ��нϾ���λ�ã����Գ�������ڴ���ʾֲ��ԡ��е�GC�㷨��������ù�ϵ���Ŷ��󣬱��縴���㷨��
- �ȵȵȵ�

������������㷨ʱ�������޴����ڡ��ϴ���������ͽ϶̵������ͣʱ���������ɼ�á�

---

### 2 ������GC�㷨

#### 2.1 ���-����㷨  STW


```
mark_sweep(){
    mark()
    sweep()
}
```

�������׶Σ�����������ҪSTW��

- mark phase��ʱ�临�Ӷȸ����������������
- sweep phase��ʱ�临�Ӷȸ��ѵĴ�С������

sweep�׶λ��յĿռ�����ӵ����������ϣ�����ռ�ʱ�ӿ���������䣬��˷���ռ�ʱ�临�Ӷȿ��ܻ��е�ߣ�����Ҫ����������������

�ŵ���ʵ�ּ򵥣������뱣��ʽGC���ݣ���û���ƶ����� 

ȱ��Ҳ�ǳ����ԣ�

- �����ڴ���Ƭ
- �����ٶȽ�����ʹ�ö����������
- ��дʱ���Ƽ��������ݣ�λͼ��Ƿ�
- sweep����ʱ�临�Ӷ�ͬ�Ѵ�С�����ȣ��ӳ������

�ӳ��������û�п�����������һ��ȫ�ֱ���sweeping����sweeping��ʼ���������¿ռ䣬�����Ŀ�ʼλ�ô�����һ��lazy_sweep�������ֵķֿ���ұߡ������ǰ�ֿ�mark=true����ȡ����ǡ����mark=false���ҷֿ���С��������ռ䣬����������
����������ĩβʱ����Ҫ��sweeping����Ϊheap_start��������Ҫ���±�ǡ�

#### 2.2 ���ü�����

�ڱ��-�����GC�㷨�У�û�зֿ����ʱ��mutator���������ĺ�������GC���տռ䣬�Ա���з��䣺


```
garbage_collect(){
    ...
}
```
Ȼ�����ü�������û������GC�����ģ�����mutator��ִ��������أ�����mutator��ִ�й�����ͨ���������ü�������ֵ��ʵ��ʵʱ���ڴ������������ա�

���ü������ĸ�����Ҫ������������

- �����¶���ʱ
- ����ָ��ʱ

������ü���ֵ��Ϊ0������������ӵ�����������ȥ������ռ�ʱ���ӿ���������䣬����ʧ�ܣ���ֱ��ʧ�ܷ��ء�

�ŵ��ǣ�

- �����ͣʱ���
- �ɼ��̻�������

ȱ���ǣ�

- ���ü���ֵ�����������أ���������
- ѭ������
- ������ռ�úܶ�λ

#### 2.3 �����㷨 STW

GC�����㷨���ѿռ�ֳɴ�С��ȵ����飺from�ռ��to�ռ䡣�¶���ֻ�ܴ�from�ռ���䡣

��from�ռ�û�п��ÿռ�ʱ���������GC����from�ռ�Ļ�����Ƶ�to�ռ䣬ͬʱ��from��to����ݻ����������ʱ�临�Ӷȣ�����������������ȣ�����ѵĴ�С�޹ء�

����ʱ���ǴӸ�����ݹ�������ƹ�ȥ�ġ���ˣ����Ƕ�����һ��free������¼�´����λ�÷�����ÿռ䣬����ռ��ʱ�临�Ӷ�Ϊ������

�ŵ㣺

- ��Ϊʱ�临�Ӷ����������������ȣ�����Ѵ�С�޹ء����ԣ����������㡣
- �����ٶȿ�
- ���ᷢ����Ƭ��
- ���ʵľֲ���ԭ��

ȱ���ǣ�

- �ѵ�ʹ��Ч�ʵ���
- �ƶ������뱣��ʽGC������


#### 2.4 ���-ѹ���㷨  STW

���-ѹ���㷨ʱ�����-����͸����㷨���ϵĲ��

��ǽ׶θ����-����㷨һ�����Ӹ����õĻ������ʼ������ʱ�临�Ӷ�ͬ���������������ȡ�

��ѹ���׶�����Ҫ���������ѣ�����֮ǰ������˳��ѹ�����ѵ�һ��ȥ��

ѹ���׶���Ҫ�������ζѣ�

- ��һ�α�������Ҫ�ҳ���������еĻ������Ҫ�ƶ����ĸ�λ��ȥ
- �ڶ��α�������Ҫ��д��ָ�룬��д������ָ��
- �����α������ƶ�����Ŀ��λ��

�ŵ��Ƕѵ������ʺܸߣ�û����Ƭ��ȱ��Ҳ�ǻҳ������ԣ�ѹ����ʱ�临�Ӷ�̫�ߣ�������ѵĴ�С�����ȡ�

#### 2.5 �ִ���������

�ִ����������ڶ����������ˡ����䡱�ĸ��ͨ�����Ȼ������׳�Ϊ�����Ķ������GC��Ч�ʡ�

���ǰѸ����ɵĶ����Ϊ**����������**������һ������Ķ����Ϊ**���������**��

���������ռ�ִ�е�GC����Ϊ**minor GC**����������ռ�ִ�е�GC����Ϊ**major GC**��

�ѵĽṹ��

![image](https://github.com/shenlanse/learning-note/raw/master/images/mem.jpg)


�����ɶ������Ŀռ䶼�Ǵ�Eden������ġ���Eden�����˵�ʱ�򣬾ͻᴥ��minor GC����Eden����From���Ļ���󶼸��Ƶ�To��Ȼ�󽻻�To��From����ݡ�

���Ƶ�ʱ���������һ�����������To�ռ䲻�㣨��Eden��From�Ļ����ռ�õĿռ䳬����To������ôֱ�Ӹ��Ƶ�������ռ䡣���������ռ䲻����ᴥ��major GC��

��Щ����Eden�ռ�Ķ���һ��Ҳ����ֱ��ʧ�ܣ�����ֱ�ӷ��䵽������ռ䡣ʵ�ʵ�ʵ�֣�һ���ǳ���һ����С����Ὣ����䵽������ռ䡣

�������ռ��������ռ���ò�ͬ��GC�㷨�����������ø����㷨����������ñ��-ѹ���㷨���߱��-����㷨��


---

### 3 Golang�����������㷨

#### 3.1 ��ɫ��Ƿ�

golang���õ��ǲ�������ɫ�������㷨��Tri-color marking����

- ��ɫ����û�����Ķ���
- ��ɫ�����������Ķ���
- ��ɫ��������ɵĶ���

GC��ʼǰ���еĶ����ǰ�ɫ����GC��ʼʱ���Ὣ�Ӹ��ܹ�ֱ�����õĶ�����Ϊ��ɫ�����ҷŵ���ջ�

Ȼ�󣬻�ɫ����ᱻ���ε���ջ�����Ӷ���Ҳ��Ϳ�ɻ�ɫ��ѹ��ջ���������е��Ӷ��󶼱�ɻ�ɫ�󣬾ͻ���������Ϳ�ɺ�ɫ��

��GC����ʱ�������ȫ��Ϊ��ɫ����������Ϊ��ɫ���󣬻��հ�ɫ���󼴿ɡ�

��Ҫ��Ϊ�ĸ��׶Σ�

- root_scan��STW
- mark...mark...mark...mark...
- mark termination��STW
- sweep...sweep...sweep...

�������ֱ����һ���������ĸ���ͬ�׶Ρ�

##### 3.1.1 root scan

�����ҽ׶���ҪSTW���ҳ��ܹ��Ӹ�ֱ�����õĶ��󣬱��Ϊ��ɫ��ѹ��ջ��


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
�����ǰ�����ֱ�ӴӸ����õĶ���Ϳ���˻�ɫʱ��root scan�׶ξͽ����ˣ�mutator����user application�������ִ�С�

##### 3.1.2 mark �� mark termination

mark�Ƿֶ�����еģ�������ʽ��mark������ҪSTW������mutator�������С�����Ҫ�ǵ���ջ����Ķ��󣬽����Ӷ���Ϳ�ɻ�ɫ��ѹ��ջ��Ȼ����������Ϳ�ɺ�ɫ���ظ�������̣�ֱ��ջΪ�ա�

mark termination������ҪSTW�ġ�����root_rescan��Ȼ������ִ��mark��

Ȼ����mark�׶���mutator�������д���һ�����⣬������ѻ����������������ա�

��������������

![image](https://github.com/shenlanse/learning-note/raw/master/images/barrier.jpg)

�ڶ���ͼ�������˴Ӻ�ɫ���󵽰�ɫ��������á�������ͼ��ɾ���˴ӻ�ɫ���󵽰�ɫ��������á����ʱ��ͻᵼ��C������Ϊ���������ա�

Ϊ�˱�����������ķ�������Ҫ����write barrier��

��õ�write barrier����Dijkstra�����insertion style write barrier��


```

write_barrier(obj,field,newobj){
    if(newobj.mark == false){
        newobj.mark = true
        push(newobj,$mark_stack)
    }
    *field = newobj
}

```
����������õĶ����ǰ�ɫ������ֱ�Ӱ���ͿΪ��ɫ��
![image](https://github.com/shenlanse/learning-note/raw/master/images/barrier2.jpg)


mark��mark termination��α����Ϊ��


```
incremental_mark_phase(){
    for(i : 1...MARK_MAX){
        // �ֶ��mark������ҪSTW
        if(is_empty($mark_stack) == false){
            obj = pop($mark_stack)
            for(child : children(obj)){
                mark(*child)
            }
        }else{
            // mark termination����ҪSTW
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

sweepҲ�Ƿֶ�εģ�����ʽ�Ļ�����������mutator�������С������-����㷨��ʵ�ֻ���һ�£�Ҳ����Ҫ���������ѣ�����ɫ����ҵ����������ϣ���ɫ����ȡ��mark��ǡ�

##### 3.1.4 �����¶���

�ӿ���������䡣

#### 3.2 golangΪʲôû�в���ѹ���㷨�ͷִ��㷨

�е����������˽�Ŀ��Բο�[golang-nuts](https://groups.google.com/forum/#!topic/golang-nuts/KJiyv2mV2pU)��������ۡ�


---

### 4 Golang�������յ���ز���

#### 4.1 ����GC

gc������ʱ����2���ӻ����ڴ�ռ�ôﵽһ����ֵ����ǰ���ڴ�ռ�����ϴ�gc����ڴ�ռ�õ���������GOGC=100ʱ��

```
 # ��ʾ��ǰӦ��ռ�õ��ڴ����ϴ�GCʱռ���ڴ������ʱ������GC
export GOGC=100
```


#### 4.2 �鿴GC��Ϣ



```
export GODEBUG=gctrace=1
```


���Բ鿴gctrace��Ϣ��

������


```
gc 1 @0.008s 6%: 0.071+2.0+0.080 ms clock, 0.21+0.22/1.9/1.9+0.24 ms cpu, 4->4->3 MB, 5 MB goal, 4 P
# command-line-arguments
gc 1 @0.001s 16%: 0.071+3.3+0.060 ms clock, 0.21+0.17/2.9/0.36+0.18 ms cpu, 4->4->4 MB, 5 MB goal, 4 P
gc 2 @0.016s 8%: 0.020+6.0+0.070 ms clock, 0.082+0.094/3.9/2.2+0.28 ms cpu, 8->9->8 MB, 9 MB goal, 4 P
gc 3 @0.046s 7%: 0.019+7.3+0.062 ms clock, 0.076+0.089/7.1/7.0+0.24 ms cpu, 14->16->14 MB, 17 MB goal, 4 P
gc 4 @0.092s 8%: 0.015+24+0.10 ms clock, 0.060+0.10/24/0.75+0.42 ms cpu, 25->27->24 MB, 28 MB goal, 4 P

```
ÿ���ֶα�ʾʲô��Ϣ���Բο� [golang doc](https://godoc.org/runtime)

---

### 5 ������GC������

Golang��GC�㷨�ǹ̶��ģ��û��޷�ȥ���ò���ʲô�㷨��Ҳû����Javaһ�������������������Ŀռ�����ȡ�golang��GC��ص����ò���ֻ��һ������GOGC��������ʾ����GC��������

Ŀǰ���������GCЧ������Ψһ�����ľ��Ǽ��������Ĳ���������˵����һ�³�Ϊ���GC������Ҳ��̫���ʡ��������Ǿ���Ҫ����һ�£���golang����μ��������Ĳ���������Щ��Ҫע��ķ��档

#### 5.1 golang�е��ڴ����

�ο�����[Frequently Asked Questions (FAQ)](https://golang.org/doc/faq#stack_or_heap)

How do I know whether a variable is allocated on the heap or the stack? 

> From a correctness standpoint, you don't need to know. Each variable in Go exists as long as there are references to it. The storage location chosen by the implementation is irrelevant to the semantics of the language.
> 
> The storage location does have an effect on writing efficient programs. When possible, the Go compilers will allocate variables that are local to a function in that function's stack frame. However, if the compiler cannot prove that the variable is not referenced after the function returns, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors. Also, if a local variable is very large, it might make more sense to store it on the heap rather than the stack.
> 
> In the current compilers, if a variable has its address taken, that variable is a candidate for allocation on the heap. However, a basic escape analysis recognizes some cases when such variables will not live past the return from the function and can reside on the stack.


���ǿ�һ�������и�ֱ�۵���ʶ��


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


#### 5.2 sync.Pool�����

sync.Pool��Ҫ��Ϊ�����ö���һ��������������ռ��ʱ�䣬��һ���棬��������GC��ѹ������������һ����ʱ����أ�Ϊʲô��ô˵�أ���Ϊ������еĶ���ᱻGC���ա�����˵����״̬�Ķ��󣬱������ݿ������ǲ��ܹ���sync.Pool��ʵ�ֵġ�

>  use sync.Pool if you frequently allocate many objects of the same type and you want to save some allocation and garbage collection overhead. However, in the current implementation, any unreferenced sync.Pool objects are removed at each garbage collection cycle, so you can't use this as a long-lived free-list of objects. If you want a free-list that maintains objects between GC runs, you'll still have to build that yourself. This is only to reuse allocated objects between garbage collection cycles.


sync.Pool��Ҫ������������


```
func (p *Pool) Get() interface{} {
    ...
}
func (p *Pool) Put(x interface{}) {
    ...
}

```

Get������ָ����ʱ��������������put��ָ�Ѳ���ʹ�õĶ��󷵻ض���أ��Ա�������á����������ʹ��Get�����¶���ʱpool��û�п��õĶ�����ô�ͻ᷵��nil������������sync.Pool��New func��


```
type Pool struct {

    ...
    
	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() interface{}
}
```


���⣬���ǲ��ܶԴӶ�������뵽�Ķ���ֵ���κμ��裬������New�����ɵģ������Ǳ�ĳ��Э���޸Ĺ��Ż����ġ�


һ���ȽϺõ�ʹ��sync.Pool�����ӣ�


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


��һ����Դ��ͨ��golang�����ʵ�֣�����Ȥ�Ŀ����о�һ�£�[Go Commons Pool](https://github.com/jolestar/go-commons-pool)���ڴ˲���׸����


#### 5.3 append

�����ȿ�һ��append�Ļ����÷���

```
nums:=make([]int,0,10)
```

������Ƭ��len=0��cap=10���ײ�ʵ���Ϸ�����10��Ԫ�ش�С�Ŀռ䡣��û��append���ݵ�����£�����ֱ��ʹ��nums[index]��

---

```
nums:=make([]int,5,10)
```
������Ƭ��len=5��cap=10���ײ�ʵ���Ϸ�����10��Ԫ�ش�С�Ŀռ䡣��û��append���ݵ�����£�����ֱ��ʹ��nums[index]��index�ķ�Χ��[0,4]��ִ��append������ʱ���Ǵ�index=5��λ�ÿ�ʼ�洢�ġ�


---


```
nums := make([]int,5)
```
���û��ָ��capacity����ôcap��len��ȡ�

---


```
nums = append(nums,10)
```
ִ��append������ʱ��nums�ĵ�ַ���ܻ�ı䣬�����Ҫ�����䷵��ֵ��������nums������nums�ĵ�ַ�᲻��ı䣬ȡ���ڻ�û�пռ����洢�µ����ݣ����û�п��пռ��ˣ��Ǿ���Ҫ����cap*2�Ŀռ䣬�����ݸ��ƹ�ȥ��

��ˣ�������ʹ��append������ʱ�����������һ���ȽϺ����capֵ���������Լ���Ӧ�ó���Ԥ�����С���ʵĿռ䣬������ν�Ĳ������������¿ռ䣬�������Լ���GC��ѹ����

��append���µ��ڴ������GCѹ������������⣬��Ҫ�ر�ע��һ�¡�


### �ο�����

> 1 https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)    
> 2 http://legendtkl.com/2017/04/28/golang-gc/    
> 3 https://github.com/golang/proposal/blob/master/design/17503-eliminate-rescan.md    
> 4 https://lengzzz.com/note/gc-in-golang    
> 5 https://making.pusher.com/golangs-real-time-gc-in-theory-and-practice/    
> 6 https://blog.twitch.tv/gos-march-to-low-latency-gc-a6fa96f06eb7    
> 7 https://golang.org/doc/faq    
> 8 ���������յ��㷨��ʵ�֡� �д���� �ന��. ����