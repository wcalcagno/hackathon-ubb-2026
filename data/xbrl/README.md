# `data/xbrl/` — Descargas de la CMF

Esta carpeta está vacía a propósito. **Los archivos XBRL los descarga cada equipo durante la jornada** desde el sitio de la Comisión para el Mercado Financiero.

No se distribuyen precargados por dos razones: la obtención de la fuente oficial forma parte del desafío, y los estados financieros son información pública que debe tomarse siempre desde el emisor.

---

## 1. Qué descargar

Estados financieros **anuales al 31-12-2025**, en formato **XBRL**, de al menos **cinco empresas del Grupo 1** definido en la sección 5.3 del README.

## 2. Dónde obtenerlos

```
https://www.cmfchile.cl
  > Mercados > Valores > Consulta por Fiscalizado > Emisores de Valores
    > [Buscar la empresa] > Estados Financieros
      > Estados Financieros Anuales > 31-12-2025 > Formato XBRL
```

La guía completa de descarga, lectura y normalización está en [`docs/03-guia-xbrl-cmf.md`](../../docs/03-guia-xbrl-cmf.md).

## 3. Cómo organizarlos

Una subcarpeta por empresa, en minúsculas y sin espacios:

```
data/xbrl/
├── blumar/
│   ├── <instancia>.xbrl
│   └── (esquemas y linkbases, si el ZIP los incluye)
├── camanchaca/
├── salmones-camanchaca/
├── multiexport-foods/
├── australis-seafoods/
└── invermar/
```

## 4. Bitácora de descarga (obligatoria)

Registre esta tabla en el informe. Es la evidencia de que el análisis es reproducible y trazable a la fuente oficial.

| Empresa | RUT | Fecha de descarga | URL exacta | Período | Consolidado / Individual | Moneda de presentación | Archivo |
|---|---|---|---|---|---|---|---|
| | | | | 31-12-2025 | | | |

## 5. Verificación previa

Antes de parsear, confirme en la CMF, para cada empresa:

- [ ] Mantiene condición de **emisor vigente**.
- [ ] **Publicó** estados financieros al 31-12-2025.
- [ ] La **razón social y el RUT** coinciden con los del registro oficial (no los tome del enunciado).
- [ ] Identificó si lo descargado es **consolidado** o **individual**, y usa el mismo criterio para todas las empresas del peer group.

Si una empresa no publicó al 31-12-2025: documéntelo, exclúyala, reemplácela por otra comparable y mantenga el mínimo de cinco empresas del Grupo 1.

---

> Los archivos descargados pueden pesar varios MB. Si va a versionar su trabajo en Git, considere que el `.gitignore` de este repositorio excluye los archivos XBRL descargados; para la entrega, inclúyalos en el ZIP según las instrucciones de [`entregas/README.md`](../../entregas/README.md).
