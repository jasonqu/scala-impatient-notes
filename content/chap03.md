###  p33练习

给定一个数组缓冲，移除除第一个负数之外的所有负数

	val b = ArrayBuffer[Int]()
	b +=(1, 2, -3, 4, -5, -6, 0)
	func(b) == ArrayBuffer(1, 2, -3, 4, 0)

一个直观的做法是

	  def func(b : ArrayBuffer[Int]) = {
	    var flag = true;
	    val result = ArrayBuffer[Int]()
	    for(i <- b) {
	      if(i >= 0) {
	        result += i
	      } else if(flag) {
	        flag = false
	        result += i
	      } else {
	      }
	    }
	    result
	  }

更简洁的做法是：取所有负数的index，然后移除除了第一个负数外所有的负数

	val indexes = for (i <- 0 until a.size if a(i) < 0) yield i
	for (j <- (1 until indexes.size).reverse) a.remove(indexes(j))

### 3.8 java 互操作

array

	import collection.JavaConversions.asScalaBuffer
	import collection.JavaConversions.bufferAsJavaList

map

	import collection.JavaConversions.mapAsScalaMap
	import collection.JavaConversions.propertiesAsScalaMap
	import collection.JavaConversions.mapAsJavaMap
