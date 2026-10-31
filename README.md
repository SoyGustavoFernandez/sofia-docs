# SOFIA — Documentación

Documentación de **SOFIA (Sistema Optimizado Farmacéutico con Inteligencia Artificial)**, un SaaS de gestión para boticas y farmacias del Perú, multiempresa y multisucursal.

El código vive en dos repositorios:

- [sofia-apis](https://github.com/SoyGustavoFernandez/sofia-apis): API REST (.NET 10, Clean Architecture, CQRS, SQL Server).
- [sofia-web-spa](https://github.com/SoyGustavoFernandez/sofia-web-spa): aplicación web (Angular 20).

> **Documentación en construcción.** Cada documento indica en su cabecera el commit de cada repositorio sobre el que se escribió y su versión. Los documentos publicados están alineados con `sofia-apis` `43f5529` y `sofia-web-spa` `eeffbb6`; los que se corrigieron por cambios posteriores del código están en la versión 1.1 y llevan un historial de cambios.

## Cómo se escribió

Cada documento sale de leer el código, los scripts de base de datos y los textos de la interfaz; no de suposiciones. Las afirmaciones llevan su fuente (`repo/ruta/archivo:línea`), lo que no se pudo comprobar está marcado como «No verificado» y lo que no existe está marcado como «Pendiente» o «Solo API (sin pantalla)». Cada documento cierra con una tabla de cobertura y la lista de fuentes consultadas, y pasó una validación por muestreo contra el código antes de publicarse.

## Documentos disponibles

### Reglas de negocio

785 reglas con identificador `RN-<MÓDULO>-<nnn>`, dónde se aplican, el código de error que devuelven, el código HTTP y el mensaje que ve el usuario.

| Documento | Contenido |
|---|---|
| [Administración y seguridad](ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) | Reglas transversales, autenticación, roles, cuentas, empresas, sucursales, empleados, series fiscales, carga masiva y exportación |
| [Catálogos](ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) | Unidades, jerarquías, laboratorios, ingredientes activos, medicamentos, presentaciones, formulaciones, proveedores, aseguradoras, catálogo DIGEMID, pacientes y profesionales de salud |
| [Inventario y recetas](ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) | Lotes, stock por sucursal y recetas médicas (incluye el análisis con IA) |
| [Ventas y comprobantes](ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) | Punto de venta, recetas en la venta, pagos, venta en espera, anulación y boleta |
| [Caja, devoluciones y dashboard](ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) | Apertura, cierre y arqueo de caja, devoluciones con nota de crédito e indicadores del dashboard |
| [Módulos solo API](ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) | Transferencias, delivery, órdenes magistrales, servicios e inmunizaciones, cuarentena y actas DIGEMID, auditoría |

### Arquitectura, datos y seguridad

| Documento | Contenido |
|---|---|
| [Arquitectura de software](ola-1-fundamentos/02-arquitectura.md) | Modelo C4, capas, CQRS, pipeline de MediatR, multiempresa, manejo de errores, configuración y requisitos no funcionales implementados |
| [Modelo de datos: maestros y seguridad](ola-1-fundamentos/03-modelo-de-datos/03a-maestros-y-seguridad.md) | Diccionario de datos de 27 tablas, diagramas entidad-relación y discrepancias entre scripts y EF |
| [Modelo de datos: transaccionales e historial](ola-1-fundamentos/03-modelo-de-datos/03b-transaccionales-e-historial.md) | Diccionario de datos de las 23 tablas de inventario, recetas, caja, ventas, comprobantes, devoluciones y módulos solo API, diagramas entidad-relación e historial de los 28 scripts |
| [Autenticación y controles de seguridad](ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md) | Inicio de sesión, renovación y rotación de tokens, cierre de sesión, recuperación de contraseña, aislamiento por empresa y sucursal, auditoría, protección de la API, datos personales y OWASP Top 10 |
| [Matriz de permisos](ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md) | Módulo × acción × endpoint × pantalla, permisos necesarios por pantalla y roles base |
| [Integraciones externas](ola-1-fundamentos/07-integraciones.md) | Gemini, Presidio, correo, y el estado real de SUNAT y DIGEMID |
| [Guía del desarrollador](ola-1-fundamentos/08-guia-del-desarrollador.md) | Requisitos, entorno local, configuración, comandos de calidad, hooks de commit y convenciones |
| [Catálogo de códigos de error](ola-1-fundamentos/21-catalogo-codigos-error.md) | Los 347 códigos de error con su HTTP, cuándo salen, la regla que los define y el mensaje que ve el usuario; códigos de validación dinámicos y códigos sin traducción |
| [Carga masiva y exportación](ola-1-fundamentos/20-carga-masiva-y-exportacion.md) | Las 12 cargas masivas (plantilla, validaciones por fila y guardado) y las 22 exportaciones a Excel |
| [Glosario](ola-1-fundamentos/06-glosario.md) | 133 términos de SUNAT, DIGEMID, caja, inventario, organización e IA, con lo que significan en el sistema y dónde aparecen |
| [Pruebas automatizadas del backend](ola-1-fundamentos/09-pruebas/09a-backend.md) | Estrategia, catálogo de las 193 clases de prueba (1254 pruebas), cobertura por módulo y huecos |
| [Pruebas automatizadas del SPA](ola-1-fundamentos/09-pruebas/09b-spa.md) | Estrategia, catálogo de los 75 specs, cobertura por pantalla y huecos |
| [Despliegue y operación](ola-1-fundamentos/10-despliegue-y-operacion.md) | Componentes desplegables, docker-compose, configuración por entorno, base de datos, operación, CI y lo pendiente para Azure |
| [Estado del sistema y deuda técnica](ola-1-fundamentos/11-estado-y-deuda-tecnica.md) | Estado de los 29 módulos, 23 funcionalidades pendientes o parciales, 82 elementos de deuda técnica verificados, decisiones de negocio frente al código y hallazgos ya resueltos |

### Referencia de la API

Una ficha por endpoint con permiso, cuerpo, respuesta, errores posibles y un ejemplo real capturado de la API local. Las convenciones comunes (autenticación, errores, paginación, límites) van en la parte de administración.

| Documento | Contenido |
|---|---|
| [Catálogos básicos](ola-1-fundamentos/04-referencia-api/04b-catalogos-basicos.md) | 50 endpoints de unidades de medida, jerarquías de unidades, laboratorios, ingredientes activos y DIGEMID (catálogo, cuarentena y actas), todos con ejemplo real |
| [Ventas, caja y devoluciones](ola-1-fundamentos/04-referencia-api/04e-ventas-caja-devoluciones.md) | 18 endpoints del punto de venta, ventas y devoluciones con ejemplo real, y un recorrido completo de apertura, ventas, anulación, devolución y cierre con el arqueo calculado a mano y comprobado contra la API |

### Inventario de pantallas

Cada pantalla enrutada con sus campos, validaciones, botones, permisos, mensajes y llamadas a la API.

| Documento | Contenido |
|---|---|
| [Acceso, layout y componentes compartidos](ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) | Inicio de sesión, registro de empresa, recuperación de contraseña, dashboard, menú, cabecera, carga masiva y textos comunes |
| [Administración](ola-1-fundamentos/19-inventario-de-pantallas/19b-administracion.md) | Empresa, sucursales, empleados, roles y permisos, cuentas de usuario y series fiscales |
| [Catálogos I](ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md) | Unidades de medida, jerarquías de unidades, laboratorios, ingredientes activos y catálogo DIGEMID |
| [Catálogos II](ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md) | Medicamentos, presentaciones de venta, formulaciones clínicas, proveedores (con historial de precios) y aseguradoras |
| [Inventario y atención](ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md) | Lotes, stock por sucursal, pacientes, profesionales de salud y recetas médicas (incluye el escáner con IA) |
| [Punto de venta](ola-1-fundamentos/19-inventario-de-pantallas/19f-punto-de-venta.md) | Sesiones de caja (apertura, cierre y arqueo), nueva venta, cobro, comprobante, búsqueda y detalle de ventas, anulación y devolución |

### Procesos de negocio

| Documento | Contenido |
|---|---|
| [Procesos de punta a punta](ola-2-funcional/14-procesos-de-negocio.md) | Alta de empresa, configuración inicial, catálogo e inventario, venta (recetas, pagos, venta en espera), devolución y anulación, caja y arqueo, recetas con IA, dashboard y procesos que solo existen en la API, con diagramas |

### Historias de usuario

| Documento | Contenido |
|---|---|
| [Acceso y administración](ola-2-funcional/12-historias-de-usuario/12a-acceso-y-administracion.md) | 50 historias con criterios de aceptación en Gherkin: registro, sesión, contraseñas, empresa, sucursales, series, empleados, roles, cuentas y reglas transversales |
| [Catálogos, inventario y atención](ola-2-funcional/12-historias-de-usuario/12b-catalogos-inventario-atencion.md) | 66 historias: catálogos, proveedores y precios, aseguradoras, DIGEMID, lotes, stock, pacientes, profesionales y recetas con escáner |
| [Punto de venta y caja](ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md) | 76 historias: ventas y comprobantes, caja y arqueo, devoluciones, dashboard y módulos que solo existen en la API |

### Casos de uso

| Documento | Contenido |
|---|---|
| [Acceso y administración](ola-2-funcional/13-casos-de-uso/13a-acceso-y-administracion.md) | 37 casos de uso con actores, precondiciones, flujo principal, alternativos, excepciones con sus códigos de error y postcondiciones |
| [Catálogos, inventario y atención](ola-2-funcional/13-casos-de-uso/13b-catalogos-inventario-atencion.md) | 66 casos de uso de catálogos, proveedores y precios, aseguradoras, DIGEMID, lotes, stock, pacientes, profesionales y recetas, con su carga masiva |
| [Punto de venta y caja](ola-2-funcional/13-casos-de-uso/13c-punto-de-venta-y-caja.md) | 76 casos de uso de ventas, cobro, venta en espera, anulación, caja y arqueo, devoluciones, dashboard y módulos que solo existen en la API |

### Manual de usuario

Escrito recorriendo cada paso en la aplicación, con los textos exactos de la pantalla. Se publica capítulo por capítulo.

| Capítulo | Contenido |
|---|---|
| [1. Primeros pasos](ola-2-funcional/manual-usuario/capitulo-01-primeros-pasos.md) | Registrar la empresa, conocer la pantalla principal, iniciar y cerrar sesión, cambiar y recuperar la contraseña |
| [2. Empresa, sucursales y series fiscales](ola-2-funcional/manual-usuario/capitulo-02-empresa-sucursales-series.md) | Datos de la empresa, sede principal y nuevas sucursales, búsqueda, exportación, eliminación y carga masiva, y series de boleta y nota de crédito |
| [3. Tu equipo y sus accesos](ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md) | Empleados, roles y permisos (con los permisos recomendados para un cajero), cuentas de usuario, sucursales y el primer ingreso del cajero |
| [4. Catálogos básicos](ola-2-funcional/manual-usuario/capitulo-04-catalogos-basicos.md) | Unidades de medida, laboratorios, ingredientes activos y catálogo DIGEMID, con su carga masiva |
| [5. Productos y proveedores](ola-2-funcional/manual-usuario/capitulo-05-productos-y-proveedores.md) | Medicamentos y su condición de venta, jerarquías de unidades, presentaciones de venta, formulaciones, proveedores con sus precios de compra y aseguradoras, con su carga masiva y qué ve el cajero |
| [6. Inventario](ola-2-funcional/manual-usuario/capitulo-06-inventario.md) | Lotes con sus validaciones, búsqueda, edición y eliminación, ingreso y ajuste de stock por sucursal, exportación y qué ve el cajero |
| [11. Problemas frecuentes](ola-2-funcional/manual-usuario/capitulo-11-problemas-frecuentes.md) | Situaciones sin mensaje de error, mensajes de error por tarea con su causa y qué hacer, cuándo pedir ayuda e índice alfabético de mensajes |

### Casos de prueba

Casos funcionales para ejecutar a mano en el SPA (o con un cliente HTTP en los módulos que solo existen en la API), con datos, pasos y resultado esperado exacto, y su trazabilidad a historias, casos de uso y reglas. Todavía no ejecutados.

| Documento | Contenido |
|---|---|
| [Acceso y administración](ola-3-calidad/16-casos-de-prueba/16a-acceso-y-administracion.md) | 413 casos (166 de prioridad alta) que cubren los 424 escenarios de las historias de acceso y administración, con suite de humo de 26 casos |
| [Catálogos, inventario y atención](ola-3-calidad/16-casos-de-prueba/16b-catalogos-inventario-atencion.md) | 321 casos (98 de prioridad alta) que cubren los 416 escenarios de catálogos, proveedores, aseguradoras, DIGEMID, lotes, stock, pacientes, profesionales y recetas (incluido el escáner con IA), con suite de humo de 30 casos |
| [Punto de venta y caja](ola-3-calidad/16-casos-de-prueba/16c-punto-de-venta-y-caja.md) | 510 casos (180 de prioridad alta) que cubren los 566 escenarios de ventas, cobro, anulación, caja y arqueo, devoluciones, dashboard y módulos que solo existen en la API, con suite de humo de 28 casos |

### Inventarios base

La carpeta [`_inventario`](_inventario/README.md) contiene las listas generadas por script desde el código (endpoints, permisos, tablas, códigos de error, rutas del SPA, pruebas). Son la lista de control contra la que se validan las tablas de cobertura.

## En preparación

- Referencia de la API.
- Manual de usuario (capítulos 7 a 10).
- Matriz de trazabilidad.

## Estructura

```
_inventario/           Listas base generadas desde el código
ola-1-fundamentos/     Reglas, arquitectura, datos, seguridad, integraciones, guía y pantallas
ola-2-funcional/       Historias, casos de uso, procesos y manual de usuario (en preparación)
ola-3-calidad/         Casos de prueba y trazabilidad (en preparación)
ola-4-cierre/          Revisión de consistencia (en preparación)
```
