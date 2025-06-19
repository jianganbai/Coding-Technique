# python面向对象学习笔记

- 面向对象的基本知识
  - 重载：同名函数，根据参数不同，决定调用哪个
  - 多态：父类指针调用虚函数，根据对象类型不同，决定调用基类/派生类的函数
  - 虚基类：解决相同参数重复继承
  - 虚函数：基类指针访问派生类同名函数
  - 纯虚函数：最原始的虚函数
  - 抽象类：含有纯虚函数的类，不能创建实例

## 类继承

- 父类构造函数
  
  - ```python
    class A:
        def __init__(self, a):
            self.a = a
    class B(A):
        def __init__(self, a):
            super().__init__(a)  # 父类构造函数
    ```

- 访问类的所有属性
  
  - ```python
    class A
    
    hasattr(A, 'B')  # 检查类A是否有方法B
    vars(A)  # 等价于实例化后a.__dict__
    dir(A)  # 类A所有属性和方法
    ```

## 静态变量

- 静态变量：在同类的不同实例之间共享数据

```python
class Counter:
    count = 0
    def __init__(self):
        Counter.count += 1
    @static_method
    def get_count(self):
        return Counter.count

count1 = Counter()
count2 = Counter()
print(Counter.get_count())  # 输出2
```

## 装饰器

```python
def b(func):  # 只能有一个参数，且为被包裹的函数
    def wrapper(*args, **kwargs):
        xxx
    return wrapper

@b
def a(x):
```

- 函数a作为参数送给函数b，函数b对函数a重新进行包裹，返回包裹后的函数
- 调用函数a实际调用函数b，不用修改函数a

### 例子

- 计算用时
  
  - ```python
    def time_decorator(func):  # 只能有一个参数，且为被包裹的函数
        def wrapper(*args, **kwargs):
            start_time = time.time()
            result = func(*args, **kwargs)
            end_time = time.time()
            print(end_time - start_time)
            return result
        return wrapper
    
    @time_decorator
    def slow_func(n):
        time.sleep(n)
    slow_func(n)
    ```

- 斐波那契数列缓存
  
  - ```python
    def cache_decorator(func):  # 只能有一个参数，且为被包裹的函数
        cache = dict()
        def wrapper(*args):
            if args in cache:
                return cache[args]
            else:
                result = func(*args)
                cache[args] = result
                return result
    @cache_decorator
    def fibonacci(n):
        if n < 2:
            return n
        else:
            return fibonacci(n-1) + fibonacci(n-2)
    ```

### 常见装饰器

- `@classmethod`装饰器：指定函数为**类方法**，可在创建类之前调用该函数
  
  - 第1个参数为cls，代表类本身
  
  - 可作为另一个构造函数
    
    - ```python
      class YUE:  # 两个参数不同的构造函数
          def __init__(self, a):  # python类中只能有1个__init__函数
              self.a = a
          @classmethod
          def construct(cls, string):
              b = cls(int(string))  # 调用默认构造函数
              return b
      ```

- `@staticmethod`装饰器：可在创建类之前调用指定的函数
  
  - 相比于`@classmethod`，没有cls参数，无法访问类本身的变量和函数
  
  - 可用于实现判断函数
    
    - ```python
      class YUE:
          @staticmethod
          def is_valid(a, b):
              return a < 10 and b < 10 and a <= b
      ```

- `@property`装饰器：指定的函数使用变量的方式访问
  
  - ```python
    class YUE:
        @property
        def yue_id(self):
            return 10
    y = YUE()
    y.yue_id  # 加了@propery，就不用y.yue_id()访问了
    ```

- `@abstractmethod`装饰器：指定函数为纯虚函数

## 特殊函数

- `__iter__`
- `__call__`