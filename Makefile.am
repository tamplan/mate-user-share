ACLOCAL_AMFLAGS = -I m4 ${ACLOCAL_FLAGS}

SUBDIRS = po data src man help

EXTRA_DIST =			\
	autogen.sh

DISTCHECK_CONFIGURE_FLAGS = --disable-schemas-compile \
	--with-cajadir='$${libdir}/caja/extensions-2.0-distcheck' \
	--enable-compile-warnings=no \
	CFLAGS='-Wno-deprecated-declarations'

# Build ChangeLog from GIT  history
ChangeLog:
	$(AM_V_GEN) if test -d $(top_srcdir)/.git; then \
		GIT_DIR="$(top_srcdir)/.git" git log --stat > $@; \
	fi

dist: ChangeLog

.PHONY: ChangeLog

-include $(top_srcdir)/git.mk
