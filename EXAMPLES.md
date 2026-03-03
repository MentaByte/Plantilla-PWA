# 💡 Casos de Uso - Ejemplos

## 🎯 Ejemplos de apps que puedes construir con esta plantilla

---

## 📚 1. Catálogo Digital (como MercadoForzado)

**Descripción:** Catálogo de productos/servicios con acceso controlado

**Personalización:**
- Cambia el contenido en `/core/index.html`
- Agrega imágenes de productos a `/portadas` o `/assets`
- Lista las imágenes en el array `PRECACHE` del Service Worker

**Código de ejemplo:**
```html
<!-- En /core/index.html -->
<div class="catalogo">
  <div class="producto">
    <img src="../assets/producto1.jpg" alt="Producto 1">
    <h3>Producto 1</h3>
    <p>Descripción del producto</p>
  </div>
  <!-- Más productos... -->
</div>
```

---

## 📖 2. Curso Online Privado

**Descripción:** Plataforma de aprendizaje con lecciones exclusivas

**Personalización:**
- Organiza las lecciones por módulos
- Usa localStorage para guardar progreso
- Agrega videos (embedidos de YouTube/Vimeo para no cachearlos)

**Código de ejemplo:**
```javascript
// Guardar progreso
function marcarLeccionCompleta(leccionId) {
  const progreso = JSON.parse(localStorage.getItem('progreso') || '{}');
  progreso[leccionId] = true;
  localStorage.setItem('progreso', JSON.stringify(progreso));
}

// Verificar progreso
function esLeccionCompleta(leccionId) {
  const progreso = JSON.parse(localStorage.getItem('progreso') || '{}');
  return progreso[leccionId] === true;
}
```

---

## 🎮 3. App de Juegos Exclusiva

**Descripción:** Mini-juegos con acceso limitado

**Personalización:**
- Crea tus juegos en canvas o con bibliotecas como Phaser
- Guarda high scores en localStorage
- Cachea todos los assets del juego

**Código de ejemplo:**
```javascript
// High Score System
function guardarScore(score) {
  const mejorScore = localStorage.getItem('highscore') || 0;
  if (score > mejorScore) {
    localStorage.setItem('highscore', score);
    return true; // Nuevo récord
  }
  return false;
}

function obtenerHighScore() {
  return localStorage.getItem('highscore') || 0;
}
```

---

## 📋 4. Lista de Recursos Premium

**Descripción:** Biblioteca de templates, assets o recursos descargables

**Personalización:**
- Lista archivos con enlaces de descarga
- Agrega categorías y filtros
- Incluye preview de recursos

**Código de ejemplo:**
```html
<div class="recurso">
  <div class="preview">
    <img src="../previews/recurso1.png" alt="Preview">
  </div>
  <h3>Template de Email</h3>
  <button onclick="descargar('template-email.zip')">Descargar</button>
</div>

<script>
function descargar(archivo) {
  // Simular descarga o enlazar a archivo
  window.open(`../recursos/${archivo}`, '_blank');
}
</script>
```

---

## 🎵 5. Reproductor de Audio Premium

**Descripción:** Podcasts, audiolibros o música exclusiva

**Personalización:**
- Usa el elemento `<audio>` de HTML5
- Guarda el progreso de reproducción
- Cachea los archivos de audio

**Código de ejemplo:**
```javascript
// Guardar progreso de reproducción
const audio = document.getElementById('audio-player');

audio.addEventListener('timeupdate', () => {
  const progreso = {
    archivo: audio.src,
    tiempo: audio.currentTime
  };
  localStorage.setItem('ultimo-audio', JSON.stringify(progreso));
});

// Retomar reproducción
window.addEventListener('load', () => {
  const ultimo = JSON.parse(localStorage.getItem('ultimo-audio') || '{}');
  if (ultimo.archivo) {
    audio.src = ultimo.archivo;
    audio.currentTime = ultimo.tiempo || 0;
  }
});
```

---

## 🗺️ 6. Guía Turística Offline

**Descripción:** Mapa interactivo con puntos de interés

**Personalización:**
- Usa Leaflet.js para mapas offline
- Pre-cachea tiles del mapa
- Agrega información de cada lugar

**Código de ejemplo:**
```html
<div id="map" style="height: 400px;"></div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">

<script>
const map = L.map('map').setView([4.6097, -74.0817], 13);
L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

// Agregar marcadores
L.marker([4.6097, -74.0817])
  .addTo(map)
  .bindPopup('Punto de interés');
</script>
```

---

## 📊 7. Dashboard de Datos Privados

**Descripción:** Visualización de métricas o estadísticas

**Personalización:**
- Usa Chart.js o ApexCharts
- Carga datos desde JSON local o API
- Actualiza datos en tiempo real (si hay conexión)

**Código de ejemplo:**
```javascript
import Chart from 'chart.js/auto';

const ctx = document.getElementById('myChart');
new Chart(ctx, {
  type: 'bar',
  data: {
    labels: ['Ene', 'Feb', 'Mar', 'Abr', 'May', 'Jun'],
    datasets: [{
      label: 'Ventas 2024',
      data: [12, 19, 3, 5, 2, 3],
      backgroundColor: 'rgba(102, 126, 234, 0.5)'
    }]
  }
});
```

---

## 🎨 8. Portfolio Privado

**Descripción:** Galería de trabajos o proyectos exclusivos

**Personalización:**
- Grid de imágenes o videos
- Lightbox para ver en detalle
- Categorías y tags

**Código de ejemplo:**
```css
.galeria {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.galeria img {
  width: 100%;
  height: 250px;
  object-fit: cover;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.3s;
}

.galeria img:hover {
  transform: scale(1.05);
}
```

---

## 📝 9. Notas Privadas / Journal

**Descripción:** App de notas con cifrado básico

**Personalización:**
- Editor de texto enriquecido (TinyMCE, Quill)
- Guarda en localStorage
- Exporta a PDF o Markdown

**Código de ejemplo:**
```javascript
// Sistema de notas simple
function guardarNota(titulo, contenido) {
  const notas = JSON.parse(localStorage.getItem('notas') || '[]');
  notas.push({
    id: Date.now(),
    titulo,
    contenido,
    fecha: new Date().toISOString()
  });
  localStorage.setItem('notas', JSON.stringify(notas));
}

function obtenerNotas() {
  return JSON.parse(localStorage.getItem('notas') || '[]');
}

function eliminarNota(id) {
  const notas = JSON.parse(localStorage.getItem('notas') || '[]');
  const filtradas = notas.filter(n => n.id !== id);
  localStorage.setItem('notas', JSON.stringify(filtradas));
}
```

---

## 🎓 10. Flashcards Educativas

**Descripción:** Sistema de tarjetas de estudio con spaced repetition

**Personalización:**
- Crea mazos de tarjetas
- Sistema de progreso
- Algoritmo de repetición espaciada

**Código de ejemplo:**
```javascript
class Flashcard {
  constructor(pregunta, respuesta) {
    this.pregunta = pregunta;
    this.respuesta = respuesta;
    this.nivel = 0; // 0-5, aumenta con respuestas correctas
    this.ultimaRevision = null;
  }
  
  respuestaCorrecta() {
    this.nivel = Math.min(this.nivel + 1, 5);
    this.ultimaRevision = new Date();
  }
  
  respuestaIncorrecta() {
    this.nivel = Math.max(this.nivel - 1, 0);
    this.ultimaRevision = new Date();
  }
  
  proximaRevision() {
    const dias = [0, 1, 3, 7, 14, 30];
    const diasEspera = dias[this.nivel];
    const proxima = new Date(this.ultimaRevision);
    proxima.setDate(proxima.getDate() + diasEspera);
    return proxima;
  }
}
```

---

## 🛠️ Tips para Personalizar

### 1. Cambia los colores
```css
:root {
  --color-primario: #667eea;
  --color-secundario: #764ba2;
  --color-fondo: #f8f9fa;
  --color-texto: #333;
}
```

### 2. Agrega fuentes personalizadas
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">

<style>
  body {
    font-family: 'Inter', sans-serif;
  }
</style>
```

### 3. Agrega iconos (Lucide, FontAwesome, etc.)
```html
<script src="https://unpkg.com/lucide@latest"></script>
<i data-lucide="home"></i>

<script>
  lucide.createIcons();
</script>
```

### 4. Usa librerías útiles
```html
<!-- Lodash para manipular datos -->
<script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>

<!-- Day.js para fechas -->
<script src="https://cdn.jsdelivr.net/npm/dayjs@1.11.9/dayjs.min.js"></script>

<!-- Axios para HTTP -->
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

---

## 🚀 Siguiente Nivel

¿Quieres agregar funcionalidades más avanzadas?

- **Sincronización en la nube:** Usa Supabase Storage o Realtime
- **Notificaciones Push:** Agrega Push API
- **Compartir contenido:** Usa Web Share API
- **Tomar fotos:** Accede a la cámara con getUserMedia
- **Geolocalización:** Usa Geolocation API

---

¡Las posibilidades son infinitas! 🎉
