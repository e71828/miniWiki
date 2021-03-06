# 泛型编程

## 函数模板

### 提升 (Lifting)
`提升 (lifting)` 是指从一个或多个`具体的 (concrete)` 普通函数出发, 提取出一个`抽象的 (abstract)` 函数模板的过程, 是一种特殊的`泛化 (generalization)`.

[标准库算法](./algorithm.md)是`提升`和`函数模板`的典型示例.

### 语法
`T` 的实际类型将根据 `compare` 的`静态`调用方式在`编译期`决定:
```cpp
template <typename T>  // 模板形参列表
int compare(const T& v1, const T& v2) {
  if (v1 < v2) return -1;
  if (v2 < v1) return 1;
  return 0;
}
```

`inline`, `constexpr` 等修饰符应当位于`模板形参列表`与`返回值类型`之间:
```cpp
template <typename T>
inline T min(const T&, const T&);
```

| 建议 | 目的 |
| --- | --- |
| 尽量减少对 `T` 的要求          | 扩大模板函数的适用范围 |
| 用 `const T&` 作为函数形参类型 | 支持不可拷贝类型       |
| 只用 `<` 进行比较操作          | `T` 不必支持其他运算符 |

### 对函数重载的影响
编译器首先根据`类型转换次数`对所有待选 (模板和非模板) 函数进行排序:
- 如果正好有一个函数的匹配程度比其他函数更高, 则它将被选中.
- 如果匹配程度最高的函数有多个, 则
  - 如果其中只有一个`非模板`, 则它将被选中.
  - 如果其中没有非模板, 且有一个模板的`特化 (specialization)` 程度更高, 则它将被选中.
  - 否则, 该调用有歧义, 编译时会报错.


### [类型推断](./type_deduction.md)

### 显式模板实参
```cpp
template <typename T1, typename T2, typename T3> T1 sum(T2, T3);
```
这里的 `T2` 和 `T3` 可以由函数实参推断, 而 `T1` 必须`显式`给出:
```cpp
int i = 0;
long lng = 1;
auto val3 = sum<long long>(i, lng);  // long long sum(int, long)
```

### 函数模板的地址
使用函数模板的地址时, 必须确保所有模板形参可以被唯一地确定:
```cpp
template <typename T> int compare(const T&, const T&);

// T 可以被唯一地确定为 int, pf1 指向 compare<int> 的地址
int (*pf1)(const int&, const int&) = compare;

// 重载的 func, 均以函数指针为形参类型:
void func(int(*)(const double&, const double&));
void func(int(*)(const int&, const int&));
func(compare<int>);  // 正确: T 被唯一地确定为 int
func(compare);       // 错误: T 无法被唯一地确定
```

## 类模板

```cpp
template <typename T> class Blob {
 public:
  typedef T value_type;
  typedef typename std::vector<T>::size_type size_type;
  Blob();
  Blob(std::initializer_list<T> il);
  size_type size() const { return data->size(); }
  bool empty() const { return data->empty(); }
  void push_back(const T& t) { data->push_back(t); }
  void push_back(T&& t) { data->emplace_back(std::move(t)); }
  void pop_back();
  T& back();
  T& operator[](size_type i);
 private:
  std::shared_ptr<std::vector<T>> data;
  // throws msg if data[i] is not valid:
  void check(size_type i, const std::string& msg) const;
};
```

### 定义成员函数
类模板的成员函数可以在类的内部或外部定义.
在内部定义的成员函数是隐式内联的.

#### 在外部定义成员函数
在类模板外部定义的成员函数以 `template` 关键词 + 类模板的形参列表 开始, 例如:
```cpp
template <typename T>
void Blob<T>::check(size_type i, const std::string& msg) const {
  if (i >= data->size())
    throw std::out_of_range(msg);
}
template <typename T>
T& Blob<T>::back() {
  check(0, "back on empty Blob");
  return data->back();
}
template <typename T>
T& Blob<T>::operator[](size_type i) {
  check(i, "subscript out of range");
  return (*data)[i];
}
template <typename T>
void Blob<T>::pop_back() {
  check(0, "pop_back on empty Blob");
  data->pop_back();
}
```

#### 在外部定义构造函数
```cpp
template <typename T>
Blob<T>::Blob() : data(std::make_shared<std::vector<T>>()) {}
template <typename T>
Blob<T>::Blob(std::initializer_list<T> il)
    : data(std::make_shared<std::vector<T>>(il)) {}
```
为使用第二个构造函数, 初始化列表的元素类型必须与模板类型实参兼容:
```cpp
Blob<string> articles = {"a", "an", "the"};
```

### 定义静态成员
`Foo` 的每个`实例化 (instantiation)` 都有其自己的静态成员 (数据或方法) `实例 (instance)`:
```cpp
template <typename T> class Foo {
 public:
  static std::size_t count() { return ctr; }
 private:
  static std::size_t ctr;
};
```
而每个`静态数据成员`应当有且仅有一个`定义`. 因此, 类模板的`静态数据成员`应当像`成员函数`一样, 在类的外部给出唯一的定义:
```cpp
template <typename T>
size_t Foo<T>::ctr = 0;  // 定义并初始化 ctr
```

### 使用类模板名

#### 在内部使用类名
`类模板名 (name of a class template)`  (不带模板实参) 不是一种`类型名 (name of a type)`, 但在类模板自己的作用域内, 可以不受此限制:

```cpp
template <typename T> class BlobPtr {
 public:
  BlobPtr() : curr(0) {}
  BlobPtr(Blob<T>& a, size_t sz = 0) : wptr(a.data), curr(sz) {}
  T& operator*() const {
    auto p = check(curr, "dereference past end");
    return (*p)[curr];  // (*p) is the vector to which this object points
  }
  // 返回值类型写为 BlobPtr& 而不是 BlobPtr<T>&
  BlobPtr& operator++();
  BlobPtr& operator--();
 private:
  // check returns a shared_ptr to the vector if the check succeeds 
  std::shared_ptr<std::vector<T>> check(std::size_t, const std::string&) const;
  // store a weak_ptr, which means the underlying vector might be destroyed 
  std::weak_ptr<std::vector<T>> wptr;
  std::size_t curr;  // current position within the array
};
```
在这里, 自增自减运算符的返回值类型可以写为 `BlobPtr&` 而不是 `BlobPtr<T>&`. 这是因为在类模板作用域内, 编译器将`类模板名`视为带有模板实参的`类型名`:
```cpp
// 相当于
BlobPtr<T>& operator++();
BlobPtr<T>& operator--();
```

#### 在外部引用类名
在类模板外部定义成员时, 类模板的作用域起始于 (带模板实参的) 类名. 因此在 `::` 之前需要显式写出模板实参, 而在其之后则不用:
```cpp
template <typename T>
BlobPtr<T> BlobPtr<T>::operator++(int) {
  BlobPtr ret = *this;  // save the current value
  ++*this;  // advance one element; prefix ++ checks the increment
  return ret;  // return the saved state
}
```

#### (C++11) 类模板别名
`实例化的 (instantiated)` 类模板是一种具体类型, 可以用 `typedef` 为其定义别名, 而对模板名称则不可以:

```cpp
typedef Blob<string> StrBlob;  // OK
typedef std::map TreeMap;      // error
```

C++11 允许用 `using` 为类模板定义别名:
```cpp
// twin 仍是类模板:
template<typename T> using twin = pair<T, T>;
// authors 的类型是 pair<string, string>
twin<string> authors;
```
这一机制可以用来固定一个或多个模板形参:
```cpp
// 固定第二个类型:
template <typename T> using partNo = pair<T, unsigned>;
// books 的类型是 pair<string, unsigned>
partNo<string> books;
```

### 友元 (少用)

`友元 (friend)` 机制破坏了类的封装, 因此要尽量少用.

如果类模板的友元不是模板, 那么它对该模板的`所有实例化`都是友元.

如果 (模板或非模板) 类的友元本身就是一个模板 (函数或类), 那么友元关系有以下几种可能.

#### 一一对应的模板友元
```cpp
// 前置声明:
template <typename> class BlobPtr;
template <typename> class Blob;
template <typename T>
bool operator==(const Blob<T>&, const Blob<T>&);
// 以 Blob 的模板实参作为友元的模板形参:
template <typename T> class Blob {
  // BlobPtr<T> 和 operator==<T> 是 Blob<T> 的友元
  friend class BlobPtr<T>;
  friend bool operator==<T>(const Blob<T>&, const Blob<T>&);
};
```

#### 一般与特定的模板友元
一个 (模板或非模板) 类可以指定一个类模板的`所有`或`特定`的实例化作为其友元:
```cpp
// 前置声明:
template <typename T> class Pal;
// 指定 普通类 的友元:
class C {
  // Pal<C> 是 C 的友元:
  friend class Pal<C>;
  // Pal2<T> 是 C 的友元, 不需要前置声明 Pal2:
  template <typename T> friend class Pal2;
};
// 指定 类模板 的友元:
template <typename T> class C2 {
  // Pal<T> 是 C2<T> 的友元, 需要前置声明 Pal:
  friend class Pal<T>;
  // Pal2<X> 是 C2<T> 的友元, 需要前置声明 Pal2:
  template <typename X> friend class Pal2;
  // Pal3 是 C2<T> 的友元, 不需要前置声明 Pal3:
  friend class Pal3;
};
```

#### (C++11) 将模板类型形参设为友元
```cpp
template <typename Type> class Bar {
  friend Type;
};
```

## 成员模板

### 普通类的成员模板

为`普通类`定义`函数成员模板`:

```cpp
class DebugDelete {
 public:
  DebugDelete(std::ostream& s = std::cerr) : os(s) { }
  template <typename T> void operator()(T* p) const {
    os << "deleting unique_ptr" << std::endl;
    delete p; 
  }
 private:
  std::ostream& os;
};
```
该类的实例可以用于替代 `delete`:
```cpp
int* ip = new int;
DebugDelete()(ip);  // 临时对象

DebugDelete del;
double* dp = new double;
std::unique_ptr<int, DebugDelete> dp(new int, del);
```

### 类模板的成员模板

为`类模板`声明`函数模板`成员, 二者拥有各自独立的模板形参:

```cpp
template <typename T> class Blob {
  template <typename Iter> Blob(Iter b, Iter e);
};
```
如果在类模板的外部定义函数成员模板, 应当
- 先给出类的模板形参列表
- 再给出成员的模板形参列表
```cpp
template <typename T>
template <typename Iter>
Blob<T>::Blob(Iter b, Iter e)
    : data(std::make_shared<std::vector<T>>(b, e)) {}
```

## 模板形参

### 类型形参

在模板形参列表中, 关键词 `class` 与 `typename` 没有区别:

```cpp
template <typename T, class U>
int calc(const T&, const U&);
```

### 非类型形参
非类型形参的值在`编译期`确定 (人为`指定`或由编译器`推断`), 因此必须为`常量表达式 (constexpr)`:

```cpp
template<unsigned N, unsigned M>
int compare(const char (&p1)[N], const char (&p2)[M]) {
  return strcmp(p1, p2);
}
// 如果以如下方式调用
compare("hi", "mom");
// 该模板将被实例化为
int compare(const char (&p1)[3], const char (&p2)[4]);
```

### 模板形参的作用域
模板形参遵循一般的作用域规则, 但已经被模板形参占用的名字在模板内部`不得`被复用:

```cpp
typedef double A;
template <typename A, typename B> 
void f(A a, B b) {
  A tmp = a;  // tmp 的类型为模板形参 A 而不是 double
  double B;   // 错误: B 已被模板形参占用, 不可复用
}
// 错误: 复用模板形参名
template <typename V, typename V>  // ...
```

### 模板声明
与函数形参名类似, 同一模板的模板形参名在各处声明或定义中不必保持一致.

一个文件所需的所有模板声明, 应当集中出现在该文件头部, 并位于所有用到这些模板名的代码之前.

### 模板形参的类型成员
默认情况下, 编译器认为由 `::` 获得的名字不是一个类型. 因此, 如果要使用模板形参的类型成员, 必须用关键词 `typename` 加以修饰:
```cpp
// T 为一种 容器类型, 并且拥有一个类型成员 value_type
template <typename T>
typename T::value_type top(const T& c) {
  if (!c.empty())
    return c.back();
  else
    return typename T::value_type();
}
```

### (C++11) 默认模板实参
```cpp
template <typename T, typename F = std::less<T>>
int compare(const T& v1, const T& v2, F f = F()) {
  if (f(v1, v2)) return -1;
  if (f(v2, v1)) return 1;
  return 0;
}
```
调用时, 可以 (而非必须) 为其提供一个比较器:

```cpp
bool i = compare(0, 42);
bool j = compare(item1, item2, compareIsbn);
```

如果为所有模板形参都指定了默认模板实参, 并且希望用它们来进行默认实例化, 则必须在模板名后面紧跟 `<>`, 例如:

```cpp
template <class T = int> class Numbers {
 public:
  Numbers(T v = 0): val(v) { }
 private:
  T val; 
};
Numbers<long double> lots_of_precision;
Numbers<> average_precision;  // Numbers<> 相当于 Numbers<int>
```

## 实例化 (Instantiation)

### 构建过程
这里笼统地将 C++ 程序的构建过程分为以下两个步骤:
1. 编译 --- 由`编译器 (compiler)` 将`源 (source) 文件`转化为`目标 (object) 文件`
2. 链接 --- 由`链接器 (linker)` 将一个或多个`目标文件`转化为一个`可执行 (executive) 文件`

### 实例化
`模板定义`是一种特殊的源码, 其中含有待定的`模板形参`, 因此编译器无法立即生成目标码.
如果模板在定义后被使用, 则编译器将对其进行`实例化 (instantiation)`:

1. 根据上下文确定模板实参:
  - 对于`函数模板`, 编译器 (通常) 会利用`函数实参`来推断`模板实参`. 如果所有的模板实参都可以被推断出来, 则不必显式地指定.
  - 对于`类模板`, 必须显式地指定模板实参.
2. 用模板实参替换模板实参, 定义出具体的类或函数.
3. 将其编译为目标码:
  - 编译器 (通常) 会为源文件中的每一种模板实参 (组合) 生成一个独立的`实例 (instance)`, 对应于目标文件中的一段目标码.
  - 编译器 (通常) 只会为那些被使用的成员函数生成实例.

### (C++11) 显式实例化
对于一个模板, 必须知道其`定义`才能进行实例化.
因此, 通常将模板的`定义`置于头文件 (`.h` 或 `.hpp`) 中.
这样做的好处是代码简单, 错误容易在`编译期 (compile-time)` 被发现, 
缺点是容易造成目标代码冗余, 即相同目标码重复出现在多个目标文件中.

为克服上述缺点, C++11 引入了`显式实例化`机制:
```cpp
extern template declaration;  // 实例化声明
       template declaration;  // 实例化定义
```
在这里, `declaration` 是一条类或函数的声明, 其中的所有`模板形参`被`显式地`替换为`模板实参`.
每一条`实例化声明`都必须有一条位于其他某个源文件中的`实例化定义`与之对应:
- 对于含有`实例化声明`的`源文件`, 与之对应的`目标文件`不含该实例的目标码.
- 对于含有`实例化定义`的`源文件`, 与之对应的`目标文件`含有该实例的目标码.
  - 类模板的`实例化定义`将导致该类模板的所有成员函数 (含内联成员函数) 全部被实例化.
- 这种对应关系将在`链接期 (link-time)` 进行检查.

在以下两个源文件所对应的目标文件中,
- `application.o` 将含有 `Blob<int>` 及其`列表初始化`和`拷贝`构造函数的目标码:
```cpp
// application.cc
Blob<int> a1 = {0,1,2,3,4,5,6,7,8,9};
Blob<int> a2(a1);
extern template class Blob<string>;  // 实例化声明
Blob<string> sa1, sa2;
extern template int compare(const int&, const int&);  // 实例化声明
int i = compare(a1[0], a2[0]);
```
- `template_build.o` 将含有 `compare<int>`  以及 `Blob<string>` 的目标码:
```cpp
// template_build.cc
template class Blob<string>;  // 实例化定义, 所有成员函数将被实例化
template int compare(const int&, const int&);  // 实例化定义
```

## 特化 (Specialization)

### 函数模板的特化
假设有以下两个版本的同名函数:
```cpp
// 版本一, 用于比较两个 任意类型的对象:
template <typename T>
int compare(const T&, const T&);
// 版本二, 用于比较两个 字符串字面值 或 字符数组:
template <size_t N, size_t M>
int compare(const char (&)[N], const char (&)[M]);
```
在第二个例子中, `T` 被推断为 `const char*`, 因此比较的是两个`地址`:
```cpp
// 传入 字符串字面值, 实例化并调用 compare(const char (&)[3], const char (&)[4])
compare("hi", "mom");
// 传入 指向字符常量的指针, 实例化并调用 compare(const char*&, const char*&)
const char* p1 = "hi";
const char* p2 = "mom";
compare(p1, p2);
```
为了使这种情形下的语义变为`比较两个 (C-风格) 字符串`, 应当对版本一进行`特化 (specialization)`:
```cpp
#include <cstring>
template <>  // 为 T 为 const char* 的情形提供特化版本
int compare(const char* const& p1, const char* const& p2) {
  return std::strcmp(p1, p2);
}
```

`特化`对模板进行`实例化`, 而不是`重载`同名函数, 因此不会影响重载函数的匹配.

模板及其特化应当在同一个头文件中进行声明, 并且应当先给出所有同名`模板`, 再紧随其后给出所有`特化`.

### 类模板的特化

#### (C++11) `std::hash` 的特化
`std::hash` 是一个类模板, 定义在头文件 `<functional>` 中:
```cpp
template <class Key>
struct hash;
```
标准库中的无序容器 (例如 `std::unordered_set<Key>`) 以 `std::hash<Key>` 为其默认散列函数.
对 `std::hash` 进行特化, 必须为其定义:
- 一个重载的调用运算符: `std::size_t operator()(const Key& key) const noexcept`
- 两个类型成员 (C++17 淘汰): `argment_type` 和 `result_type`
- 默认构造函数: 可以采用隐式定义的版本
- 拷贝赋值运算符: 可以采用隐式定义的版本

假设有一个自定义类型 `Key`:
```cpp
class Key {
 public:
  std::size_t hash() const noexcept;
}
bool operator==(const Key& lhs, const Key& rhs);  // Key 必须支持 == 运算符
```
则 `std::hash<Key>` 可以定义为
```cpp
namespace std {
template <>
struct hash<Key> {
  typedef Key argment_type;
  typedef size_t result_type;
  size_t operator()(const Key& key) const noexcept { return key.hash(); }
  // 默认构造函数 和 拷贝赋值运算符 采用隐式定义的版本
};
}
```

#### 偏特化 (Partial Specialization)
`偏特化` (又称`部分特化`) 只指定`一部分模板实参`或`模板形参的一部分属性`.
只有类模板可以进行偏特化. 偏特化的类模板依然是类模板.

`std::remove_reference` 就是通过一系列偏特化 (只指定`模板形参的一部分属性`) 来完成工作的:
```cpp
// 原始版本
template <class T> struct remove_reference {
  typedef T type;
};
// 偏特化版本, 分别适用于`左值`和`右值`引用
template <class T> struct remove_reference<T&> { typedef T type; };
template <class T> struct remove_reference<T&&> { typedef T type; };
```
