# How many source files to build in parallel within each package
jobs=-j8

# Source and binary directories
src=/src
prefix=/target
work=$(src)/macosx
stow=$(prefix)/stow

include ../makefile.common

qt=qt-everywhere-opensource-src-4.8.5
qttgz=$(downloads)/$(qt).tar.gz

# Useful macros for building packages
pkgconfig=PKG_CONFIG_PATH=$(PKG_CONFIG_PATH):$(prefix)/lib/pkgconfig:/usr/local/Library/Homebrew/pkgconfig
cflags=-m64 -I$(prefix)/include --stdlib=libc++
ldflags=-m64 -L$(prefix)/lib -lc++ -Wl,-headerpad_max_install_names
flagsm64=CFLAGS="$(cflags)" CXXFLAGS="$(cflags)" LDFLAGS="$(ldflags)" LFLAGS="$(ldflags)"
path=PATH=$(prefix)/bin:$$PATH
configure=$(pkgconfig) $(flagsm64) $(path) ./configure --prefix=$(prefix) --build=x86_64-apple-darwin10.6.0
cmake=$(pkgconfig) cmake .. \
	-DCMAKE_OSX_ARCHITECTURES=x86_64 \
	-DCMAKE_INSTALL_PREFIX=$(prefix) \
	-DQT_HEADERS_DIR=$(prefix)/include/ \
	-DQT_LIBRARY_DIR=$(prefix)/bin \
	-DQT_QMAKE_EXECUTABLE=$(prefix)/bin/qmake \
	-DCMAKE_C_FLAGS="$(cflags)" \
	-DCMAKE_CXX_FLAGS="$(cflags)" \
	-DCMAKE_EXE_LINKER_FLAGS="$(ldflags)"
make=$(path) make $(jobs)
slowmake=$(path) make
extractpkg=tar -xvf $(1)
buildpkg=(cd $(1) && $(configure) && $(make))
buildpkgcmake=(cd $(1) && mkdir build ; cd build && $(cmake) && $(make))
installpkg=rm -rf $(stow)/$(1) && \
	cd $(1) && \
	make install DESTDIR=$(stow)/$(1) && \
	mv -fv $(stow)/$(1)$(prefix)/* $(stow)/$(1)/ && \
	rm -fv $(stow)/$(1)/share/info/dir ; \
	rm -fv $(stow)/$(1)/lib/charset.alias ; \
	cd $(stow) && \
	stow -R $(1) && \
	touch $(work)/$@
installpkgcmake=rm -rf $(stow)/$(1) && \
	cd $(1)/build && \
	make install DESTDIR=$(stow)/$(1) && \
	mv -fv $(stow)/$(1)$(prefix)/* $(stow)/$(1)/ && \
	rm -fv $(stow)/$(1)/share/info/dir ; \
	cd $(stow) && \
	stow -R $(1) && \
	touch $(work)/$@

extractandbuild=$(call extractpkg,$(1)) && $(call buildpkg,$(2)) && $(call installpkg,$(2))
extractandbuildcmake=$(call extractpkg,$(1)) && $(call buildpkgcmake,$(2)) && $(call installpkgcmake,$(2))


# Top-level targets
clementine-build-deps: boost liblastfm taglib libmtp \
                       libgpod gstreamer glew qjson \
                       protobuf spotify fftw sparsehash qt
clementine-gst-plugins: gst-plugins-base gst-plugins-good gst-plugins-bad \
                        gst-plugins-ugly gst-ffmpeg
clementine-deps: clementine-build-deps clementine-gst-plugins

# The actual commands for compiling each package follow

.done-qjson: .done-qt
	$(call extractpkg,$(qjsontgz))
	$(call buildpkgcmake,$(qjson))
	sed -e 's/QtCore//' -i "" $(qjson)/build/QJson.pc
	$(call installpkgcmake,$(qjson))

.done-sqlite:
	$(call extractandbuild,$(sqlitetgz),$(sqlite))

.done-gettext:
	$(call extractpkg,$(gettexttgz))
	cd $(gettext) && patch -p0 < ../patches/gettext-02-examples.patch
	cd $(gettext) && $(configure) --without-git
	$(call buildpkg,$(gettext))
	$(call installpkg,$(gettext))

.done-glib: .done-gettext  .done-ffi
	$(call extractpkg,$(glibtgz))
	cd $(glib) && $(configure) --with-libiconv=native
	cd $(glib) && $(make)
	rm -fv
	$(call installpkg,$(glib))

.done-ffi:
	$(call extractandbuild,$(ffitgz),$(ffi))

.done-fftw:
	$(call extractpkg,$(fftwtgz))
	cd $(fftw) && $(configure) \
		--with-our-malloc16 --with-windows-f77-mangling --enable-shared \
		--disable-static --enable-threads --with-combined-threads \
		--enable-portable-binary --enable-sse2
	cd $(fftw) && $(make)
	$(call installpkg,$(fftw))

.done-libplist: .done-glib
	$(call extractpkg,$(plisttgz))
	cd $(plist) && patch -p0 < ../patches/libplist-02-iterator.patch
	-mkdir $(plist)/build
	cd $(plist)/build && $(cmake) -DENABLE_CYTHON=OFF -DENABLE_SWIG=OFF
	cd $(plist)/build && $(slowmake)
	$(call installpkgcmake,$(plist))

.done-libgpod: .done-sqlite .done-libplist
	$(call extractpkg,$(gpodtgz))
	cd $(gpod) && $(configure) --disable-gdk-pixbuf --without-libimobiledevice
	cd $(gpod) && $(make)
	$(call installpkg,$(gpod))

.done-gstreamer: .done-glib .done-orc .done-gettext 
	$(call extractpkg,$(gsttgz))
	cd $(gst) && patch -p0 < ../patches/gstreamer-01-flex.patch
	cd $(gst) && patch -p1 < ../patches/gstreamer-02-log.patch
	cd $(gst) && $(configure) --disable-examples --disable-tests
	cd $(gst) && $(make)
	$(call installpkg,$(gst))

.done-gst-plugins-base: .done-gstreamer .done-ogg .done-vorbis
	$(call extractpkg,$(gstbasetgz))
	cd $(gstbase) && $(configure) --disable-examples --disable-x --disable-xvideo --disable-xshm --disable-theora
	cd $(gstbase) && $(make)
	$(call installpkg,$(gstbase))

.done-orc:
	$(call extractandbuild,$(orctgz),$(orc))

.done-ogg:
	$(call extractandbuild,$(oggtgz),$(ogg))

.done-vorbis: .done-ogg
	$(call extractandbuild,$(vorbistgz),$(vorbis))

.done-gst-plugins-good: .done-gstreamer .done-gst-plugins-base .done-flac .done-speex .done-wavpack .done-libsoup
	$(call extractpkg,$(gstgoodtgz))
	cd $(gstgood) && SOUP_CFLAGS=-I$(prefix)/include/libsoup-2.4 $(configure) --disable-shout2 --disable-examples --disable-x --disable-deinterlace --disable-osx_video --disable-aalib --disable-libdv --disable-libpng --disable-cairo --disable-esd --disable-libcaca
	cd $(gstgood) && $(make)
	$(call installpkg,$(gstgood))

.done-flac:
	$(call extractpkg,$(flactgz))
	cd $(flac) && $(configure) --disable-xmms-plugin --disable-asm-optimizations
	cd $(flac) && $(make)
	$(call installpkg,$(flac))

.done-speex: .done-ogg
	$(call extractandbuild,$(speextgz),$(speex))

.done-gst-plugins-ugly: .done-gstreamer .done-gst-plugins-base .done-oil .done-mad .done-lame .done-cdio
	$(call extractpkg,$(gstuglytgz))
	patch -d $(gstugly) -p1 < patches/gst-plugins-ugly-01-cdio.patch
	cd $(gstugly) && $(configure) \
					--disable-amrnb \
					--disable-amrwb \
					--disable-examples \
					--disable-mpeg2dec \
					--disable-x264 \
					--enable-mad
	cd $(gstugly) && $(make)
	$(call installpkg,$(gstugly))

.done-oil:
	$(call extractpkg,$(oiltgz))
	cd $(oil) && $(pkgconfig) $(flagsm64) $(path) CFLAGS="$(cflags) -fheinous-gnu-extensions" ./configure --prefix=$(prefix) --build=x86_64-apple-darwin10.6.0
	cd $(oil) && $(make)
	$(call installpkg,$(oil))

.done-id3tag:
	$(call extractandbuild,$(id3tagtgz),$(id3tag))

.done-mad: .done-id3tag
	$(call extractandbuild,$(madtgz),$(mad))

.done-lame:
	$(call extractandbuild,$(lametgz),$(lame))

.done-gst-plugins-bad: .done-gstreamer .done-gst-plugins-base .done-faad .done-faac .done-libmms .done-oil .done-libquicktime .done-libmpcdec .done-opus
	$(call extractpkg,$(gstbadtgz))
	cd $(gstbad) && $(configure) \
				--disable-acm \
				--disable-apple_media \
				--disable-cog \
				--disable-curl \
				--disable-dirac \
				--disable-examples \
				--disable-gsettings \
				--disable-modplug \
				--disable-ofa \
				--disable-rtmp \
				--disable-zbar
	cd $(gstbad) && $(make)
	$(call installpkg,$(gstbad))

.done-gst-ffmpeg: .done-gstreamer .done-gst-plugins-base
	$(call extractandbuild,$(gstffmpegtgz),$(gstffmpeg))

.done-faad:
	$(call extractandbuild,$(faadtgz),$(faad))

.done-faac:
	$(call extractandbuild,$(faactgz),$(faac))

.done-libmms: .done-glib  .done-gettext
	$(call extractandbuild,$(mmstgz),$(mms))

.done-libquicktime:
	$(call extractpkg,$(quicktimetgz))
	cd $(quicktime) && $(configure) --without-doxygen --without-ffmpeg
	cd $(quicktime) && $(make)
	$(call installpkg,$(quicktime))

.done-usb:
	$(call extractandbuild,$(usbtgz),$(usb))

.done-gcrypt: .done-libgpgerror  .done-gettext
	$(call extractpkg,$(gcrypttgz))
	cd $(gcrypt) && $(configure) --with-gpg-error-prefix=$(prefix) --disable-asm
	cd $(gcrypt) && $(make)
	$(call installpkg,$(gcrypt))

.done-taglib:
	$(call extractpkg,$(taglibtgz))
	-mkdir $(taglib)/build
	cd $(taglib)/build && $(cmake) -DWITH_ASF=ON -DWITH_MP4=ON
	cd $(taglib)/build && $(make)
	$(call installpkgcmake,$(taglib))

.done-liblastfm: .done-qt
	$(call extractandbuildcmake,$(lastfmtgz),$(lastfm))

.done-boost:
	tar -xf $(boosttgz) -C $(stow)
	cd $(stow) && stow -R $(boost)
	touch $@

.done-protobuf:
	$(call extractandbuild,$(protobuftgz),$(protobuf))

.done-spotify:
	unzip -o $(spotifymactgz) -d $(stow)
	rm -rf $(stow)/$(spotifymac)/doc
	rm -rf $(stow)/$(spotifymac)/ChangeLog
	rm -rf $(stow)/$(spotifymac)/LICENSE
	rm -rf $(stow)/$(spotifymac)/README
	rm -rf $(stow)/$(spotifymac)/share
	cd $(stow) && stow $(spotifymac)
	touch $@

.done-libmtp: .done-usb 
	$(call extractpkg,$(mtptgz))
	cd $(mtp) && $(configure) --disable-mtpz
	cd $(mtp) && $(make)
	$(call installpkg,$(mtp))

.done-libmpcdec:
	$(call extractandbuild,$(mpcdectgz),$(mpcdec))

.done-wavpack:
	$(call extractandbuild,$(wavpacktgz),$(wavpack))

.done-gmp:
	$(call extractandbuild,$(gmptgz),$(gmp))

.done-nettle: .done-gmp
	$(call extractandbuild,$(nettletgz),$(nettle))

.done-gnutls: .done-gcrypt .done-libtasn1 .done-nettle
	$(call extractpkg,$(gnutlstgz))
	cd $(gnutls) && $(configure) --with-libgcrypt --disable-cxx
	cd $(gnutls) && $(make)
#	sed -e 's/ *, *zlib//' -i "" $(gnutls)/lib/gnutls.pc
	$(call installpkg,$(gnutls))

.done-libtasn1:
	$(call extractandbuild,$(tasn1tgz),$(tasn1))

.done-libgpgerror:
	$(call extractandbuild,$(gpgerrortgz),$(gpgerror))

.done-libproxy:
	$(call extractpkg,$(proxytgz))
	patch -d $(proxy) -p0 < patches/libproxy-03-build.patch
	patch -d $(proxy) -p0 < patches/libproxy-04-darwin.patch
	-mkdir $(proxy)/build
	cd $(proxy)/build && $(cmake) -DWITH_PYTHON=OFF -DWITH_PERL=OFF -DBUILD_TESTING=OFF
	cd $(proxy)/build && $(make)
	$(call installpkgcmake,$(proxy))


.done-glib-networking: .done-glib .done-gnutls .done-libproxy
	$(call extractpkg,$(glibnetworkingtgz))
	patch -d $(glibnetworking) -p1 < patches/glib-networking-01-literals.patch
	cd $(glibnetworking) && $(configure) --with-ca-certificates=../../root-certs.pem
	cd $(glibnetworking) && $(make)
	$(call installpkg,$(glibnetworking))

.done-libsoup: .done-glib .done-gnutls .done-glib-networking  .done-gettext
	$(call extractpkg,$(souptgz))
	cd $(soup) && $(configure) --without-gnome
	cd $(soup) && $(make)
	$(call installpkg,$(soup))

.done-glew:
	$(call extractpkg,$(glewtgz))
	cd $(glew) && $(make) CFLAGS.EXTRA=-m64 LDFLAGS.EXTRA=-m64 GLEW_DEST=$(stow)/$(glew)
	cd $(glew) && make install GLEW_DEST=$(stow)/$(glew)
	cd $(stow) && stow -R $(glew)
	touch $(work)/$@

.done-cdio: 
	$(call extractpkg,$(cdiotgz))
	cd $(cdio) && $(configure)
	cd $(cdio) && $(make)
	$(call installpkg,$(cdio))

.done-sparsehash:
	$(call extractandbuild,$(sparsehashtgz),$(sparsehash))

.done-qt:
	$(call extractpkg,$(qttgz))
	patch -d $(qt) -p1 -R < patches/qt-01-mousebuttons.patch
	patch -d $(qt) -p0 < patches/qt-05-corewlan.patch
	patch -d $(qt) -p1 < patches/qt-06-qtbug-34358-performance.patch
	cd $(qt) && ./configure \
		-release \
		-opensource -confirm-license \
		-prefix $(stow)/$(qt) \
		-no-phonon -no-webkit -no-xmlpatterns -no-qt3support \
		-no-declarative -no-audio-backend \
		-no-script -no-scripttools \
		-nomake demos -nomake examples \
		-no-exceptions \
		-no-libtiff \
		-no-sql-sqlite
	cd $(qt) && $(make)
	cd $(qt) && make install
	cd $(stow) && stow -R $(qt)
	touch $(work)/$@

.done-opus:
	$(call extractandbuild,$(opustgz),$(opus))
