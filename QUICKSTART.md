# ⚡ Guía Rápida de Implementación

## 🎯 5 Minutos para tu PWA

### Paso 1: Copia los archivos (1 min)
```bash
# Copia toda la carpeta pwa-template-auth a tu proyecto
cp -r pwa-template-auth/ mi-nueva-app/
cd mi-nueva-app/
```

### Paso 2: Personaliza el nombre (2 min)

**manifest.json:**
```json
{
  "name": "TuApp",        // ← Cambia aquí
  "short_name": "TuApp",  // ← Y aquí
  ...
}
```

**sw.js:**
```javascript
const CACHE_NAME = 'tu-app-v1.0';  // ← Cambia aquí
```

### Paso 3: Crea tu app en /core/index.html (30 seg)
- Reemplaza el contenido de ejemplo con tu aplicación
- ⚠️ NO toques el código de validación de sesión

### Paso 4: Agrega tus recursos al caché (1 min)
En `sw.js`, sección `PRECACHE`:
```javascript
const PRECACHE = [
  BASE,
  BASE + 'index.html',
  BASE + 'auth.js',
  BASE + 'core/index.html',
  BASE + 'fenix.html',
  BASE + 'manifest.json',
  // ← Agrega tus archivos aquí:
  BASE + 'styles/main.css',
  BASE + 'scripts/app.js',
  BASE + 'images/logo.png',
];
```

### Paso 5: Genera códigos en Supabase (30 seg)
En Supabase → SQL Editor:
```sql
INSERT INTO licenses DEFAULT VALUES
RETURNING code;
```

Obtendrás algo como: `MF-A1B2-C3D4-E5F6`

## 🚀 ¡Listo!

Comparte el link de activación:
```
https://tudominio.com/?k=MF-A1B2-C3D4-E5F6
```

---

## 📋 Checklist de Pre-lanzamiento

- [ ] Cambié el nombre en manifest.json
- [ ] Cambié CACHE_NAME en sw.js
- [ ] Agregué mis recursos al array PRECACHE
- [ ] Creé mi app en /core/index.html
- [ ] Generé al menos 1 código de prueba
- [ ] Probé la activación con ese código
- [ ] Probé el funcionamiento offline
- [ ] Agregué los iconos (icon-192.png, icon-512.png)
- [ ] Probé la instalación como PWA
- [ ] Probé revocar un código

---

## 🎨 Personalización Opcional

### Cambiar colores del tema
En `manifest.json`:
```json
"background_color": "#tu-color-hex",
"theme_color": "#tu-color-hex"
```

En `index.html` (activación):
```css
background: linear-gradient(135deg, #color1 0%, #color2 100%);
```

### Agregar más páginas de error
Copia `fenix.html` y personalízalo para otros escenarios.

### Integrar con APIs externas
En `/core/index.html`, después de validar la sesión, puedes hacer fetch a tus APIs.

---

## 🔧 Comandos Útiles

### Generar múltiples códigos
```sql
-- Genera 10 códigos
INSERT INTO licenses DEFAULT VALUES
FROM generate_series(1, 10)
RETURNING code;
```

### Ver todos los códigos activos
```sql
SELECT code, device_id, activated_at
FROM licenses
WHERE revoked = FALSE
  AND device_id IS NOT NULL;
```

### Revocar un código
```sql
UPDATE licenses
SET revoked = TRUE
WHERE code = 'MF-XXXX-XXXX-XXXX';
```

### Resetear un código (para reusar)
```sql
UPDATE licenses
SET device_id = NULL,
    session_token = NULL,
    activated_at = NULL,
    last_validated = NULL,
    revoked = FALSE
WHERE code = 'MF-XXXX-XXXX-XXXX';
```

---

## 💡 Tips Pro

1. **Versionado de caché**: Cambia `CACHE_NAME` cada vez que actualices recursos
2. **Testing**: Prueba siempre en modo incógnito para evitar cachés viejos
3. **Iconos**: Usa https://realfavicongenerator.net/ para generar todos los tamaños
4. **HTTPS**: Las PWAs requieren HTTPS en producción (localhost funciona sin HTTPS)
5. **Debugging**: Chrome DevTools → Application → Service Workers

---

## 🆘 Problemas Comunes

**"Failed to register service worker"**
→ Verifica que estés en HTTPS o localhost

**El caché no se actualiza**
→ Cambia el CACHE_NAME en sw.js

**"Acceso no válido" con código correcto**
→ Verifica que revoked = false en la base de datos

**No funciona offline**
→ Abre la app al menos una vez con internet

---

¡Éxito con tu PWA! 🎉
