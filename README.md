TABLE\_POSTGRESQL(5) - File Formats Manual

# NAME

**table\_postgresql** - format description for smtpd PostgreSQL tables

# DESCRIPTION

This manual page documents the file format of PostgreSQL tables used
by the
smtpd(8)
mail daemon.

The format described here applies to tables as defined in
smtpd.conf(5).

# POSTGRESQL TABLE

A postgresql table allows the storing of usernames, passwords, aliases, and
domains in a format that is shareable across various machines that support
postgres(1).

The table is used by
smtpd(8)
when authenticating a user, when user information such as user-id and/or
home directory is required for a delivery, when a domain lookup may be required,
and/or when looking for an alias.

A PostgreSQL table consists of one or more
postgresql(1)
databases with one or more tables.

If the table is used for authentication, the password should be
encrypted using the
crypt(3)
function.
Such passwords can be generated using the
encrypt(1)
utility or
smtpctl(8)
encrypt command.

# POSTGRESQL TABLE CONFIG FILE

The following configuration options are available:

**conninfo**
**host**=*'host'*
**user**=*'user'*
**password**=*'password'*
**dbname**=*'dbname'*

> Connection info needed to connect to the PostgreSQL database.
> For example:

> > conninfo host='db.example.com' user='maildba' password='...' dbname='opensmtpdb'

**query\_alias**
*SQL statement*

> This is used to provide a query to look up aliases.
> The question mark is replaced with the appropriate data.
> For alias it is the left hand side of the SMTP address.
> This expects one VARCHAR to be returned with the user name the alias
> resolves to.

**query\_credentials**
*SQL statement*

> This is used to provide a query for looking up user credentials.
> The question mark is replaced with the appropriate data.
> For credentials it is the left hand side of the SMTP address.
> The query expects that there are two VARCHARS returned, one with a user
> name and one with a password in
> crypt(3)
> format.

**query\_domain**
*SQL statement*

> This is used to provide a query for looking up a domain.
> The question mark is replaced with the appropriate data.
> For the domain it would be the right hand side of the SMTP address.
> This expects one VARCHAR to be returned with a matching domain name.

**query\_mailaddrmap**
*SQL statement*

> This is used to provide a query to look up senders.
> The question mark is replaced with the appropriate data.
> This expects one VARCHAR to be returned with the address the sender
> is allowed to send mails from.

A generic SQL statement would be something like:

	query_ SELECT value FROM table WHERE key=$1;

# FILES

*/etc/mail/postgres.conf*

> Default
> table-postgresql(5)
> configuration file.

# EXAMPLES

## GENERIC EXAMPLE

Example based on the OpenSMTPD FAQ: Building a Mail Server
The filtering part is excluded in this example.

The configuration below is for a medium-size mail server which handles
multiple domains with multiple virtual users and is based on several
assumptions.
One is that a single system user named vmail is used for all virtual users.
This user needs to be created:

	# useradd -g =uid -c "Virtual Mail" -d /var/vmail -s /sbin/nologin vmail
	# mkdir /var/vmail
	# chown vmail:vmail /var/vmail

PostgreSQL schema:

	CREATE TABLE domains (
	  id SERIAL,
	  domain VARCHAR(255) NOT NULL DEFAULT ''
	);
	CREATE TABLE virtuals (
	    id SERIAL,
	    email VARCHAR(255) NOT NULL DEFAULT '',
	    destination VARCHAR(255) NOT NULL DEFAULT ''
	);
	CREATE TABLE credentials (
	    id SERIAL,
	    email VARCHAR(255) NOT NULL DEFAULT '',
	    password VARCHAR(255) NOT NULL DEFAULT ''
	);

That can be populated as follows:

	INSERT INTO domains VALUES (1, "example.com");
	INSERT INTO domains VALUES (2, "example.net");
	INSERT INTO domains VALUES (3, "example.org");
	
	INSERT INTO virtuals VALUES (1, "abuse@example.com", "bob@example.com");
	INSERT INTO virtuals VALUES (2, "postmaster@example.com", "bob@example.com");
	INSERT INTO virtuals VALUES (3, "webmaster@example.com", "bob@example.com");
	INSERT INTO virtuals VALUES (4, "bob@example.com", "vmail");
	INSERT INTO virtuals VALUES (5, "abuse@example.net", "alice@example.net");
	INSERT INTO virtuals VALUES (6, "postmaster@example.net", "alice@example.net");
	INSERT INTO virtuals VALUES (7, "webmaster@example.net", "alice@example.net");
	INSERT INTO virtuals VALUES (8, "alice@example.net", "vmail");
	
	INSERT INTO credentials VALUES (1, "bob@example.com", "$2b$08$ANGFKBL.BnDLL0bUl7I6aumTCLRJSQluSQLuueWRG.xceworWrUIu");
	INSERT INTO credentials VALUES (2, "alice@example.net", "$2b$08$AkHdB37kaj2NEoTcISHSYOCEBA5vyW1RcD8H1HG.XX0P/G1KIYwii");

*/etc/mail/postgresql.conf*

	conninfo host='db.example.com' user='maildba' password='OpenSMTPDRules!' dbname='opensmtpdb'
	query_alias SELECT destination FROM virtuals WHERE email=$1;
	query_credentials SELECT email, password FROM credentials WHERE email=$1;
	query_domain SELECT domain FROM domains WHERE domain=$1;

*/etc/mail/smtpd.conf*

	table domains postgres:/etc/mail/postgres.conf
	table virtuals postgres:/etc/mail/postgres.conf
	table credentials postgres:/etc/mail/postgres.conf
	listen on egress port 25 tls pki mail.example.com
	listen on egress port 587 tls-require pki mail.example.com auth <credentials>
	accept from any for domain <domains> virtual <virtuals> deliver to mbox

## MOVING FROM POSTFIX (& POSTFIXADMIN)

*/etc/mail/postgres.conf*

	conninfo host='db.example.com' user='postfix' password='...' dbname='postfix'
	query_alias SELECT destination FROM alias WHERE email=$1;
	query_credentials SELECT username, password FROM mailbox WHERE username=$1;
	query_domain SELECT domain FROM domain WHERE domain=$1;

The rest of the config remains the same.

# TODO

Documenting the following query options:

	**query_netaddr**
	**query_userinfo**
	**query_source**
	**query_mailaddr**
	**query_addrname**

# SEE ALSO

encrypt(1),
crypt(3),
smtpd.conf(5),
smtpctl(8),
smtpd(8)

# HISTORY

The first version of
**table\_postgresql**
was written in 2016.
It was converted to the stdio table protocol in 2024.

# AUTHORS

**table\_postgresql**
was initially written by
Gilles Chehade &lt;[gilles@poolp.org](mailto:gilles@poolp.org)&gt;.
The conversion to the stdio table protocol was done by
Omar Polo &lt;[op@openbsd.org](mailto:op@openbsd.org)&gt;.

Nixpkgs - April 21, 2024
