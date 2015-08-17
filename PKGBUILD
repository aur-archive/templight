# Maintainer: Martin Schulze <mail dot martin dot schulze at gmx dot net>
pkgname=templight
pkgver=Jan14
pkgrel=2
pkgdesc="C++ Template Debugger/Profiler patch for clang 3.5 (rev199501)"
url="http://plc.inf.elte.hu/templight/"
arch=('i686' 'x86_64')
license=('UIUC')
depends=('ocaml' 'swig' 'libedit')
makedepends=('subversion' 'cmake' 'python2' 'libffi')
provides=('templight')
optdepends=('templar-schulmar-git')
conflicts=()

source=("llvm"::'svn+http://llvm.org/svn/llvm-project/llvm/trunk#revision=199501'
	"llvm/tools/clang"::'svn+http://llvm.org/svn/llvm-project/cfe/trunk#revision=199501'
	"llvm/tools/clang/tools/extra"::'svn+http://llvm.org/svn/llvm-project/clang-tools-extra/trunk#revision=199501'
	"llvm/projects/compiler-rt"::'svn+http://llvm.org/svn/llvm-project/compiler-rt/trunk#revision=199501'
	'http://plc.inf.elte.hu/templight/templight-20140122.tar.gz'
)
md5sums=('SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'd4fbaeed8ac58874078eb40eba634553'
)

if [ ${CARCH} == x86_64 ]; then
    _target=$CARCH
else
    _target=x86
fi

_prefix=/usr/opt/templight

prepare() {
    cd "$srcdir"
    mkdir -p build
    cd build
    msg "Entered `pwd`"
    _python2bin="$srcdir/python2bin"
    msg "Building Python2 bin '$_python2bin'"
    mkdir -p "$_python2bin"
    cd "$_python2bin"
    ln -fs /usr/bin/python2 python
    ln -fs /usr/bin/python2-config python-config
    export PATH="$_python2bin/:${PATH}"
    cd "$srcdir/build"

    # Include location of libffi headers in CPPFLAGS
    export CPPFLAGS="$CPPFLAGS $(pkg-config --cflags libffi)"

# remove "sample" from projects... it does not do anything useful
    msg "removing 'sample' project"
    rm -rf "$srcdir/$pkgname/projects/sample"
    pwd
    cd "$srcdir/llvm/tools/clang"
    patch -p0 -i "$srcdir/templight_clang34.diff"
    cd "$srcdir/llvm"
    
# several undocumented configure options that were used by someone else to build LLVM/Clang with clang and c++...
# those that fail are commented out again.
#build only enables host target.
    msg "Configuring build"

#    export CXX=clang++-templight
#    export CXXFLAGS=-templight

    ../llvm/configure \
    --prefix=$_prefix \
    --libdir=/usr/lib/llvm \
    --enable-cxx11 \
    --with-python="$_python2bin/python" \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-pic \
    --enable-polly \
    --enable-jit \
    --enable-libffi \
    --disable-libcpp \
    --enable-targets=host \
    --with-cxx-include-arch=$_target \
    --enable-ltdl-install \
    --disable-expensive-checks \
    --disable-debug-runtime \
    --disable-assertions \
    --enable-optimized \
    --enable-threads \
    --disable-multilib \
    --with-optimize-option=-O2 \
    --with-binutils-include=/usr/include

#    --with-clang \
#    --with-built-clang \
#    --disable-bootstrap \
#    --no-recursion \
#    --no-create \
#    --with-c-include-dirs=
# due to linking errors with libc++, I have currently explicitly disabled it.
# to test compiling with LLVM libc++. change --disable-libcpp to --enable-libcpp.

#fixing packaging problems with libc++ makefile
#essentially the same functions can be performed by toolchain.install
# sed -i 's/chown -R root:wheel $(HEADER_DIR)/#placeholder: chown -R broke packaging/b' $srcdir/llvm/projects/libcxx/Makefile 
}

build(){
    cd "$srcdir/llvm"
    msg "Building in `pwd` with path '${PATH}'"

    if [ ${MAKE_PARALLEL} ]; then
	make -j ${MAKE_PARALLEL}  REQUIRES_RTTI=1 || return 1
    else
	make REQUIRES_RTTI=1 || return 1
    fi
}

package() { 
    cd "$srcdir/llvm"
    make DESTDIR="$pkgdir" install || return 1 
    mkdir -p "$pkgdir/usr/bin"
    ln -s "$_prefix/bin/clang++" "$pkgdir/usr/bin/clang++-templight"
}
