# Maintainer: Miroslav Pavleski <miroslav@pavleski.net>
# Maintainer: Caleb Maclennan <caleb@alerque.com>
# Maintainer: Bruno Pagani <archange@archlinux.org>
# Contributor: Raphael Nestler <archlinux@rnstlr.ch>
# Contributor: William Gathoye <william + aur at gathoye dot be>
# Contributor: Aleksandar Trifunović <akstrfn at gmail dot com>
# Contributor: Jan Was <janek dot jan at gmail dot com>
# Contributor: AUR[Severus]
# Contributor: Matthias Mailänder <matthias@mailaender.name>

pkgname=mattermost-desktop
pkgver=6.4.0
pkgrel=1
pkgdesc='Mattermost Desktop application'
arch=(x86_64)
url="https://github.com/${pkgname/-//}"
license=(Apache-2.0)
_electron=electron44
# Mattermost 6.4.0 upstream targets electron45, which is not packaged in Arch
# yet. Build against electron44 until it lands, then switch _electron and drop
# _electron_allow_mismatch. Screen sharing in Calls may prompt as a camera or
# microphone request, and tzdata is 2026d rather than 2026e.
_electron_allow_mismatch=1
depends=($_electron
         libxcrypt-compat)
makedepends=(git
             jq
             python-setuptools
             moreutils
             nodejs-lts-krypton
             node-gyp
             npm)
_archive="${pkgname#*-}-$pkgver"
source=("$pkgname-$pkgver.tar.gz::$url/archive/v$pkgver.tar.gz"
        "$pkgname.sh"
        "$pkgname.desktop")
sha256sums=('3fcdebde6c6f79e3ab86495c45252a08f599b4e20c7575b66bf584323b0a7418'
            '1c2bf48b6397d04a5a536c5c9f4960db53249c838c380f03f808c612b00ba4c6'
            '13df5ad4c7813986403658f5f88d723907db507703942f3facfb37ceaca657ef')

_npmargs=(--no-audit
          --no-fund
          --allow-git=all)

prepare() {
	cd "$_archive"
	sed -i -e "s/git rev-parse --short HEAD/echo $pkgver/" webpack.config.base.js
	sed -e "s/@ELECTRON@/$_electron/" "../$pkgname.sh" > "$pkgname.sh"
	# Make sure we are using the same major version of Electron as upstream
	local _origElectron="electron$(jq -r '.devDependencies.electron | split(".")[0]' package.json)"
	if [ "$_electron" != "$_origElectron" ]; then
		if [ "${_electron_allow_mismatch:-0}" = 1 ]; then
			warning "Electron version mismatch: $_electron vs $_origElectron (allowed by _electron_allow_mismatch)"
		else
			error "Electron version mismatch: $_electron vs $_origElectron"
			exit 1
		fi
	fi
	# Substitute our exact distro packaged version of Electron
	local _electronVersion="$(< "/usr/lib/$_electron/version")"
	jq '	.devDependencies["electron"] = $electronVersion |
		del(.devDependencies["electron-rebuild"]) |
		del(.config)' \
			--arg electronVersion "$_electronVersion" \
			package.json |
		sponge package.json
	# Change the build type so it doesn't try to bundle Electron
	sed '/module.exports/i config.linux.target = ["dir"];' -i electron-builder.ts
	# Set desktop name to match with the desktop file
	npm pkg set desktopName="$pkgname"
	# Set path for autostart
	sed -i "/new AutoLaunch({/a\            path: '/usr/bin/mattermost-desktop'," src/main/AutoLauncher.ts
	# 6.4.0's postinstall runs node_modules/electron/install.js, which would
	# download a bundled Electron this package never uses. The distro package
	# provides the runtime, so skip that download.
	ELECTRON_SKIP_BINARY_DOWNLOAD=1 npm --cache "$srcdir/npm-cache" "${_npmargs[@]}" install
}

build() {
	cd "$_archive"
	export NODE_ENV=production
	npm --cache "$srcdir/npm-cache" "${_npmargs[@]}" --offline run build-prod
	npx electron-builder --linux dir --publish=never
}

package() {
	cd "$_archive"
	install -Dm0644 -t "$pkgdir/usr/lib/$pkgname/" release/linux-*unpacked/resources/app.asar
	cp -a --no-preserve=all release/linux-*unpacked/resources/app.asar.unpacked "$pkgdir/usr/lib/$pkgname/"
	chmod -R a+rX "$pkgdir/usr/lib/$pkgname/app.asar.unpacked"
	install -Dm0644 -t "$pkgdir/usr/share/licenses/$pkgname/" LICENSE.txt
	install -Dm0644 src/assets/appicon_48.png "$pkgdir/usr/share/icons/hicolor/48x48/apps/$pkgname.png"
	install -Dm0644 src/assets/linux/app_icon.png "$pkgdir/usr/share/icons/hicolor/512x512/apps/$pkgname.png"
	install -Dm0755 "$pkgname.sh" "$pkgdir/usr/bin/$pkgname"
	install -Dm0644 -t "$pkgdir/usr/share/applications/" "$srcdir/$pkgname.desktop"
}
