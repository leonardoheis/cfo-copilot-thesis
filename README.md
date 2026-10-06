# CFO Copilot - Overleaf / Mendeley

Este proyecto combina:

1. La estructura objetivo del documento Word de tesis suministrado.
2. El contenido de `Heis Leonardo TP5 vf1 19-02-25.docx`.
3. Los lineamientos de los PDF para **Conclusión** y **Trabajos futuros**.
4. Las referencias recuperadas directamente de los metadatos de citas Mendeley embebidos en el DOCX.

## Overleaf

Suba el ZIP como un proyecto nuevo y configure el compilador como **pdfLaTeX**. Overleaf ejecutará Biber para la bibliografía.

La bibliografía usa:

```latex
\usepackage[backend=biber,style=apa,sorting=nyt]{biblatex}
\addbibresource{references.bib}
```

Las citas fueron convertidas a comandos como:

```latex
\parencite{Jimenez2024}
```

## Mendeley

`references.bib` es la única fuente bibliográfica. Reúne las referencias generadas a partir de los metadatos Mendeley embebidos en el Word original, más las agregadas durante la redacción. Puede importarlo a Mendeley Desktop/Reference Manager.

Si en el futuro vuelve a exportar la biblioteca desde Mendeley, procure conservar las mismas **citation keys** usadas en `main.tex`; de lo contrario, deberá actualizar las claves de las citas en LaTeX.

## Conclusión y trabajos futuros

El Word fuente es un **Plan de Trabajo**, no un informe final con resultados experimentales. Por eso las secciones de Conclusión y Trabajos futuros se estructuraron según los lineamientos proporcionados, pero no se inventaron resultados ni métricas. Los comentarios `TODO` señalan los lugares donde deben incorporarse los resultados finales.
