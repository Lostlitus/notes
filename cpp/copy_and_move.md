# Copy, Move and their optimizations in C++

## Value catagories

Each C++ expression (an operator with its operands, a literal, a variable name,
etc.) is characterized by two independent properties: a type and a value
category. Each expression has some non-reference type, and each expression
belongs to exactly one of the three primary value categories: prvalue, xvalue,
and lvalue.

## Move semantic

TODO

## Copy elision

[Copy elision](https://en.cppreference.com/w/cpp/language/copy_elision) is the
first allowed form of optimization that can change the observable
side-effects.(since C++14, *allocation elision and extension* is added, which
is the second one)

When copy elision is performed, C++ constructs the object directly into the
storage where they would otherwise be copied/moved to. This optimization can be
chained, which means a continuous process of constructions and destructions can
be optimized to only one pair of construction and destruction.

The standard specifies if copy elision is guaranteed in the given scenario. If
guaranteed, the compiler is forced to apply copy elision, and if not, it leaves
the decision to the compiler. Here we explain both of them.

### Copy elision guaranteed(URVO)

*Prvalue semantics* guarantees copy elision since C++17. Just as what copy
elision means, a prvalue is not materialized until needed, and then it is
constructed directly into the storage of its final destination. This variant of
copy elision is known as URVO(unnamed return value optimization).

#### URVO in return statement

Example:

```c++
T g()
{
    return T(); // constructs the returned T directly.
}
```

#### URVO in initialization

Example:

```c++
T f();

T x = T(T(f())); // x is initialized by the result of f() directly.
```

### Copy elision not guaranteed

Copy elision is guaranteed when the variable is a prvalue(i.e. does not have
name/identity), and it is not guaranteed when the variable has name/identity.
This variant of copy elision is known as NRVO(named return value optimization).

#### NRVO in return statement

Example include:

```c++
T g()
{
    T res;
    return res; // constructs the returned T directly.
}
```

### Copy elision constraints

1. The destructor must be accessible at the point of copy elision, even though
   it may not be used.
2. The target object should has the same class type(ignoring cv-qualification)
   as the prvalue.
3. Copy elision is not guaranteed when initializing a base class
   [subobject](https://stackoverflow.com/questions/18451683/c-disambiguation-subobject-and-subclass-object),
   which can have different layout than the corresponding complete object type.
4. In NRVO, the returned operand should be the name of a non-volatile object
   with automatic storage duration, which isn't a function parameter or a catch
   clause parameter.

## Return value initialization

C++ does some extra work implicitly when it comes to initializing return value.
Some do not work as usual, and some even have side-effects. It is vital to
figure them out, or bugs grow.

### Return statement

Normally, return statement just works like initialization statement. It
calculates the return expression, and does some implicit conversion if needed,
and then uses the result to initialize the return value. In C++11, the return
expression can be a braced-init-list as well, and C++ will use
copy-list-initialization to construct the return value directly.

However, If returning by value involves copy/move construction, C++ will try to
do some optimization.

Specifically, C++ first tests if *copy elision* is available. If so, copy
elision is applied. Otherwise, C++ tests if the expression is *move eligible*.
If so, C++ moves the expression result automatically. Only after that, will C++
try copy construction.

### Automatic move

Consider this simple example.

```c++
std::string RandomString() {
  auto result = std::to_string(rand());
  return result;
}
  
std::string str = RandomString();
```

Intuitively, one may think there are multiply constructions and destructions.
