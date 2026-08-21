# 01 · Diccionario de datos — Componente A

Caso Austral Seafood Chile SpA (ASC). Fecha de corte del análisis: **31-12-2025**.

---

## 1. Convenciones de los archivos

| Aspecto | Valor |
|---|---|
| Ubicación | `data/raw/` |
| Formato | CSV plano |
| Separador de columnas | punto y coma (`;`) |
| Codificación | UTF-8 con BOM |
| Separador decimal | punto (`.`) |
| Separador de miles | no se usa |
| Formato de fecha | ISO `AAAA-MM-DD` |
| Valores nulos | celda vacía (no se usa `NULL` ni `N/A`) |
| Fin de línea | `LF` |

**Al conectar desde Power BI o Excel**, fije el origen del archivo en `65001: Unicode (UTF-8)` y la configuración regional del paso de importación en **Inglés (Estados Unidos)**. Si importa con configuración regional chilena, el punto decimal se interpretará como separador de miles y los montos quedarán multiplicados por mil.

### Volumen de los archivos

| Archivo | Filas | Granularidad |
|---|---:|---|
| `fact_facturas.csv` | 4.381 | Un documento tributario emitido |
| `fact_cobranzas.csv` | 4.311 | Una aplicación de pago a un documento |
| `fact_notas_credito.csv` | 295 | Una nota de crédito emitida |
| `fact_gestiones_cobranza.csv` | 1.032 | Una acción de cobranza registrada |
| `dim_cliente.csv` | 344 | Un cliente |
| `dim_vendedor.csv` | 12 | Un vendedor |
| `dim_condicion_pago.csv` | 6 | Una condición comercial |
| `dim_moneda_tipo_cambio.csv` | 1.462 | Una moneda por día (CLP y USD) |
| `dim_calendario.csv` | 1.096 | Un día, del 01-01-2024 al 31-12-2026 |

---

## 2. `fact_facturas.csv`

Documentos tributarios emitidos entre el 02-01-2024 y el 31-12-2025.

| # | Campo | Tipo | Nulo | Descripción |
|---|---|---|---|---|
| 1 | `id_documento` | texto | no | Identificador del documento. Formato `DOC-AAAA-NNNNN`. Es único en el archivo. |
| 2 | `tipo_documento` | texto | no | `FAC` factura afecta a IVA; `FEX` factura de exportación; `FEE` factura exenta. |
| 3 | `folio` | entero | no | Folio del documento tributario. Correlativo por tipo de documento. |
| 4 | `id_cliente` | texto | no | FK a `dim_cliente.id_cliente`. |
| 5 | `id_vendedor` | texto | no | FK a `dim_vendedor.id_vendedor`. |
| 6 | `id_condicion` | texto | no | FK a `dim_condicion_pago.id_condicion`. Condición **pactada** en la orden de venta. |
| 7 | `linea_negocio` | texto | no | `EXPORTACION`, `FOODSERVICE`, `RETAIL`. |
| 8 | `fecha_emision` | fecha | no | Fecha de emisión del documento. Siempre día hábil. |
| 9 | `fecha_vencimiento` | fecha | no | Fecha de vencimiento **registrada en el sistema**. No siempre coincide con `fecha_emision + dias_credito`. |
| 10 | `moneda` | texto | no | `CLP` o `USD`. Moneda de emisión. |
| 11 | `monto_neto` | decimal | no | Monto neto en moneda de emisión. Enteros en CLP; dos decimales en USD. |
| 12 | `monto_iva` | decimal | no | IVA 19% para `FAC`; `0` para `FEX` y `FEE`. |
| 13 | `monto_total` | decimal | no | `monto_neto + monto_iva`, en moneda de emisión. |
| 14 | `estado_sii` | texto | no | `ACEPTADO`, `RECLAMADO`, `ANULADO`. |
| 15 | `centro_costo` | texto | no | Unidad de origen del despacho. |

### Dominios observados

| Campo | Valores y frecuencia |
|---|---|
| `tipo_documento` | FAC 3.249 · FEX 881 · FEE 251 |
| `linea_negocio` | FOODSERVICE 2.758 · EXPORTACION 881 · RETAIL 742 |
| `moneda` | CLP 3.584 · USD 797 |
| `estado_sii` | ACEPTADO 4.266 · ANULADO 58 · RECLAMADO 57 |
| `centro_costo` | PLANTA_TALCAHUANO 1.380 · PLANTA_CORONEL 1.372 · CD_SANTIAGO 1.193 · PLANTA_QUELLON 230 · PLANTA_CALDERA 206 |

### Notas de interpretación

- **Toda factura de exportación (`FEX`) es exenta de IVA.** El 90% se emite en USD.
- El `monto_total` está expresado en la **moneda de emisión**. Cualquier agregación que mezcle CLP y USD sin convertir está mal.
- La conversión a CLP se hace con el tipo de cambio observado de la **fecha de emisión** del documento, no con el de cierre ni con el de la fecha de pago. La diferencia de cambio posterior es un resultado financiero, no un mayor ingreso.
- `estado_sii = ANULADO` significa que el documento fue anulado ante el SII: no genera derecho de cobro. La mayoría, pero no todos, tiene además una nota de crédito de anulación asociada.
- `estado_sii = RECLAMADO` significa que el receptor reclamó el documento dentro del plazo del artículo 3 de la Ley 19.983: el documento no constituye título ejecutivo y no debe integrar la cartera exigible.

---

## 3. `fact_cobranzas.csv`

| # | Campo | Tipo | Nulo | Descripción |
|---|---|---|---|---|
| 1 | `id_cobranza` | texto | no | Identificador de la aplicación de pago. Formato `COB-NNNNNN`. |
| 2 | `id_documento` | texto | no | Documento al que se aplica. **No siempre existe en `fact_facturas`.** |
| 3 | `fecha_pago` | fecha | no | Fecha efectiva de recaudación. Rango: 2024-01-10 a 2025-12-31. |
| 4 | `moneda` | texto | no | Moneda del pago; coincide con la del documento. |
| 5 | `monto_pagado` | decimal | no | Monto aplicado al documento. |
| 6 | `medio_pago` | texto | no | `TRANSFERENCIA`, `CHEQUE`, `FACTORING`, `COMPENSACION`. |
| 7 | `id_operacion_factoring` | texto | sí | Identificador de la operación de cesión. Vacío salvo factoring. |

### Dominios observados

`TRANSFERENCIA` 3.484 · `CHEQUE` 475 · `FACTORING` 204 · `COMPENSACION` 148

### Notas de interpretación

- Un documento admite **varias aplicaciones de pago**. Para obtener el saldo hay que agregar por `id_documento`.
- En las operaciones de **factoring**, el registro con `medio_pago = FACTORING` corresponde al **anticipo del factor** (90% del monto exigible). El 10% restante es la retención o aforo, que se libera con un segundo movimiento asociado al mismo `id_operacion_factoring` cuando el cliente paga al factor. Si ese segundo movimiento no existe al corte, el documento conserva saldo abierto en los libros de ASC.
- Los pagos cuyo `id_documento` no existe en `fact_facturas` son **huérfanos**. No pueden aplicarse a ninguna cartera y deben reportarse como hallazgo, no eliminarse en silencio.

---

## 4. `fact_notas_credito.csv`

| # | Campo | Tipo | Nulo | Descripción |
|---|---|---|---|---|
| 1 | `id_nota_credito` | texto | no | Identificador. Formato `NC-NNNNN`. |
| 2 | `folio_nc` | entero | no | Folio de la nota de crédito. |
| 3 | `id_documento_referencia` | texto | no | Documento que rebaja. FK a `fact_facturas`. |
| 4 | `fecha_emision` | fecha | no | Fecha de emisión de la nota. |
| 5 | `motivo` | texto | no | `DEVOLUCION`, `DESCUENTO`, `ANULACION`, `AJUSTE_CALIDAD`. |
| 6 | `moneda` | texto | no | Igual a la del documento referenciado. |
| 7 | `monto_total` | decimal | no | Monto de la nota, en la moneda del documento. |

### Dominios observados

`DESCUENTO` 97 · `DEVOLUCION` 78 · `AJUSTE_CALIDAD` 69 · `ANULACION` 51

### Notas de interpretación

- Las notas de crédito **rebajan el monto exigible** del documento referenciado. Un documento puede tener más de una.
- `AJUSTE_CALIDAD` no es un problema de cobranza: es una devolución por producto no conforme. Su distribución por cliente, planta y período dice algo sobre la operación, no sobre el crédito.
- Las notas emitidas en la segunda quincena de diciembre de 2025 pueden no estar aplicadas al corte. Decidir si se consideran "pendientes de aplicar" es parte del análisis de deducciones de caja.

---

## 5. `fact_gestiones_cobranza.csv`

| # | Campo | Tipo | Nulo | Descripción |
|---|---|---|---|---|
| 1 | `id_gestion` | texto | no | Identificador. Formato `GES-NNNNNN`. |
| 2 | `id_documento` | texto | no | Documento gestionado. FK a `fact_facturas`. |
| 3 | `fecha_gestion` | fecha | no | Fecha de la acción. |
| 4 | `tipo_gestion` | texto | no | `LLAMADA`, `EMAIL`, `CARTA_CERTIFICADA`, `VISITA`, `PREJUDICIAL`, `DEMANDA_JUDICIAL`. |
| 5 | `resultado` | texto | no | `COMPROMISO_PAGO`, `SIN_RESPUESTA`, `DISPUTA`, `PAGO_PARCIAL`, `INCOBRABLE_DECLARADO`. |
| 6 | `ejecutivo_cobranza` | texto | no | Responsable que ejecutó la gestión. |

### Notas de interpretación

- Solo se registran gestiones sobre documentos **vencidos**. Un documento sin filas en esta tabla es un documento que **nadie gestionó**.
- La escalada natural es `LLAMADA`/`EMAIL` → `CARTA_CERTIFICADA` → `PREJUDICIAL` → `DEMANDA_JUDICIAL`. Esta secuencia es la evidencia del agotamiento prudencial de los medios de cobro que exige el criterio tributario de castigo.
- El `ejecutivo_cobranza` de la gestión puede diferir del asignado en `dim_cliente` si hubo reasignaciones. Verifíquelo antes de medir productividad por ejecutivo.

---

## 6. `dim_cliente.csv`

| # | Campo | Tipo | Nulo | Descripción |
|---|---|---|---|---|
| 1 | `id_cliente` | texto | no | Identificador. Formato `CLNNNN`. Clave primaria. |
| 2 | `razon_social` | texto | no | Nombre del cliente. |
| 3 | `rut` | texto | sí | RUT con dígito verificador. **Vacío para clientes extranjeros.** |
| 4 | `pais` | texto | no | País de destino. `CHILE` para clientes nacionales. |
| 5 | `segmento` | texto | no | `EXPORTADOR`, `FOODSERVICE`, `CADENA_RETAIL`, `MAYORISTA`. |
| 6 | `fecha_alta` | fecha | no | Fecha de incorporación como cliente. |
| 7 | `linea_credito_clp` | decimal | no | Cupo de crédito autorizado, siempre expresado en CLP. |
| 8 | `clasificacion_riesgo` | texto | no | `A` (mejor) a `D` (peor). Evaluación comercial interna. |
| 9 | `estado` | texto | no | `ACTIVO`, `INACTIVO`, `EN_COBRANZA_JUDICIAL`, `QUIEBRA`. |
| 10 | `ejecutivo_cobranza` | texto | no | Responsable asignado de la cartera del cliente. |

### Dominios observados

| Campo | Distribución |
|---|---|
| `segmento` | FOODSERVICE 150 · MAYORISTA 76 · EXPORTADOR 72 · CADENA_RETAIL 46 |
| `clasificacion_riesgo` | B 128 · A 92 · C 87 · D 37 |
| `estado` | ACTIVO 312 · INACTIVO 18 · EN_COBRANZA_JUDICIAL 10 · QUIEBRA 4 |

### Notas de interpretación

- La `linea_credito_clp` está en pesos. Para evaluar la exposición de un cliente que factura en USD, debe convertir su cartera a CLP antes de comparar.
- Un cliente `INACTIVO` puede tener cartera abierta: dejó de comprar, no de deber.
- La relación entre `segmento` y `linea_negocio` de las facturas no es uno a uno: `CADENA_RETAIL` y `MAYORISTA` alimentan ambos la línea `RETAIL`.

---

## 7. `dim_vendedor.csv`

| # | Campo | Tipo | Descripción |
|---|---|---|---|
| 1 | `id_vendedor` | texto | Identificador. Formato `VNNN`. |
| 2 | `nombre_vendedor` | texto | Nombre del ejecutivo comercial. |
| 3 | `linea_negocio` | texto | Línea que atiende. |
| 4 | `zona` | texto | Zona o mercado asignado. |
| 5 | `fecha_ingreso` | fecha | Fecha de ingreso a la empresa. |
| 6 | `meta_anual_clp` | decimal | Meta comercial anual en CLP. |
| 7 | `estado` | texto | `ACTIVO`. |

---

## 8. `dim_condicion_pago.csv`

| # | Campo | Tipo | Descripción |
|---|---|---|---|
| 1 | `id_condicion` | texto | `CP00`, `CP30`, `CP45`, `CP60`, `CP90`, `CP120`. |
| 2 | `descripcion` | texto | Contado, 30 días, 45 días, 60 días, 90 días, 120 días. |
| 3 | `dias_credito` | entero | 0, 30, 45, 60, 90, 120. |

Esta tabla define el plazo **pactado**. La `fecha_vencimiento` de la factura es el plazo **registrado**. Comparar ambos es un control de cumplimiento de la política comercial.

---

## 9. `dim_moneda_tipo_cambio.csv`

| # | Campo | Tipo | Descripción |
|---|---|---|---|
| 1 | `fecha` | fecha | Día calendario, del 01-01-2024 al 31-12-2025. |
| 2 | `moneda` | texto | `USD` o `CLP`. |
| 3 | `tipo_cambio_clp` | decimal | Pesos por unidad de moneda. Para `CLP` siempre `1.00`. |
| 4 | `tipo_publicacion` | texto | `OBSERVADO` en días hábiles; `ARRASTRE` en fines de semana y feriados, donde se repite el último valor publicado. |

La serie **termina el 31-12-2025**. Cualquier tipo de cambio usado para proyectar 2026 es un supuesto del equipo y debe declararse de manera explícita al exponer.

---

## 10. `dim_calendario.csv`

Dimensión de fecha continua del 01-01-2024 al 31-12-2026, para permitir la proyección del primer semestre 2026.

| Campo | Descripción |
|---|---|
| `fecha` | Clave de la dimensión. |
| `anio`, `trimestre`, `mes`, `dia` | Componentes numéricos. |
| `nombre_mes`, `nombre_dia` | Etiquetas de texto. |
| `anio_mes` | `AAAA-MM`, útil como eje de la proyección mensual. |
| `anio_trimestre` | `AAAA-TN`. |
| `semana_iso` | Número de semana ISO. |
| `es_dia_habil` | 1 si es día hábil bancario; 0 en fin de semana o feriado. |
| `es_feriado` | 1 si es feriado legal. |
| `inicio_mes`, `fin_mes` | Primer y último día del mes de la fecha. |

Los feriados corresponden a los feriados legales chilenos de calendario fijo más Viernes y Sábado Santo. Es una aproximación suficiente para el caso; no reemplaza al calendario oficial.

---

## 11. Relaciones del modelo

El esquema entregado es transaccional. Una lectura razonable de las relaciones es:

```
dim_calendario[fecha]  1 ── *  fact_facturas[fecha_emision]
dim_cliente[id_cliente]  1 ── *  fact_facturas[id_cliente]
dim_vendedor[id_vendedor]  1 ── *  fact_facturas[id_vendedor]
dim_condicion_pago[id_condicion]  1 ── *  fact_facturas[id_condicion]
dim_moneda_tipo_cambio[fecha+moneda]  1 ── *  fact_facturas[fecha_emision+moneda]
fact_facturas[id_documento]  1 ── *  fact_cobranzas[id_documento]
fact_facturas[id_documento]  1 ── *  fact_notas_credito[id_documento_referencia]
fact_facturas[id_documento]  1 ── *  fact_gestiones_cobranza[id_documento]
```

Construir este modelo es parte del desafío, y las decisiones de diseño se evalúan. Algunas preguntas que el equipo debe resolver por sí mismo:

- ¿`fact_facturas` es una tabla de hechos o una dimensión degenerada respecto de cobranzas, notas de crédito y gestiones?
- ¿Cómo se relaciona el calendario con tablas que tienen más de una fecha relevante (emisión, vencimiento, pago)?
- ¿Conviene precalcular el saldo insoluto por documento en el ETL, o resolverlo con medidas DAX?
- ¿La conversión a CLP se materializa como columna en el modelo o se calcula en cada medida?

---

## 12. Checklist de validación previa

Antes de calcular un solo indicador, verifique al menos lo siguiente. Deje registro del resultado de cada control: el jurado puede preguntar por cualquiera de ellos, y las respuestas forman parte de la evaluación.

| # | Control | Pregunta |
|---|---|---|
| 1 | Unicidad | ¿`id_documento` es único? ¿Y la combinación `tipo_documento` + `folio`? |
| 2 | Integridad referencial | ¿Todo `id_cliente`, `id_vendedor` y `id_condicion` de las facturas existe en su maestro? |
| 3 | Huérfanos | ¿Todo `id_documento` de cobranzas, notas de crédito y gestiones existe en `fact_facturas`? |
| 4 | Coherencia aritmética | ¿`monto_total = monto_neto + monto_iva` en todas las filas? ¿El IVA es 19% en las `FAC` y cero en `FEX`/`FEE`? |
| 5 | Coherencia temporal | ¿Hay documentos con `fecha_vencimiento < fecha_emision`? ¿Pagos anteriores a la emisión del documento? |
| 6 | Política comercial | ¿`fecha_vencimiento - fecha_emision` coincide con `dias_credito` de la condición pactada? |
| 7 | Sobrepago | ¿Existen documentos donde pagos más notas de crédito superan el `monto_total`? |
| 8 | Moneda | ¿La moneda del pago coincide siempre con la del documento? ¿Hay fechas de emisión sin tipo de cambio publicado? |
| 9 | Cartera exigible | ¿Qué saldo aportan los documentos `RECLAMADO` y `ANULADO`? ¿Se excluyeron? |
| 10 | Cobertura de gestión | ¿Cuántos documentos vencidos no tienen ninguna gestión registrada, y por cuánto monto? |

> No basta con detectar los problemas: hay que decidir qué hacer con cada uno y dejar constancia del criterio. Una fila descartada sin justificación es un error; una fila conservada sin advertencia también.
