# Hackathon de Comunidad UBB 2026

## Analítica de Datos para Contadores

**Facultad de Ciencias Empresariales (FACE) | Universidad del Bío-Bío, Sede Concepción**

Repositorio oficial del caso. Toda la información que los equipos necesitan para resolver el desafío está en este repositorio.

---

## Tabla de contenidos

1. [Contexto general](#1-contexto-general)
2. [Cómo partir](#2-cómo-partir)
3. [Estructura del repositorio](#3-estructura-del-repositorio)
4. [Componente A: Gestión de Cuentas por Cobrar](#4-componente-a-gestión-de-cuentas-por-cobrar)
5. [Componente B: Benchmark sectorial con XBRL desde la CMF](#5-componente-b-benchmark-sectorial-con-xbrl-desde-la-cmf)
6. [Entregables](#6-entregables)
7. [Rúbrica de evaluación](#7-rúbrica-de-evaluación)
8. [Herramientas y ambiente de trabajo](#8-herramientas-y-ambiente-de-trabajo)
9. [Preguntas frecuentes](#9-preguntas-frecuentes)

---

## 1. Contexto general

### 1.1 La empresa del caso

**Austral Seafood Chile SpA** (en adelante, ASC) es una empresa chilena de comercialización y exportación de productos del mar, con casa matriz en Talcahuano, Región del Biobío. Opera tres líneas de negocio:

| Línea de negocio | Descripción | Participación aproximada en ventas |
|---|---|---|
| Exportación | Venta de salmón, jurel congelado y harina de pescado a distribuidores en Asia, Europa, Norteamérica y Brasil | 62% |
| Foodservice nacional | Venta a cadenas de restaurantes, hoteles y casinos institucionales en Chile | 23% |
| Retail nacional | Venta a supermercados y distribuidores mayoristas chilenos | 15% |

ASC cerró el ejercicio 2025 con ingresos del orden de MM$ 48.000 y una cartera de 312 clientes activos. La empresa factura en pesos chilenos (CLP) y en dólares estadounidenses (USD), según el mercado de destino.

### 1.2 El problema

La Gerencia de Administración y Finanzas de ASC ha detectado una situación crítica: **el flujo de caja operacional se ha deteriorado durante 2025 pese a que las ventas crecieron**. El Directorio sospecha que el problema no está en el negocio, sino en la gestión de cobranza.

Los síntomas reportados son:

- El saldo de cuentas por cobrar creció más rápido que las ventas.
- Se identificaron facturas antiguas que nadie ha gestionado.
- El área comercial sigue vendiendo a clientes que no pagan.
- No existe una política formal de provisión por deterioro, ni criterios claros para castigar incobrables.
- La proyección de caja del último trimestre falló por un margen amplio.

### 1.3 El encargo

Su equipo ha sido contratado como asesor externo. Debe entregar a la Gerencia de Administración y Finanzas de ASC:

1. Un **diagnóstico cuantitativo** de la cartera de cuentas por cobrar al 31 de diciembre de 2025.
2. Una **clasificación de la cartera** por estado de recuperabilidad, con criterios técnicos justificados.
3. Una **proyección de caja** derivada de la cartera vigente.
4. Un **benchmark sectorial** que permita a ASC saber si su gestión de cobranza está por encima o por debajo del estándar de la industria pesquera y acuícola chilena.

Todo debe estar sustentado en un modelo de datos y visualizado en Power BI.

---

## 2. Cómo partir

1. **Descargue el repositorio** completo (botón `Code > Download ZIP`, o `git clone`).
2. **Lea `docs/01-diccionario-datos.md`** antes de conectar nada. Ahí están las convenciones de formato y las trampas conocidas de los archivos.
3. **Conecte Power BI Desktop** a la carpeta `data/raw/`. Los archivos son CSV con separador `;` y codificación UTF-8 con BOM. En el conector de texto, use:
   - Delimitador: punto y coma
   - Origen del archivo: `65001: Unicode (UTF-8)`
   - Configuración regional: **Inglés (Estados Unidos)** para que las fechas ISO (`AAAA-MM-DD`) y el separador decimal punto se interpreten correctamente.
4. **Contraste las filas cargadas** con el cuadro "Volumen de los archivos" de `docs/01-diccionario-datos.md`. Si un archivo carga con menos filas de las indicadas, la importación está mal configurada: revise el delimitador y la codificación antes de seguir.
5. **Revise `docs/02-reglas-negocio.md`**: contiene las reglas de clasificación y provisión que son obligatorias para todos los equipos. Lo que no está normado ahí es decisión del equipo, y debe justificarse.

> **Advertencia:** los datos están sucios a propósito. Hay duplicados, huérfanos, fechas imposibles y documentos que no corresponden a cartera exigible. Detectarlos y tratarlos forma parte de la evaluación.

---

## 3. Estructura del repositorio

```
hackathon-ubb-2026/
├── README.md                          # Este documento: el caso completo
├── docs/
│   ├── 01-diccionario-datos.md        # Diccionario de las tablas del Componente A
│   ├── 02-reglas-negocio.md           # Reglas de clasificación, provisión y castigo
│   ├── 03-guia-xbrl-cmf.md            # Cómo descargar y leer XBRL desde la CMF
│   └── 04-glosario-contable.md        # Definiciones de los indicadores exigidos
├── data/
│   ├── raw/
│   │   ├── fact_facturas.csv          # Documentos emitidos 2024-2025
│   │   ├── fact_cobranzas.csv         # Pagos y aplicaciones
│   │   ├── fact_notas_credito.csv     # Notas de crédito emitidas
│   │   ├── fact_gestiones_cobranza.csv# Bitácora de gestiones de cobranza
│   │   ├── dim_cliente.csv            # Maestro de clientes
│   │   ├── dim_vendedor.csv           # Maestro de fuerza de ventas
│   │   ├── dim_condicion_pago.csv     # Condiciones comerciales pactadas
│   │   ├── dim_moneda_tipo_cambio.csv # Tipos de cambio observados diarios
│   │   └── dim_calendario.csv         # Dimensión de fecha
│   └── xbrl/
│       └── README.md                  # Instrucciones de descarga desde la CMF
├── templates/
│   ├── plantilla-informe.md           # Estructura exigida del informe ejecutivo
│   ├── plantilla-informe.docx         # La misma estructura, lista para escribir
│   ├── plantilla-presentacion.md      # Guion sugerido de la defensa oral
│   ├── plantilla-presentacion.pptx    # Las 12 láminas, listas para completar
│   └── plantilla-anexo-prompts.md     # Formato del anexo obligatorio de uso de IA
└── entregas/
    └── README.md                      # Instrucciones de entrega por equipo
```

---

## 4. Componente A: Gestión de Cuentas por Cobrar

### 4.1 Modelo de datos entregado

Los datos se entregan en formato CSV, con separador punto y coma y codificación UTF-8. Corresponden a un esquema transaccional sin modelar. **Parte del desafío consiste en construir el modelo dimensional correcto.**

El detalle campo por campo está en [`docs/01-diccionario-datos.md`](docs/01-diccionario-datos.md). A continuación, el resumen.

#### 4.1.1 `fact_facturas.csv`

Documentos tributarios emitidos entre el 01-01-2024 y el 31-12-2025.

| Campo | Tipo | Descripción |
|---|---|---|
| `id_documento` | texto | Identificador único del documento |
| `tipo_documento` | texto | FAC (factura afecta), FEX (factura de exportación), FEE (factura exenta) |
| `folio` | entero | Folio del documento tributario |
| `id_cliente` | texto | Llave foránea al maestro de clientes |
| `id_vendedor` | texto | Llave foránea al maestro de vendedores |
| `id_condicion` | texto | Llave foránea a la condición de pago pactada |
| `linea_negocio` | texto | EXPORTACION, FOODSERVICE, RETAIL |
| `fecha_emision` | fecha | Fecha de emisión del documento |
| `fecha_vencimiento` | fecha | Fecha de vencimiento registrada en el sistema |
| `moneda` | texto | CLP, USD |
| `monto_neto` | decimal | Monto neto en moneda de emisión |
| `monto_iva` | decimal | IVA (cero para FEX y FEE) |
| `monto_total` | decimal | Total documento en moneda de emisión |
| `estado_sii` | texto | ACEPTADO, RECLAMADO, ANULADO |
| `centro_costo` | texto | Planta o unidad de origen |

**Advertencias sobre la calidad de los datos:**

- Existen documentos con `estado_sii = RECLAMADO` que **no deben considerarse cartera exigible**.
- Existen documentos con `fecha_vencimiento` anterior a `fecha_emision`. Son errores de digitación que el equipo debe detectar y tratar.
- Algunos folios están duplicados. Se debe determinar cuál es el registro válido.
- Las facturas en USD deben convertirse a CLP usando el tipo de cambio observado de la fecha de emisión, no el de cierre.
- La `fecha_vencimiento` registrada no siempre coincide con la condición de pago pactada en `id_condicion`.

#### 4.1.2 `fact_cobranzas.csv`

| Campo | Tipo | Descripción |
|---|---|---|
| `id_cobranza` | texto | Identificador único del pago |
| `id_documento` | texto | Documento al que se aplica el pago |
| `fecha_pago` | fecha | Fecha efectiva de recaudación |
| `moneda` | texto | CLP, USD |
| `monto_pagado` | decimal | Monto aplicado al documento |
| `medio_pago` | texto | TRANSFERENCIA, CHEQUE, FACTORING, COMPENSACION |
| `id_operacion_factoring` | texto | Vacío salvo que `medio_pago = FACTORING` |

**Consideraciones:**

- Un documento puede tener **pagos parciales múltiples**. El saldo insoluto es la diferencia entre el total del documento y la suma de pagos aplicados, neta de notas de crédito.
- Los pagos con `medio_pago = FACTORING` corresponden a cesión de facturas, con anticipo parcial y retención (aforo). **Estos documentos ya no representan un derecho de cobro pleno de ASC**, pero sí generan una obligación contingente si el cliente no paga al factor (factoring con responsabilidad). El equipo debe decidir y justificar su tratamiento.
- Existen pagos aplicados a documentos que no existen en `fact_facturas`. Son huérfanos que deben reportarse.

#### 4.1.3 `fact_notas_credito.csv`

| Campo | Tipo | Descripción |
|---|---|---|
| `id_nota_credito` | texto | Identificador único |
| `folio_nc` | entero | Folio de la nota de crédito |
| `id_documento_referencia` | texto | Documento que rebaja |
| `fecha_emision` | fecha | Fecha de emisión |
| `motivo` | texto | DEVOLUCION, DESCUENTO, ANULACION, AJUSTE_CALIDAD |
| `moneda` | texto | Moneda de la nota, igual a la del documento referenciado |
| `monto_total` | decimal | Monto de la nota de crédito |

#### 4.1.4 `fact_gestiones_cobranza.csv`

Bitácora de las acciones de cobranza registradas por los ejecutivos.

| Campo | Tipo | Descripción |
|---|---|---|
| `id_gestion` | texto | Identificador único |
| `id_documento` | texto | Documento gestionado |
| `fecha_gestion` | fecha | Fecha de la acción |
| `tipo_gestion` | texto | LLAMADA, EMAIL, CARTA_CERTIFICADA, VISITA, PREJUDICIAL, DEMANDA_JUDICIAL |
| `resultado` | texto | COMPROMISO_PAGO, SIN_RESPUESTA, DISPUTA, PAGO_PARCIAL, INCOBRABLE_DECLARADO |
| `ejecutivo_cobranza` | texto | Responsable de la gestión |

Esta tabla es la evidencia sobre la cual se acredita el agotamiento prudencial de los medios de cobro exigido por el artículo 31 N° 4 de la Ley sobre Impuesto a la Renta. **Hay documentos morosos sin ninguna gestión registrada.**

#### 4.1.5 `dim_cliente.csv`

| Campo | Tipo | Descripción |
|---|---|---|
| `id_cliente` | texto | Identificador único |
| `razon_social` | texto | Nombre del cliente |
| `rut` | texto | RUT (vacío para clientes extranjeros) |
| `pais` | texto | País de destino |
| `segmento` | texto | EXPORTADOR, CADENA_RETAIL, FOODSERVICE, MAYORISTA |
| `fecha_alta` | fecha | Fecha de incorporación como cliente |
| `linea_credito_clp` | decimal | Cupo de crédito autorizado |
| `clasificacion_riesgo` | texto | A, B, C, D (evaluación comercial interna) |
| `estado` | texto | ACTIVO, INACTIVO, EN_COBRANZA_JUDICIAL, QUIEBRA |
| `ejecutivo_cobranza` | texto | Responsable asignado |

#### 4.1.6 `dim_condicion_pago.csv`

| Campo | Tipo | Descripción |
|---|---|---|
| `id_condicion` | texto | Identificador |
| `descripcion` | texto | Contado, 30 días, 45 días, 60 días, 90 días, 120 días |
| `dias_credito` | entero | Días de plazo pactado |

#### 4.1.7 `dim_vendedor.csv`, `dim_moneda_tipo_cambio.csv` y `dim_calendario.csv`

Maestro de la fuerza de ventas, serie diaria del dólar observado (2024-2025) y dimensión de fecha con marca de día hábil y feriado. Detalle en el diccionario de datos.

### 4.2 Reglas de negocio obligatorias

Las reglas completas, con sus casos de borde, están en [`docs/02-reglas-negocio.md`](docs/02-reglas-negocio.md). Síntesis:

#### 4.2.1 Cálculo de días de mora

```
dias_mora = fecha_corte - fecha_vencimiento
```

Donde `fecha_corte = 31-12-2025`. Se calcula solo sobre el **saldo insoluto**, no sobre el total del documento.

#### 4.2.2 Tramos de antigüedad (aging)

| Tramo | Rango de días de mora |
|---|---|
| Por vencer | dias_mora < 0 |
| 1 a 30 | 0 a 30 |
| 31 a 60 | 31 a 60 |
| 61 a 90 | 61 a 90 |
| 91 a 180 | 91 a 180 |
| 181 a 365 | 181 a 365 |
| Más de 365 | mayor a 365 |

#### 4.2.3 Clasificación por recuperabilidad

Cada documento con saldo insoluto debe clasificarse en **una y solo una** de las siguientes categorías:

| Categoría | Criterio |
|---|---|
| **Vigente** | `dias_mora < 0`. Cartera por vencer, dentro del plazo pactado. |
| **Vencida recuperable** | `dias_mora` entre 0 y 180, cliente en estado ACTIVO, sin cobranza judicial. |
| **Vencida en riesgo** | `dias_mora` entre 181 y 365, o cliente con clasificación de riesgo D, o cliente en estado EN_COBRANZA_JUDICIAL. |
| **Incobrable** | `dias_mora > 365`, o cliente en estado QUIEBRA, o documento con gestiones de cobranza agotadas. |

**Nota crítica:** la clasificación debe ser mutuamente excluyente. Si un documento cumple más de un criterio, prevalece la categoría de mayor severidad.

#### 4.2.4 Matriz de provisión por deterioro

ASC no tiene política formal. El equipo debe **proponer y justificar** una matriz de provisión bajo el enfoque simplificado de pérdida crediticia esperada de IFRS 9 (NIIF 9), aplicable a cuentas por cobrar comerciales.

Como referencia orientadora, una matriz típica del sector podría estructurarse así, pero **el equipo debe fundamentar sus propios porcentajes** a partir del comportamiento histórico observado en los datos 2024-2025:

| Tramo | Porcentaje de provisión sugerido |
|---|---|
| Por vencer | 0,5% a 1% |
| 1 a 30 | 2% a 5% |
| 31 a 60 | 5% a 10% |
| 61 a 90 | 10% a 20% |
| 91 a 180 | 25% a 40% |
| 181 a 365 | 50% a 75% |
| Más de 365 | 100% |

**Se valorará especialmente** que el equipo derive los porcentajes desde la tasa de recuperación histórica observable en los propios datos, en lugar de aplicar la tabla referencial sin análisis.

#### 4.2.5 Criterio tributario de castigo

Adicionalmente al tratamiento financiero, el equipo debe identificar qué documentos cumplen los requisitos para ser **castigados tributariamente** conforme al artículo 31 N° 4 de la Ley sobre Impuesto a la Renta (DL 824), que exige haber agotado prudencialmente los medios de cobro.

Se debe presentar la **conciliación entre la provisión financiera y el castigo tributario**, identificando la diferencia temporaria resultante.

### 4.3 Deducciones de caja y proyección de recaudación

Este es el componente de mayor complejidad del caso.

A partir de la cartera clasificada, el equipo debe construir una **proyección de recaudación mensual para el primer semestre 2026**, aplicando las siguientes deducciones sucesivas sobre el saldo bruto de cartera:

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

**Requisitos de la proyección:**

1. La **curva de recuperación** debe derivarse de los datos históricos: para cada tramo de antigüedad, calcular qué porcentaje de la cartera efectivamente se recaudó y en cuántos días promedio, observando el comportamiento 2024-2025.
2. La proyección debe presentarse **mes a mes**, de enero a junio 2026.
3. Debe distinguirse recaudación en CLP y en USD, con supuesto explícito de tipo de cambio. La serie de tipo de cambio entregada llega hasta el 31-12-2025: el valor proyectado para 2026 es un supuesto del equipo y debe declararse.
4. Debe incluirse un **análisis de sensibilidad** con al menos tres escenarios (optimista, base, pesimista), variando la tasa de recuperación.

### 4.4 Indicadores obligatorios

El tablero debe calcular y presentar, como mínimo:

| Indicador | Fórmula | Nivel de desagregación exigido |
|---|---|---|
| **DSO** (Days Sales Outstanding) | (Cuentas por cobrar / Ventas del período) x días del período | Total, por línea de negocio, por segmento |
| **DSO ajustado** | Igual, pero excluyendo cartera incobrable y factorizada | Total |
| **Índice de morosidad** | Cartera vencida / Cartera total | Total, por línea de negocio, por vendedor |
| **Tasa de incobrabilidad** | Cartera incobrable / Ventas del período | Total, por segmento |
| **Índice de concentración** | Participación de los 10 mayores deudores en la cartera total | Total |
| **Rotación de cuentas por cobrar** | Ventas a crédito / Cuentas por cobrar promedio | Total, por línea de negocio |
| **Cobertura de provisión** | Provisión constituida / Cartera vencida | Total |
| **Efectividad de cobranza (CEI)** | (Saldo inicial + Ventas - Saldo final) / (Saldo inicial + Ventas - Saldo final vigente) x 100 | Mensual |
| **Exposición sobre línea de crédito** | Saldo del cliente / Línea de crédito autorizada | Por cliente |

Las definiciones operativas de cada indicador están en [`docs/04-glosario-contable.md`](docs/04-glosario-contable.md).

### 4.5 Hallazgos que el modelo debe permitir detectar

El diseño del caso incorpora situaciones deliberadas. Un equipo con un análisis riguroso debería ser capaz de identificar:

1. **Clientes que exceden su línea de crédito** y a los que se les sigue facturando.
2. **Concentración excesiva de cartera** en pocos deudores de exportación.
3. Al menos un **vendedor con una cartera sistemáticamente más morosa** que el promedio, lo que sugiere revisar sus criterios de otorgamiento de crédito.
4. **Documentos antiguos sin ninguna gestión de cobranza registrada.**
5. **Notas de crédito por AJUSTE_CALIDAD concentradas en un cliente o línea específica**, lo que apunta a un problema operacional, no de cobranza.
6. **Inconsistencias entre la condición de pago pactada y el vencimiento efectivamente registrado** en los documentos.
7. **Clientes en estado QUIEBRA con cartera aún no provisionada.**

### 4.6 Recomendaciones de control interno

El informe debe cerrar con al menos **cinco propuestas de mejora de control interno**, derivadas directamente de los hallazgos cuantitativos. Cada propuesta debe indicar:

- Debilidad detectada (con evidencia numérica del análisis).
- Riesgo asociado.
- Control propuesto.
- Responsable sugerido.
- Indicador de seguimiento.

---

## 5. Componente B: Benchmark sectorial con XBRL desde la CMF

### 5.1 Objetivo

Construir una herramienta que permita comparar los estados financieros al **31 de diciembre de 2025** de un conjunto de empresas del sector pesquero y acuícola chileno, a partir de la información en formato XBRL publicada por la Comisión para el Mercado Financiero (CMF), y posicionar a ASC frente al estándar de la industria.

### 5.2 Criterio de construcción del peer group

El primer juicio profesional que exige este componente es la **definición del grupo comparable**. Un peer group mal construido invalida cualquier benchmark, por bien calculados que estén los indicadores.

El criterio aplicado en este caso es el siguiente: se consideran comparables únicamente las **empresas cuyo negocio principal es la pesca extractiva o la acuicultura**, que se encuentren inscritas en el Registro de Valores de la CMF y que publiquen estados financieros bajo IFRS.

Quedan **excluidos los conglomerados diversificados** cuya exposición al sector es indirecta o minoritaria. Un holding cuyos ingresos están dominados por energía, forestal o retail no es comparable con una pesquera, aunque participe en el sector a través de filiales: sus márgenes, su estructura de activos y su ciclo de capital de trabajo responden a otra realidad económica.

El equipo debe justificar por escrito su criterio de selección y declarar explícitamente qué empresas descartó y por qué.

### 5.3 Universo de empresas a analizar

El benchmark se construye sobre empresas **fiscalizadas por la CMF** que están obligadas a publicar estados financieros bajo IFRS.

#### Grupo 1: Comparables directos del sector pesquero y acuícola

| Empresa | Nemotécnico | Perfil de negocio |
|---|---|---|
| **Blumar S.A.** | BLUMAR | Pesca industrial y acuicultura. Operaciones en Caldera, Talcahuano, Coronel y Corral. Salmón atlántico, jurel, harina y aceite de pescado, choritos. |
| **Camanchaca S.A.** | CAMANCHACA | Pesca industrial y acuicultura. Harina de pescado, jurel congelado y en conserva, langostino, salmón, abalones, mejillones. |
| **Salmones Camanchaca S.A.** | SALMOCAM | Filial de Camanchaca dedicada exclusivamente al cultivo de salmón. |
| **Multiexport Foods S.A.** | MULTIFOODS | Holding acuícola, matriz de Multi X. Producción y exportación de salmón. |
| **Australis Seafoods S.A.** | AUSTRALIS | Producción y exportación de salmón atlántico y coho. |
| **Invermar S.A.** | INVERMAR | Acuicultura, cultivo de salmón y mejillones. |

#### Grupo 2: Comparable de contexto (opcional)

| Empresa | Nemotécnico | Justificación |
|---|---|---|
| **Grupo Empresas Navieras S.A.** | NAVIERA | Logística marítima y portuaria. No pertenece al sector pesquero, pero comparte exposición al ciclo exportador y a la infraestructura portuaria del Biobío. Su inclusión debe justificarse y no puede computar dentro del mínimo exigido. |

**Requisito mínimo:** el análisis debe incluir **al menos cinco (5) empresas del Grupo 1**. El Grupo 2 es complementario y su inclusión es optativa.

**Verificación obligatoria:** antes de construir el análisis, el equipo debe confirmar en el sitio de la CMF que cada empresa seleccionada mantiene su condición de emisor vigente y que efectivamente publicó estados financieros al 31-12-2025. Las razones sociales, RUT y estados de vigencia deben validarse en la fuente, no asumirse desde este documento.

### 5.4 Fuente de datos

Portal de la Comisión para el Mercado Financiero:

```
https://www.cmfchile.cl
```

Ruta de navegación sugerida:

```
Mercados > Valores > Consulta por Fiscalizado > Emisores de Valores
  > [Seleccionar empresa] > Estados Financieros
    > Estados Financieros Anuales > 31-12-2025 > Formato XBRL
```

La CMF publica los estados financieros bajo la **taxonomía IFRS**, en archivos con extensión `.xbrl` o `.zip` que contienen la instancia XBRL más los esquemas asociados.

La guía paso a paso, con la anatomía de una instancia XBRL y los elementos de taxonomía relevantes, está en [`docs/03-guia-xbrl-cmf.md`](docs/03-guia-xbrl-cmf.md). Las descargas se guardan en `data/xbrl/`.

### 5.5 Requisitos técnicos de la herramienta

El equipo debe construir un proceso reproducible que cumpla:

#### 5.5.1 Extracción

- Descargar las instancias XBRL de las empresas seleccionadas al 31-12-2025.
- Parsear los archivos identificando los elementos de la taxonomía IFRS relevantes.
- El parseo se resuelve construyendo una herramienta en **Lovable**, describiendo en lenguaje natural la aplicación que se necesita. No hay que programar: hay que especificar bien, y para eso el equipo debe entender la estructura del formato. La secuencia de prompts sugerida está en [`docs/03-guia-xbrl-cmf.md`](docs/03-guia-xbrl-cmf.md).

#### 5.5.2 Normalización

Este es el punto crítico del componente. Las empresas **no reportan de forma homogénea**. El equipo debe resolver:

- **Diferencias de moneda de presentación:** algunas empresas del sector reportan en USD y otras en CLP. Debe definirse una moneda única de análisis y documentar el tipo de cambio utilizado.
- **Diferencias de agregación:** una empresa puede reportar "Deudores comerciales y otras cuentas por cobrar corrientes" como una sola línea, y otra puede desagregarla. Se requiere un mapeo explícito.
- **Contextos XBRL:** cada hecho reportado tiene un contexto (entidad, período, dimensiones). Debe filtrarse correctamente el contexto consolidado del período anual, descartando segmentos y períodos comparativos.
- **Signos y unidades:** verificar el atributo `unitRef` y el signo de los elementos, especialmente en pasivos y en flujos de efectivo.

Se debe entregar una **tabla de mapeo** que documente, para cada indicador construido, qué elemento de la taxonomía IFRS se usó en cada empresa.

#### 5.5.3 Cálculo de indicadores

| Categoría | Indicador | Fórmula |
|---|---|---|
| **Liquidez** | Razón corriente | Activo corriente / Pasivo corriente |
| | Razón ácida | (Activo corriente - Inventarios) / Pasivo corriente |
| | Capital de trabajo neto | Activo corriente - Pasivo corriente |
| **Endeudamiento** | Razón de endeudamiento | Pasivo total / Patrimonio total |
| | Endeudamiento financiero | Deuda financiera / Patrimonio total |
| | Cobertura de gastos financieros | EBIT / Gastos financieros |
| **Rentabilidad** | Margen bruto | Ganancia bruta / Ingresos de actividades ordinarias |
| | Margen operacional | Ganancia (pérdida) de operación / Ingresos |
| | Margen neto | Ganancia (pérdida) / Ingresos |
| | ROE | Ganancia atribuible a los propietarios / Patrimonio |
| | ROA | Ganancia / Activos totales |
| **Eficiencia** | Rotación de activos | Ingresos / Activos totales |
| | **DSO sectorial** | (Deudores comerciales corrientes / Ingresos) x 365 |
| | Rotación de inventarios | Costo de ventas / Inventarios |
| | Ciclo de conversión de efectivo | DSO + Días de inventario - Días de proveedores |

**El DSO sectorial es el indicador que conecta ambos componentes del caso.** Es la métrica que permite responder la pregunta del Directorio de ASC: ¿nuestra gestión de cobranza está por encima o por debajo de la industria?

#### 5.5.4 Visualización

- Posicionamiento de cada empresa frente a la **mediana del sector** (no el promedio, dado que el sector presenta alta dispersión y valores atípicos).
- Identificación explícita de outliers y justificación de si se excluyen del cálculo del benchmark.
- Comparación de ASC contra el rango intercuartílico del sector en DSO y en indicadores de liquidez.

### 5.6 Consideraciones sectoriales que deben incorporarse al análisis

El sector pesquero y acuícola chileno tiene particularidades contables que un análisis superficial ignora y que el jurado valorará si el equipo las identifica:

1. **NIC 41 (Activos biológicos):** los peces en engorda se valorizan a valor razonable menos costos de venta. Esto genera resultados no realizados que distorsionan la comparación de márgenes entre empresas pesqueras puras y acuícolas.
2. **Estacionalidad de cosecha:** el ciclo productivo del salmón supera los 24 meses, lo que afecta la comparabilidad de la rotación de inventarios.
3. **Exposición cambiaria:** empresas con ingresos en USD y costos en CLP presentan estructuras de resultado distintas.
4. **Cuotas de pesca:** las licencias transables de pesca se reconocen como activos intangibles y afectan la base de activos totales.
5. **Eventos sanitarios:** episodios de floración de algas nocivas o brotes sanitarios generan castigos extraordinarios que deben identificarse en las notas antes de comparar.

Un análisis que compare márgenes de una pesquera extractiva contra una salmonicultora sin ajustar por estos factores será considerado técnicamente débil.

---

## 6. Entregables

Cada equipo entrega, al cierre del bloque técnico de la jornada, una carpeta comprimida con:

| # | Entregable | Formato | Requisito |
|---|---|---|---|
| 1 | Modelo y tablero de Cuentas por Cobrar | `.pbix` o carpeta PBIP | Modelo dimensional, medidas DAX documentadas |
| 2 | Herramienta de benchmark XBRL | Enlace público a la aplicación en Lovable | Funcionando, con tabla de mapeo y tabla de hechos exportadas |
| 3 | Informe ejecutivo | PDF, máximo 10 páginas | Portada, resumen ejecutivo, metodología, hallazgos A, hallazgos B, conclusiones, anexo de prompts de IA |
| 4 | Presentación oral | `.pptx` o equivalente, máximo 12 láminas | Para defensa ante jurado |
| 5 | Extensión del prototipo (opcional) | Enlace público | Funcionalidad adicional sobre la aplicación del entregable 2: por ejemplo, incorporar la cartera de ASC, simular escenarios o publicar el tablero de hallazgos |

Las plantillas de los entregables 3 y 4 están en [`templates/`](templates/). Las instrucciones de entrega, en [`entregas/README.md`](entregas/README.md).

**Presentación oral:** 12 minutos por equipo, seguidos de 5 minutos de preguntas del jurado.

---

## 7. Rúbrica de evaluación

| Criterio | Ponderación | Qué se evalúa |
|---|---|---|
| Rigurosidad técnica del análisis contable y financiero | 25% | Corrección de los cálculos, fundamentación de la matriz de provisión, conciliación financiero-tributaria, tratamiento de las particularidades sectoriales |
| Calidad del modelo de datos y dominio de las herramientas | 25% | Modelo dimensional correcto, calidad de las medidas DAX, tratamiento de la calidad de datos, robustez del parseo XBRL |
| Capacidad de comunicación y narrativa de los hallazgos | 20% | Claridad del tablero, coherencia del relato, orientación a la toma de decisiones |
| Aplicabilidad y propuestas accionables | 15% | Pertinencia de las recomendaciones de control interno, viabilidad de implementación |
| Innovación y uso complementario de IA o herramientas modernas | 15% | Calidad de los prompts con que se construyó la herramienta del Componente B, uso declarado y validado de IA generativa, soluciones creativas al problema de normalización XBRL |

**Criterio de desempate:** entrega de la extensión opcional del prototipo y profundidad en el tratamiento de las particularidades sectoriales descritas en la sección 5.6.

---

## 8. Herramientas y ambiente de trabajo

### 8.1 Herramientas cubiertas en las sesiones previas

| Sesión | Herramienta | Contenido |
|---|---|---|
| Sesión 1 | **Power BI** | Conexión a fuentes, modelado dimensional, DAX fundamental, diseño de tableros para análisis contable |
| Sesión 2 | **Lovable** | Construcción de prototipos web desde lenguaje natural, integración de visualizaciones, despliegue |

### 8.2 Herramientas permitidas

- Power BI Desktop (obligatorio para el Componente A).
- Excel y Power Query.
- Herramientas de IA generativa, con declaración obligatoria de los prompts utilizados.
- Lovable, obligatorio para construir la herramienta del Componente B.

### 8.3 Requisitos técnicos

Cada equipo debe traer sus propios equipos portátiles con Power BI Desktop instalado y actualizado. La FACE UBB provee conexión a internet, salas habilitadas y proyectores.

### 8.4 Uso ético de la inteligencia artificial

- La IA generativa es una herramienta de apoyo, no un reemplazo del criterio profesional.
- Toda salida generada por IA debe validarse contra normativa vigente o fuentes oficiales (IFRS, SII, CMF).
- Los prompts utilizados deben documentarse en un anexo del informe, según [`templates/plantilla-anexo-prompts.md`](templates/plantilla-anexo-prompts.md).
- No se permite asistencia externa de docentes ni profesionales durante la jornada.

---

## 9. Preguntas frecuentes

**¿Puedo modificar los datos entregados?**
Puede transformarlos, limpiarlos y enriquecerlos, documentando cada decisión. No puede alterar los valores originales para forzar un resultado.

**¿Qué pasa si una empresa del universo no publicó XBRL al 31-12-2025?**
Debe documentarlo, excluirla del análisis y reemplazarla por otra comparable, justificando el criterio. Mantener el mínimo de cinco empresas del Grupo 1.

**¿Debo resolver ambos componentes?**
Sí. Ambos son obligatorios y están conectados: el DSO sectorial del Componente B es el punto de referencia contra el cual se evalúa el DSO de ASC calculado en el Componente A.

**¿Puedo usar plantillas o modelos preexistentes?**
Sí, siempre que se declare su origen. Lo que se evalúa es el análisis, no la originalidad del diseño visual.

**¿Cuántas medidas DAX se esperan?**
No hay un número mínimo. Se evalúa la corrección y la pertinencia, no la cantidad.

**¿De dónde salen los datos de ASC?**
Son datos sintéticos construidos por el Comité Organizador para este caso. No corresponden a ninguna empresa real. Están listos para usar en `data/raw/`: no hay que procesarlos previamente ni ejecutar nada para obtenerlos.

**¿Los datos tienen errores?**
Sí, deliberadamente. Detectarlos, documentarlos y decidir cómo tratarlos forma parte de la evaluación.

---

## Contacto

Comité Organizador Hackathon de Comunidad UBB 2026
Facultad de Ciencias Empresariales, Universidad del Bío-Bío, Sede Concepción

Las dudas técnicas sobre el caso se canalizan exclusivamente por los medios oficiales informados en la convocatoria.

---

*Documento del caso. Hackathon de Comunidad UBB 2026. Los datos de Austral Seafood Chile SpA son ficticios y fueron construidos con fines exclusivamente académicos. Las empresas mencionadas en el Componente B son reales y su información financiera es de carácter público, disponible en el sitio de la Comisión para el Mercado Financiero.*
