# Maintainer: Vasiliy Stelmachenok <ventureo@cachyos.org>
# Contributor: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Eduardo Romero <eduardo@archlinux.org>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>

pkgname=wine-staging-ntsync-pure
pkgver=9.22.r0.g51ccd95c49c
pkgrel=4
source=(
  "$pkgname::git+https://gitlab.winehq.org/wine/wine.git"
  "git+https://gitlab.winehq.org/wine/wine-staging.git"
  30-win32-aliases.conf
  wine-binfmt.conf
  ntsync-v5-9.22-mainline.patch
  wineserver-enable-link-time-optimization.patch
  explorer-prefer-wayland-over-x11-by-default.patch
  msvcrt_nativebuiltin.patch
)
sha512sums=(
  'SKIP'
  'SKIP'
  '6e54ece7ec7022b3c9d94ad64bdf1017338da16c618966e8baf398e6f18f80f7b0576edf1d1da47ed77b96d577e4cbb2bb0156b0b11c183a0accf22654b0a2bb'
  'bdde7ae015d8a98ba55e84b86dc05aca1d4f8de85be7e4bd6187054bfe4ac83b5a20538945b63fb073caab78022141e9545685e4e3698c97ff173cf30859e285'
  '5064e7cb133bae1a7c3d177d63f3ea5a85207cfe8eea05f23ffed63ad9c3718344b11e32318c36901c163db1a4d1c8894773e213de2d6d77f1b91e7c169594fd'
  'f47afccd51f010a282cab07f343479fba6f14ed8c654c8ff7bbc5c808c0d15be967f35d555f496b5f9bc281aa34f78bb9fd2c93a6e1241682cf3ad201dfe88f3'
  'd32d06216d05bdcd6b00b26820f3ba141f58e7874bb51ed2958a7b0f4dd05da612ae0e8d792e7134fae92f2a6cc05f77077e1443a221cda242a836747a7e22ea'
  '15f767f2fd7d019bb0e15ece646f1fd1db2e7fda1830c38a05d0fbe3dca580b43d568bcb6094362a57322e78cf4bf599199b0edb7b3c4557779d64bbc1a7b071'
)

pkgdesc="A compatibility layer for running Windows programs"
url="https://www.winehq.org"
arch=(x86_64)
options=(staticlibs !lto)
license=(LGPL-2.1-or-later)
depends=(
  desktop-file-utils
  fontconfig
  freetype2
  gcc-libs
  gettext
  libxcursor
  libxkbcommon
  libxi
  libxrandr
  wayland
  ffmpeg
  unixodbc
)
makedepends=(autoconf bison perl flex mingw-w64-gcc git
  alsa-lib
  gnutls
  gst-plugins-base-libs
  libpulse
  libxcomposite
  mesa
  opencl-headers
  opencl-icd-loader
  sdl2
  vulkan-headers
  vulkan-icd-loader
)
optdepends=(
  alsa-lib
  alsa-plugins
  gnutls
  gst-plugins-bad
  gst-plugins-base
  gst-plugins-base-libs
  gst-plugins-good
  gst-plugins-ugly
  libpulse
  libxcomposite
  opencl-icd-loader
  sdl2
  wine-gecko
  wine-mono
)
provides=("wine-staging" "wine" 'wine-wow64')
conflicts=("wine")
makedepends=(${makedepends[@]} ${depends[@]})
install=wine.install

pkgver() {
  git -C "$pkgname" describe --long --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g;s/^wine.//;s/^v//;s/\.rc/rc/'
}

prepare() {
  # Get rid of old build dirs
  rm -rf "$pkgname-build"
  mkdir "$pkgname-build"

  cd "$pkgname"

  # apply wine-staging patchset (no Esync to prevent conflicts with NTSync)
  ../wine-staging/staging/patchinstall.py --backend=git-apply --all \
    -W eventfd_synchronization \
    -W ntdll-Syscall_Emulation \
    -W server-PeekMessage \
    -W server-Signal_Thread

  # NTSync for Wine 9.22
  patch -Np1 -i "${srcdir}/ntsync-v5-9.22-mainline.patch"

  # LTO for Wineserver
  patch -Np1 -i "${srcdir}/wineserver-enable-link-time-optimization.patch"

  # Prefer Wayland over X11 by default
  patch -Np1 -i "${srcdir}/explorer-prefer-wayland-over-x11-by-default.patch"

  # Use native Visual C++ DLLs
  patch -Np1 -i "${srcdir}/msvcrt_nativebuiltin.patch"
}

build() {
  # Doesn't compile without remove these flags as of 4.10
  export CFLAGS="$CFLAGS -ffat-lto-objects"

  # Apply flags for cross-compilation
  export CROSSCFLAGS="-O2 -march=native -mtune=native -fno-semantic-interposition -fcf-protection=none -mharden-sls=none -funroll-loops -fivopts -fmodulo-sched -fdata-sections -ffunction-sections -fprofile-correction -floop-nest-optimize -fipa-pta -fgraphite-identity -floop-strip-mine"
  export CROSSCXXFLAGS="-O2 -march=native -mtune=native -fno-semantic-interposition -fcf-protection=none -mharden-sls=none -funroll-loops -fivopts -fmodulo-sched -fdata-sections -ffunction-sections -fprofile-correction -floop-nest-optimize -fipa-pta -fgraphite-identity -floop-strip-mine"
  export CROSSLDFLAGS="-Wl,-O2,--sort-common,--as-needed,-lgomp,--file-alignment,4096"

  echo "Building Wine..."
  cd "$pkgname-build"
  ../$pkgname/configure \
    --disable-tests \
    --prefix=/usr \
    --libdir=/usr/lib \
    --with-x \
    --with-wayland \
    --with-gstreamer \
    --with-ffmpeg \
    --without-sane \
    --without-v4l2 \
    --without-cups \
    --without-gphoto \
    --without-xinerama \
    --without-pcsclite \
    --without-xxf86vm \
    --without-pcap \
    --enable-archs=x86_64,i386

  make
}

package() {
  cd "$pkgname-build"
  make prefix="$pkgdir/usr" \
    libdir="$pkgdir/usr/lib" \
    dlldir="$pkgdir/usr/lib/wine" install

  # Strip Windows binaries
  i686-w64-mingw32-strip --strip-debug "$pkgdir"/usr/lib/wine/i386-windows/*.dll
  x86_64-w64-mingw32-strip --strip-debug "$pkgdir"/usr/lib/wine/x86_64-windows/*.dll

  # Font aliasing settings for Win32 applications
  install -d "$pkgdir"/usr/share/fontconfig/conf.{avail,default}
  install -m644 "$srcdir/30-win32-aliases.conf" "$pkgdir/usr/share/fontconfig/conf.avail"
  ln -s ../conf.avail/30-win32-aliases.conf "$pkgdir/usr/share/fontconfig/conf.default/30-win32-aliases.conf"
  install -Dm 644 "$srcdir/wine-binfmt.conf" "$pkgdir/usr/lib/binfmt.d/wine.conf"
}

# vim:set ts=8 sts=2 sw=2 et:
