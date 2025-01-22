# Linkage

Linkage plays a role in link time. It indicates the visibility of name across
translation units. This aims to avoid name conflict. For the name with related
entity, compiler/linker or programer needs to keep its entity unique in its
scoop as well.

1. External Linkage: can be referred to from other translation units. Only one
   related entity is permitted across all translation units.
2. Internal Linkage: used in its translation unit only. One related entity per
   translation unit, but only visable in in its translation unit.
3. No Linkage: used in its scope only.

## Some common linkage

- Name of global variable/function has external linkage.
- Name of class has external linkage.
- name of local variable has no linkage.

Check detail in [here](https://en.cppreference.com/w/cpp/language/storage_duration)

## Change linkage explicitly

- Name of global variable with `static`/`const` has internal linkage.
- Name of global function with `static` has internal linkage.
- Name of `static` variable member  has external linkage. (name of `static`
  member function keeps external linkage as non-`static` one)
- Name of `const` global variable with `extern` has external linkage.
- Name of `static`/`const` global variable with `inline` has external linkage.
- External linkage name (like `static` variable member's) in in anonymous
  namespace has internal linkage.

## For name only

One should keep in mind that the talk about linkage happens in link time only
and for name only. The reason why linkage is introduced is becasue it helps to
resolve the problem of name conflict as we've said at beginning.

For example, name of a global variable has external linkage, programer should
make sure it is defined only once across all translation units. So only one
entity of it is in the program. But when it comes to name of static global
variable, each translation unit can have its own entity. And compiler and
linker would makes sure it's invisable to other translation unit.

Name of type (like class) is special, since no entity is here. But it still
meets the problem of name conflict, so linkage is needed. Remember linkage is
to avoid **name** conflict.
