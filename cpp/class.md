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
