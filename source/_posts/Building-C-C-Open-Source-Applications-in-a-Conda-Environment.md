---
title: Building C/C++ Applications in a Conda Environment
date: 2025-12-06
categories:
  - DevOps, SysAdmin
tags:
  - Reference
---

## Prerequisites

### Common Build Variables

#### Directly Used by `gcc`/`clang`

These environment variables are **automatically used by `gcc`/`clang` themselves**.

- `CPATH`
  - Colon-separated list of directories to search for headers before built-in include paths.
  - Like passing multiple `-I` options.
- `LIBRARY_PATH` 
  - Colon-separated list of directories to search for libraries (`.so`/`.a`) at **link time**.
  - Like passing multiple `-L` options.
  - **Doesn't affect how the executable finds shared libraries when running.**

#### For Build Systems (`make`, `cmake`, etc.)

These environment variables are not used by compilers **unless** your build tool (`make`, `cmake`, etc.) or script expands them.

- `CC` / `CXX`:
  - Which C/C++ compiler to use.
- `CFLAGS` / `CXXFLAGS`:
  - Extra flags for compiling C or C++ respectively, e.g., `-O2 -g -Wall`.
- `LDFLAGS`:
  - Extra flags when linking, such as `-L/path/to/lib`, `-lfoo`, `-Wl,-rpath,/my/libs`.

> 🚫 **Never stuff include/library/link flags inside `$CXX` or `$CC`. Always set them as separate variables, or pass them directly on the command line to the compiler.**

### C++ Libraries in Conda Environments

When you install a C++ library from `conda-forge`, it's placed inside your current environment, not in a system-wide directory.

- Headers: `$CONDA_PREFIX/include`
- Libraries: `$CONDA_PREFIX/lib`  

> Exception: The **C runtime** (`libc.so`, `libm.so`, etc.) always comes from the system, not Conda.

## Create a Conda Environment for C++ Compilation

```bash
conda install -c conda-forge clang clangxx libcxx libcxxabi libcxx-devel lld llvm-tools
```

- `libcxx`, `libcxxabi`, `libcxx-devel`: LLVM's C++ standard library and headers.

### Tell the Compiler Where to Look for Headers and Libraries

```bash
export CPATH="$CONDA_PREFIX/include"
export LIBRARY_PATH="$CONDA_PREFIX/lib"
```

This automatically adds Conda's include and lib directories to `clang`'s search paths.

### Set Common Build Variables for Build Systems

```bash
export CC="clang"
export CXX="clang++ -stdlib=libc++"
export LDFLAGS="-Wl,-rpath,$LIBRARY_PATH"
export AR="$CONDA_PREFIX/bin/llvm-ar"
export RANLIB="$CONDA_PREFIX/bin/llvm-ranlib"
export LD="$CONDA_PREFIX/bin/ld.lld"
```

- `LDFLAGS` sets an `rpath` such that the generated executable or shared library will hard-code `LIBRARY_PATH` as a shared library search path.

## Compiling Open Source Applications

### [autossh](https://github.com/Autossh/autossh)

Install openssh:

```bash
conda install -c conda-forge openssh
```

Then clone, compile, and install:

```bash
git clone https://github.com/Autossh/autossh.git
cd autossh
./configure --prefix="$CONDA_PREFIX" && make && make install
cd ..
```

### [busybox](https://busybox.net/)

Note that `busybox` does not use `configure`, only `make`.

```bash
make \
    CPATH="$CPATH" \
    LIBRARY_PATH="$LIBRARY_PATH" \
    CC="$CC" \
    CXX="$CXX" \
    LDFLAGS="$LDFLAGS" \
    AR="$AR" \
    RANLIB="$RANLIB" \
    LD="$LD" \
    defconfig
```

Edit `.config`: Change line `CONFIG_TC=y` to `CONFIG_TC=n`.

```bash
make \
    CPATH="$CPATH" \
    LIBRARY_PATH="$LIBRARY_PATH" \
    CC="$CC" \
    CXX="$CXX" \
    LDFLAGS="$LDFLAGS" \
    AR="$AR" \
    RANLIB="$RANLIB" \
    LD="$LD" \
    all
```

Now, the `busybox` binary is built.

### [enscript](https://www.gnu.org/software/enscript)

Install ghostscript:

```bash
conda install -c conda-forge ghostscript
```

Clone, compile, and install:

```bash
git clone git://git.savannah.gnu.org/enscript.git
cd enscript
# The makefiles hard code AR=ar. We would have to override that.
./configure --prefix="$CONDA_PREFIX" && make "AR=${AR}" && make install
cd ..
```

### [mlir](https://mlir.llvm.org/)

Install dependencies:

```bash
conda install -c conda-forge cmake git libxml2 ninja python zlib 
pip install ml_dtypes nanobind numpy pyyaml typing_extensions
```

Clone `llvm-project`:

```bash
git clone https://github.com/llvm/llvm-project.git
```

Build `llvm-project`:

```bash
mkdir -p llvm-project/build && cd llvm-project/build
cmake -G Ninja ../llvm \
    -DCMAKE_BUILD_TYPE=Release \
    -DLLVM_BUILD_EXAMPLES=ON \
    -DLLVM_ENABLE_ASSERTIONS=ON \
    -DLLVM_ENABLE_PROJECTS="mlir" \
    -DLLVM_INSTALL_UTILS=ON \
    -DLLVM_TARGETS_TO_BUILD="host" \
    -DMLIR_ENABLE_BINDINGS_PYTHON=ON \
    -DPython3_EXECUTABLE=$(which python) \
    && ninja
```

### [ocaml+dune](https://dune.build/)

First compile `ocaml`:

```bash
conda install -c conda-forge awk make sed zstd
git clone https://github.com/ocaml/ocaml.git
pushd ocaml
./configure --prefix="$CONDA_PREFIX"
make
make install
popd
```

Do not install `opam`, as `opam` is hard-coded to use `~/.opam` (even different binaries).

Install `dune`, which is not only a build system but also a dependency manager:

```bash
git clone https://github.com/ocaml/dune.git
pushd dune
PREFIX="$CONDA_PREFIX" make release
PREFIX="$CONDA_PREFIX" make install
popd
```

### [proot](https://github.com/proot-me/proot)

Install dependencies:

```bash
conda install -c conda-forge curl git libarchive pkg-config
```

Build and install `talloc`:

```bash
curl -O https://www.samba.org/ftp/talloc/talloc-2.4.3.tar.gz
tar -xvf talloc-2.4.3.tar.gz
pushd talloc-2.4.3
./configure --disable-python --prefix="$CONDA_PREFIX" && make && make install
popd
```

Install `uthash`:

```bash
git clone https://github.com/troydhanson/uthash
cp -rv uthash/include/ "${CONDA_PREFIX}"
```

Build and install `proot`:

```bash
git clone https://github.com/proot-me/proot.git
pushd proot
CC="$CC" LD="$LD" PKG_CONFIG_PATH="${CONDA_PREFIX}/lib/pkgconfig" make -C src loader.elf loader-m32.elf build.h
CC="$CC" LD="$LD" PKG_CONFIG_PATH="${CONDA_PREFIX}/lib/pkgconfig" make -C src proot care
CC="$CC" LD="$LD" PKG_CONFIG_PATH="${CONDA_PREFIX}/lib/pkgconfig" make -C test
cp src/proot src/care "${CONDA_PREFIX}/bin"
popd
```

### [qemu](https://www.qemu.org/)

```bash
conda install -c conda-forge cmake git meson ninja pkgconfig python
conda install -c conda-forge elfutils glib libcapstone libfdt libpng pixman zlib
pip install sphinx sphinx_rtd_theme
```

Download `qemu` source code and enter directory:

```bash
curl -O https://download.qemu.org/qemu-10.2.0-rc2.tar.xz
tar xvJf qemu-10.2.0-rc2.tar.xz
cd qemu-10.2.0-rc2
```

```bash
./configure \
  --disable-bsd-user \
  --disable-gtk \
  --disable-linux-user \
  --disable-rust \
  --disable-sdl \
  --disable-selinux \
  --disable-spice \
  --enable-curses \
  --enable-vnc \
  --prefix="$CONDA_PREFIX"
make
make install
```
