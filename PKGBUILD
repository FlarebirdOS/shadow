pkgname=shadow
pkgver=4.18.0
pkgrel=2
pkgdesc="Password and account management tool suite with support for shadow files and PAM"
arch=('x86_64')
url="https://github.com/shadow-maint/shadow/"
license=('BSD-3-Clause')
groups=('base')
depends=(
    'acl'
    'attr'
    'glibc'
    'libcap'
    'libxcrypt'
    'linux-pam'
    'cracklib'
    'libpwquality'
)
backup=(
    etc/default/useradd
    etc/login.defs
    etc/pam.d/{chage,chpasswd,login,newusers,passwd,su})
options=('!emptydirs' '!lto')
install=${pkgname}.install
source=(https://github.com/shadow-maint/shadow/releases/download/${pkgver}/${pkgname}-${pkgver}.tar.xz
    chage
    chpasswd
    login
    passwd
    su
    useradd.defaults)
sha256sums=(add4604d3bc410344433122a819ee4154b79dd8316a56298c60417e637c07608
    da335f605486637c33288e518a313da6a21b4f62a4f72251ec0d40b60fcdc599
    43fba8a6761448400acd5d4d77e9b492ea2419861387c34838d787dd09884cb9
    27f2bfa0d042bfd8ba3ce28f9d5dbeea047fa5f0f305b62f6c70f4e8f8c450f5
    03383e3dc19acb3ad8217e55b52e15e9ca237de36c50302d5e51a881a4d64657
    6d3c0c40db642ded494f8d422ea808322f54675f43a9bc000d7c2889aca3dba3
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
        --with-libpam
        --with-libcrack
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

    install -v -m644 ${pkgdir}/etc/login.defs ${pkgdir}/etc/login.defs.orig
    for FUNCTION in FAIL_DELAY               \
                    FAILLOG_ENAB             \
                    LASTLOG_ENAB             \
                    MAIL_CHECK_ENAB          \
                    OBSCURE_CHECKS_ENAB      \
                    PORTTIME_CHECKS_ENAB     \
                    QUOTAS_ENAB              \
                    CONSOLE MOTD_FILE        \
                    FTMP_FILE NOLOGINS_FILE  \
                    ENV_HZ PASS_MIN_LEN      \
                    SU_WHEEL_ONLY            \
                    PASS_CHANGE_TRIES        \
                    PASS_ALWAYS_WARN         \
                    CHFN_AUTH ENCRYPT_METHOD \
                    ENVIRON_FILE
    do
        sed -i "s/^${FUNCTION}/# &/" ${pkgdir}/etc/login.defs
    done

    install -vm644 ${srcdir}/{login,passwd,su,chpasswd,chage} -Dt ${pkgdir}/etc/pam.d/
    sed -e s/chpasswd/newusers/ ${pkgdir}/etc/pam.d/chpasswd > ${pkgdir}/etc/pam.d/newusers

    for PROGRAM in chfn chgpasswd chsh groupadd groupdel \
                groupmems groupmod useradd userdel usermod
    do
        install -v -m644 ${pkgdir}/etc/pam.d/chage ${pkgdir}/etc/pam.d/${PROGRAM}
        sed -i "s/chage/$PROGRAM/" ${pkgdir}/etc/pam.d/${PROGRAM}
    done
}
