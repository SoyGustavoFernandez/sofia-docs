# SOFIA — Documentación

Documentación de **SOFIA (Sistema Optimizado Farmacéutico con Inteligencia Artificial)**, un SaaS de gestión para boticas y farmacias del Perú, multiempresa y multisucursal.

El código vive en dos repositorios:

- [sofia-apis](https://github.com/SoyGustavoFernandez/sofia-apis): API REST (.NET 10, Clean Architecture, CQRS, SQL Server).
- [sofia-web-spa](https://github.com/SoyGustavoFernandez/sofia-web-spa): aplicación web (Angular 20).

> **Documentación en construcción.** Cada documento indica en su cabecera el commit de cada repositorio sobre el que se escribió y su versión. Los documentos publicados están alineados con `sofia-apis` `0678e40` y `sofia-web-spa` `37048ea` (revisión contra el código del 2026-11-09 al 2026-11-20, ver [Revisión de consistencia](calidad/revision-de-consistencia.md)); los que se corrigieron por cambios del código llevan un historial de cambios.

## Cómo se escribió

Cada documento sale de leer el código, los scripts de base de datos y los textos de la interfaz; no de suposiciones. Las afirmaciones llevan su fuente (`repo/ruta/archivo:línea`), lo que no se pudo comprobar está marcado como «No verificado» y lo que no existe está marcado como «Pendiente» o «Solo API (sin pantalla)». Cada documento cierra con una tabla de cobertura y la lista de fuentes consultadas, y pasó una validación por muestreo contra el código antes de publicarse.

## Orden de lectura

Las carpetas están ordenadas por tema, y este es el orden de lectura recomendado:

1. [Manual de usuario](#manual-de-usuario): cómo se usa la aplicación, paso a paso.
2. [Negocio](#negocio): reglas, procesos y vocabulario del dominio.
3. [Producto](#producto): historias de usuario, casos de uso e inventario de pantallas.
4. [Técnico](#técnico): arquitectura, datos, API, seguridad, integraciones, carga masiva y códigos de error.
5. [Desarrollo](#desarrollo): entorno local, despliegue y estado del sistema.
6. [Calidad](#calidad): pruebas automatizadas, casos de prueba, trazabilidad y revisión de consistencia.
7. [Inventarios base](#inventarios-base): listas generadas desde el código.

## Manual de usuario

Carpeta `manual-de-usuario/`. Escrito recorriendo cada paso en la aplicación, con los textos exactos de la pantalla. Empieza por el [documento integrador](manual-de-usuario/README.md): presentación, recorridos por rol, índice completo, tareas frecuentes y limitaciones conocidas.

| Capítulo | Contenido |
|---|---|
| [1. Primeros pasos](manual-de-usuario/capitulo-01-primeros-pasos.md) | Registrar la empresa, conocer la pantalla principal, iniciar y cerrar sesión (y qué ves si la suscripción de tu empresa venció, con los datos para pagar), cambiar y recuperar la contraseña |
| [2. Empresa, sucursales y series fiscales](manual-de-usuario/capitulo-02-empresa-sucursales-series.md) | Datos de la empresa y su fecha de vencimiento, sede principal y nuevas sucursales, búsqueda, exportación, eliminación y carga masiva, y series de boleta y nota de crédito |
| [3. Tu equipo y sus accesos](manual-de-usuario/capitulo-03-empleados-roles-cuentas.md) | Empleados, roles y permisos (con los permisos recomendados para un cajero), cuentas de usuario, sucursales y el primer ingreso del cajero |
| [4. Catálogos básicos](manual-de-usuario/capitulo-04-catalogos-basicos.md) | Unidades de medida, laboratorios, ingredientes activos y catálogo DIGEMID, con su carga masiva |
| [5. Productos y proveedores](manual-de-usuario/capitulo-05-productos-y-proveedores.md) | Medicamentos y su condición de venta, jerarquías de unidades, presentaciones de venta, formulaciones, proveedores con sus precios de compra y aseguradoras, con su carga masiva y qué ve el cajero |
| [6. Inventario](manual-de-usuario/capitulo-06-inventario.md) | Lotes con sus validaciones, búsqueda, edición y eliminación, ingreso y ajuste de stock por sucursal, exportación y qué ve el cajero |
| [7. Pacientes, profesionales y recetas](manual-de-usuario/capitulo-07-pacientes-y-recetas.md) | Pacientes y profesionales de salud con su carga masiva, registro y búsqueda de recetas, el lector de recetas con IA y qué ve el cajero |
| [8. Caja y nueva venta](manual-de-usuario/capitulo-08-caja-y-venta.md) | Abrir la caja, vender con efectivo, tarjeta, Yape / Plin, transferencia o pago dividido, vender medicamentos con receta y controlados, ventas en espera, el comprobante y qué ve el cajero |
| [9. Ventas, anulación, devolución y cierre](manual-de-usuario/capitulo-09-ventas-devoluciones-y-cierre.md) | Buscar ventas y ver su detalle, anular una venta, devolver con nota de crédito, buscar devoluciones, ver su detalle y exportarlas, cerrar la caja con arqueo y consultar las sesiones de caja |
| [10. Inicio y roles](manual-de-usuario/capitulo-10-inicio-y-roles.md) | Cada indicador del Inicio explicado con datos reales, el aviso de vencimiento de la suscripción, qué cambia para el cajero y una tabla de qué ve y puede hacer el administrador frente al cajero en toda la aplicación |
| [11. Problemas frecuentes](manual-de-usuario/capitulo-11-problemas-frecuentes.md) | Situaciones sin mensaje de error, mensajes de error por tarea con su causa y qué hacer (incluidos la suscripción vencida y la pantalla de devoluciones), cuándo pedir ayuda e índice alfabético de mensajes |
| [12. Operador de plataforma](manual-de-usuario/capitulo-12-operador-de-plataforma.md) | Para quien administra SOFIA y no una botica: la marca de operador, la configuración de pago (Yape, WhatsApp, QR), activar, extender, suspender y cancelar la suscripción de una empresa, y la sección «Suscripciones» del Inicio. Escrito desde el código, no verificado en la aplicación |

## Negocio

Carpeta `negocio/`.

### Reglas de negocio

Carpeta `negocio/reglas/`. 812 reglas con identificador `RN-<MÓDULO>-<nnn>`, dónde se aplican, el código de error que devuelven, el código HTTP y el mensaje que ve el usuario.

| Documento | Contenido |
|---|---|
| [Administración y seguridad](negocio/reglas/administracion-y-seguridad.md) | Reglas transversales, autenticación (incluido cómo pagar desde el inicio de sesión), roles, cuentas, empresas y suscripción (vigencia, operador de plataforma, activar, extender, suspender y cancelar, configuración de pago), sucursales, empleados, series fiscales, carga masiva y exportación |
| [Catálogos](negocio/reglas/catalogos.md) | Unidades, jerarquías, laboratorios, ingredientes activos, medicamentos, presentaciones, formulaciones, proveedores, aseguradoras, catálogo DIGEMID, pacientes y profesionales de salud |
| [Inventario y recetas](negocio/reglas/inventario-y-recetas.md) | Lotes, stock por sucursal y recetas médicas (incluye el análisis con IA) |
| [Ventas y comprobantes](negocio/reglas/ventas-y-comprobantes.md) | Punto de venta, recetas en la venta, pagos, venta en espera, anulación y boleta |
| [Caja, devoluciones y dashboard](negocio/reglas/caja-devoluciones-y-dashboard.md) | Apertura, cierre y arqueo de caja, devoluciones con nota de crédito (incluidos su búsqueda, detalle y exportación a Excel) e indicadores del dashboard (con las cifras de suscripciones del operador) |
| [Módulos solo API](negocio/reglas/modulos-solo-api.md) | Transferencias, delivery, órdenes magistrales, servicios e inmunizaciones, cuarentena y actas DIGEMID, auditoría |

### Procesos y vocabulario

| Documento | Contenido |
|---|---|
| [Procesos de punta a punta](negocio/procesos-de-negocio.md) | Alta de empresa, configuración inicial, catálogo e inventario, venta (recetas, pagos, venta en espera), devolución y anulación, caja y arqueo, recetas con IA, dashboard, suscripción de la empresa (alta, cobro y renovación) y procesos que solo existen en la API, con diagramas |
| [Glosario](negocio/glosario.md) | 136 términos de SUNAT, DIGEMID, caja, inventario, organización e IA, con lo que significan en el sistema y dónde aparecen |

## Producto

Carpeta `producto/`.

### Historias de usuario

Carpeta `producto/historias-de-usuario/`.

| Documento | Contenido |
|---|---|
| [Acceso y administración](producto/historias-de-usuario/acceso-y-administracion.md) | 57 historias con criterios de aceptación en Gherkin: registro, sesión, contraseñas, empresa, sucursales, series, empleados, roles, cuentas, reglas transversales y, desde la gestión de suscripciones, el operador de plataforma, la configuración de pago, el aviso de vencimiento y las cifras del panel |
| [Catálogos, inventario y atención](producto/historias-de-usuario/catalogos-inventario-y-atencion.md) | 66 historias: catálogos, proveedores y precios, aseguradoras, DIGEMID, lotes, stock, pacientes, profesionales y recetas con escáner |
| [Punto de venta y caja](producto/historias-de-usuario/punto-de-venta-y-caja.md) | 78 historias: ventas y comprobantes, caja y arqueo, devoluciones (con su búsqueda y su detalle), dashboard y módulos que solo existen en la API |

### Casos de uso

Carpeta `producto/casos-de-uso/`.

| Documento | Contenido |
|---|---|
| [Acceso y administración](producto/casos-de-uso/acceso-y-administracion.md) | 43 casos de uso con actores, precondiciones, flujo principal, alternativos, excepciones con sus códigos de error y postcondiciones, incluidos los del operador de plataforma y la suscripción |
| [Catálogos, inventario y atención](producto/casos-de-uso/catalogos-inventario-y-atencion.md) | 66 casos de uso de catálogos, proveedores y precios, aseguradoras, DIGEMID, lotes, stock, pacientes, profesionales y recetas, con su carga masiva |
| [Punto de venta y caja](producto/casos-de-uso/punto-de-venta-y-caja.md) | 78 casos de uso de ventas, cobro, venta en espera, anulación, caja y arqueo, devoluciones (con su búsqueda y su detalle), dashboard y módulos que solo existen en la API |

### Inventario de pantallas

Carpeta `producto/pantallas/`. Cada pantalla enrutada con sus campos, validaciones, botones, permisos, mensajes y llamadas a la API.

| Documento | Contenido |
|---|---|
| [Acceso, layout y componentes compartidos](producto/pantallas/acceso-layout-y-compartidos.md) | Inicio de sesión (con el recuadro «Cómo pagar»), registro de empresa, recuperación de contraseña, dashboard (con el aviso de vencimiento y la sección «Suscripciones» del operador), menú, cabecera, carga masiva y textos comunes |
| [Administración](producto/pantallas/administracion.md) | Empresa (con su vencimiento y las acciones de suscripción del operador), configuración de pago, sucursales, empleados, roles y permisos, cuentas de usuario y series fiscales |
| [Catálogos básicos](producto/pantallas/catalogos-basicos.md) | Unidades de medida, jerarquías de unidades, laboratorios, ingredientes activos y catálogo DIGEMID |
| [Productos, proveedores y aseguradoras](producto/pantallas/productos-proveedores-y-aseguradoras.md) | Medicamentos, presentaciones de venta, formulaciones clínicas, proveedores (con historial de precios) y aseguradoras |
| [Inventario y atención](producto/pantallas/inventario-y-atencion.md) | Lotes, stock por sucursal, pacientes, profesionales de salud y recetas médicas (incluye el escáner con IA) |
| [Punto de venta](producto/pantallas/punto-de-venta.md) | Sesiones de caja (apertura, cierre y arqueo), nueva venta, cobro, comprobante, búsqueda y detalle de ventas, anulación, devolución y la búsqueda y el detalle de devoluciones con su exportación |

## Técnico

Carpeta `tecnico/`.

### Arquitectura y datos

| Documento | Contenido |
|---|---|
| [Arquitectura de software](tecnico/arquitectura.md) | Modelo C4, capas, CQRS, pipeline de MediatR, multiempresa, manejo de errores, configuración y requisitos no funcionales implementados |
| [Modelo de datos: maestros y seguridad](tecnico/modelo-de-datos/maestros-y-seguridad.md) | Diccionario de datos de 28 tablas (incluida `Plataforma_Configuracion_Pago`), diagramas entidad-relación y discrepancias entre scripts y EF |
| [Modelo de datos: transaccionales e historial](tecnico/modelo-de-datos/transaccionales-e-historial.md) | Diccionario de datos de las 23 tablas de inventario, recetas, caja, ventas, comprobantes, devoluciones y módulos solo API, diagramas entidad-relación e historial de los 29 scripts |

### Referencia de la API

Carpeta `tecnico/api/`. Una ficha por endpoint con permiso, cuerpo, respuesta, errores posibles y un ejemplo real capturado de la API local. Las convenciones comunes (autenticación, errores, paginación, límites) van en la parte de administración.

| Documento | Contenido |
|---|---|
| [Convenciones y administración](tecnico/api/convenciones-y-administracion.md) | Convenciones comunes (autenticación, CSRF, errores, paginación, enumerados, límites) y 73 endpoints de autenticación, empresas (con la suscripción), plataforma, sucursales, empleados, roles, cuentas, series fiscales, dashboard y auditoría |
| [Catálogos básicos](tecnico/api/catalogos-basicos.md) | 50 endpoints de unidades de medida, jerarquías de unidades, laboratorios, ingredientes activos y DIGEMID (catálogo, cuarentena y actas), todos con ejemplo real |
| [Productos, proveedores y personas](tecnico/api/productos-proveedores-y-personas.md) | 61 endpoints de medicamentos, presentaciones de venta, formulaciones, proveedores y sus precios, aseguradoras, pacientes y profesionales de salud, todos con ejemplo real, incluidos los bloqueos al eliminar y la carga masiva |
| [Inventario, recetas y operaciones](tecnico/api/inventario-recetas-y-operaciones.md) | 46 endpoints de lotes, stock por sucursal, recetas (incluido el escáner con IA), transferencias, delivery, preparados magistrales y servicios, todos con ejemplo real |
| [Ventas, caja y devoluciones](tecnico/api/ventas-caja-y-devoluciones.md) | 19 endpoints del punto de venta, ventas y devoluciones (listado con filtros, detalle con líneas y exportación) con ejemplo real, y un recorrido completo de apertura, ventas, anulación, devolución y cierre con el arqueo calculado a mano y comprobado contra la API |

### Seguridad

Carpeta `tecnico/seguridad/`.

| Documento | Contenido |
|---|---|
| [Autenticación y controles de seguridad](tecnico/seguridad/autenticacion-y-controles.md) | Inicio de sesión, renovación y rotación de tokens, cierre de sesión, recuperación de contraseña, aislamiento por empresa y sucursal, auditoría, protección de la API, datos personales y OWASP Top 10 |
| [Matriz de permisos](tecnico/seguridad/matriz-de-permisos.md) | Módulo × acción × endpoint × pantalla de los 249 endpoints, permisos necesarios por pantalla, la política del operador de plataforma y roles base |

### Integraciones, carga masiva y códigos de error

| Documento | Contenido |
|---|---|
| [Integraciones externas](tecnico/integraciones.md) | Gemini, Presidio, correo, y el estado real de SUNAT y DIGEMID |
| [Carga masiva y exportación](tecnico/carga-masiva-y-exportacion.md) | Las 12 cargas masivas (plantilla, validaciones por fila y guardado) y las 23 exportaciones a Excel (incluida la de devoluciones) |
| [Catálogo de códigos de error](tecnico/catalogo-de-codigos-de-error.md) | Los 357 códigos de error con su HTTP, cuándo salen, la regla que los define y el mensaje que ve el usuario; códigos de validación dinámicos y códigos sin traducción |

## Desarrollo

Carpeta `desarrollo/`.

| Documento | Contenido |
|---|---|
| [Guía del desarrollador](desarrollo/guia-del-desarrollador.md) | Requisitos, entorno local, configuración, comandos de calidad, hooks de commit, convenciones, imagen del SPA y despliegue a UAT |
| [Despliegue y operación](desarrollo/despliegue-y-operacion.md) | Componentes desplegables, docker-compose, configuración por entorno, base de datos, publicación de imágenes y despliegue a UAT, alta del operador de plataforma, operación, CI y lo pendiente para Azure |
| [Estado del sistema y deuda técnica](desarrollo/estado-y-deuda-tecnica.md) | Estado de los 30 módulos, 23 funcionalidades pendientes o parciales (2 ya resueltas: listado de devoluciones y suscripción), 84 elementos de deuda técnica verificados (82 abiertos), decisiones de negocio frente al código y hallazgos ya resueltos |

## Calidad

Carpeta `calidad/`.

### Pruebas automatizadas

Carpeta `calidad/pruebas-automatizadas/`.

| Documento | Contenido |
|---|---|
| [Pruebas automatizadas del backend](calidad/pruebas-automatizadas/backend.md) | Estrategia, catálogo de las 204 clases de prueba (1344 pruebas), cobertura por módulo y huecos |
| [Pruebas automatizadas del SPA](calidad/pruebas-automatizadas/spa.md) | Estrategia, catálogo de los 83 specs (584 casos), cobertura por pantalla y huecos |

### Casos de prueba

Carpeta `calidad/casos-de-prueba/`. Casos funcionales para ejecutar a mano en el SPA (o con un cliente HTTP en los módulos que solo existen en la API), con datos, pasos y resultado esperado exacto, y su trazabilidad a historias, casos de uso y reglas. Todavía no ejecutados.

| Documento | Contenido |
|---|---|
| [Acceso y administración](calidad/casos-de-prueba/acceso-y-administracion.md) | 455 casos (187 de prioridad alta) que cubren 488 de los 489 escenarios de las historias de acceso y administración (incluidos el operador de plataforma y la suscripción), con suite de humo de 28 casos |
| [Catálogos, inventario y atención](calidad/casos-de-prueba/catalogos-inventario-y-atencion.md) | 321 casos (98 de prioridad alta) que cubren los 416 escenarios de catálogos, proveedores, aseguradoras, DIGEMID, lotes, stock, pacientes, profesionales y recetas (incluido el escáner con IA), con suite de humo de 30 casos |
| [Punto de venta y caja](calidad/casos-de-prueba/punto-de-venta-y-caja.md) | 530 casos (184 de prioridad alta) que cubren los 591 escenarios de ventas, cobro, anulación, caja y arqueo, devoluciones (con su búsqueda, detalle y exportación), dashboard y módulos que solo existen en la API, con suite de humo de 30 casos |

### Trazabilidad y revisión de consistencia

| Documento | Contenido |
|---|---|
| [Matriz de trazabilidad](calidad/matriz-de-trazabilidad.md) | Cruce por script de las 812 reglas, 201 historias, 187 casos de uso, 1306 casos de prueba y 249 endpoints con pantallas, pruebas automatizadas, procesos y los doce capítulos del manual; resumen de cobertura por módulo y huecos explicados |
| [Revisión de consistencia](calidad/revision-de-consistencia.md) | Primera ronda (2026-11-04): cada inconsistencia corregida entre documentos con su evidencia en el código y la reconciliación con `sofia-apis` `9c16f35` y `sofia-web-spa` `7070e1a`. Segunda ronda (2026-11-09 al 2026-11-20): la reorganización de carpetas, la reconciliación con `6f6b837`, `558ad35`, `0678e40` y `37048ea`, qué documentos se revisaron y con qué profundidad, los hallazgos de código, lo que quedó «No verificado» y las comprobaciones globales por script |

## Inventarios base

La carpeta [`_inventario`](_inventario/README.md) contiene las listas generadas por script desde el código (endpoints, permisos, tablas, códigos de error, rutas del SPA, pruebas). Son la lista de control contra la que se validan las tablas de cobertura.

## Estructura

```
manual-de-usuario/     Manual de usuario: documento integrador y doce capítulos
negocio/               Reglas de negocio, procesos y glosario
  reglas/
producto/              Historias de usuario, casos de uso e inventario de pantallas
  historias-de-usuario/
  casos-de-uso/
  pantallas/
tecnico/               Arquitectura, datos, API, seguridad, integraciones, carga masiva y códigos de error
  modelo-de-datos/
  api/
  seguridad/
desarrollo/            Guía del desarrollador, despliegue y operación, estado y deuda técnica
calidad/               Trazabilidad y revisión de consistencia
  pruebas-automatizadas/
  casos-de-prueba/
_inventario/           Listas base generadas desde el código
```
