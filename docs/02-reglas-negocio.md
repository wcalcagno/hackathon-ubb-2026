# 02 · Reglas de negocio — clasificación, provisión y castigo

Este documento fija lo que es **obligatorio** para todos los equipos, de modo que los resultados sean comparables entre sí, y separa explícitamente lo que queda entregado al **criterio profesional** de cada equipo.

Fecha de corte: **31-12-2025**.

---

## 1. Reglas obligatorias

### 1.1 Monto exigible de un documento

```
monto_exigible = monto_total - suma de notas de crédito del documento
```

En la moneda de emisión del documento.

### 1.2 Saldo insoluto

```
saldo_insoluto = monto_exigible - suma de pagos aplicados al documento
```

Reglas de borde:

- Si el resultado es menor o igual a cero, el documento está **cerrado**: saldo cero. No se registran saldos negativos.
- Se consideran únicamente los pagos con `fecha_pago <= 31-12-2025`.
- Los pagos huérfanos no reducen ninguna cartera.

### 1.3 Conversión a pesos

```
saldo_insoluto_clp = saldo_insoluto x tipo_cambio_clp(fecha_emision, moneda)
```

- Se usa el tipo de cambio observado de la **fecha de emisión del documento**.
- No se usa el tipo de cambio de cierre ni el de la fecha de pago. La diferencia entre ambos es diferencia de cambio, resultado financiero, y no debe alterar la valorización de la cartera comercial en este caso.
- Los importes en CLP se toman tal cual.

> El equipo puede, adicionalmente, mostrar el efecto de valorizar la cartera en USD al tipo de cambio de cierre. Es un análisis complementario valorado, pero la cartera oficial del caso se valoriza como indica esta regla.

### 1.4 Cartera exigible

Se excluyen de la cartera exigible:

- Documentos con `estado_sii = ANULADO`.
- Documentos con `estado_sii = RECLAMADO`.
- Registros identificados como duplicados de folio (se conserva uno solo por `tipo_documento` + `folio`; el criterio de selección lo define el equipo y debe justificarse).

Se **mantienen** en la cartera, con tratamiento explícito y separado:

- Documentos cedidos a factoring, por su saldo retenido.
- Documentos de clientes en `QUIEBRA` o `EN_COBRANZA_JUDICIAL`.

### 1.5 Días de mora

```
dias_mora = 31-12-2025 - fecha_vencimiento
```

- Se calcula solo para documentos con saldo insoluto mayor que cero.
- Un valor negativo significa que el documento aún no vence.
- Para los documentos con `fecha_vencimiento < fecha_emision` (error de digitación), el equipo debe **corregir o excluir**, y declarar el criterio. Calcular la mora sobre una fecha imposible infla artificialmente los tramos altos.

### 1.6 Tramos de antigüedad

| Tramo | Rango de `dias_mora` |
|---|---|
| Por vencer | menor que 0 |
| 1 a 30 | 0 a 30 |
| 31 a 60 | 31 a 60 |
| 61 a 90 | 61 a 90 |
| 91 a 180 | 91 a 180 |
| 181 a 365 | 181 a 365 |
| Más de 365 | mayor que 365 |

Los tramos son cerrados y no se superponen. Un documento con `dias_mora = 0` (vence justo el día del corte) pertenece al tramo "1 a 30".

### 1.7 Clasificación por recuperabilidad

Cada documento con saldo insoluto se clasifica en **una y solo una** categoría. Cuando se cumple más de un criterio, **prevalece el de mayor severidad**, evaluando en este orden:

**1. Incobrable** — si se cumple cualquiera de:
- `dias_mora > 365`, o
- cliente en estado `QUIEBRA`, o
- gestiones de cobranza agotadas (ver 1.8).

**2. Vencida en riesgo** — si no es incobrable y se cumple cualquiera de:
- `dias_mora` entre 181 y 365, o
- cliente con `clasificacion_riesgo = D`, o
- cliente en estado `EN_COBRANZA_JUDICIAL`.

**3. Vencida recuperable** — si no cae en las anteriores y:
- `dias_mora` entre 0 y 180, y cliente `ACTIVO` o `INACTIVO` sin cobranza judicial.

**4. Vigente** — si `dias_mora < 0`.

Consecuencia a tener presente: un documento **por vencer** de un cliente clasificado `D` no es "Vigente", es "Vencida en riesgo", porque el criterio de riesgo del cliente prevalece sobre la antigüedad. Esta asimetría es deliberada y debe manejarse correctamente en la medida DAX.

### 1.8 Gestiones de cobranza agotadas

Para efectos de este caso, se consideran agotados los medios de cobro de un documento cuando concurren copulativamente:

1. `dias_mora > 365` al corte;
2. existen al menos **tres** gestiones registradas sobre el documento; y
3. entre ellas hay al menos una de tipo `CARTA_CERTIFICADA`, `PREJUDICIAL` o `DEMANDA_JUDICIAL`.

El equipo puede proponer un criterio más exigente, justificándolo. No puede proponer uno más laxo.

---

## 2. Provisión por deterioro (IFRS 9, enfoque simplificado)

### 2.1 Qué se exige

Construir una **matriz de provisión por tramo de antigüedad** y aplicarla a la cartera exigible al corte. La matriz debe estar **fundamentada en el comportamiento observado en los datos 2024-2025**, no copiada de la tabla referencial del enunciado.

### 2.2 Método sugerido para derivar las tasas

Un camino defendible, entre otros:

1. Tomar el universo de documentos emitidos en 2024 y en el primer semestre de 2025, es decir, aquellos que ya tuvieron tiempo suficiente de maduración al corte.
2. Para cada documento, determinar el tramo de antigüedad que alcanzó (la mora máxima observada) y qué proporción de su monto exigible terminó recaudada.
3. Calcular, por tramo, la **tasa de pérdida observada** = 1 − (recaudado / exigible).
4. Ajustar por información prospectiva razonable: perspectiva del mercado exportador, tipo de cambio, situación específica de clientes relevantes. IFRS 9 exige incorporar información *forward-looking*; el ajuste debe ser explícito y justificado, no un número puesto a mano.
5. Contrastar el resultado con el rango referencial del enunciado y explicar las diferencias.

### 2.3 Segmentación

IFRS 9 admite agrupar los activos financieros por características de riesgo compartidas. Se valorará que el equipo evalúe si corresponde una matriz **única** o matrices **diferenciadas** por línea de negocio, segmento o moneda, y que fundamente la decisión con la evidencia de los datos (por ejemplo, comparando el comportamiento de pago de exportación contra foodservice).

### 2.4 Casos que la matriz por sí sola no resuelve

La matriz por antigüedad es el punto de partida, no el final. Deben tratarse de forma específica:

- Clientes en `QUIEBRA`: la provisión por antigüedad puede ser insuficiente. Un documento por vencer de un cliente quebrado no tiene 1% de pérdida esperada.
- Documentos cedidos a factoring: definir si el saldo retenido se provisiona, y qué tratamiento recibe la contingencia por el factoring con responsabilidad.
- Documentos en disputa (`resultado = DISPUTA` en las gestiones o `estado_sii = RECLAMADO`): el riesgo no es de crédito, es de validez del documento.

---

## 3. Castigo tributario (art. 31 N° 4, DL 824)

### 3.1 Marco

La Ley sobre Impuesto a la Renta permite deducir como gasto los créditos incobrables castigados durante el ejercicio, siempre que:

- se hayan **contabilizado oportunamente**, y
- se hayan **agotado prudencialmente los medios de cobro**.

La provisión financiera por pérdida crediticia esperada de IFRS 9, en cambio, **no es un gasto aceptado** mientras no se materialice el castigo del crédito. De ahí que ambos números no coincidan.

### 3.2 Qué debe hacer el equipo

1. Identificar los documentos que cumplen los requisitos para castigo tributario, aplicando el criterio de la sección 1.8 como piso.
2. Cuantificar el monto castigable.
3. Presentar la **conciliación** entre provisión financiera y castigo tributario, en un cuadro del tipo:

| Concepto | Monto CLP |
|---|---:|
| Provisión por deterioro constituida (IFRS 9) | |
| (−) Castigos que cumplen art. 31 N° 4 | |
| = Diferencia temporaria | |
| Impuesto diferido asociado (tasa vigente de primera categoría) | |

4. Indicar el tipo de diferencia (temporaria deducible), su efecto en el impuesto diferido y en qué momento se espera que revierta.

### 3.3 Advertencia

El caso es académico. Las conclusiones tributarias deben validarse contra el texto legal vigente y los pronunciamientos del SII. Si el equipo usa IA para redactar esta sección, la validación contra fuente oficial es obligatoria y el prompt debe declararse en el anexo.

---

## 4. Deducciones de caja y proyección de recaudación

### 4.1 Cascada obligatoria

```
Saldo bruto de cartera al 31-12-2025
  (-) Documentos en estado RECLAMADO o ANULADO
  (-) Notas de crédito pendientes de aplicar
  (-) Documentos cedidos a factoring
  (-) Provisión por deterioro esperado (IFRS 9)
  (-) Cartera clasificada como incobrable
  = Cartera neta recuperable
  x  Curva de recuperación histórica por tramo
  = Recaudación proyectada por mes
```

La cascada debe presentarse como un cuadro con el monto de cada línea, no solo como resultado final. El jurado necesita ver cuánto se pierde en cada escalón.

**Cuidado con la doble deducción:** la cartera incobrable normalmente ya está provisionada al 100%. Restar ambas líneas por separado sin conciliar duplica la deducción. Explique cómo evitó el doble conteo.

### 4.2 Curva de recuperación

Debe derivarse de los datos, no suponerse. Para cada tramo de antigüedad se requiere estimar:

- qué porcentaje del saldo termina recaudándose, y
- en cuántos días, en promedio, desde el corte.

Con ambos elementos se distribuye la cartera neta recuperable en los meses de enero a junio de 2026.

### 4.3 Presentación exigida

1. Proyección **mensual**, de enero a junio de 2026.
2. Apertura por **moneda** (CLP y USD), con el supuesto de tipo de cambio declarado explícitamente. Recuerde que la serie entregada termina el 31-12-2025.
3. **Tres escenarios** como mínimo: optimista, base y pesimista, variando la tasa de recuperación. Debe indicarse qué supuesto cambia en cada uno y en qué magnitud.
4. Un gráfico de la recaudación proyectada por mes y escenario.

---

## 5. Qué queda a criterio del equipo

Estas decisiones no están normadas. Cada una debe resolverse, documentarse y defenderse:

| Decisión | Preguntas que debe responder |
|---|---|
| Duplicados de folio | ¿Cuál de los dos registros es el válido? ¿Qué evidencia lo respalda? |
| Fechas de vencimiento imposibles | ¿Se corrigen usando la condición de pago pactada, o se excluyen? |
| Pagos huérfanos | ¿Se reportan y aíslan, o se intenta reconstruir su aplicación? |
| Factoring | ¿La cartera cedida sale del activo? ¿Cómo se revela la contingencia por responsabilidad? |
| Vencimiento vs. condición pactada | ¿Cuál prevalece para calcular la mora: lo pactado o lo registrado? |
| Notas de crédito de diciembre | ¿Pendientes de aplicar o ya aplicadas al corte? |
| Segmentación de la matriz IFRS 9 | ¿Una matriz o varias? ¿Con qué evidencia? |
| Ajuste prospectivo | ¿Qué información forward-looking se incorpora y con qué fundamento? |

> Criterio de evaluación: no hay una única respuesta correcta en esta tabla. Hay respuestas fundamentadas y respuestas arbitrarias. El jurado distingue entre ambas.
