# 📊 Esquema de Base de Datos - Supabase

## ⚠️ NO NECESITAS CREAR NADA

Este archivo es solo para referencia. La tabla `licenses` y las funciones ya existen en Supabase y son compartidas por todas las PWAs que uses esta plantilla.

---

## Tabla: `licenses`

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

### Columnas

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | UUID | Identificador único del registro |
| `code` | TEXT | Código de licencia (formato: MF-XXXX-XXXX-XXXX) |
| `device_id` | TEXT | ID único del dispositivo que activó el código |
| `session_token` | TEXT | Token de sesión para validaciones |
| `activated_at` | TIMESTAMPTZ | Fecha/hora de primera activación |
| `last_validated` | TIMESTAMPTZ | Última vez que se validó la sesión |
| `revoked` | BOOLEAN | Si el acceso fue revocado (true/false) |
| `created_at` | TIMESTAMPTZ | Fecha/hora de creación del código |

---

## Función: `generate_license_code()`

Genera códigos únicos en formato `MF-XXXX-XXXX-XXXX`

```sql
CREATE OR REPLACE FUNCTION generate_license_code()
RETURNS TEXT AS $$
DECLARE
  chars TEXT := 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  result TEXT := 'MF-';
  i INTEGER;
BEGIN
  FOR i IN 1..3 LOOP
    result := result || 
      substr(chars, floor(random() * length(chars) + 1)::int, 1) ||
      substr(chars, floor(random() * length(chars) + 1)::int, 1) ||
      substr(chars, floor(random() * length(chars) + 1)::int, 1) ||
      substr(chars, floor(random() * length(chars) + 1)::int, 1);
    IF i < 3 THEN
      result := result || '-';
    END IF;
  END LOOP;
  RETURN result;
END;
$$ LANGUAGE plpgsql;
```

---

## Edge Function: `activate`

**Endpoint:** `/functions/v1/activate`

**Input:**
```json
{
  "code": "MF-XXXX-XXXX-XXXX",
  "device_id": "uuid-del-dispositivo"
}
```

**Output exitoso:**
```json
{
  "session_token": "token-unico-de-sesion"
}
```

**Output con error:**
```json
{
  "error": "invalid_code" | "revoked" | "device_mismatch"
}
```

### Lógica

1. Verifica que el código existe
2. Verifica que no está revocado
3. Si no tiene `device_id` asignado:
   - Asigna el `device_id` recibido
   - Genera y guarda un `session_token` único
   - Marca `activated_at` con la fecha actual
4. Si ya tiene `device_id`:
   - Verifica que coincida con el recibido
   - Si no coincide, retorna `device_mismatch`
5. Retorna el `session_token`

---

## Edge Function: `validate`

**Endpoint:** `/functions/v1/validate`

**Input:**
```json
{
  "session_token": "token-de-sesion",
  "device_id": "uuid-del-dispositivo"
}
```

**Output exitoso:**
```json
{
  "valid": true
}
```

**Output con error:**
```json
{
  "valid": false,
  "error": "invalid_token" | "revoked" | "device_mismatch"
}
```

### Lógica

1. Busca la licencia con el `session_token`
2. Verifica que existe
3. Verifica que no está revocada
4. Verifica que el `device_id` coincida
5. Actualiza `last_validated` con NOW()
6. Retorna `valid: true`

---

## Queries Útiles

### Generar un código nuevo
```sql
INSERT INTO licenses DEFAULT VALUES
RETURNING code;
```

### Generar múltiples códigos
```sql
INSERT INTO licenses DEFAULT VALUES
FROM generate_series(1, 10)
RETURNING code;
```

### Ver códigos activos
```sql
SELECT 
  code,
  device_id,
  activated_at,
  last_validated,
  revoked
FROM licenses
WHERE device_id IS NOT NULL
  AND revoked = FALSE
ORDER BY activated_at DESC;
```

### Ver códigos sin usar
```sql
SELECT code, created_at
FROM licenses
WHERE device_id IS NULL
  AND revoked = FALSE
ORDER BY created_at DESC;
```

### Revocar un código
```sql
UPDATE licenses
SET revoked = TRUE
WHERE code = 'MF-XXXX-XXXX-XXXX';
```

### Revocar todos los códigos de un dispositivo
```sql
UPDATE licenses
SET revoked = TRUE
WHERE device_id = 'uuid-del-dispositivo';
```

### Resetear un código (para reusar)
```sql
UPDATE licenses
SET 
  device_id = NULL,
  session_token = NULL,
  activated_at = NULL,
  last_validated = NULL,
  revoked = FALSE
WHERE code = 'MF-XXXX-XXXX-XXXX';
```

### Ver estadísticas
```sql
SELECT 
  COUNT(*) FILTER (WHERE device_id IS NULL) as sin_usar,
  COUNT(*) FILTER (WHERE device_id IS NOT NULL AND revoked = FALSE) as activos,
  COUNT(*) FILTER (WHERE revoked = TRUE) as revocados,
  COUNT(*) as total
FROM licenses;
```

### Eliminar códigos sin usar (más de 30 días)
```sql
DELETE FROM licenses
WHERE device_id IS NULL
  AND created_at < NOW() - INTERVAL '30 days';
```

---

## 🔒 Seguridad

- Los códigos se generan aleatoriamente (no predecibles)
- Un código = Un dispositivo (no se puede transferir)
- Los tokens de sesión son únicos y generados con UUID
- La validación requiere tanto el token como el device_id
- La revocación toma efecto en la próxima validación online

---

## 📈 Monitoreo

### Códigos activados hoy
```sql
SELECT COUNT(*)
FROM licenses
WHERE activated_at::date = CURRENT_DATE;
```

### Últimas validaciones
```sql
SELECT code, last_validated
FROM licenses
WHERE last_validated IS NOT NULL
ORDER BY last_validated DESC
LIMIT 10;
```

### Dispositivos únicos activos
```sql
SELECT COUNT(DISTINCT device_id)
FROM licenses
WHERE device_id IS NOT NULL
  AND revoked = FALSE;
```

---

**Recuerda:** Todas las PWAs que crees con esta plantilla usarán la misma tabla y funciones. No necesitas duplicar nada en Supabase.
