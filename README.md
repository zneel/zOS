# Learning kernel development

https://wiki.osdev.org/

---

## Step 1 - cross-compiler gcc:

https://wiki.osdev.org/Why_do_I_need_a_Cross_Compiler

- check current compiler target `gcc -dumpmachine` https://wiki.osdev.org/Target_Triplet
- create a cross-compiler (i686-elf) https://wiki.osdev.org/GCC_Cross-Compiler

Prepare the build: `sudo apt-get install build-essentials bison flex libgmp3-dev libmpc-dev libmpfr-dev texinfo`

Add cross-compiler env:

```sh
export PREFIX="$HOME/opt/cross"
export TARGET=i686-elf
export PATH="$PREFIX/bin:$PATH"
```

### binutils:

Create the `cross-compiler` folder:  
Download binutils: https://www.gnu.org/software/binutils/

```sh
mkdir cross-compiler
cd cross-compiler
git clone git://sourceware.org/git/binutils-gdb.git
mkdir build-binutils
cd build-binutils
../binutils-gdb/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror
make -j $(nproc)
make install
```

> This compiles the binutils (assembler, disassembler, and various other useful stuff), runnable on your system but handling code in the format specified by $TARGET.  
> `--disable-nls` tells binutils not to include native language support. This is basically optional, but reduces dependencies and compile time. It will also result in English-language diagnostics, which the people on the Forum understand when you ask your questions.  
> `--with-sysroot` tells binutils to enable sysroot support in the cross-compiler by pointing it to a default empty directory. By default, the linker refuses to use sysroots for no good technical reason, while gcc is able to handle both cases at runtime. This will be useful later on.

### gcc:

https://wiki.osdev.org/GCC

```sh
mkdir gcc
cd gcc
wget https://ftp.gnu.org/gnu/gcc/gcc-11.2.0/gcc-11.2.0.tar.gz
tar -xf gcc-11.2.0.tar.gz
# The $PREFIX/bin dir _must_ be in the PATH. We did that above.
which -- $TARGET-as || echo $TARGET-as is not in the PATH

mkdir build-gcc
cd build-gcc
../gcc-11.2.0/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --enable-languages=c,c++ --without-headers
make -j $(nproc) all-gcc
make -j $(nproc) all-target-libgcc
make -j $(nproc) install-gcc
make -j $(nproc) install-target-libgcc
```

> We build libgcc, a low-level support library that the compiler expects available at compile time. Linking against libgcc provides integer, floating point, decimal, stack unwinding (useful for exception handling) and other support functions. Note how we are not simply running make && make install as that would build way too much, not all components of gcc are ready to target your unfinished operating system.

> --disable-nls is the same as for binutils above.

> --without-headers tells GCC not to rely on any C library (standard or runtime) being present for the target.

> --enable-languages tells GCC not to compile all the other language frontends it supports, but only C (and optionally C++).

---

## Step 2 - x86 kernel:

- create simple x86 kernel
