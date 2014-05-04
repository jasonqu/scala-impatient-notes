### 6.3 扩展类或特质的对象

一句话，对象可以继承。

例如下面的单例对象表示未实现

	abstract class UndoableAction(val description: String) {
	  def undo(): Unit
	  def redo(): Unit
	}
	
	object DoNothingAction extends UndoableAction("Do nothing") {
	  override def undo() {}
	  override def redo() {}
	}
	
	val actions = Map("open" -> DoNothingAction, "save" -> DoNothingAction, ...)
	// Open and save not yet implemented

### 6.4 apply

伴生对象的apply作为工厂方法出现，可以省去new关键字。如Array、List etc

### tips

对App特质可以使用-Dscala.time显示执行时间

### 6.6 Enumeration

http://stackoverflow.com/questions/1898932/case-classes-vs-enumerations-in-scala

