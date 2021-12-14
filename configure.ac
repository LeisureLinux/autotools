#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ(2.57)
AC_INIT(helloworld, 1.12, hello@qq.com)
AM_INIT_AUTOMAKE(helloworld, 1.1.2)
AC_CONFIG_SRCDIR([src/world.c])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([stdlib.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
# AC_FUNC_MALLOC
AC_CHECK_FUNCS([sqrt])
AC_OUTPUT(Makefile doc/Makefile man/Makefile script/Makefile src/Makefile)
