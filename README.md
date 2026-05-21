# Unison for Termux

[Unison](https://github.com/bcpierce00/unison) is a great synchronization tool, but it's not available in the Termux package manager. To use it, you must build it from source.

This guide shows you how to install Unison on your Android device using Termux.
In the process, we will also install OCaml, which, to this date, is also not available in the Termux package manager.

Largely based on [Compiler Unison dans Termux](https://lunixite.nappey.org/compiler-unison-termux.html) by [jdn06](https://lunixite.nappey.org/author/jdn06.html).

## Prerequisites

Install required build tools:
```sh
pkg update -y
pkg install -y binutils build-essential clang make git curl unzip libandroid-shmem
```

If running in a simulator (such as waydroid), you might need `ndk-multilib` as well.

## Building OCaml

Unison is built using OCaml, so we will have to install it first. This process compiles OCaml specifically for the Android architecture.

Notes:
- Unison requires Ocaml 4.08 at minimum.
  - 4.13.0 to 5.3.0 (should) work
  - 5.4.0 and 5.4.1 do not work
  - 5.5.0-beta1 (latest) works
- The android api version must be at least 28. ([more](https://github.com/omeyenburg/unison-for-termux/issues/1#issue-3927661941))
- Higher android api versions seem to have issues on aarch64 devices. ([more](https://github.com/omeyenburg/unison-for-termux/issues/1#issuecomment-4428109622))

Credits:
- [terminatorbs](https://github.com/terminatorbs): noted issues with api version and missing dependency
- [engdyn](https://github.com/engdyn): noted issues with api versions above 28 and problematic ocaml versions

```sh
OCAML_VERSION=5.3.0
TARGET="$(uname -m)-unknown-linux-android"
API=28

mkdir -p $HOME/tmp
curl -L https://github.com/ocaml/ocaml/archive/refs/tags/${OCAML_VERSION}.tar.gz \
  -o "$HOME/tmp/ocaml.tar.gz"
tar xzf "$HOME/tmp/ocaml.tar.gz" -C "$HOME/tmp"
cd "$HOME/tmp/ocaml-${OCAML_VERSION}"

# Configure OCaml for Termux/Android
# Termux provides the $PREFIX variable.
./configure --prefix=$PREFIX --disable-warn-error --without-afl CC="clang --target=${TARGET}${API}" LDFLAGS="-landroid-shmem"

# Build and install OCaml
make world
make install
```

Check whether ocaml was correctly installed:
```sh
ocaml --version
ocamlc --version
```

If you run into issues:
- "Undeclared function 'getentropy'": See [this solution](https://github.com/omeyenburg/unison-for-termux/issues/1) by [terminatorbs](https://github.com/terminatorbs).  
  Add a flag like `--target=aarch64-unknown-linux-android28` to `./configure` where 28 is the android api version you want to target, must be at least 28.

## Building Unison

Now that OCaml is installed, you can compile Unison. This guide uses Unison 2.53.7.

```sh
UNISON_VERSION=2.53.7

mkdir -p $HOME/tmp
curl -L https://github.com/bcpierce00/unison/archive/refs/tags/v${UNISON_VERSION}.tar.gz \
  -o "$HOME/tmp/unison.tar.gz"
tar xzf "$HOME/tmp/unison.tar.gz" -C "$HOME/tmp"
cd "$HOME/tmp/unison-${UNISON_VERSION}"

# Build and install Unison
# NATIVE=false tells the build system to use the OCaml bytecode compiler,
# which makes large syncs slower but is necessary for Termux.
make NATIVE=false
make NATIVE=false install
```

Unison should be installed now!

You can test it by checking the version:
```sh
unison -version
```

## Usage

If you encounter errors while running unison, try to use the options `-ignorelocks` and possibly `-perms=0` (or `-fat`, which includes `-perms=0`) as well.
Though both have downsides: With `-ignorelocks`, you must be careful not to start multiple sync jobs at once.
When using `-perms=0`, no permissions will be synchronized. More information in the [Unison Manual](https://man.archlinux.org/man/unison.1.en).
On a rooted device, there are ways to use Unison without these flags.

## Cleanup

Remove temporary build files:
```sh
cd
rm -rf $HOME/tmp/ocaml-${OCAML_VERSION} $HOME/tmp/unison-${UNISON_VERSION} $HOME/tmp/*.tar.gz
```

## Uninstall

OCaml:
```sh
rm -f $PREFIX/lib/ocaml*
rm -rf $PREFIX/lib/ocaml $PREFIX/share/man/man1/ocaml*
```

Unison:
```sh
rm -f $PREFIX/bin/unison* $PREFIX/share/man/man1/unison.1
```
