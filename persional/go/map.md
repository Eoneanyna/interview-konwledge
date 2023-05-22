# Map
- runtime.Map（线程不安全）
- sync.Map（线程安全）

定义：map 是由 key-value 对组成的；key 只会出现一次

map 的设计也被称为 “The dictionary problem”，它的任务是设计一种数据结构用来维护一个集合的数据，并且可以同时对集合进行增删查改的操作。最主要的数据结构有两种：`哈希查找表（Hash table）`、`搜索树（Search tree）`。

1. [哈希查找表](../structures/hash.md)用一个哈希函数将 key 分配到不同的桶（bucket，也就是数组的不同 index）。这样，开销主要在哈希函数的计算以及数组的常数访问时间。在很多场景下，哈希查找表的性能很高。
   
2. 搜索树法一般采用自平衡搜索树，包括：[AVL 树，红黑树](../structures/tree.md)。面试时经常会被问到，甚至被要求手写红黑树代码，很多时候，面试官自己都写不上来，非常过分。

   - 自平衡搜索树法的最差搜索效率是 O(logN)，而哈希查找表最差是 O(N)。当然，哈希查找表的平均查找效率是 O(1)，如果哈希函数设计的很好，最坏的情况基本不会出现。

还有一点，遍历自平衡搜索树，返回的 key 序列，一般会按照从小到大的顺序；而哈希查找表则是乱序的。

map 的一个关键点在于，哈希函数的选择。在程序启动时，会检测 cpu 是否支持 aes，如果支持，则使用 aes hash，否则使用 memhash。

![img_1.png](img/img_1.png)
上图就是 bucket 的内存模型，HOB Hash 指的就是 top hash。 注意到 key 和 value 是各自放在一起的，并不是 key/value/key/value/... 这样的形式。源码里说明这样的好处是在某些情况下可以省略掉 padding 字段，节省内存空间。

例如，有这样一个类型的 map：
```
map[int64]int8
```
如果按照 key/value/key/value/... 这样的模式存储，那在每一个 key/value 对之后都要额外 padding 7 个字节；而将所有的 key，value 分别绑定到一起，这种形式 key/key/.../value/value/...，则只需要在最后添加 padding。

每个 bucket 设计成最多只能放 8 个 key-value 对，如果有第 9 个 key-value 落入当前的 bucket，那就需要再构建一个 bucket ，通过 overflow 指针连接起来。




## runtime.Map中的哈希函数
### 存储
key 经过哈希计算后得到哈希值，共 64 个 bit 位（64位机，32位机就不讨论了，现在主流都是64位机），计算它到底要落在哪个桶时，只会用到最后 B 个 bit 位。还记得前面提到过的 B 吗？如果 B = 5，那么桶的数量，也就是 buckets 数组的长度是 2^5 = 32。
### 读取
1. 当我们在遍历 map 时，并不是固定地从 0 号 bucket 开始遍历，每次都是从一个随机值序号的 bucket 开始遍历，并且是从这个 bucket 的一个随机序号的 cell 开始遍历。这样，即使你是一个写死的 map，仅仅只是遍历它，也不太可能会返回一个固定序列的 key/value 对了。
   - 多说一句，“迭代 map 的结果是无序的”这个特性是从 `go 1.0 `开始加入的。
### 扩容(每次扩容为之前的两次幂)
#### 太多的溢出桶有什么问题？
    - 能形成非常长的链表，造成性能下降


#### 扩容步骤
1. 判断溢出桶的数量太多或者溢出的因子达到了最大上限，
```go
type hmap struct {
// Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
// Make sure this stays in sync with the compiler's definition.
count     int // # live cells == size of map.  Must be first (used by len() builtin)
flags     uint8
B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
hash0     uint32 // hash seed

buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

extra *mapextra // optional fields
}

//bucketCnt=8，loadFactorNum=13,loadFactorDen=2
//溢出因子判断函数
// overLoadFactor reports whether count items placed in 1<<B buckets is over loadFactor.
func overLoadFactor(count int, B uint8) bool {
return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
```
2. 原来的桶放入oldbuckets中,make了一个新桶，并将新桶赋值到buckets,并重新更新B的值。
   - 数据的迁移：growWork()函数把旧桶数据迁移到新桶，然后再把旧桶gc

- 计算方法：
哈希函数计算：key+hashseed(即hash0)获得一个二进制数
例如，现在有一个 key 经过哈希函数计算后，得到的哈希结果是：

    ```
    10010111 | 000011110110110010001111001010100010010110010101010 │ 01010
    ```
    当B=5时，用最后的 5 个 bit 位，也就是 01010，值为 10，也就是 10 号桶。这个操作实际上就是取余操作，但是取余开销太大，所以代码实现上用的位操作代替。

    再用哈希值的高 8 位，找到此 key 在 bucket 中的位置（`TopHash`），这是在寻找已有的 key。最开始桶内还没有 key，新加入的 key 会找到第一个空位，放入。

![img.png](img/img3.png)

    上图中，假定 B = 5，所以 bucket 总数就是 2^5 = 32。首先计算出待查找 key 的哈希，使用低 5 位 00110，找到对应的 6 号 bucket，使用高 8 位 10010111，对应十进制 151，在 6 号 bucket 中寻找 tophash 值（HOB hash）为 151 的 key，找到了 2 号槽位，这样整个查找过程就结束了。

    如果在 bucket 中没找到，并且 overflow 不为空，还要继续去 overflow bucket 中寻找，直到找到或是所有的 key 槽位都找遍了，包括所有的 overflow bucket。

## sync.map
有一个互斥锁 Mutex,在同一时刻只能有一个线程操作写map。
```go
// The zero Map is empty and ready for use. A Map must not be copied after first use.
type Map struct {
	mu Mutex

	// read contains the portion of the map's contents that are safe for
	// concurrent access (with or without mu held).
	//
	// The read field itself is always safe to load, but must only be stored with
	// mu held.
	//
	// Entries stored in read may be updated concurrently without mu, but updating
	// a previously-expunged entry requires that the entry be copied to the dirty
	// map and unexpunged with mu held.
	read atomic.Value // readOnly

	// dirty contains the portion of the map's contents that require mu to be
	// held. To ensure that the dirty map can be promoted to the read map quickly,
	// it also includes all of the non-expunged entries in the read map.
	//
	// Expunged entries are not stored in the dirty map. An expunged entry in the
	// clean map must be unexpunged and added to the dirty map before a new value
	// can be stored to it.
	//
	// If the dirty map is nil, the next write to the map will initialize it by
	// making a shallow copy of the clean map, omitting stale entries.
	dirty map[interface{}]*entry

	// misses counts the number of loads since the read map was last updated that
	// needed to lock mu to determine whether the key was present.
	//
	// Once enough misses have occurred to cover the cost of copying the dirty
	// map, the dirty map will be promoted to the read map (in the unamended
	// state) and the next store to the map will make a new dirty copy.
	misses int
}
```
### 写
```go
package main
import "sync"
func main(){
var m sync.Map
m.Store("key","value")
}
```
1. 先查找map中是否有key，有则对比value，相同则直接返回，否则加锁写入
   - 写入过程中，会再次read读取map的key，如果有则判断是否被删除，删除存入dirty中.未删除则更新value
   - 如果read没读到则判断是否在dirty的map,若在则更新dirty中的值value
   - 如果read和dirty都没有这个key,判断read.amended为真则同步更新read和dirty的数据，使两个map相等。


## map线程读写与删除
map 并不是一个线程安全的数据结构。同时读写一个 map 是未定义的行为，如果被检测到，会直接 panic。
- 上面说的是发生在多个协程同时读写同一个 map 的情况下。 如果在同一个协程内边遍历边删除，并不会检测到同时读写，理论上是可以这样做的。但是，遍历的结果就可能不会是相同的了，有可能结果遍历结果集中包含了删除的 key，也有可能不包含，这取决于删除 key 的时间：是在遍历到 key 所在的 bucket 时刻前或者后。

## float可作為map的kay嗎？
float 型可以作为 key，但是由于精度的问题，会导致一些诡异的问题，慎用之。

答：可以，Go 语言中只要是可比较的类型都可以作为 key。除开 slice，map，functions 这几种类型，其他类型都是 OK 的
当用 float64 作为 key 的时候，先要将其转成 uint64 类型，再插入 key 中。
## slice 和 map 分别作为函数参数时有什么区别？
- makemap 和 makeslice 的区别，带来一个不同点：当 map 和 slice 作为函数参数时，在函数参数内部对 map 的操作会影响 map 自身；而对 slice 却不会（之前讲 slice 的文章里有讲过）。
    - 主要原因：一个是指针（*hmap），一个是结构体（slice）。Go 语言中的函数传参都是值传递，在函数内部，参数会被 copy 到本地。*hmap指针 copy 完之后，仍然指向同一个 map，因此函数内部对 map 的操作会影响实参。而 slice 被 copy 后，会成为一个新的 slice，对它进行的操作不会影响到实参。

## 比较map相等
```
Amap == Bmap 只能比较出两个是不是都为空
```
map 深度相等的条件：
```
1、都为 nil
2、非空、长度相等，指向同一个 map 实体对象
3、相应的 key 指向的 value “深度”相等
```
因此只能是遍历map 的每个元素，比较元素是否都是深度相等。

## Map的初始化
```
m := make(map[string]int,16)  
```
make调用了runtime.makemap(SB)


