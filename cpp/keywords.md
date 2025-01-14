# Keywords

C++ has lots of keywords, and most of them are easy to understand its usage.
Some of them, however, may be quite difficult to figure out. Here we will
clarify them.

## `inline`

When we talk about the keyword `inline`, people can blurt out that it can
expand the function to improve the speed of the program. That's true. But in
these days, its role has been changed when we use it in a project.

### The original intent

Inline keyword was first designed to indicate the compiler to expand the
function so as to avoid overhead of calling it. But actually a compiler can
choose to ignore it or even inline a function that is not qualified by inline
keyword.  And the programer can hardly preceive it, since the compiler always
does better job in optimizing the code.

### Semantic change

In the original meaning, an inline function can be defined multiple times. This
actually a quite surprising fact. When we include a header file with an inline
function in it, we actually redefining the inline function in multiple
translation units. C++ makes it work by letting the linker to handle these
duplicated inline definitions. After linking, these definitions are unified
as if it is defined only once.

As a result, the meaning of inline has been changed to `multiple definitions
are permitted`. And since this semantic can be extended to variables, we get
`inline variable` in C++17. This works in the situation when we have a const
variable or a static member variable defined in header file. Without inline
keyword, this can leads to duplicated definition when the header file is
included more than once. Formal definition can be find
[here](https://en.cppreference.com/w/cpp/language/inline).

## `extern`

`extern` keyword lets entity (variable and function) to have *external
linkage*. (check the talk about linkage in `cpp/linkage.md`) It provides the
semantic that outer environment can use this symbol.

### 1. For global variable

Global variable has external linkage by default. But it's declaration and
definition are combined together. (global variable are zero-initialization by
default) This means if one want to share a global variable by putting it in a
header file, multiply definition may appear. So a better approach is adding
extern for the global variable. In this way, the declaration and definition can
be isolated into two parts.

```C++
// header.h
extern int val;

// source.cpp
int val = 10;
```

The example above well resloves the multiply definition error.

### 2. For function

Like global variable, function has external linkage by default. Unlike global
variable, function's declaration and definition can be separated without the
help of extern.  So extern is quite useless here. Of course you can add extern
for a function to explicitly indicate that the function has external linkage.

### 3. For const global variable

Const global variable has *internal linkage* by default. Hum, quite reasonable
if take its semantic in consideration. Since const variable just plays the same
role as literal. Well, there still exists case that share one instance across
translation units is needed. To achieve this we can add extern for it, which
grants const global variable with external linkage explicitly.

### 4. For compatible with C

C++ has *name mangling* for function, which changes the name after compiled.
Say we want to invoke a function from a C library.

```C++
// C library definition
int func() {
  return 1;
}


// C++ file
extern int func();
```

The linker will never find the corresponding name from C library, since C++
compiler has already mangling it.  To avoid this, wraps functions with `extern
"C"`.

```C++
// C library definition
int func() {
  return 1;
}


// C++ file
extern "C" {
int func();
}
```

C++ compiler does not mangling `func` now and linker can find the definition
in C library as well.

This works in reverse way. If we want to export C++ library interface to C
program.

```C++
// C++ header
// header.h
#ifdef __cplusplus
extern "C" {
#endif

int func();

#ifdef __cplusplus
}
#endif


// C++ source file
// source.cpp
#include "header.h"

int func() {
  return 1;
}


// C file
#include "header.h"

int main() {
  return func();
}
```

The `ifdef __cplusplus` guard here is to make the header file available for
both C and C++, since C can not recognize `extern "C"`. If your want to write a
library that will be used in both C and C++, add this guard. And `__cplusplus`
macro is included in the C++ standard that C++ compiler will define it when
compiling, so just consider it as the standard way.

By the way, function overloading is not permitted in the scope of `extern C`,
since C does not supports it.

### 5. For template

Like global variable, explicit template instantiation statement combines
declaration and definition. So like what `extern` does to global variable, it
can divide the two. This helps to restrict template instantiation. Check the
detail talk in `cpp/template`.
