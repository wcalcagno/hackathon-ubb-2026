# 04 · Glosario de indicadores

Definiciones operativas de los indicadores exigidos. Donde una fórmula admite más de una convención, se indica cuál se usa en este caso y qué debe declararse.

Fecha de corte: **31-12-2025**. Período de referencia para los flujos: **ejercicio 2025**.

---

## 1. Conceptos base

**Cartera bruta.** Suma de los saldos insolutos de todos los documentos con saldo mayor que cero, convertidos a CLP al tipo de cambio de la fecha de emisión.

**Cartera exigible.** Cartera bruta menos documentos `RECLAMADO`, `ANULADO` y duplicados descartados. Es la base de casi todos los indicadores.

**Cartera vencida.** Documentos de la cartera exigible con `dias_mora >= 0`.

**Cartera por vencer (vigente).** Documentos con `dias_mora < 0`.

**Cartera neta recuperable.** Resultado de la cascada de deducciones de la sección 4 de las reglas de negocio.

**Ventas del período.** Suma de `monto_neto` de los documentos emitidos en el período, convertidos a CLP a la fecha de emisión, excluyendo `ANULADO` y neteando las notas de crédito. Se usa el monto **neto**, sin IVA: el IVA no es ingreso.

> Advertencia sobre la base: el DSO relaciona un saldo que **incluye IVA** con ventas que **no lo incluyen**. Es la convención más usada en la práctica y es la que se aplica en este caso, pero el equipo debe declararla. Una alternativa igualmente válida es medir ambos términos sin IVA; lo que no es válido es mezclar criterios entre indicadores sin advertirlo.

---

## 2. Indicadores obligatorios

### 2.1 DSO — Days Sales Outstanding

```
DSO = (Cuentas por cobrar / Ventas del período) x días del período
```

- Cuentas por cobrar: cartera exigible al 31-12-2025.
- Ventas del período: ventas netas del ejercicio 2025.
- Días del período: 365.

**Interpretación.** Días promedio que ASC tarda en convertir una venta en efectivo. Un DSO de 90 significa que, en promedio, cada peso vendido permanece tres meses como derecho de cobro antes de recaudarse.

**Desagregación exigida:** total, por línea de negocio y por segmento.

**Cuidado.** Al desagregar, el numerador y el denominador deben corresponder al mismo corte: la cartera de exportación se divide por las ventas de exportación, no por las ventas totales. Además, un DSO por línea no es comparable entre líneas si los plazos pactados difieren: exportación opera a 60-120 días y foodservice a 30-60. Compare cada línea contra su propio plazo pactado, no contra el promedio de la empresa.

### 2.2 DSO ajustado

```
DSO ajustado = ((Cartera exigible - Cartera incobrable - Cartera factorizada) / Ventas del período) x 365
```

**Para qué sirve.** El DSO simple castiga a la empresa por cartera que ya no va a recaudar y por cartera que ya cedió. El DSO ajustado mide el desempeño de la cobranza sobre lo que efectivamente está gestionando. La diferencia entre ambos indicadores es, en sí misma, un hallazgo: mide cuánto del deterioro proviene del pasado no resuelto.

### 2.3 Índice de morosidad

```
Índice de morosidad = Cartera vencida / Cartera total
```

**Desagregación exigida:** total, por línea de negocio y por vendedor.

**Cuidado.** Al calcularlo por vendedor, un vendedor con cartera pequeña puede mostrar porcentajes extremos por unos pocos documentos. Reporte siempre el índice junto al **monto** de cartera y, de ser posible, junto a la mora promedio ponderada por saldo. Un ranking de porcentajes sin considerar volumen lleva a conclusiones equivocadas.

### 2.4 Tasa de incobrabilidad

```
Tasa de incobrabilidad = Cartera clasificada como incobrable / Ventas del período
```

**Desagregación exigida:** total y por segmento.

**Interpretación.** Qué proporción de lo vendido en el período terminó siendo irrecuperable. Es el indicador que traduce el problema de cobranza a lenguaje de resultado.

### 2.5 Índice de concentración

```
Índice de concentración = Saldo de los 10 mayores deudores / Cartera total
```

**Interpretación.** Mide el riesgo de dependencia. Una concentración alta no es en sí misma una falla —es propia del negocio exportador— pero sí un riesgo que debe estar identificado, con límites y monitoreo. Complementos valorados: curva de Pareto de la cartera, o índice de Herfindahl-Hirschman.

### 2.6 Rotación de cuentas por cobrar

```
Rotación = Ventas a crédito del período / Cuentas por cobrar promedio
```

- Ventas a crédito: ventas del período excluyendo las de condición `Contado` (`CP00`).
- Cuentas por cobrar promedio: promedio de los saldos de cartera. Con los datos disponibles puede estimarse como el promedio de los saldos de cierre mensuales de 2025, que es más robusto que el promedio simple entre saldo inicial y final.

**Relación con el DSO.** `DSO ≈ 365 / Rotación`. Si ambos indicadores no son consistentes entre sí, hay un error de base en alguno de los dos.

### 2.7 Cobertura de provisión

```
Cobertura de provisión = Provisión constituida / Cartera vencida
```

**Interpretación.** Qué proporción de la cartera vencida está respaldada por provisión. Un valor bajo indica que el balance no refleja el deterioro real. Como ASC no tiene política formal de provisión, este indicador se calcula sobre la provisión **propuesta** por el equipo, y debe compararse contra la situación actual (sin provisión) para dimensionar el impacto en resultados.

### 2.8 Efectividad de cobranza (CEI — *Collection Effectiveness Index*)

```
CEI = (Saldo inicial + Ventas del período - Saldo final total)
      / (Saldo inicial + Ventas del período - Saldo final vigente) x 100
```

- Saldo inicial: cartera al inicio del mes.
- Ventas del período: ventas del mes.
- Saldo final total: cartera al cierre del mes.
- Saldo final vigente: porción de la cartera de cierre que aún no está vencida.

**Interpretación.** Mide qué proporción de lo que era cobrable en el mes efectivamente se cobró. A diferencia del DSO, no se distorsiona por la estacionalidad de las ventas, lo que lo hace mejor para evaluar la gestión mes a mes. Un CEI cercano a 100% indica cobranza eficaz; una caída sostenida durante 2025 es evidencia directa del deterioro que reporta la Gerencia.

**Desagregación exigida:** mensual.

### 2.9 Exposición sobre línea de crédito

```
Exposición = Saldo del cliente / Línea de crédito autorizada
```

- Saldo del cliente: suma de sus saldos insolutos al corte, en CLP.
- Línea de crédito: `linea_credito_clp` del maestro de clientes.

**Desagregación exigida:** por cliente.

**Interpretación.** Un valor mayor que 1 indica que el cliente está sobre su cupo autorizado. El hallazgo relevante no es solo la existencia del exceso, sino la **fecha de las facturas emitidas después de que el cupo fue superado**: eso convierte un problema de crédito en un problema de control interno.

---

## 3. Indicadores del benchmark sectorial (Componente B)

### Liquidez

| Indicador | Fórmula |
|---|---|
| Razón corriente | Activo corriente / Pasivo corriente |
| Razón ácida | (Activo corriente − Inventarios) / Pasivo corriente |
| Capital de trabajo neto | Activo corriente − Pasivo corriente |

En el sector acuícola, los **activos biológicos** pueden clasificarse como corrientes o no corrientes según el ciclo de cosecha. Verifique dónde quedaron antes de comparar razones de liquidez, y considere si corresponde tratarlos como inventario para la razón ácida.

### Endeudamiento

| Indicador | Fórmula |
|---|---|
| Razón de endeudamiento | Pasivo total / Patrimonio total |
| Endeudamiento financiero | Deuda financiera / Patrimonio total |
| Cobertura de gastos financieros | EBIT / Gastos financieros |

"Deuda financiera" no existe como elemento único en la taxonomía IFRS: es una construcción del analista. Declare cómo la compuso.

### Rentabilidad

| Indicador | Fórmula |
|---|---|
| Margen bruto | Ganancia bruta / Ingresos de actividades ordinarias |
| Margen operacional | Ganancia (pérdida) de operación / Ingresos |
| Margen neto | Ganancia (pérdida) / Ingresos |
| ROE | Ganancia atribuible a los propietarios / Patrimonio |
| ROA | Ganancia / Activos totales |

El efecto de **NIC 41** —valorización de activos biológicos a valor razonable— entra al resultado como ganancia o pérdida no realizada. Comparar el margen de una salmonicultora contra el de una pesquera extractiva sin aislar ese efecto compara dos cosas distintas.

### Eficiencia

| Indicador | Fórmula |
|---|---|
| Rotación de activos | Ingresos / Activos totales |
| **DSO sectorial** | (Deudores comerciales corrientes / Ingresos) × 365 |
| Rotación de inventarios | Costo de ventas / Inventarios |
| Días de inventario | 365 / Rotación de inventarios |
| Días de proveedores | (Cuentas por pagar comerciales / Costo de ventas) × 365 |
| Ciclo de conversión de efectivo | DSO + Días de inventario − Días de proveedores |

El **DSO sectorial** es el puente entre ambos componentes del caso. Note que se calcula sobre estados financieros anuales, con cuentas por cobrar de balance —que incluyen IVA en las empresas que venden en el mercado local— e ingresos sin IVA. Al comparar con el DSO de ASC, asegúrese de que ambos se construyeron con la misma convención, o explique la diferencia.

---

## 4. Estadígrafos para el benchmark

**Mediana.** Valor central del conjunto ordenado. Es el estadígrafo exigido para el benchmark sectorial, porque no se distorsiona con valores extremos.

**Cuartiles y rango intercuartílico (RIC).** Q1 es el valor bajo el cual está el 25% de las observaciones; Q3, el 75%. El RIC es `Q3 − Q1` y describe la dispersión del sector. La posición de ASC dentro o fuera del RIC es la conclusión que el Directorio necesita.

**Outlier.** Convención habitual: observación fuera del intervalo `[Q1 − 1,5·RIC ; Q3 + 1,5·RIC]`. Identificarlos es obligatorio; excluirlos es opcional y debe justificarse. Con cinco o seis empresas, excluir una altera sustancialmente la mediana: si lo hace, muestre el benchmark con y sin la exclusión.

**Por qué no el promedio.** Con pocas empresas y alta dispersión, un solo emisor con un ejercicio atípico —un evento sanitario, una venta extraordinaria de activos— arrastra el promedio y hace parecer anómalo a todo el resto.

---

## 5. Términos del caso

**Aging.** Clasificación de la cartera por antigüedad de la mora, en los tramos definidos en las reglas de negocio.

**Aforo o retención de factoring.** Porción del documento cedido que el factor no anticipa y que libera una vez que el deudor paga. En este caso es el 10% del monto exigible.

**Castigo tributario.** Baja del crédito incobrable con efecto en la determinación de la renta líquida imponible, conforme al artículo 31 N° 4 del DL 824, previa acreditación de haber agotado prudencialmente los medios de cobro.

**Diferencia temporaria.** Discrepancia entre la base contable y la base tributaria de un activo o pasivo, que revierte en ejercicios futuros y origina impuestos diferidos. La provisión IFRS 9 no aceptada tributariamente genera una diferencia temporaria deducible.

**Factoring con responsabilidad.** Cesión de facturas en la que el cedente responde ante el factor si el deudor no paga. La cartera sale del activo, pero la contingencia permanece y debe revelarse.

**Pérdida crediticia esperada (PCE).** Modelo de deterioro de IFRS 9. Para cuentas por cobrar comerciales sin componente financiero significativo se aplica el enfoque simplificado: se reconoce desde el inicio la pérdida esperada durante toda la vida del activo, típicamente mediante una matriz de provisión por antigüedad.

**Saldo insoluto.** Parte del monto exigible de un documento que aún no ha sido pagada al corte.
