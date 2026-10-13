# Inventario de pantallas del SPA — Parte E: inventario y atención

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-13 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código del árbol de trabajo del SPA (componentes, plantillas, servicios, modelos e i18n). Del backend solo se abrieron los controladores, validadores y manejadores necesarios para anotar permisos, códigos de error y comparar validaciones. No se usó el navegador ni la API local. `git status --short` en ambos repos al empezar: sin cambios sin commitear (ver §7). |

Este documento es la parte E del inventario de pantallas. Cubre cinco módulos: Lotes, Stock por sucursal, Pacientes, Profesionales de salud y Recetas médicas (18 rutas). Lo compartido (guards, interceptor, `page-header`, `confirm-dialog`, carga masiva común, textos de la raíz) lo documenta la [parte A](19a-acceso-layout-y-compartidos.md); el formato de las fichas y el patrón de búsqueda y detalle están validados en la [parte C, §0](19c-catalogos-1.md). Las reglas de negocio de estos módulos están en [01c1](../01-reglas-de-negocio/01c1-inventario-y-recetas.md) (LOT, STK, REC) y [01b](../01-reglas-de-negocio/01b-catalogos.md) (PAC, PRO), y los permisos por endpoint en [05b](../05-seguridad/05b-matriz-de-permisos.md).

Convenciones: las rutas de archivo son relativas a `sofia-web-spa/` salvo que empiecen por `sofia-apis/`. Entre paréntesis va la clave i18n; las claves de scope se escriben con su prefijo real (por ejemplo `lotes.fields.producto` está en `public/i18n/lotes/es.json`, clave `fields.producto`). `app/` abrevia `src/app/`. Las plantillas de estos módulos abren con `*transloco="let t; scope: '<scope>'; read: '<scope>'"`, así que `t('x')` equivale a `<scope>.x`. Los números de línea de los `.html` y `.ts` son los del árbol de trabajo (que coincide con `HEAD` en estas carpetas).

## Contenido

0. Comportamientos comunes a las pantallas de este documento
1. Lotes: `/lotes`, `/lotes/nueva`, `/lotes/:id`
2. Stock por sucursal: `/stock-por-sucursal`, `/stock-por-sucursal/nueva`, `/stock-por-sucursal/:id`
3. Pacientes: `/pacientes`, `/pacientes/nueva`, `/pacientes/carga-masiva`, `/pacientes/:id`
4. Profesionales de salud: `/profesionales-salud`, `/profesionales-salud/nueva`, `/profesionales-salud/carga-masiva`, `/profesionales-salud/:id`
5. Recetas médicas: `/recetas-medicas`, `/recetas-medicas/escanear`, `/recetas-medicas/nueva`, `/recetas-medicas/:id`
6. Eliminar: mapa de códigos de error y mensajes de las pantallas con botón «Eliminar»
7. Pendientes y «No verificado»
8. Tabla de cobertura
9. Claves i18n sin uso
10. Fuentes

---

## 0. Comportamientos comunes a las pantallas de este documento

Lo que no se repite en cada ficha, con su fuente. Lo que sea distinto en una pantalla se dice en su ficha. El patrón general es el de [19c §0](19c-catalogos-1.md#0-comportamientos-comunes-a-las-pantallas-de-este-documento); aquí se indican las diferencias.

- **Cómo se decide el permiso de ruta.** Igual que en 19c §0: `canActivateChild: [passwordChangeGuard, permissionGuard]` (`app/app.routes.ts:37`), módulo por el primer segmento de la URL en `ROUTE_PERMISSIONS` (`app/core/auth/route-permissions.ts:2-26`) y siempre `Leer`; si el segundo segmento es `nueva`, `nuevo` o `carga-masiva` se exige además `Crear` (`route-permissions.ts:36,68-72`), salvo las excepciones de `ROUTE_ACTIONS` (`:29-33`); `/x/:id` solo exige `Leer`. Sin permiso: snackbar «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard` (`app/core/auth/permission.guard.ts:28-29`). Módulos y excepciones de esta parte:

  | Ruta | Módulo de permisos | Acción exigida además de `Leer` | Fuente |
  |---|---|---|---|
  | `/lotes` | `Inventarios` | — | `route-permissions.ts:17` |
  | `/lotes/nueva` | `Inventarios` | `Crear` | `route-permissions.ts:17,36` |
  | `/stock-por-sucursal` | `Inventarios` | — | `route-permissions.ts:18` |
  | `/stock-por-sucursal/nueva` | `Inventarios` | **`Actualizar`** (no `Crear`) | `route-permissions.ts:32` |
  | `/pacientes`, `/profesionales-salud` | `Pacientes`, `ProfesionalesSalud` | — | `route-permissions.ts:21-22` |
  | `/pacientes/nueva`, `/pacientes/carga-masiva`, `/profesionales-salud/nueva`, `/profesionales-salud/carga-masiva` | `Pacientes`, `ProfesionalesSalud` | `Crear` | `route-permissions.ts:36` |
  | `/recetas-medicas` | `Recetas` | — | `route-permissions.ts:23` |
  | `/recetas-medicas/nueva` | `Recetas` | `Crear` | `route-permissions.ts:36` |
  | `/recetas-medicas/escanear` | `Recetas` | **`Analizar`** (no `Crear`) | `route-permissions.ts:30` |
  | Todas las rutas `/x/:id` | el de su módulo | — | — |

- **Menú.** Dos grupos: «Inventario» (`breadcrumbs.inventario`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:100-116`) con «Lotes» (`nav.lotes`) y «Stock por Sucursal» (`nav.stockPorSucursal`), y «Pacientes & Atención» (`breadcrumbs.pacientesAtencion`, `:134-155`) con «Pacientes» (`nav.pacientes`), «Profesionales de Salud» (`nav.profesionalesSalud`) y «Recetas Médicas» (`nav.recetasMedicas`). Una opción se muestra solo si el usuario tiene `Leer` del módulo de su ruta; un grupo sin opciones visibles se oculta ([19a §4.3](19a-acceso-layout-y-compartidos.md)). La miga del grupo «Inventario» y la de «Pacientes & Atención» no tienen enlace.
- **Directiva `*appPermiso`.** Con un solo valor (`'Crear'`, `'Actualizar'`, `'Eliminar'`, `'Analizar'`) toma el módulo de la URL actual y exige esa acción; el rol `Admin` pasa siempre (`app/shared/directives/permiso.directive.ts:20-31`). Ver [19a §7.1](19a-acceso-layout-y-compartidos.md). Los 17 usos de estas carpetas están en la tabla de cobertura (§8).
- **Aviso al usuario.** Snackbar con el botón «Cerrar» (`actions.close`): error 5 s, éxito 3 s; con `showServerError` y un `traceId` en la respuesta, 10 s y «Referencia: …» ([19a §6.11 y §6.12](19a-acceso-layout-y-compartidos.md)). Pacientes y profesionales usan `showServerError` solo al exportar; el resto de avisos de esta parte usan `showError` y `showSuccess`.
- **Estado de la búsqueda.** Las cinco pantallas de búsqueda guardan filtros, página y tamaño de página en memoria (`SearchStateService`, `app/core/services/shared/search-state.service.ts:10-28`) con la URL sin query como clave; al volver desde el detalle se restauran y se repite la búsqueda (`ngOnInit` de cada componente). «Limpiar» (`actions.clear`) vacía el formulario, oculta la tabla, vuelve a la página 1 y borra ese estado. El dashboard escribe en este mismo estado para abrir Stock por sucursal con los filtros ya puestos (§2.1).
- **«Buscar» no vuelve a la página 1.** En las cinco pantallas, la búsqueda usa `pageIndex() + 1`, el índice de página actual (`lotes-search.component.ts:126`, `stock-search.component.ts:159`, `pacientes-search.component.ts:90`, `profesionales-salud-search.component.ts:75`, `recetas-search.component.ts:132`). Si estás en la página 3 y cambias un filtro, «Buscar» pide la página 3 de los nuevos filtros. Solo cambia el tamaño o la página con el paginador.
- **Resultados.** Los resultados se muestran solo tras «Buscar» (o por estado guardado). Encabezado «Resultados (N)» (`generic-labels.results`). Paginador con 10, 25 y 50 filas y botones primera/última, 10 por defecto ([19a §6.9](19a-acceso-layout-y-compartidos.md)). El primer dato de cada fila es siempre un enlace a `/<ruta>/:id`.
- **Fila sin resultados.** Lotes, Stock por sucursal y Recetas médicas muestran «Sin resultados» (`generic-labels.no-results`); Pacientes y Profesionales de salud muestran el texto «Resultados» (`generic-labels.results`) en esa fila (`pacientes-search.component.html:178`, `profesionales-salud-search.component.html:122`). Ver Observaciones de cada ficha.
- **Fechas.** Las columnas de fecha se pintan siempre `dd/MM/yyyy` (no dependen del idioma). Los selectores de fecha siguen el idioma activo ([19a §6.10](19a-acceso-layout-y-compartidos.md)). Lotes envía las fechas como medianoche local con su desfase (`toLocalMidnightIso`), Pacientes y Recetas (al guardar) como `yyyy-MM-dd` (`toDateOnly`), y la búsqueda de Recetas como instante (`toISOString()`).
- **Autocompletar con lista del servidor** (Producto en Lotes y Stock, Sucursal y Lote en Stock, Paciente y Médico en Recetas). Cada uno escribe con espera de 300 ms, trae hasta 20 elementos y elige una opción con el clic sobre `mat-option` (`(click)` en la opción). Elegir con el teclado (flechas y Enter) podría no disparar ese clic: no verificado en ejecución (§7). Ninguna de estas suscripciones tiene manejador de error: si la consulta falla (por ejemplo por falta del permiso `Leer` del módulo consultado), no se avisa.
- **Modo ver / editar** (detalle de Lotes, Pacientes y Profesionales de salud). Las rutas `/x/nueva` y `/x/:id` usan el mismo componente: sin parámetro `id` es **crear** (formulario habilitado, `isNew` e `isEditMode` verdaderos); con `id` abre en **ver** (formulario deshabilitado, carga los datos con `GET`) y «Editar» (`actions.edit`) lo habilita. «Cancelar» en crear vuelve a la búsqueda; en editar restaura la copia leída y vuelve a modo ver. «Eliminar» solo aparece en modo editar de un registro existente. Recetas médicas y Stock por sucursal tienen su propio patrón (§2 y §5.2).
- **Crear y editar, qué pasa tras guardar.** Lotes y Stock por sucursal: crear y editar navegan a la búsqueda. Pacientes, Profesionales de salud y Recetas médicas: crear navega a la búsqueda; editar se queda en la ficha, vuelve a modo ver y recarga los datos.
- **Mensaje de campo obligatorio:** «Este campo es obligatorio» (`general.errors.required`, raíz). Las longitudes máximas se impiden con el atributo HTML `maxlength` (el campo no deja escribir más). Un valor compuesto solo de espacios cumple `Validators.required` de Angular, pero la API lo rechaza (ver Observaciones de cada ficha).
- **Diálogo de eliminar.** `ConfirmDialogComponent` ([19a §6.1](19a-acceso-layout-y-compartidos.md)), ancho 400 px (Recetas médicas no fija ancho), título y mensaje de cada scope y botones por defecto «Cancelar» y «Confirmar»; confirmar llama al `DELETE`, cancelar no hace nada.
- **Guardado: mapa de errores.** Pacientes, Profesionales de salud y Recetas médicas usan `saveErrorKey` (`app/shared/utils/save-error-key.util.ts:11-15`): leen `error.code` de la respuesta y resuelven, en este orden, el código propio de la pantalla, los códigos de base de datos `Db.Duplicado` → «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» (`errors.dbDuplicado`), `Db.ReferenciaInvalida` → «El registro hace referencia a datos que ya no existen o todavía está en uso.» (`errors.dbReferenciaInvalida`) y `Db.ValorInvalido` → «Uno de los valores está fuera del rango permitido.» (`errors.dbValorInvalido`), y por último el mensaje genérico de la pantalla. **Lotes decide solo por el estado HTTP** (409 o no) y **Stock por sucursal** usa un mapa propio de dos códigos; ninguno de los dos usa los textos `errors.db*`.
- **Eliminar: solo el estado HTTP.** En Lotes, Pacientes, Profesionales de salud y Recetas médicas la pantalla no lee el código de error al eliminar: con estado 409 muestra el mensaje «en uso» de la pantalla y con cualquier otro error el mensaje genérico (§6).
- **Exportación.** Botón «Exportar» (`actions.export`, icono `download`) bajo la tabla, a la izquierda, solo tras buscar. Hace `POST` a `<base>/exportar` con `{ headers, [dateFormat,] ...filtros }` (los filtros del formulario en ese momento, no necesariamente los de la última búsqueda) y descarga el blob con un nombre fijo que sale del i18n de cada scope. El permiso del endpoint es `<Módulo>:Leer`. Ante un error muestra el mensaje «Error al buscar …» o «Error al consultar el stock» de la pantalla. Si el resultado supera el máximo de filas exportables, la API responde con el error `Export.TooManyRows` («La exportación supera el máximo de … filas. Aplica filtros para reducir los resultados.») que estas pantallas no traducen (muestran su mensaje genérico).
- **Carga masiva.** Solo Pacientes y Profesionales de salud la tienen. Las dos rutas `/x/carga-masiva` usan el componente común `app-carga-masiva-page` ([19a §6.5 y §6.6](19a-acceso-layout-y-compartidos.md)). El título de la pantalla es «Carga Masiva» (`carga-masiva.title`) y el archivo de plantilla se descarga como `plantilla-{backRoute sin la barra}.xlsx`. Las etiquetas de columna se resuelven contra la **raíz** `fields.*` (no contra el scope). El campo `required` de cada columna se declara pero el componente común no lo lee.
- **Código sin uso en los detalles.** Los detalles de Pacientes y Profesionales de salud definen `goToNew()` que ninguna plantilla llama.
- **Límite de peticiones.** Solo el análisis de recetas con IA tiene un límite propio (10 por minuto por empresa); ver §5.3.

---

## 1. Lotes

Carpeta `app/pages/lotes`, scope i18n `lotes` (`provideTranslocoScope('lotes')` en `lotes-search.component.ts:25` y `lotes-detail.component.ts:31`). Rutas (`app/pages/lotes/lotes.routes.ts`): `''` → `LotesSearchComponent`, `nueva` y `:id` → `LotesDetailComponent`. No hay carga masiva de lotes. Los lotes no llevan stock: el stock se ingresa aparte en [Stock por sucursal](#2-stock-por-sucursal) (RN-LOT-016 de 01c1).

### 1.1 `/lotes` — Lotes (búsqueda)

- **Componente:** `LotesSearchComponent` (`app/pages/lotes/lotes-search/lotes-search.component.ts`, `.html`). Scope `lotes`.
- **Para qué sirve:** buscar lotes de inventario por producto, número de lote y rangos de fecha de caducidad y de fabricación, abrirlos, exportar la lista y entrar a crear uno.
- **Cómo se llega:** menú lateral, grupo «Inventario» (`breadcrumbs.inventario`) → «Lotes» (`nav.lotes`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:104-108`, grupo `id: 4`); botón volver y migas del detalle; redirección tras guardar o eliminar un lote.
- **Permiso de ruta:** `Inventarios:Leer` (`lotes` → `Inventarios`, `app/core/auth/route-permissions.ts:17`). Sin permiso: snackbar «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard`.
- **Migas y cabecera:** título «Lotes» (`lotes.title`); migas «Inicio» (a `/dashboard`) / «Inventario» (sin enlace) / «Lotes» (activa); sin botón volver (`.html:3-6`, `.ts:62-66`). Tarjeta de filtros con título «Búsqueda de Lotes» (`lotes.search.title`) y subtítulo «Ingresa los criterios de búsqueda» (`generic-labels.search-subtitle`).
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:53-60`, `.html:29-122`; los dos primeros ocupan media fila cada uno y los cuatro de fecha un cuarto de fila cada uno)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Producto (`lotes.fields.producto`) | `productoNombre` | Texto con autocompletar y lupa; Enter busca. Placeholder «Buscar producto...» (`lotes.autocomplete.producto-placeholder`) | Texto libre. Al escribir (espera de 300 ms, `.ts:72`) consulta medicamentos por nombre comercial; cada opción muestra el nombre comercial. Si no hay coincidencias muestra una opción deshabilitada «Sin resultados» (`lotes.autocomplete.no-results`). Elegir una opción solo copia su nombre al campo; el filtro enviado es el texto del campo (coincidencia «contiene» en el servidor), no el identificador | Vacío |
| Número de Lote (`lotes.fields.numeroLote`) | `numeroLote` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Caducidad Desde (`lotes.fields.caducidadDesde`) | `caducidadDesde` | Fecha (`mat-datepicker`) | Sin mínimo ni máximo | Vacío |
| Caducidad Hasta (`lotes.fields.caducidadHasta`) | `caducidadHasta` | Fecha | Sin mínimo ni máximo; el servidor incluye el día completo (`GetLotesQueryHandler.cs:43-47`) | Vacío |
| Fabricación Desde (`lotes.fields.fabricacionDesde`) | `fabricacionDesde` | Fecha | Sin mínimo ni máximo; los lotes sin fecha de fabricación quedan excluidos de este filtro (`GetLotesQueryHandler.cs:49-52`) | Vacío |
| Fabricación Hasta (`lotes.fields.fabricacionHasta`) | `fabricacionHasta` | Fecha | Igual que el anterior, incluye el día completo (`:54-58`) | Vacío |

No hay deslizadores numéricos ni selects. Los resultados se muestran solo tras pulsar «Buscar» (o al restaurar un estado guardado, §0). «Limpiar» vacía el formulario, vacía la lista de sugerencias, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:148-156`). No hay validación cruzada entre «Desde» y «Hasta»: si «Desde» es posterior a «Hasta» la búsqueda simplemente no devuelve filas (por el filtro del servidor; no se probó).

**Tabla de resultados** (`.html:148-200`, columnas `.ts:68`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Producto (`lotes.table.producto`) | `nombreProducto` | Texto | Sí: `[routerLink]="[row.id]"` → `/lotes/:id` |
| Número de Lote (`lotes.table.numeroLote`) | `numeroLoteMfr` | Texto | No |
| Fecha de Fabricación (`lotes.table.fabricacion`) | `fechaFabricacion` | `dd/MM/yyyy` con el pipe `date` sobre los 10 primeros caracteres; «—» si es nula (`.html:172-177`) | No |
| Fecha de Caducidad (`lotes.table.caducidad`) | `fechaCaducidad` | `dd/MM/yyyy` (`.html:186`), fijo, no depende del idioma | No |

Orden por defecto: lo fija el servidor, por fecha de creación descendente (`GetLotesQueryHandler.cs:61`). Encabezado «Resultados (N)» (`generic-labels.results`). Paginación 10/25/50 (10 por defecto), con botones primera y última página. Fila sin resultados: «Sin resultados» (`generic-labels.no-results`, `.html:197`). **No hay indicador de lote vencido ni por vencer**: la plantilla no compara las fechas con la fecha actual ni colorea ni marca filas (comprobado en `.html` y `.ts`; búsqueda de «vencid» en la carpeta sin resultados).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `Inventarios:Crear`, `.html:19`) | Nunca | `goToNew()` → `/lotes/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/lotesinventario` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/lotesinventario/exportar` |
| Enlace con el nombre del producto | Primera columna | Con resultados | — | Navega a `/lotes/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar lotes» (`lotes.search.search-error`) | `lotes-search.component.ts:143` |
| Falla la exportación (cualquier error) | Error | El mismo texto «Error al buscar lotes» (no hay clave propia para exportar) | `lotes-search.component.ts:186` |
| Falla la consulta de sugerencias de producto | — | Sin aviso: la suscripción no tiene manejador de error (`.ts:70-82`) | `lotes-search.component.ts:71-81` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `LoteService.search` (`app/pages/lotes/services/lote.service.ts:18-29`) | `GET /api/v1/lotesinventario?pageNumber&pageSize[&productoNombre][&numeroLote][&caducidadDesde][&caducidadHasta][&fabricacionDesde][&fabricacionHasta]` (fechas como `yyyy-MM-ddT00:00:00±hh:mm`, medianoche local con su desfase, `date-only.util.ts:36-42`) | `Inventarios:Leer` (`LotesInventarioController.cs:19`) |
| Escribir en «Producto» | `MedicamentoService.search` (`app/pages/medicamentos/services/medicamento.service.ts:23-38`) | `GET /api/v1/medicamentos?pageNumber=1&pageSize=20&nombreComercial=…` | `Medicamentos:Leer` (`MedicamentosController.cs:27`) |
| Exportar | `LoteService.exportar` (`:47-60`) | `POST /api/v1/lotesinventario/exportar` (respuesta blob) | `Inventarios:Leer` (`LotesInventarioController.cs:95`) |

**Exportación**
- Cuerpo (`.ts:168-177`): `headers` con 4 textos traducidos, en orden: «Producto» (`lotes.table.producto`), «Número de Lote» (`lotes.table.numeroLote`), «Fecha de Fabricación» (`lotes.table.fabricacion`) y «Fecha de Caducidad» (`lotes.table.caducidad`); `dateFormat` = `dd/MM/yyyy` si el idioma activo es `es`, `MM/dd/yyyy` en otro caso; y los seis filtros del formulario en ese momento (no necesariamente los de la última búsqueda), cada uno solo si tiene valor.
- Archivo: «lotes.xlsx» (`lotes.search.export-filename`).
- Carga masiva: no existe para lotes.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0. El autocompletado del producto necesita `Medicamentos:Leer` aunque la pantalla solo exija `Inventarios:Leer`; sin ese permiso la consulta falla, no hay manejador y la lista de sugerencias no se vuelve a alimentar (el efecto visual exacto no se probó; ver §7).

**Observaciones:** `goToNew()` navega con `router.navigate(['/lotes/nueva'])`. Los specs `lotes-search.component.spec.ts` (3 casos: estado inicial oculto, `search()` muestra resultados, `clear()` los oculta) y `lote.service.spec.ts` (7 casos) cubren el servicio y la búsqueda; no cubren la visibilidad de «Nuevo» según permisos.

### 1.2 `/lotes/nueva` y `/lotes/:id` — Lotes (detalle)

- **Componente:** `LotesDetailComponent` (`app/pages/lotes/lotes-detail/lotes-detail.component.ts`, `.html`). Scope `lotes`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un lote de un producto, ver sus datos y, con permisos, editar su número y sus fechas o eliminarlo.
- **Cómo se llega:** `/lotes/nueva`: botón «Nuevo» de la búsqueda. `/lotes/:id`: enlace del producto en la tabla de resultados.
- **Permiso de ruta:** `/lotes/nueva` → `Inventarios:Leer` + `Inventarios:Crear` (segmento `nueva`, `route-permissions.ts:36,68-71`); `/lotes/:id` → `Inventarios:Leer`.
- **Migas y cabecera:** título «Lotes» (`lotes.title`); migas «Inicio» / «Inventario» (sin enlace) / «Lotes» (con enlace a `/lotes`); botón volver a `/lotes` (`.html:3-7`, `.ts:73-77`).
- **Modos** (`ngOnInit`, `.ts:93-104`): sin parámetro `id` → **crear**; con `id` → **ver** (formulario y producto deshabilitados, carga el lote con `GET`) y «Editar» habilita el formulario salvo el producto (`enterEditMode`, `.ts:138-143`: «Product cannot be changed once the batch exists»).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Lotes» + «Datos del registro» (`generic-labels.detail-subtitle`) + botón «Editar» | No | Sí (botón solo con `Inventarios:Actualizar`) | No |
| Producto | Editable (autocompletar) | Deshabilitado | Deshabilitado (no se puede cambiar) |
| Número de Lote, fechas | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:62-71`, `.html:37-100`; cada campo ocupa media fila)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Producto (`lotes.fields.producto`) | `productoCtrl` (control suelto, no está en `form`) | Texto con autocompletar; placeholder «Buscar producto...»; opción deshabilitada «Sin resultados» si no hay coincidencias | Sí, solo al crear | Debe elegirse una opción de la lista: al guardar, si el texto del campo no coincide con el nombre de la opción elegida se descarta la elección (`.ts:161-163`) y se bloquea el guardado (`.ts:165-167`) | «Este campo es obligatorio» (`general.errors.required`), pintado fuera del campo con clase `sofia-field-error`, solo si `isNew() && productoTouched() && !selectedProducto()` (`.html:61-65`) | Sí / No |
| Número de Lote (`lotes.fields.numeroLote`) | `numeroLoteMfr` | `matInput`, `maxlength="100"` | Sí | `Validators.required`, `Validators.maxLength(100)` (`.ts:68`) | «Este campo es obligatorio» (`general.errors.required`, `.html:72`); la longitud la impide el atributo `maxlength` | Sí / Sí |
| Fecha de Fabricación (`lotes.fields.fabricacion`) | `fechaFabricacion` | Fecha (`mat-datepicker`) | No | Ninguna; sin mínimo ni máximo | — | Sí / Sí |
| Fecha de Caducidad (`lotes.fields.caducidad`) | `fechaCaducidad` | Fecha (`mat-datepicker`) | Sí | `Validators.required` (`.ts:70`); sin mínimo ni máximo | «Este campo es obligatorio» (`general.errors.required`, `.html:97`) | Sí / Sí |

Sin ayuda de campo. El selector de fecha sigue el idioma activo (escritura día/mes/año con español, [19a §6.10](19a-acceso-layout-y-compartidos.md)). El cuerpo enviado al crear es `{ productoId, numeroLoteMfr, fechaFabricacion, fechaCaducidad }` y al editar `{ numeroLoteMfr, fechaFabricacion, fechaCaducidad }` (`.ts:186-191,195-199`); las fechas viajan como medianoche local con su desfase (`toLocalMidnightIso`), y la fabricación vacía se envía como `undefined` (se omite).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Inventarios:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:145-155`): en crear navega a `/lotes`; en editar restaura los datos leídos y vuelve a ver |
| Guardar (`actions.save`); con spinner mientras guarda | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:157-202`). Marca todo como tocado; inválido → no envía. Crear → `POST /api/v1/lotesinventario`; editar → `PUT /api/v1/lotesinventario/:id`. En ambos casos, con éxito, aviso y navega a `/lotes` (también al editar) |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de un lote existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `Inventarios:Eliminar`, `.html:120`) | `deleting()` | `confirmDelete()` (`.ts:204-215`) → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/lotes` |

**Diálogos que abre:** `ConfirmDialogComponent` ([19a §6.1](19a-acceso-layout-y-compartidos.md)) al pulsar «Eliminar»: ancho 400 px, título «Confirmar eliminación» (`lotes.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este lote?» (`lotes.detail.delete-confirm-message`), botones por defecto «Cancelar» y «Confirmar». Confirmar → `DELETE`; cancelar no hace nada.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Lote guardado correctamente» (`lotes.detail.save-success`) | `lotes-detail.component.ts:172` |
| Guardar: respuesta con estado HTTP 409 (cualquier código) | Error | «Ya existe un lote con ese número para el producto seleccionado» (`lotes.detail.save-conflict`) | `.ts:175-182` |
| Guardar: cualquier otro error (400, 403, 404, 500, sin conexión…) | Error | «Error al guardar el lote» (`lotes.detail.save-error`) | `.ts:175-182` |
| Eliminación correcta | Éxito | «Lote eliminado correctamente» (`lotes.detail.delete-success`); navega a `/lotes` | `.ts:221-222` |
| Eliminar: respuesta con estado 409 (cualquier código) | Error | «No se puede eliminar un lote con stock o ventas registradas.» (`lotes.detail.delete-in-use`) | `.ts:224-231` |
| Eliminar: cualquier otro error | Error | «Error al eliminar el lote» (`lotes.detail.delete-error`) | `.ts:224-231` |
| Error al cargar el lote | Error | «Error al cargar el lote» (`lotes.detail.load-error`) | `.ts:116` |

La pantalla decide el mensaje **solo por el estado HTTP**; no lee `error.code` ni usa `saveErrorKey`. Mapa de lo que devuelve el backend frente a lo que ve el usuario al guardar:

| Respuesta de la API | Código | Mensaje que ve el usuario |
|---|---|---|
| 400 por duplicado: ya existe un lote no eliminado con el mismo número para el producto (`CreateLoteInventarioCommandHandler.cs:26-29`, `UpdateLoteInventarioCommandHandler.cs:28-31`; `Result.Failure` sin código de estado usa 400, `Result.cs:32`) | `LoteInventario.Duplicate` | «Error al guardar el lote» (genérico). **El aviso de lote duplicado no aparece en el caso normal** |
| 409 del manejador global por violación del índice único `UX_Lotes_Producto_NumeroLote` (dos altas simultáneas del mismo número; `GlobalExceptionHandler.cs:102`, `LoteInventarioConfiguration.cs:50-53`) | `Db.Duplicado` | «Ya existe un lote con ese número para el producto seleccionado» |
| 409 por conflicto de concurrencia (`GlobalExceptionHandler.cs:87`) | `Concurrency.Conflict` | El mismo texto de «lote duplicado» (no es la causa real) |
| 409 por referencia inválida (`GlobalExceptionHandler.cs:104`) | `Db.ReferenciaInvalida` | El mismo texto de «lote duplicado» (no es la causa real) |
| 400 por el validador: caducidad vencida («Expiration date cannot be in the past»), fabricación futura, número vacío o de más de 100 caracteres, producto vacío (`CreateLoteInventarioCommandValidator.cs:14-27`; código `CreateLoteInventarioCommand.<Propiedad>`; al editar `UpdateLoteInventarioCommand.<Propiedad>`) | `{Request}.{Propiedad}` | «Error al guardar el lote» |
| 400 de la entidad: fabricación posterior a la caducidad, solo al crear (`LoteInventario.cs:48-51`) | `LoteInventario.FechaFabricacion` | «Error al guardar el lote» |
| 404 producto inexistente al crear (`CreateLoteInventarioCommandHandler.cs:19`) o lote inexistente al editar (`UpdateLoteInventarioCommandHandler.cs:19`) | `Medicamento.NotFound` / `LoteInventario.NotFound` | «Error al guardar el lote» |

Todo esto sale de leer el código; no se ejecutó (ver §7).

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir `/lotes/:id` | `LoteService.getById` (`lote.service.ts:31-33`) | `GET /api/v1/lotesinventario/{id}` | `Inventarios:Leer` (`LotesInventarioController.cs:27`) |
| Crear | `create` (`:35-37`) | `POST /api/v1/lotesinventario` | `Inventarios:Crear` (`:47`) |
| Editar | `update` (`:39-41`) | `PUT /api/v1/lotesinventario/{id}` | `Inventarios:Actualizar` (`:57`) |
| Eliminar | `delete` (`:43-45`) | `DELETE /api/v1/lotesinventario/{id}` | `Inventarios:Eliminar` (`:71`) |
| Escribir en «Producto» (crear) | `MedicamentoService.search` | `GET /api/v1/medicamentos?pageNumber=1&pageSize=20&nombreComercial=…` | `Medicamentos:Leer` |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga el lote muestra un spinner en lugar de la tarjeta (`.html:9-13`).
- El producto de un lote no se puede cambiar (RN-LOT-012 de 01c1): en editar el campo queda deshabilitado.
- Al editar con éxito la pantalla **no** vuelve al modo ver: navega a `/lotes`, a diferencia de los catálogos de la [parte C](19c-catalogos-1.md#0-comportamientos-comunes-a-las-pantallas-de-este-documento).
- `saving` no se reinicia tras un éxito porque se navega fuera.
- Si falla la carga del lote, la ficha queda vacía y deshabilitada y, con `Inventarios:Actualizar`, sigue visible «Editar».

**Observaciones**
- **Los selectores de fecha no limitan nada** (sin `[min]` ni `[max]`), pero el backend rechaza caducidad vencida (se considera vencida si la fecha es menor o igual a ayer, `LoteInventario.cs:95-97`), fabricación futura y, al crear, fabricación posterior a la caducidad. El usuario solo ve «Error al guardar el lote» (RN-LOT-006 a RN-LOT-008 de 01c1).
- Consecuencia de leer `UpdateLoteInventarioCommandValidator.cs:21-23`: un lote cuya caducidad ya venció no se puede guardar al editar, ni siquiera para corregir su número (la validación exige que la caducidad no esté vencida). No se ejecutó.
- La clave `lotes.detail.save-conflict` describe un duplicado pero el duplicado habitual devuelve 400; el mensaje solo aparece ante 409 (ver tabla de arriba).
- Elegir una sugerencia con el teclado (flechas y Enter) podría no disparar el `(click)` que guarda la elección (`.html:48-52`); ver §7.
- Specs `lotes-detail.component.spec.ts` (4 casos: abre en modo crear, `save()` no hace nada si es inválido, carga en ver, `enterEditMode()` habilita el formulario). No cubren eliminar ni los mensajes de error.
- Claves del scope sin uso: ver §9.

---

## 2. Stock por sucursal

Carpeta `app/pages/stock-por-sucursal`, scope i18n `stock-por-sucursal` (`provideTranslocoScope('stock-por-sucursal')` en `stock-search.component.ts:27` y `stock-detail.component.ts:30`). **Las plantillas leen el scope con el alias en camelCase** (`*transloco="let t; scope: 'stock-por-sucursal'; read: 'stockPorSucursal'"`, `.html:1-2`), y los `.ts` traducen con el prefijo `stockPorSucursal.` (por ejemplo `stockPorSucursal.search.search-error`). El archivo de textos es `public/i18n/stock-por-sucursal/es.json`; las claves de abajo se escriben con el prefijo `stockPorSucursal.`. Rutas (`stock-por-sucursal.routes.ts`): `''` → `StockSearchComponent`; `nueva` (con `data: { isNew: true }`) y `:id` → `StockDetailComponent`. No hay carga masiva ni botón «Eliminar».

Las dos rutas del detalle son **operaciones distintas** aunque compartan componente (`stock-detail.component.ts:111-125`):
- `/stock-por-sucursal/nueva` es el **ingreso de stock**: eliges sucursal, lote y cantidad; llama a `POST /api/v1/lotesinventario/stock` y, si ya existía stock de ese lote en esa sucursal, **suma** la cantidad (el SPA no envía `EsAjusteDirecto`, que por defecto es falso; `RegisterInventarioCommandHandler.cs:41-54`, RN-STK-008 de 01c1).
- `/stock-por-sucursal/:id` **muestra y ajusta** el stock de un lote en una sucursal: la cantidad nueva **reemplaza** a la anterior (`PUT /api/v1/stockporsucursal/{id}/ajustar`, `AdjustStockCommandHandler.cs:26`, RN-STK-012).

### 2.1 `/stock-por-sucursal` — Stock por Sucursal (búsqueda)

- **Componente:** `StockSearchComponent` (`app/pages/stock-por-sucursal/stock-search/stock-search.component.ts`, `.html`). Scope `stock-por-sucursal`.
- **Para qué sirve:** consultar el stock físico por sucursal, producto y lote (solo de las sucursales permitidas al usuario), filtrar por caducidad y cantidad, exportar la lista, abrir un registro y entrar al ingreso de stock.
- **Cómo se llega:** menú lateral, grupo «Inventario» (`breadcrumbs.inventario`) → «Stock por Sucursal» (`nav.stockPorSucursal`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:110-114`); botón volver y migas del detalle; redirección tras ingresar o ajustar stock; y desde el dashboard, los indicadores de lotes vencidos y por vencer abren esta pantalla con la caducidad ya filtrada (`app/pages/dashboard/dashboard-links.ts:43-77`, a través del estado de búsqueda de §0).
- **Permiso de ruta:** `Inventarios:Leer` (`stock-por-sucursal` → `Inventarios`, `route-permissions.ts:18`). Sin permiso: snackbar «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard`.
- **Migas y cabecera:** título «Stock por Sucursal» (`stockPorSucursal.title`); migas «Inicio» / «Inventario» (sin enlace) / «Stock por Sucursal» (`breadcrumbs.stock-por-sucursal`, activa); sin botón volver (`.html:4-7`, `.ts:71-75`). Tarjeta de filtros con título «Consulta de Stock por Sucursal» (`stockPorSucursal.search.title`) y subtítulo «Ingresa los criterios de búsqueda» (`generic-labels.search-subtitle`).
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:60-69`, `.html:30-153`; tres columnas de un tercio en la primera fila y cuatro de un cuarto en la segunda)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Sucursal (`stockPorSucursal.fields.sucursal`) | `sucursalNombre` | Texto con autocompletar y lupa; Enter busca. Placeholder «Buscar sucursal...» (`stockPorSucursal.autocomplete.sucursal-placeholder`) | Las sugerencias salen de `GET /api/v1/sucursales/permitidas` (se pide una sola vez y se reutiliza), filtradas en el navegador por «contiene» sin distinguir mayúsculas, con espera de 300 ms (`.ts:58,86-103`). Sin coincidencias: opción deshabilitada «Sin resultados» (`stockPorSucursal.autocomplete.no-results`). Elegir una opción copia su nombre; el filtro enviado es el texto | Vacío |
| Producto (`stockPorSucursal.fields.producto`) | `productoNombre` | Texto con autocompletar y lupa; placeholder «Buscar producto...» (`stockPorSucursal.autocomplete.producto-placeholder`) | Sugerencias de `GET /api/v1/medicamentos` por nombre comercial (20 como máximo, espera de 300 ms). El filtro enviado es el texto | Vacío |
| Número de Lote (`stockPorSucursal.fields.numeroLote`) | `numeroLote` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Caducidad Desde (`stockPorSucursal.fields.caducidadDesde`) | `caducidadDesde` | Fecha (`mat-datepicker`) | Sin mínimo ni máximo | Vacío |
| Caducidad Hasta (`stockPorSucursal.fields.caducidadHasta`) | `caducidadHasta` | Fecha | Sin mínimo ni máximo; el servidor incluye el día completo (`GetStockPorSucursalQueryHandler.cs:51-55`) | Vacío |
| Cantidad Mínima (`stockPorSucursal.fields.cantidadMin`) | `cantidadMin` | Número (`<input type="number" min="0">`); Enter busca | Atributo `min="0"` solo del navegador, sin validador del formulario; no es un deslizador | Vacío |
| Cantidad Máxima (`stockPorSucursal.fields.cantidadMax`) | `cantidadMax` | Número (`min="0"`); Enter busca | Igual | Vacío |
| Solo con stock disponible (`stockPorSucursal.fields.soloConStock`) | `soloConStock` | Casilla (`mat-checkbox`) | Marcada: el servidor devuelve solo filas con cantidad mayor que 0 (`GetStockPorSucursalQueryHandler.cs:23-26`) | Marcada |

No hay deslizadores ni selects. Los resultados se muestran solo tras pulsar «Buscar» (o al restaurar un estado guardado). «Limpiar» restablece el formulario con la casilla marcada (`reset({ soloConStock: true })`), vacía las sugerencias, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:181-190`). Si desmarcas la casilla, el SPA envía `soloConStock=false` y el listado incluye filas con cantidad 0 (`stock.service.ts:16`).

**Tabla de resultados** (`.html:179-233`, columnas `.ts:77-83`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Sucursal (`stockPorSucursal.table.sucursal`) | `sucursalNombre` | Texto | Sí: `[routerLink]="[row.id]"` → `/stock-por-sucursal/:id` |
| Producto (`stockPorSucursal.table.producto`) | `productoNombre` | Texto | No |
| Número de Lote (`stockPorSucursal.table.numeroLote`) | `numeroLote` | Texto | No |
| Fecha de Caducidad (`stockPorSucursal.table.caducidad`) | `fechaCaducidad` | `dd/MM/yyyy` (10 primeros caracteres, `.html:210`), fijo | No |
| Cantidad Física (`stockPorSucursal.table.cantidad`) | `cantidadFisica` | Número tal como llega, alineado a la derecha | No |

Orden por defecto: lo fija el servidor, por nombre de sucursal, nombre comercial del producto y fecha de caducidad (`GetStockPorSucursalQueryHandler.cs:68-70`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Sin resultados» (`generic-labels.no-results`, `.html:230`). No hay indicador de lote vencido en la tabla.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Actualizar'"` (= `Inventarios:Actualizar`, `.html:20`); no `Crear` | Nunca | `goToNew()` → `/stock-por-sucursal/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/stockporsucursal` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/stockporsucursal/exportar` |
| Enlace con el nombre de la sucursal | Primera columna | Con resultados | — | Navega a `/stock-por-sucursal/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al consultar el stock» (`stockPorSucursal.search.search-error`) | `stock-search.component.ts:176` |
| Falla la exportación (cualquier error) | Error | El mismo texto «Error al consultar el stock» | `stock-search.component.ts:221-222` |
| Falla la carga de las sucursales permitidas al escribir en «Sucursal» | Error | «No se pudieron cargar tus sucursales» (`stockPorSucursal.autocomplete.sucursales-error`); la lista de sugerencias queda vacía | `stock-search.component.ts:95-98` |
| Falla la consulta de sugerencias de producto | — | Sin aviso: la suscripción no tiene manejador de error | `stock-search.component.ts:105-115` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `StockService.search` (`app/pages/stock-por-sucursal/services/stock.service.ts:12-25`) | `GET /api/v1/stockporsucursal?pageNumber&pageSize&soloConStock[&sucursalNombre][&productoNombre][&numeroLote][&caducidadDesde][&caducidadHasta][&cantidadMin][&cantidadMax]` | `Inventarios:Leer` (`StockPorSucursalController.cs:14`) |
| Escribir en «Sucursal» (primera vez) | `SucursalService.getPermitidas` (`app/pages/sucursales/services/sucursal.service.ts:32-34`) | `GET /api/v1/sucursales/permitidas` | Solo autenticado (`SucursalesController.cs:64-65`) |
| Escribir en «Producto» | `MedicamentoService.search` | `GET /api/v1/medicamentos?pageNumber=1&pageSize=20&nombreComercial=…` | `Medicamentos:Leer` (`MedicamentosController.cs:27`) |
| Exportar | `StockService.exportar` (`:39-45`) | `POST /api/v1/stockporsucursal/exportar` (blob) | `Inventarios:Leer` (`StockPorSucursalController.cs:38`) |

**Exportación**
- Cuerpo (`.ts:202-212`): `headers` con 5 textos traducidos, en orden: «Sucursal» (`table.sucursal`), «Producto» (`table.producto`), «Número de Lote» (`table.numeroLote`), «Fecha de Caducidad» (`table.caducidad`) y «Cantidad Física» (`table.cantidad`); `dateFormat` = `dd/MM/yyyy` con idioma `es`, `MM/dd/yyyy` en otro caso; y los filtros del formulario en ese momento, incluido `soloConStock`.
- Archivo: «stock-por-sucursal.xlsx» (`stockPorSucursal.search.export-filename`).
- Carga masiva: no existe.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0. Los filtros de caducidad con que llega el dashboard se aplican como cualquier estado restaurado: la pantalla repite la búsqueda al abrir. El servidor solo devuelve filas de las sucursales a las que el usuario tiene acceso, y el rol `Admin` ve todas (`GetStockPorSucursalQueryHandler.cs:20-21`, RN-STK-014 de 01c1).

**Observaciones:** `StockService.search` envía siempre `soloConStock` (por defecto `true`, `stock.service.ts:16`). Cantidad mínima y máxima son campos numéricos y no deslizadores. No hay specs de los componentes; `stock.service.spec.ts` tiene 6 casos sobre el servicio.

### 2.2 `/stock-por-sucursal/nueva` — Nuevo Registro de Stock (ingreso de stock)

- **Componente:** `StockDetailComponent` (`app/pages/stock-por-sucursal/stock-detail/stock-detail.component.ts`, `.html:16-116`). Scope `stock-por-sucursal`.
- **Para qué sirve:** registrar el ingreso de un lote en una sucursal con una cantidad. Si el lote ya tenía stock en esa sucursal, la cantidad se suma.
- **Cómo se llega:** botón «Nuevo» de la búsqueda (§2.1).
- **Permiso de ruta:** `Inventarios:Leer` + `Inventarios:Actualizar` (`ROUTE_ACTIONS['stock-por-sucursal/nueva'] = 'Actualizar'`, `route-permissions.ts:32`). **No** se exige `Inventarios:Crear`: `routeAction` devuelve primero la entrada de `ROUTE_ACTIONS` y solo si no existe aplica la regla de `nueva` → `Crear` (`route-permissions.ts:68-72`; `permission.guard.ts:13,28`). Sin permiso: aviso «No tienes permisos para acceder a esta pantalla.» y redirección a `/dashboard`.
- **Migas y cabecera:** título «Stock por Sucursal» (`stockPorSucursal.title`; la clave `stockPorSucursal.create.title` «Nuevo Registro de Stock» existe pero no se usa en ninguna plantilla); migas «Inicio» / «Inventario» (sin enlace) / «Stock por Sucursal» (con enlace a `/stock-por-sucursal`); botón volver a `/stock-por-sucursal`.
- **Modos:** solo crear. `ngOnInit` detecta `data.isNew === true` y deja el componente en modo crear y edición, sin llamar a la API (`.ts:111-116`).

**Campos del formulario** (`.html:19-96`)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error | Ayuda |
|---|---|---|---|---|---|---|
| Sucursal (`stockPorSucursal.fields.sucursal`) | `sucursalCtrl` (control suelto, sin validadores), fila completa | Texto con autocompletar. Sugerencias: sucursales permitidas (`GET /api/v1/sucursales/permitidas`, una sola vez) filtradas en el navegador por «contiene», espera de 300 ms | Sí | Debe elegirse una opción: al guardar, si el texto no coincide con el nombre de la sucursal elegida, se descarta la elección (`.ts:182`) y no se envía | «Debes seleccionar una sucursal» (`stockPorSucursal.autocomplete.sucursal-required`), declarado en un `<mat-error>` dentro del campo (`.html:42-44`); ver nota 1 | Placeholder «Buscar sucursal...»; sin coincidencias: opción deshabilitada «Sin resultados»; si falla la carga: aviso «No se pudieron cargar tus sucursales» |
| Lote (`stockPorSucursal.fields.lote`) | `loteCtrl` (control suelto, sin validadores), 8/12 de fila | Texto con autocompletar. Se busca por **número de lote** escrito: `GET /api/v1/lotesinventario?numeroLote=…&pageNumber=1&pageSize=20`. Cada opción muestra el número de lote y debajo el nombre del producto; al elegir, el campo muestra «número — producto» (`loteDisplay`, `.ts:127-129`) | Sí | Igual que Sucursal (`.ts:183`) | «Debes seleccionar un lote» (`stockPorSucursal.autocomplete.lote-required`), en `<mat-error>` dentro del campo (`.html:77-79`); ver nota 1 | Placeholder «Buscar por número de lote...» (`stockPorSucursal.autocomplete.lote-placeholder`); sin coincidencias: opción deshabilitada «Sin resultados» |
| Cantidad Física (`stockPorSucursal.fields.cantidad`) | `cantidadFisica` (dentro de `form`), 4/12 de fila | `<input type="number" min="0">` | Sí | `Validators.required`, `Validators.min(0)` (`.ts:72-74`) | «La cantidad no puede ser negativa» (`stockPorSucursal.detail.cantidad-invalid`), también si el campo queda vacío | Valor inicial 0; el 0 es válido |

Nota 1. Los dos controles de autocompletar no tienen validadores, así que Angular Material nunca los marca como inválidos y **no pinta** el `<mat-error>` que está dentro del campo (en `@angular/material` `fesm2022/form-field2.mjs:847` el error solo se muestra si el control tiene `errorState`). Por lo tanto los textos «Debes seleccionar una sucursal» y «Debes seleccionar un lote» están declarados pero, por lectura del código, no llegan a verse. **No verificado en ejecución.**

Sucursales que ofrece: solo las que devuelve `GET /api/v1/sucursales/permitidas`, que el servidor calcula con el acceso del usuario (`GetSucursalesPermitidasQueryHandler.cs:13-22`): su sucursal base, las asignadas a su cuenta y a sus roles; `Admin`, todas (RN-STK-006 de 01c1). Lotes que ofrece: cualquier lote no eliminado de la empresa cuyo número contenga el texto, **sin filtrar por vigencia ni por producto** (el ingreso no comprueba que el lote esté vigente, RN-STK-007).

**Qué hace «Guardar» cuando no hay lote elegido — No verificado en ejecución.** El método `saveNew()` (`stock-detail.component.ts:179-202`) hace, en este orden:
1. `sucursalTouched` y `loteTouched` pasan a verdaderos (`:180-181`).
2. Si el texto de la sucursal no coincide con `selectedSucursal()?.nombre`, deja `selectedSucursal` en `null` (`:182`).
3. En la línea `:183` evalúa `this.loteCtrl.value !== this.loteDisplay(this.selectedLote()!)`. El operador `!` solo calla al compilador: `selectedLote()` vale `null` cuando no se eligió ningún lote, y `loteDisplay(lote)` lee `lote.numeroLoteMfr` (`:127-129`), así que la lectura lanza un `TypeError` («Cannot read properties of null (reading 'numeroLoteMfr')»).
4. La excepción sale del manejador del clic: el método termina en ese punto, no se ejecuta `this.form.markAllAsTouched()` ni la comprobación de la línea `:185`, **no se envía ninguna petición**, `saving` queda en falso y no se muestra aviso. El error solo queda en la consola del navegador (lo registra el `ErrorHandler` de Angular).

Efecto para el usuario según el código: al pulsar «Guardar» sin haber elegido un lote no ocurre nada visible (ni petición ni mensaje; los `<mat-error>` tampoco se pintan, nota 1). Si habías elegido un lote y luego cambiaste o borraste el texto, `selectedLote()` aún no es nulo en la línea `:183`, no hay excepción, la elección se descarta (`selectedLote.set(null)`) y el método vuelve en `:185` sin mensaje. Si falta solo la sucursal pero hay lote, tampoco hay mensaje. Ninguna de estas ramas se ejecutó; el componente no tiene spec.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie, derecha | Siempre en crear | Nunca | `cancelEdit()` → navega a `/stock-por-sucursal` |
| Guardar (`actions.save`); con spinner mientras guarda | Pie, derecha | Siempre en crear | `saving()` | `save()` → `saveNew()` (`.ts:179-202`): envía `{ sucursalId, loteId, cantidad }` a `POST /api/v1/lotesinventario/stock`; con éxito aviso y navega a `/stock-por-sucursal` |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/stock-por-sucursal` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Ingreso correcto | Éxito | «Stock registrado correctamente» (`stockPorSucursal.create.save-success`) | `stock-detail.component.ts:194` |
| Código `Concurrency.Conflict` (409) | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`, raíz) | `.ts:222-224` |
| Código `Inventario.Sucursal.NoPermitida` (403) | Error | «No tienes permiso para operar el stock de esta sucursal» (`stockPorSucursal.errors.sucursal-no-permitida`) | `.ts:225` |
| Cualquier otro error (400 de validador, 404 sucursal o lote, 409 `InventarioSucursal.RegistroConcurrente`, 500, sin conexión…) | Error | «Error al registrar el stock» (`stockPorSucursal.create.save-error`) | `.ts:199,226` |
| Falla la consulta de sugerencias de lote | — | Sin aviso (la suscripción no tiene manejador de error) | `.ts:100-108` |

El mapa es propio de la pantalla (`saveErrorKey` privado, `.ts:222-227`) y no usa la utilidad compartida: los códigos `Db.*` no se traducen aquí y caen en el mensaje genérico.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Escribir en «Sucursal» | `SucursalService.getPermitidas` | `GET /api/v1/sucursales/permitidas` | Solo autenticado |
| Escribir en «Lote» | `LoteService.search` (`lote.service.ts:18-29`) | `GET /api/v1/lotesinventario?pageNumber=1&pageSize=20&numeroLote=…` | `Inventarios:Leer` (`LotesInventarioController.cs:19`) |
| Guardar | `StockService.registrar` (`stock.service.ts:31-33`) | `POST /api/v1/lotesinventario/stock` (está en el controlador de lotes, no en el de stock) | `Inventarios:Actualizar` (`LotesInventarioController.cs:79`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- El resultado del ingreso depende de si ya existe la fila: si existe, suma; si no, la crea (RN-STK-008 y RN-STK-009). El aviso de éxito es el mismo en ambos casos.
- Backend: la sucursal debe existir (404 `Sucursal.NotFound`), el usuario debe tener acceso a ella (403 `Inventario.Sucursal.NoPermitida`) y el lote debe existir (404 `LoteInventario.NotFound`) (`RegisterInventarioCommandHandler.cs:19-35`); dos ingresos simultáneos que crean la misma fila reciben 409 `InventarioSucursal.RegistroConcurrente` (`:69-73`); la cantidad no puede ser negativa (`RegisterInventarioCommandValidator.cs:20-21`, código `RegisterInventarioCommand.Cantidad`).

**Observaciones**
- La cantidad admite decimales hasta 4 posiciones en la API (RN-STK-004 de 01c1); el campo numérico no fija `step`, así que cómo trata el navegador un decimal escrito no se probó.
- Sin spec del componente.

### 2.3 `/stock-por-sucursal/:id` — Detalle de Stock (ver y ajustar)

- **Componente:** `StockDetailComponent` (mismo componente, `.html:118-209`). Scope `stock-por-sucursal`.
- **Para qué sirve:** ver sucursal, producto, lote, caducidad y cantidad física de un registro de stock y, con permiso, **ajustar** la cantidad (fija el valor nuevo; no suma).
- **Cómo se llega:** enlace del nombre de la sucursal en la tabla de resultados (§2.1).
- **Permiso de ruta:** `Inventarios:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Stock por Sucursal»; migas «Inicio» / «Inventario» (sin enlace) / «Stock por Sucursal» (con enlace); botón volver a `/stock-por-sucursal`. La tarjeta en modo ver abre con «Detalle de Stock» (`stockPorSucursal.detail.title`) y «Datos del registro» (`generic-labels.detail-subtitle`).
- **Modos** (`ngOnInit`, `.ts:111-125`): con `id` → **ver** (formulario deshabilitado, carga con `GET`); «Editar» habilita solo el campo de cantidad. Si falta el parámetro `id` navega a `/stock-por-sucursal`. La tarjeta solo se pinta cuando la carga terminó y hay datos (`@if (!isNew() && !loading() && snapshot(); as s)`, `.html:119`).

| Aspecto | Ver (`/:id`) | Editar (tras «Editar») |
|---|---|---|
| Encabezado «Detalle de Stock» + «Datos del registro» + botón «Editar» | Sí (botón solo con `Inventarios:Actualizar`) | No |
| Sucursal, Producto, Número de Lote, Fecha de Caducidad | Solo lectura (entradas deshabilitadas) | Solo lectura |
| Cantidad Física | Deshabilitado | Editable |
| Botones | Editar | Cancelar, Guardar |

**Campos del formulario** (`.html:143-188`)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error | Editable |
|---|---|---|---|---|---|---|
| Sucursal (`stockPorSucursal.fields.sucursal`) | (valor de `snapshot`) | Entrada deshabilitada, 6/12 | — | — | — | No |
| Producto (`stockPorSucursal.fields.producto`) | (valor de `snapshot`) | Entrada deshabilitada, 6/12 | — | — | — | No |
| Número de Lote (`stockPorSucursal.fields.numeroLote`) | (valor de `snapshot`) | Entrada deshabilitada, 4/12 | — | — | — | No |
| Fecha de Caducidad (`stockPorSucursal.fields.caducidad`) | (valor de `snapshot`) | Entrada deshabilitada, `dd/MM/yyyy` (`.html:170`), 4/12 | — | — | — | No |
| Cantidad Física (`stockPorSucursal.fields.cantidad`) | `cantidadFisica` | `<input type="number" min="0">`, 4/12 | Sí | `Validators.required`, `Validators.min(0)` (`.ts:72-74`) | «La cantidad no puede ser negativa» (`stockPorSucursal.detail.cantidad-invalid`), también si el campo queda vacío | Solo en editar |

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Inventarios:Actualizar`, `.html:133`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Modo editar | Nunca | `cancelEdit()` (`.ts:161-169`): restaura la cantidad leída y vuelve a ver |
| Guardar (`actions.save`); con spinner | Pie, derecha | Modo editar | `saving()` | `save()` → `saveExisting()` (`.ts:204-220`): marca todo como tocado; inválido → no envía; envía `{ nuevaCantidad }` a `PUT /api/v1/stockporsucursal/:id/ajustar`; con éxito aviso y navega a `/stock-por-sucursal` (no vuelve al modo ver) |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/stock-por-sucursal` |

El texto `stockPorSucursal.detail.adjust` («Ajustar cantidad») existe pero no se usa en ninguna plantilla. No hay botón «Eliminar» en esta pantalla.

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Ajuste correcto | Éxito | «Cantidad ajustada correctamente» (`stockPorSucursal.detail.save-success`) | `stock-detail.component.ts:212` |
| Código `Concurrency.Conflict` (409) | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`) | `.ts:222-224` |
| Código `Inventario.Sucursal.NoPermitida` (403) | Error | «No tienes permiso para operar el stock de esta sucursal» (`stockPorSucursal.errors.sucursal-no-permitida`) | `.ts:225` |
| Cualquier otro error al ajustar (400 de validador, 404 `InventarioSucursal.NotFound`, 500, sin conexión…) | Error | «Error al ajustar la cantidad» (`stockPorSucursal.detail.save-error`) | `.ts:217,226` |
| Error al cargar el registro | Error | «Error al cargar el registro de stock» (`stockPorSucursal.detail.load-error`) | `.ts:151` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir `/stock-por-sucursal/:id` | `StockService.getById` (`stock.service.ts:27-29`) | `GET /api/v1/stockporsucursal/{id}` | `Inventarios:Leer` (`StockPorSucursalController.cs:22`) |
| Guardar el ajuste | `StockService.ajustar` (`:35-37`) | `PUT /api/v1/stockporsucursal/{id}/ajustar` con `{ nuevaCantidad }` | `Inventarios:Actualizar` (`StockPorSucursalController.cs:30`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Si falla la carga (por ejemplo el registro pertenece a una sucursal a la que el usuario no tiene acceso: el servidor responde 404 `InventarioSucursal.NotFound`, `GetStockPorSucursalByIdQueryHandler.cs:14-20,42-45`), `snapshot` queda vacío, la tarjeta no se pinta y solo se ve el aviso «Error al cargar el registro de stock».
- El ajuste exige además acceso a la sucursal del registro (403 si no, `AdjustStockCommandHandler.cs:21-24`) y genera un evento de auditoría (RN-STK-012 de 01c1).

**Observaciones**
- El mensaje «La cantidad no puede ser negativa» sale también cuando el campo está vacío (es el único `mat-error`).
- Sin spec del componente; `stock.service.spec.ts` (6 casos) cubre el servicio.
- Claves del scope sin uso: ver §9.

---

## 3. Pacientes

Carpeta `app/pages/pacientes`, scope i18n `pacientes` (`provideTranslocoScope('pacientes')` en los cuatro componentes). Rutas (`app/pages/pacientes/pacientes.routes.ts`, `loadChildren` en `app/app.routes.ts:149`): `''` → `PacientesSearchComponent`, `nueva` y `:id` → `PacientesDetailComponent`, `carga-masiva` → `PacientesCargaMasivaComponent`. El orden de las rutas pone `nueva` y `carga-masiva` antes de `:id`.

Tipos de documento (`app/shared/utils/tipo-documento.util.ts`): el SPA ofrece cuatro, guardados con el código del catálogo 06 de SUNAT: `1` «DNI», `4` «Carné de extranjería», `6` «RUC» y `7` «Pasaporte» (`tipos-documento.1`, `.4`, `.6`, `.7` en la raíz de `es.json`; existe además `tipos-documento.-` «Sin documento», que reconoce `tipoDocumentoKey` pero ninguna lista ofrece). Formato del número por tipo (`tipo-documento.util.ts:6-11`, el mismo que valida la API en `TipoDocumentoIdentidad.cs`):

| Tipo (texto, código) | Formato del número |
|---|---|
| DNI (`1`) | Exactamente 8 dígitos (`^\d{8}$`) |
| Carné de extranjería (`4`) | De 1 a 12 letras o números (`^[A-Za-z0-9]{1,12}$`) |
| RUC (`6`) | Exactamente 11 dígitos (`^\d{11}$`) |
| Pasaporte (`7`) | De 1 a 20 letras o números (`^[A-Za-z0-9]{1,20}$`) |

### 3.1 `/pacientes` — Pacientes (búsqueda)

- **Componente:** `PacientesSearchComponent` (`app/pages/pacientes/pacientes-search/pacientes-search.component.ts`, `.html`). Scope `pacientes`.
- **Para qué sirve:** buscar pacientes por tipo y número de documento, nombre y rango de fecha de nacimiento, abrirlos, exportar la lista y entrar a crear o a cargar pacientes masivamente.
- **Cómo se llega:** menú lateral, grupo «Pacientes & Atención» (`breadcrumbs.pacientesAtencion`) → «Pacientes» (`nav.pacientes`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:137-143`); botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar un paciente.
- **Permiso de ruta:** `Pacientes:Leer` (`route-permissions.ts:21`). Sin permiso: aviso «No tienes permisos para acceder a esta pantalla.» y redirección a `/dashboard`.
- **Migas y cabecera:** título «Pacientes» (`pacientes.title`); migas «Inicio» / «Pacientes & Atención» (`breadcrumbs.pacientes-atencion`, sin enlace) / «Pacientes» (activa); sin botón volver (`.html:3-6`, `.ts:58-62`). Tarjeta de filtros con título «Búsqueda de Pacientes» (`pacientes.search.title`) y subtítulo «Ingresa los criterios de búsqueda».
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:50-56`, `.html:37-95`)

| Etiqueta (clave) | Control | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Tipo de Documento (`pacientes.fields.tipoDocumento`) | `tipoDocumento` | Select (`mat-select`), 3/12 de fila | «Todos» (`pacientes.search.tipoDocumentoTodos`, valor vacío), «DNI», «Carné de extranjería», «RUC», «Pasaporte» | «Todos» (cadena vacía) |
| Documento de Identidad (`pacientes.fields.docIdentidadGub`) | `docIdentidadGub` | Texto con lupa; Enter busca, 3/12 | Texto libre; el servidor compara con «contiene» (`GetPacientesQueryHandler.cs:22-25`) | Vacío |
| Nombre y Apellidos (`pacientes.fields.nombreApellidos`) | `nombreApellidos` | Texto con lupa; Enter busca, 6/12 | Texto libre, «contiene» (`:27-30`) | Vacío |
| Fecha Nacimiento Desde (`pacientes.search.fechaNacimientoDesde`) | `fechaNacimientoDesde` | Fecha (`mat-datepicker`), 6/12 | Sin mínimo ni máximo; se envía como `yyyy-MM-dd` (`toDateOnly`); el servidor filtra `>=` | Vacío |
| Fecha Nacimiento Hasta (`pacientes.search.fechaNacimientoHasta`) | `fechaNacimientoHasta` | Fecha, 6/12 | Igual; el servidor filtra `<=` (inclusivo) | Vacío |

Los resultados se muestran solo tras pulsar «Buscar» (o al restaurar un estado guardado, §0). «Limpiar» vacía el formulario (`reset()`: el select queda sin valor), oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:112-119`). No hay validación cruzada entre «Desde» y «Hasta».

**Tabla de resultados** (`.html:121-181`, columnas `.ts:64`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Doc. Identidad (`pacientes.table.docIdentidadGub`) | `docIdentidadGub` | Texto | Sí: `[routerLink]="[row.id]"` → `/pacientes/:id` |
| Tipo Doc. (`pacientes.table.tipoDocumento`) | `tipoDocumento` | Texto del tipo (por ejemplo «DNI»); «—» si el tipo es nulo o desconocido (`.html:138-142`) | No |
| Nombre y Apellidos (`pacientes.table.nombreApellidos`) | `nombreApellidos` | Texto | No |
| Fecha Nacimiento (`pacientes.table.fechaNacimiento`) | `fechaNacimiento` | `dd/MM/yyyy` con el pipe `date` (`.html:158`) | No |
| Contacto Primario (`pacientes.table.contactoPrimario`) | `contactoPrimario` | Texto; «—» si es nulo (`.html:167`) | No |

Orden por defecto: lo fija el servidor, por nombre y apellidos y luego por identificador (`GetPacientesQueryHandler.cs:43`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:178`; no «Sin resultados»).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `Pacientes:Crear`, `.html:19`) | Nunca | `[routerLink]="['carga-masiva']"` → `/pacientes/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/pacientes/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/pacientes` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/pacientes/exportar` |
| Enlace con el documento | Primera columna | Con resultados | — | Navega a `/pacientes/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar pacientes» (`pacientes.search.search-error`) | `pacientes-search.component.ts:107` |
| Falla la exportación | Error | El mismo texto «Error al buscar pacientes»; usa `showServerError`, así que si la respuesta trae un `traceId` el aviso dura 10 s y añade «Referencia: …» ([19a §6.11](19a-acceso-layout-y-compartidos.md)) | `pacientes-search.component.ts:158` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `PacienteService.search` (`app/pages/pacientes/services/paciente.service.ts:19-39`) | `GET /api/v1/pacientes?pageNumber&pageSize[&tipoDocumento][&docIdentidadGub][&nombreApellidos][&fechaNacimientoDesde][&fechaNacimientoHasta]` | `Pacientes:Leer` (`PacientesController.cs:29`) |
| Exportar | `PacienteService.exportar` (`:57-59`) | `POST /api/v1/pacientes/exportar` (blob) | `Pacientes:Leer` (`PacientesController.cs:62`) |

**Exportación**
- Cuerpo (`.ts:131-149`): `headers` con 5 textos traducidos, en orden: «Doc. Identidad» (`table.docIdentidadGub`), «Tipo Doc.» (`table.tipoDocumento`), «Nombre y Apellidos» (`table.nombreApellidos`), «Fecha Nacimiento» (`table.fechaNacimiento`) y «Contacto Primario» (`table.contactoPrimario`); `dateFormat` = `dd/MM/yyyy` con idioma `es`, `MM/dd/yyyy` en otro caso; y los cinco filtros del formulario en ese momento. La API escribe el tipo con su abreviatura («DNI», «CE», «RUC», «PASAPORTE», `PacientesController.cs:88`).
- Archivo: «pacientes.xlsx» (`pacientes.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §3.3.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0.

**Observaciones:** la fila sin resultados dice «Resultados» y no «Sin resultados». La API devuelve `fechaNacimiento` como fecha sin hora (`DateOnly`, `GetPacientesQuery.cs`). No hay specs de los componentes de búsqueda ni de detalle (el único spec de la carpeta, `pacientes-detail.delete.spec.ts`, usa el helper común `describeDeleteInUse` y registra 0 casos `it(` propios).

### 3.2 `/pacientes/nueva` y `/pacientes/:id` — Pacientes (detalle)

- **Componente:** `PacientesDetailComponent` (`app/pages/pacientes/pacientes-detail/pacientes-detail.component.ts`, `.html`). Scope `pacientes`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un paciente, ver sus datos y, con permisos, editarlo o eliminarlo.
- **Cómo se llega:** `/pacientes/nueva`: botón «Nuevo» de la búsqueda. `/pacientes/:id`: enlace del documento en la tabla.
- **Permiso de ruta:** `/pacientes/nueva` → `Pacientes:Leer` + `Pacientes:Crear`; `/pacientes/:id` → `Pacientes:Leer`.
- **Migas y cabecera:** título «Pacientes»; migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Pacientes» (con enlace a `/pacientes`); botón volver a `/pacientes` (`.html:3-7`, `.ts:75-79`).
- **Modos** (`ngOnInit`, `.ts:81-92`): sin parámetro `id` → **crear**; con `id` → **ver** (formulario deshabilitado, carga con `GET`) y «Editar» habilita el formulario.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Pacientes» + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `Pacientes:Actualizar`) | No |
| Campos | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:66-73`, `.html:37-105`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Tipo de Documento (`pacientes.fields.tipoDocumento`) | `tipoDocumento` | `mat-select`, 3/12; opciones «DNI», «Carné de extranjería», «RUC», «Pasaporte» | Sí | `Validators.required` (`.ts:67`) | «Este campo es obligatorio» (`general.errors.required`, `.html:48`) | Sí / Sí |
| Documento de Identidad (`pacientes.fields.docIdentidadGub`) | `docIdentidadGub` | `matInput`, `maxlength="20"`, 3/12 | Sí | `Validators.required`, `Validators.maxLength(20)` y `documentoIdentidadValidator()` (`.ts:68`): el número debe cumplir el formato del tipo elegido (tabla de §3); si no hay tipo o el campo está vacío, el validador no actúa. Al cambiar el tipo se revalida el número (`.ts:82`) | Con error de formato: «El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números).» (`pacientes.detail.documento-formato`); en cualquier otro error: «Este campo es obligatorio» (`general.errors.required`) (`.html:62-68`) | Sí / Sí |
| Nombre y Apellidos (`pacientes.fields.nombreApellidos`) | `nombreApellidos` | `matInput`, `maxlength="200"`, 6/12 | Sí | `Validators.required`, `Validators.maxLength(200)` (`.ts:69`) | «Este campo es obligatorio» (`.html:79`) | Sí / Sí |
| Fecha de Nacimiento (`pacientes.fields.fechaNacimiento`) | `fechaNacimiento` | `mat-datepicker`, 6/12 | Sí | `Validators.required` (`.ts:71`); sin máximo, se puede elegir una fecha futura | «Este campo es obligatorio» (`.html:92`) | Sí / Sí |
| Contacto Primario (`pacientes.fields.contactoPrimario`) | `contactoPrimario` | `matInput`, `maxlength="150"`, 6/12 | No | `Validators.maxLength(150)` (`.ts:72`); sin patrón | Sin `mat-error` | Sí / Sí |

Sin ayuda de campo (las claves `pacientes.help.*` no se usan). Un paciente antiguo puede venir sin tipo de documento: el select queda vacío y es obligatorio elegirlo antes de guardar (comentario en `.ts:65`). Al guardar se envía `{ tipoDocumento, docIdentidadGub, nombreApellidos, fechaNacimiento (yyyy-MM-dd), contactoPrimario }`, con el contacto omitido si es nulo (`.ts:142-150`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Pacientes:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:120-134`): en crear navega a `/pacientes`; en editar restaura los datos leídos y vuelve a ver |
| Guardar (`actions.save`); con spinner | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:136-177`). Inválido → marca todo como tocado y no envía. Crear → `POST /api/v1/pacientes`, aviso y navega a `/pacientes`. Editar → `PUT /api/v1/pacientes/:id`, aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de un paciente existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `Pacientes:Eliminar`, `.html:126`) | `deleting()` | `confirmDelete()` (`.ts:179-190`) → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/pacientes` |

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar»: ancho 400 px, título «Confirmar eliminación» (`pacientes.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este paciente?» (`pacientes.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/pacientes/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Paciente guardado correctamente» (`pacientes.detail.save-success`) | `pacientes-detail.component.ts:154,165` |
| Código `Paciente.DocIdentidadGub.Duplicado` (409) | Error | «Ya existe un paciente con ese documento de identidad.» (`pacientes.detail.duplicate-documento`) | `.ts:20,159,173` |
| Códigos `PacienteCliente.DocIdentidadGub.FormatoInvalido`, `CreatePacienteCommand.DocIdentidadGub`, `UpdatePacienteCommand.DocIdentidadGub` (400) | Error | «El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números).» (`pacientes.detail.documento-formato`) | `.ts:21-23` |
| Códigos `PacienteCliente.TipoDocumento`, `CreatePacienteCommand.TipoDocumento`, `UpdatePacienteCommand.TipoDocumento` (400) | Error | «Selecciona un tipo de documento válido.» (`pacientes.detail.tipo-documento-requerido`) | `.ts:24-26` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en [19c §0](19c-catalogos-1.md#0-comportamientos-comunes-a-las-pantallas-de-este-documento) | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar | Error | «Error al guardar el paciente» (`pacientes.detail.save-error`) | `.ts:159,173` |
| Eliminación correcta | Éxito | «Paciente eliminado correctamente» (`pacientes.detail.delete-success`); navega a `/pacientes` | `.ts:196-197` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» (`pacientes.detail.delete-in-use`) | `.ts:199-204` |
| Eliminar: cualquier otro error | Error | «Error al eliminar el paciente» (`pacientes.detail.delete-error`) | `.ts:199-204` |
| Error al cargar | Error | «Error al cargar el paciente» (`pacientes.detail.load-error`) | `.ts:110` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir y tras editar | `PacienteService.getById` (`paciente.service.ts:41-43`) | `GET /api/v1/pacientes/{id}` | `Pacientes:Leer` (`PacientesController.cs:38`) |
| Crear | `create` (`:45-47`) | `POST /api/v1/pacientes` | `Pacientes:Crear` (`:19`) |
| Editar | `update` (`:49-51`) | `PUT /api/v1/pacientes/{id}` | `Pacientes:Actualizar` (`:46`) |
| Eliminar | `delete` (`:53-55`) | `DELETE /api/v1/pacientes/{id}` | `Pacientes:Eliminar` (`:55`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga muestra un spinner en lugar de la tarjeta (`.html:9-13`).
- La API rechaza eliminar un paciente referenciado por recetas, ventas, citas de servicios o inmunizaciones no eliminadas con 409 `Paciente.InUse` (`DeletePacienteCommandHandler.cs:24-29,37-41`, commit `27597cd`). Un paciente inexistente responde 404 con el código `NotFound` (`:21`), que la pantalla trata con el mensaje genérico.
- Si falla la carga del registro, la ficha queda vacía y deshabilitada y, con `Pacientes:Actualizar`, sigue visible «Editar».

**Observaciones** (discrepancias entre el formulario y la API, sacadas de leer el código; no se ejecutó)
- **Contacto Primario:** el formulario acepta hasta 150 caracteres (`pacientes-detail.component.ts:72`, `maxlength="150"`) pero la API rechaza más de 100 (`CreatePacienteCommandValidator.cs:18`, `UpdatePacienteCommandValidator.cs:18`, `PacienteCliente.cs:109`). Con 101 a 150 caracteres el usuario ve «Error al guardar el paciente» (el código `CreatePacienteCommand.ContactoPrimario` no está mapeado).
- **Fecha de Nacimiento futura:** el selector no limita la fecha y el formulario no valida que no sea futura, pero la API la rechaza con el código `PacienteCliente.FechaNacimiento` (`PacienteCliente.cs:104-107`), que no está mapeado: el usuario ve «Error al guardar el paciente». Las claves `pacientes.form.futureDateError` y `pacientes.form.phoneInvalid` existen pero ninguna plantilla las usa.
- **Documento de Identidad:** el formulario limita a 20 caracteres y la API a 50; el formato por tipo es el mismo en ambos lados. Un documento de solo espacios cumple `Validators.required` de Angular pero falla el formato (el validador del número lo rechaza para DNI, RUC, CE y pasaporte).
- **Nombre y Apellidos de solo espacios:** pasa `Validators.required` y la API lo rechaza (`NotEmpty`) con el código `CreatePacienteCommand.NombreApellidos`, sin mapa: «Error al guardar el paciente».
- Con 409 de `Paciente.DocIdentidadGub.Duplicado` el aviso es específico; la API lo devuelve también al editar si otro paciente no eliminado ya usa ese documento (`UpdatePacienteCommandHandler.cs:20-25`). Reglas PAC en [01b](../01-reglas-de-negocio/01b-catalogos.md).
- Claves del scope sin uso: ver §9.

### 3.3 `/pacientes/carga-masiva` — Carga Masiva (pacientes)

- **Componente:** `PacientesCargaMasivaComponent` (`app/pages/pacientes/pacientes-carga-masiva/pacientes-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `pacientes` (no aporta textos).
- **Para qué sirve:** cargar varios pacientes desde un Excel; el flujo común está en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/pacientes` (§3.1).
- **Permiso de ruta:** `Pacientes:Leer` + `Pacientes:Crear` (segmento `carga-masiva`).
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Pacientes» (enlace a `/pacientes`) / «Carga Masiva» (activa, `breadcrumbs.carga-masiva`); botón volver a `/pacientes`.

**Configuración que pasa** (`.ts:16-35`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/pacientes/plantilla` — `Pacientes:Crear` (`PacientesController.cs:97`); se baja como `plantilla-pacientes.xlsx` |
| `previewUrl` | `POST {api}/api/v1/pacientes/previsualizar` — `Pacientes:Crear` (`:106`) |
| `saveUrl` | `POST {api}/api/v1/pacientes/carga-masiva` — `Pacientes:Crear` (`:124`) |
| `backRoute` | `/pacientes` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz) | `tipoDocumento` → `fields.tipoDocumento` (obligatoria); `docIdentidadGub` → «Doc. Identidad» (`fields.docIdentidadGub`, obligatoria); `nombreApellidos` → «Nombre y Apellidos» (`fields.nombreApellidos`, obligatoria); `fechaNacimiento` → «Fecha de Nacimiento» (`fields.fechaNacimiento`, obligatoria); `contactoPrimario` → «Contacto Primario» (`fields.contactoPrimario`, no obligatoria). Después, la columna de errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Las claves de datos coinciden con `PacienteImportRow(TipoDocumento, DocIdentidadGub, NombreApellidos, FechaNacimiento, ContactoPrimario)` (`CargaMasivaPacientesCommand.cs:5`). Mensaje de validación propio del tipo de documento: «'{{field}}' no tiene el formato de su tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números)» (`carga-masiva.validation.invalid-document-number`).

**Observaciones:** **clave inexistente:** `fields.tipoDocumento` no existe en la raíz de `es.json` (la raíz tiene `fields.docIdentidadGub`, `fields.nombreApellidos`, `fields.fechaNacimiento` y `fields.contactoPrimario`, pero no `fields.tipoDocumento`; la que existe es `pacientes.fields.tipoDocumento`, que la carga masiva no consulta). Por lectura del código, la primera columna de la vista previa muestra el texto crudo `fields.tipoDocumento` en lugar de «Tipo de Documento». No verificado en ejecución. Además la etiqueta de la columna del documento en la carga masiva es «Doc. Identidad» (raíz), mientras que en el formulario del detalle es «Documento de Identidad».

---

## 4. Profesionales de salud

Carpeta `app/pages/profesionales-salud`, scope i18n `profesionalesSalud` (`provideTranslocoScope('profesionalesSalud')` en los cuatro componentes; archivo `public/i18n/profesionalesSalud/es.json`). Rutas (`app/pages/profesionales-salud/profesionales-salud.routes.ts`, `loadChildren` en `app/app.routes.ts:154`): `''` → `ProfesionalesSaludSearchComponent`, `nueva` y `:id` → `ProfesionalesSaludDetailComponent`, `carga-masiva` → `ProfesionalesSaludCargaMasivaComponent`.

**Qué datos tiene un profesional de salud en SOFIA.** Solo tres: número de registro, nombre del prescriptor y dirección de la clínica (`app/pages/profesionales-salud/models/profesional-salud.model.ts:8-13`; entidad `sofia-apis/src/SOFIA.Domain/Entities/ProfesionalSalud.cs`). **No existen** campos de colegiatura aparte del número de registro, especialidad ni tipo de profesional: el «Número de Registro» es el identificador del profesional (colegiatura) y no hay lista de especialidades ni de tipos.

### 4.1 `/profesionales-salud` — Profesionales de Salud (búsqueda)

- **Componente:** `ProfesionalesSaludSearchComponent` (`app/pages/profesionales-salud/profesionales-salud-search/profesionales-salud-search.component.ts`, `.html`). Scope `profesionalesSalud`.
- **Para qué sirve:** buscar profesionales por número de registro y nombre, abrirlos, exportar la lista y entrar a crear o a cargar profesionales masivamente.
- **Cómo se llega:** menú lateral, grupo «Pacientes & Atención» (`breadcrumbs.pacientesAtencion`) → «Profesionales de Salud» (`nav.profesionalesSalud`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:144-148`); botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar un profesional.
- **Permiso de ruta:** `ProfesionalesSalud:Leer` (`route-permissions.ts:22`). Sin permiso: aviso «No tienes permisos para acceder a esta pantalla.» y redirección a `/dashboard`.
- **Migas y cabecera:** título «Profesionales de Salud» (`profesionalesSalud.title`); migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Profesionales de Salud» (`breadcrumbs.profesionales-salud`, activa); sin botón volver (`.html:4-7`, `.ts:46-50`). Tarjeta de filtros con título «Búsqueda de Profesionales de Salud» (`profesionalesSalud.search.title`) y subtítulo «Ingresa los criterios de búsqueda».
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:44`, `.html:38-59`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Número de Registro (`profesionalesSalud.fields.numeroRegistro`) | `numeroRegistro` | Texto con lupa; Enter busca | Texto libre; el servidor compara con «contiene» (`GetProfesionalesSaludQueryHandler.cs:19`) | Vacío |
| Nombre del Prescriptor (`profesionalesSalud.fields.nombrePrescriptor`) | `nombrePrescriptor` | Texto con lupa; Enter busca | Texto libre, «contiene» (`:24`) | Vacío |

Los resultados se muestran solo tras pulsar «Buscar» (o al restaurar un estado guardado, §0). «Limpiar» vacía el formulario, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:97-104`).

**Tabla de resultados** (`.html:85-125`, columnas `.ts:52`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Número de Registro (`profesionalesSalud.table.numeroRegistro`) | `numeroRegistro` | Texto | Sí: `[routerLink]="[row.id]"` → `/profesionales-salud/:id` |
| Nombre del Prescriptor (`profesionalesSalud.table.nombrePrescriptor`) | `nombrePrescriptor` | Texto | No |
| Dirección de Clínica (`profesionalesSalud.table.direccionClinica`) | `direccionClinica` | Texto; «—» si es nulo (`.html:111`) | No |

Orden por defecto: lo fija el servidor, por nombre del prescriptor y luego por identificador (`GetProfesionalesSaludQueryHandler.cs:28`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:122`; no «Sin resultados»).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `ProfesionalesSalud:Crear`, `.html:20`) | Nunca | `[routerLink]="['carga-masiva']"` → `/profesionales-salud/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:28`) | Nunca | `goToNew()` → `/profesionales-salud/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/profesionalessalud` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/profesionalessalud/exportar` |
| Enlace con el número de registro | Primera columna | Con resultados | — | Navega a `/profesionales-salud/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar profesionales de salud» (`profesionalesSalud.search.search-error`) | `profesionales-salud-search.component.ts:92` |
| Falla la exportación | Error | El mismo texto; usa `showServerError`, así que con `traceId` en la respuesta dura 10 s y añade «Referencia: …» | `profesionales-salud-search.component.ts:137` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `ProfesionalSaludService.search` (`app/pages/profesionales-salud/services/profesional-salud.service.ts:19-30`) | `GET /api/v1/profesionalessalud?pageNumber&pageSize[&numeroRegistro][&nombrePrescriptor]` | `ProfesionalesSalud:Leer` (`ProfesionalesSaludController.cs:18`) |
| Exportar | `ProfesionalSaludService.exportar` (`:48-50`) | `POST /api/v1/profesionalessalud/exportar` (blob) | `ProfesionalesSalud:Leer` (`ProfesionalesSaludController.cs:97`) |

**Exportación**
- Cuerpo (`.ts:116-128`): `headers` con 3 textos traducidos, en orden: «Número de Registro» (`table.numeroRegistro`), «Nombre del Prescriptor» (`table.nombrePrescriptor`) y «Dirección de Clínica» (`table.direccionClinica`); más los dos filtros del formulario en ese momento. No envía formato de fecha (no hay fechas).
- Archivo: «profesionales-salud.xlsx» (`profesionalesSalud.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §4.3.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0.

**Observaciones:** la fila sin resultados dice «Resultados». No hay specs de búsqueda ni de detalle (`profesionales-salud-detail.delete.spec.ts` usa el helper `describeDeleteInUse` y registra 0 casos `it(` propios).

### 4.2 `/profesionales-salud/nueva` y `/profesionales-salud/:id` — Profesionales de Salud (detalle)

- **Componente:** `ProfesionalesSaludDetailComponent` (`app/pages/profesionales-salud/profesionales-salud-detail/profesionales-salud-detail.component.ts`, `.html`). Scope `profesionalesSalud`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un profesional de salud, ver sus datos y, con permisos, editarlo o eliminarlo.
- **Cómo se llega:** `/profesionales-salud/nueva`: botón «Nuevo» de la búsqueda. `/profesionales-salud/:id`: enlace del número de registro en la tabla.
- **Permiso de ruta:** `/profesionales-salud/nueva` → `ProfesionalesSalud:Leer` + `ProfesionalesSalud:Crear`; `/profesionales-salud/:id` → `ProfesionalesSalud:Leer`.
- **Migas y cabecera:** título «Profesionales de Salud»; migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Profesionales de Salud» (con enlace a `/profesionales-salud`); botón volver a `/profesionales-salud` (`.html:4-8`, `.ts:60-64`). Las claves `profesionalesSalud.detail.title-new` y `title-edit` existen pero no se usan.
- **Modos** (`ngOnInit`, `.ts:66-76`): sin parámetro `id` → **crear**; con `id` → **ver** (formulario deshabilitado, carga con `GET`) y «Editar» habilita el formulario.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Profesionales de Salud» + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `ProfesionalesSalud:Actualizar`) | No |
| Campos | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:54-58`, `.html:38-67`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Número de Registro (`profesionalesSalud.fields.numeroRegistro`) | `numeroRegistro` | `matInput`, `maxlength="50"`, 4/12 | Sí | `Validators.required`, `Validators.maxLength(50)` (`.ts:55`); sin patrón | «Este campo es obligatorio» (`general.errors.required`, `.html:43`) | Sí / Sí |
| Nombre del Prescriptor (`profesionalesSalud.fields.nombrePrescriptor`) | `nombrePrescriptor` | `matInput`, `maxlength="200"`, 8/12 | Sí | `Validators.required`, `Validators.maxLength(200)` (`.ts:56`) | «Este campo es obligatorio» (`.html:54`) | Sí / Sí |
| Dirección de Clínica (`profesionalesSalud.fields.direccionClinica`) | `direccionClinica` | `matInput`, `maxlength="300"`, fila completa | No | `Validators.maxLength(300)` (`.ts:57`) | Sin `mat-error` | Sí / Sí |

Sin ayuda ni placeholder. Al guardar se envía `{ numeroRegistro, nombrePrescriptor, direccionClinica }`, con la dirección omitida si es nula (`.ts:122-127`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `ProfesionalesSalud:Actualizar`, `.html:28`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:102-114`): en crear navega a `/profesionales-salud`; en editar restaura los datos leídos y vuelve a ver |
| Guardar (`actions.save`); con spinner | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:116-154`). Inválido → marca todo como tocado y no envía. Crear → `POST /api/v1/profesionalessalud`, aviso y navega a `/profesionales-salud`. Editar → `PUT /api/v1/profesionalessalud/:id`, aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de un profesional existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `ProfesionalesSalud:Eliminar`, `.html:88`) | `deleting()` | `confirmDelete()` (`.ts:156-167`) → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/profesionales-salud` |

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar»: ancho 400 px, título «Confirmar eliminación» (`profesionalesSalud.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este profesional de salud?» (`profesionalesSalud.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/profesionalessalud/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Profesional de salud guardado correctamente» (`profesionalesSalud.detail.save-success`) | `profesionales-salud-detail.component.ts:131,142` |
| Código `ProfesionalSalud.NumeroRegistro.Duplicado` (409) | Error | «Ya existe un profesional de salud con ese número de registro.» (`profesionalesSalud.detail.duplicate-registro`) | `.ts:17-19,136,150` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en [19c §0](19c-catalogos-1.md#0-comportamientos-comunes-a-las-pantallas-de-este-documento) | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar | Error | «Error al guardar el profesional de salud» (`profesionalesSalud.detail.save-error`) | `.ts:136,150` |
| Eliminación correcta | Éxito | «Profesional de salud eliminado correctamente» (`profesionalesSalud.detail.delete-success`); navega a `/profesionales-salud` | `.ts:173-174` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No puedes eliminar este profesional de salud porque tiene recetas asociadas.» (`profesionalesSalud.detail.delete-in-use`) | `.ts:176-182` |
| Eliminar: cualquier otro error | Error | «Error al eliminar el profesional de salud» (`profesionalesSalud.detail.delete-error`) | `.ts:176-182` |
| Error al cargar | Error | «Error al cargar el profesional de salud» (`profesionalesSalud.detail.load-error`) | `.ts:92` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir y tras editar | `ProfesionalSaludService.getById` (`profesional-salud.service.ts:32-34`) | `GET /api/v1/profesionalessalud/{id}` | `ProfesionalesSalud:Leer` (`ProfesionalesSaludController.cs:26`) |
| Crear | `create` (`:36-38`) | `POST /api/v1/profesionalessalud` | `ProfesionalesSalud:Crear` (`:34`) |
| Editar | `update` (`:40-42`) | `PUT /api/v1/profesionalessalud/{id}` | `ProfesionalesSalud:Actualizar` (`:44`) |
| Eliminar | `delete` (`:44-46`) | `DELETE /api/v1/profesionalessalud/{id}` | `ProfesionalesSalud:Eliminar` (`:53`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga muestra un spinner en lugar de la tarjeta (`.html:10-13`).
- La API rechaza eliminar un profesional referenciado por alguna receta no eliminada con 409 `ProfesionalSalud.InUse` (`DeleteProfesionalSaludCommandHandler.cs:20-27`, commit `27597cd`). Un profesional inexistente responde 404 `ProfesionalSalud.NotFound` (`:15-18`), que la pantalla muestra con el mensaje genérico.
- Si falla la carga del registro, la ficha queda vacía y deshabilitada y, con `ProfesionalesSalud:Actualizar`, sigue visible «Editar».

**Observaciones** (discrepancias entre el formulario y la API, sacadas de leer el código; no se ejecutó)
- **Nombre del Prescriptor:** el formulario acepta hasta 200 caracteres pero la API rechaza más de 150 (`CreateProfesionalSaludCommandValidator.cs:17-19`, `ProfesionalSalud.cs:33-36`). Con 151 a 200 caracteres el usuario ve «Error al guardar el profesional de salud» (código `CreateProfesionalSaludCommand.NombrePrescriptor` o `ProfesionalSalud.NombrePrescriptor`, sin mapa).
- **Dirección de Clínica:** el formulario acepta hasta 300 y la API rechaza más de 255 (`CreateProfesionalSaludCommandValidator.cs:21-22`, `ProfesionalSalud.cs:38-41`); mismo efecto, mensaje genérico.
- **Número de Registro:** 50 en ambos lados. Un valor de solo espacios pasa `Validators.required` y la API lo rechaza (`NotEmpty`/entidad) con mensaje genérico; lo mismo para el nombre.
- Al editar, la API no tiene validador de comando (no existe `UpdateProfesionalSaludCommandValidator`); la validación la hace la entidad con códigos `ProfesionalSalud.<Propiedad>` (`ProfesionalSalud.cs:58-78`).
- El 409 de duplicado se da también al editar si otro profesional no eliminado ya usa el número (`UpdateProfesionalSaludCommandHandler.cs:20-25`). Reglas PRO en [01b](../01-reglas-de-negocio/01b-catalogos.md).
- Claves del scope sin uso: ver §9.

### 4.3 `/profesionales-salud/carga-masiva` — Carga Masiva (profesionales de salud)

- **Componente:** `ProfesionalesSaludCargaMasivaComponent` (`app/pages/profesionales-salud/profesionales-salud-carga-masiva/profesionales-salud-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `profesionalesSalud` (no aporta textos).
- **Para qué sirve:** cargar varios profesionales desde un Excel; el flujo común está en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/profesionales-salud` (§4.1).
- **Permiso de ruta:** `ProfesionalesSalud:Leer` + `ProfesionalesSalud:Crear` (segmento `carga-masiva`).
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Profesionales de Salud» (enlace a `/profesionales-salud`) / «Carga Masiva» (activa); botón volver a `/profesionales-salud`.

**Configuración que pasa** (`.ts:16-33`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/profesionalessalud/plantilla` — `ProfesionalesSalud:Crear` (`ProfesionalesSaludController.cs:61`); se baja como `plantilla-profesionales-salud.xlsx` |
| `previewUrl` | `POST {api}/api/v1/profesionalessalud/previsualizar` — `ProfesionalesSalud:Crear` (`:70`) |
| `saveUrl` | `POST {api}/api/v1/profesionalessalud/carga-masiva` — `ProfesionalesSalud:Crear` (`:88`) |
| `backRoute` | `/profesionales-salud` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz) | `numeroRegistro` → «Número de Registro» (`fields.numeroRegistro`, obligatoria); `nombrePrescriptor` → «Nombre del Prescriptor» (`fields.nombrePrescriptor`, obligatoria); `direccionClinica` → «Dirección Clínica» (`fields.direccionClinica`, **no** obligatoria). Después, la columna de errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Las claves de datos coinciden con `ProfesionalSaludImportRow(NumeroRegistro, NombrePrescriptor, DireccionClinica)` (`CargaMasivaProfesionalesSaludCommand.cs:5`).

**Observaciones:** la etiqueta de la tercera columna en la carga masiva es «Dirección Clínica» (raíz `fields.direccionClinica`), distinta de «Dirección de Clínica» del formulario y de la tabla (scope `profesionalesSalud`).

---

## 5. Recetas médicas

Carpeta `app/pages/recetas-medicas`, scope i18n `recetas-medicas` (`provideTranslocoScope('recetas-medicas')` en los tres componentes). **Las plantillas leen el scope con el alias en camelCase** (`*transloco="let t; scope: 'recetas-medicas'; read: 'recetasMedicas'"`) y los `.ts` traducen con el prefijo `recetasMedicas.`; el archivo de textos es `public/i18n/recetas-medicas/es.json` y las claves de abajo llevan el prefijo `recetasMedicas.`. Rutas (`app/pages/recetas-medicas/recetas-medicas.routes.ts:6-11`, `loadChildren` en `app/app.routes.ts:159`; los componentes se importan de forma directa, sin carga diferida): `''` → `RecetasSearchComponent`, `escanear` → `RecetasScannerComponent`, `nueva` (con `data: { isNew: true }`) y `:id` → `RecetasDetailComponent`. `escanear` se declara antes de `:id`, así que no se confunde con un identificador (`recetas-medicas.routes.spec.ts`, 5 casos). No hay carga masiva.

**Qué es una receta en SOFIA.** Una receta es solo su **cabecera**: paciente, médico, fecha de expedición, repeticiones máximas, número de receta e indicaciones de uso (`app/pages/recetas-medicas/models/receta-medica.model.ts:20-30`; entidad `sofia-apis/src/SOFIA.Domain/Entities/RecetaMedica.cs:12-19`). **No existen líneas de medicamentos** (ni pantalla ni modelo para agregarlas o quitarlas) ni campo de vigencia: el sistema no vence recetas por fecha (RN-REC-007 de 01c1). Las «repeticiones máximas» son las veces adicionales que se puede dispensar; el control de dispensaciones se aplica al vender, no en esta pantalla (RN-REC-031 de 01c1).

### 5.1 `/recetas-medicas` — Recetas Médicas (búsqueda)

- **Componente:** `RecetasSearchComponent` (`app/pages/recetas-medicas/recetas-search/recetas-search.component.ts`, `.html`). Scope `recetas-medicas`.
- **Para qué sirve:** buscar recetas por paciente, médico y rango de fechas de expedición, abrirlas, exportar la lista y entrar a crear una receta o a escanearla.
- **Cómo se llega:** menú lateral, grupo «Pacientes & Atención» (`breadcrumbs.pacientesAtencion`) → «Recetas Médicas» (`nav.recetasMedicas`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:149-153`); botón volver y migas del detalle y del escáner; redirección tras crear o eliminar una receta.
- **Permiso de ruta:** `Recetas:Leer` (`route-permissions.ts:23`). Sin permiso: aviso «No tienes permisos para acceder a esta pantalla.» y redirección a `/dashboard`.
- **Migas y cabecera:** título «Recetas Médicas» (`recetasMedicas.title`); migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Recetas Médicas» (`breadcrumbs.recetasMedicas`, sin enlace ni marca de activa); sin botón volver (`.html:4-7`, `.ts:60-64`). Tarjeta de filtros con título «Búsqueda de Recetas Médicas» (`recetasMedicas.search.title`) y subtítulo «Ingresa los criterios de búsqueda».
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:53-58`, `.html:38-128`)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Paciente (`recetasMedicas.fields.paciente`) | `pacienteNombre` | Texto con autocompletar y lupa, 6/12. Placeholder «Buscar paciente...» (`recetasMedicas.autocomplete.paciente-placeholder`) | Con espera de 300 ms consulta pacientes por nombre (20 como máximo); cada opción muestra el nombre y «— {documento}». Sin coincidencias: opción deshabilitada «Sin resultados». **Solo filtra si eliges una opción de la lista**: la búsqueda envía el identificador del paciente elegido, no el texto (`.ts:125`); al escribir de nuevo se descarta la elección (`.ts:76`). Enter no busca | Vacío |
| Médico (`recetasMedicas.fields.medico`) | `medicoNombre` | Texto con autocompletar y lupa, 6/12. Placeholder «Buscar médico...» (`recetasMedicas.autocomplete.medico-placeholder`) | Igual con profesionales de salud por nombre del prescriptor; cada opción muestra el nombre y «— {número de registro}». Solo filtra si eliges una opción | Vacío |
| Fecha desde (`generic-labels.date-from`, raíz) | `fechaInicio` | Fecha (`mat-datepicker`), 4/12 | Sin mínimo ni máximo. Viaja como instante (`Date.toISOString()` de la medianoche local); el servidor usa la parte de fecha y filtra `>=` (`GetRecetasQueryHandler.cs:25-28`) | Vacío |
| Fecha hasta (`generic-labels.date-to`, raíz) | `fechaFin` | Fecha, 4/12 | Igual; filtra `<=` (inclusivo, `:30-33`) | Vacío |

Los resultados se muestran solo tras pulsar «Buscar» (o al restaurar un estado guardado, §0; el estado incluye los identificadores elegidos, `.ts:98-108,140-142`). «Limpiar» (`actions.clear`) vacía el formulario y las dos elecciones, vacía las sugerencias, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:153-164`). No hay validación cruzada entre las fechas. La zona horaria con la que el servidor interpreta el instante enviado no se probó; en Perú la medianoche local y su fecha UTC coinciden.

**Tabla de resultados** (`.html:156-208`, columnas `.ts:39`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Paciente (`recetasMedicas.table.paciente`) | `clienteNombre` | Texto | Sí: `[routerLink]="[row.id]"` → `/recetas-medicas/:id` |
| Médico (`recetasMedicas.table.medico`) | `medicoNombre` | Texto | No |
| Fecha de Expedición (`recetasMedicas.table.fechaExpedicion`) | `fechaExpedicion` | `dd/MM/yyyy` (`.html:178`) | No |
| Repeticiones (`recetasMedicas.table.repeticiones`) | `repeticionesMax` | Número | No |
| Indicaciones (`recetasMedicas.table.indicaciones`) | `indicacionesUso` | Texto; «—» si es nulo o vacío (`.html:194`) | No |

El número de receta no se muestra en la tabla. Orden por defecto: lo fija el servidor, por fecha de expedición descendente (`GetRecetasQueryHandler.cs:36`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Sin resultados» (`generic-labels.no-results`, `.html:205`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Escanear receta (`recetasMedicas.search.btn-scan`), icono `document_scanner` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Analizar'"` (= `Recetas:Analizar`, `.html:21`) | Nunca | `goToScanner()` → `/recetas-medicas/escanear` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (= `Recetas:Crear`, `.html:28`) | Nunca | `goToNew()` → `/recetas-medicas/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `limpiar()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `buscar()` → `GET /api/v1/recetas` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/recetas/exportar` |
| Enlace con el nombre del paciente | Primera columna | Con resultados | — | Navega a `/recetas-medicas/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar recetas médicas» (`recetasMedicas.search.search-error`) | `recetas-search.component.ts:148` |
| Falla la exportación (cualquier error) | Error | El mismo texto, sin referencia de seguimiento | `recetas-search.component.ts:208` |
| Falla la consulta de sugerencias de paciente o de médico | — | Sin aviso: las suscripciones no tienen manejador de error | `recetas-search.component.ts:71-95` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `RecetaMedicaService.search` (`app/pages/recetas-medicas/services/receta-medica.service.ts:21-30`) | `GET /api/v1/recetas?pageNumber&pageSize[&clienteId][&medicoId][&fechaInicio][&fechaFin]` | `Recetas:Leer` (`RecetasController.cs:24`) |
| Escribir en «Paciente» | `PacienteService.search` (`paciente.service.ts:19-39`) | `GET /api/v1/pacientes?pageNumber=1&pageSize=20&nombreApellidos=…` | `Pacientes:Leer` (`PacientesController.cs:29`) |
| Escribir en «Médico» | `ProfesionalSaludService.search` (`profesional-salud.service.ts:19-30`) | `GET /api/v1/profesionalessalud?pageNumber=1&pageSize=20&nombrePrescriptor=…` | `ProfesionalesSalud:Leer` (`ProfesionalesSaludController.cs:18`) |
| Exportar | `RecetaMedicaService.exportar` (`:48-50`) | `POST /api/v1/recetas/exportar` (blob) | `Recetas:Leer` (`RecetasController.cs:82`) |

Un usuario con `Recetas:Leer` pero sin `Pacientes:Leer` o sin `ProfesionalesSalud:Leer` no obtiene sugerencias en esos dos filtros (la API responde 403 y no hay manejador). No verificado en ejecución.

**Exportación**
- Cuerpo (`.ts:180-199`): `headers` con 5 textos traducidos, en orden: «Paciente» (`table.paciente`), «Médico» (`table.medico`), «Fecha de Expedición» (`table.fechaExpedicion`), «Repeticiones» (`table.repeticiones`) e «Indicaciones» (`table.indicaciones`); `dateFormat` = `dd/MM/yyyy` con idioma `es`, `MM/dd/yyyy` en otro caso; `clienteId` y `medicoId` elegidos y las dos fechas, cada uno solo si tiene valor (los del formulario en ese momento).
- Archivo: «recetas-medicas.xlsx» (`recetasMedicas.search.export-filename`). La API no exporta el número de receta (RN-REC-017).
- Carga masiva: no existe.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0. Al elegir una opción de paciente o de médico se vacía su lista de sugerencias (`.ts:113,119`).

**Observaciones:** el botón de la fila vacía y las etiquetas «Fecha desde» y «Fecha hasta» vienen de la raíz (`generic-labels`). Specs `recetas-search.component.spec.ts` (7 casos: visibilidad de «Escanear receta» según `Recetas:Analizar` y para `Admin`, tabla oculta al inicio, `buscar()` envía paciente y fechas, `limpiar()`, `exportar()` envía las fechas).

### 5.2 `/recetas-medicas/nueva` y `/recetas-medicas/:id` — Recetas Médicas (detalle)

- **Componente:** `RecetasDetailComponent` (`app/pages/recetas-medicas/recetas-detail/recetas-detail.component.ts`, `.html`). Scope `recetas-medicas`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** registrar una receta (cabecera), ver sus datos y, con permisos, editarla o eliminarla.
- **Cómo se llega:** `/recetas-medicas/nueva`: botón «Nuevo» de la búsqueda; y desde el punto de venta, el enlace «registrar receta» (`checkout.registrarReceta` del scope `pos`) que abre `/recetas-medicas/nueva` en otra pestaña, solo con `Recetas:Crear` (`app/pages/pos/nueva-venta/nueva-venta.component.html:443-451`). `/recetas-medicas/:id`: enlace del paciente en la tabla.
- **Permiso de ruta:** `/recetas-medicas/nueva` → `Recetas:Leer` + `Recetas:Crear`; `/recetas-medicas/:id` → `Recetas:Leer`.
- **Migas y cabecera:** título «Recetas Médicas» (`recetasMedicas.title`); migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Recetas Médicas» (con enlace a `/recetas-medicas`); botón volver a `/recetas-medicas` (`.html:4-8`, `.ts:87-91`). La clave `recetasMedicas.create.title` «Nueva Receta Médica» existe pero no se usa en ninguna plantilla.
- **Modos** (`ngOnInit`, `.ts:115-125`): con `data.isNew === true` → **crear** (formulario de edición, sin llamar a la API); con `id` → carga con `GET` y abre en **ver**; «Editar» pasa a **editar**. Si no hay `id` ni `isNew`, navega a `/recetas-medicas`.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Detalle de Receta» (`recetasMedicas.detail.title`) + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `Recetas:Actualizar`) | No |
| Campos | Editables | Entradas deshabilitadas (solo lectura) | Editables, precargados |
| Botones | Cancelar, Guardar | Editar | Eliminar (izquierda), Cancelar, Guardar (derecha) |

La vista de lectura y la de edición son dos bloques distintos de la plantilla (`.html:16-187` y `:189-261`), no el mismo formulario habilitado.

**Campos del formulario** (`.ts:71-83`, `.html:19-150`)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error | Ayuda |
|---|---|---|---|---|---|---|
| Paciente (`recetasMedicas.fields.paciente`) | `pacienteCtrl` (suelto), fila completa | Texto con autocompletar; sugerencias de `GET /api/v1/pacientes?nombreApellidos=…` (20, espera 300 ms); cada opción muestra nombre y «— {documento}» | Sí | Debe haberse elegido una opción (`validateAutocompletes`, `.ts:178-191`): si no hay paciente elegido marca el control con el error `required` | «Debes seleccionar un paciente» (`recetasMedicas.validation.paciente-required`), mostrado si el control es inválido y tocado (`.html:46-48`) | Placeholder «Buscar paciente...»; sin coincidencias: opción deshabilitada «Sin resultados» |
| Médico (`recetasMedicas.fields.medico`) | `medicoCtrl` (suelto), fila completa | Texto con autocompletar; sugerencias de `GET /api/v1/profesionalessalud?nombrePrescriptor=…`; opción: nombre y «— {número de registro}» | Sí | Igual | «Debes seleccionar un médico» (`recetasMedicas.validation.medico-required`, `.html:78-80`) | Placeholder «Buscar médico...» |
| Fecha de Expedición (`recetasMedicas.fields.fechaExpedicion`) | `fechaExpedicion`, 4/12 | `mat-datepicker` con `[max]="today"` (no se puede elegir una fecha posterior a hoy) | Sí | `Validators.required` (`.ts:79`); el máximo lo impone el selector | «La fecha de expedición es requerida» (`recetasMedicas.validation.fecha-required`), mostrado si es inválido y tocado (`.html:96-101`); también aparece si escribes a mano una fecha posterior a hoy | — |
| Repeticiones Máximas (`recetasMedicas.fields.repeticionesMax`) | `repeticionesMax`, 4/12 | `<input type="number" min="0">` | Sí | `Validators.required`, `Validators.min(0)` (`.ts:80`) | Sin `mat-error`: si queda vacío o negativo el botón «Guardar» no envía y no muestra mensaje | Valor inicial 0 |
| Número de receta (`recetasMedicas.fields.numeroReceta`) | `numeroReceta`, 4/12 | `matInput` con `maxlength` = 30 | No | `Validators.maxLength(30)` (`.ts:25,81`) | «El número de receta admite hasta {{max}} caracteres» con max = 30 (`recetasMedicas.validation.numero-receta-max`, `.html:129-135`) | Pista «Necesario para vender receta retenida o estupefacientes.» (`recetasMedicas.hints.numeroReceta`) |
| Indicaciones de Uso (`recetasMedicas.fields.indicacionesUso`) | `indicacionesUso`, fila completa | `<textarea rows="3">` | No | Ninguna en el formulario | — | — |

Al guardar se envía `{ clienteId, medicoId, fechaExpedicion (yyyy-MM-dd), repeticionesMax, numeroReceta, indicacionesUso }`; el número se recorta (`trim()`) y se omite si queda vacío, y las indicaciones se omiten si están vacías (`.ts:207-214`). En modo editar, «Editar» (`startEdit`, `.ts:141-156`) precarga el paciente y el médico como ya elegidos, así que no hay que volver a elegirlos.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta de lectura, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Recetas:Actualizar`, `.html:202`) | Nunca | `startEdit()` |
| Eliminar (`actions.delete`), clase `sofia-btn-delete`; con spinner mientras elimina | Pie, izquierda | Solo en editar de una receta existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `Recetas:Eliminar`, `.html:156`) | `deleting()` | `eliminar()` (`.ts:245-267`) → diálogo |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:158-164`): en crear navega a `/recetas-medicas`; en editar vuelve a ver |
| Guardar (`actions.save`); con spinner | Pie, derecha | Crear o editar | `saving()` | `guardar()` (`.ts:202-243`). Marca todo como tocado; si falta paciente o médico, o el formulario es inválido, no envía. Crear → `POST /api/v1/recetas`, aviso y navega a `/recetas-medicas`. Editar → `PUT /api/v1/recetas/:id`, aviso, vuelve a ver y recarga |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/recetas-medicas` |

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar» (sin ancho fijo): título «Eliminar receta» (`recetasMedicas.delete.confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta receta médica? Esta acción no se puede deshacer.» (`recetasMedicas.delete.confirm-message`), botones por defecto «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/recetas/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear correctamente | Éxito | «Receta creada correctamente» (`recetasMedicas.create.save-success`) | `recetas-detail.component.ts:221` |
| Editar correctamente | Éxito | «Receta actualizada correctamente» (`recetasMedicas.edit.save-success`) | `.ts:232` |
| Código `Paciente.NotFound` (404) | Error | «El paciente seleccionado no existe. Vuelve a seleccionarlo.» (`recetasMedicas.errors.paciente-not-found`) | `.ts:28` |
| Código `ProfesionalSalud.NotFound` (404) | Error | «El médico seleccionado no existe. Vuelve a seleccionarlo.» (`recetasMedicas.errors.medico-not-found`) | `.ts:29` |
| Código `RecetaMedica.NotFound` (404) | Error | «La receta ya no existe.» (`recetasMedicas.errors.not-found`) | `.ts:30` |
| Código `RecetaMedica.FechaExpedicion` (400, fecha futura o vacía) | Error | «La fecha de expedición no puede ser posterior a hoy.» (`recetasMedicas.errors.fecha-futura`) | `.ts:31` |
| Códigos `RecetaMedica.RepeticionesMax`, `RecetaMedica.NumeroReceta`, `Create/UpdateRecetaCommand.FechaExpedicion`, `.RepeticionesMax`, `.IndicacionesUso` y `.NumeroReceta` (400) | Error | «Revisa los datos ingresados de la receta.» (`recetasMedicas.errors.invalid-data`) | `.ts:32-41` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en [19c §0](19c-catalogos-1.md#0-comportamientos-comunes-a-las-pantallas-de-este-documento) | `save-error-key.util.ts:4-9` |
| Cualquier otro error al crear | Error | «Error al crear la receta» (`recetasMedicas.create.save-error`) | `.ts:226` |
| Cualquier otro error al editar | Error | «Error al actualizar la receta» (`recetasMedicas.edit.save-error`) | `.ts:239` |
| Eliminación correcta | Éxito | «Receta eliminada correctamente» (`recetasMedicas.delete.success`); navega a `/recetas-medicas` | `.ts:257-258` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No puedes eliminar esta receta porque ya se usó en ventas.» (`recetasMedicas.delete.in-use`) | `.ts:260-263` |
| Eliminar: cualquier otro error | Error | «Error al eliminar la receta» (`recetasMedicas.delete.error`) | `.ts:260-263` |
| Error al cargar | Error | «Error al cargar la receta» (`recetasMedicas.edit.load-error`) | `.ts:136` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir `/recetas-medicas/:id` y tras editar | `RecetaMedicaService.getById` (`receta-medica.service.ts:32-34`) | `GET /api/v1/recetas/{id}` | `Recetas:Leer` (`RecetasController.cs:40`) |
| Crear | `create` (`:36-38`) | `POST /api/v1/recetas` | `Recetas:Crear` (`:16`) |
| Editar | `update` (`:40-42`) | `PUT /api/v1/recetas/{id}` | `Recetas:Actualizar` (`:67`) |
| Eliminar | `delete` (`:44-46`) | `DELETE /api/v1/recetas/{id}` | `Recetas:Eliminar` (`:75`) |
| Escribir en «Paciente» / «Médico» | `PacienteService.search` / `ProfesionalSaludService.search` | `GET /api/v1/pacientes?…` / `GET /api/v1/profesionalessalud?…` | `Pacientes:Leer` / `ProfesionalesSalud:Leer` |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Spinner mientras se carga el registro (`.html:10-14`).
- Eliminar solo se ofrece dentro del modo editar, al pie izquierdo; en ver no hay botón de eliminar.
- La API rechaza eliminar una receta usada en una línea de venta no eliminada (incluye ventas anuladas) con 409 `RecetaMedica.InUse` (`DeleteRecetaCommandHandler.cs:24-31`, RN-REC-013); una receta inexistente responde 404 con el código `NotFound` (`:21`), que la pantalla trata con el mensaje genérico.
- La API devuelve 200 con el identificador al crear y al editar (RN-REC-015).

**Observaciones** (sacadas de leer el código; no se ejecutó)
- **Elección de paciente o médico que no se invalida al escribir:** a diferencia de la búsqueda, el detalle no descarta la elección cuando cambias el texto del campo (`.ts:94-112` solo actualiza las sugerencias; no hay equivalente a `.ts:76` de la búsqueda). Si eliges un paciente y luego modificas el texto sin elegir otra opción, «Guardar» sigue enviando el paciente anterior mientras el campo muestra otro texto. No verificado en ejecución.
- **Indicaciones de uso:** sin límite en el formulario; la API rechaza más de 1000 caracteres (`CreateRecetaCommandValidator.cs:14`), con código `CreateRecetaCommand.IndicacionesUso` que la pantalla mapea a «Revisa los datos ingresados de la receta.».
- **Número de receta:** 30 en ambos lados (`RecetaMedica.NumeroRecetaMaxLength = 30`, `RecetaMedica.cs:8`).
- **Fecha de expedición:** el selector no deja elegir fechas posteriores a hoy; el servidor la compara con la fecha UTC de hoy (RN-REC-007), por lo que cerca de la medianoche puede aceptar o rechazar distinto del navegador (no se probó).
- **Repeticiones máximas:** sin mensaje de error en el formulario; el servidor rechaza negativos con `RecetaMedica.RepeticionesMax`.
- Specs `recetas-detail.component.spec.ts` (11 casos: modo crear sin cargar, no guarda sin paciente y médico, formato `yyyy-MM-dd`, número recortado, longitud máxima del número, mapa de códigos a «Revisa los datos ingresados de la receta.», `Paciente.NotFound`, carga en ver, número de receta visible y conservado al editar, visibilidad de «Eliminar» según `Recetas:Eliminar`).
- Claves del scope sin uso: ver §9.

### 5.3 `/recetas-medicas/escanear` — Lector de Recetas IA (escáner)

- **Componente:** `RecetasScannerComponent` (`app/pages/recetas-medicas/recetas-scanner/recetas-scanner.component.ts`, `.html`, `.scss`). Scope `recetas-medicas`.
- **Para qué sirve:** subir o fotografiar una receta médica manuscrita o impresa y obtener, con inteligencia artificial, la lista de medicamentos detectados con su confianza, su stock en tu sucursal y sus equivalentes. **Solo muestra el resultado: no crea la receta, no guarda nada y no pasa al punto de venta** (la pantalla no tiene botones de acción posteriores; RN-REC-029 de 01c1).
- **Cómo se llega:** botón «Escanear receta» de `/recetas-medicas` (solo con `Recetas:Analizar`); botón volver y migas.
- **Permiso de ruta:** `Recetas:Leer` + `Recetas:Analizar` (`ROUTE_ACTIONS['recetas-medicas/escanear'] = 'Analizar'`, `route-permissions.ts:30`; `permission.guard.ts:13,28`). Sin permiso: aviso «No tienes permisos para acceder a esta pantalla.» y redirección a `/dashboard`. El componente no usa `*appPermiso`.
- **Migas y cabecera:** título «Lector de Recetas IA» (`recetasMedicas.scanner.title`); migas «Inicio» / «Pacientes & Atención» (sin enlace) / «Recetas Médicas» (con enlace a `/recetas-medicas`); botón volver a `/recetas-medicas` (`.html:19-23`, `.ts:33-37`).
- **Modos:** uno solo; la pantalla pasa por tres estados: sin imagen, con imagen elegida y analizando, y con resultados.

**Tarjeta de captura** (`.html:26-130`)

| Elemento | Texto exacto (clave) | Comportamiento |
|---|---|---|
| Título de la tarjeta | «Escanear receta médica» (`scanner.card-title`) | — |
| Insignia con icono `auto_awesome` | «Análisis con IA» (`scanner.ai-label`) | Solo informativa |
| Subtítulo | «Toma una foto de la receta o selecciona una imagen de la galería» (`scanner.card-subtitle`) | — |
| Zona vacía (sin imagen) | «Captura o selecciona una imagen para comenzar» (`scanner.empty-hint`) y «o arrastra una imagen aquí» (`scanner.drop-zone-hint`) | Admite arrastrar y soltar |
| Zona con imagen elegida | Vista previa de la imagen (con `alt="Receta"` en duro) y, debajo, el nombre del archivo con icono `image` | La vista previa se lee con `FileReader` en el navegador (`.ts:63-65`) |
| Zona analizando | Anillo giratorio con icono `auto_awesome`, «Interpretando...» (`scanner.interpreting`) y «Análisis con IA» | Desde que pulsas «Interpretar receta» hasta que llega la respuesta (`loading`) |

**Cómo se carga la imagen** (`.html:3-16`, `.ts:47-117`)

| Vía | Control | Qué acepta |
|---|---|---|
| Botón «Tomar foto» (`scanner.btn-camera`, icono `photo_camera`, botón relleno primario) | `<input type="file" accept="image/*" capture="environment">` oculto | Cualquier archivo que el navegador ofrezca como imagen; en un teléfono abre la cámara trasera, en un equipo abre el selector de archivos |
| Botón «Galería» (`scanner.btn-gallery`, icono `photo_library`, con borde) | `<input type="file" accept="image/*">` oculto | Igual, sin forzar la cámara |
| Arrastrar y soltar sobre la zona | Eventos `dragover`, `dragleave` y `drop` | Solo si el tipo del archivo empieza por `image/`; si no, se ignora sin aviso; no hace nada mientras analiza (`.ts:101-117`) |

Se usa un solo archivo (el primero). **El SPA no limita el tamaño ni comprueba la extensión** (solo `accept="image/*"` y, al soltar, el prefijo `image/`). El servidor limita el cuerpo a 10 MB (`[RequestSizeLimit(10 * 1024 * 1024)]`, `RecetasController.cs:50-51`) y no valida el tipo del archivo (RN-REC-018 y RN-REC-019 de 01c1); un archivo mayor responde con error y la pantalla muestra el mensaje genérico de abajo. Elegir un archivo limpia los resultados anteriores y reinicia el selector para poder volver a elegir el mismo archivo (`.ts:60-67`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Tomar foto (`scanner.btn-camera`) | Debajo de la zona, izquierda | Siempre | Mientras analiza | `abrirCamara()` abre el selector con captura |
| Galería (`scanner.btn-gallery`) | Junto al anterior | Siempre | Mientras analiza | `abrirGaleria()` |
| Limpiar (`scanner.btn-clear`), icono `close`, clase `sofia-btn-delete` | Debajo de la zona, derecha | Hay imagen elegida | Mientras analiza | `limpiar()`: quita la imagen, la vista previa y los resultados |
| Interpretar receta (`scanner.btn-interpret`), icono `auto_awesome` | Debajo de la tarjeta, centrado | Hay imagen elegida y no está analizando | — | `interpretar()` (`.ts:70-87`) → `POST /api/v1/recetas/analizar` |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/recetas-medicas` |

**Resultados** (`.html:132-304`; solo si ya hay respuesta y no está analizando)

La respuesta es una lista de elementos de dos tipos, que la pantalla separa en dos tarjetas (`.ts:39-45`):

1. **«Medicamentos de la receta (N)»** (`scanner.section-receta`, icono `medication`): los elementos de tipo `Receta`, uno por medicamento detectado. Cada tarjeta muestra:
   - Nombre oficial (`nombreOficial`: el medicamento de la empresa que mejor coincide; si no hay, el nombre DIGEMID o el texto detectado).
   - «Detectado como: "…"» (`scanner.detected-as`) solo si el texto detectado difiere del nombre oficial.
   - Insignias: «DIGEMID» (texto en duro, icono `verified`) si el nombre se validó contra el catálogo DIGEMID; «Cantidad sugerida: N» (`scanner.suggested-quantity`, icono `numbers`) si la IA leyó una cantidad; y «Confianza: Alta/Media/Baja» (`scanner.confidence`; `scanner.confidence-high`, `-medium`, `-low`) según la confianza final: Alta si es 0,85 o más, Media si es 0,65 o más, Baja en otro caso (`confianzaLabel`, `.ts:125-130`), con color según el tramo (`confianzaColor`, `.ts:132-136`).
   - Estado de stock: con stock «En stock: N unidades» (`scanner.stock-available`, `scanner.units`; número sin decimales) o «Sin stock» (`scanner.no-stock`). El stock es el de **tu sucursal** (la del token de sesión), solo de lotes no vencidos; si tu sesión no trae una sucursal válida el stock es 0 (`AnalizarRecetaQueryHandler.cs:23,132-147`, RN-REC-026).
   - Si hay equivalentes: «Equivalentes terapéuticos:» (`scanner.equivalents`) con una fila por cada uno: nombre comercial, «— {laboratorio}» si lo tiene, y su stock («{N} unidades» con icono de visto o «Sin stock»). Equivalente es otro medicamento con el mismo ingrediente activo, concentración y unidad (RN-REC-027).
   - Si el medicamento no tiene stock pero algún equivalente sí: aviso «No hay stock del medicamento prescrito, pero hay equivalentes disponibles.» (`scanner.no-stock-but-equivalent`, icono `info`).
2. **«Sugerencias IA (N)»** (`scanner.section-suggestions`, icono `auto_awesome`): los elementos de tipo `Recomendacion` (complementarios o genéricos que propone la IA y que existen en el catálogo de la empresa). Cada tarjeta muestra el nombre oficial, la razón («Sugerido por {medicamento}», texto que genera el servidor) y el estado de stock («En stock: N unidades» o «Sin stock»). Sin insignias ni equivalentes.
3. **Sin resultados:** si la lista llega vacía, tarjeta con el icono `search_off` y «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» (`scanner.no-results`).

Las secciones solo se pintan si tienen elementos. Los elementos se identifican por `nombreOficial` en el `@for`. La respuesta tiene como máximo 20 medicamentos y 3 sugerencias por cada uno (RN-REC-022).

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Cualquier error del análisis: límite de peticiones (429), archivo demasiado grande (413) o inválido, fallo del servicio de IA o de OCR (500), sin permiso (403), sin conexión | Error | «Error al interpretar la receta. Intenta nuevamente.» (`recetasMedicas.scanner.error`) | `recetas-scanner.component.ts:84` |
| Respuesta sin medicamentos | Información en pantalla | «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» (`recetasMedicas.scanner.no-results`) | `.html:298-303` |

La pantalla **no distingue** el motivo del fallo: no lee el estado HTTP ni `error.code`. Lo que puede devolver el servidor, para el soporte: 429 al pasar de 10 análisis por minuto por empresa con hasta 2 en cola (`RateLimitingExtensions.cs:43-49`, política `ai-endpoints`); 413 `Request.TooLarge` si el cuerpo pasa de 10 MB (`GlobalExceptionHandler.cs:47-52`); 400 con el texto «Image is required.» si falta la imagen (`RecetasController.cs:54-57`); 500 si falla Gemini o la anonimización («Failed to analyze prescription image.», `GeminiRecetaAnalyzer.cs:58-61`); 400 por especialidad no válida (el SPA nunca envía `especialidadContexto`, así que no ocurre).

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Pulsar «Interpretar receta» | `RecetaMedicaService.analizar` (`receta-medica.service.ts:52-58`) | `POST /api/v1/recetas/analizar`, cuerpo `multipart/form-data` con el campo `imagen`; el SPA no envía `especialidadContexto` | `Recetas:Analizar`, política de límite `AiPolicy` (`RecetasController.cs:47-49`) |

**Exportación y carga masiva:** no hay.

**Comportamientos particulares**
- Los botones de captura, «Limpiar» y la zona de soltar se bloquean mientras analiza; el botón «Interpretar receta» se oculta durante el análisis (`.html:115`).
- El servicio de análisis es el mismo que usa el punto de venta para escanear recetas (`nueva-venta.component.ts:754`); esa pantalla lo documenta otra parte.
- Se envía la imagen a un servicio externo de IA (Gemini); el texto extraído se anonimiza antes del segundo paso (RN-REC-021).
- El `.scss` del componente (598 líneas) solo da estilo; no contiene textos.

**Observaciones**
- Textos en duro: «DIGEMID» (`.html:164`) y `alt="Receta"` (`.html:63`).
- El mensaje de error es único para todos los casos; el usuario no sabe si debe reintentar más tarde (límite de peticiones) o cambiar la imagen.
- Sin spec del componente de escáner.
- Claves del scope sin uso: ver §9.

---

## 6. Eliminar: mapa de códigos de error y mensajes de las pantallas con botón «Eliminar»

Contexto: el commit `27597cd` del backend añadió la comprobación de dependencias al eliminar pacientes y profesionales de salud (y productos, sucursales, ingredientes activos y proveedores, fuera de esta parte), y el `01b8260` del SPA añadió los mensajes. En esta parte **cambiaron Pacientes y Profesionales de salud**. Lotes ya tenía su guarda desde el alta del módulo (SPA `4efd5de` y backend `feaf81a`, 2026-09-06) y Recetas médicas desde 2026-09-30 (SPA `adb14ce` y backend `d49fa33`), según `git log -S` sobre las carpetas y los códigos. Stock por sucursal no tiene botón de eliminar ni endpoint de borrado.

**Qué hace el SPA ante la respuesta del `DELETE`** (idéntico en las cuatro pantallas con botón «Eliminar»): no lee el código de error de la respuesta (`error.code`), solo el estado HTTP. Si el estado es 409 muestra el mensaje «en uso» de la pantalla; con cualquier otro error (404, 403, 500, sin conexión…) muestra el mensaje genérico de la pantalla. Fuente: `lotes-detail.component.ts:224-231`, `pacientes-detail.component.ts:199-204`, `profesionales-salud-detail.component.ts:176-182`, `recetas-detail.component.ts:260-263`. Ante un 409 con cualquier otro código el usuario vería igualmente el texto «en uso».

| Pantalla | `DELETE` | Códigos que puede devolver la API | Mensaje con 409 (clave) | Mensaje con otro error (clave) |
|---|---|---|---|---|
| Lotes (§1.2) | `/api/v1/lotesinventario/{id}`, `Inventarios:Eliminar` | 404 `LoteInventario.NotFound` (`DeleteLoteInventarioCommandHandler.cs:18`); 409 `LoteInventario.InUse` (`:28`) si el lote tiene una fila de stock en alguna sucursal (cualquiera que sea su cantidad) o una línea de venta no eliminadas (`:21-26`); éxito 204 | «No se puede eliminar un lote con stock o ventas registradas.» (`lotes.detail.delete-in-use`) | «Error al eliminar el lote» (`lotes.detail.delete-error`) |
| Pacientes (§3.2) | `/api/v1/pacientes/{id}`, `Pacientes:Eliminar` | 404 con el código genérico `NotFound` (`DeletePacienteCommandHandler.cs:21`); 409 `Paciente.InUse` (`:27`) si lo referencian recetas, ventas, citas de servicios o inmunizaciones no eliminadas (`:37-41`); éxito 200 con el identificador | «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» (`pacientes.detail.delete-in-use`) | «Error al eliminar el paciente» (`pacientes.detail.delete-error`) |
| Profesionales de salud (§4.2) | `/api/v1/profesionalessalud/{id}`, `ProfesionalesSalud:Eliminar` | 404 `ProfesionalSalud.NotFound` (`DeleteProfesionalSaludCommandHandler.cs:17`); 409 `ProfesionalSalud.InUse` (`:25`) si lo referencia alguna receta no eliminada (`:20-21`); éxito 204 | «No puedes eliminar este profesional de salud porque tiene recetas asociadas.» (`profesionalesSalud.detail.delete-in-use`) | «Error al eliminar el profesional de salud» (`profesionalesSalud.detail.delete-error`) |
| Recetas médicas (§5.2) | `/api/v1/recetas/{id}`, `Recetas:Eliminar` | 404 con el código genérico `NotFound` (`DeleteRecetaCommandHandler.cs:21`); 409 `RecetaMedica.InUse` (`:29`) si alguna línea de venta no eliminada la usa, incluidas las ventas anuladas (`:24-27`); éxito 200 con el identificador | «No puedes eliminar esta receta porque ya se usó en ventas.» (`recetasMedicas.delete.in-use`) | «Error al eliminar la receta» (`recetasMedicas.delete.error`) |
| Stock por sucursal (§2) | No hay botón, llamada ni endpoint de borrado | — | — | — |

Los textos de 404 no tienen mensaje propio en ninguna pantalla.

**Cobertura de los mensajes frente a lo que bloquea el backend**

| Pantalla | Qué bloquea el backend | Qué dice el mensaje | Observación |
|---|---|---|---|
| Lotes | Stock en sucursal o ventas | Stock o ventas registradas | Coincide |
| Pacientes | Recetas, ventas, citas de servicios e inmunizaciones | Recetas, ventas u otros registros asociados | Coincide («otros registros» cubre citas e inmunizaciones) |
| Profesionales de salud | Recetas | Recetas asociadas | Coincide |
| Recetas médicas | Líneas de venta (incluidas las de ventas anuladas) | Ya se usó en ventas | Coincide |

Los specs `pacientes-detail.delete.spec.ts` y `profesionales-salud-detail.delete.spec.ts` (con el helper `app/shared/testing/delete-in-use-testing.ts`) comprueban que un 409 muestra la clave `detail.delete-in-use` y otro error la clave `detail.delete-error`.

---

## 7. Pendientes y «No verificado»

**«En curso (sin commitear)»:** ninguno. Se ejecutó `git status --short` en ambos repos al empezar y al cerrar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`): sin cambios en ninguno.

**«No verificado»** (no se ejecutó la aplicación ni se llamó a la API):

| # | Tema | Por qué no se verificó |
|---|---|---|
| 1 | Stock por sucursal, «Guardar» en `/stock-por-sucursal/nueva` sin lote elegido: `TypeError` en la línea `:183` de `stock-detail.component.ts`, sin petición ni aviso visible (§2.2) | Se deduce de leer `saveNew()` y `loteDisplay()`; no se ejecutó |
| 2 | Stock por sucursal: los textos «Debes seleccionar una sucursal» y «Debes seleccionar un lote» no se pintan porque los controles no tienen validadores (§2.2, nota 1) | Se deduce de leer la plantilla y el código de `MatFormField` (`form-field2.mjs:847`); no se ejecutó |
| 3 | Lotes: que un duplicado devuelva 400 y por tanto muestre «Error al guardar el lote» y no el aviso de duplicado (§1.2) | Se deduce de `Result.Failure(error)` sin código de estado (400 por defecto) y de que el SPA solo mira el 409; no se llamó a la API |
| 4 | Lotes: no poder guardar la edición de un lote ya vencido (§1.2) | Se deduce de `UpdateLoteInventarioCommandValidator.cs:21-23`; no se probó |
| 5 | Elegir una sugerencia con el teclado (flechas y Enter) en los autocompletar de Lotes, Stock y Recetas | Depende del comportamiento interno de Angular Material; el código solo guarda la elección en el `(click)` de la opción |
| 6 | Qué ve el usuario cuando falta el permiso `Leer` del módulo consultado por un autocompletar (Medicamentos en Lotes y Stock; Pacientes y Profesionales de salud en Recetas) | Las suscripciones no tienen manejador de error; el efecto visual no se probó |
| 7 | Pacientes, carga masiva: encabezado crudo `fields.tipoDocumento` en la primera columna de la vista previa (§3.3) | La clave no existe en la raíz; el comportamiento de Transloco ante una clave inexistente no se ejecutó |
| 8 | Recetas, detalle: que el paciente o el médico anteriores sigan enviándose si cambias el texto sin elegir otra opción (§5.2) | Se deduce de que no hay código que descarte la elección; no se probó |
| 9 | Recetas, búsqueda: interpretación por la API de las fechas enviadas como instante (`toISOString()`) según la zona horaria del servidor | No se llamó a la API |
| 10 | Aspecto visual: iconos, spinner, insignias y colores de confianza del escáner, anillo de carga, vista previa de la imagen | Solo lectura de plantillas y estilos |
| 11 | Valores formados solo por espacios (número de lote, nombre del paciente, nombre del prescriptor) que llegan a la API y muestran el mensaje genérico | Se deduce de `Validators.required` de Angular y de `NotEmpty` de FluentValidation; no se probó |
| 12 | Cantidad decimal en el campo numérico de Stock por sucursal (sin `step`) | Depende de la validación nativa del navegador |

**Pendientes de producto detectados** (no son errores de documentación):
- Recetas médicas: no hay líneas de medicamentos ni vigencia en la receta, y el escáner no ofrece crear la receta ni pasar al punto de venta; el resultado del análisis no se guarda.
- Recetas médicas, escáner: un único mensaje de error para límite de peticiones, archivo inválido o demasiado grande y fallo del servicio de IA.
- Lotes: `lotes.detail.save-conflict` casi nunca aparece, porque el duplicado normal responde 400; el selector de fecha no limita la caducidad ni la fabricación.
- Stock por sucursal: validación de lote y sucursal sin efecto visible al guardar (hallazgos 1 y 2 de la tabla anterior).
- Pacientes y Profesionales de salud: límites del formulario distintos de los de la API (contacto 150 contra 100; nombre del prescriptor 200 contra 150; dirección de clínica 300 contra 255) y fecha de nacimiento futura no validada en el formulario.
- Solo API, sin pantalla: `GET /api/v1/lotesinventario/producto/{productoId}` y `GET /api/v1/lotesinventario/stock/medicamento/{medicamentoId}`.

**Documentos hermanos que el código actual contradice** (para la pasada de reconciliación):
- 01b: RN-PAC-010 y RN-PRO-007 dicen que eliminar un paciente o un profesional no valida dependencias; con `27597cd` sí lo hacen (`DeletePacienteCommandHandler.cs:24-29`, `DeleteProfesionalSaludCommandHandler.cs:20-27`). Los hallazgos 7 y 10 de 01b que mencionan pacientes y profesionales quedan afectados.

---

## 8. Tabla de cobertura

**Rutas de `docs/_inventario/rutas-spa.csv` de este alcance (18 de 18)**

| Ruta | Componente | Ficha |
|---|---|---|
| `/lotes` | `LotesSearchComponent` | §1.1 |
| `/lotes/nueva` | `LotesDetailComponent` (modo crear) | §1.2 |
| `/lotes/:id` | `LotesDetailComponent` (modos ver y editar) | §1.2 |
| `/stock-por-sucursal` | `StockSearchComponent` | §2.1 |
| `/stock-por-sucursal/nueva` | `StockDetailComponent` (ingreso de stock) | §2.2 |
| `/stock-por-sucursal/:id` | `StockDetailComponent` (ver y ajustar) | §2.3 |
| `/pacientes` | `PacientesSearchComponent` | §3.1 |
| `/pacientes/nueva` | `PacientesDetailComponent` (modo crear) | §3.2 |
| `/pacientes/carga-masiva` | `PacientesCargaMasivaComponent` | §3.3 |
| `/pacientes/:id` | `PacientesDetailComponent` (modos ver y editar) | §3.2 |
| `/profesionales-salud` | `ProfesionalesSaludSearchComponent` | §4.1 |
| `/profesionales-salud/nueva` | `ProfesionalesSaludDetailComponent` (modo crear) | §4.2 |
| `/profesionales-salud/carga-masiva` | `ProfesionalesSaludCargaMasivaComponent` | §4.3 |
| `/profesionales-salud/:id` | `ProfesionalesSaludDetailComponent` (modos ver y editar) | §4.2 |
| `/recetas-medicas` | `RecetasSearchComponent` | §5.1 |
| `/recetas-medicas/escanear` | `RecetasScannerComponent` | §5.3 |
| `/recetas-medicas/nueva` | `RecetasDetailComponent` (modo crear) | §5.2 |
| `/recetas-medicas/:id` | `RecetasDetailComponent` (modos ver y editar) | §5.2 |

**Componentes de `docs/_inventario/componentes-pages.txt` de este alcance (13 de 13)**

| Componente (carpeta bajo `app/pages`) | Ficha |
|---|---|
| `lotes/lotes-search`, `lotes/lotes-detail` | §1.1, §1.2 |
| `stock-por-sucursal/stock-search`, `stock-por-sucursal/stock-detail` | §2.1, §2.2 y §2.3 |
| `pacientes/pacientes-search`, `-detail`, `-carga-masiva` | §3.1, §3.2, §3.3 |
| `profesionales-salud/profesionales-salud-search`, `-detail`, `-carga-masiva` | §4.1, §4.2, §4.3 |
| `recetas-medicas/recetas-search`, `recetas-detail`, `recetas-scanner` | §5.1, §5.2, §5.3 |

**Diálogos de `docs/_inventario/dialogos.txt`:** ninguna carpeta de este alcance contiene un diálogo propio (búsqueda en `dialogos.txt` sin resultados). El único diálogo que abren estas pantallas es `ConfirmDialogComponent` (compartido, [19a §6.1](19a-acceso-layout-y-compartidos.md)) al eliminar, en los detalles de §1.2, §3.2, §4.2 y §5.2. Stock por sucursal y el escáner no abren ninguno.

**Usos de `*appPermiso` de `docs/_inventario/app-permiso-usos.csv` de este alcance (17 de 17)**

| Archivo:línea (bajo `app/pages/…`) | Expresión | Permiso resultante | Botón y ficha |
|---|---|---|---|
| `lotes/lotes-search/…search.component.html:19` | `'Crear'` | `Inventarios:Crear` | «Nuevo», §1.1 |
| `lotes/lotes-detail/…detail.component.html:27` | `'Actualizar'` | `Inventarios:Actualizar` | «Editar», §1.2 |
| `lotes/lotes-detail/…detail.component.html:120` | `'Eliminar'` | `Inventarios:Eliminar` | «Eliminar», §1.2 |
| `stock-por-sucursal/stock-search/…search.component.html:20` | `'Actualizar'` | `Inventarios:Actualizar` | «Nuevo», §2.1 |
| `stock-por-sucursal/stock-detail/…detail.component.html:133` | `'Actualizar'` | `Inventarios:Actualizar` | «Editar», §2.3 |
| `pacientes/pacientes-search/…search.component.html:19` | `'Crear'` | `Pacientes:Crear` | «Carga Masiva», §3.1 |
| `pacientes/pacientes-search/…search.component.html:27` | `'Crear'` | `Pacientes:Crear` | «Nuevo», §3.1 |
| `pacientes/pacientes-detail/…detail.component.html:27` | `'Actualizar'` | `Pacientes:Actualizar` | «Editar», §3.2 |
| `pacientes/pacientes-detail/…detail.component.html:126` | `'Eliminar'` | `Pacientes:Eliminar` | «Eliminar», §3.2 |
| `profesionales-salud/profesionales-salud-search/…search.component.html:20` | `'Crear'` | `ProfesionalesSalud:Crear` | «Carga Masiva», §4.1 |
| `profesionales-salud/profesionales-salud-search/…search.component.html:28` | `'Crear'` | `ProfesionalesSalud:Crear` | «Nuevo», §4.1 |
| `profesionales-salud/profesionales-salud-detail/…detail.component.html:28` | `'Actualizar'` | `ProfesionalesSalud:Actualizar` | «Editar», §4.2 |
| `profesionales-salud/profesionales-salud-detail/…detail.component.html:88` | `'Eliminar'` | `ProfesionalesSalud:Eliminar` | «Eliminar», §4.2 |
| `recetas-medicas/recetas-search/…search.component.html:21` | `'Analizar'` | `Recetas:Analizar` | «Escanear receta», §5.1 |
| `recetas-medicas/recetas-search/…search.component.html:28` | `'Crear'` | `Recetas:Crear` | «Nuevo», §5.1 |
| `recetas-medicas/recetas-detail/…detail.component.html:156` | `'Eliminar'` | `Recetas:Eliminar` | «Eliminar», §5.2 |
| `recetas-medicas/recetas-detail/…detail.component.html:202` | `'Actualizar'` | `Recetas:Actualizar` | «Editar», §5.2 |

Los componentes de estas carpetas no comprueban permisos en el `.ts` (sin llamadas a `AuthService.hasPermission` ni a `grantsPermission`); el escáner y el detalle de stock en modo crear no usan `*appPermiso` (los protege el permiso de ruta).

**Endpoints de `docs/_inventario/endpoints.csv` de este alcance (38) y quién los llama**

| Controlador | Endpoints en el CSV | Llamados por pantallas de esta parte | No llamados desde esta parte |
|---|---|---|---|
| `LotesInventario` | 9 | 7: `GET` lista, `GET` por id, `POST`, `PUT`, `DELETE`, `POST stock` (ingreso, §2.2) y `POST exportar` (§1.1) | `GET producto/{productoId}` y `GET stock/medicamento/{medicamentoId}` (solo API, sin pantalla; ningún archivo del SPA los llama) |
| `StockPorSucursal` | 4 | 4 (§2) | — |
| `Pacientes` | 9 | 9 (§3.1 a §3.3); el `GET` de lista lo llama también Recetas (§5) | — |
| `ProfesionalesSalud` | 9 | 9 (§4.1 a §4.3); el `GET` de lista lo llama también Recetas (§5) | — |
| `Recetas` | 7 | 7 (§5) | — |

Se usan además, documentados en otras partes: `GET /api/v1/medicamentos` (autocompletar de producto, §1 y §2) y `GET /api/v1/sucursales/permitidas` (§2).

---

## 9. Claves i18n sin uso

Método: para cada clave del `es.json` del scope se buscó la forma completa (`<alias>.<clave>` o `<scope>.<clave>`, entre comillas) en todo `src/` (sin specs) y la forma relativa `'<clave>'` en los archivos de la carpeta del feature. Las claves `actions.edit` y `actions.delete` de Pacientes se cuentan sin uso porque las plantillas usan las de la raíz.

| Scope (claves en `es.json`) | Claves sin ninguna referencia |
|---|---|
| `lotes` (27) | Ninguna |
| `stock-por-sucursal` (37) | 2: `detail.adjust`, `create.title` |
| `pacientes` (82) | 54: `searchPlaceholder`, `add`, `cols.docIdentidadGub`, `cols.nombreApellidos`, `cols.fechaNacimiento`, `cols.contactoPrimario`, `actions.edit`, `actions.view`, `actions.delete`, `form.titleNew`, `form.titleEdit`, `form.titleView`, `form.docIdentidadGub`, `form.nombreApellidos`, `form.fechaNacimiento`, `form.contactoPrimario`, `form.save`, `form.cancel`, `form.back`, `form.required`, `form.maxLength50`, `form.maxLength100`, `form.maxLength200`, `form.futureDateError`, `form.phoneInvalid`, `deleteDialog.title`, `deleteDialog.message`, `deleteDialog.confirm`, `deleteDialog.cancel`, `notifications.created`, `notifications.updated`, `notifications.deleted`, `help.docIdentidadGub`, `help.nombreApellidos`, `help.fechaNacimiento`, `help.contactoPrimario`, `import.title`, `import.downloadTemplate`, `import.selectFile`, `import.dragDrop`, `import.fileSelected`, `import.errorsFound`, `import.saveValid`, `import.noValid`, `import.saving`, `import.previewTitle`, `import.success`, `import.errors`, `import.status`, `import.valid`, `import.invalid`, `import.button`, `detail.title-new`, `detail.title-edit` |
| `profesionalesSalud` (21) | 2: `detail.title-new`, `detail.title-edit` |
| `recetas-medicas` (67) | 1: `create.title` |
| `inventarios` (65, scope heredado fuera del alcance) | Las 65: ningún archivo de `src/` carga el scope `inventarios` (ni `scope: 'inventarios'` ni `provideTranslocoScope('inventarios')`). Su texto es «Mantenimiento de Inventario & Lotes»; las pantallas actuales usan `lotes` y `stock-por-sucursal` |

Claves usadas pero **inexistentes**: una. `fields.tipoDocumento` (raíz), usada como etiqueta de la primera columna de la carga masiva de pacientes (`pacientes-carga-masiva.component.ts:30`); la raíz no la define (§3.3). El resto de las claves de `t('…')`, `translate('<alias>.…')`, pipe `transloco` y mapas de errores de los archivos de este alcance resuelve contra el `es.json` del scope o contra la raíz.

---

## 10. Fuentes

**SPA (`sofia-web-spa/`)**
- Enrutado y permisos: `src/app/app.routes.ts`, `src/app/core/auth/route-permissions.ts`, `src/app/core/auth/permission.guard.ts`, `src/app/shared/directives/permiso.directive.ts`, `src/app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts`.
- Compartidos: `src/app/core/services/shared/search-state.service.ts`, `src/app/core/services/shared/error-notifier.service.ts`, `src/app/shared/utils/save-error-key.util.ts`, `src/app/shared/utils/date-only.util.ts`, `src/app/shared/utils/tipo-documento.util.ts`, `src/app/shared/components/carga-masiva-page/*`, `src/app/shared/models/carga-masiva.model.ts`, `src/app/shared/testing/delete-in-use-testing.ts`.
- Lotes: `src/app/pages/lotes/lotes.routes.ts`, `lotes-search/*`, `lotes-detail/*`, `models/lote.model.ts`, `services/lote.service.ts` (con sus specs).
- Stock por sucursal: `src/app/pages/stock-por-sucursal/stock-por-sucursal.routes.ts`, `stock-search/*`, `stock-detail/*`, `models/stock.model.ts`, `services/stock.service.ts`; `src/app/pages/sucursales/services/sucursal.service.ts`; `src/app/pages/medicamentos/services/medicamento.service.ts`; `src/app/pages/dashboard/dashboard-links.ts`.
- Pacientes: `src/app/pages/pacientes/pacientes.routes.ts`, `pacientes-search/*`, `pacientes-detail/*`, `pacientes-carga-masiva/*`, `models/paciente.model.ts`, `services/paciente.service.ts`.
- Profesionales de salud: `src/app/pages/profesionales-salud/profesionales-salud.routes.ts`, `profesionales-salud-search/*`, `profesionales-salud-detail/*`, `profesionales-salud-carga-masiva/*`, `models/profesional-salud.model.ts`, `services/profesional-salud.service.ts`.
- Recetas médicas: `src/app/pages/recetas-medicas/recetas-medicas.routes.ts`, `recetas-search/*`, `recetas-detail/*`, `recetas-scanner/*` (`.ts`, `.html`, `.scss`), `models/receta-medica.model.ts`, `services/receta-medica.service.ts`; `src/app/pages/pos/nueva-venta/nueva-venta.component.{ts,html}` (solo referencia cruzada).
- Textos: `public/i18n/es.json` (raíz) y `public/i18n/{lotes,stock-por-sucursal,pacientes,profesionalesSalud,recetas-medicas,inventarios,pos}/es.json`.
- Paquetes: `package.json` y `node_modules/@angular/material/fesm2022/form-field2.mjs` (comportamiento de `mat-error`).

**Backend (`sofia-apis/src/`, solo lectura puntual)**
- Controladores: `SOFIA.API/Controllers/LotesInventarioController.cs`, `StockPorSucursalController.cs`, `PacientesController.cs`, `ProfesionalesSaludController.cs`, `RecetasController.cs`, `MedicamentosController.cs`, `SucursalesController.cs`; `SOFIA.API/Extensions/ResultExtensions.cs`, `RateLimitingExtensions.cs`; `SOFIA.API/Infrastructure/GlobalExceptionHandler.cs`.
- Inventario: `SOFIA.Application/Inventarios/Commands/{CreateLote,UpdateLote,DeleteLote,RegisterInventario,AdjustStock}/*`, `Queries/{GetLotes,GetStockPorSucursal,GetStockPorSucursalById}/*`; `SOFIA.Application/Sucursales/Queries/GetSucursalesPermitidas/*`; `SOFIA.Domain/Entities/LoteInventario.cs`, `InventarioSucursal.cs`; `SOFIA.Infrastructure/Persistence/Configurations/LoteInventarioConfiguration.cs`; `SOFIA.DbUp/Scripts/0001_Initial_Schema.sql` (índice `UX_Lotes_Producto_NumeroLote`).
- Pacientes y profesionales: `SOFIA.Application/Pacientes/*`, `SOFIA.Application/Profesionales/*`, `SOFIA.Domain/Entities/PacienteCliente.cs`, `ProfesionalSalud.cs`, `SOFIA.Domain/Enums/TipoDocumentoIdentidad.cs`.
- Recetas: `SOFIA.Application/Recetas/*` (incluido `Queries/AnalizarReceta/*`), `SOFIA.Domain/Entities/RecetaMedica.cs`, `SOFIA.Infrastructure/Services/GeminiRecetaAnalyzer.cs`.
- Comunes: `SOFIA.Application/Common/Behaviors/ValidationBehavior.cs`, `SOFIA.Domain/Common/Result.cs`, `Error.cs`.

**Git (solo lectura):** `git log -1 --format=%h` y `git status --short` de ambos repos; `git show --stat` de `27597cd` (sofia-apis) y `01b8260` (sofia-web-spa); `git log -S` sobre las carpetas del alcance y los códigos `delete-in-use`, `RecetaMedica.InUse` y `LoteInventario.InUse`.

**Inventarios y documentos hermanos**
- `docs/_inventario/rutas-spa.csv`, `componentes-pages.txt`, `dialogos.txt`, `app-permiso-usos.csv`, `endpoints.csv`, `i18n-scopes.csv`, `pruebas-spa.csv`.
- [19a — acceso, layout y compartidos](19a-acceso-layout-y-compartidos.md) (§4.3, §5, §6.1, §6.5, §6.6, §6.9 a §6.12, §7.1).
- [19c — catálogos I](19c-catalogos-1.md) (§0, §3, §6).
- [01c1 — inventario y recetas](../01-reglas-de-negocio/01c1-inventario-y-recetas.md) (RN-LOT, RN-STK, RN-REC) y [01b — catálogos](../01-reglas-de-negocio/01b-catalogos.md) (RN-PAC, RN-PRO).
- [05b — matriz de permisos](../05-seguridad/05b-matriz-de-permisos.md).

