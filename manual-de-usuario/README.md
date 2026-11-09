# Manual de usuario de SOFIA

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-11-04 |
| **Backend** | `sofia-apis`, rama `main`, commit `9c16f35` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `7070e1a` |
| **Método** | Integración de los capítulos publicados |

## 1. Presentación del manual

### 1.1 Qué es SOFIA

SOFIA (Sistema Optimizado Farmacéutico con Inteligencia Artificial) es un sistema en línea para boticas y farmacias del Perú. Con SOFIA registras tu empresa y sus sucursales, das de alta a tu personal y sus permisos, mantienes el catálogo de productos y el inventario por lotes, registras pacientes y recetas, y vendes en un terminal de venta con caja, boleta, anulaciones y devoluciones. La pantalla de inicio resume el día de tu botica.

### 1.2 Para quién es este manual

Está escrito para el personal de una botica que no sabe nada técnico. Hay dos lectores:

- **El administrador**, que registra la empresa y deja todo listo para vender: sucursales, series fiscales, personal, productos, lotes y stock.
- **El cajero**, que usa SOFIA cada día para abrir su caja, vender, anular, devolver y cerrar su caja.

Se habla siempre en segunda persona y con frases cortas. Cada capítulo dice al empezar para quién es.

### 1.3 Cómo está organizado

El manual tiene once capítulos, uno por archivo, en esta misma carpeta. Los primeros siete son de **puesta en marcha** y siguen el orden en que se prepara una botica; los capítulos 8 a 10 son de **uso diario**; el capítulo 11 es de **consulta** cuando algo no sale como esperabas.

| Capítulo | Tema | Lector principal |
|---|---|---|
| [1. Primeros pasos](capitulo-01-primeros-pasos.md) | Registrar la empresa, la pantalla principal, iniciar y cerrar sesión, contraseñas | Administrador y cajero |
| [2. Empresa, sucursales y series](capitulo-02-empresa-sucursales-series.md) | Datos de la empresa, sucursales, series fiscales | Administrador |
| [3. Empleados, roles y cuentas](capitulo-03-empleados-roles-cuentas.md) | Empleados, rol y permisos, cuenta de usuario | Administrador |
| [4. Catálogos básicos](capitulo-04-catalogos-basicos.md) | Unidades de medida, laboratorios, ingredientes activos, catálogo DIGEMID | Administrador |
| [5. Productos y proveedores](capitulo-05-productos-y-proveedores.md) | Medicamentos, jerarquías, presentaciones, formulaciones, proveedores, aseguradoras | Administrador |
| [6. Inventario](capitulo-06-inventario.md) | Lotes y stock por sucursal | Administrador |
| [7. Pacientes y recetas](capitulo-07-pacientes-y-recetas.md) | Pacientes, profesionales de salud, recetas y lector de recetas con IA | Administrador y cajero |
| [8. Caja y venta](capitulo-08-caja-y-venta.md) | Abrir la caja, el terminal de venta, cobrar, comprobante, venta en espera | Cajero |
| [9. Ventas, devoluciones y cierre](capitulo-09-ventas-devoluciones-y-cierre.md) | Buscar ventas, anular, devolver, cerrar la caja con arqueo | Cajero |
| [10. Inicio y roles](capitulo-10-inicio-y-roles.md) | El Inicio (dashboard) y qué ve cada rol | Administrador y cajero |
| [11. Problemas frecuentes](capitulo-11-problemas-frecuentes.md) | Situaciones sin mensaje, mensajes de error por tarea e índice alfabético | Todos |

### 1.4 Cómo leerlo

- **Si empiezas de cero**, sigue el recorrido de la sección 2: las tareas están en el orden en que una depende de la otra.
- **Si buscas una tarea concreta**, ve a la lista de la sección 4 (tareas frecuentes) y abre el capítulo y la sección que indica.
- **Si buscas un tema**, usa el índice completo de la sección 3: cada título es un enlace.
- **Si te apareció un mensaje de error**, búscalo por su texto exacto en el índice alfabético del [capítulo 11, sección 5](capitulo-11-problemas-frecuentes.md#5-índice-alfabético-de-mensajes).

Cada capítulo sigue la misma estructura: introducción, qué necesitas antes de empezar (incluidos los permisos), una sección por tarea con sus pasos, campos y errores, lo que ve un cajero frente a un administrador, y al final los pendientes, la tabla de cobertura y las fuentes.

## 2. Recorrido recomendado

### 2.1 Administrador que empieza de cero: capítulos 1 a 7

Lee los capítulos 1 a 7 **en orden**. Cada uno usa lo que dejó el anterior. Este es el orden de puesta en marcha que piden los capítulos y por qué:

| Paso | Qué dejar listo | Dónde | Por qué va en ese orden |
|---|---|---|---|
| 1 | Registrar tu empresa y entrar como administrador | [Capítulo 1, §3 Registrar tu empresa](capitulo-01-primeros-pasos.md#3-registrar-tu-empresa) | Todo lo demás cuelga de la empresa y de su administrador |
| 2 | Revisar los datos de la empresa y su RUC | [Capítulo 2, §3 Revisar y editar los datos de tu empresa](capitulo-02-empresa-sucursales-series.md#3-revisar-y-editar-los-datos-de-tu-empresa) | La empresa necesita RUC para emitir comprobantes; sin él el cobro se rechaza con «Tu empresa no tiene RUC registrado…» |
| 3 | Completar la sede principal y crear las demás sucursales | [Capítulo 2, §4 Completar la sede principal](capitulo-02-empresa-sucursales-series.md#4-completar-la-sede-principal) y [Capítulo 2, §5 Crear una segunda sucursal](capitulo-02-empresa-sucursales-series.md#5-crear-una-segunda-sucursal) | Las series, el personal, los lotes y las cajas se asignan a una sucursal |
| 4 | **Series fiscales** de cada sucursal: una de boleta y una de nota de crédito, las dos «Activa» | [Capítulo 2, §9 Series fiscales: qué son y para qué las usa SOFIA](capitulo-02-empresa-sucursales-series.md#9-series-fiscales-qué-son-y-para-qué-las-usa-sofia) y [Capítulo 2, §10 Crear una serie fiscal](capitulo-02-empresa-sucursales-series.md#10-crear-una-serie-fiscal) | **La serie fiscal va antes de vender**: sin una serie de boleta activa la sucursal no puede cobrar, y sin una de nota de crédito activa no se puede devolver |
| 5 | Empleados, el rol «Cajero» con sus permisos y la cuenta de cada cajero | [Capítulo 3, §3 Registrar empleados](capitulo-03-empleados-roles-cuentas.md#3-registrar-empleados), [Capítulo 3, §5 Crear el rol «Cajero» y marcar sus permisos](capitulo-03-empleados-roles-cuentas.md#5-crear-el-rol-cajero-y-marcar-sus-permisos) y [Capítulo 3, §7 Crear la cuenta del cajero](capitulo-03-empleados-roles-cuentas.md#7-crear-la-cuenta-del-cajero) | La cuenta necesita un empleado y un rol ya creados |
| 6 | Unidades de medida, laboratorios e ingredientes activos | [Capítulo 4, §5 Unidades de medida](capitulo-04-catalogos-basicos.md#5-unidades-de-medida), [Capítulo 4, §6 Laboratorios](capitulo-04-catalogos-basicos.md#6-laboratorios) y [Capítulo 4, §7 Ingredientes activos](capitulo-04-catalogos-basicos.md#7-ingredientes-activos) | Los medicamentos los eligen al crearse |
| 7 | Medicamentos (con su precio), jerarquías y presentaciones; proveedores si los usas | [Capítulo 5, §4 Medicamentos](capitulo-05-productos-y-proveedores.md#4-medicamentos), [Capítulo 5, §5 Jerarquías de unidades](capitulo-05-productos-y-proveedores.md#5-jerarquías-de-unidades), [Capítulo 5, §6 Presentaciones de venta](capitulo-05-productos-y-proveedores.md#6-presentaciones-de-venta) y [Capítulo 5, §8 Proveedores y precios de compra](capitulo-05-productos-y-proveedores.md#8-proveedores-y-precios-de-compra) | El terminal solo vende medicamentos con precio |
| 8 | **Lotes y stock** por sucursal | [Capítulo 6, §5 Crear un lote](capitulo-06-inventario.md#5-crear-un-lote) y [Capítulo 6, §8 Ingresar stock](capitulo-06-inventario.md#8-ingresar-stock) | **El lote y el stock van antes del terminal**: sin lotes con existencia no hay nada que agregar al carrito |
| 9 | Pacientes, profesionales de salud y recetas (solo si vendes productos con receta) | [Capítulo 7, §5 Registrar un paciente](capitulo-07-pacientes-y-recetas.md#5-registrar-un-paciente), [Capítulo 7, §9 Profesionales de salud](capitulo-07-pacientes-y-recetas.md#9-profesionales-de-salud) y [Capítulo 7, §10 Registrar una receta](capitulo-07-pacientes-y-recetas.md#10-registrar-una-receta) | La receta necesita un paciente y un médico ya registrados |

Cuando termines, tu botica está lista para vender. El uso diario está en los capítulos 8 a 10, y el capítulo 11 queda como consulta.

Dos cuidados que los capítulos señalan:

- Si cargas medicamentos desde Excel, la carga **no trae el precio**: los medicamentos quedan con el precio vacío y debes editarlos antes de venderlos ([Capítulo 5, §10.3 Carga masiva de medicamentos](capitulo-05-productos-y-proveedores.md#103-carga-masiva-de-medicamentos)).
- Cada cajero entra por primera vez con un cambio obligatorio de contraseña ([Capítulo 3, §8.1 El cambio obligatorio de contraseña](capitulo-03-empleados-roles-cuentas.md#81-el-cambio-obligatorio-de-contraseña)).

### 2.2 Cajero en su día a día: capítulos 1, 8, 9 y 10

Si eres cajero, tu administrador ya dejó lista la botica. Lee:

1. **Capítulo 1**, para iniciar y cerrar sesión, cambiar o recuperar tu contraseña y reconocer la pantalla principal: [Capítulo 1, §4 Conocer la pantalla principal](capitulo-01-primeros-pasos.md#4-conocer-la-pantalla-principal), [Capítulo 1, §5 Cerrar sesión](capitulo-01-primeros-pasos.md#5-cerrar-sesión), [Capítulo 1, §6 Iniciar sesión](capitulo-01-primeros-pasos.md#6-iniciar-sesión), [Capítulo 1, §7 Cambiar tu contraseña](capitulo-01-primeros-pasos.md#7-cambiar-tu-contraseña) y [Capítulo 1, §8 Recuperar tu contraseña olvidada](capitulo-01-primeros-pasos.md#8-recuperar-tu-contraseña-olvidada).
2. **Capítulo 8**, para abrir tu caja y vender.
3. **Capítulo 9**, para anular, devolver y cerrar tu caja.
4. **Capítulo 10**, para leer el Inicio y saber qué ves tú frente a un administrador: [Capítulo 10, §6 El Inicio como cajero](capitulo-10-inicio-y-roles.md#6-el-inicio-como-cajero).

Tu jornada, en orden:

| Momento | Qué haces | Dónde |
|---|---|---|
| Al llegar | Iniciar sesión | [Capítulo 1, §6 Iniciar sesión](capitulo-01-primeros-pasos.md#6-iniciar-sesión) |
| Antes de vender | Abrir **tu** caja (sin caja propia abierta no puedes vender) | [Capítulo 8, §5 Abrir tu caja](capitulo-08-caja-y-venta.md#5-abrir-tu-caja) |
| Durante el turno | Buscar el producto, elegir su lote y cobrar; con receta, elegir al cliente y su receta | [Capítulo 8, §8 Buscar un producto y elegir su lote y su presentación](capitulo-08-caja-y-venta.md#8-buscar-un-producto-y-elegir-su-lote-y-su-presentación), [Capítulo 8, §11 Vender medicamentos con receta](capitulo-08-caja-y-venta.md#11-vender-medicamentos-con-receta) y [Capítulo 8, §12 Cobrar la venta](capitulo-08-caja-y-venta.md#12-cobrar-la-venta) |
| Si el cliente se va un momento | Dejar la venta en espera y retomarla | [Capítulo 8, §15 Dejar una venta en espera y retomarla](capitulo-08-caja-y-venta.md#15-dejar-una-venta-en-espera-y-retomarla) |
| Si hay un error o una devolución | Anular la venta o devolver productos | [Capítulo 9, §5 Anular una venta cobrada](capitulo-09-ventas-devoluciones-y-cierre.md#5-anular-una-venta-cobrada) y [Capítulo 9, §7 Devolver productos](capitulo-09-ventas-devoluciones-y-cierre.md#7-devolver-productos) |
| Al terminar | Cerrar tu caja con arqueo | [Capítulo 9, §8 Cerrar tu caja con arqueo](capitulo-09-ventas-devoluciones-y-cierre.md#8-cerrar-tu-caja-con-arqueo) |
| Cuando lo necesites | Revisar tus cifras del día en el Inicio | [Capítulo 10, §6 El Inicio como cajero](capitulo-10-inicio-y-roles.md#6-el-inicio-como-cajero) |
| Si algo falla | Buscar el mensaje en los problemas frecuentes | [Capítulo 11, §3 Mensajes de error por tarea](capitulo-11-problemas-frecuentes.md#3-mensajes-de-error-por-tarea) |

## 3. Índice completo

Cada capítulo con sus secciones. Los enlaces llevan directo al título.

- **[Capítulo 1 — Primeros pasos](capitulo-01-primeros-pasos.md)**
  - [1. Introducción](capitulo-01-primeros-pasos.md#1-introducción)
  - [2. Antes de empezar](capitulo-01-primeros-pasos.md#2-antes-de-empezar)
  - [3. Registrar tu empresa](capitulo-01-primeros-pasos.md#3-registrar-tu-empresa)
    - [3.1 Cómo es la pantalla](capitulo-01-primeros-pasos.md#31-cómo-es-la-pantalla)
    - [3.2 Pasos](capitulo-01-primeros-pasos.md#32-pasos)
    - [3.3 Campos](capitulo-01-primeros-pasos.md#33-campos)
    - [3.4 Qué pasa al terminar](capitulo-01-primeros-pasos.md#34-qué-pasa-al-terminar)
    - [3.5 Errores que puedes ver](capitulo-01-primeros-pasos.md#35-errores-que-puedes-ver)
  - [4. Conocer la pantalla principal](capitulo-01-primeros-pasos.md#4-conocer-la-pantalla-principal)
    - [4.1 Menú lateral](capitulo-01-primeros-pasos.md#41-menú-lateral)
    - [4.2 Cabecera](capitulo-01-primeros-pasos.md#42-cabecera)
    - [4.3 Cambiar el idioma](capitulo-01-primeros-pasos.md#43-cambiar-el-idioma)
    - [4.4 Menú de usuario](capitulo-01-primeros-pasos.md#44-menú-de-usuario)
    - [4.5 El Inicio (dashboard)](capitulo-01-primeros-pasos.md#45-el-inicio-dashboard)
  - [5. Cerrar sesión](capitulo-01-primeros-pasos.md#5-cerrar-sesión)
  - [6. Iniciar sesión](capitulo-01-primeros-pasos.md#6-iniciar-sesión)
    - [6.1 Cómo es la pantalla](capitulo-01-primeros-pasos.md#61-cómo-es-la-pantalla)
    - [6.2 Pasos](capitulo-01-primeros-pasos.md#62-pasos)
    - [6.3 Campos](capitulo-01-primeros-pasos.md#63-campos)
    - [6.4 Qué pasa al terminar](capitulo-01-primeros-pasos.md#64-qué-pasa-al-terminar)
    - [6.5 Errores que puedes ver](capitulo-01-primeros-pasos.md#65-errores-que-puedes-ver)
    - [6.6 Otros enlaces de la pantalla](capitulo-01-primeros-pasos.md#66-otros-enlaces-de-la-pantalla)
  - [7. Cambiar tu contraseña](capitulo-01-primeros-pasos.md#7-cambiar-tu-contraseña)
    - [7.1 Cómo es la pantalla](capitulo-01-primeros-pasos.md#71-cómo-es-la-pantalla)
    - [7.2 Pasos](capitulo-01-primeros-pasos.md#72-pasos)
    - [7.3 Campos](capitulo-01-primeros-pasos.md#73-campos)
    - [7.4 Botones](capitulo-01-primeros-pasos.md#74-botones)
    - [7.5 Qué pasa al terminar](capitulo-01-primeros-pasos.md#75-qué-pasa-al-terminar)
    - [7.6 Errores que puedes ver](capitulo-01-primeros-pasos.md#76-errores-que-puedes-ver)
  - [8. Recuperar tu contraseña olvidada](capitulo-01-primeros-pasos.md#8-recuperar-tu-contraseña-olvidada)
    - [8.1 Pedir el enlace](capitulo-01-primeros-pasos.md#81-pedir-el-enlace)
    - [8.2 El correo que recibes](capitulo-01-primeros-pasos.md#82-el-correo-que-recibes)
    - [8.3 Crear la nueva contraseña](capitulo-01-primeros-pasos.md#83-crear-la-nueva-contraseña)
  - [9. Sesión vencida y cambio obligatorio de contraseña](capitulo-01-primeros-pasos.md#9-sesión-vencida-y-cambio-obligatorio-de-contraseña)
    - [9.1 Qué pasa cuando tu sesión vence](capitulo-01-primeros-pasos.md#91-qué-pasa-cuando-tu-sesión-vence)
    - [9.2 Cambio obligatorio de contraseña](capitulo-01-primeros-pasos.md#92-cambio-obligatorio-de-contraseña)
  - [10. Lo que ve un cajero frente a un administrador](capitulo-01-primeros-pasos.md#10-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [11. Diferencias con la documentación técnica](capitulo-01-primeros-pasos.md#11-diferencias-con-la-documentación-técnica)
  - [12. Pendientes y «No verificado»](capitulo-01-primeros-pasos.md#12-pendientes-y-no-verificado)
  - [13. Tabla de cobertura](capitulo-01-primeros-pasos.md#13-tabla-de-cobertura)
    - [13.1 /auth/login (`producto/pantallas/acceso-layout-y-compartidos` §1.1)](capitulo-01-primeros-pasos.md#131-authlogin-productopantallasacceso-layout-y-compartidos-11)
    - [13.2 /auth/register (`producto/pantallas/acceso-layout-y-compartidos` §1.2)](capitulo-01-primeros-pasos.md#132-authregister-productopantallasacceso-layout-y-compartidos-12)
    - [13.3 /auth/forgot-password (`producto/pantallas/acceso-layout-y-compartidos` §1.3) y panel compartido (§1.5)](capitulo-01-primeros-pasos.md#133-authforgot-password-productopantallasacceso-layout-y-compartidos-13-y-panel-compartido-15)
    - [13.4 /auth/reset-password (`producto/pantallas/acceso-layout-y-compartidos` §1.4)](capitulo-01-primeros-pasos.md#134-authreset-password-productopantallasacceso-layout-y-compartidos-14)
    - [13.5 /cambiar-clave (`producto/pantallas/acceso-layout-y-compartidos` §2)](capitulo-01-primeros-pasos.md#135-cambiar-clave-productopantallasacceso-layout-y-compartidos-2)
    - [13.6 Cabecera, menú lateral y cierre de sesión (`producto/pantallas/acceso-layout-y-compartidos` §4, §5)](capitulo-01-primeros-pasos.md#136-cabecera-menú-lateral-y-cierre-de-sesión-productopantallasacceso-layout-y-compartidos-4-5)
  - [14. Fuentes](capitulo-01-primeros-pasos.md#14-fuentes)
- **[Capítulo 2 — Configurar tu empresa: datos de la empresa, sucursales y series fiscales](capitulo-02-empresa-sucursales-series.md)**
  - [1. Introducción](capitulo-02-empresa-sucursales-series.md#1-introducción)
  - [2. Antes de empezar](capitulo-02-empresa-sucursales-series.md#2-antes-de-empezar)
  - [3. Revisar y editar los datos de tu empresa](capitulo-02-empresa-sucursales-series.md#3-revisar-y-editar-los-datos-de-tu-empresa)
    - [3.1 Buscar tu empresa](capitulo-02-empresa-sucursales-series.md#31-buscar-tu-empresa)
    - [3.2 Ver el detalle de tu empresa](capitulo-02-empresa-sucursales-series.md#32-ver-el-detalle-de-tu-empresa)
    - [3.3 Editar el nombre o el RUC](capitulo-02-empresa-sucursales-series.md#33-editar-el-nombre-o-el-ruc)
  - [4. Completar la sede principal](capitulo-02-empresa-sucursales-series.md#4-completar-la-sede-principal)
    - [4.1 La pantalla de sucursales](capitulo-02-empresa-sucursales-series.md#41-la-pantalla-de-sucursales)
    - [4.2 Abrir la sede principal](capitulo-02-empresa-sucursales-series.md#42-abrir-la-sede-principal)
    - [4.3 Editar y guardar con una licencia](capitulo-02-empresa-sucursales-series.md#43-editar-y-guardar-con-una-licencia)
  - [5. Crear una segunda sucursal](capitulo-02-empresa-sucursales-series.md#5-crear-una-segunda-sucursal)
    - [5.1 Probar los errores antes de guardar](capitulo-02-empresa-sucursales-series.md#51-probar-los-errores-antes-de-guardar)
    - [5.2 Guardar](capitulo-02-empresa-sucursales-series.md#52-guardar)
  - [6. Buscar sucursales y exportarlas a Excel](capitulo-02-empresa-sucursales-series.md#6-buscar-sucursales-y-exportarlas-a-excel)
    - [6.1 Filtros](capitulo-02-empresa-sucursales-series.md#61-filtros)
    - [6.2 Tabla de resultados](capitulo-02-empresa-sucursales-series.md#62-tabla-de-resultados)
    - [6.3 Exportar a Excel](capitulo-02-empresa-sucursales-series.md#63-exportar-a-excel)
  - [7. Eliminar una sucursal](capitulo-02-empresa-sucursales-series.md#7-eliminar-una-sucursal)
    - [7.1 Crear la sucursal de prueba](capitulo-02-empresa-sucursales-series.md#71-crear-la-sucursal-de-prueba)
    - [7.2 Eliminarla](capitulo-02-empresa-sucursales-series.md#72-eliminarla)
    - [7.3 Cuando la sucursal está en uso](capitulo-02-empresa-sucursales-series.md#73-cuando-la-sucursal-está-en-uso)
  - [8. Cargar sucursales desde un Excel (carga masiva)](capitulo-02-empresa-sucursales-series.md#8-cargar-sucursales-desde-un-excel-carga-masiva)
    - [8.1 La pantalla](capitulo-02-empresa-sucursales-series.md#81-la-pantalla)
    - [8.2 Flujo completo](capitulo-02-empresa-sucursales-series.md#82-flujo-completo)
    - [8.3 Errores por fila en la vista previa](capitulo-02-empresa-sucursales-series.md#83-errores-por-fila-en-la-vista-previa)
    - [8.4 Errores del archivo](capitulo-02-empresa-sucursales-series.md#84-errores-del-archivo)
    - [8.5 Qué se comprobó y qué no](capitulo-02-empresa-sucursales-series.md#85-qué-se-comprobó-y-qué-no)
  - [9. Series fiscales: qué son y para qué las usa SOFIA](capitulo-02-empresa-sucursales-series.md#9-series-fiscales-qué-son-y-para-qué-las-usa-sofia)
    - [9.1 Qué es una serie](capitulo-02-empresa-sucursales-series.md#91-qué-es-una-serie)
    - [9.2 Lo que debes tener claro](capitulo-02-empresa-sucursales-series.md#92-lo-que-debes-tener-claro)
    - [9.3 Qué series crear](capitulo-02-empresa-sucursales-series.md#93-qué-series-crear)
  - [10. Crear una serie fiscal](capitulo-02-empresa-sucursales-series.md#10-crear-una-serie-fiscal)
    - [10.1 Cómo es la pantalla](capitulo-02-empresa-sucursales-series.md#101-cómo-es-la-pantalla)
    - [10.2 Probar los errores antes de guardar](capitulo-02-empresa-sucursales-series.md#102-probar-los-errores-antes-de-guardar)
    - [10.3 Guardar](capitulo-02-empresa-sucursales-series.md#103-guardar)
    - [10.4 Errores del servidor al guardar](capitulo-02-empresa-sucursales-series.md#104-errores-del-servidor-al-guardar)
  - [11. Buscar, editar, desactivar y eliminar una serie](capitulo-02-empresa-sucursales-series.md#11-buscar-editar-desactivar-y-eliminar-una-serie)
    - [11.1 Buscar series y exportar](capitulo-02-empresa-sucursales-series.md#111-buscar-series-y-exportar)
    - [11.2 Ver y editar una serie](capitulo-02-empresa-sucursales-series.md#112-ver-y-editar-una-serie)
    - [11.3 Desactivar una serie](capitulo-02-empresa-sucursales-series.md#113-desactivar-una-serie)
    - [11.4 Eliminar una serie](capitulo-02-empresa-sucursales-series.md#114-eliminar-una-serie)
  - [12. Lo que ve un cajero frente a un administrador](capitulo-02-empresa-sucursales-series.md#12-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [13. Diferencias con la documentación técnica](capitulo-02-empresa-sucursales-series.md#13-diferencias-con-la-documentación-técnica)
  - [14. Pendientes y «No verificado»](capitulo-02-empresa-sucursales-series.md#14-pendientes-y-no-verificado)
  - [15. Tabla de cobertura](capitulo-02-empresa-sucursales-series.md#15-tabla-de-cobertura)
    - [15.1 Empresa — búsqueda /empresas (`producto/pantallas/administracion` §1.1)](capitulo-02-empresa-sucursales-series.md#151-empresa--búsqueda-empresas-productopantallasadministracion-11)
    - [15.2 Empresa — detalle /empresas/:id (`producto/pantallas/administracion` §1.2)](capitulo-02-empresa-sucursales-series.md#152-empresa--detalle-empresasid-productopantallasadministracion-12)
    - [15.3 Sucursales — búsqueda /sucursales (`producto/pantallas/administracion` §2.1)](capitulo-02-empresa-sucursales-series.md#153-sucursales--búsqueda-sucursales-productopantallasadministracion-21)
    - [15.4 Sucursales — detalle /sucursales/nueva y /sucursales/:id (`producto/pantallas/administracion` §2.2)](capitulo-02-empresa-sucursales-series.md#154-sucursales--detalle-sucursalesnueva-y-sucursalesid-productopantallasadministracion-22)
    - [15.5 Sucursales — carga masiva /sucursales/carga-masiva (`producto/pantallas/administracion` §2.3, `producto/pantallas/acceso-layout-y-compartidos` §6.5-6.6, doc `tecnico/carga-masiva-y-exportacion` §3.11)](capitulo-02-empresa-sucursales-series.md#155-sucursales--carga-masiva-sucursalescarga-masiva-productopantallasadministracion-23-productopantallasacceso-layout-y-compartidos-65-66-doc-tecnicocarga-masiva-y-exportacion-311)
    - [15.6 Series fiscales — búsqueda /series-fiscales (`producto/pantallas/administracion` §6.1)](capitulo-02-empresa-sucursales-series.md#156-series-fiscales--búsqueda-series-fiscales-productopantallasadministracion-61)
    - [15.7 Series fiscales — detalle /series-fiscales/nueva y /series-fiscales/:id (`producto/pantallas/administracion` §6.2)](capitulo-02-empresa-sucursales-series.md#157-series-fiscales--detalle-series-fiscalesnueva-y-series-fiscalesid-productopantallasadministracion-62)
  - [16. Fuentes](capitulo-02-empresa-sucursales-series.md#16-fuentes)
    - [Datos que quedan en la empresa de prueba](capitulo-02-empresa-sucursales-series.md#datos-que-quedan-en-la-empresa-de-prueba)
- **[Capítulo 3 — Tu equipo y sus accesos: empleados, roles y permisos, cuentas de usuario y sucursales](capitulo-03-empleados-roles-cuentas.md)**
  - [1. Introducción](capitulo-03-empleados-roles-cuentas.md#1-introducción)
  - [2. Antes de empezar](capitulo-03-empleados-roles-cuentas.md#2-antes-de-empezar)
  - [3. Registrar empleados](capitulo-03-empleados-roles-cuentas.md#3-registrar-empleados)
    - [3.1 La pantalla de búsqueda](capitulo-03-empleados-roles-cuentas.md#31-la-pantalla-de-búsqueda)
    - [3.2 Crear un empleado](capitulo-03-empleados-roles-cuentas.md#32-crear-un-empleado)
    - [3.3 Buscar y exportar empleados](capitulo-03-empleados-roles-cuentas.md#33-buscar-y-exportar-empleados)
    - [3.4 Ver el detalle de un empleado](capitulo-03-empleados-roles-cuentas.md#34-ver-el-detalle-de-un-empleado)
  - [4. Entender los roles y los permisos](capitulo-03-empleados-roles-cuentas.md#4-entender-los-roles-y-los-permisos)
    - [4.1 Qué es un rol](capitulo-03-empleados-roles-cuentas.md#41-qué-es-un-rol)
    - [4.2 Tu empresa nace solo con el rol «Admin»](capitulo-03-empleados-roles-cuentas.md#42-tu-empresa-nace-solo-con-el-rol-admin)
    - [4.3 La lista de roles que ve tu empresa](capitulo-03-empleados-roles-cuentas.md#43-la-lista-de-roles-que-ve-tu-empresa)
    - [4.4 La pantalla de un rol](capitulo-03-empleados-roles-cuentas.md#44-la-pantalla-de-un-rol)
  - [5. Crear el rol «Cajero» y marcar sus permisos](capitulo-03-empleados-roles-cuentas.md#5-crear-el-rol-cajero-y-marcar-sus-permisos)
    - [5.1 Crear el rol](capitulo-03-empleados-roles-cuentas.md#51-crear-el-rol)
    - [5.2 La matriz de permisos](capitulo-03-empleados-roles-cuentas.md#52-la-matriz-de-permisos)
    - [5.3 Permisos recomendados para un cajero](capitulo-03-empleados-roles-cuentas.md#53-permisos-recomendados-para-un-cajero)
    - [5.4 Pasos para marcar los permisos](capitulo-03-empleados-roles-cuentas.md#54-pasos-para-marcar-los-permisos)
    - [5.5 Sucursales del rol](capitulo-03-empleados-roles-cuentas.md#55-sucursales-del-rol)
  - [6. Qué pasa si quitas un permiso](capitulo-03-empleados-roles-cuentas.md#6-qué-pasa-si-quitas-un-permiso)
  - [7. Crear la cuenta del cajero](capitulo-03-empleados-roles-cuentas.md#7-crear-la-cuenta-del-cajero)
    - [7.1 La pantalla de búsqueda de cuentas](capitulo-03-empleados-roles-cuentas.md#71-la-pantalla-de-búsqueda-de-cuentas)
    - [7.2 Crear la cuenta](capitulo-03-empleados-roles-cuentas.md#72-crear-la-cuenta)
    - [7.3 Asignar el rol «Cajero»](capitulo-03-empleados-roles-cuentas.md#73-asignar-el-rol-cajero)
    - [7.4 La ficha de una cuenta](capitulo-03-empleados-roles-cuentas.md#74-la-ficha-de-una-cuenta)
    - [7.5 Activar, desactivar, forzar el cambio de contraseña y resetear intentos](capitulo-03-empleados-roles-cuentas.md#75-activar-desactivar-forzar-el-cambio-de-contraseña-y-resetear-intentos)
    - [7.6 El bloqueo automático](capitulo-03-empleados-roles-cuentas.md#76-el-bloqueo-automático)
    - [7.7 Agregar y quitar sucursales de una cuenta](capitulo-03-empleados-roles-cuentas.md#77-agregar-y-quitar-sucursales-de-una-cuenta)
  - [8. Primer ingreso del cajero](capitulo-03-empleados-roles-cuentas.md#8-primer-ingreso-del-cajero)
    - [8.1 El cambio obligatorio de contraseña](capitulo-03-empleados-roles-cuentas.md#81-el-cambio-obligatorio-de-contraseña)
    - [8.2 Qué ve el cajero al entrar](capitulo-03-empleados-roles-cuentas.md#82-qué-ve-el-cajero-al-entrar)
  - [9. Editar, desactivar y eliminar](capitulo-03-empleados-roles-cuentas.md#9-editar-desactivar-y-eliminar)
    - [9.1 Editar y eliminar un empleado](capitulo-03-empleados-roles-cuentas.md#91-editar-y-eliminar-un-empleado)
    - [9.2 Editar, desactivar y eliminar una cuenta](capitulo-03-empleados-roles-cuentas.md#92-editar-desactivar-y-eliminar-una-cuenta)
    - [9.3 Editar y eliminar un rol](capitulo-03-empleados-roles-cuentas.md#93-editar-y-eliminar-un-rol)
    - [9.4 Qué quedó al terminar las pruebas](capitulo-03-empleados-roles-cuentas.md#94-qué-quedó-al-terminar-las-pruebas)
  - [10. Buscar, exportar y cargar roles](capitulo-03-empleados-roles-cuentas.md#10-buscar-exportar-y-cargar-roles)
    - [10.1 Buscar roles](capitulo-03-empleados-roles-cuentas.md#101-buscar-roles)
    - [10.2 Exportar roles](capitulo-03-empleados-roles-cuentas.md#102-exportar-roles)
    - [10.3 Cargar roles desde un Excel (carga masiva)](capitulo-03-empleados-roles-cuentas.md#103-cargar-roles-desde-un-excel-carga-masiva)
  - [11. Lo que ve un cajero frente a un administrador](capitulo-03-empleados-roles-cuentas.md#11-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [12. Diferencias con la documentación técnica](capitulo-03-empleados-roles-cuentas.md#12-diferencias-con-la-documentación-técnica)
  - [13. Pendientes y «No verificado»](capitulo-03-empleados-roles-cuentas.md#13-pendientes-y-no-verificado)
  - [14. Tabla de cobertura](capitulo-03-empleados-roles-cuentas.md#14-tabla-de-cobertura)
    - [14.1 Empleados — búsqueda /empleados (`producto/pantallas/administracion` §3.1)](capitulo-03-empleados-roles-cuentas.md#141-empleados--búsqueda-empleados-productopantallasadministracion-31)
    - [14.2 Empleados — detalle /empleados/nueva y /empleados/:id (`producto/pantallas/administracion` §3.2)](capitulo-03-empleados-roles-cuentas.md#142-empleados--detalle-empleadosnueva-y-empleadosid-productopantallasadministracion-32)
    - [14.3 Roles — búsqueda /roles (`producto/pantallas/administracion` §4.1)](capitulo-03-empleados-roles-cuentas.md#143-roles--búsqueda-roles-productopantallasadministracion-41)
    - [14.4 Roles — detalle y configuración /roles/nueva y /roles/:id (`producto/pantallas/administracion` §4.2)](capitulo-03-empleados-roles-cuentas.md#144-roles--detalle-y-configuración-rolesnueva-y-rolesid-productopantallasadministracion-42)
    - [14.5 Roles — carga masiva /roles/carga-masiva (`producto/pantallas/administracion` §4.3, `producto/pantallas/acceso-layout-y-compartidos` §6.5)](capitulo-03-empleados-roles-cuentas.md#145-roles--carga-masiva-rolescarga-masiva-productopantallasadministracion-43-productopantallasacceso-layout-y-compartidos-65)
    - [14.6 Cuentas de usuario — búsqueda /cuentas (`producto/pantallas/administracion` §5.1)](capitulo-03-empleados-roles-cuentas.md#146-cuentas-de-usuario--búsqueda-cuentas-productopantallasadministracion-51)
    - [14.7 Cuentas de usuario — detalle /cuentas/nueva y /cuentas/:id (`producto/pantallas/administracion` §5.2)](capitulo-03-empleados-roles-cuentas.md#147-cuentas-de-usuario--detalle-cuentasnueva-y-cuentasid-productopantallasadministracion-52)
    - [14.8 Cambio obligatorio de contraseña /cambiar-clave (`producto/pantallas/acceso-layout-y-compartidos` §2)](capitulo-03-empleados-roles-cuentas.md#148-cambio-obligatorio-de-contraseña-cambiar-clave-productopantallasacceso-layout-y-compartidos-2)
    - [14.9 Tareas del encargo](capitulo-03-empleados-roles-cuentas.md#149-tareas-del-encargo)
  - [15. Fuentes](capitulo-03-empleados-roles-cuentas.md#15-fuentes)
- **[Capítulo 4 — Catálogos básicos: unidades de medida, laboratorios, ingredientes activos y catálogo DIGEMID](capitulo-04-catalogos-basicos.md)**
  - [1. Introducción](capitulo-04-catalogos-basicos.md#1-introducción)
  - [2. Antes de empezar](capitulo-04-catalogos-basicos.md#2-antes-de-empezar)
  - [3. Qué es cada catálogo (en lenguaje simple)](capitulo-04-catalogos-basicos.md#3-qué-es-cada-catálogo-en-lenguaje-simple)
  - [4. Lo que tienen en común las cuatro pantallas](capitulo-04-catalogos-basicos.md#4-lo-que-tienen-en-común-las-cuatro-pantallas)
  - [5. Unidades de medida](capitulo-04-catalogos-basicos.md#5-unidades-de-medida)
    - [5.1 Qué unidades crear](capitulo-04-catalogos-basicos.md#51-qué-unidades-crear)
    - [5.2 La pantalla de búsqueda](capitulo-04-catalogos-basicos.md#52-la-pantalla-de-búsqueda)
    - [5.3 Crear una unidad](capitulo-04-catalogos-basicos.md#53-crear-una-unidad)
    - [5.4 Buscar, filtrar y exportar](capitulo-04-catalogos-basicos.md#54-buscar-filtrar-y-exportar)
    - [5.5 Ver y editar una unidad](capitulo-04-catalogos-basicos.md#55-ver-y-editar-una-unidad)
    - [5.6 Eliminar una unidad](capitulo-04-catalogos-basicos.md#56-eliminar-una-unidad)
    - [5.7 Carga masiva](capitulo-04-catalogos-basicos.md#57-carga-masiva)
  - [6. Laboratorios](capitulo-04-catalogos-basicos.md#6-laboratorios)
    - [6.1 Qué laboratorios crear](capitulo-04-catalogos-basicos.md#61-qué-laboratorios-crear)
    - [6.2 La pantalla de búsqueda](capitulo-04-catalogos-basicos.md#62-la-pantalla-de-búsqueda)
    - [6.3 Crear un laboratorio](capitulo-04-catalogos-basicos.md#63-crear-un-laboratorio)
    - [6.4 Buscar y exportar](capitulo-04-catalogos-basicos.md#64-buscar-y-exportar)
    - [6.5 Ver, editar y eliminar un laboratorio](capitulo-04-catalogos-basicos.md#65-ver-editar-y-eliminar-un-laboratorio)
  - [7. Ingredientes activos](capitulo-04-catalogos-basicos.md#7-ingredientes-activos)
    - [7.1 Qué ingredientes crear](capitulo-04-catalogos-basicos.md#71-qué-ingredientes-crear)
    - [7.2 La pantalla de búsqueda](capitulo-04-catalogos-basicos.md#72-la-pantalla-de-búsqueda)
    - [7.3 Crear un ingrediente activo](capitulo-04-catalogos-basicos.md#73-crear-un-ingrediente-activo)
    - [7.4 Buscar y exportar](capitulo-04-catalogos-basicos.md#74-buscar-y-exportar)
    - [7.5 Ver, editar y eliminar](capitulo-04-catalogos-basicos.md#75-ver-editar-y-eliminar)
  - [8. Catálogo DIGEMID](capitulo-04-catalogos-basicos.md#8-catálogo-digemid)
    - [8.1 Qué es y para qué lo usa SOFIA](capitulo-04-catalogos-basicos.md#81-qué-es-y-para-qué-lo-usa-sofia)
    - [8.2 Qué puedes hacer y qué no en esta pantalla](capitulo-04-catalogos-basicos.md#82-qué-puedes-hacer-y-qué-no-en-esta-pantalla)
    - [8.3 Buscar y exportar](capitulo-04-catalogos-basicos.md#83-buscar-y-exportar)
    - [8.4 Ver un producto](capitulo-04-catalogos-basicos.md#84-ver-un-producto)
    - [8.5 Cargar productos desde un Excel (carga masiva)](capitulo-04-catalogos-basicos.md#85-cargar-productos-desde-un-excel-carga-masiva)
  - [9. Cargar unidades, laboratorios e ingredientes desde un Excel](capitulo-04-catalogos-basicos.md#9-cargar-unidades-laboratorios-e-ingredientes-desde-un-excel)
    - [9.1 Antes de cargar](capitulo-04-catalogos-basicos.md#91-antes-de-cargar)
    - [9.2 Unidades de medida](capitulo-04-catalogos-basicos.md#92-unidades-de-medida)
    - [9.3 Laboratorios](capitulo-04-catalogos-basicos.md#93-laboratorios)
    - [9.4 Ingredientes activos](capitulo-04-catalogos-basicos.md#94-ingredientes-activos)
  - [10. Lo que ve un cajero frente a un administrador](capitulo-04-catalogos-basicos.md#10-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [11. Diferencias con la documentación técnica](capitulo-04-catalogos-basicos.md#11-diferencias-con-la-documentación-técnica)
  - [12. Pendientes y «No verificado»](capitulo-04-catalogos-basicos.md#12-pendientes-y-no-verificado)
  - [13. Tabla de cobertura](capitulo-04-catalogos-basicos.md#13-tabla-de-cobertura)
    - [13.1 Unidades de medida — búsqueda /unidades-medida (`producto/pantallas/catalogos-basicos` §1.1)](capitulo-04-catalogos-basicos.md#131-unidades-de-medida--búsqueda-unidades-medida-productopantallascatalogos-basicos-11)
    - [13.2 Unidades de medida — detalle /unidades-medida/nueva y /:id (`producto/pantallas/catalogos-basicos` §1.2)](capitulo-04-catalogos-basicos.md#132-unidades-de-medida--detalle-unidades-medidanueva-y-id-productopantallascatalogos-basicos-12)
    - [13.3 Unidades de medida — carga masiva /unidades-medida/carga-masiva (`producto/pantallas/catalogos-basicos` §1.3)](capitulo-04-catalogos-basicos.md#133-unidades-de-medida--carga-masiva-unidades-medidacarga-masiva-productopantallascatalogos-basicos-13)
    - [13.4 Laboratorios — búsqueda /laboratorios (`producto/pantallas/catalogos-basicos` §3.1)](capitulo-04-catalogos-basicos.md#134-laboratorios--búsqueda-laboratorios-productopantallascatalogos-basicos-31)
    - [13.5 Laboratorios — detalle /laboratorios/nueva y /:id (`producto/pantallas/catalogos-basicos` §3.2)](capitulo-04-catalogos-basicos.md#135-laboratorios--detalle-laboratoriosnueva-y-id-productopantallascatalogos-basicos-32)
    - [13.6 Laboratorios — carga masiva /laboratorios/carga-masiva (`producto/pantallas/catalogos-basicos` §3.3)](capitulo-04-catalogos-basicos.md#136-laboratorios--carga-masiva-laboratorioscarga-masiva-productopantallascatalogos-basicos-33)
    - [13.7 Ingredientes activos — búsqueda /ingredientes-activos (`producto/pantallas/catalogos-basicos` §4.1)](capitulo-04-catalogos-basicos.md#137-ingredientes-activos--búsqueda-ingredientes-activos-productopantallascatalogos-basicos-41)
    - [13.8 Ingredientes activos — detalle /ingredientes-activos/nueva y /:id (`producto/pantallas/catalogos-basicos` §4.2)](capitulo-04-catalogos-basicos.md#138-ingredientes-activos--detalle-ingredientes-activosnueva-y-id-productopantallascatalogos-basicos-42)
    - [13.9 Ingredientes activos — carga masiva /ingredientes-activos/carga-masiva (`producto/pantallas/catalogos-basicos` §4.3)](capitulo-04-catalogos-basicos.md#139-ingredientes-activos--carga-masiva-ingredientes-activoscarga-masiva-productopantallascatalogos-basicos-43)
    - [13.10 Catálogo DIGEMID — búsqueda /digemid-catalogo (`producto/pantallas/catalogos-basicos` §5.1)](capitulo-04-catalogos-basicos.md#1310-catálogo-digemid--búsqueda-digemid-catalogo-productopantallascatalogos-basicos-51)
    - [13.11 Catálogo DIGEMID — detalle /digemid-catalogo/:id (`producto/pantallas/catalogos-basicos` §5.2)](capitulo-04-catalogos-basicos.md#1311-catálogo-digemid--detalle-digemid-catalogoid-productopantallascatalogos-basicos-52)
    - [13.12 Catálogo DIGEMID — carga masiva /digemid-catalogo/carga-masiva (`producto/pantallas/catalogos-basicos` §5.3)](capitulo-04-catalogos-basicos.md#1312-catálogo-digemid--carga-masiva-digemid-catalogocarga-masiva-productopantallascatalogos-basicos-53)
    - [13.13 Comportamientos comunes (`producto/pantallas/catalogos-basicos` §0) y eliminar (`producto/pantallas/catalogos-basicos` §6)](capitulo-04-catalogos-basicos.md#1313-comportamientos-comunes-productopantallascatalogos-basicos-0-y-eliminar-productopantallascatalogos-basicos-6)
  - [14. Fuentes](capitulo-04-catalogos-basicos.md#14-fuentes)
- **[Capítulo 5 — Productos y proveedores: medicamentos, jerarquías de unidades, presentaciones de venta, formulaciones clínicas, proveedores y aseguradoras](capitulo-05-productos-y-proveedores.md)**
  - [1. Introducción](capitulo-05-productos-y-proveedores.md#1-introducción)
  - [2. Antes de empezar](capitulo-05-productos-y-proveedores.md#2-antes-de-empezar)
  - [3. Cómo se relacionan estas piezas (en lenguaje simple)](capitulo-05-productos-y-proveedores.md#3-cómo-se-relacionan-estas-piezas-en-lenguaje-simple)
  - [4. Medicamentos](capitulo-05-productos-y-proveedores.md#4-medicamentos)
    - [4.1 Qué medicamentos crear](capitulo-05-productos-y-proveedores.md#41-qué-medicamentos-crear)
    - [4.2 La condición de venta y qué implica en el punto de venta](capitulo-05-productos-y-proveedores.md#42-la-condición-de-venta-y-qué-implica-en-el-punto-de-venta)
    - [4.3 La pantalla de búsqueda](capitulo-05-productos-y-proveedores.md#43-la-pantalla-de-búsqueda)
    - [4.4 Crear un medicamento](capitulo-05-productos-y-proveedores.md#44-crear-un-medicamento)
    - [4.5 Buscar, filtrar y exportar](capitulo-05-productos-y-proveedores.md#45-buscar-filtrar-y-exportar)
    - [4.6 Ver, editar y eliminar un medicamento](capitulo-05-productos-y-proveedores.md#46-ver-editar-y-eliminar-un-medicamento)
  - [5. Jerarquías de unidades](capitulo-05-productos-y-proveedores.md#5-jerarquías-de-unidades)
    - [5.1 Qué es una jerarquía](capitulo-05-productos-y-proveedores.md#51-qué-es-una-jerarquía)
    - [5.2 La pantalla de búsqueda](capitulo-05-productos-y-proveedores.md#52-la-pantalla-de-búsqueda)
    - [5.3 Crear una jerarquía](capitulo-05-productos-y-proveedores.md#53-crear-una-jerarquía)
    - [5.4 Ver, editar y eliminar](capitulo-05-productos-y-proveedores.md#54-ver-editar-y-eliminar)
  - [6. Presentaciones de venta](capitulo-05-productos-y-proveedores.md#6-presentaciones-de-venta)
    - [6.1 Qué es una presentación y cómo se calcula su equivalencia](capitulo-05-productos-y-proveedores.md#61-qué-es-una-presentación-y-cómo-se-calcula-su-equivalencia)
    - [6.2 La pantalla de búsqueda](capitulo-05-productos-y-proveedores.md#62-la-pantalla-de-búsqueda)
    - [6.3 Crear una presentación](capitulo-05-productos-y-proveedores.md#63-crear-una-presentación)
    - [6.4 Ver, editar y eliminar una presentación](capitulo-05-productos-y-proveedores.md#64-ver-editar-y-eliminar-una-presentación)
    - [6.5 Administrar las presentaciones desde la ficha del medicamento](capitulo-05-productos-y-proveedores.md#65-administrar-las-presentaciones-desde-la-ficha-del-medicamento)
  - [7. Formulaciones clínicas](capitulo-05-productos-y-proveedores.md#7-formulaciones-clínicas)
    - [7.1 Qué es una formulación clínica](capitulo-05-productos-y-proveedores.md#71-qué-es-una-formulación-clínica)
    - [7.2 La pantalla de búsqueda](capitulo-05-productos-y-proveedores.md#72-la-pantalla-de-búsqueda)
    - [7.3 Crear una formulación](capitulo-05-productos-y-proveedores.md#73-crear-una-formulación)
    - [7.4 Buscar, filtrar y exportar](capitulo-05-productos-y-proveedores.md#74-buscar-filtrar-y-exportar)
    - [7.5 Ver, editar y eliminar una formulación](capitulo-05-productos-y-proveedores.md#75-ver-editar-y-eliminar-una-formulación)
  - [8. Proveedores y precios de compra](capitulo-05-productos-y-proveedores.md#8-proveedores-y-precios-de-compra)
    - [8.1 Qué es un proveedor](capitulo-05-productos-y-proveedores.md#81-qué-es-un-proveedor)
    - [8.2 La pantalla de búsqueda](capitulo-05-productos-y-proveedores.md#82-la-pantalla-de-búsqueda)
    - [8.3 Crear un proveedor](capitulo-05-productos-y-proveedores.md#83-crear-un-proveedor)
    - [8.4 Ver, editar y eliminar un proveedor](capitulo-05-productos-y-proveedores.md#84-ver-editar-y-eliminar-un-proveedor)
    - [8.5 Precios de compra](capitulo-05-productos-y-proveedores.md#85-precios-de-compra)
  - [9. Aseguradoras](capitulo-05-productos-y-proveedores.md#9-aseguradoras)
    - [9.1 Qué es una aseguradora y para qué sirve hoy](capitulo-05-productos-y-proveedores.md#91-qué-es-una-aseguradora-y-para-qué-sirve-hoy)
    - [9.2 La pantalla de búsqueda](capitulo-05-productos-y-proveedores.md#92-la-pantalla-de-búsqueda)
    - [9.3 Crear una aseguradora](capitulo-05-productos-y-proveedores.md#93-crear-una-aseguradora)
    - [9.4 Ver, editar y eliminar](capitulo-05-productos-y-proveedores.md#94-ver-editar-y-eliminar)
  - [10. Eliminar algo que está en uso y carga masiva](capitulo-05-productos-y-proveedores.md#10-eliminar-algo-que-está-en-uso-y-carga-masiva)
    - [10.1 Eliminar un medicamento o una unidad en uso](capitulo-05-productos-y-proveedores.md#101-eliminar-un-medicamento-o-una-unidad-en-uso)
    - [10.2 Eliminar un ingrediente en una formulación o un laboratorio con medicamentos](capitulo-05-productos-y-proveedores.md#102-eliminar-un-ingrediente-en-una-formulación-o-un-laboratorio-con-medicamentos)
    - [10.3 Carga masiva de medicamentos](capitulo-05-productos-y-proveedores.md#103-carga-masiva-de-medicamentos)
    - [10.4 Carga masiva de jerarquías de unidades](capitulo-05-productos-y-proveedores.md#104-carga-masiva-de-jerarquías-de-unidades)
    - [10.5 Carga masiva de proveedores](capitulo-05-productos-y-proveedores.md#105-carga-masiva-de-proveedores)
    - [10.6 Carga masiva de aseguradoras](capitulo-05-productos-y-proveedores.md#106-carga-masiva-de-aseguradoras)
  - [11. Lo que ve un cajero frente a un administrador](capitulo-05-productos-y-proveedores.md#11-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [12. Diferencias con la documentación técnica](capitulo-05-productos-y-proveedores.md#12-diferencias-con-la-documentación-técnica)
  - [13. Pendientes y «No verificado»](capitulo-05-productos-y-proveedores.md#13-pendientes-y-no-verificado)
  - [14. Tabla de cobertura](capitulo-05-productos-y-proveedores.md#14-tabla-de-cobertura)
    - [14.1 Medicamentos — búsqueda /medicamentos (`producto/pantallas/productos-proveedores-y-aseguradoras` §1.1)](capitulo-05-productos-y-proveedores.md#141-medicamentos--búsqueda-medicamentos-productopantallasproductos-proveedores-y-aseguradoras-11)
    - [14.2 Medicamentos — detalle /medicamentos/nueva y /:id (`producto/pantallas/productos-proveedores-y-aseguradoras` §1.2)](capitulo-05-productos-y-proveedores.md#142-medicamentos--detalle-medicamentosnueva-y-id-productopantallasproductos-proveedores-y-aseguradoras-12)
    - [14.3 Medicamentos — carga masiva /medicamentos/carga-masiva (`producto/pantallas/productos-proveedores-y-aseguradoras` §1.3)](capitulo-05-productos-y-proveedores.md#143-medicamentos--carga-masiva-medicamentoscarga-masiva-productopantallasproductos-proveedores-y-aseguradoras-13)
    - [14.4 Presentaciones de venta — búsqueda /presentaciones-venta (`producto/pantallas/productos-proveedores-y-aseguradoras` §2.1)](capitulo-05-productos-y-proveedores.md#144-presentaciones-de-venta--búsqueda-presentaciones-venta-productopantallasproductos-proveedores-y-aseguradoras-21)
    - [14.5 Presentaciones de venta — detalle /presentaciones-venta/nueva y /:id (`producto/pantallas/productos-proveedores-y-aseguradoras` §2.2 y §2.3)](capitulo-05-productos-y-proveedores.md#145-presentaciones-de-venta--detalle-presentaciones-ventanueva-y-id-productopantallasproductos-proveedores-y-aseguradoras-22-y-23)
    - [14.6 Presentaciones de venta — cuadro desde el medicamento (`producto/pantallas/productos-proveedores-y-aseguradoras` §2.4)](capitulo-05-productos-y-proveedores.md#146-presentaciones-de-venta--cuadro-desde-el-medicamento-productopantallasproductos-proveedores-y-aseguradoras-24)
    - [14.7 Formulaciones clínicas — búsqueda y detalle (`producto/pantallas/productos-proveedores-y-aseguradoras` §3.1 y §3.2)](capitulo-05-productos-y-proveedores.md#147-formulaciones-clínicas--búsqueda-y-detalle-productopantallasproductos-proveedores-y-aseguradoras-31-y-32)
    - [14.8 Proveedores — búsqueda, detalle y carga masiva (`producto/pantallas/productos-proveedores-y-aseguradoras` §4.1, §4.2 y §4.4)](capitulo-05-productos-y-proveedores.md#148-proveedores--búsqueda-detalle-y-carga-masiva-productopantallasproductos-proveedores-y-aseguradoras-41-42-y-44)
    - [14.9 Proveedores — sección «Precios de compra» y cuadro de precio (`producto/pantallas/productos-proveedores-y-aseguradoras` §4.3 y §4.5)](capitulo-05-productos-y-proveedores.md#149-proveedores--sección-precios-de-compra-y-cuadro-de-precio-productopantallasproductos-proveedores-y-aseguradoras-43-y-45)
    - [14.10 Aseguradoras — búsqueda, detalle y carga masiva (`producto/pantallas/productos-proveedores-y-aseguradoras` §5.1 a §5.3)](capitulo-05-productos-y-proveedores.md#1410-aseguradoras--búsqueda-detalle-y-carga-masiva-productopantallasproductos-proveedores-y-aseguradoras-51-a-53)
    - [14.11 Jerarquías de unidades (`producto/pantallas/catalogos-basicos` §2.1, §2.2 y §2.3)](capitulo-05-productos-y-proveedores.md#1411-jerarquías-de-unidades-productopantallascatalogos-basicos-21-22-y-23)
    - [14.12 Comportamientos comunes (`producto/pantallas/productos-proveedores-y-aseguradoras` §0) y eliminar (`producto/pantallas/productos-proveedores-y-aseguradoras` §6)](capitulo-05-productos-y-proveedores.md#1412-comportamientos-comunes-productopantallasproductos-proveedores-y-aseguradoras-0-y-eliminar-productopantallasproductos-proveedores-y-aseguradoras-6)
  - [15. Fuentes](capitulo-05-productos-y-proveedores.md#15-fuentes)
- **[Capítulo 6 — Inventario: lotes y stock por sucursal](capitulo-06-inventario.md)**
  - [1. Introducción](capitulo-06-inventario.md#1-introducción)
  - [2. Antes de empezar](capitulo-06-inventario.md#2-antes-de-empezar)
  - [3. Cómo se relacionan estas piezas (en lenguaje simple)](capitulo-06-inventario.md#3-cómo-se-relacionan-estas-piezas-en-lenguaje-simple)
  - [4. Los datos que se crean en este capítulo](capitulo-06-inventario.md#4-los-datos-que-se-crean-en-este-capítulo)
  - [5. Crear un lote](capitulo-06-inventario.md#5-crear-un-lote)
    - [5.1 Pasos](capitulo-06-inventario.md#51-pasos)
    - [5.2 Qué pasa al terminar bien](capitulo-06-inventario.md#52-qué-pasa-al-terminar-bien)
    - [5.3 Errores que puedes ver (todos comprobados en la aplicación)](capitulo-06-inventario.md#53-errores-que-puedes-ver-todos-comprobados-en-la-aplicación)
  - [6. Buscar, filtrar y exportar lotes](capitulo-06-inventario.md#6-buscar-filtrar-y-exportar-lotes)
    - [6.1 Los filtros](capitulo-06-inventario.md#61-los-filtros)
    - [6.2 La tabla de resultados](capitulo-06-inventario.md#62-la-tabla-de-resultados)
    - [6.3 Comprobado con los siete lotes del capítulo](capitulo-06-inventario.md#63-comprobado-con-los-siete-lotes-del-capítulo)
    - [6.4 Exportar](capitulo-06-inventario.md#64-exportar)
  - [7. Ver y editar un lote](capitulo-06-inventario.md#7-ver-y-editar-un-lote)
  - [8. Ingresar stock](capitulo-06-inventario.md#8-ingresar-stock)
    - [8.1 Pasos](capitulo-06-inventario.md#81-pasos)
    - [8.2 Qué pasa al terminar bien](capitulo-06-inventario.md#82-qué-pasa-al-terminar-bien)
    - [8.3 Errores que puedes ver](capitulo-06-inventario.md#83-errores-que-puedes-ver)
  - [9. Consultar el stock por sucursal](capitulo-06-inventario.md#9-consultar-el-stock-por-sucursal)
    - [9.1 La pantalla de búsqueda](capitulo-06-inventario.md#91-la-pantalla-de-búsqueda)
    - [9.2 Comprobado con el stock del capítulo](capitulo-06-inventario.md#92-comprobado-con-el-stock-del-capítulo)
    - [9.3 Exportar](capitulo-06-inventario.md#93-exportar)
    - [9.4 Ver lotes vencidos o por vencer desde el Dashboard](capitulo-06-inventario.md#94-ver-lotes-vencidos-o-por-vencer-desde-el-dashboard)
    - [9.5 Ver el stock desde un medicamento](capitulo-06-inventario.md#95-ver-el-stock-desde-un-medicamento)
  - [10. Ajustar una cantidad](capitulo-06-inventario.md#10-ajustar-una-cantidad)
    - [10.1 Pasos](capitulo-06-inventario.md#101-pasos)
    - [10.2 Qué pasa al terminar bien](capitulo-06-inventario.md#102-qué-pasa-al-terminar-bien)
  - [11. Eliminar un lote](capitulo-06-inventario.md#11-eliminar-un-lote)
    - [11.1 Un lote de prueba sin stock](capitulo-06-inventario.md#111-un-lote-de-prueba-sin-stock)
    - [11.2 Un lote con stock](capitulo-06-inventario.md#112-un-lote-con-stock)
  - [12. Transferencias entre sucursales](capitulo-06-inventario.md#12-transferencias-entre-sucursales)
  - [13. Lo que ve un cajero frente a un administrador](capitulo-06-inventario.md#13-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [14. Diferencias con la documentación técnica](capitulo-06-inventario.md#14-diferencias-con-la-documentación-técnica)
  - [15. Pendientes y «No verificado»](capitulo-06-inventario.md#15-pendientes-y-no-verificado)
  - [16. Tabla de cobertura](capitulo-06-inventario.md#16-tabla-de-cobertura)
    - [16.1 Lotes — búsqueda /lotes (`producto/pantallas/inventario-y-atencion` §1.1)](capitulo-06-inventario.md#161-lotes--búsqueda-lotes-productopantallasinventario-y-atencion-11)
    - [16.2 Lotes — detalle /lotes/nueva y /lotes/:id (`producto/pantallas/inventario-y-atencion` §1.2)](capitulo-06-inventario.md#162-lotes--detalle-lotesnueva-y-lotesid-productopantallasinventario-y-atencion-12)
    - [16.3 Stock por sucursal — búsqueda /stock-por-sucursal (`producto/pantallas/inventario-y-atencion` §2.1)](capitulo-06-inventario.md#163-stock-por-sucursal--búsqueda-stock-por-sucursal-productopantallasinventario-y-atencion-21)
    - [16.4 Stock por sucursal — ingreso /stock-por-sucursal/nueva (`producto/pantallas/inventario-y-atencion` §2.2)](capitulo-06-inventario.md#164-stock-por-sucursal--ingreso-stock-por-sucursalnueva-productopantallasinventario-y-atencion-22)
    - [16.5 Stock por sucursal — detalle y ajuste /stock-por-sucursal/:id (`producto/pantallas/inventario-y-atencion` §2.3)](capitulo-06-inventario.md#165-stock-por-sucursal--detalle-y-ajuste-stock-por-sucursalid-productopantallasinventario-y-atencion-23)
    - [16.6 Comportamientos comunes y reglas](capitulo-06-inventario.md#166-comportamientos-comunes-y-reglas)
  - [17. Fuentes](capitulo-06-inventario.md#17-fuentes)
- **[Capítulo 7 — Pacientes, profesionales de salud y recetas](capitulo-07-pacientes-y-recetas.md)**
  - [1. Introducción](capitulo-07-pacientes-y-recetas.md#1-introducción)
  - [2. Antes de empezar](capitulo-07-pacientes-y-recetas.md#2-antes-de-empezar)
  - [3. Cómo se relacionan estas piezas (en lenguaje simple)](capitulo-07-pacientes-y-recetas.md#3-cómo-se-relacionan-estas-piezas-en-lenguaje-simple)
  - [4. Los datos que se crean en este capítulo](capitulo-07-pacientes-y-recetas.md#4-los-datos-que-se-crean-en-este-capítulo)
  - [5. Registrar un paciente](capitulo-07-pacientes-y-recetas.md#5-registrar-un-paciente)
    - [5.1 Pasos](capitulo-07-pacientes-y-recetas.md#51-pasos)
    - [5.2 Qué pasa al terminar bien](capitulo-07-pacientes-y-recetas.md#52-qué-pasa-al-terminar-bien)
    - [5.3 Errores que puedes ver (todos comprobados en la aplicación)](capitulo-07-pacientes-y-recetas.md#53-errores-que-puedes-ver-todos-comprobados-en-la-aplicación)
  - [6. Buscar, ver y editar pacientes](capitulo-07-pacientes-y-recetas.md#6-buscar-ver-y-editar-pacientes)
    - [6.1 Los filtros](capitulo-07-pacientes-y-recetas.md#61-los-filtros)
    - [6.2 La tabla de resultados](capitulo-07-pacientes-y-recetas.md#62-la-tabla-de-resultados)
    - [6.3 Ver un paciente](capitulo-07-pacientes-y-recetas.md#63-ver-un-paciente)
    - [6.4 Editar un paciente](capitulo-07-pacientes-y-recetas.md#64-editar-un-paciente)
  - [7. Exportar y cargar pacientes desde Excel](capitulo-07-pacientes-y-recetas.md#7-exportar-y-cargar-pacientes-desde-excel)
    - [7.1 Exportar](capitulo-07-pacientes-y-recetas.md#71-exportar)
    - [7.2 Cargar muchos pacientes (Carga Masiva)](capitulo-07-pacientes-y-recetas.md#72-cargar-muchos-pacientes-carga-masiva)
  - [8. Eliminar un paciente](capitulo-07-pacientes-y-recetas.md#8-eliminar-un-paciente)
    - [8.1 Un paciente de prueba sin recetas](capitulo-07-pacientes-y-recetas.md#81-un-paciente-de-prueba-sin-recetas)
    - [8.2 Un paciente que tiene recetas](capitulo-07-pacientes-y-recetas.md#82-un-paciente-que-tiene-recetas)
  - [9. Profesionales de salud](capitulo-07-pacientes-y-recetas.md#9-profesionales-de-salud)
    - [9.1 Registrar un profesional de salud](capitulo-07-pacientes-y-recetas.md#91-registrar-un-profesional-de-salud)
    - [9.2 Buscar, ver y editar](capitulo-07-pacientes-y-recetas.md#92-buscar-ver-y-editar)
    - [9.3 Exportar](capitulo-07-pacientes-y-recetas.md#93-exportar)
    - [9.4 Cargar profesionales desde Excel](capitulo-07-pacientes-y-recetas.md#94-cargar-profesionales-desde-excel)
    - [9.5 Eliminar un profesional de salud](capitulo-07-pacientes-y-recetas.md#95-eliminar-un-profesional-de-salud)
  - [10. Registrar una receta](capitulo-07-pacientes-y-recetas.md#10-registrar-una-receta)
    - [10.1 Pasos](capitulo-07-pacientes-y-recetas.md#101-pasos)
    - [10.2 Qué pasa al terminar bien](capitulo-07-pacientes-y-recetas.md#102-qué-pasa-al-terminar-bien)
    - [10.3 Errores que puedes ver](capitulo-07-pacientes-y-recetas.md#103-errores-que-puedes-ver)
  - [11. Buscar, ver, editar, eliminar y exportar recetas](capitulo-07-pacientes-y-recetas.md#11-buscar-ver-editar-eliminar-y-exportar-recetas)
    - [11.1 Los filtros](capitulo-07-pacientes-y-recetas.md#111-los-filtros)
    - [11.2 La tabla de resultados](capitulo-07-pacientes-y-recetas.md#112-la-tabla-de-resultados)
    - [11.3 Ver y editar una receta](capitulo-07-pacientes-y-recetas.md#113-ver-y-editar-una-receta)
    - [11.4 Eliminar una receta](capitulo-07-pacientes-y-recetas.md#114-eliminar-una-receta)
    - [11.5 Exportar](capitulo-07-pacientes-y-recetas.md#115-exportar)
  - [12. El Lector de Recetas IA (escáner)](capitulo-07-pacientes-y-recetas.md#12-el-lector-de-recetas-ia-escáner)
    - [12.1 La pantalla](capitulo-07-pacientes-y-recetas.md#121-la-pantalla)
    - [12.2 Pasos](capitulo-07-pacientes-y-recetas.md#122-pasos)
    - [12.3 Qué muestran los resultados (comprobado con esa imagen)](capitulo-07-pacientes-y-recetas.md#123-qué-muestran-los-resultados-comprobado-con-esa-imagen)
    - [12.4 Lo que no se pudo comprobar en la aplicación](capitulo-07-pacientes-y-recetas.md#124-lo-que-no-se-pudo-comprobar-en-la-aplicación)
  - [13. Lo que ve un cajero frente a un administrador](capitulo-07-pacientes-y-recetas.md#13-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [14. Diferencias con la documentación técnica](capitulo-07-pacientes-y-recetas.md#14-diferencias-con-la-documentación-técnica)
  - [15. Pendientes y «No verificado»](capitulo-07-pacientes-y-recetas.md#15-pendientes-y-no-verificado)
  - [16. Tabla de cobertura](capitulo-07-pacientes-y-recetas.md#16-tabla-de-cobertura)
    - [16.1 Pacientes — búsqueda /pacientes (`producto/pantallas/inventario-y-atencion` §3.1)](capitulo-07-pacientes-y-recetas.md#161-pacientes--búsqueda-pacientes-productopantallasinventario-y-atencion-31)
    - [16.2 Pacientes — detalle /pacientes/nueva y /pacientes/:id (`producto/pantallas/inventario-y-atencion` §3.2)](capitulo-07-pacientes-y-recetas.md#162-pacientes--detalle-pacientesnueva-y-pacientesid-productopantallasinventario-y-atencion-32)
    - [16.3 Pacientes — carga masiva /pacientes/carga-masiva (`producto/pantallas/inventario-y-atencion` §3.3)](capitulo-07-pacientes-y-recetas.md#163-pacientes--carga-masiva-pacientescarga-masiva-productopantallasinventario-y-atencion-33)
    - [16.4 Profesionales de salud — búsqueda, detalle y carga masiva (`producto/pantallas/inventario-y-atencion` §4.1 a §4.3)](capitulo-07-pacientes-y-recetas.md#164-profesionales-de-salud--búsqueda-detalle-y-carga-masiva-productopantallasinventario-y-atencion-41-a-43)
    - [16.5 Recetas médicas — búsqueda, detalle y escáner (`producto/pantallas/inventario-y-atencion` §5.1 a §5.3)](capitulo-07-pacientes-y-recetas.md#165-recetas-médicas--búsqueda-detalle-y-escáner-productopantallasinventario-y-atencion-51-a-53)
    - [16.6 Comportamientos comunes y reglas](capitulo-07-pacientes-y-recetas.md#166-comportamientos-comunes-y-reglas)
  - [17. Fuentes](capitulo-07-pacientes-y-recetas.md#17-fuentes)
- **[Capítulo 8 — Caja y nueva venta](capitulo-08-caja-y-venta.md)**
  - [1. Introducción](capitulo-08-caja-y-venta.md#1-introducción)
  - [2. Antes de empezar](capitulo-08-caja-y-venta.md#2-antes-de-empezar)
  - [3. Cómo se relacionan estas piezas (en lenguaje simple)](capitulo-08-caja-y-venta.md#3-cómo-se-relacionan-estas-piezas-en-lenguaje-simple)
  - [4. Los datos que se crean en este capítulo](capitulo-08-caja-y-venta.md#4-los-datos-que-se-crean-en-este-capítulo)
  - [5. Abrir tu caja](capitulo-08-caja-y-venta.md#5-abrir-tu-caja)
    - [5.1 Abrir la caja desde el terminal (diálogo «Aperturar Caja»)](capitulo-08-caja-y-venta.md#51-abrir-la-caja-desde-el-terminal-diálogo-aperturar-caja)
    - [5.2 Abrir la caja desde Sesiones de Caja y ver tus sesiones](capitulo-08-caja-y-venta.md#52-abrir-la-caja-desde-sesiones-de-caja-y-ver-tus-sesiones)
  - [6. Qué pasa si intentas vender sin caja](capitulo-08-caja-y-venta.md#6-qué-pasa-si-intentas-vender-sin-caja)
  - [7. El terminal de venta: cómo es la pantalla](capitulo-08-caja-y-venta.md#7-el-terminal-de-venta-cómo-es-la-pantalla)
  - [8. Buscar un producto y elegir su lote y su presentación](capitulo-08-caja-y-venta.md#8-buscar-un-producto-y-elegir-su-lote-y-su-presentación)
    - [8.1 Buscar](capitulo-08-caja-y-venta.md#81-buscar)
    - [8.2 Elegir el lote](capitulo-08-caja-y-venta.md#82-elegir-el-lote)
    - [8.3 Elegir presentación y cantidad](capitulo-08-caja-y-venta.md#83-elegir-presentación-y-cantidad)
  - [9. El carrito: cantidades, quitar líneas y totales](capitulo-08-caja-y-venta.md#9-el-carrito-cantidades-quitar-líneas-y-totales)
  - [10. El cliente (opcional) y la búsqueda de clientes](capitulo-08-caja-y-venta.md#10-el-cliente-opcional-y-la-búsqueda-de-clientes)
  - [11. Vender medicamentos con receta](capitulo-08-caja-y-venta.md#11-vender-medicamentos-con-receta)
    - [11.1 Receta simple: con la casilla «Receta verificada»](capitulo-08-caja-y-venta.md#111-receta-simple-con-la-casilla-receta-verificada)
    - [11.2 Receta simple con receta registrada](capitulo-08-caja-y-venta.md#112-receta-simple-con-receta-registrada)
    - [11.3 Receta retenida y estupefaciente: exigen número de receta](capitulo-08-caja-y-venta.md#113-receta-retenida-y-estupefaciente-exigen-número-de-receta)
    - [11.4 Sin receta](capitulo-08-caja-y-venta.md#114-sin-receta)
    - [11.5 La receta ya se usó el máximo de veces](capitulo-08-caja-y-venta.md#115-la-receta-ya-se-usó-el-máximo-de-veces)
    - [11.6 Con la política «exigir registro de receta» encendida](capitulo-08-caja-y-venta.md#116-con-la-política-exigir-registro-de-receta-encendida)
  - [12. Cobrar la venta](capitulo-08-caja-y-venta.md#12-cobrar-la-venta)
    - [12.1 Cobrar en efectivo, con vuelto](capitulo-08-caja-y-venta.md#121-cobrar-en-efectivo-con-vuelto)
    - [12.2 Cobrar con tarjeta, Yape / Plin o transferencia](capitulo-08-caja-y-venta.md#122-cobrar-con-tarjeta-yape--plin-o-transferencia)
    - [12.3 Dividir el pago entre varios métodos](capitulo-08-caja-y-venta.md#123-dividir-el-pago-entre-varios-métodos)
  - [13. Rechazos y errores al cobrar](capitulo-08-caja-y-venta.md#13-rechazos-y-errores-al-cobrar)
    - [13.1 Lo que el terminal te impide antes de cobrar (comprobado)](capitulo-08-caja-y-venta.md#131-lo-que-el-terminal-te-impide-antes-de-cobrar-comprobado)
    - [13.2 Lo que el servidor puede rechazar al confirmar](capitulo-08-caja-y-venta.md#132-lo-que-el-servidor-puede-rechazar-al-confirmar)
  - [14. Confirmar, ver el comprobante y reimprimirlo](capitulo-08-caja-y-venta.md#14-confirmar-ver-el-comprobante-y-reimprimirlo)
    - [14.1 El diálogo de confirmación](capitulo-08-caja-y-venta.md#141-el-diálogo-de-confirmación)
    - [14.2 El diálogo del comprobante](capitulo-08-caja-y-venta.md#142-el-diálogo-del-comprobante)
    - [14.3 El ticket y la reimpresión](capitulo-08-caja-y-venta.md#143-el-ticket-y-la-reimpresión)
  - [15. Dejar una venta en espera y retomarla](capitulo-08-caja-y-venta.md#15-dejar-una-venta-en-espera-y-retomarla)
    - [15.1 Guardar la venta en espera](capitulo-08-caja-y-venta.md#151-guardar-la-venta-en-espera)
    - [15.2 Retomarla y cobrarla](capitulo-08-caja-y-venta.md#152-retomarla-y-cobrarla)
  - [16. El borrador del carrito (si recargas la página)](capitulo-08-caja-y-venta.md#16-el-borrador-del-carrito-si-recargas-la-página)
  - [17. La pestaña «Receta» (lector de recetas con IA)](capitulo-08-caja-y-venta.md#17-la-pestaña-receta-lector-de-recetas-con-ia)
  - [18. Lo que ve un cajero frente a un administrador](capitulo-08-caja-y-venta.md#18-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [19. Diferencias con la documentación técnica](capitulo-08-caja-y-venta.md#19-diferencias-con-la-documentación-técnica)
  - [20. Pendientes y «No verificado»](capitulo-08-caja-y-venta.md#20-pendientes-y-no-verificado)
  - [21. Tabla de cobertura](capitulo-08-caja-y-venta.md#21-tabla-de-cobertura)
    - [21.1 Comportamientos comunes (`producto/pantallas/punto-de-venta` §0)](capitulo-08-caja-y-venta.md#211-comportamientos-comunes-productopantallaspunto-de-venta-0)
    - [21.2 Sesiones de caja: lista y apertura (`producto/pantallas/punto-de-venta` §1.1, §1.2)](capitulo-08-caja-y-venta.md#212-sesiones-de-caja-lista-y-apertura-productopantallaspunto-de-venta-11-12)
    - [21.3 Diálogo de apertura de caja (`producto/pantallas/punto-de-venta` §2)](capitulo-08-caja-y-venta.md#213-diálogo-de-apertura-de-caja-productopantallaspunto-de-venta-2)
    - [21.4 Terminal de venta (`producto/pantallas/punto-de-venta` §3)](capitulo-08-caja-y-venta.md#214-terminal-de-venta-productopantallaspunto-de-venta-3)
    - [21.5 Confirmación y comprobante (`producto/pantallas/punto-de-venta` §4, §5)](capitulo-08-caja-y-venta.md#215-confirmación-y-comprobante-productopantallaspunto-de-venta-4-5)
  - [22. Fuentes](capitulo-08-caja-y-venta.md#22-fuentes)
- **[Capítulo 9 — Ventas, anulación, devolución y cierre de caja](capitulo-09-ventas-devoluciones-y-cierre.md)**
  - [1. Introducción](capitulo-09-ventas-devoluciones-y-cierre.md#1-introducción)
  - [2. Antes de empezar](capitulo-09-ventas-devoluciones-y-cierre.md#2-antes-de-empezar)
  - [3. Buscar ventas](capitulo-09-ventas-devoluciones-y-cierre.md#3-buscar-ventas)
    - [Pasos](capitulo-09-ventas-devoluciones-y-cierre.md#pasos)
    - [Filtros](capitulo-09-ventas-devoluciones-y-cierre.md#filtros)
    - [La tabla de resultados](capitulo-09-ventas-devoluciones-y-cierre.md#la-tabla-de-resultados)
    - [Lo que se comprobó en la aplicación](capitulo-09-ventas-devoluciones-y-cierre.md#lo-que-se-comprobó-en-la-aplicación)
    - [Exportar a Excel](capitulo-09-ventas-devoluciones-y-cierre.md#exportar-a-excel)
    - [Errores que puedes ver](capitulo-09-ventas-devoluciones-y-cierre.md#errores-que-puedes-ver)
  - [4. Ver el detalle de una venta y reimprimir el comprobante](capitulo-09-ventas-devoluciones-y-cierre.md#4-ver-el-detalle-de-una-venta-y-reimprimir-el-comprobante)
    - [Qué muestra la pantalla «Detalle de Venta»](capitulo-09-ventas-devoluciones-y-cierre.md#qué-muestra-la-pantalla-detalle-de-venta)
    - [Reimprimir el comprobante](capitulo-09-ventas-devoluciones-y-cierre.md#reimprimir-el-comprobante)
    - [Errores que puedes ver](capitulo-09-ventas-devoluciones-y-cierre.md#errores-que-puedes-ver-1)
  - [5. Anular una venta cobrada](capitulo-09-ventas-devoluciones-y-cierre.md#5-anular-una-venta-cobrada)
    - [Por qué hace falta tu caja abierta](capitulo-09-ventas-devoluciones-y-cierre.md#por-qué-hace-falta-tu-caja-abierta)
    - [Pasos](capitulo-09-ventas-devoluciones-y-cierre.md#pasos-1)
    - [Qué pasa al terminar](capitulo-09-ventas-devoluciones-y-cierre.md#qué-pasa-al-terminar)
    - [Si no ves el botón](capitulo-09-ventas-devoluciones-y-cierre.md#si-no-ves-el-botón)
    - [Errores que puedes ver](capitulo-09-ventas-devoluciones-y-cierre.md#errores-que-puedes-ver-2)
  - [6. Anular una venta en espera](capitulo-09-ventas-devoluciones-y-cierre.md#6-anular-una-venta-en-espera)
    - [Pasos](capitulo-09-ventas-devoluciones-y-cierre.md#pasos-2)
    - [Qué pasa al terminar](capitulo-09-ventas-devoluciones-y-cierre.md#qué-pasa-al-terminar-1)
  - [7. Devolver productos](capitulo-09-ventas-devoluciones-y-cierre.md#7-devolver-productos)
    - [Pasos](capitulo-09-ventas-devoluciones-y-cierre.md#pasos-3)
    - [Tabla de líneas](capitulo-09-ventas-devoluciones-y-cierre.md#tabla-de-líneas)
    - [Campos del formulario](capitulo-09-ventas-devoluciones-y-cierre.md#campos-del-formulario)
    - [Qué pasa al terminar](capitulo-09-ventas-devoluciones-y-cierre.md#qué-pasa-al-terminar-2)
    - [Lo que se comprobó en la aplicación](capitulo-09-ventas-devoluciones-y-cierre.md#lo-que-se-comprobó-en-la-aplicación-1)
    - [Si no ves el botón Devolver](capitulo-09-ventas-devoluciones-y-cierre.md#si-no-ves-el-botón-devolver)
    - [Errores que puedes ver](capitulo-09-ventas-devoluciones-y-cierre.md#errores-que-puedes-ver-3)
  - [8. Cerrar tu caja con arqueo](capitulo-09-ventas-devoluciones-y-cierre.md#8-cerrar-tu-caja-con-arqueo)
    - [Cuánto efectivo debería haber (la fórmula)](capitulo-09-ventas-devoluciones-y-cierre.md#cuánto-efectivo-debería-haber-la-fórmula)
    - [Pasos](capitulo-09-ventas-devoluciones-y-cierre.md#pasos-4)
    - [Qué pasa al terminar](capitulo-09-ventas-devoluciones-y-cierre.md#qué-pasa-al-terminar-3)
    - [Resultado del cierre de la prueba](capitulo-09-ventas-devoluciones-y-cierre.md#resultado-del-cierre-de-la-prueba)
    - [Errores que puedes ver](capitulo-09-ventas-devoluciones-y-cierre.md#errores-que-puedes-ver-4)
    - [8.1 Mostrar el estado «Cuadrada»: una segunda caja corta](capitulo-09-ventas-devoluciones-y-cierre.md#81-mostrar-el-estado-cuadrada-una-segunda-caja-corta)
    - [Dos avisos de la apertura (ya probados en el capítulo 8)](capitulo-09-ventas-devoluciones-y-cierre.md#dos-avisos-de-la-apertura-ya-probados-en-el-capítulo-8)
  - [9. Lista y detalle de sesiones de caja](capitulo-09-ventas-devoluciones-y-cierre.md#9-lista-y-detalle-de-sesiones-de-caja)
    - [Filtros](capitulo-09-ventas-devoluciones-y-cierre.md#filtros-1)
    - [La tabla](capitulo-09-ventas-devoluciones-y-cierre.md#la-tabla)
    - [El detalle de una sesión](capitulo-09-ventas-devoluciones-y-cierre.md#el-detalle-de-una-sesión)
    - [Exportar a Excel](capitulo-09-ventas-devoluciones-y-cierre.md#exportar-a-excel-1)
    - [Errores que puedes ver](capitulo-09-ventas-devoluciones-y-cierre.md#errores-que-puedes-ver-5)
  - [10. Lo que ve un cajero frente a un administrador](capitulo-09-ventas-devoluciones-y-cierre.md#10-lo-que-ve-un-cajero-frente-a-un-administrador)
  - [11. Diferencias con la documentación técnica](capitulo-09-ventas-devoluciones-y-cierre.md#11-diferencias-con-la-documentación-técnica)
  - [12. Pendientes y «No verificado»](capitulo-09-ventas-devoluciones-y-cierre.md#12-pendientes-y-no-verificado)
  - [13. Lista de cobertura](capitulo-09-ventas-devoluciones-y-cierre.md#13-lista-de-cobertura)
  - [14. Fuentes](capitulo-09-ventas-devoluciones-y-cierre.md#14-fuentes)
- **[Capítulo 10 — El Inicio y lo que ve un administrador frente a un cajero](capitulo-10-inicio-y-roles.md)**
  - [1. Introducción](capitulo-10-inicio-y-roles.md#1-introducción)
  - [2. Antes de empezar](capitulo-10-inicio-y-roles.md#2-antes-de-empezar)
  - [3. Leer el Inicio como administrador](capitulo-10-inicio-y-roles.md#3-leer-el-inicio-como-administrador)
    - [3.1 Cómo es la pantalla](capitulo-10-inicio-y-roles.md#31-cómo-es-la-pantalla)
    - [3.2 Cabecera](capitulo-10-inicio-y-roles.md#32-cabecera)
    - [3.3 Ventas de hoy](capitulo-10-inicio-y-roles.md#33-ventas-de-hoy)
    - [3.4 Caja](capitulo-10-inicio-y-roles.md#34-caja)
    - [3.5 Inventario](capitulo-10-inicio-y-roles.md#35-inventario)
    - [3.6 Tendencia, más vendidos y devoluciones](capitulo-10-inicio-y-roles.md#36-tendencia-más-vendidos-y-devoluciones)
    - [3.7 Cada indicador: valor visto frente a valor esperado](capitulo-10-inicio-y-roles.md#37-cada-indicador-valor-visto-frente-a-valor-esperado)
    - [3.8 El mismo Inicio con una venta y una caja abierta](capitulo-10-inicio-y-roles.md#38-el-mismo-inicio-con-una-venta-y-una-caja-abierta)
  - [4. Los botones de acceso directo](capitulo-10-inicio-y-roles.md#4-los-botones-de-acceso-directo)
  - [5. Ventas por sucursal](capitulo-10-inicio-y-roles.md#5-ventas-por-sucursal)
  - [6. El Inicio como cajero](capitulo-10-inicio-y-roles.md#6-el-inicio-como-cajero)
    - [6.1 Qué cambia respecto del administrador](capitulo-10-inicio-y-roles.md#61-qué-cambia-respecto-del-administrador)
    - [6.2 Abrir una caja corta y ver el «Efectivo esperado» en vivo](capitulo-10-inicio-y-roles.md#62-abrir-una-caja-corta-y-ver-el-efectivo-esperado-en-vivo)
  - [7. Cuándo se actualizan los indicadores](capitulo-10-inicio-y-roles.md#7-cuándo-se-actualizan-los-indicadores)
  - [8. Administrador frente a cajero en toda la aplicación](capitulo-10-inicio-y-roles.md#8-administrador-frente-a-cajero-en-toda-la-aplicación)
    - [8.1 Menú lateral](capitulo-10-inicio-y-roles.md#81-menú-lateral)
    - [8.2 Qué puede hacer cada uno, pantalla por pantalla](capitulo-10-inicio-y-roles.md#82-qué-puede-hacer-cada-uno-pantalla-por-pantalla)
    - [8.3 Lo que solo puede hacer un administrador](capitulo-10-inicio-y-roles.md#83-lo-que-solo-puede-hacer-un-administrador)
  - [9. Diferencias con la documentación técnica](capitulo-10-inicio-y-roles.md#9-diferencias-con-la-documentación-técnica)
  - [10. Pendientes y «No verificado»](capitulo-10-inicio-y-roles.md#10-pendientes-y-no-verificado)
  - [11. Tabla de cobertura](capitulo-10-inicio-y-roles.md#11-tabla-de-cobertura)
    - [11.1 /dashboard (`producto/pantallas/acceso-layout-y-compartidos` §3)](capitulo-10-inicio-y-roles.md#111-dashboard-productopantallasacceso-layout-y-compartidos-3)
    - [11.2 Reglas RN-DAS y partes de `producto/pantallas/acceso-layout-y-compartidos` §4 y §5](capitulo-10-inicio-y-roles.md#112-reglas-rn-das-y-partes-de-productopantallasacceso-layout-y-compartidos-4-y-5)
  - [12. Fuentes](capitulo-10-inicio-y-roles.md#12-fuentes)
- **[Capítulo 11 — Problemas frecuentes y cómo resolverlos](capitulo-11-problemas-frecuentes.md)**
  - [1. Cómo usar este capítulo](capitulo-11-problemas-frecuentes.md#1-cómo-usar-este-capítulo)
  - [2. Problemas por situación (sin mensaje de error)](capitulo-11-problemas-frecuentes.md#2-problemas-por-situación-sin-mensaje-de-error)
    - [2.1 No veo una opción del menú o un botón](capitulo-11-problemas-frecuentes.md#21-no-veo-una-opción-del-menú-o-un-botón)
    - [2.2 La pantalla me saca al inicio de sesión](capitulo-11-problemas-frecuentes.md#22-la-pantalla-me-saca-al-inicio-de-sesión)
    - [2.3 No puedo entrar](capitulo-11-problemas-frecuentes.md#23-no-puedo-entrar)
    - [2.4 Cambié el idioma y volvió a español](capitulo-11-problemas-frecuentes.md#24-cambié-el-idioma-y-volvió-a-español)
    - [2.5 No puedo cerrar mi caja](capitulo-11-problemas-frecuentes.md#25-no-puedo-cerrar-mi-caja)
    - [2.6 El cajero no ve ventas de otra sucursal](capitulo-11-problemas-frecuentes.md#26-el-cajero-no-ve-ventas-de-otra-sucursal)
    - [2.7 La boleta dice «Pendiente de envío a SUNAT»](capitulo-11-problemas-frecuentes.md#27-la-boleta-dice-pendiente-de-envío-a-sunat)
    - [2.8 La exportación sale con la fecha del día siguiente](capitulo-11-problemas-frecuentes.md#28-la-exportación-sale-con-la-fecha-del-día-siguiente)
    - [2.9 Otros comportamientos que sorprenden](capitulo-11-problemas-frecuentes.md#29-otros-comportamientos-que-sorprenden)
  - [3. Mensajes de error por tarea](capitulo-11-problemas-frecuentes.md#3-mensajes-de-error-por-tarea)
    - [3.0 Avisos de base de datos y de simultaneidad](capitulo-11-problemas-frecuentes.md#30-avisos-de-base-de-datos-y-de-simultaneidad)
    - [3.1 Acceso y sesión](capitulo-11-problemas-frecuentes.md#31-acceso-y-sesión)
    - [3.2 Empresa y sucursales](capitulo-11-problemas-frecuentes.md#32-empresa-y-sucursales)
    - [3.3 Series fiscales](capitulo-11-problemas-frecuentes.md#33-series-fiscales)
    - [3.4 Empleados](capitulo-11-problemas-frecuentes.md#34-empleados)
    - [3.5 Roles y permisos](capitulo-11-problemas-frecuentes.md#35-roles-y-permisos)
    - [3.6 Cuentas de usuario](capitulo-11-problemas-frecuentes.md#36-cuentas-de-usuario)
    - [3.7 Catálogos: unidades, laboratorios, ingredientes, DIGEMID, productos, proveedores y aseguradoras](capitulo-11-problemas-frecuentes.md#37-catálogos-unidades-laboratorios-ingredientes-digemid-productos-proveedores-y-aseguradoras)
    - [3.8 Carga masiva y exportación a Excel](capitulo-11-problemas-frecuentes.md#38-carga-masiva-y-exportación-a-excel)
    - [3.9 Inventario: lotes y stock por sucursal](capitulo-11-problemas-frecuentes.md#39-inventario-lotes-y-stock-por-sucursal)
    - [3.10 Pacientes, profesionales de salud y recetas](capitulo-11-problemas-frecuentes.md#310-pacientes-profesionales-de-salud-y-recetas)
    - [3.11 Caja: apertura y cierre](capitulo-11-problemas-frecuentes.md#311-caja-apertura-y-cierre)
    - [3.12 Venta y cobro](capitulo-11-problemas-frecuentes.md#312-venta-y-cobro)
    - [3.13 Anulación de una venta](capitulo-11-problemas-frecuentes.md#313-anulación-de-una-venta)
    - [3.14 Devolución de una venta](capitulo-11-problemas-frecuentes.md#314-devolución-de-una-venta)
    - [3.15 Dashboard (Inicio)](capitulo-11-problemas-frecuentes.md#315-dashboard-inicio)
  - [4. Cuándo pedir ayuda al administrador o a soporte](capitulo-11-problemas-frecuentes.md#4-cuándo-pedir-ayuda-al-administrador-o-a-soporte)
  - [5. Índice alfabético de mensajes](capitulo-11-problemas-frecuentes.md#5-índice-alfabético-de-mensajes)
  - [6. Pendientes y «No verificado»](capitulo-11-problemas-frecuentes.md#6-pendientes-y-no-verificado)
  - [7. Tabla de cobertura](capitulo-11-problemas-frecuentes.md#7-tabla-de-cobertura)
    - [7.1 Mensajes de las tablas «Mensajes al usuario» de `producto/pantallas/acceso-layout-y-compartidos` a `producto/pantallas/punto-de-venta`](capitulo-11-problemas-frecuentes.md#71-mensajes-de-las-tablas-mensajes-al-usuario-de-productopantallasacceso-layout-y-compartidos-a-productopantallaspunto-de-venta)
    - [7.2 Tareas y situaciones pedidas en el encargo](capitulo-11-problemas-frecuentes.md#72-tareas-y-situaciones-pedidas-en-el-encargo)
  - [8. Fuentes](capitulo-11-problemas-frecuentes.md#8-fuentes)

## 4. Tareas frecuentes: en qué capítulo y sección está cada una

Busca lo que quieres hacer. Cada fila te lleva a la sección con los pasos.

### 4.1 Primeros pasos y acceso

| Quiero… | Dónde |
|---|---|
| Registrar mi empresa | [Capítulo 1, §3 Registrar tu empresa](capitulo-01-primeros-pasos.md#3-registrar-tu-empresa) |
| Iniciar sesión | [Capítulo 1, §6 Iniciar sesión](capitulo-01-primeros-pasos.md#6-iniciar-sesión) |
| Cerrar sesión | [Capítulo 1, §5 Cerrar sesión](capitulo-01-primeros-pasos.md#5-cerrar-sesión) |
| Cambiar mi contraseña | [Capítulo 1, §7 Cambiar tu contraseña](capitulo-01-primeros-pasos.md#7-cambiar-tu-contraseña) |
| Recuperar mi contraseña olvidada | [Capítulo 1, §8 Recuperar tu contraseña olvidada](capitulo-01-primeros-pasos.md#8-recuperar-tu-contraseña-olvidada) |
| Entender qué pasa cuando mi sesión vence | [Capítulo 1, §9 Sesión vencida y cambio obligatorio de contraseña](capitulo-01-primeros-pasos.md#9-sesión-vencida-y-cambio-obligatorio-de-contraseña) |
| Cambiar el idioma | [Capítulo 1, §4.3 Cambiar el idioma](capitulo-01-primeros-pasos.md#43-cambiar-el-idioma) |
| Reconocer el menú lateral y la cabecera | [Capítulo 1, §4 Conocer la pantalla principal](capitulo-01-primeros-pasos.md#4-conocer-la-pantalla-principal) |

### 4.2 Configurar la botica (administrador)

| Quiero… | Dónde |
|---|---|
| Cambiar el nombre o el RUC de mi empresa | [Capítulo 2, §3.3 Editar el nombre o el RUC](capitulo-02-empresa-sucursales-series.md#33-editar-el-nombre-o-el-ruc) |
| Completar la sede principal | [Capítulo 2, §4 Completar la sede principal](capitulo-02-empresa-sucursales-series.md#4-completar-la-sede-principal) |
| Crear una sucursal | [Capítulo 2, §5 Crear una segunda sucursal](capitulo-02-empresa-sucursales-series.md#5-crear-una-segunda-sucursal) |
| Cargar sucursales desde un Excel | [Capítulo 2, §8 Cargar sucursales desde un Excel (carga masiva)](capitulo-02-empresa-sucursales-series.md#8-cargar-sucursales-desde-un-excel-carga-masiva) |
| Eliminar una sucursal | [Capítulo 2, §7 Eliminar una sucursal](capitulo-02-empresa-sucursales-series.md#7-eliminar-una-sucursal) |
| Saber qué series fiscales crear | [Capítulo 2, §9.3 Qué series crear](capitulo-02-empresa-sucursales-series.md#93-qué-series-crear) |
| Crear una serie fiscal (boleta o nota de crédito) | [Capítulo 2, §10 Crear una serie fiscal](capitulo-02-empresa-sucursales-series.md#10-crear-una-serie-fiscal) |
| Desactivar o eliminar una serie fiscal | [Capítulo 2, §11.3 Desactivar una serie](capitulo-02-empresa-sucursales-series.md#113-desactivar-una-serie) y [Capítulo 2, §11.4 Eliminar una serie](capitulo-02-empresa-sucursales-series.md#114-eliminar-una-serie) |
| Registrar un empleado | [Capítulo 3, §3.2 Crear un empleado](capitulo-03-empleados-roles-cuentas.md#32-crear-un-empleado) |
| Entender qué es un rol y sus permisos | [Capítulo 3, §4 Entender los roles y los permisos](capitulo-03-empleados-roles-cuentas.md#4-entender-los-roles-y-los-permisos) |
| Crear el rol «Cajero» y marcar sus permisos | [Capítulo 3, §5 Crear el rol «Cajero» y marcar sus permisos](capitulo-03-empleados-roles-cuentas.md#5-crear-el-rol-cajero-y-marcar-sus-permisos) |
| **Crear un usuario cajero** (la cuenta con su rol) | [Capítulo 3, §7.2 Crear la cuenta](capitulo-03-empleados-roles-cuentas.md#72-crear-la-cuenta) y [Capítulo 3, §7.3 Asignar el rol «Cajero»](capitulo-03-empleados-roles-cuentas.md#73-asignar-el-rol-cajero) |
| Activar o desactivar una cuenta, forzar el cambio de contraseña o resetear intentos | [Capítulo 3, §7.5 Activar, desactivar, forzar el cambio de contraseña y resetear intentos](capitulo-03-empleados-roles-cuentas.md#75-activar-desactivar-forzar-el-cambio-de-contraseña-y-resetear-intentos) |
| Entender el bloqueo automático de una cuenta | [Capítulo 3, §7.6 El bloqueo automático](capitulo-03-empleados-roles-cuentas.md#76-el-bloqueo-automático) |
| Dar acceso a otra sucursal a una cuenta | [Capítulo 3, §7.7 Agregar y quitar sucursales de una cuenta](capitulo-03-empleados-roles-cuentas.md#77-agregar-y-quitar-sucursales-de-una-cuenta) |
| Ver qué ve un cajero frente a un administrador | [Capítulo 3, §11 Lo que ve un cajero frente a un administrador](capitulo-03-empleados-roles-cuentas.md#11-lo-que-ve-un-cajero-frente-a-un-administrador) y [Capítulo 10, §8 Administrador frente a cajero en toda la aplicación](capitulo-10-inicio-y-roles.md#8-administrador-frente-a-cajero-en-toda-la-aplicación) |

### 4.3 Productos, proveedores e inventario (administrador)

| Quiero… | Dónde |
|---|---|
| Crear una unidad de medida | [Capítulo 4, §5.3 Crear una unidad](capitulo-04-catalogos-basicos.md#53-crear-una-unidad) |
| Crear un laboratorio | [Capítulo 4, §6.3 Crear un laboratorio](capitulo-04-catalogos-basicos.md#63-crear-un-laboratorio) |
| Crear un ingrediente activo | [Capítulo 4, §7.3 Crear un ingrediente activo](capitulo-04-catalogos-basicos.md#73-crear-un-ingrediente-activo) |
| Consultar el catálogo DIGEMID | [Capítulo 4, §8.3 Buscar y exportar](capitulo-04-catalogos-basicos.md#83-buscar-y-exportar) |
| **Cargar unidades, laboratorios o ingredientes en masa** (Excel) | [Capítulo 4, §9 Cargar unidades, laboratorios e ingredientes desde un Excel](capitulo-04-catalogos-basicos.md#9-cargar-unidades-laboratorios-e-ingredientes-desde-un-excel) |
| Crear un medicamento | [Capítulo 5, §4.4 Crear un medicamento](capitulo-05-productos-y-proveedores.md#44-crear-un-medicamento) |
| Entender la condición de venta de un medicamento (con o sin receta) | [Capítulo 5, §4.2 La condición de venta y qué implica en el punto de venta](capitulo-05-productos-y-proveedores.md#42-la-condición-de-venta-y-qué-implica-en-el-punto-de-venta) |
| **Cargar medicamentos en masa** (Excel) | [Capítulo 5, §10.3 Carga masiva de medicamentos](capitulo-05-productos-y-proveedores.md#103-carga-masiva-de-medicamentos) |
| Crear una presentación de venta (Caja, Blíster) | [Capítulo 5, §6.3 Crear una presentación](capitulo-05-productos-y-proveedores.md#63-crear-una-presentación) |
| Crear un proveedor y registrar su precio de compra | [Capítulo 5, §8.3 Crear un proveedor](capitulo-05-productos-y-proveedores.md#83-crear-un-proveedor) y [Capítulo 5, §8.5 Precios de compra](capitulo-05-productos-y-proveedores.md#85-precios-de-compra) |
| Eliminar algo que está en uso | [Capítulo 5, §10 Eliminar algo que está en uso y carga masiva](capitulo-05-productos-y-proveedores.md#10-eliminar-algo-que-está-en-uso-y-carga-masiva) |
| Crear un lote | [Capítulo 6, §5 Crear un lote](capitulo-06-inventario.md#5-crear-un-lote) |
| Ingresar stock de un lote en una sucursal | [Capítulo 6, §8 Ingresar stock](capitulo-06-inventario.md#8-ingresar-stock) |
| Consultar el stock por sucursal | [Capítulo 6, §9 Consultar el stock por sucursal](capitulo-06-inventario.md#9-consultar-el-stock-por-sucursal) |
| Ver lotes vencidos o por vencer | [Capítulo 6, §9.4 Ver lotes vencidos o por vencer desde el Dashboard](capitulo-06-inventario.md#94-ver-lotes-vencidos-o-por-vencer-desde-el-dashboard) |
| Ajustar una cantidad | [Capítulo 6, §10 Ajustar una cantidad](capitulo-06-inventario.md#10-ajustar-una-cantidad) |
| Eliminar un lote | [Capítulo 6, §11 Eliminar un lote](capitulo-06-inventario.md#11-eliminar-un-lote) |

### 4.4 Pacientes y recetas

| Quiero… | Dónde |
|---|---|
| Registrar un paciente | [Capítulo 7, §5 Registrar un paciente](capitulo-07-pacientes-y-recetas.md#5-registrar-un-paciente) |
| **Cargar pacientes en masa** (Excel) | [Capítulo 7, §7.2 Cargar muchos pacientes (Carga Masiva)](capitulo-07-pacientes-y-recetas.md#72-cargar-muchos-pacientes-carga-masiva) |
| Registrar un médico o profesional de salud | [Capítulo 7, §9.1 Registrar un profesional de salud](capitulo-07-pacientes-y-recetas.md#91-registrar-un-profesional-de-salud) |
| **Registrar una receta** | [Capítulo 7, §10 Registrar una receta](capitulo-07-pacientes-y-recetas.md#10-registrar-una-receta) |
| Buscar, editar o eliminar una receta | [Capítulo 7, §11 Buscar, ver, editar, eliminar y exportar recetas](capitulo-07-pacientes-y-recetas.md#11-buscar-ver-editar-eliminar-y-exportar-recetas) |
| Leer una receta con la inteligencia artificial | [Capítulo 7, §12 El Lector de Recetas IA (escáner)](capitulo-07-pacientes-y-recetas.md#12-el-lector-de-recetas-ia-escáner) |

### 4.5 Vender y cerrar el día (cajero)

| Quiero… | Dónde |
|---|---|
| **Abrir caja** | [Capítulo 8, §5 Abrir tu caja](capitulo-08-caja-y-venta.md#5-abrir-tu-caja) |
| Entender qué pasa si intento vender sin caja | [Capítulo 8, §6 Qué pasa si intentas vender sin caja](capitulo-08-caja-y-venta.md#6-qué-pasa-si-intentas-vender-sin-caja) |
| Buscar un producto y elegir su lote | [Capítulo 8, §8 Buscar un producto y elegir su lote y su presentación](capitulo-08-caja-y-venta.md#8-buscar-un-producto-y-elegir-su-lote-y-su-presentación) |
| Cambiar cantidades, quitar líneas y ver los totales | [Capítulo 8, §9 El carrito: cantidades, quitar líneas y totales](capitulo-08-caja-y-venta.md#9-el-carrito-cantidades-quitar-líneas-y-totales) |
| Elegir al cliente | [Capítulo 8, §10 El cliente (opcional) y la búsqueda de clientes](capitulo-08-caja-y-venta.md#10-el-cliente-opcional-y-la-búsqueda-de-clientes) |
| **Vender con receta** | [Capítulo 8, §11 Vender medicamentos con receta](capitulo-08-caja-y-venta.md#11-vender-medicamentos-con-receta) |
| Cobrar en efectivo con vuelto | [Capítulo 8, §12.1 Cobrar en efectivo, con vuelto](capitulo-08-caja-y-venta.md#121-cobrar-en-efectivo-con-vuelto) |
| Cobrar con tarjeta, Yape / Plin o transferencia | [Capítulo 8, §12.2 Cobrar con tarjeta, Yape / Plin o transferencia](capitulo-08-caja-y-venta.md#122-cobrar-con-tarjeta-yape--plin-o-transferencia) |
| Dividir el pago entre varios métodos | [Capítulo 8, §12.3 Dividir el pago entre varios métodos](capitulo-08-caja-y-venta.md#123-dividir-el-pago-entre-varios-métodos) |
| Ver el comprobante y reimprimirlo | [Capítulo 8, §14 Confirmar, ver el comprobante y reimprimirlo](capitulo-08-caja-y-venta.md#14-confirmar-ver-el-comprobante-y-reimprimirlo) y [Capítulo 9, §4 Ver el detalle de una venta y reimprimir el comprobante](capitulo-09-ventas-devoluciones-y-cierre.md#4-ver-el-detalle-de-una-venta-y-reimprimir-el-comprobante) |
| Dejar una venta en espera y retomarla | [Capítulo 8, §15 Dejar una venta en espera y retomarla](capitulo-08-caja-y-venta.md#15-dejar-una-venta-en-espera-y-retomarla) |
| Entender qué pasa si recargo la página en plena venta | [Capítulo 8, §16 El borrador del carrito (si recargas la página)](capitulo-08-caja-y-venta.md#16-el-borrador-del-carrito-si-recargas-la-página) |
| Buscar ventas y exportarlas | [Capítulo 9, §3 Buscar ventas](capitulo-09-ventas-devoluciones-y-cierre.md#3-buscar-ventas) |
| **Anular una venta** cobrada | [Capítulo 9, §5 Anular una venta cobrada](capitulo-09-ventas-devoluciones-y-cierre.md#5-anular-una-venta-cobrada) |
| Anular una venta en espera | [Capítulo 9, §6 Anular una venta en espera](capitulo-09-ventas-devoluciones-y-cierre.md#6-anular-una-venta-en-espera) |
| **Devolver productos** (se emite una nota de crédito) | [Capítulo 9, §7 Devolver productos](capitulo-09-ventas-devoluciones-y-cierre.md#7-devolver-productos) |
| **Cerrar la caja** con arqueo | [Capítulo 9, §8 Cerrar tu caja con arqueo](capitulo-09-ventas-devoluciones-y-cierre.md#8-cerrar-tu-caja-con-arqueo) |
| Ver mis sesiones de caja | [Capítulo 9, §9 Lista y detalle de sesiones de caja](capitulo-09-ventas-devoluciones-y-cierre.md#9-lista-y-detalle-de-sesiones-de-caja) |

### 4.6 Consultar y resolver problemas

| Quiero… | Dónde |
|---|---|
| Leer el Inicio como administrador | [Capítulo 10, §3 Leer el Inicio como administrador](capitulo-10-inicio-y-roles.md#3-leer-el-inicio-como-administrador) |
| Leer el Inicio como cajero | [Capítulo 10, §6 El Inicio como cajero](capitulo-10-inicio-y-roles.md#6-el-inicio-como-cajero) |
| Entender los botones de acceso directo del Inicio | [Capítulo 10, §4 Los botones de acceso directo](capitulo-10-inicio-y-roles.md#4-los-botones-de-acceso-directo) |
| Saber cada cuánto se actualizan las cifras | [Capítulo 10, §7 Cuándo se actualizan los indicadores](capitulo-10-inicio-y-roles.md#7-cuándo-se-actualizan-los-indicadores) |
| Saber qué puede hacer cada rol, pantalla por pantalla | [Capítulo 10, §8.2 Qué puede hacer cada uno, pantalla por pantalla](capitulo-10-inicio-y-roles.md#82-qué-puede-hacer-cada-uno-pantalla-por-pantalla) |
| No veo una opción del menú o un botón | [Capítulo 11, §2.1 No veo una opción del menú o un botón](capitulo-11-problemas-frecuentes.md#21-no-veo-una-opción-del-menú-o-un-botón) |
| No puedo entrar | [Capítulo 11, §2.3 No puedo entrar](capitulo-11-problemas-frecuentes.md#23-no-puedo-entrar) |
| No puedo cerrar mi caja | [Capítulo 11, §2.5 No puedo cerrar mi caja](capitulo-11-problemas-frecuentes.md#25-no-puedo-cerrar-mi-caja) |
| Entender por qué la boleta dice «Pendiente de envío a SUNAT» | [Capítulo 11, §2.7 La boleta dice «Pendiente de envío a SUNAT»](capitulo-11-problemas-frecuentes.md#27-la-boleta-dice-pendiente-de-envío-a-sunat) |
| Buscar un mensaje de error | [Capítulo 11, §3 Mensajes de error por tarea](capitulo-11-problemas-frecuentes.md#3-mensajes-de-error-por-tarea) y [Capítulo 11, §5 Índice alfabético de mensajes](capitulo-11-problemas-frecuentes.md#5-índice-alfabético-de-mensajes) |
| Saber cuándo pedir ayuda y qué datos dar | [Capítulo 11, §4 Cuándo pedir ayuda al administrador o a soporte](capitulo-11-problemas-frecuentes.md#4-cuándo-pedir-ayuda-al-administrador-o-a-soporte) |

## 5. Convenciones del manual

| Convención | Cómo se ve | Ejemplo |
|---|---|---|
| Botones | En **negrita**, con el texto de la pantalla | Haz clic en **Guardar** |
| Textos de pantalla (mensajes, títulos, etiquetas, columnas) | Entre comillas latinas «así», copiados tal como aparecen | «Sesión aperturada correctamente» |
| Menús | Con › entre niveles | Administración › Roles & Permisos |
| Permisos | En código, primero el módulo y luego la acción, como en la pantalla de roles | `Sesiones de Caja › Cerrar caja` |
| Lo que no se pudo comprobar | La marca «No verificado en la aplicación», con el motivo | «(No verificado en la aplicación: no se pudo provocar)» |
| Mensajes del capítulo 11 | «(Verificado en el capítulo N)» si el mensaje se vio en pantalla en ese capítulo; «(No verificado en la aplicación)» si no | Ver [Capítulo 11, §3 Mensajes de error por tarea](capitulo-11-problemas-frecuentes.md#3-mensajes-de-error-por-tarea) |
| Trabajo sin guardar en el repositorio | «En curso (sin commitear)» | Cada capítulo lo declara en sus pendientes |

Otras convenciones:

- **Sin capturas de pantalla.** El navegador integrado con el que se recorrió la aplicación no guarda capturas como archivo. En su lugar, cada pantalla se describe con precisión: qué tarjetas hay, en qué orden, qué botones aparecen arriba a la derecha y qué columnas tiene cada tabla.
- **Cada paso se ejecutó de verdad.** Los pasos y los textos de los capítulos se recorrieron en la aplicación local. Lo que no se pudo recorrer (por ejemplo, subir un archivo o provocar un fallo del servidor) se marca como «No verificado en la aplicación» y se apoya en el inventario de pantallas.
- **Datos de ejemplo.** Todos los capítulos usan una sola empresa ficticia, «Botica Manual Demo S.A.C.», con datos evidentemente inventados:

| Dato | Ejemplo |
|---|---|
| Administradora | «Ana Demo Prueba» (usuario `ana.manual`) |
| Cajero | «Carlos Cajero Demo» (usuario `carlos.cajero`) |
| Segundo empleado | «Nora Norte Demo» |
| Sucursales | «Sede Principal» y «Sucursal Demo Norte» |
| Series fiscales | Boleta `B001` y nota de crédito `BC01` en la Sede Principal; boleta `B002` y nota de crédito `BC02` en la Sucursal Demo Norte |
| Correos | Del dominio `sofia-manual.test` |
| Ventas | Boletas `B001-00000001` en adelante, hechas por el cajero en la Sede Principal |

- **Contraseñas.** El manual nunca escribe una contraseña: dice «tu contraseña» o «la contraseña que definiste».
- **Tuteo.** Todo el manual te habla de «tú» («haz clic», «puedes», «tienes»).
- **Importes y fechas.** Los importes van en soles (S/) con dos decimales. Las fechas se muestran como día/mes/año (`dd/MM/aaaa`) y las horas son las de tu navegador.

## 6. Glosario breve

Los términos propios de SOFIA y de la farmacia peruana (lote, serie fiscal, sesión de caja, arqueo, receta retenida, DIGEMID y otros) están definidos en el [glosario de la documentación](../negocio/glosario.md). Este manual no lo copia: ábrelo cuando encuentres una palabra que no conozcas.

Tres palabras que el manual usa todo el tiempo:

- **Sucursal base**: la sucursal donde trabaja un empleado. Las ventas, la caja y el terminal se trabajan en ella ([Capítulo 3, §5.5 Sucursales del rol](capitulo-03-empleados-roles-cuentas.md#55-sucursales-del-rol)).
- **Caja propia**: la sesión de caja que abriste tú. La caja abierta por otra persona no te sirve para vender, anular ni devolver ([Capítulo 8, §6 Qué pasa si intentas vender sin caja](capitulo-08-caja-y-venta.md#6-qué-pasa-si-intentas-vender-sin-caja)).
- **Permiso**: un par «módulo + acción» que el administrador marca para cada rol en Administración › Roles & Permisos ([Capítulo 3, §4 Entender los roles y los permisos](capitulo-03-empleados-roles-cuentas.md#4-entender-los-roles-y-los-permisos)).

## 7. Limitaciones conocidas que ve el usuario

Esta tabla reúne lo que los capítulos dejan a la vista al usar la aplicación: comportamientos que no esperarías, funciones que aún no existen y textos que no dicen la causa. Se anotan como los describe cada capítulo, sin opinar sobre si son errores. Para cada fila, el enlace lleva a la sección donde está explicado con el detalle.

### 7.1 Comprobantes, ventas y caja

| Limitación | Qué ves | Dónde |
|---|---|---|
| El envío de comprobantes a SUNAT no existe | Toda boleta queda con el estado «Pendiente de envío a SUNAT» | [Capítulo 8, §14.2 El diálogo del comprobante](capitulo-08-caja-y-venta.md#142-el-diálogo-del-comprobante); [Capítulo 11, §2.7 La boleta dice «Pendiente de envío a SUNAT»](capitulo-11-problemas-frecuentes.md#27-la-boleta-dice-pendiente-de-envío-a-sunat) |
| No se puede quitar el cliente elegido en la venta | No hay botón para quitarlo; solo se descarta al escribir otro texto. **Vaciar** tampoco lo descarta | [Capítulo 8, §10 El cliente (opcional) y la búsqueda de clientes](capitulo-08-caja-y-venta.md#10-el-cliente-opcional-y-la-búsqueda-de-clientes); [Capítulo 8, §19 Diferencias con la documentación técnica](capitulo-08-caja-y-venta.md#19-diferencias-con-la-documentación-técnica) |
| El terminal no muestra la fecha de vencimiento del lote | Solo ves el número y el stock; tú eliges el lote | [Capítulo 8, §8.2 Elegir el lote](capitulo-08-caja-y-venta.md#82-elegir-el-lote) |
| El terminal no registra pacientes ni recibe el número de receta | El cliente y su receta se registran antes, en Pacientes y Recetas Médicas | [Capítulo 8, §10 El cliente (opcional) y la búsqueda de clientes](capitulo-08-caja-y-venta.md#10-el-cliente-opcional-y-la-búsqueda-de-clientes); [Capítulo 8, §11.3 Receta retenida y estupefaciente: exigen número de receta](capitulo-08-caja-y-venta.md#113-receta-retenida-y-estupefaciente-exigen-número-de-receta) |
| El bloque de cobertura de seguro del terminal está oculto | No aparece en la pantalla; las aseguradoras que registres no se usan hoy en la venta | [Capítulo 5, §9.1 Qué es una aseguradora y para qué sirve hoy](capitulo-05-productos-y-proveedores.md#91-qué-es-una-aseguradora-y-para-qué-sirve-hoy); [Capítulo 8, §13.2 Lo que el servidor puede rechazar al confirmar](capitulo-08-caja-y-venta.md#132-lo-que-el-servidor-puede-rechazar-al-confirmar) |
| No se puede abrir caja en una sucursal que no sea tu sucursal base | La pantalla de apertura no tiene campo de sucursal; un administrador de varias sucursales vende solo en la suya | [Capítulo 10, §5 Ventas por sucursal](capitulo-10-inicio-y-roles.md#5-ventas-por-sucursal); [Capítulo 8, §5.2 Abrir la caja desde Sesiones de Caja y ver tus sesiones](capitulo-08-caja-y-venta.md#52-abrir-la-caja-desde-sesiones-de-caja-y-ver-tus-sesiones) |
| La lista de ventas, el detalle, el terminal, las anulaciones y las devoluciones trabajan solo con la sucursal base | Abrir una venta de otra sucursal da «Error al cargar la venta» | [Capítulo 11, §2.6 El cajero no ve ventas de otra sucursal](capitulo-11-problemas-frecuentes.md#26-el-cajero-no-ve-ventas-de-otra-sucursal); [Capítulo 9, §4 Ver el detalle de una venta y reimprimir el comprobante](capitulo-09-ventas-devoluciones-y-cierre.md#4-ver-el-detalle-de-una-venta-y-reimprimir-el-comprobante) |
| Anular una venta cobrada o devolver exige tu propia caja abierta, también al administrador | «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» y «Necesitas una caja abierta para procesar la devolución.» | [Capítulo 9, §5 Anular una venta cobrada](capitulo-09-ventas-devoluciones-y-cierre.md#5-anular-una-venta-cobrada); [Capítulo 9, §7 Devolver productos](capitulo-09-ventas-devoluciones-y-cierre.md#7-devolver-productos) |
| No existe la pantalla de Devoluciones | El aviso de una venta con devoluciones remite a «Devoluciones», que no es una opción del menú; las devoluciones se hacen desde el detalle de la venta | [Capítulo 9, §5 Anular una venta cobrada](capitulo-09-ventas-devoluciones-y-cierre.md#5-anular-una-venta-cobrada); [Capítulo 9, §11 Diferencias con la documentación técnica](capitulo-09-ventas-devoluciones-y-cierre.md#11-diferencias-con-la-documentación-técnica) |
| La devolución con destino «Cuarentena DIGEMID» solo anota el destino | No repone stock ni crea una cuarentena | [Capítulo 9, §7 Devolver productos](capitulo-09-ventas-devoluciones-y-cierre.md#7-devolver-productos); [Capítulo 9, §11 Diferencias con la documentación técnica](capitulo-09-ventas-devoluciones-y-cierre.md#11-diferencias-con-la-documentación-técnica) |
| Al cerrar la caja no se muestra el arqueo | Vuelve a la lista con «Sesión cerrada correctamente»; el resultado se ve abriendo el detalle de la sesión | [Capítulo 9, §8 Cerrar tu caja con arqueo](capitulo-09-ventas-devoluciones-y-cierre.md#8-cerrar-tu-caja-con-arqueo) |
| El carrito se guarda solo en tu navegador | Si recargas la página, el borrador se restaura; se borra al cobrar, al cerrar sesión o si otra persona inicia sesión en ese equipo | [Capítulo 8, §16 El borrador del carrito (si recargas la página)](capitulo-08-caja-y-venta.md#16-el-borrador-del-carrito-si-recargas-la-página) |
| Al cambiar el método de pago de «Efectivo» a «Tarjeta Débito/Crédito» el monto puede mostrar una cifra larga | Se vio `1.2000000000000002` en lugar de 1.20 | [Capítulo 10, §10 Pendientes y «No verificado»](capitulo-10-inicio-y-roles.md#10-pendientes-y-no-verificado) |

### 7.2 Recetas y pacientes

| Limitación | Qué ves | Dónde |
|---|---|---|
| La receta no tiene líneas de medicamentos | Solo tiene cabecera; el terminal no comprueba que la receta elegida corresponda al medicamento que se vende | [Capítulo 7, §14 Diferencias con la documentación técnica](capitulo-07-pacientes-y-recetas.md#14-diferencias-con-la-documentación-técnica); [Capítulo 8, §19 Diferencias con la documentación técnica](capitulo-08-caja-y-venta.md#19-diferencias-con-la-documentación-técnica) |
| El sistema no vence recetas | No existen recetas «vigentes» o «vencidas» | [Capítulo 7, §14 Diferencias con la documentación técnica](capitulo-07-pacientes-y-recetas.md#14-diferencias-con-la-documentación-técnica) |
| El lector de recetas con IA tiene límite y un solo mensaje de error | Hasta 10 análisis por minuto por empresa; cualquier fallo muestra «Error al interpretar la receta. Intenta nuevamente.» | [Capítulo 7, §12.4 Lo que no se pudo comprobar en la aplicación](capitulo-07-pacientes-y-recetas.md#124-lo-que-no-se-pudo-comprobar-en-la-aplicación); [Capítulo 8, §17 La pestaña «Receta» (lector de recetas con IA)](capitulo-08-caja-y-venta.md#17-la-pestaña-receta-lector-de-recetas-con-ia) |
| La carga masiva de pacientes muestra nombres técnicos | La vista previa muestra `fields.tipoDocumento` y nombres de campo como `docIdentidadGub` | [Capítulo 7, §7.2 Cargar muchos pacientes (Carga Masiva)](capitulo-07-pacientes-y-recetas.md#72-cargar-muchos-pacientes-carga-masiva); [Capítulo 7, §14 Diferencias con la documentación técnica](capitulo-07-pacientes-y-recetas.md#14-diferencias-con-la-documentación-técnica) |
| El aviso de carga masiva usa un plural fijo | «Se guardaron 1 registros correctamente» | [Capítulo 7, §14 Diferencias con la documentación técnica](capitulo-07-pacientes-y-recetas.md#14-diferencias-con-la-documentación-técnica) |

### 7.3 Catálogos, productos e inventario

| Limitación | Qué ves | Dónde |
|---|---|---|
| La carga masiva de medicamentos no trae el precio | Los medicamentos quedan con el precio vacío y al editarlos el campo exige un valor | [Capítulo 5, §10.3 Carga masiva de medicamentos](capitulo-05-productos-y-proveedores.md#103-carga-masiva-de-medicamentos); [Capítulo 5, §12 Diferencias con la documentación técnica](capitulo-05-productos-y-proveedores.md#12-diferencias-con-la-documentación-técnica) |
| El precio de un medicamento se guarda con dos decimales | Un precio con tres decimales se redondea (`0.505` pasa a `0.51`) | [Capítulo 5, §12 Diferencias con la documentación técnica](capitulo-05-productos-y-proveedores.md#12-diferencias-con-la-documentación-técnica) |
| El nombre de una presentación lo toma de la unidad elegida | Se llama «Caja» o «Blíster»; no se puede poner un nombre como «Caja x100» | [Capítulo 5, §6.3 Crear una presentación](capitulo-05-productos-y-proveedores.md#63-crear-una-presentación); [Capítulo 5, §12 Diferencias con la documentación técnica](capitulo-05-productos-y-proveedores.md#12-diferencias-con-la-documentación-técnica) |
| Guardar una presentación de un producto sin jerarquía no hace nada | No hay aviso ni mensaje | [Capítulo 5, §6.3 Crear una presentación](capitulo-05-productos-y-proveedores.md#63-crear-una-presentación); [Capítulo 5, §12 Diferencias con la documentación técnica](capitulo-05-productos-y-proveedores.md#12-diferencias-con-la-documentación-técnica) |
| La carga masiva marca con error todas las filas repetidas, también la primera | La primera aparición del duplicado sale con error | [Capítulo 4, §9.1 Antes de cargar](capitulo-04-catalogos-basicos.md#91-antes-de-cargar); [Capítulo 4, §11 Diferencias con la documentación técnica](capitulo-04-catalogos-basicos.md#11-diferencias-con-la-documentación-técnica) |
| La vista previa de la carga masiva no revisa algunos duplicados | El guardado puede informar «Se guardaron 0 registros correctamente» | [Capítulo 4, §9.1 Antes de cargar](capitulo-04-catalogos-basicos.md#91-antes-de-cargar); [Capítulo 4, §11 Diferencias con la documentación técnica](capitulo-04-catalogos-basicos.md#11-diferencias-con-la-documentación-técnica) |
| El catálogo DIGEMID de una empresa nueva se ve vacío | La búsqueda muestra «Resultados (0)»; los productos se cargan desde Excel | [Capítulo 4, §8.1 Qué es y para qué lo usa SOFIA](capitulo-04-catalogos-basicos.md#81-qué-es-y-para-qué-lo-usa-sofia); [Capítulo 4, §11 Diferencias con la documentación técnica](capitulo-04-catalogos-basicos.md#11-diferencias-con-la-documentación-técnica) |
| Los errores de guardar un lote no dicen la causa | «Error al guardar el lote» para caducidad vencida, fabricación futura, número repetido o número de solo espacios | [Capítulo 6, §5.3 Errores que puedes ver (todos comprobados en la aplicación)](capitulo-06-inventario.md#53-errores-que-puedes-ver-todos-comprobados-en-la-aplicación) |
| Ingresar stock sin elegir un lote no responde | **Guardar** no hace nada y no muestra ningún mensaje | [Capítulo 6, §14 Diferencias con la documentación técnica](capitulo-06-inventario.md#14-diferencias-con-la-documentación-técnica) |
| Las transferencias entre sucursales no tienen pantalla | No se puede mover stock entre sucursales; se ingresa y se ajusta en cada una | [Capítulo 6, §12 Transferencias entre sucursales](capitulo-06-inventario.md#12-transferencias-entre-sucursales) |

### 7.4 Empresa, personal y cuentas

| Limitación | Qué ves | Dónde |
|---|---|---|
| El RUC no se puede dejar vacío aunque el rótulo diga «opcional» | El servidor rechaza un RUC vacío o mal formado con «Error al actualizar la empresa» | [Capítulo 2, §3.3 Editar el nombre o el RUC](capitulo-02-empresa-sucursales-series.md#33-editar-el-nombre-o-el-ruc); [Capítulo 2, §13 Diferencias con la documentación técnica](capitulo-02-empresa-sucursales-series.md#13-diferencias-con-la-documentación-técnica) |
| La contraseña de una cuenta nueva con menos de 8 caracteres no da aviso | No aparece ningún mensaje y no se guarda | [Capítulo 3, §7.2 Crear la cuenta](capitulo-03-empleados-roles-cuentas.md#72-crear-la-cuenta); [Capítulo 3, §12 Diferencias con la documentación técnica](capitulo-03-empleados-roles-cuentas.md#12-diferencias-con-la-documentación-técnica) |
| Eliminar un empleado con cuenta deja la cuenta oculta | La cuenta desaparece de la lista y su usuario sigue ocupado | [Capítulo 3, §9.1 Editar y eliminar un empleado](capitulo-03-empleados-roles-cuentas.md#91-editar-y-eliminar-un-empleado); [Capítulo 3, §12 Diferencias con la documentación técnica](capitulo-03-empleados-roles-cuentas.md#12-diferencias-con-la-documentación-técnica) |
| Hay avisos con la clave sin traducir | Se vieron `cuentas.detail.load-error` y `digemidCatalogo.detail.load-error` en lugar del texto, al abrir una ficha por dirección directa | [Capítulo 3, §12 Diferencias con la documentación técnica](capitulo-03-empleados-roles-cuentas.md#12-diferencias-con-la-documentación-técnica); [Capítulo 4, §11 Diferencias con la documentación técnica](capitulo-04-catalogos-basicos.md#11-diferencias-con-la-documentación-técnica) |
| El idioma elegido no se conserva | Al recargar, cerrar sesión o volver a entrar, la interfaz vuelve a español | [Capítulo 1, §4.3 Cambiar el idioma](capitulo-01-primeros-pasos.md#43-cambiar-el-idioma) |
| La campana de la cabecera no genera notificaciones | Solo dice «Sin notificaciones» | [Capítulo 1, §4.2 Cabecera](capitulo-01-primeros-pasos.md#42-cabecera) |

### 7.5 Inicio y exportaciones

| Limitación | Qué ves | Dónde |
|---|---|---|
| Algunos archivos Excel escriben la fecha y la hora en UTC | Un movimiento hecho después de las 7 de la noche sale con la fecha del día siguiente. Afecta a la exportación de Ventas, de Sesiones de Caja y a la fecha de creación de Series Fiscales | [Capítulo 11, §2.8 La exportación sale con la fecha del día siguiente](capitulo-11-problemas-frecuentes.md#28-la-exportación-sale-con-la-fecha-del-día-siguiente); [Capítulo 9, §11 Diferencias con la documentación técnica](capitulo-09-ventas-devoluciones-y-cierre.md#11-diferencias-con-la-documentación-técnica); [Capítulo 2, §11.1 Buscar series y exportar](capitulo-02-empresa-sucursales-series.md#111-buscar-series-y-exportar) |
| Los botones «Ver ventas de hoy», «Ver mis ventas» y «Ver ventas en espera» se ocultan cuando ves más de una sucursal | Un administrador con dos o más sucursales no los ve | [Capítulo 10, §4 Los botones de acceso directo](capitulo-10-inicio-y-roles.md#4-los-botones-de-acceso-directo); [Capítulo 1, §4.5 El Inicio (dashboard)](capitulo-01-primeros-pasos.md#45-el-inicio-dashboard) |
| «Vendido hoy» y la lista de ventas de hoy pueden no coincidir | La tarjeta suma las ventas «Completada» y «Devuelta»; el botón filtra solo «Completada» | [Capítulo 10, §4 Los botones de acceso directo](capitulo-10-inicio-y-roles.md#4-los-botones-de-acceso-directo); [Capítulo 11, §2.9 Otros comportamientos que sorprenden](capitulo-11-problemas-frecuentes.md#29-otros-comportamientos-que-sorprenden) |
| El texto «1 unidades» usa un plural fijo | Se ve en «Productos más vendidos en 7 días» | [Capítulo 10, §10 Pendientes y «No verificado»](capitulo-10-inicio-y-roles.md#10-pendientes-y-no-verificado) |

## 8. Pendientes y «No verificado»

- **En curso (sin commitear):** ninguno que este documento afirme. Este documento no se apoya en archivos de los repositorios: integra capítulos ya publicados.
- **No verificado en la aplicación:** este documento no se recorrió en la aplicación; es una integración de los once capítulos. Todo lo que afirma sobre la aplicación sale de esos capítulos y conserva allí su marca de «Verificado» o «No verificado en la aplicación». En particular, cada fila de la tabla de limitaciones (sección 7) remite a la sección donde el capítulo explica cómo se comprobó.
- **Commits.** La cabecera indica los commits sobre los que se integra este manual (`sofia-apis` `9c16f35`, `sofia-web-spa` `7070e1a`). Los capítulos 1 a 11 se escribieron sobre commits anteriores, que cada uno declara en su propia cabecera; este documento no volvió a recorrer la aplicación con los commits nuevos.
- **Índice generado por script.** Los enlaces y las anclas del índice (sección 3) y de las tablas de las secciones 2, 4 y 7 se generaron leyendo los títulos de los capítulos. Si un capítulo cambia sus títulos, hay que volver a generar este documento.
- **Capturas de pantalla:** ninguna, por la razón de la sección 5.

## 9. Tabla de cobertura

| Elemento pedido | Dónde quedó |
|---|---|
| Cabecera de las reglas de documentación (commits `9c16f35` y `7070e1a`; método «Integración de los capítulos publicados») | Cabecera |
| Presentación: qué es SOFIA, para quién es, cómo está organizado, cómo leerlo | Sección 1 |
| Recorrido del administrador que empieza de cero (capítulos 1 a 7) con el orden de puesta en marcha | Sección 2.1 |
| Recorrido del cajero en su día a día (capítulos 1, 8, 9 y 10) | Sección 2.2 |
| Índice completo (títulos de nivel 2 y 3 de los once capítulos, con enlaces y anclas generados por script) | Sección 3 |
| Lista de tareas frecuentes con capítulo y sección | Sección 4 |
| Convenciones: negritas, «», «No verificado en la aplicación», capturas, datos de «Botica Manual Demo» | Sección 5 |
| Glosario breve con enlace al glosario de la documentación (sin copiarlo) | Sección 6 |
| Limitaciones conocidas que ve el usuario, con capítulo y sección de cada una | Sección 7 |
| Pendientes y «No verificado», tabla de cobertura y fuentes | Secciones 8, 9 y 10 |

## 10. Fuentes

Documentos consultados (todos en esta carpeta, salvo el glosario):

- `manual-de-usuario/capitulo-01-primeros-pasos.md`, `manual-de-usuario/capitulo-02-empresa-sucursales-series.md`, `manual-de-usuario/capitulo-03-empleados-roles-cuentas.md`, `manual-de-usuario/capitulo-04-catalogos-basicos.md`, `manual-de-usuario/capitulo-05-productos-y-proveedores.md`, `manual-de-usuario/capitulo-06-inventario.md`, `manual-de-usuario/capitulo-07-pacientes-y-recetas.md`, `manual-de-usuario/capitulo-08-caja-y-venta.md`, `manual-de-usuario/capitulo-09-ventas-devoluciones-y-cierre.md`, `manual-de-usuario/capitulo-10-inicio-y-roles.md` y `manual-de-usuario/capitulo-11-problemas-frecuentes.md`: títulos para el índice, secciones «Antes de empezar» para el orden de puesta en marcha, y secciones «Diferencias con la documentación técnica» y «Pendientes y «No verificado»» para las limitaciones.
- [Glosario de la documentación](../negocio/glosario.md): solo enlazado.
- [Reglas comunes para los agentes de documentación](../_inventario/REGLAS-AGENTES.md) y [Instrucciones comunes para los capítulos del manual](../_inventario/MANUAL-AGENTES.md): formato y convenciones.

Este documento no incluye recorridos en la aplicación.
