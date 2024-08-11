# Template

Template is one of the two types of `Compile-Time Polymorphism`(another one is
function overloading) in C++. Write template demo code is easy, but using it in
large project needs extra attention. The following is some FAQ.

## 1. Template instantiation vs Template specialization

A template has its defination, and one instantiates it by providing types. A
template can be instantiated either explicitly or implicitly.

``` C++
template <typename T>
struct S {
    T m;
};

// Implicitly
int main () {
    S<int> implicitS;
}

// Explicitly
template S<double> explicitS;
```

Explicit instantiation helps when only limited types are permitted to be use.

``` C++
// template_demo.h

template <typename T>
class C {
public:
    void DoSomething();
};


// template_demo.cpp
#include "template_demo.h"

template <typename T>
void C<T>::DoSomething() {}

template C<int>;


// main.cpp
#include "template_demo.h"

int main () {
    C<int> c1;          // Valid
    C<double> c2;       // Link error.
}
```

The code above works beacause C++ needs template defination to instantiate
template. So if only provided with declaration, C++ will not instantiate it and
try to finds instantiated defination directly in the link stage. This means
only the templates instantiated explicitly can be used, or compiler reports
with a link error.

Template specialization helps template to handle special cases. By default, the
template defination is same. But one can write some template definations for
some special types given, and C++ uses these template definations when the
template is instantiated by the corresponding types.

``` C++
template <typename T>
struct S {
    T m;
};

template <>
struct S<std::string> {
    char m[100];
};

int main () {
    S<int> defaultS;                // Use default defination.
    S<std::string> specializedS;    // Use special defination.
}
```

Some templates accepts more than one type. And `Partial specialization` is
available by providing part of the types.

It's worth note that a template instantiation creates a normal class/function
defination, while a template specialization creates a template defination,
which means instantiation is still needed.
