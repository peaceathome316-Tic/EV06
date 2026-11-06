# Informe técnico: diagnóstico de calidad de datos

**Proyecto:** Accidentalidad vial en Bogotá · **Aprendiz:** david vargas franco · **Ficha:** 3169901

## 1. Pregunta analítica
¿A qué horas, en qué días y en qué localidades ocurren más accidentes de tránsito (y muertes por accidentes) en Bogotá, y qué medio de transporte está más involucrado?

## 2. Descripción de los datasets

| Dataset | Fuente | Qué representa cada fila | Período |
|---|---|---|---|
| `mortalidad` (`accidentes.csv`) | Secretaría Distrital de Salud, Datos Abiertos Bogotá | Un registro de víctima fatal de un accidente de tránsito; **cada víctima aparece dos veces** (ver 4.3), por lo que hay 6.569 víctimas únicas | 2015 a 2026 (2026 incompleto) |
| `siniestros` (hoja *Siniestros*) | Secretaría Distrital de Movilidad, Anuario de Siniestralidad 2025 | Un siniestro vial | 1 de enero al 31 de diciembre de 2025 |
| `vehiculos` (hoja *Vehículos*) | Mismo archivo | Un vehículo involucrado en un siniestro | 2025 |
| `actor_vial` (hoja *Actor vial*) | Mismo archivo | Una persona involucrada (víctima o actor) en un siniestro | 2025 |

Las tres hojas del Anuario se relacionan por `Codigo_Siniestro`. Las cuatro fuentes comparten localidad, día y hora, por lo que pueden compararse. `mortalidad` cubre muchos más años que el Anuario, así que para comparar debe filtrarse `ANO == 2025`.

## 3. Inspección estructural

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| mortalidad | 13.138 | 19 | str: 15, int64: 4 | Sin nulos ni duplicados exactos, pero con "Sin información" como texto y con cada víctima duplicada (4.3) |
| siniestros | 11.573 | 25 | str: 20, int64: 5 | 93.066 celdas nulas (casi todas en columnas `Con_*`); sin duplicados |
| vehiculos | 19.974 | 18 | str: 15, int64: 3 | 173.195 celdas nulas; sin duplicados; unos 1,7 vehículos por siniestro |
| actor_vial | 28.029 | 31 | str: 22, int64: 7, float64: 2 | 248.835 celdas nulas; sin duplicados; unos 2,4 actores por siniestro |

**Características principales**
- **mortalidad:** `casos` vale 1 en todas las filas, pero cada víctima aparece dos veces (una con `LOCALIDAD = "Bogotá"` y otra con su localidad), así que las 13.138 filas equivalen a 6.569 víctimas. Trae `_id` (identificador técnico), localidad, año, mes, día, rango de hora de 3 horas, clase de accidente, condición de la víctima y medio de desplazamiento.
- **siniestros:** fecha, hora exacta (`Hora_Siniestro` de 0 a 23), día de la semana, localidad, clase, gravedad (`Con Heridos` 11.022 y `Con Muertos` 551) y coordenadas.
- **vehiculos:** 20 clases de vehículo; la más frecuente es Motocicleta (8.770 filas, 43,9 %), seguida de Automóvil (4.513), Bus (1.841), Camioneta (1.760) y Bicicleta (1.740).
- **actor_vial:** condición de la persona (motociclista 11.131, conductor 9.384, peatón 3.011, pasajero 2.636, ciclista 1.867), edad, sexo y gravedad (herido 19.605, ileso 7.841, muerto 562).
- **Integridad entre las hojas del Anuario:** hay 11.573 siniestros únicos; todo vehículo y todo actor vial corresponde a un siniestro existente, y todo siniestro tiene al menos un vehículo y un actor. Las tres hojas se pueden unir sin pérdidas.

## 4. Valores faltantes, tipos de datos e inconsistencias

### 4.1 Valores faltantes
_Nota: como cada víctima de `mortalidad` está duplicada, sus cifras en filas son el doble de las víctimas reales; los porcentajes no cambian._

- **mortalidad no tiene nulos, pero sí datos ausentes escritos como texto.** `RANGO_DE_HORA_DEL_HECHO_XXX3_HORAS_` tiene 4.588 filas "Sin información" (34,9 %; 2.294 víctimas), `CIRCUNSTANCIA_DEL_HECHO_DETALLADA` 7.582 (57,7 %), `ANCESTRO_RACIAL` 3.116 y `PERTENENCIA_ETNICA` 5.192 (este incluye "No aplica").
- **Medio de desplazamiento en mortalidad:** el cruce con la condición de la víctima muestra que los 5.086 "No aplica" son todos peatones. Los peatones aparecen con dos etiquetas distintas ("No aplica" 5.086 y "Peatón" 498). El dato realmente desconocido es "Sin información": 122 registros (0,9 %).
- **Zona en mortalidad:** `LOCALIDAD = "Bogotá"` (código 0) aparece en 6.569 filas, justo la mitad del archivo: es la copia a nivel de ciudad de cada víctima (ver 4.3), no una localidad real. Entre las 6.569 filas con localidad específica, solo 260 (4,0 %) dicen "Sin dato"; por tanto, el 96 % de las víctimas sí tiene localidad.
- **siniestros:** `Localidad` tiene solo 1 nulo. `Elemento_Choque` tiene 3.685 nulos (31,8 %), que probablemente corresponden a siniestros que no son choques.
- **vehiculos:** `Servicio` tiene 2.769 nulos (13,9 %).
- **actor_vial:** `Codigo_Vehiculo` 3.023 nulos (10,8 %; casi igual al número de peatones, 3.011), `Edad` 1.510 (5,4 %), `Sexo` 1.424 (5,1 %), `Gravedad_Indicador_30d` 21 y `Localidad` 2.
- **Nulos que no son faltantes reales:** las columnas `Con_*` solo contienen "SI" o vacío (vacío = "no aplica"), y `Fecha_CambioGravedad` (212 valores, 99,2 % vacío) solo se llena si cambia la gravedad. `Modalidad`, `Tipo_SITP` y `Tipo_Objeto_Fijo` aplican solo a ciertos casos. El borrador automático los marca como "faltantes", pero deben leerse como "no aplica".

### 4.2 Tipos de datos
- Las fechas (`Fecha_Siniestro`, `FechaSiniestro`, `Fecha_CambioGravedad`) están como texto en las tres hojas del Anuario. Todas se convierten sin errores con `pd.to_datetime`.
- `Longitud` y `Latitud` están como texto con **coma decimal** (por ejemplo `-74,13660797754`). Al reemplazar la coma por punto, las 11.573 coordenadas se convierten sin errores y ninguna cae fuera del rango aproximado de Bogotá.
- `Edad` y `Codigo_Vehiculo` son `float64` solo porque contienen nulos.
- Las columnas numéricas relevantes (`ANO`, `casos`, `Hora_Siniestro`, `Min_Siniestro`) tienen el tipo correcto. `Hora_Siniestro` va de 0 a 23 y `Min_Siniestro` de 0 a 59, sin valores fuera de rango. El diccionario del portal describe algunas como texto, pero Pandas las lee como enteros.

### 4.3 Inconsistencias
- **Duplicación de víctimas en mortalidad (confirmada).** Cada víctima aparece dos veces: una fila con `LOCALIDAD = "Bogotá"` (código 0) y otra con su localidad. Al agrupar por todas las columnas excepto `_id`, `CODIGO_LOCALIDAD` y `LOCALIDAD`, hay 6.502 grupos de 2 filas (1 con "Bogotá"), 32 grupos de 4 filas (2 con "Bogotá") y 1 grupo de 6 filas (3 con "Bogotá"). En todos los grupos, la mitad de las filas es "Bogotá". Así, 13.138 filas corresponden a 6.569 víctimas. `duplicated()` no lo detectó porque cada fila tiene un `_id` distinto.
- **Víctimas por año en mortalidad (sin duplicar):** 2015: 556, 2016: 591, 2017: 553, 2018: 527, 2019: 523, 2020: 406, 2021: 508, 2022: 630, 2023: 629, 2024: 665, 2025: 612 y 2026: 369 (año incompleto). Para 2025, las 612 víctimas de mortalidad son del mismo orden que las 562 víctimas fatales del Anuario, aunque las fuentes miden de forma distinta y no tienen por qué coincidir.
- **Formatos mezclados en mortalidad.** `MES_DEL_HECHO` tiene 25 valores en lugar de 13 y `DIA_DEL_HECHO` 15 en lugar de 8: hay 2.346 filas (17,9 %) con mes y día capitalizados ("Mayo", "Sábado") mientras el resto está en minúsculas. El rango de hora tiene 18 valores en lugar de 10, por formatos con y sin paréntesis y por "Sin información" / "Sin Información".
- **Efecto en los conteos.** En la vista previa, "sábado" suma 1.802 y "Sábado" 412 como si fueran días distintos, y un mismo rango horario aparece partido en dos (por ejemplo "(18:00 a 20:59)" 872 y "18:00 a 20:59" 440).
- **Categorías casi iguales:** "Indigena" / "Indígena", "No aplica" / "No Aplica", "Caída del ocupante" / "Caída de ocupante", "Avión – avioneta" / "Avión, Avioneta", "Microbus" / "Microbús", y la clase de accidente mezcla categorías que se solapan ("Choque", "Choque con otro vehículo", "Choque con objeto fijo o en movimiento").
- **Nombres de localidad distintos entre fuentes:** `siniestros` las escribe en mayúsculas ("CANDELARIA"), `actor_vial` en formato título ("Candelaria") y `mortalidad` como "La Candelaria". Además `mortalidad` incluye "Bogotá" y "Sin dato" como si fueran localidades.
- **Vocabulario distinto para lo mismo:** `Sexo` es "Hombre/Mujer" en mortalidad y "Masculino/Femenino" en actor_vial; los nombres de columna varían entre hojas del Anuario (`Hora_Siniestro` frente a `hourSiniestro`, `AA_Siniestro` frente a `AnnoSiniestro`, `Fecha_Siniestro` frente a `FechaSiniestro`), y hay un error de escritura en `Ccodigo_Victima`.
- **Clases de vehículo cercanas:** "Motocicleta", "Motociclo", "Mototriciclo", "Motocarro" y "Cuatrimoto", además de "No identificado" (23) y "SIN INFORMACIÓN" (28 en `Vehiculo_Viajaba_Clasificado`).
- **Las banderas `Con_*` describen el siniestro, no el vehículo.** En `vehiculos`, `Con_Moto` es "SI" en 14.000 filas, mientras que solo 8.770 vehículos son motocicletas.
- **Inconsistencia en actor_vial confirmada:** hay 219 registros con `Muerte_Posterior = "S"`, de los cuales 212 tienen `Fecha_CambioGravedad` y **7 no la tienen**. Ninguno de los 27.810 registros con "N" tiene fecha, como corresponde.
- **Cobertura de gravedad:** en siniestros solo existen "Con Heridos" y "Con Muertos"; el archivo no parece incluir siniestros con solo daños materiales.

## 5. Hallazgos principales por dataset

| Dataset | Problema identificado | Evidencia encontrada | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| mortalidad | Cada víctima duplicada (una fila con "Bogotá" y otra con su localidad) | 6.502 grupos de 2 filas, 32 de 4 y 1 de 6, siempre con la mitad en "Bogotá"; 13.138 filas = 6.569 víctimas; `duplicated()` = 0 por el `_id` | Contar filas duplica las muertes | Conservar solo las filas con localidad distinta de "Bogotá" (6.569 víctimas) |
| mortalidad | Hora desconocida y en rangos de 3 horas | 4.588 "Sin información" (34,9 %) | No sirve para hora exacta; un tercio queda fuera | Usar `siniestros` para la hora; tratar "Sin información" como nulo |
| mortalidad | Mayúsculas y formatos mezclados | Mes 25 valores (13 reales), día 15 (8), hora 18 (10); 2.346 filas con formato distinto | Conteos por mes, día y hora partidos | Normalizar texto y quitar paréntesis con expresiones regulares |
| mortalidad | "No aplica" y "Peatón" para el mismo caso | 5.086 "No aplica" (todos peatones) y 498 "Peatón" | El medio parece desconocido en el 39 % cuando no lo es | Unificar en "Peatón" y reservar "Sin información" (122) para los desconocidos |
| mortalidad | "Sin información" como texto | 7.582 (57,7 %) en circunstancia; 3.116 en ancestro racial | Reduce la cobertura real de estas variables | Convertir a nulo y reportar el porcentaje |
| mortalidad | Cobertura temporal distinta | 2015 a 2026 (2026 incompleto: 369 víctimas) frente a solo 2025 | Comparar sin filtrar mezcla períodos | Filtrar `ANO == 2025` para comparar con el Anuario |
| mortalidad | Categorías casi iguales | "Indigena"/"Indígena", "Microbus"/"Microbús", "Avión – avioneta"/"Avión, Avioneta" | Categorías duplicadas | Unificar etiquetas |
| siniestros | Coordenadas como texto con coma decimal | `Longitud` y `Latitud` como `-74,13660797754` (todas convertibles) | No se pueden calcular distancias ni mapas sin convertir | Reemplazar la coma por punto y usar `pd.to_numeric` |
| siniestros | Fecha como texto | `Fecha_Siniestro` es `str` (0 no convertibles) | No agrupa por mes o semana | Convertir con `pd.to_datetime` |
| siniestros | Nulos que significan "no aplica" | `Con_*` solo con "SI" o vacío (hasta 99,7 % vacío); `Elemento_Choque` 31,8 % vacío | Se leerían como datos faltantes | Documentarlos y recodificarlos a 0/1 |
| siniestros | Localidad en mayúsculas y un nulo | 20 valores en mayúsculas, 1 nulo | Dificulta cruzar con otras fuentes | Normalizar y revisar el registro nulo |
| siniestros | Gravedad con solo dos categorías | "Con Heridos" 11.022 y "Con Muertos" 551 | Probablemente no incluye siniestros solo con daños | Confirmar en el diccionario y declararlo como limitación |
| vehiculos | Clases de vehículo solapadas | Motocicleta, Motociclo, Mototriciclo, Motocarro; "No identificado" (23) | Dispersa el conteo por tipo | Agrupar en categorías (motos, autos, buses, bicicletas, carga) |
| vehiculos | Banderas `Con_*` replicadas del siniestro | `Con_Moto` "SI" en 14.000 filas frente a 8.770 motocicletas | Sobreestima vehículos por tipo | Contar con `Clase`, no con `Con_*` |
| actor_vial | Faltantes en edad, sexo y vehículo | `Edad` 5,4 %, `Sexo` 5,1 %, `Codigo_Vehiculo` 10,8 % (≈ peatones) | Sesgo en perfiles por edad y sexo | Reportar porcentajes; el vehículo faltante es esperable en peatones |
| actor_vial | Muerte posterior sin fecha | 7 registros con `Muerte_Posterior = "S"` y sin `Fecha_CambioGravedad` | Inconsistencia en víctimas fatales | Revisar esos 7 registros |
| Anuario (3 hojas) | Nombres de columna distintos entre hojas | `Hora_Siniestro` / `hourSiniestro`, `AA_Siniestro` / `AnnoSiniestro` | Dificulta unir y reutilizar código | Estandarizar nombres al limpiar |
| mortalidad y Anuario | Vocabularios distintos | "Hombre/Mujer" frente a "Masculino/Femenino"; "La Candelaria" frente a "CANDELARIA" | No se comparan categorías directamente | Definir una tabla de equivalencias |

## 6. Acciones de limpieza y preparación propuestas
1. **Eliminar la copia duplicada en mortalidad:** conservar solo las filas con `LOCALIDAD` distinta de "Bogotá" (6.569 víctimas) y tratar "Sin dato" (260) como localidad desconocida.
2. **Convertir tipos:** fechas con `pd.to_datetime`; coordenadas reemplazando la coma por punto; banderas `Con_*` a 0/1.
3. **Normalizar texto:** minúsculas, sin espacios sobrantes ni paréntesis en mes, día, rango de hora, localidad y medio de desplazamiento; unificar las etiquetas casi iguales.
4. **Tratar los "sin dato":** convertir "Sin información" y "Sin dato" a nulo, reportar el porcentaje y unificar "No aplica" de los peatones con "Peatón".
5. **Homogeneizar entre fuentes:** tabla de equivalencias de localidades, sexo y clases de vehículo, y nombres de columna comunes en las hojas del Anuario.
6. **Alinear períodos:** comparar `mortalidad` con el Anuario solo para 2025.
7. **Revisar antes de eliminar:** los 7 registros de muerte posterior sin fecha y el registro con localidad nula.

## 7. Conclusiones sobre las condiciones actuales de los datos
1. **El Anuario 2025 es la mejor base para la pregunta.** Tiene hora exacta, día y localidad casi completos, cubre todo el año, las tres hojas se unen sin pérdidas, las coordenadas son válidas y no hay filas duplicadas. Con `vehiculos` y `actor_vial` permite identificar el tipo de vehículo más involucrado.
2. **`mortalidad` sirve como complemento, no como base principal.** Una de cada tres filas no tiene hora, la hora viene en rangos de 3 horas y, sobre todo, tiene cada víctima duplicada (13.138 filas = 6.569 víctimas), por lo que no se deben contar filas sin eliminar la copia.
3. **Los datos necesitan limpieza antes del análisis,** especialmente en formatos de texto (mayúsculas, paréntesis), tipos (fechas y coordenadas) y la interpretación de los nulos que significan "no aplica".
4. **Las comparaciones entre fuentes deben hacerse con cuidado:** períodos distintos, vocabularios distintos y un Anuario que parece incluir solo siniestros con heridos o muertos. Las conclusiones valen para esos registros y no para todos los accidentes de la ciudad.
