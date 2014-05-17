### 21.1 隐式转换

	implicit def int2Fraction(n: Int) = Fraction(n, 1)
	val result = 3 * Fraction(4, 5) // Calls int2Fraction(3)

### 21.2 利用隐式转换丰富现有类库

	val contents = new java.io.File("README").read
	class RichFile(val from: File) {
	  def read = Source.fromFile(from.getPath).mkString
	}
	implicit def file2RichFile(from: File) = new RichFile(from)

### 21.3 引入隐式转换

scala考虑如下情况

- 位于源或目标类型的伴生对象中的隐式函数
- 位于当前作用域，可由单个标识符指代的隐式函数

#### tips

- REPL中可以用:implicits [-v] 查看引入的隐式转换
- scalac-Xprint:typer MyProg.scala 可以看到加入隐式转换后的源代码 
- 调试时可以把隐式转换的函数加上，查看编译器错误即可
- 可以用包排除的方式排除不需要的隐式转换 `import com.horstmann.impatient.FractionConversions.{fraction2Double => _, _}`

### 21.4 隐式转换规则

三种情况下有可能调用隐式转换

- 表达式类型与预期不同: `sqrt(Fraction(1, 4)) // sqrt期望Double类型，所以调用 fraction2Double`
- 对象访问一个不存在的成员:`new File("README").read // File没有read方法，所以调用file2RichFile`
- 对象调用某方法，该方法参数声明和传入参数不符的时候`3 * Fraction(4, 5) // int的乘法不接受fraction，所以会调用int2Fraction`

不过编译器不会尝试同时执行多个转换，并对二义性的转换报错

二义性规则只适用于被尝试转换的对象
	
	Fraction(3, 4) * 5
	下面两个表达式都有效，不过会优先使用前者，即参数转换优先
	Fraction(3, 4) * int2Fraction(5)
	fraction2Double(Fraction(3, 4)) * 5

### 21.5 隐式参数

	case class Delimiters(left: String, right: String)
	def quote(what: String)(implicit delims: Delimiters) =
	  delims.left + what + delims.right
	
	// 可以显式调用
	quote("Bonjour le monde")(Delimiters("«", "»"))
	// Returns «Bonjour le monde»
	
	quote("Bonjour le monde")
	//也可以略去隐式参数列表，这时编译器将会在下面两个地方查找类型为Delimiters的隐式(implicit)值
	// - 当前作用域所有可以用单个标识符取代的满足类型要求的val、def
	// - 要求类型关联的类型的伴生对象【关联类型包括所要求类型本身、及其参数(化)类型】
	object FrenchPunctuation {
	implicit val quoteDelimiters = Delimiters("«", "»")
	...
	}
	import FrenchPunctuation._
	import FrenchPunctuation.quoteDelimiters


对于给定的数据类型，只能有一个隐式值。所以不要使用常用的隐式值，像这样`def quote(what: String)(implicit left: String, right: String)`是不行的，因为调用者没法提供两个不同的字符串。

### 21.6 利用隐式参数进行隐式转换

	def smaller[T](a: T, b: T) = if (a < b) a else b // 不能工作
	def smaller[T](a: T, b: T)(implicit order: T => Ordered[T])
	  = if (a < b) a else b // 注意到order是隐式参数，如果a没有<方法，将调用order(a) < b 

这时候的smaller可以调用`smaller(40, 2)`, `smaller("Hello", "World")`，因为PreDef中有这些对象定义好的隐式转换。如果想使用`smaller(Fraction(1, 7), Fraction(2, 9))`，就需要定义一个`Fraction => Ordered[Fraction]`的函数，要么在定义的时候显式写出，要么做成一个`implicit val`

### 21.7 上下文界定

类型参数可以有一个形式为`T:M`的上下文界定，它要求作用域中有一个类型为M[T]的隐式值

	class Pair[T : Ordering](val first: T, val second: T) {
	  def smaller(implicit ord: Ordering[T]) =
	    if (ord.compare(first, second) < 0) first else second
	}

要求存在一个Ordering[T]的隐式值。这是如果new一个Pair(20, 30)，编译器推断我们需要一个Pair[Int],并使用Predef中的Ordering[Int]，Int满足上下文界定；Ordering[Int]就会作为该类的一个字段，传给需要该值的方法中。

两个重写，是等价的【前者使用Predef中的implicitly方法；后者使用ordered特质简化代码】

	class Pair[T : Ordering](val first: T, val second: T) {
	  def smaller =
	    if (implicitly[Ordering[T]].compare(first, second) < 0) first else second
	}
	
	class Pair[T : Ordering](val first: T, val second: T) {
	  def smaller = {
	    import Ordered._;
	    if (first < second) first else second
	  }
	}

重点是：你可以随时实例化Pair[T], 只要满足存在类型为Ordering[T]的隐式值条件即可。例如如果需要Pair[Point]，则可以创建一个隐式Ordering[Point]:

	implicit object PointOrdering extends Ordering[Point] {
	  def compare(a: Point, b: Point) = ...
	}

### 21.8 类型证明

TODO





























