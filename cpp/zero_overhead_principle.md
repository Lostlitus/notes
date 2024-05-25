# Zero-overhead principle

The [defination](https://en.cppreference.com/w/cpp/language/Zero-overhead_principle)
on cppreference introduces zero-overhead principle in concise words. That is:

> 1. You don't pay for what you don't use.
> 2. What you do use is just as efficient as what you could reasonably write by
> hand.

This provides C++ with a great freedom to add new features, and keeps its
backward-compatibility in the meantime. Programer can choose to ignores the
feature, since *you don't pay for it*.  Or to use it, which is promised by C++
to be *as efficient as what you could reasonably write by hand*.

However, this principle is violated in some parts of C++. The main parts are
RTTI and exceptions.

## 1. Run-Time Type Information(RTTI)

RTTI provided by C++ allows the type of an object to be determined during
program execution. Basicly, it lets programer use `typeid` and `dynamic_cast`.
This leads to a compile time cost like larger executable, longer loading time
and so on, since extra works are needed to track objects' types. While runtime
cost is almost non-existent. So this actually only matters for those programs
that run in resource-deprived environments or work in frequently open/close
workflow.

Compiler provides an option to disable RTTI.(in g++ it is `-fno-rtti`) In most
cases, RTTI can be replaced by virtual functions and dynamic dispatching.

## 2. Exceptions

Using exceptions in C++ means extra logic for exception-handling. And the
problem is that C++ always add these logic for the program for safty reason. If
a programer uses other error-handling implementation(C-style for example), then
he/she just pays for what he/she doesn't use.

Like RTTI, compiler can disable exceptions.(in g++ it is `-fno-exceptions`) And
then all `throw` action becomes `abort`. Nowadays, C++ provides language
support(i.e. `noexcept` keyword) for optimizing out error-handling logic as
well.

## 3. Design compromises

Other than the two main parts talked above, there are some smaller issues that
impose some overhead. The reason why they introduce overhead differs, but big
picture is to balance. For example, one design may have 100 abnormal cases, and
only one of these needs extra logic. So what C++ does now may be adding the
extra logic for all these 100 cases. And we can of course say that the design
violates the zero-overhead principle. It depends on how you explain. Since one
may never meet that one case, but he/she pays for it now. That all about design
compromises.
