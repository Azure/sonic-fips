.ONESHELL:
SHELL = /bin/bash
.SHELLFLAGS += -e

ARCH ?= amd64
CMAKE_BUILD_TYPE ?= Release
BUILD_NAME = symcrypt-openssl
SYMCRYPT_OPENSSL_VERSION ?= 0.1
SYMCRYPT_PACKAGE = $(BUILD_NAME)_$(SYMCRYPT_OPENSSL_VERSION)_$(ARCH).deb
BUILD_ROOT_DIR = tmp/$(BUILD_NAME)

CMAKE_ARCH = AMD64
LIB_INSTALL_NAME = x86_64-linux-gnu
ifeq ($(ARCH), arm64)
CMAKE_ARCH = ARM64
LIB_INSTALL_NAME = aarch64-linux-gnu
else ifeq ($(ARCH), armhf)
CMAKE_ARCH = ARMHF
LIB_INSTALL_NAME = arm-linux-gnueabihf 
endif

INSTALL_PATH = $(BUILD_ROOT_DIR)/usr/lib/$(LIB_INSTALL_NAME)
ENGINES_PATH = $(INSTALL_PATH)/engines-1.1
DEBIAN_DIR = $(BUILD_ROOT_DIR)/DEBIAN

ROOT_PATH = $(shell realpath $(shell pwd)/../..)
DEST ?= $(ROOT_PATH)/target
LIBSYMCRYPT = $(DEST)/libsymcrypt.so
LIBSYMCRYPTENGINE = $(DEST)/libsymcryptengine.so
TARGET = $(DEST)/$(SYMCRYPT_PACKAGE)

DEPENDS := $(LIBSYMCRYPT) $(LIBSYMCRYPTENGINE)

all: $(TARGET)

list:
	@$(foreach target,$(DEPENDS) $(TARGET),echo $(target);)

$(LIBSYMCRYPT):
	cd ../SymCrypt
	mkdir -p bin
	cd bin
	cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake-toolchain/LinuxUserMode-$(CMAKE_ARCH).cmake -DCMAKE_BUILD_TYPE=$(CMAKE_BUILD_TYPE)
	cmake --build .
	mkdir -p $(DEST)
	cp -a module/$(CMAKE_ARCH)/LinuxUserMode/generic/libsymcrypt.so* $(DEST)/

$(LIBSYMCRYPTENGINE): $(LIBSYMCRYPT)
	cd ../SymCrypt-OpenSSL
	cp $(LIBSYMCRYPT) ./
	mkdir -p bin
	cd bin
	cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake-toolchain/LinuxUserMode-$(CMAKE_ARCH).cmake -DSYMCRYPT_ROOT_DIR=$(ROOT_PATH)/src/SymCrypt -DCMAKE_BUILD_TYPE=$(CMAKE_BUILD_TYPE)
	cmake --build .
	rm ../libsymcrypt.so
	cp SymCryptEngine/dynamic/symcryptengine.so $(LIBSYMCRYPTENGINE)

$(TARGET): $(DEPENDS)
	mkdir -p $(INSTALL_PATH)
	mkdir -p $(ENGINES_PATH)
	mkdir -p $(DEBIAN_DIR)
	mkdir -p $(BUILD_ROOT_DIR)/usr/lib/ssl
	cp -a $(DEST)/libsymcrypt.so* $(INSTALL_PATH)/
	cp $(LIBSYMCRYPTENGINE) $(INSTALL_PATH)
	ln -sf $(shell basename $(LIBSYMCRYPTENGINE)) $(INSTALL_PATH)/symcryptengine.so
	ln -sf ../$(shell basename $(LIBSYMCRYPTENGINE)) $(ENGINES_PATH)/symcryptengine.so
	chmod o+r $(INSTALL_PATH)/*
	cp -rf debian/* $(DEBIAN_DIR)/
	cp openssl.cnf $(BUILD_ROOT_DIR)/usr/lib/ssl/openssl-fips.cnf
	fakeroot dpkg-deb --build $(BUILD_ROOT_DIR) $(TARGET)
