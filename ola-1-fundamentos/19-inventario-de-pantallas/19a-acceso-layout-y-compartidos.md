# Inventario de pantallas del SPA — Parte A: acceso, layout, dashboard y componentes compartidos

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.1 |
| **Fecha** | 2026-10-14 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código del árbol de trabajo (SPA y, solo para describir qué calcula cada indicador del dashboard y el registro de empresa, los handlers de `SOFIA.Application/Dashboard` y `SOFIA.Application/Empresas`). No se usó el navegador ni la API local. La versión 1.1 reconcilia el documento (escrito sobre `sofia-apis` `00991bc` y `sofia-web-spa` `31af681`) con el código actual; no hay cambios sin commitear en ninguno de los dos repos. |

## Historial de cambios

### Versión 1.1 (2026-10-14)

| # | Apartado o ID | Decía | Dice ahora | Commit |
|---|---|---|---|---|
| 1 | §1.2 Campos del formulario | 8 campos (empresa: nombre y RUC; administrador: 4; cuenta: 2) | 9 campos: se agrega **Dirección de la sede principal** (`direccionSede`), obligatoria, máximo 255 caracteres, no admite solo espacios, con placeholder «Av. Los Olivos 123, Lima» y ayuda «Aparecerá en el encabezado de tus boletas» | `2f82346` (spa), `9dd1705` (apis) |
| 2 | §1.2 Botones (envío) y Llamadas a la API | El cuerpo no incluía la dirección | El cuerpo incluye `direccionSede` (con `trim()`); sigue sin enviar nombre de sede ni licencia. Citas de línea y de `empresa.model.ts` actualizadas | `2f82346` |
| 3 | §1.2 Mensajes al usuario | 3 códigos del backend con mensaje propio | 4: se agrega `RegistrarEmpresaCommand.DireccionSede` → «Ingresa la dirección de la sede principal (máximo 255 caracteres)» (`empresa.notifications.direccionSedeInvalida`) | `2f82346`, `9dd1705` |
| 4 | §1.2 Comportamientos particulares | Sin mención de la sede | La sede principal se crea con la dirección escrita, el nombre «Sede Principal» y el número de licencia vacío (antes la dirección y la licencia se rellenaban con «Por definir» en el backend; la licencia pasa a ser obligatoria recién al editar la sucursal) | `9dd1705`, `41f0a05` (apis) |
| 5 | §1.1 Diseño (versión) | La versión sale de `environment.appVersion`, «sustituible por `window.__env`» sin decir de dónde se carga | `index.html` carga `environment.js` (que define `window.__env`); el `public/environment.js` del repositorio lo deja vacío, por lo que rige el valor compilado. Además `ng build` usa ahora la configuración `production` por defecto (reemplaza `environment.ts` por `environment.prod.ts`) | `8f36125` (spa) |
| 6 | §3.2 Efectivo esperado y §9 «En curso (sin commitear)» | `ArqueoCajaCalculator.cs` tenía cambios locales sin commitear | Ya está commiteado: el efectivo esperado suma las ventas cobradas en la sesión (incluidas las anuladas desde otra caja) y resta el efectivo neto de las ventas de otras cajas anuladas en esta; se retira la marca y la fila de §9 | `358935e` (apis) |
| 7 | §5.4 (saveErrorKey), §6.12 (usos), §7.1 (`*appPermiso`), §8.4 (`general.errors.required`) | 34 referencias a `saveErrorKey`; 153 `showError`; 68 `showSuccess`; 84 usos de `*appPermiso`; `general.errors.required` en 20 archivos | 36 referencias; 155; 69; 85 usos en 47 plantillas; 21 archivos (recuento nuevo por el diálogo `precio-proveedor-dialog` y el historial de precios del detalle de proveedores, recontado con el mismo método) | `606cb69`, `9cbbe58`, `27597cd`, `01b8260` |
| 8 | §6.3 Dónde se usa | Gerente de `sucursales-detail` «solo en edición» | Gerente en crear y en editar (el buscador se dibuja con `isEditMode()`, verdadero también al crear); corrección de una afirmación que ya era imprecisa, no un cambio de código | — |
| 9 | §10 y §11 | Cobertura y fuentes sin los archivos nuevos | Se agregan `index.html`, `environment.js`, `angular.json` y los archivos del backend de registro y arqueo | `8f36125`, `2f82346`, `9dd1705`, `41f0a05`, `358935e` |

Este documento es la parte A de seis del inventario de pantallas. Cubre las pantallas públicas, el cambio de contraseña, el dashboard, el layout de la aplicación, guards e interceptores, los componentes y directivas compartidos y los textos comunes de la raíz de `es.json`. Las demás partes remiten aquí para todo lo compartido.

Convenciones: las rutas de archivo son relativas a `sofia-web-spa/` salvo que empiecen por `sofia-apis/`. Entre paréntesis va la clave i18n; las claves de scope se escriben con su prefijo real (por ejemplo `auth.title` está en `public/i18n/auth/es.json`, clave `title`). El idioma por defecto es español (`es`); existe también `en` (ver §4.5).

## Contenido

1. Pantallas públicas: `/auth/login`, `/auth/register`, `/auth/forgot-password`, `/auth/reset-password`, panel de marca
2. `/cambiar-clave`
3. `/dashboard`
4. Layout de la aplicación (cabecera, menú lateral, cierre de sesión, idioma, tema, pie)
5. Guards e interceptores vistos desde el usuario
6. Componentes compartidos y `ErrorNotifierService`
7. Directivas compartidas y `*appPermiso`
8. Textos comunes de la raíz de `es.json`
9. Pendientes y «No verificado»
10. Tabla de cobertura
11. Fuentes

---

## 1. Pantallas públicas

Las cuatro rutas `/auth/*` están fuera del layout: se cargan directamente con `loadComponent`, sin `authGuard` ni guard de permiso (`src/app/app.routes.ts:10-31`). No muestran cabecera ni menú. El componente raíz solo contiene `<router-outlet>` (`src/app/app.component.html:1`). Ninguna de las cuatro redirige al dashboard si ya hay sesión iniciada: el código de los componentes no lo comprueba.

### 1.1 `/auth/login` — Iniciar sesión

- **Componente:** `LoginComponent` (`src/app/pages/auth/login/login.component.ts`, `login.component.html`, `login.component.scss`). Scope i18n `auth` con `read: 'auth'` (`login.component.html:1`).
- **Para qué sirve:** autenticar con usuario y contraseña; si el servidor responde bien guarda el token de acceso y entra al sistema.
- **Cómo se llega:** URL directa; redirección del `authGuard` cuando no hay sesión (§5.1); redirección tras cerrar sesión (§4.4), tras cambiar contraseña (§2) y tras restablecerla (§1.4); enlace «Iniciar sesión» del registro (§1.2) y «Volver a iniciar sesión» de recuperación (§1.3, §1.4).
- **Permiso de ruta:** ninguno (pública, `src/app/app.routes.ts:10-14`).
- **Migas y cabecera:** no tiene migas. Título de la tarjeta «Iniciar sesión» (`auth.title`), subtítulo «Ingresa tus credenciales para acceder al sistema» (`auth.subtitle`).
- **Diseño:** panel izquierdo de marca (logo `⚕ SOFIA`, «Gestión completa de tu farmacia» (`auth.panelTitle`), «Sistema Optimizado Farmacéutico con Inteligencia Artificial» (`auth.system`), cuatro viñetas: «Control de medicamentos y lotes» (`auth.features.inventory`), «Gestión de recetas y pacientes» (`auth.features.patients`), «Reportes DIGEMID automatizados» (`auth.features.digemid`), «Seguridad y auditoría completa» (`auth.features.security`) y pie «© {año} SOFIA · v{versión}») y panel derecho con la tarjeta (`login.component.html:3-37`). El panel de marca de esta página está escrito en línea (no usa `app-auth-brand-panel`, ver §1.5). El panel izquierdo se oculta con ancho de pantalla ≤ 768 px (`login.component.scss:263-266`). La versión sale de `environment.appVersion` (`login.component.ts:55`; valor compilado `1.0.0` en `src/environments/environment.ts:5` y en `environment.prod.ts:5`; `environment.model.ts:9-31` usa `window.__env` si trae `production` booleano y `api.baseurl` no vacío, y si no los valores compilados). Desde `8f36125`, `src/index.html:18` carga `environment.js` antes de la aplicación; el `public/environment.js` del repositorio solo define `window.__env = {}`, que no cumple esa condición, así que rige el valor compilado salvo que el despliegue sustituya ese archivo. `ng build` usa ahora la configuración `production` por defecto (`angular.json:95`), que reemplaza `environment.ts` por `environment.prod.ts` (`angular.json:63-68`); `ng serve` conserva `development` por defecto (`angular.json:107`).
- **Modos:** uno solo.

**Campos del formulario**

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error | Ayuda / placeholder |
|---|---|---|---|---|---|---|
| Usuario (`auth.usuario`) | `usuario` | `matInput` texto, icono `person_outline`, `autocomplete="username"` | Sí (`Validators.required`, `login.component.ts:59`) | Solo obligatorio; sin longitud máxima | «El usuario es obligatorio» (`auth.errors.usuarioRequired`), se muestra si está tocado (`login.component.html:65-70`) | «Ej. jperez» (`auth.usuarioPlaceholder`) |
| Contraseña (`auth.password`) | `password` | `matInput` tipo `password`/`text`, icono `lock_outline`, `autocomplete="current-password"` | Sí (`login.component.ts:60`) | Solo obligatorio | «La contraseña es obligatoria» (`auth.errors.passwordRequired`), si está tocado (`login.component.html:93-98`) | «••••••••» (`auth.passwordPlaceholder`) |

**Botones y acciones**

| Texto (clave) | Dónde está | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Icono ojo (`visibility`/`visibility_off`), `aria-label` «Contraseña» (`auth.password`) | Sufijo del campo contraseña | Siempre | Nunca | `togglePassword()` alterna mostrar/ocultar (`login.component.ts:84-86`) |
| Ingresar (`auth.submit`); durante el envío muestra un spinner en lugar del texto | Botón principal del formulario (`type="submit"`) | Siempre | `submitting()` verdadero (`login.component.html:107`) | `submit()` (`login.component.ts:63-82`): limpia el error; si el formulario es inválido marca todo como tocado y no envía; si es válido llama `AuthService.login`. Al responder bien navega a `/cambiar-clave` si el token trae `pwd_change = "true"` (`auth.requiresPasswordChange()`), si no a `/dashboard` (`:75`) |
| ¿Olvidaste tu contraseña? (`auth.forgot`) | Enlace bajo el botón | Siempre | — | Navega a `/auth/forgot-password` |
| Registrar empresa nueva (`auth.register`), precedido de «¿Primera vez?» (`auth.registerPrompt`) y del separador «o» (`auth.or`) | Pie de la tarjeta | Siempre | — | Navega a `/auth/register` |

**Mensajes al usuario** (alerta roja en la tarjeta con icono `error_outline`, `login.component.html:48-53`; no hay snackbar)

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| La API no responde (estado 0), tiempo agotado (408), error de servidor (≥ 500) o el error no es una respuesta HTTP | Error | «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.» (`auth.serviceUnavailable`) | `login.component.ts:20-23` |
| Código de respuesta `Auth.EmpresaNoVigente` | Error | «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» (`auth.empresaNoVigente`) | `login.component.ts:15-17,25` |
| Estado 429 (límite de peticiones) | Error | «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» (`auth.recovery.tooManyRequests`) | `login.component.ts:26` |
| Cualquier otra respuesta de error (por ejemplo 400/401) | Error | «Credenciales inválidas. Verifica tu usuario y contraseña.» (`auth.invalidCredentials`) | `login.component.ts:26` |

**Llamadas a la API**

| Cuándo | Método del servicio | Verbo y URL | Permiso del endpoint |
|---|---|---|---|
| Enviar el formulario | `AuthService.login` (`src/app/core/auth/auth.service.ts:73-81`), con `withCredentials` (cookie de refresco) | `POST /api/v1/auth/login` con `{ nombreUsuario, password }` | Anónimo, política de límite `AuthPolicy` (`docs/_inventario/endpoints.csv`; `AuthController.cs:18`) |

**Comportamientos particulares**
- Al iniciar sesión bien: guarda el token en `localStorage` bajo la clave `sofia_token` (`auth.service.ts:33,147-152`) y borra los borradores de POS de otros usuarios (`clearOtherUsersStorage`, `auth.service.ts:78`; prefijos en `user-storage.ts:2-5`).
- Si ya tenías sesión de otro usuario, `storeToken` reinicia los permisos en memoria cuando cambia el usuario (`auth.service.ts:151`).
- No hay «recordarme» ni parámetro de retorno (`returnUrl`): siempre se entra a `/dashboard` o `/cambiar-clave`.

**Observaciones**
- El scope i18n `login` (`public/i18n/login/es.json`, 8 claves) no lo referencia ningún archivo de `src/` (búsqueda de `scope: 'login'` y `provideTranslocoScope('login')` sin resultados): está sin uso; el login usa el scope `auth`.

### 1.2 `/auth/register` — Registrar Empresa

- **Componente:** `EmpresaRegisterComponent` (`src/app/pages/empresa/empresa-register/empresa-register.component.ts`, `.html`, `.scss`). Scope `empresa` con `read: 'empresa'` (`.html:1`).
- **Para qué sirve:** registro público de una empresa nueva y de su administrador. «Crea tu empresa y obtén 1 mes de prueba gratis» (`empresa.register.subtitle`).
- **Cómo se llega:** enlace «Registrar empresa nueva» del login; URL directa.
- **Permiso de ruta:** ninguno (pública, `app.routes.ts:15-21`).
- **Migas y cabecera:** sin migas. Título «Registrar Empresa» (`empresa.register.title`).
- **Diseño:** panel de marca propio (distinto del de login): «Gestión farmacéutica completa» (`empresa.register.brand.headline`), «Una plataforma inteligente para administrar tu farmacia con precisión y eficiencia.» (`...brand.sub`), cuatro viñetas «✓»: «1 mes de prueba gratuita, sin tarjeta» (`...brand.features.trial`), «Multi-sucursal desde el primer día» (`...multiBranch`), «Control de inventario y ventas en tiempo real» (`...realTime`), «Soporte dedicado durante la incorporación» (`...support`); pie «© {año} SOFIA · Todos los derechos reservados» (`...brand.rights`). Se pasa a una columna con ≤ 900 px (`.scss:10-12`).
- **Modos:** uno solo.

**Campos del formulario** (`empresa-register.component.ts:57-67`; todos con `nonNullable`; 9 campos)

| Etiqueta (clave `empresa.register.*`) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error (clave `empresa.form.*`) |
|---|---|---|---|---|---|
| Sección «Datos de la empresa» (`sections.empresa`) | | | | | |
| Nombre de la Empresa (`nombreEmpresa`) | `nombreEmpresa` | Texto, icono `business`, `autocomplete="organization"` | Sí | `required`, `maxLength(200)` | «Este campo es obligatorio» (`required`); «Máximo 200 caracteres» (`maxLength200`) |
| RUC (`ruc`) | `ruc` | Texto, icono `badge`, `maxlength="11"`, `inputmode="numeric"` | Sí | `required`, patrón `^\d{11}$` | «Este campo es obligatorio»; «El RUC debe tener exactamente 11 dígitos numéricos» (`rucFormat`) |
| Dirección de la sede principal (`direccionSede`) | `direccionSede` | Texto, icono `location_on`, `autocomplete="street-address"`, sin atributo `maxlength` (el límite se valida, no se impide escribir). Placeholder «Av. Los Olivos 123, Lima» (`register.direccionSedePlaceholder`). Ayuda bajo el campo: «Aparecerá en el encabezado de tus boletas» (`register.direccionSedeHint`) (`.html:79-97`) | Sí | `required`, `maxLength(255)` (constante `ADDRESS_MAX_LENGTH`, `.ts:19`), patrón `/\S/` que rechaza un texto de solo espacios (`.ts:60`) | «Este campo es obligatorio» (`required`), tanto si está vacío como si solo tiene espacios (falla `required` o el patrón, `.html:88-93`); «Máximo 255 caracteres» (`maxLength255`, `.html:94-96`) |
| Sección «Datos del administrador» (`sections.admin`) | | | | | |
| Nombres del administrador (`adminNombres`) | `adminNombres` | Texto, icono `person`, `autocomplete="given-name"` | Sí | `required`, `maxLength(75)` | «Este campo es obligatorio»; «Máximo 75 caracteres» (`maxLength75`) |
| Apellido paterno (`adminApellidoPaterno`) | `adminApellidoPaterno` | Texto, `autocomplete="family-name"` | Sí | `required`, `maxLength(75)` | Igual que el anterior |
| Apellido materno (opcional) (`adminApellidoMaterno`) | `adminApellidoMaterno` | Texto | No | `maxLength(75)` | «Máximo 75 caracteres» |
| Correo electrónico (`adminEmail`) | `adminEmail` | `type="email"`, icono `mail`, `autocomplete="email"` | Sí | `required`, `email`, `maxLength(254)` | «Este campo es obligatorio»; «Ingresa un correo electrónico válido» (`emailFormat`); «Máximo 254 caracteres» (`maxLength254`). Ayuda: «Lo usarás para recuperar tu contraseña» (`register.adminEmailHint`) |
| Sección «Cuenta de acceso» (`sections.cuenta`) | | | | | |
| Usuario Administrador (`usuario`) | `usuario` | Texto, icono `account_circle`, `autocomplete="username"` | Sí | `required`, `minLength(4)`, `maxLength(50)` | «Este campo es obligatorio»; «El usuario debe tener al menos 4 caracteres» (`usuarioMin`); «Máximo 50 caracteres» (`maxLength50`) |
| Contraseña (`password`) | `password` | Texto/contraseña, icono `lock`, `autocomplete="new-password"`, botón ojo con `aria-label` «Mostrar u ocultar contraseña» (`register.togglePassword`) | Sí | `required`, `minLength(8)` | «Este campo es obligatorio»; «La contraseña debe tener al menos 8 caracteres» (`passwordMin`) |

Los mensajes de error se pintan solo cuando el control está inválido y tocado (comportamiento por defecto de `mat-error`); el `submit()` marca todo como tocado.

**Botones y acciones**

| Texto (clave) | Dónde está | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|---|
| Registrar Empresa (`empresa.register.submit`); durante el envío spinner + «Procesando...» (`register.processing`) | Fin del formulario (`type="submit"`) | Siempre | `submitting()` | `submit()` (`.ts:69-100`): si es inválido marca todo tocado y no envía; si no, envía los textos con `trim()` (incluida `direccionSede`, `.ts:80`; apellido materno vacío se envía `null`; usuario y contraseña sin `trim`) a `EmpresaService.register` |
| Icono ojo | Sufijo de contraseña | Siempre | Nunca | `togglePasswordVisibility()` |
| Iniciar sesión (`register.login`), precedido de «¿Ya tienes cuenta?» (`register.hasAccount`) | Pie del formulario | Siempre | — | Navega a `/auth/login` |

**Mensajes al usuario** (snackbar vía `ErrorNotifierService`, §6.12)

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Registro correcto (guarda el token y va a `/dashboard`) | Éxito (3 s) | «Empresa registrada correctamente. ¡Bienvenido!» (`empresa.notifications.registerSuccess`) | `.ts:88-92` |
| Código `Empresa.RUC.Duplicado` | Error (5 s) | «Ya existe una empresa registrada con este RUC» (`empresa.notifications.rucDuplicado`) | `.ts:22-26,95-97` |
| Código `Auth.DuplicateUsername` | Error | «El usuario ya está en uso. Elige otro.» (`empresa.notifications.usuarioDuplicado`) | `.ts:24` |
| Código `RegistrarEmpresaCommand.DireccionSede` (la API rechaza la dirección de la sede: vacía o de más de 255 caracteres; el formulario normalmente lo impide antes) | Error | «Ingresa la dirección de la sede principal (máximo 255 caracteres)» (`empresa.notifications.direccionSedeInvalida`) | `.ts:25,95-97` |
| Cualquier otro error (incluye 429, 5xx, sin red) | Error | «Error al registrar la empresa» (`empresa.notifications.registerError`) | `.ts:97` |

El código de validación lo arma `ValidationBehavior` con la **primera** falla del validador (`{NombreDelRequest}.{Propiedad}`, `sofia-apis/src/SOFIA.Application/Common/Behaviors/ValidationBehavior.cs:30`); por eso el mensaje de la dirección solo aparece si la dirección es la primera propiedad inválida según el orden de reglas del validador (`RegistrarEmpresaCommandValidator.cs:49-51` es la última regla). Las demás validaciones de la API (nombre, RUC, usuario, contraseña, datos del administrador) no tienen mensaje propio en esta pantalla y muestran «Error al registrar la empresa».

**Llamadas a la API**

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Enviar | `EmpresaService.register` (`src/app/pages/empresa/services/empresa.service.ts:39-42`, `withCredentials`) | `POST /api/v1/empresas/registrar` con `RegistrarEmpresaRequest` (`empresa.model.ts:34-44`: `nombreEmpresa`, `ruc`, `adminNombres`, `adminApellidoPaterno`, `adminApellidoMaterno`, `adminEmail`, `direccionSede`, `usuario`, `password`); respuesta `{ token }` | Anónimo, política `SignUpPolicy` (`EmpresasController.cs:21`) |

**Comportamientos particulares:** al registrar se guarda el token (`AuthService.storeToken`) y se entra directo a `/dashboard`; no hay pantalla de confirmación ni verificación de correo en el SPA.

**La sede principal que crea el registro** (backend, `sofia-apis/src/SOFIA.Application/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandHandler.cs:85-91`): la pantalla no envía nombre de sede ni licencia, así que el backend crea la sucursal con el nombre «Sede Principal», la dirección escrita (con `Trim()`) y el número de licencia **vacío** (`Sucursal.CreateSedePrincipal`, `sofia-apis/src/SOFIA.Domain/Entities/Sucursal.cs:56-76`). Antes de `9dd1705` y `41f0a05` la dirección y la licencia se rellenaban con el texto «Por definir». El texto de ayuda del campo indica que la dirección aparecerá en el encabezado de las boletas (`register.direccionSedeHint`); el resto de la cadena de impresión no se verificó aquí. La licencia pasa a ser obligatoria cuando se edita esa sucursal (ver [19b §2.2](19b-administracion.md)). La API también acepta `NombreSede` y `NumeroLicencia` opcionales (`RegistrarEmpresaCommand.cs`), que este formulario no usa. Por contrato de la API la dirección es obligatoria (`RegistrarEmpresaCommandValidator.cs:49-51`: no vacía y hasta 255 caracteres).

**Observaciones:** el formulario tiene `autocomplete="off"` y no `novalidate` (`.html:45`). El panel de marca de esta pantalla no reutiliza `app-auth-brand-panel`.

### 1.3 `/auth/forgot-password` — Recupera tu contraseña

- **Componente:** `ForgotPasswordComponent` (`src/app/pages/auth/forgot-password/forgot-password.component.ts`, `.html`; estilos `login.component.scss` y `auth-recovery.scss`). Scope `auth`, `read: 'auth'`.
- **Para qué sirve:** pedir el enlace de restablecimiento al correo del empleado asociado al usuario.
- **Cómo se llega:** enlace «¿Olvidaste tu contraseña?» del login; enlace «Solicitar un nuevo enlace» de §1.4.
- **Permiso de ruta:** ninguno (`app.routes.ts:23-26`).
- **Migas y cabecera:** sin migas. Título «Recupera tu contraseña» (`auth.forgotPage.title`); subtítulo «Ingresa tu usuario y te enviaremos un enlace al correo registrado de tu empleado para crear una nueva contraseña.» (`auth.forgotPage.subtitle`). Usa el panel de marca compartido (§1.5).

**Campos**

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error |
|---|---|---|---|---|---|
| Usuario (`auth.usuario`) | `usuario` | Texto, icono `person_outline`, `maxlength="50"`, `autocomplete="username"` | Sí | `required`, `maxLength(50)` (`.ts:41-43`) | «El usuario es obligatorio» (`auth.errors.usuarioRequired`) si está tocado |

**Botones y acciones**

| Texto (clave) | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|
| Enviar enlace (`auth.forgotPage.submit`); spinner mientras envía | Mientras no se haya enviado (`sent()` falso) | `submitting()` | `submit()` (`.ts:45-65`): valida y envía el usuario con `trim()` |
| Volver a iniciar sesión (`auth.recovery.backToLogin`) | Siempre | — | Navega a `/auth/login` |

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Solicitud aceptada (la misma respuesta exista o no el usuario; el formulario se reemplaza por este aviso con icono `mark_email_read`) | Éxito (alerta verde) | «Si el usuario existe y tiene un correo registrado, recibirás un enlace para restablecer tu contraseña. Revisa tu bandeja de entrada: el enlace es válido por 1 hora.» (`auth.forgotPage.sent`) | `.html:18-22` |
| Estado 429 | Error (alerta roja) | «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» (`auth.recovery.tooManyRequests`) | `.ts:61-62` |
| Cualquier otro error | Error | «No se pudo procesar la solicitud. Inténtalo de nuevo.» (`auth.forgotPage.error`) | `.ts:62` |

**Llamadas a la API:** `AuthService.forgotPassword` (`auth.service.ts:105-107`, sin cookies) → `POST /api/v1/auth/forgot-password` con `{ nombreUsuario }`; anónimo, política `AuthPolicy` (`AuthController.cs:104`).

**Observaciones:** el texto «válido por 1 hora» está dentro de la traducción; su correspondencia con la vigencia real del token no se verificó en el backend (ver §9).

### 1.4 `/auth/reset-password` — Crea una nueva contraseña

- **Componente:** `ResetPasswordComponent` (`src/app/pages/auth/reset-password/reset-password.component.ts`, `.html`). Scope `auth`.
- **Para qué sirve:** fijar la contraseña nueva usando el enlace recibido por correo.
- **Cómo se llega:** solo por el enlace del correo. El token y el usuario viajan en el **fragmento** de la URL (`#token=...&usuario=...`); al abrir la pantalla se leen y se reemplaza la URL por `/auth/reset-password` sin fragmento (`.ts:73-80`). Si abres la ruta sin fragmento válido, ves el aviso de enlace inválido.
- **Permiso de ruta:** ninguno (`app.routes.ts:28-31`).
- **Migas y cabecera:** sin migas. Título «Crea una nueva contraseña» (`auth.resetPage.title`); subtítulo «Elige una contraseña que no hayas usado antes. Al guardarla se cerrarán todas tus sesiones abiertas.» (`auth.resetPage.subtitle`). Usa el panel de marca compartido.
- **Modos:** dos estados según `linkValid()` (token y usuario presentes, `.ts:79`): enlace inválido (solo aviso) o formulario.

**Campos** (`.ts:65-71`)

| Etiqueta (clave `auth.resetPage.*`) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error |
|---|---|---|---|---|---|
| Nueva contraseña (`newPassword`) | `newPassword` | Contraseña con ojo (`aria-label` «Mostrar u ocultar contraseñas», `togglePasswords`), `autocomplete="new-password"` | Sí | `required`, `minLength(8)` (`MIN_PASSWORD_LENGTH` importado de `cambiar-clave.component.ts:12`) | «La contraseña es obligatoria» (`auth.errors.passwordRequired`) si tocado; «Debe tener al menos 8 caracteres» (`resetPage.minLength`, con `{{min}}`) |
| Confirmar nueva contraseña (`confirmPassword`) | `confirmPassword` | Contraseña (el ojo del campo anterior controla ambos) | Sí | `required`; validador de grupo `mismatch` si no coincide con la nueva (`.ts:24-27`) | «La contraseña es obligatoria»; «Las contraseñas no coinciden» (`resetPage.mismatch`), bajo el campo, si la confirmación está tocada |

**Botones y acciones**

| Texto (clave) | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|
| Guardar contraseña (`resetPage.submit`) | Enlace válido | `submitting()` | `submit()` (`.ts:86-109`): valida y envía `{ nombreUsuario, token, newPassword }`. Éxito: `clearLocalSession()`, snackbar y navega a `/auth/login` |
| Solicitar un nuevo enlace (`resetPage.requestNewLink`) | Enlace inválido | — | Navega a `/auth/forgot-password` |
| Volver a iniciar sesión (`recovery.backToLogin`) | Siempre | — | Navega a `/auth/login` |

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Enlace sin token o sin usuario (alerta roja, icono `link_off`) | Error | «Este enlace de recuperación está incompleto o ya fue usado. Solicita uno nuevo.» (`auth.resetPage.invalidLink`) | `.html:11-18` |
| Restablecimiento correcto | Éxito snackbar (6 s) | «Contraseña restablecida. Inicia sesión con tu nueva contraseña.» (`auth.resetPage.success`) | `.ts:95-99` |
| Código `Auth.InvalidToken` | Error (alerta) | «El enlace no es válido o ya expiró. Solicita uno nuevo.» (`auth.resetPage.invalidToken`) | `.ts:20-22,105` |
| Estado 429 | Error | «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» (`auth.recovery.tooManyRequests`) | `.ts:104,106` |
| Otro error | Error | «No se pudo restablecer la contraseña. Inténtalo de nuevo.» (`auth.resetPage.error`) | `.ts:106` |

**Llamadas a la API:** `AuthService.resetPassword` (`auth.service.ts:109-111`) → `POST /api/v1/auth/reset-password`; anónimo, `AuthPolicy` (`AuthController.cs:113`).

**Observaciones:** el nombre de usuario no se muestra en pantalla; viaja oculto desde el fragmento.

### 1.5 `app-auth-brand-panel` — panel lateral de marca

- **Componente:** `AuthBrandPanelComponent` (`src/app/pages/auth/auth-brand-panel/auth-brand-panel.component.ts`, `.html`); reutiliza los estilos del login (`styleUrls: ['../login/login.component.scss']`).
- **Dónde se usa:** solo en `/auth/forgot-password` y `/auth/reset-password` (`forgot-password.component.html:2`, `reset-password.component.html:2`). **No** lo usan `/auth/login` (tiene el panel copiado en línea, `login.component.html:3-37`) ni `/auth/register` (panel propio).
- **Contenido:** idéntico al del login: logo `⚕ SOFIA`, `auth.panelTitle`, `auth.system`, las cuatro viñetas `auth.features.*` y el pie «© {año} SOFIA · v{versión}» (`auth-brand-panel.component.html:1-33`). Sin interacción. Se oculta con ≤ 768 px.

---

## 2. `/cambiar-clave` — Cambiar contraseña

- **Componente:** `CambiarClaveComponent` (`src/app/pages/cambiar-clave/cambiar-clave.component.ts`, `.html`). Scope `cambiar-clave` con `read: 'cambiarClave'` (`.html:1`).
- **Para qué sirve:** cambiar la contraseña propia. Al guardar el servidor cierra todas las sesiones del usuario y debes volver a ingresar.
- **Cómo se llega:** dentro del layout (`app.routes.ts:44-47`). Redirección forzada del `passwordChangeGuard` y del login cuando el token indica cambio obligatorio (§5.2); redirección del interceptor ante `403` con código `Auth.CambioClaveRequerido` (§5.4). Para el acceso voluntario desde el menú de usuario, ver §4.2.
- **Permiso de ruta:** ninguno. `cambiar-clave` no figura en `ROUTE_PERMISSIONS` (`src/app/core/auth/route-permissions.ts:2-25`), por lo que el `permissionGuard` la deja pasar; sí exige sesión (`authGuard`) y pasa por `passwordChangeGuard`.
- **Migas y cabecera:** `csw-page-header` con título «Cambiar contraseña» (`cambiarClave.title`) y migas «Inicio» (`breadcrumbs.home`, enlaza a `/dashboard`) › «Cambiar contraseña» (`breadcrumbs.cambiarClave`). El botón «volver» del encabezado (a `/dashboard`) solo se muestra si **no** es cambio obligatorio (`[showBackButton]="!forced()"`, `.html:6`).
- **Modos:** voluntario o forzado (`forced = auth.requiresPasswordChange`, `.ts:49`). En modo forzado: aparece el aviso «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.» (`cambiarClave.forced`, caja con icono `lock_reset`, `.html:15-22`), se oculta el botón Cancelar y el de volver.

**Campos** (`.ts:58-65`)

| Etiqueta (clave) | `formControlName` | Control | Obligatorio | Validaciones | Mensaje de error |
|---|---|---|---|---|---|
| Contraseña actual (`cambiarClave.fields.currentPassword`) | `currentPassword` | Contraseña, `autocomplete="current-password"` | Sí | `required` | «Este campo es obligatorio» (`general.errors.required`, raíz) |
| Nueva contraseña (`fields.newPassword`) | `newPassword` | Contraseña, `autocomplete="new-password"` | Sí | `required`, `minLength(8)`; validador de grupo `sameAsCurrent` (la nueva no puede ser igual a la actual, `.ts:25`) | «Este campo es obligatorio»; «Debe tener al menos 8 caracteres» (`errors.minLength`, `{{min}}`); «La nueva contraseña debe ser distinta de la actual» (`errors.sameAsCurrent`), bajo el campo si está tocado |
| Confirmar nueva contraseña (`fields.confirmPassword`) | `confirmPassword` | Contraseña con ojo (`aria-label` = «Nueva contraseña») que controla los tres campos | Sí | `required`; validador de grupo `mismatch` (`.ts:26`) | «Este campo es obligatorio»; «Las contraseñas no coinciden» (`errors.mismatch`) |

**Botones y acciones**

| Texto (clave) | Visible cuando | Deshabilitado cuando | Qué hace |
|---|---|---|---|
| Cancelar (`actions.cancel`, raíz) | Solo si no es forzado | Nunca | `cancel()` navega a `/dashboard` |
| Guardar (`actions.save`, raíz); spinner mientras guarda | Siempre | `saving()` | `save()` (`.ts:75-97`): valida y llama `AuthService.changePassword`; si va bien: snackbar de éxito (6 s), `clearLocalSession()` y navega a `/auth/login` |
| Icono ojo | Siempre | — | `togglePasswords()` |

**Mensajes al usuario**

| Situación | Tipo | Texto exacto (clave) | Fuente |
|---|---|---|---|
| Cambio correcto | Éxito (6 s) | «Contraseña actualizada. Inicia sesión con tu nueva contraseña.» (`cambiarClave.success`) | `.ts:86` |
| Código `Auth.ClaveActualIncorrecta` | Error | «La contraseña actual es incorrecta» (`cambiarClave.errors.wrongCurrent`) | `.ts:15-17,93` |
| Otro error | Error | «No se pudo cambiar la contraseña. Inténtalo de nuevo.» (`cambiarClave.errors.save`) | `.ts:94` |

Los errores se muestran con `showServerError`: si la respuesta trae `traceId` aparece el snackbar con código de seguimiento (§6.11); si no, un snackbar de error simple.

**Llamadas a la API:** `AuthService.changePassword` (`auth.service.ts:99-102`) → `POST /api/v1/auth/change-password` con `{ currentPassword, newPassword }`; solo autenticado, política `AuthPolicy` (`AuthController.cs:84`).

**Comportamientos particulares:** mientras el cambio sea obligatorio el `passwordChangeGuard` redirige cualquier otra URL del layout a `/cambiar-clave`; el menú lateral sigue visible pero cualquier opción rebota aquí (§5.2). Tras guardar, el servidor revoca todas las sesiones (comentario en `auth.service.ts:99`).

**Observaciones:** la caja de aviso forzado usa colores fijos en el CSS del componente (`.ts:35`); el texto «Debe tener al menos…» existe duplicado en `auth.resetPage.minLength` y `cambiarClave.errors.minLength`.

---

## 3. `/dashboard` — Inicio (resumen de hoy)

- **Componente:** `DashboardComponent` (`src/app/pages/dashboard/dashboard.component.ts`, `.html`, `.scss`); servicio `services/dashboard.service.ts`; modelos `models/dashboard.model.ts`; enlaces `dashboard-links.ts`. Scope `dashboard` con `read: 'dashboard'` (`.html:1`). Es la única pantalla cargada de forma directa (no perezosa) en `app.routes.ts:39-42`.
- **Para qué sirve:** resumen operativo del día: ventas, caja, inventario, cuarentena y devoluciones, con tarjetas que abren la lista correspondiente ya filtrada.
- **Cómo se llega:** ruta por defecto (`''` redirige a `dashboard`, `app.routes.ts:9`); cualquier ruta desconocida redirige aquí (`**`, `app.routes.ts:178-179`); destino de los rechazos de permiso (§5.3); inicio tras login/registro; migas «Inicio» de otras pantallas. Opción de menú: ver §4.3.
- **Permiso de ruta:** ninguno de ruta (`dashboard` no está en `ROUTE_PERMISSIONS`). Cada bloque se controla por separado con el permiso de lectura del módulo del que sale (`dashboard.component.ts:112-117`).
- **Migas y cabecera:** sin migas ni `csw-page-header`; tiene su propia tarjeta de cabecera (abajo).
- **Selectores y filtros:** **no tiene** selector de sucursal, de día ni de rango de fechas. «Hoy» es el día local del navegador (`inicioDelDia(new Date())`, `dashboard-links.ts:13-15`), que se envía al servidor como instante de inicio del día (`toLocalDayStartIso`). El alcance (una sucursal, varias o toda la empresa) lo decide el servidor según el usuario (§3.2).
- **Refresco:** solo manual con el botón «Actualizar»; el código no tiene temporizador ni sondeo.

### 3.1 Qué muestra cada bloque

**Cabecera** (`dashboard.component.html:4-43`)

| Elemento | Texto (clave `dashboard.*`) | Regla |
|---|---|---|
| Saludo | «Hola, {nombre}» (`header.greeting`) con el `fullName` del token; si no hay nombre, «Bienvenido a SOFIA» (`header.welcome`) | `.html:7-11`, `.ts:110` |
| Subtítulo | «Este es el resumen de hoy.» (`header.subtitle`) | Siempre |
| Hora de actualización | «Actualizado a las {hora}» (`header.updatedAt`, formato `HH:mm`) | Solo tras la primera carga (`actualizado()` no nulo) |
| Botón | «Actualizar» (`header.refresh`), icono `refresh` | Deshabilitado mientras `cargando()` (permisos o cualquier sección cargando). Llama `actualizar()`: recalcula «hoy», fija la hora y recarga las secciones permitidas |
| Error de permisos | «No se pudieron cargar tus permisos.» (`header.permissionsError`) + botón «Reintentar» (`error.retry`) | Si `loadPermissions()` falla; el botón llama `cargarPermisos()` |
| Sin indicadores | «Tu usuario no tiene indicadores disponibles aquí. Usa el menú lateral para ir a tus módulos.» (`header.noSections`) | Cuando los permisos cargaron sin error y el usuario no tiene ninguno de los cinco permisos (`sinSecciones`, `.ts:118-127`) |

**Bloque «Ventas de hoy» (`ventas.title`)** — visible con permiso `Ventas:Leer` (`puedeVentas`). Endpoint `GET /api/v1/dashboard/ventas?hoy=<instante>` (permiso `Ventas:Leer`, `DashboardController.cs:16`). Mientras no hay datos: spinner (`aria-label` «Cargando», `dashboard.loading`).

| Tarjeta / elemento | Texto (clave) | Dato | Visible cuando |
|---|---|---|---|
| Vendido hoy | «Vendido hoy» (`ventas.soldToday`) | `S/ {hoy.monto}` con 2 decimales. Debajo: variación contra ayer con flecha `trending_up`/`trending_down`/`trending_flat`, `±x.x% vs. ayer · Ayer: S/ {ayer.monto}`; si ayer no vendió nada solo muestra «Ayer: S/ 0.00» (`variacionAyer` nulo, `.ts:144-148`) | Siempre en el bloque |
| Botón de la tarjeta | «Ver ventas de hoy» (`ventas.linkToday`) | Abre `/pos` con fecha inicio = fecha fin = hoy y estado Completada | Solo vista de **una** sucursal (`enlacesDeVentas`: `multiSucursal === false`, `.ts:140`) |
| Comprobantes | «Comprobantes» (`ventas.receipts`) | `hoy.comprobantes`; detalle «Ticket promedio: S/ {ticketPromedio}» (`ventas.avgTicket`) | Siempre |
| Mis ventas de hoy | «Mis ventas de hoy» (`ventas.mine`) | `S/ {mias.monto}` y «{count} comprobantes» (`ventas.receiptsCount`) | Solo si el servidor devuelve `mias` (token con empleado) |
| Botón | «Ver mis ventas» (`ventas.linkMine`) | Abre `/pos` con estado Completada, hoy y el empleado propio | `enlacesDeVentas` y existe `empleadoId` del token |
| Ventas en espera | «Ventas en espera» (`ventas.parked`) | `pendientes.cantidad`; detalle «Por cobrar: S/ {pendientes.monto}» (`ventas.parkedAmount`) | Siempre |
| Botón | «Ver ventas en espera» (`ventas.linkParked`) | Abre `/pos` con estado Pendiente, sin fechas | `enlacesDeVentas` |
| Cobrado por método de pago | «Cobrado por método de pago» (`ventas.byMethod`) | Una fila por método: «Efectivo», «Yape / Plin», «Tarjeta», «Transferencia» (`dashboard.metodo.*`), monto `S/` y barra proporcional al total cobrado; nota «El efectivo ya descuenta el vuelto entregado.» (`ventas.byMethodHint`) | Siempre |
| Ventas por sucursal | «Ventas por sucursal» (`ventas.byBranch`) | Tabla con columnas «Sucursal» (`ventas.colBranch`), «Comprobantes» (`ventas.colReceipts`), «Vendido» (`ventas.colSold`) y fila de total: «Total empresa» (`ventas.totalCompany`) si el alcance es toda la empresa, o «Total de tus sucursales» (`ventas.totalBranches`) | Solo vista multisucursal (`v.multiSucursal`) |
| Nota | «Los montos vendidos son brutos: las devoluciones se muestran aparte.» (`ventas.grossHint`) | Texto fijo | Siempre en el bloque |

**Bloque «Caja» (`caja.title`)** — permiso `POS:Leer` (`puedeCaja`). Endpoint `GET /api/v1/dashboard/caja` (`POS:Leer`, `DashboardController.cs:25`).

| Elemento | Texto (clave) | Dato / regla |
|---|---|---|
| Mi caja abierta | «Tienes una caja abierta» (`caja.open`), icono `lock_open` | Si hay `miSesion`: «Sucursal» (`caja.branch`), «Abierta desde» (`caja.since`, `dd/MM/yyyy HH:mm`), «Monto de apertura» (`caja.opening`), «Efectivo esperado» (`caja.expected`) y botón «Ver mi caja» (`caja.linkMine`) que navega a `/sesiones-caja/{id}` |
| Sin caja | «No tienes una caja abierta» (`caja.none`), icono `lock` | Si no hay `miSesion`. Botón «Abrir caja» (`caja.linkOpen`) a `/sesiones-caja/nueva`, visible con `*appPermiso="'POS:AperturarCaja'"` |
| Cajas abiertas | «Cajas abiertas ({n})» (`caja.openSessions`) | Solo si existe alguna sesión abierta distinta de la propia (`otrasSesiones`, `.ts:173-178`); lista **todas** las abiertas del alcance, incluida la propia (el servidor entrega como máximo 50, `GetDashboardCajaQueryHandler.cs:15`). Columnas «Cajero» (`caja.colCashier`), «Sucursal» (`caja.colBranch`), «Abierta desde» (`caja.colSince`). Botón «Ver cajas abiertas» (`caja.linkOpenSessions`): abre `/sesiones-caja` con estado Abierta |

**Bloque «Inventario» (`inventario.title`)** — se muestra si hay `Inventarios:Leer` **o** `DIGEMID:Leer` (`.html:348`). Endpoints `GET /api/v1/dashboard/inventario?hoy=` (`Inventarios:Leer`, `DashboardController.cs:34`) y `GET /api/v1/dashboard/cuarentena` (`DIGEMID:Leer`, `:43`). Cada uno falla y se reintenta por separado.

| Tarjeta | Texto (clave `dashboard.inventario.*`) | Dato / botón |
|---|---|---|
| Vencidos | «Lotes vencidos con stock» (`expired`) | `lotesVencidosConStock`; botón «Ver lotes» (`linkLots`) → `/stock-por-sucursal` con caducidad hasta ayer y «solo con stock» |
| 30 días | «Lotes que vencen en 30 días o menos» (`expiring30`) | `lotesVencen30`; botón «Ver lotes» → caducidad de hoy a hoy+30 |
| 60 días | «Lotes que vencen en 31 a 60 días» (`expiring60`) | `lotesVencen60`; botón → hoy+31 a hoy+60 |
| 90 días | «Lotes que vencen en 61 a 90 días» (`expiring90`) | `lotesVencen90`; botón → hoy+61 a hoy+90 |
| Sin stock | «Productos sin stock» (`outOfStock`) | `productosSinStock`; detalle «Sin unidades vendibles, por sucursal» (`perBranchHint`); sin botón |
| Stock bajo | «Productos con stock bajo» (`lowStock`) | `productosStockBajo`; detalle «{umbral} unidades o menos, por sucursal» (`lowStockHint`) con el umbral que envía el servidor (hoy 10, constante `UmbralStockBajo = 10m` en `GetDashboardInventarioQueryHandler.cs:14`); sin botón |
| Cuarentena | «Lotes en cuarentena» (`quarantine`) | `lotesRetenidos`; detalle «Retenidos: no se pueden vender» (`quarantineHint`); requiere `DIGEMID:Leer`; sin botón |

**Bloque de tendencia y más vendidos** — visible con `Ventas:Leer` y datos cargados (`.html:476-541`); no tiene encabezado de sección, comparte los datos de «Ventas de hoy».

| Elemento | Texto (clave) | Dato |
|---|---|---|
| Tendencia | «Ventas de los últimos 7 días» (`tendencia.title`) | 7 barras (6 días previos + hoy), monto `S/` sobre cada barra y fecha `dd/MM` debajo; la última dice «Hoy» (`tendencia.today`); cada barra tiene `aria-label` «{dia}: S/ {monto} en {comprobantes} comprobantes» (`tendencia.barAria`) |
| Más vendidos | «Productos más vendidos en 7 días» (`top.title`) | Hasta 5 productos (constante `CantidadTopProductos = 5`, `GetDashboardVentasQueryHandler.cs:16`) con `{unidades} unidades · S/ {monto}` (`top.units`) y barra proporcional; vacío: «Aún no hay ventas en los últimos 7 días.» (`top.empty`) |

**Bloque «Devoluciones de hoy» (`devoluciones.title`)** — permiso `Devoluciones:Leer`. Endpoint `GET /api/v1/dashboard/devoluciones?hoy=` (`Devoluciones:Leer`, `DashboardController.cs:52`). Una tarjeta: «Devoluciones» (`devoluciones.count`) con `cantidad` y «Monto reembolsado: S/ {montoReembolsado}» (`devoluciones.refunded`).

### 3.2 Diferencias por permiso, rol y alcance

- **Permisos:** cada bloque exige su permiso de lectura: Ventas → `Ventas:Leer`; Caja → `POS:Leer`; Inventario → `Inventarios:Leer`; Cuarentena → `DIGEMID:Leer`; Devoluciones → `Devoluciones:Leer`. Un bloque sin permiso no se pide ni se dibuja (`cargar()` retorna si no está permitido, `.ts:264-266`). El permiso lo resuelve `AuthService.hasPermission` (el rol `Admin` lo concede todo, `route-permissions.ts:45-48`).
- **Alcance de los datos (lo decide el servidor, `sofia-apis/src/SOFIA.Application/Dashboard/Common/DashboardScopeResolver.cs:19-31`):** si el usuario no tiene restricción de sucursal (rol Admin) el alcance es **toda la empresa** (`empresaCompleta = true`); si no, las sucursales a las que tiene acceso. Con más de una sucursal (`multiSucursal`) aparece la tabla «Ventas por sucursal» y se **ocultan** los botones «Ver ventas de hoy», «Ver mis ventas» y «Ver ventas en espera» (la lista de ventas solo cubre la sucursal propia, comentario en `.ts:139`). Con una sola sucursal, la vista es de esa sucursal y los botones aparecen.
- **Mis ventas / Mi caja:** dependen de que el token traiga el id de empleado; si no, `mias` y `miSesion` llegan nulos y no se muestran (`GetDashboardVentasQueryHandler.cs:87-92`, `GetDashboardCajaQueryHandler.cs:42-49`).
- **Qué cuenta como «vendido»** (servidor): ventas de las sucursales del alcance que no estén anuladas ni pendientes, monto bruto (`GetDashboardVentasQueryHandler.cs:25-28`); las pendientes se cuentan aparte, de cualquier día (`:41-44`); el desglose por método descuenta vuelto y cobertura de seguro (`:101-130`).
- **Efectivo esperado:** lo calcula `ArqueoCajaCalculator.CalcularEfectivoEsperadoAsync` (`GetDashboardCajaQueryHandler.cs:69`) como apertura + efectivo neto de las ventas cobradas en la sesión − efectivo neto de las ventas de otras cajas que se anularon desde esta sesión − reembolsos en efectivo de devoluciones de la sesión (`sofia-apis/src/SOFIA.Application/POS/Common/ArqueoCajaCalculator.cs:10-53`). «Efectivo neto» es el efectivo pagado menos el vuelto. Una venta de esta sesión anulada desde esta misma sesión no cuenta; una venta de esta sesión anulada desde otra caja sigue contando (`ArqueoCajaCalculator.cs:17-21,50`). Comportamiento commiteado en `358935e` (antes del commit, el cálculo ignoraba las ventas anuladas de cualquier caja).

### 3.3 Estados de carga, vacío y error

| Situación | Qué ve el usuario | Texto (clave) | Fuente |
|---|---|---|---|
| Cargando permisos o una sección sin datos aún | Spinner centrado en la tarjeta | `aria-label` «Cargando» (`dashboard.loading`) | `.html` (rama `else` de cada sección) |
| Recarga con datos previos | Se conservan los datos anteriores mientras recarga; el botón «Actualizar» queda deshabilitado | — | `.ts:35,266` |
| Error en una sección | Tarjeta con icono `error_outline`, texto y botón de reintento solo de esa sección; el resto del dashboard sigue | «No se pudo cargar esta sección.» (`error.section`) + «Reintentar» (`error.retry`) | `.html:49-56` y equivalentes |
| Error de permisos | Aviso en la cabecera con «Reintentar» | «No se pudieron cargar tus permisos.» (`header.permissionsError`) | `.html:31-39` |
| Sin ningún indicador permitido | Línea de ayuda en la cabecera | `header.noSections` | `.html:40-42` |
| Sin productos vendidos | Texto en la tarjeta de más vendidos | `top.empty` | `.html:533-535` |
| Cifras en cero | Se muestran como `0` / `S/ 0.00`; no hay texto de «sin datos» | — | — |

**Llamadas a la API** (todas en `DashboardService`, `dashboard.service.ts:19-37`)

| Cuándo | Método | Verbo y URL | Permiso |
|---|---|---|---|
| Al abrir (tras cargar permisos) y en «Actualizar» | `ventas(hoy)` | `GET /api/v1/dashboard/ventas?hoy=` | `Ventas:Leer` |
| Idem | `caja()` | `GET /api/v1/dashboard/caja` | `POS:Leer` |
| Idem | `inventario(hoy)` | `GET /api/v1/dashboard/inventario?hoy=` | `Inventarios:Leer` |
| Idem | `cuarentena()` | `GET /api/v1/dashboard/cuarentena` | `DIGEMID:Leer` |
| Idem | `devoluciones(hoy)` | `GET /api/v1/dashboard/devoluciones?hoy=` | `Devoluciones:Leer` |
| Al abrir (si no estaban cargados) | `AuthService.loadPermissions` | `GET /api/v1/auth/me` | Solo autenticado |

**Comportamientos particulares**
- Las tarjetas con botón guardan los filtros en `SearchStateService.save(path, state)` y navegan; la pantalla destino restaura y ejecuta esos filtros al abrirse (`dashboard.component.ts:231-234`, `dashboard-links.ts`). Destinos: `/pos` (ventas), `/sesiones-caja`, `/stock-por-sucursal` (siempre con «solo con stock» activado y página 0 de tamaño 10).
- El servidor rechaza un «hoy» a más de 36 horas de la hora actual (`DashboardDias.ToleranciaHoyHoras = 36`, `sofia-apis/src/SOFIA.Application/Dashboard/Common/DashboardDias.cs:10`).
- Formato de cifras: los pipes `number: '1.2-2'` y `date` usan el `LOCALE_ID` por defecto de Angular (`en-US`): no hay proveedor de `LOCALE_ID` en `src/` (solo `registerLocaleData(es-PE)` en `provide-sofia-dates.ts:19`), así que los montos salen como `1,234.50`. Es una deducción del código, no se vio en pantalla.

**Observaciones**
- El permiso `Devoluciones:Leer` se usa en el dashboard aunque el módulo de devoluciones no tiene pantalla de listado; el bloque depende solo del endpoint del dashboard.

---

## 4. Layout de la aplicación

Todas las pantallas que no son `/auth/*` se renderizan dentro de `FullComponent` (`src/app/matdash/layouts/full/full.component.ts`, `.html`), ruta padre `''` con `canActivate: [authGuard]` y `canActivateChild: [passwordChangeGuard, permissionGuard]` (`src/app/app.routes.ts:33-38`). Estructura (`full.component.html:1-47`): barra de progreso superior (§6.8), menú lateral a la izquierda y, a la derecha, cabecera + migas del layout + `<router-outlet>`. No hay pie de página en el layout (el HTML termina en el contenedor principal): la versión solo se ve en el menú de usuario (§4.2) y en los paneles de marca de las pantallas públicas.

Comportamiento responsive (`full.component.ts:23,67-80`): con ancho ≤ 1023 px (`MOBILE_VIEW`) el menú lateral queda cerrado y se abre como superposición con el botón de hamburguesa; se cierra solo al terminar cada navegación. Con ancho mayor está siempre abierto y el botón de hamburguesa de escritorio lo colapsa a modo «mini» (`sidenavCollapsed`, `toggleCollapsed()`, `full.component.ts:94-98`). El estado de colapso vive solo en memoria (`CoreService`, `src/app/matdash/services/core.service.ts:9`, valores por defecto en `src/app/matdash/config.ts:14-25`): al recargar vuelve a abierto.

### 4.1 Funcional, decorativo, oculto o sin uso (resumen)

| Elemento | Estado | Evidencia |
|---|---|---|
| Hamburguesa (abre el menú en móvil / colapsa el menú en escritorio) | Funcional | `vertical/header/header.component.html:4-9`; el cambio de botón por ancho está en `header.component.scss:103-115` (≥ 1024 px muestra el de escritorio) |
| Fecha larga en la cabecera | Funcional (solo informativa) | `header.component.html:10`, `header.component.ts:62-67` |
| Campana de notificaciones | **Decorativa**: abre un menú con el único texto «Sin notificaciones»; no existe origen de notificaciones | `header.component.html:19-26` |
| Selector de idioma (bandera) | Funcional, sin persistencia | §4.5 |
| Avatar y menú de usuario | Funcional (imagen fija) | §4.2 |
| Buscador global | **No existe** en el layout en uso. Los archivos `search-dialog.component.html` de `vertical/header` y `horizontal/header` no tienen componente asociado ni referencia en `src/` (búsqueda `search-dialog`/`SearchDialog` sin resultados en `.ts`) | `src/app/matdash/layouts/full/vertical/header/search-dialog.component.html` |
| Selector de tema claro/oscuro, color, dirección | **No existe** en pantalla. `CustomizerComponent` está importado en `full.component.ts:18` pero su selector `<app-customizer>` no aparece en `full.component.html`: no se renderiza | §4.6 |
| Migas del layout (`app-breadcrumb`) | Presente pero **nunca se dibuja**: solo muestra algo si la ruta activa trae `data.breadcrumb`, y ninguna ruta de `src/app` define esa propiedad (búsqueda `breadcrumb:` sin resultados) | `shared/breadcrumb/breadcrumb.component.html:1`; las pantallas pintan sus migas con `csw-page-header` (§6.2) |
| Layout horizontal | **Existe pero no se usa** | §4.7 |
| Pie de página | No existe | — |

### 4.2 Cabecera (`app-header`)

Archivo: `src/app/matdash/layouts/full/vertical/header/header.component.ts` y `.html`. Es una barra superior (`mat-toolbar`) con un grupo a la izquierda y otro a la derecha.

| Elemento | Texto (clave) | Qué hace | Fuente |
|---|---|---|---|
| Fecha | Fecha de hoy en formato `EEEE d 'de' MMMM 'de' y` (`header.date-format`), con la configuración regional `es-PE` si el idioma activo es español, `en-US` si no; por ejemplo «miércoles 7 de octubre de 2026» | Se calcula una vez al crear la cabecera (no se actualiza si la pantalla queda abierta pasada la medianoche) | `header.component.ts:59-67`, `sofia-date-adapter.ts:23-25` |
| Campana | Menú con «Sin notificaciones» (`header.noNotifications`) | Solo muestra ese texto | `header.component.html:19-26` |
| Bandera de idioma | Imagen de la bandera activa; `alt` «Idioma» (`header.language`) | Abre el menú «Español» / «English» (§4.5) | `header.component.html:29-42` |
| Botón de usuario | `aria-label` «Menú de usuario» (`header.profileMenu`); imagen fija `/images/profile/user-1.jpg` (la misma para todos los usuarios) | Abre el menú de usuario | `header.component.html:45-52` |
| Menú de usuario, tarjeta superior | Nombre: `fullName` del token (si está vacío, `unique_name`, luego `sub`, luego «—»); debajo, la empresa (`nombreEmpresa` del token o «—»). No es clicable | Informativo | `header.component.ts:78-82`, `.html:54-57` |
| «Cambiar contraseña» (`header.changePassword`) | Enlace | Navega a `/cambiar-clave` (§2). Es el único acceso voluntario a esa pantalla | `.html:59-61` |
| «Cerrar sesión» (`header.logout`) | Botón | Llama `AuthService.logout()` (§4.4); sin confirmación | `.html:62-64`, `.ts:105-107` |
| Versión | «Versión: 1.0.0» (`header.version` + `environment.appVersion`) | Informativo | `.html:66-68` |

### 4.3 Menú lateral

Archivos: `vertical/sidebar/sidebar.component.ts`, `.html`, `sidebar-data.ts`, `visible-nav-items.ts`, `nav-item/nav-item.component.*`, `branding.component.ts`.

- Arriba: marca «⚕ SOFIA», enlace a `/` (que redirige a `/dashboard`) (`branding.component.ts:9-12`). En móvil, además, un botón «×» para cerrar el menú (`sidebar.component.html:3-12`).
- Cada grupo muestra un **título de sección** (texto de `breadcrumbs.*`) y debajo sus opciones (texto de `nav.*`, icono Solar). No hay subgrupos ni opciones desplegables en los datos actuales. Los nombres internos de los grupos (`Principal`, `Administración`…) no se muestran.
- La opción activa (ruta actual o hija, comparación por prefijo, `nav-item.component.html:15-18`) se resalta. Clic izquierdo navega; Ctrl/Cmd+clic o clic central abre la ruta en una pestaña nueva (`nav-item.component.ts:162-170`). En móvil, al elegir una opción el menú se cierra.
- **Regla de visibilidad** (`sidebar.component.ts:28-34`, `visible-nav-items.ts:4-13`):
  1. Una opción con ruta se muestra si la ruta no tiene módulo asociado en `ROUTE_PERMISSIONS` (solo `/dashboard`) **o** si el usuario tiene el permiso `<Módulo>:Leer` (`AuthService.hasPermission(modulo)`, que concede todo al rol `Admin`).
  2. Un título de sección se conserva solo si justo después queda al menos una opción visible.
  3. Un grupo sin ninguna opción visible desaparece entero.
  4. Mientras los permisos no han llegado (`GET /api/v1/auth/me` en curso) todo lo que exige permiso está oculto: al principio solo se ve «Dashboard». Si la carga falla aparece el snackbar «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.» (`errors.permissionsLoad`, `sidebar.component.ts:36-43`) y el menú queda con «Dashboard».
  5. Ocultar una opción no bloquea la URL: el `permissionGuard` (§5.3) y la API siguen comprobando.

| Título de sección (clave) | Opción (clave `nav.*`) | Icono | Ruta | Módulo:Acción de visibilidad |
|---|---|---|---|---|
| Inicio (`breadcrumbs.home`) | Dashboard (`dashboard`) | `solar:widget-add-line-duotone` | `/dashboard` | Ninguno (siempre visible) |
| Administración (`breadcrumbs.administracion`) | Empresa (`empresa`) | `solar:buildings-line-duotone` | `/empresas` | `Empresas:Leer` |
| | Sucursales (`sucursales`) | `solar:shop-2-line-duotone` | `/sucursales` | `Sucursales:Leer` |
| | Empleados (`empleados`) | `solar:users-group-rounded-line-duotone` | `/empleados` | `Empleados:Leer` |
| | Roles & Permisos (`rolesPermisos`) | `solar:lock-keyhole-line-duotone` | `/roles` | `Seguridad:Leer` |
| | Cuentas de Usuario (`cuentasUsuario`) | `solar:user-id-line-duotone` | `/cuentas` | `Seguridad:Leer` |
| Catálogos (`breadcrumbs.catalogos`) | Unidades de Medida (`unidadesMedida`) | `solar:ruler-line-duotone` | `/unidades-medida` | `UnidadesMedida:Leer` |
| | Jerarquías de Unidades (`jerarquiasUom`) | `solar:sort-vertical-line-duotone` | `/jerarquias` | `JerarquiasUoM:Leer` |
| | Laboratorios (`laboratorios`) | `solar:test-tube-line-duotone` | `/laboratorios` | `Laboratorios:Leer` |
| | Ingredientes Activos (`ingredientesActivos`) | `solar:atom-line-duotone` | `/ingredientes-activos` | `IngredientesActivos:Leer` |
| | Medicamentos (`medicamentos`) | `solar:pill-line-duotone` | `/medicamentos` | `Medicamentos:Leer` |
| | Presentaciones de Venta (`presentacionesVenta`) | `solar:tag-price-line-duotone` | `/presentaciones-venta` | `PresentacionesVenta:Leer` |
| | Formulaciones Clínicas (`formulacionesClinicas`) | `solar:test-tube-minimalistic-line-duotone` | `/formulaciones-clinicas` | `FormulacionesClinicas:Leer` |
| | Proveedores (`proveedores`) | `solar:shop-line-duotone` | `/proveedores` | `Proveedores:Leer` |
| | Aseguradoras (`aseguradoras`) | `solar:shield-user-line-duotone` | `/aseguradoras` | `Seguros:Leer` |
| Inventario (`breadcrumbs.inventario`) | Lotes (`lotes`) | `solar:box-line-duotone` | `/lotes` | `Inventarios:Leer` |
| | Stock por Sucursal (`stockPorSucursal`) | `solar:checklist-minimalistic-line-duotone` | `/stock-por-sucursal` | `Inventarios:Leer` |
| Punto de Venta (`breadcrumbs.puntoVenta`) | Sesiones de Caja (`sesionesCaja`) | `solar:cash-out-line-duotone` | `/sesiones-caja` | `POS:Leer` |
| | Ventas (`ventas`) | `solar:bill-list-line-duotone` | `/pos` | `Ventas:Leer` |
| Pacientes & Atención (`breadcrumbs.pacientesAtencion`) | Pacientes (`pacientes`) | `solar:user-line-duotone` | `/pacientes` | `Pacientes:Leer` |
| | Profesionales de Salud (`profesionalesSalud`) | `solar:stethoscope-line-duotone` | `/profesionales-salud` | `ProfesionalesSalud:Leer` |
| | Recetas Médicas (`recetasMedicas`) | `solar:document-text-line-duotone` | `/recetas-medicas` | `Recetas:Leer` |
| DIGEMID (`breadcrumbs.digemid`) | Catálogo DIGEMID (`catalogoDigemid`) | `solar:shield-check-line-duotone` | `/digemid-catalogo` | `DIGEMID:Leer` |
| SUNAT (`breadcrumbs.sunat`) | Series Fiscales (`seriesFiscales`) | `solar:bill-check-line-duotone` | `/series-fiscales` | `SeriesFiscales:Leer` |

Fuente de la tabla: `sidebar-data.ts:3-180` (textos, iconos y rutas) y `route-permissions.ts:2-25` (módulo de cada ruta). Total: 8 grupos, 24 opciones (incluido Dashboard). Las pantallas de Transferencias, Delivery, Magistrales, Servicios, Cuarentena/Actas y Auditoría no tienen opción de menú ni ruta.

Observaciones del menú: `nav.inventario` existe en `es.json` pero ninguna opción lo usa (el título «Inventario» sale de `breadcrumbs.inventario`). Los comentarios de `nav-item.component.ts` hacen referencia a ejemplos de otro producto («Importación», «Documentación»): es código heredado de la plantilla.

### 4.4 Cierre de sesión (paso a paso)

Disparo: «Cerrar sesión» del menú de usuario, o automático cuando el interceptor no logra renovar la sesión (§5.4). `AuthService.logout()` (`src/app/core/auth/auth.service.ts:113-123`):

1. Envía `POST /api/v1/auth/logout` con la cookie de refresco y la cabecera `X-SOFIA-CSRF: 1` (endpoint anónimo, `AuthController.cs:70`). El servidor revoca la sesión desde la cookie aunque el token de acceso haya vencido.
2. Sin esperar la respuesta, borra del navegador los datos por usuario: claves de `localStorage` que empiezan por `pos-nueva-venta-draft` y `sofia_menu_cache` (`clearUserStorage`, `user-storage.ts:2-5,13-15`).
3. Ejecuta `clearLocalSession()`: elimina `sofia_token` de `localStorage`, deja el token en nulo, vacía los filtros de búsqueda guardados en memoria (`SearchStateService.clearAll`) y descarta roles y permisos en memoria (`auth.service.ts:155-165`).
4. Cuando el servidor responde (bien o con error; el error se ignora) navega a `/auth/login`.

No hay diálogo de confirmación ni mensaje. Un cierre por sesión vencida (§5.4) tampoco muestra aviso.

### 4.5 Selector de idioma

- Idiomas: «Español» (`es`, bandera `/images/flag/icon-flag-es.svg`) e «English» (`en`, `icon-flag-en.svg`) (`header.component.ts:69-72`); Transloco declara `availableLangs: ['es','en']`, `defaultLang` y `fallbackLang` `es` (`src/app/app.config.ts:53-55`). Existen archivos `en.json` en la raíz y en los 33 scopes (33 `es.json` y 33 `en.json` bajo `public/i18n/<scope>/`).
- Elegir el idioma que ya está activo no hace nada (`changeLang`, `header.component.ts:88-90`). Elegir el otro abre `ConfirmDialogComponent` de 400 px con título «Cambiar idioma» (`CONFIRMATION.CHANGE_LANGUAGE_TITLE`), mensaje «La interfaz se actualizará al idioma seleccionado. ¿Deseas continuar?» (`CONFIRMATION.CHANGE_LANGUAGE_MESSAGE`) y botones «Cancelar» (`actions.cancel`) y «Confirmar» (`actions.confirm`). Al confirmar se llama `TranslocoService.setActiveLang(code)`; al cancelar no pasa nada.
- **Dónde se guarda:** en ningún sitio. La única escritura es `setActiveLang` en memoria; no hay `localStorage` ni cookie de idioma (búsqueda de `localStorage`/`sessionStorage` en `src/app`: solo token, zona horaria y borradores de POS). Al recargar la página vuelve a español. Las rutas públicas (`/auth/*`) no tienen selector de idioma y siempre arrancan en español.
- El idioma activo también cambia el formato de fechas del datepicker y de la cabecera (§6.10).
- Las claves `header.reloadPermissions`, `header.roles`, `header.notifications`, `CONFIRMATION.RELOAD_PERMISSIONS_TITLE` y `CONFIRMATION.RELOAD_PERMISSIONS_MESSAGE` describen un «Recargar permisos» que no existe en ninguna pantalla (sin referencias en `src/`).

### 4.6 Personalizador de tema (Customizer)

`CustomizerComponent` (`layouts/full/shared/customizer/customizer.component.ts`) define `setDark()` (`theme: 'dark'`), `setColor()` (`activeTheme`), `setDir()` (`ltr`/`rtl`) y `setSidebar()`. **No está activo**: su selector `<app-customizer>` no se usa en ninguna plantilla (solo aparece el `import` de `full.component.ts:18` y su lista de `imports`). El usuario no puede cambiar tema, color ni dirección. Los valores por defecto (`theme: 'light'`, `activeTheme: 'blue_theme'`, `dir: 'ltr'`, `cardBorder: false`) están en `config.ts:14-25`; `FullComponent` solo usa `dir` y `cardBorder` (`full.component.html:6-12`). `defaults.language: 'en-us'` no se lee en ningún componente (el idioma real lo gobierna Transloco).

### 4.7 Layout horizontal

Existe el código `layouts/full/horizontal/` (cabecera, menú, `sidebar-data.ts`) y `FullComponent` lo muestra si `options.horizontal && !resView` (`full.component.html:30-32`). **No se usa:** el valor por defecto es `horizontal: false` (`config.ts:20`) y ningún código llama a `setOptions({ horizontal: true })` (búsqueda sin resultados). Su cabecera tiene textos en duro («Usuario SOFIA», «desarrollo@sofia.com», «SOFIA v…») (`horizontal/header/header.component.html:16,23-24`). No se inspeccionó el contenido de `horizontal/sidebar/sidebar-data.ts` (no se muestra a ningún usuario).

---

## 5. Guards e interceptores vistos desde el usuario

Registro: guards en `src/app/app.routes.ts:33-38`; interceptores en `src/app/app.config.ts:45-48` (`withInterceptors([authInterceptor, CSLoadingInterceptor])`).

### 5.1 `authGuard` — sesión iniciada

`src/app/core/auth/auth.guard.ts:5-18`. Protege todo el layout.
- Si el token guardado es válido (existe y su `exp` no venció) deja pasar.
- Si no (sin token o vencido) intenta una **renovación silenciosa** `POST /api/v1/auth/refresh` con la cookie de refresco y la cabecera `X-SOFIA-CSRF`. Si responde, guarda el token nuevo y deja pasar sin que el usuario note nada.
- Si la renovación falla, hace `clearLocalSession()` y redirige a `/auth/login`. **No muestra mensaje alguno.**
- Las renovaciones simultáneas (guard + interceptor) se unifican en una sola petición (`auth.service.ts:83-97`).

### 5.2 `passwordChangeGuard` — cambio de contraseña obligatorio

`src/app/core/auth/password-change.guard.ts:7-12`. Si el token trae `pwd_change = "true"` (`auth.requiresPasswordChange()`, `auth.service.ts:68`) y la URL no empieza por `/cambiar-clave`, redirige a `/cambiar-clave` (sin mensaje en la redirección; en esa pantalla ves el aviso `cambiarClave.forced`, §2). El usuario puede ver el menú, pero cualquier opción que elija vuelve a `/cambiar-clave`. El mismo caso se cubre al iniciar sesión (el login navega directo a `/cambiar-clave`, §1.1) y en medio de una sesión viva (§5.4). Al cambiar la contraseña todas las sesiones se revocan y se vuelve a `/auth/login`.

### 5.3 `permissionGuard` — permiso de la pantalla

`src/app/core/auth/permission.guard.ts:9-33`, reglas en `route-permissions.ts`.
- Toma el **primer segmento** de la URL (`/sucursales/…` → `sucursales`) y busca su módulo en `ROUTE_PERMISSIONS` (`route-permissions.ts:2-25`). Sin módulo (`dashboard`, `cambiar-clave`) deja pasar.
- Acción exigida: `Leer` siempre; más `Crear` si el segundo segmento es `nueva`, `nuevo` o `carga-masiva`; excepciones de `ROUTE_ACTIONS`: `recetas-medicas/escanear` → `Analizar`, `sesiones-caja/nueva` → `AperturarCaja`, `stock-por-sucursal/nueva` → `Actualizar` (`route-permissions.ts:28-33`). Las demás subrutas (`/roles/:id`, edición, etc.) solo exigen `Leer`; los botones de cada acción los controla `*appPermiso` y la API.
- Antes de decidir carga los permisos (`GET /api/v1/auth/me`, una vez por usuario). El rol `Admin` pasa siempre; los nombres de módulo y acción se comparan sin distinguir mayúsculas y recortando espacios (`route-permissions.ts:42-53`).
- Módulo de cada primer segmento: ver la columna «Módulo:Acción» de §4.3 (misma fuente, `route-permissions.ts:2-25`).
- **Si falta el permiso:** snackbar de error «No tienes permisos para acceder a esta pantalla.» (`errors.forbiddenPage`, 5 s) y redirección a `/dashboard`.
- **Si no se pudo cargar el perfil:** snackbar «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.» (`errors.permissionsLoad`) y redirección a `/dashboard`.

### 5.4 `authInterceptor` — token y renovación en segundo plano

`src/app/core/interceptors/auth.interceptor.ts:19-73`.
- Solo actúa sobre peticiones cuya URL empieza por `<api>/api/` (`isApiRequest`, `:19-21`); los archivos estáticos (`/i18n/...`) y otros hosts no reciben el token.
- Añade `Authorization: Bearer <token>` si hay token guardado.
- **Cambio de contraseña forzado en sesión viva:** si la API responde `403` con código `Auth.CambioClaveRequerido`, navega a `/cambiar-clave` y deja que el error siga su camino (`:30-33`).
- **Respuesta 401:**
  - En `/auth/login` solo propaga el error (la pantalla muestra «Credenciales inválidas…»). En `/auth/refresh` y `/auth/logout` borra la sesión local sin más llamadas y propaga (`:40-46`).
  - En cualquier otro endpoint: pide una renovación (`refreshAccessToken`) y, si funciona, **repite la petición original** con el token nuevo sin avisar al usuario; si la renovación falla, ejecuta `auth.logout()` (§4.4: borra datos locales y va a `/auth/login`) y propaga el error (`:51-64`). No hay aviso de «sesión expirada» (la clave `errors.unauthorized` no se usa en el código).
- **No hay renovación proactiva:** no existe temporizador; el token se renueva al navegar con el token vencido (§5.1) o al recibir el primer 401.
- Los demás errores (400, 403 normales, 404, 409, 429, 5xx, sin conexión) **no tienen tratamiento global**: cada pantalla decide el mensaje con su propio mapa de códigos y su mensaje genérico (ver cada ficha; el login distingue la caída del servidor con `auth.serviceUnavailable`, §1.1). No existe un traductor global de códigos de error. La única utilidad compartida es `saveErrorKey` (`src/app/shared/utils/save-error-key.util.ts:11-15`): busca el código en el mapa de la pantalla; si no está, busca `Db.Duplicado` → «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» (`errors.dbDuplicado`), `Db.ReferenciaInvalida` → «El registro hace referencia a datos que ya no existen o todavía está en uso.» (`errors.dbReferenciaInvalida`) y `Db.ValorInvalido` → «Uno de los valores está fuera del rango permitido.» (`errors.dbValorInvalido`); si tampoco, devuelve el mensaje genérico que le pasa la pantalla. Se llama desde 36 puntos de `src/app` (conteo de referencias fuera de la utilidad y de las pruebas). El código `Concurrency.Conflict` lo mapea cada pantalla que lo declara a «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (`errors.concurrencyConflict`), por ejemplo `aperturar-caja-dialog.component.ts:15`.

### 5.5 `CSLoadingInterceptor` — barra de carga

`src/app/shared/services/loading/loading.interceptor.ts:6-31`. Mientras haya al menos una petición HTTP en curso (cualquier URL, incluidas las de `/i18n`) se muestra la barra de progreso superior del layout (§6.8). Se apaga al terminar, con éxito o con error.

---

## 6. Componentes compartidos y `ErrorNotifierService`

Carpeta `src/app/shared/components`. Comprobación de uso hecha buscando el selector en `.html` y la clase en `.ts` (sin contar pruebas ni la propia carpeta del componente).

### 6.1 `ConfirmDialogComponent` (`csw-confirm-dialog`) — diálogo de confirmación

- **Archivos:** `shared/components/confirm-dialog/confirm-dialog.component.ts`, `.html`. Se abre con `MatDialog.open`; no tiene selector de uso en plantillas.
- **Datos de entrada** (`ConfirmDialogData`, `.ts:10-15`): `title`, `message` (ya traducidos por quien lo abre), `confirmButtonText?`, `cancelButtonText?`.
- **Contenido:** título (`<h2>`), mensaje en un párrafo con saltos de línea respetados (clase `pre-line`) y, alineados a la derecha, dos botones: cancelar (`mat-button`) y confirmar (`mat-flat-button` primario) (`.html:1-12`).
- **Textos por defecto:** «Confirmar» (`actions.confirm`) y «Cancelar» (`actions.cancel`) si quien lo abre no pasa otros (`.ts:30-36`).
- **Resultado:** confirmar cierra con `true`; cancelar cierra con `false` (`.ts:38-44`).
- **Dónde se usa:** el cambio de idioma de la cabecera (§4.5) y 19 pantallas de detalle/diálogos: aseguradoras, empleados, formulaciones clínicas, ingredientes activos, jerarquías, laboratorios, lotes, medicamentos, pacientes, presentaciones de venta (detalle y modal), profesionales de salud, proveedores, recetas médicas, roles, cuentas, series fiscales, sucursales y unidades de medida (búsqueda de `ConfirmDialogComponent` en `.ts`). Cada ficha de esas partes describe el título y el mensaje concretos.

### 6.2 `PageHeaderComponent` (`csw-page-header`) — cabecera de página con migas

- **Archivos:** `shared/components/page-header/page-header.component.ts`, `.html`, `.scss`. Se usa en 50 plantillas.
- **Entradas:** `title` (texto **sin traducir**: se pinta tal cual, la pantalla debe pasarlo ya traducido), `breadcrumbItems` (lista de `{ label, route?, isActive? }`), `showBackButton` (por defecto `true`), `backRoute` (`.ts:21-24`).
- **Contenido:** a la izquierda el botón de volver (icono `chevron_left`, `aria-label` «Volver» escrito en duro, `.html:6-13`; si `showBackButton` es falso deja un espacio vacío de 25 px) y el título (`<h4>`); a la derecha las migas separadas por « / ». Cada `label` de miga **sí** se traduce con el pipe `transloco` (por eso se pasan claves como `breadcrumbs.home`). La última miga se marca activa y no es clicable; las anteriores con `route` navegan (`onBreadcrumbClick`, `.ts:36-40`).
- **Volver:** si hay `backRoute` navega allí; si no, `window.history.back()` (`.ts:28-34`).

### 6.3 `AutocompleteInputComponent` (`csw-autocomplete-input`) — autocompletado con búsqueda en servidor

- **Archivos:** `shared/components/autocomplete-input/autocomplete-input.component.ts`, `.html`, `.css`.
- **Entradas principales** (`.ts:35-46`): `fetch` (función que recibe el texto y devuelve opciones), `displayWith`, `control`, `placeholder`, `initialText`, `autoSelectOnExact` (verdadero), `allowFreeText` (verdadero), `maxLength`, `minLength` (por defecto 2), `clearOnInvalidSelection` (falso), `errorSource`. Salidas: `selectedChanges` (opción elegida o `null`) y `blurEvent`.
- **Comportamiento:** campo de búsqueda con espera de 500 ms tras escribir (`debounceTime(500)`, `.ts:100-104`); no busca hasta alcanzar `minLength` caracteres; mientras busca muestra una opción deshabilitada «Cargando...» (**texto en duro**, `.html:21-23`). Al elegir una opción pone su texto en el campo y emite la opción. Con `clearOnInvalidSelection` verdadero, al salir del campo con un texto que no corresponde a una opción lo vacía y emite `null` (`.ts:173-193`). Métodos públicos `clear()` y `restoreValue()` para que la pantalla lo reinicie. Si `errorSource` está inválido y tocado/sucio, dibuja el borde de error.
- **Dónde se usa:** `cuentas-detail` (empleado, rol, sucursal; tres instancias) y `sucursales-detail` (gerente, al crear y al editar: se dibuja con `isEditMode()`, `sucursales-detail.component.html:66-67`).
- **Sin uso:** existe otra variante, `AutocompleteInputMultipleComponent` (`shared/components/autocomplete-input-multiple/`), con el **mismo selector** `csw-autocomplete-input`; ninguna plantilla ni `.ts` importa su clase (sin uso, §6.13).

### 6.4 `RangeSliderFieldComponent` (`app-range-slider-field`) — filtro numérico con deslizador de rango

- **Archivo:** `shared/components/range-slider-field/range-slider-field.component.ts` (plantilla en línea).
- **Entradas:** `label` (obligatoria), `min` (0), `max` (100), `step` (1), `unit` (''), y los modelos de doble enlace `startValue` y `endValue` (obligatorios).
- **Contenido:** una línea «{label}: **inicio**{unidad} — **fin**{unidad}» y un `mat-slider` discreto con dos tiradores (`.ts:9-17`).
- **Dónde se usa:** filtro «Multiplicador» de `jerarquias-search`, «Tasa de Cumplimiento» de `proveedores-search` y «Nivel de Jerarquía» de `roles-search` (etiquetas `fields.multiplicador`, `fields.tasaCumplimiento`, `fields.nivelJerarquia` de la raíz, §8.7).

### 6.5 `CargaMasivaPageComponent` (`app-carga-masiva-page`) — carga masiva desde Excel

- **Archivos:** `shared/components/carga-masiva-page/carga-masiva-page.component.ts`, `.html`; modelos en `shared/models/carga-masiva.model.ts`.
- **Cómo se usa:** no se enruta solo; cada maestro tiene un componente «envoltorio» que le pasa una `CargaMasivaConfig` (`downloadUrl`, `previewUrl`, `saveUrl`, `backRoute`, `breadcrumbs`, `columns`) (`carga-masiva.model.ts:11-18`). La ruta es `<maestro>/carga-masiva` (§6.6). Los textos de la pantalla vienen de la **raíz** `carga-masiva.*` (§8.9); el título se resuelve con `selectTranslate` para esperar a que cargue la raíz (`.ts:57-60`).
- **Migas y cabecera:** `csw-page-header` con título «Carga Masiva» (`carga-masiva.title`), migas de la configuración (las 12 empiezan con «Inicio» y terminan con «Carga Masiva»; en medio van el grupo y el maestro) y botón volver a `backRoute`.
- **Límites:** archivo `.xlsx` de hasta **5 MB** y **5000 filas** (`MAX_UPLOAD_MB = 5`, `MAX_IMPORT_ROWS = 5000`, `.ts:13-14`; «refleja ImportLimits del backend»).

**Flujo completo**

| Paso | Qué ve y hace el usuario | Qué ocurre | Texto (clave raíz) |
|---|---|---|---|
| 1. Descargar plantilla | Botón con icono `download` arriba a la derecha de la tarjeta | `GET downloadUrl` como blob (con token); descarga el archivo `plantilla-{entidad}.xlsx`, donde `{entidad}` es la `backRoute` sin la barra inicial (por ejemplo `plantilla-roles.xlsx`) (`.ts:123-136`) | «Descargar plantilla» (`carga-masiva.download-template`); nombre `plantilla-{{entity}}.xlsx` (`carga-masiva.template-filename`); fallo: «Error al descargar la plantilla. Intenta de nuevo.» (`carga-masiva.download-error`, con referencia de seguimiento si la respuesta trae `traceId`) |
| 2. Elegir archivo | Zona punteada: arrastrar el Excel o hacer clic (selector `accept=".xlsx"`) | Valida en el navegador antes de enviar: si el nombre no termina en `.xlsx` → error; si pesa más de 5 MB → error (`.ts:97-105`) | «Arrastra tu archivo Excel aquí o haz clic para seleccionar» (`carga-masiva.upload-zone`); «Solo archivos .xlsx (máx. 5 MB y 5000 filas)» (`carga-masiva.upload-zone-hint`); «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» (`carga-masiva.errors.invalid-file`); «El archivo supera el tamaño máximo de 5 MB.» (`carga-masiva.errors.too-large`) |
| 3. Procesar | Spinner con texto mientras responde | `POST previewUrl` con el archivo como `multipart/form-data` (campo `file`); no guarda nada (`.ts:106-119`) | «Procesando archivo...» (`carga-masiva.loading-preview`) |
| 4. Vista previa | Tarjeta «Vista previa» con contadores y tabla | Se muestran todas las filas devueltas: una columna por cada columna de la configuración (encabezado = su `label` traducido), una columna de errores sin encabezado y la columna «Estado» con icono `check_circle` (fila válida) o `error_outline` (fila con error) | «Vista previa» (`carga-masiva.preview-title`); «{count} filas totales» (`total-rows`); «{count} filas válidas» (`valid-rows`); «{count} filas con error» (`error-rows`, solo si hay errores); «Estado» (`carga-masiva.row-status`) |
| 5. Errores por fila | En la columna de errores, una lista con un mensaje por cada error de la fila inválida | El backend devuelve por cada error un `code`, el `field` y `params`; el texto es `carga-masiva.validation.<code>` con esos valores (`carga-masiva-page.component.html:94-97`, `getErrorParams` `.ts:175-177`) | Tabla de §8.9 |
| 6. Confirmar | Botón «Guardar válidos (N)» bajo la tabla | Deshabilitado mientras guarda o si no hay filas válidas. Envía **solo las filas válidas** (`data` de cada una) con `POST saveUrl` en JSON (`.ts:138-159`) | «Guardar válidos ({{count}})» (`carga-masiva.save-valid`) |
| 7. Resultado | Snackbar de éxito y regreso a la lista del maestro | `router.navigate(backRoute)` | «Se guardaron {{count}} registros correctamente» (`carga-masiva.save-success`) |

**Errores de las llamadas de vista previa y guardado** (`cargaMasivaErrorKey`, `.ts:25-34`; si no hay clave específica se usa `showServerError` con el mensaje genérico del paso):

| Respuesta | Mensaje |
|---|---|
| Estado 413, o código `Request.TooLarge` | «El archivo supera el tamaño máximo de 5 MB.» (`carga-masiva.errors.too-large`) |
| Código `CargaMasiva.Archivo.Invalido` | «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» (`carga-masiva.errors.invalid-file`) |
| Código `CargaMasiva.Archivo.ExcedeDescompresion` | «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.» (`carga-masiva.errors.decompression`) |
| Código `CargaMasiva.Filas.Excedidas`, o `CargaMasiva<X>Command.Rows` | «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.» (`carga-masiva.errors.too-many-rows`) |
| Código `CargaMasiva<X>Command.Rows[...]` (validador de fila en el guardado) | «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» (`carga-masiva.errors.invalid-rows`) |
| Otro error en la vista previa | «Error al procesar el archivo» (`carga-masiva.preview-error`) |
| Otro error al guardar | «Error al guardar los registros» (`carga-masiva.save-error`) |
| Guardar sin filas válidas (el botón lo impide; red de seguridad en el método) | «No hay filas válidas para guardar» (`carga-masiva.no-valid-rows`) |

**Comportamientos particulares:** el selector de archivo se limpia tras elegir (se puede volver a elegir el mismo); una vista previa nueva reemplaza a la anterior (`preview.set(null)` antes de enviar). La tabla no tiene paginación.

**Observaciones:** las claves `carga-masiva.row-number`, `status-valid`, `status-error` y `saving` no se usan en ningún archivo; `upload-file`, `upload-success` y `upload-error` solo las usa el componente `sofia-carga-masiva`, que no se utiliza en ninguna pantalla (§6.13).

### 6.6 Envoltorios de carga masiva (quién usa `CargaMasivaPageComponent`)

Doce componentes `*-carga-masiva` (clase en `.ts` con plantilla `<app-carga-masiva-page [config]="config()" />`), cada uno en la ruta `<ruta-del-maestro>/carga-masiva`. Los endpoints salen de sus `downloadUrl`/`previewUrl`/`saveUrl`; el permiso del endpoint es siempre `<Módulo>:Crear` (`docs/_inventario/endpoints.csv`). El permiso de ruta es el mismo (`Crear`, regla de `carga-masiva`, §5.3). El detalle de columnas de cada plantilla lo cubre el documento 20.

| Maestro | Ruta | `backRoute` | Base de la API (`/api/v1/...`) | Permiso |
|---|---|---|---|---|
| Aseguradoras | `/aseguradoras/carga-masiva` | `/aseguradoras` | `seguros` | `Seguros:Crear` |
| Catálogo DIGEMID | `/digemid-catalogo/carga-masiva` | `/digemid-catalogo` | `digemid/catalogo` | `DIGEMID:Crear` |
| Ingredientes activos | `/ingredientes-activos/carga-masiva` | `/ingredientes-activos` | `ingredientesactivos` | `IngredientesActivos:Crear` |
| Jerarquías de unidades | `/jerarquias/carga-masiva` | `/jerarquias` | `jerarquiasuom` | `JerarquiasUoM:Crear` |
| Laboratorios | `/laboratorios/carga-masiva` | `/laboratorios` | `laboratorios` | `Laboratorios:Crear` |
| Medicamentos | `/medicamentos/carga-masiva` | `/medicamentos` | `medicamentos` | `Medicamentos:Crear` |
| Pacientes | `/pacientes/carga-masiva` | `/pacientes` | `pacientes` | `Pacientes:Crear` |
| Profesionales de salud | `/profesionales-salud/carga-masiva` | `/profesionales-salud` | `profesionalessalud` | `ProfesionalesSalud:Crear` |
| Proveedores | `/proveedores/carga-masiva` | `/proveedores` | `proveedores` | `Proveedores:Crear` |
| Roles | `/roles/carga-masiva` | `/roles` | `roles` | `Seguridad:Crear` |
| Sucursales | `/sucursales/carga-masiva` | `/sucursales` | `sucursales` | `Sucursales:Crear` |
| Unidades de medida | `/unidades-medida/carga-masiva` | `/unidades-medida` | `unidadesmedida` | `UnidadesMedida:Crear` |

En cada base los tres endpoints son `GET …/plantilla`, `POST …/previsualizar` y `POST …/carga-masiva`. Estas rutas no figuran en `rutas-spa.csv` bajo mi alcance: las documenta la parte que cubre cada maestro; aquí queda el componente común.

### 6.7 Resumen de uso de componentes compartidos

| Componente | Selector | Estado |
|---|---|---|
| `AutocompleteInputComponent` | `csw-autocomplete-input` | En uso (§6.3) |
| `AutocompleteInputMultipleComponent` | `csw-autocomplete-input` (duplicado) | Sin uso |
| `CargaMasivaComponent` | `sofia-carga-masiva` | Sin uso |
| `CargaMasivaPageComponent` | `app-carga-masiva-page` | En uso (§6.5) |
| `ConfirmDialogComponent` | `csw-confirm-dialog` | En uso (§6.1) |
| `MatDatepickerIntlTransloco` | (proveedor) | En uso (§6.10) |
| `DatetimePickerComponent` | `csw-datetime-picker` | Sin uso |
| `DecimalInputComponent` | `csw-decimal-input` | Sin uso |
| `ErrorDialogComponent` | `csw-error-dialog` | Sin uso |
| `CSLoadingBarComponent` | `csw-loading-bar` | En uso (§6.8) |
| `MultiselectAutocompleteInputComponent` | `csw-multiselect-autocomplete-input` | Sin uso |
| `PageHeaderComponent` | `csw-page-header` | En uso (§6.2) |
| `MatPaginatorIntlTransloco` | (proveedor) | En uso (§6.9) |
| `RangeSliderFieldComponent` | `app-range-slider-field` | En uso (§6.4) |
| `ServerErrorSnackbarComponent` | `app-server-error-snackbar` | En uso, abierto por código (§6.11) |
| `SimpleDialogComponent` | `csw-simple-dialog` | Sin uso |
| `StatusChipsComponent` | `sofia-status-chips` | Sin uso |
| `SuccessDialogComponent` | `csw-success-dialog` | Sin uso |
| `TextInputDialogComponent` | `csw-text-input-dialog` | Sin uso |
| `ThreeOptionDialogComponent` | `csw-three-option-dialog` | Sin uso |

### 6.8 `CSLoadingBarComponent` (`csw-loading-bar`) — barra de carga

- **Archivos:** `shared/components/loading-bar/loading-bar.component.ts`, `.html`, `.scss`; estado en `shared/services/loading/loading.service.ts`.
- **Qué hace:** una barra de progreso de Material en la parte superior del layout (`full.component.html:1`), indeterminada por defecto, que aparece mientras `CSLoadingInterceptor` (§5.5) tenga alguna petición en curso. Entrada `autoMode` (verdadero) controla si la gestiona el interceptor. No tiene texto. Solo existe dentro del layout: las pantallas públicas (`/auth/*`) no la muestran.

### 6.9 Paginador internacionalizado

- **Archivo:** `shared/components/paginator-internationalization/mat-paginator-intl.component.ts` (`MatPaginatorIntlTransloco`), registrado como `MatPaginatorIntl` global en `app.config.ts:50`.
- **Textos (raíz `pagination.*`):** «Items por página» (`itemsPerPage`), «Siguiente página», «Página anterior», «Primera página», «Última página», rango «Página {{current}} de {{total}}» (`range`) y, sin resultados, «Sin resultados» (`rangeEmpty`). Se actualizan al cambiar el idioma (`langChanges$`).
- **Tamaños de página:** 23 plantillas de búsqueda usan `[pageSizeOptions]="[10, 25, 50]"` (búsqueda en `src/app/pages`).
- **Sin uso:** `shared/services/custom-paginator-intl.service.ts` (`CustomPaginatorIntl`) no se referencia en ningún otro archivo.

### 6.10 Datepicker internacionalizado

- **Archivos:** `shared/components/datepicker-internationalization/mat-datepicker-intl.component.ts` (`MatDatepickerIntlTransloco`) y `shared/date/provide-sofia-dates.ts`, `sofia-date-adapter.ts`.
- **Etiquetas (raíz `datepicker.*`):** «Calendario», «Abrir calendario», «Cerrar calendario», «Mes anterior», «Mes siguiente», «Año anterior», «Año siguiente», «24 años anteriores», «24 años siguientes», «Elegir fecha» (`switchToMonthView`), «Elegir mes y año» (`switchToMultiYearView`).
- **Formato y escritura:** el adaptador sigue el idioma activo: con español usa la configuración regional `es-PE` y lee las fechas escritas como día/mes/año; con otro idioma usa `en-US` y las lee como mes/día/año (`sofia-date-adapter.ts:23-25,52-61`). Acepta también `yyyy-MM-dd`. Formato mostrado en el campo: año numérico, mes y día de 2 dígitos (`SOFIA_DATE_FORMATS`, `:11-21`). Se registra en toda la aplicación con `provideSofiaDates()` (`app.config.ts:32`), que también carga los datos regionales `es-PE` (`provide-sofia-dates.ts:19`).

### 6.11 `ServerErrorSnackbarComponent` (`app-server-error-snackbar`)

- **Archivo:** `shared/components/server-error-snackbar/server-error-snackbar.component.ts` (plantilla en línea). Se abre solo desde `ErrorNotifierService.showServerError` (§6.12), nunca en una plantilla.
- **Contenido:** icono de error, el mensaje amigable y debajo «Referencia: **{traceId}**» (`generic-labels.trace-ref`); un botón de copiar con tooltip «Copiar referencia» (`generic-labels.copy-ref`) que copia el `traceId` al portapapeles y cambia el icono a una marca durante 2 s, y un botón de cerrar (`.ts:17-36,71-80`). Dura 10 s (§6.12).

### 6.12 `ErrorNotifierService` — avisos al usuario

Archivo: `src/app/core/services/shared/error-notifier.service.ts` (servicio raíz). Todos los avisos son `MatSnackBar` con el botón «Cerrar» (`actions.close`) y la clase `error-snackbar` o `success-snackbar`.

| Método | Qué muestra | Duración | Usos en `src/app` (sin pruebas) | Fuente |
|---|---|---|---|---|
| `showError(message, duration = 5000)` | El texto recibido, estilo de error | 5 s (configurable) | 155 | `error-notifier.service.ts:26-28` |
| `showSuccess(message, duration = 3000)` | El texto recibido, estilo de éxito | 3 s (configurable; reset-password y cambiar-clave usan 6 s) | 69 | `:30-32` |
| `showServerError(err, friendlyMessage)` | Si el error es una respuesta HTTP con `traceId` (en `error.traceId` o `error.extensions.traceId`), abre `ServerErrorSnackbarComponent` con el mensaje y la referencia (§6.11); si no, equivale a `showError(friendlyMessage)` | 10 s con referencia; 5 s sin ella | 19 | `:34-46` |
| `showSmartErrors(errors, opts?)` | Une con « \| » el `errorMessage` (o `errorCode`) de cada elemento; si la lista está vacía muestra `opts.fallback` o «Ocurrió un error desconocido. Por favor, inténtalo de nuevo más tarde.» (`errors.unknownError`) | 5 s | **0: sin uso** (solo la definición y su prueba) | `:18-24` |

El servicio no traduce por sí mismo salvo el texto «Cerrar» y el de `showSmartErrors`: cada pantalla traduce antes de llamar.

### 6.13 Componentes y servicios compartidos sin uso (comprobado)

Comprobado buscando la clase y el selector en todo `src/app` fuera de la carpeta del propio componente y de las pruebas (sin resultados):

| Elemento | Archivos | Nota |
|---|---|---|
| `AutocompleteInputMultipleComponent` | `shared/components/autocomplete-input-multiple/` | Mismo selector que el autocompletado en uso |
| `CargaMasivaComponent` (`sofia-carga-masiva`) | `shared/components/carga-masiva/carga-masiva.component.ts` | Versión anterior de la carga masiva; solo ella usa `carga-masiva.upload-file`, `upload-success` y `upload-error` |
| `DatetimePickerComponent` | `shared/components/datetime-picker/` (878 líneas) | |
| `DecimalInputComponent` | `shared/components/decimal-input/` | Es el único consumidor de `DecimalPrecisionDirective` |
| `ErrorDialogComponent` | `shared/components/error-dialog/` | |
| `MultiselectAutocompleteInputComponent` | `shared/components/multiselect-autocomplete-input/` | |
| `SimpleDialogComponent` | `shared/components/simple-dialog/` | |
| `StatusChipsComponent` | `shared/components/status-chips/` | |
| `SuccessDialogComponent` | `shared/components/success-dialog/` | |
| `TextInputDialogComponent` | `shared/components/text-input-dialog/` | |
| `ThreeOptionDialogComponent` | `shared/components/three-option-dialog/` | |
| `CustomPaginatorIntl` | `shared/services/custom-paginator-intl.service.ts` | Reemplazado por `MatPaginatorIntlTransloco` |
| Pipes `datetimeoffset`, `timezone`, `formatThousands` | `shared/pipes/` | Sin uso en plantillas ni en `.ts` (búsqueda del nombre del pipe) |
| Pipe `lineBreakToBr` | `shared/pipes/line-break-to-br.pipe.ts` | Solo se menciona en un comentario de `shared/constants/line-break-placeholder.ts:3` |
| `TimezoneService` | `core/services/shared/timezone.service.ts` | Solo lo usan los pipes y `DatetimePickerComponent`, que no se usan; nadie llama `setTimezone` |

---

## 7. Directivas compartidas

Carpeta `src/app/shared/directives`.

### 7.1 `*appPermiso` (`PermisoDirective`)

- **Archivo:** `shared/directives/permiso.directive.ts` (selector `[appPermiso]`, standalone). Es la **única directiva compartida en uso**: 47 plantillas la usan, con 85 usos en el árbol actual (`docs/_inventario/app-permiso-usos.csv` es de la línea base y lista 84; el uso nuevo es `*appPermiso="'RegistrarPrecioProveedor'"` en `proveedores-detail.component.html:134`, que documenta la parte de proveedores).
- **Qué hace:** dibuja su plantilla solo si el usuario tiene el permiso; si no lo tiene (o aún no se cargó el perfil), no dibuja nada. Falla cerrada (`permiso.directive.ts:19-25`).
- **Sintaxis de la expresión** (cadena obligatoria, `appPermiso.input.required<string>()`):
  - `'Modulo:Accion'` — por ejemplo `'POS:AperturarCaja'` (usado en el dashboard, §3.1): comprueba ese módulo y esa acción.
  - `'Accion'` — por ejemplo `'Eliminar'`, `'Actualizar'`: el módulo se toma de la URL actual (primer segmento resuelto con `routeModule`, §5.3). Si la URL no tiene módulo mapeado, no se concede.
  - Los espacios alrededor de cada parte se recortan; módulo y acción se comparan sin distinguir mayúsculas; el rol `Admin` lo cumple siempre (`grantsPermission`, `route-permissions.ts:42-48`).
- **Reactiva:** se reevalúa cuando cambia el perfil de permisos (usa `computed` y `effect`).
- **Uso en esta parte:** un solo uso en mi alcance, el botón «Abrir caja» del dashboard (`dashboard.component.html:282`, expresión `'POS:AperturarCaja'`).

### 7.2 Directivas de restricción de escritura — sin uso

Ninguna de estas directivas se importa ni se usa en ninguna plantilla ni `.ts` fuera de `shared/directives` (búsqueda de cada clase y de cada selector en `.html` y `.ts`). El formato de los campos de las pantallas lo imponen validadores de formulario y atributos HTML nativos (`maxlength`, `inputmode`), no estas directivas.

| Directiva | Selector | Qué restringiría al escribir | Uso |
|---|---|---|---|
| `NumericDirective` | `cswNumeric` | Solo dígitos 0-9 (teclas, pegado y entrada); respeta `maxLength`; bloquea Enter | Sin uso |
| `AlphanumericDirective` | `cswAlphanumeric` | Solo letras ASCII y dígitos; filtra al pegar; bloquea Enter | Sin uso |
| `PositiveDecimalDirective` | `cswPositiveDecimal` | Dígitos y un único punto; al pegar exige formato `123` o `123.45`; bloquea Enter | Sin uso |
| `PositiveIntegerDirective` | `cswPositiveInteger` | Enteros positivos; entrada opcional `formatThousands` para mostrar separadores de miles | Sin uso |
| `DecimalPrecisionDirective` | `appDecimalPrecision` | Al salir del campo redondea a `decimals` decimales (2 por defecto) | Solo lo usa `DecimalInputComponent`, que no se usa |
| `BlockManualDateInputDirective` | `blockManualDateInput` | Bloquea toda escritura, pegado y arrastre en el campo de fecha (solo permite Tab, Shift y flechas izquierda/derecha) | Sin uso |
| `ValidNameDirective` | `cswValidName` | Letras (con tildes y ñ), dígitos, espacios y `. , ' -` | Sin uso |
| `ValidDescriptionDirective` | `cswValidDescription` | Letras ASCII sin tildes, dígitos, espacios y `. , ' -`; al pegar quita las tildes y reemplaza los caracteres no permitidos por espacio | Sin uso |

---

## 8. Textos comunes de la raíz de `public/i18n/es.json`

Archivo `public/i18n/es.json` (277 líneas; 241 claves hoja). Las demás partes remiten a estas tablas. «Uso» indica si alguna parte de `src/` referencia la clave literal (búsqueda de la cadena entre comillas en `.ts` y `.html`, sin pruebas); «dinámico» significa que la clave se arma con un valor en tiempo de ejecución.

### 8.1 `nav.*` — opciones del menú lateral (§4.3)

| Clave | Texto | Uso |
|---|---|---|
| `nav.dashboard` | Dashboard | Menú |
| `nav.empresa` | Empresa | Menú |
| `nav.sucursales` | Sucursales | Menú |
| `nav.empleados` | Empleados | Menú |
| `nav.rolesPermisos` | Roles & Permisos | Menú |
| `nav.unidadesMedida` | Unidades de Medida | Menú |
| `nav.jerarquiasUom` | Jerarquías de Unidades | Menú |
| `nav.laboratorios` | Laboratorios | Menú |
| `nav.ingredientesActivos` | Ingredientes Activos | Menú |
| `nav.proveedores` | Proveedores | Menú |
| `nav.aseguradoras` | Aseguradoras | Menú |
| `nav.pacientes` | Pacientes | Menú |
| `nav.profesionalesSalud` | Profesionales de Salud | Menú |
| `nav.catalogoDigemid` | Catálogo DIGEMID | Menú |
| `nav.cuentasUsuario` | Cuentas de Usuario | Menú |
| `nav.medicamentos` | Medicamentos | Menú |
| `nav.presentacionesVenta` | Presentaciones de Venta | Menú |
| `nav.formulacionesClinicas` | Formulaciones Clínicas | Menú |
| `nav.inventario` | Inventario | **Sin uso** |
| `nav.lotes` | Lotes | Menú |
| `nav.stockPorSucursal` | Stock por Sucursal | Menú |
| `nav.sesionesCaja` | Sesiones de Caja | Menú |
| `nav.ventas` | Ventas | Menú |
| `nav.recetasMedicas` | Recetas Médicas | Menú |
| `nav.seriesFiscales` | Series Fiscales | Menú |

### 8.2 `breadcrumbs.*` — migas y títulos de sección del menú

| Clave | Texto | Observación |
|---|---|---|
| `breadcrumbs.home` | Inicio | Miga inicial y título de sección «Inicio» del menú |
| `breadcrumbs.administracion` | Administración | Título de sección del menú y miga |
| `breadcrumbs.catalogos` | Catálogos | Idem |
| `breadcrumbs.digemid` | DIGEMID | Idem |
| `breadcrumbs.pacientes-atencion` | Pacientes & Atención | Variante con guion; el menú usa `pacientesAtencion` |
| `breadcrumbs.empresa` | Empresa | |
| `breadcrumbs.sucursales` | Sucursales | |
| `breadcrumbs.empleados` | Empleados | |
| `breadcrumbs.roles` | Roles & Permisos | |
| `breadcrumbs.unidades-medida` | Unidades de Medida | |
| `breadcrumbs.jerarquias` | Jerarquías de Unidades | |
| `breadcrumbs.laboratorios` | Laboratorios | |
| `breadcrumbs.ingredientes-activos` | Ingredientes Activos | |
| `breadcrumbs.aseguradoras` | Aseguradoras | |
| `breadcrumbs.proveedores` | Proveedores | |
| `breadcrumbs.pacientes` | Pacientes | |
| `breadcrumbs.profesionales-salud` | Profesionales de Salud | |
| `breadcrumbs.digemid-catalogo` | Catálogo DIGEMID | |
| `breadcrumbs.carga-masiva` | Carga Masiva | Última miga de las 12 cargas masivas |
| `breadcrumbs.cuentas` | Cuentas de Usuario | |
| `breadcrumbs.medicamentos` | Medicamentos | |
| `breadcrumbs.presentacionesVenta` | Presentaciones de Venta | |
| `breadcrumbs.formulaciones-clinicas` | Formulaciones Clínicas | |
| `breadcrumbs.inventario` | Inventario | Título de sección del menú |
| `breadcrumbs.lotes` | Lotes | |
| `breadcrumbs.stock-por-sucursal` | Stock por Sucursal | |
| `breadcrumbs.puntoVenta` | Punto de Venta | Título de sección del menú |
| `breadcrumbs.sesionesCaja` | Sesiones de Caja | |
| `breadcrumbs.ventas` | Ventas | |
| `breadcrumbs.pacientesAtencion` | Pacientes & Atención | Título de sección del menú |
| `breadcrumbs.recetasMedicas` | Recetas Médicas | |
| `breadcrumbs.sunat` | SUNAT | Título de sección del menú |
| `breadcrumbs.seriesFiscales` | Series Fiscales | |
| `breadcrumbs.cambiarClave` | Cambiar contraseña | Miga de `/cambiar-clave` (§2) |

Cuáles de estas claves usa cada pantalla lo indica su ficha; todas tienen al menos una referencia literal salvo las que se citan como «sin uso» en la sección «Claves i18n sin uso» (no hay claves sin uso en este grupo según la búsqueda).

### 8.3 `actions.*` — botones comunes

| Clave | Texto | Uso |
|---|---|---|
| `actions.new` | Nuevo | En uso |
| `actions.search` | Buscar | En uso |
| `actions.clear` | Limpiar | En uso |
| `actions.save` | Guardar | En uso |
| `actions.cancel` | Cancelar | En uso |
| `actions.delete` | Eliminar | En uso |
| `actions.edit` | Editar | En uso |
| `actions.view` | Ver | Sin uso |
| `actions.update` | Actualizar | Sin uso |
| `actions.create` | Crear | Sin uso |
| `actions.clean` | Limpiar | Sin uso (duplica `actions.clear`) |
| `actions.export` | Exportar | En uso |
| `actions.return` | Volver | Sin uso |
| `actions.add` | Agregar | Sin uso |
| `actions.close` | Cerrar | En uso (botón de todos los snackbars) |
| `actions.ok` | OK | Sin uso |
| `actions.accept` | Aceptar | En uso |
| `actions.reject` | Rechazar | Sin uso |
| `actions.confirm` | Confirmar | En uso |
| `actions.continue` | Seguir editando | Sin uso |
| `actions.upload-files` | Cargar archivos | Sin uso |
| `actions.download` | Descargar | Sin uso |
| `actions.select` | Seleccionar | Sin uso |
| `actions.back` | Volver | Sin uso |

### 8.4 `general.*`, `generic-labels.*` y `messages.*`

| Clave | Texto | Uso |
|---|---|---|
| `general.errors.required` | Este campo es obligatorio | En uso (21 archivos) |
| `general.errors.minValue` | El valor mínimo es {{min}} | Sin uso |
| `general.errors.maxLength` | La longitud máxima es de {{max}} caracteres | En uso (`cuentas-detail`) |
| `generic-labels.yes` | Sí | En uso |
| `generic-labels.no` | No | En uso |
| `generic-labels.results` | Resultados | En uso |
| `generic-labels.search-subtitle` | Ingresa los criterios de búsqueda | En uso |
| `generic-labels.trace-ref` | Referencia | En uso (§6.11) |
| `generic-labels.copy-ref` | Copiar referencia | En uso (§6.11) |
| `generic-labels.detail-subtitle` | Datos del registro | En uso |
| `generic-labels.no-results` | Sin resultados | En uso |
| `generic-labels.all` | Todos | En uso |
| `generic-labels.date-from` | Fecha desde | En uso (`recetas-search`) |
| `generic-labels.date-to` | Fecha hasta | En uso (`recetas-search`) |
| `messages.enter-mandatory-field` | Por favor, ingresa un campo obligatorio | En uso |

### 8.5 `header.*` — cabecera (claves planas con punto en el nombre)

En el archivo estas claves están escritas planas (`"header.logout": …`), no anidadas (`es.json:111-120`).

| Clave | Texto | Uso |
|---|---|---|
| `header.logout` | Cerrar sesión | Menú de usuario (§4.2) |
| `header.changePassword` | Cambiar contraseña | Menú de usuario |
| `header.profileMenu` | Menú de usuario | `aria-label` del botón de usuario |
| `header.reloadPermissions` | Recargar permisos | **Sin uso** |
| `header.roles` | Roles | **Sin uso** |
| `header.noNotifications` | Sin notificaciones | Menú de la campana |
| `header.notifications` | Notificaciones | **Sin uso** |
| `header.language` | Idioma | `alt` de la bandera |
| `header.version` | Versión | Menú de usuario |
| `header.date-format` | EEEE d 'de' MMMM 'de' y | Fecha de la cabecera |

### 8.6 `datepicker.*`, `pagination.*` y `CONFIRMATION.*`

| Clave | Texto | Uso |
|---|---|---|
| `datepicker.calendar` | Calendario | §6.10 |
| `datepicker.openCalendar` | Abrir calendario | §6.10 |
| `datepicker.closeCalendar` | Cerrar calendario | §6.10 |
| `datepicker.prevMonth` | Mes anterior | §6.10 |
| `datepicker.nextMonth` | Mes siguiente | §6.10 |
| `datepicker.prevYear` | Año anterior | §6.10 |
| `datepicker.nextYear` | Año siguiente | §6.10 |
| `datepicker.prevMultiYear` | 24 años anteriores | §6.10 |
| `datepicker.nextMultiYear` | 24 años siguientes | §6.10 |
| `datepicker.switchToMonthView` | Elegir fecha | §6.10 |
| `datepicker.switchToMultiYearView` | Elegir mes y año | §6.10 |
| `pagination.itemsPerPage` | Items por página | §6.9 |
| `pagination.nextPage` | Siguiente página | §6.9 |
| `pagination.previousPage` | Página anterior | §6.9 |
| `pagination.firstPage` | Primera página | §6.9 |
| `pagination.lastPage` | Última página | §6.9 |
| `pagination.range` | Página {{current}} de {{total}} | §6.9 |
| `pagination.rangeEmpty` | Sin resultados | §6.9 |
| `CONFIRMATION.CHANGE_LANGUAGE_TITLE` | Cambiar idioma | §4.5 |
| `CONFIRMATION.CHANGE_LANGUAGE_MESSAGE` | La interfaz se actualizará al idioma seleccionado. ¿Deseas continuar? | §4.5 |
| `CONFIRMATION.RELOAD_PERMISSIONS_TITLE` | Recargar permisos | **Sin uso** |
| `CONFIRMATION.RELOAD_PERMISSIONS_MESSAGE` | Esto obtendrá nuevamente tus permisos y menús desde el servidor. La aplicación se recargará. ¿Deseas continuar? | **Sin uso** |

### 8.7 `errors.*` — errores comunes

| Clave | Texto | Uso |
|---|---|---|
| `errors.forbidden` | No tienes permisos para esta operación. | **Sin uso** |
| `errors.forbiddenPage` | No tienes permisos para acceder a esta pantalla. | `permissionGuard` (§5.3) |
| `errors.permissionsLoad` | No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión. | `permissionGuard` y menú lateral |
| `errors.unauthorized` | Tu sesión ha expirado. Por favor, inicia sesión de nuevo. | **Sin uso** |
| `errors.notFound` | El recurso solicitado no fue encontrado. | **Sin uso** |
| `errors.serverError` | Ocurrió un error inesperado en el servidor. Por favor, inténtalo de nuevo más tarde. | **Sin uso** |
| `errors.networkError` | Ocurrió un error de red. Por favor, verifica tu conexión a internet e inténtalo de nuevo. | **Sin uso** |
| `errors.unknownError` | Ocurrió un error desconocido. Por favor, inténtalo de nuevo más tarde. | Solo la usa `showSmartErrors`, que no se usa |
| `errors.concurrencyConflict` | Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo. | Mapas de código `Concurrency.Conflict` de varias pantallas (§5.4) |
| `errors.dbDuplicado` | Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo. | `saveErrorKey`, código `Db.Duplicado` |
| `errors.dbReferenciaInvalida` | El registro hace referencia a datos que ya no existen o todavía está en uso. | `saveErrorKey`, código `Db.ReferenciaInvalida` |
| `errors.dbValorInvalido` | Uno de los valores está fuera del rango permitido. | `saveErrorKey`, código `Db.ValorInvalido` |
| `errors.alert.denied_access` | Acceso denegado | **Sin uso** |
| `errors.alert.session` | Sesión | **Sin uso** |
| `errors.alert.warning` | Advertencia | **Sin uso** |
| `errors.alert.error` | Error | **Sin uso** |
| `errors.alert.notFound` | No encontrado | **Sin uso** |

### 8.8 `fields.*` — etiquetas de campos compartidas (columnas de carga masiva y filtros)

| Clave | Texto | Clave | Texto |
|---|---|---|---|
| `fields.codigo` | Código | `fields.codigoIdentificadorNacional` | Código Identificador Nacional |
| `fields.descripcion` | Descripción | `fields.razonSocial` | Razón Social |
| `fields.nombreCompania` | Nombre de Compañía | `fields.taxId` | Tax ID |
| `fields.codigoIdentificador` | Código Identificador | `fields.terminosFinancieros` | Términos Financieros |
| `fields.denominacionDci` | Denominación DCI | `fields.calificacionEsg` | Calificación ESG |
| `fields.codigoAtc` | Código ATC | `fields.tasaCumplimiento` | Tasa de Cumplimiento |
| `fields.nombreRol` | Nombre del Rol | `fields.codProd` | Cód. Producto |
| `fields.nivelJerarquia` | Nivel de Jerarquía | `fields.nomProd` | Nombre del Producto |
| `fields.nombre` | Nombre | `fields.concent` | Concentración |
| `fields.direccionFisica` | Dirección Física | `fields.formaFarmaceutica` | Forma Farmacéutica |
| `fields.numeroLicencia` | Nº de Licencia | `fields.fraccion` | Fracción |
| `fields.ruc` | RUC | `fields.registroSanitario` | Registro Sanitario |
| `fields.docIdentidadGub` | Doc. Identidad | `fields.titular` | Titular |
| `fields.nombreApellidos` | Nombre y Apellidos | `fields.estado` | Estado |
| `fields.fechaNacimiento` | Fecha de Nacimiento | `fields.nombreComercial` | Nombre Comercial |
| `fields.contactoPrimario` | Contacto Primario | `fields.codigoNacional` | Código Nacional |
| `fields.numeroRegistro` | Número de Registro | `fields.laboratorio` | Laboratorio |
| `fields.nombrePrescriptor` | Nombre del Prescriptor | `fields.unidadBase` | Unidad Base |
| `fields.direccionClinica` | Dirección Clínica | `fields.condicionVenta` | Condición de Venta |
| `fields.producto` | Producto | `fields.unidadMayor` | Unidad Mayor |
| `fields.unidadMenor` | Unidad Menor | `fields.multiplicador` | Multiplicador |

Todas tienen al menos una referencia literal (3 o más archivos en su mayoría).

### 8.9 `carga-masiva.*`

| Clave | Texto | Uso |
|---|---|---|
| `carga-masiva.download-template` | Descargar plantilla | Carga masiva (§6.5) |
| `carga-masiva.upload-file` | Subir archivo | Solo componente sin uso |
| `carga-masiva.upload-success` | Archivo cargado correctamente. | Solo componente sin uso |
| `carga-masiva.upload-error` | Error al cargar el archivo. Verifica el formato e intenta de nuevo. | Solo componente sin uso |
| `carga-masiva.download-error` | Error al descargar la plantilla. Intenta de nuevo. | En uso |
| `carga-masiva.template-filename` | plantilla-{{entity}}.xlsx | En uso |
| `carga-masiva.title` | Carga Masiva | En uso |
| `carga-masiva.btn` | Carga Masiva | Botón de 12 pantallas de búsqueda |
| `carga-masiva.upload-zone` | Arrastra tu archivo Excel aquí o haz clic para seleccionar | En uso |
| `carga-masiva.upload-zone-hint` | Solo archivos .xlsx (máx. {{maxMb}} MB y {{maxRows}} filas) | En uso |
| `carga-masiva.preview-title` | Vista previa | En uso |
| `carga-masiva.valid-rows` | {{count}} filas válidas | En uso |
| `carga-masiva.error-rows` | {{count}} filas con error | En uso |
| `carga-masiva.total-rows` | {{count}} filas totales | En uso |
| `carga-masiva.save-valid` | Guardar válidos ({{count}}) | En uso |
| `carga-masiva.row-number` | Fila | **Sin uso** |
| `carga-masiva.row-status` | Estado | En uso |
| `carga-masiva.status-valid` | Válido | **Sin uso** |
| `carga-masiva.status-error` | Error | **Sin uso** |
| `carga-masiva.save-success` | Se guardaron {{count}} registros correctamente | En uso |
| `carga-masiva.save-error` | Error al guardar los registros | En uso |
| `carga-masiva.preview-error` | Error al procesar el archivo | En uso |
| `carga-masiva.no-valid-rows` | No hay filas válidas para guardar | En uso |
| `carga-masiva.loading-preview` | Procesando archivo... | En uso |
| `carga-masiva.saving` | Guardando... | **Sin uso** |

**Errores por fila** `carga-masiva.validation.<code>` (dinámicas: el código lo arma el backend; `carga-masiva-page.component.html:95`). Ninguna figura como literal en `src/`; todas se consideran en uso por el mecanismo dinámico:

| Código | Texto |
|---|---|
| `required` | El campo '{{field}}' es obligatorio |
| `max-length` | '{{field}}' no puede superar los {{max}} caracteres |
| `duplicate` | El valor '{{value}}' del campo '{{field}}' ya existe en el sistema |
| `duplicate-in-file` | El valor '{{value}}' del campo '{{field}}' está duplicado en el archivo |
| `invalid-integer` | '{{field}}' debe ser un número entero |
| `min-value` | '{{field}}' debe ser mayor o igual a {{min}} |
| `invalid-date` | El formato de '{{field}}' no es válido (usa DD/MM/YYYY) |
| `future-date` | '{{field}}' no puede ser una fecha futura |
| `invalid-decimal` | '{{field}}' debe ser un número decimal válido |
| `invalid-range` | '{{field}}' debe estar entre {{min}} y {{max}} |
| `invalid-digits` | '{{field}}' debe contener exactamente {{digits}} dígitos numéricos |
| `not-found` | El valor '{{value}}' del campo '{{field}}' no existe en el sistema |
| `invalid-option` | '{{value}}' no es un valor válido para '{{field}}' |
| `positive-number` | '{{field}}' debe ser mayor que cero |
| `must-differ` | '{{field}}' no puede ser igual a '{{other}}' |
| `ambiguous` | El valor '{{value}}' del campo '{{field}}' coincide con varios registros del sistema |
| `invalid-document-number` | '{{field}}' no tiene el formato de su tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números) |

Qué códigos emite realmente cada plantilla del backend no se verificó aquí (ver §9).

**Errores de archivo** `carga-masiva.errors.*`: `invalid-file`, `too-large`, `decompression`, `too-many-rows`, `invalid-rows` (textos en §6.5).

### 8.10 `tipos-documento.*`

Se resuelven dinámicamente con `tipos-documento.<tipo>` (`src/app/shared/utils/tipo-documento.util.ts:15`).

| Clave | Texto |
|---|---|
| `tipos-documento.1` | DNI |
| `tipos-documento.4` | Carné de extranjería |
| `tipos-documento.6` | RUC |
| `tipos-documento.7` | Pasaporte |
| `tipos-documento.-` | Sin documento |


---

## 9. Pendientes y «No verificado»

**En curso (sin commitear)**

Ninguna. Al 2026-10-14 no hay cambios sin commitear en `sofia-apis` (`43f5529`) ni en `sofia-web-spa` (`eeffbb6`). La marca de la versión 1.0 sobre `ArqueoCajaCalculator.cs` se retiró porque ese cambio ya está commiteado (`358935e`).

**No verificado**

| Tema | Motivo |
|---|---|
| Vigencia real del enlace de recuperación («válido por 1 hora», `auth.forgotPage.sent`, §1.3) | Es texto de la traducción; no se leyó el backend de `ForgotPassword` |
| Separador decimal y de miles de las cifras del dashboard (§3.3) | Deducido de que no hay proveedor de `LOCALE_ID`; no se vio en pantalla (no se usó el navegador) |
| Códigos de validación que emite cada plantilla de carga masiva (`carga-masiva.validation.*`, §8.9) | Corresponde al documento 20 / backend |
| Contenido de `layouts/full/horizontal/sidebar/sidebar-data.ts` (§4.7) | El layout horizontal no se muestra a ningún usuario |
| Aspecto visual (colores, posición exacta) de cabecera, menú y tarjetas del dashboard | Solo se leyó el código; el CSS se consultó puntualmente (§4.1) |
| Qué endpoints envían `traceId` en sus errores (§6.12) | Se documenta lo que lee el SPA, no lo que envía cada endpoint |
| Que la dirección de la sede principal aparezca realmente en el encabezado de las boletas (texto de ayuda `empresa.register.direccionSedeHint`, §1.2) | Es texto de la traducción; no se revisó la impresión del comprobante en esta parte |

**Pendientes / hallazgos para otras partes**
- `showSmartErrors` y las claves `errors.forbidden`, `errors.unauthorized`, `errors.notFound`, `errors.serverError`, `errors.networkError`, `errors.alert.*` no tienen ningún uso (confirmado, §6.12, §8.7).
- Componentes, directivas y pipes compartidos sin uso: ver §6.13 y §7.2. En particular, **ninguna** directiva de restricción de escritura (`cswNumeric`, `cswAlphanumeric`, etc.) se usa en las pantallas.
- El idioma elegido no se guarda; el tema no es configurable; la campana de notificaciones es decorativa (§4.1, §4.5).
- El scope i18n `login` completo está sin uso (§1.1).

## 10. Tabla de cobertura

**Rutas de `rutas-spa.csv` de mi alcance (6 de 6)**

| Ruta | Componente | Documentada en |
|---|---|---|
| `/auth/login` | `LoginComponent` | §1.1 |
| `/auth/register` | `EmpresaRegisterComponent` | §1.2 |
| `/auth/forgot-password` | `ForgotPasswordComponent` | §1.3 |
| `/auth/reset-password` | `ResetPasswordComponent` | §1.4 |
| `/dashboard` | `DashboardComponent` | §3 |
| `/cambiar-clave` | `CambiarClaveComponent` | §2 |

**Componentes de `componentes-pages.txt` de mi alcance (7 de 7)**

| Componente | Documentado en |
|---|---|
| `pages/auth/login` | §1.1 |
| `pages/auth/forgot-password` | §1.3 |
| `pages/auth/reset-password` | §1.4 |
| `pages/auth/auth-brand-panel` | §1.5 |
| `pages/empresa/empresa-register` | §1.2 |
| `pages/cambiar-clave` | §2 |
| `pages/dashboard` | §3 |

**Diálogos de `dialogos.txt` de mi alcance**

| Diálogo | Estado | Documentado en |
|---|---|---|
| `shared/components/confirm-dialog` | En uso | §6.1 |
| `shared/components/error-dialog` | Sin uso (comprobado) | §6.13 |
| `shared/components/simple-dialog` | Sin uso (comprobado) | §6.13 |
| `shared/components/success-dialog` | Sin uso (comprobado) | §6.13 |
| `shared/components/text-input-dialog` | Sin uso (comprobado) | §6.13 |
| `shared/components/three-option-dialog` | Sin uso (comprobado) | §6.13 |
| Los 5 diálogos de `pages/pos/dialogs` | Fuera de mi alcance (parte de punto de venta) | — |

**Usos de `*appPermiso` de mi alcance (1 de 1)**

| Archivo:línea | Expresión | Elemento | Documentado en |
|---|---|---|---|
| `pages/dashboard/dashboard.component.html:282` | `'POS:AperturarCaja'` | Botón «Abrir caja» | §3.1 (bloque Caja), §7.1 |

**Otros elementos del alcance**

| Elemento | Documentado en |
|---|---|
| Cabecera (idioma, menú de usuario, campana, fecha; sin buscador ni tema) | §4.1, §4.2, §4.5 |
| Menú lateral completo (8 grupos, 24 opciones) y `visible-nav-items.ts` | §4.3 |
| Cierre de sesión | §4.4 |
| Personalizador de tema (inactivo) y layout horizontal (sin uso) | §4.6, §4.7 |
| Pie de página | §4 (no existe) |
| `authGuard`, `passwordChangeGuard`, `permissionGuard`, `route-permissions.ts` | §5.1-§5.3 |
| `authInterceptor`, `CSLoadingInterceptor`, renovación en segundo plano | §5.4, §5.5 |
| `carga-masiva-page` y sus 12 envoltorios | §6.5, §6.6 |
| `confirm-dialog`, `page-header`, `autocomplete-input`, `range-slider-field`, `loading-bar`, `server-error-snackbar`, paginador y datepicker | §6.1-§6.4, §6.8-§6.11 |
| `ErrorNotifierService` (4 métodos) | §6.12 |
| Componentes compartidos sin uso (11 componentes, `CustomPaginatorIntl`, pipes, `TimezoneService`) | §6.13 |
| Directivas (`*appPermiso` y 8 de restricción) | §7 |
| Textos de la raíz de `es.json` (`nav`, `breadcrumbs`, `actions`, `general`, `generic-labels`, `messages`, `header.*`, `datepicker`, `pagination`, `CONFIRMATION`, `errors`, `fields`, `carga-masiva`, `tipos-documento`) | §8.1-§8.10 |
| Scopes `auth` (42 claves), `cambiar-clave` (12), `dashboard` (71), `empresa` (parte de registro) | §1, §2, §3 |

**Claves i18n sin uso**

| Scope | Claves sin ninguna referencia en `src/` | Observación |
|---|---|---|
| `auth` | Ninguna | Las 42 claves se usan |
| `cambiar-clave` | Ninguna | |
| `dashboard` | Ninguna | `metodo.Efectivo`, `metodo.YapePlin`, `metodo.Tarjeta` y `metodo.Transferencia` se resuelven con `'metodo.' + MetodoPago[...]` (`dashboard.component.html:147`) |
| `empresa` (parte de registro: `register.*` incluidas `register.direccionSede`, `register.direccionSedePlaceholder` y `register.direccionSedeHint`; validaciones de `form.*` incluida `form.maxLength255`; `notifications.register*`, `rucDuplicado`, `usuarioDuplicado`, `direccionSedeInvalida`) | Ninguna (las 5 claves nuevas de la versión 1.1 están en uso: `empresa-register.component.html:80,85,87,95` y `.ts:25`) | Para las pantallas de búsqueda y detalle de empresa, las claves sin referencia literal (candidatas para la parte que las cubre) son: `searchPlaceholder`, `estado.0` a `estado.3`, `cols.fechaVencimiento`, `cols.cantidadSucursales`, `form.titleView`, `form.estado`, `form.fechaVencimiento`, `notifications.loadError`, `help.nombre`, `help.ruc`, `help.estado`, `noData`, `showing`, `detail.title-edit`, `fields.estaVigente`; algunas pueden resolverse de forma dinámica |
| `login` | Las 8 claves (`title`, `subtitle`, `username`, `password`, `usernameRequired`, `passwordRequired`, `submit`, `error`) | Scope completo sin uso (§1.1) |
| Raíz | Ver §8: `nav.inventario`; `actions.view/update/create/clean/return/add/ok/reject/continue/upload-files/download/select/back`; `general.errors.minValue`; `header.reloadPermissions/roles/notifications`; `CONFIRMATION.RELOAD_PERMISSIONS_*`; `errors.forbidden/unauthorized/notFound/serverError/networkError/alert.*`; `carga-masiva.row-number/status-valid/status-error/saving` (y `upload-file/upload-success/upload-error` solo en un componente sin uso) | `errors.unknownError` solo la usa `showSmartErrors` |

## 11. Fuentes

SPA (`sofia-web-spa/`):
- Rutas y arranque: `src/app/app.routes.ts`, `src/app/app.config.ts`, `src/app/app.component.*`, `src/environments/environment.ts`, `environment.prod.ts`, `environment.model.ts`, `src/index.html`, `public/environment.js`, `angular.json`.
- Núcleo: `src/app/core/auth/{auth.service,auth.guard,password-change.guard,permission.guard,route-permissions,user-storage}.ts`, `src/app/core/interceptors/auth.interceptor.ts`, `src/app/core/services/shared/{error-notifier,search-state,timezone}.service.ts`, `src/app/core/services/logger.service.ts`, `src/app/core/responses/common-error-response.ts`, `src/app/core/transloco/transloco-loader.ts`.
- Pantallas: `src/app/pages/auth/**`, `src/app/pages/cambiar-clave/**`, `src/app/pages/dashboard/**` (componente, plantilla, servicio, modelo, `dashboard-links.ts`), `src/app/pages/empresa/empresa-register/**`, `src/app/pages/empresa/services/empresa.service.ts`, `src/app/pages/empresa/models/empresa.model.ts`, `src/app/pages/*/*-carga-masiva/*.component.ts`, `src/app/pages/*/*.routes.ts` (rutas de carga masiva), plantillas de `cuentas-detail`, `sucursales-detail`, `jerarquias-search`, `proveedores-search`, `roles-search`.
- Layout: `src/app/matdash/layouts/full/**` (full, vertical/header, vertical/sidebar, nav-item, branding, shared/breadcrumb, shared/customizer, horizontal/header), `src/app/matdash/{config.ts,services/core.service.ts,services/nav.service.ts}`.
- Compartidos: `src/app/shared/components/**`, `src/app/shared/directives/**`, `src/app/shared/pipes/**`, `src/app/shared/services/**`, `src/app/shared/date/**`, `src/app/shared/utils/{save-error-key,tipo-documento}.util.ts`, `src/app/shared/models/carga-masiva.model.ts`.
- Textos: `public/i18n/es.json`, `public/i18n/{auth,cambiar-clave,dashboard,empresa,login}/es.json`.

Backend (`sofia-apis/src/`, solo para describir los indicadores y los permisos de endpoint): `SOFIA.API/Controllers/DashboardController.cs`, `SOFIA.Application/Dashboard/**` (handlers de ventas, caja, inventario, cuarentena y devoluciones; `DashboardScopeResolver.cs`, `DashboardDias.cs`), `SOFIA.Application/POS/Common/ArqueoCajaCalculator.cs`; para el registro de empresa: `SOFIA.Application/Empresas/Commands/RegistrarEmpresa/{RegistrarEmpresaCommand,RegistrarEmpresaCommandHandler,RegistrarEmpresaCommandValidator}.cs`, `SOFIA.Domain/Entities/Sucursal.cs`, `SOFIA.Application/Common/Behaviors/ValidationBehavior.cs`.

Solo pruebas (sin pantalla): `src/app/shared/testing/delete-in-use-testing.ts` (ayuda para las pruebas de eliminación en uso, agregada en `01b8260`); no forma parte de la interfaz.

Inventarios base: `docs/_inventario/{rutas-spa.csv,componentes-pages.txt,dialogos.txt,app-permiso-usos.csv,endpoints.csv,i18n-scopes.csv}`.
