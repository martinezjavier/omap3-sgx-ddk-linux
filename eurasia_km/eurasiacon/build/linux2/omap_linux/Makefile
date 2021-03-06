########################################################################### ###
#@Title         Root makefile for OMAP4430 Linux.  Builds everything else.
#@Copyright     Copyright (c) Imagination Technologies Ltd. All Rights Reserved
#@License       Dual MIT/GPLv2
# 
# The contents of this file are subject to the MIT license as set out below.
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
# 
# Alternatively, the contents of this file may be used under the terms of
# the GNU General Public License Version 2 ("GPL") in which case the provisions
# of GPL are applicable instead of those above.
# 
# If you wish to allow use of your version of this file only under the terms of
# GPL, and not to allow others to use your version of this file under the terms
# of the MIT license, indicate your decision by deleting the provisions above
# and replace them with the notice and other provisions required by GPL as set
# out in the file called "GPL-COPYING" included in this distribution. If you do
# not delete the provisions above, a recipient may use your version of this file
# under the terms of either the MIT license or GPL.
# 
# This License is also included in this distribution in the file called
# "MIT-COPYING".
# 
# EXCEPT AS OTHERWISE STATED IN A NEGOTIATED AGREEMENT: (A) THE SOFTWARE IS
# PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING
# BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
# PURPOSE AND NONINFRINGEMENT; AND (B) IN NO EVENT SHALL THE AUTHORS OR
# COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
# IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
# CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
### ###########################################################################

# If a product wasn't specified, we're an OMAP4430 blaze.
#
TARGET_PRODUCT ?= blaze

# Customize this build as per the TARGET_PRODUCT setting
#
ifneq ($(filter blaze blaze_tablet panda,$(TARGET_PRODUCT)),)
SGXCORE					:= 540
SGX_CORE_REV				:= 120
XORG_TOOLCHAIN ?= tarballs-omap4-ubuntu-10.10-cross
endif
ifneq ($(filter blaze.4470 blaze_tablet.4470,$(TARGET_PRODUCT)),)
SGXCORE					:= 544
SGX_CORE_REV				:= 112
endif
ifneq ($(filter omap5sevm panda5,$(TARGET_PRODUCT)),)
SGXCORE					:= 544
SGX_CORE_REV				:= 105
SGX_FEATURE_MP				:= 1
SGX_FEATURE_SYSTEM_CACHE		:= 1
SGX_FEATURE_MP_CORE_COUNT		:= 2
PVR_OMAPLFB_DRM_FB			:= 0

# FIXME: Re-enable this ASAP
SUPPORT_ACTIVE_POWER_MANAGEMENT 	:= 0
endif

PVR_SYSTEM		:= omap

KERNEL_COMPONENTS := srvkm bufferclass_example

include ../kernel_version.mk

# Only enable active power management if passive power management is
# enabled, as indicated by LDM_PLATFORM being set to 1.  On OMAP,
# the system can suspend in the case where active power management is
# enabled in the SGX driver, but passive power management isn't. As
# passive power management isn't enabled, the driver won't see the
# system suspend/resume events, and so won't take appropriate action.
LDM_PLATFORM ?= 1

ifeq ($(LDM_PLATFORM),1)
SUPPORT_LINUX_USING_WORKQUEUES := 1
DISPLAY_CONTROLLER_COMPONENT += dc_omapfb3_linux
DISPLAY_CONTROLLER := omaplfb
else
SUPPORT_LINUX_USING_SHARED_WORKQUEUES := 1
OMAP_NON_FLIP_DISPLAY := 1
DISPLAY_CONTROLLER_COMPONENT += linux_framebuffer
DISPLAY_CONTROLLER := pvrlfb
endif

SYS_CFLAGS := -march=armv7-a

# Ubuntu ARM compilers need an explicit -marm option to build native ARM code,
# rather than ARM thumb code.
UBUNTU_TOOLCHAIN :=
ifeq ($(CROSS_COMPILE),)
UBUNTU_TOOLCHAIN = 1
endif
ifeq ($(CROSS_COMPILE),arm-linux-gnueabi-)
UBUNTU_TOOLCHAIN = 1
endif
ifeq ($(CROSS_COMPILE),arm-linux-gnueabihf-)
UBUNTU_TOOLCHAIN = 1
endif

ifneq ($(UBUNTU_TOOLCHAIN),)
SYS_CFLAGS += -marm
else
OPTIM := -Os

ifneq ($(CROSS_COMPILE),)
SYS_CFLAGS += -mtls-dialect=arm
endif

SUPPORT_OMAP4430_NEON ?= 1

ifeq ($(SUPPORT_OMAP4430_NEON),1)
SYS_CFLAGS += -ftree-vectorize -mfpu=neon -mfloat-abi=softfp
endif
endif	# UBUNTU_TOOLCHAIN

LIBGCC := $(shell $(CROSS_COMPILE)gcc -print-libgcc-file-name)

SGX_DYNAMIC_TIMING_INFO := 1
SYS_CUSTOM_POWERLOCK_WRAP := 1

ifeq ($(OMAP_NON_FLIP_DISPLAY),1)
OPK_DEFAULT := libpvrPVR2D_BLITWSEGL.so
else
OPK_DEFAULT := libpvrPVR2D_FLIPWSEGL.so
endif

ifeq ($(call kernel-version-at-least,2,6,35),true)
# Work around flipping problems seen with the Taal LCDs on Blaze.
# The work around is safe to use with other types of screen on Blaze
# (e.g. HDMI) and on other platforms (e.g. Panda board).
PVR_OMAPLFB_DONT_USE_FB_PAN_DISPLAY := 1
ifeq ($(LDM_PLATFORM),1)
PVR_LDM_PLATFORM_PRE_REGISTERED := 1
ifeq ($(call kernel-version-at-least,2,6,35,7),true)
# Not all variants of the OMAP4 kernel have a DRM based framebuffer.
# Note that a non-X.Org version of the driver does not have to be built
# with DRM support if the kernel has a DRM based framebuffer.
PVR_OMAPLFB_DRM_FB ?= 1
ifeq ($(PVR_OMAPLFB_DRM_FB),1)
PVR_LDM_PLATFORM_PRE_REGISTERED_DEV := "\"pvrsrvkm_pvr\""
# There is already a "pvrsrvkm" driver (part of the DRM framebuffer code),
# so use the pre-registered device name instead.
PVR_LDM_DRIVER_REGISTRATION_NAME := "\"pvrsrvkm_pvr"\"
# The DRM library will not load the Services module on behalf of the X Server,
# as a DRM module has already been loaded (the DRM based framebuffer), so
# load the Services module before starting the X Server.
XORG_EXPLICIT_PVR_SERVICES_LOAD := 1
else
PVR_LDM_PLATFORM_PRE_REGISTERED_DEV := "\"pvrsrvkm\""
endif
endif
endif
endif

include ../common/xorg_test.mk
ifeq ($(want_xorg),1)
SUPPORT_DRI_DRM := 1
endif

ifeq ($(SUPPORT_DRI_DRM),1)
ifeq ($(call kernel-version-at-least,2,6,35),true)
PVR_DRI_DRM_PLATFORM_DEV := 1
PVR_DRI_DRM_STATIC_BUS_ID := 1
ifeq ($(call kernel-version-at-least,2,6,35,7),true)
ifeq ($(PVR_OMAPLFB_DRM_FB),1)
SUPPORT_DRI_DRM_PLUGIN := 1
endif
ifeq ($(call kernel-version-at-least,2,6,36),true)
PVR_DRI_DRM_DEV_BUS_ID := "\"platform:pvrsrvkm"\"
else
PVR_DRI_DRM_DEV_BUS_ID := "\"platform:pvrsrvkm:00"\"
endif	# kernel-version-at-least,2,6,36
endif	# kernel-version-at-least,2,6,35,7
else	# kernel-version-at-least,2,6,35
PVR_DRI_DRM_NOT_PCI := 1
endif	# kernel-version-at-least,2,6,35
endif	# SUPPORT_DRI_DRM

ifeq ($(want_xorg),1)
ifeq ($(call kernel-version-at-least,2,6,35,7),true)
# A client DRI authorisation failure, whilst switched away from the X Server
# VT, prevents all other attempts at DRI authorisation, even after
# switching back to the X server VT, so don't perform a DRM drop master
# call.
PVR_XORG_DONT_DROP_MASTER_IN_LEAVE_VT := 1
endif
XORG_PVR_CONF := omap4
XORG_PVR_VIDEO := omap4

OPK_FALLBACK := libpvrPVR2D_DRIWSEGL.so

ifneq ($(OMAP_NON_FLIP_DISPLAY),1)
XORG_PVR_VIDEO ?= omap4
PVR_DISPLAY_CONTROLLER_DRM_IOCTL := 1
endif

else  # xorg isn't excluded

OPK_FALLBACK := libpvrPVR2D_BLITWSEGL.so

endif # xorg isn't excluded

ifeq ($(SUPPORT_DRI_DRM),1)
ifeq ($(PVR_DRI_DRM_NOT_PCI),1)
KERNEL_COMPONENTS += linux_drm
EXTRA_KBUILD_SOURCE := $(KERNELDIR)
endif
EXTRA_PVRSRVKM_COMPONENTS += $(DISPLAY_CONTROLLER_COMPONENT)
else
KERNEL_COMPONENTS += $(DISPLAY_CONTROLLER_COMPONENT)
endif

FORCE_ENABLE_GROW_SHRINK ?= 1

include ../config/core.mk
include ../common/xorg.mk
include ../common/dridrm.mk
include ../common/opencl.mk
include ../common/omap4.mk

# We only need this for pvr_video's includes, which should
# really be done differently, as DISPLAY_CONTROLLER_DIR is
# now obsolete..
#
$(eval $(call UserConfigMake,DISPLAY_CONTROLLER_DIR,3rdparty/$(DISPLAY_CONTROLLER_COMPONENT)))
