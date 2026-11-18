# Proyecto Final: Análisis de E-Commerce y Logística (Olist Brasil)

Proyecto de análisis exploratorio de datos (EDA), estadística y cuadro de mando sobre el marketplace brasileño **Olist**. El objetivo es analizar cómo afectan los tiempos de entrega, los envíos interestatales y las operaciones logísticas al volumen de ventas y a la satisfacción final del cliente (puntuación de las reseñas de 1 a 5 estrellas).

---

## Estructura del proyecto

El proyecto está organizado en las siguientes carpetas:

```text
proyecto_final/
├── data/
│   ├── raw/                                           # Datos originales en CSV (8 tablas)
│   │   ├── olist_orders_dataset.csv                   # Cabecera de pedidos y fechas
│   │   ├── olist_order_items_dataset.csv              # Detalle de artículos, precios y fletes
│   │   ├── olist_customers_dataset.csv                # Datos de clientes (ciudad, estado)
│   │   ├── olist_order_payments_dataset.csv           # Métodos de pago y cuotas
│   │   ├── olist_order_reviews_dataset.csv            # Puntuaciones (1-5) y comentarios
│   │   ├── olist_products_dataset.csv                 # Medidas, peso y categorías
│   │   ├── olist_sellers_dataset.csv                  # Ubicación de vendedores
│   │   └── product_category_name_translation.csv      # Traducción de categorías
│   └── processed/
│       ├── olist_raw_merged.csv                       # Dataset tras el cruce (112.650 filas x 38 cols)
│       └── olist_cleaned_merged.csv                   # Dataset final limpio y con nuevas variables (112.650 filas x 55 cols)
├── notebooks/
│   ├── 01_carga_y_cruce_de_datos.ipynb                # Cruce de las 8 tablas y control de cardinalidad
│   ├── 02_limpieza_y_transformacion.ipynb             # Parseo de fechas, nulos y cálculo de variables
│   └── 03_analisis_exploratorio_y_estadistica.ipynb   # Análisis exploratorio, contrastes de hipótesis y RFM
├── dashboard/
│   └── dashboard_ejecutivo_olist.xlsx                 # Cuadro de mando ejecutivo interactivo en Excel (Datos / Analisis / Dashboard)
├── reports/
│   └── informe_ejecutivo.md                           # Informe con conclusiones y recomendaciones
├── requirements.txt
└── README.md
```

---

## Pasos seguidos en el proyecto

### 1. Carga e integración de los datos (`01_carga_y_cruce_de_datos.ipynb`)
- Descargué las 8 tablas de la base de datos pública de Olist.
- Para evitar multiplicar filas artificialmente al unir las tablas (producto cartesiano), me di cuenta de que un pedido puede tener varios pagos (por ejemplo, parte con tarjeta y parte con vale) o varias reseñas. Por eso, antes de unir:
  - **Pagos**: Agrupé a nivel de pedido (`order_id`) sumando el importe total, calculando el número máximo de cuotas y quedándome con el método de pago principal.
  - **Reseñas**: Dedupliqué las opiniones por pedido quedándome con la más reciente.
- Tomé como tabla base los artículos vendidos (`order_items`, con 112.650 registros) y fui haciendo `merge` con pedidos, clientes, productos, vendedores, pagos y reseñas.
- El cruce dio exactamente **112.650 filas y 38 columnas**, sin perder ningún registro de compra.

### 2. Limpieza y preparación de los datos (`02_limpieza_y_transformacion.ipynb`)
Durante la revisión inicial detecté varios aspectos a corregir:
- **Fechas**: Había 7 columnas de fechas que venían como texto (`object`). Las convertí a tipo `datetime` (`order_purchase_timestamp`, `order_delivered_customer_date`, etc.).
- **Tratamiento de nulos**:
  - `product_category_name_english`: Traduje los nombres con la tabla de traducción y a los casos sin categoría les asigné `'unknown'`.
  - Peso y medidas del producto (`weight`, `length`, `height`, `width`): En lugar de meter una media general, imputé la mediana de su correspondiente categoría de producto.
  - Comentarios de reseñas: Creé indicadores binarios para saber si el cliente escribió comentario o solo dejó la puntuación.
- **Nuevas variables calculadas (Feature Engineering)**:
  - `delivery_days`: días reales desde la compra hasta la entrega en casa del cliente.
  - `estimated_delivery_days`: días previstos que prometía la plataforma.
  - `delivery_delay_days`: diferencia entre la entrega real y la fecha prometida (positivo indica retraso, negativo entrega anticipada).
  - `is_delayed`: indicador binario (1 si el pedido llegó tarde, 0 si llegó a tiempo).
  - `is_interstate_sale`: indicador binario de si el vendedor y el cliente están en estados distintos de Brasil.
  - `freight_ratio`: peso del coste de envío sobre el valor total (`freight / (price + freight)`).
  - Variables de calendario: año, mes, día de la semana y franja horaria (madrugada, mañana, tarde, noche).
  - `review_sentiment`: segmentación en positiva (4-5), neutra (3) y negativa (1-2).

El dataset limpio final se guardó en `data/processed/olist_cleaned_merged.csv` con **112.650 filas y 55 columnas**.

### 3. Análisis exploratorio y estadística (`03_analisis_exploratorio_y_estadistica.ipynb`)
- **Estudio univariante y bivariante**: Analicé las distribuciones de precios, fletes y plazos de entrega. La media de entrega está en 12,1 días (mediana de 10,2 días).
- **Logística interestatal**: El 65,4% de las compras cruzan fronteras de estados. En estos envíos el flete sube de 14,2 R$ a 23,8 R$ y el plazo de entrega se duplica.
- **Estacionalidad**: Se observa un crecimiento continuo durante 2017 y 2018, con un pico histórico en el Black Friday de noviembre de 2017. Este pico provocó un colapso en las empresas de transporte en diciembre de 2017, disparando la tasa de retrasos al 18,2% y desplomando las valoraciones.
- **Contraste de hipótesis estadísticas**:
  - **Test U de Mann-Whitney**: Comprobé que los tiempos de entrega de los clientes que puntúan con 1 estrella (mediana 17,5 días) son significativamente mayores que los de 5 estrellas (mediana 9,8 días), con un p-valor $< 0,0001$.
  - **Test Chi-cuadrado y Odds Ratio**: Existe una dependencia muy fuerte entre retraso logístico y quejas. Un pedido con retraso tiene **14,8 veces más probabilidades** de recibir una reseña negativa que uno que llega a tiempo.
- **Segmentación RFM**: Calculé la recencia, frecuencia y gasto monetario para los 95.420 clientes únicos. El 96,9% de los clientes ha comprado una sola vez, lo que muestra que Olist tiene un problema estructural de fidelización.

### 4. Cuadro de mando y visualización (`dashboard/`)
Para la presentación del cuadro de mando he preparado el archivo listo para consultar:
- **`dashboard_ejecutivo_olist.xlsx`**: Cuadro de mando ejecutivo interactivo en Excel organizado en 3 hojas (`Datos`, `Analisis` y `Dashboard`), construido con **4 segmentadores de datos nativos (Slicers)** (Año de compra, Puntualidad de entrega, Método de pago y Satisfacción/Reseña) vinculados a tablas y gráficos dinámicos sincronizados (comparativa de satisfacción con/sin retraso, evolución mensual de ventas y ranking de categorías). Al hacer clic en cualquier botón de los segmentadores, todos los gráficos del panel se recalculan en tiempo real sin necesidad de configurar nada.

---

## Principales conclusiones del análisis

1. **La entrega a tiempo es el factor clave de la satisfacción**: El precio del producto o el coste de envío apenas influyen en la nota si el pedido llega a tiempo. Sin embargo, cuando hay retraso, la nota media cae en picado de **4,25 a 1,74**.
2. **El problema de los envíos interestatales**: Al estar la mayoría de vendedores concentrados en São Paulo, enviar a estados del norte o nordeste supone plazos de 2 a 3 semanas y fletes mucho más caros.
3. **Colapso en picos de demanda**: En campañas como el Black Friday se vende mucho más, pero si la logística no acompaña se multiplica el descontento y se deteriora la imagen de la plataforma.
4. **Falta de recurrencia**: El 97% de los usuarios no vuelve a comprar, por lo que casi todo el esfuerzo comercial se va en captar clientes nuevos en lugar de retener a los existentes.

---

## Recomendaciones para el negocio

1. **Ajustar la fecha de entrega estimada según la ruta**: En rutas lejanas es mejor prometer 18 días y llegar en 15 (cliente contento con 5 estrellas) que prometer 12 y llegar en 13 (cliente enfadado con 1 estrella).
2. **Crear almacenes regionales de distribución**: Incentivar pequeños hubs logísticos en el Nordeste para los productos más vendidos y evitar enviar todo desde São Paulo.
3. **Avisar proactivamente de incidencias**: Si el transportista se demora, avisar al cliente antes de que venza la fecha prometida y ofrecer un pequeño cupón para mitigar la molestia.
4. **Estrategia de recompra**: Aprovechar a los clientes que tuvieron una buena experiencia (4 y 5 estrellas) con descuentos personalizados en categorías de reposición habitual (salud, belleza, papelería).

---

## Requisitos para ejecutar el proyecto

Instalar las librerías necesarias ejecutando:

```bash
pip install -r requirements.txt
```

Para revisar los cuadernos, abrirlos en VS Code o Jupyter en este orden:
1. `notebooks/01_carga_y_cruce_de_datos.ipynb`
2. `notebooks/02_limpieza_y_transformacion.ipynb`
3. `notebooks/03_analisis_exploratorio_y_estadistica.ipynb`

---

## Autor

Jose Ayala Farell