# Unison for Ttermux

[Unison](https://github.com/bcpierce00/unison) is a great synchronization tool, but it's not available in the Termux package manager. To use it, you must build it from source.

This guide shows you how to install Unison on your Android device using Termux.
In the process, we will also install OCaml, which, to this date, is also not available in the Termux package manager.

Largely based on [Compiler Unison dans Termux](https://lunixite.nappey.org/compiler-unison-termux.html) by [jdn06](https://lunixite.nappey.org/author/jdn06.html).

## Prerequisites

Install required build tools:
```sh
pkg update -y
pkg install -y build-essential clang make git curl unzip libandroid-shmem
```

## Building OCaml

Unison is built using OCaml, so we will have to install it first.
This guide uses OCaml 5.3.0, but other versions should also work. This process compiles OCaml specifically for the Android architecture.

```sh
OCAML_VERSION=5.3.0

mkdir -p $HOME/tmp
curl -L https://github.com/ocaml/ocaml/releases/download/${OCAML_VERSION}/ocaml-${OCAML_VERSION}.tar.gz \
  -o "$HOME/tmp/ocaml.tar.gz"
tar xzf "$HOME/tmp/ocaml.tar.gz" -C "$HOME/tmp"
cd "$HOME/tmp/ocaml-${OCAML_VERSION}"

# Configure OCaml for Termux/Android
# Termux provides the $PREFIX variable.
./configure --prefix=$PREFIX --disable-warn-error --without-afl LDFLAGS="-landroid-shmem"

# Build and install OCaml
make world
make install
```

## Building Unison

Now that OCaml is installed, you can compile Unison. This guide uses Unison 2.53.7.

```sh
UNISON_VERSION=2.53.7

mkdir -p $HOME/tmp
curl -L https://github.com/bcpierce00/unison/archive/v${UNISON_VERSION}.tar.gz \
  -o "$HOME/tmp/unison.tar.gz"
tar xzf "$HOME/tmp/unison.tar.gz" -C "$HOME/tmp"
cd "$HOME/tmp/unison-${UNISON_VERSION}"

# Build and install Unison
# NATIVE=false tells the build system to use the OCaml bytecode compiler.
make NATIVE=false
make NATIVE=false install
```

Unison should be installed now! 🎉

You can test it by checking the version:
```sh
unison -version
```

## Cleanup

Remove temporary build files:
```sh
cd
rm -rf $HOME/tmp/ocaml-${OCAML_VERSION} $HOME/tmp/unison-${UNISON_VERSION} $HOME/tmp/*.tar.gz
```

## Uninstallation

Uninstall OCaml:
```sh
rm -f $PREFIX/lib/ocaml*
rm -rf $PREFIX/lib/ocaml $PREFIX/share/man/man1/ocaml*
```

Uninstall Unison:
```sh
rm -f $PREFIX/bin/unison* $PREFIX/share/man/man1/unison.1
```
