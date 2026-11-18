# Actualización de la tesis con el arnés de evaluación (NB02)

Este documento resume la ronda de actualización de `main.tex` con la ejecución de NB02 del 17 de noviembre de 2026. El procedimiento es el de `docs/plans/thesis-update.md` del repositorio `CFO_Copilot` (paso 17 de `docs/plans/nb02-evaluation-harness.md`), cuya bitácora de rondas registra la misma lista de cambios.

Nada de esto está commiteado.

## 1. Qué se ejecutó

No es una ronda de datos: NB02 usa el mismo congelamiento del 30 de octubre (`thesis_2026-10-30`), así que ningún número de NB00, NB01 o NB01b cambia.

| Dato | Valor |
|---|---|
| Ejecución | 17 de noviembre de 2026, kernel reiniciado |
| Código | `CFO_Copilot` en el commit `741affa` |
| Registros | 1 101 312 (4 referencias × 4 variables × h = 1 a 4 × Protocolo A y Protocolo B con 5 historias) |
| Reproducibilidad (A2) | dos ejecuciones completas, registros idénticos valor a valor |
| W&B | ocho corridas `nb02-baselines-{variable}-{A,B}`, del 17 de noviembre a las 15:07 |

Dos ejecuciones anteriores se descartaron antes de esta: en la primera, la deriva evaluada como modelo ignoraba la historia disponible en el Protocolo B; en la segunda, el kernel conservaba el código anterior a la corrección. El arnés ahora detiene la ejecución si el código cargado no coincide con el del disco.

## 2. Qué cambió en `main.tex`

**Introducción**
- Los modelos se comparan con cuatro pronósticos de referencia, no con uno; el ingenuo estacional se informa siempre.
- Los modelos fundacionales dejan de ser opcionales: TabPFN-TS, Chronos-2, TimesFM y el modelo tabular TabICLv2.

**Diseño experimental**
- Once notebooks, ya sin uno opcional.
- Tabla de notebooks: NB02 sin la mención a MAPIE; NB06 sin "opcional", con sus cuatro modelos y ejecutado localmente.
- NB00, NB01 y NB02 tienen criterio de salida; el capítulo describe NB00 a NB02.
- Subsección nueva, *Criterios para los modelos fundacionales*: las cuatro condiciones de entrada, por qué se excluyen TimeGPT y Moirai-2, y la ejecución local sin internet.

**Metodología, sección nueva: Arnés de evaluación (NB02)**
- Protocolo A (16 orígenes, desde 2022-Q2 para h = 1) y Protocolo B (dejando una empresa fuera, con historia H de 8, 12, 20, 40 trimestres o completa), y por qué no KFold ni GroupKFold.
- Escala común del error, las cuatro referencias y la regla de la referencia a superar.
- Métricas (MASE calculado en el arnés, R² solo fuera de muestra) y por qué no hay conjunto de validación.
- Intervalos: predicción conforme ponderada por empresa, ACI, *jackknife* en el Protocolo B, ponderación de credibilidad con los pares del sector, y por qué no MAPIE.
- Prueba de Diebold–Mariano por origen y controles de reproducibilidad.

**Resultados, sección nueva: NB02**
- Tabla de la referencia a superar por variable y horizonte, con la prueba de Diebold–Mariano frente a la segunda mejor.
- Tabla del error por sector (h = 1).
- Tabla de cobertura de los intervalos por protocolo e historia, y la Figura de cobertura de los ingresos.
- Figura del error según la historia disponible: la deriva necesita una historia larga.
- Implicaciones para NB03 a NB08.

**Otros**
- La introducción del capítulo de resultados incluye NB02; las hipótesis pasan a "NB03 a NB08".

## 3. Bibliografía y figuras

- `references.bib`: `buhlmann2005credibility`, `qu2026tabiclv2` (ICML 2026) y `erickson2025tabarena` (NeurIPS 2025), con autores y títulos verificados en arXiv.
- `figuras/nb02/` (carpeta nueva, agregada a `\graphicspath`): `error_por_historia_ingresos.png` y `cobertura_ingresos.png`, copiadas de las corridas de W&B de esta ejecución.

## 4. Verificación

- Compilación con `latexmk`: 106 páginas, sin referencias ni citas indefinidas; los tres cuadros desbordados son las URL de la bibliografía, como antes.
- Cada número de las tablas y del texto se tomó de las tablas de W&B de la ejecución `741affa`.
- Al revisar el texto se corrigieron tres afirmaciones: las diez celdas peores por sector no están todas al 80 % (una está al 90 %), la deriva con H = 8 es la peor referencia en los cuatro horizontes, y el MASE menor que 1 de los ingresos describe al ingenuo estacional. La especificación de NB02 tenía el mismo error de conteo por sector y se corrigió.

## 5. Qué implica para el modelado

- Cada modelo de NB03 a NB08 debe superar la referencia de la Tabla de referencias, y solo 4 de las 16 victorias de las referencias son significativas: en el resultado operativo y el EBITDA los márgenes entre referencias son mínimos.
- En los ingresos de Technology, el cambio de régimen de 2025 es la prueba de los atributos de crecimiento de NB04.
- NB04 decide, antes de evaluar cualquier modelo, si el Protocolo A escala cada empresa con una ventana reciente.
- NB06 debe superar al ingenuo estacional con H = 8, donde la deriva falla.

## 6. Pendiente

- Diebold (2015), Giacomini y White (2006) y Kim et al. (2020), citados en la nota de justificación de `CFO_Copilot`, no se citan en `main.tex`, por lo que no se agregaron a `references.bib`.
- La conclusión sigue sin resultados de modelos: se completa después de NB08.
