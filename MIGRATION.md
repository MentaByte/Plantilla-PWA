# 📋 Guía de Migración: v1.0 → v1.3

## ✅ ¿Por qué actualizar?

La versión 1.3 soluciona problemas críticos de compatibilidad con Safari iOS:
- ✅ **100% compatible con iPhone/iPad Safari**
- ✅ **Sin módulos ES6** (mayor compatibilidad)
- ✅ **Funciona en ambas plataformas** sin ajustes

---

## 🔄 Cambios principales

### 1. auth.js
**Cambio:** `crypto.randomUUID()` → función compatible

**Antes (v1.0):**
```javascript
export function getDeviceId() {
  let id = localStorage.getItem("device_id");
  if (!id) {
    id = crypto.randomUUID(); // ❌ NO funciona en Safari iOS
    localStorage.setItem("device_id", id);
  }
  return id;
}
```

**Ahora (v1.3):**
```javascript
export function getDeviceId() {
  let id = localStorage.getItem("device_id");
  if (!id) {
    // ✅ Compatible Safari iOS
    id = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
      var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
      return v.toString(16);
    });
    localStorage.setItem("device_id", id);
  }
  return id;
}
```

### 2. index.html
**Cambio:** Eliminados módulos ES6

**Antes (v1.0):**
```html
<script type="module">
  import { activateLicense, saveSession } from "./auth.js";
  // ...
</script>
```

**Ahora (v1.3):**
```html
<script>
  // Funciones de auth copiadas inline (sin import)
  const SUPABASE_URL = "...";
  const SUPABASE_ANON_KEY = "...";
  
  function getDeviceId() { /* ... */ }
  async function activateLicense(code) { /* ... */ }
  function saveSession(token) { /* ... */ }
  // ...
</script>
```

### 3. core/index.html
**Cambio:** Igual que index.html, sin módulos ES6

**Antes (v1.0):**
```html
<script type="module">
  import { validateSession, clearSession } from "../auth.js";
  // ...
</script>
```

**Ahora (v1.3):**
```html
<script>
  // Funciones de auth inline
  const SUPABASE_URL = "...";
  const SUPABASE_ANON_KEY = "...";
  
  function getDeviceId() { /* ... */ }
  async function validateSession() { /* ... */ }
  function clearSession() { /* ... */ }
  // ...
</script>
```

---

## 📝 Cómo actualizar tus PWAs existentes

### Opción A: Actualización mínima (solo Safari fix)

**Reemplaza solo:**
1. `auth.js` (v1.3)

**Resultado:** 
- ✅ Safari funcionará
- ⚠️ Sigue usando módulos ES6 (puede tener problemas en Safari en algunos casos)

### Opción B: Actualización completa (recomendada)

**Reemplaza:**
1. `auth.js` (v1.3)
2. `index.html` (v1.3)
3. `core/index.html` (v1.3 - manteniendo tu contenido personalizado)

**Resultado:**
- ✅ Safari funcionará perfectamente
- ✅ Chrome seguirá funcionando
- ✅ Mayor compatibilidad general

---

## 🔧 Actualizar core/index.html (manteniendo tu código)

Si tienes una PWA personalizada en `/core/index.html`, sigue estos pasos:

### 1. Copia tu contenido actual
Guarda tu HTML personalizado (la parte del `<div id="app">`)

### 2. Reemplaza el script de validación
**Elimina:**
```html
<script type="module">
  import { validateSession, clearSession } from "../auth.js";
  // tu código...
</script>
```

**Reemplaza con:**
```html
<script>
  // Funciones de auth inline (de la plantilla v1.3)
  const SUPABASE_URL = "https://zgatennqbagmyfbpiakr.supabase.co";
  const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...";

  function getDeviceId() {
    let id = localStorage.getItem("device_id");
    if (!id) {
      id = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
      });
      localStorage.setItem("device_id", id);
    }
    return id;
  }

  async function validateSession() {
    const session_token = localStorage.getItem("session_token");
    const device_id = getDeviceId();
    if (!session_token) return { valid: false, error: "no_token" };
    try {
      const res = await fetch(`${SUPABASE_URL}/functions/v1/validate`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${SUPABASE_ANON_KEY}`,
        },
        body: JSON.stringify({ session_token, device_id }),
      });
      const result = await res.json();
      if (result.valid === true) {
        localStorage.setItem("last_known_status", "valid");
      } else if (result.error === "revoked") {
        localStorage.setItem("last_known_status", "revoked");
      }
      return result;
    } catch (e) {
      const lastStatus = localStorage.getItem("last_known_status");
      if (lastStatus === "revoked") {
        return { valid: false, error: "revoked" };
      }
      return { valid: false, no_internet: true };
    }
  }

  function clearSession() {
    localStorage.removeItem("session_token");
    localStorage.removeItem("last_known_status");
  }

  // Aquí va tu código personalizado...
</script>
```

### 3. Mantén tu código personalizado
Todo tu código de funcionalidad va después de las funciones de auth.

---

## ✅ Checklist de actualización

Por cada PWA existente:

- [ ] Descargar plantilla v1.3
- [ ] Reemplazar `auth.js`
- [ ] Reemplazar `index.html`
- [ ] Actualizar `core/index.html` (script de validación)
- [ ] Probar en Chrome/Android (debe seguir funcionando)
- [ ] **Probar en Safari/iOS** (debe funcionar ahora)
- [ ] Verificar funcionamiento offline
- [ ] Verificar validación de sesión
- [ ] Verificar revocación

---

## 🐛 Problemas comunes

### "La app no carga en Safari"
✅ **Solución:** Asegúrate de haber actualizado tanto `auth.js` como los archivos HTML

### "Error: crypto.randomUUID is not a function"
✅ **Solución:** Todavía estás usando `auth.js` v1.0, reemplázalo con v1.3

### "La app funciona en Chrome pero no en Safari"
✅ **Solución:** Verifica que NO estés usando módulos ES6 (`type="module"`)

### "Import statement not allowed" en Safari
✅ **Solución:** Todavía tienes `import` statements, usa la versión inline sin módulos

---

## 📊 Comparación de versiones

| Característica | v1.0 | v1.3 |
|----------------|------|------|
| Chrome/Android | ✅ | ✅ |
| Safari/iOS | ❌ | ✅ |
| Módulos ES6 | Sí | No |
| crypto.randomUUID | Sí | No (alternativa) |
| Compatibilidad | Limitada | Universal |
| Código | Modular | Inline |

---

## 🎯 Resumen

**Para nuevas PWAs:** Usa siempre la plantilla v1.3

**Para PWAs existentes:** Actualiza a v1.3 para compatibilidad Safari

**Archivo crítico:** `auth.js` DEBE ser v1.3 para que funcione en Safari

---

**✅ Con v1.3 tus PWAs funcionarán en iOS y Android sin problemas**
