# Maintainer: Robert Schwarz <mail@rschwarz.net>
# Contributor: Johannes Schlatow <johannes.schlatow@googlemail.com>
# Contributor: Stephan Friedrichs <deduktionstheorem@googlemail.com>

pkgname='scipoptsuite'
pkgver='3.0.0'
pkgrel=5
pkgdesc="Tools for generating and solving optimization problems. Consists of ZIMPL, SoPlex, SCIP, GCG and UG"
arch=('i686' 'x86_64')
url='http://zibopt.zib.de/'
license=('LGPL3' 'custom:ZIB Academic License')
depends=('zlib' 'gmp' 'readline')
replaces=('ziboptsuite')
makedepends=('chrpath' 'doxygen' 'graphviz')
provides=('scip=3.0.0' 'soplex=1.7.0' 'zimpl=3.3.0' 'gcg=1.0.0' 'ug=0.7.0')
source=("http://scip.zib.de/download/release/${pkgname}-${pkgver}.tgz"
        'soplex.dxy.patch')
sha256sums=('1ce8a351e92143e1d07d9aa5f9b0f259578f3cee82fcdd984e0024e0d4e3a548'
            '49519d42fccb91806a3e62292c0af102b5748958eea34f552a4e21221990cf89')

build() {
    # Extract directory names from the $provides array.
    local _scip="${provides[0]//=/-}"
    local _soplex="${provides[1]//=/-}"
    local _zimpl="${provides[2]//=/-}"
    local _gcg="${provides[3]//=/-}"
    local _ug="${provides[4]//=/-}"

    cd "${srcdir}/${pkgname}-${pkgver}"

    make SHARED=true

    make gcg
    make ug

    # @TODO: shared lib with ZIMPL seems to be broken
    make scipoptlib ZIMPL=false SHARED=true

    # @TODO: build docs in parallel?

    cd "${srcdir}/${pkgname}-${pkgver}/${_scip}"
    make doc

    cd "${srcdir}/${pkgname}-${pkgver}/${_soplex}"

    # fix soplex.dxy
    # @FIXME: Remove this in the next version
    patch -p1 < ${srcdir}/soplex.dxy.patch
    make doc

    cd "${srcdir}/${pkgname}-${pkgver}/${_gcg}"
    make doc

    # Some files have permission 640.
    # @FIXME: Future versions might not require this line.
    chmod -R a+r "${srcdir}/${pkgname}-${pkgver}"
}

check() {
    cd "${srcdir}/${pkgname}-${pkgver}"
    make test
}

package_scipoptsuite() {
    # Extract directory names from the $provides array
    local _scip="${provides[0]//=/-}"
    local _soplex="${provides[1]//=/-}"
    local _zimpl="${provides[2]//=/-}"
    local _gcg="${provides[3]//=/-}"
    local _ug="${provides[4]//=/-}"

    # Note that, at least in ziboptsuite-2.1.0, the install targets of the
    # scip/soplex/zimpl projects are utterly broken; manually copying
    # everything where it belongs is absolutely necessary.
    # @FIXME: Maybe make install will just work in future releases...

    cd "${srcdir}/${pkgname}-${pkgver}"

    #
    # Binaries
    #
    install -D -m755 ${_scip}/bin/scip "${pkgdir}/usr/bin/scip"
    install -D -m755 ${_soplex}/bin/soplex "${pkgdir}/usr/bin/soplex"
    install -D -m755 ${_zimpl}/bin/zimpl "${pkgdir}/usr/bin/zimpl"
    install -D -m755 ${_gcg}/bin/gcg "${pkgdir}/usr/bin/gcg"
    install -D -m755 ${_ug}/bin/fscip "${pkgdir}/usr/bin/fscip"

    #
    # Includes
    #
    for dir in blockmemshell dijkstra nlpi objscip scip tclique xml; do
        mkdir -p "${pkgdir}/usr/include/scip/${dir}"
        cp ${_scip}/src/${dir}/*.h "${pkgdir}/usr/include/scip/${dir}"
    done

    mkdir -p "${pkgdir}/usr/include/"{soplex,zimpl}
    cp ${_soplex}/src/*.h "${pkgdir}/usr/include/soplex"
    cp ${_zimpl}/src/*.h "${pkgdir}/usr/include/zimpl"

    #
    # Libraries
    #
    mkdir -p "${pkgdir}/usr/lib"
    cp -d ${_scip}/lib/liblpispx* "${pkgdir}/usr/lib"
    cp -d ${_scip}/lib/libnlpi* "${pkgdir}/usr/lib"
    cp -d ${_scip}/lib/libobjscip* "${pkgdir}/usr/lib"
    cp -d ${_scip}/lib/libscip* "${pkgdir}/usr/lib"
    cp -d ${_soplex}/lib/* "${pkgdir}/usr/lib"
    cp -d ${_zimpl}/lib/* "${pkgdir}/usr/lib"
    cp -d lib/libscipopt*.so "${pkgdir}/usr/lib/libscipopt.so"

    # Repair "missing links"
    # @FIXME: I hope this is not necessary in future versions!
    cd "${pkgdir}/usr/lib"
    ln -s -T libzimpl-*.a libzimpl.a
    cd "${srcdir}/${pkgname}-${pkgver}"

    #
    # Documentation
    #
    mkdir -p "${pkgdir}/usr/share/doc/${pkgname}/"{scip,soplex,zimpl,gcg,ug}
    cp -r ${_scip}/{CHANGELOG,release-notes,doc/html} "${pkgdir}/usr/share/doc/${pkgname}/scip/"
    cp -r ${_soplex}/{CHANGELOG,doc/html} "${pkgdir}/usr/share/doc/${pkgname}/soplex/"
    install -m644 ${_soplex}/src/simpleexample.cpp "${pkgdir}/usr/share/doc/${pkgname}/soplex/"
    cp -r ${_zimpl}/{CHANGELOG,README,doc,example} "${pkgdir}/usr/share/doc/${pkgname}/zimpl/"
    cp -r ${_gcg}/{CHANGELOG,README,doc/html} "${pkgdir}/usr/share/doc/${pkgname}/gcg/"
    cp -r ${_ug}/README "${pkgdir}/usr/share/doc/${pkgname}/ug/"

    #
    # License
    #
    install -D -m644 COPYING "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
