# Expression Value categories

Expression is special in C++. An expression is characterized by a type and a
value category. Both of these two properties are a little bit different from
people may think.

## The type of an expression is non-reference type

An object in C++ can have reference type like lvalue reference. But if it is
used in an expression, the result will never be a reference type. The simplest
example is using an object itself only in an expression:

```c++
int v = 100;
int &ref = v;
std::cout << ref << std::endl;  // Expression ref has a result with type int
```

The `ref` object here is declared as a `int&` to a `int v`. But when it is
used as an expression, the result of the expression is typed as `int` rather
than `int&`.

## Value categories are for expression only

Only expression has value category, which consist of lvalue, xvalue, prvalue,
glvalue and rvalue. Their relations can be clarified in a tree view.

```txt
lvalue    xvalue    prvalue
  ^        ^  ^       ^
  |        |  |       |
  |        |  |       |
  +--------+  +-------+
    glvalue     rvalue
```

Details about which value category corresponds to which case of the expression
can be checked [here](https://en.cppreference.com/w/cpp/language/value_category).
And the original proposals can be read [here](https://stackoverflow.com/a/38169963/13692802).

*Expression only* means an object itself **never** has any value category. We
should not confuse expression and object. The reason why you can find imprecise
statements like `Object xx is a lvalue` is because an object itself can be used
as an expression, and an expression with only a named object is a lvalue
expression.
