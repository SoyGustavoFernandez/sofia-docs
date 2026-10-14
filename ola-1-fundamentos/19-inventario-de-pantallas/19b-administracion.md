# Inventario de pantallas del SPA — Parte B: administración

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.1 |
| **Fecha** | 2026-10-14 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código del árbol de trabajo del SPA (componentes, plantillas, servicios, modelos e i18n). Del backend solo se abrieron los controladores y validadores necesarios para anotar permisos y comparar validaciones. No se usó el navegador ni la API local. Ninguna carpeta de esta parte tiene cambios sin commitear. La versión 1.1 reconcilia el documento (escrito sobre `sofia-apis` `00991bc` y `sofia-web-spa` `31af681`) con el código actual. |

## Historial de cambios

### Versión 1.1 (2026-10-14)

| # | Apartado o ID | Decía | Dice ahora | Commit |
|---|---|---|---|---|
| 1 | §2.2 Mensajes al usuario, fila «Error al eliminar» | Un solo caso: cualquier error mostraba «Error al eliminar la sucursal» (`sucursales.detail.delete-error`) | Dos casos: respuesta HTTP 409 (`Sucursal.InUse`) muestra «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.» (`sucursales.detail.delete-in-use`); cualquier otro error sigue mostrando `delete-error` | `27597cd` (apis), `01b8260` (spa) |
| 2 | §2.2 Comportamientos particulares | «el SPA no avisa de dependencias y el backend no comprueba empleados, series ni cuentas asociadas (RN-SUC-009 en 01a). Cualquier fallo se ve como «Error al eliminar la sucursal»» | El backend comprueba 10 tipos de dependencia y responde 409 `Sucursal.InUse`; el SPA lo explica con `delete-in-use` (la regla RN-SUC-009 de 01a quedó desactualizada: ver el informe) | `27597cd`, `01b8260` |
| 3 | §2.2 Llamadas a la API, fila Eliminar | Sin código de respuesta | Se anota que puede responder 409 `Sucursal.InUse` | `27597cd` |
| 4 | §2.2 Observaciones | Sin mención de la licencia de la sede principal | Se agrega que la sede principal creada al registrar la empresa nace con el número de licencia vacío y el formulario lo exige al editarla; la dirección de esa sede viene del registro | `41f0a05`, `9dd1705` (apis) |
| 5 | §2.2 citas `sucursales-detail.component.ts:línea` | Líneas de la versión `31af681` | Todas las citas desplazadas por la nueva importación (+1) y por el nuevo manejo del error al eliminar | `01b8260` |
| 6 | §1 resumen y §1.1 Exportación: citas de `empresa.model.ts` | `:268-271` y `:288-296` (líneas que no existían en ese archivo ni en `31af681`, que tenía 73 líneas) | `:29-32` (`UpdateEmpresaRequest`) y `:50-58` (`EmpresaExportRequest`); corrección de una cita errónea previa, no de un cambio de código | `2f82346` (desplazó +1 las líneas posteriores a la 40) |
| 7 | §7 y §8 | «`empresa/empresa-register`: lo documenta la parte A» | Sin cambio de alcance; se anota que el registro ahora exige la dirección de la sede principal y que la ficha está en la parte A | `2f82346` |
| 8 | §9 Claves i18n sin uso | `empresa` (95 claves), `sucursales` (20 claves) | `empresa` (100 claves: se suman `register.direccionSede`, `register.direccionSedePlaceholder`, `register.direccionSedeHint`, `form.maxLength255`, `notifications.direccionSedeInvalida`, todas usadas por `empresa-register`); `sucursales` (21 claves: se suma `detail.delete-in-use`, usada). Las listas de claves sin uso no cambian | `2f82346`, `01b8260` |
| 9 | §10 Fuentes | Backend sin `DeleteSucursal` ni `Sucursal.cs` | Se agregan `DeleteSucursalCommandHandler.cs` y `Sucursal.cs` | `27597cd`, `41f0a05` |

Este documento es la parte B de seis del inventario de pantallas. Cubre los seis módulos de administración: Empresa, Sucursales, Empleados, Roles y permisos, Cuentas de usuario y Series fiscales (19 rutas). Lo compartido (guards, interceptor, `page-header`, `confirm-dialog`, `autocomplete-input`, carga masiva común, textos de la raíz) lo documenta la [parte A](19a-acceso-layout-y-compartidos.md); aquí se remite a ella y se repite el texto exacto de cada etiqueta, botón y mensaje que aparece en estas pantallas. Las reglas de negocio de cada módulo están en [01a](../01-reglas-de-negocio/01a-administracion-y-seguridad.md) y los permisos por endpoint en [05b](../05-seguridad/05b-matriz-de-permisos.md).

Convenciones: las rutas de archivo son relativas a `sofia-web-spa/` salvo que empiecen por `sofia-apis/`. Entre paréntesis va la clave i18n; las claves de scope se escriben con su prefijo real (por ejemplo `empresa.fields.nombre` está en `public/i18n/empresa/es.json`, clave `fields.nombre`). `app/` abrevia `src/app/`. Las plantillas de los módulos de este documento abren con `*transloco="let t; scope: '<scope>'; read: '<scope>'"`, así que `t('x')` equivale a `<scope>.x`.

## Contenido

0. Comportamientos comunes a las pantallas de este documento
1. Empresa: `/empresas`, `/empresas/:id`
2. Sucursales: `/sucursales`, `/sucursales/nueva`, `/sucursales/carga-masiva`, `/sucursales/:id`
3. Empleados: `/empleados`, `/empleados/nueva`, `/empleados/:id`
4. Roles y permisos: `/roles`, `/roles/nueva`, `/roles/carga-masiva`, `/roles/:id`
5. Cuentas de usuario: `/cuentas`, `/cuentas/nueva`, `/cuentas/:id`
6. Series fiscales: `/series-fiscales`, `/series-fiscales/nueva`, `/series-fiscales/:id`
7. Pendientes y «No verificado»
8. Tabla de cobertura
9. Claves i18n sin uso
10. Fuentes

---

## 0. Comportamientos comunes a las pantallas de este documento

- **Cómo se decide el permiso de ruta.** Todas cuelgan de `canActivateChild: [passwordChangeGuard, permissionGuard]` (`app/app.routes.ts:37`) y de `loadChildren` (`app.routes.ts:50-75`). El `permissionGuard` toma el primer segmento de la URL y busca su módulo en `ROUTE_PERMISSIONS` (`app/core/auth/route-permissions.ts:2-26`): `empresas` → `Empresas`, `sucursales` → `Sucursales`, `empleados` → `Empleados`, `roles` y `cuentas` → `Seguridad`, `series-fiscales` → `SeriesFiscales`. Exige siempre `Leer` y, si el segundo segmento es `nueva`, `nuevo` o `carga-masiva`, además `Crear` (`route-permissions.ts:36,68-72`); `/x/:id` solo exige `Leer`. Sin permiso: snackbar «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard` (`app/core/auth/permission.guard.ts:21-24,28-29`). Detalle en [19a §5.3](19a-acceso-layout-y-compartidos.md).
- **Menú.** Las cinco primeras opciones (`Empresa`, `Sucursales`, `Empleados`, `Roles & Permisos`, `Cuentas de Usuario`) están en el grupo «Administración» (`app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:16-47`), con título de sección «Administración» (`breadcrumbs.administracion`). `Series Fiscales` está en el grupo «SUNAT» (`sidebar-data.ts:168-179`), con título de sección «SUNAT» (`breadcrumbs.sunat`). Una opción solo se muestra si el usuario tiene `Leer` del módulo de su ruta (`sidebar.component.ts:29-35`, `visible-nav-items.ts`); un grupo sin opciones visibles se oculta.
- **Directiva `*appPermiso`.** Con un solo valor (`'Crear'`, `'Actualizar'`, `'Eliminar'`) toma el módulo de la URL actual (`ROUTE_PERMISSIONS`) y exige esa acción; el rol `Admin` pasa siempre (`app/shared/directives/permiso.directive.ts:20-25`, `route-permissions.ts:47-55`). Ver [19a §7.1](19a-acceso-layout-y-compartidos.md).
- **Aviso al usuario.** Todos los avisos son snackbar con el botón «Cerrar» (`actions.close`): error 5 s, éxito 3 s (`app/core/services/shared/error-notifier.service.ts:26-32`). No hay interceptor global de errores: cada pantalla decide el mensaje. Un `401` lo maneja el `authInterceptor` ([19a §5.4](19a-acceso-layout-y-compartidos.md)).
- **Estado de la búsqueda.** Las pantallas de búsqueda guardan filtros, página y tamaño de página en un `Map` en memoria (`SearchStateService`, `app/core/services/shared/search-state.service.ts:10-28`) con la URL (sin query) como clave. Al volver a la pantalla desde el detalle se restauran y se repite la búsqueda. No usa `localStorage` ni `sessionStorage`; se pierde al recargar el navegador. «Limpiar» borra ese estado.
- **Tamaños de página** de todas las tablas de resultados: 10, 25 y 50 (`[pageSizeOptions]="[10, 25, 50]"`, con botones primera/última), pagina por defecto de 10. Textos del paginador en [19a §6.9](19a-acceso-layout-y-compartidos.md).
- **Texto «Resultados (N)»** sobre la tabla: «Resultados» (`generic-labels.results`) seguido del total entre paréntesis. Subtítulo de las tarjetas de búsqueda: «Ingresa los criterios de búsqueda» (`generic-labels.search-subtitle`). Subtítulo de las fichas en modo ver: «Datos del registro» (`generic-labels.detail-subtitle`).
- **Modo ver / editar.** Las pantallas de detalle abren en modo ver (formulario deshabilitado) y el botón «Editar» (`actions.edit`) lo habilita; «Cancelar» restaura la copia leída (`snapshot`) y vuelve a deshabilitar. Los botones «Guardar» y «Eliminar» solo aparecen en modo editar.
- **Mensaje de campo obligatorio:** «Este campo es obligatorio» (`general.errors.required`, raíz). Las plantillas de estos módulos muestran solo ese mensaje bajo cada `mat-error`, sea cual sea el validador que falle; la longitud máxima se impide con el atributo HTML `maxlength` (el campo no deja escribir más).

---

## 1. Empresa

Carpeta `app/pages/empresa`, scope i18n `empresa` (`provideTranslocoScope('empresa')`). Componentes enrutados: `EmpresaSearchComponent` y `EmpresaDetailComponent`.

Resumen de lo que el usuario puede hacer en una empresa (comprobado en las fichas siguientes):
- **Ver:** nombre, RUC, estado de suscripción, número de sucursales, inicio del periodo de prueba y vencimiento. El indicador «Vigente» solo aparece en la tabla de búsqueda (en el detalle se calcula pero no se pinta).
- **Editar:** solo **Nombre de la Empresa** y **RUC** (`UpdateEmpresaRequest` solo lleva esos dos campos, `app/pages/empresa/models/empresa.model.ts:29-32`). Estado, fechas y sucursales son de solo lectura y no hay pantalla para cambiarlos.
- **No existe en pantalla** ninguna opción de configuración de la empresa, en particular la exigencia de registro de receta: no hay ninguna referencia a `exigirRegistroReceta` en `app/pages/empresa` (búsqueda en `src/`: solo aparece en `app/pages/pos/`, que la consume). Coincide con RN-EMP-016 (se cambia solo en base de datos).
- **No existe** crear ni eliminar empresas desde esta pantalla: el alta es el autorregistro público (parte A, `/auth/register`) y no hay ruta `/empresas/nueva` (`app/pages/empresa/empresa.routes.ts`).
- **Alcance:** el SPA no filtra por empresa; es la API la que limita cada consulta a la empresa del usuario (RN-EMP-017 en 01a). Por eso, en la práctica, la lista muestra la empresa propia. «No verificado» desde el SPA; ver §7.

### 1.1 `/empresas` — Empresas (búsqueda)

- **Componente:** `EmpresaSearchComponent` (`app/pages/empresa/empresa-search/empresa-search.component.ts`, `empresa-search.component.html`). Scope `empresa`.
- **Para qué sirve:** buscar empresas por nombre, estado de suscripción y rango de fecha de vencimiento, ver el resultado en una tabla y exportarlo a Excel.
- **Cómo se llega:** menú lateral, grupo «Administración» → «Empresa» (`nav.empresa`, `sidebar-data.ts:21-25`); migas de `/empresas/:id` («Empresa») y botón volver del detalle (`backRoute="/empresas"`, `empresa-detail.component.html:7`).
- **Permiso de ruta:** `Empresas:Leer` (`route-permissions.ts:3`). Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Empresas» (`empresa.title`); migas «Inicio» (`breadcrumbs.home`, enlaza a `/dashboard`) / «Administración» (`breadcrumbs.administracion`) / «Empresa» (`breadcrumbs.empresa`, activa). Sin botón volver (`empresa-search.component.html:3-6`, `.ts:59-63`). Tarjeta de filtros con título «Búsqueda de Empresas» (`empresa.search.title`) y el subtítulo común.
- **Modos:** uno solo (búsqueda). Los resultados no se cargan al entrar: solo se muestran tras pulsar «Buscar» o si hay estado guardado de una visita anterior (`ngOnInit`, `.ts:71-79`).

**Filtros de búsqueda** (`empresa-search.component.ts:44-49`, plantilla `.html:16-63`)

| Etiqueta (clave) | Control (`formControlName`) | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Nombre de la Empresa (`empresa.fields.nombre`) | `nombre` | Texto con icono de lupa (`matSuffix`); Enter ejecuta la búsqueda | Texto libre | Vacío |
| Estado (`empresa.fields.estado`) | `estado` | Select | «Todos» (`empresa.fields.estadoTodos`, valor nulo), «Trial Activo» (`empresa.estado.TrialActivo`, 0), «Activo» (`empresa.estado.Activo`, 1), «Suspendido» (`empresa.estado.Suspendido`, 2), «Cancelado» (`empresa.estado.Cancelado`, 3) (`.ts:51-57`) | Nulo (se ve en blanco; «Todos» es una opción elegible) |
| Vencimiento desde (`empresa.fields.fechaVencimientoDesde`) | `fechaVencimientoDesde` | Selector de fecha (`mat-datepicker`), campo `readonly` | Se envía como ISO (`Date.toISOString()`) | Nulo |
| Vencimiento hasta (`empresa.fields.fechaVencimientoHasta`) | `fechaVencimientoHasta` | Selector de fecha, `readonly` | ISO; la fecha elegida se envía a medianoche local, sin ajustar al fin del día (`.ts:88-89`) | Nulo |

El campo del nombre y el de los selectores no tienen validaciones. «Limpiar» (`actions.clear`) vacía el formulario, oculta la tabla, pone página 1 y borra el estado guardado (`clear()`, `.ts:117-124`).

**Tabla de resultados** (`.html:83-152`; orden por defecto del servidor; la API ordena por nombre, ver 01a RN-EMP-018)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Empresa (`empresa.table.nombre`) | `nombre` | Texto | Sí: `routerLink [row.id]` → `/empresas/:id` |
| RUC (`empresa.table.ruc`) | `ruc` | Texto; «—» si es nulo | No |
| Estado (`empresa.table.estado`) | `estado` | Texto traducido: «Trial Activo», «Activo», «Suspendido» o «Cancelado» (`empresa.estado.<n>`) | No |
| Vigente (`empresa.table.estaVigente`) | `estaVigente` | Icono `check_circle` (vigente) o `cancel` (no vigente), sin texto | No |

Encabezado de la tabla: «Resultados (N)». Paginación 10/25/50. **Sin resultados** la fila vacía dice «Resultados» (`generic-labels.results`, `.html:132`), no «Sin resultados» (ver Observaciones). Bajo la tabla, a la izquierda, el botón «Exportar» con icono `download` (`actions.export`) y a la derecha el paginador.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Limpiar (`actions.clear`) | Pie de la tarjeta de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con spinner mientras carga | Pie de la tarjeta de filtros | Siempre | `loading()` | `search(1)` → `GET /api/v1/empresas` (`.ts:81-110`); vuelve a la página 1 |
| Exportar (`actions.export`) | Bajo la tabla | Solo después de buscar (dentro de `@if (showResults())`) | Nunca | `exportar()` → `POST /api/v1/empresas/exportar`, descarga `empresas.xlsx` (`.ts:130-161`) |
| Enlace con el nombre de la empresa | Primera columna | Con resultados | — | Navega a `/empresas/:id` |

No hay botón «Nuevo» ni «Carga Masiva» ni `*appPermiso` en esta pantalla.

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar empresas» (`empresa.search.search-error`) | `empresa-search.component.ts:107` |
| Falla la exportación (cualquier error) | Error | «Error al buscar empresas» (`empresa.search.search-error`; se reutiliza la misma clave) | `empresa-search.component.ts:159` |

No hay mapa de códigos del backend: cualquier error muestra el mismo texto.

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar o cambiar de página | `EmpresaService.search` (`app/pages/empresa/services/empresa.service.ts:16-26`) | `GET /api/v1/empresas?pageNumber&pageSize[&nombre][&estado][&fechaVencimientoDesde][&fechaVencimientoHasta]` (solo envía los filtros con valor; `estado` 0 sí se envía) | `Empresas:Leer` (`EmpresasController.cs:35`) |
| Exportar | `EmpresaService.exportar` (`:44-46`) | `POST /api/v1/empresas/exportar` (respuesta blob) | `Empresas:Leer` (`EmpresasController.cs:74`) |

**Exportación**
- Cuerpo enviado (`EmpresaExportRequest`, `empresa.model.ts:50-58`): `headers` con 7 textos traducidos, en este orden: «Empresa» (`empresa.cols.nombre`), «RUC» (`empresa.cols.ruc`), «Estado» (`empresa.cols.estado`), «Vigente» (`empresa.cols.estaVigente`), «Inicio Trial» (`empresa.fields.fechaInicioTrial`), «Vencimiento» (`empresa.fields.fechaVencimiento`), «Sucursales» (`empresa.fields.sucursalesCount`); `yesLabel` «Sí» (`generic-labels.yes`) y `noLabel` «No» (`generic-labels.no`); y los filtros actuales del formulario (`nombre`, `estado`, `fechaVencimientoDesde`, `fechaVencimientoHasta`). No envía formato de fecha.
- Los filtros que se exportan son los del formulario en ese momento, no necesariamente los de la última búsqueda.
- Archivo: «empresas.xlsx» (`empresa.search.export-filename`).
- Carga masiva: no existe para empresas.

**Comportamientos particulares**
- Restauración de filtros descrita en §0 (clave `/empresas`).
- Los dos selectores de fecha son de solo lectura: solo se eligen con el calendario.

**Observaciones**
- La fila de «sin resultados» muestra «Resultados» (`generic-labels.results`) en lugar de «Sin resultados» (`generic-labels.no-results`), que sí existe en la raíz (`empresa-search.component.html:132`; en Sucursales se usa `no-results`, `sucursales-search.component.html:131`).
- Las claves `empresa.cols.fechaVencimiento` y `empresa.cols.cantidadSucursales` y varias de `form.*`, `notifications.*` y `help.*` no se usan en estas pantallas; ver §9.

### 1.2 `/empresas/:id` — Empresas (detalle de la empresa)

- **Componente:** `EmpresaDetailComponent` (`app/pages/empresa/empresa-detail/empresa-detail.component.ts`, `empresa-detail.component.html`). Scope `empresa`.
- **Para qué sirve:** ver los datos de una empresa y, con permiso, editar su nombre y RUC.
- **Cómo se llega:** enlace del nombre en la tabla de `/empresas` (§1.1). No hay otra entrada (solo por URL con el id).
- **Permiso de ruta:** `Empresas:Leer` (el segundo segmento es un id, no `nueva`). Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Empresas» (`empresa.title`); migas «Inicio» / «Administración» / «Empresa» (`breadcrumbs.empresa`, con enlace a `/empresas`); botón volver a `/empresas` (`.html:3-7`, `.ts:59-63`). Dentro de la tarjeta: encabezado «Empresas» y subtítulo «Datos del registro» (`generic-labels.detail-subtitle`) en modo ver.
- **Modos:** solo **ver** y **editar** (no hay modo crear: `isNew` está declarado y nunca cambia a verdadero, `.ts:40`). Con el parámetro `id` el formulario arranca deshabilitado y carga los datos (`ngOnInit`, `.ts:65-72`); «Editar» lo habilita.
  - **Ver:** campos de solo lectura + botón «Editar» (si tiene `Empresas:Actualizar`).
  - **Editar:** `nombre` y `ruc` habilitados + botones «Cancelar» y «Guardar».

**Campos del formulario**

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Editable |
|---|---|---|---|---|---|---|
| Estado (`empresa.fields.estado`) | — (no está en el formulario) | `matInput` deshabilitado | — | — | — | Nunca. Texto: «Trial Activo», «Activo», «Suspendido» o «Cancelado» (`empresa.estado.<n>`, `.ts:80`) |
| Sucursales (`empresa.fields.sucursalesCount`) | — | Texto deshabilitado | — | — | — | Nunca. Número de sucursales no eliminadas (`cantidadSucursales ?? 0`, `.ts:82`) |
| Inicio Trial (`empresa.fields.fechaInicioTrial`) | — | Texto deshabilitado, formato `dd/MM/yyyy` (`.html:57`) | — | — | — | Nunca |
| Vencimiento (`empresa.fields.fechaVencimiento`) | — | Texto deshabilitado, formato `dd/MM/yyyy` (`.html:67`) | — | — | — | Nunca |
| Nombre de la Empresa (`empresa.fields.nombre`) | `nombre` | `matInput`, `maxlength="150"` | Sí | `Validators.required`, `Validators.maxLength(150)` (`.ts:55`) | «Este campo es obligatorio» (`general.errors.required`; es el único mensaje que muestra la plantilla, `.html:80`) | Solo en modo editar |
| RUC (11 dígitos, opcional) (`empresa.fields.ruc`) | `ruc` | `matInput`, `maxlength="20"` | No | `Validators.maxLength(20)` (`.ts:56`); no valida formato ni dígitos | Sin `mat-error` | Solo en modo editar |

No hay campo para «Vigente» en la plantilla: `estaVigenteDisplay` se asigna (`.ts:81`) pero ninguna parte de `empresa-detail.component.html` lo muestra.

**Estados de suscripción y periodo de prueba que se muestran:** el SPA solo muestra el nombre del estado (`Trial Activo`, `Activo`, `Suspendido`, `Cancelado`, enum `EstadoEmpresa`, `empresa.model.ts:1-6`) y las fechas «Inicio Trial» y «Vencimiento». No muestra días restantes ni calcula el vencimiento. La regla de que una empresa nueva nace en `Trial Activo` con 30 días y la de vigencia están en el backend (`sofia-apis/src/SOFIA.Domain/Entities/Empresa.cs:32-33,37,62-64`; RN-EMP-012 y RN-EMP-014 en 01a).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Empresas:Actualizar`, `.html:27`) | Nunca | `enterEditMode()`: habilita el formulario |
| Cancelar (`actions.cancel`) | Pie de la tarjeta, derecha | Modo editar | Nunca | `cancelEdit()`: restaura `nombre` y `ruc` de la copia leída y deshabilita (`.ts:99-103`) |
| Guardar (`actions.save`); con spinner mientras guarda | Pie de la tarjeta, derecha | Modo editar | `saving()` | `save()` (`.ts:105-125`): si el formulario es inválido marca todo como tocado y no envía; si no, `PUT /api/v1/empresas/:id` con `{ nombre, ruc }` (`ruc` como nulo solo si es nulo; una cadena vacía se envía vacía). Éxito → aviso, vuelve a modo ver y recarga |
| Botón volver (icono `chevron_left`) | Cabecera de página | Siempre | — | Navega a `/empresas` |

No hay botón «Eliminar».

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Guardado correcto | Éxito | «Empresa actualizada correctamente» (`empresa.detail.save-success`) | `empresa-detail.component.ts:115` |
| Cualquier error al guardar | Error | «Error al actualizar la empresa» (`empresa.detail.save-error`) | `:122` |
| Cualquier error al cargar | Error | «Error al cargar la empresa» (`empresa.detail.load-error`) | `:89` |

No usa `saveErrorKey`: ningún código del backend se traduce (por ejemplo el RUC duplicado `Empresa.RUC.Duplicado` o el formato inválido muestran el mismo «Error al actualizar la empresa»). Existen las claves `empresa.notifications.rucDuplicado` y `empresa.form.rucFormat`, pero este componente no las usa.

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Al abrir y tras guardar | `EmpresaService.getById` (`empresa.service.ts:28-30`) | `GET /api/v1/empresas/{id}` | `Empresas:Leer` (`EmpresasController.cs:57`) |
| Guardar | `EmpresaService.update` (`:32-34`) | `PUT /api/v1/empresas/{id}` | `Empresas:Actualizar` (`EmpresasController.cs:65`) |

**Exportación y carga masiva:** no hay en esta pantalla.

**Comportamientos particulares**
- Mientras carga muestra un spinner en lugar de la tarjeta (`.html:9-13`).
- «Editar» no se muestra a quien no tiene `Empresas:Actualizar`; el rol `Admin` siempre lo ve.

**Observaciones**
- Discrepancias con el backend (`sofia-apis/src/SOFIA.Application/Empresas/Commands/UpdateEmpresa/UpdateEmpresaCommandValidator.cs:14-21`): el formulario limita el nombre a 150 caracteres y la API admite 200 (RN-EMP-002); el RUC no se valida en el formulario (solo 20 caracteres) y la API exige 11 dígitos exactos si se envía no nulo (RN-EMP-004). Un RUC vacío (el usuario borra el texto) se envía como cadena vacía, que la API trata como no nulo y rechaza por longitud, y el usuario ve el mensaje genérico.
- El rótulo del campo RUC dice «(11 dígitos, opcional)», pero el control no lo impone.
- Código muerto: `isNew`, `estaVigenteDisplay` y `EstadoEmpresa` expuesto a la plantilla sin uso en `.html`; clave `empresa.detail.title-edit` sin uso.
- `app/pages/empresa/empresa-maintenance` (`EmpresaMaintenanceComponent`, selector `app-empresa-maintenance`) **no tiene ruta** (`empresa.routes.ts` solo define `''` y `:id`) y **ninguna referencia** fuera de sus propios archivos (búsqueda de `EmpresaMaintenanceComponent` y `app-empresa-maintenance` en `src/`): es código sin uso. Importa el scope `empresa`, su formulario valida el RUC con `/^\d{11}$/` y el nombre hasta 200 (`empresa-maintenance.component.ts`), pero nunca se muestra. `empresa-register` lo cubre la parte A.

---

## 2. Sucursales

Carpeta `app/pages/sucursales`, scope i18n `sucursales`. Componentes enrutados: `SucursalesSearchComponent`, `SucursalesDetailComponent` (rutas `nueva` y `:id`) y `SucursalesCargaMasivaComponent` (`app/pages/sucursales/sucursales.routes.ts`). El orden de rutas importa: `nueva` y `carga-masiva` están antes de `:id`.

### 2.1 `/sucursales` — Sucursales (búsqueda)

- **Componente:** `SucursalesSearchComponent` (`app/pages/sucursales/sucursales-search/sucursales-search.component.ts`, `.html`). Scope `sucursales`.
- **Para qué sirve:** buscar sucursales de la empresa por nombre, licencia y dirección, abrirlas, exportar la lista, y entrar a crear o a cargar sucursales masivamente.
- **Cómo se llega:** menú lateral, grupo «Administración» → «Sucursales» (`nav.sucursales`, `sidebar-data.ts:26-30`); botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar una sucursal.
- **Permiso de ruta:** `Sucursales:Leer` (`route-permissions.ts:4`).
- **Migas y cabecera:** título «Sucursales» (`sucursales.title`); migas «Inicio» / «Administración» / «Sucursales» (`breadcrumbs.sucursales`, activa); sin botón volver. Tarjeta «Búsqueda de Sucursales» (`sucursales.search.title`) con el subtítulo común (`sucursales-search.component.html:3-16`, `.ts:43-47`).
- **Modos:** uno solo. Los resultados aparecen tras «Buscar» o por estado guardado (§0).

**Filtros de búsqueda** (`.ts:37-41`, `.html:37-65`)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Nombre (`sucursales.fields.nombre`) | `nombre` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Nº de Licencia (`sucursales.fields.numeroLicencia`) | `numeroLicencia` | Texto con lupa; Enter busca | Texto libre | Vacío |
| Dirección Física (`sucursales.fields.direccionFisica`) | `direccionFisica` | Texto con lupa; Enter busca | Texto libre | Vacío |

«Limpiar» vacía los tres campos, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:95-102`).

**Tabla de resultados** (`.html:85-151`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Nombre (`sucursales.table.nombre`) | `nombre` | Texto | Sí → `/sucursales/:id` (`[routerLink]="[row.id]"`) |
| Nº de Licencia (`sucursales.table.numeroLicencia`) | `numeroLicencia` | Texto | No |
| Dirección Física (`sucursales.table.direccionFisica`) | `direccionFisica` | Texto | No |
| Gerente (`sucursales.table.gerenteNombre`) | `gerenteNombre` | Texto; «—» si es nulo | No |

Encabezado «Resultados (N)». Paginación 10/25/50. Sin resultados: «Sin resultados» (`generic-labels.no-results`). Bajo la tabla: botón «Exportar» (icono `download`) y paginador.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`, raíz), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `Sucursales:Crear`, `.html:19`) | Nunca | Navega a `carga-masiva` relativo → `/sucursales/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/sucursales/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`) | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/sucursales` |
| Exportar (`actions.export`) | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/sucursales/exportar`, descarga `sucursales.xlsx` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda o la exportación | Error | «Error al buscar sucursales» (`sucursales.search.search-error`) | `sucursales-search.component.ts:90,138` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar o paginar | `SucursalService.search` (`app/pages/sucursales/services/sucursal.service.ts:22-30`) | `GET /api/v1/sucursales?pageNumber&pageSize[&nombre][&numeroLicencia][&direccionFisica]` | `Sucursales:Leer` (`SucursalesController.cs:56`) |
| Exportar | `SucursalService.exportar` (`:52-57`) | `POST /api/v1/sucursales/exportar` | `Sucursales:Leer` (`SucursalesController.cs:81`) |

**Exportación**
- Cuerpo: `{ headers, nombre?, numeroLicencia?, direccionFisica? }`. `headers` (4, en orden): «Nombre» (`sucursales.table.nombre`), «Dirección Física» (`sucursales.table.direccionFisica`), «Nº de Licencia» (`sucursales.table.numeroLicencia`), «Gerente» (`sucursales.table.gerenteNombre`) (`.ts:114-121`). No envía fechas ni textos sí/no.
- Archivo: «sucursales.xlsx» (`sucursales.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §2.3.

**Comportamientos particulares**
- `search()` no vuelve a la página 1: usa el índice de página actual (`pageIndex() + 1`, `.ts:73`). Si estás en la página 3 y cambias un filtro y pulsas «Buscar», se pide la página 3 de los nuevos filtros. Empresa sí reinicia a la página 1 (§1.1).

**Observaciones**
- Se muestra «Nuevo» y «Carga Masiva» a quien tiene `Sucursales:Crear`.

### 2.2 `/sucursales/nueva` y `/sucursales/:id` — Sucursales (detalle)

- **Componente:** `SucursalesDetailComponent` (`app/pages/sucursales/sucursales-detail/sucursales-detail.component.ts`, `.html`). Scope `sucursales`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear una sucursal, ver sus datos y, con permisos, editarla o eliminarla.
- **Cómo se llega:** `/sucursales/nueva`: botón «Nuevo» de la búsqueda. `/sucursales/:id`: enlace del nombre en la búsqueda.
- **Permiso de ruta:** `/sucursales/nueva` → `Sucursales:Leer` + `Sucursales:Crear`; `/sucursales/:id` → `Sucursales:Leer` (`route-permissions.ts:68-72`).
- **Migas y cabecera:** título «Sucursales» (`sucursales.title`); migas «Inicio» / «Administración» / «Sucursales» (con enlace a `/sucursales`); botón volver a `/sucursales` (`.html:3-7`, `.ts:66-70`). Misma cabecera en crear y en ver.
- **Modos** (`ngOnInit`, `.ts:77-87`): sin parámetro `id` → **crear** (`isNew` e `isEditMode` verdaderos, formulario habilitado); con `id` → **ver** (formulario deshabilitado, carga datos) y «Editar» lo habilita.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado «Sucursales» + «Datos del registro» + botón «Editar» | No | Sí (botón solo con `Sucursales:Actualizar`) | No |
| Nombre, Nº de Licencia, Dirección Física | Editables | Deshabilitados | Editables |
| Gerente | Buscador (editable) | Texto deshabilitado | Buscador (editable) |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:60-64`, `.html:37-82`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Nombre (`sucursales.fields.nombre`) | `nombre` | `matInput`, `maxlength="100"` | Sí | `required`, `maxLength(100)` (`.ts:61`) | «Este campo es obligatorio» (`general.errors.required`) | Sí / Sí |
| Nº de Licencia (`sucursales.fields.numeroLicencia`) | `numeroLicencia` | `matInput`, `maxlength="50"` | Sí | `required`, `maxLength(50)` (`.ts:63`) | Ídem | Sí / Sí |
| Dirección Física (`sucursales.fields.direccionFisica`) | `direccionFisica` | `matInput`, `maxlength="255"` | Sí | `required`, `maxLength(255)` (`.ts:62`) | Ídem | Sí / Sí |
| Gerente (`sucursales.fields.gerente`) | — (fuera del formulario; `gerenteId` en memoria) | En editar/crear: `csw-autocomplete-input` (placeholder «Gerente», mínimo 2 letras, `clearOnInvalidSelection`); en ver: `matInput` deshabilitado con la etiqueta «Gerente» | No | Solo se puede elegir un empleado de la lista; si se escribe un texto que no coincide, se vacía | — | Sí / Sí |

El buscador del gerente llama a `GET /api/v1/empleados?pageNumber=1&pageSize=20&searchTerm=<texto>` y muestra `nombre_Completo` de cada empleado (`sucursal.service.ts:59-65`, `.ts:72-75`); espera 500 ms tras escribir ([19a §6.3](19a-acceso-layout-y-compartidos.md)). Limpiar el campo envía `gerenteId` nulo al editar; al crear, un gerente no elegido se omite (`undefined`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta | Modo ver y `*appPermiso="'Actualizar'"` (= `Sucursales:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()`: en crear navega a `/sucursales`; en editar restaura los datos leídos (incluido el gerente) y vuelve a ver (`.ts:115-129`) |
| Guardar (`actions.save`) | Pie | Crear o editar | `saving()` | `save()` (`.ts:135-175`). Inválido → marca tocados y no envía. Crear → `POST /api/v1/sucursales`, aviso de éxito y navega a `/sucursales`. Editar → `PUT /api/v1/sucursales/:id` con `gerenteId` (nulo si se limpió), aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, a la derecha de Guardar | Solo en editar de una sucursal existente (`!isNew()`) y `*appPermiso="'Eliminar'"` (= `Sucursales:Eliminar`, `.html:103`) | `deleting()` | `confirmDelete()` → diálogo |
| Botón volver | Cabecera | Siempre | — | `/sucursales` |

**Diálogos que abre:** `ConfirmDialogComponent` (ancho 400 px, [19a §6.1](19a-acceso-layout-y-compartidos.md)) al pulsar «Eliminar»: título «Confirmar eliminación» (`sucursales.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta sucursal?» (`sucursales.detail.delete-confirm-message`), botones por defecto «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/sucursales/:id`; cancelar no hace nada (`.ts:177-205`).

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Sucursal guardada correctamente» (`sucursales.detail.save-success`) | `.ts:152,163` |
| Código `Sucursal.NumeroLicencia.Duplicado` | Error | «Ya existe una sucursal con ese número de licencia.» (`sucursales.detail.duplicate-licencia`) | `.ts:19-21,157,171` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` (vía `saveErrorKey`) | Error | «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» / «El registro hace referencia a datos que ya no existen o todavía está en uso.» / «Uno de los valores está fuera del rango permitido.» (`errors.dbDuplicado`, `errors.dbReferenciaInvalida`, `errors.dbValorInvalido`) | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar | Error | «Error al guardar la sucursal» (`sucursales.detail.save-error`) | `.ts:157,171` |
| Eliminación correcta | Éxito | «Sucursal eliminada correctamente» (`sucursales.detail.delete-success`); navega a `/sucursales` | `.ts:194` |
| Eliminar una sucursal en uso: la API responde HTTP 409 (`Sucursal.InUse`); el SPA decide por el estado HTTP, no por el código | Error | «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.» (`sucursales.detail.delete-in-use`) | `.ts:199-202`; `public/i18n/sucursales/es.json:14` |
| Cualquier error al eliminar cuyo estado HTTP no sea 409 (por ejemplo 404 `Sucursal.NotFound`, 403, 500 o sin conexión) | Error | «Error al eliminar la sucursal» (`sucursales.detail.delete-error`) | `.ts:199-202` |
| Error al cargar | Error | «Error al cargar la sucursal» (`sucursales.detail.load-error`) | `.ts:105` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir / tras guardar | `SucursalService.getById` | `GET /api/v1/sucursales/{id}` | `Sucursales:Leer` (`SucursalesController.cs:73`) |
| Crear | `create` | `POST /api/v1/sucursales` | `Sucursales:Crear` (`:115`) |
| Editar | `update` | `PUT /api/v1/sucursales/{id}` | `Sucursales:Actualizar` (`:125`) |
| Eliminar | `delete` | `DELETE /api/v1/sucursales/{id}` (204; 404 `Sucursal.NotFound`; 409 `Sucursal.InUse` si la sucursal está en uso, `sofia-apis/src/SOFIA.Application/Sucursales/Commands/DeleteSucursal/DeleteSucursalCommandHandler.cs:15-25`) | `Sucursales:Eliminar` (`:139`) |
| Buscar gerente | `searchEmpleados` | `GET /api/v1/empleados?pageNumber=1&pageSize=20&searchTerm=…` | `Empleados:Leer` (`EmpleadosController.cs:16`) |

**Comportamientos particulares**
- La eliminación es la eliminación lógica de la API. Antes de borrar, el backend comprueba que la sucursal no esté referenciada (registros no eliminados) por: empleados con esa sucursal base, cuentas, roles, stock (lotes en sucursal), sesiones de caja, series fiscales, ventas, transferencias (como origen o destino), inventario en cuarentena DIGEMID y órdenes de producción magistral; si lo está responde 409 `Sucursal.InUse` y no borra (`sofia-apis/src/SOFIA.Application/Sucursales/Commands/DeleteSucursal/DeleteSucursalCommandHandler.cs:20-25,34-44`). El SPA lo explica con «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.» (`sucursales.detail.delete-in-use`) cuando el estado HTTP es 409 (`.ts:199-202`); el SPA no avisa antes de confirmar, solo después de la respuesta. La regla RN-SUC-009 de 01a todavía dice que no se comprueban dependencias (ver el informe de reconciliación).
- Al editar, el gerente no se envía si no lo tocas: se reenvía el que ya tenía (`gerenteId` en memoria).
- Quien no tiene `Empleados:Leer` no podrá buscar gerente (el endpoint exige ese permiso); el SPA no lo comprueba. «No verificado» qué ve en ese caso (el buscador queda sin opciones o con error silencioso; no se probó).

**Observaciones**
- Las validaciones del formulario coinciden con las de la API (100, 255, 50; RN-SUC-002 a 004).
- La API también rechaza la licencia duplicada en la edición con el mismo código (`UpdateSucursalCommandHandler.cs:25`).
- La sede principal que se crea al registrar la empresa nace con el número de licencia vacío si en el registro no se envía licencia (`Sucursal.CreateSedePrincipal`, `sofia-apis/src/SOFIA.Domain/Entities/Sucursal.cs:56-76`; `RegistrarEmpresaCommandHandler.cs:85-91`); la pantalla de registro del SPA no envía licencia (ver [19a, `/auth/register`](19a-acceso-layout-y-compartidos.md)). Al abrir esa sede en `/sucursales/:id` el campo «Nº de Licencia» aparece vacío y, como es obligatorio (`required`, `.ts:63`) y la API también lo exige al editar (`Sucursal.cs:94-97`, `UpdateSucursalCommandValidator.cs:23-25`), solo se puede guardar tras completarlo. En la tabla de `/sucursales` esa celda se muestra en blanco (no «—», `sucursales-search.component.html:105`).

### 2.3 `/sucursales/carga-masiva` — Carga Masiva (sucursales)

- **Componente:** `SucursalesCargaMasivaComponent` (`app/pages/sucursales/sucursales-carga-masiva/sucursales-carga-masiva.component.ts`, plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `sucursales`; los textos los resuelve el componente común contra la raíz `carga-masiva.*`.
- **Para qué sirve:** cargar varias sucursales desde un Excel. El flujo común (descargar plantilla, elegir archivo, vista previa, guardar válidos) está en [19a §6.5](19a-acceso-layout-y-compartidos.md); aquí solo la configuración que pasa esta pantalla.
- **Cómo se llega:** botón «Carga Masiva» de `/sucursales` (§2.1).
- **Permiso de ruta:** `Sucursales:Leer` + `Sucursales:Crear` (segundo segmento `carga-masiva`, `route-permissions.ts:36,68-72`).
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Administración» / «Sucursales» (enlace a `/sucursales`) / «Carga Masiva» (`breadcrumbs.carga-masiva`, activa); botón volver a `/sucursales` (`.ts:23-28`).

**Configuración que pasa** (`CargaMasivaConfig`, `.ts:18-34`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/sucursales/plantilla` — `Sucursales:Crear` (`SucursalesController.cs:22`) |
| `previewUrl` | `POST {api}/api/v1/sucursales/previsualizar` — `Sucursales:Crear` (`:30`) |
| `saveUrl` | `POST {api}/api/v1/sucursales/carga-masiva` — `Sucursales:Crear` (`:47`) |
| `backRoute` | `/sucursales` (el archivo de plantilla se descarga como `plantilla-sucursales.xlsx`, [19a §6.5](19a-acceso-layout-y-compartidos.md)) |
| Columnas de la vista previa (todas obligatorias) | «Nombre» (`fields.nombre`, clave `nombre`), «Dirección Física» (`fields.direccionFisica`, clave `direccionFisica`), «Nº de Licencia» (`fields.numeroLicencia`, clave `numeroLicencia`) |

El gerente no forma parte de la carga masiva. Las columnas se rotulan con las claves de la **raíz** `fields.*` (no con las de `sucursales.fields.*`).

**Botones, mensajes y errores:** los del componente común ([19a §6.5](19a-acceso-layout-y-compartidos.md) y §8.9). Los códigos de fila de sucursales que puede traer la vista previa (`required`, `max-length`, `duplicate`, `duplicate-in-file`, ver RN-SUC-013 en 01a) usan los textos de `carga-masiva.validation.*`.

**Observaciones:** el scope `sucursales` se carga pero no aporta textos a esta pantalla.

---

## 3. Empleados

Carpeta `app/pages/empleados`, scope i18n `empleados`. Componentes enrutados: `EmpleadosSearchComponent` y `EmpleadosDetailComponent` (rutas `nueva` y `:id`) (`app/pages/empleados/empleados.routes.ts`). No hay carga masiva de empleados en el SPA.

**Relaciones que muestra esta pantalla**
- **Sucursal base:** obligatoria; se elige de las sucursales de la empresa (§3.2). Es la única relación editable.
- **Cuenta de usuario:** el formulario del empleado **no** muestra ni edita la cuenta. La relación se establece desde la ficha de la cuenta, donde se elige el empleado (§5.2). Desde el empleado no se ve si tiene cuenta.
- **Gerente de sucursal:** un empleado que gerencia una sucursal no se puede eliminar (mensaje en §3.2).
- **Campos opcionales:** Apellido Materno, Licencia Profesional y Correo electrónico (el correo sirve para el enlace de recuperación de contraseña, según el texto de ayuda).

### 3.1 `/empleados` — Empleados (búsqueda)

- **Componente:** `EmpleadosSearchComponent` (`app/pages/empleados/empleados-search/empleados-search.component.ts`, `.html`). Scope `empleados`.
- **Para qué sirve:** buscar empleados por nombres, apellidos, licencia profesional y sucursal base, abrirlos, exportar y entrar a crear.
- **Cómo se llega:** menú lateral, grupo «Administración» → «Empleados» (`nav.empleados`, `sidebar-data.ts:31-35`); volver desde el detalle.
- **Permiso de ruta:** `Empleados:Leer` (`route-permissions.ts:5`).
- **Migas y cabecera:** título «Empleados» (`empleados.title`); migas «Inicio» / «Administración» / «Empleados» (`breadcrumbs.empleados`, activa); sin botón volver. Tarjeta «Búsqueda de Empleados» (`empleados.search.title`) con subtítulo común.
- **Modos:** uno. Resultados tras «Buscar» o con estado guardado (§0).

**Filtros de búsqueda** (`.ts:52-58`, `.html:29-101`)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Nombres (`empleados.fields.nombres`) | `nombres` | Texto con lupa, Enter busca | Libre | Vacío |
| Apellido Paterno (`empleados.fields.apellidoPaterno`) | `apellidoPaterno` | Texto con lupa, Enter busca | Libre | Vacío |
| Apellido Materno (`empleados.fields.apellidoMaterno`) | `apellidoMaterno` | Texto con lupa, Enter busca | Libre | Vacío |
| Licencia Profesional (`empleados.fields.licencia`) | `licencia` | Texto con lupa, Enter busca | Libre | Vacío |
| Sucursal Base (`empleados.fields.sucursal`) | `sucursalNombre` | Texto con lupa y autocompletado (`mat-autocomplete`); placeholder «Buscar sucursal...» (`empleados.autocomplete.sucursal-placeholder`) | Escribir ofrece hasta 20 sucursales cuyo nombre contenga el texto (espera 300 ms); elegir una pone su nombre en el campo. Si no hay coincidencias muestra la opción deshabilitada «Sin resultados» (`empleados.autocomplete.no-results`). El filtro enviado es el **texto** del campo, haya o no una opción elegida (`.ts:109`) | Vacío |

«Limpiar» vacía el formulario, la lista de sugerencias y la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:132-140`).

**Tabla de resultados** (`.html:121-205`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Nombre Completo (`empleados.table.nombreCompleto`) | `nombre_Completo` | Texto | Sí → `/empleados/:id` |
| Apellido Paterno (`empleados.table.apellidoPaterno`) | `apellido_Paterno` | Texto | No |
| Apellido Materno (`empleados.table.apellidoMaterno`) | `apellido_Materno` | Texto; celda vacía si es nulo (no usa «—», `.html:150`) | No |
| Licencia Profesional (`empleados.table.licencia`) | `licencia_Prof` | Texto; «—» si es nulo | No |
| Correo electrónico (`empleados.table.email`) | `email` | Texto; «—» si es nulo | No |
| Sucursal Base (`empleados.table.sucursal`) | `sucursalNombre` | Texto | No |

Encabezado «Resultados (N)». Paginación 10/25/50. Sin resultados: «Sin resultados» (`generic-labels.no-results`). Botón «Exportar» bajo la tabla.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros | `*appPermiso="'Crear'"` (= `Empleados:Crear`, `.html:19`) | Nunca | `goToNew()` → `/empleados/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`) | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/empleados` |
| Exportar (`actions.export`) | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/empleados/exportar`, descarga `empleados.xlsx` |

No hay botón de carga masiva.

**Diálogos que abre:** ninguno.

**Mensajes al usuario:** falla la búsqueda o la exportación → error «Error al buscar empleados» (`empleados.search.search-error`, `.ts:127,180`). Sin mapa de códigos.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Buscar / paginar | `EmpleadoService.search` (`app/pages/empleados/services/empleado.service.ts:18-28`) | `GET /api/v1/empleados?pageNumber&pageSize[&nombres][&apellidoPaterno][&apellidoMaterno][&licencia][&sucursalNombre]` | `Empleados:Leer` (`EmpleadosController.cs:16`) |
| Escribir en Sucursal Base | `SucursalService.search` | `GET /api/v1/sucursales?pageNumber=1&pageSize=20&nombre=…` | `Sucursales:Leer` (`SucursalesController.cs:56`) |
| Exportar | `EmpleadoService.exportar` (`:46-57`) | `POST /api/v1/empleados/exportar` | `Empleados:Leer` (`:64`) |

**Exportación:** cuerpo `{ headers, nombres?, apellidoPaterno?, apellidoMaterno?, licencia?, sucursalNombre? }` con los filtros actuales del formulario. `headers` (6, en orden): «Nombres» (`empleados.table.nombres`), «Apellido Paterno», «Apellido Materno», «Licencia Profesional», «Correo electrónico», «Sucursal Base» (`empleados.table.*`, `.ts:154-161`). Archivo «empleados.xlsx» (`empleados.search.export-filename`). No envía formato de fecha (no hay fechas).

**Comportamientos particulares:** `search()` no reinicia la página (igual que Sucursales, §2.1). Un usuario sin `Sucursales:Leer` no recibirá sugerencias en Sucursal Base (el endpoint lo exige); el SPA no lo comprueba. «No verificado» qué ve en ese caso.

**Observaciones:** la columna «Apellido Materno» deja la celda vacía cuando es nulo, a diferencia de Licencia y Correo («—»).

### 3.2 `/empleados/nueva` y `/empleados/:id` — Empleados (detalle)

- **Componente:** `EmpleadosDetailComponent` (`app/pages/empleados/empleados-detail/empleados-detail.component.ts`, `.html`). Scope `empleados`. Misma clase para las dos rutas.
- **Para qué sirve:** crear un empleado, ver sus datos, editarlo o eliminarlo.
- **Cómo se llega:** «Nuevo» (crear) o el enlace del nombre en la búsqueda (ver/editar).
- **Permiso de ruta:** `/empleados/nueva` → `Empleados:Leer` + `Empleados:Crear`; `/empleados/:id` → `Empleados:Leer`.
- **Migas y cabecera:** título «Empleados»; migas «Inicio» / «Administración» / «Empleados» (con enlace a `/empleados`); botón volver a `/empleados`.
- **Modos** (`ngOnInit`, `.ts:101-112`): sin `id` → **crear**; con `id` → **ver** (formulario y buscador deshabilitados) y «Editar» → **editar**.

| Aspecto | Crear | Ver | Editar |
|---|---|---|---|
| Encabezado «Empleados» + «Datos del registro» + «Editar» | No | Sí (botón solo con `Empleados:Actualizar`) | No |
| Campos | Habilitados | Deshabilitados | Habilitados |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:73-79`, `.html:37-117`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Ayuda |
|---|---|---|---|---|---|---|
| Nombres (`empleados.fields.nombres`) | `nombres` | `matInput`, `maxlength="75"` | Sí | `required`, `maxLength(75)` (`.ts:74`) | «Este campo es obligatorio» (`general.errors.required`) | — |
| Apellido Paterno (`empleados.fields.apellidoPaterno`) | `apellidoPaterno` | `matInput`, `maxlength="75"` | Sí | `required`, `maxLength(75)` (`.ts:75`) | Ídem | — |
| Apellido Materno (`empleados.fields.apellidoMaterno`) | `apellidoMaterno` | `matInput`, `maxlength="75"` | No | `maxLength(75)` (`.ts:76`) | — | Si queda vacío o solo espacios se envía nulo (`.ts:184`) |
| Sucursal Base (`empleados.fields.sucursal`) | `sucursalCtrl` (control aparte, `[formControl]`) | `matInput` con `mat-autocomplete`; placeholder «Buscar sucursal...» | Sí | Debe haberse **elegido** una opción de la lista y el texto debe seguir igual al nombre elegido; si no, `selectedSucursal` queda nulo (`.ts:166-176`) | «Este campo es obligatorio» (`general.errors.required`) bajo el campo, tras pulsar «Guardar» (`.html:88-92`) | La búsqueda ofrece hasta 20 sucursales cuyo nombre contenga lo escrito, espera 300 ms; sin coincidencias, opción deshabilitada «Sin resultados» (`empleados.autocomplete.no-results`) |
| Licencia Profesional (`empleados.fields.licencia`) | `licencia` | `matInput`, `maxlength="50"` | No | `maxLength(50)` (`.ts:77`) | — | Cadena vacía → se omite del cuerpo (`undefined`, `.ts:185`) |
| Correo electrónico (`empleados.fields.email`) | `email` | `matInput type="email"`, `maxlength="254"`, `autocomplete="off"` | No | `Validators.email`, `maxLength(254)` (`.ts:78`) | Si falla el formato: «Ingresa un correo electrónico válido.» (`empleados.detail.invalid-email`) | «Opcional. Se usa para enviar el enlace de recuperación de contraseña.» (`empleados.fields.email-hint`); el correo se recorta con `trim()` y, si queda vacío, se omite (`.ts:186`) |

Detalle: el buscador de sucursal solo consulta cuando el usuario escribe (el cambio programático se hace con `emitEvent: false`), por lo que al abrir la ficha el campo muestra el nombre guardado sin lanzar búsqueda (`.ts:140,145`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta | Modo ver y `*appPermiso="'Actualizar'"` (= `Empleados:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()`: en crear navega a `/empleados`; en editar restaura la copia leída (`.ts:154-164`) |
| Guardar (`actions.save`) | Pie | Crear o editar | `saving()` | `save()` (`.ts:166-203`): marca todo como tocado; si el formulario es inválido o no hay sucursal elegida no envía. Crear → `POST /api/v1/empleados`; editar → `PUT /api/v1/empleados/:id`. En ambos casos, aviso de éxito y navega a `/empleados` |
| Eliminar (`actions.delete`) | Pie, tras Guardar | Solo editar de un empleado existente y `*appPermiso="'Eliminar'"` (= `Empleados:Eliminar`, `.html:137`) | `deleting()` | `confirmDelete()` |
| Botón volver | Cabecera | Siempre | — | `/empleados` |

**Diálogos que abre:** `ConfirmDialogComponent` (400 px) al pulsar «Eliminar»: título «Confirmar eliminación» (`empleados.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este empleado?» (`empleados.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/empleados/:id`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Empleado guardado correctamente» (`empleados.detail.save-success`) | `.ts:190` |
| Código `Empleado.LicenciaProf.Duplicado` | Error | «Ya existe un empleado con esa licencia profesional.» (`empleados.detail.duplicate-licencia`) | `.ts:20,195` |
| Código `Empleado.Email.Duplicado` | Error | «Ya existe un empleado con ese correo electrónico en tu empresa.» (`empleados.detail.duplicate-email`) | `.ts:21` |
| Código `Empleado.Email.Invalido` | Error | «Ingresa un correo electrónico válido.» (`empleados.detail.invalid-email`) | `.ts:22` |
| `Db.Duplicado` / `Db.ReferenciaInvalida` / `Db.ValorInvalido` | Error | Textos de `errors.dbDuplicado`, `errors.dbReferenciaInvalida`, `errors.dbValorInvalido` | `save-error-key.util.ts:4-9` |
| Cualquier otro error al guardar (por ejemplo `Empleado.SucursalBase.AutoAsignacion`, RN-EPL-011, o `Sucursal.NotFound`) | Error | «Error al guardar el empleado» (`empleados.detail.save-error`) | `.ts:195` |
| Eliminación correcta | Éxito | «Empleado eliminado correctamente» (`empleados.detail.delete-success`); navega a `/empleados` | `.ts:222` |
| Eliminar con respuesta HTTP 409 (gerente de sucursal, RN-EPL-014) | Error | «No se puede eliminar un empleado que gerencia una sucursal. Asigna otro gerente primero.» (`empleados.detail.delete-manager`) | `.ts:227-231` |
| Otro error al eliminar | Error | «Error al eliminar el empleado» (`empleados.detail.delete-error`) | `.ts:230` |
| Error al cargar | Error | «Error al cargar el empleado» (`empleados.detail.load-error`) | `.ts:124` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir | `EmpleadoService.getById` | `GET /api/v1/empleados/{id}` | `Empleados:Leer` (`EmpleadosController.cs:24`) |
| Crear | `create` | `POST /api/v1/empleados` | `Empleados:Crear` (`:32`) |
| Editar | `update` | `PUT /api/v1/empleados/{id}` | `Empleados:Actualizar` (`:42`) |
| Eliminar | `delete` | `DELETE /api/v1/empleados/{id}` | `Empleados:Eliminar` (`:56`) |
| Escribir en Sucursal Base | `SucursalService.search` | `GET /api/v1/sucursales?pageNumber=1&pageSize=20&nombre=…` | `Sucursales:Leer` |

El cuerpo de crear y editar es el mismo: `{ sucursal_Base_ID, nombres, apellido_Paterno, apellido_Materno (nulo si vacío), licencia_Prof?, email? }` (`empleado.model.ts:30-39`).

**Comportamientos particulares**
- Tras guardar (también al editar) el SPA vuelve a la búsqueda; no se queda en la ficha, a diferencia de Sucursales y Empresa.
- Un empleado no puede cambiar su propia sucursal base (RN-EPL-011): la API responde 403 `Empleado.SucursalBase.AutoAsignacion` y el SPA no lo traduce (mensaje genérico).
- Al cambiar la sucursal base se invalida la sesión de las cuentas asociadas (RN-EPL-012); el SPA no avisa.

**Observaciones**
- Validaciones coherentes con la API (75/75/75/50/254; RN-EPL-004 a 009).
- `SAVE_ERROR_KEYS` y dos imports están declarados entre bloques de imports (`.ts:19-26`): solo estilo.
- La clave `empleados.table.nombres` solo se usa como encabezado de la exportación (la tabla en pantalla usa «Nombre Completo»).

---

## 4. Roles y permisos

Carpeta `app/pages/roles`, scope i18n `roles`. Componentes enrutados: `RolesSearchComponent`, `RolesDetailComponent` (rutas `nueva` y `:id`) y `RolesCargaMasivaComponent` (`app/pages/roles/roles.routes.ts`). El módulo de permisos de todas las rutas es `Seguridad` (`route-permissions.ts:6`); las cuentas de usuario (§5) comparten ese módulo.

Resumen de lo que se puede hacer sobre un rol (comprobado en las fichas):
- **Crear:** nombre, descripción y nivel de jerarquía. Los permisos y las sucursales se definen después, al editar el rol (el bloque «Configuración del Rol» no aparece al crear).
- **Editar:** solo descripción y nivel; el nombre queda bloqueado. Además, con `Seguridad:GestionarPermisos`: permisos (matriz por módulo y acción) y sucursales del rol, que se guardan todos juntos con «Guardar».
- **Eliminar** y **carga masiva** con sus permisos.
- **Rol `Admin`:** en las pantallas de roles el SPA **no tiene ninguna regla visible** sobre un rol llamado `Admin` (no oculta, bloquea ni avisa; ver búsqueda `Admin` en `app/pages/roles`: solo aparece en un comentario). Lo que sí existe está en la API: `Rol.AdminReservado` al asignar o quitar el rol Admin a una cuenta (`sofia-apis/src/SOFIA.Application/Security/Commands/Roles/AssignRol/AssignRolToUserCommandHandler.cs:36-38`, ver §5.2) y `Rol.PermisosPropios` al ampliar los permisos de un rol que uno mismo tiene asignado sin ser Admin (`AssignPermissionToRolCommandHandler.cs:26`, RN-ROL-010). En la vista previa del menú de un rol llamado «Admin» todas las pantallas salen activas porque la comprobación trata ese nombre como administrador (`roles-detail.component.ts:73-79`, `route-permissions.ts:53`).

### 4.1 `/roles` — Roles (búsqueda)

- **Componente:** `RolesSearchComponent` (`app/pages/roles/roles-search/roles-search.component.ts`, `.html`). Scope `roles`.
- **Para qué sirve:** buscar roles por nombre, descripción y nivel de jerarquía, abrirlos, exportar y entrar a crear o a la carga masiva.
- **Cómo se llega:** menú lateral, grupo «Administración» → «Roles & Permisos» (`nav.rolesPermisos`, `sidebar-data.ts:36-40`); volver desde el detalle y la carga masiva.
- **Permiso de ruta:** `Seguridad:Leer`.
- **Migas y cabecera:** título «Roles» (`roles.title`); migas «Inicio» / «Administración» / «Roles & Permisos» (`breadcrumbs.roles`, activa); sin botón volver. Tarjeta «Búsqueda de Roles» (`roles.search.title`) con subtítulo común.
- **Modos:** uno. Resultados tras «Buscar» o con estado guardado (§0).

**Filtros de búsqueda** (`.ts:52-55`, `.html:37-68`)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Nombre del Rol (`roles.fields.nombreRol`) | `nombreRol` | Texto con lupa, Enter busca | Libre | Vacío |
| Descripción (`roles.fields.descripcion`) | `descripcion` | Texto con lupa, Enter busca | Libre | Vacío |
| Nivel de Jerarquía (`roles.fields.nivelJerarquia`) | señales `nivelMin` / `nivelMax` (no es `formControlName`) | Deslizador de rango (`app-range-slider-field`, [19a §6.4](19a-acceso-layout-y-compartidos.md)) | Mínimo 0, máximo 100, paso 1 (`NIVEL_MIN`, `NIVEL_MAX`, `.ts:40-41`). El filtro «desde» solo se envía si es distinto de 0 y «hasta» solo si es distinto de 100 (`.ts:85-86`) | 0 – 100 |

«Limpiar» vacía los textos, devuelve el deslizador a 0–100, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:109-118`). El deslizador no se guarda en el estado restaurado (`SearchPageState.formValues` solo contiene los dos textos); al volver a la pantalla se restauran los textos pero el rango vuelve a 0–100.

**Tabla de resultados** (`.html:88-149`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Nombre del Rol (`roles.table.nombreRol`) | `nombreRol` | Texto | Sí → `/roles/:id` |
| Descripción (`roles.table.descripcion`) | `descripcion` | Texto; «—» si es nulo | No |
| Nivel de Jerarquía (`roles.table.nivelJerarquia`) | `nivelJerarquia` | Número | No |

Encabezado «Resultados (N)». Paginación 10/25/50. **Sin resultados** la fila vacía dice «Resultados» (`generic-labels.results`, `.html:129`), no «Sin resultados». Bajo la tabla: «Exportar» y paginador.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`), icono `upload` | Encabezado de la tarjeta de filtros | `*appPermiso="'Crear'"` (= `Seguridad:Crear`, `.html:19`) | Nunca | Navega a `/roles/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Ídem, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/roles/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`) | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/roles` |
| Exportar (`actions.export`) | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/roles/exportar`, descarga `roles.xlsx` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario:** falla la búsqueda o la exportación → «Error al buscar roles» (`roles.search.search-error`, `.ts:104,153`). Sin mapa de códigos.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Buscar / paginar | `RolService.search` (`app/pages/roles/services/rol.service.ts:12-21`) | `GET /api/v1/roles?pageNumber&pageSize[&nombreRol][&descripcion][&nivelJerarquiaDesde][&nivelJerarquiaHasta]` | `Seguridad:Leer` (`RolesController.cs:27`) |
| Exportar | `RolService.exportar` (`:39-41`) | `POST /api/v1/roles/exportar` | `Seguridad:Leer` (`:161`) |

**Exportación:** cuerpo `{ headers, nombreRol?, descripcion?, nivelJerarquiaDesde?, nivelJerarquiaHasta? }` (filtros actuales; mismas reglas de 0/100). `headers` (3, en orden): «Nombre del Rol», «Descripción», «Nivel de Jerarquía» (`roles.table.*`). Archivo «roles.xlsx» (`roles.search.export-filename`). Carga masiva: §4.3.

**Comportamientos particulares:** `search()` no reinicia la página (§2.1).

**Observaciones:** la fila de «sin resultados» muestra «Resultados» (igual que Empresa, §1.1).

### 4.2 `/roles/nueva` y `/roles/:id` — Roles (detalle y configuración)

- **Componente:** `RolesDetailComponent` (`app/pages/roles/roles-detail/roles-detail.component.ts`, `.html`). Scope `roles`. Misma clase para las dos rutas. Es la única pantalla de la aplicación que edita la matriz de permisos.
- **Para qué sirve:** crear un rol; ver o editar su descripción y nivel; ver la vista previa del menú que obtendrá, marcar los permisos por módulo y acción, elegir sus sucursales y eliminarlo.
- **Cómo se llega:** «Nuevo» (crear) o el enlace del nombre en la búsqueda (ver/editar).
- **Permiso de ruta:** `/roles/nueva` → `Seguridad:Leer` + `Seguridad:Crear`; `/roles/:id` → `Seguridad:Leer`.
- **Migas y cabecera:** título «Roles»; migas «Inicio» / «Administración» / «Roles & Permisos» (con enlace a `/roles`); botón volver a `/roles`.
- **Modos** (`ngOnInit`, `.ts:101-113`): sin `id` → **crear**; con `id` → **ver** (formulario deshabilitado; carga rol, permisos y sucursales) y «Editar» → **editar**.

| Aspecto | Crear | Ver | Editar |
|---|---|---|---|
| Encabezado «Roles» + «Datos del registro» + «Editar» | No | Sí (botón solo con `Seguridad:Actualizar`) | No |
| Nombre del Rol | Editable | Deshabilitado | Deshabilitado (no se puede cambiar) |
| Nivel de Jerarquía y Descripción | Editables | Deshabilitados | Editables (`enterEditMode`, `.ts:241-245`) |
| Tarjeta «Configuración del Rol» (Menú, Permisos, Sucursales) | **No se muestra** (`@if (!isNew())`, `.html:71`) | Sí, solo lectura | Sí; casillas editables solo con `Seguridad:GestionarPermisos` |
| Botones | Cancelar, Guardar (tarjeta aparte, `.html:317-336`) | Editar | Cancelar, Guardar, Eliminar (al pie de la tarjeta de configuración, `.html:289-315`) |

**Campos del formulario** (`.ts:89-93`, `.html:37-68`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Nombre del Rol (`roles.fields.nombreRol`) | `nombreRol` | `matInput`, `maxlength="100"` | Sí | `required`, `maxLength(100)` (`.ts:90`) | «Este campo es obligatorio» (`general.errors.required`) | Sí / No |
| Nivel de Jerarquía (`roles.fields.nivelJerarquia`) | `nivelJerarquia` | `matInput type="number" min="1"` | Sí | `required`, `min(1)` (`.ts:92`); sin máximo | «Este campo es obligatorio» (`general.errors.required`) para cualquiera de las dos validaciones | Sí / Sí. Valor inicial 0, que es inválido: hay que indicar un entero ≥ 1 |
| Descripción (`roles.fields.descripcion`) | `descripcion` | `textarea` de 3 filas, `maxlength="255"` | No | `maxLength(255)` (`.ts:91`) | — | Sí / Sí |

La clave `roles.detail.nombreRolHint` («El nombre del rol es un identificador del sistema y no puede modificarse.») existe pero la plantilla no la usa: el usuario no ve ninguna explicación de por qué el nombre está bloqueado.

**Tarjeta «Configuración del Rol»** (solo `/roles/:id`, `.html:71-316`): título «Configuración del Rol» (`roles.configuracion.title`), subtítulo «Define el menú, permisos y sucursales para este rol.» (`roles.configuracion.subtitle`) y, a la derecha, un aviso amarillo con icono `info`: «Los cambios se aplican de inmediato. Los usuarios conectados verán su menú actualizado al recargar la página.» (`roles.permisos.cache-warning`). Tres pestañas (`mat-tab-group`):

1. **«Menú»** (`roles.configuracion.tab-menu`): vista previa de solo lectura del menú lateral real. Texto: «Vista previa del menú real: una pantalla aparece cuando el rol tiene el permiso «Leer» de su módulo.» (`roles.configuracion.menu-preview-hint`). Se construye con la misma lista del menú (`navItems`, `sidebar-data.ts`) y una tarjeta por grupo del menú, con el título de la sección (por ejemplo «Administración») y cada pantalla con icono `check_circle` (visible) o `radio_button_unchecked` (oculta) (`menu-preview.ts:15-25`, `.html:104-188`). Una pantalla se marca visible si no tiene módulo asignado (Dashboard) o si los permisos **marcados en ese momento** del rol incluyen `Leer` de su módulo (`.ts:72-79`); por eso la vista previa cambia al marcar casillas en la pestaña Permisos sin guardar. Mientras se cargan los permisos muestra un spinner.
2. **«Permisos»** (`roles.configuracion.tab-permisos`): matriz en un acordeón de paneles (`mat-accordion multi`, varios abiertos a la vez), uno por módulo.
   - **Catálogo:** viene de la API (`GET /api/v1/roles/permissions/catalog`, `Seguridad:Leer`), que lo arma con los pares módulo/acción declarados en los controladores (RN-ROL-016). Los permisos que tiene el rol vienen de `GET /api/v1/roles/{id}/permissions`. Se cargan en paralelo (`forkJoin`, `.ts:155-172`).
   - **Textos:** el nombre de cada módulo y acción se traduce con `roles.permisos.modulo.<Módulo>` y `roles.permisos.accion.<Acción>` (`.ts:182-186`); si no hay traducción se muestra el nombre técnico. Los 27 módulos y las 20 acciones distintas que usa hoy la API tienen traducción (verificado contra `docs/_inventario/permisos.csv`). Tabla de textos abajo.
   - **Orden:** módulos y acciones se ordenan alfabéticamente por el texto que ve el usuario, no por el nombre técnico (`.ts:82-86`).
   - **Selección:** cada panel tiene una casilla de módulo en el encabezado (marca o desmarca todas las acciones; se ve «indeterminada» si hay algunas) y a la derecha el contador «N/M acciones» (`roles.permisos.acciones` = «acciones»). Dentro, una casilla por acción en 4 columnas (2 en móvil). No hay casilla global «seleccionar todo» para todos los módulos.
   - **Cuándo se pueden marcar:** solo en modo editar **y** con `Seguridad:GestionarPermisos` (`[disabled]="!isEditMode() || !puedeGestionarPermisos()"`, `.html:217,242`). Sin ese permiso las casillas quedan deshabilitadas y «Guardar» no envía permisos ni sucursales (`.ts:287-288`).
3. **«Sucursales»** (`roles.configuracion.tab-sucursales`): lista de casillas con el nombre de **todas** las sucursales de la empresa, marcadas las asignadas al rol (`GET /api/v1/roles/{id}/sucursales`, `Seguridad:Leer`; RN-ROL-018). Mismo criterio de habilitación que los permisos. Si no hay sucursales: «No hay sucursales disponibles» (`roles.configuracion.sucursales-no-data`). Estas sucursales son las que podrán usar las cuentas que tengan el rol (ver §5.2).

**Textos de módulos** (`roles.permisos.modulo.*`): Auditoria → «Auditoría de Seguridad»; Delivery → «Despachos Delivery»; Devoluciones → «Devoluciones»; DIGEMID → «DIGEMID»; Empleados → «Empleados»; Empresas → «Empresa»; FormulacionesClinicas → «Formulaciones Clínicas»; IngredientesActivos → «Ingredientes Activos»; Inventarios → «Inventario»; JerarquiasUoM → «Jerarquías de Unidades»; Laboratorios → «Laboratorios»; Magistrales → «Órdenes Magistrales»; Medicamentos → «Medicamentos»; Pacientes → «Pacientes»; POS → «Sesiones de Caja»; PresentacionesVenta → «Presentaciones de Venta»; ProfesionalesSalud → «Profesionales de Salud»; Proveedores → «Proveedores»; Recetas → «Recetas Médicas»; Seguridad → «Roles y Cuentas de Usuario»; Seguros → «Aseguradoras»; SeriesFiscales → «Series Fiscales»; Servicios → «Servicios»; Sucursales → «Sucursales»; Transferencias → «Transferencias»; UnidadesMedida → «Unidades de Medida»; Ventas → «Ventas».

**Textos de acciones** (`roles.permisos.accion.*`): Leer → «Leer»; Crear → «Crear»; Actualizar → «Actualizar»; Eliminar → «Eliminar»; AgendarServicio → «Agendar servicio»; AislarLoteCuarentena → «Aislar lote en cuarentena»; Analizar → «Analizar con IA»; AnonimizarDatos → «Anonimizar datos»; Anular → «Anular»; AperturarCaja → «Aperturar caja»; AsignarRoles → «Asignar roles»; CerrarCaja → «Cerrar caja»; CompletarOrdenMagistral → «Completar orden magistral»; GenerarActaDestruccion → «Generar acta de destrucción»; GestionarPermisos → «Gestionar permisos»; IniciarOrdenMagistral → «Iniciar orden magistral»; ProcesarDevolucion → «Procesar devolución»; ProgramarDelivery → «Programar delivery»; RegistrarInmunizacion → «Registrar inmunización»; RegistrarPrecioProveedor → «Registrar precio de proveedor».

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la primera tarjeta | Modo ver y `*appPermiso="'Actualizar'"` (= `Seguridad:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Casillas de módulo y de acción | Pestaña Permisos | Siempre (con `/roles/:id`) | `!isEditMode()` o sin `Seguridad:GestionarPermisos` | `toggleModulo` / `togglePermiso`: solo cambian el estado en pantalla (`permisosActivos`); nada se envía hasta «Guardar» |
| Casillas de sucursal | Pestaña Sucursales | Siempre | Ídem | `toggleSucursal`: cambia el estado en pantalla |
| Cancelar (`actions.cancel`) | Pie | Crear o editar | Nunca | `cancelEdit()`: en crear navega a `/roles`; en editar restaura descripción, nivel, sucursales y permisos a lo leído (`.ts:247-261`) |
| Guardar (`actions.save`) | Pie | Crear o editar | `saving()` | `save()` (`.ts:263-308`): si el formulario es inválido marca todo y no envía. **Crear:** `POST /api/v1/roles` con `{ nombreRol, descripcion?, nivelJerarquia }`, aviso y navega a `/roles`. **Editar:** en secuencia (a) `PUT /api/v1/roles/{id}` con `{ descripcion, nivelJerarquia }`; (b) solo con `GestionarPermisos`: `PUT /api/v1/roles/{id}/sucursales` con `{ sucursalIds }` (siempre, aunque no hayas tocado nada; una lista vacía quita todas); (c) solo con `GestionarPermisos`: calcula la diferencia contra lo leído y envía en paralelo un `POST /api/v1/roles/permissions` por cada permiso agregado y un `DELETE /api/v1/roles/permissions/{permisoId}` por cada permiso quitado (si no hay diferencias no llama nada). Al terminar: aviso, vuelve a modo ver y recarga rol, sucursales y permisos |
| Eliminar (`actions.delete`) | Pie, tras Guardar | Solo editar de un rol existente y `*appPermiso="'Eliminar'"` (= `Seguridad:Eliminar`, `.html:307`) | `deleting()` | `confirmDelete()` |
| Botón volver | Cabecera | Siempre | — | `/roles` |

Los pasos de guardado no son atómicos: si falla un paso intermedio, los anteriores ya quedaron guardados y la pantalla muestra solo el error (ver mensajes).

**Diálogos que abre:** `ConfirmDialogComponent` (400 px) al pulsar «Eliminar»: título «Confirmar eliminación» (`roles.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este rol?» (`roles.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/roles/{id}`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Rol guardado correctamente» (`roles.detail.save-success`) | `.ts:276,291` |
| Editar y la API responde `Rol.PermisosPropios` (no ampliar los permisos de un rol que tú tienes asignado) | Error | «No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado» (`roles.detail.permisos-propios-error`) | `.ts:301-304` |
| Cualquier otro error al editar, y **cualquier** error al crear (por ejemplo nombre duplicado `Rol.DuplicateName`, permiso duplicado) | Error | «Error al guardar el rol» (`roles.detail.save-error`) | `.ts:281,303` |
| Eliminación correcta | Éxito | «Rol eliminado correctamente» (`roles.detail.delete-success`); navega a `/roles` | `.ts:327` |
| Error al eliminar (cualquiera, incluido `Rol.InUse` con cuentas vinculadas, RN-ROL-008) | Error | «Error al eliminar el rol» (`roles.detail.delete-error`) | `.ts:332` |
| Error al cargar el rol | Error | «Error al cargar el rol» (`roles.detail.load-error`) | `.ts:130` |
| Error al cargar permisos | Error (con referencia de seguimiento si la respuesta trae `traceId`) | «Error al cargar los permisos» (`roles.permisos.load-error`) | `.ts:169` |
| Error al cargar sucursales | Error (ídem) | «Error al cargar las sucursales» (`roles.configuracion.sucursales-load-error`) | `.ts:145` |

No usa `saveErrorKey`: los códigos `Db.*` no se traducen aquí.

**Llamadas a la API**

| Cuándo | Método (`app/pages/roles/services/rol.service.ts`) | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir | `getById` | `GET /api/v1/roles/{id}` | `Seguridad:Leer` (`RolesController.cs:49`) |
| Abrir | `getPermissionsCatalog` | `GET /api/v1/roles/permissions/catalog` | `Seguridad:Leer` (`:103`) |
| Abrir | `getPermisos` | `GET /api/v1/roles/{id}/permissions` | `Seguridad:Leer` (`:119`) |
| Abrir | `getSucursales` | `GET /api/v1/roles/{id}/sucursales` | `Seguridad:Leer` (`:145`) |
| Crear | `create` | `POST /api/v1/roles` | `Seguridad:Crear` (`:57`) |
| Editar (datos) | `update` | `PUT /api/v1/roles/{id}` | `Seguridad:Actualizar` (`:67`) |
| Editar (sucursales) | `setSucursales` | `PUT /api/v1/roles/{id}/sucursales` | `Seguridad:GestionarPermisos` (`:153`) |
| Editar (permiso agregado) | `assignPermiso` | `POST /api/v1/roles/permissions` con `{ rolId, moduloSistema, accion }` | `Seguridad:GestionarPermisos` (`:127`) |
| Editar (permiso quitado) | `revokePermiso` | `DELETE /api/v1/roles/permissions/{permisoId}` | `Seguridad:GestionarPermisos` (`:137`) |
| Eliminar | `delete` | `DELETE /api/v1/roles/{id}` | `Seguridad:Eliminar` (`:75`) |

**Comportamientos particulares**
- La vista previa del menú y las casillas usan el estado en pantalla; solo «Guardar» persiste.
- El aviso «Los cambios se aplican de inmediato» se refiere a la caché de permisos del servidor (RN-ROL-015); en la pantalla los cambios se aplican al pulsar «Guardar».
- Existen claves sin uso para botones separados de guardado de permisos y sucursales (`roles.permisos.save`, `roles.configuracion.sucursales-save`, etc.): la pantalla guarda todo con un único botón (ver §9).

**Observaciones**
- Discrepancias con la API (RN-ROL-002 y RN-ROL-004 en 01a): el formulario admite hasta 100 caracteres en el nombre y la API 50; el formulario exige nivel mínimo 1 y la API acepta 0 (la carga masiva de roles: la vista previa exige 1 y el guardado acepta 0).
- Para el nivel, tanto «obligatorio» como «mínimo 1» muestran «Este campo es obligatorio».
- El SPA no impide editar o eliminar el rol `Admin`; cualquier restricción es de la API (no verificada aquí).

### 4.3 `/roles/carga-masiva` — Carga Masiva (roles)

- **Componente:** `RolesCargaMasivaComponent` (`app/pages/roles/roles-carga-masiva/roles-carga-masiva.component.ts`, plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `roles`; los textos los resuelve el componente común contra la raíz `carga-masiva.*`.
- **Para qué sirve:** cargar varios roles desde Excel. Flujo común en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/roles`.
- **Permiso de ruta:** `Seguridad:Leer` + `Seguridad:Crear`.
- **Migas y cabecera:** título «Carga Masiva» (`carga-masiva.title`); migas «Inicio» / «Administración» / «Roles & Permisos» (enlace a `/roles`) / «Carga Masiva» (activa); botón volver a `/roles` (`.ts:23-28`).

**Configuración que pasa** (`CargaMasivaConfig`, `.ts:18-34`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/roles/plantilla` — `Seguridad:Crear` (`RolesController.cs:194`) |
| `previewUrl` | `POST {api}/api/v1/roles/previsualizar` — `Seguridad:Crear` (`:203`) |
| `saveUrl` | `POST {api}/api/v1/roles/carga-masiva` — `Seguridad:Crear` (`:221`) |
| `backRoute` | `/roles` (archivo de plantilla `plantilla-roles.xlsx`) |
| Columnas de la vista previa | «Nombre del Rol» (`fields.nombreRol`, clave `nombreRol`, obligatoria), «Descripción» (`fields.descripcion`, clave `descripcion`, no obligatoria), «Nivel de Jerarquía» (`fields.nivelJerarquia`, clave `nivelJerarquia`, obligatoria) |

Las etiquetas salen de la **raíz** `fields.*` (no de `roles.fields.*`). Códigos de fila posibles (RN-ROL-022 en 01a): `required`, `max-length`, `duplicate`, `duplicate-in-file`, `invalid-integer`, `min-value`, con los textos de `carga-masiva.validation.*`. La vista previa exige nivel ≥ 1 y el guardado acepta ≥ 0 (RN-ROL-022). Botones, mensajes y errores: los del componente común.

---

## 5. Cuentas de usuario

Carpeta `app/pages/seguridad` (el menú y la URL dicen «cuentas»), scope i18n **`cuentas`** (`provideTranslocoScope('cuentas')` y `scope: 'cuentas'` en las dos plantillas; el scope `seguridad` de `public/i18n/seguridad/es.json` no lo usa ninguna pantalla de esta carpeta, ver §9). Componentes enrutados: `CuentasSearchComponent` y `CuentasDetailComponent` (rutas `nueva` y `:id`) (`app/pages/seguridad/cuentas.routes.ts`). El módulo de permisos es `Seguridad` (`route-permissions.ts:7`), el mismo de los roles. No hay exportación ni carga masiva de cuentas.

Resumen de lo que se puede hacer sobre una cuenta (comprobado en las fichas):
- **Crear:** se elige el empleado, se escribe el nombre de usuario y se define la contraseña inicial (la escribe el administrador; no se genera y no vuelve a mostrarse). La API crea la cuenta obligada a cambiar la clave en su primer ingreso (RN-CTA-008 en 01a); el formulario no lo anuncia.
- **Editar:** activar o desactivar, forzar el cambio de contraseña en el próximo ingreso y reiniciar los intentos fallidos (que levanta el bloqueo). El nombre de usuario y el empleado no se pueden cambiar.
- **Roles:** la pantalla trabaja con **un solo rol** por cuenta (muestra y reemplaza el primero de la lista).
- **Sucursales:** además de la sucursal base de su empleado, se agregan y quitan sucursales adicionales; cada cambio se aplica al instante, sin «Guardar».
- **Bloqueo / desbloqueo:** el bloqueo es automático (por intentos fallidos); la pantalla solo lo muestra y permite quitarlo con «Resetear Intentos Fallidos». No existe una acción «Bloquear».
- **Restablecer contraseña:** **no existe** esa acción en la pantalla ni endpoint en `/api/v1/cuentas` (ver endpoints en `docs/_inventario/endpoints.csv`). La única palanca es «Forzar Cambio de Contraseña».
- **Restricciones:** una cuenta no puede cambiar sus propios roles (`Rol.AutoAsignacion`); solo un Admin puede asignar o quitar el rol Admin (`Rol.AdminReservado`) y solo un Admin puede editar o eliminar la cuenta de un administrador (`Cuenta.AdminProtegida`). Las tres tienen mensaje propio (§5.2). El SPA no deshabilita nada por ser la cuenta propia o de un administrador: es la API quien rechaza. No se encontró en los handlers de editar y eliminar ningún bloqueo para que alguien se desactive o elimine su propia cuenta (`UpdateCuentaCommandHandler.cs`, `DeleteCuentaCommandHandler.cs`).

### 5.1 `/cuentas` — Cuentas de Usuario (búsqueda)

- **Componente:** `CuentasSearchComponent` (`app/pages/seguridad/cuentas-search/cuentas-search.component.ts`, `.html`). Scope `cuentas`.
- **Para qué sirve:** buscar cuentas por usuario, empleado, estado y bloqueo, y abrirlas.
- **Cómo se llega:** menú lateral, grupo «Administración» → «Cuentas de Usuario» (`nav.cuentasUsuario`, `sidebar-data.ts:41-45`); volver desde el detalle.
- **Permiso de ruta:** `Seguridad:Leer`.
- **Migas y cabecera:** título «Cuentas de Usuario» (`cuentas.title`); migas «Inicio» / «Administración» / «Cuentas de Usuario» (`breadcrumbs.cuentas`, activa); sin botón volver. Tarjeta «Búsqueda de Cuentas» (`cuentas.search.title`) con subtítulo común.
- **Modos:** uno. Resultados tras «Buscar» o con estado guardado (§0).

**Filtros de búsqueda** (`.ts:45-50`, `.html:27-77`)

| Etiqueta (clave) | Control | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Nombre de Usuario (`cuentas.fields.nombreUsuario`) | `nombreUsuario` | Texto con lupa, Enter busca | Libre | Vacío |
| Empleado (`cuentas.fields.empleado`) | `nombreEmpleado` | Texto con lupa, Enter busca | Libre | Vacío |
| Cuenta Activa (`cuentas.fields.cuentaActiva`) | `cuentaActiva` | Select | «—» (sin filtro), «Activa» (`cuentas.status.active`), «Inactiva» (`cuentas.status.inactive`) | Nulo (sin filtro) |
| Bloqueado (`cuentas.fields.bloqueado`) | `bloqueado` | Select | «—», «Sí» (`generic-labels.yes`), «No» (`generic-labels.no`) | Nulo |

«Limpiar» vacía los cuatro filtros, oculta la tabla, vuelve a la página 1 y borra el estado guardado.

**Tabla de resultados** (`.html:97-194`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Usuario (`cuentas.table.nombreUsuario`) | `nombreUsuario` | Texto | Sí → `/cuentas/:id` |
| Empleado (`cuentas.table.empleado`) | `nombreEmpleado` | Texto | No |
| Estado (`cuentas.table.cuentaActiva`) | `cuentaActiva` | Solo icono: `check_circle` (activa) o `cancel` (inactiva) | No |
| Bloqueado (`cuentas.table.bloqueado`) | `bloqueadoHasta` | Solo icono: `lock` si la fecha de bloqueo es posterior a la hora actual del navegador (`isBloqueado`, `.ts:123-126`), `lock_open` si no | No |
| Intentos (`cuentas.table.intentosFallidos`) | `intentosFallidos` | Número | No |

Encabezado «Resultados (N)». Paginación 10/25/50. **Sin resultados** la fila vacía dice «Resultados» (`generic-labels.results`, `.html:178`). No hay botón «Exportar»; el paginador va solo, a la derecha.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros | `*appPermiso="'Crear'"` (= `Seguridad:Crear`, `.html:18`) | Nunca | `goToNew()` → `/cuentas/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`) | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/cuentas` |

**Diálogos que abre:** ninguno. **Mensajes al usuario:** falla la búsqueda → «Error al buscar cuentas» (`cuentas.search.search-error`, `.ts:99`).

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Buscar / paginar | `CuentaService.search` (`app/pages/seguridad/services/cuenta.service.ts:15-24`) | `GET /api/v1/cuentas?pageNumber&pageSize[&nombreUsuario][&nombreEmpleado][&cuentaActiva][&bloqueado]` (los booleanos se envían también si son `false`) | `Seguridad:Leer` (`CuentasController.cs:19`) |

**Exportación y carga masiva:** no hay (la clave `cuentas.search.export-filename` no se usa).

**Comportamientos particulares:** `search()` no reinicia la página (§2.1).

### 5.2 `/cuentas/nueva` y `/cuentas/:id` — Cuentas de Usuario (detalle)

- **Componente:** `CuentasDetailComponent` (`app/pages/seguridad/cuentas-detail/cuentas-detail.component.ts`, `.html`). Scope `cuentas`. Misma clase para las dos rutas.
- **Para qué sirve:** crear una cuenta de acceso para un empleado; ver sus datos; activarla o desactivarla, forzar el cambio de contraseña, quitar el bloqueo, cambiar su rol y sus sucursales adicionales; eliminarla.
- **Cómo se llega:** «Nuevo» (crear) o el enlace del usuario en la búsqueda; tras crear, el SPA abre `/cuentas/:id` de la cuenta recién creada.
- **Permiso de ruta:** `/cuentas/nueva` → `Seguridad:Leer` + `Seguridad:Crear`; `/cuentas/:id` → `Seguridad:Leer`.
- **Migas y cabecera:** título «Cuentas de Usuario»; migas «Inicio» / «Administración» / «Cuentas de Usuario» (con enlace a `/cuentas`); botón volver a `/cuentas`.
- **Modos** (`ngOnInit`, `.ts:115-124`): sin `id` → **crear**; con `id` → **ver** (carga la cuenta), «Editar» → **editar**. A diferencia de las otras fichas, aquí hay dos formularios distintos (`createForm` y `editForm`) y tres tarjetas.

| Aspecto | Crear | Ver | Editar |
|---|---|---|---|
| Tarjeta de datos | Formulario de alta (§ campos de creación) | Encabezado «Cuentas de Usuario» + «Datos del registro» + «Editar» (solo con `Seguridad:Actualizar`) y 9 datos de solo lectura | Usuario y Empleado deshabilitados + 2 interruptores + 1 casilla + Cancelar / Guardar / Eliminar |
| Tarjeta «Roles Asignados» | No se muestra | Rol actual en texto deshabilitado | Buscador de rol (solo con `Seguridad:AsignarRoles`; sin ese permiso sigue siendo texto) |
| Tarjeta «Sucursales Asignadas» | No se muestra | Lista sin botones de quitar | Lista con botón de quitar y buscador para agregar |

**Campos de creación** (`createForm`, `.ts:98-101`, `.html:16-89`)

| Etiqueta (clave) | Control | Obligatorio | Validaciones | Mensaje de error | Ayuda |
|---|---|---|---|---|---|
| Empleado (`cuentas.fields.empleado`, como placeholder) | `csw-autocomplete-input`: mínimo 2 letras, vacía el texto si no coincide con una opción | Sí | Debe elegirse un empleado de la lista (`empleadoId`); al pulsar «Guardar» sin elegir, `empleadoError` | «Este campo es obligatorio» (`general.errors.required`) bajo el campo (`.html:28-33`) | Busca con `GET /api/v1/empleados?searchTerm=…` (hasta 20, muestra `nombre_Completo`); el SPA no filtra los empleados que ya tienen cuenta |
| Nombre de Usuario (`cuentas.fields.nombreUsuario`) | `nombreUsuario`, `matInput` sin atributo `maxlength` | Sí | `required`, `maxLength(50)` (`.ts:99`) | Vacío y tocado: «Este campo es obligatorio» (`general.errors.required`). Más de 50: «La longitud máxima es de 100 caracteres» (`general.errors.maxLength` con `max: 100`, `.html:47-51`) | — |
| Contraseña (`cuentas.fields.password`) | `password`, `matInput type="password"` (sin botón para mostrarla) | Sí | `required`, `minLength(8)`, `maxLength(128)` (`.ts:100`) | Solo existe mensaje para vacío y tocado: «Este campo es obligatorio» (`.html:59-66`). Con 1 a 7 caracteres o más de 128 el formulario es inválido, «Guardar» no envía y **no se muestra ningún texto** | — |

La contraseña inicial **la define quien crea la cuenta**; no se genera, no se muestra después y el formulario no avisa de que el usuario tendrá que cambiarla al ingresar (lo impone la API, RN-CTA-008).

**Datos en modo ver** (`.html:116-203`, todos `matInput` deshabilitados)

| Etiqueta (clave) | Dato | Formato |
|---|---|---|
| Nombre de Usuario (`cuentas.fields.nombreUsuario`) | `nombreUsuario` | Texto |
| Empleado (`cuentas.fields.empleado`) | `nombreEmpleado` | Texto |
| Intentos Fallidos (`cuentas.fields.intentosFallidos`) | `intentosFallidos` | Número |
| Cuenta Activa (`cuentas.fields.cuentaActiva`) | `cuentaActiva` | «Activa» o «Inactiva» (`cuentas.status.*`) |
| Bloqueado (`cuentas.fields.bloqueado`) | calculado | «Sí» si `bloqueadoHasta` es posterior a ahora, si no «No» (`generic-labels.yes/no`; `isBloqueado()`, `.ts:257-261`) |
| Requiere Cambio de Clave (`cuentas.fields.requiresCambio`) | `requiereCambioClave` | «Sí» / «No» |
| Bloqueado Hasta (`cuentas.fields.bloqueadoHasta`) | `bloqueadoHasta` | `dd/MM/yyyy HH:mm`; solo se muestra si la cuenta tiene fecha de bloqueo (aunque ya haya pasado) |
| Creado el (`cuentas.fields.createdAt`) | `createdAt` | `dd/MM/yyyy` |

**Campos en modo editar** (`editForm`, `.ts:103-107`, `.html:207-281`)

| Etiqueta (clave) | `formControlName` | Control | Qué hace al guardar |
|---|---|---|---|
| Cuenta Activa (`cuentas.fields.cuentaActiva`) | `cuentaActiva` | Interruptor (`mat-slide-toggle`, color primario); inicia con el valor actual | Activa o desactiva la cuenta. Una cuenta desactivada no puede iniciar sesión ni renovar la sesión (RN-CTA-012) |
| Forzar Cambio de Contraseña (`cuentas.fields.forzarCambioClave`) | `forzarCambioClave` | Interruptor (color de advertencia); inicia siempre apagado | Si queda encendido, el usuario tendrá que cambiar la clave en su próximo ingreso. Solo se puede encender: el SPA siempre reenvía el valor y la API lo apaga cuando el usuario cambia la clave (RN-CTA-011) |
| Resetear Intentos Fallidos (`cuentas.fields.resetearIntentos`) | `resetearIntentos` | Casilla; inicia siempre desmarcada | Pone los intentos fallidos en 0 y quita el bloqueo. Es la forma de **desbloquear** una cuenta |
| Nombre de Usuario / Empleado | — | Texto deshabilitado | No se pueden cambiar |

**Tarjeta «Roles Asignados»** (`cuentas.roles.title`, subtítulo «Gestiona los roles de acceso de esta cuenta», `.html:284-314`). En modo ver, o en editar sin `Seguridad:AsignarRoles` (`puedeAsignarRoles`, `.ts:59`), muestra un campo deshabilitado con el nombre del rol actual o «Sin roles asignados» (`cuentas.roles.empty`). En editar con ese permiso muestra un buscador (placeholder «Agregar rol...», `cuentas.roles.add-placeholder`; mínimo 0 letras, así que lista roles sin escribir; consulta `GET /api/v1/roles?nombreRol=…&pageSize=20`, `Seguridad:Leer`) con el rol actual como texto inicial. La pantalla maneja **un solo rol**: `currentRol` es el primero de la lista (`.ts:77`); si eliges otro, «Guardar» quita el actual y asigna el nuevo. Vaciar el buscador **no quita** el rol (sin rol elegido no se hace nada, `.ts:198-200`): desde esta pantalla no se puede dejar una cuenta sin rol.

**Tarjeta «Sucursales Asignadas»** (`cuentas.sucursales.title`, subtítulo «Sucursales adicionales donde esta cuenta puede operar, además de la sucursal base de su empleado», `.html:317-358`). Lista el nombre de cada sucursal asignada; vacía: «Sin sucursales adicionales asignadas» (`cuentas.sucursales.empty`). En modo editar cada fila tiene un botón con icono `delete` (tooltip y etiqueta «Quitar sucursal», `cuentas.sucursales.remove-tooltip`) y bajo la lista un buscador (placeholder «Agregar sucursal...», `cuentas.sucursales.add-placeholder`; mínimo 0 letras; consulta `GET /api/v1/sucursales?nombre=…&pageSize=20`, `Sucursales:Leer`, y oculta las ya asignadas). **Agregar o quitar se aplica de inmediato** (`POST` / `DELETE` y recarga), sin pulsar «Guardar» y sin confirmación.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta de datos | Modo ver y `*appPermiso="'Actualizar'"` (= `Seguridad:Actualizar`, `.html:107`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie de la tarjeta de datos | Crear o editar | Nunca | `cancelEdit()`: en crear navega a `/cuentas`; en editar descarta el rol elegido y restaura los tres controles (`.ts:154-169`) |
| Guardar (`actions.save`) | Ídem | Crear o editar | `saving()` | `save()` (`.ts:171-220`). **Crear:** marca todo; si falta empleado o el formulario es inválido no envía; `POST /api/v1/cuentas` con `{ empleadoId, nombreUsuario, password }`; aviso y navega a `/cuentas/:id`. **Editar:** `PUT /api/v1/cuentas/:id` con `{ cuentaActiva, forzarCambioClave, resetearIntentos }`; luego, si elegiste un rol distinto, `DELETE …/roles/{rolActual}` seguido de `POST …/roles/{rolNuevo}`; al final aviso, vuelve a modo ver y recarga |
| Eliminar (`actions.delete`; spinner mientras elimina) | Pie, tras Guardar | Solo editar y `*appPermiso="'Eliminar'"` (= `Seguridad:Eliminar`, `.html:268`) | `deleting()` | `confirmDelete()` |
| Quitar sucursal (icono `delete`) | Fila de sucursal | Solo editar | Mientras se quita esa fila | `removeSucursal(id)` → `DELETE /api/v1/cuentas/{id}/sucursales/{sucursalId}` |
| Botón volver | Cabecera | Siempre | — | `/cuentas` |

**Diálogos que abre:** `ConfirmDialogComponent` (ancho por defecto) al pulsar «Eliminar»: título «Eliminar cuenta» (`cuentas.detail.confirm-delete-title`), mensaje «¿Estás seguro de que deseas eliminar esta cuenta de usuario? Esta acción no se puede deshacer.» (`cuentas.detail.confirm-delete-message`). Los textos de botón que pasa el componente (`confirmText`/`cancelText`, `.ts:233-234`) no coinciden con los campos que lee el diálogo (`confirmButtonText`/`cancelButtonText`, `confirm-dialog.component.ts:12-13`), por lo que el usuario ve los botones por defecto «Confirmar» y «Cancelar», no «Eliminar» y «Cancelar» de `cuentas.detail.confirm-delete-confirm` / `confirm-delete-cancel`. Confirmar → `DELETE /api/v1/cuentas/{id}`.

**Mensajes al usuario.** Los errores usan `showServerError`: si la respuesta trae `traceId` se muestra el mensaje con la referencia de seguimiento ([19a §6.11](19a-acceso-layout-y-compartidos.md)); si no, un aviso de error simple. En creación **no** se traducen los códigos de la API; en editar, eliminar y sucursales se traducen los cinco códigos de `ERROR_KEYS` (`.ts:23-29`).

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Cuenta guardada correctamente» (`cuentas.detail.save-success`) | `.ts:181,210` |
| Error al crear (cualquiera, incluido `Auth.DuplicateAccount` y `Auth.DuplicateUsername`) | Error | «Error al guardar la cuenta» (`cuentas.detail.save-error`) | `.ts:186` |
| Error al editar o al cambiar el rol, código no mapeado | Error | «Error al guardar la cuenta» (`cuentas.detail.save-error`) | `.ts:216` |
| Código `Rol.AutoAsignacion` | Error | «Una cuenta no puede modificar sus propios roles» (`cuentas.errors.auto-asignacion`) | `.ts:24` |
| Código `Rol.AdminReservado` | Error | «Solo un administrador puede asignar o quitar el rol Admin» (`cuentas.errors.admin-reservado`) | `.ts:25` |
| Código `Cuenta.AdminProtegida` | Error | «Solo un administrador puede modificar o eliminar la cuenta de un administrador» (`cuentas.errors.admin-protegida`) | `.ts:26` |
| Código `Auth.CuentaNotFound` | Error | «La cuenta no existe» (`cuentas.errors.cuenta-not-found`) | `.ts:27` |
| Código `Sucursal.NotFound` | Error | «La sucursal no existe» (`cuentas.errors.sucursal-not-found`) | `.ts:28` |
| Eliminación correcta | Éxito | «Cuenta eliminada correctamente» (`cuentas.detail.delete-success`); navega a `/cuentas` | `.ts:247` |
| Error al eliminar, código no mapeado | Error | «Error al eliminar la cuenta» (`cuentas.detail.delete-error`) | `.ts:252` |
| Sucursal asignada | Éxito | «Sucursal asignada correctamente» (`cuentas.sucursales.assign-success`) | `.ts:274` |
| Error al asignar sucursal, código no mapeado | Error | «Error al asignar la sucursal» (`cuentas.sucursales.assign-error`) | `.ts:279` |
| Sucursal quitada | Éxito | «Sucursal eliminada correctamente» (`cuentas.sucursales.remove-success`) | `.ts:290` |
| Error al quitar sucursal, código no mapeado | Error | «Error al eliminar la sucursal» (`cuentas.sucursales.remove-error`) | `.ts:295` |
| Error al cargar la cuenta | Error | «Error al cargar la cuenta» (`cuentas.detail.load-error`) | `.ts:140` |

Un usuario sin rol Admin que edite o elimine la cuenta de un administrador recibirá `Cuenta.AdminProtegida` al pulsar «Guardar».

**Llamadas a la API** (`app/pages/seguridad/services/cuenta.service.ts`; permisos de `endpoints.csv`)

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir / tras cada cambio | `getById` | `GET /api/v1/cuentas/{id}` | `Seguridad:Leer` (`CuentasController.cs:41`) |
| Crear | `create` | `POST /api/v1/cuentas` | `Seguridad:Crear` (`:49`) |
| Editar datos | `update` | `PUT /api/v1/cuentas/{id}` | `Seguridad:Actualizar` (`:59`) |
| Eliminar | `delete` | `DELETE /api/v1/cuentas/{id}` | `Seguridad:Eliminar` (`:71`) |
| Cambiar rol (asignar) | `assignRol` | `POST /api/v1/cuentas/{id}/roles/{rolId}` | `Seguridad:AsignarRoles` (`:79`) |
| Cambiar rol (quitar) | `removeRol` | `DELETE /api/v1/cuentas/{id}/roles/{rolId}` | `Seguridad:AsignarRoles` (`:87`) |
| Agregar sucursal | `assignSucursal` | `POST /api/v1/cuentas/{id}/sucursales/{sucursalId}` | `Seguridad:Actualizar` (`:95`) |
| Quitar sucursal | `removeSucursal` | `DELETE /api/v1/cuentas/{id}/sucursales/{sucursalId}` | `Seguridad:Actualizar` (`:103`) |
| Buscar empleado | `searchEmpleados` | `GET /api/v1/empleados?pageNumber=1&pageSize=20&searchTerm=…` | `Empleados:Leer` |
| Buscar rol | `RolService.search` | `GET /api/v1/roles?nombreRol=…&pageNumber=1&pageSize=20` | `Seguridad:Leer` |
| Buscar sucursal | `searchSucursales` | `GET /api/v1/sucursales?nombre=…&pageNumber=1&pageSize=20` | `Sucursales:Leer` |

**Comportamientos particulares**
- Los cambios de sucursal y los de datos/rol son independientes: «Cancelar» no deshace sucursales ya agregadas o quitadas.
- Cambiar un rol invalida la sesión de esa cuenta (RN-CTA-018); el SPA no lo anuncia.
- La secuencia de cambio de rol (quitar y luego asignar) no es atómica: si falla la asignación, el rol anterior ya fue quitado.
- Cuando se crea la cuenta, no se pide rol ni sucursales: se asignan después, en la ficha.

**Observaciones**
- Discrepancias: el aviso de longitud máxima del usuario dice 100 y el validador es 50 (RN-CTA-003); la contraseña pide 8 a 128 pero solo avisa si está vacía (RN-CTA-004).
- La fila de «sin resultados» de la búsqueda muestra «Resultados» (§5.1).
- Claves sin uso: `cuentas.detail.title-new`, `title-edit`, `cuentas.fields.roles`, `fields.sucursales` y `cuentas.roles.assign-*` / `remove-*` (ver §9).

---

## 6. Series fiscales

Carpeta `app/pages/series-fiscales`. Componentes enrutados: `SeriesFiscalesSearchComponent` y `SeriesFiscalesDetailComponent` (rutas `nueva` y `:id`) (`app/pages/series-fiscales/series-fiscales.routes.ts`). El módulo de permisos es `SeriesFiscales` (`route-permissions.ts:25`). No hay carga masiva de series.

**Scope i18n y prefijo de claves.** Los componentes cargan el scope `series-fiscales` (`provideTranslocoScope('series-fiscales')`, archivo `public/i18n/series-fiscales/es.json`) y leen con `read: 'seriesFiscales'`; las claves que usa el código en TypeScript llevan el prefijo `seriesFiscales.` (por ejemplo `seriesFiscales.detail.save-error`). En este documento se escribe esa forma, que es la que aparece en el código (`series-fiscales-search.component.ts:82`, `series-fiscales-detail.component.ts:35-41`).

**Tipos de comprobante y prefijos permitidos** (comprobado en el modelo del SPA, `app/pages/series-fiscales/models/serie-fiscal.model.ts`):
- El SPA ofrece cuatro tipos (`TIPOS_CONFIGURABLES`, `:18-23`): «Boleta» (`seriesFiscales.tipos.Boleta`), «Factura» (`tipos.Factura`), «Nota de Crédito» (`tipos.NotaCredito`) y «Proforma» (`tipos.Proforma`). El enumerado también define `Ticket`, `NotaDebito` y valores sin etiqueta en el idioma (`:9-16`), que no se pueden elegir ni filtrar.
- Prefijo: exactamente 4 caracteres, solo mayúsculas A-Z y dígitos (`^[A-Z0-9]{4}$`). Boleta empieza con B, Factura con F, Nota de Crédito con B o F, Proforma sin restricción de letra (`esPrefijoValido`, `:69-81`). Texto de ayuda: «4 caracteres en mayúsculas. Boleta empieza con B, Factura con F, Nota de Crédito con B o F.» (`seriesFiscales.detail.prefijo-hint`).
- Correlativo: entero mayor o igual a 0; es el último número emitido y el siguiente comprobante usa ese valor + 1 (`seriesFiscales.detail.correlativo-hint`).
- Estado: «Activa» o «Inactiva» (`seriesFiscales.estados.*`).
- Sucursal: obligatoria; una serie pertenece a una sucursal.
- Reglas de unicidad y numeración de la API (no se validan en pantalla): RN-SER-007 y RN-SER-008 en 01a.

### 6.1 `/series-fiscales` — Series Fiscales (búsqueda)

- **Componente:** `SeriesFiscalesSearchComponent` (`app/pages/series-fiscales/series-fiscales-search/series-fiscales-search.component.ts`, `.html`). Scope `series-fiscales` (alias `seriesFiscales`).
- **Para qué sirve:** buscar series por sucursal, tipo, estado y prefijo, abrirlas, exportar y entrar a crear.
- **Cómo se llega:** menú lateral, grupo «SUNAT» → «Series Fiscales» (`nav.seriesFiscales`, `sidebar-data.ts:168-179`); volver desde el detalle.
- **Permiso de ruta:** `SeriesFiscales:Leer`.
- **Migas y cabecera:** título «Series Fiscales» (`seriesFiscales.title`); migas «Inicio» / «SUNAT» (`breadcrumbs.sunat`) / «Series Fiscales» (`breadcrumbs.seriesFiscales`, activa); sin botón volver (`.ts:67-71`). Tarjeta «Búsqueda de Series Fiscales» (`seriesFiscales.search.title`) con subtítulo común.
- **Modos:** uno. Resultados tras «Buscar» o con estado guardado (§0).
- **Al abrir:** carga hasta 1000 sucursales (`GET /api/v1/sucursales?pageNumber=1&pageSize=1000`) para el filtro de sucursal (`.ts:80-83`); si falla: «No se pudieron cargar las sucursales» (`seriesFiscales.search.sucursales-error`).

**Filtros de búsqueda** (`.ts:60-65`, `.html:30-82`)

| Etiqueta (clave) | Control | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Sucursal (`seriesFiscales.fields.sucursal`) | `sucursalId` | Select | «Todas» (`seriesFiscales.fields.todas`, sin filtro) y el nombre de cada sucursal cargada | Nulo (sin filtro) |
| Prefijo de Serie (`seriesFiscales.fields.prefijoSerie`) | `prefijoSerie` | Texto con lupa, `maxlength="4"`, Enter busca | Se recorta y se pasa a mayúsculas antes de enviar (`currentFilters`, `.ts:100`); la API lo trata como «contiene» (RN-SER-014) | Vacío |
| Tipo de Comprobante (`seriesFiscales.fields.tipoComprobante`) | `tipoComprobante` | Select | «Todos» (`seriesFiscales.fields.todos`), «Boleta», «Factura», «Nota de Crédito», «Proforma» | Nulo |
| Estado (`seriesFiscales.fields.estadoSerie`) | `estadoSerie` | Select | «Todos», «Activa», «Inactiva» | Nulo |

«Limpiar» vacía los filtros, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`.ts:131-138`).

**Tabla de resultados** (`.html:102-181`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Serie (`seriesFiscales.table.prefijoSerie`) | `prefijoSerie` | Texto | Sí → `/series-fiscales/:id` |
| Tipo de Comprobante (`seriesFiscales.table.tipoComprobante`) | `tipoComprobante` | Texto traducido (`seriesFiscales.tipos.<Nombre>`) | No |
| Sucursal (`seriesFiscales.table.sucursal`) | `sucursalNombre` | Texto | No |
| Correlativo Actual (`seriesFiscales.table.correlativoActual`) | `correlativoActual` | Número | No |
| Estado (`seriesFiscales.table.estadoSerie`) | `estadoSerie` | Texto «Activa» / «Inactiva» | No |

Encabezado «Resultados (N)». Paginación 10/25/50. Sin resultados: «Sin resultados» (`generic-labels.no-results`). Bajo la tabla: «Exportar» y paginador.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros | `*appPermiso="'Crear'"` (= `SeriesFiscales:Crear`, `.html:20`) | Nunca | `goToNew()` → `/series-fiscales/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`) | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/seriesfiscales` |
| Exportar (`actions.export`) | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/seriesfiscales/exportar`, descarga `series-fiscales.xlsx` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar series fiscales» (`seriesFiscales.search.search-error`) | `.ts:126` |
| Falla la exportación | Error | «Error al exportar las series fiscales» (`seriesFiscales.search.export-error`) | `.ts:181` |
| Fallan las sucursales del filtro | Error | «No se pudieron cargar las sucursales» (`seriesFiscales.search.sucursales-error`) | `.ts:82` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir | `SucursalService.search` | `GET /api/v1/sucursales?pageNumber=1&pageSize=1000` | `Sucursales:Leer` (`SucursalesController.cs:56`) |
| Buscar / paginar | `SerieFiscalService.search` (`app/pages/series-fiscales/services/serie-fiscal.service.ts:18-27`) | `GET /api/v1/seriesfiscales?pageNumber&pageSize[&sucursalId][&tipoComprobante][&estadoSerie][&prefijoSerie]` (el tipo viaja como número) | `SeriesFiscales:Leer` (`SeriesFiscalesController.cs:18`) |
| Exportar | `exportar` (`:45-47`) | `POST /api/v1/seriesfiscales/exportar` | `SeriesFiscales:Leer` (`:60`) |

**Exportación:** cuerpo `SerieFiscalExportRequest` (`serie-fiscal.model.ts:61-66`): `headers` (6, en orden): «Serie», «Tipo de Comprobante», «Sucursal», «Correlativo Actual», «Estado», «Fecha de Creación» (`seriesFiscales.table.*`); `dateFormat` `dd/MM/yyyy` si el idioma activo es `es`, si no `MM/dd/yyyy` (`.ts:167`); `tipoLabels` (Boleta, Factura, NotaCredito, Proforma → su texto) y `estadoLabels` (Activa, Inactiva → su texto); y los filtros actuales. Archivo «series-fiscales.xlsx» (`seriesFiscales.search.export-filename`). Carga masiva: no existe.

**Comportamientos particulares:** `search()` no reinicia la página (§2.1). Solo se cargan 1000 sucursales para el filtro.

**Observaciones:** si la API devolviera una serie de tipo Ticket o Nota de Débito (el enumerado los define pero no son configurables), la tabla mostraría la clave cruda `seriesFiscales.tipos.Ticket` / `seriesFiscales.tipos.NotaDebito`, que no existe en `es.json` (RN-SER-003). «No verificado» si existen esas filas en datos reales.

### 6.2 `/series-fiscales/nueva` y `/series-fiscales/:id` — Series Fiscales (detalle)

- **Componente:** `SeriesFiscalesDetailComponent` (`app/pages/series-fiscales/series-fiscales-detail/series-fiscales-detail.component.ts`, `.html`). Scope `series-fiscales` (alias `seriesFiscales`). Misma clase para las dos rutas.
- **Para qué sirve:** crear una serie de numeración fiscal para una sucursal y un tipo de comprobante; ver, editar o eliminar una existente.
- **Cómo se llega:** «Nuevo» (crear) o el enlace de la serie en la búsqueda.
- **Permiso de ruta:** `/series-fiscales/nueva` → `SeriesFiscales:Leer` + `SeriesFiscales:Crear`; `/series-fiscales/:id` → `SeriesFiscales:Leer`.
- **Migas y cabecera:** título «Series Fiscales»; migas «Inicio» / «SUNAT» / «Series Fiscales» (con enlace a `/series-fiscales`); botón volver a `/series-fiscales`.
- **Modos** (`ngOnInit`, `.ts:109-124`): sin `id` → **crear**; con `id` → **ver** (formulario deshabilitado) y «Editar» → **editar**. Siempre carga las sucursales (hasta 1000) para el selector.

| Aspecto | Crear | Ver | Editar (serie sin comprobantes) | Editar (serie que ya emitió comprobantes) |
|---|---|---|---|---|
| Encabezado «Series Fiscales» + «Datos del registro» + «Editar» | No | Sí (botón solo con `SeriesFiscales:Actualizar`) | No | No |
| Campos | Habilitados | Deshabilitados | Los cinco habilitados | **Solo Estado** habilitado; Sucursal, Tipo, Prefijo y Correlativo deshabilitados (`.ts:160-170`) |
| Aviso | — | — | Ayudas bajo Prefijo y Correlativo | Aviso sobre el formulario: «Esta serie ya emitió comprobantes: solo puedes cambiar su estado.» (`seriesFiscales.detail.locked-hint`, `.html:38-42`), además de las ayudas |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar | Cancelar, Guardar, Eliminar (la API rechazará eliminar) |

**Campos del formulario** (`.ts:92-101`, `.html:44-141`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Valor inicial al crear |
|---|---|---|---|---|---|---|
| Sucursal (`seriesFiscales.fields.sucursal`) | `sucursalId` | Select con todas las sucursales cargadas (hasta 1000) | Sí | `required` | «Este campo es obligatorio» (`general.errors.required`) | Vacío |
| Tipo de Comprobante (`seriesFiscales.fields.tipoComprobante`) | `tipoComprobante` | Select: Boleta, Factura, Nota de Crédito, Proforma | Sí | `required` | «Este campo es obligatorio» | Boleta |
| Prefijo de Serie (`seriesFiscales.fields.prefijoSerie`) | `prefijoSerie` | `matInput`, `maxlength="4"`; se convierte a mayúsculas al escribir (`toUpperPrefijo`, `.ts:154-158`) | Sí | `required`; patrón `^[A-Z0-9]{4}$` (`.ts:96`); validador del grupo `prefijoPorTipoValidator` (`.ts:45-50`): si el prefijo es válido en forma pero no corresponde al tipo (Boleta sin B, Factura sin F, Nota de Crédito sin B ni F) marca `prefijoPorTipo` | Vacío: «Este campo es obligatorio». Patrón: «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.» (`seriesFiscales.errors.prefijoInvalido`). Incoherencia con el tipo: el mismo texto `prefijoInvalido`, como mensaje de grupo bajo el campo, una vez que el campo fue tocado (`.html:93-100`). Ayuda (solo al editar o crear): «4 caracteres en mayúsculas. Boleta empieza con B, Factura con F, Nota de Crédito con B o F.» (`seriesFiscales.detail.prefijo-hint`) | Vacío |
| Correlativo Actual (`seriesFiscales.fields.correlativoActual`) | `correlativoActual` | `matInput type="number" min="0" step="1"` | Sí | `required`, `min(0)`, patrón `^\d+$` (`.ts:97`) | Vacío: «Este campo es obligatorio». Otro fallo: «El correlativo debe ser un número entero mayor o igual a 0.» (`seriesFiscales.errors.correlativoInvalido`). Ayuda: «Último número emitido; el siguiente comprobante usará este valor + 1.» (`seriesFiscales.detail.correlativo-hint`) | 0 |
| Estado (`seriesFiscales.fields.estadoSerie`) | `estadoSerie` | Select: «Activa», «Inactiva» | Sí | `required` | «Este campo es obligatorio» | Activa |

Las ayudas se muestran solo cuando `isEditMode()` es verdadero (crear incluido). Si la serie ya emitió comprobantes, la API conserva esos cuatro datos aunque el cuerpo los traiga (el SPA los reenvía sin cambios, `getRawValue()`, `.ts:187`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta | Modo ver y `*appPermiso="'Actualizar'"` (= `SeriesFiscales:Actualizar`, `.html:28`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()`: en crear navega a `/series-fiscales`; en editar restaura lo leído y deshabilita |
| Guardar (`actions.save`) | Pie | Crear o editar | `saving()` | `save()` (`.ts:182-211`): marca todo como tocado; si es inválido no envía. Crear → `POST /api/v1/seriesfiscales`; editar → `PUT /api/v1/seriesfiscales/:id`; cuerpo `{ sucursalId, tipoComprobante, prefijoSerie, correlativoActual, estadoSerie }`. Éxito: aviso y navega a `/series-fiscales` |
| Eliminar (`actions.delete`) | Pie, tras Guardar | Solo editar de una serie existente y `*appPermiso="'Eliminar'"` (= `SeriesFiscales:Eliminar`, `.html:161`) | `deleting()` | `confirmDelete()` |
| Botón volver | Cabecera | Siempre | — | `/series-fiscales` |

**Diálogos que abre:** `ConfirmDialogComponent` (400 px) al pulsar «Eliminar»: título «Confirmar eliminación» (`seriesFiscales.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta serie fiscal?» (`seriesFiscales.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/seriesfiscales/{id}`.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Serie fiscal guardada correctamente» (`seriesFiscales.detail.save-success`) | `.ts:196` |
| Código `SerieFiscal.PrefijoDuplicado` | Error | «Ya existe otra serie de este tipo de comprobante con ese prefijo.» (`seriesFiscales.errors.prefijoDuplicado`) | `.ts:35` |
| Código `SerieFiscal.ActivaDuplicada` | Error | «La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra.» (`seriesFiscales.errors.activaDuplicada`) | `.ts:36` |
| Código `SerieFiscal.ConComprobantes` | Error | «La serie ya emitió comprobantes: solo puedes cambiar su estado.» (`seriesFiscales.errors.conComprobantes`) | `.ts:37` |
| Código `SerieFiscal.NotFound` | Error | «La serie fiscal no existe.» (`seriesFiscales.errors.noEncontrada`) | `.ts:38` |
| Código `Sucursal.NotFound` | Error | «La sucursal seleccionada no existe.» (`seriesFiscales.errors.sucursalNoEncontrada`) | `.ts:39` |
| Códigos `CreateSerieFiscalCommand.PrefijoSerie` / `UpdateSerieFiscalCommand.PrefijoSerie` (validador de la API) | Error | «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.» (`seriesFiscales.errors.prefijoInvalido`) | `.ts:40-41` |
| Cualquier otro error al guardar | Error | «Error al guardar la serie fiscal» (`seriesFiscales.detail.save-error`) | `.ts:203` |
| Eliminación correcta | Éxito | «Serie fiscal eliminada correctamente» (`seriesFiscales.detail.delete-success`); navega a `/series-fiscales` | `.ts:230` |
| Eliminar con respuesta 409 (serie con comprobantes, RN-SER-011) | Error | «No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla.» (`seriesFiscales.detail.delete-in-use`) | `.ts:235-237` |
| Otro error al eliminar | Error | «Error al eliminar la serie fiscal» (`seriesFiscales.detail.delete-error`) | `.ts:238` |
| Error al cargar | Error | «Error al cargar la serie fiscal» (`seriesFiscales.detail.load-error`) | `.ts:136` |
| Fallan las sucursales del selector | Error | «No se pudieron cargar las sucursales» (`seriesFiscales.search.sucursales-error`) | `.ts:112` |

Las claves `seriesFiscales.errors.*` se usan solo como mensajes de guardado (y cuatro de ellas, además, para códigos de la API); el formulario no usa `saveErrorKey`, por lo que los códigos `Db.*` caen en el mensaje genérico de guardado.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir | `SucursalService.search` | `GET /api/v1/sucursales?pageNumber=1&pageSize=1000` | `Sucursales:Leer` |
| Abrir (con `id`) | `SerieFiscalService.getById` | `GET /api/v1/seriesfiscales/{id}` | `SeriesFiscales:Leer` (`SeriesFiscalesController.cs:26`) |
| Crear | `create` | `POST /api/v1/seriesfiscales` | `SeriesFiscales:Crear` (`:34`) |
| Editar | `update` | `PUT /api/v1/seriesfiscales/{id}` | `SeriesFiscales:Actualizar` (`:44`) |
| Eliminar | `delete` | `DELETE /api/v1/seriesfiscales/{id}` | `SeriesFiscales:Eliminar` (`:52`) |

**Comportamientos particulares**
- El estado «tiene comprobantes» (`tieneComprobantes`) lo informa la API; el SPA solo lo usa para bloquear los cuatro campos.
- El SPA no avisa de la regla «una sola serie activa por sucursal y tipo» hasta que la API la rechaza (`ActivaDuplicada`).
- El campo Sucursal de una serie existente no se puede cambiar si ya emitió comprobantes; en una serie sin comprobantes sí.

**Observaciones**
- Coherencia con la API (RN-SER-004 a 006): mismo patrón de prefijo, correlativo entero ≥ 0 y los dos estados.
- No hay pantalla para el envío de comprobantes a SUNAT: la opción del menú «SUNAT» solo contiene «Series Fiscales».

---

## 7. Pendientes y «No verificado»

**«En curso (sin commitear)»:** ninguno. Ningún archivo de las carpetas de esta parte (`pages/empresa`, `pages/sucursales`, `pages/empleados`, `pages/roles`, `pages/seguridad`, `pages/series-fiscales`) ni de los i18n de sus scopes está en la lista de cambios sin commitear de `REGLAS-AGENTES.md` §3.

**«No verificado»** (no se ejecutó la aplicación ni se llamó a la API):

| # | Tema | Por qué no se verificó |
|---|---|---|
| 1 | Si `/empresas` muestra solo la empresa del usuario o varias | El SPA no filtra; `GetEmpresasQueryHandler` no filtra por empresa de forma explícita y depende del filtro global del contexto. 01a (RN-EMP-017) afirma el aislamiento; no se comprobó contra datos reales |
| 2 | Qué ve el usuario si le falta `Empleados:Leer` o `Sucursales:Leer` en los buscadores que consultan esos módulos (gerente de sucursal, empleado de la cuenta, sucursal base del empleado, filtro de sucursal de series, buscadores de sucursal de la cuenta) | Los endpoints exigen ese permiso y el SPA no lo comprueba; el comportamiento visual (lista vacía, error silencioso) no se probó |
| 3 | Si la API impide editar o eliminar el rol llamado `Admin` | El SPA no tiene regla; en `Commands/Roles` solo se encontraron `Rol.AdminReservado` (asignar o quitar el rol Admin a una cuenta) y `Rol.PermisosPropios` |
| 4 | Si existen series de tipo Ticket o Nota de Débito en datos reales (la tabla mostraría la clave cruda) | Depende de los datos |
| 5 | Aspecto visual de iconos, avisos y colores (por ejemplo `check_circle`/`cancel`, aviso amarillo de permisos) | Solo lectura de plantillas |
| 6 | Que `Validators.email` de Angular acepte o rechace correos concretos | Comportamiento del framework; se documentó solo la regla declarada |

**Pendientes de producto detectados** (no son errores de documentación):
- No existe «restablecer contraseña» de otro usuario desde las cuentas (solo «Forzar Cambio de Contraseña»).
- No existe pantalla de ninguna opción de configuración de la empresa (por ejemplo `ExigirRegistroReceta`, RN-EMP-016).
- `EmpresaMaintenanceComponent` no está enrutado ni referenciado.
- No existe carga masiva ni exportación de cuentas, ni carga masiva de empleados ni de series fiscales.

---

## 8. Tabla de cobertura

**Rutas de `docs/_inventario/rutas-spa.csv` de este alcance (19 de 19)**

| Ruta | Componente | Ficha |
|---|---|---|
| `/empresas` | `EmpresaSearchComponent` | §1.1 |
| `/empresas/:id` | `EmpresaDetailComponent` | §1.2 |
| `/sucursales` | `SucursalesSearchComponent` | §2.1 |
| `/sucursales/nueva` | `SucursalesDetailComponent` (modo crear) | §2.2 |
| `/sucursales/carga-masiva` | `SucursalesCargaMasivaComponent` | §2.3 |
| `/sucursales/:id` | `SucursalesDetailComponent` (modos ver y editar) | §2.2 |
| `/empleados` | `EmpleadosSearchComponent` | §3.1 |
| `/empleados/nueva` | `EmpleadosDetailComponent` (modo crear) | §3.2 |
| `/empleados/:id` | `EmpleadosDetailComponent` (ver y editar) | §3.2 |
| `/roles` | `RolesSearchComponent` | §4.1 |
| `/roles/nueva` | `RolesDetailComponent` (crear) | §4.2 |
| `/roles/carga-masiva` | `RolesCargaMasivaComponent` | §4.3 |
| `/roles/:id` | `RolesDetailComponent` (ver y editar) | §4.2 |
| `/cuentas` | `CuentasSearchComponent` | §5.1 |
| `/cuentas/nueva` | `CuentasDetailComponent` (crear) | §5.2 |
| `/cuentas/:id` | `CuentasDetailComponent` (ver y editar) | §5.2 |
| `/series-fiscales` | `SeriesFiscalesSearchComponent` | §6.1 |
| `/series-fiscales/nueva` | `SeriesFiscalesDetailComponent` (crear) | §6.2 |
| `/series-fiscales/:id` | `SeriesFiscalesDetailComponent` (ver y editar) | §6.2 |

**Componentes de `docs/_inventario/componentes-pages.txt` de este alcance (16 de 16)**

| Componente (carpeta bajo `app/pages`) | Ficha o motivo |
|---|---|
| `empresa/empresa-search`, `empresa/empresa-detail` | §1.1, §1.2 |
| `empresa/empresa-maintenance` | Sin uso (comprobado: sin ruta ni referencias fuera de sus archivos), §1.2 Observaciones |
| `empresa/empresa-register` | No es de esta parte: lo documenta la parte A (`/auth/register`; desde `2f82346` pide la dirección de la sede principal) |
| `sucursales/sucursales-search`, `sucursales-detail`, `sucursales-carga-masiva` | §2.1, §2.2, §2.3 |
| `empleados/empleados-search`, `empleados-detail` | §3.1, §3.2 |
| `roles/roles-search`, `roles-detail`, `roles-carga-masiva` | §4.1, §4.2, §4.3 |
| `seguridad/cuentas-search`, `cuentas-detail` | §5.1, §5.2 |
| `series-fiscales/series-fiscales-search`, `series-fiscales-detail` | §6.1, §6.2 |

**Diálogos de `docs/_inventario/dialogos.txt`:** ninguna carpeta de este alcance contiene un diálogo propio (búsqueda en `dialogos.txt` sin resultados). Los diálogos que abren estas pantallas son `ConfirmDialogComponent` (compartido, [19a §6.1](19a-acceso-layout-y-compartidos.md)) en el detalle de cada módulo salvo Empresa: §2.2, §3.2, §4.2, §5.2, §6.2.

**Usos de `*appPermiso` de `docs/_inventario/app-permiso-usos.csv` de este alcance (18 de 18)**

| Archivo:línea | Expresión | Módulo resultante | Botón y ficha |
|---|---|---|---|
| `empresa-detail.component.html:27` | `'Actualizar'` | `Empresas:Actualizar` | «Editar», §1.2 |
| `sucursales-search.component.html:19` | `'Crear'` | `Sucursales:Crear` | «Carga Masiva», §2.1 |
| `sucursales-search.component.html:27` | `'Crear'` | `Sucursales:Crear` | «Nuevo», §2.1 |
| `sucursales-detail.component.html:27` | `'Actualizar'` | `Sucursales:Actualizar` | «Editar», §2.2 |
| `sucursales-detail.component.html:103` | `'Eliminar'` | `Sucursales:Eliminar` | «Eliminar», §2.2 |
| `empleados-search.component.html:19` | `'Crear'` | `Empleados:Crear` | «Nuevo», §3.1 |
| `empleados-detail.component.html:27` | `'Actualizar'` | `Empleados:Actualizar` | «Editar», §3.2 |
| `empleados-detail.component.html:137` | `'Eliminar'` | `Empleados:Eliminar` | «Eliminar», §3.2 |
| `roles-search.component.html:19` | `'Crear'` | `Seguridad:Crear` | «Carga Masiva», §4.1 |
| `roles-search.component.html:27` | `'Crear'` | `Seguridad:Crear` | «Nuevo», §4.1 |
| `roles-detail.component.html:27` | `'Actualizar'` | `Seguridad:Actualizar` | «Editar», §4.2 |
| `roles-detail.component.html:307` | `'Eliminar'` | `Seguridad:Eliminar` | «Eliminar», §4.2 |
| `cuentas-search.component.html:18` | `'Crear'` | `Seguridad:Crear` | «Nuevo», §5.1 |
| `cuentas-detail.component.html:107` | `'Actualizar'` | `Seguridad:Actualizar` | «Editar», §5.2 |
| `cuentas-detail.component.html:268` | `'Eliminar'` | `Seguridad:Eliminar` | «Eliminar», §5.2 |
| `series-fiscales-search.component.html:20` | `'Crear'` | `SeriesFiscales:Crear` | «Nuevo», §6.1 |
| `series-fiscales-detail.component.html:28` | `'Actualizar'` | `SeriesFiscales:Actualizar` | «Editar», §6.2 |
| `series-fiscales-detail.component.html:161` | `'Eliminar'` | `SeriesFiscales:Eliminar` | «Eliminar», §6.2 |

Además de `*appPermiso`, `RolesDetailComponent` comprueba en código `Seguridad:GestionarPermisos` (`roles-detail.component.ts:48`) y `CuentasDetailComponent` comprueba `Seguridad:AsignarRoles` (`cuentas-detail.component.ts:59`); ambas están documentadas en §4.2 y §5.2.

**Endpoints de este alcance sin pantalla propia en esta parte** (de `endpoints.csv`, controladores Empresas, Sucursales, Empleados, Roles, Cuentas, SeriesFiscales): `POST /api/v1/Empresas/registrar` (parte A); `GET /api/v1/Sucursales/permitidas` (selector de sucursal de la cabecera, parte A); `POST /api/v1/Roles/assign-to-user` y `POST /api/v1/Roles/remove-from-user` (el SPA usa en su lugar `POST` y `DELETE /api/v1/cuentas/{id}/roles/{rolId}`, por lo que estos dos endpoints no se llaman desde estas pantallas: confirmado en `rol.service.ts` y `cuenta.service.ts`).

---

## 9. Claves i18n sin uso

Método: para cada clave del `es.json` del scope se buscó la forma completa (con el prefijo del scope) en todo `src/` y la forma relativa en los archivos que cargan ese scope, entre comillas. Las claves construidas dinámicamente se comprobaron aparte y **sí se usan**: `empresa.estado.0` a `3` (`empresa-detail.component.ts:80`, `empresa-search.component.ts:127`), `roles.permisos.modulo.*` y `roles.permisos.accion.*` (`roles-detail.component.ts:184`), `seriesFiscales.tipos.*` y `seriesFiscales.estados.*` (plantillas de series).

| Scope | Claves sin ninguna referencia |
|---|---|
| `empresa` (100 claves) | `searchPlaceholder`, `add`, `cols.fechaVencimiento`, `cols.cantidadSucursales`, `form.titleView`, `form.estado`, `form.fechaVencimiento`, `notifications.loadError`, `help.nombre`, `help.ruc`, `help.estado`, `noData`, `showing`, `detail.title-edit`, `fields.estaVigente` (15). Además, solo las referencia el componente sin ruta `empresa-maintenance`: `form.titleEdit`, `form.nombre`, `form.ruc`, `form.vigente`, `form.noVigente`, `form.back`, `notifications.updated`, `notifications.loadSingleError`, `notifications.updateError` (9). Las claves `empresa.register.*` y varias de `form.*` y `notifications.*` las usa `empresa-register` (parte A) |
| `sucursales` (21) | Ninguna |
| `empleados` (31) | Ninguna |
| `roles` (91) | `detail.title-new`, `detail.title-edit`, `detail.nombreRolHint`, `fields.nivelJerarquiaDesde`, `fields.nivelJerarquiaHasta`, `fields.createdAt`, `configuracion.sucursales-save`, `configuracion.sucursales-save-success`, `configuracion.sucursales-save-error`, `permisos.title`, `permisos.subtitle`, `permisos.save`, `permisos.save-success`, `permisos.save-error`, `permisos.no-changes` (15) |
| `cuentas` (57) | `search.export-filename`, `detail.title-new`, `detail.title-edit`, `fields.roles`, `fields.sucursales`, `roles.assign-success`, `roles.assign-error`, `roles.remove-success`, `roles.remove-error` (9). Además `detail.confirm-delete-confirm` y `detail.confirm-delete-cancel` se pasan al diálogo pero este no las muestra (§5.2) |
| `series-fiscales` (43) | Ninguna |
| `seguridad` (59) | **Las 59**: ningún archivo de `src/` carga el scope `seguridad` (`scope: 'seguridad'` ni `provideTranslocoScope('seguridad')`) ni usa claves con prefijo `seguridad.`. Es un scope heredado de la carpeta `pages/seguridad`; las pantallas actuales usan `cuentas` y `roles` |

---

## 10. Fuentes

**SPA (`sofia-web-spa/`)**
- Enrutado y permisos: `src/app/app.routes.ts`, `src/app/core/auth/route-permissions.ts`, `src/app/core/auth/permission.guard.ts`, `src/app/core/auth/auth.service.ts`, `src/app/core/interceptors/auth.interceptor.ts`, `src/app/shared/directives/permiso.directive.ts`, `src/app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts`, `visible-nav-items.ts`, `sidebar.component.ts`.
- Compartidos: `src/app/core/services/shared/error-notifier.service.ts`, `search-state.service.ts`, `src/app/shared/utils/save-error-key.util.ts`, `src/app/shared/components/confirm-dialog/confirm-dialog.component.ts`.
- Empresa: `src/app/pages/empresa/empresa.routes.ts`, `empresa-search/*`, `empresa-detail/*`, `empresa-maintenance/empresa-maintenance.component.ts`, `models/empresa.model.ts`, `services/empresa.service.ts`.
- Sucursales: `src/app/pages/sucursales/sucursales.routes.ts`, `sucursales-search/*`, `sucursales-detail/*`, `sucursales-carga-masiva/*`, `models/sucursal.model.ts`, `services/sucursal.service.ts`.
- Empleados: `src/app/pages/empleados/empleados.routes.ts`, `empleados-search/*`, `empleados-detail/*`, `models/empleado.model.ts`, `services/empleado.service.ts`.
- Roles: `src/app/pages/roles/roles.routes.ts`, `roles-search/*`, `roles-detail/*`, `roles-carga-masiva/*`, `models/rol.model.ts`, `models/menu-preview.ts`, `services/rol.service.ts`.
- Cuentas: `src/app/pages/seguridad/cuentas.routes.ts`, `cuentas-search/*`, `cuentas-detail/*`, `models/cuenta.model.ts`, `services/cuenta.service.ts`.
- Series fiscales: `src/app/pages/series-fiscales/series-fiscales.routes.ts`, `series-fiscales-search/*`, `series-fiscales-detail/*`, `models/serie-fiscal.model.ts`, `services/serie-fiscal.service.ts`.
- Textos: `public/i18n/es.json` (raíz) y `public/i18n/{empresa,sucursales,empleados,roles,cuentas,series-fiscales,seguridad}/es.json`.

**Backend (`sofia-apis/src/`, solo lectura puntual)**
- `SOFIA.API/Controllers/EmpresasController.cs`, `SucursalesController.cs`, `EmpleadosController.cs`, `RolesController.cs`, `CuentasController.cs`, `SeriesFiscalesController.cs` (permisos por endpoint).
- `SOFIA.Application/Empresas/Commands/UpdateEmpresa/*` y `Queries/GetEmpresas/GetEmpresasQueryHandler.cs`; `SOFIA.Domain/Entities/Empresa.cs`.
- `SOFIA.Application/Sucursales/Commands/{Create,Update}Sucursal/*Handler.cs`; `Commands/DeleteSucursal/DeleteSucursalCommandHandler.cs`, `Commands/UpdateSucursal/UpdateSucursalCommandValidator.cs`; `SOFIA.Domain/Entities/Sucursal.cs`; `SOFIA.Application/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandHandler.cs`.
- `SOFIA.Application/Empleados/Commands/DeleteEmpleado/*Handler.cs`.
- `SOFIA.Application/Security/Commands/Roles/{AssignPermission,AssignRol,RemoveRol}/*Handler.cs`; `Commands/Cuentas/{UpdateCuenta,DeleteCuenta,AssignSucursal}/*Handler.cs`; `Commands/Register/RegisterAccountCommandHandler.cs`.

**Inventarios y documentos hermanos**
- `docs/_inventario/rutas-spa.csv`, `componentes-pages.txt`, `dialogos.txt`, `app-permiso-usos.csv`, `endpoints.csv`, `permisos.csv`, `i18n-scopes.csv`.
- [19a — acceso, layout y compartidos](19a-acceso-layout-y-compartidos.md) (§5.3, §5.4, §6.1 a §6.6, §6.12, §7.1, §8).
- [01a — reglas de administración y seguridad](../01-reglas-de-negocio/01a-administracion-y-seguridad.md) (RN-EMP, RN-SUC, RN-EPL, RN-ROL, RN-CTA, RN-SER).
- [05b — matriz de permisos](../05-seguridad/05b-matriz-de-permisos.md).
