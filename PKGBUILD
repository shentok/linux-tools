# $Id$
# Maintainer: Sébastien Luttringer

pkgbase=linux-tools
pkgname=(
  'acpidump'
  'cgroup_event_listener'
  'cpupower'
  'libtraceevent'
  'linux-tools-meta'
  'perf'
  'tmon'
  'usbip'
  'x86_energy_perf_policy'
)
pkgver=3.14
pkgrel=1
license=('GPL2')
arch=('i686' 'x86_64')
url='http://www.kernel.org'
options=('!strip')
# split packages need all package dependencies set manually in makedepends
# kernel source deps
makedepends=('asciidoc' 'xmlto')
# perf deps
makedepends+=('perl' 'python2' 'libnewt' 'elfutils' 'audit' 'libunwind' 'numactl')
# cpupower deps
makedepends+=('pciutils')
# usbip deps
makedepends+=('glib2' 'sysfsutils')
# tmon deps
makedepends+=('ncurses')
groups=("$pkgbase")
source=("http://ftp.kernel.org/pub/linux/kernel/v3.x/linux-$pkgver.tar.xz"
#        "http://ftp.kernel.org/pub/linux/kernel/v3.x/patch-$pkgver.3.xz"
        'cpupower.default'
        'cpupower.systemd'
        'cpupower.service'
        'usbipd.service'
        '01-fix-perf-python.patch'
        '02-archlinux-paths.patch')
# http://www.kernel.org/pub/linux/kernel/v3.x/sha256sums.asc
sha256sums=('61558aa490855f42b6340d1a1596be47454909629327c49a5e4e10268065dffa'
            '4fa509949d6863d001075fa3e8671eff2599c046d20c98bb4a70778595cd1c3f'
            'fbf6e0ce6eb0ef15703fe212958de6ca46e62188900b5e9f9272ed3cc9cfd54e'
            'a89284d0ecb556ca53a66d1c2087b5fd6d0a901ab2769cd3aebb93f4478905dc'
            '2e187734d8aec58a3046d79883510d779aa93fb3ab20bd3132c1a607ebe5498f'
            'fce128f5e0abfa6916d5cb881456d892d1b163b9639166a4c6c1d53e4dc5086a'
            'eb866a589a26b1979ffb2fe08be09417e277a4befac34bdb279a6bb3a27b0570')

prepare() {
  cd linux-$pkgver
  #patch -N -p1 -i "$srcdir/patch-$pkgver.3"
  patch -N -p1 -i "$srcdir/01-fix-perf-python.patch"
  patch -N -p1 -i "$srcdir/02-archlinux-paths.patch"
}

build() {
  msg2 'libtraceevent'
  pushd linux-$pkgver/tools/lib/traceevent
  make
  popd

  msg2 'perf'
  pushd linux-$pkgver/tools/perf
  make \
    WERROR=0 \
    DESTDIR="$pkgdir/usr" \
    perfexecdir='lib/perf' \
    PYTHON=python2 \
    PYTHON_CONFIG=python2-config \
    NO_GTK2=1 \
    PERF_VERSION=$pkgver-$pkgrel \
    all man
  popd

  msg2 'cpupower'
  pushd linux-$pkgver/tools/power/cpupower
  # we cannot use --as-needed
  #LDFLAGS=${LDFLAGS:+"$LDFLAGS,--no-as-needed"}
  make VERSION=$pkgver-$pkgrel
  popd

  msg2 'x86_energy_perf_policy'
  pushd linux-$pkgver/tools/power/x86/x86_energy_perf_policy
  make
  popd

  msg2 'usbip'
  pushd linux-$pkgver/drivers/staging/usbip/userspace
  # fix missing man page
  sed -i 's/usbip_bind_driver.8//' Makefile.am
  ./autogen.sh
  ./configure --prefix=/usr --sbindir=/usr/bin
  make
  popd

  msg2 'tmon'
  pushd linux-$pkgver/tools/thermal/tmon
  make
  popd

  msg2 'acpidump'
  pushd linux-$pkgver/tools/power/acpi
  make
  popd

  msg2 'cgroup_event_listener'
  pushd linux-$pkgver/tools/cgroup
  make
  popd
}

package_linux-tools-meta() {
  pkgdesc='Linux kernel tools meta package'
  groups=()
  depends=(
    'acpidump'
    'cgroup_event_listener'
    'cpupower'
    'libtraceevent'
    'linux-tools-meta'
    'perf'
    'tmon'
    'usbip'
    'x86_energy_perf_policy'
  )
}

package_libtraceevent() {
  pkgdesc='Linux kernel trace event library'
  depends=('glibc')

  cd linux-$pkgver/tools/lib/traceevent
  install -dm 755 "$pkgdir/usr/lib"
  install -m 644 libtraceevent.a libtraceevent.so "$pkgdir/usr/lib"
}

package_perf() {
  pkgdesc='Linux kernel performance auditing tool'
  depends=('perl' 'python2' 'libnewt' 'elfutils' 'audit' 'libunwind' 'binutils'
           'numactl')

  cd linux-$pkgver/tools/perf
  make \
    WERROR=0 \
    DESTDIR="$pkgdir/usr" \
    perfexecdir='lib/perf' \
    PYTHON=python2 \
    PYTHON_CONFIG=python2-config \
    NO_GTK2=1 \
    PERF_VERSION=$pkgver-$pkgrel \
    install install-man
  # move completion in new directory
  cd "$pkgdir"
  install -Dm644 usr/etc/bash_completion.d/perf usr/share/bash-completion/perf
  rm -r usr/etc
}

package_cpupower() {
  pkgdesc='Linux kernel tool to examine and tune power saving related features of your processor'
  backup=('etc/default/cpupower')
  depends=('bash' 'pciutils')
  conflicts=('cpufrequtils')
  replaces=('cpufrequtils')
  install=cpupower.install

  pushd linux-$pkgver/tools/power/cpupower
  make \
    DESTDIR="$pkgdir" \
    sbindir='/usr/bin' \
    mandir='/usr/share/man' \
    docdir='/usr/share/doc/cpupower' \
    install install-man
  popd
  # install startup scripts
  install -Dm 644 $pkgname.default "$pkgdir/etc/default/$pkgname"
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

  pushd linux-$pkgver/drivers/staging/usbip/userspace
  make install DESTDIR="$pkgdir"
  popd
  # module loading
  install -Dm 644 /dev/null "$pkgdir/usr/lib/modules-load.d/$pkgname.conf"
  printf 'usbip-core\nusbip-host\n' > "$pkgdir/usr/lib/modules-load.d/$pkgname.conf"
  # systemd
  install -Dm 644 usbipd.service "$pkgdir/usr/lib/systemd/system/usbipd.service"
}

package_tmon() {
  pkgdesc='Monitoring and Testing Tool for Linux kernel thermal subsystem'
  depends=('glibc' 'ncurses')

  cd linux-$pkgver/tools/thermal/tmon
  make install INSTALL_ROOT="$pkgdir"
}

package_acpidump() {
  pkgdesc='Dump system ACPI tables to an ASCII file'
  depends=('glibc')
  conflicts=('iasl')

  cd linux-$pkgver/tools/power/acpi
  make install sbindir=/usr/bin mandir=/usr/share/man DESTDIR="$pkgdir"
  #install -Dm755 acpidump "$pkgdir/usr/bin/acpidump"
  #install -Dm644 acpidump.8 "$pkgdir/usr/share/man/man8/acpidump.8"
}

package_cgroup_event_listener() {
  pkgdesc='Simple listener of cgroup events'
  depends=('glibc')

  cd linux-$pkgver/tools/cgroup
  install -Dm755 cgroup_event_listener "$pkgdir/usr/bin/cgroup_event_listener"
}

# vim:set ts=2 sw=2 et:
