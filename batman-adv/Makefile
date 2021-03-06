#
# Copyright (C) 2010 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
# $Id: Makefile 5624 2006-11-23 00:29:07Z nbd $

include $(TOPDIR)/rules.mk

PKG_NAME:=batman-adv

PKG_VERSION:=2014.1.0
BATCTL_VERSION:=2014.1.0
PKG_RELEASE:=2
PKG_MD5SUM:=d55aabb2bc508194407187648c773c1b
BATCTL_MD5SUM:=92fd4cb60e38ec952fb5fdda4b2e6933

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=http://downloads.open-mesh.org/batman/releases/batman-adv-$(PKG_VERSION)

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/$(PKG_NAME)-$(PKG_VERSION)
PKG_BATCTL_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)/batctl-$(BATCTL_VERSION)

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/kernel.mk

define KernelPackage/batman-adv
  URL:=http://www.open-mesh.org/
  MAINTAINER:=Marek Lindner <mareklindner@neomailbox.ch>
  SUBMENU:=Network Support
  DEPENDS:=+KMOD_BATMAN_ADV_BLA:kmod-lib-crc16 +kmod-crypto-core +kmod-crypto-crc32c +kmod-lib-crc32c +libc +libnl-tiny
  TITLE:=B.A.T.M.A.N. Adv
  FILES:=$(PKG_BUILD_DIR)/batman-adv.$(LINUX_KMOD_SUFFIX)
  AUTOLOAD:=$(call AutoLoad,50,batman-adv)
endef

define KernelPackage/batman-adv/description
B.A.T.M.A.N. advanced is a kernel module which allows to
build layer 2 mesh networks. This package contains the
version $(PKG_VERSION) of the kernel module plus its user space
configuration & managerment tool batctl.
endef

define KernelPackage/batman-adv/config
	source "$(SOURCE)/Config.in"
endef

MAKE_BATMAN_ADV_ARGS += \
	CROSS_COMPILE="$(TARGET_CROSS)" \
	KERNELPATH="$(LINUX_DIR)" \
	ARCH="$(LINUX_KARCH)" \
	PATH="$(TARGET_PATH)" \
	SUBDIRS="$(PKG_BUILD_DIR)" \
	PWD="$(PKG_BUILD_DIR)" \
	LINUX_VERSION="$(LINUX_VERSION)" \
	CONFIG_BATMAN_ADV_DEBUG=$(if $(CONFIG_KMOD_BATMAN_ADV_DEBUG_LOG),y,n) \
	CONFIG_BATMAN_ADV_BLA=$(if $(CONFIG_KMOD_BATMAN_ADV_BLA),y,n) \
	CONFIG_BATMAN_ADV_DAT=$(if $(CONFIG_KMOD_BATMAN_ADV_DAT),y,n) \
	CONFIG_BATMAN_ADV_NC=$(if $(CONFIG_KMOD_BATMAN_ADV_NC),y,n) \
	REVISION="" all

# The linker can identify unused sections of a binary when each symbol is stored
# in a separate section. This mostly removes unused linker sections and reduces 
# the size by ~3% on mipsel.

TARGET_CFLAGS  += -ffunction-sections -fdata-sections
TARGET_LDFLAGS += -Wl,--gc-sections

# Link-time optimization allows to move parts of the optimization from the single
# source file to the global source view. This is done by emitting the GIMPLE
# representation in each object file and analyzing it again during the link step.

TARGET_CFLAGS  += -flto
TARGET_LDFLAGS += -fuse-linker-plugin

MAKE_BATCTL_ENV += \
	CPPFLAGS="$(TARGET_CPPFLAGS)" \
	CFLAGS="$(TARGET_CFLAGS)" \
	LDFLAGS="$(TARGET_LDFLAGS)" \
	LIBNL_NAME="libnl-tiny"

MAKE_BATCTL_ARGS += \
	REVISION="" \
	CC="$(TARGET_CC)" \
	DESTDIR="$(PKG_INSTALL_DIR)" \
	batctl install

ifneq ($(DEVELOPER)$(CONFIG_KMOD_BATMAN_ADV_BATCTL),)
define Download/batctl
  FILE:=batctl-$(BATCTL_VERSION).tar.gz
  URL:=$(PKG_SOURCE_URL)
  MD5SUM:=$(BATCTL_MD5SUM)
endef
$(eval $(call Download,batctl))

BATCTL_EXTRACT = tar xzf "$(DL_DIR)/batctl-$(BATCTL_VERSION).tar.gz" -C "$(BUILD_DIR)/$(PKG_NAME)"
BATCTL_PATCH = $(call Build/DoPatch,"$(PKG_BATCTL_BUILD_DIR)","$(PATCH_DIR)","*batctl*")
BATCTL_BUILD = $(MAKE_BATCTL_ENV) $(MAKE) -C $(PKG_BATCTL_BUILD_DIR) $(MAKE_BATCTL_ARGS)
BATCTL_INSTALL = $(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/local/sbin/batctl $(1)/usr/sbin/
endif

KPATCH ?= $(PATCH)
define Build/DoPatch
	@if [ -d "$(2)" ]; then \
		if [ "$$$$(ls $(2) | grep -Ec $(3))" -gt 0 ]; then \
			$(KPATCH) "$(1)" "$(2)" "$(3)"; \
		fi; \
	fi
endef

define Build/Patch
	$(call Build/DoPatch,"$(PKG_BUILD_DIR)","$(PATCH_DIR)","*batman*")
	$(BATCTL_EXTRACT)
	$(BATCTL_PATCH)
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	$(SED) '/#define _NET_BATMAN_ADV_MAIN_H_/a\#undef CONFIG_MODULE_STRIPPED' \
		$(PKG_BUILD_DIR)/main.h
endef


define Build/Compile
	$(MAKE) -C "$(PKG_BUILD_DIR)" $(MAKE_BATMAN_ADV_ARGS)
	$(BATCTL_BUILD)
endef

define Build/Clean
        rm -rf $(BUILD_DIR)/$(PKG_NAME)/
endef

define KernelPackage/batman-adv/install
	$(INSTALL_DIR) $(1)/etc/config $(1)/etc/hotplug.d/net $(1)/etc/hotplug.d/iface $(1)/lib/batman-adv $(1)/usr/sbin $(1)/lib/netifd/proto
	$(INSTALL_DATA) ./files/etc/config/batman-adv $(1)/etc/config
	$(INSTALL_DATA) ./files/lib/batman-adv/config.sh $(1)/lib/batman-adv
	$(INSTALL_BIN) ./files/etc/hotplug.d/net/99-batman-adv $(1)/etc/hotplug.d/net
	$(INSTALL_BIN) ./files/lib/netifd/proto/batadv.sh $(1)/lib/netifd/proto
	$(BATCTL_INSTALL)
endef

$(eval $(call KernelPackage,batman-adv))
