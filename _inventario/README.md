# Inventarios base

Generados por script (`inv-gen.js`, un script Node que no se versiona: vive en el scratchpad de la sesión) leyendo el contenido **de los commits** exportado con `git archive`, no el árbol de trabajo. Primera generación: 2026-10-06 sobre `sofia-apis` `00991bc` y `sofia-web-spa` `31af681`. Regeneración del 2026-10-13 sobre `43f5529` y `eeffbb6`. **Regenerados el 2026-11-20 sobre `sofia-apis` `0678e40` y `sofia-web-spa` `37048ea`** (ambos repositorios limpios al generarlos). Son la lista de control contra la que se validan las tablas de cobertura de cada documento.

El script se calibró antes de usarlo: se ejecutó sobre `43f5529`/`eeffbb6` y reprodujo las listas anteriores salvo en las rutas de archivo con mayúsculas distintas (ver «Rutas de archivo»).

| Archivo | Contenido | Elementos |
|---|---|---|
| `endpoints.csv` | Controlador, método HTTP, ruta, permiso (ver «Columna `permiso`»), política de límite de peticiones, acción C# y `archivo:línea` | 249 |
| `permisos.csv` | Pares módulo × acción usados en `[HasPermission]` y `[HasPermissionOrPlataforma]` y cuántos endpoints los exigen | 105 (27 módulos) |
| `scripts-dbup.csv` | Scripts DbUp y su número de líneas | 29 |
| `tablas.csv` | Tablas con el script y la línea de su `CREATE TABLE` | 51 |
| `codigos-error.csv` | Códigos de error literales distintos, con sus tipos y número de ocurrencias | 357 |
| `codigos-error-ocurrencias.csv` | Cada ocurrencia de `Error.<Tipo>("codigo", …)` con `archivo:línea` | 733 |
| `codigos-error-no-literales.csv` | Llamadas a `Error.<Tipo>(…)` cuyo código es una constante o variable (hay que resolverlas a mano) | 14 |
| `validadores.csv` | Validadores FluentValidation; su código de error es dinámico: `{Request}.{Propiedad}` | 88 |
| `commands-queries.csv` | Commands y queries de `SOFIA.Application` (118 commands y 92 queries) | 210 |
| `entidades.csv`, `enums.csv` | Entidades y enums de `SOFIA.Domain` | 51 y 12 |
| `rutas-spa.csv` | Rutas enrutadas del SPA con su componente y `archivo:línea` | 89 |
| `componentes-pages.txt` | Componentes (`*.component.ts`) bajo `src/app/pages` | 80 |
| `dialogos.txt` | Componentes de diálogo (`*-dialog.component.ts`) | 14 |
| `app-permiso-usos.csv` | Cada uso de la directiva `*appPermiso` en plantillas `.html` con su expresión | 85 |
| `i18n-scopes.csv` | Scopes de traducción y número de claves en español | 36 |
| `pruebas-backend.csv` | Archivos de prueba del backend con su número de atributos `[Fact]`/`[Theory]` | 204 archivos, 1344 pruebas |
| `pruebas-spa.csv` | Specs del SPA con su número de casos `it(` | 83 specs, 584 casos |

## Columna `permiso` de `endpoints.csv`

- `Modulo:Accion`: `[HasPermission("Modulo", "Accion")]`.
- `Modulo:Accion|PLATAFORMA`: `[HasPermissionOrPlataforma("Modulo", "Accion")]`; pasa quien tiene el permiso **o** el operador de plataforma. Hoy son los tres endpoints de lectura de `Empresas` (`Empresas:Leer|PLATAFORMA`). Estos pares se cuentan en `permisos.csv` como cualquier otro par (`Empresas:Leer` figura con 3 endpoints).
- `PLATAFORMA`: solo `[Authorize(Policy = PlataformaAdminPolicy.Name)]` (operador de plataforma, sin permiso de módulo). No cuenta en `permisos.csv`.
- `ANONIMO`: `[AllowAnonymous]`. `SOLO_AUTENTICADO`: `[Authorize]` en la acción o en la clase, sin permiso de módulo.
- Si una acción no tiene ninguno de los anteriores el script escribe `SIN_ATRIBUTO` (hoy no hay ninguna). `GET /health` es un minimal API de `Program.cs` y no figura.

## Convenciones del generador

- Las listas salen de `git archive` de cada commit: solo existen archivos versionados, sin `bin`/`obj`; las pruebas cuentan únicamente archivos con al menos un `[Fact]`/`[Theory]` (los de apoyo no aparecen).
- Orden: comparación ordinal de cadenas (mayúsculas antes que minúsculas: `DIGEMID` antes que `Dashboard`). En `codigos-error-ocurrencias.csv` las ocurrencias de un mismo código van ordenadas por la cadena `archivo:línea` (por eso `:111` precede a `:56`) y la columna `tipos` de `codigos-error.csv` sigue ese orden.
- `endpoints.csv`: el nombre del controlador es el del archivo (`DIGEMIDController.cs` → `DIGEMID`, `POSController.cs` → `POS`); ASP.NET no distingue mayúsculas en la ruta. La línea es la del atributo `[Http*]`. El orden es el de los archivos de controlador.
- `scripts-dbup.csv`: `lineas` es el número real de líneas (igual que `wc -l`). Las generaciones anteriores daban una más.
- `tablas.csv`: el nombre es el de `CREATE TABLE`, sin corchetes ni `dbo`.
- `entidades.csv` y `enums.csv`: el nombre es el de la clase o enum declarado en el archivo; ordenados por ese nombre.
- `commands-queries.csv`: archivos `*Command.cs` y `*Query.cs` de `SOFIA.Application` salvo los de `Common/Interfaces`.
- `rutas-spa.csv`: primero las rutas de `app.routes.ts` y luego las de cada `*.routes.ts` cargado con `loadChildren`, en orden de ruta de archivo; no incluye redirecciones ni la ruta comodín. La línea es la de `path:`.
- `i18n-scopes.csv`: claves hoja del `es.json` de cada scope.

## Rutas de archivo

Las rutas de `fuente` son las del commit (índice de git). En el disco de Windows cinco archivos de dominio tienen otra capitalización que el commit (`DigemidActaDestruccion.cs` frente a `DIGEMIDActaDestruccion.cs`, igual `DigemidActaDetalle`, `DigemidCatalogoProducto`, `DigemidInventarioCuarentena` y `PosSesionCaja` frente a `POSSesionCaja`); en Windows ambas abren el mismo archivo. Las generaciones anteriores usaban la capitalización del disco, así que esas rutas cambian de forma sin que el archivo haya cambiado.

Limitaciones del script: detecta códigos de error solo en llamadas `Error.<Tipo>(...)` (tipos `Conflict`, `Failure`, `Forbidden`, `NotFound`, `Unauthorized`, `Validation`); no se ejecuta la aplicación.
