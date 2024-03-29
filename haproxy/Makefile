#
# Copyright (C) 2010-2016 OpenWrt.org
# Copyright (C) 2009-2016 Thomas Heil <heil@terminal-consulting.de>
# Copyright (C) 2018 Christian Lachner <gladiac@gmail.com>
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=haproxy
PKG_VERSION:=1.8.29
PKG_RELEASE:=1

PKG_SOURCE:=haproxy-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://www.haproxy.org/download/1.8/src/
PKG_HASH:=skip

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(BUILD_VARIANT)/$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-2.0
MAINTAINER:=Thomas Heil <heil@terminal-consulting.de>, \
		Christian Lachner <gladiac@gmail.com>

include $(INCLUDE_DIR)/package.mk

define Package/haproxy/Default
  SUBMENU:=Web Servers/Proxies
  SECTION:=net
  CATEGORY:=Network
  TITLE:=The Reliable, High Performance TCP/HTTP Load Balancer
  URL:=https://www.haproxy.org/
endef

define Download/lua535
	FILE:=lua-5.3.5.tar.gz
	URL:=https://www.lua.org/ftp/
	HASH:=0c2eed3f960446e1a3e4b9a1ca2f3ff893b6ce41942cf54d5dd59ab4b3b058ac
endef

define Build/Prepare
	$(call Build/Prepare/Default)
ifeq ($(ENABLE_LUA),y)
	tar -zxvf $(DL_DIR)/lua-5.3.5.tar.gz -C $(PKG_BUILD_DIR)
	ln -s $(PKG_BUILD_DIR)/lua-5.3.5 $(PKG_BUILD_DIR)/lua
endif
endef

define Package/haproxy/Default/conffiles
/etc/haproxy.cfg
endef

define Package/haproxy/Default/description
 Open source Reliable, High Performance TCP/HTTP Load Balancer.
endef

define Package/haproxy
  DEPENDS+= +libpcre +libltdl +zlib +libpthread +libopenssl +libncursesw +libreadline +libatomic +@OPENSSL_WITH_COMPRESSION +@OPENSSL_WITH_DTLS +@OPENSSL_ENGINE_CRYPTO
  TITLE+= (with SSL support)
  VARIANT:=ssl
$(call Package/haproxy/Default)
endef

define Package/haproxy/conffiles
$(call Package/haproxy/Default/conffiles)
endef

define Package/haproxy/description
$(call Package/haproxy/Default/description)
 This package is built with SSL and LUA support.
endef

define Package/haproxy-nossl
  TITLE+= (without SSL support)
  VARIANT:=nossl
  DEPENDS+= +libpcre +libltdl +zlib +libpthread +libatomic
  TITLE+= (without SSL support)
$(call Package/haproxy/Default)
endef

define Package/haproxy-nossl/conffiles
$(call Package/haproxy/Default/conffiles)
endef

define Package/haproxy-nossl/description
$(call Package/haproxy/Default/description)
 This package is built without SSL support.
endef

ENABLE_LUA:=y
ENABLE_REGPARM:=n

ifeq ($(CONFIG_TARGET_x86),y)
  ENABLE_REGPARM:=y
endif

LINUX_TARGET:=linux2628

ifeq ($(BUILD_VARIANT),ssl)
	ADDON+=USE_OPENSSL=1
	ADDON+=ADDLIB="-lcrypto -lm "
endif

ifeq ($(ENABLE_LUA),y)
	ADDON+=USE_LUA=1
	ADDON+=LUA_LIB_NAME="lua535"
	ADDON+=LUA_INC="$(STAGING_DIR)/lua-5.3.5/include"
	ADDON+=LUA_LIB="$(STAGING_DIR)/lua-5.3.5/lib"
endif

ifeq ($(ENABLE_REGPARM),y)
	ADDON+=USE_REGPARM=1
endif

ifeq ($(ENABLE_LUA),y)
define Build/Compile/lua
	$(MAKE) TARGET=$(LINUX_TARGET) -C $(PKG_BUILD_DIR)/lua \
		INSTALL_TOP="$(STAGING_DIR)/lua-5.3.5/" \
		CC="$(TARGET_CC)" \
		CFLAGS="$(TARGET_CFLAGS) $(TARGET_CPPFLAGS)" \
		LDFLAGS="$(TARGET_LDFLAGS) -lncurses -lreadline" \
		LD="$(TARGET_LD)" \
		linux install

	mv $(STAGING_DIR)/lua-5.3.5/lib/liblua.a $(STAGING_DIR)/lua-5.3.5/lib/liblua535.a
endef
endif

define Build/Compile
	$(call Build/Compile/lua)
	$(MAKE) TARGET=$(LINUX_TARGET) -C $(PKG_BUILD_DIR) \
		DESTDIR="$(PKG_INSTALL_DIR)" \
		CC="$(TARGET_CC)" \
		PCREDIR="$(STAGING_DIR)/usr/" \
		SMALL_OPTS="-DBUFSIZE=16384 -DMAXREWRITE=1030 -DSYSTEM_MAXCONN=165530" \
		USE_LINUX_TPROXY=1 USE_LINUX_SPLICE=1 USE_TFO=1 \
		USE_ZLIB=yes USE_PCRE=1 USE_PCRE_JIT=1 USE_GETADDRINFO=1 \
		VERSION="$(PKG_VERSION)" SUBVERS="-$(PKG_RELEASE)" \
		VERDATE="$(shell date -d @$(SOURCE_DATE_EPOCH) '+%Y/%m/%d')" IGNOREGIT=1 \
		$(ADDON) \
		CFLAGS="$(TARGET_CFLAGS)" \
		LD="$(TARGET_CC)" \
		LDFLAGS="$(TARGET_LDFLAGS) -latomic"

	$(MAKE_VARS) $(MAKE) -C $(PKG_BUILD_DIR) \
		DESTDIR="$(PKG_INSTALL_DIR)" \
		LD="$(TARGET_CC)" \
		LDFLAGS="$(TARGET_LDFLAGS)" \
		$(MAKE_FLAGS) \
		install

	$(MAKE_VARS) $(MAKE) -C $(PKG_BUILD_DIR)/contrib/halog \
		DESTDIR="$(PKG_INSTALL_DIR)" \
		$(MAKE_FLAGS) \
		OPTIMIZE="$(TARGET_CFLAGS)" \
		ADDLIB="-lcrypto" \
		halog
endef

define Package/haproxy/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/haproxy $(1)/usr/sbin/
	$(INSTALL_DIR) $(1)/etc
	$(INSTALL_CONF) ./files/haproxy.cfg $(1)/etc/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/haproxy.init $(1)/etc/init.d/haproxy
	$(INSTALL_DIR) $(1)/etc/hotplug.d/net
	$(INSTALL_BIN) ./files/haproxy.hotplug $(1)/etc/hotplug.d/net/90-haproxy
endef

Package/haproxy-nossl/install = $(Package/haproxy/install)

define Package/halog
	MENU:=1
	$(call Package/haproxy)
	TITLE+= halog
	DEPENDS:=haproxy
endef

define Package/halog/description
	HAProxy Log Analyzer
endef

define Package/halog/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/contrib/halog/halog $(1)/usr/bin/
endef

$(eval $(call Download,lua535))
$(eval $(call BuildPackage,haproxy))
$(eval $(call BuildPackage,halog))
$(eval $(call BuildPackage,haproxy-nossl))
