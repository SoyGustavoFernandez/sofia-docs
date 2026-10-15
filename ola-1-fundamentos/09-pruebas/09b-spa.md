# Estrategia y catálogo de pruebas automatizadas — Parte B: SPA

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-15 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código (specs, `karma.conf.js`, `angular.json`, `package.json`, hooks) y de los inventarios `docs/_inventario`. No se ejecutó ninguna prueba ni compilación. `git status --short` sin cambios en ambos repos al 2026-10-15 |

Este documento describe qué pruebas automatizadas tiene el SPA (`sofia-web-spa`), cómo están hechas y qué queda sin cubrir. Los comandos y las particularidades de Windows para correrlas están en la [guía del desarrollador, sección 4.2](../08-guia-del-desarrollador.md#42-spa); aquí solo se resumen (sección 5). Para saber qué hace cada pantalla, consulta los inventarios [19a](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md), [19b](../19-inventario-de-pantallas/19b-administracion.md), [19e](../19-inventario-de-pantallas/19e-inventario-y-atencion.md) y [19f](../19-inventario-de-pantallas/19f-punto-de-venta.md).

## Contenido

1. Estrategia observable
2. Catálogo por área
3. Cobertura por pantalla
4. Huecos
5. Cómo se corren y dónde
6. Pendientes y «No verificado»
7. Tabla de cobertura
8. Fuentes

---

## 1. Estrategia observable

### 1.1 Cifras

| Dato | Valor | Fuente |
|---|---|---|
| Archivos `*.spec.ts` en `sofia-web-spa/src` | 75 (coincide con las 75 filas de `docs/_inventario/pruebas-spa.csv`; el README del inventario dice 69, quedó desactualizado) | `pruebas-spa.csv`; recuento en disco |
| Casos `it(` contados por el inventario | 492 en los 69 specs con casos | suma de la columna `casos_it` de `pruebas-spa.csv` |
| Specs con 0 `it(` literales | 6, todos `*.delete.spec.ts`: sus casos los registra la función compartida `describeDeleteInUse` (2 casos por spec, 12 en total) | `shared/testing/delete-in-use-testing.ts:34-76` |
| Casos desactivados (`xit`, `fit`, `pending`, `xdescribe`) | Ninguno | búsqueda en `src/**/*.spec.ts` |
| Specs que renderizan el DOM (`nativeElement`, `querySelector`, `debugElement`) | 24 de 75; el resto prueba la clase (señales y métodos), servicios o funciones puras | búsqueda en `src/**/*.spec.ts` |

Nota sobre el conteo: el número de casos del inventario es el de apariciones de `it(`; los casos generados dentro de un bucle o `forEach` (por ejemplo `blocks ${url} for a user who can read the module but not create in it` en `permission.guard.spec.ts:96`, o los de condición de venta controlada en `nueva-venta.component.spec.ts:400,437`) se ejecutan más de una vez. El total real de casos ejecutados **no se verificó** (no se corrieron las pruebas).

### 1.2 Herramientas y configuración

| Elemento | Valor | Fuente |
|---|---|---|
| Ejecutor | Karma `~6.4.0` con el builder `@angular-devkit/build-angular:karma` | `package.json:55`, `angular.json:116-117` |
| Marco de pruebas | Jasmine (`jasmine-core ~5.6.0`, `karma-jasmine ~5.1.0`, tipos `@types/jasmine ~5.1.0`) | `package.json:46,54,58` |
| Navegador | `Chrome` (con ventana) y `ChromeHeadlessCI` (`ChromeHeadless` con `--no-sandbox --disable-gpu`); por defecto `karma.conf.js` lista ambos | `karma.conf.js:32-38` |
| Modo por defecto | Observación: `autoWatch: true`, `singleRun: false`, `restartOnFileChange: true` | `karma.conf.js:31,39,40` |
| Informes | `progress` y `kjhtml` (`karma-jasmine-html-reporter`, con `suppressAll`) | `karma.conf.js:19-21,27` |
| Cobertura | Configurada pero no activada por defecto: `karma-coverage` con informes `html` y `text-summary` en `coverage/sofia-web-spa`; se genera solo si pasas la opción de cobertura del CLI. **No hay umbral mínimo** (no existe `check` en `coverageReporter`). La carpeta `/coverage` está en `.gitignore` | `karma.conf.js:22-26`, `.gitignore` |
| Compilación de specs | `tsconfig.spec.json` (extiende `tsconfig.json`, tipos `jasmine`, incluye `src/**/*.spec.ts` y `src/**/*.d.ts`) | `tsconfig.spec.json` |
| Configuración `shared` | `ng test --configuration shared` usa `tsconfig.spec.shared.json` y solo incluye `src/app/core/services/shared/**/*.spec.ts`; hoy esa carpeta contiene un único spec (`error-notifier.service.spec.ts`) | `angular.json:131-136`, `tsconfig.spec.shared.json` |
| Recursos servidos en las pruebas | La carpeta `public/` completa (`glob **/*`), de modo que las pruebas pueden hacer `fetch('/i18n/...')` a los JSON reales de traducción | `angular.json:121-126`; usado en `dashboard-i18n.spec.ts:8`, `roles-i18n.spec.ts:8` |
| Polyfills y estilos | `zone.js` y `zone.js/testing`; `src/styles.scss` | `angular.json:119,127` |
| Entorno | El target `test` no define `fileReplacements`: se usa `environment.ts` sin sustitución | guía 08 (línea 557), `angular.json:116-137` |
| Scripts | `npm test` = `ng test`; `npm run test:ci` = `ng test --watch=false --browsers=ChromeHeadlessCI` | `package.json:10-11` |

### 1.3 Utilidades compartidas de prueba (`src/app/shared/testing`)

| Archivo | Qué ofrece | Quién lo usa (nº de specs) |
|---|---|---|
| `transloco-testing.providers.ts` | `provideTranslocoTesting()`: `TranslocoTestingModule.forRoot` con `langs: {}`, `availableLangs ['es','en']`, `defaultLang 'es'`. No carga JSON: `t()` y el pipe devuelven **la propia clave** | 36 specs lo importan (más 3 que usan `TranslocoTestingModule` directamente) |
| `permissions-testing.providers.ts` | `providePermissions(permisos)`: reemplaza `AuthService` por un stub con `currentUser: () => null` y `hasPermission(modulo, accion='Leer')` que concede exactamente los pares `Modulo:Accion` dados | 11 |
| `router-url-stub.ts` | `stubRouterUrl(url)`: fija `Router.url` con `spyOnProperty`, para que `*appPermiso` resuelva el módulo de la pantalla | 13 |
| `activated-route-stub.ts` | `activatedRouteStub(params)`: `ActivatedRoute` mínimo con `snapshot.paramMap` | 9 |
| `delete-in-use-testing.ts` | `describeDeleteInUse(config)`: registra 2 casos de borrado de un registro en uso (ver 1.5) | 6 (los `*.delete.spec.ts`) |

### 1.4 Cómo se simulan las dependencias

| Dependencia | Técnica observada | Ejemplo |
|---|---|---|
| HTTP | `provideHttpClient()` + `provideHttpClientTesting()` y `HttpTestingController` (`expectOne`, `flush`, `verify`). Lo usan 45 specs. Los servicios se prueban contra la URL real construida con `environment.api.baseurl` | `aseguradora.service.spec.ts`, `venta-detail.component.spec.ts:17,62` |
| Servicios de la aplicación | `jasmine.createSpyObj<Servicio>` (9 specs); los componentes con mucha lógica sustituyen el servicio completo | `nueva-venta.component.spec.ts:60-77` (`VentaService`, `MedicamentoService`, `ErrorNotifierService`) |
| Autenticación y permisos | `providePermissions([...])` o un objeto literal `{ hasPermission, userId, ... }` como `AuthService`. `AuthService` real solo se prueba en su propio spec | `permiso.directive.spec.ts`, `nueva-venta.component.spec.ts:87-96` |
| Router | `provideRouter([])` (33 specs) con `spyOn(router, 'navigate')`; no se usa `RouterTestingHarness` (0 specs) | `auth.service.spec.ts:31-32` |
| Diálogos | `spyOn(MatDialog, 'open')` devolviendo `{ afterClosed: () => of(valor) }`; en `nueva-venta` se inyecta un `MatDialog` falso por `TestBed.overrideComponent(..., { add: { providers } })` | `delete-in-use-testing.ts:60`, `nueva-venta.component.spec.ts:72-73,103` |
| Notificaciones | `ErrorNotifierService` falso con `showError`/`showSuccess` espiados, y se comprueba la **clave i18n** enviada (no el texto) | `delete-in-use-testing.ts:44,66-71` |
| Almacenamiento del navegador | `localStorage` real, limpiado en `afterEach` (`clearUserStorage()`, `localStorage.removeItem`) | `auth.service.spec.ts:35-41`, `nueva-venta.component.spec.ts:108` |
| Tiempo | `fakeAsync` + `tick` (2 specs: `nueva-venta`, `precio-proveedor-dialog`); `jasmine.clock` no se usa | `nueva-venta.component.spec.ts:111-121` |
| Animaciones | `provideNoopAnimations()` (35 specs) | — |
| Impresión | `ReceiptPrintService` se prueba contra un `iframe` real oculto y se verifica que se elimine | `receipt-print.service.spec.ts` |
| Traducciones reales | `fetch('/i18n/<scope>/<lang>.json')` contra el servidor de Karma, para comparar claves es/en | `dashboard-i18n.spec.ts`, `roles-i18n.spec.ts`, `venta-detail.component.spec.ts:170` |
| Rutas reales | Se importa `appRoutes` o los `*_ROUTES` y se inspeccionan (sin navegar) | `route-permissions.spec.ts:1`, `menu-preview.spec.ts:5`, `recetas-medicas.routes.spec.ts` |

### 1.5 Convenciones de nombres y organización

- **Ubicación:** el spec vive junto al archivo que prueba (`x.component.ts` + `x.component.spec.ts`; `x.service.ts` + `x.service.spec.ts`). Fuente: estructura de `src/app`.
- **Títulos:** en inglés, en tercera persona y describiendo el comportamiento («hides Devolver without the ProcesarDevolucion permission»). Los `describe` de componentes suelen llevar el tema tras un guion largo: `'<Componente> — /nueva (create mode)'`, `'<Componente> — /:id (view mode)'`, `'<Componente> — action buttons'`, `'<Componente> — delete permission'`. Esto reparte un mismo componente en varios `describe` por modo o por tema.
- **Specs separados por comportamiento:** `*.delete.spec.ts` para el borrado de un registro en uso (ingredientes activos, medicamentos, pacientes, profesionales de salud, proveedores, sucursales). Cada uno es solo una llamada a `describeDeleteInUse({ component, apiPath, route, scope, record, answerExtraRequests? })`, que verifica: **409** → `showError('<scope>.detail.delete-in-use')`; **otro fallo (500)** → `showError('<scope>.detail.delete-error')`. Lo que prueba cada spec es, por tanto, idéntico; solo cambian el componente y el registro de ejemplo (`delete-in-use-testing.ts:34-76`).
- **Specs de contratos entre archivos:** `*-i18n.spec.ts` (paridad es/en y cobertura de claves), `*-links.spec.ts` (enlaces del dashboard), `recetas-medicas.routes.spec.ts` (rutas), `route-permissions.spec.ts` (tabla de permisos por ruta). Son pruebas «de consistencia» que fallan si alguien añade una pantalla, ruta o módulo sin completar el resto.
- **Qué se ejercita en componentes:** la mayoría llama a métodos y lee señales de la clase (`component.save()`, `component.puedeDevolver()`); el DOM se consulta solo en 24 specs (sobre todo `nueva-venta`, `dashboard`, `cuentas-detail`, `proveedores-detail`, `roles-detail`, `venta-detail`).
- **Qué no existe:** pruebas de extremo a extremo (no hay Cypress, Playwright ni Protractor en `package.json`), pruebas de accesibilidad automatizadas, pruebas visuales (no hay `toMatchSnapshot` ni capturas), arneses de componentes de Material (`HarnessLoader` 0 usos) ni umbrales de cobertura.

---

## 2. Catálogo por área

Los casos se agrupan por tema; el número entre paréntesis en cada spec es el de `it(` literales (`pruebas-spa.csv`). Las rutas de archivo son relativas a `sofia-web-spa/src/app`.

### 2.1 Raíz de la aplicación

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `app.component.spec.ts` | `AppComponent` | 2 | Se crea; su propiedad `title` es `'SOFIA'`. No provee HTTP, router ni Transloco: es una prueba mínima |

### 2.2 Core: autenticación, guards e interceptor

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `core/auth/auth.service.spec.ts` | `AuthService` (servicio real con `HttpTestingController`; el `Router.navigate` se espía) | 30 | **Estado del token:** arranca sin sesión; carga un token preexistente de `localStorage`; un `exp` vencido cuenta como no autenticado; sin `exp` cuenta como autenticado; un token mal formado da payload nulo sin lanzar. **Claims:** `empresaId`, `sucursalId`, `userId` (claim `sub`), `requiresPasswordChange()` según el claim `pwd_change`. **Login/logout:** `login()` guarda el token y actualiza la señal; `logout()` limpia la sesión local, llama al servidor «mejor esfuerzo» y navega a login, incluso si el servidor falla; con token vencido envía igualmente la cookie de refresco y navega solo cuando el servidor responde; `clearLocalSession()` quita el token sin llamar al servidor ni navegar. **Renovación:** `refreshAccessToken()` unifica llamadas concurrentes en una sola petición y permite otra cuando la anterior terminó; `refreshAccessToken()` y `logout()` envían la cabecera anti-CSRF. **Permisos:** `loadPermissions()` consulta `/auth/me` una vez y reutiliza el resultado; `hasPermission()` compara módulo y acción sin distinguir mayúsculas y usa `Leer` por defecto; el rol `Admin` pasa por todos los módulos; todo se deniega hasta cargar permisos; `clearLocalSession()` olvida los permisos; `storeToken()` de otro usuario descarta los permisos del anterior; `loadPermissions()` descarta una respuesta que llega tras cerrar sesión. **Datos locales por usuario:** `login()` borra los borradores del POS de otros usuarios y conserva los del propio; `logout()` borra borradores y cachés pero conserva preferencias del dispositivo; `clearLocalSession()` conserva el borrador del POS (el cajero con sesión vencida puede retomar la venta) y vacía los filtros de búsqueda en memoria. **Contraseña:** `changePassword()` envía ambas contraseñas con la cookie de refresco; `forgotPassword()` envía solo el usuario, sin cookies ni cabecera CSRF; `resetPassword()` envía usuario, token y nueva contraseña sin cookies |
| `core/auth/auth.guard.spec.ts` | `authGuard` | 3 | Deja pasar de inmediato con token local válido; con token vencido intenta un refresco silencioso y deja pasar si funciona; si el refresco falla, limpia la sesión y redirige a `/auth/login` |
| `core/auth/password-change.guard.spec.ts` | `passwordChangeGuard` | 3 | Sin cambio pendiente deja pasar cualquier ruta; con cambio pendiente redirige las demás rutas a la pantalla de cambio de contraseña; la deja pasar a ella |
| `core/auth/permission.guard.spec.ts` | `permissionGuard` | 12 | Rutas sin módulo mapeado (dashboard) pasan sin cargar permisos; una ruta anidada pasa si el usuario puede leer el módulo; sin permiso redirige al dashboard con un aviso; el escáner de recetas exige `Analizar` además de leer (bloquea con solo lectura, abre con lectura y análisis) y las demás rutas de recetas solo exigen lectura; las rutas de creación (`nueva`, `carga-masiva`) exigen `Crear` (caso generado por URL) y se abren con lectura y creación; búsqueda y detalle solo piden lectura; `sesiones-caja/nueva` pide `AperturarCaja`, no `Crear`; `stock-por-sucursal/nueva` pide `Inventarios:Actualizar`, no `Crear`; si no se pueden cargar los permisos, falla cerrado e informa al usuario |
| `core/auth/route-permissions.spec.ts` | `ROUTE_PERMISSIONS`, `ROUTE_ACTIONS`, `routeModule`, `routeAction`, `grantsPermission` (puras) | 11 | Toda ruta hija del shell salvo `dashboard` y `cambiar-clave` está mapeada (una pantalla nueva sin mapeo hace fallar la prueba); `routeModule()` usa el primer segmento e ignora ids, `?query` y parámetros matriciales (`/roles;x=1`); `routeAction()` pide `Analizar` en el escáner, `Crear` en `nueva`/`nuevo`/`carga-masiva` de toda ruta mapeada (con o sin decoración), la acción real donde no es `Crear` (`AperturarCaja`, `Actualizar`) y nada en el resto; las rutas con acción pertenecen a un módulo mapeado; `grantsPermission()` exige la acción exacta, usa `Leer` por defecto, ignora mayúsculas y espacios, y deja pasar al rol `Admin` |
| `core/auth/user-storage.spec.ts` | `userStorageKey`, `clearUserStorage`, `clearOtherUsersStorage` | 3 | La clave se separa por id de usuario y cae a «anonymous»; `clearUserStorage()` borra claves por usuario, por venta y heredadas sin usuario; `clearOtherUsersStorage()` conserva las del usuario dado y borra las de otros y las heredadas |
| `core/interceptors/auth.interceptor.spec.ts` | `authInterceptor`, `isApiRequest` | 14 | Añade `Authorization: Bearer` cuando hay token y no la añade sin token; **401** en endpoint normal: refresca y reintenta; si el refresco falla, cierra sesión y propaga el error; errores no 401 se propagan sin tocar la sesión; **403** que exige cambio de contraseña: lleva a la pantalla de cambio y propaga el error, otros 403 no se redirigen; 401 de `/auth/login` se propaga sin limpiar la sesión; 401 de `/auth/refresh` limpia la sesión local en silencio y sin reintentar; nunca envía el token a recursos estáticos (traducciones) ni a otros orígenes; no intenta refrescar ante 401 de peticiones que no son de la API; `isApiRequest` acepta rutas de la API con URL base vacía y solo el origen configurado con URL base absoluta |
| `core/services/shared/error-notifier.service.spec.ts` | `ErrorNotifierService` | 7 | `showError()` y `showSuccess()` traducen la etiqueta de cierre (no «Cerrar» fijo); `showSmartErrors()` une los mensajes del backend, cae al mensaje traducido de error desconocido, o a un `fallback` explícito si se da; `showServerError()` abre el snackbar enriquecido si hay `traceId` y, si no, un error simple traducido |

---

### 2.3 Shared: componentes, directivas, fechas y utilidades

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `shared/components/carga-masiva-page/carga-masiva-page.component.spec.ts` | `CargaMasivaPageComponent` (usado por todas las cargas masivas) | 6 | `downloadTemplate()` descarga la plantilla por `HttpClient` como blob y nunca abre una ventana; muestra error si la descarga falla; rechaza en el cliente un archivo que no es `.xlsx` y uno que supera el límite de tamaño, sin llamar a la API; traduce el código de rechazo de la vista previa; `cargaMasivaErrorKey()` mapea errores de tamaño, archivo y validador de fila e ignora los desconocidos |
| `shared/components/datepicker-internationalization/mat-datepicker-intl.component.spec.ts` | `MatDatepickerIntlTransloco` | 2 | Usa las etiquetas del idioma por defecto; cambia de etiquetas y notifica al cambiar de idioma |
| `shared/date/sofia-date-adapter.spec.ts` | `SofiaDateAdapter` | 11 | Lee `d/M/yyyy` en español y `M/d/yyyy` en inglés; rechaza mes-primero en español y día-primero en inglés; lee ISO `yyyy-MM-dd` como día local en cualquier idioma; es estricto con fechas imposibles o mal formadas; devuelve `null` con entrada vacía; deserializa fecha sin hora como día local; da formato según el idioma activo y es reversible en ambos; sigue el idioma activo de Transloco; asigna idioma de la app a configuración regional de fechas |
| `shared/directives/permiso.directive.spec.ts` | `PermisoDirective` (`*appPermiso`) | 10 | Una acción sola toma el módulo de la ruta actual; oculta la acción si el usuario la tiene en un módulo distinto al de la ruta; `Modulo:Accion` explícito se comprueba contra ese módulo sin importar la ruta; con solo lectura oculta toda acción; dibuja la plantilla una sola vez dentro de `ng-container` en un `@if`; `Admin` ve todo; falla cerrado en una ruta sin módulo mapeado incluso para `Admin`; permanece oculta hasta que carga el perfil y aparece al cargar; se oculta de nuevo al terminar la sesión; sigue los permisos del nuevo usuario tras un cambio de usuario |
| `shared/utils/date-only.util.spec.ts` | `toDateOnly`, `fromDateOnly`, `toLocalMidnightIso`, `toLocalDayStartIso`, `toLocalDayEndIso`, `buildLocalDate` | 10 | Serialización de fechas como día local aunque sea tarde en el día; `undefined`/`null` con entradas vacías o inválidas; conserva el día escrito de un valor con hora y desfase; ida y vuelta sin cambio; medianoche local con el desfase explícito de ese día; inicio y fin del día local como instantes; `buildLocalDate` rechaza componentes desbordados |
| `shared/utils/save-error-key.util.spec.ts` | `saveErrorKey` | 3 | Prefiere el mensaje propio de la pantalla para un código de negocio conocido; recurre a los mensajes genéricos de base de datos para códigos de restricción; usa el mensaje de respaldo de la pantalla ante códigos desconocidos o errores que no son HTTP |
| `shared/utils/tipo-documento.util.spec.ts` | utilidades de tipo de documento | 3 | Valida cada número contra el formato de su tipo; marca el control del número cuando no coincide con el tipo elegido; mapea códigos conocidos a claves i18n de la raíz e ignora los desconocidos |

### 2.4 Layout (`matdash`)

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `matdash/layouts/full/vertical/header/header.component.spec.ts` | `HeaderComponent` vertical | 4 | Muestra el disparador del perfil con nombre accesible en cualquier ancho; muestra quién tiene sesión y ofrece cambiar contraseña y cerrar sesión; cierra sesión mediante `AuthService`; navega a la pantalla de cambio de contraseña desde el menú de perfil |
| `matdash/layouts/full/vertical/sidebar/visible-nav-items.spec.ts` | `visibleNavItems` | 3 | Oculta rutas no permitidas y los grupos que quedan solo con su encabezado; conserva todo si puede abrir todas las rutas; no modifica los datos estáticos de navegación |

---

### 2.5 Pages: acceso, cuenta propia, dashboard y empresa

Pantallas descritas en [19a](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) (acceso, cambio de clave, dashboard) y [19b](../19-inventario-de-pantallas/19b-administracion.md) (empresa).

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `pages/auth/login/login.component.spec.ts` | `LoginComponent`, `loginErrorKey` | 7 | No llama a la API con el formulario incompleto; 401 culpa a las credenciales; si la API no responde o falla con error de servidor, informa servicio no disponible (no culpa a las credenciales); conserva el mensaje específico de un rechazo que la API explica; 429 muestra el mensaje de límite de intentos; `loginErrorKey` trata como servicio no disponible un fallo que no es respuesta HTTP (tiempo agotado, error de script) |
| `pages/auth/forgot-password/forgot-password.component.spec.ts` | `ForgotPasswordComponent` | 4 | No llama sin usuario; envía el usuario recortado y muestra la confirmación genérica; 429 muestra el mensaje de límite y conserva el formulario; otro fallo muestra un error genérico |
| `pages/auth/reset-password/reset-password.component.spec.ts` | `ResetPasswordComponent` | 5 | Lee token y usuario del fragmento de la URL y los quita de la barra de direcciones; un enlace sin token o usuario es inválido y oculta el formulario; aplica las reglas de cambio de contraseña (largo mínimo y confirmación); envía usuario, token y nueva contraseña, descarta cualquier sesión local y va a login; muestra el mensaje de enlace vencido si la API rechaza el token |
| `pages/cambiar-clave/cambiar-clave.component.spec.ts` | `CambiarClaveComponent` | 5 | No llama con el formulario inválido; rechaza clave nueva más corta que el mínimo; rechaza clave igual a la actual y confirmación distinta; envía el cambio, descarta la sesión local y va a login; si la clave actual es incorrecta deja al usuario en la pantalla |
| `pages/dashboard/dashboard.component.spec.ts` | `DashboardComponent` | 15 | Pide solo las secciones que el cajero puede leer, enviando el inicio del día local; sin ningún módulo legible no hace peticiones y muestra solo el encabezado; la tarjeta de cuarentena se pide y muestra solo con `DIGEMID:Leer`; en la vista multisucursal muestra la tabla por sucursal con el título de empresa y oculta los enlaces de la propia sucursal; el título cambia cuando el alcance no es toda la empresa; en vista de una sucursal oculta la tabla y ofrece los enlaces de ventas; siete barras relativas al mejor día con hoy resaltado; nunca pinta `NaN` si toda la tendencia es cero; porcentaje contra ayer solo si ayer vendió; una sección que falla no afecta a las otras y se reintenta sola; refrescar recarga todas las legibles; lista sesiones abiertas solo si el alcance incluye más que las propias; ofrece abrir caja solo con `POS:AperturarCaja`; al pulsar un enlace de ventas abre la lista ya filtrada |
| `pages/dashboard/dashboard-links.spec.ts` | `dashboard-links.ts` | 6 | Enlaces del dashboard a otras búsquedas: ventas de hoy solo cobradas; mis ventas de hoy filtradas por mi empleado; todas las ventas en espera sin importar el día; sesiones de caja abiertas; lotes con stock cuyo vencimiento es anterior a hoy; lotes por vencer en cada tramo disjunto con ambos días incluidos |
| `pages/dashboard/dashboard-i18n.spec.ts` | `public/i18n/dashboard/{es,en}.json` | 2 | Mismas claves en español e inglés; todo método de pago de `MetodoPago` tiene etiqueta en ambos idiomas |
| `pages/dashboard/services/dashboard.service.spec.ts` | `DashboardService` | 4 | `ventas()`, `inventario()` y `devoluciones()` envían `hoy` (inicio del día local); `caja()` y `cuarentena()` no necesitan día |
| `pages/empresa/empresa-register/empresa-register.component.spec.ts` | `EmpresaRegisterComponent` (`/auth/register`) | 8 | Exige RUC, nombres y correo del administrador pero no el apellido materno; rechaza correo inválido y RUC que no tiene 11 dígitos; envía los datos recortados con apellido materno `null` si queda vacío; es inválido sin la dirección de la sede principal y válido al completarla; rechaza dirección mayor a 255 caracteres; mensaje específico si el backend rechaza la dirección; mensaje específico si el RUC ya está registrado; error genérico ante fallos desconocidos |

---

### 2.6 Pages: administración (empleados, roles, cuentas, series fiscales, sucursales)

Pantallas descritas en [19b](../19-inventario-de-pantallas/19b-administracion.md).

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `pages/empleados/empleados-detail/empleados-detail.component.spec.ts` | `EmpleadosDetailComponent` | 5 | Segundo apellido opcional: nombres y primer apellido obligatorios, segundo no; al crear envía `null` si queda vacío; lo envía recortado si se da; en modo vista lo muestra vacío si no existe; al editar puede borrarse enviando `null` |
| `pages/empleados/empleados-search/empleados-search.component.spec.ts` | `EmpleadosSearchComponent` | 2 | Pinta el nombre sin hueco y la celda del segundo apellido vacía si no existe; envía el filtro de segundo apellido solo si tiene valor |
| `pages/empleados/services/empleado.service.spec.ts` | `EmpleadoService` | 3 | `search()` envía todos los filtros dados; `getById/create/update/delete` golpean los endpoints esperados; `exportar()` pide un blob con los filtros |
| `pages/roles/roles-search/roles-search.component.spec.ts` | `RolesSearchComponent` | 2 | Crea con resultados ocultos y sin HTTP hasta `search()`; `search()` envía los límites de `nivelJerarquia` solo si difieren de los valores por defecto del deslizador |
| `pages/roles/roles-detail/roles-detail.component.spec.ts` | `RolesDetailComponent` | 16 | **Nueva:** abre en edición sin cargar datos. **Vista:** carga el rol, sus permisos y sus sucursales en paralelo. **Etiquetas de permisos:** traduce módulo y acción y recurre al nombre crudo si la API añade uno nuevo; ordena por la etiqueta que lee el usuario; dibuja etiquetas traducidas (nunca una clave); activa/desactiva por los nombres crudos del backend. **Gestión:** oculta Editar a quien solo lee; ofrece Editar con `Seguridad:Actualizar` y Eliminar solo con `Seguridad:Eliminar`; permisos y sucursales quedan de solo lectura al editar sin `Seguridad:GestionarPermisos` y se habilitan con él; `save()` actualiza solo los datos del rol sin ese permiso y envía además sucursales y cambios de permisos con él. **Vista previa del menú:** marca visible una pantalla solo si el rol puede leer su módulo, como el menú lateral; se actualiza al alternar permisos antes de guardar; el rol `Admin` ve todo; usa los mismos nombres traducidos de grupos y pantallas que el menú |
| `pages/roles/models/menu-preview.spec.ts` | `buildMenuPreview` + `navItems`, `appRoutes`, `ROUTE_PERMISSIONS` | 5 | Conserva todas las pantallas de todos los grupos y marca las que el rol no puede abrir; sobre el menú real: todas las rutas listadas existen en las rutas del shell; todas (salvo `/dashboard`) tienen módulo de permiso; grupos y pantallas llevan las claves `breadcrumbs.*` y `nav.*` que usa el menú; lo visible en la vista previa coincide exactamente con lo que lista el menú para los mismos permisos |
| `pages/roles/roles-i18n.spec.ts` | `public/i18n/roles/{es,en}.json` | 2 | Todo módulo detrás de una pantalla tiene etiqueta en ambos idiomas; módulos y acciones iguales en español e inglés |
| `pages/roles/services/rol.service.spec.ts` | `RolService` | 6 | Filtros de `nivelJerarquia` solo si no son nulos; `getById/create/update/delete`; `exportar()`; catálogo de permisos; `getPermisos/assignPermiso/revokePermiso`; `getSucursales/setSucursales` |
| `pages/seguridad/cuentas-detail/cuentas-detail.component.spec.ts` | `CuentasDetailComponent` | 18 | **Roles:** vista sin error y texto de «sin rol» si la cuenta no tiene; muestra el nombre del rol; el autocompletado de rol recibe texto inicial vacío si no hay rol; `save()` sin rol actual solo asigna el nuevo, con rol distinto quita el actual y asigna el nuevo, con el mismo rol no hace nada. **Botones:** oculta Editar a quien solo lee; Editar con `Seguridad:Actualizar` y Eliminar solo con `Seguridad:Eliminar`; el rol queda de solo lectura al editar sin `Seguridad:AsignarRoles` y se ofrece el selector con él. **Bloqueo:** «No» si no está bloqueada o el bloqueo venció, «Sí» mientras rige. **Sucursales:** lista las asignadas; asigna la elegida y recarga la cuenta; ignora una selección vaciada; quita una sucursal y recarga; excluye del autocompletado las ya asignadas |
| `pages/series-fiscales/series-fiscales-detail/series-fiscales-detail.component.spec.ts` | `SeriesFiscalesDetailComponent` | 4 | Nueva: abre en edición y rechaza un prefijo que no corresponde al tipo de documento; `save()` envía (POST) una serie válida. Detalle: carga en solo lectura y solo desbloquea el estado cuando ya se emitieron documentos; desbloquea todos los campos mientras la serie no tiene documentos |
| `pages/series-fiscales/series-fiscales-search/series-fiscales-search.component.spec.ts` | `SeriesFiscalesSearchComponent` | 3 | Resultados ocultos al inicio; `search()` envía los filtros (prefijo en mayúsculas) y muestra resultados; `clear()` limpia y oculta |
| `pages/series-fiscales/services/serie-fiscal.service.spec.ts` | `SerieFiscalService`, `esPrefijoValido` | 6 | Filtros solo si se dan; envía tipo Factura aunque su valor de enum parezca falso; `create()` devuelve el id; `update()` y `delete()`; `exportar()` envía etiquetas traducidas y filtros como blob; `esPrefijoValido` aplica la regla de letras SUNAT por tipo de documento |
| `pages/sucursales/services/sucursal.service.spec.ts` | `SucursalService` | 1 | `getPermitidas()` pide las sucursales que el usuario puede operar |
| `pages/sucursales/sucursales-detail/sucursales-detail.delete.spec.ts` | `SucursalesDetailComponent` vía `describeDeleteInUse` | 0 (2 efectivos) | 409 → `sucursales.detail.delete-in-use`; otro fallo → `sucursales.detail.delete-error` |

---

### 2.7 Pages: catálogos (aseguradoras, laboratorios, medicamentos, ingredientes activos, presentaciones de venta, proveedores)

Pantallas descritas en [19c](../19-inventario-de-pantallas/19c-catalogos-1.md) y [19d](../19-inventario-de-pantallas/19d-catalogos-2.md).

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `pages/aseguradoras/aseguradoras-detail/aseguradoras-detail.component.spec.ts` | `AseguradorasDetailComponent` | 3 | `/nueva`: abre en edición con el formulario vacío y habilitado y sin HTTP; `save()` no llama a la API con campos obligatorios vacíos. `/:id`: carga la aseguradora y deshabilita el formulario |
| `pages/aseguradoras/aseguradoras-search/aseguradoras-search.component.spec.ts` | `AseguradorasSearchComponent` | 3 | Crea con resultados ocultos y sin HTTP; `search()` llena y muestra la tabla; `clear()` oculta y limpia filtros |
| `pages/aseguradoras/services/aseguradora.service.spec.ts` | `AseguradoraService` | 7 | `search()` sin parámetros de filtro vacíos y con ellos; `getById`; `create` (POST); `update` (PUT); `delete`; `exportar()` como blob con filtros |
| `pages/laboratorios/laboratorios-detail/laboratorios-detail.component.spec.ts` | `LaboratoriosDetailComponent` | 3 | Botones por permiso: solo lectura sin Editar; Editar con `Laboratorios:Actualizar` y Eliminar oculto sin `Laboratorios:Eliminar`; Eliminar al editar con `Laboratorios:Eliminar` |
| `pages/laboratorios/laboratorios-search/laboratorios-search.component.spec.ts` | `LaboratoriosSearchComponent` | 4 | Sin `Crear` no hay Nuevo ni Carga masiva; con `Laboratorios:Crear` sí; un `Crear` de otro módulo no cuenta; Exportar sigue disponible con solo lectura |
| `pages/medicamentos/medicamentos-detail/medicamentos-detail.component.spec.ts` | `MedicamentosDetailComponent` | 3 | `/nueva`: edición y carga del catálogo de condiciones de venta; `/:id`: carga el medicamento con referencias de laboratorio y unidad y deshabilita el formulario; las presentaciones de venta se abren solo con permiso de leer presentaciones y jerarquías de unidades |
| `pages/medicamentos/medicamentos-detail/medicamentos-detail.delete.spec.ts` | `MedicamentosDetailComponent` vía `describeDeleteInUse` | 0 (2 efectivos) | 409 → `medicamentos.detail.delete-in-use`; otro fallo → `medicamentos.detail.delete-error` (atiende antes la petición de condiciones de venta) |
| `pages/medicamentos/medicamentos-search/medicamentos-search.component.spec.ts` | `MedicamentosSearchComponent` | 2 | Carga condiciones de venta al iniciar y arranca con resultados ocultos; `search()` llena la tabla |
| `pages/medicamentos/services/medicamento.service.spec.ts` | `MedicamentoService` | 8 | Catálogo de condiciones de venta; `search()` con paginación y solo filtros dados (incluidos booleanos falsos); reenvía las banderas del POS solo si están activas; `getById`, `create`, `update`, `delete`; `exportar()` |
| `pages/ingredientes-activos/ingredientes-activos-detail/ingredientes-activos-detail.delete.spec.ts` | `IngredientesActivosDetailComponent` vía `describeDeleteInUse` | 0 (2 efectivos) | 409 → `ingredientesActivos.detail.delete-in-use`; otro fallo → `...delete-error` |
| `pages/presentaciones-venta/presentaciones-venta-modal/presentaciones-venta-modal.component.spec.ts` | `PresentacionesVentaModalComponent` | 3 | Solo lectura: lista sin acciones de escritura; cada acción de escritura exige su propio permiso de `PresentacionesVenta`; el enlace a crear jerarquía de unidades exige `JerarquiasUoM:Crear` |
| `pages/proveedores/proveedores-detail/proveedores-detail.component.spec.ts` | `ProveedoresDetailComponent` (tarjeta de precios de compra) | 9 | En `/nueva` no muestra la tarjeta ni pide precios; pide la primera página y pinta una fila por precio; estado vacío; oculta la tarjeta mientras se edita; el botón de registrar exige `RegistrarPrecioProveedor`; abre el diálogo con el id del proveedor y recarga la primera página si se registró; no recarga si se cancela; pide la página elegida en el paginador; error traducido si no se pueden cargar |
| `pages/proveedores/proveedores-detail/proveedores-detail.delete.spec.ts` | `ProveedoresDetailComponent` vía `describeDeleteInUse` | 0 (2 efectivos) | 409 → `proveedores.detail.delete-in-use`; otro fallo → `...delete-error` (atiende antes la petición de precios) |
| `pages/proveedores/precio-proveedor-dialog/precio-proveedor-dialog.component.spec.ts` | `PrecioProveedorDialogComponent`, `costoNeto` | 10 | `costoNeto` aplica el descuento y redondea a 4 decimales; rechaza precio no positivo y descuento fuera de 0-100; exige elegir el producto de la lista (no solo texto); no envía y marca todo como tocado si es inválido; muestra el costo neto al escribir; envía proveedor, producto, precio y descuento, notifica y cierra con `true`; mensaje traducido del error del backend y el diálogo sigue abierto; mensaje genérico sin mapeo; busca en el servicio de medicamentos al escribir; sin permiso de leer medicamentos deja las opciones vacías sin romper |

### 2.8 Pages: inventario, pacientes y atención (lotes, stock, pacientes, profesionales, recetas)

Pantallas descritas en [19e](../19-inventario-de-pantallas/19e-inventario-y-atencion.md).

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `pages/lotes/lotes-detail/lotes-detail.component.spec.ts` | `LotesDetailComponent` | 4 | `/nueva`: abre en edición con el formulario vacío y habilitado; `save()` no hace nada con el formulario inválido y sin producto. `/:id`: carga el lote, deshabilita el formulario y muestra datos de solo lectura; `enterEditMode()` habilita el formulario |
| `pages/lotes/lotes-search/lotes-search.component.spec.ts` | `LotesSearchComponent` | 3 | Crea con resultados vacíos y ocultos; `search()` llena y muestra; `clear()` restablece y oculta |
| `pages/lotes/services/lote.service.spec.ts` | `LoteService` | 7 | `search()` siempre envía `pageNumber` y `pageSize` y solo los filtros dados; `getById`; `create` devuelve el id; `update`; `delete`; `exportar()` hace POST con cabeceras, formato de fecha y filtros y pide un blob |
| `pages/stock-por-sucursal/services/stock.service.spec.ts` | `StockService` | 6 | `search()` por defecto `soloConStock=true`, respeta `false` explícito y rangos; `getById`; `registrar()` va al endpoint `lotesinventario/stock`, no a `stockporsucursal`; `ajustar()` hace PUT de la nueva cantidad; `exportar()` |
| `pages/pacientes/pacientes-detail/pacientes-detail.delete.spec.ts` | `PacientesDetailComponent` vía `describeDeleteInUse` | 0 (2 efectivos) | 409 → `pacientes.detail.delete-in-use`; otro fallo → `...delete-error` |
| `pages/profesionales-salud/profesionales-salud-detail/profesionales-salud-detail.delete.spec.ts` | `ProfesionalesSaludDetailComponent` vía `describeDeleteInUse` | 0 (2 efectivos) | 409 → `profesionalesSalud.detail.delete-in-use`; otro fallo → `...delete-error` |
| `pages/recetas-medicas/recetas-medicas.routes.spec.ts` | `RECETAS_MEDICAS_ROUTES` | 5 | La ruta vacía va a la búsqueda; `nueva` va al detalle en modo nuevo; `:id` al detalle sin la bandera de nuevo; el escáner vive bajo `escanear` y se declara antes de `:id`; declara toda ruta de recetas que el guard gobierna por acción (un renombrado no la deja sin guard) |
| `pages/recetas-medicas/recetas-search/recetas-search.component.spec.ts` | `RecetasSearchComponent` | 7 | La entrada al escáner se oculta a quien lee y crea pero no tiene `Recetas:Analizar`; se muestra con `Analizar` y al administrador sin permisos explícitos; resultados ocultos al inicio; `buscar()` envía paciente y fechas y muestra resultados; `limpiar()` restablece y oculta; `exportar()` envía el rango de fechas |
| `pages/recetas-medicas/recetas-detail/recetas-detail.component.spec.ts` | `RecetasDetailComponent` | 11 | `/nueva`: edición sin cargar nada; no guarda sin paciente y prescriptor; envía la fecha de emisión como `yyyy-MM-dd` con las referencias; envía el número de receta recortado; no guarda un número más largo que el permitido; mapea los códigos del backend (generado por código) al mensaje de datos inválidos; mapea `Paciente.NotFound` a su mensaje. `/:id`: carga y queda en vista; muestra el número de receta y lo conserva al editar. Eliminar: oculto a quien actualiza pero no elimina; visible al editar con `Recetas:Eliminar` |

---

### 2.9 Pages: punto de venta y caja

Pantallas descritas en [19f](../19-inventario-de-pantallas/19f-punto-de-venta.md).

| Spec | Bajo prueba | Casos | Qué verifica |
|---|---|---|---|
| `pages/pos/nueva-venta/nueva-venta.component.spec.ts` | `NuevaVentaComponent` (`/pos/nueva`), con `VentaService`, `MedicamentoService`, `ErrorNotifierService` y `MatDialog` falsos | 52 | **Consultas del POS (7):** catálogo ordenado por más vendidos con el stock vendible de la propia sucursal; un fallo de búsqueda se informa y se sigue buscando; lotes del producto elegido; error al fallar la consulta de lotes (limpia los lotes); mensaje específico ante un código conocido del backend; presentaciones del producto del lote y su error. **Pestaña de escaneo de receta (2):** oculta sin `Recetas:Analizar` y se mantiene la búsqueda de catálogo; visible con `Recetas:Analizar`. **Acciones entre módulos (2):** abrir caja desde el POS solo con `POS:AperturarCaja`; enlace a registrar receta solo con `Recetas:Crear`. **Tarjetas de producto y carrito (4):** marca como «requiere receta» solo lo que no es de venta libre; precio unitario o aviso traducido si el catálogo no lo trae; aviso en el carrito de una empresa estricta mientras un producto con receta no tiene receta elegida; sin aviso para venta libre. **Verificación de receta (23):** carga la regla de la empresa una vez al iniciar y asume empresa estricta si no puede cargarla; en empresa flexible muestra la casilla y bloquea el cobro hasta marcarla; sin casilla para venta libre o con receta elegida; ayuda en lugar de casilla para condiciones controladas (caso por condición) y en empresa estricta; avisa que la línea controlada necesita número de receta; bloquea el cobro pero no dejar en espera si falta receta (controlada y estricta); bloquea el cobro si la receta de una línea controlada no tiene número y lo permite con una numerada; deja en espera una venta bloqueada para cobro; no bloquea líneas de un borrador viejo sin condición de venta; mantiene el cobro permitido al añadir otra línea de receta simple; envía `recetaVerificada` al cobrar una venta nueva, no la declara si no hace falta ni al dejar en espera; al retomar una venta en espera conoce la condición y la envía solo al cobrarla; añade una línea escaneada con el precio del lote y su condición y no añade una línea escaneada cuyo lote no tiene precio; ofrece la casilla cuando el backend pide la confirmación que el POS no ofreció (mapea cada código del backend a su mensaje) y la retira cuando deja de ser necesaria. **Pagos (6):** el pago añadido se rellena con el saldo restante; céntimos sin residuo de coma flotante y nunca negativo; bloquea el cobro si un pago no efectivo no tiene número de operación y no lo pide al efectivo; error de referencia requerida y cobro deshabilitado con tarjeta sin referencia; mapea cada código del backend al mensaje de referencia requerida. **Pagos obsoletos (5):** al vaciar el carrito vuelve a una sola fila de efectivo; mantiene los pagos mientras queden líneas y los reinicia al quitar la última; el borrador de una venta en espera no guarda el efectivo escrito; conserva solo las filas no efectivo con número de operación; ignora el efectivo de un borrador antiguo al retomar. **Datos del comprobante (3):** entrega al comprobante el vuelto y la presentación de cada línea; sin vuelto si el pago es exacto; muestra el vuelto con el cálculo compartido (`calcularVuelto`) |
| `pages/pos/nueva-venta/pos-draft.spec.ts` | `serializePosDraft`, `parsePosDraft` | 7 | Ida y vuelta dentro de su tiempo de vida; guarda solo el id del cliente, nunca datos personales; descarta borradores vencidos y los antiguos sin marca de tiempo (que llevaban el cliente completo); `null` ante borradores ausentes o corruptos; normaliza campos mal formados a valores seguros; un borrador está vacío solo sin carrito, cliente ni aseguradora |
| `pages/pos/utils/vuelto.util.spec.ts` | `calcularVuelto`, `roundMoney` | 8 | Efectivo entregado menos lo que se paga; 0 si es exacto o si los pagos no cubren; pago dividido con efectivo mayor a lo restante; 0 con pagos no efectivo exactos; redondeo a céntimos; monto ausente cuenta 0; `roundMoney` redondea a dos decimales |
| `pages/pos/services/venta.service.spec.ts` | `VentaService` | 12 | `search()` envía `estado` aun siendo el valor falso 0 (Completada); `getById`; `configuracion()` lee las reglas de venta de la empresa; `crear` (POST); `completar` (PUT); `actualizarDetalles` (PUT); `anular()` envía el cuerpo como cadena JSON con `Content-Type` explícito; `procesarDevolucion()` hace POST con enums numéricos a devoluciones; `lotesDisponibles()` por id o por nombre; `presentacionesProducto()`; `exportar()` |
| `pages/pos/services/receipt-print.service.spec.ts` | `ReceiptPrintService` | 4 | Imprime una copia del comprobante sola en un `iframe` oculto; página de rollo de 80 mm sin márgenes; conserva todas las líneas de un ticket largo y la página crece con él; quita el `iframe` tras imprimir y nunca apila marcos |
| `pages/pos/dialogs/comprobante-dialog/comprobante-dialog.component.spec.ts` | `ComprobanteDialogComponent` | 11 | Imprime razón social y datos legales del emisor y el documento del cliente; muestra el estado por su clave i18n; sin documento para el cliente anónimo; ofrece nueva venta solo con permiso de crear ventas; envía solo el elemento del comprobante a la impresora aislada; vuelto impreso tras los pagos solo si lo hay; líneas impresas: por presentación (cantidad, precio y total), suelta en unidades, dos filas para dos líneas del mismo lote, con descripción de presentación pero sin cantidad de presentación en unidades |
| `pages/pos/dialogs/devolver-venta-dialog/devolver-venta-dialog.component.spec.ts` | `DevolverVentaDialogComponent` | 6 | Lista solo las líneas con unidades por devolver y limita cada cantidad a lo devolvible; motivo `06` si se devuelve todo lo restante y `07` si no; calcula el reembolso estimado; envía solo líneas con cantidad, con enums numéricos, y cierra al terminar; no envía sin cantidades o sin justificación; mapea el código de cantidad excedida del backend a su mensaje y deja el diálogo abierto |
| `pages/pos/venta-detail/venta-detail.component.spec.ts` | `VentaDetailComponent` (`/pos/:id`) | 20 | **Devoluciones (6):** ofrece Devolver en venta completada, con comprobante emitido y unidades por devolver y permiso de procesar devoluciones; lo oculta sin `ProcesarDevolucion`, si no está Completada, si no hay comprobante emitido o si todo se devolvió; oculta Anular cuando ya tiene devoluciones y lista sus notas de crédito. **Cobro de venta pendiente (3):** ofrece Cobrar en pendiente con `Ventas:Crear` y `Ventas:Actualizar`; lo oculta sin alguno; nunca en una venta no pendiente. **Verificación de receta (2):** muestra quién verificó y cuándo; nada si no hizo falta. **Errores al anular (3):** explica que anular una venta cobrada exige la propia caja abierta; mensaje genérico ante un código sin mapeo; el mensaje existe en español e inglés (lee los JSON reales). **Comprobante (2):** permite reimprimir el de una venta completada y no el de una anulada. **Vuelto y reimpresión (4):** vuelto bajo los pagos cuando el efectivo excedió el total y ninguno en pago exacto; los datos de reimpresión son idénticos a los de la impresión original; nombra al cliente anónimo como la impresión original aunque el backend envíe «Público General» |
| `pages/sesiones-caja/services/sesion-caja.service.spec.ts` | `SesionCajaService` | 2 | `search()` se limita a la sesión abierta del propio cajero cuando se da `empleadoId`; lo omite si no se da |
| `pages/sesiones-caja/sesiones-search/sesiones-search.component.spec.ts` | `SesionesSearchComponent` | 3 | Nuevo se ofrece con `POS:AperturarCaja`; se oculta a quien solo lee; `POS:Crear` o `POS:CerrarCaja` no sirven para abrir caja |
| `pages/sesiones-caja/sesiones-detail/sesiones-detail.component.spec.ts` | `SesionesDetailComponent` | 3 | Ofrece cerrar una sesión abierta con `POS:CerrarCaja`; lo oculta a quien puede leer y abrir pero no cerrar; nunca lo ofrece en una sesión ya cerrada |

---

## 3. Cobertura por pantalla

Criterio: un componente cuenta «con pruebas» si existe un spec que lo instancia o lo ejercita (`TestBed.createComponent` o import directo del componente). Una prueba no implica cobertura completa: la columna «Alcance» indica de qué tema trata. «Sin pruebas» significa que ningún spec lo menciona. No se midió cobertura de líneas (no se ejecutaron las pruebas).

Resumen: de los 75 componentes de `componentes-pages.txt`, **36 tienen al menos un spec** y **39 no tienen ninguno** (de estos, 12 son envoltorios de carga masiva sobre `CargaMasivaPageComponent`, que sí está probado, ver 2.3).

### 3.1 Componentes de `componentes-pages.txt`

| Componente (bajo `src/app/pages/`) | Specs | Alcance |
|---|---|---|
| `aseguradoras/aseguradoras-carga-masiva` | Sin pruebas | Envoltorio de `CargaMasivaPageComponent` |
| `aseguradoras/aseguradoras-detail` | `aseguradoras-detail.component.spec.ts` | Modo nueva, modo vista, guardado con campos vacíos |
| `aseguradoras/aseguradoras-search` | `aseguradoras-search.component.spec.ts` | Buscar y limpiar |
| `auth/auth-brand-panel` | Sin pruebas | Panel decorativo |
| `auth/forgot-password` | `forgot-password.component.spec.ts` | Completo en sus ramas de envío |
| `auth/login` | `login.component.spec.ts` | Errores de credenciales, servicio y límite |
| `auth/reset-password` | `reset-password.component.spec.ts` | Fragmento, reglas, envío, enlace vencido |
| `cambiar-clave` | `cambiar-clave.component.spec.ts` | Reglas y envío |
| `dashboard` | `dashboard.component.spec.ts`, `dashboard-links.spec.ts`, `dashboard-i18n.spec.ts` | Secciones por permiso, tendencia, errores, enlaces |
| `digemid-catalogo/digemid-catalogo-carga-masiva` | Sin pruebas | Envoltorio |
| `digemid-catalogo/digemid-catalogo-detail` | Sin pruebas | — |
| `digemid-catalogo/digemid-catalogo-search` | Sin pruebas | — |
| `empleados/empleados-detail` | `empleados-detail.component.spec.ts` | Solo segundo apellido opcional |
| `empleados/empleados-search` | `empleados-search.component.spec.ts` | Solo segundo apellido opcional |
| `empresa/empresa-detail` | Sin pruebas | — |
| `empresa/empresa-maintenance` | Sin pruebas | — |
| `empresa/empresa-register` | `empresa-register.component.spec.ts` | Validaciones y errores del registro |
| `empresa/empresa-search` | Sin pruebas | — |
| `formulaciones-clinicas/formulaciones-clinicas-detail` | Sin pruebas | — |
| `formulaciones-clinicas/formulaciones-clinicas-search` | Sin pruebas | — |
| `ingredientes-activos/ingredientes-activos-carga-masiva` | Sin pruebas | Envoltorio |
| `ingredientes-activos/ingredientes-activos-detail` | `ingredientes-activos-detail.delete.spec.ts` | Solo borrado de registro en uso |
| `ingredientes-activos/ingredientes-activos-search` | Sin pruebas | — |
| `jerarquias/jerarquias-carga-masiva` | Sin pruebas | Envoltorio |
| `jerarquias/jerarquias-detail` | Sin pruebas | — |
| `jerarquias/jerarquias-search` | Sin pruebas | — |
| `laboratorios/laboratorios-carga-masiva` | Sin pruebas | Envoltorio |
| `laboratorios/laboratorios-detail` | `laboratorios-detail.component.spec.ts` | Botones por permiso |
| `laboratorios/laboratorios-search` | `laboratorios-search.component.spec.ts` | Botones por permiso |
| `lotes/lotes-detail` | `lotes-detail.component.spec.ts` | Nueva, vista, edición |
| `lotes/lotes-search` | `lotes-search.component.spec.ts` | Buscar y limpiar |
| `medicamentos/medicamentos-carga-masiva` | Sin pruebas | Envoltorio |
| `medicamentos/medicamentos-detail` | `medicamentos-detail.component.spec.ts`, `medicamentos-detail.delete.spec.ts` | Nueva, vista, enlace a presentaciones, borrado en uso |
| `medicamentos/medicamentos-search` | `medicamentos-search.component.spec.ts` | Carga inicial y buscar |
| `pacientes/pacientes-carga-masiva` | Sin pruebas | Envoltorio |
| `pacientes/pacientes-detail` | `pacientes-detail.delete.spec.ts` | Solo borrado en uso |
| `pacientes/pacientes-search` | Sin pruebas | — |
| `pos/dialogs/anular-venta-dialog` | Sin pruebas | — |
| `pos/dialogs/aperturar-caja-dialog` | Sin pruebas | — |
| `pos/dialogs/comprobante-dialog` | `comprobante-dialog.component.spec.ts` | Contenido impreso, vuelto, líneas |
| `pos/dialogs/confirmar-venta-dialog` | Sin pruebas | (en `nueva-venta` se simula su cierre, no se prueba) |
| `pos/dialogs/devolver-venta-dialog` | `devolver-venta-dialog.component.spec.ts` | Cantidades, motivo, envío, error |
| `pos/nueva-venta` | `nueva-venta.component.spec.ts` (+ `pos-draft.spec.ts`, `vuelto.util.spec.ts`) | Ver 3.3 |
| `pos/search` (`ventas-search`) | Sin pruebas | — |
| `pos/venta-detail` | `venta-detail.component.spec.ts` | Ver 3.3 |
| `presentaciones-venta/presentaciones-venta-detail` | Sin pruebas | — |
| `presentaciones-venta/presentaciones-venta-modal` | `presentaciones-venta-modal.component.spec.ts` | Botones por permiso |
| `presentaciones-venta/presentaciones-venta-search` | Sin pruebas | — |
| `profesionales-salud/profesionales-salud-carga-masiva` | Sin pruebas | Envoltorio |
| `profesionales-salud/profesionales-salud-detail` | `profesionales-salud-detail.delete.spec.ts` | Solo borrado en uso |
| `profesionales-salud/profesionales-salud-search` | Sin pruebas | — |
| `proveedores/precio-proveedor-dialog` | `precio-proveedor-dialog.component.spec.ts` | Validaciones, costo neto, envío, errores |
| `proveedores/proveedores-carga-masiva` | Sin pruebas | Envoltorio |
| `proveedores/proveedores-detail` | `proveedores-detail.component.spec.ts`, `proveedores-detail.delete.spec.ts` | Tarjeta de precios y borrado en uso |
| `proveedores/proveedores-search` | Sin pruebas | — |
| `recetas-medicas/recetas-detail` | `recetas-detail.component.spec.ts` | Ver 3.3 |
| `recetas-medicas/recetas-scanner` | Sin pruebas | — |
| `recetas-medicas/recetas-search` | `recetas-search.component.spec.ts` | Acceso al escáner, buscar, limpiar, exportar |
| `roles/roles-carga-masiva` | Sin pruebas | Envoltorio |
| `roles/roles-detail` | `roles-detail.component.spec.ts` | Ver 3.3 |
| `roles/roles-search` | `roles-search.component.spec.ts` | Carga y filtro de jerarquía |
| `seguridad/cuentas-detail` | `cuentas-detail.component.spec.ts` | Ver 3.3 |
| `seguridad/cuentas-search` | Sin pruebas | — |
| `series-fiscales/series-fiscales-detail` | `series-fiscales-detail.component.spec.ts` | Prefijo, guardado, bloqueo por documentos emitidos |
| `series-fiscales/series-fiscales-search` | `series-fiscales-search.component.spec.ts` | Buscar y limpiar |
| `sesiones-caja/sesiones-detail` | `sesiones-detail.component.spec.ts` | Solo visibilidad del botón de cierre |
| `sesiones-caja/sesiones-search` | `sesiones-search.component.spec.ts` | Solo visibilidad del botón Nuevo |
| `stock-por-sucursal/stock-detail` | Sin pruebas | — |
| `stock-por-sucursal/stock-search` | Sin pruebas | — |
| `sucursales/sucursales-carga-masiva` | Sin pruebas | Envoltorio |
| `sucursales/sucursales-detail` | `sucursales-detail.delete.spec.ts` | Solo borrado en uso |
| `sucursales/sucursales-search` | Sin pruebas | — |
| `unidades-medida/unidades-medida-carga-masiva` | Sin pruebas | Envoltorio |
| `unidades-medida/unidades-medida-detail` | Sin pruebas | — |
| `unidades-medida/unidades-medida-search` | Sin pruebas | — |

Nota: «Envoltorio» designa a los 12 componentes `*-carga-masiva` (aseguradoras, DIGEMID, ingredientes activos, jerarquías, laboratorios, medicamentos, pacientes, profesionales de salud, proveedores, roles, sucursales y unidades de medida). Los 12 archivos referencian `CargaMasivaPageComponent` (búsqueda en `src/app/pages/**/*carga-masiva.component.ts`, 3 apariciones en cada uno); ver también [19a, sección 6.6](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md).

### 3.2 Rutas de `rutas-spa.csv`

Las 86 rutas. «Estado» refiere al componente que la sirve (ver 3.1). Además, **todas las rutas del shell** (las que cuelgan de `FullComponent`, es decir todas menos `/auth/*`) están cubiertas en cuanto a su permiso por `route-permissions.spec.ts` (toda ruta hija está mapeada a un módulo, salvo `dashboard` y `cambiar-clave`) y `permission.guard.spec.ts` (módulo y acción exigidos). Esas pruebas no ejecutan el enrutador real con cada URL: usan `appRoutes` y funciones puras.

| URL | Componente | Estado del componente |
|---|---|---|
| `/auth/login` | `LoginComponent` | Probado (`login.component.spec.ts`) |
| `/auth/register` | `EmpresaRegisterComponent` | Probado (`empresa-register.component.spec.ts`) |
| `/auth/forgot-password` | `ForgotPasswordComponent` | Probado |
| `/auth/reset-password` | `ResetPasswordComponent` | Probado |
| `/dashboard` | `DashboardComponent` | Probado (3 specs) |
| `/cambiar-clave` | `CambiarClaveComponent` | Probado |
| `/aseguradoras` | `AseguradorasSearchComponent` | Probado |
| `/aseguradoras/nueva` | `AseguradorasDetailComponent` | Probado |
| `/aseguradoras/carga-masiva` | `AseguradorasCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/aseguradoras/:id` | `AseguradorasDetailComponent` | Probado |
| `/digemid-catalogo` | `DigemidCatalogoSearchComponent` | Sin pruebas |
| `/digemid-catalogo/carga-masiva` | `DigemidCatalogoCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/digemid-catalogo/:id` | `DigemidCatalogoDetailComponent` | Sin pruebas |
| `/empleados` | `EmpleadosSearchComponent` | Parcial (segundo apellido) |
| `/empleados/nueva` | `EmpleadosDetailComponent` | Parcial (segundo apellido) |
| `/empleados/:id` | `EmpleadosDetailComponent` | Parcial (segundo apellido) |
| `/empresas` | `EmpresaSearchComponent` | Sin pruebas |
| `/empresas/:id` | `EmpresaDetailComponent` | Sin pruebas |
| `/formulaciones-clinicas` | `FormulacionesClinicasSearchComponent` | Sin pruebas |
| `/formulaciones-clinicas/nueva` | `FormulacionesClinicasDetailComponent` | Sin pruebas |
| `/formulaciones-clinicas/:id` | `FormulacionesClinicasDetailComponent` | Sin pruebas |
| `/ingredientes-activos` | `IngredientesActivosSearchComponent` | Sin pruebas |
| `/ingredientes-activos/nueva` | `IngredientesActivosDetailComponent` | Parcial (solo borrado en uso) |
| `/ingredientes-activos/carga-masiva` | `IngredientesActivosCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/ingredientes-activos/:id` | `IngredientesActivosDetailComponent` | Parcial (solo borrado en uso) |
| `/jerarquias` | `JerarquiasSearchComponent` | Sin pruebas |
| `/jerarquias/nueva` | `JerarquiasDetailComponent` | Sin pruebas |
| `/jerarquias/carga-masiva` | `JerarquiasCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/jerarquias/:id` | `JerarquiasDetailComponent` | Sin pruebas |
| `/laboratorios` | `LaboratoriosSearchComponent` | Parcial (botones por permiso) |
| `/laboratorios/nueva` | `LaboratoriosDetailComponent` | Parcial (botones por permiso) |
| `/laboratorios/carga-masiva` | `LaboratoriosCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/laboratorios/:id` | `LaboratoriosDetailComponent` | Parcial (botones por permiso) |
| `/lotes` | `LotesSearchComponent` | Probado |
| `/lotes/nueva` | `LotesDetailComponent` | Probado |
| `/lotes/:id` | `LotesDetailComponent` | Probado |
| `/medicamentos` | `MedicamentosSearchComponent` | Probado (carga y buscar) |
| `/medicamentos/nueva` | `MedicamentosDetailComponent` | Probado (nueva y borrado) |
| `/medicamentos/carga-masiva` | `MedicamentosCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/medicamentos/:id` | `MedicamentosDetailComponent` | Probado (vista y borrado) |
| `/pacientes` | `PacientesSearchComponent` | Sin pruebas |
| `/pacientes/nueva` | `PacientesDetailComponent` | Parcial (solo borrado en uso) |
| `/pacientes/carga-masiva` | `PacientesCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/pacientes/:id` | `PacientesDetailComponent` | Parcial (solo borrado en uso) |
| `/pos` | `VentasSearchComponent` | Sin pruebas |
| `/pos/nueva` | `NuevaVentaComponent` | Probado (ver 3.3) |
| `/pos/:id` | `VentaDetailComponent` | Probado (ver 3.3) |
| `/presentaciones-venta` | `PresentacionesVentaSearchComponent` | Sin pruebas |
| `/presentaciones-venta/nueva` | `PresentacionesVentaDetailComponent` | Sin pruebas |
| `/presentaciones-venta/:id` | `PresentacionesVentaDetailComponent` | Sin pruebas |
| `/profesionales-salud` | `ProfesionalesSaludSearchComponent` | Sin pruebas |
| `/profesionales-salud/nueva` | `ProfesionalesSaludDetailComponent` | Parcial (solo borrado en uso) |
| `/profesionales-salud/carga-masiva` | `ProfesionalesSaludCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/profesionales-salud/:id` | `ProfesionalesSaludDetailComponent` | Parcial (solo borrado en uso) |
| `/proveedores` | `ProveedoresSearchComponent` | Sin pruebas |
| `/proveedores/nueva` | `ProveedoresDetailComponent` | Parcial (precios y borrado) |
| `/proveedores/carga-masiva` | `ProveedoresCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/proveedores/:id` | `ProveedoresDetailComponent` | Parcial (precios y borrado) |
| `/recetas-medicas` | `RecetasSearchComponent` | Probado (+ `recetas-medicas.routes.spec.ts`) |
| `/recetas-medicas/escanear` | `RecetasScannerComponent` | Sin pruebas (la ruta y su permiso `Analizar` sí están probados) |
| `/recetas-medicas/nueva` | `RecetasDetailComponent` | Probado |
| `/recetas-medicas/:id` | `RecetasDetailComponent` | Probado |
| `/roles` | `RolesSearchComponent` | Parcial |
| `/roles/nueva` | `RolesDetailComponent` | Probado |
| `/roles/carga-masiva` | `RolesCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/roles/:id` | `RolesDetailComponent` | Probado |
| `/cuentas` | `CuentasSearchComponent` | Sin pruebas |
| `/cuentas/nueva` | `CuentasDetailComponent` | Parcial (el spec cubre vista y edición de una cuenta; no hay caso de creación) |
| `/cuentas/:id` | `CuentasDetailComponent` | Probado |
| `/series-fiscales` | `SeriesFiscalesSearchComponent` | Probado |
| `/series-fiscales/nueva` | `SeriesFiscalesDetailComponent` | Probado |
| `/series-fiscales/:id` | `SeriesFiscalesDetailComponent` | Probado |
| `/sesiones-caja` | `SesionesSearchComponent` | Parcial (solo botón Nuevo) |
| `/sesiones-caja/nueva` | `SesionesDetailComponent` | Sin pruebas del modo nueva (el spec solo cubre el botón de cierre) |
| `/sesiones-caja/:id` | `SesionesDetailComponent` | Parcial (solo botón de cierre) |
| `/stock-por-sucursal` | `StockSearchComponent` | Sin pruebas (solo el servicio) |
| `/stock-por-sucursal/nueva` | `StockDetailComponent` | Sin pruebas (solo el servicio) |
| `/stock-por-sucursal/:id` | `StockDetailComponent` | Sin pruebas (solo el servicio) |
| `/sucursales` | `SucursalesSearchComponent` | Sin pruebas |
| `/sucursales/nueva` | `SucursalesDetailComponent` | Parcial (solo borrado en uso) |
| `/sucursales/carga-masiva` | `SucursalesCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/sucursales/:id` | `SucursalesDetailComponent` | Parcial (solo borrado en uso) |
| `/unidades-medida` | `UnidadesMedidaSearchComponent` | Sin pruebas |
| `/unidades-medida/nueva` | `UnidadesMedidaDetailComponent` | Sin pruebas |
| `/unidades-medida/carga-masiva` | `UnidadesMedidaCargaMasivaComponent` | Sin pruebas (envoltorio) |
| `/unidades-medida/:id` | `UnidadesMedidaDetailComponent` | Sin pruebas |

### 3.3 Pantallas críticas: qué SÍ y qué NO está probado

Método: se cruzó la lista de métodos de cada componente con las apariciones de su nombre en el spec (si un método no aparece en ningún spec, no se ejercita directamente) y se leyeron los títulos de los casos. «Directamente» significa que el spec lo llama o provoca. Los apartados citados son los de los inventarios de pantallas.

#### Punto de venta: nueva venta (`/pos/nueva`, [19f §3](../19-inventario-de-pantallas/19f-punto-de-venta.md))

| Apartado del inventario | SÍ está probado | NO está probado |
|---|---|---|
| §3.2 búsqueda de productos | Catálogo ordenado por más vendidos con el stock vendible de la sucursal; fallo de búsqueda; lotes y presentaciones del producto y sus errores; marca de «requiere receta» y precio o aviso | `setTab`, insignias de stock (`stockBadgeClass`, `stockBadgeLabel`), `onPresentacionChange` |
| §3.3 carrito | Aviso de receta faltante; vaciar el carrito y quitar la última línea reinician los pagos | `agregarAlCarrito` se llama solo en 2 casos (`nueva-venta.component.spec.ts:296,315`, ambos sobre el aviso de receta); el ajuste de cantidades (`ajustarCantidadCarrito`) y los topes de stock no tienen caso |
| §3.4 cliente | — (solo el id del cliente en el borrador, en `pos-draft.spec.ts`) | Selección y limpieza de cliente (`selectCliente`, `limpiarCliente`), carga de sus recetas (`cargarRecetasCliente`), anónimo |
| §3.5 receta (aviso, selector, número y verificación) | Muy cubierto: regla de empresa flexible o estricta, casilla «Receta verificada», condiciones controladas, número de receta obligatorio, bloqueo de cobro frente a dejar en espera, `recetaVerificada` en el envío, códigos de error del backend | — |
| §3.6 escaneo de receta | Pestaña visible solo con `Recetas:Analizar`; añadir una línea sugerida con el precio del lote | Subir archivo y arrastrar (`onRecetaFileSelected`, `onRecetaDrop`), `interpretarReceta`, `limpiarReceta`, etiquetas de confianza |
| §3.7 pagos | Saldo restante prellenado, céntimos, número de operación para no efectivo, códigos del backend, pagos obsoletos, vuelto (`calcularVuelto`) | `quitarPago`, modo de pago simple vs. avanzado más allá de lo que usan los casos, iconos de método |
| §3.8 aseguradora | — | `toggleSeguro`, `selectAseguradora` (la zona está oculta por una bandera según el inventario) |
| §3.9 borrador local | `pos-draft.spec.ts` (vigencia, privacidad del cliente, normalización) y los casos de pagos en el borrador de una venta en espera | El guardado y la restauración del carrito completo en el componente: solo se leen y escriben en el spec los borradores por venta en espera (`${DRAFT_KEY}-venta-9`, `nueva-venta.component.spec.ts:819-845`); no hay caso del borrador del carrito de una venta nueva |
| §3.10 retomar venta en espera | Conocer la condición de venta de una venta retomada y enviar `recetaVerificada` solo al cobrar; ignorar efectivo viejo | Carga de la venta pendiente (`cargarVentaPendiente`) como flujo propio |
| §3.11 acciones del terminal | Cobrar (`procesarVenta`) y dejar en espera (`guardarPendiente`) en sus bloqueos por receta y referencia; abrir caja solo con `POS:AperturarCaja`; enlace a registrar receta con `Recetas:Crear` | Lo posterior al cierre del comprobante (reinicio de la venta, `resetVenta`): los casos usan un diálogo de confirmación falso que acepta y un diálogo de comprobante que nunca cierra (`nueva-venta.component.spec.ts:71-73`); `abrirCaja()` no se llama |
| §3.12 mapa de códigos de error | Los códigos de receta y de referencia de pago | Los demás códigos del mapa |
| §4 diálogo de confirmación | — | `ConfirmarVentaDialogComponent` sin spec |
| §5 comprobante | `comprobante-dialog.component.spec.ts` y `receipt-print.service.spec.ts`: datos legales, documento del cliente, estado, vuelto, líneas por presentación o sueltas, impresión aislada de 80 mm | Impresión real en el navegador; el botón de nueva venta solo se prueba en su visibilidad por permiso |

#### Punto de venta: ventas y detalle (`/pos`, `/pos/:id`, [19f §6 a §9](../19-inventario-de-pantallas/19f-punto-de-venta.md))

| Apartado | SÍ está probado | NO está probado |
|---|---|---|
| §6 búsqueda de ventas (`/pos`) | `VentaService.search` (envío de `estado` 0) y `exportar`; los enlaces del dashboard que abren esta búsqueda ya filtrada (`dashboard-links.spec.ts`) | `VentasSearchComponent` completo: filtros, buscar, limpiar, paginación, exportar |
| §7 detalle de venta | Visibilidad de Devolver, Anular, Cobrar y reimpresión; notas de crédito; quién verificó la receta; vuelto; datos de reimpresión iguales a la impresión original | Las acciones en sí: `cobrar()` y `devolver()` no se llaman; la anulación se prueba solo en su ruta de error |
| §8 anulación | Mensaje cuando anular una venta cobrada exige la propia caja abierta (`Venta.Anular.SinSesionAbierta`) y mensaje genérico ante otro código; el texto existe en es/en; `VentaService.anular` envía JSON explícito | Anulación exitosa y recarga; `AnularVentaDialogComponent` sin spec (validación del motivo) |
| §9 devolución | `DevolverVentaDialogComponent`: cantidades máximas, motivo 06/07, reembolso estimado, envío solo de líneas con cantidad, justificación obligatoria, error de cantidad excedida; `VentaService.procesarDevolucion` | Cancelar (`onCancel`), la etiqueta del método de reembolso (`metodoPagoLabel`), la apertura del diálogo desde el detalle y la recarga tras devolver |

#### Caja: sesiones de caja ([19f §1 y §2](../19-inventario-de-pantallas/19f-punto-de-venta.md))

| Apartado | SÍ está probado | NO está probado |
|---|---|---|
| §1.1 búsqueda (`/sesiones-caja`) | Botón Nuevo solo con `POS:AperturarCaja` (no sirven `POS:Crear` ni `POS:CerrarCaja`); `SesionCajaService.search` por empleado | `buscar`, `limpiar`, `exportar`, paginación |
| §1.2 apertura como pantalla (`/sesiones-caja/nueva`) | El permiso exigido para entrar (`AperturarCaja`, en `permission.guard.spec.ts` y `route-permissions.spec.ts`) | El formulario de apertura (`aperturar`) |
| §1.3 detalle, cierre y arqueo | Visibilidad del botón de cierre según `POS:CerrarCaja` y estado de la sesión | `confirmarCierre` (cierre con monto declarado), `cancelar`, y la presentación de monto calculado, declarado y diferencia (la plantilla solo muestra `montoCierreCalculado`, `montoCierreDeclarado` y `diferenciaArqueo` que llegan del backend: `sesion-caja.model.ts:17-19`, `sesiones-detail.component.html:143-171`). Tampoco tienen caso `SesionCajaService.cerrar` (POST `/cierre`, `sesion-caja.service.ts:39-40`) ni la apertura. **El SPA no tiene ninguna prueba del cierre ni del arqueo** |
| §2 diálogo de apertura | — | `AperturarCajaDialogComponent` sin spec (`aperturar`, `cancelar`) |

#### Devoluciones

No existe listado ni detalle de devoluciones como pantalla (la regla del proyecto lo indica y las rutas de `rutas-spa.csv` no incluyen ninguna). El único flujo del SPA es el diálogo desde `/pos/:id`, cubierto arriba. No hay spec de un servicio de devoluciones aparte: la llamada vive en `VentaService.procesarDevolucion` (probada).

#### Recetas médicas (`/recetas-medicas`, [19e §5](../19-inventario-de-pantallas/19e-inventario-y-atencion.md))

| Apartado | SÍ está probado | NO está probado |
|---|---|---|
| §5.1 búsqueda | Entrada al escáner según `Recetas:Analizar` y rol `Admin`; `buscar` con paciente y fechas; `limpiar`; `exportar` | `goToScanner`, `goToNew`, `selectMedico`, paginación (`onPageChange`) |
| §5.2 detalle | Crear: obligatoriedad de paciente y prescriptor, fecha `yyyy-MM-dd`, número de receta recortado y con tope, códigos del backend; vista; número conservado al editar; Eliminar según `Recetas:Eliminar` | `eliminar` (confirmación y borrado), `cancelEdit`, `validateAutocompletes`; no hay servicio `RecetaMedicaService` con spec |
| §5.3 escáner (`/recetas-medicas/escanear`) | Solo la ruta: declarada antes de `:id` y con `Analizar` exigido por el guard (`recetas-medicas.routes.spec.ts`, `route-permissions.spec.ts`, `permission.guard.spec.ts`) | `RecetasScannerComponent` completo: carga de imagen, cámara, galería, arrastre, interpretar, confianza, equivalentes con stock |

#### Roles (`/roles`, [19b §4](../19-inventario-de-pantallas/19b-administracion.md))

| Apartado | SÍ está probado | NO está probado |
|---|---|---|
| §4.1 búsqueda | Sin HTTP hasta `search()`; límites de jerarquía solo si difieren del deslizador | `limpiar`, exportar, paginación, `goToNew` como casos propios |
| §4.2 detalle y configuración | Todo lo del permiso (Editar, Eliminar, `GestionarPermisos`), `save()` con y sin gestión de permisos, etiquetas traducidas, alternar permisos y sucursales, vista previa del menú | `confirmDelete`, `cancelEdit`, `toggleModulo` (marcar un módulo entero), `areAllSelected`/`areSomeSelected`/`countSelected` (contadores), pestañas de carga (`loadPermisos`, `loadSucursales`) |
| §4.3 carga masiva | — (envoltorio sin spec; el componente compartido sí está probado) | `RolesCargaMasivaComponent` |
| Servicio | `RolService` (CRUD, exportar, catálogo, permisos, sucursales) | — |
| i18n | `roles-i18n.spec.ts` | — |

#### Cuentas de usuario (`/cuentas`, [19b §5](../19-inventario-de-pantallas/19b-administracion.md))

| Apartado | SÍ está probado | NO está probado |
|---|---|---|
| §5.1 búsqueda | — | `CuentasSearchComponent` completo |
| §5.2 detalle | Rol (asignar, cambiar, sin cambios), botones por `Seguridad:Actualizar`/`Eliminar`/`AsignarRoles`, campo Bloqueado, sucursales (asignar, quitar, excluir asignadas) | Creación de la cuenta (`/cuentas/nueva`: `onEmpleadoSelected`), `confirmDelete`, `cancelEdit`, mapeo de errores (`errorMessage`), `isBloqueado` solo se ve por el texto en pantalla |
| Servicio | — (`CuentaService` sin spec) | `CuentaService` |

---

## 4. Huecos

### 4.1 Servicios HTTP de `pages` sin spec

De los 25 archivos `*.service.ts` bajo `src/app/pages` (24 servicios HTTP y `ReceiptPrintService`), tienen spec 12 (completos o parciales) y 13 no tienen ninguno:

| Servicio | Archivo |
|---|---|
| `CuentaService` | `pages/seguridad/services/cuenta.service.ts` |
| `RecetaMedicaService` | `pages/recetas-medicas/services/receta-medica.service.ts` |
| `PacienteService` | `pages/pacientes/services/paciente.service.ts` |
| `ProfesionalSaludService` | `pages/profesionales-salud/services/profesional-salud.service.ts` |
| `ProveedorService` (incluye precios de compra) | `pages/proveedores/services/proveedor.service.ts` |
| `EmpresaService` | `pages/empresa/services/empresa.service.ts` |
| `LaboratorioService` | `pages/laboratorios/services/laboratorio.service.ts` |
| `IngredienteActivoService` | `pages/ingredientes-activos/services/ingrediente-activo.service.ts` |
| `UnidadMedidaService` | `pages/unidades-medida/services/unidad-medida.service.ts` |
| `JerarquiaUomService` | `pages/jerarquias/services/jerarquia-uom.service.ts` |
| `PresentacionVentaService` | `pages/presentaciones-venta/services/presentacion-venta.service.ts` |
| `FormulacionClinicaService` | `pages/formulaciones-clinicas/services/formulacion-clinica.service.ts` |
| `DigemidCatalogoService` | `pages/digemid-catalogo/services/digemid-catalogo.service.ts` |

Con spec parcial: `SesionCajaService` (2 casos, solo `search`; faltan apertura, cierre y demás métodos), `SucursalService` (1 caso, solo `getPermitidas`). Con spec razonablemente completo: `AseguradoraService`, `EmpleadoService`, `LoteService`, `MedicamentoService`, `RolService`, `SerieFiscalService`, `StockService`, `VentaService`, `DashboardService`, `ReceiptPrintService`.

### 4.2 Core y shared sin spec

| Elemento | Archivo | Nota |
|---|---|---|
| `CSLoadingInterceptor` y `LoadingService` | `shared/services/loading/loading.interceptor.ts`, `loading.service.ts` | Único interceptor sin spec; está registrado en `app.config.ts:47` |
| `SearchStateService` | `core/services/shared/search-state.service.ts` | Solo se verifica de forma indirecta que `AuthService` lo vacía (`auth.service.spec.ts`) |
| `TranslocoHttpLoader` | `core/transloco/transloco-loader.ts` | Sin spec |
| `LoggerService`, `TimezoneService` | `core/services/logger.service.ts`, `core/services/shared/timezone.service.ts` | Sin spec (`TimezoneService` sin uso según [19a §6.13](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md)) |
| Componentes compartidos en uso | `ConfirmDialogComponent`, `PageHeaderComponent`, `AutocompleteInputComponent`, `RangeSliderFieldComponent`, `CSLoadingBarComponent`, `ServerErrorSnackbarComponent`, `MatPaginatorIntlTransloco` | Sin spec propio ([19a §6.7](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) los marca «en uso»). Solo `CargaMasivaPageComponent` y `MatDatepickerIntlTransloco` tienen spec |
| `catalog-item-autocomplete.util.ts` | `shared/utils/` (262 líneas) | Sin spec; su uso real no se verificó |
| Componentes, directivas y pipes sin uso | `DatetimePickerComponent`, `DecimalInputComponent`, `AutocompleteInputMultipleComponent`, `MultiselectAutocompleteInputComponent`, `CargaMasivaComponent`, diálogos `ErrorDialog`/`SimpleDialog`/`SuccessDialog`/`TextInputDialog`/`ThreeOptionDialog`, `StatusChipsComponent`, directivas `Numeric`, `Alphanumeric`, `PositiveDecimal`, `PositiveInteger`, `DecimalPrecision`, `BlockManualDateInput`, `ValidName`, `ValidDescription`, pipes `datetimeoffset`, `timezone`, `formatThousands`, `lineBreakToBr`, `CustomPaginatorIntl` | Sin spec. Según [19a §6.13 y §7.2](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) no se usan en ninguna plantilla, por lo que su ausencia de pruebas es de bajo riesgo (el riesgo está en conservarlos sin pruebas) |
| Layout | `FullComponent`, `SidebarComponent`, `BreadcrumbComponent`, `NavItemComponent`, `BlankComponent`, `CustomizerComponent`, `NavService`, `CoreService`, layout horizontal | Sin spec. Solo `HeaderComponent` vertical y `visibleNavItems` tienen spec |
| Configuración de la aplicación | `app.config.ts`, `app.routes.ts` | Sin spec propio; `app.routes.ts` se usa como dato en `route-permissions.spec.ts` y `menu-preview.spec.ts`. No hay prueba de que `app.config.ts` registre los interceptores en el orden correcto (`authInterceptor`, `CSLoadingInterceptor`, `app.config.ts:47`) |
| Rutas por módulo | `*.routes.ts` | Solo `recetas-medicas.routes.ts` tiene spec |

### 4.3 Pantallas sin ningún spec

Ver 3.1 (39 componentes). Resumen por módulo, de más a menos crítico:

- **Caja y ventas:** `VentasSearchComponent`, `AnularVentaDialogComponent`, `AperturarCajaDialogComponent`, `ConfirmarVentaDialogComponent`.
- **Seguridad:** `CuentasSearchComponent`.
- **Recetas:** `RecetasScannerComponent`.
- **Inventario:** `StockSearchComponent`, `StockDetailComponent` (solo el servicio tiene spec).
- **Pacientes, profesionales, sucursales (búsqueda):** `PacientesSearchComponent`, `ProfesionalesSaludSearchComponent`, `SucursalesSearchComponent`.
- **Empresa:** `EmpresaSearchComponent`, `EmpresaDetailComponent`, `EmpresaMaintenanceComponent`.
- **Catálogos:** unidades de medida (búsqueda y detalle), jerarquías (búsqueda y detalle), ingredientes activos (búsqueda), presentaciones de venta (búsqueda y detalle), formulaciones clínicas (búsqueda y detalle), proveedores (búsqueda), DIGEMID (búsqueda, detalle).
- **Carga masiva:** los 12 envoltorios (el componente compartido sí tiene spec).
- **Auth:** `AuthBrandPanelComponent` (decorativo).

### 4.4 Comportamientos críticos sin prueba

| # | Comportamiento | Dónde (pantalla e inventario) | Observación |
|---|---|---|---|
| 1 | Cierre de caja y arqueo: enviar monto declarado y mostrar calculado y diferencia | `/sesiones-caja/:id`, [19f §1.3](../19-inventario-de-pantallas/19f-punto-de-venta.md) | Ninguna prueba del cierre en el SPA (solo la visibilidad del botón) |
| 2 | Apertura de caja (pantalla `/sesiones-caja/nueva` y diálogo desde el POS) | [19f §1.2 y §2](../19-inventario-de-pantallas/19f-punto-de-venta.md) | Solo se prueba el permiso que da acceso |
| 3 | Anulación de venta exitosa y el diálogo del motivo | `/pos/:id`, [19f §7 y §8](../19-inventario-de-pantallas/19f-punto-de-venta.md) | Solo se prueba el error `Venta.Anular.SinSesionAbierta` y el genérico |
| 4 | Ejecución de Cobrar y Devolver desde el detalle de venta (apertura del diálogo, recarga) | `/pos/:id`, [19f §7](../19-inventario-de-pantallas/19f-punto-de-venta.md) | Solo la visibilidad de los botones |
| 5 | Diálogo de confirmación de venta | [19f §4](../19-inventario-de-pantallas/19f-punto-de-venta.md) | En las pruebas del POS se simula que acepta |
| 6 | Cliente, aseguradora y ajuste de cantidades del carrito en el POS | [19f §3.3, §3.4, §3.8](../19-inventario-de-pantallas/19f-punto-de-venta.md) | Sin casos |
| 7 | Escaneo de receta: interpretar y arrastrar archivo (POS y `/recetas-medicas/escanear`) | [19f §3.6](../19-inventario-de-pantallas/19f-punto-de-venta.md), [19e §5.3](../19-inventario-de-pantallas/19e-inventario-y-atencion.md) | Sin casos sobre la IA ni sobre la pantalla del escáner |
| 8 | Búsqueda de ventas con sus filtros | `/pos`, [19f §6](../19-inventario-de-pantallas/19f-punto-de-venta.md) | Sin spec |
| 9 | Creación de cuenta de usuario (selección de empleado) y borrado | `/cuentas/nueva`, [19b §5.2](../19-inventario-de-pantallas/19b-administracion.md) | Sin casos; `CuentaService` sin spec |
| 10 | Borrado de un rol, cancelar edición y marcado de un módulo entero de permisos | `/roles/:id`, [19b §4.2](../19-inventario-de-pantallas/19b-administracion.md) | Sin casos |
| 11 | Eliminar una receta (confirmación y borrado) | `/recetas-medicas/:id`, [19e §5.2](../19-inventario-de-pantallas/19e-inventario-y-atencion.md) | Solo se prueba que el botón aparezca según el permiso |
| 12 | Stock por sucursal: pantalla de ingreso y ajuste | [19e §2](../19-inventario-de-pantallas/19e-inventario-y-atencion.md) | Solo el servicio (`registrar`, `ajustar`) |
| 13 | Reglas de dependencias al borrar sucursales, productos, pacientes, profesionales, ingredientes y proveedores: solo el mensaje de 409 en uso | Ver 2.7 y 2.8; [19e §6](../19-inventario-de-pantallas/19e-inventario-y-atencion.md) | Se prueba con `describeDeleteInUse`; el borrado exitoso y la navegación posterior no tienen caso |
| 14 | Patrón común de las pantallas maestras (tres tarjetas de búsqueda, alternancia vista/edición, guardar, eliminar con confirmación, exportar) | Todos los maestros | Solo lo prueban `aseguradoras`, `lotes`, `series-fiscales` y, parcialmente, `medicamentos` y `laboratorios`. El resto no tiene regresión |
| 15 | Renovación de sesión y reintento ante 401 en una pantalla real | Transversal | Probado a nivel de interceptor y de servicio; no existe prueba integrada |
| 16 | Barra de carga ligada a peticiones (`CSLoadingInterceptor`) | [19a §5.5](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) | Sin spec |
| 17 | Cambio de idioma en toda la aplicación | [19a §4.5](../19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) | Solo hay paridad de claves es/en en `dashboard` y `roles`, y las etiquetas del datepicker; ningún otro scope tiene prueba de paridad |
| 18 | Presentaciones de venta (formulario) y jerarquías de unidades | Catálogos | Solo el modal tiene spec (botones por permiso) |

### 4.5 Huecos de estrategia

- No hay pruebas de extremo a extremo ni de integración contra la API real (ninguna herramienta en `package.json`).
- No hay umbral de cobertura ni informe de cobertura publicado (ver 1.2); no se puede afirmar un porcentaje.
- Los componentes grandes se prueban sobre todo por su clase: la plantilla (HTML) se renderiza en pocos casos (24 specs), así que un fallo de enlace de plantilla (botón que no llama al método correcto, por ejemplo) puede pasar sin ser detectado en los componentes que solo se prueban por su clase.
- El `shared` de Karma (`--configuration shared`) solo alcanza un spec.
- Las pruebas de i18n dependen de que `public/` se sirva en Karma (`angular.json:121-126`); si se cambia la configuración de `assets`, fallarían los `fetch('/i18n/...')`.

---

## 5. Cómo se corren y dónde

### 5.1 Comandos (resumen)

Detalle, requisitos (Chrome instalado) y particularidades de Windows en la [guía del desarrollador, sección 4.2](../08-guia-del-desarrollador.md#42-spa). Resumen verificado contra `package.json:10-11`, `karma.conf.js` y `angular.json:116-137`:

| Objetivo | Comando | Efecto |
|---|---|---|
| Desarrollo con observación | `npm test` (`ng test`) | Karma abre Chrome con ventana y vuelve a ejecutar al guardar |
| Una sola pasada, sin interfaz | `npm run test:ci` | `ng test --watch=false --browsers=ChromeHeadlessCI` |
| Solo el spec de `core/services/shared` | `npx ng test --configuration shared` | Un único spec hoy |
| Con informe de cobertura | `npx ng test --watch=false --code-coverage` (según la guía) | Genera `coverage/sofia-web-spa` (html y resumen de texto) |

### 5.2 Integración continua

- El repositorio `sofia-web-spa` **no tiene carpeta `.github`** ni otro archivo de pipeline: se comprobó en el sistema de archivos (`ls .github` falla) y con `git ls-files` (sin coincidencias para `.github`, `.yml`, pipelines). Por tanto **ninguna prueba del SPA corre en CI**.
- El único workflow del conjunto está en el backend: `sofia-apis/.github/workflows/security.yml`; no contiene pasos de `npm` ni de `ng test` (búsqueda de «npm», «ng test», «frontend»: sin coincidencias; la guía lo describe en [la sección 6](../08-guia-del-desarrollador.md#6-integración-continua-ci)).
- El script `test:ci` existe y está preparado para CI, pero nadie lo invoca automáticamente.

### 5.3 Hooks de commit (`sofia-web-spa/.husky`)

| Hook | Qué ejecuta | ¿Corre pruebas? |
|---|---|---|
| `pre-commit` | `npm run -s precommit:lint` → `lint-staged`: ESLint con `--fix` sobre `*.{ts,tsx,js,mjs,cjs}` y Prettier sobre `*.{html,css,scss,md,json,yml,yaml}` (`package.json:66-73`, `.husky/pre-commit:4`) | **No** |
| `commit-msg` | `commitlint` (Conventional Commits; cabecera máx. 100; asunto sin mayúscula inicial) (`.husky/commit-msg:4`, `commitlint.config.js`) | **No** |

En consecuencia, las pruebas del SPA solo se ejecutan cuando un desarrollador las corre a mano; nada impide commitear con pruebas fallando. La guía ([sección 8.3, paso 10](../08-guia-del-desarrollador.md)) recomienda `npm run lint` y `npm run test:ci` antes del commit, como convención, no como control.

---

## 6. Pendientes y «No verificado»

### 6.1 No verificado

- **Que las pruebas pasen hoy.** No se ejecutó ninguna prueba ni compilación (lo prohíbe el encargo). Todo lo descrito sale de la lectura de los specs, no de una corrida.
- **Número real de casos ejecutados.** Las cifras (492 `it(`) son apariciones de texto; los casos generados en bucles (p. ej. `permission.guard.spec.ts:96`, `nueva-venta.component.spec.ts:400,437,648,763`) y los 12 de `describeDeleteInUse` hacen que el total ejecutado sea distinto.
- **Cobertura de líneas o ramas.** No se midió; las columnas «Alcance» y «NO está probado» salen de cruzar nombres de métodos con el texto de los specs (un método llamado desde otro método probado queda cubierto aunque no aparezca por nombre, y un nombre que aparece en un spec no implica que se pruebe a fondo).
- **Contenido de los 39 componentes sin spec y de los inventarios 19c y 19d** (catálogos): se usaron solo para saber el nombre de las pantallas; no se contrastaron sus apartados con huecos concretos de esas pantallas.
- **Uso real de `shared/utils/catalog-item-autocomplete.util.ts`** (si se usa en alguna pantalla).
- **Los apartados citados de 19e y 19f** (§3.x, §5.x) se tomaron de sus encabezados; no se releyó cada apartado completo para esta parte.

### 6.2 En curso (sin commitear)

Ninguno: `git status --short` sin cambios en `sofia-web-spa` y `sofia-apis` al 2026-10-15; commits `eeffbb6` y `43f5529`, iguales a la línea base.

### 6.3 Pendientes

- El README de `docs/_inventario` dice 69 specs; en disco hay **75** (6 son `*.delete.spec.ts` con 0 `it(` literales). Conviene actualizarlo.
- Añadir cobertura mínima y un pipeline para el SPA queda fuera de este documento (solo se describe el estado actual).
- La Parte A (backend) de esta estrategia está en el documento hermano `09a`, que no se consultó para este archivo.

---

## 7. Tabla de cobertura

Cada spec de `docs/_inventario/pruebas-spa.csv` (75) y el apartado donde quedó documentado.

| Spec | Casos `it(` | Apartado |
|---|---|---|
| `app.component.spec.ts` | 2 | 2.1 |
| `core/auth/auth.guard.spec.ts` | 3 | 2.2 |
| `core/auth/auth.service.spec.ts` | 30 | 2.2 |
| `core/auth/password-change.guard.spec.ts` | 3 | 2.2 |
| `core/auth/permission.guard.spec.ts` | 12 | 2.2 |
| `core/auth/route-permissions.spec.ts` | 11 | 2.2 |
| `core/auth/user-storage.spec.ts` | 3 | 2.2 |
| `core/interceptors/auth.interceptor.spec.ts` | 14 | 2.2 |
| `core/services/shared/error-notifier.service.spec.ts` | 7 | 2.2 |
| `matdash/.../vertical/header/header.component.spec.ts` | 4 | 2.4 |
| `matdash/.../vertical/sidebar/visible-nav-items.spec.ts` | 3 | 2.4 |
| `pages/aseguradoras/aseguradoras-detail/aseguradoras-detail.component.spec.ts` | 3 | 2.7 |
| `pages/aseguradoras/aseguradoras-search/aseguradoras-search.component.spec.ts` | 3 | 2.7 |
| `pages/aseguradoras/services/aseguradora.service.spec.ts` | 7 | 2.7 |
| `pages/auth/forgot-password/forgot-password.component.spec.ts` | 4 | 2.5 |
| `pages/auth/login/login.component.spec.ts` | 7 | 2.5 |
| `pages/auth/reset-password/reset-password.component.spec.ts` | 5 | 2.5 |
| `pages/cambiar-clave/cambiar-clave.component.spec.ts` | 5 | 2.5 |
| `pages/dashboard/dashboard-i18n.spec.ts` | 2 | 2.5 |
| `pages/dashboard/dashboard-links.spec.ts` | 6 | 2.5 |
| `pages/dashboard/dashboard.component.spec.ts` | 15 | 2.5 |
| `pages/dashboard/services/dashboard.service.spec.ts` | 4 | 2.5 |
| `pages/empleados/empleados-detail/empleados-detail.component.spec.ts` | 5 | 2.6 |
| `pages/empleados/empleados-search/empleados-search.component.spec.ts` | 2 | 2.6 |
| `pages/empleados/services/empleado.service.spec.ts` | 3 | 2.6 |
| `pages/empresa/empresa-register/empresa-register.component.spec.ts` | 8 | 2.5 |
| `pages/ingredientes-activos/ingredientes-activos-detail/ingredientes-activos-detail.delete.spec.ts` | 0 | 2.7 |
| `pages/laboratorios/laboratorios-detail/laboratorios-detail.component.spec.ts` | 3 | 2.7 |
| `pages/laboratorios/laboratorios-search/laboratorios-search.component.spec.ts` | 4 | 2.7 |
| `pages/lotes/lotes-detail/lotes-detail.component.spec.ts` | 4 | 2.8 |
| `pages/lotes/lotes-search/lotes-search.component.spec.ts` | 3 | 2.8 |
| `pages/lotes/services/lote.service.spec.ts` | 7 | 2.8 |
| `pages/medicamentos/medicamentos-detail/medicamentos-detail.component.spec.ts` | 3 | 2.7 |
| `pages/medicamentos/medicamentos-detail/medicamentos-detail.delete.spec.ts` | 0 | 2.7 |
| `pages/medicamentos/medicamentos-search/medicamentos-search.component.spec.ts` | 2 | 2.7 |
| `pages/medicamentos/services/medicamento.service.spec.ts` | 8 | 2.7 |
| `pages/pacientes/pacientes-detail/pacientes-detail.delete.spec.ts` | 0 | 2.8 |
| `pages/pos/dialogs/comprobante-dialog/comprobante-dialog.component.spec.ts` | 11 | 2.9 |
| `pages/pos/dialogs/devolver-venta-dialog/devolver-venta-dialog.component.spec.ts` | 6 | 2.9 |
| `pages/pos/nueva-venta/nueva-venta.component.spec.ts` | 52 | 2.9 |
| `pages/pos/nueva-venta/pos-draft.spec.ts` | 7 | 2.9 |
| `pages/pos/services/receipt-print.service.spec.ts` | 4 | 2.9 |
| `pages/pos/services/venta.service.spec.ts` | 12 | 2.9 |
| `pages/pos/utils/vuelto.util.spec.ts` | 8 | 2.9 |
| `pages/pos/venta-detail/venta-detail.component.spec.ts` | 20 | 2.9 |
| `pages/presentaciones-venta/presentaciones-venta-modal/presentaciones-venta-modal.component.spec.ts` | 3 | 2.7 |
| `pages/profesionales-salud/profesionales-salud-detail/profesionales-salud-detail.delete.spec.ts` | 0 | 2.8 |
| `pages/proveedores/precio-proveedor-dialog/precio-proveedor-dialog.component.spec.ts` | 10 | 2.7 |
| `pages/proveedores/proveedores-detail/proveedores-detail.component.spec.ts` | 9 | 2.7 |
| `pages/proveedores/proveedores-detail/proveedores-detail.delete.spec.ts` | 0 | 2.7 |
| `pages/recetas-medicas/recetas-detail/recetas-detail.component.spec.ts` | 11 | 2.8 |
| `pages/recetas-medicas/recetas-medicas.routes.spec.ts` | 5 | 2.8 |
| `pages/recetas-medicas/recetas-search/recetas-search.component.spec.ts` | 7 | 2.8 |
| `pages/roles/models/menu-preview.spec.ts` | 5 | 2.6 |
| `pages/roles/roles-detail/roles-detail.component.spec.ts` | 16 | 2.6 |
| `pages/roles/roles-i18n.spec.ts` | 2 | 2.6 |
| `pages/roles/roles-search/roles-search.component.spec.ts` | 2 | 2.6 |
| `pages/roles/services/rol.service.spec.ts` | 6 | 2.6 |
| `pages/seguridad/cuentas-detail/cuentas-detail.component.spec.ts` | 18 | 2.6 |
| `pages/series-fiscales/series-fiscales-detail/series-fiscales-detail.component.spec.ts` | 4 | 2.6 |
| `pages/series-fiscales/series-fiscales-search/series-fiscales-search.component.spec.ts` | 3 | 2.6 |
| `pages/series-fiscales/services/serie-fiscal.service.spec.ts` | 6 | 2.6 |
| `pages/sesiones-caja/services/sesion-caja.service.spec.ts` | 2 | 2.9 |
| `pages/sesiones-caja/sesiones-detail/sesiones-detail.component.spec.ts` | 3 | 2.9 |
| `pages/sesiones-caja/sesiones-search/sesiones-search.component.spec.ts` | 3 | 2.9 |
| `pages/stock-por-sucursal/services/stock.service.spec.ts` | 6 | 2.8 |
| `pages/sucursales/services/sucursal.service.spec.ts` | 1 | 2.6 |
| `pages/sucursales/sucursales-detail/sucursales-detail.delete.spec.ts` | 0 | 2.6 |
| `shared/components/carga-masiva-page/carga-masiva-page.component.spec.ts` | 6 | 2.3 |
| `shared/components/datepicker-internationalization/mat-datepicker-intl.component.spec.ts` | 2 | 2.3 |
| `shared/date/sofia-date-adapter.spec.ts` | 11 | 2.3 |
| `shared/directives/permiso.directive.spec.ts` | 10 | 2.3 |
| `shared/utils/date-only.util.spec.ts` | 10 | 2.3 |
| `shared/utils/save-error-key.util.spec.ts` | 3 | 2.3 |
| `shared/utils/tipo-documento.util.spec.ts` | 3 | 2.3 |

Otros elementos del encargo: utilidades de `shared/testing` (5 archivos) → 1.3; rutas de `rutas-spa.csv` (86) → 3.2; componentes de `componentes-pages.txt` (75) → 3.1; pantallas críticas → 3.3; huecos → 4; ejecución, CI y hooks → 5.

---

## 8. Fuentes

- `sofia-web-spa/karma.conf.js`, `angular.json` (líneas 105-137), `package.json`, `tsconfig.spec.json`, `tsconfig.spec.shared.json`, `.gitignore`
- `sofia-web-spa/.husky/pre-commit`, `.husky/commit-msg`, `commitlint.config.js`
- `sofia-web-spa/src/app/shared/testing/*.ts` (5 archivos)
- Los 75 archivos `sofia-web-spa/src/**/*.spec.ts` (títulos de `describe` e `it`; leídos con detalle `nueva-venta.component.spec.ts` (inicio), `venta-detail.component.spec.ts`, `auth.service.spec.ts` (inicio), `route-permissions.spec.ts`, `menu-preview.spec.ts`, `dashboard-i18n.spec.ts`, `roles-i18n.spec.ts`, `app.component.spec.ts`, los 6 `*.delete.spec.ts`)
- `sofia-web-spa/src/app/app.routes.ts`, `app.config.ts`, `core/auth/route-permissions.ts`
- Componentes y servicios de `sofia-web-spa/src/app/pages` y `shared` (lista de archivos y cruce de nombres de método), `sesiones-caja/models/sesion-caja.model.ts`, `sesiones-detail.component.html`, `sesion-caja.service.ts`
- `sofia-apis/.github/workflows/security.yml` (solo para confirmar que no ejecuta pruebas del SPA)
- `docs/_inventario/pruebas-spa.csv`, `rutas-spa.csv`, `componentes-pages.txt`, `dialogos.txt`, `README.md`, `REGLAS-AGENTES.md`
- `docs/ola-1-fundamentos/08-guia-del-desarrollador.md` (secciones 4.2, 5.2, 6)
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19a`, `19b`, `19e`, `19f` (encabezados y 19a §6.7, §6.13, §7.2)

