# Build from Source Note

Build from source is always a way to install the program we want. And a project
that claim itself open source always provide a README to state how to build it.
Some of them are quite straightforward and detailed, while others seems
complicated.  The reason why they are difficult to understand is because they
have conventions and they assume we already knew them. If these conventions are
listed in plain words, however, they are quite clear.

## Get source code

We can get source code in various ways. The most basic way is cloning from the
git repository. Some repositories maintain multiple versions, so we need to
checkout to the corresponding branch we want. Another basic way is downloading
archive from official website and we then get the source code by extracting it.

## Configuration

For those large project, we may find executable files with name like
`configure` under source folder. Before building the whole project, we need to
use it to configure build metadata first, which may includes library to use,
folder to put the build result.

## Build

Normally, a project utilizes a build tool like make. Additional configuration
may needed to finish building.

## Example

We want to build vim9. And there occurs two problem.

1. We do not have root permission, so we have to install it to user folder.
2. It relies on ncurses, but the ncurses we have is too stale.

To solve the first problem, we make use of the configure program `configure`
under `vim/src`.  It has a option formed `--prefix=PREFIX` which specify where
to install the build result. Vim is installed to root by default, which we do
not have permission.  So we can make use of this option to install vim to user
folder, for example, `$HOME/usr/local`.

To solve the second problem, we download the latest tarball file from ncurses
website and extract it. Like how we solve the first problem, we specify target
folder with option `--prefix=PREFIX`. After that we use the environment
variable `LDFLAGS` introduced in vim's configure program to explicitly make vim
use our new ncurses library to build.
