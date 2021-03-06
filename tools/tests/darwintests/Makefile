PROJECT := xnu/darwintests

# When building as part of xnu_tests, we get passed a DSTROOT that's got the
# unit test path in it already.  But, BASEDSTROOT doesn't, so use that instead.
ifdef BASEDSTROOT
override DSTROOT = $(BASEDSTROOT)
endif

ENABLE_LTE_TESTS=YES

OTHER_LTE_INCLUDE_FILES += \
	/System/Library/PrivateFrameworks/LoggingSupport.framework, \
	/System/Library/PrivateFrameworks/MobileKeyBag.framework, \
	/usr/local/lib/libdarwintest_utils.dylib, \
	/usr/lib/libapple_crypto.dylib,

DEVELOPER_DIR ?= /Applications/Xcode.app/Contents/Developer/

# the xnu build system will only ever call us with the default target
.DEFAULT_GOAL := install

include $(DEVELOPER_DIR)/AppleInternal/Makefiles/darwintest/Makefile.common

OTHER_CFLAGS  = -Weverything -Wno-gnu-union-cast -Wno-missing-field-initializers -Wno-partial-availability
OTHER_CFLAGS += -Wno-missing-noreturn -Wno-vla -Wno-reserved-id-macro -Wno-documentation-unknown-command
OTHER_CFLAGS += -Wno-padded -Wno-used-but-marked-unused -Wno-covered-switch-default -Wno-nullability-extension
OTHER_CFLAGS += --std=gnu11 -isystem $(SDKROOT)/System/Library/Frameworks/System.framework/PrivateHeaders
OTHER_CFLAGS += -DT_NAMESPACE_PREFIX=xnu
OTHER_CFLAGS += -F $(SDKROOT)/System/Library/PrivateFrameworks

CODESIGN:=$(shell xcrun -sdk "$(TARGETSDK)" -find codesign)
CODESIGN_ALLOCATE:=$(shell xcrun -sdk "$(TARGETSDK)" -find codesign_allocate)

# to have custom compiler flags to
# target: OTHER_CFLAGS += <my flags>

avx: INVALID_ARCHS = i386
avx: OTHER_CFLAGS += -mavx512f -mavx512bw -mavx512vl
avx: OTHER_CFLAGS += -I$(SDKROOT)/System/Library/Frameworks/System.framework/PrivateHeaders
avx: CONFIG_FLAGS := $(filter-out -O%,$(CONFIG_FLAGS))
# Level 2 optimization must be used to prevent compiler from generating
# invalid instructions when compiling with AVX-512 flags.
avx: CONFIG_FLAGS += -O2
ifneq (osx,$(TARGET_NAME))
EXCLUDED_SOURCES += avx.c
endif

backtracing: OTHER_LDFLAGS += -framework CoreSymbolication

data_protection: OTHER_LDFLAGS += -framework IOKit

kdebug: INVALID_ARCHS = i386
kdebug: OTHER_LDFLAGS = -framework ktrace

EXCLUDED_SOURCES += kperf_helpers.c xnu_quick_test_helpers.c

ifeq ($(PLATFORM),iPhoneOS)
CONFIG_FREEZE_DEFINE:= -DCONFIG_FREEZE
else
CONFIG_FREEZE_DEFINE:=
EXCLUDED_SOURCES += jumbo_va_spaces_28530648.c
endif

perf_compressor: OTHER_CFLAGS += $(CONFIG_FREEZE_DEFINE)
perf_compressor: CODE_SIGN_ENTITLEMENTS=./private_entitlement.plist

stackshot: OTHER_LDFLAGS += -lkdd -framework Foundation
stackshot: INVALID_ARCHS = i386

memorystatus_zone_test: INVALID_ARCHS = i386
memorystatus_zone_test: OTHER_CFLAGS += -isystem $(SDKROOT)/System/Library/Frameworks/System.framework/PrivateHeaders
memorystatus_zone_test: OTHER_LDFLAGS += -framework ktrace
memorystatus_zone_test: OTHER_LDFLAGS += -ldarwintest_utils

kpc: OTHER_LDFLAGS += -framework kperf

kperf: INVALID_ARCHS = i386
kperf: OTHER_CFLAGS += kperf_helpers.c
kperf: OTHER_LDFLAGS += -framework kperf -framework kperfdata -framework ktrace -ldarwintest_utils

kperf_backtracing: INVALID_ARCHS = i386
kperf_backtracing: OTHER_CFLAGS += kperf_helpers.c
kperf_backtracing: OTHER_LDFLAGS += -framework kperf -framework kperfdata -framework ktrace
kperf_backtracing: OTHER_LDFLAGS += -framework CoreSymbolication

kevent_qos: OTHER_CFLAGS += -Wno-unused-macros
kevent_qos: OTHER_CFLAGS += -I $(OBJROOT)/

mach_get_times: OTHER_LDFLAGS += -ldarwintest_utils

monotonic_core: OTHER_LDFLAGS += -framework ktrace
monotonic_core: INVALID_ARCHS = i386

perf_exit: OTHER_LDFLAGS = -framework ktrace
perf_exit: INVALID_ARCHS = i386
perf_exit: CODE_SIGN_ENTITLEMENTS=./private_entitlement.plist

perf_spawn_fork: CODE_SIGN_ENTITLEMENTS=./private_entitlement.plist

os_thread_self_restrict: os_thread_self_restrict.c os_thread_self_restrict-entitlements.plist
os_thread_self_restrict: CODE_SIGN_ENTITLEMENTS=os_thread_self_restrict-entitlements.plist

task_inspect: CODE_SIGN_ENTITLEMENTS = task_inspect.entitlements
task_inspect: OTHER_CFLAGS += -DENTITLED=1

CUSTOM_TARGETS += perf_exit_proc

perf_exit_proc:
	$(CC) $(DT_CFLAGS) $(OTHER_CFLAGS) $(CFLAGS) $(DT_LDFLAGS) $(OTHER_LDFLAGS) $(LDFLAGS) perf_exit_proc.c -o $(SYMROOT)/perf_exit_proc

install-perf_exit_proc: perf_exit_proc
	mkdir -p $(INSTALLDIR)
	cp $(SYMROOT)/perf_exit_proc $(INSTALLDIR)/

perf_kdebug: INVALID_ARCHS = i386

stackshot_idle_25570396: INVALID_ARCHS = i386
stackshot_idle_25570396: OTHER_LDFLAGS += -lkdd -framework Foundation

stackshot_block_owner_14362384: INVALID_ARCHS = i386
stackshot_block_owner_14362384: OTHER_LDFLAGS += -framework Foundation -lpthread -lkdd

xnu_quick_test: OTHER_CFLAGS += xnu_quick_test_helpers.c
	
ifeq ($(PLATFORM),iPhoneOS)
OTHER_TEST_TARGETS += jumbo_va_spaces_28530648_unentitled
jumbo_va_spaces_28530648: CODE_SIGN_ENTITLEMENTS = jumbo_va_spaces_28530648.entitlements
jumbo_va_spaces_28530648: OTHER_CFLAGS += -DENTITLED=1
jumbo_va_spaces_28530648: OTHER_LDFLAGS += -ldarwintest_utils

jumbo_va_spaces_28530648_unentitled: OTHER_LDFLAGS += -ldarwintest_utils
jumbo_va_spaces_28530648_unentitled: jumbo_va_spaces_28530648.c
	$(CC) $(DT_CFLAGS) $(OTHER_CFLAGS) $(CFLAGS) $(DT_LDFLAGS) $(OTHER_LDFLAGS) $(LDFLAGS) $< -o $(SYMROOT)/$@
endif

task_info_28439149: CODE_SIGN_ENTITLEMENTS = ./task_for_pid_entitlement.plist

disk_mount_conditioner: disk_mount_conditioner*
disk_mount_conditioner: CODE_SIGN_ENTITLEMENTS=./disk_mount_conditioner-entitlements.plist
disk_mount_conditioner: OTHER_LDFLAGS += -ldarwintest_utils

OTHER_TEST_TARGETS += disk_mount_conditioner_unentitled
disk_mount_conditioner_unentitled: OTHER_CFLAGS += -DTEST_UNENTITLED
disk_mount_conditioner_unentitled: OTHER_LDFLAGS += -ldarwintest_utils
disk_mount_conditioner_unentitled: disk_mount_conditioner.c
	$(CC) $(DT_CFLAGS) $(OTHER_CFLAGS) $(CFLAGS) $(DT_LDFLAGS) $(OTHER_LDFLAGS) $(LDFLAGS) $< -o $(SYMROOT)/$@

work_interval_test: CODE_SIGN_ENTITLEMENTS = work_interval_test.entitlements
work_interval_test: OTHER_CFLAGS += -DENTITLED=1

settimeofday_29193041_entitled: CODE_SIGN_ENTITLEMENTS = settimeofday_29193041.entitlements
thread_group_set_32261625: OTHER_LDFLAGS = -framework ktrace
thread_group_set_32261625: INVALID_ARCHS = i386

task_info: CODE_SIGN_ENTITLEMENTS = task_for_pid_entitlement.plist

include $(DEVELOPER_DIR)/AppleInternal/Makefiles/darwintest/Makefile.targets
