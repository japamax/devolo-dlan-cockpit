# Maintainer: japama <9z4jprfo@duck.com>
# Contributor: japama <9z4jprfo@duck.com> 01/11/2026

pkgname=devolo-dlan-cockpit
pkgver=5.2.0.185
pkgrel=2.5


_name=devolo-cockpit-v${pkgver//./-}-linux.run
install=${pkgname}.install
pkgdesc="Display and configure settings of your devolo device"
arch=('i686' 'x86_64')
url="https://www.devolo.de/de/software-downloads/cockpit"
license=('nonfree')
depends=( 'lib32-gtk2' 'lib32-curl' 'lib32-libxt' 'lib32-libxslt' 'lib32-nss' )

if [ "${CARCH}" = "x86_64" ]; then
  _arch="amd64"
  makedepends=( 'gcc' 'lib32-glibc' 'lib32-gcc-libs' )
else
  _arch="i386" 
fi 

source=("https://devolo.scalecommerce.cloud/media/62/e1/13/1758704956/${_name}" 'devolonetsvc.service')

#source=("https://www.devolo.fr/fileadmin/Web-Content/DE/products/hnw/devolo-cockpit/software/$_name" 'devolonetsvc.service')

build() {
  cd $srcdir
  skip=$(grep -a -m1 -n "HERE_BE_DRAG[O]NS" $_name | cut -d: -f1)
  tail $_name -n +$((skip+1)) | tar -x -C .
  ar x "devolo-dlan-cockpit_${pkgver}-0_${_arch}.deb"
  find . -name "adobeair*${_arch}.deb" -print | xargs ar x
  tar xJf data.tar.xz
  tar xJf data.tar.lzma
  sed -i 's/\.appdata\//~\/\.appdata\//g' "${srcdir}/opt/devolo/dlancockpit/bin/dlancockpit-run.sh"
  echo "StartupWMClass=dlancockpit" >> "${srcdir}/usr/share/applications/devolo-dlan-cockpit.desktop"

  # libCore.so (Adobe AIR 2.6) référence la variable xmlLoadExtDtdDefaultValue,
  # absente du libxml2 32 bits actuel : dlopen échoue. Sur x86_64, on compile
  # un shim 32 bits qui la définit.
  if [ "${CARCH}" = "x86_64" ]; then
    echo 'int xmlLoadExtDtdDefaultValue = 0;' > xmlshim.c
    gcc -m32 -shared -fPIC -O2 xmlshim.c -o libxmlshim.so
    [ -s libxmlshim.so ] || { echo "ERREUR : libxmlshim.so introuvable après compilation" >&2; return 1; }
  fi

  # Le script de lancement n'utilise le shim qu'en x86_64, et seulement s'il est présent
  # (sinon : avertissement sur stderr, lancement sans shim). Le LD_PRELOAD ne vise que
  # le processus dlancockpit, pas la boucle xdg-open.
  _run="${srcdir}/opt/devolo/dlancockpit/bin/dlancockpit-run.sh"
  cat > shim-preamble.sh <<'EOF'
# libCore.so (Adobe AIR 2.6) a besoin du shim xmlLoadExtDtdDefaultValue sur x86_64
_shim=
if [ "$(uname -m)" = "x86_64" ]; then
  _shim=/opt/devolo/dlancockpit/bin/libxmlshim.so
  [ -f "$_shim" ] || { echo "devolo-dlan-cockpit: $_shim introuvable, lancement sans shim" >&2; _shim=; }
fi
EOF
  sed -i '1r shim-preamble.sh' "${_run}"
  sed -i 's|^/opt/devolo/dlancockpit/bin/dlancockpit |${_shim:+env LD_PRELOAD=$_shim} &|' "${_run}"
  grep -q '_shim:+env' "${_run}" || { echo "ERREUR : lancement avec shim non inséré dans dlancockpit-run.sh" >&2; return 1; }
}

package() {
  cp -r  "${srcdir}/etc/opt" "${srcdir}/opt" "${srcdir}/usr" "${srcdir}/var" "${pkgdir}/"
  mkdir -p "${pkgdir}/var/lib/devolonetsvc"
  printf "<?xml version="1.0" encoding="utf-8"?>\n<data_collection><allowed>2</allowed></data_collection>" > "${srcdir}/config.xml"
  install -Dm644 "${srcdir}/config.xml" "${pkgdir}/var/lib/devolonetsvc/config.xml"  
  install -Dm644 "${srcdir}/devolonetsvc.service" "${pkgdir}/usr/lib/systemd/system/devolonetsvc.service"
  if [ "${CARCH}" = "x86_64" ]; then
    install -Dm755 "${srcdir}/libxmlshim.so" "${pkgdir}/opt/devolo/dlancockpit/bin/libxmlshim.so"
  fi
}

sha256sums=('976099b5aa4785edc9ceabb6246abe724b78d2e717d1906eb91bf137f585a1e6'
            '6f187ca5c7a599b5394ea09cd68885168dbd19b5bd72df5ce083e721e2f0a12c')
