# PostgreSQL Foreign Data Wrapper (FDW) definition generator

The `generate-foreign-tables` script produces SQL that uses the [`postgres_fdw`][]
extension to define a foreign server/schema combination and populate it with
foreign table definitions created by reading table metadata from the remote
server.

[`postgres_fdw`]: https://www.postgresql.org/docs/current/static/postgres-fdw.html

# Why would I want to use FDWs?

Our use case is to bring together tables from three separate databases into one
read-only _reporting_ database.  The _reporting_ database contains only foreign
tables and so might be considered "virtual".  The upshot of this "virtual
database" is that a single query can use tables from all three actual databases
at once.  Joining across databases is a typical reporting and analysis process
and is made much easier when it can be done transparently all in SQL!

If you're a programmer, think of foreign data wrappers as a way to mixin tables
from other databases.

# Other options for easy creation of foreign tables

Starting with version 9.5, PostgreSQL added support for the [`IMPORT FOREIGN
SCHEMA` command][].  This command connects to the foreign server and creates a
local foreign-table definition for you.  That's really nice!  You may still
want to maintain your own definitions externally to have more control over
them, but `IMPORT FOREIGN SCHEMA` is probably enough if you're using 9.5 or
newer.

[`IMPORT FOREIGN SCHEMA` command]: https://www.postgresql.org/docs/current/static/sql-importforeignschema.html

# Usage

```
generate-foreign-tables --local-schema [local name] --remote-schema [target schema] --tables [desired tables]

	Connects to a Pg database and generates SQL to define each table using a
	foreign data wrapper (via the postgres_fdw extension).

	Use PGHOST, PGDATABASE, and PGUSER to control connection information.  If a
	password is necessary, use a ~/.pgpass file.

	Please review the output before using, in particular the SERVER, USER MAPPING,
	and GRANT definitions, to make sure they are suitable.

	The following options are required:

	--local-schema STR    New database schema in which to define the
	                      foreign tables, used for the foreign server
	                      definition as well
	--remote-schema STR   Existing database schema containing the source
	                      tables for which to create foreign table
	                      definitions

	Other options are:  

	--server-host STR     Foreign server hostname, if it should be
	                      different than the host we connect to to read
	                      the schema
	--tables STR...       List of tables for which to generate
	                      definitions; separate multiple tables by commas
	                      or specify the option multiple times; if not
	                      specified all tables in the schema are dumped
	--help                print usage message and exit
```

# Requirements

This tool is written in Perl and requires a few modules from CPAN.  You can
install them with [cpanm][] like so:

    cpanm --installdeps .   # from inside this git repo

If you don't have the `postgres_fdw` extension installed, first do that as a Pg
superuser by running in your database:

```sql
CREATE EXTENSION IF NOT EXISTS postgres_fdw;
```

You only need to do this on the "local" database, not any of the "foreign"
databases.

We currently use this tool with Pg 9.4.  Your mileage may vary if you're using
a different version, but give a shout if something breaks.

[cpanm]: https://metacpan.org/pod/cpanm

# Copyright

Written by Thomas Sibley <trsibley@uw.edu>.  Copyright 2016– by the University
of Washington.

This repository is free software; you can redistribute it and/or modify it
under the same terms as Perl itself.
