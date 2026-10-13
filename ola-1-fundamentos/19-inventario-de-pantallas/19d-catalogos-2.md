# Inventario de pantallas del SPA — Parte D: catálogos II

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-13 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código del árbol de trabajo del SPA (componentes, plantillas, servicios, modelos e i18n). Del backend solo se abrieron controladores, validadores, manejadores y entidades necesarios para anotar permisos, códigos de error y comparar validaciones. No se usó el navegador ni la API local. Al empezar, `git status --short` salió vacío en ambos repos (sin cambios sin commitear); `git diff --stat 01b8260..HEAD -- src/app/pages` en el SPA no muestra cambios en las carpetas de este documento (los commits `8f36125` y `eeffbb6` solo tocan README, `angular.json` e `index.html`). |

Este documento es la parte D de seis del inventario de pantallas. Cubre cinco catálogos: Medicamentos, Presentaciones de venta, Formulaciones clínicas, Proveedores (con el registro e historial de precios de compra) y Aseguradoras (18 rutas). Lo compartido (guards, interceptor, `page-header`, `confirm-dialog`, `range-slider-field`, carga masiva común, textos de la raíz) lo documenta la [parte A](19a-acceso-layout-y-compartidos.md); los comportamientos comunes de las pantallas de búsqueda y detalle de catálogo están en la [parte C, §0](19c-catalogos-1.md) y aquí solo se enlazan. Las reglas de negocio están en [01b](../01-reglas-de-negocio/01b-catalogos.md) (MED, PRE, FOR, PRV, ASE) y los permisos por endpoint en [05b](../05-seguridad/05b-matriz-de-permisos.md).

Convenciones: las rutas de archivo son relativas a `sofia-web-spa/` salvo que empiecen por `sofia-apis/`. `app/` abrevia `src/app/`. Entre paréntesis va la clave i18n; las claves de scope se escriben con su prefijo real (por ejemplo `medicamentos.fields.codigoNacional` está en `public/i18n/medicamentos/es.json`, clave `fields.codigoNacional`). El scope `presentaciones-venta` se expone en camelCase como `presentacionesVenta` (las plantillas lo declaran como `scope: 'presentaciones-venta'; read: 'presentacionesVenta'` y el código usa `presentacionesVenta.…`; `provideTransloco` no activa `keepCasing`, `app/app.config.ts:51-60`). Las plantillas abren con `*transloco="let t; scope: '<scope>'; read: '<clave>'"`, así que `t('x')` equivale a `<clave>.x`. Los números de línea de `.ts` y `.html` son los del árbol de trabajo (que coincide con `HEAD`).

## Contenido

0. Diferencias respecto a los comportamientos comunes de la parte C
1. Medicamentos: `/medicamentos`, `/medicamentos/nueva`, `/medicamentos/carga-masiva`, `/medicamentos/:id`
2. Presentaciones de venta: `/presentaciones-venta`, `/presentaciones-venta/nueva`, `/presentaciones-venta/:id` y el diálogo del producto
3. Formulaciones clínicas: `/formulaciones-clinicas`, `/formulaciones-clinicas/nueva`, `/formulaciones-clinicas/:id`
4. Proveedores: `/proveedores`, `/proveedores/nueva`, `/proveedores/carga-masiva`, `/proveedores/:id`, sección de precios y diálogo `precio-proveedor-dialog`
5. Aseguradoras: `/aseguradoras`, `/aseguradoras/nueva`, `/aseguradoras/carga-masiva`, `/aseguradoras/:id`
6. Eliminar: mapa de estados y mensajes de las cinco pantallas
7. Pendientes y «No verificado»
8. Tabla de cobertura
9. Claves i18n sin uso
10. Fuentes

---

## 0. Diferencias respecto a los comportamientos comunes de la parte C

Todo lo descrito en [19c §0](19c-catalogos-1.md) vale aquí salvo lo que se indica. Se repiten solo los puntos que cambian o que conviene tener a mano.

- **Permiso de ruta.** Mismo mecanismo (`canActivateChild: [passwordChangeGuard, permissionGuard]`, `app/app.routes.ts:37`; `app/core/auth/route-permissions.ts`): `Leer` del módulo del primer segmento; `nueva`, `nuevo` y `carga-masiva` exigen además `Crear`; `/x/:id` solo `Leer`. Sin permiso: snackbar «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`) y redirección a `/dashboard`. Módulos de esta parte (`route-permissions.ts:12-16`):

  | Primer segmento | Módulo de permisos |
  |---|---|
  | `medicamentos` | `Medicamentos` |
  | `presentaciones-venta` | `PresentacionesVenta` |
  | `formulaciones-clinicas` | `FormulacionesClinicas` |
  | `proveedores` | `Proveedores` |
  | `aseguradoras` | **`Seguros`** (el controlador y los permisos se llaman «Seguros»; la pantalla «Aseguradoras») |

- **Menú.** Las cinco opciones están en el grupo «Catálogos» (`breadcrumbs.catalogos`): «Medicamentos» (`nav.medicamentos`), «Presentaciones de Venta» (`nav.presentacionesVenta`), «Formulaciones Clínicas» (`nav.formulacionesClinicas`), «Proveedores» (`nav.proveedores`) y «Aseguradoras» (`nav.aseguradoras`); fuente `app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:74,79,84,89,94`. Una opción se muestra solo con `Leer` del módulo de su ruta ([19a §4.3](19a-acceso-layout-y-compartidos.md)).
- **Directiva `*appPermiso`.** Con un solo valor toma el módulo de la URL actual; con `Modulo:Accion` usa ese módulo (`app/shared/directives/permiso.directive.ts:16-23`). Esta parte usa también la forma `Modulo:Accion` (Medicamentos, Presentaciones y Proveedores).
- **Fila sin resultados.** Medicamentos y Presentaciones de venta muestran «Sin resultados» (`generic-labels.no-results`); Formulaciones clínicas, Proveedores y Aseguradoras muestran «Resultados» (`generic-labels.results`). Detalle en cada ficha.
- **Errores de exportación.** Medicamentos y Presentaciones usan `showError` (sin referencia de seguimiento); Formulaciones, Proveedores y Aseguradoras usan `showServerError` (con «Referencia: …» si la respuesta trae `traceId`).
- **Mapa de errores al guardar** (`saveErrorKey`, ver [19c §0](19c-catalogos-1.md)): lo usan Medicamentos, Proveedores y el diálogo de precios. **Aseguradoras, Presentaciones de venta y Formulaciones clínicas no lo usan** (cada una decide por estado HTTP o muestra un mensaje fijo; ver sus fichas). Detalle de los tres textos `errors.db*`: «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» (`errors.dbDuplicado`), «El registro hace referencia a datos que ya no existen o todavía está en uso.» (`errors.dbReferenciaInvalida`), «Uno de los valores está fuera del rango permitido.» (`errors.dbValorInvalido`).
- **Estado de la búsqueda.** Igual que en 19c (`SearchStateService`, clave = URL sin query), con una excepción: en Proveedores el deslizador de tasa de cumplimiento **no** se guarda ni se restaura (§4.1).
- **«Buscar» no vuelve a la página 1.** En las cinco búsquedas `search()` usa `pageIndex() + 1`.
- **Carga masiva.** Solo Medicamentos, Proveedores y Aseguradoras la tienen. Usan `app-carga-masiva-page` ([19a §6.5 y §6.6](19a-acceso-layout-y-compartidos.md)); título «Carga Masiva» (`carga-masiva.title`), plantilla descargada como `plantilla-{backRoute sin la barra}.xlsx` (`carga-masiva.template-filename`), etiquetas de columna resueltas contra la **raíz** `fields.*`, y el campo `required` de cada columna no se lee. Presentaciones de venta y Formulaciones clínicas **no** tienen carga masiva.
- **Botón volver / migas.** Igual que 19c: en las búsquedas sin botón volver; en detalle y carga masiva, botón con icono `chevron_left` que va a la búsqueda.

---

## 1. Medicamentos

Carpeta `app/pages/medicamentos`, scope i18n `medicamentos` (`provideTranslocoScope('medicamentos')` en los componentes). Componentes enrutados (`app/pages/medicamentos/medicamentos.routes.ts`, `loadChildren` desde `app/app.routes.ts:96-98`): `MedicamentosSearchComponent` (`''`), `MedicamentosDetailComponent` (rutas `nueva` y `:id`) y `MedicamentosCargaMasivaComponent` (`carga-masiva`). El orden importa: `nueva` y `carga-masiva` están antes de `:id`.

Qué **no** tiene este módulo (comprobado en el modelo `app/pages/medicamentos/models/medicamento.model.ts`, la plantilla y el backend `sofia-apis/src/SOFIA.Domain/Entities/Medicamento.cs:10-20`): no hay campo de estado/activo, no hay vínculo con el catálogo DIGEMID, y las búsquedas no tienen filtros numéricos con deslizador ni de fecha. Los campos son los seis que se detallan abajo; el stock se muestra solo en lectura.

### 1.1 `/medicamentos` — Medicamentos (búsqueda)

- **Componente:** `MedicamentosSearchComponent` (`app/pages/medicamentos/medicamentos-search/medicamentos-search.component.ts`, `.html`). Scope `medicamentos`.
- **Para qué sirve:** buscar medicamentos por código nacional, nombre comercial, laboratorio, unidad base y condición de venta; abrirlos, exportar la lista y entrar a crear o a cargar medicamentos masivamente.
- **Cómo se llega:** menú «Catálogos» → «Medicamentos»; botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar.
- **Permiso de ruta:** `Medicamentos:Leer`. Sin permiso: aviso y redirección a `/dashboard`.
- **Migas y cabecera:** título «Medicamentos» (`medicamentos.title`); migas «Inicio» (`breadcrumbs.home`, enlaza a `/dashboard`) / «Catálogos» (`breadcrumbs.catalogos`) / «Medicamentos» (`breadcrumbs.medicamentos`, activa); sin botón volver (`.html:3-6`, `.ts:65-69`). Tarjeta de filtros con título «Búsqueda de Medicamentos» (`medicamentos.search.title`) y subtítulo «Ingresa los criterios de búsqueda» (`generic-labels.search-subtitle`).
- **Modos:** uno solo (búsqueda). Los resultados aparecen tras «Buscar» o por estado guardado.

**Filtros de búsqueda** (`.ts:57-63`, `.html:37-143`)

| Etiqueta (clave) | Control | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Código Nacional (`medicamentos.fields.codigoNacional`) | `codigoNacional` | Texto con lupa; Enter busca | Texto libre; el servidor compara «contiene» sin distinguir mayúsculas (`GetMedicamentosQueryHandler.cs:81-85`) | Vacío |
| Nombre Comercial (`medicamentos.fields.nombreComercial`) | `nombreComercial` | Texto con lupa; Enter busca | Igual | Vacío |
| Laboratorio (`medicamentos.fields.laboratorio`) | `laboratorioNombre` | Texto con lupa y autocompletado; placeholder «Buscar laboratorio...» (`medicamentos.autocomplete.laboratorio-placeholder`) | Al escribir (espera 300 ms) llama a `GET /api/v1/laboratorios?nombreCompania=<texto>&pageNumber=1&pageSize=20` y lista el nombre de compañía; si no hay coincidencias y hay texto muestra una opción deshabilitada «Sin resultados» (`medicamentos.autocomplete.no-results`). Con texto vacío no consulta. Elegir una opción solo copia el nombre al campo: la búsqueda se hace por **nombre «contiene»**, no por identificador | Vacío |
| Unidad Base (`medicamentos.fields.unidadBase`) | `unidadBaseNombre` | Texto con lupa y autocompletado; placeholder «Buscar unidad de medida...» (`medicamentos.autocomplete.unidad-placeholder`) | Igual con `GET /api/v1/unidadesmedida?descripcion=<texto>&pageNumber=1&pageSize=20`; cada opción muestra el código (si existe) en gris y la descripción | Vacío |
| Condición de Venta (`medicamentos.fields.condicionVenta`) | `condicionVenta` | Select | Primera opción «— Todos —» (`generic-labels.all`, valor nulo) y las cuatro condiciones que devuelve `GET /api/v1/medicamentos/condiciones-venta`: «Venta Libre (OTC)», «Receta Simple», «Receta Retenida», «Estupefaciente» (en ese orden; el valor enviado es la posición 0 a 3) | «Todos» |

Los textos de las condiciones **no están en el SPA**: vienen de la API (`Medicamento.CondicionesValidas`, construido con los `[Description]` del enum `CondicionVenta`, `sofia-apis/src/SOFIA.Domain/Enums/CondicionVenta.cs:7-10`, `Medicamento.cs:27-31`; endpoint `MedicamentosController.cs:24`). Si la carga de la lista falla, la búsqueda no avisa (`ngOnInit` solo tiene `next`, `.ts:99-102`): el select queda solo con «Todos» y la columna de condición mostraría el número.

**Tabla de resultados** (`.html:163-240`, columnas `.ts:71`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Código Nacional (`medicamentos.table.codigoNacional`) | `codigoNacional` | Texto | Sí: `[routerLink]="[row.id]"` → `/medicamentos/:id` |
| Nombre Comercial (`medicamentos.table.nombreComercial`) | `nombreComercial` | Texto | No |
| Laboratorio (`medicamentos.table.laboratorio`) | `laboratorioNombre` | Texto | No |
| Unidad Base (`medicamentos.table.unidadBase`) | `unidadBaseNombre` | Texto | No |
| Condición de Venta (`medicamentos.table.condicionVenta`) | `condicionVenta` | Texto de la lista de condiciones (o el número si la lista no cargó) | No |

La tabla **no muestra el precio de venta base ni el stock** (la búsqueda no pide `incluirStock`). Orden por defecto: lo fija el servidor, por nombre comercial (`GetMedicamentosQueryHandler.cs:33`); la lista excluye medicamentos cuyo laboratorio o unidad base estén eliminados (`:77`). Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Sin resultados» (`generic-labels.no-results`, `.html:220`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`), icono `upload` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `Medicamentos:Crear`, `.html:19`) | Nunca | `[routerLink]="['carga-masiva']"` → `/medicamentos/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/medicamentos/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` (`.ts:148-157`): vacía el formulario, las listas de autocompletado y el estado guardado; oculta la tabla |
| Buscar (`actions.search`); spinner mientras carga | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/medicamentos` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/medicamentos/exportar` |
| Enlace con el código nacional | Primera columna | Con resultados | — | Navega a `/medicamentos/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda (cualquier error) | Error | «Error al buscar medicamentos» (`medicamentos.search.search-error`) | `medicamentos-search.component.ts:143` |
| Falla la exportación (cualquier error) | Error | El mismo texto, sin referencia de seguimiento (`showError`) | `medicamentos-search.component.ts:205` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Al abrir (lista de condiciones) | `MedicamentoService.getCondicionesVenta` (`app/pages/medicamentos/services/medicamento.service.ts`) | `GET /api/v1/medicamentos/condiciones-venta` | `Medicamentos:Leer` (`MedicamentosController.cs:24`) |
| Buscar, paginar o restaurar estado | `MedicamentoService.search` | `GET /api/v1/medicamentos?pageNumber&pageSize[&codigoNacional][&nombreComercial][&laboratorioNombre][&unidadBaseNombre][&condicionVenta]` | `Medicamentos:Leer` (`:28`) |
| Autocompletar laboratorio | `LaboratorioService.search` | `GET /api/v1/laboratorios?nombreCompania=…&pageNumber=1&pageSize=20` | `Laboratorios:Leer` |
| Autocompletar unidad base | `UnidadMedidaService.search` | `GET /api/v1/unidadesmedida?descripcion=…&pageNumber=1&pageSize=20` | `UnidadesMedida:Leer` |
| Exportar | `MedicamentoService.exportar` | `POST /api/v1/medicamentos/exportar` (blob) | `Medicamentos:Leer` (`:76`) |

Las dos listas de autocompletado dependen de permisos de **otros módulos** (`Laboratorios:Leer`, `UnidadesMedida:Leer`). El SPA no los comprueba ni maneja el error de esas peticiones dentro de la suscripción (`.ts:73-92`); qué ve un usuario sin ellos: ver «No verificado» (§7).

**Exportación**
- Cuerpo (`.ts:177-196`): `headers` con 5 textos traducidos, en orden: «Código Nacional» (`medicamentos.table.codigoNacional`), «Nombre Comercial», «Laboratorio», «Unidad Base», «Condición de Venta»; más los cinco filtros si tienen valor (`condicionVenta` como número). No envía formato de fecha (no hay fechas).
- Archivo: «medicamentos.xlsx» (`medicamentos.search.export-filename`). El Excel del servidor lleva solo las cinco columnas, con la condición en texto (`MedicamentosController.cs:76-109`); no incluye precio ni stock.
- Carga masiva: botón «Carga Masiva» → §1.3.

**Comportamientos particulares:** restauración de filtros al volver del detalle (se guarda el formulario completo, incluida la condición de venta); «Buscar» no vuelve a la página 1 (§0).

**Observaciones**
- Los specs `medicamentos-search.component.spec.ts` (2 casos) cubren la visibilidad de botones según permisos.
- No hay filtro por precio ni por stock; el módulo Medicamentos no ofrece los deslizadores que sí tienen otras búsquedas (Proveedores, Jerarquías).

### 1.2 `/medicamentos/nueva` y `/medicamentos/:id` — Medicamentos (detalle)

- **Componente:** `MedicamentosDetailComponent` (`app/pages/medicamentos/medicamentos-detail/medicamentos-detail.component.ts`, `.html`). Scope `medicamentos`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un medicamento, ver sus datos y su stock por sucursal, y con permisos editarlo, eliminarlo o abrir sus presentaciones de venta.
- **Cómo se llega:** `/medicamentos/nueva`: botón «Nuevo» de la búsqueda. `/medicamentos/:id`: enlace del código nacional en la tabla.
- **Permiso de ruta:** `/medicamentos/nueva` → `Medicamentos:Leer` + `Medicamentos:Crear`; `/medicamentos/:id` → `Medicamentos:Leer`.
- **Migas y cabecera:** título «Medicamentos» (`medicamentos.title`); migas «Inicio» / «Catálogos» / «Medicamentos» (con enlace a `/medicamentos`); botón volver a `/medicamentos` (`.html:3-7`, `.ts:85-89`). Misma cabecera en crear, ver y editar. (Las claves `detail.title-new` y `detail.title-edit` existen pero no se usan.)
- **Modos** (`ngOnInit`, `.ts:113-130`): sin parámetro `id` → **crear** (`isNew` e `isEditMode` verdaderos); con `id` → **ver** (formulario y autocompletados deshabilitados, carga el registro) y «Editar» los habilita. Siempre, al abrir, pide la lista de condiciones de venta; si falla, aviso «Error al cargar las condiciones de venta» (`medicamentos.detail.condiciones-error`, `.ts:116`).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar (`/:id` tras «Editar») |
|---|---|---|---|
| Encabezado de la tarjeta «Medicamentos» + «Datos del registro» + botones «Presentaciones de Venta» y «Editar» | No | Sí | No |
| Los seis campos | Editables | Deshabilitados | Editables |
| Tarjeta «Stock por Sucursal» | No | Sí | No |
| Botones del pie | Cancelar, Guardar | — | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:78-83`, `.html:47-168`)

| Etiqueta (clave) | Control | Tipo de control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Código Nacional (`medicamentos.fields.codigoNacional`) | `codigoNacional` | `matInput`, `maxlength="50"`, columna 4/12 | Sí | `Validators.required`, `maxLength(50)` (`.ts:79`) | «Este campo es obligatorio» (`general.errors.required`, `.html:52`) | Sí / Sí |
| Nombre Comercial (`medicamentos.fields.nombreComercial`) | `nombreComercial` | `matInput`, `maxlength="150"`, columna 8/12 | Sí | `required`, `maxLength(150)` (`.ts:80`) | «Este campo es obligatorio» (`.html:63`) | Sí / Sí |
| Laboratorio (`medicamentos.fields.laboratorio`) | `labSearchCtrl` (control suelto, fuera del formulario) | `matInput` con autocompletado y lupa; placeholder «Buscar laboratorio...» | Sí | Debe elegirse una opción de la lista: al guardar, si el texto no coincide con el nombre del laboratorio elegido, se descarta la elección (`.ts:221-223`) y falla la comprobación `!selectedLab()` (`.ts:228`) | «Este campo es obligatorio» (`.html:92-97`, aparece tras pulsar Guardar) | Sí / Sí |
| Unidad Base (`medicamentos.fields.unidadBase`) | `unidadSearchCtrl` | Igual; placeholder «Buscar unidad de medida...» | Sí | Igual (`.ts:224-226,228`) | «Este campo es obligatorio» (`.html:133-138`) | Sí / Sí |
| Condición de Venta (`medicamentos.fields.condicionVenta`) | `condicionVenta` | `mat-select` con las condiciones de la API (valor = posición) | Sí | `Validators.required` (`.ts:81`) | «Este campo es obligatorio» (`.html:150`) | Sí / Sí |
| Precio de Venta Base (`medicamentos.fields.precioVentaBase`) | `precioVentaBase` | `matInput type="number" min="0" step="0.01"`, prefijo «S/» | Sí | `required`, `Validators.min(0)` (`.ts:82`) | «El precio no puede ser negativo» (`medicamentos.detail.precioVentaBase-error`, `.html:165`) para **cualquier** error del campo, incluido el vacío | Sí / Sí |

Detalles de los autocompletados del detalle: al escribir (300 ms) consultan `GET /api/v1/laboratorios` o `GET /api/v1/unidadesmedida` con 20 resultados y, a diferencia de la búsqueda, **también con el campo vacío** (muestran los primeros 20, `.ts:92-110`). La lista de unidades muestra el código en gris y la descripción. Sin resultados: opción deshabilitada «Sin resultados» (`medicamentos.autocomplete.no-results`). Al abrir un registro, el laboratorio y la unidad base se precargan con el nombre devuelto por el detalle.

Campos solo de lectura: tarjeta de stock (§siguiente). No hay estado, ni vínculo DIGEMID, ni campos de auditoría en pantalla.

**Tarjeta «Stock por Sucursal»** (solo en ver, `.html:200-250`): título «Stock por Sucursal» (`medicamentos.detail.stock-section`) y línea «Stock Total: **N**» (`medicamentos.detail.stock-total`; `snapshot.stockTotal ?? 0`). Si el detalle trae sucursales, tabla con «Sucursal» (`detail.stock-col-sucursal`) y «Stock» (`detail.stock-col-cantidad`, alineado a la derecha) con la cantidad física; si no, «Sin stock registrado» (`detail.sin-stock`). El dato lo calcula el servidor (RN-MED-018).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Presentaciones de Venta (`medicamentos.detail.presentaciones-btn`), icono `sell` | Encabezado de la tarjeta, izquierda de «Editar» | Modo ver **y** `*appPermiso="'JerarquiasUoM:Leer'"` **y** `*appPermiso="'PresentacionesVenta:Leer'"` (`.html:26,28`) | Nunca | `abrirPresentacionesVenta()` (`.ts:275-284`) abre el diálogo de §2.4 (ancho 700 px) con producto, nombre y unidad base |
| Editar (`actions.edit`) | Encabezado, derecha | Modo ver y `*appPermiso="'Actualizar'"` (= `Medicamentos:Actualizar`, `.html:37`) | Nunca | `enterEditMode()` habilita formulario y autocompletados |
| Cancelar (`actions.cancel`) | Pie, derecha | Crear o editar | Nunca | `cancelEdit()` (`.ts:192-213`): en crear va a `/medicamentos`; en editar restaura la copia leída (campos, laboratorio y unidad) y vuelve a ver |
| Guardar (`actions.save`); spinner mientras guarda | Pie | Crear o editar | `saving()` | `save()` (`.ts:215-273`): marca todo como tocado; inválido o sin laboratorio/unidad elegidos → no envía. Crear → `POST /api/v1/medicamentos`, aviso y navega a `/medicamentos`. Editar → `PUT /api/v1/medicamentos/:id`, aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie, tras Guardar | Solo en editar de un registro existente y `*appPermiso="'Eliminar'"` (= `Medicamentos:Eliminar`, `.html:188`) | `deleting()` | `confirmDelete()` → diálogo |
| Botón volver (`chevron_left`) | Cabecera | Siempre | — | Navega a `/medicamentos` |

**Diálogos que abre**
- `ConfirmDialogComponent` al pulsar «Eliminar» (`.ts:286-297`), ancho 400 px: título «Confirmar eliminación» (`medicamentos.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este medicamento?» (`medicamentos.detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Confirmar → `DELETE /api/v1/medicamentos/:id`.
- El diálogo de presentaciones de venta (§2.4).

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Medicamento guardado correctamente» (`medicamentos.detail.save-success`) | `.ts:246,257` |
| Código `Medicamento.CodigoNacional.Duplicado` (409) | Error | «Ya existe un medicamento con ese código nacional.» (`medicamentos.detail.duplicate-codigo`) | `.ts:19-21,251,269` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos `errors.db*` (§0) | `save-error-key.util.ts` |
| Cualquier otro error al guardar (incluidos `Laboratorio.NotFound`, `UnidadMedida.NotFound` y los de validación) | Error | «Error al guardar el medicamento» (`medicamentos.detail.save-error`) | `.ts:251,269` |
| Eliminación correcta | Éxito | «Medicamento eliminado correctamente» (`medicamentos.detail.delete-success`); navega a `/medicamentos` | `.ts:303-304` |
| Eliminar: respuesta 409 (cualquier código) | Error | «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.» (`medicamentos.detail.delete-in-use`) | `.ts:306-312` |
| Eliminar: cualquier otro error | Error | «Error al eliminar el medicamento» (`medicamentos.detail.delete-error`) | `.ts:306-312` |
| Error al cargar el registro | Error | «Error al cargar el medicamento» (`medicamentos.detail.load-error`) | `.ts:163` |
| Error al cargar condiciones de venta | Error | «Error al cargar las condiciones de venta» (`medicamentos.detail.condiciones-error`) | `.ts:116` |

El mapa completo de eliminar está en §6.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir (siempre) | `getCondicionesVenta` | `GET /api/v1/medicamentos/condiciones-venta` | `Medicamentos:Leer` (`MedicamentosController.cs:24`) |
| Abrir un existente y tras editar | `MedicamentoService.getById` | `GET /api/v1/medicamentos/{id}` | `Medicamentos:Leer` (`:36`) |
| Autocompletar | `LaboratorioService.search`, `UnidadMedidaService.search` | `GET /api/v1/laboratorios`, `GET /api/v1/unidadesmedida` | `Laboratorios:Leer`, `UnidadesMedida:Leer` |
| Crear | `create` | `POST /api/v1/medicamentos` | `Medicamentos:Crear` (`:44`) |
| Editar | `update` | `PUT /api/v1/medicamentos/{id}` (cuerpo sin `id`; el servidor lo toma de la URL) | `Medicamentos:Actualizar` (`:54`) |
| Eliminar | `delete` | `DELETE /api/v1/medicamentos/{id}` | `Medicamentos:Eliminar` (`:68`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga, un spinner reemplaza la tarjeta (`.html:9-13`).
- Si falla la carga del registro, la tarjeta queda con campos vacíos y deshabilitados; con `Medicamentos:Actualizar` sigue visible «Editar», y «Presentaciones de Venta» (con sus permisos) fallaría porque `snapshot` es nulo (`abrirPresentacionesVenta` usa `this.snapshot!`). Sale de leer el código; no se ejecutó.
- Un medicamento antiguo puede tener el precio vacío en la base (`PrecioVentaBase` admite nulo, `Medicamento.cs:17`): al editarlo el campo aparece vacío y, por ser obligatorio, hay que informarlo para guardar.
- Al guardar con éxito una edición, la página recarga el registro (`load()`), con spinner.

**Observaciones**
- Longitudes y precio coinciden con la API (50, 150, no negativo; `CreateMedicamentoCommandValidator.cs:13-31`, RN-MED-002 a RN-MED-008). La API rechaza además código o nombre formados solo por espacios (`NotEmpty`), que `Validators.required` de Angular acepta; el código sería `{Request}.{Propiedad}` (por ejemplo `CreateMedicamentoCommand.CodigoNacional`, no mapeado) y se vería «Error al guardar el medicamento». Sale de leer el código; no se ejecutó.
- **El texto de «en uso» es más corto que lo que bloquea el backend**: el handler comprueba lotes, presentaciones de venta, jerarquías de unidades, formulaciones clínicas, historial de precios de proveedor, servicios de agenda, servicios de inmunización y órdenes de producción magistral (`DeleteMedicamentoCommandHandler.cs:34-42`); el mensaje dice «lotes, presentaciones u otros registros asociados». Esto actualiza RN-MED-013 de 01b (que decía que no valida dependencias) tras el commit `27597cd`.
- El mensaje de precio se muestra también cuando el campo está vacío (no hay texto de «obligatorio» para ese campo).
- Specs: `medicamentos-detail.component.spec.ts` (3 casos) y `medicamentos-detail.delete.spec.ts` (usa `describeDeleteInUse`, no suma casos propios en el inventario).

### 1.3 `/medicamentos/carga-masiva` — Carga Masiva (medicamentos)

- **Componente:** `MedicamentosCargaMasivaComponent` (`app/pages/medicamentos/medicamentos-carga-masiva/medicamentos-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `medicamentos` (se declara pero no aporta textos).
- **Para qué sirve:** cargar varios medicamentos desde un Excel. Flujo común en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/medicamentos` (§1.1).
- **Permiso de ruta:** `Medicamentos:Leer` + `Medicamentos:Crear`.
- **Migas y cabecera:** título «Carga Masiva»; migas «Inicio» / «Catálogos» / «Medicamentos» (enlace a `/medicamentos`) / «Carga Masiva» (activa); botón volver a `/medicamentos` (`.ts:19-24`).

**Configuración que pasa** (`CargaMasivaConfig`, `.ts:18-33`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` (plantilla) | `GET {api}/api/v1/medicamentos/plantilla` — `Medicamentos:Crear` (`MedicamentosController.cs:112`); se baja como `plantilla-medicamentos.xlsx` |
| `previewUrl` | `POST {api}/api/v1/medicamentos/previsualizar` — `Medicamentos:Crear` (`:120`) |
| `saveUrl` | `POST {api}/api/v1/medicamentos/carga-masiva` — `Medicamentos:Crear` (`:137`) |
| `backRoute` | `/medicamentos` |
| Columnas de la vista previa (clave del dato → etiqueta de la raíz) | `codigoNacional` → «Código Nacional» (`fields.codigoNacional`); `nombreComercial` → «Nombre Comercial» (`fields.nombreComercial`); `laboratorio` → «Laboratorio» (`fields.laboratorio`); `unidadBase` → «Unidad Base» (`fields.unidadBase`); `condicionVenta` → «Condición de Venta» (`fields.condicionVenta`). Las cinco declaradas `required: true`. Después, la columna de errores y «Estado». No hay columna de precio |

**Botones, mensajes y errores:** los del componente común. Reglas de la carga en RN-MED-021 a RN-MED-026 de 01b (laboratorio por nombre de compañía y unidad base por descripción, condición de venta por texto). Las claves de datos coinciden con `MedicamentoImportRow(CodigoNacional, NombreComercial, Laboratorio, UnidadBase, CondicionVenta)` (`sofia-apis/src/SOFIA.Application/Medicamentos/Commands/CargaMasivaMedicamentos/CargaMasivaMedicamentosCommand.cs:5-10`).

**Observaciones:** la carga masiva no incluye el precio de venta base, mientras que el formulario lo exige (los medicamentos cargados por Excel quedan con precio nulo en la base; `Medicamento.cs:15-17`). Sale de leer el código y del modelo de la fila; no se ejecutó.

---

## 2. Presentaciones de venta

Carpeta `app/pages/presentaciones-venta`, scope i18n `presentaciones-venta` (clave `presentacionesVenta`, ver convenciones). Componentes enrutados (`presentaciones-venta.routes.ts`, `loadChildren` desde `app/app.routes.ts:101-105`): `PresentacionesVentaSearchComponent` (`''`) y `PresentacionesVentaDetailComponent` (rutas `nueva` y `:id`). **No hay carga masiva ni ruta `carga-masiva`.** Además existe `PresentacionesVentaModalComponent`, un diálogo que no tiene ruta y se abre desde el detalle de Medicamentos (§2.4).

Una presentación de venta liga **un medicamento** con **una unidad de venta** (una unidad de medida distinta de la unidad base del medicamento) y un **precio de venta propio**. La equivalencia en unidades base la calcula el servidor (§2.3).

### 2.1 `/presentaciones-venta` — Presentaciones de Venta (búsqueda)

- **Componente:** `PresentacionesVentaSearchComponent` (`app/pages/presentaciones-venta/presentaciones-venta-search/presentaciones-venta-search.component.ts`, `.html`). Scope `presentaciones-venta`.
- **Para qué sirve:** buscar presentaciones por producto y por unidad de venta, abrirlas, exportar la lista y entrar a crear.
- **Cómo se llega:** menú «Catálogos» → «Presentaciones de Venta»; botón volver y migas del detalle; redirección tras crear o eliminar.
- **Permiso de ruta:** `PresentacionesVenta:Leer`.
- **Migas y cabecera:** título «Presentaciones de Venta» (`presentacionesVenta.title`); migas «Inicio» / «Catálogos» / «Presentaciones de Venta» (`breadcrumbs.presentacionesVenta`, activa); sin botón volver. Tarjeta de filtros con título «Búsqueda de Presentaciones de Venta» (`presentacionesVenta.search.title`) y el subtítulo común.
- **Modos:** uno solo.

**Filtros de búsqueda** (`.ts:161-164`, `.html:34-75`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Producto (`presentacionesVenta.fields.producto`) | `productoNombre` | Texto con lupa y autocompletado; placeholder «Buscar producto...» (`autocomplete.producto-placeholder`); Enter busca | Al escribir (300 ms, solo con texto) llama a `GET /api/v1/medicamentos?nombreComercial=<texto>&pageNumber=1&pageSize=20` y lista el nombre comercial; sin coincidencias: opción deshabilitada «Sin resultados» (`autocomplete.no-results`). Si eliges una opción se guarda su identificador y la búsqueda filtra por **producto exacto** (`productoId`); si solo escribes, filtra por nombre «contiene» | Vacío |
| Unidad de Venta (`presentacionesVenta.fields.descripcion`) | `descripcion` | Texto con lupa; placeholder «Unidad de Venta»; Enter busca | «Contiene» sobre la descripción de la presentación (que es la descripción de la unidad de venta) | Vacío |

**Tabla de resultados** (`.html:95-167`, columnas `.ts:172`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Producto (`presentacionesVenta.table.producto`) | `productoNombre` | Texto | Sí: `[routerLink]="[row.id]"` → `/presentaciones-venta/:id` |
| Unidad de Venta (`presentacionesVenta.table.descripcion`) | `descripcion` | Texto | No |
| Equivalencia (`presentacionesVenta.table.cantidadUnidadesBase`) | `cantidadUnidadesBase` | Número tal como lo devuelve la API, alineado a la derecha, **sin** el nombre de la unidad base | No |
| Precio de Venta (`presentacionesVenta.table.precioVenta`) | `precioVenta` | «S/ » + número con dos decimales (`number: '1.2-2'`), a la derecha | No |

Orden por defecto: servidor, por nombre del producto y luego por descripción (`GetPresentacionesVentaQueryHandler.cs`, `OrderBy(x => x.Producto!.NombreComercial).ThenBy(x => x.Descripcion)`); excluye presentaciones cuyo producto esté eliminado. Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Sin resultados» (`generic-labels.no-results`, `.html:147`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros, derecha | `*appPermiso="'Crear'"` (= `PresentacionesVenta:Crear`, `.html:24`) | Nunca | `goToNew()` → `/presentaciones-venta/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` (`.ts:140-149`): vacía formulario, lista de productos y el producto elegido; oculta la tabla |
| Buscar (`actions.search`); spinner | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/presentacionesventa` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla, izquierda | Tras buscar | Nunca | `exportar()` → `POST /api/v1/presentacionesventa/exportar` |
| Enlace con el producto | Primera columna | Con resultados | — | Navega a `/presentaciones-venta/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar presentaciones de venta» (`presentacionesVenta.search.search-error`) | `presentaciones-venta-search.component.ts:135` |
| Falla la exportación | Error | El mismo texto, sin referencia (`showError`) | `.ts:174` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar | `PresentacionVentaService.search` (`app/pages/presentaciones-venta/services/presentacion-venta.service.ts`) | `GET /api/v1/presentacionesventa?pageNumber&pageSize[&productoId][&productoNombre][&descripcion]` | `PresentacionesVenta:Leer` (`PresentacionesVentaController.cs:16`) |
| Autocompletar producto | `MedicamentoService.search` | `GET /api/v1/medicamentos?nombreComercial=…&pageNumber=1&pageSize=20` | `Medicamentos:Leer` |
| Exportar | `PresentacionVentaService.exportar` | `POST /api/v1/presentacionesventa/exportar` (blob) | `PresentacionesVenta:Leer` (`:60`) |

**Exportación**
- Cuerpo (`.ts:161-176`): `headers` con 4 textos, en orden: «Producto» (`table.producto`), «Unidad de Venta» (`table.descripcion`), «Equivalencia» (`table.cantidadUnidadesBase`), «Precio de Venta» (`table.precioVenta`); más `productoNombre` y `descripcion` si tienen valor. Archivo: «presentaciones-venta.xlsx» (`presentacionesVenta.search.export-filename`). El Excel del servidor lleva producto, descripción, cantidad de unidades base y precio (`PresentacionesVentaController.cs:60-90`).
- Sin carga masiva.

**Comportamientos particulares**
- Se guarda en el estado de búsqueda, además del formulario, el identificador del producto elegido (`_productoId`, `.ts:128`) y se restaura al volver.
- El producto elegido en el autocompletado se descarta cuando escribes de nuevo un texto no vacío (`.ts:74`). Si **borras** el texto del campo, el identificador anterior **no** se descarta (la rama de texto vacío sale antes de limpiarlo, `.ts:73`) y «Buscar» sigue filtrando por ese producto. Sale de leer el código; no se ejecutó.

**Observaciones**
- **La exportación ignora el producto elegido en la lista.** `currentFilters()` pone `productoNombre` en `undefined` cuando hay un producto elegido (`.ts:108`) y `exportar()` solo envía `productoNombre` y `descripcion` (`.ts:163-165`), sin `productoId`. Resultado: si eliges un producto del autocompletado y pulsas «Exportar», el Excel sale sin filtrar por producto (solo por unidad de venta). Hallazgo por lectura de código; no se ejecutó.
- Las claves `presentacionesVenta.*` se usan en el código con el prefijo camelCase; no hay claves sin uso en este scope (§9).

### 2.2 `/presentaciones-venta/nueva` y `/presentaciones-venta/:id` — Presentaciones de Venta (detalle)

- **Componente:** `PresentacionesVentaDetailComponent` (`app/pages/presentaciones-venta/presentaciones-venta-detail/presentaciones-venta-detail.component.ts`, `.html`). Scope `presentaciones-venta`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear una presentación (elegir producto, unidad de venta y precio), verla y, con permisos, editarla (unidad y precio) o eliminarla.
- **Cómo se llega:** `/nueva`: botón «Nuevo» de la búsqueda. `/:id`: enlace del producto en la tabla.
- **Permiso de ruta:** `/presentaciones-venta/nueva` → `PresentacionesVenta:Leer` + `PresentacionesVenta:Crear`; `/presentaciones-venta/:id` → `PresentacionesVenta:Leer`.
- **Migas y cabecera:** título «Presentaciones de Venta»; migas «Inicio» / «Catálogos» / «Presentaciones de Venta» (con enlace a `/presentaciones-venta`); botón volver a `/presentaciones-venta` (`.html:8-12`, `.ts:79-83`).
- **Modos** (`ngOnInit`, `.ts:100-112`): sin `id` → **crear**; con `id` → **ver** (formulario, producto y unidad deshabilitados) y «Editar» habilita unidad y precio (`enterEditMode`, `.ts:166-178`). **El producto solo se elige al crear**; en editar permanece deshabilitado.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar |
|---|---|---|---|
| Encabezado con «Datos del registro» + «Editar» | No | Sí | No |
| Producto | Editable (autocompletado) | Deshabilitado | Deshabilitado, con ayuda |
| Unidad de Venta | Select, solo tras elegir un producto con jerarquía | Texto plano (`snapshot.descripcion`) | Select |
| Precio de Venta | Campo, solo tras elegir un producto con jerarquía | Deshabilitado | Editable |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:68-77`, `.html:42-147`)

| Etiqueta (clave) | Control | Tipo de control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Producto (`presentacionesVenta.fields.producto`) | `productoCtrl` (suelto) | `matInput` con autocompletado, sin lupa; placeholder «Buscar producto...»; opciones = nombre comercial de `GET /api/v1/medicamentos?nombreComercial=<texto>&pageNumber=1&pageSize=20` (también con texto vacío) | Sí | Debe elegirse una opción (`selectedProducto`); escribir de nuevo en crear descarta la elección (`.ts:92`). `save()` exige `selectedProducto()` (`.ts:198`) | «Este campo es obligatorio» (`general.errors.required`, `.html:65-69`, tras pulsar Guardar) | Sí / No. En existentes aparece la ayuda «El producto de una presentación no se puede modificar luego de creada» (`presentacionesVenta.detail.producto-hint`, `.html:70-74`) |
| Unidad de Venta (`presentacionesVenta.fields.unidadVenta`) | `unidadVentaCtrl` (suelto) | `mat-select`; cada opción dice «{descripción} ({equivalencia} {unidad base})», por ejemplo «Caja (30 Tableta)» (`.html:103-106`) | Sí | `Validators.required` (`.ts:68`) | «Debes seleccionar una unidad de venta» (`presentacionesVenta.detail.unidadVenta-error`, `.html:109`) | Sí / Sí |
| Precio de Venta (`presentacionesVenta.fields.precioVenta`) | `precioVenta` | `matInput type="number" min="0" step="0.01"`, prefijo «S/» | Sí | `required`, `Validators.min(0)` (`.ts:76`) | «El precio de venta no puede ser negativo» (`presentacionesVenta.detail.precioVenta-error`, `.html:123`) para cualquier error | Sí / Sí |

La **equivalencia en unidades base no es un campo**: solo aparece dentro del texto de cada opción de la unidad de venta, y no se muestra en la ficha en modo ver. Las opciones salen de `GET /api/v1/jerarquiasuom/unidades-vendibles` (§2.3). El nombre de la unidad base sale del producto elegido (en crear, de la lista de autocompletado, `.ts:143`) o de `GET /api/v1/medicamentos/{id}` (en editar, `.ts:175`).

**Estados de la zona unidad/precio (solo en crear o editar con producto elegido)**: mientras carga las unidades, spinner; si no hay unidades vendibles, texto «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.» (`presentacionesVenta.detail.sin-jerarquia`) y, con `*appPermiso="'JerarquiasUoM:Crear'"`, botón «Ir a Jerarquías de Unidades» (`detail.ir-a-jerarquias-btn`, icono `add`) que navega a `/jerarquias/nueva` (`irAJerarquias()`, `.ts:162`); en ese estado los campos unidad y precio **no se dibujan**.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado de la tarjeta | Modo ver y `*appPermiso="'Actualizar'"` (`PresentacionesVenta:Actualizar`, `.html:32`) | Nunca | `enterEditMode()` |
| Ir a Jerarquías de Unidades (`presentacionesVenta.detail.ir-a-jerarquias-btn`) | Zona de unidad | Producto elegido sin unidades vendibles y `JerarquiasUoM:Crear` (`.html:88`) | Nunca | `irAJerarquias()` → `/jerarquias/nueva` |
| Cancelar (`actions.cancel`) | Pie | Crear o editar | Nunca | `cancelEdit()` (`.ts:180-191`): en crear va a `/presentaciones-venta`; en editar restaura la copia leída y vuelve a ver |
| Guardar (`actions.save`); spinner | Pie | Crear o editar | `saving()` | `save()` (`.ts:193-233`): crear → `POST /api/v1/presentacionesventa` con `{productoId, unidadVentaId, precioVenta}`; editar → `PUT /api/v1/presentacionesventa/:id` con `{unidadVentaId, precioVenta}`. En ambos casos aviso de éxito y navega a `/presentaciones-venta` |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie | Solo en editar y `*appPermiso="'Eliminar'"` (`PresentacionesVenta:Eliminar`, `.html:167`) | `deleting()` | `confirmDelete()` → diálogo |

**Diálogos que abre:** `ConfirmDialogComponent` al eliminar (`.ts:235-246`), ancho 400 px: título «Confirmar eliminación» (`presentacionesVenta.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta presentación de venta?» (`detail.delete-confirm-message`), botones «Cancelar» y «Confirmar».

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Presentación de venta guardada correctamente» (`presentacionesVenta.detail.save-success`) | `.ts:205` |
| Guardar: respuesta 409 (cualquier código; la API usa `PresentacionVenta.UnidadVenta.Duplicada`) | Error | «Ya existe una presentación de venta para esa unidad en este producto.» (`detail.unidad-duplicada`) | `.ts:208-215` |
| Guardar: cualquier otro error (incluidos 404 `PresentacionVenta.Producto`/`.UnidadVenta` y 400 `PresentacionVenta.Jerarquia.NoResuelta`) | Error | «Error al guardar la presentación de venta» (`detail.save-error`) | `.ts:208-215` |
| Eliminación correcta | Éxito | «Presentación de venta eliminada correctamente» (`detail.delete-success`); navega a `/presentaciones-venta` | `.ts:252-253` |
| Eliminar: 409 | Error | «No se puede eliminar la presentación porque tiene ventas registradas.» (`detail.delete-in-use`) | `.ts:255-262` |
| Eliminar: otro error | Error | «Error al eliminar la presentación de venta» (`detail.delete-error`) | `.ts:255-262` |
| Error al cargar | Error | «Error al cargar la presentación de venta» (`detail.load-error`) | `.ts:124` |

No usa `saveErrorKey`: ni el mapa por código ni los textos `errors.db*`. Si falla la carga de unidades vendibles no hay aviso: el SPA deja la lista vacía y muestra el texto «sin jerarquía» (`.ts:155-158`).

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir existente | `PresentacionVentaService.getById` | `GET /api/v1/presentacionesventa/{id}` | `PresentacionesVenta:Leer` (`PresentacionesVentaController.cs:24`) |
| Autocompletar producto (crear) | `MedicamentoService.search` | `GET /api/v1/medicamentos?nombreComercial=…` | `Medicamentos:Leer` |
| Elegir producto (crear) o pulsar «Editar» | `JerarquiaUoMService.getUnidadesVendibles` (`app/pages/jerarquias/services/jerarquia-uom.service.ts`) | `GET /api/v1/jerarquiasuom/unidades-vendibles?productoId=<guid>` | `JerarquiasUoM:Leer` (`JerarquiasUoMController.cs:30`) |
| Pulsar «Editar» | `MedicamentoService.getById` | `GET /api/v1/medicamentos/{productoId}` (para el nombre de la unidad base) | `Medicamentos:Leer` |
| Crear | `create` | `POST /api/v1/presentacionesventa` | `PresentacionesVenta:Crear` (`:32`) |
| Editar | `update` | `PUT /api/v1/presentacionesventa/{id}` | `PresentacionesVenta:Actualizar` (`:42`) |
| Eliminar | `delete` | `DELETE /api/v1/presentacionesventa/{id}` | `PresentacionesVenta:Eliminar` (`:52`) |

Esta pantalla depende de permisos de **tres módulos más** (`Medicamentos:Leer`, `JerarquiasUoM:Leer` y, para el atajo, `JerarquiasUoM:Crear`). El SPA no comprueba `JerarquiasUoM:Leer`: sin él, `getUnidadesVendibles` falla y el usuario ve el texto «sin jerarquía» (efecto no verificado en ejecución).

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga, spinner en lugar de la tarjeta (`.html:14-17`).
- En crear, hasta que eliges un producto de la lista no se dibujan unidad ni precio; si pulsas Guardar sin producto se marca el error del producto. Si eliges un producto **sin** jerarquía y pulsas Guardar, `save()` sale sin enviar porque el precio y la unidad son obligatorios y no están dibujados (no hay error visible en ese estado). Sale de leer el código; no se ejecutó.
- Al editar, la unidad actual aparece preseleccionada porque `unidadVentaCtrl` conserva el id de la presentación; si esa unidad ya no figura entre las vendibles (jerarquía modificada), el select no mostraría texto para ella (no verificado).

**Observaciones**
- El formulario valida precio ≥ 0 igual que la API (`CreatePresentacionVentaCommandValidator`, `PrecioVenta.GreaterThanOrEqualTo(0)`; entidad `PresentacionVenta.cs`); producto y unidad obligatorios (RN-PRE-002, RN-PRE-004).
- La descripción de la presentación no se escribe: el servidor la copia de la unidad elegida (RN-PRE-008; `CreatePresentacionVentaCommandHandler.cs`).
- No hay specs de la búsqueda ni del detalle; solo del diálogo (§2.4).

### 2.3 Unidades vendibles y equivalencia en unidades base

Cómo se obtienen y calculan (todo en el servidor; el SPA solo los muestra):

1. **Origen de la lista de unidades vendibles.** `GET /api/v1/jerarquiasuom/unidades-vendibles?productoId=<guid>` (`JerarquiasUoM:Leer`, `JerarquiasUoMController.cs:30`). El manejador (`sofia-apis/src/SOFIA.Application/JerarquiasUoM/Queries/GetUnidadesVendiblesUoM/GetUnidadesVendiblesUoMQueryHandler.cs`) toma las jerarquías del producto, reúne todas las unidades (mayores y menores) que aparecen, excluye la unidad base del medicamento (`:30-33`; vender por la unidad base no necesita presentación), calcula para cada una la cantidad de unidades base con `JerarquiaConversionResolver.Resolve` (`:35-38`) y descarta las que no se pueden convertir. Devuelve `{ unidadMedidaId, descripcion, cantidadUnidadesBase }` ordenado por cantidad descendente (`:53`). Producto inexistente: 404 `Medicamento.NotFound` (`:19`).
2. **Equivalencia mostrada.** En las opciones del select: «{descripcion} ({cantidadUnidadesBase} {unidadBaseNombre})». En la tabla de la búsqueda y del diálogo es el valor guardado en la presentación (`CantidadUnidadesBase`).
3. **Quién calcula el valor guardado.** El servidor, al crear y al editar: `CreatePresentacionVentaCommandHandler.cs:35-40` y `UpdatePresentacionVentaCommandHandler.cs:42-47` resuelven la conversión de la unidad de venta a la unidad base con las jerarquías del producto y guardan el resultado; si no hay conversión, error 400 de validación `PresentacionVenta.Jerarquia.NoResuelta` (`:44`/`:51`). El SPA **no envía** la equivalencia (el cuerpo lleva solo producto, unidad y precio) ni la calcula.
4. **Precio.** Es propio de la presentación (RN-PRE-009); el SPA no lo deriva del precio base del medicamento.

### 2.4 Diálogo «Presentaciones de Venta» (desde el detalle de Medicamentos)

- **Componente:** `PresentacionesVentaModalComponent` (`app/pages/presentaciones-venta/presentaciones-venta-modal/presentaciones-venta-modal.component.ts`, `.html`). Scope `presentaciones-venta`. Se abre con `MatDialog.open` desde `MedicamentosDetailComponent.abrirPresentacionesVenta()` (`medicamentos-detail.component.ts:275-284`), ancho 700 px, con `{ productoId, productoNombre, unidadBaseNombre }`.
- **Para qué sirve:** administrar las presentaciones de venta de un medicamento sin salir de su ficha (listar, agregar, editar y eliminar).
- **Cómo se llega:** botón «Presentaciones de Venta» del detalle del medicamento en modo ver (exige `JerarquiasUoM:Leer` y `PresentacionesVenta:Leer`).
- **Cabecera del diálogo:** título «Presentaciones de Venta» (`presentacionesVenta.modal.title`) y debajo el nombre comercial del medicamento. Contenido de ancho mínimo 520 px. Pie con botón «Cerrar» (`modal.close-btn`, `close()`).
- **Carga inicial** (`ngOnInit`, `.ts:65-92`): primero `GET /api/v1/jerarquiasuom/unidades-vendibles?productoId=…` y luego `GET /api/v1/presentacionesventa?productoId=…&pageNumber=1&pageSize=100` (solo la primera página de 100). Si cualquiera falla: «Error al cargar las presentaciones de venta» (`modal.load-error`).

**Estados del contenido** (`.html:13-155`)

| Estado | Qué se ve |
|---|---|
| Cargando | Spinner |
| El producto no tiene unidades vendibles | «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.» (`modal.sin-jerarquia`) y, con `*appPermiso="'JerarquiasUoM:Crear'"` (`.html:22`), botón «Ir a Jerarquías de Unidades» (`modal.ir-a-jerarquias-btn`, icono `add`): cierra el diálogo y navega a `/jerarquias/nueva` |
| Con unidades, sin presentaciones y sin formulario abierto | «Este producto todavía no tiene presentaciones de venta.» (`modal.empty`) |
| Con presentaciones | Tabla (abajo) |

**Tabla** (`displayedColumns`, `.ts:55`): «Unidad de Venta» (`table.descripcion`, `descripcion`); «Equivalencia» (`table.cantidadUnidadesBase`, valor + nombre de la unidad base, p. ej. «30 Tableta», a la derecha); «Precio de Venta» (`table.precioVenta`, «S/ » con dos decimales, a la derecha); última columna sin encabezado con dos botones de icono: editar (icono `edit`, tooltip y etiqueta «Editar» = `modal.edit-tooltip`; `*appPermiso="'PresentacionesVenta:Actualizar'"`, `.html:64`; llama a `openEditForm(row)`) y eliminar (icono `delete`, «Eliminar» = `modal.delete-tooltip`; `*appPermiso="'PresentacionesVenta:Eliminar'"`, `.html:73`; llama a `confirmDelete(row)`). Ambos deshabilitados mientras esa fila se elimina.

**Campos del formulario en línea** (aparece al pulsar «Agregar presentación» o el lápiz; `.ts:50-53`, `.html:90-137`)

| Etiqueta (clave) | Control | Tipo | Obligatorio | Validaciones | Mensaje de error |
|---|---|---|---|---|---|
| Unidad de Venta (`presentacionesVenta.fields.unidadVenta`) | `unidadVentaId` | `mat-select`; opciones = unidades vendibles **menos** las ya usadas por otra presentación del producto (al editar sigue disponible la propia); cada opción «{descripción} ({equivalencia} {unidad base})» | Sí | `Validators.required` | «Este campo es obligatorio» (`general.errors.required`) |
| Precio de Venta (`presentacionesVenta.fields.precioVenta`) | `precioVenta` | `matInput type="number" min="0" step="0.01"`, prefijo «S/» | Sí | `required`, `min(0)` | «El precio de venta no puede ser negativo» (`presentacionesVenta.detail.precioVenta-error`) |

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Agregar presentación (`modal.add-btn`), icono `add` | Bajo la tabla | Formulario cerrado y `*appPermiso="'PresentacionesVenta:Crear'"` (`.html:139`) | No quedan unidades sin presentación; entonces el tooltip dice «Ya existe una presentación para cada unidad definida en la Jerarquía de este producto.» (`modal.sin-unidades-disponibles`) | `openCreateForm()` |
| Cancelar (`actions.cancel`) | Formulario en línea | Formulario abierto | Nunca | `cancelForm()` |
| Guardar (`actions.save`); spinner | Formulario en línea | Formulario abierto | `saving()` | `save()` (`.ts:120-152`): crear → `POST /api/v1/presentacionesventa` `{productoId, unidadVentaId, precioVenta}`; editar → `PUT /api/v1/presentacionesventa/:id` `{unidadVentaId, precioVenta}`; recarga la lista y cierra el formulario |
| Cerrar (`modal.close-btn`) | Pie | Siempre | Nunca | `close()` |

**Diálogo de confirmación al eliminar** (`.ts:154-165`): `ConfirmDialogComponent` 400 px, título «Confirmar eliminación» (`modal.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta presentación de venta?» (`modal.delete-confirm-message`).

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Guardar correctamente | Éxito | «Presentación de venta guardada correctamente» (`modal.save-success`) | `.ts:131` |
| Guardar: 409 | Error | «Ya existe una presentación de venta para esa unidad.» (`modal.unidad-duplicada`) | `.ts:136-143` |
| Guardar: otro error | Error | «Error al guardar la presentación de venta» (`modal.save-error`) | `.ts:136-143` |
| Eliminar correctamente | Éxito | «Presentación de venta eliminada correctamente» (`modal.delete-success`) | `.ts:171` |
| Eliminar: 409 | Error | «No se puede eliminar la presentación porque tiene ventas registradas.» (`modal.delete-in-use`) | `.ts:175-182` |
| Eliminar: otro error | Error | «Error al eliminar la presentación de venta» (`modal.delete-error`) | `.ts:175-182` |
| Error al cargar | Error | «Error al cargar las presentaciones de venta» (`modal.load-error`) | `.ts:74,89` |

**Llamadas a la API:** las de §2.2 (`unidades-vendibles`, listado con `productoId`, crear, editar, eliminar) con los mismos permisos.

**Observaciones**
- El diálogo no valida el permiso `PresentacionesVenta:Leer` ni `JerarquiasUoM:Leer`: lo hace el botón que lo abre en Medicamentos.
- Specs: `presentaciones-venta-modal.component.spec.ts` (3 casos): permisos de las acciones y del enlace a jerarquías.
- Los textos de la modal que se repiten con los del detalle (`modal.*` frente a `detail.*`) están duplicados en el `es.json`.

---

## 3. Formulaciones clínicas

Carpeta `app/pages/formulaciones-clinicas`, scope i18n `formulacionesClinicas` (`provideTranslocoScope('formulacionesClinicas')`). Componentes enrutados (`formulaciones-clinicas.routes.ts`, `loadChildren` desde `app/app.routes.ts:108-112`): `FormulacionesClinicasSearchComponent` (`''`) y `FormulacionesClinicasDetailComponent` (rutas `nueva` y `:id`). No hay carga masiva.

Una **formulación clínica** liga **un medicamento** con **un ingrediente activo**, una **concentración de dosis** (número) expresada en una **unidad de medida**, y un **código TE Orange** opcional. Un medicamento lista cada ingrediente activo una sola vez; para cambiar la concentración se edita esa fila (RN-FOR-005; `CreateFormulacionClinicaCommandHandler.cs:34-39`, índice único `UX_Formulacion_Clinica_Producto_Ingrediente` filtrado por `IsDeleted = 0`, `sofia-apis/src/SOFIA.DbUp/Scripts/0020_SoftDeleteAware_Uniques_And_Checks.sql:82`). Un medicamento con varios ingredientes tiene varias filas.

### 3.1 `/formulaciones-clinicas` — Formulaciones Clínicas (búsqueda)

- **Componente:** `FormulacionesClinicasSearchComponent` (`app/pages/formulaciones-clinicas/formulaciones-clinicas-search/formulaciones-clinicas-search.component.ts`, `.html`). Scope `formulacionesClinicas`.
- **Para qué sirve:** buscar formulaciones por medicamento e ingrediente activo, abrirlas, exportar la lista y entrar a crear.
- **Cómo se llega:** menú «Catálogos» → «Formulaciones Clínicas»; botón volver y migas del detalle; redirección tras crear, editar o eliminar.
- **Permiso de ruta:** `FormulacionesClinicas:Leer`.
- **Migas y cabecera:** título «Formulaciones Clínicas» (`formulacionesClinicas.title`); migas «Inicio» / «Catálogos» / «Formulaciones Clínicas» (`breadcrumbs.formulaciones-clinicas`, activa); sin botón volver. Tarjeta de filtros con título «Búsqueda de Formulaciones Clínicas» (`formulacionesClinicas.search.title`) y el subtítulo común.
- **Modos:** uno solo.

**Filtros de búsqueda** (`.ts:45`, `.html:34-55`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Medicamento (`formulacionesClinicas.fields.producto`) | `productoNombre` | Texto con lupa; Enter busca | «Contiene» sobre el nombre comercial, sin distinguir mayúsculas (`GetFormulacionesClinicasWithPaginationQueryHandler.cs`) | Vacío |
| Ingrediente Activo (`formulacionesClinicas.fields.ingrediente`) | `ingredienteNombre` | Texto con lupa; Enter busca | «Contiene» sobre la denominación DCI | Vacío |

**Tabla de resultados** (`.html:75-154`, columnas `.ts:53-59`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Medicamento (`formulacionesClinicas.table.producto`) | `productoNombre` | Texto | Sí: `[routerLink]="[row.id]"` → `/formulaciones-clinicas/:id` |
| Ingrediente Activo (`table.ingrediente`) | `ingredienteNombre` | Texto | No |
| Concentración (`table.concentracion`) | `concentracionDosis` | Número tal como lo devuelve la API (hasta 4 decimales) | No |
| Unidad de Medida (`table.unidadMedida`) | `unidadMedidaNombre` | Texto (descripción de la unidad) | No |
| Código TE Orange (`table.codigoTeOrange`) | `codigoTeOrange` | Texto; vacío si es nulo | No |

Orden por defecto: servidor, por nombre del medicamento. Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:134`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Nuevo (`actions.new`), icono `add` | Encabezado de la tarjeta de filtros | `*appPermiso="'Crear'"` (= `FormulacionesClinicas:Crear`, `.html:24`) | Nunca | `goToNew()` → `/formulaciones-clinicas/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` (`.ts:104-111`) |
| Buscar (`actions.search`); spinner | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/formulacionesclinicas` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/formulacionesclinicas/exportar` |
| Enlace con el medicamento | Primera columna | Con resultados | — | Navega a `/formulaciones-clinicas/:id` |

No hay botón «Carga Masiva».

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar formulaciones clínicas» (`formulacionesClinicas.search.search-error`) | `formulaciones-clinicas-search.component.ts:99` |
| Falla la exportación | Error | El mismo texto, con referencia de seguimiento si la respuesta trae `traceId` (`showServerError`) | `.ts:146` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar | `FormulacionClinicaService.search` (`app/pages/formulaciones-clinicas/services/formulacion-clinica.service.ts`) | `GET /api/v1/formulacionesclinicas?pageNumber&pageSize[&productoNombre][&ingredienteNombre]` | `FormulacionesClinicas:Leer` (`FormulacionesClinicasController.cs:17`) |
| Exportar | `exportar` | `POST /api/v1/formulacionesclinicas/exportar` (blob) | `FormulacionesClinicas:Leer` (`:72`) |

**Exportación**
- Cuerpo (`.ts:123-148`): `headers` con 5 textos, en orden: «Medicamento» (`table.producto`), «Ingrediente Activo» (`table.ingrediente`), «Concentración» (`table.concentracion`), «Unidad de Medida» (`table.unidadMedida`), «Código TE Orange» (`table.codigoTeOrange`); más los dos filtros si tienen valor. Archivo: «formulaciones-clinicas.xlsx» (`formulacionesClinicas.search.export-filename`).
- Sin carga masiva.

**Comportamientos particulares:** restauración de filtros (se aplica con `patchValue` sin silenciar eventos, `.ts:68`); «Buscar» no vuelve a la página 1.

**Observaciones**
- La fila vacía dice «Resultados», no «Sin resultados».
- El endpoint `GET /api/v1/formulacionesclinicas/medicamento/{productoId}` (`FormulacionesClinicas:Leer`, `FormulacionesClinicasController.cs:24`) **no lo llama ninguna pantalla**: el servicio del SPA no tiene método para él (comprobado en `formulacion-clinica.service.ts`). Solo API (sin pantalla).

### 3.2 `/formulaciones-clinicas/nueva` y `/formulaciones-clinicas/:id` — Formulaciones Clínicas (detalle)

- **Componente:** `FormulacionesClinicasDetailComponent` (`app/pages/formulaciones-clinicas/formulaciones-clinicas-detail/formulaciones-clinicas-detail.component.ts`, `.html`). Scope `formulacionesClinicas`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** registrar qué ingrediente activo, en qué concentración y unidad, contiene un medicamento; verlo y, con permisos, editarlo o eliminarlo.
- **Cómo se llega:** `/nueva`: botón «Nuevo» de la búsqueda. `/:id`: enlace del medicamento en la tabla.
- **Permiso de ruta:** `/formulaciones-clinicas/nueva` → `FormulacionesClinicas:Leer` + `FormulacionesClinicas:Crear`; `/formulaciones-clinicas/:id` → `FormulacionesClinicas:Leer`.
- **Migas y cabecera:** título «Formulaciones Clínicas»; migas «Inicio» / «Catálogos» / «Formulaciones Clínicas» (con enlace a `/formulaciones-clinicas`); botón volver a `/formulaciones-clinicas` (`.html:8-12`, `.ts:86-90`).
- **Modos** (`ngOnInit`, `.ts:130-143`): sin `id` → **crear**; con `id` → **ver** (todo deshabilitado) y «Editar» habilita ingrediente, concentración, unidad y código. **El medicamento no se puede cambiar una vez creada la formulación** (`enterEditMode` lo deja deshabilitado, `.ts:196`; el cuerpo del `PUT` no lo incluye).

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar |
|---|---|---|---|
| Encabezado con «Datos del registro» + «Editar» | No | Sí | No |
| Medicamento | Editable | Deshabilitado | Deshabilitado |
| Ingrediente, concentración, unidad, código | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:66-84`, `.html:42-155`)

| Etiqueta (clave) | Control | Tipo de control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Medicamento (`formulacionesClinicas.fields.producto`) | `productoCtrl` (suelto) | `matInput` con autocompletado, columna 6/12; placeholder «Buscar medicamento...» (`autocomplete.producto-placeholder`); opciones = nombre comercial de `GET /api/v1/medicamentos?nombreComercial=<texto>&pageNumber=1&pageSize=20` (solo con texto) | Sí (solo al crear) | Debe elegirse una opción y el texto debe seguir coincidiendo con el nombre elegido (`refSelectionValid`, `.ts:213-215`); `productoValid = !isNew() \|\| !!selectedProducto()` (`.ts:233`) | «Este campo es obligatorio» (`general.errors.required`, `.html:66-70`, solo en crear tras pulsar Guardar) | Sí / No |
| Ingrediente Activo (`formulacionesClinicas.fields.ingrediente`) | `ingredienteCtrl` | Igual, columna 6/12; placeholder «Buscar ingrediente activo...» (`autocomplete.ingrediente-placeholder`); opciones = denominación DCI de `GET /api/v1/ingredientesactivos?denominacionDci=<texto>&pageNumber=1&pageSize=20` | Sí | Debe elegirse una opción (`.ts:226-228,234`) | «Este campo es obligatorio» (`.html:98-102`) | Sí / Sí |
| Concentración de Dosis (`formulacionesClinicas.fields.concentracionDosis`) | `concentracionDosis` | `matInput type="number" step="0.0001"`, columna 4/12 | Sí | `Validators.required`, `Validators.min(0.0001)` (`.ts:82`) | «Este campo es obligatorio» (`.html:113`) para **cualquier** error del campo (incluido un valor menor que 0,0001 o cero) | Sí / Sí |
| Unidad de Medida (`formulacionesClinicas.fields.unidadMedida`) | `unidadMedidaCtrl` | Autocompletado, columna 4/12; placeholder «Buscar unidad de medida...» (`autocomplete.unidad-medida-placeholder`); opciones = descripción de `GET /api/v1/unidadesmedida?descripcion=<texto>&pageNumber=1&pageSize=20` | Sí | Debe elegirse una opción (`.ts:229-231,234`) | «Este campo es obligatorio» (`.html:142-146`) | Sí / Sí |
| Código TE Orange (`formulacionesClinicas.fields.codigoTeOrange`) | `codigoTeOrange` | `matInput`, `maxlength="5"`, columna 4/12 | No | `Validators.maxLength(5)` (`.ts:83`) | Sin `mat-error` (el atributo `maxlength` impide pasar de 5) | Sí / Sí |

Sin coincidencias: cuando el texto escrito no encuentra coincidencias aparece la opción deshabilitada «Sin resultados» (`autocomplete.no-results`). Las opciones se eligen con clic (`(click)="selectProducto(p)"` etc.); si cambias el texto después de elegir, `save()` descarta la elección y marca «Este campo es obligatorio». El código TE Orange se envía como `undefined` si queda vacío (`.ts:260,269`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado | Modo ver y `*appPermiso="'Actualizar'"` (`FormulacionesClinicas:Actualizar`, `.html:32`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie | Crear o editar | Nunca | `cancelEdit()` (`.ts:199-211`): crear → `/formulaciones-clinicas`; editar → restaura la copia leída y vuelve a ver |
| Guardar (`actions.save`); spinner | Pie | Crear o editar | `saving()` | `save()` (`.ts:217-273`): crear → `POST /api/v1/formulacionesclinicas` `{productoId, ingredienteId, concentracionDosis, unidadMedidaId, codigoTeOrange?}`; editar → `PUT /api/v1/formulacionesclinicas/:id` `{ingredienteId, concentracionDosis, unidadMedidaId, codigoTeOrange?}`. En ambos casos aviso de éxito y navega a `/formulaciones-clinicas` |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie | Solo en editar y `*appPermiso="'Eliminar'"` (`FormulacionesClinicas:Eliminar`, `.html:176`) | `deleting()` | `confirmDelete()` → diálogo |

**Diálogos que abre:** `ConfirmDialogComponent` al eliminar (`.ts:275-286`), 400 px: título «Confirmar eliminación» (`formulacionesClinicas.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta formulación clínica?» (`detail.delete-confirm-message`), botones «Cancelar» y «Confirmar».

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Formulación clínica guardada correctamente» (`formulacionesClinicas.detail.save-success`) | `.ts:241` |
| Guardar: respuesta 409 (cualquier código; la API usa `Formulacion.Ingrediente.Duplicado`) | Error | «Ya existe una formulación con los datos ingresados» (`detail.save-conflict`) | `.ts:244-251` |
| Guardar: cualquier otro error (404 de medicamento, ingrediente o unidad; 400 de validación) | Error | «Error al guardar la formulación clínica» (`detail.save-error`) | `.ts:244-251` |
| Eliminación correcta | Éxito | «Formulación clínica eliminada correctamente» (`detail.delete-success`); navega a `/formulaciones-clinicas` | `.ts:292-293` |
| Eliminar: cualquier error | Error | «Error al eliminar la formulación clínica» (`detail.delete-error`) | `.ts:295-298` |
| Error al cargar | Error | «Error al cargar la formulación clínica» (`detail.load-error`) | `.ts:155` |

No usa `saveErrorKey`. El mapa de eliminar está en §6.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir existente | `FormulacionClinicaService.getById` | `GET /api/v1/formulacionesclinicas/{id}` | `FormulacionesClinicas:Leer` (`FormulacionesClinicasController.cs:32`) |
| Autocompletar medicamento | `MedicamentoService.search` | `GET /api/v1/medicamentos` | `Medicamentos:Leer` |
| Autocompletar ingrediente | `IngredienteActivoService.search` | `GET /api/v1/ingredientesactivos` | `IngredientesActivos:Leer` |
| Autocompletar unidad | `UnidadMedidaService.search` | `GET /api/v1/unidadesmedida` | `UnidadesMedida:Leer` |
| Crear | `create` | `POST /api/v1/formulacionesclinicas` | `FormulacionesClinicas:Crear` (`:40`) |
| Editar | `update` | `PUT /api/v1/formulacionesclinicas/{id}` | `FormulacionesClinicas:Actualizar` (`:50`) |
| Eliminar | `delete` | `DELETE /api/v1/formulacionesclinicas/{id}` | `FormulacionesClinicas:Eliminar` (`:64`) |

La pantalla necesita permisos de lectura de **otros tres módulos** (`Medicamentos`, `IngredientesActivos`, `UnidadesMedida`) para poder elegir valores; el SPA no los comprueba.

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga, spinner (`.html:14-17`).
- En ver/editar de un registro existente, los tres nombres (medicamento, ingrediente, unidad) vienen del detalle (`productoNombre`, `ingredienteNombre`, `unidadMedidaNombre`; `null` se muestra vacío).
- Los autocompletados de esta pantalla no buscan con el campo vacío (devuelven lista vacía, `.ts:98,110,122`).

**Observaciones**
- El formulario valida lo mismo que la API: concentración > 0 (`CreateFormulacionClinicaCommandValidator`, `ConcentracionDosis.GreaterThan(0)`), código TE Orange hasta 5 caracteres, identificadores obligatorios (RN-FOR-002 a RN-FOR-008). La API no pone tope máximo a la concentración; la base guarda 4 decimales (RN-FOR-006).
- Un cero o negativo en la concentración muestra el texto «Este campo es obligatorio», que no describe el problema.
- Eliminar no valida dependencias en el backend (`DeleteFormulacionClinicaCommandHandler.cs`) y el SPA no distingue el 409.
- No hay specs de formulaciones clínicas.

---

## 4. Proveedores

Carpeta `app/pages/proveedores`, scope i18n `proveedores` (`provideTranslocoScope('proveedores')` en los componentes y en el diálogo de precios). Componentes enrutados (`proveedores.routes.ts`, `loadChildren` desde `app/app.routes.ts:115-117`): `ProveedoresSearchComponent` (`''`), `ProveedoresDetailComponent` (rutas `nueva` y `:id`) y `ProveedoresCargaMasivaComponent` (`carga-masiva`). Más el diálogo `PrecioProveedorDialogComponent` (sin ruta; §4.5). El orden de rutas importa: `nueva` y `carga-masiva` antes de `:id`.

Novedades de los commits `9cbbe58` (SPA: registro y listado de precios de compra), `606cb69` (API: el registro de precio se guarda de verdad y se expone el historial) y `27597cd`/`01b8260` (no se puede eliminar un proveedor con precios de compra). La parte de precios está en §4.3 y §4.5.

### 4.1 `/proveedores` — Proveedores (búsqueda)

- **Componente:** `ProveedoresSearchComponent` (`app/pages/proveedores/proveedores-search/proveedores-search.component.ts`, `.html`). Scope `proveedores`.
- **Para qué sirve:** buscar proveedores por razón social, RUC/identificador tributario y rango de tasa de cumplimiento; abrirlos, exportar la lista y entrar a crear o a cargar proveedores masivamente.
- **Cómo se llega:** menú «Catálogos» → «Proveedores»; botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar.
- **Permiso de ruta:** `Proveedores:Leer`.
- **Migas y cabecera:** título «Proveedores» (`proveedores.title`); migas «Inicio» / «Catálogos» / «Proveedores» (`breadcrumbs.proveedores`, activa); sin botón volver. Tarjeta de filtros con título «Búsqueda de Proveedores» (`proveedores.search.title`) y el subtítulo común.
- **Modos:** uno solo.

**Filtros de búsqueda** (`.ts:47-53`, `.html:37-63`)

| Etiqueta (clave) | Control | Tipo | Opciones o formato | Valor inicial |
|---|---|---|---|---|
| Razón Social (`proveedores.fields.razonSocial`) | `razonSocial` | Texto con lupa; Enter busca; mitad de ancho | «Contiene» (`GetProveedoresQueryHandler.cs`) | Vacío |
| RUC / Tax ID (`proveedores.fields.taxId`) | `taxId` | Texto con lupa; Enter busca; mitad de ancho | «Contiene» | Vacío |
| Tasa de Cumplimiento (`proveedores.fields.tasaCumplimiento`) | `tasaMin`, `tasaMax` (señales, fuera del formulario) | **Deslizador de rango** `app-range-slider-field` ([19a](19a-acceso-layout-y-compartidos.md)), ancho completo | Mínimo 0, máximo 100, paso 1, sin unidad; la etiqueta muestra «Tasa de Cumplimiento: **0** — **100**» con los valores actuales. Solo se envían al servidor si difieren de los extremos: `tasaCumplimientoDesde` si el mínimo ≠ 0 y `tasaCumplimientoHasta` si el máximo ≠ 100 (`.ts:84-85`) | 0 — 100 |

**Tabla de resultados** (`.html:83-140`, columnas `.ts:61`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Razón Social (`proveedores.table.razonSocial`) | `razonSocial` | Texto | Sí: `[routerLink]="[row.id]"` → `/proveedores/:id` |
| RUC / Tax ID (`proveedores.table.taxId`) | `taxId` | Texto | No |
| Tasa de Cumplimiento (`proveedores.table.tasaCumplimiento`) | `tasaCumplimiento` | Número tal como lo devuelve la API | No |

La tabla no muestra términos financieros ni calificación ESG. Orden por defecto: servidor, por razón social y luego identificador. Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:120`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`), icono `upload` | Encabezado de la tarjeta, derecha | `*appPermiso="'Crear'"` (= `Proveedores:Crear`, `.html:19`) | Nunca | `[routerLink]="['carga-masiva']"` → `/proveedores/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/proveedores/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` (`.ts:108-117`): vacía los textos, **devuelve el deslizador a 0 — 100**, oculta la tabla y borra el estado guardado |
| Buscar (`actions.search`); spinner | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/proveedores` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/proveedores/exportar` |
| Enlace con la razón social | Primera columna | Con resultados | — | Navega a `/proveedores/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar proveedores» (`proveedores.search.search-error`) | `proveedores-search.component.ts:103` |
| Falla la exportación | Error | El mismo texto, con referencia de seguimiento si la respuesta trae `traceId` (`showServerError`) | `.ts:152` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar | `ProveedorService.search` (`app/pages/proveedores/services/proveedor.service.ts`) | `GET /api/v1/proveedores?pageNumber&pageSize[&razonSocial][&taxId][&tasaCumplimientoDesde][&tasaCumplimientoHasta]` | `Proveedores:Leer` (`ProveedoresController.cs:37`) |
| Exportar | `exportar` | `POST /api/v1/proveedores/exportar` (blob) | `Proveedores:Leer` (`:78`) |

**Exportación**
- Cuerpo (`.ts:129-142`): `headers` con 3 textos, en orden: «Razón Social» (`table.razonSocial`), «RUC / Tax ID» (`table.taxId`), «Tasa de Cumplimiento» (`table.tasaCumplimiento`); más `razonSocial`, `taxId` y los límites de tasa solo si no están en sus extremos. Archivo: «proveedores.xlsx» (`proveedores.search.export-filename`). El Excel del servidor lleva razón social, identificador tributario y tasa (`ProveedoresController.cs:78-108`).
- Carga masiva: botón «Carga Masiva» → §4.4.

**Comportamientos particulares:** restauración de filtros al volver del detalle: se restauran los dos textos, la página y el tamaño, pero **no el deslizador**: `searchState.save` guarda solo `searchForm.getRawValue()` (`.ts:96`), de modo que al volver la tabla se repite con la tasa en 0 — 100 aunque la búsqueda anterior la hubiese acotado. Sale de leer el código; no se ejecutó.

**Observaciones**
- La fila vacía dice «Resultados», no «Sin resultados».
- Los archivos `.html` y `.ts` de esta carpeta empiezan con BOM; sin efecto funcional.

### 4.2 `/proveedores/nueva` y `/proveedores/:id` — Proveedores (detalle)

- **Componente:** `ProveedoresDetailComponent` (`app/pages/proveedores/proveedores-detail/proveedores-detail.component.ts`, `.html`). Scope `proveedores`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear un proveedor, ver sus datos y, con permisos, editarlo o eliminarlo; en modo ver muestra además el historial de precios de compra (§4.3).
- **Cómo se llega:** `/nueva`: botón «Nuevo» de la búsqueda. `/:id`: enlace de la razón social en la tabla.
- **Permiso de ruta:** `/proveedores/nueva` → `Proveedores:Leer` + `Proveedores:Crear`; `/proveedores/:id` → `Proveedores:Leer`.
- **Migas y cabecera:** título «Proveedores»; migas «Inicio» / «Catálogos» / «Proveedores» (con enlace a `/proveedores`); botón volver a `/proveedores` (`.html:3-7`, `.ts:72-76`).
- **Modos** (`ngOnInit`, `.ts:78-89`): sin `id` → **crear**; con `id` → **ver** (formulario deshabilitado; carga el proveedor **y** la primera página de precios) y «Editar» lo habilita.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar |
|---|---|---|---|
| Encabezado con «Datos del registro» + «Editar» | No | Sí | No |
| Cinco campos | Editables | Deshabilitados | Editables |
| Tarjeta «Precios de compra» | No | Sí | **No** (se oculta al editar) |
| Botones del pie | Cancelar, Guardar | — | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:64-70`, `.html:37-89`)

| Etiqueta (clave) | `formControlName` | Tipo de control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Razón Social (`proveedores.fields.razonSocial`) | `razonSocial` | `matInput`, `maxlength="200"`, columna 8/12 | Sí | `required`, `maxLength(200)` (`.ts:65`) | «Este campo es obligatorio» (`general.errors.required`, `.html:42`) | Sí / Sí |
| RUC / Tax ID (`proveedores.fields.taxId`) | `taxId` | `matInput`, `maxlength="30"`, columna 4/12 | Sí | `required`, `maxLength(30)` (`.ts:66`) | «Este campo es obligatorio» (`.html:50`) | Sí / Sí |
| Tasa de Cumplimiento (`proveedores.fields.tasaCumplimiento`) | `tasaCumplimiento` | `matInput type="number" min="0" max="100"`, columna 4/12 | Sí | `required`, `min(0)`, `max(100)` (`.ts:69`) | «Este campo es obligatorio» (`.html:63`) para cualquier error (también un valor fuera de rango) | Sí / Sí. Valor inicial al crear: 100 |
| Calificación ESG (`proveedores.fields.calificacionEsg`) | `calificacionEsg` | `matInput type="number" min="0" max="100"`, columna 4/12 | No | `min(0)`, `max(100)` (`.ts:68`) | Sin `mat-error`: un valor fuera de rango invalida el formulario (Guardar no envía) sin texto visible | Sí / Sí |
| Términos Financieros (`proveedores.fields.terminosFinancieros`) | `terminosFinancieros` | `textarea` de 3 filas, `maxlength="500"`, columna 12/12 | No | `maxLength(500)` (`.ts:67`) | Sin `mat-error` (el atributo `maxlength` impide pasar de 500) | Sí / Sí |

No hay ayudas ni placeholders (las claves `proveedores.help.*` no se usan). Si la calificación ESG o los términos quedan vacíos, se envían como `undefined` (`.ts:178-179,196-197`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado | Modo ver y `*appPermiso="'Actualizar'"` (`Proveedores:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie | Crear o editar | Nunca | `cancelEdit()` (`.ts:151-165`): crear → `/proveedores`; editar → restaura la copia leída y vuelve a ver |
| Guardar (`actions.save`); spinner | Pie | Crear o editar | `saving()` | `save()` (`.ts:167-214`): si el formulario es inválido marca todo como tocado y no envía. Crear → `POST /api/v1/proveedores`, aviso y navega a `/proveedores`. Editar → `PUT /api/v1/proveedores/:id` (el cuerpo incluye `id`), aviso, vuelve a ver y recarga el proveedor |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie | Solo en editar y `*appPermiso="'Eliminar'"` (`Proveedores:Eliminar`, `.html:110`) | `deleting()` | `confirmDelete()` → diálogo |
| Registrar precio (`proveedores.precios.register-btn`) | Tarjeta de precios | Ver §4.3 | Nunca | `openRegistrarPrecio()` |

**Diálogos que abre:** `ConfirmDialogComponent` al eliminar (`.ts:216-227`), 400 px: título «Confirmar eliminación» (`proveedores.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar este proveedor?» (`detail.delete-confirm-message`), botones «Cancelar» y «Confirmar». Y el diálogo de precios (§4.5).

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Proveedor guardado correctamente» (`proveedores.detail.save-success`) | `.ts:183,202` |
| Código `Proveedor.TaxId.Duplicado` (409) | Error | «Ya existe un proveedor con ese identificador tributario.» (`detail.duplicate-tax-id`) | `.ts:19-22,188,210` |
| Código `Proveedor.RazonSocial.Duplicado` (409) | Error | «Ya existe un proveedor con esa razón social.» (`detail.duplicate-razon-social`) | `.ts:19-22,188,210` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos `errors.db*` (§0) | `save-error-key.util.ts` |
| Cualquier otro error al guardar (incluidos los de validación del dominio, p. ej. `ProveedorDistribuidor.TerminosFinancieros`) | Error | «Error al guardar el proveedor» (`detail.save-error`) | `.ts:188,210` |
| Eliminación correcta | Éxito | «Proveedor eliminado correctamente» (`detail.delete-success`); navega a `/proveedores` | `.ts:233-234` |
| Eliminar: respuesta 409 | Error | «No puedes eliminar este proveedor porque tiene precios de compra registrados.» (`detail.delete-in-use`) | `.ts:236-243` |
| Eliminar: otro error | Error | «Error al eliminar el proveedor» (`detail.delete-error`) | `.ts:236-243` |
| Error al cargar el proveedor | Error | «Error al cargar el proveedor» (`detail.load-error`) | `.ts:141` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir existente y tras editar | `ProveedorService.getById` | `GET /api/v1/proveedores/{id}` | `Proveedores:Leer` (`ProveedoresController.cs:44`) |
| Abrir existente, cambiar de página, tras registrar precio | `getPrecios` | `GET /api/v1/proveedores/{id}/precios?pageNumber&pageSize` | `Proveedores:Leer` (`:52`) |
| Crear | `create` | `POST /api/v1/proveedores` | `Proveedores:Crear` (`:21`) |
| Editar | `update` | `PUT /api/v1/proveedores/{id}` | `Proveedores:Actualizar` (`:60`) |
| Eliminar | `delete` | `DELETE /api/v1/proveedores/{id}` | `Proveedores:Eliminar` (`:69`) |
| Registrar precio (desde el diálogo) | `registrarPrecio` | `POST /api/v1/proveedores/precios` | `Proveedores:RegistrarPrecioProveedor` (`:29`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares**
- Mientras carga el proveedor, un spinner reemplaza todo, incluida la tarjeta de precios (`.html:9-13`). Tras guardar una edición se vuelve a mostrar el spinner (por `load()`), pero los precios no se recargan.
- Los precios solo se piden en modo ver de un registro existente (en `/nueva` no se piden ni se muestra la tarjeta).

**Observaciones**
- **Longitudes del formulario distintas de las de la API:** el SPA limita el RUC/Tax ID a **30** caracteres (`.ts:66`) y los términos financieros a **500** (`.ts:67`); la API admite **50** para el identificador y **100** para los términos (`sofia-apis/src/SOFIA.Domain/Entities/ProveedorDistribuidor.cs:36-44`; RN-PRV-003 y RN-PRV-004 de 01b). Un texto de 101 a 500 caracteres en términos financieros pasa el formulario y la API lo rechaza con el código `ProveedorDistribuidor.TerminosFinancieros`, no mapeado, de modo que el usuario ve «Error al guardar el proveedor». Sale de leer el código; no se ejecutó.
- El validador `CreateProveedorCommandValidator` está vacío (`// TODO: Add validation rules`, `sofia-apis/src/SOFIA.Application/Proveedores/Commands/CreateProveedor/CreateProveedorCommandValidator.cs:9-12`); las reglas las aplican los guards de la entidad.
- Un valor de solo espacios en razón social o RUC pasa `Validators.required` de Angular; la entidad lo rechaza (`ProveedorDistribuidor.cs:24,34`) con un código no mapeado y se vería el mensaje genérico (no ejecutado).
- Calificación ESG fuera de rango bloquea Guardar sin mostrar ningún mensaje.
- **Actualiza 01b:** RN-PRV-010 («eliminar no valida dependencias») ya no es cierto; ver §6.

### 4.3 Sección «Precios de compra» del detalle (registro e historial)

Vive en la misma ficha `/proveedores/:id` (`proveedores-detail.component.html:123-207`, contenedor `id="precios-card"`).

- **Cuándo se ve:** `!isNew() && !isEditMode()`, es decir solo en modo ver de un proveedor existente. No tiene `*appPermiso` propio: se muestra a todo usuario que haya entrado a la ficha, y su lectura depende de `Proveedores:Leer` (permiso del endpoint `GET /{id}/precios`). El **botón de registro** sí está protegido.
- **Cabecera de la tarjeta:** título «Precios de compra» (`proveedores.precios.title`) y subtítulo «Costo por unidad base que te cobra este proveedor, con su descuento ya aplicado. Cada producto tiene un solo precio vigente y los anteriores quedan en el historial.» (`precios.subtitle`).
- **Botón «Registrar precio»** (`precios.register-btn`, `id="registrar-precio-btn"`): visible con `*appPermiso="'RegistrarPrecioProveedor'"` (`.html:134`), que equivale a `Proveedores:RegistrarPrecioProveedor` porque la URL pertenece al módulo `Proveedores` (`permiso.directive.ts:16-23`). El rol `Admin` pasa siempre. Al pulsarlo abre el diálogo de §4.5 (`openRegistrarPrecio()`, `.ts:112-123`, ancho 520 px) con el identificador del proveedor; si el diálogo se cierra con `true` (precio registrado), vuelve a la primera página y recarga la lista; con cancelar no recarga.

**Tabla** (`preciosColumns`, `.ts:59`; `.html:144-197`)

| Encabezado (clave) | Dato | Formato |
|---|---|---|
| Producto (`proveedores.precios.cols.producto`) | `productoNombre` | Nombre comercial del medicamento (el servidor lo resuelve; vacío si el medicamento ya no existe) |
| Costo por unidad base (`precios.cols.costo`) | `costoPorUnidadBase` | «S/ » + número con exactamente 4 decimales (`number: '1.4-4'`). Es el costo **neto**, con el descuento ya aplicado |
| Vigente desde (`precios.cols.desde`) | `fechaInicioVigencia` | `dd/MM/yyyy` (pipe `date`, formato fijo, hora local del navegador) |
| Vigente hasta (`precios.cols.hasta`) | `fechaFinVigencia` | `dd/MM/yyyy`; si es nulo, el texto «Vigente» (`precios.vigente`) |

El texto «Vigente» depende solo de que `fechaFinVigencia` sea nula; el SPA no usa el indicador `vigente` que devuelve la API (`PrecioProveedor.vigente` está en el modelo, `proveedor.model.ts:59`, pero ninguna plantilla lo lee). La tabla no tiene enlaces ni acciones por fila; no se puede editar ni borrar un precio.

- **Orden (servidor):** por fecha de inicio de vigencia descendente, luego fecha de creación descendente, luego identificador (`GetPreciosProveedorQueryHandler.cs`, `OrderByDescending(h => h.FechaInicioVigencia).ThenByDescending(h => h.CreatedAt).ThenBy(h => h.Id)`).
- **Paginación:** `mat-paginator` propio con 5, 10 y 25 filas, 10 por defecto, botones primera/última (`.html:199-205`). Cada cambio llama a `loadPrecios()` (`onPreciosPageChange`, `.ts:106-110`).
- **Estados:** sin precios y sin estar cargando, «Este proveedor aún no tiene precios registrados.» (`precios.empty`, `.html:191-193`); mientras carga, la fila vacía queda sin texto. Error de carga: «Error al cargar los precios del proveedor» (`precios.load-error`, `.ts:101`).
- **Llamadas:** `GET /api/v1/proveedores/{id}/precios?pageNumber&pageSize` (`Proveedores:Leer`, `ProveedoresController.cs:52`; respuesta paginada de `PrecioProveedorDto(Id, ProductoId, ProductoNombre, CostoPorUnidadBase, FechaInicioVigencia, FechaFinVigencia, Vigente)`; proveedor inexistente: 404 `Proveedor.NotFound`) y, desde el diálogo, `POST /api/v1/proveedores/precios` (`Proveedores:RegistrarPrecioProveedor`, `:29`).
- **Qué hace el servidor al registrar** (`sofia-apis/src/SOFIA.Application/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandler.cs`): comprueba que existan el proveedor (404 `Proveedor.NotFound`, `:16`) y el medicamento (404 `Medicamento.NotFound`, `:22`); calcula el costo neto como precio × (1 − descuento/100) redondeado a 4 decimales (`:26`); crea un registro del historial con inicio de vigencia en el instante actual (UTC) y sin fin (`:28`); cierra en ese mismo instante los precios abiertos del mismo proveedor y producto (`:36-48`); guarda y devuelve el identificador. Así, cada producto tiene un solo precio vigente con ese proveedor y los anteriores quedan con fecha de fin. El plazo de entrega se guarda en 0 y la cantidad mínima de compra en su valor por defecto (1) (`HistorialPrecioProveedor.Create`).
- **Actualiza 01b:** RN-PRV-018 a RN-PRV-020 describían un registro que "no guarda nada"; con `606cb69` sí persiste. Los guards de RN-PRV-021 a RN-PRV-024 siguen vigentes.

### 4.4 `/proveedores/carga-masiva` — Carga Masiva (proveedores)

- **Componente:** `ProveedoresCargaMasivaComponent` (`app/pages/proveedores/proveedores-carga-masiva/proveedores-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `proveedores` (no aporta textos).
- **Para qué sirve:** cargar varios proveedores desde un Excel; flujo común en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/proveedores`.
- **Permiso de ruta:** `Proveedores:Leer` + `Proveedores:Crear`.
- **Migas y cabecera:** título «Carga Masiva»; migas «Inicio» / «Catálogos» / «Proveedores» (enlace a `/proveedores`) / «Carga Masiva» (activa); botón volver a `/proveedores`.

**Configuración que pasa** (`.ts:18-36`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` | `GET {api}/api/v1/proveedores/plantilla` — `Proveedores:Crear` (`ProveedoresController.cs:110`); se baja como `plantilla-proveedores.xlsx` |
| `previewUrl` | `POST {api}/api/v1/proveedores/previsualizar` — `Proveedores:Crear` (`:119`) |
| `saveUrl` | `POST {api}/api/v1/proveedores/carga-masiva` — `Proveedores:Crear` (`:137`) |
| `backRoute` | `/proveedores` |
| Columnas de la vista previa (clave → etiqueta de la raíz) | `razonSocial` → «Razón Social» (`fields.razonSocial`, obligatoria); `taxId` → «Tax ID» (`fields.taxId` de la raíz, que **no** dice «RUC / Tax ID»; obligatoria); `terminosFinancieros` → «Términos Financieros» (`fields.terminosFinancieros`, opcional); `calificacionEsg` → «Calificación ESG» (`fields.calificacionEsg`, opcional); `tasaCumplimiento` → «Tasa de Cumplimiento» (`fields.tasaCumplimiento`, obligatoria). Después, errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Reglas en RN-PRV-013 a RN-PRV-017 de 01b. Las claves de datos coinciden con `ProveedorImportRow(RazonSocial, TaxId, TerminosFinancieros, CalificacionEsg, TasaCumplimiento)` (`CargaMasivaProveedoresCommand.cs:5-10`) y con las columnas de la plantilla del controlador (`ProveedoresController.cs:110-115`: `RazonSocial`, `TaxId`, `TerminosFinancieros`, `CalificacionEsg`, `TasaCumplimiento`).

### 4.5 Diálogo `precio-proveedor-dialog` — «Registrar precio de compra»

- **Componente:** `PrecioProveedorDialogComponent` (`app/pages/proveedores/precio-proveedor-dialog/precio-proveedor-dialog.component.ts`, `.html`). Scope `proveedores`. Datos de entrada `PrecioProveedorDialogData { proveedorId }`. Se abre con `MatDialog.open` desde `ProveedoresDetailComponent.openRegistrarPrecio()` con ancho 520 px.
- **Para qué sirve:** registrar un nuevo precio de compra de un producto con este proveedor. El nuevo precio reemplaza al vigente de ese producto con ese proveedor; el anterior queda en el historial.
- **Cómo se llega:** botón «Registrar precio» de la tarjeta de precios (§4.3), con `Proveedores:RegistrarPrecioProveedor`.
- **Texto introductorio:** título «Registrar precio de compra» (`proveedores.precios.dialog.title`, `.html:2`) y párrafo «El nuevo precio reemplaza al vigente de este producto con este proveedor; el anterior queda en el historial.» (`precios.dialog.hint`, `.html:5`).

**Campos del formulario** (`.ts:62-66`, `.html:7-83`)

| Etiqueta (clave) | `formControlName` | Tipo de control | Obligatorio | Validaciones | Mensaje de error | Ayuda |
|---|---|---|---|---|---|---|
| Producto (`precios.dialog.producto`) | `producto` | `matInput` con autocompletado, ancho completo. El valor del control es el medicamento elegido (un objeto) o, mientras no se elige ninguno, el texto escrito. Placeholder «Escribe para buscar un producto» (`precios.dialog.producto-placeholder`) | Sí | `Validators.required` y `productoSeleccionado` (`.ts:33-36,63`): solo vale un valor **objeto**, es decir una opción elegida de la lista; texto libre no vale | «Selecciona un producto de la lista» (`precios.dialog.errors.producto`) | Al escribir (300 ms, con texto no vacío tras `trim`), llama a `GET /api/v1/medicamentos?nombreComercial=<texto>&pageNumber=1&pageSize=20` y lista el nombre comercial; sin coincidencias y con texto, opción deshabilitada «Sin resultados» (`precios.dialog.no-results`). Si la búsqueda falla (por ejemplo sin `Medicamentos:Leer`) la lista queda vacía sin aviso (`catchError`, `.ts:84-87`) |
| Precio de compra por unidad base (S/) (`precios.dialog.precio`) | `precioCompra` | `matInput type="number" step="0.0001"`, 7/12 de ancho | Sí | `required`, `min(0.0001)`, `max(9999999.9999)` (`.ts:19,64`) | Vacío: «Este campo es obligatorio» (`general.errors.required`); mayor que el máximo: «El precio es demasiado alto» (`precios.dialog.errors.precio-max`); cualquier otro (cero o negativo): «Debe ser mayor que 0» (`precios.dialog.errors.precio-min`) (`.html:43-49`) | — |
| Descuento (%) (`precios.dialog.descuento`) | `descuentoPorcentaje` | `matInput type="number" step="0.01"`, 5/12 de ancho | Sí | `required`, `min(0)`, `max(100)` (`.ts:65`) | Vacío: «Este campo es obligatorio»; otro: «Debe estar entre 0 y 100» (`precios.dialog.errors.descuento-rango`) (`.html:63-67`) | Valor inicial 0 |
| Costo neto por unidad base (`precios.dialog.costo-neto`) | — (solo lectura, no es un control) | Texto con el resultado: «S/ » + número (`number: '1.2-4'`, entre 2 y 4 decimales), o «—» mientras el precio o el descuento no sean válidos | — | Calculado en pantalla: `precioCompra × (1 − descuento/100)` redondeado a 4 decimales (`costoNeto()`, `.ts:39-42`), la misma regla que aplica el servidor al guardar | — | Se actualiza al teclear (`toSignal` sobre `valueChanges`, `.ts:68-74`) |

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Cancelar (`actions.cancel`) | Pie del diálogo, derecha | Siempre | `saving()` | `cancel()`: cierra el diálogo devolviendo `false` |
| Guardar (`actions.save`); spinner mientras guarda | Pie, derecha | Siempre | `saving()` | `save()` (`.ts:98-122`): si el formulario es inválido marca todos los campos como tocados y no envía; si es válido llama a `POST /api/v1/proveedores/precios` con `{ proveedorId, medicamentoId, precioCompra, descuentoPorcentaje }` |

Los dos botones son `type="button"` y el formulario no tiene botón de envío (el `(ngSubmit)` de `.html:7` queda sin disparador; Enter no guarda: sin botón de envío, no hay envío implícito). No hay botón de cerrar adicional; Esc o clic fuera cierran el diálogo con el comportamiento por defecto de Angular Material (el padre trata cualquier valor distinto de `true` como "sin cambios").

**Al confirmar:** éxito → aviso «Precio registrado correctamente» (`precios.dialog.success`), el diálogo se cierra con `true` y el detalle recarga la lista desde la página 1. Error → el diálogo permanece abierto y el botón se habilita de nuevo.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Precio registrado | Éxito | «Precio registrado correctamente» (`proveedores.precios.dialog.success`) | `precio-proveedor-dialog.component.ts:114` |
| Código `Proveedor.NotFound` (404) | Error | «El proveedor ya no existe.» (`precios.dialog.errors.proveedor-not-found`) | `.ts:22,119` |
| Código `Medicamento.NotFound` (404) | Error | «El producto seleccionado ya no existe.» (`precios.dialog.errors.medicamento-not-found`) | `.ts:23,119` |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | Error | Los tres textos `errors.db*` (§0) | `save-error-key.util.ts` |
| Cualquier otro error (incluido 400 de validación `RegistrarPrecioProveedorCommand.PrecioCompra`, etc.) | Error | «Error al registrar el precio» (`precios.dialog.save-error`) | `.ts:119` |

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Autocompletar producto | `MedicamentoService.search` | `GET /api/v1/medicamentos?nombreComercial=…&pageNumber=1&pageSize=20` | `Medicamentos:Leer` |
| Guardar | `ProveedorService.registrarPrecio` (`proveedor.service.ts`) | `POST /api/v1/proveedores/precios` | `Proveedores:RegistrarPrecioProveedor` (`ProveedoresController.cs:29`) |

**Validaciones del backend** (`RegistrarPrecioProveedorCommandValidator.cs`): `ProveedorId` y `MedicamentoId` no vacíos; `PrecioCompra` mayor que 0 y como máximo 9 999 999,9999 (constante `PrecioCompraMaximo`, igual a `PRECIO_COMPRA_MAX` del SPA); `DescuentoPorcentaje` entre 0 y 100 inclusive. Coinciden con las del formulario. Los códigos de error de validación (`RegistrarPrecioProveedorCommand.<Propiedad>`) no están mapeados.

**Observaciones**
- Specs: `precio-proveedor-dialog.component.spec.ts` (10 casos): descuento y redondeo, rango del precio y descuento, producto elegido de la lista, no envía si es inválido, costo neto en vivo, envío correcto, mensaje de error traducido y genérico, búsqueda de productos y falta de permiso de lectura.
- El diálogo no comprueba el permiso `Proveedores:RegistrarPrecioProveedor`: lo hace el botón que lo abre.
- Dentro del diálogo no se puede elegir una fecha de vigencia: el servidor usa el instante actual.

---

## 5. Aseguradoras

Carpeta `app/pages/aseguradoras`, scope i18n `aseguradoras` (`provideTranslocoScope('aseguradoras')`). Componentes enrutados (`aseguradoras.routes.ts`, `loadChildren` desde `app/app.routes.ts:120-122`): `AseguradorasSearchComponent` (`''`), `AseguradorasDetailComponent` (rutas `nueva` y `:id`) y `AseguradorasCargaMasivaComponent` (`carga-masiva`). El orden importa: `nueva` y `carga-masiva` antes de `:id`.

Particularidades del módulo: la pantalla se llama «Aseguradoras» pero el módulo de permisos es **`Seguros`** y la API vive bajo `/api/v1/seguros` (`SegurosController`; `aseguradora.service.ts:17`); la entidad del backend es `AseguradoraMedica`. El título que ven los usuarios en la cabecera de las pantallas es «Aseguradoras Médicas» (`aseguradoras.title`), distinto del texto del menú y de las migas, «Aseguradoras».

### 5.1 `/aseguradoras` — Aseguradoras Médicas (búsqueda)

- **Componente:** `AseguradorasSearchComponent` (`app/pages/aseguradoras/aseguradoras-search/aseguradoras-search.component.ts`, `.html`). Scope `aseguradoras`.
- **Para qué sirve:** buscar aseguradoras por nombre comercial y por código identificador nacional, abrirlas, exportar la lista y entrar a crear o a cargar aseguradoras masivamente.
- **Cómo se llega:** menú «Catálogos» → «Aseguradoras»; botón volver y migas del detalle y de la carga masiva; redirección tras crear o eliminar.
- **Permiso de ruta:** `Seguros:Leer`.
- **Migas y cabecera:** título «Aseguradoras Médicas» (`aseguradoras.title`); migas «Inicio» / «Catálogos» / «Aseguradoras» (`breadcrumbs.aseguradoras`, activa); sin botón volver. Tarjeta de filtros con título «Búsqueda de Aseguradoras» (`aseguradoras.search.title`) y el subtítulo común.
- **Modos:** uno solo.

**Filtros de búsqueda** (`.ts:45`, `.html:37-58`; dos columnas de igual ancho)

| Etiqueta (clave) | Control | Tipo | Formato | Valor inicial |
|---|---|---|---|---|
| Nombre Comercial (`aseguradoras.fields.nombreComercial`) | `nombreComercial` | Texto con lupa; Enter busca | «Contiene» (RN-ASE-008) | Vacío |
| Código Identificador Nacional (`aseguradoras.fields.codigoIdentificadorNacional`) | `codigoIdentificadorNacional` | Texto con lupa; Enter busca | «Contiene» | Vacío |

**Tabla de resultados** (`.html:78-117`, columnas `.ts:53`)

| Encabezado (clave) | Dato | Formato | Enlace |
|---|---|---|---|
| Nombre Comercial (`aseguradoras.table.nombreComercial`) | `nombreComercial` | Texto | Sí: `[routerLink]="[row.id]"` → `/aseguradoras/:id` |
| Código Identificador Nacional (`aseguradoras.table.codigoIdentificadorNacional`) | `codigoIdentificadorNacional` | Texto | No |

Orden: el servidor no define uno explícito (`GetAseguradorasQueryHandler.cs` no tiene `OrderBy`; RN-ASE-008 de 01b), de modo que el orden de las filas no está garantizado. Encabezado «Resultados (N)». Paginación 10/25/50. Fila sin resultados: «Resultados» (`generic-labels.results`, `.html:112`).

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Carga Masiva (`carga-masiva.btn`), icono `upload` | Encabezado de la tarjeta, derecha | `*appPermiso="'Crear'"` (= `Seguros:Crear`, `.html:19`) | Nunca | `[routerLink]="['carga-masiva']"` → `/aseguradoras/carga-masiva` |
| Nuevo (`actions.new`), icono `add` | Igual, a su derecha | `*appPermiso="'Crear'"` (`.html:27`) | Nunca | `goToNew()` → `/aseguradoras/nueva` |
| Limpiar (`actions.clear`) | Pie de filtros | Siempre | Nunca | `clear()` (`.ts:98-105`) |
| Buscar (`actions.search`); spinner | Pie de filtros | Siempre | `loading()` | `search()` → `GET /api/v1/seguros` |
| Exportar (`actions.export`), icono `download` | Bajo la tabla | Tras buscar | Nunca | `exportar()` → `POST /api/v1/seguros/exportar` |
| Enlace con el nombre | Primera columna | Con resultados | — | Navega a `/aseguradoras/:id` |

**Diálogos que abre:** ninguno.

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Falla la búsqueda | Error | «Error al buscar aseguradoras» (`aseguradoras.search.search-error`) | `aseguradoras-search.component.ts:93` |
| Falla la exportación | Error | El mismo texto, con referencia de seguimiento si la respuesta trae `traceId` (`showServerError`) | `.ts:137` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Buscar, paginar o restaurar | `AseguradoraService.search` (`app/pages/aseguradoras/services/aseguradora.service.ts`) | `GET /api/v1/seguros?pageNumber&pageSize[&nombreComercial][&codigoIdentificadorNacional]` | `Seguros:Leer` (`SegurosController.cs:27`) |
| Exportar | `exportar` | `POST /api/v1/seguros/exportar` (blob) | `Seguros:Leer` (`:60`) |

**Exportación**
- Cuerpo (`.ts:117-139`): `headers` con 2 textos, en orden: «Nombre Comercial» (`table.nombreComercial`) y «Código Identificador Nacional» (`table.codigoIdentificadorNacional`); más los dos filtros si tienen valor. Archivo: «aseguradoras.xlsx» (`aseguradoras.search.export-filename`).
- Carga masiva: botón «Carga Masiva» → §5.3.

**Comportamientos particulares:** restauración de filtros al volver del detalle; «Buscar» no vuelve a la página 1.

**Observaciones**
- La fila vacía dice «Resultados», no «Sin resultados».
- Specs: `aseguradoras-search.component.spec.ts` (3 casos) y `aseguradora.service.spec.ts` (7 casos).

### 5.2 `/aseguradoras/nueva` y `/aseguradoras/:id` — Aseguradoras Médicas (detalle)

- **Componente:** `AseguradorasDetailComponent` (`app/pages/aseguradoras/aseguradoras-detail/aseguradoras-detail.component.ts`, `.html`). Scope `aseguradoras`. Las dos rutas usan el mismo componente.
- **Para qué sirve:** crear una aseguradora, ver sus datos y, con permisos, editarla o eliminarla.
- **Cómo se llega:** `/nueva`: botón «Nuevo» de la búsqueda. `/:id`: enlace del nombre en la tabla.
- **Permiso de ruta:** `/aseguradoras/nueva` → `Seguros:Leer` + `Seguros:Crear`; `/aseguradoras/:id` → `Seguros:Leer`.
- **Migas y cabecera:** título «Aseguradoras Médicas»; migas «Inicio» / «Catálogos» / «Aseguradoras» (con enlace a `/aseguradoras`); botón volver a `/aseguradoras` (`.html:3-7`, `.ts:53-57`).
- **Modos** (`ngOnInit`, `.ts:59-69`): sin `id` → **crear**; con `id` → **ver** (formulario deshabilitado) y «Editar» lo habilita.

| Aspecto | Crear (`/nueva`) | Ver (`/:id`) | Editar |
|---|---|---|---|
| Encabezado con «Datos del registro» + «Editar» | No | Sí | No |
| Nombre comercial y código | Editables | Deshabilitados | Editables |
| Botones | Cancelar, Guardar | Editar | Cancelar, Guardar, Eliminar |

**Campos del formulario** (`.ts:48-51`, `.html:37-61`)

| Etiqueta (clave) | `formControlName` | Tipo de control | Obligatorio | Validaciones | Mensaje de error | Crear / Editar |
|---|---|---|---|---|---|---|
| Nombre Comercial (`aseguradoras.fields.nombreComercial`) | `nombreComercial` | `matInput`, `maxlength="150"`, columna 8/12 | Sí | `required`, `maxLength(150)` (`.ts:49`) | «Este campo es obligatorio» (`general.errors.required`, `.html:45`) | Sí / Sí |
| Código Identificador Nacional (`aseguradoras.fields.codigoIdentificadorNacional`) | `codigoIdentificadorNacional` | `matInput`, `maxlength="50"`, columna 4/12 | Sí | `required`, `maxLength(50)` (`.ts:50`) | «Este campo es obligatorio» (`.html:58`) | Sí / Sí |

Sin ayudas ni placeholders.

**Botones y acciones**

| Texto (clave) | Dónde | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Editar (`actions.edit`) | Encabezado | Modo ver y `*appPermiso="'Actualizar'"` (= `Seguros:Actualizar`, `.html:27`) | Nunca | `enterEditMode()` |
| Cancelar (`actions.cancel`) | Pie | Crear o editar | Nunca | `cancelEdit()` (`.ts:94-105`): crear → `/aseguradoras`; editar → restaura la copia leída y vuelve a ver |
| Guardar (`actions.save`); spinner | Pie | Crear o editar | `saving()` | `save()` (`.ts:107-148`): inválido → marca todo y no envía. Crear → `POST /api/v1/seguros`, aviso y navega a `/aseguradoras`. Editar → `PUT /api/v1/seguros/:id` (el cuerpo incluye `id`), aviso, vuelve a ver y recarga |
| Eliminar (`actions.delete`), clase `sofia-btn-delete` | Pie | Solo en editar y `*appPermiso="'Eliminar'"` (= `Seguros:Eliminar`, `.html:82`) | `deleting()` | `confirmDelete()` → diálogo |

**Diálogos que abre:** `ConfirmDialogComponent` al eliminar (`.ts:150-161`), 400 px: título «Confirmar eliminación» (`aseguradoras.detail.delete-confirm-title`), mensaje «¿Estás seguro de que deseas eliminar esta aseguradora?» (`detail.delete-confirm-message`), botones «Cancelar» y «Confirmar».

**Mensajes al usuario y manejo de errores** (comprobado en el código actual: este componente **no importa ni usa `saveErrorKey`** y no define ningún mapa de códigos)

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Crear o editar correctamente | Éxito | «Aseguradora guardada correctamente» (`aseguradoras.detail.save-success`) | `.ts:120,136` |
| **Cualquier** error al guardar (409 de código duplicado, `Db.Duplicado`, validación, 404, 500, sin conexión) | Error | «Error al guardar la aseguradora» (`detail.save-error`) | `.ts:123-126,142-145` |
| Eliminación correcta | Éxito | «Aseguradora eliminada correctamente» (`detail.delete-success`); navega a `/aseguradoras` | `.ts:167-168` |
| **Cualquier** error al eliminar | Error | «Error al eliminar la aseguradora» (`detail.delete-error`) | `.ts:170-173` |
| Error al cargar | Error | «Error al cargar la aseguradora» (`detail.load-error`) | `.ts:84` |

Consecuencia: un código identificador nacional repetido (que la base rechaza con su índice único, RN-ASE-004 de 01b) se ve como el mensaje genérico, no como un «ya existe». Esto confirma lo que indicaba un documento anterior (no se usa `saveErrorKey`). Las claves `aseguradoras.detail.title-new` y `title-edit` existen sin uso.

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Abrir existente y tras editar | `AseguradoraService.getById` | `GET /api/v1/seguros/{id}` | `Seguros:Leer` (`SegurosController.cs:34`) |
| Crear | `create` | `POST /api/v1/seguros` | `Seguros:Crear` (`:19`) |
| Editar | `update` | `PUT /api/v1/seguros/{id}` | `Seguros:Actualizar` (`:42`) |
| Eliminar | `delete` | `DELETE /api/v1/seguros/{id}` | `Seguros:Eliminar` (`:51`) |

**Exportación y carga masiva:** no hay botones en esta pantalla.

**Comportamientos particulares:** mientras carga, spinner (`.html:9-13`). Tras guardar una edición se recarga el registro.

**Observaciones**
- Las longitudes (150 y 50) coinciden con la API (`AseguradoraMedica.cs:28,38`; RN-ASE-002 y RN-ASE-003). Un valor de solo espacios pasa `Validators.required` y la entidad lo rechaza con un código `AseguradoraMedica.<Propiedad>` que se vería como mensaje genérico (no ejecutado).
- `CreateAseguradoraCommandValidator` está vacío (`// TODO: Add validation rules`).
- Eliminar no valida dependencias en el backend (`DeleteAseguradoraCommandHandler.cs`, sin comprobaciones; el `NotFound` de ese handler no declara estado 404). Ver §6.
- Specs: `aseguradoras-detail.component.spec.ts` (3 casos).

### 5.3 `/aseguradoras/carga-masiva` — Carga Masiva (aseguradoras)

- **Componente:** `AseguradorasCargaMasivaComponent` (`app/pages/aseguradoras/aseguradoras-carga-masiva/aseguradoras-carga-masiva.component.ts`; plantilla en línea `<app-carga-masiva-page [config]="config()" />`). Scope `aseguradoras` (no aporta textos).
- **Para qué sirve:** cargar varias aseguradoras desde un Excel; flujo común en [19a §6.5](19a-acceso-layout-y-compartidos.md).
- **Cómo se llega:** botón «Carga Masiva» de `/aseguradoras`.
- **Permiso de ruta:** `Seguros:Leer` + `Seguros:Crear`.
- **Migas y cabecera:** título «Carga Masiva»; migas «Inicio» / «Catálogos» / «Aseguradoras» (enlace a `/aseguradoras`) / «Carga Masiva» (activa); botón volver a `/aseguradoras`.

**Configuración que pasa** (`.ts:18-33`)

| Parámetro | Valor |
|---|---|
| `downloadUrl` | `GET {api}/api/v1/seguros/plantilla` — `Seguros:Crear` (`SegurosController.cs:89`); la API la nombra `plantilla-seguros.xlsx` (`:94`), pero el SPA la guarda como `plantilla-aseguradoras.xlsx` (el nombre sale de `backRoute`) |
| `previewUrl` | `POST {api}/api/v1/seguros/previsualizar` — `Seguros:Crear` (`:98`) |
| `saveUrl` | `POST {api}/api/v1/seguros/carga-masiva` — `Seguros:Crear` (`:114`) |
| `backRoute` | `/aseguradoras` |
| Columnas de la vista previa | `nombreComercial` → «Nombre Comercial» (`fields.nombreComercial` de la raíz, obligatoria); `codigoIdentificadorNacional` → «Código Identificador Nacional» (`fields.codigoIdentificadorNacional` de la raíz, obligatoria). Después, errores y «Estado» |

**Botones, mensajes y errores:** los del componente común. Reglas en RN-ASE-009 a RN-ASE-012 de 01b (incluye que el guardado no controla duplicados). Las claves de datos coinciden con `SeguroImportRow(NombreComercial, CodigoIdentificadorNacional)` (`CargaMasivaSegurosCommand.cs:5`).

---

## 6. Eliminar: mapa de estados y mensajes de las cinco pantallas

Contexto: el commit `27597cd` del backend añadió la comprobación de dependencias al eliminar medicamentos y proveedores (además de sucursales, pacientes, profesionales e ingredientes, fuera de esta parte) y el `01b8260` del SPA añadió los mensajes. Presentaciones de venta ya validaba ventas antes. Formulaciones clínicas y Aseguradoras no validan dependencias.

**Qué hace cada pantalla ante la respuesta del `DELETE`.** Ninguna lee el código de error (`error.code`); deciden por el estado HTTP o ni siquiera lo miran:

| Pantalla | Cómo decide | Fuente |
|---|---|---|
| Medicamentos | Si el estado es 409, mensaje «en uso»; cualquier otro error, mensaje genérico | `medicamentos-detail.component.ts:306-312` |
| Presentaciones de venta (detalle) | Igual | `presentaciones-venta-detail.component.ts:255-262` |
| Presentaciones de venta (diálogo) | Igual, con las claves `modal.*` | `presentaciones-venta-modal.component.ts:175-182` |
| Proveedores | Igual | `proveedores-detail.component.ts:236-243` |
| Formulaciones clínicas | **No distingue**: siempre el mensaje genérico | `formulaciones-clinicas-detail.component.ts:295-298` |
| Aseguradoras | **No distingue**: siempre el mensaje genérico | `aseguradoras-detail.component.ts:170-173` |

| Pantalla | `DELETE` y permiso | Códigos y estados que puede devolver la API | Mensaje con 409 (clave) | Mensaje con otro error (clave) |
|---|---|---|---|---|
| Medicamentos (§1.2) | `/api/v1/medicamentos/{id}`, `Medicamentos:Eliminar` (`MedicamentosController.cs:68`) | 404 `Medicamento.NotFound` (`DeleteMedicamentoCommandHandler.cs:17`); 409 `Medicamento.InUse` (`:23`) si hay lotes, presentaciones de venta, jerarquías de unidades, formulaciones clínicas, historial de precios de proveedor, servicios de agenda, servicios clínicos de inmunización u órdenes de producción magistral no eliminados (`:34-42`); éxito 204 | «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.» (`medicamentos.detail.delete-in-use`) | «Error al eliminar el medicamento» (`medicamentos.detail.delete-error`) |
| Presentaciones de venta (§2.2 y §2.4) | `/api/v1/presentacionesventa/{id}`, `PresentacionesVenta:Eliminar` (`PresentacionesVentaController.cs:52`) | 404 `PresentacionVenta.NotFound` (`DeletePresentacionVentaCommandHandler.cs:17`); 409 `PresentacionVenta.InUse` (`:25`) si aparece en algún detalle de venta no eliminado; éxito 204 | «No se puede eliminar la presentación porque tiene ventas registradas.» (`presentacionesVenta.detail.delete-in-use` y `presentacionesVenta.modal.delete-in-use`, mismo texto) | «Error al eliminar la presentación de venta» (`detail.delete-error` / `modal.delete-error`) |
| Formulaciones clínicas (§3.2) | `/api/v1/formulacionesclinicas/{id}`, `FormulacionesClinicas:Eliminar` (`FormulacionesClinicasController.cs:64`) | 404 `Formulacion.NotFound` (`DeleteFormulacionClinicaCommandHandler.cs:17`); **no valida dependencias** (no existe 409); éxito 204 | No hay mensaje propio | «Error al eliminar la formulación clínica» (`formulacionesClinicas.detail.delete-error`) para cualquier error |
| Proveedores (§4.2) | `/api/v1/proveedores/{id}`, `Proveedores:Eliminar` (`ProveedoresController.cs:69`) | 404 `NotFound` (`DeleteProveedorCommandHandler.cs:21`); 409 `Proveedor.InUse` (`:29`) si el proveedor tiene algún registro del historial de precios no eliminado (`:24-26`); éxito 200 con el identificador | «No puedes eliminar este proveedor porque tiene precios de compra registrados.» (`proveedores.detail.delete-in-use`) | «Error al eliminar el proveedor» (`proveedores.detail.delete-error`) |
| Aseguradoras (§5.2) | `/api/v1/seguros/{id}`, `Seguros:Eliminar` (`SegurosController.cs:51`) | `NotFound` sin estado explícito (`DeleteAseguradoraCommandHandler.cs:21`; RN-ASE-005 de 01b indica 400); **no valida dependencias**; éxito 200 con el identificador | No hay mensaje propio | «Error al eliminar la aseguradora» (`aseguradoras.detail.delete-error`) para cualquier error |

Los 404 no tienen mensaje propio en ninguna pantalla (siempre el genérico).

**Cobertura de los mensajes frente a lo que bloquea el backend**

| Pantalla | Qué bloquea el backend | Qué dice el mensaje | Observación |
|---|---|---|---|
| Medicamentos | Ocho tipos de registro (lotes, presentaciones, jerarquías, formulaciones, precios de proveedor, servicios de agenda, inmunizaciones, órdenes magistrales) | «lotes, presentaciones u otros registros asociados» | Genérico: no dice cuál de las ocho causas aplica |
| Presentaciones de venta | Ventas | «tiene ventas registradas» | Coincide |
| Formulaciones clínicas | Nada | — | Sin validación |
| Proveedores | Precios de compra | «precios de compra registrados» | Coincide; **esto implica que un proveedor al que se registró un precio nunca más se podrá eliminar**, porque los precios no se pueden borrar desde ninguna pantalla (§4.3) |
| Aseguradoras | Nada | — | Sin validación |

Cualquier 409 distinto de los citados mostraría igualmente el texto «en uso» en las cuatro pantallas que lo tratan.

---

## 7. Pendientes y «No verificado»

**«En curso (sin commitear)»:** ninguno. Se ejecutó `git status --short` en ambos repos al empezar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`): sin cambios locales. `git diff --stat 01b8260..HEAD -- src/app/pages` en el SPA no muestra cambios en ninguna carpeta de este documento.

**«No verificado»** (no se ejecutó la aplicación ni se llamó a la API):

| # | Tema | Por qué no se verificó |
|---|---|---|
| 1 | Qué ve un usuario sin permiso de lectura de los módulos auxiliares: `Laboratorios:Leer` y `UnidadesMedida:Leer` en Medicamentos; `Medicamentos:Leer` y `JerarquiasUoM:Leer` en Presentaciones de venta; `Medicamentos:Leer`, `IngredientesActivos:Leer` y `UnidadesMedida:Leer` en Formulaciones clínicas | El SPA no comprueba esos permisos y las suscripciones de los autocompletados no tienen manejador de error (excepto el diálogo de precios, que usa `catchError`); el efecto visual no se probó |
| 2 | Si elegir una sugerencia con el teclado (flechas y Enter) en los autocompletados de Medicamentos (detalle) y Formulaciones clínicas dispara el `(click)` que guarda la elección | Depende de Angular Material; no se ejecutó. Si no lo dispara, al guardar la elección se descartaría y el campo mostraría «Este campo es obligatorio» |
| 3 | Que valores formados solo por espacios lleguen a la API y muestren el mensaje genérico en Medicamentos, Proveedores y Aseguradoras | Se deduce de `Validators.required` de Angular y de las validaciones de FluentValidation/entidad; no se probó |
| 4 | Si modificar una jerarquía de unidades recalcula la equivalencia de las presentaciones de venta ya guardadas | Solo se leyeron los manejadores de crear y editar la presentación, que calculan y guardan el valor; no se revisaron los de jerarquías |
| 5 | Cómo se asigna el permiso `Proveedores:RegistrarPrecioProveedor` a un rol (pantalla de permisos del rol y catálogo de permisos) | No se revisó la pantalla de roles ni el origen del catálogo de permisos; solo el uso en `[HasPermission]` y en `*appPermiso` |
| 6 | Efecto de la zona horaria en las fechas «Vigente desde» y «Vigente hasta» de los precios | Las fechas llegan como `DateTime` UTC y se pintan con el pipe `date` del navegador; no se probó |
| 7 | Aspecto visual: iconos (`upload`, `add`, `download`, `sell`, `edit`, `delete`, `chevron_left`), deslizador de Proveedores, spinners | Solo lectura de plantillas |
| 8 | Qué ve el usuario al abrir `/medicamentos/:id` con un identificador inexistente | No se llamó a la API; por el código, «Error al cargar el medicamento» y la ficha vacía |

**Hallazgos de producto y de código** (no son errores de documentación):
- Presentaciones de venta: «Exportar» ignora el producto elegido en el autocompletado (§2.1). Borrar el texto del filtro no descarta el producto elegido (§2.1).
- Proveedores: el formulario limita el RUC a 30 y los términos financieros a 500 caracteres; la API admite 50 y 100 (§4.2). Un proveedor con precios registrados no se puede eliminar y ninguna pantalla permite borrar precios (§6). El deslizador de tasa no se restaura al volver al listado (§4.1).
- Aseguradoras: el formulario no mapea ningún código de error; un código duplicado se ve como «Error al guardar la aseguradora» (§5.2). El listado no tiene orden definido (§5.1).
- Medicamentos: la carga masiva no trae precio de venta base, pero el formulario lo exige (§1.3); el mensaje de precio aparece también si el campo está vacío (§1.2).
- Formulaciones clínicas: el endpoint `GET /formulacionesclinicas/medicamento/{productoId}` no tiene pantalla (§3.1); un valor cero en la concentración muestra «Este campo es obligatorio» (§3.2).
- Código sin uso: el método `goToNew()` de los detalles de Formulaciones clínicas, Proveedores y Aseguradoras no lo llama ninguna plantilla (`formulaciones-clinicas-detail.component.ts:302`, `proveedores-detail.component.ts:246`, `aseguradoras-detail.component.ts:298`). Los detalles de Medicamentos y Presentaciones de venta no lo definen. Además, el campo `PrecioProveedor.vigente` del modelo no lo lee ninguna plantilla (§4.3).

**Documentos hermanos que el código actual contradice** (para la pasada de reconciliación):
- 01b RN-MED-013: dice que eliminar un medicamento no valida dependencias; con `27597cd` sí lo hace (`DeleteMedicamentoCommandHandler.cs:20-42`).
- 01b RN-PRV-010: idem para proveedores (`DeleteProveedorCommandHandler.cs:23-31`).
- 01b RN-PRV-018 a RN-PRV-020: dicen que el registro de precio de proveedor no guarda nada; con `606cb69` guarda y cierra el precio anterior; además existe `GET /api/v1/Proveedores/{id}/precios` (no listado en 05b).
- 01b RN-PRV-003 y RN-PRV-004 fijan 50 y 100 caracteres; el formulario del SPA permite 30 y 500 (§4.2).
- 05b: anterior al endpoint `GET /api/v1/Proveedores/{id}/precios` (`Proveedores:Leer`).

---

## 8. Tabla de cobertura

**Rutas de `docs/_inventario/rutas-spa.csv` de este alcance (18 de 18)**

| Ruta | Componente | Ficha |
|---|---|---|
| `/medicamentos` | `MedicamentosSearchComponent` | §1.1 |
| `/medicamentos/nueva` | `MedicamentosDetailComponent` (modo crear) | §1.2 |
| `/medicamentos/carga-masiva` | `MedicamentosCargaMasivaComponent` | §1.3 |
| `/medicamentos/:id` | `MedicamentosDetailComponent` (modos ver y editar) | §1.2 |
| `/presentaciones-venta` | `PresentacionesVentaSearchComponent` | §2.1 |
| `/presentaciones-venta/nueva` | `PresentacionesVentaDetailComponent` (modo crear) | §2.2 |
| `/presentaciones-venta/:id` | `PresentacionesVentaDetailComponent` (modos ver y editar) | §2.2 |
| `/formulaciones-clinicas` | `FormulacionesClinicasSearchComponent` | §3.1 |
| `/formulaciones-clinicas/nueva` | `FormulacionesClinicasDetailComponent` (modo crear) | §3.2 |
| `/formulaciones-clinicas/:id` | `FormulacionesClinicasDetailComponent` (modos ver y editar) | §3.2 |
| `/proveedores` | `ProveedoresSearchComponent` | §4.1 |
| `/proveedores/nueva` | `ProveedoresDetailComponent` (modo crear) | §4.2 |
| `/proveedores/carga-masiva` | `ProveedoresCargaMasivaComponent` | §4.4 |
| `/proveedores/:id` | `ProveedoresDetailComponent` (modos ver y editar) | §4.2 y §4.3 |
| `/aseguradoras` | `AseguradorasSearchComponent` | §5.1 |
| `/aseguradoras/nueva` | `AseguradorasDetailComponent` (modo crear) | §5.2 |
| `/aseguradoras/carga-masiva` | `AseguradorasCargaMasivaComponent` | §5.3 |
| `/aseguradoras/:id` | `AseguradorasDetailComponent` (modos ver y editar) | §5.2 |

Presentaciones de venta y Formulaciones clínicas no tienen `/carga-masiva` (comprobado en sus `*.routes.ts` y en `rutas-spa.csv`).

**Componentes de `docs/_inventario/componentes-pages.txt` de este alcance (15 de 15)**

| Componente (carpeta bajo `app/pages`) | Ficha o motivo |
|---|---|
| `medicamentos/medicamentos-search`, `-detail`, `-carga-masiva` | §1.1, §1.2, §1.3 |
| `presentaciones-venta/presentaciones-venta-search`, `-detail` | §2.1, §2.2 |
| `presentaciones-venta/presentaciones-venta-modal` | §2.4 (diálogo sin ruta; lo abre el detalle de Medicamentos) |
| `formulaciones-clinicas/formulaciones-clinicas-search`, `-detail` | §3.1, §3.2 |
| `proveedores/proveedores-search`, `-detail`, `-carga-masiva` | §4.1, §4.2 (y §4.3), §4.4 |
| `proveedores/precio-proveedor-dialog` | §4.5 |
| `aseguradoras/aseguradoras-search`, `-detail`, `-carga-masiva` | §5.1, §5.2, §5.3 |

**Diálogos de `docs/_inventario/dialogos.txt` de este alcance (1 de 1):** `proveedores/precio-proveedor-dialog` → §4.5. `PresentacionesVentaModalComponent` no figura en `dialogos.txt` (su nombre no termina en «dialog») aunque funciona como diálogo; documentado en §2.4. Además, todos los detalles abren `ConfirmDialogComponent` ([19a §6.1](19a-acceso-layout-y-compartidos.md)) al eliminar (§1.2, §2.2, §2.4, §3.2, §4.2, §5.2).

**Usos de `*appPermiso` de `docs/_inventario/app-permiso-usos.csv` de este alcance (26 de 26)**

| Archivo:línea (bajo `app/pages/…`) | Expresión | Permiso resultante | Botón y ficha |
|---|---|---|---|
| `medicamentos/medicamentos-search/…search.component.html:19` | `'Crear'` | `Medicamentos:Crear` | «Carga Masiva», §1.1 |
| `medicamentos/medicamentos-search/…search.component.html:27` | `'Crear'` | `Medicamentos:Crear` | «Nuevo», §1.1 |
| `medicamentos/medicamentos-detail/…detail.component.html:26` | `'JerarquiasUoM:Leer'` | `JerarquiasUoM:Leer` | «Presentaciones de Venta» (contenedor), §1.2 |
| `medicamentos/medicamentos-detail/…detail.component.html:28` | `'PresentacionesVenta:Leer'` | `PresentacionesVenta:Leer` | «Presentaciones de Venta», §1.2 |
| `medicamentos/medicamentos-detail/…detail.component.html:37` | `'Actualizar'` | `Medicamentos:Actualizar` | «Editar», §1.2 |
| `medicamentos/medicamentos-detail/…detail.component.html:188` | `'Eliminar'` | `Medicamentos:Eliminar` | «Eliminar», §1.2 |
| `presentaciones-venta/presentaciones-venta-search/…search.component.html:24` | `'Crear'` | `PresentacionesVenta:Crear` | «Nuevo», §2.1 |
| `presentaciones-venta/presentaciones-venta-detail/…detail.component.html:32` | `'Actualizar'` | `PresentacionesVenta:Actualizar` | «Editar», §2.2 |
| `presentaciones-venta/presentaciones-venta-detail/…detail.component.html:88` | `'JerarquiasUoM:Crear'` | `JerarquiasUoM:Crear` | «Ir a Jerarquías de Unidades», §2.2 |
| `presentaciones-venta/presentaciones-venta-detail/…detail.component.html:167` | `'Eliminar'` | `PresentacionesVenta:Eliminar` | «Eliminar», §2.2 |
| `presentaciones-venta/presentaciones-venta-modal/…modal.component.html:22` | `'JerarquiasUoM:Crear'` | `JerarquiasUoM:Crear` | «Ir a Jerarquías de Unidades», §2.4 |
| `presentaciones-venta/presentaciones-venta-modal/…modal.component.html:64` | `'PresentacionesVenta:Actualizar'` | `PresentacionesVenta:Actualizar` | Botón editar de la fila, §2.4 |
| `presentaciones-venta/presentaciones-venta-modal/…modal.component.html:73` | `'PresentacionesVenta:Eliminar'` | `PresentacionesVenta:Eliminar` | Botón eliminar de la fila, §2.4 |
| `presentaciones-venta/presentaciones-venta-modal/…modal.component.html:139` | `'PresentacionesVenta:Crear'` | `PresentacionesVenta:Crear` | «Agregar presentación», §2.4 |
| `formulaciones-clinicas/formulaciones-clinicas-search/…search.component.html:24` | `'Crear'` | `FormulacionesClinicas:Crear` | «Nuevo», §3.1 |
| `formulaciones-clinicas/formulaciones-clinicas-detail/…detail.component.html:32` | `'Actualizar'` | `FormulacionesClinicas:Actualizar` | «Editar», §3.2 |
| `formulaciones-clinicas/formulaciones-clinicas-detail/…detail.component.html:176` | `'Eliminar'` | `FormulacionesClinicas:Eliminar` | «Eliminar», §3.2 |
| `proveedores/proveedores-search/…search.component.html:19` | `'Crear'` | `Proveedores:Crear` | «Carga Masiva», §4.1 |
| `proveedores/proveedores-search/…search.component.html:27` | `'Crear'` | `Proveedores:Crear` | «Nuevo», §4.1 |
| `proveedores/proveedores-detail/…detail.component.html:27` | `'Actualizar'` | `Proveedores:Actualizar` | «Editar», §4.2 |
| `proveedores/proveedores-detail/…detail.component.html:110` | `'Eliminar'` | `Proveedores:Eliminar` | «Eliminar», §4.2 |
| `proveedores/proveedores-detail/…detail.component.html:134` | `'RegistrarPrecioProveedor'` | `Proveedores:RegistrarPrecioProveedor` | «Registrar precio», §4.3 |
| `aseguradoras/aseguradoras-search/…search.component.html:19` | `'Crear'` | `Seguros:Crear` | «Carga Masiva», §5.1 |
| `aseguradoras/aseguradoras-search/…search.component.html:27` | `'Crear'` | `Seguros:Crear` | «Nuevo», §5.1 |
| `aseguradoras/aseguradoras-detail/…detail.component.html:27` | `'Actualizar'` | `Seguros:Actualizar` | «Editar», §5.2 |
| `aseguradoras/aseguradoras-detail/…detail.component.html:82` | `'Eliminar'` | `Seguros:Eliminar` | «Eliminar», §5.2 |

Los componentes de estas carpetas no comprueban permisos en el `.ts` (no hay llamadas a `AuthService.hasPermission` ni a `grantsPermission`; los permisos de módulos ajenos que necesitan se resuelven solo por la respuesta del servidor).

**Endpoints de `docs/_inventario/endpoints.csv` de este alcance (43) y quién los llama**

| Controlador | Endpoints en el CSV | Llamados por pantallas de esta parte | No llamados desde esta parte |
|---|---|---|---|
| `Medicamentos` | 10 | 10 (§1.1 a §1.3); además `GET` de la lista lo llaman Presentaciones, Formulaciones y el diálogo de precios | — |
| `PresentacionesVenta` | 6 | 6 (§2.1, §2.2, §2.4) | — |
| `FormulacionesClinicas` | 7 | 6 (§3.1, §3.2) | `GET /medicamento/{productoId}` (sin pantalla) |
| `Proveedores` | 11 | 11 (§4.1 a §4.5, incluidos `POST /precios` y `GET /{id}/precios`) | — |
| `Seguros` | 9 | 9 (§5.1 a §5.3) | — |

Endpoints de otros módulos que llaman estas pantallas: `GET /laboratorios`, `GET /unidadesmedida`, `GET /ingredientesactivos` (autocompletados) y `GET /jerarquiasuom/unidades-vendibles` (§2.3).

---

## 9. Claves i18n sin uso

Método: se leyeron todas las plantillas y componentes de las cinco carpetas (y de las claves de raíz que usan) y se buscó cada clave de los `es.json` de los cinco scopes entre `t('…')`, `translate('…')` y `'<scope>.<clave>'` en `src/` (sin specs), con búsqueda de texto sobre las carpetas.

| Scope | Claves sin ninguna referencia |
|---|---|
| `medicamentos` | 3: `detail.title-new`, `detail.title-edit`, `fields.stockTotal` (la tarjeta de stock usa `detail.stock-total`) |
| `presentacionesVenta` (carpeta `presentaciones-venta`) | Ninguna |
| `formulacionesClinicas` | Ninguna |
| `proveedores` | `searchPlaceholder`, `add`, `cols.razonSocial`, `cols.taxId`, `cols.terminosFinancieros`, `cols.calificacionEsg`, `cols.tasaCumplimiento`, `cols.acciones`, `actions.edit`, `actions.view`, `actions.delete`, `form.titleNew`, `form.titleEdit`, `form.titleView`, `form.razonSocial`, `form.taxId`, `form.terminosFinancieros`, `form.calificacionEsg`, `form.tasaCumplimiento`, `form.save`, `form.cancel`, `form.back`, `form.required`, `form.maxLength200`, `form.maxLength50`, `form.maxLength100`, `form.range0to100`, `deleteDialog.title`, `deleteDialog.message`, `deleteDialog.confirm`, `deleteDialog.cancel`, `notifications.created`, `notifications.updated`, `notifications.deleted`, `help.razonSocial`, `help.taxId`, `help.terminosFinancieros`, `help.calificacionEsg`, `help.tasaCumplimiento`, `import.title`, `import.downloadTemplate`, `import.selectFile`, `import.dragDrop`, `import.fileSelected`, `import.errorsFound`, `import.saveValid`, `import.noValid`, `import.saving`, `import.previewTitle`, `import.success`, `import.errors`, `import.status`, `import.valid`, `import.invalid`, `import.button`, `detail.title-new`, `detail.title-edit` (restos de pantallas anteriores; los botones usan `actions.*` de la raíz) |
| `aseguradoras` | 2: `detail.title-new`, `detail.title-edit` |

Claves usadas pero **inexistentes**: ninguna detectada en las plantillas y componentes del alcance (todas las claves `t('…')`, `transloco.translate('<scope>.…')` y del pipe `transloco` se resolvieron contra los `es.json` de su scope y de la raíz: `breadcrumbs.*`, `actions.*`, `generic-labels.*`, `general.errors.required`, `carga-masiva.*`, `fields.*`).

---

## 10. Fuentes

**SPA (`sofia-web-spa/`)**
- Enrutado y permisos: `src/app/app.routes.ts` (líneas 37, 96-122), `src/app/core/auth/route-permissions.ts`, `src/app/core/auth/permission.guard.ts`, `src/app/shared/directives/permiso.directive.ts`, `src/app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts`, `src/app/app.config.ts` (configuración de Transloco).
- Compartidos: `src/app/core/services/shared/search-state.service.ts`, `src/app/shared/utils/save-error-key.util.ts`, `src/app/shared/components/carga-masiva-page/*`, `src/app/shared/models/carga-masiva.model.ts`, `src/app/shared/components/range-slider-field/range-slider-field.component.ts`, `src/app/shared/components/confirm-dialog/*`, `src/app/shared/testing/delete-in-use-testing.ts`.
- Medicamentos: `src/app/pages/medicamentos/medicamentos.routes.ts`, `medicamentos-search/*` (con su spec), `medicamentos-detail/*` (con `medicamentos-detail.component.spec.ts` y `medicamentos-detail.delete.spec.ts`), `medicamentos-carga-masiva/*`, `models/medicamento.model.ts`, `services/medicamento.service.ts` (y su spec).
- Presentaciones de venta: `src/app/pages/presentaciones-venta/presentaciones-venta.routes.ts`, `presentaciones-venta-search/*`, `presentaciones-venta-detail/*`, `presentaciones-venta-modal/*` (con su spec), `models/presentacion-venta.model.ts`, `services/presentacion-venta.service.ts`; `src/app/pages/jerarquias/services/jerarquia-uom.service.ts` y `models/jerarquia-uom.model.ts` (unidades vendibles).
- Formulaciones clínicas: `src/app/pages/formulaciones-clinicas/formulaciones-clinicas.routes.ts`, `formulaciones-clinicas-search/*`, `formulaciones-clinicas-detail/*`, `models/formulacion-clinica.model.ts`, `services/formulacion-clinica.service.ts`; modelos y servicios de ingredientes activos y unidades de medida para los autocompletados.
- Proveedores: `src/app/pages/proveedores/proveedores.routes.ts`, `proveedores-search/*`, `proveedores-detail/*` (con `proveedores-detail.component.spec.ts` y `proveedores-detail.delete.spec.ts`), `precio-proveedor-dialog/*` (con su spec), `proveedores-carga-masiva/*`, `models/proveedor.model.ts`, `services/proveedor.service.ts`.
- Aseguradoras: `src/app/pages/aseguradoras/aseguradoras.routes.ts`, `aseguradoras-search/*` (con su spec), `aseguradoras-detail/*` (con su spec), `aseguradoras-carga-masiva/*`, `models/aseguradora.model.ts`, `services/aseguradora.service.ts` (y su spec).
- Textos: `public/i18n/es.json` (raíz: `nav`, `breadcrumbs`, `actions`, `general`, `generic-labels`, `errors`, `fields`, `carga-masiva`) y `public/i18n/{medicamentos,presentaciones-venta,formulacionesClinicas,proveedores,aseguradoras}/es.json`.

**Backend (`sofia-apis/src/`, solo lectura puntual)**
- Controladores (permisos y líneas): `SOFIA.API/Controllers/MedicamentosController.cs`, `PresentacionesVentaController.cs`, `FormulacionesClinicasController.cs`, `ProveedoresController.cs`, `SegurosController.cs`, `JerarquiasUoMController.cs` (unidades vendibles).
- Medicamentos: `SOFIA.Domain/Entities/Medicamento.cs`, `SOFIA.Domain/Enums/CondicionVenta.cs`, `SOFIA.Application/Medicamentos/Commands/{Create,Update,Delete,CargaMasiva}Medicamento*`, `Queries/GetMedicamentos/GetMedicamentosQueryHandler.cs`.
- Presentaciones de venta: `SOFIA.Domain/Entities/PresentacionVenta.cs`, `SOFIA.Application/PresentacionesVenta/*` (comandos, consulta y DTO), `SOFIA.Application/JerarquiasUoM/Queries/GetUnidadesVendiblesUoM/*`.
- Formulaciones clínicas: `SOFIA.Domain/Entities/FormulacionClinica.cs`, `SOFIA.Application/FormulacionesClinicas/*`, `SOFIA.DbUp/Scripts/0020_SoftDeleteAware_Uniques_And_Checks.sql`.
- Proveedores: `SOFIA.Domain/Entities/ProveedorDistribuidor.cs`, `HistorialPrecioProveedor.cs`, `SOFIA.Application/Proveedores/*` (incluidos `RegistrarPrecioProveedor`, `GetPreciosProveedor`, `DeleteProveedor`, `Common/ProveedorDuplicateChecker.cs`).
- Aseguradoras: `SOFIA.Domain/Entities/AseguradoraMedica.cs`, `SOFIA.Application/Seguros/*`.
- Transversal: `SOFIA.Application/Common/Behaviors/ValidationBehavior.cs`.

**Git (solo lectura):** `git log -1`, `git status --short` y `git diff --stat 01b8260..HEAD -- src/app/pages` en ambos repos; `git log --oneline` (commits `9cbbe58`, `01b8260`, `606cb69`, `27597cd`). No se ejecutó ningún comando git dentro de `docs`.

**Inventarios y documentos hermanos**
- `docs/_inventario/rutas-spa.csv`, `componentes-pages.txt`, `dialogos.txt`, `app-permiso-usos.csv`, `endpoints.csv`, `pruebas-spa.csv`.
- [19a — acceso, layout y compartidos](19a-acceso-layout-y-compartidos.md) (§4.3, §6.1, §6.5, §6.6, §7.1).
- [19c — catálogos I](19c-catalogos-1.md) (§0 y §6).
- [01b — catálogos](../01-reglas-de-negocio/01b-catalogos.md) (RN-MED, RN-PRE, RN-FOR, RN-PRV, RN-ASE).
- [05b — matriz de permisos](../05-seguridad/05b-matriz-de-permisos.md).


