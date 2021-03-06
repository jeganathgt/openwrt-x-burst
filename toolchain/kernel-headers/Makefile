# 
# Copyright (C) 2006-2009 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk

KERNEL_BUILD_DIR := $(BUILD_DIR_TOOLCHAIN)
BUILD_DIR := $(KERNEL_BUILD_DIR)

override QUILT:=
override HOST_QUILT:=

include $(INCLUDE_DIR)/kernel.mk

PKG_NAME:=linux
PKG_VERSION:=$(LINUX_VERSION)
PKG_SOURCE:=$(LINUX_SOURCE)
PKG_SOURCE_URL:=$(LINUX_SITE)
HOST_BUILD_DIR:=$(KERNEL_BUILD_DIR)/linux-$(LINUX_VERSION)
PKG_MD5SUM:=$(LINUX_KERNEL_MD5SUM)
LINUX_DIR := $(HOST_BUILD_DIR)

include $(INCLUDE_DIR)/toolchain-build.mk
include $(INCLUDE_DIR)/kernel-defaults.mk

ifeq ($(strip $(BOARD)),uml)
  LINUX_KARCH:=$(ARCH)
endif

ifeq ($(strip $(call CompareKernelPatchVer,$(KERNEL_PATCHVER),ge,2.6.18)),1)
  LINUX_HAS_HEADERS_INSTALL:=y
endif

KMAKE := $(MAKE) -C $(HOST_BUILD_DIR) \
	ARCH=$(LINUX_KARCH) \
	KBUILD_HAVE_NLS=no \
	CONFIG_SHELL=$(BASH)

define Host/Prepare/pre/powerpc
	if [ -d $(HOST_BUILD_DIR)/include/asm-ppc ]; then \
		mkdir -p $(PKG_BUILD_DIR)/include/asm-powerpc/; \
		$(CP) $(HOST_BUILD_DIR)/include/asm-ppc/* $(HOST_BUILD_DIR)/include/asm-powerpc/; \
		rm -rf $(HOST_BUILD_DIR)/include/asm-ppc; \
		ln -s $(HOST_BUILD_DIR)/include/asm-powerpc $(HOST_BUILD_DIR)/include/asm-ppc; \
	fi
endef

ifneq ($(LINUX_HAS_HEADERS_INSTALL),)
  define Host/Prepare/all
	mkdir -p $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr
	$(KMAKE) \
		CROSS_COMPILE=$(TARGET_CROSS) \
		INSTALL_HDR_PATH="$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/" \
		headers_install
  endef
else
  define Host/Prepare/all
	mkdir -p $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include
	cp -pLR $(HOST_BUILD_DIR)/include/asm $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/
	cp -pLR $(HOST_BUILD_DIR)/include/asm-generic $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/
	cp -pLR $(HOST_BUILD_DIR)/include/asm-$(LINUX_KARCH) $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/
	cp -pLR $(HOST_BUILD_DIR)/include/linux $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/
  endef
endif

# XXX: the following is needed to build lzma-loader
define Host/Prepare/lzma
	$(CP) \
		$(HOST_BUILD_DIR)/include/asm-mips/asm.h \
		$(HOST_BUILD_DIR)/include/asm-mips/regdef.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/asm/
endef

# XXX: the following are needed to build a cris toolchain
ifeq ($(strip $(call CompareKernelPatchVer,$(KERNEL_PATCHVER),lt,2.6.28)),1)
  define Host/Prepare/pre/cris
	ln -sf $(HOST_BUILD_DIR)/include/asm-cris/arch-v10 $(HOST_BUILD_DIR)/include/asm-cris/arch
	ln -sf $(HOST_BUILD_DIR)/include/asm-cris/arch-v10 $(HOST_BUILD_DIR)/arch/cris/arch
  endef

  define Host/Prepare/post/cris
	$(CP) \
		$(HOST_BUILD_DIR)/include/linux/user.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/linux/
	$(CP) \
		$(HOST_BUILD_DIR)/include/linux/user.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/linux/
	$(CP) \
		$(HOST_BUILD_DIR)/include/asm-cris/elf.h \
		$(HOST_BUILD_DIR)/include/asm-cris/page.h \
		$(HOST_BUILD_DIR)/include/asm-cris/user.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/asm/
	$(CP) \
		$(HOST_BUILD_DIR)/include/asm-generic/memory_model.h \
		$(HOST_BUILD_DIR)/include/asm-generic/page.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/asm-generic/

	mkdir -p $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/asm/arch
	$(CP) \
		$(HOST_BUILD_DIR)/include/asm-cris/arch/elf.h \
		$(HOST_BUILD_DIR)/include/asm-cris/arch/page.h \
		$(HOST_BUILD_DIR)/include/asm-cris/arch/ptrace.h \
		$(HOST_BUILD_DIR)/include/asm-cris/arch/user.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/asm/arch/
  endef
else
  define Host/Prepare/post/cris
	$(CP) \
		$(HOST_BUILD_DIR)/include/linux/user.h \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/linux/
	ln -snf $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/arch-v10/arch \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/arch
	$(SED) '/#include <asm\/page\.h>/d' $(BUILD_DIR_TOOLCHAIN)/linux-dev/usr/include/asm/user.h
  endef
endif

define Host/Prepare/post/mips
	$(call Host/Prepare/lzma)
endef

define Host/Prepare/post/mipsel
	$(call Host/Prepare/lzma)
endef

define Host/Prepare
	$(call Kernel/Prepare/Default)
	ln -sf linux-$(LINUX_VERSION) $(BUILD_DIR_TOOLCHAIN)/linux
	$(SED) 's/@expr length/@-expr length/' $(HOST_BUILD_DIR)/Makefile
	yes '' | $(KMAKE) oldconfig
	$(KMAKE) include/linux/version.h include/asm
	if [ -d $(HOST_BUILD_DIR)/arch/$(LINUX_KARCH)/include/asm ]; then \
		$(CP) \
			$(HOST_BUILD_DIR)/arch/$(LINUX_KARCH)/include/asm/. \
			$(HOST_BUILD_DIR)/include/asm-$(LINUX_KARCH)/; \
	fi
	$(call Host/Prepare/pre/$(ARCH))
	$(call Host/Prepare/all)
	$(call Host/Prepare/post/$(ARCH))
endef

define Host/Configure
endef

define Host/Compile
endef

define Host/Install
	$(CP) $(BUILD_DIR_TOOLCHAIN)/linux-dev/* $(TOOLCHAIN_DIR)/
endef

define Host/Clean
	rm -rf \
		$(HOST_BUILD_DIR) \
		$(BUILD_DIR_TOOLCHAIN)/linux \
		$(BUILD_DIR_TOOLCHAIN)/linux-dev
endef

$(eval $(call HostBuild))
