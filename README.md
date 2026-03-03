# 🔐 Plantilla PWA con Autenticación por Código

Esta plantilla te permite crear PWAs con sistema de autenticación basado en códigos de licencia, sin necesidad de modificar la base de datos ni las funciones de Supabase.

## 📋 Características

- ✅ Autenticación mediante código único de activación
- ✅ Validación de dispositivo (un código = un dispositivo)
- ✅ Detección de revocación de acceso
- ✅ Funcionamiento offline con Service Worker
- ✅ Almacenamiento de sesión persistente
- ✅ Manejo de errores sin conexión

## 🗂️ Estructura de archivos

```
tu-app/
├── index.html          # Página de activación (raíz)
├── auth.js             # Lógica de autenticación
├── manifest.json       # Configuración PWA
├── sw.js              # Service Worker
├── fenix.html         # Página de acceso denegado
└── core/
    └── index.html     # Tu aplicación principal
```

## 🚀 Guía de implementación

### 1. Configura tu proyecto

1. Copia todos los archivos de esta plantilla a tu proyecto
2. **NO modifiques** `auth.js` (usa las mismas credenciales de Supabase)
3. Personaliza los siguientes archivos según tu app:

### 2. Personaliza `manifest.json`

```json
{
  "name": "TuApp",              // ← Cambia esto
  "short_name": "TuApp",        // ← Cambia esto
  "description": "...",         // ← Cambia esto
  "background_color": "#color", // ← Cambia esto
  "theme_color": "#color"       // ← Cambia esto
}
```

### 3. Personaliza `sw.js`

Actualiza estas variables:

```javascript
const CACHE_NAME = 'tu-app-v1.0';  // ← Nombre único de caché

const PRECACHE = [
  BASE,
  BASE + 'index.html',
  BASE + 'auth.js',
  BASE + 'core/index.html',
  BASE + 'fenix.html',
  BASE + 'manifest.json',
  // ← Agrega aquí tus recursos (imágenes, CSS, JS, etc.)
];
```

### 4. Crea tu aplicación en `/core/index.html`

Este es el archivo donde va tu aplicación principal. La plantilla incluye:
- Validación automática de sesión al cargar
- Redirección a `/fenix.html` si el acceso es revocado
- Función para cerrar sesión

**Estructura básica incluida:**

```javascript
import { validateSession, clearSession } from "../auth.js";

// Validar sesión al cargar
const result = await validateSession();

if (result.valid) {
  // ✅ Sesión válida - cargar app
} else if (result.error === "revoked") {
  // ❌ Acceso revocado - redirigir
  window.location.href = "../fenix.html";
} else if (result.no_internet) {
  // ⚠️ Sin internet - usar caché
}
```

### 5. Genera códigos de licencia en Supabase

**Opción A: Desde la interfaz de Supabase**
1. Ve a Table Editor → `licenses`
2. Insert row
3. Deja el `code` vacío (se genera automático)
4. Guarda

**Opción B: Desde SQL Editor**
```sql
INSERT INTO licenses DEFAULT VALUES
RETURNING code;
```

Esto generará un código como: `MF-XXXX-XXXX-XXXX`

### 6. Comparte el código con tus usuarios

**Formato del link de activación:**
```
https://tudominio.com/?k=MF-XXXX-XXXX-XXXX
```

## 🔄 Flujo de autenticación

```
Usuario recibe link
    ↓
/?k=MF-XXXX-XXXX-XXXX
    ↓
index.html valida código
    ↓
    ├─ ✅ Válido → Guarda sesión → Redirige a /core/
    ├─ ❌ Inválido → Muestra error
    ├─ 🚫 Revocado → Muestra error
    └─ 📱 Ya usado en otro device → Muestra error
```

## 🛡️ Revocar acceso

Para revocar el acceso de un usuario:

```sql
UPDATE licenses
SET revoked = TRUE
WHERE code = 'MF-XXXX-XXXX-XXXX';
```

La próxima vez que el usuario abra la app (con internet), será redirigido a `fenix.html`.

## 📱 Instalación como PWA

Los usuarios pueden instalar tu app siguiendo las instrucciones de su navegador:

- **Chrome/Edge:** Menú → Instalar app
- **Safari iOS:** Compartir → Agregar a pantalla de inicio
- **Firefox:** Menú → Instalar

## ⚡ Modo Offline

El Service Worker permite que tu app funcione sin conexión después de la primera carga:

- Cache-first para archivos locales
- Network-first para `core/index.html` y `auth.js` (validación fresca)
- Fallback a página offline si no hay conexión

## 🎨 Personalización avanzada

### Cambiar el mensaje de error en `fenix.html`

Edita el contenido HTML según tu marca y estilo.

### Agregar más validaciones

Puedes extender `validateSession()` en `auth.js` si necesitas validaciones adicionales (aunque no es recomendado para mantener compatibilidad).

### Cambiar el formato del código

Por defecto usa el formato `MF-XXXX-XXXX-XXXX`. Si quieres cambiarlo, deberás modificar la función `generate_license_code()` en Supabase (no recomendado).

## 📊 Tabla de Supabase (solo referencia)

**NO necesitas crear nada, ya existe:**

```sql
CREATE TABLE licenses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code TEXT UNIQUE DEFAULT generate_license_code(),
  device_id TEXT,
  session_token TEXT UNIQUE,
  activated_at TIMESTAMPTZ,
  last_validated TIMESTAMPTZ,
  revoked BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## 🔧 Troubleshooting

### "Acceso no válido" al abrir el link
- Verifica que el código exista en la base de datos
- Confirma que `revoked = false`

### La app no carga offline
- Asegúrate de haber abierto la app al menos una vez con internet
- Verifica que todos los archivos estén en el array `PRECACHE` de `sw.js`

### "Este código ya está activo en otro dispositivo"
- Cada código solo puede usarse en UN dispositivo
- Si necesitas cambiar de dispositivo, debes generar un nuevo código

### La validación siempre dice "sin internet"
- Verifica que las credenciales de Supabase en `auth.js` sean correctas
- Confirma que las Edge Functions estén desplegadas

## 📝 Checklist de implementación

- [ ] Copiar todos los archivos de la plantilla
- [ ] Personalizar `manifest.json`
- [ ] Actualizar `CACHE_NAME` y `PRECACHE` en `sw.js`
- [ ] Crear tu app en `/core/index.html`
- [ ] Generar códigos de licencia en Supabase
- [ ] Probar activación con un código válido
- [ ] Probar revocación de acceso
- [ ] Probar funcionamiento offline
- [ ] Agregar iconos (icon-192.png, icon-512.png)

## 🎯 Listo para producción

Una vez personalizada, tu PWA estará lista para:
- Distribuir códigos de acceso
- Funcionar 100% offline después de la primera carga
- Validar sesiones automáticamente
- Revocar accesos cuando sea necesario

---

**Creado para usar con la infraestructura de MercadoForzado**
