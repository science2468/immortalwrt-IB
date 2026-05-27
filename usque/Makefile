include $(TOPDIR)/rules.mk

PKG_NAME:=usque
PKG_VERSION:=3.0.0
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/Diniboy1123/usque
PKG_SOURCE_VERSION:=47c1b1bf2c8675d906bc8026515585ced27beef7
PKG_MIRROR_HASH:=skip

PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE

PKG_BUILD_DEPENDS:=golang/host

GO_PKG:=github.com/Diniboy1123/usque

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

GO_PKG_LDFLAGS:=-s -w
# 禁用CGO
override GO_PKG_TARGET_VARS := $(filter-out CGO_ENABLED=1,$(GO_PKG_TARGET_VARS)) CGO_ENABLED=0

# 覆盖默认链接标志，移除 -linkmode external 和 -extldflags
GO_PKG_DEFAULT_LDFLAGS := -buildid= -linkmode internal

define Package/usque
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Usque MASQUE Client
  URL:=https://github.com/Diniboy1123/usque
endef

define Package/usque/description
 Usque is a MASQUE-based proxy client (Cloudflare WARP alternative).
endef

define Package/usque/install
        $(INSTALL_DIR) $(1)/usr/sbin
        $(INSTALL_BIN) $(GO_PKG_BUILD_BIN_DIR)/usque $(1)/usr/sbin/
        $(INSTALL_DIR) $(1)/etc/init.d
        $(INSTALL_BIN) ./files/usque.init $(1)/etc/init.d/usque
        $(INSTALL_DIR) $(1)/etc/usque
endef

$(eval $(call GoBinPackage,usque))
$(eval $(call BuildPackage,usque))
