# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Metal A-wing <1 at 233 dot email>

pkgname=deno
pkgver=2.9.6
pkgrel=1
_rusty_v8_ver=150.4.0
pkgdesc="A secure runtime for JavaScript and TypeScript"
arch=('x86_64')
url="https://deno.com"
license=('MIT')
depends=('dbus' 'lcms2' 'libffi' 'libgcc' 'sqlite' 'wayland' 'zlib' 'zstd')
makedepends=('git' 'python' 'rust' 'nodejs' 'gn' 'ninja' 'clang' 'lld' 'cmake' 'protobuf')
source=("git+https://github.com/denoland/deno.git#tag=v$pkgver"
        "git+https://github.com/denoland/rusty_v8.git#tag=v$_rusty_v8_ver"
        "compiler-rt-adjust-paths.patch")
sha512sums=('cc4e68c4c24c0fa383d5319fdac1f019e3a55deeb41bda4eecde032e392f4a392d23d7cefb7a9b0c88b2c7d2d6e0086c7c6f69c2482d13a54aecd00f00f631a6'
            'ae0d6d585cf7ba0172930d09e3d7a2d4bb5d748409e86b44dfa5a12741a51aab138ab12ca8327ac6a36506b9caf6a20fed20f0efd5b7fdf145bd8fdca20f5ed0'
            '8a782d68a6140f739f00d3eb341d742584ee0be80e85e89bc1540a21d15ad8b75274672ebd02e1e4fd1925ed9ca68b05142388e795dff81b0a864d38f5514253')

prepare() {
  cd rusty_v8
  git config -f .gitmodules submodule.v8.shallow true
  git submodule update --init --recursive

  # Drop flags rejected by the clang++ invoked in our build environment.
  sed -i \
    -e '/-fno-lifetime-dse/d' \
    -e '/-fdiagnostics-show-inlining-chain/d' \
    build/config/compiler/BUILD.gn
  sed -i '/-fsanitize-ignore-for-ubsan-feature=/d' build/config/sanitizers/sanitizers.gni

  # https://github.com/denoland/rusty_v8/issues/1587
  patch -Np1 -i ../compiler-rt-adjust-paths.patch

  cd ../deno
  echo -e "\n[patch.crates-io]\nv8 = { path = '../rusty_v8' }" >> Cargo.toml
  # Keep the V8 backend while disabling Deno's self-upgrade and vendored zlib-ng.
  sed -i \
    -e '/^v8 = \[/s/"__runtime_defaults", //' \
    -e '/^keyring =/s/, "vendored"//' \
    cli/Cargo.toml

  cargo fetch --target host-tuple
}

build() {
  cd $pkgname

  # this uses malloc_usable_size, which is incompatible with fortification level 3
  export CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  export CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"

  local _clang_version=$(clang -dumpversion | cut -d '.' -f 1)
  local _extra_gn_args=(
    'custom_toolchain="//build/toolchain/linux/unbundle:default"'
    'host_toolchain="//build/toolchain/linux/unbundle:default"'
    "clang_version=\"$_clang_version\""
    'use_system_libffi=true'
  )

  export CC=clang CXX=clang++ AR=/usr/bin/ar NM=nm
  export BUILD_CC=clang BUILD_CXX=clang++ BUILD_AR=/usr/bin/ar BUILD_NM=nm
  export V8_FROM_SOURCE=1
  export CLANG_BASE_PATH=/usr
  export GN=/usr/bin/gn NINJA=/usr/bin/ninja
  export EXTRA_GN_ARGS="${_extra_gn_args[@]}"

  export LCMS2_LIB_DIR=/usr/lib
  export LIBSQLITE3_SYS_USE_PKG_CONFIG=1
  export ZSTD_SYS_USE_PKG_CONFIG=1
  export CARGO_FEATURE_SYSTEM=1 # Use system-provided libffi

  cargo build --frozen --release
}

check() {
  cd $pkgname
  ./target/release/deno run tests/testdata/run/002_hello.ts
}

package() {
  cd $pkgname
  install -Dm755 target/release/deno "$pkgdir"/usr/bin/deno

  install -dm755 "$pkgdir"/usr/share/bash-completion/completions
  ./target/release/deno completions bash > "$pkgdir"/usr/share/bash-completion/completions/deno
  install -dm755 "$pkgdir"/usr/share/zsh/site-functions
  ./target/release/deno completions zsh > "$pkgdir"/usr/share/zsh/site-functions/_deno
  install -dm755 "$pkgdir"/usr/share/fish/vendor_completions.d
  ./target/release/deno completions fish > "$pkgdir"/usr/share/fish/vendor_completions.d/deno.fish

  install -Dm644 LICENSE.md -t "$pkgdir"/usr/share/licenses/$pkgname/
}
