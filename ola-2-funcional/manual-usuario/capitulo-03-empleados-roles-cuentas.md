# Capítulo 3 — Tu equipo y sus accesos: empleados, roles y permisos, cuentas de usuario y sucursales

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-15 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo aprendes a dar de alta a tu equipo y a controlar qué puede hacer cada persona en SOFIA. Vas a:

- Registrar **empleados** (la persona, con su sucursal base) y buscarlos, exportarlos, editarlos y eliminarlos.
- Entender qué es un **rol** y qué es un **permiso**, y por qué tu empresa nace solo con el rol «Admin».
- Crear el rol **«Cajero»** y marcar, en la matriz de permisos, lo que necesita una persona de mostrador para usar el dashboard, la caja y el punto de venta.
- Crear la **cuenta de usuario** de un empleado (su usuario y su contraseña inicial), asignarle un rol y sucursales, y administrarla: activarla, desactivarla, forzar el cambio de contraseña y quitar un bloqueo.
- Vivir el **primer ingreso** de esa persona: el cambio obligatorio de contraseña y lo que ve (y no ve) un cajero.
- Usar la carga masiva y la exportación de roles.

Es para el **administrador** (rol «Admin»), con una parte que vive el **cajero**: la sección 8 cuenta su primer ingreso.

Un empleado y una cuenta son cosas distintas. El **empleado** es la persona (nombre, correo, sucursal base) y no puede entrar a SOFIA por sí solo. La **cuenta de usuario** es la llave de acceso (usuario y contraseña) de un empleado, y lleva un rol que define qué puede hacer.

Los ejemplos usan los datos ficticios del manual: la empresa «Botica Manual Demo S.A.C.», la administradora «Ana Demo Prueba» y dos empleados nuevos, «Carlos Cajero Demo» (sucursal base «Sede Principal») y «Nora Norte Demo» (sucursal base «Sucursal Demo Norte»). Los correos usan el dominio de prueba `sofia-manual.test`. Las contraseñas no se escriben en este manual: donde aparezca «tu contraseña» es la que tú definiste.

## 2. Antes de empezar

- Haz los capítulos 1 y 2: debes tener tu empresa registrada, la sesión del administrador iniciada y las dos sucursales «Sede Principal» y «Sucursal Demo Norte».
- Los nombres de los permisos son los que ves en la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción. Ojo: el módulo de roles y cuentas se llama «Roles y Cuentas de Usuario» en esa pantalla, y las sesiones de caja viven en el módulo «Sesiones de Caja».

| Tarea | Módulo | Acciones |
|---|---|---|
| Ver, buscar y exportar empleados | Empleados | `Leer` (además, para elegir la sucursal base: Sucursales `Leer`) |
| Crear un empleado | Empleados | `Crear` (además de `Leer`) |
| Editar un empleado | Empleados | `Actualizar` |
| Eliminar un empleado | Empleados | `Eliminar` |
| Ver, buscar y exportar roles | Roles y Cuentas de Usuario | `Leer` |
| Crear un rol y usar la carga masiva de roles | Roles y Cuentas de Usuario | `Crear` (además de `Leer`) |
| Editar la descripción o el nivel de un rol | Roles y Cuentas de Usuario | `Actualizar` |
| Marcar permisos y sucursales de un rol | Roles y Cuentas de Usuario | `Gestionar permisos` |
| Eliminar un rol | Roles y Cuentas de Usuario | `Eliminar` |
| Ver y buscar cuentas | Roles y Cuentas de Usuario | `Leer` |
| Crear una cuenta | Roles y Cuentas de Usuario | `Crear` (además de `Leer`; para elegir al empleado: Empleados `Leer`) |
| Activar, desactivar, forzar cambio de contraseña, resetear intentos, agregar o quitar sucursales de una cuenta | Roles y Cuentas de Usuario | `Actualizar` |
| Cambiar el rol de una cuenta | Roles y Cuentas de Usuario | `Asignar roles` |
| Eliminar una cuenta | Roles y Cuentas de Usuario | `Eliminar` |

- Los menús «Empleados», «Roles & Permisos» y «Cuentas de Usuario» están en el grupo **ADMINISTRACIÓN** del menú lateral.
- Los avisos de la aplicación (verdes de éxito, rojos de error) aparecen abajo al centro de la pantalla con un botón **Cerrar** y desaparecen solos a los pocos segundos. Los mensajes de este capítulo están copiados tal como se vieron en pantalla.
- Una regla de negocio de SOFIA que verás varias veces: **nadie puede cambiar sus propios roles ni su propia sucursal base**. Eso lo tiene que hacer otro administrador.

## 3. Registrar empleados

Un empleado es la ficha de una persona de tu equipo. Se registra primero el empleado y después, si esa persona va a usar SOFIA, se le crea una cuenta (sección 7).

**Cómo llegar:** menú lateral › ADMINISTRACIÓN › **Empleados** (`/empleados`).

### 3.1 La pantalla de búsqueda

La pantalla se llama «Empleados», con las migas «Inicio / Administración / Empleados». Tiene una tarjeta «Búsqueda de Empleados» («Ingresa los criterios de búsqueda»). En la esquina superior derecha de la tarjeta está el botón **Nuevo** (con un signo más), que solo ves si tu rol tiene el permiso `Crear` del módulo «Empleados». Debajo hay cinco filtros y, al pie, **Limpiar** y **Buscar**. Los resultados no aparecen al abrir la pantalla: haz clic en **Buscar**.

Al abrir la pantalla por primera vez y buscar sin filtros, aparece «Resultados (1)» con la administradora «Ana Demo Prueba».

### 3.2 Crear un empleado

1. Haz clic en **Nuevo** (`/empleados/nueva`).
2. Se abre el formulario con la flecha de volver, el título «Empleados», las migas, seis campos y los botones **Cancelar** y **Guardar**.
3. Escribe los datos (tabla siguiente).
4. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombres | Los nombres de pila. Ejemplo: `Carlos` | Sí | Vacío: «Este campo es obligatorio». El campo no deja escribir más de 75 caracteres |
| Apellido Paterno | Ejemplo: `Cajero` | Sí | Vacío: «Este campo es obligatorio». Máximo 75 caracteres |
| Apellido Materno | Ejemplo: `Demo` | No | Si lo dejas vacío, el empleado se guarda sin apellido materno (la columna aparece vacía en la tabla). Máximo 75 caracteres |
| Sucursal Base | La sucursal donde trabaja. Escribe al menos parte del nombre y elige una opción de la lista | Sí | Si no eliges ninguna: «Este campo es obligatorio». Con lo que escribas, el campo muestra hasta 20 sucursales cuyo nombre contenga el texto (en el ejemplo, escribir «Sed» muestra «Sede Principal»). Si escribes algo que no coincide con ninguna, la lista muestra una opción deshabilitada «Sin resultados» |
| Licencia Profesional | El número de colegiatura o licencia del profesional, si lo tiene (por ejemplo de un químico farmacéutico). Ejemplo: `QF-DEMO-0001` | No | Máximo 50 caracteres. No puede repetirse en otro empleado de tu empresa |
| Correo electrónico | El correo de la persona. Ejemplo: `carlos.cajero@sofia-manual.test` | No | Máximo 254 caracteres. Debajo del campo se lee «Opcional. Se usa para enviar el enlace de recuperación de contraseña.» Con un formato imposible (por ejemplo `carlos.cajero`): «Ingresa un correo electrónico válido.» (debajo del campo). No puede repetirse en otro empleado de tu empresa |

**Para qué sirve el correo.** SOFIA lo usa para enviar el enlace de **recuperación de contraseña** («¿Olvidaste tu contraseña?» en la pantalla de ingreso, capítulo 1). Si el empleado no tiene correo, no podrá recuperar su contraseña por sí mismo y tendrás que ayudarlo con **Forzar Cambio de Contraseña** (sección 7.5). El correo se guarda sin espacios sobrantes y en minúsculas.

**Antes de guardar, prueba los errores** (comprobado en la aplicación):

1. Con el formulario vacío, haz clic en **Guardar**: aparece «Este campo es obligatorio» bajo «Nombres», «Apellido Paterno» y «Sucursal Base», y no se envía nada.
2. Escribe `carlos.cajero@sofia` (sin punto después de la arroba) en el correo. La pantalla lo acepta, pero al hacer clic en **Guardar** el servidor lo rechaza y aparece el aviso rojo «Ingresa un correo electrónico válido.»; te quedas en el formulario con lo que escribiste. Escribe el correo completo, `carlos.cajero@sofia-manual.test`.
3. Si usas el correo de otra persona que ya está registrada (por ejemplo, intentar guardar a «Nora Norte Demo» con el correo de Carlos), aparece «Ya existe un empleado con ese correo electrónico en tu empresa.»

**Qué pasa al terminar bien:** aparece el aviso «Empleado guardado correctamente» y el sistema te lleva a la búsqueda de empleados (`/empleados`) con los filtros vacíos y **sin** tabla: haz clic en **Buscar** para ver la lista. **Cancelar** te devuelve a la búsqueda sin guardar nada.

Repite los pasos para el segundo empleado: Nombres `Nora`, Apellido Paterno `Norte`, Apellido Materno `Demo`, Sucursal Base «Sucursal Demo Norte», Licencia Profesional `QF-DEMO-0001`, correo `nora.norte@sofia-manual.test`. Quedan estos empleados en la empresa de prueba (haz clic en **Buscar**):

| Nombre Completo | Apellido Paterno | Apellido Materno | Licencia Profesional | Correo electrónico | Sucursal Base |
|---|---|---|---|---|---|
| Carlos Cajero Demo | Cajero | Demo | — | carlos.cajero@sofia-manual.test | Sede Principal |
| Ana Demo Prueba | Demo | Prueba | — | ana.demo@sofia-manual.test | Sede Principal |
| Nora Norte Demo | Norte | Demo | QF-DEMO-0001 | nora.norte@sofia-manual.test | Sucursal Demo Norte |

La lista sale ordenada por apellido paterno, luego apellido materno y luego nombres. «Nombre Completo» junta nombres, apellido paterno y apellido materno.

Errores que puedes ver al guardar un empleado:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Este campo es obligatorio» | Falta nombres, apellido paterno o sucursal base | Complétalo |
| «Ingresa un correo electrónico válido.» | El correo no tiene forma de correo | Corrígelo |
| «Ya existe un empleado con ese correo electrónico en tu empresa.» | Otro empleado de tu empresa ya usa ese correo (no distingue mayúsculas) | Usa otro correo |
| «Ya existe un empleado con esa licencia profesional.» | Otro empleado de tu empresa ya tiene esa licencia | Revisa el número |
| «Error al guardar el empleado» | Cualquier otro problema (por ejemplo, la sucursal ya no existe) | Revisa los datos e inténtalo de nuevo |

(El de la licencia repetida se comprueba en la sección 9.1. «Error al guardar el empleado» no verificado en la aplicación: no se pudo provocar; viene del inventario 19b §3.2.)

### 3.3 Buscar y exportar empleados

Los cinco filtros de la tarjeta «Búsqueda de Empleados» son:

| Filtro | Qué hace |
|---|---|
| Nombres | Texto con lupa. Busca empleados cuyos nombres contengan lo que escribas |
| Apellido Paterno | Texto con lupa. Igual, sobre el apellido paterno |
| Apellido Materno | Texto con lupa. Igual, sobre el apellido materno |
| Licencia Profesional | Texto con lupa. Igual, sobre la licencia |
| Sucursal Base | Texto con lupa y lista de ayuda con el texto «Buscar sucursal...». Busca empleados cuya sucursal base contenga el texto |

Los filtros se combinan (deben cumplirse todos), no distinguen mayúsculas y **Enter** dentro de cualquier campo también busca. Comprobado en la aplicación: «nor» en «Nombres» trae solo «Nora Norte Demo»; «qf-demo» en «Licencia Profesional» trae solo a Nora; «demo» en «Apellido Materno» trae a Carlos y a Nora (2 resultados). En «Sucursal Base» lo que cuenta es el **texto** del campo, lo hayas elegido de la lista o no: con `zzz` escrito, la búsqueda muestra «Resultados (0)» y la tabla dice «Sin resultados». **Limpiar** vacía los cinco filtros y oculta la tabla.

La tabla tiene estas columnas: «Nombre Completo» (es un enlace al detalle), «Apellido Paterno», «Apellido Materno» (vacía si no tiene), «Licencia Profesional» («—» si no tiene), «Correo electrónico» («—» si no tiene) y «Sucursal Base». Encima se lee «Resultados (N)». Abajo a la izquierda está **Exportar** y a la derecha el paginador («Items por página» 10, 25 o 50, «Página 1 de 1» y los botones de primera, anterior, siguiente y última página).

**Exportar** genera `empleados.xlsx` con los resultados de los filtros que tengas escritos. Comprobado en la aplicación: al hacer clic en **Exportar** el servidor respondió correctamente (código 200) y no apareció ningún aviso de error. (No verificado en la aplicación: el contenido del archivo ni su descarga, porque el navegador integrado no muestra los archivos descargados; las columnas «Nombres», «Apellido Paterno», «Apellido Materno», «Licencia Profesional», «Correo electrónico» y «Sucursal Base» vienen del inventario 19b §3.1.) Si falla la búsqueda o la exportación: «Error al buscar empleados».

### 3.4 Ver el detalle de un empleado

Haz clic en el nombre de un empleado en la tabla (`/empleados/…`). Se abre la ficha con el título «Empleados», «Datos del registro» y el botón **Editar** (solo con el permiso `Actualizar` del módulo «Empleados»). Los seis campos aparecen deshabilitados. Editar y eliminar se explican en la sección 9.

La ficha del empleado **no** muestra si tiene cuenta de usuario: eso se ve en la pantalla de cuentas.

## 4. Entender los roles y los permisos

### 4.1 Qué es un rol

Un **rol** es un perfil de trabajo («Cajero», «Gerente», «Químico farmacéutico»…). Cada rol tiene una lista de **permisos**: cosas concretas que puede hacer. Una cuenta de usuario recibe un rol y con eso hereda sus permisos. Cuando quieras cambiar lo que puede hacer toda la gente de un puesto, cambias el rol una sola vez en lugar de tocar cuenta por cuenta.

Un permiso es un par **módulo + acción**. El módulo es una parte de SOFIA («Ventas», «Pacientes», «Sucursales»…) y la acción es lo que se puede hacer en ella: `Leer` (ver y buscar), `Crear`, `Actualizar`, `Eliminar` y algunas acciones propias (por ejemplo «Anular» en Ventas o «Cerrar caja» en Sesiones de Caja). Regla general: **para ver una pantalla en el menú y entrar a ella se necesita `Leer` de su módulo**; para usar un botón de crear, editar o eliminar se necesita la acción correspondiente.

### 4.2 Tu empresa nace solo con el rol «Admin»

Al registrar tu empresa (capítulo 1), SOFIA crea **un solo rol: «Admin»**, con la descripción «Administrador de la empresa» y nivel de jerarquía 0, y se lo asigna a la persona que registró la empresa. **No existe ningún otro rol preconfigurado**: «Cajero», «Gerente» y cualquier otro los tienes que crear tú (sección 5).

El rol «Admin» tiene **acceso total** a todo lo de tu empresa y a todas sus sucursales. Se comprobó en la aplicación: si abres el rol «Admin» y entras a la pestaña «Permisos», **ninguna casilla está marcada** (todos los módulos dicen «0/N acciones»), y aun así el administrador puede hacer todo. Es normal: el acceso del «Admin» no depende de las casillas, depende de que el rol se llame «Admin». Por eso no hace falta (ni sirve) marcarle permisos. Además, solo un «Admin» puede asignar o quitar el rol «Admin» a otra cuenta, y solo un «Admin» puede modificar o eliminar la cuenta de otro administrador.

### 4.3 La lista de roles que ve tu empresa

**Cómo llegar:** menú lateral › ADMINISTRACIÓN › **Roles & Permisos** (`/roles`).

La pantalla se llama «Roles», con las migas «Inicio / Administración / Roles & Permisos». Tiene una tarjeta «Búsqueda de Roles» con los botones **Carga Masiva** (icono de subida) y **Nuevo** (signo más) arriba a la derecha (solo con el permiso `Crear` del módulo «Roles y Cuentas de Usuario»), tres filtros («Nombre del Rol», «Descripción» y «Nivel de Jerarquía», con un deslizador de 0 a 100) y los botones **Limpiar** y **Buscar**. Los filtros y la tabla se explican en la sección 10.

Haz clic en **Buscar** sin filtros. Con la empresa recién registrada se ve «Resultados (1)»:

| Nombre del Rol | Descripción | Nivel de Jerarquía |
|---|---|---|
| Admin | Administrador de la empresa | 0 |

Cuando creas roles nuevos aparecen en esta lista (en la sección 5 se crea «Cajero»). La lista se ordena por nivel de jerarquía, del más alto al más bajo.

### 4.4 La pantalla de un rol

Haz clic en el nombre del rol (`/roles/…`). Se abre la ficha con el título «Roles», «Datos del registro» y el botón **Editar** (solo con el permiso `Actualizar` del módulo «Roles y Cuentas de Usuario»). Arriba hay tres campos deshabilitados: «Nombre del Rol», «Nivel de Jerarquía» y «Descripción». Debajo hay una tarjeta **«Configuración del Rol»** con el texto «Define el menú, permisos y sucursales para este rol.» y un aviso amarillo con un icono de información:

> «Los cambios se aplican de inmediato. Los usuarios conectados verán su menú actualizado al recargar la página.»

La tarjeta tiene tres pestañas:

| Pestaña | Qué muestra |
|---|---|
| **Menú** | Una vista previa del menú lateral real que tendría una persona con este rol. Dice «Vista previa del menú real: una pantalla aparece cuando el rol tiene el permiso «Leer» de su módulo.» Cada grupo del menú (INICIO, ADMINISTRACIÓN, CATÁLOGOS, INVENTARIO, PUNTO DE VENTA, PACIENTES & ATENCIÓN, DIGEMID y SUNAT) lista sus pantallas con una marca verde (visible) o un círculo vacío (oculta). Solo lectura. Cambia en vivo cuando marcas casillas en la pestaña «Permisos», aunque todavía no hayas guardado |
| **Permisos** | La matriz de permisos: un panel por cada módulo (27 en total), explicada en la sección 5.2 |
| **Sucursales** | Una casilla por cada sucursal de tu empresa; las marcadas son las sucursales asignadas al rol (sección 5.5) |

## 5. Crear el rol «Cajero» y marcar sus permisos

El «Cajero» es la persona de mostrador: abre su caja, vende, cobra, anula y devuelve. En esta sección creas el rol y le marcas los permisos que necesita para que el dashboard, la caja y el punto de venta le funcionen completos.

Crear un rol y marcar sus permisos son **dos pasos**: primero se crea el rol (nombre, nivel y descripción) y después se vuelve a entrar a él con **Editar** para marcar permisos y sucursales. La tarjeta «Configuración del Rol» no existe en la pantalla de creación.

### 5.1 Crear el rol

**Cómo llegar:** **Roles & Permisos** › botón **Nuevo** (`/roles/nueva`). Necesitas el permiso `Crear` del módulo «Roles y Cuentas de Usuario».

La pantalla muestra la flecha de volver, el título «Roles», las migas, tres campos y los botones **Cancelar** y **Guardar** (sin **Eliminar**).

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombre del Rol | El nombre del puesto. Ejemplo: `Cajero` | Sí | Vacío: «Este campo es obligatorio». El campo deja escribir hasta 100 caracteres, pero el servidor solo acepta 50. **No se puede cambiar después de crear el rol** (en la ficha queda deshabilitado) y no se puede repetir en tu empresa |
| Nivel de Jerarquía | Un número entero, 1 o más. Ejemplo: `30` | Sí | Empieza en `0`, que **no** es válido en esta pantalla: al guardar con `0` aparece «Este campo es obligatorio» (el mismo mensaje que si estuviera vacío). No tiene máximo |
| Descripción | Una frase que explique el rol. Ejemplo: `Cajero de mostrador: atención al público y punto de venta` | No | Máximo 255 caracteres |

**Sobre el nivel de jerarquía:** es un número que sirve para ordenar la lista de roles (los de nivel más alto salen primero) y como referencia para ti (por ejemplo, 30 para cajeros y un número mayor para gerentes). **Hoy no limita nada**: un rol de nivel 5 y uno de nivel 90 pueden hacer exactamente lo mismo si tienen los mismos permisos.

**Antes de guardar, prueba los errores** (comprobado en la aplicación):

1. Con el formulario como se abre, haz clic en **Guardar**: aparece «Este campo es obligatorio» bajo «Nombre del Rol» y bajo «Nivel de Jerarquía».
2. Escribe `Admin` como nombre y `5` como nivel y haz clic en **Guardar**: aparece el aviso rojo «Error al guardar el rol» y te quedas en la pantalla. Esa es la forma en que SOFIA avisa que **el nombre ya existe** (no lo dice con más detalle).

**Guardar el rol «Cajero»:** escribe `Cajero`, nivel `30` y la descripción de la tabla, y haz clic en **Guardar**. Aparece el aviso «Rol guardado correctamente» y el sistema te lleva a la lista de roles (`/roles`, sin tabla hasta que hagas clic en **Buscar**). Ahora la lista muestra:

| Nombre del Rol | Descripción | Nivel de Jerarquía |
|---|---|---|
| Cajero | Cajero de mostrador: atención al público y punto de venta | 30 |
| Admin | Administrador de la empresa | 0 |

### 5.2 La matriz de permisos

**Cómo llegar:** **Roles & Permisos** › **Buscar** › clic en el nombre del rol (en el ejemplo, «Cajero») › **Editar** › pestaña **Permisos**. Para poder marcar casillas necesitas el permiso `Gestionar permisos` del módulo «Roles y Cuentas de Usuario»; sin él las casillas quedan deshabilitadas.

Al hacer clic en **Editar** se habilitan la descripción y el nivel (el nombre sigue bloqueado) y aparecen abajo **Cancelar**, **Guardar** y **Eliminar** (este último solo con el permiso `Eliminar`).

La pestaña **Permisos** es una lista de 27 paneles, uno por módulo, ordenados alfabéticamente por el nombre que ves en pantalla. Cada panel tiene:

- Una **casilla del módulo** en el encabezado con el nombre del módulo: marca o desmarca todas las acciones de golpe (se ve «a medias» si hay algunas marcadas).
- A la derecha, un contador **«N/M acciones»** (por ejemplo «3/5 acciones» en Recetas Médicas).
- Al abrir el panel, una casilla por acción, en varias columnas.

No hay una casilla para «marcar todo» en todos los módulos. **Marcar una casilla no guarda nada**: el cambio queda solo en la pantalla hasta que haces clic en **Guardar**, que guarda a la vez la descripción, el nivel, los permisos y las sucursales. **Cancelar** descarta todo y deja el rol como estaba.

Estos son los 27 módulos de la matriz (105 permisos en total) y sus acciones:

| Módulo (como se ve en pantalla) | Acciones |
|---|---|
| Aseguradoras | Actualizar, Crear, Eliminar, Leer |
| Auditoría de Seguridad | Anonimizar datos |
| Despachos Delivery | Actualizar, Eliminar, Leer, Programar delivery |
| Devoluciones | Leer, Procesar devolución |
| DIGEMID | Actualizar, Aislar lote en cuarentena, Crear, Eliminar, Generar acta de destrucción, Leer |
| Empleados | Actualizar, Crear, Eliminar, Leer |
| Empresa | Actualizar, Leer |
| Formulaciones Clínicas | Actualizar, Crear, Eliminar, Leer |
| Ingredientes Activos | Actualizar, Crear, Eliminar, Leer |
| Inventario | Actualizar, Crear, Eliminar, Leer |
| Jerarquías de Unidades | Actualizar, Crear, Eliminar, Leer |
| Laboratorios | Actualizar, Crear, Eliminar, Leer |
| Medicamentos | Actualizar, Crear, Eliminar, Leer |
| Órdenes Magistrales | Completar orden magistral, Iniciar orden magistral, Leer |
| Pacientes | Actualizar, Crear, Eliminar, Leer |
| Presentaciones de Venta | Actualizar, Crear, Eliminar, Leer |
| Profesionales de Salud | Actualizar, Crear, Eliminar, Leer |
| Proveedores | Actualizar, Crear, Eliminar, Leer, Registrar precio de proveedor |
| Recetas Médicas | Actualizar, Analizar con IA, Crear, Eliminar, Leer |
| Roles y Cuentas de Usuario | Actualizar, Asignar roles, Crear, Eliminar, Gestionar permisos, Leer |
| Series Fiscales | Actualizar, Crear, Eliminar, Leer |
| Servicios | Actualizar, Agendar servicio, Eliminar, Leer, Registrar inmunización |
| Sesiones de Caja | Aperturar caja, Cerrar caja, Leer |
| Sucursales | Actualizar, Crear, Eliminar, Leer |
| Transferencias | Actualizar, Crear, Leer |
| Unidades de Medida | Actualizar, Crear, Eliminar, Leer |
| Ventas | Actualizar, Anular, Crear, Leer |

Algunos módulos no tienen todas las acciones (por ejemplo, «Servicios» no tiene `Crear` y «Transferencias» no tiene `Eliminar`) porque SOFIA solo muestra las que existen hoy. Varios módulos de la lista (Despachos Delivery, Órdenes Magistrales, Servicios, Transferencias, Auditoría de Seguridad y parte de DIGEMID) todavía no tienen pantalla en el menú: sus permisos existen pero no abren ninguna pantalla en esta versión.

### 5.3 Permisos recomendados para un cajero

Esta lista sale de cruzar lo que necesita cada pantalla del cajero (dashboard, sesiones de caja, punto de venta) con las listas que esa pantalla consulta por detrás (por ejemplo, el punto de venta busca medicamentos, pacientes, recetas y aseguradoras). Si falta uno, esa parte de la pantalla falla o se queda vacía. Es una **recomendación**: la ajustas a cómo trabaja tu botica.

| Módulo › Acción | Para qué sirve |
|---|---|
| Ventas › Leer | Entrar a «Ventas» y a cada venta; ver el indicador de ventas del dashboard; exportar ventas |
| Ventas › Crear | Hacer una venta nueva en el punto de venta y cobrarla (también carga los lotes, presentaciones y la configuración de la venta) |
| Ventas › Actualizar | Retomar y cobrar una venta que quedó en espera |
| Ventas › Anular | Usar el botón «Anular Venta» en el detalle de una venta |
| Sesiones de Caja › Leer | Ver sus sesiones de caja; el punto de venta comprueba con esto si ya tienes una caja abierta; indicador de caja del dashboard |
| Sesiones de Caja › Aperturar caja | Abrir la caja (desde Sesiones de Caja, el punto de venta o el dashboard) |
| Sesiones de Caja › Cerrar caja | Cerrar su propia caja al terminar el turno |
| Devoluciones › Procesar devolución | Usar el botón «Devolver» en el detalle de una venta |
| Devoluciones › Leer | Mostrar el indicador de devoluciones del dashboard |
| Inventario › Leer | Mostrar el indicador de inventario del dashboard (además abre «Lotes» y «Stock por Sucursal» en el menú) |
| DIGEMID › Leer | Mostrar el indicador de cuarentena del dashboard (además abre «Catálogo DIGEMID» en el menú) |
| Medicamentos › Leer | Buscar productos en el punto de venta (además abre «Medicamentos» en el menú) |
| Pacientes › Leer | Elegir al cliente en el punto de venta y filtrar ventas por cliente (además abre «Pacientes» en el menú) |
| Aseguradoras › Leer | Elegir la aseguradora al cobrar con seguro (además abre «Aseguradoras» en el menú) |
| Recetas Médicas › Leer | Ver y vincular las recetas del cliente en el punto de venta (además abre «Recetas Médicas» en el menú) |
| Recetas Médicas › Analizar con IA | Usar la pestaña de escaneo de recetas del punto de venta |
| Recetas Médicas › Crear | Usar el enlace «Registrar receta» del punto de venta |
| Profesionales de Salud › Leer | Poder abrir la pantalla de registrar una receta (consulta la lista de profesionales) |
| Empleados › Leer | Filtrar la lista de ventas por empleado (además abre «Empleados» en el menú) |

Son 19 permisos. En el ejemplo se marcaron todos en el rol «Cajero». Fíjate en dos efectos:

- Como el menú muestra una pantalla cuando el rol tiene su `Leer`, el cajero verá también opciones como «Pacientes», «Medicamentos» o «Empleados» (solo para consultar, sin botones de crear, editar ni eliminar porque no tiene esas acciones). Si no quieres que las vea, tendrás que quitar ese `Leer`; a cambio, esa lista auxiliar fallará en el punto de venta. Es una decisión de tu negocio.
- Los permisos de «Ventas» (Crear, Actualizar, Anular) y «Devoluciones» (Procesar devolución) son sensibles: dan acceso al dinero. Dáselos solo a quien deba tenerlos.

El resultado se ve en la pestaña **Menú** antes de guardar: con estos 19 permisos, el menú del rol «Cajero» queda así:

| Grupo | Pantallas visibles | Pantallas ocultas |
|---|---|---|
| INICIO | Dashboard | — |
| ADMINISTRACIÓN | Empleados | Empresa, Sucursales, Roles & Permisos, Cuentas de Usuario |
| CATÁLOGOS | Medicamentos, Aseguradoras | Unidades de Medida, Jerarquías de Unidades, Laboratorios, Ingredientes Activos, Presentaciones de Venta, Formulaciones Clínicas, Proveedores |
| INVENTARIO | Lotes, Stock por Sucursal | — |
| PUNTO DE VENTA | Sesiones de Caja, Ventas | — |
| PACIENTES & ATENCIÓN | Pacientes, Profesionales de Salud, Recetas Médicas | — |
| DIGEMID | Catálogo DIGEMID | — |
| SUNAT | — | Series Fiscales |

### 5.4 Pasos para marcar los permisos

1. Abre el rol «Cajero» (**Roles & Permisos** › **Buscar** › clic en «Cajero») y haz clic en **Editar**.
2. Entra a la pestaña **Permisos**.
3. Abre el panel de cada módulo de la tabla 5.3 y marca sus casillas. Por ejemplo: en «Ventas» marca `Leer`, `Crear`, `Actualizar` y `Anular`; en «Sesiones de Caja» marca `Leer`, `Aperturar caja` y `Cerrar caja`.
4. Comprueba los contadores: «Ventas 4/4 acciones», «Sesiones de Caja 3/3 acciones», «Devoluciones 2/2 acciones», «Recetas Médicas 3/5 acciones», «DIGEMID 1/6 acciones» y «1/4 acciones» en Aseguradoras, Empleados, Inventario, Medicamentos, Pacientes y Profesionales de Salud.
5. Entra a la pestaña **Menú** para ver cómo quedaría el menú del cajero.
6. Entra a la pestaña **Sucursales** (sección 5.5) y marca las sucursales del rol.
7. Haz clic en **Guardar**.

**Qué pasa al terminar:** aparece el aviso «Rol guardado correctamente», el rol vuelve al modo ver y **te quedas en la misma ficha** (no te lleva a la lista). Se comprobó recargando la pantalla: los contadores y las sucursales quedaron guardados.

### 5.5 Sucursales del rol

La pestaña **Sucursales** muestra una casilla por cada sucursal de tu empresa (en el ejemplo, «Sede Principal» y «Sucursal Demo Norte»). Si no hubiera sucursales, diría «No hay sucursales disponibles». Las sucursales marcadas son sucursales en las que podrán operar **todas las cuentas que tengan este rol**, además de la sucursal base del empleado de cada cuenta y de las que le agregues a una cuenta en particular (sección 7.7).

En el ejemplo se marcó «Sede Principal». Como la sucursal base de Carlos ya es la «Sede Principal», en la práctica no cambia nada para él; marcar una sucursal en el rol sirve cuando quieres que todos los cajeros puedan operar también en ella. **Guardar** guarda siempre el conjunto completo de casillas: si las desmarcas todas, el rol queda sin sucursales.

Ten presente que el punto de venta y las ventas se trabajan siempre en la **sucursal base** de la persona; las sucursales adicionales se usan para consultar, por ejemplo, las sesiones de caja y el inventario de otras sucursales (ver capítulos siguientes).

## 6. Qué pasa si quitas un permiso

Quitar un permiso a un rol es igual de simple que ponerlo: entra al rol, haz clic en **Editar**, desmarca la casilla en la pestaña **Permisos** y haz clic en **Guardar**. El aviso es el mismo, «Rol guardado correctamente».

Comprobado en la aplicación con el rol «Cajero» y el permiso `Profesionales de Salud › Leer`:

1. Al desmarcar la casilla, el contador del módulo pasa de «1/4 acciones» a «0/4 acciones» y, en la pestaña **Menú**, «Profesionales de Salud» cambia de la marca verde al círculo vacío **antes de guardar**.
2. Si haces clic en **Cancelar**, todo vuelve a como estaba (el contador vuelve a «1/4 acciones»). Nada se había guardado.
3. Si guardas, la siguiente vez que Carlos (el cajero) inicia sesión, la opción «Profesionales de Salud» ya no aparece en su menú, y si escribe la dirección `/profesionales-salud` en el navegador, el sistema lo devuelve al Inicio con el aviso «No tienes permisos para acceder a esta pantalla.»

Consecuencias que debes tener presentes:

- **El permiso quitado deja de funcionar en todas las cuentas con ese rol**, no solo en una.
- **Una persona que ya tiene la sesión abierta no ve el cambio al instante**: la pantalla del rol lo avisa («Los usuarios conectados verán su menú actualizado al recargar la página.»). Si necesitas que el cambio se aplique ya, pídele que cierre sesión y vuelva a entrar. (No verificado en la aplicación el comportamiento exacto de una sesión abierta al mismo tiempo: el navegador integrado solo mantiene una sesión a la vez. Lo comprobado es lo que ve la persona al volver a entrar.)
- Si quitas un permiso que otra pantalla usa por dentro, esa pantalla puede fallar en silencio. Por ejemplo, sin `Pacientes › Leer` el punto de venta no puede buscar al cliente. Por eso conviene quitar permisos con cuidado y revisar la tabla 5.3.
- Si desmarcas **todas** las casillas y guardas, el rol queda sin permisos: las cuentas con ese rol solo ven el Dashboard, sin tarjetas de indicadores (No verificado en la aplicación: viene de la matriz 05b §5, ruta `/dashboard`).
- El permiso `Gestionar permisos` es poderoso: quien lo tiene puede cambiar los permisos de cualquier rol. Además, SOFIA impide que una persona **amplíe** los permisos del rol que ella misma tiene asignado: en ese caso aparece «No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado». (No verificado en la aplicación: la administradora tiene el rol «Admin», que está exento; viene del inventario 19b §4.2 y de la regla RN-ROL-010.)

Para el ejemplo del manual, el permiso `Profesionales de Salud › Leer` se volvió a marcar y guardar al terminar la prueba: el rol «Cajero» quedó con sus 19 permisos.

## 7. Crear la cuenta del cajero

Un empleado entra a SOFIA con una **cuenta de usuario**. La cuenta tiene un usuario, una contraseña, un rol y, si hace falta, sucursales adicionales.

**Cómo llegar:** menú lateral › ADMINISTRACIÓN › **Cuentas de Usuario** (`/cuentas`).

### 7.1 La pantalla de búsqueda de cuentas

La pantalla se llama «Cuentas de Usuario», con las migas «Inicio / Administración / Cuentas de Usuario». Tiene una tarjeta «Búsqueda de Cuentas» («Ingresa los criterios de búsqueda») con el botón **Nuevo** arriba a la derecha (solo con el permiso `Crear` del módulo «Roles y Cuentas de Usuario»), cuatro filtros y los botones **Limpiar** y **Buscar**. No existe **Exportar** ni carga masiva de cuentas.

| Filtro | Qué hace |
|---|---|
| Nombre de Usuario | Texto con lupa. Busca usuarios que contengan lo que escribas, sin distinguir mayúsculas (`CARLOS` encuentra `carlos.cajero`) |
| Empleado | Texto con lupa. Busca por el nombre del empleado (`ana` encuentra a «Ana Demo Prueba») |
| Cuenta Activa | Lista con «—» (sin filtro), «Activa» e «Inactiva» |
| Bloqueado | Lista con «—» (sin filtro), «Sí» y «No» |

Los filtros se combinan y **Enter** en los campos de texto también busca. **Limpiar** vacía los filtros y oculta la tabla. Los resultados aparecen al hacer clic en **Buscar**.

La tabla tiene estas columnas, y dos de ellas muestran solo un icono (sin texto):

| Columna | Qué muestra |
|---|---|
| Usuario | El nombre de usuario; es un enlace a la ficha |
| Empleado | El nombre completo del empleado |
| Estado | Una marca verde de aprobado si la cuenta está activa; un círculo con una equis si está inactiva |
| Bloqueado | Un candado abierto si la cuenta no está bloqueada; un candado cerrado si lo está |
| Intentos | El número de intentos fallidos de ingreso |

Encima de la tabla se lee «Resultados (N)». Debajo hay el paginador («Items por página» 10, 25 o 50, «Página 1 de 1» y los botones de primera, anterior, siguiente y última página). Con la empresa recién creada, la búsqueda sin filtros muestra una fila: `ana.manual` · Ana Demo Prueba · activa · candado abierto · 0.

Comprobado: el filtro «Bloqueado» = «Sí» trae solo las cuentas bloqueadas (se vio con una cuenta de prueba, sección 7.6); «Cuenta Activa» = «Inactiva» sin cuentas inactivas muestra «Resultados (0)». Si falla la búsqueda: «Error al buscar cuentas».

### 7.2 Crear la cuenta

1. Haz clic en **Nuevo** (`/cuentas/nueva`). Necesitas los permisos `Crear` (módulo «Roles y Cuentas de Usuario») y, para elegir al empleado, `Leer` de «Empleados».
2. Se abre el formulario con la flecha de volver, el título «Cuentas de Usuario», las migas, tres campos y los botones **Cancelar** y **Guardar**.
3. Elige el empleado, escribe el usuario y la contraseña inicial.
4. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Empleado | Escribe al menos 2 letras del nombre y elige a la persona de la lista (en el ejemplo, «Ca» muestra «Carlos Cajero Demo») | Sí | Si no eliges a nadie: «Este campo es obligatorio». Si lo que escribes no coincide con ninguna opción, el campo se vacía al salir. **La lista también muestra empleados que ya tienen cuenta**: si eliges uno, al guardar aparece «Error al guardar la cuenta» |
| Nombre de Usuario | El usuario con el que entrará. Ejemplo: `carlos.cajero` | Sí | Vacío: «Este campo es obligatorio». Máximo 50 caracteres. Con más de 50 aparece «La longitud máxima es de 100 caracteres» (el mensaje dice 100 pero el límite real es 50). Debe ser **único en todo SOFIA**, no solo en tu empresa: si ya existe, aparece «Error al guardar la cuenta» |
| Contraseña | La contraseña inicial que le darás a la persona (la escribes tú). Mínimo 8 y máximo 128 caracteres | Sí | Vacía: «Este campo es obligatorio». **Con menos de 8 caracteres la pantalla no muestra ningún mensaje**: simplemente no guarda. El campo no tiene botón para mostrar el texto |

**Antes de guardar, prueba los errores** (comprobado en la aplicación):

1. Con el formulario vacío haz clic en **Guardar**: aparece «Este campo es obligatorio» bajo los tres campos.
2. Escribe un usuario de más de 50 caracteres: aparece «La longitud máxima es de 100 caracteres».
3. Escribe una contraseña de 7 caracteres y un usuario válido: no aparece ningún mensaje y no se guarda. Usa 8 o más caracteres.
4. Escribe un usuario que ya existe (por ejemplo `ana.manual`): aparece el aviso rojo «Error al guardar la cuenta».
5. Elige un empleado que ya tiene cuenta: aparece «Error al guardar la cuenta».

**Qué contraseña inicial usar.** Genera una contraseña de al menos 12 caracteres con mayúsculas, minúsculas, números y un símbolo, y entrégasela a la persona por un medio privado. SOFIA no la vuelve a mostrar y la persona **tendrá que cambiarla** en su primer ingreso (sección 8). Para el cajero del ejemplo se usó el usuario `carlos.cajero` con una contraseña generada para esta prueba (no se escribe en el manual).

**Qué pasa al terminar bien:** aparece el aviso «Cuenta guardada correctamente» y SOFIA abre la ficha de la cuenta recién creada (`/cuentas/…`) en modo ver. La cuenta nace **activa**, con 0 intentos fallidos, «Requiere Cambio de Clave: Sí», sin rol («Sin roles asignados») y sin sucursales adicionales («Sin sucursales adicionales asignadas»). **El formulario de creación no pide rol ni sucursales: se asignan después, en la ficha** (secciones 7.3 y 7.7). Una cuenta sin rol no puede hacer nada útil.

### 7.3 Asignar el rol «Cajero»

1. En la ficha, haz clic en **Editar**. Necesitas el permiso `Actualizar`; para cambiar el rol, además `Asignar roles` del módulo «Roles y Cuentas de Usuario» (sin él, la tarjeta «Roles Asignados» sigue siendo solo texto).
2. En la tarjeta **«Roles Asignados»** («Gestiona los roles de acceso de esta cuenta») haz clic en el campo «Agregar rol...». Sin escribir nada la lista muestra todos los roles de tu empresa (en el ejemplo, «Cajero» y «Admin»).
3. Elige «Cajero».
4. Haz clic en **Guardar**.

Aparece «Cuenta guardada correctamente» y la ficha vuelve al modo ver mostrando «Roles Asignados: Cajero». Te quedas en la ficha.

Detalles que debes conocer:

- **Una cuenta tiene un solo rol en esta pantalla.** Si la cuenta ya tiene uno y eliges otro, SOFIA quita el anterior y asigna el nuevo.
- **Cuando la cuenta ya tiene un rol, el campo lo muestra escrito, y la lista solo ofrece los roles que coinciden con ese texto.** Para cambiar de rol, borra el texto del campo y escribe las primeras letras del rol nuevo (comprobado: con «Rol Prueba Eliminar» escrito, la lista no ofrecía «Cajero»; al escribir «Caj» apareció).
- **No puedes dejar una cuenta sin rol desde esta pantalla**: vaciar el campo no quita el rol.
- Cambiar el rol de una cuenta cierra su sesión actual; los permisos nuevos se aplican la próxima vez que entre.
- Nadie puede cambiar **sus propios** roles: si intentas cambiar el rol de tu propia cuenta aparece «Una cuenta no puede modificar sus propios roles». Solo un «Admin» puede asignar o quitar el rol «Admin» («Solo un administrador puede asignar o quitar el rol Admin»). (No verificado en la aplicación: no se probó con la cuenta propia para no arriesgar el acceso del administrador; los textos vienen del inventario 19b §5.2.)

### 7.4 La ficha de una cuenta

La ficha en modo ver (`/cuentas/…`) tiene el título «Cuentas de Usuario», «Datos del registro», el botón **Editar** y tres tarjetas. En la primera ves, todos deshabilitados:

| Dato | Qué muestra |
|---|---|
| Nombre de Usuario | El usuario (por ejemplo `carlos.cajero`) |
| Empleado | El nombre completo del empleado |
| Intentos Fallidos | Cuántas veces se equivocó de contraseña seguidas |
| Cuenta Activa | «Activa» o «Inactiva» |
| Bloqueado | «Sí» o «No» |
| Requiere Cambio de Clave | «Sí» o «No» |
| Bloqueado Hasta | Fecha y hora hasta la que sigue bloqueada (por ejemplo `14/10/2026 23:16`). Solo aparece si la cuenta tiene una fecha de bloqueo |
| Creado el | La fecha de creación (`14/10/2026`) |

Las otras dos tarjetas son «Roles Asignados» y «Sucursales Asignadas» (sección 7.7).

### 7.5 Activar, desactivar, forzar el cambio de contraseña y resetear intentos

Al hacer clic en **Editar** la ficha muestra «Nombre de Usuario» y «Empleado» deshabilitados (**no se pueden cambiar**) y estos tres controles:

| Control | Qué hace |
|---|---|
| **Cuenta Activa** (interruptor) | Empieza con el valor actual. Apagado, la cuenta queda «Inactiva»: la persona **no puede iniciar sesión** ni renovar su sesión; el resto de su ficha no se borra. Sirve para suspender un acceso sin eliminar la cuenta (por ejemplo, vacaciones largas o una investigación). Se vuelve a activar con el mismo interruptor |
| **Forzar Cambio de Contraseña** (interruptor) | Empieza siempre apagado. Si lo enciendes y guardas, la ficha muestra «Requiere Cambio de Clave: Sí» y en su próximo ingreso la persona tendrá que cambiar su contraseña (sección 8). Úsalo, por ejemplo, cuando sospeches que otras personas conocen la contraseña. No hay forma de apagarlo a mano: se apaga solo cuando la persona cambia su contraseña |
| **Resetear Intentos Fallidos** (casilla) | Empieza siempre desmarcada. Si la marcas y guardas, los intentos fallidos vuelven a 0 y **se quita el bloqueo**. Es la única forma de desbloquear una cuenta antes de que pasen los 15 minutos |

Los tres se aplican con **Guardar**; aparece «Cuenta guardada correctamente» y la ficha vuelve al modo ver, en la misma pantalla. **Cancelar** descarta lo que cambiaste.

**No existe una acción «Restablecer contraseña» ni «Bloquear» en esta pantalla.** Si una persona olvida su contraseña, el camino es la recuperación por correo desde la pantalla de ingreso (requiere que su empleado tenga correo, sección 3.2). Desde esta pantalla el administrador no puede escribirle una contraseña nueva: su única palanca es **Forzar Cambio de Contraseña**, que sirve mientras la persona todavía conozca su contraseña actual. Si no la recuerda y su empleado no tiene correo, la salida que ofrece hoy la aplicación es eliminar la cuenta y crearla de nuevo (según la regla RN-CTA-013, el usuario queda libre al eliminar la cuenta). (No verificado en la aplicación: la recuperación por correo se vio en el capítulo 1; no se probó recrear una cuenta con el mismo usuario.)

Comprobado en la aplicación con una cuenta de prueba:

- Con «Cuenta Activa» apagado y **Guardar**: la ficha muestra «Cuenta Activa: Inactiva».
- «Forzar Cambio de Contraseña» encendido y **Guardar**: «Requiere Cambio de Clave: Sí».
- Casilla «Resetear Intentos Fallidos» (ver sección 7.6): «Intentos Fallidos: 0» y «Bloqueado: No».

### 7.6 El bloqueo automático

Si alguien se equivoca de contraseña **5 veces seguidas** al iniciar sesión, la cuenta se **bloquea 15 minutos**. Comprobado con una cuenta de prueba (no con la del cajero): después de 5 intentos fallidos, SOFIA mostró en la pantalla de ingreso, cada vez, el mensaje «Credenciales inválidas. Verifica tu usuario y contraseña.» (el mismo mensaje para usuario inexistente, contraseña mal, cuenta desactivada y cuenta bloqueada, para no dar pistas). **Ya bloqueada, ni la contraseña correcta deja entrar**: se vio el mismo mensaje.

En la lista de cuentas, la cuenta bloqueada se ve con el candado cerrado y «Intentos» en 5, y el filtro «Bloqueado = Sí» la encuentra. En su ficha: «Intentos Fallidos: 5», «Bloqueado: Sí» y «Bloqueado Hasta: 14/10/2026 23:16» (15 minutos después del último intento). Para desbloquearla: **Editar** › marca **Resetear Intentos Fallidos** › **Guardar**. Después del reseteo, «Bloqueado Hasta» desaparece de la ficha.

### 7.7 Agregar y quitar sucursales de una cuenta

Una cuenta puede operar en la sucursal base de su empleado y, además, en las sucursales que le agregues aquí o que estén asignadas a su rol (sección 5.5). La tarjeta **«Sucursales Asignadas»** («Sucursales adicionales donde esta cuenta puede operar, además de la sucursal base de su empleado») lista las adicionales; si no hay, dice «Sin sucursales adicionales asignadas».

1. En la ficha, haz clic en **Editar**.
2. En «Agregar sucursal...», haz clic y elige una sucursal de la lista (muestra todas las de tu empresa, también la sucursal base).
3. **Se aplica al instante**, sin **Guardar** y sin confirmación: aparece «Sucursal asignada correctamente» y la sucursal aparece en la lista.
4. Para quitar una, haz clic en el icono de papelera de su fila (se llama «Quitar sucursal»). También es inmediato: aparece «Sucursal eliminada correctamente» y la fila desaparece. (El mensaje dice «eliminada» pero solo la quita de esta cuenta; la sucursal sigue existiendo.)

Comprobado: se agregó «Sucursal Demo Norte» a Carlos y se quitó enseguida; el cajero del ejemplo quedó **sin** sucursales adicionales (solo su sucursal base, «Sede Principal»). **Cancelar** no deshace los cambios de sucursales ya aplicados. Necesitas el permiso `Actualizar` del módulo «Roles y Cuentas de Usuario» y `Leer` de «Sucursales» para ver la lista.

Errores posibles: «Error al asignar la sucursal», «Error al eliminar la sucursal», «La sucursal no existe» y «La cuenta no existe» (No verificado en la aplicación: no se pudieron provocar; vienen del inventario 19b §5.2).

## 8. Primer ingreso del cajero

Esta sección la vive el **cajero**. Cierra la sesión del administrador (menú de usuario, arriba a la derecha › **CERRAR SESIÓN**) y entra con el usuario y la contraseña inicial que te dio el administrador.

### 8.1 El cambio obligatorio de contraseña

Al iniciar sesión por primera vez, SOFIA no te lleva al Dashboard: te lleva a la pantalla **«Cambiar contraseña»** (`/cambiar-clave`). Se ve así:

- El menú lateral de tu rol está a la izquierda, pero **no sirve para navegar** mientras no cambies la contraseña: cada opción te devuelve a esta pantalla sin mostrar ningún aviso. Si escribes a mano otra dirección en el navegador (por ejemplo `/pos`), también te devuelve aquí.
- Arriba, el título «Cambiar contraseña», las migas «Inicio / Cambiar contraseña» y el texto «Al guardar se cerrarán todas tus sesiones y tendrás que iniciar sesión con tu nueva contraseña».
- Un recuadro con un icono de candado y el mensaje «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.»
- Tres campos y un solo botón, **Guardar**. En este modo **no** hay botón **Cancelar** ni flecha de volver. Un icono de ojo muestra u oculta lo que escribes en los tres campos.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Contraseña actual | La contraseña inicial que te dio el administrador | Sí | Vacío: «Este campo es obligatorio». Si no es la correcta, aparece el aviso rojo «La contraseña actual es incorrecta» |
| Nueva contraseña | La que elijas, de 8 caracteres o más (usa mayúsculas, minúsculas, números y un símbolo) | Sí | Vacío: «Este campo es obligatorio». Menos de 8 caracteres: «Debe tener al menos 8 caracteres». Igual a la actual: «La nueva contraseña debe ser distinta de la actual» |
| Confirmar nueva contraseña | Repite la nueva | Sí | Vacío: «Este campo es obligatorio». Distinta de la nueva: «Las contraseñas no coinciden» |

Los mensajes de los campos aparecen debajo de cada uno (a veces tardan cerca de un segundo en dibujarse). Un error del servidor que no sea el de la contraseña actual muestra «No se pudo cambiar la contraseña. Inténtalo de nuevo.» (No verificado en la aplicación: no se pudo provocar; viene de 19a §2.)

**Qué pasa al terminar bien:** aparece el aviso «Contraseña actualizada. Inicia sesión con tu nueva contraseña.», SOFIA **cierra todas tus sesiones** y te lleva a la pantalla de ingreso. Entra con tu usuario y la contraseña nueva. Ya no te pedirá cambiarla otra vez (en la ficha de la cuenta, «Requiere Cambio de Clave» pasa a «No»).

**Nota para el administrador:** equivocarte en la «Contraseña actual» de esta pantalla no suma intentos de bloqueo: el bloqueo de 15 minutos es solo por intentos fallidos al iniciar sesión.

El cambio de contraseña también está disponible cuando quieras, sin que nadie te lo obligue: menú de usuario (arriba a la derecha, con tu nombre y el de la empresa) › **Cambiar contraseña**. En ese caso la pantalla trae además el botón **Cancelar** y la flecha de volver.

### 8.2 Qué ve el cajero al entrar

**El menú lateral** muestra solo lo que el rol «Cajero» permite (el mismo que mostró la vista previa del rol, sección 5.3):

| Grupo | Opciones que ve |
|---|---|
| INICIO | Dashboard |
| ADMINISTRACIÓN | Empleados |
| CATÁLOGOS | Medicamentos, Aseguradoras |
| INVENTARIO | Lotes, Stock por Sucursal |
| PUNTO DE VENTA | Sesiones de Caja, Ventas |
| PACIENTES & ATENCIÓN | Pacientes, Profesionales de Salud, Recetas Médicas |
| DIGEMID | Catálogo DIGEMID |

**Opciones ocultas:** del grupo ADMINISTRACIÓN no ve «Empresa», «Sucursales», «Roles & Permisos» ni «Cuentas de Usuario»; del grupo CATÁLOGOS no ve «Unidades de Medida», «Jerarquías de Unidades», «Laboratorios», «Ingredientes Activos», «Presentaciones de Venta», «Formulaciones Clínicas» ni «Proveedores»; y el grupo SUNAT (Series Fiscales) no aparece.

**Si intenta abrir por la dirección una pantalla sin permiso**, por ejemplo `/roles`, el sistema lo devuelve al Dashboard y muestra el aviso rojo «No tienes permisos para acceder a esta pantalla.» Comprobado también con `/empleados/nueva` (el cajero puede ver empleados pero no crearlos).

**El Dashboard** saluda «Hola, Carlos Cajero Demo», con el texto «Este es el resumen de hoy.», la hora de la última actualización y el botón **Actualizar**. Como el rol «Cajero» tiene `Leer` en Ventas, Sesiones de Caja, Inventario, DIGEMID y Devoluciones, ve todas las tarjetas: «Ventas de hoy» (vendido hoy, comprobantes, «Mis ventas de hoy», «Ventas en espera» y el cobro por método de pago: Efectivo, Yape / Plin, Tarjeta y Transferencia), «Caja» (con la caja aún cerrada dice «No tienes una caja abierta» y ofrece el botón **Abrir caja**), «Inventario» (lotes vencidos y por vencer, productos sin stock o con stock bajo y lotes en cuarentena), «Ventas de los últimos 7 días», «Productos más vendidos en 7 días» y «Devoluciones de hoy». Todo en cero porque la empresa de prueba todavía no vendió nada. Los capítulos de caja y ventas explican estas tarjetas.

**Las pantallas que sí abre:**

| Pantalla | Qué ve y qué botones tiene |
|---|---|
| Empleados | La lista con los tres empleados y sus correos (no tiene **Nuevo**, ni **Editar**, ni **Eliminar** en la ficha: son solo de consulta) |
| Ventas (`/pos`) | Tiene el botón **Nueva Venta POS** y los filtros Empleado, Cliente, Fecha desde, Fecha hasta y Estado |
| Sesiones de Caja | Tiene el botón **Nuevo** (abrir caja) y la tabla Sucursal / Cajero / Apertura / Cierre / Monto Apertura / Estado, sin filas todavía |
| Punto de venta (`/pos/nueva`) | Mientras no tenga caja abierta, dice «Debes aperturar la caja del turno antes de procesar una venta.» y ofrece **Aperturar Caja** |

Se revisó que estas pantallas cargan sin errores de permisos (todas las consultas de la caja, el punto de venta y el dashboard respondieron correctamente). La única consulta rechazada fue la lista de sucursales al abrir la ficha de un empleado, que el cajero no necesita (no tiene `Sucursales › Leer`) y que no muestra ningún aviso.

## 9. Editar, desactivar y eliminar

Para probar estas acciones sin tocar a tu equipo real se crearon, solo para esto, un empleado de prueba («Pedro Prueba») con su cuenta de usuario y un rol de prueba («Rol Prueba Eliminar»). Qué quedó de ellos al terminar está en la sección 9.4.

### 9.1 Editar y eliminar un empleado

**Editar.**

1. **Empleados** › **Buscar** › clic en el nombre › **Editar** (permiso `Actualizar` del módulo «Empleados»).
2. Los seis campos se habilitan y abajo aparecen **Cancelar**, **Guardar** y **Eliminar** (este último solo con el permiso `Eliminar`). Las reglas de los campos son las de la sección 3.2.
3. Cambia lo que necesites y haz clic en **Guardar**. Aparece «Empleado guardado correctamente» y el sistema te lleva a la **lista** de empleados (a diferencia de Sucursales o Roles, no te quedas en la ficha).

Comprobado en la aplicación: se agregó el apellido materno «Borrar» a «Pedro Prueba» y se guardó. Registrar un empleado con la licencia de otro (`QF-DEMO-0001`, que ya tiene Nora) mostró «Ya existe un empleado con esa licencia profesional.»

Dos reglas que no verás en la pantalla: **nadie puede cambiar su propia sucursal base** (si lo intentas con tu propio empleado aparece solo «Error al guardar el empleado»), y cuando le cambias la sucursal base a un empleado con cuenta, SOFIA le cierra la sesión actual. (No verificado en la aplicación: vienen de las reglas RN-EPL-011 y RN-EPL-012.)

**Eliminar.**

1. En la ficha, haz clic en **Editar** y luego en **Eliminar** (abajo a la derecha, después de **Guardar**).
2. Se abre un cuadro con el título «Confirmar eliminación», el mensaje «¿Estás seguro de que deseas eliminar este empleado?» y los botones **Cancelar** y **Confirmar**.
3. Haz clic en **Confirmar**. Aparece «Empleado eliminado correctamente» y vuelves a la lista, donde ya no aparece.

**Qué impide eliminar a un empleado.**

| Situación | Resultado | Qué hacer |
|---|---|---|
| El empleado es **gerente de una sucursal** | Aparece el aviso rojo «No se puede eliminar un empleado que gerencia una sucursal. Asigna otro gerente primero.» y el empleado no se borra. Comprobado: se puso a «Pedro Prueba» como gerente de «Sucursal Demo Norte» y se intentó eliminarlo | Ve a **Sucursales**, abre la sucursal, **Editar**, cambia o borra el «Gerente» y **Guardar**. Luego elimina al empleado |
| El empleado **tiene una cuenta de usuario** | **No lo impide.** Comprobado: se eliminó a «Pedro Prueba» teniendo cuenta y apareció «Empleado eliminado correctamente» | **Elimina primero la cuenta** (sección 9.2). Ver el aviso siguiente |

**Aviso importante.** Si eliminas un empleado que todavía tiene cuenta, la cuenta **desaparece de la lista de cuentas** y ya no puedes abrirla desde la pantalla (si abres su dirección a mano aparece el aviso «cuentas.detail.load-error», un texto sin traducir), pero **sigue existiendo en el sistema**: su nombre de usuario queda ocupado (comprobado: al crear otra cuenta con el usuario `pedro.prueba` apareció «Error al guardar la cuenta») y tú no tienes forma de eliminarla ni de desbloquearla desde la pantalla. Por eso, el orden correcto para dar de baja a una persona es: **primero eliminar su cuenta, después su empleado**.

### 9.2 Editar, desactivar y eliminar una cuenta

**Editar y desactivar.** Se hacen desde la ficha de la cuenta con **Editar** (sección 7.5): activar o desactivar, forzar el cambio de contraseña, resetear intentos, cambiar el rol (7.3) y las sucursales (7.7). Comprobado con la cuenta de prueba: se desactivó («Cuenta Activa: Inactiva»), se forzó el cambio de contraseña y se reseteó un bloqueo.

Una cuenta **desactivada** no puede entrar: en la pantalla de ingreso recibiría el mismo mensaje genérico «Credenciales inválidas. Verifica tu usuario y contraseña.» (No verificado en la aplicación el intento de ingreso con una cuenta desactivada; viene de la regla RN-CTA-012.) En la lista de cuentas se ve con el círculo de la equis en «Estado».

**Eliminar.**

1. En la ficha de la cuenta, haz clic en **Editar** y luego en **Eliminar**.
2. Se abre un cuadro con el título «Eliminar cuenta» y el mensaje «¿Estás seguro de que deseas eliminar esta cuenta de usuario? Esta acción no se puede deshacer.», con los botones **Cancelar** (cierra el cuadro y no borra nada, comprobado) y **Confirmar**.
3. Haz clic en **Confirmar**. Aparece «Cuenta eliminada correctamente» y vuelves a la lista de cuentas, donde ya no aparece.

Eliminar la cuenta **no** elimina al empleado: la persona sigue en la lista de empleados y se le puede crear otra cuenta. Necesitas el permiso `Eliminar` del módulo «Roles y Cuentas de Usuario». Solo un «Admin» puede modificar o eliminar la cuenta de otro administrador («Solo un administrador puede modificar o eliminar la cuenta de un administrador»; No verificado en la aplicación, viene del inventario 19b §5.2). Otros mensajes posibles: «Error al eliminar la cuenta», «Error al cargar la cuenta» y «La cuenta no existe» (No verificado en la aplicación).

### 9.3 Editar y eliminar un rol

**Editar.** En un rol solo se pueden cambiar la **descripción** y el **nivel de jerarquía** (el nombre queda bloqueado), además de los permisos y las sucursales (secciones 5.4 y 5.5). Todo se guarda junto con **Guardar**, con el aviso «Rol guardado correctamente», y te quedas en la ficha.

**Eliminar.**

1. En la ficha del rol, haz clic en **Editar** y luego en **Eliminar** (al pie de la tarjeta «Configuración del Rol»; permiso `Eliminar`).
2. Se abre un cuadro «Confirmar eliminación» con el mensaje «¿Estás seguro de que deseas eliminar este rol?» y los botones **Cancelar** y **Confirmar**. **Cancelar** cierra el cuadro y te deja en el modo editar (comprobado).
3. **Confirmar**: aparece «Rol eliminado correctamente» y vuelves a la lista de roles.

**Qué impide eliminar un rol: que tenga cuentas.** Comprobado: se asignó «Rol Prueba Eliminar» a la cuenta de prueba y se intentó eliminar; apareció el aviso rojo «Error al eliminar el rol» (sin más explicación) y el rol siguió existiendo. Para poder eliminarlo, cambia primero el rol de esas cuentas a otro (sección 7.3); después sí aparece «Rol eliminado correctamente». El aviso es el mismo para cualquier otro fallo al eliminar.

El rol «Admin» no tiene una protección especial en la pantalla: no lo elimines mientras alguna cuenta lo use. Si borraras el rol «Admin» sin cuentas, tu empresa perdería su acceso total. Por tanto, **no elimines el rol «Admin»**.

### 9.4 Qué quedó al terminar las pruebas

- El empleado «Pedro Prueba» y otro empleado de prueba («Lucia Prueba Cuenta») se eliminaron. La cuenta `pedro.prueba` quedó en el estado oculto descrito en 9.1 (se eliminó su empleado antes que la cuenta, para mostrar el efecto); la cuenta de «Lucia Prueba Cuenta» sí se eliminó primero, como corresponde.
- El rol «Rol Prueba Eliminar» se eliminó. El rol «Rol Carga Uno» (creado por la carga masiva de la sección 10.3) también se eliminó.
- «Sucursal Demo Norte» quedó otra vez sin gerente.

## 10. Buscar, exportar y cargar roles

### 10.1 Buscar roles

**Cómo llegar:** **Roles & Permisos** (`/roles`). Los tres filtros son:

| Filtro | Qué hace |
|---|---|
| Nombre del Rol | Texto con lupa. Busca roles cuyo nombre contenga lo que escribas, sin distinguir mayúsculas (`CAJ` encuentra «Cajero») |
| Descripción | Texto con lupa. Igual, sobre la descripción (`administrador` encuentra «Admin») |
| Nivel de Jerarquía | Un deslizador con dos puntas, de 0 a 100. Arriba se lee «Nivel de Jerarquía: 0 — 100» y cambia al mover las puntas |

Los filtros se combinan; **Enter** en los campos de texto también busca. Comprobado: con el mínimo en 20 («20 — 100») la búsqueda trae solo «Cajero» (nivel 30); con el máximo en 20 («0 — 20») trae solo «Admin» (nivel 0). **Limpiar** vacía los textos, devuelve el deslizador a 0 — 100 y oculta la tabla. Los resultados aparecen al hacer clic en **Buscar**.

La tabla tiene tres columnas: «Nombre del Rol» (enlace a la ficha), «Descripción» («—» si no tiene) y «Nivel de Jerarquía». Se ordena por nivel, del más alto al más bajo. Encima se lee «Resultados (N)» y debajo están **Exportar** y el paginador (10, 25 o 50 por página). Si falla la búsqueda: «Error al buscar roles».

### 10.2 Exportar roles

Con la tabla a la vista, haz clic en **Exportar** (abajo a la izquierda). Se genera `roles.xlsx` con los roles que cumplan los filtros que tengas escritos, con tres columnas: «Nombre del Rol», «Descripción» y «Nivel de Jerarquía». Comprobado: el servidor respondió correctamente (código 200) y no apareció ningún aviso de error. (No verificado en la aplicación: el contenido del archivo ni su descarga, porque el navegador integrado no muestra los archivos descargados; las columnas vienen del inventario 19b §4.1.) Si falla: «Error al buscar roles».

### 10.3 Cargar roles desde un Excel (carga masiva)

Sirve para crear muchos roles de una vez. **Cómo llegar:** **Roles & Permisos** › botón **Carga Masiva** (`/roles/carga-masiva`). Necesitas `Crear` en «Roles y Cuentas de Usuario». Sin ese permiso no ves el botón y, si escribes la dirección, el sistema te devuelve al Inicio con «No tienes permisos para acceder a esta pantalla.»

La pantalla es la misma que la de sucursales (capítulo 2, sección 8): flecha de volver, título «Carga Masiva», migas «Inicio / Administración / Roles & Permisos / Carga Masiva», el botón **Descargar plantilla** y la zona punteada «Arrastra tu archivo Excel aquí o haz clic para seleccionar» con el texto «Solo archivos .xlsx (máx. 5 MB y 5000 filas)». El flujo es el mismo: descarga la plantilla, llénala (una fila por rol, desde la fila 2), súbela, revisa la **vista previa** y haz clic en **Guardar válidos (N)**.

Las columnas del Excel (escritas así, sin tildes ni espacios) son `NombreRol` (obligatoria, hasta 50 caracteres), `Descripcion` (opcional, hasta 255) y `NivelJerarquia` (obligatoria, número entero de 1 o más). (No verificado en la aplicación: el contenido del archivo de plantilla, porque no se puede ver la descarga; los nombres vienen de la regla RN-ROL-022. Sí se comprobó que **Descargar plantilla** respondió correctamente y sin avisos.)

**Cómo se probó.** El navegador integrado no tiene un selector de archivos del sistema. Para poder comprobar el flujo se construyó, dentro del propio navegador, un archivo `.xlsx` de prueba con siete filas y se entregó al control de archivo de la pantalla. Por eso el gesto de arrastrar o hacer clic **no se verificó en la aplicación**, pero sí lo que hace la pantalla con el archivo.

**La vista previa** mostró «Vista previa», «7 filas totales», «1 filas válidas» y «6 filas con error», con una tabla de columnas «Nombre del Rol», «Descripción», «Nivel de Jerarquía», una columna sin título con los errores de la fila y «Estado» (marca verde si la fila es válida, icono de error si no). Estos fueron los errores, copiados tal como aparecieron:

| Fila de prueba | Mensaje | Qué significa | Qué hacer |
|---|---|---|---|
| Nivel `0` | «'nivelJerarquia' debe ser mayor o igual a 1» | La vista previa exige nivel 1 o más | Usa un nivel de 1 en adelante |
| Rol `Cajero` (ya existe) | «El valor 'Cajero' del campo 'nombreRol' ya existe en el sistema» | Ya tienes un rol con ese nombre | Cambia el nombre o quita la fila |
| Dos filas con el mismo nombre | «El valor 'Rol Carga Repetido' del campo 'nombreRol' está duplicado en el archivo» | El nombre aparece más de una vez; se marcan **todas** las filas repetidas, incluida la primera | Deja una sola fila |
| Nombre vacío | «El campo 'nombreRol' es obligatorio» | Falta el nombre | Complétalo |
| Nivel `abc` | «'nivelJerarquia' debe ser un número entero» | El nivel no es un número entero | Escribe un número |

Los nombres de campo salen como los usa el sistema por dentro (`nombreRol`, `nivelJerarquia`). No puedes corregir las filas en la pantalla: corrige tu Excel y súbelo de nuevo; una vista previa nueva reemplaza a la anterior.

**Guardar.** Haz clic en **Guardar válidos (1)**. Solo se guardan las filas válidas. Aparece «Se guardaron 1 registros correctamente» y el sistema te lleva a la lista de roles. El rol creado así («Rol Carga Uno», nivel 20, sin permisos ni sucursales) se eliminó después desde su ficha. Un rol creado por carga masiva **nace sin permisos**: ábrelo con **Editar** para marcarlos.

Otros errores del archivo (los mismos de la carga masiva de sucursales; **No verificado en la aplicación**, vienen de 19a §6.5): «El archivo supera el tamaño máximo de 5 MB.», «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.», «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.», «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.», «Error al procesar el archivo», «Error al guardar los registros», «Error al descargar la plantilla. Intenta de nuevo.» y «No hay filas válidas para guardar». El aviso «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» sí se vio: apareció cuando un primer archivo de prueba mal construido fue rechazado por el servidor.

Una diferencia entre vista previa y guardado: la vista previa exige nivel 1 o más, pero el servidor al guardar aceptaría 0 (regla RN-ROL-022). Como la pantalla guarda solo las filas que pasaron la vista previa, en la práctica los niveles siempre serán 1 o más. Y si entre la vista previa y el guardado alguien crea un rol con el mismo nombre, esa fila se omite sin avisar (el aviso dice cuántas se guardaron realmente; No verificado en la aplicación).

## 11. Lo que ve un cajero frente a un administrador

| Pantalla o elemento | Administrador (rol «Admin») | Cajero (rol «Cajero» del ejemplo) |
|---|---|---|
| Menú «Empleados», «Roles & Permisos» y «Cuentas de Usuario» | Los ve | Solo ve «Empleados» (tiene `Empleados › Leer`). «Roles & Permisos» y «Cuentas de Usuario» no aparecen porque no tiene `Roles y Cuentas de Usuario › Leer` |
| Escribir la dirección de una pantalla sin permiso (`/roles`, `/cuentas`, `/empleados/nueva`) | — | El sistema lo devuelve al Dashboard con «No tienes permisos para acceder a esta pantalla.» |
| Empleados: botón **Nuevo** | Lo ve | No lo ve (falta `Empleados › Crear`) |
| Empleados: botones **Editar** y **Eliminar** de la ficha | Los ve | No los ve (la ficha es de solo lectura) |
| Empleados: **Exportar** | Lo ve | Lo ve (la pantalla no lo condiciona a un permiso propio; el servidor exige `Leer`) |
| Empleados: lista de sucursales al buscar o al abrir la ficha | Funciona | El cajero no tiene `Sucursales › Leer`: esa consulta falla en silencio (sin aviso) y el autocompletado de «Sucursal Base» no ofrece sugerencias; el nombre de la sucursal del empleado sí se ve en la ficha |
| Matriz de permisos, cuentas, carga masiva de roles | Los usa | No tiene acceso |
| Cambiar su propia contraseña (menú de usuario › **Cambiar contraseña**) | Sí | Sí: cualquier persona con sesión puede, sin permiso especial |
| Cambio obligatorio de contraseña en el primer ingreso | No aplica a la administradora del registro | Sí, la primera vez que entra con la contraseña inicial (sección 8.1) |
| Cuenta de otro administrador | Puede modificarla o eliminarla | No puede (y no tiene acceso a las cuentas) |

Un detalle de privacidad: como el cajero tiene `Empleados › Leer`, ve en la lista los **correos y licencias** de todos los empleados de la empresa. Si no quieres eso, quita ese permiso al rol (el filtro de ventas por empleado del punto de venta dejará de funcionar).

## 12. Diferencias con la documentación técnica

Lo que hace la aplicación coincidió con el inventario `19b-administracion.md` (versión 1.1) y con la matriz `05b-matriz-de-permisos.md` en casi todo lo comprobado. Estas son las diferencias y los matices:

| # | Documento y apartado | Dice | La aplicación muestra |
|---|---|---|---|
| 1 | 19b §5.2 (error al cargar la cuenta, `cuentas.detail.load-error`) | Se muestra «Error al cargar la cuenta» | Al abrir por dirección a mano la ficha de una cuenta cuyo empleado fue eliminado, el aviso mostró la clave sin traducir `cuentas.detail.load-error` (la traducción del módulo no estaba cargada en ese momento) |
| 2 | 19b §3.2 y RN-EPL-015 (eliminar empleado) | El handler no comprueba si el empleado tiene cuenta | Coincide: se pudo eliminar un empleado con cuenta. Lo que los documentos no dicen: la cuenta desaparece de la lista de cuentas y no se puede abrir ni eliminar, y su usuario sigue ocupado (sección 9.1) |
| 3 | 19b §3.2 (correo) | Solo valida formato en pantalla (`Validators.email`) y el servidor valida de nuevo | Coincide, con un matiz comprobado: `carlos.cajero@sofia` (sin punto tras la arroba) pasa la validación de la pantalla y el servidor responde «Ingresa un correo electrónico válido.» |
| 4 | 19b §5.2 (cuenta, mensaje de longitud) | El aviso de longitud máxima del usuario dice 100 y el validador es 50 | Coincide: con más de 50 caracteres aparece «La longitud máxima es de 100 caracteres» |
| 5 | 19b §5.2 y RN-CTA-004 (contraseña) | Pide de 8 a 128 caracteres pero solo avisa si está vacía | Coincide: con 7 caracteres no aparece ningún mensaje y no se guarda |
| 6 | 19b §4.2 y RN-ROL-004 (nivel) | El formulario exige mínimo 1 y muestra «Este campo es obligatorio» | Coincide: el valor inicial `0` da «Este campo es obligatorio» |
| 7 | 19b §4.2 (nombre del rol) | El formulario admite 100 caracteres y la API 50; el nombre duplicado y los de 51 a 100 dan «Error al guardar el rol» | Coincide (se comprobó con `Admin` repetido y con un nombre de 60 caracteres) |
| 8 | 19b §4.2 (eliminar rol con cuentas) | Cualquier error, incluido el rol en uso, muestra «Error al eliminar el rol» | Coincide (comprobado con un rol asignado a una cuenta) |
| 9 | 19b §5.2 (rol de la cuenta) | La pantalla trabaja con un solo rol; el buscador trae el rol actual como texto inicial | Coincide. Matiz comprobado: como el texto inicial filtra la lista, para cambiar de rol hay que borrar el texto y escribir el nombre del rol nuevo |
| 10 | 05b §6.5 (permisos de un cajero) | «No es una configuración existente ni una recomendación de negocio» | Se tomó como insumo; el conjunto de 19 permisos de la sección 5.3 se verificó en la aplicación: con él cargaron sin error de permisos el dashboard, las sesiones de caja, la lista de ventas y la pantalla de venta nueva. Se agregaron `Profesionales de Salud › Leer` (para registrar una receta) y `Empleados › Leer` (filtro de la lista de ventas), que 05b §5 lista como lecturas adicionales |
| 11 | 19a §2 (cambio obligatorio de contraseña) | Cualquier opción del menú rebota a `/cambiar-clave` | Coincide, sin ningún aviso. Matiz: el mensaje de ingreso fallido aparece dentro de la propia pantalla de ingreso, no como aviso inferior |
| 12 | 05b §6.3 (roles de `0002`) | Los 6 roles sembrados no son utilizables | Coincide: la lista de roles de una empresa recién creada solo muestra «Admin» |

## 13. Pendientes y «No verificado»

En curso (sin commitear): ninguno. Los dos repositorios estaban sin cambios locales al empezar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`).

No verificado en la aplicación (cada caso, con su motivo):

| # | Qué | Motivo |
|---|---|---|
| 1 | Contenido y descarga de los archivos de **Exportar** (empleados, roles) y de la plantilla de carga masiva de roles | El navegador integrado no muestra los archivos descargados. Se comprobó que la petición responde 200 y que no hay avisos de error. Las columnas vienen del inventario 19b §3.1 y §4.1 y del documento 20 |
| 2 | El gesto de arrastrar o hacer clic para elegir el archivo en la carga masiva de roles, y los errores de archivo de tamaño, número de filas, descompresión y errores del servidor | Se entregó el archivo al control desde el propio navegador; no se prepararon archivos de ese tipo. Vienen de 19a §6.5 y §8.9 |
| 3 | Mensajes `Rol.AutoAsignacion` («Una cuenta no puede modificar sus propios roles»), `Rol.AdminReservado`, `Cuenta.AdminProtegida`, `Rol.PermisosPropios` y los de sucursal o cuenta inexistente | Exigen una cuenta no administradora con permisos de gestión o probar sobre la cuenta propia del administrador (riesgo de perder el acceso). Vienen de 19b §4.2 y §5.2 |
| 4 | Pantallas de gestión de un rol sin `Gestionar permisos` (casillas deshabilitadas), de una cuenta sin `Asignar roles` (rol solo en texto) y de un usuario sin `Empleados › Leer` al elegir empleado | Solo existe un administrador y un cajero sin esos permisos de gestión; vienen de 19b §4.2 y §5.2 |
| 5 | Qué ve en vivo una persona con la sesión abierta cuando se le cambian los permisos del rol | El navegador integrado mantiene una sola sesión; solo se comprobó el efecto al volver a iniciar sesión |
| 6 | Ingreso con una cuenta desactivada; eliminar y recrear una cuenta con el mismo usuario; reutilizar licencia y correo de un empleado eliminado | No se probaron para no ensuciar los datos; vienen de las reglas RN-CTA-012, RN-CTA-013 y RN-EPL-008 |
| 7 | Avisos genéricos «Error al guardar el empleado», «Error al cargar el empleado», «Error al eliminar el empleado», «Error al guardar la cuenta» por causas distintas de las probadas, «Error al guardar el rol», «Error al cargar el rol», «Error al cargar los permisos», «Error al cargar las sucursales», «Error al buscar empleados/roles/cuentas», «Error al eliminar la cuenta», «La cuenta no existe», «La sucursal no existe» | Son errores del servidor o de red que no se pueden provocar desde la pantalla. Vienen de 19b §3, §4 y §5 |
| 8 | Paginación real (más de una página) en empleados, roles y cuentas, y el comportamiento de «Sin resultados» de la tabla de cuentas y roles | Hay 3 empleados, 2 roles y 2 cuentas. Viene de 19b §0 y §3.1, §4.1, §5.1 |
| 9 | Aviso «Procesando archivo...» de la vista previa de carga masiva | Con archivos pequeños la espera es demasiado corta. Viene de 19a §6.5 |
| 10 | Efecto de la sucursal asignada al rol «Cajero» (más allá de la sucursal base) | No hay una segunda sucursal asignada a la cuenta del cajero; se verá en los capítulos de caja e inventario |
| 11 | Lo que pasa al cambiar la sucursal base de un empleado con cuenta (cierre de sesión) y el cambio propio | Viene de RN-EPL-011 y RN-EPL-012 |

Hallazgos de la aplicación que conviene conocer (no son errores del manual):

- Un empleado con cuenta se puede eliminar sin ningún aviso y deja una cuenta oculta e inaccesible desde la pantalla, con el usuario ocupado. Debería exigir eliminar primero la cuenta o eliminarla junto con el empleado.
- El aviso «Error al cargar la cuenta» puede mostrarse como `cuentas.detail.load-error` (clave sin traducir) al abrir la ficha por dirección directa.
- Una cuenta recién creada no pide rol ni sucursales en el formulario; quien la crea tiene que recordar asignarlos (queda «Sin roles asignados»).
- El correo se valida en pantalla de forma laxa (`persona@dominio` sin punto pasa) y recién el servidor lo rechaza.
- Los avisos de rol duplicado, nombre demasiado largo o rol en uso son todos el mismo genérico «Error al guardar el rol» / «Error al eliminar el rol», sin decir la causa.
- El cajero con `Empleados › Leer` ve los correos de todos los empleados.

## 14. Tabla de cobertura

Cada elemento de las fichas de `19b-administracion.md` de Empleados (§3), Roles y permisos (§4) y Cuentas de usuario (§5), más el cambio obligatorio de contraseña de `19a` (§2), con el apartado del capítulo donde quedó.

### 14.1 Empleados — búsqueda `/empleados` (19b §3.1)

| Elemento | Apartado |
|---|---|
| Título «Empleados», migas «Inicio / Administración / Empleados», sin botón volver; tarjeta «Búsqueda de Empleados» y subtítulo | 3.1 |
| Botón **Nuevo** (permiso `Empleados › Crear`) | 3.1, 3.2, 11 |
| Filtros Nombres, Apellido Paterno, Apellido Materno, Licencia Profesional (lupa, Enter, «contiene») | 3.3 (verificado) |
| Filtro Sucursal Base (autocompletado «Buscar sucursal...», «Sin resultados», filtra por texto) | 3.2, 3.3 (verificado) |
| **Limpiar** y **Buscar**; los resultados no aparecen hasta buscar | 3.1, 3.3 (verificado) |
| Tabla: Nombre Completo (enlace), Apellido Paterno, Apellido Materno (vacío), Licencia Profesional («—»), Correo electrónico («—»), Sucursal Base | 3.2, 3.3 |
| «Resultados (N)», «Sin resultados», paginador 10/25/50 | 3.3 (verificado; varias páginas: No verificado, ver 13 #8) |
| **Exportar**, archivo `empleados.xlsx` y sus 6 columnas | 3.3 (contenido del archivo: No verificado, ver 13 #1) |
| «Error al buscar empleados» | 3.3 (No verificado, ver 13 #7) |
| Permiso `Empleados › Leer`; `Sucursales › Leer` para el autocompletado | 2, 11 |

### 14.2 Empleados — detalle `/empleados/nueva` y `/empleados/:id` (19b §3.2)

| Elemento | Apartado |
|---|---|
| Cabecera (título, migas, flecha de volver); modos crear, ver y editar | 3.2, 3.4, 9.1 |
| Campo Nombres (obligatorio, 75) | 3.2 (verificado) |
| Campo Apellido Paterno (obligatorio, 75) | 3.2 (verificado) |
| Campo Apellido Materno (opcional, 75) | 3.2 (verificado: Carlos y Nora con materno; Pedro sin materno) |
| Campo Sucursal Base (autocompletado, obligatorio, 20 sugerencias, «Sin resultados», debe elegirse de la lista) | 3.2, 9.1 (verificado) |
| Campo Licencia Profesional (opcional, 50, única) | 3.2, 9.1 (verificado) |
| Campo Correo electrónico (opcional, 254, formato, único) y ayuda «Opcional. Se usa para enviar el enlace de recuperación de contraseña.» | 3.2 (verificado) |
| Botón **Editar** (permiso `Actualizar`) | 3.4, 9.1 |
| Botones **Cancelar**, **Guardar**, **Eliminar** (permiso `Eliminar`) | 3.2, 9.1 |
| Cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar este empleado?», **Cancelar**, **Confirmar**) | 9.1 (verificado) |
| «Empleado guardado correctamente» | 3.2, 9.1 (verificado) |
| «Ya existe un empleado con esa licencia profesional.» | 3.2, 9.1 (verificado) |
| «Ya existe un empleado con ese correo electrónico en tu empresa.» | 3.2 (verificado) |
| «Ingresa un correo electrónico válido.» (en el campo y como aviso del servidor) | 3.2 (verificado) |
| Códigos `Db.*` (`errors.dbDuplicado`, etc.) | No aplica: no se pudieron provocar; caen en «Error al guardar el empleado» (3.2; ver 13 #7) |
| «Error al guardar el empleado» (por ejemplo autoasignación de sucursal base) | 3.2, 9.1 (No verificado, ver 13 #11) |
| «Empleado eliminado correctamente» | 9.1 (verificado) |
| «No se puede eliminar un empleado que gerencia una sucursal. Asigna otro gerente primero.» | 9.1 (verificado) |
| «Error al eliminar el empleado», «Error al cargar el empleado» | 9.1, 3.4 (No verificado, ver 13 #7) |
| Sin cuenta visible en la ficha; empleado con cuenta eliminable | 3.4, 9.1 (verificado) |
| Relación empleado, cuenta, gerente de sucursal | 1, 3.4, 7.2, 9.1 |
| No hay carga masiva de empleados | No aplica: no existe en la aplicación (3.1) |

### 14.3 Roles — búsqueda `/roles` (19b §4.1)

| Elemento | Apartado |
|---|---|
| Título «Roles», migas, tarjeta «Búsqueda de Roles» y subtítulo | 4.3 |
| Botones **Carga Masiva** y **Nuevo** (permiso `Crear`) | 4.3, 5.1, 10.3 |
| Filtros Nombre del Rol y Descripción (lupa, Enter) | 10.1 (verificado) |
| Filtro Nivel de Jerarquía (deslizador 0 — 100) | 10.1 (verificado) |
| **Limpiar** y **Buscar** | 10.1 (verificado) |
| Tabla: Nombre del Rol (enlace), Descripción («—»), Nivel de Jerarquía; orden por nivel | 4.3, 10.1 (verificado) |
| «Resultados (N)», fila vacía, paginador | 10.1 (varias páginas: No verificado, ver 13 #8) |
| **Exportar**, `roles.xlsx` con 3 columnas | 10.2 (contenido: No verificado, ver 13 #1) |
| «Error al buscar roles» | 10.1, 10.2 (No verificado, ver 13 #7) |
| Permiso `Roles y Cuentas de Usuario › Leer` | 2 |

### 14.4 Roles — detalle y configuración `/roles/nueva` y `/roles/:id` (19b §4.2)

| Elemento | Apartado |
|---|---|
| Cabecera, modos crear, ver, editar; tarjeta «Configuración del Rol» solo en rol existente | 4.4, 5, 5.1, 5.2 |
| Campo Nombre del Rol (obligatorio, 100 en pantalla, 50 en servidor, bloqueado al editar) | 5.1 (verificado) |
| Campo Nivel de Jerarquía (obligatorio, mínimo 1; `0` da «Este campo es obligatorio») | 5.1 (verificado) |
| Campo Descripción (opcional, 255) | 5.1 |
| Botón **Editar** (permiso `Actualizar`) | 4.4, 5.2, 9.3 |
| Subtítulo «Define el menú, permisos y sucursales para este rol.» y aviso «Los cambios se aplican de inmediato…» | 4.4 |
| Pestaña **Menú** (vista previa, marca verde y círculo vacío, cambia en vivo) | 4.4, 5.3, 6 (verificado) |
| Pestaña **Permisos**: 27 módulos, casilla de módulo, contador «N/M acciones», casillas de acción | 5.2, 5.4 (verificado) |
| Los 27 módulos y sus 105 permisos (Aseguradoras, Auditoría de Seguridad, Despachos Delivery, Devoluciones, DIGEMID, Empleados, Empresa, Formulaciones Clínicas, Ingredientes Activos, Inventario, Jerarquías de Unidades, Laboratorios, Medicamentos, Órdenes Magistrales, Pacientes, Presentaciones de Venta, Profesionales de Salud, Proveedores, Recetas Médicas, Roles y Cuentas de Usuario, Series Fiscales, Servicios, Sesiones de Caja, Sucursales, Transferencias, Unidades de Medida, Ventas) con sus acciones, incluidas las acciones propias (Agendar servicio, Aislar lote en cuarentena, Analizar con IA, Anonimizar datos, Anular, Aperturar caja, Asignar roles, Cerrar caja, Completar orden magistral, Generar acta de destrucción, Gestionar permisos, Iniciar orden magistral, Procesar devolución, Programar delivery, Registrar inmunización, Registrar precio de proveedor) | 5.2 (tabla de 27 filas, leída de la pantalla) |
| Rol «Admin»: sin casillas marcadas pero con acceso total | 4.2 (verificado) |
| Pestaña **Sucursales**: casillas, «No hay sucursales disponibles» | 4.4, 5.5 (verificado; el mensaje vacío: No verificado) |
| Casillas habilitadas solo con `Gestionar permisos` | 2, 5.2 (No verificado, ver 13 #4) |
| Botones **Cancelar**, **Guardar** (guarda descripción, nivel, sucursales y permisos), **Eliminar** (permiso `Eliminar`) | 5.2, 5.4, 6, 9.3 (verificado) |
| Cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar este rol?») | 9.3 (verificado) |
| «Rol guardado correctamente» | 5.1, 5.4, 6 (verificado) |
| «No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado» | 6 (No verificado, ver 13 #3) |
| «Error al guardar el rol» (duplicado, nombre largo) | 5.1 (verificado) |
| «Rol eliminado correctamente»; «Error al eliminar el rol» (rol con cuentas) | 9.3 (verificado) |
| «Error al cargar el rol», «Error al cargar los permisos», «Error al cargar las sucursales» | 5.2, 5.5 (No verificado, ver 13 #7) |
| Nivel de jerarquía: solo ordena, no limita nada | 5.1 |
| Permisos recomendados de un cajero (19 permisos) | 5.3, 5.4 (verificado) |

### 14.5 Roles — carga masiva `/roles/carga-masiva` (19b §4.3, 19a §6.5)

| Elemento | Apartado |
|---|---|
| Título «Carga Masiva», migas, flecha de volver, **Descargar plantilla**, zona de arrastre | 10.3 (verificado) |
| Plantilla `plantilla-roles.xlsx` y sus columnas | 10.3 (contenido: No verificado, ver 13 #1) |
| Vista previa: contadores, tabla con columnas Nombre del Rol, Descripción, Nivel de Jerarquía, Estado | 10.3 (verificado) |
| Errores por fila: `min-value`, `duplicate`, `duplicate-in-file`, `required`, `invalid-integer` | 10.3 (verificado); `max-length`: No verificado |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 10.3 (verificado) |
| Errores del archivo (formato, 5 MB, 5000 filas, descompresión, servidor) | 10.3 («.xlsx válido» verificado; los demás, ver 13 #2) |
| Permiso `Crear` y redirección sin permiso | 10.3 |

### 14.6 Cuentas de usuario — búsqueda `/cuentas` (19b §5.1)

| Elemento | Apartado |
|---|---|
| Título, migas, tarjeta «Búsqueda de Cuentas» y subtítulo | 7.1 |
| Botón **Nuevo** (permiso `Crear`) | 7.1, 7.2 |
| Filtros Nombre de Usuario, Empleado, Cuenta Activa, Bloqueado | 7.1 (verificado) |
| **Limpiar** y **Buscar** | 7.1 (verificado) |
| Tabla: Usuario (enlace), Empleado, Estado (icono), Bloqueado (icono), Intentos | 7.1, 7.6 (verificado) |
| «Resultados (N)», fila vacía, paginador | 7.1 (varias páginas: No verificado, ver 13 #8) |
| No hay **Exportar** ni carga masiva de cuentas | 7.1 |
| «Error al buscar cuentas» | 7.1 (No verificado, ver 13 #7) |

### 14.7 Cuentas de usuario — detalle `/cuentas/nueva` y `/cuentas/:id` (19b §5.2)

| Elemento | Apartado |
|---|---|
| Cabecera y modos crear, ver, editar | 7.2, 7.4, 7.5 |
| Campo Empleado (autocompletado, mínimo 2 letras, obligatorio, no filtra los que ya tienen cuenta) | 7.2 (verificado) |
| Campo Nombre de Usuario (50 caracteres, mensaje «La longitud máxima es de 100 caracteres», único) | 7.2 (verificado) |
| Campo Contraseña (8 a 128, sin mensaje si es corta) | 7.2 (verificado) |
| Contraseña inicial definida por el administrador; cuenta nace con cambio obligatorio | 7.2, 8.1 (verificado) |
| Datos en modo ver (Nombre de Usuario, Empleado, Intentos Fallidos, Cuenta Activa, Bloqueado, Requiere Cambio de Clave, Bloqueado Hasta, Creado el) | 7.4 (verificado) |
| Interruptor **Cuenta Activa** | 7.5 (verificado) |
| Interruptor **Forzar Cambio de Contraseña** | 7.5 (verificado) |
| Casilla **Resetear Intentos Fallidos** y desbloqueo | 7.5, 7.6 (verificado) |
| Tarjeta «Roles Asignados» (subtítulo, «Agregar rol...», «Sin roles asignados», un solo rol, no se puede vaciar) | 7.3 (verificado) |
| Tarjeta «Sucursales Asignadas» (subtítulo, «Agregar sucursal...», «Sin sucursales adicionales asignadas», icono «Quitar sucursal», cambios inmediatos) | 7.7 (verificado) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar** | 7.3, 7.5, 9.2 (verificado) |
| Cuadro «Eliminar cuenta» y mensaje «¿Estás seguro de que deseas eliminar esta cuenta de usuario? Esta acción no se puede deshacer.» con **Confirmar** y **Cancelar** | 9.2 (verificado) |
| «Cuenta guardada correctamente» | 7.2, 7.3, 7.5 (verificado) |
| «Error al guardar la cuenta» (usuario repetido, empleado con cuenta) | 7.2 (verificado) |
| «Cuenta eliminada correctamente» | 9.2 (verificado) |
| «Sucursal asignada correctamente» y «Sucursal eliminada correctamente» | 7.7 (verificado) |
| «Una cuenta no puede modificar sus propios roles», «Solo un administrador puede asignar o quitar el rol Admin», «Solo un administrador puede modificar o eliminar la cuenta de un administrador», «La cuenta no existe», «La sucursal no existe» | 7.3, 7.7, 9.2 (No verificado, ver 13 #3 y #7) |
| «Error al eliminar la cuenta», «Error al asignar la sucursal», «Error al eliminar la sucursal», «Error al cargar la cuenta» | 7.7, 9.2 (No verificado, ver 13 #7; el de cargar apareció sin traducir, sección 12 #1) |
| Bloqueo automático tras 5 intentos fallidos durante 15 minutos | 7.6 (verificado con una cuenta de prueba) |
| No existe «Restablecer contraseña» ni «Bloquear» | 7.5 |

### 14.8 Cambio obligatorio de contraseña `/cambiar-clave` (19a §2)

| Elemento | Apartado |
|---|---|
| Redirección al primer ingreso y bloqueo de menú y URL | 8.1 (verificado) |
| Cabecera «Cambiar contraseña», migas, subtítulo y recuadro «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.» | 8.1 (verificado) |
| Campos Contraseña actual, Nueva contraseña, Confirmar nueva contraseña, icono de ojo | 8.1 (verificado) |
| Sin **Cancelar** ni flecha de volver en modo obligatorio; botón **Guardar** | 8.1 (verificado) |
| «Este campo es obligatorio», «Debe tener al menos 8 caracteres», «La nueva contraseña debe ser distinta de la actual», «Las contraseñas no coinciden» | 8.1 (verificado) |
| «La contraseña actual es incorrecta» | 8.1 (verificado) |
| «No se pudo cambiar la contraseña. Inténtalo de nuevo.» | 8.1 (No verificado, ver 13 #7) |
| «Contraseña actualizada. Inicia sesión con tu nueva contraseña.», cierre de sesiones y vuelta al ingreso | 8.1 (verificado) |
| Acceso voluntario desde el menú de usuario (**Cambiar contraseña**) | 8.1 (el menú se vio; la pantalla en modo voluntario: No verificado) |

### 14.9 Tareas del encargo

| Tarea | Apartado |
|---|---|
| 1 Registrar empleados (Carlos y Nora, opcionales, correo) | 3 |
| 2 Entender los roles (solo «Admin», lista de roles) | 4 |
| 3 Rol «Cajero», matriz, permisos recomendados, sucursales del rol, nivel, qué pasa al quitar un permiso | 5, 6 |
| 4 Cuenta del cajero, errores, forzar cambio, resetear intentos, activar o desactivar, sucursales | 7 |
| 5 Primer ingreso del cajero, qué ve | 8, 11 |
| 6 Editar y eliminar (empleado, cuenta, rol) | 9 |
| 7 Buscar, exportar y carga masiva de roles | 10 |

## 15. Fuentes

Documentos consultados:

- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19b-administracion.md` (§3 Empleados, §4 Roles y permisos, §5 Cuentas de usuario, §0 comportamientos comunes).
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md` (§2 cambio de contraseña, §6.5 carga masiva).
- `docs/ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md` (§1.3, §4.a, §4.b, §5 matriz por pantalla, §6 roles base, §7 acceso por sucursal).
- `docs/ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md` (reglas RN-EPL, RN-ROL, RN-CTA, RN-AUT).
- `docs/_inventario/endpoints.csv` (permisos de cada endpoint).
- Capítulo 2 del manual (`capitulo-02-empresa-sucursales-series.md`) como modelo de formato.

Recorridos hechos en la aplicación local (SPA `http://localhost:4200`, sin datos sensibles):

- Empleados: búsqueda sin filtros y con cada filtro, **Limpiar**, **Exportar**; formulario vacío; correo imposible, incompleto (`persona@dominio`) y repetido; licencia repetida; sucursal base escrita sin elegir; creación de Carlos y Nora; ficha en modo ver; edición, eliminación con gerente, eliminación con cuenta, eliminación normal.
- Roles: lista de la empresa recién creada; ficha del rol «Admin» con sus tres pestañas; creación con formulario vacío, nombre repetido y nombre largo; creación de «Cajero»; marcado de 19 permisos y de una sucursal; contadores y vista previa del menú; **Cancelar**; guardado y recarga; quitar y volver a marcar un permiso; rol de prueba, intento de eliminarlo con una cuenta y eliminación; filtros (texto y deslizador), **Limpiar**, **Exportar**, **Descargar plantilla**, carga masiva con vista previa y guardado.
- Cuentas: búsqueda y filtros; creación con formulario vacío, usuario largo, contraseña corta, usuario repetido y empleado con cuenta; creación de `carlos.cajero`; ficha en modo ver y editar; asignar rol; agregar y quitar sucursal; activar y desactivar; forzar cambio; bloqueo con 5 intentos fallidos de una cuenta de prueba y reseteo; eliminar cuenta (cancelar y confirmar).
- Primer ingreso: cierre de sesión del administrador; ingreso del cajero; pantalla de cambio obligatorio con todos sus errores; bloqueo del menú y de la URL; cambio de contraseña y nuevo ingreso; menú, dashboard, Empleados, Ventas, Sesiones de Caja y punto de venta del cajero; aviso al abrir `/roles`, `/empleados/nueva` y `/profesionales-salud`; revisión de las peticiones a la API de esas pantallas.
