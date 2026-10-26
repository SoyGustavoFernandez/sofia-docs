# Historias de usuario — Parte A: acceso y administración

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-26 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de los documentos validados (reglas de negocio 01a v1.1, inventario de pantallas 19a y 19b v1.1, matriz de permisos 05b, autenticación 05a, manual de usuario capítulos 1 a 3, glosario) y comprobación puntual en el código. No se ejecutó la aplicación ni se llamó a la API: los mensajes exactos salen de los documentos fuente, que los comprobaron en la aplicación local. |

## 1. Cómo leer este documento

Este documento reúne las historias de usuario de la **parte A**: registro de la empresa, acceso al sistema, menú y permisos de pantalla, y administración (empresa, sucursales, empleados, roles y permisos, cuentas de usuario y series fiscales). Las otras partes viven en la misma carpeta: la parte B cubre catálogos, inventario y atención, y la parte C punto de venta, caja, devoluciones y dashboard.

**Convenciones**

- Los IDs son `HU-ADM-nnn`. Cada historia es una capacidad de negocio, no un botón.
- Las reglas `RN-…` son las del [catálogo de reglas de negocio, parte A](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) (versión 1.1). En cada historia, las reglas de un mismo prefijo se enlazan juntas.
- Los textos entre « » son los que ve el usuario, copiados de los `es.json` a través de los documentos fuente. Si el sistema no tiene un mensaje propio para un caso, el escenario dice cuál es el mensaje genérico que se ve.
- **Estado.** «Implementada»: hay pantalla (o endpoint, si la historia es de API) y funciona como describen los escenarios. «Parcial»: falta algo, y la historia lo dice. «Solo API»: existe el endpoint y ninguna pantalla lo usa.
- **Brecha conocida.** Cuando el sistema hoy se comporta distinto de lo que el negocio esperaría (por ejemplo, un mensaje genérico donde debería haber uno específico), el escenario describe el comportamiento **actual** y una nota «Brecha conocida» cita la fuente. Ningún escenario da por cumplido algo que el sistema no hace.
- **No verificado en la aplicación.** Algunos escenarios dependen de casos que los recorridos del manual no pudieron provocar (por ejemplo, el límite de peticiones). Se marcan así y la fuente de la regla es el código leído en el catálogo de reglas.
- Los permisos se escriben `Módulo:Acción` como los exige el endpoint (por ejemplo `Seguridad:Leer`). En la matriz de permisos de la pantalla de roles, el módulo `Seguridad` se rotula «Roles y Cuentas de Usuario», `Empresas` «Empresa» y `POS` «Sesiones de Caja».
- Los datos de los ejemplos son los ficticios del manual: la empresa «Botica Manual Demo S.A.C.» (RUC `20538291746`), la administradora «Ana Demo Prueba» (usuario `ana.manual`), el cajero «Carlos Cajero Demo» (usuario `carlos.cajero`) y las sucursales «Sede Principal» y «Sucursal Demo Norte».

**Actores**

| Actor | Quién es |
|---|---|
| Visitante | Persona sin sesión: registra su empresa o recupera su contraseña. |
| Usuario autenticado | Cualquier persona con sesión iniciada; no necesita permisos para cambiar su propia contraseña ni para cerrar sesión. |
| Administrador | Usuario con el rol llamado `Admin`: pasa cualquier comprobación de permiso ([RN-GEN-021](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)). |
| Cajero (u otro rol limitado) | Usuario cuyo rol tiene solo algunos permisos; el rol «Cajero» del manual se crea en la historia [HU-ADM-028](#hu-adm-028--crear-un-rol) y se completa en [HU-ADM-030](#hu-adm-030--definir-los-permisos-de-un-rol). |
| Integrador / equipo técnico | Quien consume la API directamente o la opera (historias de plataforma, sección 11). |

Los flujos de punta a punta (alta de empresa y configuración inicial) están en el [documento de procesos de negocio](../14-procesos-de-negocio.md), procesos 1 y 2; estas historias son su desglose por capacidad.

## 2. Índice

| Sección | Historias |
|---|---|
| 3. Registro y acceso | HU-ADM-001 a HU-ADM-009 |
| 4. Menú, permisos de pantalla y cabecera | HU-ADM-010 a HU-ADM-012 |
| 5. Empresa y suscripción | HU-ADM-013 a HU-ADM-016 |
| 6. Sucursales | HU-ADM-017 a HU-ADM-022 |
| 7. Empleados | HU-ADM-023 a HU-ADM-026 |
| 8. Roles y permisos | HU-ADM-027 a HU-ADM-033 |
| 9. Cuentas de usuario | HU-ADM-034 a HU-ADM-039 |
| 10. Series fiscales | HU-ADM-040 a HU-ADM-044 |
| 11. Reglas transversales y de plataforma | HU-ADM-045 a HU-ADM-050 |

---

## 3. Registro y acceso

### HU-ADM-001 — Registrar la empresa y su administrador

**Como** visitante **quiero** registrar mi botica o farmacia y crear a su administrador **para** empezar a usar SOFIA con un mes de prueba gratuito, sin pedir ayuda a nadie.

| | |
|---|---|
| **Módulo** | Empresas (registro público) |
| **Pantalla** | `/auth/register` — «Registrar Empresa» |
| **Endpoint(s)** | `POST /api/v1/empresas/registrar` |
| **Permiso** | Anónimo (política de límite `SignUpPolicy`: 5 peticiones por hora por IP) |
| **Reglas** | [RN-EMP-001, RN-EMP-002, RN-EMP-003, RN-EMP-005, RN-EMP-006, RN-EMP-007, RN-EMP-008, RN-EMP-009, RN-EMP-010, RN-EMP-011, RN-EMP-012, RN-EMP-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-007](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-004, RN-GEN-027](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar la empresa con datos válidos
  Dado que soy un visitante sin sesión
  Y estoy en "/auth/register" con el formulario "Registrar Empresa"
  Cuando escribo Nombre de la Empresa "Botica Manual Demo S.A.C."
  Y RUC "20538291746"
  Y Dirección de la sede principal "Av. Ficticia 123, Miraflores, Lima"
  Y Nombres del administrador "Ana", Apellido paterno "Demo", Apellido materno "Prueba"
  Y Correo electrónico "ana.demo@sofia-manual.test"
  Y Usuario Administrador "ana.manual" y una Contraseña de 8 caracteres o más
  Y hago clic en "Registrar Empresa"
  Entonces el botón muestra "Procesando..." mientras el sistema trabaja
  Y veo el aviso "Empresa registrada correctamente. ¡Bienvenido!"
  Y entro directamente al Inicio ("/dashboard") con la sesión ya iniciada
  Y no se me pide volver a escribir usuario y contraseña ni se envía un correo de verificación

Escenario: El registro crea todo lo necesario en una sola operación
  Dado que el registro terminó correctamente
  Entonces existe la empresa en estado "Trial Activo", vigente, con inicio de prueba en el momento del registro y vencimiento 30 días después
  Y existe una sola sucursal llamada "Sede Principal" con la dirección escrita y el "Nº de Licencia" vacío
  Y existe el empleado "Ana Demo Prueba" con el correo escrito
  Y existe la cuenta "ana.manual", que no me obliga a cambiar la contraseña al entrar
  Y existe el rol "Admin" con la descripción "Administrador de la empresa", nivel de jerarquía 0, asignado a mi cuenta
  Y todos esos registros llevan el identificador de la nueva empresa

Escenario: Si algo falla al registrar, no se crea nada
  Dado que el registro falla a mitad de la operación
  Entonces no queda creada la empresa, ni la sede, ni el empleado, ni la cuenta, ni el rol

Escenario: Campos obligatorios vacíos
  Dado que el formulario está vacío
  Cuando hago clic en "Registrar Empresa"
  Entonces cada campo obligatorio muestra "Este campo es obligatorio"
  Y no se envía nada al servidor

Esquema del escenario: Validaciones de cada campo
  Dado que estoy en "/auth/register"
  Cuando escribo <valor> en <campo> y salgo del campo
  Entonces veo debajo del campo el mensaje <mensaje>

  Ejemplos:
    | campo                      | valor                                  | mensaje                                            |
    | Nombre de la Empresa       | 201 caracteres                         | Máximo 200 caracteres                              |
    | RUC                        | 2053829174 (10 dígitos)                | El RUC debe tener exactamente 11 dígitos numéricos |
    | RUC                        | una letra entre los dígitos            | El RUC debe tener exactamente 11 dígitos numéricos |
    | Dirección de la sede principal | solo espacios                      | Este campo es obligatorio                          |
    | Dirección de la sede principal | 256 caracteres                     | Máximo 255 caracteres                              |
    | Nombres del administrador  | 76 caracteres                          | Máximo 75 caracteres                               |
    | Apellido paterno           | 76 caracteres                          | Máximo 75 caracteres                               |
    | Apellido materno (opcional)| 76 caracteres                          | Máximo 75 caracteres                               |
    | Correo electrónico         | ana.demo                               | Ingresa un correo electrónico válido               |
    | Correo electrónico         | 255 caracteres                         | Máximo 254 caracteres                              |
    | Usuario Administrador      | ana (3 caracteres)                     | El usuario debe tener al menos 4 caracteres        |
    | Usuario Administrador      | 51 caracteres                          | Máximo 50 caracteres                               |
    | Contraseña                 | abc1234 (7 caracteres)                 | La contraseña debe tener al menos 8 caracteres     |

Escenario: El apellido materno es opcional y la contraseña solo exige 8 caracteres
  Dado que dejo vacío el "Apellido materno (opcional)"
  Y escribo una contraseña de 8 caracteres sin mayúsculas, números ni símbolos
  Cuando hago clic en "Registrar Empresa"
  Entonces el formulario no marca error en esos campos
  Y el registro continúa

Escenario: La dirección de la sede principal es obligatoria
  Dado que escribo solo espacios en "Dirección de la sede principal"
  Cuando hago clic en "Registrar Empresa"
  Entonces veo "Este campo es obligatorio" y no se envía nada
  Y bajo el campo se lee la ayuda "Aparecerá en el encabezado de tus boletas"

Escenario: El RUC ya está registrado por otra empresa
  Dado que ya existe una empresa con el RUC "20538291746"
  Cuando intento registrar otra empresa con ese RUC
  Entonces veo el aviso "Ya existe una empresa registrada con este RUC"
  Y no se crea nada

Escenario: El usuario ya existe en cualquier empresa
  Dado que otra empresa ya tiene la cuenta "ana.manual"
  Cuando registro mi empresa con el usuario "ana.manual"
  Entonces veo el aviso "El usuario ya está en uso. Elige otro."
  # No verificado en la aplicación: el registro tiene límite de envíos por hora

Escenario: La API rechaza la dirección de la sede
  Dado que el formulario dejó pasar una dirección vacía o de más de 255 caracteres
  Cuando el servidor la rechaza
  Entonces veo el aviso "Ingresa la dirección de la sede principal (máximo 255 caracteres)"
  # No verificado en la aplicación: el formulario normalmente lo impide antes

Escenario: Cualquier otro rechazo del servidor o exceso de envíos
  Dado que ya hice 5 registros en la última hora desde el mismo equipo
  Cuando intento un registro más
  Entonces veo el aviso "Error al registrar la empresa"
  Y el mismo aviso aparece si el servidor no responde o rechaza otro dato (nombre, RUC, usuario, contraseña o datos del administrador)
```

> **Brecha conocida.** El servidor arma el código de error de validación con la **primera** regla que falla, y la pantalla solo traduce `RegistrarEmpresaCommand.DireccionSede`; cualquier otra validación rechazada por el servidor se ve como «Error al registrar la empresa» (fuente: inventario 19a §1.2). La API acepta además `NombreSede` y `NumeroLicencia` opcionales que este formulario no usa; por eso la sede nace como «Sede Principal» y sin licencia, que se completa después ([HU-ADM-019](#hu-adm-019--ver-y-editar-una-sucursal)). La pantalla no pide repetir la contraseña (manual cap. 1, §3.3).

### HU-ADM-002 — Iniciar sesión

**Como** usuario con una cuenta **quiero** entrar a SOFIA con mi usuario y contraseña **para** trabajar en mi empresa con los permisos de mi rol.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | `/auth/login` — «Iniciar sesión» |
| **Endpoint(s)** | `POST /api/v1/auth/login` |
| **Permiso** | Anónimo (política `AuthPolicy`: 10 peticiones por minuto por IP) |
| **Reglas** | [RN-AUT-001, RN-AUT-002, RN-AUT-003, RN-AUT-004, RN-AUT-008, RN-AUT-009, RN-AUT-010, RN-AUT-011, RN-AUT-012, RN-AUT-031](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-026, RN-GEN-040](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Inicio de sesión correcto
  Dado que la cuenta "ana.manual" está activa, no bloqueada y su empresa está vigente
  Cuando escribo Usuario "ana.manual" y la Contraseña correcta y hago clic en "Ingresar"
  Entonces el botón muestra un círculo de espera y no puedo volver a pulsarlo
  Y entro al Inicio ("/dashboard") sin ningún aviso
  Y si recargo la página con la sesión vigente, sigo dentro

Escenario: No se pide la empresa
  Dado que el nombre de usuario es único en todo el sistema
  Cuando inicio sesión
  Entonces el formulario solo tiene los campos "Usuario" y "Contraseña"
  Y el sistema encuentra mi cuenta en cualquier empresa

Escenario: El token y el token de refresco
  Dado que inicio sesión correctamente
  Entonces recibo un token de acceso con mi id de cuenta, usuario, nombre completo, nombre de empresa, empleado, sucursal base, sello de seguridad y empresa
  Y el token lleva un claim por cada rol vivo de mi misma empresa
  Y el token de refresco no viaja en el cuerpo: llega en la cookie "refresh_token", HttpOnly, Secure, SameSite=None, con vigencia de 7 días
  Y en la base de datos solo se guarda el hash SHA-256 del token de refresco

Escenario: Cuenta que debe cambiar la contraseña
  Dado que mi cuenta tiene "Requiere Cambio de Clave" en "Sí"
  Cuando inicio sesión correctamente
  Entonces el sistema me lleva a "/cambiar-clave" en lugar del Inicio

Escenario: Usuario y contraseña obligatorios
  Dado que dejo vacíos los campos
  Cuando hago clic en "Ingresar"
  Entonces veo "El usuario es obligatorio" y "La contraseña es obligatoria"
  Y no se envía nada al servidor

Escenario: Usuario de más de 50 caracteres
  Dado que llamo a "POST /api/v1/auth/login" con un usuario de 51 caracteres
  Entonces la API responde 400 "LoginCommand.NombreUsuario"
  Y la pantalla muestra "Credenciales inválidas. Verifica tu usuario y contraseña." porque no tiene mapa para ese código

Escenario: Contraseña incorrecta
  Dado que escribo una contraseña que no es la de "ana.manual"
  Cuando hago clic en "Ingresar"
  Entonces veo en una alerta roja "Credenciales inválidas. Verifica tu usuario y contraseña."
  Y me quedo en "/auth/login"

Escenario: El sistema no revela por qué falló el ingreso
  Dado que el usuario no existe, o la cuenta está eliminada, o está desactivada, o está bloqueada, o la sucursal base de mi empleado es de otra empresa
  Cuando intento iniciar sesión
  Entonces veo en todos los casos "Credenciales inválidas. Verifica tu usuario y contraseña."
  Y la contraseña se verifica siempre con BCrypt, contra el hash real o contra uno ficticio, para que el tiempo de respuesta no delate si la cuenta existe

Escenario: La empresa no tiene suscripción vigente
  Dado que la contraseña es correcta
  Pero la empresa está suspendida, cancelada o con la prueba vencida
  Cuando hago clic en "Ingresar"
  Entonces veo "La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla."
  # No verificado en la aplicación: la empresa de prueba está vigente

Escenario: La vigencia solo se comprueba con la contraseña correcta
  Dado que la empresa no está vigente
  Cuando escribo una contraseña incorrecta
  Entonces veo "Credenciales inválidas. Verifica tu usuario y contraseña."
  Y no se me indica que la empresa no está vigente

Escenario: Demasiados intentos desde el mismo equipo
  Dado que ya hice 10 intentos de inicio de sesión en el último minuto desde la misma IP
  Cuando hago otro intento
  Entonces veo "Demasiados intentos. Espera un minuto e inténtalo de nuevo."
  # No verificado en la aplicación: no se provocó para no bloquear la cuenta de prueba

Escenario: El servidor no responde
  Dado que la API no responde, agota el tiempo o devuelve un error de servidor
  Cuando hago clic en "Ingresar"
  Entonces veo "No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos."
  # No verificado en la aplicación

Escenario: Contraseñas guardadas de forma segura
  Dado que existe una cuenta
  Entonces su contraseña nunca se guarda en claro: se guarda un hash BCrypt
```

> **Brecha conocida.** El sistema no tiene «recordarme» ni vuelve a la pantalla donde estabas: siempre entra al Inicio o a «Cambiar contraseña» (inventario 19a §1.1). Un 4xx sin mapa, por ejemplo un validador, también se ve como «Credenciales inválidas…» (RN-AUT-001 y RN-AUT-002).

### HU-ADM-003 — Bloquear la cuenta tras intentos fallidos

**Como** administrador de la empresa **quiero** que una cuenta se bloquee sola tras varias contraseñas erróneas **para** impedir que alguien adivine la clave de mis empleados.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | `/auth/login` (el bloqueo no se avisa en pantalla) y `/cuentas/:id` (para levantarlo, ver [HU-ADM-036](#hu-adm-036--administrar-el-estado-de-una-cuenta)) |
| **Endpoint(s)** | `POST /api/v1/auth/login` |
| **Permiso** | Anónimo |
| **Reglas** | [RN-AUT-005, RN-AUT-006, RN-AUT-007, RN-AUT-010, RN-AUT-039](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (el SPA no informa del bloqueo, por diseño) |

**Criterios de aceptación**

```gherkin
Escenario: Cada contraseña errónea suma un intento
  Dado que la cuenta "carlos.cajero" tiene 0 intentos fallidos
  Cuando escribo una contraseña incorrecta
  Entonces veo "Credenciales inválidas. Verifica tu usuario y contraseña."
  Y la cuenta pasa a 1 intento fallido
  Y el intento queda guardado aunque la petición termine en error

Escenario: Cinco intentos fallidos bloquean la cuenta 15 minutos
  Dado que la cuenta "carlos.cajero" tiene 4 intentos fallidos
  Cuando fallo el quinto intento
  Entonces la cuenta queda bloqueada durante 15 minutos
  Y en "Cuentas de Usuario" la cuenta se ve con el candado cerrado, "Intentos" en 5 y, en su ficha, "Bloqueado: Sí" y la fecha de "Bloqueado Hasta"

Escenario: Una cuenta bloqueada no entra ni con la contraseña correcta
  Dado que la cuenta está bloqueada
  Cuando escribo la contraseña correcta
  Entonces veo "Credenciales inválidas. Verifica tu usuario y contraseña."
  Y el intento no suma al contador
  Y la pantalla no avisa del bloqueo

Escenario: El bloqueo vencido reinicia el contador
  Dado que el bloqueo anterior ya venció
  Cuando fallo una contraseña
  Entonces el contador vuelve a 0 antes de sumar el nuevo fallo y queda en 1

Escenario: Un ingreso correcto reinicia el contador
  Dado que la cuenta tiene 3 intentos fallidos
  Cuando inicio sesión con la contraseña correcta
  Entonces los intentos vuelven a 0 y no queda bloqueo

Escenario: Formas de desbloquear una cuenta
  Dado que la cuenta está bloqueada
  Cuando un administrador marca "Resetear Intentos Fallidos" y guarda
  Entonces los intentos quedan en 0 y "Bloqueado: No"
  Pero también se desbloquea si la persona restablece su contraseña por el enlace del correo ([HU-ADM-009](#hu-adm-009--restablecer-la-contraseña-con-el-enlace))
```

> **Nota.** El SPA muestra el mismo mensaje para todo fallo a propósito, para no confirmar que la cuenta existe (RN-AUT-005; manual cap. 1, §6.5). Un cambio de contraseña voluntario con la contraseña actual equivocada **no** suma intentos de bloqueo (manual cap. 3, §8.1).

### HU-ADM-004 — Mantener y renovar la sesión

**Como** usuario autenticado **quiero** que mi sesión se renueve sola mientras trabajo **para** no tener que escribir mi contraseña cada pocos minutos.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | Ninguna propia (guard de sesión e interceptor); termina en `/auth/login` |
| **Endpoint(s)** | `POST /api/v1/auth/refresh`; `GET /api/v1/auth/me` |
| **Permiso** | `refresh`: anónimo con cabecera `X-SOFIA-CSRF`; `me`: solo autenticado |
| **Reglas** | [RN-AUT-013, RN-AUT-014, RN-AUT-015, RN-AUT-016, RN-AUT-017, RN-AUT-018, RN-AUT-019, RN-AUT-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-015, RN-GEN-016, RN-GEN-017, RN-GEN-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: El token de acceso vence y la sesión se renueva sin que me entere
  Dado que mi token de acceso venció y mi cookie "refresh_token" sigue vigente
  Cuando navego a otra pantalla
  Entonces el sistema pide un token nuevo a "/auth/refresh" en segundo plano
  Y sigo trabajando sin ver ningún aviso

Escenario: Una petición recibe 401 y se repite una sola vez
  Dado que una petición a la API responde 401
  Cuando el sistema logra renovar la sesión
  Entonces repite la petición original con el token nuevo sin avisarme
  Y las renovaciones simultáneas de una misma pestaña se unifican en una sola petición

Escenario: No se puede renovar la sesión
  Dado que pasaron más de 7 días sin usar SOFIA, o cerré sesión en otro equipo, o un administrador desactivó mi cuenta
  Cuando el sistema intenta renovar
  Entonces me lleva a "Iniciar sesión" sin mostrar ningún aviso
  Y borra mis datos locales de sesión

Escenario: Abrir una pantalla sin sesión
  Dado que no tengo sesión
  Cuando abro "/dashboard"
  Entonces el sistema me lleva a "/auth/login" sin ningún aviso

Escenario: Renovar exige la cookie y la cabecera de seguridad
  Dado que llamo a "POST /api/v1/auth/refresh" sin la cabecera "X-SOFIA-CSRF"
  Entonces la API responde 403 con código "Auth.Csrf"
  Pero si falta la cookie "refresh_token" responde 401 sin cuerpo de error propio

Escenario: Cada renovación rota el token de refresco
  Dado que renuevo mi sesión
  Entonces el token de refresco usado queda revocado y recibo un par nuevo (acceso y refresco de 7 días)
  Y el token de acceso nuevo refleja mis roles, sucursal base e indicador de cambio de clave actuales

Escenario: Dos pestañas renuevan a la vez con el mismo token
  Dado que dos renovaciones usan el mismo token de refresco al mismo tiempo
  Entonces solo una obtiene el par nuevo
  Y la otra recibe 401 y no se crea una segunda sesión

Escenario: Detección de copia de la cookie
  Dado que un token de refresco ya fue revocado hace más de 30 segundos
  Cuando alguien lo presenta otra vez
  Entonces la API responde 401 "Auth.InvalidRefreshToken"
  Y se revocan todos los tokens de refresco de la cuenta y cambia su sello de seguridad
  Y mis tokens de acceso vivos dejan de servir

Escenario: Al renovar se vuelven a comprobar cuenta y empresa
  Dado que mi cuenta fue eliminada o desactivada, o mi sucursal base es de otra empresa, o la empresa dejó de estar vigente
  Cuando intento renovar
  Entonces la API responde 401 "Auth.InvalidRefreshToken"
  Pero si el motivo es que la empresa no está vigente, el token de refresco no se revoca, para que al reactivar la empresa no parezca un robo

Escenario: Un cambio de rol invalida el token actual
  Dado que un administrador me asigna o quita un rol
  Cuando uso mi token de acceso anterior
  Entonces la API responde 401 (el sello de seguridad cambió)
  Y tras renovar la sesión recibo el token con los roles nuevos

Escenario: Un token no válido recibe 401 sin código
  Dado que el token falta, está vencido, mal firmado o su empresa no está vigente
  Cuando llamo a un endpoint protegido
  Entonces la API responde 401 sin campo "code"
  Y todo endpoint sin metadatos de autorización exige un usuario autenticado

Escenario: Consultar mi perfil
  Dado que tengo sesión
  Cuando el sistema pide "GET /api/v1/auth/me"
  Entonces recibe mis roles y mis permisos distintos (módulo y acción)
  Y sigue disponible aunque deba cambiar mi contraseña
  Pero si la cuenta no se encuentra responde 404 "Cuenta.NotFound"
```

> **Brecha conocida.** El SPA nunca avisa «sesión expirada»: todo fallo de renovación termina en «Iniciar sesión» sin texto (las claves `errors.unauthorized` y similares existen en `es.json` y no se usan; inventario 19a §5.4). No hay renovación proactiva: el token se renueva al navegar con el token vencido o al recibir el primer 401. La duración del token de acceso la fija la clave `Jwt:ExpiryMinutes` ([RN-GEN-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)); el valor concreto no se verificó (el manual cap. 1, §9.1 dice «unos 15 minutos» sin comprobarlo).

### HU-ADM-005 — Cerrar sesión

**Como** usuario autenticado **quiero** cerrar mi sesión **para** que nadie use mi cuenta desde este ni desde otros equipos.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | Cabecera › imagen de perfil («Menú de usuario») › «Cerrar sesión» |
| **Endpoint(s)** | `POST /api/v1/auth/logout` |
| **Permiso** | Anónimo con cabecera `X-SOFIA-CSRF` (no exige token de acceso válido) |
| **Reglas** | [RN-AUT-013, RN-AUT-020, RN-AUT-021, RN-AUT-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Cerrar sesión desde el menú de usuario
  Dado que tengo la sesión iniciada
  Cuando hago clic en la imagen de perfil y luego en "Cerrar sesión"
  Entonces el sistema me lleva a "/auth/login"
  Y no pide confirmación ni muestra ningún mensaje

Escenario: Se cierran todas las sesiones de la cuenta
  Dado que mi cuenta tiene sesiones abiertas en otro navegador
  Cuando cierro sesión
  Entonces se revocan todos los tokens de refresco activos de mi cuenta y cambia su sello de seguridad
  Y mis tokens de acceso vivos dejan de servir en todos los dispositivos

Escenario: Se borran los datos locales
  Dado que cierro sesión
  Entonces se eliminan del navegador el token, los filtros de búsqueda guardados, los borradores de venta del punto de venta y los permisos en memoria

Escenario: El cierre funciona aunque la llamada falle
  Dado que la API no responde al cierre de sesión
  Cuando hago clic en "Cerrar sesión"
  Entonces el SPA borra la sesión local y me lleva igualmente a "/auth/login"
  Y el error de la llamada se ignora

Escenario: El cierre exige la cabecera de seguridad
  Dado que llamo a "POST /api/v1/auth/logout" sin la cabecera "X-SOFIA-CSRF"
  Entonces la API responde 403 "Auth.Csrf" (el SPA siempre la envía)

Escenario: El cierre es anónimo e idempotente
  Dado que mi token de acceso ya venció
  Cuando se llama a "POST /api/v1/auth/logout" con la cookie de refresco vigente
  Entonces la API identifica mi cuenta por la cookie y la cierra
  Pero si no logra identificar ninguna cuenta responde 204 igualmente
  Y la cookie "refresh_token" se borra siempre

Escenario: Renovación simultánea durante el cierre
  Dado que una renovación cambia un token mientras se cierra la sesión
  Entonces el cierre recarga los datos y reintenta hasta 3 veces, incluyendo el token recién emitido
  Pero si el tercer intento también falla responde 409 "Concurrency.Conflict"
  Y el SPA ignora ese error
```

### HU-ADM-006 — Cambiar mi contraseña

**Como** usuario autenticado **quiero** cambiar mi propia contraseña **para** renovarla o protegerla si creo que alguien más la conoce.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | `/cambiar-clave` — «Cambiar contraseña» (acceso: menú de usuario › «Cambiar contraseña») |
| **Endpoint(s)** | `POST /api/v1/auth/change-password` |
| **Permiso** | Solo autenticado (política `AuthPolicy`); no exige ningún permiso de módulo |
| **Reglas** | [RN-AUT-024, RN-AUT-025, RN-AUT-026, RN-AUT-027, RN-AUT-028, RN-AUT-029, RN-AUT-030, RN-AUT-031](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013, RN-GEN-026](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Cambiar la contraseña
  Dado que tengo la sesión iniciada y entré por "Menú de usuario › Cambiar contraseña"
  Cuando escribo mi Contraseña actual, una Nueva contraseña de 8 caracteres o más distinta de la actual y la misma en Confirmar nueva contraseña
  Y hago clic en "Guardar"
  Entonces veo el aviso "Contraseña actualizada. Inicia sesión con tu nueva contraseña."
  Y el sistema cierra todas mis sesiones, incluida la actual, y me lleva a "Iniciar sesión"
  Y la contraseña anterior ya no sirve
  Y se registra el evento de auditoría "CLAVE_CAMBIAR"

Escenario: La pantalla advierte del cierre de sesiones
  Dado que abro "/cambiar-clave"
  Entonces leo "Al guardar se cerrarán todas tus sesiones y tendrás que iniciar sesión con tu nueva contraseña"
  Y veo los botones "Cancelar" y "Guardar" y la flecha de volver

Escenario: Volver sin cambiar nada
  Dado que el cambio no es obligatorio
  Cuando hago clic en "Cancelar", en la flecha de volver o en la miga "Inicio"
  Entonces vuelvo al Inicio y mi contraseña no cambia

Esquema del escenario: Validaciones del formulario
  Dado que estoy en "/cambiar-clave"
  Cuando dejo <campo> con <valor> y hago clic en "Guardar"
  Entonces veo <mensaje> debajo del campo
  Y no se envía nada al servidor

  Ejemplos:
    | campo                      | valor                              | mensaje                                         |
    | Contraseña actual          | vacío                              | Este campo es obligatorio                       |
    | Nueva contraseña           | vacío                              | Este campo es obligatorio                       |
    | Nueva contraseña           | 7 caracteres                       | Debe tener al menos 8 caracteres                |
    | Nueva contraseña           | igual a la contraseña actual       | La nueva contraseña debe ser distinta de la actual |
    | Confirmar nueva contraseña | vacío                              | Este campo es obligatorio                       |
    | Confirmar nueva contraseña | distinta de la nueva               | Las contraseñas no coinciden                    |

Escenario: Contraseña actual incorrecta
  Dado que escribo una Contraseña actual que no es la mía
  Cuando hago clic en "Guardar"
  Entonces veo el aviso "La contraseña actual es incorrecta"
  Y me quedo en la pantalla, con mi sesión intacta (la API responde 400 y no 401 para no confundirlo con una sesión vencida)
  Y el intento no suma al contador de bloqueo

Escenario: Solo se cambia la contraseña de la cuenta del token
  Dado que llamo a "POST /api/v1/auth/change-password"
  Entonces la cuenta que cambia su clave es siempre la del token, no puedo indicar otra
  Pero si el token no trae un id válido la API responde 401

Escenario: Efectos del cambio sobre la cuenta
  Dado que cambio mi contraseña
  Entonces se guarda el nuevo hash BCrypt y cambia mi sello de seguridad
  Y se apaga "Requiere Cambio de Clave"
  Y se anula cualquier token de recuperación pendiente
  Y se revocan todos mis tokens de refresco y se borra la cookie

Escenario: Cuenta desactivada o eliminada durante el cambio
  Dado que mi cuenta se desactivó o eliminó mientras estaba en la pantalla
  Cuando hago clic en "Guardar"
  Entonces la API responde 401 "Auth.InvalidCredentials"
  Y el sistema intenta renovar mi sesión y, si no puede, me lleva a "Iniciar sesión"

Escenario: Error del servidor o exceso de intentos
  Dado que ya hice más de 10 peticiones por minuto, o el servidor falla
  Cuando hago clic en "Guardar"
  Entonces veo "No se pudo cambiar la contraseña. Inténtalo de nuevo."
  # No verificado en la aplicación: no se pudo provocar
```

> **Nota.** «Confirmar nueva contraseña» es una validación solo del SPA: la API recibe `currentPassword` y `newPassword`. La longitud mínima (8) es la misma que al crear una cuenta, registrar una empresa y restablecer la clave; no hay más requisitos de complejidad (RN-AUT-026). **No verificado:** qué caracteres como `<` o `>` altera el saneador de HTML sobre `CurrentPassword`, que no está en la lista de propiedades excluidas ([RN-GEN-010](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), sección 12.2 del catálogo).

### HU-ADM-007 — Cambiar la contraseña cuando es obligatorio

**Como** cajero con una contraseña inicial asignada por el administrador **quiero** que el sistema me obligue a elegir la mía en el primer ingreso **para** que solo yo conozca mi contraseña.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | `/cambiar-clave` en modo forzado |
| **Endpoint(s)** | `POST /api/v1/auth/change-password`; `GET /api/v1/auth/me` (exentos); cualquier otro endpoint autenticado responde 403 |
| **Permiso** | Solo autenticado |
| **Reglas** | [RN-GEN-019](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-AUT-010, RN-AUT-023, RN-AUT-030](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-008, RN-CTA-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Primer ingreso de una cuenta creada por un administrador
  Dado que la cuenta "carlos.cajero" fue creada por un administrador y tiene "Requiere Cambio de Clave: Sí"
  Cuando inicio sesión con la contraseña inicial
  Entonces no veo el Inicio: veo "Cambiar contraseña"
  Y leo el aviso "Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar."
  Y no existe el botón "Cancelar" ni la flecha de volver

Escenario: El menú no deja salir de la pantalla
  Dado que debo cambiar mi contraseña
  Cuando hago clic en cualquier opción del menú lateral o escribo a mano otra dirección, por ejemplo "/pos"
  Entonces el sistema me devuelve a "/cambiar-clave" sin mostrar ningún aviso

Escenario: Completar el cambio obligatorio
  Dado que estoy en la pantalla en modo forzado
  Cuando escribo la contraseña inicial como Contraseña actual, una Nueva contraseña válida y su confirmación y hago clic en "Guardar"
  Entonces veo "Contraseña actualizada. Inicia sesión con tu nueva contraseña."
  Y me lleva a "Iniciar sesión"
  Y al entrar con la contraseña nueva no se me vuelve a pedir el cambio
  Y en la ficha de mi cuenta "Requiere Cambio de Clave" pasa a "No"

Escenario: El administrador fuerza el cambio con mi sesión ya abierta
  Dado que tengo la sesión abierta y un administrador activa "Forzar Cambio de Contraseña" en mi cuenta
  Cuando hago mi siguiente acción
  Entonces la API responde 403 con código "Auth.CambioClaveRequerido" a cualquier endpoint, salvo "GET /api/v1/auth/me" y "POST /api/v1/auth/change-password"
  Y el sistema me lleva a "/cambiar-clave"
  # El indicador se lee de la base de datos en cada petición: también aplica a tokens vivos

Escenario: Contraseña actual equivocada en el modo forzado
  Dado que estoy en modo forzado
  Cuando escribo una Contraseña actual incorrecta
  Entonces veo el aviso rojo "La contraseña actual es incorrecta"

Escenario: El administrador que registra la empresa no pasa por este paso
  Dado que registré mi empresa en "/auth/register"
  Entonces mi cuenta nace sin obligación de cambiar la contraseña ([HU-ADM-001](#hu-adm-001--registrar-la-empresa-y-su-administrador))
```

> **No verificado en la aplicación:** el modo forzado se documentó desde el inventario 19a §2 y se vio en vivo en el capítulo 3 del manual (§8.1). La API no exime ningún otro endpoint autenticado.

### HU-ADM-008 — Pedir el enlace para recuperar la contraseña

**Como** visitante que olvidó su contraseña **quiero** pedir un enlace a mi correo **para** crear una nueva sin ayuda del administrador.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | `/auth/forgot-password` — «Recupera tu contraseña» |
| **Endpoint(s)** | `POST /api/v1/auth/forgot-password` |
| **Permiso** | Anónimo (política `AuthPolicy`) |
| **Reglas** | [RN-AUT-032, RN-AUT-033, RN-AUT-034, RN-AUT-035, RN-AUT-036](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-026](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EPL-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Pedir el enlace
  Dado que estoy en "/auth/forgot-password"
  Cuando escribo Usuario "ana.manual" y hago clic en "Enviar enlace"
  Entonces el botón muestra un círculo de espera mientras el sistema trabaja
  Y el formulario se reemplaza por una alerta verde que dice "Si el usuario existe y tiene un correo registrado, recibirás un enlace para restablecer tu contraseña. Revisa tu bandeja de entrada: el enlace es válido por 1 hora."
  Y solo queda el enlace "Volver a iniciar sesión"

Escenario: La respuesta es siempre la misma
  Dado que escribo un usuario que no existe, o sin correo registrado, o con la cuenta desactivada
  Cuando hago clic en "Enviar enlace"
  Entonces veo exactamente el mismo mensaje de la alerta verde
  Y no se envía ningún correo (no se puede averiguar qué usuarios existen)

Escenario: Cuándo se genera de verdad el enlace
  Dado que la cuenta existe, no está eliminada y está activa
  Y su empleado no está eliminado y tiene correo
  Y la empresa tiene la suscripción vigente
  Cuando se pide el enlace
  Entonces se genera un token aleatorio de 32 bytes que vence 1 hora después y reemplaza a cualquier token anterior de la cuenta
  Y en la base de datos solo se guarda su hash SHA-256

Escenario: Solo sirve el último enlace
  Dado que pido el enlace dos veces
  Cuando uso el enlace del primer correo
  Entonces el enlace ya no es válido ([HU-ADM-009](#hu-adm-009--restablecer-la-contraseña-con-el-enlace))

Escenario: El correo que recibo
  Dado que se generó el enlace
  Entonces recibo en el correo del empleado un mensaje con asunto "Restablece tu contraseña de SOFIA"
  Y el mensaje dice que el enlace es válido por 1 hora y solo puede usarse una vez
  Y el enlace tiene la forma ".../auth/reset-password#token=…&usuario=…", con el token en el fragmento, que el navegador no envía a los servidores

Escenario: Falla el envío del correo
  Dado que el proveedor de correo falla
  Cuando pido el enlace
  Entonces sigo viendo el mensaje uniforme
  Y el token sigue siendo válido durante su hora (solo se registra una advertencia)

Escenario: Usuario vacío o demasiado largo
  Dado que dejo vacío el campo Usuario
  Cuando hago clic en "Enviar enlace"
  Entonces veo "El usuario es obligatorio" y no se envía nada
  Y el campo no deja escribir más de 50 caracteres

Escenario: Demasiadas solicitudes
  Dado que ya hice 10 peticiones de credenciales en el último minuto desde mi IP
  Cuando hago clic en "Enviar enlace"
  Entonces veo "Demasiados intentos. Espera un minuto e inténtalo de nuevo."
  # No verificado en la aplicación

Escenario: Error del servidor
  Dado que el servidor falla
  Cuando hago clic en "Enviar enlace"
  Entonces veo "No se pudo procesar la solicitud. Inténtalo de nuevo."
  # No verificado en la aplicación

Escenario: Un cajero recupera su contraseña con el correo de su ficha de empleado
  Dado que el administrador registró el correo "carlos.cajero@sofia-manual.test" en la ficha de Carlos
  Cuando Carlos pide el enlace con su usuario
  Entonces el enlace se envía a ese correo
```

> **Brecha conocida.** El controlador descarta el resultado del comando, así que un usuario vacío o de más de 50 caracteres llamado directamente a la API también responde 200 con el mensaje uniforme (RN-AUT-032; hallazgo 5 de la sección 12.3 del catálogo). El SPA lo impide antes. Si un empleado no tiene correo, no puede recuperar su contraseña por sí mismo y el administrador solo puede usar «Forzar Cambio de Contraseña» ([HU-ADM-036](#hu-adm-036--administrar-el-estado-de-una-cuenta)). En los entornos de Desarrollo y Pruebas sin proveedor de correo configurado el mensaje se descarta (RN-AUT-036).

### HU-ADM-009 — Restablecer la contraseña con el enlace

**Como** visitante con el enlace de recuperación **quiero** fijar una contraseña nueva **para** volver a entrar a mi cuenta.

| | |
|---|---|
| **Módulo** | Autenticación |
| **Pantalla** | `/auth/reset-password` — «Crea una nueva contraseña» (se llega solo por el enlace del correo) |
| **Endpoint(s)** | `POST /api/v1/auth/reset-password` |
| **Permiso** | Anónimo (política `AuthPolicy`) |
| **Reglas** | [RN-AUT-037, RN-AUT-038, RN-AUT-039](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013, RN-GEN-014, RN-GEN-026](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Restablecer la contraseña
  Dado que abrí el enlace del correo y veo "Crea una nueva contraseña"
  Y la dirección del navegador se limpió sola y no muestra mi usuario
  Cuando escribo una Nueva contraseña de 8 caracteres o más y la misma en Confirmar nueva contraseña y hago clic en "Guardar contraseña"
  Entonces veo el aviso "Contraseña restablecida. Inicia sesión con tu nueva contraseña."
  Y el sistema me lleva a "Iniciar sesión"
  Y entro con mi usuario y la contraseña nueva

Escenario: Efectos del restablecimiento
  Dado que se restableció la contraseña
  Entonces se guarda el nuevo hash y cambia el sello de seguridad
  Y se apaga "Requiere Cambio de Clave"
  Y el token queda consumido: solo sirve una vez
  Y los intentos fallidos vuelven a 0 y se quita el bloqueo
  Y se revocan todos los tokens de refresco: se cierran todas las sesiones de la cuenta
  Y se registra el evento "CLAVE_RESTABLECER" con el dueño de la cuenta como actor

Escenario: Enlace incompleto
  Dado que abro "/auth/reset-password" sin token o sin usuario en el fragmento
  Entonces veo la alerta roja "Este enlace de recuperación está incompleto o ya fue usado. Solicita uno nuevo."
  Y no aparece el formulario
  Y hago clic en "Solicitar un nuevo enlace" y voy a "/auth/forgot-password"

Esquema del escenario: Validaciones del formulario
  Dado que estoy en el formulario
  Cuando dejo <campo> con <valor> y hago clic en "Guardar contraseña"
  Entonces veo <mensaje> y no se envía nada

  Ejemplos:
    | campo                      | valor                  | mensaje                          |
    | Nueva contraseña           | vacío                  | La contraseña es obligatoria     |
    | Nueva contraseña           | 7 caracteres           | Debe tener al menos 8 caracteres |
    | Confirmar nueva contraseña | vacío                  | La contraseña es obligatoria     |
    | Confirmar nueva contraseña | distinta de la nueva   | Las contraseñas no coinciden     |

Escenario: El enlace ya se usó, venció o fue reemplazado
  Dado que el token ya se usó, o pasó más de 1 hora, o pedí un enlace más nuevo
  Cuando hago clic en "Guardar contraseña"
  Entonces veo la alerta roja "El enlace no es válido o ya expiró. Solicita uno nuevo."
  Y un usuario inexistente recibe la misma respuesta que un token incorrecto

Escenario: Demasiados envíos
  Dado que superé el límite de 10 peticiones por minuto
  Cuando hago clic en "Guardar contraseña"
  Entonces veo "Demasiados intentos. Espera un minuto e inténtalo de nuevo."
  # No verificado en la aplicación

Escenario: Error del servidor
  Dado que el servidor falla
  Cuando hago clic en "Guardar contraseña"
  Entonces veo "No se pudo restablecer la contraseña. Inténtalo de nuevo."
  # No verificado en la aplicación
```

> **Brecha conocida.** El handler no vuelve a comprobar que la cuenta esté activa ni que la empresa esté vigente: eso se verificó al emitir el token ([RN-AUT-039](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)). Si el evento de auditoría fuera inválido, se omite en silencio ([RN-GEN-014](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)).

---

## 4. Menú, permisos de pantalla y cabecera

### HU-ADM-010 — Ver solo las opciones de menú que mi rol permite

**Como** usuario autenticado **quiero** que el menú lateral me muestre solo las pantallas que mi rol puede leer **para** no perderme entre opciones que no puedo usar.

| | |
|---|---|
| **Módulo** | Layout (menú lateral) y perfil de permisos |
| **Pantalla** | Menú lateral de toda la aplicación (grupos INICIO, ADMINISTRACIÓN, CATÁLOGOS, INVENTARIO, PUNTO DE VENTA, PACIENTES & ATENCIÓN, DIGEMID y SUNAT) |
| **Endpoint(s)** | `GET /api/v1/auth/me` |
| **Permiso** | Solo autenticado; la visibilidad de cada opción exige `<Módulo>:Leer` de su ruta |
| **Reglas** | [RN-GEN-020, RN-GEN-021, RN-GEN-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-AUT-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-ROL-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: El administrador ve todo el menú
  Dado que mi cuenta tiene el rol "Admin"
  Cuando entro al sistema
  Entonces el menú lateral muestra 8 grupos y 24 opciones: Dashboard; Empresa, Sucursales, Empleados, Roles & Permisos y Cuentas de Usuario; los nueve catálogos; Lotes y Stock por Sucursal; Sesiones de Caja y Ventas; Pacientes, Profesionales de Salud y Recetas Médicas; Catálogo DIGEMID; y Series Fiscales

Escenario: El cajero ve solo lo que su rol puede leer
  Dado que el rol "Cajero" tiene "Leer" en Empleados, Medicamentos, Aseguradoras, Inventario, Ventas, Sesiones de Caja, Pacientes, Profesionales de Salud, Recetas Médicas y DIGEMID
  Y no tiene "Leer" en Empresa, Sucursales, Roles y Cuentas de Usuario ni Series Fiscales
  Cuando "carlos.cajero" entra al sistema
  Entonces ve Dashboard, Empleados, Medicamentos, Aseguradoras, Lotes, Stock por Sucursal, Sesiones de Caja, Ventas, Pacientes, Profesionales de Salud, Recetas Médicas y Catálogo DIGEMID
  Y no ve "Empresa", "Sucursales", "Roles & Permisos", "Cuentas de Usuario" ni "Series Fiscales"
  Y el grupo SUNAT desaparece porque se quedó sin opciones

Escenario: Dashboard siempre está
  Dado que mi rol no tiene ningún permiso
  Cuando entro al sistema
  Entonces el menú solo muestra "Dashboard"

Escenario: Mientras llegan los permisos solo se ve Dashboard
  Dado que el sistema todavía está cargando mi perfil de permisos
  Entonces el menú muestra únicamente "Dashboard"
  Y las demás opciones aparecen cuando llegan los permisos

Escenario: No se pueden cargar los permisos
  Dado que "GET /api/v1/auth/me" falla
  Entonces veo el aviso "No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión."
  Y el menú queda solo con "Dashboard"

Escenario: Quitar un permiso al rol cambia el menú
  Dado que el administrador desmarca "Profesionales de Salud › Leer" en el rol "Cajero" y guarda
  Cuando Carlos vuelve a iniciar sesión
  Entonces la opción "Profesionales de Salud" ya no aparece en su menú
  Pero una persona con la sesión abierta no ve el cambio al instante: la pantalla del rol avisa que los usuarios conectados verán el menú actualizado al recargar la página

Escenario: El perfil devuelve roles y permisos distintos
  Dado que mi cuenta tiene un rol con permisos
  Cuando el sistema consulta "GET /api/v1/auth/me"
  Entonces recibe mis roles y la lista de permisos (módulo y acción) sin repetidos

Escenario: Módulos sin pantalla no tienen opción de menú
  Dado que existen permisos de Transferencias, Despachos Delivery, Órdenes Magistrales, Servicios y Auditoría de Seguridad
  Entonces el menú no tiene opciones para esos módulos porque todavía no tienen pantalla

Escenario: Comportamiento del menú
  Dado que estoy en una pantalla del menú
  Entonces la opción actual aparece resaltada
  Y el botón de tres rayas colapsa el menú a una columna de 80 píxeles con solo iconos
  Y esa preferencia no se guarda: al recargar el menú vuelve a verse abierto
  Y no hay buscador de opciones en el menú ni en la cabecera
```

> **Nota.** Las reglas del menú (visibilidad por `Leer`, grupos que desaparecen) están en el inventario 19a §4.3. **No verificado en la aplicación:** la vista angosta (menos de 1024 px) y el Ctrl + clic para abrir una opción en otra pestaña (manual cap. 1, §12, puntos 8 y 9). La caché de permisos de un rol dura 5 minutos en el servidor y se invalida de inmediato al asignar o revocar un permiso, pero solo en la instancia que atendió la petición ([RN-GEN-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)).

### HU-ADM-011 — Entrar solo a las pantallas que mi rol permite

**Como** administrador de la empresa **quiero** que el sistema impida abrir pantallas sin permiso aunque alguien escriba la dirección a mano **para** que ocultar una opción del menú no sea la única protección.

| | |
|---|---|
| **Módulo** | Autorización |
| **Pantalla** | Cualquier ruta del layout (guard de permisos); destino de rechazo `/dashboard` |
| **Endpoint(s)** | Todos los protegidos con `[HasPermission]`; `GET /api/v1/auth/me` para cargar los permisos |
| **Permiso** | `Leer` del módulo de la ruta; `Crear` además si la ruta es `nueva`, `nuevo` o `carga-masiva` |
| **Reglas** | [RN-GEN-015, RN-GEN-020, RN-GEN-021, RN-GEN-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-ROL-001, RN-SUC-001, RN-EPL-001, RN-SER-001, RN-EMP-001, RN-CTA-001](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Abrir por dirección una pantalla sin permiso
  Dado que soy "carlos.cajero" y mi rol no tiene "Roles y Cuentas de Usuario › Leer"
  Cuando escribo "/roles" en el navegador
  Entonces el sistema me devuelve al Dashboard
  Y veo el aviso rojo "No tienes permisos para acceder a esta pantalla."

Escenario: Pantalla de creación sin la acción Crear
  Dado que mi rol tiene "Empleados › Leer" pero no "Empleados › Crear"
  Cuando escribo "/empleados/nueva"
  Entonces veo "No tienes permisos para acceder a esta pantalla." y vuelvo al Dashboard

Escenario: Carga masiva sin la acción Crear
  Dado que mi rol no tiene "Sucursales › Crear"
  Cuando escribo "/sucursales/carga-masiva"
  Entonces veo "No tienes permisos para acceder a esta pantalla." y vuelvo al Dashboard
  # No verificado en la aplicación: el administrador tiene todos los permisos

Escenario: Las subrutas de detalle solo exigen Leer
  Dado que mi rol tiene "Empleados › Leer" y no "Empleados › Actualizar"
  Cuando abro "/empleados/:id"
  Entonces veo la ficha en modo ver
  Y no veo los botones "Editar" ni "Eliminar" porque cada acción la controlan los permisos del botón

Escenario: Los botones dependen de la acción
  Dado que mi rol solo tiene "Sucursales › Leer"
  Entonces en "/sucursales" no veo "Nuevo" ni "Carga Masiva"
  Y en el detalle no veo "Editar"
  Y el administrador sí los ve todos

Escenario: El administrador pasa todas las comprobaciones
  Dado que mi token tiene el rol "Admin" (sin distinguir mayúsculas)
  Entonces ninguna comprobación de permiso me bloquea, aunque en la matriz de permisos del rol "Admin" ninguna casilla esté marcada

Escenario: La API rechaza por permiso aunque el SPA no lo impida
  Dado que mi rol no tiene el permiso del endpoint
  Cuando llamo directamente a la API
  Entonces responde 403 sin campo "code"
  Y la pantalla que hizo la petición muestra su mensaje genérico (por ejemplo "Error al guardar el rol")

Escenario: Comparación de permisos
  Dado que el rol tiene el permiso "seguridad " con espacios y en minúsculas
  Entonces la comprobación lo reconoce igual que "Seguridad" (módulo y acción se comparan sin distinguir mayúsculas ni espacios sobrantes)

Escenario: Sin roles o sin empresa en el token
  Dado que el token no trae roles o no trae el identificador de empresa
  Cuando llamo a un endpoint con permiso
  Entonces la API responde 403

Escenario: Falla la carga del perfil al entrar a una pantalla
  Dado que "GET /api/v1/auth/me" falla al abrir una pantalla protegida
  Entonces veo "No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión." y vuelvo al Dashboard
```

> **Brecha conocida.** Las pantallas `/cambiar-clave` y `/dashboard` no tienen módulo asociado y cualquier usuario con sesión las abre (inventario 19a §5.3). El botón «Exportar» de las búsquedas no lleva un permiso propio en el SPA; el servidor exige `Leer` del módulo, que ya es necesario para llegar a la búsqueda (manual cap. 2, §12). La API no valida que el módulo y la acción de un permiso existan en el catálogo al asignarlos ([RN-ROL-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)); ver [HU-ADM-030](#hu-adm-030--definir-los-permisos-de-un-rol).

### HU-ADM-012 — Cambiar el idioma de la interfaz

**Como** usuario autenticado **quiero** cambiar la interfaz entre español e inglés **para** trabajar en el idioma que prefiera.

| | |
|---|---|
| **Módulo** | Layout (cabecera) |
| **Pantalla** | Cabecera › bandera de idioma (texto alternativo «Idioma») |
| **Endpoint(s)** | Ninguno |
| **Permiso** | Solo autenticado (las pantallas públicas `/auth/*` no tienen selector) |
| **Reglas** | Sin regla `RN` propia (comportamiento de interfaz, inventario 19a §4.5) |
| **Estado** | Parcial (el idioma elegido no se guarda) |

**Criterios de aceptación**

```gherkin
Escenario: Cambiar a inglés
  Dado que estoy en cualquier pantalla con el idioma "Español"
  Cuando hago clic en la bandera y elijo "English"
  Entonces veo un cuadro con el título "Cambiar idioma", el mensaje "La interfaz se actualizará al idioma seleccionado. ¿Deseas continuar?" y los botones "Cancelar" y "Confirmar"
  Y al hacer clic en "Confirmar" el menú, la cabecera y la pantalla se traducen de inmediato (el saludo pasa a "Hello, Ana Demo Prueba")
  Y las fechas cambian de formato (en español, día/mes/año)

Escenario: Cancelar el cambio
  Dado que se abrió el cuadro "Cambiar idioma"
  Cuando hago clic en "Cancelar"
  Entonces el cuadro se cierra y el idioma no cambia

Escenario: Elegir el idioma ya activo
  Dado que el idioma activo es "Español"
  Cuando elijo "Español"
  Entonces no pasa nada y no se abre ningún cuadro

Escenario: El idioma no se conserva
  Dado que cambié a "English"
  Cuando recargo la página, cierro sesión o vuelvo a entrar
  Entonces la interfaz vuelve a español
  # No verificado en la aplicación: viene del inventario 19a §4.5
```

> **Brecha conocida.** El idioma se guarda solo en memoria (ni `localStorage` ni cookie). Las pantallas de acceso (iniciar sesión, registro y recuperación) están siempre en español y no tienen selector. La campana de la cabecera es decorativa: solo muestra «Sin notificaciones».

---

## 5. Empresa y suscripción

### HU-ADM-013 — Buscar mi empresa y exportarla a Excel

**Como** administrador **quiero** buscar la empresa por nombre, estado y vencimiento, y exportar el resultado **para** revisar el estado de mi suscripción.

| | |
|---|---|
| **Módulo** | Empresas |
| **Pantalla** | `/empresas` — «Empresas» (migas «Inicio / Administración / Empresa») |
| **Endpoint(s)** | `GET /api/v1/Empresas`; `POST /api/v1/Empresas/exportar` |
| **Permiso** | `Empresas:Leer` |
| **Reglas** | [RN-EMP-001, RN-EMP-017, RN-EMP-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-010, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Los resultados no aparecen hasta buscar
  Dado que abro "/empresas"
  Entonces veo la tarjeta "Búsqueda de Empresas" con "Ingresa los criterios de búsqueda" y los filtros "Nombre de la Empresa", "Estado", "Vencimiento desde" y "Vencimiento hasta"
  Y no veo ninguna tabla hasta hacer clic en "Buscar"

Escenario: Buscar sin filtros
  Dado que mi empresa es "Botica Manual Demo S.A.C."
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (1)" con una fila con las columnas "Empresa", "RUC", "Estado" y "Vigente"
  Y la fila dice "Botica Manual Demo S.A.C.", "20538291746", "Trial Activo" y una marca verde en "Vigente"
  Y solo veo mi propia empresa: las demás se comportan como inexistentes

Escenario: Filtrar por nombre sin distinguir mayúsculas
  Dado que escribo "manual" en "Nombre de la Empresa"
  Cuando presiono Enter
  Entonces la tabla trae "Botica Manual Demo S.A.C."

Escenario: Filtrar por estado
  Dado que elijo "Trial Activo" en "Estado"
  Cuando hago clic en "Buscar"
  Entonces aparece mi empresa
  Pero con "Activo" veo "Resultados (0)" y el paginador dice "Sin resultados"

Escenario: Filtrar por vencimiento
  Dado que mi empresa vence el 13/11/2026
  Cuando elijo con el calendario "Vencimiento desde" 20/10/2026
  Entonces mi empresa aparece
  Pero con "Vencimiento desde" y "Vencimiento hasta" ambos en 20/10/2026 no aparece
  Y los dos campos de fecha solo se llenan con el calendario (no se pueden escribir)

Escenario: Limpiar
  Dado que hay filtros escritos y una tabla visible
  Cuando hago clic en "Limpiar"
  Entonces los cuatro filtros quedan vacíos y la tabla se oculta

Escenario: Paginar
  Dado que hay resultados
  Entonces el paginador ofrece "Items por página" de 10, 25 o 50 y botones de primera, anterior, siguiente y última página

Escenario: Exportar a Excel
  Dado que ya hice clic en "Buscar"
  Cuando hago clic en "Exportar"
  Entonces se descarga "empresas.xlsx" con una hoja "Data" y las columnas "Empresa", "RUC", "Estado", "Vigente" (con "Sí" o "No"), "Inicio Trial", "Vencimiento" (con formato dd/MM/yyyy) y "Sucursales" (cantidad)
  Y el archivo trae los filtros escritos en ese momento, no necesariamente los de la última búsqueda
  Y el botón "Exportar" no aparece antes de buscar

Escenario: Exportación demasiado grande
  Dado que la consulta filtrada tiene más de 10 000 filas
  Cuando hago clic en "Exportar"
  Entonces la API responde 400 "Export.TooManyRows" y no genera el archivo
  Y veo el aviso "Error al buscar empresas"

Escenario: Error al buscar
  Dado que la búsqueda o la exportación fallan
  Entonces veo "Error al buscar empresas"
```

> **Brecha conocida.** En esta pantalla la exportación reutiliza la clave de «Error al buscar empresas» (inventario 19b §1.1). Según el documento de carga masiva y exportación (§4.4), la columna «Estado» del archivo trae el nombre interno sin traducir (por ejemplo `TrialActivo`); **No verificado en la aplicación:** el contenido del archivo descargado (manual cap. 2, §3.1). La fila vacía de la tabla dice «Resultados» en lugar de «Sin resultados». No existen «Nuevo» ni «Carga Masiva» para empresas: el alta es el autorregistro ([HU-ADM-001](#hu-adm-001--registrar-la-empresa-y-su-administrador)).

### HU-ADM-014 — Ver los datos y la suscripción de mi empresa

**Como** administrador **quiero** abrir el detalle de mi empresa **para** ver su estado de suscripción, las fechas de la prueba y cuántas sucursales tiene.

| | |
|---|---|
| **Módulo** | Empresas |
| **Pantalla** | `/empresas/:id` — «Empresas» (tarjeta «Datos del registro») |
| **Endpoint(s)** | `GET /api/v1/Empresas/{id}` |
| **Permiso** | `Empresas:Leer` |
| **Reglas** | [RN-EMP-012, RN-EMP-014, RN-EMP-016, RN-EMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (no muestra días restantes ni el indicador «Vigente»; la política de receta no tiene pantalla) |

**Criterios de aceptación**

```gherkin
Escenario: Ver el detalle
  Dado que busqué mi empresa
  Cuando hago clic en "Botica Manual Demo S.A.C."
  Entonces se abre "/empresas/…" con la flecha de volver, el título "Empresas" y la tarjeta "Datos del registro"
  Y veo deshabilitados "Estado" ("Trial Activo"), "Sucursales" (cantidad de sucursales no eliminadas), "Inicio Trial" (14/10/2026), "Vencimiento" (13/11/2026), "Nombre de la Empresa" y "RUC (11 dígitos, opcional)"

Escenario: Una empresa nueva nace con 30 días de prueba
  Dado que registré mi empresa el 14/10/2026
  Entonces su estado es "Trial Activo" y su vencimiento es 30 días después (13/11/2026)

Escenario: Estado, fechas y sucursales no se editan
  Dado que estoy en el detalle
  Entonces "Estado", "Sucursales", "Inicio Trial" y "Vencimiento" nunca se habilitan
  Y no existe el botón "Eliminar" ni una pantalla para crear empresas

Escenario: Solo veo mi propia empresa
  Dado que conozco el identificador de la empresa de otra botica
  Cuando abro "/empresas/<ese id>"
  Entonces la empresa se comporta como inexistente (404 "Empresa.NotFound")
  Y veo "Error al cargar la empresa"

Escenario: Error al cargar
  Dado que falla la lectura
  Entonces veo el aviso "Error al cargar la empresa"
```

> **Brecha conocida.** El SPA muestra el nombre del estado y las fechas, pero no los días restantes ni el indicador «Vigente» (se calcula y no se pinta; solo aparece en la tabla de búsqueda). **Pendiente:** la política «exigir registro de receta» de la empresa está apagada por defecto y solo se cambia en la base de datos; no hay endpoint ni pantalla ([RN-EMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md); el efecto en las ventas lo describe la parte C). El componente `EmpresaMaintenanceComponent` existe pero no está enrutado.

### HU-ADM-015 — Editar el nombre y el RUC de mi empresa

**Como** administrador **quiero** corregir el nombre y el RUC de mi empresa **para** que los comprobantes salgan con los datos correctos.

| | |
|---|---|
| **Módulo** | Empresas |
| **Pantalla** | `/empresas/:id` (botón «Editar») |
| **Endpoint(s)** | `PUT /api/v1/Empresas/{id}` |
| **Permiso** | `Empresas:Actualizar` |
| **Reglas** | [RN-EMP-001, RN-EMP-002, RN-EMP-004, RN-EMP-005, RN-EMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (con brechas de validación en pantalla) |

**Criterios de aceptación**

```gherkin
Escenario: Editar y guardar
  Dado que tengo el permiso "Empresas › Actualizar" y estoy en el detalle
  Cuando hago clic en "Editar"
  Entonces se habilitan "Nombre de la Empresa" y "RUC (11 dígitos, opcional)" y aparecen "Cancelar" y "Guardar"
  Y cuando cambio el nombre y hago clic en "Guardar"
  Entonces veo "Empresa actualizada correctamente"
  Y la pantalla vuelve al modo ver, recargada, sin ir a la búsqueda
  Y se registra el evento de auditoría "EMPRESA_ACTUALIZAR" sin incluir el nombre ni el RUC

Escenario: Sin el permiso no hay botón
  Dado que mi rol no tiene "Empresas › Actualizar"
  Entonces no veo el botón "Editar"

Escenario: Cancelar descarta los cambios
  Dado que escribí otro nombre y otro RUC
  Cuando hago clic en "Cancelar"
  Entonces el nombre y el RUC vuelven a los que tenía la empresa y la pantalla vuelve al modo ver

Escenario: Nombre obligatorio
  Dado que borro el "Nombre de la Empresa"
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo del campo y no se envía nada

Escenario: Longitud del nombre
  Dado que escribo 262 caracteres en "Nombre de la Empresa"
  Entonces el campo se detiene en 150 caracteres

Escenario: RUC con formato inválido
  Dado que escribo un RUC de 10 dígitos, o con una letra entre 11 caracteres
  Cuando hago clic en "Guardar"
  Entonces la API rechaza el RUC (debe tener exactamente 11 dígitos)
  Y veo "Error al actualizar la empresa" sin más explicación

Escenario: RUC de otra empresa
  Dado que otra empresa ya registró el RUC que escribo
  Cuando hago clic en "Guardar"
  Entonces la API responde 409 "Empresa.RUC.Duplicado"
  Y veo "Error al actualizar la empresa"

Escenario: RUC vacío
  Dado que borro el RUC
  Cuando hago clic en "Guardar"
  Entonces el SPA envía un texto vacío, la API lo rechaza y veo "Error al actualizar la empresa"
  Y la empresa conserva su RUC

Escenario: Solo se editan nombre y RUC
  Dado que llamo a "PUT /api/v1/Empresas/{id}"
  Entonces solo se pueden cambiar nombre y RUC; el estado y las fechas no

Escenario: Otra empresa no se puede editar
  Dado que el identificador es de otra empresa
  Cuando llamo a "PUT /api/v1/Empresas/{id}"
  Entonces la API responde 404 "Empresa.NotFound"
```

> **Brecha conocida.** (1) El formulario limita el nombre a 150 caracteres y la API admite 200 (RN-EMP-002). (2) El formulario no valida el formato del RUC (solo 20 caracteres) pese al rótulo «(11 dígitos, opcional)»: el servidor lo rechaza y el usuario ve el genérico «Error al actualizar la empresa»; existen las claves `empresa.form.rucFormat` y `empresa.notifications.rucDuplicado`, que esta pantalla no usa. (3) Desde la pantalla no se puede dejar la empresa sin RUC; la API sí acepta un RUC nulo (RN-EMP-004), pero **no verificado en la aplicación** porque la pantalla no puede enviarlo. Sin RUC el punto de venta no emite comprobantes (parte C). Fuentes: inventario 19b §1.2 y manual cap. 2, §3.3.

### HU-ADM-016 — Restringir el acceso cuando la suscripción no está vigente

**Como** equipo de SOFIA **quiero** que una empresa suspendida, cancelada o con la prueba vencida no pueda operar **para** hacer cumplir la suscripción.

| | |
|---|---|
| **Módulo** | Empresas y autenticación |
| **Pantalla** | `/auth/login` (mensaje); sin pantalla de gestión del estado |
| **Endpoint(s)** | `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/forgot-password` y toda petición autenticada |
| **Permiso** | Anónimo / solo autenticado |
| **Reglas** | [RN-EMP-014, RN-EMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-AUT-009, RN-AUT-017, RN-AUT-034](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (la comprobación de vigencia funciona; ningún endpoint ni pantalla permite activar, suspender, cancelar ni extender la suscripción) |

**Criterios de aceptación**

```gherkin
Escenario: Cuándo una empresa está vigente
  Dado que una empresa no está eliminada
  Cuando su estado es "Activo", o es "Trial Activo" con vencimiento posterior a la hora actual
  Entonces está vigente

Escenario: Cuándo no está vigente
  Dado que el estado es "Suspendido" o "Cancelado", o la prueba venció, o la empresa no existe
  Entonces no está vigente

Escenario: En estado Activo la fecha de vencimiento no se evalúa
  Dado que la empresa está en estado "Activo" con un vencimiento pasado
  Entonces sigue vigente

Escenario: No se puede iniciar sesión
  Dado que la empresa no está vigente y la contraseña es correcta
  Cuando inicio sesión
  Entonces veo "La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla."

Escenario: Las sesiones vivas se cortan en un minuto como máximo
  Dado que tengo la sesión abierta y mi empresa deja de estar vigente
  Cuando pasa a lo sumo 1 minuto (la vigencia se guarda en caché por empresa)
  Entonces mis peticiones reciben 401 sin código
  Y el sistema me lleva a "Iniciar sesión"

Escenario: La renovación se rechaza sin revocar la sesión
  Dado que mi empresa no está vigente
  Cuando el sistema intenta renovar mi sesión
  Entonces la API responde 401 "Auth.InvalidRefreshToken"
  Pero el token de refresco no se revoca, para que al reactivar la empresa no parezca un robo

Escenario: No se envía el correo de recuperación
  Dado que mi empresa no está vigente
  Cuando pido el enlace de recuperación
  Entonces veo el mensaje uniforme y no se envía ningún correo
```

> **Pendiente.** Activar, suspender, cancelar y extender la prueba existen en la entidad `Empresa` pero ningún handler ni endpoint las invoca (solo las pruebas automáticas): hoy el estado solo puede cambiarse directamente en la base de datos ([RN-EMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)). **No verificado en la aplicación:** el mensaje de inicio de sesión con empresa no vigente (la empresa de prueba está vigente; manual cap. 1, §12).

---

## 6. Sucursales

### HU-ADM-017 — Buscar sucursales y exportarlas a Excel

**Como** administrador **quiero** buscar las sucursales por nombre, licencia y dirección, y exportar la lista **para** tener a la vista los locales de mi empresa.

| | |
|---|---|
| **Módulo** | Sucursales |
| **Pantalla** | `/sucursales` — «Sucursales» (tarjeta «Búsqueda de Sucursales») |
| **Endpoint(s)** | `GET /api/v1/Sucursales`; `POST /api/v1/Sucursales/exportar` |
| **Permiso** | `Sucursales:Leer` |
| **Reglas** | [RN-SUC-001, RN-SUC-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar sin filtros
  Dado que mi empresa tiene "Sede Principal" y "Sucursal Demo Norte"
  Cuando abro "/sucursales" y hago clic en "Buscar"
  Entonces veo "Resultados (2)" ordenadas por nombre con las columnas "Nombre", "Nº de Licencia", "Dirección Física" y "Gerente"
  Y el gerente se muestra como "—" cuando no hay
  Y la tabla no aparece antes de hacer clic en "Buscar"

Escenario: Filtros que se combinan
  Dado que escribo "demo-0002" en "Nº de Licencia" (en minúsculas)
  Cuando hago clic en "Buscar"
  Entonces veo solo "Sucursal Demo Norte"
  Y si además escribo "miraflores" en "Dirección Física" veo "Resultados (0)" y "Sin resultados"
  Y Enter dentro de cualquiera de los tres filtros también busca

Escenario: Limpiar
  Dado que hay filtros y tabla
  Cuando hago clic en "Limpiar"
  Entonces los tres filtros quedan vacíos y la tabla se oculta

Escenario: Paginación
  Dado que hay resultados
  Entonces el paginador empieza en 10 por página y ofrece 10, 25 y 50
  Y al volver desde un detalle se restauran filtros y página mientras no recargue el navegador

Escenario: Límites de paginación de la API
  Dado que llamo a "GET /api/v1/Sucursales" con "pageNumber" menor que 1, con "pageSize" fuera del rango de 1 a 10 000 o con un número de página enorme
  Entonces la página mínima se trata como 1, el tamaño se ajusta al rango de 1 a 10 000 y un número de página enorme devuelve una página vacía en lugar de fallar

Escenario: Exportar
  Dado que ya busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "sucursales.xlsx" con una hoja "Data" y las columnas "Nombre", "Dirección Física", "Nº de Licencia" y "Gerente" (vacío si no hay)
  Y se aplican los filtros escritos en ese momento
  # No verificado en la aplicación: el contenido del archivo (manual cap. 2, §6.3)

Escenario: Exportación de más de 10 000 filas
  Dado que la consulta tiene más de 10 000 filas
  Cuando hago clic en "Exportar"
  Entonces la API responde 400 "Export.TooManyRows"
  Y veo "Error al buscar sucursales"

Escenario: Solo veo las sucursales de mi empresa
  Dado que otra empresa tiene sucursales
  Cuando busco
  Entonces no aparecen en mis resultados
```

### HU-ADM-018 — Crear una sucursal

**Como** administrador **quiero** registrar una nueva sucursal con su licencia y dirección **para** poder asignarle empleados, series fiscales, stock y ventas.

| | |
|---|---|
| **Módulo** | Sucursales |
| **Pantalla** | `/sucursales/nueva` — «Sucursales» (botón «Nuevo») |
| **Endpoint(s)** | `POST /api/v1/Sucursales` |
| **Permiso** | `Sucursales:Crear` (la ruta exige además `Sucursales:Leer`) |
| **Reglas** | [RN-SUC-001, RN-SUC-002, RN-SUC-003, RN-SUC-004, RN-SUC-005, RN-SUC-006, RN-SUC-007](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-024, RN-GEN-036, RN-GEN-037, RN-GEN-038](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Crear la sucursal
  Dado que tengo el permiso "Sucursales › Crear" y estoy en "/sucursales/nueva"
  Cuando escribo Nombre "Sucursal Demo Norte", Nº de Licencia "LIC-DEMO-0002" y Dirección Física "Av. Ficticia Norte 456, Los Olivos, Lima"
  Y dejo "Gerente" en blanco
  Y hago clic en "Guardar"
  Entonces veo "Sucursal guardada correctamente"
  Y el sistema me lleva a "/sucursales" con los filtros vacíos y sin tabla
  Y la sucursal pertenece a la empresa de mi token

Escenario: Campos obligatorios
  Dado que el formulario está vacío
  Cuando hago clic en "Guardar"
  Entonces "Nombre", "Nº de Licencia" y "Dirección Física" muestran "Este campo es obligatorio"
  Y no se envía nada

Escenario: Longitudes máximas
  Dado que escribo más caracteres de los permitidos
  Entonces "Nombre" se detiene en 100, "Nº de Licencia" en 50 y "Dirección Física" en 255

Escenario: Licencia repetida en la empresa
  Dado que la "Sede Principal" ya tiene la licencia "LIC-DEMO-0001"
  Cuando intento crear otra sucursal con "LIC-DEMO-0001"
  Entonces veo el aviso rojo "Ya existe una sucursal con ese número de licencia."
  Y me quedo en la pantalla con lo que escribí
  # La licencia es única entre sucursales no eliminadas de la empresa, sin distinguir mayúsculas

Escenario: Dos altas simultáneas con la misma licencia
  Dado que otra persona guarda la misma licencia a la vez
  Cuando la base de datos rechaza el valor repetido
  Entonces veo "Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo."

Escenario: Valor fuera de rango o referencia inexistente
  Dado que la base de datos rechaza el guardado por una restricción CHECK o una clave foránea
  Entonces veo "Uno de los valores está fuera del rango permitido." o "El registro hace referencia a datos que ya no existen o todavía está en uso."

Escenario: Elegir un gerente
  Dado que tengo el permiso "Empleados › Leer"
  Cuando escribo al menos 2 letras del nombre en "Gerente" ("Ana")
  Entonces tras medio segundo la lista muestra "Ana Demo Prueba"
  Y solo puedo elegir uno de la lista: si dejo un texto que no corresponde a un empleado y salgo del campo, el campo se vacía

Escenario: El gerente debe ser un empleado de la empresa
  Dado que la API recibe un gerente que no existe en mi empresa
  Cuando guarda
  Entonces responde 404 "Empleado.NotFound"
  Y veo "Error al guardar la sucursal"

Escenario: El token no trae empresa
  Dado que el token no identifica a la empresa
  Cuando se guarda la sucursal
  Entonces la API responde 400 "Auth.Empresa" y veo "Error al guardar la sucursal"

Escenario: Cancelar
  Dado que escribí datos
  Cuando hago clic en "Cancelar"
  Entonces vuelvo a "/sucursales" sin guardar nada
```

### HU-ADM-019 — Ver y editar una sucursal

**Como** administrador **quiero** abrir una sucursal y modificar sus datos, incluida la licencia de la sede principal **para** mantener actualizados los datos que salen en las boletas.

| | |
|---|---|
| **Módulo** | Sucursales |
| **Pantalla** | `/sucursales/:id` — «Sucursales» (botón «Editar») |
| **Endpoint(s)** | `GET /api/v1/Sucursales/{id}`; `PUT /api/v1/Sucursales/{id}`; `GET /api/v1/Empleados` (para buscar gerente) |
| **Permiso** | `Sucursales:Leer`; `Sucursales:Actualizar` para editar |
| **Reglas** | [RN-SUC-002, RN-SUC-003, RN-SUC-004, RN-SUC-005, RN-SUC-006, RN-SUC-008](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EMP-010](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-036, RN-GEN-037, RN-GEN-038](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver el detalle
  Dado que hice clic en "Sede Principal" en la búsqueda
  Entonces veo el título "Sucursales", "Datos del registro" y los campos "Nombre", "Nº de Licencia", "Dirección Física" y "Gerente" deshabilitados
  Y el botón "Editar" solo si tengo "Sucursales › Actualizar"

Escenario: La sede principal creada por el registro nace sin licencia
  Dado que registré mi empresa
  Entonces la "Sede Principal" muestra el "Nº de Licencia" vacío (en la tabla, celda en blanco)
  Y la licencia es obligatoria al editar, así que la primera edición debe informarla

Escenario: Completar la licencia de la sede principal
  Dado que hice clic en "Editar" en la "Sede Principal"
  Cuando hago clic en "Guardar" con "Nº de Licencia" vacío
  Entonces veo "Este campo es obligatorio" debajo de "Nº de Licencia" y no se guarda nada
  Y cuando escribo "LIC-DEMO-0001" y hago clic en "Guardar"
  Entonces veo "Sucursal guardada correctamente"
  Y la pantalla vuelve al modo ver en el mismo detalle con la licencia "LIC-DEMO-0001"

Escenario: Editar muestra los tres botones
  Dado que estoy en modo editar de una sucursal existente
  Entonces veo "Cancelar", "Guardar" y, solo con "Sucursales › Eliminar", "Eliminar"

Escenario: Cancelar la edición
  Dado que cambié el nombre
  Cuando hago clic en "Cancelar"
  Entonces se restauran los datos leídos (incluido el gerente) y vuelvo al modo ver

Escenario: Licencia usada por otra sucursal
  Dado que otra sucursal tiene "LIC-DEMO-0002"
  Cuando edito esta sucursal con esa licencia y hago clic en "Guardar"
  Entonces veo "Ya existe una sucursal con ese número de licencia."

Escenario: Cambiar o quitar el gerente
  Dado que la sucursal tiene gerente
  Cuando vacío el campo "Gerente" y guardo
  Entonces la sucursal queda sin gerente
  Y si no toco el gerente, se reenvía el que ya tenía
  Y el gerente solo se valida contra la empresa cuando cambia

Escenario: Errores de base de datos al guardar
  Dado que la base de datos rechaza el guardado por un valor único repetido, una restricción CHECK o una clave foránea
  Entonces veo "Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.", "Uno de los valores está fuera del rango permitido." o "El registro hace referencia a datos que ya no existen o todavía está en uso." (el detalle de sucursales usa `saveErrorKey`)

Escenario: Errores genéricos
  Dado que ocurre otro error al guardar o al abrir la sucursal
  Entonces veo "Error al guardar la sucursal" o "Error al cargar la sucursal"
  Y esto incluye la sucursal inexistente o de otra empresa (404 "Sucursal.NotFound") y el gerente inexistente (404 "Empleado.NotFound")

Escenario: Buscar gerente sin permiso sobre empleados
  Dado que mi rol no tiene "Empleados › Leer"
  Cuando escribo en "Gerente"
  Entonces la consulta falla en silencio y el buscador no ofrece sugerencias
  # No verificado en la aplicación: no se probó en el módulo de sucursales (inventario 19b §2.2)
```

> **Nota.** El manual cap. 2 (§4.3) no asignó un gerente a la sede principal para no dejar una relación inesperada; **no verificado en la aplicación:** guardar un gerente elegido. El modelo EF declara la relación sucursal-gerente uno a uno, pero no hay índice único ni comprobación en los handlers, así que **no verificado** si un empleado puede gerenciar dos sucursales (sección 12.2 del catálogo de reglas).

### HU-ADM-020 — Eliminar una sucursal

**Como** administrador **quiero** eliminar una sucursal que ya no usamos **para** que desaparezca de las listas, sin romper registros que dependen de ella.

| | |
|---|---|
| **Módulo** | Sucursales |
| **Pantalla** | `/sucursales/:id` en modo editar (botón «Eliminar») |
| **Endpoint(s)** | `DELETE /api/v1/Sucursales/{id}` |
| **Permiso** | `Sucursales:Eliminar` |
| **Reglas** | [RN-SUC-008, RN-SUC-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una sucursal sin registros asociados
  Dado que "Sucursal Prueba Eliminar" no tiene empleados, cuentas, roles, stock, sesiones de caja, series fiscales, ventas, transferencias, cuarentena DIGEMID ni órdenes magistrales
  Cuando hago clic en "Editar", luego en "Eliminar"
  Entonces se abre el cuadro "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta sucursal?" y los botones "Cancelar" y "Confirmar"
  Y al hacer clic en "Confirmar" veo "Sucursal eliminada correctamente" y vuelvo a "/sucursales"
  Y al volver a buscar la sucursal ya no aparece (eliminación lógica)

Escenario: Cancelar la confirmación
  Dado que se abrió el cuadro de confirmación
  Cuando hago clic en "Cancelar"
  Entonces no se borra nada y sigo en modo editar

Escenario: La sucursal tiene registros asociados
  Dado que la "Sede Principal" tiene empleados y cuentas
  Cuando confirmo la eliminación
  Entonces la API responde 409 "Sucursal.InUse" y no borra nada
  Y veo el aviso rojo "No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados."
  Y me quedo en modo editar
  Y el aviso no dice cuál de los registros es el que bloquea

Escenario: Otros errores al eliminar
  Dado que la respuesta no es un 409 (por ejemplo, la sucursal ya no existe: 404 "Sucursal.NotFound")
  Cuando confirmo
  Entonces veo "Error al eliminar la sucursal"

Escenario: Sin permiso
  Dado que mi rol no tiene "Sucursales › Eliminar"
  Entonces el botón "Eliminar" no aparece en modo editar
```

> **Brecha conocida.** La pantalla decide el aviso por el estado HTTP 409, sin mirar el código: un 409 por otro motivo (por ejemplo `Concurrency.Conflict`) mostraría igualmente el texto de «registros asociados» (hallazgo 3 de la sección 12.3 del catálogo). El SPA no avisa antes de confirmar.

### HU-ADM-021 — Cargar sucursales desde un Excel

**Como** administrador de una cadena **quiero** crear muchas sucursales desde un archivo **para** no registrarlas una por una.

| | |
|---|---|
| **Módulo** | Sucursales (carga masiva) |
| **Pantalla** | `/sucursales/carga-masiva` — «Carga Masiva» (migas «Inicio / Administración / Sucursales / Carga Masiva») |
| **Endpoint(s)** | `GET /api/v1/Sucursales/plantilla`; `POST /api/v1/Sucursales/previsualizar`; `POST /api/v1/Sucursales/carga-masiva` |
| **Permiso** | `Sucursales:Crear` (la ruta exige además `Sucursales:Leer`) |
| **Reglas** | [RN-SUC-001, RN-SUC-012, RN-SUC-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-006, RN-IMP-007, RN-IMP-012, RN-IMP-014, RN-IMP-015, RN-IMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-024](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (las reglas del archivo en sí están en [HU-ADM-046](#hu-adm-046--cargar-un-excel-con-la-pantalla-de-carga-masiva)) |

**Criterios de aceptación**

```gherkin
Escenario: Descargar la plantilla
  Dado que estoy en "/sucursales/carga-masiva"
  Cuando hago clic en "Descargar plantilla"
  Entonces se descarga "plantilla-sucursales.xlsx" con una hoja "Plantilla" que solo contiene la fila de cabecera con las columnas "Nombre", "DireccionFisica" y "NumeroLicencia"
  Y no hay columna de gerente ni filas de ejemplo
  # No verificado en la aplicación: el contenido del archivo descargado

Escenario: Cargar un archivo con todas las filas válidas
  Dado que mi archivo tiene una fila "Sucursal Carga Uno", "Calle Ficticia 1, Lima", "LIC-CARGA-0001"
  Cuando lo elijo
  Entonces veo "Vista previa" con "1 filas totales" y "1 filas válidas"
  Y la columna "Estado" muestra una marca verde
  Y cuando hago clic en "Guardar válidos (1)" veo "Se guardaron 1 registros correctamente" y voy a "/sucursales"
  Y la sucursal queda en mi empresa y sin gerente

Esquema del escenario: Errores por fila en la vista previa
  Dado que mi archivo tiene una fila con <problema>
  Cuando lo elijo
  Entonces la columna de errores de esa fila muestra <mensaje>
  Y la fila queda con el icono de error en "Estado" y cuenta en "N filas con error"

  Ejemplos:
    | problema                                         | mensaje                                                                             |
    | Nombre vacío                                     | El campo 'nombre' es obligatorio                                                    |
    | Dirección vacía                                  | El campo 'direccionFisica' es obligatorio                                           |
    | Licencia vacía                                   | El campo 'numeroLicencia' es obligatorio                                            |
    | Nombre de más de 100 caracteres                  | 'nombre' no puede superar los 100 caracteres                                        |
    | Licencia "LIC-DEMO-0001" que ya existe           | El valor 'LIC-DEMO-0001' del campo 'numeroLicencia' ya existe en el sistema         |
    | Licencia repetida en dos filas del archivo       | El valor 'LIC-CARGA-0001' del campo 'numeroLicencia' está duplicado en el archivo   |

Escenario: Una licencia repetida marca todas las filas repetidas
  Dado que dos filas del archivo tienen "LIC-CARGA-0001"
  Cuando veo la vista previa
  Entonces las dos filas, incluida la primera, quedan con error y se ven "0 filas válidas"

Escenario: Solo se guardan las filas válidas
  Dado que la vista previa tiene 3 filas válidas y 2 con error
  Cuando hago clic en "Guardar válidos (3)"
  Entonces se guardan solo las 3 y el botón está deshabilitado si no hay ninguna válida

Escenario: Corregir y volver a subir
  Dado que hay filas con error
  Entonces no puedo corregirlas en la pantalla: corrijo el Excel y lo subo otra vez, y la vista previa nueva reemplaza a la anterior

Escenario: Al guardar se omiten licencias que ya existen
  Dado que entre la vista previa y el guardado alguien creó una sucursal con la misma licencia
  Cuando guardo
  Entonces esa fila se omite sin error y el aviso dice cuántas se guardaron realmente
  # No verificado en la aplicación: requiere dos usuarios a la vez (manual cap. 2, §8.5)

Escenario: El token no trae empresa al guardar la carga
  Dado que el token no identifica a la empresa
  Cuando guardo la carga masiva
  Entonces la API responde 400 "Auth.Empresa" y veo "Error al guardar los registros"

Escenario: Al guardar se vuelve a validar cada fila
  Dado que una fila no cumple las reglas del módulo al llegar a "POST /api/v1/Sucursales/carga-masiva"
  Entonces no se guarda ninguna fila y el código indica la primera fila y campo inválidos ("CargaMasivaSucursalesCommand.Rows[0].Nombre")
  Y veo "Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa."

Escenario: Sin permiso para crear
  Dado que mi rol no tiene "Sucursales › Crear"
  Entonces no veo el botón "Carga Masiva" y, si escribo la dirección, veo "No tienes permisos para acceder a esta pantalla."
```

> **Nota.** Los nombres de campo salen como los usa el sistema por dentro (`nombre`, `direccionFisica`, `numeroLicencia`), no con la etiqueta de la pantalla (manual cap. 2, §8.3). Si cambias el nombre de una columna (por ejemplo, con tilde), esa columna se lee vacía ([RN-IMP-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)). La carga masiva de sucursales no genera eventos de auditoría.

### HU-ADM-022 — Elegir entre las sucursales a las que tengo acceso

**Como** usuario autenticado **quiero** que los selectores de sucursal me ofrezcan solo las sucursales a las que puedo acceder **para** no operar en una sucursal ajena.

| | |
|---|---|
| **Módulo** | Sucursales |
| **Pantalla** | Sin pantalla propia; la usan los selectores de «Stock por Sucursal» (`/stock-por-sucursal`, parte B) |
| **Endpoint(s)** | `GET /api/v1/Sucursales/permitidas` |
| **Permiso** | Solo autenticado |
| **Reglas** | [RN-SUC-001, RN-SUC-010](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (consumida por Stock por Sucursal; sin pantalla de administración) |

**Criterios de aceptación**

```gherkin
Escenario: El administrador accede a todas las sucursales
  Dado que mi token tiene el rol "Admin"
  Cuando consulto las sucursales permitidas
  Entonces recibo todas las sucursales no eliminadas de mi empresa, ordenadas por nombre

Escenario: Otro usuario accede a su sucursal base, las asignadas a su cuenta y las de sus roles
  Dado que "carlos.cajero" tiene como sucursal base "Sede Principal"
  Y su cuenta tiene asignada "Sucursal Demo Norte"
  Cuando consulta las sucursales permitidas
  Entonces recibe "Sede Principal" y "Sucursal Demo Norte" ordenadas por nombre

Escenario: Una asignación obsoleta o ajena no da acceso
  Dado que una sucursal asignada fue eliminada o pertenece a otra empresa
  Cuando consulto las sucursales permitidas
  Entonces no aparece (el conjunto se cruza con las sucursales no eliminadas de mi empresa)

Escenario: Sin autenticación
  Dado que no hay usuario autenticado
  Entonces el conjunto de sucursales accesibles es vacío
```

---

## 7. Empleados

### HU-ADM-023 — Buscar empleados y exportarlos a Excel

**Como** administrador **quiero** buscar a mi personal por nombre, apellido, licencia y sucursal base, y exportar la lista **para** encontrar rápido la ficha de cada persona.

| | |
|---|---|
| **Módulo** | Empleados |
| **Pantalla** | `/empleados` — «Empleados» (tarjeta «Búsqueda de Empleados») |
| **Endpoint(s)** | `GET /api/v1/Empleados`; `POST /api/v1/Empleados/exportar` |
| **Permiso** | `Empleados:Leer` |
| **Reglas** | [RN-EPL-001, RN-EPL-016, RN-EPL-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar sin filtros
  Dado que hay tres empleados: Ana Demo Prueba, Carlos Cajero Demo y Nora Norte Demo
  Cuando abro "/empleados" y hago clic en "Buscar"
  Entonces veo "Resultados (3)" con las columnas "Nombre Completo", "Apellido Paterno", "Apellido Materno", "Licencia Profesional", "Correo electrónico" y "Sucursal Base"
  Y la lista sale ordenada por apellido paterno, luego apellido materno y luego nombres
  Y "Nombre Completo" junta nombres, apellido paterno y apellido materno, omitiendo las partes vacías
  Y la licencia y el correo se muestran como "—" cuando no hay

Escenario: Filtros combinados sin distinguir mayúsculas
  Dado que escribo "nor" en "Nombres"
  Cuando hago clic en "Buscar"
  Entonces veo solo "Nora Norte Demo"
  Y "qf-demo" en "Licencia Profesional" trae solo a Nora, y "demo" en "Apellido Materno" trae a Carlos y a Nora
  Y Enter dentro de cualquier campo también busca

Escenario: Filtrar por sucursal base
  Dado que escribo "zzz" en "Sucursal Base"
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (0)" y "Sin resultados"
  Y cuenta el texto escrito, lo haya elegido de la lista o no

Escenario: Empleados de una sucursal eliminada no aparecen
  Dado que la sucursal base de un empleado fue eliminada
  Cuando busco
  Entonces ese empleado no aparece en el listado

Escenario: Filtro de sucursal inválido en la API
  Dado que llamo a la API con el filtro de sucursal igual al GUID vacío
  Entonces responde 400 "GetEmpleadosWithPaginationQuery.SucursalId"

Escenario: Limpiar y paginar
  Dado que hay tabla
  Cuando hago clic en "Limpiar"
  Entonces los cinco filtros quedan vacíos y la tabla se oculta
  Y el paginador ofrece 10, 25 y 50 por página, empezando en 10

Escenario: Exportar
  Dado que ya busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "empleados.xlsx" con las columnas "Nombres", "Apellido Paterno", "Apellido Materno", "Licencia Profesional", "Correo electrónico" y "Sucursal Base" según los filtros escritos
  # No verificado en la aplicación: el contenido del archivo; el servidor respondió 200 (manual cap. 3, §3.3)

Escenario: Error al buscar o exportar
  Dado que falla la búsqueda, la exportación o se superan las 10 000 filas
  Entonces veo "Error al buscar empleados"

Escenario: Vista de solo consulta para un cajero
  Dado que el rol "Cajero" tiene "Empleados › Leer" pero no "Crear", "Actualizar" ni "Eliminar"
  Cuando "carlos.cajero" abre "/empleados"
  Entonces ve la lista, incluidos los correos y licencias de todos los empleados, pero no el botón "Nuevo"
```

### HU-ADM-024 — Registrar un empleado

**Como** administrador **quiero** registrar la ficha de una persona de mi equipo con su sucursal base **para** poder crearle luego una cuenta y que reciba su enlace de recuperación de contraseña.

| | |
|---|---|
| **Módulo** | Empleados |
| **Pantalla** | `/empleados/nueva` — «Empleados» (botón «Nuevo») |
| **Endpoint(s)** | `POST /api/v1/Empleados`; `GET /api/v1/Sucursales` (autocompletado de sucursal) |
| **Permiso** | `Empleados:Crear` (la ruta exige además `Empleados:Leer`; el autocompletado, `Sucursales:Leer`) |
| **Reglas** | [RN-EPL-001, RN-EPL-002, RN-EPL-003, RN-EPL-004, RN-EPL-005, RN-EPL-006, RN-EPL-007, RN-EPL-008, RN-EPL-009, RN-EPL-010, RN-EPL-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-036](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un empleado
  Dado que estoy en "/empleados/nueva"
  Cuando escribo Nombres "Carlos", Apellido Paterno "Cajero", Apellido Materno "Demo"
  Y elijo Sucursal Base "Sede Principal" de la lista
  Y escribo Correo electrónico "carlos.cajero@sofia-manual.test"
  Y hago clic en "Guardar"
  Entonces veo "Empleado guardado correctamente"
  Y el sistema me lleva a "/empleados" con los filtros vacíos y sin tabla
  Y el empleado pertenece a la empresa de mi token

Escenario: Campos obligatorios
  Dado que el formulario está vacío
  Cuando hago clic en "Guardar"
  Entonces "Nombres", "Apellido Paterno" y "Sucursal Base" muestran "Este campo es obligatorio" y no se envía nada

Escenario: La sucursal base se elige de la lista
  Dado que escribo "Sed" en "Sucursal Base"
  Entonces la lista muestra hasta 20 sucursales cuyo nombre contiene el texto, por ejemplo "Sede Principal"
  Y si nada coincide veo la opción deshabilitada "Sin resultados"
  Y si no elijo ninguna opción veo "Este campo es obligatorio"

Escenario: La sucursal base debe existir en la empresa
  Dado que la API recibe una sucursal inexistente, eliminada o de otra empresa
  Cuando guarda
  Entonces responde 404 "Sucursal.NotFound" y veo "Error al guardar el empleado"

Escenario: Longitudes máximas
  Entonces "Nombres", "Apellido Paterno" y "Apellido Materno" se detienen en 75 caracteres, "Licencia Profesional" en 50 y "Correo electrónico" en 254

Escenario: El apellido materno es opcional
  Dado que dejo vacío "Apellido Materno"
  Cuando guardo
  Entonces el empleado se guarda sin apellido materno (nulo, nunca un texto de relleno) y la columna aparece vacía en la tabla

Escenario: La licencia profesional es opcional y única
  Dado que Nora tiene la licencia "QF-DEMO-0001"
  Cuando registro a otro empleado con "QF-DEMO-0001"
  Entonces veo "Ya existe un empleado con esa licencia profesional."
  Y sin licencia no hay conflicto

Escenario: El correo es opcional, con formato válido y único en la empresa
  Dado que escribo "carlos.cajero" en "Correo electrónico"
  Entonces veo debajo del campo "Ingresa un correo electrónico válido."
  Y bajo el campo se lee la ayuda "Opcional. Se usa para enviar el enlace de recuperación de contraseña."

Escenario: Correo sin punto tras la arroba
  Dado que escribo "carlos.cajero@sofia"
  Cuando hago clic en "Guardar"
  Entonces la pantalla lo acepta pero el servidor lo rechaza
  Y veo el aviso rojo "Ingresa un correo electrónico válido." y me quedo con lo que escribí

Escenario: Correo repetido en la empresa
  Dado que Carlos ya tiene "carlos.cajero@sofia-manual.test"
  Cuando registro a Nora con ese correo, aunque sea en mayúsculas
  Entonces veo "Ya existe un empleado con ese correo electrónico en tu empresa."
  Y el mismo correo sí puede existir en otra empresa

Escenario: Normalización del correo
  Dado que escribo " Carlos.Cajero@Sofia-Manual.test " con espacios
  Cuando guardo
  Entonces se guarda sin espacios sobrantes y en minúsculas, y un valor en blanco se guarda como nulo

Escenario: Alta simultánea con el mismo valor único
  Dado que otra persona guarda la misma licencia o correo a la vez
  Cuando la base de datos rechaza el valor
  Entonces veo "Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo."

Escenario: Cancelar
  Dado que escribí datos
  Cuando hago clic en "Cancelar"
  Entonces vuelvo a "/empleados" sin guardar
```

### HU-ADM-025 — Editar un empleado

**Como** administrador **quiero** corregir los datos de un empleado o cambiarle de sucursal base **para** mantener su ficha al día.

| | |
|---|---|
| **Módulo** | Empleados |
| **Pantalla** | `/empleados/:id` (botón «Editar») |
| **Endpoint(s)** | `GET /api/v1/Empleados/{id}`; `PUT /api/v1/Empleados/{id}` |
| **Permiso** | `Empleados:Leer`; `Empleados:Actualizar` para editar |
| **Reglas** | [RN-EPL-002, RN-EPL-003, RN-EPL-008, RN-EPL-010, RN-EPL-011, RN-EPL-012, RN-EPL-013, RN-EPL-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-036](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Editar un empleado
  Dado que abrí la ficha de "Pedro Prueba" y tengo "Empleados › Actualizar"
  Cuando hago clic en "Editar", agrego el apellido materno "Borrar" y hago clic en "Guardar"
  Entonces veo "Empleado guardado correctamente"
  Y el sistema me lleva a la lista de empleados (no me quedo en la ficha)

Escenario: La ficha de solo lectura
  Dado que mi rol solo tiene "Empleados › Leer"
  Entonces veo los seis campos deshabilitados y no veo "Editar" ni "Eliminar"

Escenario: Editar reemplaza todos los datos
  Dado que edito un empleado
  Entonces los datos que llegan reemplazan a todos los anteriores, incluida la huella biométrica (dato binario opcional)

Escenario: Las reglas de licencia y correo se aplican al editar
  Dado que cambio la licencia por una de otro empleado o el correo por uno ya registrado
  Cuando guardo
  Entonces veo "Ya existe un empleado con esa licencia profesional." o "Ya existe un empleado con ese correo electrónico en tu empresa."
  Y el valor solo se compara con otros empleados cuando cambia

Escenario: Nadie cambia su propia sucursal base
  Dado que edito mi propio empleado y elijo otra sucursal base
  Cuando guardo
  Entonces la API responde 403 "Empleado.SucursalBase.AutoAsignacion"
  Y veo "Error al guardar el empleado" sin más explicación

Escenario: Edición simultánea con un valor único repetido
  Dado que otra persona guarda la misma licencia o correo a la vez
  Cuando la base de datos rechaza el valor al editar
  Entonces veo "Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo."

Escenario: Cambiar la sucursal base cierra la sesión del empleado
  Dado que Carlos tiene cuenta y la sesión abierta
  Cuando un administrador cambia su sucursal base
  Entonces cambia el sello de seguridad de sus cuentas y su token de acceso anterior deja de servir
  # No verificado en la aplicación: viene de las reglas RN-EPL-011 y RN-EPL-012

Escenario: Empleado inexistente o de otra empresa
  Dado que el identificador no existe en mi empresa
  Entonces la API responde 404 "Empleado.NotFound" y veo "Error al cargar el empleado" o "Error al guardar el empleado"
```

### HU-ADM-026 — Eliminar un empleado

**Como** administrador **quiero** dar de baja a un empleado **para** que no aparezca más en las listas.

| | |
|---|---|
| **Módulo** | Empleados |
| **Pantalla** | `/empleados/:id` en modo editar (botón «Eliminar») |
| **Endpoint(s)** | `DELETE /api/v1/Empleados/{id}` |
| **Permiso** | `Empleados:Eliminar` |
| **Reglas** | [RN-EPL-013, RN-EPL-014, RN-EPL-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (no valida si el empleado tiene cuenta de usuario) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un empleado
  Dado que "Pedro Prueba" no gerencia ninguna sucursal
  Cuando hago clic en "Editar", luego en "Eliminar"
  Entonces se abre el cuadro "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este empleado?"
  Y al hacer clic en "Confirmar" veo "Empleado eliminado correctamente" y vuelvo a la lista, donde ya no aparece

Escenario: El empleado es gerente de una sucursal
  Dado que "Pedro Prueba" es gerente de "Sucursal Demo Norte"
  Cuando confirmo la eliminación
  Entonces la API responde 409 "Empleado.EsGerente"
  Y veo el aviso rojo "No se puede eliminar un empleado que gerencia una sucursal. Asigna otro gerente primero."
  Y el empleado no se borra

Escenario: Quitar el gerente para poder eliminar
  Dado que cambio o borro el "Gerente" de la sucursal y guardo ([HU-ADM-019](#hu-adm-019--ver-y-editar-una-sucursal))
  Cuando elimino al empleado
  Entonces la eliminación procede

Escenario: Otros errores
  Dado que la respuesta no es un 409
  Entonces veo "Error al eliminar el empleado"

Escenario: Eliminar un empleado que tiene cuenta de usuario
  Dado que "Pedro Prueba" tiene la cuenta "pedro.prueba"
  Cuando confirmo la eliminación
  Entonces veo "Empleado eliminado correctamente" (el sistema no lo impide)
  Y la cuenta desaparece de la lista de cuentas y no se puede abrir
  Y su nombre de usuario sigue ocupado
```

> **Brecha conocida.** El handler no comprueba si el empleado tiene cuenta ni otras referencias (RN-EPL-015). Comprobado en la aplicación (manual cap. 3, §9.1): la cuenta queda oculta, inaccesible desde la pantalla y con el usuario ocupado (crear otra con `pedro.prueba` da «Error al guardar la cuenta»). El orden correcto para dar de baja a una persona es eliminar primero su cuenta ([HU-ADM-039](#hu-adm-039--eliminar-una-cuenta)) y luego su empleado. La pantalla muestra el texto de «es gerente» ante cualquier 409 (hallazgo 3 de la sección 12.3 del catálogo). No existe carga masiva de empleados.

---

## 8. Roles y permisos

### HU-ADM-027 — Buscar roles y exportarlos a Excel

**Como** administrador **quiero** buscar los roles de mi empresa por nombre, descripción y nivel, y exportar la lista **para** saber qué perfiles existen.

| | |
|---|---|
| **Módulo** | Roles y permisos |
| **Pantalla** | `/roles` — «Roles» (migas «Inicio / Administración / Roles & Permisos»; tarjeta «Búsqueda de Roles») |
| **Endpoint(s)** | `GET /api/v1/Roles`; `POST /api/v1/Roles/exportar` |
| **Permiso** | `Seguridad:Leer` |
| **Reglas** | [RN-ROL-001, RN-ROL-019, RN-ROL-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar roles
  Dado que mi empresa tiene los roles "Cajero" (nivel 30) y "Admin" (nivel 0)
  Cuando abro "/roles" y hago clic en "Buscar"
  Entonces veo "Resultados (2)" con las columnas "Nombre del Rol", "Descripción" ("—" si no tiene) y "Nivel de Jerarquía"
  Y los roles salen ordenados por nivel, del más alto al más bajo: "Cajero" y luego "Admin"
  Y la tabla no aparece antes de hacer clic en "Buscar"

Escenario: Una empresa recién registrada solo tiene el rol Admin
  Dado que acabo de registrar mi empresa
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (1)": Admin, "Administrador de la empresa", nivel 0
  Y no existe ningún otro rol preconfigurado

Escenario: Filtros de texto
  Dado que escribo "CAJ" en "Nombre del Rol"
  Cuando hago clic en "Buscar"
  Entonces veo "Cajero" (contiene, sin distinguir mayúsculas)
  Y "administrador" en "Descripción" encuentra "Admin"

Escenario: Filtro de nivel con deslizador
  Dado que muevo el mínimo del deslizador a 20 ("Nivel de Jerarquía: 20 — 100")
  Cuando hago clic en "Buscar"
  Entonces veo solo "Cajero"
  Y con el máximo en 20 ("0 — 20") veo solo "Admin"

Escenario: Limpiar
  Dado que hay filtros y tabla
  Cuando hago clic en "Limpiar"
  Entonces los textos quedan vacíos, el deslizador vuelve a 0 — 100 y la tabla se oculta

Escenario: Exportar
  Dado que ya busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "roles.xlsx" con las columnas "Nombre del Rol", "Descripción" y "Nivel de Jerarquía" según los filtros escritos
  # No verificado en la aplicación: el contenido del archivo; el servidor respondió 200 (manual cap. 3, §10.2)

Escenario: Error al buscar o exportar
  Dado que falla la búsqueda o la exportación (también si superan las 10 000 filas)
  Entonces veo "Error al buscar roles"

Escenario: Paginación
  Dado que hay resultados
  Entonces el paginador ofrece 10, 25 y 50 por página
```

### HU-ADM-028 — Crear un rol

**Como** administrador **quiero** crear un rol con nombre, nivel y descripción **para** después asignarle permisos y asociarlo a las cuentas de un puesto, por ejemplo «Cajero».

| | |
|---|---|
| **Módulo** | Roles y permisos |
| **Pantalla** | `/roles/nueva` — «Roles» (botón «Nuevo») |
| **Endpoint(s)** | `POST /api/v1/Roles` |
| **Permiso** | `Seguridad:Crear` (la ruta exige además `Seguridad:Leer`) |
| **Reglas** | [RN-ROL-001, RN-ROL-002, RN-ROL-003, RN-ROL-004, RN-ROL-005, RN-ROL-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013, RN-GEN-036](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (con brechas de mensajes) |

**Criterios de aceptación**

```gherkin
Escenario: Crear el rol Cajero
  Dado que estoy en "/roles/nueva" con los campos "Nombre del Rol", "Nivel de Jerarquía" y "Descripción"
  Cuando escribo Nombre del Rol "Cajero", Nivel de Jerarquía 30 y Descripción "Cajero de mostrador: atención al público y punto de venta"
  Y hago clic en "Guardar"
  Entonces veo "Rol guardado correctamente"
  Y el sistema me lleva a "/roles" sin tabla hasta que hago clic en "Buscar"
  Y se registra el evento de auditoría "ROL_CREAR"
  Y el rol nace sin permisos ni sucursales

Escenario: Crear y configurar son dos pasos
  Dado que estoy creando un rol
  Entonces no existe la tarjeta "Configuración del Rol" en la pantalla de creación
  Y para marcar permisos y sucursales debo volver a entrar al rol con "Editar" ([HU-ADM-030](#hu-adm-030--definir-los-permisos-de-un-rol))

Escenario: Campos obligatorios
  Dado que el formulario está como se abre (nivel en 0)
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" bajo "Nombre del Rol" y bajo "Nivel de Jerarquía"

Escenario: El nivel mínimo en la pantalla es 1
  Dado que dejo el nivel en 0
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" (el mismo mensaje que si estuviera vacío)
  Pero la API acepta 0 y vale 0 si no se indica al crear

Escenario: El nombre no puede repetirse en la empresa
  Dado que ya existe el rol "Admin"
  Cuando creo un rol con nombre "Admin" y nivel 5
  Entonces veo el aviso rojo "Error al guardar el rol" y me quedo en la pantalla
  Y es la única forma en que el sistema avisa del nombre duplicado

Escenario: Nombre de 51 a 100 caracteres
  Dado que escribo un nombre de 60 caracteres (el campo admite hasta 100)
  Cuando hago clic en "Guardar"
  Entonces la API lo rechaza (el máximo es 50) y veo "Error al guardar el rol"

Escenario: La descripción es opcional
  Entonces la descripción admite hasta 255 caracteres y puede quedar vacía

Escenario: Alta simultánea con el mismo nombre
  Dado que otra persona crea el mismo nombre a la vez
  Cuando la base de datos rechaza el valor
  Entonces la API responde 409 "Db.Duplicado" y veo "Error al guardar el rol"

Escenario: El nivel de jerarquía no limita nada
  Dado que un rol de nivel 5 y otro de nivel 90 tienen los mismos permisos
  Entonces ambos pueden hacer lo mismo: el nivel solo ordena la lista

Escenario: Cancelar
  Dado que escribí datos
  Cuando hago clic en "Cancelar"
  Entonces vuelvo a "/roles"
```

> **Brecha conocida.** El formulario admite 100 caracteres de nombre pero la API 50; el formulario exige nivel mínimo 1 y la API acepta 0; el nombre duplicado (`Rol.DuplicateName`, que la API responde con 400 aunque su tipo es «Conflict») y los nombres largos terminan en el mismo genérico «Error al guardar el rol» (RN-ROL-002, RN-ROL-004 y RN-ROL-005; hallazgos 1, 2 y 4 de la sección 12.3 del catálogo).

### HU-ADM-029 — Editar la descripción y el nivel de un rol

**Como** administrador **quiero** cambiar la descripción y el nivel de un rol **para** mantener claro para qué sirve.

| | |
|---|---|
| **Módulo** | Roles y permisos |
| **Pantalla** | `/roles/:id` (botón «Editar») |
| **Endpoint(s)** | `GET /api/v1/Roles/{id}`; `PUT /api/v1/Roles/{id}` |
| **Permiso** | `Seguridad:Leer`; `Seguridad:Actualizar` para editar |
| **Reglas** | [RN-ROL-003, RN-ROL-004, RN-ROL-006, RN-ROL-007, RN-ROL-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-041](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver la ficha de un rol
  Dado que hice clic en "Cajero" en la búsqueda
  Entonces veo el título "Roles", "Datos del registro", los campos deshabilitados "Nombre del Rol", "Nivel de Jerarquía" y "Descripción" y la tarjeta "Configuración del Rol"
  Y el botón "Editar" solo si tengo "Seguridad › Actualizar"

Escenario: Editar descripción y nivel
  Dado que hice clic en "Editar"
  Entonces se habilitan la descripción y el nivel
  Y el nombre sigue deshabilitado
  Y aparecen "Cancelar", "Guardar" y, con "Seguridad › Eliminar", "Eliminar"

Escenario: Guardar
  Dado que cambié el nivel a 35
  Cuando hago clic en "Guardar"
  Entonces veo "Rol guardado correctamente"
  Y el rol vuelve al modo ver y me quedo en la misma ficha, no en la lista
  Y se registra el evento "ROL_ACTUALIZAR" con el nivel como detalle

Escenario: El nombre de un rol no se cambia
  Dado que el rol ya existe
  Entonces la edición solo acepta descripción y nivel

Escenario: Cancelar
  Dado que cambié la descripción y marqué casillas
  Cuando hago clic en "Cancelar"
  Entonces todo vuelve a como estaba: descripción, nivel, permisos y sucursales

Escenario: El nivel es requerido en el cuerpo
  Dado que llamo a "PUT /api/v1/Roles/{id}" sin "NivelJerarquia"
  Entonces la API responde 400 sin campo "code"

Escenario: Rol inexistente o de otra empresa
  Dado que el identificador no existe en mi empresa
  Entonces la API responde 404 "Rol.NotFound" y veo "Error al cargar el rol" o "Error al guardar el rol"

Escenario: El guardado de la ficha no es atómico
  Dado que "Guardar" envía en secuencia datos, sucursales y permisos
  Cuando falla un paso intermedio
  Entonces los pasos anteriores ya quedaron guardados y la pantalla muestra solo el error
```

> **Brecha conocida.** El texto «El nombre del rol es un identificador del sistema y no puede modificarse.» (`roles.detail.nombreRolHint`) existe en `es.json` pero la plantilla no lo usa: el usuario ve el nombre deshabilitado sin explicación (inventario 19b §4.2; verificado: sin referencias en `src/app`). Las pantallas no impiden editar o eliminar el rol «Admin»; **no verificado** si la API lo impide (inventario 19b §7, punto 3).

### HU-ADM-030 — Definir los permisos de un rol

**Como** administrador **quiero** marcar en una matriz qué puede hacer cada rol **para** que cada puesto vea y use solo lo que le corresponde.

| | |
|---|---|
| **Módulo** | Roles y permisos |
| **Pantalla** | `/roles/:id` › «Configuración del Rol» › pestañas «Menú» y «Permisos» |
| **Endpoint(s)** | `GET /api/v1/Roles/permissions/catalog`; `GET /api/v1/Roles/{id}/permissions`; `POST /api/v1/Roles/permissions`; `DELETE /api/v1/Roles/permissions/{permisoId}` |
| **Permiso** | `Seguridad:Leer` para ver; `Seguridad:GestionarPermisos` para marcar y guardar |
| **Reglas** | [RN-ROL-001, RN-ROL-010, RN-ROL-011, RN-ROL-012, RN-ROL-013, RN-ROL-014, RN-ROL-015, RN-ROL-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-020, RN-GEN-021, RN-GEN-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: La matriz de permisos
  Dado que abrí el rol "Cajero", hice clic en "Editar" y estoy en la pestaña "Permisos"
  Entonces veo 27 paneles, uno por módulo, ordenados alfabéticamente por el nombre visible
  Y cada panel tiene una casilla del módulo que marca o desmarca todas sus acciones, el contador "N/M acciones" y una casilla por acción
  Y no existe una casilla para marcar todos los módulos a la vez

Escenario: El catálogo sale de la API
  Dado que la matriz muestra los permisos que se pueden asignar
  Entonces son exactamente los pares módulo y acción declarados en los controladores de la API (105 pares en 27 módulos), agrupados por módulo y ordenados alfabéticamente

Escenario: Marcar casillas no guarda nada
  Dado que marco "Ventas › Leer", "Ventas › Crear", "Ventas › Actualizar" y "Ventas › Anular"
  Entonces el contador de "Ventas" pasa a "4/4 acciones"
  Y no se envía nada hasta hacer clic en "Guardar"

Escenario: La vista previa del menú cambia en vivo
  Dado que estoy en la pestaña "Menú"
  Cuando marco o desmarco "Profesionales de Salud › Leer"
  Entonces la pantalla "Profesionales de Salud" cambia de marca verde a círculo vacío antes de guardar
  Y el texto dice "Vista previa del menú real: una pantalla aparece cuando el rol tiene el permiso «Leer» de su módulo."

Escenario: Guardar los permisos
  Dado que marqué los 19 permisos recomendados del cajero
  Cuando hago clic en "Guardar"
  Entonces veo "Rol guardado correctamente" y me quedo en la ficha
  Y los contadores y casillas se conservan al recargar
  Y cada permiso agregado o quitado se registra como "PERMISO_ASIGNAR" o "PERMISO_REVOCAR"
  Y la caché de permisos de ese rol se invalida de inmediato en la instancia que atendió la petición

Escenario: Quitar un permiso
  Dado que desmarco "Profesionales de Salud › Leer" y hago clic en "Guardar"
  Entonces el permiso queda revocado (borrado lógico)
  Y deja de funcionar en todas las cuentas con ese rol, aunque quien ya tiene la sesión abierta lo notará al recargar

Escenario: Cancelar descarta las casillas
  Dado que marqué casillas
  Cuando hago clic en "Cancelar"
  Entonces el contador vuelve a su valor anterior ("1/4 acciones")

Escenario: Un permiso revocado se restaura en lugar de duplicarse
  Dado que el permiso "Ventas › Leer" fue revocado antes
  Cuando lo marco de nuevo y guardo
  Entonces se restaura la misma fila en lugar de crear otra

Escenario: Un permiso no se asigna dos veces
  Dado que el rol ya tiene "Ventas › Leer"
  Cuando la API recibe otra asignación igual
  Entonces responde 400 "Permiso.Duplicate" y la pantalla muestra "Error al guardar el rol"
  Pero la pantalla solo envía los permisos nuevos respecto de los cargados

Escenario: La API no contrasta el permiso con el catálogo
  Dado que llamo a "POST /api/v1/Roles/permissions" con un módulo y una acción que no están en el catálogo
  Cuando ambos tienen valor y hasta 50 caracteres
  Entonces la API los acepta, porque no hay validador de request ni contraste con el catálogo
  Pero si falta el rol, el módulo o la acción responde 400 con el código "Permiso.RolId", "Permiso.Modulo" o "Permiso.Accion" y la pantalla muestra "Error al guardar el rol"

Escenario: Revocar un permiso inexistente
  Dado que el permiso no existe o ya fue revocado
  Cuando la API recibe la revocación
  Entonces responde 404 "Permiso.NotFound" y veo "Error al guardar el rol"

Escenario: Sin la acción Gestionar permisos
  Dado que mi rol no tiene "Roles y Cuentas de Usuario › Gestionar permisos"
  Entonces las casillas están deshabilitadas y "Guardar" no envía permisos ni sucursales
  # No verificado en la aplicación: solo existen un administrador y un cajero (manual cap. 3, §13, punto 4)

Escenario: No ampliar los permisos de mi propio rol
  Dado que no soy Admin y mi cuenta tiene asignado el rol "Gerente"
  Cuando intento agregar un permiso al rol "Gerente"
  Entonces la API responde 403 "Rol.PermisosPropios"
  Y veo "No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado"
  # No verificado en la aplicación: la administradora tiene el rol Admin, que está exento

Escenario: El rol Admin lo puede todo sin casillas
  Dado que abro el rol "Admin" y entro a "Permisos"
  Entonces ninguna casilla está marcada (todos los módulos dicen "0/N acciones")
  Y aun así el administrador puede hacer todo, porque el acceso depende de que el rol se llame "Admin"

Escenario: Un rol sin permisos solo ve el Dashboard
  Dado que desmarco todas las casillas y guardo
  Entonces las cuentas con ese rol solo ven el Dashboard, sin tarjetas de indicadores
  # No verificado en la aplicación (viene de la matriz 05b §5)
```

> **Brecha conocida.** La API no valida módulo ni acción contra el catálogo al asignar un permiso (`Permiso.RolId`, `Permiso.Modulo`, `Permiso.Accion` solo exigen valores no vacíos de hasta 50 caracteres); la pantalla solo ofrece pares del catálogo (RN-ROL-013). El aviso «Los cambios se aplican de inmediato…» se refiere a la caché del servidor; en la pantalla los cambios se aplican al hacer clic en «Guardar». `Permiso.Duplicate` es de tipo Conflict pero responde 400 (hallazgo 1 de la sección 12.3).

### HU-ADM-031 — Definir las sucursales de un rol

**Como** administrador **quiero** asociar sucursales a un rol **para** que todas las cuentas con ese rol puedan operar también en ellas.

| | |
|---|---|
| **Módulo** | Roles y permisos |
| **Pantalla** | `/roles/:id` › «Configuración del Rol» › pestaña «Sucursales» |
| **Endpoint(s)** | `GET /api/v1/Roles/{id}/sucursales`; `PUT /api/v1/Roles/{id}/sucursales` |
| **Permiso** | `Seguridad:Leer` para ver; `Seguridad:GestionarPermisos` para guardar |
| **Reglas** | [RN-ROL-001, RN-ROL-017, RN-ROL-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver las sucursales del rol
  Dado que abro el rol "Cajero"
  Cuando entro a la pestaña "Sucursales"
  Entonces veo una casilla por cada sucursal no eliminada de mi empresa, ordenadas por nombre ("Sede Principal" y "Sucursal Demo Norte"), con las asignadas marcadas
  Y si no hay sucursales veo "No hay sucursales disponibles"

Escenario: Asignar sucursales
  Dado que marco "Sede Principal" y hago clic en "Guardar"
  Entonces veo "Rol guardado correctamente"
  Y las cuentas con ese rol pueden operar en esa sucursal además de su sucursal base y de las que se les agreguen a cada una

Escenario: Guardar reemplaza el conjunto completo
  Dado que desmarco todas las sucursales y guardo
  Entonces el rol queda sin sucursales (una lista vacía las quita todas)
  Y "Guardar" envía siempre el conjunto, aunque no haya tocado nada

Escenario: Auditoría
  Dado que guardo las sucursales del rol
  Entonces se registra el evento "ROL_SUCURSALES" con la cantidad de sucursales

Escenario: Sucursal inexistente o de otra empresa
  Dado que la API recibe una sucursal que no pertenece a mi empresa o está eliminada
  Cuando guarda
  Entonces responde 404 "Sucursal.NotFound" y veo "Error al guardar el rol"

Escenario: Error al cargar las sucursales
  Dado que falla la lectura
  Entonces veo "Error al cargar las sucursales"
```

> **Nota.** El punto de venta y las ventas se trabajan siempre en la **sucursal base** de la persona; las sucursales adicionales sirven para consultar, por ejemplo, sesiones de caja e inventario de otras sucursales (manual cap. 3, §5.5). **No verificado en la aplicación:** el efecto de la sucursal asignada al rol «Cajero» más allá de la sucursal base (manual cap. 3, §13, punto 10).

### HU-ADM-032 — Eliminar un rol

**Como** administrador **quiero** eliminar un rol que ya no se usa **para** mantener ordenada la lista de roles.

| | |
|---|---|
| **Módulo** | Roles y permisos |
| **Pantalla** | `/roles/:id` en modo editar (botón «Eliminar», al pie de «Configuración del Rol») |
| **Endpoint(s)** | `DELETE /api/v1/Roles/{id}` |
| **Permiso** | `Seguridad:Eliminar` |
| **Reglas** | [RN-ROL-007, RN-ROL-008, RN-ROL-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-003, RN-GEN-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (con brechas de mensajes) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un rol sin cuentas
  Dado que "Rol Prueba Eliminar" no tiene cuentas vinculadas
  Cuando hago clic en "Editar", luego en "Eliminar"
  Entonces se abre el cuadro "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este rol?"
  Y al hacer clic en "Confirmar" veo "Rol eliminado correctamente" y vuelvo a la lista de roles
  Y el borrado es lógico, se invalida la caché de permisos de ese nombre de rol y se registra "ROL_ELIMINAR"

Escenario: Cancelar la confirmación
  Dado que se abrió el cuadro de confirmación
  Cuando hago clic en "Cancelar"
  Entonces el rol no se borra y sigo en modo editar

Escenario: El rol tiene cuentas
  Dado que "Rol Prueba Eliminar" está asignado a la cuenta "pedro.prueba"
  Cuando confirmo la eliminación
  Entonces la API responde 400 "Rol.InUse"
  Y veo el aviso rojo "Error al eliminar el rol" sin más explicación y el rol sigue existiendo
  Y para poder eliminarlo, primero cambio el rol de esas cuentas ([HU-ADM-037](#hu-adm-037--asignar-o-cambiar-el-rol-de-una-cuenta))

Escenario: Un rol nuevo con el mismo nombre no hereda permisos
  Dado que eliminé el rol "Cajero" y creo otro llamado "Cajero"
  Entonces el rol nuevo no hereda los permisos en caché del anterior

Escenario: Rol inexistente
  Dado que el rol no existe en mi empresa
  Entonces la API responde 404 "Rol.NotFound" y veo "Error al eliminar el rol"
```

> **Brecha conocida.** No existe regla que impida eliminar el rol «Admin» si no tiene cuentas vinculadas: la empresa perdería su acceso total (manual cap. 3, §9.3; hallazgo 7 de la sección 12.3). `Rol.InUse` es de tipo Conflict pero responde 400.

### HU-ADM-033 — Cargar roles desde un Excel

**Como** administrador **quiero** crear varios roles desde un archivo **para** no registrarlos uno por uno.

| | |
|---|---|
| **Módulo** | Roles y permisos (carga masiva) |
| **Pantalla** | `/roles/carga-masiva` — «Carga Masiva» (migas «Inicio / Administración / Roles & Permisos / Carga Masiva») |
| **Endpoint(s)** | `GET /api/v1/Roles/plantilla`; `POST /api/v1/Roles/previsualizar`; `POST /api/v1/Roles/carga-masiva` |
| **Permiso** | `Seguridad:Crear` (la ruta exige además `Seguridad:Leer`) |
| **Reglas** | [RN-ROL-001, RN-ROL-020, RN-ROL-021, RN-ROL-022](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-006, RN-IMP-007, RN-IMP-013, RN-IMP-014, RN-IMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (las reglas del archivo están en [HU-ADM-046](#hu-adm-046--cargar-un-excel-con-la-pantalla-de-carga-masiva)) |

**Criterios de aceptación**

```gherkin
Escenario: Plantilla de roles
  Dado que estoy en "/roles/carga-masiva"
  Cuando hago clic en "Descargar plantilla"
  Entonces se descarga "plantilla-roles.xlsx" con las columnas "NombreRol", "Descripcion" y "NivelJerarquia"
  # No verificado en la aplicación: el contenido del archivo descargado

Escenario: Cargar un rol válido
  Dado que mi archivo tiene la fila "Rol Carga Uno", sin descripción, nivel 20
  Cuando lo elijo y veo "1 filas totales" y "1 filas válidas"
  Y hago clic en "Guardar válidos (1)"
  Entonces veo "Se guardaron 1 registros correctamente" y voy a la lista de roles
  Y el rol nace sin permisos ni sucursales: debo abrirlo con "Editar" para marcarlos
  Y la carga masiva no genera eventos de auditoría

Esquema del escenario: Errores por fila en la vista previa
  Dado que mi archivo tiene una fila con <problema>
  Cuando lo elijo
  Entonces la fila queda con error y la columna de errores muestra <mensaje>

  Ejemplos:
    | problema                          | mensaje                                                                     |
    | Nivel 0                           | 'nivelJerarquia' debe ser mayor o igual a 1                                 |
    | Nivel "abc"                       | 'nivelJerarquia' debe ser un número entero                                  |
    | Nivel vacío                       | El campo 'nivelJerarquia' es obligatorio                                    |
    | Nombre vacío                      | El campo 'nombreRol' es obligatorio                                         |
    | Nombre "Cajero" que ya existe     | El valor 'Cajero' del campo 'nombreRol' ya existe en el sistema             |
    | Nombre repetido en dos filas      | El valor 'Rol Carga Repetido' del campo 'nombreRol' está duplicado en el archivo |
    | Nombre de más de 50 caracteres    | 'nombreRol' no puede superar los 50 caracteres                              |

Escenario: Una vista previa mixta
  Dado que mi archivo de siete filas tiene una válida y seis con error
  Entonces veo "7 filas totales", "1 filas válidas" y "6 filas con error"
  Y las dos filas con nombre repetido quedan marcadas, incluida la primera

Escenario: La vista previa exige nivel 1 y el guardado acepta 0
  Dado que la pantalla solo guarda las filas que pasaron la vista previa
  Entonces en la práctica los niveles guardados son siempre 1 o más

Escenario: Al guardar se omiten los nombres ya existentes
  Dado que entre la vista previa y el guardado alguien creó un rol con el mismo nombre (sin distinguir mayúsculas) o el nombre se repite en una fila anterior
  Cuando guardo
  Entonces esa fila se omite sin error y el aviso dice cuántas se guardaron realmente
  # No verificado en la aplicación

Escenario: Al guardar se vuelve a validar cada fila
  Dado que una fila no cumple nombre obligatorio de hasta 50, descripción de hasta 255 o nivel mayor o igual a 0
  Cuando la API recibe el guardado
  Entonces no se guarda ninguna fila y veo "Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa."

Escenario: Vista previa sin archivo
  Dado que llamo a "POST /api/v1/Roles/previsualizar" sin adjuntar un archivo
  Entonces la API responde 400 con el texto "Debe adjuntar un archivo Excel."
  Y la pantalla, que siempre envía un archivo, mostraría "Error al procesar el archivo"
```

---

## 9. Cuentas de usuario

### HU-ADM-034 — Buscar cuentas de usuario

**Como** administrador **quiero** buscar las cuentas por usuario, empleado, estado y bloqueo **para** encontrar rápido la que necesito administrar.

| | |
|---|---|
| **Módulo** | Cuentas de usuario |
| **Pantalla** | `/cuentas` — «Cuentas de Usuario» (tarjeta «Búsqueda de Cuentas») |
| **Endpoint(s)** | `GET /api/v1/Cuentas` |
| **Permiso** | `Seguridad:Leer` |
| **Reglas** | [RN-CTA-001, RN-CTA-020](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (no existe exportación ni carga masiva de cuentas) |

**Criterios de aceptación**

```gherkin
Escenario: Buscar sin filtros
  Dado que mi empresa recién registrada tiene la cuenta de la administradora
  Cuando abro "/cuentas" y hago clic en "Buscar"
  Entonces veo "Resultados (1)" con las columnas "Usuario", "Empleado", "Estado", "Bloqueado" e "Intentos"
  Y la fila dice "ana.manual", "Ana Demo Prueba", una marca verde, un candado abierto y 0
  Y la tabla ordena por usuario

Escenario: Filtros de texto
  Dado que escribo "CARLOS" en "Nombre de Usuario"
  Entonces la búsqueda encuentra "carlos.cajero" (contiene, sin distinguir mayúsculas)
  Y "ana" en "Empleado" encuentra a "Ana Demo Prueba"

Escenario: Filtros de estado y bloqueo
  Dado que elijo "Inactiva" en "Cuenta Activa"
  Entonces solo veo cuentas desactivadas, o "Resultados (0)" si no hay
  Y "Bloqueado" = "Sí" trae solo las cuentas con fecha de bloqueo posterior a la hora actual

Escenario: Limpiar
  Dado que hay filtros y tabla
  Cuando hago clic en "Limpiar"
  Entonces los filtros quedan vacíos y la tabla se oculta

Escenario: Estado visual
  Entonces la cuenta inactiva se ve con un círculo con una equis y la bloqueada con el candado cerrado
  Y la pantalla no tiene "Exportar" ni carga masiva

Escenario: Error al buscar
  Dado que falla la consulta
  Entonces veo "Error al buscar cuentas"
```

### HU-ADM-035 — Crear la cuenta de un empleado

**Como** administrador **quiero** darle a un empleado un usuario y una contraseña inicial **para** que pueda entrar a SOFIA.

| | |
|---|---|
| **Módulo** | Cuentas de usuario |
| **Pantalla** | `/cuentas/nueva` — «Cuentas de Usuario» (botón «Nuevo») |
| **Endpoint(s)** | `POST /api/v1/Cuentas`; `GET /api/v1/Empleados` (buscar empleado) |
| **Permiso** | `Seguridad:Crear` (la ruta exige además `Seguridad:Leer`; elegir al empleado exige `Empleados:Leer`) |
| **Reglas** | [RN-CTA-001, RN-CTA-002, RN-CTA-003, RN-CTA-004, RN-CTA-005, RN-CTA-006, RN-CTA-007, RN-CTA-008](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-AUT-031](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013, RN-GEN-036](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (con brechas de mensajes) |

**Criterios de aceptación**

```gherkin
Escenario: Crear la cuenta del cajero
  Dado que estoy en "/cuentas/nueva" con los campos "Empleado", "Nombre de Usuario" y "Contraseña"
  Cuando escribo "Ca", elijo "Carlos Cajero Demo" de la lista, escribo Nombre de Usuario "carlos.cajero" y una Contraseña inicial de 8 caracteres o más
  Y hago clic en "Guardar"
  Entonces veo "Cuenta guardada correctamente"
  Y el sistema abre la ficha de la cuenta recién creada ("/cuentas/…") en modo ver
  Y la cuenta nace activa, con 0 intentos fallidos, "Requiere Cambio de Clave: Sí", "Sin roles asignados" y "Sin sucursales adicionales asignadas"
  Y pertenece a mi empresa y se registra el evento "CUENTA_REGISTRAR"
  Y la contraseña se guarda como hash BCrypt y no vuelve a mostrarse

Escenario: El formulario no pide rol ni sucursales
  Dado que la cuenta se acaba de crear
  Entonces una cuenta sin rol no puede hacer nada útil hasta que le asigne uno ([HU-ADM-037](#hu-adm-037--asignar-o-cambiar-el-rol-de-una-cuenta))

Escenario: Campos obligatorios
  Dado que el formulario está vacío
  Cuando hago clic en "Guardar"
  Entonces los tres campos muestran "Este campo es obligatorio" y no se envía nada

Escenario: El empleado se elige de la lista
  Dado que escribo menos de 2 letras
  Entonces no se busca
  Y si escribo un texto que no coincide con ninguna opción, el campo se vacía al salir

Escenario: Usuario demasiado largo
  Dado que escribo un usuario de más de 50 caracteres
  Entonces veo "La longitud máxima es de 100 caracteres"

Escenario: Contraseña de menos de 8 caracteres
  Dado que escribo una contraseña de 7 caracteres y un usuario válido
  Cuando hago clic en "Guardar"
  Entonces no aparece ningún mensaje y no se guarda
  Y la contraseña admite de 8 a 128 caracteres

Escenario: El usuario ya existe en el sistema
  Dado que "ana.manual" ya existe en cualquier empresa
  Cuando creo otra cuenta con "ana.manual"
  Entonces veo el aviso rojo "Error al guardar la cuenta"
  # El usuario es único en todo el sistema entre cuentas no eliminadas, porque el inicio de sesión no pide empresa

Escenario: El empleado ya tiene cuenta
  Dado que "Ana Demo Prueba" ya tiene la cuenta "ana.manual"
  Cuando elijo a "Ana Demo Prueba" y guardo
  Entonces veo "Error al guardar la cuenta"
  Y la lista de empleados también muestra a quienes ya tienen cuenta

Escenario: Alta simultánea del mismo usuario
  Dado que otra persona crea la misma cuenta o el mismo usuario a la vez
  Cuando la base de datos rechaza el valor
  Entonces la API responde 409 "Db.Duplicado"
  Y veo "Error al guardar la cuenta" porque la pantalla de creación no traduce los códigos de la API

Escenario: Empleado inexistente
  Dado que la API recibe un empleado que no existe en mi empresa o está eliminado
  Entonces responde 404 "Empleado.NotFound" y veo "Error al guardar la cuenta"

Escenario: Cancelar
  Dado que escribí datos
  Cuando hago clic en "Cancelar"
  Entonces vuelvo a "/cuentas"
```

> **Brecha conocida.** En la creación no se traduce ningún código de la API: `Auth.DuplicateAccount`, `Auth.DuplicateUsername` y los demás terminan en «Error al guardar la cuenta». El mensaje de longitud dice 100 pero el límite real es 50. La contraseña de 1 a 7 caracteres no muestra mensaje. El campo de contraseña no tiene botón para mostrar el texto. El formulario no avisa de que la persona deberá cambiar la contraseña al ingresar (la API lo impone). Fuentes: inventario 19b §5.2; manual cap. 3, §7.2.

### HU-ADM-036 — Administrar el estado de una cuenta

**Como** administrador **quiero** activar o desactivar una cuenta, forzar el cambio de contraseña y quitar un bloqueo **para** controlar el acceso de mi equipo.

| | |
|---|---|
| **Módulo** | Cuentas de usuario |
| **Pantalla** | `/cuentas/:id` (botón «Editar») |
| **Endpoint(s)** | `GET /api/v1/Cuentas/{id}`; `PUT /api/v1/Cuentas/{id}` |
| **Permiso** | `Seguridad:Leer`; `Seguridad:Actualizar` para editar |
| **Reglas** | [RN-CTA-001, RN-CTA-009, RN-CTA-010, RN-CTA-011, RN-CTA-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (no existe «Restablecer contraseña» de otro usuario ni «Bloquear»; el bloqueo solo es automático) |

**Criterios de aceptación**

```gherkin
Escenario: Ver la ficha de una cuenta
  Dado que hice clic en "carlos.cajero" en la búsqueda
  Entonces veo deshabilitados "Nombre de Usuario", "Empleado", "Intentos Fallidos", "Cuenta Activa" ("Activa" o "Inactiva"), "Bloqueado" ("Sí" o "No"), "Requiere Cambio de Clave", "Bloqueado Hasta" (solo si hay fecha de bloqueo) y "Creado el"
  Y las tarjetas "Roles Asignados" y "Sucursales Asignadas"

Escenario: Desactivar una cuenta
  Dado que hice clic en "Editar"
  Cuando apago el interruptor "Cuenta Activa" y hago clic en "Guardar"
  Entonces veo "Cuenta guardada correctamente" y la ficha muestra "Cuenta Activa: Inactiva"
  Y la persona no puede iniciar sesión, renovar la sesión, cambiar su clave ni usar tokens vivos (se comprueba en cada petición)
  Y el nombre de usuario, el empleado y el resto de la ficha no se borran
  Y se registra el evento "CUENTA_ACTUALIZAR"

Escenario: Reactivar una cuenta
  Dado que la cuenta está inactiva
  Cuando enciendo "Cuenta Activa" y guardo
  Entonces la cuenta vuelve a estar "Activa"
  Y el valor solo cambia si el enviado difiere del actual

Escenario: Forzar el cambio de contraseña
  Dado que enciendo "Forzar Cambio de Contraseña" y guardo
  Entonces la ficha muestra "Requiere Cambio de Clave: Sí"
  Y en su próximo ingreso la persona deberá cambiar su contraseña ([HU-ADM-007](#hu-adm-007--cambiar-la-contraseña-cuando-es-obligatorio))
  Y el interruptor empieza siempre apagado y no hay forma de apagarlo a mano: se apaga solo cuando la persona cambia o restablece su contraseña

Escenario: Desbloquear una cuenta
  Dado que la cuenta tiene "Intentos Fallidos: 5" y "Bloqueado: Sí"
  Cuando marco la casilla "Resetear Intentos Fallidos" y guardo
  Entonces "Intentos Fallidos: 0", "Bloqueado: No" y "Bloqueado Hasta" desaparece de la ficha
  Y es la única forma de desbloquear antes de los 15 minutos

Escenario: Usuario y empleado no se pueden cambiar
  Dado que estoy en modo editar
  Entonces "Nombre de Usuario" y "Empleado" están deshabilitados

Escenario: Cancelar
  Dado que cambié los controles
  Cuando hago clic en "Cancelar"
  Entonces se descartan los cambios (pero no deshace sucursales ya agregadas o quitadas)

Escenario: Solo un administrador modifica la cuenta de un administrador
  Dado que la cuenta "ana.manual" tiene el rol Admin y yo no soy Admin
  Cuando hago clic en "Guardar"
  Entonces la API responde 403 "Cuenta.AdminProtegida"
  Y veo "Solo un administrador puede modificar o eliminar la cuenta de un administrador"
  # No verificado en la aplicación

Escenario: Cuenta inexistente
  Dado que la cuenta no existe en mi empresa o está eliminada
  Entonces la API responde 404 "Cuenta.NotFound" y veo "Error al cargar la cuenta" o "Error al guardar la cuenta"

Escenario: Sin el permiso no hay botón
  Dado que mi rol no tiene "Roles y Cuentas de Usuario › Actualizar"
  Entonces no veo el botón "Editar"
```

> **Brecha conocida.** Si una persona olvida su contraseña y su empleado no tiene correo, la única palanca del administrador es «Forzar Cambio de Contraseña» (que exige conocer la actual); la salida que ofrece hoy la aplicación sería eliminar la cuenta y crearla otra vez (manual cap. 3, §7.5; **no verificado**: recrear una cuenta con el mismo usuario). Al abrir por dirección la ficha de una cuenta cuyo empleado fue eliminado se vio la clave sin traducir `cuentas.detail.load-error` en lugar de «Error al cargar la cuenta» (manual cap. 3, §12, punto 1).

### HU-ADM-037 — Asignar o cambiar el rol de una cuenta

**Como** administrador **quiero** darle a una cuenta el rol que corresponde a su puesto **para** que reciba los permisos de ese rol.

| | |
|---|---|
| **Módulo** | Cuentas de usuario y roles |
| **Pantalla** | `/cuentas/:id` › tarjeta «Roles Asignados» (modo editar) |
| **Endpoint(s)** | `POST /api/v1/Cuentas/{id}/roles/{rolId}`; `DELETE /api/v1/Cuentas/{id}/roles/{rolId}`; `GET /api/v1/Roles` (buscador). Equivalentes sin pantalla: `POST /api/v1/Roles/assign-to-user` y `POST /api/v1/Roles/remove-from-user` |
| **Permiso** | `Seguridad:AsignarRoles` (buscar el rol exige `Seguridad:Leer`) |
| **Reglas** | [RN-CTA-001, RN-CTA-014, RN-CTA-015, RN-CTA-016, RN-CTA-017, RN-CTA-018](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-ROL-001, RN-ROL-007](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (la pantalla trabaja con un solo rol por cuenta y no permite dejarla sin rol; la API admite varios roles) |

**Criterios de aceptación**

```gherkin
Escenario: Asignar el rol Cajero
  Dado que la cuenta "carlos.cajero" muestra "Sin roles asignados" y tengo "Roles y Cuentas de Usuario › Asignar roles"
  Cuando hago clic en "Editar", abro el campo "Agregar rol..." y elijo "Cajero"
  Y hago clic en "Guardar"
  Entonces veo "Cuenta guardada correctamente"
  Y la ficha vuelve al modo ver mostrando "Roles Asignados: Cajero"
  Y el campo "Agregar rol..." sin escribir nada lista todos los roles de mi empresa

Escenario: Cambiar de rol reemplaza el anterior
  Dado que la cuenta ya tiene un rol
  Cuando elijo otro y guardo
  Entonces el sistema quita el rol anterior y asigna el nuevo
  Y para cambiarlo debo borrar el texto del campo y escribir las primeras letras del rol nuevo, porque el rol actual aparece escrito y filtra la lista

Escenario: No se puede dejar una cuenta sin rol desde la pantalla
  Dado que vacío el campo del rol
  Cuando guardo
  Entonces el rol no se quita

Escenario: Efecto sobre la sesión
  Dado que le cambio el rol a una cuenta
  Entonces cambia su sello de seguridad: su sesión actual se cierra y los permisos nuevos se aplican en su siguiente ingreso
  Y se registra "ROL_ASIGNAR_CUENTA" o "ROL_QUITAR_CUENTA"

Escenario: Asignar un rol que ya se tiene o quitar uno que no se tiene
  Dado que la cuenta ya tiene el rol, o no lo tiene
  Cuando se asigna o se quita
  Entonces no cambia nada y la API responde éxito

Escenario: Nadie cambia sus propios roles
  Dado que asigno un rol a mi propia cuenta
  Entonces la API responde 403 "Rol.AutoAsignacion"
  Y veo "Una cuenta no puede modificar sus propios roles"
  # No verificado en la aplicación: no se probó con la cuenta propia para no arriesgar el acceso

Escenario: El rol Admin está reservado
  Dado que no soy Admin
  Cuando intento asignar o quitar el rol "Admin"
  Entonces la API responde 403 "Rol.AdminReservado"
  Y veo "Solo un administrador puede asignar o quitar el rol Admin"
  # No verificado en la aplicación

Escenario: Rol o cuenta inexistentes
  Dado que el rol no existe en mi empresa
  Entonces la API responde 404 "Rol.NotFound" y veo "Error al guardar la cuenta"
  Pero si la cuenta no existe o está eliminada responde 404 "Auth.CuentaNotFound" y veo "La cuenta no existe"

Escenario: Sin el permiso de asignar roles
  Dado que mi rol no tiene "Asignar roles"
  Cuando edito la cuenta
  Entonces la tarjeta "Roles Asignados" sigue siendo solo texto
  # No verificado en la aplicación

Escenario: Falla la asignación después de quitar el rol anterior
  Dado que "Guardar" primero quita el rol actual y luego asigna el nuevo
  Cuando falla la asignación
  Entonces el rol anterior ya fue quitado (la secuencia no es atómica)
```

### HU-ADM-038 — Agregar y quitar sucursales de una cuenta

**Como** administrador **quiero** que una cuenta pueda operar en sucursales además de la de su empleado **para** cubrir a quien trabaja en más de un local.

| | |
|---|---|
| **Módulo** | Cuentas de usuario |
| **Pantalla** | `/cuentas/:id` › tarjeta «Sucursales Asignadas» (modo editar) |
| **Endpoint(s)** | `POST /api/v1/Cuentas/{id}/sucursales/{sucursalId}`; `DELETE /api/v1/Cuentas/{id}/sucursales/{sucursalId}`; `GET /api/v1/Sucursales` (buscador) |
| **Permiso** | `Seguridad:Actualizar` (ver la lista exige `Sucursales:Leer`) |
| **Reglas** | [RN-CTA-001, RN-CTA-014, RN-CTA-019](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Agregar una sucursal
  Dado que estoy en modo editar de la cuenta "carlos.cajero"
  Cuando elijo "Sucursal Demo Norte" en "Agregar sucursal..."
  Entonces se aplica al instante, sin "Guardar" y sin confirmación
  Y veo "Sucursal asignada correctamente" y la sucursal aparece en la lista
  Y se registra "CUENTA_SUC_ASIGNAR"

Escenario: Quitar una sucursal
  Dado que la cuenta tiene "Sucursal Demo Norte" asignada
  Cuando hago clic en el icono de papelera "Quitar sucursal" de su fila
  Entonces veo "Sucursal eliminada correctamente" y la fila desaparece
  Y solo se quita de esta cuenta: la sucursal sigue existiendo
  Y se registra "CUENTA_SUC_QUITAR"

Escenario: Sin sucursales adicionales
  Dado que la cuenta no tiene sucursales adicionales
  Entonces la tarjeta dice "Sin sucursales adicionales asignadas"
  Y la cuenta solo opera en la sucursal base de su empleado

Escenario: Cancelar no deshace los cambios de sucursales
  Dado que agregué una sucursal
  Cuando hago clic en "Cancelar"
  Entonces la sucursal sigue asignada

Escenario: Repetir la operación no cambia nada
  Dado que asigno una sucursal ya asignada o quito una no asignada
  Entonces la API responde éxito sin cambios

Escenario: Sucursal o cuenta inexistente
  Dado que la sucursal no existe en mi empresa o está eliminada
  Entonces la API responde 404 "Sucursal.NotFound" y veo "La sucursal no existe"
  Pero si la cuenta no existe responde 404 "Auth.CuentaNotFound" y veo "La cuenta no existe"

Escenario: El efecto es inmediato, sin renovar el token
  Dado que agrego una sucursal a la cuenta
  Entonces el acceso se calcula en cada petición y no se invalida el token
```

> **No verificado en la aplicación:** «Error al asignar la sucursal» y «Error al eliminar la sucursal» no se pudieron provocar (manual cap. 3, §7.7).

### HU-ADM-039 — Eliminar una cuenta

**Como** administrador **quiero** eliminar la cuenta de quien dejó de trabajar con nosotros **para** que no pueda entrar más y su usuario quede libre.

| | |
|---|---|
| **Módulo** | Cuentas de usuario |
| **Pantalla** | `/cuentas/:id` en modo editar (botón «Eliminar») |
| **Endpoint(s)** | `DELETE /api/v1/Cuentas/{id}` |
| **Permiso** | `Seguridad:Eliminar` |
| **Reglas** | [RN-CTA-001, RN-CTA-009, RN-CTA-010, RN-CTA-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-003, RN-GEN-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar la cuenta
  Dado que estoy en modo editar de la cuenta de prueba
  Cuando hago clic en "Eliminar"
  Entonces se abre el cuadro "Eliminar cuenta" con "¿Estás seguro de que deseas eliminar esta cuenta de usuario? Esta acción no se puede deshacer." y los botones "Cancelar" y "Confirmar"
  Y al hacer clic en "Confirmar" veo "Cuenta eliminada correctamente" y vuelvo a la lista de cuentas, donde ya no aparece
  Y se registra "CUENTA_ELIMINAR"

Escenario: Cancelar la confirmación
  Dado que se abrió el cuadro
  Cuando hago clic en "Cancelar"
  Entonces no se borra nada

Escenario: El empleado no se elimina
  Dado que eliminé la cuenta de Carlos
  Entonces el empleado "Carlos Cajero Demo" sigue en la lista de empleados y se le puede crear otra cuenta
  Y su nombre de usuario y su empleado quedan libres para una cuenta nueva

Escenario: Solo un administrador elimina la cuenta de un administrador
  Dado que no soy Admin y la cuenta tiene el rol Admin
  Cuando confirmo la eliminación
  Entonces la API responde 403 "Cuenta.AdminProtegida"
  Y veo "Solo un administrador puede modificar o eliminar la cuenta de un administrador"
  # No verificado en la aplicación

Escenario: Otros errores
  Dado que falla la eliminación por otro motivo
  Entonces veo "Error al eliminar la cuenta"

Escenario: Sin el permiso no hay botón
  Dado que mi rol no tiene "Roles y Cuentas de Usuario › Eliminar"
  Entonces no veo el botón "Eliminar"
```

> **Brecha conocida.** No hay regla que impida a un usuario eliminar su propia cuenta (hallazgo 7 de la sección 12.3 del catálogo). Los botones del cuadro son los de defecto «Confirmar» y «Cancelar», no los textos que declara el componente (inventario 19b §5.2).

---

## 10. Series fiscales

### HU-ADM-040 — Buscar series fiscales y exportarlas a Excel

**Como** administrador **quiero** buscar las series de numeración por sucursal, prefijo, tipo y estado **para** saber qué series están activas en cada local.

| | |
|---|---|
| **Módulo** | Series fiscales |
| **Pantalla** | `/series-fiscales` — «Series Fiscales» (migas «Inicio / SUNAT / Series Fiscales»; tarjeta «Búsqueda de Series Fiscales») |
| **Endpoint(s)** | `GET /api/v1/SeriesFiscales`; `POST /api/v1/SeriesFiscales/exportar` |
| **Permiso** | `SeriesFiscales:Leer` |
| **Reglas** | [RN-SER-001, RN-SER-014](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-011, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar sin filtros
  Dado que mi empresa tiene las series B001 y BC01 en "Sede Principal" y B002 y BC02 en "Sucursal Demo Norte"
  Cuando abro "/series-fiscales" y hago clic en "Buscar"
  Entonces veo "Resultados (4)" ordenadas por prefijo con las columnas "Serie", "Tipo de Comprobante", "Sucursal", "Correlativo Actual" y "Estado"
  Y la tabla no aparece antes de hacer clic en "Buscar"

Escenario: Filtros que se combinan
  Dado que elijo "Nota de Crédito" en "Tipo de Comprobante" y "Sucursal Demo Norte" en "Sucursal"
  Cuando hago clic en "Buscar"
  Entonces veo solo "BC02"
  Y el filtro "Prefijo de Serie" (máximo 4 caracteres) busca prefijos que contienen el texto, sin distinguir mayúsculas: "c0" encuentra BC01 y BC02
  Y "Estado" ofrece "Todos", "Activa" e "Inactiva"

Escenario: Sin series
  Dado que no hay series
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (0)" y "Sin resultados"

Escenario: Las series de una sucursal eliminada no aparecen
  Dado que la sucursal de una serie fue eliminada
  Cuando busco
  Entonces esa serie no aparece

Escenario: Recordar filtros al volver
  Dado que abrí una serie y volví con la flecha
  Entonces la pantalla recuerda los filtros y repite la búsqueda, mientras no recargue el navegador

Escenario: Limpiar
  Dado que hay filtros y tabla
  Cuando hago clic en "Limpiar"
  Entonces los filtros quedan vacíos y la tabla se oculta

Escenario: Exportar
  Dado que ya busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "series-fiscales.xlsx" con las columnas "Serie", "Tipo de Comprobante", "Sucursal", "Correlativo Actual", "Estado" y "Fecha de Creación"
  Y los textos de tipo y estado y el formato de fecha (dd/MM/yyyy en español, en hora UTC) los envía el cliente
  # No verificado en la aplicación: el contenido del archivo; el servidor respondió 200 (manual cap. 2, §11.1)

Escenario: Formato de fecha inválido en la exportación
  Dado que llamo a la API con un formato de fecha distinto de "dd/MM/yyyy", "MM/dd/yyyy", "dd/MM/yyyy HH:mm" o "MM/dd/yyyy HH:mm"
  Entonces la API responde 400 sin campo "code" y la pantalla muestra su genérico de exportación

Escenario: Errores
  Dado que falla la búsqueda o la exportación
  Entonces veo "Error al buscar series fiscales" o "Error al exportar las series fiscales"
```

### HU-ADM-041 — Crear una serie fiscal

**Como** administrador **quiero** crear la serie de boletas y la de notas de crédito de cada sucursal **para** que el punto de venta pueda cobrar y devolver.

| | |
|---|---|
| **Módulo** | Series fiscales |
| **Pantalla** | `/series-fiscales/nueva` — «Series Fiscales» (botón «Nuevo») |
| **Endpoint(s)** | `POST /api/v1/SeriesFiscales`; `GET /api/v1/Sucursales` (selector de sucursal) |
| **Permiso** | `SeriesFiscales:Crear` (la ruta exige además `SeriesFiscales:Leer`; el selector, `Sucursales:Leer`) |
| **Reglas** | [RN-SER-001, RN-SER-002, RN-SER-003, RN-SER-004, RN-SER-005, RN-SER-006, RN-SER-007, RN-SER-008, RN-SER-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Crear la serie de boletas
  Dado que estoy en "/series-fiscales/nueva" con el tipo en "Boleta", el correlativo en 0 y el estado en "Activa"
  Cuando elijo Sucursal "Sede Principal", escribo Prefijo de Serie "B001" y hago clic en "Guardar"
  Entonces veo "Serie fiscal guardada correctamente"
  Y el sistema me lleva a "/series-fiscales"
  Y se registra "SERIE_CREAR" con tipo, prefijo, correlativo y estado

Escenario: Tipos disponibles
  Dado que abro el selector "Tipo de Comprobante"
  Entonces ofrece "Boleta", "Factura", "Nota de Crédito" y "Proforma"
  Y "Ticket" y "Nota de débito" existen en el sistema pero no se aceptan

Escenario: Campos obligatorios
  Dado que el formulario está como se abre
  Cuando hago clic en "Guardar"
  Entonces "Sucursal" y "Prefijo de Serie" muestran "Este campo es obligatorio" y no se envía nada

Esquema del escenario: El prefijo debe tener 4 caracteres y la letra del tipo
  Dado que elijo el tipo <tipo>
  Cuando escribo el prefijo <prefijo> y salgo del campo
  Entonces <resultado>

  Ejemplos:
    | tipo            | prefijo | resultado                                                                                                             |
    | Boleta          | B001    | no hay error                                                                                                          |
    | Boleta          | F001    | veo "El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante."      |
    | Boleta          | b00     | el campo muestra B00 y veo el mismo mensaje                                                                           |
    | Boleta          | B-01    | veo el mismo mensaje                                                                                                  |
    | Factura         | F001    | no hay error                                                                                                          |
    | Factura         | BC01    | veo el mismo mensaje                                                                                                  |
    | Nota de Crédito | BC01    | no hay error (acepta B o F)                                                                                           |
    | Proforma        | PF01    | no hay error (sin restricción de letra)                                                                               |

Escenario: El prefijo se reevalúa al cambiar el tipo
  Dado que el prefijo "BC01" es válido para "Nota de Crédito" y "Boleta"
  Cuando cambio el tipo a "Factura"
  Entonces veo el mensaje del prefijo

Escenario: El prefijo se convierte a mayúsculas y tiene ayuda
  Dado que escribo "b001"
  Entonces el campo lo muestra como "B001" y no deja escribir más de 4 caracteres
  Y debajo se lee "4 caracteres en mayúsculas. Boleta empieza con B, Factura con F, Nota de Crédito con B o F."

Escenario: Correlativo
  Dado que el campo "Correlativo Actual" empieza en 0 con la ayuda "Último número emitido; el siguiente comprobante usará este valor + 1."
  Cuando escribo -3 o 1.5
  Entonces veo "El correlativo debe ser un número entero mayor o igual a 0."
  Y si lo dejo vacío veo "Este campo es obligatorio"
  Y si escribo 250, el siguiente comprobante será el 251

Escenario: Estado
  Entonces "Estado" ofrece solo "Activa" e "Inactiva" y empieza en "Activa"

Escenario: El prefijo ya existe para el mismo tipo en la empresa
  Dado que la "Sede Principal" ya tiene la Boleta "B001"
  Cuando creo una Boleta "B001" para "Sucursal Demo Norte"
  Entonces veo "Ya existe otra serie de este tipo de comprobante con ese prefijo."
  Y un mismo prefijo sí puede usarse en tipos distintos

Escenario: Solo una serie activa por sucursal y tipo
  Dado que la "Sede Principal" ya tiene una Boleta "Activa"
  Cuando creo otra Boleta "B009" "Activa" en esa sucursal
  Entonces veo "La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra."
  Pero puedo guardarla como "Inactiva": no hay límite de series inactivas

Escenario: Prefijo rechazado por el servidor
  Dado que el validador de la API rechaza el prefijo
  Entonces veo el mismo mensaje "El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante."

Escenario: Sucursal inexistente
  Dado que la sucursal fue eliminada mientras trabajaba
  Entonces la API responde 404 "Sucursal.NotFound" y veo "La sucursal seleccionada no existe."

Escenario: Otros errores
  Dado que ocurre otro error al guardar
  Entonces veo "Error al guardar la serie fiscal"
  Y si no se pueden cargar las sucursales del selector veo "No se pudieron cargar las sucursales"
```

> **Nota.** El punto de venta solo emite boletas y notas de crédito; sin serie de Boleta activa no se puede cobrar y sin serie de Nota de Crédito activa no se puede devolver (manual cap. 2, §9.2; los avisos se ven en las historias de la parte C). Los comprobantes no se envían a SUNAT. Las dos cosas que no ve el usuario antes de guardar: el SPA no anticipa la regla «una sola serie activa por sucursal y tipo» hasta que la API la rechaza.

### HU-ADM-042 — Editar y desactivar una serie fiscal

**Como** administrador **quiero** cambiar o desactivar una serie **para** corregir datos o dejar de usarla sin perder la numeración emitida.

| | |
|---|---|
| **Módulo** | Series fiscales |
| **Pantalla** | `/series-fiscales/:id` (botón «Editar») |
| **Endpoint(s)** | `GET /api/v1/SeriesFiscales/{id}`; `PUT /api/v1/SeriesFiscales/{id}` |
| **Permiso** | `SeriesFiscales:Leer`; `SeriesFiscales:Actualizar` para editar |
| **Reglas** | [RN-SER-002, RN-SER-004, RN-SER-007, RN-SER-008, RN-SER-009, RN-SER-010, RN-SER-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver y editar una serie sin comprobantes
  Dado que abrí la serie "PF01" de la "Sede Principal"
  Cuando hago clic en "Editar", cambio el prefijo a "PF02" y el correlativo a 5 y hago clic en "Guardar"
  Entonces veo "Serie fiscal guardada correctamente"
  Y el sistema me lleva a la búsqueda ("/series-fiscales"), donde la tabla muestra PF02, Proforma, Sede Principal, 5, Activa
  Y se registra "SERIE_ACTUALIZAR"

Escenario: Desactivar una serie
  Dado que estoy en modo editar
  Cuando elijo "Inactiva" en "Estado" y hago clic en "Guardar"
  Entonces veo "Serie fiscal guardada correctamente" y la tabla muestra el estado "Inactiva"
  Y mientras no haya otra serie activa del mismo tipo en la sucursal, el punto de venta no podrá emitir ese tipo de comprobante
  Y para volver a usarla repito los pasos eligiendo "Activa"

Escenario: Activar otra serie del mismo tipo
  Dado que la sucursal ya tiene una Boleta "Activa"
  Cuando activo otra Boleta de la misma sucursal
  Entonces veo "La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra."

Escenario: Prefijo repetido al editar
  Dado que cambio el prefijo por uno que otra serie del mismo tipo ya usa
  Cuando guardo
  Entonces veo "Ya existe otra serie de este tipo de comprobante con ese prefijo."

Escenario: Serie que ya emitió comprobantes
  Dado que la serie ya emitió algún comprobante no eliminado
  Cuando hago clic en "Editar"
  Entonces veo sobre el formulario "Esta serie ya emitió comprobantes: solo puedes cambiar su estado."
  Y solo está habilitado "Estado"; "Sucursal", "Tipo de Comprobante", "Prefijo de Serie" y "Correlativo Actual" quedan bloqueados
  # No verificado en la aplicación: la empresa de prueba todavía no emitió comprobantes

Escenario: La API protege la numeración ya emitida
  Dado que llamo a la API para cambiar la sucursal, el tipo, el prefijo o el correlativo de una serie con comprobantes
  Entonces responde 409 "SerieFiscal.ConComprobantes"
  Y la pantalla muestra "La serie ya emitió comprobantes: solo puedes cambiar su estado."

Escenario: Cancelar
  Dado que cambié datos
  Cuando hago clic en "Cancelar"
  Entonces se restaura lo leído y el formulario vuelve a deshabilitarse

Escenario: Serie o sucursal inexistente
  Dado que alguien eliminó la serie o la sucursal mientras trabajaba
  Entonces veo "La serie fiscal no existe." o "La sucursal seleccionada no existe."
  Y si no puedo abrir la serie veo "Error al cargar la serie fiscal"
```

### HU-ADM-043 — Eliminar una serie fiscal

**Como** administrador **quiero** eliminar una serie creada por error **para** que no estorbe, sin alterar la numeración ya emitida.

| | |
|---|---|
| **Módulo** | Series fiscales |
| **Pantalla** | `/series-fiscales/:id` en modo editar (botón «Eliminar») |
| **Endpoint(s)** | `DELETE /api/v1/SeriesFiscales/{id}` |
| **Permiso** | `SeriesFiscales:Eliminar` |
| **Reglas** | [RN-SER-009, RN-SER-011, RN-SER-012](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una serie sin comprobantes
  Dado que la serie "PF02" no emitió comprobantes
  Cuando hago clic en "Editar" y luego en "Eliminar"
  Entonces se abre el cuadro "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta serie fiscal?"
  Y al hacer clic en "Confirmar" veo "Serie fiscal eliminada correctamente" y vuelvo a la búsqueda, donde ya no aparece
  Y el borrado es lógico y se registra "SERIE_ELIMINAR"

Escenario: Serie con comprobantes emitidos
  Dado que la serie ya emitió comprobantes
  Cuando confirmo la eliminación
  Entonces la API responde 409 "SerieFiscal.InUse"
  Y veo "No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla."
  # No verificado en la aplicación: no hay una serie con comprobantes en la empresa de prueba

Escenario: Otros errores
  Dado que falla la eliminación por otro motivo
  Entonces veo "Error al eliminar la serie fiscal"

Escenario: Serie inexistente
  Dado que la serie no existe en mi empresa
  Entonces la API responde 404 "SerieFiscal.NotFound"

Escenario: Una sucursal con series no se puede eliminar
  Dado que la sucursal tiene series fiscales
  Cuando intento eliminarla
  Entonces las series cuentan como registros asociados y la bloquean ([HU-ADM-020](#hu-adm-020--eliminar-una-sucursal))
```

### HU-ADM-044 — Numerar los comprobantes sin repetir números

**Como** contador de la botica **quiero** que dos ventas simultáneas nunca reciban el mismo número de comprobante **para** que la numeración fiscal sea continua y sin duplicados.

| | |
|---|---|
| **Módulo** | Series fiscales (mecanismo interno) |
| **Pantalla** | Sin pantalla propia; lo usan la emisión de comprobantes de venta y de devolución (parte C) |
| **Endpoint(s)** | Ninguno propio |
| **Permiso** | No aplica (mecanismo interno) |
| **Reglas** | [RN-SER-005, RN-SER-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (interna, sin pantalla ni endpoint propios) |

**Criterios de aceptación**

```gherkin
Escenario: El siguiente número es el correlativo más uno
  Dado que la serie B001 tiene "Correlativo Actual" 0
  Cuando se emite la primera boleta
  Entonces recibe el número 1 y el correlativo actual pasa a 1

Escenario: Continuar una numeración anterior
  Dado que escribí 250 en "Correlativo Actual" al crear la serie
  Cuando se emite la siguiente boleta
  Entonces recibe el número 251

Escenario: Dos emisiones simultáneas
  Dado que dos ventas piden número de la misma serie al mismo tiempo
  Entonces cada una recibe un número distinto, porque el siguiente número se obtiene con una sola instrucción que suma 1 y devuelve el valor nuevo
  Y además existe un índice único por serie y correlativo en los comprobantes
```

> **Nota.** El formato impreso del comprobante (por ejemplo `B001-00000001`) y los avisos «Esta sucursal no tiene una serie de boletas activa…» son de las historias de la parte C. **No verificado en la aplicación:** el comportamiento concurrente (viene del código, `ApplicationDbContext.cs:180-188`, citado en [RN-SER-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)).

---

## 11. Reglas transversales y de plataforma

### HU-ADM-045 — Aislar los datos de cada empresa y conservar el historial

**Como** dueño de una botica **quiero** que mis datos solo los vea mi empresa y que lo que elimino quede como historial **para** que ninguna otra empresa vea mi información y no se pierda lo que ya pasó.

| | |
|---|---|
| **Módulo** | Plataforma (datos multiempresa) |
| **Pantalla** | Todas (efecto visible: cada empresa solo ve sus registros) |
| **Endpoint(s)** | Todos |
| **Permiso** | No aplica (regla transversal) |
| **Reglas** | [RN-GEN-001, RN-GEN-002, RN-GEN-003, RN-GEN-004, RN-GEN-005, RN-GEN-006, RN-GEN-007, RN-GEN-008](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Un registro de otra empresa se comporta como inexistente
  Dado que existe un registro de otra empresa
  Cuando abro su dirección o consulto su identificador
  Entonces la API responde como si no existiera (códigos "*.NotFound")

Escenario: Las consultas se limitan a mi empresa
  Dado que mi token trae el identificador de mi empresa
  Cuando consulto cualquier lista
  Entonces solo recibo filas de mi empresa
  Y si la petición no tiene empresa (anónima o tarea en segundo plano), o la fila no tiene empresa, no coincide ninguna fila

Escenario: Las excepciones son los flujos anteriores a conocer la empresa
  Dado que el inicio de sesión, la renovación, el registro y la recuperación no conocen aún la empresa
  Entonces esos flujos saltan el filtro de empresa de forma explícita

Escenario: La empresa se asigna sola al crear
  Dado que creo un registro sin indicar empresa
  Entonces el sistema le asigna la empresa de mi token
  Pero si el registro ya trae empresa (como en el registro público) se respeta

Escenario: Eliminar es un borrado lógico
  Dado que elimino una sucursal, un empleado, una cuenta, un rol o una serie
  Entonces la fila no se borra: se marca como eliminada con fecha UTC y con el usuario del token (o "SYSTEM" si no hay)
  Y las lecturas ya no la devuelven

Escenario: Quién y cuándo
  Dado que creo o modifico un registro
  Entonces se guardan quién lo hizo (usuario del token o "SYSTEM") y cuándo (UTC)

Escenario: Fechas en UTC
  Dado que se guarda cualquier fecha
  Entonces se convierte a UTC y se lee marcada como UTC; un valor sin zona se interpreta como UTC

Escenario: Filtros de fechas inclusivos
  Dado que filtro empresas por "Vencimiento desde" y "Vencimiento hasta"
  Entonces ambos extremos son inclusivos (mayor o igual y menor o igual), normalizados a UTC
  Y la fecha elegida se envía a medianoche local, sin ajustar al fin del día

Escenario: Eventos de dominio a la bandeja de salida
  Dado que una entidad tiene eventos de dominio pendientes
  Cuando se guarda
  Entonces cada evento se convierte en una fila de la bandeja de salida y se limpian de la entidad
  Y una fila que no pasa su validación se descarta sin error
```

> **No verificado:** que `GET /api/v1/Empresas` muestre solo la empresa propia depende del filtro global y no de un filtro explícito del handler (inventario 19b §7, punto 1).

### HU-ADM-046 — Cargar un Excel con la pantalla de carga masiva

**Como** administrador **quiero** subir un archivo `.xlsx`, revisar la vista previa y guardar solo las filas válidas **para** cargar muchos registros de una vez sin errores.

| | |
|---|---|
| **Módulo** | Carga masiva (componente común; aquí con sucursales y roles) |
| **Pantalla** | `/sucursales/carga-masiva` y `/roles/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET …/plantilla`, `POST …/previsualizar` y `POST …/carga-masiva` de `Sucursales` y `Roles` |
| **Permiso** | `<Módulo>:Crear` (`Sucursales:Crear` o `Seguridad:Crear`) |
| **Reglas** | [RN-IMP-001, RN-IMP-002, RN-IMP-003, RN-IMP-004, RN-IMP-005, RN-IMP-006, RN-IMP-007, RN-IMP-012, RN-IMP-013, RN-IMP-014, RN-IMP-015, RN-IMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-039](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: La pantalla de carga masiva
  Dado que estoy en una pantalla de carga masiva
  Entonces veo la flecha de volver, el título "Carga Masiva", el botón "Descargar plantilla" y una zona que dice "Arrastra tu archivo Excel aquí o haz clic para seleccionar" con "Solo archivos .xlsx (máx. 5 MB y 5000 filas)"

Escenario: La plantilla
  Dado que hago clic en "Descargar plantilla"
  Entonces recibo un libro con una hoja "Plantilla" que solo contiene la fila de cabecera con los nombres de columna del módulo, en negrita, texto blanco sobre azul y centrado
  Y si falla la descarga veo "Error al descargar la plantilla. Intenta de nuevo."

Escenario: Archivo que no es .xlsx
  Dado que elijo un archivo cuyo nombre no termina en ".xlsx" (sin distinguir mayúsculas)
  Entonces veo "El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo."
  Y el navegador no lo envía

Escenario: Archivo renombrado o dañado
  Dado que el archivo no empieza con la firma ZIP o no se puede abrir como libro
  Cuando el servidor lo procesa
  Entonces responde 400 "CargaMasiva.Archivo.Invalido" y veo el mismo mensaje

Escenario: Archivo de más de 5 MB
  Dado que el archivo pesa más de 5 MB
  Entonces veo "El archivo supera el tamaño máximo de 5 MB."
  Y lo mismo ocurre si el servidor responde 413 "Request.TooLarge" (el cuerpo JSON del guardado también tiene el límite de 5 MB)

Escenario: Más de 5000 filas
  Dado que el archivo tiene más de 5000 filas con datos (sin contar la cabecera ni las vacías)
  Entonces la API responde 400 "CargaMasiva.Filas.Excedidas" y veo "El archivo supera el máximo de 5000 filas. Divídelo en varios archivos."
  Y al guardar, una lista vacía o de más de 5000 elementos da el mismo texto, porque el SPA asocia ese texto a cualquier código que termine en ".Rows"

Escenario: Archivo comprimido de forma sospechosa
  Dado que el contenido descomprimido supera 50 MB, o una parte de más de 1 MB se expande más de 100 veces
  Entonces la API responde 400 "CargaMasiva.Archivo.ExcedeDescompresion"
  Y veo "El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida."

Escenario: Cómo se lee el archivo
  Dado que el libro tiene varias hojas
  Entonces se lee solo la primera
  Y la primera fila es la cabecera, comparada sin distinguir mayúsculas ni espacios sobrantes
  Y si un título se repite vale la primera columna, las columnas extra se ignoran y una columna faltante se lee como nula
  Y los valores se leen como texto sin espacios sobrantes y las filas totalmente vacías se ignoran
  Y cada fila conserva su número real de la hoja

Escenario: La vista previa
  Dado que el servidor procesó el archivo
  Entonces veo "Vista previa", "N filas totales", "N filas válidas" y, si hay errores, "N filas con error"
  Y cada fila trae sus errores y una columna "Estado" con marca verde si es válida o icono de error si no
  Y la tabla no tiene paginación: se ven todas las filas
  Y una vista previa nueva reemplaza a la anterior

Escenario: Procesando
  Dado que elijo el archivo
  Entonces veo "Procesando archivo..." mientras responde la vista previa y no se guarda nada todavía
  # No verificado en la aplicación: con archivos pequeños la espera es demasiado breve

Escenario: Guardar solo las filas válidas
  Dado que la vista previa tiene filas válidas
  Cuando hago clic en "Guardar válidos (N)"
  Entonces el SPA envía solo las filas válidas
  Y veo "Se guardaron N registros correctamente" y voy a la lista del módulo
  Y si no hay filas válidas el botón está deshabilitado; en el método, "No hay filas válidas para guardar"

Escenario: Al guardar cada fila se valida de nuevo
  Dado que alguna fila no cumple las reglas del módulo al guardar
  Entonces no se guarda ninguna y el código indica la primera fila y campo inválidos ("CargaMasiva…Command.Rows[i].Campo")
  Y veo "Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa."

Escenario: Errores genéricos
  Dado que falla la vista previa o el guardado por otro motivo
  Entonces veo "Error al procesar el archivo" o "Error al guardar los registros"

Escenario: Vista previa sin archivo adjunto
  Dado que llamo a la API sin adjuntar el archivo
  Entonces responde 400 con el texto plano "Debe adjuntar un archivo Excel."

Escenario: Resolver nombres de otros catálogos
  Dado que una carga referencia otro catálogo por nombre
  Entonces la comparación no distingue mayúsculas y un nombre repetido se considera ambiguo y no se resuelve a ningún registro
  # Lo usan módulos de otras partes del catálogo, no sucursales ni roles
```

> **Nota.** Las pruebas del manual entregaron el archivo `.xlsx` a la pantalla desde el propio navegador porque el navegador integrado no tiene selector de archivos: el gesto de arrastrar o hacer clic **no se verificó en la aplicación** (manual cap. 2, §8.5). Los errores de tamaño, filas y descompresión tampoco se provocaron; vienen de 19a §6.5 y §8.9.

### HU-ADM-047 — Exportar una búsqueda a Excel

**Como** administrador **quiero** exportar los resultados filtrados de una búsqueda **para** trabajarlos fuera del sistema.

| | |
|---|---|
| **Módulo** | Exportación (componente común; aquí con empresas, sucursales, empleados, roles y series fiscales) |
| **Pantalla** | Botón «Exportar» bajo la tabla de cada búsqueda |
| **Endpoint(s)** | `POST …/exportar` de `Empresas`, `Sucursales`, `Empleados`, `Roles` y `SeriesFiscales` |
| **Permiso** | El `Leer` del módulo de cada búsqueda |
| **Reglas** | [RN-IMP-008, RN-IMP-009, RN-IMP-010, RN-IMP-011, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Exportar con los filtros escritos
  Dado que ya hice clic en "Buscar"
  Cuando hago clic en "Exportar"
  Entonces se descarga un libro con una hoja "Data"
  Y la cabecera tiene el mismo estilo de la plantilla, las filas alternas van sombreadas y las columnas se ajustan al contenido
  Y los nulos salen como celda vacía y los números, fechas y booleanos conservan su tipo
  Y los títulos de columna los envía el cliente ya traducidos

Escenario: Más de 10 000 filas
  Dado que la consulta filtrada tiene más de 10 000 filas
  Cuando hago clic en "Exportar"
  Entonces la API responde 400 "Export.TooManyRows", no genera el archivo y pide aplicar filtros
  Y la pantalla muestra el genérico de búsqueda o exportación de ese módulo (por ejemplo "Error al buscar roles" o, en series fiscales, "Error al exportar las series fiscales")

Escenario: Títulos de columna inválidos
  Dado que el cuerpo trae más de 50 títulos, algún título nulo o uno de más de 100 caracteres
  Entonces la API responde 400 sin campo "code" (mensaje del modelo "Export.Headers.Invalid")

Escenario: Etiquetas de celda demasiado largas
  Dado que una etiqueta como "Sí" o "No" tiene más de 20 caracteres
  Entonces la API responde 400 sin campo "code" (mensaje del modelo "Export.Label.Invalid")

Escenario: Formato de fecha
  Dado que el cliente envía un formato de fecha
  Entonces debe ser exactamente "dd/MM/yyyy", "MM/dd/yyyy", "dd/MM/yyyy HH:mm" o "MM/dd/yyyy HH:mm"; otro valor da 400 sin "code" (mensaje del modelo "Export.DateFormat.Invalid")
```

> **Brecha conocida.** El mensaje del backend nunca se muestra: ante estos 400 el usuario ve solo el genérico de la pantalla. **No verificado en la aplicación:** el contenido de los archivos descargados (el navegador integrado no muestra descargas; manual cap. 2, §14, punto 1).

### HU-ADM-048 — Dejar rastro de las acciones de seguridad

**Como** responsable de la botica **quiero** que el sistema registre quién cambió roles, permisos, cuentas, claves, series y datos de la empresa **para** poder investigar un incidente.

| | |
|---|---|
| **Módulo** | Auditoría de seguridad |
| **Pantalla** | Ninguna (no existe pantalla ni endpoint de consulta) |
| **Endpoint(s)** | Ninguno propio (`POST /api/v1/Auditoria/anonimizar` existe y no está implementado: responde 501) |
| **Permiso** | No aplica (automático) |
| **Reglas** | [RN-GEN-012, RN-GEN-013, RN-GEN-014](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Solo API (se registra el rastro; nadie puede consultarlo por la API ni por el SPA) |

**Criterios de aceptación**

```gherkin
Escenario: Un comando auditable exitoso deja un evento
  Dado que un empleado con sesión ejecuta, por ejemplo, "Asignar rol a cuenta"
  Cuando el comando termina con éxito
  Entonces se inserta, en la misma transacción, un evento con el empleado que actúa, la tabla afectada, el id del registro, el tipo de acción, un detalle breve sin datos personales (en "Payload_Nuevo") y la IP del cliente
  Y los eventos de este alcance son PERMISO_ASIGNAR, PERMISO_REVOCAR, ROL_ASIGNAR_CUENTA, ROL_QUITAR_CUENTA, ROL_SUCURSALES, ROL_CREAR, ROL_ACTUALIZAR, ROL_ELIMINAR, CUENTA_REGISTRAR, CUENTA_ACTUALIZAR, CUENTA_ELIMINAR, CUENTA_SUC_ASIGNAR, CUENTA_SUC_QUITAR, CLAVE_CAMBIAR, CLAVE_RESTABLECER, SERIE_CREAR, SERIE_ACTUALIZAR, SERIE_ELIMINAR y EMPRESA_ACTUALIZAR

Escenario: Lo que no se audita por este mecanismo
  Dado que creo o edito una sucursal o un empleado, hago una carga masiva de roles, o inicio o cierro sesión
  Entonces no se inserta ningún evento de auditoría de seguridad

Escenario: Un comando fallido no deja evento
  Dado que el comando termina con error de negocio
  Entonces no se inserta evento
  Pero tampoco se revierte lo que el handler ya hubiera guardado, como el contador de intentos fallidos del inicio de sesión (un Result.Failure sin excepción no revierte la transacción)

Escenario: Una excepción revierte todo
  Dado que el handler lanza una excepción
  Entonces se revierte toda la transacción del comando

Escenario: Forma mínima de un evento
  Dado que se arma un evento
  Entonces el empleado, la tabla afectada (hasta 50 caracteres), el id de registro y el tipo de acción (hasta 20) son obligatorios, y la IP admite hasta 45 caracteres
  Y la fecha es la hora UTC actual si no se indica
  Y si la entrada es inválida el comando se revierte con un error 500 (en el restablecimiento de clave el evento inválido se omite en silencio)

Escenario: Los eventos no se modifican
  Dado que existe un evento
  Entonces la entidad no tiene métodos de modificación ni de borrado

Escenario: La consulta de eventos
  Dado que un administrador quiere ver el rastro
  Entonces no hay pantalla ni endpoint que lea los eventos de seguridad
```

> **Pendiente.** Consultar el rastro de auditoría no es posible hoy; el endpoint `POST /api/v1/Auditoria/anonimizar` (permiso `Auditoria:AnonimizarDatos`) responde siempre 501 `Auditoria.Anonimizacion.NoImplementada` y no hay ruta del SPA para auditoría (05a §10.2). No hay política de retención ni purga.

### HU-ADM-049 — Proteger la API contra abuso y configuraciones inseguras

**Como** equipo técnico de la botica **quiero** que la API limite peticiones, filtre el contenido y arranque solo con una configuración segura **para** reducir el riesgo de ataques y errores.

| | |
|---|---|
| **Módulo** | Plataforma (API) |
| **Pantalla** | Ninguna |
| **Endpoint(s)** | Todos; `GET /health` |
| **Permiso** | Anónimo (`/health`); los demás según cada endpoint |
| **Reglas** | [RN-GEN-010, RN-GEN-011, RN-GEN-015, RN-GEN-025, RN-GEN-026, RN-GEN-027, RN-GEN-029, RN-GEN-030, RN-GEN-031, RN-GEN-032, RN-GEN-033, RN-GEN-034](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Límite general de peticiones
  Dado que un endpoint no tiene política propia
  Cuando una IP supera 200 peticiones por minuto (ventana fija, con hasta 10 en cola)
  Entonces la API responde 429 sin campo "code"

Escenario: Límite de credenciales
  Dado que llamo a inicio de sesión, cambio de clave, recuperación o restablecimiento
  Cuando supero 10 peticiones por minuto por IP, sin cola
  Entonces la API responde 429 sin campo "code"

Escenario: Límite del registro de empresas
  Dado que llamo al registro público
  Cuando supero 5 peticiones por hora por IP, sin cola
  Entonces la API responde 429 sin campo "code"

Escenario: IP real del cliente
  Dado que la petición llega con la cabecera "X-Forwarded-For"
  Entonces solo se acepta si viene de un proxy listado en "ForwardedHeaders:KnownProxies"; sin proxies configurados se ignora
  Y esa IP se usa para límites, registros y auditoría

Escenario: Limpieza de HTML en la entrada
  Dado que envío un texto con etiquetas HTML en cualquier propiedad de primer nivel del request
  Entonces el saneador las elimina (no admite etiquetas, atributos, propiedades CSS ni esquemas de URL)
  Pero no sanea las propiedades "password", "passwordhash", "token", "recoverytoken", "secretkey", "hash", "newpassword" ni "confirmpassword"
  Y no recorre objetos anidados ni listas, como las filas de una carga masiva

Escenario: Cabeceras de seguridad
  Dado que recibo cualquier respuesta
  Entonces lleva "X-Content-Type-Options: nosniff", "X-Frame-Options: DENY", "Referrer-Policy: strict-origin-when-cross-origin", "X-Permitted-Cross-Domain-Policies: none" y una "Content-Security-Policy" con "default-src" y "frame-ancestors" en "none"
  Y las rutas bajo "/api" llevan además "Cache-Control: no-store"

Escenario: HTTPS
  Dado que llamo por HTTP
  Entonces se redirige a HTTPS
  Y fuera del entorno de desarrollo se envía HSTS por 365 días con subdominios

Escenario: Orígenes permitidos
  Dado que estoy en desarrollo
  Entonces solo se aceptan "http://localhost:4200" y "https://localhost:4200"
  Y en otros entornos solo los orígenes de la clave "AllowedOrigins"; si está vacía la API no arranca
  Y se permiten credenciales (cookie de refresco)

Escenario: Configuración mínima del token
  Dado que "Jwt:SecretKey" está vacía o mide menos de 32 bytes, o faltan "Jwt:Issuer" o "Jwt:Audience"
  Entonces la API no arranca

Escenario: Estado del servicio
  Dado que llamo a "GET /health"
  Entonces responde 200 solo con el estado y la hora, sin versión ni entorno, y sin exigir autenticación

Escenario: Denegación por defecto
  Dado que un endpoint no tiene metadatos de autorización
  Entonces exige un usuario autenticado
  Y solo son anónimos "POST auth/login", "auth/refresh", "auth/logout", "auth/forgot-password", "auth/reset-password", "POST empresas/registrar" y "GET /health"

Escenario: Validación previa de cada request
  Dado que un request tiene un validador y alguna regla falla
  Entonces el handler no se ejecuta
  Y la API responde 400 con código "{Request}.{Propiedad del primer fallo}" y el detalle con todos los mensajes unidos por "; "
```

> **Nota.** Estas historias de plataforma son comprobables leyendo código y configuración; **no verificado en la aplicación:** el cuerpo exacto de las respuestas 401, 403, 429 y 400 que genera el framework (sección 12.2 del catálogo de reglas).

### HU-ADM-050 — Recibir respuestas de error coherentes de la API

**Como** integrador o usuario **quiero** que los errores de base de datos, concurrencia, tamaño y fallos inesperados tengan una respuesta clara y sin detalles internos **para** corregir mi petición o pedir soporte con una referencia.

| | |
|---|---|
| **Módulo** | Plataforma (manejo de errores) |
| **Pantalla** | Todas (cada pantalla decide su mensaje; ver los escenarios) |
| **Endpoint(s)** | Todos |
| **Permiso** | No aplica |
| **Reglas** | [RN-GEN-012, RN-GEN-024, RN-GEN-035, RN-GEN-036, RN-GEN-037, RN-GEN-038, RN-GEN-039, RN-GEN-040, RN-GEN-041](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (el SPA no traduce todos los códigos; el mensaje del backend nunca se muestra) |

**Criterios de aceptación**

```gherkin
Escenario: Cada comando es una transacción
  Dado que un comando lanza una excepción
  Entonces se revierte todo lo que había guardado
  Pero un Result.Failure sin excepción no revierte lo ya guardado

Escenario: Conflicto de concurrencia
  Dado que otra operación cambió una fila con "RowVersion" antes que yo
  Entonces la API responde 409 "Concurrency.Conflict" con un "traceId"
  Y en este alcance solo la tabla de tokens de refresco tiene "RowVersion"; sus handlers capturan la excepción

Escenario: Valor único repetido
  Dado que la base de datos rechaza un guardado por un índice o restricción única que el handler no comprobó
  Entonces la API responde 409 "Db.Duplicado"
  Y en Empleados y Sucursales veo "Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo."
  Y en el resto de las pantallas de este alcance veo su mensaje genérico (por ejemplo "Error al guardar el rol")
  Y en el registro del servidor solo se anota el nombre de la restricción, nunca los valores

Escenario: Valor fuera de rango
  Dado que la base de datos rechaza el guardado por una restricción CHECK
  Entonces la API responde 400 "Db.ValorInvalido"
  Y en Empleados y Sucursales veo "Uno de los valores está fuera del rango permitido."

Escenario: Referencia inexistente o en uso
  Dado que la base de datos rechaza el guardado por una clave foránea
  Entonces la API responde 409 "Db.ReferenciaInvalida"
  Y en Empleados y Sucursales veo "El registro hace referencia a datos que ya no existen o todavía está en uso."

Escenario: Cuerpo demasiado grande
  Dado que el cuerpo supera el tamaño permitido del endpoint
  Entonces la API responde 413 "Request.TooLarge"
  Y en la carga masiva veo "El archivo supera el tamaño máximo de 5 MB."
  Pero si no puede leerlo responde "Request.Invalid" con el estado de la excepción y la pantalla muestra su genérico

Escenario: Cuerpo con forma inválida
  Dado que el JSON está mal formado, un tipo es incorrecto, falta un campo marcado como requerido (por ejemplo "NivelJerarquia" al actualizar un rol) o un atributo de validación del modelo no se cumple
  Entonces la API responde 400 sin campo "code" antes de llegar al handler
  Y la pantalla muestra su mensaje genérico

Escenario: Error inesperado
  Dado que ocurre una excepción no prevista
  Entonces la API la registra con su "traceId" y responde 500 con un texto fijo que no revela detalles internos, más el "traceId" para soporte
  Y las pantallas que usan el aviso con referencia añaden "Referencia: <traceId>" con un botón para copiar durante 10 segundos
  Y en el inicio de sesión veo "No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos."

Escenario: Contexto del usuario faltante
  Dado que el token no trae la empresa y creo una sucursal o hago una carga masiva de sucursales
  Entonces la API responde 400 "Auth.Empresa" (aunque su tipo es "Unauthorized")
  Y veo "Error al guardar la sucursal" o "Error al guardar los registros"

Escenario: El mensaje del backend no se muestra
  Dado que la API devuelve el campo "detail" con el texto en inglés
  Entonces ninguna pantalla lo muestra: usa su mapa de códigos o su genérico
```

> **Brecha conocida.** No existe un traductor global de códigos; cada pantalla declara su mapa. Los códigos de negocio de este alcance sin texto propio están listados en el hallazgo 2 de la sección 12.3 del catálogo de reglas (por ejemplo `Rol.DuplicateName`, `Rol.InUse`, `Empresa.NotFound`, `Empresa.RUC.Duplicado` en el detalle de empresa, `Empleado.SucursalBase.AutoAsignacion` y `Export.TooManyRows`). Las claves `errors.forbidden`, `errors.unauthorized`, `errors.notFound`, `errors.serverError` y `errors.networkError` existen y no se usan.

---

## 12. Trazabilidad

Cada fila resume la historia con las reglas que cita, la pantalla, los endpoints y el apartado del manual de usuario donde se recorre en la aplicación ([capítulo 1](../manual-usuario/capitulo-01-primeros-pasos.md), [capítulo 2](../manual-usuario/capitulo-02-empresa-sucursales-series.md), [capítulo 3](../manual-usuario/capitulo-03-empleados-roles-cuentas.md)). Las rutas de endpoint conservan la mayúscula del controlador.

| Historia | Reglas | Pantallas | Endpoints | Capítulo del manual |
|---|---|---|---|---|
| HU-ADM-001 — Registrar la empresa y su administrador | RN-CTA-007, RN-EMP-001 a RN-EMP-003, RN-EMP-005 a RN-EMP-013, RN-GEN-004, RN-GEN-027 | `/auth/register` — «Registrar Empresa» | `POST /api/v1/empresas/registrar` | Cap. 1 §3 |
| HU-ADM-002 — Iniciar sesión | RN-AUT-001 a RN-AUT-004, RN-AUT-008 a RN-AUT-012, RN-AUT-031, RN-CTA-012, RN-GEN-026, RN-GEN-040 | `/auth/login` — «Iniciar sesión» | `POST /api/v1/auth/login` | Cap. 1 §6 |
| HU-ADM-003 — Bloquear la cuenta tras intentos fallidos | RN-AUT-005 a RN-AUT-007, RN-AUT-010, RN-AUT-039, RN-GEN-012 | `/auth/login` y `/cuentas/:id` | `POST /api/v1/auth/login` | Cap. 1 §6.5; cap. 3 §7.6 |
| HU-ADM-004 — Mantener y renovar la sesión | RN-AUT-013 a RN-AUT-019, RN-AUT-023, RN-CTA-018, RN-GEN-015 a RN-GEN-018 | Ninguna propia (guard e interceptor); termina en `/auth/login` | `POST /api/v1/auth/refresh`; `GET /api/v1/auth/me` | Cap. 1 §9.1 |
| HU-ADM-005 — Cerrar sesión | RN-AUT-013, RN-AUT-020 a RN-AUT-022, RN-GEN-017 | Cabecera › «Menú de usuario» › «Cerrar sesión» | `POST /api/v1/auth/logout` | Cap. 1 §5 |
| HU-ADM-006 — Cambiar mi contraseña | RN-AUT-024 a RN-AUT-031, RN-GEN-013, RN-GEN-026 | `/cambiar-clave` — «Cambiar contraseña» | `POST /api/v1/auth/change-password` | Cap. 1 §7 |
| HU-ADM-007 — Cambiar la contraseña cuando es obligatorio | RN-AUT-010, RN-AUT-023, RN-AUT-030, RN-CTA-008, RN-CTA-011, RN-GEN-019 | `/cambiar-clave` en modo forzado | `POST /api/v1/auth/change-password`; `GET /api/v1/auth/me` | Cap. 1 §9.2; cap. 3 §8.1 |
| HU-ADM-008 — Pedir el enlace para recuperar la contraseña | RN-AUT-032 a RN-AUT-036, RN-EPL-009, RN-GEN-026 | `/auth/forgot-password` — «Recupera tu contraseña» | `POST /api/v1/auth/forgot-password` | Cap. 1 §8.1-8.2 |
| HU-ADM-009 — Restablecer la contraseña con el enlace | RN-AUT-037 a RN-AUT-039, RN-GEN-013, RN-GEN-014, RN-GEN-026 | `/auth/reset-password` — «Crea una nueva contraseña» | `POST /api/v1/auth/reset-password` | Cap. 1 §8.3 |
| HU-ADM-010 — Ver solo las opciones de menú que mi rol permite | RN-AUT-023, RN-GEN-020 a RN-GEN-022, RN-ROL-015 | Menú lateral de toda la aplicación | `GET /api/v1/auth/me` | Cap. 1 §4.1; cap. 3 §5.3 |
| HU-ADM-011 — Entrar solo a las pantallas que mi rol permite | RN-CTA-001, RN-EMP-001, RN-EPL-001, RN-GEN-015, RN-GEN-020 a RN-GEN-022, RN-ROL-001, RN-SER-001, RN-SUC-001 | Guard de permisos de toda ruta del layout; destino `/dashboard` | Todos los protegidos con `[HasPermission]`; `GET /api/v1/auth/me` | Cap. 1 §4.1; cap. 3 §8.2 |
| HU-ADM-012 — Cambiar el idioma de la interfaz | Sin regla RN propia | Cabecera › bandera de idioma | Ninguno | Cap. 1 §4.3 |
| HU-ADM-013 — Buscar mi empresa y exportarla a Excel | RN-EMP-001, RN-EMP-017, RN-EMP-018, RN-GEN-001, RN-GEN-009, RN-IMP-008 a RN-IMP-010, RN-IMP-017 | `/empresas` — «Empresas» | `GET /api/v1/Empresas`; `POST /api/v1/Empresas/exportar` | Cap. 2 §3.1 |
| HU-ADM-014 — Ver los datos y la suscripción de mi empresa | RN-EMP-012, RN-EMP-014, RN-EMP-016, RN-EMP-017, RN-GEN-001 | `/empresas/:id` | `GET /api/v1/Empresas/{id}` | Cap. 2 §3.2 |
| HU-ADM-015 — Editar el nombre y el RUC de mi empresa | RN-EMP-001, RN-EMP-002, RN-EMP-004, RN-EMP-005, RN-EMP-017, RN-GEN-013 | `/empresas/:id` (botón «Editar») | `PUT /api/v1/Empresas/{id}` | Cap. 2 §3.3 |
| HU-ADM-016 — Restringir el acceso cuando la suscripción no está vigente | RN-AUT-009, RN-AUT-017, RN-AUT-034, RN-EMP-014, RN-EMP-015, RN-GEN-018 | `/auth/login` (mensaje); sin pantalla de gestión del estado | `POST /api/v1/auth/login`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/forgot-password` y toda petición autenticada | Cap. 1 §6.5; cap. 2 §3.2 |
| HU-ADM-017 — Buscar sucursales y exportarlas a Excel | RN-GEN-001, RN-GEN-009, RN-IMP-008, RN-IMP-009, RN-IMP-017, RN-SUC-001, RN-SUC-011 | `/sucursales` — «Sucursales» | `GET /api/v1/Sucursales`; `POST /api/v1/Sucursales/exportar` | Cap. 2 §6 |
| HU-ADM-018 — Crear una sucursal | RN-GEN-024, RN-GEN-036 a RN-GEN-038, RN-SUC-001 a RN-SUC-007 | `/sucursales/nueva` | `POST /api/v1/Sucursales` | Cap. 2 §5 |
| HU-ADM-019 — Ver y editar una sucursal | RN-EMP-010, RN-GEN-036 a RN-GEN-038, RN-SUC-002 a RN-SUC-006, RN-SUC-008 | `/sucursales/:id` | `GET /api/v1/Sucursales/{id}`; `PUT /api/v1/Sucursales/{id}`; `GET /api/v1/Empleados` | Cap. 2 §4 |
| HU-ADM-020 — Eliminar una sucursal | RN-GEN-003, RN-SUC-008, RN-SUC-009 | `/sucursales/:id` en modo editar | `DELETE /api/v1/Sucursales/{id}` | Cap. 2 §7 |
| HU-ADM-021 — Cargar sucursales desde un Excel | RN-GEN-024, RN-IMP-006, RN-IMP-007, RN-IMP-012, RN-IMP-014 a RN-IMP-016, RN-SUC-001, RN-SUC-012, RN-SUC-013 | `/sucursales/carga-masiva` — «Carga Masiva» | `GET /api/v1/Sucursales/plantilla`; `POST /api/v1/Sucursales/previsualizar`; `POST /api/v1/Sucursales/carga-masiva` | Cap. 2 §8 |
| HU-ADM-022 — Elegir entre las sucursales a las que tengo acceso | RN-GEN-023, RN-SUC-001, RN-SUC-010 | Sin pantalla propia (la usa `/stock-por-sucursal`, parte B) | `GET /api/v1/Sucursales/permitidas` | Cap. 3 §5.5 (sin sección propia) |
| HU-ADM-023 — Buscar empleados y exportarlos a Excel | RN-EPL-001, RN-EPL-016, RN-EPL-017, RN-GEN-001, RN-GEN-009, RN-IMP-008, RN-IMP-009, RN-IMP-017 | `/empleados` — «Empleados» | `GET /api/v1/Empleados`; `POST /api/v1/Empleados/exportar` | Cap. 3 §3.3 |
| HU-ADM-024 — Registrar un empleado | RN-EPL-001 a RN-EPL-010, RN-EPL-018, RN-GEN-036 | `/empleados/nueva` | `POST /api/v1/Empleados`; `GET /api/v1/Sucursales` | Cap. 3 §3.2 |
| HU-ADM-025 — Editar un empleado | RN-EPL-002, RN-EPL-003, RN-EPL-008, RN-EPL-010 a RN-EPL-013, RN-EPL-018, RN-GEN-036 | `/empleados/:id` | `GET /api/v1/Empleados/{id}`; `PUT /api/v1/Empleados/{id}` | Cap. 3 §9.1 |
| HU-ADM-026 — Eliminar un empleado | RN-EPL-013 a RN-EPL-015, RN-GEN-003 | `/empleados/:id` en modo editar | `DELETE /api/v1/Empleados/{id}` | Cap. 3 §9.1 |
| HU-ADM-027 — Buscar roles y exportarlos a Excel | RN-GEN-001, RN-GEN-009, RN-IMP-008, RN-IMP-009, RN-IMP-017, RN-ROL-001, RN-ROL-019, RN-ROL-023 | `/roles` — «Roles» | `GET /api/v1/Roles`; `POST /api/v1/Roles/exportar` | Cap. 3 §10.1-10.2 |
| HU-ADM-028 — Crear un rol | RN-GEN-013, RN-GEN-036, RN-ROL-001 a RN-ROL-005, RN-ROL-009 | `/roles/nueva` | `POST /api/v1/Roles` | Cap. 3 §5.1 |
| HU-ADM-029 — Editar la descripción y el nivel de un rol | RN-GEN-041, RN-ROL-003, RN-ROL-004, RN-ROL-006, RN-ROL-007, RN-ROL-009 | `/roles/:id` | `GET /api/v1/Roles/{id}`; `PUT /api/v1/Roles/{id}` | Cap. 3 §9.3 |
| HU-ADM-030 — Definir los permisos de un rol | RN-GEN-020 a RN-GEN-022, RN-ROL-001, RN-ROL-010 a RN-ROL-016 | `/roles/:id` › pestañas «Menú» y «Permisos» | `GET /api/v1/Roles/permissions/catalog`; `GET /api/v1/Roles/{id}/permissions`; `POST /api/v1/Roles/permissions`; `DELETE /api/v1/Roles/permissions/{permisoId}` | Cap. 3 §5.2-5.4, §6 |
| HU-ADM-031 — Definir las sucursales de un rol | RN-GEN-023, RN-ROL-001, RN-ROL-017, RN-ROL-018 | `/roles/:id` › pestaña «Sucursales» | `GET /api/v1/Roles/{id}/sucursales`; `PUT /api/v1/Roles/{id}/sucursales` | Cap. 3 §5.5 |
| HU-ADM-032 — Eliminar un rol | RN-GEN-003, RN-GEN-022, RN-ROL-007 a RN-ROL-009 | `/roles/:id` en modo editar | `DELETE /api/v1/Roles/{id}` | Cap. 3 §9.3 |
| HU-ADM-033 — Cargar roles desde un Excel | RN-IMP-006, RN-IMP-007, RN-IMP-013, RN-IMP-014, RN-IMP-016, RN-ROL-001, RN-ROL-020 a RN-ROL-022 | `/roles/carga-masiva` — «Carga Masiva» | `GET /api/v1/Roles/plantilla`; `POST /api/v1/Roles/previsualizar`; `POST /api/v1/Roles/carga-masiva` | Cap. 3 §10.3 |
| HU-ADM-034 — Buscar cuentas de usuario | RN-CTA-001, RN-CTA-020, RN-GEN-001, RN-GEN-009 | `/cuentas` — «Cuentas de Usuario» | `GET /api/v1/Cuentas` | Cap. 3 §7.1 |
| HU-ADM-035 — Crear la cuenta de un empleado | RN-AUT-031, RN-CTA-001 a RN-CTA-008, RN-GEN-013, RN-GEN-036 | `/cuentas/nueva` | `POST /api/v1/Cuentas`; `GET /api/v1/Empleados` | Cap. 3 §7.2 |
| HU-ADM-036 — Administrar el estado de una cuenta | RN-CTA-001, RN-CTA-009 a RN-CTA-012, RN-GEN-013 | `/cuentas/:id` | `GET /api/v1/Cuentas/{id}`; `PUT /api/v1/Cuentas/{id}` | Cap. 3 §7.4-7.6 |
| HU-ADM-037 — Asignar o cambiar el rol de una cuenta | RN-CTA-001, RN-CTA-014 a RN-CTA-018, RN-GEN-017, RN-ROL-001, RN-ROL-007 | `/cuentas/:id` › «Roles Asignados» | `POST /api/v1/Cuentas/{id}/roles/{rolId}`; `DELETE /api/v1/Cuentas/{id}/roles/{rolId}`; `GET /api/v1/Roles`; `POST /api/v1/Roles/assign-to-user` y `POST /api/v1/Roles/remove-from-user` (sin pantalla) | Cap. 3 §7.3 |
| HU-ADM-038 — Agregar y quitar sucursales de una cuenta | RN-CTA-001, RN-CTA-014, RN-CTA-019, RN-GEN-023 | `/cuentas/:id` › «Sucursales Asignadas» | `POST /api/v1/Cuentas/{id}/sucursales/{sucursalId}`; `DELETE /api/v1/Cuentas/{id}/sucursales/{sucursalId}`; `GET /api/v1/Sucursales` | Cap. 3 §7.7 |
| HU-ADM-039 — Eliminar una cuenta | RN-CTA-001, RN-CTA-009, RN-CTA-010, RN-CTA-013, RN-GEN-003, RN-GEN-013 | `/cuentas/:id` en modo editar | `DELETE /api/v1/Cuentas/{id}` | Cap. 3 §9.2 |
| HU-ADM-040 — Buscar series fiscales y exportarlas a Excel | RN-GEN-001, RN-GEN-009, RN-IMP-008, RN-IMP-009, RN-IMP-011, RN-IMP-017, RN-SER-001, RN-SER-014 | `/series-fiscales` — «Series Fiscales» | `GET /api/v1/SeriesFiscales`; `POST /api/v1/SeriesFiscales/exportar` | Cap. 2 §11.1 |
| HU-ADM-041 — Crear una serie fiscal | RN-GEN-013, RN-SER-001 a RN-SER-008, RN-SER-012 | `/series-fiscales/nueva` | `POST /api/v1/SeriesFiscales`; `GET /api/v1/Sucursales` | Cap. 2 §9-10 |
| HU-ADM-042 — Editar y desactivar una serie fiscal | RN-SER-002, RN-SER-004, RN-SER-007 a RN-SER-010, RN-SER-012 | `/series-fiscales/:id` | `GET /api/v1/SeriesFiscales/{id}`; `PUT /api/v1/SeriesFiscales/{id}` | Cap. 2 §11.2-11.3 |
| HU-ADM-043 — Eliminar una serie fiscal | RN-GEN-003, RN-SER-009, RN-SER-011, RN-SER-012 | `/series-fiscales/:id` en modo editar | `DELETE /api/v1/SeriesFiscales/{id}` | Cap. 2 §11.4 |
| HU-ADM-044 — Numerar los comprobantes sin repetir números | RN-SER-005, RN-SER-013 | Sin pantalla propia (parte C) | Ninguno propio | Cap. 2 §9.1 |
| HU-ADM-045 — Aislar los datos de cada empresa y conservar el historial | RN-EMP-017, RN-GEN-001 a RN-GEN-008 | Todas | Todos | Sin capítulo (regla de plataforma) |
| HU-ADM-046 — Cargar un Excel con la pantalla de carga masiva | RN-GEN-039, RN-IMP-001 a RN-IMP-007, RN-IMP-012 a RN-IMP-016 | `/sucursales/carga-masiva` y `/roles/carga-masiva` | `GET …/plantilla`, `POST …/previsualizar` y `POST …/carga-masiva` de `Sucursales` y `Roles` | Cap. 2 §8; cap. 3 §10.3 |
| HU-ADM-047 — Exportar una búsqueda a Excel | RN-IMP-008 a RN-IMP-011, RN-IMP-017 | Botón «Exportar» de cada búsqueda | `POST …/exportar` de `Empresas`, `Sucursales`, `Empleados`, `Roles` y `SeriesFiscales` | Cap. 2 §3.1, §6.3, §11.1; cap. 3 §3.3, §10.2 |
| HU-ADM-048 — Dejar rastro de las acciones de seguridad | RN-GEN-012 a RN-GEN-014 | Ninguna | Ninguno propio (`POST /api/v1/Auditoria/anonimizar` responde 501) | Sin capítulo (sin pantalla) |
| HU-ADM-049 — Proteger la API contra abuso y configuraciones inseguras | RN-GEN-010, RN-GEN-011, RN-GEN-015, RN-GEN-025 a RN-GEN-027, RN-GEN-029 a RN-GEN-034 | Ninguna | Todos; `GET /health` | Sin capítulo (plataforma) |
| HU-ADM-050 — Recibir respuestas de error coherentes de la API | RN-GEN-012, RN-GEN-024, RN-GEN-035 a RN-GEN-041 | Todas | Todos | Sin capítulo (plataforma) |

---

## 13. Pendientes y «No verificado»

### 13.1 En curso (sin commitear)

Ninguno. Al empezar, `git status --short` estaba vacío en `sofia-apis` (`43f5529`) y en `sofia-web-spa` (`eeffbb6`); ninguna afirmación de este documento se apoya en un archivo con cambios sin commitear.

### 13.2 Pendientes de producto que las historias dejan visibles

| # | Pendiente | Dónde se refleja |
|---|---|---|
| 1 | Activar, suspender, cancelar y extender la suscripción de una empresa no tiene endpoint ni pantalla ([RN-EMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)) | HU-ADM-016 (Parcial) |
| 2 | La política «exigir registro de receta» solo se cambia en la base de datos ([RN-EMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)) | HU-ADM-014 (Parcial) |
| 3 | No se pueden consultar los eventos de auditoría de seguridad ni hay retención; el endpoint de anonimización responde 501 | HU-ADM-048 (Solo API) |
| 4 | El administrador no puede escribirle una contraseña nueva a otro usuario ni bloquear una cuenta; solo «Forzar Cambio de Contraseña» y desbloqueo | HU-ADM-036 (Parcial) |
| 5 | No existe carga masiva de empleados, cuentas ni series fiscales, ni exportación de cuentas | HU-ADM-023, HU-ADM-034, HU-ADM-040 |
| 6 | Eliminar un empleado con cuenta de usuario no se impide y deja la cuenta oculta con el usuario ocupado | HU-ADM-026 (Parcial) |
| 7 | No hay regla que impida eliminar el rol «Admin» sin cuentas ni eliminar la propia cuenta | HU-ADM-032, HU-ADM-039 |
| 8 | El idioma de la interfaz no se guarda; la campana de la cabecera es decorativa | HU-ADM-012 (Parcial) |
| 9 | El texto `roles.detail.nombreRolHint` no se muestra, así que el nombre bloqueado del rol no tiene explicación | HU-ADM-029 |
| 10 | `EmpresaMaintenanceComponent` existe sin ruta ni referencia | HU-ADM-014 |
| 11 | Formulario y API con límites distintos: nombre de rol (100/50), nivel de rol (1/0), usuario de la cuenta (el mensaje dice 100, el límite es 50), contraseña inicial de la cuenta sin mensaje, nombre de empresa en el detalle (150/200), RUC en el detalle sin validar el formato | HU-ADM-015, HU-ADM-028, HU-ADM-035 |
| 12 | Varios códigos de negocio terminan en el mensaje genérico de la pantalla (sección 12.3, hallazgo 2, del catálogo de reglas) | Brechas conocidas en cada historia |

### 13.3 No verificado

| # | Qué | Por qué |
|---|---|---|
| 1 | Mensajes de límite de peticiones (429), error de servidor en inicio de sesión, recuperación y restablecimiento, bloqueo tras 5 intentos | El manual no los provocó para no bloquear la cuenta de prueba ni superar el límite de registros (manual cap. 1, §12) |
| 2 | Mensaje de inicio de sesión con empresa no vigente | La empresa de prueba está vigente y no hay pantalla para cambiar su estado |
| 3 | Vencimiento real de la sesión y renovación en segundo plano | No se esperó a que venciera el token (manual cap. 1, §12) |
| 4 | Contenido de los archivos descargados (plantillas y exportaciones) | El navegador integrado no muestra las descargas (manual cap. 2, §14, y cap. 3, §13) |
| 5 | Gesto de arrastrar o hacer clic para elegir el archivo en la carga masiva y los errores de tamaño, filas y descompresión | Se entregó el archivo a la pantalla desde el navegador; no se prepararon archivos de ese tipo |
| 6 | Mensajes que exigen un usuario no administrador con permisos de gestión: `Rol.AutoAsignacion`, `Rol.AdminReservado`, `Cuenta.AdminProtegida`, `Rol.PermisosPropios`, casillas sin «Gestionar permisos» | Solo existen un administrador y un cajero sin esos permisos (manual cap. 3, §13) |
| 7 | Qué ve en vivo una persona con la sesión abierta cuando se cambian los permisos de su rol | El navegador integrado mantiene una sola sesión |
| 8 | Efecto de la sucursal asignada al rol más allá de la sucursal base; asignar un gerente a una sucursal; si un empleado puede ser gerente de dos sucursales | Ver HU-ADM-019 y HU-ADM-031 |
| 9 | Si la API impide editar o eliminar el rol «Admin» | El SPA no tiene regla y no se leyó una restricción en `Commands/Roles` (inventario 19b §7, punto 3) |
| 10 | Efecto del saneador de HTML sobre una contraseña con `<` o `>` en el cambio de contraseña | `CurrentPassword` no está en la lista de excluidas (sección 12.2 del catálogo) |
| 11 | Cuerpo exacto de las respuestas 401, 403, 429 y 400 generadas por el framework; duración concreta del token de acceso | Las genera el framework o la configuración (`Jwt:ExpiryMinutes`); no se llamó a la API |
| 12 | Que `GET /api/v1/Empresas` muestre solo la empresa propia | Depende del filtro global, no de un filtro explícito del handler (inventario 19b §7, punto 1) |
| 13 | Vista angosta (menos de 1024 px) del menú y Ctrl + clic en una opción | No se recorrió (manual cap. 1, §12) |
| 14 | Comportamiento concurrente de la numeración de comprobantes | Se apoya en el código (RN-SER-013), no se ejecutó |

---

## 14. Tabla de cobertura

### 14.1 Reglas de negocio del alcance (203)

Cada regla `RN-GEN`, `RN-AUT`, `RN-ROL`, `RN-CTA`, `RN-EMP`, `RN-SUC`, `RN-EPL`, `RN-SER` e `RN-IMP` del [catálogo parte A](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) con las historias que la cubren y el escenario (entre « ») donde se comprueba. «Sin escenario propio» significa que la historia cita la regla en su fila «Reglas» pero la comprueba un escenario de otra historia de la misma fila.

| Regla | Historias y escenarios |
|---|---|
| RN-GEN-001 | HU-ADM-045: «Un registro de otra empresa se comporta como inexistente»; «Las consultas se limitan a mi empresa»; «Las excepciones son los flujos»; HU-ADM-013: «Buscar sin filtros»; HU-ADM-014: «Solo veo mi propia empresa»; HU-ADM-017: «Solo veo las sucursales de mi empresa»; HU-ADM-023, HU-ADM-027, HU-ADM-034 y HU-ADM-040: sin escenario propio |
| RN-GEN-002 | HU-ADM-045: «Eliminar es un borrado lógico» |
| RN-GEN-003 | HU-ADM-045: «Eliminar es un borrado lógico»; HU-ADM-020: «Eliminar una sucursal sin registros asociados»; HU-ADM-026: «Eliminar un empleado»; HU-ADM-032: «Eliminar un rol sin cuentas»; HU-ADM-039: «Eliminar la cuenta»; HU-ADM-043: «Eliminar una serie sin comprobantes» |
| RN-GEN-004 | HU-ADM-045: «La empresa se asigna sola al crear»; HU-ADM-001: «El registro crea todo lo necesario en una sola operación» |
| RN-GEN-005 | HU-ADM-045: «Quién y cuándo» |
| RN-GEN-006 | HU-ADM-045: «Fechas en UTC» |
| RN-GEN-007 | HU-ADM-045: «Eventos de dominio a la bandeja de salida» |
| RN-GEN-008 | HU-ADM-045: «Filtros de fechas inclusivos» |
| RN-GEN-009 | HU-ADM-013: «Paginar»; HU-ADM-017: «Paginación»; «Límites de paginación de la API»; HU-ADM-023: «Limpiar y paginar»; HU-ADM-027: «Paginación»; HU-ADM-034 y HU-ADM-040: sin escenario propio |
| RN-GEN-010 | HU-ADM-049: «Limpieza de HTML en la entrada» |
| RN-GEN-011 | HU-ADM-049: «Validación previa de cada request» |
| RN-GEN-012 | HU-ADM-050: «Cada comando es una transacción»; HU-ADM-048: «Un comando fallido no deja evento»; «Una excepción revierte todo»; HU-ADM-003: «Cada contraseña errónea suma un intento» |
| RN-GEN-013 | HU-ADM-048: «Un comando auditable exitoso deja un evento»; HU-ADM-006: «Cambiar la contraseña»; HU-ADM-009: «Efectos del restablecimiento»; HU-ADM-015: «Editar y guardar»; HU-ADM-028: «Crear el rol Cajero»; HU-ADM-035: «Crear la cuenta del cajero»; HU-ADM-036: «Desactivar una cuenta»; HU-ADM-039: «Eliminar la cuenta»; HU-ADM-041: «Crear la serie de boletas» |
| RN-GEN-014 | HU-ADM-048: «Forma mínima de un evento»; HU-ADM-009: «Efectos del restablecimiento» |
| RN-GEN-015 | HU-ADM-049: «Denegación por defecto»; HU-ADM-004: «Un token no válido recibe 401 sin código»; HU-ADM-011: sin escenario propio |
| RN-GEN-016 | HU-ADM-004: «Un token no válido recibe 401 sin código» |
| RN-GEN-017 | HU-ADM-004: «Un cambio de rol invalida el token actual»; HU-ADM-005: «Se cierran todas las sesiones de la cuenta»; HU-ADM-037: «Efecto sobre la sesión» |
| RN-GEN-018 | HU-ADM-016: «Las sesiones vivas se cortan en un minuto como máximo»; HU-ADM-004: sin escenario propio |
| RN-GEN-019 | HU-ADM-007: «El administrador fuerza el cambio con mi sesión ya abierta» |
| RN-GEN-020 | HU-ADM-011: «La API rechaza por permiso aunque el SPA no lo impida»; «Sin roles o sin empresa en el token»; HU-ADM-010: «El cajero ve solo lo que su rol puede leer»; HU-ADM-030: «Sin la acción Gestionar permisos» |
| RN-GEN-021 | HU-ADM-011: «El administrador pasa todas las comprobaciones»; HU-ADM-010: «El administrador ve todo el menú»; HU-ADM-030: «El rol Admin lo puede todo sin casillas» |
| RN-GEN-022 | HU-ADM-030: «Guardar los permisos»; HU-ADM-032: «Un rol nuevo con el mismo nombre no hereda permisos»; HU-ADM-010: «Quitar un permiso al rol cambia el menú»; HU-ADM-011: sin escenario propio |
| RN-GEN-023 | HU-ADM-022: «El administrador accede a todas las sucursales»; «Otro usuario accede…»; «Una asignación obsoleta o ajena no da acceso»; «Sin autenticación»; HU-ADM-038: «El efecto es inmediato, sin renovar el token»; HU-ADM-031: «Asignar sucursales» |
| RN-GEN-024 | HU-ADM-018: «El token no trae empresa»; HU-ADM-021: «El token no trae empresa al guardar la carga»; HU-ADM-050: «Contexto del usuario faltante» |
| RN-GEN-025 | HU-ADM-049: «Límite general de peticiones» |
| RN-GEN-026 | HU-ADM-002: «Demasiados intentos desde el mismo equipo»; HU-ADM-006: «Error del servidor o exceso de intentos»; HU-ADM-008: «Demasiadas solicitudes»; HU-ADM-009: «Demasiados envíos»; HU-ADM-049: «Límite de credenciales» |
| RN-GEN-027 | HU-ADM-001: «Cualquier otro rechazo del servidor o exceso de envíos»; HU-ADM-049: «Límite del registro de empresas» |
| RN-GEN-028 | No aplica a esta parte: la política de IA se aplica en `RecetasController` y la documenta la parte de recetas |
| RN-GEN-029 | HU-ADM-049: «IP real del cliente» |
| RN-GEN-030 | HU-ADM-049: «Cabeceras de seguridad» |
| RN-GEN-031 | HU-ADM-049: «HTTPS» |
| RN-GEN-032 | HU-ADM-049: «Orígenes permitidos» |
| RN-GEN-033 | HU-ADM-049: «Configuración mínima del token» |
| RN-GEN-034 | HU-ADM-049: «Estado del servicio» |
| RN-GEN-035 | HU-ADM-050: «Conflicto de concurrencia» |
| RN-GEN-036 | HU-ADM-050: «Valor único repetido»; HU-ADM-018: «Dos altas simultáneas con la misma licencia»; HU-ADM-019: «Errores de base de datos al guardar»; HU-ADM-024: «Alta simultánea con el mismo valor único»; HU-ADM-025: «Edición simultánea con un valor único repetido»; HU-ADM-028: «Alta simultánea con el mismo nombre»; HU-ADM-035: «Alta simultánea del mismo usuario» |
| RN-GEN-037 | HU-ADM-050: «Valor fuera de rango»; HU-ADM-018: «Valor fuera de rango o referencia inexistente»; HU-ADM-019: «Errores de base de datos al guardar» |
| RN-GEN-038 | HU-ADM-050: «Referencia inexistente o en uso»; HU-ADM-018: «Valor fuera de rango o referencia inexistente»; HU-ADM-019: «Errores de base de datos al guardar» |
| RN-GEN-039 | HU-ADM-046: «Archivo de más de 5 MB»; HU-ADM-050: «Cuerpo demasiado grande» |
| RN-GEN-040 | HU-ADM-050: «Error inesperado»; HU-ADM-002: «El servidor no responde» |
| RN-GEN-041 | HU-ADM-050: «Cuerpo con forma inválida»; HU-ADM-029: «El nivel es requerido en el cuerpo» |
| RN-AUT-001 | HU-ADM-002: «Usuario y contraseña obligatorios»; «Usuario de más de 50 caracteres» |
| RN-AUT-002 | HU-ADM-002: «Usuario y contraseña obligatorios» |
| RN-AUT-003 | HU-ADM-002: «El sistema no revela por qué falló el ingreso»; «No se pide la empresa» |
| RN-AUT-004 | HU-ADM-002: «El sistema no revela por qué falló el ingreso» |
| RN-AUT-005 | HU-ADM-003: «Una cuenta bloqueada no entra ni con la contraseña correcta» |
| RN-AUT-006 | HU-ADM-003: «Cada contraseña errónea suma un intento» |
| RN-AUT-007 | HU-ADM-003: «Cinco intentos fallidos bloquean la cuenta 15 minutos»; «El bloqueo vencido reinicia el contador» |
| RN-AUT-008 | HU-ADM-002: «El sistema no revela por qué falló el ingreso» |
| RN-AUT-009 | HU-ADM-002: «La empresa no tiene suscripción vigente»; «La vigencia solo se comprueba con la contraseña correcta»; HU-ADM-016: «No se puede iniciar sesión» |
| RN-AUT-010 | HU-ADM-002: «Inicio de sesión correcto»; «Cuenta que debe cambiar la contraseña»; HU-ADM-003: «Un ingreso correcto reinicia el contador»; HU-ADM-007: «Primer ingreso de una cuenta creada por un administrador» |
| RN-AUT-011 | HU-ADM-002: «El token y el token de refresco» |
| RN-AUT-012 | HU-ADM-002: «El token y el token de refresco» |
| RN-AUT-013 | HU-ADM-004: «Renovar exige la cookie y la cabecera de seguridad»; HU-ADM-005: «El cierre exige la cabecera de seguridad» |
| RN-AUT-014 | HU-ADM-004: «Renovar exige la cookie y la cabecera de seguridad» |
| RN-AUT-015 | HU-ADM-004: «No se puede renovar la sesión» |
| RN-AUT-016 | HU-ADM-004: «Detección de copia de la cookie» |
| RN-AUT-017 | HU-ADM-004: «Al renovar se vuelven a comprobar cuenta y empresa»; HU-ADM-016: «La renovación se rechaza sin revocar la sesión» |
| RN-AUT-018 | HU-ADM-004: «Cada renovación rota el token de refresco» |
| RN-AUT-019 | HU-ADM-004: «Dos pestañas renuevan a la vez con el mismo token» |
| RN-AUT-020 | HU-ADM-005: «El cierre es anónimo e idempotente»; «El cierre funciona aunque la llamada falle» |
| RN-AUT-021 | HU-ADM-005: «Se cierran todas las sesiones de la cuenta» |
| RN-AUT-022 | HU-ADM-005: «Renovación simultánea durante el cierre» |
| RN-AUT-023 | HU-ADM-004: «Consultar mi perfil»; HU-ADM-010: «El perfil devuelve roles y permisos distintos»; HU-ADM-007: sin escenario propio |
| RN-AUT-024 | HU-ADM-006: «Solo se cambia la contraseña de la cuenta del token» |
| RN-AUT-025 | HU-ADM-006: «Validaciones del formulario» |
| RN-AUT-026 | HU-ADM-006: «Validaciones del formulario» |
| RN-AUT-027 | HU-ADM-006: «Validaciones del formulario» |
| RN-AUT-028 | HU-ADM-006: «Cuenta desactivada o eliminada durante el cambio» |
| RN-AUT-029 | HU-ADM-006: «Contraseña actual incorrecta» |
| RN-AUT-030 | HU-ADM-006: «Efectos del cambio sobre la cuenta»; «Cambiar la contraseña»; HU-ADM-007: «Completar el cambio obligatorio» |
| RN-AUT-031 | HU-ADM-002: «Contraseñas guardadas de forma segura»; HU-ADM-006: «Efectos del cambio sobre la cuenta»; HU-ADM-035: «Crear la cuenta del cajero» |
| RN-AUT-032 | HU-ADM-008: «Usuario vacío o demasiado largo» |
| RN-AUT-033 | HU-ADM-008: «La respuesta es siempre la misma»; «Pedir el enlace» |
| RN-AUT-034 | HU-ADM-008: «Cuándo se genera de verdad el enlace»; HU-ADM-016: «No se envía el correo de recuperación» |
| RN-AUT-035 | HU-ADM-008: «Cuándo se genera de verdad el enlace»; «Solo sirve el último enlace» |
| RN-AUT-036 | HU-ADM-008: «El correo que recibo»; «Falla el envío del correo» |
| RN-AUT-037 | HU-ADM-009: «Validaciones del formulario»; «Enlace incompleto» |
| RN-AUT-038 | HU-ADM-009: «El enlace ya se usó, venció o fue reemplazado» |
| RN-AUT-039 | HU-ADM-009: «Efectos del restablecimiento»; HU-ADM-003: «Formas de desbloquear una cuenta» |
| RN-ROL-001 | HU-ADM-011: «La API rechaza por permiso aunque el SPA no lo impida»; HU-ADM-030: «Sin la acción Gestionar permisos»; HU-ADM-037: «Sin el permiso de asignar roles»; HU-ADM-027: «Buscar roles»; HU-ADM-028, HU-ADM-031 y HU-ADM-033: sin escenario propio |
| RN-ROL-002 | HU-ADM-028: «Campos obligatorios»; «Nombre de 51 a 100 caracteres» |
| RN-ROL-003 | HU-ADM-028: «La descripción es opcional»; HU-ADM-029: sin escenario propio |
| RN-ROL-004 | HU-ADM-028: «El nivel mínimo en la pantalla es 1»; HU-ADM-029: «El nivel es requerido en el cuerpo» |
| RN-ROL-005 | HU-ADM-028: «El nombre no puede repetirse en la empresa» |
| RN-ROL-006 | HU-ADM-029: «El nombre de un rol no se cambia» |
| RN-ROL-007 | HU-ADM-029: «Rol inexistente o de otra empresa»; HU-ADM-032: «Rol inexistente»; HU-ADM-037: «Rol o cuenta inexistentes» |
| RN-ROL-008 | HU-ADM-032: «El rol tiene cuentas» |
| RN-ROL-009 | HU-ADM-032: «Eliminar un rol sin cuentas»; HU-ADM-028: «Crear el rol Cajero»; HU-ADM-029: «Guardar» |
| RN-ROL-010 | HU-ADM-030: «No ampliar los permisos de mi propio rol» |
| RN-ROL-011 | HU-ADM-030: «Un permiso no se asigna dos veces» |
| RN-ROL-012 | HU-ADM-030: «Un permiso revocado se restaura en lugar de duplicarse» |
| RN-ROL-013 | HU-ADM-030: «La API no contrasta el permiso con el catálogo» |
| RN-ROL-014 | HU-ADM-030: «Revocar un permiso inexistente»; «Quitar un permiso» |
| RN-ROL-015 | HU-ADM-030: «Guardar los permisos»; HU-ADM-010: «Quitar un permiso al rol cambia el menú» |
| RN-ROL-016 | HU-ADM-030: «El catálogo sale de la API» |
| RN-ROL-017 | HU-ADM-031: «Guardar reemplaza el conjunto completo»; «Auditoría»; «Sucursal inexistente o de otra empresa» |
| RN-ROL-018 | HU-ADM-031: «Ver las sucursales del rol» |
| RN-ROL-019 | HU-ADM-027: «Buscar roles»; «Filtros de texto»; «Filtro de nivel con deslizador» |
| RN-ROL-020 | HU-ADM-033: «Al guardar se vuelve a validar cada fila» |
| RN-ROL-021 | HU-ADM-033: «Al guardar se omiten los nombres ya existentes» |
| RN-ROL-022 | HU-ADM-033: «Errores por fila en la vista previa»; «La vista previa exige nivel 1 y el guardado acepta 0»; «Plantilla de roles» |
| RN-ROL-023 | HU-ADM-027: «Exportar» |
| RN-CTA-001 | HU-ADM-011: «Los botones dependen de la acción»; HU-ADM-036: «Sin el permiso no hay botón»; HU-ADM-037: «Sin el permiso de asignar roles»; HU-ADM-039: «Sin el permiso no hay botón»; HU-ADM-034, HU-ADM-035 y HU-ADM-038: sin escenario propio |
| RN-CTA-002 | HU-ADM-035: «Campos obligatorios» |
| RN-CTA-003 | HU-ADM-035: «Usuario demasiado largo»; «Campos obligatorios» |
| RN-CTA-004 | HU-ADM-035: «Contraseña de menos de 8 caracteres» |
| RN-CTA-005 | HU-ADM-035: «Empleado inexistente» |
| RN-CTA-006 | HU-ADM-035: «El empleado ya tiene cuenta» |
| RN-CTA-007 | HU-ADM-035: «El usuario ya existe en el sistema»; HU-ADM-001: «El usuario ya existe en cualquier empresa» |
| RN-CTA-008 | HU-ADM-035: «Crear la cuenta del cajero»; HU-ADM-007: «Primer ingreso de una cuenta creada por un administrador» |
| RN-CTA-009 | HU-ADM-036: «Cuenta inexistente»; HU-ADM-039: sin escenario propio |
| RN-CTA-010 | HU-ADM-036: «Solo un administrador modifica la cuenta de un administrador»; HU-ADM-039: «Solo un administrador elimina la cuenta de un administrador» |
| RN-CTA-011 | HU-ADM-036: «Desactivar una cuenta»; «Forzar el cambio de contraseña»; «Desbloquear una cuenta»; «Usuario y empleado no se pueden cambiar»; HU-ADM-007: «El administrador fuerza el cambio con mi sesión ya abierta» |
| RN-CTA-012 | HU-ADM-036: «Desactivar una cuenta»; HU-ADM-002: «El sistema no revela por qué falló el ingreso» |
| RN-CTA-013 | HU-ADM-039: «Eliminar la cuenta»; «El empleado no se elimina» |
| RN-CTA-014 | HU-ADM-037: «Rol o cuenta inexistentes»; HU-ADM-038: «Sucursal o cuenta inexistente» |
| RN-CTA-015 | HU-ADM-037: «Nadie cambia sus propios roles» |
| RN-CTA-016 | HU-ADM-037: «El rol Admin está reservado» |
| RN-CTA-017 | HU-ADM-037: «Asignar un rol que ya se tiene o quitar uno que no se tiene»; «Rol o cuenta inexistentes» |
| RN-CTA-018 | HU-ADM-037: «Efecto sobre la sesión»; HU-ADM-004: «Un cambio de rol invalida el token actual» |
| RN-CTA-019 | HU-ADM-038: «Agregar una sucursal»; «Quitar una sucursal»; «Repetir la operación no cambia nada»; «Sucursal o cuenta inexistente» |
| RN-CTA-020 | HU-ADM-034: «Buscar sin filtros»; «Filtros de texto»; «Filtros de estado y bloqueo» |
| RN-EMP-001 | HU-ADM-015: «Sin el permiso no hay botón»; HU-ADM-001: «Cualquier otro rechazo del servidor o exceso de envíos»; HU-ADM-011: «Los botones dependen de la acción»; HU-ADM-013: sin escenario propio |
| RN-EMP-002 | HU-ADM-001: «Validaciones de cada campo»; HU-ADM-015: «Nombre obligatorio»; «Longitud del nombre» |
| RN-EMP-003 | HU-ADM-001: «Validaciones de cada campo» |
| RN-EMP-004 | HU-ADM-015: «RUC con formato inválido»; «RUC vacío»; «Solo se editan nombre y RUC» |
| RN-EMP-005 | HU-ADM-001: «El RUC ya está registrado por otra empresa»; HU-ADM-015: «RUC de otra empresa» |
| RN-EMP-006 | HU-ADM-001: «Validaciones de cada campo»; «El usuario ya existe en cualquier empresa» |
| RN-EMP-007 | HU-ADM-001: «Validaciones de cada campo»; «El apellido materno es opcional y la contraseña solo exige 8 caracteres» |
| RN-EMP-008 | HU-ADM-001: «Validaciones de cada campo»; «El apellido materno es opcional y la contraseña solo exige 8 caracteres» |
| RN-EMP-009 | HU-ADM-001: «Validaciones de cada campo» |
| RN-EMP-010 | HU-ADM-001: «La dirección de la sede principal es obligatoria»; «La API rechaza la dirección de la sede»; «El registro crea todo lo necesario en una sola operación»; HU-ADM-019: «La sede principal creada por el registro nace sin licencia» |
| RN-EMP-011 | HU-ADM-001: «El registro crea todo lo necesario en una sola operación»; «Si algo falla al registrar, no se crea nada» |
| RN-EMP-012 | HU-ADM-001: «El registro crea todo lo necesario en una sola operación»; HU-ADM-014: «Una empresa nueva nace con 30 días de prueba» |
| RN-EMP-013 | HU-ADM-001: «Registrar la empresa con datos válidos» |
| RN-EMP-014 | HU-ADM-016: «Cuándo una empresa está vigente»; «Cuándo no está vigente»; «En estado Activo la fecha de vencimiento no se evalúa»; HU-ADM-014: sin escenario propio |
| RN-EMP-015 | Pendiente: HU-ADM-016 (Parcial) documenta que ningún endpoint ni pantalla cambia el estado; sin escenario porque no hay comportamiento que verificar |
| RN-EMP-016 | Pendiente: HU-ADM-014 (Parcial) documenta que la política solo se cambia en la base de datos; sin pantalla ni endpoint, sin escenario |
| RN-EMP-017 | HU-ADM-014: «Solo veo mi propia empresa»; HU-ADM-015: «Otra empresa no se puede editar»; HU-ADM-013: «Buscar sin filtros»; HU-ADM-045: «Un registro de otra empresa se comporta como inexistente» |
| RN-EMP-018 | HU-ADM-013: «Buscar sin filtros»; «Filtrar por estado»; «Filtrar por vencimiento»; «Exportar a Excel» |
| RN-SUC-001 | HU-ADM-011: «Los botones dependen de la acción»; HU-ADM-021: «Sin permiso para crear»; HU-ADM-022: «El administrador accede a todas las sucursales»; HU-ADM-017: «Buscar sin filtros»; HU-ADM-018: «Crear la sucursal» |
| RN-SUC-002 | HU-ADM-018: «Campos obligatorios»; «Longitudes máximas»; HU-ADM-019: sin escenario propio |
| RN-SUC-003 | HU-ADM-018: «Campos obligatorios»; «Longitudes máximas»; HU-ADM-019: sin escenario propio |
| RN-SUC-004 | HU-ADM-018: «Campos obligatorios»; «Longitudes máximas»; HU-ADM-019: «La sede principal creada por el registro nace sin licencia»; «Completar la licencia de la sede principal» |
| RN-SUC-005 | HU-ADM-018: «Licencia repetida en la empresa»; «Dos altas simultáneas con la misma licencia»; HU-ADM-019: «Licencia usada por otra sucursal» |
| RN-SUC-006 | HU-ADM-018: «Elegir un gerente»; «El gerente debe ser un empleado de la empresa»; HU-ADM-019: «Cambiar o quitar el gerente» |
| RN-SUC-007 | HU-ADM-018: «Crear la sucursal» |
| RN-SUC-008 | HU-ADM-019: «Errores genéricos»; HU-ADM-020: «Otros errores al eliminar» |
| RN-SUC-009 | HU-ADM-020: «La sucursal tiene registros asociados»; «Eliminar una sucursal sin registros asociados» |
| RN-SUC-010 | HU-ADM-022: «El administrador accede a todas las sucursales»; «Otro usuario accede…» |
| RN-SUC-011 | HU-ADM-017: «Buscar sin filtros»; «Filtros que se combinan»; «Exportar» |
| RN-SUC-012 | HU-ADM-021: «Cargar un archivo con todas las filas válidas»; «Al guardar se omiten licencias que ya existen»; «Al guardar se vuelve a validar cada fila» |
| RN-SUC-013 | HU-ADM-021: «Errores por fila en la vista previa»; «Una licencia repetida marca todas las filas repetidas»; «Descargar la plantilla» |
| RN-EPL-001 | HU-ADM-023: «Vista de solo consulta para un cajero»; HU-ADM-011: «Los botones dependen de la acción»; HU-ADM-024: sin escenario propio |
| RN-EPL-002 | HU-ADM-024: «Campos obligatorios»; «La sucursal base se elige de la lista»; HU-ADM-025: sin escenario propio |
| RN-EPL-003 | HU-ADM-024: «La sucursal base debe existir en la empresa»; HU-ADM-025: sin escenario propio |
| RN-EPL-004 | HU-ADM-024: «Campos obligatorios»; «Longitudes máximas» |
| RN-EPL-005 | HU-ADM-024: «Campos obligatorios»; «Longitudes máximas» |
| RN-EPL-006 | HU-ADM-024: «El apellido materno es opcional» |
| RN-EPL-007 | HU-ADM-024: «Longitudes máximas»; «La licencia profesional es opcional y única» |
| RN-EPL-008 | HU-ADM-024: «La licencia profesional es opcional y única»; HU-ADM-025: «Las reglas de licencia y correo se aplican al editar» |
| RN-EPL-009 | HU-ADM-024: «El correo es opcional, con formato válido y único en la empresa»; «Correo sin punto tras la arroba»; «Normalización del correo»; HU-ADM-008: «Un cajero recupera su contraseña con el correo de su ficha de empleado» |
| RN-EPL-010 | HU-ADM-024: «Correo repetido en la empresa»; HU-ADM-025: «Las reglas de licencia y correo se aplican al editar» |
| RN-EPL-011 | HU-ADM-025: «Nadie cambia su propia sucursal base» |
| RN-EPL-012 | HU-ADM-025: «Cambiar la sucursal base cierra la sesión del empleado» |
| RN-EPL-013 | HU-ADM-025: «Empleado inexistente o de otra empresa»; HU-ADM-026: sin escenario propio |
| RN-EPL-014 | HU-ADM-026: «El empleado es gerente de una sucursal»; «Quitar el gerente para poder eliminar» |
| RN-EPL-015 | HU-ADM-026: «Eliminar un empleado que tiene cuenta de usuario» |
| RN-EPL-016 | HU-ADM-023: «Buscar sin filtros» |
| RN-EPL-017 | HU-ADM-023: «Buscar sin filtros»; «Empleados de una sucursal eliminada no aparecen»; «Filtro de sucursal inválido en la API»; «Exportar» |
| RN-EPL-018 | HU-ADM-024: «Registrar un empleado»; HU-ADM-025: «Editar reemplaza todos los datos» |
| RN-SER-001 | HU-ADM-011: «Los botones dependen de la acción»; HU-ADM-040: «Buscar sin filtros»; HU-ADM-041: sin escenario propio |
| RN-SER-002 | HU-ADM-041: «Campos obligatorios»; «Sucursal inexistente»; HU-ADM-042: «Serie o sucursal inexistente» |
| RN-SER-003 | HU-ADM-041: «Tipos disponibles» |
| RN-SER-004 | HU-ADM-041: «El prefijo debe tener 4 caracteres y la letra del tipo»; «El prefijo se reevalúa al cambiar el tipo»; «Prefijo rechazado por el servidor»; HU-ADM-042: «Prefijo repetido al editar» |
| RN-SER-005 | HU-ADM-041: «Correlativo»; HU-ADM-044: «El siguiente número es el correlativo más uno»; «Continuar una numeración anterior» |
| RN-SER-006 | HU-ADM-041: «Estado» |
| RN-SER-007 | HU-ADM-041: «El prefijo ya existe para el mismo tipo en la empresa»; HU-ADM-042: «Prefijo repetido al editar» |
| RN-SER-008 | HU-ADM-041: «Solo una serie activa por sucursal y tipo»; HU-ADM-042: «Activar otra serie del mismo tipo» |
| RN-SER-009 | HU-ADM-042: «Serie o sucursal inexistente»; HU-ADM-043: «Serie inexistente» |
| RN-SER-010 | HU-ADM-042: «Serie que ya emitió comprobantes»; «La API protege la numeración ya emitida» |
| RN-SER-011 | HU-ADM-043: «Serie con comprobantes emitidos» |
| RN-SER-012 | HU-ADM-043: «Eliminar una serie sin comprobantes»; HU-ADM-041: «Crear la serie de boletas»; HU-ADM-042: «Ver y editar una serie sin comprobantes» |
| RN-SER-013 | HU-ADM-044: «Dos emisiones simultáneas» |
| RN-SER-014 | HU-ADM-040: «Buscar sin filtros»; «Las series de una sucursal eliminada no aparecen»; «Exportar» |
| RN-IMP-001 | HU-ADM-046: «Archivo que no es .xlsx» |
| RN-IMP-002 | HU-ADM-046: «Archivo renombrado o dañado» |
| RN-IMP-003 | HU-ADM-046: «Archivo comprimido de forma sospechosa» |
| RN-IMP-004 | HU-ADM-046: «Archivo de más de 5 MB» |
| RN-IMP-005 | HU-ADM-046: «Más de 5000 filas» |
| RN-IMP-006 | HU-ADM-046: «Más de 5000 filas»; HU-ADM-021 y HU-ADM-033: «Al guardar se vuelve a validar cada fila» |
| RN-IMP-007 | HU-ADM-046: «Al guardar cada fila se valida de nuevo»; HU-ADM-021 y HU-ADM-033: «Al guardar se vuelve a validar cada fila» |
| RN-IMP-008 | HU-ADM-047: «Más de 10 000 filas»; HU-ADM-013: «Exportación demasiado grande»; HU-ADM-017: «Exportación de más de 10 000 filas»; HU-ADM-023, HU-ADM-027 y HU-ADM-040: sin escenario propio |
| RN-IMP-009 | HU-ADM-047: «Títulos de columna inválidos»; HU-ADM-013, HU-ADM-017, HU-ADM-023, HU-ADM-027 y HU-ADM-040: sin escenario propio |
| RN-IMP-010 | HU-ADM-047: «Etiquetas de celda demasiado largas»; HU-ADM-013: sin escenario propio |
| RN-IMP-011 | HU-ADM-047: «Formato de fecha»; HU-ADM-040: «Formato de fecha inválido en la exportación» |
| RN-IMP-012 | HU-ADM-046: «Cómo se lee el archivo»; HU-ADM-021: sin escenario propio |
| RN-IMP-013 | HU-ADM-046: «Vista previa sin archivo adjunto»; HU-ADM-033: «Vista previa sin archivo» |
| RN-IMP-014 | HU-ADM-046: «La vista previa»; HU-ADM-021: «Cargar un archivo con todas las filas válidas»; «Solo se guardan las filas válidas»; HU-ADM-033: «Una vista previa mixta» |
| RN-IMP-015 | HU-ADM-046: «Resolver nombres de otros catálogos» (lo usan módulos de otras partes); HU-ADM-021: sin escenario propio |
| RN-IMP-016 | HU-ADM-046: «La plantilla»; HU-ADM-021: «Descargar la plantilla»; HU-ADM-033: «Plantilla de roles» |
| RN-IMP-017 | HU-ADM-047: «Exportar con los filtros escritos»; HU-ADM-013: «Exportar a Excel»; HU-ADM-017: «Exportar»; HU-ADM-023: «Exportar»; HU-ADM-027: «Exportar»; HU-ADM-040: «Exportar» |

### 14.2 Rutas del SPA del alcance (24)

| Ruta | Historia(s) |
|---|---|
| `/auth/login` | HU-ADM-002, HU-ADM-003, HU-ADM-016 |
| `/auth/register` | HU-ADM-001 |
| `/auth/forgot-password` | HU-ADM-008 |
| `/auth/reset-password` | HU-ADM-009 |
| `/cambiar-clave` | HU-ADM-006, HU-ADM-007 |
| `/empresas` | HU-ADM-013 |
| `/empresas/:id` | HU-ADM-014, HU-ADM-015 |
| `/sucursales` | HU-ADM-017 |
| `/sucursales/nueva` | HU-ADM-018 |
| `/sucursales/carga-masiva` | HU-ADM-021, HU-ADM-046 |
| `/sucursales/:id` | HU-ADM-019, HU-ADM-020 |
| `/empleados` | HU-ADM-023 |
| `/empleados/nueva` | HU-ADM-024 |
| `/empleados/:id` | HU-ADM-025, HU-ADM-026 |
| `/roles` | HU-ADM-027 |
| `/roles/nueva` | HU-ADM-028 |
| `/roles/carga-masiva` | HU-ADM-033, HU-ADM-046 |
| `/roles/:id` | HU-ADM-029, HU-ADM-030, HU-ADM-031, HU-ADM-032 |
| `/cuentas` | HU-ADM-034 |
| `/cuentas/nueva` | HU-ADM-035 |
| `/cuentas/:id` | HU-ADM-036, HU-ADM-037, HU-ADM-038, HU-ADM-039 |
| `/series-fiscales` | HU-ADM-040 |
| `/series-fiscales/nueva` | HU-ADM-041 |
| `/series-fiscales/:id` | HU-ADM-042, HU-ADM-043 |

Fuera de la lista: `/dashboard` pertenece a la parte C, y las redirecciones de `''` y `**` hacia `dashboard` (`sofia-web-spa/src/app/app.routes.ts:9,177-180`) no tienen escenario propio. El menú, la cabecera y el guard de permisos (sin ruta propia) están en HU-ADM-010, HU-ADM-011 y HU-ADM-012.

### 14.3 Endpoints del alcance (60)

Los 60 endpoints de `Auth`, `Cuentas`, `Empleados`, `Empresas`, `Roles`, `SeriesFiscales` y `Sucursales` de [`endpoints.csv`](../../_inventario/endpoints.csv) aparecen en la fila «Endpoint(s)» de alguna historia (tabla de la sección 12). Los únicos sin pantalla son `POST /api/v1/Roles/assign-to-user` y `POST /api/v1/Roles/remove-from-user` (HU-ADM-037), que duplican a los de `Cuentas` que usa la pantalla.

### 14.4 Verificación

Se ejecutó un script de lectura (Node, en el directorio temporal de la sesión, sin modificar ningún repositorio) que:

1. Extrajo los 203 IDs `RN-…` definidos en el catálogo (filas de tabla que empiezan por `| RN-`) y comprobó que todos los IDs citados en este documento existen (ninguno inexistente).
2. Comprobó que 202 de los 203 IDs están en la fila «Reglas» de alguna historia; el que falta es `RN-GEN-028`, que se aplica a recetas y no a esta parte (explicado en la tabla 14.1).
3. Comprobó que cada escenario nombrado en la tabla 14.1 existe en la historia indicada.
4. Comprobó que los 60 endpoints y las 24 rutas del alcance aparecen en el documento.

Resultado: 50 historias, 203 reglas (202 cubiertas más 1 que no aplica), 24 rutas y 60 endpoints, sin reglas, rutas ni endpoints sin cubrir. Las reglas `RN-EMP-015` y `RN-EMP-016` quedan cubiertas por la declaración de estado «Parcial» de su historia y no por un escenario, porque el sistema no tiene ese comportamiento.

---

## 15. Fuentes

Documentos (todos bajo `sofia-docs/`):

- `_inventario/REGLAS-AGENTES.md`, `_inventario/README.md`, `_inventario/endpoints.csv`, `_inventario/rutas-spa.csv`.
- `ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md` (versión 1.1): reglas `RN-GEN`, `RN-AUT`, `RN-ROL`, `RN-CTA`, `RN-EMP`, `RN-SUC`, `RN-EPL`, `RN-SER` e `RN-IMP`.
- `ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md` y `19b-administracion.md` (versión 1.1).
- `ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md` (apartado 10 sobre auditoría) y `05b-matriz-de-permisos.md`.
- `ola-1-fundamentos/06-glosario.md` (términos preferidos: sucursal, sede principal, rol Admin, empresa, serie y prefijo, correlativo).
- `ola-2-funcional/manual-usuario/capitulo-01-primeros-pasos.md`, `capitulo-02-empresa-sucursales-series.md` y `capitulo-03-empleados-roles-cuentas.md`.
- `ola-2-funcional/14-procesos-de-negocio.md` (procesos 1 y 2).

Código consultado directamente:

- `sofia-web-spa/src/app/app.routes.ts` (rutas y guards del layout).
- Búsqueda de `nombreRolHint` en `sofia-web-spa/src/app` (sin referencias).
- Búsqueda de `getPermitidas` en `sofia-web-spa/src/app` (solo las pantallas de Stock por Sucursal: `stock-search.component.ts` y `stock-detail.component.ts`).

No se usaron el navegador, la API local, `README.md`/`CLAUDE.md` de los repositorios ni `swagger.json`.
