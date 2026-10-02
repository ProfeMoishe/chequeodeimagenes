# PNG → BMP · 16×16 / 25×25

Herramienta web para comprobar las dimensiones de imágenes PNG y convertirlas a BMP en tamaños **16×16** y/o **25×25**.  
Todo el procesamiento ocurre en el navegador: no se sube nada a ningún servidor.

Demo: `https://tu-usuario.github.io/tu-repo/`

---

## Tabla de contenidos

- [Características](#características)
- [Cómo usarlo](#cómo-usarlo)
- [Despliegue en GitHub Pages](#despliegue-en-github-pages)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Cómo funciona internamente](#cómo-funciona-internamente)
- [Personalización](#personalización)
- [Limitaciones](#limitaciones)
- [Solución de problemas](#solución-de-problemas)
- [Contribuciones](#contribuciones)
- [Licencia](#licencia)

---

## Características

- Carga de PNG por:
  - Selector de archivos.
  - Arrastrar y soltar.
  - Pegado desde el portapapeles (`Ctrl+V`).
- Validación automática de dimensiones:
  - **16×16**
  - **25×25**
- Aviso visual si la imagen no tiene uno de esos tamaños.
- Previsualizaciones en vivo para 16×16 y 25×25.
- Redimensionado con opción de suavizado.
- Exportación a BMP:
  - **24 bits**: fondo blanco, máxima compatibilidad.
  - **32 bits**: conserva transparencia (alfa), según visor.
- Descarga individual o de ambas medidas.
- Diseño responsive y táctil, cómodo en móvil.
- Sin dependencias, sin frameworks, sin servidor.
- Listo para GitHub Pages.

---

## Cómo usarlo

1. Abre `index.html` en un navegador moderno o visita la URL desplegada.
2. Carga una imagen PNG:
   - Haz clic en la zona de carga.
   - Arrastra y suelta el archivo.
   - Pega con `Ctrl+V`.
3. Revisa el tamaño original y el aviso:
   - Si mide **16×16** o **25×25**, se guardará sin redimensionar.
   - Si no, se generarán las versiones 16×16 y 25×25.
4. Ajusta las opciones:
   - **Suavizar al redimensionar**: activado para escalados suaves, desactivado para estilo pixel art.
   - **BMP con transparencia (32 bits)**: desactivado para BMP de 24 bits con fondo blanco.
5. Pulsa:
   - **Guardar BMP 16×16**
   - **Guardar BMP 25×25**
   - **Descargar ambas**
6. Usa **Cargar otra imagen** para reiniciar.

---

## Despliegue en GitHub Pages

1. Crea un repositorio en GitHub.
2. Sube el archivo `index.html` a la raíz del repositorio.
3. Opcionalmente añade `README.md` y `LICENSE`.
4. Ve a **Settings → Pages**.
5. En **Source**, selecciona:
   - **Deploy from a branch**
   - Branch: `main`
   - Folder: `/ (root)`
6. Guarda los cambios.
7. Espera unos segundos. La URL será:

   ```
   https://tu-usuario.github.io/tu-repo/
   ```

No se necesita build ni configuración adicional.

---

## Estructura del proyecto

```
.
├── index.html      # Aplicación completa (HTML + CSS + JS)
├── README.md       # Este archivo
└── LICENSE         # Opcional, por ejemplo MIT
```

---

## Cómo funciona internamente

### 1. Carga de archivos

Se usan:

- `File API` para leer el archivo.
- `URL.createObjectURL()` para crear una URL temporal.
- `new Image()` para decodificar la imagen.
- `img.onload` para obtener `naturalWidth` y `naturalHeight`.

También se soportan:

- `dragover`, `dragleave`, `drop` para arrastrar y soltar.
- `paste` para pegar imágenes desde el portapapeles.

### 2. Validación

```js
const es16 = (src.w === 16 && src.h === 16);
const es25 = (src.w === 25 && src.h === 25);
const valido = es16 || es25;
```

Si no coincide, se muestra una advertencia y se ofrecen los redimensionados.

### 3. Previsualización

Se usan `<canvas>`:

- Miniatura de la imagen original.
- Lienzo de 16×16.
- Lienzo de 25×25.

Para que los píxeles se vean nítidos en imágenes pequeñas:

```css
canvas {
  image-rendering: pixelated;
  image-rendering: crisp-edges;
}
```

Y en JavaScript:

```js
ctx.imageSmoothingEnabled = opcionSuavizar;
```

### 4. Codificación BMP

El BMP se genera desde cero con `ArrayBuffer` y `DataView`.

Estructura escrita:

- **BITMAPFILEHEADER** (14 bytes)
  - Firma `BM`
  - Tamaño total
  - Offset a los datos de píxel
- **BITMAPINFOHEADER** (40 bytes)
  - Ancho, alto
  - Planos
  - Bits por píxel (24 o 32)
  - Compresión `BI_RGB`
  - Tamaño de imagen
  - Resolución
- **Datos de píxel**
  - Formato BGR o BGRA.
  - Orden de abajo hacia arriba.
  - Cada fila alineada a múltiplos de 4 bytes.

Para 24 bits, la transparencia se compone sobre fondo blanco:

```js
const af = a / 255;
u8[p++] = Math.round(b * af + 255 * (1 - af));
u8[p++] = Math.round(g * af + 255 * (1 - af));
u8[p++] = Math.round(r * af + 255 * (1 - af));
```

### 5. Descarga

Se crea un `Blob`:

```js
const blob = new Blob([buffer], { type: 'image/bmp' });
```

Y se descarga con:

```js
const a = document.createElement('a');
a.href = URL.createObjectURL(blob);
a.download = nombre + '_16x16.bmp';
a.click();
```

---

## Personalización

### Cambiar los tamaños permitidos

Actualmente los tamaños son 16 y 25. Para cambiarlos:

1. Modifica el array en JavaScript:

   ```js
   const SIZES = [16, 25];
   ```

2. Actualiza los textos y botones del HTML:

   ```html
   <h2>16 × 16</h2>
   <h2>25 × 25</h2>
   ```

3. Ajusta los `id` de los canvas y botones si añades más tamaños.

### Cambiar colores

Los colores principales están en variables CSS:

```css
:root {
  --bg: #0f1117;
  --card: #171a23;
  --acc: #5b8cff;
  --ok: #37d399;
  --warn: #ffb020;
  --err: #ff6b6b;
}
```

### Añadir más formatos de salida

El codificador actual solo genera BMP. Para añadir PNG o ICO:

- **PNG**: usa `canvas.toBlob(callback, 'image/png')`.
- **ICO**: requiere construir una cabecera ICO y empaquetar BMP o PNG.

---

## Limitaciones

- El BMP de 32 bits conserva el canal alfa, pero **no todos los visores lo respetan**. Para máxima compatibilidad usa 24 bits.
- Navegadores muy antiguos sin `Canvas` o `File API` no son compatibles.
- No hay procesamiento por lotes: se trabaja con una imagen a la vez.
- El redimensionado es de alta calidad, pero no reemplaza a un editor profesional para pixel art complejo.

---

## Solución de problemas

### La imagen no se carga

- Verifica que el archivo sea una imagen válida.
- Prueba con PNG, JPG o WebP.
- Revisa la consola del navegador por errores.

### No se descarga el BMP

- Comprueba que el navegador no esté bloqueando descargas.
- Prueba en otro navegador.
- Asegúrate de que la imagen se haya cargado correctamente.

### El resultado se ve borroso

- Desactiva **“Suavizar al redimensionar”**.
- Usa imágenes de origen con buena definición.
- Para pixel art, trabaja con tamaños exactos o múltiplos.

### La transparencia no se ve en el BMP

- Algunos visores ignoran el canal alfa en BMP.
- Usa la opción de 24 bits con fondo blanco si necesitas compatibilidad total.

---

## Contribuciones

Las contribuciones son bienvenidas.

1. Haz un fork del repositorio.
2. Crea una rama:

   ```bash
   git checkout -b feature/nueva-funcion
   ```

3. Realiza tus cambios.
4. Haz commit:

   ```bash
   git commit -m "Añade nueva función"
   ```

5. Sube la rama:

   ```bash
   git push origin feature/nueva-funcion
   ```

6. Abre un Pull Request.

---

## Licencia

Este proyecto se distribuye bajo la licencia **MIT**.  
Puedes usarlo, modificarlo y distribuirlo libremente, manteniendo el aviso de copyright.

---

## Créditos

- Inspirado en la necesidad de preparar iconos BMP de 16×16 y 25×25 para interfaces retro, dashboards o sistemas embebidos.
- Construido con HTML, CSS y JavaScript vanilla.
