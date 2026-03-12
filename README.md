# SQLcl Projects Workflow

_By Andrés Mejías at 12/03/2026_

This manual outlines the standardized CI/CD workflow for database development using SQLcl Projects. It defines the end-to-end process for initializing a project, managing database state changes, and generating deployable artifacts for production environments.

## Project Creation

New projects or existing projects

```cmd
SQL> PROJECT init -name pname -makeroot -schemas cicd
```

```cmd
SQL> project init -name demo_project -schemas demo

SQL> !git init --initial-branch=main
SQL> !git add .
SQL> !git commit -m "chore: initializing pname git repository"
```

### Connection

To connect to database you can use just SQLcl or Oracle SQL Developer Extension for VSCode.

```SQL
connect user/password@url
```

```SQL
CONNECT -SAVE myconn user@localhost:1521/orcl
```

## Database Export

Export source from database to files
Control with filters in config
Sync repo with changes in Development DB

```cmd
SQL> PROJECT export
```

### Sample

Create Liquibase changelogs/sets

```SQL
UPDATE emp SET name = first_name || ' ' || last_name;
```

```sql
ALTER TABLE emp ADD depto NUMBER;
```

```cmd
project export -o emp
project export -o apex.100
```

## State Changes

Create the actual changes to be applied.

```cmd
SQL> project state -verbose
```

For custom scripts, use

```
SQL> project stage add-custom
```

```
project stage add-custom -file-name populate-customers.sql
```

`add-custom` command create a new file empty that we can modify to add our script.

```sql
INSERT INTO customers (dni,name) values ('123456789','Sabrina Carpenter');
```

NOTE: Afterwards, we commit the changes to our branch and then create a Pull Request.

### Directories in dist

Hierarchical changelogs in dist.

Main => Release => Change

We utilize a hierarchical changelog system within the dist folder, following a **Main** > **Release** > **Change** structure.

For every ticket, we create a **project stage** and perform **Git commits**. These are later bundled into a **Release**, which is ultimately merged into the **Main** branch.

Currently, the active release the team is developing is named "**Next**". A single release may consist of multiple **Changes**, and each change contains one or more scripts that modify database objects.

## Artifact Generation

Generate a releaseble set of object

```cmd
SQL> project release -version 1.0
```

Generate an artifact for this set ot installable objects.
The zip file with the changes.

```cmd
SQL> project gen-artifact
```

```
project gen-artifact -name hr -version 1.0 -format zip -verbose
```

Before to create you can test changelogs/sets using:

```
SQL> project verify
```

## Release Deployment

1. Create object on Development env
2. Create artifact
3. Install artifact on Production env

[ DEV ] => [ Artifact ] => [ PROC ]

```cmd
SQL> project deploy -file pname-1.0.zip
```

```
project deploy -file artifact/demo_project-1.0.0.zip
```

## References

February2025. (2025, February 13). About the project command. Oracle Help Center. https://docs.oracle.com/en/database/oracle/sql-developer-command-line/24.4/sqcug/project-command.html

Oracle Developers. (2024, December 2). SQLCL Projects: CI/CD made Easy for APEX [Video]. YouTube. https://www.youtube.com/watch?v=EM3_2Dd3LOs

Oracle Developers. (2024b, December 27). Proyectos SQLcl: CI/CD Simplificado para APEX [Video]. YouTube. https://www.youtube.com/watch?v=FkNRKTuXQpY

Oracle Developers. (2025, May 30). Developer Coaching: Effortless Oracle Database Change Management with SQLcl project [Video]. YouTube. https://www.youtube.com/watch?v=A4Z2FmNLITM

Thatjeffsmith, & Thatjeffsmith. (2025, June 18). Getting started with Oracle Database CI/CD & SQLcl Projects. ThatJeffSmith | Helping You Be More Successful With Oracle Database. https://www.thatjeffsmith.com/archive/2025/05/getting-started-with-sqlcl-projects/
