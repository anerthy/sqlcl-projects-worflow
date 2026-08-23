# SQLcl Projects Workflow

_By Andrés Mejías at 12/03/2026_

This manual outlines the standardized CI/CD workflow for database development using SQLcl Projects. It defines the end-to-end process for initializing a project, managing database state changes, and generating deployable artifacts for production environments.

---

## 1. Project Creation

To initialize a new project or wrap an existing repository structure:

```cmd
SQL> project init -name pname -makeroot -schemas cicd

```

### Example

```cmd
SQL> project init -name demo_project -schemas demo

-- Initialize Git repository
SQL> host git init --initial-branch=main
SQL> host git add .
SQL> host git commit -m "chore: initializing demo_project git repository"

```

### Connection Setup

To connect to the database, you can use SQLcl CLI or the Oracle SQL Developer Extension for VS Code.

```sql
CONNECT user/password@url

```

```sql
CONNECT -SAVE myconn user@localhost:1521/orcl

```

---

## 2. Database Export & Filtering

Export source definitions from the database to local workspace files in `src/`. This syncs your Git repository with changes made in the Development Database.

```cmd
SQL> project export

```

### Custom Object Exports

You can filter exports by object name, APEX application ID, or schema:

```cmd
-- Export a specific table or APEX application
SQL> project export -o emp
SQL> project export -o dep --schemas HR
SQL> project export -o apex.100

```

### Project Configurations & Exclusions

To exclude unwanted objects (like synonyms or object grants) from being staged or exported:

```cmd
-- Exclude synonyms and grants from staging
SQL> project config set -name stage.excludeObjects -value "ALL.OBJECT_GRANT,ALL.GRANT,ALL.SYNONYM"

```

---

## 3. Staging Changes

Generate Liquibase changelogs and changesets by comparing source files against the target/default branch.

```cmd
SQL> project stage -verbose

```

### Custom Scripts (`add-custom`)

For data manipulation (DML), seed data, or ad-hoc scripts, create a custom staged file:

```cmd
SQL> project stage add-custom -file-name populate-customers.sql

```

This generates an empty file under the active custom directory (`dist/releases/next/_custom/`). Edit this file with your custom SQL:

```sql
INSERT INTO customers (dni, name) VALUES ('123456789', 'Sabrina Carpenter');

```

> **NOTE:** Afterwards, commit the generated/modified files to your branch and open a Pull Request (PR) for review.

### Directory Structure in `dist`

SQLcl Projects utilizes a hierarchical changelog system within the `dist` folder:

```text
dist/
└── releases/
    ├── main/             <-- Promoted production releases
    └── next/             <-- Active development release
        ├── _custom/      <-- Custom DML/DDL scripts
        └── branch-name/  <-- Feature branch changelogs

```

- **Main** ➔ **Release** ➔ **Change**
- For every ticket/feature, perform a `project stage` and commit the changes.
- Multiple **Changes** are bundled into a **Release** (currently named `next`), which is ultimately deployed and merged into **Main**.

---

## 4. Verification & Artifact Generation

Before packaging, verify that all changelogs and changesets are valid:

```cmd
SQL> project verify

```

### Create a Release

Package the current staged changes into a specific version state:

```cmd
SQL> project release -version 1.0.0

```

### Generate Artifact

Create an installable ZIP archive containing the full deployment package (`dist/install.sql`, Liquibase controllers, and changelogs):

```cmd
SQL> project gen-artifact -name hr -version 1.0.0 -format zip -verbose

```

---

## 5. Release Deployment

The promotion flow follows the standard pipeline cycle:
**[ DEV ] ➔ [ Artifact (.zip) ] ➔ [ QA / PROD ]**

To deploy an artifact onto a target database environment:

```cmd
SQL> project deploy -file artifact/demo_project-1.0.0.zip -verbose

```

---

## References

- February2025. (2025, February 13). _About the project command_. Oracle Help Center. https://docs.oracle.com/en/database/oracle/sql-developer-command-line/24.4/sqcug/project-command.html
- Oracle Developers. (2024, December 2). _SQLCL Projects: CI/CD made Easy for APEX_ [Video]. YouTube. https://www.youtube.com/watch?v=EM3_2Dd3LOs
- Oracle Developers. (2024b, December 27). _Proyectos SQLcl: CI/CD Simplificado para APEX_ [Video]. YouTube. https://www.youtube.com/watch?v=FkNRKTuXQpY
- Oracle Developers. (2025, May 30). _Developer Coaching: Effortless Oracle Database Change Management with SQLcl project_ [Video]. YouTube. https://www.youtube.com/watch?v=A4Z2FmNLITM
- Thatjeffsmith. (2025, June 18). _Getting started with Oracle Database CI/CD & SQLcl Projects_. ThatJeffSmith. https://www.thatjeffsmith.com/archive/2025/05/getting-started-with-sqlcl-projects/
