# Maintainer: Elena (github.com/elenax1d)
# Driver author: jdtanner — https://github.com/jdtanner/elan0c7f-libfprint-driver
#
# libfprint with the experimental driver for the ELAN 04f3:0c7f fingerprint
# sensor (Acer Swift and others). Builds against OpenCV 4 or OpenCV 5.

pkgname=libfprint-elan0c7f-git
_branch=elan-0c7f-1.94.100
pkgver=1.94.100
pkgrel=1
pkgdesc="libfprint with the experimental ELAN 04f3:0c7f fingerprint driver"
arch=(x86_64)
url="https://github.com/jdtanner/elan0c7f-libfprint-driver"
license=(LGPL-2.1-or-later)
depends=(glib2 libgusb libgudev pixman nss systemd-libs opencv)
makedepends=(git meson gtk-doc gobject-introspection systemd)
provides=("libfprint=${pkgver%%.r*}" "libfprint-2.so=2-64")
conflicts=(libfprint)
source=("libfprint::git+https://github.com/jdtanner/libfprint.git#branch=$_branch")
sha256sums=(SKIP)

pkgver() {
  cd libfprint
  printf "1.94.100.r%s.g%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
}

prepare() {
  cd libfprint
  # The driver asks for OpenCV 4 by name; accept OpenCV 5 too.
  sed -i -E "s/dependency\((['\"])opencv4\1/dependency('opencv4', 'opencv5', 'opencv'/" meson.build
  grep -q "dependency('opencv4', 'opencv5'" meson.build || { echo "!! could not patch the OpenCV check"; return 1; }

  # OpenCV 5's pkg-config file lists every module, including opencv_viz (needs vtk).
  # Link only what the driver uses: core, imgproc, features, geometry, flann.
  sed -i "/opencv_dep = dependency('opencv4', 'opencv5'/a\\
        if opencv_dep.found() and opencv_dep.version().version_compare('>=5')\\
            _cxx = meson.get_compiler('cpp')\\
            opencv_dep = declare_dependency(compile_args: ['-I' + opencv_dep.get_variable(pkgconfig: 'includedir_new', default_value: '/usr/include/opencv5')], dependencies: [\\
                _cxx.find_library('opencv_core'), _cxx.find_library('opencv_imgproc'),\\
                _cxx.find_library('opencv_features'), _cxx.find_library('opencv_geometry'),\\
                _cxx.find_library('opencv_flann')])\\
        endif" meson.build
  grep -q "find_library('opencv_core')" meson.build || { echo "!! could not narrow OpenCV"; return 1; }

  # OpenCV 5 renamed the features2d module to "features".
  if ! pkg-config --exists opencv4; then
    grep -rl 'opencv2/features2d.hpp' libfprint | xargs -r sed -i 's#opencv2/features2d.hpp#opencv2/features.hpp#g'
  fi
}

build() {
  arch-meson libfprint build -Ddoc=false
  meson compile -C build
}

package() {
  meson install -C build --destdir "$pkgdir"
}
