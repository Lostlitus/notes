# Standard library

The *standard library* (or *std* in short form) consists of various kinds of
developing toolkits. Though designed with clean interfaces, some parts of it
needs additional attention. And we introduces them here.

## Data structure

### Special implementation

#### 1. `std::deque`

`std::deque` provides interface similar to `std::vector`. And it has an
additional feature, that is, manipulating head element. But their
implementations are different.

For `std::vector`, elements are stored together in one chunk of successive
memory.  For `std::deque`, on the other hand, elements are stored in serval
separated fixed-size chunks, which are managed by a controller data structure.

So for `std::vector`, when hiting the capacity bound, all elements are moved to
a new area. And its locality is great.

For `std::deque`, when entending capacity, it allocates new chunk for new
elements and the original elements stay still. And its locality is bad compared
to `std::vector`.

### Iterator invalidation

Iterator is a clean abstraction interface for container in std, implemented by
encapsulating raw pointer. Since raw pointer may become invalid after some
memory modification operation, iterator would loses its validation in same
scenario as well. When learning these cases, consider we are manipulating raw
memory via raw pointer.

In the scoop of container, most frequently used memory modification operations
are inserting and erasing.  Whether an iterator would become invalid or not is
related to the type of container. The basic rule is avoiding using the origin
iterator after modification. Instead, using the iterator returned by `insert`
and `erase` function, which is promised to be valid.

However, we may meet some code that does not follow this rule. That is, using
old iterator after modifying the container. Some are reasonable, the others are
error-prone. So we needs to understand in which case would the iterator loses
validation.

#### 1. Sequential container (non-list)

For non-list *sequential container* (`std::vector` for example), both inserting
and erasing lead to invalidation of current iterator and iterators after it.
Specially, if the memory is reallocated when inserting (hits the capacity
bound), all iterators become invalid.

#### 2. Sequential container (list)

For list *sequential container* (`std::list` for example), only erasing leads
to invalidation, and only current iterator.

#### 3. Associative container (ordered)

For ordered *associative container* (`std::map` for example), only erasing
leads to invalidation, and only current iterator.

#### 4. Associative container (unordered)

For unordered *associative container* (`std::unordered_map` for example),
erasing leads to invalidation, and only current iterator. For inserting,
current iterator would loses validation if rehashing happens.

#### 5. Manually change memory

Other than inserting and erasing, container supports some interfaces that
manipulates memory directly. `clear` for example, it leads to invalidation for
all iterators. Since the whole memory of the container is cleared, so raw
pointer becomes invalid, so iterator becomes invalid.

Another one is `resize` in sequential container. For non-list, it invalidates
erased elements' iterators if reducing capacity, and invalidates all iterators
if reallocation happens when increasing capacity. For list, the same as non-list
for reducing capacity, but iterator keeps valid when increasing capacity since
reallocation does not influence existing node.

There are other operations, refer to the their documents for details about
iterator invalidation.

#### Conclusion

There are various scenarios and quite hard to remember them all. Well, as we
said above, the best practice is avoiding handling these cases. So here is the
conclusion.

1. Always reacquire iterator after modification.
2. If you **must** use the origin iterator, write down your reason nearby.
3. If you are reading other's code, consult the container document with care.
