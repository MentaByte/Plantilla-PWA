# 📁 Estructura de la Plantilla PWA v1.3

## Arquitectura de archivos

```
tu-pwa/
│
├── 📄 index.html              # Página de activación (raíz)
│   └── Valida código de licencia
│   └── Guarda sesión
│   └── Redirige a /core/
│
├── 🔐 auth.js                 # Sistema de autenticación v1.3
│   └── getDeviceId()          # ✅ Compatible Safari
│   └── activateLicense()
│   └── validateSession()
│   └── saveSession()
│   └── clearSession()
│
├── 📱 manifest.json           # Configuración PWA
│   └── Nombre de la app
│   └── Iconos
│   └── Colores del tema
│   └── Display mode
│
├── ⚙️ sw.js                   # Service Worker
│   └── CACHE_NAME
│   └── PRECACHE (archivos a cachear)
│   └── Estrategias de caché
│   └── Manejo offline
│
├── 🚫 fenix.html              # Página de acceso revocado
│   └── Mensaje de acceso denegado
│   └── Opción de contacto
│
└── 📂 core/
    └── 📄 index.html          # Tu aplicación principal
        └── Validación automática
        └── Interfaz de usuario
        └── Funcionalidad principal
        └── ✅ Sin módulos ES6
```

---

## 🔄 Flujo de datos

### 1. Primera activación
```
Usuario recibe link con código
        ↓
https://tuapp.com/?k=MF-XXXX-XXXX-XXXX
        ↓
index.html carga
        ↓
auth.js: activateLicense(code)
        ↓
Supabase valida código
        ↓
    ┌──────────┴──────────┐
    ↓                     ↓
✅ Válido              ❌ Error
    ↓                     ↓
saveSession()         Muestra error
    ↓                     ↓
localStorage          Usuario ve mensaje
    ↓
Redirige a /core/
    ↓
App cargada
```

### 2. Validación en cada carga
```
Usuario abre /core/
        ↓
initApp() ejecuta
        ↓
validateSession()
        ↓
    ┌──────┴──────┐
    ↓             ↓
✅ Válida      ❌ Inválida
    ↓             ↓
Muestra app   Redirige
    ↓             ↓
              ┌───┴───┐
              ↓       ↓
          Revoked   Sin sesión
              ↓       ↓
          fenix.html  index.html
```

---

## 💾 LocalStorage Schema

```javascript
{
  "device_id": "uuid-v4-format",
  "session_token": "token-from-supabase",
  "last_known_status": "valid" | "revoked"
}
```

---

## 🌐 Endpoints de Supabase

### 1. Activación
```
POST /functions/v1/activate
Body: {
  code: "MF-XXXX-XXXX-XXXX",
  device_id: "uuid"
}
Response: {
  session_token: "token" | null,
  error: "invalid_code" | "revoked" | "device_mismatch"
}
```

### 2. Validación
```
POST /functions/v1/validate
Body: {
  session_token: "token",
  device_id: "uuid"
}
Response: {
  valid: true | false,
  error: "revoked" | "invalid_token" | "device_mismatch"
}
```

---

## 🎨 Puntos de personalización

### ✏️ Para cada nueva PWA, personaliza:

1. **manifest.json**
   - name
   - short_name
   - description
   - background_color
   - theme_color

2. **sw.js**
   - CACHE_NAME (nombre único)
   - PRECACHE (archivos de tu app)

3. **core/index.html**
   - Todo el contenido de tu aplicación
   - Estilos personalizados
   - Funcionalidades específicas

### 🚫 NO modificar:

1. **auth.js** - Sistema de autenticación compartido
2. **index.html** - Página de activación estándar
3. **fenix.html** - Página de revocación estándar

---

## 📦 Archivos en caché (Service Worker)

### Estrategia Cache-First:
- `index.html`
- `fenix.html`
- `manifest.json`
- Iconos (icon-192.png, icon-512.png)
- Tus recursos estáticos (CSS, imágenes, etc.)

### Estrategia Network-First:
- `core/index.html` (siempre validar primero)
- `auth.js` (siempre obtener versión fresca)

### Never Cache:
- `*.supabase.co` (APIs de Supabase)

---

## 🔧 Compatibilidad Safari iOS

### ✅ v1.3 incluye:

**auth.js:**
```javascript
// ❌ Antes (NO funciona en Safari)
id = crypto.randomUUID();

// ✅ Ahora (Compatible Safari)
id = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
  var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
  return v.toString(16);
});
```

**index.html y core/index.html:**
```javascript
// ❌ Antes (NO funciona bien en Safari)
<script type="module">
import { activateLicense } from './auth.js';

// ✅ Ahora (Compatible Safari)
<script>
// Funciones inline sin módulos ES6
```

---

## 🚀 Despliegue

### Checklist de despliegue:

1. ✅ Archivos personalizados
2. ✅ CACHE_NAME único en sw.js
3. ✅ Iconos agregados (192x192 y 512x512)
4. ✅ Todos los archivos en PRECACHE
5. ✅ Probado en Chrome
6. ✅ **Probado en Safari iOS** ✅
7. ✅ Funciona offline
8. ✅ Validación funciona
9. ✅ Revocación funciona
10. ✅ Códigos generados en Supabase

---

**v1.3 - Compatible Safari iOS y Chrome Android**
