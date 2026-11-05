# Accidentalidad vial en Bogotá: diagnóstico de calidad de datos

**Aprendiz:** _tu nombre_ · **Ficha:** _tu ficha_ · **Actividad:** GA1-220501094-01-AA1-EV06 (SENA)

## Propósito
Realizar un diagnóstico inicial de calidad de los datasets del proyecto (estructura, tipos de datos, valores faltantes e inconsistencias) antes de iniciar el análisis, y documentar las acciones de limpieza propuestas. En esta etapa **no se limpian los datos**: se diagnostican y se documentan.

## Pregunta analítica
¿A qué horas, en qué días y en qué localidades ocurren más accidentes de tránsito (y muertes por accidentes) en Bogotá, y qué medio de transporte está más involucrado?

## Datasets
| Dataset | Fuente | Cobertura | Archivo local |
|---|---|---|---|
| Mortalidad por accidentes de tránsito | Secretaría Distrital de Salud, [Datos Abiertos Bogotá](https://datosabiertos.bogota.gov.co/en/dataset/mortalidad-por-accidentes-de-transito/resource/a956ec09-e22b-4b84-aeb9-25702cf610a0) | Víctimas fatales, 2015–2026 (2026 incompleto) | `data/raw/accidentes.csv` |
| Anuario de Siniestralidad Vial 2025 (hojas *Siniestros*, *Vehículos* y *Actor vial*) | Secretaría Distrital de Movilidad, [Observatorio de Movilidad de Bogotá](https://observatorio.movilidadbogota.gov.co/node/354) | Siniestros viales de 2025 | `data/raw/BaseSiniestralidad.xlsx` |

El notebook convierte una sola vez las tres hojas del Excel a CSV (`anuario_siniestros.csv`, `anuario_vehiculos.csv` y `anuario_actor_vial.csv`) y diagnostica cuatro DataFrames: `mortalidad`, `siniestros`, `vehiculos` y `actor_vial`.

Licencia de los datos de mortalidad: Creative Commons Attribution 4.0. Revisa la licencia de cada fuente antes de reutilizar los datos.

Los archivos de datos **no se suben al repositorio** (están en `.gitignore`) por su tamaño. Para obtenerlos, descárgalos desde los enlaces de la tabla y guárdalos en `data/raw/` con los nombres indicados (en el Observatorio, el Excel se descarga como `Base_Anuario_de_Siniestralidad_Vial_2025 (2).xlsx`; renómbralo a `BaseSiniestralidad.xlsx`).

## Estructura del repositorio
```
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/                          # archivos de datos (ignorados por Git)
├── notebooks/
│   ├── 00_entorno.ipynb              # comprobación del entorno
│   └── diagnostico_calidad.ipynb     # diagnóstico de calidad
└── reports/
    └── informe_diagnostico.md        # informe técnico
```

## Cómo reproducir el diagnóstico
Requisitos: Python 3.12 o superior y Git.

```bash
git clone <URL_DEL_REPOSITORIO>
cd <carpeta_del_repositorio>
python -m venv .venv
.venv\Scripts\activate             # macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
```
1. Descarga los dos archivos de datos y guárdalos en `data/raw/` (ver "Datasets").
2. Abre JupyterLab con `jupyter lab`.
3. Abre `notebooks/diagnostico_calidad.ipynb` y ejecuta todo con **Kernel → Restart Kernel and Run All Cells**. La primera vez tarda más, porque convierte el Excel a CSV.

## Principales hallazgos
- El Anuario 2025 es la mejor base para la pregunta: tiene hora exacta, día y localidad casi completos, y sus tres hojas se unen sin pérdidas por `Codigo_Siniestro`.
- En `mortalidad`, cada víctima aparece duplicada (13.138 filas equivalen a 6.569 víctimas), el 34,9 % no tiene hora y hay formatos de texto mezclados (mayúsculas, paréntesis).
- Las fechas y las coordenadas del Anuario están como texto, y la mayoría de los nulos de sus columnas `Con_*` significan "no aplica".

El detalle, con las cifras y las acciones de limpieza propuestas, está en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md).
