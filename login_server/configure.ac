dnl
dnl This file is part of Sylverant PSO Server.
dnl
dnl Copyright (C) 2009, 2011 Lawrence Sebald
dnl
dnl This program is free software: you can redistribute it and/or modify
dnl it under the terms of the GNU Affero General Public License version 3
dnl as published by the Free Software Foundation.
dnl
dnl This program is distributed in the hope that it will be useful,
dnl but WITHOUT ANY WARRANTY; without even the implied warranty of
dnl MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
dnl GNU Affero General Public License for more details.
dnl
dnl You should have received a copy of the GNU Affero General Public License
dnl along with this program.  If not, see <http://www.gnu.org/licenses/>.

AC_PREREQ(2.61)
AC_INIT([login_server],[0.1.0])
AC_CONFIG_MACRO_DIR([m4])

AC_CANONICAL_HOST
AC_CANONICAL_TARGET

AM_INIT_AUTOMAKE
AC_PROG_INSTALL
LT_INIT

AC_LANG(C)

AC_CONFIG_SRCDIR([src/dclogin.c])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.
PKG_CHECK_MODULES([libxml2], [libxml-2.0 >= 2.7])
CFLAGS="$CFLAGS $libxml2_CFLAGS"
LIBS="$LIBS $libxml2_LIBS"

MYSQL_CLIENT()

MYSQL_LIBS="`mysql_config --libs`"

AC_SUBST(MYSQL_LIBS)
AC_CHECK_LIB([crypto], [SHA256], , AC_MSG_ERROR([libcrypto with SHA256 support is required!]))
AC_CHECK_LIB([sylverant], [sylverant_read_config], , AC_MSG_ERROR([libsylverant is required!]))
AC_CHECK_LIB([mini18n], [mini18n_get], , AC_MSG_WARN([Internationalization support requires mini18n]))
AC_CHECK_LIB([z], [compress2], , AC_MSG_ERROR([zlib is required!]))

AM_ICONV

# Checks for header files.
AC_CHECK_HEADERS([arpa/inet.h inttypes.h netdb.h netinet/in.h stdlib.h string.h sys/socket.h sys/time.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T
AC_C_BIGENDIAN

# Checks for library functions.
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_CHECK_FUNCS([gethostname gettimeofday inet_ntoa memmove memset select socket strtoul])

ADD_CFLAGS([-Wall])

AC_ARG_ENABLE([ipv6], [AS_HELP_STRING([--disable-ipv6],
               [disable IPv6 support (enabled by default)])],
               [enable_ipv6=$withval],
               [enable_ipv6=yes])

AS_IF([test "x$enable_ipv6" != xno],
      [AC_DEFINE([ENABLE_IPV6], [1], [Define if you want IPv6 support])])

AC_CONFIG_FILES([Makefile]
                [l10n/Makefile])

AC_OUTPUT
