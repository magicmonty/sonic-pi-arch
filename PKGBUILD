# Maintainer: Martin Gondermann <magicmonty@pagansoft.de>
# Based on AUR package by: Nigel Michki <nigeil@yahoo.com>
# Contributor: Serhii Balbieko <sergey@balbeko.com>
# Contributor: Simon Dreher <code@simon-dreher.de>

pkgname=sonic-pi-git
pkgver=v3.2.0.r1330.g070be9422
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
        'lambdaphonic.patch'
        'Monoid-Regular.ttf'
        'Monoid-Italic.ttf')
md5sums=('SKIP'
         'ce83889f21eff588c69b7c0cb36f8b03'
         'ba86680be610cc3d6f12d4a89b0f434d'
         'fd330b2be9b52e9bee2fb9922141e2ca'
         'c65353a6903eab6bc26c8793e13d855a'
         '7e9c019819d3c84efb61a3abded177aa'
         '3f772e57770d2d3a6850af070a37b194')

prepare() {
# cd $srcdir/sonic-pi
#  patch -p 1 -l < ../lambdaphonic.patch
 cp $srcdir/*.ttf $srcdir/sonic-pi/app/gui/qt/fonts/
}

build() {
  rootdir=$srcdir/sonic-pi
  rubybin=$rootdir/app/server/ruby/bin

  echo "Compiling Erlang parts"
  cd $rootdir/app/server/erlang
  erlc osc.erl
  erlc pi_server.erl

  cd $rootdir/app/gui/qt
  echo "Compiling native ruby extensions"
  $rubybin/compile-extensions.rb
  echo "Translating tutorial..."
  $rubybin/i18n-tool.rb -t
  echo "Creating QT docs"
  cp -fn utils/ruby_help.tmpl utils/ruby_help.h || true
  $rubybin/qt-doc.rb -o utils/ruby_help.h
  echo "Updating GUI translation files..."
  lrelease lang/*.ts
  ./unix-config.sh
  cd build
  cmake --build . --config Release
  mkdir -p ../bin
  install sonic-pi -t ../bin

  cd ../external/osmid
  cmake .
  cmake --build . --config Release
  mkdir -p $rootdir/app/server/native/osmid
  install m2o o2m -t $rootdir/app/server/native/osmid/

  #Cleaning up object files
  cd $srcdir
  find . -type f -name "*.o" -exec rm {} +
  find . ! -perm -g+r -exec chmod 666 {} +
}

pkgver() {
  cd $srcdir/sonic-pi
  postfix=`git describe --long --tags | sed -r 's/([^-]*-g)/r\1/;s/-/./g' | cut -d. -f3-4`
  prefix=`cat app/gui/qt/CMakeLists.txt | grep ' VERSION ' | sed -r 's/^(\s*VERSION\s*)(.*)$/\2/'`
  echo v$prefix.$postfix
}

package() {
  #Install sources to /opt/
  mkdir -p $pkgdir/opt/sonic-pi
  cp -R $srcdir/sonic-pi/* $pkgdir/opt/sonic-pi/ > /dev/null
  ln -s -r $pkgdir/opt/sonic-pi/app/server $pkgdir/opt/sonic-pi/server

  #Add a launcher script to /usr/bin
  mkdir -p $pkgdir/usr/bin
  install -Dm644 "$srcdir/launcher.sh" "$pkgdir/usr/bin/sonic-pi"
  chmod +x $pkgdir/usr/bin/sonic-pi

  #Add a desktop entry
  mkdir -p $pkgdir/usr/share/applications
  install -Dm644 "$pkgname.desktop" "$pkgdir/usr/share/applications/$pkgname.desktop"
  mkdir -p $pkgdir/usr/share/pixmaps
  install -Dm644 "sonic-pi-git.png" "$pkgdir/usr/share/pixmaps/$pkgname.png"

  cd $srcdir/sonic-pi/app/gui/qt
  cat prune.rb | sed 's/rehearse = true/rehearse = false/' > prune_hot.rb
  chmod 0755 prune_hot.rb
  ./prune_hot.rb $pkgdir/opt/sonic-pi/app/server/ruby/vendor

  rm $pkgdir/opt/sonic-pi/CORETEAM.html

  cd $pkgdir
  find $pkgdir -type f -iname '*.cpp' -delete
  find $pkgdir -type f -iname '*.erl' -delete
  find $pkgdir -type f -iname '*.hpp' -delete
  find $pkgdir -type f -iname '*.c' -delete
  find $pkgdir -type f -iname '*.h' -delete
  find $pkgdir -type f -iname '*.hh' -delete
  find $pkgdir -type f -iname '*.o' -delete
  find $pkgdir -type f -iname '*.patch' -delete
  find $pkgdir -type f -iname 'build-*' -delete
  find $pkgdir -type f -iname '*.pro' -delete
  find $pkgdir -type f -iname '*.qrc' -delete
  find $pkgdir -type f -iname '*.bat' -delete
  find $pkgdir -type f -iname '*.tmpl' -delete
  find $pkgdir -type f -iname 'Makefile' -delete
  find $pkgdir -type f -iname 'Rakefile' -delete
  find $pkgdir -type f -iname 'Brewfile*' -delete
  find $pkgdir -type d -name 'test' -exec rm -rf {} +
  find $pkgdir -type d -name 'tests' -exec rm -rf {} +
  find $pkgdir -type f -name 'INSTALL-*.md' -delete
  find $pkgdir -type f -name '*.orig' -delete
  find $pkgdir -type f -name '.gitkeep' -delete

  rootdir=$pkgdir/opt/sonic-pi
  qtdir=$rootdir/app/gui/qt
  serverdir=$rootdir/app/server

  rm -rf $rootdir/install
  rm -rf $rootdir/prebuilt
  rm -rf $serverdir/ruby/test
  rm -rf $rootdir/etc/synthdefs/graphviz
  rm -rf $rootdir/etc/wavetables

  rm -rf $qtdir/platform
  rm -rf $qtdir/wix
  rm -rf $qtdir/cmake
  rm -rf $qtdir/CMakeFiles
  rm -rf $qtdir/external
  rm -rf $qtdir/model
  rm -rf $qtdir/old
  rm -rf $qtdir/osc
  rm -rf $qtdir/utils
  rm -rf $qtdir/visualizer
  rm -rf $qtdir/widgets
  rm -rf $qtdir/build
  rm -rf $qtdir/image_source

  rm $serverdir/ruby/bin/compile-extensions.rb
  rm $serverdir/erlang/print_erlang_version

  rm $qtdir/mac-*
  rm $qtdir/prune*.rb
  rm $qtdir/rp-*
  rm $qtdir/SonicPi.rc
  rm $qtdir/CMakeLists.txt
  rm $qtdir/unix-config.sh
  rm $qtdir/unix-prebuild.sh
  rm $qtdir/util-create-pdf
  rm $qtdir/README.md

  mkdir -p $serverdir/native/ruby/bin
  ln -s /usr/bin/ruby $serverdir/native/ruby/bin
}
