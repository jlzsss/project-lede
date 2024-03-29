# Copyright (C) 2009-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=nfs-kernel-server
PKG_VERSION:=2.3.3
PKG_RELEASE:=3
PKG_HASH:=skip

PKG_SOURCE_URL:=@SF/nfs
PKG_SOURCE:=nfs-utils-$(PKG_VERSION).tar.xz

HOST_BUILD_DIR:=$(BUILD_DIR_HOST)/nfs-utils-$(PKG_VERSION)
PKG_BUILD_DIR:=$(BUILD_DIR)/nfs-utils-$(PKG_VERSION)
PKG_INSTALL:=1
PKG_BUILD_PARALLEL:=1
PKG_BUILD_DEPENDS:=$(PKG_NAME)/host
HOST_BUILD_DEPENDS:=libtirpc/host
PKG_CONFIG_DEPENDS:= CONFIG_IPV6

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/nfs-kernel-server/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Filesystem
  DEPENDS:=+libwrap +libblkid +libuuid +libtirpc
  URL:=http://nfs.sourceforge.net/
  MAINTAINER:=Peter Wagner <tripolar@gmx.at>
endef

define Package/nfs-kernel-server
  $(call Package/nfs-kernel-server/Default)
  TITLE:=Kernel NFS server support
  DEPENDS+= +kmod-fs-nfsd +kmod-fs-nfs +portmap
endef

define Package/nfs-kernel-server/description
  Kernel NFS server support
endef

define Package/nfs-kernel-server-utils
  $(call Package/nfs-kernel-server/Default)
  TITLE:=NFS server utils
  DEPENDS:=nfs-kernel-server
endef

define Package/nfs-kernel-server-utils/description
  NFS server utils
endef

define Package/nfs-kernel-server/conffiles
/etc/exports
endef

define Package/nfs-utils
  $(call Package/nfs-kernel-server/Default)
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS+= +libevent2
  TITLE:=updated mount utility (includes nfs4)
endef

define Package/nfs-utils/description
  Updated mount.nfs command - allows mounting nfs4 volumes
endef

TARGET_CFLAGS += -Wno-error=implicit-function-declaration \
		 -Wno-error=strict-prototypes \
		 -Wno-error=incompatible-pointer-types \
		 -Wno-error=format-security \
		 -Wno-error=undef \
		 -Wno-error=missing-include-dirs

TARGET_LDFLAGS += -Wl,-rpath-link=$(STAGING_DIR)/usr/lib \
		  -L$(STAGING_DIR)/usr/lib/libevent

CONFIGURE_ARGS += \
	--disable-gss \
	--disable-nfsv4 \
	--disable-nfsv41 \
	--enable-static \
	--enable-shared \
	--disable-caps

ifeq ($(CONFIG_IPV6),n)
CONFIGURE_ARGS += --disable-ipv6
endif

CONFIGURE_VARS += \
	libblkid_cv_is_recent=yes \
	ac_cv_lib_resolv___res_querydomain=yes \
	ac_cv_lib_bsd_daemon=no \
	CONFIG_SQLITE3_TRUE="\#" \
	CONFIG_NFSDCLD_TRUE="\#"

MAKE_FLAGS += \
	OPT="$(TARGET_CFLAGS)" \
	INSTALLSUID="install -m 4755" \
	DESTDIR="$(PKG_INSTALL_DIR)" \
	RPCGEN_PATH=$(STAGING_DIR_HOSTPKG)/bin/rpcgen \
	RPCGEN=$(STAGING_DIR_HOSTPKG)/bin/rpcgen

HOST_CFLAGS += -Dlinux \
	-Wno-error=missing-include-dirs

HOST_CONFIGURE_ARGS += \
	--disable-gss \
	--disable-nfsv4 \
	--disable-nfsv41 \
	--without-tcp-wrappers

HOST_CONFIGURE_VARS += \
	ac_cv_lib_event_event_dispatch=yes \
	ac_cv_lib_nfsidmap_nfs4_init_name_mapping=yes \
	ac_cv_lib_blkid_blkid_get_library_version=yes \
	ac_cv_header_event_h=yes \
	ac_cv_header_nfsidmap_h=yes \
	ac_cv_header_blkid_blkid_h=yes \
	ac_cv_lib_resolv___res_querydomain=yes \
	GSSGLUE_CFLAGS=" " \
	GSSGLUE_LIBS=" " \
	RPCSECGSS_CFLAGS=" " \
	RPCSECGSS_LIBS=" " \
	CONFIG_SQLITE3_TRUE="\#" \
	CONFIG_NFSDCLD_TRUE="\#"

define Host/Compile
	$(MAKE) -C $(HOST_BUILD_DIR)/tools/rpcgen all
endef

define Host/Install
	$(INSTALL_DIR) $(STAGING_DIR_HOSTPKG)/bin
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/tools/rpcgen/rpcgen $(STAGING_DIR_HOSTPKG)/bin/rpcgen
endef

define Package/nfs-kernel-server/install
	$(INSTALL_DIR) $(1)/etc/init.d $(1)/usr/sbin
	$(INSTALL_DATA) ./files/nfsd.exports $(1)/etc/exports
	$(INSTALL_BIN) ./files/nfsd.init $(1)/etc/init.d/nfsd
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/statd/sm-notify $(1)/usr/sbin/
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/statd/statd $(1)/usr/sbin/rpc.statd
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/nfsd/nfsd $(1)/usr/sbin/rpc.nfsd
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/mountd/mountd $(1)/usr/sbin/rpc.mountd
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/exportfs/exportfs $(1)/usr/sbin/
endef

define Package/nfs-kernel-server-utils/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/showmount/showmount $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/utils/nfsstat/nfsstat $(1)/usr/sbin
endef

define Package/nfs-utils/install
	$(INSTALL_DIR) $(1)/sbin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/sbin/mount.nfs $(1)/sbin/
	(cd $(1)/sbin; ln -sf mount.nfs mount.nfs4; ln -sf mount.nfs umount.nfs; ln -sf mount.nfs umount.nfs4)
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,nfs-kernel-server))
$(eval $(call BuildPackage,nfs-kernel-server-utils))
$(eval $(call BuildPackage,nfs-utils))
