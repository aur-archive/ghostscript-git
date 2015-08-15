# Maintainer: Alessandro Pezzoni <alessandro_pezzoni@lavabit.com>
pkgname=ghostscript-git
pkgver=20130323
pkgrel=1
pkgdesc="An interpreter for the PostScript language"
arch=('i686' 'x86_64')
license=('AGPL' 'custom')
depends=('libxt' 'libcups' 'fontconfig' 'jasper' 'zlib' 'libpng>=1.5.7' 'libjpeg'
         'libtiff>=4.0.0' 'lcms' 'dbus')
makedepends=('gtk2' 'gnutls')
optdepends=('texlive-core:      needed for dvipdf'
            'gtk2:              needed for gsx')
url="http://www.ghostscript.com/"
provides=('ghostscript=9.07')
conflicts=('ghostscript')
options=('!libtool' '!makeflags')

_gitroot=git://git.ghostscript.com/ghostpdl.git
_gitname=ghostpdl

build() {
  cd "$srcdir"

  if [[ -d "$_gitname" ]]; then
    msg "Removing local files to make a shallow clone"
    rm -rf "$_gitname"
  fi

  msg "Connecting to GIT server...."
  git clone --depth=1 "$_gitroot" "$_gitname"

  msg "GIT checkout done or server timeout"
  msg "Starting build..."

  rm -rf "$srcdir/gs-build"
  # do not copy over the .git folder
  mkdir "$srcdir/gs-build"
  cd "$srcdir/$_gitname/gs" && ls -A | grep -v .git | xargs -d '\n' cp -r -t "$srcdir/gs-build"
  cd "$srcdir/gs-build"

  # Force it to use system-libs
  rm -rf jpeg libpng zlib jasper expat tiff lcms freetype 
  
  # Fix compilation error
  sed -i "s:AM_PROG_CC_STDC:AC_PROG_CC:g" configure.ac
  ./autogen.sh

  ./configure --prefix=/usr \
	--enable-dynamic \
	--with-ijs \
	--with-jbig2dec \
	--with-omni \
	--with-x \
	--with-drivers=ALL\
	--with-fontpath=/usr/share/fonts/Type1:/usr/share/fonts \
	--with-install-cups \
	--enable-fontconfig \
	--enable-freetype \
	--without-luratech \
	--without-omni \
	--with-system-libtiff \
	--disable-compile-inits #--help # needed for linking with system-zlib
  make

  # Build IJS
  cd ${srcdir}/gs-build/ijs
  sed -i "s:AM_PROG_CC_STDC:AC_PROG_CC:g" configure.ac
  ./autogen.sh
  ./configure --prefix=/usr --enable-shared --disable-static
  make
}

package() {
  cd "$srcdir/gs-build"
  make DESTDIR=${pkgdir} \
	cups_serverroot=${pkgdir}/etc/cups \
	cups_serverbin=${pkgdir}/usr/lib/cups install soinstall

  # install missing doc files # http://bugs.archlinux.org/task/18023
  mkdir -p ${pkgdir}/usr/share/ghostscript/doc/
  install -m 644 ${srcdir}/gs-build/doc/{Ps2ps2.htm,gs-vms.hlp,gsdoc.el,pscet_status.txt} ${pkgdir}/usr/share/ghostscript/doc/
  
  mkdir -p ${pkgdir}/usr/share/licenses/${pkgname}
  install -m644 LICENSE ${pkgdir}/usr/share/licenses/${pkgname}/

  # remove unwanted localized man-pages
  rm -rf $pkgdir/usr/share/man/[^man1]*

  # install IJS
  cd ${srcdir}/gs-build/ijs
  make DESTDIR=${pkgdir} install
}

# vim:set ts=2 sw=2 et:
