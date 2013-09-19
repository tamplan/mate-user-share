AC_INIT(mate-user-share, 1.6.0)

AM_INIT_AUTOMAKE([1.9 dist-bzip2 no-dist-gzip check-news])
AM_CONFIG_HEADER(config.h)

# Enable silent build when available (Automake 1.11)
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

AM_SANITY_CHECK
AM_MAINTAINER_MODE

IT_PROG_INTLTOOL([0.35.0])

GNOME_DOC_INIT

AC_C_CONST
AC_ISC_POSIX
AC_PROG_CC
AC_PROG_CPP
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AM_DISABLE_STATIC
AM_PROG_LIBTOOL
AC_PATH_XTRA
AC_PATH_PROG(PKG_CONFIG, pkg-config, no)

AC_TYPE_PID_T
AC_TYPE_SIGNAL
AC_TYPE_SIZE_T
AC_TYPE_UID_T

AC_ARG_WITH([httpd],
	    [AS_HELP_STRING([--with-httpd],[Specify the binary used for the Apache httpd binary (default: httpd)])],
	    [HTTPD=$withval])
if test -z "$HTTPD" ; then
	AC_PATH_PROG([HTTPD], [httpd],,
		     [$PATH:/usr/sbin])
	if test -z "$HTTPD" ; then
		# As used in SUSE
		AC_PATH_PROG([HTTPD], [apache2],,
			     [$PATH:/usr/sbin])
	fi
	if test -z "$HTTPD" ; then
		# As used in Ubuntu
		AC_PATH_PROG([HTTPD], [httpd2],,
			     [$PATH:/usr/sbin])
	fi
fi

AC_SUBST(HTTPD)

BLUETOOTH_PKG="mate-bluetooth-1.0 >= 1.2.0"
AC_ARG_ENABLE(bluetooth, AS_HELP_STRING([--disable-bluetooth],[compile without bluetooth support]),,enable_bluetooth=yes)
if test "x$enable_bluetooth" = "xyes"; then
 PKG_CHECK_MODULES(BLUETOOTH, $BLUETOOTH_PKG)
 AC_DEFINE(HAVE_BLUETOOTH, 1, [Set to true if mate-bluetooth support is available])
else
 BLUETOOTH_PKG=""
 AC_MSG_WARN([Bluetooth support is disabled.])
fi
AM_CONDITIONAL(USE_BLUETOOTH, [test "$enable_bluetooth" = "yes"])

PKG_CHECK_MODULES(USER_SHARE, glib-2.0 >= 2.15.2 gio-2.0 >= 2.26 gdk-x11-2.0 gtk+-2.0 dbus-1 >= 1.1.1 dbus-glib-1 libnotify >= 0.7.0 libcanberra-gtk $DBUS_MODULES $BLUETOOTH_PKG)
AC_SUBST(USER_SHARE_CFLAGS)
AC_SUBST(USER_SHARE_LIBS)

PKG_CHECK_MODULES(USER_SHARE_CONFIG, glib-2.0 >= 2.15.2 gio-2.0 >= 2.25.0 gtk+-2.0 >= 2.12.0 unique-1.0)
AC_SUBST(USER_SHARE_CONFIG_CFLAGS)
AC_SUBST(USER_SHARE_CONFIG_LIBS)
	
AC_CHECK_LIB(socket, socket)

have_socket=no
AC_CHECK_FUNCS(socket, have_socket=yes)
if test $have_socket = no; then
  # socket is not in the default libraries.  See if it's in some other.
  for lib in bsd socket inet; do
    AC_CHECK_LIB($lib, socket, [
	  LIBS="$LIBS -l$lib"
	  have_socket=yes
	  AC_DEFINE(HAVE_SOCKET)
	  break])
  done
fi

AC_PATH_PROG([GLIB_GENMARSHAL],[glib-genmarshal])

dnl 
dnl Enable gsettigns schema macros
dnl

GLIB_GSETTINGS

dnl where to install the mate-user-share binary
if test "x$prefix" = "xNONE"; then
  prefix=$ac_default_prefix
fi
if test "x$exec_prefix" = "xNONE"; then
   exec_prefix=$prefix
fi
installdir="`eval echo $libexecdir`"
AC_SUBST(installdir, $installdir)

dnl ==========================================================================

GETTEXT_PACKAGE=mate-user-share
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED([GETTEXT_PACKAGE],"$GETTEXT_PACKAGE", [The gettext package])

AM_GLIB_GNU_GETTEXT

	
dnl ==========================================================================
dnl Check for SELinux

AC_CHECK_LIB(selinux, is_selinux_enabled, [ SELINUX_LIBS=-lselinux
  AC_DEFINE(HAVE_SELINUX,[1],[Found Selinux]) ])
AC_SUBST(SELINUX_LIBS)

dnl ==========================================================================
dnl Check for the HTTPD modules path



AC_ARG_WITH([modules-path],
	    [AS_HELP_STRING([--with-modules-path],[Path where the httpd modules are located (default: /etc/httpd/modules/)])],
	    [with_modules_path=$withval])

MODULES_PATH=$with_modules_path
AC_SUBST(MODULES_PATH)

dnl===========================================================================
dnl Check for caja for the share bar

AC_ARG_WITH(cajadir,
	    AS_HELP_STRING([--with-cajadir=DIR],[Installation path for Caja extension @<:@auto@:>@]),
	    [ac_with_cajadir=$withval],
	    [ac_with_cajadir=""])

PKG_CHECK_MODULES(EXTENSION,
		   libcaja-extension $BLUETOOTH_PKG)
if test "${ac_with_cajadir}" = ""; then
	ac_with_cajadir=`pkg-config --variable=extensiondir libcaja-extension`
fi
AC_SUBST(CAJADIR)
AC_MSG_NOTICE([installing caja plugin in ${ac_with_cajadir}])
AC_SUBST([CAJADIR],[${ac_with_cajadir}])

dnl ==========================================================================

dnl Turn on the additional warnings last, so -Werror doesn't affect other tests.

AC_ARG_ENABLE(more-warnings,
[  --enable-more-warnings  Maximum compiler warnings],
set_more_warnings="$enableval",[
if test -f $srcdir/CVSVERSION; then
	is_cvs_version=true
	set_more_warnings=yes
else
	set_more_warnings=no
fi
])
AC_MSG_CHECKING(for more warnings, including -Werror)
if test "$GCC" = "yes" -a "$set_more_warnings" != "no"; then
	AC_MSG_RESULT(yes)
	CFLAGS="\
	-Wall \
	-Wchar-subscripts -Wmissing-declarations -Wmissing-prototypes \
	-Wnested-externs -Wpointer-arith \
	-Wcast-align -Wsign-compare \
	-Werror \
	$CFLAGS"

	for option in -Wno-strict-aliasing -Wno-sign-compare; do
		SAVE_CFLAGS="$CFLAGS"
		CFLAGS="$CFLAGS $option"
		AC_MSG_CHECKING([whether gcc understands $option])
		AC_TRY_COMPILE([], [],
			has_option=yes,
			has_option=no,)
		if test $has_option = no; then
			CFLAGS="$SAVE_CFLAGS"
		fi
		AC_MSG_RESULT($has_option)
		unset has_option
		unset SAVE_CFLAGS
	done
	unset option
else
	AC_MSG_RESULT(no)
fi
			       
AC_CONFIG_FILES([
Makefile
data/Makefile
data/icons/Makefile
data/icons/16x16/Makefile
data/icons/22x22/Makefile
data/icons/32x32/Makefile
data/icons/48x48/Makefile
data/mate-user-share-webdav.desktop.in
data/mate-user-share-obexftp.desktop.in
data/mate-user-share-obexpush.desktop.in
po/Makefile.in
src/Makefile
man/Makefile
])

AC_OUTPUT

if test "x$enable_bluetooth" = "xyes"; then
	AC_MSG_NOTICE([** Bluetooth support enabled (ObexPush and ObexFTP)])
else
	AC_MSG_NOTICE([   Bluetooth support disabled])
fi

AC_MSG_NOTICE([mate-user-share was configured with the following options:])
AC_MSG_NOTICE([** httpd location: $HTTP])
AC_MSG_NOTICE([** httpd modules path: $MODULES_PATH])
AC_MSG_NOTICE([** caja extension path: $ac_with_cajadir])

echo "
Do note Apache 2.2 and mod_dnssd 0.6 are required to use mate-user-share.
"