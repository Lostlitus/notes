# Template

Template is one of the two types of `compile-time polymorphism`(another one is
function overloading) in C++. Write template demo code is easy, but using it in
large project needs extra attention.

## Template instantiation

A template has its definition, and one instantiates it by providing types, the
instantiated code will be generated in compiler time. A template can be
instantiated either explicitly or implicitly.

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

### Why explicit instantiation

Explicit instantiation helps when limited types are permitted.

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

The code above works beacause C++ needs template definition to instantiate
template. If only provided with declaration, C++ does not instantiate it in
compile time and try to find instantiated definition directly in the link time.
This means only the template that has been instantiated explicitly in other
translation unit helps, or compiler reports with a link error.

One thing to note here is that modern C++ linker does not instantiate template
in link time, it's the job of compiler in compile time. So even an object file
gets its template definition in link time from others, linker does not
instantiate for it.

Another benefit of explicit instantiation is that it avoids code bloat.
Compiler generates instantiated template code per translation unit. If one
template is used among translation units, it may be instantiated by some set of
types multiple times.  Although linker will discard the redundant codes in link time,
it waste our compile time disk and CPU.  However, if we arrange the code
structure like the example above, in which the compiler generates the code of
`C<int>` only in the translation unit of `template_demo.cpp`. We can limit the
instantiation number to one.

### How to use explicit instantiation

There are two common ways to limit permitted type and restrict instantiation
with explicit instantiation. The first has been shown above. What it does is
declaring template in header file, hiding the definition in source file and
explicitly instantiating needed type in source file.

The second way is keeping definition in header file, giving `extern` explicit
instantiation in header file and explicitly instantiating in source file. Here
is a similar example.

``` C++
// template_demo.h
template <typename T>
class C {
public:
    void DoSomething() {}
};

extern template C<int>;


// template_demo.cpp
#include "template_demo.h"

template C<int>;


// main.cpp
#include "template_demo.h"

int main () {
    C<int> c1;          // Valid
    C<double> c2;       // Link error.
}
```

What `extern` does is suppressing template instantiation of file including it.
Since the `extern` statement only declares the instantiated template with given
type, so the compiler does not generate code for the translation unit including
it.  This way, instantiation is controlled in the scoop of file truly defining
it. That is, template source file.  And instantiation in other types not
defined in source file will lead to link error in link time, just like what the
first approach introduced above does.

## Template specialization

Template specialization helps template to handle special cases. By default, the
template definition is same. But one can write a template definitions for a
special types given, and C++ uses this template definition when the template is
instantiated by the corresponding types.

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
    S<int> defaultS;                // Use default definition.
    S<std::string> specializedS;    // Use special definition.
}
```

Some templates accept more than one type. And `partial specialization` is
available by specializing with part of the types provided.
