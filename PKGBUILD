# Maintainer: artist for XLibre

pkgbase=xlibre-xserver-beta
_pkgbase="${pkgbase%-*}"
pkgname=($_pkgbase-beta $_pkgbase-xephyr-beta $_pkgbase-xvfb-beta $_pkgbase-xnest-beta $_pkgbase-common-beta $_pkgbase-devel-beta)
pkgver=25.1.5
pkgrel=1
arch=('x86_64')
license=('LicenseRef-Adobe-Display-PostScript'
         'BSD-3-Clause' 
         'LicenseRef-DEC-3-Clause' 
         'HPND'
         'LicenseRef-HPND-sell-MIT-disclaimer-xserver'
         'HPND-sell-variant' 
         'ICU'
         'ISC'
         'MIT'
         'MIT-open-group'
         'NTP'
         'SGI-B-2.0'
         'SMLNJ'
         'X11'
         'X11-distribute-modifications-variant')
groups=('xlibre')
url="https://github.com/X11Libre/xserver"
makedepends=('xlibre-xserver' 'xorgproto' 'pixman' 'libx11' 'mesa' 'mesa-libgl'
             'libxkbfile' 'libxfont2' 'libpciaccess' 'libxv' 'libxcvt'
             'libxmu' 'libxrender' 'libxi' 'libxaw' 'libxtst' 'libxres'
             'xorg-xkbcomp' 'xorg-util-macros' 'xorg-font-util' 'libepoxy'
             'xcb-util' 'xcb-util-image' 'xcb-util-renderutil' 'xcb-util-wm' 'xcb-util-keysyms'
             'libxshmfence' 'libunwind' 'meson' 'dbus')
options=('!debug')
source=("${url}/archive/refs/tags/${_pkgbase}-${pkgver}.tar.gz"
        xvfb-run # with updates from FC master
        xvfb-run.1)

build() {
  export CFLAGS=${CFLAGS/-fno-plt}
  export CXXFLAGS=${CXXFLAGS/-fno-plt}
  export LDFLAGS=${LDFLAGS/-Wl,-z,now}

  arch-meson xserver-${_pkgbase}-${pkgver} build \
    --buildtype=release \
    -D ipv6=true \
    -D xvfb=true \
    -D xnest=true \
    -D xcsecurity=true \
    -D xorg=true \
    -D xephyr=true \
    -D xfbdev=true \
    -D glamor=true \
    -D udev=true \
    -D udev_kms=true \
    -D dtrace=false \
    -D systemd_logind=true \
    -D suid_wrapper=true \
    -D linux_acpi=false \
    -D xkb_dir=/usr/share/X11/xkb \
    -D xkb_output_dir=/var/lib/xkb \
    -D libunwind=true

  # Print config
  meson configure build
  ninja -C build

  # fake installation to be seperated into packages
  DESTDIR="${srcdir}/fakeinstall" ninja -C build install
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    # use copy so a new file is created and fakeroot can track properties such as setuid
    cp -av "${src}" "${dir}/"
    rm -rf "${src}"
  done
}

package_xlibre-xserver-common-beta() {
  pkgdesc="XLibre fork of X.Org Xorg server common files"
  depends=(xkeyboard-config xorg-xkbcomp xorg-setxkbmap)
  provides=('xlibre-xserver-common' 'xorg-server-common')
  conflicts=('xlibre-xserver-common' 'xorg-server-common' "xorg-server<$pkgver")

  _install fakeinstall/usr/lib/xorg/protocol.txt
  _install fakeinstall/usr/share/man/man1/Xserver.1

  install -m644 -Dt "${pkgdir}/var/lib/xkb/" xserver-${_pkgbase}-${pkgver}/xkb/README.compiled
  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver-${_pkgbase}-${pkgver}/COPYING
}

package_xlibre-xserver-beta() {
  pkgdesc="XLibre fork of X.Org X server"
  depends=(xlibre-xserver-common-beta xlibre-input-libinput libepoxy libxfont2 pixman libunwind
           dbus libgl nettle libxdmcp sh glibc libxau libtirpc libbsd seatd
           libpciaccess libdrm libxshmfence libxcvt) # FS#52949
  provides=('xorg-server' 'X-ABI-VIDEODRV_VERSION=28.0' 'X-ABI-XINPUT_VERSION=26.0' 'X-ABI-EXTENSION_VERSION=11.0' 'x-server' 'x11win-server')
  conflicts=('xlibre-xserver' 'xorg-server' 'nvidia-utils<=331.20' 'glamor-egl' 'xf86-video-modesetting')
  replaces=('xlibre-xserver' 'glamor-egl' 'xf86-video-modesetting')

  _install fakeinstall/usr/bin/{X,Xorg,gtf,Xfbdev}
  _install fakeinstall/usr/lib/Xorg{,.wrap}
  _install fakeinstall/usr/lib/xorg/modules/*
  _install fakeinstall/usr/share/X11/xorg.conf.d/10-{nvidia,quirks}.conf
  _install fakeinstall/usr/share/man/man1/{Xorg,Xorg.wrap,gtf,Xfbdev}.1
  _install fakeinstall/usr/share/man/man4/{exa,fbdevhw,inputtestdrv,modesetting}.4
  _install fakeinstall/usr/share/man/man5/{Xwrapper.config,xorg.conf,xorg.conf.d}.5

  # distro specific files must be installed in /usr/share/X11/xorg.conf.d
  install -m755 -d "${pkgdir}/etc/X11/xorg.conf.d"

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver-${_pkgbase}-${pkgver}/COPYING
}

package_xlibre-xserver-xephyr-beta() {
  pkgdesc="XLibre fork of X.Org nested X server that runs as an X application"
  depends=(xlibre-xserver-common-beta 'X-ABI-XINPUT_VERSION=26.0' libxfont2 libgl libepoxy libunwind
           xcb-util-image xcb-util-renderutil xcb-util-wm xcb-util-keysyms pixman
           nettle libtirpc xcb-util libxdmcp libx11 libxau libxshmfence glibc seatd)
  provides=('xlibre-xserver-xephyr' 'xorg-server-xephyr')
  conflicts=('xlibre-xserver-xephyr' 'xorg-server-xephyr')

  _install fakeinstall/usr/bin/Xephyr
  _install fakeinstall/usr/share/man/man1/Xephyr.1

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver-${_pkgbase}-${pkgver}/COPYING
}

package_xlibre-xserver-xvfb-beta() {
  pkgdesc="XLibre fork of X.Org virtual framebuffer X server"
  # xvfb-run is GPLv2, rest is MIT
  license=('MIT' 'GPL-2.0-only')
  depends=(xlibre-xserver-common-beta 'X-ABI-XINPUT_VERSION=26.0' libxfont2 libunwind pixman xorg-xauth 
           libgl nettle libtirpc seatd
           libxdmcp sh glibc libxau)
  provides=('xlibre-xserver-xvfb' 'xorg-server-xvfb')
  conflicts=('xlibre-xserver-xvfb' 'xorg-server-xvfb')

  _install fakeinstall/usr/bin/Xvfb
  _install fakeinstall/usr/share/man/man1/Xvfb.1

  install -m755 "${srcdir}/xvfb-run" "${pkgdir}/usr/bin/"
  install -m644 "${srcdir}/xvfb-run.1" "${pkgdir}/usr/share/man/man1/"

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver-${_pkgbase}-${pkgver}/COPYING
}

package_xlibre-xserver-xnest-beta() {
  pkgdesc="XLibre fork of X.Org nested X server that runs as an X application"
  depends=(xlibre-xserver-common-beta 'X-ABI-XINPUT_VERSION=26.0' libxfont2 libunwind libxext pixman nettle
           libtirpc seatd
           libxdmcp glibc libx11 libxau)
  provides=('xlibre-xserver-xnest' 'xorg-server-xnest')
  conflicts=('xlibre-xserver-xnest' 'xorg-server-xnest')

  _install fakeinstall/usr/bin/Xnest
  _install fakeinstall/usr/share/man/man1/Xnest.1

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver-${_pkgbase}-${pkgver}/COPYING
}

package_xlibre-xserver-devel-beta() {
  pkgdesc="XLibre fork of X.Org development files for the X.Org X server"
  depends=('xlibre-xserver-beta' 'X-ABI-XINPUT_VERSION=26.0' 'xorgproto' 'mesa' 'libpciaccess' 'pixman'
           # not technically required but almost every Xorg pkg needs it to build
           'xorg-util-macros')
  provides=('xlibre-xserver-devel' 'xorg-server-devel')
  conflicts=('xlibre-xserver-devel' 'xorg-server-devel')

  _install fakeinstall/usr/include/xorg/*
  _install fakeinstall/usr/lib/pkgconfig/xorg-server.pc
  _install fakeinstall/usr/lib/pkgconfig/xlibre-server.pc
  _install fakeinstall/usr/share/aclocal/xorg-server.m4

  # license
  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" xserver-${_pkgbase}-${pkgver}/COPYING

  # make sure there are no files left to install
  find fakeinstall -depth -print0 | xargs -0 rmdir
}
sha256sums=('7ee3612e7674595d7cef337d6955fddf02e63719cbc2607b29d80a17891eca0e'
            '27ce50f4432e5549e662db857118761fa9cd74c6900aac52c4db768c956838db'
            '2460adccd3362fefd4cdc5f1c70f332d7b578091fb9167bf88b5f91265bbd776')
