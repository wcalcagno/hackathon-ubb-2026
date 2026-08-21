# Plantilla — Informe ejecutivo

Estructura exigida del entregable N° 3. **PDF, máximo 10 páginas**, incluida la portada y sin contar el anexo de prompts.

Reemplace el texto entre corchetes. Las indicaciones en cursiva son guía, no contenido: elimínelas antes de exportar.

---

## Portada (½ página)

- Título: Diagnóstico de cartera y benchmark sectorial — Austral Seafood Chile SpA
- Nombre del equipo
- Integrantes
- Fecha
- Hackathon de Comunidad UBB 2026 · FACE · Universidad del Bío-Bío

---

## 1. Resumen ejecutivo (1 página)

*Escrito para la Gerencia de Administración y Finanzas, no para el jurado técnico. Si el gerente solo lee esta página, debe poder tomar decisiones.*

- **Situación.** [Dos o tres frases con el diagnóstico central, con las cifras clave.]
- **Cifras de cabecera.** [Cartera bruta, cartera exigible, DSO, morosidad, provisión propuesta, recaudación proyectada del primer semestre 2026.]
- **Los tres hallazgos más relevantes.** [Uno por línea, cada uno con su cifra.]
- **Las tres decisiones que se solicitan al Directorio.** [Acción, monto involucrado, plazo.]

---

## 2. Metodología (1 página)

- Fuentes utilizadas y fecha de corte.
- Tratamiento de la calidad de datos: qué se detectó, qué se corrigió, qué se excluyó y con qué criterio. *Una tabla resumen es más eficiente que un párrafo.*
- Decisiones de modelado relevantes.
- Supuestos declarados: tipo de cambio 2026, criterios de clasificación, ajuste prospectivo de IFRS 9, tratamiento del factoring.

| Problema detectado | Casos | Criterio aplicado | Efecto en la cartera (CLP) |
|---|---:|---|---:|
| | | | |

---

## 3. Hallazgos del Componente A (3 páginas)

### 3.1 Composición y antigüedad de la cartera

[Aging por tramo, con monto y participación. Apertura por línea de negocio.]

### 3.2 Clasificación por recuperabilidad

[Vigente / Vencida recuperable / Vencida en riesgo / Incobrable, con monto y participación.]

### 3.3 Indicadores

[Los nueve indicadores obligatorios, con su desagregación. Priorice la lectura: qué dice cada número, no solo cuál es.]

### 3.4 Hallazgos específicos

*Cada hallazgo con evidencia numérica. Sin cifra, no es un hallazgo: es una opinión.*

| # | Hallazgo | Evidencia | Monto expuesto (CLP) |
|---|---|---|---:|
| 1 | | | |

### 3.5 Provisión por deterioro (IFRS 9)

- Matriz propuesta y **cómo se derivó** de los datos 2024-2025.
- Provisión resultante, total y por tramo.
- Impacto en el resultado del ejercicio.

### 3.6 Castigo tributario y conciliación

| Concepto | Monto CLP |
|---|---:|
| Provisión por deterioro (IFRS 9) | |
| (−) Castigos que cumplen art. 31 N° 4 | |
| = Diferencia temporaria | |
| Impuesto diferido asociado | |

### 3.7 Proyección de recaudación enero-junio 2026

- Cascada de deducciones, línea por línea.
- Curva de recuperación utilizada y cómo se estimó.
- Proyección mensual, con apertura CLP/USD.
- Tres escenarios y el supuesto que cambia en cada uno.

---

## 4. Hallazgos del Componente B (2 páginas)

### 4.1 Peer group

- Empresas incluidas y por qué.
- **Empresas descartadas y por qué.**
- Verificación realizada en la CMF (bitácora resumida).

### 4.2 La herramienta

- Enlace público a la aplicación construida en Lovable.
- Cómo se especificó: qué se le pidió y cómo se verificó que la extracción fuera correcta.
- Cifras contrastadas contra el estado financiero en PDF (al menos dos por empresa).

### 4.3 Normalización

- Moneda de análisis y tipo de cambio aplicado.
- Principales diferencias de agregación encontradas y cómo se homologaron.
- Referencia a la tabla de mapeo completa (anexo o archivo adjunto).

### 4.4 Benchmark

- Indicadores por empresa, con mediana y rango intercuartílico del sector.
- Outliers identificados y su tratamiento.
- **Posicionamiento de ASC frente al DSO sectorial.**

### 4.5 Consideraciones sectoriales

[Cómo afectan NIC 41, la estacionalidad de cosecha, la exposición cambiaria, las cuotas de pesca y los eventos sanitarios a la comparabilidad de las cifras.]

---

## 5. Recomendaciones de control interno (1½ páginas)

Mínimo cinco. Una tabla por recomendación, o una tabla con cinco filas.

| # | Debilidad detectada (con evidencia) | Riesgo | Control propuesto | Responsable | Indicador de seguimiento |
|---|---|---|---|---|---|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

---

## 6. Conclusiones (½ página)

*Responda las cuatro preguntas del encargo, en orden, con una cifra cada una:*

1. ¿Cuál es el estado real de la cartera al 31-12-2025?
2. ¿Cuánto de esa cartera es efectivamente recuperable?
3. ¿Cuánta caja va a entrar el primer semestre de 2026?
4. ¿La gestión de cobranza de ASC está por encima o por debajo de la industria, y por qué?

---

## Anexo · Prompts de IA

Ver [`plantilla-anexo-prompts.md`](plantilla-anexo-prompts.md). No cuenta dentro del máximo de 10 páginas.

---

### Recomendaciones de forma

- Cifras en CLP con separador de miles, sin decimales. Los porcentajes con un decimal.
- Toda tabla y todo gráfico llevan título, unidad y fuente.
- Si un gráfico y un párrafo dicen lo mismo, elimine el párrafo.
- Evite capturas de pantalla del tablero como sustituto del análisis: el tablero se entrega aparte y se evalúa aparte.
