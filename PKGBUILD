pkgname=shadow
pkgver=4.18.0
pkgrel=1
pkgdesc="Password and account management tool suite with support for shadow files and PAM"
arch=('x86_64')
url="https://github.com/shadow-maint/shadow/"
license=('BSD-3-Clause')
groups=('base')
depends=(
    'glibc'
    'acl'
    'attr'
    'libcap'
    'libxcrypt'
)
backup=(
    etc/default/useradd
    etc/login.defs
)
options=('!emptydirs' '!lto')
install=${pkgname}.install
source=(https://github.com/shadow-maint/shadow/releases/download/${pkgver}/${pkgname}-${pkgver}.tar.xz
    useradd.defaults)
sha256sums=(add4604d3bc410344433122a819ee4154b79dd8316a56298c60417e637c07608
    727c4f4ebf4597822b9fd62256f7e547f9c2f90fcad7d471db7a00d1bbc92e3d)

prepare() {
    cd ${pkgname}-${pkgver}

    sed -i 's/groups$(EXEEXT) //' src/Makefile.in
    find man -name Makefile.in -exec sed -i 's/groups\.1 / /'   {} \;
    find man -name Makefile.in -exec sed -i 's/getspnam\.3 / /' {} \;
    find man -name Makefile.in -exec sed -i 's/passwd\.5 / /'   {} \;

    sed -e 's:#ENCRYPT_METHOD DES:ENCRYPT_METHOD YESCRYPT:' \
        -e 's:/var/spool/mail:/var/mail:'                   \
        -e '/PATH=/{s@/sbin:@@;s@/bin:@@}'                  \
        -i etc/login.defs
}

build() {
    cd ${pkgname}-${pkgver}

    local configure_args=(
        --sysconfdir=/etc
        --bindir=/usr/bin
        --sbindir=/usr/sbin
        --disable-static
        --with-{b,yes}crypt
        --without-libbsd
        --with-group-name-max-length=32
        ${configure_options}
    )

    export CFLAGS="${CFLAGS} -DEXTRA_CHECK_HOME_DIR"

    ./configure "${configure_args[@]}"

    make
}

package() {
    cd ${pkgname}-${pkgver}

    make DESTDIR=${pkgdir} pamddir= install
    make -C man DESTDIR=${pkgdir} install-man

    install -vDm644 ${srcdir}/useradd.defaults ${pkgdir}/etc/default/useradd
}
