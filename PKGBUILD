 
# Maintainer: Konstantin Gizdov < arch at kge dot pw >
# Contributor: Frank Siegert < frank.siegert at googlemail dot com >
# Contributor: Scott Lawrence < bytbox at gmail dot com >
# Contributor: Thomas Dziedzic < gostrc at gmail dot com >
# Contributor: Sebastian Voecking < voeck at web dot de >
# Adapted for arm : Lagarde François

pkgbase=root
pkgname=('root')
pkgver=6.16.00
pkgrel=6
pkgdesc='C++ data analysis framework and interpreter from CERN'
arch=('x86_64' 'aarch64')
url='https://root.cern.ch'
license=('LGPL2.1')
makedepends=('ccache'
             'cfitsio'
             'cmake'
             'fcgi'
             'fftw'
             'ftgl'
             'blas'
             'gcc-fortran'
             'giflib'
             'git'
             'gl2ps'
             'glew'
             'go-pie'
             'gsl'
             'hicolor-icon-theme'
             'intel-tbb'
             'libafterimage'
             'libmariadbclient'
             'librsvg'
             'libxpm'
             'ocaml'
             'ocaml-ctypes'
             'openssl'
             'postgresql-libs'
             'python'
             'python-numpy'
             'sqlite'
             'tex-gyre-fonts'
             'unuran'
             'xmlrpc-c'
             'xrootd>=4.6.0-2'
             'xxhash>=0.6.5-1'
             'z3')
depends=('blas'
         'desktop-file-utils'
         'fcgi'
         'fftw'
         'ftgl'
         'giflib'
         'gl2ps'
         'glew'
         'gsl'
         'hicolor-icon-theme'
         'intel-tbb'
         'libafterimage'
         'librsvg'
         'libxpm'
         'tex-gyre-fonts'
         'xxhash>=0.6.5-1')
optdepends=('cfitsio: Read images and data from FITS files'
            'libmariadbclient: MySQL support'
            'openssl: OpenSSL support'
            'postgresql-libs: PostgreSQL support'
            'sqlite: SQLite support'
            'tcsh: Legacy CSH support'
            'unuran: Support non-uniform random numbers'
            'libxml2: XML parser interface'
            'xrootd: Support remote file server and client')
source=("https://root.cern.ch/download/root_v${pkgver}.source.tar.gz"
        'root.xml'
        'rootd'
        'settings.cmake'
        'fix_compile_time_install_clad.patch'
        'adding_directories_needed_to_use_libxml.patch'
        'rename_based_fix_for_rconfig_on_case_sensitive_systems.patch')
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')
get_pyver () {
    python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}
prepare() {
    # cd "${srcdir}/${pkgbase}-${pkgver}"

    2to3 -w "${srcdir}/${pkgbase}-${pkgver}"/etc/dictpch/makepch.py 2>&1 > /dev/null

    patch -d "${srcdir}/${pkgbase}-${pkgver}" -Np1 -i "${srcdir}/fix_compile_time_install_clad.patch"
    patch -d "${srcdir}/${pkgbase}-${pkgver}" -Np1 -i "${srcdir}/adding_directories_needed_to_use_libxml.patch"
    patch -d "${srcdir}/${pkgbase}-${pkgver}" -Np1 -i "${srcdir}/rename_based_fix_for_rconfig_on_case_sensitive_systems.patch"

    # don't let ROOT play around with lib paths
    sed -i -e 's@SetLibraryPath();@@g' \
        "${srcdir}/${pkgbase}-${pkgver}/rootx/src/rootx.cxx"

    # trust system to find GSL
    rm "${srcdir}/${pkgbase}-${pkgver}/cmake/modules/FindGSL.cmake"
}

build() {
    ## ROOT
    mkdir -p "${srcdir}/build"
    cd "${srcdir}/build"

    CFLAGS="$distcc gcc {CFLAGS} -pthread" \
    CXXFLAGS="distcc gcc ${CXXFLAGS} -pthread" \
    LDFLAGS="${LDFLAGS} -pthread -Wl,--no-undefined" \
    cmake -C "${srcdir}/settings.cmake" -DTARGET_ARCHITECTURE:STRING=generic -DPYTHON_EXECUTABLE:PATH=/usr/bin/python \
    "${srcdir}/${pkgbase}-${pkgver}"

    cd "${srcdir}/build"
    make
}

package_root() {
    provides=('root-extra' 'python-pyroot')
    replaces=('root-extra' 'python-pyroot')
    conflicts=('root-extra' 'python-pyroot' 'python2-pyroot')
    optdepends+=('gcc-fortran: Enable the Fortran components of ROOT')
    cd "${srcdir}/build"

    make DESTDIR="${pkgdir}" install

    # fix python env call
    sed -e 's/@python@/python/' -i "${pkgdir}/usr/lib/root/cmdLineUtils.py"

    # try to deal with weird PyROOT, PyMVA and JupyROOT stuff
    install -d "${pkgdir}/usr/lib/python$(get_pyver)/site-packages"
    ln -s "/usr/lib/root/ROOT.py" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/_pythonization.py" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/cmdLineUtils.py" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/cppyy.py" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/JsMVA/" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/JupyROOT/" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/libPyROOT.so" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/libPyMVA.so" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"
    ln -s "/usr/lib/root/libJupyROOT.so" "${pkgdir}/usr/lib/python$(get_pyver)/site-packages/"

    install -D "${srcdir}/rootd" \
        "${pkgdir}/etc/rc.d/rootd"
    install -D -m644 "${srcdir}/root.xml" \
        "${pkgdir}/usr/share/mime/packages/root.xml"

    install -D -m644 "${srcdir}/${pkgbase}-${pkgver}/build/package/debian/root-system-bin.desktop.in" \
        "${pkgdir}/usr/share/applications/root-system-bin.desktop"

    # replace @prefix@ with /usr for the desktop
    sed -e 's_@prefix@_/usr_' -i "${pkgdir}/usr/share/applications/root-system-bin.desktop"

    install -D -m644 "${srcdir}/${pkgbase}-${pkgver}/build/package/debian/root-system-bin.png" \
        "${pkgdir}/usr/share/icons/hicolor/48x48/apps/root-system-bin.png"

    # use a file that pacman can track instead of adding directly to ld.so.conf
    install -d "${pkgdir}/etc/ld.so.conf.d"
    echo '/usr/lib/root' > "${pkgdir}/etc/ld.so.conf.d/root.conf"

    rm -rf "${pkgdir}/etc/root/daemons"
}
