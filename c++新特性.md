[toc] 



## **c++新特性**

### 显示转换

- 一个命名的强制类型转换具有如下形式：
  - `cast-name<type>(expression);`
- **static_cast**
  - 任何有明确定义的类型转换，只要不包含底层`const`就都可以使用`static_cast`
- **const_cast**
  - 只能改变对象的底层`const`

### **bind参数绑定**

- 定义在`<functional>`中


  - 需要  using namespace placeholders  引入命名空间 、以便单独使用\_1、_2...


  - 使用

    - 假设f是一个接受5个参数的可调用函数

      - ```c++
        auto g = bind(f, a, b, _2, c, _1);  //a、b、c是三个具体的值
        ```
      
      - a、b、c将绑定在f的第1、2、4个参数位置上
      
      - bind参数中的_2与_1指调用g时的参数的位置
      
        - 例如：当调用`g(e, h)`时就等价于调用`f(a, b, h, c, e)`
        - 因为g的形参形式是**g(\_1, _2)**，`bind`将**\_1**绑定在了**f**的第五个参数上，将**\_2**绑定在了**f**的第三个参数上，故调用**g(e, h)**就将这两个参数放在了f的制定位置上
    
  - 作用

    - 可以将参数列表顺序进行调换
      - 例如：将bind作用在一个二元谓词上时，假设f是一个二元谓词，则bind(f, _2, _1)就实现了参数列表的调换

### **lambda表达式**


  - 表达式结构
    - [] () -> returnType {}
      - [] 捕获列表
      - ( ) 参数列表
      - ->指定返回类型
      - { } 函数体
    


  - 使用
    - 捕获main函数中的sz变量，并且应用到lambda中

```c++
int sz = 0;
auto fun = [sz](int a, int b) ->int {return a - b > sz ? a : b; };
cout << fun(20, 3) << endl;
```

- 主要用于泛型算法与异步处理
- 作用
  - 可以捕获包含lambda函数体中的变量，以达到只传递进一个参数就可以满足需求
- 引用捕获
  - []里的变量前面加&即可，一般用于捕获`ostream`类型的
  - 隐式引用捕获就是一个**&**

- 值捕获
  - 默认的捕获方式，如果变量前边没有&那就是值捕获
  - 隐式值捕获就是一个**=**号

- 混合的捕获方式（值捕获与引用捕获混用）
  - `[&, identifier_list]`  identifier_list必须是值捕获
  - `[=, identifier_list]` identifier_list必须是引用捕获

- 可变lambda **P352**
  - 如果要改变一个被捕获的变量的值，则在参数列表首上加上关键字`mutable`
    - `auto f = [v1] () mutable {return ++v1;};`

  - 也可以用引用捕获来改变捕获的值，但是要保证其不指向一个const类型的值-


### **constexpr**

- constexpr变量

  - constexpr定义的变量（无指针）是顶层const的
  - constexpr能够判断表达式是否是常量表达式，会通过报错来提示


  - constexpr指针

    - constexpr定义的指针变量是顶层const的

    - 由于其是顶层const的原因，其定义的指针仅对指针有效，与指针所指的对象无关

      - 所以constexpr指针既可以指向常量，也可以指向一个非常量

        - 指向的常量的地址必须是固定不变的

          - 一般来说函数体内定义的变量并非存放在固定地址中P59

        - ```c++
          int j = 0;
          constexpr int i = 42;
          
          int main(){
              constexpr const int * p = &i;  // p是常量指针 指向整型常量i
              constexpr int *p1 = &j;  // p1是常量指针  指向非常量整数j
          }
          ```


  - 常量表达式

    - 与常量的区别
      - 常量不用在编译过程就计算出来
      - 常量表达式必须在编译过程就计算出来
    - 值不会改变，并且在编译过程就能计算结果的表达式
      - const修饰的不一定是常量表达式
        - `const int a =func( );`
          - 只有到了运行阶段才能够知道值
          - 除非func是constexpr函数
      - 能用constexpr修饰的一定是常量表达式
        - constexpr会检测后面的表达式是否是常量表达式，不是的话会报错


  - constexpr函数

    - 作用
      - 可以用当做常量表达式来赋值给常量
        - `constexpr int a = func( );`
          - 当func( )是constexpr函数时a才是常量表达式
    - 返回值和所有形参列表都必须是字面值类型P59
    - 只有一个return语句

### **explicit**

- 作用
  - 抑制构造函数定义的隐式类型转换
  - 被explicit关键字修饰的构造函数只能用于直接初始化
  
- 何时会发生隐式类类型转换P263
  - 如果构造函数只接受一个实参，则他实际上定义了转换为此类类型的隐式转换机制。这种构造函数称作为转换构造函数，在P514会讲
  
- 例：
  - 智能指针的单一参数的构造函数是explicit的，故不能隐式的将一个内置类型指针转换为一个智能指针
    - `shared_ptr<int> p1 = new int(1024);  //错误`  拷贝初始化赋值运算符右侧的类型必须与赋值运算符左侧的类型相同
      - new int(1024) 返回一个int内置类型的指针，此处发生了从int *到shared_ptr<int>的隐式转换
      - 修改
        - `shared_ptr<int> p1 = shared_ptr<int>(new int(1024));  `
    - `shared_ptr<int> p2(new int(1024));  //正确`

### **智能指针**

- 特性
  - 会自动在合适的时候释放内存
    - shared_ptr对象内部会有一个计数器，用来计算引用该对象的次数，当该计数器为0时会自动销毁。
    - 特别的 unique_ptr内部计数器计数不会超过1个
    - 只声明，未初始化的智能指针默认指向nullptr
  
- 智能指针的统一操作
  
  - 他们的构造函数都被声明为了**explicit**
    - 不能隐式转换成对应的对象类型
  
  - `*p` 解引用得到某对象
  - `p.get()` 返回该类型的普通指针
  - `p->men` 指向内存对象的成员
  - `swap(p,q)` 交换p和q中的指针
  - `p.swap(q)`交换p和q中的指针
  
- 种类
  - shared_ptr
    - 可以有多个shared_ptr同时指向同一个内存
    
    - 专属操作
    
      - `shared_ptr p(u)`  p从unique_ptr u那里接管对象的所有权，并且将u置为空
    
      - `shared_ptr p(q,d)` q是内置指针，p接管了q的所有权，并且p将使用可调用对象d来代替delete
    
      - `make_shared<T>(args)` 返回一个shared_ptr，指向一个动态分配的类型为T的对象
    
        - 如果T是内置类型的话，则会用args对其进行值初始化
        - 如果T不是内置类型的话，则会调用其构造函数，构造函数的参数就是args
    
      - `p.use_count()` 返回引用p的智能指针的个数
    
      - `p.unique()` 返回一个bool值，如果`p.use_count() = 1`则返回true，否则返回false
    
      - ```c++
        p.reset();  //将p置空。若p是唯一指向，则释放p，否则不释放
        p.reset(q);  //将p指向q，若p是唯一指向，则释放p，否则不释放
        p.reset(q, d);  //将p指向q，若p是唯一指向，则释放p，否则不释放 ，释放p的操作用d来代替
        ```
    
  - unique_ptr
    - 只能有一个unique_ptr指向同一个内存
    
    - 专属操作
      - `unique_ptr<T> u(q)`  q可以使new的对象，也可以是unique调用release的返回值。 
      
      - 由于被unique_ptr绑定的对象只能有一个指针指向他，所以没有拷贝与赋值操作
      
        - ```c++
          unique_ptr<string> p1(new string("sdasd"));
          unique_ptr<string> p2(p1);  //错误  不可以拷贝
          unique_ptr<string> p3;
          p3 = p2; // 错误 不可以赋值
          ```
        
      - `u = nullptr` 置空u，且释放u指向的内存
      
      - `u.release()`  置空u，u放弃对指针的控制权，返回指针，不会释放内存空间
      
      - ```c++
        u.reset();  //释放u指向的对象
        u.reset(q);	//q是内置指针，令u指向q，会释放u吗？。。。
        u.reset(nullptr); //令u指向nullptr
        ```
    
  - weak_ptr
    - 指向shared_ptr对象，并且不会增加shared_ptr对象的计数器
    - 专属操作
      - `weak_ptr<T> w(sp)` 指向一个shared_ptr的对象，weak智能指针不能自己初始化weak类型的，只能初始化指向sp类型的
      - `w = p` p可以是sp，也可以是wp
      - `w.reset()` 没有上面两种重载的多，仅仅是置wp为空
      - `w.use_count()` 计算w所绑定的sp对象的引用个数
      - `w.expired()` 译为过期 ,当use_count()=0时返回true，否则返回false
      - `w.lock()` 如果expired为true，则返回一个空shared_ptr，否则返回一个指向w的对象shared_ptr

### **inline 内联函数**

#### **特征**

- 相当于把内联函数里面的内容写在调用内联函数处；
- 相当于不用执行进入函数的步骤，直接执行函数体；
- 相当于宏，却比宏多了类型检查，真正具有函数特性；
- 不能包含循环、递归、switch 等复杂操作；
- 在类声明中定义的函数，除了虚函数的其他函数都会自动隐式地当成内联函数。

#### **使用**

```text
// 声明1（加 inline，建议使用）
inline int functionName(int first, int secend,...);

// 声明2（不加 inline）
int functionName(int first, int secend,...);

// 定义
inline int functionName(int first, int secend,...) {/****/};

// 类内定义，隐式内联
class A {
    int doA() { return 0; }         // 隐式内联
}

// 类外定义，需要显式内联
class A {
    int doA();
}
inline int A::doA() { return 0; }   // 需要显式内联
```

#### **编译器对 inline 函数的处理步骤**

1. 将 inline 函数体复制到 inline 函数调用点处；
2. 为所用 inline 函数中的局部变量分配内存空间；
3. 将 inline 函数的的输入参数和返回值映射到调用方法的局部变量空间中；
4. 如果 inline 函数有多个返回点，将其转变为 inline 函数代码块末尾的分支（使用 GOTO）。

#### **优缺点**

优点

1. 内联函数同宏函数一样将在被调用处进行代码展开，省去了参数压栈、栈帧开辟与回收，结果返回等，从而提高程序运行速度。
2. 内联函数相比宏函数来说，在代码展开时，会做安全检查或自动类型转换（同普通函数），而宏定义则不会。
3. 在类中声明同时定义的成员函数，自动转化为内联函数，因此内联函数可以访问类的成员变量，宏定义则不能。
4. 内联函数在运行时可调试，而宏定义不可以。

缺点

1. 代码膨胀。内联是以代码膨胀（复制）为代价，消除函数调用带来的开销。如果执行函数体内代码的时间，相比于函数调用的开销较大，那么效率的收获会很少。另一方面，每一处内联函数的调用都要复制代码，将使程序的总代码量增大，消耗更多的内存空间。
2. inline 函数无法随着函数库升级而升级。inline函数的改变需要重新编译，不像 non-inline 可以直接链接。
3. 是否内联，程序员不可控。内联函数只是对编译器的建议，是否对函数内联，决定权在于编译器。

### getline相关

#### cin.getline

##### 放在头文件\<iostream>中

- **cin.getline(char\* s, streamsize n, char delim)**

  - 参数说明：
    - 从s中获取n-1个字符，并且设置第n个字符是'\0'，如果入到了delim则会提前结束，且不包括delim字符

- 使用

- ```c++
  #include<iostream>
  using namespace std;
  int main(){
      char str[10];
      cin.getline(str, 7, 'a');  //从键盘上获取6个字符，第七个字符设置为0，且如果在中途遇见了字符'a'，则会提前结束且填上'\0'字符
  }
  ```

- 

#### getline

##### 放在头文件\<string>中

- **getline(istream& is, string& str, char delim)**

- 参数介绍

  - 标准输入
  - string类型的变量
  - 提前结束的标志

- 使用

  - ```c++
    #include<string>
    using namespace std;
    int main(){
        string str;
        getline(cin, str, 'c');  //从cin读取字符到str中，遇见'c'结束
    }
    ```

### 左值引用的细节

- **常量引用：**一般的左值引用只能引用左值，但是常量引用可以引用右值

  - ```c++
    int i1 = 0;
    int &i2 = i1;      // 正确
    int &i3 = i1 + 1;  // 错误 i1 + 1 是常量
    const int &i4 = i1 + 1; //正确 i4是常量引用
    const int &i5 = i1;  // 正确 允许将const int &绑定到一个普通int对象上，这样做的目的是防止i5改变i1。但是i5并不是固定不变的，如果i1变了，则i5会变，也就是说可以通过i1来改变i5
    ```

### move

#### 在头文件\<utility>中

- 个人对std::move函数的理解
  - 是将左值参数刨开漏出他的右值
  - 如下代码所示，我的理解是
    - **rr是左值，&rr是一个地址，里面存放的值是0，move函数将rr的地址赋给了rrr，其实可以类似指针操作，rrr与rr指向了同一个地址空间**
  
- ```c++
  int rr = 0;
  int && rrr = std::move(rr); // 将rr变成右值
  ```

### noexcept关键字

#### 放在参数列表之后

- 例如在移动构造函数中
  - `StrVec::StrVec(StrVec &&s) noexcept : 列表初始化 {函数体}`

- 也可以写成noexcept(表达式)
  - 如果表达式为真则等同于noexcept，反之等同于不声明noexcept
- 一个类设计移动系列函数时，如果可以，最应该为其加上noexcept，以便此类在使用标准库容器时可以用移动操作来代替拷贝。

### 移动构造函数

#### 由于移动操作通常不分配任何资源，故移动操作通常不会抛出任何异常。所以我们应该声明移动构造函数为noexcept的

- `StrVec(StrVec &&) noexcept;` 声明一个移动构造函数
  - 实现从一个StrVec到另一个StrVec的元素
- 参数不使用左值引用
  - 因为如果是左值引用，则实参不能为右值
- 参数不使用常量引用（类似于const int &）
  - 其不能改变，也就是不能将被移动者置空。

### 为什么有些变量必须要初始化

- `int b;`可以不用初始化，只定义，这样就是为b开辟了一个内存空间，&b可以，但是b是一个脏数据，但是他还是可以这样写
- `int &b;` 这样写就是错误的，因为b需要绑定到一个变量上，与那个变量的地址是一样的，所以只定义的话不知道b的地址，只有初始化了才知道b的地址
- `int &&b;`这样写也是错误的，因为b需要绑定到一个右值上去，可以配合move来扒开左值使其变为右值然后赋值给b，右值引用的

### 左右值引用的区别

- 基本上就是他俩的对比

```c++
int rvalue = 0;
//对左值引用
int &a = rvalue;
int &&a = std::move(rvalue);
//对右值引用
const int &a = 0;
int &&a = 0;

const person &a = person(); //性能没有下面的高  下面直接steal
person &&a = person();  
```

#### 实际的例子

- ```c++
  class rvaltest {
  public:
  	rvaltest() = default;
  	rvaltest(int val) {
  		cout << "发生了常数构造" << endl;
  		a = new int(val);
  	}
  	rvaltest(const rvaltest & clas) {
  		cout << "发生了拷贝构造函数" << endl;
  		this->a = new int(*clas.a);
  	}
  	rvaltest& operator=(const rvaltest& clas)
  	{
  		cout << "发生了拷贝赋值" << endl;
  		a = new int(*clas.a);
  		return *this;
  	}
  	rvaltest(rvaltest && clas) {
  		cout << "发生了移动构造函数" << endl;
  		this->a = clas.a;
  		clas.a = nullptr;
  	}
  	rvaltest& operator=(rvaltest&& clas) {
  		cout << "发生了移动赋值" << endl;
  		this->a = clas.a;
  		clas.a = nullptr;
  		return *this;
  	}
  	int *a;
  };
  int main() {
  
  	rvaltest f = rvaltest(1);   //发生常数构造函数
  	rvaltest u(f);  			//发生拷贝构造函数
  	u = rvaltest(0);  			//先发生常数构造函数，再发生移动赋值运算，因为rvaltest(0) 没有名字 是一个右值
  	system("pause");
  	return 0;
  
  }
  ```


### 标准库function类型

- 定义在`<function>`中

- 大概意义

  - 存放**不同类型**的**相同可调用形式**的**可调用对象**

- 直接意义**P511**

  - ```c++
    struct divide{
        int operator()(int a, int b){
            return a/b;
        }
    };
    map<string, int(*)(int,int)> m;
    m.insert({"/", divide()});  // 编译错误， divide()的类型不是对应的函数指针
    // 上述就可以用function来解决
    // 将map定义为map<string, function<int(*)(int, int)>> m;
    m.insert({"/", divide()});
    ```

### 类类型转换运算符P514

- 可以转换成函数指针、数组指针、基本数据类型
- 不能转换成函数和数组

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               

- 要求

  - 必须是成员函数
  - 不能声明返回类型
  - 形参列表必须为空

- 例如

  - ```C++
    class SmallInt{
    public:
        SmallInt(int i = 0): val(i){
            if(i < 0 || i > 255)
                throw std::out_of_range("Bad SmallInt Value");
        }
        operator int() const {return val; }  // 无返回值、无形参、为成员函数  将SmallInt转换为int类型
    private:
        std::size_t val;
    };
    ```

#### 显式的类型转换符

- 将类型转换运算成员函数写为`explicit`的

- `explicit`之后下列操作将不允许

  - ```c++
    SmallInt si = 3;
    si + 3;					   // 错误，此处需要隐式的类型转换，但类的运算符是显示的
    static_cast<int>(si) + 3;  // 正确
    ```

- 向`bool`的类型转换运算符通畅用在条件部分，因此**`operator bool`** 一般定义成**`explicit`**的

#### 类型转换函数的注意事项

- 不要令两个类执行相同的类型转换
  - 如果A有一个接收B类对象的构造函数，那么B类就不应该再定义转义目标是A的类型转换运算符了

- 避免转换目标是内置算术类型的类型转换。特别是已经定义了一个转换成算术类型的类型转换时

- 注意重载函数与转换构造函数产生的二义性

  - ```c++
    struct C{
        C(int);
    	// 其他成员
    };
    struct D{
        D(int);
        // 其他成员
    };
    void manip(const C&);
    void manip(const D&);
    manip(10);  //产生二义性：含义是manip(C(10))还是manip(D(10))
    ```

  - 
