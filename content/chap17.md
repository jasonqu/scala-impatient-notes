### 17.2 泛型函数

	def getMiddle[T](a: Array[T]) = a(a.length / 2)
	getMiddle(Array("Mary", "had", "a", "little", "lamb")) // Calls getMiddle[String]
	val f = getMiddle[String] _ // The function, saved in f

### 17.3 类型变量界定

Problem. 求Pair中较小的一个

	class Pair[T](val first: T, val second: T) {
	  def smaller = if (first.compareTo(second) < 0) first else second // Error
	}

上面代码没有标志比较运算，所以报错，可以这样修改，表示T是Ordered的子类【注意到相比Comparable，Ordered提供了类似<的方法】：

	class Pair[T <: Ordered[T]](val first: T, val second: T) {
	  def smaller = if (first < second) first else second // Error
	}

返回一个替换了first的新对偶，下面两个方法都可以返回由超类组成的对偶

	def replaceFirst[R >: T](newFirst: R) = new Pair[R](newFirst, second)
	def replaceFirst[R >: T](newFirst: R) = new Pair(newFirst, second)

如果不加上界，返回的是Pair[Any]

	def replaceFirst[R](newFirst: R) = new Pair(newFirst, second)

### 17.4 视图界定

前面的Pair不能用于Int，因为他没有继承，但是可以使用视图界定，因为有一个Int到RichInt的隐式转换

	class Pair[T <% Ordered[T]]

### 17.5 上下文界定

T:M 要求上下文中必须存在一个类型为M[T]的隐式值(implicit value)，它比隐式转换更为灵活

	class Pair3[T : Ordering](val first: T, val second: T) {
	  def smaller(implicit ord: Ordering[T]) =
	    if (ord.compare(first, second) < 0) first else second
	}

### 17.6 Manifest上下文界定

实例一个Array[T]，需要一个Manifest[T]对象，为了让数组能够在基本类型中使用。比如对Int，scala编译器会自动转为Int[]

	def makePair[T : Manifest](first: T, second: T) = {
	  val r = new Array[T](2); r(0) = first; r(1) = second; r
	}
	makePair(2, 3) // 实际调用makePair(2, 3)(intManifest) 即 int[2]

### 17.7 多重界定

	T <: Lower >: Lower
	T <: Comparable[T] with Serializable with Cloneable
	T <% Comparable[T] <% String
	T : Ordering : Manifest

### 17.8 类型约束

|| T =:= U || T是否等于U ||
|| T <:< U || T是否是U的子类型 ||
|| T <%< U || T是否能被视图(隐式)转换为U ||

并没有带来比类型界定更多的优点，只是适用于几处：

尽管File没有Ordered特质，但是一般是可用的，只有调用smaller时才报错

	class Pair[T](val first: T, val second: T) {
	  def smaller(implicit ev: T <:< Ordered[T]) =
	    if (first < second) first else second
	}

同上，Option的OrNull，用到了Null<:<A，可以实例化Option[Int]，但是基本类型是没有null的

	val friends = Map("Fred" -> "Barney", ...)
	val friendOpt = friends.get("Wilma") // An Option[String]
	val friendOrNull = friendOpt.orNull // A String or null

改进类型推断

def firstLast[A, C <: Iterable[A]](it: C) = (it.head, it.last)
firstLast(List(1, 2, 3)) // 报错，因为编译器无法推断A的类型
def firstLast[A, C](it: C)(implicit ev: C <:< Iterable[A]) = (it.head, it.last)
// 解决问题，先匹配C，再匹配A
def corresponds[B](that: Seq[B])(match: (A, B) => Boolean): Boolean
// 利用currying 先判定类型B，再判定类型A

### 17.11 对象不能泛型

例如空队列最好是一个单例对象

	object Nil[T] extends List[T]

但是对象不能泛型，所以解决方法是

	object Nil extends List[Nothing]

### 17.12 类型通配符 略




