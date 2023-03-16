---
title: The road to modern C++
date: 2020-12-17
categories:
- Programming_Language
tags:
- blogs
---

This blog explains modern C++ features with easy to understand words:

**C++11, C++14 and C++17:**
1. **Rvalue References**:
    - Lvalue: value with actual memory address.
    - Rvalue: the value itself. It exists in the middle of the expressions, or in function return etc. Rvalue disappear right after the instruction. Rvalue cannot be bind to actual reference type before. For example, ***int& a = 1+2;*** will not works. After C++11, you can use && for Rvalue reference, it directly bind the Rvalue to the reference type, without copying instructions. 
    - Pure Rvalue: numbers, actual char values such as 'a', Lambda expressions.
    - Rvalue reference usage: std::move(), std::unique_ptr() etc. To reduce copying overhead for moving the variables.
    - Addition: 
        - ***for (auto b:a) {}***: value type iteration.
        - ***for (auto& b:a) {}*** reference type iteration.
        - ***for (auto&& b:a) {}*** exactly the same as the previous reference type one.

2. **lambda expression**: 
    - Usage: you can define a function that is place in another function call. This is an **inline function**.
    - Syntax: 
        - In the capture area, [=] means only readable, [&] means w&r. Only use [&] could inherit all the reference from the parent namespace.
    
    ``` 
    [capture](parameters)->return-type {body}  //Function pointer
    [capture](parameters)->return-type {body}()  // Function call.
    ```

    - Example code:

    ```C++
    // Count the uppercase number from char[] s.
    int uppercase = 0;
    for_each(s, s+sizeof(s), [&uppercase](char c){
        if (isupper(c)) uppercase++;
    });  // The end of for_each statement.
    ```
    
    or 
    
    ```C++
    auto fun = [](){ std::cout << "Hello Lambda" << std::endl; };
    ```
    
1. **auto** and **decltype**
    - automatic type deduction. 
    
    ```C++
    auto a = 'a'; //char
    auto b = 3; //int
    auto c = 0.01; //float
    ```
    
    - **decltype**: capturing the type of an object or an expression, for example:
    
    ```C++
    const vector<int> vi;
    typedef decltype (vi.begin()) CIT;
    CIT another_const_iterator;
    ```
    
    or
    
    ```C++
    struct A { double x; };
    const A* a;
 
    decltype(a->x) y;       // type of y is double (declared type)
    decltype((a->x)) z = y; // type of z is const double& (lvalue expression)
    //OR
    template<typename T, typename U>
    auto add(T t, U u) -> decltype(t + u) // return type depends on template parameters
                                          // return type can be deduced since C++14
    {
        return t + u;
    }
    ```
    
1. **Deleted and Defaulted functions**
    - Usage: Default: Specify default implementation of the function. (use in class) 
    - Usage: Deleted: delete automatic generated function implementation. For example, delete class copy function:

    ```C++
       struct NoCopy
       {
           NoCopy & operator =( const NoCopy & ) = delete;
           NoCopy ( const NoCopy & ) = delete;
       };
       NoCopy a;
       NoCopy b(a); //compilation error, copy ctor is deleted 
    ```
    
1. **nullptr**
    - Usage: to representing **NULL**. Use for null pointer.

1. Difference between **const** and **constexpr**
    - const: read-only.
    - constexpr: ***actually constant value***, should be able to determine its value at compile time, and not change at all at runtime!
    - Errors that could be caused by using const as actual constant:

        ```C++
            constexpr int FivePlus(int x) { return x+5;}
            void g() {
                const int x = 5;
                C<x> c1; // OK!!!
                *(int *)(&x) = 6; // Still OK! 
                C<x> c2; // Still OK! But c2 is C<5>, not C<6>!
                C<FivePlus(x)> c3; // Still OK! c3 is C<10>!

                printf("%d\n", x); // can be either 5 or 6!!!
                const int* p = &x;
                printf("%d\n", *p); // can be either 5 or 6!!!
            }
        ```
    - constexpr function: function that can be used in constant expression, for example, used for declaring the size of a variable. (as shown above) The results of the return value should be able to determined at compile time.


     