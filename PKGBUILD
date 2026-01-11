# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Metal A-wing <1 at 233 dot email>

pkgname=deno
pkgver=2.6.3
pkgrel=1
_rusty_v8_ver=142.2.0
pkgdesc="A secure runtime for JavaScript and TypeScript"
arch=('x86_64')
url="https://deno.land"
license=('MIT')
options=('!lto')
depends=('gcc-libs')
makedepends=('git' 'python' 'rust' 'nodejs' 'gn' 'ninja' 'clang' 'lld' 'cmake' 'protobuf')
source=("git+https://github.com/denoland/deno.git#tag=v$pkgver"
        "git+https://github.com/denoland/rusty_v8.git#tag=v$_rusty_v8_ver"
        "compiler-rt-adjust-paths.patch")
sha512sums=('5be732c1858dafe1dae8c233eadcbd17dc925c1f7cd662e9f2488bfebdd7c3923472aecbe80ec16c349e394973af328a0ce60fd9645cacb768cb1098eb802dfb'
            '424c1e11a7cb0ffa25e86a089c7f34d065fc984119254cc8ac5680e36f7eba457a5fdd4fc7a84e6bcab6c801c78afb88f22eb214c717a32f90064d2e83384fcb'
            'b3afc9305c5c7884f66e18e12fafded471cd5842ca39391cb21b19f8009ffb502d3534d93af6fdbce27a6690d5ea94cb94cc20793436c108c83dae5b17af3ffa')

prepare() {
  cd rusty_v8
  git config -f .gitmodules submodule.v8.shallow true
  git submodule update --init --recursive

  # https://github.com/denoland/rusty_v8/issues/1587
  patch -Np1 -i ../compiler-rt-adjust-paths.patch

  cd ../deno
  echo -e "\n[patch.crates-io]\nv8 = { path = '../rusty_v8' }" >> Cargo.toml
}

build() {
  cd $pkgname

  # this uses malloc_usable_size, which is incompatible with fortification level 3
  export CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  export CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"

  local _clang_version=$(clang --version | grep -m1 version | sed 's/.* \([0-9]\+\).*/\1/')
  local _extra_gn_args=(
    'custom_toolchain="//build/toolchain/linux/unbundle:default"'
    'host_toolchain="//build/toolchain/linux/unbundle:default"'
    "clang_version=\"$_clang_version\""
  )

  export CC=clang CXX=clang++ AR=ar NM=nm
  export V8_FROM_SOURCE=1
  export CLANG_BASE_PATH=/usr
  export GN=/usr/bin/gn NINJA=/usr/bin/ninja
  export EXTRA_GN_ARGS="${_extra_gn_args[@]}"

  cargo build --release
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
