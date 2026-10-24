# Modelo predictivo para priorizar controles preventivos de ACV

**A quién citar primero cuando los turnos no alcanzan para todos**

Proyecto Final Integrador del **Diplomado en Ciencia de Datos y Análisis Avanzado**, desarrollado por el **Grupo 16**:

- Emanuel Aguirre
- María Alejandra Arocena
- Juan Pablo Luna
- Horacio Javier Ortiz

Octubre de 2026

## El problema

Un programa de prevención de accidente cerebrovascular (ACV) no puede citar a control a toda la población adulta, porque los turnos, los profesionales y los estudios son limitados. Hoy se elige a quién citar con una regla simple: **toda persona de 60 años o más, o con hipertensión**. Es fácil de aplicar, pero poco eficiente: en nuestros datos cita a un tercio de los adultos, y de cada 100 personas citadas solo unas 13 tuvieron un ACV.

## La solución

Desarrollamos un modelo que asigna a cada persona un **puntaje de riesgo** a partir de datos clínicos básicos y ordena a la población de mayor a menor riesgo. Con ese orden, el programa puede citar primero a quienes más lo necesitan y ajustar la cantidad de citaciones a los turnos disponibles.

Probamos **Regresión Logística, Random Forest y XGBoost**, cada uno en una versión base y otra optimizada. Elegimos la **Regresión Logística optimizada** porque obtuvo el mejor rendimiento, no se sobreajusta, es la más rápida de entrenar entre las optimizadas y permite explicar por qué una persona queda priorizada.

El modelo es una herramienta de **priorización, no de diagnóstico**.

## Resultados principales

Los resultados se midieron sobre el conjunto de prueba: 851 adultos que el modelo no vio durante el entrenamiento, de los cuales 49 tuvieron un ACV.

| Indicador | Meta | Resultado | Estado |
|---|---|---|---|
| PR-AUC (capacidad de ordenar el riesgo) | Superar al azar (0,058) | 0,229 (IC 95 %: 0,145 a 0,343) | Cumplido |
| KPI de focalización | Capturar al menos el 70 % de los casos citando como máximo al 40 % de la población de mayor riesgo | 85,7 % (71,4 % con el 30 %) | Cumplido |
| KPI operativo | Detectar al menos el 60 % de los casos con el punto de corte elegido | 57,1 % (IC 95 %: 42,9 % a 70,6 %) | No cumplido |

Comparado con la regla tradicional:

- **Con el mismo esfuerzo, el modelo detecta los mismos casos.** La regla ya usa las dos señales más fuertes (edad e hipertensión), así que el aporte del modelo está en la eficiencia y la flexibilidad.
- **Para detectar prácticamente los mismos casos, el modelo cita a 3,6 puntos porcentuales menos de la población.** La diferencia es estadísticamente significativa según un bootstrap pareado de 2.000 réplicas.
- **El KPI operativo no se cumplió** porque el punto de corte se fijó para lograr exactamente un 60 % en entrenamiento, sin margen de seguridad.

Proyectado a 10.000 adultos, el modelo permite elegir entre dos estrategias:
- Igualar prácticamente los casos detectados por la regla con unas **365 citaciones menos**.
- Reducir las citaciones un **36 %** (1.175 menos), aceptando detectar 94 casos menos.

Los factores con un efecto claro en el modelo son la **edad** (odds ratio 3,90), la **hipertensión** (1,89) y la **glucosa promedio** (1,25).

## Datos

- **Fuente:** [Stroke Prediction Dataset](https://www.kaggle.com/datasets/fedesoriano/stroke-prediction-dataset), publicado en Kaggle por fedesoriano.
- **Tamaño original:** 5.110 registros y 12 columnas.
- **Población analizada:** 4.253 adultos (18 años o más), de los cuales 247 tuvieron un ACV (5,8 %).
- **Variables:** edad, hipertensión, enfermedad cardíaca, glucosa promedio, IMC, tabaquismo, género, estado civil, tipo de trabajo y tipo de residencia.

El dataset **no se incluye en el repositorio**. La notebook lo descarga automáticamente desde Kaggle con la librería `kagglehub`.

## Metodología

El proyecto sigue la metodología **CRISP-DM**. Las decisiones principales fueron:

- **Limpieza:** se eliminó el identificador de paciente y un único registro con género "Other", y se limitó el análisis a adultos.
- **IMC faltante (4,3 %):** se completó con la mediana por grupo de edad y género, calculada solo con los datos de entrenamiento.
- **Tabaquismo desconocido (20,3 %):** se mantuvo como una categoría propia.
- **Separación de datos:** 80 % para entrenamiento y 20 % para prueba, estratificada. El conjunto de prueba se usó una sola vez, al final.
- **Preprocesamiento:** pipeline sin fuga de información, con imputación del IMC, escalado de variables numéricas y one-hot encoding de categóricas.
- **Desbalance de clases:** ponderación de clases (aprendizaje sensible al costo) en lugar de técnicas de remuestreo como SMOTE.
- **Evaluación:** validación cruzada anidada 5 × 3, con PR-AUC como métrica principal.

La notebook está organizada en las siguientes secciones:

| Sección | Contenido |
|---|---|
| 0 | Configuración, semilla fija (`SEED = 42`) e importación de librerías |
| 1 | Carga, limpieza, filtrado de adultos y auditoría de calidad de datos |
| 2 | Análisis exploratorio y pruebas de hipótesis (riesgos relativos, chi-cuadrado, t de Welch y Mann-Whitney) |
| 3 | Separación entrenamiento / prueba |
| 4 | Benchmark de negocio: regla "edad ≥ 60 o hipertensión" |
| 5 | Pipeline de preprocesamiento |
| 6 | Modelado con validación cruzada anidada, selección del modelo y elección de puntos de corte |
| 7 | Evaluación final en el conjunto de prueba, bootstrap pareado y verificación de KPIs |
| 8 | Interpretabilidad (importancia por permutación, odds ratios y curva lift) |
| 9 | Proyección a 10.000 personas y análisis de equidad por género y residencia |
| 10 | Conclusiones, limitaciones y próximos pasos |

## Estructura del repositorio

```
.
├── Grupo16_ACV_20.10.ipynb              # Notebook con todo el análisis
├── Grupo16_Informe_Final_ACV.pdf        # Informe final
├── requirements.txt                     # Librerías y versiones utilizadas
└── README.md
```

## Cómo ejecutar el proyecto

### Opción 1: Google Colab (recomendada)

1. Abrir [Google Colab](https://colab.research.google.com/), ir a **Archivo > Abrir notebook > GitHub**, pegar la URL de este repositorio y elegir `Grupo16_ACV_20.10.ipynb`.
2. Ejecutar **Entorno de ejecución > Reiniciar y ejecutar todo**.

No hace falta instalar nada: la primera celda instala `kagglehub` y descarga el dataset.

### Opción 2: Ejecución local

Requiere Python 3.10 o superior.

```bash
git clone [URL de este repositorio]
cd [nombre de la carpeta del repositorio]
python -m venv venv
source venv/bin/activate        # En Windows: venv\Scripts\activate
pip install -r requirements.txt
pip install jupyter
jupyter notebook Grupo16_ACV_20.10.ipynb
```

Después, en Jupyter, ejecutar **Kernel > Restart & Run All**.

La descarga del dataset requiere conexión a internet. Si `kagglehub` pide credenciales, hay que generar un token de API en la cuenta de Kaggle (**Settings > API > Create New Token**) y guardar el archivo `kaggle.json` en `~/.kaggle/`. En Windows, la carpeta es `C:\Users\<usuario>\.kaggle\`.

### Reproducibilidad

Todas las particiones, búsquedas de hiperparámetros y remuestreos usan la semilla fija `SEED = 42`. Si se ejecuta la notebook de principio a fin, las métricas son idénticas a las del informe. Lo único que puede variar son los tiempos de cómputo.

En Colab, la validación cruzada anidada tarda unos 3 minutos; la parte más lenta es el Random Forest optimizado. A eso se suma el bootstrap de los odds ratios, que reentrena el modelo 1.000 veces.

## Librerías utilizadas

| Librería | Versión | Uso |
|---|---|---|
| pandas | 2.2.3 | Manipulación de datos |
| numpy | 2.1.3 | Cálculo numérico |
| scipy | 1.16.3 | Pruebas estadísticas |
| scikit-learn | 1.6.1 | Preprocesamiento, Regresión Logística, Random Forest, validación cruzada y métricas |
| xgboost | 3.4.1 | Modelo XGBoost |
| matplotlib | 3.10.0 | Visualizaciones |
| seaborn | 0.13.2 | Visualizaciones |
| kagglehub | 1.0.2 | Descarga del dataset |

## Limitaciones

- **Pocos casos de ACV** (247 en total y 49 en la prueba final), por lo que los resultados tienen márgenes de error amplios.
- **Datos transversales:** el dataset indica si la persona ya tuvo un ACV, no si lo tendrá. Para usar el modelo como predictor de riesgo futuro haría falta validarlo con datos de seguimiento.
- **Prevalencia más alta que la de la población general:** antes de aplicarlo en otra población habría que recalibrar el modelo y revisar el punto de corte.
- **Ventaja moderada sobre la regla:** el modelo mejora la eficiencia, pero no reemplaza el criterio clínico.

## Consideraciones éticas

Se trabajó con datos públicos y anónimos. Una implementación real debería cumplir la Ley 25.326 de Protección de Datos Personales y la Ley 26.529 de Derechos del Paciente en su Relación con los Profesionales e Instituciones de la Salud. También debería mantener supervisión humana: el modelo nunca debe usarse para negar o postergar la atención de una persona con síntomas. Además, conviene monitorear periódicamente que funcione igual para distintos grupos de género, edad y lugar de residencia.

## Próximos pasos

1. Elegir el punto de corte según los turnos disponibles, con un margen de seguridad sobre la meta de detección.
2. Validar el modelo con datos locales antes de usarlo.
3. Recalibrar el puntaje según la proporción real de casos de la población donde se aplique.
4. Sumar información clínica que hoy no está en los datos, como valores de presión arterial, antecedentes familiares y colesterol.
5. Hacer una prueba piloto supervisada en un programa de prevención.
