# Inventarios base

Generados por script (`inventario.js`, en el scratchpad de la sesión) leyendo el árbol de trabajo. Primera generación: 2026-10-06 sobre `sofia-apis` `00991bc` y `sofia-web-spa` `31af681`. **Regenerados el 2026-10-13 12:17 sobre `43f5529` y `eeffbb6`** (los conteos de la tabla son los de esa regeneración si difieren; ver la salida del script). Son la lista de control contra la que se validan las tablas de cobertura de cada documento.

El árbol de trabajo tenía cambios sin commitear al generarlos (ver `REGLAS-AGENTES.md`, sección 3). El script `0028_Ventas_Sesion_Anulacion.sql` quedó commiteado en `358935e`.

| Archivo | Contenido | Elementos |
|---|---|---|
| `endpoints.csv` | Controlador, método HTTP, ruta, permiso (`Modulo:Accion`, `ANONIMO` o `SOLO_AUTENTICADO`), política de límite de peticiones, acción C# y `archivo:línea` | 241 |
| `permisos.csv` | Pares módulo × acción usados en `[HasPermission]` y cuántos endpoints los exigen | 105 (27 módulos) |
| `scripts-dbup.csv` | Scripts DbUp y su número de líneas | 28 |
| `tablas.csv` | Tablas con el script y la línea de su `CREATE TABLE` | 50 |
| `codigos-error.csv` | Códigos de error literales distintos, con su tipo y número de ocurrencias | 347 |
| `codigos-error-ocurrencias.csv` | Cada ocurrencia de `Error.<Tipo>("codigo", …)` con `archivo:línea` | 719 |
| `codigos-error-no-literales.csv` | Llamadas a `Error.<Tipo>(…)` cuyo código es una constante o variable (hay que resolverlas a mano) | 14 |
| `validadores.csv` | Validadores FluentValidation; su código de error es dinámico: `{Request}.{Propiedad}` | 83 |
| `commands-queries.csv` | Commands y queries de `SOFIA.Application` (113 commands y 90 queries) | 203 |
| `entidades.csv`, `enums.csv` | Entidades y enums de `SOFIA.Domain` | — |
| `rutas-spa.csv` | Rutas enrutadas del SPA con su componente y `archivo:línea` | 86 |
| `componentes-pages.txt` | Componentes bajo `src/app/pages` | 75 |
| `dialogos.txt` | Componentes de diálogo | 12 |
| `app-permiso-usos.csv` | Cada uso de la directiva `*appPermiso` con su expresión | 85 |
| `i18n-scopes.csv` | Scopes de traducción y número de claves en español | 34 |
| `pruebas-backend.csv` | Archivos de prueba del backend con su número de atributos `[Fact]`/`[Theory]` | 193 archivos, 1254 pruebas |
| `pruebas-spa.csv` | Specs del SPA con su número de casos `it(` | 75 specs, 492 casos |

Limitaciones del script: detecta códigos de error solo en llamadas `Error.<Tipo>(...)`; las rutas de endpoints conservan el nombre del controlador tal como está en la clase (ASP.NET no distingue mayúsculas en la ruta).
