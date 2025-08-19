# Reflection

> 反射的概念是由Smith在1982年首次提出的，主要是指**程序可以访问、检测和修改它本身状态或行为**的一种能力。

在程序运行时可以获取对象类型定义信息，例如，Python中的type(obj)将返回obj对象的类型，这种<u>获取对象的type、attribute或者method的能力</u>称为反射。通过反射机制，可以用来检查对象里的某个方法，或某个变量是否存在。也就是可以**通过字符串映射对象的方法或者属性**。

## Reflection Built-in Function

- **type**(obj)：返回对象类型
- **isinstance**(object, classinfo)：判断一个对象是否是一个已知的类型，类似 type()
- **callable**(obj)：对象是否可以被调用
- **dir**([obj])：返回obj属性列表
- **getattr**(obj, attr)：返回对象属性值
- **hasattr**(obj, attr)：判断某个函数或者变量是否存在
- **setattr**(obj, attr, val)：给模块添加属性（函数或者变量）
- **delattr**(obj, attr)：删除模块中某个变量或者函数

## Usage of Reflection function

create a class:

```python
class Person():
    def __init__(self, x, y):
        self.age = x
        self.height = y
        
    def __new__(cls, *args, **kwargs):
        print("begin!!!")
        return object.__new__(cls)
        
    def __call__(self, *args, **kwargs):
        print("hello!!!")

    def talk(self):
        print(f"My age is {self.age} and height is {self.height}")

```

### dir()

利用反射的能力，我们可以通过属性字典`__dict__`来访问对象的属性：

```python
p = Person(20, 180) # begin!!!
print(p) # <__main__.Person object at 0x7fa718124850>
p() # hello!!! 
print(p.__dict__) # {'age': 20, 'height': 180}
p.__dict__['age']=22
print(p.__dict__) # {'age': 22, 'height': 180}
p.weight = 60
print(p.__dict__) # {'age': 22, 'height': 180, 'weight': 60}
print(dir(p)) # ['__call__', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'age', 'height', 'talk', 'weight']
```

- 在实例创建之前调用`__new__`方法，返回值（实例）将传递给`__init__`方法的第一个参数。`__new__`方法的详细介绍可参考：[Python中的`__new__`和`__init__`](https://hiyongz.github.io/posts/python-notes-for-reflection/)
- 实例化对象时会自动执行 `__init__` 方法
- 打印一个对象时，会自动执行`__str__` 方法
- 调用实例化对象时，会自动触发`__call__` 方法
- 通过`dir()`方法可以打印出了对象p的属性。
- `__dict__`
  - `obj.__dict__` (instance) --> returns object's data attributes
  - `Class.__dict__` (class) --> returns the class's namespace, including methods, class variables, and descriptors.

### callable()

```python
if (callable(p)):
    print("p is callable")
else:
    print("p is not callable")
# p is callable
```

### isinstance() & type()

```python
print(isinstance(p, Person)) # True
print(type(p) == Person) # True
print(isinstance(p.age, int)) # True
print(type(p.age) == int) # True
```

### getattr() & hasattr()

```python
print(getattr(p,"talk")) # <bound method Person.talk of <__main__.Person object at 0x000001FF52868288>>
print(getattr(p.talk, "__call__")) # <method-wrapper '__call__' of method object at 0x000001FF52155048>

print(hasattr(p,"talk")) # True
if hasattr(p,'walk'):
    print(getattr(p,'walk'))
else:
    print("I can't walk")
# I can't walk

print(getattr(p, "walk", None)) # 如果没有walk属性就返回None # None
```

### setattr()

```python
setattr(p,'walk','ON')

if hasattr(p,'walk'):
    print(getattr(p,'walk')) # on
else:
    print("I can't walk")
print(p.__dict__) # {'age': 22, 'height': 180, 'weight': 60, 'walk': 'ON'}
```

### delattr()

```python
delattr(p,'walk')
if hasattr(p,'walk'):
    print(getattr(p,'walk'))
else:
    print("I can't walk") # I can't walk
print(p.__dict__) # {'age': 22, 'height': 180, 'weight': 60}
```

## Applications

### 动态调用

可以通过**字符串**来获取对象的属性`getattr()`

比如，一个类中有很多方法，它们提供不同的服务，通过输入的参数来判断执行某个方法，一般的使用如下写法：

```python
# bad example
class MyService():
    def service1(self):
        print("service1")

    def service2(self):
        print("service2")

    def service3(self):
        print("service3")

if __name__ == '__main__':
    Ser = MyService()
    s = input("请输入您想要的服务: ").strip()
    if s == "service1":
        Ser.service1()
    elif s == "service2":
        Ser.service2()
    elif s == "service3":
        Ser.service3()
    else:
        print("error!")
        
# good example
if __name__ == '__main__':
    Ser = MyService()
    s = input("请输入您想要的服务: ").strip()
    if hasattr(Ser, s):
        func = getattr(Ser, s)
        func()
    else:
        print("error!")
```

### 动态属性设置

可以通过`setattr()`方法进行动态属性设置，在使用scapy库构造报文时，我们需要设置某些报文字段，然而网络协议的报文字段很多，在需要设置大量字段时，一个一个的赋值就很麻烦：

```python
>>> ls(IP)
version    : BitField  (4 bits)                  = ('4')
ihl        : BitField  (4 bits)                  = ('None')
tos        : XByteField                          = ('0')
len        : ShortField                          = ('None')
id         : ShortField                          = ('1')
flags      : FlagsField                          = ('<Flag 0 ()>')
frag       : BitField  (13 bits)                 = ('0')
ttl        : ByteField                           = ('64')
proto      : ByteEnumField                       = ('0')
chksum     : XShortField                         = ('None')
src        : SourceIPField                       = ('None')
dst        : DestIPField                         = ('None')
options    : PacketListField                     = ('[]')

# use setattr() assign the values
from scapy.all import *

fields = {"version":4, "src":"192.168.0.1","dst":"192.168.10.1"}
ip = IP()
for key, val in fields.items():
    setattr(ip, key, val)
```