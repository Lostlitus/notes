# Type deduction

In C++ we have `auto` and `decltype`. For `auto`, it works similar to template
deduction. Here we introduce `auto` and `decltype`.

## `auto`

`auto` only focuses on type of expression. This means value category is
ignored, so no reference type. And it omits `const`, so no const type. And
C-style array is considered as pointer, so no array type.

```C++
// Gives `int`
auto v1 = 10;

// Gives `int`, no reference
int &rV = v1;
auto v2 = rV;

// Gives `int &` by adding & manually
auto &v3 = rV;

// Gives `int`, no const
const int cV = 20;
auto v4 = cV;

// Gives `const int` by adding const manually
const auto v5 = cV;

// Well, for the `const` in pointer to const, `auto` keeps it
// Gives `const char *`
auto v6 = "123";

// Gives `int*`, no array type
int array[10];
auto v6 = array;
```

`auto` is designed to be intuitive when user simply want to initialize an
object without writing type manually.

## `decltype`

`decltype` focuses on both type and value category of the expression, so with
reference type. And `const` is considered, so with const type. And C-style
array is not considered as pointer, so with array type. Check document
[here](https://en.cppreference.com/w/cpp/language/decltype) for detail.

One thing here is decltype has two syntax, for entity or for expression.  Even
though entity is included in expression. It is designed in this way because its
type deduction relies on the value category.

- xvalue yields rvalue reference `T&&`
- lvalue yields lvalue reference `T&`
- prvalue yields non-reference type `T`

The name of an entity (here we means variable) as an expression has a lvalue
category, so decltype should gives `T&` if there is no distinction. But it's
not intuitive.

```C++
// Gives `int`
int v1 = 10;
decltype(v1) v2 = v1;
```

Image the decltype here gives `int&`. Then programer would say "fxxk you" to
C++. As a result, entity is special in decltype's case. So the same type of
the entity would be yielded by decltype.

However, if you really want the entity to be interpreted as lvalue in decltype,
you can add parenthesis for it. Now it is lvalue.

```C++
// Gives `int&`
int v1 = 10;
decltype((v1)) v2 = v1;
```

Other cases here, there is nothing special.

```C++
int f1();
int& f2();
int&& f3();

// Gives `int`, since f1() has prvalue category
decltype(f1()) v1 = f1();

// Gives `int&`, since f2() has lvalue category
decltype(f2()) v2 = f2();

// Gives `int&&`, since f3() has xvalue category
decltype(f3()) v3 = f3();

// Gives `const int`, `const` is not omitted
const int cv = 10;
decltype(cv) v4 = cv;

// Gives `int[10]`, array type is not considered as pointer
int array[10];
decltype(array) v5;
```

Compared to auto, decltype gives the exact type. Precise. But if you simply
want to initialize a variable, auto is always better.

## `decltype(auto)`

Crazy at first glance. But reasonable. It means applying decltype rule to auto,
so works identically to decltype. The benefit is programer now need not to
write the expression twice when initialization.

```C++
int func();

decltype(func()) v1 = func();   // Write expression `func()` two times
decltype(auto) v2 = func();     // Write expression `func()` only once
```
