# Time

Rust is quite strict when handling time stuffs. Well, the concept of time
occurred together with human civilization, so it deserves the complexity.

## Build from `NavieDate`/`NavieTime` to `DateTime`

When we talk about generating a timestamp, we may refer to different things.  A
timestamp without date information like `15:04:05`, or with date information
like `2006-01-02 15:04:05`, or even with time zone like `2006-01-02 15:04:05
UTC+0:00`.  Rust creates various struct for handling this in `chrono` library.

To begin with, we have `NavieDate`, which keeps date info only, like
`2006-01-02`. It is the most basic one. It can be built given year, month and
day in either digital or string form. It can also be built from existing
`NavieDate` by adding day or month.

Like `NavieDate`, we has `NavieTime`, which keeps hour, minute and second
information.  It provides similar interface as well.

Then we have `NavieDateTime`, which is the combination of `NavieDate` and
`NavieTime`.  Of course it can be built given string form of year, month, day,
hour, minute and second. But it does not has digital form interface. I do not
know why, maybe it costs too many arguments. Instead, it has another interface
that accepts a bias (i.e.  second and nanosecond) to [Unix
time](https://en.wikipedia.org/wiki/Unix_time). However, this interface is
deprecated, we would talk the reason below.

Finally we get `DateTime`, which adds `TimeZone` (keeps stuffs for `UTC`) to
`NavieDateTime`. It is the most complete one. Like `NavieDateTime`, it can be
built in string form, and it has an non-deprecated digital interface accepting
bias to Unix time. So why `NavieDateTime` one is deprecated? I guess that's
because we can directly get `DateTime` from this kind of interface assuming the
time zone is `UTC+0:00`.  Then we can generate `NavieDateTime` from this
`DateTime`.

So many types here. Well, the big picture is that we can build each one via
string form of corresponding interface. Or build from the basic
`NavieDate`/`NavieTime` to `DateTime`. If bias is needed (like adding one
second), all of them have implemented `Add` and `Sub`.

## Take time zone into consideration

For `NavieDateTime`, it does not have information about time zone, so we can
treat it as a `DateTime` with time zone `UTC+0:00`.

For `DateTime`, it treats time zone as a separate decoration field. In other
words, changing time zone only affects how the `DateTime` is displayed, but
would not change the date information in it. Say we have a `DateTime` with
`to_string` prints `2006-01-02 15:04:05 UTC+0:00`, and we change time zone to
`UTC+8:00`, it would then print `2006-01-02 15:04:05 UTC+8:00` instead of
adding hour field by eight.

So what if we want the time zone bias to be added to the date information? For
example, we get a `DateTime` with `2006-01-02 15:04:05 UTC+8:00`, and we want
to prints `2006-01-02 23:04:05`. Well, `DateTime` has an method named
`naive_local` that returns a `NavieDateTime` with bias added.
