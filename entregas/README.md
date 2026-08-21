# Instrucciones de entrega

Hackathon de Comunidad UBB 2026 · Analítica de Datos para Contadores

---

## 1. Qué se entrega

Una única carpeta comprimida (`.zip`) por equipo, con el siguiente contenido:

| # | Entregable | Formato | Obligatorio |
|---|---|---|---|
| 1 | Modelo y tablero de Cuentas por Cobrar | `.pbix` o carpeta PBIP | Sí |
| 2 | Herramienta de benchmark XBRL | `.pbix` o libro de Excel con Power Query | Sí |
| 3 | Informe ejecutivo | PDF, máximo 10 páginas | Sí |
| 4 | Presentación de defensa | `.pptx` o equivalente, máximo 12 láminas | Sí |
| 5 | Archivos XBRL descargados de la CMF | `.xbrl` / `.zip` originales | Sí |
| 6 | Prototipo web | Enlace público en un `.txt` o dentro del informe | Opcional |

## 2. Nombre del archivo

```
EQUIPO-<NN>-<nombre-corto-del-equipo>.zip
```

Ejemplo: `EQUIPO-07-marea-alta.zip`

Use solo letras, números y guiones. Sin espacios, sin tildes, sin ñ.

## 3. Estructura interna sugerida

```
EQUIPO-07-marea-alta/
├── 01-tablero-cxc/
│   └── cuentas-por-cobrar.pbix
├── 02-benchmark-xbrl/
│   ├── benchmark.pbix            (o libro de Excel con Power Query)
│   ├── tabla-mapeo.xlsx
│   └── xbrl/                    (archivos originales descargados de la CMF)
├── 03-informe/
│   └── informe-ejecutivo.pdf
├── 04-presentacion/
│   └── defensa.pptx
└── 05-prototipo/
    └── enlace.txt               (opcional)
```

## 4. Plazo y canal

- **Plazo:** al cierre del bloque técnico de la jornada. La hora exacta se informa en sala.
- **Canal:** el medio oficial de entrega informado por el Comité Organizador en la convocatoria.

> **Comité Organizador:** completar aquí el canal definitivo (formulario, carpeta compartida o correo) antes de publicar el repositorio a los equipos.

Las entregas fuera de plazo se reciben, pero se evalúan con la penalización que el jurado informe al inicio de la jornada.

## 5. Lista de verificación antes de comprimir

**Componente A**

- [ ] El modelo abre sin errores de actualización y las rutas de datos son relativas o están documentadas.
- [ ] Las medidas DAX principales están descritas (nombre, propósito, fórmula) en el informe o en un documento anexo.
- [ ] Están calculados los nueve indicadores obligatorios, con la desagregación exigida.
- [ ] Está documentado el tratamiento de cada problema de calidad de datos detectado.
- [ ] La matriz de provisión está fundamentada en los datos, no copiada de la tabla referencial.
- [ ] Está la conciliación entre provisión financiera y castigo tributario.
- [ ] Está la proyección mensual enero-junio 2026, con apertura CLP/USD y tres escenarios.
- [ ] Están las cinco propuestas de control interno, con evidencia numérica cada una.

**Componente B**

- [ ] Al menos cinco empresas del Grupo 1, verificadas en la CMF.
- [ ] Está la bitácora de descarga (empresa, RUT, URL, fecha, período, moneda).
- [ ] Está la tabla de mapeo indicador-elemento-empresa.
- [ ] El benchmark usa la mediana y el rango intercuartílico, no el promedio.
- [ ] Los outliers están identificados y su tratamiento justificado.
- [ ] Está el posicionamiento de ASC frente al DSO sectorial.
- [ ] Están incorporadas las consideraciones sectoriales pertinentes (NIC 41, estacionalidad, moneda, cuotas, eventos sanitarios).

**Formales**

- [ ] El informe no supera 10 páginas y la presentación no supera 12 láminas.
- [ ] Está el anexo de prompts de IA, con las validaciones realizadas.
- [ ] El nombre del archivo sigue la convención.
- [ ] El ZIP abre correctamente y pesa menos de 250 MB.

## 6. Sobre el uso de IA

El anexo de prompts es obligatorio. Debe declarar qué herramienta se usó, para qué, qué se le pidió y cómo se validó la salida contra fuente oficial. Use [`templates/plantilla-anexo-prompts.md`](../templates/plantilla-anexo-prompts.md).

Declarar el uso de IA no resta puntaje: es parte del criterio de innovación de la rúbrica. Omitirlo o presentar salidas de IA sin validar sí resta.
