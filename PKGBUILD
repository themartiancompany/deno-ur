# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Metal A-wing <1 at 233 dot email>

pkgname=deno
pkgver=1.36.1
_commit=9f1455b75087e9e41a995bbb6a8a7f3059a0185a
pkgrel=1
pkgdesc="A secure runtime for JavaScript and TypeScript"
arch=('x86_64')
url="https://deno.land"
license=('MIT')
options=('!lto')
depends=('gcc-libs')
makedepends=('git' 'python' 'rust' 'nodejs' 'cmake')
source=("git+https://github.com/denoland/deno.git#commit=$_commit")
sha512sums=('SKIP')

prepare() {
  cd $pkgname
  # https://github.com/denoland/deno/issues/19528
  git cherry-pick -n c8dc6b14ec5c1b6de28118ed3b07d037eaaaf702
}

build() {
  cd $pkgname
  cargo build --release
}

check() {
  cd $pkgname
  ./target/release/deno run cli/tests/testdata/run/002_hello.ts
}

package() {
  cd $pkgname
  install -Dm755 target/release/deno "$pkgdir"/usr/bin/deno

  install -dm755 "$pkgdir"/usr/share/bash-completion/completions
  ./target/release/deno completions bash > "$pkgdir"/usr/share/bash-completion/completions/deno
  install -dm755 "$pkgdir"/usr/share/zsh/site-functions
  ./target/release/deno completions zsh > "$pkgdir"/usr/share/zsh/site-functions/_deno
  install -dm755 "$pkgdir"/usr/share/fish/vendor_functions.d
  ./target/release/deno completions fish > "$pkgdir"/usr/share/fish/vendor_functions.d/deno.fish

  install -Dm644 LICENSE.md -t "$pkgdir"/usr/share/licenses/$pkgname/
}
