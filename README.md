# Learning kernel development

https://wiki.osdev.org/

---

## Step 1:

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

Create the `cross-compiler` folder:  
Download binutils: https://www.gnu.org/software/binutils/

```sh
mkdir cross-compiler
cd cross-compiler
git clone git://sourceware.org/git/binutils-gdb.git
mkdir build-binutils
cd build-binutils
../binutils-gdb/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror
make
make install
```

---

## Step 2:

- create simple x86 kernel
