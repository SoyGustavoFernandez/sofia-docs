# Capítulo 1 — Primeros pasos

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-14 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo aprendes a dar tus primeros pasos en SOFIA:

- Registrar tu botica o farmacia y crear a su administrador.
- Reconocer la pantalla principal: el menú lateral, la cabecera y el Inicio.
- Cerrar sesión e iniciar sesión.
- Cambiar tu contraseña.
- Recuperar tu contraseña si la olvidas.
- Saber qué ocurre cuando tu sesión vence.

Es para el **administrador** (la persona que registra la empresa). El **cajero** también usa las partes de cerrar sesión, iniciar sesión, cambiar contraseña, recuperar contraseña y sesión vencida (secciones 5 a 9), y reconoce la pantalla principal (sección 4) con los menús que su rol permite; el registro de la empresa lo hace una sola vez el administrador.

Todos los ejemplos del manual usan datos ficticios: la empresa «Botica Manual Demo S.A.C.», la administradora «Ana Demo Prueba» y correos del dominio `sofia-manual.test`.

## 2. Antes de empezar

- Necesitas un navegador y la dirección de SOFIA que te dio tu proveedor. En el entorno de pruebas del manual es `http://localhost:4200`.
- Para registrar la empresa **no** necesitas permisos ni cuenta previa: la pantalla es pública.
- Necesitas un correo electrónico real al que tengas acceso: ahí llegará el enlace si olvidas tu contraseña (ver sección 8).
- Para entrar a las demás pantallas del sistema se necesitan permisos, pero el administrador que crea la empresa recibe el rol **Admin** y lo puede todo. Los permisos de otros usuarios los define el administrador (capítulo 3).
- Si tu empresa tiene la suscripción vencida o suspendida, no podrás entrar (ver «Errores que puedes ver» en la sección 6.5).

## 3. Registrar tu empresa

**Cómo llegar:** abre `http://localhost:4200/auth/register`, o en la pantalla «Iniciar sesión» haz clic en **Registrar empresa nueva** (está al pie de la tarjeta, después de «¿Primera vez?» y del separador «o»).

### 3.1 Cómo es la pantalla

La pantalla tiene dos zonas:

- **Panel de marca** (a la izquierda; en pantallas angostas, de menos de 900 píxeles, queda en una sola columna): el logo «SOFIA», el título «Gestión farmacéutica completa», el texto «Una plataforma inteligente para administrar tu farmacia con precisión y eficiencia.», cuatro viñetas («1 mes de prueba gratuita, sin tarjeta», «Multi-sucursal desde el primer día», «Control de inventario y ventas en tiempo real», «Soporte dedicado durante la incorporación») y el pie «© 2026 SOFIA · Todos los derechos reservados». Solo informan; no se hace clic en ellos.
- **Formulario** (a la derecha): el título «Registrar Empresa» y el subtítulo «Crea tu empresa y obtén 1 mes de prueba gratis». Tiene tres secciones: «DATOS DE LA EMPRESA», «DATOS DEL ADMINISTRADOR» y «CUENTA DE ACCESO», un botón **Registrar Empresa** y, debajo, «¿Ya tienes cuenta?» con el enlace **Iniciar sesión**.

### 3.2 Pasos

1. Escribe el **Nombre de la Empresa**.
2. Escribe el **RUC** (11 dígitos).
3. Escribe la **Dirección de la sede principal**. Es la dirección de tu primera sucursal; bajo el campo se lee «Aparecerá en el encabezado de tus boletas».
4. En «DATOS DEL ADMINISTRADOR», escribe los **Nombres del administrador**, el **Apellido paterno** y, si quieres, el **Apellido materno (opcional)**.
5. Escribe tu **Correo electrónico**. Bajo el campo se lee «Lo usarás para recuperar tu contraseña».
6. En «CUENTA DE ACCESO», elige el **Usuario Administrador** (con el que entrarás al sistema) y la **Contraseña**. Con el icono del ojo, a la derecha de la contraseña, puedes mostrarla u ocultarla (`aria-label` «Mostrar u ocultar contraseña»).
7. Haz clic en **Registrar Empresa**. Mientras el sistema trabaja, el botón muestra «Procesando...» y no se puede volver a pulsar; puede tardar unos segundos.

### 3.3 Campos

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombre de la Empresa | La razón social, por ejemplo «Botica Manual Demo S.A.C.» | Sí | Vacío: «Este campo es obligatorio». Más de 200 caracteres: «Máximo 200 caracteres» |
| RUC | 11 dígitos numéricos, por ejemplo `20538291746` | Sí | Vacío: «Este campo es obligatorio». Con menos de 11 dígitos o con una letra: «El RUC debe tener exactamente 11 dígitos numéricos». El campo no deja escribir más de 11 caracteres. El sistema no comprueba el dígito verificador |
| Dirección de la sede principal | Calle y número de tu primera sucursal, por ejemplo «Av. Ficticia 123, Miraflores, Lima» (el campo muestra de ejemplo «Av. Los Olivos 123, Lima») | Sí | Vacío o solo espacios: «Este campo es obligatorio». Más de 255 caracteres: «Máximo 255 caracteres» (el campo deja escribirlos, pero no te dejará registrar) |
| Nombres del administrador | Tus nombres, por ejemplo «Ana» | Sí | Vacío: «Este campo es obligatorio». Más de 75 caracteres: «Máximo 75 caracteres» |
| Apellido paterno | Tu primer apellido, por ejemplo «Demo» | Sí | Vacío: «Este campo es obligatorio». Más de 75 caracteres: «Máximo 75 caracteres» |
| Apellido materno (opcional) | Tu segundo apellido, por ejemplo «Prueba» | No | Más de 75 caracteres: «Máximo 75 caracteres». Vacío no da error |
| Correo electrónico | Un correo tuyo válido, por ejemplo `ana.demo@sofia-manual.test` | Sí | Vacío: «Este campo es obligatorio». Mal escrito (por ejemplo «ana.demo»): «Ingresa un correo electrónico válido». Más de 254 caracteres: «Máximo 254 caracteres» |
| Usuario Administrador | El nombre con el que entrarás, de 4 a 50 caracteres, por ejemplo `ana.manual` | Sí | Vacío: «Este campo es obligatorio». Menos de 4 caracteres: «El usuario debe tener al menos 4 caracteres». Más de 50: «Máximo 50 caracteres» |
| Contraseña | Una contraseña de tu elección, de 8 caracteres o más | Sí | Vacía: «Este campo es obligatorio». Menos de 8 caracteres: «La contraseña debe tener al menos 8 caracteres» |

Observaciones sobre este formulario (comprobadas en pantalla):

- Los mensajes de error aparecen debajo de cada campo cuando sales de él, o cuando haces clic en **Registrar Empresa** con el formulario incompleto. En ese caso el sistema marca todos los campos con error y **no** envía nada.
- Algunos mensajes tardan alrededor de un segundo en dibujarse. Espera antes de concluir que no hay error.
- **No existe un campo para repetir (confirmar) la contraseña.** Escríbela con cuidado; puedes usar el ojo para verla.
- La contraseña solo exige 8 caracteres como mínimo: no obliga a usar mayúsculas, números ni símbolos. Aun así te recomendamos usar una larga que mezcle letras, números y símbolos.
- El nombre de usuario puede llevar letras, números, puntos, etc.; el sistema no pone otro requisito que su longitud. Recuérdalo bien, porque lo necesitarás para entrar y para recuperar tu contraseña.
- El nombre y la licencia de la sede no se piden aquí: el sistema le pone el nombre «Sede Principal» y deja el número de licencia vacío (se completa después, desde Sucursales; capítulo 3).

### 3.4 Qué pasa al terminar

1. Aparece un aviso en la parte inferior central de la pantalla: «Empresa registrada correctamente. ¡Bienvenido!» (con un botón **Cerrar**).
2. Entras directamente al **Inicio** (`/dashboard`) con tu sesión ya iniciada. No hay pantalla de confirmación ni correo de verificación: no tienes que volver a escribir tu usuario y contraseña.
3. El saludo de la parte superior del Inicio dice «Hola, Ana Demo Prueba» (tus nombres y apellidos).

El sistema crea por ti, en una sola operación (si algo falla, no se crea nada):

| Qué crea | Detalle (comprobado en la aplicación) |
|---|---|
| La empresa | Aparece en **Administración › Empresa** con el estado **Trial Activo** y marcada como vigente. Tiene 1 mes (30 días) de prueba gratuita desde el momento del registro |
| La sede principal | En **Administración › Sucursales** hay una sola sucursal llamada «Sede Principal», con la dirección que escribiste y el «Nº de Licencia» vacío |
| Tu empleado | Con tus nombres, apellidos y el correo que escribiste |
| Tu cuenta de usuario | Con el usuario y la contraseña que elegiste; **no** te obliga a cambiar la contraseña al entrar |
| El rol «Admin» | En **Administración › Roles & Permisos** aparece el rol «Admin», con la descripción «Administrador de la empresa» y nivel de jerarquía 0, asignado a tu cuenta. Con este rol puedes ver y hacer todo en el sistema |

### 3.5 Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe una empresa registrada con este RUC» (aviso en la parte inferior central, con el botón **Cerrar**) | Otra empresa ya usa ese RUC. El RUC es único en todo el sistema | Revisa que lo hayas escrito bien. Si es el RUC de tu empresa y nunca la registraste, comunícate con soporte |
| «El usuario ya está en uso. Elige otro.» | Otro usuario del sistema ya se llama así (el nombre de usuario es único en todas las empresas) | Elige otro nombre de usuario, por ejemplo agregando tu apellido o un número |
| «Ingresa la dirección de la sede principal (máximo 255 caracteres)» | El servidor rechazó la dirección (vacía o demasiado larga). El formulario normalmente lo impide antes | Escribe una dirección de hasta 255 caracteres |
| «Error al registrar la empresa» | Cualquier otro problema: el servidor no responde, un dato no pasó la validación del servidor, o hiciste demasiados envíos | Revisa tus datos y vuelve a intentarlo en unos minutos. El registro tiene un límite de envíos por hora desde el mismo equipo; si lo superas, espera |

> (No verificado en la aplicación: los avisos «El usuario ya está en uso. Elige otro.», «Ingresa la dirección de la sede principal (máximo 255 caracteres)» y «Error al registrar la empresa» no se provocaron porque el registro tiene un límite de envíos por hora; se documentan desde el inventario de pantallas, 19a §1.2.)

## 4. Conocer la pantalla principal

Cuando entras al sistema ves siempre la misma estructura: el **menú lateral** a la izquierda, la **cabecera** arriba y, debajo de ella, la pantalla en la que estás. Las pantallas de acceso (iniciar sesión, registrar empresa, recuperar contraseña) no tienen menú ni cabecera.

### 4.1 Menú lateral

**Cómo llegar:** está siempre a la izquierda mientras tengas la sesión iniciada. Arriba muestra la marca «⚕ SOFIA»; al hacer clic en ella vas al Inicio.

Cada grupo tiene un título en mayúsculas y debajo sus opciones. Estas son las que ve el **administrador** (el rol «Admin» lo ve todo): 8 grupos y 24 opciones.

| Grupo (título) | Opciones | Ruta |
|---|---|---|
| INICIO | Dashboard | `/dashboard` |
| ADMINISTRACIÓN | Empresa | `/empresas` |
| | Sucursales | `/sucursales` |
| | Empleados | `/empleados` |
| | Roles & Permisos | `/roles` |
| | Cuentas de Usuario | `/cuentas` |
| CATÁLOGOS | Unidades de Medida | `/unidades-medida` |
| | Jerarquías de Unidades | `/jerarquias` |
| | Laboratorios | `/laboratorios` |
| | Ingredientes Activos | `/ingredientes-activos` |
| | Medicamentos | `/medicamentos` |
| | Presentaciones de Venta | `/presentaciones-venta` |
| | Formulaciones Clínicas | `/formulaciones-clinicas` |
| | Proveedores | `/proveedores` |
| | Aseguradoras | `/aseguradoras` |
| INVENTARIO | Lotes | `/lotes` |
| | Stock por Sucursal | `/stock-por-sucursal` |
| PUNTO DE VENTA | Sesiones de Caja | `/sesiones-caja` |
| | Ventas | `/pos` |
| PACIENTES & ATENCIÓN | Pacientes | `/pacientes` |
| | Profesionales de Salud | `/profesionales-salud` |
| | Recetas Médicas | `/recetas-medicas` |
| DIGEMID | Catálogo DIGEMID | `/digemid-catalogo` |
| SUNAT | Series Fiscales | `/series-fiscales` |

Cómo se comporta el menú:

- Cada opción lleva un icono a la izquierda del nombre. La opción de la pantalla en la que estás aparece resaltada.
- Haz clic en una opción para abrir esa pantalla. Con **Ctrl** + clic (o clic con la rueda del ratón) se abre en una pestaña nueva del navegador.
- **El menú depende de tus permisos.** Una opción solo aparece si tu rol tiene permiso de lectura (`Leer`) sobre su módulo; si un grupo se queda sin ninguna opción, desaparece entero. «Dashboard» siempre está. El cajero, por ejemplo, ve menos opciones (capítulo 3 explica cómo se define). Mientras el sistema carga tus permisos, al principio solo se ve «Dashboard».
- Si no se pueden cargar tus permisos aparece el aviso «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.» y el menú queda solo con «Dashboard».
- Ocultar una opción no es lo único que te protege: si escribes a mano la dirección de una pantalla para la que no tienes permiso, el sistema te devuelve al Inicio con el aviso «No tienes permisos para acceder a esta pantalla.» (No verificado en la aplicación: el administrador tiene todos los permisos y no se puede provocar; viene de 19a §5.3).
- El botón de las tres rayas, a la izquierda de la cabecera, **colapsa** el menú: queda una columna angosta (80 píxeles) solo con los iconos, y al volver a hacer clic se abre completo. Esta preferencia no se guarda: al recargar la página el menú vuelve a verse abierto. En pantallas angostas (hasta 1023 píxeles) el menú queda cerrado y se abre sobre la pantalla con ese mismo botón; se cierra solo al elegir una opción. (No verificado en la aplicación: la vista angosta; viene de 19a §4.)
- No hay un buscador de opciones en el menú ni en la cabecera.
- Las pantallas de Transferencias, Delivery, Magistrales, Servicios, Cuarentena y Actas DIGEMID y Auditoría no tienen opción de menú: todavía no están disponibles en la interfaz.

### 4.2 Cabecera

Es la barra de la parte superior. De izquierda a derecha:

| Elemento | Qué es | ¿Hace algo? |
|---|---|---|
| Botón de tres rayas | Colapsa o abre el menú lateral (ver 4.1) | Sí |
| Fecha de hoy | Texto como «miércoles 14 de octubre de 2026» (en inglés, «Wednesday, October 14, 2026») | Solo informa. Se calcula al abrir la pantalla: si dejas la ventana abierta pasada la medianoche no se actualiza |
| Campana | Al hacer clic abre un pequeño menú que dice solamente «Sin notificaciones» | **No hace nada útil**: es decorativa; el sistema todavía no genera notificaciones |
| Bandera de idioma (su texto alternativo es «Idioma») | Abre un menú con «Español» y «English» | Sí (ver 4.3) |
| Imagen de perfil (su etiqueta es «Menú de usuario») | Abre el menú de usuario (ver 4.4). Es la misma imagen para todos los usuarios | Sí |

No hay buscador, ni selector de tema claro u oscuro, ni de color.

### 4.3 Cambiar el idioma

1. Haz clic en la bandera de idioma.
2. Elige «Español» o «English». Si eliges el idioma que ya está activo, no pasa nada.
3. Aparece un cuadro de confirmación con el título «Cambiar idioma», el mensaje «La interfaz se actualizará al idioma seleccionado. ¿Deseas continuar?» y los botones **Cancelar** y **Confirmar** (en inglés: «Change language», «The interface will update to the selected language. Do you want to continue?», **Cancel** y **Confirm**).
4. **Cancelar** cierra el cuadro y no cambia nada. **Confirmar** traduce de inmediato el menú, la cabecera y la pantalla (el saludo pasa a «Hello, Ana Demo Prueba»).

Importante: el idioma **no se guarda**. Si recargas la página, cierras sesión o vuelves a entrar, la interfaz vuelve a español (No verificado en la aplicación: viene de 19a §4.5). Las pantallas de acceso (iniciar sesión, registro, recuperar contraseña) están siempre en español y no tienen selector de idioma. El idioma también cambia el formato de las fechas (en español, día/mes/año).

### 4.4 Menú de usuario

1. Haz clic en la imagen de perfil de la cabecera.
2. Se abre una tarjeta con:
   - Tu nombre completo en mayúsculas (por ejemplo «ANA DEMO PRUEBA») y, debajo, el nombre de tu empresa («Botica Manual Demo S.A.C.»). No son clicables.
   - **Cambiar contraseña**: te lleva a la pantalla de cambio de contraseña (sección 7). Es la única forma de llegar a ella cuando no es obligatoria.
   - **Cerrar sesión** (se ve «CERRAR SESIÓN» en mayúsculas): sale del sistema (sección 5).
   - «Versión: 1.0.0»: la versión de SOFIA que estás usando. Menciónala si llamas a soporte.
3. Para cerrar la tarjeta sin elegir nada, haz clic fuera de ella o presiona **Esc**.

### 4.5 El Inicio (dashboard)

Al registrarte o iniciar sesión llegas al **Inicio** (`/dashboard`, opción «Dashboard» del menú). Es el resumen del día de tu empresa. Su detalle completo se explica en el capítulo 10; aquí solo lo reconoces.

Lo que ves, de arriba hacia abajo, en una empresa recién creada (todo en cero):

- Una tarjeta de cabecera con el saludo «Hola, Ana Demo Prueba» (tu nombre), «Este es el resumen de hoy.», «Actualizado a las {hora}» y el botón **Actualizar**, que vuelve a pedir los datos (el Inicio no se refresca solo).
- **Ventas de hoy**: «Vendido hoy» (S/ 0.00), «Ayer», «Comprobantes», «Ticket promedio», «Mis ventas de hoy», «Ventas en espera» y «Cobrado por método de pago» (Efectivo, Yape / Plin, Tarjeta, Transferencia), con los botones **Ver ventas de hoy**, **Ver mis ventas** y **Ver ventas en espera**.
- **Caja**: «No tienes una caja abierta», con el botón **Abrir caja**.
- **Inventario**: lotes vencidos, lotes que vencen en 30 días o menos, entre 31 y 60 y entre 61 y 90, productos sin stock, productos con stock bajo y lotes en cuarentena.
- **Ventas de los últimos 7 días** y **Productos más vendidos en 7 días** («Aún no hay ventas en los últimos 7 días.»).
- **Devoluciones de hoy**.

Cada bloque depende de tus permisos: solo ves los bloques de los módulos que puedes leer. Cuando la empresa es nueva y aún no registraste nada, todas las cifras salen en 0 o «S/ 0.00».

## 5. Cerrar sesión

**Cómo llegar:** cabecera › imagen de perfil («Menú de usuario») › **Cerrar sesión**.

1. Haz clic en la imagen de perfil, en la esquina superior derecha.
2. Haz clic en **Cerrar sesión**.

**Qué pasa al terminar:**

- El sistema te lleva a la pantalla «Iniciar sesión» (`/auth/login`). No pide confirmación y no muestra ningún mensaje.
- Se cierran **todas** tus sesiones abiertas de esa cuenta, también las de otros navegadores o equipos.
- Se borran del navegador tus datos locales de la sesión (por ejemplo, los borradores de venta del punto de venta y los filtros de búsqueda guardados).
- Si intentas abrir una pantalla del sistema (por ejemplo `/dashboard`) sin haber iniciado sesión, el sistema te lleva a «Iniciar sesión» sin ningún aviso.

Si no ves el menú de usuario, no es por falta de permisos: todos los usuarios que han iniciado sesión lo ven.

## 6. Iniciar sesión

**Cómo llegar:** abre `http://localhost:4200/auth/login` (también es adonde el sistema te lleva solo cuando no tienes sesión), o haz clic en **Iniciar sesión** en las pantallas de registro y de recuperación de contraseña.

### 6.1 Cómo es la pantalla

A la izquierda, el panel de marca: el logo «SOFIA», «Gestión completa de tu farmacia», «Sistema Optimizado Farmacéutico con Inteligencia Artificial», cuatro viñetas («Control de medicamentos y lotes», «Gestión de recetas y pacientes», «Reportes DIGEMID automatizados», «Seguridad y auditoría completa») y el pie «© 2026 SOFIA · v1.0.0». En pantallas angostas (de 768 píxeles o menos) este panel se oculta.

A la derecha, la tarjeta «Iniciar sesión», con el subtítulo «Ingresa tus credenciales para acceder al sistema», los campos **Usuario** y **Contraseña**, el botón **Ingresar**, el enlace **¿Olvidaste tu contraseña?** y, al pie, «¿Primera vez?» con el enlace **Registrar empresa nueva**.

### 6.2 Pasos

1. Escribe tu **Usuario** (el campo muestra de ejemplo «Ej. jperez»).
2. Escribe tu **Contraseña**. Con el icono del ojo (a la derecha del campo) puedes mostrarla u ocultarla.
3. Haz clic en **Ingresar**. Mientras el sistema verifica tus datos, el botón muestra un círculo de espera y no se puede volver a pulsar.

### 6.3 Campos

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Usuario | El nombre de usuario que elegiste al registrarte, o el que te asignó tu administrador (no hace falta escribir la empresa: el usuario es único en todo el sistema) | Sí | Vacío: «El usuario es obligatorio» (aparece al hacer clic en **Ingresar** o al salir del campo) |
| Contraseña | Tu contraseña | Sí | Vacía: «La contraseña es obligatoria» |

### 6.4 Qué pasa al terminar

- Si tus datos son correctos, entras al **Inicio** (`/dashboard`) sin ningún aviso. Si tu contraseña fue asignada por un administrador y debes cambiarla, entras directamente a la pantalla «Cambiar contraseña» (ver sección 9).
- El sistema no tiene «recordarme» ni vuelve a la pantalla en la que estabas: siempre entras al Inicio (o a «Cambiar contraseña»).
- Tu sesión se mantiene sola mientras trabajas (ver sección 9). Si recargas la página con la sesión vigente, sigues dentro.

### 6.5 Errores que puedes ver

El mensaje aparece en una alerta roja con un icono de error, entre el subtítulo y el campo Usuario.

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Credenciales inválidas. Verifica tu usuario y contraseña.» | El usuario o la contraseña no coinciden, o la cuenta está desactivada o bloqueada. El sistema no dice cuál de las dos cosas falló, a propósito, para proteger tu cuenta | Revisa mayúsculas y minúsculas. Si no recuerdas la contraseña, usa **¿Olvidaste tu contraseña?** (sección 8) |
| «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» | La contraseña era correcta, pero la empresa tiene la suscripción vencida, suspendida o cancelada | Comunícate con soporte. (No verificado en la aplicación: la empresa de prueba está vigente; viene de 19a §1.1) |
| «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | Se hicieron demasiados intentos de inicio de sesión desde tu equipo en poco tiempo (el límite es de 10 por minuto) | Espera un minuto. (No verificado en la aplicación: no se provocó para no bloquear la cuenta de prueba) |
| «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.» | El servidor no responde o tu conexión falló | Revisa tu internet y vuelve a intentar. (No verificado en la aplicación) |

**Bloqueo de la cuenta.** Si te equivocas de contraseña **5 veces seguidas**, la cuenta se bloquea durante **15 minutos**. Mientras está bloqueada, ni siquiera la contraseña correcta funciona, y la pantalla muestra el mismo mensaje «Credenciales inválidas. Verifica tu usuario y contraseña.»: no te avisa del bloqueo. Espera 15 minutos o recupera tu contraseña por correo (sección 8), que además quita el bloqueo. Un inicio de sesión correcto pone el contador de errores en cero. (No verificado en la aplicación: no se provocó el bloqueo; viene de la regla RN-AUT-007 en `01a-administracion-y-seguridad.md`.)

### 6.6 Otros enlaces de la pantalla

- **¿Olvidaste tu contraseña?** (bajo el botón **Ingresar**): abre la pantalla de recuperación (sección 8).
- **Registrar empresa nueva** (al pie): abre el registro (sección 3).

## 7. Cambiar tu contraseña

Cambia tu contraseña cuando la quieras renovar o si crees que alguien más la conoce. Cualquier usuario con la sesión iniciada puede hacerlo; no se necesita ningún permiso.

**Cómo llegar:** cabecera › imagen de perfil («Menú de usuario») › **Cambiar contraseña**. También llegas ahí a la fuerza cuando tu contraseña fue asignada por un administrador (sección 9).

### 7.1 Cómo es la pantalla

Arriba, un título «Cambiar contraseña» con un botón de volver (una flecha a la izquierda, «Volver») y las migas «Inicio» › «Cambiar contraseña». Debajo, una tarjeta con el título «Cambiar contraseña» y el aviso «Al guardar se cerrarán todas tus sesiones y tendrás que iniciar sesión con tu nueva contraseña». Luego, tres campos y, al final, los botones **Cancelar** y **Guardar**.

### 7.2 Pasos

1. Escribe tu **Contraseña actual**.
2. Escribe la **Nueva contraseña** (8 caracteres o más, distinta de la actual).
3. Escribe la misma contraseña en **Confirmar nueva contraseña**. Con el icono del ojo, a la derecha de este campo (su etiqueta es «Nueva contraseña»), puedes mostrar u ocultar los tres campos a la vez.
4. Haz clic en **Guardar**. Mientras guarda, el botón muestra un círculo de espera.

### 7.3 Campos

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Contraseña actual | La contraseña con la que entraste | Sí | Vacía: «Este campo es obligatorio». Si no coincide con la tuya: «La contraseña actual es incorrecta» (ver errores) |
| Nueva contraseña | La contraseña nueva | Sí | Vacía: «Este campo es obligatorio». Menos de 8 caracteres: «Debe tener al menos 8 caracteres». Igual a la actual: «La nueva contraseña debe ser distinta de la actual». No se exigen mayúsculas, números ni símbolos, pero conviene usarlos |
| Confirmar nueva contraseña | La misma contraseña nueva | Sí | Vacía: «Este campo es obligatorio». Distinta de la nueva: «Las contraseñas no coinciden» (aparece debajo del campo) |

Los mensajes aparecen debajo de cada campo al salir de él o al hacer clic en **Guardar** con el formulario incompleto; en ese caso no se envía nada.

### 7.4 Botones

| Botón | Qué hace |
|---|---|
| **Guardar** | Cambia la contraseña (ver «Qué pasa al terminar») |
| **Cancelar** | Vuelve al Inicio sin cambiar nada. Solo aparece cuando el cambio no es obligatorio |
| Flecha «Volver» (arriba a la izquierda) | Vuelve al Inicio. Solo aparece cuando el cambio no es obligatorio |
| Ojo | Muestra u oculta las tres contraseñas |
| «Inicio» (miga) | Vuelve al Inicio |

### 7.5 Qué pasa al terminar

1. Aparece el aviso «Contraseña actualizada. Inicia sesión con tu nueva contraseña.» (con un botón **Cerrar**).
2. El sistema **cierra tu sesión** (y todas las demás sesiones de tu cuenta) y te lleva a «Iniciar sesión».
3. Entra con tu usuario y la contraseña nueva. La anterior ya no sirve.

### 7.6 Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «La contraseña actual es incorrecta» (aviso inferior con el botón **Cerrar**; te quedas en la pantalla) | La contraseña actual que escribiste no es la tuya | Vuelve a escribirla. Si no la recuerdas, cierra sesión y usa la recuperación por correo (sección 8) |
| «No se pudo cambiar la contraseña. Inténtalo de nuevo.» | Falló el servidor o tu sesión ya no es válida | Inténtalo otra vez; si persiste, cierra sesión, entra de nuevo y repite |

> (No verificado en la aplicación: «No se pudo cambiar la contraseña. Inténtalo de nuevo.» no se pudo provocar; viene de 19a §2.)

La versión «obligatoria» de esta pantalla (con el aviso «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.», sin **Cancelar** ni flecha de volver) se explica en la sección 9 y se ve en el capítulo 3.

## 8. Recuperar tu contraseña olvidada

Si no recuerdas tu contraseña, puedes crear una nueva sin ayuda del administrador, siempre que tu empleado tenga un correo registrado. El sistema te envía un enlace a ese correo. No necesitas tener la sesión iniciada ni ningún permiso.

El enlace se envía al correo del **empleado** ligado a tu usuario: si registraste la empresa, es el correo del formulario de registro; si eres cajero, el que el administrador escribió en tu ficha de empleado (capítulo 3).

### 8.1 Pedir el enlace

**Cómo llegar:** en «Iniciar sesión», haz clic en **¿Olvidaste tu contraseña?** (`/auth/forgot-password`).

La pantalla tiene el mismo panel de marca que el inicio de sesión y, a la derecha, el título «Recupera tu contraseña», el texto «Ingresa tu usuario y te enviaremos un enlace al correo registrado de tu empleado para crear una nueva contraseña.», el campo **Usuario**, el botón **Enviar enlace** y el enlace **Volver a iniciar sesión**.

1. Escribe tu **Usuario**.
2. Haz clic en **Enviar enlace**. El botón muestra un círculo de espera mientras el sistema trabaja (puede tardar unos segundos).

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Usuario | Tu nombre de usuario (no el correo) | Sí | Vacío: «El usuario es obligatorio». El campo no deja escribir más de 50 caracteres |

**Qué pasa al terminar:** el formulario se reemplaza por una alerta verde con un icono de correo y el mensaje «Si el usuario existe y tiene un correo registrado, recibirás un enlace para restablecer tu contraseña. Revisa tu bandeja de entrada: el enlace es válido por 1 hora.» Solo queda el enlace **Volver a iniciar sesión**.

Importante: el mensaje es **siempre el mismo**, exista o no el usuario y tenga o no correo. Es a propósito, para que nadie pueda averiguar qué usuarios existen. Por eso, si no llega el correo, revisa que escribiste bien el usuario y revisa la carpeta de correo no deseado. El sistema solo envía el enlace si la cuenta está activa, su empleado tiene correo y la suscripción de la empresa está vigente.

Errores:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | Hiciste demasiadas solicitudes seguidas | Espera un minuto. (No verificado en la aplicación: no se provocó) |
| «No se pudo procesar la solicitud. Inténtalo de nuevo.» | Falló el servidor o tu conexión | Inténtalo otra vez. (No verificado en la aplicación: no se provocó) |

Si pides el enlace dos veces, **solo el último sirve**: el nuevo reemplaza al anterior.

### 8.2 El correo que recibes

En el entorno de pruebas del manual, los correos no salen a internet: se leen en la bandeja de pruebas Mailpit (`http://localhost:8025`). En una instalación real, el correo llega a tu bandeja de entrada normal. El contenido es el mismo:

| Dato | Valor |
|---|---|
| Remitente | «SOFIA» (dirección `no-reply@sofia.local` en el entorno de pruebas) |
| Para | El correo del empleado (por ejemplo `ana.demo@sofia-manual.test`) |
| Asunto | «Restablece tu contraseña de SOFIA» |
| Llega | A los pocos segundos de pedir el enlace |

El mensaje dice: «Hola, Ana:» (tu nombre), «Recibimos una solicitud para restablecer la contraseña de tu usuario ana.manual en SOFIA.», «Haz clic en el botón para elegir una nueva contraseña:», un botón **Restablecer contraseña**, «El enlace es válido por 1 hora y solo puede usarse una vez.», «Si no lo solicitaste, ignora este correo: tu contraseña actual seguirá funcionando.» y, como alternativa, «Si el botón no funciona, copia y pega esta dirección en tu navegador:» seguido de la dirección completa. Al pie, «Sistema Optimizado Farmacéutico con Inteligencia Artificial».

La dirección del enlace es `.../auth/reset-password` seguida de un código largo y secreto y de tu usuario. **No compartas ese enlace con nadie**: quien lo tenga puede cambiar tu contraseña durante esa hora.

Dos datos de seguridad que conviene conocer:

- El enlace caduca a la **1 hora** y solo sirve **una vez**. Si ya lo usaste o caducó, verás «El enlace no es válido o ya expiró. Solicita uno nuevo.» (ver 8.3).
- Si no pediste el cambio, ignora el correo: tu contraseña no cambia.

### 8.3 Crear la nueva contraseña

**Cómo llegar:** abre el correo y haz clic en **Restablecer contraseña** (o pega la dirección en el navegador). Se abre `/auth/reset-password` con el título «Crea una nueva contraseña» y el texto «Elige una contraseña que no hayas usado antes. Al guardarla se cerrarán todas tus sesiones abiertas.» La pantalla no muestra tu usuario: viaja oculto en el enlace, y la dirección del navegador se limpia sola al abrir la pantalla.

1. Escribe la **Nueva contraseña**.
2. Escribe la misma contraseña en **Confirmar nueva contraseña**. Con el ojo, a la derecha del primer campo (su etiqueta es «Mostrar u ocultar contraseñas»), puedes ver u ocultar las dos.
3. Haz clic en **Guardar contraseña**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nueva contraseña | Una contraseña nueva, de 8 caracteres o más | Sí | Vacía: «La contraseña es obligatoria». Menos de 8 caracteres: «Debe tener al menos 8 caracteres» |
| Confirmar nueva contraseña | La misma contraseña | Sí | Vacía: «La contraseña es obligatoria». Distinta de la nueva: «Las contraseñas no coinciden» |

Los mensajes aparecen al salir del campo o al hacer clic en **Guardar contraseña** con el formulario incompleto (en ese caso no se envía nada).

**Qué pasa al terminar:**

1. Aparece el aviso «Contraseña restablecida. Inicia sesión con tu nueva contraseña.» (con el botón **Cerrar**).
2. El sistema te lleva a «Iniciar sesión». Todas las sesiones abiertas de tu cuenta se cierran.
3. Entra con tu usuario y la contraseña nueva. Además, el restablecimiento **quita el bloqueo** de la cuenta si lo había y pone en cero el contador de intentos fallidos (esto último no se verificó en la aplicación; viene de la regla RN-AUT-039).

Errores y estados:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Este enlace de recuperación está incompleto o ya fue usado. Solicita uno nuevo.» (alerta roja con el botón **Solicitar un nuevo enlace**; el formulario no aparece) | Abriste `/auth/reset-password` sin el código del enlace, o con un enlace cortado | Haz clic en **Solicitar un nuevo enlace** (te lleva a la sección 8.1) o usa el enlace completo del correo |
| «El enlace no es válido o ya expiró. Solicita uno nuevo.» (alerta roja sobre el formulario) | El enlace ya se usó, caducó (pasó más de 1 hora) o fue reemplazado por uno más nuevo | Vuelve a **¿Olvidaste tu contraseña?** y pide un enlace nuevo |
| «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | Demasiados envíos seguidos | Espera un minuto. (No verificado en la aplicación) |
| «No se pudo restablecer la contraseña. Inténtalo de nuevo.» | Falló el servidor o tu conexión | Inténtalo otra vez. (No verificado en la aplicación) |

En las pantallas de recuperación, el enlace **Volver a iniciar sesión** te lleva al inicio de sesión sin cambiar nada.

## 9. Sesión vencida y cambio obligatorio de contraseña

### 9.1 Qué pasa cuando tu sesión vence

SOFIA mantiene tu sesión viva mientras trabajas: cuando tu acceso caduca (dura unos 15 minutos), el sistema lo renueva solo, en segundo plano, y no te enteras. Una sesión puede mantenerse hasta 7 días sin volver a escribir tu contraseña.

Solo se te pide volver a entrar cuando la renovación ya no es posible, por ejemplo:

- Pasaron más de 7 días sin usar SOFIA.
- Cerraste sesión (o cambiaste o restableciste tu contraseña) en otro navegador o equipo, lo que cierra todas las sesiones de tu cuenta.
- Un administrador desactivó tu cuenta.
- La suscripción de tu empresa dejó de estar vigente.

En esos casos, el sistema te lleva a «Iniciar sesión» **sin mostrar ningún aviso**: de repente ves la pantalla de inicio de sesión. Para continuar, escribe tu usuario y contraseña otra vez. Lo que hubieras escrito en un formulario sin guardar se pierde, así que guarda con frecuencia.

(No verificado en la aplicación: no se esperó a que la sesión venciera; viene de 19a §5.1 y §5.4 y de las reglas RN-AUT-015 a RN-AUT-019. Lo que sí se comprobó es que, sin sesión, abrir una pantalla como `/dashboard` lleva a «Iniciar sesión» sin ningún aviso, ver sección 5. También se comprobó que al recargar la página con la sesión vigente sigues dentro.)

### 9.2 Cambio obligatorio de contraseña

Cuando un administrador crea la cuenta de otro usuario (por ejemplo, un cajero) y le asigna una contraseña inicial, esa cuenta puede quedar marcada con «debe cambiar la contraseña». Entonces, la primera vez que ese usuario inicia sesión:

1. Al hacer clic en **Ingresar**, en lugar del Inicio ve la pantalla «Cambiar contraseña».
2. Arriba aparece el aviso «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.»
3. No tiene el botón **Cancelar** ni la flecha de volver. El menú lateral se ve, pero cualquier opción que elija lo devuelve a esta pantalla.
4. Debe escribir la contraseña actual (la que le dio el administrador), la nueva y su confirmación, y hacer clic en **Guardar**. Al terminar, el sistema cierra la sesión y debe entrar de nuevo con la contraseña nueva.

El administrador que registra la empresa **no** pasa por este paso: su cuenta se crea con la contraseña que él mismo eligió. Este cambio obligatorio se ve paso a paso en el capítulo 3, cuando el administrador crea la cuenta de un cajero.

(No verificado en la aplicación en este capítulo: todavía no existe ninguna cuenta marcada con cambio obligatorio; viene de 19a §2 y §5.2.)

## 10. Lo que ve un cajero frente a un administrador

| Pantalla o elemento | Administrador (rol «Admin») | Cajero (u otro rol con permisos limitados) |
|---|---|---|
| Registrar empresa | Lo hace una sola vez al comenzar; la pantalla es pública | No lo usa: su cuenta la crea el administrador (capítulo 3) |
| Iniciar sesión, cerrar sesión, recuperar contraseña | Igual | Igual. Un cajero recupera su contraseña con el correo que el administrador registró en su ficha de empleado |
| Cambiar contraseña | Voluntario: tiene **Cancelar** y la flecha de volver | Voluntario igual; pero si el administrador le asignó una contraseña inicial, la primera vez es **obligatorio** (sección 9.2) |
| Menú lateral | Las 24 opciones en 8 grupos (sección 4.1) | Solo las opciones de los módulos sobre los que su rol tiene el permiso `Leer`; los grupos sin opciones desaparecen. «Dashboard» siempre aparece |
| Cabecera | Igual para todos: fecha, campana, idioma, menú de usuario | Igual |
| Menú de usuario | Nombre, empresa, **Cambiar contraseña**, **Cerrar sesión**, versión | Igual |
| Inicio (dashboard) | Ve el total de toda la empresa (y, con varias sucursales, el desglose por sucursal) | Ve solo los bloques de los módulos que puede leer y los datos de sus sucursales; sus propias ventas y su caja aparecen en «Mis ventas de hoy» y «Caja». El detalle está en el capítulo 10 |

(No verificado en la aplicación en este capítulo: todavía no existe ningún cajero en la empresa de prueba; la columna del cajero viene de 19a §3.2 y §4.3, y se verá en vivo en los capítulos 3 y 10.)

## 11. Diferencias con la documentación técnica

El inventario `19a-acceso-layout-y-compartidos.md` coincidió con lo que hace la aplicación en todo lo comprobado. Solo se anotan estas diferencias menores:

| # | Documento y apartado | Dice | La aplicación muestra |
|---|---|---|---|
| 1 | 19a §2 (`/cambiar-clave`, «Migas y cabecera») | Describe el título «Cambiar contraseña» y las migas, pero no menciona el texto de apoyo de la tarjeta | La tarjeta tiene además el título «Cambiar contraseña» y el texto «Al guardar se cerrarán todas tus sesiones y tendrás que iniciar sesión con tu nueva contraseña» (clave `cambiarClave.subtitle`, en `sofia-web-spa/public/i18n/cambiar-clave/es.json:3`) |
| 2 | 19a §4.2 (menú de usuario, tarjeta superior) | El nombre es el `fullName` del token | Se ve en mayúsculas («ANA DEMO PRUEBA») y «Cerrar sesión» también («CERRAR SESIÓN»): es un estilo visual; el texto guardado es «Ana Demo Prueba» y «Cerrar sesión» |
| 3 | 19a §1.2 (mensajes del registro) | «Ya existe una empresa registrada con este RUC» | Coincide en la pantalla (aviso inferior con el botón **Cerrar**). La respuesta técnica del servidor lleva un punto final («...con este RUC.»), que el usuario no ve |

## 12. Pendientes y «No verificado»

No verificado en la aplicación (cada caso, con su motivo):

| # | Qué | Motivo |
|---|---|---|
| 1 | Avisos «El usuario ya está en uso. Elige otro.», «Ingresa la dirección de la sede principal (máximo 255 caracteres)» y «Error al registrar la empresa» | El registro tiene un límite de 5 envíos por hora desde el mismo equipo; ya se usaron los necesarios. Vienen de 19a §1.2 |
| 2 | Mensaje de login «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» | La empresa de prueba está vigente; no se puede cambiar su estado desde la pantalla |
| 3 | Mensajes «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» y «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.» (login, recuperación, restablecimiento) | No se provocaron para no bloquear el servidor ni la cuenta de prueba |
| 4 | Bloqueo de la cuenta tras 5 intentos fallidos (15 minutos) | No se provocó, según el encargo; viene de RN-AUT-007 |
| 5 | «No se pudo cambiar la contraseña. Inténtalo de nuevo.», «No se pudo procesar la solicitud. Inténtalo de nuevo.» y «No se pudo restablecer la contraseña. Inténtalo de nuevo.» | Son errores genéricos del servidor; no se pueden provocar desde la pantalla |
| 6 | Vencimiento de la sesión y renovación en segundo plano | No se esperó a que vencieran los 15 minutos; viene de 19a §5.1 y §5.4 |
| 7 | Cambio obligatorio de contraseña (pantalla en modo forzado) | No existe aún una cuenta marcada con ese cambio; se verá en el capítulo 3 |
| 8 | Vista angosta (menos de 1024 píxeles): menú lateral como superposición, y panel de marca oculto con 768 píxeles o menos | No se recorrió; viene de 19a §4 y §1.1 |
| 9 | Ctrl + clic en una opción del menú para abrirla en otra pestaña | No se probó; viene de 19a §4.3 |
| 10 | Mensaje «No tienes permisos para acceder a esta pantalla.» y el menú reducido de un cajero | El administrador tiene todos los permisos; se verá en los capítulos 3 y 10 |
| 11 | Contador de intentos en cero y desbloqueo tras restablecer la contraseña | Efecto interno; viene de RN-AUT-039 |
| 12 | Que el idioma elegido no se conserva al recargar la página, cerrar sesión o volver a entrar | Se volvió a español a mano al terminar la prueba, sin recargar; viene de 19a §4.5 |

En curso (sin commitear): ninguno. Los dos repositorios estaban sin cambios locales al empezar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`).

## 13. Tabla de cobertura

Cada elemento de las fichas de `19a-acceso-layout-y-compartidos.md` de las pantallas del alcance, con el apartado donde quedó.

### 13.1 `/auth/login` (19a §1.1)

| Elemento | Apartado |
|---|---|
| Panel de marca (logo, título, sistema, cuatro viñetas, pie con versión), se oculta con 768 píxeles o menos | 6.1 (la vista angosta, No verificado) |
| Título «Iniciar sesión» y subtítulo «Ingresa tus credenciales para acceder al sistema» | 6.1 |
| Campo Usuario (obligatorio, mensaje «El usuario es obligatorio», ejemplo «Ej. jperez») | 6.2, 6.3 |
| Campo Contraseña (obligatorio, mensaje «La contraseña es obligatoria») | 6.2, 6.3 |
| Icono del ojo (mostrar u ocultar contraseña) | 6.2 |
| Botón **Ingresar** (con círculo de espera; entra a `/dashboard` o a `/cambiar-clave`) | 6.2, 6.4 |
| Enlace **¿Olvidaste tu contraseña?** | 6.6, 8.1 |
| «¿Primera vez?» y enlace **Registrar empresa nueva** | 6.1, 6.6, 3 |
| Mensaje «Credenciales inválidas. Verifica tu usuario y contraseña.» | 6.5 (verificado) |
| Mensaje «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» | 6.5 (No verificado en la aplicación: la empresa está vigente) |
| Mensaje «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | 6.5 (No verificado en la aplicación: no se provocó) |
| Mensaje «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.» | 6.5 (No verificado en la aplicación: no se provocó) |
| Sin «recordarme» ni regreso a la pantalla anterior | 6.4 |
| Bloqueo de 5 intentos / 15 minutos | 6.5 (No verificado en la aplicación: no se provocó) |
| Scope `login` sin uso | No aplica: es una observación técnica sin efecto para el usuario |

### 13.2 `/auth/register` (19a §1.2)

| Elemento | Apartado |
|---|---|
| Panel de marca propio (titular, texto, cuatro viñetas, pie) | 3.1 |
| Título «Registrar Empresa» y subtítulo «Crea tu empresa y obtén 1 mes de prueba gratis» | 3.1 |
| Secciones «DATOS DE LA EMPRESA», «DATOS DEL ADMINISTRADOR», «CUENTA DE ACCESO» | 3.1, 3.2 |
| Nombre de la Empresa (obligatorio, máximo 200) | 3.3 (verificado) |
| RUC (11 dígitos) | 3.3 (verificado) |
| Dirección de la sede principal (obligatoria, máximo 255, no solo espacios, ayuda y ejemplo) | 3.3 (verificado) |
| Nombres del administrador (máximo 75) | 3.3 (verificado) |
| Apellido paterno (máximo 75) | 3.3 (verificado) |
| Apellido materno (opcional, máximo 75) | 3.3 (verificado) |
| Correo electrónico (formato, máximo 254, ayuda «Lo usarás para recuperar tu contraseña») | 3.3 (verificado) |
| Usuario Administrador (4 a 50) | 3.3 (verificado) |
| Contraseña (mínimo 8) y su ojo | 3.2, 3.3 (verificado) |
| Botón **Registrar Empresa** y estado «Procesando...» | 3.2 |
| Enlace **Iniciar sesión** y «¿Ya tienes cuenta?» | 3.1 |
| Aviso «Empresa registrada correctamente. ¡Bienvenido!» y entrada al Inicio | 3.4 (verificado) |
| Aviso «Ya existe una empresa registrada con este RUC» | 3.5 (verificado) |
| Aviso «El usuario ya está en uso. Elige otro.» | 3.5 (No verificado en la aplicación: límite de envíos) |
| Aviso «Ingresa la dirección de la sede principal (máximo 255 caracteres)» | 3.5 (No verificado en la aplicación: límite de envíos) |
| Aviso «Error al registrar la empresa» | 3.5 (No verificado en la aplicación: límite de envíos) |
| Qué crea el registro: empresa en prueba, «Sede Principal», empleado, cuenta, rol «Admin» | 3.4 (verificado en Empresa, Sucursales y Roles) |

### 13.3 `/auth/forgot-password` (19a §1.3) y panel compartido (§1.5)

| Elemento | Apartado |
|---|---|
| Título «Recupera tu contraseña» y subtítulo | 8.1 |
| Campo Usuario (obligatorio, máximo 50, «El usuario es obligatorio») | 8.1 |
| Botón **Enviar enlace** (círculo de espera) | 8.1 |
| Enlace **Volver a iniciar sesión** | 8.1, 8.3 |
| Aviso verde «Si el usuario existe y tiene un correo registrado, recibirás un enlace para restablecer tu contraseña. Revisa tu bandeja de entrada: el enlace es válido por 1 hora.» | 8.1 (verificado) |
| «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | 8.1 (No verificado en la aplicación: no se provocó) |
| «No se pudo procesar la solicitud. Inténtalo de nuevo.» | 8.1 (No verificado en la aplicación: no se provocó) |
| Panel de marca compartido (`app-auth-brand-panel`) | 8.1 |
| Vigencia del enlace (1 hora) | 8.2 (comprobado en el texto del correo) |

### 13.4 `/auth/reset-password` (19a §1.4)

| Elemento | Apartado |
|---|---|
| Título «Crea una nueva contraseña» y subtítulo | 8.3 |
| Enlace del correo con el código en el fragmento y limpieza de la dirección | 8.3 (verificado) |
| Campo Nueva contraseña («La contraseña es obligatoria», «Debe tener al menos 8 caracteres») | 8.3 (verificado) |
| Campo Confirmar nueva contraseña («La contraseña es obligatoria», «Las contraseñas no coinciden») | 8.3 (verificado) |
| Ojo («Mostrar u ocultar contraseñas») | 8.3 |
| Botón **Guardar contraseña** | 8.3 |
| Botón **Solicitar un nuevo enlace** | 8.3 (verificado) |
| Enlace **Volver a iniciar sesión** | 8.3 |
| «Este enlace de recuperación está incompleto o ya fue usado. Solicita uno nuevo.» | 8.3 (verificado) |
| «Contraseña restablecida. Inicia sesión con tu nueva contraseña.» | 8.3 (verificado) |
| «El enlace no es válido o ya expiró. Solicita uno nuevo.» | 8.3 (verificado con un enlace ya usado) |
| «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | 8.3 (No verificado en la aplicación: no se provocó) |
| «No se pudo restablecer la contraseña. Inténtalo de nuevo.» | 8.3 (No verificado en la aplicación: no se provocó) |
| El correo (remitente, asunto, texto, caducidad) | 8.2 (verificado en la bandeja de pruebas) |

### 13.5 `/cambiar-clave` (19a §2)

| Elemento | Apartado |
|---|---|
| Título, migas «Inicio» › «Cambiar contraseña» y botón de volver | 7.1, 7.4 |
| Campo Contraseña actual (obligatorio) | 7.3 (verificado) |
| Campo Nueva contraseña (obligatorio, mínimo 8, distinta de la actual) | 7.3 (verificado) |
| Campo Confirmar nueva contraseña (obligatorio, coincide) y ojo | 7.3, 7.4 (verificado) |
| Botones **Cancelar** y **Guardar** | 7.4 |
| «Contraseña actualizada. Inicia sesión con tu nueva contraseña.» y cierre de sesión | 7.5 (verificado) |
| «La contraseña actual es incorrecta» | 7.6 (verificado) |
| «No se pudo cambiar la contraseña. Inténtalo de nuevo.» | 7.6 (No verificado en la aplicación) |
| Modo forzado: aviso «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.», sin **Cancelar** ni volver | 9.2 (No verificado en la aplicación: no hay cuenta marcada) |
| Acceso desde el menú de usuario | 4.4, 7 |

### 13.6 Cabecera, menú lateral y cierre de sesión (19a §4, §5)

| Elemento | Apartado |
|---|---|
| Botón de tres rayas (colapsar o abrir el menú) | 4.1, 4.2 (verificado en escritorio) |
| Fecha larga | 4.2 |
| Campana («Sin notificaciones») | 4.2 (verificado) |
| Bandera de idioma, «Español» y «English», cuadro «Cambiar idioma» con **Cancelar** y **Confirmar** | 4.3 (verificado) |
| El idioma no se guarda | 4.3 (No verificado en la aplicación: viene de 19a §4.5) |
| Imagen de perfil («Menú de usuario») y tarjeta (nombre, empresa) | 4.4 (verificado) |
| **Cambiar contraseña** y **Cerrar sesión** en el menú de usuario | 4.4, 5, 7 |
| «Versión: 1.0.0» | 4.4 (verificado) |
| Sin buscador ni selector de tema | 4.2 |
| Marca «⚕ SOFIA» y enlace al Inicio | 4.1 |
| 8 grupos y 24 opciones del menú (administrador) | 4.1 (verificado) |
| Reglas de visibilidad por permiso, aviso de permisos no cargados | 4.1 (reglas; el menú del cajero No verificado en la aplicación) |
| Opción resaltada, Ctrl + clic, vista angosta | 4.1 (Ctrl + clic y vista angosta: No verificado en la aplicación) |
| Opciones sin pantalla (Transferencias, Delivery, etc.) | 4.1 |
| Cierre de sesión: pasos, sin confirmación, destino «Iniciar sesión» | 5 (verificado) |
| Acceso sin sesión lleva a «Iniciar sesión» sin aviso | 5, 9.1 (verificado) |
| Aviso «No tienes permisos para acceder a esta pantalla.» | 4.1 (No verificado en la aplicación) |
| Sesión vencida y renovación en segundo plano | 9.1 (No verificado en la aplicación) |
| Cambio obligatorio de contraseña (redirección) | 9.2 (No verificado en la aplicación) |
| Inicio (dashboard), reconocimiento | 4.5 (detalle en el capítulo 10) |

## 14. Fuentes

Documentos consultados:

- `docs/_inventario/REGLAS-AGENTES.md` y `docs/_inventario/MANUAL-AGENTES.md`.
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md` (§1 a §5, §8).
- `docs/ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md` (secciones AUT y EMP: RN-AUT-001 a 039, RN-EMP-001 a 014).
- `sofia-web-spa/public/i18n/cambiar-clave/es.json` (texto de apoyo de la pantalla de cambio de contraseña).
- `sofia-web-spa/src/app/pages/empresa/empresa-register/empresa-register.component.html` (condiciones de los mensajes de la dirección).

Recorridos hechos en la aplicación local (`http://localhost:4200`, API `https://localhost:7300`, bandeja `http://localhost:8025`):

1. Registro: formulario vacío; RUC corto o con letra; correo mal escrito; usuario y contraseña cortos; dirección con solo espacios y de más de 255; nombre de empresa de más de 200; nombres, apellidos y correo por encima del máximo; ojo de la contraseña.
2. Registro con un RUC ya registrado (aviso de duplicado) y registro real de la empresa de prueba (entrada al Inicio).
3. Inicio: lectura de bloques; menú lateral completo; cabecera (colapsar menú, campana, idioma con **Cancelar** y **Confirmar** en ambos sentidos, menú de usuario).
4. Empresa, Sucursales y Roles & Permisos: lectura de lo que creó el registro.
5. Cerrar sesión; abrir `/dashboard` sin sesión.
6. Iniciar sesión: campos vacíos, un solo intento con contraseña equivocada, inicio correcto y ojo.
7. Cambiar contraseña: vacío; nueva corta con confirmación distinta; nueva igual a la actual; contraseña actual equivocada; cambio real y vuelta a «Iniciar sesión».
8. Recuperación: usuario vacío; pedido del enlace; lectura del correo en Mailpit; apertura del enlace desde el correo; errores de nueva contraseña; restablecimiento real; reuso del enlace ya usado; ruta sin código; inicio de sesión con la contraseña restablecida.

Estado en que queda la empresa de prueba: ver el archivo de estado compartido del manual.
