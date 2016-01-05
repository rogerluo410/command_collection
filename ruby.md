> http://www.kuqin.com/rubycndocument/man/built-in-class/class_object_module.html#method_defined.3f   --document
> https://ruby-hacking-guide.github.io/  --Ruby Hacking Guide

###Ruby 基础相关
1. each 和 map 的区别 : map 会返回新的对象  
    http://stackoverflow.com/questions/9586989/difference-between-map-and-each  
    http://nikitasinghblogger.blogspot.com/2013/08/difference-between-collect-select-map.html  
2. proc, lambda, 和 Proc 的区别
3. alias 的用法, alias 与 alias_method 的区别
4. 用过那些 gem, 简单的说一说各自的用途
5. 画个图, 体现一下Ruby的对象体系

###Ruby 与 Rails 进阶相关   
1. yield self 的用法, 写个例子  
把自身当一个块进行释放：  
```
  spec = Gem::Specification.new do | s |
      s.name    = "My Gem"
      s.version = "0.0.1"
  end
  
  module Gem
    class Specification
       def initialize 
          yield self if block_given?
    end
  end     
```
2. scope 的实现原理  
3. ActiveSupport::Concern 模块的用法及源码解读  
4. alias_method_chain 的用法及源码解读   

> http://naixspirit.github.io/2013/08/13/a-ruby-face-questions/

###REST / JSON / XML-RPC / SOAP
REST mandates the general semantics and concepts. The transport and encodings are up to you. They were originally formulated on XML, but JSON is totally applicable.

XML-RPC / SOAP are different mechanisms, but mostly the same ideas: how to map OO APIs on top of XML and HTTP. IMHO, they're disgusting from a design view. I was so relieved when found about REST. In your case, i'm sure that the lots of layers would mean a lot more CPU demand.

I'd say go REST, using JSON for encoding; but if your requirements are really that simple as just uploading, then you can use simply HTTP (which might be RESTful in design even without adding any specific library)

> http://stackoverflow.com/questions/1371312/rest-json-xml-rpc-soap


###ruby的hash variable

It's similar with single map in C++, separated by '=>' between key and value, the structured data like this :  
```
student = {
              "name" => 'xxx',
              "age"  => 22,
              "sex"  => "Male"
          }
```
Each hash object is an instance of Hash class, so we can create hash object through :  
```
  student = Hash.new
```
Commonly using functions :   
```
  size()   return how many records.
  length() return the number of elements in one record.
  include?(key)
  has_key?(key)
  delete(key)
  keys() return an array which is made up of all keys in hash object.
  values() 
```
Add new item to hash object :   
```
 hash[:newKey] = "newValue"
```

If you want to add new items from another hash - use merge method:
```
hash = {:item1 => 1}
another_hash = {:item2 => 2, :item3 => 3}
hash.merge(another_hash) # {:item1=>1, :item2=>2, :item3=>3}
```
In your specific case it could be:
```
hash = {:item1 => 1}
hash.merge({:item2 => 2}) # {:item1=>1, :item2=>2}
```
but it's not wise to use it when you should to add just one element more.

Pay attention that merge will replace the values with the existing keys:
```
hash = {:item1 => 1}
hash.merge({:item1 => 2}) # {:item1=>2}
```
exactly like hash[:item1] = 2

Also you should pay attention that merge method (of course) doesn't effect the original value of hash variable - it returns a new merged hash. If you want to replace the value of the hash variable then use merge! instead:
```
hash = {:item1 => 1}
hash.merge!({:item2 => 2})
now hash == {:item1=>1, :item2=>2}
```

- **Traversing the hash**
```
  each: Iterating over all elements in hash including key and value, if using one parameter in loop only, traversing all keys and all values in order.
  each_key: Iterating over all keys in hash
  each_pair: Iterating all key-value pair in hash
  each_value: Iterating over all values in hash

  student.each do | key , value |
     do something
  end
  
  student.each_pair do | key , value |
     do something
  end
```
If you define hash like this with symbol :  
```
  h = { :name=> "roger", :conf => "Windy" }
  p h[:name] #=> roger #just can visit by symbol , not with string "name" which will return nil
  p h[:conf] #=> Windy 
  p h["name"] #=> nil
  p h[:conf]  #=> nil  :name and "name" are different keys.
```



###Symbol object
It's similar with Marco in C++, the difference is that Marco makes effect on compiling time, but symbol object has effect on running time.

Symbol object has a symbol list in Ruby, one named symbol has only one copy, different variables or functions or classes can be bound to symbol object.

```
x = :sym
y = :sym
puts (x.__id__ == y.__id__ ) && ( :sym.__id__ == x.__id__) # => true

x = "string"
y = "string"
puts (x.__id__ == y.__id__ )        # => false
puts ( "string".__id__ == x.__id__) # => false
```

So, however you create a symbol object, as long as its contents are the same, it will refer to the same object in memory. This is not a problem because a symbol is an immutable object. Strings are mutable.

Edit: (In response to the comment below)

In the original article, the value is not being stored in a symbol, it is being stored in a hash. Consider this:
```
hash1 = { "string" => "value"}
hash2 = { "string" => "value"}
```
This creates six objects in the memory -- four string objects and two hash objects.


```
hash1 = { :symbol => "value"}
hash2 = { :symbol => "value"}
```
This only creates five objects in memory -- one symbol, two strings and two hash objects.

In short, symbols are lightweight strings, but they also are immutable and non-garbage-collectable.
You should not use them as immutable strings in your data processing tasks (remember, once symbol is created, it can't be destroyed). You typically use symbols for naming things.

```
# typical use cases

# access hash value
user = User.find(params[:id])

# name something
attr_accessor :first_name

# set hash value in opts parameter
db.collection.update(query, update, multi: true, upsert: true)  
```

Let's take first example, params[:id]. In a moderately big rails app there maybe hundreds/thousands of those scattered around the code. If we accessed that value with a string, params["id"], that means new string allocation each time (and that string needs to be collected afterwards). In case of symbol, it's actually the same symbol everywhere. Less work for memory allocator, garbage collector and even you (: is faster to type than "")

If you have a simple one-word string that appears often in your code and you don't do something funky to it (interpolation, gsub, upcase, etc), then it's likely a good candidate to be a symbol.

However, does this apply only to text that is used as part of the actual program logic such as naming, not text that you get while actually running the program...such as text from the user/web etc?
I can not think of a single case where I'd want to turn data from user/web to symbol (except for parsing command-line options, maybe). Mainly because of the consequences (once created symbols live forever).

Also, many editors provide different coloring for symbols, to highlight them in the code. Take a look at this example.


###类型转换
```
to_s
to_i
to_f
to_a
to_time
to_date
```



###逻辑控制
- **if 表达式**  
```
  if something then  #then 可以不使用
   
  elsif something then
  
  else
  
  end
```

- **if 修饰句**
```
score = 85
info = "Goood" if score >= 90 #only if the right [if] sentence is true, and then the left sentence can be executed. 
puts info
```
- **if you use "if 修饰句" , you can't use keywords: end,elsif,else.**

- **unless , 语句中不能使用elsif语句.**

- **case语句**
```
  case sentence
    when value1 then
          code block
    when value2 then
          code block
    else
          code block
  end  
```

- **loop**   
 while:  
 ```
   while sentence do  
      do something  
   end  
 ```

 until:  
 ```
   until sentence do  #until sentence is true, the loop is over.  
      do something  
   end  
 ```

- **for in 语句**
```
  for 变量名 in 集合(set) do
      do something
  end
```

- **each 语句, each is an iterator for array,hash and range in ruby**
```
  menus = [1,2,3]
  menus.each do | menu |
     do something
  end
```

### array
- **Create an array**
```
a = [  ]
a = Array.new
```
- **add new item**
```
  unshift: 头部添加一个或多个数组元素
  push:    尾部添加一个或多个数组元素
  <<:      尾部添加一个数组元素
```

- **delete existing item**
```
  shift: 头部删除数组元素
  pop:   尾部删除数组元素
  these two functions return the deleted item and the array has deleted the item permanently.
```

- **截取数组**
```
  array[n,m] 从索引n开始，长度为m，组成新数组
  array[n..m] 从索引n开始，到索引m
  array[n...m] n<<index<m , exclude array[m]
```

- **合并数组concat**
```
  colors.concat(students)
```

###String
- ** 使用单引号的字符串会原样输出，使用双引号的字符串可以使用表达式**
``` 
  "string can use sentence like this : #{variable}"
```

- **复制字符串**  
dup() and clone() of String class can copy string, dup() only copys string to a new object, however, clone() will be inherited some special functions,such as singleton.

Subclasses may override these methods to provide different semantics. In Object itself, there are two key differences.

First, clone copies the singleton class, while dup does not.
```
o = Object.new
def o.foo
  42
end

o.dup.foo   # raises NoMethodError
o.clone.foo # returns 42
```

Second, clone preserves the frozen state, while dup does not.
```
class Foo
  attr_accessor :bar
end
o = Foo.new
o.freeze

o.dup.bar = 10   # succeeds
o.clone.bar = 10 # raises RuntimeError
```

About `frozen` : 
You freeze objects, not variables, i.e. you can't update a frozen object but you can assign a new object to the same variable. Consider this:
```
a = [1,2,3]
a.freeze
a << 4
# RuntimeError: can't modify frozen Array

# `b` and `a` references the same frozen object
b = a
b << 4    
# RuntimeError: can't modify frozen Array

# You can replace the object referenced by `a` with an unfrozen one
a = [4, 5, 6]
a << 7
# => [4, 5, 6, 7]
```
As an aside: it is quite useless to freeze Fixnums, since they are immutable objects.

In Ruby, variables are references to objects. You freeze the object, not the variable.

Please note also that
```
a = [1, 2]
a.freeze
a += [3]
```
is not an error because + for arrays creates a new object.


About `Singleton` : 
Note that a class mixing in the Singleton module is functionally equivalent to a class or module with 'class' methods and either a guaranteed initialization call or inline initialization. Compare this usage of Singleton:
```
require 'singleton'
class Bar
  include Singleton
  attr_reader :jam
  def initialize
    @jam = 42
  end
  def double
    @jam *= 2
  end
end

b1 = Bar.instance
b1.double
b2 = Bar.instance
b2.double
p b1.jam          #=> 168
with this no-magic module:

module Foo
  @jam = 42
  def self.double
    @jam *= 2
  end
  def self.jam
    @jam
  end
end

Foo.double
Foo.double
p Foo.jam          #=> 168
```
In both cases you have a single global object that maintains state. (Because every constant you create in the global scope, including classes and modules, is a global object.)

The only functional difference is that with the Singleton you delay the initialization of the object until the first time you ask for it.

So, if you ever have 'class' methods on a class or module and you use those to change the state of that object (e.g. a class keeping track of all subclasses that inherit from it) you are essentially using a singleton.

> http://stackoverflow.com/questions/212407/what-exactly-is-the-singleton-class-in-ruby
> http://stackoverflow.com/questions/13313993/when-is-it-wise-to-use-singleton-classes-in-ruby

- **合并字符串**  
When merge two strings in Ruby, have two situations below:
  one : Return a new string, original string object is immutable, this situation uses `+` operation. 
  two : Assigning original string, this situation uses `<<` appending operation.

- **获取子字符串**  
```
 str = "www.org"
 str[4,3] # 从索引4 开始取3个字符
 str[-3]  # Reverse index 3, from minus 1 starting count  #=>o 
```

- **比较字符串的内容**  
 Explain each of the following operators and how and when they should be used: ==, ===, eql?, equal?.
 ```
   == – Checks if the value of two operands are equal (often overridden to provide a class-specific definition of equality).
   expanding: ==仅仅比较值是否相等
   
   === – Specifically used to test equality within the when clause of a case statement (also often overridden to provide meaningful class-specific semantics in case statements).
   expanding: 当普通对象处于运算符左边时，该运算符与 == 功能相同，但左边对象是一个range对象，且右边对象包含在该range内时，返回true，否则返回false. (1..12)===6 #=>true

   eql? – Checks if the value and type of two operands are the same (as opposed to the == operator which compares values but ignores types). For example, 1 == 1.0 evaluates to true, whereas 1.eql?(1.0) evaluates to false.
   expanding: eql? 不仅比较值，还要比较类型

   equal? – Compares the identity of two objects; i.e., returns true if both operands have the same object id (i.e., if they both refer to the same object). Note that this will return false when comparing two identical copies of the same object.
   expanding: equal? 比较变量的id是否相同，是否是同一变量
 ```
 > http://www.toptal.com/ruby/interview-questions  10 great Ruby interview questions.
 
- **改变字符串内容**
```
 strip      去掉首位空格
 capitalize 首字母大小
 reverse    字符串反转
```

### 时间和日期
```
  today = Time.new
```
- **格式化日期**
```
   today = Time.new
   today.strftime()
```

### 正则表达式  
用于匹配字符串的模式，it's an instance of `Regexp` class.
```
  /[0-9a-zA-Z]/.match(str) #如果返回nil, 则没有返回成功
```

### Class in Ruby  
- **构造函数**  
  `initialize` keyword is construction function in Ruby.

```
  class Student
     def initialize name,age
         @name = name
         @age  = age
     end
  end
  
  std = Student.new('Roger',20)
```

- **内部类**  
We can define an internal class inside another class in Ruby, but, when we're going to create the nested class object, we need to add domain in front of the class name like this: 外部类::内部类.new

Two different way to create nested class:  
```
 One:
 class Phone
    ...
    class Cpu
     ...
    end
 end
 
 Two:
 class Phone
    ...
 end
 class Phone::Cpu
    ...
 end
```

- **可变参数的方法**  
参数名前加`*`
```
 def sum(*nums)
     ...
 end
 
 sum(1,2,3,4)
 sum 1,2,3,4  # these two approaches are allowed.
```
- **指定参数默认值**  
```
  def sum num=10
      ...
  end
```
- **定义类方法**  
```
  class Person
     def Person.speak
         ...
     end
     #or
     def self.speak
         ...
     end
  end
  
  #call:
  Person.speak
```

- **实例变量和类变量**  
```
  @a  #实例变量
  @@a #类变量
```
All instance variables and class variables are private attribute, they aren't accessed outside of the class that they belong to.
Therefore, we can create read/write attributes for these functions.
```
  attr_reader :name,:color,:power  #只读属性
  attr_writer :company             #只写属性
  attr_accessor :date              #可读可写属性
```
- **作用域修饰符**
```
  class Test
   public
    def func1
    
    end
   private
    def func2
    
    end
   protected
     def func3
     
     end
  end    
```

- **继承类**  
Ruby is a single inherit language. the form is :
```
  class Food
  
  end
  
  class Fruit < Food
    #Access base's construct
    def initialize name,weight,sortname
       super name,weight
       @sortname = sortname
    end
  end
```
 ! ruby 不支持像java和c++中的重载，as long as 函数名相同，even though the number of arguments is different, base's func will be overwrited also.
 
 ! Besides 实例方法外, class的类方法同样也继承：  
 ```
   class A
   def self.classFunc
       p "In A::classFunc"
   end
end

class B < A
   def instanceFunc
      p "In B::instanceFunc"
      A::classFunc
      B::classFunc  #类B 继承了类A的类方法 classFunc
   end
end

b = B.new
b.instanceFunc

#output:  
"In B::instanceFunc"
"In A::classFunc"
"In A::classFunc"
 ```
 
 ! With `super` keyword, we can call the same named function in Base class.
 ```
   class Car
      def go 
         puts "go..."
      end 
   end
   
   class BWMCar < Car
         def go
            super
            puts "BWM go..."
         end
   end
   
   bwm = BWMCar.new
   bwm.go  #=> "go..." "BWM go..."
 ```
 Note also that : 在派生类中定义与基类同名方法时，如果改变参数类表，将不能使用super关键字, otherwise, system throws error:
 `wrong number of arguments`
 
###模块 Module  
 - **Define**
 ```
  module ModuleName
     ...
  end
 ```
 In module,it can create instance variables,instance functions,class variables,class functions and attributes as class does.
 
 - **Call module**
 ```
    module MusicModule
       def goto        #instance function
          puts "Jumping..."
        end
        def self.play  #class function
          puts "Playing..."
        end
    end
    class Music
          include MusicModule
          def getMusic
               goto() #调用 module 的实例方法,类中以及类的对象可直接调用，无需作用域
               MusicModule.play() #调用 module 的类方法
          end
    end
    
    #Outside of the class
    m = Music.new
    m.goto
    MusicModule.play
 ```
 
###加载外部文件   
 load,require,include,extend
 ```
    load: 加载源文件
    require：The same functionality as load has, and it can load the source code file that is writed by other programming language.
    include: 将一个模块插入(or contain)到一个类或者其他模块中.
    extend:  要使模块中的方法成为类方法时需要使用extend方法.
 ```
 
###线程  
Ruby中的线程是用户级线程并依赖于操作系统。
- **创建线程**
```
 thread = Thread.new do
     #as block executed by thread
 end
 
 #create a thread with parameters
 today = 'raining'
 hasUmbrella = 'Yes'
 thread = Thread.new today,hasUmbrella do |t,h|
   puts "It's a #{t} day,I have #{h} umbrella."
 end
```

- **挂起线程**  
Why we should use Join in threads?

Joining a thread means that one waits for the other to end, so that you can safely access its result or continue after both have finished their jobs.

Example: if you start a new thread in the main thread and both do some work, you'd join the main thread on the newly created one, causing the main thread to wait for the second thread to finish. Thus you can do some work in parallel until you reach the join.

If you split a job into two parts which are executed by different threads you may get a performance improvement, if

the threads can run independently, i.e. if they don't rely on each other's data, otherwise you'd have to synchronize which costs performance
the JVM is able to execute multiple threads in parallel, i.e. you have a hyperthreading/multicore machine and the JVM utilizes that

```
@threads = []
@threads << Thread.new do
   do something
end
@threads.each { |t| t.join } #join child threads in main thread.
```
- **线程暂停及退出**  
```
  Thread.new do 
     Thread.pass #线程暂停
  end
  
  Thread.new do
     Thread.exit #线程退出
  end
```

- **线程状态及优先级**  
```
   #Thread status
   sleep :  线程处于sleep状态，或正在等待I/O
   run   :  正在运行
   aborting ： 线程被取消
   false    :  线程正常终止
   nil      ： 线程非正常终止
   
   t1 = Thread.new do 
      loop()
      #or
      sleep
      #or
      Thread.stop
      #or
      Thread.exit
      #or
      raise "exception"
   end
   t1.status #查看线程状态
   
   t2 = Thread.new do
      Thread.main.status #查看主线程状态
      Thread.main.alive? #查看主线程是否可执行
   end
   t2.priority = 3 #设置线程优先级
   puts t2.priority #查看线程优先级
```

###BEGIN块和END块  
Ruby 语言使用BEGIN块和END块实现"前置通知"和"后置通知"。
```
BEGIN {
  #Initialize block
  puts 1
}
BEGIN {
  #Initialize block
  puts 2
}
puts 3
END {
  #Release block 
  puts 4
}
END {
  #Release block 
  puts 5
}

#The sequence is 1 2 3 5 4 
```

###异常处理    
Deal with the exceptions when the programming runs. `Exception` class is internal exception class in Ruby.

- **常见系统异常**  
`RuntimeError` ： 由raise方法抛出的默认异常  
`NoMethodError`:  调用不存在方法时抛出的异常  
`NameError`    :  接收器碰到不能解析的变量  
`IOError`      :  当读取关闭的流，写入只读的流或者类似操作时抛出的异常  
`ArgumentError`:  传递参数的数量出错时抛出的异常  

- **捕获异常**  
  rescue语句用于将程序从异常中跳出，前提是程序需要包含在begin end块中.
```
  begin
    do something
    rescue  Exception => e
     do something for the exception
    rescue  Exception => e
     do something for the exception
    ensure    #无论是否出现异常都执行的语句
     do something
  end
```

- **获取异常详细信息**  
 ```
   begin
    do something
    rescue  Exception => e
     #do something for the exception
     puts "#{e.backtrace}"  #异常栈信息
     puts "#{e.to_s}"       #e对象to_s和message方法，用于获取异常的详信息
     puts "#{e.message}"
   end     
 ```
 
- **手动抛出异常**  
 ```
   def hello
     raise ArgumentError
   end
   
   begin
     hello
     rescue ArgumentError
       puts '捕获到hello抛出的异常'
    end   
 ```
 
- **自定义异常类**  
 ```
   #encoding:utf-8   #字符编码转utf-8 可以识别中文字符
   class MyException < Exception
      def getMsg
      
      end
   end
   
   begin
     message = "Has an exception"
     raise MyException.new(message)
     rescue MyException => e
       puts '捕获抛出的自定义异常'
       puts "#{e.getMsg}"
    end   
 ```
 
###动态语言特性和元编程    
For dynamic language feature, first of all, we must classify the meaning of the concepts: `object(对象)`,`class（类）`,`module(模块)`， and what are the relationships among these three.

Secondly, the internal keywords in Ruby, `Object` class and `BasicObject` and `Kernel` module, which are so important point, we must be familiar with them.

Thirdly, the object model in Ruby includes `class` and `superclass`, these two main concepts.


- **the object model :**   
```  
                           BasicObject        level 1 Root class in Ruby
                               | 
                             Object           level 2 次根类
                               |
                             Module           level 3 模块, Module 的类都是Class, superclass是Object
                              /  \                           
                           Class  M1          level 4 CLass 的类都是Class, superclass是Module.自定义模块M1的 
                          /  |  \                     类是Module,自定义模块M1没有superclass. 
                         C1  C2  C3           level 5 自定义类(C1,C2,C3)的class都是Class, superclass是Object, 
                         |   |    |                   
                        obj1 obj2 obj3        level 6 对象(obj1,obj2,obj3)的class 分别为 C1 , C2 ,C3 , 对象没有superclass
```             

- **The method lookup(方法查找) in the object model**  
Two important concepts: ancestor chain(祖先链)  and receiver(接收者)  

Receiver is the one who calls the function, meanwhile, we can replace the object name with `self` which represents the current object. Furthermore, if in the class and outside of an instance function, `self` means the class itself.
```
   class Cself
      puts self
      def func
        puts self
      end
   end
   #output: #=>Cself  #the first self
   #Also, we can find that the self in the function func isn't executed yet. After creating an object, and the object calls the function, the second self will be executed, and the output is object itself. #=> #<Cself:0x000000016e5a90>
```

Ancestor chain(祖先链)：  
We have known that an object is a `self` when it calls a function, what is more important is how the object find the called function.

There is an institution named ancestor chain in Ruby.Ruby maintains an inherit relationship for each class. All public and protected functions in the inherit ralationship will be collected into the ancestor chain, while private functions will be excluded.
For example:  
```
   class Ca
     def ca_func
        puts "ca_func"
     end
     private
     def ca_pri_func
         puts "ca_pri_func"
     end
   end
   
   class Cb < Ca
     #Overwrite
     def cafunc 
        puts "ca_func in Cb"
     end
     def cb_func
        puts "cb_func"
     end
   end
  
   #ancestor chain as follows:
    BasicObject
        |
    Kernel(a module)  #内核模块, 包含一些系统方法 Kernel.private_instance_methods.grep(/^pr/)
        |              Kernel.private_instance_methods 查看Kernel的方法
     Object
        |
       Ca      # has ca_func collected into the ancestor chain
        |
       Cb      # has ca_func and cb_func collected into the ancestor chain
```
An example for private function:  
```
class Cprivate
   def funcA
      puts "funcA"
      self.funcB   # can't use self to chain to the funcB. using self, it will be mapped to ancestor chain automatically. 
      #funcB       #It's ok. 这个称为似有规则, 只能在自身中调用一个私有方法
   end

   #protected
   private
   def funcB
      puts "funcB"
   end
end

b = Cprivate.new
b.funcA  #=> throw xception: `funcA': private method `funcB' called for #<Cprivate:0x00000001f88198> (NoMethodError)
```

An inherit relationship with modules which have same named function:  
``` 
  module Printable
     def print
         puts "Printable's print"
     end
     
     def prepare_cover
     
     end
  end
  
  module Document
     def print_to_screen
         prepare_cover
         format_for_screen
         print
     end
     
     def format_for_screen
     
     end
     
     def print
         puts "Document's print"
     end
  end
  
  class Book
        include Document
        include Printable
  end
  
  #Creating instance of class Book
  b = Book.new
  b.print_to_screen 
```
Note that loading module in class is like a behavior of pushing in stack. So, the ancestor chain of this example is like this:
`Book -> Printable -> Document -> Object -> Kernel -> BasicObject`, 当self所属的类开始方法查找，until find the function named `print()`,In ancestor chain, the closest `print` function is `Printable#print()`, therefore, `Printable#print()` is called.

- **Methods**  
The keys are `Dynamic methods call` and `Dynamic methods define`, and the special one is `method_missing`.  

 (1). Dynamic methods call  
 使用动态派发技术,使用Object#send()取代`点标记符`来调用MyClass#my_method()
 
 send方法的强大之处在于，即使是私有方法，也可以在类外动态调用： 
 
 ```
   class MyClass
         private
         def my_method my_arg
             my_arg * 2
         end
      end

      #使用动态派发技术,使用Object#send()取代`点标记符`来调用MyClass#my_method()
      obj = MyClass.new
      obj.my_method(3)
 ```
 
 (2). Dynamic methods define  
 可以利用`Module#define_method` 方法定义一个方法，只需要为其提供一个方法名和一个充当方法主体的块即可。
 `define_method` 是否也是扁平化作用域的一种方式?  
 ```
    class MyClass1
        puts "Enter MyClass1"
        define_method :my_method do | my_arg |
           puts "when the define_method executed"
           my_arg * 3
        end
        p self.instance_methods.grep /^my_method/  #We can find that define_method will be executed and my_method will be
        puts "Enter 2"                              created in the period that defining the class.  
     end
     obj = MyClass1.new
     p obj.my_method(2)  #=>6
 ```
 
 在类中可以将参数传递给用`define_method`创建的实例方法 :   
 ```
   class MyClass2
        puts "Enter MyClass2"
      def self.define_component(name)
        puts "When the define_component executed"
        define_method name do | my_arg |
           puts "When the define_method executed"
           my_arg * 3
        end
      end
        define_component :my_method  #类方法调用
        puts "Finish MyClass2"
     end
     obj = MyClass2.new
     p obj.my_method(2)  #=>6
 ```

The best way to shrink the scale of code :  
```
  class DS
    def get_cpu_info(id)
        "2.16Ghz"
    end
    
    def get_cpu_price(id)
        150
    end
  end
  
  class Computer
    def initialize computer_id, data_source
        @id = id
        @data_source = data_source
        data_source.methods.grep /^get_cpu_info$/ { Computer.define_component("cpu") }
    end
    
    def self.define_component(name)
        define_method(name) do
          info   = @data_source.send "get_#{name}_info", @id
          price  = @data_source.send "get_#{name}_price", @id
          result = "#{name.capitalize} : #{info} ($#{price})"
          return "*#{result}" if price >= 100
          result
        end
    end
  end
  ds = DS.new
  cp = Computer.new(1,ds)
  p cp.cpu  #此处会报错：找不到该方法, apparently, ruby元编程书中存在错误，因为在构造类的时候，根本没有动态绑定的定义该实例方法
```

Solutions:  
  one: 在类定义中调用类方法`define_component` 完成`define_method`动态绑定方法, 缺点是代码扩充, 模块间耦合性增加。 DS模块有方法，Computer也得相应增加。
  ```
  def self.define_component(name)
        define_method(name) do
          info   = @data_source.send "get_#{name}_info", @id
          price  = @data_source.send "get_#{name}_price", @id
          result = "#{name.capitalize} : #{info} ($#{price})"
          return "*#{result}" if price >= 100
          result
        end
    end
    define_component :cpu  #调用类方法`define_component`
  ```
 two: 用class_eval动态绑定打开类  
 ```
    def initialize id, data_source
        @id = id
        @data_source = data_source
        data_source.methods.grep(/^get_cpu_info$/) { 
           Computer.class_eval do
           Computer.define_component("cpu")
           end
         }
    end
 ```
 
  `define_method` 是否也是扁平化作用域的一种方式?  答案是明显的, 通过一个example来观察:  
  ```
  class Flatten
    local_v1 = "Within_Flat"  #类局部变量

    #define instance methods with def and Module#define
    def m1
       puts local_v1
    end

    define_method :m2 do
       puts local_v1
    end
   
end

f = Flatten.new
f.m1 #=>Throw exception:  undefined local variable or method `local_v1' for #<Flatten:0x00000000e93770> (NameError)                                      无法访问类局部变量!
f.m2 #=>Within_Flat #define_method 定义的方法可以访问类的局部变量!
  ```
  
  Extend: 区分类实例变量/类变量/类局部变量/对象的实例变量/对象的局部变量  
  ```
    class Flatten
    @local_v1 = "Within_Flat"        #类实例变量，belongs to class itself,               
                                      只能在类内部使用，不能被类的实例和子类所访问，在类外会有语法错误
    @@class_v1 = "A class variable"  #类变量，belongs to class itself, 只能在类内部使用，在类外会有语法错误，所有类的对象都
                                      可以使用它，且只有一份拷贝,类似于C++中的static 静态成员变量
    #define instance methods with def and Module#define
    def m1
       local_obj = 1   #对象的局部变量, 方法调用结束则消亡
       @local_v1 = 2   #对象的实例变量，伴随对象的消亡则消亡
       puts @@class_v1  #=> A class variable  访问成功!
    end

    define_method :m2 do
       @local_v2 = 3  #对象的实例变量，伴随对象的消亡则消亡
    end
   
    puts @local_v1  #=> Within_Flat 
    
end

#puts Flatten.@local_v1 #=>exception: flattening.rb:17: syntax error, unexpected tIVAR
#puts Flatten.@@class_v1 #=>exception: flattening.rb:17: syntax error, unexpected tIVAR
f = Flatten.new
f.m1
f.m2
  ```
 
 (3). Method_missing  
  `method_missing()` is an instance method of `Kernel` module. We can realize `Dynamical delegation`(动态代理) with `method_missing`.  
  The principle is that Ruby will take undefined methods into `method_missing` this recalled method(将所有未定义的方法通过method_missing进行回调).  
  Due to it is `Kernel#method_missing`, in general, we will overwrite this method in the class created by us.   
  In super(`Kernel#method_missing`), if a method isn't defined, and system will throw an exception :  ` undefined method [roger] for #<Roulette:0x00000000a0a078> (NoMethodError)` , but , if we overwrite the method_missing in ourselves' class, we can do whatever we want in its body, here is an example:  
  ```
     #Recall super Kernel#method_missing in the overwrited method_missing by you ,when meets a method that we don't know how to deal with it

class Roulette
      def method_missing(name,&args)
          person = name.to_s.capitalize
          super unless %w(Bob Frank Bill).include? person #如果方法不是Bob Frank Bill, 则调用super，即Kernel#method_missing
          number = 0   #此处不定义该变量,会发生什么?

          3.times do 
             number = rand(10) + 1
             puts "#{number}..."
          end
          p "#{person} got a #{number}"
      end
    
end

ro = Roulette.new
ro.bill
ro.roger  #=> `method_missing': undefined method `roger' for #<Roulette:0x00000000a0a078> (NoMethodError)
  ```
  One more place we must pay attention is if we don't define variable `number = 0` outside of the loop `3.times do ... end`. What will be happened? Apparently, `p "#{person} got a #{number}"` this sentence is out of the scope of variable `number = rand(10) + 1` , Ruby interpreter(ruby解释器) don't know what `number` is, in default, interpreter think `number` as a method which omits  '()' behind its name. In genaral, you will see an obvious 'NoMethodError' error, It's easy to observe the error.
But, while you overwrite `method_missing` of Kernel(当父类的方法被覆盖，可以在子类的覆盖方法中调用super 执行该父类方法的调用),
and `number()` finally goes into `method_missing` overwrited by us,  therefore, the method call will always be executed, until 函数调用堆栈 overflows.  
  
  Extending:  
  %w(foo bar) is a shortcut for ["foo", "bar"]. Meaning it's a notation to write an array of strings separated by spaces instead of commas and without quotes(引用) around them.    
  %w quotes like single quotes '' (no variable interpolation(添加内容), fewer escape sequences), while %W quotes like double quotes "".   
```
irb(main):001:0> foo="hello"
=> "hello"
irb(main):002:0> %W(foo bar baz #{foo})
=> ["foo", "bar", "baz", "hello"]
irb(main):003:0> %w(foo bar baz #{foo})
=> ["foo", "bar", "baz", "\#{foo}"]
```  
  
  About super keyword and super method:  
  ```
class A
  def aFunc a, b
      puts "#{a}#{b}"
  end
end

class B < A

  def aFunc c
      puts "Enter B::aFunc"
      b = 1
      #super b,c #This kind of call is ok!   it's a super method
      super #if we call super aFunc without parameters , what will be happened ? The answer is throwing wrong number of arguments exception!  It's a super keyword
  end
end

b = B.new
b.aFunc 5

  ```
  
The super keyword can be used to call a method of the same name in the superclass of the class making the call.

It passes all the arguments to parent class method.

**super is not same as super()**
```
class Foo
  def show
   puts "Foo#show"
  end
end

class Bar < Foo
  def show(text)
   super

   puts text
  end
end
Bar.new.show("Hello Ruby")
ArgumentError: wrong number of arguments (1 for 0)
```
super(without parentheses) within subclass will call parent method with `exactly same arguments` that were passed to original method (so super inside Bar#show becomes super("Hello Ruby") and causing error because parent method does not takes any argument)   
   ! 如果使用`super` keyword, super class's method and subclass's  overrided(**overwrited重写/覆盖,overload重载，在ruby中只有重写，没有重载**) method must be same argument list.   
  
  **当方法冲突时 when methods clash**    
  There is a case that the ghost method has the same name with a real defined method which is defined in same scope(同一作用域下) or inherited up by super class(和真实定义的方法同名的情况). In default,  the real method will be called, rather than the ghost on priority.  这个问题是动态代理技术的通病, when methods clash, how to solve it?  If we don't need the inherited real method, we can delete the real method to solve the issue. If you are in a delegation class, for safety, we should delete all inherited method from super class, which calls `Blank Slate` 白板类.   
  
  Two ways to delete methods for a class, one is `Module#undef_method(method_name)` which can delete all of methods it has including inherited(可以删除所有的，包括被继承下来的), the other is `Module#remove_method(method_name)` which just delete methods belonging to receiver itself and reserve inherited(只删除接收者自己的，保留被继承下来的)   
   ```
     class Computer
         instance_methods.each do | m |
              undef_method m unless m.to_s =~ /^__|method_missing|respond_to?/  #双划线打头的方法还是保留吧
         end
     end
   ```
  Extending:  
  unless 后的表达式为假，则执行块内代码。 可接else , 但不能接elsif.  
  if 是表达式为真，则执行块内代码. 可接else，elsif.  
  
  正则表达式: 
  /a Regular expression/.match(a string)  
  a string =~ /a Regular expression/  =~ 匹配符 or RegExp.match()    
  The difference of two ways：   
  ```
str = "066"
p  /[0-9]+/.match(str) #=> #<MatchData "066">  match successfully, return a obj
p str =~ /[0-9]+/ #=> 0 match successfully, return 0  

str1 = "abc"
p  /[0-9]+/.match(str1) #=> nil
p  str1 =~ /[0-9]+/     #=> nil
  ```
  
  覆写(override/overwrite)respond_to? 方法:  
  当你查看一个对象是否响应一个ghost方法时，他会返回false.    
  ```
     cmp = Computer.new(0, DS.new)
     cmp.respond_to?(:mouse)  #=> false
  ```
  
  覆写`method_missing`的同时，覆写`respond_to?` 可以解决这个问题:   
  ```
     class Computer
         def respond_to?(method)
             @data_source.respond_to?("get_#{method}_info") || super  #通过返回DS的real方法确保返回true
         end
     end
  ```
  在`respond_to?`中调用super,是为了assure 在查询其他real方法时，会调用默认的`respond_to?`方法.   
  
  Top-level methods are defined as private, and `Object#respond_to?` ignores private methods by default (you need to pass second argument for it to recognize say_hello):  
```
def say_hello
  puts 'hi'
end

puts respond_to?(:say_hello)                    #=> false
puts respond_to?(:say_hello, :include_private)  #=> true
say_hello
```
  > http://stackoverflow.com/questions/17893977/confused-about-respond-to-method  
  
  What is the purpose of “!” and “?” at the end of method names?   方法名后边带上！（重磅方法）或者？（布尔类型返回值）。    
  It's "just sugarcoating" for readability, but they do have common meanings:
  Methods ending in ! perform some permanent or potentially dangerous change; for example:   

  `Enumerable#sort` returns a sorted version of the object while `Enumerable#sort!` sorts it in place.  

  In Rails, `ActiveRecord::Base#save` returns false if saving failed, while `ActiveRecord::Base#save!` raises an exception.  

  `Kernel::exit` causes a script to exit, while `Kernel::exit!` does so immediately, bypassing any exit handlers.  

  **Methods ending in ? return a boolean**, which makes the code flow even more intuitively like a sentence — if number.zero? reads like "if the number is zero", but if number.zero just looks weird.   
  > http://stackoverflow.com/questions/7179016/what-is-the-purpose-of-and-at-the-end-of-method-names  
  > http://www.jb51.net/article/57112.htm  
  
  _-----------------Methodmissing is over---------------_    
   
### 代码块  
The main content is about these three features which are the important knowledges for functional programming.    
  1. code block(代码块) :   
    ```
        do |x,y,z| #代码块参数
           do something with parameters x,y,z 代码块体
           #the last line is a return sentence.
        end
    ```
  2.  `proc` , `lambda`  and the class `Proc`   
    
  3.  `Scope` , `扁平化作用域` and `instance_eval` method.   
  

- **代码块 block**  
First of all, Only when calling a method, block can be defined behind the method calling, and the block will be callbacked in the body of the method defining.(只有在调用一个方法时，才可以定义一个块，块会直接传递给这个方法，然后在该方法中可以用yield关键字  回调这个块)   
```
   def a_method(a,b)
       yield(a,b)
   end
   
   a method(1,2) { |x,y| (x+y) * 3  } #=>9
```
  可以像调用方法那样为块传递参数，块中最后一行会被作为返回值.  
  
  When we are in the body of a method, we can use `kernel#block_given?()` to ask the method whether includes a block or not :  
  ```
    def a_method(a,b)
       return "#{yield(a,b)}" if block_given?
       "No block given"
    end
  ```
  If we use `yield` in `if block_given?` body, while block_given? returns false, it will catch a runtime error.   
  
- **闭包 closures**  
  
   块的要点在于它是完整的，可以立即运行，它们既包含代码，也包含一组绑定(局部变量,实例变量,self)。  
The question is where a block gains the bindings(绑定) belonging to the block.  Obviously, the block belongs to a certian scope, it will gain bindings from the scope, here is an example:  
```
   def my_method
     x = "Good"
     yield("Roger")   
   end 
   x = "Hello"
   my_method { |y| "#{x}, #{y} world"} #=> "Hello, Roger world"
```
We can find out that the block get the binding which is a local variable `x`, not the `x` in the method `my_method` which is another scope.   

- **作用域门 Scope gate**  
  Exactly speaking, 程序会在三个地方关闭前一个作用域，同时打开新的作用域：  
     1. 类定义   `class MyClass ... end`
     2. 模块定义 `module MyModule ... end`
     3. 方法     `def MyMethod ... end`  
   这3个边界分别用class,module,def关键字作为标志，每一个作用域都充当一个作用域门.  
   
   一个切换作用域的例子, 用`Kernel#local_variables()`方法来跟踪绑定的名字:  
   ```
       v1 = 1  
       
       class MyClass
            v2 = 2
            p local_variables
            
            def my_method 
                v3 = 3
                p local_variables
            end 
            
            p local_variables
            
       end 
       obj = MyClass.new
       obj.my_method     #=> [:v3]
       obj.my_method     #=> [:v3]
       local_variables   #=> [:v1,:obj]
   ```
  我们可以发现，当进入class之后， `v1` 便超出class `MyClass`的作用域，此时发生过作用域切换，超出当前scope的变量都被屏蔽。  
  Ruby 没有像C++那样有 `内部作用域` 和 `外部作用域` 这样嵌套作用域的concept.      **它的作用域是明显分开的,只要发生作用域切换,之前作用域的bindings将被屏蔽**     
  
  **全局变量和顶层实例变量**  
  全局变量`$var` ，是以`$`开头的变量，它可以在任何作用域中使用，会使模块间高度耦合. 因此，可以使用顶层实例变量代替全局变量。  
  ```
     @var = "123" #它是顶层对象main的实例变量
     
     def my_method
        p @var
     end
     
     my_method #当main对象在扮演self，就可以访问顶层实例变量；当其他对象成为self，顶层实例变量就退出作用域了. 
  ```
  
- **扁平化作用域**  
  What if we hope a variable can get through scope having effect in another scope, what should we do ?  
  怎么让绑定穿越一个作用域门，换言之， 怎么去开门？  
  
  (1). 用`Class.new()`,`Module.new`,`Module#define_method` 替换会产生作用域门的关键字`class MyClass...` , `module MyModule...`,`def MyMethod...`   
  ```
     my_var = "Success"
     MyClass = Class.new do 
          puts "#{my_var} in class"
          define_method :my_method do 
             puts "#{my_var} in method"
          end
     end
     MyClass.new.my_method
  ```
  **这就是扁平化作用域! 它让一个作用域看到另一个作用域的变量**  
  
  (2). 如果你想在一组方法中间共享一个变量，但是又不希望其他方法也能访问这个变量，就可以把这些方法定义在那个变量所在的扁平化作用域中:  
```
     def define_methods
         shared = 0 
         Kernel.send :define_method, :counter do  #如果去掉Kernel,会抛异常: undefined method `define_method' for main:Object   (NoMethodError)
                                                  #define_method须是被class ，Object ,Kernel 等调用，因为它是类方法
           shared 
         end
         Kernel.send :define_method, :inc do |x|
           shared += x
         end
     end
     
     define_methods 
     counter #=> 0
     inc(4)
     counter #=> 4
     
     class M

    def self.our
     p "In class method..."
    end
    def me
        shared = 0
        M.send :define_method , :count do
           shared
        end
        M.send :define_method, :inc do |x|
         shared += x
       end
       M::our  #=> It's ok!
    end
end

m = M.new
m.me
p m.count
m.inc(4)
p m.count
m.our #=>Throw exception: undefined method `our' for #<M:0x89ea748> (NoMethodError)
      #类方法，对象无法调用，不在对象的祖先链中
```
  
  (3). **Object#instance_eval()**  
  它在一个对象的上下文中执行一个块，也是一种扁平化作用域的方式。 可以把传递给`instance_eval()`方法的块称为一个上下文探针.  
 
```
my_var = 3

class Ca
    def initialize a,b
        @x,@y = a,b
    end
end

#we can see that it can be beyond scope to deal with instance variables of itself including private variables in the block #behind instance_eval and loca variable my_var
Ca.new(1,2).instance_eval {  p  @x+@y+my_var } #=> 6    
#instance_exec is same as instance_eval with parameters. 带参数版本的instance_eval
Ca.new(1,2).instance_exec(my_var) { |arg| p  @x+@y+arg } #=> 6
 ```
  
- **可调用对象**  
  `proc`,`lambda` and method.   
  
  从底层来看，使用块需要分两部。第一步，将代码打包备用；第二步，通过yield来执行代码. In Ruby, We have three approaches to pack code, each of those is `proc`,`lambda` and method.   

  (1). **proc**  
   Ruby provides a class named `Proc` which is a block transformed to object(它其实就是被转换成对象的块).  In ruby, most of things are object, but block not.  Imagining that you want to preserve a block for callable from time to time, meanwhile, you need a object to do so.  
   ```
      inc = Proc.new {|x| x+1}
      # more...
      inc.call(2) #=>3 
   ```
   We can use `Proc#call` to execute the object which is transformed from block.  
   
   In Ruby, it provides two kernel methods engaged in transforming block to callable object: 
   `Proc#lambda()` and `proc()` . As the matter of fact, those three approaches(方式) have  a little bit of slight difference.      
   
   (2). **&操作符**  
   块就像方法的extra的匿名参数，In the case that we want to push the block to another method. How should we do ?  
   要将块附加到一个绑定上，可以给这个方法添加一个特殊的参数，这个参数必须是参数列表中的最后一个，且以&符号开头：  
   ```
     def math(a,b)
         yield(a,b)
     end
     
     def teach_math(a,b,&operation)
         puts "Let's do the math:" 
         puts math(a,b,&operation)
     end
     
     teach_math(2,3) {|x,y| x*y}
     
     #output:
     Let's do the math:
     6
   ```
  在调用teach_math()时，如果没有附加一个块，则&operation参数将被赋为nil，这样在math()方法中的yield操作会失败. 

  **&操作符的真正含义是，这是一个Proc对象，我要把它当一个块使用。 去掉&操作符，就再次得到一个Proc对象. &oper : block, oper: Proc object**  
  
  ```
     def math(a,b)
         yield(a,b)
     end
     
     def teach_math(a,b,&operation)
         puts "Let's do the math:" 
         p operation.call(a,b)     #=> 6
         p yield(a,b)              #=>6 
         puts math(a,b,&operation) #=>6
     end
     
     teach_math(2,3) {|x,y| x*y}
     
     def method(&the_proc)
          the_proc
     end    
     
     p = method { |name| "Hello", #{name}!"}
     puts p.class
     puts p.call("Bill")
     
     #output:
     Proc
     Hello,Bill!
  ```
  
  **The contrast(对比) between proc and lamdba**   
  There are two important different between proc and lambda. One is ralated to `return` keyword, the other is the checking of argument list.  
  
  (1). **lambda 中的return语句仅从这个lambda块中放回, while proc是从定义它的作用域中返回(也可以说是作为该作用域的返回). **  , here gives two example:  
  lambda:    
  ```
     def double (callable_obj)
         callable_obj.call * 2  #此处调用lambda, 即lambda在此处返回
     end
     
     l = lambda { return 10 }
     double(l) #=> 20 
  ```
  proc:  
  ```
    def another_double
        p = Proc.new { return 10 }
        result = p.call #处在作用域 another_double中，当前的self为 main，proc中的return语句作为该作用域的返回
        return result * 2 #unreachable code!
    end
    
    another_double #=> 10
    
    #一个LocalJumpError错误：  
    def double(callable_obj)
        callable_obj.call * 2  #在作用域double中无法获得其他作用域的返回
    end
    
    p =Proc.new { return 10 }
    double(p) # failed to execute, 并会产生一个LocalJumpError错误
    
    #We can avoid the problem with explicit return.
    p = Proc.new {10}
    double(p) #=>20
  ```
   
   (2). 参数个数的问题  
   lambda 适应能力比proc差，如果`块参数个数`与`调用中传入的参数个数`不同，则会失败并抛出`ArgumentError`错误. 而proc会自我调整:   
   ```
       p = Proc.new { |x,y,z| [x,y,z] }
       p.call(1,2,3,4) #=> [1,2,3]
       p.call(1,2)     #=> [1,2,nil]
   ```
   **Totally, lambda更直观，因为它更像一个方法，它不仅对参数个数有严格要求，而且在调用return时只从它的代码块中返回.**  
   
   (3). Kernel#proc()函数  
   Ruby1.9 ： Kernel#proc() == Proc.new()  
   Ruby1.8 :  Kernel#proc() == Kernel#lambda()  
   
   In Ruby1.9 :  
   ```
      p = proc { return 10 }
      p p.call  #这样会失败，因为定义proc的self为main对象，而执行调用的对象为p对象，在p对象所处的作用域中无法获得main对象中的返回
   ```
   
   _-------Block is over---------_
   
   
###类定义  

定义ruby中的类实际也是在运行一段代码。 在类定义中有3个强大的法术: 类宏(Class Macro),环绕别名(Around alias),单件类(eigen class). 
- **当前类**  
  (1). 在类定义中， 当前对象self就是正在定义的类.  
  (2). 如果有一个类的引用，则可以用class_eval()方法打开这个类，使之扁平化作用域，共享当前上下文的绑定.    

- **单件方法 Singleton Methods**  
   **仅属于某个对象的方法，称为单件方法,类方法也属于一种单件方法**  
   ```
      def obj.a_singleton_method; end
      def MyClass.another_class_method; end
   ```

- **类宏(Class Macro)**  
  可以通过使用`Module#attr_*()` 方法家族中的一员来定义访问器  
  ```
    class MyClass
      attr_accessor :my_attribute  #实际内部用到了define_method
    end  
  ```

- **Eigenclass 特殊类**    
  对象和类都有一个隐藏的类，也叫特殊类 ： 
  获得这个隐藏类的引用：  
```
  obj = Object.new
  eigenclass  =  class << obj
     self
  end
  eigenclass.class #=> Class
     
  class MyClass
    class << self   #打开类的隐藏类
       def a_class_method 
         p "In class method..."
       end
    end
    def m
       p "In m..."
    end
   end

o = MyClass.new

class << o   #打开对象o的隐藏类
   def n
     p "In n..."
   end
end
o.n

module M
    class << self  #打开模块的隐藏类
         def a_module_class_method
           p "Module class method..."
         end
    end
end

M.a_module_class_method

class << Kernel
   def a_kernel_singleton_method
      p "a_kernel_singleton_method..."
   end
end

Kernel.a_kernel_singleton_method

class << Module
def a_module_singleton_method
      p "a_module_singleton_method..."
   end
end

Module.a_module_singleton_method

```
  
  **带有隐藏类的对象模型**   
  
```
      BasicObject ---> #BasicObject特殊类
         \         
           Kernel ---> #Kernel特殊类, Kernel 在BasicObject 与 Object 之间非继承关系，而是类似一种代理接口的关系。
         /             #Kernel也有自己的eigenclass,但是它并不是继承祖链上的一员!
      Object      ---> #Object特殊类
        |                |
      Module      ---> #Module特殊类      
        |                |
       Class      ---> #Class特殊类
        |                |
        C         ---> #自定义类C的特殊类
  [a_method()]    [a_class_method()]
        |                |         
        D         ---> #自定义类D的特殊类       
[继承C::a_method()]   [继承C::a_class_method() --—> D::a_class_method()] 
        |    
obj -> #obj
    [a_singleton_method]
    
```
  Needs to note :   
  (1). 实例对象无法调用类方法.  
  (2). 私有的实例方法和类方法将不被继承，即不会进入祖先链中.  
  
```
  #private method

class Mc
  class<<self
     #private
     def a_private_class_method
         p "a_private_class_method..."
     end
  end
end

class Mb < Mc

end

p Mb.methods.grep /^a_private/ #如果Mc中加入private,则此处输出空
b =Mb.new
#b.a_private_class_method  #=>  undefined method `a_private_class_method' for #<Mb:0x000000015eff28> (NoMethodError)
                           #对象无法调用类方法  
```
  
  (3). `Class`,`Module`中定义的方法都是类方法  
```
class << Object
def a_Object_singleton_method
      p "a_Object_singleton_method..."
   end
end

Module.instance_eval do
    define_method :a_Module_instance_method do
         p  "a_Module_instance_method"
    end
end

Class.a_Module_instance_method  #=> "a_Module_instance_method"
Mb.a_Module_instance_method     #=> "a_Module_instance_method"
Mb.a_Object_singleton_method    #=> "a_Object_singleton_method..."
```
  
- **类方法和include()**  

  对象扩展：  
  ```
     module MyModule
         def my_method; "hello"; end
     end
     
     obj=Object.new
     class<< obj
         include MyModule
     end
     
     obj.my_method         #=> "hello"
     obj.singleton_method  #=> [:my_method]
  ```
  类扩展和对象扩展的应用非常普遍，因此，Ruby为它们专门提供了一个叫做`Object#extend()`的方法：  
  ```
     module MyMdoule
        def my_method; "Hello"; end
     end
     
     obj = Object.new
     obj.extend MyModule
     obj.my_method     #=> "hello"
     
     class MyClass
           extend MyModule
     end
     MyClass.my_method  #=> "hello"
  ```
  
- **Alias**  
`alias` 关键字 ,可以给Ruby方法取一个别名：  
```
class MyClass
    def my_method; 'my_method()'; end
    alias :m :my_method  #中间没有逗号， 因为是关键字
end

obj = MyClass.new
obj.my_method  
obj.m
```  
Ruby 还提供了`Module#alias_method()`方法,它的功能与alias关键字相同.  
```
   class MyClass
      alias_method :m2,:m
   end
   
   obj.m2   #=>  "my_method()"
```

- **Around Aliases 环绕别名**  
(1).  给方法定义一个别名  
(2).  重新定义这个方法  
(3).  在新的方法中调用老的方法  

```
  class String
     alias :real_length :length  
     
     def length
        real_length > 5 ? 'Long' : 'Short'
     end
  end 
  
  "War and Peace".length       #=> "Long"
  "War and Peace".real_length  #=> 13
```

###编写代码的代码  
`Kernel#eval`  , 它不是使用块，而是直接执行包含Ruby代码的字符串.  
```
  array = [10,20]
  element = 30
  eval("array << element") #=> [10,20,30]
```
由于使用字符串代码函数`eval()` 可能导致注入式攻击，最好的方式是不是用`eval`, 改为`send()`.  




  



   
   
  


  

  
 
 

