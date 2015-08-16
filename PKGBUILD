pkgname=ncurses-git
pkgver=5.9.r199.g8d00601
pkgrel=1

pkgdesc='Unofficial git mirror of snapshots from ftp://invisible-island.net/ncurses/current/'
url='http://ncurses.scripts.mit.edu/?p=ncurses.git'
arch=('i686' 'x86_64')
license=('MIT')

depends=('glibc')
makedepends=('git')

provides=('ncurses')
conflicts=('ncurses')

source=('git://ncurses.scripts.mit.edu/ncurses.git')

md5sums=('SKIP')

pkgver() {
    cd ncurses
    git describe --tags | sed 's/^v//; s/-/.r/; s/-/./'
}

prepare() {
    mkdir ncurses-build
    mkdir ncursesw-build
}

build() {
    cd ncursesw-build

    # Add --enable-ext-colors and --enable-ext-mouse with next soname bump.
    ../ncurses/configure --prefix=/usr \
        --with-shared \
        --with-normal \
        --without-ada  \
        --enable-widec \
        --without-debug \
        --enable-pc-files \
        --with-cxx-shared \
        --with-cxx-binding \
        --with-pkg-config-libdir=/usr/lib/pkgconfig
    make

    # Libraries for external binary support.
    cd "$srcdir"/ncurses-build
    if [[ "$CARCH" = x86_64 ]]; then
        config_chtype_long='--with-chtype=long'
    fi
   
    ../ncurses/configure --prefix=/usr \
        --with-shared \
        --with-normal \
        --without-ada \
        --without-debug \
        --with-cxx-share \
        "$config_chtype_long" \
        --with-cxx-binding
    make
}

package() {
    cd ncursesw-build
    make DESTDIR="$pkgdir" install

    # Fool packages looking to link to non-wide-character ncurses libraries.
    for lib in ncurses form panel menu; do
        printf 'INPUT(-l%sw)\n' "$lib" > "$pkgdir"/usr/lib/lib"$lib".so
    done

    for lib in ncurses ncurses++ form panel menu; do
        ln -s "$lib"w.pc "$pkgdir"/usr/lib/pkgconfig/"$lib".pc
    done

    # Some packages look for -lcurses during build.
    printf 'INPUT(-lncursesw)\n' > "$pkgdir"/usr/lib/libcursesw.so
    ln -s libncurses.so "$pkgdir"/usr/lib/libcurses.so

    # Non-widec compatibility libraries.
    cd "$srcdir"/ncurses-build

    for lib in ncurses form panel menu; do
        install -Dm755 lib/lib"$lib".so.5.9 "$pkgdir"/usr/lib/lib"$lib".so.5.9
        ln -s lib"$lib".so.5.9 "$pkgdir"/usr/lib/lib"$lib".so.5
    done

    cd "$srcdir"/ncurses
    install -Dm755 COPYING "$pkgdir"/usr/share/licenses/ncurses/COPYING
} 
