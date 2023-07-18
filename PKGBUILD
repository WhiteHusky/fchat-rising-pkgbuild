# Maintainer: Carlen White <whitersuburban@gmail.com>
pkgname=fchat-rising
_pkgname=fchat-rising
pkgver=1.23.5
pkgrel=4
pkgdesc="The F-Chat 3.0 client from F-List modified by MrStallion. Uses a system-wide electron instead of the built in."
arch=('x86_64')
url="https://github.com/mrstallion/fchat-rising"
license=('MIT')
conflicts=(
    'fchat-rising-git'
)
depends=(
    'electron>=17.0.0'
    'electron<18.0.0'
    'libappindicator-gtk3'
    'libindicator-gtk3'
    'libnotify'
    'libxss'
    'libxtst'
    'libsecret'
    'icu72-bin'
)
makedepends=('yarn' 'npm' 'nvm' 'node-gyp')
provides=('fchat-3.0')
source=(
    "fchat::git+https://github.com/mrstallion/fchat-rising#tag=v$pkgver"
    'local://fchat.desktop'
    'local://remove-electron-requirement.patch'
    'local://deadletter.patch'
)
sha256sums=(
    'SKIP'
    '9f1a95982e8327f32173056d03d79705c01f11307bae59e8f5d3ec3435421f1f'
    '15d92bd2030b4046f10ac8b7e94b329f1f558987fd8fe0b51933dcc8d47f9ce1'
    '1c136916bb968ffaa3e8e6dfb03f1c219f9703bddf278312be0360660a9cec91'
)

_ensure_local_nvm() {
    # https://wiki.archlinux.org/title/Node.js_package_guidelines#Using_nvm
    # let's be sure we are starting clean
    which nvm >/dev/null 2>&1 && nvm deactivate && nvm unload
    export NVM_DIR="${srcdir}/.nvm"

    # The init script returns 3 if version specified
    # in ./.nvrc is not (yet) installed in $NVM_DIR
    # but nvm itself still gets loaded ok
    source /usr/share/nvm/init-nvm.sh || [[ $? != 1 ]]
}

prepare() {
    cd $srcdir
    patch --directory="$srcdir/fchat/" --forward --strip=1 --input="$srcdir/remove-electron-requirement.patch"
    patch --directory="$srcdir/fchat/" --forward --strip=1 --input="$srcdir/deadletter.patch"
    echo "Init NVM..."
    _ensure_local_nvm
    echo "Install Node v16..."
    nvm install v16
    echo "Enabling Node v16..."
    nvm use v16
}

build() {
    echo "Init NVM..."
    _ensure_local_nvm
    echo "Using Node v16..."
    nvm use v16
    cd $srcdir/fchat/
    # https://wiki.archlinux.org/title/Electron_package_guidelines#Building_compiled_extensions_against_the_system_electron
    # Avoid trying to download a copy of Electron.
    export npm_config_target=$(tail /usr/lib/electron17/version)
    export npm_config_arch=x64
    export npm_config_target_arch=x64
    export npm_config_disturl=https://electronjs.org/headers
    export npm_config_runtime=electron
    export npm_config_build_from_source=true
    HOME="$srcdir/.node" yarn install
    cd $srcdir/fchat/electron
    export GENERATE_SOURCEMAP=false
    HOME="$srcdir/.node" yarn build:dist
    sed -i "s|Exec=.*|Exec=/usr/bin/$_pkgname|" "$srcdir"/fchat.desktop
    sed -i "s|Icon=.*|Icon=/usr/share/pixmaps/$_pkgname.png|" "$srcdir"/fchat.desktop
}

package() {
    cd $srcdir/fchat/electron
    install -d "$pkgdir"/usr/lib/$_pkgname
    cp -r "$srcdir"/fchat/electron/app/* "$pkgdir"/usr/lib/$_pkgname
    rm "$srcdir"/$_pkgname || true
    echo "#!/bin/sh" >> "$srcdir"/$_pkgname
    # F-Chat needs to executed within it's directory
    echo "cd /usr/lib/$_pkgname" >> "$srcdir"/$_pkgname
    echo "exec /usr/bin/electron17 ./ \$@" >> "$srcdir"/$_pkgname
    install -d "$pkgdir"/usr/{bin,share/{pixmaps,applications}}
    install -Dm 755 "$srcdir"/$_pkgname "$pkgdir"/usr/bin/$_pkgname
    cp "$srcdir"/fchat/electron/build/icon.png "$pkgdir"/usr/share/pixmaps/$_pkgname.png
    cp "$srcdir"/fchat.desktop "$pkgdir"/usr/share/applications/$_pkgname.desktop

    install -Dm 644 "$srcdir"/fchat/LICENSE "$pkgdir"/usr/share/licenses/$_pkgname/LICENSE
}
