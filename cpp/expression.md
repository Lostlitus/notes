# Expression Value categories

Expression is special in C++. An expression is characterized by a type and a
value category. Both of these two properties are a little bit different from
people may think.

## The type of an expression is non-reference type

A variable in C++ can have reference type like lvalue reference. But if it is
used in an expression, the result will never be a reference type. The simplest
example is using a variable itself only in an expression:

```c++
int v = 100;                    // Variable v is a lvalue with type int
int &ref = v;                   // Variable ref is a lvalue reference with type int&
std::cout << ref << std::endl;  // Expression ref has a result with type int
```

The `ref` variable here is declared as a `int&` to a `int v`. But when it is
used as an expression, the result of the expression is typed as `int` rather
than `int&`.

## Value categories are for expression only

Only expression has value categories, which consists of lvalue, xvalue, prvalue,
glvalue and rvalue. Their relation can be clarified in a tree view:

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

*Expression only* means variable itself **never** has any value category. We
should not confuse expression and variable . The reason why you can find
imprecise statements like `Object xx is a lvalue` is because an variable
itself can be used as an expression, and an expression with only a named
variable is a lvalue expression.
