# 📁 Estructura del Proyecto

```
pwa-template-auth/
│
├── 📄 index.html              # Página de activación (recibe ?k=CODIGO)
├── 🔐 auth.js                 # Lógica de autenticación (NO MODIFICAR)
├── ⚙️ sw.js                   # Service Worker (PERSONALIZAR caché)
├── 📱 manifest.json           # Configuración PWA (PERSONALIZAR nombre/colores)
├── 🚫 fenix.html              # Página de acceso revocado (PERSONALIZAR diseño)
│
├── 📂 core/                   # 🎯 TU APLICACIÓN VA AQUÍ
│   └── 📄 index.html          # App principal protegida
│
├── 📂 assets/                 # (CREAR) Tus recursos
│   ├── 🖼️ images/
│   ├── 🎨 styles/
│   └── 📜 scripts/
│
├── 🖼️ icon-192.png            # (AGREGAR) Ícono PWA 192x192
├── 🖼️ icon-512.png            # (AGREGAR) Ícono PWA 512x512
│
├── 📖 README.md               # Documentación completa
├── ⚡ QUICKSTART.md           # Guía rápida (5 minutos)
├── 📊 DATABASE.md             # Referencia de base de datos
├── 💡 EXAMPLES.md             # Casos de uso y ejemplos
└── 🚫 .gitignore              # Archivos a ignorar en Git
```

---

## 📄 Descripción de Archivos

### Archivos que NO debes modificar ❌

| Archivo | Descripción | ¿Modificar? |
|---------|-------------|-------------|
| `auth.js` | Sistema de autenticación | ❌ NO |

### Archivos que DEBES personalizar ✅

| Archivo | Descripción | ¿Modificar? |
|---------|-------------|-------------|
| `manifest.json` | Nombre, colores, iconos de la PWA | ✅ SÍ |
| `sw.js` | Nombre del caché y lista de recursos | ✅ SÍ |
| `core/index.html` | Tu aplicación principal | ✅ SÍ |
| `index.html` | Página de activación (opcional) | 🟡 OPCIONAL |
| `fenix.html` | Página de acceso revocado (opcional) | 🟡 OPCIONAL |

### Archivos que DEBES crear 🆕

| Archivo | Descripción | Obligatorio |
|---------|-------------|-------------|
| `icon-192.png` | Ícono 192x192 | ✅ SÍ |
| `icon-512.png` | Ícono 512x512 | ✅ SÍ |
| `assets/*` | Tus recursos (CSS, JS, imágenes) | 🟡 SEGÚN NECESITES |

---

## 🎯 Flujo de Archivos

```
Usuario recibe link
    ↓
/?k=MF-XXXX-XXXX-XXXX
    ↓
┌─────────────────┐
│  index.html     │ ← Valida código con auth.js
└─────────────────┘
    ↓
    ├─ ✅ Válido → Guarda sesión
    │               ↓
    │           Redirige a /core/
    │               ↓
    │       ┌─────────────────┐
    │       │ core/index.html │ ← Valida sesión
    │       └─────────────────┘
    │               ↓
    │           ┌──────┴──────┐
    │           ↓             ↓
    │       Válida        Revocada
    │           ↓             ↓
    │       Carga App    Redirige a fenix.html
    │
    └─ ❌ Inválido → Muestra error
```

---

## 🔄 Service Worker - Qué cachea

```
sw.js
├── Cache First (rápido, offline)
│   ├── HTML estáticos
│   ├── CSS
│   ├── JS
│   ├── Imágenes
│   └── Fuentes
│
├── Network First (siempre fresco)
│   ├── core/index.html
│   └── auth.js
│
└── Never Cache (siempre online)
    └── *.supabase.co
```

---

## 📦 Recursos que necesitas agregar al caché

En `sw.js`, sección `PRECACHE`, agrega:

```javascript
const PRECACHE = [
  BASE,
  BASE + 'index.html',
  BASE + 'auth.js',
  BASE + 'core/index.html',
  BASE + 'fenix.html',
  BASE + 'manifest.json',
  
  // ✅ Agrega tus archivos aquí:
  BASE + 'assets/styles/main.css',
  BASE + 'assets/scripts/app.js',
  BASE + 'assets/images/logo.png',
  BASE + 'icon-192.png',
  BASE + 'icon-512.png',
  // ...más recursos
];
```

---

## 🎨 Personalización por Archivo

### manifest.json
```json
{
  "name": "Tu App",           // ← Nombre completo
  "short_name": "App",        // ← Nombre corto
  "description": "...",       // ← Descripción
  "background_color": "#hex", // ← Color de fondo al abrir
  "theme_color": "#hex"       // ← Color de la barra superior
}
```

### sw.js
```javascript
// Línea 7:
const CACHE_NAME = 'tu-app-v1.0';  // ← Nombre único del caché

// Líneas 11-25:
const PRECACHE = [
  // ← Lista de archivos a cachear
];
```

### core/index.html
```html
<!-- Línea 6: -->
<meta name="theme-color" content="#tu-color">

<!-- Línea 8: -->
<title>Tu App</title>

<!-- Líneas 15+: -->
<!-- Todo tu contenido HTML aquí -->
```

---

## 🚀 Orden de Implementación Recomendado

1. ✅ Copia todos los archivos
2. ✅ Personaliza `manifest.json` (nombre, colores)
3. ✅ Actualiza `CACHE_NAME` en `sw.js`
4. ✅ Crea tu app en `core/index.html`
5. ✅ Agrega tus recursos a `assets/`
6. ✅ Lista tus recursos en `PRECACHE` (sw.js)
7. ✅ Genera íconos y agrégalos a la raíz
8. ✅ Prueba con un código de Supabase
9. ✅ Verifica funcionamiento offline
10. ✅ ¡Listo para producción!

---

## 📱 Testing Local

```bash
# 1. Sirve los archivos con un servidor local
npx serve .
# o
python -m http.server 8000

# 2. Abre en navegador
http://localhost:8000/?k=TU-CODIGO-AQUI

# 3. DevTools → Application → Service Workers
# Verifica que esté activo

# 4. DevTools → Application → Cache Storage
# Verifica que los archivos estén cacheados

# 5. DevTools → Network
# Activa "Offline" y recarga
# La app debe funcionar sin internet
```

---

## 🆘 Archivos Faltantes Comunes

**Error:** Manifest no carga
→ Falta `manifest.json` o tiene JSON inválido

**Error:** Service Worker no registra
→ Verifica que `sw.js` esté en la raíz

**Error:** Íconos rotos
→ Agrega `icon-192.png` e `icon-512.png`

**Error:** Assets no cargan offline
→ Agrégalos al array `PRECACHE` en `sw.js`

**Error:** "Cannot read property 'replace' of undefined"
→ Service Worker mal configurado, revisa rutas en `sw.js`

---

## 📊 Tamaños Recomendados

| Tipo | Tamaño Recomendado |
|------|-------------------|
| Ícono 192x192 | ~10-20 KB |
| Ícono 512x512 | ~30-50 KB |
| Imágenes | Máx 500 KB c/u |
| CSS | Máx 100 KB |
| JS | Máx 500 KB |
| Total caché | Máx 50 MB |

---

**Tip:** Usa herramientas como TinyPNG para optimizar imágenes antes de agregarlas.

---

¡Tu PWA está lista para despegar! 🚀
