# Concepts

## Why Concepts

Concepts is a new terminology introduced in C++20. It is aimed to make
templates more readable and simpler. It achieves the former goal by add
constrains on template parameters, and achieves the latter by optimize the
grammer of templates. Besides, other benefits are introduced as well.

## More readable

### Simple function template Example

Example first:

```C++
template <typename T>
requires std::integral<T>
constexpr auto minus1(const T& value) {
  return value - 1;
}
```

The template above minus the `value` by one and returns the result. What is
different is that it has a `requires - concept` pair. The `requires` keyword
indicates this template has requirement to its parameters.(or say, constrains)
And the `std::integral<T>` here is one requirement(i.e. concept), which is
provided by the `<concepts>` standard library. This `requires clause` together
means the parameter `T` should be an integer.

The `requires clause` can have more than one concept and concepts are combined
together by boolean operator.

### Write our own concept

The example above uses the concept in standard library. We can write our own
concept with the `concept` keyword:

```C++
template <typename T>
concept Addable = requires(T a, T b) {
  a + b;
};
```

`requires expression` shown above creates our own concept that can be used int
`requires clause`. Simply put, the compiler will try to compile the expressions
in the bracket. Only when the template parameters fulfill all the expressions
can it be considered as pass the check.

One usage of this is to constrain the interface of template parameters given.
Say we need a template parameter that has `start` and `end` interface, then we
can write the following concept:

```C++
template <typename T>
concept StartableAndEndable = requires(T t) {
    t.start();
    t.end();
};
```

## Simpler

In C++20, we have:

- Abbreviated Function Templates
- Constrained auto

Like this:

```C++
constexpr auto minus1(const std::integral auto& value) {
  return value - 1;
}
```

This example is the simplified version of the example given above, but much
simpler. First, it omits the `parameter list` and replaces it with `auto`
keyword. Second, it omits the `requires clause` and the concept is placed
before the `auto` keyword.

## Other benefits

### Compile time check

Before we have concepts, most of the bugs occur in run time. Now these works
are moved to compile time, which advances the discovery of bugs.

### Better error message

Error message now with concepts is much more understandable.
