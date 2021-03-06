# $Id: configure.ac,v 1.31 2008/01/25 18:05:07 d3p181 Exp $                                            -*- Autoconf -*-
# Copyright (C) 2008 Battelle Memorial Institute
# Process this file with autoconf to produce a configure script.
# This file is distributed under the same terms as GridLAB-D.

# Parse version info from core/version.h into ./version.sh
esyscmd([sh -c 'cat core/version.h | tr -d "\r" | while read DEFINE NAME VALUE; do [ "$DEFINE" != "#define" ] || case $NAME in REV_*|BRANCH) echo ${NAME#REV_}=$VALUE;; esac; done > version.sh; echo VERSION=\"\$MAJOR.\$MINOR.\$PATCH\" >> version.sh'])
# Define M4 macros for version info
define([pkgversion], esyscmd([sh -c ". ./version.sh; echo \$VERSION | tr -d '\n'"]))dnl
define([pkgbranch], esyscmd([sh -c ". ./version.sh; echo \$BRANCH | tr -d '\n'"]))dnl
define([svnversion], esyscmd([sh -c "svnversion | sed 's/^exported$/0/;s/[^A-Za-z0-9]/+/g' | tr -d '\n'"]))dnl

AC_PREREQ(2.59)
AC_INIT([Gridlab-D], [pkgversion.svnversion], [David Chassin <david.chassin@pnl.gov>], gridlabd)
AC_CONFIG_SRCDIR([core/aggregate.c])
# Older versions of automake may require the that following macro be replaced
# with AM_CONFIG_HEADER([core/config.h]).
AC_CONFIG_HEADER([core/config.h])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([foreign])
AC_DISABLE_STATIC

AC_PREFIX_DEFAULT([/usr])
AC_CANONICAL_HOST

# Export version info as autoconf variables
AC_SUBST(PACKAGE_BASEVERSION, [pkgversion])
AC_SUBST(PACKAGE_BRANCH, [pkgbranch])
AC_SUBST(PACKAGE_REVISION, [svnversion])

if [[ -z "$docdir" ]]; then
	if [[ -z "$datarootdir" ]]; then
		AC_SUBST([datarootdir], [\${prefix}/share])
	fi
	AC_SUBST([docdir], [\${datarootdir}/doc/\${PACKAGE_TARNAME}])
fi

AC_ARG_ENABLE([debug],
				  AS_HELP_STRING([--enable-debug],
				  					  [turn on debugging @<:@default=yes@:>@]),
				  [case "${enableval}" in
				  	 yes) debug=true ;;
				  	 no) debug=false ;;
				  	 *) AC_MSG_ERROR([bad value ${enableval} for --enable-debug]) ;;
				  	esac], [debug=true])

AC_ARG_ENABLE([optimization],
				  AS_HELP_STRING([--enable-optimization@<:@=LEVEL@:>@],
				  					  [turn on optimization @<:@default=0@:>@]),
				  [case "${enableval}" in
				  	 0|1|2|3|s) optimization=${enableval} ;;
				  	 yes) optimization=2 ;;
				  	 no) optimization=0 ;;
				  	 *) AC_MSG_ERROR([bad value ${enableval} for --enable-optimization]) ;;
				  	esac], [optimization=0])

GLFLAGS="$GLFLAGS -I`pwd`/core"
if [[ "$debug" = "true" ]]; then
	GLFLAGS="$GLFLAGS -g"
fi
if [[ "$optimization" ]]; then
	GLFLAGS="$GLFLAGS -O$optimization"
fi

GLRT_LDFLAGS="-export-all-symbols"

case "$host_os" in
darwin*)
     export LIBTOOL=`which glibtool`
     export LIBTOOLIZE=`which glibtoolize`
     GLRT_LDFLAGS="-bundle"
     LIBEXT=.so
	 march=""
     ;;
cygwin*)
     AC_LIBTOOL_WIN32_DLL
     LIBEXT=.a
	 march=""
     ;;
mingw*)
     AC_LIBTOOL_WIN32_DLL
     LIBEXT=.dll
	 march=""
     ;;
*)
     LIBEXT=.so
	 case "$host_cpu" in
		i386) AC_MSG_FAILURE([Sorry, i386 is not a supported architecture]);;
		i486|i586|i686) march="-march=$host_cpu";;
		*) march="";;
	 esac
     ;;
esac

# Putting -ansi in CPPFLAGS causes errors that prevent building.
#CPPFLAGS="$CPPFLAGS $GLFLAGS -ansi"
CPPFLAGS="$CPPFLAGS $GLFLAGS $march"
CXXFLAGS="$CXXFLAGS $GLFLAGS $march"
CFLAGS="$CFLAGS $GLFLAGS $march"

AC_DEFINE_UNQUOTED([REALTIME_LDFLAGS], ["$GLRT_LDFLAGS"], [Realtime linker flags])
AC_SUBST(LIBEXT)
AC_DEFINE_UNQUOTED([DLEXT], ["$LIBEXT"], [Dynamic library extension])

# Checks for programs.
AC_PROG_CXX
AC_PROG_CC
AC_PROG_MAKE_SET
AC_PROG_LN_S
#AC_PROG_MKDIR_P
AC_PROG_INSTALL
AC_PROG_LIBTOOL

if `$LD --version | grep -q '^GNU ld'`; then
	LDFLAGS="$LDFLAGS -Wl,--no-undefined"
	export LDFLAGS
fi

unset XERCES_VERSION
AX_LIB_XERCES([2.7])
AC_SUBST([HAVE_XERCES])

DX_INIT_DOXYGEN(gridlabd, [doxygen/gridlabd.conf])
DX_HTML_FEATURE([ON])
DX_CHM_FEATURE([OFF])
DX_CHI_FEATURE([OFF])
DX_MAN_FEATURE([OFF])
DX_RTF_FEATURE([OFF])
DX_XML_FEATURE([OFF])
DX_PDF_FEATURE([OFF])
DX_PS_FEATURE([OFF])

GLD_CHECK_BUILTIN(
	[__sync_bool_compare_and_swap],
	[],
	[volatile unsigned int value = 0;
	 __sync_bool_compare_and_swap(&value, value, 1);],
	[HAVE_SYNC_BOOL_COMPARE_AND_SWAP=yes],
	[HAVE_SYNC_BOOL_COMPARE_AND_SWAP=no])
GLD_CHECK_BUILTIN(
	[__sync_add_and_fetch],
	[],
	[volatile unsigned int value = 0;
	 __sync_add_and_fetch(&value, 1);],
	[HAVE_SYNC_ADD_AND_FETCH=yes],
	[HAVE_SYNC_ADD_AND_FETCH=no])

# Check for CppUnit
AM_PATH_CPPUNIT([1.12], [HAVE_CPPUNIT=yes], [HAVE_CPPUNIT=no])
if test x"$HAVE_CPPUNIT" = xyes; then
	AC_MSG_CHECKING([if compiler can link libcppunit])
	AC_LANG_PUSH([C++])
	SAVED_CXXFLAGS="$CFLAGS"
	SAVED_LIBS="$LIBS"
	CFLAGS="$CFLAGS $CPPUNIT_CFLAGS"
	LIBS="$LIBS $CPPUNIT_LIBS"
	AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <cppunit/extensions/TestFactoryRegistry.h>],
		[CppUnit::TestFactoryRegistry::getRegistry();])],,
		[HAVE_CPPUNIT=no])
	AC_LANG_POP([C++])
	AC_MSG_RESULT([$HAVE_CPPUNIT])
	if test x"$HAVE_CPPUNIT" != xyes; then
		CFLAGS="$SAVED_CFLAGS"
		LIBS="$SAVED_LIBS"
	fi
fi
test x"$HAVE_CPPUNIT" = xyes || CPPFLAGS="$CPPFLAGS -D_NO_CPPUNIT"


# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([malloc.h memory.h stdlib.h string.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_HEADER_STDBOOL
AC_C_CONST
AC_C_INLINE
AC_STRUCT_TM

# Checks for library functions.
AC_FUNC_MALLOC
AC_FUNC_MKTIME
AC_FUNC_REALLOC
AC_FUNC_STRFTIME
AC_FUNC_VPRINTF
AC_CHECK_FUNCS([isfinite atexit getcwd memset pow putenv sqrt strchr strerror strrchr tzset])
ACX_PTHREAD
GLD_GET_NPROCS
#GCC_CT_FEATURES

# DO NOT EDIT THE FOLLOWING LINE
AC_CONFIG_FILES([./Makefile \
	core/Makefile \
	core/rt/Makefile \
	assert/Makefile \
	climate/Makefile \
	climate/tmy/Makefile \
	commercial/Makefile \
	network/Makefile \
	powerflow/Makefile \
	residential/Makefile \
	reliability/Makefile \
	generators/Makefile\
	plc/Makefile \
	market/Makefile \
	tape_file/Makefile \
	tape_plot/Makefile \
	tape/Makefile \
	third_party/CBLAS/Makefile \
	third_party/superLU_MT/Makefile \
	gridlabd.spec \
	])

AM_CONDITIONAL([MISSING_XERCES], [test x$HAVE_XERCES != xyes])
AC_SUBST(CPPFLAGS)
AC_OUTPUT

BINDIR=`eval echo $(eval echo $bindir)`
LIBDIR=`eval echo $(eval echo $libdir)`
DOCDIR=`eval echo $(eval echo $docdir)`
test -n "$DX_DOXYGEN" && HAVE_DOXYGEN="$DX_DOXYGEN" || HAVE_DOXYGEN=no
if test x"$HAVE_XERCES" = xyes; then
	CONFIG_RESULT=OK
	test -z "$XERCES_VERSION" || expr "$XERCES_VERSION" \< 3 >/dev/null || AC_MSG_WARN([libxerces-c version may not be suitable for building $PACKAGE: $XERCES_VERSION])
else
	CONFIG_RESULT=failed
	HAVE_XERCES="$HAVE_XERCES (required)"
	AC_MSG_WARN([missing libxerces-c; build will fail until dependency is met])
fi

AC_MSG_RESULT([
-----------------------------------------------------------------------

  $PACKAGE $VERSION: Automatic configuration $CONFIG_RESULT.

  Dependencies:

    Doxygen: .................... $HAVE_DOXYGEN
    libcppunit: ................. $HAVE_CPPUNIT
    libxerces-c: ................ $HAVE_XERCES

  Atomic Builtins:

    __sync_bool_compare_and_swap: $HAVE_SYNC_BOOL_COMPARE_AND_SWAP
    __sync_add_and_fetch: ....... $HAVE_SYNC_ADD_AND_FETCH

  Installation paths:

    Scripts: .................... $BINDIR
    Libraries: .................. $LIBDIR/$PACKAGE
    Documentation: .............. $DOCDIR

  Building:

    type 'make' to compile $PACKAGE.
    type 'make install' to install $PACKAGE.
    type 'make install DESTDIR=ROOT' to install $PACKAGE in ROOT.

-----------------------------------------------------------------------
])
