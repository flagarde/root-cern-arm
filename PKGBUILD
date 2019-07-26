# Maintainer: Konstantin Gizdov < arch at kge dot pw >
# Contributor: Frank Siegert < frank.siegert at googlemail dot com >
# Contributor: Scott Lawrence < bytbox at gmail dot com >
# Contributor: Thomas Dziedzic < gostrc at gmail dot com >
# Contributor: Sebastian Voecking < voeck at web dot de >

pkgbase=root
pkgname=('root')
pkgver=6.18.00
pkgrel=2
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
             'gcc'
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
         'graphviz'
         'gsl'
         'hicolor-icon-theme'
         'intel-tbb'
         'libafterimage'
         'librsvg'
         'libxpm'
         'tex-gyre-fonts'
         'unixodbc'
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
        )
sha256sums=('e6698d6cfe585f186490b667163db65e7d1b92a2447658d77fa831096383ea71'
            'SKIP'
            '3c45b03761d5254142710b7004af0077f18efece7c95511910140d0542c8de8a'
            'SKIP'
            )
get_pyver () {
    python -c 'import sys; print(str(sys.version_info[0]) + "." + str(sys.version_info[1]))'
}
prepare() {
    # cd "${srcdir}/${pkgbase}-${pkgver}"

    2to3 -w "${srcdir}/${pkgbase}-${pkgver}"/etc/dictpch/makepch.py 2>&1 > /dev/null

    # don't let ROOT play around with lib paths
    sed -i -e 's@SetLibraryPath();@@g' \
        "${srcdir}/${pkgbase}-${pkgver}/rootx/src/rootx.cxx"

    cp -r "${pkgbase}-${pkgver}" "${pkgbase}-${pkgver}-cuda"
}

build() {
    ## ROOT
    mkdir -p "${srcdir}/build"
    cd "${srcdir}/build"

    CFLAGS="${CFLAGS} -pthread" \
    CXXFLAGS="${CXXFLAGS} -pthread" \
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

    install -D -m644 "${srcdir}/${pkgbase}-${pkgver}/etc/root.desktop" \
        "${pkgdir}/usr/share/applications/root.desktop"

    install -D -m644 "${srcdir}/${pkgbase}-${pkgver}/icons/Root6Icon.png" \
        "${pkgdir}/usr/share/icons/hicolor/48x48/apps/root.png"
    echo 'Icon=root.png' >> "${pkgdir}/usr/share/applications/root.desktop"

    # use a file that pacman can track instead of adding directly to ld.so.conf
    install -d "${pkgdir}/etc/ld.so.conf.d"
    echo '/usr/lib/root' > "${pkgdir}/etc/ld.so.conf.d/root.conf"

    rm -rf "${pkgdir}/etc/root/daemons"
}

