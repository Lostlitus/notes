# Linkage

Linkage plays a role in link time. It indicates the visibility of an entity
(variable or function) across translation units.

1. External Linkage: can be referred to from other translation units. Only one
   instance across all translation units.
2. Internal Linkage: used in its translation unit. One instance per translation
   unit if included.
3. No Linkage: used in its scope only.

## Default linkage

- Global variable and function have external linkage.
- Local variable has no linkage.

## Change linkage explicitly

- Global variable with `static`/`const` has internal linkage.
- Global function with `static` has internal linkage.
- `const` global variable with `extern` has external linkage.
- `static`/`const` global variable with `inline` has external linkage.
- Global variable/function in anonymous namespace has internal linkage.

## For entity only

One should remember that the talk about linkage happens in link time only and
for entity only. The reason why linkage is introduced is becasue it helps to
resolve the problem that how many instance of one entity should a program keep.

But there has contents that are not entity, like class definition. And they can
meet problems like definition conflict as well. So here comes the verse of
static and anonymous namespace. Both of them can turn global variable/function
to have internal linkage. But anonymous namespace also can hiding name of other
contents like we said above, class definition. This way, name conflict during
link time can be resolved. So now the much more mordern way is using anonymous
namespace.
