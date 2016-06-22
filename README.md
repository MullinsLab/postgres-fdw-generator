# PostgreSQL Foreign Data Wrapper (FDW) definition generator

The `generate-foreign-tables` script produces SQL that uses the [`postgres_fdw`][]
extension to define a foreign server/schema combination and populate it with
foreign table definitons matching a set of [DBIx::Class::Schema][] models.

There's no particular reason it couldn't use `pg_catalog` as the table metadata
source, but our [DBIx::Class::Schema][] models were handier.  Consider this a
proof-of-concept for something more general!

[DBIx::Class::Schema]: https://metacpan.org/pod/DBIx::Class::Schema
[`postgres_fdw`]: https://www.postgresql.org/docs/current/static/postgres-fdw.html

# Why would I want to use FDWs?

Our use case is to bring together tables from three separate databases into one
read-only _reporting_ database.  The _reporting_ database contains only foreign
tables, and so might be considered "virtual".  The upshot of this "virtual
database" is that a single query can use tables from all three actual databases
at once.  Joining across databases is a typical reporting and analysis process
and is made much easier when it can be done transparently all in SQL!

If you're a programmer, think of foreign data wrappers as a way to mixin tables
from other databases.

# Usage

```
generate-foreign-tables --db-schema [target schema] --perl-schema [DBIx::Class::Schema subclass] --models [Result classes]
	                  
	Connects to a Pg database via a DBIx::Class::Schema subclass and generates SQL
	to define each table using a foreign data wrapper (via the postgres_fdw
	extension).       
	                  
	Use PGHOST, PGDATABASE, and PGUSER to control connection information.  If a
	password is necessary, use a ~/.pgpass file.
	                  
	Please review the output before using, in particular the SERVER, USER MAPPING,
	and GRANT definitions, to make sure they are suitable.
	                  
	The following options are required:
	                  
	--server-name STR   Name to use for the foreign server and of the
	                    database schema for the foreign table definitions
	--perl-schema STR   Perl package name of your DBIx::Class::Schema
	                    subclass; used to connect to your source database
	                  
	Other options are:
	                  
	--db-schema STR     Database schema of source tables
	--server-host STR   Foreign server hostname, if it should be
	                    different than the host we connect to to read the
	                    schema
	--models STR...     List of abbreviated model/result classes for
	                    which to generate definitions; separate multiple
	                    models by commas or specify the option multiple
	                    times; if not specified all models are dumped
	-I STR...           Adds additional paths to the front of @INC,
	                    similar to perl's own -I option
	--help              print usage message and exit
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
