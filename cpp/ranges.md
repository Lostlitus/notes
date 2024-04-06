# Ranges

## What is ranges and why

The most regular way to handle a list of datas is using `for loop`. And then it
turns to the `for each` for simplity. Now things become much simpler and more
standardized in C++20 with ranges. That is, ranges as a abstraction layer,
uniforms the way to handle datas. Generally speaking, now it looks like the
`pipe` in shell script.

Ranges works in a quite simple way. There should be one data source, then it is
processed, and finally we get the result. Here is a example:

```C++
const std::string proto = {"lower case"}; // Data source

auto view_proto = std::views::transform(
                  std::views::reverse(proto),
                  ::toupper); // Processing

const std::string result(view_proto.begin(), view_proto.end()); // Result
```

And we will explain each of these 3 steps.

## Data source

In the context of ranges, the data source is just one of the different types of
ranges. And the mostly used types of ranges are `container` and `view`. The
former owns data and the latter doesn't.

In the example above, `proto` is `container` and `view_proto` is `view`. A
container is transformed to view after processing.

## Processing

The callable object given in the example are part of `std::views`, and they
return what is called `adaptor` in `std::ranges`. The `adaptor` is then passed
to next view callable object until all the callable objects are gone through.

So there are two ways for one to generate and process adaptors. The first is by
using callable objects in `std::views` like the example shows. And another one
is by creating adaptor directly with its constructor like this:

```C++
auto adaptor_result = std::ranges::transform_view(
                      std::ranges::reverse_view(proto),
                      ::toupper);
```

The `std::ranges` way should only be used when we are implementing a view. Not
only because the `std::view` way is more intuitive, but it also has some
smarter implementation.(We are not going to have a in-depth introduction here)

There is a syntactic sugar for `std::view` way. That is, we can process ranges
in a shell-style way:

```C++
auto pipe_result = proto | 
                   std::views::reverse | 
                   std::views::transform(::toupper);
```

The `|` operator here works like `pipe` in shell. It is much intuitive, right?

## Result

In C++20, if we want to get the result after processing, we need to call the
constructor-like function like what we do in the example. So in C++23, the
missing part is introduced:

```C++
auto result = proto | 
              std::views::reverse | 
              std::views::transform(::toupper) |
              std::ranges::to<std::string>();
```

The `std::ranges::to` optimize the additional step to build the final result in
a separate sentence, which complete the way we processing ranges.
