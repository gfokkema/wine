# Maintainer: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Eduardo Romero <eduardo@archlinux.org>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>

pkgname=wine
pkgver=10.2
pkgrel=2

_pkgbasever=${pkgver/rc/-rc}

source=("git+https://gitlab.winehq.org/wine/wine.git?signed#tag=wine-$_pkgbasever"
        fix-ptr-access.patch::https://gitlab.winehq.org/wine/wine/-/commit/05315ce3da4d6f04232611fb5dd6ffbd77f87ce7.patch
        30-win32-aliases.conf
        wine-binfmt.conf)
sha512sums=('2664d57860cd74706556bcf7e6ec48b4a7c49d8f22ed791bfc43caa50d43733a5a28c68a9d89842b9cce7b403d68dd21b9e328efc3349229cdeaf1f31188b51e'
            '547a1a31fcfa421e982da6032cdf398e5814d6f35c0670029d69853b89bde6dddfe47c514c2c5ff59d1d0f673c9a9df9adce88d199305f8ed2143ae8d65e2057'
            '6e54ece7ec7022b3c9d94ad64bdf1017338da16c618966e8baf398e6f18f80f7b0576edf1d1da47ed77b96d577e4cbb2bb0156b0b11c183a0accf22654b0a2bb'
            'bdde7ae015d8a98ba55e84b86dc05aca1d4f8de85be7e4bd6187054bfe4ac83b5a20538945b63fb073caab78022141e9545685e4e3698c97ff173cf30859e285')
validpgpkeys=(5AC1A08B03BD7A313E0A955AF5E6E9EEB9461DD7
              DA23579A74D4AD9AF9D3F945CEFAC8EAAF17519D)

pkgdesc="A compatibility layer for running Windows programs"
url="https://www.winehq.org"
arch=(x86_64)
options=(staticlibs !lto)
license=(LGPL-2.1-or-later)
depends=(
  desktop-file-utils
  fontconfig      lib32-fontconfig
  freetype2       lib32-freetype2
  gcc-libs        lib32-gcc-libs
  gettext         lib32-gettext
  libpcap         lib32-libpcap
  libunwind       lib32-libunwind
  libxcursor      lib32-libxcursor
  libxkbcommon    lib32-libxkbcommon
  libxi           lib32-libxi
  libxrandr       lib32-libxrandr
  wayland         lib32-wayland
)
makedepends=(autoconf bison perl flex mingw-w64-gcc
  git
  alsa-lib              lib32-alsa-lib
  gnutls                lib32-gnutls
  gst-plugins-base-libs lib32-gst-plugins-base-libs
  libcups               lib32-libcups
  libgphoto2
  libpulse              lib32-libpulse
  libxcomposite         lib32-libxcomposite
  libxinerama           lib32-libxinerama
  libxxf86vm            lib32-libxxf86vm
  mesa                  lib32-mesa
  mesa-libgl            lib32-mesa-libgl
  opencl-headers
  opencl-icd-loader     lib32-opencl-icd-loader
  pcsclite              lib32-pcsclite
  samba
  sane
  sdl2                  lib32-sdl2
  unixodbc
  v4l-utils             lib32-v4l-utils
  vulkan-headers
  vulkan-icd-loader     lib32-vulkan-icd-loader
)
optdepends=(
  alsa-lib              lib32-alsa-lib
  alsa-plugins          lib32-alsa-plugins
  cups                  lib32-libcups
  dosbox
  gnutls                lib32-gnutls
  gst-plugins-bad
  gst-plugins-base      lib32-gst-plugins-base
  gst-plugins-base-libs lib32-gst-plugins-base-libs
  gst-plugins-good      lib32-gst-plugins-good
  gst-plugins-ugly
  libgphoto2
  libpulse              lib32-libpulse
  libxcomposite         lib32-libxcomposite
  libxinerama           lib32-libxinerama
  opencl-icd-loader     lib32-opencl-icd-loader
  pcsclite              lib32-pcsclite
  samba
  sane
  sdl2                  lib32-sdl2
  unixodbc
  v4l-utils             lib32-v4l-utils
  wine-gecko
  wine-mono
)
makedepends=(${makedepends[@]} ${depends[@]})
install=wine.install

prepare() {
  # Get rid of old build dirs
  rm -rf $pkgname-{32,64}-build
  mkdir $pkgname-{32,64}-build

  cd wine
  # Fix for https://bugs.winehq.org/show_bug.cgi?id=57854
  patch -Np1 -i "$srcdir"/fix-ptr-access.patch
}

build() {
  # Doesn't compile without remove these flags as of 4.10
  export CFLAGS="$CFLAGS -ffat-lto-objects"

  # Apply flags for cross-compilation
  export CROSSCFLAGS="-O2 -pipe -g"
  export CROSSCXXFLAGS="-O2 -pipe -g"
  export CROSSLDFLAGS="-Wl,-O1"

  echo "Building Wine-64..."
  cd "$srcdir/$pkgname-64-build"
  ../wine/configure \
    --prefix=/usr \
    --libdir=/usr/lib \
    --with-x \
    --with-wayland \
    --with-gstreamer \
    --enable-win64

  make

  echo "Building Wine-32..."
  export PKG_CONFIG_PATH="/usr/lib32/pkgconfig"
  cd "$srcdir/$pkgname-32-build"
  ../wine/configure \
    --prefix=/usr \
    --libdir=/usr/lib \
    --with-x \
    --with-wayland \
    --with-gstreamer \
    --with-wine64="$srcdir/$pkgname-64-build"

  make
}

package() {
  echo "Packaging Wine-32..."
  cd "$srcdir/$pkgname-32-build"
  make prefix="$pkgdir/usr" \
    libdir="$pkgdir/usr/lib" \
    dlldir="$pkgdir/usr/lib/wine" install

  echo "Packaging Wine-64..."
  cd "$srcdir/$pkgname-64-build"
  make prefix="$pkgdir/usr" \
    libdir="$pkgdir/usr/lib" \
    dlldir="$pkgdir/usr/lib/wine" install

  # Font aliasing settings for Win32 applications
  install -d "$pkgdir"/usr/share/fontconfig/conf.{avail,default}
  install -m644 "$srcdir/30-win32-aliases.conf" "$pkgdir/usr/share/fontconfig/conf.avail"
  ln -s ../conf.avail/30-win32-aliases.conf "$pkgdir/usr/share/fontconfig/conf.default/30-win32-aliases.conf"
  install -Dm 644 "$srcdir/wine-binfmt.conf" "$pkgdir/usr/lib/binfmt.d/wine.conf"
}

# vim:set ts=8 sts=2 sw=2 et:
