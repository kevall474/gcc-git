#_                   _ _ _  _ _____ _  _
#| | _______   ____ _| | | || |___  | || |
#| |/ / _ \ \ / / _` | | | || |_ / /| || |_
#|   <  __/\ V / (_| | | |__   _/ / |__   _|
#|_|\_\___| \_/ \__,_|_|_|  |_|/_/     |_|


#Maintainer: kevall474 <kevall474@tuta.io> <https://github.com/kevall474>
#Credits: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
#Credits: Allan McRae <allan@archlinux.org>
#Credits: Daniel Kozak <kozzi11@gmail.com>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc
# NOTE: libtool requires rebuilt with each new gcc version

pkgname=('gcc-git' 'gcc-libs-git' 'gcc-fortran-git' 'gcc-objc-git' 'gcc-ada-git' 'gcc-go-git' 'lib32-gcc-libs-git' 'gcc-d-git')
pkgver=12.0.0
_majorver=${pkgver%%.*}
_islver=0.24
pkgrel=1
pkgdesc='The GNU Compiler Collection'
arch=(x86_64)
license=(GPL LGPL FDL custom)
url='https://gcc.gnu.org'
makedepends=('binutils' 'libmpc' 'gcc-ada' 'doxygen' 'lib32-glibc' 'lib32-gcc-libs' 'python' 'git')
checkdepends=(dejagnu inetutils)
options=(!emptydirs !buildflags)
_libdir=usr/lib/gcc/$CHOST/12.0.0
source=(#'git+https://gcc.gnu.org/git/gcc.git'
        "http://isl.gforge.inria.fr/isl-${_islver}.tar.xz"
        "c89"
        "c99")
md5sums=(#'SKIP'
         'SKIP'
         'SKIP'
         'SKIP')

pkgver(){
 cd gcc
 #echo $(cat gcc/BASE-VER).r$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
 echo $(cat gcc/BASE-VER).$(date -I | sed 's/-/_/' | sed 's/-/_/').$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
}

prepare(){

  git clone --depth=1 https://gcc.gnu.org/git/gcc.git

  cd gcc

  # link isl for in-tree build
  ln -s ../isl-${_islver} isl

  # Do not run fixincludes
  sed -i 's@\./fixinc\.sh@-c true@' gcc/Makefile.in

  # Arch Linux installs x86_64 libraries /lib
  sed -i '/m64=/s/lib64/lib/' gcc/config/i386/t-linux64

  # hack! - some configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" {libiberty,gcc}/configure

  mkdir -p "$srcdir/gcc-build"
}

build() {
  cd gcc-build

  # using -pipe causes spurious test-suite failures
  # http://gcc.gnu.org/bugzilla/show_bug.cgi?id=48565
  CFLAGS=${CFLAGS/-pipe/}
  CXXFLAGS=${CXXFLAGS/-pipe/}

  "$srcdir/gcc/configure" --prefix=/usr \
      --libdir=/usr/lib \
      --libexecdir=/usr/lib \
      --mandir=/usr/share/man \
      --infodir=/usr/share/info \
      --with-bugurl=https://bugs.archlinux.org/ \
      --enable-languages=c,c++,ada,fortran,go,lto,objc,obj-c++,d \
      --with-isl \
      --with-linker-hash-style=gnu \
      --with-system-zlib \
      --enable-__cxa_atexit \
      --enable-cet=auto \
      --enable-checking=release \
      --enable-clocale=gnu \
      --enable-default-pie \
      --enable-default-ssp \
      --enable-gnu-indirect-function \
      --enable-gnu-unique-object \
      --enable-install-libiberty \
      --enable-linker-build-id \
      --enable-lto \
      --enable-multilib \
      --enable-plugin \
      --enable-shared \
      --enable-threads=posix \
      --disable-libssp \
      --disable-libstdcxx-pch \
      --disable-libunwind-exceptions \
      --disable-werror \
      gdc_include_dir=/usr/include/dlang/gdc

  make -j$(nproc)

  # make documentation
  make -j$(nproc) -C $CHOST/libstdc++-v3/doc doc-man-doxygen
}

#check() {
#  cd gcc-build
#
#  # do not abort on error as some are "expected"
#  make -j$(nproc) -k check || true
#  "$srcdir/gcc/contrib/test_summary"
#}

package_gcc-libs-git() {
  pkgdesc='Runtime libraries shipped by GCC'
  depends=('glibc>=2.27')
  options+=(!strip)
  provides=(gcc-libs-multilib=${pkgver}-${pkgrel} libgo.so=${pkgver}-${pkgrel} libgfortran.so=${pkgver}-${pkgrel} libgphobos.so=${pkgver}-${pkgrel}
            libubsan.so=${pkgver}-${pkgrel} libasan.so=${pkgver}-${pkgrel} libtsan.so=${pkgver}-${pkgrel} liblsan.so=${pkgver}-${pkgrel} gcc-libs=${pkgver}-${pkgrel})
  conflicts=(gcc-libs)

  cd gcc-build
  make -j$(nproc) -C $CHOST/libgcc DESTDIR="$pkgdir" install-shared
  rm -f "$pkgdir/$_libdir/libgcc_eh.a"

  for lib in libatomic \
             libgfortran \
             libgo \
             libgomp \
             libitm \
             libquadmath \
             libsanitizer/{a,l,ub,t}san \
             libstdc++-v3/src \
             libvtv; do
    make -j$(nproc) -C $CHOST/$lib DESTDIR="$pkgdir" install-toolexeclibLTLIBRARIES
  done

  make -j$(nproc) -C $CHOST/libobjc DESTDIR="$pkgdir" install-libs
  make -j$(nproc) -C $CHOST/libstdc++-v3/po DESTDIR="$pkgdir" install

  make -j$(nproc) -C $CHOST/libphobos DESTDIR="$pkgdir" install
  rm -rf "$pkgdir"/$_libdir/include/d/
  rm -f "$pkgdir"/usr/lib/libgphobos.spec

  for lib in libgomp \
             libitm \
             libquadmath; do
    make -j$(nproc) -C $CHOST/$lib DESTDIR="$pkgdir" install-info
  done

  # remove files provided by lib32-gcc-libs-gitb
  rm -rf "$pkgdir"/usr/lib32/

  # Install Runtime Library Exception
  install -Dm644 "$srcdir/gcc/COPYING.RUNTIME" \
    "$pkgdir/usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION"
}

package_gcc-git() {
  pkgdesc="The GNU Compiler Collection - C and C++ frontends"
  depends=("gcc-libs=$pkgver-$pkgrel" 'binutils>=2.28' libmpc)
  groups=('base-devel')
  optdepends=('lib32-gcc-libs: for generating code for 32-bit ABI')
  provides=(gcc-multilib=${pkgver}-${pkgrel} gcc=${pkgver}-${pkgrel})
  conflicts=(gcc)
  options+=(staticlibs)

  cd gcc-build

  make -j$(nproc) -C gcc DESTDIR="$pkgdir" install-driver \
                                           install-cpp \
                                           install-gcc-ar \
                                           c++.install-common \
                                           install-headers \
                                           install-plugin \
                                           install-lto-wrapper

  install -m755 -t "$pkgdir/usr/bin/" gcc/gcov{,-tool}
  install -m755 -t "$pkgdir/${_libdir}/" gcc/{cc1,cc1plus,collect2,lto1}

  make -j$(nproc) -C $CHOST/libgcc DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/32/libgcc DESTDIR="$pkgdir" install
  rm -f "$pkgdir"/usr/lib{,32}/libgcc_s.so*

  make -j$(nproc) -C $CHOST/libstdc++-v3/src DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/libstdc++-v3/include DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/libstdc++-v3/libsupc++ DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/libstdc++-v3/python DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/32/libstdc++-v3/src DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/32/libstdc++-v3/include DESTDIR="$pkgdir" install
  make -j$(nproc) -C $CHOST/32/libstdc++-v3/libsupc++ DESTDIR="$pkgdir" install

  make -j$(nproc) DESTDIR="$pkgdir" install-libcc1
  install -d "$pkgdir/usr/share/gdb/auto-load/usr/lib"
  mv "$pkgdir"/usr/lib/libstdc++.so.6.*-gdb.py "$pkgdir/usr/share/gdb/auto-load/usr/lib/"
  rm "$pkgdir"/usr/lib{,32}/libstdc++.so*

  make -j$(nproc) DESTDIR="$pkgdir" install-fixincludes
  make -j$(nproc) -C gcc DESTDIR="$pkgdir" install-mkheaders

  make -j$(nproc) -C lto-plugin DESTDIR="$pkgdir" install
  install -dm755 "$pkgdir"/usr/lib/bfd-plugins/
  ln -s /${_libdir}/liblto_plugin.so "$pkgdir/usr/lib/bfd-plugins/"

  make -j$(nproc) -C $CHOST/libgomp DESTDIR="$pkgdir" install-nodist_{libsubinclude,toolexeclib}HEADERS
  make -j$(nproc) -C $CHOST/libitm DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -j$(nproc) -C $CHOST/libquadmath DESTDIR="$pkgdir" install-nodist_libsubincludeHEADERS
  make -j$(nproc) -C $CHOST/libsanitizer DESTDIR="$pkgdir" install-nodist_{saninclude,toolexeclib}HEADERS
  make -j$(nproc) -C $CHOST/libsanitizer/asan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -j$(nproc) -C $CHOST/libsanitizer/tsan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -j$(nproc) -C $CHOST/libsanitizer/lsan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -j$(nproc) -C $CHOST/32/libgomp DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -j$(nproc) -C $CHOST/32/libitm DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS
  make -j$(nproc) -C $CHOST/32/libsanitizer DESTDIR="$pkgdir" install-nodist_{saninclude,toolexeclib}HEADERS
  make -j$(nproc) -C $CHOST/32/libsanitizer/asan DESTDIR="$pkgdir" install-nodist_toolexeclibHEADERS

  make -j$(nproc) -C libiberty DESTDIR="$pkgdir" install
  install -m644 libiberty/pic/libiberty.a "$pkgdir/usr/lib"

  make -j$(nproc) -C gcc DESTDIR="$pkgdir" install-man install-info
  rm "$pkgdir"/usr/share/man/man1/{gccgo,gfortran,gdc}.1
  rm "$pkgdir"/usr/share/info/{gccgo,gfortran,gnat-style,gnat_rm,gnat_ugn,gdc}.info

  make -j$(nproc) -C libcpp DESTDIR="$pkgdir" install
  make -j$(nproc) -C gcc DESTDIR="$pkgdir" install-po

  # many packages expect this symlink
  ln -s gcc "$pkgdir"/usr/bin/cc

  # POSIX conformance launcher scripts for c89 and c99
  install -Dm755 "$srcdir/c89" "$pkgdir/usr/bin/c89"
  install -Dm755 "$srcdir/c99" "$pkgdir/usr/bin/c99"

  # install the libstdc++ man pages
  make -j$(nproc) -C $CHOST/libstdc++-v3/doc DESTDIR="$pkgdir" doc-install-man

  # remove files provided by lib32-gcc-libs-gitb
  rm -f "$pkgdir"/usr/lib32/lib{stdc++,gcc_s}.so

  # byte-compile python libraries
  python -m compileall "$pkgdir/usr/share/gcc-${pkgver%%+*}/"
  python -O -m compileall "$pkgdir/usr/share/gcc-${pkgver%%+*}/"

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gcc-fortran-git() {
  pkgdesc='Fortran front-end for GCC'
  depends=("gcc=$pkgver-$pkgrel")
  provides=(gcc-fortran-multilib=${pkgver}-${pkgrel} gcc-fortran=${pkgver}-${pkgrel})
  conflicts=(gcc-fortran)

  cd gcc-build
  make -j$(nproc) -C $CHOST/libgfortran DESTDIR="$pkgdir" install-cafexeclibLTLIBRARIES install-{toolexeclibDATA,nodist_fincludeHEADERS,gfor_cHEADERS}
  make -j$(nproc) -C $CHOST/32/libgfortran DESTDIR=$pkgdir install-cafexeclibLTLIBRARIES install-{toolexeclibDATA,nodist_fincludeHEADERS,gfor_cHEADERS}
  make -j$(nproc) -C $CHOST/libgomp DESTDIR="$pkgdir" install-nodist_fincludeHEADERS
  make -j$(nproc) -C gcc DESTDIR="$pkgdir" fortran.install-{common,man,info}
  install -Dm755 gcc/f951 "$pkgdir/${_libdir}/f951"

  ln -s gfortran "$pkgdir/usr/bin/f95"

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gcc-objc-git() {
  pkgdesc='Objective-C front-end for GCC'
  depends=("gcc=$pkgver-$pkgrel")
  provides=(gcc-objc-multilib=${pkgver}-${pkgrel} gcc-objc=${pkgver}-${pkgrel})
  conflicts=(gcc-objc)

  cd gcc-build
  make -j$(nproc) DESTDIR="$pkgdir" -C $CHOST/libobjc install-headers
  install -dm755 "$pkgdir/${_libdir}"
  install -m755 gcc/cc1obj{,plus} "$pkgdir/${_libdir}/"

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gcc-ada-git() {
  pkgdesc='Ada front-end for GCC (GNAT)'
  depends=("gcc=$pkgver-$pkgrel")
  provides=(gcc-ada-multilib=${pkgver}-${pkgrel} gcc-ada=${pkgver}-${pkgrel})
  conflicts=(gcc-ada)
  options+=(staticlibs)

  cd gcc-build/gcc
  make -j$(nproc) DESTDIR="$pkgdir" ada.install-{common,info}
  install -m755 gnat1 "$pkgdir/${_libdir}"

  cd "$srcdir"/gcc-build/$CHOST/libada
  make -j$(nproc) DESTDIR=${pkgdir} INSTALL="install" INSTALL_DATA="install -m644" install-libada

  cd "$srcdir"/gcc-build/$CHOST/32/libada
  make -j$(nproc) DESTDIR=${pkgdir} INSTALL="install" INSTALL_DATA="install -m644" install-libada

  ln -s gcc "$pkgdir/usr/bin/gnatgcc"

  # insist on dynamic linking, but keep static libraries because gnatmake complains
  mv "$pkgdir"/${_libdir}/adalib/libgna{rl,t}-${_majorver}.so "$pkgdir/usr/lib"
  ln -s libgnarl-${_majorver}.so "$pkgdir/usr/lib/libgnarl.so"
  ln -s libgnat-${_majorver}.so "$pkgdir/usr/lib/libgnat.so"
  rm -f "$pkgdir"/${_libdir}/adalib/libgna{rl,t}.so

  install -d "$pkgdir/usr/lib32/"
  mv "$pkgdir"/${_libdir}/32/adalib/libgna{rl,t}-${_majorver}.so "$pkgdir/usr/lib32"
  ln -s libgnarl-${_majorver}.so "$pkgdir/usr/lib32/libgnarl.so"
  ln -s libgnat-${_majorver}.so "$pkgdir/usr/lib32/libgnat.so"
  rm -f "$pkgdir"/${_libdir}/32/adalib/libgna{rl,t}.so

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gcc-go-git() {
  pkgdesc='Go front-end for GCC'
  depends=("gcc=$pkgver-$pkgrel")
  provides=("go=1.12.2" gcc-go-multilib=${pkgver}-${pkgrel} gcc-go=${pkgver}-${pkgrel})
  conflicts=(go gcc-go)

  cd gcc-build
  make -j$(nproc) -C $CHOST/libgo DESTDIR="$pkgdir" install-exec-am
  make -j$(nproc) -C $CHOST/32/libgo DESTDIR="$pkgdir" install-exec-am
  make -j$(nproc) DESTDIR="$pkgdir" install-gotools
  make -j$(nproc) -C gcc DESTDIR="$pkgdir" go.install-{common,man,info}

  rm -f "$pkgdir"/usr/lib{,32}/libgo.so*
  install -Dm755 gcc/go1 "$pkgdir/${_libdir}/go1"

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION "$pkgdir/usr/share/licenses/$pkgname/"
}

package_lib32-gcc-libs-git() {
  pkgdesc='32-bit runtime libraries shipped by GCC'
  depends=('lib32-glibc>=2.27')
  provides=(libgo.so=${pkgver}-${pkgrel} libgfortran.so=${pkgver}-${pkgrel} libubsan.so=${pkgver}-${pkgrel} libasan.so=${pkgver}-${pkgrel} lib32-gcc-libs=${pkgver}-${pkgrel})
  conflicts=(lib32-gcc-libs)
  options=(!emptydirs !strip)

  cd gcc-build

  make -j$(nproc) -C $CHOST/32/libgcc DESTDIR="$pkgdir" install-shared
  rm -f "$pkgdir/$_libdir/32/libgcc_eh.a"

  for lib in libatomic \
             libgfortran \
             libgo \
             libgomp \
             libitm \
             libquadmath \
             libsanitizer/{a,l,ub}san \
             libstdc++-v3/src \
             libvtv; do
    make -C $CHOST/32/$lib DESTDIR="$pkgdir" install-toolexeclibLTLIBRARIES
  done

  make -j$(nproc) -C $CHOST/32/libobjc DESTDIR="$pkgdir" install-libs

  make -j$(nproc) -C $CHOST/libphobos DESTDIR="$pkgdir" install
  rm -f "$pkgdir"/usr/lib32/libgphobos.spec

  # remove files provided by gcc-libs-git
  rm -rf "$pkgdir"/usr/lib

  # Install Runtime Library Exception
  install -Dm644 "$srcdir/gcc/COPYING.RUNTIME" "$pkgdir/usr/share/licenses/lib32-gcc-libs/RUNTIME.LIBRARY.EXCEPTION"
}

package_gcc-d-git() {
  pkgdesc="D frontend for GCC"
  depends=("gcc=$pkgver-$pkgrel")
  provides=(gdc=${pkgver}-${pkgrel} gcc-d=${pkgver}-${pkgrel})
  conflicts=(gcc-d)
  options=('staticlibs')

  cd gcc-build
  make -j$(nproc) -C gcc DESTDIR="$pkgdir" d.install-{common,man,info}

  install -Dm755 gcc/gdc "$pkgdir"/usr/bin/gdc
  install -Dm755 gcc/d21 "$pkgdir"/"$_libdir"/d21

  make -j$(nproc) -C $CHOST/libphobos DESTDIR="$pkgdir" install
  rm -f "$pkgdir/usr/lib/"lib{gphobos,gdruntime}.so*
  rm -f "$pkgdir/usr/lib32/"lib{gphobos,gdruntime}.so*

  install -d "$pkgdir"/usr/include/dlang
  ln -s /"${_libdir}"/include/d "$pkgdir"/usr/include/dlang/gdc

  # Install Runtime Library Exception
  install -d "$pkgdir/usr/share/licenses/$pkgname/"
  ln -s /usr/share/licenses/gcc-libs/RUNTIME.LIBRARY.EXCEPTION "$pkgdir/usr/share/licenses/$pkgname/"
}
