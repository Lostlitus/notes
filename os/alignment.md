# Alignment

## Why alignment

Hardware performs efficiently in accessing data when the data is *naturally
aligned*, which means the data's memory address is a multiple of alignment
size. This way, fetching the data can be finished in one cycle since the data
is stored in one memory field. Otherwise, the data can be stored across two
memory fields so multiple cycles may be needed, and we call it *misaligned*.
Some instructions even rely on alignment to work correctly, like SSE of Intel
CPU.  Language together with compiler would attempt to make sure data is
naturally aligned.

## Usually, aligned by data size

In most cases, alignment size equals to data size. Say we have a 4 byte `int`,
the compiler would try to assign it with an address which is a multiple of 4.
The same rule is applied to other types.

One exception is that the alignment size of an array would be the size of its
element type rather than the array size. For example, 1 byte alignment for
`char a[10]`.

Note these just work in common cases. The exact alignment size is
compiler-specific.  Different compiler chooses different policy, and even in
same compiler, the policy is diverse across different platform.

## Padding between two data

If we put two data together, *padding* may needed. For example, we declare one
`char` and one `int` together in a function like this.

```C++
int main() {
  char a;   // 1 byte
  // 3 byte padding
  int b;    // 4 byte
}
```

To make sure both `a` and `b` here naturally aligned, compiler adds 3 bytes
padding between two. This applies to structure as well.

```C++
struct Block {
  char a;   // 1 byte
  // 3 byte padding
  int b;    // 4 byte
  char c;   // 1 byte
  // 3 byte padding
};
```

Note the `Block` above has extra 3 bytes padding at the end. That's because the
whole structure's alignment is equal to the maximum alignment of all its
elements, and in `Block` it is int type `b` with 4 bytes alignment. Compiler
adds this rule to make sure when there is an array of `Block`s, each element in
each `Block` instance is still naturally aligned.

The padding wastes space. But we can adjust variable order to reduce it as much
as possible.

```C++
struct BetterBlock {
  char a;   // 1 byte
  char c;   // 1 byte
  // 2 byte padding
  int b;    // 4 byte
};
```

The `Block` takes 12 bytes in total. And our `BetterBlock` takes only 8 bytes,
just by reordering the variables. So it's important to make your code naturally
aligned friendly.

## Change alignment size manually

Language or compiler provides instructions to adjust compiler alignment policy,
or even to disable it. In C++ for example, we has `#pragma pack` to adjust
maximum align size. If we set it to 1, then no padding would be generated. This
makes sense when we need a compact layout. Say, network pack.
