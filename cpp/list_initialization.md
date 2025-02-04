# List initialization

*list initialization* introduced in C++11 is aimed to provide a unified and
safe form of initialization. It uses braces `{}` instead of the old parenthesis
`()`.

```C++
// Initialize a with 0
int a{0};
```

## Works differently for different type

However, list initialization's meaning varies in different cases. So to
properly use it, we need to figure these scenarios out.

### For built-in type

For built-in type like `int` and `double`, it works the same as `=`.

```C++
// Identical
int a{0};
int b = 0;
```

Only one special thing. List initialization for built-in type limits [narrowing
conversions](https://en.cppreference.com/w/cpp/language/list_initialization#Narrowing_conversions).

```C++
// Compiler would throw an error
int a{3.14};
```

In short, it makes sure no information would lose when initialization.

### For C-style array

Now we can initialize array with elements. And C++ would infer length from the
list if no length is specified.

```C++
// Length 5. With elements 1, 2, 3, and the remaining are initialized to 0.
int a1[5] = {1, 2, 3};

// Length 5. With elements 1, 2, 3, 4, 5.
int a2[] = {1, 2, 3, 4, 5};

// Same for dynamic allocation
int *ap1 = new int[5]{1, 2, 3};
int *ap2 = new int[]{1, 2, 3, 4, 5};
```

### For class

Most complex part when comes to class (here we means class, structure and
union). When initializing, C++ would try to match several cases one by one.
Checks the details
[here](https://en.cppreference.com/w/cpp/language/list_initialization).

Well, the document above is too crazy to remember. We have a shorter one that
covers most of the cases.

First, check if the class has a constructor accpets a `std::initializer_list`.
If so, this constructor is called and the whole initializer list is used to
construct a `std::initializer_list` object implicitly for the constructor.

Otherwise, check if the class has a constructor matching the elements provided
in the initializer list. If so, this constructor is called and these elements
are passed for the constructor.

Otherwise, check if the class is an
[aggregate](https://en.cppreference.com/w/cpp/language/aggregate_initialization).
If so, initialize member data one by one with the initializer list elements.
This form of list initialization is named aggregate initialization.

That's all. One may say what about other cases. Well, don't use list
initialization for them. No one could understand these code.

### For empty initializer list

For built-in type, value initialization is called. (i.e. zero initialize the
object). For class, default constructor rather than the constructor accpeting a
`std::initializer_list` is called. For aggregate type, all elements is value
initialized.

Note if the class is an aggregate, the aggregate type rule rather than class
rule is applied. It makes sense when the class has built-in type. Default
constructor generated implicitly default initializes member data, so built-in
type member would be left uninitialized.  But for aggregate type's value
initialization, built-in type member is zero initialized.

By the way, `{}` has one additional benefit that `()` does not have. See this
code snippet.

```C++
Class MyClass;

MyClass obj{};  // Global variable
MyClass func(); // Global function
```

If we want to defining a global `MyClass` variable via `()` default
constructor, the compiler would interpret it as a no argument function which
returns `MyClass`. That is, ambiguous code here. But if `{}` is used, it's OK.

## Direct vs copy

List initialization has two form, *direct list initialization* and *copy list
initialization*.

```C++
int a{1};
int b = {1};
```

The direct form construct the object directly. While the copy form construct a
temporary object, and the object copy constructs from it. Since compiler would
optimize the additional copy, the two gives identical result. Expect for two
cases.

The first is the copy form cannot uses `explicit` constructor. Quite
reasonable, since explicit constructor cannot be used to create temporary
object.

The second one is `auto` deduction. For direct form, auto gives the same type.
But for copy form, auto gives `std::initializer_list`. Humm, still reasonable?
Note initializer list in list initialization is not a `std::initializer_list`
object. Remeber the example of a constructor accpeting a
`std::initializer_list`, C++ constructs a temporary `std::initializer_list` via
initializer list first then calls the constructor. So the deduction here should
be considered as an exception.

## Best practice

After reading the talk above, we find that list initialization seems to mess
things up. Yes, but only if you use it to do all initialization. Here is the
best practice.

- For built-in type, initialize with `=`.
- For C-style array, initialize with direct list initialization.
- For class, initialize with parenthesis. Only uses list initialization when it
  has a `std::initializer_list` constructor.
- For aggregate type class, initialize with direct list initialization.
- Never use copy list initialization.
