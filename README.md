# Accidentalidad vial en Bogotá: diagnóstico de calidad de datos

**Aprendiz:** _tu nombre_ · **Ficha:** _tu ficha_ · **Actividad:** GA1-220501094-01-AA1-EV06

## Propósito
Realizar un diagnóstico inicial de calidad de los datasets del proyecto (estructura, tipos de datos, valores faltantes e inconsistencias) antes de iniciar el análisis, y documentar las acciones de limpieza propuestas.

## Pregunta analítica
¿A qué horas, en qué días y en qué localidades ocurren más accidentes de tránsito (y muertes por accidentes) en Bogotá, y qué medio de transporte está más involucrado?

## Datasets
| Dataset | Fuente | Cobertura | Archivo local |
|---|---|---|---|
| Mortalidad por accidentes de tránsito | Secretaría Distrital de Salud, [Datos Abiertos Bogotá](https://datosabiertos.bogota.gov.co/dataset/mortalidad-por-accidentes-de-transito) | Accidentes con víctimas fatales | `data/raw/accidentes.csv` |
| Anuario de Siniestralidad (Siniestros, Vehículos y Actor vial) | Secretaría Distrital de Movilidad, [Observatorio de Movilidad de Bogotá](https://observatorio.movilidadbogota.gov.co/node/354) | Siniestros viales en general | `data/raw/BaseSiniestralidad.xlsx` (se convierte a CSV desde el notebook) |

Licencia de los datos: Creative Commons Attribution 4.0. Los archivos de datos (CSV y Excel) **no se suben al repositorio** (están en `.gitignore`): descárgalos desde los enlaces y guárdalos en `data/raw/`.

## Estructura
```
├── README.md
├── .gitignore
├── requirements.txt
├── data/raw/                  # CSV descargados (ignorados por Git)
├── notebooks/diagnostico_calidad.ipynb
└── reports/informe_diagnostico.md
```

## Cómo reproducir
```bash
git clone <URL_DEL_REPOSITORIO>
cd <carpeta>
python -m venv .venv
.venv\Scripts\activate           # macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
# descargar los archivos de datos en data/raw/
jupyter lab                        # abrir notebooks/diagnostico_calidad.ipynb
```
En la segunda celda del notebook, verifica los nombres de los archivos en `ARCHIVOS` y ejecuta todo con *Restart Kernel and Run All Cells*.

## Resultados
El informe técnico está en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md).
