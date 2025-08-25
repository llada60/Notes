[TOC]

# Build_in Decorator

### 1. @property

使用@property装饰器来创建只读属性，@property装饰器会将方法转换为相同名称的只读属性,可以与所定义的属性配合使用，这样可以防止属性被修改。
其使用场景有以下两种：

##### 场景一：用于修饰方法，使得方法可以像属性一样访问。

```python
class DataSet(object):
  @property
  def method_with_property(self): ##含有@property
      return 15
  def method_without_property(self): ##不含@property
      return 15

l = DataSet()
print(l.method_with_property) # 加了@property后，可以用调用属性的形式来调用方法,后面不需要加（）。
print(l.method_without_property())  #没有加@property , 必须使用正常的调用方法的形式，即在后面加()
```

注意：如果使用property进行修饰后，又在调用的时候在方法后面添加了()， 那么就会显示错误信息。因为它已经变成了一个属性而不是方法了。

##### 场景二：与所定义的属性配合使用，这样可以防止属性被修改

由于python进行属性的定义时，没办法设置私有属性，因此要通过@property的方法来进行设置。这样可以隐藏属性名，让用户进行使用的时候无法随意修改。

比如给获取属性的方法设置@propert,这个方法相当于一个属性，这个属性可以让用户进行使用，而且用户有没办法随意修改。

```python
class DataSet(object):
    def __init__(self):
        self._images = 1
        self._labels = 2 #定义属性的名称
    @property
    def images(self): #方法加入@property后，这个方法相当于一个属性，这个属性可以让用户进行使用，而且用户有没办法随意修改。
        return self._images 
    @property
    def labels(self):
        return self._labels
l = DataSet()
#用户进行属性调用的时候，直接调用images即可，而不用知道属性名_images，因此用户无法更改属性，从而保护了类的属性。
print(l.images) # 加了@property后，可以用调用属性的形式来调用方法,后面不需要加（）。
```

### 2. @classmethod 的使用

#### 类方法

原则上，类方法是将类本身作为对象进行操作的方法。假设有个方法，且这个方法在逻辑上采用类本身作为对象来调用更合理，那么这个方法就可以定义为类方法。另外，如果需要继承，也可以定义为类方法。

**定义**
使用装饰器@classmethod。第一个参数必须是<u>当前类对象</u>，该参数名一般约定为“cls”，通过它来传递类的属性和方法（不能传实例的属性和方法）；

**调用**
类和实例对象都可以调用。

假设如下场景： 我有一个学生类和一个班级类，想要实现的功能为：
执行班级人数增加的操作、获得班级的总人数；
学生类继承自班级类，每实例化一个学生，班级人数都能增加；
最后，我想定义一些学生，获得班级中的总人数。
这个问题用类方法做比较合适。

```python
class ClassTest(object):
    __num = 0

    @classmethod
    def addNum(cls):
        cls.__num += 1 # cls only; self no

    @classmethod
    def getNum(cls):
        return cls.__num

    # __new__ is called *before* __init__, during object creation
    # 这里我用到魔术方法__new__，主要是为了在创建实例的时候调用累加方法。
    def __new__(self):
        ClassTest.addNum()
        return super(ClassTest, self).__new__(self)


class Student(ClassTest):
    def __init__(self):
        self.name = ''

a = Student()
b = Student()
print(ClassTest.getNum())
```

```python
# default __new__
class tmp():
  def __new__(cls, *args, **kwargs):
    # when enter the object does not exist yet
    # python passes the class being instantiated as the first argument
    return super().__new__(cls)
```

- common of `__new__` and `@classmethod`: both receive the `cls` as the first argument

  - `@classmethod` is explicitly designed that way.

  - `__new__` is invoked before the instance exists.

- difference:

  - `@classmethod` is a normal method that can call on a class or instance:

    ```python
    class Foo:
        @classmethod
        def hello(cls):
            print("Hi, I’m class:", cls)
    
    Foo.hello()   # works
    Foo().hello() # also works
    ```

  - `__new__`  is a special hook that Python automatically calls when creating a new instance

    ```python
    obj = Foo()   # automatically triggers Foo.__new__(cls)
    ```

### @staticmethod 的使用

#### 静态方法

静态方法是类中的函数，不需要实例。静态方法主要是用来存放逻辑性的代码，逻辑上属于类，但是和类本身没有关系，也就是说在静态方法中，不会涉及到类中的属性和方法的操作。可以理解为，静态方法是个独立的、单纯的函数，它仅仅托管于某个类的名称空间中，便于使用和维护。

**定义：**
使用装饰器@staticmethod。参数随意，没有“self”和“cls”参数，但是方法体中不能使用类或实例的任何属性和方法；

**调用：**
类和实例对象都可以调用。

譬如，我想定义一个关于时间操作的类，其中有一个获取当前时间的函数。

```python
import time

class TimeTest(object):
    def __init__(self, hour, minute, second):
        self.hour = hour
        self.minute = minute
        self.second = second

    @staticmethod
    def showTime():
        return time.strftime("%H:%M:%S", time.localtime())


print(TimeTest.showTime()) # == 定义了全局静态变量，但是放在了某个类里面，和把函数放在外面使用一致，里面没有cls/self的参数
t = TimeTest(2, 10, 10)
nowTime = t.showTime()
print(nowTime)
```

如上，使用了静态方法（函数），然而方法体中并没使用（也不能使用）类或实例的属性（或方法）。若要获得当前时间的字符串时，并不一定需要实例化对象，此时对于静态方法而言，所在类更像是一种名称空间。其实，我们也可以在类外面写一个同样的函数来做这些事，但是这样做就打乱了逻辑关系，也会导致以后代码维护困难

**`@property`**

- 场景：想要“计算属性”，但又希望用户像访问属性一样用 `obj.attr` 而不是 `obj.method()`。
- 典型：`Circle.diameter`、`Rectangle.area`。

**`@classmethod`**

- 场景：工厂方法（如 `Date.from_string("2025-08-23")`），或需要修改/读取类级别状态。
- 典型：`dict.fromkeys()`、`datetime.fromtimestamp()`。

**`@staticmethod`**

- 场景：逻辑和类相关，但既不依赖实例，也不依赖类。主要是**归类用的工具函数**。
- 典型：`math` 操作、验证函数。
