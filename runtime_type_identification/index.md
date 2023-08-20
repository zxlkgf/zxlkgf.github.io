# Runtime_type_identification

# 类型转换运算符

---

## 1. daynamic_cast
dynamic_cast不能回答指针指向的是哪种类型，但是可以回答是否可以**安全的**将对象的地址赋值给特定类型的指针。  
dynamic_cast用于类继承层次间的指针或引用转换.如果是单纯的派生类指针或引用转换为基类的指针或引用，这本是就是安全的。使用dynamic_cast略显多余。  
如下:  
```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>

using std::cout;

class Grand
{
private:
    int hold;
public:
    Grand(int h = 0):hold(h){}
    virtual void Speak()const{cout << "I am a grand class!\n";}
    virtual int Value() const{return hold;}
};

class Superb : public Grand
{
public:
    Superb(int h = 0):Grand(h){}
    void Speak()const{cout << "I am a superb class!!\n";}
    virtual void Say() const{cout << "I hold the superb value of " << Value() << "!\n";}
};

class Magnificent : public Superb
{
private:
    char ch;
public:
    Magnificent(int h = 0, char c = 'A'):Superb(h),ch(c){}
    void Speak() const { cout << "I am a Magnificent class !!!\n";}
    void Say() const { cout << "I hold the character " << ch << " and the integer " << Value() << "\n";}
};

int main()
{
    Grand * pg;
    Magnificent * ps = new Magnificent();     
    pg = dynamic_cast<Grand *>(ps);     // 正确，但是没有必要，向上转型总是安全的，而且dynamic_cast有开销
    pg->Speak();
    delete ps;
    
    return 0;
}
```

---

dynamic_cast 真正的作用是进行安全的向下转型。  
1. 基类指针指向派生类，将该指针转化为派生类的指针。成功返回派生类指针。  
2. 基类指针指向基类，将该指针转化为派生类指针，失败返回0(NULL)  
如下:  
```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>

using std::cout;
using std::endl;

class Grand
{
private:
    int hold;
public:
    Grand(int h = 0):hold(h){}
    virtual void Speak()const{cout << "I am a grand class!\n";}
    virtual int Value() const{return hold;}
};

class Superb : public Grand
{
public:
    Superb(int h = 0):Grand(h){}
    void Speak()const{cout << "I am a superb class!!\n";}
    virtual void Say() const{cout << "I hold the superb value of " << Value() << "!\n";}
};

class Magnificent : public Superb
{
private:
    char ch;
public:
    Magnificent(int h = 0, char c = 'A'):Superb(h),ch(c){}
    void Speak() const { cout << "I am a Magnificent class !!!\n";}
    void Say() const { cout << "I hold the character " << ch << " and the integer " << Value() << "\n";}
};

int main()
{
    Grand * ps[3];
    ps[0] = new Magnificent();
    ps[1] = new Superb();
    ps[2] = new Grand();

    for(int i = 0; i < 3; i++)
    {
        Superb * pb;
        if(pb = dynamic_cast<Superb*>(ps[i]))
        {
            pb->Say();
        }
        else
        {
            cout << "错误的转换" << endl;
        }
        delete ps[i];
    }
    return 0;
}
```
补充: 使用引用进行dynamic_cast转换，正确的转换并不会发生任何事，但是，错误的转换不会返回0,而是抛出bad_cast异常。

---


## 2.const_cast
const_cast用于改变值为const或Volatile.  
用法: const_cast<type_name>(expression)  
返回值为新类型。这里我们需要强调的是**const_cast**主要用于更改指针或引用的**const**或**volatile**限定符。其中，**type_name**必须是指针、引用或者成员指针类型.
如下:  
```cpp
#include <iostream>
using namespace std;

void change(const int * pt, int n);

int main()
{
    int pop1 = 38383;
    const int pop2 = 2000;

    cout << "pop1: " << pop1 << ",address: "<< &pop1 << endl;
    cout << "pop2: " << pop2 << ",address: "<< &pop2 << endl;
    cout << endl;
    change(&pop1, -103);
    change(&pop2, -103);
    cout << endl;
    cout << "pop1: " << pop1 << ",address: "<< &pop1 << endl;
    cout << "pop2: " << pop2 << ",address: "<< &pop2 << endl;
    return 0;
}

void change(const int * pt,int n)
{
    int * pc;
    pc = const_cast<int *>(pt);
    *pc += n;
    cout << "ps: " << *pt << ",address: " << pt << endl;
}
```
```text
result:
pop1: 38383,address: 0x7ffda82a80e0
pop2: 2000,address: 0x7ffda82a80e4

ps: 38280,address: 0x7ffda82a80e0
ps: 1897,address: 0x7ffda82a80e4

pop1: 38280,address: 0x7ffda82a80e0
pop2: 2000,address: 0x7ffda82a80e4
```
从运行结果可以看出，pop1和ps的地址相同，pop2和ps的地址相同，在函数内部,该地址所对应的值已经被-103，但是通过函数之后最终pop2却没有修改，为什么会出现这种结果呢?  
实际上这就是因为编译器优化结果造成的，因为在声明pop2的时候，其类型是const int，在编译阶段，编译器认为它就是不变的类型，当编译到cout << "pop2: " << pop2 << ",address: "<< &pop2 << endl;时，会将pop2直接替换为常量2000，即cout << "pop2: " << 2000 << ",address: "<< &pop2 << endl;，因此打印出来的就过就是2000。也正是由于该行为是未定义的行为，才导致输出结果与我们的预期不一致。所以，在我们日常使用中，const_cast可以用用来修改最初声明非const的值，而且应该尽量避免常量转换，除非我们真的需要使用它。

---
const_cast另一个作用  
将volatile丢弃，例子如下:
```cpp
#include <iostream>
#include <typeinfo>

int main()
{
    using namespace std;
    int a = 100;
    volatile int *atemp = &a;
    cout << typeid(a).name() << endl;
    cout << typeid(atemp).name() << endl;

    cout << "将Volatile int *转换为 int * ..." << endl;
    int * aptr = const_cast<int *>(atemp);
    cout << typeid(aptr).name() << endl;

    return 0;
}
```

---

## 3.static_cast
static_cast关键字一般用来将枚举类型转换成整型，或者短整形转换成长整形，又或者整型转换成浮点型。也可以用来将指向父类的指针转换成指向子类的指针。  
static_cast使用注意事项：

1. static_cast可以用于基本类型的转换，如short与int、int与float、enum与int之间；

2. static_cast也可以用于类类型的转换，但目标类型必须含有相应的构造函数；

3. static_cast还可以转换对象的指针类型，但它不进行运行时类型检查，所以是不安全的；

4. static_cast甚至可以把任何表达式都转换成void类型；

5. satic_cast不能移除变量的const属性，请参考const_cast操作符；

6. static_cast进行的是简单粗暴的转换（仅仅依靠尖括号中的类型），所以其正确性完全由程序员自己保证。

7. static_cast是在编译时进行的，这与dynamic_cast正好相反。



---

## 4.reinterpret_cast
首先从英文字面的意思理解，interpret是“解释，诠释”的意思，加上前缀“re”，就是“重新诠释”的意思；cast在这里可以翻译成“转型”（在侯捷大大翻译的《深度探索C++对象模型》、《Effective C++（第三版）》中，cast都被翻译成了转型），这样整个词顺下来就是“重新诠释的转型”。我们知道变量在内存中是以“…0101…”二进制格式存储的，一个int型变量一般占用32个位，参考下面的代码  

```cpp
#include <iostream>
using namespace std;
int main(int argc, char** argv)
{
	int num = 0x00636261;//用16进制表示32位int，0x61是字符'a'的ASCII码
	int * pnum = &num;
	char * pstr = reinterpret_cast<char *>(pnum);
	cout<<"pnum指针的值: "<<pnum<<endl;
	cout<<"pstr指针的值: "<<static_cast<void *>(pstr)<<endl;//直接输出pstr会输出其指向的字符串，这里的类型转换是为了保证输出pstr的值
	cout<<"pnum指向的内容: "<<hex<<*pnum<<endl;
	cout<<"pstr指向的内容: "<<pstr<<endl;
	return 0;
}
```

```text
result:
pnum指针的值: 0x7ffd7246df64
pstr指针的值: 0x7ffd7246df64
pnum指向的内容: 636261
pstr指向的内容: abc
```
第6行定义了一个整型变量num，并初始化为0x00636261（十六进制表示），然后取num的地址用来初始化整型指针变量pnum。接着到了关键的地方，使用reinterpret_cast运算符把pnum从int*转变成char*类型并用于初始化pstr。

将pnum和pstr两个指针的值输出，对比发现，两个指针的值是完全相同的，这是因为“reinterpret_cast 运算符并不会改变括号中运算对象的值，而是对该对象从位模式上进行重新解释”。如何理解位模式上的重新解释呢？通过推敲代码11行和12行的输出内容，就可见一斑。

很显然，按照十六进制输出pnum指向的内容，得到636261；但是输出pstr指向的内容，为什么会得到”abc”呢？
可以参考char占一个byte,但是由于int4byte内没有定义'\0'所以一直输出，直到遇到0。
