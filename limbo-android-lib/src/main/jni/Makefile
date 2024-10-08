include android-limbo-build.mak

ifeq ($(NDK_ENV), windows)
	NDK_BUILD = ndk-build.cmd
else ifeq ($(NDK_ENV), windows-x86_64)
	NDK_BUILD = ndk-build.cmd
else
	NDK_BUILD = ndk-build
endif

ndkbuild=cd $(NDK_PROJECT_PATH) && $(NDK_BUILD) -j $(BUILD_THREADS)
ndkclean=cd $(NDK_PROJECT_PATH) && $(NDK_BUILD) clean
ndkcleanobj=rm -rf $(NDK_PROJECT_PATH)/obj/

ifeq ($(NDK_ENV), windows)
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb.cmd
else ifeq ($(NDK_ENV), windows-x86_64)
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb.cmd
else
	ndkgdb=cd $(NDK_PROJECT_PATH) && ndk-gdb
endif

CREATE_OBJDIR=mkdir -p $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/
CREATE_LIBSDIR=mkdir -p $(NDK_PROJECT_PATH)/libs/$(APP_ABI_DIR)/
DEL_LIBSDIR=rm -rf $(NDK_PROJECT_PATH)/libs/* \
	$(NDK_PROJECT_PATH)/jniLibs/* \
	$(LIMBO_APP_X86)/jniLibs/* \
	$(LIMBO_APP_ARM)/jniLibs/*


INSTALL_GLIB_LIBS=cp -f glib/glib/.libs/*.so \
    $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/

INSTALL_FFI_LIBS=cp -f libffi/$(GNU_HOST)/.libs/*.a \
    $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/

INSTALL_PIXMAN_LIBS=cp -f pixman/pixman/.libs/*.so \
    $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/

MOD_QEMU_LIBS=$(OBJCOPY) --redefine-sym open=android_open  \
    qemu/x86_64-softmmu/qemu-system-x86_64

INSTALL_LIBS=cp -f $(NDK_PROJECT_PATH)/obj/local/$(APP_ABI)/*.so \
	../libs/$(APP_ABI_DIR)/

REMOVE_GDBSETUP=rm -f ../libs/$(APP_ABI_DIR)/gdb.setup
REMOVE_GDBSERVER=rm -f ../libs/$(APP_ABI_DIR)/gdbserver

SYNC_LIBS=rsync -av --exclude libqemu-system-* ../libs/* ../jniLibs/

LIMBO_PRJ_ROOT=../../../..
LIMBO_APP_X86=$(LIMBO_PRJ_ROOT)/limbo-android-x86/src/main
LIMBO_APP_ARM=$(LIMBO_PRJ_ROOT)/limbo-android-arm/src/main

SYNC_LIBS_X86=rsync -av --include='libqemu-system-x86*.so' \
    --include='gdbserver' --include='gdb.setup' \
    --include='*/' --exclude='*' \
    ../libs/ $(LIMBO_APP_X86)/jniLibs/
SYNC_LIBS_ARM=rsync -av --include='libqemu-system-arm*.so' \
    --include='libqemu-system-aarch64*.so' \
    --include='gdbserver' --include='gdb.setup' \
    --include='*/' --exclude='*' \
    ../libs/ $(LIMBO_APP_ARM)/jniLibs/

COPY_ROMS=-mkdir -p ../assets/roms && cp -rf ./qemu/pc-bios/*.rom \
./qemu/pc-bios/*.bin \
./qemu/pc-bios/*.img \
./qemu/pc-bios/*.bmp \
./qemu/pc-bios/*.ico \
./qemu/pc-bios/*.svg \
./qemu/pc-bios/*.rsrc \
./qemu/pc-bios/openbios-* \
./qemu/pc-bios/README \
../assets/roms/

COPY_KEYMAPS=cp -rf ./qemu/pc-bios/keymaps ../assets/roms/
COPY_QEMU_VERSION=cp -f ./qemu/VERSION ../assets/QEMU_VERSION
COPY_RESOLV_CONF=mkdir -p ../assets/roms/etc/ && printf "nameserver 8.8.8.8\n\n" > ../assets/roms/etc/resolv.conf

COPY_ASSETS_X86=mkdir -p ../../../../limbo-android-x86/src/main/assets/ && \
    cp -rf ../assets/* ../../../../limbo-android-x86/src/main/assets/
COPY_ASSETS_ARM=mkdir -p ../../../../limbo-android-arm/src/main/assets/ && \
    cp -rf ../assets/* ../../../../limbo-android-arm/src/main/assets/

#RENAME_TUNCTL=mv -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/tunctl \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/libtunctl.so
#RENAME_PARPROUTED=mv -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/parprouted \
	$(NDK_PROJECT_PATH)/libs/$(APP_ABI)/libparprouted.so
	
#GDB needs the gdb.setup under ./libs
COPY_GDB_SETUP_LIBS=-cp -f $(NDK_PROJECT_PATH)/libs/$(APP_ABI)/gdb.setup \
	$(NDK_PROJECT_PATH)/libs/gdb.setup
	
ifneq ($(NDK_DEBUG),1)
	STRIP_LIBS=$(STRIP) $(NDK_PROJECT_PATH)/libs/$(APP_ABI_DIR)/*.so
endif

# we disable for now some options so armv7 host can compile with gcc
PIXMAN_DISABLE_OPTIONS=--disable-mmx --disable-arm-iwmmxt --disable-arm-iwmmxt2  \
	--disable-openmp  --disable-loongson-mmi  --disable-mmx    --disable-sse2   --disable-ssse3 \
	--disable-vmx --disable-arm-simd   --disable-arm-neon   --disable-arm-iwmmxt   \
	--disable-arm-iwmmxt2  --disable-mips-dspr2  --disable-gcc-inline-asm

DEBUG =
ifeq ($(NDK_DEBUG), 1)
	LIBFFI_DEBUG = --enable-debug
	LIBGLIB_DEBUG = --enable-debug
	LIBPIXMAN_DEBUG =
else
	LIBFFI_DEBUG = --disable-debug
    LIBGLIB_DEBUG = --disable-debug
    LIBPIXMAN_DEBUG =
endif

export

.PHONY: limbo clean distclean clean-ndk createdirs clean-ffi clean-glib clean-qemu clean-pixman

default: limbo
	
createdirs:
	$(CREATE_OBJDIR)
	$(DEL_LIBSDIR)
	$(CREATE_LIBSDIR)

ndk:
	$(ndkbuild)

clean-ndk:
	$(ndkclean)
	$(ndkcleanobj)

clean-ffi:
	-cd $(LIMBO_JNI_ROOT)/libffi && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/libffi && $(MAKE) clean

config-ffi:
	cd $(LIMBO_JNI_ROOT)/libffi && ./configure --host=$(GNU_HOST) $(LIBFFI_DEBUG) --enable-static --disable-shared \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) --sysroot=$(NDK_SYSROOT) $(NDK_SYSROOT_ARCH_INC) " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib -fno-unwind-tables "

ffi:
	cd $(LIMBO_JNI_ROOT)/libffi && $(MAKE) -j $(BUILD_THREADS) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) --sysroot=$(NDK_SYSROOT) $(NDK_SYSROOT_ARCH_INC) " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) -nostdlib -fno-unwind-tables "
	$(INSTALL_FFI_LIBS)

config-glib: ndk
	cd $(LIMBO_JNI_ROOT)/glib && NOCONFIGURE=1 ./autogen.sh
	cd $(LIMBO_JNI_ROOT)/glib && cp -rf $(LIMBO_JNI_ROOT)/patches/android.cache . \
	&& chmod a-w android.cache && ./configure --host=$(GNU_HOST) --cache-file=android.cache \
	--disable-libmount  --disable-dtrace  --disable-systemtap --with-pcre=internal \
	--with-libiconv=no \
	--with-sysroot=$(SYSROOT) $(LIBGLIB_DEBUG) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) \
	$(NDK_SYSROOT_INC) \
	$(NDK_SYSROOT_ARCH_INC) \
	-I${SYSROOT}/usr/include $(SYS_ROOT) \
	-I${LIMBO_JNI_ROOT}/compat \
	-I${LIMBO_JNI_ROOT}/compat/musl -I${LIMBO_JNI_ROOT}/compat/musl/include " \
	LDFLAGS=" $(ARCH_LD_CLANG_FLAGS) $(ARCH_LD_FLAGS) $(SYS_ROOT) \
	-L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) -lc -lcompat-musl -llog -lcompat-limbo " \
	LIBFFI_CFLAGS="-I${LIMBO_JNI_ROOT}/libffi/$(GNU_HOST)/include " \
	LIBFFI_LIBS="${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI)/libffi.a"

glib: ndk
	cd $(LIMBO_JNI_ROOT)/glib/glib && $(MAKE) -j $(BUILD_THREADS) CFLAGS="$(ARCH_CFLAGS) \
	$(NDK_SYSROOT_INC) \
	$(NDK_SYSROOT_ARCH_INC) \
	-I$(NDK_INCLUDE)  -I${SYSROOT}/usr/include $(SYS_ROOT) \
	-I${NDK_ROOT}/sources/android/support/include -I${LIMBO_JNI_ROOT}/compat \
	-include ${LIMBO_JNI_ROOT}/compat/musl/compat_iconv.h -I${LIMBO_JNI_ROOT}/compat/musl -I${LIMBO_JNI_ROOT}/compat/musl/include " \
	LDFLAGS=" $(ARCH_LD_CLANG_FLAGS) $(ARCH_LD_FLAGS) $(SYS_ROOT) -L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) \
	-lc -lcompat-musl -llog -lcompat-limbo " \
	-lgcc \
	LIBFFI_CFLAGS="-I${LIMBO_JNI_ROOT}/libffi/$(GNU_HOST)/include " \
	LIBFFI_LIBS="${LIMBO_JNI_ROOT}/libffi/$(GNU_HOST)/.libs/libffi.a "
	$(INSTALL_GLIB_LIBS)

clean-glib:
	-find $(LIMBO_JNI_ROOT)/glib/ -name "*.lo" -exec rm -rf {} \;
	-cd $(LIMBO_JNI_ROOT)/glib && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/glib && $(MAKE) clean
	-rm -rf $(LIMBO_JNI_ROOT)/glib/android.cache


config-pixman: ndk
	cd $(LIMBO_JNI_ROOT)/pixman && ./configure --host=$(GNU_HOST) --with-sysroot=$(SYSROOT) \
	--disable-libtool-lock \
	$(PIXMAN_DISABLE_OPTIONS) \
	$(LIBPIXMAN_DEBUG) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE) \
	$(NDK_SYSROOT_INC) \
	$(NDK_SYSROOT_ARCH_INC) \
	-I${SYSROOT}/usr/include $(SYS_ROOT) -I${NDK_ROOT}/sources/android/support/include \
	-I${NDK_ROOT}/sources/android/cpufeatures -I${LIMBO_JNI_ROOT}/compat " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) \
	-L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) -lc -llog \
	-lcompat-limbo "

pixman: ndk
	cd $(LIMBO_JNI_ROOT)/pixman/pixman && $(MAKE) -j $(BUILD_THREADS) \
	CFLAGS="$(ARCH_CFLAGS) -I$(NDK_INCLUDE)  -I${SYSROOT}/usr/include \
	$(SYS_ROOT) -I${NDK_ROOT}/sources/android/support/include \
	$(NDK_SYSROOT_INC) \
	$(NDK_SYSROOT_ARCH_INC) \
	-I${NDK_ROOT}/sources/android/cpufeatures -I${LIMBO_JNI_ROOT}/compat " \
	LDFLAGS="$(ARCH_LD_FLAGS) $(SYS_ROOT) \
	-L${LIMBO_JNI_ROOT}/../obj/local/$(APP_ABI) -lc -llog \
	-lcompat-limbo "
	$(INSTALL_PIXMAN_LIBS)

clean-pixman:
	-find $(LIMBO_JNI_ROOT)/pixman -name "*.lo" -exec rm -rf {} \;
	-cd $(LIMBO_JNI_ROOT)/pixman && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/pixman && $(MAKE) clean

_qemu: ndk
	cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) all -j $(BUILD_THREADS) BUILD_GUEST=$(BUILD_GUEST) CFLAGS="$(ARCH_CFLAGS)"

config-qemu: ndk
	$(MAKE) -f $(LIMBO_JNI_ROOT)/android-config/android-qemu-config.mak

clean-qemu:
	-cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) distclean
	-cd $(LIMBO_JNI_ROOT)/qemu && $(MAKE) clean
	-rm $(NDK_PROJECT_PATH)/obj/local/arm64-v8a/libqemu-system-x86_64.a
	-rm -rf ../assets/roms/* 
	-rm -rf $(LIMBO_APP_X86)/assets/roms/*
	-rm -rf $(LIMBO_APP_ARM)/assets/roms/*

config-deps: ndk config-ffi config-glib config-pixman

deps: ndk ffi glib pixman 

_build-qemu: ndk 
	make _qemu
	$(INSTALL_LIBS)
	$(RENAME_TUNCTL)
	$(RENAME_PARPROUTED)
	$(COPY_ROMS)
	$(COPY_KEYMAPS)
	$(COPY_QEMU_VERSION)
	$(COPY_RESOLV_CONF)

qemu: ndk
	make _build-qemu
	make install

install-common:
	$(CREATE_LIBSDIR)
	$(INSTALL_LIBS)
	$(STRIP_LIBS)
	$(SYNC_LIBS)

install:
	make install-common
	$(SYNC_LIBS_X86)
	$(COPY_ASSETS_X86)
	$(SYNC_LIBS_ARM)
	$(COPY_ASSETS_ARM)

limbo:
	make ndk
	make config-deps
	make deps
	make config-qemu
	make qemu
	make install

EXPORT_GUEST_ALL=export BUILD_GUEST=x86_64-softmmu,aarch64-softmmu

#most common combinations
all:
	make distclean && $(EXPORT_GUEST_ALL) && export BUILD_HOST=arm64-v8a && make limbo
	make distclean && $(EXPORT_GUEST_ALL) && export BUILD_HOST=armeabi-v7a && make limbo
	make distclean && $(EXPORT_GUEST_ALL) && export BUILD_HOST=x86 && make limbo
	make distclean && $(EXPORT_GUEST_ALL) && export BUILD_HOST=x86_64 && make limbo

ndk-gdb:
	@echo APP_ABI=$(APP_ABI)
	$(COPY_GDB_SETUP_LIBS)
	$(ndkgdb) --attach $(PKG_NAME) --verbose -t

distclean: clean-qemu clean-ffi clean-glib clean-pixman

clean: 
	make clean-qemu 
	make clean-ffi 
	make clean-glib 
	make clean-pixman
	make clean-ndk
	-find . -name "*.d" -exec rm -rf {} \;
	-find . -name "*.a" -exec rm -rf {} \;
	-find . -name "*.o" -exec rm -rf {} \;
	-find . -name "*.so" -exec rm -rf {} \;
	$(MAKE) createdirs
	
help:
	@echo " Usage: export BUILD_GUEST=<emu-arch>-softmmu && export BUILD_HOST=<eabi> && make limbo"
	@echo "  emu-arch (emulator): x86_64 (x86 compatible), aarch64 (armv7 compatible)"
	@echo "  eabi (device): armeabi-v7a, arm64-v8a, x86, x86_64"
	@echo "  optional: export NDK_DEBUG=1 for debuggable binaries"
	@echo " example: export BUILD_GUEST=x86_64-softmmu && export BUILD_HOST=armeabi-v7a && make limbo"
