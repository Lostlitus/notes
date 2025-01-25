# Virtual function

C++ supports virtual function. Programer could uses base class pointer or
reference to invoke derived class's virtual function. Very intutive. But how
does it works under this abstraction? The way to achieve it is
implementation-specific, we introduce the most common one here.

## Virtual table for virtual function

For a class with virtual function, compiler generates a unique *virtual table*.
The table stores all address of virtual functions. For base class, they are
just virtual functions for the base class itself. For derived class, it stores
base class virtual function as well, but the overrided function from derived
class would replace those from base class.

Say base class `Base` has virtual function `func1` and `func2`, derived class
`Derive` overrides function `func2` and has a new virtual function `func3`.
Their virtual tables look like this.

```txt
virtual table of Base:
1. address of Base::func1
2. address of Base::func2

virtual table of Derive:
1. address of Base::func1
2. address of Derive::func2
2. address of Derive::func3
```

So when invoking a virtual function, the code would search it in the virtual
table to find which version of it should be used.

## Virtual pointer for virtual table

So how to find virtual table of current instance? Here comes *virtual pointer*.
It stores the address of virtual table.  Virtual table is shared across the
class, but virtual pointer is stored one per instance. When a virtual function
is called, the code would go find the virtual pointer from the instance, and
use it to get the target function.

The virtual pointer is invisible for user, but it takes space like data member.
Say we have a class `Base` that only has one virtual function.

```C++
class Base {
  virtual void func1();
};
```

After compiling, compiler would add a virtual pointer for it.

```C++
class Base {
  FuncPtr *vPointer;
  virtual void func1();
};
```

So each instance of `Base` would takes 8 bytes space. This makes sense when it
comes to alignment.

```C++
class Base {
  FuncPtr *vPointer;        // 8 bytes
  int data;                 // 4 bytes
  // 4 bytes padding
  virtual void func1();
};
```

Without virtual class `func1`, it only takes 4 bytes. But with it, an extra
virtual pointer is needed, so plus 8 bytes, and alignment for `data` is needed,
so plus 4 bytes padding, with 16 bytes in total.

## Multiple virtual pointer for multiple inheritance

Above we assume there uses single inheritance. When it comes to multiple
inheritance, multiple virtual pointer is needed. Say we have two base class
`Base1` and `Base2` inherited by `Derive`.

```C++
class Base1 {
  int data1;
  virtual void func1();
};

class Base2 {
  int data2;
  virtual void func2();
};

class Derive : public Base1, public Base2 {
  int data3;
  virtual void func3();
};
```

Now the memory layout of `Derive` instance would look like this.

```txt
| vptr of Base1 | data1 | vptr of Base2 | data2 | data3 |
```

We see the virtual pointer of `Base2` is at the address that with a bias from
the `Derive` instance's address. It means the virtual table is split into two
parts.  And normally overrided virtual function would be recorded in the first
virtual table. The split virtual tables make a difference when trying to assign
`Derive` instance's address to `Base2` pointer or reference.

```C++
Derive *d = new Derive();   // Address of Base1 ptr
Base1 *b1 = d;              // Address of Base1 ptr
Base2 *b2 = d;              // Address of Base2 ptr
```

The special bias here for `Base2` is called *pointer fixup*. It occurs in
virtual inheritance as well. Say we have an additional virtual inherited base
class `VBase` for `Base1` and `Base2`.

```C++
class VBase {
  int data0;
  virtual void func0();
};

class Base1 : public virtual VBase {
  int data1;
  virtual void func1();
};

class Base2 : public virtual VBase {
  int data2;
  virtual void func2();
};

class Derive : public Base1, public Base2 {
  int data3;
  virtual void func3();
};
```

The memory layout now looks like this.

```txt
| vptr of Base1 | data1 | vptr of Base2 | data2 | data3 | ptr of VBase | data0 |
```

Since virtual inheritance is used, `VBase` would have a independent instance in
`Derive`'s instance. And it has a independent virtual pointer as well.
