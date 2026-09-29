# Calibración Cromática y Constancia de Color Computacional

Este repositorio contiene una solución en Python para la **calibración cromática y corrección de iluminantes** basada en imágenes de una carta de color (Macbeth ColorChecker de 24 parches) capturada bajo 7 condiciones de iluminación distintas.

El objetivo del proyecto es simular la **constancia de color computacional**, permitiendo que un sistema de visión por computador reconozca los colores reales de una escena independientemente del espectro de la fuente de luz.

---

## 📌 Características Principales

* **Carga y Conversión Espacial:** Conversión automática del espacio de color BGR nativo de OpenCV a RGB.
* **Rectificación Geométrica por Homografía:** Ajuste de perspectiva de la paleta inclinada a un mapa bidimensional estandarizado de $400 \times 600$ píxeles (`cv2.getPerspectiveTransform` y `cv2.warpPerspective`).
* **Muestreo Robusto de Parches:** Extracción de regiones centrales de $16 \times 16$ píxeles para evitar ruido y artefactos en los bordes.
* **Modelo Cromático Lineal $3 \times 3$:** Estimación de la matriz de corrección $M_{3 \times 3}$ resolviendo un sistema de ecuaciones sobredeterminado ($O \cdot M = G$) mediante Mínimos Cuadrados Ordinarios (OLS) y Descomposición en Valores Singulares (SVD).
* **Procesamiento Vectorizado:** Aplicación rápida de la transformación sobre millones de píxeles mediante producto punto matricial (`np.dot`).
* **Evaluación Cuantitativa (MAE):** Medición objetiva del error absoluto medio en niveles de intensidad RGB ($0$ a $255$).

---

## 🛠️ Estructura del Pipeline

```text
[Imagen Original BGR]
          │
          ▼
 [Conversión a RGB] ──► [Recorte ROI & Esquinas]
          │
          ▼
    [Homografía (400x600 px)]
          │
          ▼
 [Muestreo de 24 Parches (O)]
          │
          ▼
[Ground Truth (G)] ──► [OLS: np.linalg.lstsq]
          │
          ▼
     [Matriz M_spec (3x3)]
          │
          ▼
 [Inferencia: apply_color_correction]
          │
          ▼
   [Evaluación MAE & Mejora %]

```
---

## 📑 Fundamento Matemático

El sistema resuelve la optimización:

$$\min_M \| O \cdot M - G \|_F^2$$

Donde $O_{24 \times 3}$ es la matriz de colores observados, $G_{24 \times 3}$ es el Ground Truth teórico y $M_{3 \times 3}$ es la matriz de transformación estimada algebraicamente como:

$$M = (O^T O)^{-1} O^T G$$

---

## 📊 Resultados Obtenidos

El algoritmo demostró una alta efectividad para neutralizar cast cromáticos severos, recuperando tonos neutros y tonos de piel con alta precisión.

| Métrica | Valor Promedio |
| :--- | :--- |
| **MAE Inicial (Sin corregir)** | ~22.87 unidades RGB |
| **MAE Corregido (Con matriz $M$)** | ~13.79 unidades RGB |
| **Eficiencia de Corrección Global** | **~39.7% de reducción del error** |
