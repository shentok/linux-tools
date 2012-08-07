# $Id$
# Maintainer: Sébastien Luttringer <seblu@aur.archlinux.org>

pkgbase=linux-tools
pkgname=('perf' 'cpupower' 'x86_energy_perf_policy' 'usbip')
pkgver=3.5
pkgrel=3
license=('GPL2')
arch=('i686' 'x86_64')
url='http://www.kernel.org'
options=('!strip')
# split packages need all package dependencies set manually in makedepends
# kernel source deps
makedepends=('asciidoc' 'xmlto')
# perf deps
makedepends+=('perl' 'python2' 'libnewt' 'elfutils')
# cpupower deps
makedepends+=('pciutils')
# usbip deps
makedepends+=('glib2' 'sysfsutils')
groups=("$pkgbase")
source=("http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-$pkgver.tar.xz"
#        "http://ftp.kernel.org/pub/linux/kernel/v3.x/patch-$pkgver.4.xz"
        'cpupower.conf'
        'cpupower.rc'
        'cpupower.systemd'
        'cpupower.service'
        'usbipd.conf'
        'usbipd.rc'
        'usbipd.service')
md5sums=('24153eaaa81dedc9481ada8cd9c3b83d'
         '857ccdd0598511e3bf4b63522754dc48'
         '1d9214637968b91706b6e616a100d44b'
         'c0d17b5295fe964623c772a2dd981771'
         '2450e8ff41b30eb58d43b5fffbfde1f4'
         'e8fac9c45a628015644b4150b139278a'
         '8a3831d962ff6a9968c0c20fd601cdec'
         'ba7c1c513314dd21fb2334fb8417738f')

build() {
  # apply stable patching set
  if (( NOEXTRACT == 0 )) && [[ -e "$srcdir"/patch-* ]]; then
    msg2 'Applying stable patch set'
    patch -N -p1 -i "$srcdir"/patch-*
  fi

  msg2 'Build perf'
  pushd linux-$pkgver/tools/perf
  make \
    WERROR=0 \
    DESTDIR="$pkgdir/usr" \
    perfexecdir='lib/perf' \
    PYTHON=python2 \
    NO_GTK2=1 \
    PERF_VERSION=$pkgver-$pkgrel \
    all man
  popd

  msg2 'Build cpupower'
  pushd linux-$pkgver/tools/power/cpupower
  # we cannot use --as-needed
  LDFLAGS=${LDFLAGS:+"$LDFLAGS,--no-as-needed"}
  make VERSION=$pkgver-$pkgrel
  popd

  msg2 'Build x86_energy_perf_policy'
  pushd linux-$pkgver/tools/power/x86/x86_energy_perf_policy
  make
  popd

  msg2 'Build usbip'
  pushd linux-$pkgver/drivers/staging/usbip/userspace
  ./autogen.sh
  ./configure --prefix=/usr
  make
  popd
}

package_perf() {
  pkgdesc='Linux kernel performance auditing tool'
  depends=('perl' 'python2' 'libnewt' 'elfutils')

  cd linux-$pkgver/tools/perf
  make \
    WERROR=0 \
    DESTDIR="$pkgdir/usr" \
    perfexecdir='lib/perf' \
    PYTHON=python2 \
    NO_GTK2=1 \
    PERF_VERSION=$pkgver-$pkgrel \
    install install-man
}

package_cpupower() {
  pkgdesc='Linux kernel tool to examine and tune power saving related features of your processor'
  backup=('etc/conf.d/cpupower')
  depends=('bash' 'pciutils')
  conflicts=('cpufrequtils')

  pushd linux-$pkgver/tools/power/cpupower
  make \
    DESTDIR="$pkgdir" \
    mandir='/usr/share/man' \
    docdir='/usr/share/doc/cpupower' \
    install install-man
  popd
  # install daemon scripts
  install -Dm 755 $pkgname.rc "$pkgdir/etc/rc.d/$pkgname"
  install -Dm 644 $pkgname.conf "$pkgdir/etc/conf.d/$pkgname"
  install -Dm 644 $pkgname.service "$pkgdir/usr/lib/systemd/system/$pkgname.service"
  install -Dm 755 $pkgname.systemd "$pkgdir/usr/lib/systemd/scripts/$pkgname"
}

package_x86_energy_perf_policy() {
  pkgdesc='Read or write MSR_IA32_ENERGY_PERF_BIAS'
  depends=('glibc')

  cd linux-$pkgver/tools/power/x86/x86_energy_perf_policy
  install -Dm 755 x86_energy_perf_policy "$pkgdir/usr/bin/x86_energy_perf_policy"
  install -Dm 644 x86_energy_perf_policy.8 "$pkgdir/usr/share/man/man8/x86_energy_perf_policy.8"
}

package_usbip() {
  pkgdesc='An USB device sharing system over IP network'
  depends=('glib2' 'sysfsutils')
  options=('!libtool')

  pushd linux-$pkgver/drivers/staging/usbip/userspace
  make install DESTDIR="$pkgdir"
  popd
  # module loading
  install -Dm 644 /dev/null "$pkgdir/usr/lib/modules-load.d/$pkgname.conf"
  printf 'usbip-core\nusbip-host\n' > "$pkgdir/usr/lib/modules-load.d/$pkgname.conf"
  # install daemon scripts
  install -Dm 755 usbipd.rc "$pkgdir/etc/rc.d/usbipd"
  install -Dm 644 usbipd.conf "$pkgdir/etc/conf.d/usbipd"
  install -Dm 644 usbipd.service "$pkgdir/usr/lib/systemd/system/usbipd.service"
}

# vim:set ts=2 sw=2 ft=sh et:
