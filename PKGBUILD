# $Id$
# Maintainer: Sébastien Luttringer <seblu@aur.archlinux.org>

pkgbase=linux-tools
pkgname=('perf' 'cpupower')
pkgver=3.4
pkgrel=1
license=('GPL2')
arch=('i686' 'x86_64')
url='http://www.kernel.org'
options=('!strip')
makedepends=('asciidoc' 'xmlto')
# split packages need all package dependencies set manually in makedepends
makedepends+=('python2' 'libnewt' 'elfutils' 'pciutils')
source=("http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-$pkgver.tar.xz"
#        "http://ftp.kernel.org/pub/linux/kernel/v3.x/patch-$pkgver.4.xz"
        'cpupower.rc'
        'cpupower.conf'
        'cpupower.service')
md5sums=('967f72983655e2479f951195953e8480'
         '73dbc931e86b3b73d6e2338dcbee81a4'
         '857ccdd0598511e3bf4b63522754dc48'
         '20870541e88109d2f153be3c58a277f1')

build() {
  # apply stable patching set
  if [[ -e "$srcdir"/patch-* ]]; then
    msg2 'Applying stable patch set'
    patch -N -p1 -i "$srcdir"/patch-*
  fi

  msg2 'Build perf'
  pushd linux-$pkgver/tools/perf
  make \
    DESTDIR="$pkgdir/usr" \
    perfexecdir="lib/$pkgname" \
    PYTHON=python2 \
    PERF_VERSION=$pkgver-$pkgrel \
    all man
  popd

  msg2 'Build cpupower'
  cd linux-$pkgver/tools/power/cpupower
  # we cannot use --as-needed
  LDFLAGS=${LDFLAGS:+"$LDFLAGS,--no-as-needed"}
  make VERSION=$pkgver-$pkgrel
}

package_perf() {
  pkgdesc='Linux kernel performance auditing tool'
  depends=('python2' 'libnewt' 'elfutils')

  cd linux-$pkgver/tools/perf
  make \
    DESTDIR="$pkgdir/usr" \
    perfexecdir="lib/$pkgname" \
    PYTHON=python2 \
    PERF_VERSION=$pkgver-$pkgrel \
    install install-man
}

package_cpupower() {
  pkgdesc='Linux kernel tool to examine and tune power saving related features of your processor'
  backup=('etc/conf.d/cpupower')
  depends=('pciutils')
  conflicts=('cpufrequtils')

  pushd linux-$pkgver/tools/power/cpupower
  make \
    DESTDIR="$pkgdir" \
    mandir='/usr/share/man' \
    docdir='/usr/share/doc/cpupower' \
    install install-man
  popd
  # install rc.d script
  install -D -m 755 cpupower.rc "$pkgdir/etc/rc.d/cpupower"
  install -D -m 644 cpupower.conf "$pkgdir/etc/conf.d/cpupower"
  install -D -m 644 cpupower.service "$pkgdir/usr/lib/systemd/system/cpupower.service"
}

# vim:set ts=2 sw=2 ft=sh et:
