# Inventario de pantallas del SPA — Parte F: punto de venta

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-13 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código del árbol de trabajo del SPA (componentes, plantillas, servicios, modelos, utilidades e i18n de `pages/pos` y `pages/sesiones-caja`). Del backend solo se abrieron controladores, algunos handlers y los DTO necesarios para anotar permisos y comparar validaciones. No se usó el navegador ni la API local. `git status --short` no mostró cambios sin commitear en ninguno de los dos repos al empezar, así que no hay marcas «En curso (sin commitear)» en este documento. |

Este documento es la parte F del inventario de pantallas. Cubre el punto de venta completo: las **6 rutas** `/sesiones-caja`, `/sesiones-caja/nueva`, `/sesiones-caja/:id`, `/pos`, `/pos/nueva` (incluido el modo `?continuar=<id>`) y `/pos/:id`, y los **5 diálogos** de `pages/pos/dialogs`. Es la base de los capítulos del manual sobre caja y ventas, así que cada elemento que el cajero ve o toca se describe con su texto exacto. Lo compartido (guards, `page-header`, `ErrorNotifierService`, textos de la raíz) lo documenta la [parte A](19a-acceso-layout-y-compartidos.md); las reglas de negocio están en [01c2](../01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) (reglas `VEN` y `CMP`) y [01c3](../01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) (reglas `CAJ` y `DEV`), y los permisos por endpoint en [05b](../05-seguridad/05b-matriz-de-permisos.md).

Convenciones: las rutas de archivo son relativas a `sofia-web-spa/` salvo que empiecen por `sofia-apis/`; `app/` abrevia `src/app/`. Entre paréntesis va la clave i18n con su prefijo real: `pos.*` está en `public/i18n/pos/es.json`, `sesionesCaja.*` en `public/i18n/sesiones-caja/es.json` (scope `sesiones-caja`, leído con `read: 'sesionesCaja'`), `recetasMedicas.*` en `public/i18n/recetas-medicas/es.json` y el resto en la raíz `public/i18n/es.json`. Las plantillas del punto de venta abren con `*transloco="let t; scope: 'pos'; read: 'pos'"`, así que `t('x')` equivale a `pos.x`. Para no repetir rutas largas, los archivos se citan con estas abreviaturas (todas bajo `app/pages/`):

| Abreviatura | Archivo |
|---|---|
| `NV.ts`, `NV.html` | `pos/nueva-venta/nueva-venta.component.ts`, `.html` |
| `VS.ts`, `VS.html` | `pos/search/ventas-search.component.ts`, `.html` |
| `VD.ts`, `VD.html` | `pos/venta-detail/venta-detail.component.ts`, `.html` |
| `APE.ts`, `APE.html` | `pos/dialogs/aperturar-caja-dialog/aperturar-caja-dialog.component.ts`, `.html` |
| `CNF.ts`, `CNF.html` | `pos/dialogs/confirmar-venta-dialog/confirmar-venta-dialog.component.ts`, `.html` |
| `CMP.ts`, `CMP.html` | `pos/dialogs/comprobante-dialog/comprobante-dialog.component.ts`, `.html` |
| `ANU.ts`, `ANU.html` | `pos/dialogs/anular-venta-dialog/anular-venta-dialog.component.ts`, `.html` |
| `DEV.ts`, `DEV.html` | `pos/dialogs/devolver-venta-dialog/devolver-venta-dialog.component.ts`, `.html` |
| `VSV.ts` | `pos/services/venta.service.ts` |
| `RPS.ts` | `pos/services/receipt-print.service.ts` |
| `VM.ts` | `pos/models/venta.model.ts` |
| `DRAFT.ts` | `pos/nueva-venta/pos-draft.ts` |
| `VUELTO.ts` | `pos/utils/vuelto.util.ts` |
| `SS.ts`, `SS.html` | `sesiones-caja/sesiones-search/sesiones-search.component.ts`, `.html` |
| `SD.ts`, `SD.html` | `sesiones-caja/sesiones-detail/sesiones-detail.component.ts`, `.html` |
| `SCS.ts` | `sesiones-caja/services/sesion-caja.service.ts` |
| `SCM.ts` | `sesiones-caja/models/sesion-caja.model.ts` |

## Contenido

0. Comportamientos comunes, inventario de archivos y mapa del flujo
1. Sesiones de caja: `/sesiones-caja`, `/sesiones-caja/nueva`, `/sesiones-caja/:id`
2. Diálogo de apertura de caja (`AperturarCajaDialogComponent`)
3. Nueva venta: `/pos/nueva` (incluye `?continuar=<id>`)
4. Diálogo de confirmación de venta (`ConfirmarVentaDialogComponent`)
5. Diálogo del comprobante (`ComprobanteDialogComponent`)
6. Búsqueda de ventas: `/pos`
7. Detalle de venta: `/pos/:id`
8. Diálogo de anulación (`AnularVentaDialogComponent`)
9. Diálogo de devolución (`DevolverVentaDialogComponent`)
10. Pendientes y «No verificado»
11. Tabla de cobertura
12. Claves i18n sin uso
13. Fuentes

---

## 0. Comportamientos comunes, inventario de archivos y mapa del flujo

### 0.1 Inventario de archivos de `pages/pos` y `pages/sesiones-caja`

Resultado de listar las dos carpetas completas (37 archivos, 8070 líneas contando `.scss` y pruebas). Todo lo que tiene pantalla o diálogo está en la tabla del apartado 0.2; aquí se fija qué hace cada archivo.

| Archivo | Tipo | Para qué sirve |
|---|---|---|
| `pos/pos.routes.ts` | Rutas | `''` → `VentasSearchComponent`; `nueva` (con `data: { isNew: true }`, que el componente no lee) → `NuevaVentaComponent`; `:id` → `VentaDetailComponent` (`pos.routes.ts:3-19`) |
| `pos/nueva-venta/nueva-venta.component.ts`, `.html`, `.scss`, `.spec.ts` | Pantalla | Terminal de venta (apartado 3) |
| `pos/nueva-venta/pos-draft.ts`, `pos-draft.spec.ts` | Utilidad | Serializa, lee y caduca el borrador local del carrito (apartado 3.9) |
| `pos/search/ventas-search.component.ts`, `.html` | Pantalla | Búsqueda de ventas (apartado 6) |
| `pos/venta-detail/venta-detail.component.ts`, `.html`, `.spec.ts` | Pantalla | Detalle de venta (apartado 7) |
| `pos/dialogs/aperturar-caja-dialog/*` | Diálogo | Apertura de caja desde el terminal (apartado 2) |
| `pos/dialogs/confirmar-venta-dialog/*` | Diálogo | Confirmación previa al cobro (apartado 4) |
| `pos/dialogs/comprobante-dialog/*` (`.ts`, `.html`, `.scss`, `.spec.ts`) | Diálogo | Comprobante y ticket imprimible (apartado 5) |
| `pos/dialogs/anular-venta-dialog/*` | Diálogo | Motivo de anulación (apartado 8) |
| `pos/dialogs/devolver-venta-dialog/*` (`.ts`, `.html`, `.spec.ts`) | Diálogo | Devolución de líneas (apartado 9) |
| `pos/services/venta.service.ts` (+ `.spec.ts`) | Servicio | Todas las llamadas a `/api/v1/ventas` y `POST /api/v1/devoluciones` (`VSV.ts:24-85`) |
| `pos/services/receipt-print.service.ts` (+ `.spec.ts`) | Servicio | Imprime el ticket en un `iframe` oculto (apartado 5) |
| `pos/models/venta.model.ts` | Modelos | Enumerados `EstadoVenta`, `MetodoPago`, `CondicionVenta`, `DestinoDevolucion`, DTO y peticiones |
| `pos/utils/vuelto.util.ts` (+ `.spec.ts`) | Utilidad | `roundMoney` y `calcularVuelto` (apartado 3.7) |
| `sesiones-caja/sesiones-caja.routes.ts` | Rutas | `''` → `SesionesSearchComponent`; `nueva` y `:id` → `SesionesDetailComponent` (`sesiones-caja.routes.ts:3-19`) |
| `sesiones-caja/sesiones-search/*` (`.ts`, `.html`, `.spec.ts`) | Pantalla | Búsqueda de sesiones de caja (apartado 1.1) |
| `sesiones-caja/sesiones-detail/*` (`.ts`, `.html`, `.spec.ts`) | Pantalla | Apertura como pantalla, detalle y cierre (apartados 1.2 y 1.3) |
| `sesiones-caja/services/sesion-caja.service.ts` (+ `.spec.ts`) | Servicio | Llamadas a `/api/v1/pos` (`SCS.ts:17-45`) |
| `sesiones-caja/models/sesion-caja.model.ts` | Modelos | `EstadoSesion` (0 Abierta, 1 Cerrada, 2 Cuadrada), `SesionCaja` y peticiones |

No hay más componentes ni diálogos que los 8 + 2 componentes y 5 diálogos de `componentes-pages.txt` y `dialogos.txt` (comprobado con el listado de archivos). `nueva-venta.component.scss` (979 líneas) es solo estilo y no se documenta.

### 0.2 Mapa de rutas, permisos y diálogos

| Ruta | Componente | Permiso de ruta (guard) | Diálogos que abre |
|---|---|---|---|
| `/sesiones-caja` | `SesionesSearchComponent` | `POS:Leer` | — |
| `/sesiones-caja/nueva` | `SesionesDetailComponent` (modo crear) | `POS:Leer` + `POS:AperturarCaja` | — |
| `/sesiones-caja/:id` | `SesionesDetailComponent` (modo ver) | `POS:Leer` | — |
| `/pos` | `VentasSearchComponent` | `Ventas:Leer` | — |
| `/pos/nueva` | `NuevaVentaComponent` | `Ventas:Leer` + `Ventas:Crear` | Apertura de caja, Confirmación de venta, Comprobante |
| `/pos/:id` | `VentaDetailComponent` | `Ventas:Leer` | Comprobante (reimpresión), Anulación, Devolución |

El `permissionGuard` toma el primer segmento de la URL: `sesiones-caja` → módulo `POS` y `pos` → módulo `Ventas` (`app/core/auth/route-permissions.ts:19-20`); exige siempre `Leer` y, si el segundo segmento es `nueva`, además la acción de `ROUTE_ACTIONS` (`sesiones-caja/nueva` → `AperturarCaja`, `route-permissions.ts:31`) o, para cualquier otra ruta `nueva`, `Crear` (`route-permissions.ts:36,68-72`). `/pos/nueva?continuar=<id>` se resuelve igual que `/pos/nueva` porque el guard descarta la query (`route-permissions.ts:75-81`): **no exige `Ventas:Actualizar`**, aunque cobrar o actualizar una venta en espera sí lo exige la API (hallazgo a1 de [05b](../05-seguridad/05b-matriz-de-permisos.md)). Sin permiso: aviso «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard` (`app/core/auth/permission.guard.ts:28-29`). Detalle del guard en [19a §5.3](19a-acceso-layout-y-compartidos.md).

### 0.3 Mapa del flujo de caja y venta

```mermaid
flowchart TD
    A[Entrar a /pos/nueva] --> B{Caja propia abierta}
    B -- No --> C[Aviso y boton Aperturar Caja]
    C --> D[Dialogo Aperturar Caja]
    D --> B
    B -- Si --> E[Buscar producto y lote]
    E --> F[Carrito]
    F --> G[Cliente y receta]
    G --> H[Pagos]
    H --> I{Accion}
    I -- Procesar Venta --> J[Dialogo de confirmacion]
    J --> K[POST o PUT cobro]
    K --> L[Dialogo del comprobante]
    L --> M[Imprimir, Nueva Venta POS o Volver a Caja POS]
    I -- Guardar y Atender Siguiente --> N[Venta en espera Pendiente]
    N --> O[Detalle de la venta: Cobrar Venta]
    O --> P[/pos/nueva con continuar]
    P --> B
    M --> Q[Detalle: Devolver o Anular]
    Q --> R[Cierre de caja en sesiones-caja]
```

### 0.4 Permisos de otros módulos que el punto de venta necesita

Un rol que solo tenga permisos de `Ventas` y `POS` no puede vender: el terminal consulta endpoints de otros módulos. Ninguno de estos controles existe en el SPA (si falta el permiso, el terminal muestra el mensaje genérico de esa llamada o, en la búsqueda de caja, el aviso de «aperturar la caja»). Fuente: [05b §4.b](../05-seguridad/05b-matriz-de-permisos.md), reverificado en el código actual.

| Pantalla | Permiso extra | Endpoint | Para qué lo usa | Fuente |
|---|---|---|---|---|
| `/pos/nueva` | `POS:Leer` | `GET /api/v1/POS/sesiones` | Buscar la caja abierta propia | `NV.ts:384`; `POSController.cs:28` |
| `/pos/nueva` | `Medicamentos:Leer` | `GET /api/v1/medicamentos`, `GET /api/v1/medicamentos/condiciones-venta` | Catálogo de productos y chips de condición | `NV.ts:280,371`; `MedicamentosController.cs:23,27` |
| `/pos/nueva` | `Pacientes:Leer` | `GET /api/v1/pacientes`, `GET /api/v1/pacientes/{id}` | Buscar cliente y restaurarlo del borrador | `NV.ts:318-319,529`; `PacientesController.cs:29,38` |
| `/pos/nueva` | `Recetas:Leer` | `GET /api/v1/recetas` | Recetas registradas del cliente | `NV.ts:875`; `RecetasController.cs:24` |
| `/pos/nueva` | `Recetas:Analizar` | `POST /api/v1/recetas/analizar` | Pestaña «Receta» (escaneo con IA); sí tiene control en el SPA | `NV.ts:150,754`; `RecetasController.cs:48` |
| `/pos/nueva` | `Recetas:Crear` | (enlace a `/recetas-medicas/nueva`) | Enlace «Registrar receta»; sí tiene control en el SPA | `NV.html:443` |
| `/pos/nueva` | `Seguros:Leer` | `GET /api/v1/seguros` | Solo el bloque de aseguradora, que está oculto (apartado 3.8) | `NV.ts:332`; `SegurosController.cs:26` |
| `/pos` | `Empleados:Leer` | `GET /api/v1/empleados` | Autocompletar del filtro «Empleado» | `VS.ts:87`; `EmpleadosController.cs:15` |
| `/pos` | `Pacientes:Leer` | `GET /api/v1/pacientes` | Autocompletar del filtro «Cliente» | `VS.ts:102-103` |
| `/pos/:id` | `Devoluciones:ProcesarDevolucion` | `POST /api/v1/devoluciones` | Botón «Devolver» (control en el SPA) | `VD.ts:165`; `DevolucionesController.cs:11` |
| `/pos/:id` | `Ventas:Anular`, `Ventas:Crear`, `Ventas:Actualizar` | `PUT ventas/{id}/anular`, `PUT ventas/{id}/completar`, `PUT ventas/{id}/detalles` | Botones «Anular Venta» y «Cobrar Venta» (controles en el SPA) | `VD.ts:95,101` |

### 0.5 Avisos al usuario, i18n y estado de las búsquedas

- **Avisos.** Todos los avisos son snackbar con el botón «Cerrar» (`actions.close`): error 5 s, éxito 3 s (`app/core/services/shared/error-notifier.service.ts:26,30`; [19a §6.12](19a-acceso-layout-y-compartidos.md)). No hay interceptor global de errores: cada pantalla decide el texto. El texto viaja desde el código de error de la respuesta (`err.error?.code`) a un mapa propio de cada pantalla; si el código no está mapeado se muestra el mensaje genérico de la acción. Los mapas completos están en los apartados 2, 3.12, 7, 8 y 9.
- **Textos de la raíz que se usan aquí.** «Cancelar» (`actions.cancel`), «Limpiar» (`actions.clear`), «Buscar» (`actions.search`), «Exportar» (`actions.export`), «Nuevo» (`actions.new`), «Todos» (`generic-labels.all`), «Resultados» (`generic-labels.results`), «Sin resultados» (`generic-labels.no-results`), «Ingresa los criterios de búsqueda» (`generic-labels.search-subtitle`), «Datos del registro» (`generic-labels.detail-subtitle`), migas «Inicio» (`breadcrumbs.home`), «Punto de Venta» (`breadcrumbs.puntoVenta`), «Ventas» (`breadcrumbs.ventas`), «Sesiones de Caja» (`breadcrumbs.sesionesCaja`) y «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`).
- **Estado de las búsquedas.** `/pos` y `/sesiones-caja` guardan filtros, página y tamaño de página en `SearchStateService` (memoria; se pierde al recargar el navegador) con la URL sin query como clave, y los restauran al volver; «Limpiar» lo borra ([19b §0](19b-administracion.md)). El tablero (`/dashboard`) abre estas pantallas con filtros ya puestos mediante ese mismo estado: «Ver mi caja» abre `/sesiones-caja/:id` (`dashboard.component.html:273`), «Abrir caja» abre `/sesiones-caja/nueva` con `POS:AperturarCaja` (`dashboard.component.html:282-287`), y los enlaces de `app/pages/dashboard/dashboard-links.ts` abren `/pos` con «Completada» + empleado + día de hoy (`misVentasDeHoyLink`, `:27-30`), con «Pendiente» (`ventasEnEsperaLink`, `:32-34`) o `/sesiones-caja` con «Abierta» (`sesionesAbiertasLink`, `:36-41`). Los textos del tablero están en [19a §3](19a-acceso-layout-y-compartidos.md).
- **Menú.** Grupo «Punto de Venta» (título de sección `breadcrumbs.puntoVenta`) con dos opciones: «Sesiones de Caja» (`nav.sesionesCaja`, `/sesiones-caja`) y «Ventas» (`nav.ventas`, `/pos`) (`app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:118-133`). **No hay opción de menú para el terminal** `/pos/nueva`, ni para Devoluciones (el módulo no tiene pantalla de listado): al terminal se llega desde el botón «Nueva Venta POS» de `/pos`, desde «Cobrar Venta» del detalle (con `?continuar=`), desde «Nueva Venta POS» del diálogo del comprobante o escribiendo la URL. El layout horizontal tiene una opción «POS / Ventas» que apunta a `/pos` (`app/matdash/layouts/full/horizontal/sidebar/sidebar-data.ts:16-18`); el layout horizontal se documenta en [19a §4.7](19a-acceso-layout-y-compartidos.md).

---

## 1. Sesiones de caja

Carpeta `app/pages/sesiones-caja`, scope i18n `sesiones-caja` (`provideTranslocoScope('sesiones-caja')`, archivo `public/i18n/sesiones-caja/es.json`) leído con el alias `sesionesCaja`: las claves que usa el código llevan el prefijo `sesionesCaja.` (por ejemplo `sesionesCaja.search.search-error`, `SS.ts:98`). Una **sesión de caja** es el turno de un cajero: se abre con un monto inicial en efectivo y se cierra con el monto contado (arqueo). Reglas: RN-CAJ-001 a RN-CAJ-027 en [01c3](../01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md). Estados: `Abierta` (0), `Cerrada` (1) y `Cuadrada` (2) (`SCM.ts:1-5`), con los textos «Abierta», «Cerrada» y «Cuadrada» (`sesionesCaja.estado.*`).

### 1.1 `/sesiones-caja` — Sesiones de Caja (búsqueda)

- **Componente:** `SesionesSearchComponent` (`SS.ts`, `SS.html`). Scope `sesiones-caja` (alias `sesionesCaja`).
- **Para qué sirve:** buscar sesiones de caja por estado y rango de fecha de apertura, abrirlas para ver su detalle, exportarlas a Excel y entrar a abrir una caja nueva.
- **Cómo se llega:** menú lateral, grupo «Punto de Venta» → «Sesiones de Caja» (`nav.sesionesCaja`); enlace «Ver cajas abiertas» del tablero (que ya trae el filtro «Abierta»); botón volver del detalle (`backRoute="/sesiones-caja"`).
- **Permiso de ruta:** `POS:Leer` (`route-permissions.ts:19`). Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Sesiones de Caja» (`sesionesCaja.title`); migas «Inicio» (enlace a `/dashboard`) / «Punto de Venta» / «Sesiones de Caja» (activa). Sin botón volver (`SS.html:3-6`, `SS.ts:48-52`). Tarjeta de filtros con el título «Búsqueda de Sesiones de Caja» (`sesionesCaja.search.title`) y el subtítulo «Ingresa los criterios de búsqueda».
- **Modos:** uno. Los resultados no se cargan al entrar: solo tras pulsar «Buscar» o si hay estado guardado (o llegado desde el tablero) (`ngOnInit`, `SS.ts:58-66`).

**Filtros de búsqueda** (`SS.ts:42-46`, `SS.html:30-69`)

| Etiqueta (clave) | Control (`formControlName`) | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Estado (`sesionesCaja.fields.estado`) | `estadoSesion` | Select | «— Todos —» (`generic-labels.all` entre guiones largos, valor nulo = sin filtro), «Abierta», «Cerrada», «Cuadrada» (`SS.ts:32`) | Nulo |
| Apertura desde (`sesionesCaja.fields.fechaInicio`) | `fechaInicio` | Campo con selector de fecha (`mat-datepicker`) | Se envía como el inicio del día local en ISO (`toLocalDayStartIso`) | Nulo |
| Apertura hasta (`sesionesCaja.fields.fechaFin`) | `fechaFin` | Campo con selector de fecha | Se envía como el último instante del día local (`23:59:59.999`, `toLocalDayEndIso`) | Nulo |

La pantalla **no ofrece filtro por sucursal ni por cajero**, aunque la API los admite (RN-CAJ-021): el servicio acepta `sucursalId` y `empleadoId` (`SCS.ts:23-24`) pero esta búsqueda nunca los envía. Qué sesiones se ven lo decide la API: un usuario que no es Admin ve las de sus sucursales accesibles y de **todos** los cajeros (RN-CAJ-020). «Limpiar» (`actions.clear`) vacía el formulario, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`limpiar()`, `SS.ts:103-110`).

**Tabla de resultados** (`SS.html:92-166`; el SPA no ordena: la API ordena por apertura descendente, RN-CAJ-021)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Sucursal (`sesionesCaja.table.sucursal`) | `sucursalNombre` | Texto | Sí: `routerLink [row.id]` → `/sesiones-caja/:id` |
| Cajero (`sesionesCaja.table.empleado`) | `empleadoNombre` | Texto | No |
| Apertura (`sesionesCaja.table.fechaApertura`) | `fechaHoraApertura` | `dd/MM/yyyy HH:mm`, en la hora del navegador | No |
| Cierre (`sesionesCaja.table.fechaCierre`) | `fechaHoraCierre` | `dd/MM/yyyy HH:mm`, o «—» si está abierta | No |
| Monto Apertura (`sesionesCaja.table.montoApertura`) | `montoAperturaEfectivo` | Número con 2 decimales | No |
| Estado (`sesionesCaja.table.estado`) | `estadoSesion` | «Abierta», «Cerrada» o «Cuadrada» | No |

Encabezado de la tabla «Resultados (N)» (`generic-labels.results` y el total). Paginación 10, 25 y 50, con botones primera/última y 10 por defecto. Sin resultados: «Sin resultados» (`generic-labels.no-results`). Bajo la tabla, a la izquierda el botón «Exportar» con icono `download` y a la derecha el paginador. El monto declarado, el calculado y la diferencia no están en la tabla: se ven en el detalle y en el Excel.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros | `*appPermiso="'AperturarCaja'"` (= `POS:AperturarCaja`, `SS.html:20`) | Nunca | `goToNew()` → `/sesiones-caja/nueva` |
| Limpiar (`actions.clear`) | Pie de la tarjeta de filtros | Siempre | Nunca | `limpiar()` |
| Buscar (`actions.search`); con indicador de carga | Pie de la tarjeta de filtros | Siempre | `loading()` | `buscar()` → `GET /api/v1/pos/sesiones` (`SS.ts:77-101`) |
| Exportar (`actions.export`), icono `download` | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/pos/sesiones/exportar` |
| Enlace con el nombre de la sucursal | Primera columna | Con resultados | — | Navega a `/sesiones-caja/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar sesiones de caja» (`sesionesCaja.search.search-error`) | `SS.ts:98` |
| Falla la exportación | Error | «Error al buscar sesiones de caja» (misma clave) | `SS.ts:151` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso que exige el endpoint |
|---|---|---|---|
| Buscar o cambiar de página | `SesionCajaService.search` (`SCS.ts:18-29`) | `GET /api/v1/pos/sesiones?pageNumber&pageSize[&estadoSesion][&fechaInicio][&fechaFin]` (solo envía los filtros con valor; `estadoSesion=0` sí se envía) | `POS:Leer` (`POSController.cs:28`) |
| Exportar | `exportar` (`SCS.ts:43-45`) | `POST /api/v1/pos/sesiones/exportar` (respuesta blob) | `POS:Leer` (`POSController.cs:45`) |

**Exportación:** cuerpo `SesionCajaExportRequest` (`SCM.ts:44-51`): `headers` con 9 textos en este orden: «Sucursal» (`table.sucursal`), «Cajero» (`table.empleado`), «Apertura» (`table.fechaApertura`), «Cierre» (`table.fechaCierre`), «Monto Apertura» (`table.montoApertura`), «Monto Cierre Declarado (S/)» (`fields.montoCierreDeclarado`), «Monto Cierre Calculado (S/)» (`fields.montoCierreCalculado`), «Diferencia Arqueo (S/)» (`fields.diferencia`) y «Estado» (`table.estado`); `dateFormat` `dd/MM/yyyy HH:mm` si el idioma activo es `es` y `MM/dd/yyyy HH:mm` si no (`SS.ts:140`); y los filtros actuales del formulario (`estadoSesion`, `fechaInicio`, `fechaFin`; `sucursalId` va siempre vacío). Archivo descargado: «sesiones-caja.xlsx» (`sesionesCaja.search.export-filename`). Más de 10 000 filas: la API no genera el archivo (RN-CAJ-024, RN-IMP-008). No hay carga masiva de sesiones.

**Comportamientos particulares:** restaura filtros, página y tamaño de página al volver (apartado 0.5); `buscar()` **no reinicia la página**: si el cajero cambia un filtro estando en la página 3, «Buscar» pide la página 3 (`SS.ts:79-83`); la fecha de apertura se muestra en la hora del navegador, pero el Excel la formatea en UTC sin convertir (RN-CAJ-024).

**Observaciones:** no se puede ver «solo mis sesiones» sin filtro de cajero (RN-CAJ-020). El botón dice «Nuevo» (no «Aperturar»). La clave `sesionesCaja.create.title` («Nueva Sesión de Caja») no se usa (apartado 12).

### 1.2 `/sesiones-caja/nueva` — Apertura de caja como pantalla

- **Componente:** `SesionesDetailComponent` (`SD.ts`, `SD.html`), modo **crear** (`data: { isNew: true }` de la ruta, que el componente lee en `ngOnInit`, `SD.ts:76-79`). Scope `sesiones-caja`.
- **Para qué sirve:** abrir la caja del turno a nombre del usuario conectado. La sucursal, el cajero y la hora los pone el servidor (RN-CAJ-002).
- **Cómo se llega:** botón «Nuevo» de `/sesiones-caja`; botón «Abrir caja» del tablero (`dashboard.component.html:282-287`); la URL. Desde el terminal de venta se abre el [diálogo del apartado 2](#2-diálogo-de-apertura-de-caja-aperturarcajadialogcomponent), no esta pantalla.
- **Permiso de ruta:** `POS:Leer` + `POS:AperturarCaja` (`ROUTE_ACTIONS['sesiones-caja/nueva']`, `route-permissions.ts:31`). Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Sesiones de Caja»; migas «Inicio» / «Punto de Venta» / «Sesiones de Caja» (con enlace a `/sesiones-caja`); botón volver a `/sesiones-caja` (`SD.html:3-7`, `SD.ts:69-73`). La clave «Nueva Sesión de Caja» (`sesionesCaja.create.title`) no se usa.
- **Modos:** solo crear en esta ruta. Al cargar no se consulta nada.

Sobre el formulario aparece el texto «La caja se abrirá a tu nombre, en tu sucursal activa y con la hora actual.» (`sesionesCaja.create.subtitle`).

**Campos del formulario** (`SD.ts:61-63`, `SD.html:16-41`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Editable | Valor inicial |
|---|---|---|---|---|---|---|---|
| Cajero (`sesionesCaja.fields.empleado`) | (ninguno) | Campo de texto deshabilitado con el nombre completo del usuario del token (`currentUser().fullName`) | — | — | — | No | Nombre del usuario |
| Monto Apertura (S/) (`sesionesCaja.fields.montoApertura`) | `montoAperturaEfectivo` | `matInput type="number" min="0"` | Sí | `Validators.required`, `Validators.min(0)` (`SD.ts:62`) | «Ingresa un monto válido» (`sesionesCaja.create.monto-requerido`), el mismo texto para ambos errores | Sí | 0 |

El campo no valida un máximo ni el número de decimales (coincide con la API, RN-CAJ-003). No se muestra la sucursal en pantalla.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie, a la derecha | Siempre | Nunca | `cancelar()` → navega a `/sesiones-caja` |
| Aperturar (`sesionesCaja.actions.aperturar`); con indicador de carga | Pie, a la derecha | Siempre | `saving()` | `aperturar()` (`SD.ts:103-122`): marca el formulario como tocado; si es inválido no envía; `POST /api/v1/pos/apertura` con `{ montoAperturaEfectivo }`. Éxito: aviso y navega a `/sesiones-caja` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Caja abierta correctamente | Éxito | «Sesión aperturada correctamente» (`sesionesCaja.create.save-success`) | `SD.ts:112` |
| Código `PosSesionCaja.YaAbierta` (el cajero ya tiene una caja abierta, en cualquier sucursal, RN-CAJ-004) | Error | «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (`sesionesCaja.create.ya-abierta`) | `SD.ts:23` |
| Código `Concurrency.Conflict` | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`) | `SD.ts:24` |
| Cualquier otro error | Error | «Error al aperturar la sesión» (`sesionesCaja.create.save-error`) | `SD.ts:119` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| «Aperturar» | `SesionCajaService.aperturar` (`SCS.ts:35-37`) | `POST /api/v1/pos/apertura` | `POS:AperturarCaja` (`POSController.cs:13`) |

**Comportamientos particulares:** tras abrir con éxito el SPA no navega al detalle ni al terminal: vuelve a la búsqueda (`SD.ts:113`) y deja `saving` activo hasta salir. Si la apertura falla, `saving` vuelve a falso y el formulario conserva el monto.

**Observaciones:** los errores de la API sin traducción (por ejemplo `Auth.Empleado`, el token sin empleado) caen en «Error al aperturar la sesión».

### 1.3 `/sesiones-caja/:id` — Detalle de sesión, cierre y arqueo

- **Componente:** `SesionesDetailComponent` (`SD.ts`, `SD.html`), modo **ver**. Scope `sesiones-caja`.
- **Para qué sirve:** ver los datos de una sesión de caja y, si está abierta, cerrarla declarando el efectivo contado (cierre y arqueo).
- **Cómo se llega:** enlace de la primera columna de `/sesiones-caja`; enlace «Ver mi caja» del tablero (`dashboard.component.html:273-274`).
- **Permiso de ruta:** `POS:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** igual que el apartado 1.2.
- **Modos:** ver; el id sale de la URL (`SD.ts:80-86`); sin id, navega a `/sesiones-caja`. No hay modo editar: los datos de la sesión no se modifican. El único cambio posible es el cierre, mediante el panel que se abre bajo los datos.

**Datos de la sesión** (todos en campos deshabilitados, `SD.html:63-140`; carga con indicador de carga):

| Etiqueta (clave) | Dato | Formato |
|---|---|---|
| Sucursal (`sesionesCaja.fields.sucursal`) | `sucursalNombre` | Texto |
| Cajero (`sesionesCaja.fields.empleado`) | `empleadoNombre` | Texto |
| Fecha / Hora Apertura (`sesionesCaja.fields.fechaApertura`) | `fechaHoraApertura` | `dd/MM/yyyy HH:mm` |
| Fecha / Hora Cierre (`sesionesCaja.fields.fechaCierre`) | `fechaHoraCierre` | `dd/MM/yyyy HH:mm` o «—» |
| Estado (`sesionesCaja.fields.estado`) | `estadoSesion` | «Abierta», «Cerrada» o «Cuadrada» (texto, no chip) |
| Monto Apertura (S/) (`sesionesCaja.fields.montoApertura`) | `montoAperturaEfectivo` | 2 decimales |
| Monto Cierre Declarado (S/) (`sesionesCaja.fields.montoCierreDeclarado`) | `montoCierreDeclarado` | 2 decimales o «—» si aún no hay |
| Monto Cierre Calculado (S/) (`sesionesCaja.fields.montoCierreCalculado`) | `montoCierreCalculado` | 2 decimales o «—» |
| Diferencia Arqueo (S/) (`sesionesCaja.fields.diferencia`) | `diferenciaArqueo` | 2 decimales con su signo, o «—» |

Encabezado de la tarjeta: «Detalle de Sesión» (`sesionesCaja.detail.title`) y «Datos del registro».

**Qué muestra tras cerrar.** El cierre calcula en el servidor el **efectivo esperado** (apertura + ventas en efectivo − vuelto − reembolsos en efectivo, RN-CAJ-011 a RN-CAJ-014), la **diferencia** (`declarado − esperado`) y el estado: **«Cuadrada» si la diferencia es exactamente 0, «Cerrada» con cualquier otra diferencia, sin tolerancia** (RN-CAJ-015). Al volver a abrir el detalle se ven «Monto Cierre Declarado (S/)», «Monto Cierre Calculado (S/)» (= el esperado guardado), «Diferencia Arqueo (S/)» (negativa = faltante, positiva = sobrante, sin etiqueta que lo diga; un valor positivo se muestra sin signo «+») y la fecha de cierre. **El SPA no muestra ese resultado al cerrar**: tras confirmar vuelve a la búsqueda. Antes de cerrar, la pantalla no muestra el efectivo esperado (el tablero sí lo muestra para la caja propia).

**Panel de cierre** (`SD.html:185-231`): aparece bajo los datos al pulsar «Cerrar Sesión».

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Valor inicial |
|---|---|---|---|---|---|---|
| Título del panel «Cerrar Sesión» (`sesionesCaja.cierre.title`) | — | Encabezado | — | — | — | — |
| Monto Cierre Declarado (S/) (`sesionesCaja.fields.montoCierreDeclarado`) | `montoCierreDeclarado` (`cierreForm`) | `matInput type="number" min="0"` | Sí | `Validators.required`, `Validators.min(0)` (`SD.ts:65-67`) | «Ingresa el monto declarado al cierre» (`sesionesCaja.cierre.monto-requerido`), solo cuando el campo es inválido **y** fue tocado | 0 |

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cerrar Sesión (`sesionesCaja.actions.cerrar-sesion`) | Encabezado de la tarjeta, a la derecha | `estadoSesion === Abierta` **y** `*appPermiso="'CerrarCaja'"` (= `POS:CerrarCaja`, `SD.html:76`). Se muestra a cualquier usuario con ese permiso que vea una sesión abierta, sea o no la suya (RN-CAJ-007) | Nunca | `showCierrePanel.set(true)` (muestra el panel) |
| Cancelar (`actions.cancel`) | Panel de cierre | Con el panel abierto | Nunca | Oculta el panel (no borra el monto escrito) |
| Confirmar Cierre (`sesionesCaja.actions.confirmar-cierre`); con indicador de carga | Panel de cierre | Con el panel abierto | `saving()` | `confirmarCierre()` (`SD.ts:128-146`): marca el formulario como tocado; si es inválido no envía; `POST /api/v1/pos/cierre` con `{ sesionId, montoCierre }`. Éxito: aviso y navega a `/sesiones-caja` |

No hay botón para reabrir ni para imprimir un reporte de arqueo. No hay confirmación adicional («¿estás seguro?») antes de cerrar: el panel y su botón «Confirmar Cierre» son la confirmación.

**Quién puede cerrar.** La pantalla ofrece el botón a quien tenga `POS:CerrarCaja`; el servidor solo deja cerrar al cajero que abrió la sesión o a un usuario con rol `Admin`, y rechaza a los demás con «Solo el cajero que abrió la caja o un administrador puede cerrarla.» al confirmar (RN-CAJ-007; hallazgo a3 de [05b](../05-seguridad/05b-matriz-de-permisos.md)).

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la carga de la sesión | Error | «Error al cargar la sesión de caja» (`sesionesCaja.detail.load-error`) | `SD.ts:98` |
| Sesión cerrada correctamente | Éxito | «Sesión cerrada correctamente» (`sesionesCaja.cierre.save-success`) | `SD.ts:136` |
| Código `PosSesionCaja.NoPropia` | Error | «Solo el cajero que abrió la caja o un administrador puede cerrarla.» (`sesionesCaja.cierre.no-propia`) | `SD.ts:17` |
| Código `Concurrency.Conflict` | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`) | `SD.ts:18` |
| Cualquier otro error al cerrar (incluye el efectivo esperado negativo, RN-CAJ-016, y la sesión ya cerrada) | Error | «Error al cerrar la sesión» (`sesionesCaja.cierre.save-error`) | `SD.ts:143` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir | `SesionCajaService.getById` (`SCS.ts:31-33`) | `GET /api/v1/pos/{id}` (la respuesta trae la sesión en `value`) | `POS:Leer` (`POSController.cs:38`) |
| «Confirmar Cierre» | `cerrar` (`SCS.ts:39-41`) | `POST /api/v1/pos/cierre` con `{ sesionId, montoCierre }` | `POS:CerrarCaja` (`POSController.cs:20`) |

**Observaciones:** las clases `.lote-option*` del bloque `styles` de `SD.ts:33-35` no se usan (estilos sobrantes). Los textos de arqueo con «Faltante», «Sobrante», «Cuadre Exacto» y el reporte «Reporte de Cierre de Caja» (`pos.dialogs.*`, `pos.reportDialog.*`) pertenecen a un diálogo de cierre y de reporte que ya no existe (apartado 12).

---

## 2. Diálogo de apertura de caja (`AperturarCajaDialogComponent`)

- **Componente:** `AperturarCajaDialogComponent` (`APE.ts`, `APE.html`). Scope `pos`.
- **Para qué sirve:** abrir la caja del turno sin salir del terminal de venta.
- **Cómo se abre:** solo desde `/pos/nueva`, botón «Aperturar Caja» de la tarjeta de estado de caja (`abrirCaja()`, `NV.ts:393-400`), con ancho de 450 px. El botón existe solo con `POS:AperturarCaja` (`NV.html:17`); el endpoint exige el mismo permiso. El diálogo se puede cerrar con Esc o pulsando fuera (no tiene `disableClose`); en ese caso el terminal no vuelve a comprobar la caja.
- **Título y subtítulo:** «Aperturar Caja» (`pos.checkout.aperturarCajaDialogTitle`) y «Ingresa los datos para aperturar la caja de esta sucursal y continuar con la venta.» (`pos.checkout.aperturarCajaDialogSubtitle`).

**Campos del formulario** (`APE.ts:36`, `APE.html`)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error | Editable | Valor inicial |
|---|---|---|---|---|---|---|---|
| Empleado (`pos.checkout.empleadoLabel`) | (sin control de formulario) | Campo de texto deshabilitado con el nombre completo del usuario (`cajero()`, `APE.ts:34`) | — | — | — | No | Nombre del usuario conectado |
| Monto de Apertura (S/) (`pos.checkout.montoAperturaLabel`) | `montoCtrl` | `matInput type="number" min="0" step="0.01"` | Sí | `Validators.required`, `Validators.min(0)` | «Ingresa un monto válido» (`pos.checkout.montoAperturaRequired`), mismo texto para ambos errores | Sí | 0 |

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie, a la derecha | Siempre | Nunca | `cancelar()` → cierra con `false` |
| Aperturar Caja (`pos.actions.aperturar`); con indicador de carga | Pie, a la derecha | Siempre | `saving()` | `aperturar()` (`APE.ts:38-61`): marca el monto como tocado; si es inválido no envía; `POST /api/v1/pos/apertura` con `{ montoAperturaEfectivo }`; éxito → aviso y cierra con `true` |

**Qué pasa al cerrar:** con `true`, el terminal vuelve a buscar la caja abierta y se habilita (apartado 3.1); con `false` o `undefined`, nada cambia.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Caja abierta | Éxito | «Caja aperturada correctamente» (`pos.checkout.aperturarCajaSuccess`) | `APE.ts:50` |
| Código `PosSesionCaja.YaAbierta` | Error | «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (`pos.checkout.cajaYaAbierta`) | `APE.ts:14` |
| Código `Concurrency.Conflict` | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`) | `APE.ts:15` |
| Cualquier otro error | Error | «No se pudo aperturar la caja» (`pos.checkout.aperturarCajaError`) | `APE.ts:57` |

**Llamadas a la API:** `POST /api/v1/pos/apertura` (`SesionCajaService.aperturar`), permiso `POS:AperturarCaja` (`POSController.cs:13`).

**Observaciones:** es la misma operación que la pantalla `/sesiones-caja/nueva` (apartado 1.2) con textos de otro scope. Las claves `pos.dialogs.aperturaTitle`, `aperturaSubtitle`, `montoInicialLabel`, `montoInicialRequired` y `montoInicialInvalid` («Aperturar Caja de Turno», etc.) son de una versión anterior del diálogo y no se usan (apartado 12).

---


## 3. Nueva venta: `/pos/nueva` (incluye `?continuar=<id>`)

- **Componente:** `NuevaVentaComponent` (`NV.ts`, `NV.html`, `nueva-venta.component.scss`). Providers de i18n: `provideTranslocoScope('pos')` y `provideTranslocoScope('recetas-medicas')` (`NV.ts:104`). Importa `PermisoDirective` (`NV.ts:105`).
- **Para qué sirve:** es el terminal de venta. Con la caja propia abierta, el cajero busca productos y lotes, arma el carrito, elige cliente y receta, define los pagos y cobra (se emite la boleta) o deja la venta «en espera». Con `?continuar=<id>` retoma una venta en espera para cambiarla y cobrarla o guardarla de nuevo.
- **Cómo se llega:** no está en el menú. Botón «Nueva Venta POS» de `/pos` (con `Ventas:Crear`, apartado 6), «Cobrar Venta» de `/pos/:id` (con `?continuar=<id>`, apartado 7), «Nueva Venta POS» del diálogo del comprobante (apartado 5) o la URL escrita.
- **Permiso de ruta:** `Ventas:Leer` + `Ventas:Crear`. `?continuar=` no suma `Ventas:Actualizar` (apartado 0.2).
- **Migas y cabecera:** título «Terminal de Punto de Venta (POS)» (`pos.nuevaVentaTitle`); migas «Inicio» (enlace a `/dashboard`) / «Punto de Venta» (sin enlace) / «Ventas» (enlace a `/pos`); botón volver a `/pos` (`NV.html:3-7`, `NV.ts:125-129`).
- **Modos:** dos, según la query. Sin `continuar`: **venta nueva**; se restaura el borrador local, si existe (apartado 3.9). Con `?continuar=<id>`: **retomar una venta en espera** (apartado 3.10); no se restaura el borrador genérico (`NV.ts:358-364`). Los dos comparten la misma plantilla; el modo se nota en un banner, en el texto del segundo botón y en los endpoints.
- **Efectos automáticos al abrir** (`NV.ts:358-373`): (1) cargar la venta o restaurar el borrador; (2) buscar la caja abierta propia; (3) leer la configuración de la empresa (`GET ventas/configuracion`); (4) leer los nombres de las condiciones de venta; (5) lanzar la primera búsqueda de productos con el texto vacío.

La pantalla se describe por zonas, en el orden en que el cajero las usa. Antes de la primera zona se evalúa un único bloque de arranque (`NV.html:9-24`):

| Estado | Qué se ve |
|---|---|
| `checkingSesion()` o `cargandoContinuacion()` verdadero | Solo un indicador de carga (`mat-spinner`, 40 px) bajo la cabecera |
| Terminada la comprobación y sin caja propia abierta | Tarjeta de aviso «Estado de caja» (apartado 3.1); no se ve nada más |
| Con caja propia abierta | El terminal: banner de continuación (si aplica) y dos columnas (A: búsqueda o escaneo; B: cliente, carrito y pago) |

### 3.1 Zona: estado de caja

Se decide con `checkSesionCaja()` (`NV.ts:376-391`): toma `sucursalId` y `empleadoId` del token (`authService.sucursalId()` y `empleadoId()`) y llama `GET /api/v1/POS/sesiones` con `sucursalId`, `empleadoId`, `estadoSesion=0` (Abierta), `pageNumber=1`, `pageSize=1`. Si vuelve una sesión, guarda su `id` en `sesionId` y el terminal queda habilitado. Se necesita la **caja propia, de la sucursal del token**: una caja abierta por otro cajero no sirve.

| Situación | Qué muestra | Fuente |
|---|---|---|
| Hay caja propia abierta | El terminal completo. El terminal **no muestra** datos de la sesión (hora de apertura, monto inicial ni efectivo esperado) | `NV.html:24-848` |
| No hay caja abierta | Tarjeta con el texto «Debes aperturar la caja del turno antes de procesar una venta.» (`pos.checkout.cajaRequeridaAviso`, respeta saltos de línea) y, solo con `POS:AperturarCaja`, el botón «Aperturar Caja» (`pos.actions.aperturar`) | `NV.html:13-23` |
| El token no trae `sucursalId` o `empleadoId` | Igual que «no hay caja abierta» (no se hace la búsqueda y se apaga el indicador de carga) | `NV.ts:379-382` |
| Falla la búsqueda de la caja (por ejemplo, el rol no tiene `POS:Leer`) | Igual que «no hay caja abierta» y **sin ningún aviso de error** (`error: () => this.checkingSesion.set(false)`) | `NV.ts:388` |

Qué se bloquea sin caja: todo el terminal. No se renderizan ni el catálogo, ni el carrito, ni los pagos; no hay forma de vender ni de dejar una venta en espera. Si el rol no tiene `POS:AperturarCaja`, la tarjeta solo informa y no ofrece botón. Con el mensaje «Debes aperturar...» y una falla de `POS:Leer`, el cajero que sí tiene una caja abierta puede ver el aviso y, al pulsar «Aperturar Caja», recibir «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (apartado 2).

Cómo se abre desde aquí: el botón «Aperturar Caja» ejecuta `abrirCaja()` (`NV.ts:393-400`), que abre el [diálogo de apertura](#2-diálogo-de-apertura-de-caja-aperturarcajadialogcomponent) (450 px). Si el diálogo se cierra con éxito (`true`), vuelve a ejecutar `checkSesionCaja()` y el terminal aparece; si se cancela o falla, la tarjeta sigue igual.

### 3.2 Zona: búsqueda de productos (columna A, pestaña «Producto»)

La columna A tiene un panel con dos pestañas solo si el usuario tiene `Recetas:Analizar` (`puedeEscanearReceta`, `NV.ts:150`, `NV.html:36-52`): «Producto» (`pos.checkout.tabBuscar`, icono `search`) y «Receta» (`pos.checkout.tabReceta`, icono `document_scanner`). Sin ese permiso no aparece el selector de pestañas y solo existe la búsqueda de productos. La pestaña activa empieza en «Producto» y vuelve a ella al limpiar la venta (`resetVenta`, `NV.ts:1175`).

**Campo de búsqueda** (`NV.html:54-63`)

| Etiqueta (clave) | Control | Tipo | Comportamiento | Valor inicial |
|---|---|---|---|---|
| «Buscar producto...» (`pos.checkout.searchProductPlaceholder`; es la etiqueta flotante) | `productoNombre` (`catalogoForm`) | `matInput` de texto, con icono de lupa a la derecha (`matSuffix`) | Busca mientras se escribe, con retardo de 300 ms y sin repetir el mismo texto (`NV.ts:276-306`); solo busca por **nombre comercial** (`nombreComercial`) | Vacío (se lanza igualmente una búsqueda inicial) |

**Chips de condición de venta** (`NV.html:65-82`): una fila de botones que filtra por condición. Primero «Todos» (`pos.checkout.condicionTodos`; sin filtro) y después un botón por cada nombre que devuelve `GET /api/v1/medicamentos/condiciones-venta` (`NV.ts:371`). Los nombres vienen del backend, en este orden: «Venta Libre (OTC)», «Receta Simple», «Receta Retenida», «Estupefaciente» (`sofia-apis/src/SOFIA.Domain/Enums/CondicionVenta.cs:7-10`). El filtro se envía como el **índice** del botón (0 a 3) en `condicionVenta`; coincide con el enumerado `CondicionVenta` (`VM.ts:16-21`) mientras el backend conserve ese orden. Pulsar un chip repite la búsqueda con el texto actual (`setCondicionFiltro`, `NV.ts:572-576`). El chip activo se marca con la clase `active`.

**Qué busca y cómo** (`NV.ts:279-300`): `GET /api/v1/medicamentos` con `pageNumber=1`, `pageSize=12` y los parámetros `nombreComercial` (texto), `condicionVenta` (chip), `incluirStock=true`, `soloStockVendibleEnMiSucursal=true` («la insignia de stock debe coincidir con los lotes ofrecidos: sucursal propia, solo stock vendible», comentario `NV.ts:285-286`) y `ordenarPorMasVendidos=true` solo cuando no hay texto ni chip (`NV.ts:288`: sin filtros se muestran primero los productos más vendidos). Se muestran como máximo 12 productos. Si la búsqueda falla se muestra «No se pudo cargar el catálogo de medicamentos. Inténtalo nuevamente.» (`pos.messages.catalogoError`) y la búsqueda sigue funcionando para el siguiente texto (el error se atrapa dentro del `switchMap`, `NV.ts:292-297`).

**Tarjeta de producto** (`NV.html:84-131`), una por producto, botón completo que se deshabilita si `stockTotal` es 0 o nulo:

| Dato | Texto | Fuente |
|---|---|---|
| Nombre | `nombreComercial` | `NV.html:93-95` |
| Subtítulo | «{laboratorio} · {código nacional}» (`laboratorioNombre`, `codigoNacional`) | `NV.html:96-98` |
| Aviso de receta | Si la condición no es «Venta Libre (OTC)»: icono `description` y «Requiere receta» (`pos.checkout.requiereReceta`); el tooltip (`title`) es el nombre de la condición (por ejemplo «Receta Retenida») | `NV.html:99-106`, `NV.ts:589-591` |
| Precio | «S/ {precioVentaBase} c/u» con 2 decimales (`pos.checkout.precioCadaUno` = «c/u»), o «Sin precio» (`pos.checkout.sinPrecioCatalogo`) si el producto no tiene precio de catálogo | `NV.html:107-114` |
| Insignia de stock | Según `stockTotal` (stock vendible de la sucursal): 0 o nulo → «Agotado» (`pos.checkout.stockAgotado`); de 1 a 10 → «Stock bajo» (`stockBajo`); más de 10 → «Disponible» (`stockDisponible`) (`stockBadgeClass`/`stockBadgeLabel`, `NV.ts:578-587`) | `NV.html:115-128` |
| Icono `+` | Solo si hay stock | `NV.html:123-127` |

**Sin resultados:** «No se encontraron medicamentos o lotes disponibles.» (`pos.checkout.noCatalogData`, `NV.html:132-136`).

**Fecha de vencimiento: el terminal no la muestra** (el DTO `LoteDisponibleVenta.fechaCaducidad` se recibe pero ninguna plantilla la pinta; búsqueda de «caduc» en `NV.html` y `NV.ts` sin resultados). El filtro de lotes vencidos lo hace el backend (la lista solo trae lotes no vencidos, RN-VEN-075 en [01c2](../01-reglas-de-negocio/01c2-ventas-y-comprobantes.md)).

**«Sin stock en la sucursal» y «disponible en otra sede»:** el terminal **no** informa si hay stock en otra sucursal ni ofrece pedir una transferencia. Los textos `pos.messages.noStockLocal`, `noStockAnywhere`, `otherBranchAvailableInfo`, `transferRequestedSuccess`, `transferRequestedError` y todo el grupo `pos.transferDialog.*` existen en `es.json` pero ningún archivo los usa (apartado 12). Un producto sin stock vendible en la sucursal aparece (si el filtro lo devuelve) como tarjeta deshabilitada con la insignia «Agotado»; el filtro `soloStockVendibleEnMiSucursal` hace que, en la práctica, la lista solo traiga productos con stock vendible (`NV.ts:286`; semántica exacta en el backend, «No verificado»).

**Elegir un producto y un lote** (`NV.html:138-164`, `NV.ts:593-618`):

1. Pulsar la tarjeta (`selectMedicamento`) llama `GET /api/v1/ventas/lotes-disponibles?productoId=<id>`. Si falla: lista de lotes vacía y «No se pudieron cargar los lotes disponibles. Inténtalo nuevamente.» (`pos.messages.lotesError`; o el mensaje del código si está mapeado, apartado 3.12).
2. Aparece la etiqueta «Lote:» (`pos.checkout.loteLabel`) y una fila por lote con icono `medication`, el **número de lote** y «Stock Disp: {cantidadFisica}» (`pos.checkout.stockAvailable` + cantidad física del lote en la sucursal). La fila se marca activa al elegirla; el icono `+` indica que se puede elegir.
3. Si no vuelve ningún lote para el producto elegido: «No se encontraron medicamentos o lotes disponibles.» (`NV.html:160-164`).
4. Pulsar un lote (`selectLote`) lo deja seleccionado, pone `cantidad = 1` y `precio = precioVentaBase del lote` (o 0 si es nulo), vacía la presentación y llama `GET /api/v1/ventas/presentaciones?productoId=<id>`. Si falla: sin presentaciones y «No se pudieron cargar las presentaciones del producto. Inténtalo nuevamente.» (`pos.messages.presentacionesError`).

**Formulario de agregado** (`addForm`, `NV.html:166-238`; aparece cuando hay un lote seleccionado)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio / validaciones | Mensaje | Editable | Ayuda |
|---|---|---|---|---|---|---|
| (título, sin etiqueta) | — | Texto en negrita con `productoNombre` del lote | — | — | No | — |
| «Presentación» (`pos.checkout.presentacionLabel`) | (sin `formControlName`; usa el signal `selectedPresentacionId`) | `mat-select` | Opcional. Solo aparece si el producto tiene al menos una presentación de venta | — | Sí | Opciones: «Unidad» (`pos.checkout.presentacionUnidad`, venta por unidad base) y, por cada presentación, «{descripcion} (S/ {precioVenta})» con 2 decimales |
| «Cantidad» (`pos.checkout.cantidadLabel`; con presentación elegida `pos.checkout.cantidadPresentacionLabel`, que también dice «Cantidad») | `cantidad` | `matInput type="number"`, `min="1"`, `max` = existencia del lote (o existencia ÷ unidades base de la presentación, redondeada hacia abajo, `floorDiv`) | Reglas reales al pulsar «Agregar» en el apartado 3.3 | Avisos del apartado 3.3 | Sí | Valor inicial 1 |
| «Precio Unitario (S/)» (`pos.checkout.precioLabel`; con presentación `pos.checkout.precioPresentacionLabel` = «Precio de la Presentación (S/)») | `precio` | `matInput type="number"` **de solo lectura** (`readonly`) | — | — | **No** (viene del catálogo o de la presentación) | Valor inicial: `precioVentaBase` del lote |

**De dónde salen las opciones de presentación y cómo cambian precio y equivalencia** (`onPresentacionChange`, `NV.ts:624-629`; `NV.ts:631-664`): las opciones salen de `GET ventas/presentaciones` (presentaciones de venta del producto: `descripcion`, `cantidadUnidadesBase`, `precioVenta`, `VM.ts:218-223`). Al elegir una presentación, la cantidad vuelve a 1 y el precio pasa a ser el `precioVenta` de la presentación (por ejemplo «Caja x10» a S/ 45.00); al elegir «Unidad», el precio vuelve a `precioVentaBase` del lote. Al agregar, el terminal convierte a **unidades base**: `cantidadBase = cantidadIngresada × cantidadUnidadesBase` y `precioUnitarioBase = precioIngresado ÷ cantidadUnidadesBase`; el carrito y el backend trabajan en unidades base (comentario `NV.ts:640-641`). La equivalencia se ve luego en el carrito como «{n} × {presentación}» (apartado 3.3).

**Botón «Agregar»** (`pos.actions.agregar`, icono `add_shopping_cart`, ancho completo, `NV.html:229-236`): ejecuta `agregarAlCarrito()`. Después de agregar con éxito se limpia la selección: lote, producto, lista de lotes y presentaciones (`NV.ts:659-663`).

**Teclado:** el formulario de agregado es un `<form>` cuyo único botón no tiene `type` (por tanto es de tipo «submit» por defecto del navegador): pulsar Enter dentro de «Cantidad» debería hacer clic en «Agregar». Es comportamiento estándar del navegador deducido de la plantilla; **No verificado** en ejecución.

### 3.3 Zona: carrito (columna B, panel «Resumen de Venta»)

Encabezado del panel (`NV.html:515-524`): icono `shopping_cart`, «Resumen de Venta · {n}» (`pos.checkout.cartTitle` y el número de líneas) y, solo si hay líneas, el enlace «Vaciar» (`pos.checkout.vaciarCart`). Con el carrito vacío se ve el icono y «El carrito está vacío.» (`pos.checkout.emptyCart`) y no se ven los totales.

**Cada línea del carrito** (`NV.html:532-580`; no es una tabla, es una lista de filas):

| Elemento | Qué muestra | Fuente |
|---|---|---|
| Icono | `medication` | `NV.html:535` |
| Nombre | `productoNombre` | `NV.html:539-541` |
| Presentación (solo si la línea se vendió por presentación) | «{cantidadEnPresentacion} × {descripción}», por ejemplo «2 × Caja x10» | `NV.html:542-547` |
| Lote y precio | «{número de lote} · S/ {precioUnitario} c/u». El precio es **por unidad base** (2 decimales); en una línea por presentación es el precio de la presentación ÷ unidades base | `NV.html:548-552` |
| Cantidad | Botón «−», la cantidad **en unidades base** (`line.cantidad`) y botón «+» | `NV.html:554-566` |
| Total de la línea | «S/ {cantidad × precioUnitario}» con 2 decimales | `NV.html:567-572` |
| Quitar | Botón con icono `close` | `NV.html:573-577` |

**Cambio de cantidad** (`ajustarCantidadCarrito`, `NV.ts:814-830`): los botones «−» y «+» suman o restan un **paso**: 1 unidad base, o las unidades base de la presentación si la línea se vendió por presentación (por ejemplo, de 10 en 10 para «Caja x10»); en ese caso también ajustan en 1 la cantidad en presentaciones. No hay campo para escribir la cantidad en el carrito. No se puede bajar de un paso: pulsar «−» en la cantidad mínima no hace nada y no avisa (`NV.ts:821`). Subir por encima de la existencia del lote cargada al agregar (`cantidadDisponible`) muestra «Stock máximo disponible alcanzado ({{stock}} un.).» y deja la cantidad como estaba (`NV.ts:822-825`).

**Agregar el mismo lote y presentación otra vez** suma a la línea existente en vez de crear otra: la clave de línea es «lote + presentación» (`pushCartLine`, `NV.ts:673-695`). Si la suma supera la existencia del lote se muestra el mismo aviso de stock máximo y la cantidad queda en la existencia. **Un mismo producto con lotes distintos, o con y sin presentación, genera líneas separadas.**

**Quitar y vaciar:** «×» elimina la línea (`quitarDelCarrito`) y «Vaciar» elimina todas (`vaciarCarrito`), ambos sin pedir confirmación. Cuando el carrito queda vacío se reinician los pagos a un único pago en efectivo de 0 y se vuelve al modo de pago simple (`resetPagos`, `NV.ts:832-846`) para que los montos escritos no pasen a la siguiente venta.

**Totales** (`NV.html:601-634`, solo con el carrito no vacío; fórmulas en `NV.ts:226-244`). Los precios del catálogo ya incluyen IGV (RN-CMP-006 en [01c2](../01-reglas-de-negocio/01c2-ventas-y-comprobantes.md)); la pantalla separa el IGV con la tasa fija de 18 % escrita en el código:

| Etiqueta (clave) | Cálculo en pantalla | Formato |
|---|---|---|
| «Subtotal (Op. Gravada):» (`pos.checkout.subtotalLabel`) | `subtotalSinIgv = subtotalBruto ÷ 1.18`, donde `subtotalBruto = Σ (cantidad × precioUnitario)` de las líneas | «S/ x.xx» (pipe `number` de 2 decimales, formato según el idioma activo) |
| «IGV (18%):» (`pos.checkout.igvLabel`) | `igv = subtotalBruto − subtotalSinIgv` | igual |
| «Monto Pendiente:» (`pos.checkout.montoPendienteLabel`) **o** «Vuelto a Entregar:» (`pos.checkout.vueltoDisponibleLabel`) | `montoPendiente = redondeo a centavos de (montoAPagar − Σ pagos)`. Si es mayor que 0 se muestra «Monto Pendiente:» con ese valor; si es 0 o negativo se muestra «Vuelto a Entregar:» con el valor positivo (`−montoPendiente`) | igual |
| «TOTAL A PAGAR:» (`pos.checkout.totalLabel`) | `montoAPagar = subtotalBruto − montoCubiertoSeguro`; la cobertura es 0 mientras el bloque de aseguradora esté oculto | «S/ x.xx» |

`subtotalSinIgv` e `igv` no se redondean antes de mostrarse: cada cifra se redondea sola al pintarla, así que «Subtotal + IGV» puede diferir en un centavo de «TOTAL A PAGAR» (derivado de la lectura; **No verificado** en ejecución). Solo `montoPendiente` y el vuelto se redondean a centavos (`roundMoney`, `VUELTO.ts:1`). No hay descuentos (RN-VEN-023).

**Límites y validaciones al agregar y ajustar** (el servidor vuelve a validar todo; RN-VEN-024 describe el mismo orden):

| Orden | Situación | Dónde se ve | Mensaje exacto (clave) | Fuente |
|---|---|---|---|---|
| 1 | `cantidad × unidades base` supera `cantidadFisica` del lote al pulsar «Agregar» | Snackbar de error | «Stock máximo disponible alcanzado ({{stock}} un.).» con `stock` = existencia del lote (`pos.messages.stockMaxReached`) | `NV.ts:645-648` |
| 2 | Precio ingresado ≤ 0 (producto sin precio en el catálogo) | Snackbar de error | «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» (`pos.messages.sinPrecio`) | `NV.ts:649-652` |
| 3 | Cantidad ingresada ≤ 0 (o vacía) | Snackbar de error | «Ingresa una cantidad válida para agregar el producto.» (`pos.messages.cantidadPrecioInvalidos`) | `NV.ts:653-656` |
| — | «+» por encima de `cantidadDisponible` | Snackbar de error | «Stock máximo disponible alcanzado ({{stock}} un.).» con `stock` = `cantidadDisponible` de la línea | `NV.ts:822-825` |
| — | Repetir lote y presentación y superar la existencia | Snackbar de error y la cantidad queda en la existencia | «Stock máximo disponible alcanzado ({{stock}} un.).» | `NV.ts:677-681` |

No hay tope en el número de líneas ni en la cantidad de unidades del carrito aparte de la existencia de cada lote. Se admiten cantidades decimales (la validación solo exige mayor que 0). El lote vencido, en cuarentena o sin stock lo filtra el backend al listar lotes; si aun así un lote venció antes de cobrar, el servidor responde `Venta.Lote.Vencido` («Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.», apartado 3.12).

### 3.4 Zona: cliente (columna B, panel «Datos del Cliente»)

Encabezado: icono `person` y «Datos del Cliente» (`pos.checkout.clienteTitle`) (`NV.html:379-383`).

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Comportamiento | Valor inicial |
|---|---|---|---|---|---|
| «Buscar paciente por nombre o documento...» (`pos.checkout.clienteSearchPlaceholder`) | `clienteNombre` (`clienteForm`) | `matInput` con `matAutocomplete` y lupa (`matSuffix`) | **No.** El SPA nunca exige cliente | Retardo de 300 ms. Si el texto son solo dígitos busca por documento (`docIdentidadGub`), si no por `nombreApellidos`; hasta 20 resultados. Cada opción se muestra «{nombre} · {tipo de documento} {número}», por ejemplo «DNI 45781236» (tipos: «DNI», «Carné de extranjería», «RUC», «Pasaporte», «Sin documento»; un paciente antiguo sin tipo muestra solo el número) | Vacío |

- **Venta sin cliente:** debajo del campo se lee «Cliente Eventual (Público General)» (`pos.checkout.clienteEventual`, `NV.html:407-410`). La boleta sale a nombre del cliente eventual (RN-CMP-005, RN-VEN-025). Los textos del tipo de documento vienen de `tipos-documento.*` de la raíz (`tipoDocumentoKey`, `app/shared/utils/tipo-documento.util.ts:16-18`).
- **Búsqueda sin resultados:** no hay opción «Sin resultados» (la clave `pos.checkout.clienteAutocompleteNoResults` no se usa): la lista simplemente queda vacía.
- **Con cliente elegido** (`NV.html:411-454`): debajo del campo aparece su documento («DNI 45781236») y el selector de receta (apartado 3.5). Al elegirlo (`selectCliente`, `NV.ts:861-865`) se cargan sus recetas.
- **Cuándo es obligatorio en la práctica:** el SPA no lo exige. Hace falta elegir cliente para poder elegir una receta registrada (el selector solo aparece con cliente) y, por tanto, para vender productos de receta retenida o estupefaciente, que siempre exigen receta registrada (apartado 3.5; RN-VEN-026, RN-VEN-029). Los productos de venta libre se venden sin cliente.
- **Cambiar de cliente:** al **escribir** otro texto se descarta el cliente elegido y su receta (`selectedCliente.set(null)` y `limpiarRecetasCliente()` dentro del `switchMap`, `NV.ts:312-316`). **No hay botón para quitar el cliente**: vaciar por completo el campo no lo descarta, porque con el texto vacío el flujo termina antes de limpiar la selección (`if (!term ...) return of(...)`, `NV.ts:313`); el cliente solo se descarta al escribir algo distinto o al limpiar la venta. El método `limpiarCliente()` existe pero ninguna plantilla lo invoca (derivado de la lectura; **No verificado** en ejecución).
- **No se puede registrar un paciente desde el terminal:** no hay enlace a `/pacientes/nueva`.

### 3.5 Zona: receta (aviso, selector, número y verificación)

**Selector «Receta médica»** (`NV.html:417-441`, solo con cliente elegido):

| Etiqueta (clave) | Control | Obligatorio | Contenido | Valor inicial |
|---|---|---|---|---|
| «Receta médica» (`pos.checkout.recetaLabel`) | `mat-select` ligado al signal `selectedRecetaId` | Depende de los productos del carrito (tabla siguiente) | Primera opción «Sin receta» (`pos.checkout.sinReceta`, valor vacío); después una opción por receta registrada del cliente: «{fecha de expedición dd/MM/yyyy} · {médico} · N.° {número}» (`pos.checkout.recetaNumero` = «N.° {{numero}}», solo si la receta tiene número). Hasta 50 recetas (`GET recetas?clienteId=…&pageNumber=1&pageSize=50`, `NV.ts:873-879`) | «Sin receta» |
| Ayuda bajo el campo (`mat-hint`) | Texto | — | «Receta retenida y estupefacientes siempre exigen una receta registrada con su número.» (`pos.checkout.recetaHint`) | — |

- **Enlace «Registrar receta»** (`pos.checkout.registrarReceta`, icono `open_in_new`): botón de texto bajo el selector, visible con `Recetas:Crear` (`*appPermiso="'Recetas:Crear'"`, `NV.html:442-453`). Abre `/recetas-medicas/nueva` **en otra pestaña** (`target="_blank"`, `rel="noopener"`) sin datos precargados. Al volver a la pestaña del terminal (evento `window:focus`) se recargan las recetas del cliente conservando la receta elegida (`recargarRecetasCliente`, `NV.ts:881-886`), así la receta recién registrada aparece en la lista.
- **La receta elegida se adjunta a todas las líneas del carrito**, también a las de venta libre (`buildDetalleRequests`, `NV.ts:987-994`; RN-VEN-036). El backend comprueba que exista y que sea del cliente de la venta (RN-VEN-030, RN-VEN-031).
- **Número de receta para productos controlados:** el terminal **no tiene un campo para escribir el número**. El número es un dato de la receta registrada (`numeroReceta`); el terminal solo lo muestra en el selector y lo exige (apartado siguiente). Para agregarlo hay que completar o corregir la receta en su propia pantalla (RN-VEN-032).

**Qué avisos y bloqueos genera el contenido del carrito** (`NV.ts:186-212`, `NV.html:582-599`). Cuenta como «línea con receta» toda línea cuya `condicionVenta` se conoce y no es «Venta Libre (OTC)»; «controlada» es «Receta Retenida» o «Estupefaciente». Las líneas de un borrador antiguo sin condición guardada se consideran desconocidas: el terminal no las bloquea y decide el servidor (`NV.ts:56-57`). El valor `exigirRegistroReceta` sale de `GET ventas/configuracion` (la empresa); mientras no se conoce, o si la consulta falla, se asume **estricto** (`exigirRegistroReceta = true`, `NV.ts:187,367-370`).

| # | Carrito | Receta elegida | `exigirRegistroReceta` | Qué se ve en el carrito | «Procesar Venta» |
|---|---|---|---|---|---|
| 1 | Solo venta libre | cualquiera | cualquiera | Nada | No se bloquea por receta |
| 2 | Hay receta simple y ninguna controlada | Ninguna | `true` (la empresa exige registro) | Icono `description` y «Hay productos que requieren una receta registrada: selecciona al cliente y su receta antes de cobrar.» (`pos.checkout.recetaPendienteHint`) | Bloqueado |
| 3 | Hay receta simple y ninguna controlada | Ninguna | `false` (empresa flexible) | Casilla «Receta verificada» (`pos.checkout.recetaVerificadaLabel`) con el texto «Confirmas que revisaste la receta física del cliente. Quedará registrado a tu nombre.» (`pos.checkout.recetaVerificadaHint`) | Bloqueado hasta marcar la casilla |
| 4 | Hay alguna controlada | Ninguna | cualquiera | Icono `description` y «Hay productos de receta retenida o estupefacientes: selecciona al cliente y una receta registrada que tenga número de receta.» (`pos.checkout.recetaControladaHint`) | Bloqueado |
| 5 | Hay alguna controlada | Receta **sin** número | cualquiera | «La receta seleccionada no tiene número de receta: agrégalo en la receta para vender productos de receta retenida o estupefacientes.» (`pos.checkout.recetaSinNumeroHint`) | Bloqueado |
| 6 | Hay alguna controlada | Receta con número | cualquiera | Nada | No se bloquea por receta |
| 7 | Solo receta simple | Cualquier receta | cualquiera | Nada (la receta sin número no se avisa si no hay controladas) | No se bloquea por receta |

Casilla «Receta verificada» (`mat-checkbox`, `NV.html:582-593`): aparece **solo** en el caso 3 (`requiereVerificacionReceta`: empresa flexible, sin receta elegida, sin línea controlada y con alguna línea de receta). Se desmarca sola en cuanto el carrito deja de necesitarla (`effect`, `NV.ts:352-355`), no se guarda en el borrador local y se desmarca al terminar la venta. Solo se envía al **cobrar** (`recetaVerificada` en `POST ventas` o `PUT ventas/{id}/completar`); **dejar la venta en espera no la pide** (`NV.ts:1079-1080`; `puedeGuardarPendiente` no mira la receta). Si el servidor responde `Venta.Receta.VerificacionRequerida` y el terminal no la había mostrado (regla vieja o borrador antiguo), pasa a modo flexible y muestra la casilla (`mostrarVerificacionSiElServidorLaPide`, `NV.ts:1145-1150`). La verificación queda registrada a nombre de quien cobra (RN-VEN-034, RN-VEN-035); se ve luego en el detalle de la venta (apartado 7).

El bloque de aviso solo se renderiza con el carrito no vacío, entre la lista de líneas y los totales.

### 3.6 Zona: escaneo de receta (columna A, pestaña «Receta»)

Solo existe con `Recetas:Analizar` (`NV.ts:150`). Es una ayuda para **agregar al carrito** los medicamentos que la receta indica; **no** asocia ninguna receta registrada a la venta (para eso está el selector del apartado 3.5).

| Elemento | Texto (clave) | Comportamiento | Fuente |
|---|---|---|---|
| Zona de carga | Icono `document_scanner` y «Captura o selecciona una imagen para comenzar» (`recetasMedicas.scanner.empty-hint`); con imagen elegida, su vista previa (texto alternativo «Receta», `pos.checkout.tabReceta`); mientras interpreta, `mat-spinner` y «Interpretando...» (`recetasMedicas.scanner.interpreting`) | Acepta arrastrar y soltar una imagen (solo archivos cuyo tipo empieza por `image/`); resalta al arrastrar; ignora soltar mientras interpreta | `NV.html:254-276`, `NV.ts:731-747` |
| Botón «Tomar foto» (icono `photo_camera`; `recetasMedicas.scanner.btn-camera`) | — | Abre un selector de archivo con `accept="image/*"` y `capture="environment"` (cámara trasera en móviles). Deshabilitado mientras interpreta | `NV.html:240-246,279-285` |
| Botón «Galería» (icono `photo_library`; `recetasMedicas.scanner.btn-gallery`) | — | Selector de archivo `image/*`. Deshabilitado mientras interpreta | `NV.html:247-252,286-292` |
| Botón «Interpretar receta» (icono `auto_awesome`; `recetasMedicas.scanner.btn-interpret`) | — | Aparece cuando hay imagen y no se está interpretando. `interpretarReceta()` → `POST /api/v1/recetas/analizar` (multipart, permiso `Recetas:Analizar`, política de límite `AiPolicy`) | `NV.html:295-304`, `NV.ts:749-764` |

Elegir otra imagen reinicia los resultados y los marcados como agregados (`readSelectedFile`, `NV.ts:715-722`); **no hay botón «Limpiar»** en esta pestaña (la clave `recetasMedicas.scanner.btn-clear` no se usa aquí). La imagen y los resultados se descartan al terminar o limpiar la venta (`limpiarReceta` desde `resetVenta`).

**Resultados** (panel aparte bajo la pestaña, `NV.html:308-374`, solo en la pestaña «Receta» y después de interpretar):
- Banner con icono `auto_awesome`: «{n} medicamento(s) detectado(s) en la receta» (`pos.checkout.recetaDetectedBanner`). **`n` cuenta todos los ítems devueltos**, también los de tipo «Recomendacion», que el terminal no lista (el getter `itemsSugerencia` existe pero la plantilla no lo usa).
- Una fila por ítem de tipo «Receta»: nombre oficial; botón circular `+` (cambia a `✓` una vez agregado) solo si el ítem tiene stock; insignia «DIGEMID» (icono `verified`) si está validado por DIGEMID; insignia de confianza «Alta» (≥ 0,85), «Media» (≥ 0,65) o «Baja» (`recetasMedicas.scanner.confidence-high|medium|low`); insignia «En stock» (`recetasMedicas.scanner.stock-available`, icono `check_circle`) o «Sin stock» (`recetasMedicas.scanner.no-stock`, icono `cancel`).
- Si no se detectó nada: «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» (`recetasMedicas.scanner.no-results`). Los equivalentes terapéuticos que devuelve el análisis no se muestran en el terminal.
- **Agregar un ítem** (`agregarItemDesdeReceta`, `NV.ts:790-811`): consulta `GET ventas/lotes-disponibles?productoNombre=<nombre oficial>` y toma el **primer lote** de la lista (el que vence antes). Sin lotes: «El medicamento detectado no tiene stock en esta sucursal.» (`pos.messages.noStockLocalReceta`). Lote sin precio de catálogo: «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» (`pos.messages.sinPrecio`). Si todo va bien agrega al carrito `mínimo(cantidad sugerida o 1; existencia del lote)` unidades base, sin presentación y a precio de catálogo, y marca el ítem como agregado; un ítem ya agregado no se vuelve a agregar. Si la consulta de lotes falla: «No se pudieron cargar los lotes disponibles. Inténtalo nuevamente.» (`pos.messages.lotesError`).
- Error al interpretar: «Error al interpretar la receta. Intenta nuevamente.» (`recetasMedicas.scanner.error`, `NV.ts:761`).

### 3.7 Zona: pagos (columna B, panel «Método de Pago»)

Encabezado: icono `payments` y «Método de Pago» (`pos.checkout.pagoTitle`, `NV.html:638-642`). Estado inicial: **un solo pago en efectivo por 0** (`pagos = [{ Efectivo, monto: 0, referencia: '' }]`, `NV.ts:222`), en modo simple.

**Métodos disponibles** (`metodoPagoOpciones`, `NV.ts:123`; texto por `metodoPagoLabel`, `NV.ts:979-982`; valores del enumerado en `VM.ts:8-13`):

| Valor | Texto exacto (clave) | Icono de la ficha | ¿Pide número de operación? |
|---|---|---|---|
| 0 `Efectivo` | «Efectivo» (`pos.checkout.efectivo`) | `payments` | No |
| 1 `YapePlin` | «Yape / Plin» (`pos.checkout.yapePlin`) | `smartphone` | Sí |
| 2 `Tarjeta` | «Tarjeta Débito/Crédito» (`pos.checkout.tarjeta`) | `credit_card` | Sí |
| 3 `Transferencia` | «Transferencia» (`pos.checkout.transferencia`) | `account_balance` | Sí |

**Modo simple** (`!modoPagoAvanzado()`, `NV.html:644-744`): cuatro fichas (una por método); la activa se marca. Pulsar una ficha (`seleccionarMetodoPagoSimple`, `NV.ts:916-918`) deja **un único pago** de ese método con el monto igual al total a pagar en ese instante y sin número de operación. Debajo, según el método:

| Método | Campo | Control | Notas |
|---|---|---|---|
| Efectivo | «Recibido (S/)» (`pos.checkout.efectivoRecibidoLabel`) | Número, `min="0"`, `step="0.01"`; escribe en `pagos[0].monto` | El monto recibido; el pago registrado es lo recibido, no lo neto (el vuelto lo calcula el servidor al cobrar, RN-VEN-044). Valor inicial 0 |
| Efectivo | «Vuelto: S/ x.xx» (`pos.checkout.vueltoCortoLabel`) | Caja de solo lectura con `cambioAEntregar` | Ver cálculo del vuelto abajo |
| Yape / Plin, Tarjeta o Transferencia | «Monto (S/)» (`pos.checkout.montoPagoLabel`) | Número, `min="0"`, `step="0.01"` | Se rellena con el total a pagar al pulsar la ficha |
| Yape / Plin, Tarjeta o Transferencia | «Nro Operación» (`pos.checkout.referenciaLabel`), con marcador «N° operación» (`pos.checkout.referenciaPlaceholder`) | Texto, `required`, `maxlength="100"` (sin marca de obligatorio) | Obligatorio para cobrar; el SPA no valida el formato |

Debajo de las fichas, el enlace «+ Dividir en varios métodos de pago» (`pos.checkout.dividirPagoBtn`) pasa al modo dividido (`activarPagoAvanzado`, no cambia los pagos).

**El monto de un pago no se vuelve a ajustar solo** cuando cambia el carrito: en efectivo es lo que escriba el cajero; en los demás métodos es el total del momento en que se pulsó la ficha. Si el carrito cambia después, «Monto Pendiente:» o «Vuelto a Entregar:» mostrarán la diferencia y el cobro seguirá bloqueado hasta corregir el monto o pulsar la ficha otra vez (derivado de la lectura; **No verificado** en ejecución).

**Modo dividido** (`modoPagoAvanzado()`, `NV.html:745-808`): una ficha por pago con un `<select>` nativo de método (los cuatro textos), un campo numérico de monto (`min="0"`, `step="0.01"`), un campo de texto para el número de operación con marcador «N° operación» (solo si el método no es efectivo; se marca en rojo mientras falta) y una «×» para quitar el pago (solo si hay más de uno). Después de las fichas, el botón «Agregar Método de Pago» (`pos.checkout.agregarPagoBtn`, icono `add`) y el enlace «Volver a pago simple» (`pos.checkout.volverPagoSimpleBtn`).
- **Agregar un pago** (`agregarPago`, `NV.ts:946-951`): usa el primer método que aún no esté en la lista (si ya están los cuatro, efectivo) y rellena el monto con lo pendiente (nunca negativo).
- **Quitar un pago** (`quitarPago`): elimina la fila; con un solo pago no hace nada.
- **Cambiar el método de una fila** (`updatePago`, `NV.ts:964-977`): si el método elegido ya existe en otra fila, las dos se **fusionan** en una: se suman los montos y se conserva el primer número de operación no vacío. El terminal no deja dos filas del mismo método.
- **Volver a pago simple** (`volverAPagoSimple`, `NV.ts:928-931`): deja un único pago en efectivo por el total a pagar y sale del modo dividido (pierde los demás pagos).

**Monto recibido y vuelto.** `cambioAEntregar = máximo(0; redondeo a centavos de (Σ de todos los pagos − montoAPagar))` (`calcularVuelto`, `VUELTO.ts:4-7`); la suma incluye los pagos en cualquier método. Se muestra en tres sitios: «Vuelto: S/ x.xx» junto a «Recibido (S/)» (solo pago simple en efectivo), la línea «Vuelto a Entregar:» de los totales (cuando lo pendiente es 0 o negativo) y, al confirmar y en el comprobante, «Vuelto Entregado:» / «Vuelto:». El cobro solo se permite si el vuelto se cubre con efectivo: `puedeProcesar` exige `montoPendiente = 0` o (`montoPendiente < 0` **y** `cambioAEntregar ≤ efectivoRecibido`, donde `efectivoRecibido` es la suma de los pagos en efectivo) (`NV.ts:246-261`). Es el mismo criterio de la API (`Venta.Pagos.VueltoExcedeEfectivo`, RN-VEN-044). Si sobra dinero y no hay efectivo no aparece mensaje: el botón queda deshabilitado.

**Número de operación (cuándo es obligatorio).** Todo pago cuyo método no sea efectivo necesita un número de operación con algún carácter que no sea espacio (`referenciaFaltante`, `NV.ts:953-955`). Mientras falte alguno aparece, bajo los pagos y en rojo, «Ingresa el N° de operación para poder cobrar.» (`pos.checkout.referenciaRequerida`, `NV.html:810-814`) y «Procesar Venta» queda deshabilitado. El efectivo no lleva número.

**Validaciones previas al cobro** (`puedeProcesar`, `NV.ts:246-261`): el botón «Procesar Venta» solo se habilita si se cumplen **todas** estas condiciones; ninguna muestra un mensaje propio salvo donde se indica:

| Condición | Mensaje que ve el cajero cuando no se cumple |
|---|---|
| Hay al menos una línea en el carrito | Ninguno; el carrito muestra «El carrito está vacío.» |
| Hay caja propia abierta (`sesionId`) | Se ve la tarjeta de «Estado de caja» en lugar del terminal |
| `montoPendiente` es 0, o es negativo y el vuelto se cubre con efectivo | Ninguno; «Monto Pendiente:» o «Vuelto a Entregar:» en los totales |
| Todos los pagos no efectivo tienen número de operación | «Ingresa el N° de operación para poder cobrar.» |
| La cobertura de seguro es válida (siempre cumplida con el bloque oculto) | — |
| Si hace falta, la casilla «Receta verificada» está marcada | La casilla y su ayuda (apartado 3.5) |
| No hay aviso de receta pendiente (`recetaAvisoKey`) | El aviso correspondiente (apartado 3.5) |
| No se está procesando otra operación | Indicador de carga en el botón |

Los textos `pos.messages.montoInsuficiente`, `sumaPagosInsuficiente`, `vueltoSinEfectivoError`, `cartEmptyWarning` y `cajaRequiredWarning` están en `es.json` pero ningún archivo los usa (apartado 12): no existe aviso de «monto insuficiente».

### 3.8 Zona: aseguradora (oculta por una bandera)

`NV.ts:214-215`: `readonly mostrarAseguradora = false;` precedido del comentario «Insurance coverage is built but hidden from the UI until the pharmacy starts working with aseguradoras.». La plantilla envuelve el bloque completo en `@if (mostrarAseguradora)` (`NV.html:456-512`). Es una constante de instancia fijada en el código: ningún rol, empresa ni configuración la cambia. **El bloque no se muestra a ningún usuario.**

Lo que existe en el código y no se ve:
- Bloque «Cobertura de Seguro (opcional)» (`pos.checkout.aseguradoraTitle`) dentro del panel de cliente, con una fila plegada «Aseguradora» (`pos.checkout.aseguradoraLabel`, icono `add_circle_outline`) que despliega, al pulsarla, el autocompletar «Aseguradora» (busca por `nombreComercial`, 20 resultados, `GET seguros`, `Seguros:Leer`) y el campo numérico «Monto Cubierto por Seguro (S/)» (`pos.checkout.montoCubiertoLabel`, `min=0`, `max` = subtotal, `step=0.01`) con el error «El monto cubierto por el seguro debe estar entre 0 y el total de la venta.» (`pos.messages.seguroMontoInvalido`).
- Lógica: `montoAPagar = subtotalBruto − montoCubiertoSeguro` (la cobertura solo cuenta con aseguradora elegida); `puedeProcesar` exige `montoCubiertoValido`; `aseguradoraId` y `montoCubiertoSeguro` viajan en `POST ventas` y `PUT ventas/{id}/completar`; el borrador local guarda la aseguradora, el monto y si el bloque estaba desplegado (`NV.ts:226-233,482-490,1088-1098`). Los códigos del servidor `Venta.Seguro.MontoInvalido`, `CreateVentaCommand.AseguradoraId` y similares están en el mapa de errores (apartado 3.12). Reglas: RN-VEN-047.
- **Riesgo derivado de la lectura:** `restoreDraft` aplica la aseguradora y el monto cubierto de un borrador guardado antes de ocultar el bloque (`NV.ts:507-515`); como el bloque no se ve, esa cobertura reduciría «TOTAL A PAGAR» sin forma visible de quitarla. **No verificado** en ejecución (depende de que existan borradores antiguos con aseguradora en un navegador concreto; caducan a las 12 horas, apartado 3.9).

### 3.9 Borrador local del carrito (autoguardado en el navegador)

El terminal guarda sola la venta en curso en `localStorage` del navegador y la recupera si el cajero recarga la página o cierra la pestaña. Código: `NV.ts:137-145,476-523,526-569`; formato y caducidad en `DRAFT.ts`; claves por usuario en `app/core/auth/user-storage.ts`.

| Aspecto | Detalle | Fuente |
|---|---|---|
| Clave de almacenamiento | `pos-nueva-venta-draft:<id de usuario>` (el id es la reclamación `sub` del token). Una venta en espera retomada usa su propia clave: `pos-nueva-venta-draft:<id de usuario>-<id de la venta>` | `user-storage.ts:3-4,8-10`; `NV.ts:137-145` |
| Cuándo se guarda | En cada cambio del carrito, de los pagos, del cliente, de la receta elegida, de la aseguradora o del bloque de seguro (`effect(() => this.saveDraft())`, `NV.ts:350`). No guarda mientras se carga una venta en espera (`cargandoContinuacion`) ni si no hay usuario en el token | `NV.ts:476-481` |
| Qué guarda | `cart` (cada línea con lote, nombre, número de lote, existencia, cantidad, precio unitario, datos de presentación y condición de venta), `pagos` (método, monto y número de operación), `clienteId` (solo el id, sin datos personales), `recetaId`, `aseguradora`, `montoCubierto`, `seguroExpanded` y la marca de hora `savedAt` | `DRAFT.ts:4-12,22-34`; `NV.ts:482-490` |
| Qué no guarda | La casilla «Receta verificada» (quien cobra debe volver a marcarla), el texto de búsqueda, el lote seleccionado, la imagen y los resultados del escaneo, la pestaña activa | `NV.ts:188-189` |
| Borrador vacío | Si el carrito está vacío y no hay cliente ni aseguradora, se **borra** la clave en lugar de guardar | `DRAFT.ts:18-20`; `NV.ts:491-494` |
| Caducidad | 12 horas contadas desde el último guardado (`POS_DRAFT_TTL_MS`). Se comprueba **al leer**: un borrador caducado, corrupto o antiguo (sin `savedAt`, de cuando se guardaban datos del cliente) se descarta y se borra | `DRAFT.ts:2,36-56`; `NV.ts:519-523` |
| Cómo se restaura | Solo en **venta nueva** (sin `?continuar=`), al abrir la pantalla: recupera carrito, pagos (pasa al modo dividido si hay más de uno), cliente (con `GET pacientes/{id}`; si falla, descarta el cliente), receta elegida, aseguradora y monto cubierto. No vuelve a validar la existencia de los lotes: las líneas conservan la existencia con que se guardaron | `NV.ts:359-364,498-516,526-547` |
| Con `?continuar=<id>` | Solo se recuperan los pagos «recuperables» del borrador de esa venta: filas que no son efectivo, con monto mayor que 0 y número de operación. El efectivo escrito nunca se conserva para una venta en espera | `NV.ts:549-569` |
| Cuándo se borra | (1) Al cobrar con éxito (`removeItem` de la clave vigente, `NV.ts:1105`). (2) Al quedar vacío el carrito sin cliente ni aseguradora. (3) Al leerlo caducado o inválido. (4) Al **cerrar sesión**: `clearUserStorage` borra todas las claves con el prefijo (`auth.service.ts:121`). (5) Al **iniciar sesión otro usuario** en el mismo navegador: `clearOtherUsersStorage` borra los de los demás y conserva los propios (`auth.service.ts:78`). Un borrador de venta en espera retomada que se actualiza (`Actualizar Venta en Espera`) no se borra: queda hasta caducar | `user-storage.ts:13-28` |
| Dónde queda | En el navegador y equipo del cajero; las pestañas del mismo usuario comparten la clave | derivado |

`saveDraft` y `restoreDraft` leen y escriben `localStorage` sin `try/catch` (`NV.ts:492,495,520-521`); si el navegador bloquea el almacenamiento, esas llamadas fallarían (`user-storage.ts` sí protege sus accesos). **No verificado** en ejecución.

### 3.10 Retomar una venta en espera: `/pos/nueva?continuar=<id>`

**Cómo se llega:** botón «Cobrar Venta» del detalle de una venta en estado «Pendiente» (apartado 7), que navega a `/pos/nueva` con `queryParams: { continuar: <id> }` (`VD.ts:104-106`). **Permisos:** el guard de ruta exige `Ventas:Leer` + `Ventas:Crear`; el botón del detalle exige además `Ventas:Actualizar` (`VD.ts:99-102`); escribir la URL a mano con solo `Ventas:Crear` abre la pantalla y el servidor responde 403 al cobrar o actualizar (hallazgo a1 de [05b](../05-seguridad/05b-matriz-de-permisos.md)). Cualquier usuario de la misma sucursal con esos permisos puede retomarla, no solo quien la guardó (RN-VEN-053).

**Qué carga** (`cargarVentaPendiente`, `NV.ts:402-470`), mostrando el indicador de carga mientras tanto:
1. `GET /api/v1/ventas/{id}` (permiso `Ventas:Leer`). Si falla: «Error al cargar la venta» (`pos.ventasDetail.loadError`); el terminal queda en modo venta nueva, vacío, y la URL conserva el `?continuar=`.
2. Si la venta ya no está «Pendiente»: «Esta venta ya no está en espera; te llevamos a su detalle.» (`pos.messages.ventaYaNoPendiente`, snackbar de error) y navega a `/pos/{id}`.
3. Si está pendiente, por cada línea consulta `GET ventas/lotes-disponibles?productoNombre=<nombre>` y calcula la existencia como «stock vendible actual del lote + cantidad que la propia venta ya tenía reservada». Si esa consulta falla se avisa con `pos.messages.lotesError` y la existencia queda en lo reservado por la venta.
4. Arma el carrito con lote, nombre, número de lote, cantidad, **precio guardado en la línea**, presentación y condición de venta de cada línea.
5. Si la venta tiene cliente, lo selecciona y carga sus recetas; elige la receta de la primera línea que tenga una (`NV.ts:446-458`).
6. Recupera los pagos recuperables del borrador (apartado 3.9).

Mientras se retoma, aparece sobre el terminal un banner con icono `schedule`: «Estás retomando una venta en espera: puedes agregar, quitar o modificar productos antes de cobrar.» (`pos.messages.continuandoVentaPendiente`, `NV.html:25-30`).

**Qué se puede cambiar:** todo lo del terminal: agregar productos y lotes, quitar líneas, cambiar cantidades, cliente, receta y pagos. Las líneas cargadas conservan el precio guardado; las nuevas usan el del catálogo (el servidor cobra siempre el del catálogo, `NV.ts:984-986`).

**Cómo se cobra:** «Procesar Venta» → confirmación → `PUT /api/v1/ventas/{id}/completar` con `pagos`, `detalles` (todas las líneas del carrito, otra vez), `clienteId`, `aseguradoraId`, `montoCubiertoSeguro` y `recetaVerificada` (`NV.ts:1084-1091`). Permiso del endpoint: `Ventas:Actualizar`. El servidor cobra en la **caja propia** de quien cobra y reasigna la venta a esa sesión (RN-VEN-006), emite la boleta y devuelve el comprobante, que se muestra en el diálogo del apartado 5. **Cómo se guarda de nuevo:** «Actualizar Venta en Espera» → `PUT /api/v1/ventas/{id}/detalles` con `detalles` y `clienteId` (`NV.ts:1026-1044`; permiso `Ventas:Actualizar`); tras guardar muestra «Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva.», limpia el terminal y navega a `/pos/nueva` sin `continuar`.

### 3.11 Acciones del terminal

| Texto exacto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Procesar Venta (`pos.actions.procesarVenta`), icono `check_circle`, con indicador de carga mientras procesa | Panel «Método de Pago», pie, botón principal | Siempre que el terminal esté visible | `!puedeProcesar()` (apartado 3.7) | `procesarVenta()`: si es válido, abre el [diálogo de confirmación](#4-diálogo-de-confirmación-de-venta-confirmarventadialogcomponent) (450 px); si el cajero confirma, `enviarVenta()` |
| Guardar y Atender Siguiente (`pos.actions.guardarPendiente`) o, con `?continuar=`, Actualizar Venta en Espera (`pos.actions.actualizarPendiente`), icono `schedule`, con indicador de carga | Pie del panel «Método de Pago», botón secundario | Siempre que el terminal esté visible | `!puedeGuardarPendiente()`: carrito vacío, sin caja, procesando o guardando (`NV.ts:263-265`). **No pide receta, pagos ni número de operación** | `guardarPendiente()`: venta nueva → `POST /api/v1/ventas` con `estado = Pendiente` y `pagos = []`; con `continuar` → `PUT /api/v1/ventas/{id}/detalles` |
| Vaciar (`pos.checkout.vaciarCart`) | Encabezado del carrito | Carrito no vacío | Nunca | `vaciarCarrito()`: borra las líneas y reinicia los pagos; **no** descarta cliente, receta ni aseguradora |
| × en cada línea | Línea del carrito | Siempre | Nunca | `quitarDelCarrito(i)` |
| Agregar (`pos.actions.agregar`) | Formulario de agregado | Con un lote seleccionado | Nunca | `agregarAlCarrito()` (apartado 3.2) |
| Aperturar Caja (`pos.actions.aperturar`) | Tarjeta de estado de caja | Sin caja y con `POS:AperturarCaja` | Nunca | `abrirCaja()` → diálogo de apertura |
| Botón volver (icono de flecha en la cabecera) | Cabecera | Siempre | Nunca | Navega a `/pos` sin avisar; el borrador queda guardado |

No existe un botón «Cancelar venta» ni «Limpiar» (la clave `pos.actions.cancelar` no se usa); el botón «Cobrar Venta» (`pos.actions.cobrar`) solo existe en el detalle de la venta. El terminal tampoco tiene atajos de teclado, foco automático ni listeners de teclado (búsqueda de `keydown`, `keyup`, `autofocus` y `focus()` en `NV.ts` y `NV.html` sin resultados). Los comportamientos automáticos son: búsqueda de productos con retardo de 300 ms, búsqueda de clientes con retardo de 300 ms, recarga de recetas del cliente cuando la ventana recupera el foco (`@HostListener('window:focus')`, `NV.ts:881-886`), autoguardado del borrador, desmarcado automático de «Receta verificada» y reinicio de los pagos al vaciar el carrito.

**Flujo de «Procesar Venta» en detalle** (`procesarVenta`, `enviarVenta`; `NV.ts:997-1016,1069-1142`):
1. Abre el diálogo de confirmación con el total a pagar, el número de líneas, el nombre del cliente (o «Cliente Eventual (Público General)»), los métodos de pago y el monto pendiente.
2. Si se confirma: marca «procesando»; arma `detalles` (por línea: `loteId`, cantidad —en unidades de la presentación si la línea la tiene, en unidades base si no—, `presentacionVentaId`, `recetaId` de la receta elegida; **el precio no se envía**) y `pagos` (método, monto, número de operación recortado o ausente). Venta nueva → `POST /api/v1/ventas` con `clienteId`, `sesionId`, `detalles`, `pagos`, `aseguradoraId`, `montoCubiertoSeguro` y `recetaVerificada`; venta en espera → `PUT ventas/{id}/completar`.
3. Éxito: borra el borrador y abre el [diálogo del comprobante](#5-diálogo-del-comprobante-comprobantedialogcomponent) (500 px, **no se puede cerrar fuera de sus botones**, `disableClose`). Cuando el diálogo se cierra, limpia el terminal y navega a `/pos` si se eligió «Volver a Caja POS» o a `/pos/nueva` en cualquier otro caso. **No hay aviso de éxito en snackbar**: el diálogo es la confirmación.
4. Error: apaga «procesando», muestra el mensaje del mapa del apartado 3.12 y, si el código es `Venta.Receta.VerificacionRequerida`, muestra la casilla «Receta verificada». El carrito y los pagos se conservan para corregir y reintentar.

### 3.12 Mapa de códigos de error del backend → mensaje

El objeto `VENTA_ERROR_KEYS` (`NV.ts:72-97`) traduce el código que trae la respuesta de error (`err.error?.code`) a una clave `pos.messages.*` (o a `errors.concurrencyConflict`). Se aplica a cobrar, guardar en espera, actualizar en espera y, con otro mensaje por defecto, a las consultas de catálogo, lotes y presentaciones. Las 24 filas del objeto:

| # | Código del backend | Texto que ve el cajero (clave) |
|---|---|---|
| 1 | `Venta.SinPrecio` | «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» (`pos.messages.sinPrecio`) |
| 2 | `Venta.Seguro.MontoInvalido` | «El monto cubierto por el seguro debe estar entre 0 y el total de la venta.» (`pos.messages.seguroMontoInvalido`) |
| 3 | `CreateVentaCommand.MontoCubiertoSeguro` | Igual que la fila 2 |
| 4 | `CompletarVentaCommand.MontoCubiertoSeguro` | Igual que la fila 2 |
| 5 | `CreateVentaCommand.AseguradoraId` | «Selecciona una aseguradora para aplicar la cobertura del seguro.» (`pos.messages.seguroSinAseguradora`) |
| 6 | `CompletarVentaCommand.AseguradoraId` | Igual que la fila 5 |
| 7 | `Aseguradora.NotFound` | «La aseguradora seleccionada no existe o fue eliminada.» (`pos.messages.aseguradoraNoEncontrada`) |
| 8 | `Venta.Caja.NoPropia` | «La caja seleccionada pertenece a otro cajero o sucursal. Recarga la página para usar tu propia caja abierta.» (`pos.messages.cajaNoPropia`) |
| 9 | `Venta.Caja.SinSesionAbierta` | «Necesitas tener tu propia caja abierta en esta sucursal para cobrar la venta.» (`pos.messages.cajaSinSesionAbierta`) |
| 10 | `Venta.Pagos.VueltoExcedeEfectivo` | «El vuelto no puede ser mayor que el efectivo recibido. Ajusta los montos de pago.» (`pos.messages.vueltoExcedeEfectivo`) |
| 11 | `Venta.Pagos.ReferenciaRequerida` | «Ingresa el número de operación de cada pago que no sea en efectivo.» (`pos.messages.referenciaRequerida`) |
| 12 | `CreateVentaCommand.Pagos.ReferenciaRequerida` | Igual que la fila 11 |
| 13 | `CompletarVentaCommand.Pagos.ReferenciaRequerida` | Igual que la fila 11 |
| 14 | `Venta.Receta.Requerida` | «Uno de los productos requiere receta médica. Selecciona el cliente y una de sus recetas registradas.» (`pos.messages.recetaRequerida`) |
| 15 | `Venta.Receta.NoEncontrada` | «La receta médica seleccionada no existe o fue eliminada.» (`pos.messages.recetaNoEncontrada`) |
| 16 | `Venta.Receta.OtroPaciente` | «La receta médica no pertenece al cliente de la venta.» (`pos.messages.recetaOtroPaciente`) |
| 17 | `Venta.Receta.Agotada` | «La receta médica ya fue dispensada el máximo de veces permitido.» (`pos.messages.recetaAgotada`) |
| 18 | `Venta.Receta.NumeroRequerido` | «La receta seleccionada no tiene número de receta. Agrégalo en la receta para vender productos de receta retenida o estupefacientes.» (`pos.messages.recetaNumeroRequerido`) |
| 19 | `Venta.Receta.VerificacionRequerida` | «Marca "Receta verificada" para confirmar que revisaste la receta del cliente antes de cobrar.» (`pos.messages.recetaVerificacionRequerida`); además activa la casilla (apartado 3.5) |
| 20 | `Venta.Lote.Vencido` | «Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.» (`pos.messages.loteVencido`) |
| 21 | `Venta.SerieBoleta.NoConfigurada` | «Esta sucursal no tiene una serie de boletas activa. Configúrala en SUNAT > Series Fiscales para poder cobrar.» (`pos.messages.serieBoletaNoConfigurada`) |
| 22 | `Venta.Empresa.SinRuc` | «Tu empresa no tiene RUC registrado, por lo que no puede emitir comprobantes. Complétalo en Administración > Empresa para poder cobrar.» (`pos.messages.empresaSinRuc`) |
| 23 | `Auth.Sucursal` | «No se pudo determinar el contexto de sucursal asignada del usuario.» (`pos.messages.contextError`) |
| 24 | `Concurrency.Conflict` | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`, raíz) |

**Mensaje genérico para todo código no mapeado, sin código o sin respuesta HTTP** (error de red incluido): «Ocurrió un error al procesar la venta.» (`pos.messages.ventaError`) al cobrar, guardar o actualizar; «No se pudo cargar el catálogo de medicamentos. Inténtalo nuevamente.» (`catalogoError`) en la búsqueda de productos; «No se pudieron cargar los lotes disponibles. Inténtalo nuevamente.» (`lotesError`) al listar lotes; «No se pudieron cargar las presentaciones del producto. Inténtalo nuevamente.» (`presentacionesError`) al listar presentaciones.

**Códigos del backend que el terminal no traduce y por tanto muestran el mensaje genérico** (de `docs/_inventario/codigos-error.csv` y `codigos-error-ocurrencias.csv`; los textos entre paréntesis son los mensajes internos en inglés del backend, solo para identificar el caso):

| Código | Cuándo ocurre | Fuente |
|---|---|---|
| `Venta.Stock` | La existencia no alcanza al cobrar («Stock insuficiente para el lote…») | `VentaDetalleFactory.cs:68` |
| `Venta.Cuarentena` | El lote está en cuarentena | `VentaDetalleFactory.cs:172` |
| `Venta.Lote` (no encontrado) | El lote no existe en la empresa o en la sucursal | `VentaDetalleFactory.cs:55,126,181,196` |
| `Venta.Presentacion` | Presentación inexistente o de otro producto | `VentaDetalleFactory.cs:229,234` |
| `Venta.Pagos` | Suma de pagos insuficiente, vuelto sin efectivo, sin pagos o venta anulada/devuelta | `Venta.cs:223,228,241,247` |
| `Venta.Caja` | Falta la sesión de caja o no está abierta | `CreateVentaCommandHandler.cs:136,144`; `Venta.cs:192,197` |
| `Venta.Completar`, `Venta.Actualizar` | Venta inexistente, de otra sucursal o ya no pendiente | `CompletarVentaCommandHandler.cs:34,39,44`; `ActualizarVentaPendienteCommandHandler.cs:31,36,41` |
| `Paciente.NotFound` | El cliente ya no existe | RN-VEN-025 |
| `Auth.Empleado` y errores de base de datos (`Db.*`) | Token sin empleado; duplicados o referencias | RN-VEN-002; `errors.db*` de la raíz no se usan aquí |

Los validadores de comandos (`CreateVentaCommand.Detalles`, `…Pagos.MontoPagado`, etc.) siguen el patrón `{Request}.{Propiedad}` (REGLAS-AGENTES §4) y, salvo los listados en el mapa, también caen en el genérico. Que la pantalla impida el caso (stock, pagos, referencias) reduce la frecuencia de estos errores, pero no los elimina: por ejemplo, otro cajero puede agotar un lote entre que se agregó al carrito y se cobró.

### 3.13 Mensajes al usuario del terminal (resumen)

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda de productos | Error | «No se pudo cargar el catálogo de medicamentos. Inténtalo nuevamente.» (`pos.messages.catalogoError`) | `NV.ts:295` |
| Falla la lista de lotes | Error | «No se pudieron cargar los lotes disponibles. Inténtalo nuevamente.» (`pos.messages.lotesError`) | `NV.ts:420,601,809` |
| Falla la lista de presentaciones | Error | «No se pudieron cargar las presentaciones del producto. Inténtalo nuevamente.» (`pos.messages.presentacionesError`) | `NV.ts:615` |
| Cantidad supera la existencia | Error | «Stock máximo disponible alcanzado ({{stock}} un.).» (`pos.messages.stockMaxReached`) | `NV.ts:646,679,823` |
| Producto sin precio | Error | «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» (`pos.messages.sinPrecio`) | `NV.ts:650,802` |
| Cantidad inválida | Error | «Ingresa una cantidad válida para agregar el producto.» (`pos.messages.cantidadPrecioInvalidos`) | `NV.ts:654` |
| Ítem escaneado sin lotes | Error | «El medicamento detectado no tiene stock en esta sucursal.» (`pos.messages.noStockLocalReceta`) | `NV.ts:797` |
| Falla el análisis de la receta | Error | «Error al interpretar la receta. Intenta nuevamente.» (`recetasMedicas.scanner.error`) | `NV.ts:761` |
| Falta número de operación | Mensaje en pantalla (no snackbar) | «Ingresa el N° de operación para poder cobrar.» (`pos.checkout.referenciaRequerida`) | `NV.html:810-814` |
| Venta guardada en espera | Éxito | «Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva.» (`pos.messages.ventaPendienteGuardada`) | `NV.ts:1035,1058` |
| La venta a retomar ya no está pendiente | Error | «Esta venta ya no está en espera; te llevamos a su detalle.» (`pos.messages.ventaYaNoPendiente`) | `NV.ts:409` |
| Falla la carga de la venta a retomar | Error | «Error al cargar la venta» (`pos.ventasDetail.loadError`) | `NV.ts:465` |
| Retomando una venta | Banner (no snackbar) | «Estás retomando una venta en espera: puedes agregar, quitar o modificar productos antes de cobrar.» (`pos.messages.continuandoVentaPendiente`) | `NV.html:25-30` |
| Error al cobrar, guardar o actualizar | Error | Mapa del apartado 3.12 | `NV.ts:1041,1064,1139` |
| Cobro correcto | (sin snackbar) | Se abre el diálogo «Venta Procesada Exitosamente» | `NV.ts:1126` |

### 3.14 Llamadas a la API del terminal

| Cuándo | Método del servicio | Verbo y URL | Permiso que exige el endpoint |
|---|---|---|---|
| Abrir (buscar la caja propia) | `SesionCajaService.search` | `GET /api/v1/pos/sesiones?pageNumber=1&pageSize=1&sucursalId&empleadoId&estadoSesion=0` | `POS:Leer` (`POSController.cs:28`) |
| Abrir | `VentaService.configuracion` (`VSV.ts:57-59`) | `GET /api/v1/ventas/configuracion` | `Ventas:Crear` (`VentasController.cs:66`) |
| Abrir | `MedicamentoService.getCondicionesVenta` | `GET /api/v1/medicamentos/condiciones-venta` | `Medicamentos:Leer` (`MedicamentosController.cs:23`) |
| Escribir en el buscador, pulsar un chip y al abrir | `MedicamentoService.search` | `GET /api/v1/medicamentos?pageNumber=1&pageSize=12&nombreComercial&condicionVenta&incluirStock&ordenarPorMasVendidos&soloStockVendibleEnMiSucursal` | `Medicamentos:Leer` (`MedicamentosController.cs:27`) |
| Pulsar un producto; agregar desde la receta; retomar | `VentaService.lotesDisponibles` (`VSV.ts:44-50`) | `GET /api/v1/ventas/lotes-disponibles?productoId` o `?productoNombre` | `Ventas:Crear` (`VentasController.cs:48`) |
| Pulsar un lote | `VentaService.presentacionesProducto` (`VSV.ts:52-55`) | `GET /api/v1/ventas/presentaciones?productoId` | `Ventas:Crear` (`VentasController.cs:57`) |
| Buscar cliente | `PacienteService.search` | `GET /api/v1/pacientes?pageNumber=1&pageSize=20&docIdentidadGub` o `&nombreApellidos` | `Pacientes:Leer` (`PacientesController.cs:29`) |
| Restaurar el cliente de un borrador | `PacienteService.getById` | `GET /api/v1/pacientes/{id}` | `Pacientes:Leer` (`PacientesController.cs:38`) |
| Elegir cliente; al volver el foco a la ventana | `RecetaMedicaService.search` | `GET /api/v1/recetas?clienteId&pageNumber=1&pageSize=50` | `Recetas:Leer` (`RecetasController.cs:24`) |
| «Interpretar receta» | `RecetaMedicaService.analizar` | `POST /api/v1/recetas/analizar` (multipart) | `Recetas:Analizar` (`RecetasController.cs:48`), con límite de la política `AiPolicy` |
| Escribir una aseguradora (bloque oculto, inalcanzable) | `AseguradoraService.search` | `GET /api/v1/seguros` | `Seguros:Leer` (`SegurosController.cs:26`) |
| «Procesar Venta» (nueva) o «Guardar y Atender Siguiente» | `VentaService.crear` (`VSV.ts:61-63`) | `POST /api/v1/ventas` | `Ventas:Crear` (`VentasController.cs:38`) |
| Retomar | `VentaService.getById` (`VSV.ts:40-42`) | `GET /api/v1/ventas/{id}` | `Ventas:Leer` (`VentasController.cs:29`) |
| «Procesar Venta» con `continuar` | `VentaService.completar` (`VSV.ts:65-67`) | `PUT /api/v1/ventas/{id}/completar` | `Ventas:Actualizar` (`VentasController.cs:84`) |
| «Actualizar Venta en Espera» | `VentaService.actualizarDetalles` (`VSV.ts:69-71`) | `PUT /api/v1/ventas/{id}/detalles` | `Ventas:Actualizar` (`VentasController.cs:93`) |
| «Aperturar Caja» | (diálogo, apartado 2) | `POST /api/v1/pos/apertura` | `POS:AperturarCaja` (`POSController.cs:13`) |

### 3.15 Observaciones

- **Código muerto o sin pantalla en `NV.ts`:** el bloque de aseguradora (apartado 3.8); `toggleSeguro`, `selectAseguradora` y el formulario `seguroForm` solo se alcanzan con ese bloque; el getter `itemsSugerencia`; `limpiarCliente` sin botón (apartado 3.4); `data: { isNew: true }` de la ruta `nueva` (`pos.routes.ts:11`) que `NuevaVentaComponent` no lee.
- **Claves de texto sin uso** de este terminal (carrito de texto, tarjetas de otra sede, diálogos de cobro y reporte): apartado 12.
- **Sin vencimiento en pantalla:** el cajero no ve la fecha de vencimiento del lote al elegirlo (apartado 3.2).
- **Línea por presentación al sumar:** al repetir una presentación del mismo lote, la cantidad en unidades base se limita a la existencia, pero la cantidad en presentaciones (`cantidadEnPresentacion`) se suma sin tope (`NV.ts:682-693`). Como lo que se envía al servidor para esas líneas es la cantidad en presentaciones (`NV.ts:990`), el servidor es quien detiene el exceso (con `Venta.Stock`, que el terminal no traduce). Derivado de la lectura; **No verificado** en ejecución.
- **Cliente que no se puede quitar:** apartado 3.4.
- **Mensaje de «monto insuficiente»:** no existe; el cajero solo ve el botón deshabilitado y «Monto Pendiente:» (apartado 3.7).
- **Símbolo de moneda en duro:** «S/» está escrito directamente en las plantillas (`NV.html`, `CMP.html`, `CNF.html`); no pasa por i18n.

---

## 4. Diálogo de confirmación de venta (`ConfirmarVentaDialogComponent`)

- **Componente:** `ConfirmarVentaDialogComponent` (`CNF.ts`, `CNF.html`). Scope `pos`. Recibe los datos por `MAT_DIALOG_DATA` (`ConfirmarVentaDialogData`: `total`, `itemsCount`, `clienteNombre`, `metodosPago`, `montoPendiente`, `CNF.ts:7-13`).
- **Para qué sirve:** es el último paso antes de cobrar: el cajero revisa un resumen y confirma. Es el «diálogo de cobro» del terminal (no existe un diálogo aparte de cobro: ver `cobrarDialog.*` en el apartado 12).
- **Cómo se abre:** desde `/pos/nueva`, botón «Procesar Venta», solo si `puedeProcesar()` es verdadero (`procesarVenta()`, `NV.ts:997-1016`), con ancho de 450 px. Se puede cerrar con Esc o pulsando fuera; eso equivale a cancelar (`afterClosed` solo actúa con `true`).
- **Título y subtítulo:** «Confirmar Procesamiento de Venta» (`pos.confirmarDialog.title`) y «Verifica los detalles antes de registrar la transacción.» (`pos.confirmarDialog.subtitle`).

**Datos que muestra** (`CNF.html:5-30`; etiqueta en negrita y valor a la derecha):

| Etiqueta (clave) | Valor | Fuente del dato |
|---|---|---|
| Cliente: (`pos.confirmarDialog.clienteLabel`) | Nombre del cliente elegido o «Cliente Eventual (Público General)» | `NV.ts:1005` |
| Cantidad de Ítems: (`pos.confirmarDialog.itemsLabel`) | «{n} producto(s)» (`pos.confirmarDialog.itemsCountText`), con `n` = número de **líneas** del carrito, no de unidades | `NV.ts:1004` |
| Método de Pago: (`pos.confirmarDialog.metodoLabel`) | Los métodos de cada fila de pago separados por coma, por ejemplo «Efectivo, Yape / Plin» | `NV.ts:1006` |
| Total a Pagar: (`pos.confirmarDialog.totalLabel`) | «S/ {montoAPagar}» con 2 decimales | `NV.ts:1003` |
| Vuelto Entregado: (`pos.confirmarDialog.vueltoLabel`) | «S/ {−montoPendiente}»; **solo aparece** si `montoPendiente < 0`, es decir, si se pagó de más | `CNF.html:24-29` |

El diálogo no muestra los montos de cada pago, ni el monto recibido (la clave `pos.confirmarDialog.recibidoLabel` no se usa), ni el detalle de líneas.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie, a la derecha | Siempre | Nunca | `onCancel()` → cierra con `false`; vuelve al terminal sin cambios |
| Confirmar y Procesar Venta (`pos.confirmarDialog.btnConfirmar`) | Pie, a la derecha | Siempre | Nunca | `onConfirm()` → cierra con `true`; el terminal ejecuta `enviarVenta()` (apartado 3.11): `POST /api/v1/ventas` o `PUT /api/v1/ventas/{id}/completar` |

**Mensajes y llamadas a la API:** el diálogo no muestra mensajes ni llama a la API; los errores del cobro los muestra el terminal (apartado 3.12).

**Observaciones:** el botón de confirmación no se deshabilita tras pulsarlo, pero el diálogo se cierra al instante y el terminal pone el botón «Procesar Venta» en «procesando», lo que impide un segundo envío (`NV.ts:1070,819`).

---

## 5. Diálogo del comprobante (`ComprobanteDialogComponent`)

- **Componente:** `ComprobanteDialogComponent` (`CMP.ts`, `CMP.html`, `comprobante-dialog.component.scss`). Scope `pos`. Recibe `ComprobanteDialogData` (`CMP.ts:27-39`): comprobante (o nulo), total, subtotal, IGV, métodos de pago como texto, líneas, pagos, vuelto, nombre del cliente, nombre del cajero y fecha. Usa `ReceiptPrintService` (`RPS.ts`) para imprimir y `*appPermiso` (`PermisoDirective`).
- **Para qué sirve:** mostrar el comprobante emitido (boleta) al terminar el cobro y permitir imprimir el ticket, empezar otra venta o volver a la lista. También sirve para **reimprimir** desde el detalle de la venta.
- **Cómo se abre:**
  - Tras cobrar en `/pos/nueva`: ancho 500 px y `disableClose: true`; solo se cierra con sus tres botones (`NV.ts:1126`). Los datos se arman con lo que había en pantalla al cobrar (`NV.ts:1106-1125`): total = «TOTAL A PAGAR», subtotal e IGV de la pantalla, pagos tal cual (el efectivo con el monto recibido), vuelto de la pantalla, cliente, el nombre completo del usuario conectado como cajero y la **hora del navegador** (`new Date()`).
  - Desde `/pos/:id`, botón «Reimprimir Comprobante»: ancho 500 px, sin `disableClose` (Esc o clic fuera la cierran); los datos se arman de la venta guardada (apartado 7).
- **Permiso:** el botón «Nueva Venta POS» del diálogo exige `Ventas:Crear` (`*appPermiso="'Ventas:Crear'"`, `CMP.html:38`); los otros dos no tienen control.

### 5.1 Lo que se ve en el diálogo

Título «Venta Procesada Exitosamente» (`pos.comprobante.title`) y, en una línea fija, «Comprobante registrado. Pendiente de envío a SUNAT.» (`pos.comprobante.subtitle`; respeta saltos de línea). **Ambos textos son fijos**: se muestran igual en la reimpresión desde el detalle y para cualquier estado del comprobante. Después, en dos columnas (`CMP.html:5-30`):

| Etiqueta (clave) | Valor | Cuándo aparece |
|---|---|---|
| Nº Comprobante: (`pos.comprobante.boletaNumero`) | «{tipo} {número}»: el tipo es el nombre del tipo de comprobante que devuelve la API (para una boleta, «Boleta») y el número tiene la forma «{prefijo de la serie}-{correlativo de 8 dígitos}» | Si hay comprobante |
| Estado SUNAT: (`pos.comprobante.estadoSunat`) | Texto del estado: «Pendiente de envío a SUNAT» para `NoEnviado` y para `Pendiente` (`pos.estadoComprobante.NoEnviado` y `Pendiente`), «Aceptado por SUNAT» (`Aceptado`), «Rechazado por SUNAT» (`Rechazado`), «Anulado» (`Anulado`); un estado desconocido se muestra tal cual (`estadoComprobanteKey`, `VM.ts:77-82`) | Si hay comprobante |
| Total Pagado: (`pos.comprobante.totalPagado`) | «S/ {total}» | Siempre |
| Medio de Pago: (`pos.comprobante.medioPago`) | Los métodos separados por coma | Siempre |

El estado que emite hoy el backend es siempre `NoEnviado` (RN-CMP-007): el envío a SUNAT no existe en el sistema, y por eso el ticket dice «Pendiente de envío a SUNAT».

**Botones y acciones** (`CMP.html:35-53`)

| Texto exacto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nueva Venta POS (`pos.actions.nuevaVenta`) | Fila principal, ancho completo | Con `Ventas:Crear` | Nunca | `nuevaVenta()` → cierra con `'nueva'`: el terminal limpia la venta y navega a `/pos/nueva`; desde el detalle, navega a `/pos/nueva` |
| Imprimir Comprobante (`pos.actions.imprimirComprobante`) | Fila inferior, a la izquierda | Siempre | Nunca | `imprimir()` (`CMP.ts:89-92`): imprime el ticket (apartado 5.3) |
| Volver a Caja POS (`pos.actions.volverPOS`) | Fila inferior, a la derecha | Siempre | Nunca | `volverPOS()` → cierra con `'volver'`: el terminal limpia la venta y navega a `/pos`; desde el detalle, navega a `/pos` |

Desde el terminal, cerrar el diálogo (por cualquiera de los botones; no hay otra forma) limpia siempre el terminal (`resetVenta`).

### 5.2 El ticket que se imprime (bloque oculto en pantalla)

El diálogo contiene además un bloque `.pos-print-receipt` que **no se ve en pantalla** (`display: none`, `src/styles.scss:995-997`); es lo que se copia al papel. Contenido, en orden (`CMP.html:56-153`):

| Zona | Texto | Detalle |
|---|---|---|
| Encabezado, centrado | Razón social del emisor (en negrita, 14 px) | Nombre de la empresa; solo si existe |
| | «RUC: {ruc}» (`pos.comprobante.rucLabel` = «RUC:») | RUC de la empresa; solo si existe |
| | Dirección de la sucursal | Solo si existe |
| | «{tipo} {número}» | Por ejemplo la serie y el correlativo de la boleta; vacío si no hay comprobante |
| | Fecha y hora `dd/MM/yyyy HH:mm` | Al cobrar: hora del navegador al armar el diálogo. Al reimprimir: `fechaHora` de la venta |
| Tabla de 4 columnas | Encabezados «Producto / Lote» (`pos.checkout.colItem`), «Cant.» (`colCant`), «P. Unit.» (`colPrecio`), «Subtotal» (`colTotal`) | Anchos 50 %, 10 %, 19 % y 21 % |
| Una fila por línea | Producto: «{producto} ({lote})». Línea **sin** presentación: cantidad en unidades base y precio unitario. Línea **por presentación**: cantidad «{n} × {presentación} ({unidades base})», por ejemplo «1 × Caja (10)», y precio por presentación (`cantidad × precio unitario ÷ n`). Subtotal de la línea: `cantidad × precio unitario` | Importes con 2 decimales y sin «S/» |
| Cliente | «Datos del Cliente:» (`pos.checkout.clienteTitle` + «:») y el nombre; «Cliente Eventual (Público General)» si no hay cliente | — |
| Documento | «Documento:» (`pos.comprobante.documentoLabel`) y «{tipo} {número}», por ejemplo «DNI 45781236» | No se imprime para el cliente eventual (número `00000000`); un tipo sin etiqueta imprime solo el número (`clienteDocumento()`, `CMP.ts:65-70`) |
| Importes | «Subtotal (Op. Gravada):», «IGV (18%):» y, en negrita con línea superior, «TOTAL A PAGAR:» | «S/ x.xx»; el navegador los calcula (subtotal = total ÷ 1.18); no se leen de la boleta guardada (RN-CMP-014) |
| Pagos | Una fila por pago: «{método}» y «S/ {monto}» | Al cobrar, el efectivo sale con el monto **recibido**; al reimprimir, con lo guardado en la venta |
| Vuelto | «Vuelto:» (`pos.checkout.vueltoCortoLabel`) y «S/ {vuelto}» | Solo si el vuelto es mayor que 0. Al reimprimir se recalcula como pagos menos el total de la venta, sin considerar cobertura de seguro (`VD.ts:119-122`) |
| Estado | «Estado SUNAT:» y el texto del estado (por ejemplo «Pendiente de envío a SUNAT») | Si hay comprobante |
| Cajero | «Cajero:» (`pos.reportDialog.cajeroLabel`) y el nombre | Al cobrar: nombre completo del usuario conectado. Al reimprimir: `empleadoNombre` de la venta (el cajero que la hizo). Solo si hay nombre |

El ticket **no incluye** código QR, hash de firma, enlace de verificación ni leyenda legal de representación impresa; tampoco lista los datos de la serie fiscal más allá del número. La única leyenda de estado es «Estado SUNAT: Pendiente de envío a SUNAT» (para `NoEnviado`).

### 5.3 Cómo funciona la impresión

`ReceiptPrintService.print(elemento)` (`RPS.ts:37-73`) no abre una ventana nueva: copia el bloque `#receipt` a un `iframe` oculto del mismo origen y llama a la impresión del navegador.
1. Borra un `iframe` anterior (`pos-receipt-print-frame`, `RPS.ts:4,38`) y crea uno nuevo fuera de pantalla (`position:fixed; left:-10000px`), de 80 mm de ancho y 0 de alto.
2. Escribe un documento con `<!DOCTYPE html>` (evita el modo de compatibilidad), le agrega los estilos del ticket (`RECEIPT_PRINT_STYLES`, `RPS.ts:11-33`) y una copia del bloque.
3. Mide la altura del ticket y define la hoja como continua: `@page { size: 80mm {alto}mm; margin: 0; }`, con el alto = altura medida convertida a mm + 6 mm de holgura (`RPS.ts:65-67`), de modo que un ticket largo sale completo en una sola hoja.
4. Enfoca el `iframe`, llama a `print()` (abre el diálogo de impresión del navegador; el usuario elige impresora y confirma) y, al terminar (`afterprint`), elimina el `iframe`.

Estilos del ticket (`RPS.ts:11-33`): papel de rollo de 80 mm (`RECEIPT_PAPER_WIDTH_MM`), cuerpo de 80 mm con 2 mm / 4 mm de relleno (columna imprimible de unos 72 mm), fuente Arial/Helvetica de 12 px, tabla de 11 px con líneas punteadas, encabezado centrado, filas «etiqueta a la izquierda, valor a la derecha» y total en negrita de 13 px. Solo sale el ticket: nada de la aplicación llega al papel. La impresión **no es automática**: se produce cuando el cajero pulsa «Imprimir Comprobante» y se puede repetir las veces que haga falta.

**Reimpresión desde el detalle.** Botón «Reimprimir Comprobante» (`pos.actions.reimprimirComprobante`) del detalle de la venta, visible si la venta tiene comprobante y no está «Anulada» (apartado 7). Abre este mismo diálogo con los datos de la venta guardada, así que **también muestra el título «Venta Procesada Exitosamente» y el subtítulo fijo**.

**Mensajes al usuario:** el diálogo no muestra mensajes. **Llamadas a la API:** ninguna (el comprobante viaja en la respuesta de `POST /api/v1/ventas` o `PUT ventas/{id}/completar`, o en `GET ventas/{id}` para la reimpresión).

**Observaciones**
- La fecha del ticket recién cobrado es la del navegador, no la de la venta guardada; la reimpresión usa la del servidor, por lo que ambas pueden diferir por segundos o por la hora del equipo.
- Los importes impresos los calcula el navegador; no coinciden necesariamente al centavo con la boleta guardada (RN-CMP-014, RN-CMP-006).
- El título del diálogo en una reimpresión dice «Venta Procesada Exitosamente» aunque no se haya procesado nada.
- `checkout.colTotal` se rotula «Subtotal» en el ticket (columna del importe de la línea) y no «Total».

---

## 6. Búsqueda de ventas: `/pos` — Ventas

- **Componente:** `VentasSearchComponent` (`VS.ts`, `VS.html`). Scope `pos`.
- **Para qué sirve:** consultar el historial de ventas de la sucursal por empleado, cliente, fechas y estado, abrir el detalle, exportar a Excel y entrar al terminal de venta. Es también donde se encuentran las ventas «Pendiente» (en espera) para retomarlas.
- **Cómo se llega:** menú lateral, grupo «Punto de Venta» → «Ventas» (`nav.ventas`); enlaces del tablero que abren la pantalla ya filtrada (apartado 0.5); botón «Volver a Caja POS» del diálogo del comprobante; botón volver del terminal y del detalle.
- **Permiso de ruta:** `Ventas:Leer` (`route-permissions.ts:20`). Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Ventas» (`pos.ventasSearch.title`); migas «Inicio» / «Punto de Venta» / «Ventas» (activa); sin botón volver (`VS.ts:71-75`, `VS.html:3-6`). La tarjeta de filtros repite «Ventas» como encabezado con el subtítulo «Consulta el historial de ventas registradas» (`pos.ventasSearch.subtitle`; no usa el subtítulo común).
- **Modos:** uno. Los resultados aparecen tras «Buscar» o si hay estado guardado (`VS.ts:114-124`).

**Filtros de búsqueda** (`VS.ts:63-69`, `VS.html:27-125`; en el orden de la pantalla)

| Etiqueta (clave) | Control (`formControlName`) | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Empleado (`pos.ventasSearch.fieldEmpleado`) | `empleadoNombre` | Texto con autocompletar y lupa; Enter busca | Con 300 ms de retardo busca empleados por `nombres` (20 resultados; `Empleados:Leer`); cada opción muestra el nombre completo. Si no hay resultados y hay texto: opción deshabilitada «Sin resultados» (`pos.ventasSearch.autocompleteNoResults`). **El filtro solo se aplica si se elige una opción**: texto escrito sin elegir no filtra | Vacío |
| Cliente (`pos.ventasSearch.fieldCliente`) | `clienteNombre` | Texto con autocompletar y lupa; Enter busca | Busca pacientes: solo dígitos → por documento; si no → por nombre y apellidos (20 resultados; `Pacientes:Leer`). Opción: el nombre. «Sin resultados» igual que arriba. Solo filtra si se elige una opción | Vacío |
| Fecha desde (`pos.ventasSearch.fieldFechaInicio`) | `fechaInicio` | Campo con selector de fecha | Inicio del día local en ISO | Nulo |
| Fecha hasta (`pos.ventasSearch.fieldFechaFin`) | `fechaFin` | Campo con selector de fecha | Último instante del día local | Nulo |
| Estado (`pos.ventasSearch.fieldEstado`) | `estado` | Select | «— Todos —» (nulo), «Completada», «Anulada», «Devuelta», «Pendiente» (`pos.estado.*`) | Nulo |

**No existe filtro por código de venta, por monto, ni por sucursal** (la sucursal es siempre la del token). «Cajero» es el filtro «Empleado». «Limpiar» (`actions.clear`) vacía el formulario y los autocompletar, oculta la tabla, vuelve a la página 1 y borra el estado guardado (`clear()`, `VS.ts:191-202`).

**Qué ventas ve cada usuario.** El SPA no restringe nada por rol ni por usuario. La API devuelve solo las ventas de la **sucursal del token**, de **todos** los empleados; cajero y administrador ven lo mismo, y para ver las propias se filtra por «Empleado» (RN-VEN-070, RN-VEN-071; `GetVentasQueryHandler.cs:28-30`).

**Tabla de resultados** (`VS.html:151-225`; orden: lo fija la API, por fecha-hora descendente, RN-VEN-073)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Código de Venta (`pos.ventasTable.colCodigo`) | `codigoVenta` | Texto: «{prefijo de la serie}-{correlativo de 8 dígitos}» si tiene boleta; si no, los 8 primeros caracteres del identificador (RN-VEN-059) | Sí: `routerLink [row.id]` → `/pos/:id` |
| Fecha (`pos.ventasTable.colFecha`) | `fechaHora` | `dd/MM/yyyy HH:mm`, hora del navegador | No |
| Cliente (`pos.ventasTable.colCliente`) | `clienteNombre` | Texto; «—» si es nulo. La API envía «Público General» cuando no hay cliente (`GetVentasQueryHandler.cs:89`), así que el «—» no se ve | No |
| Empleado (`pos.ventasTable.colEmpleado`) | `empleadoNombre` | Texto | No |
| Total (S/) (`pos.ventasTable.colTotal`) | `total` | 2 decimales | No |
| Estado (`pos.ventasTable.colEstado`) | `estado` | «Completada», «Anulada», «Devuelta» o «Pendiente» (`pos.estado.<estado>`) | No |
| Ítems (`pos.ventasTable.colItems`) | `itemsCount` | Número de líneas | No |

Encabezado «Resultados (N)». Paginación 10, 25 y 50 (primera/última; 10 por defecto). Sin resultados: «Sin resultados». Bajo la tabla: «Exportar» a la izquierda y el paginador a la derecha.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nueva Venta POS (`pos.actions.nuevaVenta`), icono `add` | Encabezado de la tarjeta de filtros | `*appPermiso="'Crear'"` (= `Ventas:Crear`, `VS.html:17`) | Nunca | `goToNew()` → `/pos/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` |
| Buscar (`actions.search`); con indicador de carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/ventas` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/ventas/exportar` |
| Enlace con el código de venta | Primera columna | Con resultados | — | Navega a `/pos/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar ventas» (`pos.ventasSearch.searchError`) | `VS.ts:186` |
| Falla la exportación | Error | «Error al buscar ventas» (misma clave) | `VS.ts:235` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Buscar o cambiar de página | `VentaService.search` (`VSV.ts:28-38`) | `GET /api/v1/ventas?pageNumber&pageSize[&fechaInicio][&fechaFin][&estado][&empleadoId][&clienteId]` (`estado=0` sí se envía) | `Ventas:Leer` (`VentasController.cs:20`) |
| Escribir en «Empleado» | `EmpleadoService.search` | `GET /api/v1/empleados?nombres&pageNumber=1&pageSize=20` | `Empleados:Leer` (`EmpleadosController.cs:15`) |
| Escribir en «Cliente» | `PacienteService.search` | `GET /api/v1/pacientes?docIdentidadGub` o `nombreApellidos`, `pageSize=20` | `Pacientes:Leer` (`PacientesController.cs:29`) |
| Exportar | `VentaService.exportar` (`VSV.ts:83-85`) | `POST /api/v1/ventas/exportar` (blob) | `Ventas:Leer` (`VentasController.cs:102`) |

**Exportación:** cuerpo con `headers` (7, en orden): «Código de Venta», «Fecha», «Cliente», «Empleado», «Total (S/)», «Estado», «Ítems» (`pos.ventasTable.*`); `dateFormat` `dd/MM/yyyy` en español y `MM/dd/yyyy` si no (`VS.ts:225`); y los filtros actuales (`fechaInicio`, `fechaFin`, `estado`, `empleadoId`, `clienteId`). Archivo «ventas.xlsx» (`pos.ventasSearch.exportFilename`). Detalle de columnas: RN-VEN-078. Sin carga masiva.

**Comportamientos particulares:** guarda en memoria filtros, ids elegidos (`_empleadoId`, `_clienteId`), página y tamaño (`VS.ts:174-182`); `search()` no reinicia la página (igual que `/sesiones-caja`); el tablero abre esta pantalla con filtros ya puestos (apartado 0.5).

**Observaciones:** no se puede buscar por código de venta; el filtro de texto sin elegir opción se ignora en silencio; para retomar una venta en espera hay que filtrar por «Pendiente» y abrirla (RN-VEN-053).

---

## 7. Detalle de venta: `/pos/:id` — Detalle de Venta

- **Componente:** `VentaDetailComponent` (`VD.ts`, `VD.html`). Scope `pos`.
- **Para qué sirve:** ver una venta completa (datos, comprobante, líneas, pagos y devoluciones) y operar sobre ella: reimprimir el comprobante, cobrarla si está en espera, devolver o anular.
- **Cómo se llega:** enlace del código en `/pos`; redirección desde el terminal cuando una venta a retomar ya no está pendiente.
- **Permiso de ruta:** `Ventas:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Detalle de Venta» (`pos.ventasDetail.title`); migas «Inicio» / «Punto de Venta» / «Ventas» (enlace a `/pos`); botón volver a `/pos` (`VD.html:3-7`, `VD.ts:45-49`). La clave `ventasDetail.subtitle` no se usa.
- **Modos:** uno, solo lectura. Carga con indicador; si falla: «Error al cargar la venta» (`pos.ventasDetail.loadError`, `VD.ts:69`) y la pantalla queda sin datos.

**Datos mostrados** (`VD.html:15-130`)

| Zona | Etiqueta (clave) | Dato / formato | Condición |
|---|---|---|---|
| Encabezado | (sin etiqueta) | Código de venta en `h4`; debajo «dd/MM/yyyy HH:mm — {estado}», con el estado como **texto** («Completada», «Anulada», «Devuelta», «Pendiente»), no como chip | Siempre |
| Motivo | Motivo de Anulación: (`ventasDetail.motivoAnulacionLabel`) | `motivoAnulacion` | Si existe |
| Cabecera | Cliente (`ventasTable.colCliente`) | `clienteNombre`; la API envía «Público General» si no hay cliente (`GetVentaByIdQueryHandler.cs:72`), por lo que «Cliente Eventual (Público General)» no llega a verse | Siempre |
| | Empleado (`ventasTable.colEmpleado`) | `empleadoNombre` | Siempre |
| | Total (S/) (`ventasTable.colTotal`) | «S/ x.xx» | Siempre |
| | Verificación de receta (`ventasDetail.recetaVerificadaLabel`) | «Receta verificada por {nombre} el {dd/MM/yyyy HH:mm}» (`ventasDetail.recetaVerificadaPor`) | Si `recetaVerificadaPor` tiene valor |
| Comprobante | Título «Comprobante» (`ventasDetail.comprobanteTitle`) | «Nº Comprobante:» `{tipo} {número}` y «Estado SUNAT:» con el texto del estado (por ejemplo «Pendiente de envío a SUNAT»); no muestra emisor ni cliente | Si hay comprobante |

**Líneas** (título «Productos Vendidos», `ventasDetail.detallesTitle`): «Producto», «Lote», «Cantidad» (si la línea se vendió por presentación: «{n} × {presentación} ({unidades base})»; si no, el número), «Devuelto» (`devolucion.colDevuelto`), «P. Unitario (S/)» y «Subtotal (S/)», con 2 decimales en importes.
**Pagos** (título «Pagos Registrados»): «Método de Pago» (Efectivo, Yape / Plin, Tarjeta Débito/Crédito, Transferencia), «Monto (S/)», «Referencia de Operación» («—» si no hay) y «Fecha de Pago» (`dd/MM/yyyy HH:mm`). Debajo, «Vuelto:» y «S/ x.xx» si es mayor que 0, calculado como pagos menos el total de la venta, sin considerar cobertura de seguro (`vueltoVenta`, `VD.ts:119-122`).
**Devoluciones** (título «Devoluciones / Notas de crédito», `devolucion.sectionTitle`; solo si hay): «Nota de crédito» («—» si no), «Fecha», «Reembolsado (S/)» y «Método de reembolso».

**Botones y acciones** (encabezado de la tarjeta, a la derecha, en este orden; `VD.html:26-58`)

| Texto exacto (clave) | Visible cuando (`VD.ts`) | Deshabilitado cuando | Qué hace |
|---|---|---|---|
| Reimprimir Comprobante (`pos.actions.reimprimirComprobante`) | `puedeReimprimir()` (`:114-117`): la venta tiene comprobante y su estado no es «Anulada». Sin comprobación de permiso | Nunca | Abre el [diálogo del comprobante](#5-diálogo-del-comprobante-comprobantedialogcomponent) con los datos guardados (`:124-157`) |
| Cobrar Venta (`pos.actions.cobrar`) | `puedeCobrar()` (`:99-102`): estado «Pendiente» **y** `Ventas:Crear` **y** `Ventas:Actualizar` | Nunca | Navega a `/pos/nueva?continuar=<id>` (apartado 3.10) |
| Devolver (`pos.devolucion.btnDevolver`) | `puedeDevolver()` (`:160-166`): estado «Completada», con comprobante, alguna línea con cantidad devolvible > 0 (vendida − devuelta) **y** `Devoluciones:ProcesarDevolucion` | Nunca | Abre el [diálogo de devolución](#9-diálogo-de-devolución-devolverventadialogcomponent) (900 px, máx. 95 vw); si devuelve `true`, recarga la venta |
| Anular Venta (`pos.ventasDetail.btnAnular`), estilo de borrado | `puedeAnular()` (`:90-96`): estado «Completada» o «Pendiente» **y** sin devoluciones **y** `Ventas:Anular` | `anulando()` (con indicador de carga) | Abre el [diálogo de anulación](#8-diálogo-de-anulación-anularventadialogcomponent) y, si hay motivo, llama a la API |

Ninguno comprueba la sucursal de la venta: la API ya niega ver ventas de otra sucursal (`Venta.Forbidden`, hallazgo a4 de [05b](../05-seguridad/05b-matriz-de-permisos.md)). Una venta «Devuelta» (todo devuelto) solo permite reimprimir; una «Anulada» no ofrece botones.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la carga | Error | «Error al cargar la venta» (`pos.ventasDetail.loadError`) | `VD.ts:69` |
| Venta anulada | Éxito | «Venta anulada correctamente.» (`pos.ventasDetail.anularSuccess`); recarga la venta | `VD.ts:188` |
| Error al anular | Error | Mapa del apartado 8 | `VD.ts:195` |
| Devolución procesada | Éxito | «Devolución procesada y nota de crédito emitida.» (lo muestra el diálogo, apartado 9) | `DEV.ts:158` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Abrir y tras anular o devolver | `VentaService.getById` | `GET /api/v1/ventas/{id}` | `Ventas:Leer` (`VentasController.cs:29`) |
| Confirmar anulación | `VentaService.anular` (`VSV.ts:73-76`) | `PUT /api/v1/ventas/{id}/anular`, cuerpo = el motivo como texto JSON | `Ventas:Anular` (`VentasController.cs:75`) |
| Confirmar devolución | `VentaService.procesarDevolucion` | `POST /api/v1/devoluciones` | `Devoluciones:ProcesarDevolucion` (`DevolucionesController.cs:11`) |

**Observaciones**
- El estado no es un chip sino texto junto a la fecha.
- «Cobrar Venta» no exige caja abierta: si no la hay, el terminal mostrará el aviso de abrirla (apartado 3.1).
- El permiso `Devoluciones:ProcesarDevolucion` no lo siembra ningún rol (RN-DEV-001); el botón «Devolver» solo lo ven Admin y roles a los que se les haya dado.
- Las devoluciones solo se ven aquí: no existe pantalla de listado ni de detalle de devoluciones (RN-DEV-032).

---

## 8. Diálogo de anulación (`AnularVentaDialogComponent`)

- **Componente:** `AnularVentaDialogComponent` (`ANU.ts`, `ANU.html`). Scope `pos`.
- **Para qué sirve:** pedir el motivo antes de anular una venta. La anulación revierte el stock, anula el comprobante y, si la venta estaba cobrada, se registra desde la caja propia (RN-VEN-065 a RN-VEN-069, RN-CMP-012).
- **Cómo se abre:** botón «Anular Venta» de `/pos/:id` (`anular()`, `VD.ts:178-199`), ancho 450 px, sin `disableClose` (Esc o clic fuera equivalen a cancelar). Permiso: `Ventas:Anular` (control en el botón y en el endpoint).
- **Título y subtítulo:** «Anular Venta» (`pos.ventasDetail.anularDialogTitle`) y «Esta acción revierte el stock descontado. Ingresa el motivo de la anulación.» (`pos.ventasDetail.anularDialogSubtitle`).

**Campos del formulario** (`ANU.ts:17-19`, `ANU.html`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Valor inicial |
|---|---|---|---|---|---|---|
| Motivo de Anulación (`pos.ventasDetail.motivoLabel`) | `motivo` | `textarea` de 3 filas | Sí | `Validators.required` y `Validators.maxLength(255)` | Solo para `required`: «El motivo es obligatorio.» (`pos.ventasDetail.motivoRequired`). El `textarea` **no** tiene atributo `maxlength`: si se escriben más de 255 caracteres el formulario es inválido y «Anular Venta» no hace nada, sin mensaje | Vacío |

Un motivo de solo espacios pasa la validación del formulario (`required` solo rechaza vacío), se envía y la API lo rechaza (RN-VEN-063); la pantalla muestra entonces el mensaje genérico.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie, a la derecha | Siempre | Nunca | `onCancel()` → cierra con `null`; no se anula |
| Anular Venta (`pos.ventasDetail.btnAnular`), estilo de borrado | Pie, a la derecha | Siempre | Nunca | `onConfirm()`: marca todo como tocado; si es inválido no cierra; si es válido cierra con el texto del motivo (sin recortar) |

**Qué pasa al cerrar** (`VD.ts:178-199`): si el resultado es nulo o vacío no hace nada; si trae motivo, pone «anulando» (el botón del detalle muestra el indicador y se deshabilita) y llama a `PUT /api/v1/ventas/{id}/anular` (`Ventas:Anular`) con el motivo como texto JSON.

**Mensajes al usuario y mapeo de errores** (`ANULAR_ERROR_KEYS`, `VD.ts:19-23`)

| Código / situación | Tipo | Texto exacto (clave) |
|---|---|---|
| Éxito | Éxito | «Venta anulada correctamente.» (`pos.ventasDetail.anularSuccess`); recarga el detalle |
| `Venta.Anular.ConDevoluciones` (la venta ya tiene devoluciones, RN-VEN-066) | Error | «Esta venta tiene devoluciones registradas y ya no puede anularse. Gestiona los cambios desde Devoluciones.» (`pos.ventasDetail.anularConDevoluciones`) |
| `Venta.Anular.SinSesionAbierta` (venta cobrada y el usuario no tiene su propia caja abierta en esa sucursal, RN-VEN-067) | Error | «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» (`pos.ventasDetail.anularSinSesionAbierta`) |
| `Concurrency.Conflict` | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`) |
| Cualquier otro código (incluye `Venta.Anular` por estado no anulable o de otra sucursal, motivo en blanco) | Error | «Error al anular la venta.» (`pos.ventasDetail.anularError`) |

**Observaciones:** el texto de `anularConDevoluciones` remite a «Devoluciones», que no es una opción de menú ni una pantalla (las devoluciones se procesan desde este mismo detalle). En una venta «Pendiente» (en espera) la anulación no exige caja abierta; sí en una «Completada».

---

## 9. Diálogo de devolución (`DevolverVentaDialogComponent`)

- **Componente:** `DevolverVentaDialogComponent` (`DEV.ts`, `DEV.html`). Scope `pos`. Recibe la venta completa (`DevolverVentaDialogData`).
- **Para qué sirve:** devolver total o parcialmente las líneas de una venta cobrada. Emite una nota de crédito y registra el reembolso, que sale de la caja propia abierta (RN-DEV-005, RN-DEV-019, RN-DEV-020).
- **Cómo se abre:** botón «Devolver» de `/pos/:id` (apartado 7), ancho 900 px y máximo 95 vw. Permiso: `Devoluciones:ProcesarDevolucion`.
- **Título y subtítulo:** «Procesar devolución — {código de la venta}» (`pos.devolucion.dialogTitle`) y «Indica cuántas unidades devuelve el cliente en cada línea y su destino. Se emitirá una nota de crédito y el reembolso saldrá de tu caja abierta.» (`pos.devolucion.dialogSubtitle`).

**Tabla de líneas** (`DEV.ts:69-83`; solo líneas con cantidad devolvible mayor que 0, es decir `cantidad vendida − cantidad devuelta`, en **unidades base**)

| Encabezado (clave) | Dato | Control |
|---|---|---|
| Producto (`pos.ventasDetail.colProducto`) | Nombre | Texto |
| Lote (`pos.ventasDetail.colLote`) | Número de lote | Texto |
| Vendido (`pos.devolucion.colVendido`) | Cantidad vendida | Texto |
| Devuelto (`pos.devolucion.colDevuelto`) | Ya devuelto | Texto |
| Devolvible (`pos.devolucion.colDevolvible`) | Máximo que se puede devolver | Texto |
| A devolver (`pos.devolucion.colCantidad`) | — | Número, `min=0`, `max` = devolvible, 96 px; valor inicial 0. Validadores: requerido, mínimo 0 y máximo = devolvible. Al salir del campo (`change`) la cantidad se **ajusta** a un valor entre 0 y el devolvible (`capCantidad`, `DEV.ts:109-115`) |
| Destino (`pos.devolucion.colDestino`) | — | Select de 190 px: «Reingreso a venta» (`pos.devolucion.destino.reingreso`, valor por defecto) o «Cuarentena DIGEMID» (`destino.cuarentena`) |

Sobre la tabla, a la derecha, el botón «Devolver todo lo pendiente» (`pos.devolucion.btnDevolverTodo`) pone en cada línea su cantidad devolvible (`devolverTodo()`, `DEV.ts:117-119`).

**Campos del formulario** (`DEV.ts:85-89`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Valor inicial |
|---|---|---|---|---|---|---|
| Motivo SUNAT (`pos.devolucion.motivoLabel`) | `motivo` | Select: «06 — Devolución total» y «07 — Devolución por ítem» (`pos.devolucion.motivo.06`, `.07`) | Sí | `required` | — | «07». **Se recalcula solo** cada vez que cambian las cantidades: «06» si cada línea tiene su cantidad devolvible completa, «07» en cualquier otro caso (`DEV.ts:100-106`); el cajero puede cambiarlo a mano, pero el siguiente cambio de cantidades lo vuelve a fijar |
| Método de reembolso (`pos.devolucion.metodoReembolsoLabel`) | `metodoReembolso` | Select: «Efectivo», «Yape / Plin», «Tarjeta Débito/Crédito», «Transferencia» | Sí | — | — | Efectivo |
| Sustento (`pos.devolucion.sustentoLabel`) | `sustento` | `textarea` de 2 filas, `maxlength="255"` | Sí | `required`, `maxLength(255)`; además `onConfirm` rechaza un texto de solo espacios | «El sustento es obligatorio.» (`pos.devolucion.sustentoRequired`), solo para `required`. Un sustento de solo espacios no muestra mensaje: el botón no hace nada | Vacío |

**Reembolso estimado** (`pos.devolucion.montoEstimado`, «S/ x.xx» en negrita, a la derecha): `Σ (cantidad a devolver × precio unitario de la línea)` (`montoEstimado`, `DEV.ts:94-96`). Es el crédito de la nota; la API calcula el reembolso real como ese monto por la proporción que pagó el cliente (descontando cobertura de seguro, RN-DEV-016, RN-DEV-017), por lo que coinciden salvo ventas con seguro y redondeos.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie, a la derecha | Siempre | `saving()` | `onCancel()` → cierra con `false` |
| Procesar devolución (`pos.devolucion.btnConfirmar`); con indicador de carga | Pie, a la derecha | Siempre | `saving()` | `onConfirm()` (`DEV.ts:143-168`): marca todo como tocado; sin ninguna cantidad mayor que 0 → aviso y no envía; formulario inválido o sustento en blanco → no envía; si no, `POST /api/v1/devoluciones` |

Cuerpo enviado (`buildRequest`, `DEV.ts:125-141`): `comprobanteOrigenId` = **id de la venta** (así lo espera la API), `motivoSunatCatalogo`, `sustentoDescriptivo` (recortado), `metodoReembolso` y `detalles` con `detalleVentaId`, `cantidadDevuelta` y `destinoFisicoLogico` solo de las líneas con cantidad mayor que 0.

**Mensajes al usuario** (mapa `DEVOLVER_ERROR_KEYS`, `DEV.ts:22-32`)

| Código / situación | Tipo | Texto exacto (clave) |
|---|---|---|
| Ninguna cantidad mayor que 0 | Error | «Ingresa al menos una cantidad a devolver.» (`pos.devolucion.sinCantidades`) |
| Éxito | Éxito | «Devolución procesada y nota de crédito emitida.» (`pos.devolucion.success`); cierra con `true` y el detalle se recarga |
| `Venta.NotFound` | Error | «La venta ya no existe.» (`pos.devolucion.errors.ventaNotFound`) |
| `Devolucion.Venta.OtraSucursal` | Error | «Solo puedes devolver ventas de tu propia sucursal.» (`errors.otraSucursal`) |
| `Devolucion.Caja.SinSesionAbierta` | Error | «Necesitas una caja abierta para procesar la devolución.» (`errors.sinSesionAbierta`) |
| `Devolucion.ComprobanteOrigen.NoEmitido` | Error | «La venta no tiene boleta ni factura emitida que acreditar.» (`errors.comprobanteNoEmitido`) |
| `Devolucion.SerieNotaCredito.NoConfigurada` | Error | «La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales.» (`errors.serieNcNoConfigurada`) |
| `Devolucion.Venta.EstadoInvalido` | Error | «Solo se pueden devolver ventas completadas.» (`errors.estadoInvalido`) |
| `Devolucion.Cantidad.Excedida` | Error | «La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.» (`errors.cantidadExcedida`) |
| `DetalleVenta.NotFound` | Error | «Una de las líneas ya no pertenece a la venta.» (`errors.detalleNotFound`) |
| `Concurrency.Conflict` | Error | «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`) |
| Cualquier otro código | Error | «Error al procesar la devolución.» (`pos.devolucion.error`) |

**Llamadas a la API:** `POST /api/v1/devoluciones` (`VentaService.procesarDevolucion`, `VSV.ts:79-81`), permiso `Devoluciones:ProcesarDevolucion` (`DevolucionesController.cs:11`).

**Observaciones:** «Cuarentena DIGEMID» solo deja anotado el destino: la API no repone stock ni crea un registro de cuarentena (RN-DEV-024), mientras que «Reingreso a venta» suma las unidades al mismo lote de la sucursal (RN-DEV-023). El reembolso en efectivo afecta el arqueo de la caja (RN-CAJ-014). No hay plazo máximo para devolver (RN-DEV-008). Las cantidades se expresan en unidades base aunque la línea se haya vendido por presentación.

---

## 10. Pendientes y «No verificado»

| # | Tema | Detalle |
|---|---|---|
| 1 | Comportamientos no ejecutados | Todo el documento sale de lectura de código; no se usó el navegador. Marcados **No verificado** en el texto: Enter en «Cantidad» agrega al carrito (3.2); redondeo de un centavo entre subtotal, IGV y total (3.3); montos de pago que no se re-sincronizan al cambiar el carrito (3.7); cliente que no se puede quitar (3.4); aplicación de una aseguradora guardada en un borrador antiguo (3.8); acceso a `localStorage` sin protección (3.9); cantidad en presentaciones sin tope al sumar líneas (3.15) |
| 2 | Semántica exacta de `soloStockVendibleEnMiSucursal` | Se cita el comentario del SPA; no se abrió el handler de medicamentos |
| 3 | Orden de las condiciones de venta | El chip usa el índice como valor del filtro; coincide con el enumerado mientras el backend conserve el orden (`CondicionVenta.cs:7-10`) |
| 4 | Formato de números | El pipe `number: '1.2-2'` formatea según el idioma activo; no se verificó el separador decimal efectivo |
| 5 | Código de venta y tipo del comprobante en pantalla | Se describe la forma derivada del backend (`{prefijo}-{correlativo:D8}`); no se vio un ejemplo real |
| 6 | Rutas y estados de texto | El layout horizontal (opción «POS / Ventas») no se documenta aquí (19a §4.7) |
| 7 | Cambios sin commitear | Ninguno: `sofia-apis` `43f5529` y `sofia-web-spa` `eeffbb6` sin cambios locales; no hay marcas «En curso (sin commitear)» |

---

## 11. Tabla de cobertura

### Rutas (`rutas-spa.csv`)

| Ruta | Ficha |
|---|---|
| `/sesiones-caja` | 1.1 |
| `/sesiones-caja/nueva` | 1.2 |
| `/sesiones-caja/:id` | 1.3 |
| `/pos` | 6 |
| `/pos/nueva` (y `?continuar=`) | 3 (3.10 para `?continuar=`) |
| `/pos/:id` | 7 |

### Componentes (`componentes-pages.txt`) y diálogos (`dialogos.txt`)

| Componente | Ficha |
|---|---|
| `SesionesSearchComponent` | 1.1 |
| `SesionesDetailComponent` | 1.2 y 1.3 |
| `VentasSearchComponent` | 6 |
| `NuevaVentaComponent` | 3 |
| `VentaDetailComponent` | 7 |
| `AperturarCajaDialogComponent` | 2 |
| `ConfirmarVentaDialogComponent` | 4 |
| `ComprobanteDialogComponent` | 5 |
| `AnularVentaDialogComponent` | 8 |
| `DevolverVentaDialogComponent` | 9 |

No hay más componentes ni diálogos en las dos carpetas (apartado 0.1).

### Usos de `*appPermiso` (`app-permiso-usos.csv`)

| Archivo y línea | Expresión | Dónde quedó documentado |
|---|---|---|
| `nueva-venta.component.html:17` | `'POS:AperturarCaja'` | 3.1 (botón «Aperturar Caja») |
| `nueva-venta.component.html:443` | `'Recetas:Crear'` | 3.5 (enlace «Registrar receta») |
| `comprobante-dialog.component.html:38` | `'Ventas:Crear'` | 5.1 (botón «Nueva Venta POS») |
| `ventas-search.component.html:17` | `'Crear'` (= `Ventas:Crear`) | 6 (botón «Nueva Venta POS») |
| `sesiones-search.component.html:20` | `'AperturarCaja'` (= `POS:AperturarCaja`) | 1.1 (botón «Nuevo») |
| `sesiones-detail.component.html:76` | `'CerrarCaja'` (= `POS:CerrarCaja`) | 1.3 (botón «Cerrar Sesión») |

### Otros archivos del inventario 0.1

| Archivo | Dónde |
|---|---|
| `venta.service.ts`, `sesion-caja.service.ts` | Tablas «Llamadas a la API» de cada ficha |
| `receipt-print.service.ts` | 5.3 |
| `pos-draft.ts` | 3.9 |
| `vuelto.util.ts` | 3.7 |
| `venta.model.ts`, `sesion-caja.model.ts` | Enumerados y DTO citados en cada ficha |
| `*.routes.ts`, `*.spec.ts`, `*.scss` | Rutas en 0.2; pruebas y estilos no se documentan |

---

## 12. Claves i18n sin uso

Comprobado buscando cada clave en `src/` (archivos `.ts` y `.html`, sin pruebas), con coincidencia exacta de la clave completa o relativa. Usadas de forma dinámica (no son huérfanas): `pos.estado.*` (`pos.estado.${estado}`, `t('estado.' + …)`), `pos.checkout.efectivo|yapePlin|tarjeta|transferencia` (`metodoPagoLabel`), `pos.devolucion.motivo.06|07` (`'devolucion.motivo.' + m`), `pos.estadoComprobante.*` (`estadoComprobanteKey`) y `sesionesCaja.estado.*`.

### Scope `pos` (`public/i18n/pos/es.json`)

| Grupo | Claves sin uso | Comentario |
|---|---|---|
| Raíz | `title` («Gestión de Caja y Punto de Venta POS») | Ninguna plantilla de `pos` la usa |
| `actions` | `cerrar`, `cancelar`, `confirmar`, `verDetalle`, `imprimirReporte`, `aceptar` | Los diálogos usan `actions.cancel` de la raíz |
| `search` | `title`, `searchPlaceholder`, `statusActive`, `statusClosed`, `statusCuadrada`, `assignedBranch`, `cajeroDefault`, `sucursalDefault`, `loadingBranch`, `noSessions`, `showing` | Restos de una pantalla de sesiones dentro de `pos`; la actual vive en `sesiones-caja` |
| `table` | `colSucursal`, `colEmpleado`, `colApertura`, `colCierre`, `colMontoApertura`, `colMontoCierre`, `colEstado`, `colProducto`, `colStock`, `colAcciones`, `faltanteText`, `sobranteText` | Igual |
| `ventasDetail` | `subtitle` | |
| `checkout` | `catalogTitle`, `clienteAutocompleteNoResults`, `aseguradoraSearchPlaceholder`, `montoRecibidoLabel`, `vueltoLabel` | |
| `transferDialog` | `title`, `sucursalOrigenLabel`, `sucursalOrigenRequired`, `cantidadLabel`, `cantidadRequired`, `cantidadMin`, `observacionesLabel`, `observacionesPlaceholder`, `submit`, `sucursalAvailable` | Todo el grupo: no existe diálogo de transferencia |
| `dialogs` | `aperturaTitle`, `aperturaSubtitle`, `montoInicialLabel`, `montoInicialRequired`, `montoInicialInvalid`, `cierreTitle`, `cierreSubtitle`, `montoAperturaInicial`, `ventasEfectivoTurno`, `cobrosDigitales`, `efectivoEsperado`, `montoCierreLabel`, `montoCierreRequired`, `resultadoArqueo`, `cuadreExacto`, `faltanteMonto`, `sobranteMonto` | Todo el grupo (diálogos de apertura y cierre antiguos) |
| `cobrarDialog` | `title`, `subtitle`, `totalLabel`, `btnCobrar` | Todo el grupo: no existe diálogo de cobro de venta en espera |
| `confirmarDialog` | `recibidoLabel` | |
| `reportDialog` | `title`, `subtitle`, `companyTitle`, `rucSubheader`, `sesionNumeroLabel`, `fechaAperturaLabel`, `fechaCierreLabel`, `montoAperturaLabel`, `ventasEfectivoLabel`, `cobrosDigitalesLabel`, `efectivoEsperadoLabel`, `efectivoDeclaradoLabel`, `resultadoFinalHeader`, `cuadrePerfectoText`, `faltanteText`, `sobranteText`, `firmaCajero`, `firmaSupervisor` | Solo `reportDialog.cajeroLabel` se usa (en el ticket) |
| `messages` | `aperturaSuccess`, `cierreSuccess`, `ventaSuccess`, `cartEmptyWarning`, `cajaRequiredWarning`, `cajaAlreadyOpenWarning`, `noSucursalFound`, `noEmpleadoFound`, `errorEmpleadosApi`, `errorSucursalesApi`, `errorGeneric`, `noStockLocal`, `noStockAnywhere`, `otherBranchAvailableInfo`, `transferRequestedSuccess`, `transferRequestedError`, `montoInsuficiente`, `sumaPagosInsuficiente`, `vueltoSinEfectivoError`, `cobrarSuccess`, `cobrarError` | |

### Scope `sesiones-caja` (`public/i18n/sesiones-caja/es.json`)

| Clave sin uso | Comentario |
|---|---|
| `create.title` («Nueva Sesión de Caja») | La pantalla `/sesiones-caja/nueva` usa el título «Sesiones de Caja» |

Las demás claves de `sesiones-caja` se usan en `SS.*` y `SD.*`.

### Otros hallazgos de texto

- Claves inexistentes: no se detectó ninguna clave usada por `pos` o `sesionesCaja` que falte en su `es.json` (comprobado para las claves de scope; las de la raíz y de `recetasMedicas` se tomaron del `es.json` correspondiente).
- Textos en duro: el símbolo «S/» en las plantillas de `NV`, `CMP`, `CNF`, `VD` y `DEV`.

---

## 13. Fuentes

- Código del SPA (`sofia-web-spa/src/app/pages/pos/**` y `sofia-web-spa/src/app/pages/sesiones-caja/**`): componentes, plantillas, servicios, modelos, `pos-draft.ts`, `vuelto.util.ts`, `receipt-print.service.ts`, rutas.
- `sofia-web-spa/src/app/core/auth/`: `route-permissions.ts`, `permission.guard.ts`, `auth.service.ts`, `user-storage.ts`; `src/app/shared/directives/permiso.directive.ts`, `src/app/shared/utils/tipo-documento.util.ts`, `src/app/shared/utils/date-only.util.ts`; `src/app/core/services/shared/error-notifier.service.ts`; `src/app/matdash/layouts/full/**/sidebar/sidebar-data.ts`; `src/app/pages/dashboard/dashboard-links.ts` y `dashboard.component.html`; `src/styles.scss`; `src/app/app.routes.ts`.
- Servicios de otros módulos consultados: `medicamentos`, `pacientes`, `recetas-medicas`, `empleados` (`*.service.ts` y modelos).
- Textos: `public/i18n/pos/es.json`, `public/i18n/sesiones-caja/es.json`, `public/i18n/recetas-medicas/es.json`, `public/i18n/dashboard/es.json`, `public/i18n/es.json`.
- Backend (solo lectura): `SOFIA.API/Controllers/{POS,Ventas,Devoluciones,Medicamentos,Pacientes,Recetas,Seguros,Empleados}Controller.cs`; `SOFIA.Application/Ventas/Queries/GetVentas/GetVentasQueryHandler.cs`, `GetVentaById/GetVentaByIdQueryHandler.cs`; `Ventas/Common/VentaComprobanteGenerator.cs`, `VentaDetalleFactory.cs`; `Ventas/Commands/{CreateVenta,CompletarVenta,ActualizarVentaPendiente}`; `SOFIA.Domain/Entities/{Venta,Medicamento}.cs`; `SOFIA.Domain/Enums/CondicionVenta.cs`.
- Inventarios: `docs/_inventario/{rutas-spa.csv, componentes-pages.txt, dialogos.txt, app-permiso-usos.csv, endpoints.csv, codigos-error.csv, codigos-error-ocurrencias.csv}`.
- Documentos validados enlazados: [01c2](../01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [01c3](../01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [05b](../05-seguridad/05b-matriz-de-permisos.md), [19a](19a-acceso-layout-y-compartidos.md), [19b](19b-administracion.md).
