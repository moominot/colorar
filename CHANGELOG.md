# Colorar — Registre de canvis

## Historial

### Favicon
- Afegit `favicon.ico` (16×16 i 32×32) amb icona de paleta de colors.
- Referència `<link rel="icon">` a `index.html`.

### Correcció desat/restauració PBN i edició de paleta
- **fillMap per regió connexa**: el fillMap ara desa un píxel representatiu per cada regió connexa pintada (índex de píxel → color hex), en lloc d'un entry per cluster de color. En restaurar, es fa flood-fill des de cada píxel desat, reconstruint exactament quines regions s'han pintat.
- **Edició de paleta**: `editPBNColor` ja no repintava regions buides; ara comprova `alpha > 0` abans d'actualitzar píxels.
- Bump de clau localStorage a `v4`.

### Mode "Zones de colorar" (reemplaça Blanc i negre)
- El mode BW ara usa el mateix algorisme k-means que PBN per segmentar la imatge en zones.
- Slider de **Resolució** (4–24 zones) i **Contorn** (Cap/Prim/Mitjà/Gruixut) al modal de configuració.
- L'usuari pinta lliurement amb la paleta normal (sense números, sense restricció de color).
- **Fons**: blanc si hi ha contorn visible; tons pastel si el contorn és "Cap".
- Contorns zoom-invariants (overlay canvas, igual que PBN).
- Desat/restauració en format compacte (labels + fillMap), compatible amb el nou sistema per regions.

### Edició de color de paleta PBN
- Clicar un mosaic PBN ja seleccionat obre el selector de color natiu del navegador.
- En triar un color nou, actualitza `pbnColors`, els píxels ja pintats d'aquella regió i el mosaic de la paleta en temps real.

### Contorn PBN zoom-invariant
- Els contorns PBN es dibuixen en un canvas d'overlay (`#borderOverlay`) a nivell de viewport, no al canvas principal escalat.
- `renderBorderOverlay()` mapeja cada píxel de pantalla a l'espai de la imatge i consulta `linesImageData`.
- El contorn sempre ocupa exactament 1px CSS independentment del zoom.
- Color gris semitransparent (Prim) → més fosc (Mitjà) → quasi negre (Gruixut).

### Slider de llindar blanc/negre en viu
- Slider de llindar al modal de configuració BW amb preview en temps real.
- Control en viu a la barra d'estat després de carregar.
- `bwSourceRaw` desa la ImageData original per recalcular sense recarregar el fitxer.

### Opció Blanc i botó Il·luminar (mode PBN)
- Checkbox **Blanc** a la barra d'estat PBN: mostra fons blanc en comptes de pastel per a les regions sense emplenar.
- Botó 💡 **Il·luminar**: ressalta temporalment (1,5 s) totes les zones del color seleccionat.

### Contorn PBN: reducció de gruixos
- **Prim**: hairline d'1px (només veí dret + inferior), molt subtil i gris.
- **Mitjà**: equivalent a l'anterior "Prim" (1px amb diagonals).
- **Gruixut**: equivalent a l'anterior "Mitjà" (~2px).

### Desat/restauració PBN robusta
- El mode PBN ja no desa `colorLayer` ni `linesImageData` com PNG (causava errors de color space i stack overflow).
- Desa `pbnFillMap`, `pbnLabels` (base64 amb loop), `pbnColors` i `pbnBorderThick`.
- En restaurar, reconstrueix `colorLayer` des del fillMap i `linesImageData` des de `rebuildPBNLines`.

### Botó ? (pipeta) — identificar color/número
- Botó `?` a la barra de paleta. En activar-lo, el clic al canvas mostra el color del píxel.
- En mode PBN, mostra el número de la regió (colorIdx+1) i el color corresponent.
- Tooltip flotant amb preview de color.

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

### Eliminació meta tag obsolet
- Eliminat `apple-mobile-web-app-capable` (deprecated). Cobert per `mobile-web-app-capable`.

### Desat i restauració de sessió PBN (versió inicial)
- Fix stack overflow: codificació de `pbnLabels` amb bucle en lloc de spread.
- `buildPBNStatsBar()` extret per reutilitzar a `onPBNDone` i `restoreSession`.

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
- **Línies de contorn d'1px**: canvi a detecció únicament dret+inferior.

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
