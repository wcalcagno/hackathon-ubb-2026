# 03 · Guía de XBRL de la CMF

Cómo descargar, leer y normalizar los estados financieros en formato XBRL publicados por la Comisión para el Mercado Financiero, para construir el benchmark sectorial del Componente B.

> Esta guía explica **cómo funciona el formato**. No entrega el parser resuelto: construirlo es parte del desafío y de la rúbrica.

---

## 1. Qué es XBRL y por qué importa aquí

XBRL (*eXtensible Business Reporting Language*) es un formato XML para reportar información financiera de manera estructurada. En lugar de una tabla de PDF donde "Activo corriente total" es texto, cada cifra es un **hecho** etiquetado con un concepto de una taxonomía estándar, un contexto (quién y cuándo) y una unidad.

Esto permite comparar empresas sin transcribir a mano. También permite equivocarse a escala: si el equipo toma el hecho con el contexto equivocado, obtendrá cifras del período comparativo, de un segmento, o de los estados individuales en vez de los consolidados, y el benchmark completo quedará mal.

---

## 2. Descarga desde la CMF

### 2.1 Ruta de navegación

```
https://www.cmfchile.cl
  > Mercados > Valores > Consulta por Fiscalizado > Emisores de Valores
    > [Buscar la empresa por razón social o RUT]
      > Estados Financieros
        > Estados Financieros Anuales > 31-12-2025
          > Formato XBRL
```

### 2.2 Qué descargar

La CMF entrega, según el emisor y el período, un archivo `.xbrl` (la instancia) o un `.zip` que contiene la instancia más los esquemas y bases de enlace (`.xsd`, `_cal.xml`, `_pre.xml`, `_lab.xml`, `_def.xml`).

Para calcular indicadores basta la **instancia**. Los archivos de enlace sirven para leer etiquetas en español (`_lab`), el orden de presentación (`_pre`) y las relaciones de suma (`_cal`), que son muy útiles para validar que los totales cuadran.

### 2.3 Dónde guardar

Guarde las descargas en `data/xbrl/`, una subcarpeta por empresa:

```
data/xbrl/
├── blumar/
│   └── <archivo descargado>.xbrl
├── camanchaca/
└── ...
```

Y registre en una bitácora, para cada empresa: **fecha de descarga, URL exacta, período, tipo de estado (consolidado o individual) y moneda de presentación**. Esa bitácora es evidencia de reproducibilidad y se evalúa.

### 2.4 Antes de parsear: verificación obligatoria

Para cada empresa del peer group, confirme en el sitio de la CMF:

- que mantiene su **condición de emisor vigente**;
- que efectivamente **publicó estados financieros al 31-12-2025**;
- la **razón social y RUT exactos** (no los asuma desde el enunciado del caso);
- si lo publicado corresponde a estados **consolidados** o **individuales**.

Si una empresa no publicó, documente el hecho, exclúyala y reemplácela por otra comparable, manteniendo el mínimo de cinco empresas del Grupo 1.

---

## 3. Anatomía de una instancia XBRL

Una instancia tiene tres bloques que hay que leer juntos.

### 3.1 Contextos: quién y cuándo

```xml
<context id="I-2025-12-31">
  <entity>
    <identifier scheme="http://www.cmfchile.cl">99999999-9</identifier>
  </entity>
  <period>
    <instant>2025-12-31</instant>
  </period>
</context>

<context id="D-2025">
  <entity>
    <identifier scheme="http://www.cmfchile.cl">99999999-9</identifier>
  </entity>
  <period>
    <startDate>2025-01-01</startDate>
    <endDate>2025-12-31</endDate>
  </period>
</context>
```

Dos tipos de período:

- **Instante** (`<instant>`): saldos de balance. Activos, pasivos, patrimonio, cuentas por cobrar, inventarios.
- **Duración** (`<startDate>`/`<endDate>`): flujos del período. Ingresos, costo de ventas, resultado, gastos financieros.

Cruzar ambos es el error más común: un indicador como el DSO combina un saldo (instante al 31-12-2025) con un flujo (duración del ejercicio 2025). Tomar el "activo corriente" de una duración no tiene sentido, y tomar los "ingresos" de un instante tampoco.

**Los identificadores de contexto (`id`) no están estandarizados.** Cada emisor y cada software generador usa su propia convención. No asuma el patrón: léalo del archivo.

### 3.2 Contextos con dimensiones: la trampa principal

```xml
<context id="D-2025-SegmentoSalmones">
  <entity>
    <identifier scheme="http://www.cmfchile.cl">99999999-9</identifier>
    <segment>
      <xbrldi:explicitMember dimension="ifrs-full:SegmentsAxis">...</xbrldi:explicitMember>
    </segment>
  </entity>
  <period><startDate>2025-01-01</startDate><endDate>2025-12-31</endDate></period>
</context>
```

Un contexto que contiene `<segment>` o `<scenario>` con `explicitMember` **no es la cifra consolidada total**: es la apertura por segmento, por clase de activo, por moneda o por cualquier otro eje.

Regla práctica: para los totales del estado financiero, use **solo contextos sin dimensiones**. Si suma los hechos dimensionados junto con el total, duplicará las cifras.

### 3.3 Unidades

```xml
<unit id="CLP"><measure>iso4217:CLP</measure></unit>
<unit id="USD"><measure>iso4217:USD</measure></unit>
```

Cada hecho numérico apunta a una unidad con `unitRef`. **En este sector conviven ambas monedas**: hay emisores que presentan en dólares y otros en pesos. Detectar la moneda de presentación leyendo el `unitRef` —y no suponiéndola— es requisito del caso.

### 3.4 Hechos

```xml
<ifrs-full:Assets contextRef="I-2025-12-31" unitRef="CLP" decimals="-3">1234567000</ifrs-full:Assets>
<ifrs-full:Revenue contextRef="D-2025" unitRef="CLP" decimals="-3">987654000</ifrs-full:Revenue>
```

Cada hecho tiene:

- el **concepto**, que es el nombre del elemento (`ifrs-full:Assets`);
- `contextRef`, que apunta a un contexto;
- `unitRef`, que apunta a una unidad;
- `decimals` o `precision`, que indican la exactitud declarada;
- el **valor**, que ya viene en la unidad de la moneda. `decimals="-3"` significa redondeo al millar, **no** que el valor esté expresado en miles. No multiplique ni divida por eso.

Ojo con `sign="-"` y con los elementos que por definición se reportan como positivos aunque representen egresos (por ejemplo, costo de ventas o gastos financieros). Antes de restar, verifique el signo real de cada emisor.

---

## 4. Elementos de la taxonomía IFRS relevantes

Nombres habituales en el espacio `ifrs-full`. **Verifique cada uno contra el archivo real**: no todos los emisores reportan los mismos elementos, y algunos usan extensiones propias.

### Estado de situación financiera (contextos de instante)

| Concepto | Elemento habitual |
|---|---|
| Activos totales | `Assets` |
| Activos corrientes | `CurrentAssets` |
| Efectivo y equivalentes | `CashAndCashEquivalents` |
| Deudores comerciales y otras cuentas por cobrar, corrientes | `TradeAndOtherCurrentReceivables` |
| Cuentas por cobrar comerciales, corrientes | `CurrentTradeReceivables` |
| Inventarios | `Inventories` |
| Activos biológicos corrientes | `CurrentBiologicalAssets` |
| Activos biológicos no corrientes | `NoncurrentBiologicalAssets` |
| Intangibles distintos de plusvalía | `IntangibleAssetsOtherThanGoodwill` |
| Pasivos totales | `Liabilities` |
| Pasivos corrientes | `CurrentLiabilities` |
| Pasivos no corrientes | `NoncurrentLiabilities` |
| Cuentas por pagar comerciales, corrientes | `TradeAndOtherCurrentPayables` |
| Otros pasivos financieros corrientes | `OtherCurrentFinancialLiabilities` |
| Otros pasivos financieros no corrientes | `OtherNoncurrentFinancialLiabilities` |
| Patrimonio total | `Equity` |
| Patrimonio atribuible a los propietarios de la controladora | `EquityAttributableToOwnersOfParent` |

### Estado de resultados (contextos de duración)

| Concepto | Elemento habitual |
|---|---|
| Ingresos de actividades ordinarias | `Revenue` o `RevenueFromContractsWithCustomers` |
| Costo de ventas | `CostOfSales` |
| Ganancia bruta | `GrossProfit` |
| Ganancia (pérdida) de actividades operacionales | `ProfitLossFromOperatingActivities` |
| Costos financieros | `FinanceCosts` |
| Ingresos financieros | `FinanceIncome` |
| Ganancia (pérdida) del ejercicio | `ProfitLoss` |
| Ganancia atribuible a los propietarios de la controladora | `ProfitLossAttributableToOwnersOfParent` |
| Depreciación y amortización | `DepreciationAndAmortisationExpense` |

### Diferencias de agregación que tendrá que resolver

- Una empresa reporta `TradeAndOtherCurrentReceivables` como línea única y otra la desagrega en `CurrentTradeReceivables` más otras cuentas. Si compara la primera contra la segunda sin homologar, el DSO sectorial quedará sesgado.
- `ProfitLossFromOperatingActivities` no siempre está presente. Puede ser necesario reconstruir el EBIT desde `GrossProfit` menos gastos de administración y distribución.
- "Deuda financiera" no es un elemento de la taxonomía: es una construcción. Defina si la arma con `OtherCurrentFinancialLiabilities` más `OtherNoncurrentFinancialLiabilities`, y declárelo en la tabla de mapeo.

---

## 5. Normalización: el punto crítico del componente

### 5.1 Moneda de presentación

1. Determine la moneda de cada emisor leyendo el `unitRef` de los hechos, no la portada del informe.
2. Defina **una** moneda de análisis y declárela.
3. Documente el tipo de cambio usado, su fuente y su fecha. Para saldos de balance corresponde el tipo de cambio de cierre; para flujos del período, un tipo de cambio promedio del ejercicio es más defendible que el de cierre. Si usa el mismo para todo, dígalo y explique por qué.

> Muchos indicadores son **razones entre dos cifras de la misma moneda** (margen bruto, razón corriente, ROE, DSO). Esos no requieren conversión, y convertirlos innecesariamente solo introduce error. Distinga los indicadores que sí requieren una moneda común —cualquier comparación de magnitudes absolutas— de los que no.

### 5.2 Tabla de mapeo

Entregable obligatorio del Componente B. Una fila por indicador y empresa:

| Indicador | Empresa | Elemento(s) de la taxonomía | Contexto usado | Unidad | Valor | Observación |
|---|---|---|---|---|---|---|
| DSO sectorial | Empresa A | `TradeAndOtherCurrentReceivables` / `Revenue` | `I-2025-12-31` / `D-2025` | USD | 62,4 | Línea única |
| DSO sectorial | Empresa B | `CurrentTradeReceivables` / `RevenueFromContractsWithCustomers` | ... | CLP | 48,1 | Desagregado; se toma solo comercial |

Sin esta tabla, el jurado no puede verificar el análisis y el componente se evalúa como no reproducible.

### 5.3 Controles de calidad recomendados

| Control | Qué verifica |
|---|---|
| `Assets = Liabilities + Equity` | Que tomó los contextos correctos y sin dimensiones. |
| `CurrentAssets <= Assets` | Que no mezcló corriente con total. |
| `GrossProfit = Revenue - CostOfSales` | Signos y homogeneidad del estado de resultados. |
| Orden de magnitud entre empresas | Que no confundió unidades ni escalas. |
| Comparación con el PDF publicado | Que la cifra extraída es la que la empresa reportó. |

Este último control es el más barato y el más efectivo: tome dos o tres cifras por empresa y compárelas contra el estado financiero en PDF del mismo período.

---

## 6. Implementación con Power Query

Un archivo `.xbrl` es un XML: Power Query lo lee de forma nativa, sin complementos ni programación. Todo el proceso queda dentro del `.pbix` o del libro de Excel, y se actualiza con un clic. Esa es la ruta del caso.

### 6.1 Punto de partida

En Power BI Desktop: `Obtener datos > XML`, y seleccione la instancia descargada. Si el conector no acepta la extensión `.xbrl`, renombre el archivo a `.xml`: el contenido es exactamente el mismo. También puede escribirlo directo en el editor avanzado:

```
= Xml.Tables(File.Contents("C:\ruta\instancia.xbrl"))
```

Power Query expande el XML en tablas anidadas, con columnas `Name`, `Value`, `Attribute:...` y tablas hijas que se despliegan con el botón de expandir.

### 6.2 Estrategia recomendada: dos consultas y un merge

**Consulta 1 — Contextos.** Una fila por `context`, con:

| Columna | De dónde sale |
|---|---|
| `id_contexto` | atributo `id` del elemento `context` |
| `entidad` | contenido de `identifier` |
| `tipo_periodo` | `instant` si tiene `instant`; `duration` si tiene `startDate`/`endDate` |
| `fecha_instante` / `fecha_inicio` / `fecha_fin` | los elementos del `period` |
| `tiene_dimensiones` | verdadero si el contexto contiene `segment` o `scenario` |

**Consulta 2 — Hechos.** Una fila por hecho numérico, con: `elemento` (el nombre de la etiqueta), `contextRef`, `unitRef`, `decimals` y `valor`.

**Merge.** Combine hechos con contextos por `contextRef` = `id_contexto`. Luego filtre:

- `tiene_dimensiones = false`;
- período correcto (instante 31-12-2025 para balance, duración 2025 completa para resultado);
- entidad correcta.

**Consolidación.** Agregue una columna `empresa` en cada consulta y use `Table.Combine` (o `Anexar consultas`) para unir las cinco o seis empresas en una sola **tabla larga**: `empresa`, `elemento`, `contexto`, `tipo_periodo`, `fecha`, `unidad`, `valor`.

### 6.3 Por qué una tabla larga y no una columna por indicador

Es tentador extraer solo los diez elementos que se necesitan. No lo haga: cada emisor reporta un conjunto distinto, y si filtra demasiado pronto descubrirá a mitad del análisis que a una empresa le falta justo el elemento que usó para las demás.

Con la tabla larga completa:

- el pivoteo hacia los indicadores se hace al final, con una tabla de mapeo como puente;
- se puede verificar qué reportó cada empresa y qué no;
- la tabla misma es la evidencia que el jurado necesita para validar el trabajo.

### 6.4 Dificultades típicas de Power Query con XBRL

| Situación | Cómo abordarla |
|---|---|
| Nombres de columna larguísimos por los espacios de nombres | Renombre en un paso propio y documente la equivalencia |
| Las columnas expandidas cambian de nombre entre emisores | No dependa de posiciones fijas: seleccione por nombre y agregue un paso de normalización por empresa |
| Los valores llegan como texto | Conviértalos con configuración regional **Inglés (Estados Unidos)**: el XBRL usa punto decimal |
| Rutas absolutas al archivo | Use un **parámetro** de carpeta, para que el `.pbix` funcione en otro equipo |
| Un emisor no trae un elemento | Deje el hueco visible en la tabla de mapeo; no lo rellene con el de otra empresa |

---

## 7. Errores frecuentes

| Error | Consecuencia |
|---|---|
| Tomar hechos de contextos con dimensiones | Se suman segmentos al total y las cifras se duplican |
| No filtrar el período comparativo (2024) | Se mezclan dos ejercicios en el mismo indicador |
| Confundir instante con duración | DSO, rotaciones y márgenes sin sentido económico |
| Ignorar `unitRef` | Se compara una empresa en USD contra otra en CLP |
| Interpretar `decimals="-3"` como escala | Cifras mil veces mayores o menores |
| Usar el promedio en vez de la mediana | Un outlier del sector distorsiona todo el benchmark |
| Comparar una pesquera extractiva con una salmonicultora sin ajustar | Márgenes no comparables por efecto de NIC 41 |
| Usar estados individuales de una y consolidados de otra | Comparación inválida entre bases distintas |

---

## 8. Cierre: conectar con el Componente A

El resultado que importa de todo este ejercicio es el **DSO sectorial**:

```
DSO sectorial = (Deudores comerciales corrientes / Ingresos de actividades ordinarias) x 365
```

Calculado para cada empresa del peer group, permite construir la mediana y el rango intercuartílico del sector, y responder la pregunta del Directorio de ASC: dado el DSO calculado en el Componente A, ¿dónde se ubica ASC respecto de su industria, y qué parte de esa brecha es explicable por su mezcla de negocios y qué parte por su gestión de cobranza?

Esa última distinción —mezcla versus gestión— es lo que separa un análisis descriptivo de uno profesional.
