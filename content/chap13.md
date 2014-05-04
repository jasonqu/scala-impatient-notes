### Tips

	Iterable 【iterator方法】
	^
	|-------------|-----------|
	Seq           Set         Map
	^             ^           ^
	|             |           |
	IndexedSeq    SortedSet   SortedMap

IndexedSeq 子类 不可变：Vector、Range；可变： ArrayBuffer

Seq 子类【except IndexedSeq】 不可变：List、Stream、Stack、Queue；可变：Stack、Queue、Priority Queue、LinkedList、Double LinkedList

Set操作union【|或++】、intersect【&】、diff【&~或--】

若对flatMap传入函数返回值为Option，最终返回的集合是v值集合

collect用于偏函数partial function

zipWithIndex在需要获取下标时很有用

### 13.10

foldleft、/:

	list.foldleft(0)(_+_) <=> (0 /: list)(_+_)

获取词频的方式：

	val freq = scala.collection.mutable.Map[Char, Int]()
	for (c <- "Mississippi") freq(c) = freq.getOrElse(c, 0) + 1
	
	(Map[Char, Int]() /: "Mississippi") {
	  (m, c) => m + (c -> (m.getOrElse(c, 0) + 1))
	}

scanLeft 和scanRight得到的是中间结果的和：

	(1 to 10).scanLeft(0)(_ + _)

### 13.13 流

迭代器相对于集合是一个lazy实现，但是每次next都会改变迭代器指向。Stream则是其不可变替代品——tail lazy-evaluated immutable list.

<code>#::</code> 就像 <code>::</code>，不过构建的是流。

流的tail会对下一元素求值，想得到多个答案，可以调用take，然后使用force：<code>squares.take(5).force</code>

Source.getLines 返回Iterable，可以调用toStream来反复使用。

### 13.14 懒视图

对集合使用view方法将得到懒执行的集合，但是没有缓存，再调用会被再执行。

	val powers = (0 until 1000).view.map(pow(10, _))
	
	powers(100)

### 13.15 java 集合互操作 略

### 13.17 并行集合

scala可以轻松实现大集合的并行计算，例如对coll这个大集合求和，可以这样写

	coll.par.sum
	coll.par.count(_ % 2 == 0)
	for (i <- (0 until 100).par) print(i + " ") // 乱序
	for (i <- (0 until 100).par) yield i + " "  // 顺序

par返回的是ParSeq、ParSet、ParMap trait；可以用ser将他们在串行化

par能接受的操作必须是顺序无关的，复杂的操作可以使用fold或aggregate实现

### 练习8

将Array按传入的数字分列，如Array(1,2,3,4,5,6,7) 3 变成 Array(Array(1, 2, 3), Array(4, 5, 6), Array(7))

	Array(1,2,3,4,5,6,7).grouped(3).toArray
	Array(1,2,3,4,5,6,7).sliding(3).toArray // 可以看看sliding

