# Casos de prueba — Parte A: acceso y administración

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-30 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de documentos validados y comprobación puntual en el código; casos no ejecutados |

## 1. Convenciones

Este documento reúne los casos de prueba funcionales de la **parte A** (módulo `ADM`): registro de la empresa, acceso y sesión, menú y permisos de pantalla, empresa y suscripción, sucursales, empleados, roles y permisos, cuentas de usuario, series fiscales y reglas transversales de plataforma. Las partes B y C (catálogos, inventario y atención; punto de venta y caja) están en documentos aparte de esta misma carpeta.

**Estado de ejecución.** Todos los casos están en «No ejecutado»: el documento se escribió desde las historias de usuario ([12a][hu]), los casos de uso ([13a][cu]), las reglas ([01a][rn]), los inventarios de pantallas ([19a][p19a], [19b][p19b]) y el manual de usuario (capítulos [1][m1], [2][m2] y [3][m3]), sin ejecutar la aplicación.

**Tipos**

| Tipo | Qué comprueba |
|---|---|
| Positivo | El flujo principal con datos válidos |
| Negativo | Una validación o regla de negocio que debe rechazar la operación |
| Permisos | Qué se ve o qué responde la API cuando falta un permiso o no hay sesión |
| Límite | Valores en el borde (longitud máxima, mínimo, un carácter de más) |
| Concurrencia | Dos acciones simultáneas sobre el mismo dato |
| Integración | Intervención de un sistema externo (en esta parte, el correo) |

**Prioridades**

| Prioridad | Criterio |
|---|---|
| Alta | Acceso y sesión, contraseñas, permisos, aislamiento entre empresas o sucursales, suscripción, numeración de comprobantes y cualquier operación que afecte dinero, stock o comprobantes |
| Media | Resto del flujo principal de cada pantalla: altas, ediciones, bajas, búsquedas, cargas masivas y exportaciones |
| Baja | Presentación, ordenación, textos de ayuda, navegación y migas |

**Cómo leer un caso**

- Cada caso tiene un ID `TC-ADM-nnn`, una tabla con **Tipo**, **Prioridad** y **Cubre**, y los bloques **Precondiciones**, **Datos**, **Pasos** y **Resultado esperado**.
- **Cubre** cita el escenario de la historia (`HU-ADM-nnn` con el título del escenario entre comillas), el caso de uso (`CU-ADM-nn`; «flujo básico», alternativo `An` o «excepción paso N» según las tablas de excepciones del caso de uso) y las reglas `RN-…`. Los enlaces usan las definiciones del final del documento.
- Los textos entre « » son los que ve el usuario, copiados de los documentos fuente (que los tomaron de los `es.json`). Donde el resultado esperado dice «genérico» es el mensaje genérico de la operación, con su texto.
- Un «Esquema del escenario» con tabla de ejemplos se cubre con **un solo caso** que lleva la misma tabla de datos y resultados; el tester recorre todas las filas.
- Los casos «Solo API» llevan la petición (método, ruta, cuerpo) y la respuesta esperada (HTTP y `code`). El token se escribe `<token>`. Las rutas llevan el prefijo `/api/v1/`.
- **Hallazgo:** cuando el sistema se comporta distinto de lo que el negocio esperaría, el caso tiene como resultado esperado el comportamiento **actual** y lo marca con el hallazgo o la «Brecha conocida» del documento fuente. No se inventa el comportamiento deseado.
- **No verificado:** el comportamiento no pudo respaldarse en la aplicación ni en el código; el caso dice el motivo.
- Los casos que dependen del límite de peticiones (429) o de un fallo del servidor son difíciles de provocar en el entorno local; llevan «No verificado en la aplicación» como lo marcan las historias.

## 2. Entorno y datos de prueba

### 2.1 Entorno

Para levantar el entorno local (servicios Docker con SQL Server, Presidio y el servidor de correo de pruebas Mailpit; scripts DbUp; API y SPA) sigue la [guía del desarrollador, sección 3][g08] y el [documento de despliegue y operación][g10]; este documento no los repite. El SPA se prueba en un navegador contra la API local. El correo de recuperación de contraseña se lee en la bandeja web de Mailpit; si el entorno no tiene proveedor de correo configurado, el mensaje se descarta ([RN-AUT-036][rn]) y los casos de integración de correo no se pueden ejecutar.

Mientras no se indique otra cosa, los casos se ejecutan en el SPA con la sesión de la persona indicada en las precondiciones. Para los casos «Solo API» se usa un cliente HTTP (Postman, Insomnia o `curl`) contra la API local, con `Authorization: Bearer <token>`.

### 2.2 Empresas, usuarios y datos maestros

Todos los datos son ficticios y evidentes (dominio de correo `sofia-manual.test`). Nunca se escriben contraseñas reales: donde un caso dice «una contraseña válida de 8 caracteres o más», el tester usa una de su elección y la anota fuera del documento.

| Dato | Valor | Para qué |
|---|---|---|
| Empresa A | «Botica Manual Demo S.A.C.», RUC `20538291746`, sede «Sede Principal» con dirección «Av. Ficticia 123, Miraflores, Lima» | Empresa principal de casi todos los casos |
| Administradora de A | «Ana Demo Prueba», usuario `ana.manual`, correo `ana.demo@sofia-manual.test`, rol `Admin` | Registrada en el caso TC-ADM-001; el resto de casos la reutiliza |
| Sucursal adicional de A | «Sucursal Demo Norte» (dirección ficticia y licencia ficticia que el tester define) | Casos de sucursales, roles y cuentas |
| Cajero de A | Empleado «Carlos Cajero Demo», correo `carlos.cajero@sofia-manual.test`, sucursal base «Sede Principal», usuario `carlos.cajero` | Casos de permisos limitados |
| Segundo empleado | «Nora Norte Demo», sucursal base «Sucursal Demo Norte», usuario `nora.norte` | Casos de sucursales y de eliminación |
| Rol «Cajero» | Nivel 30, descripción «Cajero de mostrador: atención al público y punto de venta», con los 19 permisos recomendados del manual ([capítulo 3, sección 5.3][m3]): Ventas (Leer, Crear, Actualizar, Anular), Sesiones de Caja (Leer, Aperturar caja, Cerrar caja), Devoluciones (Procesar devolución, Leer), Inventario Leer, DIGEMID Leer, Medicamentos Leer, Pacientes Leer, Aseguradoras Leer, Recetas Médicas (Leer, Analizar con IA, Crear), Profesionales de Salud Leer, Empleados Leer | Casos de menú, rutas y permisos. Se crea en TC-ADM-216 y se completa en TC-ADM-238 |
| Empresa B | «Farmacia Prueba Dos S.A.C.», RUC `20123456789`, administrador `luis.dos`, sede «Sede Principal» | Casos de aislamiento entre empresas y de unicidad entre empresas |
| Empresa con suscripción no vigente | Una empresa de prueba cuyo estado sea suspendido, cancelado o con prueba vencida | Casos de HU-ADM-016. Ningún endpoint ni pantalla cambia el estado de la suscripción ([RN-EMP-015][rn]); hay que dejarla así por base de datos, y los casos lo marcan «No verificado en la aplicación» como la historia |
| Rol «Consulta Demo» y cuenta `consulta.demo` | Rol de solo lectura: `Leer` en Empresa, Sucursales, Empleados, Roles y Cuentas de Usuario y Series Fiscales, sin ninguna otra acción | Casos de permisos que necesitan una cuenta con `Leer` y sin `Crear`, `Actualizar` ni `Eliminar` |
| Rol «Sin Permisos Demo» y cuenta `sin.permisos` | Rol con todas las casillas desmarcadas | Casos de menú y Dashboard sin permisos |
| Rol «Rol Prueba Eliminar» | Rol descartable sin permisos | Casos de eliminación de roles y de cambio de rol |
| Gestor sin rol `Admin` | Cuenta con un rol «Gerente Demo» que tiene `Leer`, `Actualizar`, `Eliminar`, `Asignar roles` y `Gestionar permisos` de Roles y Cuentas de Usuario | Casos de protección del rol y de las cuentas `Admin` y de ampliación de permisos propios |
| Excel de carga masiva | Plantillas descargadas desde la propia pantalla de carga de sucursales y de roles | Casos de HU-ADM-021, HU-ADM-033 y HU-ADM-046 |

Antes de cada ronda conviene tener a mano: una cuenta de prueba **solo para bloqueos** (no la de `carlos.cajero` si se va a usar después) y dos navegadores o perfiles distintos para los casos de dos sesiones.

## 3. Suite de humo

Estos 26 casos, todos de prioridad Alta, se corren primero y en este orden, porque cada uno prepara datos para el siguiente (la empresa A, el rol «Cajero», la cuenta de Carlos y las series). Si alguno falla, detén la ronda y repórtalo antes de seguir con los demás módulos.

| Orden | Caso | Qué comprueba |
|---|---|---|
| 1 | TC-ADM-001 | Registrar la empresa y su administrador |
| 2 | TC-ADM-012 | Iniciar sesión con credenciales correctas |
| 3 | TC-ADM-018 | Contraseña incorrecta: mensaje uniforme |
| 4 | TC-ADM-135 | Crear la sucursal «Sucursal Demo Norte» |
| 5 | TC-ADM-182 | Registrar el empleado «Carlos Cajero Demo» |
| 6 | TC-ADM-216 | Crear el rol «Cajero» |
| 7 | TC-ADM-238 | Marcar los 19 permisos del cajero |
| 8 | TC-ADM-274 | Crear la cuenta `carlos.cajero` |
| 9 | TC-ADM-295 | Asignar el rol «Cajero» a la cuenta |
| 10 | TC-ADM-058 | Primer ingreso: cambio de contraseña obligatorio |
| 11 | TC-ADM-059 | El menú no deja salir del cambio obligatorio |
| 12 | TC-ADM-060 | Completar el cambio obligatorio |
| 13 | TC-ADM-083 | El cajero ve solo lo que su rol puede leer |
| 14 | TC-ADM-091 | Pantalla sin permiso por dirección |
| 15 | TC-ADM-328 | Crear la serie de boletas |
| 16 | TC-ADM-337 | Solo una serie activa por sucursal y tipo |
| 17 | TC-ADM-049 | Cambiar mi contraseña (cierra todas las sesiones) |
| 18 | TC-ADM-026 | Cinco intentos fallidos bloquean la cuenta |
| 19 | TC-ADM-289 | Desbloquear con «Resetear Intentos Fallidos» |
| 20 | TC-ADM-031 | Renovación automática de la sesión |
| 21 | TC-ADM-043 | Cerrar sesión |
| 22 | TC-ADM-065 | Pedir el enlace de recuperación (necesita Mailpit) |
| 23 | TC-ADM-075 | Restablecer la contraseña con el enlace |
| 24 | TC-ADM-112 | Solo veo mi propia empresa |
| 25 | TC-ADM-156 | No se elimina una sucursal con registros asociados |
| 26 | TC-ADM-357 | Un registro de otra empresa se comporta como inexistente |

Los casos de numeración de comprobantes (TC-ADM-354 a TC-ADM-356) también son de prioridad Alta, pero necesitan caja abierta y un producto con stock (parte C), así que se corren después de la suite de humo de esa parte.

## 4. Casos por módulo

## 4.1 Registro y acceso

### HU-ADM-001 — Registrar la empresa y su administrador

#### TC-ADM-001 — Registrar la empresa con datos válidos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-001][hu] «Registrar la empresa con datos válidos»; [CU-ADM-01][cu] flujo básico; [RN-EMP-002, RN-EMP-003, RN-EMP-005, RN-EMP-006, RN-EMP-007, RN-EMP-008, RN-EMP-009, RN-EMP-010, RN-EMP-013][rn] |

**Precondiciones:** sin sesión iniciada; el RUC `20538291746` y el usuario `ana.manual` no existen en ninguna empresa.
**Datos:** Nombre de la Empresa «Botica Manual Demo S.A.C.»; RUC `20538291746`; Dirección de la sede principal «Av. Ficticia 123, Miraflores, Lima»; Nombres del administrador «Ana»; Apellido paterno «Demo»; Apellido materno «Prueba»; Correo electrónico `ana.demo@sofia-manual.test`; Usuario Administrador `ana.manual`; Contraseña: una contraseña válida de 8 caracteres o más.
**Pasos:**
1. Abre `/auth/register` (formulario «Registrar Empresa»).
2. Escribe los datos y haz clic en «Registrar Empresa».
3. Observa el botón mientras el sistema trabaja.
**Resultado esperado:** el botón muestra «Procesando...»; aparece el aviso «Empresa registrada correctamente. ¡Bienvenido!»; entras directamente al Inicio (`/dashboard`) con la sesión iniciada, sin escribir otra vez usuario y contraseña y sin que llegue ningún correo de verificación.

#### TC-ADM-002 — El registro crea la empresa, la sede, el empleado, la cuenta y el rol Admin

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-001][hu] «El registro crea todo lo necesario en una sola operación»; [CU-ADM-01][cu] flujo básico (paso 6) y postcondiciones; [RN-EMP-011, RN-EMP-012, RN-EMP-010, RN-CTA-007, RN-GEN-004][rn] |

**Precondiciones:** TC-ADM-001 ejecutado con éxito.
**Datos:** los de TC-ADM-001.
**Pasos:**
1. En la sesión de Ana abre la pantalla «Empresa» y la ficha de la empresa (ver TC-ADM-109 para el detalle de la ficha).
2. Abre «Sucursales» y busca todas.
3. Abre «Empleados» y busca todos.
4. Abre «Cuentas de Usuario» y busca por «ana.manual».
5. Abre «Roles & Permisos» y busca todos.
6. Cierra sesión y vuelve a entrar con `ana.manual`.
**Resultado esperado:** la empresa aparece en estado «Trial Activo», vigente, con inicio de prueba en el momento del registro y vencimiento 30 días después; existe una sola sucursal «Sede Principal» con la dirección escrita y el «Nº de Licencia» vacío; existe el empleado «Ana Demo Prueba» con el correo escrito; existe la cuenta `ana.manual`; existe el rol «Admin» con descripción «Administrador de la empresa» y nivel 0 asignado a la cuenta; al volver a entrar no se te lleva a «Cambiar contraseña» (la cuenta no obliga a cambiar la contraseña). Con acceso a la base de datos: todas las filas nuevas llevan el mismo `TenantId`, el de la empresa nueva.

#### TC-ADM-003 — Dos registros simultáneos con el mismo RUC no dejan datos a medias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-ADM-001][hu] «Si algo falla al registrar, no se crea nada»; [CU-ADM-01][cu] excepción paso 6 (`Db.Duplicado`) y postcondición «Fallo»; [RN-GEN-012, RN-EMP-011][rn] |

**Precondiciones:** ninguna empresa con RUC `20111111112`. Cliente HTTP capaz de lanzar dos peticiones a la vez. No verificado en la aplicación: la interfaz no permite provocar un fallo a mitad de la operación; este caso usa la colisión del índice único como forma práctica de provocarlo.
**Datos:** dos cuerpos de `POST /empresas/registrar` con el mismo `ruc` `20111111112` y usuarios distintos (`dup.uno`, `dup.dos`); el resto de campos válidos (`nombreEmpresa`, `usuario`, `password`, `adminNombres`, `adminApellidoPaterno`, `adminEmail`, `direccionSede`).
**Pasos:**
1. Envía las dos peticiones al mismo tiempo.
2. Con la respuesta de cada una, busca en «Sucursales», «Empleados», «Cuentas de Usuario» y «Roles & Permisos» de cada empresa resultante (o consulta la base de datos).
**Resultado esperado:** una petición responde 201 y la otra 409 (`Empresa.RUC.Duplicado` o `Db.Duplicado`); solo existe una empresa con ese RUC; la petición rechazada no dejó empresa, sede, empleado, cuenta ni rol (se revierte todo).

#### TC-ADM-004 — Campos obligatorios vacíos en el registro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-001][hu] «Campos obligatorios vacíos»; [CU-ADM-01][cu] excepción paso 2; [RN-EMP-006, RN-EMP-007, RN-EMP-008, RN-EMP-009][rn] |

**Precondiciones:** sin sesión; estás en `/auth/register`.
**Datos:** formulario vacío.
**Pasos:**
1. Haz clic en «Registrar Empresa» sin escribir nada.
2. Observa los campos y la pestaña de red del navegador.
**Resultado esperado:** cada campo obligatorio muestra «Este campo es obligatorio»; el «Apellido materno (opcional)» no marca error; no se envía ninguna petición al servidor.

#### TC-ADM-005 — Validaciones de cada campo del registro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-001][hu] «Validaciones de cada campo» (esquema); [CU-ADM-01][cu] excepción paso 2; [RN-EMP-003, RN-EMP-006, RN-EMP-007, RN-EMP-008, RN-EMP-009, RN-EMP-010][rn] |

**Precondiciones:** sin sesión; estás en `/auth/register`.
**Datos:** una fila por vez; el tester escribe el valor en el campo y sale del campo.
**Pasos:**
1. Para cada fila de la tabla, escribe el valor en el campo y sal del campo (Tab).
2. Lee el mensaje bajo el campo.
**Resultado esperado:**

| Campo | Valor | Mensaje bajo el campo |
|---|---|---|
| Nombre de la Empresa | 201 caracteres | Máximo 200 caracteres |
| RUC | `2053829174` (10 dígitos) | El RUC debe tener exactamente 11 dígitos numéricos |
| RUC | una letra entre los dígitos | El RUC debe tener exactamente 11 dígitos numéricos |
| Dirección de la sede principal | solo espacios | Este campo es obligatorio |
| Dirección de la sede principal | 256 caracteres | Máximo 255 caracteres |
| Nombres del administrador | 76 caracteres | Máximo 75 caracteres |
| Apellido paterno | 76 caracteres | Máximo 75 caracteres |
| Apellido materno (opcional) | 76 caracteres | Máximo 75 caracteres |
| Correo electrónico | `ana.demo` | Ingresa un correo electrónico válido |
| Correo electrónico | 255 caracteres | Máximo 254 caracteres |
| Usuario Administrador | `ana` (3 caracteres) | El usuario debe tener al menos 4 caracteres |
| Usuario Administrador | 51 caracteres | Máximo 50 caracteres |
| Contraseña | `abc1234` (7 caracteres) | La contraseña debe tener al menos 8 caracteres |

#### TC-ADM-006 — Apellido materno opcional y contraseña de solo 8 caracteres

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-001][hu] «El apellido materno es opcional y la contraseña solo exige 8 caracteres»; [CU-ADM-01][cu] flujo básico (paso 2); [RN-EMP-007, RN-AUT-026][rn] |

**Precondiciones:** sin sesión; RUC y usuario libres (por ejemplo empresa de prueba `20222222223`, usuario `soloocho`).
**Datos:** todos los campos válidos, «Apellido materno (opcional)» vacío y una contraseña de exactamente 8 letras minúsculas sin números ni símbolos.
**Pasos:**
1. Completa el formulario con esos datos.
2. Haz clic en «Registrar Empresa».
**Resultado esperado:** el formulario no marca error en esos campos y el registro continúa hasta el Inicio; en «Empleados» el empleado administrador queda sin apellido materno.

#### TC-ADM-007 — La dirección de la sede principal es obligatoria y tiene ayuda

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-001][hu] «La dirección de la sede principal es obligatoria»; [CU-ADM-01][cu] excepción paso 2; [RN-EMP-010][rn] |

**Precondiciones:** sin sesión; estás en `/auth/register` con los demás campos válidos.
**Datos:** «Dirección de la sede principal» con solo espacios.
**Pasos:**
1. Escribe solo espacios en la dirección y haz clic en «Registrar Empresa».
2. Lee el texto bajo el campo.
**Resultado esperado:** se ve «Este campo es obligatorio» y no se envía nada; bajo el campo se lee la ayuda «Aparecerá en el encabezado de tus boletas».

#### TC-ADM-008 — RUC ya registrado por otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-001][hu] «El RUC ya está registrado por otra empresa»; [CU-ADM-01][cu] excepción paso 5; [RN-EMP-005][rn] |

**Precondiciones:** existe la empresa A (RUC `20538291746`).
**Datos:** datos válidos para una empresa nueva, pero con RUC `20538291746` y un usuario libre (`otra.empresa`).
**Pasos:**
1. En `/auth/register` completa el formulario con ese RUC.
2. Haz clic en «Registrar Empresa».
3. Revisa «Empresa» o la base de datos para comprobar que no se creó nada.
**Resultado esperado:** aviso «Ya existe una empresa registrada con este RUC»; no se crea empresa, sede, empleado, cuenta ni rol.

#### TC-ADM-009 — El usuario ya existe en cualquier empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-001][hu] «El usuario ya existe en cualquier empresa»; [CU-ADM-01][cu] excepción paso 5; [RN-CTA-007][rn] |

**Precondiciones:** la empresa A tiene la cuenta `ana.manual`.
**Datos:** datos válidos para una empresa nueva (por ejemplo la empresa B, RUC `20123456789`) con Usuario Administrador `ana.manual`.
**Pasos:**
1. En `/auth/register` completa el formulario con ese usuario.
2. Haz clic en «Registrar Empresa».
**Resultado esperado:** aviso «El usuario ya está en uso. Elige otro.»; no se crea nada. Nota: el registro tiene límite de 5 envíos por hora por IP; planifica los registros de esta suite (No verificado en la aplicación, como indica la historia).

#### TC-ADM-010 — La API rechaza la dirección de la sede (vacía o de más de 255 caracteres)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-001][hu] «La API rechaza la dirección de la sede»; [CU-ADM-01][cu] excepción paso 4; [RN-EMP-010, RN-GEN-011][rn] |

**Precondiciones:** cliente HTTP; No verificado en la aplicación: el formulario normalmente lo impide antes.
**Datos:** `POST /empresas/registrar` con cuerpo válido salvo `"direccionSede": ""`; repite con una dirección de 256 caracteres.
**Pasos:**
1. Envía cada petición.
2. Si el cuerpo se envía desde el SPA con el formulario manipulado (por ejemplo quitando el validador en las herramientas del navegador), observa el aviso en pantalla.
**Resultado esperado:** la API responde 400 con `code` `RegistrarEmpresaCommand.DireccionSede`; en el SPA el aviso es «Ingresa la dirección de la sede principal (máximo 255 caracteres)».

#### TC-ADM-011 — Cualquier otro rechazo del servidor y exceso de registros por hora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-001][hu] «Cualquier otro rechazo del servidor o exceso de envíos»; [CU-ADM-01][cu] excepción paso 4 y paso 6; [RN-GEN-027][rn] |

**Precondiciones:** cliente HTTP y SPA. No verificado en la aplicación: el 429 no se provocó para no agotar el límite.
**Datos:** (a) petición `POST /empresas/registrar` con `nombreEmpresa` de 201 caracteres; (b) seis registros válidos seguidos desde la misma IP.
**Pasos:**
1. Envía (a) y observa la respuesta y, si se hace desde el SPA, el aviso.
2. Envía los registros de (b) uno tras otro en menos de una hora.
**Resultado esperado:** (a) 400 con `code` `RegistrarEmpresaCommand.NombreEmpresa`; el SPA muestra el aviso genérico «Error al registrar la empresa». (b) A partir del sexto intento la API responde 429 sin `code` y el SPA muestra «Error al registrar la empresa». Hallazgo: la pantalla solo traduce `RegistrarEmpresaCommand.DireccionSede` (Brecha conocida de HU-ADM-001; inventario 19a §1.2).

### HU-ADM-002 — Iniciar sesión

#### TC-ADM-012 — Iniciar sesión con credenciales correctas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-002][hu] «Inicio de sesión correcto»; [CU-ADM-02][cu] flujo básico; [RN-AUT-001, RN-AUT-002, RN-AUT-003, RN-AUT-004, RN-AUT-008, RN-AUT-009][rn] |

**Precondiciones:** la cuenta `ana.manual` está activa y no bloqueada; la empresa está vigente; sin sesión.
**Datos:** Usuario `ana.manual`; su contraseña.
**Pasos:**
1. Abre `/auth/login` («Iniciar sesión»).
2. Escribe usuario y contraseña y haz clic en «Ingresar».
3. Una vez dentro, recarga la página.
**Resultado esperado:** el botón muestra un círculo de espera y no puedes pulsarlo otra vez; entras al Inicio (`/dashboard`) sin ningún aviso; tras recargar con la sesión vigente sigues dentro.

#### TC-ADM-013 — El formulario de ingreso no pide la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-002][hu] «No se pide la empresa»; [CU-ADM-02][cu] flujo básico (paso 4); [RN-AUT-003][rn] |

**Precondiciones:** existen las cuentas `ana.manual` (empresa A) y `luis.dos` (empresa B).
**Datos:** cada usuario con su contraseña.
**Pasos:**
1. Abre `/auth/login` y cuenta los campos del formulario.
2. Entra con `ana.manual`, cierra sesión y entra con `luis.dos`.
**Resultado esperado:** el formulario solo tiene «Usuario» y «Contraseña»; cada cuenta entra a su propia empresa sin indicar cuál es.

#### TC-ADM-014 — Token de acceso, cookie de refresco y hash en base de datos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-002][hu] «El token y el token de refresco»; [CU-ADM-02][cu] flujo básico (pasos 7 y 8); [RN-AUT-010, RN-AUT-011, RN-AUT-012][rn] |

**Precondiciones:** herramientas de desarrollo del navegador abiertas (pestaña de red y almacenamiento); acceso de solo lectura a la base de datos, si se tiene.
**Datos:** `ana.manual`.
**Pasos:**
1. Inicia sesión y selecciona la respuesta de `POST /api/v1/auth/login` en la pestaña de red.
2. Revisa el cuerpo de la respuesta y las cookies.
3. Decodifica el token de acceso (solo su contenido, sin compartirlo).
4. Con acceso a la base de datos, mira la fila nueva en `RefreshTokens`.
**Resultado esperado:** el cuerpo trae el token de acceso y el indicador `requiereCambioClave`, pero no el token de refresco; este llega en la cookie `refresh_token` con `HttpOnly`, `Secure`, `SameSite=None` y vigencia de 7 días; el token de acceso contiene id de cuenta, usuario, nombre completo, nombre de empresa, empleado, sucursal base, sello de seguridad y empresa, y un claim por cada rol vivo de la misma empresa; en la base de datos solo está el hash SHA-256 del token de refresco.

#### TC-ADM-015 — Una cuenta que debe cambiar la contraseña va a «Cambiar contraseña» al ingresar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-002][hu] «Cuenta que debe cambiar la contraseña»; [CU-ADM-02][cu] A1; [RN-AUT-010][rn] |

**Precondiciones:** una cuenta con «Requiere Cambio de Clave: Sí» (por ejemplo `carlos.cajero` recién creada, TC-ADM-274).
**Datos:** usuario y contraseña inicial de la cuenta.
**Pasos:**
1. Inicia sesión en `/auth/login`.
**Resultado esperado:** el sistema te lleva a `/cambiar-clave` en lugar del Inicio.

#### TC-ADM-016 — Usuario y contraseña obligatorios en el ingreso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-002][hu] «Usuario y contraseña obligatorios»; [CU-ADM-02][cu] excepción paso 1; [RN-AUT-001, RN-AUT-002][rn] |

**Precondiciones:** sin sesión; estás en `/auth/login`.
**Datos:** campos vacíos.
**Pasos:**
1. Haz clic en «Ingresar» sin escribir nada.
**Resultado esperado:** se ve «El usuario es obligatorio» y «La contraseña es obligatoria»; no se envía nada al servidor.

#### TC-ADM-017 — Usuario de 51 caracteres enviado a la API

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-002][hu] «Usuario de más de 50 caracteres»; [CU-ADM-02][cu] excepción paso 3; [RN-AUT-001][rn] |

**Precondiciones:** cliente HTTP; para ver el mensaje en pantalla, SPA con el campo manipulado.
**Datos:** `POST /auth/login` con `nombreUsuario` de 51 caracteres y cualquier `password`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** la API responde 400 con `code` `LoginCommand.NombreUsuario`; la pantalla, si recibe esa respuesta, muestra «Credenciales inválidas. Verifica tu usuario y contraseña.» porque no tiene mapa para ese código.

#### TC-ADM-018 — Contraseña incorrecta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-002][hu] «Contraseña incorrecta»; [CU-ADM-02][cu] excepción paso 5; [RN-AUT-006][rn] |

**Precondiciones:** la cuenta `ana.manual` activa y no bloqueada.
**Datos:** Usuario `ana.manual`; una contraseña que no es la suya.
**Pasos:**
1. En `/auth/login` escribe los datos y haz clic en «Ingresar».
**Resultado esperado:** alerta roja «Credenciales inválidas. Verifica tu usuario y contraseña.»; permaneces en `/auth/login`.

#### TC-ADM-019 — El sistema no revela por qué falló el ingreso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-002][hu] «El sistema no revela por qué falló el ingreso»; [CU-ADM-02][cu] excepción paso 5; [RN-AUT-003, RN-AUT-004, RN-AUT-005, RN-AUT-008][rn] |

**Precondiciones:** cuentas de prueba preparadas: una desactivada, una eliminada, una bloqueada; un usuario inexistente. La condición «sucursal base de otra empresa» solo se prepara por base de datos (No verificado en la aplicación).
**Datos:** cada fila de la tabla, con la contraseña correcta de la cuenta cuando existe.
**Pasos:**
1. Para cada fila intenta iniciar sesión.
2. Compara el mensaje y, con herramientas de red, el tiempo de respuesta de un usuario inexistente frente a una contraseña equivocada de una cuenta real.
**Resultado esperado:**

| Situación | Resultado |
|---|---|
| Usuario inexistente | «Credenciales inválidas. Verifica tu usuario y contraseña.» |
| Cuenta eliminada | el mismo mensaje |
| Cuenta desactivada | el mismo mensaje |
| Cuenta bloqueada | el mismo mensaje |
| Sucursal base del empleado de otra empresa | el mismo mensaje |

Los tiempos de respuesta son del mismo orden (la contraseña se verifica siempre con BCrypt, contra el hash real o uno ficticio). La medida exacta del tiempo es No verificada.

#### TC-ADM-020 — Empresa sin suscripción vigente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-002][hu] «La empresa no tiene suscripción vigente»; [HU-ADM-016][hu] «No se puede iniciar sesión»; [CU-ADM-02][cu] excepción paso 6; [RN-AUT-009, RN-EMP-014][rn] |

**Precondiciones:** empresa de prueba suspendida, cancelada o con la prueba vencida (se prepara por base de datos). No verificado en la aplicación: la empresa de prueba del manual está vigente.
**Datos:** usuario y contraseña **correctos** de esa empresa.
**Pasos:**
1. Intenta iniciar sesión.
**Resultado esperado:** aviso «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» (API 403 `Auth.EmpresaNoVigente`).

#### TC-ADM-021 — La vigencia solo se comprueba con la contraseña correcta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-002][hu] «La vigencia solo se comprueba con la contraseña correcta»; [CU-ADM-02][cu] flujo básico (paso 6); [RN-AUT-009][rn] |

**Precondiciones:** la misma empresa no vigente de TC-ADM-020.
**Datos:** usuario correcto y contraseña incorrecta.
**Pasos:**
1. Intenta iniciar sesión.
**Resultado esperado:** «Credenciales inválidas. Verifica tu usuario y contraseña.»; no se indica que la empresa no está vigente.

#### TC-ADM-022 — Demasiados intentos de inicio de sesión desde la misma IP

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-002][hu] «Demasiados intentos desde el mismo equipo»; [CU-ADM-02][cu] excepción paso 3; [RN-GEN-026][rn] |

**Precondiciones:** usa un usuario inexistente (`no.existe`) para no bloquear una cuenta real. No verificado en la aplicación: no se provocó para no bloquear la cuenta de prueba.
**Datos:** 11 intentos de inicio de sesión en un minuto.
**Pasos:**
1. Intenta iniciar sesión 11 veces con ese usuario en menos de un minuto.
2. Espera un minuto e inténtalo de nuevo.
**Resultado esperado:** el intento 11 muestra «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» (API 429 sin `code`); pasado el minuto vuelve a responder «Credenciales inválidas…».

#### TC-ADM-023 — El servidor no responde al iniciar sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-002][hu] «El servidor no responde»; [CU-ADM-02][cu] excepción paso 2; [RN-GEN-040][rn] |

**Precondiciones:** No verificado en la aplicación. Se prueba con la API detenida (solo si el tester controla su propio entorno).
**Datos:** cualquier usuario y contraseña.
**Pasos:**
1. Detén la API local, o bloquea la URL de la API en las herramientas del navegador.
2. Haz clic en «Ingresar».
**Resultado esperado:** «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.»

#### TC-ADM-024 — La contraseña se guarda como hash BCrypt

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-002][hu] «Contraseñas guardadas de forma segura»; [CU-ADM-02][cu] flujo básico (paso 4); [RN-AUT-031, RN-AUT-004][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos.
**Datos:** la cuenta `ana.manual`.
**Pasos:**
1. Consulta el campo `Password_Hash` de la cuenta en `Seguridad_Cuentas`.
**Resultado esperado:** el valor es un hash BCrypt (empieza por `$2`) y no contiene la contraseña escrita. No copies el hash a ningún documento.

### HU-ADM-003 — Bloquear la cuenta tras intentos fallidos

#### TC-ADM-025 — Cada contraseña errónea suma un intento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-003][hu] «Cada contraseña errónea suma un intento»; [CU-ADM-02][cu] excepción paso 5 y postcondiciones; [RN-AUT-005, RN-AUT-006, RN-AUT-007][rn] |

**Precondiciones:** una cuenta de prueba solo para bloqueos (por ejemplo `bloqueo.prueba`) con 0 intentos fallidos; sesión de Ana abierta en otro navegador para consultar la cuenta.
**Datos:** contraseña incorrecta.
**Pasos:**
1. En `/auth/login` escribe el usuario y la contraseña incorrecta y haz clic en «Ingresar».
2. En la sesión de Ana abre «Cuentas de Usuario» y la ficha de la cuenta.
**Resultado esperado:** «Credenciales inválidas. Verifica tu usuario y contraseña.»; la cuenta pasa a 1 intento fallido (campo «Intentos»), y el intento queda guardado aunque la petición termine en error.

#### TC-ADM-026 — Cinco intentos fallidos bloquean la cuenta 15 minutos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-ADM-003][hu] «Cinco intentos fallidos bloquean la cuenta 15 minutos»; [CU-ADM-02][cu] excepción paso 5; [RN-AUT-007][rn] |

**Precondiciones:** la cuenta de TC-ADM-025 con 4 intentos fallidos.
**Datos:** contraseña incorrecta.
**Pasos:**
1. Falla el quinto intento.
2. En «Cuentas de Usuario» busca la cuenta y abre su ficha.
**Resultado esperado:** la cuenta se ve con el candado cerrado y «Intentos» en 5; en la ficha, «Bloqueado: Sí» y la fecha de «Bloqueado Hasta» unos 15 minutos después.

#### TC-ADM-027 — Una cuenta bloqueada no entra ni con la contraseña correcta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-003][hu] «Una cuenta bloqueada no entra ni con la contraseña correcta»; [CU-ADM-02][cu] excepción paso 5; [RN-AUT-005][rn] |

**Precondiciones:** cuenta bloqueada (TC-ADM-026).
**Datos:** la contraseña correcta.
**Pasos:**
1. Intenta iniciar sesión con la contraseña correcta.
2. Revisa «Intentos» en la ficha de la cuenta.
**Resultado esperado:** «Credenciales inválidas. Verifica tu usuario y contraseña.»; el contador no sube (sigue en 5); la pantalla no avisa del bloqueo.

#### TC-ADM-028 — El bloqueo vencido reinicia el contador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-003][hu] «El bloqueo vencido reinicia el contador»; [CU-ADM-02][cu] postcondición «Fallo por contraseña incorrecta»; [RN-AUT-007][rn] |

**Precondiciones:** cuenta bloqueada cuyo `Bloqueado_Hasta` ya pasó (espera 15 minutos o, con acceso a la base de datos, pon la fecha en el pasado).
**Datos:** contraseña incorrecta.
**Pasos:**
1. Falla una contraseña.
2. Mira «Intentos» en la ficha.
**Resultado esperado:** el contador vuelve a 0 antes de sumar el nuevo fallo y queda en 1.

#### TC-ADM-029 — Un ingreso correcto reinicia el contador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-003][hu] «Un ingreso correcto reinicia el contador»; [CU-ADM-02][cu] flujo básico (paso 7); [RN-AUT-010][rn] |

**Precondiciones:** cuenta con 3 intentos fallidos y sin bloqueo.
**Datos:** la contraseña correcta.
**Pasos:**
1. Inicia sesión.
2. Mira «Intentos» y «Bloqueado» en la ficha desde otra sesión.
**Resultado esperado:** los intentos vuelven a 0 y no hay bloqueo.

#### TC-ADM-030 — Formas de desbloquear una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-003][hu] «Formas de desbloquear una cuenta»; [CU-ADM-02][cu] A4; [CU-ADM-34][cu] flujo básico; [RN-AUT-039, RN-GEN-012][rn] |

**Precondiciones:** dos cuentas bloqueadas (A y B); la cuenta B tiene un empleado con correo y el correo de pruebas funciona (si no, solo la parte 1).
**Datos:** cuenta A: administración; cuenta B: enlace del correo.
**Pasos:**
1. Con Ana, abre la ficha de la cuenta A, haz clic en «Editar», marca «Resetear Intentos Fallidos» y guarda.
2. Inicia sesión como A con su contraseña correcta.
3. Para B: pide el enlace en `/auth/forgot-password`, restablece la contraseña (TC-ADM-075) e inicia sesión.
**Resultado esperado:** tras el paso 1 los intentos están en 0 y «Bloqueado: No», y A entra; B queda desbloqueada al restablecer su contraseña y entra con la contraseña nueva.

### HU-ADM-004 — Mantener y renovar la sesión

#### TC-ADM-031 — El token vence y la sesión se renueva sin aviso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-004][hu] «El token de acceso vence y la sesión se renueva sin que me entere»; [CU-ADM-08][cu] flujo básico; [RN-AUT-014, RN-AUT-015, RN-AUT-018, RN-GEN-016][rn] |

**Precondiciones:** sesión iniciada. Para provocar el vencimiento: espera lo que dure el token (la clave `Jwt:ExpiryMinutes`; el valor concreto No verificado) o borra solo el token de `localStorage` (clave `sofia_token`) dejando la cookie `refresh_token`.
**Datos:** ninguno.
**Pasos:**
1. Con el token vencido o borrado, navega a otra pantalla del menú.
2. Mira la pestaña de red.
**Resultado esperado:** aparece una llamada a `POST /auth/refresh` en segundo plano y la pantalla nueva se abre sin ningún aviso.

#### TC-ADM-032 — Una petición con 401 se repite una sola vez

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-004][hu] «Una petición recibe 401 y se repite una sola vez»; [CU-ADM-08][cu] flujo básico (pasos 2 y 8); [RN-GEN-017, RN-GEN-018][rn] |

**Precondiciones:** sesión iniciada con dos pestañas del mismo navegador en una pantalla con listas; el token de acceso vencido (ver TC-ADM-031).
**Datos:** ninguno.
**Pasos:**
1. En ambas pestañas haz clic en «Buscar» casi a la vez.
2. Mira la pestaña de red de cada una.
**Resultado esperado:** cada petición que recibió 401 se repite una vez con el token nuevo y la búsqueda se muestra sin avisos; las renovaciones simultáneas de una misma pestaña se unifican en una sola llamada a `/auth/refresh`.

#### TC-ADM-033 — No se puede renovar la sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-004][hu] «No se puede renovar la sesión»; [CU-ADM-08][cu] excepciones pasos 4 y 5; [RN-AUT-015, RN-AUT-017][rn] |

**Precondiciones:** sesión iniciada; token de acceso vencido (TC-ADM-031).
**Datos:** una de estas causas por vez: cierre de sesión desde otro equipo, cuenta desactivada por un administrador, o cookie `refresh_token` borrada (los 7 días sin uso no se pueden esperar: No verificado).
**Pasos:**
1. Provoca la causa.
2. Navega a otra pantalla.
3. Revisa `localStorage`.
**Resultado esperado:** el sistema te lleva a «Iniciar sesión» sin mostrar ningún aviso y borra tus datos locales de sesión.

#### TC-ADM-034 — Abrir una pantalla sin sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-004][hu] «Abrir una pantalla sin sesión»; [CU-ADM-08][cu] excepción paso 3 (sin cookie); [RN-GEN-018][rn] |

**Precondiciones:** sin sesión y sin cookies de SOFIA.
**Datos:** direcciones `/dashboard`, `/empleados`, `/roles`.
**Pasos:**
1. Escribe cada dirección en el navegador.
**Resultado esperado:** el sistema te lleva a `/auth/login` sin ningún aviso.

#### TC-ADM-035 — Renovar exige la cookie y la cabecera de seguridad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-004][hu] «Renovar exige la cookie y la cabecera de seguridad»; [CU-ADM-08][cu] excepciones paso 3; [RN-AUT-013, RN-AUT-014][rn] |

**Precondiciones:** cliente HTTP; Solo API.
**Datos:** (a) `POST /auth/refresh` con cookie `refresh_token` vigente y **sin** cabecera `X-SOFIA-CSRF`; (b) `POST /auth/refresh` con la cabecera `X-SOFIA-CSRF: 1` y **sin** cookie.
**Pasos:**
1. Envía la petición (a) y luego la (b).
**Resultado esperado:** (a) 403 con `code` `Auth.Csrf`; (b) 401 sin cuerpo de error propio.

#### TC-ADM-036 — Cada renovación rota el token de refresco

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-004][hu] «Cada renovación rota el token de refresco»; [CU-ADM-08][cu] flujo básico (paso 6); [RN-AUT-018][rn] |

**Precondiciones:** sesión iniciada; acceso de solo lectura a `RefreshTokens` (opcional).
**Datos:** ninguno.
**Pasos:**
1. Renueva la sesión (TC-ADM-031) y mira las cookies antes y después.
2. Con la base de datos, compara las filas de `RefreshTokens` de la cuenta.
**Resultado esperado:** el token de refresco usado queda revocado y aparece uno nuevo con 7 días de vigencia; el nuevo token de acceso refleja los roles, la sucursal base y el indicador de cambio de clave actuales.

#### TC-ADM-037 — Dos pestañas renuevan con el mismo token a la vez

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-ADM-004][hu] «Dos pestañas renuevan a la vez con el mismo token»; [CU-ADM-08][cu] excepción paso 6 (carrera); [RN-AUT-019, RN-GEN-035][rn] |

**Precondiciones:** cliente HTTP con la misma cookie de refresco copiada en dos peticiones; las dos se lanzan en paralelo.
**Datos:** una cookie `refresh_token` vigente y la cabecera `X-SOFIA-CSRF: 1`.
**Pasos:**
1. Envía a la vez dos `POST /auth/refresh` con esa cookie.
2. Mira las respuestas y la tabla `RefreshTokens`.
**Resultado esperado:** solo una obtiene el par nuevo (200); la otra recibe 401 y no se crea una segunda sesión.

#### TC-ADM-038 — Detección de copia de la cookie de refresco

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-004][hu] «Detección de copia de la cookie»; [CU-ADM-08][cu] A2; [RN-AUT-016][rn] |

**Precondiciones:** una cookie de refresco ya rotada hace más de 30 segundos (guarda el valor antes de renovar la sesión y espera); sesión viva en otro navegador con la misma cuenta.
**Datos:** la cookie antigua y la cabecera `X-SOFIA-CSRF: 1`.
**Pasos:**
1. Presenta la cookie antigua a `POST /auth/refresh`.
2. En el otro navegador, haz cualquier acción que llame a la API.
**Resultado esperado:** 401 con `code` `Auth.InvalidRefreshToken`; se revocan todos los tokens de refresco de la cuenta y cambia su sello de seguridad; los tokens de acceso vivos dejan de servir (el otro navegador termina en «Iniciar sesión»).

#### TC-ADM-039 — Al renovar se vuelven a comprobar cuenta y empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-004][hu] «Al renovar se vuelven a comprobar cuenta y empresa»; [HU-ADM-016][hu] «La renovación se rechaza sin revocar la sesión»; [CU-ADM-08][cu] excepción paso 5; [RN-AUT-017, RN-CTA-012][rn] |

**Precondiciones:** sesión de una cuenta de prueba con el token de acceso vencido; el caso de empresa no vigente se prepara por base de datos (No verificado en la aplicación).
**Datos:** (a) cuenta desactivada por un administrador; (b) cuenta eliminada; (c) empresa no vigente.
**Pasos:**
1. Para cada causa, provoca la renovación (TC-ADM-031).
2. Para (c), después reactiva la empresa y vuelve a renovar con la misma cookie.
**Resultado esperado:** la API responde 401 `Auth.InvalidRefreshToken` en los tres casos y el SPA te lleva a «Iniciar sesión» sin aviso. En (c) el token de refresco no se revoca: tras reactivar la empresa la renovación funciona y no se interpreta como robo de sesión.

#### TC-ADM-040 — Un cambio de rol invalida el token de acceso actual

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-004][hu] «Un cambio de rol invalida el token actual»; [CU-ADM-08][cu] flujo básico (paso 6) y [CU-ADM-35][cu] flujo básico; [RN-AUT-023, RN-GEN-017][rn] |

**Precondiciones:** `carlos.cajero` con sesión abierta en un navegador; Ana en otro.
**Datos:** Ana cambia el rol de Carlos (por ejemplo, quita «Cajero» y agrega «Admin», o al revés).
**Pasos:**
1. Ana asigna o quita un rol a la cuenta de Carlos (ver TC-ADM-296).
2. Carlos hace cualquier acción con el token que tenía.
3. Carlos vuelve a abrir una pantalla tras la renovación.
**Resultado esperado:** la primera llamada con el token anterior responde 401 (el sello de seguridad cambió); tras renovar la sesión, Carlos recibe un token con los roles nuevos.

#### TC-ADM-041 — Un token inválido recibe 401 sin código

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-004][hu] «Un token no válido recibe 401 sin código»; [CU-ADM-08][cu] excepción paso 4; [RN-GEN-015, RN-GEN-016, RN-GEN-018][rn] |

**Precondiciones:** cliente HTTP; Solo API.
**Datos:** `GET /empleados` con (a) sin cabecera `Authorization`; (b) un token vencido; (c) un token con la firma alterada.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** 401 sin campo `code` en los tres casos. Todo endpoint sin metadatos de autorización exige un usuario autenticado.

#### TC-ADM-042 — Consultar mi perfil con «auth/me»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-004][hu] «Consultar mi perfil»; [CU-ADM-08][cu] flujo básico (paso 8); [RN-AUT-023, RN-CTA-018][rn] |

**Precondiciones:** cliente HTTP, o el SPA con la pestaña de red.
**Datos:** `GET /auth/me` con `Authorization: Bearer <token>` de (a) `carlos.cajero`, (b) una cuenta con «Requiere Cambio de Clave: Sí», (c) un token de una cuenta eliminada.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** (a) 200 con los roles y los permisos distintos (módulo y acción) de Carlos; (b) también 200, porque `me` sigue disponible aunque deba cambiar la contraseña; (c) 404 con `code` `Cuenta.NotFound`.

### HU-ADM-005 — Cerrar sesión

#### TC-ADM-043 — Cerrar sesión desde el menú de usuario

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-005][hu] «Cerrar sesión desde el menú de usuario»; [CU-ADM-03][cu] flujo básico; [RN-AUT-020, RN-AUT-021][rn] |

**Precondiciones:** sesión iniciada.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en la imagen de perfil de la cabecera («Menú de usuario»).
2. Haz clic en «Cerrar sesión».
**Resultado esperado:** el sistema te lleva a `/auth/login`; no pide confirmación ni muestra ningún mensaje.

#### TC-ADM-044 — Cerrar sesión cierra todas las sesiones de la cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-005][hu] «Se cierran todas las sesiones de la cuenta»; [CU-ADM-03][cu] flujo básico (paso 5); [RN-AUT-021, RN-GEN-017][rn] |

**Precondiciones:** la misma cuenta con sesión en dos navegadores distintos (A y B).
**Datos:** `ana.manual`.
**Pasos:**
1. En el navegador A haz clic en «Cerrar sesión».
2. En el navegador B navega a otra pantalla o pulsa «Buscar».
**Resultado esperado:** el navegador B deja de funcionar (los tokens de acceso vivos dejan de servir y el token de refresco está revocado) y termina en «Iniciar sesión» sin aviso.

#### TC-ADM-045 — Cerrar sesión borra los datos locales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-005][hu] «Se borran los datos locales»; [CU-ADM-03][cu] flujo básico (paso 7); [RN-AUT-022][rn] |

**Precondiciones:** sesión iniciada con un filtro de búsqueda aplicado (por ejemplo en «Empleados») y, si hay punto de venta, un borrador de venta.
**Datos:** ninguno.
**Pasos:**
1. Revisa `localStorage` y `sessionStorage` (herramientas del navegador).
2. Cierra sesión y vuelve a revisar.
3. Inicia sesión otra vez y abre la misma búsqueda.
**Resultado esperado:** se eliminan el token (`sofia_token`), los filtros de búsqueda guardados, los borradores de venta y los permisos en memoria; la búsqueda aparece sin los filtros anteriores.

#### TC-ADM-046 — El cierre funciona aunque la llamada falle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-005][hu] «El cierre funciona aunque la llamada falle»; [CU-ADM-03][cu] excepciones paso 6; [RN-AUT-022][rn] |

**Precondiciones:** sesión iniciada; el tester bloquea la llamada `POST /auth/logout` en las herramientas del navegador (o detiene la API).
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cerrar sesión».
**Resultado esperado:** el SPA borra la sesión local y te lleva a `/auth/login`; el error de la llamada se ignora y no se muestra aviso.

#### TC-ADM-047 — El cierre exige la cabecera de seguridad y es anónimo e idempotente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-005][hu] «El cierre exige la cabecera de seguridad»; [HU-ADM-005][hu] «El cierre es anónimo e idempotente»; [CU-ADM-03][cu] excepciones paso 3 y A1; [RN-AUT-013, RN-AUT-020][rn] |

**Precondiciones:** cliente HTTP; Solo API; una cookie de refresco vigente con el token de acceso vencido.
**Datos:** (a) `POST /auth/logout` sin `X-SOFIA-CSRF`; (b) `POST /auth/logout` con `X-SOFIA-CSRF: 1` y la cookie vigente sin token de acceso; (c) `POST /auth/logout` con `X-SOFIA-CSRF: 1`, sin cookie ni token.
**Pasos:**
1. Envía las tres peticiones.
2. Tras (b), usa la cookie en `POST /auth/refresh`.
**Resultado esperado:** (a) 403 `Auth.Csrf`; (b) 204, la cuenta se identifica por la cookie y se cierra, y la cookie `refresh_token` se borra; la renovación posterior falla con 401; (c) 204 igualmente y la cookie se borra.

#### TC-ADM-048 — Renovación simultánea durante el cierre de sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-005][hu] «Renovación simultánea durante el cierre»; [CU-ADM-03][cu] excepción paso 5 (`Concurrency.Conflict`); [RN-AUT-022][rn] |

**Precondiciones:** cliente HTTP capaz de lanzar `logout` y `refresh` a la vez con la misma cookie. No verificado en la aplicación: la carrera es difícil de provocar de forma repetible.
**Datos:** una cookie de refresco vigente.
**Pasos:**
1. Lanza a la vez `POST /auth/logout` y `POST /auth/refresh` con esa cookie, y repite varias veces.
**Resultado esperado:** el cierre recarga los datos y reintenta hasta 3 veces, incluyendo el token recién emitido; casi siempre responde 204; si el tercer intento también falla responde 409 `Concurrency.Conflict`, que el SPA ignora.

### HU-ADM-006 — Cambiar mi contraseña

#### TC-ADM-049 — Cambiar la contraseña

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-006][hu] «Cambiar la contraseña»; [CU-ADM-04][cu] flujo básico; [RN-AUT-024, RN-AUT-025, RN-AUT-026, RN-AUT-027, RN-AUT-030, RN-AUT-031, RN-GEN-013][rn] |

**Precondiciones:** sesión de `ana.manual`, con una segunda sesión de la misma cuenta abierta en otro navegador; acceso de solo lectura a `Auditoria_Eventos_Seguridad` (opcional).
**Datos:** Contraseña actual: la de Ana; Nueva contraseña: otra de 8 caracteres o más y distinta; Confirmar nueva contraseña: la misma.
**Pasos:**
1. Menú de usuario › «Cambiar contraseña».
2. Escribe los tres campos y haz clic en «Guardar».
3. En el otro navegador, realiza una acción.
4. Intenta entrar con la contraseña anterior y luego con la nueva.
**Resultado esperado:** aviso «Contraseña actualizada. Inicia sesión con tu nueva contraseña.»; el sistema cierra todas las sesiones (incluida la actual y la del otro navegador) y te lleva a «Iniciar sesión»; la contraseña anterior ya no sirve y la nueva sí; se registra el evento de auditoría `CLAVE_CAMBIAR`.

#### TC-ADM-050 — La pantalla de cambio voluntario advierte del cierre de sesiones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-006][hu] «La pantalla advierte del cierre de sesiones»; [CU-ADM-04][cu] flujo básico (paso 1); [RN-AUT-030][rn] |

**Precondiciones:** sesión iniciada sin obligación de cambiar la contraseña.
**Datos:** ninguno.
**Pasos:**
1. Abre `/cambiar-clave` desde el menú de usuario.
**Resultado esperado:** se lee «Al guardar se cerrarán todas tus sesiones y tendrás que iniciar sesión con tu nueva contraseña»; ves los botones «Cancelar» y «Guardar» y la flecha de volver.

#### TC-ADM-051 — Volver sin cambiar la contraseña

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-006][hu] «Volver sin cambiar nada»; [CU-ADM-04][cu] A1 |

**Precondiciones:** estás en `/cambiar-clave` en modo voluntario.
**Datos:** campos con texto escrito.
**Pasos:**
1. Haz clic en «Cancelar»; repite con la flecha de volver y con la miga «Inicio».
2. Intenta entrar de nuevo más tarde con la contraseña actual.
**Resultado esperado:** vuelves al Inicio con las tres formas; la contraseña no cambia y la sesión sigue abierta.

#### TC-ADM-052 — Validaciones del formulario de cambio de contraseña

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-006][hu] «Validaciones del formulario» (esquema); [CU-ADM-04][cu] excepción paso 3; [RN-AUT-025, RN-AUT-026, RN-AUT-027][rn] |

**Precondiciones:** sesión iniciada; estás en `/cambiar-clave`.
**Datos:** una fila por vez.
**Pasos:**
1. Para cada fila deja el campo con el valor indicado (los demás válidos) y haz clic en «Guardar».
2. Mira la pestaña de red.
**Resultado esperado:** no se envía nada al servidor y se ve el mensaje debajo del campo:

| Campo | Valor | Mensaje |
|---|---|---|
| Contraseña actual | vacío | Este campo es obligatorio |
| Nueva contraseña | vacío | Este campo es obligatorio |
| Nueva contraseña | 7 caracteres | Debe tener al menos 8 caracteres |
| Nueva contraseña | igual a la contraseña actual | La nueva contraseña debe ser distinta de la actual |
| Confirmar nueva contraseña | vacío | Este campo es obligatorio |
| Confirmar nueva contraseña | distinta de la nueva | Las contraseñas no coinciden |

#### TC-ADM-053 — Contraseña actual incorrecta al cambiarla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-006][hu] «Contraseña actual incorrecta»; [CU-ADM-04][cu] excepción paso 6; [RN-AUT-029][rn] |

**Precondiciones:** sesión iniciada; la cuenta con 0 intentos fallidos.
**Datos:** Contraseña actual equivocada; Nueva contraseña válida y confirmada.
**Pasos:**
1. Haz clic en «Guardar».
2. Mira la respuesta de red y la ficha de la cuenta (campo «Intentos»).
**Resultado esperado:** aviso «La contraseña actual es incorrecta»; permaneces en la pantalla con la sesión intacta (la API responde 400, no 401); el contador de intentos no sube.

#### TC-ADM-054 — Solo se cambia la contraseña de la cuenta del token

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-006][hu] «Solo se cambia la contraseña de la cuenta del token»; [CU-ADM-04][cu] flujo básico (paso 5); [RN-AUT-024][rn] |

**Precondiciones:** cliente HTTP; Solo API; `<token>` de Carlos.
**Datos:** `POST /auth/change-password` con `{ "currentPassword": "<la de Carlos>", "newPassword": "<8+ caracteres>", "nombreUsuario": "ana.manual" }` (el campo extra no debe tener efecto); y otra con un token cuyo id de cuenta no es válido.
**Pasos:**
1. Envía la primera petición.
2. Intenta entrar como `ana.manual` con su contraseña anterior.
3. Envía la segunda con un token manipulado.
**Resultado esperado:** la contraseña cambiada es la de Carlos (la de Ana no cambia); si el token no trae un id válido, la API responde 401.

#### TC-ADM-055 — Efectos del cambio sobre la cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-006][hu] «Efectos del cambio sobre la cuenta»; [CU-ADM-04][cu] flujo básico (pasos 7 y 8) y postcondiciones; [RN-AUT-030, RN-AUT-031][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos; una cuenta con «Requiere Cambio de Clave: Sí» y con un token de recuperación pendiente (pide el enlace sin usarlo).
**Datos:** la contraseña actual y una nueva válida.
**Pasos:**
1. Cambia la contraseña desde `/cambiar-clave`.
2. Consulta `Seguridad_Cuentas` y `RefreshTokens` de la cuenta.
3. Usa el enlace de recuperación pendiente (TC-ADM-066).
**Resultado esperado:** `Password_Hash` y `Security_Stamp` cambian; `Requiere_Cambio_Clave` pasa a falso; el `Recovery_Token` queda anulado (el enlace pendiente ya no sirve: «El enlace no es válido o ya expiró. Solicita uno nuevo.»); todos los tokens de refresco quedan revocados y la cookie se borra.

#### TC-ADM-056 — Cuenta desactivada o eliminada durante el cambio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-006][hu] «Cuenta desactivada o eliminada durante el cambio»; [CU-ADM-04][cu] excepción paso 5; [RN-AUT-028][rn] |

**Precondiciones:** Carlos tiene abierta `/cambiar-clave`; Ana desactiva la cuenta de Carlos (TC-ADM-286).
**Datos:** campos válidos.
**Pasos:**
1. Carlos hace clic en «Guardar».
**Resultado esperado:** la API responde 401 `Auth.InvalidCredentials`; el sistema intenta renovar la sesión y, como no puede, lleva a Carlos a «Iniciar sesión».

#### TC-ADM-057 — Error del servidor o exceso de intentos al cambiar la contraseña

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-006][hu] «Error del servidor o exceso de intentos»; [CU-ADM-04][cu] excepciones pasos 4 y 5; [RN-GEN-026][rn] |

**Precondiciones:** No verificado en la aplicación: no se pudo provocar. Se necesita lanzar más de 10 peticiones por minuto a `POST /auth/change-password` o detener la API.
**Datos:** contraseña actual incorrecta repetida 11 veces en un minuto.
**Pasos:**
1. Haz clic en «Guardar» 11 veces en menos de un minuto.
**Resultado esperado:** a partir de la petición 11 (429), el aviso es «No se pudo cambiar la contraseña. Inténtalo de nuevo.»; el mismo aviso aparece si el servidor falla.

### HU-ADM-007 — Cambiar la contraseña cuando es obligatorio

#### TC-ADM-058 — Primer ingreso de una cuenta creada por un administrador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-007][hu] «Primer ingreso de una cuenta creada por un administrador»; [CU-ADM-05][cu] flujo básico (pasos 1 y 2); [RN-AUT-010, RN-CTA-008][rn] |

**Precondiciones:** la cuenta `carlos.cajero` fue creada por Ana (TC-ADM-274) y tiene «Requiere Cambio de Clave: Sí».
**Datos:** usuario `carlos.cajero` y su contraseña inicial.
**Pasos:**
1. Inicia sesión.
**Resultado esperado:** no ves el Inicio: ves «Cambiar contraseña» con el aviso «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.»; no existen el botón «Cancelar» ni la flecha de volver.

#### TC-ADM-059 — El menú no deja salir de la pantalla de cambio obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-007][hu] «El menú no deja salir de la pantalla»; [CU-ADM-05][cu] flujo básico (paso 3); [RN-GEN-019][rn] |

**Precondiciones:** Carlos en modo forzado (TC-ADM-058).
**Datos:** direcciones `/pos`, `/dashboard`; clic en cualquier opción del menú lateral.
**Pasos:**
1. Haz clic en cualquier opción del menú lateral.
2. Escribe a mano `/pos` en la barra del navegador.
**Resultado esperado:** en ambos casos el sistema te devuelve a `/cambiar-clave` sin mostrar ningún aviso.

#### TC-ADM-060 — Completar el cambio obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-007][hu] «Completar el cambio obligatorio»; [CU-ADM-05][cu] flujo básico (pasos 5 y 6); [RN-AUT-030, RN-CTA-008][rn] |

**Precondiciones:** Carlos en modo forzado.
**Datos:** Contraseña actual: la inicial; Nueva contraseña válida y confirmada.
**Pasos:**
1. Escribe los tres campos y haz clic en «Guardar».
2. En «Iniciar sesión», entra con la contraseña nueva.
3. Como Ana, abre la ficha de la cuenta de Carlos.
**Resultado esperado:** aviso «Contraseña actualizada. Inicia sesión con tu nueva contraseña.»; el sistema te lleva a «Iniciar sesión»; al entrar con la contraseña nueva no se pide el cambio; en la ficha de la cuenta «Requiere Cambio de Clave» pasa a «No».

#### TC-ADM-061 — El administrador fuerza el cambio con la sesión ya abierta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-007][hu] «El administrador fuerza el cambio con mi sesión ya abierta»; [CU-ADM-05][cu] A1; [RN-GEN-019, RN-CTA-011][rn] |

**Precondiciones:** Carlos con sesión abierta en un navegador; Ana en otro.
**Datos:** Ana activa «Forzar Cambio de Contraseña» en la cuenta de Carlos (TC-ADM-288).
**Pasos:**
1. Carlos hace su siguiente acción (navegar a otra pantalla).
2. Con cliente HTTP, llama con el token vivo de Carlos a `GET /empleados`, a `GET /auth/me` y a `POST /auth/change-password`.
**Resultado esperado:** el sistema lleva a Carlos a `/cambiar-clave`; la API responde 403 con `code` `Auth.CambioClaveRequerido` a cualquier endpoint, salvo `GET /auth/me` y `POST /auth/change-password`; el indicador se lee de la base de datos en cada petición, por lo que también aplica a tokens vivos.

#### TC-ADM-062 — Contraseña actual equivocada en el modo forzado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-007][hu] «Contraseña actual equivocada en el modo forzado»; [CU-ADM-05][cu] A2; [RN-AUT-029][rn] |

**Precondiciones:** Carlos en modo forzado.
**Datos:** Contraseña actual incorrecta.
**Pasos:**
1. Completa el formulario con una contraseña actual equivocada y haz clic en «Guardar».
**Resultado esperado:** aviso rojo «La contraseña actual es incorrecta»; sigues en `/cambiar-clave` en modo forzado.

#### TC-ADM-063 — El administrador que registra la empresa no pasa por el cambio obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-007][hu] «El administrador que registra la empresa no pasa por este paso»; [CU-ADM-01][cu] flujo básico (paso 6); [RN-AUT-010][rn] |

**Precondiciones:** una empresa recién registrada (TC-ADM-001).
**Datos:** ninguno.
**Pasos:**
1. Tras registrar, cierra sesión y entra con el usuario administrador.
2. Revisa la ficha de la cuenta.
**Resultado esperado:** entras al Inicio sin pasar por «Cambiar contraseña»; «Requiere Cambio de Clave» es «No».

#### TC-ADM-064 — Con el cambio obligatorio pendiente, la API rechaza los demás endpoints

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-007][hu] «El administrador fuerza el cambio con mi sesión ya abierta» (nota final: la API no exime ningún otro endpoint); [CU-ADM-05][cu] flujo básico (paso 4); [RN-GEN-019][rn] |

**Precondiciones:** cliente HTTP; Solo API; token de una cuenta con «Requiere Cambio de Clave: Sí».
**Datos:** `GET /sucursales`, `GET /roles`, `POST /auth/logout` (anónimo), `POST /auth/refresh` (anónimo).
**Pasos:**
1. Envía cada petición con el token.
**Resultado esperado:** los dos primeros responden 403 `Auth.CambioClaveRequerido`; los endpoints anónimos (`logout`, `refresh`) no se bloquean por este motivo.

### HU-ADM-008 — Pedir el enlace para recuperar la contraseña

#### TC-ADM-065 — Pedir el enlace de recuperación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | [HU-ADM-008][hu] «Pedir el enlace»; [CU-ADM-06][cu] flujo básico; [RN-AUT-032, RN-AUT-033, RN-AUT-034, RN-AUT-035, RN-AUT-036][rn] |

**Precondiciones:** Ana activa, con empleado con correo `ana.demo@sofia-manual.test` y empresa vigente; Mailpit disponible.
**Datos:** Usuario `ana.manual`.
**Pasos:**
1. Abre `/auth/forgot-password` («Recupera tu contraseña»).
2. Escribe el usuario y haz clic en «Enviar enlace».
**Resultado esperado:** el botón muestra un círculo de espera; el formulario se reemplaza por una alerta verde con «Si el usuario existe y tiene un correo registrado, recibirás un enlace para restablecer tu contraseña. Revisa tu bandeja de entrada: el enlace es válido por 1 hora.»; solo queda el enlace «Volver a iniciar sesión».

#### TC-ADM-066 — El correo de recuperación y su enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | [HU-ADM-008][hu] «El correo que recibo»; [CU-ADM-06][cu] flujo básico (paso 5); [RN-AUT-036][rn] |

**Precondiciones:** TC-ADM-065 ejecutado; bandeja de Mailpit abierta.
**Datos:** ninguno.
**Pasos:**
1. Abre el mensaje recibido en la bandeja de `ana.demo@sofia-manual.test`.
2. Lee el asunto, el texto y la dirección del enlace (sin compartirla: es un secreto de un solo uso).
**Resultado esperado:** asunto «Restablece tu contraseña de SOFIA»; el texto dice que el enlace es válido por 1 hora y solo puede usarse una vez; el enlace tiene la forma `.../auth/reset-password#token=…&usuario=…` con el token en el fragmento.

#### TC-ADM-067 — La respuesta es siempre la misma, exista o no la cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-008][hu] «La respuesta es siempre la misma»; [CU-ADM-06][cu] A1; [RN-AUT-033, RN-AUT-034][rn] |

**Precondiciones:** Mailpit vacío o anotado el número de mensajes; cuentas: una inexistente, una con empleado sin correo, una desactivada.
**Datos:** `no.existe`, la cuenta sin correo, la cuenta desactivada.
**Pasos:**
1. Para cada usuario, pide el enlace.
2. Revisa la bandeja de Mailpit.
**Resultado esperado:** en los tres casos aparece exactamente la misma alerta verde de TC-ADM-065 y no llega ningún correo.

#### TC-ADM-068 — Cuándo se genera de verdad el enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-008][hu] «Cuándo se genera de verdad el enlace»; [CU-ADM-06][cu] flujo básico (paso 4); [RN-AUT-034, RN-AUT-035][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos; una cuenta elegible (activa, no eliminada, con empleado con correo y empresa vigente). No verificado en la aplicación: la comprobación del token se basa en el código.
**Datos:** la cuenta `ana.manual`.
**Pasos:**
1. Pide el enlace.
2. Consulta `Recovery_Token` y `Recovery_Token_Expiry` de la cuenta en `Seguridad_Cuentas`.
3. Repite con una cuenta de empresa no vigente.
**Resultado esperado:** se genera un token aleatorio de 32 bytes cuyo hash SHA-256 (no el token) está guardado, con vencimiento 1 hora después; reemplaza cualquier token anterior; con la empresa no vigente no se genera token.

#### TC-ADM-069 — Solo sirve el último enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-008][hu] «Solo sirve el último enlace»; [CU-ADM-06][cu] A2; [RN-AUT-035][rn] |

**Precondiciones:** Mailpit disponible.
**Datos:** `ana.manual`.
**Pasos:**
1. Pide el enlace dos veces seguidas.
2. Abre el enlace del primer correo y trata de guardar una contraseña nueva.
3. Abre el enlace del segundo correo y guarda una contraseña nueva.
**Resultado esperado:** con el primer enlace aparece la alerta roja «El enlace no es válido o ya expiró. Solicita uno nuevo.»; con el segundo, la contraseña se restablece (TC-ADM-075).

#### TC-ADM-070 — Falla el envío del correo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | [HU-ADM-008][hu] «Falla el envío del correo»; [CU-ADM-06][cu] flujo básico (paso 5) y A4; [RN-AUT-036][rn] |

**Precondiciones:** No verificado en la aplicación. Se necesita un entorno con el proveedor de correo caído (por ejemplo, Mailpit detenido con la API configurada para SMTP).
**Datos:** `ana.manual`.
**Pasos:**
1. Detén el servidor de correo de pruebas.
2. Pide el enlace.
3. Revisa la base de datos.
**Resultado esperado:** sigues viendo la alerta verde uniforme; el token sigue guardado y válido durante su hora (solo se registra una advertencia en el log). Si el entorno no tiene proveedor de correo, el mensaje se descarta (Brecha conocida de la historia).

#### TC-ADM-071 — Usuario vacío o demasiado largo al pedir el enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-008][hu] «Usuario vacío o demasiado largo»; [CU-ADM-06][cu] excepción paso 1 y paso 3; [RN-AUT-032][rn] |

**Precondiciones:** estás en `/auth/forgot-password`; cliente HTTP para la parte de API.
**Datos:** (a) campo vacío; (b) escribir más de 50 caracteres en el campo; (c) `POST /auth/forgot-password` con `nombreUsuario` vacío y otra con 51 caracteres.
**Pasos:**
1. Haz clic en «Enviar enlace» con el campo vacío.
2. Intenta escribir 51 caracteres en el campo.
3. Envía las peticiones de (c).
**Resultado esperado:** (a) «El usuario es obligatorio» y no se envía nada; (b) el campo no deja escribir más de 50 caracteres; (c) Hallazgo: la API responde 200 con el mensaje uniforme en ambos casos, porque el controlador descarta el resultado del comando (Brecha conocida de HU-ADM-008; RN-AUT-032).

#### TC-ADM-072 — Demasiadas solicitudes de enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-008][hu] «Demasiadas solicitudes»; [CU-ADM-06][cu] excepción paso 3; [RN-GEN-026][rn] |

**Precondiciones:** No verificado en la aplicación. Usa un usuario inexistente.
**Datos:** 11 pedidos en un minuto desde la misma IP.
**Pasos:**
1. Haz clic en «Enviar enlace» 11 veces en menos de un minuto.
**Resultado esperado:** «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» (API 429).

#### TC-ADM-073 — Error del servidor al pedir el enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-008][hu] «Error del servidor»; [CU-ADM-06][cu] excepción paso 2 (sin conexión o 5xx); [RN-GEN-040][rn] |

**Precondiciones:** No verificado en la aplicación; API detenida o bloqueada desde el navegador.
**Datos:** cualquier usuario.
**Pasos:**
1. Haz clic en «Enviar enlace».
**Resultado esperado:** «No se pudo procesar la solicitud. Inténtalo de nuevo.»

#### TC-ADM-074 — Un cajero recupera su contraseña con el correo de su ficha de empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | [HU-ADM-008][hu] «Un cajero recupera su contraseña con el correo de su ficha de empleado»; [CU-ADM-06][cu] flujo básico (paso 5); [RN-EPL-009][rn] |

**Precondiciones:** la ficha de Carlos tiene el correo `carlos.cajero@sofia-manual.test`; la cuenta de Carlos está activa.
**Datos:** Usuario `carlos.cajero`.
**Pasos:**
1. Pide el enlace con el usuario de Carlos.
2. Mira la bandeja de Mailpit.
**Resultado esperado:** el correo llega a `carlos.cajero@sofia-manual.test`. Si el empleado no tiene correo, no llega nada y el administrador solo puede usar «Forzar Cambio de Contraseña» (Brecha conocida de la historia).

### HU-ADM-009 — Restablecer la contraseña con el enlace

#### TC-ADM-075 — Restablecer la contraseña con el enlace

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | [HU-ADM-009][hu] «Restablecer la contraseña»; [CU-ADM-07][cu] flujo básico; [RN-AUT-037, RN-AUT-038, RN-AUT-031][rn] |

**Precondiciones:** enlace vigente en el correo (TC-ADM-066).
**Datos:** Nueva contraseña de 8 caracteres o más y la misma en Confirmar nueva contraseña.
**Pasos:**
1. Abre el enlace del correo («Crea una nueva contraseña»); mira la barra de direcciones.
2. Escribe las dos contraseñas y haz clic en «Guardar contraseña».
3. Entra con el usuario y la contraseña nueva.
**Resultado esperado:** la dirección del navegador se limpia sola y no muestra el usuario; aviso «Contraseña restablecida. Inicia sesión con tu nueva contraseña.»; el sistema te lleva a «Iniciar sesión» y entras con la contraseña nueva.

#### TC-ADM-076 — Efectos del restablecimiento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-009][hu] «Efectos del restablecimiento»; [CU-ADM-07][cu] flujo básico (pasos 6 y 7); [RN-AUT-039, RN-GEN-013, RN-GEN-017][rn] |

**Precondiciones:** cuenta bloqueada y con «Requiere Cambio de Clave: Sí», con sesión viva en otro navegador; acceso de solo lectura a la base de datos.
**Datos:** enlace vigente.
**Pasos:**
1. Restablece la contraseña (TC-ADM-075).
2. En el otro navegador realiza una acción.
3. Consulta la cuenta, `RefreshTokens` y `Auditoria_Eventos_Seguridad`.
4. Intenta usar el mismo enlace otra vez.
**Resultado esperado:** nuevo hash y nuevo sello; «Requiere Cambio de Clave» apagado; el token queda consumido (el segundo uso falla con «El enlace no es válido o ya expiró. Solicita uno nuevo.»); intentos fallidos en 0 y sin bloqueo; todas las sesiones de la cuenta se cierran; existe el evento `CLAVE_RESTABLECER` con el dueño de la cuenta como actor.

#### TC-ADM-077 — Enlace incompleto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-009][hu] «Enlace incompleto»; [CU-ADM-07][cu] A1; [RN-AUT-037][rn] |

**Precondiciones:** sin sesión.
**Datos:** direcciones `/auth/reset-password`, `/auth/reset-password#token=abc` y `/auth/reset-password#usuario=ana.manual`.
**Pasos:**
1. Abre cada dirección.
2. Haz clic en «Solicitar un nuevo enlace».
**Resultado esperado:** alerta roja «Este enlace de recuperación está incompleto o ya fue usado. Solicita uno nuevo.»; no aparece el formulario; el botón lleva a `/auth/forgot-password`.

#### TC-ADM-078 — Validaciones del formulario de restablecimiento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-009][hu] «Validaciones del formulario» (esquema); [CU-ADM-07][cu] excepción paso 2; [RN-AUT-037][rn] |

**Precondiciones:** enlace válido abierto.
**Datos:** una fila por vez.
**Pasos:**
1. Deja cada campo como indica la tabla y haz clic en «Guardar contraseña».
**Resultado esperado:** no se envía nada y se ve:

| Campo | Valor | Mensaje |
|---|---|---|
| Nueva contraseña | vacío | La contraseña es obligatoria |
| Nueva contraseña | 7 caracteres | Debe tener al menos 8 caracteres |
| Confirmar nueva contraseña | vacío | La contraseña es obligatoria |
| Confirmar nueva contraseña | distinta de la nueva | Las contraseñas no coinciden |

#### TC-ADM-079 — El enlace ya se usó, venció o fue reemplazado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-009][hu] «El enlace ya se usó, venció o fue reemplazado»; [CU-ADM-07][cu] excepción paso 5; [RN-AUT-038][rn] |

**Precondiciones:** (a) un enlace ya usado; (b) un enlace de más de 1 hora (espera, o con acceso a la base de datos pon el vencimiento en el pasado); (c) un enlace reemplazado por otro más nuevo.
**Datos:** nueva contraseña válida; además una petición a `POST /auth/reset-password` con un `nombreUsuario` inexistente y un token cualquiera.
**Pasos:**
1. Con cada enlace, escribe la contraseña y haz clic en «Guardar contraseña».
2. Envía la petición con usuario inexistente.
**Resultado esperado:** alerta roja «El enlace no es válido o ya expiró. Solicita uno nuevo.»; un usuario inexistente recibe la misma respuesta (400 `Auth.InvalidToken`) que un token incorrecto.

#### TC-ADM-080 — Demasiados envíos al restablecer

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-009][hu] «Demasiados envíos»; [CU-ADM-07][cu] excepción paso 3; [RN-GEN-026][rn] |

**Precondiciones:** No verificado en la aplicación.
**Datos:** 11 envíos en un minuto con un token incorrecto.
**Pasos:**
1. Haz clic en «Guardar contraseña» 11 veces en menos de un minuto.
**Resultado esperado:** «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» (429).

#### TC-ADM-081 — Error del servidor al restablecer

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-009][hu] «Error del servidor»; [CU-ADM-07][cu] excepciones pasos 3 y 4; [RN-GEN-040][rn] |

**Precondiciones:** No verificado en la aplicación; API detenida.
**Datos:** formulario válido.
**Pasos:**
1. Haz clic en «Guardar contraseña».
**Resultado esperado:** «No se pudo restablecer la contraseña. Inténtalo de nuevo.»

## 4.2 Menú, permisos de pantalla y cabecera

### HU-ADM-010 — Ver solo las opciones de menú que mi rol permite

#### TC-ADM-082 — El administrador ve todo el menú

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-010][hu] «El administrador ve todo el menú»; sin caso de uso propio (el menú se describe en [19a][p19a] §4.3); [RN-GEN-020, RN-GEN-021, RN-ROL-015][rn] |

**Precondiciones:** sesión de `ana.manual` (rol `Admin`).
**Datos:** ninguno.
**Pasos:**
1. Entra al sistema y mira el menú lateral.
2. Cuenta los grupos y las opciones.
**Resultado esperado:** 8 grupos y 24 opciones: Dashboard; Empresa, Sucursales, Empleados, Roles & Permisos y Cuentas de Usuario; los nueve catálogos; Lotes y Stock por Sucursal; Sesiones de Caja y Ventas; Pacientes, Profesionales de Salud y Recetas Médicas; Catálogo DIGEMID; y Series Fiscales.

#### TC-ADM-083 — El cajero ve solo lo que su rol puede leer

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-010][hu] «El cajero ve solo lo que su rol puede leer»; [RN-GEN-020, RN-ROL-015][rn] |

**Precondiciones:** rol «Cajero» con los 19 permisos recomendados (ver sección 2.2); la cuenta `carlos.cajero` con ese rol y sin cambio de contraseña pendiente.
**Datos:** ninguno.
**Pasos:**
1. Inicia sesión como `carlos.cajero`.
2. Revisa el menú lateral.
**Resultado esperado:** ve Dashboard, Empleados, Medicamentos, Aseguradoras, Lotes, Stock por Sucursal, Sesiones de Caja, Ventas, Pacientes, Profesionales de Salud, Recetas Médicas y Catálogo DIGEMID; no ve «Empresa», «Sucursales», «Roles & Permisos», «Cuentas de Usuario» ni «Series Fiscales»; el grupo SUNAT desaparece por quedarse sin opciones.

#### TC-ADM-084 — Un rol sin permisos solo ve Dashboard

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-ADM-010][hu] «Dashboard siempre está»; [RN-GEN-020][rn] |

**Precondiciones:** rol «Sin Permisos Demo» creado sin marcar ningún permiso, asignado a una cuenta `sin.permisos`.
**Datos:** ninguno.
**Pasos:**
1. Inicia sesión como `sin.permisos`.
**Resultado esperado:** el menú solo muestra «Dashboard».

#### TC-ADM-085 — Mientras llegan los permisos solo se ve Dashboard

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-ADM-010][hu] «Mientras llegan los permisos solo se ve Dashboard»; [RN-AUT-023][rn] |

**Precondiciones:** sesión de Carlos; en las herramientas del navegador, limitación de red a «conexión lenta» o retraso de la llamada `GET /auth/me`.
**Datos:** ninguno.
**Pasos:**
1. Recarga la página con la red limitada.
2. Mira el menú antes de que llegue la respuesta y después.
**Resultado esperado:** al principio el menú muestra únicamente «Dashboard»; las demás opciones aparecen cuando llegan los permisos.

#### TC-ADM-086 — No se pueden cargar los permisos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-010][hu] «No se pueden cargar los permisos»; [RN-AUT-023][rn] |

**Precondiciones:** sesión de Carlos; en las herramientas del navegador, bloqueo de la URL `/auth/me`.
**Datos:** ninguno.
**Pasos:**
1. Con el bloqueo activo, recarga la página.
**Resultado esperado:** aviso «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.»; el menú queda solo con «Dashboard».

#### TC-ADM-087 — Quitar un permiso al rol cambia el menú

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-010][hu] «Quitar un permiso al rol cambia el menú»; [CU-ADM-28][cu] flujo básico; [RN-GEN-022, RN-ROL-015][rn] |

**Precondiciones:** rol «Cajero» con `Profesionales de Salud › Leer`; Carlos con sesión abierta en otro navegador.
**Datos:** desmarcar «Profesionales de Salud › Leer».
**Pasos:**
1. Con Ana, abre el rol «Cajero» › «Editar» › pestaña «Permisos», desmarca esa casilla y guarda; lee el aviso de la pantalla del rol sobre usuarios conectados.
2. En el navegador de Carlos, sin recargar, mira el menú.
3. Carlos recarga la página o vuelve a iniciar sesión.
**Resultado esperado:** la pantalla del rol avisa que los usuarios conectados verán el menú actualizado al recargar la página; Carlos no ve el cambio al instante; cuando vuelve a iniciar sesión (o recarga) la opción «Profesionales de Salud» ya no aparece. Restaura el permiso al terminar.

#### TC-ADM-088 — El perfil devuelve roles y permisos distintos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-010][hu] «El perfil devuelve roles y permisos distintos»; [RN-AUT-023][rn] |

**Precondiciones:** Solo API; `<token>` de Carlos; si es posible, un rol adicional que repita un permiso del rol «Cajero».
**Datos:** `GET /auth/me`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 200 con los roles de la cuenta y la lista de permisos (módulo y acción) sin repetidos.

#### TC-ADM-089 — Los módulos sin pantalla no tienen opción de menú

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-010][hu] «Módulos sin pantalla no tienen opción de menú»; [RN-GEN-020][rn] |

**Precondiciones:** sesión de `ana.manual`; abre la pestaña «Permisos» de un rol.
**Datos:** ninguno.
**Pasos:**
1. En un rol, revisa la matriz de permisos y busca Transferencias, Despachos Delivery, Órdenes Magistrales, Servicios y Auditoría de Seguridad.
2. Revisa el menú lateral.
**Resultado esperado:** esos módulos existen como permisos en la matriz, pero el menú no tiene ninguna opción para ellos.

#### TC-ADM-090 — Comportamiento del menú: resaltado y colapso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-010][hu] «Comportamiento del menú»; [RN-GEN-020][rn] |

**Precondiciones:** sesión iniciada en una pantalla del menú.
**Datos:** ninguno.
**Pasos:**
1. Observa la opción actual del menú.
2. Haz clic en el botón de tres rayas.
3. Recarga la página.
4. Busca un buscador de opciones en el menú y en la cabecera.
**Resultado esperado:** la opción actual aparece resaltada; el menú se colapsa a una columna de 80 píxeles con solo iconos; al recargar vuelve a verse abierto (la preferencia no se guarda); no hay buscador de opciones. No verificado en la aplicación: la vista angosta (menos de 1024 px) y el Ctrl + clic para abrir una opción en otra pestaña.

### HU-ADM-011 — Entrar solo a las pantallas que mi rol permite

#### TC-ADM-091 — Abrir por dirección una pantalla sin permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-011][hu] «Abrir por dirección una pantalla sin permiso»; [CU-ADM-09][cu] A2; [RN-GEN-020, RN-ROL-001, RN-SUC-001, RN-EPL-001, RN-SER-001, RN-EMP-001, RN-CTA-001][rn] |

**Precondiciones:** Carlos (rol «Cajero») con sesión iniciada.
**Datos:** direcciones `/roles`, `/cuentas`, `/empresas`, `/sucursales`, `/series-fiscales`.
**Pasos:**
1. Escribe cada dirección en la barra del navegador.
**Resultado esperado:** en cada una el sistema te devuelve al Dashboard y se ve el aviso rojo «No tienes permisos para acceder a esta pantalla.»

#### TC-ADM-092 — Pantallas de creación y carga masiva sin la acción Crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-011][hu] «Pantalla de creación sin la acción Crear»; [HU-ADM-011][hu] «Carga masiva sin la acción Crear»; [CU-ADM-12][cu] precondiciones; [RN-EPL-001, RN-SUC-001][rn] |

**Precondiciones:** rol «Consulta Demo» (solo `Leer` en Empresas, Sucursales, Empleados, Roles y Cuentas de Usuario y Series Fiscales) asignado a la cuenta `consulta.demo`. La historia marca la carga masiva «No verificado» porque el administrador tiene todos los permisos; con este rol sí se puede probar.
**Datos:** direcciones de la tabla.
**Pasos:**
1. Inicia sesión como `consulta.demo`.
2. Escribe cada dirección en la barra.
**Resultado esperado:**

| Dirección | Permiso que falta | Resultado |
|---|---|---|
| `/empleados/nueva` | Empleados › Crear | «No tienes permisos para acceder a esta pantalla.» y vuelves al Dashboard |
| `/sucursales/nueva` | Sucursales › Crear | igual |
| `/sucursales/carga-masiva` | Sucursales › Crear | igual |
| `/roles/nueva` | Roles y Cuentas de Usuario › Crear | igual |
| `/series-fiscales/nueva` | Series Fiscales › Crear | igual |

#### TC-ADM-093 — Las subrutas de detalle solo exigen Leer y los botones dependen de la acción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-011][hu] «Las subrutas de detalle solo exigen Leer»; [HU-ADM-011][hu] «Los botones dependen de la acción»; [CU-ADM-13][cu] flujo básico (paso 2); [RN-GEN-020, RN-SUC-001, RN-EPL-001][rn] |

**Precondiciones:** cuenta `consulta.demo` (rol «Consulta Demo», solo `Leer`); existen una sucursal y un empleado.
**Datos:** ninguno.
**Pasos:**
1. Abre `/sucursales`, haz clic en «Buscar» y mira los botones.
2. Abre el detalle de una sucursal.
3. Abre `/empleados`, busca y abre la ficha de un empleado.
4. Repite 1 a 3 con `ana.manual`.
**Resultado esperado:** `consulta.demo` ve las fichas en modo ver; en `/sucursales` no ve «Nuevo» ni «Carga Masiva»; en el detalle de sucursal no ve «Editar»; en la ficha de empleado no ve «Editar» ni «Eliminar». Ana ve todos esos botones.

#### TC-ADM-094 — El administrador pasa todas las comprobaciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-011][hu] «El administrador pasa todas las comprobaciones»; [RN-GEN-021][rn] |

**Precondiciones:** sesión de `ana.manual`; abre el rol «Admin» › pestaña «Permisos».
**Datos:** ninguno.
**Pasos:**
1. Mira la matriz de permisos del rol «Admin».
2. Con Ana, abre cualquier pantalla y ejecuta una acción protegida (por ejemplo, crear una sucursal).
**Resultado esperado:** ninguna casilla del rol «Admin» está marcada y, aun así, ninguna comprobación de permiso bloquea a Ana. No verificado en la aplicación: si el nombre del rol con otras mayúsculas (por ejemplo «ADMIN») también pasa; la historia lo afirma («sin distinguir mayúsculas») desde el código.

#### TC-ADM-095 — La API rechaza por permiso aunque el SPA no lo impida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-011][hu] «La API rechaza por permiso aunque el SPA no lo impida»; [CU-ADM-26][cu] excepción (403); [RN-GEN-015, RN-GEN-020, RN-ROL-001][rn] |

**Precondiciones:** Solo API; `<token>` de Carlos (sin `Roles y Cuentas de Usuario`).
**Datos:** `POST /roles` con `{ "nombre": "Rol Prueba 403", "descripcion": "x", "nivelJerarquia": 10 }`.
**Pasos:**
1. Envía la petición.
2. Desde el SPA, haz que la pantalla haga la misma petición (por ejemplo, con una cuenta que vea la pantalla pero sin el permiso, `consulta.demo` en «Guardar» de un rol nuevo, si pudiera llegar).
**Resultado esperado:** la API responde 403 sin campo `code`; la pantalla que hizo la petición muestra su mensaje genérico (por ejemplo «Error al guardar el rol»). No verificado: los nombres exactos de las propiedades del cuerpo (se toman del comando `CreateRolCommand`; confirma con el esquema de la API antes de ejecutar).

#### TC-ADM-096 — Comparación de permisos sin distinguir mayúsculas ni espacios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-011][hu] «Comparación de permisos»; [RN-GEN-020, RN-ROL-013][rn] |

**Precondiciones:** Solo API; rol de prueba asignado a una cuenta de prueba; No verificado en la aplicación: la historia lo describe desde el código. El permiso se asigna con el endpoint de permisos del rol (ver el caso de HU-ADM-030).
**Datos:** al rol de prueba se le asigna el permiso `"seguridad "` (minúsculas y con un espacio) con la acción `Leer` en lugar de `Seguridad`.
**Pasos:**
1. Asigna el permiso con la API.
2. Con el token de la cuenta de prueba (tras volver a iniciar sesión), llama a `GET /roles`.
**Resultado esperado:** la llamada responde 200: la comprobación reconoce el permiso igual que «Seguridad». Hallazgo: la API no valida que el módulo y la acción existan en el catálogo al asignarlos (RN-ROL-013).

#### TC-ADM-097 — Token sin roles o sin empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-011][hu] «Sin roles o sin empresa en el token»; [RN-GEN-015, RN-GEN-020][rn] |

**Precondiciones:** No verificado en la aplicación: un token firmado sin roles o sin el identificador de empresa solo se puede fabricar con la clave de firma, que este documento no usa. El comportamiento se respalda en el código y en las pruebas automáticas ([09a][p09a]).
**Datos:** token firmado sin claims de rol o sin claim de empresa.
**Pasos:**
1. Si el equipo técnico dispone de un token así, llama a `GET /empleados`.
**Resultado esperado:** la API responde 403.

#### TC-ADM-098 — Falla la carga del perfil al entrar a una pantalla protegida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-011][hu] «Falla la carga del perfil al entrar a una pantalla»; [RN-AUT-023][rn] |

**Precondiciones:** sesión iniciada recién abierta (sin perfil cargado); bloqueo de `/auth/me` en las herramientas del navegador.
**Datos:** dirección `/empleados`.
**Pasos:**
1. Con el bloqueo activo, abre `/empleados` escribiendo la dirección.
**Resultado esperado:** aviso «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.» y vuelves al Dashboard.

### HU-ADM-012 — Cambiar el idioma de la interfaz

#### TC-ADM-099 — Cambiar a inglés

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-012][hu] «Cambiar a inglés»; sin caso de uso propio ([19a][p19a] §4.5) |

**Precondiciones:** sesión de `ana.manual` en cualquier pantalla en español.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en la bandera de la cabecera (texto alternativo «Idioma») y elige «English».
2. Lee el cuadro y haz clic en «Confirmar».
3. Mira el saludo del Dashboard y alguna fecha.
**Resultado esperado:** cuadro con título «Cambiar idioma», mensaje «La interfaz se actualizará al idioma seleccionado. ¿Deseas continuar?» y botones «Cancelar» y «Confirmar»; al confirmar, el menú, la cabecera y la pantalla se traducen de inmediato (el saludo pasa a «Hello, Ana Demo Prueba») y las fechas cambian de formato (en español, día/mes/año).

#### TC-ADM-100 — Cancelar el cambio de idioma y elegir el idioma ya activo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-012][hu] «Cancelar el cambio»; [HU-ADM-012][hu] «Elegir el idioma ya activo» |

**Precondiciones:** sesión iniciada con la interfaz en español.
**Datos:** ninguno.
**Pasos:**
1. Elige «English» en la bandera y, en el cuadro «Cambiar idioma», haz clic en «Cancelar».
2. Elige «Español» en la bandera.
**Resultado esperado:** con «Cancelar» el cuadro se cierra y el idioma no cambia; al elegir el idioma ya activo no pasa nada y no se abre ningún cuadro.

#### TC-ADM-101 — El idioma elegido no se conserva

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-012][hu] «El idioma no se conserva» |

**Precondiciones:** interfaz cambiada a «English». No verificado en la aplicación: viene del inventario [19a][p19a] §4.5.
**Datos:** ninguno.
**Pasos:**
1. Recarga la página.
2. Cambia de nuevo a inglés, cierra sesión y vuelve a entrar.
**Resultado esperado:** en ambos casos la interfaz vuelve a español (el idioma se guarda solo en memoria).

## 4.3 Empresa y suscripción

### HU-ADM-013 — Buscar mi empresa y exportarla a Excel

#### TC-ADM-102 — La tabla no aparece hasta buscar y «Buscar» sin filtros muestra solo mi empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-013][hu] «Los resultados no aparecen hasta buscar»; [HU-ADM-013][hu] «Buscar sin filtros»; [CU-ADM-09][cu] flujo básico (pasos 1 a 5); [RN-EMP-001, RN-EMP-017, RN-EMP-018, RN-GEN-001][rn] |

**Precondiciones:** sesión de `ana.manual`; existen la empresa A y la empresa B.
**Datos:** ninguno.
**Pasos:**
1. Abre `/empresas` (menú Administración › «Empresa»).
2. Mira la pantalla antes de buscar.
3. Haz clic en «Buscar».
**Resultado esperado:** antes de buscar ves la tarjeta «Búsqueda de Empresas» con «Ingresa los criterios de búsqueda» y los filtros «Nombre de la Empresa», «Estado», «Vencimiento desde» y «Vencimiento hasta», sin tabla; tras buscar ves «Resultados (1)» con una fila de columnas «Empresa», «RUC», «Estado» y «Vigente»: «Botica Manual Demo S.A.C.», «20538291746», «Trial Activo» y una marca verde en «Vigente». La empresa B no aparece (las demás se comportan como inexistentes).

#### TC-ADM-103 — Filtrar empresas por nombre sin distinguir mayúsculas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-013][hu] «Filtrar por nombre sin distinguir mayúsculas»; [CU-ADM-09][cu] flujo básico (paso 2); [RN-EMP-017][rn] |

**Precondiciones:** estás en `/empresas`.
**Datos:** «manual» (minúsculas); luego «xyz» (sin coincidencias).
**Pasos:**
1. Escribe «manual» en «Nombre de la Empresa» y presiona Enter.
2. Cambia el texto por «xyz» y vuelve a buscar.
**Resultado esperado:** con «manual» la tabla trae «Botica Manual Demo S.A.C.»; con «xyz» muestra «Resultados (0)».

#### TC-ADM-104 — Filtrar empresas por estado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-013][hu] «Filtrar por estado»; [CU-ADM-09][cu] flujo básico (paso 2) |

**Precondiciones:** la empresa A está en «Trial Activo».
**Datos:** Estado «Trial Activo» y luego «Activo».
**Pasos:**
1. Elige «Trial Activo» en «Estado» y haz clic en «Buscar».
2. Elige «Activo» y vuelve a buscar.
**Resultado esperado:** con «Trial Activo» aparece la empresa; con «Activo» se ve «Resultados (0)» y el paginador dice «Sin resultados». Las opciones del filtro son «Todos», «Trial Activo», «Activo», «Suspendido» y «Cancelado».

#### TC-ADM-105 — Filtrar empresas por vencimiento con el calendario

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-013][hu] «Filtrar por vencimiento»; [CU-ADM-09][cu] flujo básico (paso 2); [RN-GEN-008][rn] |

**Precondiciones:** la empresa A vence en una fecha conocida (ejemplo de la historia: 13/11/2026; usa la que muestre la ficha).
**Datos:** «Vencimiento desde» anterior al vencimiento; después «Vencimiento desde» y «Vencimiento hasta» ambos en una fecha anterior al vencimiento.
**Pasos:**
1. Elige «Vencimiento desde» con el calendario (una fecha antes del vencimiento) y busca.
2. Elige «Vencimiento desde» y «Vencimiento hasta» en la misma fecha anterior y busca.
3. Intenta escribir una fecha a mano en ambos campos.
**Resultado esperado:** en el paso 1 la empresa aparece; en el paso 2 no aparece; los dos campos solo se llenan con el calendario (no se pueden escribir).

#### TC-ADM-106 — Limpiar los filtros y paginar la búsqueda de empresas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-013][hu] «Limpiar»; [HU-ADM-013][hu] «Paginar»; [CU-ADM-09][cu] A1; [RN-GEN-009][rn] |

**Precondiciones:** hay filtros escritos y la tabla visible.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Limpiar».
2. Busca de nuevo y mira el paginador.
**Resultado esperado:** los cuatro filtros quedan vacíos y la tabla se oculta; el paginador ofrece «Items por página» de 10, 25 o 50 y botones de primera, anterior, siguiente y última página.

#### TC-ADM-107 — Exportar la búsqueda de empresas a Excel

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-013][hu] «Exportar a Excel»; [CU-ADM-09][cu] flujo básico (pasos 7 y 8); [RN-IMP-008, RN-IMP-009, RN-IMP-010, RN-IMP-017][rn] |

**Precondiciones:** ya hiciste clic en «Buscar» (el botón «Exportar» no aparece antes). Si cambias un filtro sin buscar, el archivo lleva el filtro actual.
**Datos:** sin filtros.
**Pasos:**
1. Antes de buscar, comprueba que no hay botón «Exportar».
2. Busca y haz clic en «Exportar».
3. Abre el archivo descargado.
**Resultado esperado:** se descarga `empresas.xlsx` con una hoja «Data» y las columnas «Empresa», «RUC», «Estado», «Vigente» («Sí» o «No»), «Inicio Trial», «Vencimiento» (con formato dd/MM/yyyy) y «Sucursales» (cantidad). No verificado en la aplicación: el contenido del archivo; según el documento [20][p20] §4.4 la columna «Estado» trae el nombre interno sin traducir (por ejemplo `TrialActivo`); registra lo que veas.

#### TC-ADM-108 — Exportación demasiado grande y error al buscar empresas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-013][hu] «Exportación demasiado grande»; [HU-ADM-013][hu] «Error al buscar»; [CU-ADM-09][cu] excepciones pasos 3 y 8; [RN-IMP-017][rn] |

**Precondiciones:** No verificado en la aplicación para las más de 10 000 filas: la búsqueda de empresas solo devuelve la empresa propia, por lo que no se puede superar el límite desde este módulo (el límite se prueba con sucursales o empleados, ver el caso de HU-ADM-047).
**Datos:** bloqueo de las llamadas `GET /empresas` y `POST /empresas/exportar` en las herramientas del navegador.
**Pasos:**
1. Con el bloqueo activo, haz clic en «Buscar».
2. Quita el bloqueo, busca, vuelve a bloquear solo `exportar` y haz clic en «Exportar».
**Resultado esperado:** en ambos casos aparece el aviso «Error al buscar empresas» (la exportación reutiliza esa clave). Con más de 10 000 filas la API respondería 400 `Export.TooManyRows` y se vería el mismo aviso.

### HU-ADM-014 — Ver los datos y la suscripción de mi empresa

#### TC-ADM-109 — Ver el detalle de la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-014][hu] «Ver el detalle»; [CU-ADM-09][cu] flujo básico (paso 6); [RN-EMP-012, RN-EMP-016, RN-EMP-017][rn] |

**Precondiciones:** sesión de `ana.manual`; la empresa A tiene una sola sucursal.
**Datos:** ninguno.
**Pasos:**
1. Busca la empresa y haz clic en «Botica Manual Demo S.A.C.».
**Resultado esperado:** se abre `/empresas/…` con la flecha de volver, el título «Empresas» y la tarjeta «Datos del registro»; ves deshabilitados «Estado» («Trial Activo»), «Sucursales» (cantidad de sucursales no eliminadas), «Inicio Trial», «Vencimiento», «Nombre de la Empresa» y «RUC (11 dígitos, opcional)». Hallazgo: el detalle no muestra los días restantes ni el indicador «Vigente» (Brecha conocida de HU-ADM-014); la política «exigir registro de receta» no tiene pantalla.

#### TC-ADM-110 — Una empresa nueva nace con 30 días de prueba

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-014][hu] «Una empresa nueva nace con 30 días de prueba»; [CU-ADM-01][cu] flujo básico (paso 6); [RN-EMP-012][rn] |

**Precondiciones:** una empresa registrada hoy (TC-ADM-001).
**Datos:** fecha de registro conocida.
**Pasos:**
1. Abre el detalle de la empresa.
2. Compara «Inicio Trial» y «Vencimiento».
**Resultado esperado:** «Estado» es «Trial Activo», «Inicio Trial» es la fecha del registro y «Vencimiento» es 30 días después (ejemplo: registro el 14/10/2026, vencimiento 13/11/2026).

#### TC-ADM-111 — Estado, fechas y sucursales no se editan

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-014][hu] «Estado, fechas y sucursales no se editan»; [CU-ADM-09][cu] A3; [RN-EMP-015][rn] |

**Precondiciones:** sesión de Ana en el detalle de la empresa.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Editar».
2. Observa qué campos se habilitan y si existe «Eliminar» o una forma de crear empresas.
**Resultado esperado:** solo se habilitan «Nombre de la Empresa» y «RUC (11 dígitos, opcional)»; «Estado», «Sucursales», «Inicio Trial» y «Vencimiento» nunca se habilitan; no existe el botón «Eliminar» ni una pantalla para crear empresas.

#### TC-ADM-112 — Solo veo mi propia empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-014][hu] «Solo veo mi propia empresa»; [CU-ADM-09][cu] excepción paso 6; [RN-EMP-017, RN-GEN-001][rn] |

**Precondiciones:** conoces el identificador de la empresa B (por ejemplo, desde la sesión de `luis.dos` o la base de datos); sesión de Ana (empresa A).
**Datos:** `GET /empresas/<id de B>` con `<token>` de Ana; en el SPA, la dirección `/empresas/<id de B>`.
**Pasos:**
1. Llama a la API con el token de Ana.
2. Abre la dirección en el SPA con la sesión de Ana.
**Resultado esperado:** la API responde 404 `Empresa.NotFound`; el SPA muestra «Error al cargar la empresa».

#### TC-ADM-113 — Error al cargar el detalle de la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-014][hu] «Error al cargar»; [CU-ADM-09][cu] excepción paso 6 |

**Precondiciones:** bloqueo de la llamada `GET /empresas/<id>` en las herramientas del navegador.
**Datos:** ninguno.
**Pasos:**
1. Abre el detalle de la empresa con el bloqueo activo.
**Resultado esperado:** aviso «Error al cargar la empresa».

### HU-ADM-015 — Editar el nombre y el RUC de mi empresa

#### TC-ADM-114 — Editar y guardar el nombre de la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-015][hu] «Editar y guardar»; [CU-ADM-10][cu] flujo básico; [RN-EMP-001, RN-EMP-002, RN-EMP-017, RN-GEN-013][rn] |

**Precondiciones:** `ana.manual` en el detalle de la empresa; acceso de solo lectura a `Auditoria_Eventos_Seguridad` (opcional).
**Datos:** Nombre de la Empresa «Botica Manual Demo S.A.C. (editada)».
**Pasos:**
1. Haz clic en «Editar».
2. Cambia el nombre y haz clic en «Guardar».
3. Consulta el evento de auditoría.
4. Restaura el nombre original.
**Resultado esperado:** se habilitan «Nombre de la Empresa» y «RUC (11 dígitos, opcional)» y aparecen «Cancelar» y «Guardar»; tras guardar ves «Empresa actualizada correctamente» y la pantalla vuelve al modo ver, recargada, sin ir a la búsqueda; se registra el evento `EMPRESA_ACTUALIZAR` sin incluir el nombre ni el RUC.

#### TC-ADM-115 — Sin el permiso Actualizar no hay botón «Editar»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-015][hu] «Sin el permiso no hay botón»; [CU-ADM-10][cu] A2; [RN-EMP-001][rn] |

**Precondiciones:** cuenta `consulta.demo` (solo `Empresas › Leer`).
**Datos:** ninguno.
**Pasos:**
1. Abre el detalle de la empresa.
2. Con cliente HTTP, llama a `PUT /empresas/<id>` con el token de esa cuenta y `{ "nombre": "X", "ruc": "20538291746" }`.
**Resultado esperado:** no ves el botón «Editar»; la API responde 403 sin `code`.

#### TC-ADM-116 — Cancelar descarta los cambios de la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-015][hu] «Cancelar descarta los cambios»; [CU-ADM-10][cu] A1 |

**Precondiciones:** detalle de la empresa en modo editar.
**Datos:** otro nombre y otro RUC.
**Pasos:**
1. Escribe otro nombre y otro RUC.
2. Haz clic en «Cancelar».
**Resultado esperado:** el nombre y el RUC vuelven a los que tenía la empresa y la pantalla vuelve al modo ver; no se guarda nada.

#### TC-ADM-117 — Nombre de la empresa obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-015][hu] «Nombre obligatorio»; [CU-ADM-10][cu] excepción paso 3; [RN-EMP-002][rn] |

**Precondiciones:** modo editar.
**Datos:** «Nombre de la Empresa» vacío.
**Pasos:**
1. Borra el nombre y haz clic en «Guardar».
**Resultado esperado:** «Este campo es obligatorio» debajo del campo; no se envía nada.

#### TC-ADM-118 — Longitud del nombre de la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-015][hu] «Longitud del nombre»; [CU-ADM-10][cu] flujo básico (paso 2); [RN-EMP-002][rn] |

**Precondiciones:** modo editar.
**Datos:** texto de 262 caracteres.
**Pasos:**
1. Pega o escribe 262 caracteres en «Nombre de la Empresa».
2. Cuenta los caracteres que quedan.
**Resultado esperado:** el campo se detiene en 150 caracteres. Hallazgo: el formulario limita a 150 y la API admite 200 (Brecha conocida de HU-ADM-015).

#### TC-ADM-119 — RUC con formato inválido al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-015][hu] «RUC con formato inválido»; [CU-ADM-10][cu] excepción paso 5; [RN-EMP-004][rn] |

**Precondiciones:** modo editar.
**Datos:** RUC de 10 dígitos (`2053829174`); otro con una letra entre 11 caracteres (`2053829174A`).
**Pasos:**
1. Escribe cada RUC y haz clic en «Guardar».
**Resultado esperado:** en ambos casos la API lo rechaza (400 `UpdateEmpresaCommand.RUC`) y ves «Error al actualizar la empresa» sin más explicación; el RUC no cambia. Hallazgo: el formulario no valida el formato (solo 20 caracteres) pese al rótulo «(11 dígitos, opcional)» (Brecha conocida de HU-ADM-015).

#### TC-ADM-120 — RUC de otra empresa al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-015][hu] «RUC de otra empresa»; [CU-ADM-10][cu] excepción paso 6; [RN-EMP-005][rn] |

**Precondiciones:** la empresa B tiene el RUC `20123456789`; sesión de Ana (empresa A).
**Datos:** RUC `20123456789`.
**Pasos:**
1. Escribe ese RUC y haz clic en «Guardar».
**Resultado esperado:** la API responde 409 `Empresa.RUC.Duplicado`; ves «Error al actualizar la empresa» (la pantalla no traduce el código); la empresa conserva su RUC.

#### TC-ADM-121 — RUC vacío al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-015][hu] «RUC vacío»; [CU-ADM-10][cu] excepción paso 5; [RN-EMP-004][rn] |

**Precondiciones:** modo editar.
**Datos:** campo «RUC (11 dígitos, opcional)» vacío.
**Pasos:**
1. Borra el RUC y haz clic en «Guardar».
2. Vuelve a abrir el detalle.
**Resultado esperado:** el SPA envía un texto vacío, la API lo rechaza y ves «Error al actualizar la empresa»; la empresa conserva su RUC. Hallazgo: desde la pantalla no se puede dejar la empresa sin RUC aunque la API acepta un RUC nulo (RN-EMP-004; No verificado en la aplicación porque la pantalla no puede enviarlo).

#### TC-ADM-122 — Solo se editan nombre y RUC (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-015][hu] «Solo se editan nombre y RUC»; [CU-ADM-10][cu] flujo básico (paso 7) |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `PUT /empresas/<id de A>` con `{ "nombre": "Botica Manual Demo S.A.C.", "ruc": "20538291746", "estado": "Activo", "fechaVencimiento": "2030-01-01" }`.
**Pasos:**
1. Envía la petición.
2. Abre el detalle de la empresa en el SPA.
**Resultado esperado:** la API responde 204 y solo se aplican nombre y RUC; «Estado» sigue en «Trial Activo» y «Vencimiento» no cambia.

#### TC-ADM-123 — Otra empresa no se puede editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-015][hu] «Otra empresa no se puede editar»; [CU-ADM-10][cu] excepción paso 6; [RN-EMP-017, RN-GEN-001][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; identificador de la empresa B.
**Datos:** `PUT /empresas/<id de B>` con `{ "nombre": "Hackeada", "ruc": "20123456789" }`.
**Pasos:**
1. Envía la petición.
2. Con la sesión de `luis.dos` comprueba el nombre de su empresa.
**Resultado esperado:** la API responde 404 `Empresa.NotFound`; el nombre de la empresa B no cambia.

### HU-ADM-016 — Restringir el acceso cuando la suscripción no está vigente

#### TC-ADM-124 — Cuándo una empresa está vigente y cuándo no

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-016][hu] «Cuándo una empresa está vigente»; [HU-ADM-016][hu] «Cuándo no está vigente»; [HU-ADM-016][hu] «En estado Activo la fecha de vencimiento no se evalúa»; [CU-ADM-02][cu] excepción paso 6; [RN-EMP-014, RN-EMP-015][rn] |

**Precondiciones:** empresas de prueba que se llevan al estado de cada fila por base de datos (ningún endpoint ni pantalla cambia el estado: Pendiente de producto). No verificado en la aplicación.
**Datos:** una fila por vez; cuenta de la empresa con contraseña correcta.
**Pasos:**
1. Para cada fila, deja la empresa en el estado y vencimiento indicados.
2. Intenta iniciar sesión.
**Resultado esperado:**

| Estado | Vencimiento | ¿Vigente? | Resultado al iniciar sesión |
|---|---|---|---|
| Trial Activo | posterior a la hora actual | Sí | entra |
| Trial Activo | ya pasó (prueba vencida) | No | «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» |
| Activo | posterior | Sí | entra |
| Activo | ya pasó | Sí (la fecha no se evalúa) | entra |
| Suspendido | cualquiera | No | el mismo aviso de suscripción |
| Cancelado | cualquiera | No | el mismo aviso de suscripción |
| Empresa eliminada | cualquiera | No | «Credenciales inválidas. Verifica tu usuario y contraseña.» (la cuenta ya no se encuentra) |

La última fila es un comportamiento esperado derivado de la historia («la empresa no existe → no vigente»); su mensaje exacto es No verificado.

#### TC-ADM-125 — Las sesiones vivas se cortan en un minuto como máximo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-016][hu] «Las sesiones vivas se cortan en un minuto como máximo»; [CU-ADM-08][cu] excepción paso 5; [RN-EMP-014, RN-GEN-018][rn] |

**Precondiciones:** sesión abierta de una cuenta de una empresa de prueba vigente; No verificado en la aplicación: requiere cambiar el estado de la empresa por base de datos con la sesión abierta.
**Datos:** cambio de estado de la empresa a «Suspendido».
**Pasos:**
1. Deja la sesión abierta y suspende la empresa por base de datos.
2. Espera hasta un minuto y navega a otra pantalla.
**Resultado esperado:** pasado como máximo 1 minuto (la vigencia está en caché por empresa), las peticiones reciben 401 sin `code` y el sistema te lleva a «Iniciar sesión».

#### TC-ADM-126 — No se envía el correo de recuperación si la empresa no está vigente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-016][hu] «No se envía el correo de recuperación»; [CU-ADM-06][cu] A1; [RN-AUT-034][rn] |

**Precondiciones:** empresa de prueba no vigente; Mailpit disponible; la cuenta con correo.
**Datos:** usuario de esa empresa.
**Pasos:**
1. En `/auth/forgot-password` escribe el usuario y haz clic en «Enviar enlace».
2. Revisa Mailpit.
**Resultado esperado:** ves el mensaje uniforme de la alerta verde y no llega ningún correo.

## 4.4 Sucursales

### HU-ADM-017 — Buscar sucursales y exportarlas a Excel

#### TC-ADM-127 — Buscar sucursales sin filtros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-017][hu] «Buscar sin filtros»; [CU-ADM-11][cu] flujo básico (pasos 1 a 5); [RN-SUC-001, RN-SUC-011, RN-GEN-001][rn] |

**Precondiciones:** la empresa A tiene «Sede Principal» y «Sucursal Demo Norte»; la empresa B tiene sus propias sucursales.
**Datos:** ninguno.
**Pasos:**
1. Abre `/sucursales`; comprueba que no hay tabla.
2. Haz clic en «Buscar».
**Resultado esperado:** «Resultados (2)» ordenadas por nombre con las columnas «Nombre», «Nº de Licencia», «Dirección Física» y «Gerente»; el gerente se muestra como «—» cuando no hay; las sucursales de la empresa B no aparecen.

#### TC-ADM-128 — Filtros de sucursales que se combinan

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-017][hu] «Filtros que se combinan»; [CU-ADM-11][cu] flujo básico (paso 2) |

**Precondiciones:** «Sucursal Demo Norte» con licencia `LIC-DEMO-0002` y dirección «Av. Ficticia Norte 456, Los Olivos, Lima».
**Datos:** «Nº de Licencia» `demo-0002` (minúsculas); «Dirección Física» `miraflores`.
**Pasos:**
1. Escribe `demo-0002` y haz clic en «Buscar».
2. Agrega `miraflores` en «Dirección Física» y busca.
3. Quita la dirección y busca con Enter dentro de cada uno de los tres filtros.
**Resultado esperado:** con solo la licencia ves únicamente «Sucursal Demo Norte»; con ambos filtros ves «Resultados (0)» y «Sin resultados»; Enter dentro de cualquiera de los tres filtros también busca.

#### TC-ADM-129 — Limpiar la búsqueda de sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-017][hu] «Limpiar»; [CU-ADM-11][cu] A1 |

**Precondiciones:** hay filtros y tabla.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Limpiar».
**Resultado esperado:** los tres filtros quedan vacíos y la tabla se oculta.

#### TC-ADM-130 — Paginación de sucursales y restauración al volver desde un detalle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-017][hu] «Paginación»; [CU-ADM-11][cu] A2; [RN-GEN-009][rn] |

**Precondiciones:** hay al menos 12 sucursales (crea sucursales con la carga masiva, TC-ADM-160) para ver más de una página.
**Datos:** filtros cualesquiera.
**Pasos:**
1. Busca y observa el paginador.
2. Pasa a la página 2, abre el detalle de una sucursal y vuelve con la flecha.
3. Recarga el navegador.
**Resultado esperado:** el paginador empieza en 10 por página y ofrece 10, 25 y 50; al volver desde un detalle se restauran filtros y página; tras recargar el navegador ya no se restauran.

#### TC-ADM-131 — Límites de paginación de la API de sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-017][hu] «Límites de paginación de la API»; [CU-ADM-11][cu] flujo básico (paso 4); [RN-GEN-009][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `GET /sucursales?pageNumber=0&pageSize=10`; `GET /sucursales?pageNumber=1&pageSize=0`; `GET /sucursales?pageNumber=1&pageSize=20000`; `GET /sucursales?pageNumber=999999&pageSize=10`.
**Pasos:**
1. Envía cada petición y mira los campos de página y la lista de la respuesta.
**Resultado esperado:** `pageNumber` menor que 1 se trata como 1; `pageSize` se ajusta al rango de 1 a 10 000; un número de página enorme devuelve una página vacía en lugar de fallar (200).

#### TC-ADM-132 — Exportar sucursales a Excel

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-017][hu] «Exportar»; [CU-ADM-11][cu] flujo básico (pasos 6 y 7); [RN-IMP-008, RN-IMP-009, RN-IMP-017][rn] |

**Precondiciones:** hiciste clic en «Buscar».
**Datos:** filtro «Nº de Licencia» `demo` escrito justo antes de exportar, sin volver a buscar.
**Pasos:**
1. Haz clic en «Exportar».
2. Abre el archivo.
**Resultado esperado:** se descarga `sucursales.xlsx` con una hoja «Data» y las columnas «Nombre», «Dirección Física», «Nº de Licencia» y «Gerente» (vacío si no hay); se aplican los filtros escritos en ese momento. No verificado en la aplicación: el contenido del archivo (manual cap. 2, §6.3).

#### TC-ADM-133 — Exportación de más de 10 000 sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-017][hu] «Exportación de más de 10 000 filas»; [CU-ADM-11][cu] excepción paso 7; [RN-IMP-017][rn] |

**Precondiciones:** No verificado en la aplicación: se necesitan más de 10 000 sucursales en la empresa; solo se alcanza con una carga de datos por script. Ver el caso común de HU-ADM-047.
**Datos:** empresa de prueba con 10 001 sucursales.
**Pasos:**
1. Busca y haz clic en «Exportar».
**Resultado esperado:** la API responde 400 `Export.TooManyRows` y no genera el archivo; ves «Error al buscar sucursales».

#### TC-ADM-134 — Solo veo las sucursales de mi empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-017][hu] «Solo veo las sucursales de mi empresa»; [CU-ADM-11][cu] flujo básico (paso 4); [RN-GEN-001][rn] |

**Precondiciones:** la empresa B tiene una sucursal «Sede B Prueba»; sesión de Ana (empresa A).
**Datos:** `GET /sucursales/<id de la sucursal de B>` con el token de Ana, y una búsqueda por nombre «Sede B».
**Pasos:**
1. Busca «Sede B» en `/sucursales`.
2. Llama a la API con el identificador de la sucursal de B.
**Resultado esperado:** «Resultados (0)»; la API responde 404 `Sucursal.NotFound`.

### HU-ADM-018 — Crear una sucursal

#### TC-ADM-135 — Crear una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-018][hu] «Crear la sucursal»; [CU-ADM-12][cu] flujo básico; [RN-SUC-001, RN-SUC-002, RN-SUC-003, RN-SUC-004, RN-SUC-007, RN-GEN-024][rn] |

**Precondiciones:** sesión de Ana en `/sucursales/nueva` (botón «Nuevo»).
**Datos:** Nombre «Sucursal Demo Norte»; Nº de Licencia `LIC-DEMO-0002`; Dirección Física «Av. Ficticia Norte 456, Los Olivos, Lima»; «Gerente» en blanco.
**Pasos:**
1. Completa los campos y haz clic en «Guardar».
2. Busca la sucursal en `/sucursales`.
**Resultado esperado:** aviso «Sucursal guardada correctamente»; el sistema te lleva a `/sucursales` con los filtros vacíos y sin tabla; la sucursal pertenece a la empresa A (no aparece en la empresa B).

#### TC-ADM-136 — Campos obligatorios al crear una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-018][hu] «Campos obligatorios»; [CU-ADM-12][cu] excepción paso 2; [RN-SUC-002, RN-SUC-003, RN-SUC-004][rn] |

**Precondiciones:** `/sucursales/nueva` con el formulario vacío.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Guardar».
**Resultado esperado:** «Nombre», «Nº de Licencia» y «Dirección Física» muestran «Este campo es obligatorio»; no se envía nada.

#### TC-ADM-137 — Longitudes máximas al crear una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-018][hu] «Longitudes máximas»; [CU-ADM-12][cu] flujo básico (paso 1); [RN-SUC-002, RN-SUC-003, RN-SUC-004][rn] |

**Precondiciones:** `/sucursales/nueva`.
**Datos:** 120 caracteres en Nombre; 70 en Nº de Licencia; 300 en Dirección Física.
**Pasos:**
1. Pega cada texto en su campo y cuenta lo que queda.
**Resultado esperado:** «Nombre» se detiene en 100, «Nº de Licencia» en 50 y «Dirección Física» en 255.

#### TC-ADM-138 — Licencia repetida en la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-018][hu] «Licencia repetida en la empresa»; [CU-ADM-12][cu] excepción paso 4; [RN-SUC-005][rn] |

**Precondiciones:** la «Sede Principal» ya tiene la licencia `LIC-DEMO-0001` (TC-ADM-147).
**Datos:** Nombre «Sucursal Repetida»; licencia `LIC-DEMO-0001`, y otra vez con `lic-demo-0001` en minúsculas.
**Pasos:**
1. Completa el formulario con cada licencia y haz clic en «Guardar».
**Resultado esperado:** aviso rojo «Ya existe una sucursal con ese número de licencia.»; te quedas en la pantalla con lo que escribiste; la licencia es única entre sucursales no eliminadas de la empresa, sin distinguir mayúsculas.

#### TC-ADM-139 — Dos altas simultáneas con la misma licencia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-018][hu] «Dos altas simultáneas con la misma licencia»; [CU-ADM-12][cu] excepción paso 6 (`Db.Duplicado`); [RN-GEN-036][rn] |

**Precondiciones:** cliente HTTP capaz de enviar dos peticiones a la vez. No verificado en la aplicación: la carrera es difícil de repetir.
**Datos:** dos `POST /sucursales` con la misma licencia nueva `LIC-CARRERA-01` y nombres distintos.
**Pasos:**
1. Envía las dos peticiones a la vez y repite si ambas pasan.
**Resultado esperado:** una responde 201 y la otra 409 (`Sucursal.NumeroLicencia.Duplicado` o `Db.Duplicado`); cuando es `Db.Duplicado` el SPA muestra «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.»; solo existe una sucursal con esa licencia.

#### TC-ADM-140 — Valor fuera de rango o referencia inexistente al guardar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-018][hu] «Valor fuera de rango o referencia inexistente»; [HU-ADM-019][hu] «Errores de base de datos al guardar»; [CU-ADM-12][cu] excepción paso 6; [CU-ADM-13][cu] excepción paso 7; [RN-GEN-037, RN-GEN-038][rn] |

**Precondiciones:** No verificado en la aplicación: no hay datos que provoquen una restricción CHECK o una clave foránea desde la pantalla; se respaldan en el código (traducción de errores de infraestructura).
**Datos:** gerente eliminado entre elegirlo y guardar (para la clave foránea).
**Pasos:**
1. En `/sucursales/nueva` elige un gerente de la lista.
2. En otra sesión elimina a ese empleado (si no tiene dependencias).
3. Guarda la sucursal.
**Resultado esperado:** según la restricción: «Uno de los valores está fuera del rango permitido.» o «El registro hace referencia a datos que ya no existen o todavía está en uso.»; si el empleado ya no existe en la empresa, el servidor puede responder antes 404 `Empleado.NotFound` y el aviso genérico «Error al guardar la sucursal». Registra cuál ocurre.

#### TC-ADM-141 — Elegir un gerente de la lista

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-018][hu] «Elegir un gerente»; [CU-ADM-12][cu] flujo básico (paso 1); [RN-SUC-006][rn] |

**Precondiciones:** Ana tiene `Empleados › Leer`; existe el empleado «Ana Demo Prueba».
**Datos:** texto «Ana» en «Gerente»; después un texto sin coincidencia («Zzz») y salir del campo.
**Pasos:**
1. Escribe «A» (una letra) y observa; escribe «Ana».
2. Elige «Ana Demo Prueba» y guarda la sucursal.
3. En un formulario nuevo escribe «Zzz» y sal del campo.
**Resultado esperado:** con una sola letra no hay lista; con «Ana», tras medio segundo, la lista muestra «Ana Demo Prueba»; solo se puede elegir uno de la lista: el texto que no corresponde a un empleado se vacía al salir del campo. No verificado en la aplicación: guardar un gerente elegido (el manual no lo probó).

#### TC-ADM-142 — El gerente debe ser un empleado de la empresa (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-018][hu] «El gerente debe ser un empleado de la empresa»; [CU-ADM-12][cu] excepción paso 5; [RN-SUC-006, RN-GEN-001][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; identificador de un empleado de la empresa B.
**Datos:** `POST /sucursales` con `{ "nombre": "Sucursal Gerente Ajeno", "direccionFisica": "Calle Ficticia 1", "numeroLicencia": "LIC-GER-0001", "gerenteId": "<id del empleado de B>" }`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 404 `Empleado.NotFound`; no se crea la sucursal; si se hiciera desde el SPA se vería «Error al guardar la sucursal».

#### TC-ADM-143 — El token no trae empresa al crear una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-018][hu] «El token no trae empresa»; [CU-ADM-12][cu] excepción paso 4; [RN-SUC-007, RN-GEN-024][rn] |

**Precondiciones:** No verificado en la aplicación: requiere un token firmado sin el identificador de empresa, que solo se fabrica con la clave de firma.
**Datos:** ninguno.
**Pasos:**
1. Si el equipo técnico dispone de un token así, envía `POST /sucursales` con un cuerpo válido.
**Resultado esperado:** la API responde 400 `Auth.Empresa`; el SPA mostraría «Error al guardar la sucursal».

#### TC-ADM-144 — Cancelar la creación de una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-018][hu] «Cancelar»; [CU-ADM-12][cu] A1 |

**Precondiciones:** `/sucursales/nueva` con datos escritos.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
2. Busca la sucursal.
**Resultado esperado:** vuelves a `/sucursales` sin guardar nada.

### HU-ADM-019 — Ver y editar una sucursal

#### TC-ADM-145 — Ver el detalle de una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-019][hu] «Ver el detalle»; [CU-ADM-13][cu] flujo básico (paso 1); [RN-SUC-008][rn] |

**Precondiciones:** `ana.manual`; también `consulta.demo` (sin `Actualizar`).
**Datos:** ninguno.
**Pasos:**
1. Busca y haz clic en «Sede Principal».
2. Repite con `consulta.demo`.
**Resultado esperado:** ves el título «Sucursales», «Datos del registro» y los campos «Nombre», «Nº de Licencia», «Dirección Física» y «Gerente» deshabilitados; el botón «Editar» aparece solo con «Sucursales › Actualizar».

#### TC-ADM-146 — La sede principal creada por el registro nace sin licencia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-019][hu] «La sede principal creada por el registro nace sin licencia»; [CU-ADM-01][cu] A3; [CU-ADM-13][cu] flujo básico (paso 3); [RN-EMP-010, RN-SUC-004][rn] |

**Precondiciones:** empresa recién registrada (TC-ADM-001).
**Datos:** ninguno.
**Pasos:**
1. Busca las sucursales y mira la celda «Nº de Licencia» de «Sede Principal».
2. Abre el detalle.
**Resultado esperado:** la tabla muestra la celda de licencia en blanco (no «—») y el detalle tiene «Nº de Licencia» vacío; la licencia es obligatoria al editar.

#### TC-ADM-147 — Completar la licencia de la sede principal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-019][hu] «Completar la licencia de la sede principal»; [CU-ADM-13][cu] flujo básico (pasos 3 a 7); [RN-SUC-004, RN-EMP-010][rn] |

**Precondiciones:** «Sede Principal» sin licencia; Ana con `Sucursales › Actualizar`.
**Datos:** `LIC-DEMO-0001`.
**Pasos:**
1. Abre el detalle y haz clic en «Editar».
2. Haz clic en «Guardar» con «Nº de Licencia» vacío.
3. Escribe `LIC-DEMO-0001` y haz clic en «Guardar».
**Resultado esperado:** en el paso 2 ves «Este campo es obligatorio» debajo de «Nº de Licencia» y no se guarda nada; en el paso 3 ves «Sucursal guardada correctamente» y la pantalla vuelve al modo ver en el mismo detalle con la licencia `LIC-DEMO-0001`.

#### TC-ADM-148 — Editar muestra los tres botones según los permisos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-ADM-019][hu] «Editar muestra los tres botones»; [CU-ADM-13][cu] flujo básico (paso 2); [RN-SUC-001][rn] |

**Precondiciones:** una cuenta con `Sucursales › Leer` y `Actualizar` pero no `Eliminar`; `ana.manual`.
**Datos:** ninguno.
**Pasos:**
1. En el detalle de una sucursal existente haz clic en «Editar» con cada cuenta.
**Resultado esperado:** Ana ve «Cancelar», «Guardar» y «Eliminar»; la cuenta sin `Eliminar` ve «Cancelar» y «Guardar» y no ve «Eliminar».

#### TC-ADM-149 — Cancelar la edición de una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-019][hu] «Cancelar la edición»; [CU-ADM-13][cu] A1 |

**Precondiciones:** sucursal con gerente; modo editar.
**Datos:** otro nombre y otro gerente.
**Pasos:**
1. Cambia el nombre y el gerente.
2. Haz clic en «Cancelar».
**Resultado esperado:** se restauran los datos leídos (incluido el gerente) y vuelves al modo ver.

#### TC-ADM-150 — Licencia usada por otra sucursal al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-019][hu] «Licencia usada por otra sucursal»; [CU-ADM-13][cu] excepción paso 6; [RN-SUC-005][rn] |

**Precondiciones:** «Sucursal Demo Norte» tiene `LIC-DEMO-0002`; «Sede Principal» en modo editar.
**Datos:** `LIC-DEMO-0002` en la «Sede Principal».
**Pasos:**
1. Escribe esa licencia y haz clic en «Guardar».
2. Escribe la licencia de la misma sucursal sin cambios y guarda.
**Resultado esperado:** con la licencia ajena, aviso «Ya existe una sucursal con ese número de licencia.»; guardar sin cambiar la licencia propia funciona (no choca consigo misma).

#### TC-ADM-151 — Cambiar o quitar el gerente de una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-019][hu] «Cambiar o quitar el gerente»; [CU-ADM-13][cu] flujo básico (pasos 3 y 5); [RN-SUC-006][rn] |

**Precondiciones:** una sucursal con gerente asignado.
**Datos:** vaciar el campo «Gerente»; otra vez sin tocarlo.
**Pasos:**
1. Edita, vacía «Gerente» y guarda; revisa la tabla.
2. Asigna un gerente, guarda; edita otro campo sin tocar «Gerente» y guarda; revisa.
**Resultado esperado:** al vaciar, la sucursal queda sin gerente (la tabla muestra «—»); si no tocas el gerente se reenvía el que ya tenía y se conserva. No verificado en la aplicación: guardar un gerente elegido y que un empleado gerencie dos sucursales (historia, nota final).

#### TC-ADM-152 — Errores genéricos al guardar o abrir una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-019][hu] «Errores genéricos»; [CU-ADM-13][cu] excepciones pasos 1 y 6; [RN-SUC-008][rn] |

**Precondiciones:** Solo API para el 404; bloqueo de red para el aviso.
**Datos:** `GET /sucursales/<id inexistente>`; `PUT /sucursales/<id>` con un `gerenteId` inexistente.
**Pasos:**
1. En el SPA abre `/sucursales/<id inexistente>`.
2. Guarda una sucursal con un gerente que no exista en la empresa (API).
**Resultado esperado:** «Error al cargar la sucursal» (404 `Sucursal.NotFound`, incluida una sucursal de otra empresa) y «Error al guardar la sucursal» (404 `Empleado.NotFound`).

#### TC-ADM-153 — Buscar gerente sin permiso sobre empleados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-ADM-019][hu] «Buscar gerente sin permiso sobre empleados»; [CU-ADM-12][cu] precondiciones; [RN-EPL-001][rn] |

**Precondiciones:** una cuenta con `Sucursales › Crear` y `Actualizar` pero sin `Empleados › Leer`. No verificado en la aplicación: no se probó en el módulo de sucursales ([19b][p19b] §2.2).
**Datos:** texto «Ana» en «Gerente».
**Pasos:**
1. Abre una sucursal en modo editar y escribe en «Gerente».
**Resultado esperado:** la consulta falla en silencio y el buscador no ofrece sugerencias.

### HU-ADM-020 — Eliminar una sucursal

#### TC-ADM-154 — Eliminar una sucursal sin registros asociados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-020][hu] «Eliminar una sucursal sin registros asociados»; [CU-ADM-14][cu] flujo básico; [RN-SUC-008, RN-SUC-009, RN-GEN-003][rn] |

**Precondiciones:** sucursal «Sucursal Prueba Eliminar» recién creada, sin empleados, cuentas, roles, stock, sesiones de caja, series fiscales, ventas, transferencias, cuarentena DIGEMID ni órdenes magistrales.
**Datos:** ninguno.
**Pasos:**
1. Abre su detalle, haz clic en «Editar» y luego en «Eliminar».
2. Lee el cuadro y haz clic en «Confirmar».
3. Busca la sucursal.
**Resultado esperado:** cuadro «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta sucursal?» y botones «Cancelar» y «Confirmar»; al confirmar ves «Sucursal eliminada correctamente» y vuelves a `/sucursales`; al volver a buscar la sucursal ya no aparece (eliminación lógica).

#### TC-ADM-155 — Cancelar la confirmación de eliminar una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-020][hu] «Cancelar la confirmación»; [CU-ADM-14][cu] A1 |

**Precondiciones:** cuadro «Confirmar eliminación» abierto.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** no se borra nada y sigues en modo editar.

#### TC-ADM-156 — No se elimina una sucursal con registros asociados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-020][hu] «La sucursal tiene registros asociados»; [CU-ADM-14][cu] excepción paso 4; [RN-SUC-009][rn] |

**Precondiciones:** la «Sede Principal» tiene empleados y cuentas (al menos el administrador).
**Datos:** ninguno.
**Pasos:**
1. Abre la «Sede Principal» en modo editar, haz clic en «Eliminar» y luego en «Confirmar».
2. Repite con una sucursal que solo tenga un rol asignado, una serie fiscal, etc., una dependencia por vez (opcional).
**Resultado esperado:** la API responde 409 `Sucursal.InUse` y no borra nada; aviso rojo «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.»; te quedas en modo editar. Hallazgo: el aviso no dice cuál de los registros bloquea; el SPA decide por el estado 409, de modo que cualquier otro 409 (por ejemplo `Concurrency.Conflict`) mostraría el mismo texto (Brecha conocida de HU-ADM-020).

#### TC-ADM-157 — Otros errores al eliminar una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-020][hu] «Otros errores al eliminar»; [CU-ADM-14][cu] excepciones pasos 2 y 3; [RN-SUC-008][rn] |

**Precondiciones:** dos sesiones; Ana abre el detalle de una sucursal sin dependencias en modo editar.
**Datos:** otra sesión elimina antes esa sucursal.
**Pasos:**
1. En la otra sesión elimina la sucursal.
2. En la primera, haz clic en «Eliminar» y «Confirmar».
**Resultado esperado:** la API responde 404 `Sucursal.NotFound` y ves «Error al eliminar la sucursal».

#### TC-ADM-158 — Sin el permiso Eliminar no hay botón

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-020][hu] «Sin permiso»; [CU-ADM-14][cu] precondiciones; [RN-SUC-001][rn] |

**Precondiciones:** una cuenta con `Sucursales › Leer` y `Actualizar` pero no `Eliminar`.
**Datos:** `DELETE /sucursales/<id>` con el token de esa cuenta.
**Pasos:**
1. Abre una sucursal en modo editar.
2. Llama a la API.
**Resultado esperado:** el botón «Eliminar» no aparece; la API responde 403 sin `code`.

### HU-ADM-021 — Cargar sucursales desde un Excel

#### TC-ADM-159 — Descargar la plantilla de sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-021][hu] «Descargar la plantilla»; [CU-ADM-15][cu] flujo básico (paso 1); [RN-IMP-016][rn] |

**Precondiciones:** `ana.manual` en `/sucursales/carga-masiva` (botón «Carga Masiva»).
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Descargar plantilla».
2. Abre el archivo.
**Resultado esperado:** se descarga `plantilla-sucursales.xlsx` con una hoja «Plantilla» que solo contiene la fila de cabecera con las columnas «Nombre», «DireccionFisica» y «NumeroLicencia»; no hay columna de gerente ni filas de ejemplo. No verificado en la aplicación: el contenido del archivo descargado.

#### TC-ADM-160 — Cargar un archivo con todas las filas válidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-021][hu] «Cargar un archivo con todas las filas válidas»; [CU-ADM-15][cu] flujo básico (pasos 2 a 10); [RN-SUC-012, RN-SUC-013, RN-IMP-014, RN-GEN-024][rn] |

**Precondiciones:** archivo `.xlsx` hecho con la plantilla.
**Datos:** una fila: «Sucursal Carga Uno», «Calle Ficticia 1, Lima», `LIC-CARGA-0001`.
**Pasos:**
1. Arrastra o elige el archivo.
2. Mira la «Vista previa».
3. Haz clic en «Guardar válidos (1)».
4. Busca la sucursal.
**Resultado esperado:** «Vista previa» con «1 filas totales» y «1 filas válidas» y una marca verde en «Estado»; al guardar ves «Se guardaron 1 registros correctamente» y vas a `/sucursales`; la sucursal queda en la empresa A y sin gerente.

#### TC-ADM-161 — Errores por fila en la vista previa de sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-021][hu] «Errores por fila en la vista previa» (esquema); [CU-ADM-15][cu] excepción paso 6; [RN-SUC-013, RN-IMP-014][rn] |

**Precondiciones:** archivos con una fila por problema (o un archivo con varias filas); existe una sucursal con `LIC-DEMO-0001`.
**Datos:** una fila por problema de la tabla.
**Pasos:**
1. Elige el archivo.
2. Lee la columna de errores de cada fila y el contador «N filas con error».
**Resultado esperado:** cada fila queda con el icono de error en «Estado», cuenta en «N filas con error» y muestra:

| Problema | Mensaje |
|---|---|
| Nombre vacío | El campo 'nombre' es obligatorio |
| Dirección vacía | El campo 'direccionFisica' es obligatorio |
| Licencia vacía | El campo 'numeroLicencia' es obligatorio |
| Nombre de más de 100 caracteres | 'nombre' no puede superar los 100 caracteres |
| Licencia `LIC-DEMO-0001` que ya existe | El valor 'LIC-DEMO-0001' del campo 'numeroLicencia' ya existe en el sistema |
| Licencia repetida en dos filas del archivo | El valor 'LIC-CARGA-0001' del campo 'numeroLicencia' está duplicado en el archivo |

#### TC-ADM-162 — Una licencia repetida marca todas las filas repetidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-021][hu] «Una licencia repetida marca todas las filas repetidas»; [CU-ADM-15][cu] flujo básico (paso 6); [RN-SUC-013][rn] |

**Precondiciones:** archivo con dos filas con `LIC-CARGA-0001`.
**Datos:** las dos filas.
**Pasos:**
1. Elige el archivo.
**Resultado esperado:** las dos filas, incluida la primera, quedan con error y se ven «0 filas válidas». Hallazgo: se marca también la primera (Brecha conocida de CU-ADM-15; manual cap. 2, §8.3).

#### TC-ADM-163 — Solo se guardan las filas válidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-021][hu] «Solo se guardan las filas válidas»; [CU-ADM-15][cu] flujo básico (paso 8) y A2; [RN-SUC-012][rn] |

**Precondiciones:** archivo con 3 filas válidas y 2 con error; otro archivo con todas las filas con error.
**Datos:** ambos archivos.
**Pasos:**
1. Elige el primer archivo y haz clic en «Guardar válidos (3)».
2. Elige el segundo y observa el botón.
**Resultado esperado:** se guardan solo las 3 (aviso «Se guardaron 3 registros correctamente»); en el segundo archivo el botón «Guardar válidos» está deshabilitado.

#### TC-ADM-164 — Corregir el Excel y volver a subirlo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-021][hu] «Corregir y volver a subir»; [CU-ADM-15][cu] A1 |

**Precondiciones:** vista previa con filas con error.
**Datos:** el mismo archivo corregido.
**Pasos:**
1. Intenta editar una fila con error en la pantalla.
2. Corrige el Excel y súbelo otra vez.
**Resultado esperado:** las filas no se pueden corregir en la pantalla; la vista previa nueva reemplaza a la anterior.

#### TC-ADM-165 — Al guardar se omiten las licencias que ya existen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-021][hu] «Al guardar se omiten licencias que ya existen»; [CU-ADM-15][cu] A3; [RN-SUC-012][rn] |

**Precondiciones:** dos sesiones de la misma empresa. No verificado en la aplicación: requiere dos usuarios a la vez (manual cap. 2, §8.5).
**Datos:** archivo con 2 filas válidas; una segunda sesión crea una sucursal con la licencia de la fila 2 después de la vista previa.
**Pasos:**
1. En la sesión 1 muestra la vista previa (2 filas válidas).
2. En la sesión 2 crea la sucursal con la licencia de la fila 2.
3. En la sesión 1 haz clic en «Guardar válidos (2)».
**Resultado esperado:** esa fila se omite sin error y el aviso dice cuántas se guardaron realmente («Se guardaron 1 registros correctamente»).

#### TC-ADM-166 — El token no trae empresa al guardar la carga masiva

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-021][hu] «El token no trae empresa al guardar la carga»; [CU-ADM-15][cu] excepción paso 9; [RN-GEN-024][rn] |

**Precondiciones:** No verificado en la aplicación: requiere un token sin identificador de empresa.
**Datos:** `POST /sucursales/carga-masiva` con filas válidas.
**Pasos:**
1. Si el equipo técnico dispone de un token así, envía la petición.
**Resultado esperado:** la API responde 400 `Auth.Empresa`; el SPA muestra «Error al guardar los registros».

#### TC-ADM-167 — Al guardar se vuelve a validar cada fila

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-021][hu] «Al guardar se vuelve a validar cada fila»; [CU-ADM-15][cu] excepciones paso 9; [RN-IMP-006, RN-IMP-007, RN-SUC-012][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /sucursales/carga-masiva` con `{ "rows": [ { "nombre": "", "direccionFisica": "Calle 1", "numeroLicencia": "LIC-X-1" } ] }` (los nombres de propiedad se confirman con la respuesta de `previsualizar`); otra con `rows` vacío y otra con 5001 elementos.
**Pasos:**
1. Envía cada petición.
2. Busca después en `/sucursales`.
**Resultado esperado:** 400 con `code` `CargaMasivaSucursalesCommand.Rows[0].Nombre` (primera fila y campo inválidos) y no se guarda ninguna fila; `rows` vacío o de más de 5000 filas responde 400 `CargaMasivaSucursalesCommand.Rows`; en el SPA, el primer caso se vería como «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.»

#### TC-ADM-168 — Sin permiso para crear no hay carga masiva

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-021][hu] «Sin permiso para crear»; [CU-ADM-15][cu] precondiciones; [RN-SUC-001][rn] |

**Precondiciones:** cuenta `consulta.demo` (sin `Sucursales › Crear`).
**Datos:** `GET /sucursales/plantilla`, `POST /sucursales/previsualizar` con el token de esa cuenta.
**Pasos:**
1. Mira `/sucursales`.
2. Escribe `/sucursales/carga-masiva`.
3. Llama a la API.
**Resultado esperado:** no ves el botón «Carga Masiva»; la dirección muestra «No tienes permisos para acceder a esta pantalla.»; la API responde 403 sin `code`.

### HU-ADM-022 — Elegir entre las sucursales a las que tengo acceso

#### TC-ADM-169 — El administrador accede a todas las sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-022][hu] «El administrador accede a todas las sucursales»; [CU-ADM-16][cu] flujo básico; [RN-SUC-001, RN-SUC-010, RN-GEN-023][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; la empresa A tiene «Sede Principal» y «Sucursal Demo Norte» (y la empresa B otras).
**Datos:** `GET /sucursales/permitidas`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 200 con todas las sucursales no eliminadas de la empresa A, ordenadas por nombre, sin ninguna de la empresa B.

#### TC-ADM-170 — Otro usuario accede a su sucursal base, las de su cuenta y las de sus roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-022][hu] «Otro usuario accede a su sucursal base, las asignadas a su cuenta y las de sus roles»; [CU-ADM-16][cu] flujo básico (paso 2); [RN-SUC-010, RN-GEN-023][rn] |

**Precondiciones:** Carlos con sucursal base «Sede Principal» y la «Sucursal Demo Norte» asignada a su cuenta (HU-ADM-038); luego, además, a su rol.
**Datos:** `GET /sucursales/permitidas` con el token de Carlos; o abrir el selector de sucursal de «Stock por Sucursal».
**Pasos:**
1. Con la sucursal adicional asignada a la cuenta, envía la petición.
2. Quita la asignación a la cuenta y asígnala solo al rol «Cajero» (HU-ADM-031); vuelve a consultar.
**Resultado esperado:** recibe «Sede Principal» y «Sucursal Demo Norte» ordenadas por nombre en ambos casos; sin ninguna asignación solo recibe «Sede Principal».

#### TC-ADM-171 — Una asignación obsoleta o ajena no da acceso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-022][hu] «Una asignación obsoleta o ajena no da acceso»; [CU-ADM-16][cu] flujo básico (paso 3); [RN-SUC-010][rn] |

**Precondiciones:** Carlos con «Sucursal Demo Norte» asignada a su cuenta; esa sucursal se elimina después (para eso no puede tener la asignación; por lo tanto se prepara por base de datos: asignación a una sucursal marcada como eliminada o de otra empresa). No verificado en la aplicación.
**Datos:** `GET /sucursales/permitidas` con el token de Carlos.
**Pasos:**
1. Prepara la asignación obsoleta o ajena por base de datos.
2. Envía la petición.
**Resultado esperado:** la sucursal eliminada o ajena no aparece (el conjunto se cruza con las sucursales no eliminadas de la empresa).

#### TC-ADM-172 — Sin autenticación no hay sucursales permitidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-022][hu] «Sin autenticación»; [CU-ADM-16][cu] A1 y excepción paso 1; [RN-GEN-023][rn] |

**Precondiciones:** Solo API.
**Datos:** `GET /sucursales/permitidas` sin cabecera `Authorization`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 401 sin `code`; el conjunto de sucursales accesibles es vacío.

## 4.5 Empleados

### HU-ADM-023 — Buscar empleados y exportarlos a Excel

#### TC-ADM-173 — Buscar empleados sin filtros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-023][hu] «Buscar sin filtros»; [CU-ADM-21][cu] flujo básico; [RN-EPL-001, RN-EPL-016, RN-GEN-001, RN-GEN-009][rn] |

**Precondiciones:** existen tres empleados: Ana Demo Prueba, Carlos Cajero Demo y Nora Norte Demo (Nora con licencia profesional `QF-DEMO-0001`); sesión de Ana.
**Datos:** ninguno.
**Pasos:**
1. Abre `/empleados` y haz clic en «Buscar».
2. Revisa columnas, orden y celdas vacías.
**Resultado esperado:** «Resultados (3)» con las columnas «Nombre Completo», «Apellido Paterno», «Apellido Materno», «Licencia Profesional», «Correo electrónico» y «Sucursal Base»; lista ordenada por apellido paterno, luego apellido materno y luego nombres; «Nombre Completo» junta nombres, apellido paterno y apellido materno omitiendo las partes vacías; la licencia y el correo se muestran como «—» cuando no hay.

#### TC-ADM-174 — Filtros de empleados combinados y sin distinguir mayúsculas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-023][hu] «Filtros combinados sin distinguir mayúsculas»; [CU-ADM-21][cu] flujo básico |

**Precondiciones:** los tres empleados de TC-ADM-173.
**Datos:** «Nombres» `nor`; «Licencia Profesional» `qf-demo`; «Apellido Materno» `demo`.
**Pasos:**
1. Busca con cada filtro por separado (limpia entre uno y otro).
2. Busca con Enter dentro de cada campo.
**Resultado esperado:** `nor` trae solo «Nora Norte Demo»; `qf-demo` trae solo a Nora; `demo` en «Apellido Materno» trae a Carlos y a Nora; Enter dentro de cualquier campo también busca.

#### TC-ADM-175 — Filtrar empleados por sucursal base

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-023][hu] «Filtrar por sucursal base»; [CU-ADM-21][cu] flujo básico y A3 |

**Precondiciones:** los tres empleados.
**Datos:** `zzz`; luego `Norte` escrito sin elegir de la lista, y luego `Sucursal Demo Norte` elegida de la lista.
**Pasos:**
1. Escribe cada valor en «Sucursal Base» y haz clic en «Buscar».
**Resultado esperado:** `zzz` muestra «Resultados (0)» y «Sin resultados»; `Norte` y la opción elegida traen a Nora (cuenta el texto escrito, lo haya elegido de la lista o no).

#### TC-ADM-176 — Los empleados de una sucursal eliminada no aparecen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-023][hu] «Empleados de una sucursal eliminada no aparecen»; [CU-ADM-21][cu] flujo básico; [RN-EPL-017][rn] |

**Precondiciones:** la validación de dependencias impide eliminar una sucursal con empleados desde la pantalla; este caso se prepara por base de datos (marcar eliminada la sucursal base de un empleado de prueba). No verificado en la aplicación.
**Datos:** empleado «Pedro Prueba» con sucursal base marcada como eliminada.
**Pasos:**
1. Busca en `/empleados`.
**Resultado esperado:** ese empleado no aparece en el listado.

#### TC-ADM-177 — Filtro de sucursal con GUID vacío en la API

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-023][hu] «Filtro de sucursal inválido en la API»; [CU-ADM-21][cu] excepción paso 3; [RN-EPL-017][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `GET /empleados?pageNumber=1&pageSize=10&sucursalId=00000000-0000-0000-0000-000000000000`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 400 con `code` `GetEmpleadosWithPaginationQuery.SucursalId`.

#### TC-ADM-178 — Limpiar y paginar empleados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-023][hu] «Limpiar y paginar»; [CU-ADM-21][cu] A1; [RN-GEN-009][rn] |

**Precondiciones:** hay tabla y filtros.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Limpiar».
2. Busca y mira el paginador.
**Resultado esperado:** los cinco filtros quedan vacíos y la tabla se oculta; el paginador ofrece 10, 25 y 50 por página, empezando en 10.

#### TC-ADM-179 — Exportar empleados a Excel

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-023][hu] «Exportar»; [CU-ADM-21][cu] flujo básico (exportación); [RN-IMP-008, RN-IMP-009, RN-IMP-017][rn] |

**Precondiciones:** ya buscaste.
**Datos:** filtro «Nombres» `nor` escrito antes de exportar.
**Pasos:**
1. Haz clic en «Exportar».
2. Abre el archivo.
**Resultado esperado:** se descarga `empleados.xlsx` con las columnas «Nombres», «Apellido Paterno», «Apellido Materno», «Licencia Profesional», «Correo electrónico» y «Sucursal Base» según los filtros escritos. No verificado en la aplicación: el contenido del archivo; el servidor respondió 200 (manual cap. 3, §3.3).

#### TC-ADM-180 — Error al buscar o exportar empleados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-023][hu] «Error al buscar o exportar»; [CU-ADM-21][cu] excepciones pasos 3 y 7; [RN-IMP-017][rn] |

**Precondiciones:** bloqueo de red de `GET /empleados` y de `POST /empleados/exportar`; para las más de 10 000 filas, No verificado en la aplicación (ver HU-ADM-047).
**Datos:** ninguno.
**Pasos:**
1. Con el bloqueo activo, haz clic en «Buscar».
2. Quita el bloqueo de la búsqueda, busca y haz clic en «Exportar» con ese bloqueo.
**Resultado esperado:** en ambos casos ves «Error al buscar empleados».

#### TC-ADM-181 — Vista de solo consulta para un cajero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-023][hu] «Vista de solo consulta para un cajero»; [CU-ADM-21][cu] A2; [RN-EPL-001][rn] |

**Precondiciones:** Carlos (rol «Cajero» con `Empleados › Leer` sin `Crear`, `Actualizar` ni `Eliminar`).
**Datos:** ninguno.
**Pasos:**
1. Como Carlos abre `/empleados` y busca.
2. Escribe `/empleados/nueva`.
**Resultado esperado:** ve la lista, incluidos los correos y licencias de todos los empleados, pero no el botón «Nuevo»; `/empleados/nueva` lo devuelve al Dashboard con «No tienes permisos para acceder a esta pantalla.»

### HU-ADM-024 — Registrar un empleado

#### TC-ADM-182 — Registrar un empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-024][hu] «Registrar un empleado»; [CU-ADM-22][cu] flujo básico; [RN-EPL-001, RN-EPL-002, RN-EPL-003, RN-EPL-004, RN-EPL-005, RN-EPL-018][rn] |

**Precondiciones:** sesión de Ana con `Empleados › Crear`; la empresa A tiene «Sede Principal».
**Datos:** Nombres «Carlos»; Apellido Paterno «Cajero»; Apellido Materno «Demo»; Sucursal Base «Sede Principal» (elegida de la lista); Correo electrónico `carlos.cajero@sofia-manual.test`.
**Pasos:**
1. Abre `/empleados/nueva` (botón «Nuevo»), completa y haz clic en «Guardar».
2. Busca el empleado.
**Resultado esperado:** aviso «Empleado guardado correctamente»; el sistema te lleva a `/empleados` con los filtros vacíos y sin tabla; el empleado pertenece a la empresa A (no aparece en la B).

#### TC-ADM-183 — Campos obligatorios al registrar un empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-024][hu] «Campos obligatorios»; [CU-ADM-22][cu] excepción paso 2; [RN-EPL-002, RN-EPL-003, RN-EPL-018][rn] |

**Precondiciones:** `/empleados/nueva` vacío.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Guardar».
**Resultado esperado:** «Nombres», «Apellido Paterno» y «Sucursal Base» muestran «Este campo es obligatorio» y no se envía nada.

#### TC-ADM-184 — La sucursal base se elige de la lista

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-024][hu] «La sucursal base se elige de la lista»; [CU-ADM-22][cu] flujo básico; [RN-EPL-018][rn] |

**Precondiciones:** `/empleados/nueva`; la empresa tiene «Sede Principal».
**Datos:** `Sed`; luego `zzz`; luego texto sin elegir opción.
**Pasos:**
1. Escribe `Sed` en «Sucursal Base».
2. Cambia por `zzz`.
3. Escribe `Sede` sin elegir ninguna opción y haz clic en «Guardar».
**Resultado esperado:** con `Sed` la lista muestra hasta 20 sucursales cuyo nombre contiene el texto (por ejemplo «Sede Principal»); con `zzz` aparece la opción deshabilitada «Sin resultados»; si no eliges ninguna opción ves «Este campo es obligatorio».

#### TC-ADM-185 — La sucursal base debe existir en la empresa (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-024][hu] «La sucursal base debe existir en la empresa»; [CU-ADM-22][cu] excepción paso 4; [RN-EPL-018, RN-GEN-001][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; identificador de una sucursal de la empresa B; otro de una sucursal inexistente.
**Datos:** `POST /empleados` con `{ "nombres": "Prueba", "apellido_Paterno": "Ajena", "sucursal_Base_ID": "<id de B>" }`. Los nombres exactos de las propiedades salen de `CreateEmpleadoCommand` (No verificado: confírmalos en la documentación de la API antes de ejecutar).
**Pasos:**
1. Envía la petición con cada identificador.
**Resultado esperado:** 404 `Sucursal.NotFound` y no se crea el empleado; si la petición se hiciera desde el SPA se vería «Error al guardar el empleado».

#### TC-ADM-186 — Longitudes máximas de la ficha de empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-024][hu] «Longitudes máximas»; [CU-ADM-22][cu] excepción paso 4; [RN-EPL-002, RN-EPL-003, RN-EPL-004, RN-EPL-005, RN-EPL-008][rn] |

**Precondiciones:** `/empleados/nueva`.
**Datos:** 80 caracteres en cada nombre y apellido, 60 en «Licencia Profesional», 300 en «Correo electrónico».
**Pasos:**
1. Pega cada texto en su campo y cuenta lo que queda.
**Resultado esperado:** «Nombres», «Apellido Paterno» y «Apellido Materno» se detienen en 75 caracteres, «Licencia Profesional» en 50 y «Correo electrónico» en 254.

#### TC-ADM-187 — El apellido materno es opcional

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-024][hu] «El apellido materno es opcional»; [CU-ADM-22][cu] flujo básico; [RN-EPL-004][rn] |

**Precondiciones:** `/empleados/nueva`.
**Datos:** Nombres «Pedro»; Apellido Paterno «Prueba»; Apellido Materno vacío; Sucursal Base «Sede Principal».
**Pasos:**
1. Guarda el empleado.
2. Búscalo en `/empleados`.
**Resultado esperado:** el empleado se guarda sin apellido materno (nulo, nunca un texto de relleno) y la columna aparece vacía en la tabla.

#### TC-ADM-188 — La licencia profesional es opcional y única

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-024][hu] «La licencia profesional es opcional y única»; [CU-ADM-22][cu] excepción paso 5; [RN-EPL-006, RN-EPL-007][rn] |

**Precondiciones:** Nora tiene la licencia `QF-DEMO-0001`.
**Datos:** un empleado nuevo «Repetido Licencia» con `QF-DEMO-0001`; otro sin licencia.
**Pasos:**
1. Intenta guardar el empleado con la licencia repetida.
2. Guarda otro empleado sin licencia.
**Resultado esperado:** con la licencia repetida ves «Ya existe un empleado con esa licencia profesional.» (409 `Empleado.LicenciaProf.Duplicado`); sin licencia no hay conflicto y se guarda.

#### TC-ADM-189 — Correo con formato inválido en la ficha de empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-024][hu] «El correo es opcional, con formato válido y único en la empresa»; [CU-ADM-22][cu] excepción paso 2; [RN-EPL-008, RN-EPL-009][rn] |

**Precondiciones:** `/empleados/nueva`.
**Datos:** `carlos.cajero` en «Correo electrónico».
**Pasos:**
1. Escribe el correo y sal del campo.
2. Lee el texto bajo el campo.
**Resultado esperado:** debajo del campo ves «Ingresa un correo electrónico válido.»; sin error la ayuda dice «Opcional. Se usa para enviar el enlace de recuperación de contraseña.»

#### TC-ADM-190 — Correo sin punto tras la arroba

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-024][hu] «Correo sin punto tras la arroba»; [CU-ADM-22][cu] excepción paso 4 (`Empleado.Email.Invalido`); [RN-EPL-008][rn] |

**Precondiciones:** `/empleados/nueva` con los demás campos válidos.
**Datos:** `carlos.cajero@sofia`.
**Pasos:**
1. Escribe ese correo y haz clic en «Guardar».
**Resultado esperado:** la pantalla lo acepta, pero el servidor lo rechaza (400 `Empleado.Email.Invalido`) y ves el aviso rojo «Ingresa un correo electrónico válido.»; te quedas con lo que escribiste.

#### TC-ADM-191 — Correo repetido en la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-024][hu] «Correo repetido en la empresa»; [CU-ADM-22][cu] excepción paso 5; [RN-EPL-009, RN-GEN-001][rn] |

**Precondiciones:** Carlos tiene `carlos.cajero@sofia-manual.test`; la empresa B existe.
**Datos:** Nora (nuevo registro de prueba) con `CARLOS.CAJERO@sofia-manual.test`; luego, en la empresa B, un empleado con `carlos.cajero@sofia-manual.test`.
**Pasos:**
1. En la empresa A intenta guardar el empleado con el correo en mayúsculas.
2. En la empresa B guarda un empleado con el mismo correo.
**Resultado esperado:** en la empresa A ves «Ya existe un empleado con ese correo electrónico en tu empresa.» (409 `Empleado.Email.Duplicado`); en la empresa B el mismo correo sí se guarda.

#### TC-ADM-192 — Normalización del correo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-024][hu] «Normalización del correo»; [CU-ADM-22][cu] flujo básico; [RN-EPL-009][rn] |

**Precondiciones:** `/empleados/nueva`.
**Datos:** correo ` Carlos.Cajero@Sofia-Manual.test ` (con espacios); otro empleado con el correo solo con espacios.
**Pasos:**
1. Guarda cada empleado y abre su ficha.
**Resultado esperado:** el primero se guarda sin espacios sobrantes y en minúsculas (`carlos.cajero@sofia-manual.test`); el correo en blanco se guarda como nulo (la tabla muestra «—»).

#### TC-ADM-193 — Alta simultánea con la misma licencia o correo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-024][hu] «Alta simultánea con el mismo valor único»; [CU-ADM-22][cu] excepción paso 6; [RN-GEN-036][rn] |

**Precondiciones:** cliente HTTP con dos peticiones simultáneas. No verificado en la aplicación: la carrera es difícil de repetir.
**Datos:** dos `POST /empleados` con la misma licencia nueva y nombres distintos.
**Pasos:**
1. Envía las dos a la vez.
**Resultado esperado:** una responde 201 y la otra 409 (`Empleado.LicenciaProf.Duplicado` o `Db.Duplicado`); con `Db.Duplicado` el SPA muestra «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.»

#### TC-ADM-194 — Cancelar el registro de un empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-024][hu] «Cancelar»; [CU-ADM-22][cu] A1 |

**Precondiciones:** `/empleados/nueva` con datos.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** vuelves a `/empleados` sin guardar.

### HU-ADM-025 — Editar un empleado

#### TC-ADM-195 — Editar un empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-025][hu] «Editar un empleado»; [CU-ADM-23][cu] flujo básico; [RN-EPL-002, RN-EPL-003, RN-EPL-010, RN-EPL-013][rn] |

**Precondiciones:** empleado «Pedro Prueba» (sin apellido materno); Ana con `Empleados › Actualizar`.
**Datos:** Apellido Materno «Borrar».
**Pasos:**
1. Abre la ficha, haz clic en «Editar», escribe el apellido materno y haz clic en «Guardar».
**Resultado esperado:** aviso «Empleado guardado correctamente»; el sistema te lleva a la lista de empleados (no te quedas en la ficha).

#### TC-ADM-196 — La ficha de empleado es de solo lectura sin Actualizar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-025][hu] «La ficha de solo lectura»; [CU-ADM-23][cu] flujo básico; [RN-EPL-001][rn] |

**Precondiciones:** Carlos (solo `Empleados › Leer`).
**Datos:** `PUT /empleados/<id>` con el token de Carlos.
**Pasos:**
1. Como Carlos abre la ficha de un empleado.
2. Llama a la API con el token de Carlos.
**Resultado esperado:** ve los seis campos deshabilitados y no ve «Editar» ni «Eliminar»; la API responde 403 sin `code`.

#### TC-ADM-197 — Editar reemplaza todos los datos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-025][hu] «Editar reemplaza todos los datos»; [CU-ADM-23][cu] flujo básico (paso 7); [RN-EPL-010, RN-EPL-011][rn] |

**Precondiciones:** Solo API; un empleado con correo y licencia.
**Datos:** `PUT /empleados/<id>` con el cuerpo completo salvo el correo omitido.
**Pasos:**
1. Envía la petición.
2. Abre la ficha.
**Resultado esperado:** los datos que llegan reemplazan a todos los anteriores (lo omitido queda vacío), incluida la huella biométrica (dato binario opcional, que no tiene pantalla). No verificado en la aplicación: el efecto sobre la huella.

#### TC-ADM-198 — Licencia y correo repetidos al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-025][hu] «Las reglas de licencia y correo se aplican al editar»; [CU-ADM-23][cu] excepción paso 6; [RN-EPL-006, RN-EPL-009, RN-EPL-012][rn] |

**Precondiciones:** Nora tiene `QF-DEMO-0001`; Carlos tiene su correo.
**Datos:** a Pedro le pones la licencia `QF-DEMO-0001`; luego el correo de Carlos; luego guardas a Nora sin cambiar su propia licencia.
**Pasos:**
1. Edita a Pedro y guarda con cada valor.
2. Edita a Nora cambiando solo el apellido materno y guarda.
**Resultado esperado:** «Ya existe un empleado con esa licencia profesional.» y «Ya existe un empleado con ese correo electrónico en tu empresa.»; Nora se guarda sin conflicto consigo misma (el valor solo se compara con otros empleados cuando cambia).

#### TC-ADM-199 — Nadie cambia su propia sucursal base

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-025][hu] «Nadie cambia su propia sucursal base»; [CU-ADM-23][cu] excepción paso 5; [RN-EPL-011][rn] |

**Precondiciones:** Ana con permiso de editar empleados y dos sucursales; la ficha de empleado de Ana.
**Datos:** Sucursal Base «Sucursal Demo Norte».
**Pasos:**
1. Abre tu propia ficha (empleado «Ana Demo Prueba»), haz clic en «Editar», cambia la «Sucursal Base» y haz clic en «Guardar».
**Resultado esperado:** la API responde 403 `Empleado.SucursalBase.AutoAsignacion` y ves «Error al guardar el empleado» sin más explicación; la sucursal base no cambia.

#### TC-ADM-200 — Edición simultánea con un valor único repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-025][hu] «Edición simultánea con un valor único repetido»; [CU-ADM-23][cu] excepción paso 8; [RN-GEN-036][rn] |

**Precondiciones:** cliente HTTP con dos `PUT` a la vez. No verificado en la aplicación.
**Datos:** dos empleados distintos que reciben la misma licencia nueva a la vez.
**Pasos:**
1. Envía los dos `PUT /empleados/<id>` a la vez.
**Resultado esperado:** uno responde 204 y el otro 409; con `Db.Duplicado` el SPA muestra «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.»

#### TC-ADM-201 — Cambiar la sucursal base cierra la sesión del empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-025][hu] «Cambiar la sucursal base cierra la sesión del empleado»; [CU-ADM-23][cu] flujo básico; [RN-EPL-011, RN-EPL-012][rn] |

**Precondiciones:** Carlos tiene cuenta y sesión abierta en otro navegador. No verificado en la aplicación: viene de RN-EPL-011 y RN-EPL-012.
**Datos:** Ana cambia la sucursal base de Carlos a «Sucursal Demo Norte».
**Pasos:**
1. Ana edita el empleado de Carlos y guarda.
2. Carlos realiza una acción con su token anterior.
3. Carlos vuelve a entrar.
**Resultado esperado:** cambia el sello de seguridad de sus cuentas y su token de acceso anterior deja de servir (401); tras volver a entrar, su sucursal base es la nueva. Restaura la sucursal base al terminar.

#### TC-ADM-202 — Empleado inexistente o de otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-025][hu] «Empleado inexistente o de otra empresa»; [CU-ADM-23][cu] excepciones pasos 1 y 5; [RN-EPL-013, RN-GEN-001][rn] |

**Precondiciones:** identificador de un empleado de la empresa B; sesión de Ana.
**Datos:** `GET /empleados/<id de B>` y `PUT /empleados/<id de B>`; en el SPA, `/empleados/<id de B>`.
**Pasos:**
1. Abre la dirección en el SPA.
2. Llama a la API con ambos métodos.
**Resultado esperado:** la API responde 404 `Empleado.NotFound`; el SPA muestra «Error al cargar el empleado» (al abrir) o «Error al guardar el empleado» (al guardar).

### HU-ADM-026 — Eliminar un empleado

#### TC-ADM-203 — Eliminar un empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-026][hu] «Eliminar un empleado»; [CU-ADM-24][cu] flujo básico; [RN-EPL-013, RN-EPL-014, RN-GEN-003][rn] |

**Precondiciones:** «Pedro Prueba» no gerencia ninguna sucursal ni tiene cuenta.
**Datos:** ninguno.
**Pasos:**
1. Abre su ficha, «Editar» y «Eliminar».
2. Lee el cuadro y haz clic en «Confirmar».
3. Busca al empleado.
**Resultado esperado:** cuadro «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este empleado?»; al confirmar ves «Empleado eliminado correctamente» y vuelves a la lista, donde ya no aparece.

#### TC-ADM-204 — No se elimina un empleado que gerencia una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-026][hu] «El empleado es gerente de una sucursal»; [CU-ADM-24][cu] excepción paso 3; [RN-EPL-014][rn] |

**Precondiciones:** «Pedro Prueba» es gerente de «Sucursal Demo Norte».
**Datos:** ninguno.
**Pasos:**
1. Abre su ficha, «Editar», «Eliminar» y «Confirmar».
**Resultado esperado:** la API responde 409 `Empleado.EsGerente` y ves el aviso rojo «No se puede eliminar un empleado que gerencia una sucursal. Asigna otro gerente primero.»; el empleado no se borra.

#### TC-ADM-205 — Quitar el gerente para poder eliminar al empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-026][hu] «Quitar el gerente para poder eliminar»; [CU-ADM-24][cu] A2; [RN-EPL-014][rn] |

**Precondiciones:** el caso anterior.
**Datos:** vaciar «Gerente» en «Sucursal Demo Norte».
**Pasos:**
1. Edita la sucursal, vacía el gerente y guarda.
2. Elimina al empleado.
**Resultado esperado:** la eliminación procede («Empleado eliminado correctamente»).

#### TC-ADM-206 — Otros errores al eliminar un empleado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-026][hu] «Otros errores»; [CU-ADM-24][cu] excepciones pasos 2 y 3; [RN-EPL-013][rn] |

**Precondiciones:** dos sesiones; un empleado en modo editar en la primera, eliminado antes desde la segunda.
**Datos:** ninguno.
**Pasos:**
1. En la sesión 2 elimina al empleado.
2. En la sesión 1 haz clic en «Eliminar» y «Confirmar».
**Resultado esperado:** la API responde 404 `Empleado.NotFound` y ves «Error al eliminar el empleado». Hallazgo: la pantalla muestra el texto de «es gerente» ante cualquier 409 (hallazgo 3 de la sección 12.3 del catálogo de reglas).

#### TC-ADM-207 — Eliminar un empleado que tiene cuenta de usuario

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-026][hu] «Eliminar un empleado que tiene cuenta de usuario»; [CU-ADM-24][cu] A3; [RN-EPL-015, RN-CTA-007][rn] |

**Precondiciones:** el empleado «Pedro Prueba» tiene la cuenta `pedro.prueba`.
**Datos:** ninguno.
**Pasos:**
1. Elimina al empleado.
2. Busca la cuenta en «Cuentas de Usuario» y trata de abrirla.
3. Crea una cuenta nueva con el usuario `pedro.prueba` para otro empleado.
**Resultado esperado:** Hallazgo (Brecha conocida de HU-ADM-026): el sistema no lo impide y muestra «Empleado eliminado correctamente»; la cuenta desaparece de la lista de cuentas y no se puede abrir; su nombre de usuario sigue ocupado y crear otra cuenta con `pedro.prueba` da «Error al guardar la cuenta». El orden correcto es eliminar primero la cuenta y luego el empleado.

## 4.6 Roles y permisos

### HU-ADM-027 — Buscar roles y exportarlos a Excel

#### TC-ADM-208 — Buscar roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-027][hu] «Buscar roles»; [CU-ADM-25][cu] flujo básico; [RN-ROL-001, RN-ROL-019, RN-ROL-023, RN-GEN-001][rn] |

**Precondiciones:** la empresa A tiene los roles «Cajero» (nivel 30) y «Admin» (nivel 0); la empresa B tiene su propio rol «Admin».
**Datos:** ninguno.
**Pasos:**
1. Abre `/roles` (menú «Roles & Permisos»); comprueba que no hay tabla.
2. Haz clic en «Buscar».
**Resultado esperado:** «Resultados (2)» con las columnas «Nombre del Rol», «Descripción» («—» si no tiene) y «Nivel de Jerarquía»; orden por nivel de más alto a más bajo: «Cajero» y luego «Admin»; los roles de la empresa B no aparecen.

#### TC-ADM-209 — Una empresa recién registrada solo tiene el rol Admin

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-027][hu] «Una empresa recién registrada solo tiene el rol Admin»; [CU-ADM-01][cu] flujo básico (paso 6); [RN-ROL-001, RN-EMP-011][rn] |

**Precondiciones:** una empresa recién registrada.
**Datos:** ninguno.
**Pasos:**
1. Abre `/roles` y haz clic en «Buscar».
**Resultado esperado:** «Resultados (1)»: Admin, «Administrador de la empresa», nivel 0; no existe ningún otro rol preconfigurado.

#### TC-ADM-210 — Filtros de texto de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-027][hu] «Filtros de texto»; [CU-ADM-25][cu] flujo básico |

**Precondiciones:** roles «Cajero» y «Admin».
**Datos:** «Nombre del Rol» `CAJ`; «Descripción» `administrador`.
**Pasos:**
1. Busca con cada filtro por separado.
**Resultado esperado:** `CAJ` muestra «Cajero» (contiene, sin distinguir mayúsculas); `administrador` en «Descripción» encuentra «Admin».

#### TC-ADM-211 — Filtro de nivel con deslizador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-027][hu] «Filtro de nivel con deslizador»; [CU-ADM-25][cu] flujo básico |

**Precondiciones:** roles «Cajero» (30) y «Admin» (0).
**Datos:** mínimo 20 ("Nivel de Jerarquía: 20 — 100"); máximo 20 ("0 — 20").
**Pasos:**
1. Mueve el mínimo a 20 y busca.
2. Vuelve el mínimo a 0, mueve el máximo a 20 y busca.
**Resultado esperado:** con el mínimo en 20 ves solo «Cajero»; con el máximo en 20 ves solo «Admin».

#### TC-ADM-212 — Limpiar la búsqueda de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-027][hu] «Limpiar»; [CU-ADM-25][cu] A1 |

**Precondiciones:** filtros y tabla visibles.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Limpiar».
**Resultado esperado:** los textos quedan vacíos, el deslizador vuelve a 0 — 100 y la tabla se oculta.

#### TC-ADM-213 — Exportar roles a Excel

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-027][hu] «Exportar»; [CU-ADM-25][cu] flujo básico (exportación); [RN-IMP-008, RN-IMP-009, RN-IMP-017][rn] |

**Precondiciones:** ya buscaste.
**Datos:** filtro «Nombre del Rol» `caj` escrito.
**Pasos:**
1. Haz clic en «Exportar» y abre el archivo.
**Resultado esperado:** se descarga `roles.xlsx` con las columnas «Nombre del Rol», «Descripción» y «Nivel de Jerarquía» según los filtros escritos. No verificado en la aplicación: el contenido del archivo; el servidor respondió 200 (manual cap. 3, §10.2).

#### TC-ADM-214 — Error al buscar o exportar roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-027][hu] «Error al buscar o exportar»; [CU-ADM-25][cu] excepciones pasos 3 y 7; [RN-IMP-017][rn] |

**Precondiciones:** bloqueo de red; las más de 10 000 filas son No verificado (ver HU-ADM-047).
**Datos:** ninguno.
**Pasos:**
1. Bloquea `GET /roles` y haz clic en «Buscar»; luego bloquea `POST /roles/exportar` y haz clic en «Exportar».
**Resultado esperado:** «Error al buscar roles» en ambos casos.

#### TC-ADM-215 — Paginación de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-027][hu] «Paginación»; [CU-ADM-25][cu] flujo básico; [RN-GEN-009][rn] |

**Precondiciones:** hay resultados.
**Datos:** ninguno.
**Pasos:**
1. Mira el paginador.
**Resultado esperado:** el paginador ofrece 10, 25 y 50 por página.

### HU-ADM-028 — Crear un rol

#### TC-ADM-216 — Crear el rol Cajero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-028][hu] «Crear el rol Cajero»; [CU-ADM-26][cu] flujo básico; [RN-ROL-001, RN-ROL-002, RN-ROL-003, RN-ROL-004, RN-ROL-009, RN-GEN-013][rn] |

**Precondiciones:** sesión de Ana en `/roles/nueva` (botón «Nuevo»); acceso a `Auditoria_Eventos_Seguridad` (opcional).
**Datos:** Nombre del Rol «Cajero»; Nivel de Jerarquía 30; Descripción «Cajero de mostrador: atención al público y punto de venta».
**Pasos:**
1. Completa los campos y haz clic en «Guardar».
2. En `/roles` haz clic en «Buscar» y abre el rol.
3. Mira sus permisos y sucursales.
**Resultado esperado:** aviso «Rol guardado correctamente»; el sistema te lleva a `/roles` sin tabla hasta que hagas clic en «Buscar»; se registra el evento `ROL_CREAR`; el rol nace sin permisos ni sucursales.

#### TC-ADM-217 — Crear y configurar un rol son dos pasos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-028][hu] «Crear y configurar son dos pasos»; [CU-ADM-26][cu] flujo básico |

**Precondiciones:** `/roles/nueva`.
**Datos:** ninguno.
**Pasos:**
1. Busca la tarjeta «Configuración del Rol» en la pantalla de creación.
**Resultado esperado:** no existe; para marcar permisos y sucursales hay que volver a entrar al rol con «Editar».

#### TC-ADM-218 — Campos obligatorios al crear un rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-028][hu] «Campos obligatorios»; [CU-ADM-26][cu] excepción paso 2; [RN-ROL-002, RN-ROL-004][rn] |

**Precondiciones:** `/roles/nueva` como se abre (nivel en 0).
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Guardar».
**Resultado esperado:** «Este campo es obligatorio» bajo «Nombre del Rol» y bajo «Nivel de Jerarquía».

#### TC-ADM-219 — El nivel mínimo en la pantalla es 1

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-028][hu] «El nivel mínimo en la pantalla es 1»; [CU-ADM-26][cu] excepción paso 2; [RN-ROL-004][rn] |

**Precondiciones:** `/roles/nueva` con nombre válido; cliente HTTP para la parte de API.
**Datos:** nivel 0 en la pantalla; `POST /roles` con `nivelJerarquia` 0 y otro sin indicar el nivel.
**Pasos:**
1. Deja el nivel en 0 y haz clic en «Guardar».
2. Envía las dos peticiones de API.
**Resultado esperado:** la pantalla muestra «Este campo es obligatorio» (el mismo mensaje que si estuviera vacío); la API acepta 0 y asigna 0 si no se indica el nivel al crear (Brecha conocida de HU-ADM-028).

#### TC-ADM-220 — El nombre del rol no puede repetirse en la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-028][hu] «El nombre no puede repetirse en la empresa»; [CU-ADM-26][cu] excepción paso 5; [RN-ROL-003, RN-ROL-005][rn] |

**Precondiciones:** existe el rol «Admin».
**Datos:** Nombre «Admin», nivel 5; luego la empresa B crea un rol «Cajero» aunque la A ya tiene uno.
**Pasos:**
1. Intenta guardar el rol duplicado.
2. En la empresa B crea un rol con un nombre que ya existe en la A.
**Resultado esperado:** aviso rojo «Error al guardar el rol» y te quedas en la pantalla (la API responde 400 `Rol.DuplicateName`); es la única forma en que el sistema avisa del nombre duplicado; en la empresa B el mismo nombre sí se guarda.

#### TC-ADM-221 — Nombre de rol de 51 a 100 caracteres

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-028][hu] «Nombre de 51 a 100 caracteres»; [CU-ADM-26][cu] excepción paso 4; [RN-ROL-002][rn] |

**Precondiciones:** `/roles/nueva`.
**Datos:** nombre de 50 caracteres, de 51 y de 60.
**Pasos:**
1. Guarda con el nombre de 50 caracteres.
2. Intenta guardar con 51 y con 60.
**Resultado esperado:** con 50 se guarda («Rol guardado correctamente»); con 51 y 60 la API lo rechaza (400 `CreateRolCommand.NombreRol`) y ves «Error al guardar el rol». Hallazgo: el campo admite hasta 100 caracteres pero la API solo 50.

#### TC-ADM-222 — La descripción del rol es opcional

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-028][hu] «La descripción es opcional»; [CU-ADM-26][cu] excepción paso 4; [RN-ROL-003][rn] |

**Precondiciones:** `/roles/nueva`.
**Datos:** descripción vacía; descripción de 255 y de 256 caracteres.
**Pasos:**
1. Guarda con cada valor (nombre distinto cada vez).
**Resultado esperado:** vacía y de 255 se guardan; con 256 el campo se detiene en 255 o la API lo rechaza con 400 `CreateRolCommand.Descripcion` («Error al guardar el rol»). No verificado: el tope del campo en pantalla para la descripción.

#### TC-ADM-223 — Alta simultánea de roles con el mismo nombre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-028][hu] «Alta simultánea con el mismo nombre»; [CU-ADM-26][cu] excepción paso 6; [RN-GEN-036][rn] |

**Precondiciones:** cliente HTTP con dos `POST /roles` simultáneos. No verificado en la aplicación.
**Datos:** mismo nombre nuevo `Rol Carrera`.
**Pasos:**
1. Envía las dos a la vez.
**Resultado esperado:** una responde 201 y la otra 409 `Db.Duplicado` (o 400 `Rol.DuplicateName` si llegó después); en el SPA se vería «Error al guardar el rol».

#### TC-ADM-224 — El nivel de jerarquía no limita nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-028][hu] «El nivel de jerarquía no limita nada»; [CU-ADM-26][cu] flujo básico; [RN-ROL-004][rn] |

**Precondiciones:** dos roles «Rol Nivel 5» y «Rol Nivel 90» con los mismos permisos (por ejemplo `Empleados › Leer`), asignados a dos cuentas.
**Datos:** ninguno.
**Pasos:**
1. Inicia sesión con cada cuenta y abre `/empleados`.
2. Mira el orden en `/roles`.
**Resultado esperado:** ambos pueden hacer exactamente lo mismo; el nivel solo ordena la lista de roles.

#### TC-ADM-225 — Cancelar la creación de un rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-028][hu] «Cancelar»; [CU-ADM-26][cu] A1 |

**Precondiciones:** `/roles/nueva` con datos.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** vuelves a `/roles` sin guardar.

### HU-ADM-029 — Editar la descripción y el nivel de un rol

#### TC-ADM-226 — Ver la ficha de un rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-029][hu] «Ver la ficha de un rol»; [CU-ADM-27][cu] flujo básico; [RN-ROL-006, RN-ROL-007][rn] |

**Precondiciones:** rol «Cajero»; también una cuenta con `Seguridad › Leer` sin `Actualizar`.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cajero» en la búsqueda con ambas cuentas.
**Resultado esperado:** ves el título «Roles», «Datos del registro», los campos deshabilitados «Nombre del Rol», «Nivel de Jerarquía» y «Descripción» y la tarjeta «Configuración del Rol»; el botón «Editar» aparece solo con «Seguridad › Actualizar».

#### TC-ADM-227 — Editar descripción y nivel habilita los botones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-029][hu] «Editar descripción y nivel»; [CU-ADM-27][cu] flujo básico |

**Precondiciones:** ficha del rol «Cajero».
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Editar» con Ana y con una cuenta con `Actualizar` sin `Eliminar`.
**Resultado esperado:** se habilitan la descripción y el nivel; el nombre sigue deshabilitado; aparecen «Cancelar», «Guardar» y, con «Seguridad › Eliminar», «Eliminar». Hallazgo: el texto de ayuda `roles.detail.nombreRolHint` existe pero no se muestra (Brecha conocida).

#### TC-ADM-228 — Guardar la edición del rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-029][hu] «Guardar»; [CU-ADM-27][cu] flujo básico; [RN-ROL-006, RN-ROL-009][rn] |

**Precondiciones:** ficha del rol «Cajero» en modo editar; acceso a `Auditoria_Eventos_Seguridad` (opcional).
**Datos:** Nivel de Jerarquía 35.
**Pasos:**
1. Cambia el nivel a 35 y haz clic en «Guardar».
2. Busca roles y consulta el evento de auditoría.
3. Restaura el nivel a 30.
**Resultado esperado:** aviso «Rol guardado correctamente»; el rol vuelve al modo ver y te quedas en la misma ficha (no en la lista); se registra el evento `ROL_ACTUALIZAR` con el nivel como detalle.

#### TC-ADM-229 — El nombre de un rol no se cambia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-029][hu] «El nombre de un rol no se cambia»; [CU-ADM-27][cu] flujo básico; [RN-ROL-006][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `PUT /roles/<id>` con `{ "nombreRol": "Cajero Renombrado", "descripcion": "x", "nivelJerarquia": 30 }`.
**Pasos:**
1. Envía la petición.
2. Busca el rol.
**Resultado esperado:** la edición solo acepta descripción y nivel; el nombre sigue siendo «Cajero».

#### TC-ADM-230 — Cancelar la edición del rol revierte todo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-029][hu] «Cancelar»; [CU-ADM-27][cu] A1 |

**Precondiciones:** rol «Cajero» en modo editar.
**Datos:** cambiar la descripción, marcar casillas de permisos y una sucursal.
**Pasos:**
1. Haz los cambios sin guardar.
2. Haz clic en «Cancelar».
**Resultado esperado:** todo vuelve a como estaba: descripción, nivel, permisos y sucursales.

#### TC-ADM-231 — El nivel es requerido en el cuerpo de la edición

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-029][hu] «El nivel es requerido en el cuerpo»; [CU-ADM-27][cu] excepción paso 5; [RN-GEN-041][rn] |

**Precondiciones:** Solo API.
**Datos:** `PUT /roles/<id>` con `{ "descripcion": "x" }`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 400 sin campo `code`.

#### TC-ADM-232 — Rol inexistente o de otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-029][hu] «Rol inexistente o de otra empresa»; [CU-ADM-27][cu] excepciones pasos 1 y 5; [RN-ROL-007, RN-GEN-001][rn] |

**Precondiciones:** identificador del rol «Admin» de la empresa B; sesión de Ana.
**Datos:** `/roles/<id de B>` y `PUT /roles/<id de B>`.
**Pasos:**
1. Abre la dirección en el SPA.
2. Llama a la API con el token de Ana.
**Resultado esperado:** 404 `Rol.NotFound`; el SPA muestra «Error al cargar el rol» o «Error al guardar el rol».

#### TC-ADM-233 — El guardado de la ficha del rol no es atómico

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-029][hu] «El guardado de la ficha no es atómico»; [CU-ADM-27][cu] flujo básico (paso 5) |

**Precondiciones:** rol «Cajero» en modo editar; bloqueo de red de la llamada de permisos (`POST /roles/permissions`) tras los pasos anteriores.
**Datos:** cambiar el nivel, marcar una sucursal y marcar un permiso nuevo.
**Pasos:**
1. Bloquea `POST /roles/permissions` en las herramientas del navegador.
2. Haz clic en «Guardar».
3. Recarga y revisa nivel, sucursales y permisos.
**Resultado esperado:** Hallazgo (historia): los pasos anteriores (datos y sucursales) ya quedaron guardados y la pantalla muestra solo el error «Error al guardar el rol»; el permiso nuevo no se guardó.

### HU-ADM-030 — Definir los permisos de un rol

#### TC-ADM-234 — La matriz de permisos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-030][hu] «La matriz de permisos»; [CU-ADM-28][cu] flujo básico; [RN-ROL-010, RN-ROL-014][rn] |

**Precondiciones:** rol «Cajero» abierto en «Editar», pestaña «Permisos»; Ana con `Gestionar permisos`.
**Datos:** ninguno.
**Pasos:**
1. Cuenta los paneles y mira su orden.
2. Abre un panel y revisa casilla del módulo, contador y casillas por acción.
3. Busca una casilla para marcar todos los módulos.
**Resultado esperado:** 27 paneles, uno por módulo, ordenados alfabéticamente por el nombre visible; cada panel tiene la casilla del módulo que marca o desmarca todas sus acciones, el contador «N/M acciones» y una casilla por acción; no existe una casilla para marcar todos los módulos a la vez.

#### TC-ADM-235 — El catálogo de permisos sale de la API

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-030][hu] «El catálogo sale de la API»; [CU-ADM-28][cu] flujo básico; [RN-ROL-010][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `GET /roles/permissions/catalog`.
**Pasos:**
1. Envía la petición y cuenta módulos y pares módulo-acción.
2. Compara con los paneles de la matriz.
**Resultado esperado:** 105 pares en 27 módulos, agrupados por módulo y ordenados alfabéticamente; la matriz muestra exactamente esos pares.

#### TC-ADM-236 — Marcar casillas no guarda nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-030][hu] «Marcar casillas no guarda nada»; [CU-ADM-28][cu] flujo básico; [RN-ROL-011][rn] |

**Precondiciones:** rol «Cajero» en la pestaña «Permisos».
**Datos:** `Ventas › Leer`, `Crear`, `Actualizar` y `Anular`.
**Pasos:**
1. Marca las cuatro casillas y mira el contador.
2. Observa la pestaña de red.
**Resultado esperado:** el contador de «Ventas» pasa a «4/4 acciones»; no se envía nada hasta hacer clic en «Guardar».

#### TC-ADM-237 — La vista previa del menú cambia en vivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-030][hu] «La vista previa del menú cambia en vivo»; [CU-ADM-28][cu] flujo básico; [RN-ROL-015][rn] |

**Precondiciones:** rol «Cajero» en la pestaña «Menú».
**Datos:** `Profesionales de Salud › Leer`.
**Pasos:**
1. En «Permisos» marca y desmarca `Profesionales de Salud › Leer`.
2. En «Menú» mira la pantalla «Profesionales de Salud» y el texto de la pestaña.
**Resultado esperado:** la pantalla «Profesionales de Salud» cambia de marca verde a círculo vacío antes de guardar; el texto dice «Vista previa del menú real: una pantalla aparece cuando el rol tiene el permiso «Leer» de su módulo.»

#### TC-ADM-238 — Guardar los permisos del cajero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-030][hu] «Guardar los permisos»; [CU-ADM-28][cu] flujo básico; [RN-ROL-011, RN-ROL-012, RN-ROL-016, RN-GEN-022][rn] |

**Precondiciones:** rol «Cajero» sin permisos; acceso a `Auditoria_Eventos_Seguridad` (opcional).
**Datos:** los 19 permisos recomendados (ver sección 2.2): contadores esperados «Ventas 4/4 acciones», «Sesiones de Caja 3/3 acciones», «Devoluciones 2/2 acciones», «Recetas Médicas 3/5 acciones», «DIGEMID 1/6 acciones» y «1/4 acciones» en Aseguradoras, Empleados, Inventario, Medicamentos, Pacientes y Profesionales de Salud.
**Pasos:**
1. Marca los 19 permisos y revisa los contadores.
2. Haz clic en «Guardar».
3. Recarga la página y vuelve a abrir el rol.
**Resultado esperado:** aviso «Rol guardado correctamente» y te quedas en la ficha; los contadores y casillas se conservan al recargar; cada permiso agregado se registra como `PERMISO_ASIGNAR`; la caché de permisos del rol se invalida de inmediato en la instancia que atendió la petición.

#### TC-ADM-239 — Quitar un permiso a un rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-030][hu] «Quitar un permiso»; [CU-ADM-28][cu] flujo básico (revocación); [RN-ROL-011, RN-ROL-016][rn] |

**Precondiciones:** rol «Cajero» con `Profesionales de Salud › Leer`; Carlos con sesión abierta.
**Datos:** desmarcar `Profesionales de Salud › Leer`.
**Pasos:**
1. Desmarca la casilla y haz clic en «Guardar».
2. Consulta el evento de auditoría.
3. Carlos recarga y escribe `/profesionales-salud`.
4. Marca de nuevo el permiso y guarda.
**Resultado esperado:** el permiso queda revocado (borrado lógico) y se registra `PERMISO_REVOCAR`; deja de funcionar en todas las cuentas con ese rol (quien ya tiene la sesión abierta lo nota al recargar); Carlos ve «No tienes permisos para acceder a esta pantalla.» y vuelve al Inicio.

#### TC-ADM-240 — Cancelar descarta las casillas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-030][hu] «Cancelar descarta las casillas»; [CU-ADM-28][cu] A3 |

**Precondiciones:** rol «Cajero» con una sola acción en un módulo («1/4 acciones»).
**Datos:** marca más casillas.
**Pasos:**
1. Marca casillas y haz clic en «Cancelar».
**Resultado esperado:** el contador vuelve a su valor anterior («1/4 acciones»).

#### TC-ADM-241 — Un permiso revocado se restaura en lugar de duplicarse

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-030][hu] «Un permiso revocado se restaura en lugar de duplicarse»; [CU-ADM-28][cu] flujo básico (paso 5); [RN-ROL-012][rn] |

**Precondiciones:** `Ventas › Leer` revocado antes en el rol «Cajero»; acceso de solo lectura a la tabla de permisos (opcional).
**Datos:** marcar de nuevo `Ventas › Leer`.
**Pasos:**
1. Marca la casilla y guarda.
2. Revisa la tabla de permisos.
**Resultado esperado:** se restaura la misma fila en lugar de crear otra.

#### TC-ADM-242 — Un permiso no se asigna dos veces

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-030][hu] «Un permiso no se asigna dos veces»; [CU-ADM-28][cu] excepción paso 5 (`Permiso.Duplicate`); [RN-ROL-012][rn] |

**Precondiciones:** Solo API; el rol ya tiene `Ventas › Leer`.
**Datos:** `POST /roles/permissions` con `{ "rolId": "<id>", "modulo": "Ventas", "accion": "Leer" }`.
**Pasos:**
1. Envía la petición.
2. En la pantalla, guarda sin cambios.
**Resultado esperado:** 400 `Permiso.Duplicate` (aunque su tipo es Conflict); si pasara desde la pantalla se vería «Error al guardar el rol»; la pantalla solo envía los permisos nuevos respecto de los cargados, por lo que guardar sin cambios no llama a ningún endpoint de permisos.

#### TC-ADM-243 — La API no contrasta el permiso con el catálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-030][hu] «La API no contrasta el permiso con el catálogo»; [CU-ADM-28][cu] excepción paso 5; [RN-ROL-013][rn] |

**Precondiciones:** Solo API; rol de prueba descartable.
**Datos:** (a) `{ "rolId": "<id>", "modulo": "ModuloInventado", "accion": "Hacer" }`; (b) sin `modulo`; (c) sin `accion`; (d) sin `rolId`.
**Pasos:**
1. Envía cada petición a `POST /roles/permissions`.
**Resultado esperado:** Hallazgo (Brecha conocida de HU-ADM-030): (a) la API la acepta; (b), (c) y (d) responden 400 con `Permiso.Modulo`, `Permiso.Accion` y `Permiso.RolId`; desde la pantalla se vería «Error al guardar el rol». Revoca el permiso inventado al terminar.

#### TC-ADM-244 — Revocar un permiso inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-030][hu] «Revocar un permiso inexistente»; [CU-ADM-28][cu] excepción paso 7; [RN-ROL-016][rn] |

**Precondiciones:** Solo API; un `permisoId` inexistente o ya revocado.
**Datos:** `DELETE /roles/permissions/<permisoId>`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 404 `Permiso.NotFound`; en pantalla se vería «Error al guardar el rol».

#### TC-ADM-245 — Sin la acción Gestionar permisos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-030][hu] «Sin la acción Gestionar permisos»; [CU-ADM-27][cu] A2; [CU-ADM-28][cu] A4; [RN-ROL-001, RN-GEN-020][rn] |

**Precondiciones:** cuenta con `Seguridad › Leer` y `Actualizar` pero sin `Gestionar permisos` (la historia lo marca No verificado porque solo había un administrador y un cajero; con una cuenta nueva se puede probar).
**Datos:** `POST /roles/permissions` con el token de esa cuenta.
**Pasos:**
1. Abre un rol y entra a «Editar» › «Permisos».
2. Cambia la descripción y haz clic en «Guardar».
3. Llama a la API.
**Resultado esperado:** las casillas de permisos y sucursales están deshabilitadas; «Guardar» solo envía descripción y nivel; la API responde 403 sin `code`.

#### TC-ADM-246 — No ampliar los permisos de mi propio rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-030][hu] «No ampliar los permisos de mi propio rol»; [CU-ADM-28][cu] excepción paso 5 (`Rol.PermisosPropios`); [RN-ROL-016, RN-GEN-021][rn] |

**Precondiciones:** una cuenta no Admin con el rol «Gerente» y `Seguridad › Leer`, `Actualizar` y `Gestionar permisos`. La historia lo marca No verificado porque la administradora tiene el rol Admin, que está exento.
**Datos:** agregar un permiso nuevo al rol «Gerente» con esa cuenta.
**Pasos:**
1. Con la cuenta del Gerente abre su propio rol, marca un permiso nuevo y haz clic en «Guardar».
2. Intenta lo mismo sobre otro rol que la cuenta no tiene asignado.
**Resultado esperado:** la API responde 403 `Rol.PermisosPropios` y ves «No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado»; sobre otro rol sí se permite.

#### TC-ADM-247 — El rol Admin lo puede todo sin casillas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-030][hu] «El rol Admin lo puede todo sin casillas»; [CU-ADM-28][cu] A1; [RN-GEN-021][rn] |

**Precondiciones:** sesión de Ana.
**Datos:** ninguno.
**Pasos:**
1. Abre el rol «Admin» y entra a «Permisos».
2. Ejecuta una acción protegida (por ejemplo, abrir `/cuentas`).
**Resultado esperado:** ninguna casilla está marcada (todos los módulos dicen «0/N acciones») y aun así el administrador puede hacer todo.

#### TC-ADM-248 — Un rol sin permisos solo ve el Dashboard

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-ADM-030][hu] «Un rol sin permisos solo ve el Dashboard»; [CU-ADM-28][cu] flujo básico; [RN-GEN-020][rn] |

**Precondiciones:** rol «Sin Permisos Demo» (TC-ADM-084) con todas las casillas desmarcadas. No verificado en la aplicación (viene de la matriz [05b][p05b] §5).
**Datos:** cuenta `sin.permisos`.
**Pasos:**
1. Inicia sesión como `sin.permisos` y mira el Dashboard.
**Resultado esperado:** las cuentas con ese rol solo ven el Dashboard, sin tarjetas de indicadores.

### HU-ADM-031 — Definir las sucursales de un rol

#### TC-ADM-249 — Ver las sucursales del rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-031][hu] «Ver las sucursales del rol»; [CU-ADM-29][cu] flujo básico; [RN-ROL-017][rn] |

**Precondiciones:** rol «Cajero»; la empresa tiene «Sede Principal» y «Sucursal Demo Norte», una tercera eliminada y una de la empresa B.
**Datos:** ninguno.
**Pasos:**
1. Abre el rol y entra a la pestaña «Sucursales».
2. Repite en una empresa sin sucursales no eliminadas (solo por base de datos).
**Resultado esperado:** una casilla por cada sucursal no eliminada de la empresa, ordenadas por nombre («Sede Principal» y «Sucursal Demo Norte»), con las asignadas marcadas; la eliminada y la de la empresa B no aparecen; sin sucursales se lee «No hay sucursales disponibles» (esto último, No verificado: requiere una empresa sin sucursales).

#### TC-ADM-250 — Asignar sucursales a un rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-031][hu] «Asignar sucursales»; [CU-ADM-29][cu] flujo básico; [RN-ROL-017, RN-GEN-023][rn] |

**Precondiciones:** rol «Cajero» en modo editar; Carlos con rol «Cajero».
**Datos:** marcar «Sede Principal».
**Pasos:**
1. Marca la sucursal y haz clic en «Guardar».
2. Vuelve a abrir el rol.
**Resultado esperado:** aviso «Rol guardado correctamente»; las cuentas con ese rol pueden operar en esa sucursal además de su sucursal base y de las que se les agreguen a cada una. No verificado en la aplicación: el efecto más allá de la sucursal base (manual cap. 3, §13, punto 10).

#### TC-ADM-251 — Guardar reemplaza el conjunto completo de sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-031][hu] «Guardar reemplaza el conjunto completo»; [CU-ADM-29][cu] A1; [RN-ROL-017][rn] |

**Precondiciones:** rol con dos sucursales marcadas.
**Datos:** desmarcar todas.
**Pasos:**
1. Desmarca todas las sucursales y haz clic en «Guardar».
2. Abre el rol de nuevo.
3. Haz clic en «Guardar» sin tocar nada y observa la pestaña de red.
**Resultado esperado:** el rol queda sin sucursales (una lista vacía las quita todas); «Guardar» envía siempre el conjunto completo, aunque no hayas tocado nada.

#### TC-ADM-252 — Auditoría de las sucursales del rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-031][hu] «Auditoría»; [CU-ADM-29][cu] flujo básico; [RN-ROL-018, RN-GEN-013][rn] |

**Precondiciones:** acceso de solo lectura a `Auditoria_Eventos_Seguridad`.
**Datos:** guardar las sucursales del rol.
**Pasos:**
1. Guarda las sucursales del rol y consulta el último evento.
**Resultado esperado:** se registra el evento `ROL_SUCURSALES` con la cantidad de sucursales.

#### TC-ADM-253 — Sucursal inexistente o de otra empresa en el rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-031][hu] «Sucursal inexistente o de otra empresa»; [CU-ADM-29][cu] excepción paso 5; [RN-ROL-018, RN-GEN-001][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; identificador de una sucursal de la empresa B.
**Datos:** `PUT /roles/<id>/sucursales` con `{ "sucursalIds": ["<id de B>"] }` (el nombre de la propiedad se confirma con la respuesta de `GET /roles/<id>/sucursales`).
**Pasos:**
1. Envía la petición.
2. Consulta las sucursales del rol.
**Resultado esperado:** 404 `Sucursal.NotFound`; no se cambia el conjunto; en el SPA se vería «Error al guardar el rol».

#### TC-ADM-254 — Error al cargar las sucursales del rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-031][hu] «Error al cargar las sucursales»; [CU-ADM-29][cu] excepción paso 1 |

**Precondiciones:** bloqueo de red de `GET /roles/<id>/sucursales`.
**Datos:** ninguno.
**Pasos:**
1. Abre el rol y la pestaña «Sucursales».
**Resultado esperado:** aviso «Error al cargar las sucursales».

### HU-ADM-032 — Eliminar un rol

#### TC-ADM-255 — Eliminar un rol sin cuentas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-032][hu] «Eliminar un rol sin cuentas»; [CU-ADM-30][cu] flujo básico; [RN-ROL-007, RN-ROL-008, RN-ROL-009, RN-GEN-003, RN-GEN-022][rn] |

**Precondiciones:** rol «Rol Prueba Eliminar» sin cuentas vinculadas; acceso a la auditoría (opcional).
**Datos:** ninguno.
**Pasos:**
1. Abre el rol, «Editar» y «Eliminar» (al pie de «Configuración del Rol»).
2. Lee el cuadro y haz clic en «Confirmar».
**Resultado esperado:** cuadro «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este rol?»; al confirmar ves «Rol eliminado correctamente» y vuelves a la lista de roles; el borrado es lógico, se invalida la caché de permisos de ese nombre de rol y se registra `ROL_ELIMINAR`.

#### TC-ADM-256 — Cancelar la confirmación de eliminar un rol

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-032][hu] «Cancelar la confirmación»; [CU-ADM-30][cu] A1 |

**Precondiciones:** cuadro de confirmación abierto.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** el rol no se borra y sigues en modo editar.

#### TC-ADM-257 — No se elimina un rol que tiene cuentas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-032][hu] «El rol tiene cuentas»; [CU-ADM-30][cu] excepción paso 3 y A2; [RN-ROL-008][rn] |

**Precondiciones:** «Rol Prueba Eliminar» está asignado a la cuenta `pedro.prueba`.
**Datos:** ninguno.
**Pasos:**
1. Intenta eliminar el rol y confirma.
2. Cambia el rol de la cuenta (ver el caso de HU-ADM-037) y vuelve a intentar.
**Resultado esperado:** la API responde 400 `Rol.InUse` (aunque su tipo es Conflict); ves el aviso rojo «Error al eliminar el rol» sin más explicación y el rol sigue existiendo; tras cambiar el rol de las cuentas, la eliminación procede.

#### TC-ADM-258 — Un rol nuevo con el mismo nombre no hereda permisos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-032][hu] «Un rol nuevo con el mismo nombre no hereda permisos»; [CU-ADM-30][cu] flujo básico; [RN-GEN-022, RN-ROL-009][rn] |

**Precondiciones:** un rol descartable «Rol Reciclado» con `Empleados › Leer`, sin cuentas; cuenta de prueba que luego se le asigna el rol nuevo.
**Datos:** ninguno.
**Pasos:**
1. Usa una cuenta con ese rol para abrir `/empleados` (comprueba que funciona); quita el rol a la cuenta.
2. Elimina el rol y crea otro llamado «Rol Reciclado» sin permisos.
3. Asigna el rol nuevo a la cuenta, inicia sesión e intenta abrir `/empleados`.
**Resultado esperado:** el rol nuevo no hereda los permisos en caché del anterior: la cuenta no puede abrir `/empleados` (aviso «No tienes permisos para acceder a esta pantalla.»).

#### TC-ADM-259 — Eliminar un rol inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-032][hu] «Rol inexistente»; [CU-ADM-30][cu] excepción paso 3; [RN-ROL-007][rn] |

**Precondiciones:** dos sesiones; rol en modo editar eliminado antes por la otra.
**Datos:** ninguno.
**Pasos:**
1. Elimina el rol desde la sesión 2.
2. En la sesión 1 haz clic en «Eliminar» y «Confirmar».
**Resultado esperado:** 404 `Rol.NotFound` y el aviso «Error al eliminar el rol». Hallazgo (Brecha conocida de HU-ADM-032): no existe regla que impida eliminar el rol «Admin» sin cuentas; no probes este caso en una empresa con datos que quieras conservar. No verificado si la API lo impide.

### HU-ADM-033 — Cargar roles desde un Excel

#### TC-ADM-260 — Descargar la plantilla de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-033][hu] «Plantilla de roles»; [CU-ADM-31][cu] flujo básico; [RN-IMP-016, RN-ROL-020][rn] |

**Precondiciones:** `/roles/carga-masiva`.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Descargar plantilla» y abre el archivo.
**Resultado esperado:** se descarga `plantilla-roles.xlsx` con las columnas «NombreRol», «Descripcion» y «NivelJerarquia». No verificado en la aplicación: el contenido del archivo descargado.

#### TC-ADM-261 — Cargar un rol válido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-033][hu] «Cargar un rol válido»; [CU-ADM-31][cu] flujo básico; [RN-ROL-020, RN-ROL-021, RN-IMP-006, RN-IMP-007, RN-IMP-014][rn] |

**Precondiciones:** archivo con la fila «Rol Carga Uno», sin descripción, nivel 20.
**Datos:** la fila.
**Pasos:**
1. Elige el archivo y mira la vista previa.
2. Haz clic en «Guardar válidos (1)».
3. Abre el rol y mira sus permisos; revisa la auditoría.
**Resultado esperado:** «1 filas totales» y «1 filas válidas»; aviso «Se guardaron 1 registros correctamente» y voy a la lista de roles; el rol nace sin permisos ni sucursales (hay que abrirlo con «Editar»); la carga masiva no genera eventos de auditoría.

#### TC-ADM-262 — Errores por fila en la vista previa de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-033][hu] «Errores por fila en la vista previa» (esquema); [CU-ADM-31][cu] excepción paso 4; [RN-ROL-021, RN-IMP-014][rn] |

**Precondiciones:** archivo con una fila por problema; existe el rol «Cajero».
**Datos:** las filas de la tabla.
**Pasos:**
1. Elige el archivo y lee la columna de errores de cada fila.
**Resultado esperado:** cada fila queda con error y muestra:

| Problema | Mensaje |
|---|---|
| Nivel 0 | 'nivelJerarquia' debe ser mayor o igual a 1 |
| Nivel «abc» | 'nivelJerarquia' debe ser un número entero |
| Nivel vacío | El campo 'nivelJerarquia' es obligatorio |
| Nombre vacío | El campo 'nombreRol' es obligatorio |
| Nombre «Cajero» que ya existe | El valor 'Cajero' del campo 'nombreRol' ya existe en el sistema |
| Nombre repetido en dos filas | El valor 'Rol Carga Repetido' del campo 'nombreRol' está duplicado en el archivo |
| Nombre de más de 50 caracteres | 'nombreRol' no puede superar los 50 caracteres |

#### TC-ADM-263 — Una vista previa mixta de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-033][hu] «Una vista previa mixta»; [CU-ADM-31][cu] flujo básico (paso 4) |

**Precondiciones:** archivo de siete filas: una válida y seis con error (dos de ellas con nombre repetido).
**Datos:** el archivo.
**Pasos:**
1. Elige el archivo.
**Resultado esperado:** «7 filas totales», «1 filas válidas» y «6 filas con error»; las dos filas con nombre repetido quedan marcadas, incluida la primera.

#### TC-ADM-264 — La vista previa exige nivel 1 y el guardado acepta 0

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-033][hu] «La vista previa exige nivel 1 y el guardado acepta 0»; [CU-ADM-31][cu] excepción paso 7; [RN-ROL-022][rn] |

**Precondiciones:** Solo API para el nivel 0; archivo con nivel 0 para la pantalla.
**Datos:** `POST /roles/carga-masiva` con una fila de nivel 0; el mismo valor en un Excel.
**Pasos:**
1. En la pantalla, sube el Excel con nivel 0.
2. Envía la petición de API.
**Resultado esperado:** en la pantalla la fila queda con error («'nivelJerarquia' debe ser mayor o igual a 1»), por lo que en la práctica los niveles guardados desde la pantalla son 1 o más; la API acepta el nivel 0 al guardar (Hallazgo: diferencia entre vista previa y guardado).

#### TC-ADM-265 — Al guardar se omiten los nombres de rol ya existentes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-033][hu] «Al guardar se omiten los nombres ya existentes»; [CU-ADM-31][cu] A1; [RN-ROL-020][rn] |

**Precondiciones:** dos sesiones. No verificado en la aplicación.
**Datos:** archivo con dos roles válidos; la sesión 2 crea el segundo (con otra combinación de mayúsculas) tras la vista previa.
**Pasos:**
1. Sesión 1: vista previa con 2 filas válidas.
2. Sesión 2: crea un rol con el mismo nombre del segundo, en mayúsculas.
3. Sesión 1: «Guardar válidos (2)».
**Resultado esperado:** esa fila se omite sin error y el aviso dice cuántas se guardaron realmente («Se guardaron 1 registros correctamente»).

#### TC-ADM-266 — Al guardar se vuelve a validar cada fila de roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-033][hu] «Al guardar se vuelve a validar cada fila»; [CU-ADM-31][cu] excepciones paso 7; [RN-IMP-006, RN-IMP-007, RN-ROL-022][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /roles/carga-masiva` con una fila de descripción de 256 caracteres (que la vista previa no comprueba); otra con `rows` vacío.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** 400 `CargaMasivaRolesCommand.Rows[0].Descripcion` y no se guarda ninguna fila (en el SPA, «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.»); `rows` vacío responde 400 `CargaMasivaRolesCommand.Rows`.

#### TC-ADM-267 — Vista previa de roles sin archivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-033][hu] «Vista previa sin archivo»; [CU-ADM-31][cu] excepción paso 3 |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /roles/previsualizar` multipart sin el campo `file`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 400 con el texto «Debe adjuntar un archivo Excel.» (sin `code`); la pantalla, que siempre envía un archivo, mostraría «Error al procesar el archivo».

## 4.7 Cuentas de usuario

### HU-ADM-034 — Buscar cuentas de usuario

#### TC-ADM-268 — Buscar cuentas sin filtros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-034][hu] «Buscar sin filtros»; [CU-ADM-32][cu] flujo básico; [RN-CTA-001, RN-CTA-020, RN-GEN-001, RN-GEN-009][rn] |

**Precondiciones:** empresa recién registrada con solo la cuenta `ana.manual`; la empresa B tiene sus cuentas.
**Datos:** ninguno.
**Pasos:**
1. Abre `/cuentas` (menú «Cuentas de Usuario») y comprueba que no hay tabla.
2. Haz clic en «Buscar».
**Resultado esperado:** «Resultados (1)» con las columnas «Usuario», «Empleado», «Estado», «Bloqueado» e «Intentos»; la fila dice `ana.manual`, «Ana Demo Prueba», una marca verde, un candado abierto y 0; la tabla ordena por usuario; las cuentas de la empresa B no aparecen.

#### TC-ADM-269 — Filtros de texto de cuentas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-034][hu] «Filtros de texto»; [CU-ADM-32][cu] flujo básico |

**Precondiciones:** existen `ana.manual` y `carlos.cajero`.
**Datos:** «Nombre de Usuario» `CARLOS`; «Empleado» `ana`.
**Pasos:**
1. Busca con cada filtro por separado.
**Resultado esperado:** `CARLOS` encuentra `carlos.cajero` (contiene, sin distinguir mayúsculas); `ana` en «Empleado» encuentra a «Ana Demo Prueba».

#### TC-ADM-270 — Filtros de estado y bloqueo de cuentas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-034][hu] «Filtros de estado y bloqueo»; [CU-ADM-32][cu] A3; [RN-CTA-020][rn] |

**Precondiciones:** una cuenta desactivada y una bloqueada (TC-ADM-026).
**Datos:** «Cuenta Activa» = «Inactiva»; «Bloqueado» = «Sí».
**Pasos:**
1. Busca con cada filtro.
**Resultado esperado:** con «Inactiva» solo ves cuentas desactivadas (o «Resultados (0)» si no hay); «Bloqueado» = «Sí» trae solo las cuentas con fecha de bloqueo posterior a la hora actual.

#### TC-ADM-271 — Limpiar la búsqueda de cuentas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-034][hu] «Limpiar»; [CU-ADM-32][cu] A1 |

**Precondiciones:** filtros y tabla visibles.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Limpiar».
**Resultado esperado:** los filtros quedan vacíos y la tabla se oculta.

#### TC-ADM-272 — Estado visual de la cuenta y falta de exportación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-034][hu] «Estado visual»; [CU-ADM-32][cu] flujo básico |

**Precondiciones:** una cuenta inactiva y una bloqueada en los resultados.
**Datos:** ninguno.
**Pasos:**
1. Busca y mira los iconos de «Estado» y «Bloqueado».
2. Busca los botones «Exportar» y «Carga Masiva».
**Resultado esperado:** la cuenta inactiva se ve con un círculo con una equis y la bloqueada con el candado cerrado; la pantalla no tiene «Exportar» ni carga masiva.

#### TC-ADM-273 — Error al buscar cuentas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-034][hu] «Error al buscar»; [CU-ADM-32][cu] excepción paso 3 |

**Precondiciones:** bloqueo de red de `GET /cuentas`.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Buscar».
**Resultado esperado:** aviso «Error al buscar cuentas».

### HU-ADM-035 — Crear la cuenta de un empleado

#### TC-ADM-274 — Crear la cuenta del cajero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-035][hu] «Crear la cuenta del cajero»; [CU-ADM-33][cu] flujo básico; [RN-CTA-001, RN-CTA-002, RN-CTA-003, RN-CTA-004, RN-CTA-005, RN-CTA-008, RN-AUT-031, RN-GEN-013][rn] |

**Precondiciones:** el empleado «Carlos Cajero Demo» existe y no tiene cuenta; Ana tiene `Seguridad › Crear` y `Empleados › Leer`; acceso a auditoría (opcional).
**Datos:** Empleado: escribir «Ca» y elegir «Carlos Cajero Demo»; Nombre de Usuario `carlos.cajero`; Contraseña inicial: una contraseña válida de 8 caracteres o más.
**Pasos:**
1. Abre `/cuentas/nueva` (botón «Nuevo»), completa los tres campos y haz clic en «Guardar».
2. Mira la ficha.
**Resultado esperado:** aviso «Cuenta guardada correctamente»; el sistema abre la ficha de la cuenta (`/cuentas/…`) en modo ver; la cuenta nace activa, con 0 intentos fallidos, «Requiere Cambio de Clave: Sí», «Sin roles asignados» y «Sin sucursales adicionales asignadas»; pertenece a la empresa A; se registra el evento `CUENTA_REGISTRAR`; la contraseña se guarda como hash BCrypt y no vuelve a mostrarse.

#### TC-ADM-275 — El formulario de alta no pide rol ni sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-035][hu] «El formulario no pide rol ni sucursales»; [CU-ADM-33][cu] A1 |

**Precondiciones:** la cuenta recién creada (TC-ADM-274).
**Datos:** ninguno.
**Pasos:**
1. Revisa los campos de `/cuentas/nueva`.
2. Inicia sesión con la cuenta recién creada (cambia primero la contraseña, TC-ADM-060) y abre cualquier pantalla con permiso.
**Resultado esperado:** el formulario solo tiene «Empleado», «Nombre de Usuario» y «Contraseña»; una cuenta sin rol no puede hacer nada útil: ve solo el Dashboard y cualquier endpoint con permiso responde 403 hasta que se le asigne un rol.

#### TC-ADM-276 — Campos obligatorios al crear una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-035][hu] «Campos obligatorios»; [CU-ADM-33][cu] excepción paso 2; [RN-CTA-002, RN-CTA-003, RN-CTA-004][rn] |

**Precondiciones:** `/cuentas/nueva` vacío.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Guardar».
**Resultado esperado:** los tres campos muestran «Este campo es obligatorio» y no se envía nada.

#### TC-ADM-277 — El empleado se elige de la lista

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-035][hu] «El empleado se elige de la lista»; [CU-ADM-33][cu] flujo básico (paso 1); [RN-CTA-002][rn] |

**Precondiciones:** `/cuentas/nueva`.
**Datos:** `C`; `Ca`; `Zzzz`.
**Pasos:**
1. Escribe una letra y observa la pestaña de red.
2. Escribe «Ca» y luego un texto sin coincidencia («Zzzz») y sal del campo.
**Resultado esperado:** con menos de 2 letras no se busca; con «Ca» aparece «Carlos Cajero Demo»; si escribes un texto que no coincide con ninguna opción, el campo se vacía al salir.

#### TC-ADM-278 — Usuario demasiado largo al crear una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-035][hu] «Usuario demasiado largo»; [CU-ADM-33][cu] excepción paso 2; [RN-CTA-003][rn] |

**Precondiciones:** `/cuentas/nueva`.
**Datos:** usuario de 50 caracteres y otro de 51.
**Pasos:**
1. Escribe cada usuario y haz clic en «Guardar» (con empleado y contraseña válidos).
**Resultado esperado:** con 50 se guarda; con 51 ves «La longitud máxima es de 100 caracteres». Hallazgo: el mensaje dice 100 pero el límite real es 50 (Brecha conocida de HU-ADM-035).

#### TC-ADM-279 — Contraseña de menos de 8 caracteres al crear una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-035][hu] «Contraseña de menos de 8 caracteres»; [CU-ADM-33][cu] excepción paso 2; [RN-CTA-004][rn] |

**Precondiciones:** `/cuentas/nueva` con empleado y usuario válidos.
**Datos:** contraseña de 7 caracteres; de 8; de 128; de 129.
**Pasos:**
1. Intenta guardar con cada contraseña.
**Resultado esperado:** con 7 (y con 129) no aparece ningún mensaje y no se guarda; con 8 y con 128 se guarda. Hallazgo: la contraseña de 1 a 7 caracteres no muestra mensaje; el campo de contraseña no tiene botón para mostrar el texto.

#### TC-ADM-280 — El usuario ya existe en el sistema

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-035][hu] «El usuario ya existe en el sistema»; [CU-ADM-33][cu] excepción paso 5; [RN-CTA-007][rn] |

**Precondiciones:** `ana.manual` existe; existe un empleado sin cuenta; la empresa B tiene `luis.dos`.
**Datos:** usuario `ana.manual`; en la empresa A, usuario `luis.dos`.
**Pasos:**
1. Crea la cuenta con cada usuario.
**Resultado esperado:** aviso rojo «Error al guardar la cuenta» en ambos (409 `Auth.DuplicateUsername`); el usuario es único en todo el sistema entre cuentas no eliminadas, porque el inicio de sesión no pide empresa.

#### TC-ADM-281 — El empleado ya tiene cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-035][hu] «El empleado ya tiene cuenta»; [CU-ADM-33][cu] excepción paso 5; [RN-CTA-006][rn] |

**Precondiciones:** «Ana Demo Prueba» tiene `ana.manual`.
**Datos:** empleado «Ana Demo Prueba»; usuario libre `ana.otra`.
**Pasos:**
1. Elige a Ana en la lista y guarda.
**Resultado esperado:** aviso «Error al guardar la cuenta» (409 `Auth.DuplicateAccount`); la lista de empleados también muestra a quienes ya tienen cuenta.

#### TC-ADM-282 — Alta simultánea del mismo usuario

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-035][hu] «Alta simultánea del mismo usuario»; [CU-ADM-33][cu] excepción paso 6; [RN-GEN-036][rn] |

**Precondiciones:** cliente HTTP con dos `POST /cuentas` simultáneos. No verificado en la aplicación.
**Datos:** mismo usuario nuevo `cuenta.carrera` para dos empleados distintos.
**Pasos:**
1. Envía las dos a la vez.
**Resultado esperado:** una responde 201 y la otra 409 (`Db.Duplicado` o `Auth.DuplicateUsername`); en el SPA se vería «Error al guardar la cuenta» porque la pantalla de creación no traduce los códigos de la API.

#### TC-ADM-283 — Empleado inexistente al crear una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-035][hu] «Empleado inexistente»; [CU-ADM-33][cu] excepción paso 5; [RN-CTA-002, RN-GEN-001][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; identificador de un empleado de la empresa B y otro de un empleado eliminado.
**Datos:** `POST /cuentas` con `{ "empleadoId": "<id>", "nombreUsuario": "cuenta.ajena", "password": "<8+ caracteres>" }`.
**Pasos:**
1. Envía la petición con cada identificador.
**Resultado esperado:** 404 `Empleado.NotFound`; no se crea la cuenta.

#### TC-ADM-284 — Cancelar la creación de una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-035][hu] «Cancelar»; [CU-ADM-33][cu] A2 |

**Precondiciones:** `/cuentas/nueva` con datos.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** vuelves a `/cuentas`.

### HU-ADM-036 — Administrar el estado de una cuenta

#### TC-ADM-285 — Ver la ficha de una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-036][hu] «Ver la ficha de una cuenta»; [CU-ADM-34][cu] flujo básico; [RN-CTA-001][rn] |

**Precondiciones:** la cuenta `carlos.cajero`; una cuenta bloqueada para ver «Bloqueado Hasta».
**Datos:** ninguno.
**Pasos:**
1. Haz clic en `carlos.cajero` en la búsqueda.
2. Abre también la ficha de una cuenta bloqueada.
**Resultado esperado:** ves deshabilitados «Nombre de Usuario», «Empleado», «Intentos Fallidos», «Cuenta Activa» («Activa» o «Inactiva»), «Bloqueado» («Sí» o «No»), «Requiere Cambio de Clave», «Bloqueado Hasta» (solo si hay fecha de bloqueo) y «Creado el»; y las tarjetas «Roles Asignados» y «Sucursales Asignadas».

#### TC-ADM-286 — Desactivar una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-036][hu] «Desactivar una cuenta»; [CU-ADM-34][cu] flujo básico; [RN-CTA-009, RN-CTA-010, RN-CTA-012, RN-GEN-013][rn] |

**Precondiciones:** la cuenta `carlos.cajero` activa, con sesión abierta en otro navegador.
**Datos:** interruptor «Cuenta Activa» apagado.
**Pasos:**
1. Haz clic en «Editar», apaga «Cuenta Activa» y haz clic en «Guardar».
2. Carlos intenta iniciar sesión, y en su sesión viva hace una acción.
3. Ana consulta auditoría y la ficha.
**Resultado esperado:** aviso «Cuenta guardada correctamente» y la ficha muestra «Cuenta Activa: Inactiva»; Carlos no puede iniciar sesión («Credenciales inválidas. Verifica tu usuario y contraseña.»), ni renovar la sesión, ni cambiar su clave, ni usar tokens vivos (se comprueba en cada petición); el usuario, el empleado y el resto de la ficha no se borran; se registra `CUENTA_ACTUALIZAR`.

#### TC-ADM-287 — Reactivar una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-036][hu] «Reactivar una cuenta»; [CU-ADM-34][cu] flujo básico; [RN-CTA-009, RN-CTA-012][rn] |

**Precondiciones:** cuenta inactiva (TC-ADM-286).
**Datos:** interruptor encendido.
**Pasos:**
1. Edita, enciende «Cuenta Activa» y guarda.
2. Carlos inicia sesión.
**Resultado esperado:** la cuenta vuelve a estar «Activa» y puede iniciar sesión; el valor solo cambia si el enviado difiere del actual.

#### TC-ADM-288 — Forzar el cambio de contraseña

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-036][hu] «Forzar el cambio de contraseña»; [CU-ADM-34][cu] flujo básico; [CU-ADM-05][cu] A1; [RN-CTA-011, RN-GEN-019][rn] |

**Precondiciones:** la cuenta `carlos.cajero` con «Requiere Cambio de Clave: No».
**Datos:** interruptor «Forzar Cambio de Contraseña» encendido.
**Pasos:**
1. Edita, observa que el interruptor empieza apagado, enciéndelo y guarda.
2. Carlos inicia sesión.
3. Vuelve a editar la cuenta y busca una forma de apagar el cambio forzado.
**Resultado esperado:** la ficha muestra «Requiere Cambio de Clave: Sí»; en su próximo ingreso Carlos debe cambiar su contraseña (TC-ADM-058); el interruptor empieza siempre apagado y no hay forma de apagarlo a mano: se apaga solo cuando la persona cambia o restablece su contraseña.

#### TC-ADM-289 — Desbloquear una cuenta con «Resetear Intentos Fallidos»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-036][hu] «Desbloquear una cuenta»; [CU-ADM-34][cu] A2; [RN-CTA-010][rn] |

**Precondiciones:** cuenta con «Intentos Fallidos: 5» y «Bloqueado: Sí» (TC-ADM-026).
**Datos:** casilla «Resetear Intentos Fallidos» marcada.
**Pasos:**
1. Edita, marca la casilla y guarda.
2. Mira la ficha; intenta iniciar sesión con la contraseña correcta.
**Resultado esperado:** «Intentos Fallidos: 0», «Bloqueado: No» y «Bloqueado Hasta» desaparece de la ficha; la persona entra; es la única forma de desbloquear antes de los 15 minutos desde esta pantalla.

#### TC-ADM-290 — Usuario y empleado no se pueden cambiar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-036][hu] «Usuario y empleado no se pueden cambiar»; [CU-ADM-34][cu] flujo básico |

**Precondiciones:** ficha de una cuenta en modo editar.
**Datos:** ninguno.
**Pasos:**
1. Intenta escribir en «Nombre de Usuario» y «Empleado».
**Resultado esperado:** ambos están deshabilitados.

#### TC-ADM-291 — Cancelar la edición de la cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-036][hu] «Cancelar»; [CU-ADM-34][cu] A1; [CU-ADM-36][cu] A1 |

**Precondiciones:** cuenta en modo editar.
**Datos:** cambia «Cuenta Activa», «Forzar Cambio de Contraseña» y agrega una sucursal.
**Pasos:**
1. Haz los cambios y haz clic en «Cancelar».
**Resultado esperado:** se descartan los cambios de los controles, pero no se deshacen las sucursales ya agregadas o quitadas (se aplican al instante).

#### TC-ADM-292 — Solo un administrador modifica la cuenta de un administrador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-036][hu] «Solo un administrador modifica la cuenta de un administrador»; [CU-ADM-34][cu] excepción paso 5; [RN-CTA-009][rn] |

**Precondiciones:** cuenta de un gestor sin rol `Admin` con `Seguridad › Leer` y `Actualizar`; la cuenta `ana.manual` con rol `Admin`. La historia lo marca No verificado en la aplicación.
**Datos:** modificar `ana.manual` con la cuenta del gestor.
**Pasos:**
1. Con el gestor abre la ficha de `ana.manual`, haz clic en «Editar» y «Guardar».
**Resultado esperado:** la API responde 403 `Cuenta.AdminProtegida` y ves «Solo un administrador puede modificar o eliminar la cuenta de un administrador».

#### TC-ADM-293 — Cuenta inexistente o de otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-036][hu] «Cuenta inexistente»; [CU-ADM-34][cu] excepciones pasos 1 y 5; [RN-CTA-001, RN-GEN-001][rn] |

**Precondiciones:** identificador de una cuenta de la empresa B y otro de una cuenta eliminada.
**Datos:** `/cuentas/<id>` y `PUT /cuentas/<id>`.
**Pasos:**
1. Abre la dirección en el SPA.
2. Llama a la API con el token de Ana.
**Resultado esperado:** 404 `Cuenta.NotFound`; el SPA muestra «Error al cargar la cuenta» o «Error al guardar la cuenta». Hallazgo: al abrir por dirección la ficha de una cuenta cuyo empleado fue eliminado se vio la clave sin traducir `cuentas.detail.load-error` en lugar del mensaje (manual cap. 3, §12, punto 1).

#### TC-ADM-294 — Sin el permiso Actualizar no hay botón «Editar» en la cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-036][hu] «Sin el permiso no hay botón»; [CU-ADM-34][cu] excepción paso 4; [RN-CTA-001][rn] |

**Precondiciones:** cuenta `consulta.demo` con `Seguridad › Leer`.
**Datos:** `PUT /cuentas/<id>` con el token de esa cuenta.
**Pasos:**
1. Abre la ficha de una cuenta.
2. Llama a la API.
**Resultado esperado:** no ves «Editar»; la API responde 403 sin `code`.

### HU-ADM-037 — Asignar o cambiar el rol de una cuenta

#### TC-ADM-295 — Asignar el rol Cajero a una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-037][hu] «Asignar el rol Cajero»; [CU-ADM-35][cu] flujo básico; [RN-CTA-014, RN-CTA-015, RN-CTA-016, RN-CTA-017, RN-ROL-001][rn] |

**Precondiciones:** `carlos.cajero` muestra «Sin roles asignados»; Ana con `Asignar roles`; rol «Cajero» existe.
**Datos:** rol «Cajero».
**Pasos:**
1. Edita la cuenta, abre «Agregar rol...» y elige «Cajero».
2. Haz clic en «Guardar».
3. Abre de nuevo el campo sin escribir nada.
**Resultado esperado:** aviso «Cuenta guardada correctamente»; la ficha vuelve al modo ver mostrando «Roles Asignados: Cajero»; el campo «Agregar rol...» sin escribir nada lista todos los roles de la empresa.

#### TC-ADM-296 — Cambiar de rol reemplaza el anterior

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-037][hu] «Cambiar de rol reemplaza el anterior»; [CU-ADM-35][cu] flujo básico; [RN-CTA-015, RN-CTA-016][rn] |

**Precondiciones:** la cuenta tiene el rol «Cajero»; existe otro rol «Rol Prueba Eliminar».
**Datos:** rol nuevo «Rol Prueba Eliminar».
**Pasos:**
1. Edita, borra el texto del campo de rol y escribe las primeras letras del rol nuevo; elígelo y guarda.
**Resultado esperado:** el sistema quita el rol anterior y asigna el nuevo; para cambiarlo hay que borrar el texto del campo y escribir las primeras letras del rol nuevo, porque el rol actual aparece escrito y filtra la lista.

#### TC-ADM-297 — No se puede dejar una cuenta sin rol desde la pantalla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-037][hu] «No se puede dejar una cuenta sin rol desde la pantalla»; [CU-ADM-35][cu] A1 |

**Precondiciones:** la cuenta tiene un rol.
**Datos:** campo de rol vacío.
**Pasos:**
1. Edita, vacía el campo del rol y guarda.
**Resultado esperado:** el rol no se quita. Hallazgo: la API admite varios roles y quitarlos (`POST /roles/remove-from-user`), pero la pantalla trabaja con un solo rol.

#### TC-ADM-298 — Efecto del cambio de rol sobre la sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-037][hu] «Efecto sobre la sesión»; [CU-ADM-35][cu] A3; [RN-CTA-018, RN-GEN-017][rn] |

**Precondiciones:** Carlos con sesión abierta; acceso a auditoría (opcional).
**Datos:** Ana cambia el rol de Carlos.
**Pasos:**
1. Ana asigna otro rol a Carlos y guarda.
2. Carlos hace una acción.
3. Consulta auditoría.
**Resultado esperado:** cambia el sello de seguridad: la sesión actual de Carlos se cierra (401 → «Iniciar sesión») y los permisos nuevos se aplican en su siguiente ingreso; se registra `ROL_ASIGNAR_CUENTA` o `ROL_QUITAR_CUENTA`.

#### TC-ADM-299 — Asignar un rol que ya se tiene o quitar uno que no se tiene

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-037][hu] «Asignar un rol que ya se tiene o quitar uno que no se tiene»; [CU-ADM-35][cu] flujo básico; [RN-CTA-016][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; la cuenta de Carlos con rol «Cajero».
**Datos:** `POST /cuentas/<cuentaId>/roles/<rolId del Cajero>` (repetida); `DELETE /cuentas/<cuentaId>/roles/<otro rolId>` (no asignado).
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** la API responde éxito en ambas y no cambia nada.

#### TC-ADM-300 — Nadie cambia sus propios roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-037][hu] «Nadie cambia sus propios roles»; [CU-ADM-35][cu] excepción paso 3; [RN-CTA-017][rn] |

**Precondiciones:** una cuenta de prueba (no la única Admin de la empresa) con `Asignar roles`. La historia lo marca No verificado: no se probó con la cuenta propia para no arriesgar el acceso.
**Datos:** `POST /cuentas/<id de la propia cuenta>/roles/<rolId>`.
**Pasos:**
1. Con esa cuenta intenta asignarse un rol a sí misma.
**Resultado esperado:** 403 `Rol.AutoAsignacion` y «Una cuenta no puede modificar sus propios roles».

#### TC-ADM-301 — El rol Admin está reservado a los administradores

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-037][hu] «El rol Admin está reservado»; [CU-ADM-35][cu] excepción paso 3; [RN-CTA-017, RN-ROL-001][rn] |

**Precondiciones:** gestor sin rol `Admin` con `Asignar roles`. La historia lo marca No verificado.
**Datos:** asignar o quitar el rol «Admin» a la cuenta de Carlos.
**Pasos:**
1. Con el gestor, edita la cuenta de Carlos, elige «Admin» y guarda; repite para quitarlo de otra cuenta.
**Resultado esperado:** 403 `Rol.AdminReservado` y «Solo un administrador puede asignar o quitar el rol Admin».

#### TC-ADM-302 — Rol o cuenta inexistentes al asignar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-037][hu] «Rol o cuenta inexistentes»; [CU-ADM-35][cu] excepción paso 3; [RN-ROL-007, RN-CTA-014][rn] |

**Precondiciones:** Solo API; identificadores de un rol de la empresa B y de una cuenta eliminada.
**Datos:** `POST /cuentas/<cuenta>/roles/<rol de B>`; `POST /cuentas/<cuenta eliminada>/roles/<rol>`.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** 404 `Rol.NotFound` (en el SPA «Error al guardar la cuenta») y 404 `Auth.CuentaNotFound` (en el SPA «La cuenta no existe»).

#### TC-ADM-303 — Sin el permiso de asignar roles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-037][hu] «Sin el permiso de asignar roles»; [CU-ADM-35][cu] excepción paso 2; [RN-CTA-014][rn] |

**Precondiciones:** cuenta con `Seguridad › Actualizar` pero sin `Asignar roles`. La historia lo marca No verificado.
**Datos:** `POST /cuentas/<id>/roles/<rolId>` con su token.
**Pasos:**
1. Edita una cuenta.
2. Llama a la API.
**Resultado esperado:** la tarjeta «Roles Asignados» sigue siendo solo texto; la API responde 403 sin `code`.

#### TC-ADM-304 — Falla la asignación después de quitar el rol anterior

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-037][hu] «Falla la asignación después de quitar el rol anterior»; [CU-ADM-35][cu] flujo básico |

**Precondiciones:** cuenta con el rol «Cajero»; en las herramientas del navegador, bloqueo de la llamada `POST /cuentas/<id>/roles/<rolId>` (la que asigna).
**Datos:** rol nuevo.
**Pasos:**
1. Elige otro rol, activa el bloqueo de la asignación y haz clic en «Guardar».
2. Quita el bloqueo y recarga la ficha.
**Resultado esperado:** Hallazgo (historia): el rol anterior ya fue quitado (la secuencia no es atómica) y la cuenta queda «Sin roles asignados»; la pantalla muestra el error de la operación.

### HU-ADM-038 — Agregar y quitar sucursales de una cuenta

#### TC-ADM-305 — Agregar una sucursal a una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-038][hu] «Agregar una sucursal»; [CU-ADM-36][cu] flujo básico; [RN-CTA-014, RN-CTA-019, RN-GEN-023][rn] |

**Precondiciones:** Ana con `Seguridad › Actualizar` y `Sucursales › Leer`; existe «Sucursal Demo Norte».
**Datos:** sucursal «Sucursal Demo Norte».
**Pasos:**
1. Edita la cuenta de Carlos y elige «Sucursal Demo Norte» en «Agregar sucursal...».
2. Mira la pestaña de red y la auditoría.
**Resultado esperado:** se aplica al instante, sin «Guardar» y sin confirmación; aviso «Sucursal asignada correctamente» y la sucursal aparece en la lista; se registra `CUENTA_SUC_ASIGNAR`.

#### TC-ADM-306 — Quitar una sucursal de una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-038][hu] «Quitar una sucursal»; [CU-ADM-36][cu] flujo básico; [RN-CTA-019][rn] |

**Precondiciones:** Carlos tiene «Sucursal Demo Norte» asignada.
**Datos:** ninguno.
**Pasos:**
1. En modo editar, haz clic en la papelera «Quitar sucursal» de su fila.
2. Busca la sucursal en `/sucursales`.
**Resultado esperado:** aviso «Sucursal eliminada correctamente» y la fila desaparece; solo se quita de esta cuenta, la sucursal sigue existiendo; se registra `CUENTA_SUC_QUITAR`.

#### TC-ADM-307 — Una cuenta sin sucursales adicionales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-038][hu] «Sin sucursales adicionales»; [CU-ADM-36][cu] flujo básico |

**Precondiciones:** Carlos sin sucursales adicionales.
**Datos:** ninguno.
**Pasos:**
1. Abre la ficha de Carlos.
**Resultado esperado:** la tarjeta dice «Sin sucursales adicionales asignadas»; la cuenta solo opera en la sucursal base de su empleado.

#### TC-ADM-308 — Cancelar no deshace los cambios de sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-038][hu] «Cancelar no deshace los cambios de sucursales»; [CU-ADM-36][cu] A1 |

**Precondiciones:** cuenta en modo editar.
**Datos:** agrega una sucursal.
**Pasos:**
1. Agrega una sucursal y haz clic en «Cancelar».
2. Recarga la ficha.
**Resultado esperado:** la sucursal sigue asignada.

#### TC-ADM-309 — Repetir la operación de sucursal no cambia nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-038][hu] «Repetir la operación no cambia nada»; [CU-ADM-36][cu] flujo básico; [RN-CTA-019][rn] |

**Precondiciones:** Solo API; Carlos con «Sucursal Demo Norte» asignada.
**Datos:** `POST /cuentas/<id>/sucursales/<id Norte>` (repetida); `DELETE /cuentas/<id>/sucursales/<id Sede Principal>` (no asignada).
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** la API responde éxito sin cambios. No verificado en la aplicación: «Error al asignar la sucursal» y «Error al eliminar la sucursal» no se pudieron provocar.

#### TC-ADM-310 — Sucursal o cuenta inexistente al asignar sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-038][hu] «Sucursal o cuenta inexistente»; [CU-ADM-36][cu] excepción paso 3; [RN-CTA-014, RN-GEN-001][rn] |

**Precondiciones:** Solo API; identificador de una sucursal de la empresa B y de una cuenta inexistente.
**Datos:** `POST /cuentas/<id>/sucursales/<id de B>`; `POST /cuentas/<id inexistente>/sucursales/<id Norte>`.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** 404 `Sucursal.NotFound` (el SPA diría «La sucursal no existe») y 404 `Auth.CuentaNotFound` («La cuenta no existe»).

#### TC-ADM-311 — El efecto de una sucursal asignada es inmediato

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-038][hu] «El efecto es inmediato, sin renovar el token»; [CU-ADM-36][cu] A2; [RN-CTA-019, RN-GEN-023][rn] |

**Precondiciones:** Carlos con sesión abierta y sin sucursales adicionales; `GET /sucursales/permitidas`.
**Datos:** agregar «Sucursal Demo Norte» a Carlos.
**Pasos:**
1. Con el token de Carlos consulta `GET /sucursales/permitidas`.
2. Ana agrega la sucursal.
3. Con el mismo token (sin renovar) vuelve a consultar.
**Resultado esperado:** antes solo «Sede Principal»; después «Sede Principal» y «Sucursal Demo Norte» sin renovar el token ni cerrar sesión (el acceso se calcula en cada petición).

### HU-ADM-039 — Eliminar una cuenta

#### TC-ADM-312 — Eliminar una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-039][hu] «Eliminar la cuenta»; [CU-ADM-37][cu] flujo básico; [RN-CTA-001, RN-CTA-009, RN-CTA-013, RN-GEN-003, RN-GEN-013][rn] |

**Precondiciones:** una cuenta de prueba (`cuenta.eliminar`); acceso a auditoría (opcional).
**Datos:** ninguno.
**Pasos:**
1. Abre la ficha, «Editar» y «Eliminar».
2. Lee el cuadro y haz clic en «Confirmar».
**Resultado esperado:** cuadro «Eliminar cuenta» con «¿Estás seguro de que deseas eliminar esta cuenta de usuario? Esta acción no se puede deshacer.» y botones «Cancelar» y «Confirmar»; al confirmar ves «Cuenta eliminada correctamente» y vuelves a la lista de cuentas, donde ya no aparece; se registra `CUENTA_ELIMINAR`.

#### TC-ADM-313 — Cancelar la eliminación de una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-039][hu] «Cancelar la confirmación»; [CU-ADM-37][cu] A1 |

**Precondiciones:** cuadro «Eliminar cuenta» abierto.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** no se borra nada.

#### TC-ADM-314 — Eliminar la cuenta no elimina al empleado y libera el usuario

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-039][hu] «El empleado no se elimina»; [CU-ADM-37][cu] flujo básico; [CU-ADM-24][cu] A3; [RN-CTA-013, RN-CTA-007][rn] |

**Precondiciones:** la cuenta de Carlos eliminada.
**Datos:** crear una cuenta nueva con el usuario `carlos.cajero` para el mismo empleado.
**Pasos:**
1. Busca a Carlos en `/empleados`.
2. Crea una cuenta nueva para él con el mismo usuario.
**Resultado esperado:** el empleado «Carlos Cajero Demo» sigue en la lista de empleados; su nombre de usuario y su empleado quedan libres y la cuenta nueva se guarda.

#### TC-ADM-315 — Solo un administrador elimina la cuenta de un administrador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-039][hu] «Solo un administrador elimina la cuenta de un administrador»; [CU-ADM-37][cu] excepción paso 3; [RN-CTA-009][rn] |

**Precondiciones:** gestor sin rol `Admin` con `Seguridad › Eliminar`. La historia lo marca No verificado.
**Datos:** eliminar `ana.manual` con el gestor.
**Pasos:**
1. Con el gestor, abre la ficha de `ana.manual`, «Editar», «Eliminar», «Confirmar».
**Resultado esperado:** 403 `Cuenta.AdminProtegida` y «Solo un administrador puede modificar o eliminar la cuenta de un administrador».

#### TC-ADM-316 — Otros errores al eliminar una cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-039][hu] «Otros errores»; [CU-ADM-37][cu] excepciones pasos 2 y 3; [RN-CTA-013][rn] |

**Precondiciones:** dos sesiones; una cuenta eliminada por la segunda mientras la primera tiene el cuadro abierto.
**Datos:** ninguno.
**Pasos:**
1. Elimina la cuenta en la sesión 2.
2. En la sesión 1 haz clic en «Confirmar».
**Resultado esperado:** 404 `Cuenta.NotFound` y el aviso «Error al eliminar la cuenta».

#### TC-ADM-317 — Sin el permiso Eliminar no hay botón en la cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-039][hu] «Sin el permiso no hay botón»; [CU-ADM-37][cu] excepción paso 2; [RN-CTA-001][rn] |

**Precondiciones:** cuenta con `Seguridad › Leer` y `Actualizar` sin `Eliminar`.
**Datos:** `DELETE /cuentas/<id>` con su token.
**Pasos:**
1. Abre una cuenta en modo editar.
2. Llama a la API.
**Resultado esperado:** no ves «Eliminar»; la API responde 403 sin `code`.

#### TC-ADM-318 — Un usuario puede eliminar su propia cuenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-039][hu] nota «Brecha conocida» (sin escenario propio); [CU-ADM-37][cu] flujo básico; [RN-CTA-013][rn] |

**Precondiciones:** una cuenta de prueba descartable con `Seguridad › Eliminar` y un administrador aparte para verificar. No uses la cuenta de Ana.
**Datos:** la propia cuenta del tester.
**Pasos:**
1. Con esa cuenta abre su propia ficha, «Editar», «Eliminar» y «Confirmar».
2. Con el administrador, busca la cuenta.
**Resultado esperado:** Hallazgo: ninguna regla lo impide (hallazgo 7 de la sección 12.3 del catálogo de reglas); la cuenta se elimina («Cuenta eliminada correctamente») y su sesión deja de servir. Los botones del cuadro son los de defecto «Confirmar» y «Cancelar».

## 4.8 Series fiscales

### HU-ADM-040 — Buscar series fiscales y exportarlas a Excel

#### TC-ADM-319 — Buscar series fiscales sin filtros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-040][hu] «Buscar sin filtros»; [CU-ADM-17][cu] flujo básico; [RN-SER-001, RN-GEN-001, RN-GEN-009][rn] |

**Precondiciones:** la empresa A tiene las series B001 y BC01 en «Sede Principal» y B002 y BC02 en «Sucursal Demo Norte»; sesión de Ana. Las series se crean con TC-ADM-328.
**Datos:** ninguno.
**Pasos:**
1. Abre `/series-fiscales` (menú SUNAT › «Series Fiscales») y comprueba que no hay tabla.
2. Haz clic en «Buscar».
**Resultado esperado:** «Resultados (4)» ordenadas por prefijo con las columnas «Serie», «Tipo de Comprobante», «Sucursal», «Correlativo Actual» y «Estado».

#### TC-ADM-320 — Filtros de series fiscales que se combinan

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-040][hu] «Filtros que se combinan»; [CU-ADM-17][cu] flujo básico |

**Precondiciones:** las cuatro series de TC-ADM-319.
**Datos:** Tipo «Nota de Crédito» y Sucursal «Sucursal Demo Norte»; Prefijo de Serie `c0`; Estado.
**Pasos:**
1. Busca con tipo y sucursal.
2. Limpia y busca con el prefijo `c0`.
3. Abre el selector «Estado».
**Resultado esperado:** con tipo y sucursal ves solo BC02; el filtro «Prefijo de Serie» (máximo 4 caracteres) busca prefijos que contienen el texto, sin distinguir mayúsculas: `c0` encuentra BC01 y BC02; «Estado» ofrece «Todos», «Activa» e «Inactiva».

#### TC-ADM-321 — Sin series

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-040][hu] «Sin series»; [CU-ADM-17][cu] flujo básico |

**Precondiciones:** una empresa sin series (por ejemplo, recién registrada).
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Buscar».
**Resultado esperado:** «Resultados (0)» y «Sin resultados».

#### TC-ADM-322 — Las series de una sucursal eliminada no aparecen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-040][hu] «Las series de una sucursal eliminada no aparecen»; [CU-ADM-17][cu] flujo básico; [RN-SER-014][rn] |

**Precondiciones:** la validación impide eliminar una sucursal con series; se prepara por base de datos (marcar eliminada la sucursal de una serie de prueba). No verificado en la aplicación.
**Datos:** serie de la sucursal marcada como eliminada.
**Pasos:**
1. Busca en `/series-fiscales`.
**Resultado esperado:** esa serie no aparece.

#### TC-ADM-323 — Recordar los filtros al volver de una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-040][hu] «Recordar filtros al volver»; [CU-ADM-17][cu] flujo básico |

**Precondiciones:** filtros escritos y tabla visible.
**Datos:** ninguno.
**Pasos:**
1. Abre una serie y vuelve con la flecha.
2. Recarga el navegador y entra otra vez.
**Resultado esperado:** al volver, la pantalla recuerda los filtros y repite la búsqueda; tras recargar el navegador ya no.

#### TC-ADM-324 — Limpiar la búsqueda de series fiscales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-040][hu] «Limpiar»; [CU-ADM-17][cu] A1 |

**Precondiciones:** filtros y tabla visibles.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Limpiar».
**Resultado esperado:** los filtros quedan vacíos y la tabla se oculta.

#### TC-ADM-325 — Exportar series fiscales a Excel

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-040][hu] «Exportar»; [CU-ADM-17][cu] flujo básico (exportación); [RN-IMP-008, RN-IMP-009, RN-IMP-011, RN-IMP-017][rn] |

**Precondiciones:** ya buscaste; idioma español.
**Datos:** sin filtros.
**Pasos:**
1. Haz clic en «Exportar» y abre el archivo.
**Resultado esperado:** se descarga `series-fiscales.xlsx` con las columnas «Serie», «Tipo de Comprobante», «Sucursal», «Correlativo Actual», «Estado» y «Fecha de Creación»; los textos de tipo y estado y el formato de fecha (dd/MM/yyyy en español, en hora UTC) los envía el cliente. No verificado en la aplicación: el contenido del archivo; el servidor respondió 200 (manual cap. 2, §11.1).

#### TC-ADM-326 — Formato de fecha inválido en la exportación de series

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-040][hu] «Formato de fecha inválido en la exportación»; [CU-ADM-17][cu] excepción paso 7; [RN-IMP-011][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /series-fiscales/exportar` con `dateFormat` `yyyy-MM-dd` (el cuerpo completo sale de una exportación hecha desde el SPA); repite con `dd/MM/yyyy HH:mm`.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** con `yyyy-MM-dd` la API responde 400 sin campo `code`; la pantalla, si lo recibiera, muestra su genérico «Error al exportar las series fiscales»; con `dd/MM/yyyy HH:mm` (formato admitido, junto con `dd/MM/yyyy`, `MM/dd/yyyy` y `MM/dd/yyyy HH:mm`) responde 200. No verificado: el nombre exacto de la propiedad del formato de fecha en el cuerpo.

#### TC-ADM-327 — Errores al buscar o exportar series fiscales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-040][hu] «Errores»; [CU-ADM-17][cu] excepciones pasos 1, 3 y 7; [RN-IMP-017][rn] |

**Precondiciones:** bloqueos de red; las más de 10 000 filas son No verificado (ver HU-ADM-047).
**Datos:** ninguno.
**Pasos:**
1. Bloquea `GET /series-fiscales` y haz clic en «Buscar».
2. Con búsqueda hecha, bloquea `POST /series-fiscales/exportar` y haz clic en «Exportar».
3. Con una cuenta sin `Sucursales › Leer` abre la pantalla.
**Resultado esperado:** «Error al buscar series fiscales» y «Error al exportar las series fiscales»; sin `Sucursales › Leer` el filtro de sucursal queda sin opciones y se ve «No se pudieron cargar las sucursales».

### HU-ADM-041 — Crear una serie fiscal

#### TC-ADM-328 — Crear la serie de boletas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-041][hu] «Crear la serie de boletas»; [CU-ADM-18][cu] flujo básico; [RN-SER-001, RN-SER-002, RN-SER-003, RN-SER-004, RN-SER-007, RN-SER-008, RN-GEN-013][rn] |

**Precondiciones:** Ana con `SeriesFiscales › Crear` y `Sucursales › Leer`; existe «Sede Principal».
**Datos:** Tipo «Boleta» (por defecto), Correlativo Actual 0, Estado «Activa»; Sucursal «Sede Principal»; Prefijo de Serie `B001`.
**Pasos:**
1. Abre `/series-fiscales/nueva` (botón «Nuevo»), elige la sucursal, escribe el prefijo y haz clic en «Guardar».
2. Repite para crear BC01 (Nota de Crédito, Sede Principal), B002 (Boleta, Sucursal Demo Norte) y BC02 (Nota de Crédito, Sucursal Demo Norte).
**Resultado esperado:** aviso «Serie fiscal guardada correctamente»; el sistema te lleva a `/series-fiscales`; se registra `SERIE_CREAR` con tipo, prefijo, correlativo y estado.

#### TC-ADM-329 — Tipos de comprobante disponibles al crear una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-041][hu] «Tipos disponibles»; [CU-ADM-18][cu] flujo básico; [RN-SER-002][rn] |

**Precondiciones:** `/series-fiscales/nueva`.
**Datos:** ninguno.
**Pasos:**
1. Abre el selector «Tipo de Comprobante».
**Resultado esperado:** ofrece «Boleta», «Factura», «Nota de Crédito» y «Proforma»; «Ticket» y «Nota de débito» existen en el sistema pero no se aceptan.

#### TC-ADM-330 — Campos obligatorios al crear una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-041][hu] «Campos obligatorios»; [CU-ADM-18][cu] excepción paso 3; [RN-SER-003, RN-SER-004][rn] |

**Precondiciones:** `/series-fiscales/nueva` como se abre.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Guardar».
**Resultado esperado:** «Sucursal» y «Prefijo de Serie» muestran «Este campo es obligatorio» y no se envía nada.

#### TC-ADM-331 — El prefijo debe tener 4 caracteres y la letra del tipo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-ADM-041][hu] «El prefijo debe tener 4 caracteres y la letra del tipo» (esquema); [CU-ADM-18][cu] excepción paso 3; [RN-SER-003, RN-SER-004][rn] |

**Precondiciones:** `/series-fiscales/nueva`.
**Datos:** una fila por vez; elige el tipo, escribe el prefijo y sal del campo.
**Pasos:**
1. Para cada fila de la tabla elige el tipo y escribe el prefijo.
**Resultado esperado:** el mensaje completo es «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.»

| Tipo | Prefijo | Resultado |
|---|---|---|
| Boleta | `B001` | no hay error |
| Boleta | `F001` | ves el mensaje |
| Boleta | `b00` | el campo muestra `B00` y ves el mismo mensaje |
| Boleta | `B-01` | ves el mismo mensaje |
| Factura | `F001` | no hay error |
| Factura | `BC01` | ves el mismo mensaje |
| Nota de Crédito | `BC01` | no hay error (acepta B o F) |
| Proforma | `PF01` | no hay error (sin restricción de letra) |

#### TC-ADM-332 — El prefijo se reevalúa al cambiar el tipo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-041][hu] «El prefijo se reevalúa al cambiar el tipo»; [CU-ADM-18][cu] excepción paso 3; [RN-SER-004][rn] |

**Precondiciones:** `/series-fiscales/nueva`.
**Datos:** prefijo `BC01`; tipos «Nota de Crédito», «Boleta» y «Factura».
**Pasos:**
1. Escribe `BC01` con «Nota de Crédito» y luego con «Boleta».
2. Cambia el tipo a «Factura».
**Resultado esperado:** sin error con «Nota de Crédito» y «Boleta»; al cambiar a «Factura» aparece el mensaje del prefijo.

#### TC-ADM-333 — El prefijo se convierte a mayúsculas y muestra su ayuda

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-041][hu] «El prefijo se convierte a mayúsculas y tiene ayuda»; [CU-ADM-18][cu] flujo básico; [RN-SER-004][rn] |

**Precondiciones:** `/series-fiscales/nueva`.
**Datos:** `b001` y un texto de 6 caracteres.
**Pasos:**
1. Escribe `b001`.
2. Intenta escribir más de 4 caracteres.
3. Lee la ayuda bajo el campo.
**Resultado esperado:** el campo lo muestra como `B001` y no deja escribir más de 4 caracteres; debajo se lee «4 caracteres en mayúsculas. Boleta empieza con B, Factura con F, Nota de Crédito con B o F.»

#### TC-ADM-334 — Validación del correlativo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-ADM-041][hu] «Correlativo»; [CU-ADM-18][cu] excepción paso 3 y A2; [RN-SER-005, RN-SER-006][rn] |

**Precondiciones:** `/series-fiscales/nueva` con sucursal y prefijo válidos.
**Datos:** `-3`; `1.5`; vacío; `0`; `250`.
**Pasos:**
1. Escribe cada valor en «Correlativo Actual» y haz clic en «Guardar».
2. Lee la ayuda del campo.
**Resultado esperado:** la ayuda dice «Último número emitido; el siguiente comprobante usará este valor + 1.»; con `-3` y `1.5` ves «El correlativo debe ser un número entero mayor o igual a 0.»; vacío muestra «Este campo es obligatorio»; con `0` y `250` se guarda (con 250, el siguiente comprobante será el 251; ver TC-ADM-355).

#### TC-ADM-335 — Estado de la serie al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-041][hu] «Estado»; [CU-ADM-18][cu] flujo básico; [RN-SER-007][rn] |

**Precondiciones:** `/series-fiscales/nueva`.
**Datos:** ninguno.
**Pasos:**
1. Abre el selector «Estado».
**Resultado esperado:** ofrece solo «Activa» e «Inactiva» y empieza en «Activa».

#### TC-ADM-336 — El prefijo ya existe para el mismo tipo en la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-041][hu] «El prefijo ya existe para el mismo tipo en la empresa»; [CU-ADM-18][cu] excepción paso 6; [RN-SER-008][rn] |

**Precondiciones:** la «Sede Principal» tiene la Boleta `B001`.
**Datos:** Boleta `B001` para «Sucursal Demo Norte»; Factura `B001`... (tipo distinto) con prefijo permitido: Nota de Crédito `B001` en «Sucursal Demo Norte».
**Pasos:**
1. Crea la Boleta `B001` en «Sucursal Demo Norte».
2. Crea una Nota de Crédito `B001` en «Sucursal Demo Norte».
**Resultado esperado:** el paso 1 muestra «Ya existe otra serie de este tipo de comprobante con ese prefijo.» (409 `SerieFiscal.PrefijoDuplicado`); el paso 2 se guarda: un mismo prefijo sí puede usarse en tipos distintos.

#### TC-ADM-337 — Solo una serie activa por sucursal y tipo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-041][hu] «Solo una serie activa por sucursal y tipo»; [CU-ADM-18][cu] excepción paso 6 y A3; [RN-SER-008, RN-SER-012][rn] |

**Precondiciones:** la «Sede Principal» tiene una Boleta «Activa».
**Datos:** Boleta `B009` «Activa» en la misma sucursal; luego `B009` «Inactiva».
**Pasos:**
1. Intenta guardar `B009` como «Activa».
2. Guárdala como «Inactiva».
**Resultado esperado:** el paso 1 muestra «La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra.» (409 `SerieFiscal.ActivaDuplicada`); el paso 2 se guarda: no hay límite de series inactivas. Hallazgo: el SPA no anticipa la regla hasta que la API la rechaza.

#### TC-ADM-338 — Prefijo rechazado por el servidor al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-041][hu] «Prefijo rechazado por el servidor»; [CU-ADM-18][cu] excepción paso 5; [RN-SER-004][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /series-fiscales` con `{ "sucursalId": "<id>", "tipoComprobante": "Boleta", "prefijoSerie": "F001", "correlativoActual": 0, "estadoSerie": "Activa" }`.
**Pasos:**
1. Envía la petición.
**Resultado esperado:** 400 `CreateSerieFiscalCommand.PrefijoSerie`; el SPA mostraría «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.» No verificado: la forma exacta de los valores de `tipoComprobante` y `estadoSerie` en el cuerpo.

#### TC-ADM-339 — Sucursal inexistente al crear una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-041][hu] «Sucursal inexistente»; [CU-ADM-18][cu] excepción paso 6; [RN-SER-001][rn] |

**Precondiciones:** `/series-fiscales/nueva` con sucursal elegida que otra sesión elimina antes de guardar (la sucursal no debe tener dependencias).
**Datos:** sucursal «Sucursal Prueba Eliminar».
**Pasos:**
1. Elige la sucursal.
2. En otra sesión elimina la sucursal.
3. Guarda la serie.
**Resultado esperado:** 404 `Sucursal.NotFound` y «La sucursal seleccionada no existe.»

#### TC-ADM-340 — Otros errores al guardar una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-041][hu] «Otros errores»; [CU-ADM-18][cu] excepciones pasos 5 y 7; [CU-ADM-17][cu] excepción paso 1 |

**Precondiciones:** bloqueos de red.
**Datos:** ninguno.
**Pasos:**
1. Bloquea `POST /series-fiscales` y guarda un formulario válido.
2. Bloquea `GET /sucursales` y abre `/series-fiscales/nueva`.
**Resultado esperado:** «Error al guardar la serie fiscal»; con el selector sin datos ves «No se pudieron cargar las sucursales».

### HU-ADM-042 — Editar y desactivar una serie fiscal

#### TC-ADM-341 — Ver y editar una serie sin comprobantes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-042][hu] «Ver y editar una serie sin comprobantes»; [CU-ADM-19][cu] flujo básico; [RN-SER-002, RN-SER-004, RN-SER-007, RN-SER-008][rn] |

**Precondiciones:** la serie `PF01` (Proforma) de «Sede Principal» sin comprobantes, creada para esta prueba.
**Datos:** prefijo `PF02`; Correlativo Actual 5.
**Pasos:**
1. Abre la serie, «Editar», cambia prefijo y correlativo y haz clic en «Guardar».
**Resultado esperado:** aviso «Serie fiscal guardada correctamente»; el sistema te lleva a `/series-fiscales`, donde la tabla muestra PF02, Proforma, Sede Principal, 5, Activa; se registra `SERIE_ACTUALIZAR`.

#### TC-ADM-342 — Desactivar y reactivar una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-042][hu] «Desactivar una serie»; [CU-ADM-19][cu] flujo básico y A1; [RN-SER-007, RN-SER-012][rn] |

**Precondiciones:** la Boleta `B001` de «Sede Principal» activa; sin otra Boleta activa en esa sucursal.
**Datos:** Estado «Inactiva» y luego «Activa».
**Pasos:**
1. Edita la serie, elige «Inactiva» y guarda.
2. Edita otra vez, elige «Activa» y guarda.
**Resultado esperado:** «Serie fiscal guardada correctamente» y la tabla muestra el estado «Inactiva»; mientras no haya otra serie activa del mismo tipo en la sucursal, el punto de venta no podrá emitir ese tipo de comprobante; para volver a usarla se elige «Activa».

#### TC-ADM-343 — Activar otra serie del mismo tipo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-042][hu] «Activar otra serie del mismo tipo»; [CU-ADM-19][cu] excepción paso 6; [RN-SER-012][rn] |

**Precondiciones:** la sucursal tiene una Boleta «Activa» y otra «Inactiva» (`B009`).
**Datos:** Estado «Activa» en `B009`.
**Pasos:**
1. Edita `B009`, elige «Activa» y guarda.
**Resultado esperado:** «La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra.»

#### TC-ADM-344 — Prefijo repetido al editar una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-042][hu] «Prefijo repetido al editar»; [CU-ADM-19][cu] excepción paso 6; [RN-SER-008][rn] |

**Precondiciones:** dos Boletas, `B001` y `B002`.
**Datos:** prefijo `B001` en la serie `B002`.
**Pasos:**
1. Edita `B002`, cambia el prefijo a `B001` y guarda.
**Resultado esperado:** «Ya existe otra serie de este tipo de comprobante con ese prefijo.»

#### TC-ADM-345 — Una serie que ya emitió comprobantes solo cambia de estado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-042][hu] «Serie que ya emitió comprobantes»; [CU-ADM-19][cu] flujo básico; [RN-SER-009, RN-SER-010][rn] |

**Precondiciones:** una serie que ya emitió al menos un comprobante no eliminado (se necesita una venta cobrada; ver la parte C). No verificado en la aplicación: la empresa de prueba todavía no emitió comprobantes.
**Datos:** ninguno.
**Pasos:**
1. Abre la serie y haz clic en «Editar».
**Resultado esperado:** sobre el formulario se lee «Esta serie ya emitió comprobantes: solo puedes cambiar su estado.»; solo está habilitado «Estado»; «Sucursal», «Tipo de Comprobante», «Prefijo de Serie» y «Correlativo Actual» quedan bloqueados.

#### TC-ADM-346 — La API protege la numeración ya emitida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-042][hu] «La API protege la numeración ya emitida»; [CU-ADM-19][cu] excepción paso 5; [RN-SER-009, RN-SER-010][rn] |

**Precondiciones:** Solo API; serie con comprobantes (ver TC-ADM-345).
**Datos:** `PUT /series-fiscales/<id>` cambiando el prefijo, otra vez el correlativo, y otra solo el estado.
**Pasos:**
1. Envía las tres peticiones.
**Resultado esperado:** al cambiar sucursal, tipo, prefijo o correlativo responde 409 `SerieFiscal.ConComprobantes` y la pantalla muestra «La serie ya emitió comprobantes: solo puedes cambiar su estado.»; al cambiar solo el estado responde 204.

#### TC-ADM-347 — Cancelar la edición de una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-042][hu] «Cancelar»; [CU-ADM-19][cu] A2 |

**Precondiciones:** serie en modo editar con cambios.
**Datos:** ninguno.
**Pasos:**
1. Haz clic en «Cancelar».
**Resultado esperado:** se restaura lo leído y el formulario vuelve a deshabilitarse.

#### TC-ADM-348 — Serie o sucursal inexistente al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-042][hu] «Serie o sucursal inexistente»; [CU-ADM-19][cu] excepciones pasos 1, 5 y 6; [RN-SER-001][rn] |

**Precondiciones:** dos sesiones; una elimina la serie (o la sucursal de destino) mientras la otra edita.
**Datos:** `/series-fiscales/<id inexistente>`.
**Pasos:**
1. Abre la dirección con un identificador inexistente.
2. Edita una serie que otra sesión eliminó y guarda.
3. Cambia la sucursal de una serie a una eliminada en otra sesión y guarda.
**Resultado esperado:** «Error al cargar la serie fiscal» al abrir; «La serie fiscal no existe.» (404 `SerieFiscal.NotFound`) y «La sucursal seleccionada no existe.» (404 `Sucursal.NotFound`) al guardar.

### HU-ADM-043 — Eliminar una serie fiscal

#### TC-ADM-349 — Eliminar una serie sin comprobantes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-043][hu] «Eliminar una serie sin comprobantes»; [CU-ADM-20][cu] flujo básico; [RN-SER-009, RN-SER-011, RN-GEN-003][rn] |

**Precondiciones:** la serie `PF02` sin comprobantes.
**Datos:** ninguno.
**Pasos:**
1. Abre la serie, «Editar» y «Eliminar».
2. Lee el cuadro y haz clic en «Confirmar».
**Resultado esperado:** cuadro «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta serie fiscal?»; «Serie fiscal eliminada correctamente»; vuelves a la búsqueda, donde ya no aparece; el borrado es lógico y se registra `SERIE_ELIMINAR`.

#### TC-ADM-350 — No se elimina una serie con comprobantes emitidos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-043][hu] «Serie con comprobantes emitidos»; [CU-ADM-20][cu] excepción paso 3 y A2; [RN-SER-009, RN-SER-011][rn] |

**Precondiciones:** serie con comprobantes (ver TC-ADM-345). No verificado en la aplicación: no hay una serie con comprobantes en la empresa de prueba.
**Datos:** ninguno.
**Pasos:**
1. Intenta eliminarla y confirma.
**Resultado esperado:** 409 `SerieFiscal.InUse` y «No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla.»

#### TC-ADM-351 — Otros errores al eliminar una serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-043][hu] «Otros errores»; [CU-ADM-20][cu] excepción paso 2 |

**Precondiciones:** bloqueo de red de `DELETE /series-fiscales/<id>`.
**Datos:** ninguno.
**Pasos:**
1. Intenta eliminar una serie con el bloqueo activo.
**Resultado esperado:** «Error al eliminar la serie fiscal».

#### TC-ADM-352 — Eliminar una serie inexistente o de otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-043][hu] «Serie inexistente»; [CU-ADM-20][cu] excepción paso 3; [RN-SER-001, RN-GEN-001][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; identificador de una serie de la empresa B.
**Datos:** `DELETE /series-fiscales/<id de B>`.
**Pasos:**
1. Envía la petición.
2. Comprueba la serie desde la empresa B.
**Resultado esperado:** 404 `SerieFiscal.NotFound`; la serie de B sigue existiendo; en el SPA se vería «Error al eliminar la serie fiscal».

#### TC-ADM-353 — Una sucursal con series no se puede eliminar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-043][hu] «Una sucursal con series no se puede eliminar»; [CU-ADM-14][cu] excepción paso 4 y A2; [RN-SUC-009][rn] |

**Precondiciones:** «Sucursal Demo Norte» sin otros registros pero con la serie B002.
**Datos:** ninguno.
**Pasos:**
1. Intenta eliminar la sucursal y confirma.
2. Elimina primero sus series (sin comprobantes) y vuelve a intentarlo.
**Resultado esperado:** 409 `Sucursal.InUse` con el aviso «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.»; tras eliminar las series (y si no hay otras dependencias) la eliminación procede.

### HU-ADM-044 — Numerar los comprobantes sin repetir números

#### TC-ADM-354 — El siguiente número es el correlativo más uno

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | [HU-ADM-044][hu] «El siguiente número es el correlativo más uno»; [CU-ADM-18][cu] flujo básico (nota de la historia); [RN-SER-005, RN-SER-013][rn] |

**Precondiciones:** la Boleta `B001` de «Sede Principal» activa con «Correlativo Actual» 0; Carlos o Ana con caja abierta en «Sede Principal» y un producto con stock (preparación de la parte C: ventas y caja).
**Datos:** una venta cobrada en efectivo.
**Pasos:**
1. Cobra la primera venta.
2. Abre la venta y mira su comprobante.
3. Busca la serie en `/series-fiscales`.
**Resultado esperado:** la primera boleta recibe el número 1 (formato `B001-00000001`) y el «Correlativo Actual» pasa a 1.

#### TC-ADM-355 — Continuar una numeración anterior

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | [HU-ADM-044][hu] «Continuar una numeración anterior»; [CU-ADM-18][cu] A2; [RN-SER-005, RN-SER-006][rn] |

**Precondiciones:** una serie nueva de Boleta con «Correlativo Actual» 250 en una sucursal sin otra Boleta activa; caja abierta en esa sucursal.
**Datos:** una venta cobrada.
**Pasos:**
1. Crea la serie con 250 (TC-ADM-334).
2. Cobra una venta.
3. Mira el comprobante y la serie.
**Resultado esperado:** la siguiente boleta recibe el número 251 y el correlativo actual pasa a 251.

#### TC-ADM-356 — Dos emisiones simultáneas nunca repiten número

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-ADM-044][hu] «Dos emisiones simultáneas»; [CU-ADM-18][cu] flujo básico; [RN-SER-013][rn] |

**Precondiciones:** dos cajeros (o dos clientes HTTP) con caja abierta en la misma sucursal, cada uno con una venta lista para cobrar. No verificado en la aplicación: el comportamiento concurrente viene del código (`ApplicationDbContext.cs:180-188`, citado en RN-SER-013).
**Datos:** dos cobros lanzados a la vez.
**Pasos:**
1. Cobra las dos ventas al mismo tiempo.
2. Compara los números de comprobante.
**Resultado esperado:** cada una recibe un número distinto y consecutivo; no hay duplicados (además existe un índice único por serie y correlativo).

## 4.9 Reglas transversales y de plataforma

Las historias de esta sección no tienen caso de uso propio: sus reglas se citan en los pasos de guardado de cada caso de uso (ver el [documento de casos de uso, sección 12.2][cu]). Donde un caso dice «Transversal» en vez de un `CU-ADM`, es por eso.

### HU-ADM-045 — Aislar los datos de cada empresa y conservar el historial

#### TC-ADM-357 — Un registro de otra empresa se comporta como inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-045][hu] «Un registro de otra empresa se comporta como inexistente»; Transversal; [RN-GEN-001, RN-GEN-002, RN-EMP-017][rn] |

**Precondiciones:** Solo API; `<token>` de Ana (empresa A) y, de la empresa B, los identificadores de: empresa, sucursal, empleado, rol, cuenta y serie fiscal.
**Datos:** las siete llamadas de la tabla.
**Pasos:**
1. Envía cada petición con el token de Ana.
**Resultado esperado:** la API responde como si el registro no existiera:

| Petición | Respuesta |
|---|---|
| `GET /empresas/<id de B>` | 404 `Empresa.NotFound` |
| `GET /sucursales/<id de B>` | 404 `Sucursal.NotFound` |
| `GET /empleados/<id de B>` | 404 `Empleado.NotFound` |
| `GET /roles/<id de B>` | 404 `Rol.NotFound` |
| `GET /cuentas/<id de B>` | 404 `Cuenta.NotFound` |
| `GET /series-fiscales/<id de B>` | 404 `SerieFiscal.NotFound` |

#### TC-ADM-358 — Las listas solo traen filas de mi empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-045][hu] «Las consultas se limitan a mi empresa»; Transversal; [RN-GEN-001, RN-GEN-002][rn] |

**Precondiciones:** ambas empresas con datos propios (sucursales, empleados, roles, cuentas, series).
**Datos:** búsquedas sin filtros en cada pantalla de administración.
**Pasos:**
1. Con la sesión de Ana busca en Sucursales, Empleados, Roles, Cuentas de Usuario y Series Fiscales.
2. Con la sesión de `luis.dos` repite.
**Resultado esperado:** cada sesión recibe solo filas de su empresa; ninguna fila aparece en ambas listas. No verificado: que `GET /empresas` muestre solo la empresa propia depende del filtro global y no de un filtro explícito del handler ([19b][p19b] §7, punto 1).

#### TC-ADM-359 — Los flujos anteriores a conocer la empresa funcionan sin sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-045][hu] «Las excepciones son los flujos anteriores a conocer la empresa»; [CU-ADM-02][cu] flujo básico; [CU-ADM-01][cu] flujo básico; [RN-GEN-001, RN-GEN-002][rn] |

**Precondiciones:** sin sesión.
**Datos:** inicio de sesión, renovación, registro de empresa y recuperación de contraseña.
**Pasos:**
1. Inicia sesión como `ana.manual` (TC-ADM-012).
2. Renueva la sesión (TC-ADM-031).
3. Registra una empresa nueva (TC-ADM-001).
4. Pide el enlace de recuperación (TC-ADM-065).
**Resultado esperado:** los cuatro flujos encuentran las cuentas y empresas sin que la petición traiga empresa (saltan el filtro de empresa de forma explícita), mientras que cualquier otra consulta anónima no devuelve filas.

#### TC-ADM-360 — La empresa se asigna sola al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-045][hu] «La empresa se asigna sola al crear»; Transversal; [RN-GEN-004, RN-GEN-005][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos.
**Datos:** una sucursal, un empleado, un rol, una cuenta y una serie recién creados por Ana; la empresa registrada en TC-ADM-001.
**Pasos:**
1. Consulta el campo `TenantId` de cada fila nueva.
**Resultado esperado:** las filas creadas por Ana llevan el identificador de la empresa A (el de su token) sin que lo haya indicado; las filas del registro público llevan el de la empresa nueva (si el registro ya trae empresa, se respeta).

#### TC-ADM-361 — Eliminar es un borrado lógico

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-045][hu] «Eliminar es un borrado lógico»; Transversal; [RN-GEN-003][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos; una sucursal, un empleado, una cuenta, un rol y una serie descartables.
**Datos:** elimina cada uno desde su pantalla (TC-ADM-154, 203, 255, 312, 349).
**Pasos:**
1. Tras eliminar, consulta cada fila en la base de datos.
2. Busca en la pantalla.
**Resultado esperado:** la fila no se borra: queda marcada como eliminada, con fecha UTC y con el usuario del token (o `SYSTEM` si no hay); las lecturas ya no la devuelven.

#### TC-ADM-362 — Quién y cuándo se guardan al crear o modificar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-045][hu] «Quién y cuándo»; Transversal; [RN-GEN-005][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos.
**Datos:** una sucursal creada y luego editada por Ana.
**Pasos:**
1. Consulta `CreatedBy`, `CreatedAt`, `LastModifiedBy` y `LastModifiedAt`.
**Resultado esperado:** quedan el usuario del token (o `SYSTEM`) y la fecha en UTC de cada acción.

#### TC-ADM-363 — Las fechas se guardan en UTC

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-045][hu] «Fechas en UTC»; Transversal; [RN-GEN-006][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos; el navegador en la zona horaria de Perú (UTC-5).
**Datos:** la fecha de bloqueo `Bloqueado_Hasta` de una cuenta bloqueada (TC-ADM-026) y `CreatedAt` de una fila nueva.
**Pasos:**
1. Compara la hora local con el valor guardado.
2. Mira «Bloqueado Hasta» en la ficha de la cuenta.
**Resultado esperado:** el valor guardado está en UTC (5 horas por delante de la hora de Perú); un valor sin zona se interpreta como UTC. No verificado en la aplicación: cómo muestra la ficha la zona horaria de «Bloqueado Hasta».

#### TC-ADM-364 — Los filtros de fecha de la empresa son inclusivos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-045][hu] «Filtros de fechas inclusivos»; [CU-ADM-09][cu] flujo básico (paso 4); [RN-GEN-008][rn] |

**Precondiciones:** la empresa A vence un día conocido D (ver la ficha).
**Datos:** «Vencimiento desde» = D; «Vencimiento hasta» = D; «Vencimiento hasta» = D − 1 día.
**Pasos:**
1. Busca con cada combinación.
**Resultado esperado:** ambos extremos son inclusivos (mayor o igual y menor o igual), normalizados a UTC; la fecha elegida se envía a medianoche local, sin ajustar al fin del día, por lo que un vencimiento a mitad del día D puede quedar fuera con «Vencimiento hasta» = D. Registra lo que ves (Hallazgo en [CU-ADM-09][cu], «Brecha conocida»).

#### TC-ADM-365 — Los eventos de dominio pasan a la bandeja de salida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Baja | [HU-ADM-045][hu] «Eventos de dominio a la bandeja de salida»; Transversal; [RN-GEN-007][rn] |

**Precondiciones:** No verificado en la aplicación: no hay pantalla ni endpoint que lea la bandeja de salida y las entidades de esta parte no declaran eventos de dominio; se respalda en el código y en las pruebas automáticas ([09a][p09a]).
**Datos:** ninguno.
**Pasos:**
1. Si el equipo técnico dispone de una operación que genere eventos, guarda y consulta la tabla de la bandeja de salida.
**Resultado esperado:** cada evento se convierte en una fila de la bandeja y se limpia de la entidad; una fila que no pasa su validación se descarta sin error.

### HU-ADM-046 — Cargar un Excel con la pantalla de carga masiva

#### TC-ADM-366 — Los elementos de la pantalla de carga masiva

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-046][hu] «La pantalla de carga masiva»; [CU-ADM-15][cu] flujo básico (pasos 1 a 3); [RN-IMP-016][rn] |

**Precondiciones:** Ana en `/sucursales/carga-masiva` y en `/roles/carga-masiva`.
**Datos:** ninguno.
**Pasos:**
1. Mira la pantalla en ambas rutas.
**Resultado esperado:** ves la flecha de volver, el título «Carga Masiva», el botón «Descargar plantilla» y una zona que dice «Arrastra tu archivo Excel aquí o haz clic para seleccionar» con «Solo archivos .xlsx (máx. 5 MB y 5000 filas)».

#### TC-ADM-367 — La plantilla y el error de descarga

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-046][hu] «La plantilla»; [CU-ADM-15][cu] flujo básico (paso 1) y excepción paso 1; [RN-IMP-016][rn] |

**Precondiciones:** pantalla de carga masiva.
**Datos:** bloqueo de red de la llamada de la plantilla para la segunda parte.
**Pasos:**
1. Haz clic en «Descargar plantilla» y abre el libro.
2. Bloquea `GET /sucursales/plantilla` y vuelve a hacer clic.
**Resultado esperado:** un libro con una hoja «Plantilla» que solo contiene la fila de cabecera con los nombres de columna del módulo, en negrita, texto blanco sobre azul y centrado; con el bloqueo ves «Error al descargar la plantilla. Intenta de nuevo.»

#### TC-ADM-368 — Archivo que no es .xlsx

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-046][hu] «Archivo que no es .xlsx»; [CU-ADM-15][cu] excepción paso 3; [RN-IMP-001][rn] |

**Precondiciones:** un archivo `datos.csv` y otro `datos.XLSX` válido (mayúsculas).
**Datos:** ambos archivos.
**Pasos:**
1. Elige cada archivo.
2. Observa la pestaña de red.
**Resultado esperado:** con `datos.csv` ves «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» y el navegador no lo envía; la extensión se compara sin distinguir mayúsculas (`.XLSX` sí se acepta).

#### TC-ADM-369 — Archivo renombrado o dañado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-046][hu] «Archivo renombrado o dañado»; [CU-ADM-15][cu] excepción paso 5; [RN-IMP-001, RN-IMP-002][rn] |

**Precondiciones:** un `.txt` renombrado a `falso.xlsx`; un `.xlsx` truncado.
**Datos:** ambos archivos.
**Pasos:**
1. Elige cada archivo.
**Resultado esperado:** la API responde 400 `CargaMasiva.Archivo.Invalido` y ves «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.»

#### TC-ADM-370 — Archivo de más de 5 MB

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-046][hu] «Archivo de más de 5 MB»; [CU-ADM-15][cu] excepción paso 3; [RN-IMP-004, RN-GEN-039][rn] |

**Precondiciones:** un `.xlsx` de más de 5 MB; No verificado en la aplicación (los errores de tamaño no se provocaron).
**Datos:** el archivo; además un `POST …/carga-masiva` con un cuerpo JSON de más de 5 MB (Solo API).
**Pasos:**
1. Elige el archivo.
2. Envía el cuerpo grande por API.
**Resultado esperado:** ves «El archivo supera el tamaño máximo de 5 MB.»; el servidor responde 413 `Request.TooLarge` y la pantalla muestra el mismo texto.

#### TC-ADM-371 — Más de 5000 filas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-046][hu] «Más de 5000 filas»; [CU-ADM-15][cu] excepciones pasos 5 y 9; [RN-IMP-005, RN-IMP-006][rn] |

**Precondiciones:** un `.xlsx` con 5000 filas de datos y otro con 5001 (sin contar cabecera ni filas vacías); No verificado en la aplicación.
**Datos:** ambos archivos; `POST /sucursales/carga-masiva` con `rows` vacío y con 5001 elementos.
**Pasos:**
1. Elige cada archivo.
2. Envía las dos peticiones de API.
**Resultado esperado:** con 5000 la vista previa se muestra; con 5001 la API responde 400 `CargaMasiva.Filas.Excedidas` y ves «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.»; al guardar, una lista vacía o de más de 5000 elementos da el mismo texto (el SPA asocia ese texto a cualquier código que termine en `.Rows`).

#### TC-ADM-372 — Archivo comprimido de forma sospechosa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-046][hu] «Archivo comprimido de forma sospechosa»; [CU-ADM-15][cu] excepción paso 5; [RN-IMP-003][rn] |

**Precondiciones:** No verificado en la aplicación: no se provocó; requiere fabricar un `.xlsx` cuyo contenido descomprimido supere 50 MB o con una parte de más de 1 MB que se expanda más de 100 veces. No lo descargues de fuentes externas: genéralo tú.
**Datos:** el archivo fabricado.
**Pasos:**
1. Elige el archivo en la pantalla.
**Resultado esperado:** la API responde 400 `CargaMasiva.Archivo.ExcedeDescompresion` y ves «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.»

#### TC-ADM-373 — Cómo se lee el archivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-046][hu] «Cómo se lee el archivo»; [CU-ADM-15][cu] flujo básico (paso 5); [RN-IMP-012, RN-IMP-013][rn] |

**Precondiciones:** un libro de sucursales con dos hojas, una cabecera con un título repetido y con espacios y mayúsculas distintas, una columna extra, una fila totalmente vacía entre dos filas y valores con espacios sobrantes.
**Datos:** el libro armado según la tabla.
**Pasos:**
1. Elige el libro y mira la vista previa.
**Resultado esperado:**

| Particularidad del libro | Resultado |
|---|---|
| Varias hojas | se lee solo la primera |
| Cabecera con mayúsculas o espacios distintos (` nombre `, `NOMBRE`) | se reconoce igual |
| Título repetido | vale la primera columna |
| Columna extra | se ignora |
| Columna faltante (por ejemplo sin `NumeroLicencia`) | se lee como nula y cada fila marca «El campo 'numeroLicencia' es obligatorio» |
| Valores con espacios al inicio o al final | se leen sin los espacios |
| Fila totalmente vacía | se ignora y no cuenta en «filas totales» |
| Número de fila | cada fila conserva su número real de la hoja |

#### TC-ADM-374 — La vista previa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-046][hu] «La vista previa»; [CU-ADM-15][cu] flujo básico (pasos 6 y 7); [RN-IMP-014][rn] |

**Precondiciones:** un archivo de roles con filas válidas y con error, de más de 25 filas.
**Datos:** el archivo; después otro archivo.
**Pasos:**
1. Elige el primer archivo y revisa encabezados y contadores.
2. Revisa si la tabla tiene paginación.
3. Elige el segundo archivo.
**Resultado esperado:** ves «Vista previa», «N filas totales», «N filas válidas» y, si hay errores, «N filas con error»; cada fila trae sus errores y una columna «Estado» con marca verde si es válida o icono de error si no; la tabla no tiene paginación (se ven todas las filas); una vista previa nueva reemplaza a la anterior.

#### TC-ADM-375 — Mensaje «Procesando archivo...»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-046][hu] «Procesando»; [CU-ADM-15][cu] flujo básico (paso 4) |

**Precondiciones:** red limitada a «conexión lenta» en las herramientas del navegador. No verificado en la aplicación: con archivos pequeños la espera es demasiado breve.
**Datos:** cualquier archivo válido.
**Pasos:**
1. Elige el archivo y mira la pantalla mientras responde.
**Resultado esperado:** ves «Procesando archivo...» mientras responde la vista previa y no se guarda nada todavía.

#### TC-ADM-376 — Guardar solo las filas válidas (carga masiva común)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-046][hu] «Guardar solo las filas válidas»; [CU-ADM-15][cu] flujo básico (pasos 8 a 10); [CU-ADM-31][cu] flujo básico; [RN-IMP-006, RN-IMP-007, RN-IMP-015][rn] |

**Precondiciones:** un archivo de roles con 2 filas válidas y 1 con error.
**Datos:** el archivo; otro archivo sin filas válidas.
**Pasos:**
1. Elige el primer archivo, haz clic en «Guardar válidos (2)» y mira la red.
2. Elige el segundo y mira el botón.
**Resultado esperado:** el SPA envía solo las 2 filas válidas; ves «Se guardaron 2 registros correctamente» y vas a la lista del módulo; sin filas válidas el botón está deshabilitado (en el método, «No hay filas válidas para guardar»).

#### TC-ADM-377 — Al guardar se vuelve a validar cada fila (carga masiva común)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-046][hu] «Al guardar cada fila se valida de nuevo»; [CU-ADM-15][cu] excepción paso 9; [CU-ADM-31][cu] excepción paso 7; [RN-IMP-006, RN-IMP-007][rn] |

**Precondiciones:** Solo API.
**Datos:** `POST /sucursales/carga-masiva` con una fila válida y otra con nombre vacío; `POST /roles/carga-masiva` con una fila con nombre de 51 caracteres.
**Pasos:**
1. Envía cada petición y revisa después las listas.
**Resultado esperado:** no se guarda ninguna fila; el código indica la primera fila y campo inválidos (`CargaMasivaSucursalesCommand.Rows[1].Nombre`, `CargaMasivaRolesCommand.Rows[0].NombreRol`); el SPA mostraría «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» (casos de módulo: TC-ADM-167 y TC-ADM-266).

#### TC-ADM-378 — Errores genéricos de la vista previa y del guardado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-046][hu] «Errores genéricos»; [CU-ADM-15][cu] excepciones pasos 4 y 9; [CU-ADM-31][cu] excepciones |

**Precondiciones:** bloqueos de red.
**Datos:** archivo válido.
**Pasos:**
1. Bloquea `POST /roles/previsualizar` y elige el archivo.
2. Quita el bloqueo, obtén la vista previa, bloquea `POST /roles/carga-masiva` y haz clic en «Guardar válidos».
**Resultado esperado:** «Error al procesar el archivo» y «Error al guardar los registros».

#### TC-ADM-379 — Vista previa de sucursales sin archivo adjunto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-046][hu] «Vista previa sin archivo adjunto»; [CU-ADM-15][cu] excepción paso 5 |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /sucursales/previsualizar` multipart sin el campo `file`; y con un archivo vacío (0 bytes).
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** 400 con el texto plano «Debe adjuntar un archivo Excel.» (sin `code`).

### HU-ADM-047 — Exportar una búsqueda a Excel

#### TC-ADM-380 — Exportar con los filtros escritos (formato común)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-047][hu] «Exportar con los filtros escritos»; [CU-ADM-09][cu] flujo básico (paso 7); [CU-ADM-11][cu] flujo básico; [CU-ADM-17][cu] flujo básico; [CU-ADM-21][cu] flujo básico; [CU-ADM-25][cu] flujo básico; [RN-IMP-008, RN-IMP-009, RN-IMP-010][rn] |

**Precondiciones:** las cinco búsquedas con resultados (Empresa, Sucursales, Empleados, Roles, Series Fiscales); ya hiciste clic en «Buscar» en cada una. No verificado en la aplicación: el contenido de los archivos descargados.
**Datos:** sin filtros.
**Pasos:**
1. En cada búsqueda haz clic en «Exportar».
2. Abre cada libro (los casos por pantalla son TC-ADM-107, 132, 179, 213 y 325).
**Resultado esperado:** se descarga un libro con una hoja «Data»; la cabecera tiene el mismo estilo de la plantilla, las filas alternas van sombreadas y las columnas se ajustan al contenido; los nulos salen como celda vacía y los números, fechas y booleanos conservan su tipo; los títulos de columna son los que envía el cliente ya traducidos.

#### TC-ADM-381 — Exportación de más de 10 000 filas (formato común)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-047][hu] «Más de 10 000 filas»; [CU-ADM-09][cu] excepción paso 8; [CU-ADM-11][cu] excepción paso 7; [CU-ADM-17][cu] excepción paso 7; [CU-ADM-21][cu] excepción paso 7; [CU-ADM-25][cu] excepción paso 7; [RN-IMP-017][rn] |

**Precondiciones:** una empresa de prueba con 10 001 roles o 10 001 empleados, cargados por script o por carga masiva de roles (dos archivos de 5000 filas más uno de 1 fila; los roles pasan por carga masiva). No verificado en la aplicación.
**Datos:** búsqueda sin filtros y luego con un filtro que deje 10 000 o menos filas.
**Pasos:**
1. Busca y haz clic en «Exportar» en «Roles & Permisos».
2. Aplica un filtro que deje 10 000 filas o menos y exporta.
**Resultado esperado:** con más de 10 000 filas la API responde 400 `Export.TooManyRows`, no genera el archivo, y ves «Error al buscar roles» (en Series Fiscales, «Error al exportar las series fiscales»); con 10 000 o menos se descarga `roles.xlsx`.

#### TC-ADM-382 — Títulos de columna inválidos en la exportación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-047][hu] «Títulos de columna inválidos»; [CU-ADM-09][cu] excepción paso 8; [RN-IMP-009][rn] |

**Precondiciones:** Solo API; `<token>` de Ana; el cuerpo de una exportación real de roles como base.
**Datos:** (a) 51 títulos; (b) un título nulo; (c) un título de 101 caracteres.
**Pasos:**
1. Envía `POST /roles/exportar` con cada variante.
**Resultado esperado:** 400 sin campo `code` (mensaje del modelo `Export.Headers.Invalid`) en los tres casos. No verificado: los nombres de las propiedades del cuerpo (tómalos de una exportación hecha desde el SPA).

#### TC-ADM-383 — Etiquetas de celda demasiado largas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-047][hu] «Etiquetas de celda demasiado largas»; [CU-ADM-09][cu] excepción paso 8; [RN-IMP-010][rn] |

**Precondiciones:** Solo API; exportación de empresas (que usa las etiquetas «Sí» y «No»).
**Datos:** etiqueta de «Sí» de 21 caracteres; otra de 20.
**Pasos:**
1. Envía `POST /empresas/exportar` con cada etiqueta.
**Resultado esperado:** con 21 caracteres, 400 sin campo `code` (mensaje del modelo `Export.Label.Invalid`); con 20 responde 200.

#### TC-ADM-384 — Formato de fecha de la exportación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-047][hu] «Formato de fecha»; [CU-ADM-17][cu] excepción paso 7; [RN-IMP-011][rn] |

**Precondiciones:** Solo API; exportación de series fiscales.
**Datos:** `dd/MM/yyyy`, `MM/dd/yyyy`, `dd/MM/yyyy HH:mm`, `MM/dd/yyyy HH:mm`, y `yyyy/MM/dd`.
**Pasos:**
1. Envía `POST /series-fiscales/exportar` con cada valor.
**Resultado esperado:** los cuatro primeros responden 200; `yyyy/MM/dd` responde 400 sin `code` (mensaje del modelo `Export.DateFormat.Invalid`). Hallazgo (Brecha conocida de HU-ADM-047): el usuario nunca ve el mensaje del backend, solo el genérico de la pantalla.

### HU-ADM-048 — Dejar rastro de las acciones de seguridad

#### TC-ADM-385 — Un comando auditable exitoso deja un evento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-ADM-048][hu] «Un comando auditable exitoso deja un evento»; Transversal; [RN-GEN-012, RN-GEN-013][rn] |

**Precondiciones:** acceso de solo lectura a `Auditoria_Eventos_Seguridad`.
**Datos:** una acción por tipo de evento.
**Pasos:**
1. Ejecuta cada acción de la tabla con Ana.
2. Consulta el último evento de la tabla `Auditoria_Eventos_Seguridad`.
**Resultado esperado:** se inserta, en la misma transacción, un evento con el empleado que actúa, la tabla afectada, el id del registro, el tipo de acción, un detalle breve sin datos personales (en `Payload_Nuevo`) y la IP del cliente:

| Evento | Acción que lo provoca | Caso donde se prueba |
|---|---|---|
| `ROL_CREAR` / `ROL_ACTUALIZAR` / `ROL_ELIMINAR` | crear, editar y eliminar un rol | TC-ADM-216, 228, 255 |
| `PERMISO_ASIGNAR` / `PERMISO_REVOCAR` | marcar y desmarcar permisos | TC-ADM-238, 239 |
| `ROL_SUCURSALES` | guardar las sucursales de un rol | TC-ADM-252 |
| `ROL_ASIGNAR_CUENTA` / `ROL_QUITAR_CUENTA` | cambiar el rol de una cuenta | TC-ADM-298 |
| `CUENTA_REGISTRAR` / `CUENTA_ACTUALIZAR` / `CUENTA_ELIMINAR` | crear, editar y eliminar una cuenta | TC-ADM-274, 286, 312 |
| `CUENTA_SUC_ASIGNAR` / `CUENTA_SUC_QUITAR` | agregar y quitar una sucursal de una cuenta | TC-ADM-305, 306 |
| `CLAVE_CAMBIAR` / `CLAVE_RESTABLECER` | cambiar y restablecer la contraseña | TC-ADM-049, 076 |
| `SERIE_CREAR` / `SERIE_ACTUALIZAR` / `SERIE_ELIMINAR` | crear, editar y eliminar una serie | TC-ADM-328, 341, 349 |
| `EMPRESA_ACTUALIZAR` | editar la empresa | TC-ADM-114 |

#### TC-ADM-386 — Lo que no se audita por este mecanismo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-048][hu] «Lo que no se audita por este mecanismo»; Transversal; [RN-GEN-013][rn] |

**Precondiciones:** acceso de solo lectura a `Auditoria_Eventos_Seguridad`; cuenta el número de filas antes de empezar.
**Datos:** crear y editar una sucursal; crear y editar un empleado; una carga masiva de roles; iniciar y cerrar sesión.
**Pasos:**
1. Ejecuta cada acción.
2. Vuelve a contar las filas.
**Resultado esperado:** no se inserta ningún evento de auditoría de seguridad en ninguno de los casos (Hallazgo: las acciones sobre sucursales, empleados, carga masiva, inicio y cierre de sesión no se auditan; [CU-ADM-12][cu] «Brecha conocida»).

#### TC-ADM-387 — Un comando fallido no deja evento ni revierte los intentos fallidos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-048][hu] «Un comando fallido no deja evento»; [CU-ADM-02][cu] postcondiciones; [RN-GEN-012, RN-GEN-013, RN-AUT-007][rn] |

**Precondiciones:** acceso de solo lectura a la base de datos; una cuenta de prueba para fallos.
**Datos:** un cambio de contraseña con la contraseña actual incorrecta; un intento de inicio de sesión fallido.
**Pasos:**
1. Cuenta los eventos, ejecuta una acción que termine en error de negocio (por ejemplo, TC-ADM-053) y vuelve a contar.
2. Mira «Intentos Fallidos» tras el login fallido.
**Resultado esperado:** no se inserta evento; lo que el handler ya hubiera guardado, como el contador de intentos fallidos del inicio de sesión, no se revierte.

#### TC-ADM-388 — Una excepción revierte toda la transacción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Baja | [HU-ADM-048][hu] «Una excepción revierte todo»; Transversal; [RN-GEN-012][rn] |

**Precondiciones:** No verificado en la aplicación: provocar una excepción en un handler requiere alterar el entorno (por ejemplo, detener la base de datos a mitad de operación); se respalda en las pruebas automáticas ([09a][p09a]).
**Datos:** ninguno.
**Pasos:**
1. Si el equipo técnico puede provocarlo, ejecuta un comando auditable que lance una excepción.
**Resultado esperado:** se revierte toda la transacción del comando: no queda el cambio ni el evento.

#### TC-ADM-389 — Forma mínima de un evento de auditoría

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-ADM-048][hu] «Forma mínima de un evento»; Transversal; [RN-GEN-014][rn] |

**Precondiciones:** acceso de solo lectura a `Auditoria_Eventos_Seguridad`. No verificado en la aplicación: los límites de longitud y los errores 500 por entrada inválida no se pueden provocar desde la aplicación.
**Datos:** los eventos creados en los casos anteriores.
**Pasos:**
1. Revisa los campos de varios eventos.
**Resultado esperado:** empleado, tabla afectada (hasta 50 caracteres), id de registro y tipo de acción (hasta 20) siempre presentes; IP de hasta 45 caracteres; la fecha es la hora UTC actual; en el restablecimiento de clave un evento inválido se omite en silencio.

#### TC-ADM-390 — Los eventos de auditoría no se modifican

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-048][hu] «Los eventos no se modifican»; Transversal; [RN-GEN-013][rn] |

**Precondiciones:** No verificado en la aplicación: la afirmación es del código (la entidad no tiene métodos de modificación ni de borrado) y no hay pantalla ni endpoint que permita probarlo.
**Datos:** ninguno.
**Pasos:**
1. Busca en las pantallas y en la API una forma de editar o borrar un evento.
**Resultado esperado:** no existe ninguna.

#### TC-ADM-391 — No hay pantalla ni endpoint para consultar los eventos de seguridad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-ADM-048][hu] «La consulta de eventos»; Transversal; [RN-GEN-013][rn] |

**Precondiciones:** `<token>` de Ana; Solo API para la parte del 501.
**Datos:** `POST /auditoria/anonimizar` con cualquier cuerpo; la dirección `/auditoria` en el SPA.
**Pasos:**
1. Revisa el menú y escribe `/auditoria`.
2. Envía la petición a la API.
**Resultado esperado:** no hay pantalla ni ruta del SPA para auditoría; `POST /auditoria/anonimizar` responde 501 `Auditoria.Anonimizacion.NoImplementada` (permiso `Auditoria:AnonimizarDatos`). Hallazgo: nadie puede consultar el rastro por la API ni por el SPA; no hay política de retención ni purga.

### HU-ADM-049 — Proteger la API contra abuso y configuraciones inseguras

#### TC-ADM-392 — Límite general de peticiones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-049][hu] «Límite general de peticiones»; [CU-ADM-08][cu] flujo básico (paso 2); [RN-GEN-025][rn] |

**Precondiciones:** Solo API; un script que envíe peticiones rápidas desde una sola IP (no lo corras contra un entorno compartido).
**Datos:** 250 `GET /sucursales/permitidas` en menos de un minuto.
**Pasos:**
1. Envía las peticiones y registra los códigos HTTP.
**Resultado esperado:** al superar 200 por minuto (ventana fija, con hasta 10 en cola) la API responde 429 sin campo `code`. No verificado en la aplicación: no se provocó el límite.

#### TC-ADM-393 — Límite de credenciales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-ADM-049][hu] «Límite de credenciales»; [CU-ADM-02][cu] excepción paso 3; [RN-GEN-026][rn] |

**Precondiciones:** Solo API; usa usuarios inexistentes para no bloquear cuentas.
**Datos:** 11 peticiones en un minuto a cada uno de `POST /auth/login`, `/auth/forgot-password`, `/auth/reset-password` y `/auth/change-password` (esta última con un token válido).
**Pasos:**
1. Envía las 11 peticiones a un endpoint, espera un minuto y repite con el siguiente.
**Resultado esperado:** la petición 11 responde 429 sin campo `code` (sin cola); pasado el minuto vuelve a responder normalmente. No verificado en la aplicación: no se provocó.

#### TC-ADM-394 — Límite del registro público de empresas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-049][hu] «Límite del registro de empresas»; [CU-ADM-01][cu] excepción paso 4; [RN-GEN-027][rn] |

**Precondiciones:** Solo API; datos de registro válidos distintos en cada intento (RUC y usuario distintos). No verificado en la aplicación.
**Datos:** 6 `POST /empresas/registrar` en una hora desde la misma IP.
**Pasos:**
1. Envía las 6 peticiones.
**Resultado esperado:** las 5 primeras responden 201 (o el error de negocio que corresponda) y la sexta 429 sin campo `code`, sin cola.

#### TC-ADM-395 — IP real del cliente con X-Forwarded-For

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Baja | [HU-ADM-049][hu] «IP real del cliente»; Transversal; [RN-GEN-029][rn] |

**Precondiciones:** No verificado en la aplicación: depende de la clave `ForwardedHeaders:KnownProxies` del entorno (solo su nombre se documenta). Solo para el equipo técnico.
**Datos:** `POST /auth/login` con cabecera `X-Forwarded-For: 203.0.113.9` desde un cliente que no está en la lista de proxies conocidos.
**Pasos:**
1. Envía la petición y revisa la IP que registra el log o un evento de auditoría.
**Resultado esperado:** sin proxies configurados (o desde un origen que no es un proxy conocido) la cabecera se ignora y la IP registrada es la del cliente real; solo con un proxy conocido se acepta y se usa para límites, registros y auditoría.

#### TC-ADM-396 — Limpieza de HTML en la entrada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-049][hu] «Limpieza de HTML en la entrada»; [CU-ADM-01][cu] flujo básico (paso 4); [RN-GEN-010][rn] |

**Precondiciones:** sesión de Ana; Solo API para la carga masiva.
**Datos:** (a) nombre de sucursal `<b>Norte</b><script>alert(1)</script>`; (b) cambio de contraseña con nueva contraseña `Abc<123>xyz`; (c) carga masiva de sucursales con una fila de nombre `<i>Fila</i>`.
**Pasos:**
1. Crea la sucursal (a) desde `/sucursales/nueva` y mira su nombre.
2. Cambia la contraseña (b) y vuelve a iniciar sesión con ella.
3. Guarda (c) con `POST /sucursales/carga-masiva`.
**Resultado esperado:** (a) el saneador elimina las etiquetas (el nombre queda `Norte`, sin atributos ni esquemas de URL); (b) las propiedades `password`, `passwordhash`, `token`, `recoverytoken`, `secretkey`, `hash`, `newpassword` y `confirmpassword` no se sanean, por lo que la contraseña se conserva tal cual (No verificado: `CurrentPassword` no está en esa lista); (c) no se recorren objetos anidados ni listas: la fila de la carga masiva conserva el texto con etiquetas (Hallazgo del documento de reglas).

#### TC-ADM-397 — Cabeceras de seguridad de las respuestas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-049][hu] «Cabeceras de seguridad»; Transversal; [RN-GEN-030][rn] |

**Precondiciones:** Solo API; cliente HTTP que muestre las cabeceras.
**Datos:** `GET /health`; `GET /sucursales/permitidas` con `<token>`.
**Pasos:**
1. Envía ambas peticiones y lee las cabeceras de la respuesta.
**Resultado esperado:** toda respuesta lleva `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `X-Permitted-Cross-Domain-Policies: none` y una `Content-Security-Policy` con `default-src` y `frame-ancestors` en `none`; las rutas bajo `/api` llevan además `Cache-Control: no-store`.

#### TC-ADM-398 — Redirección a HTTPS y HSTS

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-ADM-049][hu] «HTTPS»; Transversal; [RN-GEN-031][rn] |

**Precondiciones:** No verificado: el puerto HTTP del entorno local no está documentado en este documento; el HSTS solo se ve fuera del entorno de desarrollo.
**Datos:** una llamada por HTTP a la API.
**Pasos:**
1. Llama a la API por HTTP (sin TLS) y mira la respuesta.
2. En un entorno que no sea de desarrollo, mira la cabecera `Strict-Transport-Security` de una respuesta HTTPS.
**Resultado esperado:** la llamada por HTTP se redirige a HTTPS; fuera del entorno de desarrollo se envía HSTS por 365 días con subdominios.

#### TC-ADM-399 — Orígenes permitidos (CORS)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-049][hu] «Orígenes permitidos»; Transversal; [RN-GEN-032][rn] |

**Precondiciones:** Solo API; entorno de desarrollo.
**Datos:** peticiones `OPTIONS /auth/login` con `Origin: http://localhost:4200`, `Origin: https://localhost:4200` y `Origin: http://sitio-ajeno.test`.
**Pasos:**
1. Envía cada petición y mira `Access-Control-Allow-Origin` y `Access-Control-Allow-Credentials`.
**Resultado esperado:** solo se aceptan `http://localhost:4200` y `https://localhost:4200` (con credenciales permitidas para la cookie de refresco); el origen ajeno no recibe `Access-Control-Allow-Origin`. En otros entornos solo los orígenes de la clave `AllowedOrigins`; si está vacía la API no arranca (No verificado: requiere reiniciar la API).

#### TC-ADM-400 — Configuración mínima del token

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-049][hu] «Configuración mínima del token»; Transversal; [RN-GEN-033][rn] |

**Precondiciones:** No verificado en la aplicación y no ejecutable en el entorno compartido: exige arrancar la API con la clave `Jwt:SecretKey` vacía o de menos de 32 bytes, o sin `Jwt:Issuer` o `Jwt:Audience`, y no se deben tocar los servidores en ejecución ni los secretos. Lo ejerce el equipo técnico en su propio entorno.
**Datos:** configuración inválida por clave (solo se nombran las claves, nunca sus valores).
**Pasos:**
1. En un entorno aparte, arranca la API con una de las claves inválidas.
**Resultado esperado:** la API no arranca.

#### TC-ADM-401 — Estado del servicio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-ADM-049][hu] «Estado del servicio»; Transversal; [RN-GEN-034][rn] |

**Precondiciones:** Solo API; sin token.
**Datos:** `GET /health`.
**Pasos:**
1. Envía la petición sin cabecera `Authorization`.
**Resultado esperado:** responde 200 solo con el estado y la hora, sin versión ni entorno, y sin exigir autenticación.

#### TC-ADM-402 — Denegación por defecto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-ADM-049][hu] «Denegación por defecto»; [HU-ADM-004][hu] «Un token no válido recibe 401 sin código»; Transversal; [RN-GEN-015][rn] |

**Precondiciones:** Solo API; sin token.
**Datos:** una llamada sin token a un endpoint de cada controlador de esta parte, y a los siete anónimos.
**Pasos:**
1. Envía `GET /sucursales`, `GET /empleados`, `GET /roles`, `GET /cuentas`, `GET /series-fiscales`, `GET /empresas`, `GET /auth/me` y `POST /auditoria/anonimizar` sin token.
2. Envía los anónimos sin token (con cuerpos inválidos para no crear datos): `POST /auth/login`, `/auth/refresh`, `/auth/logout`, `/auth/forgot-password`, `/auth/reset-password`, `POST /empresas/registrar`, `GET /health`.
**Resultado esperado:** los del paso 1 responden 401; los del paso 2 no responden 401 (responden 400, 403 `Auth.Csrf`, 200 o 204 según el caso). Solo esos siete son anónimos.

#### TC-ADM-403 — Validación previa de cada request

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-ADM-049][hu] «Validación previa de cada request»; Transversal; [RN-GEN-011][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** `POST /sucursales` con `nombre` vacío y `numeroLicencia` vacío (dos fallos); `POST /roles` con `nombreRol` de 51 caracteres.
**Pasos:**
1. Envía cada petición.
2. Verifica que no se creó nada.
**Resultado esperado:** el handler no se ejecuta; la API responde 400 con `code` `{Request}.{Propiedad del primer fallo}` (por ejemplo `CreateSucursalCommand.Nombre`) y el detalle con todos los mensajes unidos por `; `.

### HU-ADM-050 — Recibir respuestas de error coherentes de la API

#### TC-ADM-404 — Cada comando es una transacción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | [HU-ADM-050][hu] «Cada comando es una transacción»; [CU-ADM-01][cu] postcondición «Fallo»; [RN-GEN-012][rn] |

**Precondiciones:** ver TC-ADM-003 (registro con colisión) y TC-ADM-388 (excepción provocada).
**Datos:** los de esos casos.
**Pasos:**
1. Ejecuta TC-ADM-003.
2. Ejecuta TC-ADM-387 (y TC-ADM-388 si el entorno lo permite).
**Resultado esperado:** cuando el comando lanza una excepción se revierte todo lo que había guardado; un `Result.Failure` sin excepción no revierte lo ya guardado (por ejemplo, el contador de intentos fallidos).

#### TC-ADM-405 — Conflicto de concurrencia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-050][hu] «Conflicto de concurrencia»; [CU-ADM-03][cu] excepción paso 5; [CU-ADM-08][cu] excepción paso 6; [RN-GEN-035][rn] |

**Precondiciones:** ver TC-ADM-037 y TC-ADM-048.
**Datos:** dos operaciones simultáneas sobre el mismo token de refresco.
**Pasos:**
1. Ejecuta TC-ADM-048 varias veces.
**Resultado esperado:** cuando otra operación cambió la fila con `RowVersion` antes, la API responde 409 `Concurrency.Conflict` con un `traceId`; en este alcance solo la tabla de tokens de refresco tiene `RowVersion` y sus handlers capturan la excepción.

#### TC-ADM-406 — Valor único repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-ADM-050][hu] «Valor único repetido»; [CU-ADM-12][cu] excepción paso 6; [CU-ADM-22][cu] excepción paso 6; [RN-GEN-036][rn] |

**Precondiciones:** ver TC-ADM-139 (sucursales), TC-ADM-193 (empleados) y TC-ADM-223 (roles).
**Datos:** los de esos casos.
**Pasos:**
1. Ejecuta los tres casos y anota el aviso de cada pantalla.
**Resultado esperado:** la API responde 409 `Db.Duplicado`; en Empleados y Sucursales ves «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.»; en las demás pantallas de este alcance ves su mensaje genérico (por ejemplo «Error al guardar el rol»); el registro del servidor anota solo el nombre de la restricción, nunca los valores.

#### TC-ADM-407 — Valor fuera de rango (restricción CHECK)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-050][hu] «Valor fuera de rango»; [CU-ADM-12][cu] excepción paso 6; [RN-GEN-037][rn] |

**Precondiciones:** No verificado en la aplicación: ningún formulario de esta parte permite enviar un valor que pase los validadores y viole una restricción CHECK; ver TC-ADM-140.
**Datos:** ninguno.
**Pasos:**
1. Si el equipo técnico conoce una restricción CHECK alcanzable, ejecútala desde la API.
**Resultado esperado:** 400 `Db.ValorInvalido`; en Empleados y Sucursales «Uno de los valores está fuera del rango permitido.»

#### TC-ADM-408 — Referencia inexistente o en uso (clave foránea)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-050][hu] «Referencia inexistente o en uso»; [CU-ADM-13][cu] excepción paso 7; [RN-GEN-038][rn] |

**Precondiciones:** ver TC-ADM-140. No verificado en la aplicación.
**Datos:** ninguno.
**Pasos:**
1. Provoca una violación de clave foránea que no haya comprobado el handler.
**Resultado esperado:** 409 `Db.ReferenciaInvalida`; en Empleados y Sucursales «El registro hace referencia a datos que ya no existen o todavía está en uso.»

#### TC-ADM-409 — Cuerpo demasiado grande

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-ADM-050][hu] «Cuerpo demasiado grande»; [CU-ADM-15][cu] excepción paso 3; [RN-GEN-039][rn] |

**Precondiciones:** Solo API; ver TC-ADM-370.
**Datos:** cuerpo de más de 5 MB a `POST /sucursales/carga-masiva`; y otro que el servidor no pueda leer.
**Pasos:**
1. Envía el cuerpo grande y observa la respuesta y la pantalla de carga masiva.
**Resultado esperado:** 413 `Request.TooLarge`; en la carga masiva ves «El archivo supera el tamaño máximo de 5 MB.»; si el servidor no puede leer el cuerpo responde `Request.Invalid` con el estado de la excepción y la pantalla muestra su genérico. No verificado en la aplicación.

#### TC-ADM-410 — Cuerpo con forma inválida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-050][hu] «Cuerpo con forma inválida»; [CU-ADM-27][cu] excepción paso 5; [RN-GEN-041][rn] |

**Precondiciones:** Solo API; `<token>` de Ana.
**Datos:** (a) `POST /sucursales` con JSON mal formado; (b) con `nombre` numérico; (c) `PUT /roles/<id>` sin `nivelJerarquia`.
**Pasos:**
1. Envía cada petición.
**Resultado esperado:** 400 sin campo `code`, antes de llegar al handler; la pantalla que hiciera esa petición muestra su mensaje genérico.

#### TC-ADM-411 — Error inesperado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-050][hu] «Error inesperado»; [CU-ADM-02][cu] excepción paso 2; [RN-GEN-040][rn] |

**Precondiciones:** No verificado en la aplicación: provocar una excepción no prevista exige alterar el entorno (por ejemplo, detener SQL Server con la API corriendo; no lo hagas en el entorno compartido).
**Datos:** ninguno.
**Pasos:**
1. En un entorno propio, detén la base de datos y haz una acción.
**Resultado esperado:** la API registra el error con su `traceId` y responde 500 con un texto fijo que no revela detalles internos, más el `traceId`; las pantallas que usan el aviso con referencia añaden «Referencia: <traceId>» con un botón para copiar durante 10 segundos; en el inicio de sesión ves «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.»

#### TC-ADM-412 — Contexto del usuario faltante

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-ADM-050][hu] «Contexto del usuario faltante»; [CU-ADM-12][cu] excepción paso 4; [CU-ADM-15][cu] excepción paso 9; [RN-GEN-024][rn] |

**Precondiciones:** ver TC-ADM-143 y TC-ADM-166. No verificado en la aplicación.
**Datos:** token sin identificador de empresa.
**Pasos:**
1. Crea una sucursal y haz una carga masiva de sucursales con ese token.
**Resultado esperado:** la API responde 400 `Auth.Empresa` (aunque su tipo es `Unauthorized`); ves «Error al guardar la sucursal» o «Error al guardar los registros».

#### TC-ADM-413 — El mensaje del backend no se muestra

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-ADM-050][hu] «El mensaje del backend no se muestra»; [CU-ADM-01][cu] excepciones; [RN-GEN-011][rn] |

**Precondiciones:** pestaña de red abierta.
**Datos:** un error con `detail` en inglés (por ejemplo TC-ADM-220: nombre de rol duplicado).
**Pasos:**
1. Provoca el error y lee el cuerpo de la respuesta en la pestaña de red.
2. Lee el aviso de la pantalla.
**Resultado esperado:** la API devuelve el campo `detail` con texto en inglés, pero ninguna pantalla lo muestra: usa su mapa de códigos o su mensaje genérico. Hallazgo (Brecha conocida de HU-ADM-050): no existe un traductor global de códigos; las claves `errors.forbidden`, `errors.unauthorized`, `errors.notFound`, `errors.serverError` y `errors.networkError` existen y no se usan.

## 5. Pendientes y «No verificado»

### 5.1 En curso (sin commitear)

Ninguno. Este documento no ejecutó comandos git (lo prohíben las instrucciones de QA); las fuentes ([12a][hu] y [13a][cu]) informan que el árbol de trabajo estaba sin cambios pendientes sobre los commits `43f5529` (backend) y `eeffbb6` (frontend). **No verificado:** que no haya commits más nuevos que esos al ejecutar los casos; si los hay, hay que reconciliar los casos de eliminación (la validación de dependencias de sucursales, empleados y otros maestros) con el código vigente.

### 5.2 No verificado, por motivo

| Motivo | Casos |
|---|---|
| Límite de peticiones (429) no provocado para no bloquear cuentas ni la IP | TC-ADM-011, 022, 057, 072, 080, 392, 393, 394 |
| Falla del servidor o de la red no provocada | TC-ADM-023, 073, 081, 411 |
| Estado de la empresa (suspendida, cancelada, prueba vencida, eliminada) solo cambiable por base de datos; no hay endpoint ni pantalla | TC-ADM-020, 021, 039, 124, 125, 126 |
| Token fabricado sin roles o sin empresa (exige la clave de firma) | TC-ADM-097, 143, 166, 412 |
| Carreras entre dos peticiones (difíciles de repetir) | TC-ADM-003, 048, 139, 165, 193, 200, 223, 265, 282, 356 |
| Contenido de los archivos `.xlsx` descargados (el navegador integrado no mostró descargas) | TC-ADM-107, 132, 159, 179, 213, 260, 325, 380 |
| Límites de 5 MB, 5000 filas, descompresión y 10 000 filas de exportación no provocados | TC-ADM-108, 133, 180, 214, 327, 370, 371, 372, 381 |
| Cuentas o roles que la historia no pudo probar (rol sin `Admin`, `Asignar roles` o `Gestionar permisos`, cuenta propia) | TC-ADM-245, 246, 292, 300, 301, 303, 315 |
| Serie fiscal que ya emitió comprobantes (la empresa de prueba no vendió nada); depende de la parte C | TC-ADM-345, 346, 350, 354, 355, 356 |
| Efecto de la sucursal de un rol más allá de la sucursal base (manual cap. 3, §13, punto 10) y guardado de un gerente elegido | TC-ADM-141, 151, 250 |
| Restricciones CHECK o claves foráneas de la base de datos, excepciones no previstas y bandeja de salida | TC-ADM-140, 365, 388, 389, 390, 407, 408, 411 |
| Arranque de la API con configuración inválida (no se toca el entorno compartido ni los secretos) | TC-ADM-400 |
| Redirección HTTP, HSTS y `X-Forwarded-For` dependen del entorno | TC-ADM-395, 398 |
| Rol «Admin» con otras mayúsculas, duración exacta del token de acceso (`Jwt:ExpiryMinutes`), zona horaria mostrada en «Bloqueado Hasta», medición de tiempos de respuesta del login | TC-ADM-019, 031, 094, 363 |
| Nombres exactos de las propiedades de los cuerpos JSON de algunas peticiones «Solo API» (se marcan en cada caso; se confirman con el esquema de la API o con una petición hecha desde el SPA) | TC-ADM-095, 185, 253, 326, 338, 382 |

### 5.3 Hallazgos que los casos fijan con el comportamiento actual

| Hallazgo (fuente) | Casos |
|---|---|
| El registro solo traduce `RegistrarEmpresaCommand.DireccionSede`; los demás rechazos se ven como «Error al registrar la empresa» (12a HU-ADM-001, 19a §1.2) | TC-ADM-010, 011 |
| La pantalla no avisa del bloqueo ni de la causa del rechazo del ingreso (12a HU-ADM-002 y 003) | TC-ADM-019, 027 |
| `forgot-password` responde 200 aunque el usuario esté vacío o tenga más de 50 caracteres (RN-AUT-032) | TC-ADM-071 |
| El SPA nunca avisa «sesión expirada» (19a §5.4) | TC-ADM-033 |
| El formulario de empresa limita el nombre a 150 (la API admite 200) y no valida el RUC (12a HU-ADM-015) | TC-ADM-118, 119, 121 |
| El detalle de la empresa no muestra días restantes ni «Vigente»; la política de receta no tiene pantalla; ningún endpoint cambia el estado de la suscripción (12a HU-ADM-014 y 016) | TC-ADM-109, 124 |
| El filtro «Vencimiento hasta» se envía a medianoche local; la exportación de empresas trae el estado sin traducir (13a CU-ADM-09) | TC-ADM-107, 364 |
| Eliminar una sucursal, un empleado o un rol: el aviso no dice qué bloquea y el SPA decide por el estado 409 (hallazgo 3 de la sección 12.3 de 01a) | TC-ADM-156, 206, 257 |
| Eliminar un empleado con cuenta no se impide: la cuenta queda oculta y su usuario ocupado (12a HU-ADM-026) | TC-ADM-207 |
| Roles: nombre de 51 a 100 caracteres, nivel 0 y nombre duplicado terminan en mensajes genéricos o incoherentes con el formulario (12a HU-ADM-028) | TC-ADM-219, 220, 221 |
| Nada impide eliminar el rol «Admin» sin cuentas (12a HU-ADM-032) | TC-ADM-259 |
| Cuentas: el mensaje de longitud dice 100 y el límite real es 50; la contraseña de 1 a 7 caracteres no muestra mensaje; ningún código de la API se traduce en el alta (12a HU-ADM-035) | TC-ADM-278, 279, 280, 281, 282 |
| Un usuario puede eliminar su propia cuenta (hallazgo 7 de la sección 12.3 de 01a) | TC-ADM-318 |
| Un guardado de la ficha de un rol y el cambio de rol de una cuenta no son atómicos (12a HU-ADM-029 y 037) | TC-ADM-233, 304 |
| La API acepta permisos que no están en el catálogo (RN-ROL-013) | TC-ADM-096, 243 |
| La carga masiva marca todas las filas repetidas, muestra los campos con claves internas y la vista previa de roles exige nivel 1 mientras el guardado acepta 0 (12a HU-ADM-021 y 033) | TC-ADM-161, 162, 263, 264 |
| No hay pantalla ni endpoint para consultar la auditoría, y varias acciones no se auditan (12a HU-ADM-048) | TC-ADM-386, 391 |
| El saneador de HTML no recorre listas ni objetos anidados (RN-GEN-010) | TC-ADM-396 |
| El mensaje del backend nunca se muestra y las claves `errors.*` globales no se usan (12a HU-ADM-050) | TC-ADM-384, 413 |
| El idioma elegido no se conserva (19a §4.5) | TC-ADM-101 |
| Al abrir la ficha de una cuenta cuyo empleado fue eliminado se vio la clave sin traducir `cuentas.detail.load-error` (manual cap. 3, §12, punto 1) | TC-ADM-293 |

### 5.4 Brechas conocidas de las fuentes que no tienen caso propio

| Brecha | Motivo |
|---|---|
| «Buscar» en `/sucursales` no reinicia la página (CU-ADM-11) | No lo comprobó el manual en vivo; la historia HU-ADM-017 no lo tiene como escenario. Se puede agregar un caso cuando se confirme |
| La fila vacía de la tabla de empresas dice «Resultados» y no «Sin resultados» (HU-ADM-013, nota final) | Presentación, sin escenario |
| Los botones del cuadro «Eliminar cuenta» son «Confirmar» y «Cancelar» (HU-ADM-039, nota final) | Se verifica de paso en TC-ADM-312 |
| `Resolver nombres de otros catálogos` (HU-ADM-046) | No aplica a sucursales ni a roles; lo ejercen los casos de carga masiva de la parte B |
| `RN-GEN-028` (límite de los endpoints de IA) | Fuera de este alcance: lo ejerce la parte B (recetas) |

## 6. Tabla de cobertura

### 6.1 Resumen

El script de comprobación (solo lectura, en el directorio temporal de la sesión) cruzó este documento con las fuentes y dio:

| Concepto | Total | Con caso |
|---|---|---|
| Escenarios (`Escenario:` y `Esquema del escenario:`) de las 50 historias | 424 | 423 (el que falta es el de «Resolver nombres de otros catálogos», que no aplica) |
| Historias de usuario HU-ADM-001 a HU-ADM-050 | 50 | 50 |
| Casos de uso CU-ADM-01 a CU-ADM-37 | 37 | 37 |
| Reglas de negocio de 01a (RN) | 203 | 202 (falta `RN-GEN-028`, de la parte B) |
| Casos de prueba | 413 | — |

Los casos por tipo son: Positivo 163, Negativo 129, Permisos 60, Límite 37, Concurrencia 13, Integración 11. Por prioridad: Alta 166, Media 149, Baja 98.

| Módulo | Sección | Casos | Alta | Media | Baja |
|---|---|---|---|---|---|
| Registro y acceso | 4.1 | 81 | 47 | 24 | 10 |
| Menú, permisos de pantalla y cabecera | 4.2 | 20 | 8 | 6 | 6 |
| Empresa y suscripción | 4.3 | 25 | 10 | 11 | 4 |
| Sucursales | 4.4 | 46 | 18 | 14 | 14 |
| Empleados | 4.5 | 35 | 14 | 14 | 7 |
| Roles y permisos | 4.6 | 60 | 17 | 29 | 14 |
| Cuentas de usuario | 4.7 | 51 | 22 | 17 | 12 |
| Series fiscales | 4.8 | 38 | 18 | 7 | 13 |
| Reglas transversales y de plataforma | 4.9 | 57 | 12 | 27 | 18 |
| **Total** | | **413** | **166** | **149** | **98** |

### 6.2 Escenarios de las historias → casos

Cada fila lista, en el orden en que aparecen en la historia, el número del escenario y el caso que lo cubre (`n→nnn`, donde `nnn` es `TC-ADM-nnn`; `+` indica más de un caso). Los títulos están citados entre « » en la celda «Cubre» de cada caso.

| Historia | Escenarios | Escenario → caso |
|---|---|---|
| HU-ADM-001 | 11 | 1→001; 2→002; 3→003; 4→004; 5→005; 6→006; 7→007; 8→008; 9→009; 10→010; 11→011 |
| HU-ADM-002 | 13 | 1→012; 2→013; 3→014; 4→015; 5→016; 6→017; 7→018; 8→019; 9→020; 10→021; 11→022; 12→023; 13→024 |
| HU-ADM-003 | 6 | 1→025; 2→026; 3→027; 4→028; 5→029; 6→030 |
| HU-ADM-004 | 12 | 1→031; 2→032; 3→033; 4→034; 5→035; 6→036; 7→037; 8→038; 9→039; 10→040; 11→041+402; 12→042 |
| HU-ADM-005 | 7 | 1→043; 2→044; 3→045; 4→046; 5→047; 6→047; 7→048 |
| HU-ADM-006 | 9 | 1→049; 2→050; 3→051; 4→052; 5→053; 6→054; 7→055; 8→056; 9→057 |
| HU-ADM-007 | 6 | 1→058; 2→059; 3→060; 4→061+064; 5→062; 6→063 |
| HU-ADM-008 | 10 | 1→065; 2→067; 3→068; 4→069; 5→066; 6→070; 7→071; 8→072; 9→073; 10→074 |
| HU-ADM-009 | 7 | 1→075; 2→076; 3→077; 4→078; 5→079; 6→080; 7→081 |
| HU-ADM-010 | 9 | 1→082; 2→083; 3→084; 4→085; 5→086; 6→087; 7→088; 8→089; 9→090 |
| HU-ADM-011 | 10 | 1→091; 2→092; 3→092; 4→093; 5→093; 6→094; 7→095; 8→096; 9→097; 10→098 |
| HU-ADM-012 | 4 | 1→099; 2→100; 3→100; 4→101 |
| HU-ADM-013 | 10 | 1→102; 2→102; 3→103; 4→104; 5→105; 6→106; 7→106; 8→107; 9→108; 10→108 |
| HU-ADM-014 | 5 | 1→109; 2→110; 3→111; 4→112; 5→113 |
| HU-ADM-015 | 10 | 1→114; 2→115; 3→116; 4→117; 5→118; 6→119; 7→120; 8→121; 9→122; 10→123 |
| HU-ADM-016 | 7 | 1→124; 2→124; 3→124; 4→020; 5→125; 6→039; 7→126 |
| HU-ADM-017 | 8 | 1→127; 2→128; 3→129; 4→130; 5→131; 6→132; 7→133; 8→134 |
| HU-ADM-018 | 10 | 1→135; 2→136; 3→137; 4→138; 5→139; 6→140; 7→141; 8→142; 9→143; 10→144 |
| HU-ADM-019 | 10 | 1→145; 2→146; 3→147; 4→148; 5→149; 6→150; 7→151; 8→140; 9→152; 10→153 |
| HU-ADM-020 | 5 | 1→154; 2→155; 3→156; 4→157; 5→158 |
| HU-ADM-021 | 10 | 1→159; 2→160; 3→161; 4→162; 5→163; 6→164; 7→165; 8→166; 9→167; 10→168 |
| HU-ADM-022 | 4 | 1→169; 2→170; 3→171; 4→172 |
| HU-ADM-023 | 9 | 1→173; 2→174; 3→175; 4→176; 5→177; 6→178; 7→179; 8→180; 9→181 |
| HU-ADM-024 | 13 | 1→182; 2→183; 3→184; 4→185; 5→186; 6→187; 7→188; 8→189; 9→190; 10→191; 11→192; 12→193; 13→194 |
| HU-ADM-025 | 8 | 1→195; 2→196; 3→197; 4→198; 5→199; 6→200; 7→201; 8→202 |
| HU-ADM-026 | 5 | 1→203; 2→204; 3→205; 4→206; 5→207 |
| HU-ADM-027 | 8 | 1→208; 2→209; 3→210; 4→211; 5→212; 6→213; 7→214; 8→215 |
| HU-ADM-028 | 10 | 1→216; 2→217; 3→218; 4→219; 5→220; 6→221; 7→222; 8→223; 9→224; 10→225 |
| HU-ADM-029 | 8 | 1→226; 2→227; 3→228; 4→229; 5→230; 6→231; 7→232; 8→233 |
| HU-ADM-030 | 15 | 1→234; 2→235; 3→236; 4→237; 5→238; 6→239; 7→240; 8→241; 9→242; 10→243; 11→244; 12→245; 13→246; 14→247; 15→248 |
| HU-ADM-031 | 6 | 1→249; 2→250; 3→251; 4→252; 5→253; 6→254 |
| HU-ADM-032 | 5 | 1→255; 2→256; 3→257; 4→258; 5→259 |
| HU-ADM-033 | 8 | 1→260; 2→261; 3→262; 4→263; 5→264; 6→265; 7→266; 8→267 |
| HU-ADM-034 | 6 | 1→268; 2→269; 3→270; 4→271; 5→272; 6→273 |
| HU-ADM-035 | 11 | 1→274; 2→275; 3→276; 4→277; 5→278; 6→279; 7→280; 8→281; 9→282; 10→283; 11→284 |
| HU-ADM-036 | 10 | 1→285; 2→286; 3→287; 4→288; 5→289; 6→290; 7→291; 8→292; 9→293; 10→294 |
| HU-ADM-037 | 10 | 1→295; 2→296; 3→297; 4→298; 5→299; 6→300; 7→301; 8→302; 9→303; 10→304 |
| HU-ADM-038 | 7 | 1→305; 2→306; 3→307; 4→308; 5→309; 6→310; 7→311 |
| HU-ADM-039 | 6 | 1→312; 2→313; 3→314; 4→315; 5→316; 6→317 |
| HU-ADM-040 | 9 | 1→319; 2→320; 3→321; 4→322; 5→323; 6→324; 7→325; 8→326; 9→327 |
| HU-ADM-041 | 13 | 1→328; 2→329; 3→330; 4→331; 5→332; 6→333; 7→334; 8→335; 9→336; 10→337; 11→338; 12→339; 13→340 |
| HU-ADM-042 | 8 | 1→341; 2→342; 3→343; 4→344; 5→345; 6→346; 7→347; 8→348 |
| HU-ADM-043 | 5 | 1→349; 2→350; 3→351; 4→352; 5→353 |
| HU-ADM-044 | 3 | 1→354; 2→355; 3→356 |
| HU-ADM-045 | 9 | 1→357; 2→358; 3→359; 4→360; 5→361; 6→362; 7→363; 8→364; 9→365 |
| HU-ADM-046 | 15 | 1→366; 2→367; 3→368; 4→369; 5→370; 6→371; 7→372; 8→373; 9→374; 10→375; 11→376; 12→377; 13→378; 14→379; 15→sin caso (no aplica a sucursales ni roles; parte B) |
| HU-ADM-047 | 5 | 1→380; 2→381; 3→382; 4→383; 5→384 |
| HU-ADM-048 | 7 | 1→385; 2→386; 3→387; 4→388; 5→389; 6→390; 7→391 |
| HU-ADM-049 | 12 | 1→392; 2→393; 3→394; 4→395; 5→396; 6→397; 7→398; 8→399; 9→400; 10→401; 11→402; 12→403 |
| HU-ADM-050 | 10 | 1→404; 2→405; 3→406; 4→407; 5→408; 6→409; 7→410; 8→411; 9→412; 10→413 |

### 6.3 Historias de usuario → casos

| Historia | Casos que la citan |
|---|---|
| HU-ADM-001 | 001 a 011 |
| HU-ADM-002 | 012 a 024 |
| HU-ADM-003 | 025 a 030 |
| HU-ADM-004 | 031 a 042 y 402 |
| HU-ADM-005 | 043 a 048 |
| HU-ADM-006 | 049 a 057 |
| HU-ADM-007 | 058 a 064 |
| HU-ADM-008 | 065 a 074 |
| HU-ADM-009 | 075 a 081 |
| HU-ADM-010 | 082 a 090 |
| HU-ADM-011 | 091 a 098 |
| HU-ADM-012 | 099 a 101 |
| HU-ADM-013 | 102 a 108 |
| HU-ADM-014 | 109 a 113 |
| HU-ADM-015 | 114 a 123 |
| HU-ADM-016 | 020, 039, 124, 125, 126 |
| HU-ADM-017 | 127 a 134 |
| HU-ADM-018 | 135 a 144 |
| HU-ADM-019 | 140, 145 a 153 |
| HU-ADM-020 | 154 a 158 |
| HU-ADM-021 | 159 a 168 |
| HU-ADM-022 | 169 a 172 |
| HU-ADM-023 | 173 a 181 |
| HU-ADM-024 | 182 a 194 |
| HU-ADM-025 | 195 a 202 |
| HU-ADM-026 | 203 a 207 |
| HU-ADM-027 | 208 a 215 |
| HU-ADM-028 | 216 a 225 |
| HU-ADM-029 | 226 a 233 |
| HU-ADM-030 | 234 a 248 |
| HU-ADM-031 | 249 a 254 |
| HU-ADM-032 | 255 a 259 |
| HU-ADM-033 | 260 a 267 |
| HU-ADM-034 | 268 a 273 |
| HU-ADM-035 | 274 a 284 |
| HU-ADM-036 | 285 a 294 |
| HU-ADM-037 | 295 a 304 |
| HU-ADM-038 | 305 a 311 |
| HU-ADM-039 | 312 a 318 |
| HU-ADM-040 | 319 a 327 |
| HU-ADM-041 | 328 a 340 |
| HU-ADM-042 | 341 a 348 |
| HU-ADM-043 | 349 a 353 |
| HU-ADM-044 | 354 a 356 |
| HU-ADM-045 | 357 a 365 |
| HU-ADM-046 | 366 a 379 |
| HU-ADM-047 | 380 a 384 |
| HU-ADM-048 | 385 a 391 |
| HU-ADM-049 | 392 a 403 |
| HU-ADM-050 | 404 a 413 |

### 6.4 Casos de uso → casos

| Caso de uso | Casos que lo citan |
|---|---|
| CU-ADM-01 | 001 a 011, 063, 110, 146, 209, 359, 394, 396, 404, 413 |
| CU-ADM-02 | 012 a 030, 124, 359, 387, 393, 411 |
| CU-ADM-03 | 043 a 048, 405 |
| CU-ADM-04 | 049 a 057 |
| CU-ADM-05 | 058 a 062, 064, 288 |
| CU-ADM-06 | 065 a 074, 126 |
| CU-ADM-07 | 075 a 081 |
| CU-ADM-08 | 031 a 042, 125, 392, 405 |
| CU-ADM-09 | 091, 102 a 109, 111 a 113, 364, 380 a 383 |
| CU-ADM-10 | 114 a 123 |
| CU-ADM-11 | 127 a 134, 380, 381 |
| CU-ADM-12 | 092, 135 a 144, 153, 406, 407, 412 |
| CU-ADM-13 | 093, 140, 145 a 152, 408 |
| CU-ADM-14 | 154 a 158, 353 |
| CU-ADM-15 | 159 a 168, 366 a 379, 409, 412 |
| CU-ADM-16 | 169 a 172 |
| CU-ADM-17 | 319 a 327, 340, 380, 381, 384 |
| CU-ADM-18 | 328 a 340, 354 a 356 |
| CU-ADM-19 | 341 a 348 |
| CU-ADM-20 | 349 a 352 |
| CU-ADM-21 | 173 a 181, 380, 381 |
| CU-ADM-22 | 182 a 194, 406 |
| CU-ADM-23 | 195 a 202 |
| CU-ADM-24 | 203 a 207, 314 |
| CU-ADM-25 | 208, 210 a 215, 380, 381 |
| CU-ADM-26 | 095, 216 a 225 |
| CU-ADM-27 | 226 a 233, 245, 410 |
| CU-ADM-28 | 087, 234 a 248 |
| CU-ADM-29 | 249 a 254 |
| CU-ADM-30 | 255 a 259 |
| CU-ADM-31 | 260 a 267, 376 a 378 |
| CU-ADM-32 | 268 a 273 |
| CU-ADM-33 | 274 a 284 |
| CU-ADM-34 | 030, 285 a 294 |
| CU-ADM-35 | 040, 295 a 304 |
| CU-ADM-36 | 291, 305 a 311 |
| CU-ADM-37 | 312 a 318 |

### 6.5 Reglas de negocio → casos

Cada fila agrupa las reglas de un prefijo; `nnn→casos` indica la regla (por ejemplo `AUT-001`) y los primeros casos que la ejercen (el signo … indica que hay más). Las 203 reglas de [01a][rn] tienen caso salvo `GEN-028`, que es del límite de los endpoints de IA y lo ejerce la parte B.

| Prefijo | Reglas | Regla → casos |
|---|---|---|
| EMP | 18 | 001→091,102,114,115; 002→001,114,117,118; 003→001,005; 004→119,121; 005→001,008,120; 006→001,004,005; 007→001,004,005,006; 008→001,004,005; 009→001,004,005; 010→001,002,005,007…; 011→002,003,209; 012→002,109,110; 013→001; 014→020,124,125; 015→111,124; 016→109; 017→102,103,109,112…; 018→102 |
| SUC | 13 | 001→091,092,093,127…; 002→135,136,137; 003→135,136,137; 004→135,136,137,146…; 005→138,150; 006→141,142,151; 007→135,143; 008→145,152,154,157; 009→154,156,353; 010→169,170,171; 011→127; 012→160,163,165,167; 013→160,161,162 |
| GEN | 41 | 001→102,112,123,127…; 002→357,358,359; 003→154,203,255,312…; 004→002,360; 005→360,362; 006→363; 007→365; 008→105,364; 009→106,130,131,173…; 010→396; 011→010,403,413; 012→003,030,385,387…; 013→049,076,114,216…; 014→389; 015→041,095,097,402; 016→031,041; 017→032,040,044,076…; 018→032,034,041,125; 019→059,061,064,288; 020→082,083,084,089…; 021→082,094,246,247; 022→087,238,255,258; 023→169,170,172,250…; 024→135,143,160,166…; 025→392; 026→022,057,072,080…; 027→011,394; 028→sin caso (parte B); 029→395; 030→397; 031→398; 032→399; 033→400; 034→401; 035→037,405; 036→139,193,200,223…; 037→140,407; 038→140,408; 039→370,409; 040→023,073,081,411; 041→231,410 |
| AUT | 39 | 001→012,016,017; 002→012,016; 003→012,013,019; 004→012,019,024; 005→019,025,027; 006→018,025; 007→025,026,028,387; 008→012,019; 009→012,020,021; 010→014,015,029,058…; 011→014; 012→014; 013→035,047; 014→031,035; 015→031,033; 016→038; 017→033,039; 018→031,036; 019→037; 020→043,047; 021→043,044; 022→045,046,048; 023→040,042,085,086…; 024→049,054; 025→049,052; 026→006,049,052; 027→049,052; 028→056; 029→053,062; 030→049,050,055,060; 031→024,049,055,075…; 032→065,071; 033→065,067; 034→065,067,068,126; 035→065,068,069; 036→065,066,070; 037→075,077,078; 038→075,079; 039→030,076 |
| ROL | 23 | 001→091,095,208,209…; 002→216,218,221; 003→216,220,222; 004→216,218,219,224; 005→220,223; 006→226,228,229; 007→226,232,255,259…; 008→255,257; 009→216,228,255,258; 010→234,235; 011→236,238,239; 012→238,241,242; 013→096,243; 014→234; 015→082,083,087,237; 016→238,239,244,246; 017→249,250,251; 018→252,253; 019→208; 020→260,261,265; 021→261,262; 022→264,266; 023→208 |
| CTA | 20 | 001→091,268,274,285…; 002→274,276,277,283; 003→274,276,278; 004→274,276,279; 005→274; 006→281; 007→002,009,207,280…; 008→058,060,274; 009→286,287,292,312…; 010→286,289; 011→061,288; 012→039,286,287; 013→312,314,316,318; 014→295,302,303,305…; 015→295,296; 016→295,296,299; 017→295,300,301; 018→042,298; 019→305,306,309,311; 020→268,270 |
| EPL | 18 | 001→091,092,093,153…; 002→182,183,186,195; 003→182,183,186,195; 004→182,186,187; 005→182,186; 006→188,198; 007→188; 008→186,189,190; 009→074,189,191,192…; 010→195,197; 011→197,199,201; 012→198,201; 013→195,202,203,206; 014→203,204,205; 015→207; 016→173; 017→176,177; 018→182,183,184,185 |
| SER | 14 | 001→091,319,328,339…; 002→328,329,341; 003→328,330,331; 004→328,330,331,332…; 005→334,354,355; 006→334,355; 007→328,335,341,342; 008→328,336,337,341…; 009→345,346,349,350; 010→345,346; 011→349,350; 012→337,342,343; 013→354,356; 014→322 |
| IMP | 17 | 001→368,369; 002→369; 003→372; 004→370; 005→371; 006→167,261,266,371…; 007→167,261,266,376…; 008→107,132,179,213…; 009→107,132,179,213…; 010→107,380,383; 011→325,326,384; 012→373; 013→373; 014→160,161,261,262…; 015→376; 016→159,260,366,367; 017→107,108,132,133… |

### 6.6 Verificaciones del script

| Verificación | Resultado |
|---|---|
| Los 413 casos tienen IDs únicos y consecutivos de `TC-ADM-001` a `TC-ADM-413` | Sí |
| Todos los `RN-…` citados existen en [01a][rn] | Sí |
| Todos los `HU-ADM-…` y `CU-ADM-…` citados existen en [12a][hu] y [13a][cu] | Sí |
| Cada título de escenario citado entre « » existe en su historia | Sí (0 referencias sin escenario) |
| Todos los `TC-ADM-…` mencionados en pasos y precondiciones existen | Sí |
| Las filas «Tipo, Prioridad, Cubre» de todos los casos tienen tres celdas | Sí |
| Este documento no contiene bloques de código (los esquemas se resolvieron con tablas) | Sí (0 delimitadores) |
| Las definiciones de enlaces del final del documento apuntan a archivos que existen | Sí |
| Los 26 casos de la suite de humo son de prioridad Alta | Sí |

## 7. Fuentes

Documentos consultados (rutas relativas a la raíz de `sofia-docs`):

- [Historias de usuario, parte A][hu] (`ola-2-funcional/12-historias-de-usuario/12a-acceso-y-administracion.md`): fuente principal de escenarios, mensajes y datos.
- [Casos de uso, parte A][cu] (`ola-2-funcional/13-casos-de-uso/13a-acceso-y-administracion.md`): flujos, excepciones por paso, códigos de error y mensajes.
- [Catálogo de reglas de negocio, parte A][rn] (`ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md`): identificadores `RN-…` y su existencia.
- [Inventario de pantallas, acceso y compartidos][p19a] y [administración][p19b].
- [Autenticación y controles][p05a], [matriz de permisos][p05b] y [catálogo de códigos de error][p21].
- [Carga masiva y exportación a Excel][p20].
- [Procesos de negocio][p14] (procesos 1 y 2) y [glosario][p06].
- Manual de usuario: [capítulo 1][m1], [capítulo 2][m2] (secciones de sucursales, series y carga masiva) y [capítulo 3][m3] (secciones 5.3 a 5.5 para los permisos recomendados del cajero).
- [Guía del desarrollador][g08] (sección 3, entorno local), [despliegue y operación][g10] y [pruebas automáticas del backend][p09a].
- Código, solo para comprobar nombres de propiedades de una petición: `sofia-apis/src/SOFIA.Application/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommand.cs:12-26` y la lista de controladores de `sofia-apis/src/SOFIA.API/Controllers`. El resto de nombres de propiedades de los cuerpos JSON se marca «No verificado» en cada caso.

No se usaron los `README.md` ni `CLAUDE.md` de los repositorios, ni `swagger.json`, ni la memoria del proyecto.

[hu]: ../../ola-2-funcional/12-historias-de-usuario/12a-acceso-y-administracion.md
[cu]: ../../ola-2-funcional/13-casos-de-uso/13a-acceso-y-administracion.md
[rn]: ../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md
[p19a]: ../../ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md
[p19b]: ../../ola-1-fundamentos/19-inventario-de-pantallas/19b-administracion.md
[m1]: ../../ola-2-funcional/manual-usuario/capitulo-01-primeros-pasos.md
[m2]: ../../ola-2-funcional/manual-usuario/capitulo-02-empresa-sucursales-series.md
[m3]: ../../ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md
[g08]: ../../ola-1-fundamentos/08-guia-del-desarrollador.md
[g10]: ../../ola-1-fundamentos/10-despliegue-y-operacion.md
[p05a]: ../../ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md
[p05b]: ../../ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md
[p21]: ../../ola-1-fundamentos/21-catalogo-codigos-error.md
[p20]: ../../ola-1-fundamentos/20-carga-masiva-y-exportacion.md
[p09a]: ../../ola-1-fundamentos/09-pruebas/09a-backend.md
[p14]: ../../ola-2-funcional/14-procesos-de-negocio.md
[p06]: ../../ola-1-fundamentos/06-glosario.md
