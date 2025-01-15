# Endianness

I feels like everyone is talking about endianness, like it is a crazy complex
concept and none can truly understand. Well, when I search it, I find it is
really simple.

## Why endianness

Computer uses bytes as the minimal unit to manipulate data. Bytes are stored
and read from lower address to higher in order. No endianness is needed in
these case. The following snippet shows how an array of `char`(one byte) is
stored in C.

```txt
code:

char data[4];
data[0] = 0x1;
data[1] = 0x2;
data[2] = 0x3;
data[3] = 0x4;

memory:

lower address -> higher address
0x1 0x2 0x3 0x4
```

But there are types of data block need multiple bytes to store, from `int` to
`structure` and to `unicode`. Here comes the question. Are these the bytes in
these data stored from lower byte to higher byte as well? Now endianness
appears.

## Little Endian vs Big Endian

Take 4 bytes `int` as example, it stores distinctly in little endian machine
and big endian machine.

```txt
code:

int data[2];
data[0] = 0x11223344;
data[1] = 0x55667788;

little endian memory:

lower address -> higher address
0x44 0x33 0x22 0x11 0x88 0x77 0x66 0x55

big endian memory:

lower address -> higher address
0x11 0x22 0x33 0x44 0x55 0x66 0x77 0x88
```

We can see that `int`s are still stored from lower to higher, but how the bytes
inside an `int` are stored is determined by endianness.  From lower address to
higher address, little endian machine machine stores lower byte first and big
endian stores higher byte first. That is, endianness influences the way inner
bytes of a data stores. That's all about endianness.

## Misunderstanding point

The point people misunderstand endianness, I guess, is the inner bytes of big
endian. In big endian, data are stored from lower byte to higher byte, but the
inner bytes of data are stored in reverse, really weird. Especially the address
of the whole data is the start address of the data, while the lowest byte of
the data is at the end of the data.

```txt
code:

int val = 0x11223344;

big endian addresses:

address of the whole data(i.e. &val)
|
v
0x11 0x22 0x33 0x44
               ^
               |
               address of the lowest byte of the data(i.e. address of (uint8_t)val)
```

It's annoying to distinguish these two address when talking about big endian.
And it is not a case in little endian, since the two are equal.

## Pros and Cons

Each endianness has pros and cons, so choosing which endianness depends on the
use case.

In the case of memory dump, big endian machine prints in a human readable way.

```txt
code:

int val = 0x11223344;

memory dump:

little endian -> 0x44 0x33 0x22 0x11
big endian -> 0x11 0x22 0x33 0x44
```

In big endian machine, the memory dump just prints in the order we write in the
code.

But in the case of data casting, little endian is better.

```txt
Change 4 byte int type data 0x11223344 to 1 byte uint8_t type data 0x44.

little endian machine:

address(4 bytes)       address(1 byte)
|                      |
v                      v
0x44 0x33 0x22 0x11 -> 0x44 0x33 0x22 0x11

big endian machine:

address(4 bytes)                      address(1 byte)
|                                     |
v                                     v
0x11 0x22 0x33 0x44 -> 0x11 0x22 0x33 0x44
```

To truncate the higher 3 bytes, little endian here only needs to change the
type, while big endian needs to move the address pointer to the lowest byte of
the data in addition.

## In practice

Modern OS supports both little endian and big endian. For user computer, we can
say that Intel, AMD and Apple provide only little endian chips. So your PC is
always a little endian machine. But there still exists cases that big endian
makes sense. For example, big endian is common in network protocols like
TCP/IP, DNS and so on. This makes most network devices like router to uses big
endian.

So to communicate between PC and network, convention between little endian and
big endian is neeeded. In C for example, we have functions like `htonl` in
standard library to do this job. Check details via `$ man byteorder`.

It's also available to write our own code to determine whether current machine
is little endian or big endian.

```C++
union test {
  char c[4];
  unsigned int endian;
} t;

int main() {
  t.c[0] = 'l';
  t.c[1] = '?';
  t.c[2] = '?';
  t.c[3] = 'b';
  std::cout << (char)t.endian << std::endl;
}
```

The example above stores 4 bytes one by one from lower address to higher
address and then interprets these bytes as an 4 byte long `int` and reads the
content of the byte its address points to. In the case of little endian, it is
the byte `l` in lower memory address.  In the case of big endian, it is the
byte `b` in higher memory address.
