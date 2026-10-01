# Plan para el Desarrollo de una Aplicación Generadora de QR Dinámicos para Transfermóvil (Cuba)

## 1. 🎯 Resumen Ejecutivo

**Objetivo:** Desarrollar una aplicación que permita a comercios y emprendedores en Cuba generar códigos QR dinámicos para cobros a través de Transfermóvil, agilizando las ventas presenciales y en línea.

**Alcance:** La aplicación se conectará al backend de **"Bulevar Mi Transfer"** de ETECSA, utilizando las credenciales de vendedor contratadas. No se accederá directamente a la app Transfermóvil del cliente, sino que se generarán los datos que el cliente escaneará con su propia app.

**Modelo de Negocio:** Suscripción o licencia para comercios que ya tengan contratado el servicio de ETECSA, ofreciendo una interfaz más ágil, gestión de múltiples usuarios (agentes) y funcionalidades adicionales de conciliación.

---

## 2. 🔍 Fase 1: Análisis del Ecosistema y Requisitos Previos

### 2.1. Entendimiento del Servicio "Bulevar Mi Transfer"

El servicio de ETECSA se divide en dos modalidades:
- **CON BULEVAR:** Para cobros presenciales mediante QR en comercios físicos.
- **SIN BULEVAR:** Para cobros online mediante tiendas virtuales e integración directa a la API de Transfermóvil.

Para generar **QR dinámicos** (que ya incluyen el monto exacto a pagar), es imprescindible contar con el **módulo de vendedor** habilitado, el cual solo se otorga a quienes contratan el servicio.

### 2.2. Requisitos Legales y de Contratación

Antes de desarrollar cualquier línea de código, el negocio para el que se desarrolla la app debe cumplir con:

- **Contratación con ETECSA:** Solicitar el servicio en [www.transfermovil.etecsa.cu/contratacion](http://www.transfermovil.etecsa.cu/contratacion).
- **Documentación:** Contar con línea móvil activa, correo electrónico nacional (dominio .cu), carné de identidad y contrato de cuenta fiscal asociada.
- **Credenciales de Vendedor:** Una vez aprobada la solicitud, ETECSA envía por correo el anexo del contrato y las credenciales (usuario y contraseña) para el portal Bulevar.
- **Configuración de APN:** Para acceso libre de costo, se debe configurar el APN privado `mitransfer` en los dispositivos.

**Nota regulatoria:** El Decreto-Ley No. 370 y el Decreto 359 regulan la industria cubana de programas y aplicaciones informáticas, promoviendo el desarrollo nacional. Es recomendable revisar estas normativas para asegurar el cumplimiento.

---

## 3. 🏗️ Fase 2: Arquitectura Técnica Propuesta

Dado que no hay documentación pública de la API de Bulevar Mi Transfer, la arquitectura debe ser **flexible y basada en la ingeniería inversa del flujo oficial** o en la documentación que ETECSA proporcione a los contratantes.

### 3.1. Componentes de la Aplicación

| Componente | Descripción | Tecnologías Sugeridas |
|------------|-------------|------------------------|
| **Frontend (App Móvil)** | Interfaz para el vendedor: ingreso de monto, generación y visualización del QR. Debe funcionar offline para mostrar el QR. | Flutter (multiplataforma) o Android nativo (Kotlin). |
| **Backend (Servidor de Integración)** | Servicio que gestiona la comunicación con la API de Bulevar Mi Transfer, autenticación, registro de transacciones y conciliación. | Python (FastAPI) o Node.js (Express). |
| **Módulo de Generación QR** | Genera la imagen del código QR a partir de los datos de pago (ID de transacción, monto, etc.). | Librería `qrcode` (Python) o `qr_flutter` (Flutter). |
| **Base de Datos** | Almacena el historial de ventas, estados de transacciones y usuarios/agentes. | PostgreSQL o SQLite (para despliegue ligero). |

### 3.2. Flujo de Generación de QR Dinámico

1. **Autenticación:** La app se autentica con las credenciales de vendedor obtenidas de ETECSA.
2. **Creación de Orden:** El vendedor ingresa el monto y conceptos. La app envía una solicitud al backend.
3. **Llamada a API de Bulevar:** El backend, usando las credenciales, solicita a la API de ETECSA la creación de una orden de pago. (Esta API no está documentada públicamente; se debe obtener acceso a través de ETECSA).
4. **Recepción de Datos QR:** La API devuelve una cadena de datos (posiblemente un JSON o una URL) que contiene el ID de transacción y el monto.
5. **Generación de Imagen QR:** El módulo de la app convierte esa cadena en una imagen QR.
6. **Visualización:** El vendedor muestra el QR en pantalla. El cliente lo escanea con su app Transfermóvil y confirma el pago.

---

## 4. 💻 Fase 3: Desarrollo de la Aplicación

### 4.1. Backend (Servidor de Integración)

- **Tecnología:** Python con FastAPI por su rapidez y facilidad para crear APIs REST.
- **Funcionalidades:**
    - Endpoint `/auth/login` para autenticación con credenciales de ETECSA.
    - Endpoint `/orders/create` que recibe `{monto, concepto}` y devuelve `{qr_data, order_id}`.
    - Webhook o polling para verificar el estado de la transacción (si la API de ETECSA lo permite).
- **Manejo de Errores:** Implementar reintentos y logs detallados, ya que la conectividad en Cuba puede ser intermitente.

### 4.2. Frontend (App Móvil)

- **Tecnología:** Flutter (para Android e iOS) o Kotlin (solo Android, dado el predominio de Android en Cuba).
- **Pantallas:**
    1. **Login:** Ingreso de credenciales de vendedor.
    2. **Nueva Venta:** Campo para monto y descripción. Botón "Generar QR".
    3. **Visualización QR:** Muestra el código QR generado, con opción de aumentar brillo y compartir la cadena alfanumérica (para pago remoto).
    4. **Historial:** Lista de ventas con estados (pendiente, pagado, cancelado).
    5. **Configuración:** Gestión de agentes (si el negocio tiene varios vendedores).

### 4.3. Generación del Código QR

- **Librerías:**
    - **Python:** `qrcode` o `segno`.
    - **Flutter:** `qr_flutter`.
- **Formato de Datos:** El QR debe contener exactamente la cadena que la app Transfermóvil espera. Según un ejemplo de código en GitHub (aunque no oficial), la estructura podría ser similar a: `{"id_transaccion": "...", "monto": "...", ...}`. **Es crucial validar este formato con ETECSA o mediante pruebas controladas.**

---

## 5. 🧪 Fase 4: Pruebas y Validación

- **Pruebas Unitarias:** Para la lógica de generación de QR y comunicación con el backend.
- **Pruebas de Integración:** Con el entorno de pruebas (sandbox) de ETECSA, si está disponible. La plataforma UIC ofrece "dos espacios (sandbox) para realizar pruebas de sus integraciones".
- **Pruebas de Usuario:** Con comercios reales en un entorno controlado, verificando que los pagos se reflejen correctamente en el Bulevar Mi Transfer.

---

## 6. 🚀 Fase 5: Despliegue y Distribución

- **Distribución:** Dado que las restricciones para publicar en Google Play desde Cuba son conocidas, la distribución puede realizarse a través de **Apklis** (tienda de aplicaciones cubana) o mediante instalación directa del APK.
- **Infraestructura:** El backend puede alojarse en servidores cubanos (ej. en la UCI o ETECSA) o en la nube, siempre cumpliendo con las regulaciones de datos del país.

---

## 7. ⚠️ Desafíos y Riesgos Clave

| Riesgo | Mitigación |
|--------|------------|
| **Falta de documentación oficial de la API** | Contactar directamente a ETECSA a través de `negociosdigitales@etecsa.cu` para obtener acceso y documentación técnica. |
| **Conectividad limitada** | Diseñar la app para funcionar offline: el QR se puede generar localmente una vez obtenidos los datos, y la sincronización se realiza cuando haya conexión. |
| **Cambios en la plataforma de ETECSA** | Mantener una capa de abstracción en el backend para adaptarse rápidamente a cambios en los endpoints o formatos. |
| **Cumplimiento regulatorio** | Asesorarse legalmente sobre el Decreto-Ley 370 y obtener las licencias necesarias para operar como proveedor de software. |

---

## 8. 📅 Cronograma Estimado (3-6 Meses)

| Fase | Duración | Hitos |
|------|----------|-------|
| Análisis y Contratación | 4-8 semanas | Contrato con ETECSA, obtención de credenciales y documentación de API. |
| Desarrollo Backend | 4-6 semanas | API de integración funcional, pruebas unitarias. |
| Desarrollo Frontend | 4-6 semanas | App móvil con generación de QR y flujo completo. |
| Pruebas e Integración | 3-4 semanas | Validación en sandbox, pruebas con usuarios reales. |
| Despliegue y Capacitación | 2-4 semanas | Publicación en Apklis, manual de usuario y soporte. |

---

Este plan te proporciona una hoja de ruta clara. El paso más crítico y determinante es **contactar a ETECSA** para obtener acceso a la API de Bulevar Mi Transfer, ya que sin ella, la aplicación no podrá generar QR dinámicos válidos. Una vez que tengas esa documentación, el desarrollo técnico es totalmente viable con las herramientas mencionadas.
