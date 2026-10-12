# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + Canvas 2D. No hay `package.json`, bundler, transpilador, linter ni tests. Los textos de la UI y el README están en español; los identificadores y comentarios del código, en inglés.

## Ejecutar

Abrir `index.html` directamente o servir la carpeta con cualquier servidor estático:

    python -m http.server 8000   # luego http://localhost:8000

No hay suite de tests: la verificación es jugar en el navegador.

## Arquitectura (game.js)

- Un único script clásico (no módulo) con `'use strict'`; todo el estado es global y mutable, declarado en una sola línea `let board, current, next, score, ...`. `init()` lo reinicia todo y también es el handler del botón Reiniciar.
- **Modelo**: `board` es `ROWS × COLS`; cada celda es `0` o un índice 1–7. Ese mismo índice es el tipo de pieza, el valor dentro de su matriz en `PIECES` y la posición en `COLORS`. Añadir o cambiar una pieza implica tocar `PIECES`, `COLORS` y el `* 7` de `randomPiece()`.
- **Piezas**: matrices cuadradas; `rotateCW` transpone + invierte. `tryRotate` aplica wall kicks horizontales `[0, -1, 1, -2, 2]` (no es SRS).
- **Ciclo de pieza**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promueve `next` a `current` y llama a `endGame()` si colisiona al aparecer.
- **Bucle**: `loop` con `requestAnimationFrame` acumula `dt` en `dropAccum` y baja una fila al superar `dropInterval`. Pausa y fin de juego detienen el bucle con `cancelAnimationFrame(animId)`; reanudar llama a `loop()` directamente.
- **Render**: `draw()` repinta todo cada frame (grid → tablero → ghost con alpha 0.2 → pieza actual). `drawNext()` solo se llama desde `spawn()`. `drawBlock` se comparte entre ambos canvas recibiendo contexto y tamaño.
- **HUD**: `updateHUD()` se llama en `clearLines`, `softDrop` y al final de cada `keydown`; si añades una fuente de puntos fuera de esos caminos, llámalo.
- Nivel = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 − (level − 1) × 90)`.

## Acoplamientos con index.html

- `game.js` depende de los ids `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`.
- El tamaño de `<canvas id="board">` (300×600) debe ser `COLS × BLOCK` por `ROWS × BLOCK`; no se calcula en JS. `next-canvas` (120×120) asume una cuadrícula 4×4 de 30 px (`NB` en `drawNext`).
- La lista de controles del panel está escrita a mano en el HTML y en la tabla del README; mantenlas en sincronía con el `switch` del `keydown`.

## Comportamientos actuales a tener en cuenta

- Al reanudar la pausa (`togglePause`) no se vuelve a ocultar el overlay.
- Si el game over ocurre dentro de `loop` (caída por gravedad), `endGame()` cancela el frame en curso pero `loop` vuelve a pedir otro con `requestAnimationFrame` al final, así que el bucle sigue corriendo detrás del overlay.
