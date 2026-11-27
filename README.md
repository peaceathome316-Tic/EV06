# Accidentalidad vial en Bogotá: diagnóstico de calidad de datos

**Aprendiz:** david vargas franco · **Ficha:** 3169901 · 
:::writing{variant="document" id="73192"}
# Accidentalidad vial en Bogotá: preparación y análisis de datos

**Actividad:** GA1-220501094-01-AA1-EV07 (SENA)  
**Proyecto:** Preparación, limpieza y estructuración de datos con Pandas  
**Cobertura:** Bogotá, Colombia, año 2025.

## 1. Propósito

Preparar, limpiar, estandarizar e integrar diferentes conjuntos de datos sobre siniestralidad vial en Bogotá durante 2025, utilizando Python y Pandas.

El proyecto busca identificar patrones de accidentalidad relacionados con horarios, días de la semana, localidades y clases de vehículos involucradas.

## 2. Pregunta de investigación

¿A qué horas, en qué días de la semana y en qué localidades de Bogotá ocurrieron más siniestros viales con heridos o muertos durante 2025, y qué clases de vehículo estuvieron más involucradas?

## 3. Fuentes de datos

| Dataset | Fuente | Archivo local |
|---|---|---|
| Anuario de Siniestralidad Vial 2025 | Secretaría Distrital de Movilidad | `data/raw/BaseSiniestralidad.xlsx` |
| Siniestros viales 2025 | Derivado del Anuario 2025 | `data/raw/anuario_siniestros.csv` |
| Vehículos involucrados 2025 | Derivado del Anuario 2025 | `data/raw/anuario_vehiculos.csv` |
| Actores viales 2025 | Derivado del Anuario 2025 | `data/raw/anuario_actor_vial.csv` |
| Mortalidad por accidentes de tránsito | Secretaría Distrital de Salud | `data/raw/accidentes.csv` |

Fuentes oficiales:

- [Observatorio de Movilidad de Bogotá](https://observatorio.movilidadbogota.gov.co/node/354).
- [Datos Abiertos Bogotá: Mortalidad por accidentes de tránsito](https://datosabiertos.bogota.gov.co/en/dataset/mortalidad-por-accidentes-de-transito/resource/a956ec09-e22b-4b84-aeb9-25702cf610a0).

El conjunto `accidentes.csv` contiene información histórica de 2015 a 2026 y se conserva como fuente complementaria. No se integra directamente con las tablas del Anuario 2025 porque no se identificó una clave común válida para relacionarlas.

## 4. Estructura del repositorio

```text
accidentalidad-Colombia/
│
├── README.md
├── .gitignore
├── requirements.txt
│
├── data/
│   ├── raw/
│   │   └── Archivos originales (ignorados por Git)
│   │
│   └── processed/
│       ├── siniestros_integrados_2025.csv
│       └── siniestros_integrados_2025.parquet
│
├── notebooks/
│   ├── 00_entorno.ipynb
│   ├── data_quality.ipynb
│   └── data_preparation.ipynb
│
└── reports/
    └── data_quality.md
```

## 5. Preparación y limpieza de datos

Se trabajó principalmente con tres tablas del Anuario de Siniestralidad Vial 2025:

| Tabla | Registros originales | Columnas |
|---|---:|---:|
| Siniestros | 11.573 | 25 |
| Vehículos | 19.974 | 18 |
| Actores viales | 28.029 | 31 |

Las actividades de limpieza incluyeron:

- Estandarización de nombres de columnas y campos de texto.
- Conversión de fechas a tipos datetime.
- Conversión de coordenadas geográficas a valores numéricos.
- Normalización de indicadores de presencia de elementos (`Con_*`).
- Tratamiento de valores faltantes según el significado y comportamiento observado de las variables.
- Conservación de registros sin información cuando no era posible determinar un valor correcto.
- Validación de dimensiones y registros duplicados.

En los campos indicadores `Con_*`, los valores nulos se reemplazaron por `NO`, de acuerdo con la estructura observada de estos campos, que contenían únicamente `SI` y valores nulos.

También se implementaron funciones reutilizables para limpiar texto, nombres de columnas, indicadores, fechas y coordenadas. Estas funciones se aplicaron mediante el método `pipe()` de Pandas.

## 6. Filtrado y análisis de datos

Se seleccionaron los siniestros clasificados como `Con Heridos` o `Con Muertos`.

El conjunto de análisis contiene 11.573 siniestros:

- 11.022 siniestros con heridos.
- 551 siniestros con muertos.

Se desarrollaron agrupaciones para responder la pregunta de investigación:

- Cantidad de siniestros por hora.
- Cantidad de siniestros por día de la semana.
- Cantidad de siniestros por localidad.
- Cantidad de vehículos involucrados por clase.

### Principales resultados

**Siniestros por hora**

La hora con mayor cantidad de siniestros fue las 7:00 a. m., con 789 registros.

**Siniestros por día**

| Día | Siniestros |
|---|---:|
| Viernes | 1.826 |
| Miércoles | 1.812 |
| Sábado | 1.738 |
| Jueves | 1.717 |
| Martes | 1.665 |
| Lunes | 1.463 |
| Domingo | 1.352 |

El viernes fue el día con mayor cantidad de siniestros.

**Siniestros por localidad**

| Localidad | Siniestros |
|---|---:|
| Kennedy | 1.510 |
| Suba | 1.030 |
| Engativá | 955 |
| Puente Aranda | 873 |
| Fontibón | 844 |

Kennedy presentó la mayor cantidad de siniestros registrados.

**Vehículos involucrados por clase**

| Clase | Vehículos |
|---|---:|
| Motocicleta | 8.770 |
| Automóvil | 4.513 |
| Bus | 1.841 |
| Camioneta | 1.760 |
| Bicicleta | 1.740 |

Las motocicletas fueron la clase de vehículo con mayor participación en los registros analizados.

Estos resultados representan cantidades absolutas y no tasas de accidentalidad por población o cantidad de vehículos.

## 7. Integración de datos

Para integrar las tablas se utilizó `Codigo_Siniestro` como clave de relación.

Antes de realizar las uniones, se agruparon los registros de vehículos y actores por código de siniestro. Esto permitió evitar duplicaciones y relaciones muchos a muchos que pudieran multiplicar los registros.

Se construyeron dos resúmenes:

- **Resumen de vehículos:** cantidad de vehículos y clases involucradas por siniestro.
- **Resumen de actores:** cantidad de actores, heridos, muertos y condiciones de los actores por siniestro.

Posteriormente, estos resúmenes se integraron con la tabla principal de siniestros mediante `merge()` y validación `one_to_one`.

### Resultado de la integración

| Indicador | Resultado |
|---|---:|
| Siniestros integrados | 11.573 |
| Vehículos involucrados | 19.974 |
| Actores viales | 28.029 |
| Actores heridos | 19.605 |
| Actores muertos | 562 |
| Columnas del conjunto integrado | 31 |

Cada `Codigo_Siniestro` aparece una sola vez en el conjunto final.

## 8. Archivos procesados

Los resultados se exportaron en dos formatos:

| Archivo | Descripción |
|---|---|
| `siniestros_integrados_2025.csv` | Conjunto integrado en formato CSV. |
| `siniestros_integrados_2025.parquet` | Conjunto integrado en formato Parquet. |

Ambos archivos contienen 11.573 registros y 31 columnas.

Se realizó una lectura posterior de los archivos exportados para comprobar sus dimensiones y la unicidad de los códigos de siniestro.

## 9. Validaciones realizadas

Se implementaron comprobaciones mediante `assert` para verificar:

- Conservación de las dimensiones originales durante la limpieza.
- Ausencia de filas completamente duplicadas.
- Ausencia de valores nulos en los indicadores estandarizados.
- Correspondencia de los códigos de siniestro entre las tablas.
- Unicidad de los códigos en los resúmenes y en el conjunto integrado.
- Conservación de los 11.573 registros durante la integración.
- Correcta exportación y lectura de los archivos CSV y Parquet.

Todas las validaciones finales fueron superadas.

## 10. Tecnologías utilizadas

- Python.
- Pandas.
- NumPy.
- PyArrow.
- Jupyter Notebook.
- Git y GitHub.

## 11. Reproducción del proyecto

Se requiere Python y Git.

Clonar el repositorio:

```bash
git clone https://github.com/peaceathome316-Tic/EV06.git
cd EV06
```

Crear y activar el entorno virtual en Windows:

```cmd
python -m venv .venv
.venv\Scripts\activate
```

Instalar las dependencias:

```bash
python -m pip install -r requirements.txt
```

Descargar los archivos originales desde las fuentes oficiales y guardarlos en `data/raw/` con los nombres indicados en la sección de fuentes de datos.

Iniciar Jupyter:

```bash
jupyter notebook
```

Abrir y ejecutar los notebooks en este orden:

1. `notebooks/00_entorno.ipynb`
2. `notebooks/data_quality.ipynb`
3. `notebooks/data_preparation.ipynb`

Para ejecutar completamente cada notebook, utilizar la opción **Kernel → Restart Kernel and Run All Cells**.

Los archivos procesados se generan en `data/processed/`.

## 12. Consideraciones

Los archivos originales se conservan sin modificaciones en `data/raw/` y están excluidos del seguimiento de Git mediante `.gitignore`.

Los resultados corresponden a los registros disponibles en el Anuario de Siniestralidad Vial 2025 y dependen de la calidad y cobertura de la fuente original.

:::