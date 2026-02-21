# Maintainer: pussyhut <pussyhut@icloud.com>
# Contributor: envolution
# Contributor: txtsd <aur.archlinux@ihavea.quest>
# shellcheck shell=bash source=/dev/null disable=SC2034,SC2154
# ci|prebuild=setcommitid.sh,envset_aur_llamacpp_build_universal=true| https://github.com/envolution/aur/blob/main/maintain/build/llama.cpp-cuda/setcommitid.sh

: "${aur_llamacpp_build_universal:=false}"

pkgname=llama.cpp-cuda
_pkgname="${pkgname%-cuda}"
pkgver=b8121
pkgrel=1
_build_number=8121
_commit_id=a0c91e8
pkgdesc="Port of Facebook's LLaMA model in C/C++ (with NVIDIA CUDA optimizations)"
arch=(x86_64 armv7h aarch64 loongarch64)
url='https://github.com/ggerganov/llama.cpp'
license=('MIT')
depends=(
  cuda
  curl
  gcc-libs
  glibc
  nvidia-utils
)
makedepends=(
  cmake
  cuda
)
optdepends=(
  'python-numpy: needed for convert_hf_to_gguf.py'
  'python-safetensors: needed for convert_hf_to_gguf.py'
  'python-sentencepiece: needed for convert_hf_to_gguf.py'
  'python-pytorch: needed for convert_hf_to_gguf.py'
  'python-transformers: needed for convert_hf_to_gguf.py'
)
provides=("${_pkgname}")
conflicts=("${_pkgname}" libggml ggml)
replaces=(llama.cpp-cuda-f16)
source=(
  "${pkgname}-${pkgver}.tar.gz::https://github.com/ggml-org/llama.cpp/archive/refs/tags/${pkgver}.tar.gz"
  llama.cpp.conf
  llama.cpp.service
)
sha256sums=('62c72953b2a3563e34076fa3fd4d17ba51e208cbfda9457528e74999cd60433d'
            '53fa70cfe40cb8a3ca432590e4f76561df0f129a31b121c9b4b34af0da7c4d87'
            '38c65c7a50f21fe5423a2034386f47983db49fadee4860caed3a49eee128c4c0')
backup=(
  etc/conf.d/llama.cpp
)

prepare() {
  ln -sf "${_pkgname}-${pkgver}" llama.cpp
}
build() {
  # This may not be set if the user's session
  # has not restarted on a new 'cuda' install
  if [[ -z "${NVCC_CCBIN}" ]]; then
    source /etc/profile
  fi

  local _cmake_options=(
    -B build
    -S "${_pkgname}"
    -DCMAKE_BUILD_TYPE=Release
    -DCMAKE_INSTALL_PREFIX='/usr'
    -DBUILD_SHARED_LIBS=ON
    -DLLAMA_BUILD_TESTS=OFF
    -DLLAMA_USE_SYSTEM_GGML=OFF
    -DGGML_ALL_WARNINGS=OFF
    -DGGML_ALL_WARNINGS_3RD_PARTY=OFF
    -DGGML_BUILD_EXAMPLES=OFF
    -DGGML_BUILD_TESTS=OFF
    -DGGML_LTO=ON
    -DGGML_RPC=ON
    -DGGML_CUDA=ON
    -DGGML_BUILD_SERVER=ON
    -DLLAMA_BUILD_NUMBER="${_build_number}"
    -DLLAMA_BUILD_COMMIT="${_commit_id}"
    -Wno-dev
  )
  if [[ ${aur_llamacpp_build_universal} == true ]]; then
    echo "Building universal binary [aur_llamacpp_build_universal == true]"
    _cmake_options+=(
      -DGGML_BACKEND_DL=ON
      -DGGML_NATIVE=OFF
      -DGGML_CPU_ALL_VARIANTS=ON
    )
  else
    # we lose GGML_NATIVE_DEFAULT due to how makepkg includes
    # $SOURCE_DATE_EPOCH in ENV
    _cmake_options+=(
      -DGGML_NATIVE=ON
    )
  fi
  # Allow user-specified additional flags
  if [[ -n "${aur_llamacpp_cmakeopts:-}" ]]; then
    echo "Applying custom CMake options: ${aur_llamacpp_cmakeopts}"
    # shellcheck disable=SC2206 # intentional word splitting
    _cmake_options+=(${aur_llamacpp_cmakeopts})
  fi
  cmake "${_cmake_options[@]}"
  cmake --build build
}

package() {
  DESTDIR="${pkgdir}" cmake --install build

  install -Dm644 "llama.cpp.conf" "${pkgdir}/etc/conf.d/llama.cpp"
  install -Dm644 "llama.cpp.service" "${pkgdir}/usr/lib/systemd/system/llama.cpp.service"
  install -Dm644 "${_pkgname}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
# vim:set ts=2 sw=2 et:
