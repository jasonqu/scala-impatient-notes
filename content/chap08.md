### 8.3 类型检查与转换

|| *scala*                || *desc*        || *java* ||
|| classOf[String]        || class literal || String.class ||
|| x.isInstanceOf[String] || type check (runtime) || x instanceof String ||
|| x.asInstanceOf[String] || type cast (runtime)  || (String) x ||

不过尽量使用pattern matching

### 8.6 重写字段限制

- def 只能重写另一个 def
- val 只能重写另一个 val 或者不带参数的def
- var 只能重写另一个 抽象的 var

### 8.7 匿名子类

实际是结构类型对象 

	val alien = new Person("Fred") {
	  override val toString = "Greetings, Earthling! My name is Fred."
	}

这个类的类型是 Person{val toString: String}， 这个类型也可以作为参数

### 8.10 构造顺序与提前定义 L3

	class Creature {
	  val range: Int = 10
	  val env: Array[Int] = new Array[Int](range)
	}
	class Ant extends Creature {
	  override val range = 2
	}

	scala> new Creature().env
	res47: Array[Int] = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0)
	
	scala> new Ant().env
	res48: Array[Int] = Array()

因为java设计允许在超类构造方法中使用子类的方法，所以初始化env的时候使用的range是Ant的未初始化的值，解决方法：

- val声明为final，安全但不灵活
- val声明为lazy，安全但低效
- 子类提前定义语法

代码

	class Bug extends {
	  override val range = 3
	} with Creature

练习9：如果Creative或Ant的range定义使用def会怎样呢？


### Tips

编译器允许任何值被替换为()

	def printAny(x: Any) { println(x) }
	def printUnit(x: Unit) { println(x) }
	printAny("Hello") // Prints Hello
	printUnit("Hello") // Replaces "Hello" with () and calls printUnit(()), which prints ()

