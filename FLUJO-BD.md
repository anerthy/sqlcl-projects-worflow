# Flujo de cambios de base de datos (SQLcl Projects)

> Sección lista para copiar/pegar en el README de un proyecto nuevo.

Todos los comandos se ejecutan dentro de SQLcl (`sql /nolog`), conectado a la base de **Desarrollo**:

```cmd
SQL> CONNECT usuario/clave@host:1521/servicio
```

## Ciclo diario

1. **Rama por ticket**

   ```cmd
   SQL> host git checkout -b feature/TICKET-123
   ```

2. **Bajar los cambios de la base a los archivos** (`src/`)

   ```cmd
   SQL> project export                      -- todo el esquema
   SQL> project export -o emp               -- un solo objeto
   SQL> project export -o apex.100          -- una aplicación APEX
   ```

3. **Generar los changelogs de Liquibase** (compara `src/` contra `main`)

   ```cmd
   SQL> project stage -verbose
   ```

   ¿Necesita DML, datos semilla o un script manual?

   ```cmd
   SQL> project stage add-custom -file-name TICKET-123-datos.sql
   ```

   Se crea vacío en `dist/releases/next/_custom/`; edítelo y escriba su SQL ahí.

4. **Commit y Pull Request**

   ```cmd
   SQL> host git add .
   SQL> host git commit -m "feat(TICKET-123): descripción del cambio"
   SQL> host git push -u origin feature/TICKET-123
   ```

## Cierre de release (después del merge a `main`)

```cmd
SQL> project verify                                              -- valida changelogs
SQL> project release -version 1.0.0                              -- congela `next` como 1.0.0
SQL> project gen-artifact -name miapp -version 1.0.0 -format zip -verbose
```

El ZIP queda en `artifact/`.

## Despliegue en QA / PROD

Conectado a la base **destino**:

```cmd
SQL> project deploy -file artifact/miapp-1.0.0.zip -verbose
```

**[ DEV ] ➔ [ Artefacto .zip ] ➔ [ QA / PROD ]**

## Configuración útil (una sola vez por proyecto)

```cmd
-- Excluir sinónimos y grants del staging
SQL> project config set -name stage.excludeObjects -value "ALL.OBJECT_GRANT,ALL.GRANT,ALL.SYNONYM"
```

## Referencia rápida

| Comando | Para qué |
|---|---|
| `project init -name x -schemas y` | Crear el proyecto |
| `project export` | Base ➔ archivos (`src/`) |
| `project stage` | Archivos ➔ changelogs (`dist/releases/next/`) |
| `project stage add-custom -file-name f.sql` | Script DML/manual |
| `project verify` | Validar changelogs |
| `project release -version 1.0.0` | Congelar el release |
| `project gen-artifact -name x -version 1.0.0 -format zip` | Generar el ZIP |
| `project deploy -file artifact/x-1.0.0.zip` | Desplegar en el destino |
