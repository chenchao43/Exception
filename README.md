## 7.1处理错误
假设程序运行期间出现错误，那么应该
* 返回到一种安全状态，并且能够让用户执行一些其他命令
* 或允许用户保存所有操作的结果，妥善终止程序


### 常见的错误 

* 用户输入错误：输入语法错误
* 设备错误：打印机被关，网络错误
* 物理限制：硬盘满
* 代码错误：返回-1这种方法并不通用

异常分类:所有异常都继承与Throwable类的一个实例
Throwable

Error:描述了java运行时系统的内部错误或者资源耗尽错误。应用程序不应该抛出这种类型的对象。如果出现了这样的内部错误，除了通告给用户，并尽力是程序安全的终止以外，再也无能为力了。这种情况很少出现。

Exception:关注点，RuntimeException和IOException，程序本身没有问题，I/O导致的错误属于IOException，程序错误的导致的异常是RuntimeException

IOException:由IO错误等导致的异常
打开一个不存在的文件，试图在文件尾读取数据等，取决于环境而不是取决于代码的异常

RuntimeException出现了，一定是程序的问题，比如数组越界，访问空指针，错误的类型转换

声明受查异常（除了error和RuntimeException），非受查异常要么不可控制，要么应该避免发生

throw new EOFException（）；
//或者
EOFException e=new EOFException();
throw e;
//下面将这些代码放在一起：
String readData（Scanner in）throws EOFException
{
    while(...)
    {
        if(!in.hasNext()) //EOF encounter
        {
            if(n<len)
                throw new EOFException();
            /*
                EOFException还有一个带有字符串型参数的构造器
                String gripe="a";
                throw new EOFException(gripe);
            */
        }
    }
    return s;
}


创建异常类

一般定义两个构造器，一个是默认的构造器，一个是带有详细描述信息的的构造器
class FileFormatException extends IOException
{
    public FileFormatException(){};
    public FileFormatException(String gripe)
    {
        super(gripe);
    }
}

java.lang.Throwable()
Throwable()
  //构造一个新的Throwable对象，这个对象没有详细的描述信息
Throwable(String message)
//构造一个新的throwable对象，这个对象带有特定的详细描述信息。习惯上,所有派生的异常类都支持一个默认的构造器和一个带有详细描述信息的构造器
String getMessage（）
//获得Throwable对象的详细描述信息


捕获异常

如果抛出了受查异常，必须对它进行处理，或者继续传递。

继续传递：public void read（String filename）throws IOException

处理
try{

}
catch(IOException e){

    e.getMassage()
    }
    
finally语句 
当代码中抛出一个异常时，就会终止方法中的剩余代码的处理，并推出这个方法的执行。如果方法获得了一些本地资源，并且只有这个方法自己知道，有如果这些资源在退出方法之前必须被收回，那么就会产资源回收问题。一种解决方案是捕获并重新抛出所有的异常。但是这种解决方案比较乏味，这是因为需要在两个地方清除所分配的资源。一个在正常的代码中，另一个是在异常的代码中。 
Java有一种更好的解决方案，这就是finally子句，
不管是否有异常被捕获，finally子句中的代码都被执行。
try语句可以只有finally子句，而没有catch子句

建议使用下面方式
InputStream in=...;
try
{
    try
    {
        code that might throw exception
    }
    finally
    {
        in.close();
    }
}
catch(IOException e)
{
    show error message;
}
//内层的try语句块只有一个职责，就是确保关闭输入流，外层的try语句块也只有一个职责，就是确定报告中出现的错误。
最好是使用带资源的try语句，因为抛出错误执行finally时，遇到资源关闭错误，原始错误会丢失，会抛出资源错误，

try（Scanner in =new Scanner (new FileInputStream("...")),"")
{
    while(in.hasNext())
        System.out.println(in.next());
}

分析堆栈轨迹元素 
堆栈轨迹（stack trace）是一个方法调用过程的列表，它包含了程序执行过程中方法调用的特定位置

一种更灵活的方法是使用getStackTrace方法，他会得到StackTraceElement对象的一个数组，可以在你的程序中分析这个对象数组。 
例如：

Throwable t=new Throwable（）；
StackTraceElement[] frames=t.getStackTrace();
for(StackTraceElement frame:frames)
    analyst frame;


使用异常机制的技巧 
目前，存在着大量有关如何恰当的使用异常机制的争论。 
下面给出几个使用异常机制的技巧 
1、异常处理不能代替简单的测试 
例如检测栈是否为空，应当用isEmpty（），捕获异常时间长，因此使用异常的基本规则是，只在异常情况下使用异常机制 
2、不要过分细化异常 

3、不要压制异常 
关闭异常的方法（出现异常也会视而不见）

记录日志

基本日志
Logger.getGlobal().info("File->opened");
Logger.getGlobal().setLevel(Level.off);//设置日志级别

实际中不可能所有日志都记录在全局的日志中：
可以调用getLogger方法来串讲或者获取记录器

public static final Logger mylogger=Logger.getLogger("helo");

未被任何变量引用的日志记录其可能会被垃圾回收，所以要用一个静态变量存储日志记录器的一个引用 

与包名类似，日志记录器名也有层次结构。事实上，与包名相比，日志记录器的层次性更强。对于包名来说，一个包的名字与其父包的名字之间没有语义关系，但是日志记录器的父与子之间将共享某些属性。例如：如果对com.mycompany日志记录器设置了日志级别，他的子记录器也会继承这个级别 
通常，有以下7个日志记录器级别

SEVERE
WARNING
INFO
CONFIG
FINE
FINER
FINEST 


logger.warning(message)
logger.log(Level.FINE,message); 

  默认的日志配置记录了INFO以及以上跟高级别的所有记录。因此，应该使用CONFIG，FINE、FINER，FINEST记录级别来记录那些有助于诊断，但对程序员没有太大意义的调试信息。 
  如果将记录级别设计为INFO或者更低，则需要修改日志处理器的配置。默认的日志处理器不会处理低于INFO级别的信息 
  默认的日志记录将显示包含日志调用的类名和方法名，如同堆栈队显示的信息那样。但是，如果虚拟机对执行过程进行了优化，就得不到准确的调用信息。此时，可以调用logp方法获得调用类的方法和确切位置，这个方法的签名为：

    void logp(Level 1,String className , String methodName , String message)

    logp
    public void logp(Level level,
                     String sourceClass,
                     String sourceMethod,
                     String msg)
    记录消息，指定源类和方法，没有参数。
    如果当前对于给定的消息级别启用了记录器，则给定的消息将转发给所有注册的输出处理程序对象。

    参数
    level - 消息级标识符之一，例如SEVERE
    sourceClass - 发出日志记录请求的类的名称
    sourceMethod - 发出日志记录请求的方法的名称
    msg - 字符串消息（或消息目录中的键）
