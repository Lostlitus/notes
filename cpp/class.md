# Class

Class is the core conception of OOP, the same goes for C++. Defining and using
a class is quite easy in C++. But there are some questions that may mess you
up. Now we go through these in FAQ format.

## 1. Construction && destruction

### 1.1 Why no virtual constructor?

Given that C++ supports virtual destruction, it's intrigue to ponder why it
does not have virtual constructor. Well, the purpose of virtual function in C++
is actually using patial information to get work done. That is, you invoke the
interface from base class, and the derived class's function is then found and
actually invoked. Back to constructor, you always need the whole information of
the object you want to create, which does not fit the conception of virtual
function.  So just no virtual constructor. Details can be viewed
[here](https://www.stroustrup.com/bs_faq2.html#virtual-ctor) from Bjarne
Stroustrup.

### 1.2 What does a derived class do when constructing?

In C++, a derived class constructor always invoke base class constructor first.
We can call it explicitly in member-initializer list, and the real order would
be the same order as inheritance order in the case of multiple inheritance.  If
no explicit invocation, there would be an implicit call to base class's default
constructor.

After that data member would be initialized. Similarly, we can initialize it
explicitly in member-initializer list, but the real order would be the
definition order.

Finally, the code block in derived class constructor would be executed.

### 1.3 What happens when invoking virtual function in constructor?

It depends on language implementation. In C++, a class is built from base class
to derived class. Say virtual function is called in both base class constructor
and derived class constructor. When constructing, base class constructor is
called and the virtual pointer is set to base class virtual table, so base
class version virtual function is called. After that derived class constructor
is called and the virtual pointer would set to derived class virtual table, so
now derived class version virtual function would be called.  As a result, it's
always the virtual function that in the same hierarchy level would be called.
This rule applies to destructor as well.

One thing to note is if the constructor calls a pure virtual function, compiler
would throw an error, since it cannot find the implementation.

Here we can see the virtual function actually works like a non-virtual
function.  So yes, you **should** avoid invoking virtual function in
constructor. It makes no sense. No polymorphic here.

## 2. Member function

### 2.1 How does const member function work?

It tells the user this interface is read-only. Member function is function with
a `this` pointer as extra argument. In const member function's case, the
pointer points to a const `this`.

```C++
clase Test {
  void func();
  void func() const;
};

int main() {
  Test vt;
  const Test ct;

  vt.func();
  ct.func();
}
```

Above we see overloading works for non-const and const member function. Here,
`void func()` equals to `void func(Test *this)` and `void func() const` equals
to `void func(const Test *this)`. So `vt.func()` invokes the non-const one and
`ct.func()` invokes the const one. Both get the best match signature.

If there only has non-const version, compiler would throw an error for
`ct.func()` since we cannot pass a `const Test *` to a `Test *`. In other side,
if only has const version, `vt.func()` still OK, since `Test *` can be accepted
by `const Test *`. It's all about signature match.

Normally, code in const member function cannot modify any member data. But we
can declare a member data with `mutable` to make it modifiable in const member
function. The reason why C++ supports this is because the read-only here
indicates *logical read-only*. It means the interface promises the whole
instance would keep its consistency after invocation. So modification that does
not change the consistency is permitted. A classic scenario is using lock in
const member function. Acquiring and releasing lock modifies data in lock
member, but the class instance's consistency remains unchanged, so `mutable` is
required.

## 3. Inheritance

### 3.1 Why virtual inheritance

C++ supports multiple inheritance. It's possible for a class to inherit from
two classes which inherit from same base class. This way, the derived class
would have two set of identical data from that base class. It leads to
ambiguous look up when we try to access the data in the code.

```C++
class Base {
public:
  int data;
};

class Derive1 : public Base {
};

class Derive2 : public Base {
};

class Final : public Derive1, public Derive2 {
  void func() {
    this->data;
  }
};
```

The `func` above access `data` member in `Base`, but `Final` has two instance
of `data`, so `this->data` is ambiguous.

One quickfix is indicating which one to access via base class path.

```C++
class Final : public Derive1, public Derive2 {
  void func() {
    this->Derive1::data;
    this->Derive2::data;
  }
};
```

But the problem still exists, `data` is redundant. So here comes *virtual
inheritance*. By marking classes with same base class as `virtual` inherit.
Compiler would drop the duplicate base class data and keep only one copy of the
data.

```C++
class Base {
public:
  int data;
};

class Derive1 : virtual public Base {
};

class Derive2 : virtual public Base {
};

class Final : public Derive1, public Derive2 {
  void func() {
    this->data;
  }
};
```

Here we marks both `Derive1` and `Derive2` as `virtual` inherit from `Base`, so
`Final` only has one set of `data` from `Base`. No ambiguity now.
