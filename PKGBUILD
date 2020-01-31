# Maintainer: Martin Gondermann <magicmonty@pagansoft.de>
# Based on AUR package by: Nigel Michki <nigeil@yahoo.com>
# Contributor: Serhii Balbieko <sergey@balbeko.com>
# Contributor: Simon Dreher <code@simon-dreher.de>

pkgname=sonic-pi-git
pkgver=v3.2.r1176.g3538a1840
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
	 'lua'
	 'qscintilla-qt5'
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
        'build-ubuntu-app.patch'
        'SonicPi.patch'
        'lambdaphonic.patch'
        'Monoid-Regular.ttf'
        'Monoid-Italic.ttf')
md5sums=('SKIP'
         '298e2729cda0c33c9cec7f7f721c1bbd'
         'ba86680be610cc3d6f12d4a89b0f434d'
         'fd330b2be9b52e9bee2fb9922141e2ca'
         '6317fe781fbad36be19946567fc877e8'
         '18ae5d7db132a68387aba07dcdd10a33'
         '5ea85a2191b9fe2ce9ae92141b3fc46a'
         '7e9c019819d3c84efb61a3abded177aa'
         '3f772e57770d2d3a6850af070a37b194')

prepare() {
  msg2 "Hook up qwt to qmake"
  qmake -set QMAKEFEATURES usr/share/qt4/mkspecs/features

  #  msg2 "Fix wrongly-named (on Arch) QT library"
  # find $srcdir/sonic-pi/app/gui/qt -type f -name "*" -readable -exec sed -i 's/lqt5scintilla2/lqscintilla2_qt5/g' {} +

  #Patch build-ubuntu-app script to skip ubuntu-specific (and redundant) options
  msg2 "Patch build-ubuntu-app script for Arch Linux"
  cd $srcdir/sonic-pi/app/gui/qt
  patch < $srcdir/build-ubuntu-app.patch
  patch < $srcdir/SonicPi.patch
  cd $srcdir/sonic-pi
  # patch -p 1 < ../sonic-pi-ruby-2.5.patch
  patch -p 1 < ../lambdaphonic.patch
  cp $srcdir/*.ttf $srcdir/sonic-pi/app/gui/qt/fonts/
}

build() {
  #Based on instructions from INSTALL_LINUX.md in upstream sources
  cd $srcdir/sonic-pi/app/gui/qt
  ./build-ubuntu-app

  #Cleaning up object files
  cd $srcdir
  find . -type f -name "*.o" -exec rm {} +
  find . ! -perm -g+r -exec chmod 666 {} +
  rm -f $srcdir/sonic-pi/app/gui/qt/sed*
}

pkgver() {
  cd $srcdir/sonic-pi
  postfix=`git describe --long --tags | sed -r 's/([^-]*-g)/r\1/;s/-/./g' | cut -d. -f3-4`
  prefix=`cat app/gui/qt/mainwindow.cpp | grep ' version = "' | cut -d\" -f2`
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

  mv $pkgdir/opt/sonic-pi/app/server/native/linux/* $pkgdir/opt/sonic-pi/app/server/native
  rm -rf $pkgdir/opt/sonic-pi/app/server/native/linux
  rm -rf $pkgdir/opt/sonic-pi/app/server/ruby/test
  rm -rf $pkgdir/opt/sonic-pi/app/gui/qt/platform
  rm -rf $pkgdir/opt/sonic-pi/app/gui/qt/wix
  rm -rf $pkgdir/opt/sonic-pi/app/gui/qt/image_source
  rm -rf $pkgdir/opt/sonic-pi/etc/synthdefs/graphviz
  rm -rf $pkgdir/opt/sonic-pi/etc/wavetables

  rm $pkgdir/opt/sonic-pi/app/gui/qt/.qmake.stash
  rm $pkgdir/opt/sonic-pi/app/gui/qt/create-pdf
  rm $pkgdir/opt/sonic-pi/app/gui/qt/mac-*
  rm $pkgdir/opt/sonic-pi/app/gui/qt/prune*.rb
  rm $pkgdir/opt/sonic-pi/app/gui/qt/rp-*
  rm $pkgdir/opt/sonic-pi/app/gui/qt/SonicPi.rc
}
