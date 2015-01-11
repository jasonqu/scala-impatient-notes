# chapter2

### 2.8 默认参数和带名参数

java中的函数重载可以使用默认参数轻松实现：

	def decorate(str: String, left: String = "[", right: String = "]") =
		left + str + right
	
	scala> decorate("Hello")
	res10: String = [Hello]
	
	scala> decorate("Hello", "<<<", ">>>")
	res11: String = <<<Hello>>>

	scala> decorate("Hello", ">>>[")
	res12: String = >>>[Hello]

甚至可以在参数上带上名字：

	scala> decorate(left = "<<<", str = "Hello", right = ">>>")
	res13: String = <<<Hello>>>

主构造函数可以通过默认参数来减少辅助构造函数

	class Pserson(val name : String = "", val age : Int = 0)


### 2.9 变长参数

对变长参数，序列可以使用:_*来转换成参数序列

	def sum(args: Int*) = {
	  for(arg <- args) print(arg + ";") 
	}
	
	scala> sum(1,2,3)
	1;2;3;
	
	scala> sum(1 to 5 :_*)
	1;2;3;4;5;

### 2.11 lazy val

val 声明为 lazy的时候，初始化将被延至首次被调用的时候

	val words = io.Source.fromFile("/usr/share/dict/words").mkString
	// 定义时取值 Evaluated as soon as words is defined
	lazy val words = io.Source.fromFile("/usr/share/dict/words").mkString
	// 首次使用时取值，开销是每次调用都会有一个检查
	def words = io.Source.fromFile("/usr/share/dict/words").mkString
	// 每次使用时取值

# Chapter 7

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

# Chapter 8


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

### 关于Unit

编译器允许任何值被替换为()

	def printAny(x: Any) { println(x) }
	def printUnit(x: Unit) { println(x) }
	printAny("Hello") // Prints Hello
	printUnit("Hello") // Replaces "Hello" with () and calls printUnit(()), which prints ()

# Chapter 9

### scala io

	import scala.io.Source
	val source = Source.fromFile("myfile.txt", "ISO-8859-1")
	val lineIterator = source.getLines
	
	val lines : Array[String] = source.getLines.toArray	
	val contents : String = source.mkString

写文本

	val out = new java.io.PrintWriter("numbers.txt")
	for (i <- 1 to 100) out.println(i)
	out.close()

### 9.5 读取二进制 java库

	val file = new File(filename)
	val in = new FileInputStream(file)
	val bytes = new Array[Byte](file.length.toInt)
	in.read(bytes)
	in.close()

### 9.7 访问目录

遍历所有子目录

	import java.io.File
	def subdirs(dir: File): Iterator[File] = {
	  val children = dir.listFiles.filter(_.isDirectory)
	  children.toIterator ++ children.toIterator.flatMap(subdirs _)
	}

	for (d <- subdirs(dir)) process d

或者使用java7的java.nio.file.Files中的walkFileTree

	import java.nio.file._
	implicit def makeFileVisitor(f: (Path) => Unit) = new SimpleFileVisitor[T] {
	  override def visitFile(p: Path, attrs: attribute.BasicFileAttributes) = {
	    f(p)
	    FileVisitResult.CONTINUE
	  }
	}

	Files.walkFileTree(dir.toPath, (f: Path) => println(f))

### 9.9 进程控制

可以使用scala方便的编写shell脚本

	import sys.process._
	"ls" !

sys.process包含一个字符串到ProcessBuilder的隐式转换，！就是执行，成功执行返回0，否则是1

其它特性（| > >> < && ||）

	val result = "ls" !！ // 获取执行输出字符串
	"ls" #| "grep x" !    // 管道
	"ls" #> new File("out.txt") ! // 重定向
	"ls" #>> new File("out.txt") ! // 追加
	"grep x" #< new File("out.txt") ! // 输入
	"grep x" #< new Url("http://google.com") ! // 输入

直接构造ProcessBuilder，给出起始目录、环境变量和命令

	val p = Process(cmd, new File(dirname）, ("LANG", "en_US"))
	"ls" #| p !

### 9.10 Regex

	val numPattern = "[0-9]+".r
	val wsnumwsPattern = """\s+[0-9]+\s+""".r
	// A bit easier to read than "\\s+[0-9]+\\s+".
	
	// findAllIn returns MatchIterator, can be converted to Array
	val matches = numPattern.findAllIn("99 bottles, 98 bottles").toArray
	
	val m1 = wsnumwsPattern.findFirstIn("99 bottles, 98 bottles")
	// Some(" 98 ")
	
	numPattern.findPrefixOf("99 bottles, 98 bottles")
	// Some(99)
	wsnumwsPattern.findPrefixOf("99 bottles, 98 bottles")
	// None
	
	numPattern.replaceFirstIn("99 bottles, 98 bottles", "XX")
	// "XX bottles, 98 bottles"
	numPattern.replaceAllIn("99 bottles, 98 bottles", "XX")
	// "XX bottles, XX bottles"

### 9.11 regex groups

regex可以被当做提取器，来提取组

	val numitemPattern = "([0-9]+) ([a-z]+)".r
	val numitemPattern(num, item) = "99 bottles"
	// sets num to "99", item to "bottles"
	for (numitemPattern(num, item) <- numitemPattern.findAllIn("99 bottles, 98 bottles"))


### 练习8

使用regex group 打印某网页中所有img标签的src属性

	val html = io.Source.fromURL("http://horstmann.com", "UTF-8").mkString
	
	val srcPattern = """(?is)<\s*img[^>]*src\s*=\s*['"\s]*([^'"]+)['"\s]*[^>]*>""".r
	
	for(srcPattern(s) <- srcPattern findAllIn html) println(s)

# Chapter 10 Trait
