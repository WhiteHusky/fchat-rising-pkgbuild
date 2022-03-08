# Maintainer: Carlen White <whitersuburban@gmail.com>
pkgname=fchat-rising-git
_pkgname=fchat-rising
pkgver=1.15.1
pkgrel=4
pkgdesc="The F-Chat 3.0 client from F-List modifed by MrStallion. Uses a system-wide electron instead of the built in."
arch=('x86_64')
url="https://github.com/mrstallion/fchat-rising"
license=('MIT')
depends=(
    'electron13'
    'libappindicator-gtk3'
    'libindicator-gtk3'
    'libnotify'
    'libxss'
    'libxtst'
    'libsecret'
)
makedepends=('yarn' 'nvm' 'node-gyp' 'python2')
provides=('fchat-3.0')
source=(
    'fchat::git+https://github.com/mrstallion/fchat-rising#tag=v1.15.1'
    'local://fchat.desktop'
)
sha256sums=('SKIP' 'eaa27f1eb8bd228e9bd11a1cd068f30b3129abce85ab9f275de34dbf60ba8fba')

prepare() {
    source /usr/share/nvm/init-nvm.sh
    nvm install v14
    nvm use v14
}

build() {
    source /usr/share/nvm/init-nvm.sh
    nvm use v14
    cd $srcdir/fchat/
    HOME="$srcdir/.node" yarn install
    cd $srcdir/fchat/electron
    HOME="$srcdir/.node" yarn build:dist
    export SKIP_INSTALLER=TRUE
    node ./pack
    sed -i "s|Exec=.*|Exec=/usr/bin/$_pkgname|" "$srcdir"/fchat.desktop
    sed -i "s|Icon=.*|Icon=/usr/share/pixmaps/$_pkgname.png|" "$srcdir"/fchat.desktop
}

package() {
    export SKIP_INSTALLER=TRUE
    source /usr/share/nvm/init-nvm.sh
    nvm use v14
    cd $srcdir/fchat/electron
    HOME="$srcdir/.node" yarn run pack
    install -d "$pkgdir"/usr/lib/$_pkgname
    cp -r "$srcdir"/fchat/electron/dist/F-Chat-linux-x64/resources/app/* "$pkgdir"/usr/lib/$_pkgname
    rm "$srcdir"/$_pkgname || true
    echo "#!/bin/sh" >> "$srcdir"/$_pkgname
    # F-Chat needs to executed within it's directory
    echo "cd /usr/lib/$_pkgname" >> "$srcdir"/$_pkgname
    echo "exec electron13 ./ \$@" >> "$srcdir"/$_pkgname
    install -d "$pkgdir"/usr/{bin,share/{pixmaps,applications}}
    install -Dm 755 "$srcdir"/$_pkgname "$pkgdir"/usr/bin/$_pkgname
    cp "$srcdir"/fchat/electron/build/icon.png "$pkgdir"/usr/share/pixmaps/$_pkgname.png
    cp "$srcdir"/fchat.desktop "$pkgdir"/usr/share/applications/$_pkgname.desktop

    install -Dm 644 "$srcdir"/fchat/LICENSE "$pkgdir"/usr/share/licenses/$_pkgname/LICENSE
}
