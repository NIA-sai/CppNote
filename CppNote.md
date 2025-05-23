# Cpp Note

> by NIA-sai
>
> [github](https://github.com/NIA-sai/CppNote) 

~2025年4月26日16点38分~

# todo

- RAII



## OOP

### 权限

- `friend` 可以写在类的任何位置

- | 父类成员权限     | public继承 | protected继承    | private继承    |
  | ---------------- | ---------- | ---------------- | -------------- |
  | `public` 成员    | 不变       | 变成 `protected` | 变成 `private` |
  | `protected` 成员 | 不变       | 变成 `protected` | 变成 `private` |
  | `private` 成员   | 无法访问   | 无法访问         | 无法访问       |

### Virtual

- 会无法取消地继承

- virtual`~`

- `=0` 纯虚

### 方法

- `=default`强制生成默认函数
- `=delete`禁用函数（阻止调用或生成）
- 基类构造器在初始化列表直接这样 `MyClass(int x) : Base(x) `
- 有显式对象形参的函数体内不能使用 this 指针
- 成员函数后面加&只能被r-value的对象调用
- && 只能l-value

#### 拷贝

- 狗操的浅拷贝和局部变量（btw std::sort没有考虑到无深拷贝方法的”低端数组“）
  tip: 手动将局部变量转移让他去瞎几把delete

### 基类

- 基类的 初始化顺序 取决于它们在 继承时 的顺序 ，与初始化列表中的顺序无关。

### 成员

- 成员变量的 初始化顺序 取决于它们在 类 中的 声明顺序 ，与初始化列表中的顺序无关。
- const和引用和无默认构造的成员需要在初始化列表内初始化

### 继承

- `std::any` (c:void*,go:any,java:Object)
  `std::any_cast</*truetype*/>`

### 多态

- 对象不能实现多态

- `typeid` 返回实际类型的type_info







-----











## 结构体/联合体

- `std::variant<type1,type2,...>`类型安全性的union?
  结合`std::visit([](auto&& val){},v)`
  或者`std::holds_alternative<type>(v)`(bool), `std::get<type>(v)`





-----







## 汇编

- 查看反汇编: `-exec disassemble /m`





-----





## 模板

- `typename`和`class`等价并非class就不能接受struct

- ... 的用法很是糟糕,`template<typename...Args>` 有具体意义：“ **变参模板**”

- 用 `concept//#include <concepts>` 优雅约束(编译时检验)类型:
  ```c++
  template <typename T>
  concept Addable = requires(T a, T b) {
      { a + b } -> std::same_as<T>;  // 要能加，并且结果也是 T//加 {} 的形式——表达式的类型和值要求
      cout<<a;//要能输出//不加 {} 的形式——表达式是否有效
  };
  
  template <Addable T>
  T add(const T& a, const T& b) {
      return a + b;
  }
  ```




  std提供了一些标准概念：

  | Concept 名                   | 意义                      |
  | ---------------------------- | ------------------------- |
  | `std::integral`              | 整型（int, long, ...）    |
  | `std::floating_point`        | 浮点类型（float, double） |
  | `std::same_as<T>`            | 类型与 T 相同             |
  | `std::derived_from<Base>`    | 是某类的派生类            |
  | `std::default_initializable` | 可以默认构造              |
  | `std::invocable<F, Args...>` | 可调用对象                |

-  非类型模板参数`template <int N>` ,`N` 是 **编译时常量**
  例子

  ```c++
  template <int N>
  void printNTimes(const std::string& s) {
      for (int i = 0; i < N; ++i)
          std::cout << s << '\n';
  }
  int main() {
      printNTimes<3>("Hello");  // 打印 3 次
  }
  
  template <int Rows, int Cols>
  struct Matrix {
      int data[Rows][Cols];
  };
  //Matrix<1, 2>和Matrix<3,4>是不同的struct
  ```

- 模板模板参数: `template <template <typename> class Container>` 表示参数 `Container` 是一个接收 `typename` 的模板

- 嵌套依赖名必须加 `typename`: 
  ```c++
  template <typename T>
  void func() {
      typename T::value_type x;  // 若不加 typename，编译器会尝试将 T::XXX 视作静态成员
  }
  ```





****





-----









## Lambda

```cpp
[capture](parameters) -> return_type {
    // function body
}
```

- `[capture]`
  
  | 写法  | 说明                       |
  | ----- | -------------------------- |
  | `[=]` | 捕获外部所有变量，按值拷贝 |
  | `[&]` | 捕获外部所有变量，按引用 |
  | `[x]` | 只按值捕获 `x` |
  | `[&x]` | 只按引用捕获 `x` |
  | `[=, &y]` | 捕获其它变量用值，`y` 用引用 |
  | `[this]` | 捕获当前对象（常用于类成员里） |
  
-   也可以存在`std::function<>`里

- 不用上一点而实现递归的奇技淫巧：

  “Y组合子”技巧（gpt:有点函数式风格,虽然有点 nerd，但挺酷的 😎

  ```cpp
  auto qs = [&](auto&& self, size_t l, size_t h) -> void {
      if (l < h) {
          size_t p = partition(l, h);
          if (p > 0) self(self, l, p - 1);
          self(self, p + 1, h);
      }
  };
  qs(qs, low, high);//传入自身
  ```





`



-----







## 宏

- `__FUNCSIG__`
- `_CRT_SECURE_NO_WARNINGS`(msvc用于禁用与 **CRT**（C Runtime Library）安全函数相关的编译器警告)

- `##` 按字符串连接
- `#`转换为字符串
- `\`多行







-----







## 指针

- 指针按值传递只能修改指针指向的内容的内容，若要修改指向的内容，需要使用二级指针或者引用指针

- `*` 号前的几乎意味指针指向的类型及类型的修饰，之后再是对指针的修饰，如
  `const int* const& a`
  常量整型         常指针的引用









### 函数指针

- 直接定义`返回值类型 (*名称)(参数类型,);`（以`void (*func_ptr)(); `为例）

- 通过 `typedef` 简化:`
typedef void (*FuncPtr)();// 定义类型别名` `FuncPtrFuncPtr func_ptr;// 声明变量`

- 作为类型`void (*)()`

- new 出来的指针也是需要自己 delete 的





-----







## 引用

- `&&`右值引用 一般用于移动构造函数







------







## 内存

-  必须手动 `delete` 的场景
  1. 直接使用 `new`/`new[]` 分配的裸指针。
  2. 类成员持有动态资源且未使用 RAII 封装。
  3. 工厂函数返回动态对象且未用智能指针。
  4. 多态对象的基类指针删除派生类对象（需虚析构函数）。

- `new operator`  操作符做两件事，分配内存+调用构造函数初始化

  `operator new` 只分配内存 `void *rawMemory = operator new(sizeof(string))`,`//new[](n*sizeof())`
  `new operator` 操作符为分配内存所调用函数的j就是 `operator new`
  `placement new` 对一些已经（用operator new）被分配可是尚未处理的的(raw)内存，你须要在这些内存中构造一个对象`new (rwa) Object();`

  全都是返回指针
  使用`operator new` 需要用 `operator delete`(不会调用析构函数) 来delete

- 最好到处=nullptr

- 浅拷贝其一问题：局部变量调用析构函数使得原变量指针delete



-----







## 错误处理

- `noexcept` 用于明确指示函数 不会抛出异常
  条件性 `noexcept`： `noexcept(/*bool*/)` , `noexcept(func());  // 若 func() 不抛异常，返回 true`,  根据编译期布尔表达式决定是否标记为 `noexcept`







-----





## Attribute

- [[deprecated]] 弃用
- [[nodiscard]] 不可忽略返回值









## 其他

- explicit 禁隐式，构造函数更安全
  ````cpp
  class MyClass {
  public:
      explicit MyClass(int x) { std::cout << "MyClass(" << x << ")\n"; }
  };
  
  void print(MyClass m) {}
  
  int main() {
      print(10);  // ❌ 编译错误：不能从 int 隐式构造 MyClass
      print(MyClass(10));  // ✅ 显式构造就没问题
  }
  
  class MyClass {//c++11
  public:
      explicit operator bool() const {
          return true;
      }
  };
  
  ````

- 根据 ADL(Argument-Dependent Lookup，参数相关查找), using 了未必会一定使用对应 namespace 的成员，因此
  ````
  using std::swap;
  swap(*a,*b);
  ````

  确有其用

- `<=>` 返回值为

- ```
  std::strong_ordering
  std::weak_ordering//忽略大小写
  std::partial_ordering//浮点数"NaN"
  ```

- `{}` 窄化转换（narrowing conversion）调用构造函数时不允许隐式类型转换

- typeid()对没有虚函数表的（没实现多态的）并不会返回实际类型，这很奇怪，就算只写virtual而不在基类覆盖，也会改变typeid()的行为

- 栈只有几MB

- 模板显式实例化（template explicit instantiation）技巧，可以在某些情况下绕过访问控制（访问 private 或 protected 成员）。
  ````cpp
  template<typename T, typename M, M T::*Member>
  struct Accessor {
      friend M get(T& obj) {
          return obj.*Member;
      }
  };
  
  // 专门实例化出 secret 指针：
  template struct Accessor<MyClass, int, &MyClass::secret>;
  ````

  
