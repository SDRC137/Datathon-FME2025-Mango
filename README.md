# Datathon-FME2025-Mango

# 🥭 Mango Demand Forecast – Datathon Hackathon Edition

Este repo recoge el proyecto que estamos haciendo para el reto de Mango, donde el reto es básicamente responder a una pregunta muy sencilla de decir… y nada sencilla de resolver:

> ¿Cuántas prendas deberíamos producir la próxima temporada para no quedarnos cortos ni ahogarnos en stock?

La gracia del problema es que se parece muchísimo a lo que hace Mango en la vida real:

- Tenemos **datos históricos de varias temporadas** (Spring–Summer y Fall–Winter).
- Cada producto viene con:
  - **Embeddings de imagen** (la foto de la prenda convertida a números).
  - **Familia y categoría de producto** (vestidos, pantalones, abrigos, etc.).
  - **Atributos de la prenda** (mangas, cuello, silueta, tejido…).
  - Info de negocio como:
    - Número de **tiendas** donde se vende.
    - Número de **tallas** disponibles.
    - **Ciclo de vida** (cuándo entra y cuándo sale a la venta).
- Y además:
  - **Ventas semanales reales**.
  - **Producción histórica** (lo que Mango pidió al proveedor).
  - Una estimación interna de la **demanda “real”** (qué se habría vendido con stock infinito).

El objetivo del challenge es predecir, para cada producto de la nueva temporada, una cantidad llamada `Production`, que en la práctica es:

> *“Cuánto deberíamos encargar al proveedor para cubrir la demanda sin pasarnos demasiado.”*

La evaluación no es el típico RMSE de Kaggle, sino una métrica más cercana al negocio:

- Se calcula a partir del ratio **VAR (Ventas Antes de Rebajas / Producción)**.
- **Quedarse corto** (no producir suficiente y perder ventas) se penaliza más que **producir de más** (sobrará stock, pero algo se puede recolocar/vender luego).
- El resultado es un score de **0 a 100**, y nuestra misión es llevarlo lo más arriba posible.

En resumen:

- Es un problema de **forecasting de demanda**,
- con mezcla de **datos tabulares + visión (embeddings)**,
- y una métrica **asimétrica**, muy centrada en el impacto real de negocio.

---

## 🔧 Proceso Implementado

### 1. Preprocesamiento de Datos

#### Limpieza inicial
- **Eliminación de columnas con >40% de valores faltantes**: Identificamos y eliminamos columnas poco informativas que podrían introducir ruido en el modelo.
- **Eliminación de filas con valores críticos faltantes**: Removemos productos sin `Production` o `image_embedding`, ya que son esenciales para el entrenamiento.

#### Agregación por ID
- Agrupamos las ventas semanales por producto (`ID`) para obtener métricas agregadas:
  - `total_demand`: Suma de la demanda semanal
  - `total_sales`: Suma de las ventas semanales
  - `sell_through`: Ratio de ventas sobre producción (indicador de stockout)
  - `is_stockout`: Flag binario para productos con sell-through ≥ 0.98

#### Feature engineering
- **Variables temporales**: Extracción de mes y semana del año desde `phase_in`.
- **One-Hot Encoding**: Codificación de todas las variables categóricas (familia, categoría, fabric, color, silhouette, etc.).
- **Escalado de variables numéricas**: Normalización con `StandardScaler` de las features numéricas continuas (excluyendo variables binarias one-hot encoded).
- **Limpieza de nombres**: Eliminación de caracteres especiales en nombres de columnas para compatibilidad con LightGBM.

### 2. Modelo: LightGBM

Utilizamos **LightGBM Regressor** con los siguientes parámetros:

```python
lgbm_params = {
    'objective': 'regression_l1',  # MAE como función de pérdida
    'metric': 'mae',
    'n_estimators': 115,  # Número óptimo de árboles
    'learning_rate': 0.05,
    'n_jobs': -1,
    'random_state': 42,
    'verbose': -1
}
```

**Características del modelo:**
- Entrenamiento con el 100% de los datos disponibles
- Función de pérdida: MAE (Mean Absolute Error)
- One-hot encoding de categóricas (no usamos el manejo nativo de categóricas de LightGBM)
- Features totales: ~200+ (incluyendo one-hot encoded)

### 3. Generación de Submissions

Generamos múltiples archivos de submission aplicando diferentes **factores de riesgo** a las predicciones base:
- `intento_1`: factor 30x
- `intento_2`: factor 20x
- `intento_3`: factor 25x

Esto nos permite explorar diferentes niveles de conservadurismo en la producción.

---

## 📊 Resultados

**Score obtenido: 32.69%**

Este score refleja el desempeño en la métrica VAR (Ventas Antes de Rebajas / Producción) evaluada por la competición.

---

## 🔍 Posibles Mejoras

### Sobreentrenamiento detectado
Creemos que el modelo está **sobreajustando los datos de entrenamiento**. Posibles acciones:

1. **Regularización más agresiva**: 
   - Aumentar `min_child_samples`
   - Reducir `max_depth`
   - Aumentar `reg_alpha` y `reg_beta`

2. **Validación cruzada temporal**: 
   - Implementar un esquema de validación que respete el orden temporal de las temporadas
   - Usar TimeSeriesSplit para evaluar el modelo de forma más robusta

3. **Feature selection**: 
   - Analizar importancia de features y eliminar las menos relevantes
   - Reducir la dimensionalidad del one-hot encoding agrupando categorías poco frecuentes

4. **Early stopping**: 
   - Implementar early stopping con un validation set separado
   - Monitorear la curva de aprendizaje para detectar overfitting

5. **Ensemble con otros modelos**: 
   - Probar XGBoost, CatBoost o modelos lineales
   - Combinar predicciones mediante stacking o blending