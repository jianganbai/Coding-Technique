# C&C++常用库

## string

- 头文件`#include <string.h>`
  - string和char\*是两种不同的类型

- 构建

  - ```c++
    // 构造
    string a = "Hello!";
    string s1("Hello!");
    string s2(4, 'K');  // "KKKK"
    string s3("12345", 1, 3);  // "234"，1为开始下标，3为结束下标（包含）
    string s4 = to_string(12345);  // int转string
    ```
    
  - ```c++
    // 相互转化
    string a = "Hello!";
    const char* p = a.c_str();  // string转char*（必须为const）
    string b = p;  // char*转string
    int c = atoi(a.c_str())  // string先转为char*，再利用atoi函数将string转为int
    ```

  - ```c++
    // 连接
    string s1("Hello"), s2("World");
    s1 = s1 + ' ';
    s1.append(s2);  // "Hello World"
    ```

- 切片

  - `a[index] = 'a'`|`a.at(index) = 'a'`：访问a的第index个（返回char类型），并修改之为'a'（地址结合）

    - `[]`在越界访问时不会报错，`.at()`越界访问时会报错

  - `a.substr(int n=0, int m=string::npos)`：从a的第n个开始，截取m个元素（默认截到最后），返回string

    - `string::npos`表示不存在的位置，类似于False，强制转化为int后为-1

  - ```c++
    string s1 = "1,2,3,4,5,6,7";
    string[] s2 = s1.split(',');
    ```

- 统计

  - `a.length()`：计算a的长度

- 查找find

  - ```c++
    string a = "Hello World!", b = "Hello";
    int n = 0
    a.find(b，n);  // 从n位置向后查找，若能找到则返回开始下标，若不能则返回string::npos
    a.rfind(b, n);  // 从n位置向前查找
    a.find_first_of(b);  // 查找b第1次出现的位置
    a.find_last_of(b);  // 查找b最后1次出现的位置
    ```

## vector

- vector是1个容器，给出各种数据类型的数组

- 创建

  - **创建时不指定大小，使用时可灵活扩容**

  - ```c++
    #include <vector>
    vector<int> a;  // 空的
    vector<int> a(10);  // 指定大小
    vector<int> a(10, 1);  // 初始值为1
    vector<int> a = {1,2,3,4,5};  // 指定内部大小
    vector<int> a(b);  // b也是vector，a被复制为b
    vector<int> a(b.begin(), b.begin()+3);  // 用b的第0-第2个元素构造a
    // 传统数组如此创建：int c[5] = {1,2,3,4,5};
    vector<int> a(c, c+5);  // 用int数组创建vector
    ```

- 常用操作

  - ```c++
    a.assign(b.begin(), b.begin()+3);  // a和b都是vector，将b的第0-第2个元素赋给a
    a.push_back(5);  // 在a的最后加入5
    a.insert(a.begin()+2, 5);  // 在a的第2个元素处插入5，后续的往后顺延。.begin()是1个迭代器
    ```

  - ```c++
    a.front();  // a的第0个元素
    a.back();  // a的最后1个元素
    a[i];  // a的第i个元素，仅能用于获取存在的元素！！！
    ```

  - ```c++
    a.clear();  // 清空a的元素
    a.empty();  // 判断a是否为空
    a.size();  // 返回a的大小
    ```

- 迭代器：类似指针

  - `a.begin()`返回第0个元素的迭代器；`a.end()`返回最后1个元素的迭代器

  - 类型为`vector<int>::Iterator`

  - ```c++
    for(vector<int>::Iterator p = a.begin(); p != a.end(); p++){  // 自增运算符向后移
        cout<<*p;
    }
    ```

## stack

- C++在\<stack\>头文件中实现了栈

  - ```c++
    #include <stack>
    using namespace std;
    
    stack<int> s;
    s.push(1);  // 入栈，先拷贝再入栈
    s.emplace(1);  // 原位入栈，地址结合
    s.pop();  // 出栈
    s.empty();  // 判断栈是否为空
    s.top();  // 获得最后入栈的元素
    
    stack<int> t;
    s.swap(t);  // 交换两个栈，读入t时为引用
    ```

## hash

- C++ 11推出了`unordered_map`：hash的首次实现

  - 存储<key, value>，类似于python的dict
  - 头文件：`#include <unordered_map>`

- 创建

  - ```c++
    unorder_map<int, int> m;
    m[1] = 0;  // 插入key=1, value=0的pair
    m.insert(n.begin(), n.end());  // 插入另一hash表的元素
    m.insert(pair<int, int>(0, 1));  // 按pair插入元素
    ```

- 调用方法

  - 判断key是否存在：`m.find(key)`，若存在返回对应迭代器（指针），若不存在则返回`m.end()`

  - ```c++
    unordered_map<int, int> m;
    auto iter = m.find(key);
    if (iter != m.end()){
        return iter->second;  // iter迭代器对应元素的value值
    }
    ```

- 例子

  - ```c++
    #include <unordered_map>
    // 寻找数组内出现次数为N的元素
    class Solution{
    public:
        int repeatedNTimes(vector<int>& A){
            size_t N = A.size() / 2;
            unordered_map<int, int> m;  // 给出key和value的格式
            for (auto e: A)
                m[e]++;
            for (auto& e: m){
                if (e.second == N){
                    return e.first
                }
            }
        }
    }
    ```

### auto

- auto：可自动推断数据类型，仅在语言块内使用

  - 适合类型比较复杂的（如iterator）、基于模板定义的（未知数据类型）

- 使用auto遍历vector

  - ```c++
    struct node{
        int id;
        int coor;
    };
    int main(void){
        vector<node> arr;
        node tmp;
        for (int i=1; i<10; i++){
            tmp.id = i;
            arr.push_back(tmp);
        }
        for (auto& n: arr){
            cout<<n.id<<" ";
        }
    }
    ```

