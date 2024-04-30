noinst_PROGRAMS =	table-postgres

table_postgres_SOURCES =	table_postgres.c dict.c log.c table_stdio.c util.c

LDADD =			$(LIBOBJS)

dist_man5_MANS =	table-postgres.5

EXTRA_DIST =		README.md compat.h config.h.in dict.h log.h \
			table_stdio.h util.h

smtpdir =		${prefix}/libexec/smtpd

install-exec-local: $(noinst_PROGRAMS)
	$(MKDIR_P) $(DESTDIR)$(smtpdir)
	$(INSTALL_PROGRAM_ENV) $(INSTALL_PROGRAM) $(noinst_PROGRAMS) $(DESTDIR)$(smtpdir)

uninstall-local:
	rm $(DESTDIR)$(smtpdir)/$(noinst_PROGRAMS)

regen-readme:
	mandoc -Tmarkdown -l table-postgres.5 > README.md
