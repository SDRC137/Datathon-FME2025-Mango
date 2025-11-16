# Datathon-FME2025-Mango

# 🥭 Mango Demand Forecast – Datathon Hackathon Edition

Este repo recoge el proyecto que estamos haciendo para un **datathon de Mango**, donde el reto es básicamente responder a una pregunta muy sencilla de decir… y nada sencilla de resolver:

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

Por ahora, este repo es el punto de partida del proyecto: contiene el código y los experimentos para jugar con los datos y entender mejor el reto. Más adelante iremos documentando también las ideas de modelado, validación y truquitos que vayamos probando durante la hackathon. 😄
