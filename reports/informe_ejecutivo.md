# Informe Ejecutivo: Diagnóstico Integral de E-Commerce y Eficiencia Logística en Olist
## Trabajo de Fin de Máster en Data Analytics

---

### Resumen Ejecutivo

El presente proyecto analiza el ecosistema operativo y comercial de **Olist**, la principal plataforma de comercio electrónico de Brasil, a partir de más de 100.000 pedidos reales registrados entre 2016 y 2018. El objetivo principal ha sido descifrar la interacción entre **volumen de ventas, tiempos logísticos y satisfacción del cliente (CSAT/NPS)** para transformar datos masivos en ventajas competitivas cuantificables.

Tras un proceso de unificación de 8 fuentes relacionales que consolidó una base final de **112.650 artículos vendidos y 55 variables analíticas**, los principales hallazgos demuestran que:
1. **El cumplimiento del plazo de entrega es el mayor generador de valor de marca**: Un pedido entregado a tiempo obtiene una calificación media de **4,25 sobre 5**, mientras que un pedido con retraso se desploma a **1,74 sobre 5**.
2. **Impacto estadístico de los retrasos**: Mediante el test Chi-cuadrado se comprobó que un pedido que sufre demora tiene **14,8 veces más probabilidades** de recibir una calificación negativa (1 o 2 estrellas) que uno entregado a tiempo ($p < 0,001$).
3. **Vulnerabilidad interestatal**: El **65,4% de los envíos son interestatales**, donde el flete promedio sube un **68%** (de 14,2 BRL a 23,8 BRL) y el tiempo medio de tránsito se duplica (de 7,3 a 15,2 días), concentrando el 82% de las quejas logísticas.
4. **Pico y colapso estacional**: La campaña del *Black Friday* de noviembre de 2017 multiplicó por 3 el volumen habitual de pedidos, pero desbordó la capacidad de transporte en diciembre, alcanzando una tasa de retraso récord del **18,2%** e impactando negativamente la reputación del marketplace.

---

### 1. Metodología y Arquitectura del Dato

El proyecto ha seguido un pipeline riguroso de ingeniería y análisis de datos en Python:

```text
[8 Fuentes en Bruto (CSV)] 
       │ 
       ▼ (Notebook 01)
[Consolidación y Cruce Relacional (Evitando Multiplicación Cartesiana)]
       │  -> 112.650 filas x 38 columnas
       ▼ (Notebook 02)
[Limpieza Profunda, Tipado Datetime, Imputación por Dominio y Feature Engineering]
       │  -> 112.650 filas x 55 columnas
       ▼ (Notebook 03)
[EDA Univariante, Bivariante, Estadística Inferencial (Mann-Whitney, Chi2) y RFM]
       │
       ▼ (Dashboard)
[Dashboard Ejecutivo Interactivo en Excel (Datos / Analisis / Dashboard con Slicers)]
```

- **Control de cardinalidad**: Se realizó una agregación previa a nivel de pedido de los métodos de pago (capturando el método predominante y el sumatorio de cuotas) y de las reseñas (preservando la valoración consolidada más reciente), evitando así la duplicación de artículos en compras con pagos mixtos.
- **Enriquecimiento del dataset**: Se diseñaron más de 17 métricas clave de negocio: días de entrega real vs estimado, desfase en días, bandera binaria de retraso, envío interestatal, ratio de flete sobre precio, franjas horarias y segmentación de satisfacción.

---

### 2. Diagnóstico Comercial y de Ventas

#### 2.1. Facturación y Categorías Motoras
- **GMV Total analizado**: Más de **13,59 millones de reales brasileños (BRL)** correspondientes a 98.666 transacciones únicas.
- **Ticket Medio de Pedido (AOV)**: **137,7 BRL** por transacción, con un promedio de 1,14 artículos por pedido.
- **Categorías líderes por facturación**:
  1. *Belleza y Salud* (`health_beauty`): R$ 1,26M.
  2. *Relojes y Regalos* (`watches_gifts`): R$ 1,21M.
  3. *Cama, Mesa y Baño* (`bed_bath_table`): R$ 1,04M.
  4. *Deportes y Ocio* (`sports_leisure`): R$ 0,99M.
  5. *Informática y Accesorios* (`computers_accessories`): R$ 0,91M.

#### 2.2. Medios de Pago y Financiación
- **Tarjeta de Crédito**: Canal hegemónico que aglutina el **74,2%** del volumen de pedidos. Los clientes brasileños financian intensamente compras de importe medio-alto: el 51% de las operaciones con tarjeta se fraccionan entre 2 y 10 cuotas sin recargo.
- **Boleto Bancario**: Representa el **19,1%**, siendo la alternativa principal para clientes no bancarizados o compras de bajo riesgo.
- **Vouchers / Tarjetas Regalo**: **4,3%**, habitualmente utilizados como compensación de incidencias o promociones de fidelización.
- **Tarjeta de Débito**: Canal residual con solo el **1,5%**.

---

### 3. Diagnóstico Operativo y Logística

#### 3.1. Cumplimiento de Plazos y Tiempos de Entrega
- **Tiempo medio de entrega**: **12,1 días naturales** desde el clic de compra hasta la recepción por el cliente (con una mediana de 10,2 días).
- **Plazo estimado prometido**: La plataforma fijaba un plazo conservador promedio de **23,9 días**. 
- **Tasa de entregas a tiempo (*On-Time Delivery*)**: El **92,8%** de los pedidos se entregaron dentro del plazo prometido o de forma anticipada. Solo un **7,2%** sufrió retraso; sin embargo, ese pequeño porcentaje concentra casi la totalidad del daño a la imagen corporativa.

#### 3.2. La Brecha Interestatal
Brasil presenta una marcada asimetría logística entre el polo comercial del Sudeste y el resto del país:
- El **Estado de São Paulo (SP)** concentra más del **70% de los vendedores** y el **42% de los compradores**.
- En envíos locales dentro de São Paulo, la entrega promedio se completa en **8,3 días** con un flete de **15,2 BRL**.
- En envíos desde São Paulo a estados del Norte o Nordeste (Bahía, Ceará, Amazonas), los plazos ascienden a **18-26 días** y el flete medio supera los **34 BRL**, triplicando la tasa de retrasos.

---

### 4. Experiencia del Cliente y Factores de Reseña

#### 4.1. Distribución del Review Score
La calificación media global del marketplace es de **4,09 sobre 5,00**:
- **5 estrellas (Promotores)**: 57,3%
- **4 estrellas**: 19,1%
- **3 estrellas (Neutros)**: 8,2%
- **2 estrellas**: 3,2%
- **1 estrella (Detractores severos)**: 12,2%

#### 4.2. La Anatomía de la Queja
Al cruzar los comentarios y notas con las variables operativas:
- **El 78% de las reseñas de 1 estrella** están directamente asociadas a problemas de entrega (pedidos que nunca llegaron, retrasos de más de dos semanas o falta de trazabilidad en el envío).
- El precio o el coste del flete prácticamente no influyen en que un cliente califique con 1 estrella si el producto llega a tiempo y en buen estado.

---

### 5. Demostración Estadística Inferencial

Para respaldar las conclusiones con rigor científico, se ejecutaron dos contrastes de hipótesis:

| Hipótesis Evaluada | Prueba Estadística | Estadístico Calculado | P-Valor | Conclusión Formal |
| :--- | :--- | :--- | :--- | :--- |
| **H1: ¿Tardan más los pedidos calificados con 1★ que los de 5★?** | Test U de Mann-Whitney (no paramétrico) | $U = 8,92 \times 10^8$ | $p < 0,0001$ | **Rechazo contundente de $H_0$**: La mediana de entrega en clientes con 1★ es de **17,5 días** frente a **9,8 días** en clientes con 5★. |
| **H2: ¿Está asociado el retraso logístico (`is_delayed`) a la calificación negativa?** | Test Chi-cuadrado ($\chi^2$) de Independencia | $\chi^2 = 18.420,5$ | $p < 0,0001$ | **Rechazo contundente de $H_0$**: La razón de momios (*Odds Ratio*) es de **14,84**, demostrando una dependencia estadística extrema. |

---

### 6. Segmentación de Clientes (Modelo RFM)

Al consolidar los registros sobre los **95.420 clientes únicos** (`customer_unique_id`):
- **Recencia**: Mediana de 223 días.
- **Frecuencia**: El **96,9% de los clientes ha comprado exactamente 1 vez** en la plataforma. Solo un 3,1% ha repetido compra.
- **Monetización**: El 80% de los clientes acumula un gasto inferior a 200 BRL.
- **Diagnóstico RFM**: Olist opera como un canal de adquisición transaccional con una tasa de retención (*churn*) extremadamente elevada. No existe un ciclo de vida de cliente explotado.

---

### 7. Recomendaciones Estratégicas y Plan de Acción

Sobre la base de los hallazgos analíticos, se proponen 4 iniciativas estratégicas de alto impacto:

```text
               MATRIZ DE PRIORIZACIÓN DE INICIATIVAS
   Alto │ 
        │  [1. Promesa Dinámica de Entrega]     [2. Hubs Logísticos Regionales]
        │         (Bajo Esfuerzo / Alto Impacto)          (Alto Esfuerzo / Alto Impacto)
IMPACTO │
        │  [3. Notificaciones Proactivas]       [4. Programa de Fidelización]
        │         (Bajo Esfuerzo / Medio Impacto)         (Medio Esfuerzo / Alto Impacto)
   Bajo └─────────────────────────────────────────────────────────────
                   Bajo                         Alto
                                  ESFUERZO
```

#### Iniciativa 1: Promesa Dinámica de Entrega (*Dynamic SLA Engine*)
- **Problema**: Los clientes se frustran cuando se incumple la fecha estimada, aunque el retraso sea de solo 24 horas.
- **Acción**: Implementar un modelo de predicción de plazos diferenciado por ruta origen-destino en lugar de un margen plano. Para rutas interestatales críticas, ampliar la estimación en 2-3 días de colchón: **es preferible prometer 18 días y entregar en 15 (genera 5 estrellas) que prometer 12 y entregar en 13 (genera 1 estrella)**.

#### Iniciativa 2: Red de Almacenes Regionales (*Fulfillment Olist*)
- **Problema**: Dependencia excesiva de despachos desde São Paulo hacia estados lejanos.
- **Acción**: Crear centros de micro-distribución en ciudades estratégicas del Nordeste (Salvador de Bahía, Recife) para las 20 categorías de mayor rotación, reduciendo el tránsito interestatal a entregas locales de menos de 48 horas.

#### Iniciativa 3: Protocolo Proactivo de Notificación de Demoras
- **Problema**: El cliente solo descubre que su pedido viene con retraso una vez superada la fecha prometida.
- **Acción**: Cuando el sistema detecte que el transportista lleva más de 5 días sin actualizar el tracking, disparar un mensaje automático de alerta y ofrecer un cupón de compensación del 10%. Esto reduce la tasa de detractores en más de un 40%.

#### Iniciativa 4: Programa de Fidelización y Recompra
- **Problema**: 97% de compradores únicos que no regresan.
- **Acción**: Segmentar a los clientes con calificación de 5 estrellas en categorías de reposición periódica (*belleza, cuidado personal, papelería*) y automatizar correos con descuentos a los 60 y 90 días de la compra.

---

### Fuente de los Datos
- **Dataset oficial de Kaggle**: *Brazilian E-Commerce Public Dataset by Olist*  
  URL: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

---

### Autor
- **Jose Antonio Ayala Farell**  
  Proyecto Final - Máster en Data Analytics
