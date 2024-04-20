noinst_PROGRAMS =	table-postgres

table_postgres_SOURCES =	table_postgres.c dict.c log.c table_stdio.c util.c

AM_CFLAGS =
LDADD =			$(LIBOBJS)

smtpdir =		${prefix}/libexec/smtpd

install-exec-local: $(noinst_PROGRAMS)
	$(MKDIR_P) $(DESTDIR)$(smtpdir)
	$(INSTALL_PROGRAM_ENV) $(INSTALL_PROGRAM) $(noinst_PROGRAMS) $(DESTDIR)$(smtpdir)

README.md: table-postgres.5
	mandoc -Tmarkdown -l table-postgres.5 > README.md
