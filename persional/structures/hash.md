# hash
- 给定表M，存在函数f(key)，对任意给定的关键字值key，代入函数后若能得到包含该关键字的记录在表中的地址，则称表M为哈希（Hash）表，函数f(key)为哈希(Hash) 函数。

## hash冲突
两个不同的key通过同一个hash函数算出来的地址相同，此时就会发生冲突。
解决此冲突的方法是开放寻址法和拉链法（链表法）
### 开放寻址法
1. 存储：
   - hash出的地址如果已经被存储，则按顺序往后找空地址存储。
2. 读取：
   - 对键进行hash
   - 拿到全体格子的总数然后取模
   - 找到位置是1，但发现键key不一样，则按地址顺序往后查找。

### 拉链法/链表法
关键词：hash表的buckets
1. 存储：
   - 进行hash计算，算出存储地址，此存储地址只储存了一个指针，此指针存储了一个链表的地址 ，新存储的数据则会存储到这个链表（头）中，此链表也称为hash表的buckets
2. 读取：
   - 对键进行hash,找到对应的桶和对应的Tophash
     - 如果遍历完了TopHash没找到对应键值，则看看有没有溢出桶（overflow buckets）遍历溢出桶查找，直到找到对应的值或者遍历一遍未找到。
   
