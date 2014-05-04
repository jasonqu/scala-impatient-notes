### Tips

scala中使用的是相对包名，如果要用绝对包名，需要以_root_

### 7.5 包对象

使用包对象可以放置一些工具方法和值

	package com.horstmann.impatient
	package object people {
	  val defaultName = "John Q. Public"
	}
	package people {
	  class Person {
	    var name = defaultName // A constant from the package
	  }
	}

### 7.9 重命名和隐藏

通过重命名区分java.util.HashMap 和 scala.collection.mutable.HashMap

	import java.util.{HashMap => JavaHashMap}
	import scala.collection.mutable.HashMap

也可以隐藏掉java.util.HashMap

	import java.util.{HashMap => _, _}

### 练习3 线性同余随机数生成

	package object random {
		var seed : Int = 0
	
		def setSeed(value: Int) = seed = value
		def nextInt() = {
			seed = seed * 1664525 + 1013904223 % (2 ^ 32)
			seed
		}
		def nextDouble() : Double = nextInt().toDouble / Int.MaxValue.toDouble
	}
