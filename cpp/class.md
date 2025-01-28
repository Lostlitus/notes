# Class

Class is the core conception of OOP, the same goes for C++. Defining and using
a class is quite easy in C++. But there are some questions that may mess you
up. Now we go through these in FAQ format.

## 1. Construction

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
