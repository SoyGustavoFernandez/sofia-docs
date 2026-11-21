# Capítulo 12 — Operador de plataforma: suscripciones de las empresas y datos de pago

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-11-20 |
| **Backend** | `sofia-apis`, rama `main`, commit `0678e40` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `37048ea` |
| **Método** | Lectura de código (`sofia-apis` `6f6b837` y `sofia-web-spa` `558ad35`). **Este capítulo no se recorrió en la aplicación**: todo lo que dice es **No verificado en la aplicación** |

## Índice

1. Introducción
2. Quién es el operador de plataforma
3. Cómo cambia el menú del operador
4. Configuración de Pago
5. Cambiar la suscripción de una empresa
6. La sección «Suscripciones» del Inicio
7. Qué ve la empresa que no paga
8. Errores que puedes ver
9. Pendientes y «No verificado»
10. Tabla de cobertura
11. Fuentes

## 1. Introducción

Este capítulo es para la **persona de SOFIA que administra las suscripciones de todas las empresas**, el operador de plataforma. Una botica normal no ve nada de lo que aquí se explica. En este capítulo aprendes a:

- Publicar los datos de pago (código QR de Yape, número de Yape, número de WhatsApp e indicaciones) que ven las empresas que deben pagar.
- Activar la suscripción de una empresa hasta una fecha, extender su prueba, suspenderla o cancelarla.
- Leer las cifras de suscripciones del Inicio.

Cómo se ve esto desde la botica está en el [capítulo 1](capitulo-01-primeros-pasos.md) (sección 6.5.1) y en el [capítulo 10](capitulo-10-inicio-y-roles.md) (sección 3.9).

**Ideas que debes tener claras**

- **El acceso de una empresa termina solo en su fecha de vencimiento**, tanto en la prueba como en la suscripción pagada. Una empresa está vigente solo si su estado es «Trial Activo» o «Activo» **y** su fecha de vencimiento todavía no pasó (`sofia-apis/src/SOFIA.Domain/Entities/Empresa.cs`, `EstaVigenteEn`).
- **SOFIA no cobra en línea.** La empresa paga por Yape y envía la captura por WhatsApp; el operador comprueba el pago por fuera de SOFIA y luego activa la suscripción desde la pantalla de la empresa.
- **Suspender y cancelar** quitan el acceso de inmediato. **Activar la suscripción** lo devuelve, también a una empresa suspendida o cancelada.

## 2. Quién es el operador de plataforma

El operador no es un rol ni un permiso de los que ves en «Roles & Permisos»: es una marca de la cuenta en la base de datos, la columna `Es_Admin_Plataforma` de `Seguridad_Cuentas` (script `0029_Plataforma_Suscripciones.sql`). El script dice que esa marca **se pone a mano y nunca desde la aplicación**: no existe ninguna pantalla para crear un operador ni para quitarle la marca. Mientras la cuenta tenga la marca, el token de sesión lleva la señal `plat_admin` y la API vuelve a comprobarla contra la base de datos en cada petición (`sofia-apis/src/SOFIA.Application/Security/PlataformaAdminPolicy.cs`).

Reglas del operador (fuente: `sofia-apis/src/SOFIA.Application/Security/EmpresaVigencia.cs`, `sofia-apis/src/SOFIA.Application/Empresas/Common/EmpresaPlataformaExtensions.cs` y `sofia-apis/src/SOFIA.API/Controllers/EmpresasController.cs`):

| Regla | Detalle |
|---|---|
| Nunca se queda fuera de su propia empresa | Su cuenta puede iniciar sesión aunque la suscripción de su empresa esté vencida, suspendida o cancelada: nadie más podría reactivarla |
| No puede suspender ni cancelar su propia empresa | Responde con el aviso «No puedes suspender ni cancelar tu propia empresa.» |
| Lee todas las empresas | Las pantallas de Empresa le muestran las empresas de todos, aunque su rol no tenga `Empresa › Leer`. Para el resto de la aplicación sigue valiendo el rol de su cuenta |
| Edita solo su propia empresa | El botón **Editar** (nombre y RUC) solo aparece cuando la empresa abierta es la suya |

## 3. Cómo cambia el menú del operador

| Dónde | Qué ve el operador |
|---|---|
| Menú ADMINISTRACIÓN › **Empresa** (`/empresas`) | La búsqueda con **todas** las empresas, con las columnas «Empresa», «RUC», «Estado», «Vencimiento» y «Vigente» y los filtros «Nombre de la Empresa», «Estado», «Vencimiento desde» y «Vencimiento hasta» (capítulo 2, sección 3) |
| Menú ADMINISTRACIÓN › **Configuración de Pago** (`/configuracion-pago`) | La pantalla de la sección 4. Solo la ve el operador |
| Inicio | La sección «Suscripciones» (sección 6) |

**Configuración de Pago** no aparece en la vista previa del menú de un rol (capítulo 3, sección 5): ningún rol de una empresa puede darla. Si alguien que no es operador escribe `/configuracion-pago` en la barra de direcciones, el sistema lo devuelve al Inicio con el aviso «No tienes permisos para acceder a esta pantalla.» (`sofia-web-spa/src/app/core/auth/platform-admin.guard.ts`).

## 4. Configuración de Pago

**Cómo llegar:** menú lateral › ADMINISTRACIÓN › **Configuración de Pago**. Migas «Inicio / Administración / Configuración de Pago».

Es **un solo registro** para todo SOFIA, no uno por empresa. La tarjeta se titula «Configuración de Pago», con el texto «Estos datos se muestran a las empresas que deben pagar su suscripción.». Entra en modo ver, con los campos deshabilitados; haz clic en **Editar** para cambiarlos y usa **Cancelar** o **Guardar** abajo a la derecha.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| WhatsApp para recibir capturas | El número de WhatsApp al que las empresas enviarán la captura. Ejemplo: `51987654321` | Sí | De 8 a 15 dígitos, con código de país, sin espacios ni signos. Vacío: «Este campo es obligatorio». Con otro formato: «Ingresa de 8 a 15 dígitos, con código de país y sin espacios ni signos» |
| Número de Yape (opcional) | El número de Yape, para quien no pueda escanear el código | No | Exactamente 9 dígitos. Con otro formato: «Ingresa exactamente 9 dígitos» |
| Indicaciones de pago (opcional) | Por ejemplo, el monto y el periodo que cubre | No | Hasta 500 caracteres: «La longitud máxima es de 500 caracteres» |
| Código QR de Yape | Una imagen: botón **Elegir imagen** (o **Cambiar imagen** si ya hay una) | No | PNG o JPG de hasta 1 MB. Otro formato: «La imagen debe ser PNG o JPG.» Más de 1 MB: «La imagen no puede pesar más de 1 MB.» Mientras no hay imagen dice «Aún no has subido un código QR.»; una imagen elegida y aún no guardada dice «Imagen nueva, se publicará al guardar» |

Qué pasa al guardar:

- Aparece «Configuración de pago guardada.» y la pantalla vuelve al modo ver con los datos recargados.
- Si no eliges una imagen nueva, el QR que ya estaba publicado **se conserva**. Para cambiarlo, elige una imagen nueva. No hay un botón para quitar el QR.
- El servidor no se fía de la extensión del archivo: reconoce PNG o JPEG por el contenido del archivo y rechaza lo demás (`ConfiguracionPago.Qr.Formato`, `sofia-apis/src/SOFIA.Domain/Entities/PlataformaConfiguracionPago.cs`).
- A las indicaciones se les quitan los espacios sobrantes del principio y del final; el número de Yape y las indicaciones en blanco se guardan como vacíos.
- Los datos publicados los lee **cualquiera, sin iniciar sesión** (`GET /api/v1/Plataforma/pago`, con un límite de 30 peticiones por minuto por equipo), porque los necesita la pantalla de inicio de sesión de una empresa bloqueada. No escribas aquí nada que no quieras mostrar al público.
- Solo el operador puede guardar: la API responde 403 con «Auth.PlataformaAdminRequerido» a cualquier otra cuenta.

Avisos de error de la pantalla: «No se pudo cargar la configuración de pago.» (al abrirla) y «No se pudo guardar la configuración de pago.» (genérico al guardar).

## 5. Cambiar la suscripción de una empresa

**Cómo llegar:** ADMINISTRACIÓN › **Empresa** › **Buscar** › clic en el nombre de la empresa. En el detalle, arriba a la derecha, el operador ve estos botones (el resto de los usuarios ve solo **Editar** en su propia empresa):

| Botón | Cuándo aparece | Qué hace |
|---|---|---|
| **Activar suscripción** | Siempre | Pide una fecha y deja la empresa en estado «Activo» hasta esa fecha. Sirve para empresas en prueba, vencidas, suspendidas o canceladas |
| **Extender prueba** | Solo si la empresa está en «Trial Activo» | Pide una fecha y la deja como nuevo vencimiento de la prueba |
| **Suspender** | Si no es tu propia empresa y no está ya «Suspendido» | Quita el acceso de inmediato, tras confirmar |
| **Cancelar suscripción** | Si no es tu propia empresa y no está ya «Cancelado» | Quita el acceso de inmediato y deja la empresa «Cancelado», tras confirmar |

Mientras una acción se ejecuta, los botones se deshabilitan.

### 5.1 Activar la suscripción o extender la prueba

1. Haz clic en **Activar suscripción** (o en **Extender prueba**). Se abre un cuadro con el título «Activar suscripción» (o «Extender prueba») y este texto: «Elige hasta qué día queda activa la suscripción de {empresa}. Al llegar esa fecha el acceso se bloquea solo.» (o «Elige hasta qué día dura el periodo de prueba de {empresa}.»).
2. Elige la fecha en «Vigente hasta». El campo no se escribe a mano: usa el calendario. La fecha mínima es **mañana**.
3. Como atajo, usa los botones «1 mes», «3 meses», «6 meses» o «12 meses». La cuenta parte de la fecha de vencimiento actual de la empresa si todavía no pasó, o de hoy si ya pasó, para que una renovación pagada a tiempo no le quite días a la empresa. Al abrir el cuadro, la fecha ya viene puesta en «1 mes».
4. Haz clic en **Confirmar** (o en **Cancelar** para salir sin cambiar nada).

Qué pasa al terminar bien: aparece «Suscripción activada.» (o «Periodo de prueba extendido.») y el detalle se recarga con el nuevo estado y la nueva fecha. La empresa conserva el acceso durante **todo el día elegido**, hasta las 23:59:59 en la hora de tu navegador.

Reglas que aplica la API (fuente: `sofia-apis/src/SOFIA.Domain/Entities/Empresa.cs` y los comandos de `sofia-apis/src/SOFIA.Application/Empresas/Commands/`):

- La fecha tiene que ser posterior al momento actual; si no, «La fecha de vencimiento debe ser posterior a hoy.»
- **Activar suscripción** cambia el estado a «Activo» y **reemplaza** la fecha de vencimiento por la elegida (no suma días).
- **Extender prueba** solo funciona si la empresa sigue en «Trial Activo»; si no, «Solo se puede extender la prueba de una empresa que sigue en periodo de prueba.» También **reemplaza** la fecha.
- Una suscripción «Activa» ya no dura indefinidamente: vence en la fecha elegida igual que la prueba. Antes de `6f6b837` activar una suscripción daba 365 días fijos desde ese momento; ahora el operador elige la fecha.

### 5.2 Suspender o cancelar

1. Haz clic en **Suspender** o en **Cancelar suscripción**.
2. Se abre un cuadro de confirmación. Para suspender: título «Suspender empresa» y texto «{empresa} perderá el acceso de inmediato. Puedes devolvérselo activando su suscripción.». Para cancelar: título «Cancelar suscripción» y texto «{empresa} perderá el acceso de inmediato y quedará como cancelada. Puedes devolvérselo activando su suscripción.»
3. Confirma. Aparece «Empresa suspendida.» o «Suscripción cancelada.».

Suspender y cancelar solo cambian el estado; no tocan la fecha de vencimiento. Las dos son reversibles con **Activar suscripción**. Sobre tu propia empresa responden «No puedes suspender ni cancelar tu propia empresa.» y el botón ni siquiera aparece.

### 5.3 Qué le pasa a la empresa

- En el siguiente inicio de sesión, la empresa recibe «La suscripción de tu empresa no está activa o ha vencido. Realiza el pago para reactivarla.» y ve el recuadro de pago (capítulo 1, sección 6.5.1).
- La misma regla se aplica al renovar la sesión y en cada petición, así que quien ya tenía la sesión abierta pierde el acceso sin esperar a cerrarla (según el comentario de `Empresa.EstaVigenteEn`: «login, token refresh and every authenticated request»). **No verificado en la aplicación**: no se probó con una sesión abierta.
- El correo de recuperación de contraseña tampoco sale para una empresa que no puede operar (capítulo 1, sección 8).

## 6. La sección «Suscripciones» del Inicio

Cuando entra el operador, el Inicio muestra, después de la tarjeta de cabecera, una sección titulada «Suscripciones» con las cifras de **todas** las empresas (`GET /api/v1/Plataforma/dashboard`, solo operador). Las cifras salen en tarjetas:

| Tarjeta | Qué cuenta |
|---|---|
| Empresas vigentes | Las que están en «Trial Activo» o «Activo» y todavía no vencieron. Debajo: «N en prueba · N con suscripción» |
| Vencen en 7 días o menos | Las vigentes cuyo vencimiento cae en los próximos 7 días |
| Vencidas sin renovar | Las que están en «Trial Activo» o «Activo» pero cuya fecha ya pasó |
| Suspendidas | Las que están en «Suspendido». Con el enlace «Ver lista» (abre Empresa filtrada por ese estado) |
| Canceladas | Las que están en «Cancelado». Con el enlace «Ver lista» |
| Registradas este mes | Las empresas creadas desde el primer día del mes, según la hora de tu navegador. Debajo: «N empresas en total» |

Debajo de las tarjetas, la lista «Próximas a vencer (7 días)» muestra las empresas que vencen pronto, de la más próxima a la más lejana, con las columnas «Empresa» (enlace al detalle de la empresa), «Tipo» («Prueba» o «Suscripción»), «Vence» (`dd/MM/yyyy`) y «Días restantes». Muestra como máximo 10 y, si hay más, el texto «Se muestran las {n} más próximas de {total}.»; si no hay ninguna, «Ninguna empresa vence en los próximos días.» Al final, el botón «Ver todas las empresas» abre la búsqueda de empresas sin filtros.

Si falla la carga de la sección aparece un aviso con el botón de reintentar. Como todo el Inicio, **no se actualiza solo**: usa **Actualizar** (capítulo 10, sección 7). Una cuenta de operador sin ningún permiso de empresa igual ve esta sección, y no ve el aviso «Tu usuario no tiene indicadores disponibles aquí».

## 7. Qué ve la empresa que no paga

| Momento | Qué ve la empresa | Dónde está explicado |
|---|---|---|
| Faltan 7 días o menos para el vencimiento | Una franja sobre el Inicio con «Tu periodo de prueba vence en N días (fecha).» o «Tu suscripción vence en N días (fecha).» y el botón **Cómo pagar**. Solo la ve quien puede leer la empresa (el administrador) | Capítulo 10, sección 3.9 |
| Ya venció, o está suspendida o cancelada | No puede iniciar sesión. Ve el aviso de suscripción y el recuadro «Paga tu suscripción con Yape» con los datos que cargaste en la sección 4 | Capítulo 1, sección 6.5.1 |
| Todavía no cargaste datos de pago | El recuadro dice «Todavía no hay datos de pago publicados. Comunícate con soporte para reactivar tu suscripción.» | Capítulo 1, sección 6.5.1 |

El botón **Enviar captura por WhatsApp** abre `https://wa.me/<tu número de WhatsApp>` con un mensaje ya escrito que lleva el usuario con el que intentó entrar (en el inicio de sesión) o el nombre de la empresa (en el aviso del Inicio), para que sepas a quién activar.

## 8. Errores que puedes ver

| Mensaje | Dónde sale | Qué significa | Qué hacer |
|---|---|---|---|
| «No se pudo cambiar la suscripción.» | Aviso inferior en el detalle de la empresa | Mensaje general cuando la API rechaza el cambio por otra causa | Recarga el detalle y repite |
| «La fecha de vencimiento debe ser posterior a hoy.» | Aviso inferior al activar o extender | La fecha ya pasó | Elige una fecha desde mañana |
| «Solo se puede extender la prueba de una empresa que sigue en periodo de prueba.» | Aviso inferior al extender | La empresa ya no está en «Trial Activo» | Usa **Activar suscripción** |
| «No puedes suspender ni cancelar tu propia empresa.» | Aviso inferior | Intentaste bloquear la empresa del operador | No se puede |
| «No tienes permisos para esta operación.» | Aviso inferior | Tu cuenta no tiene la marca de operador (la API respondió 403) | Pide que se te marque como operador en la base de datos |
| «No tienes permisos para acceder a esta pantalla.» | Aviso al abrir `/configuracion-pago` | No eres operador | — |
| «No se pudo cargar la configuración de pago.» / «No se pudo guardar la configuración de pago.» | Aviso inferior en Configuración de Pago | Falló la lectura o el guardado | Repite; si sigue, revisa los datos |
| «Ingresa de 8 a 15 dígitos, con código de país y sin espacios ni signos», «Ingresa exactamente 9 dígitos», «La imagen debe ser PNG o JPG.», «La imagen no puede pesar más de 1 MB.» | Bajo el campo | Validación del formulario (y de la API, con los mismos límites) | Corrige el dato |

Códigos de la API detrás de estos avisos (fuente: `sofia-web-spa/src/app/pages/empresa/empresa-detail/empresa-detail.component.ts` y `sofia-web-spa/src/app/pages/plataforma/configuracion-pago/configuracion-pago.component.ts`): `Empresa.Suscripcion.NoEstaEnPrueba` y `Empresa.Suscripcion.PropiaEmpresa` (409), `Empresa.Suscripcion.FechaVencimiento` (validación), `Auth.PlataformaAdminRequerido` (403), `Empresa.NotFound` (404), `ConfiguracionPago.WhatsappNumero`, `ConfiguracionPago.YapeNumero`, `ConfiguracionPago.Qr.Formato` y `ConfiguracionPago.Qr.Tamano` (400). Una imagen de más de 1 MB se rechaza con `ConfiguracionPago.Qr.Tamano`; el resto de la petición tiene un margen de 64 KB sobre ese tamaño (`PlataformaController.MaxRequestBytes`). Las indicaciones de más de 500 caracteres se rechazan con `ConfiguracionPago.Instrucciones`.

## 9. Pendientes y «No verificado»

| # | Tema | Detalle |
|---|---|---|
| 1 | Todo el capítulo | No se recorrió en la aplicación: no existe en la empresa de prueba una cuenta de operador (la marca solo se pone en la base de datos) y este trabajo no usa el navegador. Sale del código de `sofia-apis` `6f6b837` y `sofia-web-spa` `558ad35` |
| 2 | Pantallas sin recorrer | Configuración de Pago, los cuatro botones y los cuadros de la sección 5, la sección «Suscripciones» del Inicio, el aviso de vencimiento y el recuadro de pago del inicio de sesión |
| 3 | Efecto sobre una sesión abierta | Si una empresa suspendida pierde el acceso en la siguiente petición, o solo al renovar la sesión |
| 4 | Cómo se marca a un operador | Solo se confirmó por el script `0029` que es manual en la base de datos; el procedimiento exacto no está documentado aquí |
| 5 | Historial de suscripciones y pagos | No existe: SOFIA no guarda quién pagó, cuánto ni cuándo. El estado y la fecha son lo único que queda |
| 6 | **En curso (sin commitear)** | Ninguno: ambos repositorios estaban sin cambios locales |

## 10. Tabla de cobertura

| Elemento del código | Dónde quedó | Estado |
|---|---|---|
| `PlataformaController` (`GET pago`, `GET dashboard`, `PUT pago`) | Secciones 4 y 6 | No verificado en la aplicación |
| `EmpresasController` (`PUT {id}/suscripcion/activar`, `extender-prueba`, `suspender`, `cancelar`) | Sección 5 | No verificado en la aplicación |
| `PlataformaAdminPolicy`, `HasPermissionOrPlataforma`, `Es_Admin_Plataforma` | Sección 2 | No verificado en la aplicación |
| `Empresa.EstaVigenteEn`, `ActivarSuscripcion`, `ExtenderTrial` | Secciones 1 y 5 | No verificado en la aplicación |
| `PlataformaConfiguracionPago` y script `0029` | Sección 4 | No verificado en la aplicación |
| `suscripcion/es.json`, `empresa/es.json` (bloque `suscripcion`) y `dashboard/es.json` (bloque `plataforma`) | Secciones 4 a 8 | Textos copiados del archivo |
| Aviso de vencimiento y recuadro de pago | Sección 7 y capítulos 1 y 10 | No verificado en la aplicación |

## 11. Fuentes

- `sofia-apis`: `src/SOFIA.API/Controllers/PlataformaController.cs`, `src/SOFIA.API/Controllers/EmpresasController.cs`, `src/SOFIA.API/Extensions/RateLimitingExtensions.cs`, `src/SOFIA.Application/Security/EmpresaVigencia.cs`, `src/SOFIA.Application/Security/PlataformaAdminPolicy.cs`, `src/SOFIA.Application/Empresas/Common/EmpresaPlataformaExtensions.cs`, `src/SOFIA.Application/Empresas/Commands/` (`ActivarSuscripcionEmpresa`, `ExtenderPruebaEmpresa`, `SuspenderEmpresa`, `CancelarEmpresa`), `src/SOFIA.Application/Plataforma/`, `src/SOFIA.Domain/Entities/Empresa.cs`, `src/SOFIA.Domain/Entities/PlataformaConfiguracionPago.cs`, `src/SOFIA.DbUp/Scripts/0029_Plataforma_Suscripciones.sql`.
- `sofia-web-spa`: `src/app/app.routes.ts`, `src/app/core/auth/` (`platform-admin.guard.ts`, `route-permissions.ts`, `auth.service.ts`), `src/app/pages/plataforma/configuracion-pago/`, `src/app/pages/empresa/empresa-detail/`, `src/app/pages/empresa/dialogs/suscripcion-fecha-dialog/`, `src/app/pages/dashboard/`, `src/app/shared/components/pago-suscripcion/`, `public/i18n/suscripcion/es.json`, `public/i18n/empresa/es.json`, `public/i18n/dashboard/es.json`, `public/i18n/auth/es.json`.
