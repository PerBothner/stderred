# stderred

stderr in red.

## About

stderred hooks on write() and a family of stream functions (fwrite, fprintf,
error...) from libc in order to colorize all stderr output that goes to
terminal thus making it distinguishable from stdout.  Basically it wraps text
that goes to file with descriptor "2" with proper ANSI escape codes making text
red.

It's implemented as a shared library and doesn't require recompilation of
existing binaries thanks to _preload/insert_ feature of dynamic linkers.

It's supported on Linux (with `LD_PRELOAD`), FreeBSD (also `LD_PRELOAD`) and
OSX (with `DYLD_INSERT_LIBRARIES`).

## Installation

### Ubuntu (64-bit)

Download latest .deb package from https://github.com/sickill/stderred/downloads
and install it with:

    sudo dpkg -i stderred_1.0_amd64.deb

Add following line to your .bashrc/.zshrc:

    [[ -s /usr/share/stderred/stderred.sh ]] && source /usr/share/stderred/stderred.sh

### Fedora (64-bit)

Download latest .rpm package from https://github.com/sickill/stderred/downloads
and install it with:

    sudo rpm -i stderred-1.0-1.x86_64.rpm

Add following line to your .bashrc/.zshrc:

    [[ -s /usr/share/stderred/stderred.sh ]] && source /usr/share/stderred/stderred.sh

### From source

Clone the repository:

    $ git clone git://github.com/sickill/stderred.git
    $ cd stderred

Important: In all cases below make sure that path to `libstderred.so` is
absolute!

#### Linux and FreeBSD

First, make sure you have basic gcc toolchain installed. gcc, make and cmake
should be enough.

Build:

    $ make

In your shell's config file export `LD_PRELOAD` variable pointing to built
shared library. Put following in your .bashrc/.zshrc:

    export LD_PRELOAD="/absolute/path/to/stderred/build/libstderred.so${LD_PRELOAD:+:$LD_PRELOAD}"

##### Note for 64-bit Linux and FreeBSD

With some Linux distros you can install 32-bit packages on 64-bit system.  Shared
libraries compiled for 64-bit doesn't work with 32-bit binaries though. It
happens that 64-bit binaries call 32-bit ones resulting in warning message
printed to terminal about not compatible `LD_PRELOAD` shared lib.

Fortunately Linux's dynamic linker has a feature called Dynamic String Token
(DST). It allows dynamic substitution of `$LIB` token in `LD_PRELOAD` variable
with "lib" or "lib64" respectively for 32 and 64-bit binaries when the binary
is being run. Thanks to that you can compile stderred for both architectures
and automatically use proper version of this shared library.

First install libc development headers for both architectures and then compile
it like this:

    $ make 32 && make 64

and export `LD_PRELOAD` like this in your shell's config:

    export LD_PRELOAD="/path/to/stderred/\$LIB/libstderred.so${LD_PRELOAD:+:$LD_PRELOAD}"

_\* Note that [there is no support for $LIB token on Ubuntu](http://comments.gmane.org/gmane.comp.lib.glibc.user/974)._

#### OSX

    $ make

Export `DYLD_INSERT_LIBRARIES` variable in your shell's config file by putting following
in your .bashrc/.zshrc:

    export DYLD_INSERT_LIBRARIES="/absolute/path/to/build/libstderred.dylib${DYLD_INSERT_LIBRARIES:+:$DYLD_INSERT_LIBRARIES}"

##### Universal lib on OSX

OSX solves multi-arch problem (described above in "Multi-arch Linux and
FreeBSD") by supporting so called "universal" libraries that include 2 copies
of code compiled for both 32 and 64-bit architecture in the single library
file.

If you feel you will want universal library then build it this way:

    $ make universal

and export shell env like above.

#### Aliasing instead of exporting LD_PRELOAD

By exporting `LD_PRELOAD` env variable you're enabling stderred for all
commands. If you prefer to enable it on ad-hoc (per command) basis instead then
don't export `LD_PRELOAD` but create an alias like this:

    $ alias stderred="LD_PRELOAD=/absolute/path/to/build/libstderred.so\${LD_PRELOAD:+:\$LD_PRELOAD}"
    $ stderred java lol

This way you can selectively colorize stderr for the commands you run.

## Checking if it works

    $ find -q
    $ cat nonexistingfile
    $ ls nonexistingfile
    $ python -c 'import os; print "Yo!"; os.write(2, "Jola\n\r")'
    $ STDERRED_ESC_CODE=$(echo -e '\e[;92m') ruby -e 'puts "Yo!"; warn "Jola"'

Jola should be in a red and green dress.

![stderred in action](https://github.com/downloads/cehoffman/stderred/stderred.png)

## Configuration

### Custom color code

If you prefer other color or you want to use additional escape codes
(for bold/bright, italic, different background) you can export
`STDERRED_ESC_CODE` with desired escape code sequence.

Here's an example for bold red:

    export STDERRED_ESC_CODE=`echo -e "\e[1;31m"`

### Program Blacklisting

Some programs abuse stderr and print stuff on it even when they really
shouldn't.  If you want to turn off stderred for particular programs you can
export `STDERRED_BLACKLIST` with a desired POSIX Extended Regular Expression
matching names of these programs.

Here's an example that will blacklist bash, and all programs with names
starting with "test":

    export STDERRED_BLACKLIST="^(bash|test.*)$"

## Alternative implementations

Simpler and much less reliable solution when using Zsh is to use named pipes
trick proposed on
[Gentoo Linux wiki](http://en.gentoo-wiki.com/wiki/Zsh#Colorize_STDERR).
It has some race condition/buffering issues and breaks on interactive commands
writing to stderr though.

## Authors

Current implementation:

* [Marcin Kulik](https://github.com/sickill/)
* [Chris Hoffman](https://github.com/cehoffman/)
* [Brian Tarricone](https://github.com/kelnos/)

[Original concept](http://www.asheesh.org/note/software/stderred.html) and
[initial implementation](http://git.asheesh.org/?p=zzz/colorize-stderr.git;a=summary):

* Asheesh Laroia

## License

You are free to use this program under the terms of the license found in
LICENSE file.
