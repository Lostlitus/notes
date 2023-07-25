# SQL PLUS Note

## Background

Oracle Database is worked as a server, so it needs a client to interact with,
and one of the most basic Oracle Database utility is [SQL
PLUS](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqpug/index.html#SQL*Plus®),
officially called SQL\*PLUS. It is installed with every Oracle Database
installation and has a command-line interface. We can either use the cli or
some wrapper gui programs to utilize SQL PLUS and then interact with Oracle
Database. It is important to understand the cli, however, because many gui
programs are made above the cli, and some weird designs can be reasonable after
understanding the cli.

## Syntax

Now introduce three mostly used parts of SQL PLUS.

1. SQL commands, for working with information in the database.
2. PL/SQL blocks, also for working with information in the database.
3. SQL Plus commands, for formatting query results, setting options, and
   editing and storing SQL commands and PL/SQL blocks.

In cli, the manner in which you continue a command on additional lines, end a
command, or execute a command differs depending on the type of command. Each
type of them has a corresponding mode, so their syntaxes in SQL PLUS differs.

### SQL Buffer

Before introducing these parts, we should first understand the concept called
SQL Buffer in SQL PLUS. It is designed to store the most recently entered SQL
command or PL/SQL block. Command or block stored can be executed by entering
RUN or forward slash commands. It is a good design especially in the scenario
of cli.  Because it makes the type-and-run mode in cli more flexible.

It is worth mention that SQL Plus commands are not stored, nor the semicolon or
forward slash character typed for executing a command.

### SQL commands

They are just SQL that share concept with SQL in other databases like Mysql.

To continue a SQL command in additional lines, press return.

To end a SQL command, enter a semicolon, a forward slash or a blank line.  Both
semicolon and forward slash then execute the command immediately. The only
difference is that semicolon trails the last command while forward slash is on
a new line. And a blank line just tells SQL PLUS to save the command to the SQL
buffer(the older command is overwritten in this case) without executing it.  As
mentioned before, entering forward slash or RUN command to execute the command
stored.

### PL/SQL blocks

PL/SQL is Oracle's extension for SQL.  It is also called PL/SQL subprograms.
PL/SQL is a block-structured language, that is, its basic units(procedure,
function and anonymous block) are logical blocks. And these blocks can contain
any number of nested sub-blocks, making PL/SQL a powerful tool.

A PL/SQL block has three parts: a declarative part, an executable part, and an
exception-handling part. Only the executable part is required. Obviously, the
order of these parts is logically designed. First comes the declarative part,
in which items can be declared. Once declared, items can be manipulated in the
executable part.  Exceptions raised during execution can be dealt with in the
exception-handling part. Take anonymous block as an example, the whole
structure looks like this.

```sql
DECLARE
    --declaration
BEGIN
    --statement
EXCEPTION
    --handler
END;
```

Details of PL/SQL language can be found in [Database PL/SQL Language
Reference](https://docs.oracle.com/en/database/oracle/oracle-database/21/lnpls/index.html#Oracle®-Database)

To continue a PL/SQL command in additional lines, press return.

To end a PL/SQL block, enter a period. Unlike SQL command, a blank line or a
semicolon does not end PL/SQL block.

To end and run or to run buffered PL/SQL block, enter a forward slash.

### SQL Plus command

SQL Plus command can be used to manipulate SQL commands and PL/SQL blocks.
Details of SQL Plus command can be found in [SQL Plus Command
Reference](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqpug/SQL-Plus-command-reference.html#GUID-177F24B7-D154-4F8B-A05B-7568079800C6)

To continue a SQL Plus command in additional lines, type a hyphen at the end
of the line and press return.

To end a SQL Plus command, press return. And this also run the command
immediately.

## PL/SQL blocks details

Unlike SQL command, `SELECT` statement in PL/SQL blocks is forced to assign
values to a variable by using `SELECT INTO`. This means we can not use select
statement to print value to console.

### Store Procedure

Store procedure in SQL Plus has similar syntax as anonymous block. What differs
most is that it has name and input/output arguments. Generally, it looks like
this.

```sql
CREATE [OR REPLACE ] PROCEDURE procedure_name (parameter_list)     
IS
    --declaration statements
BEGIN
    --execution statements
EXCEPTION
    --exception handler
END procedure_name;
/
```

#### Privilege

Oracle database compiles store procedure before invocation. And a compiled
store procedure becomes invalid if its owner's privilege changes. In orcale
there are two ways to changes privileges, grant directly and grant indirectly
by role. Oracle believes role itself is a fluid properity, which may be changed
easily. So oracle only uses directly granted privilege in store procedure to
avoid frequently recompiling.  This means a user with privileges granted by
role may encounter error when invoking store procedure, while in anonymous
block everything goes well.

