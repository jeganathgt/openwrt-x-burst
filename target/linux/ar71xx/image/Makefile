#
# Copyright (C) 2008-2009 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

IMGNAME:=$(BIN_DIR)/openwrt-$(BOARD)

define imgname
$(IMGNAME)-$(2)-$(patsubst jffs2-%,jffs2,$(1))
endef

VMLINUX:=$(IMGNAME)-vmlinux
ifeq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),y)
	VMLINUX:=$(IMGNAME)-vmlinux-initramfs
endif

define Image/BuildKernel
	cp $(KDIR)/vmlinux.elf $(VMLINUX).elf
	cp $(KDIR)/vmlinux $(VMLINUX).bin
	gzip -9 -c $(KDIR)/vmlinux > $(KDIR)/vmlinux.bin.gz
	$(STAGING_DIR_HOST)/bin/lzma e $(KDIR)/vmlinux $(KDIR)/vmlinux.bin.l7
	$(STAGING_DIR_HOST)/bin/lzma e $(KDIR)/vmlinux -lc1 -lp2 -pb2 $(KDIR)/vmlinux.bin.lzma
	dd if=$(KDIR)/vmlinux.bin.l7 of=$(VMLINUX).lzma bs=65536 conv=sync
	dd if=$(KDIR)/vmlinux.bin.gz of=$(VMLINUX).gz bs=65536 conv=sync
	mkimage -A mips -O linux -T kernel -a 0x80060000 -C gzip -e \
		0x80060000 \
		-n 'MIPS OpenWrt Linux-$(LINUX_VERSION)' \
		-d $(KDIR)/vmlinux.bin.gz $(IMGNAME)-uImage-gzip.bin
	mkimage -A mips -O linux -T kernel -a 0x80060000 -C lzma -e \
		0x80060000 \
		-n 'MIPS OpenWrt Linux-$(LINUX_VERSION)' \
		-d $(KDIR)/vmlinux.bin.lzma $(IMGNAME)-uImage-lzma.bin
endef

define Image/Build/Linksys8M
	cp $(KDIR)/vmlinux $(KDIR)/vmlinux-$(2)
	$(STAGING_DIR_HOST)/bin/patch-cmdline $(KDIR)/vmlinux-$(2) '$(strip $(3))'
	$(STAGING_DIR_HOST)/bin/lzma e $(KDIR)/vmlinux-$(2) $(KDIR)/vmlinux-$(2).bin.lzma
	if [ `stat -c%s "$(KDIR)/vmlinux-$(2).bin.lzma"` -gt 1310720 ]; then \
		echo "Warning: $(KDIR)/vmlinux-$(2).bin.lzma is too big"; \
	else if [ `stat -c%s $(KDIR)/root.$(1)` -gt 6488064 ]; then \
		echo "Warning: $(KDIR)/root.$(1) is too big"; \
	else \
		mkimage -A mips -O linux -T kernel -a 0x80060000 -C lzma -e \
			0x80060000 \
			-n 'MIPS OpenWrt Linux-$(LINUX_VERSION)' \
			-d $(KDIR)/vmlinux-$(2).bin.lzma $(KDIR)/vmlinux-$(2).uImage; \
		( \
			dd if=$(KDIR)/vmlinux-$(2).uImage bs=1280k conv=sync; \
			dd if=$(KDIR)/root.$(1) bs=6336k conv=sync; \
			echo -n $(4); \
		) > $(call imgname,$(1),$(2)).bin; \
	fi; fi
endef

define Image/Build/AP81
	cp $(KDIR)/vmlinux $(KDIR)/vmlinux-$(2)
	$(STAGING_DIR_HOST)/bin/patch-cmdline $(KDIR)/vmlinux-$(2) '$(strip $(3))'
	$(STAGING_DIR_HOST)/bin/lzma e $(KDIR)/vmlinux-$(2) $(KDIR)/vmlinux-$(2).bin.lzma
	if [ `stat -c%s "$(KDIR)/vmlinux-$(2).bin.lzma"` -gt 786432 ]; then \
		echo "Warning: $(KDIR)/vmlinux-$(2).bin.lzma is too big"; \
	else if [ `stat -c%s $(KDIR)/root.$(1)` -gt 3080192 ]; then \
		echo "Warning: $(KDIR)/root.$(1) is too big"; \
	else \
		mkimage -A mips -O linux -T kernel -a 0x80060000 -C lzma -e \
			0x80060000 \
			-n 'MIPS OpenWrt Linux-$(LINUX_VERSION)' \
			-d $(KDIR)/vmlinux-$(2).bin.lzma $(KDIR)/vmlinux-$(2).uImage; \
		( \
			dd if=$(KDIR)/vmlinux-$(2).uImage bs=768k conv=sync; \
			dd if=$(KDIR)/root.$(1) bs=3008k conv=sync; \
			echo -n $(4); \
		) > $(call imgname,$(1),$(2)).uni; \
	fi; fi
endef

define Image/Build/AP83
	cp $(KDIR)/vmlinux $(KDIR)/vmlinux-$(2)
	$(STAGING_DIR_HOST)/bin/patch-cmdline $(KDIR)/vmlinux-$(2) '$(strip $(3))'
	gzip -9 -c $(KDIR)/vmlinux-$(2) > $(KDIR)/vmlinux-$(2).bin.gz
	if [ `stat -c%s "$(KDIR)/vmlinux-$(2).bin.gz"` -gt 1310720 ]; then \
		echo "Warning: $(KDIR)/vmlinux-$(2).bin.gz is too big"; \
	else if [ `stat -c%s $(KDIR)/root.$(1)` -gt 6619136 ]; then \
		echo "Warning: $(KDIR)/root.$(1) is too big"; \
	else \
		mkimage -A mips -O linux -T kernel -a 0x80060000 -C gzip -e \
			0x80060000 \
			-n 'MIPS OpenWrt Linux-$(LINUX_VERSION)' \
			-d $(KDIR)/vmlinux-$(2).bin.gz \
			$(KDIR)/vmlinux-$(2).uImage; \
		dd if=$(KDIR)/vmlinux-$(2).uImage \
			of=$(call imgname,kernel,$(2)).bin bs=64k conv=sync; \
		dd if=$(KDIR)/root.$(1) \
			of=$(call imgname,$(1),$(2)-rootfs).bin bs=128k conv=sync; \
		( \
			dd if=$(KDIR)/vmlinux-$(2).uImage bs=1280k conv=sync; \
			dd if=$(KDIR)/root.$(1); \
		) > $(call imgname,$(1),$(2)-firmware).bin; \
	fi; fi
endef

define Image/Build/MyLoader
	-$(STAGING_DIR_HOST)/bin/mkmylofw -B $(2) \
		-p0x30000:0xc0000:ahl:0x80060000:kernel:$(KDIR)/vmlinux.bin.lzma  \
		-p0xf0000:0:::rootfs:$(KDIR)/root.$(1) \
		$(call imgname,$(1),$(2)).img
endef

define Image/Build/UBNT
	cp $(KDIR)/vmlinux $(KDIR)/vmlinux-$(2)
	$(STAGING_DIR_HOST)/bin/patch-cmdline $(KDIR)/vmlinux-$(2) '$(strip $(3))'
	$(STAGING_DIR_HOST)/bin/lzma e $(KDIR)/vmlinux-$(2) $(KDIR)/vmlinux-$(2).lzma
	dd if=$(KDIR)/vmlinux-$(2).lzma of=$(KDIR)/vmlinux-$(2).bin.lzma bs=64k conv=sync
	-$(STAGING_DIR_HOST)/bin/mkfwimage \
		-B $(4) -v $(5).$(6).OpenWrt.$(REVISION) \
		-k $(KDIR)/vmlinux-$(2).bin.lzma \
		-r $(BIN_DIR)/openwrt-$(BOARD)-root.$(1) \
		-o $(call imgname,$(1),$(2)).bin
endef

define Image/Build/Planex
	cp $(KDIR)/vmlinux $(KDIR)/vmlinux-$(2)
	$(STAGING_DIR_HOST)/bin/patch-cmdline $(KDIR)/vmlinux-$(2) '$(strip $(3))'
	gzip -9 -c $(KDIR)/vmlinux-$(2) > $(KDIR)/vmlinux-$(2).bin.gzip
	if [ `stat -c%s "$(KDIR)/vmlinux-$(2).bin.gzip"` -gt 1441792 ]; then \
		echo "Warning: $(KDIR)/vmlinux-$(2).bin.gzip is too big"; \
	else if [ `stat -c%s $(KDIR)/root.$(1)` -gt 6356992 ]; then \
		echo "Warning: $(KDIR)/root.$(1) is too big"; \
	else \
		mkimage -A mips -O linux -T kernel -a 0x80060000 -C gzip -e \
			0x80060000 \
			-n 'MIPS OpenWrt Linux-$(LINUX_VERSION)' \
			-d $(KDIR)/vmlinux-$(2).bin.gzip $(KDIR)/vmlinux-$(2).uImage; \
		( \
			dd if=$(KDIR)/vmlinux-$(2).uImage bs=1408k conv=sync; \
			dd if=$(KDIR)/root.$(1) bs=6208k conv=sync; \
		) > $(call imgname,$(1),$(2)).bin; \
		$(STAGING_DIR_HOST)/bin/mkplanexfw \
			-B $(2) \
			-v 2.00.00 \
			-i $(call imgname,$(1),$(2)).bin \
			-o $(call imgname,$(1),$(2)).webui; \
	fi; fi
endef

define Image/Build/TPLINK
	cp $(KDIR)/vmlinux $(KDIR)/vmlinux-$(2)
	$(STAGING_DIR_HOST)/bin/patch-cmdline $(KDIR)/vmlinux-$(2) '$(strip $(3))'
	gzip -9 -c $(KDIR)/vmlinux-$(2) > $(KDIR)/vmlinux-$(2).bin.gzip
	-$(STAGING_DIR_HOST)/bin/mktplinkfw \
		-B $(4) -N OpenWrt -V $(REVISION)\
		-k $(KDIR)/vmlinux-$(2).bin.gzip \
		-r $(BIN_DIR)/openwrt-$(BOARD)-root.$(1) \
		-o $(BIN_DIR)/openwrt-$(BOARD)-$(2)-$(1).uni
endef

define Image/Build/Template/Compex
	$(call Image/Build/MyLoader,$(1),$(2))
endef

define Image/Build/Template/Compex/squashfs
	$(call Image/Build/Template/Compex,squashfs,$(1))
endef

define Image/Build/Template/Linksys8M
	$(call Image/Build/Linksys8M,$(1),$(2),$(3),$(4))
endef

define Image/Build/Template/Linksys8M/squashfs
	$(call Image/Build/Template/Linksys8M,squashfs,$(1),$(2),$(3))
endef

define Image/Build/Template/Linksys8M/jffs2-64k
	$(call Image/Build/Template/Linksys8M,jffs2-64k,$(1),$(2),$(3))
endef

define Image/Build/Template/AP81
	$(call Image/Build/AP81,$(1),$(2),$(3),$(4))
endef

define Image/Build/Template/AP81/squashfs
	$(call Image/Build/Template/AP81,squashfs,$(1),$(2),$(3))
endef

define Image/Build/Template/AP83
	$(call Image/Build/AP83,$(1),$(2),$(3))
endef

define Image/Build/Template/AP83/squashfs
	$(call Image/Build/Template/AP83,squashfs,$(1),$(2))
endef

define Image/Build/Template/AP83/jffs2-64k
	$(call Image/Build/Template/AP83,jffs2-64k,$(1),$(2))
endef

define Image/Build/Template/TPLINK
	$(call Image/Build/TPLINK,$(1),$(2),$(3),$(4))
endef

define Image/Build/Template/TPLINK/squashfs
	$(call Image/Build/Template/TPLINK,squashfs,$(1),$(2),$(3))
endef

define Image/Build/Template/UBNT
	$(call Image/Build/UBNT,$(1),$(2),$(3),$(4),$(5),$(6))
endef

define Image/Build/Template/UBNT/squashfs
	$(call Image/Build/Template/UBNT,squashfs,$(1),$(2),$(3),$(4),$(5))
endef

define Image/Build/Template/UBNT/jffs2-64k
	$(call Image/Build/Template/UBNT,jffs2-64k,$(1),$(2),$(3),$(4),$(5))
endef

define Image/Build/Template/Planex
	$(call Image/Build/Planex,$(1),$(2),$(3))
endef

define Image/Build/Template/Planex/squashfs
	$(call Image/Build/Template/Planex,squashfs,$(1),$(2))
endef

define Image/Build/Template/Planex/jffs2-64k
	$(call Image/Build/Template/Planex,jffs2-64k,$(1),$(2))
endef

define Image/Build/Profile/AP83
	$(call Image/Build/Template/AP83/$(1),ap83,board=AP83)
endef

define Image/Build/Profile/WP543
	$(call Image/Build/Template/Compex/$(1),wp543)
endef

define Image/Build/Profile/DIR615C1
	$(call Image/Build/Template/AP81/$(1),dir-615c1,board=TEW-632BRP,"AP81-AR9130-RT-070614-02")
endef

define Image/Build/Profile/TEW632BRP
	$(call Image/Build/Template/AP81/$(1),tew-632brp,board=TEW-632BRP,"AP81-AR9130-RT-070614-00")
endef

define Image/Build/Profile/TEW652BRP
	$(call Image/Build/Template/AP81/$(1),tew-652brp,board=TEW-632BRP,"AP81-AR9130-RT-080609-05")
endef

define Image/Build/Profile/A02RBW300N
	$(call Image/Build/Template/AP81/$(1),a02-rb-w300n,board=TEW-632BRP,"AP81-AR9130-RT-070614-03")
endef

define Image/Build/Profile/UBNTRS
	$(call Image/Build/Template/UBNT/$(1),ubnt-rs,board=UBNT-RS,RS,RSx,ar7100)
endef

define Image/Build/Profile/UBNTRSPRO
	$(call Image/Build/Template/UBNT/$(1),ubnt-rspro,board=UBNT-RSPRO,RSPRO,RSPRO,ar7100pro)
endef

define Image/Build/Profile/UBNTLSSR71
	$(call Image/Build/Template/UBNT/$(1),ubnt-ls-sr71,board=UBNT-LS-SR71,LS-SR71,LS-SR71,ar7100)
endef

define Image/Build/Profile/UBNT
	$(call Image/Build/Profile/UBNTRS,$(1))
	$(call Image/Build/Profile/UBNTRSPRO,$(1))
	$(call Image/Build/Profile/UBNTLSSR71,$(1))
endef

define Image/Build/Profile/MZKW04NU
	$(call Image/Build/Template/Planex/$(1),mzk-w04nu,board=MZK-W04NU)
endef

define Image/Build/Profile/MZKW300NH
	$(call Image/Build/Template/Planex/$(1),mzk-w300nh,board=MZK-W300NH)
endef

define Image/Build/Profile/TLWR841NDV3
	$(call Image/Build/Template/TPLINK/$(1),tl-wr841ndv3,board=TL-WR941ND,TL-WR841NDv3)
endef

define Image/Build/Profile/TLWR941NDV2
	$(call Image/Build/Template/TPLINK/$(1),tl-wr941ndv2,board=TL-WR941ND,TL-WR941NDv2)
endef

define Image/Build/Profile/WRT400N
	$(call Image/Build/Template/Linksys8M/$(1),wrt400n,board=WRT400N)
endef

define Image/Build/Profile/Default
	$(call Image/Build/Profile/AP83,$(1))
	$(call Image/Build/Profile/A02RBW300N,$(1))
	$(call Image/Build/Profile/DIR615C1,$(1))
	$(call Image/Build/Profile/MZKW04NU,$(1))
	$(call Image/Build/Profile/MZKW300NH,$(1))
	$(call Image/Build/Profile/TEW632BRP,$(1))
	$(call Image/Build/Profile/TEW652BRP,$(1))
	$(call Image/Build/Profile/TLWR841NDV3,$(1))
	$(call Image/Build/Profile/TLWR941NDV2,$(1))
	$(call Image/Build/Profile/UBNT,$(1))
	$(call Image/Build/Profile/WP543,$(1))
	$(call Image/Build/Profile/WRT400N,$(1))
endef

define Image/Build/Profile/Madwifi
	$(call Image/Build/Profile/UBNT,$(1))
	$(call Image/Build/Profile/WP543,$(1))
endef

define Image/Build/squashfs
	$(call prepare_generic_squashfs,$(KDIR)/root.squashfs)
endef

define Image/Build
	$(call Image/Build/$(1))
	dd if=$(KDIR)/root.$(1) of=$(IMGNAME)-root.$(1) bs=128k conv=sync

	$(call Image/Build/Profile/$(PROFILE),$(1))
endef

$(eval $(call BuildImage))
