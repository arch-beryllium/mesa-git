pkgname=mesa-git
pkgdesc="an open-source implementation of the OpenGL specification, git version"
pkgver=21.1.0_devel.137369.a084d012a9e
pkgrel=1
arch=('x86_64' 'aarch64')
makedepends=('git' 'python-mako' 'xorgproto'
              'libxml2' 'libx11' 'elfutils' 'libxrandr'
              'ocl-icd' 'wayland-protocols' 'meson' 'ninja' 'glslang')
depends=('libdrm' 'libxxf86vm' 'libxdamage' 'libxshmfence' 'libelf'
         'libunwind' 'libglvnd' 'wayland' 'lm_sensors' 'libclc' 'vulkan-icd-loader' 'zstd' 'expat')
optdepends=('opengl-man-pages: for the OpenGL API man pages')
provides=('mesa' 'opencl-mesa' 'vulkan-mesa-layer' 'vulkan-driver' 'opengl-driver' 'opencl-driver')
conflicts=('mesa' 'opencl-mesa' 'vulkan-mesa-layer')
url="https://www.mesa3d.org"
license=('custom')
source=('mesa::git+https://gitlab.freedesktop.org/mesa/mesa.git'
        'LICENSE'
        '0001-clover-llvm13-use-FixedVectorType.patch')
md5sums=('SKIP'
         '5c65a0fe315dd347e09b1f2826a1df5a'
         '5f0620ce35da2d1f80dc1b3c03eafc32')
sha512sums=('SKIP'
            '25da77914dded10c1f432ebcbf29941124138824ceecaf1367b3deedafaecabc082d463abcfa3d15abff59f177491472b505bcb5ba0c4a51bb6b93b4721a23c2'
            'c3a567df69a263c6b87d9c7887464ed03b5a0bf008caf180c45cf50dca3f4710e0eda3e453a7a69b89720e333c33b9aa7da1b7f9c79d72bb76d847b567558d19')

# NINJAFLAGS is an env var used to pass commandline options to ninja
# NOTE: It's your responbility to validate the value of $NINJAFLAGS. If unsure, don't set it.

# MESA_WHICH_LLVM is an environment variable that determines which llvm package tree is used to built mesa-git against.
# Adding a line to ~/.bashrc  that sets this value is the simplest way to ensure a specific choice.
#
# NOTE: Aur helpers don't handle this method well, check the sticky comments on mesa-git aur page .
#
# 1: llvm-minimal-git (aur) preferred value
# 2: AUR llvm-git
# 3: llvm-git from LordHeavy unofficial repo
# 4  llvm (stable from extra) Default value
#

if [[ ! $MESA_WHICH_LLVM ]] ; then
    MESA_WHICH_LLVM=4
fi

case $MESA_WHICH_LLVM in
    1)
        # aur llvm-minimal-git
        makedepends+=('llvm-minimal-git')
        depends+=('llvm-libs-minimal-git')
        optdepends+=('llvm-minimal-git: opencl')
        ;;
    2)
        # aur llvm-git
        # depending on aur-llvm-* to avoid mixup with LH llvm-git
        makedepends+=('aur-llvm-git')
        depends+=('aur-llvm-libs-git')
        optdepends+=('aur-llvm-git: opencl')
        ;;
    3)
        # mesa-git/llvm-git (lordheavy unofficial repo)
        makedepends+=('llvm-git' 'clang-git')
        depends+=('llvm-libs-git')
        optdepends+=('clang-git: opencl' 'compiler-rt: opencl')
        ;;
    4)
        # extra/llvm
        makedepends+=(llvm=11.1.0 clang=11.1.0)
        depends+=(llvm-libs=11.1.0)
        optdepends+=('clang: opencl' 'compiler-rt: opencl')
        ;;
    *)
esac





pkgver() {
    cd mesa
    read -r _ver <VERSION
    echo ${_ver/-/_}.$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
}

prepare() {
    # although removing _build folder in build() function feels more natural,
    # that interferes with the spirit of makepkg --noextract
    if [  -d _build ]; then
        rm -rf _build
    fi
    cd mesa
    patch --forward --strip=1 --input="${srcdir}/0001-clover-llvm13-use-FixedVectorType.patch"
}

build () {
    meson setup mesa _build \
       -D b_ndebug=false \
       -D b_lto=false \
       --buildtype debug \
       --wrap-mode=nofallback \
       -D prefix=/usr \
       -D sysconfdir=/etc \
       -D platforms=x11,wayland \
       -D gallium-drivers=freedreno \
       -D vulkan-drivers=freedreno \
       -D dri3=enabled \
       -D egl=enabled \
       -D gallium-extra-hud=true \
       -D gallium-xa=enabled \
       -D gallium-xvmc=disabled \
       -D gbm=enabled \
       -D gles1=disabled \
       -D gles2=enabled \
       -D glvnd=true \
       -D glx=dri \
       -D libunwind=enabled \
       -D llvm=enabled \
       -D lmsensors=enabled \
       -D shared-glapi=enabled \
       -D gallium-opencl=icd \
       -D valgrind=disabled \
       -D vulkan-overlay-layer=true \
       -D vulkan-device-select-layer=true \
       -D tools=[] \
       -D zstd=enabled \
       -D microsoft-clc=disabled

    meson configure _build

    ninja $NINJAFLAGS -C _build
}

package() {
    DESTDIR="${pkgdir}" ninja $NINJAFLAGS -C _build install

    # indirect rendering
    ln -s /usr/lib/libGLX_mesa.so.0 "${pkgdir}/usr/lib/libGLX_indirect.so.0"

    install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" "${srcdir}/LICENSE"
}
