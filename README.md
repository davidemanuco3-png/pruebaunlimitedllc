# pruebaunlimitedllcCONCEPTO
Un carrusel de Instagram es una secuencia de imágenes cuadradas (idealmente 1080×1080px o 540×540px para preview). El sistema aquí genera cada imagen como una página HTML con 5 "slides" navegables, luego las captura como PNG usando Playwright headless.
Árbol de archivos requerido:
proyecto/
├── carousels/html/         ← los HTMLs van aquí
├── photos/                 ← imágenes PNG sin fondo (cutouts)
├── output/ciclo-png/       ← PNGs generados (se crea automático)
└── screenshot.py           ← script de captura
​
PARTE 1: ESTRUCTURA DEL HTML
Cada archivo HTML es un carrusel completo de 5 slides. El esqueleto base:
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: #111;
  gap: 16px;
}
.carousel { width: 540px; height: 540px; overflow: hidden; border-radius: 8px; }
.slides   { display: flex; transition: transform 0.4s ease; height: 100%; }
.slide    { min-width: 540px; height: 540px; position: relative; overflow: hidden; }
</style>
</head>
<body>

<div class="carousel">
  <div class="slides" id="slides">
    <div class="slide s1"><!-- Slide 1 --></div>
    <div class="slide s2"><!-- Slide 2 --></div>
    <div class="slide s3"><!-- Slide 3 --></div>
    <div class="slide s4"><!-- Slide 4 --></div>
    <div class="slide s5"><!-- Slide 5 --></div>
  </div>
</div>

<div class="nav">
  <button onclick="move(-1)">← Anterior</button>
  <span id="counter">1 / 5</span>
  <button onclick="move(1)">Siguiente →</button>
</div>

<script>
let cur = 0;
function move(d) {
  cur = Math.max(0, Math.min(4, cur + d));
  document.getElementById('slides').style.transform = 'translateX(-' + (cur * 540) + 'px)';
  document.getElementById('counter').textContent = (cur + 1) + ' / 5';
}
</script>
</body>
</html>
​
PARTE 2: CSS PARA INTEGRAR FOTOS SIN FONDO (PNG cutout)
Las fotos son PNG con fondo transparente. Se posicionan absolutas dentro del slide.
Regla crítica de CSS:
/* La foto */
.doc {
  position: absolute;
  bottom: 0;
  right: 0;         /* o left: 0 para layout invertido */
  height: 100%;     /* controla el tamaño — nunca usar width fijo */
  width: auto;      /* mantiene proporción */
  z-index: 1;
}

/* El contenido de texto */
.wrap {
  position: relative;
  z-index: 2;       /* siempre encima de la foto */
}
​
¿Por qué height y no width?
Porque los cutouts PNG tienen proporciones verticales. Si forzás width: 40% + height: 100% distorsiona o recorta. height: 100%; width: auto respeta la figura completa.
Referencia a la foto en el HTML:
<!-- CORRECTO: ruta relativa al archivo local -->
<img class="doc" src="../../photos/nombre-foto.png" alt="">

<!-- INCORRECTO: URL externa → CORS bloquea en file:// -->
<img src="https://raw.githubusercontent.com/.../foto.png">
​
PARTE 3: LOS 8 LAYOUTS CREATIVOS
Variar la composición en cada carrusel. Nunca repetir el mismo más de 3 veces en un ciclo.
Layout 1 — Right float (default)
.doc  { right: 0; bottom: 0; height: 100%; }
.wrap { padding: 44px; width: 54%; }
​
Texto izquierda, figura derecha.
Layout 2 — Left reversed
.doc          { left: 0; bottom: 0; height: 100%; }
.col-right    { position: absolute; right: 0; width: 52%; height: 100%;
                display: flex; flex-direction: column; justify-content: center;
                padding: 40px 44px; z-index: 2; }
​
Figura izquierda, texto derecha.
Layout 3 — Oversized bleed
.doc { right: -15px; bottom: 0; height: 115%; }
.wrap { width: 50%; padding: 44px; }
​
La figura sale por arriba del frame. Efecto editorial.
Layout 4 — Center pedestal
.doc {
  left: 50%;
  transform: translateX(-50%);
  bottom: 0;
  height: 86%;
  opacity: 0.85;
}
.top-text {
  position: absolute; top: 0; left: 0; width: 100%;
  text-align: center; padding: 36px 44px; z-index: 2;
}
​
Figura centrada, texto encima.
Layout 5 — Bottom anchored low
.doc      { right: 0; bottom: 0; height: 78%; }
.top-text { position: absolute; top: 0; left: 0; width: 100%; padding: 44px; z-index: 2; }
.bot-text { position: absolute; bottom: 26px; left: 44px; width: 52%; z-index: 2; }
​
Figura pequeña abajo, texto domina arriba.
Layout 6 — Ghost behind
.doc  { right: -20px; bottom: 0; height: 100%; opacity: 0.18; filter: grayscale(20%); }
.wrap { position: relative; z-index: 2; width: 100%; text-align: center; padding: 50px; }
​
Figura fantasma de fondo, texto encima centrado.
Layout 7 — Angled
.doc {
  bottom: -5px; right: -5px; height: 100%;
  transform: rotate(-3deg);
  transform-origin: bottom right;
}
​
Figura rotada ligeramente. Da energía y movimiento.
Layout 8 — Top block + bottom figure
.doc      { right: 0; bottom: 0; height: 82%; }
.top-block { position: absolute; top: 0; left: 0; width: 100%; padding: 44px; z-index: 2; }
.bot-left  { position: absolute; bottom: 30px; left: 44px; max-width: 54%; z-index: 2; }
​
Texto en bloque top, figura al fondo inferior.
PARTE 4: ESTRUCTURA DE LOS 5 SLIDES
Slide 1 — Hero (hook)
Fondo oscuro con color temático
Foto prominente (layout 1, 2, 3 o 7 preferentemente)
Headline en 2 líneas: primera en blanco, segunda en color acento
Subtítulo pequeño
Handle/firma en esquina inferior derecha — z-index: 10
<div class="slide s1" style="background: #0A1428;">
  <img class="doc" src="../../photos/foto.png" alt="">
  <div class="wrap">
    <div class="eyebrow">Categoría</div>
    <h1>Texto principal<br><em>con acento aquí.</em></h1>
    <p class="sub">Subtítulo breve explicando el tema.</p>
  </div>
  <div class="handle">@usuario</div>
</div>
​
Slides 2 y 3 — Contenido educativo
Fondo más claro o diferente al hero
Eyebrow label en caps + headline serif + cuerpo de texto
Sin foto O foto pequeña (layout 5 o 6)
Slide 4 — Dato de impacto
Número grande o estadística dominante
Fondo oscuro (navy) para contraste
Fuente/referencia pequeña en gris
Foto opcional en formato circle o pequeña
<div class="slide s4" style="background: #0A0F20;">
  <div class="center">
    <div class="eyebrow">El dato</div>
    <div class="big-number">75%</div>
    <div class="label">descripción del dato aquí</div>
    <p class="source">Fuente, Año</p>
  </div>
</div>
​
Slide 5 — CTA (call to action)
Foto de la persona en posición más pequeña (60-75% altura) o circle
Texto de acción claro
Credenciales al pie derecho
Handle visible
PARTE 5: ZONA PROHIBIDA
Esquina inferior izquierda ≈ 200px × 200px — no poner nada ahí.
Instagram superpone el sticker de re-compartir exactamente en esa zona cuando alguien comparte el carrusel en Stories. Si hay texto o elementos importantes ahí, quedan tapados.
┌─────────────────────────────────┐
│                                 │
│                                 │
│                                 │
│                                 │
│                       [HANDLE]  │  ← handle SIEMPRE derecha
│ [ZONA PROHIBIDA]                │
└─────────────────────────────────┘
   ↑ ~200px × 200px: vacío
​
PARTE 6: CONVERTIR HTML → PNG con Playwright
# screenshot.py
import asyncio
from playwright.async_api import async_playwright
from pathlib import Path

async def main():
    html_dir = Path("carousels/html")
    out_dir  = Path("output/ciclo-png")
    out_dir.mkdir(parents=True, exist_ok=True)

    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        page    = await browser.new_page(viewport={"width": 540, "height": 540})

        for f in sorted(html_dir.glob("*.html")):
            carousel_dir = out_dir / f.stem
            carousel_dir.mkdir(exist_ok=True)

            await page.goto(f.as_uri(), wait_until="networkidle")

            for slide_num in range(1, 6):
                out = carousel_dir / f"slide_{slide_num:02d}.png"
                await page.screenshot(path=str(out))
                if slide_num < 5:
                    await page.locator("button", has_text="Siguiente").click()
                    await page.wait_for_timeout(400)

            print(f"✓ {f.stem} — 5 slides")

        await browser.close()

asyncio.run(main())
​
Instalar y correr:
pip install playwright
playwright install chromium
python screenshot.py
​
Resultado: por cada HTML genera una carpeta con 5 PNGs:
output/ciclo-png/
  nombre_carrusel_01/
    slide_01.png
    slide_02.png
    slide_03.png
    slide_04.png
    slide_05.png
  nombre_carrusel_02/
    ...
​
CHECKLIST ANTES DE EXPORTAR

Foto usa ruta relativa, no URL externa

Esquina inferior izquierda vacía en todos los slides

Handle/firma en esquina inferior derecha con z-index: 10

Slide 1 tiene hook claro y foto prominente

Slide 5 tiene CTA y credenciales visibles

Botón "Siguiente →" existe en el HTML (lo necesita screenshot.py)

Cada carrusel usa un layout fotográfico diferente al anteriorvvvv
