# Class

Class is the core conception of OOP, the same goes for C++. Defining and using
a class is quite easy in C++. But there are some special cases that may mess
you up. Now we go through these in FAQ format.

## 1. Construction

### 1.1 Why no virtual constructor?

Given that C++ supports virtual destruction, it's intrigue to ponder why it
does not have virtual constructor. Well, the purpose of virtual function in C++
is actually using patial information to get work done. That is, you invoke the
interface from base class, and the derived class's function is then found and
actually invoked. Back to constructor, you always need the whole information of
the object you want to create, which does not fit the conception of virtual
function.  So just no virtual constructor. Details can be viewed
[here](http://www.stroustrup.com/bs_faq2.html#virtual-ctor) from Bjarne
Stroustrup.
