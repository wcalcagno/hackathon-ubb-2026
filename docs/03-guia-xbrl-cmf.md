# 03 · Guía de XBRL de la CMF

Cómo descargar, leer y normalizar los estados financieros en formato XBRL publicados por la Comisión para el Mercado Financiero, para construir el benchmark sectorial del Componente B.

> Esta guía explica **cómo funciona el formato** y cómo pedirle a Lovable que construya la herramienta. No entrega la aplicación resuelta: especificarla correctamente es parte del desafío y de la rúbrica.

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

Obligatoria: es lo que permite al jurado verificar el análisis. Una fila por indicador y empresa:

| Indicador | Empresa | Elemento(s) de la taxonomía | Contexto usado | Unidad | Valor | Observación |
|---|---|---|---|---|---|---|
| DSO sectorial | Empresa A | `TradeAndOtherCurrentReceivables` / `Revenue` | `I-2025-12-31` / `D-2025` | USD | 62,4 | Línea única |
| DSO sectorial | Empresa B | `CurrentTradeReceivables` / `RevenueFromContractsWithCustomers` | ... | CLP | 48,1 | Desagregado; se toma solo comercial |

Sin esta tabla, el jurado no puede verificar el análisis y el componente se evalúa como no reproducible. Téngala a la vista durante la ronda de preguntas.

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

## 6. Construcción de la herramienta en Lovable

La herramienta de benchmark se construye en **Lovable**, describiendo en lenguaje natural la aplicación que se necesita. No hay que programar: hay que **especificar bien**.

Y ahí está el verdadero desafío del componente. Lovable escribe el código que usted le pida; si usted no sabe qué pedir —qué es un contexto XBRL, por qué hay que descartar los dimensionados, qué diferencia un instante de una duración— la aplicación va a producir números con toda confianza y todos equivocados. Las secciones 3, 4 y 5 de esta guía son las que permiten escribir el prompt correcto.

### 6.1 Cómo trabajar con Lovable

| Regla | Por qué |
|---|---|
| **Un cambio por prompt** | Si pide cinco cosas a la vez y algo sale mal, no sabrá cuál falló |
| **Pida ver la tabla intermedia** | Antes de calcular indicadores, exija ver los hechos extraídos. Si la extracción está mal, todo lo demás está mal |
| **Sea específico con los nombres** | Diga `ifrs-full:Assets`, no "los activos". El nombre exacto del elemento evita que la aplicación adivine |
| **Verifique contra el PDF** | Tome dos o tres cifras por empresa y compárelas con el estado financiero publicado. Es el único control que no se puede saltar |
| **No borre el historial** | El jurado puede pedirle los prompts. Lovable los conserva, y son la bitácora de cómo llegó al resultado |

> **Advertencia:** una aplicación que muestra un DSO de 47 días con un gráfico impecable no es evidencia de que el dato sea correcto. Si el equipo no puede explicar de qué contexto XBRL salió cada cifra, el componente se evalúa como no fundamentado, por bonita que sea la interfaz.

### 6.2 Secuencia de prompts sugerida

Adapte el texto: son un punto de partida, no un guion que copiar sin leer.

**Prompt 1 — Estructura de la aplicación**

```
Crea una aplicación web de una sola página para analizar estados financieros
en formato XBRL de empresas chilenas.

Necesito:
- Una zona donde subir varios archivos .xbrl o .xml (hasta 8 archivos).
- Para cada archivo subido, un campo de texto donde yo escriba el nombre de
  la empresa.
- Una tabla que liste los archivos cargados con su nombre de empresa.
- Todo el procesamiento en el navegador, sin backend.
- Interfaz en español, sobria, con tipografía legible.
```

**Prompt 2 — Extracción de contextos**

```
Al subir un archivo, parsea el XML y extrae todos los elementos <context>.

Para cada contexto genera una fila con:
- id_contexto: el atributo id
- entidad: el contenido de <identifier>
- tipo_periodo: "instante" si tiene <instant>, "duracion" si tiene
  <startDate> y <endDate>
- fecha_instante, fecha_inicio, fecha_fin
- tiene_dimensiones: verdadero si el contexto contiene <segment> o
  <scenario> con algún <explicitMember>

Muestra el resultado en una tabla que yo pueda revisar antes de seguir.
```

**Prompt 3 — Extracción de hechos**

```
Ahora extrae todos los hechos numéricos del archivo: cualquier elemento que
tenga el atributo contextRef.

Para cada hecho, una fila con:
- empresa
- elemento: el nombre de la etiqueta sin el prefijo del espacio de nombres
- contextRef
- unitRef
- decimals
- valor numérico

Cruza cada hecho con su contexto por contextRef y agrega las columnas del
contexto. Muestra el total de hechos extraídos por empresa y deja la tabla
descargable como CSV.

No filtres nada todavía: quiero la tabla completa.
```

**Prompt 4 — Filtro del contexto correcto**

```
Agrega un filtro, activado por defecto, que deje solo los hechos que cumplan
las tres condiciones:

1. tiene_dimensiones = falso (descarta segmentos y aperturas por dimensión)
2. para tipo_periodo = instante: fecha_instante igual a 2025-12-31
3. para tipo_periodo = duracion: fecha_inicio 2025-01-01 y fecha_fin
   2025-12-31

Muestra un contador de cuántos hechos quedaron dentro y cuántos se
descartaron, con el motivo de descarte. Quiero poder desactivar el filtro
para inspeccionar lo excluido.
```

**Prompt 5 — Tabla de mapeo editable**

```
Crea una tabla de mapeo editable que relacione cada concepto financiero con
el elemento de la taxonomía IFRS que lo representa, por empresa.

Conceptos: activos totales, activo corriente, pasivo corriente, pasivo total,
patrimonio total, inventarios, deudores comerciales corrientes, cuentas por
pagar comerciales, ingresos, costo de ventas, ganancia bruta, resultado
operacional, costos financieros y resultado del ejercicio.

Precarga los nombres habituales (por ejemplo Assets, CurrentAssets,
CurrentLiabilities, Equity, Inventories, TradeAndOtherCurrentReceivables,
Revenue, CostOfSales, GrossProfit, ProfitLoss, FinanceCosts) y permite
cambiarlos por empresa mediante un desplegable con los elementos que esa
empresa efectivamente reportó.

Si una empresa no reporta el elemento asignado, marca la celda en rojo y deja
el valor vacío. No la rellenes con el dato de otra empresa ni con cero.

La tabla debe poder exportarse a CSV y mostrarse al jurado.
```

**Prompt 6 — Moneda de presentación**

```
Detecta la moneda de presentación de cada empresa leyendo el unitRef de sus
hechos, y muéstrala en la tabla de empresas.

Agrega un campo donde yo ingrese el tipo de cambio USD/CLP y un selector de
moneda de análisis. Convierte solo las magnitudes absolutas; las razones y
porcentajes no se convierten.

Indica de forma visible qué cifras fueron convertidas y con qué tipo de
cambio.
```

**Prompt 7 — Indicadores**

```
Calcula por empresa, usando la tabla de mapeo:

Liquidez: razón corriente, razón ácida, capital de trabajo neto.
Endeudamiento: pasivo total / patrimonio, cobertura de gastos financieros
(resultado operacional / costos financieros).
Rentabilidad: margen bruto, margen operacional, margen neto, ROE, ROA.
Eficiencia: rotación de activos, rotación de inventarios, días de inventario,
días de proveedores, ciclo de conversión de efectivo y, sobre todo,
DSO = (deudores comerciales corrientes / ingresos) x 365.

Si falta algún insumo, muestra "sin dato" en vez de un número. Nunca estimes
ni interpoles un valor faltante.

Al pasar el cursor sobre cada indicador, muestra los dos valores que lo
componen y de qué elemento XBRL salió cada uno.
```

**Prompt 8 — Benchmark y visualización**

```
Calcula, para cada indicador, la mediana y los cuartiles Q1 y Q3 del conjunto
de empresas. Usa mediana, no promedio.

Marca como outlier toda observación fuera de [Q1 - 1,5*RIC ; Q3 + 1,5*RIC] y
permite excluirla del cálculo con un interruptor, mostrando el benchmark con
y sin la exclusión.

Grafica el DSO de todas las empresas en un gráfico de puntos sobre una línea,
con la banda del rango intercuartílico sombreada y la mediana marcada.
```

**Prompt 9 — Posicionamiento de ASC**

```
Agrega un campo donde yo ingrese manualmente los indicadores de una empresa
externa llamada Austral Seafood Chile SpA, calculados fuera de esta
aplicación: DSO, razón corriente y rotación de cuentas por cobrar.

Muéstrala en los gráficos con un color distinto y una etiqueta clara de que
no proviene de un archivo XBRL.

Debajo, indica en qué cuartil del sector queda ASC en cada indicador y cuántos
días de diferencia hay entre su DSO y la mediana sectorial.
```

**Prompt 10 — Evidencia exportable**

```
Agrega botones para exportar a CSV: la tabla completa de hechos extraídos, la
tabla de mapeo y la tabla de indicadores por empresa.

Agrega también un pie de página con la fecha de análisis, el tipo de cambio
utilizado y la lista de empresas con su moneda de presentación.
```

### 6.3 Prompts de verificación

Cuando la aplicación ya calcula, dedique al menos un ciclo a romperla. Esto vale más puntaje que una lámina extra:

```
Agrega un panel de control de calidad que verifique, por empresa:
- que activos totales sea igual a pasivos más patrimonio, con la diferencia
  en pesos si no cuadra;
- que activo corriente no supere los activos totales;
- que ganancia bruta sea igual a ingresos menos costo de ventas;
- que ninguna empresa quede con más del 20% de los conceptos sin dato.

Muestra cada control en verde o rojo, con el detalle numérico.
```

```
Muéstrame, para la empresa [NOMBRE], todos los hechos del elemento
[ELEMENTO], con su contexto completo y sin filtrar, para poder comparar
contra el estado financiero en PDF.
```

### 6.4 Qué debe poder mostrar

No hay entrega de archivos: todo se exhibe durante la exposición. A las 16:00 debe tener abierto y funcionando:

- La **aplicación en Lovable**, con su enlace público.
- La **tabla de mapeo**, que es donde se ve el criterio del equipo.
- La **tabla de hechos extraídos**, que es la evidencia de que la extracción es correcta.
- El **historial de prompts**, por si el jurado pregunta cómo lo construyó.

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
| Dar por buenas las cifras de la aplicación sin contrastarlas con el PDF | Un error de extracción se propaga a todo el benchmark, con apariencia de rigor |
| Usar estados individuales de una y consolidados de otra | Comparación inválida entre bases distintas |

---

## 8. Cierre: conectar con el Componente A

El resultado que importa de todo este ejercicio es el **DSO sectorial**:

```
DSO sectorial = (Deudores comerciales corrientes / Ingresos de actividades ordinarias) x 365
```

Calculado para cada empresa del peer group, permite construir la mediana y el rango intercuartílico del sector, y responder la pregunta del Directorio de ASC: dado el DSO calculado en el Componente A, ¿dónde se ubica ASC respecto de su industria, y qué parte de esa brecha es explicable por su mezcla de negocios y qué parte por su gestión de cobranza?

Esa última distinción —mezcla versus gestión— es lo que separa un análisis descriptivo de uno profesional.
