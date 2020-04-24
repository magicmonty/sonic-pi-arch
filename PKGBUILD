# Maintainer: Martin Gondermann <magicmonty@pagansoft.de>
# Based on AUR package by: Nigel Michki <nigeil@yahoo.com>
# Contributor: Serhii Balbieko <sergey@balbeko.com>
# Contributor: Simon Dreher <code@simon-dreher.de>

pkgname=sonic-pi-git
pkgver=v3.2.0.2.r91
pkgrel=1
pkgdesc="A music-centric programming environment, originally built for the raspberry pi."
arch=('i686'
      'x86_64')
url="http://sonic-pi.net/"
license=('MIT')
groups=()
conflicts=('sonic-pi')
depends=('sed'
	 'ruby'
	 'libffi'
	 'supercollider'
   'sc3-plugins'
	 'jack'
   'aubio'
   'erlang'
   'qwt')
makedepends=('cmake'
	     'git'
       'qt5-tools'
       'boost')
optdepends=('qjackctl: for graphical jackd spawning/configuration'
	    'jack2: better jackd if you want to use without gui'
	    'pulseaudio-jack: support for jack2-pulseaudio integration'
       'cadence: easy jack/pulseaudio cros2over')
source=('sonic-pi::git+https://github.com/samaaron/sonic-pi.git'
	     'launcher.sh'
        'sonic-pi-git.png'
        'sonic-pi-git.desktop'
        'lambdaphonic.patch')
md5sums=('SKIP'
         'ce83889f21eff588c69b7c0cb36f8b03'
         'ba86680be610cc3d6f12d4a89b0f434d'
         'fd330b2be9b52e9bee2fb9922141e2ca'
         'c65353a6903eab6bc26c8793e13d855a')
config='Release'

build() {
  cd $srcdir/sonic-pi/app/gui/qt

  ./unix-prebuild.sh --build-aubio
  ./unix-config.sh --config $config
  cd build
  cmake --build . --config $config
}

pkgver() {
  cd $srcdir/sonic-pi
  postfix=`git describe --long --tags | sed -r 's/([^-]*-g)/r\1/;s/-/./g' | cut -d. -f3-4`
  prefix=`cat app/gui/qt/CMakeLists.txt | grep ' VERSION ' | sed -r 's/^(\s*VERSION\s*)(.*)$/\2/'`
  echo v$prefix.$postfix
}

package() {
  #Install sources to /opt/
  targetRoot=$pkgdir/opt/sonic-pi
  sourceRoot=$srcdir/sonic-pi

  mkdir -p $targetRoot
  cd $targetRoot

  mkdir -p app
  cp -R $sourceRoot/app/server app/server
  mkdir -p app/server/native/ruby/bin
  ln -s /usr/bin/ruby app/server/native/ruby/bin
  ln -s app/server .
  rm app/server/ruby/bin/compile-extensions.rb
  rm app/server/erlang/print_erlang_version

  cp -R $sourceRoot/etc .
  rm -rf etc/synthdefs/graphviz
  rm -rf etc/wavetables

  mkdir -p app/gui/qt
  cp -R $sourceRoot/app/gui/qt/theme app/gui/qt/
  echo "QWidget { background-color: paneColor; }" >> app/gui/qt/theme/app.qss

  cat $sourceRoot/app/gui/qt/prune.rb | sed 's/rehearse = true/rehearse = false/' > $sourceRoot/app/gui/qt/prune_hot.rb
  chmod 0755 $sourceRoot/app/gui/qt/prune_hot.rb
  $sourceRoot/app/gui/qt/prune_hot.rb app/server/ruby/vendor
  rm $sourceRoot/app/gui/qt/prune_hot.rb

  mkdir -p app/gui/qt/bin
  install -Dm755 $sourceRoot/app/gui/qt/build/sonic-pi app/gui/qt/bin/

  # Add a launcher script to /usr/bin
  cd $pkgdir
  mkdir -p usr/bin
  install -Dm755 $srcdir/launcher.sh usr/bin/sonic-pi

  #Add a desktop entry
  mkdir -p usr/share/applications
  install -Dm644 $srcdir/$pkgname.desktop usr/share/applications/$pkgname.desktop
  mkdir -p usr/share/pixmaps
  install -Dm644 $srcdir/$pkgname.png usr/share/pixmaps/$pkgname.png

  find . -type f -iname '*.erl' -delete
  find . -type f -iname '*.c' -delete
  find . -type f -iname '*.h' -delete
  find . -type f -iname 'Rakefile' -delete
  find . -type d -name 'test' -exec rm -rf {} +
  find . -type d -name 'tests' -exec rm -rf {} +
  find . -type f -name '.gitkeep' -delete
}
