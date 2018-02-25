#
# Main Makefile for building SMART HOME SWITCH
# 

ifndef PRODUCT_NAME
$(error "You have to define Product Name to use this Makefile")
endif

ifndef BUILD_TYPE
export BUILD_TYPE=squashfs
endif

#
# Include the variables
#

include rules.mk

#
# Include the product specific make file
#

include profile/$(PRODUCT_NAME)/Makefile.$(PRODUCT_NAME)

ifndef BOARD_TYPE
$(error "You have to define Board Type to use this Makefile")
endif

ifndef WIRELESSTOOLNAMES
$(warning "Should specify WIRELESSTOOLNAMES value")
## Note: WIRELESSTOOLNAMES can contain more files that we actually have...
## e.g. WIRELESSTOOLNAMES := athstats athstatsclr athdebug 80211stats 80211debug \
		athkey athampdutrc athcwm atrc pktlogconf pktlogdump radartool
WIRELESSTOOLNAMES :=
endif

#
# Common targts
#

rootfs_prep:
	@echo Begin rootfs_prep $(INSTALL_ROOT)
	rm -rf $(IMAGE_ROOT)
	rm -rf $(INSTALL_ROOT); mkdir $(INSTALL_ROOT)
#	cp -R ../rootfs/cgiCommon/* $(INSTALL_ROOT)
#	cp -rf ../rootfs/cgiCommon/etc/ath.$(AP_TYPE)/* $(INSTALL_ROOT)/etc/ath
	rm -rf $(INSTALL_ROOT)/etc/ath.single $(INSTALL_ROOT)/etc/ath.dual
#	cp -rf ../rootfs/cgiCommon/usr/www.$(AP_TYPE)/* $(INSTALL_ROOT)/usr/www
	rm -rf $(INSTALL_ROOT)/usr/www.single
	rm -rf $(INSTALL_ROOT)/usr/www.dual
	rm -rf $(INSTALL_ROOT)/etc/ath/hostapd0.7.0_conf
ifeq ($(KERNELVER),2.6.31)
	rm -rf $(INSTALL_ROOT)/lib/modules/2.6.15
endif
	mkdir -p $(INSTALL_ROOT)/lib/modules/$(KERNELVER)
	if test -d "../rootfs/$(BOARD_TYPE)$(BUILD_CONFIG)" ; then  \
 	    cp -Rf ../rootfs/$(BOARD_TYPE)$(BUILD_CONFIG)/* $(INSTALL_ROOT) ; \
        else \
 	    cp -Rf ../rootfs/$(BOARD_TYPE)/* $(INSTALL_ROOT) ; \
	fi;
#	cp profile/$(PRODUCT_NAME)/smartd.conf	$(INSTALL_ROOT)/etc/
	cp key/public.key	$(INSTALL_ROOT)/etc/
	cp key/2048_newroot.cer	$(INSTALL_ROOT)/etc/
	chmod 755 $(INSTALL_ROOT)/etc/rc.d/*
	chmod 755 $(INSTALL_ROOT)/etc/ath/*
#	mv -f $(INSTALL_ROOT)/etc/ath/wpscli_ap $(INSTALL_ROOT)/sbin
#	mv -f $(INSTALL_ROOT)/etc/ath/wpscli_sta $(INSTALL_ROOT)/sbin
	@echo End rootfs_prep $(INSTALL_ROOT)

#
# Build toolchain, takes a long time to build
#
	
ifeq ($(BUILD_UCLIBC_DEBUG),y)
BUILD_UCLIBC_DEBUG_FILTER = sed -e '/^DODEBUG=n/s/=n/=y/'
else
BUILD_UCLIBC_DEBUG_FILTER = "cat"
endif
toolchain_build: rootfs_prep
	echo "***** Checking toolchain status .."
	if [ "$(TOOLCHAIN)" = "gcc-4.3.3" -o "$(TOOLCHAIN)" = "gcc-4.5.1" ]; then \
	if test -f "../toolchain/$(TOOLCHAIN)/$(TOOLARCH)/.tcbuilt"; then echo "***** Toolchain already built.... ******"; \
	else \
	cd ../toolchain/$(TOOLCHAIN) && \
	cp -f $(BOARD_TYPE).config .config && \
	rm -f toolchain/uClibc/uClibc.config && \
	cat toolchain/uClibc/$(BOARD_TYPE).config | $(BUILD_UCLIBC_DEBUG_FILTER) >toolchain/uClibc/uClibc.config && \
	$(MAKE) && cp -R ../mtd $(TOOLARCH)/include && \
	touch $(TOOLARCH)/.tcbuilt; \
	fi;     \
	else \
	cd ../toolchain/$(TOOLCHAIN) && \
	cp -f $(BOARD_TYPE).config .config && \
	rm -f toolchain/uClibc/uClibc.config && \
	cat toolchain/uClibc/$(BOARD_TYPE).config | $(BUILD_UCLIBC_DEBUG_FILTER) >toolchain/uClibc/uClibc.config && \
	$(MAKE); \
	fi;
	mkdir -p $(INSTALL_ROOT)/lib && \
	cd ../toolchain/$(TOOLCHAIN) && $(MAKE) PREFIX=$(INSTALL_ROOT) install;
	# gdbserver to support debugging (if it has been created)
	if [ -f $(TOOLPATH)/root/usr/bin/gdbserver ] ; then \
	    mkdir -p $(INSTALL_ROOT)/usr/bin ; \
	    cp -f $(TOOLPATH)/root/usr/bin/gdbserver $(INSTALL_ROOT)/usr/bin/. ; \
	fi
	@echo End Making toolchain_build at `date`

#toolchain_prep:
#	@echo Prepareing toolchain ....
#	if test -f "../toolchain/$(TOOLCHAIN)/$(TOOLARCH)/.tcbuilt"; then echo "***** Toolchain already built.... ******"; \
#	else \
#	cd ../toolchain && \
#	tar -zxvf $(TOOLCHAIN_TGZ) && \
#	mkdir -p $(CCACHE_DIR); \
#	fi;
#	mkdir -p $(INSTALL_ROOT)/lib && \
#	cp -P $(STAGING_DIR)/lib/*so* $(INSTALL_ROOT)/lib && \
#	cp -P $(STAGING_DIR)/usr/mips-linux-uclibc/lib/libgcc* $(INSTALL_ROOT)/lib;
	
toolchain_clean:
	@echo Cleaning toolchain ....
	#cd ../toolchain/$(TOOLCHAIN) &&  $(MAKE) clean && rm ../toolchain/$(TOOLCHAIN)/$(TOOLARCH)/.tcbuilt
	rm -rf ../toolchain/$(TOOLCHAIN)
	
check_tftp: image_prep
	mkdir -p $(IMAGEPATH)
	if test -d $(TFTPPATH); then echo $(TFTPPATH) exists; else mkdir $(TFTPPATH); fi;

image_prep:
	if test -d $(TOPDIR)/images; then echo $(TOPDIR)/images exists; \
	else \
	mkdir $(TOPDIR)/images; \
	fi;
	if test -d $(IMAGEPATH); then echo $(IMAGEPATH) exists; \
	else \
	mkdir -p $(IMAGEPATH); \
	fi;
	if test -d $(TOPDIR)/modules; then echo $(TOPDIR)/modules exists; \
	else \
	mkdir $(TOPDIR)/modules; \
	fi;
	if test -d $(MODCPPATH); then echo $(MODCPPATH) exists; \
	else \
	mkdir -p $(MODCPPATH); \
	fi;
	
#
# Use different kernel_build rules depending on the type of target
#

ifeq ($(KERNELTARGET), zImage)
kernel_build: image_prep
	@echo " Making Kernel Build Version $(EXTRAVERSION)" at `date`
	exit
	cd $(KERNELPATH) && $(MAKEARCH) $(BOARD_TYPE)$(BUILD_CONFIG)_defconfig
	cd $(KERNELPATH) && $(MAKEARCH)
	cd $(KERNELPATH)/arch/$(KERNELARCH)/boot && \
	cp $(KERNELTARGET) $(TFTPPATH) && cp $(KERNELTARGET) $(IMAGEPATH)
else
kernel_build: image_prep
	@echo " Making Kernel Build Version $(EXTRAVERSION)" at `date`
#	cd $(KERNELPATH) && $(MAKEARCH) $(BOARD_TYPE)$(KERNEL_BUILD_CONFIG)$(NAND)_defconfig
	cp $(TOPDIR)/build/profile/$(PRODUCT_NAME)/$(PRODUCT_NAME)_defconfig.kernel $(KERNELPATH)/arch/$(KERNELARCH)/configs/$(PRODUCT_NAME)_defconfig
	cd $(KERNELPATH) && $(MAKEARCH) $(PRODUCT_NAME)_defconfig
#	rm -rf $(KERNELPATH)/.config
#	cp $(TOPDIR)/build/profile/$(PRODUCT_NAME)/$(PRODUCT_NAME)_defconfig.kernel $(KERNELPATH)/.config
	cd $(KERNELPATH) && $(MAKEARCH)
	cd $(KERNELPATH) && $(MAKEARCH) $(KERNELTARGET)
	cd $(KERNELPATH)/arch/$(KERNELARCH)/boot && \
#	cp $(KERNELTARGET) $(TFTPPATH)/$(KTFILE) && \
	cp $(KERNELTARGET) $(IMAGEPATH)/$(KTFILE);
#	gzip -9c $(KERNELTARGET) > $(KERNELTARGET).gz && \
#	cp $(KERNELTARGET).gz $(TFTPPATH)/$(KTFILE).gz && \
#	cp $(KERNELTARGET).gz $(IMAGEPATH)/$(KTFILE).gz;
endif

kernel_mconfig:
	cp $(TOPDIR)/build/profile/$(PRODUCT_NAME)/$(PRODUCT_NAME)_defconfig.kernel $(KERNELPATH)/.config
	cd $(KERNELPATH) && $(MAKEARCH) menuconfig && \
	cp .config $(TOPDIR)/build/profile/$(PRODUCT_NAME)/$(PRODUCT_NAME)_defconfig.kernel;

kernel_clean: 
	cd $(KERNELPATH) &&  $(MAKEARCH) mrproper 

kernel_info:
	cd $(KERNELPATH) && rm -f $(INFOFILE)
	cd $(KERNELPATH) && echo "entry:"${ENTRY} >> $(INFOFILE)
	cd $(KERNELPATH) && echo "link: 0x"${LDADR} >> $(INFOFILE)
	cp $(KERNELPATH)/$(INFOFILE) $(TFTPPATH)
	cp $(KERNELPATH)/$(INFOFILE) $(IMAGEPATH)

uimage:	kernel_build
	@echo Making uImage at `date`
	cd ../util && mkuImage.sh $(UBOOTDIR)/tools $(KERNELPATH) "$(BUILD_CONFIG)$(BUILD_EXT)"
	@echo Done Making uImage at `date`

lzma_uimage: kernel_build
	@echo "Generating LZMA'ed kernel....."
	cd $(KERNELPATH)/arch/$(KERNELARCH)/boot && \
	rm -rf $(KERNELTARGET).lzma && \
	$(TOPDIR)/util/lzma --best --keep $(KERNELTARGET) && \
#	cp $(KERNELTARGET).lzma $(TFTPPATH)/$(KTFILE).lzma && \
	cp $(KERNELTARGET).lzma $(IMAGEPATH)/$(KTFILE).lzma;
	echo Making uImage...
	cd ../util && ./mkuImage.sh $(UBOOTDIR)/tools $(KERNELPATH) $(BUILD_CONFIG) lzma

#
# Build uboot
#

uboot_build:image_prep
	@echo Making uboot at `date`
ifneq ($(BOARD_TYPE), $(temp_BOARD_TYPE))
	echo ====Using $(temp_BOARD_TYPE) config for $(BOARD_TYPE) ===
	cd $(UBOOTDIR) && $(MAKEARCH) $(temp_BOARD_TYPE)_config
else
	cd $(UBOOTDIR) && $(MAKEARCH) $(BOARD_TYPE)_config
endif
	@echo ========= build dir: $(TOPDIR)/build ============
	cd $(UBOOTDIR) && $(MAKEARCH) all BUILD_DIR=$(TOPDIR)/build
	cp -f $(UBOOTDIR)/${UBOOT_BINARY} ${IMAGEPATH}/${UBOOTFILE}
#	cp -f $(UBOOTDIR)/${UBOOT_BINARY} $(TFTPPATH)/${UBOOTFILE}
	@echo End Making uboot at `date`
	
uboot_clean:
	cd $(UBOOTDIR) && $(MAKEARCH) mrproper && rm -rf u-boot.bin.lzma

	