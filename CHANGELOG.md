# Colorar — Registre de canvis

## En curs

### Contorn PBN: nous gruixos
- **Prim**: hairline d'1px (només veí dret + inferior), molt subtil
- **Mitjà**: equivalent a l'anterior "Prim" (1px amb diagonals)
- **Gruixut**: equivalent a l'anterior "Mitjà" (~2px)

---

## Historial

### Desat/restauració PBN robusta
- El mode PBN ja no desa `colorLayer` ni `linesImageData` com PNG (causava errors de color space i stack overflow).
- Ara desa `pbnFillMap` (`{label → "#rrggbb"}`), `pbnLabels` (base64 amb loop), `pbnColors` i `pbnBorderThick`.
- En restaurar, reconstrueix `colorLayer` des del fillMap amb colors exactes i `linesImageData` des de `rebuildPBNLines`.
- Bump de clau localStorage a `v3` per netejar sessions antigues incompatibles.

### Botó ? (pipeta) — identificar color/número
- Botó `?` a la barra de paleta. En activar-lo, el clic al canvas mostra el color del píxel.
- En mode PBN, mostra el número de la regió (colorIdx+1) i el color corresponent llegit de `pbnLabels`.
- Tooltip flotant amb preview de color. Es cancel·la en seleccionar altra eina o clicar fora.

### Control de gruix de contorn PBN en viu
- Slider de gruix al modal de configuració PBN (Cap / Prim / Mitjà / Gruixut).
- Control en viu a la barra d'estat PBN sense necessitat de reprocessar.
- `rebuildPBNLines(thickness)` recalcula les línies des de `pbnLabels` sense el worker.

### Botó ✨ Resoldre (mode PBN)
- Omple automàticament totes les zones amb el seu color correcte.
- Acció reversible amb ↩ Desfer.

### Menú hamburguesa i botó d'informació (mòbil)
- Mòbil (≤540px): la barra d'eines s'amaga i apareix una fila compacta 📂 ☰ ℹ️.
- ☰ obre un panell desplegable amb tots els controls (desfer/refer/esborrar/desar, zoom, tolerància).
- ℹ️ obre un modal amb instruccions d'ús i l'estat de la imatge actual.
- Escriptori: barra completa amb ℹ️ afegit al final.
- Zoom i tolerància sincronitzats entre controls d'escriptori i mòbil.

### Eliminació meta tag obsolet
- Eliminat `apple-mobile-web-app-capable` (deprecated). Cobert per `mobile-web-app-capable`.

### Desat i restauració de sessió PBN (versió inicial)
- Fix stack overflow: codificació de `pbnLabels` amb bucle en lloc de spread.
- `buildPBNStatsBar()` extret per reutilitzar a `onPBNDone` i `restoreSession`.
- Restore reconstrueix els botons Resoldre i el control de gruix en viu.

### Fix slider PBN i control de gruix
- CSS `.pbn-config` amb grid layout per garantir que slider i valor es vegin sencers.
- Slider de gruix de contorn al modal (Cap/Prim/Mitjà/Gruixut).

### Fix desat i restauració PBN (sessió incompleta)
- Correcció de `restoreSession` per mode PBN: ara reconstrueix correctament la paleta, la barra d'estat i el primer color seleccionat.

### Optimització Web Worker per PBN
- Tot el processament k-means++ es fa en un Web Worker inline (Blob URL).
- Float32Array per a centroides, 20k mostres màxim, 20 iteracions.
- Transferable buffers per a labels i lines per evitar còpies.
- `mergeSmallRegions()`: absorbeix regions <30px al veí dominant.
- Barra de progrés durant el processament.

### Correccions mode PBN
- **Punts negres**: `mergeSmallRegions()` elimina regions aïllades petites.
- **Mismatch de color**: `Math.round()` en convertir pbnColors a hex.
- **Omplir region incorrecta**: `selectedPBNIdx` i validació de label abans d'omplir.
- **Línies de contorn d'1px**: canvi a detecció únicament dret+inferior (en lloc dels 4 veïns).

### PWA i optimització mòbil/tauleta
- Service worker per a ús offline.
- Web manifest per a instal·lació com a app.
- Suport a safe-area-inset (notch iOS/Android).
- Touch events: 1 dit = pan, 2 dits = pinch-zoom, tap = omplir.
- Desambiguació tap vs. pan amb llindar de 8px.
- CSS `overscroll-behavior: none` per evitar rubber-band iOS.

### Joc de colorar com a PWA independent
- Adaptat de `moominot.github.io/colorar.html` com a repositori independent.
- Modes: Blanc i negre, Imatge original, Pintar per números.
- Paleta de colors, color personalitzat, goma d'esborrar.
- Zoom (roda + botons + pinch), pan (Alt+arrossega / 1 dit).
- Flood fill amb tolerància ajustable.
- Undo/redo, desat automàtic a localStorage, exportació PNG.
- Enganxar imatge des del portaretalls (Ctrl+V).
- Drag & drop de fitxers.
