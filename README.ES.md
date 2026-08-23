# Flujo de trabajo con SQLcl Projects

_Por Andrés Mejías el 23/08/2026_

Este manual describe el flujo de trabajo estandarizado de CI/CD para el desarrollo de bases de datos usando SQLcl Projects. Define el proceso de principio a fin para inicializar un proyecto, gestionar los cambios de estado de la base de datos y generar artefactos desplegables para ambientes de producción.

---

## 1. Creación del proyecto

Para inicializar un proyecto nuevo o envolver la estructura de un repositorio existente:

```cmd
SQL> project init -name pname -makeroot -schemas cicd
```

### Ejemplo

```cmd
SQL> project init -name demo_project -schemas demo

-- Inicializar el repositorio Git
SQL> host git init --initial-branch=main
SQL> host git add .
SQL> host git commit -m "chore: initializing demo_project git repository"
```

### Configuración de la conexión

Para conectarse a la base de datos puede usar la CLI de SQLcl o la extensión Oracle SQL Developer para VS Code.

```sql
CONNECT user/password@url
```

```sql
CONNECT -SAVE myconn user@localhost:1521/orcl
```

---

## 2. Exportación y filtrado de la base de datos

Exporte las definiciones de los objetos desde la base de datos hacia los archivos locales del workspace en `src/`. Esto sincroniza su repositorio Git con los cambios hechos en la base de datos de Desarrollo.

```cmd
SQL> project export
```

### Exportaciones de objetos personalizadas

Puede filtrar las exportaciones por nombre de objeto, ID de aplicación APEX o esquema:

```cmd
-- Exportar una tabla específica o una aplicación APEX
SQL> project export -o emp
SQL> project export -o dep --schemas HR
SQL> project export -o apex.100

```

### Configuraciones y exclusiones del proyecto

Para excluir objetos no deseados (como sinónimos o grants sobre objetos) del staging o de la exportación:

```cmd
-- Excluir sinónimos y grants del staging
SQL> project config set -name stage.excludeObjects -value "ALL.OBJECT_GRANT,ALL.GRANT,ALL.SYNONYM"
```

---

## 3. Staging de cambios

Genere los changelogs y changesets de Liquibase comparando los archivos fuente contra la rama destino/predeterminada.

```cmd
SQL> project stage -verbose
```

### Scripts personalizados (`add-custom`)

Para manipulación de datos (DML), datos semilla o scripts ad-hoc, cree un archivo personalizado en el staging:

```cmd
SQL> project stage add-custom -file-name populate-customers.sql
```

Esto genera un archivo vacío dentro del directorio custom activo (`dist/releases/next/_custom/`). Edite ese archivo con su SQL personalizado:

```sql
INSERT INTO customers (dni, name) VALUES ('123456789', 'Sabrina Carpenter');
```

> **NOTA:** Luego, haga commit de los archivos generados/modificados en su rama y abra un Pull Request (PR) para revisión.

### Estructura de directorios en `dist`

SQLcl Projects utiliza un sistema jerárquico de changelogs dentro de la carpeta `dist`:

```text
dist/
└── releases/
    ├── main/             <-- Releases promovidos a producción
    └── next/             <-- Release de desarrollo activo
        ├── _custom/      <-- Scripts DML/DDL personalizados
        └── branch-name/  <-- Changelogs de la rama de la funcionalidad

```

- **Main** ➔ **Release** ➔ **Change**
- Por cada ticket/funcionalidad, ejecute un `project stage` y haga commit de los cambios.
- Varios **Changes** se agrupan en un **Release** (actualmente llamado `next`), que finalmente se despliega y se fusiona en **Main**.

---

## 4. Verificación y generación del artefacto

Antes de empaquetar, verifique que todos los changelogs y changesets sean válidos:

```cmd
SQL> project verify
```

### Crear un release

Empaquete los cambios preparados actualmente en un estado de versión específico:

```cmd
SQL> project release -version 1.0.0
```

### Generar el artefacto

Cree un archivo ZIP instalable que contiene el paquete de despliegue completo (`dist/install.sql`, controladores de Liquibase y changelogs):

```cmd
SQL> project gen-artifact -name hr -version 1.0.0 -format zip -verbose
```

---

## 5. Despliegue del release

El flujo de promoción sigue el ciclo estándar del pipeline:
**[ DEV ] ➔ [ Artefacto (.zip) ] ➔ [ QA / PROD ]**

Para desplegar un artefacto en una base de datos destino:

```cmd
SQL> project deploy -file artifact/demo_project-1.0.0.zip -verbose
```

---

## Referencias

- February2025. (2025, 13 de febrero). _About the project command_. Oracle Help Center. https://docs.oracle.com/en/database/oracle/sql-developer-command-line/24.4/sqcug/project-command.html
- Oracle Developers. (2024, 2 de diciembre). _SQLCL Projects: CI/CD made Easy for APEX_ [Video]. YouTube. https://www.youtube.com/watch?v=EM3_2Dd3LOs
- Oracle Developers. (2024b, 27 de diciembre). _Proyectos SQLcl: CI/CD Simplificado para APEX_ [Video]. YouTube. https://www.youtube.com/watch?v=FkNRKTuXQpY
- Oracle Developers. (2025, 30 de mayo). _Developer Coaching: Effortless Oracle Database Change Management with SQLcl project_ [Video]. YouTube. https://www.youtube.com/watch?v=A4Z2FmNLITM
- Thatjeffsmith. (2025, 18 de junio). _Getting started with Oracle Database CI/CD & SQLcl Projects_. ThatJeffSmith. https://www.thatjeffsmith.com/archive/2025/05/getting-started-with-sqlcl-projects/
