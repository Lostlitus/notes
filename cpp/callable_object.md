# Callable object

## Various kinds

The *callable object* is an object that can be called like a function. (i.e.
invoked via `()`) There are various kinds of them, and we introduce here.

### 1. Function

No doubt that function is one kind of callable object.

### 2. Function pointer

The pointer of a function can be called like a function. Dereference of the
pointer can be omitted.

```C++
void func();

int main() {
  void (*ptr)() = func;
  ptr();
  *ptr();
}
```

It's worth note that member function pointer is also callable.

```C++
class MyClass {
public:
  void func();
};

int main() {
  void (MyClass::*ptr)() = &MyClass::func;
  MyClass obj;
  (obj.*ptr)();
}
```

### 3. Function object

A *function object* (or *functor*) is an object with `()` operator overloaded.

```C++
struct MyFunctor {
  void operator()(int x) const {
    cout << x << endl;
  }
};
int main() {
  MyFunctor functor;
  functor();
}
```

### 4. Lambda expression

*lambda expression* is an anonymous function that can be defined and used in
code directly. It works great when we need a one-time function.

```C++
int main () {
  std::vector<int> vec;
  vec.push_back(1);
  vec.push_back(4);
  vec.push_back(3);
  std::sort(vec.begin(), vec.end(), [](const int &a, const int &b) -> bool {
      return a > b;
  });
}
```

### 5. `std::bind`

`std::bind` binds arguments with a callable object and generates a wrapper
callable object.

```C++
void Print(const std::string &msg) {
  std::cout << msg << std::endl;
}

int main () {
  auto PrintHello = std::bind(Print, "Hello");
  PrintHello();
}
```

### 6. `std::function`

`std::function` is used to encapsulate callable object.

```C++
void Print(const std::string &msg) {
  std::cout << msg << std::endl;
}

int main () {
  std::function<void(const std::string &)> func = Print;
  func("Hello");

  std::function<void(const std::string &)> lam = [](const std::string &msg) {
    std::cout << msg << std::endl;
  };
  lam("World");
}
```

## Predicate in algorithm

C++ standard library provides lots of algorithm. Some of them accept a
*predicate* to check if condition meets request. A *predicate* is one subset of
callable object, which accepts either one argument (*unary predicate*) or two
arguments (*binary predicate*) and returns a boolean as result.

The sort example given in the lambda section shows how does an algorithm use a
unary predicate to find element in a vector.

One case here is that we may use some auxiliary variables in predicate body.

```C++
struct MyFunctor {
public:
  int target = 0;
  bool operator()(const int &val) const {
    return val == target;
  }
};

int main () {
  std::vector<int> vec{1, 2, 3, 4};

  auto res = std::find_if(vec.cbegin(), vec.cend(), MyFunctor());
}
```

Here we use functor as predicate for `std::find_if`. In the body of overloaded
`()`, an auxiliary variable `target` is used. Functor stores it as a member.

This can be achieved by lambda expression via *capture list*.

```C++
int main() {
  std::vector<int> vec{1, 2, 3, 4};
  int target = 1;

  auto res = std::find_if(vec.cbegin(), vec.cend(), [target](const int &val) {
      return val == target;
  });
}
```

The lambda expression given above captures `target` as an auxiliary variable,
similar to the functor example above.

It's available for `std::bind` as well, but with another method.

```C++
bool check(const int &val, const int target) {
  return val == target;
}

int main () {
  std::vector<int> vec{1, 2, 3, 4};
  int target = 3;

  auto bindCheck = std::bind(check, std::placeholders::_1, target);
  auto res = std::find_if(vec.cbegin(), vec.cend(), bindCheck);
}
```

Auxiliary variables in above examples have non-static lifecycle, so each method
has its workaround. For static lifecycle data, just use it directl in the code
body. Take function as an example.

```C++
const int sTarget = 4;
bool checkS(const int &val) {
  return val == sTarget;
}

int main() {
  std::vector<int> vec{1, 2, 3, 4};

  auto res = std::find_if(vec.cbegin(), vec.cend(), checkS);
}
```

This works for functor, lambda expression and `std::bind` as well.
