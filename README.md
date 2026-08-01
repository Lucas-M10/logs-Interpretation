# Interpretación de Logs Distribuidos

## Descripción

Este proyecto analiza un dataset de logs distribuidos para identificar el momento de mayor degradación del sistema y determinar qué servicios, endpoints y mensajes concentraron los errores.

El análisis fue desarrollado en un **Jupyter Notebook** utilizando **Pandas** para la preparación y agrupación de los datos, y **Matplotlib** para representar su evolución temporal.

## Objetivo

El notebook busca responder las siguientes preguntas:

- ¿Cuándo estuvo peor el sistema?
- ¿Qué servicio fue el más afectado?
- ¿Qué endpoint concentró más eventos malos?
- ¿Qué mensaje de error fue el más frecuente?
- ¿Qué cambió durante el incidente respecto al baseline?

## Dataset

El archivo CSV contiene registros generados por distintos servicios del sistema.

Campos principales:

- `timestamp_event`
- `received_at`
- `service_name`
- `severity`
- `message`
- `method`
- `endpoint`
- `status_code`
- `latency_ms`
- `trace_id`

Dataset proporcionado para el challenge:

[Descargar dataset desde Google Drive](https://drive.google.com/file/d/1JuQvmvIsmp965S4-Is8SQAIOnM-PN_CN/view?usp=sharing)

## Definiciones utilizadas

### Evento malo

Un registro se considera malo cuando cumple al menos una de estas condiciones:

- `severity` es `ERROR` o `CRITICAL`.
- `status_code` es mayor o igual que `500`.

### Ventana temporal

Los logs se agrupan en ventanas consecutivas de cinco minutos utilizando `timestamp_event`.

### Momento crítico

Es la ventana con mayor `bad_rate`, considerando únicamente las ventanas con al menos 20 eventos.

```text
bad_rate = bad_events / total_events
```

### Baseline

El baseline contiene todos los registros que están fuera de la ventana crítica y funciona como referencia para comparar el comportamiento del sistema durante el incidente.

## Proceso de análisis

1. Carga e inspección del archivo CSV.
2. Conversión de fechas al tipo `datetime`.
3. Revisión de tipos de datos, duplicados y valores nulos.
4. Creación de las columnas `is_5xx` e `is_bad`.
5. Exploración general de severidades, servicios y mensajes.
6. Agrupación de los eventos en ventanas de cinco minutos.
7. Cálculo de `total_events`, `bad_events` y `bad_rate`.
8. Selección del momento crítico.
9. Diagnóstico de servicios, mensajes y endpoints dentro del incidente.
10. Comparación del incidente contra el baseline.
11. Generación de los dos gráficos temporales obligatorios.
12. Elaboración automática de las conclusiones.

## Resultados principales

- **Momento crítico:** 10 de enero de 2026, entre las 11:10 y las 11:15 UTC.
- **Bad rate del incidente:** 58,2 %.
- **Bad rate del baseline:** 14,0 %.
- **Servicio más afectado:** `orders-service`.
- **Endpoint más comprometido:** `/orders/cancel`.
- **Mensaje dominante:** `Order creation failed - inventory lock timeout`.

Los resultados muestran una concentración de eventos malos en el flujo de órdenes durante la ventana crítica. Esta evidencia señala dónde se manifestó el problema, pero no demuestra por sí sola la causa raíz ni una propagación entre servicios. Para investigar esa secuencia sería necesario analizar los registros asociados a un mismo `trace_id`.

## Visualizaciones

El notebook genera exactamente dos gráficos:

1. Conteo de eventos por severidad en ventanas de cinco minutos.
2. Evolución del `bad_rate` por ventana, destacando visualmente el momento crítico.

## Tecnologías utilizadas

- [Python](https://www.python.org/)
- [Jupyter Notebook](https://jupyter.org/)
- [Pandas](https://pandas.pydata.org/)
- [Matplotlib](https://matplotlib.org/)

## Requisitos

- Python 3.11 o superior
- Pandas
- Matplotlib
- Jupyter Notebook

Instalación de dependencias:

```bash
python -m pip install pandas matplotlib notebook ipykernel
```

## Ejecución

1. Clonar el repositorio:

```bash
git clone <URL_DEL_REPOSITORIO>
```

2. Entrar en la carpeta del proyecto:

```bash
cd Challenge_6-LogsInterpretation
```

3. Colocar `server_logs.csv` en el mismo directorio que el notebook.

4. Iniciar Jupyter Notebook:

```bash
jupyter notebook
```

5. Abrir el archivo `.ipynb` y ejecutar todas las celdas desde el inicio.

## Estructura sugerida

```text
Challenge_6-LogsInterpretation/
├── README.md
├── server_logs.csv
├── logs_interpretation.ipynb
└── .gitignore
```

## Reproducibilidad

El notebook está organizado para ejecutarse de arriba hacia abajo. Las tablas, métricas, gráficos y conclusiones se generan a partir de los datos, sin modificar manualmente los resultados.

## Bonus opcional

Como análisis adicional, se puede seleccionar un `trace_id` presente en el momento crítico, ordenar sus registros por `timestamp_event` e identificar:

- El primer evento malo.
- Los servicios participantes.
- La secuencia temporal de la solicitud.

Este análisis puede aportar evidencia sobre una posible propagación del incidente, aunque una secuencia temporal por sí sola no prueba causalidad absoluta.
