# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Metal A-wing <1 at 233 dot email>

pkgname=deno
pkgver=2.7.5
pkgrel=1
_rusty_v8_ver=146.3.0
pkgdesc="A secure runtime for JavaScript and TypeScript"
arch=('x86_64')
url="https://deno.land"
license=('MIT')
depends=('dbus' 'lcms2' 'libffi' 'libgcc' 'zlib' 'zstd')
makedepends=('git' 'python' 'rust' 'nodejs' 'gn' 'ninja' 'clang' 'lld' 'cmake' 'protobuf')
source=("git+https://github.com/denoland/deno.git#tag=v$pkgver"
        "git+https://github.com/denoland/rusty_v8.git#tag=v$_rusty_v8_ver"
        "compiler-rt-adjust-paths.patch")
sha512sums=('eb3ac4f0a3e3e09bf59cb8e1df4a2656213680af4d22019f6c68777f628e3f762e9b589f645c299f258a3a8e0c097ac0428cc7859503ff8fdc3cbe32af23a346'
            '7f003c3d41bace23552e20971979373724282402e47e210e210c3374c074b65b1bf5655ab542bd84fb98e79e7ae6b26511ad969151a5a92c19b550f023d0ca72'
            'b3afc9305c5c7884f66e18e12fafded471cd5842ca39391cb21b19f8009ffb502d3534d93af6fdbce27a6690d5ea94cb94cc20793436c108c83dae5b17af3ffa')

prepare() {
  cd rusty_v8
  git config -f .gitmodules submodule.v8.shallow true
  git submodule update --init --recursive

  # https://github.com/denoland/rusty_v8/issues/1587
  patch -Np1 -i ../compiler-rt-adjust-paths.patch

  cd ../deno
  echo -e "\n[patch.crates-io]\nv8 = { path = '../rusty_v8' }" >> Cargo.toml
  sed -i '/default = \["upgrade", "__vendored_zlib_ng"\]/d; /keyring/s/, "vendored"//' cli/Cargo.toml

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

  export CC=clang CXX=clang++ AR=ar NM=nm
  export V8_FROM_SOURCE=1
  export CLANG_BASE_PATH=/usr
  export GN=/usr/bin/gn NINJA=/usr/bin/ninja
  export EXTRA_GN_ARGS="${_extra_gn_args[@]}"

  export LCMS2_LIB_DIR=/usr/lib
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
