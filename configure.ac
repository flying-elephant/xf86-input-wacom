#  Copyright 2005 Adam Jackson.
#
#  Permission is hereby granted, free of charge, to any person obtaining a
#  copy of this software and associated documentation files (the "Software"),
#  to deal in the Software without restriction, including without limitation
#  on the rights to use, copy, modify, merge, publish, distribute, sub
#  license, and/or sell copies of the Software, and to permit persons to whom
#  the Software is furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice (including the next
#  paragraph) shall be included in all copies or substantial portions of the
#  Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT.  IN NO EVENT SHALL
#  ADAM JACKSON BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
#  IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
#  CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
# Process this file with autoconf to produce a configure script

# Initialize Autoconf
AC_PREREQ([2.60])
# bump version in meson.build
AC_INIT([xf86-input-wacom],
        [1.2.2],
        [https://github.com/linuxwacom/xf86-input-wacom/issues/],
        [xf86-input-wacom])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS([config.h])

# Initialize Automake
AM_INIT_AUTOMAKE([foreign dist-bzip2 no-dist-gzip subdir-objects])
AM_MAINTAINER_MODE([enable])
AC_USE_SYSTEM_EXTENSIONS

AC_PROG_CC_STDC

m4_include([m4/ax_check_compile_flag.m4])

# Initialize libtool
AC_DISABLE_STATIC
AC_PROG_LIBTOOL

# -----------------------------------------------------------------------------
#                      Compiler Flags / Libraries
# -----------------------------------------------------------------------------
m4_define(COMPILER_FLAGS, [
          -Wall
          -Wextra
          -Wno-unused-parameter
          -Wmissing-prototypes
          -Wstrict-prototypes
          -Wlogical-op
          -Wpointer-arith
          -Wuninitialized
          -Winit-self
          -Wstrict-prototypes
          -Wimplicit-fallthrough
          -Wredundant-decls
          -Wincompatible-pointer-types
          -Wformat=2
          -Wformat-overflow=2
          -Wformat-signedness
          -Wformat-truncation=2
          -Wmissing-declarations
          -Wshift-overflow=2
          -Wstrict-overflow=2
          -Wswitch-enum

          -fvisibility=hidden
])
m4_foreach_w([_flag], COMPILER_FLAGS, [
    AX_CHECK_COMPILE_FLAG(_flag, AVAILABLE_FLAGS="$AVAILABLE_FLAGS _flag", [], [-Werror])
])
AM_CFLAGS="$AM_CFLAGS $AVAILABLE_FLAGS"
AM_CFLAGS="$AM_CFLAGS -D_GNU_SOURCE"
AC_SUBST(AM_CFLAGS)

# Initialize X.Org macros 1.8 or later for MAN_SUBSTS set by XORG_MANPAGE_SECTIONS
m4_ifndef([XORG_MACROS_VERSION],
          [m4_fatal([must install xorg-macros 1.8 or later before running autoconf/autogen])])
XORG_MACROS_VERSION([1.8])
XORG_DEFAULT_OPTIONS

# Checks for libraries.
AC_CHECK_LIB([m], [rint])

XPROTOS="xproto xext kbproto inputproto randrproto"

# Obtain compiler/linker options from server and required extensions
PKG_CHECK_MODULES(XORG, [xorg-server >= 1.13.0] $XPROTOS)

# Obtain compiler/linker options for the xsetwacom tool
PKG_CHECK_MODULES(X11, x11 xi xrandr xinerama $XPROTOS)

# Obtain compiler/linker options for libudev used by ISDV4 code
PKG_CHECK_MODULES(UDEV, libudev)

# X Server SDK location is required to install wacom header files
# This location is also relayed in the xorg-wacom.pc file
sdkdir=`$PKG_CONFIG --variable=sdkdir xorg-server`

# Workaround overriding sdkdir to be able to create a tarball when user has no
# write permission in sdkdir. See DISTCHECK_CONFIGURE_FLAGS in Makefile.am
AC_ARG_WITH([sdkdir], [], [sdkdir="$withval"])
AC_SUBST([sdkdir])

DRIVER_NAME=wacom
AC_SUBST([DRIVER_NAME])

# -----------------------------------------------------------------------------
#			Configuration options
# -----------------------------------------------------------------------------
# Define a configure option for code debugging
AC_ARG_ENABLE(debug,
              AS_HELP_STRING([--disable-debug],
                             [Disable debugging (default: enabled)]),
              [DEBUGGING=$enableval],
              [DEBUGGING=yes])

# Define the C preprocessor macro DEBUG in config.h
if test "x$DEBUGGING" = xyes; then
       AC_DEFINE(DEBUG, 1, [Enable debugging code])
fi

# Define a configure option for an alternate input module directory
AC_ARG_WITH(xorg-module-dir,
            AS_HELP_STRING([--with-xorg-module-dir=DIR],
                           [Default xorg module directory [[default=$libdir/xorg/modules]]]),
            [moduledir="$withval"],
            [moduledir="$libdir/xorg/modules"])
inputdir=${moduledir}/input
AC_SUBST(inputdir)

# Define a configure option for an alternate X Server configuration directory
sysconfigdir=`$PKG_CONFIG --variable=sysconfigdir xorg-server`
AC_ARG_WITH([xorg-conf-dir],
	    AS_HELP_STRING([--with-xorg-conf-dir=DIR],
			   [Default xorg.conf.d directory [[default=from $PKG_CONFIG xorg-server]]]),
            [configdir="$withval"],
            [configdir="$sysconfigdir"])
AC_SUBST(configdir)
AM_CONDITIONAL(HAS_XORG_CONF_DIR, [test "x$configdir" != "x"])

AC_ARG_ENABLE(hal-fdi-preprobe-quirk, AS_HELP_STRING([--enable-hal-fdi-preprobe-quirk],
                          [Install HAL workaround for devices with unsupported EV_SW events (default: auto)]),
                          [USE_HAL_FDI_PREPROBE_QUIRK=$enableval],
                          [USE_HAL_FDI_PREPROBE_QUIRK=auto])
if test "x$USE_HAL_FDI_PREPROBE_QUIRK" = xauto ; then
       AC_CHECK_PROG([LSHAL], [lshal], [yes], [no])
       AC_CHECK_PROG([HAL_SET_PROPERTY], [hal-set-property], [yes], [no])
       AS_IF([test "x$LSHAL" = "xyes" -a "x$HAL_SET_PROPERTY" = "xyes"],
             [USE_HAL_FDI_PREPROBE_QUIRK=yes],
             [USE_HAL_FDI_PREPROBE_QUIRK=no])
fi
AC_MSG_CHECKING([if HAL preprobe quirk should be installed])
AC_MSG_RESULT($USE_HAL_FDI_PREPROBE_QUIRK)
AM_CONDITIONAL(USE_HAL_FDI_PREPROBE_QUIRK, [test "x$USE_HAL_FDI_PREPROBE_QUIRK" = xyes])

AC_ARG_ENABLE(fuzz-interface, AS_HELP_STRING([--enable-fuzz-interface],
                          [Enable xsetwacom to take NUL-separated commands from stdin (default: no)]),
                          [FUZZINTERFACE=$enableval],
                          [FUZZINTERFACE=no])
AM_CONDITIONAL(FUZZINTERFACE, [test "x$FUZZINTERFACE" = xyes])

AC_ARG_ENABLE(unit-tests, AS_HELP_STRING([--enable-unit-tests],
                          [Enable unit-tests (default: auto)]),
                          [UNITTESTS=$enableval],
                          [UNITTESTS=auto])

# Check for the availability of dlsym and friends
AC_CHECK_LIB([dl], [dlsym],
    [libdl_available="yes"],
    [libdl_available="no"]
)

# If unittests aren't explicitly enabled, check for required support
if test "x$UNITTESTS" = xauto; then
    UNITTESTS=$libdl_available
fi

if test "x$UNITTESTS" = xyes; then
       if test "x$libdl_available" = xno; then
           AC_MSG_ERROR([dlsym required to build unit tests])
       fi
       AC_DEFINE(UNITTESTS, 1, [Enable unit tests])
fi
AM_CONDITIONAL(UNITTESTS, [test "x$UNITTESTS" = xyes])

AC_ARG_WITH([systemd-unit-dir],
            AS_HELP_STRING([--with-systemd-unit-dir=DIR],
                           [systemd unit directory [[default=$libdir/systemd/system]]]),
            [unitdir="$withval"],
            [unitdir="$libdir/systemd/system"])
SYSTEMD_UNIT_DIR=${unitdir}
AC_SUBST(SYSTEMD_UNIT_DIR)
AM_CONDITIONAL(HAVE_SYSTEMD_UNIT_DIR, [test "x$SYSTEMD_UNIT_DIR" != "xno"])

AC_ARG_WITH(udev-rules-dir,
            AS_HELP_STRING([--with-udev-rules-dir=DIR],
                           [Directory where udev expects its rules files
                           [[default=$libdir/udev/rules.d]]]),
            [udevdir="$withval"],
            [udevdir="$libdir/udev/rules.d"])
UDEV_RULES_DIR=${udevdir}
AC_SUBST(UDEV_RULES_DIR)
AM_CONDITIONAL(HAVE_UDEV_RULES_DIR, [test "x$UDEV_RULES_DIR" != "xno"])


# -----------------------------------------------------------------------------

AC_CONFIG_FILES([Makefile
                 conf/Makefile
                 src/Makefile
                 man/Makefile
                 include/Makefile
                 tools/Makefile
                 test/Makefile
                 xorg-wacom.pc])
AC_OUTPUT
