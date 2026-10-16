# Correcciones de datos y EDA de cuatro variables: qué se hizo y qué implica

Este documento resume, paso a paso, el trabajo realizado entre el 7 y el 9 de octubre de 2026 en el repositorio `CFO_Copilot` y en esta tesis. Explica qué problema había, cómo se resolvió, cómo se verificó y qué implica para la etapa de modelado.

Los detalles técnicos completos están en el repositorio de código:

- Especificación (qué debe cumplirse y con qué evidencia): `docs/specs/ingestion-defects-cop-ge.md`
- Plan (pasos de implementación): `docs/plans/ingestion-defects-cop-ge.md`
- Marco del EDA: `docs/specs/eda-actionable-framework.md`

Nada de esto está commiteado todavía en ninguno de los dos repositorios.

---

## 1. Punto de partida

La primera ejecución de NB01 (2 de octubre) marcó cuatro series como **errores de datos**, no como eventos económicos:

| Serie | Síntoma |
|---|---|
| Margen bruto de UnitedHealth (UNH) | Salta de ≈0,25 a ≈0,98 en 2008 |
| Gastos operativos de Union Pacific (UNP) | Valores de −5 000 intercalados en 2007–2008 |
| Gastos operativos de SLB | Meseta de ≈4 500 en 2016–2018, contra ≈400 habitual |
| Ingresos de Pfizer (PFE) | Caídas a casi cero en 2018–2019 |

Además había dos defectos conocidos que obligaban a **excluir a ConocoPhillips (COP)** del análisis (ingresos de −81 500 millones de USD en 2010-T4) y a **bloquear los gastos operativos** como variable objetivo (General Electric tenía 12 trimestres de gastos operativos negativos).

Por último, NB01 analizaba **solo los ingresos**, aunque el plan maestro define cuatro variables a pronosticar: ingresos, resultado operativo, EBITDA y flujo de caja libre.

## 2. Método de trabajo

Cada corrección siguió el mismo orden:

1. **Especificación primero:** se documentó cada defecto con números verificados y contraejemplos, y se definieron criterios de aceptación (A1 a A11).
2. **Plan:** pasos concretos, archivos a tocar y pruebas a escribir.
3. **Implementación con pruebas:** cada regla nueva tiene una prueba automática que falla con el código anterior.
4. **Re-ingesta de las 60 empresas** y comparación celda por celda contra una copia previa del panel: solo podían cambiar los valores previstos.
5. **Ejecución de NB00 y NB01** (las ejecuta el autor, porque registran resultados en Weights & Biases).

Las respuestas de Alpha Vantage están guardadas en disco (`data/raw/alpha_vantage/`), por lo que las re-ingestas no consumen cuota de la API; solo vuelven a descargar los datos de la SEC.

## 3. Correcciones de ingesta, una por una

Cada regla lleva el número que tiene en la especificación.

### I1. Trimestres derivados con bases contables distintas (COP, PFE, IBM, OXY, KMB, MRK, GE)

**Problema.** La SEC publica a veces solo el total anual o el acumulado del año; el cuarto trimestre se calcula como *año − nueve meses*. Si el total anual fue **reexpresado** (por una escisión, por ejemplo) y los trimestres no, la resta mezcla dos empresas distintas. Así salían los −81 500 de COP (escisión de Phillips 66) y las caídas de Pfizer.

**Solución.** Cada trimestre derivado se calcula dos veces: con las cifras tal como se informaron por primera vez y con las últimas reexpresadas.

- Si ambas coinciden (tolerancia del 1 %), se usa ese valor.
- Si no coinciden, se elige la más cercana al trimestre anterior.
- Si no hay trimestre anterior, la celda queda vacía (nunca se inventa un valor).

**Resultado.** COP 2010-T4 pasa a 51 726 (entre 47 208 y 56 530); PFE 2018-T4 y 2019-T4 vuelven a 13 977 y 12 688. La regla corrigió además defectos que nadie había listado: IBM 2019–2020 (lo que NB01 había leído como "escisión de Kyndryl" era un error de datos), OXY 2012–2024, KMB 2022–2023, MRK 2019 y GE 2013.

**Límite conocido (decisión D7).** En un pico estacional la regla puede elegir mal: los ingresos de Alphabet en 2012-T4 quedan en 13 215 en lugar de 14 419. Se probó una variante estacional que corregía Alphabet, pero empeoraba otros casos, así que se descartó y el defecto quedó documentado.

### I7. Costo de ventas informado en partes (SLB, VZ, GE)

Algunas empresas informan el costo de productos y el de servicios por separado, sin un total. El scraper tomaba solo una parte y por eso la ganancia bruta (y los gastos operativos que dependen de ella) salían mal. **Solución:** si no hay total, se suman las partes.

### I8. Empresas sin costo de ventas (UNH y UNP) — decisiones D3 y D4

UnitedHealth es una aseguradora y Union Pacific un ferrocarril: ninguna informa un "costo de lo vendido". Reconstruirlo daría una cifra distinta bajo el mismo nombre de columna. **Solución:** se declara en `config/companies.yaml` que no tienen costo de ventas; su ganancia bruta, margen bruto y gastos operativos quedan vacíos en todos los trimestres, con el motivo registrado en el libro de faltantes de NB00.

### I9. Gastos operativos de General Electric no confiables — decisión D6

GE fue un conglomerado con un banco propio (GE Capital) y luego se dividió en tres empresas. Sus gastos operativos no pueden reconstruirse a partir de la SEC sin inventar cifras. **Solución:** solo sus gastos operativos quedan vacíos; GE conserva el resto de sus variables, y los gastos operativos vuelven a poder usarse como variable objetivo para las demás 59 empresas.

### Riesgo asumido: Alpha Vantage completa celdas (decisión D5)

El panel toma primero la SEC y deja que Alpha Vantage complete solo las celdas vacías. Para la ganancia bruta esto afecta 819 trimestres de 22 empresas desde 2009. Alpha Vantage no publica cómo calcula sus cifras, por lo que en una misma serie pueden convivir dos definiciones. Se decidió **asumir el riesgo** (dejar esas celdas vacías eliminaría la ganancia bruta de cinco empresas) y documentarlo en la especificación, el plan y la tesis.

### I10. Un trimestre que en realidad es el año completo (ORCL)

Oracle etiquetó en cinco de sus 10-K el total del año fiscal como si fuera el cuarto trimestre (por ejemplo, 39 383 en lugar de 11 014). **Solución:** si un "trimestre" coincide (0,1 %) con un total anual que termina el mismo día, en cualquier etiqueta del mismo concepto, se descarta y se deriva como año − nueve meses. Un barrido de las 60 empresas encontró el patrón solo en Oracle.

### I11. Costo de ventas informado con signo negativo (CAT, BA)

Caterpillar (2008-T2) y Boeing (2009-T4) presentaron el costo de ventas con signo negativo, lo que daba márgenes brutos imposibles (CAT 1,74). **Solución:** el costo de ventas se lee por su magnitud y cada corrección queda registrada en el log. CAT pasa a un margen de 0,26 y BA a 0,19.

### I12. EBITDA de Alpha Vantage con otra definición — decisión D8

**Problema.** La SEC define el EBITDA del panel como *resultado operativo + depreciación y amortización*. Alpha Vantage lo calcula sobre su propio EBIT, que en algunos trimestres está roto: Kimberly-Clark tenía un EBITDA de −10 806 en 2006-T2 y Union Pacific ≈8 900 en los cuartos trimestres de 2006 y 2007 (contra ≈1 100 en el resto). En total, 1 060 celdas de 42 empresas diferían en más de un 1 % de la definición de la SEC.

**Solución.** El EBITDA de Alpha Vantage se recalcula como su resultado operativo más la depreciación y amortización de su estado de resultados o, si falta, de su estado de flujos de efectivo. Si no hay ninguna de las dos, la celda queda vacía.

**Resultado.** Cambiaron 1 398 celdas de EBITDA en 53 empresas y **ninguna otra columna**. KMB 2006-T2 = 782,7; UNP 2006-T4 = 1 125 y 2007-T4 = 1 201. El EBITDA de Costco en 2026-T2, que estaba vacío, se completó (3 412). Quedan vacíos seis trimestres de Ford (2006-T2 a 2008-T1), en los que ninguna fuente informa depreciación.

### Quiebres estructurales: COP y AT&T — decisión D9

- **COP 2011-T2:** la escisión de Phillips 66 cerró en 2012, pero las reexpresiones trasladan el cambio de base del panel a 2011-T2.
- **AT&T 2007-T1:** la fusión con BellSouth (cerrada el 29/12/2006) lleva los ingresos de 15 891 a 28 969 de un trimestre a otro.

Ahora hay 9 quiebres × 4 trimestres = 36 filas marcadas. Otras adquisiciones grandes (Pfizer–Wyeth, Merck–Schering-Plough, Comcast–NBCUniversal, Ecolab–Nalco, Duke–Progress y Medtronic–Covidien) **no** se registraron por decisión explícita; quedan como pendiente.

## 4. Verificación final (paso 8, 9 de octubre)

Sobre el panel reconstruido por NB00 se comprobaron todos los criterios de aceptación:

| Criterio | Resultado |
|---|---|
| A1 COP | Sin ingresos nulos ni negativos; 2010-T4 = 51 726 |
| A2 GE | Gastos operativos vacíos en los 81 trimestres, con motivo `unreliable_opex` |
| A4 LMT | Sus gastos operativos negativos siguen siendo correctos (cumplen la identidad contable en el 98 % de los trimestres) |
| A6 PFE | 13 977 y 12 688, los valores correctos |
| A7/A9 UNP y UNH | Sin ganancia bruta, margen ni gastos operativos; ingresos y resultados idénticos a antes |
| A11 EBITDA | Solo cambió el EBITDA; KMB y UNP correctos; 36 filas de quiebre |

Con eso se **levantó la exclusión de COP** y el **bloqueo de los gastos operativos**: ahora se analizan las 60 empresas.

## 5. NB01 ahora analiza las cuatro variables objetivo

- Las secciones de objetivo, segmentos, comportamiento temporal, Feature Engineering y decisiones se repiten para **ingresos, resultado operativo, EBITDA y flujo de caja libre**.
- Los ingresos usan transformaciones logarítmicas. Las otras tres pueden ser negativas, así que usan el **cambio interanual dividido por los ingresos de los últimos cuatro trimestres**.
- Se corrigió el análisis macro, que fallaba con variables negativas.
- NB01 exporta un dataset por variable y horizonte (16 en total) y una tabla de decisiones por variable.

### Resultados principales

| Variable | Transformación | Grupos débiles (ablación en NB02) | Un solo modelo | Pronosticable |
|---|---|---|---|---|
| Ingresos | Residuo estacional ingenuo | Ninguno | Sí (p = 0,727) | 53 de 60 (88 %) |
| Resultado operativo | Cambio escalado por ingresos | Banderas, calendario | Sí (p = 0,636) | 51 de 60 (85 %) |
| EBITDA | Cambio escalado por ingresos | Banderas, calendario | Sí (p = 0,236) | 54 de 60 (90 %) |
| Flujo de caja libre | Cambio escalado por ingresos | Tasa real, banderas, calendario | Sí (p = 0,890) | 57 de 60 (95 %) |

## 6. Qué implica para la tesis y para el modelado

1. **Los datos son más confiables.** Las cuatro series que NB01 había marcado como errores se corrigieron. Ahora las series más extremas del registro son, en su mayoría, eventos reales: el desplome de Occidental en 2020, los dividendos extraordinarios de Costco y el deterioro de SLB en 2019.
2. **El universo completo vuelve al análisis.** COP entra y los gastos operativos pueden usarse como objetivo.
3. **Las cuatro variables son pronosticables** y todas admiten un único modelo para todas las empresas.
4. **La historia propia es la mejor señal**, pero se comporta distinto según la variable. El crecimiento de los ingresos tiende a revertir a su media; los cambios del resultado operativo y del EBITDA persisten un trimestre antes de revertir.
5. **Las variables macro aportan poco al flujo de caja libre**, y en los ingresos su señal está sobrestimada (funcionan como una etiqueta de fecha). Su valor real se medirá fuera de muestra.
6. **El resultado operativo y el EBITDA son casi la misma serie** (correlación de 0,956 tras unificar la definición del EBITDA). Conviene evaluar si hace falta modelarlos por separado.
7. **Las colas son muy pesadas** en las cuatro variables (curtosis de 12 a 204): hacen falta pérdidas y métricas robustas.

## 7. Cambios en la tesis (`main.tex`)

- **Problemas del scraper:** el punto 6 (trimestres con bases distintas) dejó de estar "pendiente" y describe la solución; hay un punto nuevo sobre el EBITDA de Alpha Vantage.
- **Quiebres estructurales:** se agregaron COP 2011-T2 y AT&T 2007-T1 (nueve quiebres, 36 filas); antes el texto decía siete quiebres y 28 filas.
- **Metodología:** la variable objetivo pasa a ser cuatro variables, con la fórmula del cambio escalado por ingresos.
- **Capítulo de resultados:** se reescribió completo con los resultados de las cuatro variables y las 60 empresas. Las 14 figuras se regeneraron con los datos corregidos.
- **Apéndice:**
  - La tabla de empresas ahora incluye razón social y actividad principal, y explica qué es un ticker.
  - El diccionario de datos agrega `is_public` e indica qué columnas son variables objetivo y cuáles quedan vacías por decisión.
  - Hay una tabla nueva con las 35 columnas de los datasets de atributos.
- **Compilación:** la tesis compila sin errores ni referencias rotas (77 páginas).

Las cifras de NB01 de la tesis provienen de una ejecución del notebook con el mismo código y los mismos datos, sin registrar en Weights & Biases; la ejecución oficial de NB01 debe dar los mismos números.

## 8. Cambios de herramientas

- `uv run poe check` ya no ejecuta `nbstripout`, de modo que no borra las salidas de los notebooks. `nbstripout` sigue corriendo al hacer `git commit`, así que los notebooks se commitean limpios.
- `end-of-file-fixer` ya no revisa `data/reports/`: ydata-profiling genera esos HTML sin salto de línea final en cada ejecución de NB01.

## 9. Pendientes

1. **Depreciación de Abbott:** Abbott informa depreciación (≈370 por trimestre) y amortización de intangibles (≈420) por separado, y el panel toma solo la primera, así que su EBITDA de la SEC omite la amortización. La solución es sumar las partes, como en I7, previo barrido de las 60 empresas.
2. **Candidatos a defecto en el registro de ingresos:** Abbott 2012-T4, Bristol-Myers Squibb 2008-T4 y Occidental 2025-T4.
3. **Saltos del EPS en el origen del pronóstico** (2026-T2): Alphabet, Honeywell y Amazon.
4. **Otras adquisiciones como quiebres estructurales**, y si el quiebre de Abbott debe pasar de 2013-T1 a 2012-T1.
5. **Alphabet 2012-T4**, el límite conocido de la regla I1.
6. **Actualizar el texto de NB01** con los números de la ejecución oficial y verificar el criterio A10 (el registro de atípicos ya no lista los defectos corregidos).

## 10. Próximos pasos del autor

1. Ejecutar **NB01** con el kernel reiniciado (*Restart Kernel* → *Run All*). Va a generar una corrida de W&B por variable y los archivos `features_*` de las cuatro variables.
2. Revisar que los números coincidan con los de la tesis; de lo contrario, avisar para corregirlos.
3. Revisar los cambios en ambos repositorios y decidir cuándo commitearlos.

## 11. NB01b: comparación de transformaciones del objetivo

NB01b (ejecutado el 16 de octubre de 2026) resuelve la cuestión O1 del plan maestro: compara, para las cuatro variables, la transformación actual con sus alternativas según la dependencia del nivel y la normalidad del residuo STL.

- **Cada variable conserva su transformación:** logaritmo para los ingresos y escalado por ingresos para las otras tres. Ninguna candidata supera a la base en más de la mitad de las empresas; la que más se acerca es Box-Cox con el λ del sector en los ingresos (38 %, 42 % con Anderson-Darling).
- **Ninguna transformación vuelve normal el residuo** en ninguna empresa: las colas pesadas vienen de eventos, no de la escala, así que se mantienen las pérdidas robustas.
- **Excepciones por sector** para verificar en NB05: Box-Cox en los ingresos de Utilities, Energy y Technology; el escalado por ingresos es muy superior en el flujo de caja libre de Communication Services.

Cambios en la tesis: una subsección de metodología (*Comparación de transformaciones del objetivo (NB01b)*), una sección de resultados con una tabla y una figura de cuatro paneles (`figuras/eda/transformaciones_*.png`, tomadas de las corridas de W&B), y cuatro referencias nuevas (Yeo-Johnson, seno hiperbólico inverso, Jarque-Bera, Anderson-Darling).
