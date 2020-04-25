# Maintainer: Martin Gondermann <magicmonty@pagansoft.de>
# Based on AUR package by: Nigel Michki <nigeil@yahoo.com>
# Contributor: Serhii Balbieko <sergey@balbeko.com>
# Contributor: Simon Dreher <code@simon-dreher.de>

_name=sonic-pi
pkgname=sonic-pi-git
pkgver=v3.2.0.2.r91
pkgrel=1
pkgdesc="The Live Coding Music Synth for Everyone"
arch=('i686' 'x86_64')
url="http://sonic-pi.net/"
license=('MIT')
groups=('pro-audio')
conflicts=('sonic-pi')
depends=(
   'aubio'
   'gcc-libs'
   'glibc'
   'qt5-base'
   'ruby'
   'ruby-rake'
   'sed'
	 'libffi'
	 'supercollider'
   'sc3-plugins'
   'sox'
	 'jack'
   'qwt')
makedepends=('cmake' 'git' 'erlang-nox' 'gendesk' 'lua' 'qt5-tools' 'boost' 'wkhtmltopdf')
optdepends=('qjackctl: for graphical jackd spawning/configuration'
	    'jack2: better jackd if you want to use without gui'
	    'pulseaudio-jack: support for jack2-pulseaudio integration'
       'cadence: easy jack/pulseaudio cros2over')
source=('sonic-pi::git+https://github.com/samaaron/sonic-pi.git'
	     'launcher.sh'
        'sonic-pi.png')
md5sums=('SKIP'
         'ce83889f21eff588c69b7c0cb36f8b03'
         'ba86680be610cc3d6f12d4a89b0f434d')
config='Release'

pkgver() {
  cd "${srcdir}/${_name}"
  postfix=`git describe --long --tags | sed -r 's/([^-]*-g)/r\1/;s/-/./g' | cut -d. -f3-4`
  prefix=`cat app/gui/qt/CMakeLists.txt | grep ' VERSION ' | sed -r 's/^(\s*VERSION\s*)(.*)$/\2/'`
  echo v$prefix.$postfix
}

prepare() {
  cd "${srcdir}"
  gendesk -n -f \
          --pkgname "${pkgname}" \
          --pkgdesc "${pkgdesc}" \
          --name "${pkgname}" \
          --exec "/usr/bin/${_name}" \
          --categories "AudioVideo;Audio"
}

build() {
  cd "${srcdir}/${_name}/app/gui/qt"

./unix-prebuild.sh
  ./unix-config.sh --config $config
  cd build
  cmake --build . --config $config
}

package() {
  #Install sources to /opt/
  targetRoot=${pkgdir}/opt/${_name}
  sourceRoot=${srcdir}/${_name}
  qtsrc=${sourceRoot}/app/gui/qt
  qttgt=app/gui/qt

  mkdir -p $targetRoot
  cd $targetRoot

  mkdir -p app
  cp -R $sourceRoot/app/server app/server
  mkdir -p app/server/native/ruby/bin
  ln -s /usr/bin/ruby app/server/native/ruby/bin
  ln -s app/server .
  rm app/server/ruby/bin/compile-extensions.rb
  rm app/server/erlang/print_erlang_version
  rm app/server/erlang/*.app
  rm app/server/erlang/*.md
  chmod 0755 app/server/erlang/*.beam

  mkdir -p ${qttgt}

  cat ${qtsrc}/prune.rb | sed 's/rehearse = true/rehearse = false/' > ${qtsrc}/prune_hot.rb
  chmod 0755 ${qtsrc}/prune_hot.rb
  ${qtsrc}/prune_hot.rb app/server/ruby/vendor
  rm ${qtsrc}/prune_hot.rb

  install -vDm 755 ${qtsrc}/build/sonic-pi -t ${qttgt}/bin/
  install -vDm 644 ${qtsrc}/book/*.html -t ${qttgt}/book
  install -vDm 644 ${qtsrc}/fonts/*.ttf -t ${qttgt}/fonts
  install -vDm 644 ${qtsrc}/info/*.html -t ${qttgt}/info
  install -vDm 644 ${qtsrc}/lang/*.qm -t ${qttgt}/lang
  install -vDm 644 ${qtsrc}/help/*.html -t ${qttgt}/help
  install -vDm 644 ${qtsrc}/html/*.html -t ${qttgt}/html
  install -vDm 644 ${qtsrc}/images/*.png -t ${qttgt}/images
  install -vDm 644 ${qtsrc}/images/coreteam/*.png -t ${qttgt}/images/coreteam
  install -vDm 644 ${qtsrc}/images/toolbar/default/*.png -t ${qttgt}/images/toolbar/default
  install -vDm 644 ${qtsrc}/images/toolbar/pro/*.png -t ${qttgt}/images/toolbar/pro
  install -vDm 644 ${qtsrc}/images/tutorial/*.png -t ${qttgt}/images/tutorial

  install -vDm 644 ${qtsrc}/theme/app.qss -t ${qttgt}/theme
  echo "QWidget { background-color: paneColor; }" >> ${qttgt}/theme/app.qss
  install -vDm 644 ${qtsrc}/theme/dark/doc-styles.css -t ${qttgt}/theme/dark
  install -vDm 644 ${qtsrc}/theme/light/doc-styles.css -t ${qttgt}/theme/light
  install -vDm 644 ${qtsrc}/theme/high_contrast/doc-styles.css -t ${qttgt}/theme/high_contrast

  install -vDm 644 ${sourceRoot}/etc/samples/*.{flac,md} -t etc/samples
  install -vDm 644 ${sourceRoot}/etc/snippets/fx/*.sps -t etc/snippets/fx
  install -vDm 644 ${sourceRoot}/etc/snippets/live_loop/*.sps -t etc/snippets/live_loop
  install -vDm 644 ${sourceRoot}/etc/snippets/syntax/*.sps -t etc/snippets/syntax
  install -vDm 644 ${sourceRoot}/etc/synthdefs/compiled/*.scsyndef -t etc/synthdefs/compiled
  install -vDm 644 ${sourceRoot}/etc/buffers/*.wav -t etc/buffers
  install -vDm 644 ${sourceRoot}/etc/doc/cheatsheets/*.md -t etc/doc/cheatsheets
  install -vDm 644 ${sourceRoot}/etc/doc/tutorial/*.md -t etc/doc/tutorial
  install -vDm 644 ${sourceRoot}/etc/examples/algomancer/*.rb -t etc/examples/algomancer
  install -vDm 644 ${sourceRoot}/etc/examples/apprentice/*.rb -t etc/examples/apprentice
  install -vDm 644 ${sourceRoot}/etc/examples/illusionist/*.rb -t etc/examples/illusionist
  install -vDm 644 ${sourceRoot}/etc/examples/incubation/*.rb -t etc/examples/incubation
  install -vDm 644 ${sourceRoot}/etc/examples/magician/*.rb -t etc/examples/magician
  install -vDm 644 ${sourceRoot}/etc/examples/sorcerer/*.rb -t etc/examples/sorcerer
  install -vDm 644 ${sourceRoot}/etc/examples/wizard/*.rb -t etc/examples/wizard

  install -vDm 644 ${sourceRoot}/{FAQ,LICENSE,CHANGELOG,CONTRIBUTORS}.md -t .

  # Add a launcher script to /usr/bin
  cd $pkgdir
  mkdir -p usr/bin
  install -vDm 755 $srcdir/launcher.sh usr/bin/sonic-pi

  #Add a desktop entry
  install -vDm 644 ${srcdir}/${_name}.desktop -t usr/share/applications
  install -vDm 644 ${srcdir}/${_name}.png -t usr/share/pixmaps

  find . -type f -iname '*.erl' -delete
  find . -type f -iname '*.c' -delete
  find . -type f -iname '*.h' -delete
  find . -type f -iname 'Rakefile' -delete
  find . -type d -name 'test' -exec rm -rf {} +
  find . -type d -name 'tests' -exec rm -rf {} +
  find . -type f -name '.gitkeep' -delete
}
