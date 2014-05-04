### scala io

	import scala.io.Source
	val source = Source.fromFile("myfile.txt", "ISO-8859-1")
	val lineIterator = source.getLines
	
	val lines : Array[String] = source.getLines.toArray
	
	val contents : String = source.mkString
	
	val source1 = Source.fromURL("http://horstmann.com", "UTF-8")
	val source2 = Source.fromString("Hello, World!")
	val source3 = Source.stdin

### 9.5 读取二进制 java库

	val file = new File(filename)
	val in = new FileInputStream(file)
	val bytes = new Array[Byte](file.length.toInt)
	in.read(bytes)
	in.close()

### 9.6 写文本 Java库

val out = new java.io.PrintWriter("numbers.txt")
for (i <- 1 to 100) out.println(i)
out.close()

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

