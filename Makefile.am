#;**********************************************************************;
# Copyright (c) 2018 Intel Corporation
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice,
# this list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
# THE POSSIBILITY OF SUCH DAMAGE.
#;**********************************************************************;

INCLUDE_DIRS    = -I$(srcdir)/src -I$(top_srcdir)/src/lib
ACLOCAL_AMFLAGS = -I m4
AM_CFLAGS       = $(INCLUDE_DIRS) $(EXTRA_CFLAGS) $(CODE_COVERAGE_CFLAGS) \
                  $(TSS2_SYS_CFLAGS) $(SQLITE3_CFLAGS) $(PTHREAD_CFLAGS) \
                  $(CRYPTO_CFLAGS)

AM_LDFLAGS      = $(EXTRA_LDFLAGS) $(CODE_COVERAGE_LIBS) $(TSS2_SYS_LIBS) \
                  $(SQLITE3_LIBS) $(PTHREAD_LIBS) $(CRYPTO_LIBS) \
                  $(TCTI_DEVICE_LIBS) $(TCTI_SOCKET_LIBS) $(TCTI_MSSIM_LIBS)

@CODE_COVERAGE_RULES@

# Add source code files from bootstrap
include src_vars.mk

EXTRA_DIST = \
    LICENSE \
    docs \
    $(top_srcdir)/test/integration/scripts

# Generate the AUTHORS file from git log
AUTHORS :
	$(AM_V_GEN)git log --format='%aN <%aE>' | grep -v 'users.noreply.github.com' | sort | \
	    uniq -c | sort -nr | sed 's/^\s*//' | cut -d" " -f2- > $@
EXTRA_DIST += AUTHORS
CLEANFILES = AUTHORS

# pkg-config setup. pc-file declarations happen in the corresponding modules
pkgconfigdir          = $(libdir)/pkgconfig
nodist_pkgconfig_DATA =
CLEANFILES += $(nodist_pkgconfig_DATA)

%.pc : %.pc.in
	$(AM_V_GEN)$(call make_parent_dir,$@) && \
	sed -e "s,[@]VERSION[@],$(PACKAGE_VERSION),g; \
	        s,[@]libdir[@],$(libdir),g; \
	        s,[@]includedir[@],$(includedir),g;" $^ > $@

### PKCS#11 Library Definition ###
libtpm2_pkcs11 = src/libtpm2_pkcs11.la
lib_LTLIBRARIES = $(libtpm2_pkcs11)
nodist_pkgconfig_DATA += lib/tpm2-pkcs11.pc
EXTRA_DIST += lib/tpm2-pkcs11.map lib/tpm2-pkcs11.pc.in

if HAVE_LD_VERSION_SCRIPT
src_libtpm2_pkcs11_la_LDFLAGS = -Wl,--version-script=$(srcdir)/lib/tpm2-pkcs11.map
endif # HAVE_LD_VERSION_SCRIPT
src_libtpm2_pkcs11_la_LIBADD = $(AM_LDFLAGS) -ldl
src_libtpm2_pkcs11_la_SOURCES = $(LIB_PKCS11_SRC) $(LIB_PKCS11_INTERNAL_LIB_SRC)

# test harness configuration
TEST_EXTENSIONS = .int
AM_TESTS_ENVIRONMENT = \
    TEST_FUNC_LIB=$(srcdir)/test/integration/scripts/int-test-funcs.sh \
    PATH=./src:$(PATH)
INT_LOG_COMPILER = $(srcdir)/test/integration/scripts/int-test-setup.sh

TESTS_LDADD = $(noinst_LTLIBRARIES) $(lib_LTLIBRARIES) $(AM_LDFLAGS)

# Intentionally empty so INTEGRATION and UNIT tests can append to it.
check_PROGRAMS =

### Integration Tests ###
if ENABLE_INTEGRATION

TESTS = $(check_PROGRAMS)
check_PROGRAMS += \
    test/integration/pkcs-find-objects.int \
    test/integration/pkcs-open-close-session.int \
    test/integration/pkcs-get-mechanism.int \
    test/integration/pkcs-login-logout.int \
    test/integration/pkcs-sign-verify.int

XFAIL_TESTS=test/unit/test_pkcs11

INT_LOG_FLAGS = --tabrmd-tcti=mssim --tsetup-script=$(top_srcdir)/test/integration/scripts/create_pkcs_store.sh
test_integration_pkcs_open_close_session_int_CFLAGS  = $(AM_CFLAGS) $(TESTS_CFLAGS)
test_integration_pkcs_open_close_session_int_LDADD   = $(TESTS_LDADD)
test_integration_pkcs_open_close_session_int_SOURCES = test/integration/pkcs-open-close-session.int.c \
    test/integration/main-test.c test/integration/test.h

test_integration_pkcs_find_objects_int_CFLAGS  = $(AM_CFLAGS) $(TESTS_CFLAGS)
test_integration_pkcs_find_objects_int_LDADD   = $(TESTS_LDADD)
test_integration_pkcs_find_objects_int_SOURCES = test/integration/pkcs-find-objects.int.c \
    test/integration/main-test.c test/integration/test.h

test_integration_pkcs_get_mechanism_int_CFLAGS  = $(AM_CFLAGS) $(TESTS_CFLAGS)
test_integration_pkcs_get_mechanism_int_LDADD   = $(TESTS_LDADD)  $(SQLITE3_LIBS)
test_integration_pkcs_get_mechanism_int_SOURCES = test/integration/pkcs-get-mechanism.int.c \
    test/integration/main-test.c test/integration/test.h

test_integration_pkcs_login_logout_int_CFLAGS  = $(AM_CFLAGS) $(TESTS_CFLAGS)
test_integration_pkcs_login_logout_int_LDADD   = $(TESTS_LDADD)  $(SQLITE3_LIBS)
test_integration_pkcs_login_logout_int_SOURCES = test/integration/pkcs-login-logout.int.c \
    test/integration/main-test.c test/integration/test.h

test_integration_pkcs_sign_verify_int_CFLAGS  = $(AM_CFLAGS) $(TESTS_CFLAGS)
test_integration_pkcs_sign_verify_int_LDADD   = $(TESTS_LDADD)  $(SQLITE3_LIBS)
test_integration_pkcs_sign_verify_int_SOURCES = test/integration/pkcs-sign-verify.int.c \
    test/integration/main-test.c test/integration/test.h

endif
# END INTEGRATION

### Unit Tests ###
if UNIT

#
# Due to limitations in how cmocka works, we build a seperate library here so we
# can have a PKCS11 shared object with undefined calls into the rest of the lib
# code. This allows us to deploy a single library, but be able to dissect it for
# testing.
#
### PKCS#11 TEST Library Definitions ###
libtpm2_test_internal = src/libtpm2_test_internal.la
src_libtpm2_test_internal_la_LIBADD = $(AM_LDFLAGS) -ldl
src_libtpm2_test_internal_la_SOURCES = $(LIB_PKCS11_INTERNAL_LIB_SRC)

libtpm2_test_pkcs11 = src/libtpm2_test_pkcs11.la
src_libtpm2_test_pkcs11_la_LIBADD =  $(AM_LDFLAGS) -ldl $(libtpm2_test_internal) -ldl
src_libtpm2_test_pkcs11_la_SOURCES = $(LIB_PKCS11_SRC)

noinst_LTLIBRARIES = $(libtpm2_test_pkcs11) $(libtpm2_test_internal)

## End of test libraries, start tests ##

TESTS = $(check_PROGRAMS)
check_PROGRAMS += \
    test/unit/test_pkcs11 \
    test/unit/test_twist

test_unit_test_pkcs11_CFLAGS   = $(AM_CFLAGS) $(CMOCKA_CFLAGS)
test_unit_test_pkcs11_LDADD    = $(CMOCKA_LIBS) $(libtpm2_test_internal) $(libtpm2_test_pkcs11)
test_unit_test_pkcs11_SOURCES  = test/unit/test_pkcs11.c

test_unit_test_twist_CFLAGS    = $(AM_CFLAGS) $(CMOCKA_CFLAGS)
test_unit_test_twist_LDADD     = $(CMOCKA_LIBS) $(libtpm2_test_internal) $(libtpm2_test_pkcs11)
test_unit_test_twist_SOURCES   = test/unit/test_twist.c

endif
# END UNIT

### Helper Functions ###
define make_parent_dir
    if [ ! -d $(dir $1) ]; then mkdir -p $(dir $1); fi
endef
