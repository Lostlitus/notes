# Pass value

Intuitively speaking, a variable is passed by value. And understanding what the
program is doing is quite easy with this mechanism. But C++ need performance.
Passing by value can leads to tons of constructions and destructions without
further optimization, which obviously can not meet the expection of the
language designers.

Here are three concepts C++ introduced so as to optimize the performance when it
comes to passing value.

1. Pass by lvalue reference
2. Pass by rvalue reference
3. Copy elision

## 0. Understand value category first

The remaining part of this article is mixed with tons of discussion about value
category. You should make sure you've fully understand what exactly value
category is, or you'll be messed up. To figure out what is value category, read
the note `cpp/expression` first.

The value categories talked below are under the context that they are used in
expressions.(since only expression has value categories)

## 1. Pass by lvalue reference

### A lvalue reference is not totally an alias

A lvalue reference has its own name, and C++ try to make it works like the
alias of the referenced variable. It's ok, but we should not consider them as
totally the same. For example.

```C++
int obj = 1;            // decltype(obj) --> int
int &objRef = obj;      // decltype(obj) --> int&
```

In the cases like the example above, `obj` and its reference `objRef` are not
identical.  And there are much more differences between when it comes to const
lvalue reference.

### Const lvalue reference can refer to a temporary

Other than non-const lvalue reference, a const qualified lvalue reference can
be initialized by a prvalue expression. In this case, a temporary object will
be created and the const lvalue reference then points to it. Normally, a
temporary object's lifetime ends when the expression that creates it ends, but
if it's bound to a const lvalue reference or a rvalue reference(discussed
below), its lifetime is extended to be the same as the reference's.

```C++
int GetInt();

int main() {
    // Expression 2 is prvalue and the created temporary's lifetime is extended.
    const int &tmpRef1 = 2;

    // Expression GetInt() is prvalue and the created temporary's lifetime is extended.
    const int &tmpRef2 = GetInt();

    // Do something
}
```

This mechanism is called *Lifetime extension*. It's worth noting that the
concept temporary object is not the same concept as prvalue. A temporary object
is a unnamed object, while a prvalue is an expression. A reference cannot bind
to an expression, it is the temporary object created after that bound to the
reference.

This mechanism can be optimized. A prvalue expression may has a temporary
object result.  In this case, the const lvalue reference refers to it directly.

## 2. Pass by rvalue reference

### Looks much less like an alias

If one is told lvalue reference is the alias of the referenced object, without
a doubt he/she will believe rvalue reference is the alias of the referenced
object as well.  However, like we said above, a lvalue reference is not totally
an alias. A rvalue reference is not an alias. Actually it even looks much less
like an alias. Or radically speaking, forget the alias words when it comes to
rvalue reference.

A rvalue reference accepts the result of a xvalue expression or a prvalue
expression. The former one is quite easy to understand. And the latter one
works the same as const lvalue reference, including constructing a new
temporary object and the optimization stuff.

## 3. Copy elision

### An optimization with side effects

Copy elision is the first allowed form of optimization that leads to observable
side effects. What it does is constructing the object directly into the storage
where it would otherwise be copied/moved to. This optimization can be chained,
which means a continuous process of constructions and destructions can be
optimized to only one pair of construction and destruction.

The standard specifies if copy elision is guaranteed in the given scenario.  If
guaranteed, the compiler is forced to apply copy elision, and if not, it leaves
the decision to the compiler.  One should write copy elision friendly code to
trigger this optimization.

### Copy elision guaranteed

*Prvalue semantics* guarantees copy elision since C++17. Just as what copy
elision means, a prvalue is not materialized until needed, and then it is
constructed directly into the storage of its final destination. This variant of
copy elision is known as *URVO(unnamed return value optimization)*.

Example:

```c++
T f();

T x = T(T(f())); // x is initialized by the result of f() directly.
```

### Copy elision not guaranteed

Copy elision is not guaranteed when the variable has name/identity, but
compiler is allowed to omit the copy.  This variant of copy elision is known as
*NRVO(named return value optimization)*.

Example:

```c++
T g()
{
    T res;
    return res; // constructs the returned T directly.
}
```
