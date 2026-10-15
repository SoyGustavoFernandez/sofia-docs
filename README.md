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
| [Matriz de permisos](ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md) | Módulo × acción × endpoint × pantalla, permisos necesarios por pantalla y roles base |
| [Integraciones externas](ola-1-fundamentos/07-integraciones.md) | Gemini, Presidio, correo, y el estado real de SUNAT y DIGEMID |
| [Guía del desarrollador](ola-1-fundamentos/08-guia-del-desarrollador.md) | Requisitos, entorno local, configuración, comandos de calidad, hooks de commit y convenciones |
| [Carga masiva y exportación](ola-1-fundamentos/20-carga-masiva-y-exportacion.md) | Las 12 cargas masivas (plantilla, validaciones por fila y guardado) y las 22 exportaciones a Excel |
| [Glosario](ola-1-fundamentos/06-glosario.md) | 133 términos de SUNAT, DIGEMID, caja, inventario, organización e IA, con lo que significan en el sistema y dónde aparecen |

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

### Manual de usuario

Escrito recorriendo cada paso en la aplicación, con los textos exactos de la pantalla. Se publica capítulo por capítulo.

| Capítulo | Contenido |
|---|---|
| [1. Primeros pasos](ola-2-funcional/manual-usuario/capitulo-01-primeros-pasos.md) | Registrar la empresa, conocer la pantalla principal, iniciar y cerrar sesión, cambiar y recuperar la contraseña |

### Inventarios base

La carpeta [`_inventario`](_inventario/README.md) contiene las listas generadas por script desde el código (endpoints, permisos, tablas, códigos de error, rutas del SPA, pruebas). Son la lista de control contra la que se validan las tablas de cobertura.

## En preparación

- Modelo de datos transaccional e historial de scripts.
- Seguridad: autenticación y controles.
- Catálogo de códigos de error y referencia de la API.
- Pruebas automatizadas, despliegue y operación, estado del sistema y deuda técnica.
- Historias de usuario, casos de uso, procesos de negocio y manual de usuario.
- Casos de prueba funcionales y matriz de trazabilidad.

## Estructura

```
_inventario/           Listas base generadas desde el código
ola-1-fundamentos/     Reglas, arquitectura, datos, seguridad, integraciones, guía y pantallas
ola-2-funcional/       Historias, casos de uso, procesos y manual de usuario (en preparación)
ola-3-derivados/       Casos de prueba y trazabilidad (en preparación)
ola-4-cierre/          Revisión de consistencia (en preparación)
```
