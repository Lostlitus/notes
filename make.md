# Make Note

## Make a Makefile

Make needs a Makefile to work, and a Makefile is mainly composed of rules.
Each rule has a target, prerequisites and a receipt. A target is always a file
to make, prerequisites are files needed to make the target and a receipt
consists of a series of commands that achieve the target. A rule looks like
this.

```make
# target prerequisites
#   |        |
#   v        v
main.o : main.c
    clang -c main.c -o main.o # <- receipt
```

## Variable

It is common in Make that rules have similar parts, so Make uses variable to
manage them. For example, in a C program, you always need a compiler driver.
And then you write a Makefile like this.(without variable)

```make
all: main.o tool.o
    clang main.o tool.o -o main

main.o : main.c tool.h
    clang -c main.c -o main.o

tool.o : tool.c tool.h
    clang -c tool.c -o tool.o
```

We keep on writing `clang -c` and `main.o tool.o`. Each time we want to change
them, we have to carefully modify them all in the Makefile, which is not only
boring but also error-prone. Now we can write Makefile like this.(with
variable)

```make
OBJS = main.o tool.o
CC = clang

all: $(OBJS)
    $(CC) $(OBJS) -o main

main.o : main.c tool.h
    $(CC) -c main.c -o main.o

tool.o : tool.c tool.h
    $(CC) -c tool.c -o tool.o
```

Now we only need to modify the defination, then the affect applies to all.

## PHONY Target

In Make, each rule has a target, and to achieve this target, receipt is
executed . Target itself are considered as a file to make by default. However,
there exists a case that we only want to let Make execute the corresponding
procedure(includes checking prerequisites and running receipt) and the target
is only a name rather than an actually file to make.  To make this available,
PHONY Target is proposed.

PHONY Targets provides two guarantees.

1. Make does not check if the target file exists.
2. Make skips implict rule search for PHONY Targets.

These two guarantees brings convenience. We always use PHONY Target like below.

- Mark final target as PHONY. This makes command make always available.
- Mark rule that do clean jobs as PHONY. Same reason.
- Use a variable to store PHONY Target to make Makefile more readable, for
  example, `PHONY = all clean` and `.PHONY = $(PHONY)`

## Rules without Recipes or Prerequisites

If a rule has no prerequisites or recipe, and the target of the rule is a
nonexistent file, then make imagines this target to have been updated whenever
its rule is run. This implies that all targets depending on this one will
always have their recipe run. As a result, it can achieve the same goal as
PHONY Target.

```make
clean: FORCE
    rm $(objects)

FORCE:
```

The target FORCE here fits the conditions, so the target clean, which depends
on it, is always runnable, just like PHONY Target.

## Automatic Variable

There exists case that we want to specify some names, for example, the target.
We often use target in corresponding receipt, so actually we write it at least
twice, which is error-prone. In Make, we use automatic variable to solve it.
It is actually an auto-refreshed variable that computes its value for each
rule.  And it works like this.

```make
main : main.c
    clang $< -o $@
```

In this example, `$<` and `$@` are automatic variables and they refer to the
first prerequisite and the target, respectively. And there are many other
automatic variables. Make good use of them help tidy Makefile.
