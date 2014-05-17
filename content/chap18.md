### 18.1 单例类型

对任何引用v，可以得到类型v.type。他有两个可能值v.type 和 null

如实现builder模式，最后返回自己的引用，就可以串起来了

	class Document {
	  def setTitle(title: String) = { ...; this }
	  def setAuthor(author: String) = { ...; this }
	}
	article.setTitle("Whatever Floats Your Boat").setAuthor("Cay Horstmann")

但是如果有子类型，就会发生错误；因为this的类型是Document

	class Book extends Document {
	  def addChapter(chapter: String) = { ...; this }
	  ...
	}
	val book = new Book()
	book.setTitle("Scala for the Impatient").addChapter(chapter1) // Error

解决方法是加上返回类型声明：

	def setTitle(title: String): this.type = { ...; this }

另一个用途是在接受单例对象为参数的情况下，从略

### 18.2-3 类型投影、路径 略

### 18.4 类型别名

	class Book {
	  import scala.collection.mutable._
	  type Index = HashMap[String, (Int, Int)]
	  ...
	}

### 18.5 结构类型

指的是一组关于抽象方法、字段和类型的规格声明。例如下面方法规定了参数必须带有append方法：

	def appendLines(target: { def append(str: String): Any },
	    lines: Iterable[String]) {
	  for (l <- lines) { target.append(l); target.append("\n") }
	}

幕后，scala使用反射来调用的target.append；所以开销也大很多。【结构类型像js和ruby中的Duck Type，在这些语言中，变量没有类型。但写下obj.quack()时，运行时会检查对象在那一刻是否具有quack方法；即不需要声明obj为Duck，只要它会呱呱叫就行】

### 18.6 复合类型

	T1 with T2 with T3 ...

### 18.7 中置类型

带有两个参数的类型，可以用中置表示法

	String Map Int <=> Map[String, Int] 
	type ×[A, B] = (A, B) // 卡氏积
	String x Int x Int <=> (String x Int) x Int

### 18.8 存在类型

带有两个参数的类型，可以用中置表示法

	Array[T] forSome { type T <: JComponent } <=> Array[_ <: JComponent]
	Map[_, _] <=> Map[T, U] forSome { type T; type U }
	Map[T, U] forSome { type T; type U <: T }

### 18.9 类型系统总结

表格略

	def square(x: Int) = x * x
	// 类型 square (x: Int)Int 方法
	val triple = (x: Int) => 3 * x
	// 类型 triple: Int => Int 函数
	square _
	// 类型 square Int => Int 函数

### 18.10 自身类型

表示特质只能混入给定类型的子类中：

	trait LoggedException extends Logged {
	  this: Exception =>
	    def log() { log(getMessage()) } // 可以调用getMessage因为this是一个Exception
	}

自身类型不会自动继承：

	trait ManagedException extends LoggedException {
	  this: Exception => // 不带这句会报错
	    ...
	}

### 18.11 依赖注入

例如Logger有两个实现：ConsoleLogger和FileLogger

认证有一个Logger的依赖：

	trait Auth {
	  this: Logger =>
	    def login(id: String, password: String): Boolean
	}

应用依赖于这两个特质

	trait App {
	  this: Logger with Auth => ...
	}

所以就可以这样组装：

	object MyApp extends App with FileLogger("test.log") with MockAuth("users.txt")

但是这样组合类型的方式，在处理复杂类型的时候可能会有问题；所以就有了蛋糕模式：

- 每个服务提供一个组件特质
- 任何依赖组件，以自身类型表述
- 描述服务接口的特质
- 一个抽象的val，将被初始化为服务的实例
- 也可以有选择性的包含服务接口实现

前面的例子可以这样实现：

	trait LoggerComponent {
	  trait Logger { ... }
	  val logger: Logger
	  class FileLogger(file: String) extends Logger { ... }
	  ...
	}
	trait AuthComponent {
	  this: LoggerComponent => // Gives access to logger
	  trait Auth { ... }
	  val auth: Auth
	  class MockAuth(file: String) extends Auth { ... }
	  ...
	}
	object AppComponents extends LoggerComponent with AuthComponent {
	  val logger = new FileLogger("test.log")
	  val auth = new MockAuth("users.txt")
	}

### 18.12 抽象类型

	trait Reader {
	  type Contents
	  def read(fileName: String): Contents
	}
	class StringReader extends Reader {
	  type Contents = String
	  def read(fileName: String) = Source.fromFile(fileName, "UTF-8").mkString
	}
	class ImageReader extends Reader {
	  type Contents = BufferedImage
	  def read(fileName: String) = ImageIO.read(new File(fileName))
	}

与下面代码功能等价

	trait Reader[C] {
	  def read(fileName: String): C
	}
	class StringReader extends Reader[String] {
	  def read(fileName: String) = Source.fromFile(fileName, "UTF-8").mkString
	}
	class ImageReader extends Reader[BufferedImage] {
	  def read(fileName: String) = ImageIO.read(new File(fileName))
	}

哪种方式好呢？scala经验法则：

- 如果类型在类被实例化的时候给出，使用类型参数。如构造HashMap[String, Int]时
- 如果类型是在子类中给出，使用抽象类型。例如这里的Reader

### 18.13 家族多态 略

### 18.14 高等类型 略

### 练习 TODO