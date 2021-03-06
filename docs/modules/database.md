# OpenSIPS CLI - Database module

Module used to manipulate the database used by OpenSIPS.

## Commands

This module exports the following commands:
* `create` - creates a new database. Can receive an optional parameter,
specifying the name of the database to be created. This command also deploys
the standard OpenSIPS table, as well as other standard tables
* `drop` - drops a database. Can receive an optional parameter, specifying
which database to delete.
* `add` - adds a new module's tables in an existing database. Receives as
parameter the name of the module, as specified in the OpenSIPS scripts
hierarchy.
* `migrate` - copy and convert an OpenSIPS 2.4 database into its OpenSIPS
3.0 equivalent

## Configuration

The parameters for this tool can be provisioned in two forms:

*  via a declaration in the configuration file
*  typed in when prompted at execution

Specifying a parameter in the configuration file may simplify the user
interaction with the console (less prompts).

The following parameters are allowed in the config file:

* `database_path` - the directory to the OpenSIPS DB scripts, usually the
`scripts/` directory in the OpenSIPS source tree, or `/usr/share/opensips/`
* `database_url` - the connection string to the database.
The URL combines schema, username, password, host and port.
Example: `mysql://user:password@host`
* `template_url` - the connection string to the database in template mode.
The URL will connect to a given database and select a template to execute
the given task. Only database products supporting a role concept will
evaluate this config option (e.g. PostgreSQL).
Example: `postgres://user:password@host:5432`
* `database_name` - the name of the database. Modules may be
created, dropped or added to this database.
* `database_modules` - a space-separated list of the module names.
If processed with the `create` command, the corresponding tables will be
deployed.  Default modules: `acc alias_db auth_db avpops clusterer dialog
dialplan dispatcher domain drouting group load_balancer msilo permissions
rtpproxy rtpengine speeddial tls_mgm usrloc`
* `database_force_drop` - indicates whether the `drop` command will drop the
database without user interaction.

## Examples

Consider the following configuration file:

```
[default]
database_url: mysql://root@localhost
database_name: opensips
database_modules: dialog usrloc

# optional DB override instance, invoked using `opensips-cli -i postgres ...`
[postgres]
database_url: postgres://opensips@localhost:5432
template_url: postgres://postgres@localhost:5432
database_name: opensips
database_modules: dialog usrloc
```

The following command will create the `opensips` table, containing only the
`version`, `dialog` and `location` tables (according to the `database_modules`
parameter):

```
opensips-cli -x database create
```

If we want to add a new module, let's say `rtpproxy`, we have to run:

```
opensips-cli -x database add rtpproxy
```
The command above will create the `rtpproxy_sockets` table.

A drop command will prompt the user whether he really wants to drop the
database or not:

```
$ opensips-cli -x database drop
Do you really want to drop the 'opensips' database [Y/n] (Default is n): n
```

But setting the `database_force_drop` parameter will drop it without asking:
```
opensips-cli -o database_force_drop=true -x database drop
```

## Dependencies

* [sqlalchemy and sqlalchemy_utils](https://www.sqlalchemy.org/) - used to
abstract the SQL database regardless of the backend used

## Limitations

This module can only manipulate database backends that are supported by the
[SQLAlchemy](https://www.sqlalchemy.org/) project, such as  SQLite,
Postgresql, MySQL, Oracle, MS-SQL.
