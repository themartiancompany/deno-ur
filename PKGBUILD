# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Metal A-wing <1 at 233 dot email>

pkgname=deno
pkgver=2.5.2
pkgrel=1
_rusty_v8_ver=140.2.0
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
sha512sums=('3e64c8b47aedc8442737afa19ae3112453f97a1a742a313cecf5894bc2a61154d18157537f51f87109fcdb9779e7694d574fbd4557285b343fcc1c4cc6599052'
            '8af470ec317894e71a632824ad1b6f3fbaec18147e947697e9eea2ab7f3439568f5f59994dd17586cc9658dff77d16d365c4676eef70f5aa0de3f73193d7575f'
            'e0409a66fc27268f49303f415fb14cb78d0f96da82da2aee788c879605a1626715c4a743377ccd2fd7567becf867471e8002bdecbaddf1c85411d62b1eb334c8')

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
