# Keywords

C++ has lots of keywords, and most of them are easy to understand its usage.
Some of them, however, may be quite difficult to figure out. Here we will
clarify them.

## 1. Inline

When we talk about the keyword `inline`, people can blurt out that it can
expand the function to improve the speed of the program. That's true. But in
these days, its role has been changed when we use it in a project.

### The original intent

Inline keyword was first designed to indicate the compiler to expand the
funtion so as to avoid overhead of calling it. But actually a compiler can
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
`inline variable` in C++17. Formal definition can be find
[here](https://en.cppreference.com/w/cpp/language/inline).
