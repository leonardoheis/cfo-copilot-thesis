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

Ninguna decisión de NB01 cambia, salvo una: el EBITDA ya no tiene grupos débiles. El calendario queda exactamente en el umbral (0,005), así que conviene incluirlo igualmente en la ablación de NB04 (antes NB02; ver la sección 6). El defecto de base del EPS de Honeywell no llega a los modelos, porque el EPS no es un atributo.

## 6. Métricas, modelos y notebooks: una sola versión (31 de octubre)

La tesis y el plan maestro (`CFO_Copilot/docs/CFO_COPILOT_MASTER_PLAN.md`) describían métricas y modelos distintos. Ahora dicen lo mismo; en el plan maestro quedó registrado como las decisiones D25 (métricas) y D26 (modelos).

**Métricas.** Se mantienen las cuatro que prometía la tesis y se agrega MASE:

| Métrica | Dónde se usa | Por qué |
|---|---|---|
| RMSE | Las cuatro variables | Penaliza los errores grandes |
| MAE | Las cuatro variables | Error típico, robusto |
| MASE | Las cuatro variables | Sin escala; compara empresas de tamaños distintos (menor que 1 = mejor que el ingenuo estacional) |
| MAPE | Solo los ingresos | En el resultado operativo y el flujo de caja libre, el ingenuo estacional da un MAPE medio de 71 % y 221 %, por trimestres cercanos a cero |
| R² fuera de muestra | Las cuatro variables, contra la referencia a superar | El R² sobre niveles da 0,98 para el ingenuo estacional en los ingresos (mide el tamaño de las empresas, no la calidad del pronóstico); sobre la variación escalada da cero por construcción. Fórmula: 1 − SSE(modelo) / SSE(referencia), de Campbell y Thompson (2008) |

Los intervalos se evalúan con cobertura, ancho medio y puntaje de Winkler, y la significancia con Diebold–Mariano con la corrección de Harvey–Leybourne–Newbold. Todas las métricas se calculan en NB02, nunca en los notebooks de modelos.

**Modelos.** El núcleo son los compromisos de la tesis, ahora concretos:

- ARIMA/SARIMA(X) y ETS, con StatsForecast (NB03);
- regresión lineal regularizada (Ridge/ElasticNet) y LightGBM (NB04);
- LSTM (NB05);
- AutoML con AutoGluon-TimeSeries (NB07). Reemplaza a PyCaret, cuyo módulo de series de tiempo ajusta un modelo por empresa y no puede entrenar sobre las 60 juntas;
- el ensamble final (NB08);
- intervalos conformales con MAPIE, donde su interfaz respeta la regla de calibración sin mirar el futuro.

Son opcionales CatBoost, NHITS, TFT y los modelos fundacionales (TabPFN-TS, Chronos-2, NB06).

**Notebooks.** La numeración sigue la del plan maestro: NB02 arnés de evaluación, NB03 estadísticos, NB04 aprendizaje automático sobre atributos, NB05 redes neuronales, NB06 modelos fundacionales (opcional), NB07 AutoML, NB08 síntesis y ensamble, NB09 registro y predicción, NB10 preguntas con GenAI.

**Cambios en `main.tex`:**

- Introducción: la lista de modelos y AutoGluon-TimeSeries en lugar de PyCaret/StatForecasting.
- Sección de modelos y métricas: AutoGluon con el motivo del cambio; un párrafo sobre LightGBM, el ensamble final, la referencia ingenua estacional y los modelos opcionales; la lista de cinco métricas con su justificación; los intervalos y la prueba de Diebold–Mariano.
- `tab:notebooks`: reemplazada por la numeración nueva (once notebooks, uno opcional).
- Referencias a notebooks corregidas: la validación pasa a NB02, la ablación a NB04, la etapa de modelado a "NB04 y NB08", y "NB02 a NB05" a "NB02 a NB08".
- Siglas nuevas: MAE, MAPE, MASE y RMSE.

**Cambios en `references.bib`:** se agregan `campbell2008predicting` y `taquet2022mapie`.

**Verificación:** `latexmk` compila sin errores, 95 páginas, sin referencias ni citas indefinidas.

En `CFO_Copilot` se actualizaron en la misma línea el plan maestro y la especificación y el plan de NB02 (`docs/specs/nb02-evaluation-harness.md`, `docs/plans/nb02-evaluation-harness.md`). La copia de `CFO_COPILOT_MASTER_PLAN.md` en este repositorio es una versión anterior y no se tocó.
