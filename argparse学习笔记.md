# argparse学习笔记

## 基本概念

- argparse模块提供了从命令行`cmd`中读取参数的功能

- ```python
  parser = argparse.ArgumentParser()  # 例化用于分析命令行的对象
  parser.add_argument()  # 加入需要从命令行中提取出的属性
  args = parser.parse_args()  # 分析结果
  ```

## 语法

### `argparse.ArgumentParser()`

- 例化时可加入参数

  - `prog`：使用-h显示帮助时，显示出的程序名字
  - `description`：程序的功能，在使用-h显示帮助会出现

- ```markdown
  **prog** - The name of the program (default: sys.argv[0])
  
  **usage** - The string describing the program usage (default: generated from arguments added to parser)
  
  **description** - Text to display before the argument help (default: none)
  
  **epilog** - Text to display after the argument help (default: none)
  
  **parents** - A list of ArgumentParser objects whose arguments should also be included
  
  **formatter_class** - A class for customizing the help output
  
  **prefix_chars** - The set of characters that prefix optional arguments (default: ‘-‘)
  
  **fromfile_prefix_chars** - The set of characters that prefix files from which additional arguments should be read (default: None)
  
  **argument_default** - The global default value for arguments (default: None)
  
  **conflict_handler** - The strategy for resolving conflicting optionals (usually unnecessary)
  
  **add_help** - Add a -h/--help option to the parser (default: True)
  
  **allow_abbrev** - Allows long options to be abbreviated if the abbreviation is unambiguous. (default: True)
  ```

### `parser.add_argument()`

- 提供从命令行中分析的属性
- ``ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest])`
  - 增加属性时，仅`name or flags`不可缺省，其余均可缺省
- `name or flags`为分析命令行时该属性的名称
  - 不加`-`代表不可缺省的属性（该属性在命令行中不可出现），加`-`或`--`代表可缺省的属性（`-`一般为简写，`--`一般为全名）
  - 不可缺省的属性：
    - 命令行输入时不需要在值前面加上属性名
    - 符合`type`时，可连续读多个，返回列表
  - 可缺省的属性：
    - 命令行输入的格式为**属性名 属性值**
- `action`为用于简单处理命令行参数的内置函数
  - `action='store_true'`：只要出现了可缺省属性的名字，则该属性记为True，一般与`default`配合
  - `action='store_false'`：只要出现了可缺省属性的名字，则该属性记为False
  - `action='append'`：允许某属性出现多次，将多次出现结果合并为列表并返回
  - `action='store_const'`：只要出现了可缺省属性的名字，则该属性赋值为`const`
- `const`与`action='store_const'`相结合
- `default`为该属性在命令行中缺省时的默认值
- `choices`为该属性所有可取的值，不在其中则报错
- `required`为该参数是否能被省略
- `help`为该属性的功能，使用-h显示帮助时会显示出来
- `dest`为该属性存储在分析结果对象`args`中的名称
- 基础属性`-h`：显示帮助

### `args = parser.parse_args()`

- 括号缺省则分析命令行，括号内部可填上带有切分成参数的列表，这样就分析这个列表
  - 命令行的构成为`python [文件名] [参数]`，这里跳过前2个参数，直接分析命令行后面的参数
  - 指定参数的例子：`args = parser.parse_args('1 2 3 -foo 4'.split())`
- `args`为带有分析结果的对象
  - 访问对象中的数据时，直接用`args.dest`，`dest`为之前指定的名称，未指定则用原始的`name or flags`

## 使用

- 命令行组成：`python [文件名] [参数]`

- `python [文件名] -h`给出了命令行参数的规范，按照该规范输入即可

