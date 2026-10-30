# Capítulo 11 — Problemas frecuentes y cómo resolverlos

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-26 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de documentos validados (catálogo de códigos de error, inventario de pantallas 19a a 19f, reglas de negocio, matriz de permisos y capítulos 1 a 5 del manual). Este capítulo no se recorrió en la aplicación; los mensajes que sí se vieron en pantalla en los capítulos 1 a 5 se marcan «(Verificado en el capítulo N)» y los demás «(No verificado en la aplicación)» |

## 1. Cómo usar este capítulo

Este capítulo reúne lo que puede salir mal al trabajar con SOFIA y qué hacer en cada caso. Úsalo de dos maneras:

- **Si te apareció un mensaje**, búscalo por su texto exacto en el índice alfabético (sección 5) o en las tablas de la sección 3, que están en el mismo orden del manual: acceso, empresa y sucursales, series fiscales, empleados, roles, cuentas, catálogos, carga masiva, inventario, pacientes y recetas, caja, venta y cobro, anulación, devolución y dashboard.
- **Si algo no funciona pero no hay ningún mensaje** (no ves una opción del menú, la pantalla te saca al inicio de sesión, la boleta dice «Pendiente de envío a SUNAT»…), busca tu situación en la sección 2.

Cómo se ven los avisos:

- Los avisos de error son cuadros rojos que aparecen abajo al centro de la pantalla, con un botón **Cerrar**. Duran unos 5 segundos y se van solos. Si no alcanzas a leerlo, repite la acción.
- En algunas pantallas el aviso lleva, debajo del texto, «Referencia:» seguida de un código largo. Es un identificador de seguimiento: **anótalo o cópialo con el botón de copiar** y dáselo a soporte (sección 4). Ese aviso dura unos 10 segundos.
- En el inicio de sesión y en la recuperación de contraseña el mensaje no es un aviso inferior: es una alerta dentro de la propia tarjeta.
- Los mensajes de **campo** («Este campo es obligatorio», «Ingresa un correo electrónico válido.») aparecen en rojo debajo del campo, y no se envía nada hasta que los corriges.

Una regla general: SOFIA a veces muestra un mensaje **genérico** («Error al guardar la sucursal», «Ocurrió un error al procesar la venta.») que no dice la causa. Para cada mensaje genérico, la sección 3 lista las causas más comunes y qué revisar.

Los permisos se nombran como en la pantalla de roles: primero el módulo y después la acción, por ejemplo `Ventas › Anular`.

## 2. Problemas por situación (sin mensaje de error)

### 2.1 No veo una opción del menú o un botón

Casi siempre es una cuestión de **permisos**. Un permiso es un par «módulo + acción» que el administrador marca en **Administración › Roles & Permisos** para el rol de cada persona (capítulo 3). Las reglas generales:

- Una opción del menú aparece solo si tu rol tiene `Leer` del módulo de esa pantalla. Si un grupo se queda sin opciones, desaparece entero. «Dashboard» siempre está.
- Un botón (**Nuevo**, **Carga Masiva**, **Editar**, **Eliminar**…) aparece solo si tu rol tiene la acción correspondiente: **Nuevo** y **Carga Masiva** necesitan `Crear`; **Editar** necesita `Actualizar`; **Eliminar** necesita `Eliminar`. **Exportar** lo ve cualquiera que pueda abrir la búsqueda.
- El rol **Admin** lo ve y lo hace todo; no depende de casillas marcadas.
- Si cambian tus permisos mientras trabajas, **cierra sesión y vuelve a entrar**: el menú se actualiza al iniciar sesión.
- Si escribes a mano la dirección de una pantalla que no puedes ver, vuelves al Inicio con «No tienes permisos para acceder a esta pantalla.» (Verificado en el capítulo 3.)

Qué permisos necesita cada pantalla (nombres como en la pantalla de roles):

| Pantalla (menú) | Para entrar necesitas | Para usar sus botones | Además necesita (si falta, esa parte falla o queda vacía) |
|---|---|---|---|
| Dashboard | Nada | Cada tarjeta pide el `Leer` de su módulo: Ventas, Sesiones de Caja, Inventario, DIGEMID, Devoluciones. **Abrir caja** pide `Sesiones de Caja › Aperturar caja` | — |
| Empresa | `Empresa › Leer` | Guardar cambios: `Empresa › Actualizar` | — |
| Sucursales | `Sucursales › Leer` | `Crear`, `Actualizar`, `Eliminar` del mismo módulo | `Empleados › Leer` para elegir el gerente |
| Empleados | `Empleados › Leer` | `Crear`, `Actualizar`, `Eliminar` del mismo módulo | `Sucursales › Leer` para elegir la sucursal base |
| Roles & Permisos | `Roles y Cuentas de Usuario › Leer` | `Crear` (nuevo rol y carga masiva), `Actualizar` (descripción y nivel), `Gestionar permisos` (casillas de permisos y sucursales), `Eliminar` | — |
| Cuentas de Usuario | `Roles y Cuentas de Usuario › Leer` | `Crear`; `Actualizar` (activar, desactivar, forzar cambio, sucursales); `Asignar roles` (cambiar el rol); `Eliminar` | `Empleados › Leer` para elegir al empleado; `Sucursales › Leer` para las sucursales de la cuenta |
| Series Fiscales | `Series Fiscales › Leer` | `Crear`, `Actualizar`, `Eliminar` del mismo módulo | `Sucursales › Leer` |
| Unidades de Medida, Laboratorios, Ingredientes Activos | `Leer` de su módulo | `Crear` (**Nuevo** y **Carga Masiva**), `Actualizar`, `Eliminar` | — |
| Catálogo DIGEMID | `DIGEMID › Leer` | `DIGEMID › Crear` para **Carga Masiva** (no hay **Nuevo**, **Editar** ni **Eliminar**) | — |
| Medicamentos | `Medicamentos › Leer` | `Crear`, `Actualizar`, `Eliminar` | `Laboratorios › Leer` y `Unidades de Medida › Leer` para las listas del formulario |
| Jerarquías de Unidades | `Jerarquías de Unidades › Leer` | `Crear`, `Actualizar`, `Eliminar` | `Medicamentos › Leer` y `Unidades de Medida › Leer` |
| Presentaciones de Venta | `Presentaciones de Venta › Leer` | `Crear`, `Actualizar`, `Eliminar` | `Medicamentos › Leer` y `Jerarquías de Unidades › Leer` |
| Formulaciones Clínicas | `Formulaciones Clínicas › Leer` | `Crear`, `Actualizar`, `Eliminar` | `Ingredientes Activos › Leer`, `Medicamentos › Leer` y `Unidades de Medida › Leer` |
| Proveedores | `Proveedores › Leer` | `Crear`, `Actualizar`, `Eliminar`; registrar un precio de compra: `Registrar precio de proveedor` | `Medicamentos › Leer` para buscar el producto en el precio |
| Aseguradoras | `Aseguradoras › Leer` | `Crear`, `Actualizar`, `Eliminar` | — |
| Lotes y Stock por Sucursal | `Inventario › Leer` | Lotes: `Crear`, `Actualizar`, `Eliminar`; registrar o ajustar stock: `Inventario › Actualizar` | `Medicamentos › Leer` para buscar el producto |
| Sesiones de Caja | `Sesiones de Caja › Leer` | **Nuevo**: `Aperturar caja`; **Cerrar Sesión**: `Cerrar caja` | — |
| Ventas | `Ventas › Leer` | **Nueva Venta POS**: `Ventas › Crear`; **Cobrar Venta** (venta en espera): `Crear` y `Actualizar`; **Anular Venta**: `Ventas › Anular`; **Devolver**: `Devoluciones › Procesar devolución` | `Empleados › Leer` y `Pacientes › Leer` para los filtros |
| Terminal de venta (se abre desde Ventas) | `Ventas › Leer` y `Ventas › Crear` | Pestaña «Receta»: `Recetas Médicas › Analizar con IA`; enlace «Registrar receta»: `Recetas Médicas › Crear` | `Medicamentos › Leer`, `Sesiones de Caja › Leer` (para encontrar tu caja abierta), `Pacientes › Leer`, `Recetas Médicas › Leer` |
| Pacientes y Profesionales de Salud | `Leer` de su módulo | `Crear`, `Actualizar`, `Eliminar` | — |
| Recetas Médicas | `Recetas Médicas › Leer` | `Crear`, `Actualizar`, `Eliminar`; escáner: `Analizar con IA` | `Pacientes › Leer` y `Profesionales de Salud › Leer` |

Si te falta `Sesiones de Caja › Leer`, el terminal de venta no encuentra tu caja y te pide abrirla aunque ya la tengas abierta; al intentarlo verás «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (Fuente: 19f §3.1.)

### 2.2 La pantalla me saca al inicio de sesión

SOFIA renueva tu sesión sola mientras trabajas. Solo te lleva a «Iniciar sesión» **sin ningún aviso** cuando la renovación ya no es posible:

- Pasaron más de 7 días sin usar SOFIA.
- Cerraste sesión, o cambiaste o restableciste tu contraseña, en otro navegador o equipo (se cierran todas las sesiones de la cuenta).
- Un administrador desactivó tu cuenta.
- La suscripción de tu empresa dejó de estar vigente.

Qué hacer: escribe de nuevo tu usuario y contraseña. Lo que no habías guardado en un formulario se pierde, así que guarda con frecuencia. Si al volver a entrar ves «Credenciales inválidas…» o el aviso de suscripción, ve a la sección 2.3. (Fuente: capítulo 1, sección 9.1; No verificado en la aplicación: no se esperó a que la sesión venciera.)

### 2.3 No puedo entrar

| Situación | Qué ves | Qué hacer |
|---|---|---|
| Te equivocaste de contraseña 5 veces seguidas | «Credenciales inválidas. Verifica tu usuario y contraseña.» (el mismo mensaje aunque escribas bien la contraseña) | La cuenta se bloquea **15 minutos**. Espera, o recupera tu contraseña por correo (quita el bloqueo), o pide al administrador que marque **Resetear Intentos Fallidos** en tu cuenta. (Verificado en el capítulo 3) |
| Olvidaste tu contraseña | — | Haz clic en **¿Olvidaste tu contraseña?**, escribe tu usuario y abre el enlace del correo (vale 1 hora y solo el último sirve). Requiere que tu ficha de empleado tenga correo. (Verificado en el capítulo 1) |
| Pediste el enlace pero no llega el correo | El aviso verde dice siempre lo mismo, exista o no el usuario | Revisa que escribiste bien el **usuario** (no el correo) y la carpeta de correo no deseado. El correo solo sale si la cuenta está activa, tu empleado tiene correo y la empresa está vigente. Si tu empleado no tiene correo, el administrador puede usar **Forzar Cambio de Contraseña** solo mientras recuerdes la actual. (Verificado en los capítulos 1 y 3) |
| Tu cuenta fue desactivada | «Credenciales inválidas. Verifica tu usuario y contraseña.» | Pide al administrador que active la cuenta (**Cuenta Activa**). (No verificado en la aplicación) |
| La suscripción de tu empresa venció o fue suspendida | «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» | Solo soporte puede reactivarla. (No verificado en la aplicación) |
| Escribes bien todo y no entra | «Credenciales inválidas…» | Revisa mayúsculas y minúsculas, y que escribes el nombre de usuario (por ejemplo `carlos.cajero`), no tu nombre. |

### 2.4 Cambié el idioma y volvió a español

Es normal: el idioma **no se guarda**. Si recargas la página, cierras sesión o vuelves a entrar, la interfaz vuelve a español. Las pantallas de acceso (iniciar sesión, registro, recuperar contraseña) siempre están en español. Si trabajas en inglés, tendrás que volver a elegirlo cada vez desde la bandera de la cabecera. (Fuente: capítulo 1, sección 4.3.)

### 2.5 No puedo cerrar mi caja

| Causa | Cómo reconocerla | Qué hacer |
|---|---|---|
| No ves el botón **Cerrar Sesión** | El botón solo aparece en una sesión «Abierta» y con el permiso `Sesiones de Caja › Cerrar caja` | Pide el permiso al administrador. |
| La caja la abrió otra persona | Al **Confirmar Cierre** ves «Solo el cajero que abrió la caja o un administrador puede cerrarla.» | Que la cierre quien la abrió o un administrador. |
| La sesión ya estaba cerrada | «Error al cerrar la sesión»; la sesión figura como «Cerrada» o «Cuadrada» | No hay nada más que hacer. Revisa el estado en **Sesiones de Caja**. |
| El efectivo esperado resulta negativo | «Error al cerrar la sesión» aunque la sesión esté abierta; pasa si los reembolsos en efectivo o las anulaciones de otras cajas superan lo que hay en el cajón | La pantalla no ofrece otra vía: avisa al administrador y a soporte (sección 4). |
| Falta el monto declarado | «Ingresa el monto declarado al cierre» | Escribe el efectivo contado. |
| Otra operación cambió la sesión a la vez | «Otra operación modificó estos datos al mismo tiempo…» | Recarga y repite. |

Después de cerrar, SOFIA te lleva a la lista sin mostrar el resultado del arqueo: ábrelo desde la lista para ver el monto declarado, el calculado y la diferencia. El estado es «Cuadrada» solo si la diferencia es exactamente 0; con cualquier otra diferencia queda «Cerrada». (Fuente: 19f §1.3; No verificado en la aplicación en este capítulo.)

### 2.6 El cajero no ve ventas de otra sucursal

Es a propósito. La lista de **Ventas**, el detalle de una venta, el terminal, las anulaciones, los cobros y las devoluciones trabajan **solo con la sucursal base** de la persona, aunque tenga más sucursales asignadas. Si abres una venta de otra sucursal verás «Error al cargar la venta». En cambio, **Sesiones de Caja**, **Stock por Sucursal** y el **Dashboard** sí usan todas las sucursales a las que la persona tiene acceso (su sucursal base, las de su cuenta y las de su rol). El rol Admin ve todas. Para que alguien venda en otra sucursal, el administrador debe cambiar su **sucursal base** (en **Empleados**; nadie puede cambiar la propia). (Fuente: matriz de permisos 05b §7; No verificado en la aplicación.)

Algo parecido pasa en el Dashboard: con **más de una** sucursal visible aparece la tabla «Ventas por sucursal» y se **ocultan** los botones «Ver ventas de hoy», «Ver mis ventas» y «Ver ventas en espera», porque la lista de ventas solo cubre una sucursal. (Fuente: 19a §3.2.)

### 2.7 La boleta dice «Pendiente de envío a SUNAT»

Es lo normal hoy. SOFIA emite la boleta y la guarda, pero **todavía no la envía a SUNAT**: no hay integración con un proveedor de facturación electrónica. Por eso toda boleta nueva nace con ese estado, y el cuadro «Venta Procesada Exitosamente» siempre dice «Comprobante registrado. Pendiente de envío a SUNAT.», incluso al reimprimir. No es un error y no hay nada que corregir de tu lado. Los estados «Aceptado por SUNAT» y «Rechazado por SUNAT» existen en el texto pero hoy nada los asigna; «Anulado» aparece cuando anulas la venta. Consulta con tu administrador cómo cumple tu botica con la obligación de enviar sus comprobantes mientras tanto. (Fuente: RN-CMP-007 en 01c2; No verificado en la aplicación en este capítulo.)

### 2.8 La exportación sale con la fecha del día siguiente

Las pantallas muestran las fechas en la hora de tu navegador, pero algunos archivos Excel escriben la fecha y hora en **UTC** sin convertirla a hora de Lima (UTC menos 5 horas). Un movimiento hecho después de las 7 de la noche aparece con la fecha del día siguiente en el archivo. Afecta a la exportación de **Sesiones de Caja** (apertura y cierre) y a la fecha de creación de **Series Fiscales**. Para tener la hora correcta, mira la pantalla. (Fuente: capítulo 2 sección 11.1 y regla RN-CAJ-024; No verificado en la aplicación: no se pudo abrir el archivo descargado.)

### 2.9 Otros comportamientos que sorprenden

| Lo que ves | Por qué pasa | Qué hacer |
|---|---|---|
| Al abrir una búsqueda no hay tabla de resultados | Los resultados solo aparecen al hacer clic en **Buscar**, aunque vuelvas desde un guardado | Haz clic en **Buscar**. |
| Un filtro de empleado o cliente (Ventas) no filtra | El filtro solo se aplica si **eliges una opción** de la lista; el texto escrito sin elegir se ignora | Elige la opción del desplegable. |
| El filtro de nombre del catálogo DIGEMID no encuentra «CAPSULA» al escribir «cápsula» | Ese filtro distingue los acentos | Escríbelo igual que el catálogo. (Verificado en el capítulo 4) |
| Al guardar, a veces vuelves a la lista y a veces te quedas en la ficha | Empleados, Series Fiscales y los catálogos al crear vuelven a la lista; Empresa, Sucursales, Roles y Cuentas se quedan en la ficha | Es normal. |
| Cambiaste los permisos de un rol y la persona no ve el cambio | Se aplica al volver a iniciar sesión | Que cierre sesión y entre de nuevo. |
| La carga masiva dice «Se guardaron 0 registros correctamente» | La vista previa no revisa algunos duplicados (código identificador de laboratorio, DCI de ingrediente) y el guardado los omite en silencio | Compara siempre guardados con filas válidas. (Verificado en el capítulo 4) |
| En la carga masiva se marcan con error todas las filas repetidas, también la primera | Así está hecha la validación | Deja una sola fila. (Verificado en los capítulos 2 a 4) |
| No puedes quitar al cliente elegido en el terminal de venta | No hay botón para quitarlo; solo se descarta al escribir otro texto o al limpiar la venta | Escribe otro cliente, o vacía el carrito y empieza de nuevo. (No verificado en la aplicación) |
| El terminal no muestra la fecha de vencimiento del lote | Solo ofrece lotes no vencidos; no la pinta | Pregunta al administrador si necesitas verla en **Lotes**. (No verificado en la aplicación) |
| Cerraste el navegador a mitad de una venta | El carrito se guarda solo en tu navegador durante 12 horas; se borra al cobrar, al cerrar sesión o si otra persona inicia sesión en ese equipo | Vuelve a abrir el terminal con tu usuario. (No verificado en la aplicación) |
| «Subtotal» más «IGV» no suma exactamente el «TOTAL A PAGAR» | La pantalla redondea cada cifra por separado (hasta un centavo de diferencia) | El total a pagar es el que vale. (No verificado en la aplicación) |
| «Vendido hoy» no coincide con la lista de ventas de hoy | La tarjeta suma las ventas «Completada» y «Devuelta» a monto bruto; el botón «Ver ventas de hoy» filtra solo «Completada» | Es normal. (Fuente: RN-DAS-005 y RN-DAS-014; No verificado en la aplicación) |
| Un empleado eliminado dejó una cuenta oculta | SOFIA permite eliminar un empleado con cuenta | Elimina primero la cuenta y luego el empleado (sección 3.4). (Verificado en el capítulo 3) |
| La campana de la cabecera dice «Sin notificaciones» | Es decorativa; no genera notificaciones | No hace falta hacer nada. (Verificado en el capítulo 1) |


## 3. Mensajes de error por tarea

Las tablas siguen el orden del manual. Cada una tiene cuatro columnas: el mensaje exacto, dónde aparece, qué significa y cómo resolverlo. Al final de cada mensaje verás una marca: **(Verificado en el capítulo N)** si el mensaje se vio en pantalla en ese capítulo, o **(No verificado en la aplicación)** si el texto sale del catálogo de códigos de error y de los inventarios de pantallas pero nadie lo provocó en un recorrido real.

Dos avisos que aplican a todas las pantallas de guardado:

- Si escribes **solo espacios** en un campo obligatorio, la pantalla lo acepta, pero el servidor lo rechaza y verás el mensaje genérico de guardado de esa pantalla (por ejemplo «Error al guardar la unidad de medida»). Escribe siempre un valor real. (Verificado en el capítulo 4.)
- Tres avisos de base de datos pueden aparecer al guardar en Sucursales, Empleados, Unidades de medida, Laboratorios, Ingredientes activos, Jerarquías, Medicamentos, Proveedores, Precios de proveedor, Pacientes, Profesionales de salud y Recetas. Son poco frecuentes (por ejemplo, dos personas guardando lo mismo a la vez). Los verás en la tabla 3.0.

### 3.0 Avisos de base de datos y de simultaneidad

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» | Al guardar, en las pantallas de la lista anterior | Otro registro con el mismo valor único se guardó casi al mismo tiempo que el tuyo | Busca el registro existente y revísalo; si no está, vuelve a guardar. (No verificado en la aplicación) |
| «El registro hace referencia a datos que ya no existen o todavía está en uso.» | Igual | Lo que guardas apunta a algo que alguien eliminó hace un instante, o intentas quitar algo que todavía se usa | Actualiza la pantalla (F5) y revisa que lo que elegiste siga existiendo. (No verificado en la aplicación) |
| «Uno de los valores está fuera del rango permitido.» | Igual | Un número es demasiado grande, negativo donde no se permite o fuera del rango que acepta la base de datos | Revisa los números que escribiste (cantidades, precios, porcentajes). (No verificado en la aplicación) |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Terminal de venta, anulación de venta, apertura y cierre de caja, devolución y stock por sucursal | Otra persona (o tú en otra pestaña) cambió el mismo registro mientras lo trabajabas; por ejemplo, otro cajero vendió del mismo lote | Recarga la página con F5 y repite la operación. Si se repite, avisa al administrador (sección 4). (No verificado en la aplicación) |

### 3.1 Acceso y sesión

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Credenciales inválidas. Verifica tu usuario y contraseña.» | Alerta roja en «Iniciar sesión» | El usuario o la contraseña no coinciden, o la cuenta está desactivada o bloqueada. SOFIA no dice cuál para proteger tu cuenta | Revisa mayúsculas y minúsculas. Si olvidaste la contraseña, usa **¿Olvidaste tu contraseña?**. Si te equivocaste 5 veces, la cuenta se bloqueó 15 minutos (sección 2.3). Si nada funciona, pide al administrador que revise tu cuenta. (Verificado en el capítulo 1) |
| «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.» | Alerta roja en «Iniciar sesión» | La contraseña era correcta, pero la empresa tiene la suscripción vencida, suspendida o cancelada | Comunícate con soporte (sección 4). No lo puedes resolver tú. (No verificado en la aplicación) |
| «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» | «Iniciar sesión», «Recupera tu contraseña» y «Crea una nueva contraseña» | Se hicieron demasiados intentos desde tu equipo en poco tiempo (el inicio de sesión admite 10 por minuto) | Espera un minuto sin pulsar nada y vuelve a intentar. (No verificado en la aplicación) |
| «No pudimos conectar con el servidor. Verifica tu conexión a internet e inténtalo de nuevo en unos minutos.» | «Iniciar sesión» | El servidor no responde, tardó demasiado o falló, o tu internet se cayó | Revisa tu conexión y prueba de nuevo en unos minutos. Si persiste, avisa a soporte. (No verificado en la aplicación) |
| «Ya existe una empresa registrada con este RUC» | Aviso inferior en «Registrar Empresa» | Otra empresa ya usa ese RUC; el RUC es único en todo SOFIA | Revisa que lo escribiste bien. Si es el RUC de tu empresa y nunca la registraste, comunícate con soporte. (Verificado en el capítulo 1) |
| «El usuario ya está en uso. Elige otro.» | Aviso inferior en «Registrar Empresa» | Otra cuenta de cualquier empresa ya usa ese nombre de usuario | Elige otro (por ejemplo, agrega tu apellido o un número). (No verificado en la aplicación) |
| «Ingresa la dirección de la sede principal (máximo 255 caracteres)» | Aviso inferior en «Registrar Empresa» | La dirección de la sede llegó vacía o con más de 255 caracteres | Escribe una dirección de hasta 255 caracteres. (No verificado en la aplicación) |
| «Error al registrar la empresa» | Aviso inferior en «Registrar Empresa» | Mensaje genérico. Causas comunes: RUC que no tiene 11 dígitos, nombre vacío, exceso de envíos desde el mismo equipo (el registro tiene un límite por hora) o servidor sin respuesta | Revisa los datos, espera unos minutos y reintenta. Si persiste, soporte. (No verificado en la aplicación) |
| «No se pudo procesar la solicitud. Inténtalo de nuevo.» | «Recupera tu contraseña» | Falló el servidor o tu conexión al pedir el enlace | Inténtalo otra vez. (No verificado en la aplicación) |
| «Este enlace de recuperación está incompleto o ya fue usado. Solicita uno nuevo.» | «Crea una nueva contraseña» | Abriste la pantalla sin el código del enlace, o con el enlace cortado | Usa el enlace completo del correo o haz clic en **Solicitar un nuevo enlace**. (Verificado en el capítulo 1) |
| «El enlace no es válido o ya expiró. Solicita uno nuevo.» | «Crea una nueva contraseña» | El enlace ya se usó, pasó más de 1 hora o pediste otro enlace después (solo el último sirve) | Pide un enlace nuevo en **¿Olvidaste tu contraseña?** y usa el último correo que llegue. (Verificado en el capítulo 1) |
| «No se pudo restablecer la contraseña. Inténtalo de nuevo.» | «Crea una nueva contraseña» | Falló el servidor o tu conexión | Inténtalo otra vez; si no, pide un enlace nuevo. (No verificado en la aplicación) |
| «La contraseña actual es incorrecta» | Aviso inferior en «Cambiar contraseña» | La contraseña actual que escribiste no es la tuya | Vuelve a escribirla. Si no la recuerdas, cierra sesión y usa la recuperación por correo. Equivocarte aquí no bloquea tu cuenta. (Verificado en el capítulo 1) |
| «No se pudo cambiar la contraseña. Inténtalo de nuevo.» | «Cambiar contraseña» | Falló el servidor o tu sesión ya no es válida (por ejemplo, tu cuenta fue desactivada) | Cierra sesión, entra de nuevo e inténtalo. Si no puedes entrar, ver sección 2.3. (No verificado en la aplicación) |
| «No tienes permisos para acceder a esta pantalla.» | Aviso inferior, y te devuelve al Inicio | Escribiste (o llegaste a) una dirección para la que tu rol no tiene permiso de lectura | Pide al administrador el permiso `Leer` del módulo (sección 2.1). (Verificado en el capítulo 3) |
| «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.» | Aviso inferior; el menú queda solo con «Dashboard» | SOFIA no pudo leer los permisos de tu rol (servidor o conexión) | Recarga con F5; si sigue, cierra sesión y entra de nuevo. (No verificado en la aplicación) |

### 3.2 Empresa y sucursales

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Este campo es obligatorio» | Bajo un campo, en cualquier formulario | Dejaste vacío un campo obligatorio (o con el valor `0` en «Nivel de Jerarquía» de un rol) | Complétalo. No se envía nada hasta que lo hagas. (Verificado en los capítulos 1 a 4) |
| «Error al actualizar la empresa» | Aviso inferior al guardar en el detalle de la empresa | Mensaje genérico. Causas comprobadas: RUC con menos o más de 11 dígitos, RUC con una letra, RUC ya usado por otra empresa, RUC vacío. También puede ser que la empresa ya no exista | Corrige el RUC (11 dígitos numéricos). Si es tu RUC y otra empresa lo usa, comunícate con soporte. Aunque el rótulo diga «opcional», el RUC no se puede dejar vacío. (Verificado en el capítulo 2) |
| «Error al cargar la empresa» | Aviso inferior al abrir el detalle de la empresa | No se pudo leer la empresa (servidor o conexión) | Vuelve a la búsqueda e inténtalo de nuevo. (No verificado en la aplicación) |
| «Error al buscar empresas» | Aviso inferior en la búsqueda o al exportar empresas | Falló la búsqueda o la exportación | Inténtalo de nuevo; si persiste, soporte. (No verificado en la aplicación) |
| «Ya existe una sucursal con ese número de licencia.» | Aviso inferior al guardar una sucursal | Otra sucursal de tu empresa ya usa esa licencia | Escribe la licencia correcta. (Verificado en el capítulo 2) |
| «Error al guardar la sucursal» | Aviso inferior al guardar una sucursal | Mensaje genérico. Causas comunes: nombre, dirección o licencia vacíos o solo espacios, la sucursal o el gerente ya no existen | Revisa que los tres campos tengan texto real y que el gerente elegido siga existiendo. (No verificado en la aplicación) |
| «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.» | Aviso inferior al confirmar la eliminación | La sucursal tiene empleados, cuentas, roles, stock, sesiones de caja, series fiscales, ventas, transferencias, cuarentena u órdenes magistrales. El aviso no dice cuál | Pasa primero a otra sucursal los empleados y cuentas, quita la sucursal de los roles y elimina las series que no hayan emitido comprobantes. (Verificado en el capítulo 2) |
| «Error al eliminar la sucursal» | Aviso inferior al eliminar | Cualquier otro fallo al eliminar (la sucursal ya no existe, sin permiso, sin conexión) | Recarga y vuelve a intentar. (No verificado en la aplicación) |
| «Error al cargar la sucursal» | Aviso inferior al abrir el detalle | No se pudo abrir la sucursal | Vuelve a la búsqueda y ábrela otra vez. (No verificado en la aplicación) |
| «Error al buscar sucursales» | Aviso inferior en la búsqueda o al exportar | Falló la búsqueda o la exportación | Inténtalo de nuevo. (No verificado en la aplicación) |

### 3.3 Series fiscales

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.» | Bajo «Prefijo de Serie» (y como aviso si el servidor lo rechaza) | El prefijo no tiene 4 caracteres, lleva un símbolo o su primera letra no corresponde al tipo (Boleta empieza con B, Factura con F, Nota de Crédito con B o F) | Corrige el prefijo, por ejemplo `B001` para Boleta. (Verificado en el capítulo 2) |
| «El correlativo debe ser un número entero mayor o igual a 0.» | Bajo «Correlativo Actual» | Escribiste un negativo o un decimal | Escribe un entero desde 0. (Verificado en el capítulo 2) |
| «Ya existe otra serie de este tipo de comprobante con ese prefijo.» | Aviso inferior al guardar | El prefijo no se puede repetir en la empresa para el mismo tipo, aunque sea de otra sucursal | Elige otro prefijo (por ejemplo `B002`). (Verificado en el capítulo 2) |
| «La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra.» | Aviso inferior al guardar | Solo puede haber una serie activa por sucursal y tipo | Desactiva la anterior (Estado «Inactiva») o guarda la nueva como «Inactiva». (Verificado en el capítulo 2) |
| «La serie ya emitió comprobantes: solo puedes cambiar su estado.» | Aviso inferior al guardar | Intentas cambiar sucursal, tipo, prefijo o correlativo de una serie que ya se usó | Cambia solo el Estado. (No verificado en la aplicación) |
| «No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla.» | Aviso inferior al eliminar | La serie ya emitió comprobantes | Desactívala en lugar de eliminarla. (No verificado en la aplicación) |
| «La serie fiscal no existe.» | Aviso inferior | Alguien eliminó la serie mientras trabajabas | Vuelve a la búsqueda. (No verificado en la aplicación) |
| «La sucursal seleccionada no existe.» | Aviso inferior al guardar | Alguien eliminó la sucursal elegida | Elige otra sucursal. (No verificado en la aplicación) |
| «Error al guardar la serie fiscal» | Aviso inferior al guardar | Mensaje genérico (por ejemplo, un dato que el formulario dejó pasar) | Revisa los datos e inténtalo de nuevo. (No verificado en la aplicación) |
| «Error al eliminar la serie fiscal» | Aviso inferior al eliminar | Cualquier otro fallo al eliminar | Recarga e inténtalo. (No verificado en la aplicación) |
| «Error al cargar la serie fiscal» | Aviso inferior al abrir el detalle | No se pudo abrir la serie | Vuelve a la búsqueda. (No verificado en la aplicación) |
| «Error al buscar series fiscales» / «Error al exportar las series fiscales» | Aviso inferior en la búsqueda / al exportar | Falló la búsqueda o la exportación | Inténtalo de nuevo. Si exportas más de 10 000 filas, aplica filtros. (No verificado en la aplicación) |
| «No se pudieron cargar las sucursales» | Aviso inferior en la búsqueda o el detalle de series | No se llenó la lista de sucursales | Recarga la pantalla. Necesitas el permiso `Leer` de «Sucursales». (No verificado en la aplicación) |

Los mensajes que te avisan que falta una serie al cobrar o devolver están en las secciones 3.12 y 3.14.

### 3.4 Empleados

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ingresa un correo electrónico válido.» | Bajo el campo de correo, o como aviso rojo al guardar | El correo no tiene forma de correo (por ejemplo `carlos.cajero@sofia`, sin punto tras la arroba, pasa la pantalla pero lo rechaza el servidor) | Escribe el correo completo. (Verificado en el capítulo 3) |
| «Ya existe un empleado con ese correo electrónico en tu empresa.» | Aviso inferior al guardar | Otro empleado de tu empresa usa ese correo (sin distinguir mayúsculas) | Usa otro correo. (Verificado en el capítulo 3) |
| «Ya existe un empleado con esa licencia profesional.» | Aviso inferior al guardar | Otro empleado de tu empresa tiene esa licencia | Revisa el número. (Verificado en el capítulo 3) |
| «Error al guardar el empleado» | Aviso inferior al guardar | Mensaje genérico. Causas: intentas cambiar **tu propia** sucursal base (lo debe hacer otro administrador), la sucursal elegida ya no existe, nombres o apellido paterno vacíos o solo espacios | Pide a otro administrador que cambie tu sucursal base; si no, revisa los datos. (No verificado en la aplicación) |
| «No se puede eliminar un empleado que gerencia una sucursal. Asigna otro gerente primero.» | Aviso inferior al eliminar | El empleado es gerente de una sucursal | En Sucursales, abre la sucursal, **Editar**, cambia o borra el «Gerente» y **Guardar**; luego elimina al empleado. (Verificado en el capítulo 3) |
| «Error al eliminar el empleado» | Aviso inferior al eliminar | Cualquier otro fallo al eliminar | Recarga e inténtalo. (No verificado en la aplicación) |
| «Error al cargar el empleado» | Aviso inferior al abrir la ficha | No se pudo abrir | Vuelve a la búsqueda. (No verificado en la aplicación) |
| «Error al buscar empleados» | Aviso inferior en la búsqueda o al exportar | Falló la búsqueda o la exportación | Inténtalo de nuevo. (No verificado en la aplicación) |

Cuidado con el orden al dar de baja a una persona: SOFIA **permite** eliminar un empleado que todavía tiene cuenta y no avisa. La cuenta desaparece de la lista, deja su nombre de usuario ocupado y ya nadie puede abrirla ni eliminarla desde la pantalla. **Elimina primero la cuenta y después el empleado.** (Verificado en el capítulo 3.)

### 3.5 Roles y permisos

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Error al guardar el rol» | Aviso inferior al guardar un rol | Mensaje genérico que no dice la causa. Causas comprobadas: el nombre del rol ya existe en tu empresa; el nombre tiene más de 50 caracteres (la pantalla deja escribir 100). Al crear, **cualquier** error sale con este texto. También puede ser que un permiso ya estuviera asignado | Usa un nombre distinto y de hasta 50 caracteres. (Verificado en el capítulo 3) |
| «Error al eliminar el rol» | Aviso inferior al eliminar un rol | Casi siempre: el rol todavía tiene cuentas asignadas (también sale ante cualquier otro fallo) | Cambia primero el rol de esas cuentas a otro y vuelve a eliminar. No elimines el rol «Admin». (Verificado en el capítulo 3) |
| «No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado» | Aviso inferior al guardar permisos | Intentas darle más permisos al rol que tú mismo tienes asignado (solo un «Admin» puede) | Pide a un administrador que lo haga. (No verificado en la aplicación) |
| «Error al cargar el rol» / «Error al cargar los permisos» / «Error al cargar las sucursales» | Aviso inferior al abrir el rol o sus pestañas | No se pudo leer el rol, la matriz o la lista de sucursales (para ver sucursales necesitas `Leer` en «Sucursales») | Recarga la ficha. (No verificado en la aplicación) |
| «Error al buscar roles» | Aviso inferior en la búsqueda o al exportar | Falló la búsqueda o la exportación | Inténtalo de nuevo. (No verificado en la aplicación) |

Los errores de la carga masiva de roles están en la sección 3.8.

### 3.6 Cuentas de usuario

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «La longitud máxima es de 100 caracteres» | Bajo «Nombre de Usuario» | Escribiste más de 50 caracteres (el mensaje dice 100 pero el límite real es 50) | Acorta el usuario. (Verificado en el capítulo 3) |
| (Sin mensaje) | Al guardar una cuenta con contraseña de menos de 8 caracteres | La pantalla no avisa: simplemente no guarda | Usa 8 o más caracteres. (Verificado en el capítulo 3) |
| «Error al guardar la cuenta» | Aviso inferior al crear o editar una cuenta | Mensaje genérico. Al crear, causas comprobadas: el nombre de usuario ya existe (es único en **todas** las empresas, también entre cuentas ocultas) o el empleado ya tiene una cuenta. También: el empleado o el rol ya no existen | Elige otro usuario; revisa que el empleado no tenga cuenta; si el usuario quedó ocupado por una cuenta oculta (ver sección 3.4), soporte. (Verificado en el capítulo 3) |
| «Una cuenta no puede modificar sus propios roles» | Aviso inferior al cambiar el rol de tu propia cuenta | Nadie puede cambiar sus propios roles | Pide a otro administrador. (No verificado en la aplicación) |
| «Solo un administrador puede asignar o quitar el rol Admin» | Aviso inferior al asignar el rol | Solo un «Admin» puede dar o quitar el rol «Admin» | Pide a un administrador. (No verificado en la aplicación) |
| «Solo un administrador puede modificar o eliminar la cuenta de un administrador» | Aviso inferior al guardar o eliminar | La cuenta pertenece a un administrador y tú no lo eres | Pide a un administrador. (No verificado en la aplicación) |
| «La cuenta no existe» / «La sucursal no existe» | Aviso inferior al asignar roles o sucursales | Alguien eliminó la cuenta o la sucursal mientras trabajabas | Recarga la pantalla. (No verificado en la aplicación) |
| «Error al eliminar la cuenta» | Aviso inferior al eliminar | Cualquier otro fallo al eliminar | Recarga e inténtalo. (No verificado en la aplicación) |
| «Error al asignar la sucursal» / «Error al eliminar la sucursal» | Aviso inferior en la tarjeta «Sucursales Asignadas» | Falló agregar o quitar la sucursal de la cuenta (este segundo texto dice «eliminar» pero solo la quita de esa cuenta) | Recarga e inténtalo de nuevo. (No verificado en la aplicación) |
| «Error al cargar la cuenta» (a veces se ve como `cuentas.detail.load-error`) | Aviso inferior al abrir la ficha | No se pudo abrir la cuenta; el texto sin traducir aparece al abrir la dirección directamente | Entra desde la lista de cuentas. Si la cuenta es de un empleado eliminado, ver sección 3.4. (Verificado en el capítulo 3) |
| «Error al buscar cuentas» | Aviso inferior en la búsqueda | Falló la búsqueda | Inténtalo de nuevo. (No verificado en la aplicación) |

### 3.7 Catálogos: unidades, laboratorios, ingredientes, DIGEMID, productos, proveedores y aseguradoras

En todos los catálogos, un mensaje «Ya existe…» significa que el valor único (código, nombre, DCI) ya lo tiene otro registro **no eliminado** de tu empresa. Un mensaje «No puedes eliminar… porque…» significa que otro registro todavía usa este: quita primero esa relación.

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ya existe una unidad de medida con ese código.» | Al guardar en Unidades de Medida | El código ya lo usa otra unidad (sin distinguir mayúsculas) | Elige otro código. (Verificado en el capítulo 4) |
| «Error al guardar la unidad de medida» | Al guardar en Unidades de Medida | Genérico. Lo viste con un código de solo espacios; también sale si la unidad ya no existe | Escribe un valor real en código y descripción. (Verificado en el capítulo 4) |
| «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.» | Al eliminar una unidad | La usa algún medicamento, jerarquía o formulación clínica (el texto no nombra las formulaciones, pero también bloquean) | Cambia o elimina primero esos registros. (No verificado en la aplicación) |
| «Error al eliminar la unidad de medida» / «Error al cargar la unidad de medida» / «Error al buscar unidades de medida» | Eliminar / abrir / buscar | Fallo genérico del servidor o la conexión | Recarga e inténtalo. (No verificado en la aplicación) |
| «Ya existe un laboratorio con ese nombre de compañía.» / «Ya existe un laboratorio con ese código identificador.» | Al guardar un laboratorio | El nombre o el código identificador ya lo tiene otro laboratorio | Usa otro, o abre el existente. (Verificado en el capítulo 4) |
| «Error al guardar el laboratorio» | Al guardar un laboratorio | Genérico (nombre de solo espacios, texto demasiado largo, laboratorio inexistente) | Revisa los datos. (Verificado en el capítulo 4) |
| «No se puede eliminar el laboratorio porque tiene medicamentos asociados.» | Al eliminar un laboratorio | Hay medicamentos de ese laboratorio | Elimina o cambia de laboratorio esos medicamentos. (No verificado en la aplicación) |
| «Error al eliminar el laboratorio» / «Error al cargar el laboratorio» / «Error al buscar laboratorios» | Eliminar / abrir / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Ya existe un ingrediente activo con esa denominación DCI.» | Al guardar un ingrediente | La DCI ya está registrada | Abre el ingrediente existente. (Verificado en el capítulo 4) |
| «Error al guardar el ingrediente activo» | Al guardar un ingrediente | Genérico (valor de solo espacios, longitudes excedidas) | Revisa los datos. (Verificado en el capítulo 4) |
| «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» | Al eliminar un ingrediente | Alguna formulación clínica lo usa | Elimina primero esas formulaciones. (No verificado en la aplicación) |
| «Error al eliminar el ingrediente activo» / «Error al cargar el ingrediente activo» / «Error al buscar ingredientes activos» | Eliminar / abrir / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Error al buscar en el catálogo DIGEMID» | Búsqueda o exportación del catálogo | Fallo genérico | Inténtalo de nuevo. (No verificado en la aplicación) |
| «Error al cargar el producto DIGEMID» (a veces se ve como `digemidCatalogo.detail.load-error`) | Al abrir un producto DIGEMID, o al escribir la dirección `/digemid-catalogo/nueva` | El producto no se pudo abrir. El catálogo no se edita ni se crea producto por producto, solo con carga masiva | Vuelve a la búsqueda. (Verificado en el capítulo 4, con el texto sin traducir) |
| «Ya existe un medicamento con ese código nacional.» | Al guardar un medicamento | Otro medicamento no eliminado ya usa ese código | Usa otro código o abre el existente. (Verificado en el capítulo 5) |
| «Error al guardar el medicamento» | Al guardar un medicamento | Genérico: código o nombre de solo espacios, laboratorio o unidad que ya no existen, precio negativo | Revisa los datos. (Verificado en el capítulo 5) |
| «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.» | Al eliminar un medicamento | Tiene lotes, presentaciones, jerarquías, formulaciones, precios de proveedor, citas, inmunizaciones u órdenes magistrales | Quita primero esos registros. (No verificado en la aplicación) |
| «Error al eliminar el medicamento» / «Error al cargar el medicamento» / «Error al cargar las condiciones de venta» / «Error al buscar medicamentos» | Eliminar / abrir / condiciones de venta / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Este producto ya tiene una conversión entre esas unidades.» | Al guardar una jerarquía | Ya existe una conversión de ese producto con esa unidad mayor y menor | Edita la existente. (Verificado en el capítulo 5) |
| «Error al guardar la jerarquía» | Al guardar una jerarquía | Genérico. Causas: la conversión contradice otras equivalencias del producto (incluso pasando por otras unidades), unidad mayor y menor iguales, multiplicador cero o negativo, producto o unidad que ya no existen | Revisa las demás jerarquías del producto y el multiplicador. (Verificado en el capítulo 5) |
| «No se puede eliminar la jerarquía porque está en uso.» | Al eliminar una jerarquía | Alguna relación la usa (hoy el servidor casi no comprueba esto) | Quita lo que la usa. (No verificado en la aplicación) |
| «Error al eliminar la jerarquía» / «Error al cargar la jerarquía» / «Error al buscar jerarquías de unidades» | Eliminar / abrir / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Ya existe una presentación de venta para esa unidad en este producto.» (en el cuadro «Presentaciones de Venta» del medicamento: «Ya existe una presentación de venta para esa unidad.») | Al guardar una presentación | El producto ya tiene una presentación para esa unidad | Edita la existente. (Verificado en el capítulo 5) |
| «Error al guardar la presentación de venta» | Al guardar una presentación | Genérico. Causa típica: las jerarquías del producto no permiten convertir esa unidad a la unidad base; también cantidad en unidades base cero o precio negativo | Registra primero la jerarquía de unidades del producto. (No verificado en la aplicación) |
| «No se puede eliminar la presentación porque tiene ventas registradas.» | Al eliminar una presentación | La presentación ya se vendió | No se puede eliminar. (No verificado en la aplicación) |
| «Error al eliminar la presentación de venta» / «Error al cargar la presentación de venta» / «Error al cargar las presentaciones de venta» / «Error al buscar presentaciones de venta» | Eliminar / abrir / cuadro del medicamento / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Ya existe una formulación con los datos ingresados» | Al guardar una formulación clínica | El medicamento ya lista ese ingrediente activo | Edita la existente. (No verificado en la aplicación) |
| «Error al guardar la formulación clínica» | Al guardar | Genérico: concentración cero o negativa, medicamento, ingrediente o unidad que ya no existen | Revisa los datos. (No verificado en la aplicación) |
| «Error al eliminar la formulación clínica» / «Error al cargar la formulación clínica» / «Error al buscar formulaciones clínicas» | Eliminar / abrir / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Ya existe un proveedor con ese identificador tributario.» / «Ya existe un proveedor con esa razón social.» | Al guardar un proveedor | Otro proveedor tiene el mismo identificador o razón social | Abre el existente. (No verificado en la aplicación) |
| «Error al guardar el proveedor» | Al guardar un proveedor | Genérico: razón social o identificador vacíos o demasiado largos, calificación ESG o tasa de cumplimiento fuera de 0 a 100, términos financieros de más de 100 caracteres | Revisa los datos. (No verificado en la aplicación) |
| «No puedes eliminar este proveedor porque tiene precios de compra registrados.» | Al eliminar un proveedor | Tiene precios de compra | No se puede eliminar mientras los tenga. (No verificado en la aplicación) |
| «Error al eliminar el proveedor» / «Error al cargar el proveedor» / «Error al cargar los precios del proveedor» / «Error al buscar proveedores» | Eliminar / abrir / precios / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «El proveedor ya no existe.» / «El producto seleccionado ya no existe.» | Al registrar un precio de compra | Alguien eliminó el proveedor o el producto | Cierra el cuadro, recarga y elige otro. (No verificado en la aplicación) |
| «Error al registrar el precio» | Al registrar un precio de compra | Genérico: costo negativo, cantidad mínima menor que 1, días de entrega negativos, fecha fin anterior a la de inicio | Revisa los números y las fechas. Para elegir el producto necesitas `Leer` en «Medicamentos». (No verificado en la aplicación) |
| «Error al guardar la aseguradora» | Al guardar una aseguradora | Genérico para cualquier fallo: nombre o código identificador vacío o demasiado largo, código repetido | Revisa los datos. (No verificado en la aplicación) |
| «Error al eliminar la aseguradora» / «Error al cargar la aseguradora» / «Error al buscar aseguradoras» | Eliminar / abrir / buscar | Fallo genérico (también si la aseguradora ya no existe) | Recarga e inténtalo. (No verificado en la aplicación) |

### 3.8 Carga masiva y exportación a Excel

La pantalla de carga masiva es la misma para sucursales, roles, unidades, laboratorios, ingredientes, catálogo DIGEMID, medicamentos, jerarquías, proveedores, pacientes, profesionales y aseguradoras. Primero valida en tu navegador, luego te muestra una **vista previa** y solo guarda las filas válidas.

**Errores del archivo** (aparecen como aviso inferior):

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» | Al elegir el archivo | El nombre no termina en `.xlsx` o el archivo está dañado | Crea el archivo desde la plantilla y guárdalo como `.xlsx`. (Verificado en los capítulos 2 a 4) |
| «El archivo supera el tamaño máximo de 5 MB.» | Al elegir o subir | Pesa más de 5 MB | Divídelo en varios. (No verificado en la aplicación) |
| «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.» | Al subir | Más de 5000 filas con datos | Divídelo. (No verificado en la aplicación) |
| «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.» | Al subir | El contenido descomprimido es sospechosamente grande | Vuelve a crearlo desde la plantilla. (No verificado en la aplicación) |
| «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» | Al hacer clic en **Guardar válidos** | El servidor rechazó filas al guardar; en el catálogo DIGEMID, por ejemplo, un texto demasiado largo pasa la vista previa y hace fallar **todo** el guardado | Revisa los límites de cada columna y sube de nuevo. (Verificado en el capítulo 4) |
| «Error al procesar el archivo» | Al subir | Fallo genérico en la vista previa | Inténtalo de nuevo. (No verificado en la aplicación) |
| «Error al guardar los registros» | Al guardar | Fallo genérico | Inténtalo de nuevo. (No verificado en la aplicación) |
| «Error al descargar la plantilla. Intenta de nuevo.» | Al hacer clic en **Descargar plantilla** | No se pudo bajar | Inténtalo de nuevo. (No verificado en la aplicación) |
| «No hay filas válidas para guardar» | Al guardar | No hay filas válidas (el botón lo impide) | Corrige el Excel. (No verificado en la aplicación) |
| «Se guardaron 0 registros correctamente» (o menos que las filas válidas) | Al guardar | No es un error: SOFIA omitió en silencio filas que repiten un dato que la vista previa no revisa (por ejemplo, el código identificador de un laboratorio o la DCI de un ingrediente) | Compara siempre el número guardado con el de filas válidas. (Verificado en el capítulo 4) |

**Errores por fila** (en la columna sin título de la vista previa; el nombre del campo sale como lo usa el sistema por dentro, por ejemplo `codProd` o `numeroLicencia`):

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «El campo '…' es obligatorio» | Vista previa | Celda obligatoria vacía | Complétala. (Verificado en los capítulos 2 a 4) |
| «'…' no puede superar los N caracteres» | Vista previa | El texto es más largo que el máximo | Acórtalo. (Verificado en el capítulo 4) |
| «El valor '…' del campo '…' ya existe en el sistema» | Vista previa | Ya existe un registro con ese valor | Cambia el valor o quita la fila. (Verificado en los capítulos 2 a 4) |
| «El valor '…' del campo '…' está duplicado en el archivo» | Vista previa | Se repite en el archivo; se marcan **todas** las filas repetidas, incluida la primera, sin distinguir mayúsculas | Deja una sola fila. (Verificado en los capítulos 2 a 4) |
| «'…' debe ser un número entero» / «'nivelJerarquia' debe ser mayor o igual a 1» | Vista previa de roles | El nivel no es un entero o es menor que 1 | Escribe un entero desde 1. (Verificado en el capítulo 3) |
| «'…' debe ser un número decimal válido» / «'…' debe ser mayor que cero» / «'…' debe estar entre N y M» / «'…' debe ser mayor o igual a N» | Vista previa | Número fuera de formato o de rango | Corrige el número. (No verificado en la aplicación) |
| «El formato de '…' no es válido (usa DD/MM/YYYY)» / «'…' no puede ser una fecha futura» | Vista previa | Fecha mal escrita o posterior a hoy | Usa día/mes/año y una fecha pasada. (No verificado en la aplicación) |
| «'…' debe contener exactamente N dígitos numéricos» / «'…' no tiene el formato de su tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números)» | Vista previa | Documento mal escrito | Corrígelo según el tipo. (No verificado en la aplicación) |
| «El valor '…' del campo '…' no existe en el sistema» / «El valor '…' del campo '…' coincide con varios registros del sistema» / «'…' no es un valor válido para '…'» / «'…' no puede ser igual a '…'» | Vista previa | El valor referido (laboratorio, unidad, producto…) no existe, es ambiguo, no está entre las opciones o coincide con otro campo | Escribe el nombre exacto del registro que ya creaste. (No verificado en la aplicación) |

**Exportar a Excel.** Si la exportación falla verás el mismo mensaje de error de búsqueda de esa pantalla (por ejemplo «Error al buscar empleados»), y en Series Fiscales «Error al exportar las series fiscales». Una exportación no puede pasar de **10 000 filas**: aplica filtros para reducirla. El navegador descarga el archivo; si no lo ves, revisa la carpeta de descargas. (No verificado en la aplicación el contenido de los archivos.)

### 3.9 Inventario: lotes y stock por sucursal

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ya existe un lote con ese número para el producto seleccionado» | Aviso inferior al guardar un lote (cuando el servidor responde con conflicto) | Ya hay un lote con ese número para ese producto | Revisa el número o abre el lote existente. (No verificado en la aplicación) |
| «Error al guardar el lote» | Aviso inferior al guardar un lote | Genérico. Causas: el número de lote ya existe para ese producto (el servidor también lo responde así), la fecha de caducidad está en el pasado, la fecha de fabricación es posterior a hoy o a la caducidad, el número tiene más de 100 caracteres, el producto ya no existe | Revisa las fechas y el número. (No verificado en la aplicación) |
| «No se puede eliminar un lote con stock o ventas registradas.» | Aviso inferior al eliminar un lote | El lote tiene una fila de stock en alguna sucursal (aunque sea 0) o una venta | No se puede eliminar. (No verificado en la aplicación) |
| «Error al eliminar el lote» / «Error al cargar el lote» / «Error al buscar lotes» | Eliminar / abrir / buscar o exportar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Error al registrar el stock» | Aviso inferior al registrar stock nuevo | Genérico. Causas: el lote o la sucursal ya no existen, cantidad negativa, dos personas registraron el mismo lote en la misma sucursal a la vez | Espera unos segundos y reintenta; revisa la cantidad. (No verificado en la aplicación) |
| «No tienes permiso para operar el stock de esta sucursal» | Al registrar o ajustar stock | Esa sucursal no está entre las tuyas (sucursal base, asignadas a tu cuenta o a tu rol) | Pide al administrador que te asigne la sucursal. (No verificado en la aplicación) |
| «Error al ajustar la cantidad» | Aviso inferior al ajustar | Genérico: cantidad negativa o el registro de stock ya no existe | Revisa la cantidad. (No verificado en la aplicación) |
| «Error al cargar el registro de stock» / «Error al consultar el stock» | Abrir / buscar o exportar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «No se pudieron cargar tus sucursales» | Al escribir en el filtro «Sucursal» de stock | Falló la lista de tus sucursales | Recarga la pantalla. (No verificado en la aplicación) |

### 3.10 Pacientes, profesionales de salud y recetas

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ya existe un paciente con ese documento de identidad.» | Al guardar un paciente | Otro paciente de tu empresa usa ese número de documento (sin importar el tipo) | Abre el paciente existente. (No verificado en la aplicación) |
| «El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números).» | Al guardar un paciente | El número no cumple el formato de su tipo | Corrige el número o el tipo. (No verificado en la aplicación) |
| «Selecciona un tipo de documento válido.» | Al guardar un paciente | El tipo no es DNI, carné de extranjería, RUC o pasaporte | Elige uno. (No verificado en la aplicación) |
| «Error al guardar el paciente» | Al guardar | Genérico: nombre vacío o de más de 200 caracteres, fecha de nacimiento vacía o futura, contacto de más de 100 caracteres | Revisa los datos. (No verificado en la aplicación) |
| «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» | Al eliminar | Tiene recetas, ventas, citas o inmunizaciones | No se puede eliminar. (No verificado en la aplicación) |
| «Error al eliminar el paciente» / «Error al cargar el paciente» / «Error al buscar pacientes» | Eliminar / abrir / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «Ya existe un profesional de salud con ese número de registro.» | Al guardar un profesional | El número de registro ya está en uso | Abre el existente. (No verificado en la aplicación) |
| «Error al guardar el profesional de salud» | Al guardar | Genérico: nombre o registro vacío o demasiado largo, dirección de más de 255 caracteres | Revisa los datos. (No verificado en la aplicación) |
| «No puedes eliminar este profesional de salud porque tiene recetas asociadas.» | Al eliminar | Tiene recetas | No se puede eliminar. (No verificado en la aplicación) |
| «Error al eliminar el profesional de salud» / «Error al cargar el profesional de salud» / «Error al buscar profesionales de salud» | Eliminar / abrir / buscar | Fallo genérico | Recarga e inténtalo. (No verificado en la aplicación) |
| «El paciente seleccionado no existe. Vuelve a seleccionarlo.» / «El médico seleccionado no existe. Vuelve a seleccionarlo.» | Al guardar una receta | Alguien eliminó el paciente o el médico | Elígelo de nuevo. (No verificado en la aplicación) |
| «La fecha de expedición no puede ser posterior a hoy.» | Al guardar una receta | La fecha es futura o falta | Usa una fecha de hoy o anterior. (No verificado en la aplicación) |
| «Revisa los datos ingresados de la receta.» | Al guardar una receta | Repeticiones máximas negativas o número de receta demasiado largo | Corrige esos campos. (No verificado en la aplicación) |
| «Error al crear la receta» / «Error al actualizar la receta» | Al guardar | Genérico (paciente o médico vacíos, etc.) | Revisa los datos. (No verificado en la aplicación) |
| «No puedes eliminar esta receta porque ya se usó en ventas.» | Al eliminar una receta | La receta figura en una venta | No se puede eliminar. (No verificado en la aplicación) |
| «La receta ya no existe.» / «Error al eliminar la receta» / «Error al cargar la receta» / «Error al buscar recetas médicas» | Abrir, eliminar o buscar | La receta no existe o falló el servidor | Vuelve a la lista y recarga. (No verificado en la aplicación) |
| «Error al interpretar la receta. Intenta nuevamente.» | Escáner de recetas y pestaña «Receta» del punto de venta | Falló el análisis con IA: demasiados análisis seguidos, imagen demasiado grande o inválida, fallo del servicio de IA o de lectura, falta el permiso `Recetas Médicas › Analizar con IA`, sin conexión | Espera un minuto, usa una foto más pequeña y nítida e inténtalo otra vez. (No verificado en la aplicación) |
| «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» | Escáner de recetas (mensaje informativo) | La imagen no contenía medicamentos legibles | Toma otra foto con buena luz y la receta completa. (No verificado en la aplicación) |

### 3.11 Caja: apertura y cierre

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ingresa un monto válido» | Bajo «Monto Apertura (S/)» | Vacío o negativo | Escribe 0 o más. (No verificado en la aplicación) |
| «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» | Al abrir caja (pantalla de sesiones o diálogo del punto de venta) | Ya tienes una caja abierta, en esta o en cualquier otra sucursal | Ve a **Sesiones de Caja**, abre tu sesión y ciérrala; o usa la que tienes. (No verificado en la aplicación) |
| «Error al aperturar la sesión» / «No se pudo aperturar la caja» | Pantalla de apertura / diálogo del punto de venta | Genérico: falta el permiso `Sesiones de Caja › Aperturar caja`, tu cuenta no tiene empleado o sucursal, monto negativo | Revisa tu permiso con el administrador. (No verificado en la aplicación) |
| «Ingresa el monto declarado al cierre» | Bajo «Monto Cierre Declarado (S/)» | Vacío o negativo | Escribe el efectivo contado. (No verificado en la aplicación) |
| «Solo el cajero que abrió la caja o un administrador puede cerrarla.» | Al hacer clic en **Confirmar Cierre** | La caja la abrió otra persona; el botón **Cerrar Sesión** lo ve cualquiera con el permiso, pero el servidor solo deja cerrar al dueño o a un administrador | Pide a quien la abrió, o a un administrador, que la cierre. (No verificado en la aplicación) |
| «Error al cerrar la sesión» | Al cerrar | Genérico. Causas: la sesión ya estaba cerrada, la sesión no existe, o el efectivo esperado resulta **negativo** (reembolsos en efectivo o anulaciones de otras cajas mayores que lo que hay en el cajón): en ese caso la caja no se puede cerrar con esta pantalla | Recarga y revisa el estado de la sesión. Si el estado sigue «Abierta» y el efectivo parece correcto, avisa al administrador y a soporte con los datos de la sección 4. (No verificado en la aplicación) |
| «Error al cargar la sesión de caja» | Al abrir el detalle de una sesión | La sesión no existe o no es de una sucursal que puedes ver | Vuelve a la lista. (No verificado en la aplicación) |
| «Error al buscar sesiones de caja» | Búsqueda o exportación de sesiones | Falló la búsqueda | Inténtalo de nuevo. (No verificado en la aplicación) |

### 3.12 Venta y cobro

El terminal de venta (`/pos/nueva`) tiene mensajes de dos tipos: avisos **antes de enviar** (la pantalla te lo dice) y avisos **del servidor** al cobrar. Cuando el cobro falla, tu carrito y tus pagos se conservan: corrige y vuelve a intentar.

**Avisos antes de cobrar:**

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Debes aperturar la caja del turno antes de procesar una venta.» | En lugar del terminal | No tienes una caja abierta propia en tu sucursal (o tu rol no puede consultar sesiones) | Haz clic en **Aperturar Caja** (si no lo ves, te falta `Sesiones de Caja › Aperturar caja`). (Verificado en el capítulo 3) |
| «No se pudo cargar el catálogo de medicamentos. Inténtalo nuevamente.» | Búsqueda de productos | Falló la lista de productos; también si tu rol no tiene `Medicamentos › Leer` | Vuelve a escribir en el buscador. (No verificado en la aplicación) |
| «No se pudieron cargar los lotes disponibles. Inténtalo nuevamente.» | Al elegir un producto | Falló la lista de lotes | Haz clic en el producto otra vez. (No verificado en la aplicación) |
| «No se pudieron cargar las presentaciones del producto. Inténtalo nuevamente.» | Al elegir un lote | Falló la lista de presentaciones | Elige el lote otra vez. (No verificado en la aplicación) |
| «Stock máximo disponible alcanzado ({{stock}} un.).» | Al agregar o subir cantidad | Pediste más unidades de las que hay en el lote | Baja la cantidad o elige otro lote. (No verificado en la aplicación) |
| «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» | Al agregar al carrito o al cobrar | El producto no tiene precio de venta (los productos de carga masiva nacen sin precio) | Pide al administrador que registre el precio en **Medicamentos**. (No verificado en la aplicación) |
| «Ingresa una cantidad válida para agregar el producto.» | Al pulsar **Agregar** | Cantidad vacía, cero o negativa | Escribe una cantidad mayor que 0. (No verificado en la aplicación) |
| «El medicamento detectado no tiene stock en esta sucursal.» | Al agregar un medicamento detectado en una receta | No hay lotes de ese producto en tu sucursal | Busca otro producto o pide stock. (No verificado en la aplicación) |
| «Ingresa el N° de operación para poder cobrar.» | Bajo los pagos | Un pago con Yape / Plin, tarjeta o transferencia no tiene número de operación | Escríbelo en «Nro Operación». (No verificado en la aplicación) |
| Aviso del carrito: «Hay productos que requieren una receta registrada: selecciona al cliente y su receta antes de cobrar.» | Entre el carrito y los totales | Tu empresa exige receta registrada para productos de receta simple | Elige al cliente y su receta. (No verificado en la aplicación) |
| Aviso del carrito: «Hay productos de receta retenida o estupefacientes: selecciona al cliente y una receta registrada que tenga número de receta.» | Igual | Productos controlados exigen receta registrada con número | Elige cliente y receta con número. (No verificado en la aplicación) |
| Aviso del carrito: «La receta seleccionada no tiene número de receta: agrégalo en la receta para vender productos de receta retenida o estupefacientes.» | Igual | La receta elegida no tiene número | Completa el número en **Recetas Médicas**. (No verificado en la aplicación) |
| Casilla «Receta verificada» con «Confirmas que revisaste la receta física del cliente. Quedará registrado a tu nombre.» | Entre el carrito y los totales | Tu empresa permite vender receta simple con la receta de papel; el botón **Procesar Venta** queda bloqueado hasta marcarla | Revisa la receta de papel y marca la casilla. (No verificado en la aplicación) |

**Avisos del servidor al cobrar o guardar la venta en espera:**

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Necesitas tener tu propia caja abierta en esta sucursal para cobrar la venta.» | Al cobrar una venta que estaba en espera | Quien cobra no tiene su propia caja abierta en la sucursal de la venta | Abre tu caja y cobra de nuevo. (No verificado en la aplicación) |
| «La caja seleccionada pertenece a otro cajero o sucursal. Recarga la página para usar tu propia caja abierta.» | Al cobrar | El terminal tenía guardada una caja que no es tuya | Recarga la página con F5. (No verificado en la aplicación) |
| «Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.» | Al cobrar | Un lote del carrito pasó su fecha de vencimiento (se vende hasta el final de ese día) | Quita esa línea y elige otro lote; avisa al administrador. (No verificado en la aplicación) |
| «El vuelto no puede ser mayor que el efectivo recibido. Ajusta los montos de pago.» | Al cobrar | Sobra dinero y no hay suficiente efectivo para dar el vuelto | Ajusta los montos o agrega efectivo. (No verificado en la aplicación) |
| «Ingresa el número de operación de cada pago que no sea en efectivo.» | Al cobrar | Falta el número de operación en algún pago no efectivo | Complétalo. (No verificado en la aplicación) |
| «Uno de los productos requiere receta médica. Selecciona el cliente y una de sus recetas registradas.» | Al cobrar | Hay un producto que exige receta y no hay una válida | Elige cliente y receta. (No verificado en la aplicación) |
| «La receta médica seleccionada no existe o fue eliminada.» | Al cobrar | Alguien eliminó la receta | Elige otra. (No verificado en la aplicación) |
| «La receta médica no pertenece al cliente de la venta.» | Al cobrar | La receta es de otro paciente | Elige el cliente correcto o su receta. (No verificado en la aplicación) |
| «La receta médica ya fue dispensada el máximo de veces permitido.» | Al cobrar | La receta ya no tiene dispensaciones disponibles | Pide al paciente una receta nueva. (No verificado en la aplicación) |
| «La receta seleccionada no tiene número de receta. Agrégalo en la receta para vender productos de receta retenida o estupefacientes.» | Al cobrar | La receta no tiene número | Agrégalo en **Recetas Médicas**. (No verificado en la aplicación) |
| «Marca "Receta verificada" para confirmar que revisaste la receta del cliente antes de cobrar.» | Al cobrar | El servidor exige la confirmación; aparece la casilla | Márcala y cobra. (No verificado en la aplicación) |
| «El monto cubierto por el seguro debe estar entre 0 y el total de la venta.» / «Selecciona una aseguradora para aplicar la cobertura del seguro.» / «La aseguradora seleccionada no existe o fue eliminada.» | Al cobrar | Mensajes del cobro con seguro; hoy el bloque de aseguradora está oculto en el terminal y no deberías verlos | Si los ves, avisa a soporte. (No verificado en la aplicación) |
| «Esta sucursal no tiene una serie de boletas activa. Configúrala en SUNAT > Series Fiscales para poder cobrar.» | Al cobrar | La sucursal no tiene una serie de Boleta con estado «Activa» | Pide al administrador que cree o active la serie (sección 3.3). (No verificado en la aplicación) |
| «Tu empresa no tiene RUC registrado, por lo que no puede emitir comprobantes. Complétalo en Administración > Empresa para poder cobrar.» | Al cobrar | La empresa no tiene RUC | El administrador debe registrarlo en **Empresa**. (No verificado en la aplicación) |
| «No se pudo determinar el contexto de sucursal asignada del usuario.» | En el terminal | Tu sesión no trae sucursal | Cierra sesión y entra de nuevo; si persiste, el administrador debe revisar tu sucursal base. (No verificado en la aplicación) |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Al cobrar | Otra venta o un ajuste cambió el stock del mismo lote | Recarga y repite. (No verificado en la aplicación) |
| «Ocurrió un error al procesar la venta.» | Al cobrar, guardar en espera o actualizar | Mensaje genérico; ver la lista de causas siguiente | Revisa las causas y reintenta. (No verificado en la aplicación) |
| «Esta venta ya no está en espera; te llevamos a su detalle.» | Al retomar una venta en espera | Otra persona ya la cobró o la anuló | Revisa el estado en el detalle. (No verificado en la aplicación) |
| «Error al cargar la venta» | Al retomar o abrir el detalle de una venta | La venta no existe o es de otra sucursal (solo ves ventas de tu sucursal base) | Vuelve a **Ventas**. (No verificado en la aplicación) |
| «Error al buscar ventas» | Búsqueda o exportación de ventas | Falló la búsqueda | Inténtalo de nuevo. (No verificado en la aplicación) |

**Qué revisar cuando ves «Ocurrió un error al procesar la venta.»** Este texto aparece para todo lo que el terminal no traduce. Las causas más comunes, por orden:

1. **Stock insuficiente**: otro cajero vendió del mismo lote entre que lo agregaste y cobraste (`Venta.Stock`). Recarga el catálogo y vuelve a agregar el producto.
2. **Lote en cuarentena** (`Venta.Cuarentena`): el lote fue retenido; quítalo y elige otro.
3. **El lote ya no existe en tu sucursal** (`Venta.Lote`) o la presentación elegida ya no corresponde al producto (`Venta.Presentacion`).
4. **Pagos que no cuadran** (`Venta.Pagos`): la suma no cubre el total, no hay pagos o hay vuelto sin efectivo.
5. **Tu caja no está abierta** o la sesión ya se cerró (`Venta.Caja`).
6. **La venta en espera ya no está pendiente** o es de otra sucursal (`Venta.Completar`, `Venta.Actualizar`).
7. **Te falta un permiso** de `Ventas` (`Crear` para vender; `Actualizar` para cobrar una venta en espera): el servidor rechaza con 403 y el terminal solo muestra el genérico.
8. **El cliente (paciente) ya no existe** (`Paciente.NotFound`).
9. **Sin conexión** o el servidor no respondió.

Si después de revisar todo sigue igual, anota la hora y lo que estabas vendiendo y avisa al administrador (sección 4).

### 3.13 Anulación de una venta

El botón **Anular Venta** del detalle de la venta aparece cuando la venta está «Completada» o «Pendiente», no tiene devoluciones y tu rol tiene `Ventas › Anular`.

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «El motivo es obligatorio.» | Bajo «Motivo de Anulación» | Dejaste el motivo vacío | Escribe el motivo. Si escribes más de 255 caracteres, **no aparece ningún mensaje** y el botón no hace nada: acórtalo. (No verificado en la aplicación) |
| «Esta venta tiene devoluciones registradas y ya no puede anularse. Gestiona los cambios desde Devoluciones.» | Aviso inferior | La venta ya tuvo una devolución (parcial o total). No existe una pantalla «Devoluciones» en el menú: las devoluciones se hacen desde el detalle de la venta | No se puede anular. Si queda algo por devolver, usa **Devolver**. (No verificado en la aplicación) |
| «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» | Aviso inferior | Para anular una venta ya cobrada, el dinero se registra en tu caja | Abre tu caja en **Sesiones de Caja** y vuelve a anular. Una venta «Pendiente» (en espera) se anula sin caja. (No verificado en la aplicación) |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Aviso inferior | Alguien cambió la venta al mismo tiempo | Recarga y revisa el estado de la venta. (No verificado en la aplicación) |
| «Error al anular la venta.» | Aviso inferior | Genérico. Causas: la venta ya está anulada, es de otra sucursal, el motivo era solo espacios, o no tienes el permiso `Ventas › Anular` | Recarga el detalle y mira el estado; revisa tus permisos. (No verificado en la aplicación) |
| «Error al cargar la venta» | Al abrir el detalle | La venta no existe o es de otra sucursal | Vuelve a **Ventas**. (No verificado en la aplicación) |

### 3.14 Devolución de una venta

El botón **Devolver** aparece si la venta está «Completada», tiene comprobante, queda algo por devolver y tu rol tiene `Devoluciones › Procesar devolución`. Ningún rol trae este permiso por defecto: el administrador debe dárselo.

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «Ingresa al menos una cantidad a devolver.» | Aviso al pulsar **Procesar devolución** | Todas las cantidades están en 0 | Escribe una cantidad o usa **Devolver todo lo pendiente**. (No verificado en la aplicación) |
| «El sustento es obligatorio.» | Bajo «Sustento» | Está vacío (si escribes solo espacios no aparece mensaje y el botón no hace nada) | Escribe el sustento. (No verificado en la aplicación) |
| «Necesitas una caja abierta para procesar la devolución.» | Aviso inferior | El reembolso sale de tu caja y no tienes una abierta en esta sucursal | Abre tu caja. (No verificado en la aplicación) |
| «Solo puedes devolver ventas de tu propia sucursal.» | Aviso inferior | La venta es de otra sucursal | Pide que lo haga alguien de esa sucursal. (No verificado en la aplicación) |
| «Solo se pueden devolver ventas completadas.» | Aviso inferior | La venta está en espera, anulada o ya totalmente devuelta | Revisa el estado de la venta. (No verificado en la aplicación) |
| «La venta no tiene boleta ni factura emitida que acreditar.» | Aviso inferior | No hay comprobante al que emitir la nota de crédito | Avisa a soporte. (No verificado en la aplicación) |
| «La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales.» | Aviso inferior | Falta una serie «Nota de Crédito» con estado «Activa» en la sucursal de la venta | El administrador debe crearla (sección 3.3 y capítulo 2). (No verificado en la aplicación) |
| «La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.» | Aviso inferior | Otra devolución ya se llevó parte de esa cantidad | Cierra el cuadro, recarga el detalle y repite. (No verificado en la aplicación) |
| «Una de las líneas ya no pertenece a la venta.» / «La venta ya no existe.» | Aviso inferior | La venta cambió o fue eliminada mientras tenías el cuadro abierto | Recarga el detalle. (No verificado en la aplicación) |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Aviso inferior | Alguien cambió la venta o la caja al mismo tiempo | Recarga y repite. (No verificado en la aplicación) |
| «Error al procesar la devolución.» | Aviso inferior | Genérico: motivo SUNAT o sustento inválidos (el sustento admite hasta 255 caracteres), cantidad cero, falta el permiso `Devoluciones › Procesar devolución` o falló el servidor | Revisa los datos y los permisos. (No verificado en la aplicación) |

### 3.15 Dashboard (Inicio)

| Mensaje exacto | Dónde aparece | Qué significa | Cómo resolverlo |
|---|---|---|---|
| «No se pudo cargar esta sección.» (con **Reintentar**) | Dentro de una tarjeta del Inicio | Esa sección falló y las demás siguen visibles. Causa frecuente: el reloj de tu equipo está desfasado más de 36 horas; el servidor rechaza el día «hoy» en las secciones de ventas, inventario y devoluciones | Haz clic en **Reintentar**; revisa la fecha y hora de tu computadora. (No verificado en la aplicación) |
| «No se pudieron cargar tus permisos.» (con **Reintentar**) | Cabecera del Inicio | No se pudieron leer tus permisos | Haz clic en **Reintentar** o recarga. (No verificado en la aplicación) |
| «Tu usuario no tiene indicadores disponibles aquí. Usa el menú lateral para ir a tus módulos.» | Cabecera del Inicio | Tu rol no tiene `Leer` en Ventas, Sesiones de Caja, Inventario, DIGEMID ni Devoluciones | Pide los permisos al administrador, o usa el menú. (No verificado en la aplicación) |

## 4. Cuándo pedir ayuda al administrador o a soporte

**Pide ayuda al administrador de tu botica** (la persona con el rol «Admin») cuando el problema se arregla desde SOFIA con un permiso o un dato de configuración:

- No ves una opción o un botón, o te sale «No tienes permisos para acceder a esta pantalla.» (permisos del rol, sección 2.1).
- Tu cuenta está bloqueada, desactivada o necesitas **Resetear Intentos Fallidos** o **Forzar Cambio de Contraseña** (capítulo 3).
- Tu sucursal base o tus sucursales asignadas no son las correctas.
- Falta la serie de boletas o de notas de crédito de tu sucursal, o un producto no tiene precio (capítulos 2 y 5).
- Un registro no se deja eliminar porque otro lo usa (sucursal, unidad, laboratorio, medicamento…): el administrador sabe qué relación quitar.

**Pide ayuda a soporte** (el contacto que te dio tu proveedor de SOFIA) cuando el administrador no puede resolverlo desde la aplicación:

- «La suscripción de tu empresa no está activa o ha vencido. Comunícate con soporte para reactivarla.»
- «Ya existe una empresa registrada con este RUC» y es el RUC de tu empresa.
- No puedes cerrar una caja y el estado sigue «Abierta» (sección 2.5).
- Un empleado se eliminó con su cuenta y el nombre de usuario quedó ocupado (sección 3.4).
- Un mismo error se repite aunque recargues, cierres sesión y vuelvas a entrar, o ves «Referencia:» seguida de un código en el aviso.
- Ves un texto raro sin traducir (por ejemplo `cuentas.detail.load-error`).

**Qué datos dar** (sin contraseñas):

| Dato | Cómo obtenerlo |
|---|---|
| Qué pantalla estabas usando y qué hacías | Por ejemplo: «Ventas › Nueva Venta POS, al hacer clic en **Procesar Venta**» |
| Hora exacta y fecha | Hora de tu reloj, con la fecha |
| El mensaje exacto | Cópialo tal como se ve, con comillas y puntos. Si el aviso tiene «Referencia:», copia ese código con el botón de copiar |
| Tu usuario, tu empresa y tu sucursal | Tu nombre de usuario (no tu contraseña), la empresa y la sucursal que se ven en el menú de usuario |
| El número de venta o de comprobante | El «Código de Venta» de la lista de Ventas (tiene la forma prefijo de la serie, guion y correlativo de 8 dígitos) o el «Nº Comprobante» del detalle |
| La versión de SOFIA | En el menú de usuario, «Versión: 1.0.0» |
| Una captura de pantalla | Si puedes, incluye el aviso completo y la pantalla donde apareció |

**Nunca compartas tu contraseña** ni el enlace de recuperación del correo, ni siquiera con soporte: nadie de SOFIA te los pedirá. Si crees que alguien conoce tu contraseña, cámbiala desde el menú de usuario › **Cambiar contraseña**.

## 5. Índice alfabético de mensajes

Cada línea lleva el apartado de la sección 3 donde está la fila del mensaje (haz clic en el número). Los mensajes largos se cortan con «…» después de sus primeras palabras. Los mensajes con «'…'» llevan en su lugar el nombre de un campo.

- «Algunas filas tienen datos inválidos. Vuelve a subir el archivo…» — [3.8]
- «Confirmas que revisaste la receta física del cliente…» — [3.12]
- «Credenciales inválidas. Verifica tu usuario y contraseña.» — [3.1]
- «Debes aperturar la caja del turno antes de procesar una venta.» — [3.12]
- «Demasiados intentos. Espera un minuto e inténtalo de nuevo.» — [3.1]
- «El archivo excede los límites de contenido permitidos…» — [3.8]
- «El archivo no es un Excel .xlsx válido…» — [3.8]
- «El archivo supera el máximo de 5000 filas…» — [3.8]
- «El archivo supera el tamaño máximo de 5 MB.» — [3.8]
- «El campo '…' es obligatorio» — [3.8]
- «El correlativo debe ser un número entero mayor o igual a 0.» — [3.3]
- «El enlace no es válido o ya expiró. Solicita uno nuevo.» — [3.1]
- «El formato de '…' no es válido (usa DD/MM/YYYY)» — [3.8]
- «El medicamento detectado no tiene stock en esta sucursal.» — [3.12]
- «El médico seleccionado no existe. Vuelve a seleccionarlo.» — [3.10]
- «El monto cubierto por el seguro debe estar entre 0 y el total de la venta.» — [3.12]
- «El motivo es obligatorio.» — [3.13]
- «El número no corresponde al tipo de documento…» — [3.10]
- «El paciente seleccionado no existe. Vuelve a seleccionarlo.» — [3.10]
- «El prefijo debe tener 4 caracteres en mayúsculas…» — [3.3]
- «El producto seleccionado ya no existe.» — [3.7]
- «El proveedor ya no existe.» — [3.7]
- «El registro hace referencia a datos que ya no existen o todavía está en uso.» — [3.0]
- «El sustento es obligatorio.» — [3.14]
- «El usuario ya está en uso. Elige otro.» — [3.1]
- «El valor '…' del campo '…' coincide con varios registros del sistema» — [3.8]
- «El valor '…' del campo '…' está duplicado en el archivo» — [3.8]
- «El valor '…' del campo '…' no existe en el sistema» — [3.8]
- «El valor '…' del campo '…' ya existe en el sistema» — [3.8]
- «El vuelto no puede ser mayor que el efectivo recibido…» — [3.12]
- «Error al actualizar la empresa» — [3.2]
- «Error al actualizar la receta» — [3.10]
- «Error al ajustar la cantidad» — [3.9]
- «Error al anular la venta.» — [3.13]
- «Error al aperturar la sesión» — [3.11]
- «Error al asignar la sucursal» — [3.6]
- «Error al buscar aseguradoras» — [3.7]
- «Error al buscar cuentas» — [3.6]
- «Error al buscar empleados» — [3.4]
- «Error al buscar empresas» — [3.2]
- «Error al buscar en el catálogo DIGEMID» — [3.7]
- «Error al buscar formulaciones clínicas» — [3.7]
- «Error al buscar ingredientes activos» — [3.7]
- «Error al buscar jerarquías de unidades» — [3.7]
- «Error al buscar laboratorios» — [3.7]
- «Error al buscar lotes» — [3.9]
- «Error al buscar medicamentos» — [3.7]
- «Error al buscar pacientes» — [3.10]
- «Error al buscar presentaciones de venta» — [3.7]
- «Error al buscar profesionales de salud» — [3.10]
- «Error al buscar proveedores» — [3.7]
- «Error al buscar recetas médicas» — [3.10]
- «Error al buscar roles» — [3.5]
- «Error al buscar series fiscales» — [3.3]
- «Error al buscar sesiones de caja» — [3.11]
- «Error al buscar sucursales» — [3.2]
- «Error al buscar unidades de medida» — [3.7]
- «Error al buscar ventas» — [3.12]
- «Error al cargar el empleado» — [3.4]
- «Error al cargar el ingrediente activo» — [3.7]
- «Error al cargar el laboratorio» — [3.7]
- «Error al cargar el lote» — [3.9]
- «Error al cargar el medicamento» — [3.7]
- «Error al cargar el paciente» — [3.10]
- «Error al cargar el producto DIGEMID» — [3.7]
- «Error al cargar el profesional de salud» — [3.10]
- «Error al cargar el proveedor» — [3.7]
- «Error al cargar el registro de stock» — [3.9]
- «Error al cargar el rol» — [3.5]
- «Error al cargar la aseguradora» — [3.7]
- «Error al cargar la cuenta» — [3.6]
- «Error al cargar la empresa» — [3.2]
- «Error al cargar la formulación clínica» — [3.7]
- «Error al cargar la jerarquía» — [3.7]
- «Error al cargar la presentación de venta» — [3.7]
- «Error al cargar la receta» — [3.10]
- «Error al cargar la serie fiscal» — [3.3]
- «Error al cargar la sesión de caja» — [3.11]
- «Error al cargar la sucursal» — [3.2]
- «Error al cargar la unidad de medida» — [3.7]
- «Error al cargar la venta» — [3.12], [3.13]
- «Error al cargar las condiciones de venta» — [3.7]
- «Error al cargar las presentaciones de venta» — [3.7]
- «Error al cargar las sucursales» — [3.5]
- «Error al cargar los permisos» — [3.5]
- «Error al cargar los precios del proveedor» — [3.7]
- «Error al cerrar la sesión» — [3.11]
- «Error al consultar el stock» — [3.9]
- «Error al crear la receta» — [3.10]
- «Error al descargar la plantilla. Intenta de nuevo.» — [3.8]
- «Error al eliminar el empleado» — [3.4]
- «Error al eliminar el ingrediente activo» — [3.7]
- «Error al eliminar el laboratorio» — [3.7]
- «Error al eliminar el lote» — [3.9]
- «Error al eliminar el medicamento» — [3.7]
- «Error al eliminar el paciente» — [3.10]
- «Error al eliminar el profesional de salud» — [3.10]
- «Error al eliminar el proveedor» — [3.7]
- «Error al eliminar el rol» — [3.5]
- «Error al eliminar la aseguradora» — [3.7]
- «Error al eliminar la cuenta» — [3.6]
- «Error al eliminar la formulación clínica» — [3.7]
- «Error al eliminar la jerarquía» — [3.7]
- «Error al eliminar la presentación de venta» — [3.7]
- «Error al eliminar la receta» — [3.10]
- «Error al eliminar la serie fiscal» — [3.3]
- «Error al eliminar la sucursal» — [3.2], [3.6]
- «Error al eliminar la unidad de medida» — [3.7]
- «Error al exportar las series fiscales» — [3.3]
- «Error al guardar el empleado» — [3.4]
- «Error al guardar el ingrediente activo» — [3.7]
- «Error al guardar el laboratorio» — [3.7]
- «Error al guardar el lote» — [3.9]
- «Error al guardar el medicamento» — [3.7]
- «Error al guardar el paciente» — [3.10]
- «Error al guardar el profesional de salud» — [3.10]
- «Error al guardar el proveedor» — [3.7]
- «Error al guardar el rol» — [3.5]
- «Error al guardar la aseguradora» — [3.7]
- «Error al guardar la cuenta» — [3.6]
- «Error al guardar la formulación clínica» — [3.7]
- «Error al guardar la jerarquía» — [3.7]
- «Error al guardar la presentación de venta» — [3.7]
- «Error al guardar la serie fiscal» — [3.3]
- «Error al guardar la sucursal» — [3.2]
- «Error al guardar la unidad de medida» — [3.7]
- «Error al guardar los registros» — [3.8]
- «Error al interpretar la receta. Intenta nuevamente.» — [3.10]
- «Error al procesar el archivo» — [3.8]
- «Error al procesar la devolución.» — [3.14]
- «Error al registrar el precio» — [3.7]
- «Error al registrar el stock» — [3.9]
- «Error al registrar la empresa» — [3.1]
- «Esta sucursal no tiene una serie de boletas activa…» — [3.12]
- «Esta venta tiene devoluciones registradas y ya no puede anularse…» — [3.13]
- «Esta venta ya no está en espera; te llevamos a su detalle.» — [3.12]
- «Este campo es obligatorio» — [3.2]
- «Este enlace de recuperación está incompleto o ya fue usado…» — [3.1]
- «Este producto no tiene precio de venta configurado en el catálogo…» — [3.12]
- «Este producto ya tiene una conversión entre esas unidades.» — [3.7]
- «Hay productos de receta retenida o estupefacientes…» — [3.12]
- «Hay productos que requieren una receta registrada…» — [3.12]
- «Ingresa al menos una cantidad a devolver.» — [3.14]
- «Ingresa el monto declarado al cierre» — [3.11]
- «Ingresa el N° de operación para poder cobrar.» — [3.12]
- «Ingresa el número de operación de cada pago que no sea en efectivo.» — [3.12]
- «Ingresa la dirección de la sede principal (máximo 255 caracteres)» — [3.1]
- «Ingresa un correo electrónico válido.» — [3.4]
- «Ingresa un monto válido» — [3.11]
- «Ingresa una cantidad válida para agregar el producto.» — [3.12]
- «La aseguradora seleccionada no existe o fue eliminada.» — [3.12]
- «La caja seleccionada pertenece a otro cajero o sucursal…» — [3.12]
- «La cantidad supera lo que queda por devolver en alguna línea…» — [3.14]
- «La contraseña actual es incorrecta» — [3.1]
- «La cuenta no existe» — [3.6]
- «La fecha de expedición no puede ser posterior a hoy.» — [3.10]
- «La longitud máxima es de 100 caracteres» — [3.6]
- «La receta médica no pertenece al cliente de la venta.» — [3.12]
- «La receta médica seleccionada no existe o fue eliminada.» — [3.12]
- «La receta médica ya fue dispensada el máximo de veces permitido.» — [3.12]
- «La receta seleccionada no tiene número de receta…» — [3.12]
- «La receta ya no existe.» — [3.10]
- «La serie fiscal no existe.» — [3.3]
- «La serie ya emitió comprobantes: solo puedes cambiar su estado.» — [3.3]
- «La sucursal no existe» — [3.6]
- «La sucursal no tiene una serie de nota de crédito activa…» — [3.14]
- «La sucursal seleccionada no existe.» — [3.3]
- «La sucursal ya tiene una serie activa de este tipo…» — [3.3]
- «La suscripción de tu empresa no está activa o ha vencido…» — [3.1]
- «La venta no tiene boleta ni factura emitida que acreditar.» — [3.14]
- «La venta ya no existe.» — [3.14]
- «Marca "Receta verificada" para confirmar que revisaste la receta…» — [3.12]
- «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» — [3.13]
- «Necesitas tener tu propia caja abierta en esta sucursal para cobrar la venta.» — [3.12]
- «Necesitas una caja abierta para procesar la devolución.» — [3.14]
- «No hay filas válidas para guardar» — [3.8]
- «No pudimos conectar con el servidor…» — [3.1]
- «No puedes eliminar esta receta porque ya se usó en ventas.» — [3.10]
- «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.» — [3.2]
- «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» — [3.7]
- «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.» — [3.7]
- «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» — [3.10]
- «No puedes eliminar este profesional de salud porque tiene recetas asociadas.» — [3.10]
- «No puedes eliminar este proveedor porque tiene precios de compra registrados.» — [3.7]
- «No se detectaron medicamentos en la imagen…» — [3.10]
- «No se pudieron cargar las presentaciones del producto…» — [3.12]
- «No se pudieron cargar las sucursales» — [3.3]
- «No se pudieron cargar los lotes disponibles…» — [3.12]
- «No se pudieron cargar tus permisos.» — [3.15]
- «No se pudieron cargar tus permisos. Recarga la página o vuelve a iniciar sesión.» — [3.1]
- «No se pudieron cargar tus sucursales» — [3.9]
- «No se pudo aperturar la caja» — [3.11]
- «No se pudo cambiar la contraseña. Inténtalo de nuevo.» — [3.1]
- «No se pudo cargar el catálogo de medicamentos…» — [3.12]
- «No se pudo cargar esta sección.» — [3.15]
- «No se pudo determinar el contexto de sucursal asignada del usuario.» — [3.12]
- «No se pudo procesar la solicitud. Inténtalo de nuevo.» — [3.1]
- «No se pudo restablecer la contraseña. Inténtalo de nuevo.» — [3.1]
- «No se puede eliminar el laboratorio porque tiene medicamentos asociados.» — [3.7]
- «No se puede eliminar la jerarquía porque está en uso.» — [3.7]
- «No se puede eliminar la presentación porque tiene ventas registradas.» — [3.7]
- «No se puede eliminar la unidad de medida porque está asociada…» — [3.7]
- «No se puede eliminar un empleado que gerencia una sucursal…» — [3.4]
- «No se puede eliminar un lote con stock o ventas registradas.» — [3.9]
- «No se puede eliminar una serie que ya emitió comprobantes…» — [3.3]
- «No se pueden ampliar los permisos de un rol que la propia cuenta tiene asignado» — [3.5]
- «No tienes permiso para operar el stock de esta sucursal» — [3.9]
- «No tienes permisos para acceder a esta pantalla.» — [3.1]
- «Ocurrió un error al procesar la venta.» — [3.12]
- «Otra operación modificó estos datos al mismo tiempo…» — [3.0], [3.12], [3.13], [3.14]
- «Revisa los datos ingresados de la receta.» — [3.10]
- «Se guardaron 0 registros correctamente» — [3.8]
- «Selecciona un tipo de documento válido.» — [3.10]
- «Selecciona una aseguradora para aplicar la cobertura del seguro.» — [3.12]
- «Solo el cajero que abrió la caja o un administrador puede cerrarla.» — [3.11]
- «Solo puedes devolver ventas de tu propia sucursal.» — [3.14]
- «Solo se pueden devolver ventas completadas.» — [3.14]
- «Solo un administrador puede asignar o quitar el rol Admin» — [3.6]
- «Solo un administrador puede modificar o eliminar la cuenta de un administrador» — [3.6]
- «Stock máximo disponible alcanzado ({{stock}} un.).» — [3.12]
- «Tu empresa no tiene RUC registrado…» — [3.12]
- «Tu usuario no tiene indicadores disponibles aquí…» — [3.15]
- «Una cuenta no puede modificar sus propios roles» — [3.6]
- «Una de las líneas ya no pertenece a la venta.» — [3.14]
- «Uno de los lotes está vencido y no se puede vender…» — [3.12]
- «Uno de los productos requiere receta médica…» — [3.12]
- «Uno de los valores está fuera del rango permitido.» — [3.0]
- «Ya existe otra serie de este tipo de comprobante con ese prefijo.» — [3.3]
- «Ya existe un empleado con esa licencia profesional.» — [3.4]
- «Ya existe un empleado con ese correo electrónico en tu empresa.» — [3.4]
- «Ya existe un ingrediente activo con esa denominación DCI.» — [3.7]
- «Ya existe un laboratorio con ese código identificador.» — [3.7]
- «Ya existe un laboratorio con ese nombre de compañía.» — [3.7]
- «Ya existe un lote con ese número para el producto seleccionado» — [3.9]
- «Ya existe un medicamento con ese código nacional.» — [3.7]
- «Ya existe un paciente con ese documento de identidad.» — [3.10]
- «Ya existe un profesional de salud con ese número de registro.» — [3.10]
- «Ya existe un proveedor con esa razón social.» — [3.7]
- «Ya existe un proveedor con ese identificador tributario.» — [3.7]
- «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» — [3.0]
- «Ya existe una empresa registrada con este RUC» — [3.1]
- «Ya existe una formulación con los datos ingresados» — [3.7]
- «Ya existe una presentación de venta para esa unidad en este producto.» — [3.7]
- «Ya existe una sucursal con ese número de licencia.» — [3.2]
- «Ya existe una unidad de medida con ese código.» — [3.7]
- «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» — [3.11]

Mensajes de carga masiva con un campo entre comillas simples (por ejemplo «'…' debe ser un número entero» o «'…' no puede superar los N caracteres»): están todos en [3.8].

[3.0]: #30-avisos-de-base-de-datos-y-de-simultaneidad
[3.1]: #31-acceso-y-sesión
[3.2]: #32-empresa-y-sucursales
[3.3]: #33-series-fiscales
[3.4]: #34-empleados
[3.5]: #35-roles-y-permisos
[3.6]: #36-cuentas-de-usuario
[3.7]: #37-catálogos-unidades-laboratorios-ingredientes-digemid-productos-proveedores-y-aseguradoras
[3.8]: #38-carga-masiva-y-exportación-a-excel
[3.9]: #39-inventario-lotes-y-stock-por-sucursal
[3.10]: #310-pacientes-profesionales-de-salud-y-recetas
[3.11]: #311-caja-apertura-y-cierre
[3.12]: #312-venta-y-cobro
[3.13]: #313-anulación-de-una-venta
[3.14]: #314-devolución-de-una-venta
[3.15]: #315-dashboard-inicio

## 6. Pendientes y «No verificado»

En curso (sin commitear): ninguno. Los dos repositorios estaban sin cambios locales al empezar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`).

Este capítulo no se recorrió en la aplicación, por decisión del encargo (el navegador lo usaba otro agente). De las 206 filas de mensaje de la sección 3, **44 se vieron en pantalla** en los capítulos 1 a 5 (marcadas «Verificado en el capítulo N») y **162 son «No verificado en la aplicación»**: su texto sale del catálogo de códigos de error (`21-catalogo-codigos-error.md`) y de los inventarios de pantallas, pero nadie lo provocó en un recorrido real.

| # | Qué | Motivo |
|---|---|---|
| 1 | Mensajes de límite de peticiones («Demasiados intentos…») y de caída del servidor («No pudimos conectar con el servidor…») | Habría bloqueado la cuenta o el servidor de prueba |
| 2 | Mensaje de suscripción vencida y bloqueo por 5 intentos (15 minutos) en la cuenta del cajero real | Se probó el bloqueo con una cuenta de prueba en el capítulo 3; la suscripción vencida no se puede provocar desde la pantalla |
| 3 | Todos los mensajes «Error al buscar…», «Error al cargar…», «Error al eliminar…» y «Error al guardar…» genéricos | Son fallos del servidor o de la red que no se pueden provocar desde la pantalla. Las **causas** listadas salen de las columnas «Cuándo sale» del catálogo 21 |
| 4 | Mensajes de «en uso» al eliminar (unidad, laboratorio, ingrediente, medicamento, proveedor, paciente, profesional, receta, lote, presentación, jerarquía) | Faltaba que existiera el registro que los usa al escribirse los capítulos 4 y 5; el catálogo 21 los respalda con su código 409 |
| 5 | Todo lo del cobro (stock agotado entre medias, lote vencido o en cuarentena, recetas, vuelto, números de operación), anulación y devolución | Los capítulos 6 a 10 los escribe otro agente y no estaban publicados al redactarse este capítulo. Los textos vienen de 19f §3.12, §8 y §9 y de las reglas RN-VEN, RN-DEV y RN-CAJ |
| 6 | Cierre de caja con efectivo esperado negativo | Está en RN-CAJ-016 (regla) y el catálogo 21; no se provocó |
| 7 | «Error al interpretar la receta. Intenta nuevamente.» y sus causas | Requiere el servicio de IA y una imagen; viene de 19e §5.3 |
| 8 | Fecha del día siguiente en las exportaciones de Sesiones de Caja y Series Fiscales | El navegador integrado no abre los archivos descargados; viene de RN-CAJ-024 y del capítulo 2 |
| 9 | Pantalla en vista angosta, Ctrl + clic, y que el idioma elegido no se conserva al recargar | Ver los «No verificado» del capítulo 1 |
| 10 | Mensajes de los bloques de aseguradora del terminal | El bloque está oculto en el terminal (19f §3.8); no se pueden ver |
| 11 | Autoguardado del borrador del carrito (12 horas) y botón de cliente que no se quita | Viene de 19f §3.9 y §3.4 (derivados de la lectura de código) |

Hallazgos que conviene conocer:

- Muchos mensajes de **eliminar** se deciden por el estado HTTP 409, no por el código: cualquier 409 al eliminar muestra el mismo texto de «en uso», aunque la causa sea otra (catálogo 21, sección 1.3).
- Los textos de carga masiva y los de «Error al…» no dicen la causa; este capítulo la reconstruye desde el catálogo.
- Algunos mensajes remiten a pantallas que no existen: «Gestiona los cambios desde Devoluciones.» (no hay pantalla de Devoluciones; se devuelve desde el detalle de la venta) y «Configúrala en SUNAT > Series Fiscales» (en el menú es el grupo **SUNAT** › **Series Fiscales**, esa sí existe).
- La clave sin traducir `cuentas.detail.load-error` y `digemidCatalogo.detail.load-error` puede verse en lugar del texto en algunas cargas directas (capítulos 3 y 4).
- La pantalla de nueva cuenta no avisa si la contraseña tiene menos de 8 caracteres.

## 7. Tabla de cobertura

### 7.1 Mensajes de las tablas «Mensajes al usuario» de 19a a 19f

| Inventario | Mensajes de error y de aviso | Dónde quedaron |
|---|---|---|
| 19a §1.1 (inicio de sesión) | 4 mensajes de la alerta roja | 3.1; situación en 2.3 |
| 19a §1.2 (registro de empresa) | RUC duplicado, usuario en uso, dirección de la sede, «Error al registrar la empresa» | 3.1 |
| 19a §1.3 y §1.4 (recuperar y restablecer contraseña) | Solicitud, enlace incompleto, enlace inválido, límite, errores genéricos | 3.1; situación en 2.3 |
| 19a §2 (cambiar contraseña) | Contraseña actual incorrecta y error genérico | 3.1 |
| 19a §3.3, §5.3 (permisos y dashboard) | «No se pudieron cargar tus permisos…», «No tienes permisos…», «No se pudo cargar esta sección.» | 3.1, 3.15, 2.1 |
| 19a §5.1 y §5.4 (sesión vencida) | Sin texto: te lleva al inicio de sesión | 2.2 |
| 19a §6.5 y §8.9 (carga masiva) | Errores de archivo y 17 errores por fila | 3.8 |
| 19a §8.7 (`errors.*`) | Tres avisos de base de datos y simultaneidad | 3.0 |
| 19b §1 (Empresa) | Buscar, actualizar, cargar | 3.2 |
| 19b §2 (Sucursales) | Duplicado, genérico, en uso, eliminar, cargar, buscar | 3.2 |
| 19b §3 (Empleados) | Correo, licencia, gerente, genéricos | 3.4 |
| 19b §4 (Roles) | Guardar, eliminar, permisos propios, cargar | 3.5 |
| 19b §5 (Cuentas) | Siete códigos traducidos y genéricos | 3.6 |
| 19b §6 (Series fiscales) | Prefijo, activa, con comprobantes, no existe, genéricos | 3.3 |
| 19c §1 a §5 (Unidades, Jerarquías, Laboratorios, Ingredientes, DIGEMID) | Duplicados, en uso y genéricos | 3.7 |
| 19d §1 a §5 (Medicamentos, Presentaciones, Formulaciones, Proveedores, Aseguradoras) y precio de proveedor | Duplicados, en uso, genéricos, precio | 3.7 |
| 19e §1 y §2 (Lotes y Stock) | Conflicto, en uso, permiso de sucursal, genéricos | 3.9 |
| 19e §3 y §4 (Pacientes y Profesionales) | Documento, formato, registro, en uso, genéricos | 3.10 |
| 19e §5 (Recetas y escáner) | Paciente, médico, fecha, datos, en uso, IA | 3.10 |
| 19f §1 y §2 (Sesiones de caja y diálogo de apertura) | Ya abierta, no propia, genéricos, montos | 3.11; situación en 2.5 |
| 19f §3 (terminal de venta): mapa de 24 códigos y 13 avisos | Todos | 3.12 |
| 19f §6 y §7 (Ventas y detalle) | Buscar, cargar | 3.12, 3.13 |
| 19f §8 (anulación) | Devoluciones, caja propia, genérico, motivo | 3.13 |
| 19f §9 (devolución) | 11 códigos y genérico | 3.14 |

Avisos de **éxito** y mensajes informativos de las tablas «Mensajes al usuario» que **no** se incluyen en este capítulo porque no son problemas (aparecen en el capítulo donde se hace la tarea): «Empresa registrada correctamente. ¡Bienvenido!», «Contraseña actualizada. Inicia sesión con tu nueva contraseña.», «Contraseña restablecida. Inicia sesión con tu nueva contraseña.», «Si el usuario existe y tiene un correo registrado, recibirás un enlace para restablecer tu contraseña. Revisa tu bandeja de entrada: el enlace es válido por 1 hora.», «Empresa actualizada correctamente», «Sucursal guardada correctamente», «Sucursal eliminada correctamente», «Sucursal asignada correctamente», «Empleado guardado correctamente», «Empleado eliminado correctamente», «Rol guardado correctamente», «Rol eliminado correctamente», «Cuenta guardada correctamente», «Cuenta eliminada correctamente», «Serie fiscal guardada correctamente», «Serie fiscal eliminada correctamente», los avisos «… guardado correctamente» y «… eliminado correctamente» de cada catálogo, «Precio registrado correctamente», «Stock registrado correctamente», «Cantidad ajustada correctamente», «Receta creada correctamente», «Receta actualizada correctamente», «Receta eliminada correctamente», «Sesión aperturada correctamente», «Caja aperturada correctamente», «Sesión cerrada correctamente», «Venta anulada correctamente.», «Devolución procesada y nota de crédito emitida.», «Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva.» y el banner «Estás retomando una venta en espera: puedes agregar, quitar o modificar productos antes de cobrar.». El aviso «Referencia: …» se explica en la sección 1.

### 7.2 Tareas y situaciones pedidas en el encargo

| Encargo | Apartado |
|---|---|
| Cómo usar este capítulo | 1 |
| No veo una opción del menú o un botón (permisos por pantalla) | 2.1 |
| La pantalla me saca al inicio de sesión | 2.2 |
| No puedo entrar (bloqueada, empresa no vigente, contraseña olvidada) | 2.3 |
| Cambié el idioma y volvió a español | 2.4 |
| No puedo cerrar mi caja | 2.5 |
| El cajero no ve ventas de otra sucursal | 2.6 |
| La boleta dice «Pendiente de envío a SUNAT» | 2.7 |
| La exportación sale con la fecha del día siguiente | 2.8 |
| Otros comportamientos sorpresivos | 2.9 |
| Mensajes por tarea: acceso, empresa y sucursales, series, empleados, roles, cuentas, catálogos, carga masiva, inventario, pacientes y recetas, caja, venta y cobro, anulación, devolución, dashboard | 3.1 a 3.15 (y 3.0 para los avisos de base de datos) |
| Mensajes genéricos con causas | 3.2 a 3.15 (columna «Qué significa»); «Ocurrió un error al procesar la venta.» con lista de causas en 3.12 |
| Cuándo pedir ayuda y qué datos dar | 4 |
| Índice alfabético | 5 |

## 8. Fuentes

Documentos consultados:

- `docs/_inventario/REGLAS-AGENTES.md` y `docs/_inventario/MANUAL-AGENTES.md`.
- `docs/ola-1-fundamentos/21-catalogo-codigos-error.md` (fuente principal de los mensajes: secciones 1.3, 2, 3.1 a 3.28 y 5).
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19a` a `19f` (tablas «Mensajes al usuario», §5 de 19a para guards e interceptores, §6.5 y §8.9 de 19a para carga masiva, §3.12 de 19f para el mapa de códigos del terminal).
- `docs/ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md` (RN-VEN-001 a RN-VEN-038, RN-CMP-007), `01c3-caja-devoluciones-dashboard.md` (RN-CAJ-016, RN-CAJ-020, RN-CAJ-024, RN-DAS-001 a RN-DAS-024) y `01a-administracion-y-seguridad.md` (RN-AUT, RN-CTA).
- `docs/ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md` (§1.2, §1.3, §5 y §7).
- Capítulos 1 a 5 del manual (`capitulo-01` a `capitulo-05` en esta carpeta), de donde salen las marcas «Verificado en el capítulo N».
- Textos de la interfaz: `sofia-web-spa/public/i18n/es.json` y `public/i18n/<scope>/es.json` (se compararon con la lista de mensajes de este capítulo).

Este capítulo no incluye recorridos en la aplicación.

