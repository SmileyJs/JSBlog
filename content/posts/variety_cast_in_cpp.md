+++
title = 'Variety_cast_in_cpp'
date = 2023-09-13T19:07:29+08:00
draft = false
+++

在cpp语言中一共有4中cast方式,分别是static_cast, dynamic_cast, const_cast和reinterpret_cast. 发现自己对这些没有完全理解, 下面具体单独分析:

### static_cast

编译时进行类型检查. 通常执行隐式的类型转换或者调用显式的类型转换方法

#### 执行隐式的类型转换
```cpp
// 1) implicit conversion
float f = 3.14;
int a = static_cast<int>(f);
```

#### 调用显式的类型转换方法
```cpp
// 2) explicit call convert method
class Integer {
	Integer(int x)
		: data_(x)
	{
		std::cout << "Ctor of Integer" << std::endl;
	}

	operator string()
	{
		std::cout << "Conversion from Integer to string" << std::endl;
		return to_string(data_);
	}

private:
	int data_;
};

int main()
{
	Integer a(10);
	std::string str = static_cast<string>(a);
	
	std::cout << "str is " << str << std::endl;

	return 0;
}
```

#### 在继承体系中的转换
通常static_cast适用于向上转换的，否则其结果是未定义。但是如果引用或者指针指向的对象本身确实是目标类型，那么这个转换也是可行的。

```cpp
// 3) cast in the inheritance
struct Base
{
	int b;
};

struct Derived : public Base
{
	int d;
};

int main()
{
	Derived d;
	d.b = 1;
	d.d = 2;

	Base* b1 = static_cast<Base*>(&d);
	std::cout << "b1.b " << b1->b << std::endl;

	Base b2;
	Derived* d1 = static_cast<Derived*>(&b2);
	std::cout << "d1.d " << d1->d << std::endl; // undefined behavior

	Derived* d3 = static_cast<Derived*>(b1);
	std::cout << "d3.d " << d3->d << std::endl; 

	return 0;
}
```
#### 实现move的功能
将左值转换为右值，转换后原表达式失效
```cpp
std::vector<int> v0{1,2,3};
std::vector<int> v2 = static_cast<std::vector<int>&&>(v0);
std::cout << "2) after move, v0.size() = " << v0.size() << '\n';
```
### dynamic_cast
多用于在多态继承体系中向下转换。

- dynamic_cast在运行时要根据RTTI进行检查动态类型检查，有运行时的开销
- 如果类型转换确认是安全的，应该使用static_cast

#### 继承体系中的指针转换
```cpp
class Base
{
public:
    virtual void hello()
    {
        std::cout << "hello from base" << std::endl;
    }

	int b;
};

class Derived : public Base
{
public:
    void hello() override
    {
        std::cout << "hello from derived1" << std::endl;
    }

	int d;
};

class Derived2 : public Base
{
public:
    void hello() override
    {
        std::cout << "hello from derived2" << std::endl;
    }
};

int main()
{
    Derived d1;

    Base* b = static_cast<Base*>(&d1);

    Derived* d2 = dynamic_cast<Derived*>(b);
    if (d2) {
        std::cout << "d2 is not nullptr" << std::endl;
    } else {
        std::cout << "d2 is nullptr" << std::endl;
    }

    Derived2* d3 = dynamic_cast<Derived2*>(b);
    if (d3) {
        std::cout << "d3 is not nullptr" << std::endl;
    } else {
        std::cout << "d3 is nullptr" << std::endl;
    }

		return 0;
}
```

#### 继承体系中的引用转换
```cpp
class Base
{
public:
    virtual void hello()
    {
        std::cout << "hello from base" << std::endl;
    }

	int b;
};

class Derived : public Base
{
public:
    void hello() override
    {
        std::cout << "hello from derived1" << std::endl;
    }

	int d;
};

class Derived2 : public Base
{
public:
    void hello() override
    {
        std::cout << "hello from derived2" << std::endl;
    }
};

int main()
{
    Derived d1;
    Base& b2 = d1;

    try {
        Derived& d2 = dynamic_cast<Derived&>(b2);
    } catch (std::exception& e) {
        std::cout << "convert to derived failed! " << e.what() << std::endl;
    }
    
		try {
        //  convert failed. std::bad_cast
        Derived& d2 = dynamic_cast<Derived&>(b3);
    } catch (std::exception& e) {
        std::cout << "convert to derived failed! " << e.what() << std::endl;
    }

    try {
				// Compile: error: invalid initialization of reference of type ‘Derived&’ from expression of type ‘Derived2’
        Derived& d2 = dynamic_cast<Derived2&>(b2);
    } catch (std::exception& e) {
        std::cout << "convert to derived2 failed! " << e.what() << std::endl;
    }

    return 0;
}
```

### const_cast
用于消除const引用或者指针指向对象的const修饰符.

尽管如此, 如果指针或者引用指向的对象本身确实是const对象,那对他的修改的行为是未定义的

#### 用于在非const对象的const成员函数中修改非const成员变量
```cpp
class Student
{
public:
    Student(int id)
        : id_(id)
    {
    }

    void fun() const
    {
        const_cast<Student*>(this)->id_ = 20;
    }

    int getId()
    {
        return id_;
    }

private:
    int id_;
};

int main()
{
    Student* s1 = new Student(10);

    std::cout << "id: " << s1->getId() << std::endl;

    s1->fun();

    std::cout << "id: " << s1->getId() << std::endl;

    return 0;
}
```

#### 将const的指针或者引用用于接收非const参数的函数
```cpp
void update(int* a)
{
    *a = 100;
}

int main()
{
		int a = 10;
    const int* p = &a;

    update(const_cast<int*>(p));
    std::cout << "a: " << a << std::endl;

    const int b = 20;
    const int* p2 = &b;

    update(const_cast<int*>(p2));   // undefined behavior
    std::cout << "b: " << b << std::endl;
}
```

#### 消除对象的volatile属性
```cpp
int a = 10;

const volatile int* b = &a;
std::cout << "type id of b: " << typeid(b).name() << std::endl;
int* c = const_cast<int*>(b);
std::cout << "type if of c: " << typeid(c).name() << std::endl;
```

### reinterpret_cast

编译时不会出错，可以将指针和引用在任意类型间互相转换，但除非将其转回为最初的真是类型， 否则行为是未定义的

```cpp
int f()
{
    return 77;
}

int main()
{
    int a = 9;
    std::uintptr_t p1 = reinterpret_cast<std::uintptr_t>(&a);
    std::cout << "Value of &i is " << std::showbase << std::hex << p1 << std::endl;
    int* p2 = reinterpret_cast<int*>(p1);
    std::cout << "Value of p2 is " << std::dec << *p2 << std::endl;
 
    void(*fp1)() = reinterpret_cast<void(*)()>(f);
    fp1();
    int(*fp2)() = reinterpret_cast<int(*)()>(fp1);
    std::cout << "Value of f() is " << fp2() << std::endl;

    char* p3 = reinterpret_cast<char*>(&a);
    std::cout << (p3[0] == '0x9' ? "This system is little-endian" :
        "This system is big-endian") << std::endl;

    reinterpret_cast<unsigned int&>(a) = 42;
    std::cout << "a: " << a << std::endl;

    return 0;
}
```