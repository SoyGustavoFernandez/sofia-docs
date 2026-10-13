# Inventario de pantallas del SPA — Parte C: catálogos I

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-13 |
| **Backend** | `sofia-apis`, rama `main`, commit `27597cd` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `01b8260` |
| **Método** | Lectura de código del árbol de trabajo del SPA (componentes, plantillas, servicios, modelos e i18n). Del backend solo se abrieron los controladores, validadores y manejadores necesarios para anotar permisos, códigos de error y comparar validaciones. No se usó el navegador ni la API local. Ninguna carpeta de esta parte tiene cambios sin commitear (ver §7). |

Este documento es la parte C de seis del inventario de pantallas. Cubre cinco catálogos: Unidades de medida, Jerarquías de unidades, Laboratorios, Ingredientes activos y Catálogo DIGEMID (19 rutas). Lo compartido (guards, interceptor, `page-header`, `confirm-dialog`, `range-slider-field`, carga masiva común, textos de la raíz) lo documenta la [parte A](19a-acceso-layout-y-compartidos.md); el patrón de las pantallas de búsqueda y detalle ya validado está en la [parte B, §0](19b-administracion.md). Aquí se enlaza a esas secciones y se repite el texto exacto de cada etiqueta, botón y mensaje que aparece en estas pantallas. Las reglas de negocio de los catálogos están en [01b](../01-reglas-de-negocio/01b-catalogos.md) (UDM, JER, LAB, ING, DIG) y los permisos por endpoint en [05b](../05-seguridad/05b-matriz-de-permisos.md).

Convenciones: las rutas de archivo son relativas a `sofia-web-spa/` salvo que empiecen por `sofia-apis/`. Entre paréntesis va la clave i18n; las claves de scope se escriben con su prefijo real (por ejemplo `unidadesMedida.fields.codigo` está en `public/i18n/unidadesMedida/es.json`, clave `fields.codigo`). `app/` abrevia `src/app/`. Las plantillas de estos módulos abren con `*transloco="let t; scope: '<scope>'; read: '<scope>'"`, así que `t('x')` equivale a `<scope>.x`. Los números de línea de los `.html` y `.ts` son los del árbol de trabajo (que coincide con `HEAD` en estas carpetas).

## Contenido

0. Comportamientos comunes a las pantallas de este documento
1. Unidades de medida: `/unidades-medida`, `/unidades-medida/nueva`, `/unidades-medida/carga-masiva`, `/unidades-medida/:id`
2. Jerarquías de unidades: `/jerarquias`, `/jerarquias/nueva`, `/jerarquias/carga-masiva`, `/jerarquias/:id`
3. Laboratorios: `/laboratorios`, `/laboratorios/nueva`, `/laboratorios/carga-masiva`, `/laboratorios/:id`
4. Ingredientes activos: `/ingredientes-activos`, `/ingredientes-activos/nueva`, `/ingredientes-activos/carga-masiva`, `/ingredientes-activos/:id`
5. Catálogo DIGEMID: `/digemid-catalogo`, `/digemid-catalogo/carga-masiva`, `/digemid-catalogo/:id` (no existe `/nueva`)
6. Eliminar: mapa de códigos de error y mensajes de las cinco pantallas
7. Pendientes y «No verificado»
8. Tabla de cobertura
9. Claves i18n sin uso
10. Fuentes

---

## 0. Comportamientos comunes a las pantallas de este documento

Lo que no se repite en cada ficha, con su fuente. Lo que sea distinto en una pantalla se dice en su ficha.

- **Cómo se decide el permiso de ruta.** Igual que en [19b §0](19b-administracion.md): `canActivateChild: [passwordChangeGuard, permissionGuard]` (`app/app.routes.ts:37`), módulo por el primer segmento de la URL en `ROUTE_PERMISSIONS` (`app/core/auth/route-permissions.ts:2-26`) y siempre `Leer`; si el segundo segmento es `nueva`, `nuevo` o `carga-masiva` se exige además `Crear` (`route-permissions.ts:36,68-71`); `/x/:id` solo exige `Leer`. Sin permiso: snackbar «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard` (`app/core/auth/permission.guard.ts:28-29`). Módulos de esta parte:

  | Primer segmento | Módulo de permisos | Fuente |
  |---|---|---|
  | `unidades-medida` | `UnidadesMedida` | `route-permissions.ts:8` |
  | `jerarquias` | `JerarquiasUoM` | `route-permissions.ts:9` |
  | `laboratorios` | `Laboratorios` | `route-permissions.ts:10` |
  | `ingredientes-activos` | `IngredientesActivos` | `route-permissions.ts:11` |
  | `digemid-catalogo` | `DIGEMID` | `route-permissions.ts:24` |

- **Menú.** Cuatro opciones están en el grupo «Catálogos» (`breadcrumbs.catalogos`, `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:48-52`): «Unidades de Medida» (`nav.unidadesMedida`, `:53-57`), «Jerarquías de Unidades» (`nav.jerarquiasUom`, `:58-62`), «Laboratorios» (`nav.laboratorios`, `:63-67`) e «Ingredientes Activos» (`nav.ingredientesActivos`, `:68-72`). «Catálogo DIGEMID» (`nav.catalogoDigemid`, `:161-165`) está en el grupo «DIGEMID» (`breadcrumbs.digemid`, `:157-160`), que solo tiene esa opción. Una opción se muestra solo si el usuario tiene `Leer` del módulo de su ruta; un grupo sin opciones visibles se oculta ([19a §4.3](19a-acceso-layout-y-compartidos.md)).
- **Directiva `*appPermiso`.** Con un solo valor (`'Crear'`, `'Actualizar'`, `'Eliminar'`) toma el módulo de la URL actual y exige esa acción; el rol `Admin` pasa siempre (`app/shared/directives/permiso.directive.ts:20-31`). Ver [19a §7.1](19a-acceso-layout-y-compartidos.md). Los 17 usos de estas carpetas están en la tabla de cobertura (§8).
- **Aviso al usuario.** Snackbar con el botón «Cerrar» (`actions.close`): error 5 s, éxito 3 s; con `showServerError` y un `traceId` en la respuesta, 10 s y «Referencia: …» ([19a §6.11 y §6.12](19a-acceso-layout-y-compartidos.md)).
- **Estado de la búsqueda.** Las cinco pantallas de búsqueda guardan filtros, página y tamaño de página en memoria (`SearchStateService`, `app/core/services/shared/search-state.service.ts:10-28`) con la URL sin query como clave; al volver desde el detalle se restauran y se repite la búsqueda (`ngOnInit` de cada componente). «Limpiar» (`actions.clear`) vacía el formulario, oculta la tabla, vuelve a la página 1 y borra ese estado.
- **«Buscar» no vuelve a la página 1.** En las cinco pantallas, `search()` usa `pageIndex() + 1`, el índice de página actual. Si estás en la página 3 y cambias un filtro, «Buscar» pide la página 3 de los nuevos filtros (igual que Sucursales, [19b §2.1](19b-administracion.md)). Solo cambia el tamaño o la página con el paginador.
- **Resultados.** Los resultados se muestran solo tras «Buscar» (o por estado guardado). Encabezado «Resultados (N)» (`generic-labels.results`). Paginador con 10, 25 y 50 filas y botones primera/última, 10 por defecto ([19a §6.9](19a-acceso-layout-y-compartidos.md)). El nombre de la primera columna es siempre un enlace a `/<ruta>/:id`.
- **Fila sin resultados.** Cuatro pantallas (Unidades de medida, Laboratorios, Ingredientes activos y Catálogo DIGEMID) muestran en la fila vacía el texto «Resultados» (`generic-labels.results`) en lugar de «Sin resultados». Solo Jerarquías usa «Sin resultados» (`generic-labels.no-results`, `jerarquias-search.component.html:208`). Ver Observaciones de cada ficha.
- **Campos de texto de los filtros.** Todos llevan el icono de lupa (`<mat-icon matSuffix>search</mat-icon>`) y Enter ejecuta la búsqueda (`(keyup.enter)="search()"`). Sin validaciones. Se envían al servidor solo si tienen valor (`if (params.x)` en cada servicio); la coincidencia es «contiene» (ver 01b).
- **Modo ver / editar** (detalle de Unidades de medida, Jerarquías, Laboratorios e Ingredientes activos). Las rutas `/x/nueva` y `/x/:id` usan el mismo componente: sin parámetro `id` es **crear** (formulario habilitado, `isNew` e `isEditMode` verdaderos); con `id` abre en **ver** (formulario deshabilitado, carga los datos con `GET`) y «Editar» (`actions.edit`) lo habilita. «Cancelar» en crear vuelve a la búsqueda; en editar restaura la copia leída (`snapshot`) y vuelve a modo ver. «Eliminar» solo aparece en modo editar de un registro existente. Detalle del patrón en [19b §2.2](19b-administracion.md).
- **Crear y editar, diferencia de comportamiento al guardar.** En Unidades de medida, Laboratorios e Ingredientes activos, crear con éxito navega a la búsqueda y editar con éxito se queda en la ficha, vuelve a modo ver y recarga los datos. En Jerarquías, crear y editar con éxito navegan a la búsqueda.
- **Mensaje de campo obligatorio:** «Este campo es obligatorio» (`general.errors.required`, raíz). Es el único texto de validación que muestran las plantillas de Unidades de medida, Laboratorios e Ingredientes activos (Jerarquías añade los suyos para el multiplicador y las unidades iguales, §2.2); las longitudes máximas se impiden con el atributo HTML `maxlength` (el campo no deja escribir más). Un valor compuesto solo de espacios cumple `Validators.required` de Angular, pero la API lo rechaza (ver Observaciones de cada ficha).
- **Diálogo de eliminar.** `ConfirmDialogComponent` ([19a §6.1](19a-acceso-layout-y-compartidos.md)), ancho 400 px, título y mensaje de cada scope y botones por defecto «Cancelar» y «Confirmar»; confirmar llama al `DELETE`, cancelar no hace nada.
- **Guardado: mapa de errores.** `saveErrorKey` (`app/shared/utils/save-error-key.util.ts:11-15`) lee `error.code` de la respuesta y resuelve, en este orden: el código propio de la pantalla (`SAVE_ERROR_KEYS`), los códigos de base de datos `Db.Duplicado` → «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» (`errors.dbDuplicado`), `Db.ReferenciaInvalida` → «El registro hace referencia a datos que ya no existen o todavía está en uso.» (`errors.dbReferenciaInvalida`) y `Db.ValorInvalido` → «Uno de los valores está fuera del rango permitido.» (`errors.dbValorInvalido`), y por último el mensaje genérico de la pantalla. Cualquier otro código (incluidos los de FluentValidation, `{Request}.{Propiedad}`) cae en el genérico.
- **Exportación.** Botón «Exportar» (`actions.export`, icono `download`) bajo la tabla, a la izquierda, solo tras buscar. Hace `POST` a `<base>/exportar` con `{ headers, ...filtros }` (los filtros del formulario en ese momento, no necesariamente los de la última búsqueda) y descarga el blob con un nombre fijo que sale del i18n de cada scope. No envía formato de fecha ni textos sí/no (estos catálogos no tienen fechas ni booleanos). El permiso del endpoint es `<Módulo>:Leer`. Ante un error muestra el mensaje «Error al buscar …» de la pantalla (con referencia de seguimiento solo donde se usa `showServerError`; ver cada ficha).
- **Carga masiva.** Las cinco rutas `/x/carga-masiva` de este documento (los cuatro catálogos con alta más Catálogo DIGEMID) usan el componente común `app-carga-masiva-page` ([19a §6.5 y §6.6](19a-acceso-layout-y-compartidos.md)). Cada envoltorio le pasa solo la configuración que se ve en las fichas `x.3`. El título de la pantalla es «Carga Masiva» (`carga-masiva.title`) y el archivo de plantilla se descarga como `plantilla-{backRoute sin la barra}.xlsx` (`carga-masiva.template-filename`, `carga-masiva-page.component.ts:123-134`). Las etiquetas de columna se resuelven contra la **raíz** `fields.*` (`{{ col.label | transloco }}`, `carga-masiva-page.component.html:80`), no contra el scope del catálogo. El campo `required` de cada columna se declara pero el componente común no lo lee (no se pinta ningún marcador de obligatorio).
- **Código en los detalles sin uso.** Los detalles de Unidades de medida, Laboratorios e Ingredientes activos definen `goToNew()`, que ninguna plantilla llama (no existe botón «Nuevo» en la ficha). Ver Observaciones.
- **Otros consumidores de estos servicios.** Las listas de Unidades de medida, Laboratorios e Ingredientes activos las usan también Medicamentos y Formulaciones clínicas (importan `UnidadMedidaService`, `LaboratorioService` e `IngredienteActivoService`); Presentaciones de venta usa `JerarquiaUoMService.getUnidadesVendibles`. Esas pantallas las documentan otras partes.

---

## 1. Unidades de medida

Carpeta `app/pages/unidades-medida`, scope i18n `unidadesMedida` (`provideTranslocoScope('unidadesMedida')` en los cuatro componentes). Componentes enrutados (`app/pages/unidades-medida/unidades-medida.routes.ts`, cargados con `loadChildren` desde `app/app.routes.ts:76`): `UnidadesMedidaSearchComponent` (`''`), `UnidadesMedidaDetailComponent` (rutas `nueva` y `:id`) y `UnidadesMedidaCargaMasivaComponent` (`carga-masiva`). El orden de rutas importa: `nueva` y `carga-masiva` están antes de `:id`.

### 1.1 `/unidades-medida` — Unidades de Medida (búsqueda)

- **Componente:** `UnidadesMedidaSearchComponent` (`app/pages/unidades-medida/unidades-medida-search/unidades-medida-search.component.ts`, `.html`). Scope `unidadesMedida`.
- **Para qué sirve:** buscar unidades de medida por código y descripción, abrirlas, exportar la lista y entrar a crear o a cargar unidades masivamente.
- **Cómo se llega:** menú lateral, grupo «Catálogos» → «Unidades de Medida» (`nav.unidadesMedida`); botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar una unidad.
- **Permiso de ruta:** `UnidadesMedida:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Unidades de Medida» (`unidadesMedida.title`); migas «Inicio» (`breadcrumbs.home`, enlaza a `/dashboard`) / «Catálogos» (`breadcrumbs.catalogos`, sin enlace) / «Unidades de Medida» (`breadcrumbs.unidades-medida`, activa); sin botón volver (`unidades-medida-search.component.html:4-7`, `.ts:46-50`). Tarjeta de filtros con título «Búsqueda de Unidades de Medida» (`unidadesMedida.search.title`) y el subtítulo «Ingresa los criterios de búsqueda» (`generic-labels.search-subtitle`).
- **Modos:** uno solo (búsqueda). Los resultados aparecen tras «Buscar» o por estado guardado (§0).

**Filtros de búsqueda** (`.ts:44`, `.html:38-56`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Código (`unidadesMedida.fields.codigo`) | `codigo` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Descripción (`unidadesMedida.fields.descripcion`) | `descripcion` | Texto con lupa; Enter busca | Texto libre | Vacío |

**Tabla de resultados** (`.html:82-109`, columnas `.ts:52`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Código (`unidadesMedida.table.codigo`) | `codigo` | Texto | Sí: `[routerLink]="[row.id]"` → `/unidades-medida/:id` |
| Descripción (`unidadesMedida.table.descripcion`) | `descripcion` | Texto | No |

Orden por defecto: lo fija el servidor, por descripción y luego por identificador (`sofia-apis/src/SOFIA.Application/UnidadesMedida/Queries/GetUnidadesMedida/GetUnidadesMedidaQueryHandler.cs:28`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:106`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `UnidadesMedida:Crear`, `.html:20`) | Nunca | `[routerLink]="['carga-masiva']"` → `/unidades-medida/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:28`) | Nunca | `goToNew()` → `/unidades-medida/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/unidadesmedida` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/unidadesmedida/exportar` |
| Enlace con el código | Primera columna | Con resultados | — | Navega a `/unidades-medida/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar unidades de medida» (`unidadesMedida.search.search-error`) | `unidades-medida-search.component.ts:92` |
| Falla la exportación | Error | El mismo texto, con referencia de seguimiento si la respuesta trae `traceId` (`showServerError`) | `unidades-medida-search.component.ts:136` |

No hay mapa de códigos del backend en esta pantalla.

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `UnidadMedidaService.search` (`app/pages/unidades-medida/services/unidad-medida.service.ts:19-30`) | `GET /api/v1/unidadesmedida?pageNumber&pageSize[&codigo][&descripcion]` | `UnidadesMedida:Leer` (`UnidadesMedidaController.cs:19`) |
| Exportar | `UnidadMedidaService.exportar` (`:48-50`) | `POST /api/v1/unidadesmedida/exportar` (respuesta blob) | `UnidadesMedida:Leer` (`UnidadesMedidaController.cs:103`) |

**Exportación**
- Cuerpo (`.ts:116-126`): `headers` con 2 textos traducidos, en orden: «Código» (`unidadesMedida.table.codigo`) y «Descripción» (`unidadesMedida.table.descripcion`); más `codigo` y `descripcion` del formulario si tienen valor.
- Archivo: «unidades-medida.xlsx» (`unidadesMedida.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §1.3.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0.

**Observaciones**
- La fila sin resultados dice «Resultados» y no «Sin resultados» (`unidades-medida-search.component.html:106`).
- Claves del scope sin uso: ver §9.

### 1.2 `/unidades-medida/nueva` y `/unidades-medida/:id` — Unidades de Medida (detalle)

- **Componente:** `UnidadesMedidaDetailComponent` (`app/pages/unidades-medida/unidades-medida-detail/unidades-medida-detail.component.ts`, `.html`). Scope `unidadesMedida`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear una unidad de medida, ver sus datos y, con permisos, editarla o eliminarla.
- **Cómo se llega:** `/unidades-medida/nueva`: botón «Nuevo» de la búsqueda (§1.1). `/unidades-medida/:id`: enlace del código en la tabla de la búsqueda.
- **Permiso de ruta:** `/unidades-medida/nueva` → `UnidadesMedida:Leer` + `UnidadesMedida:Crear`; `/unidades-medida/:id` → `UnidadesMedida:Leer`.
- **Migas y cabecera:** título «Unidades de Medida» (`unidadesMedida.title`); migas «Inicio» / «Catálogos» / «Unidades de Medida» (con enlace a `/unidades-medida`); botón volver a `/unidades-medida` (`.html:4-8`, `.ts:59-63`). Misma cabecera en crear, ver y editar.
- **Modos** (`ngOnInit`, `.ts:65-75`): sin parámetro `id` → **crear**; con `id` → **ver** (formulario deshabilitado, carga los datos) y «Editar» lo habilita (§0).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado de la tarjeta «Unidades de Medida» + «Datos del registro» (`generic-labels.detail-subtitle`) + botón «Editar» | No | Sí (botón solo con `UnidadesMedida:Actualizar`) | No |
| Código y Descripción | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:54-57`, `.html:38-54`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Código (`unidadesMedida.fields.codigo`) | `codigo` | `matInput`, `maxlength="10"`, columna de 4/12 | Sí | `Validators.required`, `Validators.maxLength(10)` (`.ts:55`) | «Este campo es obligatorio» (`general.errors.required`, `.html:43`) | Sí / Sí |
| Descripción (`unidadesMedida.fields.descripcion`) | `descripcion` | `matInput`, `maxlength="50"`, columna de 8/12 | Sí | `Validators.required`, `Validators.maxLength(50)` (`.ts:56`) | «Este campo es obligatorio» (`general.errors.required`, `.html:51`) | Sí / Sí |

Sin ayuda ni placeholder (las claves `unidadesMedida.help.*` no se usan).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `UnidadesMedida:Actualizar`, `.html:28`) | Nunca | `enterEditMode()`: habilita el formulario |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:97-105`): en crear navega a `/unidades-medida`; en editar restaura código y descripción de la copia leída y vuelve a ver |
| Guardar (`actions.save`); con spinner mientras guarda | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:107-140`). Inválido → marca todo como tocado y no envía. Crear → `POST /api/v1/unidadesmedida`, aviso de éxito y navega a `/unidades-medida`. Editar → `PUT /api/v1/unidadesmedida/:id`, aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de una unidad existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `UnidadesMedida:Eliminar`, `.html:75`) | `deleting()` | `confirmDelete()` → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/unidades-medida` |

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar» (`.ts:142-153`): título «Confirmar eliminación» (`unidadesMedida.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta unidad de medida?» (`unidadesMedida.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/unidadesmedida/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Unidad de medida guardada correctamente» (`unidadesMedida.detail.save-success`) | `.ts:117,128` |
| Código `UnidadMedida.Codigo.Duplicado` (409) | Error | «Ya existe una unidad de medida con ese código.» (`unidadesMedida.detail.duplicate-codigo`) | `.ts:17-19,122,136` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en §0 | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar | Error | «Error al guardar la unidad de medida» (`unidadesMedida.detail.save-error`) | `.ts:122,136` |
| Eliminación correcta | Éxito | «Unidad de medida eliminada correctamente» (`unidadesMedida.detail.delete-success`); navega a `/unidades-medida` | `.ts:159-160` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.» (`unidadesMedida.detail.delete-in-use`) | `.ts:164-167` |
| Eliminar: cualquier otro error | Error | «Error al eliminar la unidad de medida» (`unidadesMedida.detail.delete-error`) | `.ts:164-167` |
| Error al cargar | Error | «Error al cargar la unidad de medida» (`unidadesMedida.detail.load-error`) | `.ts:87` |

El mapa de eliminar está completo en §6.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir y tras editar | `UnidadMedidaService.getById` (`unidad-medida.service.ts:32-34`) | `GET /api/v1/unidadesmedida/{id}` | `UnidadesMedida:Leer` (`UnidadesMedidaController.cs:27`) |
| Crear | `create` (`:36-38`) | `POST /api/v1/unidadesmedida` | `UnidadesMedida:Crear` (`:35`) |
| Editar | `update` (`:40-42`) | `PUT /api/v1/unidadesmedida/{id}` | `UnidadesMedida:Actualizar` (`:45`) |
| Eliminar | `delete` (`:44-46`) | `DELETE /api/v1/unidadesmedida/{id}` | `UnidadesMedida:Eliminar` (`:59`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga muestra un spinner en lugar de la tarjeta (`.html:10-14`).
- El 409 de eliminar se trata como «en uso» sin mirar el código de la respuesta (`.ts:164`); la API devuelve `UnidadMedida.InUse` cuando la unidad la usan medicamentos (unidad base), jerarquías (unidad mayor o menor) o formulaciones clínicas (`sofia-apis/src/SOFIA.Application/UnidadesMedida/Commands/DeleteUnidadMedida/DeleteUnidadMedidaCommandHandler.cs:20-31`).
- Si la carga del registro falla, la tarjeta queda con los campos vacíos y deshabilitados y, con `UnidadesMedida:Actualizar`, sigue visible el botón «Editar» (no hay estado de error en la plantilla).

**Observaciones**
- **El mensaje de «en uso» no menciona las formulaciones clínicas**, que el backend también comprueba (`DeleteUnidadMedidaCommandHandler.cs:24-26`): el texto dice «asociada a medicamentos o jerarquías de unidades». Coincide con el hallazgo 8 de 01b.
- Las validaciones del formulario coinciden con las de la API en longitudes (10 y 50; `CreateUnidadMedidaCommandValidator.cs:13-19`, RN-UDM-002 a RN-UDM-005). La API además rechaza un código o descripción formados solo por espacios (`NotEmpty`), que `Validators.required` de Angular acepta; en ese caso el código de error es `{Request}.{Propiedad}` (por ejemplo `CreateUnidadMedidaCommand.Codigo`, `ValidationBehavior.cs:28`), que no está mapeado, y el usuario vería «Error al guardar la unidad de medida». Lo anterior sale de leer el código; no se ejecutó.
- Código sin uso: `goToNew()` (`.ts:172-174`).
- Claves del scope sin uso: ver §9.

### 1.3 `/unidades-medida/carga-masiva` — Carga Masiva (unidades de medida)

- **Componente:** `UnidadesMedidaCargaMasivaComponent` (`app/pages/unidades-medida/unidades-medida-carga-masiva/unidades-medida-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `unidadesMedida` (se declara pero no aporta textos: las etiquetas salen de la raíz).
- **Para qué sirve:** cargar varias unidades de medida desde un Excel. El flujo común (descargar plantilla, elegir archivo, vista previa, guardar válidos) está en [19a §6.5](19a-acceso-layout-y-compartidos.md); aquí solo la configuración que pasa esta pantalla.
- **Cómo se llega:** botón «Carga Masiva» de `/unidades-medida` (§1.1).
- **Permiso de ruta:** `UnidadesMedida:Leer` + `UnidadesMedida:Crear` (segundo segmento `carga-masiva`).
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Catálogos» / «Unidades de Medida» (enlace a `/unidades-medida`) / «Carga Masiva» (`breadcrumbs.carga-masiva`, activa); botón volver a `/unidades-medida` (`.ts:23-28`).

**Configuración que pasa** (`CargaMasivaConfig`, `.ts:18-33`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/unidadesmedida/plantilla` — `UnidadesMedida:Crear` (`UnidadesMedidaController.cs:67`); se baja como `plantilla-unidades-medida.xlsx` |
| `previewUrl` | `POST {api}/api/v1/unidadesmedida/previsualizar` — `UnidadesMedida:Crear` (`:76`) |
| `saveUrl` | `POST {api}/api/v1/unidadesmedida/carga-masiva` — `UnidadesMedida:Crear` (`:94`) |
| `backRoute` | `/unidades-medida` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz) | `codigo` → «Código» (`fields.codigo`, obligatoria); `descripcion` → «Descripción» (`fields.descripcion`, obligatoria). Después, la columna de errores sin encabezado y «Estado» |

**Botones, mensajes y errores:** los del componente común ([19a §6.5](19a-acceso-layout-y-compartidos.md) y §8.9). Los códigos de fila que puede traer la vista previa de unidades (reglas RN-UDM-015 y RN-UDM-016 de 01b) usan los textos de `carga-masiva.validation.*`.

**Observaciones:** las claves de datos (`codigo`, `descripcion`) coinciden con las propiedades de la fila de importación del backend (`UnidadMedidaImportRow(string Codigo, string Descripcion)`, `sofia-apis/src/SOFIA.Application/UnidadesMedida/Commands/CargaMasivaUnidadesMedida/CargaMasivaUnidadesMedidaCommand.cs:5`).

---

## 2. Jerarquías de unidades

Carpeta `app/pages/jerarquias`, scope i18n `jerarquias` (`provideTranslocoScope('jerarquias')` en los cuatro componentes). Componentes enrutados (`app/pages/jerarquias/jerarquias.routes.ts`, `loadChildren` en `app/app.routes.ts:81`): `JerarquiasSearchComponent` (`''`), `JerarquiasDetailComponent` (rutas `nueva` y `:id`) y `JerarquiasCargaMasivaComponent` (`carga-masiva`).

Una jerarquía es **una sola conversión de un producto**: «un producto, una unidad mayor, una unidad menor y un multiplicador» (cuántas unidades menores equivalen a una mayor). Cada registro es un par de unidades; la pantalla **no muestra ninguna cadena de conversión** ni calcula equivalencias. La cadena (por ejemplo Caja → Blíster → Unidad) la resuelve el backend al guardar para rechazar contradicciones (`sofia-apis/src/SOFIA.Application/JerarquiasUoM/Common/JerarquiaConversionResolver.cs`, usado en `CreateJerarquiaUoMCommandHandler.cs:38-59`) y al calcular las unidades vendibles de un producto, que consume la pantalla de Presentaciones de venta con `JerarquiaUoMService.getUnidadesVendibles` (`GET /api/v1/jerarquiasuom/unidades-vendibles?productoId=…`, `JerarquiasUoM:Leer`, `JerarquiasUoMController.cs:30`); ese método no lo llama ninguna pantalla de este módulo.

### 2.1 `/jerarquias` — Jerarquías de Unidades (búsqueda)

- **Componente:** `JerarquiasSearchComponent` (`app/pages/jerarquias/jerarquias-search/jerarquias-search.component.ts`, `.html`). Scope `jerarquias`. Importa `RangeSliderFieldComponent` ([19a §6.4](19a-acceso-layout-y-compartidos.md)).
- **Para qué sirve:** buscar jerarquías por producto, unidad mayor, unidad menor y rango de multiplicador, abrirlas, exportar la lista y entrar a crear o a cargar jerarquías masivamente.
- **Cómo se llega:** menú lateral, grupo «Catálogos» → «Jerarquías de Unidades» (`nav.jerarquiasUom`); botón volver y migas del detalle y de la carga masiva; redirección tras crear, editar o eliminar una jerarquía.
- **Permiso de ruta:** `JerarquiasUoM:Leer`. Sin permiso: aviso y redirección a `/dashboard`. Para que funcionen las listas de producto y de unidades hacen falta además `Medicamentos:Leer` y `UnidadesMedida:Leer` (ver «Llamadas a la API»); el SPA no lo comprueba.
- **Migas y cabecera:** título «Jerarquías de Unidades» (`jerarquias.title`); migas «Inicio» / «Catálogos» / «Jerarquías de Unidades» (`breadcrumbs.jerarquias`, activa); sin botón volver (`.html:3-6`, `.ts:70-74`). Tarjeta de filtros con título «Búsqueda de Jerarquías» (`jerarquias.search.title`) y el subtítulo común.
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:64-68`, `.html:37-136`)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Producto (`jerarquias.fields.producto`) | `productoNombre` | Texto con lupa y lista de autocompletado (`matAutocomplete`); placeholder «Buscar producto...» (`jerarquias.autocomplete.producto-placeholder`); Enter busca. Columna de 4/12 | Texto libre: se envía lo escrito aunque no elijas una opción. Las sugerencias salen de `GET /api/v1/medicamentos?nombreComercial=<texto>&pageNumber=1&pageSize=20` tras 300 ms sin escribir y solo con texto; cada opción muestra el nombre comercial | Vacío |
| Unidad Mayor (`jerarquias.fields.unidadMayor`) | `unidadMayorNombre` | Igual; placeholder «Buscar unidad de medida...» (`jerarquias.autocomplete.unidad-placeholder`) | Texto libre. Sugerencias de `GET /api/v1/unidadesmedida?descripcion=<texto>&pageNumber=1&pageSize=20`; cada opción muestra la descripción | Vacío |
| Unidad Menor (`jerarquias.fields.unidadMenor`) | `unidadMenorNombre` | Igual que Unidad Mayor | Igual | Vacío |
| Multiplicador (`jerarquias.fields.multiplicador`) | — (señales `multMin` y `multMax`, fuera del formulario) | Deslizador de rango de dos tiradores (`app-range-slider-field`), ocupa todo el ancho; la línea superior dice «Multiplicador: **inicio** — **fin**» | Mínimo 0, máximo 1000, paso 1 (valor por defecto del componente, `range-slider-field.component.ts:20`). Solo se envía el límite que se movió: `multiplicadorMin` si el inicio es mayor que 0, `multiplicadorMax` si el fin es menor que 1000 (`.ts:148-149`) | 0 — 1000 |

Cuando la lista de sugerencias queda vacía y el campo tiene texto, muestra una opción deshabilitada «Sin resultados» (`jerarquias.autocomplete.no-results`, `.html:55-62`). Las sugerencias solo se cargan al escribir: no hay carga al abrir la pantalla. Al restaurar el estado guardado el formulario se rellena sin disparar las sugerencias (`patchValue(..., { emitEvent: false })`, `.ts:124`). «Limpiar» (`actions.clear`) vacía los tres campos, devuelve el deslizador a 0 — 1000, vacía las tres listas de sugerencias, oculta la tabla y borra el estado guardado (`.ts:179-191`).

**Tabla de resultados** (`.html:162-211`, columnas `.ts:76`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Producto (`jerarquias.table.producto`) | `productoNombre` | Texto | Sí: `[routerLink]="[row.id]"` → `/jerarquias/:id` |
| Unidad Mayor (`jerarquias.table.unidadMayor`) | `unidadMayorNombre` | Texto | No |
| Unidad Menor (`jerarquias.table.unidadMenor`) | `unidadMenorNombre` | Texto | No |
| Multiplicador (`jerarquias.table.multiplicador`) | `multiplicador` | Número tal como lo devuelve la API, alineado a la derecha (encabezado y celdas) | No |

Orden por defecto: lo fija el servidor, por nombre comercial del producto (`GetJerarquiasUoMQueryHandler.cs:52`). El listado excluye las jerarquías cuyo producto o cuyas unidades estén eliminados (`:16`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Sin resultados» (`generic-labels.no-results`, `.html:208`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `JerarquiasUoM:Crear`, `.html:19`) | Nunca | `[routerLink]="['carga-masiva']"` → `/jerarquias/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/jerarquias/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/jerarquiasuom` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/jerarquiasuom/exportar` |
| Enlace con el nombre del producto | Primera columna | Con resultados | — | Navega a `/jerarquias/:id` |

Elegir una opción de una lista de sugerencias escribe su texto en el campo (`(click)` de la opción llama a `selectProducto`, `selectUnidadMayor` o `selectUnidadMenor`, `.ts:133-143`); no guarda ningún identificador.

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda o la exportación (cualquier error) | Error | «Error al buscar jerarquías de unidades» (`jerarquias.search.search-error`) | `jerarquias-search.component.ts:174,231` |

No hay mapa de códigos del backend. Un error al cargar las sugerencias no muestra ningún aviso: las suscripciones de `valueChanges` no tienen manejador de error (`.ts:78-114`).

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `JerarquiaUoMService.search` (`app/pages/jerarquias/services/jerarquia-uom.service.ts:19-29`) | `GET /api/v1/jerarquiasuom?pageNumber&pageSize[&productoNombre][&unidadMayorNombre][&unidadMenorNombre][&multiplicadorMin][&multiplicadorMax]` | `JerarquiasUoM:Leer` (`JerarquiasUoMController.cs:22`) |
| Escribir en «Producto» | `MedicamentoService.search` (`app/pages/medicamentos/services/medicamento.service.ts:23-38`) | `GET /api/v1/medicamentos?pageNumber=1&pageSize=20&nombreComercial=…` | `Medicamentos:Leer` (`MedicamentosController.cs:28`) |
| Escribir en «Unidad Mayor» o «Unidad Menor» | `UnidadMedidaService.search` (`unidad-medida.service.ts:19-30`) | `GET /api/v1/unidadesmedida?pageNumber=1&pageSize=20&descripcion=…` | `UnidadesMedida:Leer` (`UnidadesMedidaController.cs:19`) |
| Exportar | `JerarquiaUoMService.exportar` (`:51-62`) | `POST /api/v1/jerarquiasuom/exportar` (respuesta blob) | `JerarquiasUoM:Leer` (`JerarquiasUoMController.cs:78`) |

**Exportación**
- Cuerpo (`.ts:203-233`): `headers` con 4 textos traducidos, en orden: «Producto» (`jerarquias.table.producto`), «Unidad Mayor» (`jerarquias.table.unidadMayor`), «Unidad Menor» (`jerarquias.table.unidadMenor`), «Multiplicador» (`jerarquias.table.multiplicador`); más `productoNombre`, `unidadMayorNombre`, `unidadMenorNombre` si tienen valor y `multiplicadorMin` / `multiplicadorMax` si el deslizador se movió.
- Archivo: «jerarquias-unidades.xlsx» (`jerarquias.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §2.3.

**Comportamientos particulares**
- Restauración de filtros, incluidas las posiciones del deslizador (se guardan como `multMin` y `multMax` dentro de los valores del formulario, `.ts:167`), y «Buscar» sin volver a la página 1: ver §0.
- El producto y las unidades se filtran por **nombre** («contiene», sin distinguir mayúsculas; `GetJerarquiasUoMQueryHandler.cs:24-45`), no por identificador. Por eso el filtro vale aunque no elijas una sugerencia.
- Si no tienes `Medicamentos:Leer` o `UnidadesMedida:Leer`, los endpoints de sugerencias responden con error y esa lista no se llena; el resto de la pantalla sigue funcionando. Qué ve el usuario exactamente: «No verificado» (ver §7).

**Observaciones**
- Esta es la única de las cinco búsquedas que dice «Sin resultados» en la tabla vacía.
- El deslizador usa paso 1: no permite filtrar por multiplicadores con decimales ni por valores mayores a 1000 (con el fin en 1000 el límite superior no se envía).

### 2.2 `/jerarquias/nueva` y `/jerarquias/:id` — Jerarquías de Unidades (detalle)

- **Componente:** `JerarquiasDetailComponent` (`app/pages/jerarquias/jerarquias-detail/jerarquias-detail.component.ts`, `.html`). Scope `jerarquias`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear la conversión de un producto entre dos unidades, ver sus datos y, con permisos, editarla o eliminarla.
- **Cómo se llega:** `/jerarquias/nueva`: botón «Nuevo» de la búsqueda. `/jerarquias/:id`: enlace del producto en la tabla. No hay otro enlace (ninguna otra pantalla navega a `/jerarquias`).
- **Permiso de ruta:** `/jerarquias/nueva` → `JerarquiasUoM:Leer` + `JerarquiasUoM:Crear`; `/jerarquias/:id` → `JerarquiasUoM:Leer`. Además, para elegir producto y unidades hacen falta `Medicamentos:Leer` y `UnidadesMedida:Leer` (el SPA no lo comprueba).
- **Migas y cabecera:** título «Jerarquías de Unidades» (`jerarquias.title`); migas «Inicio» / «Catálogos» / «Jerarquías de Unidades» (con enlace a `/jerarquias`); botón volver a `/jerarquias` (`.html:3-8`, `.ts:91-95`).
- **Modos** (`ngOnInit`, `.ts:135-148`): sin parámetro `id` → **crear**; con `id` → **ver** (el formulario y los tres campos de búsqueda se deshabilitan y se carga el registro) y «Editar» los habilita (§0).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado de la tarjeta «Jerarquías de Unidades» + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `JerarquiasUoM:Actualizar`) | No |
| Producto, Unidad Mayor, Unidad Menor, Multiplicador | Editables | Deshabilitados, con el nombre cargado | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Cómo se eligen el producto y las unidades.** Producto, Unidad Mayor y Unidad Menor son tres campos de texto con lista de sugerencias. Al escribir (300 ms sin teclear) se consulta al servidor y la lista muestra hasta 20 resultados: productos por nombre comercial (`MedicamentoService.search`, `Medicamentos:Leer`) y unidades por descripción (`UnidadMedidaService.search`, `UnidadesMedida:Leer`) (`.ts:97-133`). Los dos campos de unidad hacen la misma consulta y no excluyen de la lista la unidad elegida en el otro campo. Elegir una opción guarda su identificador y su nombre (`selectProducto`, `selectMayor`, `selectMenor`, `.ts:177-191`). Al guardar, si el texto del campo ya no coincide con el nombre de la opción elegida, la elección se descarta y el campo se marca como obligatorio (`.ts:222-224`). Hay que **elegir una opción de la lista**; escribir el nombre a mano sin elegirla no basta si cambiaste el texto. Si el campo tiene texto y la lista está vacía, aparece una opción deshabilitada «Sin resultados» (`jerarquias.autocomplete.no-results`). La opción se selecciona con `(click)` sobre la opción (`.html:47-53,77-83,107-113`); si la selección con teclado (flechas y Enter) dispara ese mismo evento: «No verificado» (ver §7).

**Campos del formulario** (`.ts:87-89`, `.html:38-148`)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Producto (`jerarquias.fields.producto`) | `productoCtrl` (control suelto, fuera del grupo `form`); columna de 12/12 | Texto con lista de sugerencias; placeholder «Buscar producto...» (`jerarquias.autocomplete.producto-placeholder`) | Sí | Debe haber una opción elegida (`selectedProducto()`), comprobada al guardar (`.ts:222`, `.html:61-65`) | «Este campo es obligatorio» (`general.errors.required`) bajo el campo | Sí / Sí |
| Unidad Mayor (`jerarquias.fields.unidadMayor`) | `mayorCtrl`; columna de 3/12 | Texto con sugerencias; placeholder «Buscar unidad de medida...» (`jerarquias.autocomplete.unidad-placeholder`) | Sí | Igual (`selectedMayor()`) | «Este campo es obligatorio» | Sí / Sí |
| Unidad Menor (`jerarquias.fields.unidadMenor`) | `menorCtrl`; columna de 3/12 | Igual | Sí | Igual (`selectedMenor()`) | «Este campo es obligatorio» | Sí / Sí |
| Multiplicador (`jerarquias.fields.multiplicador`) | `multiplicador` (grupo `form`); columna de 6/12 | `matInput` `type="number"`, `min="0"`, `step="0.01"` | Sí | `Validators.required`, `Validators.min(0.0001)` (`.ts:88`) | «El multiplicador debe ser mayor que cero» (`jerarquias.detail.multiplicador-error`; el mismo texto para vacío y para menor que 0,0001) | Sí / Sí |

- Ayuda bajo el multiplicador: «Cuántas unidades menores equivalen a una unidad mayor» (`jerarquias.detail.multiplicador-hint`, `.html:137`).
- Regla cruzada (unidades distintas): si las dos unidades elegidas son la misma, en modo crear o editar aparece bajo el formulario «La unidad mayor y la unidad menor no pueden ser la misma» (`jerarquias.detail.same-unidad-error`, `.html:142-148`; cálculo `sameUnidad`, `.ts:76-78`) y `save()` no envía.
- Los errores de los tres campos con lista se muestran solo después de pulsar «Guardar» (`productoTouched`, `mayorTouched`, `menorTouched` se ponen en verdadero en `save()`, `.ts:218-220`).
- No hay nada que muestre una cadena de conversión, una vista previa del cálculo ni las otras jerarquías del mismo producto.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `JerarquiasUoM:Actualizar`, `.html:27`) | Nunca | `enterEditMode()`: habilita el formulario y los tres campos de lista |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:200-214`): en crear navega a `/jerarquias`; en editar vuelve a poner los datos leídos (`applySnapshot`), deshabilita todo y limpia las marcas de campo tocado |
| Guardar (`actions.save`); con spinner mientras guarda | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:216-258`). Sin enviar si el multiplicador es inválido, falta alguna de las tres elecciones o las unidades son iguales. Crear → `POST /api/v1/jerarquiasuom`; editar → `PUT /api/v1/jerarquiasuom/:id`. En ambos casos, éxito → aviso y navega a `/jerarquias` |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de una jerarquía existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `JerarquiasUoM:Eliminar`, `.html:169`) | `deleting()` | `confirmDelete()` → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/jerarquias` |

El cuerpo enviado al guardar es `{ productoId, unidadMayorId, unidadMenorId, multiplicador }` (`.ts:237-242`).

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar» (`.ts:260-271`): título «Confirmar eliminación» (`jerarquias.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta jerarquía?» (`jerarquias.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/jerarquiasuom/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Jerarquía guardada correctamente» (`jerarquias.detail.save-success`); navega a `/jerarquias` | `.ts:245-246` |
| Código `JerarquiaUoM.Duplicada` (409) | Error | «Este producto ya tiene una conversión entre esas unidades.» (`jerarquias.detail.duplicate`) | `.ts:19-21,250` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en §0 | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar (incluidos `JerarquiaUoM.Multiplicador.Inconsistente`, `Medicamento.NotFound` y `UnidadMedida.NotFound`, que la API sí devuelve y el SPA no mapea) | Error | «Error al guardar la jerarquía» (`jerarquias.detail.save-error`) | `.ts:250` |
| Eliminación correcta | Éxito | «Jerarquía eliminada correctamente» (`jerarquias.detail.delete-success`); navega a `/jerarquias` | `.ts:276-278` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No se puede eliminar la jerarquía porque está en uso.» (`jerarquias.detail.delete-in-use`) | `.ts:283-286` |
| Eliminar: cualquier otro error | Error | «Error al eliminar la jerarquía» (`jerarquias.detail.delete-error`) | `.ts:283-286` |
| Error al cargar | Error | «Error al cargar la jerarquía» (`jerarquias.detail.load-error`) | `.ts:160` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir | `JerarquiaUoMService.getById` (`jerarquia-uom.service.ts:31-33`) | `GET /api/v1/jerarquiasuom/{id}` | `JerarquiasUoM:Leer` (`JerarquiasUoMController.cs:38`) |
| Crear | `create` | `POST /api/v1/jerarquiasuom` | `JerarquiasUoM:Crear` (`:46`) |
| Editar | `update` | `PUT /api/v1/jerarquiasuom/{id}` | `JerarquiasUoM:Actualizar` (`:56`) |
| Eliminar | `delete` | `DELETE /api/v1/jerarquiasuom/{id}` | `JerarquiasUoM:Eliminar` (`:70`) |
| Escribir en «Producto» | `MedicamentoService.search` | `GET /api/v1/medicamentos?pageNumber=1&pageSize=20&nombreComercial=…` | `Medicamentos:Leer` (`MedicamentosController.cs:28`) |
| Escribir en «Unidad Mayor» o «Unidad Menor» | `UnidadMedidaService.search` | `GET /api/v1/unidadesmedida?pageNumber=1&pageSize=20&descripcion=…` | `UnidadesMedida:Leer` (`UnidadesMedidaController.cs:19`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- A diferencia de Unidades de medida, Laboratorios e Ingredientes activos, **editar con éxito también navega a la búsqueda** (no se queda en la ficha).
- Después de abrir una jerarquía las listas de sugerencias están vacías hasta que escribas: la lista se carga solo con el texto que teclees. En el detalle, a diferencia de la búsqueda, una búsqueda con el campo vacío sí se envía (devuelve los primeros 20) (`.ts:102-104`, comparar con `jerarquias-search.component.ts:84`).
- Si falla la carga de sugerencias no hay aviso (suscripciones sin manejador de error, `.ts:97-133`).
- La API valida en el servidor lo que el formulario no ve: producto y unidades deben existir en la empresa (`Medicamento.NotFound`, `UnidadMedida.NotFound`, `CreateJerarquiaUoMCommandHandler.cs:27-36`), no se puede repetir la misma conversión del producto (`JerarquiaUoM.Duplicada`, `:43-46`) y la equivalencia no puede contradecir la que ya resulta de otras conversiones del mismo producto, con tolerancia de 4 decimales (`JerarquiaUoM.Multiplicador.Inconsistente`, `:51-59`). Para el usuario, solo la segunda tiene mensaje propio; las otras muestran el genérico.
- La API no valida dependencias al eliminar una jerarquía (`DeleteJerarquiaUoMCommandHandler.cs:12-24`: solo `JerarquiaUoM.NotFound` o éxito), por lo que el mensaje «está en uso» no tiene hoy un caso real en el backend; queda por el tratamiento genérico de cualquier 409.

**Observaciones**
- Discrepancia formulario / API en el multiplicador: el formulario exige al menos 0,0001 (`Validators.min(0.0001)`); la API exige mayor que 0 (`CreateJerarquiaUoMCommandValidator.cs:16`) y la columna guarda 4 decimales (RN-JER-005 de 01b). Con `step="0.01"` el selector del navegador avanza de a 0,01, pero se pueden teclear más decimales.
- El atributo `min="0"` del `<input type="number">` (`.html:135`) no cambia la regla efectiva: el mínimo del formulario sigue siendo 0,0001.
- El texto «Sin resultados» de las sugerencias (`jerarquias.autocomplete.no-results`) también aparece mientras la primera consulta de sugerencias aún no ha terminado de cargar, porque la condición solo mira que la lista esté vacía y el campo tenga texto (`.html:55-59`).
- Cambiar el producto en una jerarquía existente es posible (el `PUT` reenvía `productoId`); la API solo revalida el producto si cambió (`UpdateJerarquiaUoMCommandHandler.cs:23-27`).
- Todos los textos de esta pantalla están en i18n; no hay textos en duro.

### 2.3 `/jerarquias/carga-masiva` — Carga Masiva (jerarquías)

- **Componente:** `JerarquiasCargaMasivaComponent` (`app/pages/jerarquias/jerarquias-carga-masiva/jerarquias-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `jerarquias` (no aporta textos).
- **Para qué sirve:** cargar varias jerarquías desde un Excel. El flujo común está en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/jerarquias` (§2.1).
- **Permiso de ruta:** `JerarquiasUoM:Leer` + `JerarquiasUoM:Crear`.
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Catálogos» / «Jerarquías de Unidades» (enlace a `/jerarquias`) / «Carga Masiva» (activa); botón volver a `/jerarquias`.

**Configuración que pasa** (`.ts:18-36`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/jerarquiasuom/plantilla` — `JerarquiasUoM:Crear` (`JerarquiasUoMController.cs:113`); se baja como `plantilla-jerarquias.xlsx` |
| `previewUrl` | `POST {api}/api/v1/jerarquiasuom/previsualizar` — `JerarquiasUoM:Crear` (`:121`) |
| `saveUrl` | `POST {api}/api/v1/jerarquiasuom/carga-masiva` — `JerarquiasUoM:Crear` (`:138`) |
| `backRoute` | `/jerarquias` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz; todas obligatorias) | `producto` → «Producto» (`fields.producto`); `unidadMayor` → «Unidad Mayor» (`fields.unidadMayor`); `unidadMenor` → «Unidad Menor» (`fields.unidadMenor`); `multiplicador` → «Multiplicador» (`fields.multiplicador`). Después, la columna de errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Las reglas de la carga (el producto se identifica por nombre comercial y las unidades por descripción; la vista previa no anticipa duplicados ni multiplicadores inconsistentes) están en RN-JER-018 a RN-JER-021 de 01b. Las claves de datos coinciden con `JerarquiaUoMImportRow(Producto, UnidadMayor, UnidadMenor, Multiplicador)` (`CargaMasivaJerarquiasUoMCommand.cs:5-9`).

---

## 3. Laboratorios

Carpeta `app/pages/laboratorios`, scope i18n `laboratorios` (`provideTranslocoScope('laboratorios')` en los cuatro componentes). Componentes enrutados (`app/pages/laboratorios/laboratorios.routes.ts`, `loadChildren` en `app/app.routes.ts:86`): `LaboratoriosSearchComponent` (`''`), `LaboratoriosDetailComponent` (rutas `nueva` y `:id`) y `LaboratoriosCargaMasivaComponent` (`carga-masiva`). Misma estructura que Unidades de medida (§1); aquí solo lo propio.

### 3.1 `/laboratorios` — Laboratorios (búsqueda)

- **Componente:** `LaboratoriosSearchComponent` (`app/pages/laboratorios/laboratorios-search/laboratorios-search.component.ts`, `.html`). Scope `laboratorios`.
- **Para qué sirve:** buscar laboratorios por nombre de compañía y código identificador, abrirlos, exportar la lista y entrar a crear o a cargar laboratorios masivamente.
- **Cómo se llega:** menú lateral, grupo «Catálogos» → «Laboratorios» (`nav.laboratorios`); botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar un laboratorio.
- **Permiso de ruta:** `Laboratorios:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Laboratorios» (`laboratorios.title`); migas «Inicio» / «Catálogos» / «Laboratorios» (`breadcrumbs.laboratorios`, activa); sin botón volver (`.html:3-6`, `.ts:47-51`). Tarjeta de filtros con título «Búsqueda de Laboratorios» (`laboratorios.search.title`) y el subtítulo común.
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:45`, `.html:37-58`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Nombre de Compañía (`laboratorios.fields.nombreCompania`) | `nombreCompania` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Código Identificador (`laboratorios.fields.codigoIdentificador`) | `codigoIdentificador` | Texto con lupa; Enter busca | Texto libre | Vacío |

**Tabla de resultados** (`.html:84-115`, columnas `.ts:53`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Nombre de Compañía (`laboratorios.table.nombreCompania`) | `nombreCompania` | Texto | Sí: `[routerLink]="[row.id]"` → `/laboratorios/:id` |
| Código Identificador (`laboratorios.table.codigoIdentificador`) | `codigoIdentificador` | Texto; «—» si es nulo (`.html:101`) | No |

Orden por defecto: lo fija el servidor, por nombre de compañía y luego por identificador (`sofia-apis/src/SOFIA.Application/Laboratorios/Queries/GetLaboratorios/GetLaboratoriosQueryHandler.cs:28`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:112`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `Laboratorios:Crear`, `.html:19`) | Nunca | `[routerLink]="['carga-masiva']"` → `/laboratorios/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/laboratorios/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/laboratorios` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/laboratorios/exportar` |
| Enlace con el nombre | Primera columna | Con resultados | — | Navega a `/laboratorios/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar laboratorios» (`laboratorios.search.search-error`) | `laboratorios-search.component.ts:93` |
| Falla la exportación (cualquier error) | Error | El mismo texto, sin referencia de seguimiento (usa `showError`, no `showServerError`) | `laboratorios-search.component.ts:137` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `LaboratorioService.search` (`app/pages/laboratorios/services/laboratorio.service.ts:19-30`) | `GET /api/v1/laboratorios?pageNumber&pageSize[&nombreCompania][&codigoIdentificador]` | `Laboratorios:Leer` (`LaboratoriosController.cs:19`) |
| Exportar | `LaboratorioService.exportar` (`:48-50`) | `POST /api/v1/laboratorios/exportar` (respuesta blob) | `Laboratorios:Leer` (`LaboratoriosController.cs:67`) |

**Exportación**
- Cuerpo (`.ts:117-127`): `headers` con 2 textos traducidos, en orden: «Nombre de Compañía» (`laboratorios.table.nombreCompania`) y «Código Identificador» (`laboratorios.table.codigoIdentificador`); más los dos filtros si tienen valor.
- Archivo: «laboratorios.xlsx» (`laboratorios.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §3.3.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0.

**Observaciones:** la fila sin resultados dice «Resultados» y no «Sin resultados» (`laboratorios-search.component.html:112`). Los specs `laboratorios-search.component.spec.ts` (4 casos) cubren la visibilidad de «Nuevo», «Carga Masiva» y «Exportar» según permisos.

### 3.2 `/laboratorios/nueva` y `/laboratorios/:id` — Laboratorios (detalle)

- **Componente:** `LaboratoriosDetailComponent` (`app/pages/laboratorios/laboratorios-detail/laboratorios-detail.component.ts`, `.html`). Scope `laboratorios`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un laboratorio, ver sus datos y, con permisos, editarlo o eliminarlo.
- **Cómo se llega:** `/laboratorios/nueva`: botón «Nuevo» de la búsqueda. `/laboratorios/:id`: enlace del nombre en la tabla.
- **Permiso de ruta:** `/laboratorios/nueva` → `Laboratorios:Leer` + `Laboratorios:Crear`; `/laboratorios/:id` → `Laboratorios:Leer`.
- **Migas y cabecera:** título «Laboratorios» (`laboratorios.title`); migas «Inicio» / «Catálogos» / «Laboratorios» (con enlace a `/laboratorios`); botón volver a `/laboratorios` (`.html:3-7`, `.ts:60-64`).
- **Modos** (`ngOnInit`, `.ts:66-76`): sin parámetro `id` → **crear**; con `id` → **ver** y «Editar» habilita el formulario (§0).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Laboratorios» + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `Laboratorios:Actualizar`) | No |
| Nombre de Compañía y Código Identificador | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:55-58`, `.html:38-58`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Nombre de Compañía (`laboratorios.fields.nombreCompania`) | `nombreCompania` | `matInput`, `maxlength="150"`, columna de 8/12 | Sí | `Validators.required`, `Validators.maxLength(150)` (`.ts:56`) | «Este campo es obligatorio» (`general.errors.required`, `.html:45`) | Sí / Sí |
| Código Identificador (`laboratorios.fields.codigoIdentificador`) | `codigoIdentificador` | `matInput`, `maxlength="50"`, columna de 4/12 | No | `Validators.maxLength(50)` (`.ts:57`) | Sin `mat-error` (el atributo `maxlength` impide pasar de 50) | Sí / Sí |

Sin ayuda ni placeholder (las claves `laboratorios.help.*` no se usan). El cuerpo enviado al guardar es `{ nombreCompania, codigoIdentificador }`; si nunca tocaste el código se omite (`null ?? undefined`); si lo escribiste y lo borraste se envía como cadena vacía (`'' ?? undefined` conserva la cadena, `.ts:115-118`) y la API en ambos casos lo guarda como «sin código» (`Laboratorio.NormalizarCodigo`, `sofia-apis/src/SOFIA.Domain/Entities/Laboratorio.cs:63-64`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Laboratorios:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` (`.ts:93`) |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:98-106`): en crear navega a `/laboratorios`; en editar restaura los datos leídos y vuelve a ver |
| Guardar (`actions.save`); con spinner mientras guarda | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:108-145`). Inválido → marca todo como tocado y no envía. Crear → `POST /api/v1/laboratorios`, aviso y navega a `/laboratorios`. Editar → `PUT /api/v1/laboratorios/:id`, aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de un laboratorio existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `Laboratorios:Eliminar`, `.html:79`) | `deleting()` | `confirmDelete()` (`.ts:147`) → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/laboratorios` |

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar»: título «Confirmar eliminación» (`laboratorios.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este laboratorio?» (`laboratorios.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/laboratorios/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Laboratorio guardado correctamente» (`laboratorios.detail.save-success`) | `.ts:122,133` |
| Código `Laboratorio.NombreCompania.Duplicado` (409) | Error | «Ya existe un laboratorio con ese nombre de compañía.» (`laboratorios.detail.duplicate-nombre`) | `.ts:17-20,127,141` |
| Código `Laboratorio.CodigoIdentificador.Duplicado` (409) | Error | «Ya existe un laboratorio con ese código identificador.» (`laboratorios.detail.duplicate-codigo`) | `.ts:17-20,127,141` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en §0 | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar | Error | «Error al guardar el laboratorio» (`laboratorios.detail.save-error`) | `.ts:127,141` |
| Eliminación correcta | Éxito | «Laboratorio eliminado correctamente» (`laboratorios.detail.delete-success`); navega a `/laboratorios` | `.ts:164-165` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No se puede eliminar el laboratorio porque tiene medicamentos asociados.» (`laboratorios.detail.delete-in-use`) | `.ts:169-172` |
| Eliminar: cualquier otro error | Error | «Error al eliminar el laboratorio» (`laboratorios.detail.delete-error`) | `.ts:169-172` |
| Error al cargar | Error | «Error al cargar el laboratorio» (`laboratorios.detail.load-error`) | `.ts:88` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir y tras editar | `LaboratorioService.getById` (`laboratorio.service.ts:32-34`) | `GET /api/v1/laboratorios/{id}` | `Laboratorios:Leer` (`LaboratoriosController.cs:27`) |
| Crear | `create` | `POST /api/v1/laboratorios` | `Laboratorios:Crear` (`:35`) |
| Editar | `update` | `PUT /api/v1/laboratorios/{id}` | `Laboratorios:Actualizar` (`:45`) |
| Eliminar | `delete` | `DELETE /api/v1/laboratorios/{id}` | `Laboratorios:Eliminar` (`:59`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga muestra un spinner en lugar de la tarjeta (`.html:9-13`).
- El 409 de eliminar se trata como «en uso» sin mirar el código; la API devuelve `Laboratorio.InUse` cuando algún medicamento no eliminado usa el laboratorio (`sofia-apis/src/SOFIA.Application/Laboratorios/Commands/DeleteLaboratorio/DeleteLaboratorioCommandHandler.cs:20-27`).
- Si falla la carga del registro, la ficha queda vacía y deshabilitada y, con `Laboratorios:Actualizar`, sigue visible «Editar».

**Observaciones**
- Las validaciones del formulario coinciden con las de la API (150 y 50; `CreateLaboratorioCommandValidator.cs:13-18`, RN-LAB-002 a RN-LAB-004). Un nombre de compañía de solo espacios pasa `Validators.required` pero la API lo rechaza (`NotEmpty`) con un código `{Request}.{Propiedad}` no mapeado; el usuario vería «Error al guardar el laboratorio». Sale de leer el código; no se ejecutó.
- Código sin uso: `goToNew()` (`.ts:177-179`).
- Los specs `laboratorios-detail.component.spec.ts` (3 casos) cubren la visibilidad de «Editar» y «Eliminar» según permisos.
- Claves del scope sin uso: ver §9.

### 3.3 `/laboratorios/carga-masiva` — Carga Masiva (laboratorios)

- **Componente:** `LaboratoriosCargaMasivaComponent` (`app/pages/laboratorios/laboratorios-carga-masiva/laboratorios-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `laboratorios` (no aporta textos).
- **Para qué sirve:** cargar varios laboratorios desde un Excel; el flujo común está en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/laboratorios` (§3.1).
- **Permiso de ruta:** `Laboratorios:Leer` + `Laboratorios:Crear`.
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Catálogos» / «Laboratorios» (enlace a `/laboratorios`) / «Carga Masiva» (activa); botón volver a `/laboratorios`.

**Configuración que pasa** (`.ts:18-33`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/laboratorios/plantilla` — `Laboratorios:Crear` (`LaboratoriosController.cs:97`); se baja como `plantilla-laboratorios.xlsx` |
| `previewUrl` | `POST {api}/api/v1/laboratorios/previsualizar` — `Laboratorios:Crear` (`:106`) |
| `saveUrl` | `POST {api}/api/v1/laboratorios/carga-masiva` — `Laboratorios:Crear` (`:124`) |
| `backRoute` | `/laboratorios` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz) | `nombreCompania` → «Nombre de Compañía» (`fields.nombreCompania`, obligatoria); `codigoIdentificador` → «Código Identificador» (`fields.codigoIdentificador`, **no** obligatoria). Después, la columna de errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Reglas de la carga en RN-LAB-013 a RN-LAB-017 de 01b. Las claves de datos coinciden con `LaboratorioImportRow(NombreCompania, CodigoIdentificador)` (`CargaMasivaLaboratoriosCommand.cs:5`).

---

## 4. Ingredientes activos

Carpeta `app/pages/ingredientes-activos`, scope i18n `ingredientesActivos` (`provideTranslocoScope('ingredientesActivos')` en los cuatro componentes). Componentes enrutados (`app/pages/ingredientes-activos/ingredientes-activos.routes.ts`, `loadChildren` en `app/app.routes.ts:91`): `IngredientesActivosSearchComponent` (`''`), `IngredientesActivosDetailComponent` (rutas `nueva` y `:id`) y `IngredientesActivosCargaMasivaComponent` (`carga-masiva`). Misma estructura que Unidades de medida (§1). Este es el módulo cuya eliminación recibió la validación de dependencias de los commits `27597cd` (API) y `01b8260` (SPA): ver «Mensajes al usuario» de §4.2 y §6.

### 4.1 `/ingredientes-activos` — Ingredientes Activos (búsqueda)

- **Componente:** `IngredientesActivosSearchComponent` (`app/pages/ingredientes-activos/ingredientes-activos-search/ingredientes-activos-search.component.ts`, `.html`). Scope `ingredientesActivos`.
- **Para qué sirve:** buscar ingredientes activos por denominación DCI y código ATC, abrirlos, exportar la lista y entrar a crear o a cargar ingredientes masivamente.
- **Cómo se llega:** menú lateral, grupo «Catálogos» → «Ingredientes Activos» (`nav.ingredientesActivos`); botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar un ingrediente.
- **Permiso de ruta:** `IngredientesActivos:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Ingredientes Activos» (`ingredientesActivos.title`); migas «Inicio» / «Catálogos» / «Ingredientes Activos» (`breadcrumbs.ingredientes-activos`, activa); sin botón volver (`.html:4-7`, `.ts:47-51`). Tarjeta de filtros con título «Búsqueda de Ingredientes Activos» (`ingredientesActivos.search.title`) y el subtítulo común.
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:45`, `.html:38-59`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Denominación DCI (`ingredientesActivos.fields.denominacionDci`) | `denominacionDci` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Código ATC (`ingredientesActivos.fields.codigoAtc`) | `codigoAtc` | Texto con lupa; Enter busca | Texto libre | Vacío |

**Tabla de resultados** (`.html:85-114`, columnas `.ts:53`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Denominación DCI (`ingredientesActivos.table.denominacionDci`) | `denominacionDci` | Texto | Sí: `[routerLink]="[row.id]"` → `/ingredientes-activos/:id` |
| Código ATC (`ingredientesActivos.table.codigoAtc`) | `codigoAtc` | Texto | No |

Orden por defecto: lo fija el servidor, por denominación DCI y luego por identificador (`sofia-apis/src/SOFIA.Application/IngredientesActivos/Queries/GetIngredientesActivos/GetIngredientesActivosQueryHandler.cs:28`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:111`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `IngredientesActivos:Crear`, `.html:20`) | Nunca | `[routerLink]="['carga-masiva']"` → `/ingredientes-activos/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:28`) | Nunca | `goToNew()` → `/ingredientes-activos/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/ingredientesactivos` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/ingredientesactivos/exportar` |
| Enlace con la denominación | Primera columna | Con resultados | — | Navega a `/ingredientes-activos/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar ingredientes activos» (`ingredientesActivos.search.search-error`) | `ingredientes-activos-search.component.ts:93` |
| Falla la exportación | Error | El mismo texto, con referencia de seguimiento si la respuesta trae `traceId` (`showServerError`) | `ingredientes-activos-search.component.ts:137` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `IngredienteActivoService.search` (`app/pages/ingredientes-activos/services/ingrediente-activo.service.ts:19-30`) | `GET /api/v1/ingredientesactivos?pageNumber&pageSize[&denominacionDci][&codigoAtc]` | `IngredientesActivos:Leer` (`IngredientesActivosController.cs:19`) |
| Exportar | `IngredienteActivoService.exportar` (`:48-50`) | `POST /api/v1/ingredientesactivos/exportar` (respuesta blob) | `IngredientesActivos:Leer` (`IngredientesActivosController.cs:67`) |

**Exportación**
- Cuerpo (`.ts:117-127`): `headers` con 2 textos traducidos, en orden: «Denominación DCI» (`ingredientesActivos.table.denominacionDci`) y «Código ATC» (`ingredientesActivos.table.codigoAtc`); más los dos filtros si tienen valor.
- Archivo: «ingredientes-activos.xlsx» (`ingredientesActivos.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §4.3.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0.

**Observaciones:** la fila sin resultados dice «Resultados» y no «Sin resultados» (`ingredientes-activos-search.component.html:111`).

### 4.2 `/ingredientes-activos/nueva` y `/ingredientes-activos/:id` — Ingredientes Activos (detalle)

- **Componente:** `IngredientesActivosDetailComponent` (`app/pages/ingredientes-activos/ingredientes-activos-detail/ingredientes-activos-detail.component.ts`, `.html`). Scope `ingredientesActivos`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un ingrediente activo, ver sus datos y, con permisos, editarlo o eliminarlo.
- **Cómo se llega:** `/ingredientes-activos/nueva`: botón «Nuevo» de la búsqueda. `/ingredientes-activos/:id`: enlace de la denominación en la tabla.
- **Permiso de ruta:** `/ingredientes-activos/nueva` → `IngredientesActivos:Leer` + `IngredientesActivos:Crear`; `/ingredientes-activos/:id` → `IngredientesActivos:Leer`.
- **Migas y cabecera:** título «Ingredientes Activos» (`ingredientesActivos.title`); migas «Inicio» / «Catálogos» / «Ingredientes Activos» (con enlace a `/ingredientes-activos`); botón volver a `/ingredientes-activos` (`.html:4-8`, `.ts:59-63`).
- **Modos** (`ngOnInit`, `.ts:65-75`): sin parámetro `id` → **crear**; con `id` → **ver** y «Editar» habilita el formulario (§0).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Ingredientes Activos» + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `IngredientesActivos:Actualizar`) | No |
| Denominación DCI y Código ATC | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:54-57`, `.html:39-57`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Denominación DCI (`ingredientesActivos.fields.denominacionDci`) | `denominacionDci` | `matInput`, `maxlength="255"`, columna de 8/12 | Sí | `Validators.required`, `Validators.maxLength(255)` (`.ts:55`) | «Este campo es obligatorio» (`general.errors.required`, `.html:46`) | Sí / Sí |
| Código ATC (`ingredientesActivos.fields.codigoAtc`) | `codigoAtc` | `matInput`, `maxlength="15"`, columna de 4/12 | Sí | `Validators.required`, `Validators.maxLength(15)` (`.ts:56`) | «Este campo es obligatorio» (`general.errors.required`, `.html:54`) | Sí / Sí |

Sin ayuda ni placeholder. No se valida la estructura del código ATC (cualquier texto de hasta 15 caracteres), igual que la API (RN-ING-005 de 01b).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `IngredientesActivos:Actualizar`, `.html:28`) | Nunca | `enterEditMode()` (`.ts:92`) |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:97-105`): en crear navega a `/ingredientes-activos`; en editar restaura los datos leídos y vuelve a ver |
| Guardar (`actions.save`); con spinner mientras guarda | Pie, derecha | Crear o editar | `saving()` | `save()` (`.ts:107-141`). Inválido → marca todo como tocado y no envía. Crear → `POST /api/v1/ingredientesactivos`, aviso y navega a `/ingredientes-activos`. Editar → `PUT /api/v1/ingredientesactivos/:id`, aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de un ingrediente existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `IngredientesActivos:Eliminar`, `.html:78`) | `deleting()` | `confirmDelete()` (`.ts:143`) → diálogo |
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/ingredientes-activos` |

**Diálogos que abre:** `ConfirmDialogComponent` al pulsar «Eliminar» (`.ts:143-154`): título «Confirmar eliminación» (`ingredientesActivos.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este ingrediente activo?» (`ingredientesActivos.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/ingredientesactivos/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Ingrediente activo guardado correctamente» (`ingredientesActivos.detail.save-success`) | `.ts:118,129` |
| Código `IngredienteActivo.DenominacionDci.Duplicado` (409) | Error | «Ya existe un ingrediente activo con esa denominación DCI.» (`ingredientesActivos.detail.duplicate-dci`) | `.ts:17-19,123,137` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos de `errors.db*` descritos en §0 | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar | Error | «Error al guardar el ingrediente activo» (`ingredientesActivos.detail.save-error`) | `.ts:123,137` |
| Eliminación correcta | Éxito | «Ingrediente activo eliminado correctamente» (`ingredientesActivos.detail.delete-success`); navega a `/ingredientes-activos` | `.ts:160-161` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» (`ingredientesActivos.detail.delete-in-use`) | `.ts:165-168` |
| Eliminar: cualquier otro error | Error | «Error al eliminar el ingrediente activo» (`ingredientesActivos.detail.delete-error`) | `.ts:165-168` |
| Error al cargar | Error | «Error al cargar el ingrediente activo» (`ingredientesActivos.detail.load-error`) | `.ts:87` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir y tras editar | `IngredienteActivoService.getById` (`ingrediente-activo.service.ts:32-34`) | `GET /api/v1/ingredientesactivos/{id}` | `IngredientesActivos:Leer` (`IngredientesActivosController.cs:27`) |
| Crear | `create` | `POST /api/v1/ingredientesactivos` | `IngredientesActivos:Crear` (`:35`) |
| Editar | `update` | `PUT /api/v1/ingredientesactivos/{id}` | `IngredientesActivos:Actualizar` (`:45`) |
| Eliminar | `delete` | `DELETE /api/v1/ingredientesactivos/{id}` | `IngredientesActivos:Eliminar` (`:59`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga muestra un spinner en lugar de la tarjeta (`.html:10-14`).
- **Eliminar con dependencias.** Desde `27597cd` la API rechaza con 409 y código `IngredienteActivo.InUse` eliminar un ingrediente que esté en alguna formulación clínica no eliminada (`sofia-apis/src/SOFIA.Application/IngredientesActivos/Commands/DeleteIngredienteActivo/DeleteIngredienteActivoCommandHandler.cs:20-27`). Desde `01b8260` el SPA traduce cualquier 409 al mensaje «…incluido en formulaciones clínicas» (`.ts:165-168`). El SPA no mira el código de la respuesta, solo el estado HTTP. Los specs `ingredientes-activos-detail.delete.spec.ts` (con el ayudante compartido `app/shared/testing/delete-in-use-testing.ts`) cubren el 409 y el caso de otro estado (500).
- Si falla la carga del registro, la ficha queda vacía y deshabilitada y, con `IngredientesActivos:Actualizar`, sigue visible «Editar».

**Observaciones**
- Las validaciones del formulario coinciden con las de la API (255 y 15; ambos obligatorios; `CreateIngredienteActivoCommandValidator.cs:13-19`, RN-ING-002 a RN-ING-005). Valores de solo espacios pasan `Validators.required` pero la API los rechaza con un código `{Request}.{Propiedad}` no mapeado y el usuario vería «Error al guardar el ingrediente activo». Sale de leer el código; no se ejecutó.
- **Documento 01b desactualizado en este punto:** RN-ING-009 de 01b afirma que la eliminación no valida dependencias; con `27597cd` sí lo hace (ver la línea base en `REGLAS-AGENTES.md` §3).
- Código sin uso: `goToNew()` (`.ts:173-175`).
- Claves del scope sin uso: ver §9.

### 4.3 `/ingredientes-activos/carga-masiva` — Carga Masiva (ingredientes activos)

- **Componente:** `IngredientesActivosCargaMasivaComponent` (`app/pages/ingredientes-activos/ingredientes-activos-carga-masiva/ingredientes-activos-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `ingredientesActivos` (no aporta textos).
- **Para qué sirve:** cargar varios ingredientes desde un Excel; el flujo común está en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/ingredientes-activos` (§4.1).
- **Permiso de ruta:** `IngredientesActivos:Leer` + `IngredientesActivos:Crear`.
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Catálogos» / «Ingredientes Activos» (enlace a `/ingredientes-activos`) / «Carga Masiva» (activa); botón volver a `/ingredientes-activos`.

**Configuración que pasa** (`.ts:18-33`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/ingredientesactivos/plantilla` — `IngredientesActivos:Crear` (`IngredientesActivosController.cs:97`); se baja como `plantilla-ingredientes-activos.xlsx` |
| `previewUrl` | `POST {api}/api/v1/ingredientesactivos/previsualizar` — `IngredientesActivos:Crear` (`:106`) |
| `saveUrl` | `POST {api}/api/v1/ingredientesactivos/carga-masiva` — `IngredientesActivos:Crear` (`:124`) |
| `backRoute` | `/ingredientes-activos` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz; las dos obligatorias) | `denominacionDci` → «Denominación DCI» (`fields.denominacionDci`); `codigoAtc` → «Código ATC» (`fields.codigoAtc`). Después, la columna de errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Reglas de la carga en RN-ING-011 a RN-ING-015 de 01b. Dato relevante para la vista previa (verificado en `PreviewImportIngredientesActivosQueryHandler.cs:13-47`): los errores `duplicate` y `duplicate-in-file` se evalúan sobre el campo **`codigoAtc`**, aunque en crear y editar la clave única es la denominación DCI; las claves de datos coinciden con `IngredienteActivoImportRow(DenominacionDci, CodigoAtc)` (`CargaMasivaIngredientesActivosCommand.cs:5`).

---

## 5. Catálogo DIGEMID

Carpeta `app/pages/digemid-catalogo`, scope i18n `digemidCatalogo` (`provideTranslocoScope('digemidCatalogo')` en los tres componentes). Componentes enrutados (`app/pages/digemid-catalogo/digemid-catalogo.routes.ts`, `loadChildren` en `app/app.routes.ts:165`): `DigemidCatalogoSearchComponent` (`''`), `DigemidCatalogoCargaMasivaComponent` (`carga-masiva`) y `DigemidCatalogoDetailComponent` (`:id`). `carga-masiva` está antes de `:id`.

**Qué se puede hacer y qué no desde la pantalla** (comprobado en las plantillas y en el servicio):

| Acción | ¿Hay pantalla? | Evidencia |
|---|---|---|
| Buscar por código y nombre del producto | Sí (§5.1) | `digemid-catalogo-search.component.html:30-51` |
| Ver el detalle de un producto (solo lectura) | Sí (§5.2) | `digemid-catalogo-detail.component.ts:51-57` deshabilita el formulario y nada lo vuelve a habilitar |
| Exportar el resultado a Excel | Sí (§5.1) | `.html:124-127` |
| Descargar plantilla, previsualizar y guardar productos por carga masiva | Sí, con `DIGEMID:Crear` (§5.3) | `.html:17-28`, `digemid-catalogo.routes.ts:11-17` |
| **Crear un producto individual** | **No.** No hay ruta `/digemid-catalogo/nueva` (`digemid-catalogo.routes.ts` solo define `''`, `carga-masiva` y `:id`), ni botón «Nuevo» en la búsqueda (el encabezado de filtros solo tiene «Carga Masiva», `.html:17-28`) | `DigemidCatalogoService` solo tiene `search`, `getById` y `exportar` (`digemid-catalogo.service.ts:17-37`) |
| **Editar un producto** | **No.** El componente de detalle no importa `PermisoDirective`, no tiene botón «Editar» y el formulario queda deshabilitado | `digemid-catalogo-detail.component.ts:11-21`, `.html` completo |
| **Eliminar un producto** | **No.** No hay botón «Eliminar» ni diálogo de confirmación; el detalle no importa `MatDialog` | Mismas fuentes |

La API sí admite estas tres operaciones: `POST /api/v1/DIGEMID/catalogo` (`DIGEMID:Crear`), `PUT /api/v1/DIGEMID/catalogo/{id}` (`DIGEMID:Actualizar`) y `DELETE /api/v1/DIGEMID/catalogo/{id}` (`DIGEMID:Eliminar`) (`DIGEMIDController.cs:71,81,90`), pero **ninguna pantalla del SPA las llama** (el servicio no las define). Quedan en el modelo `CreateDigemidCatalogoRequest` y `UpdateDigemidCatalogoRequest` (`models/digemid-catalogo.model.ts:31-52`) sin ningún uso y en el i18n las claves `detail.title-new`, `detail.title-edit`, `detail.save-success`, `detail.save-error`, `detail.delete-success`, `detail.delete-error`, `detail.delete-confirm-title` y `detail.delete-confirm-message` (ver §9). Por eso este módulo no tiene mapa de errores de guardado ni de eliminación (§6).

Si escribes `/digemid-catalogo/nueva` en la barra de direcciones: el guard exige `DIGEMID:Leer` + `DIGEMID:Crear` (el segundo segmento `nueva` es de creación, `route-permissions.ts:36,68-71`); si pasa, la URL coincide con la ruta `:id` y `DigemidCatalogoDetailComponent` pide `GET /api/v1/digemid/catalogo/nueva`, que no es un identificador válido (la ruta de la API es `catalogo/{id:guid}`, `DIGEMIDController.cs:63`). El componente mostraría «Error al cargar el producto DIGEMID» con el formulario vacío. La respuesta exacta de la API: «No verificado».

### 5.1 `/digemid-catalogo` — Catálogo DIGEMID (búsqueda)

- **Componente:** `DigemidCatalogoSearchComponent` (`app/pages/digemid-catalogo/digemid-catalogo-search/digemid-catalogo-search.component.ts`, `.html`). Scope `digemidCatalogo`.
- **Para qué sirve:** consultar el catálogo de productos DIGEMID de la empresa por código y nombre, abrir un producto, exportar la lista y entrar a la carga masiva.
- **Cómo se llega:** menú lateral, grupo «DIGEMID» → «Catálogo DIGEMID» (`nav.catalogoDigemid`); botón volver y migas del detalle y de la carga masiva; redirección tras guardar una carga masiva.
- **Permiso de ruta:** `DIGEMID:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Catálogo DIGEMID» (`digemidCatalogo.title`); migas «Inicio» (`breadcrumbs.home`) / «DIGEMID» (`breadcrumbs.digemid`, sin enlace) / «Catálogo DIGEMID» (`breadcrumbs.digemid-catalogo`, activa); sin botón volver (`.html:4-7`, `.ts:47-51`). Tarjeta de filtros con título «Búsqueda en Catálogo DIGEMID» (`digemidCatalogo.search.title`) y el subtítulo común.
- **Modos:** uno solo (búsqueda).

**Filtros de búsqueda** (`.ts:45`, `.html:30-51`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Código de Producto (`digemidCatalogo.fields.codProd`) | `codProd` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Nombre del Producto (`digemidCatalogo.fields.nomProd`) | `nomProd` | Texto con lupa; Enter busca | Texto libre | Vacío |

**Tabla de resultados** (`.html:77-120`, columnas `.ts:53`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Código (`digemidCatalogo.table.codProd`) | `codProd` | Texto | Sí: `[routerLink]="[row.id]"` → `/digemid-catalogo/:id` |
| Nombre del Producto (`digemidCatalogo.table.nomProd`) | `nomProd` | Texto | No |
| Forma Farmacéutica (`digemidCatalogo.table.formaFarmaceutica`) | `formaFarmaceutica` | Texto; «—» si es nulo (`.html:101`) | No |
| Estado (`digemidCatalogo.table.estado`) | `estado` | Texto tal como viene (texto libre, no se traduce) | No |

La API devuelve además concentración, registro sanitario y titular en cada fila (`DigemidCatalogoListItem`, `models/digemid-catalogo.model.ts:8-17`) pero la tabla no los muestra. Orden por defecto: lo fija el servidor, por nombre de producto y luego por identificador (`sofia-apis/src/SOFIA.Application/DIGEMID/Queries/GetDigemidCatalogo/GetDigemidCatalogoQueryHandler.cs:28`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:117`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha (único botón del encabezado) | `*appPermiso="'Crear'"` (= `DIGEMID:Crear`, `.html:20`) | Nunca | `[routerLink]="['carga-masiva']"` → `/digemid-catalogo/carga-masiva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/digemid/catalogo` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/digemid/catalogo/exportar` |
| Enlace con el código | Primera columna | Con resultados | — | Navega a `/digemid-catalogo/:id` |

No hay botón «Nuevo».

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar en el catálogo DIGEMID» (`digemidCatalogo.search.search-error`) | `digemid-catalogo-search.component.ts:93` |
| Falla la exportación | Error | El mismo texto, con referencia de seguimiento si la respuesta trae `traceId` (`showServerError`) | `digemid-catalogo-search.component.ts:135` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar el estado | `DigemidCatalogoService.search` (`app/pages/digemid-catalogo/services/digemid-catalogo.service.ts:17-28`) | `GET /api/v1/digemid/catalogo?pageNumber&pageSize[&codProd][&nomProd]` | `DIGEMID:Leer` (`DIGEMIDController.cs:55`) |
| Exportar | `DigemidCatalogoService.exportar` (`:34-36`) | `POST /api/v1/digemid/catalogo/exportar` (respuesta blob) | `DIGEMID:Leer` (`DIGEMIDController.cs:134`) |

**Exportación**
- Cuerpo (`.ts:113-137`): `headers` con 4 textos traducidos, en orden: «Código» (`digemidCatalogo.table.codProd`), «Nombre del Producto» (`digemidCatalogo.table.nomProd`), «Forma Farmacéutica» (`digemidCatalogo.table.formaFarmaceutica`), «Estado» (`digemidCatalogo.table.estado`); más `codProd` y `nomProd` si tienen valor. El Excel lleva esas cuatro columnas, las mismas que la tabla (`DIGEMIDController.cs:153-159`).
- Archivo: «digemid-catalogo.xlsx» (`digemidCatalogo.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §5.3.

**Comportamientos particulares:** restauración de filtros y «Buscar» sin volver a la página 1, ver §0.

**Observaciones:** la fila sin resultados dice «Resultados» y no «Sin resultados» (`digemid-catalogo-search.component.html:117`). El menú del grupo «DIGEMID» solo tiene esta opción: no hay pantalla de cuarentena ni de actas, aunque la API las expone (`DIGEMIDController.cs:21-52`).

### 5.2 `/digemid-catalogo/:id` — Catálogo DIGEMID (detalle de solo lectura)

- **Componente:** `DigemidCatalogoDetailComponent` (`app/pages/digemid-catalogo/digemid-catalogo-detail/digemid-catalogo-detail.component.ts`, `.html`). Scope `digemidCatalogo`.
- **Para qué sirve:** ver todos los datos de un producto del catálogo DIGEMID. No permite cambiarlos.
- **Cómo se llega:** enlace del código en la tabla de `/digemid-catalogo` (§5.1). No hay otro enlace.
- **Permiso de ruta:** `DIGEMID:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Catálogo DIGEMID» (`digemidCatalogo.title`); migas «Inicio» / «DIGEMID» / «Catálogo DIGEMID» (con enlace a `/digemid-catalogo`); botón volver a `/digemid-catalogo` (`.html:4-8`, `.ts:45-49`). La tarjeta no tiene encabezado ni subtítulo «Datos del registro».
- **Modos:** solo **ver**. Con el parámetro `id` el formulario se deshabilita y se cargan los datos (`ngOnInit`, `.ts:51-57`); no hay modo crear ni editar (§5, tabla inicial).

**Campos del formulario** (`.ts:34-43`, `.html:16-72`; todos deshabilitados siempre; sin validadores)

| Etiqueta (clave) | `formControlName` | Control | Columna | Obligatorio | Validaciones | Editable |
|---|---|---|---|---|---|---|
| Código de Producto (`digemidCatalogo.fields.codProd`) | `codProd` | `matInput` | 3/12 | — | Ninguna | Nunca |
| Nombre del Producto (`digemidCatalogo.fields.nomProd`) | `nomProd` | `matInput` | 9/12 | — | Ninguna | Nunca |
| Concentración (`digemidCatalogo.fields.concent`) | `concent` | `matInput` | 4/12 | — | Ninguna | Nunca |
| Forma Farmacéutica (`digemidCatalogo.fields.formaFarmaceutica`) | `formaFarmaceutica` | `matInput` | 4/12 | — | Ninguna | Nunca |
| Fracción (`digemidCatalogo.fields.fraccion`) | `fraccion` | `matInput` | 4/12 | — | Ninguna | Nunca |
| Registro Sanitario (`digemidCatalogo.fields.registroSanitario`) | `registroSanitario` | `matInput` | 4/12 | — | Ninguna | Nunca |
| Titular (`digemidCatalogo.fields.titular`) | `titular` | `matInput` | 5/12 | — | Ninguna | Nunca |
| Estado (`digemidCatalogo.fields.estado`) | `estado` | `matInput` (texto libre, no chip) | 3/12 | — | Ninguna | Nunca |

Los valores nulos (concentración, forma, fracción, registro y titular pueden ser nulos, `DigemidCatalogoDetail`, `models/digemid-catalogo.model.ts:19-29`) se ven como campo vacío. No hay mensajes de error de campo, ayuda ni placeholder.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Botón volver (icono `chevron_left`) | Cabecera | Siempre | — | Navega a `/digemid-catalogo` |

No hay otros botones: ni «Editar», ni «Guardar», ni «Eliminar», ni `*appPermiso`.

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Error al cargar (cualquier error) | Error | «Error al cargar el producto DIGEMID» (`digemidCatalogo.detail.load-error`) | `digemid-catalogo-detail.component.ts:68` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Al abrir | `DigemidCatalogoService.getById` (`digemid-catalogo.service.ts:30-32`) | `GET /api/v1/digemid/catalogo/{id}` | `DIGEMID:Leer` (`DIGEMIDController.cs:63`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares:** mientras carga muestra un spinner en lugar de la tarjeta (`.html:10-14`). Si la ruta no trae `id` el formulario no se deshabilita, pero la ruta `:id` siempre lo trae.

**Observaciones**
- Esta pantalla muestra los textos de `digemidCatalogo.fields.*`, distintos de los de la carga masiva, que usa la raíz (por ejemplo «Código de Producto» aquí y «Cód. Producto» en la vista previa).
- Los cinco campos opcionales y el estado son texto libre; los límites de longitud (RN-DIG-005 de 01b) solo se validan en la carga masiva.

### 5.3 `/digemid-catalogo/carga-masiva` — Carga Masiva (catálogo DIGEMID)

- **Componente:** `DigemidCatalogoCargaMasivaComponent` (`app/pages/digemid-catalogo/digemid-catalogo-carga-masiva/digemid-catalogo-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `digemidCatalogo` (no aporta textos). Es **la única vía del SPA para agregar productos al catálogo**.
- **Para qué sirve:** cargar productos del catálogo DIGEMID desde un Excel; el flujo común está en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/digemid-catalogo` (§5.1).
- **Permiso de ruta:** `DIGEMID:Leer` + `DIGEMID:Crear` (segundo segmento `carga-masiva`).
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «DIGEMID» / «Catálogo DIGEMID» (enlace a `/digemid-catalogo`) / «Carga Masiva» (activa); botón volver a `/digemid-catalogo`.

**Configuración que pasa** (`.ts:18-40`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/digemid/catalogo/plantilla` — `DIGEMID:Crear` (`DIGEMIDController.cs:98`); se baja como `plantilla-digemid-catalogo.xlsx` |
| `previewUrl` | `POST {api}/api/v1/digemid/catalogo/previsualizar` — `DIGEMID:Crear` (`:107`) |
| `saveUrl` | `POST {api}/api/v1/digemid/catalogo/carga-masiva` — `DIGEMID:Crear` (`:125`) |
| `backRoute` | `/digemid-catalogo` |
| Columnas de la vista previa, en este orden (clave del dato → etiqueta de la raíz) | `codProd` → «Cód. Producto» (`fields.codProd`, obligatoria); `nomProd` → «Nombre del Producto» (`fields.nomProd`, obligatoria); `concent` → «Concentración» (`fields.concent`); `formaFarmaceutica` → «Forma Farmacéutica» (`fields.formaFarmaceutica`); `fraccion` → «Fracción» (`fields.fraccion`); `registroSanitario` → «Registro Sanitario» (`fields.registroSanitario`); `titular` → «Titular» (`fields.titular`); `estado` → «Estado» (`fields.estado`, obligatoria). Después, la columna de errores y «Estado» (`carga-masiva.row-status`) |

**Botones, mensajes y errores:** los del componente común. Reglas de la carga en RN-DIG-010 a RN-DIG-013 de 01b; en particular, la vista previa no valida longitudes y el validador del guardado sí (hallazgo 6 de 01b). Las claves de datos coinciden con `DigemidImportRow` (`CargaMasivaDigemidCommand.cs:5-13`).

**Observaciones:** las dos columnas de «Estado» de la vista previa (la del dato del producto, `estado`, y la de validez de la fila) llevan el mismo encabezado «Estado» (`fields.estado` y `carga-masiva.row-status`).

---

## 6. Eliminar: mapa de códigos de error y mensajes de las cinco pantallas

Contexto: el commit `27597cd` del backend añadió la comprobación de dependencias al eliminar ingredientes activos (y productos, sucursales, pacientes, profesionales y proveedores, fuera de esta parte) y el `01b8260` del SPA añadió los mensajes. En esta parte, **solo Ingredientes activos cambió**; Unidades de medida, Laboratorios y Jerarquías ya tenían su tratamiento de 409 antes (el de Unidades de medida y Laboratorios viene del commit `cefa4d5` y el de Jerarquías del `c1a93c3`, según `git log -S"delete-in-use"` sobre las carpetas). Catálogo DIGEMID no tiene botón de eliminar.

**Qué hace el SPA ante la respuesta del `DELETE`** (idéntico en las cuatro pantallas con botón «Eliminar»): no lee el código de error de la respuesta (`error.code`), solo el estado HTTP. Si el estado es 409 muestra el mensaje «en uso» de la pantalla; con cualquier otro error (404, 403, 500, sin conexión…) muestra el mensaje genérico de la pantalla. Fuente: `unidades-medida-detail.component.ts:164-167`, `laboratorios-detail.component.ts:169-172`, `ingredientes-activos-detail.component.ts:165-168`, `jerarquias-detail.component.ts:283-286`. Ante un 409 genérico, es decir, un 409 con cualquier otro código, el usuario vería igualmente el texto «en uso».

| Pantalla | `DELETE` | Códigos que puede devolver la API | Mensaje con 409 (clave) | Mensaje con otro error (clave) |
|---|---|---|---|---|
| Unidades de medida (§1.2) | `/api/v1/unidadesmedida/{id}`, `UnidadesMedida:Eliminar` | 404 `UnidadMedida.NotFound` (`DeleteUnidadMedidaCommandHandler.cs:17`); 409 `UnidadMedida.InUse` (`:29`) si la usan medicamentos como unidad base, jerarquías como mayor o menor, o formulaciones clínicas (`:20-26`); éxito 204 | «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.» (`unidadesMedida.detail.delete-in-use`) | «Error al eliminar la unidad de medida» (`unidadesMedida.detail.delete-error`) |
| Jerarquías de unidades (§2.2) | `/api/v1/jerarquiasuom/{id}`, `JerarquiasUoM:Eliminar` | 404 `JerarquiaUoM.NotFound` (`DeleteJerarquiaUoMCommandHandler.cs:16`); éxito 204. **No devuelve 409**: no valida dependencias | «No se puede eliminar la jerarquía porque está en uso.» (`jerarquias.detail.delete-in-use`) | «Error al eliminar la jerarquía» (`jerarquias.detail.delete-error`) |
| Laboratorios (§3.2) | `/api/v1/laboratorios/{id}`, `Laboratorios:Eliminar` | 404 `Laboratorio.NotFound` (`DeleteLaboratorioCommandHandler.cs:17`); 409 `Laboratorio.InUse` (`:25`) si lo usa algún medicamento no eliminado (`:20-21`); éxito 204 | «No se puede eliminar el laboratorio porque tiene medicamentos asociados.» (`laboratorios.detail.delete-in-use`) | «Error al eliminar el laboratorio» (`laboratorios.detail.delete-error`) |
| Ingredientes activos (§4.2) | `/api/v1/ingredientesactivos/{id}`, `IngredientesActivos:Eliminar` | 404 `IngredienteActivo.NotFound` (`DeleteIngredienteActivoCommandHandler.cs:17`); 409 `IngredienteActivo.InUse` (`:25`) si está en alguna formulación clínica no eliminada (`:20-21`); éxito 204 | «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» (`ingredientesActivos.detail.delete-in-use`) | «Error al eliminar el ingrediente activo» (`ingredientesActivos.detail.delete-error`) |
| Catálogo DIGEMID (§5) | No hay botón ni llamada. La API tiene `DELETE /api/v1/DIGEMID/catalogo/{id}` (`DIGEMID:Eliminar`, `DIGEMIDController.cs:90`), que no valida dependencias (RN-DIG-008) | — | — | — |

Los textos de 404 no tienen mensaje propio en ninguna pantalla.

**Cobertura de los mensajes frente a lo que bloquea el backend**

| Pantalla | Qué bloquea el backend | Qué dice el mensaje | Observación |
|---|---|---|---|
| Unidades de medida | Medicamentos, jerarquías y formulaciones clínicas | Medicamentos o jerarquías | El texto omite las formulaciones clínicas |
| Laboratorios | Medicamentos | Medicamentos asociados | Coincide |
| Ingredientes activos | Formulaciones clínicas | Formulaciones clínicas | Coincide |
| Jerarquías de unidades | Nada | «está en uso» | El 409 no ocurre hoy con este endpoint; el mensaje queda por el tratamiento genérico de cualquier 409 |

Lo que sigue **sin** tratamiento de dependencias en esta parte: eliminar jerarquías (no valida) y, por API, productos del catálogo DIGEMID.

---

## 7. Pendientes y «No verificado»

**«En curso (sin commitear)»:** ninguno. Se ejecutó `git status --short` en ambos repos al empezar (`sofia-apis` `27597cd`, `sofia-web-spa` `01b8260`): el SPA tiene cambios solo en `README.md`, `angular.json` y `src/index.html`, y el backend en `README.md`, `Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandHandler.cs`, `Domain/Entities/Sucursal.cs` y pruebas de Empresas y Sucursales. Ningún archivo de las carpetas de esta parte (`pages/unidades-medida`, `pages/jerarquias`, `pages/laboratorios`, `pages/ingredientes-activos`, `pages/digemid-catalogo`), de los scopes i18n que usan ni de los controladores, validadores y manejadores citados está en esa lista.

**«No verificado»** (no se ejecutó la aplicación ni se llamó a la API):

| # | Tema | Por qué no se verificó |
|---|---|---|
| 1 | Respuesta de la API a `GET /api/v1/digemid/catalogo/nueva` (lo que produce abrir `/digemid-catalogo/nueva`) y lo que ve exactamente el usuario | La ruta de la API es `catalogo/{id:guid}`; no se llamó a la API. Por el código del componente, cualquier error de carga muestra «Error al cargar el producto DIGEMID» |
| 2 | Qué ve el usuario de Jerarquías (búsqueda y detalle) si no tiene `Medicamentos:Leer` o `UnidadesMedida:Leer` | El SPA no comprueba esos permisos y las suscripciones de las listas no tienen manejador de error; el efecto visual (lista vacía, error sin aviso) no se probó |
| 3 | Si elegir una sugerencia con el teclado (flechas y Enter) en las listas de Jerarquías dispara el `(click)` que guarda la elección | Depende del comportamiento interno de Angular Material; no se ejecutó. Si no lo dispara, al guardar la elección se descartaría y el campo mostraría «Este campo es obligatorio» |
| 4 | Que valores formados solo por espacios lleguen a la API y muestren el mensaje genérico en Unidades de medida, Laboratorios (nombre) e Ingredientes activos | Se deduce de `Validators.required` de Angular y de `NotEmpty` de FluentValidation; no se probó |
| 5 | Aspecto visual: iconos (`upload`, `add`, `download`, `chevron_left`), deslizador de Jerarquías, spinner, alineación a la derecha del multiplicador | Solo lectura de plantillas |
| 6 | Si la tabla de resultados de Jerarquías muestra el multiplicador con ceros decimales (por ejemplo `10` o `10.0000`) | Se imprime tal como lo serializa la API; no se llamó |

**Pendientes de producto detectados** (no son errores de documentación):
- Catálogo DIGEMID: la API permite crear, editar y eliminar un producto, pero el SPA no tiene botones ni rutas para ello; solo consulta y carga masiva.
- Jerarquías: ninguna vista de la cadena de conversión; el código de error `JerarquiaUoM.Multiplicador.Inconsistente` (409) no tiene mensaje propio y se ve como «Error al guardar la jerarquía».
- Unidades de medida: el mensaje «en uso» omite las formulaciones clínicas.
- No hay pantallas de cuarentena ni de actas DIGEMID (ver `REGLAS-AGENTES.md` §4).
- Detalles de Unidades de medida, Laboratorios e Ingredientes activos: método `goToNew()` sin uso.

**Documentos hermanos que el código actual contradice** (para la pasada de reconciliación):
- 01b: RN-ING-009, que dice que el handler de eliminar ingredientes no valida dependencias; con `27597cd` sí lo hace (`DeleteIngredienteActivoCommandHandler.cs:20-27`). Los hallazgos 7 y 10 de 01b (eliminaciones sin dependencias y pantallas que tratan cualquier 409 como un caso concreto) se ven afectados en lo que toca a ingredientes activos.
- 19b §2.2 y su línea sobre Sucursales («el backend no comprueba empleados, series ni cuentas asociadas») quedan fuera de esta parte; `REGLAS-AGENTES.md` §3 indica que el commit `27597cd` añadió esa validación. No se re-verificó aquí.

---

## 8. Tabla de cobertura

**Rutas de `docs/_inventario/rutas-spa.csv` de este alcance (19 de 19)**

| Ruta | Componente | Ficha |
|---|---|---|
| `/unidades-medida` | `UnidadesMedidaSearchComponent` | §1.1 |
| `/unidades-medida/nueva` | `UnidadesMedidaDetailComponent` (modo crear) | §1.2 |
| `/unidades-medida/carga-masiva` | `UnidadesMedidaCargaMasivaComponent` | §1.3 |
| `/unidades-medida/:id` | `UnidadesMedidaDetailComponent` (modos ver y editar) | §1.2 |
| `/jerarquias` | `JerarquiasSearchComponent` | §2.1 |
| `/jerarquias/nueva` | `JerarquiasDetailComponent` (modo crear) | §2.2 |
| `/jerarquias/carga-masiva` | `JerarquiasCargaMasivaComponent` | §2.3 |
| `/jerarquias/:id` | `JerarquiasDetailComponent` (modos ver y editar) | §2.2 |
| `/laboratorios` | `LaboratoriosSearchComponent` | §3.1 |
| `/laboratorios/nueva` | `LaboratoriosDetailComponent` (modo crear) | §3.2 |
| `/laboratorios/carga-masiva` | `LaboratoriosCargaMasivaComponent` | §3.3 |
| `/laboratorios/:id` | `LaboratoriosDetailComponent` (modos ver y editar) | §3.2 |
| `/ingredientes-activos` | `IngredientesActivosSearchComponent` | §4.1 |
| `/ingredientes-activos/nueva` | `IngredientesActivosDetailComponent` (modo crear) | §4.2 |
| `/ingredientes-activos/carga-masiva` | `IngredientesActivosCargaMasivaComponent` | §4.3 |
| `/ingredientes-activos/:id` | `IngredientesActivosDetailComponent` (modos ver y editar) | §4.2 |
| `/digemid-catalogo` | `DigemidCatalogoSearchComponent` | §5.1 |
| `/digemid-catalogo/carga-masiva` | `DigemidCatalogoCargaMasivaComponent` | §5.3 |
| `/digemid-catalogo/:id` | `DigemidCatalogoDetailComponent` (solo ver) | §5.2 |

`/digemid-catalogo/nueva` no existe como ruta (comprobado en `digemid-catalogo.routes.ts` y en `rutas-spa.csv`): ver §5.

**Componentes de `docs/_inventario/componentes-pages.txt` de este alcance (15 de 15)**

| Componente (carpeta bajo `app/pages`) | Ficha o motivo |
|---|---|
| `unidades-medida/unidades-medida-search`, `-detail`, `-carga-masiva` | §1.1, §1.2, §1.3 |
| `jerarquias/jerarquias-search`, `-detail`, `-carga-masiva` | §2.1, §2.2, §2.3 |
| `laboratorios/laboratorios-search`, `-detail`, `-carga-masiva` | §3.1, §3.2, §3.3 |
| `ingredientes-activos/ingredientes-activos-search`, `-detail`, `-carga-masiva` | §4.1, §4.2, §4.3 |
| `digemid-catalogo/digemid-catalogo-search`, `-detail`, `-carga-masiva` | §5.1, §5.2, §5.3 |

**Diálogos de `docs/_inventario/dialogos.txt`:** ninguna carpeta de este alcance contiene un diálogo propio (búsqueda en `dialogos.txt` sin resultados). El único diálogo que abren estas pantallas es `ConfirmDialogComponent` (compartido, [19a §6.1](19a-acceso-layout-y-compartidos.md)) al eliminar, en los detalles de §1.2, §2.2, §3.2 y §4.2. Catálogo DIGEMID no abre ninguno.

**Usos de `*appPermiso` de `docs/_inventario/app-permiso-usos.csv` de este alcance (17 de 17)**

| Archivo:línea (bajo `app/pages/…`) | Expresión | Permiso resultante | Botón y ficha |
|---|---|---|---|
| `unidades-medida/unidades-medida-search/…search.component.html:20` | `'Crear'` | `UnidadesMedida:Crear` | «Carga Masiva», §1.1 |
| `unidades-medida/unidades-medida-search/…search.component.html:28` | `'Crear'` | `UnidadesMedida:Crear` | «Nuevo», §1.1 |
| `unidades-medida/unidades-medida-detail/…detail.component.html:28` | `'Actualizar'` | `UnidadesMedida:Actualizar` | «Editar», §1.2 |
| `unidades-medida/unidades-medida-detail/…detail.component.html:75` | `'Eliminar'` | `UnidadesMedida:Eliminar` | «Eliminar», §1.2 |
| `jerarquias/jerarquias-search/…search.component.html:19` | `'Crear'` | `JerarquiasUoM:Crear` | «Carga Masiva», §2.1 |
| `jerarquias/jerarquias-search/…search.component.html:27` | `'Crear'` | `JerarquiasUoM:Crear` | «Nuevo», §2.1 |
| `jerarquias/jerarquias-detail/…detail.component.html:27` | `'Actualizar'` | `JerarquiasUoM:Actualizar` | «Editar», §2.2 |
| `jerarquias/jerarquias-detail/…detail.component.html:169` | `'Eliminar'` | `JerarquiasUoM:Eliminar` | «Eliminar», §2.2 |
| `laboratorios/laboratorios-search/…search.component.html:19` | `'Crear'` | `Laboratorios:Crear` | «Carga Masiva», §3.1 |
| `laboratorios/laboratorios-search/…search.component.html:27` | `'Crear'` | `Laboratorios:Crear` | «Nuevo», §3.1 |
| `laboratorios/laboratorios-detail/…detail.component.html:27` | `'Actualizar'` | `Laboratorios:Actualizar` | «Editar», §3.2 |
| `laboratorios/laboratorios-detail/…detail.component.html:79` | `'Eliminar'` | `Laboratorios:Eliminar` | «Eliminar», §3.2 |
| `ingredientes-activos/ingredientes-activos-search/…search.component.html:20` | `'Crear'` | `IngredientesActivos:Crear` | «Carga Masiva», §4.1 |
| `ingredientes-activos/ingredientes-activos-search/…search.component.html:28` | `'Crear'` | `IngredientesActivos:Crear` | «Nuevo», §4.1 |
| `ingredientes-activos/ingredientes-activos-detail/…detail.component.html:28` | `'Actualizar'` | `IngredientesActivos:Actualizar` | «Editar», §4.2 |
| `ingredientes-activos/ingredientes-activos-detail/…detail.component.html:78` | `'Eliminar'` | `IngredientesActivos:Eliminar` | «Eliminar», §4.2 |
| `digemid-catalogo/digemid-catalogo-search/…search.component.html:20` | `'Crear'` | `DIGEMID:Crear` | «Carga Masiva», §5.1 |

Los componentes de estas carpetas no comprueban permisos en el `.ts` (no hay llamadas a `AuthService.hasPermission` ni a `grantsPermission`); el detalle de DIGEMID no usa `*appPermiso`.

**Endpoints de `docs/_inventario/endpoints.csv` de este alcance (46) y quién los llama**

| Controlador | Endpoints en el CSV | Llamados por pantallas de esta parte | No llamados desde esta parte |
|---|---|---|---|
| `UnidadesMedida` | 9 | 9 (§1.1 a §1.3); además `GET` de la lista la llama Jerarquías (§2.1, §2.2) | — |
| `JerarquiasUoM` | 10 | 9 (§2.1 a §2.3) | `GET /unidades-vendibles` (lo llama Presentaciones de venta, otra parte) |
| `Laboratorios` | 9 | 9 (§3.1 a §3.3) | — |
| `IngredientesActivos` | 9 | 9 (§4.1 a §4.3) | — |
| `DIGEMID` (rutas `catalogo`) | 9 | 6: `GET` lista, `GET` por id, `POST exportar`, plantilla, previsualizar y carga masiva (§5.1 a §5.3) | `POST`, `PUT` y `DELETE /catalogo` (sin pantalla, §5) |

Fuera del conteo: los endpoints `cuarentena`, `acta-destruccion` y `actas` del controlador DIGEMID (sin pantalla; ver `REGLAS-AGENTES.md` §4).

---

## 9. Claves i18n sin uso

Método: para cada clave del `es.json` del scope se buscó la forma completa (`<scope>.<clave>`) entre comillas en todo `src/` (sin specs) y la forma relativa `t('<clave>')` en los archivos de la carpeta del feature. Las claves de etiquetas de carga masiva (`fields.*` del scope) **sí** se usan en las plantillas de búsqueda y detalle; las columnas de la carga masiva usan la raíz, no el scope.

| Scope (claves en `es.json`) | Claves sin ninguna referencia |
|---|---|
| `unidadesMedida` (63) | 46: `searchPlaceholder`, `add`, `cols.codigo`, `cols.descripcion`, `cols.acciones`, `actions.edit`, `actions.view`, `actions.delete`, `form.titleNew`, `form.titleEdit`, `form.titleView`, `form.codigo`, `form.descripcion`, `form.save`, `form.cancel`, `form.back`, `form.required`, `form.maxLength10`, `form.maxLength50`, `deleteDialog.title`, `deleteDialog.message`, `deleteDialog.confirm`, `deleteDialog.cancel`, `notifications.created`, `notifications.updated`, `notifications.deleted`, `help.codigo`, `help.descripcion`, `import.title`, `import.downloadTemplate`, `import.selectFile`, `import.dragDrop`, `import.fileSelected`, `import.errorsFound`, `import.saveValid`, `import.noValid`, `import.saving`, `import.previewTitle`, `import.success`, `import.errors`, `import.status`, `import.valid`, `import.invalid`, `import.button`, `detail.title-new`, `detail.title-edit`. (Los botones usan `actions.*` de la raíz, no los del scope.) |
| `jerarquias` (27) | Ninguna |
| `laboratorios` (64) | 46: `searchPlaceholder`, `add`, `cols.nombreCompania`, `cols.codigoIdentificador`, `cols.acciones`, `actions.edit`, `actions.view`, `actions.delete`, `form.titleNew`, `form.titleEdit`, `form.titleView`, `form.nombreCompania`, `form.codigoIdentificador`, `form.save`, `form.cancel`, `form.back`, `form.required`, `form.maxLength150`, `form.maxLength50`, `deleteDialog.title`, `deleteDialog.message`, `deleteDialog.confirm`, `deleteDialog.cancel`, `notifications.created`, `notifications.updated`, `notifications.deleted`, `help.nombreCompania`, `help.codigoIdentificador`, `import.title`, `import.downloadTemplate`, `import.selectFile`, `import.dragDrop`, `import.fileSelected`, `import.errorsFound`, `import.saveValid`, `import.noValid`, `import.saving`, `import.previewTitle`, `import.success`, `import.errors`, `import.status`, `import.valid`, `import.invalid`, `import.button`, `detail.title-new`, `detail.title-edit` |
| `ingredientesActivos` (19) | 2: `detail.title-new`, `detail.title-edit` |
| `digemidCatalogo` (25) | 8: `detail.title-new`, `detail.title-edit`, `detail.save-success`, `detail.save-error`, `detail.delete-success`, `detail.delete-error`, `detail.delete-confirm-title`, `detail.delete-confirm-message` (restos de un detalle editable que no existe) |
| `uom` (51) | **Las 51**: ningún archivo de `src/` carga el scope `uom` (`scope: 'uom'` ni `provideTranslocoScope('uom')`) ni usa claves con prefijo `uom.`. Es un scope heredado («Mantenimiento de Jerarquías UoM»); la pantalla actual de Jerarquías usa `jerarquias` |

Claves usadas pero **inexistentes**: ninguna. Se resolvió contra los `es.json` cada clave de `t('…')`, `transloco.translate('<scope>.…')` y del pipe `transloco` de las 15 plantillas y componentes del alcance, y contra la raíz las claves de `fields.*`, `breadcrumbs.*`, `actions.*`, `generic-labels.*`, `general.errors.required` y `carga-masiva.*` que usan.

---

## 10. Fuentes

**SPA (`sofia-web-spa/`)**
- Enrutado y permisos: `src/app/app.routes.ts` (líneas 37, 76, 81, 86, 91, 165), `src/app/core/auth/route-permissions.ts`, `src/app/core/auth/permission.guard.ts`, `src/app/shared/directives/permiso.directive.ts`, `src/app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts`.
- Compartidos: `src/app/core/services/shared/search-state.service.ts`, `src/app/shared/utils/save-error-key.util.ts`, `src/app/shared/components/carga-masiva-page/carga-masiva-page.component.{ts,html}`, `src/app/shared/models/carga-masiva.model.ts`, `src/app/shared/components/range-slider-field/range-slider-field.component.ts`, `src/app/shared/testing/delete-in-use-testing.ts`.
- Unidades de medida: `src/app/pages/unidades-medida/unidades-medida.routes.ts`, `unidades-medida-search/*`, `unidades-medida-detail/*`, `unidades-medida-carga-masiva/*`, `models/unidad-medida.model.ts`, `services/unidad-medida.service.ts`.
- Jerarquías: `src/app/pages/jerarquias/jerarquias.routes.ts`, `jerarquias-search/*`, `jerarquias-detail/*`, `jerarquias-carga-masiva/*`, `models/jerarquia-uom.model.ts`, `services/jerarquia-uom.service.ts`; `src/app/pages/medicamentos/services/medicamento.service.ts` (lista de productos).
- Laboratorios: `src/app/pages/laboratorios/laboratorios.routes.ts`, `laboratorios-search/*` (con su spec), `laboratorios-detail/*` (con su spec), `laboratorios-carga-masiva/*`, `models/laboratorio.model.ts`, `services/laboratorio.service.ts`.
- Ingredientes activos: `src/app/pages/ingredientes-activos/ingredientes-activos.routes.ts`, `ingredientes-activos-search/*`, `ingredientes-activos-detail/*` (con `ingredientes-activos-detail.delete.spec.ts`), `ingredientes-activos-carga-masiva/*`, `models/ingrediente-activo.model.ts`, `services/ingrediente-activo.service.ts`.
- Catálogo DIGEMID: `src/app/pages/digemid-catalogo/digemid-catalogo.routes.ts`, `digemid-catalogo-search/*`, `digemid-catalogo-detail/*`, `digemid-catalogo-carga-masiva/*`, `models/digemid-catalogo.model.ts`, `services/digemid-catalogo.service.ts`.
- Textos: `public/i18n/es.json` (raíz: `nav`, `breadcrumbs`, `actions`, `general`, `generic-labels`, `errors`, `fields`, `carga-masiva`) y `public/i18n/{unidadesMedida,jerarquias,laboratorios,ingredientesActivos,digemidCatalogo,uom}/es.json`.

**Backend (`sofia-apis/src/`, solo lectura puntual)**
- Controladores (permisos y líneas): `SOFIA.API/Controllers/UnidadesMedidaController.cs`, `JerarquiasUoMController.cs`, `LaboratoriosController.cs`, `IngredientesActivosController.cs`, `DIGEMIDController.cs`.
- Eliminar: `SOFIA.Application/UnidadesMedida/Commands/DeleteUnidadMedida/*Handler.cs`, `JerarquiasUoM/Commands/DeleteJerarquiaUoM/*Handler.cs`, `Laboratorios/Commands/DeleteLaboratorio/*Handler.cs`, `IngredientesActivos/Commands/DeleteIngredienteActivo/*Handler.cs`.
- Guardar y validar: `Create*/Update*` (manejadores y validadores) de los cuatro módulos con alta; `Laboratorios/Common/LaboratorioDuplicateChecker.cs`; `JerarquiasUoM/Common/JerarquiaConversionResolver.cs`; `SOFIA.Domain/Entities/JerarquiaUoM.cs` y `Laboratorio.cs`; `Common/Behaviors/ValidationBehavior.cs`.
- Listados: `Queries/Get*/Get*QueryHandler.cs` de los cinco módulos (orden y filtros); `PreviewImportIngredientesActivosQueryHandler.cs` y `CargaMasivaIngredientesActivosCommandHandler.cs`; registros de importación `*ImportRow`.

**Git (solo lectura):** `git show --stat` de `27597cd` (sofia-apis) y `01b8260` (sofia-web-spa); `git log` de las carpetas del alcance; `git status --short` de ambos repos.

**Inventarios y documentos hermanos**
- `docs/_inventario/rutas-spa.csv`, `componentes-pages.txt`, `dialogos.txt`, `app-permiso-usos.csv`, `endpoints.csv`, `i18n-scopes.csv`, `pruebas-spa.csv`.
- [19a — acceso, layout y compartidos](19a-acceso-layout-y-compartidos.md) (§4.3, §6.1, §6.4, §6.5, §6.6, §6.9, §6.11, §6.12, §7.1, §8).
- [19b — administración](19b-administracion.md) (§0 y §2).
- [01b — catálogos](../01-reglas-de-negocio/01b-catalogos.md) (RN-UDM, RN-JER, RN-LAB, RN-ING, RN-DIG).
- [05b — matriz de permisos](../05-seguridad/05b-matriz-de-permisos.md).

