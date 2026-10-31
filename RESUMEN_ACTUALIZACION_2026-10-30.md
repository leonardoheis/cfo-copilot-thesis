# Actualización de la tesis con los datos congelados del 30 de octubre de 2026

Este documento resume la ronda de actualización de `main.tex` y de `figuras/eda/` con las ejecuciones de NB00, NB01 y NB01b del 30 de octubre de 2026. El procedimiento es el de `docs/plans/thesis-update.md` del repositorio `CFO_Copilot`, cuya bitácora de rondas registra la misma lista de cambios.

Nada de esto está commiteado.

## 1. Qué cambió en los datos

| Cambio | Origen | Efecto en la tesis |
|---|---|---|
| Rendimiento por dividendo con el día de cierre del trimestre | Corrección O2 del scraper | El falso cero de 2006-Q2 desaparece; el registro de atípicos del dividendo queda equilibrado (15 subas, 14 caídas) |
| `gdp_yoy` pasa a crecimiento interanual (A191RO1Q156NBEA); `dxy` se llama `usd_broad_index` | Corrección O3 | El PBI gana correlación con varios sectores (ingresos de Energy: 0,51) |
| Honeywell 2025-Q3 retenido | Escisión y agrupamiento de acciones de 2026 (especificación `hon-separation-and-reverse-split.md`, H1) | Cuatro trimestres retenidos en lugar de tres; NB01b evalúa 58 empresas en las variables escaladas |

Datos congelados en `CFO_Copilot/data/backups/thesis_2026-10-30/` (96 archivos). Ejecuciones de W&B: NB00 `5qo4anfj`, NB01 `fk4v1k3k` a `160gjfh2`, NB01b `l3gnhk3v`, `q2pf4aoq`, `2tbrq4ec` y `4fqj30o4`.

## 2. Qué cambió en `main.tex`

**Scraper, problemas encontrados**
- Dos puntos nuevos: el rendimiento por dividendo sin el día de cierre, y la escisión y el agrupamiento de acciones de Honeywell (qué se retiene y qué queda pendiente).

**Resultados, NB00 y NB01**
- La fecha de ejecución es ahora el 30 de octubre.
- Cuatro trimestres retenidos, con Honeywell 2025-Q3; Honeywell entra en la lista de empresas con 80 trimestres.
- Faltantes: los ingresos, el resultado operativo, el resultado neto y el flujo de caja libre pasan de 0,2 % a 0,3 %, y los gastos operativos de 5,2 % a 5,3 %.
- Registro de atípicos: 796 entradas (antes 817) y 155 de gastos operativos (antes 157). Se reescribió la frase del dividendo. Entre las cuatro series más extremas, el EPS de Honeywell reemplaza al rendimiento por dividendo de Occidental; el salto se explica por la ganancia de desconsolidación y, en parte, por la base del agrupamiento.
- `tab:objetivo`: cinco porcentajes de disponibilidad bajan una décima, y una curtosis pasa de 203,7 a 203,6.
- Sensibilidad macro por sector: se agrega el PBI (Energy 0,51 en los ingresos y 0,46 en el resultado operativo y el EBITDA; 0,39 en el flujo de caja libre). Healthcare pasa a ser "el sector menos sensible" (0,21), ya no "insensible".
- `tab:diferenciacion`: el resultado operativo queda en 8/41/11 y el EBITDA en 8/42/10.
- `tab:export`: los conteos de filas con objetivo conocido bajan de 1 a 4 filas.
- Feature Engineering: ρ del flujo de caja libre −0,28 → −0,29. En `tab:grupos-res` cambian siete valores; el calendario del EBITDA (0,005) deja de estar en negrita y las banderas del resultado operativo pasan a 0,004. Se invirtió el orden de los márgenes (0,10 y 0,09).

**NB01b**
- 58 empresas (antes 59) en las variables escaladas, porque se suma Honeywell.
- Rango de mejora: 16 %–43 %.
- `tab:transformaciones` actualizada completa.
- En el texto: los valores de Anderson–Darling de los ingresos, la dependencia y Jarque–Bera del resultado operativo y el EBITDA, y la dependencia del flujo de caja libre (0,35).

**Decisiones**
- `tab:decisiones`: el EBITDA ya no excluye ningún grupo (antes "C"). Los valores p de Kruskal–Wallis pasan a 0,726, 0,636, 0,296 y 0,849.
- Pendientes: se agrega Honeywell. Son cuatro límites vigentes en lugar de dos.
- En la hipótesis de ablación se elimina "solo el calendario en el EBITDA".

**Apéndice, diccionario de datos**
- La fila de `gdp_yoy` describe ahora la nueva serie.
- `dxy` → `usd_broad_index`.

**Sin cambios, verificados contra las ejecuciones congeladas**
- La matriz de correlaciones (0,71, −0,30, −0,53; máximos 0,30, 0,20 y 0,08).
- La fuerza estacional.
- Los rankings de atributos en h1 y h4.
- La procedencia (77/12/11 %).
- Las banderas.
- Los siete faltantes sin motivo de Ford.

## 3. Figuras

Las 42 figuras de `figuras/eda/` se reemplazaron por las imágenes de W&B de esta ronda, con los mismos nombres; 39 cambiaron. Se revisaron a mano `registro_atipicos_series.png` y `sensibilidad_macro_sector_ebitda.png`, y coinciden con el texto.

## 4. Verificación

- `latexmk` compila sin errores: 93 páginas, sin referencias indefinidas. Los tres *overfull boxes* son de la bibliografía.
- No queda en `main.tex` ninguno de los valores anteriores de la lista de cambios.
- Los cambios propios del autor en `main.tex` (el `\ref` reparado y la lista de quiebres estructurales) se conservaron.

## 5. Qué implica para el modelado

Ninguna decisión de NB01 cambia, salvo una: el EBITDA ya no tiene grupos débiles. El calendario queda exactamente en el umbral (0,005), así que conviene incluirlo igualmente en la ablación de NB02. El defecto de base del EPS de Honeywell no llega a los modelos, porque el EPS no es un atributo.
