# Definition conflict

Definition conflict is a common thing in C++. Once appear it can be quite
tricky to resolve. It's better to avoid it before actually meeting it.

## Compile time conflicts

Compile time conflicts appear in the scoop of translation unit.  C++ expands
the file indicated by `#include` preprocessor directive in preprocess time and
compiles it in compile time. Here comes the question.  If a header file is
expanded multiple times, then the definitions in it appears more than once, and
definition conflict occurs.

```C++
// header1.h
int val;

// header2.h
#include "header1.h"

// example.cpp
#include "header1.h"
#include "header2.h"
```

The example above is quite common in real environment. The global variable
`val` occurs twice in file `example.cpp` after expanding, and the compiler will
throw an error in compiling.

To avoid it we have two choices. The more traditional approach is by using
*include guard*.

```C++
// header1.h
#ifndef HEADER1_H
#define HEADER1_H
int val;
#endif

// header2.h
#include "header1.h"

// example.cpp
#include "header1.h"
#include "header2.h"
```

When preprocessing, preprocessor expands `header1.h` twice as well, but only
one copy of the definitions in it remains. Here, the `#ifndef` macro play the
effort. Note that this approach needs a unique name for each header file, or
wired bugs appear. Imagine your project uses `CONFIG_H` for your configuration
header file, and you introduced a third-part library which happens using
`CONFIG_H` as well. Then boom, one of the header files never works.

Another simplier way is adding `#pragma once` in the header file. This way the
compiler just ignores this file if it has been parsed once before. Compared to
*include guard*, it's semantic is much more clear for compiler, and build speed
is faster. Compiler achieve the goal by storing the header files information
when compiling. For those files with `#pragma once`, compiler expands them only
one time.  This syntax is not in C++ standard and it relies on the support of
compiler.  However, now all mordern compilers(gcc, clang, msvc) support it, so
feel free to use it in practice.

## Link time conflict

Link time conflict appear when linking the object files of translation units.

```C++
// header.h
#pragma once
int val;

// a.cpp
#include "header.h"

// b.cpp
#include "header.h"
```

In the example above, if we compile `a.cpp` and `b.cpp` alone, no error is
thrown by compiler. However, if we link the object file of them, an error will
be thrown by the linker. Even though we add `#pragma once` for the header file.
Include guard or `#pragma once` only provides protection in compile time. There
are three approaches to solve this.

One method is defining the global value/function with `static` keyword in
source file.  This works when the definition only means to share in one
translation unit.  Since `static` limit the symbol linkage to *internal
linkage*. That is, each translation unit keeps one private instance of the
symbol.

The second approach is declaring the global value/function with `extern`
keyword in header file and defining it in source file. The `extern` keyword
indicates the symbol to have *external linkage*. In this way, the symbol is
only defined once. Other files including the header file only declare the name,
so no conflict here. This makes sure there will be only one instance of the
symbol.

The last way is defining with `inline` keyword. Compiler and linker assume the
definition of an inline symbol(inline variable is supported since C++17) is
identical in every place. In this case, it is programer's responsibility to
keep the definition unique. A common way to achieve this is defining in the
header file only. Like the second approach, this make sure there will be only
one instance of the symbol in the final object file. The promise here is
provided by compiler and linker.

```C++
// Approach 1
// Define in source file rather than in header file

// source1.cpp
static int val = 10;

// source2.cpp
static int val = 20;

// Links source1.cpp and source2.cpp works well.


// Approach 2
// Declare with extern in header file and defining in source file

// header.h
extern int val;

// source.cpp
int val = 10;

// a.cpp
#include "header.h"

// b.cpp
#include "header.h"


// Approach 3
// Defining with inline in header file

// header.h
inline int val = 10;

// a.cpp
#include "header.h"

// b.cpp
#include "header.h"
```
