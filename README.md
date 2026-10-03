# Control

**Control** es una aplicación web progresiva (PWA) de gestión financiera para negocios pequeños y personales: ventas, gastos, inventario, deudas de clientes y proveedores, estadísticas y reportes — todo sincronizado en la nube y accesible desde el celular (instalada como app) o desde cualquier navegador.

Construida como una aplicación de una sola página (HTML + CSS + JavaScript puro, sin frameworks ni paso de compilación), con autenticación y sincronización de datos en tiempo real mediante Firebase.

---

## Índice

- [Características generales](#características-generales)
- [Páginas de la aplicación](#páginas-de-la-aplicación)
- [Cuenta y sincronización en la nube](#cuenta-y-sincronización-en-la-nube)
- [Reportes](#reportes)
- [Importar datos anteriores](#importar-datos-anteriores)
- [Instalación como app (PWA)](#instalación-como-app-pwa)
- [Seguridad y privacidad](#seguridad-y-privacidad)
- [Stack técnico](#stack-técnico)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Configuración de Firebase](#configuración-de-firebase)
- [Despliegue en GitHub Pages](#despliegue-en-github-pages)
- [Generar el .apk para Android](#generar-el-apk-para-android)
- [Nota legal](#nota-legal)

---

## Características generales

- **Registro de ingresos y egresos** con categoría, medio de pago, nota y fecha editable.
- **Cuentas por cobrar y por pagar** (fiado a clientes, compras a crédito con proveedores), con seguimiento de saldo por persona.
- **Catálogo de productos** con stock, usado tanto en el módulo de ventas como en inventario.
- **Punto de venta simple**: carrito de productos, cálculo de total y registro automático del ingreso y descuento de stock.
- **Filtros de período** en Balance y Estadísticas: Diario, Semanal, Mensual, **Trimestral** y Anual, con navegación hacia adelante/atrás (flechas o deslizando el dedo sobre la fecha) y selectores específicos (calendario, mes+año, trimestre+año, año).
- **Calendario propio** que siempre empieza en lunes (no depende del idioma/región del dispositivo).
- **Reportes** descargables en Excel con diseño a color, y una vista imprimible/PDF con el mismo diseño (ver sección [Reportes](#reportes)).
- **Importación de reportes antiguos** (de antes de usar Control) desde un archivo Excel, sin duplicar datos.
- **Modo oscuro/claro** y diseño responsive (menú lateral deslizable en pantallas grandes, barra de navegación inferior en celular).
- **Bloqueo con PIN** independiente del login, para proteger el acceso rápido al abrir la app.
- **Funciona sin internet** una vez cargada (service worker), y sincroniza los cambios cuando vuelve la conexión.
- **Multi-dispositivo**: lo que se registra en el celular aparece en la web y viceversa (gracias a la cuenta en la nube).

## Páginas de la aplicación

| Página | Qué hace |
|---|---|
| **Inicio** | Panel resumen: accesos rápidos (Vender, Balance, Inventario, Deudas), ventas/egresos/balance de hoy con variación vs. ayer, total por cobrar/por pagar, y los últimos movimientos registrados. |
| **Balance** | Libro de ingresos, egresos, por cobrar y por pagar. Filtro por período (con navegación), por categoría, buscador, edición/eliminación de movimientos, y acceso a los reportes. |
| **Vender** | Catálogo de productos con buscador, carrito, cálculo de total, selección de medio de pago y confirmación — registra la venta y descuenta stock automáticamente. |
| **Estadísticas** | Productos más vendidos y gastos por categoría en el período seleccionado. |
| **Inventario** | Lista de productos con su stock; permite ajustar stock con +/-, editar nombre/precio/stock, o crear productos nuevos. |
| **Clientes** | Lista de clientes con su saldo (lo que te deben). Ficha por cliente con historial de fiados y abonos. |
| **Proveedores** | Igual que Clientes pero al revés: compras a crédito y pagos a proveedores. |
| **Configuraciones** | Perfil del negocio (nombre, propietario, teléfono opcional, moneda), seguridad (PIN), importar movimientos desde Excel, respaldo completo de datos, borrar todos los datos, cerrar sesión. |
| **Ayuda** | Preguntas frecuentes sobre el uso de la app. |

## Cuenta y sincronización en la nube

Control usa **Firebase Authentication** (correo y contraseña) y **Cloud Firestore** para que los datos no dependan de un solo dispositivo:

- Cada usuario tiene su propia cuenta; los datos se guardan en `users/{uid}/data/...` en Firestore, con reglas de seguridad que solo permiten a cada usuario leer/escribir sus propios datos.
- Si se pierde o se daña el celular, los datos siguen disponibles entrando con la misma cuenta desde cualquier navegador.
- La app funciona offline (Firestore guarda localmente y sincroniza al reconectar).

## Reportes

Desde Balance → "Descargar reporte" se abre un selector de período en una sola pantalla (pestañas Diario/Semanal/Mensual/Trimestral/Anual + el detalle exacto: fecha, mes, trimestre o año), con dos salidas:

1. **Ver reporte para imprimir / PDF**: abre una vista con diseño propio — encabezado con nombre del negocio, teléfono (si se configuró) y número de transacciones, tarjetas de resumen (Ingresos/Egresos/Balance) a color, y la tabla de movimientos con totales. Tiene un botón para imprimir o guardar como PDF directamente desde el navegador.
2. **Descargar en Excel**: el mismo reporte como archivo `.xlsx`, con el mismo encabezado, resumen y tabla, con colores y formato de moneda reales (usa la librería [`xlsx-js-style`](https://github.com/gitbrent/xlsx-js-style), que sí permite escribir estilos — a diferencia de SheetJS Community estándar).

## Importar datos anteriores

Si el negocio ya tenía registros en otra app (por ejemplo, reportes exportados en Excel antes de migrar a Control), en **Configuraciones → "Importar movimientos anteriores"** se puede subir ese Excel: la app reconoce las columnas típicas de un reporte de ventas/gastos (Fecha, Tipo, Categoría, Contacto, Medio de pago, Valor), convierte cada fila en un movimiento, y evita duplicados aunque se vuelva a subir el mismo archivo.

## Instalación como app (PWA)

- **Android / escritorio**: el navegador ofrece "Agregar a pantalla de inicio" / "Instalar app" gracias al `manifest.json` y al ícono incluidos.
- **.apk generado con [PWABuilder](https://www.pwabuilder.com/)**: se puede empaquetar la PWA (apuntando a la URL publicada en GitHub Pages) como un `.apk` instalable de Android, para distribución fuera de la Play Store.
- Un **service worker** (`sw.js`) cachea los archivos estáticos y usa estrategia *network-first* para el HTML principal, de forma que las actualizaciones se reflejan sin necesidad de borrar caché manualmente (salvo cambios mayores de versión del service worker).

## Seguridad y privacidad

- Autenticación por correo/contraseña vía Firebase Auth.
- Reglas de Firestore que aíslan los datos de cada cuenta.
- PIN de bloqueo local opcional (independiente del login) para evitar que alguien con el celular desbloqueado entre directo a la app.
- Ningún dato se comparte con terceros; no hay anuncios ni analítica externa.

## Stack técnico

- **Frontend**: HTML + CSS + JavaScript vanilla (una sola página, sin build step, sin framework).
- **Autenticación y base de datos**: Firebase Authentication + Cloud Firestore (SDK compat v10.13.2).
- **Generación de Excel**: [`xlsx-js-style`](https://github.com/gitbrent/xlsx-js-style) (fork de SheetJS con soporte de estilos de celda), embebido directamente en el HTML.
- **PWA**: `manifest.json`, íconos, `sw.js` (service worker).
- **Empaquetado Android**: PWABuilder (TWA) para generar el `.apk`.
- **Gráficos**: SVG dibujado a mano (sin librerías externas de charts).

## Estructura del proyecto

```
├── index.html        # Toda la app (HTML + CSS + JS + librería de Excel embebida)
├── manifest.json      # Manifest de la PWA (nombre, íconos, colores, start_url)
├── sw.js              # Service worker (caché offline)
├── icon-192.png       # Ícono 192×192
├── icon-512.png       # Ícono 512×512
└── icon-180.png       # Ícono para iOS (apple-touch-icon)
```

> Nota: al ser una sola página con todo embebido, actualizar la app es tan simple como reemplazar `index.html` en el repositorio — no hay paso de build ni dependencias que instalar para desplegar.

## Configuración de Firebase

La app espera un objeto `firebaseConfig` embebido en `index.html` (dentro del `<script>` principal), con los datos del proyecto de Firebase:

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "....firebaseapp.com",
  projectId: "...",
  storageBucket: "....firebasestorage.app",
  messagingSenderId: "...",
  appId: "...",
};
```

Pasos para configurarlo desde cero:

1. Crear un proyecto en [Firebase Console](https://console.firebase.google.com/).
2. Habilitar **Authentication → Sign-in method → Correo electrónico/contraseña**.
3. Crear una base de datos **Cloud Firestore** (modo producción).
4. Configurar las reglas de seguridad:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId}/data/{doc} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
5. Registrar una **app web** dentro del proyecto para obtener el `firebaseConfig`, y pegarlo en `index.html`.

## Despliegue en GitHub Pages

1. Subir `index.html`, `manifest.json`, `sw.js` y los íconos a la raíz del repositorio (rama `main`).
2. En **Settings → Pages**, seleccionar la rama `main` y la carpeta raíz (`/`).
3. La app queda disponible en `https://<usuario>.github.io/<repositorio>/`.
4. Cada vez que se reemplace `index.html` con una versión nueva, el service worker la sirve actualizada automáticamente (estrategia *network-first* para el HTML).

## Generar el .apk para Android

1. Publicar la PWA en GitHub Pages (paso anterior) — PWABuilder necesita una URL pública con un `manifest.json` real (no sirve un manifest embebido como `data:` URI).
2. Entrar a [pwabuilder.com](https://www.pwabuilder.com/), pegar la URL de GitHub Pages.
3. Generar el paquete para Android (Trusted Web Activity) y descargar el `.apk`.
4. Instalar el `.apk` en el dispositivo (puede requerir habilitar "Instalar apps de orígenes desconocidos").

## Nota legal

Control está inspirada conceptualmente en apps de gestión financiera para pequeños negocios, pero es una implementación propia, sin usar su nombre, logo, código ni activos visuales. El diseño (estructura de navegación, layout de inicio, paleta de colores) fue elaborado de forma independiente para diferenciarse visualmente. Este README no constituye asesoría legal; si se planea monetizar la app, se recomienda verificar la disponibilidad de la marca "Control" y consultar con un abogado de propiedad intelectual antes del lanzamiento comercial.
