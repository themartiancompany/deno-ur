# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Metal A-wing <1 at 233 dot email>

pkgname=deno
pkgver=2.7.8
pkgrel=1
_rusty_v8_ver=147.0.0
pkgdesc="A secure runtime for JavaScript and TypeScript"
arch=('x86_64')
url="https://deno.land"
license=('MIT')
depends=('dbus' 'lcms2' 'libffi' 'libgcc' 'zlib' 'zstd')
makedepends=('git' 'python' 'rust' 'nodejs' 'gn' 'ninja' 'clang' 'lld' 'cmake' 'protobuf')
source=("git+https://github.com/denoland/deno.git#tag=v$pkgver"
        "git+https://github.com/denoland/rusty_v8.git#tag=v$_rusty_v8_ver"
        "compiler-rt-adjust-paths.patch")
sha512sums=('e8cbc5e32dd0ac7eb6add637a0d1f0917fa28239370a79d75b210b119491d7eb131d1a2e49c1b6ed4fe57f82fb6b477c0d43d1c20cf2abd8b95779e18df801f1'
            '89ba1a1261ee7424d16902d1624d946ea5cbfeaa666fcf414fada0e0827879009c349f1cf594fcfb28a110fe1ef00753bbfcc9d27482655bfb1cc7619d9a0c39'
            'b3afc9305c5c7884f66e18e12fafded471cd5842ca39391cb21b19f8009ffb502d3534d93af6fdbce27a6690d5ea94cb94cc20793436c108c83dae5b17af3ffa')

prepare() {
  cd rusty_v8
  git config -f .gitmodules submodule.v8.shallow true
  git submodule update --init --recursive

  # Drop flags rejected by the clang++ invoked in our build environment.
  sed -i \
    -e '/-fno-lifetime-dse/d' \
    -e '/-fsanitize-ignore-for-ubsan-feature=array-bounds/d' \
    build/config/compiler/BUILD.gn

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
