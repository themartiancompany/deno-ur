# SPDX-License-Identifier: AGPL-3.0

#    -----------------------------------------------------
#    Copyright © 2024, 2025, 2026  Pellegrino Prevete
#
#    All rights reserved
#    -----------------------------------------------------
#
#    This program is free software: you can redistribute
#    it and/or modify it under the terms of the
#    GNU Affero General Public License as published by
#    the Free Software Foundation, either version 3 of
#    the License, or (at your option) any later version.
#
#    This program is distributed in the hope that it
#    will be useful, but WITHOUT ANY WARRANTY;
#    without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
#    See the GNU Affero General Public License for
#    more details.
#
#    You should have received a copy of the
#    GNU Affero General Public License
#    along with this program.
#    If not, see <https://www.gnu.org/licenses/>.

# Maintainers:
#   Truocolo
#     <truocolo@aol.com>
#     <truocolo@0x6E5163fC4BFc1511Dbe06bB605cc14a3e462332b>
#   Pellegrino Prevete (dvorak)
#     <pellegrinoprevete@gmail.com>
#     <dvorak@0x87003Bd6C074C713783df04f36517451fF34CBEf>
# Contributors:
#   Felix Yan
#     <felixonmars@archlinux.org>
#   Metal A-wing
#     <1 at 233 dot email>


if [[ ! -v "_git_service" ]]; then
  _git_service="github"
fi
_pkg=deno
if [[ ! -v "_ns" ]]; then
  _ns="${_pkg}land"
  _ns="themartiancompany"
fi
pkgbase="${_pkg}"
pkgname=(
  "${pkgbase}"
)
pkgver=2.9.6
pkgrel=2
_rusty_v8_ver=150.4.0
pkgdesc="A secure runtime for JavaScript and TypeScript"
arch=(
  "aarch64"
  "arm"
  "armv7l"
  "armv8l"
  "i686"
  "mips"
  "pentium4"
  "powerpc"
  "x86_64"
)
url="https://${_pkg}.com"
_http="https://${_git_service}.com"
_url="${_http}/${_ns}/${_pkg}"
license=(
  'MIT'
)
depends=(
  'dbus'
  'lcms2'
  'libffi'
  'libgcc'
  'sqlite'
  'wayland'
  'zlib'
  'zstd'
)
makedepends=(
  'git'
  'python'
  'rust'
  'rust-bindgen'
  'nodejs'
  'gn'
  'ninja'
  'clang'
  'lld'
  'cmake'
  'protobuf'
)
_uri="git+${_url}.git#tag=v${pkgver}"
_rusty_uri="git+${_http}/${_ns}/rusty_v8.git#tag=v${_rusty_v8_ver}"
_src="${_pkg}::${_uri}"
source=(
  "${_src}"
  "rusty_v8::${_rusty_uri}"
  "compiler-rt-adjust-paths.patch"
  "${_pkg}-2.9.6-rust-system.patch"
)
sha512sums=(
  'cc4e68c4c24c0fa383d5319fdac1f019e3a55deeb41bda4eecde032e392f4a392d23d7cefb7a9b0c88b2c7d2d6e0086c7c6f69c2482d13a54aecd00f00f631a6'
  'ae0d6d585cf7ba0172930d09e3d7a2d4bb5d748409e86b44dfa5a12741a51aab138ab12ca8327ac6a36506b9caf6a20fed20f0efd5b7fdf145bd8fdca20f5ed0'
  '8a782d68a6140f739f00d3eb341d742584ee0be80e85e89bc1540a21d15ad8b75274672ebd02e1e4fd1925ed9ca68b05142388e795dff81b0a864d38f5514253'
  '658e32634fc7463f79099d19e2e54ef59318811209d1e5f7adf5caae9b57ad06dfe9bbe41d272c47d92cd0ad1c746cc0e04775c495c8e120cbd62c88239d1945'
)

prepare() {
  cd \
    "rusty_v8"
  git \
    config \
    -f \
      ".gitmodules" \
    "submodule.v8.shallow" \
    true
  git \
    submodule \
      update \
        --init \
	--recursive
  # Use system-provided Rust toolchain for V8 build
  sed \
    -i \
    '/download_rust_toolchain();/d' \
    "build.rs"
  patch \
    -Np1 \
    -i \
    "../${_pkg}-2.9.6-rust-system.patch"
  # Drop flags rejected by the clang++
  # invoked in our build environment.
  sed \
    -i \
    -e \
      '/-fno-lifetime-dse/d' \
    -e \
      '/-fdiagnostics-show-inlining-chain/d' \
    "build/config/compiler/BUILD.gn"
  sed \
    -i \
    '/-fsanitize-ignore-for-ubsan-feature=/d' \
    "build/config/sanitizers/sanitizers.gni"
  # https://github.com/denoland/rusty_v8/issues/1587
  patch \
    -Np1 \
    -i \
    "../compiler-rt-adjust-paths.patch"
  cd \
    "../${_pkg}"
  echo \
    -e \
    "\n[patch.crates-io]\nv8 = { path = '../rusty_v8' }" >> \
    "Cargo.toml"
  # Keep the V8 backend while disabling
  # Deno's self-upgrade and vendored zlib-ng.
  sed \
    -i \
    -e \
      '/^v8 = \[/s/"__runtime_defaults", //' \
    -e \
      '/^keyring =/s/, "vendored"//' \
    "cli/Cargo.toml"
  cargo \
    fetch \
    --target \
      "host-tuple"
}

build() {
  local \
    _extra_gn_args=() \
    _clang_version \
    _rustc_version
  cd \
    "${_pkg}"
  # this uses malloc_usable_size,
  # which is incompatible with fortification level 3
  export \
    CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  export \
    CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  _clang_version="$(
    clang \
      -dumpversion |
      cut \
        -d \
          '.' \
        -f \
          1)"
  _rustc_version="$(
    rustc \
      --version |
      awk \
        '{ print $2 ;}')"
  _extra_gn_args+=(
    'custom_toolchain="//build/toolchain/linux/unbundle:default"'
    'host_toolchain="//build/toolchain/linux/unbundle:default"'
    "clang_version=\"${_clang_version}\""
    'rust_sysroot_absolute="/usr"'
    'rust_bindgen_root="/usr"'
    "rustc_version=\"${_rustc_version}\""
    'use_sysroot=false'
    'use_system_libffi=true'
  )
  export \
    CC="clang" \
    CXX="clang++" \
    AR="/usr/bin/ar" \
    NM="nm"
  export \
    BUILD_CC="clang" \
    BUILD_CXX="clang++" \
    BUILD_AR="/usr/bin/ar" \
    BUILD_NM="nm"
  export \
    V8_FROM_SOURCE=1 \
    CLANG_BASE_PATH="/usr"
    GN="/usr/bin/gn" \
    NINJA="/usr/bin/ninja"
  export \
    EXTRA_GN_ARGS="${_extra_gn_args[@]}"
  export \
    LCMS2_LIB_DIR="/usr/lib" \
    LIBSQLITE3_SYS_USE_PKG_CONFIG=1 \
    ZSTD_SYS_USE_PKG_CONFIG=1
  # Use system-provided libffi
  export \
    CARGO_FEATURE_SYSTEM=1 \
    RUSTC_BOOTSTRAP=1
  cargo \
    build \
      --frozen \
      --release
}

check() {
  cd \
    "${_pkg}"
  "./target/release/${_pkg}" \
    run \
      "tests/testdata/run/002_hello.ts"
}

package() {
  cd \
    "${_pkg}"
  install \
    -vDm755 \
    "target/release/${_pkg}" \
    "${pkgdir}/usr/bin/${_pkg}"
  install \
    -vdm755 \
    "${pkgdir}/usr/share/bash-completion/completions"
  "./target/release/deno" \
    completions \
      "bash" > \
    "${pkgdir}/usr/share/bash-completion/completions/${_pkg}"
  install \
    -vdm755 \
    "${pkgdir}/usr/share/zsh/site-functions"
  "./target/release/deno" \
    completions \
      "zsh" > \
    "${pkgdir}/usr/share/zsh/site-functions/_${_pkg}"
  install \
    -vdm755 \
    "${pkgdir}/usr/share/fish/vendor_completions.d"
  "./target/release/deno" \
    completions \
      "fish" > \
    "${pkgdir}/usr/share/fish/vendor_completions.d/deno.fish"
  install \
    -vDm644 \
    "LICENSE.md" \
    -t \
    "${pkgdir}/usr/share/licenses/${pkgname}/"
}
