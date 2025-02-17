# python数据类型管理学习笔记

## typing

- 功能：标注函数输入输出的类型，**但运行时不检查类型**
  
  - 与pylance配合，实时检查变量在函数内的调用，是否符合该类型
  - python 3.5引入

- 语法
  
  - ```python
    def 函数名(参数: 数据类型) -> 返回值类型:
    def 函数名(参数: 数据类型 = 默认值) -> 返回值类型:
    ```

- 基本类型
  
  - int, float, bool, str, bytes
  - `Union`：返回值有多个类型，`Union[int, None]`也可写成`int | None`
  - `Tuple`：固定长度的元组，如：`Tuple[torch.Tensor, torch.Tensor]`
    - 不定长：`Tuple[int, ...]`
  - `List`：列表，如：`List[int, float, str]`
  - `Dict`：字典，`Dict[key类型, value类型]`
  - `Set`：集合，`Set[元素类型]`
  - `Literal`：指定取值集合，`Literal['foo', 'bar']`
    - `Union`：多种类型；`Literal`：多种取值

```python
from typing import Union, Tuple, List, Dict

def func1(a: int, s: str, f: float) -> Tuple[List, Dict]:
    return [list(range(a)), {s: f}]
def func2(a: int, s: str = 'aaa') -> List[int or str]:
    return [a, s]
def func3(a: int, s: str = 'bbb') -> Dict[str, Union[int, str]]:
    return {'a': a, 's': s}
```

- 泛型
  
  - `Any`：任意数据类型
  
  - `Callable`：可调用对象
  
  - `Optional`：可以为给定类型或None，`Optional[float]`等价于`float | None`
    
    - `Optional[Literal['a', 'b']]`
    - **不表示该变量有默认值**
  
  - `Iterable`：可迭代对象
  
  - `Generator`：生成器
    
    ```python
    # The generator yields values of type int.
    # It does not accept any values sent to it (hence None).
    # When the generator completes, it returns a value of type str.
    def my_generator() -> Generator[int, None, str]:
        yield 1
        yield 2
        yield 3
        return "Done"
    ```
  
  - `Type`：所有类
    
    - `Type[Animal]`：指定为Animal 类（可以是基类）
  
  ```python
    from  typing import Callable, Iterable
  
    def processor(f: Callable, a: Iterable) -> Iterable:
      return f(a)
  ```

- 其他
  
  - ```python
    def process_object(obj: A) -> None:  # obj是类A的对象
    def create_instance(cls: type[A], value: int) -> A:  # cls就是类A
    ```

## dataclass

- **数据类**：只存数据的类，不定义方法
  
  - 特点：按`对象名.属性名`访问数据；初始化时可有默认值；初始化时缺少某属性会报错
  - **适合与typing配合**；python 3.7引入

- **数据类定义**
  
  - ```python
    from dataclass import dataclass
    from typing import Any, List
    @dataclass
    class Player:
        name: str  # name, skill, age这种称为字段
        skill: List[Any]
        age: int = 20
    p = Player('foo', ['bar', 'baz'])
    print(p.name)
    p.name = 'haha'
    ```

- **可变数据类型的初始化：field**
  
  - 默认不能用可变数据类型初始化：可变数据类型会返回引用
  
  - ```python
    from dataclass import dataclass, field
    from typing import Any, List
    @dataclass
    class Player:
        name: str
        skill: List[Any] = field(default_factory=lambda : ['sleep'])
    ```
  
  - field参数
    
    - `default`：字段的默认值；`default_factory`：返回字段初始值的函数
    - metadata：注释信息，如`metadata={'help': 'player skills'}`
